# 4. Directivas y control flow moderno

## 4.1 Introducción al nuevo control flow con `@if`, `@for` y `@switch`

Uno de los cambios más significativos que trae **Angular 20** es la introducción del **nuevo sistema de control de flujo en plantillas**, que reemplaza y moderniza a las clásicas directivas estructurales (`*ngIf`, `*ngFor`, `*ngSwitch`).  

Este nuevo enfoque no solo simplifica la sintaxis, sino que también mejora la **legibilidad**, la **seguridad de tipos** y el **rendimiento**. Ahora, en lugar de usar directivas con asterisco, Angular nos ofrece **bloques de control nativos** con una sintaxis más clara y cercana a otros lenguajes de programación.

### 4.1.1. ¿Por qué un nuevo control de flujo?

Hasta Angular 19, el control de flujo en plantillas se basaba en directivas estructurales con un asterisco (`*`). Aunque funcionaban bien, tenían algunas limitaciones:

- La sintaxis podía resultar confusa para principiantes (`*ngIf="condición; else otraPlantilla"`).  
- La composición de condiciones y bucles anidados era poco legible.  
- No existía un soporte tan fuerte de **TypeScript** en las expresiones de plantilla.  

Con Angular 20, el equipo introduce un **lenguaje de plantillas más expresivo**, que hace que el código sea más fácil de leer, mantener y depurar.

### 4.1.2. El bloque `@if`

El nuevo `@if` sustituye a `*ngIf`.  
Ejemplo clásico con `*ngIf`:

```html
<div *ngIf="isLoggedIn; else loginBlock">
  Bienvenido, usuario
</div>
<ng-template #loginBlock>
  <p>Por favor, inicia sesión</p>
</ng-template>
```

Con el nuevo `@if`:

```html
@if (isLoggedIn) {
  <div>Bienvenido, usuario</div>
} @else {
  <p>Por favor, inicia sesión</p>
}
```

**Ventajas**:
- Sintaxis más clara y natural.  
- El bloque `@else` se integra directamente, sin necesidad de `ng-template`.  
- Mejor soporte de autocompletado y tipos en editores.

### 4.1.3. El bloque `@for`

El nuevo `@for` reemplaza a `*ngFor`.  
Ejemplo clásico:

```html
<ul>
  <li *ngFor="let user of users; trackBy: trackById">
    {{ user.name }}
  </li>
</ul>
```

Con `@for`:

```html
<ul>
  @for (user of users; track user.id) {
    <li>{{ user.name }}</li>
  }
</ul>
```

**Ventajas**:
- La cláusula `track` es más explícita y clara que `trackBy`.  
- La sintaxis se parece más a un bucle tradicional.  
- Permite anidar bucles y condiciones de forma más legible.

### 4.1.4. El bloque `@switch`

El nuevo `@switch` sustituye a `*ngSwitch`.  
Ejemplo clásico:

```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'loading'">Cargando...</p>
  <p *ngSwitchCase="'success'">Datos cargados</p>
  <p *ngSwitchDefault>Error</p>
</div>
```

Con `@switch`:

```html
@switch (status) {
  @case ('loading') {
    <p>Cargando...</p>
  }
  @case ('success') {
    <p>Datos cargados</p>
  }
  @default {
    <p>Error</p>
  }
}
```

**Ventajas**:
- Sintaxis más limpia y estructurada.  
- Los bloques `@case` y `@default` son más fáciles de leer.  
- Se acerca a la sintaxis de un `switch` en TypeScript.

### 4.1.5. Beneficios globales del nuevo control flow

- **Legibilidad**: el código se parece más a estructuras de control de TypeScript.  
- **Menos plantillas auxiliares**: ya no necesitas `ng-template` para `else` o `switch`.  
- **Mejor integración con Signals**: los bloques reaccionan de forma más natural a cambios en datos reactivos.  
- **Mayor consistencia**: la sintaxis es uniforme entre `@if`, `@for` y `@switch`.

## 4.2 Diferencias y ventajas respecto a `*ngIf` y `*ngFor` tradicionales

Con Angular 20, el nuevo **control de flujo con `@if` y `@for`** no es simplemente un cambio estético: representa una evolución importante respecto a las directivas estructurales clásicas `*ngIf` y `*ngFor`.  
Veamos en detalle cuáles son las diferencias y qué ventajas aportan en proyectos reales.

### 4.2.1. Sintaxis más clara y natural

- **Antes**:  
  ```html
  <div *ngIf="isLoggedIn; else loginBlock">
    Bienvenido
  </div>
  <ng-template #loginBlock>
    <p>Por favor, inicia sesión</p>
  </ng-template>
  ```

- **Ahora**:  
  ```html
  @if (isLoggedIn) {
    <div>Bienvenido</div>
  } @else {
    <p>Por favor, inicia sesión</p>
  }
  ```

👉 La nueva sintaxis elimina la necesidad de `ng-template` y se parece más a un `if/else` de TypeScript, lo que mejora la **legibilidad** y reduce el código “ceremonial”.

### 4.2.2. Eliminación del asterisco y plantillas implícitas

- Con `*ngIf` y `*ngFor`, el asterisco (`*`) era un atajo para crear un `ng-template` oculto.  
- Esto generaba confusión, sobre todo para quienes se iniciaban en Angular.  

Con `@if` y `@for`, el bloque es explícito y no hay plantillas ocultas: lo que ves en el HTML es exactamente lo que Angular procesa.

### 4.2.3. Mejor soporte de tipos y tooling

- El nuevo control flow está **integrado en el compilador** de Angular, lo que permite:
  - Autocompletado más preciso en editores.
  - Errores de compilación más claros.
  - Mejor integración con **Signals** y datos reactivos.

Esto significa menos errores en tiempo de ejecución y más ayuda en tiempo de desarrollo.

### 4.2.4. `@for` y el nuevo algoritmo de *diffing*

- `*ngFor` usaba un algoritmo de comparación menos eficiente, que podía provocar renders innecesarios.  
- `@for` introduce un **nuevo algoritmo de diffing** que mejora hasta un **90% el rendimiento en listas grandes**.  
- Además, la cláusula `track` es más clara que `trackBy`:

  ```html
  @for (user of users; track user.id) {
    <li>{{ user.name }}</li>
  }
  ```

👉 Esto hace que el renderizado de listas sea más rápido y predecible.

### 4.2.5. Nuevas capacidades: `@empty`

Con `*ngFor`, si querías mostrar un mensaje cuando la lista estaba vacía, debías combinarlo con un `*ngIf`.  
Ahora, `@for` incluye directamente un bloque `@empty`:

```html
@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <p>No hay elementos disponibles</p>
}
```

Esto simplifica mucho el código y lo hace más expresivo.

### 4.2.6. Ventajas globales

| Aspecto                  | `*ngIf` / `*ngFor` (clásicos) | `@if` / `@for` (Angular 20) |
|---------------------------|-------------------------------|-----------------------------|
| Sintaxis                  | Basada en directivas con `*`  | Bloques nativos, estilo TS  |
| Else / casos alternativos | Requiere `ng-template`        | Bloques `@else`, `@empty`   |
| Legibilidad               | Menor, más verbosa            | Mayor, más natural          |
| Rendimiento en listas     | Algoritmo clásico             | Nuevo diffing, más rápido   |
| Tooling y tipos           | Limitado                      | Mejor soporte en compilador |
| Futuro                    | En proceso de deprecación     | ✅ Nuevo estándar recomendado |

## 4.3 Directivas de atributos reactivas integradas con Signals

Hasta ahora hemos visto cómo los **bloques de control de flujo** (`@if`, `@for`, `@switch`) se integran de forma natural con Signals. Pero Angular 20 no se queda ahí: también podemos aprovechar la reactividad de Signals en las **directivas de atributos**, es decir, aquellas que modifican el comportamiento o el estilo de un elemento existente sin alterar su estructura.

Esto abre la puerta a un patrón muy poderoso: **directivas reactivas**, que responden automáticamente a cambios en Signals sin necesidad de suscripciones manuales ni `async pipe`.

### 4.3.1. ¿Qué son las directivas de atributos reactivas?

Una **directiva de atributo** es aquella que se aplica sobre un elemento HTML para modificar su apariencia o comportamiento. Ejemplos clásicos son `ngClass`, `ngStyle` o `ngModel`.

Con Signals, podemos crear directivas que:

- Escuchen cambios en un Signal.
- Actualicen automáticamente el DOM.
- Mantengan el código más declarativo y limpio.

### 4.3.2. Ejemplo básico: directiva `highlightOn`

Imaginemos que queremos resaltar un elemento cuando un Signal booleano esté activo.

```ts
import { Directive, effect, ElementRef, input } from '@angular/core';

@Directive({
  selector: '[highlightOn]'
})
export class HighlightOn {

  highlightOn = input.required<boolean>();

  constructor(private el: ElementRef) {
    // Creamos un efecto reactivo
    effect(() => {
      if (this.highlightOn()) {
        this.el.nativeElement.style.backgroundColor = 'yellow';
      } else {
        this.el.nativeElement.style.backgroundColor = 'transparent';
      }
    });
  }

}
```

Uso en plantilla:

```html
<div [highlightOn]="isHighlighted()">
  Este texto se resalta automáticamente
</div>
```

Donde `isHighlighted` es un **Signal** en el componente:

```ts
isHighlighted = signal(false);
```

### 4.3.3. Ventajas frente a directivas tradicionales

- **Sin suscripciones manuales**: no necesitamos `subscribe()` ni `ngOnDestroy`.  
- **Reactividad declarativa**: el `effect()` se encarga de escuchar el Signal y actualizar el DOM.  
- **Menos boilerplate**: el código es más corto y expresivo.  
- **Mejor integración con el compilador**: Angular sabe qué Signals afectan a la directiva y optimiza el *Change Detection*.

### 4.3.4. Ejemplo avanzado: directiva `debounceInput`

Podemos crear una directiva que convierta el valor de un `<input>` en un Signal reactivo con *debounce* integrado.

```ts
import { Directive, effect, ElementRef, inject, output, signal } from '@angular/core';

@Directive({
  selector: '[debounceInput]'
})
export class DebounceInput {

  private el = inject(ElementRef<HTMLInputElement>);
  private valueSignal = signal('');

  // Exponemos un output reactivo
  value = output<string>();

  constructor() {
    // Escuchamos cambios en el input
    this.el.nativeElement.addEventListener('input', (e: Event) => {
      const target = e.target as HTMLInputElement;
      this.valueSignal.set(target.value);
    });

    // Creamos un efecto con debounce manual
    let timeout: any;
    effect(() => {
      const current = this.valueSignal();
      clearTimeout(timeout);
      timeout = setTimeout(() => this.value.emit(current), 500);
    });
  }

}
```

Uso en plantilla:

```html
<input debounceInput (value)="onSearch($event)" />
```

Ahora cada vez que el usuario escribe, el evento `value` se emite con un retardo de 300ms, sin necesidad de RxJS ni `FormControl`.

### 4.3.5. Buenas prácticas

- Usa `effect()` dentro de la directiva para reaccionar a Signals.  
- Prefiere `input()` y `output()` en lugar de `@Input()` y `@Output()` cuando trabajes con Signals.  
- Mantén las directivas **pequeñas y específicas**: una directiva = un comportamiento.  
- Evita lógica compleja en la directiva; delega en servicios si es necesario.  

## 4.4 Host Directives: reutilización y encapsulación de lógica transversal

En aplicaciones grandes, es común que distintos componentes necesiten **comportamientos repetidos**: validaciones, estilos dinámicos, accesibilidad, manejo de eventos globales, etc.

Hasta ahora, la solución típica era crear **directivas de atributos** y aplicarlas manualmente en cada plantilla. Sin embargo, esto podía generar **duplicación de código** y dificultar la **encapsulación**.

Con Angular 20, gracias a la **Directive Composition API**, disponemos de las **Host Directives**: una forma de **inyectar directivas directamente en un componente** para reutilizar lógica transversal sin necesidad de aplicarlas explícitamente en la plantilla.

### 4.4.1. ¿Qué son las Host Directives?

Las **Host Directives** permiten que un componente “herede” el comportamiento de una o varias directivas, aplicándolas automáticamente a su elemento host.  

En otras palabras:  
- Son directivas que se **componen dentro de un componente**.  
- Se aplican de forma **estática en tiempo de compilación**.  
- Sus **host bindings, listeners e inputs/outputs** se integran en el componente.  

Esto significa que puedes encapsular lógica común en directivas y luego **reutilizarla en múltiples componentes** sin repetir código ni ensuciar las plantillas.

### 4.4.2. Ejemplo básico

Supongamos que tenemos una directiva que añade un tooltip:

```ts
import { Directive, ElementRef, HostListener, input } from '@angular/core';

@Directive({
  selector: '[appTooltip]'
})
export class Tooltip {

  text = input<string>('');

  constructor(private elRef: ElementRef<HTMLElement>) {}

  @HostListener('mouseenter')
  showTooltip() {
    const host = this.elRef.nativeElement;
    
    // Lógica para mostrar tooltip (ver en la consola del navegador)
    console.log('Mostrar tooltip:', this.text(), host);
  }

  @HostListener('mouseleave')
  hideTooltip() {
    console.log('Ocultar tooltip');
  }
}
```

Ahora queremos que un componente `UserCard` siempre tenga este comportamiento, sin necesidad de escribir `[tooltip]` en la plantilla.

```ts
import { Component } from '@angular/core';
import { Tooltip } from '../directives/tooltip';


@Component({
  selector: 'app-user-card',
  template: `<div class="card">Contenido de la tarjeta</div>`,
  hostDirectives: [
    {
      directive: Tooltip,
      inputs: ['text: tooltipText'] // Exponemos el input con un alias
    }
  ]
})
export class UserCardComponent {}
```

Uso en plantilla:

```html
<app-user-card tooltipText="Información del usuario"></app-user-card>
```

👉 El componente `UserCard` **hereda automáticamente** la lógica de `TooltipDirective`.  
No necesitamos aplicarla manualmente en la plantilla.

### 4.4.3. Ventajas de las Host Directives

- **Reutilización real**: encapsulas lógica transversal (tooltips, accesibilidad, validaciones, estilos dinámicos) en directivas y las aplicas en múltiples componentes.  
- **Plantillas más limpias**: no necesitas añadir atributos extra en cada uso.  
- **Encapsulación**: el componente expone solo los inputs/outputs que decidas.  
- **Composición flexible**: puedes aplicar varias directivas a un mismo componente.  
- **Consistencia**: todos los componentes que usan la misma host directive comparten el mismo comportamiento.

### 4.4.4. Ejemplo avanzado: accesibilidad

Imagina que quieres que todos tus botones personalizados tengan soporte de accesibilidad (`aria-label`, `role`, etc.).  

```ts
import { Directive, effect, ElementRef, HostBinding, input } from '@angular/core';

@Directive({
  selector: '[a11y]'
})
export class A11y {

  label = input<string>('');
  role = input<string>('button');

  @HostBinding('attr.aria-label') ariaLabel: string | null = null;
  @HostBinding('attr.role') ariaRole: string | null = null;

  constructor() {
    effect(() => {
      // Mantener atributos aria/role sin tocar el contenido proyectado
      this.ariaRole = this.role() || null;
      this.ariaLabel = this.label() || null;
    });
  }

}

```

Ahora, cualquier componente de botón puede heredar esta directiva:

```ts
import { Component } from "@angular/core";
import { A11y } from "../directives/a11y";

@Component({
  selector: 'app-primary-button',
  template: `<button class="btn-primary"><ng-content></ng-content></button>`,
  hostDirectives: [
    {
      directive: A11y,
      inputs: ['label: ariaLabel', 'role: ariaRole']
    }
  ]
})
export class PrimaryButtonComponent {}

```

Uso:

```html
<app-primary-button ariaLabel="Guardar cambios">Guardar</app-primary-button>
```

### 4.4.5. Buenas prácticas

- **Usa Host Directives para lógica transversal**: accesibilidad, estilos comunes, tooltips, validaciones.  
- **No abuses**: si la directiva solo se usa en un lugar, probablemente no necesite ser host directive.  
- **Expón solo lo necesario**: controla qué inputs/outputs se heredan para no sobrecargar la API del componente.  
- **Combínalas con Signals**: los `input()` y `effect()` hacen que la lógica sea aún más reactiva y declarativa.  

## 4.5 Manipulación del DOM de forma reactiva y segura en Angular 20

En Angular 20, la manipulación del DOM se ha vuelto más **declarativa, reactiva y segura** gracias a la integración con **Signals** y a nuevas recomendaciones que evitan inconsistencias en entornos como **SSR (Server-Side Rendering)**, **hidratación** y **renderizado híbrido**.  

Modificar el DOM directamente con `ElementRef` o APIs nativas de JavaScript puede romper la coherencia entre lo que Angular cree que hay en la vista y lo que realmente existe en el navegador. Por eso, la filosofía actual es:  

- **Evitar manipulación manual siempre que sea posible.**  
- **Usar bindings, host bindings y directivas reactivas** para expresar cambios en el DOM.  
- **Aprovechar Signals y efectos (`effect()`)** para que las actualizaciones sean automáticas y seguras.  

### 4.5.1. El problema de la manipulación manual

Ejemplo clásico (no recomendado):

```ts
constructor(private el: ElementRef) {}

ngOnInit() {
  this.el.nativeElement.style.backgroundColor = 'red';
}
```

Este enfoque:  
- Funciona en el navegador, pero puede fallar en SSR o durante la hidratación.  
- No es reactivo: si cambia el estado, el DOM no se actualiza automáticamente.  
- Puede introducir vulnerabilidades de seguridad (XSS) si se manipula contenido HTML sin sanitización.  

### 4.5.2. La forma recomendada: bindings declarativos

Angular ofrece **host bindings** y **data bindings** que permiten manipular el DOM de forma declarativa:

```ts
import { Component, signal } from "@angular/core";

@Component({
  selector: 'app-alert',
  template: `<p>{{ message() }}</p>`,
  host: {
    '[class.visible]': 'isVisible()',
    '[style.backgroundColor]': '"limegreen"',
    '[style.display]': '"block"'
  }
})
export class AlertComponent {
  message = signal('Atención: cambios guardados');
  isVisible = signal(true);
}

```

Aquí:  
- La clase `visible` se añade o elimina automáticamente según el Signal `isVisible`.  
- El color de fondo se aplica de forma declarativa.  
- No hay manipulación manual: Angular mantiene la coherencia del DOM.  

### 4.5.3. Directivas reactivas para encapsular lógica

Si necesitas lógica más compleja, encapsúlala en una directiva con Signals y `effect()`:

```ts
@Directive({
  selector: '[autoFocus]',
  standalone: true
})
export class AutoFocusDirective {
  enabled = input<boolean>(true);

  constructor(private el: ElementRef) {
    effect(() => {
      if (this.enabled()) {
        queueMicrotask(() => this.el.nativeElement.focus());
      }
    });
  }
}
```

Uso:

```html
<input autoFocus [enabled]="shouldFocus()" />
```

👉 Con esto, el input se enfoca automáticamente cuando el Signal `shouldFocus` es `true`, sin necesidad de `ViewChild` ni `ngAfterViewInit`.

### 4.5.4. Manipulación segura de contenido dinámico

Cuando necesites insertar HTML dinámico, **nunca lo hagas directamente** con `innerHTML`, ya que puede abrir la puerta a ataques XSS.  
Angular proporciona el servicio `DomSanitizer`:

```ts
constructor(private sanitizer: DomSanitizer) {}

htmlContent = signal('<b>Texto seguro</b>');

safeHtml = computed(() =>
  this.sanitizer.bypassSecurityTrustHtml(this.htmlContent())
);
```

En la plantilla:

```html
<div [innerHTML]="safeHtml()"></div>
```

### 4.5.5. Integración con SSR e hidratación

En aplicaciones modernas con **SSR** e **hidratación**, es fundamental que el DOM generado en el servidor coincida con el del cliente.  
Buenas prácticas:  
- Evita añadir o eliminar nodos manualmente antes de la hidratación.  
- Usa `@if`, `@for` y `@switch` para controlar el flujo de la vista.  
- Si necesitas manipulación condicional, hazlo **después de la hidratación** (ej. en `ngAfterViewInit`).  

### 4.5.6. Buenas prácticas de manipulación reactiva

- **Prefiere Signals + bindings** en lugar de `ElementRef`.  
- **Encapsula lógica en directivas** para reutilizar comportamientos.  
- **Usa `Renderer2` solo cuando sea imprescindible** (ej. compatibilidad con plataformas no DOM).  
- **Sanitiza siempre contenido dinámico** con `DomSanitizer`.  
- **Minimiza el acceso directo al DOM** para no romper SSR ni hidratación.  

## 4.6 Integración de eventos con Signals en directivas personalizadas

Uno de los grandes avances de Angular 20 es la posibilidad de trabajar con **Signals** como mecanismo central de reactividad. Hasta ahora, cuando queríamos escuchar eventos del DOM en una directiva personalizada, solíamos recurrir a decoradores como `@HostListener` o a la suscripción manual de eventos con `addEventListener`. Aunque estos enfoques siguen siendo válidos, la llegada de Signals nos permite **integrar los eventos de usuario directamente en el flujo reactivo de la aplicación**, lo que simplifica el código, reduce la necesidad de suscripciones manuales y mejora la legibilidad.

En este apartado veremos cómo crear directivas que no solo reaccionen a eventos del DOM, sino que además **expongan esos eventos como Signals**, de modo que otros componentes o directivas puedan reaccionar automáticamente a ellos.

### 4.6.1. El enfoque tradicional: `@HostListener`

Antes de Angular 20, una directiva que reaccionaba a un evento podía escribirse así:

```ts
@Directive({
  selector: '[hoverHighlight]'
})
export class HoverHighlightDirective {
  constructor(private el: ElementRef) {}

  @HostListener('mouseenter')
  onMouseEnter() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }

  @HostListener('mouseleave')
  onMouseLeave() {
    this.el.nativeElement.style.backgroundColor = 'transparent';
  }
}
```

Este patrón funciona, pero tiene limitaciones:  
- La lógica está acoplada directamente al DOM.  
- No es fácil reutilizar el estado del evento en otros lugares.  
- No se integra con Signals, por lo que no podemos reaccionar de forma declarativa en otras partes de la aplicación.  

## 4.6.2. El nuevo enfoque: eventos como Signals

En **Angular 20**, podemos representar el estado de eventos del DOM mediante **Signals**, lo que nos permite reaccionar de forma declarativa y sin suscripciones manuales. Cada vez que el evento ocurre, el Signal se actualiza y cualquier parte de la aplicación que lo consuma se reactualiza automáticamente.

### ¿Por qué usar Signals para eventos?

*   Evitamos **callbacks** y **suscripciones manuales**.
*   El estado se vuelve **reactivo** y fácil de consumir en plantillas.
*   Compatible con **computed** y **effect**, para lógica derivada.

***

### Ejemplo: Directiva `hoverSignal`

Usaremos una directiva que expone un Signal indicando si el ratón está sobre el elemento. Para ello, aplicamos buenas prácticas:

*   **`@HostListener`** para gestionar eventos.
*   **`exportAs`** para acceder al Signal desde la plantilla.
*   Lectura del Signal como **función** (`isHovered()`).

```ts
import { Directive, HostListener, signal } from '@angular/core';

@Directive({
  selector: '[hoverSignal]',
  exportAs: 'hover' // permite #h="hover" en la plantilla
})
export class HoverSignalDirective {
  readonly isHovered = signal(false);

  @HostListener('mouseenter')
  onEnter() {
    this.isHovered.set(true);
  }

  @HostListener('mouseleave')
  onLeave() {
    this.isHovered.set(false);
  }
}
```

***

### Uso en plantilla

```html
<div hoverSignal #h="hover">
  Hover: {{ h.isHovered() ? 'sí' : 'no' }}
</div>
```

### Consideraciones importantes

*   **Alcance del Signal**: vive en la instancia de la directiva. Para usarlo en otras partes:
    *   `exportAs` en la plantilla.
    *   Elevar el estado a un **servicio con Signals** si necesitas compartirlo globalmente.
*   **Limpieza**: si usas `addEventListener`, elimina listeners en `ngOnDestroy`.
*   **SSR**: usa `Renderer2.listen` para mayor compatibilidad.
*   **Lectura en plantilla**: recuerda que los Signals se leen como función: `isHovered()`.


## 4.6.3. Exponiendo eventos con `output()` (API function‑based)

Angular ofrece la utilidad `output()` para declarar **eventos personalizados** en componentes y directivas. Esta API devuelve un `OutputRef` que permite:

*   **Emitir eventos** con `.emit(...)`.
*   **Escuchar eventos** en el padre mediante la sintaxis `(evento)="..."`.
*   **Suscribirse** directamente con `.subscribe(...)` si lo necesitas en código.

> **Nota de versión**: `output()` se introdujo en Angular **v17.3** y es **estable desde v19**. Existe migración oficial desde `@Output`.

### Ejemplo: Directiva `clickSignal`

Usaremos `@HostListener` para gestionar el evento y exponerlo como output:

```ts
import { Directive, HostListener, output } from '@angular/core';

@Directive({
  selector: '[clickSignal]',
  standalone: true
})
export class ClickSignalDirective {
  // Declaramos un output tipado
  readonly clicked = output<MouseEvent>(); // OutputRef<MouseEvent>

  @HostListener('click', ['$event'])
  onClick(event: MouseEvent) {
    this.clicked.emit(event); // Emitimos el evento al padre
  }
}
```

**Uso en plantilla del padre:**

```html
<button clickSignal (clicked)="onButtonClicked($event)">
  Haz clic aquí
</button>
```

### Consideraciones importantes

*   **No son Signals**: `output()` no convierte el evento en un Signal; es una API declarativa para outputs.
*   **Buenas prácticas**:
    *   Prefiere `@HostListener` o `Renderer2.listen(...)` para gestionar eventos.
    *   Si usas `addEventListener`, limpia en `ngOnDestroy`.
*   **Tipado**: especifica el tipo genérico en `output<T>()` para mayor seguridad.


### 4.6.4. Combinando eventos y Signals con `effect()`

La verdadera potencia aparece cuando combinamos **eventos** con **efectos reactivos**.  
Imagina una directiva que cambia dinámicamente el estilo de un elemento en función de si está siendo clicado o no:

```ts
import { Directive, effect, ElementRef, signal } from '@angular/core';

@Directive({
  selector: '[toggleHighlight]'
})
export class ToggleHighlight {

  private active = signal(false);

  constructor(private el: ElementRef) {
    this.el.nativeElement.addEventListener('click', () => {
      this.active.update(v => !v);
    });

    effect(() => {
      this.el.nativeElement.style.backgroundColor = this.active()
        ? 'lightblue'
        : 'transparent';
    });
  }
}
```

Aquí ocurre algo muy interesante:  
- El evento `click` actualiza el Signal `active`.  
- El `effect()` escucha ese Signal y actualiza el DOM automáticamente.  
- No necesitamos `ngOnDestroy`, ni `Renderer2`, ni `ChangeDetectorRef`. Todo fluye de manera declarativa.  

### 4.6.5. Ventajas de este enfoque

- **Reactividad total**: los eventos se convierten en parte del sistema de Signals, lo que permite combinarlos con otros estados reactivos.  
- **Menos boilerplate**: no necesitamos suscripciones manuales ni `ngOnDestroy`.  
- **Mayor expresividad**: el código refleja directamente la intención: “cuando ocurra este evento, actualiza este Signal”.  
- **Reutilización**: el estado derivado de un evento puede ser consumido por múltiples componentes o directivas.  
- **Compatibilidad**: podemos seguir usando `@HostListener` o `Renderer2` cuando sea necesario, pero ahora tenemos una alternativa más declarativa.  

### 4.6.6. Ejemplo práctico: directiva de contador de clics

Para cerrar, un ejemplo completo que combina todo lo visto:

```ts
@Directive({
  selector: '[clickCounter]',
  standalone: true
})
export class ClickCounterDirective {
  count = signal(0);

  constructor(private el: ElementRef) {
    this.el.nativeElement.addEventListener('click', () => {
      this.count.update(c => c + 1);
    });

    effect(() => {
      this.el.nativeElement.textContent =
        `Has hecho clic ${this.count()} veces`;
    });
  }
}
```

Uso:

```html
<button clickCounter></button>
```

Cada clic actualiza el Signal `count`, y el `effect()` se encarga de reflejarlo en el DOM.  
El resultado es un comportamiento completamente reactivo, sin necesidad de lógica extra en el componente.

## 4.7 Compatibilidad con proyectos legacy: coexistencia con `*ngIf` y `*ngFor`

La introducción del nuevo sistema de control de flujo en Angular 20 —con `@if`, `@for` y `@switch`— no significa que las directivas estructurales clásicas (`*ngIf`, `*ngFor`, `*ngSwitch`) hayan dejado de funcionar de inmediato. Angular mantiene un fuerte compromiso con la **compatibilidad hacia atrás**, especialmente en entornos **enterprise** donde existen aplicaciones grandes, con años de desarrollo y miles de plantillas que utilizan la sintaxis tradicional.  

Por ello, Angular 20 permite la **coexistencia de ambos enfoques**: puedes seguir utilizando `*ngIf` y `*ngFor` en tus plantillas legacy, mientras introduces progresivamente `@if` y `@for` en nuevas secciones del proyecto.  

### 4.7.1. Estado actual de la compatibilidad

- **Soporte dual**: Angular 20 soporta tanto la sintaxis clásica como la nueva.  
- **Deprecación progresiva**: aunque no hay una eliminación inmediata, el equipo de Angular ha dejado claro que el futuro está en los bloques `@if` y `@for`.  
- **Migración gradual**: se recomienda migrar poco a poco, aprovechando nuevas funcionalidades en módulos o componentes nuevos, sin necesidad de refactorizar todo el proyecto de golpe.  

### 4.7.2. Estrategias de coexistencia

En proyectos grandes, lo más habitual es encontrarse con **miles de ocurrencias** de `*ngIf` y `*ngFor`. Migrarlas todas de golpe puede ser arriesgado y costoso. Por eso, la estrategia recomendada es la **coexistencia controlada**:

- **Nuevos componentes → nueva sintaxis**  
  Todo lo que se desarrolle a partir de Angular 20 debería usar `@if` y `@for`.  
- **Componentes legacy → sintaxis clásica**  
  Mantén `*ngIf` y `*ngFor` en componentes antiguos hasta que haya tiempo y recursos para migrarlos.  
- **Migración progresiva**  
  Usa herramientas de migración automática (`ng generate @angular/core:control-flow-migration`) o reglas de ESLint que te avisen cuando uses directivas legacy.  

### 4.7.3. Ejemplo de coexistencia en un mismo proyecto

```html
<!-- Componente legacy -->
<div *ngIf="isLoggedIn; else loginBlock">
  Bienvenido, usuario
</div>
<ng-template #loginBlock>
  <p>Por favor, inicia sesión</p>
</ng-template>

<!-- Componente nuevo -->
@if (isLoggedIn) {
  <div>Bienvenido, usuario</div>
} @else {
  <p>Por favor, inicia sesión</p>
}
```

Ambos fragmentos pueden convivir en la misma aplicación sin ningún problema.  

### 4.7.4. Herramientas de ayuda a la migración

- **Esquemas de migración oficiales**: Angular incluye comandos de migración que transforman automáticamente `*ngIf` y `*ngFor` en `@if` y `@for`.  
- **Reglas de ESLint**: puedes configurar reglas que marquen como error el uso de directivas legacy, forzando a tu equipo a usar la nueva sintaxis en código nuevo.  
- **Refactor manual progresivo**: en componentes críticos, conviene revisar manualmente para aprovechar mejoras como `@empty` en `@for`.  

### 4.7.5. Buenas prácticas en proyectos enterprise

- **Define una política de migración**: decide si migrarás todo de golpe o de forma progresiva.  
- **Capacita al equipo**: asegúrate de que todos los desarrolladores entienden la nueva sintaxis.  
- **Usa linters y revisiones de código**: para evitar que se introduzcan nuevas directivas legacy en código nuevo.  
- **Prioriza componentes críticos**: migra primero aquellos que se renderizan con más frecuencia o que contienen listas grandes, para aprovechar las mejoras de rendimiento.  


## 4.8 Casos prácticos de directivas avanzadas en entornos enterprise

En aplicaciones **enterprise**, las directivas avanzadas no son un simple recurso técnico: se convierten en **bloques reutilizables de lógica transversal**, que permiten mantener la consistencia, mejorar la productividad del equipo y garantizar la escalabilidad del proyecto.  
En este apartado veremos **casos prácticos reales** donde las directivas personalizadas, integradas con **Signals** y las nuevas APIs de Angular 20, aportan un valor diferencial.

### 4.8.1. Directiva de control de permisos (seguridad en UI)

En entornos corporativos, no todos los usuarios tienen los mismos permisos. Una directiva puede encargarse de mostrar u ocultar elementos según el rol del usuario.

```ts
@Directive({
  selector: '[hasPermission]',
  standalone: true
})
export class HasPermissionDirective {
  private el = inject(ElementRef);
  private authService = inject(AuthService);

  permission = input.required<string>();

  constructor() {
    effect(() => {
      const canShow = this.authService.hasPermission(this.permission());
      this.el.nativeElement.style.display = canShow ? '' : 'none';
    });
  }
}
```

Uso en plantilla:

```html
<button hasPermission="ADMIN">Eliminar usuario</button>
```

👉 Con esto, la lógica de permisos queda centralizada y reutilizable en toda la aplicación.

### 4.8.2. Directiva de *loading state* para botones

En aplicaciones enterprise, los botones suelen necesitar estados de carga para evitar acciones duplicadas.

```ts
@Directive({
  selector: '[loadingButton]',
  standalone: true
})
export class LoadingButtonDirective {
  isLoading = input<boolean>(false);

  constructor(private el: ElementRef<HTMLButtonElement>) {
    effect(() => {
      this.el.nativeElement.disabled = this.isLoading();
      this.el.nativeElement.textContent = this.isLoading()
        ? 'Procesando...'
        : this.el.nativeElement.getAttribute('data-label') ?? 'Enviar';
    });
  }
}
```

Uso:

```html
<button loadingButton [isLoading]="saving()" data-label="Guardar cambios"></button>
```

👉 Así evitamos duplicar lógica de deshabilitado y mensajes en cada componente.

### 4.8.3. Directiva de *tracking* para analítica

En entornos enterprise es habitual integrar analítica (Google Analytics, Azure Insights, etc.). Una directiva puede encargarse de enviar eventos automáticamente.

```ts
@Directive({
  selector: '[trackEvent]',
  standalone: true
})
export class TrackEventDirective {
  eventName = input.required<string>();

  constructor(private el: ElementRef, private analytics: AnalyticsService) {
    this.el.nativeElement.addEventListener('click', () => {
      this.analytics.track(this.eventName());
    });
  }
}
```

Uso:

```html
<button trackEvent="DownloadReport">Descargar informe</button>
```

👉 Esto permite instrumentar la aplicación sin ensuciar los componentes con lógica de analítica.

### 4.8.4. Directiva de accesibilidad (A11y)

En proyectos grandes, garantizar la accesibilidad es crítico. Una directiva puede añadir atributos ARIA de forma automática.

```ts
@Directive({
  selector: '[a11yLabel]',
  standalone: true
})
export class A11yLabelDirective {
  label = input.required<string>();

  constructor(private el: ElementRef) {
    effect(() => {
      this.el.nativeElement.setAttribute('aria-label', this.label());
    });
  }
}
```

Uso:

```html
<input type="text" a11yLabel="Nombre completo" />
```

👉 Con esto, aseguramos que todos los inputs cumplen con estándares de accesibilidad.

### 4.8.5. Directiva de auditoría de formularios

En entornos enterprise, los formularios suelen ser críticos. Una directiva puede registrar automáticamente cambios para auditoría.

```ts
@Directive({
  selector: '[auditForm]',
  standalone: true
})
export class AuditFormDirective {
  constructor(private el: ElementRef<HTMLFormElement>, private audit: AuditService) {
    this.el.nativeElement.addEventListener('change', (e: Event) => {
      const target = e.target as HTMLInputElement;
      this.audit.logChange({
        field: target.name,
        value: target.value,
        timestamp: new Date()
      });
    });
  }
}
```

👉 Esto permite cumplir con normativas de trazabilidad sin añadir lógica repetida en cada formulario.
