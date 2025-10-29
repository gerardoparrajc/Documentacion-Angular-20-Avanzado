# 6. Inyección de dependencias y Standalone APIs

## 6.1. Introducción a los Standalone APIs y la inyección moderna en Angular 20

Uno de los cambios más profundos en la evolución de Angular ha sido la simplificación de su modelo de arquitectura. Durante años, los **NgModules** fueron el corazón del framework: todo componente, directiva o pipe debía declararse en un módulo, y los servicios se registraban en sus `providers`. Esto funcionaba, pero añadía una capa de complejidad que muchas veces no aportaba valor.  

Con la llegada de Angular 14 se introdujeron los **Standalone Components**, y a partir de Angular 19 esta idea se consolidó: **todos los artefactos son standalone por defecto**. Esto significa que ya no necesitas marcar explícitamente `standalone: true` en los decoradores. En Angular 20, este modelo es la norma, y los NgModules han pasado a ser opcionales y, en la práctica, prescindibles en la mayoría de los proyectos.

### 6.1.1. ¿Qué significa que todo sea *standalone*?

Un componente, directiva o pipe puede existir y usarse sin necesidad de estar declarado en un módulo. Basta con importarlo allí donde lo necesites.  

Ejemplo de un componente standalone en Angular 20:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>Hola desde Angular 20</h1>`
})
export class HelloComponent {}
```

Este componente puede usarse directamente en una ruta o importarse en otro componente sin necesidad de un NgModule. Esto simplifica la arquitectura y hace que el código sea más directo y fácil de mantener.

### 6.1.2. La inyección de dependencias en Angular 20

La **inyección de dependencias (DI)** sigue siendo uno de los pilares del framework. Angular mantiene un **inyector jerárquico** que se encarga de entregar instancias de servicios allí donde se necesiten.  

Lo que ha cambiado es la forma de trabajar con esa inyección:  
- Los servicios pueden declararse con `providedIn: 'root'` para estar disponibles en toda la aplicación, o limitarse a un componente o ruta concreta.  
- Ya no dependemos de NgModules para registrar servicios.  
- Podemos usar la función `inject()` como alternativa moderna a la inyección por constructor.  

### 6.1.3. `inject()`: la forma moderna de acceder a dependencias

La función `inject()` permite obtener una instancia de un servicio sin necesidad de declararlo en el constructor. Sin embargo, es importante entender que **solo puede usarse dentro de un contexto de inyección**.  

Según la documentación oficial, los contextos válidos son:  
- El constructor de un componente, directiva, pipe o servicio.  
- La inicialización de propiedades en esas clases.  
- Una función de fábrica (`useFactory`) de un provider o un `InjectionToken`.  
- Funciones que Angular ejecuta en un contexto de inyección, como guards de router (`CanActivateFn`, `ResolveFn`).  
- Bloques ejecutados explícitamente con `runInInjectionContext()`.  

#### Ejemplo válido en un componente

```ts
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-profile',
  template: `<p>Usuario: {{ user.name }}</p>`
})
export class ProfileComponent {
  private userService = inject(UserService); // ✅ válido en inicialización de propiedad
  user = this.userService.getUser();
}
```

#### Ejemplo válido en un guard de router

```ts
import { CanActivateFn } from '@angular/router';
import { AuthService } from './auth.service';
import { inject } from '@angular/core';

export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService); // ✅ válido en un guard
  return auth.isLoggedIn();
};
```

#### Ejemplo inválido

```ts
// ❌ Esto lanzará NG0203 porque no hay contexto de inyección
const userService = inject(UserService);

export function helper() {
  return userService.getUser();
}
```

Si necesitas usar `inject()` fuera de un contexto natural, puedes envolver la llamada con `runInInjectionContext()` y pasarle un inyector (por ejemplo, el `EnvironmentInjector`).

### 6.1.4. Inyección en rutas y *scoped providers*

Otra novedad poderosa es que ahora podemos **definir servicios directamente en las rutas**. Esto permite que un servicio exista solo mientras esa ruta esté activa, reduciendo consumo de memoria y mejorando el rendimiento.

```ts
import { Routes } from '@angular/router';
import { DashboardComponent } from './dashboard.component';
import { DashboardService } from './dashboard.service';

export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    providers: [DashboardService] // servicio disponible solo en esta ruta
  }
];
```

### 6.1.5. Beneficios del modelo moderno

1. **Menos complejidad**: ya no necesitamos NgModules para organizar dependencias.  
2. **Mayor flexibilidad**: podemos decidir el alcance de un servicio (global, por componente, por ruta).  
3. **Mejor rendimiento**: los servicios se crean solo cuando son necesarios.  
4. **Código más limpio**: `inject()` evita constructores largos y repetitivos, siempre que se use en un contexto válido.  


## 6.2. Providers funcionales vs providers clásicos: diferencias y casos de uso

La **inyección de dependencias** en Angular siempre ha sido muy flexible. Desde las primeras versiones, hemos podido declarar servicios y decidir cómo se crean, qué alcance tienen y cómo se sustituyen por implementaciones alternativas.  

En Angular 20, además de los **providers clásicos**, contamos con los **providers funcionales**, una forma más moderna y declarativa de configurar la inyección. Entender ambos enfoques es clave para equipos que migran proyectos o que quieren aprovechar las ventajas de la nueva sintaxis.


### 6.2.1. Providers clásicos

Los providers clásicos son los que se han usado desde siempre en Angular. Se configuran mediante objetos que indican **qué token se provee** y **cómo se resuelve**.  

Ejemplos típicos:

```ts
import { NgModule } from '@angular/core';
import { LoggerService } from './logger.service';

// Ejemplo clásico en un módulo o componente
@NgModule({
  providers: [
    LoggerService, // shorthand de { provide: LoggerService, useClass: LoggerService }
    { provide: 'API_URL', useValue: 'https://api.midominio.com' },
    { provide: LoggerService, useClass: BetterLoggerService },
    { provide: LoggerService, useExisting: OtherLoggerService },
    { provide: LoggerService, useFactory: () => new LoggerService(true) }
  ]
})
export class AppModule {}
```

Características:  
- Se basan en objetos de configuración (`provide`, `useClass`, `useValue`, `useFactory`, `useExisting`).  
- Son muy expresivos y permiten sustituir implementaciones fácilmente.  
- Han sido la base de Angular desde sus inicios.  

### 6.2.2. Providers funcionales

Con Angular moderno, especialmente desde Angular 15 en adelante, se introdujo una nueva forma de declarar providers: **los providers funcionales**.  

En lugar de objetos, se usan **funciones auxiliares** que simplifican la sintaxis y mejoran la legibilidad.  

Ejemplo:

```ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { importProvidersFrom } from '@angular/core';

export const appConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor])
    ),
    provideRouter(routes),
  ]
};
```

Características:  
- Usan funciones como `provideHttpClient()`, `provideRouter()`, `provideAnimations()`.  
- Son más declarativos y fáciles de leer.  
- Permiten configurar dependencias complejas con menos código repetitivo.  
- Están pensados para el mundo **standalone**, donde ya no dependemos de NgModules.  

### 6.2.3. Diferencias clave

| Aspecto | Providers clásicos | Providers funcionales |
|---------|-------------------|------------------------|
| **Sintaxis** | Basada en objetos `{ provide, useClass, useValue, useFactory }` | Basada en funciones (`provideX()`) |
| **Legibilidad** | Verbosa en configuraciones complejas | Más concisa y declarativa |
| **Compatibilidad** | Disponible desde Angular 2 | Introducidos en Angular 15, consolidados en Angular 20 |
| **Uso típico** | Sustitución de clases, valores estáticos, factories personalizadas | Configuración de servicios del framework (HTTP, Router, Animations, i18n) |
| **Migración** | Sigue siendo válido y soportado | Recomendado para nuevos proyectos standalone |

### 6.2.4. Casos de uso

#### Cuándo usar providers clásicos
- Cuando necesitas **control fino** sobre cómo se crea un servicio.  
- Para **aliasing** (`useExisting`) o **factories personalizadas**.  
- En proyectos legacy que aún usan NgModules.  

Ejemplo:

```ts
{ provide: LoggerService, useFactory: () => new LoggerService(true) }
```

#### Cuándo usar providers funcionales
- En **aplicaciones nuevas standalone**.  
- Para configurar **servicios del framework** (HTTP, Router, Animations, i18n).  
- Cuando quieres **menos boilerplate** y una sintaxis más clara.  

Ejemplo:

```ts
provideHttpClient(withInterceptors([authInterceptor]))
```


## 6.3. Jerarquía avanzada de inyección y modificadores (`@Optional`, `@Self`, `@SkipSelf`)

La **inyección de dependencias (DI)** en Angular se basa en un sistema jerárquico de inyectores. Esto significa que cuando un componente solicita un servicio, Angular no busca únicamente en un único lugar, sino que recorre una **cadena de inyectores** hasta encontrar una coincidencia.  

Comprender esta jerarquía es fundamental para aplicaciones grandes, donde distintos componentes pueden necesitar **instancias distintas** de un mismo servicio o, por el contrario, compartir una única instancia global.

### 6.3.1. La jerarquía de inyectores en Angular

En Angular existen principalmente dos niveles de inyectores:

- **EnvironmentInjector (nivel de aplicación)**: es el inyector raíz, creado al arrancar la aplicación. Los servicios con `providedIn: 'root'` viven aquí y son compartidos en toda la app.  
- **ElementInjector (nivel de componente/directiva)**: cada componente/directiva puede tener su propio inyector local, definido en su propiedad `providers`.  

Cuando Angular necesita resolver una dependencia:  
1. Busca primero en el **inyector local** del componente.  
2. Si no la encuentra, sube en la jerarquía hacia el padre.  
3. Si llega al inyector raíz y tampoco está, lanza un error `NullInjectorError`.  

### 6.3.2. Modificadores de inyección

Angular nos da decoradores especiales para **alterar este comportamiento por defecto**.  

#### 🔹 `@Self()`

Indica que la dependencia debe resolverse **únicamente en el inyector local** del componente/directiva.  
- Si no existe allí, Angular lanza un error.  
- Útil cuando queremos asegurarnos de que un servicio se provea de forma explícita en ese componente.  

```ts
constructor(@Self() private logger: LoggerService) {}
```

👉 Aquí, si `LoggerService` no está en los `providers` del componente, fallará aunque exista en el inyector raíz.

#### 🔹 `@SkipSelf()`

Indica que Angular debe **saltar el inyector local** y empezar la búsqueda en el padre.  
- Útil cuando queremos evitar una redefinición local y usar la instancia de un nivel superior.  

```ts
constructor(@SkipSelf() private logger: LoggerService) {}
```

👉 Aquí, aunque el componente tenga su propio `LoggerService` en `providers`, Angular usará el del padre.

#### 🔹 `@Optional()`

Indica que la dependencia es **opcional**.  
- Si Angular no encuentra el servicio en la jerarquía, en lugar de lanzar un error, inyecta `null`.  
- Muy útil para dependencias que pueden o no estar presentes.  

```ts
constructor(@Optional() private analytics?: AnalyticsService) {}
```

👉 Aquí, si `AnalyticsService` no está registrado en ningún inyector, la propiedad será `undefined` y el componente seguirá funcionando.

### 6.3.3. Ejemplo práctico combinando modificadores

Imaginemos un escenario con un componente padre y un hijo, y un servicio `LoggerService`:

```ts
@Component({
  selector: 'app-parent',
  template: `<app-child></app-child>`,
  providers: [LoggerService] // Proveedor en el padre
})
export class ParentComponent {}

@Component({
  selector: 'app-child',
  template: `<p>Child works!</p>`,
  providers: [LoggerService] // Proveedor en el hijo
})
export class ChildComponent {
  constructor(
    @Self() private localLogger: LoggerService,
    @SkipSelf() private parentLogger: LoggerService,
    @Optional() private maybeAnalytics?: AnalyticsService
  ) {
    console.log('Local logger:', this.localLogger);
    console.log('Parent logger:', this.parentLogger);
    console.log('Analytics:', this.maybeAnalytics);
  }
}
```

- `@Self()` → obtiene el `LoggerService` definido en el hijo.  
- `@SkipSelf()` → ignora el del hijo y obtiene el del padre.  
- `@Optional()` → si `AnalyticsService` no existe en ningún inyector, no rompe la app.  

### 6.3.4. Casos de uso reales

- **`@Self()`**: cuando queremos garantizar que un servicio se provea de forma explícita en un componente (ej. un `FormControlDirective` que necesita su propio `NgControl`).  
- **`@SkipSelf()`**: útil en directivas que se comunican con un servicio del componente padre (ej. `ControlContainer` en formularios reactivos).  
- **`@Optional()`**: ideal para servicios que son opcionales, como un `LoggerService` que solo se inyecta en entornos de desarrollo.  


## 6.4. Uso de `InjectionToken` para inyecciones condicionales y seguras

En Angular, la inyección de dependencias se basa en **tokens**: identificadores que el inyector utiliza para saber qué instancia debe entregar. Cuando inyectamos una clase (por ejemplo, `UserService`), la propia clase actúa como token.  

Pero ¿qué ocurre si queremos inyectar algo que **no es una clase**? Por ejemplo:  
- Una **configuración** (un objeto con parámetros).  
- Un **valor primitivo** (string, number, boolean).  
- Una **interfaz** de TypeScript (que no existe en tiempo de ejecución).  

En estos casos, necesitamos un **`InjectionToken`**: un objeto especial que Angular puede usar como identificador único y seguro.

### 6.4.1. Creación de un `InjectionToken`

Un `InjectionToken` se crea con el constructor `new InjectionToken<T>()`, donde `T` es el tipo de dato que queremos inyectar.

```ts
import { InjectionToken } from '@angular/core';

export interface AppConfig {
  apiUrl: string;
  featureFlag: boolean;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');
```

👉 Aquí hemos creado un token llamado `APP_CONFIG` que representa un objeto de configuración de tipo `AppConfig`.

### 6.4.2. Proveer un valor con un `InjectionToken`

Podemos asociar un valor al token en los `providers` de la aplicación:

```ts
import { APP_CONFIG } from './app.config';

export const appConfig: AppConfig = {
  apiUrl: 'https://api.midominio.com',
  featureFlag: true
};

export const appProviders = [
  { provide: APP_CONFIG, useValue: appConfig }
];
```

### 6.4.3. Inyectar el valor en un componente o servicio

```ts
import { Component, inject } from '@angular/core';
import { APP_CONFIG, AppConfig } from './app.config';

@Component({
  selector: 'app-root',
  template: `<p>API: {{ config.apiUrl }}</p>`
})
export class AppComponent {
  config = inject(APP_CONFIG); // ✅ inyección segura y tipada
}
```

👉 Angular garantiza que el valor inyectado corresponde al tipo `AppConfig`, lo que evita errores en tiempo de compilación.

### 6.4.4. `InjectionToken` con factorías y lógica condicional

Un `InjectionToken` también puede definirse con una **factoría**, lo que permite crear el valor dinámicamente e incluso inyectar otros servicios dentro de esa factoría.

```ts
import { InjectionToken, inject } from '@angular/core';
import { EnvironmentService } from './environment.service';

export const API_URL = new InjectionToken<string>('api.url', {
  providedIn: 'root',
  factory: () => {
    const env = inject(EnvironmentService);
    return env.isProd ? 'https://api.prod.com' : 'https://api.dev.com';
  }
});
```

👉 Aquí el valor del token `API_URL` depende de si estamos en producción o en desarrollo.  
Esto nos da **inyecciones condicionales** sin necesidad de escribir lógica repetida en cada componente.

### 6.4.5. Ejemplo práctico: configuración multi-entorno

Supongamos que tenemos distintos endpoints según el entorno:

```ts
export const FEATURE_FLAG = new InjectionToken<boolean>('feature.flag', {
  providedIn: 'root',
  factory: () => {
    return window.location.hostname.includes('staging');
  }
});
```

En un componente:

```ts
@Component({
  selector: 'app-feature',
  template: `
    @if (featureFlag) {
      <p>Funcionalidad beta activada 🚀</p>
    }
  `
})
export class FeatureComponent {
  featureFlag = inject(FEATURE_FLAG);
}
```

👉 De esta forma, el componente se adapta automáticamente al entorno sin necesidad de condicionales dispersos en el código.

### 6.4.6. Buenas prácticas con `InjectionToken`

- **Usa `InjectionToken` para interfaces y valores primitivos**: nunca intentes inyectar directamente un string o un número, ya que no son únicos.  
- **Centraliza la configuración**: define tokens para parámetros globales (API URLs, flags, claves de terceros).  
- **Aprovecha las factorías**: permiten lógica condicional y valores calculados en tiempo de ejecución.  
- **Evita duplicados**: recuerda que cada `new InjectionToken()` crea un identificador único; si lo defines en varios archivos, Angular los tratará como tokens distintos.  


## 6.5. Array Providers y View Providers en escenarios complejos

En Angular, los **providers** son la base del sistema de inyección de dependencias: le indican al framework cómo crear y entregar instancias de servicios. Hasta aquí, todo claro. Pero en aplicaciones enterprise, con jerarquías de componentes complejas y servicios que deben comportarse de manera distinta según el contexto, necesitamos herramientas más avanzadas.  

Dos de esas herramientas son los **Array Providers** y los **View Providers**. Ambos permiten un control más fino sobre qué servicios se inyectan y dónde, resolviendo problemas que aparecen en escenarios reales de gran escala.

### 6.5.1. Array Providers

Un **Array Provider** es una forma de declarar múltiples implementaciones para un mismo token de inyección. En lugar de sobrescribir el valor anterior, Angular acumula todas las instancias en un **array** y las inyecta juntas.

Esto se logra con la propiedad `multi: true`.

### Ejemplo: múltiples interceptores de HTTP

```ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthInterceptor } from './auth.interceptor';
import { LoggingInterceptor } from './logging.interceptor';

export const appProviders = [
  { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
  { provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true }
];
```

👉 Aquí, cuando Angular resuelva `HTTP_INTERCEPTORS`, inyectará un **array con ambos interceptores**. Esto permite encadenar lógica de autenticación, logging, cacheo, etc., sin que una implementación sobrescriba a la otra.

### Escenarios típicos
- Interceptores de HTTP.  
- Estrategias de logging múltiples.  
- Plugins o extensiones que deben coexistir.  

### 6.5.2. View Providers

Los **View Providers** son una variante de los `providers` que se declaran en un componente, pero con un alcance más restringido: **solo están disponibles para la vista del componente, no para sus proyecciones de contenido (ng-content)**.

Esto es clave cuando queremos que un servicio se use dentro del propio componente y sus hijos internos, pero no se filtre hacia componentes que se proyectan en él.

### Ejemplo: servicio de control interno

```ts
import { Component } from '@angular/core';
import { LoggerService } from './logger.service';

@Component({
  selector: 'app-panel',
  template: `
    <h2>Panel</h2>
    <ng-content></ng-content>
  `,
  viewProviders: [LoggerService]
})
export class PanelComponent {}
```

- Si un componente hijo declarado dentro de la plantilla de `PanelComponent` inyecta `LoggerService`, recibirá la instancia definida en `viewProviders`.  
- Pero si un componente externo se proyecta dentro de `<ng-content>`, no tendrá acceso a ese `LoggerService`.  

👉 Esto evita fugas de dependencias y mantiene la encapsulación del componente.

### 6.5.3. Escenarios complejos donde brillan

#### 🔹 Plugins y extensiones con Array Providers
En aplicaciones grandes, es común tener un sistema de **plugins** donde cada módulo aporta su propia lógica (ej. validadores, interceptores, estrategias de cache). Con `multi: true`, todos se acumulan en un array y Angular los ejecuta en orden.

#### 🔹 Encapsulación estricta con View Providers
Imagina un componente de librería que expone un API pública, pero que internamente necesita un servicio auxiliar. Si usáramos `providers`, ese servicio podría filtrarse a componentes proyectados, generando comportamientos inesperados. Con `viewProviders`, garantizamos que ese servicio solo se use dentro de la vista interna.

#### 🔹 Formularios avanzados
En formularios reactivos, Angular usa `viewProviders` para inyectar directivas como `NgControl` y evitar conflictos entre controles internos y externos.

### 6.5.4. Buenas prácticas

- **Usa Array Providers solo cuando realmente necesites múltiples instancias**. Si no añades `multi: true`, el último provider sobrescribirá a los anteriores.  
- **Prefiere View Providers para servicios internos** que no deben ser visibles desde fuera del componente.  
- **Documenta el orden de ejecución** en Array Providers (ej. interceptores), ya que puede afectar al resultado final.  
- **Combínalos con InjectionTokens** para mayor claridad y seguridad en configuraciones complejas.  


## 6.6. Functional Providers y su integración con Standalone Components

Con la llegada del modelo **standalone** en Angular, la forma de configurar dependencias también evolucionó. Los **Functional Providers** son una nueva manera, más declarativa y expresiva, de registrar servicios y dependencias en la aplicación.  

En lugar de usar objetos de configuración con `provide`, `useClass`, `useValue` o `useFactory`, los Functional Providers se apoyan en **funciones auxiliares** que encapsulan la lógica de registro. Esto hace que el código sea más legible, más fácil de mantener y más coherente con el estilo moderno de Angular.

### 6.6.1. ¿Qué son los Functional Providers?

Un **Functional Provider** es simplemente una función que devuelve un conjunto de providers ya configurados. Angular ofrece varias funciones listas para usar, como:  

- `provideHttpClient()` → configura el cliente HTTP.  
- `provideRouter()` → configura el enrutador.  
- `provideAnimations()` → habilita animaciones.  
- `provideZoneChangeDetection()` → configura la detección de cambios.  

Estas funciones encapsulan la configuración que antes requería NgModules o providers clásicos, y están pensadas para integrarse directamente en aplicaciones standalone.

### 6.6.2. Ejemplo básico con `bootstrapApplication`

En Angular 20, las aplicaciones standalone se inician con `bootstrapApplication`. Allí es donde los Functional Providers brillan:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app.component';
import { routes } from './app.routes';
import { authInterceptor } from './auth.interceptor';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])
    ),
    provideRouter(routes)
  ]
});
```

👉 Aquí, en lugar de importar `HttpClientModule` o `RouterModule.forRoot()`, usamos funciones declarativas (`provideHttpClient`, `provideRouter`) que configuran todo lo necesario.  

### 6.6.3. Integración con Standalone Components

Los **Standalone Components** pueden declarar sus propios providers, y estos pueden ser tanto clásicos como funcionales.  

Ejemplo: un componente que necesita un cliente HTTP con configuración específica:

```ts
import { Component } from '@angular/core';
import { provideHttpClient, withFetch } from '@angular/common/http';

@Component({
  selector: 'app-dashboard',
  template: `<h2>Dashboard</h2>`,
  providers: [
    provideHttpClient(withFetch()) // Functional Provider a nivel de componente
  ]
})
export class DashboardComponent {}
```

👉 De esta forma, el `DashboardComponent` y sus hijos tendrán un cliente HTTP configurado con `fetch`, sin afectar al resto de la aplicación.

### 6.6.4. Functional Providers personalizados

También podemos crear nuestros propios Functional Providers para encapsular configuraciones comunes.  

Ejemplo: un provider para configuración de API:

```ts
import { InjectionToken } from '@angular/core';

export const API_URL = new InjectionToken<string>('api.url');

export function provideApiConfig(url: string) {
  return [
    { provide: API_URL, useValue: url }
  ];
}
```

Uso en `bootstrapApplication`:

```ts
bootstrapApplication(AppComponent, {
  providers: [
    provideApiConfig('https://api.midominio.com')
  ]
});
```

👉 Esto nos permite centralizar configuraciones y reutilizarlas en distintos contextos.

### 6.6.5. Diferencias frente a providers clásicos

- **Sintaxis**: los clásicos usan objetos `{ provide, useClass }`; los funcionales usan funciones (`provideX()`).  
- **Legibilidad**: los funcionales son más concisos y expresivos.  
- **Compatibilidad**: ambos conviven; los clásicos siguen siendo válidos.  
- **Recomendación**: en aplicaciones nuevas standalone, se recomienda preferir los funcionales.  

### 6.6.6. Casos de uso recomendados

- **Configuración de servicios del framework**: HTTP, Router, Animations, i18n.  
- **Encapsulación de configuraciones comunes**: crear funciones `provideX()` propias para centralizar lógica.  
- **Componentes standalone con dependencias específicas**: aislar configuraciones sin afectar al resto de la app.  


## 6.7. Casos prácticos de DI en aplicaciones enterprise distribuidas

En aplicaciones grandes y distribuidas, la **DI** no solo sirve para inyectar un servicio de utilidades o un cliente HTTP. Se convierte en una herramienta estratégica para:  
- **Orquestar comunicación entre módulos distribuidos**.  
- **Configurar dinámicamente servicios según el entorno**.  
- **Permitir extensibilidad** en arquitecturas de microfrontends o librerías compartidas.  
- **Garantizar testabilidad y mantenibilidad** en equipos grandes.  

Veamos algunos casos prácticos.

### 6.7.1. Configuración multi-entorno con `InjectionToken`

En entornos distribuidos, cada microservicio puede tener su propio endpoint. En lugar de “hardcodear” URLs, podemos centralizar la configuración con `InjectionToken`.

```ts
import { InjectionToken } from '@angular/core';

export interface ApiConfig {
  users: string;
  orders: string;
}

export const API_CONFIG = new InjectionToken<ApiConfig>('api.config');
```

Provisión en `bootstrapApplication`:

```ts
bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: API_CONFIG,
      useValue: {
        users: 'https://api.empresa.com/users',
        orders: 'https://api.empresa.com/orders'
      }
    }
  ]
});
```

Uso en un servicio:

```ts
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(@Inject(API_CONFIG) private config: ApiConfig, private http: HttpClient) {}

  getUsers() {
    return this.http.get(`${this.config.users}`);
  }
}
```

👉 Esto permite que cada entorno (dev, staging, prod) tenga su propia configuración sin tocar el código de los servicios.

### 6.7.2. Microfrontends y DI compartida

En arquitecturas de **microfrontends**, distintos equipos desarrollan módulos independientes que se integran en una misma aplicación. La DI permite **compartir servicios globales** (ej. autenticación, tracking) y a la vez **aislar servicios locales**.

Ejemplo:  
- El *shell* de la aplicación provee un `AuthService` global.  
- Cada microfrontend puede inyectarlo sin redefinirlo.  
- Si un microfrontend necesita un `LoggerService` distinto, puede declararlo en sus `providers`, sin afectar al resto.

```ts
@Component({
  selector: 'app-orders',
  template: `<h2>Pedidos</h2>`,
  providers: [LoggerService] // instancia propia, aislada
})
export class OrdersComponent {
  constructor(private auth: AuthService, private logger: LoggerService) {}
}
```

👉 Así, `AuthService` es compartido, pero `LoggerService` es independiente en cada microfrontend.

### 6.7.3. Plugins y extensibilidad con Array Providers

En aplicaciones distribuidas, es común tener un sistema de **plugins** donde cada módulo aporta lógica adicional (ej. validadores, interceptores, estrategias de cache).  

Con `multi: true`, Angular acumula todas las implementaciones en un array:

```ts
export const VALIDATORS = new InjectionToken<Array<ValidatorFn>>('validators');

bootstrapApplication(AppComponent, {
  providers: [
    { provide: VALIDATORS, useValue: (v: string) => v.length > 3, multi: true },
    { provide: VALIDATORS, useValue: (v: string) => /^[a-z]+$/.test(v), multi: true }
  ]
});
```

Uso en un servicio:

```ts
@Injectable({ providedIn: 'root' })
export class ValidationService {
  constructor(@Inject(VALIDATORS) private validators: ValidatorFn[]) {}

  validate(value: string): boolean {
    return this.validators.every(fn => fn(value));
  }
}
```

👉 Esto permite que cada equipo añada sus validadores sin modificar el código central.

### 6.7.4. Inyección condicional con factorías

En sistemas distribuidos, a veces necesitamos que un servicio cambie de implementación según el entorno o el rol del usuario.  

```ts
@Injectable()
export class MockPaymentService { /* ... */ }

@Injectable()
export class RealPaymentService { /* ... */ }

export const PAYMENT_SERVICE = new InjectionToken<PaymentService>('payment.service');

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: PAYMENT_SERVICE,
      useFactory: () => {
        return environment.production ? new RealPaymentService() : new MockPaymentService();
      }
    }
  ]
});
```

👉 Esto permite que en desarrollo usemos mocks y en producción el servicio real, sin cambiar el código de los componentes.

### 6.7.5. Comunicación entre equipos con servicios compartidos

En aplicaciones distribuidas, distintos equipos pueden necesitar comunicarse a través de un **bus de eventos**. Este bus puede ser un servicio inyectado globalmente:

```ts
@Injectable({ providedIn: 'root' })
export class EventBus {
  private events = new Subject<{ type: string; payload: any }>();

  emit(event: { type: string; payload: any }) {
    this.events.next(event);
  }

  on(type: string): Observable<any> {
    return this.events.asObservable().pipe(filter(e => e.type === type), map(e => e.payload));
  }
}
```

👉 Cualquier microfrontend puede inyectar `EventBus` y emitir/escuchar eventos, sin acoplarse directamente a otros módulos.


## 6.8. Estrategias de troubleshooting en jerarquías de inyección profundas

La **jerarquía de inyectores** en Angular es poderosa, pero también puede ser fuente de errores difíciles de rastrear. Cuando un servicio no se resuelve como esperábamos, o cuando aparecen instancias duplicadas, es señal de que algo en la cadena de inyección no está funcionando como debería.  

En esta sección veremos **estrategias prácticas de diagnóstico y resolución** para este tipo de problemas.

### 6.8.1. Comprender la jerarquía de inyectores

Angular mantiene distintos niveles de inyectores:  
- **EnvironmentInjector (raíz de la aplicación)**: contiene los servicios globales (`providedIn: 'root'`).  
- **ElementInjector (por componente/directiva)**: cada componente puede declarar sus propios `providers`.  
- **Inyectores de rutas**: cada ruta puede tener sus propios servicios en `providers`.  

Cuando un componente solicita un servicio, Angular busca en su **ElementInjector** y, si no lo encuentra, sube hacia el padre hasta llegar al `EnvironmentInjector`. Si no lo encuentra en ningún nivel, lanza un error `NullInjectorError`.

### 6.8.2. Errores comunes en jerarquías profundas

- **`NullInjectorError`**: Angular no encuentra un provider para el token solicitado.  
- **Instancias duplicadas**: el mismo servicio se declara en varios niveles, generando múltiples instancias en lugar de una compartida.  
- **Confusión con modificadores** (`@Self`, `@SkipSelf`, `@Optional`): usados incorrectamente, pueden provocar que Angular busque en el inyector equivocado o devuelva `null`.  
- **Uso indebido de `inject()` fuera de contexto**: recordemos que `inject()` solo funciona dentro de un contexto de inyección válido.  

### 6.8.3. Estrategias de diagnóstico

#### 🔹 1. Revisar el alcance del servicio
- Verifica si el servicio está marcado con `providedIn: 'root'` o si se está declarando en `providers` de un componente.  
- Si aparece más de una instancia, probablemente se está declarando en varios niveles.  

#### 🔹 2. Usar `console.log` en el constructor del servicio
Una técnica simple pero efectiva:  
```ts
@Injectable({ providedIn: 'root' })
export class LoggerService {
  constructor() {
    console.log('LoggerService instanciado');
  }
}
```
Si ves múltiples logs, significa que el servicio se está creando más de una vez (probablemente por providers locales).

#### 🔹 3. Aplicar modificadores de inyección
- Usa `@Self()` para confirmar si el servicio está realmente en el inyector local.  
- Usa `@SkipSelf()` para forzar a Angular a buscar en el padre.  
- Usa `@Optional()` para evitar errores cuando un servicio puede no estar presente.  

#### 🔹 4. Verificar el contexto de `inject()`
Si usas `inject()`, asegúrate de que se ejecute en un contexto válido: constructor, inicialización de propiedad, factoría de provider o dentro de `runInInjectionContext()`.

#### 🔹 5. Revisar la jerarquía de componentes
En aplicaciones grandes, es útil **mapear el árbol de componentes** y anotar dónde se declaran los providers. Esto ayuda a entender por qué un servicio se resuelve en un nivel y no en otro.

### 6.8.4. Estrategias de resolución

- **Centralizar servicios globales** en `providedIn: 'root'` para evitar duplicados.  
- **Usar providers locales solo cuando sea necesario** (ej. un servicio que debe tener estado aislado por componente).  
- **Evitar declarar el mismo servicio en múltiples niveles** salvo que se busque explícitamente instancias distintas.  
- **Crear `InjectionToken`s para configuraciones** en lugar de usar servicios duplicados.  
- **Documentar la jerarquía de inyección** en proyectos grandes para que todo el equipo entienda dónde se proveen los servicios.  

### 6.8.5. Ejemplo práctico

```ts
@Component({
  selector: 'app-parent',
  template: `<app-child></app-child>`,
  providers: [LoggerService] // instancia propia
})
export class ParentComponent {}

@Component({
  selector: 'app-child',
  template: `<p>Child works!</p>`
})
export class ChildComponent {
  constructor(
    @Self() private localLogger: LoggerService,       // busca en el hijo
    @SkipSelf() private parentLogger: LoggerService,  // busca en el padre
    @Optional() private maybeAnalytics?: AnalyticsService // puede ser null
  ) {}
}
```

👉 Aquí podemos diagnosticar fácilmente qué instancia se está inyectando en cada nivel y evitar confusiones.


## 6.9. Diferencias clave entre DI clásico (NgModules) y DI funcional moderno

La **Inyección de Dependencias (DI)** ha sido siempre uno de los pilares de Angular. Sin embargo, la forma de configurar y consumir dependencias ha cambiado radicalmente en los últimos años.  

En las primeras versiones, todo giraba en torno a los **NgModules**: eran la unidad de organización y el lugar donde se declaraban los providers. Con la llegada de los **Standalone Components** y los **Functional Providers**, Angular 20 ofrece un modelo mucho más simple, declarativo y flexible.  

### 6.9.1. DI clásico con NgModules

En el modelo clásico, los servicios se registraban en los `providers` de un NgModule.  

Ejemplo:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { AppComponent } from './app.component';
import { UserService } from './user.service';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [UserService],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

Características:  
- Los **NgModules** eran obligatorios para organizar componentes, directivas, pipes y servicios.  
- Los servicios se registraban en `providers` o con `@Injectable({ providedIn: 'root' })`.  
- La configuración de dependencias estaba muy ligada a la estructura modular.  

### 6.9.2. DI funcional moderno con Standalone + Functional Providers

En Angular 20, los NgModules ya no son necesarios. Todo es **standalone por defecto**, y los servicios se configuran con **Functional Providers**.  

Ejemplo con `bootstrapApplication`:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app.component';
import { routes } from './app.routes';
import { AuthInterceptor } from './auth.interceptor';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(withInterceptors([AuthInterceptor])),
    provideRouter(routes)
  ]
});
```

Características:  
- No se necesitan NgModules: los componentes, directivas y pipes son standalone.  
- Los servicios del framework (HTTP, Router, Animations, i18n) se configuran con funciones declarativas (`provideHttpClient`, `provideRouter`, etc.).  
- Es más fácil **aislar providers a nivel de componente o ruta**, sin depender de módulos.  

### 6.9.3. Diferencias clave

| Aspecto | DI clásico (NgModules) | DI funcional moderno (Angular 20) |
|---------|-------------------------|-----------------------------------|
| **Unidad de organización** | NgModules obligatorios | Standalone Components por defecto |
| **Registro de servicios** | `providers` en NgModules o `@Injectable({ providedIn })` | Functional Providers (`provideX()`) en `bootstrapApplication`, rutas o componentes |
| **Configuración del framework** | Importación de módulos (`HttpClientModule`, `RouterModule.forRoot`) | Funciones declarativas (`provideHttpClient`, `provideRouter`) |
| **Flexibilidad** | Configuración centralizada en módulos | Configuración granular en componentes y rutas |
| **Legibilidad** | Verbosa, con boilerplate | Concisa y declarativa |
| **Migración** | Necesario en proyectos legacy | Recomendado en proyectos nuevos |

### 6.9.4. Casos de uso

- **DI clásico (NgModules)**  
  - Proyectos legacy que aún dependen de módulos.  
  - Migraciones progresivas donde conviven NgModules y Standalone.  
  - Escenarios donde se quiere mantener compatibilidad con librerías antiguas.  

- **DI funcional moderno**  
  - Nuevos proyectos en Angular 20.  
  - Aplicaciones que buscan simplicidad y menor boilerplate.  
  - Configuración declarativa de servicios del framework.  
  - Escenarios donde se necesita aislar dependencias por componente o ruta.  


## 6.10. Buenas prácticas para mantener escalabilidad y testabilidad con DI

La DI en Angular es mucho más que un mecanismo técnico: es una estrategia de arquitectura. En proyectos enterprise, donde múltiples equipos trabajan sobre la misma base de código y la aplicación debe crecer sin perder calidad, aplicar buenas prácticas en DI es esencial.  

### 6.10.1. Centralizar y tipar la configuración con `InjectionToken`

- Usa **`InjectionToken`** para valores de configuración (URLs, flags, claves).  
- Evita inyectar directamente strings o números, ya que no son únicos.  
- Define interfaces para garantizar tipado fuerte y claridad.  

Ejemplo:

```ts
export interface ApiConfig {
  users: string;
  orders: string;
}

export const API_CONFIG = new InjectionToken<ApiConfig>('api.config');

bootstrapApplication(AppComponent, {
  providers: [
    { provide: API_CONFIG, useValue: { users: '/api/users', orders: '/api/orders' } }
  ]
});
```

👉 Esto facilita la escalabilidad, ya que cada equipo puede extender la configuración sin romper el contrato.

### 6.10.2. Usar Functional Providers para servicios del framework

- Prefiere `provideHttpClient`, `provideRouter`, `provideAnimations` en lugar de módulos clásicos.  
- Encapsula configuraciones comunes en funciones `provideX()` propias.  
- Esto reduce el *boilerplate* y hace que la configuración sea más declarativa y fácil de mantener.  

### 6.10.3. Controlar el alcance de los servicios

- **Globales (`providedIn: 'root'`)**: para servicios compartidos en toda la aplicación (ej. autenticación).  
- **Locales (providers en componentes o rutas)**: para servicios con estado aislado o dependencias específicas.  
- **ViewProviders**: para encapsular servicios internos que no deben filtrarse a contenido proyectado.  

👉 Esto evita instancias duplicadas y mantiene la arquitectura limpia.

### 6.10.4. Favorecer la inyección sobre la creación manual

- Nunca uses `new` para instanciar servicios dentro de componentes.  
- Siempre inyecta dependencias: esto permite sustituirlas fácilmente en pruebas o entornos distintos.  

Ejemplo en pruebas:

```ts
TestBed.configureTestingModule({
  providers: [
    { provide: UserService, useValue: jasmine.createSpyObj('UserService', ['getUser']) }
  ]
});
```

👉 Gracias a la DI, podemos sustituir servicios reales por *mocks* en pruebas unitarias.

### 6.10.5. Diseñar servicios pequeños y especializados

- Aplica el **Principio de Responsabilidad Única (SRP)**: cada servicio debe encargarse de una sola cosa.  
- Divide servicios grandes en varios más pequeños y composables.  
- Esto mejora la testabilidad y facilita la colaboración entre equipos.  

### 6.10.6. Aprovechar modificadores de inyección para escenarios complejos

- `@Optional()` para dependencias que pueden no estar presentes.  
- `@Self()` para garantizar que un servicio se provea localmente.  
- `@SkipSelf()` para forzar el uso de un servicio del padre.  

👉 Estos modificadores ayudan a mantener jerarquías de inyección predecibles y fáciles de depurar.

### 6.10.7. Integrar DI en la estrategia de pruebas

- Usa **TestBed** solo cuando sea necesario (componentes, directivas).  
- Para servicios puros, prueba directamente sin infraestructura de Angular.  
- Simula dependencias con *spies* o *mocks* inyectados mediante providers.  
- En pruebas E2E, usa atributos `data-cy` o similares para desacoplar la lógica de test del CSS o del contenido.  
