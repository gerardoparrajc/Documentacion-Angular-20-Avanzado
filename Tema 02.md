# 2. Angular Signals en profundidad y su interoperabilidad con RxJS

## 2.1 Creación y uso de Signals en Angular 20

En Angular 20 los Signals pasan a ser la forma principal de manejar estado reactivo. La idea es sencilla: un Signal guarda un valor actual y Angular sabe quién lo está usando para poder actualizar justo lo que hace falta cuando cambia. A diferencia de un `Observable` de RxJS (que representa una secuencia de valores con el tiempo), un Signal representa “el valor ahora mismo”. Lo lees llamándolo como si fuera una función (`contador()`) y te da siempre el último dato. Así Angular evita repasar toda la interfaz y se centra en lo mínimo necesario.

### 2.1.1 Creación básica de un Signal

```ts
import { signal } from '@angular/core';

// WritableSignal<number>
const counter = signal(0);

// Lectura (NO se hace counter.value, se invoca):
console.log(counter()); // 0

// Escritura explícita:
counter.set(5);         // ahora vale 5

// Actualización derivada del valor actual:
counter.update(v => v + 1); // 6
```

Cuando lees `counter()` dentro de un cálculo (`computed`) o de un efecto (`effect` que veremos más adelante), Angular toma nota: “este cálculo depende de ese Signal”. Después, si el valor cambia, solo recalcula lo que depende de él. Para cambiar un Signal tienes dos opciones: `set(nuevoValor)` cuando ya sabes el valor final y `update(fn)` cuando necesitas partir del valor anterior (por ejemplo sumar 1). No hay suscripciones ni `unsubscribe`: Angular lleva la cuenta automáticamente usando versiones internas (cada cambio incrementa una “versión” del valor). Si nadie cambió el valor, leerlo es prácticamente gratis.

Casos típicos para elegir uno u otro:
- `set`: ya obtuviste el resultado (respuesta HTTP, formulario validado, etc.).
- `update`: quieres basarte en el valor anterior sin escribir `const actual = s(); s.set(actual+1);`.

Veamos ejemplos sencillos, fijándote en que siempre creamos nuevas referencias para arrays u objetos (nada de modificar en sitio) porque así Angular detecta claramente que hubo cambio.

```ts
// Toggling boolean
const open = signal(false);
const toggle = () => open.update(v => !v);

// Acumulando en un array (IMPORTANTE: crear nueva referencia)
const items = signal<string[]>([]);
function addItem(label: string) {
	items.update(list => [...list, label]);
}

// Reemplazo parcial de un objeto (crear nuevo objeto, no mutar in-place)
interface User { id: string; name: string; active: boolean; }
const user = signal<User>({ id: 'u1', name: 'Ana', active: true });
function deactivate() {
	user.update(u => ({ ...u, active: false }));
}
```

Resumen rápido de buenas prácticas aquí:
1. No hagas `items().push(...)`; crea un nuevo array (`[...items(), nuevo]`).
2. Si el dato cambia “de verdad”, usa una nueva referencia (objeto/array nuevo).
3. No busques una propiedad `.value`: se lee invocando el Signal.

Si vienes de `BehaviorSubject`, verás el paralelismo: `signal(0)` ≈ `new BehaviorSubject(0)`, leer es `s()` en vez de `b$.value`, escribir es `set` o `update` en lugar de `next`. La ventaja: nada de suscripciones ni fugas. RxJS sigue siendo útil para flujos asíncronos complejos (cancelaciones, combinaciones avanzadas), pero para estado local sincronizado el Signal resulta más directo.

Errores típicos al empezar:
- Crear Signals para variables temporales que solo viven dentro de una función (no hace falta).
- Dividir un mismo concepto en demasiados Signals pequeños sin necesidad (dificulta entender el estado).
- Mutar estructuras internas sin cambiar la referencia.

Rendimiento en una frase: leer un Signal que no ha cambiado es muy barato y no dispara recálculos innecesarios.

### 2.1.2 Control de mutabilidad y encapsulación

Para mantener el código limpio a medida que la app crece, conviene no exponer directamente los Signals que se pueden cambiar. Patrón simple: un Signal privado (mutable) y uno público de solo lectura. Además expones métodos claros que describen acciones (`increment`, `reset`, `add`, etc.). Así controlas dónde y cómo cambia el estado.

```ts
import { signal } from '@angular/core';

class CounterStore {
	// Interno mutable
	private readonly _count = signal(0);
	// Público inmutable
	readonly count = this._count.asReadonly();

	increment() { this._count.update(v => v + 1); }
	reset() { this._count.set(0); }
}
```

`asReadonly()` devuelve la versión de solo lectura: los componentes pueden mirar el valor pero no cambiarlo. Nombrar bien ayuda: sustantivos para datos (`count`, `tasks`) y verbos para acciones (`add`, `toggle`). Cuando necesitas datos derivados (totales, pendientes, etc.) usas `computed` y evitas duplicar lógica en muchos sitios.

Veamos un ejemplo algo más completo con validación ligera y datos derivados:

```ts
import { signal, computed } from '@angular/core';

interface Task { id: string; title: string; done: boolean; }

class TasksStore {
	private readonly _tasks = signal<Task[]>([]);
	readonly tasks = this._tasks.asReadonly();

	// Derivado privado (no expuesto) para lógica interna
	private readonly _completedCount = computed(() =>
		this._tasks().filter(t => t.done).length
	);

	// Derivado público agregado
	readonly stats = computed(() => ({
		total: this._tasks().length,
		completed: this._completedCount(),
		pending: this._tasks().length - this._completedCount()
	}));

	add(title: string) {
		const trimmed = title.trim();
		if (!trimmed) return; // invariante: no strings vacíos
		this._tasks.update(list => [
			...list,
			{ id: crypto.randomUUID(), title: trimmed, done: false }
		]);
	}

	toggle(id: string) {
		this._tasks.update(list => list.map(t =>
			t.id === id ? { ...t, done: !t.done } : t
		));
	}

	remove(id: string) {
		this._tasks.update(list => list.filter(t => t.id !== id));
	}
}
```

El componente no necesita saber cómo se calculan las estadísticas: solo las usa. Decide si expones varias propiedades o un objeto agrupado según se usen siempre juntas o no. Errores comunes aquí: exponer el Signal mutable, crear getters que no añaden nada o mezclar llamadas HTTP dentro de métodos que solo deberían actualizar estado (mejor: primero obtienes datos, luego haces `set`).

Idea clave: métodos claros para cambiar; Signals (o `computed`) claros para leer.

## 2.2 Signals computados (`computed`) y cálculo derivado 

Un `computed` es un Signal “de lectura” que se obtiene a partir de otros Signals. Solo se recalcula cuando cambia algo de lo que leyó la última vez. Si nada cambió, devuelve el resultado anterior al instante. Esto hace barato tener muchos `computed` pequeños y claros.

Piensa en `computed` como “qué valor quiero mostrar o usar”, no “qué acción quiero ejecutar”. Dentro de un `computed` intenta no hacer tareas externas (no peticiones HTTP, no escribir logs cada vez); limítalo a combinar y transformar datos.

Ejemplo típico: filtrar y calcular un promedio. Solo debería recalcularse si cambia la lista original o los filtros, no por otros detalles de la interfaz.

```ts
import { signal, computed } from '@angular/core';

interface Product { id: string; name: string; category: string; price: number; stock: number; }

const allProducts = signal<Product[]>([]);
const filters = signal<{ category?: string; minStock?: number }>({});

const visibleProducts = computed(() => {
	const { category, minStock } = filters();
	const list = allProducts();
	return list.filter(p =>
		(category ? p.category === category : true) &&
		(minStock != null ? p.stock >= minStock : true)
	);
});

const averagePrice = computed(() => {
	const list = visibleProducts(); // dependemos de la derivación previa
	if (!list.length) return 0;
	return list.reduce((acc, p) => acc + p.price, 0) / list.length;
});
```

Fíjate: `averagePrice` depende de `visibleProducts`, y esta a su vez de la lista original y los filtros. Si cambias cómo filtras internamente, el resto no se entera; solo lee el resultado.

`untracked` (opcional) sirve para leer un Signal dentro de un `computed` sin que cuente como dependencia. Úsalo solo si estás seguro de que un cambio en ese valor NO debería recalcular el resultado (casos raros). Si sí debe recalcularse, entonces no uses `untracked`.

Los `computed` son “perezosos”: si nadie los lee, no se calculan. Esto viene genial si tienes pestañas con cálculos pesados: solo la pestaña visible fuerza el cálculo.

Consejos en cálculos intensivos:
- Prepara tus datos base en estructuras cómodas (diccionario, arrays ordenados) para no rehacer trabajo.
- Separa responsabilidades: uno ordena, otro filtra, otro calcula un total.

Evita ciclos: un `computed` no debería, directa o indirectamente, depender de sí mismo. Si quieres guardar un histórico, usa otro Signal para almacenar ese histórico.

Integración con RxJS: puedes pasar de `Observable` a Signal con `toSignal(observable$, { initialValue })` y luego crear `computed` encima. Si necesitas el proceso inverso: `toObservable(miComputed)`.

¿Muchos pequeños o uno grande? Mejor varios pequeños con nombres claros: facilitan pruebas y entender qué cambió. En plantillas basta con `{{ averagePrice() }}`; no caches manualmente salvo que haya una operación pesada ajena al cálculo.

Regla rápida para crear un `computed`: ¿tiene sentido ponerle un nombre de negocio? Si sí (“productosVisibles”, “totalConImpuestos”), probablemente merece uno. Si es algo súper puntual, puede ir inline.

Resumen: base mutable mínima, `computed` para valores derivados y nada de lógica externa allí dentro.

## 2.3 Gestión de efectos reactivos con `effect()`

Los `computed` calculan valores. Los `effect` hacen cosas “hacia fuera”: log, guardar en `localStorage`, arrancar un temporizador, escuchar eventos del navegador, etc. Un efecto se ejecuta al inicio (cuando alguien lo crea) y después cada vez que cambia algún Signal que se leyó dentro.

Regla mental: si lo que quieres es “obtener un valor”, usa `computed`. Si lo que quieres es “disparar una acción externa cuando cambie algo”, usa `effect`.

```ts
import { signal, effect } from '@angular/core';

const searchQuery = signal('');
const resultsCount = signal(0);

effect(() => {
	console.log('Consulta actual:', searchQuery(), 'Total:', resultsCount());
});
```

Cada ejecución lee los valores actuales y se guarda qué Signals se usaron. Cuando cualquiera cambia, se vuelve a ejecutar. Punto importante: muchos efectos crean recursos que hay que limpiar (intervalos, listeners...). Angular te da un callback para eso: `onCleanup`.

```ts
import { signal, effect } from '@angular/core';

const tickInterval = signal(1000);
const tick = signal(0);

effect(onCleanup => {
	const id = setInterval(() => tick.update(v => v + 1), tickInterval());
	onCleanup(() => clearInterval(id));
});

// Cambiar el intervalo reinicia el timer limpiamente
tickInterval.set(500);
```

Si cambias el intervalo en el ejemplo, el anterior se limpia y se crea uno nuevo. Simple y sin fugas. Si un efecto necesita un cálculo pesado, haz ese cálculo en un `computed` y que el efecto solo lo use (no lo reconstruyas dentro cada vez).

```ts
import { signal, computed, effect } from '@angular/core';

const cart = signal<{ id: string; qty: number; price: number }[]>([]);
const snapshot = computed(() => JSON.stringify(cart()));

effect(() => {
	// Persistencia minimalista
	localStorage.setItem('cart_snapshot', snapshot());
});
```

Ventaja importante: no gestionas suscripciones manuales. Angular recuerda las dependencias y limpia cuando corresponde.

Anti‑patrones comunes:
1. Usar un efecto para calcular un valor que luego necesitas leer (eso es un `computed`).
2. Cadena de efectos donde uno solo prepara datos para el siguiente. Mejor usa `computed` intermedios.
3. Leer distintos Signals según ramas condicionales que cambian todo el rato (depurar se vuelve difícil).

## 2.4 Uso avanzado de `linkedSignal` y `resource` para orquestar flujos de datos

En este punto ya tienes: Signals para estado directo, `computed` para derivar y `effect` para actuar fuera. Nos faltan dos piezas que ayudan en escenarios donde hay que sincronizar varios Signals a la vez o cargar datos asíncronos de forma declarativa: `linkedSignal` y `resource`.

### 2.4.1 `linkedSignal`: un puente de ida y vuelta

`computed` es de solo lectura. A veces, sin embargo, quieres un “valor derivado” que también permita escritura y que esa escritura se reparta entre varios Signals base. Para eso está `linkedSignal`. Piensa en él como un `computed` con setter manual.

Caso sencillo: estado local derivado de un @input() con reseteo automático (selector de categoría).

Un componente recibe una lista (input signal) y necesita un “seleccionado” que arranque con un valor por defecto, pero que también pueda ser cambiado por el usuario sin perder la reactividad cuando cambie la lista.

```ts
import { Component, input, linkedSignal, Signal } from '@angular/core';

export interface Category { id: number; name: string; }

@Component({
  selector: 'app-category-picker',
  standalone: true,
  template: `
    <label>Categoría:</label>
    <select [ngModel]="selectedCategory()?.id" (ngModelChange)="onPick($event)">
      @for (c of categories(); track c.id) {
        <option [value]="c.id">{{ c.name }}</option>
      }
    </select>

    <p>Seleccionada: {{ selectedCategory()?.name }}</p>
  `
})
export class CategoryPickerComponent {
  // input() en Angular moderno crea un InputSignal (no escribible desde dentro)
  categories = input.required<Category[]>();

  // Arranca con la primera categoría; si cambia 'categories', se recalcula
  selectedCategory = linkedSignal(() => this.categories()[0]);

  onPick(id: number) {
    const cat = this.categories().find(c => c.id === +id)!;
    this.selectedCategory.set(cat); // writable!
  }
}
```


**Por qué `linkedSignal()` y no `computed()`**: aquí queremos *derivar* el valor inicial de categories, **pero también** permitir cambios locales con `.set()`; `computed()` no es escribible.

Otro ejemplo útil: respetar la selección previa si sigue siendo válida.

La fuente cambia (lista remota, filtros, etc.) y no quieres que el seleccionado vuelva siempre al primer elemento; si el item todavía existe, mantén la elección del usuario.

```ts
import { Component, input, linkedSignal, Signal } from '@angular/core';

interface ShippingMethod { id: number; name: string; }

@Component({
  selector: 'app-shipping-method',
  standalone: true,
  template: `
    @if (methods().length) {
      <div class="methods">
        @for (m of methods(); track m.id) {
          <button
            type="button"
            [class.active]="selected()?.id === m.id"
            (click)="selected.set(m)">
            {{ m.name }}
          </button>
        }
      </div>
      <p>Usando: <strong>{{ selected()?.name }}</strong></p>
    } @else {
      <p>No hay métodos disponibles.</p>
    }
  `,
  styles: [`.active { font-weight: bold; }`]
})
export class ShippingMethodComponent {
  methods = input.required<ShippingMethod[]>();

  selected = linkedSignal<ShippingMethod | null, ShippingMethod | null>({
    source: () => this.methods(), // fuente explícita
    computation: (currentList, previous) => {
      if (!currentList.length) return null;
      // Si la selección previa sigue existiendo, mantenla
      const stillThere = previous && currentList.some(m => m.id === previous.id);
      return stillThere ? previous! : currentList[0];
    }
  });

  track = (_: number, m: ShippingMethod) => m.id;
}
```

Cuándo usarlo:
- Formularios que manipulan varias piezas de estado como un solo campo.
- Adaptar una estructura interna a un formato que espera un componente externo y permitir que ese componente escriba de vuelta.

Cuándo NO usarlo:
- Si solo necesitas lectura -> usa `computed`.
- Si la escritura “derivada” es compleja y llena de reglas, quizá es mejor exponer métodos explícitos (más claro que un set opaco).

### 2.4.2 `resource`: cargar datos asíncronos como Signals

`resource` (estable en Angular 20) ofrece una forma declarativa de gestionar peticiones asíncronas atadas a parámetros reactivos. En lugar de: escuchar cambios, cancelar peticiones anteriores manualmente, gestionar estados loading/error…, defines la receta una vez y `resource` se encarga.

Idea básica: le das una función `loader` que recibe los parámetros actuales (derivados de Signals) y un objeto con utilidades como `abortSignal`. El resultado se expone como un objeto Signal‑like con estados (`status`, `value`, `error`) y métodos para refrescar.

Ejemplo: cargar información de un usuario según su id.

```ts
import { signal, resource } from '@angular/core';

public userId = signal('1');

public userResource = resource({
	// De qué depende (parámetros). Cambia -> se vuelve a cargar.
	params: () => ({ id: userId() }),
	// Cómo cargar
	loader: async ({ params, abortSignal }) => {
		const res = await fetch(`/api/users/${params.id}`, { signal: abortSignal });
		if (!res.ok) throw new Error('No se pudo cargar el usuario');
		return res.json();
	}
});

// En un template podrías usar:
// @if (userResource.status() === 'loading') { Cargando... }
// @else if (userResource.status() === 'error') { Error: {{ userResource.error()?.message }} }
// @else { {{ userResource.value()?.name }} }

userId.set('2'); // Dispara nueva carga; la anterior se aborta automáticamente
```

Notas clave:
- `params` recoge los valores que forman el “cache key” o la identidad de la petición.
- Si cambia algo en `params`, se cancela la petición anterior (si sigue en vuelo) usando `AbortController` y se lanza otra.
- `status()` suele devolver: `idle` | `loading` | `success` | `error`.
- `value()` es el dato ya resuelto (solo disponible en `success`).
- `error()` contiene la excepción capturada (solo en `error`).

Podemos añadir control manual de refresco (por ejemplo un botón “Reintentar”):

```ts
// Forzar recarga sin cambiar el id
userResource.reload();
```

Ejemplo un poco más completo con paginación y transformación:

```ts
import { JsonPipe } from '@angular/common';
import { Component, computed, resource, signal } from '@angular/core';

@Component({
  selector: 'app-root',
  imports: [JsonPipe],
  templateUrl: './app.html',
  styleUrl: './app.scss'
})
export class App {
  public limit = signal(10);
  public skip = signal(10);

  // Derivado que usamos como parámetros (buena práctica: agrupar)
  public pagination = computed(() => ({ limit: this.limit(), skip: this.skip() }));

  public listResource = resource({
    params: () => this.pagination(),
    loader: async ({ params, abortSignal }) => {
      const url = `https://dummyjson.com/posts?limit=${params.limit}&skip=${params.skip}`;
      const res = await fetch(url, { signal: abortSignal });
      if (!res.ok) throw new Error('Fallo cargando items');
      const json = await res.json();
      // Transformación ligera
      return {
        items: json.posts.map((i: any) => ({ id: i.id, title: i.title })),
        total: json.total
      };
    }
  });

  public  nextPage() { this.skip.update(p => p + 10); }
  public  prevPage() { this.skip.update(p => Math.max(0, p - 10)); }

}

```

En la plantilla podrías mostrar un indicador de carga mientras `status() === 'loading'`, la lista cuando esté en `resolved` y un bloque de error con botón `listResource.reload()` en caso de fallo.

```html
<button (click)="prevPage()">Anterior</button>
<button (click)="nextPage()">Siguiente</button>

@if (listResource.status() === 'loading') {
  <p>Cargando...</p>
}

@if (listResource.status() === 'error') {
  <p>Se ha producido un error. Inténta cargar de nuevo.</p>
  <button (click)="listResource.reload()">Recargar</button>
}

@if (listResource.status() === 'resolved') {
  <pre>
    {{ listResource.value()  | json}}
  </pre>
}
```

### 2.4.3 Estrategias y pequeños trucos

1. Separar parámetros: si tienes varias partes que forman la identidad de la petición (por ejemplo filtros + paginación), crea un `computed` que los agrupe y úsalo en `params`. Así un solo objeto determina cuándo recargar.
2. Evita meter lógica de formateo pesada dentro de `loader`. Mejor: devuelve datos limpios y crea `computed` encima si hace falta.
3. Control de errores: si necesitas un mensaje amigable, puedes mapear `error()` a otro `computed` que traduzca `Error` → texto de UI.
4. Limitar refrescos: si un parámetro cambia demasiado rápido (tecleando), añade un Signal “estable” que solo se actualice tras un debounce manual y úsalo en `params`.
5. No mezcles efectos redundantes: para actualizar vistas, la propia lectura de `status()/value()/error()` en la plantilla ya reactiva el flujo.

### 2.4.4 Decidir qué usar en cada caso

| Necesito | Herramienta |
|----------|-------------|
| Combinar varios Signals en algo editable | `linkedSignal` |
| Solo derivar valor de lectura | `computed` |
| Lanzar lógica externa al cambiar algo | `effect` |
| Cargar datos async con gestión de estados y cancelación | `resource` |

Piensa en flujos complejos como capas: parámetros (Signals) → derivaciones (`computed`) → carga (`resource`) → acciones externas puntuales (`effect`) → UI. Ese orden evita mezclar responsabilidades y hace que añadir una nueva pieza (un filtro, una métrica, un formato) sea un paso pequeño y seguro.

## 2.5 Interoperabilidad entre Signals y Observables (`toSignal`, `toObservable`)  

Signals no sustituyen a RxJS: se complementan. RxJS sigue siendo muy fuerte para flujos asíncronos complejos (cancelación avanzada, combinación de múltiples fuentes, backpressure) y Signals brillan para representar el “estado actual” y derivarlo de forma simple. La interoperabilidad se basa en dos funciones del paquete `@angular/core/rxjs-interop`: `toSignal` y `toObservable`.

### 2.5.1 De Observable a Signal: `toSignal`

Convierte la emisión más reciente de un `Observable` en un Signal de lectura. Necesitas un `initialValue` (o `requireSync: true` cuando el observable emite de inmediato de forma síncrona). A partir de ahí, cada nueva emisión actualiza el Signal.

Ejemplo: parámetros de ruta (Observable) → Signal + derivado.

```ts
import { Component, computed, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { map } from 'rxjs/operators';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
	template: `
		<h3>Usuario: {{ userId() }}</h3>
		<p>Mayúsculas: {{ userIdUpper() }}</p>
	`
})
export class UserDetailComponent {
	private route = inject(ActivatedRoute);

	// paramMap es un Observable, lo convertimos
	userId = toSignal(
		this.route.paramMap.pipe(map(p => p.get('id') ?? '')),
		{ initialValue: '' }
	);

	userIdUpper = computed(() => this.userId().toUpperCase());
}
```

Si el observable puede no emitir nunca (o emite tarde), `initialValue` evita estados `undefined` en la plantilla. Si sabes que siempre emite inmediatamente (por ejemplo un `BehaviorSubject`), puedes usar `requireSync: true` y omitir `initialValue`. Si no emite síncrono y usas `requireSync`, Angular lanzará un error para avisarte.

Caso con HTTP (lo normal es que uses directamente Signals para estado, pero es ilustrativo):

```ts
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';

class VersionService {
	constructor(private http: HttpClient) {}

	// Exponemos como Signal la última versión obtenida (una sola petición)
	version = toSignal(this.http.get<{ version: string }>('/api/version'), {
		initialValue: { version: '...' }
	});
}
```

### 2.5.2 De Signal a Observable: `toObservable`

Convierte un Signal en un `Observable` que emite cada vez que cambia el valor del Signal. Útil para APIs o librerías que esperan Observables (por ejemplo ciertos operadores de router, combinaciones con streams de sockets, etc.).

```ts
import { signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { map } from 'rxjs/operators';

const count = signal(0);
const count$ = toObservable(count);

// Podemos seguir usando operadores RxJS
const doubled$ = count$.pipe(map(v => v * 2));

const sub = doubled$.subscribe(v => console.log('Doble:', v));
count.set(2); // Emite 4
sub.unsubscribe();
```

El observable resultante se “completa” implícitamente cuando el Signal deja de existir (por ejemplo si era interno a un componente que se destruye). Esto reduce riesgos de fugas.

## 2.6 Patrones de migración desde `Subject` y `BehaviorSubject` hacia Signals

Esta sección es una guía para “despejar la mesa” de RxJS cuando solo lo estabas usando como contenedor de estado. No buscamos eliminar RxJS (lo seguiremos usando para lo temporal complejo), sino mover a Signals aquello que realmente representa un valor actual simplificable. Imagina que te sientas frente a tu código y haces una pequeña conversación contigo mismo antes de tocar nada: eso es lo que haremos aquí, pensando cada decisión “en voz alta”.

### 2.6.1 Pensar en voz alta: ¿Migro esto… sí o no?

Tomamos cada pieza de estado/stream y nos hacemos estas preguntas (monólogo interno incluido):
1. ¿Estoy usando un `BehaviorSubject` solo para tener “el último valor” y acceder con `.value`? → “Sí.” Entonces: Signal.
2. ¿Tengo una cadena de `combineLatest(...).pipe(map(...))` que solo combina valores sin operadores de tiempo (`debounce`, `switchMap`, `retry`, `bufferTime`, etc.)? → “Solo mezcla datos.” Entonces: `computed`.
3. ¿Puse `shareReplay(1)` solo para que los nuevos suscriptores reciban el último valor? → “Exacto.” Entonces: ya no necesito eso; un Signal cachea por definición.
4. ¿Estoy gestionando manualmente triple estado `loading/error/data`? → “Sí, con tres Subjects.” Entonces: `resource` lo unifica.
5. ¿Hay lógica temporal (debounce, backoff, retrys, websockets) que vive bien en RxJS? → “Sí, bastante.” Entonces: conservo esa parte y cierro el flujo final en un Signal con `toSignal`.
6. ¿Es un `Subject` que solo emite ‘evento ocurrió’ (sin datos complejos)? → “Sí, un click de refresco.” Entonces: quizá reemplazar por un simple `signal` contador + método, o integrarlo en un `resource.reload()`.

Hecho esto, marcamos qué migrar primero: lo simple y local (bajo riesgo) y dejamos para después lo que mezcla tiempo, concurrencia o librerías externas.

### 2.6.2 Escenario 1 – `BehaviorSubject` trivial → `signal`

Antes (mucho ruido para algo simple):
```ts
private readonly _count$ = new BehaviorSubject(0);
public readonly count$ = this._count$.asObservable();

increment() { this._count$.next(this._count$.value + 1); }
reset() { this._count$.next(0); }
```
Problemas: `.value`, `.next`, exposición doble (`_count$` / `count$`).

Después (intención limpia):
```ts
private readonly _count = signal(0);
public readonly count = this._count.asReadonly();

increment() { this._count.update(v => v + 1); }
reset() { this._count.set(0); }
```
Comentario: Desaparecen conceptos accidentales; la plantilla usa `{{ count() }}` sin suscripción.

### 2.6.3 Escenario 2 – Varios `BehaviorSubject` + `combineLatest` → Signals + `computed`

Antes (tres fuentes + mezcla):
```ts
filters$ = new BehaviorSubject<{ search: string; category: string | null }>({ search: '', category: null });
items$   = new BehaviorSubject<Item[]>([]);
sort$    = new BehaviorSubject<'asc' | 'desc'>('asc');

visible$ = combineLatest([items$, filters$, sort$]).pipe(
	map(([items, f, s]) => {
		let out = items.filter(i => !f.search || i.name.includes(f.search));
		if (f.category) out = out.filter(i => i.category === f.category);
		return out.sort((a,b) => s === 'asc' ? a.price - b.price : b.price - a.price);
	})
);
```
Después (código declarativo y legible):
```ts
const items = signal<Item[]>([]);
const filters = signal<{ search: string; category: string | null }>({ search: '', category: null });
const sort = signal<'asc' | 'desc'>('asc');

const visible = computed(() => {
	const { search, category } = filters();
	const s = sort();
	let list = items();
	if (search) list = list.filter(i => i.name.includes(search));
	if (category) list = list.filter(i => i.category === category);
	return [...list].sort((a,b) => s === 'asc' ? a.price - b.price : b.price - a.price);
});
```
Comentario: Eliminamos suscripciones; cada cambio recalcula lo justo.

### 2.6.4 Escenario 3 – `Subject` de eventos → función/método o `resource`

Caso: un `Subject<void>` que solo dispara un refresco.

Antes:
```ts
private refreshClicks$ = new Subject<void>();
refresh$ = refreshClicks$.pipe(
	switchMap(() => this.http.get<Data>('/api/data'))
);

triggerRefresh() { refreshClicks$.next(); }
```
Opción A (conservo canalización RxJS pero el resultado final lo trato como valor):
```ts
private readonly _refresh = signal(0);
refreshData = toSignal(
	toObservable(this._refresh).pipe(
		switchMap(() => this.http.get<Data>('/api/data'))
	), { initialValue: undefined }
);

triggerRefresh() { this._refresh.update(v => v + 1); }
```
Opción B (lo que realmente quería: cargar y tener estados):
```ts
const reloadVersion = signal(0);
const dataRes = resource({
	params: () => reloadVersion(),
	loader: async ({ abortSignal }) => {
		const r = await fetch('/api/data', { signal: abortSignal });
		if (!r.ok) throw new Error('Error');
		return r.json();
	}
});
triggerRefresh() { reloadVersion.update(v => v + 1); }
```
Decisión: si necesito operadores temporales → Opción A; si solo es “carga y estados” → Opción B.

### 2.6.5 Escenario 4 – `shareReplay(1)` heredado

Antes (cache manual implícita):
```ts
stats$ = this.http.get<Data>('/api/stats').pipe(shareReplay(1));
```
Después con `resource` (gestión de estados incluida):
```ts
const statsRes = resource({
	loader: async () => {
		const r = await fetch('/api/stats');
		if (!r.ok) throw new Error('Error');
		return r.json();
	}
});
```
O si era algo que se carga una sola vez y ya: una línea `fetch` y un `signal` bastan. El 80% de `shareReplay(1)` “solo para cache” desaparece.

### 2.6.6 Escenario 5 – Triple `loading$` / `error$` / `data$`

Antes (verboso y propenso a olvidos en algún camino de error):
```ts
loading$ = new BehaviorSubject(false);
error$ = new BehaviorSubject<string | null>(null);
data$ = new BehaviorSubject<Data | null>(null);

load() {
	loading$.next(true);
	error$.next(null);
	this.http.get<Data>('/api/items').subscribe({
		next: d => { data$.next(d); loading$.next(false); },
		error: () => { error$.next('Fallo'); loading$.next(false); }
	});
}
```
Después (estado cohesionado):
```ts
const itemsRes = resource({
	params: () => 0,
	loader: async ({ abortSignal }) => {
		const r = await fetch('/api/items', { signal: abortSignal });
		if (!r.ok) throw new Error('Fallo');
		return r.json();
	}
});
```
La plantilla reacciona a `itemsRes.status()` sin necesitar 3 suscripciones y sincronización manual.

### 2.6.7 Escenario 6 – `combineLatest` + `map` puramente derivado

Antes:
```ts
summary$ = combineLatest([a$, b$, c$]).pipe(
	map(([a, b, c]) => a.total + b.total - c.discount)
);
```
Después:
```ts
const summary = computed(() => a().total + b().total - c().discount);
```
La semántica es idéntica; desaparece el plumbing de Rx.

### 2.6.8 Dos caminos para “búsqueda con debounce”: RxJS vs Signals + `resource`

Opción RxJS (ideal si ya tienes operadores dominados):
```ts
const query = signal('');
const results = toSignal(
	toObservable(query).pipe(
		debounceTime(300),
		distinctUntilChanged(),
		switchMap(q => this.http.get<Result[]>(`/api/search?q=${q}`))
	), { initialValue: [] }
);
```
Opción pura Signals (sin operadores, control manual del debounce):
```ts
const raw = signal('');
const stable = signal('');
let t:any;
function setQuery(q:string){
	raw.set(q);
	clearTimeout(t);
	t = setTimeout(() => stable.set(q), 300);
}
const searchRes = resource({
	params: () => stable(),
	loader: async ({ params, abortSignal }) => {
		if (!params) return [];
		const r = await fetch(`/api/search?q=${encodeURIComponent(params)}`, { signal: abortSignal });
		if (!r.ok) throw new Error('Error');
		return r.json();
	}
});
```
Comparación rápida:
| Criterio | RxJS + toSignal | Debounce manual + resource |
|----------|-----------------|-----------------------------|
| Expresividad temporal | Muy alta (más operadores) | Básica (debes codificar) |
| Complejidad inicial | Mayor si no conoces operadores | Muy baja (setTimeout) |
| Cancelación HTTP | `switchMap` lo hace | `resource` + abortSignal |
| Escalado a casos raros | Fácil (añadir más operators) | Puede volverse ad-hoc |

Regla: si ya es un pipeline RxJS, mantenlo; si no, empieza simple con Signals.

### 2.6.9 Patrones mixtos (conectar mundos)

Hay casos donde quieres sacar un Signal a RxJS o traer RxJS a Signals:
- Tengo un Signal y necesito `auditTime`, `bufferCount` → `toObservable(signal).pipe(...operadores...)`.
- Tengo un Observable websocket que emite valores → proceso con operadores → cierro en `toSignal` para leer en plantilla.
- Un `resource` me da `value()` pero necesito tratarlo como stream (para un operador puntual): `toObservable(miResource.value)` (o envolver con un `computed`).

Piensa “borde de traducción” claro y evita conversiones ping‑pong.

### 2.6.10 Checklist práctica de migración

Para cada pieza:
1. ¿Es valor actual y accedo con `.value`? → `signal`.
2. ¿Solo combino sin tiempo? → `computed`.
3. ¿Uso `shareReplay(1)`? → evaluar cambio a `signal` / `resource`.
4. ¿Gestiono manualmente loading/error/data? → `resource`.
5. ¿Hay debounce/switchMap/retry/backoff? → dejar en RxJS y cerrar con `toSignal`.
6. ¿Solo disparo un evento sin datos? → método + `signal` contador o `resource.reload()`.
7. ¿Necesito escritura derivada sobre combinación? → quizá `linkedSignal` (no forzar un `Subject`).

### 2.6.11 Tabla comparativa final

| Caso original (RxJS) | Señal / Herramienta destino | Motivo del cambio | Comentario extra |
|----------------------|-----------------------------|------------------|------------------|
| `BehaviorSubject` estado simple | `signal` + `asReadonly()` | El `.value` desaparece | Mutaciones claras con `update` |
| Varios `BehaviorSubject` + `combineLatest` puro | Signals + `computed` | Derivación sin temporalidad | Menos boilerplate |
| `Subject<void>` para refrescar | `signal` contador / `resource.reload()` | Evento sin payload | Incrementar número dispara recarga |
| `shareReplay(1)` solo cache | `signal` | Cache implícita | Menos operadores |
| loading$ / error$ / data$ | `resource` | Estado cohesionado | Cancelación automática |
| Búsqueda debounce RxJS | RxJS + `toSignal` | Temporalidad rica | Mantener operadores |
| Búsqueda simple | Debounce manual + `resource` | Evitar pipeline innecesario | Fácil de leer |
| combineLatest + map derivado | `computed` | Valor puro derivado | Recalcula selectivo |
| Flujo complejo (retry/backoff) | Mantener RxJS | Requiere operadores | Cerrar con `toSignal` si hace falta UI |
| Bus de eventos ligero | Métodos + Signals | Semántica explícita | Menos riesgo de abusar del bus |

## 2.7 Integración de Angular Signals con el cliente HTTP (`httpResource()`) 

En Angular 20, el sistema de **Signals** ha transformado la forma en que gestionamos el estado y la reactividad en nuestras aplicaciones. Y con la llegada de la función `httpResource()` en el paquete `@angular/common/http`, ahora podemos integrar de forma **declarativa** las peticiones HTTP directamente en este sistema reactivo, sin necesidad de suscripciones manuales ni `async pipe`.


### 2.7.1. ¿Qué es `httpResource()`?

`httpResource()` es una función que crea un **recurso reactivo** basado en una petición HTTP. Internamente utiliza `HttpClient`, por lo que sigue respetando interceptores, configuración global y utilidades de prueba que ya conoces.

La gran diferencia es que el resultado de la petición se expone como un **Signal**, lo que significa que:

- No necesitas suscribirte manualmente.
- Angular se encarga de cancelar peticiones cuando el recurso deja de usarse.
- Puedes reaccionar automáticamente a cambios en parámetros o URLs.


### 2.7.2. Uso básico

Supongamos que queremos obtener una lista de usuarios desde una API pública.

```ts
import { httpResource } from '@angular/common/http';
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-root',
  imports: [CommonModule],
  styleUrl: './app.css',
  template: `
    @switch (usersResource.status()) {
      @case ('loading') {
        <p>Loading...</p>
      }
      @case ('resolved') {
        <pre>{{ usersResource.value() | json }}</pre>
      }
      @case ('error') {
        <p>Error loading data:</p>
        <pre>{{ usersResource.error() | json }}</pre>
      }
    }
  `
})
export class App {
  public usersResource = httpResource(() => 'https://jsonplaceholder.typicode.com/users');
}

```

**Qué está pasando aquí:**

- `httpResource()` recibe una función que devuelve la URL de la petición.
- El resultado (`usersResource`) es un objeto con Signals como:
  - `.value()` → contiene la respuesta (o `null` si aún no ha llegado).
  - `.error()` → contiene el error si la petición falla.
  - `.status()` → indica el estado (`'loading'`, `'resolved'`, `'error'`).
- Angular gestiona automáticamente la petición y su ciclo de vida.


### 2.7.3. Reactividad con Signals

La verdadera potencia de `httpResource()` aparece cuando la URL depende de un **Signal**. Por ejemplo, si queremos cargar un usuario específico según su ID:

```ts
import { signal } from '@angular/core';

userId = signal(1);

userResource = httpResource(
  () => `https://jsonplaceholder.typicode.com/users/${this.userId()}`);
```

Cada vez que `userId` cambie, `httpResource()` volverá a ejecutar la petición automáticamente.

```html
<button (click)="userId.update(id => id + 1)">Siguiente usuario</button>
```


### 2.7.4. Control manual: recargar datos

Aunque `httpResource()` se actualiza automáticamente cuando cambian sus dependencias, también puedes forzar una recarga manual:

```ts
this.userResource.reload();
```

Esto es útil después de realizar una operación de escritura (POST, PUT, DELETE) y querer refrescar la vista.


### 2.7.5. Manejo de progreso y opciones avanzadas

`httpResource()` también permite:

- **Progreso de descarga**:
Para poder disponer de información relativa a los datos descargados, la configuración de `httpResource()` se deberá realizar de la siguiente forma:

  ```ts
  userResource(() => ({
	url: 'https://api.example.com/data',
	reportProgress: true	// Activa la funcionalidad de conteo de bytes descargados
  }))
  ```

Para utilizar los datos que devuelve este modo, utilizamos el signal `progress()`, el cual contendrá un objeto, donde la propiedad `loaded` contiene el número de bytes transferidos, y si está disponible, la propiedad `total` indicará el número de bytes totales de la transferencia.

  ```ts
  userResource.progress(); // Signal con HttpProgressEvent
  ```

- **Métodos distintos a GET**:  
  ```ts
  httpResource(() => ({
    url: 'https://api.example.com/data',
    method: 'POST',
    body: { name: 'Nuevo' }
  }));
  ```


### 2.7.6. Ventajas frente a `HttpClient` tradicional

- **Menos código repetitivo**: no necesitas `subscribe()` ni `async pipe`.
- **Cancelación automática**: evita fugas de memoria.
- **Reactividad declarativa**: se integra perfectamente con Signals.
- **Estados integrados**: carga, error y valor listos para usar.



## 2.8 Uso de operadores RxJS aplicados a Signals y viceversa

En Angular 20, la interoperabilidad entre Signals y RxJS es muy potente y flexible. Aunque los Signals están pensados para manejar el estado actual de forma reactiva y sencilla, hay ocasiones en las que los operadores de RxJS nos permiten realizar transformaciones, filtrados o combinaciones avanzadas que serían más complejas de implementar solo con Signals.

### 2.8.1 ¿Por qué combinar Signals y operadores RxJS?

Los operadores de RxJS (como `map`, `filter`, `debounceTime`, `switchMap`, etc.) son ideales para trabajar con flujos de datos asíncronos, temporales o que requieren lógica de transformación compleja. Los Signals, por su parte, son perfectos para representar el “valor actual” y derivar nuevos valores de forma declarativa y eficiente.

La clave está en saber cuándo conviene usar cada herramienta y cómo conectarlas:

- Si necesitas lógica temporal, operadores avanzados o integración con APIs que esperan Observables, usa RxJS.
- Si solo necesitas derivar valores actuales y mantener el estado sincronizado, Signals son más directos.

### 2.8.2 De Signal a Observable: aplicar operadores RxJS

Puedes convertir cualquier Signal en un Observable usando la función `toObservable` del paquete `@angular/core/rxjs-interop`. Una vez convertido, puedes aplicar cualquier operador de RxJS sobre ese Observable.

**Ejemplo práctico:**

Supongamos que tienes un Signal que representa el texto de búsqueda de un usuario, y quieres aplicar un debounce para evitar lanzar la búsqueda en cada pulsación:

```ts
import { signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
import { HttpClient } from '@angular/common/http';

public search = signal('');
public search$ = toObservable(search);

public results$ = search$.pipe(
	debounceTime(300),
	distinctUntilChanged(),
	switchMap(query => http.get(`/api/search?q=${query}`))
);

// Suscribimos para obtener los resultados
results$.subscribe(data => {
	// Actualiza el estado, muestra resultados, etc.
});
```

En este ejemplo, el Signal `search` se convierte en un Observable, sobre el que aplicamos operadores RxJS para controlar el flujo de peticiones HTTP.

### 2.8.3 De Observable a Signal: usar operadores y derivar estado

Si tienes un Observable (por ejemplo, de una API, un WebSocket, o una librería externa), puedes aplicar operadores RxJS y luego convertir el resultado en un Signal usando `toSignal`.

**Ejemplo práctico:**

```ts
import { toSignal } from '@angular/core/rxjs-interop';
import { map } from 'rxjs/operators';

public user$ = api.getUserStream(); // Observable que emite datos de usuario
public userName$ = user$.pipe(map(user => user.name.toUpperCase()));

public userNameSignal = toSignal(userName$, { initialValue: '' });

// Ahora puedes usar userNameSignal() en tu plantilla o lógica reactiva
```

Esto te permite aprovechar la potencia de RxJS para transformar los datos y luego integrarlos en el sistema de Signals de Angular, manteniendo la reactividad y la eficiencia.

### 2.8.4 Ejemplo completo: búsqueda reactiva con debounce y estado

Veamos un ejemplo donde combinamos ambos mundos para una búsqueda reactiva:

```ts
import { CommonModule } from '@angular/common';
import { HttpClient } from '@angular/common/http';
import { Component, inject, signal } from '@angular/core';
import { toObservable, toSignal } from '@angular/core/rxjs-interop';
import { FormsModule } from '@angular/forms';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

@Component({
  selector: 'app-root',
  imports: [CommonModule, FormsModule],
  template: `
    <input type="text" (input)="query.set($event.target.value)" />
    <pre>{{ resultsSignal() | json }}</pre>
  `  
})
export class App {
  public http = inject(HttpClient);
  public query = signal('');
  public query$ = toObservable(this.query);

  public results$ = this.query$.pipe(
    debounceTime(400),
    distinctUntilChanged(),
    switchMap(q => this.http.get(`https://dummyjson.com/posts/search?q=${q}`))
  );

  public resultsSignal = toSignal(this.results$, { initialValue: [] });
}
```

Así, el usuario puede escribir en el campo de búsqueda, el Signal `query` se actualiza, el Observable aplica el debounce y la petición HTTP, y el Signal final `resultsSignal` contiene los resultados listos para mostrar en la interfaz.

### 2.8.5 Recomendaciones y buenas prácticas

- Usa Signals para el estado local y derivaciones simples.
- Usa RxJS y sus operadores para lógica temporal, flujos asíncronos y transformaciones complejas.
- Conecta ambos mundos con `toObservable` y `toSignal` según lo que necesites en cada caso.
- Evita mezclar demasiada lógica en un solo lugar: separa la transformación de datos (RxJS) del estado actual (Signals).
- Recuerda que los Signals son más eficientes para leer el valor actual y evitar recálculos innecesarios.



## 2.9 Implementación de patrones arquitectónicos con Signals (state management)

En aplicaciones Angular avanzadas, la gestión del estado (state management) es clave para mantener el código organizado, predecible y fácil de mantener. Tradicionalmente, se han usado librerías como NgRx, Akita o incluso servicios con RxJS para este propósito. Sin embargo, con la llegada de Signals en Angular 20, podemos implementar patrones arquitectónicos robustos de forma más sencilla y directa, sin perder claridad ni escalabilidad.

### 2.9.1 ¿Por qué usar Signals para el state management?

Los Signals permiten:
- Representar el estado actual de manera reactiva y eficiente.
- Derivar datos fácilmente con `computed`.
- Encapsular la mutabilidad y exponer solo lo necesario.
- Evitar fugas de memoria y suscripciones manuales.

Esto facilita la implementación de patrones como el Store, el patrón de selector, y la separación clara entre lógica de negocio y presentación.

### 2.9.2 Patrón Store con Signals

El patrón Store consiste en centralizar el estado de una funcionalidad en una clase, exponiendo solo los datos y métodos necesarios. Con Signals, esto se vuelve muy natural:

```ts
import { signal, computed } from '@angular/core';

interface Product {
	id: string;
	name: string;
	price: number;
	stock: number;
}

export class ProductsStore {
	// Estado privado
	private readonly _products = signal<Product[]>([]);
	private readonly _filter = signal<string>('');

	// Estado público (solo lectura)
	readonly products = this._products.asReadonly();
	readonly filter = this._filter.asReadonly();

	// Selector derivado
	readonly visibleProducts = computed(() => {
		const f = this._filter().toLowerCase();
		return this._products().filter(p => p.name.toLowerCase().includes(f));
	});

	// Métodos para modificar el estado
	setFilter(value: string) {
		this._filter.set(value);
	}

	addProduct(product: Product) {
		this._products.update(list => [...list, product]);
	}

	removeProduct(id: string) {
		this._products.update(list => list.filter(p => p.id !== id));
	}
}
```

**Ventajas:**
- El estado está encapsulado y solo se modifica a través de métodos.
- Los componentes solo leen el estado necesario y reaccionan automáticamente a los cambios.
- Los datos derivados (`visibleProducts`) se recalculan solo cuando cambian las dependencias.

### 2.9.3 Selectores y lógica derivada

Los selectores (`computed`) permiten obtener datos derivados del estado base, evitando duplicar lógica y facilitando la reutilización. Puedes crear tantos selectores como necesites, agrupando lógica de negocio y manteniendo los componentes limpios.

**Ejemplo:**

```ts
readonly totalStock = computed(() =>
	this._products().reduce((acc, p) => acc + p.stock, 0)
);

readonly expensiveProducts = computed(() =>
	this._products().filter(p => p.price > 1000)
);
```

### 2.9.4 Encapsulación y control de mutabilidad

Es recomendable exponer solo Signals de solo lectura (`asReadonly()`) y métodos claros para modificar el estado. Así evitas cambios accidentales y mantienes la lógica centralizada.

**Ejemplo de uso en un componente:**

```ts
@Component({
	selector: 'app-products',
	template: `
		<input (input)="store.setFilter($event.target.value)" placeholder="Buscar...">
		<ul>
			@for(p of store.visibleProducts(); track p.id) {
				<li>
					{{ p.name }} - {{ p.price }}€
					<button (click)="store.removeProduct(p.id)">Eliminar</button>
				</li>
			}
		</ul>
		<p>Total en stock: {{ store.totalStock() }}</p>
	`
})
export class ProductsComponent {
	store = inject(ProductsStore);
}
```

### 2.9.5 Integración con recursos asíncronos

Puedes combinar Signals con `resource` para cargar datos desde APIs y mantener el estado sincronizado:

```ts
import { signal, resource } from '@angular/core';

const reloadVersion = signal(0);
const productsResource = resource({
	params: () => reloadVersion(),
	loader: async ({ abortSignal }) => {
		const r = await fetch('https://dummyjson.com/products', { signal: abortSignal });
		if (!r.ok) throw new Error('Error');
		return r.json();
	}
});

function reloadProducts() {
	reloadVersion.update(v => v + 1);
}
```

Así puedes refrescar los datos fácilmente y mostrar estados de carga o error en la interfaz.

### 2.9.6 Composición y escalabilidad

Puedes crear varios Stores para distintas partes de la aplicación (usuarios, productos, carrito, etc.), y componerlos según las necesidades. Signals y `computed` facilitan la composición y el desacoplamiento.

**Ejemplo de composición:**

```ts
class AppStore {
	users = inject(UsersStore);
	products = inject(ProductsStore);
}
```


## 2.10 Anti-patrones comunes y cómo evitarlos en aplicaciones reales

Al trabajar con Signals en Angular 20, es fácil caer en ciertos anti-patrones que pueden dificultar el mantenimiento, la escalabilidad o el rendimiento de la aplicación. Reconocer estos errores y saber cómo evitarlos te ayudará a escribir código más limpio, eficiente y fácil de entender.

### 2.10.1 Anti-patrón: Mutar estructuras internas sin cambiar la referencia

**Problema:**
Modificar directamente arrays u objetos dentro de un Signal (por ejemplo, usando `.push()` o cambiando una propiedad) impide que Angular detecte el cambio y actualice la interfaz.

**Ejemplo incorrecto:**
```ts
const items = signal<string[]>([]);
items().push('nuevo'); // No se detecta el cambio
```

**Solución:**
Siempre crea una nueva referencia al modificar arrays u objetos.
```ts
items.update(list => [...list, 'nuevo']);
```

### 2.10.2 Anti-patrón: Exponer Signals mutables directamente

**Problema:**
Permitir que los componentes accedan y modifiquen Signals mutables directamente puede provocar cambios accidentales y dificultar el seguimiento del estado.

**Ejemplo incorrecto:**
```ts
export const count = signal(0); // Mutable y público
```

**Solución:**
Encapsula el Signal y expón solo la versión de solo lectura junto con métodos claros para modificar el estado.
```ts
private readonly _count = signal(0);
readonly count = this._count.asReadonly();
increment() { this._count.update(v => v + 1); }
```

### 2.10.3 Anti-patrón: Dividir el estado en demasiados Signals pequeños

**Problema:**
Crear Signals para cada propiedad individual puede fragmentar el estado y dificultar la comprensión global de la aplicación.

**Ejemplo incorrecto:**
```ts
const name = signal('');
const age = signal(0);
const active = signal(true);
```

**Solución:**
Agrupa propiedades relacionadas en un solo objeto o array dentro de un Signal.
```ts
const user = signal({ name: '', age: 0, active: true });
```

### 2.10.4 Anti-patrón: Usar Signals para variables temporales o locales

**Problema:**
No tiene sentido crear Signals para datos que solo existen dentro de una función o método y no forman parte del estado reactivo de la aplicación.

**Ejemplo incorrecto:**
```ts
function calcular() {
	const temp = signal(0); // No es necesario
	// ...
}
```

**Solución:**
Usa variables normales para datos temporales y Signals solo para estado que deba ser reactivo y compartido.

### 2.10.5 Anti-patrón: Mezclar lógica de negocio y de presentación en los métodos de los Stores

**Problema:**
Incluir lógica de presentación (como mostrar mensajes, manipular el DOM, etc.) dentro de los métodos que modifican el estado dificulta la reutilización y las pruebas.

**Ejemplo incorrecto:**
```ts
addProduct(product) {
	this._products.update(list => [...list, product]);
	alert('Producto añadido'); // Lógica de presentación
}
```

**Solución:**
Mantén los Stores enfocados en la lógica de negocio y deja la presentación para los componentes.

### 2.10.6 Anti-patrón: Efectos que calculan valores en vez de disparar acciones externas

**Problema:**
Usar `effect` para calcular valores derivados en vez de disparar acciones externas puede provocar recálculos innecesarios y confusión.

**Ejemplo incorrecto:**
```ts
effect(() => {
	this.total = this.items().length; // Esto debería ser un computed
});
```

**Solución:**
Usa `computed` para valores derivados y reserva `effect` para acciones externas (logs, persistencia, side effects).

### 2.10.7 Anti-patrón: No limpiar recursos en efectos

**Problema:**
Crear intervalos, listeners o suscripciones dentro de un `effect` sin usar `onCleanup` puede provocar fugas de memoria.

**Ejemplo incorrecto:**
```ts
effect(() => {
	setInterval(() => { ... }, 1000); // No se limpia
});
```

**Solución:**
Usa el callback `onCleanup` para limpiar recursos cuando el efecto se destruye o se vuelve a ejecutar.
```ts
effect(onCleanup => {
	const id = setInterval(() => { ... }, 1000);
	onCleanup(() => clearInterval(id));
});
```


