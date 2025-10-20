# 1. Repaso de las novedades y fundamentos avanzados incorporados en Angular 20

## 1.1 Standalone Components y el fin progresivo de NgModules

La transición hacia Standalone Components representa probablemente el cambio conceptual más profundo en la arquitectura de Angular desde su lanzamiento inicial. Aunque Angular 20 aún soporta NgModules por compatibilidad, el modelo recomendado para nuevas funcionalidades (y para la migración gradual de las existentes) se basa ya en componentes, directivas y pipes independientes, combinados con APIs funcionales para el arranque, el enrutamiento y la inyección de dependencias. Esta sección profundiza en el porqué, el cómo y el cuándo de esta evolución, ofreciendo ejemplos concretos y estrategias de migración pensadas para equipos que mantienen aplicaciones enterprise maduras.

### 1.1.1 ¿Por qué abandonar progresivamente NgModules?

Durante años, NgModule fue la unidad de organización y compilación: agrupaba declaraciones, exportaciones, imports y providers. Sin embargo, esto introducía complejidad accidental: múltiples lugares para registrar artefactos, patrones artificiales (p.ej. `SharedModule`, `CoreModule`), cascadas de imports difíciles de rastrear y una curva cognitiva elevada para nuevos desarrolladores. Standalone persigue tres objetivos:

1. Simplicidad mental: cada componente declara sus dependencias explícitamente en un único lugar (su metadato `imports`).
2. Composición granular: se facilita el lazy loading a nivel de componente o grupo de rutas sin crear módulos intermedios.
3. Mejor tree-shaking y tiempos de compilación: al eliminar las capas de indirección introducidas por los metadatos de los módulos, el compilador puede interpretar y optimizar el código de manera más eficiente, reduciendo el tamaño final del bundle y acelerando el proceso de build.

El resultado es un código más declarativo donde lo que ves es lo que hay (WYSIWYG): no hay magia oculta en un NgModule distante.

### 1.1.2 Anatomía de un (ahora implícito) Standalone Component

En Angular 20 los componentes son standalone por defecto, por lo que el metadato `standalone: true` es redundante (aunque todavía aceptado para compatibilidad / claridad durante una migración). El componente declara explícitamente sus dependencias en el campo `imports`, que permanece porque sigue siendo necesario expresar qué directivas, componentes y pipes externos se usan. También puedes seguir importando módulos legacy imprescindibles durante una migración gradual.

```ts
import { Component, signal } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
	selector: 'app-counter',
	// standalone: true (Opcional y redundante en Angular 20+)
	imports: [CommonModule], // Directivas/pipes estructurales: *ngIf, *ngFor (o nuevo control flow)
	template: `
		<section class="counter">
			<h2>Counter (standalone)</h2>
			<p>Valor: {{ count() }}</p>
			<button (click)="inc()">Incrementar</button>
			<button (click)="reset()" [disabled]="count() === 0">Reiniciar</button>
		</section>
	`,
	styles: [`.counter { display: inline-block; padding: 1rem; border: 1px solid #ccc; border-radius: 8px; }`]
})
export class CounterComponent {
	private readonly _count = signal(0);
	count = this._count.asReadonly();
	inc() { this._count.update(v => v + 1); }
	reset() { this._count.set(0); }
}
```

Observa que no existe ya un `CounterModule`. El propio componente es exportable e importable por otros directamente y no hemos necesitado declarar explícitamente `standalone: true`.

### 1.1.3 Arranque de la aplicación sin NgModule raíz

La función `bootstrapApplication()` sustituye a `platformBrowserDynamic().bootstrapModule(AppModule)`. Se invoca con el componente raíz y un objeto de opciones que agrega providers funcionales.

```ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
	providers: [
		provideRouter(routes),
		provideHttpClient(
			withInterceptors([
				(req, next) => {
					// Ejemplo mínimo interceptor funcional
					console.debug('[HTTP]', req.method, req.url);
					return next(req);
				}
			])
		)
	]
}).catch(err => console.error(err));
```

Este enfoque elimina la necesidad de un `AppModule`. La configuración de router, HTTP y otros servicios globales se centraliza de forma explícita y componible.

### 1.1.4 Rutas y lazy loading granular con Standalone

El enrutamiento moderno se apoya en `provideRouter()` y permite cargar componentes standalone directamente mediante `loadComponent`, o conjuntos de rutas mediante `loadChildren` que devuelven un array de rutas (sin NgModule). 

```ts
// app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component'; // standalone

export const routes: Routes = [
	{ path: '', component: HomeComponent },
	{ 
		path: 'admin', 
		loadComponent: () => import('./admin/admin.component')
			.then(m => m.AdminComponent) 
	},
	{
		path: 'products',
		loadChildren: () => import('./products/product.routes')
			.then(m => m.PRODUCT_ROUTES)
	}
];
```

Un archivo de rutas de feature puede quedar así, sin módulo:

```ts
// products/product.routes.ts
import { Routes } from '@angular/router';
export const PRODUCT_ROUTES: Routes = [
	{ path: '', loadComponent: () => import('./list/product-list.component').then(c => c.ProductListComponent) },
	{ path: ':id', loadComponent: () => import('./detail/product-detail.component').then(c => c.ProductDetailComponent) }
];
```

Esto reduce la fricción de crear un `ProductsModule` únicamente para declarar rutas y exportar componentes.

### 1.1.5 Migración incremental desde NgModules

La migración no exige un big bang. Puedes convertir módulos a un conjunto de componentes standalone de forma escalonada.

Escenario típico: un `UserModule` declaraba `UserListComponent`, `UserDetailComponent` y exportaba el primero. Pasos:

1. Convertir cada componente a standalone: (en Angular <18 agregando `standalone: true`; en Angular 18+ simplemente eliminándolo del NgModule y añadiendo/manteniendo el array `imports` con sus dependencias: CommonModule, FormsModule, etc.).
2. Eliminar las declaraciones y exportaciones correspondientes del NgModule (temporalmente seguirá existiendo para otros componentes no migrados).
3. Sustituir en las rutas cualquier referencia a `loadChildren: () => import('./user/user.module').then(m => m.UserModule)` por rutas con `loadComponent` o un archivo de rutas de feature si ya todos son standalone.
4. Cuando el NgModule queda vacío (sin `declarations` ni `providers` únicos), eliminarlo.

Ejemplo de antes (simplificado):

```ts
// user/user.module.ts (antes)
@NgModule({
	declarations: [UserListComponent, UserDetailComponent],
	imports: [CommonModule, RouterModule.forChild(USER_ROUTES)],
	exports: [UserListComponent]
})
export class UserModule {}
```

Después de la migración de `UserListComponent`:

```ts
// user/user-list.component.ts (después)
@Component({
	selector: 'app-user-list',
	// standalone implícito en Angular 20+
	imports: [CommonModule, RouterLink],
	template: `
		<h2>Usuarios</h2>
		<ul>
			@for (u of users(); track u.id) {
				<li><a [routerLink]="['/users', u.id]">{{ u.name }}</a></li>
			}
		</ul>
	`
})
export class UserListComponent { /* ... */ }
```

Archivo de rutas resultante:

```ts
// user/user.routes.ts
import { Routes } from '@angular/router';
export const USER_ROUTES: Routes = [
	{ path: '', loadComponent: () => import('./user-list.component').then(c => c.UserListComponent) },
	{ path: ':id', loadComponent: () => import('./user-detail.component').then(c => c.UserDetailComponent) }
];
```

### 1.1.6 Providers: de NgModule a APIs funcionales

Antes, muchos servicios globales se registraban en `providers` de un módulo raíz o en módulos de feature. Ahora se concentran en la llamada a `bootstrapApplication()` o se definen como providers específicos en componentes standalone cuando su ámbito debe ser local.

```ts
// api.provider.ts
import { ENVIRONMENT_INITIALIZER, inject, Provider } from '@angular/core';
import { HttpClient } from '@angular/common/http';

export const API_BASE_URL = 'https://api.ejemplo.dev';

class ApiClient {
	constructor(private http: HttpClient) {}
	getUsers() { return this.http.get(`${API_BASE_URL}/users`); }
}

export function provideApi(): Provider[] {
	return [
		ApiClient,
		{
			provide: ENVIRONMENT_INITIALIZER,
			multi: true,
			useValue: () => {
				const client = inject(ApiClient);
				console.log('ApiClient inicializado');
			}
		}
	];
}
```

Integración en el arranque:

```ts
bootstrapApplication(AppComponent, {
	providers: [
		provideRouter(routes),
		provideHttpClient(),
		...provideApi()
	]
});
```

Para un provider de ámbito de feature, puede exportarse un array y reusarse en las rutas lazy:

```ts
export const USER_PROVIDERS: Provider[] = [UserService, CacheService];

export const USER_ROUTES: Routes = [
	{ path: '', loadComponent: () => import('./user-list.component').then(c => c.UserListComponent), providers: USER_PROVIDERS },
	{ path: ':id', loadComponent: () => import('./user-detail.component').then(c => c.UserDetailComponent), providers: USER_PROVIDERS }
];
```

El campo `providers` a nivel de ruta crea un nuevo injector jerárquico para esa rama, reemplazando patrones previos donde se necesitaban módulos intermedios.

### 1.1.7 Patrones de organización en ausencia de NgModules

Sin módulos, la organización física cobra más relevancia. Recomendaciones:

- Agrupa por feature vertical (`/users`, `/products`) con subcarpetas para componentes, servicios y modelos. 
- Usa archivos `index.ts` (barrels) únicamente cuando mejoren la DX; evita inflar imports opacos.
- Define `*.routes.ts` para cada feature con su array de rutas y providers de feature.
- Centraliza providers globales en `main.ts` y evita `importProvidersFrom` salvo para integrar librerías que aún no exponen APIs funcionales.

Ejemplo de estructura de una feature:

```
users/
	user-list.component.ts
	user-detail.component.ts
	user.service.ts
	user.routes.ts
	index.ts (opcional)
```

### 1.1.8 Anti-patrones y trampas comunes

1. Sobrecargar `imports` de un componente con docenas de dependencias que solo usa una plantilla hija. Mitiga extrayendo componentes intermedios o directivas dedicadas.
2. Re-crear un pseudo `SharedModule` como archivo que exporta arrays gigantes de imports y providers. Úsalo solo si evitas duplicación real; de lo contrario, importa puntualmente lo necesario (el compilador optimizará la repetición).
3. Dejar providers críticos en componentes hijos sin documentar alcance, generando instancias múltiples inesperadas (p.ej. servicios de caché). Define explícitamente su nivel (root vs feature vs componente).
4. Usar `importProvidersFrom(HttpClientModule)` en lugar de `provideHttpClient()` por inercia. Prefiere las APIs funcionales que habilitan opciones modernas (`withFetch`, interceptores funcionales, etc.).
5. Migrar todo de golpe sin métricas. Migra feature a feature, midiendo sizes de bundles y tiempos de compilación para justificar el esfuerzo incremental.

### 1.1.9 Testing de componentes standalone

Las pruebas se simplifican: `TestBed` puede importar directamente el componente bajo prueba sin crear un módulo de test.

```ts
import { TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';

describe('CounterComponent', () => {
	it('incrementa el contador', () => {
		const fixture = TestBed.createComponent(CounterComponent);
		fixture.detectChanges();
		const button: HTMLButtonElement = fixture.nativeElement.querySelector('button');
		button.click();
		fixture.detectChanges();
		expect(fixture.nativeElement.textContent).toContain('Valor: 1');
	});
});
```

Para pruebas de integración con routing, se puede usar `provideRouter` directamente dentro de `TestBed.configureTestingModule({ providers: [provideRouter(routes)] })` sin módulos auxiliares.

### 1.1.10 Métricas y beneficios observables

Equipos que han migrado parcialmente reportan:

- Reducción de archivos “ceremoniales” (~10–20% menos en features simples).
- Menor tiempo cognitivo para nuevos desarrolladores: entienden dependencias revisando solo el componente.
- Lazy loading más fino, reduciendo Time To Interactive en rutas secundarias.
- Mejor separación de dominios gracias a providers por ruta y jerarquías más pequeñas.

Es aconsejable instrumentar el bundle (con `source-map-explorer` o `webpack-bundle-analyzer`) antes y después de la migración para cuantificar.

### 1.1.11 Estrategia recomendada de migración en proyectos enterprise

1. Auditoría: inventario de módulos y dependencias transversales (servicios singleton, directivas compartidas, pipes). 
2. Priorización: selecciona features con fronteras claras y pocos providers personalizados.
3. Conversión piloto: migra 1–2 features para documentar un playbook interno (guía + checklist + pasos de testing/regresión).
4. Automatización: crea scripts (schematics o codemods) para marcar `standalone: true`, mover imports y generar archivos de rutas.
5. Observabilidad: activa métricas de build y performance para comparar.
6. Comunicación: socializa resultados y refina patrones antes de escalar.

### 1.1.12 ¿Cuándo mantener NgModules (temporalmente)?

Aunque su finalidad es desaparecer, conserva un NgModule cuando:

- Una librería third-party aún no expone APIs funcionales y su refactor es costoso ahora.
- Necesitas `forRoot()/forChild()` hasta que esa librería adopte un provider funcional equivalente.
- Un área legacy altamente estable no aporta ROI inmediato al migrarse (postergar hasta un refactor mayor).

Incluso en estos casos, puedes encapsular el uso de `importProvidersFrom()` en un provider funcional propio para aislar la dependencia.

## 1.2. Nueva sintaxis de control flow (`@if`, `@for`, `@switch`)

Esta nueva sintaxis de control de flujo introduce bloques explícitos (`@if`, `@for`, `@switch`) que reemplazan progresivamente a las directivas estructurales clásicas (`*ngIf`, `*ngFor`, `*ngSwitch`). El objetivo es simplificar la lectura, hacer más predecible la interacción con Signals y habilitar optimizaciones de compilación al eliminar la microsintaxis basada en el asterisco. Gracias a un formato más declarativo (uso de llaves y scoping claro), desaparece la necesidad de múltiples `<ng-container>` anidados y se facilita expresar ramas complejas de forma lineal (`@if / @else if / @else`). Del mismo modo, `@for` incorpora un bloque `@empty` que liga semánticamente la representación del “estado vacío” con la iteración, y exige un tracking explícito que favorece el rendimiento en listas dinámicas. Finalmente, `@switch` alinea su estilo con los otros bloques, ofreciendo una sintaxis consistente para flujos condicionales múltiples. 

En secciones posteriores profundizaremos en motivaciones internas, patrones de migración, integración pormenorizada con Signals, rendimiento, testing, SSR/Hydration y anti‑patrones. Por ahora, basta entender que estos bloques constituyen la base expresiva moderna de templates en Angular 20 y que su adopción incremental mejora claridad, mantenibilidad y performance futura del proyecto.

## 1.3 Reactividad moderna con Signals (`signal`, `computed`, `effect`)

La introducción de Signals consolida un modelo de reactividad explícita y predecible en Angular 20, reemplazando progresivamente patrones basados en `Observable` para estado local, derivaciones simples y coordinación de vistas. Mientras RxJS sigue siendo imprescindible para flujos asíncronos complejos, streaming y composición avanzada, Signals cubren el espacio de: (a) estado inmutable (o mutaciones controladas) de componente, (b) derivaciones puras memoizadas y (c) efectos controlados que reaccionan a cambios sin disparar cascadas difíciles de seguir.

En esencia, un `signal<T>` es un contenedor de valor con lectura tipo función (invocación) y escritura mediante métodos (`set`, `update`, mutación estructural), que notifica de forma síncrona a los consumidores dependientes. Sobre esa base:

- `signal`: fuente primaria de verdad local (o compartida si se expone a otros componentes / servicios) con API mínima y predecible.
- `computed`: produce un valor derivado, recalculándose solo cuando cambia alguno de los Signals que lee (con caching interno y eliminación de recomputaciones redundantes).
- `effect`: ejecuta lógica con efectos secundarios (side effects controlados) en respuesta a cambios de los Signals leídos durante su ejecución inicial; Angular gestiona su re-ejecución y limpieza.

Ejemplo:

```ts
import { Component, signal, computed, effect } from '@angular/core';

@Component({
	selector: 'app-cart-summary',
	imports: [],
	template: `
		<h3>Carrito ({{ totalItems() }} items)</h3>
		<p>Total: {{ totalPrice() | currency:'EUR' }}</p>
		<button (click)="addRandom()">Añadir aleatorio</button>
	`
})
export class CartSummaryComponent {
	private readonly items = signal<{ name: string; price: number }[]>([]);

	// Derivación memoizada: recalcula solo si items() cambia por referencia o contenido mutado seguido de set/update.
	readonly totalItems = computed(() => this.items().length);
	readonly totalPrice = computed(() => this.items().reduce((acc, it) => acc + it.price, 0));

	constructor() {
		// Efecto: logging y potencial integración con analítica.
		effect(() => {
			console.debug('[Cart] cambio -> items:', this.totalItems(), 'importe:', this.totalPrice());
		});
	}

	addRandom() {
		const product = { name: 'Item ' + (this.totalItems() + 1), price: +(Math.random() * 20 + 5).toFixed(2) };
		this.items.update(list => [...list, product]);
	}
}
```

Características diferenciales clave frente a aproximaciones previas:

1. Pull-based determinista: una lectura (`count()`) no dispara efectos colaterales; solo obtiene el valor actual.
2. Eliminación de suscripciones manuales: no hay `subscribe()` para estado local, evitando fugas de memoria y boilerplate de `unsubscribe`.
3. Derivaciones acotadas: `computed` no se re-ejecuta si los valores fuente no han variado (shallow equality de dependencias interpretada a nivel de grafo de Signals).
4. Trazabilidad de dependencias: el grafo reactivo se construye de forma automática a partir de las lecturas realizadas durante la ejecución inicial.
5. Integración futura: sienta la base para optimizaciones zoneless y granularidad fina de render al saber exactamente qué parte del árbol depende de qué valores.

Relación con RxJS: en esta fase, RxJS se reserva para streams asincrónicos (websocket, intervalos, combinaciones complejas, backpressure), mientras Signals simplifican el estado in-process. La interoperabilidad (`toSignal`, `toObservable`) permite mover datos entre ambos mundos gradualmente sin bloqueos arquitectónicos.

## 1.4 Integración de `linkedSignal`, `toSignal` y `resource`

Los mecanismos de interoperabilidad y orquestación amplían el modelo base de Signals para cubrir casos donde el estado no es puramente local o requiere sincronización con fuentes externas. Tres APIs destacan en Angular 20 para este propósito:

- `toSignal(observable, options)`: puente unidireccional que convierte un `Observable` en un Signal reactivo, manteniendo la suscripción internamente y gestionando el valor inicial, estrategias de error y finalización. Facilita consumir streams existentes (WebSocket, intervalos, subjects legacy) sin reescribir la capa de datos inmediatamente.
- `linkedSignal(factory)`: crea un Signal cuyo valor depende de otros Signals u objetos y que puede reaccionar a su ciclo de vida, permitiendo lógica de enlace avanzada (por ejemplo, resetear derivaciones cuando cambia un contexto superior o gestionar recursos asociados al valor actual).
- `resource(...)` / `httpResource(...)`: abstracción declarativa para cargar, cachear y revalidar datos asíncronos (generalmente HTTP) con estados integrados (loading, error, success) expuestos como Signals derivados. Simplifica patrones repetidos de fetch + estado + cancelación.

Motivación: en aplicaciones enterprise existen capas ya consolidadas sobre RxJS (servicios con operators complejos). Una migración directa a Signals puros sería costosa; `toSignal` habilita una transición progresiva: se encapsula el observable y se expone una interfaz de lectura sin suscripción manual. Más adelante, partes críticas pueden refactorizarse a Signals nativos si aporta simplificación o performance.

Ejemplo introductorio de `toSignal`:

```ts
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { PricesService } from './prices.service';

@Component({
	selector: 'app-live-prices',
	template: `
		<h3>Precio actual BTC/EUR: {{ price() ?? '—' }}</h3>
	`
})
export class LivePricesComponent {
	private svc = inject(PricesService);
	price = toSignal(this.svc.btcPrice$(), { initialValue: null });
}
```

`linkedSignal` (ejemplo conceptual) permite asociar cleanup o recalcular bajo condiciones de entorno:

```ts
import { linkedSignal, signal } from '@angular/core';

const timezone = signal('UTC');
const now = linkedSignal(() => {
	const tz = timezone();
	const handle = setInterval(() => current.set(new Date()), 1000);
	const current = signal(new Date());
	return {
		value: current(),
		onCleanup: () => clearInterval(handle)
	};
});
```

Aunque el patrón exacto puede variar (la API puede evolucionar), la idea central es vincular el ciclo de vida del valor derivado a dependencias reactivas y disponer de un hook limpio para liberar recursos.

`resource` aporta un enfoque declarativo a la obtención de datos con gestión de concurrencia y revalidación opcional, exponiendo Signals como `value()`, `status()`, `error()` y triggers para refrescar. Esto reduce boilerplate clásico: estados booleanos, manejo manual de cancelaciones o condiciones de carrera al disparar peticiones rápidas sucesivas.

Cuándo usar cada uno (visión preliminar):

1. `toSignal`: reutilizar Observables existentes, integrar librerías externas basadas en RxJS, mediciones de performance donde mantener un pipeline Rx es conveniente.
2. `linkedSignal`: modelar valores que necesitan acoplar un ciclo de vida (timers, listeners, websockets livianos) a la reactividad de otros Signals sin exponer un efecto separado difícil de sincronizar.
3. `resource`: encapsular fetch + estados + caching + reintentos leves sin reinventar un mini state machine en cada servicio.

## 1.5 Zoneless Angular: detección de cambios sin Zone.js

La adopción del modo “zoneless” elimina la dependencia histórica de Zone.js como mecanismo genérico para interceptar tareas asíncronas (macro/microtasks) y disparar ciclos globales de detección de cambios. En Angular 20 esto deja de ser un experimento marginal y se consolida como una opción estratégica para reducir trabajo redundante, incrementar el control explícito y disminuir el coste cognitivo al depurar cascadas de renders. El motor de reactividad moderno (Signals + control flow de bloques) permite saber con precisión qué componentes deben reevaluarse, haciendo innecesario un hook omnipresente como Zone.js en muchos escenarios.

Motivaciones principales:

1. Performance: se evita ejecutar detección de cambios global ante cualquier setTimeout, evento DOM o promesa resuelta que no afecte al estado observado por la vista.
2. Predictibilidad: las actualizaciones se producen cuando cambia un Signal (pull control) o se invoca explícitamente un trigger, no por “mágica” intercepción del runtime.
3. Menor complejidad: desaparecen trampas asociadas a librerías externas que parchean el event loop o introducen zonas hijas.
4. Debug simplificado: los flujos de actualización quedan restringidos a efectos y mutaciones de Signals, trazables con tooling moderno.

Concepto clave: sin Zone.js no existe la detección de cambios automática tradicional. La responsabilidad pasa a la reactividad declarativa (Signals) y, para APIs no instrumentadas (callbacks de librerías externas, listeners manuales), al uso explícito de mecanismos que informen a Angular del cambio.

Activación típica (visión preliminar):

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { provideZonelessChangeDetection } from '@angular/core';

bootstrapApplication(AppComponent, {
	providers: [
		provideZonelessChangeDetection(),
		// ...otros providers
	]
});
```

En este modo, mutar un `signal` dispara la propagación reactiva y render selectivo. Si una librería externa actualiza estado fuera del grafo de Signals, necesitarás enlazarlo vía `effect`, convertirlo con `toSignal` o usar un adaptador que notifique la mutación.

Ejemplo conceptual comparativo:

```ts
// Con Zone.js (legacy)
setTimeout(() => {
	this.counter++;// disparará la detección de cambios global
}, 0);

// Zoneless + Signals
setTimeout(() => {
	this.counter.update(v => v + 1); // Solo los consumidores de counter() se actualizan
}, 0);
```

Beneficios esperados: menor trabajo en rutas complejas con muchos componentes no dependientes del cambio producido; reducción de renders fantasma en los registros de rendimiento (“perf traces”, es decir, trazas de profiling que muestran cuándo y por qué se renderiza cada parte de la aplicación); acoplamiento más claro entre la mutación y el impacto visual.

Implicaciones de migración inicial:

- Revisar puntos donde dependías implícitamente de Zone.js (asignaciones directas en callbacks de librerías) y envolverlos en Signals / efectos.
- Sustituir patrones `ChangeDetectorRef.detectChanges()` ad hoc por modelado de estado reactivo; reservar llamadas manuales únicamente para escenarios muy específicos (portales, integraciones host externas, micro-frontends).
- Instrumentar métricas antes/después (FPS, commits DOM, scripting time) para validar beneficios reales y ajustar estrategia.

Limitaciones y cautelas tempranas:

1. Algunas librerías de terceros no reactivas pueden requerir adaptadores; evaluar coste antes de activar zoneless globalmente.
2. Efectos mal diseñados (mutaciones dentro de efectos sin control) pueden generar loops; la disciplina es más visible sin el “ruido” de triggers globales.
3. Mezclar Change Detection tradicional y Signals de forma híbrida debe ser transitorio y acotado.

## 1.6 SSR con Hydration incremental estable

Angular 20 consolida una estrategia de render universal orientada a rendimiento percibido y coste inicial reducido mediante Hydration incremental estable. En lugar de re-renderizar agresivamente toda la aplicación en el cliente tras recibir HTML del servidor, el motor identifica islas de la interfaz que pueden activarse bajo demanda (primer scroll, intersección en viewport, interacción del usuario o disponibilidad de datos asíncronos), manteniendo operativo el contenido estático mientras se completa la hidratación selectiva. Esto reduce Time To Interactive (TTI), mejora Core Web Vitals (particularmente LCP y INP) y evita trabajo JavaScript innecesario para zonas no críticas.

Pilares de la aproximación:

1. HTML servible y semántico listo para indexación y accesibilidad desde el primer byte.
2. Reconciliación DOM no destructiva: se anexan listeners y se instancian componentes sin volver a pintar nodos idénticos que ya están en el markup inicial.
3. Estrategias de defer / lazy en combinación con control flow moderno para evitar cargar rutas o componentes pesados prematuramente.
4. Coordinación con Signals: la hidratación respeta el grafo reactivo, activando efectos solo cuando la isla asociada entra en contexto.

Activación básica (visión conceptual):

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
	providers: [
		provideClientHydration(/* opciones: logging, errorHandling, incremental */)
	]
});
```

Patrón de islas / deferred views (ejemplo simplificado):

```html
@if (featureFlag('analytics')) {
	<defer on="idle">
		<app-analytics-panel />
	</defer>
}

<defer on="viewport">
	<app-related-products />
</defer>
```

Donde `<defer>` (o sintaxis equivalente ofrecida por utilidades complementarias) retrasa la hidratación de esos nodos hasta que se cumple la condición (`idle`, `viewport`, interacción, etc.), liberando recursos iniciales para contenido prioritario.

Beneficios clave frente al SSR clásico:

- Menos JS ejecutado durante la ventana crítica: la lógica no esencial se pospone.
- Reducción del riesgo de mismatches al limitar el número de nodos rehidratados simultáneamente.
- Mejor escalabilidad en páginas densas (catálogos, dashboards) al fragmentar la hidratación.

## 1.7 Angular CLI: recarga en caliente y diagnósticos mejorados

La evolución del Angular CLI en la versión 20 acompaña los cambios de arquitectura (standalone, Signals, zoneless) con mejoras en el ciclo de desarrollo: Hot Module Replacement (HMR) más estable y granular, diagnósticos incrementales enriquecidos y tooling integrado que reduce dependencias de utilidades externas. El objetivo es acortar el feedback loop y elevar la calidad de la información que recibe el equipo mientras desarrolla y refactoriza.

Elementos clave de la experiencia renovada:

1. HMR orientado a estado local: al modificar plantillas o estilos, el CLI aplica parches manteniendo Signals y estado de componentes siempre que no se cambie su contrato público (inputs/outputs). Esto permite iterar sobre UI sin re-navegar ni re-ejecutar secuencias de inicialización costosas.
2. Diagnósticos semánticos avanzados: se incorporan verificaciones específicas de la nueva sintaxis de control flow (`@if`, `@for`) y uso de APIs funcionales (`provideRouter`, `provideHttpClient`) ofreciendo sugerencias concretas (quick fixes) antes de que el error se materialice en runtime.
3. Incremental build más fino: la granularidad de recompilación se alinea con la ausencia de NgModules; cambios en un componente no invalidan de forma amplia rutas no relacionadas.
4. Reportes de perf integrados: al ejecutar builds de producción se pueden habilitar métricas de tamaño por chunk, coste de inicialización y hints sobre oportunidades de lazy / defer.
5. Integración mejorada con ESLint y formateo: configuración base generada con reglas que fomentan adopción de Signals y evitan patrones legacy (`importProvidersFrom` redundante, tracking de `@for` ausente, etc.).

Estado de HMR: a partir de Angular 20 viene activado por defecto en `ng serve`. Solo necesitas intervenir si deseas desactivarlo (por depuración de edge cases o reproducibilidad de sesiones completas) usando:

```bash
ng serve --no-hmr
```

El CLI detecta cambios, aplica parches y preserva el grafo de Signals sin recargar la página. En contextos más complejos (micro frontends o integraciones personalizadas), puedes ajustar comportamiento inspeccionando `import.meta.hot` (cuando esté disponible) para limpieza manual de recursos o invalidación selectiva de estado.

Diagnósticos ejemplificativos (conceptuales):

- Advertencia: “Bloque @for sin cláusula track explícita” → sugiere añadir `track item.id` o una función para evitar re-renders globales.
- Sugerencia: “Uso de *ngIf detectado junto a @if en el mismo archivo” → propone migrar para uniformidad y evitar estilos mixtos.
- Nota de compatibilidad: “importProvidersFrom(HttpClientModule) usado: considerar provideHttpClient() para interceptores funcionales”.

Flujo típico optimizado:

1. Desarrollador edita plantilla -> HMR aplica parche -> Estado del carrito (Signal) se conserva -> Vista actualizada instantáneamente.
2. CLI muestra diagnóstico inline sobre un `@for` sin tracking -> Se corrige sin esperar a pruebas end-to-end.
3. Antes de commit, un análisis rápido produce reporte de tamaños y sugiere diferir un componente pesado de dashboard.

Relación con la nueva arquitectura: la precisión de invalidación y los diagnósticos se apalancan en metadatos más locales (componentes standalone y providers funcionales) reduciendo falsos positivos. Esto eleva la confianza en refactors incrementales.

Limitaciones y expectativas iniciales:

- HMR no garantiza preservación cuando cambian estructuras internas profundas del componente (nuevos Signals esenciales, eliminación de campos críticos). En esos casos, se reinicializa esa rama.
- Algunas reglas de diagnóstico pueden requerir ajuste fino para proyectos con convenciones muy personalizadas; conviene centralizar overrides para mantener consistencia.
- Métricas de perf integradas no sustituyen un análisis profundo con herramientas como Lighthouse o WebPageTest, pero sirven como filtro preliminar.

## 1.8 DevTools integrados y profiling avanzado en Angular 20

El ecosistema de herramientas oficiales se alinea con la nueva capa reactiva y el enfoque zoneless para ofrecer visibilidad precisa sobre qué cambia, cuándo y por qué. La versión 20 refuerza Angular DevTools (extensión del navegador + APIs subyacentes) y la interoperabilidad con Chrome Performance Panel, permitiendo aislar cuellos de botella sin depender de heurísticas difusas. La meta: pasar de “sospechas” sobre cascadas de render a diagnósticos accionables apoyados en el grafo de Signals, el flujo de hydration incremental y los límites de componentes standalone.

Pilares de la experiencia modernizada:

1. Timeline reactivo: visualización de mutaciones de Signals, recomputaciones de `computed` y ejecuciones de `effect` correlacionadas con frames de render.
2. Árbol de componentes optimizado: cada nodo expone dependencias reactivas declaradas (Signals leídos) y métricas de actualización (conteo de renders, tiempo acumulado MS).
3. Integración Hydration: marcadores que identifican islas hidratadas y su latencia hasta estar interactivas.
4. Insight zoneless: cuando se activa el modo sin Zone.js, la herramienta etiqueta triggers explícitos (mutaciones de Signals, navegación, input del usuario) evitando ruido de macro/microtasks irrelevantes.
5. Export y reproducibilidad: posibilidad (conceptual) de exportar un snapshot de eventos para análisis offline o compartir con el equipo en revisiones de performance.

Caso de uso rápido (flujo típico de diagnóstico):

1. Se detecta jank (“saltos” de UI) al expandir un panel de filtros.
2. Abrir DevTools Angular -> pestaña Performance -> grabar interacción.
3. El timeline muestra múltiples recomputaciones de un `computed` de agregación que depende de una lista grande mutada in-place.
4. Acción: refactorizar a una actualización inmutable + granularidad en `@for` con tracking estable para reducir diffs.
5. Nuevo perfil confirma reducción de renders y menor scripting time.

Interacción con Chrome Performance Panel: Angular anota (trace events) momentos significativos (inicio/fin de hydration de isla, propagación de batch de Signals, montaje de un lazy route). Esto facilita correlacionar eventos de framework con layout thrashing o picos de scripting sin tener que inferir manualmente.

Buenas prácticas iniciales al usar las herramientas:

- Perf budgets: define umbrales (ms por render de componente crítico) y monitoriza regresiones.
- Pre-profiling checklist: reproduce en modo producción (`ng build --configuration=production` + servir estático) para eliminar ruido de dev.
- Aísla variables: cambia una sola cosa (p.ej. estrategia de tracking en `@for`) antes de un nuevo perfil para atribuir mejoras correctamente.
- Usa markers de usuario (User Timing API) para delimitar fases de interacción específicas si el escenario es complejo.

## 1.9 Deprecaciones clave desde Angular 17 y su impacto en proyectos actuales

La evolución acelerada desde Angular 17 hasta la versión 20 ha venido acompañada de una serie de deprecaciones y cambios de estado (deprecated → removido / legacy) alineados con la adopción de Standalone Components, Signals y el nuevo control flow. Entender su alcance permite planificar migraciones sin sobresaltos, evitando refactors reactivos “a última hora”. Esta introducción resume los ejes principales: APIs marcadas para retirada, sustituciones recomendadas y riesgos de posponer la adaptación.

### 1.9.1 Resumen de áreas afectadas

1. NgModules (estatus: legacy progresivo): uso opcional, plan de desaparición a medio plazo salvo en integraciones aún no migradas de terceros.
2. Directivas estructurales clásicas `*ngIf` / `*ngFor` / `*ngSwitch`: coexistencia temporal; preferencia clara por bloques `@if` / `@for` / `@switch`.
3. Guards, resolvers e interceptors basados en clases: reemplazados por funciones (`canActivate`, `canMatch`, interceptores funcionales).
4. APIs de `HttpClientModule` vía `imports` + `providers`: desplazadas por `provideHttpClient()` y extensiones (`withInterceptors`, `withFetch`).
5. Parches automáticos de Zone.js: orientados a ser opcionales con el modo zoneless; ciertos flujos dependerán de adaptación explícita.
6. Formularios no tipados (uso extensivo de `any` en `AbstractControl`): paulatinamente sustituidos por Typed Forms y nuevas propuestas basadas en Signals.
7. Imagen optimizada manual (`<img>` + atributos ad-hoc): recomendación de migrar a `NgOptimizedImage`.

### 1.9.2 Tabla conceptual de sustitución (descriptiva)

- `NgModule` de feature simple → Componente(s) standalone + archivo `feature.routes.ts` + providers funcionales locales.
- `*ngIf` anidado complejo → Bloques `@if / @else if / @else` lineales.
- `*ngFor` con `trackBy` → `@for (...; track ...) {}` + `@empty` asociado.
- Clase guard con `implements CanActivate` → Función `canActivate: () => boolean | UrlTree | Observable | Promise` usando `inject()`.
- Interceptor clase → Función `(req, next) => next(req.clone(...))` registrada en `provideHttpClient(withInterceptors([...]))`.
- `HttpClientModule` en `imports` → `provideHttpClient()` (y variantes con opciones).
- Formularios con `FormControl<any>` → Typed Forms (`FormControl<string>`), luego transición a API de formularios con Signals donde aplique.
- Optimización manual de imágenes → `NgOptimizedImage` (`ngSrc`, prioridad, densidades, lazy integrado).

### 1.9.3 Impacto de no migrar a tiempo

Posponer migraciones incrementa “intereses de deuda”:

- Superficie de refactor mayor y más difícil de testear en bloque cuando el soporte legacy se retira.
- Doble mental model en el equipo (legacy vs moderno) ralentizando onboarding.
- Oportunidades perdidas de performance (render granular, hydration incremental eficiente) al mantener APIs antiguas.
- Mayor riesgo en upgrades futuros: combinaciones de patrones legacy pueden ocultar edge cases.

### 1.9.4 Estrategia de priorización sugerida

1. Control flow: migrar vistas críticas de alto churn (plantillas que se tocan a menudo) para reducir coste de cambios futuros.
2. Providers / DI: sustituir gradualmente `forRoot/forChild` y `HttpClientModule` por providers funcionales; documentar el patrón en una guía interna.
3. Routing: convertir guards/resolvers a funciones; establecer linter que marque nuevas clases guard como error.
4. Estado local: adoptar Signals para componentes nuevos y migrar features aisladas medibles (ej. panel de carrito, sidebar) antes de abordar secciones core.
5. Formularios: introducir Typed Forms en formularios nuevos; refactor de legacy en fases (validaciones críticas primero, luego UI secundaria).
6. Imágenes: migrar recursos mayores (hero, galerías) a `NgOptimizedImage` para impactos de LCP visibles.
