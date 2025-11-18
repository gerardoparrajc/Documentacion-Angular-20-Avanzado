# 17. Migración de Angular 17 a Angular 20

## 17.1. Recordatorio de las diferencias clave entre Angular 17 y Angular 20

Antes de planificar una migración, es fundamental repasar qué ha cambiado entre **Angular 17** y **Angular 20**.  
No se trata solo de nuevas APIs, sino de un cambio de paradigma en cómo Angular se concibe: más ligero, más declarativo y con soporte nativo para capacidades que antes dependían de librerías externas.

### 17.1.1. Standalone-first y desaparición progresiva de NgModules
- En Angular 17 todavía convivían **NgModules** y **standalone components**.  
- En Angular 20, el enfoque es **standalone-first**:  
  - Los módulos ya no son necesarios en la mayoría de los casos.  
  - La API de `bootstrapApplication` y `provide*` (ej. `provideHttpClient`, `provideRouter`) es la forma recomendada de configurar la aplicación.  
  - Los imports en el decorador de componentes deben ser explícitos y los pipes también pueden ser standalone.  
- Esto simplifica la arquitectura y reduce el boilerplate.

### 17.1.2. Signals como modelo de reactividad principal
- En Angular 17, los **Signals** se introdujeron como una API experimental.  
- En Angular 20, los Signals son **el modelo de reactividad recomendado**:  
  - Sustituyen muchos casos de `RxJS` para estado local y bindings simples.  
  - Integración profunda con el **Change Detection**: Angular detecta cambios automáticamente cuando un Signal se actualiza.  
- RxJS sigue siendo útil para flujos complejos, pero Signals son ahora la primera opción.

### 17.1.3. SSR nativo integrado en el framework
- En Angular 17, el **Server-Side Rendering (SSR)** dependía de Angular Universal.  
- En Angular 20, el SSR es **parte del framework**, activable desde la creación del proyecto (`ng new mi-app --ssr`).  
- Esto significa:  
  - Menos configuración manual.  
  - Mejor soporte para **SEO multilingüe**.  
  - Integración directa con streaming y prefetch de datos.

### 17.1.4. Routing y lazy loading simplificados
- Angular 17 aún requería configuraciones más verbosas para lazy loading.  
- Angular 20 permite:  
  - **Lazy loading con funciones** (`loadComponent`, `loadChildren`).  
  - Integración con Signals para reaccionar a parámetros de ruta.  
  - Prefijos de idioma en rutas más fáciles de configurar para i18n.  

### 17.1.5. Herramientas y Dev Experience
- **Angular DevTools** ahora soporta Signals y SSR.  
- **CLI** más optimizada: builds más rápidas, soporte mejorado para `esbuild` y optimizaciones automáticas.  
- **Testing**: integración más fluida con Jest, Playwright y herramientas de accesibilidad como axe-core.  

### 17.1.6. Deprecaciones y cambios importantes
- **NgModules**: siguen funcionando, pero ya no son el camino recomendado.  
- **Zone.js**: aunque sigue presente, Angular 20 avanza hacia un modelo **zoneless** gracias a Signals.  
- **APIs obsoletas**: algunas funciones de `Renderer2` y configuraciones antiguas de i18n han sido eliminadas.  


## 17.2. Migración de sintaxis de control flow: de `*ngIf`/`*ngFor` a `@if`/`@for`

Uno de los cambios más visibles entre Angular 17 y Angular 20 es la **nueva sintaxis de control flow**.  
Hasta Angular 17, usábamos las directivas estructurales clásicas `*ngIf`, `*ngFor` y `*ngSwitch`.  
En Angular 20, estas directivas han sido reemplazadas (aunque siguen funcionando por compatibilidad) por la nueva sintaxis **`@if`, `@for` y `@switch`**, más clara, más expresiva y mejor integrada con el compilador.

### 17.2.1. ¿Por qué este cambio?

- **Legibilidad**: la nueva sintaxis se parece más a estructuras de control de lenguajes como TypeScript o C#.  
- **Expresividad**: soporta bloques `else` y `else if` de forma nativa.  
- **Rendimiento**: el compilador puede optimizar mejor el renderizado.  
- **Consistencia**: unifica la forma de escribir control flow en Angular con un estilo más declarativo.  

### 17.2.2. De `*ngIf` a `@if`

#### Antes (Angular 17)
```html
<div *ngIf="isLoggedIn; else guest">
  <p>Bienvenido, {{ user.name }}</p>
</div>
<ng-template #guest>
  <p>Por favor, inicia sesión</p>
</ng-template>
```

#### Ahora (Angular 20)
```html
@if (isLoggedIn) {
  <p>Bienvenido, {{ user.name }}</p>
} @else {
  <p>Por favor, inicia sesión</p>
}
```

**Ventajas**:
- No necesitamos `ng-template` para el bloque alternativo.  
- La sintaxis es más natural y cercana a un `if/else` clásico.  

### 17.2.3. De `*ngFor` a `@for`

#### Antes (Angular 17)
```html
<ul>
  <li *ngFor="let item of items; index as i">
    {{ i + 1 }} - {{ item }}
  </li>
</ul>
```

#### Ahora (Angular 20)
```html
<ul>
  @for (item of items; let i = $index) {
    <li>{{ i + 1 }} - {{ item.name }}</li>
  }
</ul>
```

Ejemplo con `@empty`:
```html
@for (item of items; track item.id) {
  <p>{{ item.name }}</p>
} @empty {
  <p>No hay elementos disponibles</p>
}
```

**Novedades**:
- Uso de `track` directamente en la sintaxis, más claro que `trackBy` (solo si los items tienen un id).  
- Variables contextuales (`$index`, `$first`, `$last`) siguen disponibles.  
- Bloques `@empty` para manejar listas vacías y mejorar UX y SEO.  

### 17.2.4. Migración progresiva

- **Compatibilidad**: `*ngIf` y `*ngFor` siguen funcionando en Angular 20, por lo que la migración puede hacerse gradualmente.  
- **Estrategia recomendada**:  
  1. Migrar primero componentes nuevos con `@if` y `@for`.  
  2. Refactorizar componentes existentes poco a poco.  
  3. Usar linters y reglas de ESLint para detectar directivas antiguas.  

### 17.2.5. Buenas prácticas

- **Prefiere `@if` y `@for` en código nuevo**: son el estándar moderno.  
- **Usa `@empty` en listas**: mejora la experiencia de usuario.  
- **Aprovecha `track`**: evita renders innecesarios en listas grandes.  
- **Mantén consistencia**: no mezclar en un mismo componente `*ngIf` y `@if`.  


## 17.3. Sustitución progresiva de NgModules por Standalone Components

Uno de los cambios más profundos en la evolución de Angular es el paso de una arquitectura basada en **NgModules** a un modelo **standalone-first**.  
En Angular 17 todavía era común estructurar la aplicación en módulos, pero en Angular 20 los **Standalone Components** son la forma recomendada de construir aplicaciones: más simples, más directas y con menos boilerplate.  

La clave está en que la migración no tiene que ser traumática: podemos hacerlo de forma **progresiva**, manteniendo compatibilidad mientras adoptamos las nuevas prácticas.

### 17.3.1. ¿Por qué abandonar NgModules?

- **Menos complejidad**: ya no necesitamos declarar componentes en un módulo.  
- **Arranque más simple**: `bootstrapApplication` sustituye a `AppModule`.  
- **Imports explícitos**: cada componente importa directamente lo que necesita.  
- **Mejor tree-shaking**: el compilador elimina más fácilmente lo que no se usa.  
- **Consistencia**: todo (componentes, directivas, pipes) puede ser standalone.  

### 17.3.2. Ejemplo de arranque de aplicación

#### Antes (con NgModules)
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

#### Ahora (standalone-first)
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app.component';

bootstrapApplication(AppComponent);
```

### 17.3.3. Migración progresiva de módulos a standalone

#### Paso 1: Identificar módulos candidatos
- **SharedModule**: suele ser el primero en migrar, convirtiendo pipes y componentes compartidos en standalone.  
- **FeatureModules aislados**: migrar módulos que no dependan de demasiados otros.  

#### Paso 2: Convertir componentes a standalone
```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `<p>{{ user.name }}</p>`
})
export class UserCardComponent {
  user = { name: 'Gerardo' };
}
```
// Los pipes también pueden ser standalone y se importan igual que los módulos.

#### Paso 3: Sustituir imports de módulos por imports directos
- En lugar de importar `SharedModule`, cada componente importa lo que necesita (`CommonModule`, `FormsModule`, etc.).  

#### Paso 4: Migrar el enrutador
- Usar `provideRouter` en `main.ts` en lugar de `RouterModule.forRoot`.  
```ts
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)]
});
```

#### Paso 5: Eliminar NgModules obsoletos
- Una vez que todos los componentes de un módulo son standalone, el módulo puede eliminarse.  

### 17.3.4. Estrategia Enterprise

En proyectos grandes:
- **Migración incremental**: no intentar eliminar todos los NgModules de golpe.  
- **Compatibilidad híbrida**: Angular 20 permite usar NgModules y standalone juntos.  
- **Automatización**: usar `ng g @angular/core:standalone` para generar componentes standalone.  
- **Testing continuo**: cada refactorización debe estar cubierta por pruebas unitarias e integradas.  
- **Documentación del proceso**: registrar qué módulos se han migrado y qué queda pendiente.  

### 17.3.5. Buenas prácticas

- **Usar standalone en todo código nuevo**.  
- **Evitar módulos “vacíos”** que solo reexportan.  
- **Centralizar providers** con `provide*` en `main.ts`.  
- **Mantener consistencia**: no mezclar patrones antiguos y nuevos en un mismo feature.  
- **Refactorizar en paralelo con otras mejoras** (ej. lazy loading, signals).  


## 17.4. Reemplazo de guards, resolvers e interceptors basados en clases por APIs funcionales

En Angular 17 todavía era habitual implementar **guards**, **resolvers** e **interceptors** como **clases** que implementaban interfaces (`CanActivate`, `Resolve`, `HttpInterceptor`).  
En Angular 20, el framework recomienda usar **APIs funcionales**: funciones puras que reemplazan a las clases, con menos boilerplate y mayor expresividad.  
Este cambio se alinea con la filosofía **standalone-first** y con el uso de **providers funcionales** (`provideRouter`, `provideHttpClient`).

### 17.4.1. Guards: de clases a funciones

#### Antes (Angular 17, basado en clases)
```ts
import { Injectable } from '@angular/core';
import { CanActivate } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(private auth: AuthService) {}

  canActivate(): boolean {
    return this.auth.isLoggedIn();
  }
}
```

#### Ahora (Angular 20, API funcional)
```ts
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  return auth.isLoggedIn();
};
```

**Ventajas**:
- Menos código y sin necesidad de `@Injectable`.  
- Uso directo de `inject()` para acceder a dependencias.  
- Más fácil de testear como función pura.  

### 17.4.2. Resolvers: de clases a funciones

#### Antes (Angular 17)
```ts
import { Injectable } from '@angular/core';
import { Resolve } from '@angular/router';
import { UserService } from './user.service';

@Injectable({ providedIn: 'root' })
export class UserResolver implements Resolve<any> {
  constructor(private userService: UserService) {}

  resolve() {
    return this.userService.getUser();
  }
}
```

#### Ahora (Angular 20)
```ts
import { inject } from '@angular/core';
import { ResolveFn } from '@angular/router';
import { UserService } from './user.service';

export const userResolver: ResolveFn<any> = () => {
  const userService = inject(UserService);
  return userService.getUser();
};
```

**Ventajas**:
- Sintaxis más clara y directa.  
- Menos boilerplate.  
- Integración natural con `provideRouter`.  

### 17.4.3. Interceptors: de clases a funciones

#### Antes (Angular 17)
```ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler } from '@angular/common/http';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const cloned = req.clone({ setHeaders: { Authorization: 'Bearer token' } });
    return next.handle(cloned);
  }
}
```

#### Ahora (Angular 20)
```ts
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const cloned = req.clone({ setHeaders: { Authorization: 'Bearer token' } });
  return next(cloned);
};
```

**Ventajas**:
- Definición más concisa.  
- No requiere `@Injectable`.  
- Se registra fácilmente con `provideHttpClient`.  

Ejemplo de registro:
```ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './auth.interceptor';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
});
```
// Se recomienda usar cookies seguras y signals para el estado de autenticación.

### 17.4.4. Estrategia de migración progresiva

1. **Identificar guards, resolvers e interceptors existentes**.  
2. **Convertir primero los más simples** a funciones.  
3. **Mantener compatibilidad híbrida**: Angular 20 soporta ambos enfoques.  
4. **Actualizar configuración de rutas y HttpClient** para usar las nuevas APIs (`provideRouter`, `provideHttpClient`).  
5. **Eliminar clases obsoletas** una vez migrado todo.  

### 17.4.5. Buenas prácticas

- Usar **APIs funcionales en todo código nuevo**.  
- Centralizar dependencias con `inject()` en lugar de constructores.  
- Mantener consistencia: no mezclar guardas de clase y funciones en un mismo módulo de rutas.  
- Escribir **tests unitarios** para cada guard/resolver/interceptor antes de migrar.  
- Documentar la migración para que todo el equipo adopte el nuevo estándar.  


## 17.5. Adaptación de formularios clásicos a Typed Forms y formularios con Signals

Los formularios han sido siempre una parte esencial de Angular. En versiones anteriores, trabajábamos con **Reactive Forms** y **Template-driven Forms**.  
Con Angular 14 llegaron los **Typed Forms**, que aportan tipado estático a los formularios reactivos.  
En Angular 20, además, podemos aprovechar **Signals** para gestionar el estado de los formularios de manera más declarativa y reactiva.  
Migrar desde formularios clásicos a estos nuevos enfoques nos permite ganar **seguridad de tipos, mejor DX (developer experience) y mayor control sobre la reactividad**.

### 17.5.1. De Reactive Forms clásicos a Typed Forms

#### Antes (Reactive Forms sin tipado estricto)
```ts
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

form: FormGroup;

constructor(private fb: FormBuilder) {
  this.form = this.fb.group({
    name: ['', Validators.required],
    age: [0, Validators.min(18)]
  });
}

submit() {
  const value = this.form.value; // value: any
  console.log(value.name.toUpperCase()); // riesgo: name podría ser undefined
}
```

#### Ahora (Typed Forms)
```ts
import { FormBuilder, Validators, NonNullableFormBuilder } from '@angular/forms';

form = this.fb.group({
  name: this.fb.control<string>('', { validators: [Validators.required] }),
  age: this.fb.control<number>(0, { validators: [Validators.min(18)] })
});

constructor(private fb: NonNullableFormBuilder) {}

submit() {
  const value = this.form.value; // value: { name: string; age: number }
  console.log(value.name.toUpperCase()); // seguro: name es string
}
```

**Ventajas**:
- El compilador detecta errores de tipo en tiempo de desarrollo.  
- Se eliminan los `any` implícitos.  
- Mejor autocompletado en editores.  

### 17.5.2. Formularios con Signals

Angular 20 permite combinar **Forms API** con **Signals**, lo que abre la puerta a un modelo de reactividad más natural.

#### Ejemplo con Signals
```ts
import { Component, signal, effect, input } from '@angular/core';
import { FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-profile-form',
  standalone: true,
  template: `
    <input [formControl]="nameControl" placeholder="Nombre" />
    @if (nameInvalid()) {
      <p>El nombre es obligatorio</p>
    }
  `
})
export class ProfileFormComponent {
  static readonly initialName = input<string>('');
  nameControl = new FormControl(ProfileFormComponent.initialName(), { validators: [Validators.required] });
  name = signal(this.nameControl.value);

  constructor() {
    // Sincronizar el FormControl con el Signal
    this.nameControl.valueChanges.subscribe(value => this.name.set(value ?? ''));

    // Efecto reactivo
    effect(() => {
      console.log('Nombre actualizado:', this.name());
    });
  }

  nameInvalid = () => this.nameControl.invalid && this.nameControl.touched;
}
```
// Se recomienda evitar formularios template-driven en Angular 20+.

### 17.5.3. Estrategia de migración progresiva

1. **Identificar formularios críticos**: empezar por los más pequeños o aislados.  
2. **Migrar primero a Typed Forms**: es un cambio seguro y compatible con Reactive Forms.  
3. **Introducir Signals en formularios nuevos**: aprovecharlos en componentes donde la reactividad sea clave.  
4. **Refactorizar validaciones**: mover lógica repetida a funciones reutilizables y tipadas.  
5. **Mantener compatibilidad híbrida**: Angular 20 permite usar formularios clásicos, Typed Forms y Signals en paralelo.  

### 17.5.4. Buenas prácticas

- Usar **Typed Forms por defecto** en proyectos nuevos.  
- Integrar **Signals** cuando el formulario interactúe con otros estados reactivos de la aplicación.  
- Evitar duplicar estado: si un valor ya está en un `FormControl`, sincronizarlo con un `signal` solo si es necesario.  
- Escribir **tests unitarios** para validaciones y lógica de formularios antes de migrar.  
- Documentar el patrón de formularios adoptado para mantener consistencia en el equipo.  


## 17.6. Uso progresivo de Zoneless Change Detection en lugar de Zone.js

Desde sus primeras versiones, Angular ha dependido de **Zone.js** para detectar automáticamente los cambios en la aplicación y actualizar la vista.  
Zone.js funciona “parcheando” APIs del navegador (eventos, timers, promesas) para saber cuándo ejecutar el **ciclo de detección de cambios**.  
Aunque este enfoque ha sido muy útil, también introduce **sobrecarga de rendimiento** y cierta **opacidad** en cómo y cuándo se actualiza la UI.  

Con Angular 20, el framework da un paso hacia un modelo **zoneless**, es decir, **sin Zone.js**, apoyándose en **Signals** y en un control más explícito de la reactividad.  

### 17.6.1. ¿Por qué migrar a zoneless?

- **Rendimiento**: menos overhead, ya que no se interceptan todas las APIs del navegador.  
- **Previsibilidad**: el desarrollador sabe exactamente qué dispara la detección de cambios.  
- **Integración con Signals**: los Signals notifican cambios de forma explícita, eliminando la necesidad de Zone.js.  
- **Menor complejidad**: menos dependencias y menos “magia” en el runtime.  

### 17.6.2. Activando zoneless en Angular 20

En Angular 20 podemos arrancar la aplicación sin Zone.js:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  // Desactiva Zone.js
  ngZone: 'noop'
});
```
// Nota: Usar ngZone: 'noop' requiere que todo el estado reactivo esté gestionado por signals o APIs explícitas.

Esto indica a Angular que no use Zone.js para el ciclo de detección de cambios.  
A partir de aquí, la reactividad depende de **Signals** y de APIs explícitas como `ChangeDetectorRef`.

### 17.6.3. Ejemplo práctico

#### Con Zone.js (clásico)
```ts
@Component({
  selector: 'app-counter',
  template: `
    <p>{{ count }}</p>
    <button (click)="increment()">Incrementar</button>
  `
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++;
    // Zone.js detecta el evento y actualiza la vista automáticamente
  }
}
```

#### Zoneless con Signals
```ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>{{ count() }}</p>
    <button (click)="increment()">Incrementar</button>
  `
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update(v => v + 1);
    // No necesitamos Zone.js: el Signal notifica el cambio
  }
}
```

### 17.6.4. Estrategia de migración progresiva

1. **Identificar componentes candidatos**: empezar por aquellos que ya usan Signals o que tienen lógica de estado local.  
2. **Arrancar la app en modo zoneless en entornos de prueba** (`ngZone: 'noop'`).  
3. **Refactorizar paso a paso**:  
   - Sustituir propiedades simples por Signals.  
   - Reemplazar `async pipe` en observables simples por Signals derivados.  
   - Usar `effect()` para sincronizar lógica reactiva.  
4. **Mantener compatibilidad híbrida**: Angular permite seguir usando Zone.js mientras migramos gradualmente.  
5. **Medir rendimiento**: validar mejoras en tiempo de renderizado y consumo de CPU.  

### 17.6.5. Buenas prácticas

- **Usar Signals como primera opción** para estado local y bindings.  
- **Reservar RxJS** para flujos complejos o asincronía avanzada.  
- **Evitar llamadas manuales a `detectChanges()`** salvo en casos muy específicos.  
- **Documentar el patrón zoneless** para que todo el equipo lo adopte de forma consistente.  
- **Probar exhaustivamente**: la migración puede revelar dependencias ocultas de Zone.js.  


## 17.7. Migración hacia `NgOptimizedImage` desde librerías de terceros

En aplicaciones Angular anteriores a la versión 15, era común recurrir a librerías externas (ej. `ngx-image`, `ng-lazyload-image`, directivas personalizadas) para optimizar imágenes: lazy loading, tamaños adaptativos, placeholders, etc.  
Con Angular 15 se introdujo **`NgOptimizedImage`**, y en Angular 20 esta API se consolida como la **forma recomendada y nativa** de gestionar imágenes optimizadas.  
Migrar hacia `NgOptimizedImage` nos permite **reducir dependencias externas**, mejorar el **rendimiento** y aprovechar la **integración directa con el compilador y el ecosistema Angular**.

### 17.7.1. ¿Qué aporta `NgOptimizedImage`?

- **Lazy loading automático** (`loading="lazy"`).  
- **Soporte para `srcset` y `sizes`**: imágenes responsivas según el viewport.  
- **Preconexión y prefetch**: mejora el rendimiento percibido.  
- **Validaciones en tiempo de compilación**: Angular avisa si falta un atributo crítico.  
- **Integración con CDNs**: fácil de usar con servicios como Cloudinary, Imgix o Akamai.  

### 17.7.2. Ejemplo de uso básico

#### Antes (con librerías externas)
```html
<img
  [defaultImage]="'/assets/placeholder.png'"
  [lazyLoad]="'/assets/hero.jpg'"
  [errorImage]="'/assets/error.png'"
  alt="Imagen hero"
/>
```

#### Ahora (con `NgOptimizedImage`)
```html
<img
  ngSrc="/assets/hero.jpg"
  width="1200"
  height="600"
  priority
  alt="Imagen hero"
/>
```

**Notas**:
- `ngSrc` sustituye a `src`.  
- `width` y `height` son obligatorios para evitar *layout shifts*.  
- `priority` indica que la imagen debe cargarse lo antes posible (ej. hero banners).  

### 17.7.3. Ejemplo con imágenes responsivas

```html
<img
  ngSrc="/assets/product.jpg"
  width="800"
  height="600"
  alt="Producto"
  [ngSrcset]="{
    '/assets/product-small.jpg': '480w',
    '/assets/product-medium.jpg': '800w',
    '/assets/product-large.jpg': '1200w'
  }"
  sizes="(max-width: 600px) 480px, 800px"
/>
```

Esto permite que el navegador elija automáticamente la mejor versión según el dispositivo.

### 17.7.4. Estrategia de migración progresiva

1. **Auditoría de librerías de imágenes**: identificar dónde se usan directivas externas (`lazyLoad`, `defaultImage`, etc.).  
2. **Migrar primero imágenes críticas**: banners, portadas, imágenes de productos.  
3. **Sustituir atributos personalizados** por `ngSrc`, `ngSrcset`, `priority`.  
4. **Eliminar dependencias externas** una vez migradas todas las imágenes.  
5. **Validar con Lighthouse**: comprobar mejoras en métricas como LCP (Largest Contentful Paint).  

### 17.7.5. Buenas prácticas

- Siempre definir **width y height** para evitar *cumulative layout shift*.  
- Usar `priority` solo en imágenes críticas (ej. hero, logo principal).  
- Aprovechar `ngSrcset` para dispositivos móviles y pantallas retina.  
- Integrar con un **CDN de imágenes** para redimensionado y compresión automática.  
- Revisar logs de compilación: Angular avisa si falta algún atributo obligatorio.  


## 17.8. Ajustes en proyectos con SSR: introducción del Hydration incremental

El **Server-Side Rendering (SSR)** en Angular ha evolucionado de forma significativa.  
Hasta Angular 17, el SSR dependía de Angular Universal y el proceso de **hydration** (la fase en la que el cliente “toma el control” del HTML renderizado en el servidor) era **global**: toda la aplicación debía hidratarse de una sola vez.  
En Angular 20, se introduce el **Hydration incremental**, un modelo más flexible y eficiente que permite **hidratar la aplicación por partes**, optimizando el rendimiento y reduciendo el tiempo hasta la interactividad.

### 17.8.1. ¿Qué es el Hydration incremental?

- **Hydration clásico**: el navegador recibe el HTML renderizado en el servidor y Angular lo “rehidrata” completo, adjuntando listeners y reconstruyendo el árbol de componentes.  
- **Hydration incremental**: Angular permite hidratar **secciones específicas** de la aplicación de forma progresiva, en lugar de hacerlo todo de golpe.  

**Beneficios**:
- Mejor **Time to Interactive (TTI)**: la aplicación responde más rápido.  
- Menor consumo de memoria en dispositivos móviles.  
- Posibilidad de priorizar la hidratación de componentes críticos (ej. cabecera, menú, formulario inicial).  

### 17.8.2. Activando el Hydration en Angular 20

#### Configuración básica
En `main.ts`:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { provideClientHydration } from '@angular/platform-browser';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration()
  ]
});
```

Esto habilita el **hydration estándar** en toda la aplicación.

### 17.8.3. Hydration incremental

Angular 20 introduce la posibilidad de marcar **componentes o secciones** para hidratación diferida o progresiva.

Ejemplo de componente con hidratación diferida:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-heavy-widget',
  standalone: true,
  template: `
    <section>
      <h2>Estadísticas avanzadas</h2>
      <!-- Este bloque se hidratará de forma diferida -->
    </section>
  `,
  hydration: {
    strategy: 'defer' // otras opciones: 'immediate', 'on-interaction'
  }
})
export class HeavyWidgetComponent {}
```

#### Estrategias disponibles
- **`immediate`**: hidrata el componente tan pronto como se carga.  
- **`defer`**: pospone la hidratación hasta que el navegador esté libre.  
- **`on-interaction`**: hidrata solo cuando el usuario interactúa (ej. scroll, click).  

### 17.8.4. Estrategia de migración progresiva

1. **Auditoría de componentes SSR**: identificar cuáles son críticos para la primera interacción.  
2. **Hidratar inmediatamente** cabecera, menú y formularios iniciales.  
3. **Diferir widgets pesados** (gráficas, dashboards, componentes secundarios).  
4. **Medir métricas de rendimiento** con Lighthouse o Web Vitals.  
5. **Iterar**: ajustar estrategias de hidratación según el comportamiento real de usuarios.  

### 17.8.5. Buenas prácticas

- Usar **hydration incremental en componentes no críticos** para mejorar TTI.  
- Evitar hidratar en diferido elementos interactivos esenciales (botones de login, navegación).  
- Combinar con **lazy loading** de rutas y módulos.  
- Monitorizar métricas como **LCP** y **INP** para validar mejoras.  
- Documentar qué componentes usan qué estrategia de hidratación.  


## 17.9. Sustitución de test runners obsoletos por Jest y Cypress

Durante años, Angular venía configurado por defecto con **Karma + Jasmine** como entorno de pruebas.  
Aunque fueron herramientas muy útiles en su momento, hoy se consideran **obsoletas** para proyectos modernos: lentas, difíciles de configurar en pipelines CI/CD y con poca integración con el ecosistema actual.  

En Angular 20, la tendencia es clara: **migrar a Jest para pruebas unitarias** y **Cypress para pruebas end-to-end (E2E)**.  
Ambas herramientas ofrecen mayor velocidad, mejor DX (developer experience) y una comunidad activa que asegura soporte a largo plazo.

### 17.9.1. Limitaciones de Karma/Jasmine

- Ejecución lenta en navegadores reales.  
- Configuración compleja para CI/CD.  
- Ecosistema estancado, con pocas actualizaciones recientes.  
- Difícil integración con pruebas modernas (a11y, snapshots, mocks avanzados).  

### 17.9.2. Jest para pruebas unitarias

**Ventajas de Jest**:
- Corre en **Node.js**, sin necesidad de navegador real.  
- **Rápido y paralelo**: ejecuta tests en múltiples workers.  
- **Snapshots**: permite validar salidas de componentes y funciones.  
- **Mocks integrados**: no requiere librerías externas para simular dependencias.  
- Excelente integración con **Typed Forms, Signals y APIs modernas de Angular**.  

#### Ejemplo de test con Jest
```ts
import { sum } from './math';

describe('sum', () => {
  it('debería sumar dos números', () => {
    expect(sum(2, 3)).toBe(5);
  });
});
```

#### Configuración básica
```bash
npm install --save-dev jest jest-preset-angular @types/jest
```

En `package.json`:
```json
"scripts": {
  "test": "jest"
}
```

### 17.9.3. Cypress para pruebas E2E

**Ventajas de Cypress**:
- Corre en un navegador real, pero con un **entorno de testing controlado**.  
- API sencilla y legible, pensada para simular la experiencia del usuario.  
- **Time-travel debugging**: podemos ver paso a paso qué ocurrió en cada test.  
- Integración con **axe-core** para pruebas de accesibilidad.  
- Compatible con pipelines CI/CD y dashboards de monitoreo.  

#### Ejemplo de test con Cypress
```js
describe('Página de login', () => {
  it('debería permitir iniciar sesión con credenciales válidas', () => {
    cy.visit('/login');
    cy.get('input[name="email"]').type('user@test.com');
    cy.get('input[name="password"]').type('123456');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

### 17.9.4. Estrategia de migración progresiva

1. **Mantener Karma/Jasmine temporalmente** mientras se introduce Jest en paralelo.  
2. **Migrar pruebas unitarias simples** (pipes, servicios).  
3. **Configurar Jest en CI/CD** y validar que los tiempos de ejecución mejoran.  
4. **Reemplazar Protractor (si aún existe) por Cypress** en pruebas E2E.  
5. **Eliminar Karma/Jasmine** una vez migradas todas las pruebas.  

### 17.9.5. Buenas prácticas
- Usar **Jest para unit tests** y **Cypress para E2E**, no mezclar roles.  
- Mantener **tests rápidos y aislados** en Jest; reservar Cypress para flujos críticos.  
- Integrar **axe-core** en Cypress para validar accesibilidad automáticamente.  
- Ejecutar Jest en cada commit y Cypress en cada PR o nightly build.  
- Documentar la estrategia de testing para que todo el equipo la adopte.  
- Integrar los tests de accesibilidad en el pipeline de CI/CD.


## 17.10. Herramientas de apoyo para la migración: Angular CLI, Angular Update Guide y checklists

Migrar de Angular 17 a Angular 20 no es solo cuestión de cambiar versiones en `package.json`.  
La clave está en **apoyarse en las herramientas oficiales y en procesos sistemáticos** que reduzcan riesgos y aseguren que el equipo avanza con confianza.  
En esta sección veremos tres pilares fundamentales: **Angular CLI**, la **Angular Update Guide** y los **checklists de migración**.

### 17.10.1. Angular CLI como asistente de migración

El **Angular CLI** no es solo un generador de proyectos: es también un **motor de migraciones automáticas**.  
Cada vez que actualizamos Angular, el CLI ejecuta **schematics** que adaptan el código a las nuevas APIs.

#### Ejemplo de actualización con CLI
```bash
ng update @angular/core@20 @angular/cli@20
```

Esto:
- Actualiza dependencias en `package.json`.  
- Ejecuta migraciones automáticas (ej. convertir configuraciones obsoletas, ajustar imports).  
- Muestra advertencias si hay APIs deprecadas.  

**Ventajas**:
- Reduce errores manuales.  
- Garantiza que el proyecto queda alineado con las prácticas recomendadas.  
- Compatible con monorepos (Nx, Lerna).  

### 17.10.2. Angular Update Guide

La **Angular Update Guide** (https://update.angular.io) es la referencia oficial para planificar migraciones.  
Permite seleccionar:
- **Versión actual** y **versión destino**.  
- **Complejidad del proyecto** (básico, intermedio, avanzado).  
- **Tipo de aplicación** (app, librería, monorepo).  

Y genera un **plan detallado** con:
- Cambios automáticos que hará el CLI.  
- Cambios manuales que debe aplicar el equipo.  
- Notas sobre APIs deprecadas y alternativas recomendadas.  

**Ejemplo**:  
Migrar de Angular 17 a 20 puede incluir pasos como:
- Sustituir `*ngIf`/`*ngFor` por `@if`/`@for`.  
- Migrar guards de clase a funciones.  
- Ajustar SSR para usar Hydration incremental.  


### 17.10.3. Checklists de migración
Un **checklist** es la forma más práctica de asegurar que nada se queda atrás.  
En proyectos Enterprise, donde participan múltiples equipos, los checklists permiten **coordinar y auditar** el avance de la migración.

#### Ejemplo de checklist de migración Angular 17 → 20
- [ ] Actualizar Angular CLI y Core con `ng update`.  
- [ ] Revisar dependencias externas y su compatibilidad.  
- [ ] Migrar control flow (`*ngIf` → `@if`, `*ngFor` → `@for`).  
- [ ] Sustituir NgModules por Standalone Components.  
- [ ] Migrar guards/resolvers/interceptors a APIs funcionales.  
- [ ] Revisar formularios y adoptar Typed Forms/Signals.  
- [ ] Activar zoneless change detection en entornos de prueba.  
- [ ] Migrar imágenes a `NgOptimizedImage`.  
- [ ] Ajustar SSR con Hydration incremental.  
- [ ] Sustituir Karma/Jasmine por Jest y Cypress.  
- [ ] Configurar monitorización y logging en producción.  
- [ ] Usar linters y reglas de ESLint para detectar patrones obsoletos.  
- [ ] Documentar la migración y versionar con SemVer.  
- [ ] Integrar tests de accesibilidad en CI/CD.


## 17.11. Estrategia Enterprise para la migración a Angular 20

La migración a Angular 20 puede parecer una tarea monumental, especialmente en aplicaciones grandes y complejas.  
Sin embargo, con una planificación cuidadosa y un enfoque sistemático, es posible realizar la migración de manera eficiente y con un riesgo mínimo.  
En esta sección, presentamos una estrategia recomendada para llevar a cabo la migración en un entorno empresarial.

### 17.11.1. Fases de la migración

1. **Preparación**:
   - Actualizar a la última versión de Angular 17.x.
   - Revisar y actualizar todas las dependencias externas.
   - Realizar una copia de seguridad completa del proyecto.

2. **Capacitación y familiarización**:
   - Capacitar al equipo en las nuevas características y cambios de Angular 20.
   - Revisar la documentación oficial de migración y las notas de la versión.

3. **Migración inicial**:
   - Usar Angular CLI para realizar una migración automática de las dependencias.
   - Resolver manualmente cualquier conflicto o advertencia que surja durante la migración.

4. **Refactorización de código**:
   - Reemplazar patrones obsoletos (ej. `*ngIf`, `*ngFor`) por las nuevas sintaxis (`@if`, `@for`).
   - Migrar componentes y módulos a la nueva arquitectura standalone.
   - Actualizar guards, resolvers e interceptors a las nuevas APIs funcionales.

5. **Optimización de formularios**:
   - Convertir formularios reactivos a Typed Forms.
   - Integrar Signals en la gestión del estado de los formularios.

6. **Ajustes de rendimiento**:
   - Activar el modo zoneless y ajustar el código para asegurar la compatibilidad.
   - Migrar imágenes y otros activos multimedia a las nuevas APIs optimizadas.

7. **Pruebas y validación**:
   - Ejecutar pruebas unitarias y de integración para validar el funcionamiento de la aplicación.
   - Realizar pruebas de rendimiento y ajustar según sea necesario.

8. **Despliegue y monitoreo**:
   - Desplegar la aplicación migrada en un entorno de staging.
   - Monitorear el rendimiento y los errores en tiempo real.
   - Realizar ajustes finales antes del despliegue en producción.

### 17.11.2. Buenas prácticas para la migración en Enterprise

- **Planificación detallada**: dedicar tiempo a planificar cada fase de la migración, identificando riesgos y dependencias.
- **Pruebas exhaustivas**: asegurar que todas las funcionalidades críticas están cubiertas por pruebas automatizadas antes de iniciar la migración.
- **Capacitación continua**: proporcionar recursos y tiempo para que el equipo se familiarice con las nuevas herramientas y técnicas.
- **Revisiones de código**: implementar revisiones de código regulares para asegurar que se siguen las mejores prácticas y se evita la introducción de nuevos problemas.
- **Despliegue gradual**: considerar un despliegue gradual de la migración, comenzando por módulos o características menos críticas.
- **Documentación clara**: mantener una documentación clara y actualizada sobre el estado de la migración, decisiones tomadas y próximos pasos.

### 17.11.3. Herramientas recomendadas

- **Angular CLI**: para la gestión de la migración y actualización de dependencias.
- **Jest**: para pruebas unitarias rápidas y eficientes.
- **Cypress**: para pruebas end-to-end y validación de flujos críticos.
- **ESLint**: para la detección de patrones obsoletos y aseguramiento de la calidad del código.
- **Lighthouse**: para auditorías de rendimiento y accesibilidad.

### 17.11.4. Conclusión

La migración a Angular 20 es una oportunidad para modernizar y optimizar aplicaciones, aprovechando las últimas mejoras en el framework.  
Si bien el proceso de migración puede parecer desalentador, con una planificación cuidadosa, capacitación y un enfoque sistemático, es posible realizar la migración de manera efectiva y con un riesgo mínimo.  
Anime a su equipo a adoptar las nuevas características y mejoras, y aproveche al máximo lo que Angular 20 tiene para ofrecer.

