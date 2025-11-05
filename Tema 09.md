# 9. SSR, Prerender e Hydration en Angular 20

## 9.1. Fundamentos de Server Side Rendering en Angular

### 9.1.1. ¿Qué es el Server Side Rendering (SSR)?

El **Server Side Rendering (SSR)** es una técnica en la que el **HTML de la aplicación se genera en el servidor** y se envía ya renderizado al navegador.  
- En el modelo tradicional de **Client Side Rendering (CSR)**, el servidor solo envía un “shell” vacío y el bundle de JavaScript; es el navegador quien ejecuta Angular y construye el DOM.  
- Con **SSR**, el servidor procesa la aplicación Angular, genera el HTML completo de la ruta solicitada y lo entrega al cliente.  

👉 Resultado: el usuario ve contenido inmediato, incluso antes de que se descargue y ejecute el JavaScript.  

### 9.1.2. Beneficios principales del SSR

- **Mejor rendimiento percibido**: el contenido aparece más rápido, reduciendo métricas como **First Contentful Paint (FCP)** y **Largest Contentful Paint (LCP)**.  
- **SEO optimizado**: los motores de búsqueda pueden indexar fácilmente el contenido ya renderizado.  
- **Accesibilidad mejorada**: los lectores de pantalla y bots reciben HTML completo desde el inicio.  
- **Experiencia en redes lentas**: los usuarios en dispositivos móviles o conexiones 3G ven contenido antes de que cargue todo el bundle.  
- **Integración con datos dinámicos**: el servidor puede consultar APIs o bases de datos y renderizar el resultado directamente en el HTML.  

### 9.1.3. Cómo funciona el SSR en Angular 20

Angular 20 incorpora un modelo de **renderizado híbrido** que combina SSR, prerender y CSR según convenga:  

1. **Solicitud inicial**:  
   - El usuario accede a una ruta.  
   - El servidor ejecuta Angular en Node.js (usando `CommonEngine`) y genera el HTML correspondiente.  

2. **Entrega al cliente**:  
   - El navegador recibe un documento HTML ya poblado con contenido.  
   - El usuario percibe la página como cargada inmediatamente.  

3. **Hydration**:  
   - Una vez descargado el bundle de Angular, el framework “hidrata” el HTML existente, conectando los componentes renderizados en servidor con la lógica interactiva en cliente.  

### 9.1.4. Configuración básica de SSR en Angular 20

- **Crear un proyecto con SSR desde cero**:  

```bash
ng new mi-app --ssr
```

- **Añadir SSR a un proyecto existente**:  

```bash
ng add @angular/ssr
```

Esto genera:  
- `server.ts`: servidor Node.js con Express.  
- `main.server.ts`: punto de arranque de la app en servidor.  
- `app.config.server.ts`: configuración específica para SSR.  

### 9.1.5. Diferencias clave con CSR

| Aspecto | CSR (Client Side Rendering) | SSR (Server Side Rendering) |
|---------|-----------------------------|-----------------------------|
| **Entrega inicial** | HTML vacío + JS | HTML completo renderizado |
| **Tiempo hasta contenido visible** | Más lento (depende del JS) | Más rápido (contenido inmediato) |
| **SEO** | Limitado (bots deben ejecutar JS) | Óptimo (HTML listo para indexar) |
| **Carga en servidor** | Baja | Alta (renderizado en cada request) |
| **Experiencia offline** | Depende de PWA | Puede combinarse con PWA |

### 9.1.6. Cuándo usar SSR

- **Aplicaciones públicas** donde el SEO es crítico (blogs, e‑commerce, medios).  
- **Landing pages** que necesitan mostrar contenido inmediato.  
- **Aplicaciones con contenido dinámico** que debe estar disponible en HTML desde el inicio.  
- **Proyectos enterprise** que buscan mejorar Core Web Vitals y experiencia en dispositivos móviles.  


## 9.2. Configuración inicial de SSR con Angular CLI y Standalone Components

### 9.2.1. Creación de un proyecto con SSR desde cero

Angular 20 simplifica enormemente la configuración de SSR gracias al **Angular CLI**. Desde la creación del proyecto podemos habilitarlo con un simple flag:

```bash
ng new mi-proyecto --ssr
```

Esto genera automáticamente:  
- Una aplicación basada en **Standalone Components** (ya son el estándar por defecto).  
- Configuración de **SSR híbrido** lista para usarse.  
- Archivos adicionales para el servidor (`main.server.ts`, `server.ts`, `app.config.server.ts`).  

👉 Con este comando, el proyecto queda preparado para renderizar en servidor y cliente sin necesidad de configuraciones manuales complejas.  

### 9.2.2. Añadir SSR a un proyecto existente

Si ya tienes una aplicación Angular (CSR) y quieres habilitar SSR, basta con ejecutar:

```bash
ng add @angular/ssr
```

Esto:  
- Instala las dependencias necesarias (`@angular/ssr`).  
- Genera los archivos de configuración de servidor.  
- Ajusta `angular.json` para incluir los targets de build y serve para SSR.  

### 9.2.3. Archivos clave generados

- **`main.server.ts`**: punto de entrada de la aplicación en servidor.  
- **`server.ts`**: servidor Node.js (Express) que renderiza las rutas.  
- **`app.config.server.ts`**: configuración específica para el entorno SSR (providers, rutas de servidor, etc.).  

Ejemplo de configuración en `app.config.server.ts`:

```ts
import { ApplicationConfig } from '@angular/core';
import { provideServerRendering, withRoutes } from '@angular/ssr';
import { serverRoutes } from './app.routes.server';

export const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes)),
  ]
};
```

👉 Aquí se definen las rutas que el servidor debe renderizar y se habilita el motor de SSR.  

### 9.2.4. Integración con Standalone Components

En Angular 20, los **Standalone Components** son la base de la arquitectura. Esto significa que:  
- No necesitas `AppModule`.  
- La configuración de SSR se hace directamente sobre `ApplicationConfig`.  
- Los componentes standalone funcionan de forma nativa en SSR, sin adaptaciones adicionales.  

Ejemplo de bootstrap en `main.server.ts`:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { serverConfig } from './app/app.config.server';

const bootstrap = () => bootstrapApplication(AppComponent, serverConfig);

export default bootstrap;
```

### 9.2.5. Ejecución y verificación

Una vez configurado, puedes probar SSR con:

```bash
npm run dev:ssr
```

o en producción:

```bash
npm run build:ssr
npm run serve:ssr
```

Al abrir la aplicación en el navegador, si inspeccionas el **HTML inicial** verás que el contenido ya está renderizado en servidor, incluso antes de que se ejecute el JavaScript del cliente.  

### 9.2.6. Buenas prácticas iniciales

- **Usar `priority` en imágenes críticas** (`NgOptimizedImage`) para que se pre-rendericen en SSR.  
- **Configurar resolvers y guards** con cuidado: se ejecutan en servidor y deben ser eficientes.  
- **Evitar dependencias del `window` o `document`** directamente en componentes, ya que no existen en el servidor.  
- **Probar con Lighthouse** para verificar mejoras en LCP y SEO tras habilitar SSR.  

⚠️ **Advertencia importante:** Nunca accedas directamente a APIs del navegador (`window`, `document`, `navigator`) en componentes o servicios que se ejecutan en SSR. Si necesitas acceder a ellas, encapsula la lógica en servicios y usa comprobaciones de entorno (`isPlatformBrowser`) para evitar errores en el servidor.

**Recomendación:** Usa siempre `ChangeDetectionStrategy.OnPush` en los componentes que se renderizan en SSR para minimizar ciclos de cambio y mejorar el rendimiento.


## 9.3. Prerender híbrido: cuándo conviene y cómo implementarlo

### 9.3.1. ¿Qué es el prerender híbrido?

El **prerender híbrido** combina lo mejor de tres mundos:  
- **SSR (Server-Side Rendering)**: renderizado en servidor bajo demanda.  
- **Prerender (SSG, Static Site Generation)**: generación de HTML estático en tiempo de build.  
- **CSR (Client-Side Rendering)**: renderizado clásico en cliente.  

Angular 20 permite decidir **qué rutas se prerenderizan en build**, cuáles se sirven con **SSR dinámico** y cuáles se dejan en **CSR puro**, ofreciendo un control granular sobre el rendimiento y la carga del servidor.  

👉 En otras palabras: no todas las páginas de la aplicación deben tratarse igual. Algunas conviene generarlas estáticamente, otras dinámicamente y otras dejarlas al cliente.  

### 9.3.2. ¿Cuándo conviene usar prerender híbrido?

- **Páginas estáticas y universales** (ej. aviso legal, FAQ, landing pages):  
  → Conviene prerenderizarlas en build, ya que no dependen de datos dinámicos ni de usuario.  

- **Páginas dinámicas pero públicas** (ej. detalle de producto, artículos de blog):  
  → Conviene servirlas con SSR, para que cada request reciba HTML actualizado y optimizado para SEO.  

- **Páginas privadas o dashboards internos** (ej. panel de administración, área de usuario):  
  → Conviene dejarlas en CSR, ya que requieren datos específicos del usuario y no aportan valor al prerender.  

- **Sitios de gran escala** (ej. e‑commerce con miles de productos):  
  → Se puede prerender solo un subconjunto (productos destacados) y dejar el resto en SSR bajo demanda.  

### 9.3.3. Ventajas del enfoque híbrido

- **Rendimiento óptimo**: las páginas estáticas cargan instantáneamente, las dinámicas se sirven rápido con SSR y las privadas no sobrecargan el servidor.  
- **SEO mejorado**: los buscadores reciben HTML completo en las rutas públicas.  
- **Escalabilidad**: se reduce la carga en el servidor al prerenderizar lo que no cambia.  
- **Flexibilidad**: cada ruta puede configurarse según sus necesidades.  

## 9.3.4. Cómo implementar **prerender híbrido** (SSR + SSG + CSR) en **Angular 20**

En Angular 20 puedes decidir **por ruta** si se renderiza en **servidor** (SSR), **en build** como **estático** (SSG/prerender) o **solo en cliente** (CSR). La configuración correcta se hace separando **rutas del cliente** (router normal) y **rutas del servidor** (donde declaras el `RenderMode`).


### Paso 1 · Habilitar SSR (y, opcionalmente, *server routing*)

En un proyecto existente:

```bash
ng add @angular/ssr
# opcional: añade el fichero de rutas de servidor
ng add @angular/ssr --server-routing
```

Esto añade la tubería de SSR/híbrido y deja lista la hidratación del cliente. Si usas `--server-routing`, la CLI crea el archivo de rutas de servidor.


### Paso 2 · Definir **rutas del cliente** (`app.routes.ts`)

Aquí **no** se declara el `RenderMode`. Son tus rutas normales del router:

```ts
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./home/home.component').then(m => m.HomeComponent),
  },
  {
    path: 'product/:id',
    loadComponent: () =>
      import('./product/product.component').then(m => m.ProductComponent),
  },
  {
    path: 'dashboard',
    loadComponent: () =>
      import('./dashboard/dashboard.component').then(m => m.DashboardComponent),
  },
];
```

> Estas rutas son las que se usan en el navegador (CSR) y deben **corresponder 1:1** con los paths que declares en las rutas de servidor del siguiente paso. 


### Paso 3 · Definir **rutas del servidor** con `RenderMode` (`app.routes.server.ts`)

En este archivo indicas **cómo** se renderiza cada path: `Server` (SSR), `Prerender` (SSG) o `Client` (CSR).

```ts
// app.routes.server.ts
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  // Home: prerender (SSG)
  { path: '', renderMode: RenderMode.Prerender },

  // Ficha de producto: SSR dinámico en cada request
  { path: 'product/:id', renderMode: RenderMode.Server },

  // Dashboard: solo CSR
  { path: 'dashboard', renderMode: RenderMode.Client },

  // Resto de rutas: SSR por defecto
  { path: '**', renderMode: RenderMode.Server },
];
```

Los valores del enum `RenderMode` son exactamente `Server`, `Client` y `Prerender`.


### Paso 4 · Registrar las **rutas de servidor** en la app (`app.config.server.ts`)

Registra el array `serverRoutes` para que la app conozca los *render modes* por path:

```ts
// app.config.server.ts
import { ApplicationConfig } from '@angular/core';
import { provideServerRendering, withRoutes } from '@angular/ssr';
import { serverRoutes } from './app.routes.server';

export const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(
      withRoutes(serverRoutes)      
    ),
  ],
};
```

### Paso 5 · **Prerender** de rutas dinámicas con `getPrerenderParams`

Si una ruta con `RenderMode.Prerender` tiene **parámetros** (p. ej. `product/:id`), debes indicar qué combinaciones se prerenderán en *build*:

```ts
// app.routes.server.ts (variante con SSG para algunos productos)
import { RenderMode, ServerRoute, PrerenderFallback } from '@angular/ssr';
import { inject } from '@angular/core';
import { ProductService } from './data/product.service';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'product/:id',
    renderMode: RenderMode.Prerender,
    // Qué hacer si llega un ID no prerenderizado:
    fallback: PrerenderFallback.Server, // por defecto es Server (SSR)
    async getPrerenderParams() {
      // ¡Ojo! `inject` debe llamarse SINCRÓNICAMENTE (no tras un await)
      const api = inject(ProductService);
      const ids = await api.getFeaturedIds(); // p.ej. ['123', '456']
      return ids.map(id => ({ id })); // => /product/123, /product/456
    },
  },
];
```

*   `getPrerenderParams` se ejecuta **en build** y devuelve un array de objetos `{ nombreParam: valor }`.
*   Puedes definir un **fallback**: `Server` (SSR), `Client` (CSR) o `None` (no gestiona la ruta si no fue prerenderada).
*   Evita usar APIs del navegador; y recuerda que `inject(...)` debe invocarse **antes** de cualquier `await`.

### Paso 6 · Compilar y servir

*   **Build**: `ng build`  
    Por defecto, Angular **prerenderiza** lo configurado y **genera** también el *server bundle* para SSR.
*   **Sitio 100% estático** (sin server Node): en `angular.json` puedes fijar  
    `"outputMode": "static"` para generar únicamente archivos estáticos (SSG) sin servidor.


### Paso 7 · Hidratación y *transfer cache* (opcional)

La hidratación ya viene activa con la CLI. Si necesitas ajustar la reutilización de respuestas HTTP entre servidor y cliente, añade:

```ts
// main.ts (cliente)
import { bootstrapApplication, provideClientHydration, withHttpTransferCacheOptions } from '@angular/platform-browser';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(
      withHttpTransferCacheOptions({
        // ejemplos:
        includeHeaders: ['ETag', 'Cache-Control'],
        includePostRequests: false,
        includeRequestsWithAuthHeaders: false,
      }),
    ),
  ],
});
```

Esto evita dobles peticiones al hidratar y te permite afinar qué respuestas se transfieren al cliente.


### 9.3.5. Consideraciones prácticas

- **Tiempo de build**: prerenderizar muchas rutas puede alargar la compilación. Conviene limitarlo a páginas realmente estáticas.  
- **Tamaño del despliegue**: cada página prerenderizada genera un archivo HTML adicional.  
- **Datos dinámicos**: prerender solo es viable si los datos están disponibles en build. Si dependen de usuario o cambian constantemente, mejor usar SSR.  
- **SEO**: prerender híbrido es ideal para sitios que combinan contenido estático (indexable) con áreas privadas (no indexables).  


## 9.4. Hydration incremental estable en Angular 20

### 9.4.1. ¿Qué es la hydration en Angular?

Cuando usamos **SSR** o **Prerender**, el servidor entrega al navegador un HTML ya renderizado. Sin embargo, ese HTML es estático: no tiene interactividad. La **hydration** es el proceso por el cual Angular “conecta” ese HTML con la lógica de los componentes, eventos y estado en el cliente, devolviendo la interactividad a la aplicación.  

Hasta Angular 19, la hydration era **completa**: toda la aplicación se rehidrataba de golpe. Esto funcionaba, pero podía ser costoso en aplicaciones grandes, ya que el navegador debía procesar todos los componentes inmediatamente.  

### 9.4.2. ¿Qué es la hydration incremental?

La **hydration incremental** es una evolución de este proceso. En lugar de rehidratar toda la aplicación de una sola vez, Angular 20 permite **hidratar secciones del DOM de forma progresiva**, solo cuando son necesarias.  

- El HTML inicial sigue estando completo gracias al SSR o al prerender.  
- Angular no activa todos los componentes de inmediato, sino que los va “despertando” según condiciones definidas.  
- Esto reduce el coste inicial de JavaScript y mejora métricas como **Time to Interactive (TTI)** y **Largest Contentful Paint (LCP)**.  

### 9.4.3. ¿Por qué es importante?

- **Mejor rendimiento en móviles y redes lentas**: la aplicación se siente usable antes, incluso en dispositivos de gama baja.  
- **Bundles más pequeños**: al no necesitar hidratar todo de golpe, se pueden diferir dependencias.  
- **Menos *layout shifts***: con hydration incremental, incluso los bloques diferidos (`@defer`) pueden renderizarse sin causar saltos visuales.  
- **Control granular**: los desarrolladores deciden qué partes de la app se hidratan primero y cuáles pueden esperar.  

### 9.4.4. Cómo habilitar la hydration incremental

Para usarla, primero hay que tener **SSR + hydration básica** activada. Luego, basta con añadir la opción `withIncrementalHydration()` al provider de cliente:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withIncrementalHydration } from '@angular/platform-browser';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(withIncrementalHydration())
  ]
});
```

👉 Esto habilita la hydration incremental en toda la aplicación.  

### 9.4.5. Uso con bloques `@defer`

La hydration incremental se integra con las **deferrable views** (`@defer`), permitiendo que incluso contenido “above the fold” (visible al inicio) pueda diferirse sin causar parpadeos:  

```html
@defer (hydrate on interaction)
  <comments-section />
@placeholder
  <p>Cargando comentarios...</p>
@end
```

- `hydrate on interaction`: el bloque se hidrata solo cuando el usuario interactúa.  
- `hydrate when visible`: se hidrata al entrar en el viewport.  
- `hydrate never`: permanece estático, útil para contenido puramente decorativo.  

### 9.4.6. Estabilidad en Angular 20

En Angular 19, la hydration incremental estaba en **preview**. Con Angular 20, pasa a ser **estable y lista para producción**.  
- Compatible con SSR, prerender y PWA.  
- Integrada con **event replay**: los eventos del usuario (clics, scrolls) se almacenan y se reproducen cuando el bloque se hidrata.  
- Soportada oficialmente en la documentación y herramientas de Angular.  

### 9.4.7. Buenas prácticas

- **Priorizar contenido crítico**: hidratar primero lo que impacta en LCP (ej. hero image, título principal).  
- **Diferir contenido secundario**: comentarios, secciones de recomendaciones, widgets.  
- **Combinar con loaders externos**: imágenes y recursos pueden diferirse junto con la lógica de sus componentes.  
- **Medir con Lighthouse y Angular DevTools**: validar mejoras en TTI y LCP tras aplicar hydration incremental.  

**Manejo de errores de hydration:**
Si Angular detecta problemas durante la hydration (por ejemplo, diferencias entre el HTML renderizado en servidor y el esperado en cliente), los errores aparecerán en la consola y en Angular DevTools. Revisa la pestaña de hydration en DevTools para identificar componentes problemáticos y corrige el uso de APIs incompatibles o dependencias no sincronizadas.


## 9.5. Comparación entre hydration parcial e incremental

### 9.5.1. Contexto: ¿por qué existen distintos tipos de hydration?

Cuando Angular renderiza en servidor (SSR) o prerenderiza (SSG), el navegador recibe un HTML ya poblado. Para que ese HTML se vuelva interactivo, Angular debe **hidratarlo**: conectar el DOM estático con la lógica de los componentes.  
- En un modelo clásico, la hydration era **total**: toda la aplicación se rehidrataba de golpe.  
- Para mejorar rendimiento, surgieron enfoques más finos: **hydration parcial** y **hydration incremental**.  

### 9.5.2. Hydration parcial

La **hydration parcial** consiste en **hidratar solo una parte de la aplicación**, dejando otras secciones sin hidratar (o directamente renderizadas solo en servidor).  

- **Cómo funciona**:  
  - Se seleccionan manualmente zonas de la aplicación que no necesitan interactividad.  
  - Esas zonas permanecen como HTML estático, sin lógica de Angular.  
  - Solo se hidratan los componentes que realmente requieren interacción.  

- **Ventajas**:  
  - Reduce el coste de JavaScript en cliente.  
  - Útil para contenido puramente estático (ej. artículos, secciones decorativas).  

- **Limitaciones**:  
  - Las partes no hidratadas nunca serán interactivas.  
  - Requiere decisiones manuales y puede fragmentar la experiencia.  

👉 Ejemplo: en un blog, el cuerpo del artículo se deja sin hidratar (solo HTML), mientras que los comentarios sí se hidratan.  

### 9.5.3. Hydration incremental

La **hydration incremental** es más sofisticada: en lugar de decidir qué nunca se hidrata, Angular **hidrata progresivamente distintas secciones cuando se cumplen ciertos triggers**.  

- **Cómo funciona**:  
  - Todo el HTML se entrega renderizado desde el servidor.  
  - Angular marca bloques diferibles (`@defer`) como “pendientes de hidratar”.  
  - La hydration se activa **cuando ocurre un evento**:  
    - El bloque entra en el viewport (`hydrate when visible`).  
    - El usuario interactúa (`hydrate on interaction`).  
    - Se cumple una condición de red o tiempo (`hydrate after idle`).  

- **Ventajas**:  
  - Permite que todo el contenido sea interactivo, pero solo cuando hace falta.  
  - Mejora métricas como **TTI (Time to Interactive)** y **LCP**.  
  - Evita *layout shifts* incluso en contenido “above the fold”.  

- **Limitaciones**:  
  - Más compleja de configurar.  
  - Requiere pensar en triggers adecuados para cada bloque.  

👉 Ejemplo: en un e‑commerce, la ficha del producto se hidrata al instante, pero la sección de “productos recomendados” se hidrata solo cuando el usuario hace scroll.  

### 9.5.4. Comparación directa

| Aspecto | Hydration parcial | Hydration incremental |
|---------|------------------|-----------------------|
| **Definición** | Solo se hidratan algunas partes, otras quedan estáticas | Toda la app puede hidratarse, pero de forma progresiva |
| **Interactividad** | Limitada: lo no hidratado nunca será interactivo | Completa: todo puede ser interactivo, pero bajo demanda |
| **Control** | Manual: el desarrollador decide qué no se hidrata | Automático: triggers (`@defer`) controlan cuándo hidratar |
| **Rendimiento inicial** | Muy alto (menos JS ejecutado) | Alto, con balance entre velocidad e interactividad |
| **Casos de uso** | Contenido estático permanente (blogs, landing pages) | Apps dinámicas grandes (e‑commerce, dashboards, redes sociales) |

### 9.5.5. Resumen

- La **hydration parcial** es útil cuando hay secciones que nunca necesitan interactividad: se dejan como HTML estático y se hidrata solo lo esencial.  
- La **hydration incremental** es ideal para aplicaciones complejas: todo puede ser interactivo, pero Angular lo activa progresivamente según la interacción o visibilidad.  
- En Angular 20, la **hydration incremental es estable y recomendada** para proyectos enterprise, ya que ofrece un equilibrio perfecto entre rendimiento y experiencia de usuario.  

## 9.6. Estrategias para mejorar métricas clave: FCP, TTI y LCP

Las **Core Web Vitals** son indicadores esenciales de la experiencia de usuario. En Angular 20, gracias a la integración nativa de **SSR, prerender e hydration incremental**, tenemos nuevas herramientas para optimizarlas.  

### 9.6.1. First Contentful Paint (FCP)

El **FCP** mide el tiempo que tarda en aparecer el **primer contenido visible** en pantalla (texto, imagen, SVG).  

### Estrategias en Angular 20:
- **SSR o prerender**: entregar HTML ya renderizado desde el servidor reduce drásticamente el tiempo hasta el primer píxel.  
- **Optimización de CSS crítico**: cargar solo los estilos necesarios para el *above the fold*.  
- **Uso de `priority` en imágenes clave** (`NgOptimizedImage`): asegura que la imagen principal se precargue.  
- **Evitar bloqueos por JavaScript**: dividir el bundle en *chunks* y cargar diferido lo no esencial.  

👉 Ejemplo: prerenderizar la home y marcar el hero banner como `priority` garantiza que el usuario vea contenido en milisegundos.  

### 9.6.2. Time to Interactive (TTI)

El **TTI** mide cuánto tarda la página en volverse **plenamente interactiva**. Una app puede mostrar contenido rápido (buen FCP), pero si el JS sigue bloqueando, el usuario no puede interactuar.  

### Estrategias en Angular 20:
- **Hydration incremental**: en lugar de hidratar toda la app de golpe, Angular activa progresivamente los componentes.  
- **Uso de `@defer`**: diferir la carga de secciones no críticas (ej. comentarios, recomendaciones).  
- **Event replay**: Angular almacena interacciones del usuario y las reproduce cuando el bloque se hidrata, evitando frustración.  
- **Optimización de dependencias**: eliminar librerías pesadas o cargarlas bajo demanda.  

👉 Ejemplo: en un e‑commerce, el detalle del producto se hidrata primero, mientras que la sección de “productos relacionados” se hidrata solo al hacer scroll.  

### 9.6.3. Largest Contentful Paint (LCP)

El **LCP** mide el tiempo que tarda en renderizarse el **elemento más grande y relevante** de la vista (generalmente una imagen o título principal).  

### Estrategias en Angular 20:
- **SSR + `priority`**: renderizar en servidor y precargar la imagen principal mejora el LCP de forma inmediata.  
- **Formatos modernos (WebP, AVIF)**: reducen el peso de las imágenes hasta un 50–80%.  
- **CDNs de imágenes**: sirven versiones optimizadas según dispositivo y red.  
- **Reservar espacio con `width` y `height`**: evita *layout shifts* que retrasan el LCP.  
- **Optimización de fuentes**: usar `font-display: swap` para que el texto principal aparezca sin bloqueo.  

👉 Ejemplo: en una landing page, el hero image en AVIF precargado con `priority` puede reducir el LCP en más de un 40%.  

### 9.6.4. Estrategias transversales

- **Transferencia de caché de HttpClient**: Angular 20 permite cachear respuestas en SSR y reutilizarlas en cliente.  
- **División de rutas con RenderMode**: usar prerender para páginas estáticas, SSR para dinámicas y CSR para privadas.  
- **Medición continua**: integrar Lighthouse y Angular DevTools en CI/CD para detectar regresiones.  
- **Uso de Signals y APIs reactivas**: reducen renders innecesarios y mejoran la fluidez.  


## 9.7. SSR con APIs dinámicas y consideraciones de SEO avanzado

### 9.7.1. El reto de las APIs dinámicas en SSR

Cuando usamos **Server-Side Rendering (SSR)** en Angular, el servidor no solo genera el HTML inicial, sino que también puede **consultar APIs externas o internas** antes de enviar la respuesta al cliente. Esto permite:  
- Renderizar contenido personalizado o actualizado en tiempo real.  
- Evitar que el navegador tenga que hacer múltiples llamadas iniciales.  
- Mejorar la experiencia de usuario y el SEO, ya que los crawlers reciben el contenido ya resuelto.  

👉 Ejemplo: en un e‑commerce, la página de producto puede renderizarse en servidor con datos actualizados de inventario y precio obtenidos desde una API.  

### 9.7.2. Cómo integrar APIs dinámicas en Angular SSR

Angular 20 ofrece varias herramientas para trabajar con datos dinámicos en SSR:  

- **Uso de `HttpClient` con Transfer State**:  
  - En el servidor, las peticiones se resuelven y se serializan en el HTML inicial.  
  - En el cliente, Angular reutiliza esos datos en lugar de volver a llamar a la API.  
  - Esto reduce la duplicación de requests y acelera la hidratación.  

```ts
import { HttpClient } from '@angular/common/http';
import { Component, inject } from '@angular/core';

@Component({
  selector: 'app-product',
  template: `<h1>{{ product?.name }}</h1>`
})
export class ProductComponent {
  product: any;
  private http = inject(HttpClient);

  ngOnInit() {
    this.http.get('/api/products/123').subscribe(data => this.product = data);
  }
}
```

- **Tokens de inyección en SSR** (`REQUEST`, `RESPONSE_INIT`):  
  Permiten acceder a cabeceras, cookies y contexto de la petición para personalizar la respuesta.  

- **Rutas con `RenderMode.Server`**:  
  Se configuran en `app.routes.server.ts` para indicar que ciertas páginas deben renderizarse siempre en servidor, ideal para contenido dinámico.  

### 9.7.3. Consideraciones avanzadas de SEO

El SSR con APIs dinámicas no solo mejora la experiencia de usuario, también es clave para el **SEO avanzado**:  

- **Metadatos dinámicos**:  
  - Usar los servicios `Title` y `Meta` de Angular para actualizar `<title>`, `<meta name="description">` y etiquetas Open Graph/Twitter Cards en función de los datos obtenidos de la API.  
  - Ejemplo: cada producto debe tener su propio título y descripción únicos.  

- **Canonical y hreflang**:  
  - Añadir etiquetas `<link rel="canonical">` para evitar contenido duplicado.  
  - Usar `hreflang` en sitios multilingües para guiar a Google en la indexación correcta.  

- **Rich Snippets y Schema.org**:  
  - Inyectar datos estructurados JSON-LD en el SSR para mejorar la visibilidad en resultados enriquecidos.  

- **Gestión de errores y códigos de estado**:  
  - Configurar `status` en las rutas de servidor para devolver `404` o `301` cuando corresponda.  
  - Esto evita indexar páginas inexistentes o duplicadas.  

- **Velocidad y Core Web Vitals**:  
  - Usar prerender híbrido para páginas estáticas y SSR para dinámicas.  
  - Minimizar el tiempo de respuesta de las APIs, ya que impacta directamente en el LCP.  

### 9.7.4. Ejemplo narrativo

Imagina un portal de noticias:  
- Cada artículo se renderiza en servidor con su título, descripción y contenido obtenidos de la API.  
- El SSR genera también las etiquetas Open Graph para que, al compartir en redes sociales, aparezca la imagen y resumen correctos.  
- Si un artículo no existe, el servidor devuelve un `404` real, evitando que Google indexe páginas vacías.  
- Mientras tanto, las secciones estáticas como “Acerca de” o “Contacto” se prerenderizan en build, reduciendo carga en el servidor.  


## 9.8. Uso de Angular DevTools y Chrome Profiler para SSR

### 9.8.1. ¿Por qué es importante el profiling en SSR?

Cuando habilitamos **Server-Side Rendering (SSR)**, la aplicación Angular no solo se ejecuta en el navegador, sino también en el servidor. Esto introduce nuevos retos:  
- ¿Dónde se están produciendo los cuellos de botella: en el servidor o en el cliente?  
- ¿Qué parte del tiempo de carga corresponde al renderizado inicial en servidor y cuál a la hidratación en cliente?  
- ¿Cómo afectan los ciclos de detección de cambios y la carga de datos a métricas como **TTI** o **LCP**?  

👉 Aquí entran en juego **Angular DevTools** y el **Chrome Profiler**, que nos permiten observar la aplicación desde dentro y correlacionar el trabajo de Angular con el del navegador.  

### 9.8.2. Angular DevTools: visión interna del framework

**Angular DevTools** es la extensión oficial para Chrome y Edge que ofrece:  
- **Árbol de componentes**: inspección jerárquica de componentes, inputs, outputs y estado.  
- **Profiler de Angular**: muestra ciclos de detección de cambios, tiempos de renderizado y triggers de actualización.  
- **Integración con SSR/Hydration**: permite ver qué componentes se hidratan primero y cómo se distribuye la carga entre servidor y cliente.  

### Ejemplo de uso en SSR:
- Tras habilitar SSR, puedes abrir DevTools y observar cómo el **AppComponent** ya está renderizado en servidor.  
- Al interactuar con la página, el profiler muestra qué componentes se hidratan y cuánto tarda cada ciclo de detección.  
- Esto ayuda a identificar componentes que deberían diferirse con `@defer` o que necesitan `ChangeDetectionStrategy.OnPush`.  

### 9.8.3. Chrome Profiler: visión del navegador

El **Chrome DevTools Performance Panel** ofrece una visión más amplia, a nivel de navegador:  
- **Flame charts**: muestran la ejecución de funciones, incluyendo las llamadas internas de Angular.  
- **Tracks personalizados de Angular**: en Angular 20, Chrome Profiler integra un track específico que refleja eventos del framework (ciclos de cambio, bindings, hooks).  
- **Correlación Angular + navegador**: puedes ver cuándo Angular ejecuta detección de cambios y cómo eso se relaciona con tareas del navegador como layout, paint o recálculo de estilos.  

### Ejemplo de uso en SSR:
- Grabar un perfil de la carga inicial.  
- Observar cómo el navegador recibe el HTML prerenderizado (FCP rápido).  
- Analizar el momento exacto en que Angular comienza la **hydration** y cómo impacta en el **TTI**.  
- Detectar si hay ciclos de cambio innecesarios que retrasan la interactividad.  

### 9.8.4. Estrategia combinada: DevTools + Profiler

- **Angular DevTools** → visión semántica del framework (componentes, bindings, ciclos de cambio).  
- **Chrome Profiler** → visión técnica del navegador (CPU, memoria, layout, paint).  
- **Combinación**:  
  - Angular DevTools te dice *qué componente* está causando trabajo extra.  
  - Chrome Profiler te dice *cómo* ese trabajo impacta en el rendimiento global del navegador.  

👉 Juntos, permiten un diagnóstico completo: desde el código Angular hasta el renderizado en el navegador.  

### 9.8.5. Buenas prácticas al perfilar SSR

- **Perfilar en modo desarrollo y producción**: en dev se ven más detalles, pero en prod se mide el rendimiento real.  
- **Analizar la carga inicial**: identificar el momento exacto en que se produce el FCP y el LCP.  
- **Medir la hidratación**: comprobar cuánto tarda Angular en conectar el HTML SSR con la lógica interactiva.  
- **Buscar ciclos innecesarios**: usar `OnPush` y `Signals` para reducir trabajo de detección de cambios.  
- **Comparar escenarios**: SSR puro vs prerender vs híbrido, para decidir la mejor estrategia.  


## 9.9. Casos de uso en aplicaciones de alto tráfico y escalabilidad global

### 9.9.1. El reto del alto tráfico en aplicaciones modernas

Cuando hablamos de aplicaciones con millones de usuarios concurrentes —e‑commerce internacionales, medios de comunicación, plataformas de streaming o redes sociales—, el **rendimiento y la escalabilidad** dejan de ser un detalle técnico y se convierten en un factor estratégico.  
- **Latencia mínima**: los usuarios esperan tiempos de carga instantáneos, incluso en dispositivos móviles y redes lentas.  
- **Disponibilidad global**: el contenido debe servirse de forma eficiente desde múltiples regiones.  
- **SEO y Core Web Vitals**: en sitios públicos, la visibilidad en buscadores depende de métricas como LCP, FCP y CLS.  

👉 Aquí es donde Angular 20, con **SSR, prerender híbrido e hydration incremental**, ofrece un marco sólido para escalar aplicaciones a nivel global.  

### 9.9.2. Casos de uso típicos

### 🛒 E‑commerce internacional
- **SSR dinámico** para páginas de producto: precios, stock y promociones se renderizan en servidor con datos actualizados.  
- **Prerender híbrido** para páginas estáticas como “Sobre nosotros” o “FAQ”.  
- **Hydration incremental** para diferir secciones secundarias (ej. reseñas, productos recomendados).  
- **CDNs globales** para servir imágenes optimizadas (WebP/AVIF) y HTML prerenderizado.  

### 📰 Medios digitales y portales de noticias
- **Prerender** de artículos evergreen (contenido que no cambia).  
- **SSR** para noticias de última hora, garantizando indexación inmediata en buscadores.  
- **SEO avanzado**: metadatos dinámicos, Open Graph y Schema.org inyectados en SSR.  
- **Escalabilidad global**: uso de caching en CDN para distribuir contenido en múltiples regiones.  

### 🎥 Plataformas de streaming y entretenimiento
- **SSR** para páginas de detalle de películas/series, con metadatos optimizados para SEO.  
- **Hydration incremental** para diferir la carga de recomendaciones, trailers o comentarios.  
- **Service Workers (PWA)** para mejorar la experiencia offline y reducir carga en servidores.  

### 📊 Dashboards corporativos y SaaS
- **CSR** en áreas privadas con datos sensibles.  
- **SSR** en páginas públicas de marketing y documentación.  
- **Hydration incremental** en widgets secundarios, optimizando TTI en paneles complejos.  

### 9.9.3. Estrategias de escalabilidad global

- **Hybrid Rendering**: combinar SSR, prerender y CSR según la naturaleza de cada ruta.  
- **CDN + Edge Rendering**: distribuir contenido estático y prerenderizado en nodos cercanos al usuario.  
- **Transfer State con HttpClient**: cachear datos en SSR y reutilizarlos en cliente, reduciendo requests duplicados.  
- **Fallbacks inteligentes**: prerender de rutas críticas y fallback a SSR o CSR para rutas dinámicas.  
- **Optimización de recursos**: uso de loaders de imágenes, compresión y formatos modernos.  

### 9.9.4. Ejemplo narrativo

Imagina un **marketplace global**:  
1. El usuario en España accede a la home → prerender estático servido desde un CDN europeo.  
2. Navega a un producto → SSR dinámico obtiene precio y stock en tiempo real desde la API.  
3. La sección de “productos recomendados” se hidrata de forma incremental al hacer scroll.  
4. En visitas posteriores, la PWA cachea imágenes y datos, reduciendo la carga en servidores centrales.  

👉 Resultado: experiencia rápida, consistente y escalable, incluso con millones de usuarios simultáneos en distintas regiones.  

## 9.10. Buenas prácticas de mantenimiento y despliegue de SSR/Hydration

### 9.10.1. Mantenimiento continuo de proyectos SSR/Hydration

El renderizado en servidor y la hydration incremental aportan grandes beneficios, pero también introducen complejidad. Para mantener la aplicación sana a largo plazo:  

- **Mantener dependencias actualizadas**  
  Angular 20 introduce cambios importantes en SSR e hydration. Es fundamental mantener versiones alineadas de Angular, Node.js y TypeScript.  

- **Monitorizar errores de hydration**  
  Angular 18+ incluye visualización de errores de hydration en DevTools. Esto permite identificar componentes que no se hidratan correctamente (ej. por usar APIs de navegador en servidor o librerías incompatibles).  

- **Evitar APIs no compatibles con servidor**  
  No usar directamente `window`, `document` o `navigator` en componentes. Si es necesario, encapsularlos en servicios que se ejecuten solo en cliente.  

- **Transferencia de estado (Transfer State)**  
  Usar `HttpClient` con caché de SSR para evitar llamadas duplicadas en cliente y servidor.  

- **Pruebas en entornos reales**  
  Algunos problemas de hydration solo aparecen en producción (ej. CDNs que alteran comentarios HTML necesarios para hydration). Es recomendable probar en staging con la misma infraestructura que en producción.  

### 9.10.2. Buenas prácticas de despliegue

- **Uso de CDNs y edge rendering**  
  Distribuir contenido prerenderizado en nodos cercanos al usuario para reducir latencia global.  

- **Estrategia híbrida de renderizado**  
  - **Prerender** para páginas estáticas.  
  - **SSR** para contenido dinámico indexable.  
  - **CSR** para áreas privadas o dashboards.  

- **Gestión de cabeceras y códigos de estado**  
  Configurar correctamente `status` y `headers` en rutas SSR para SEO avanzado (ej. devolver 404 reales en páginas inexistentes).  

- **Automatización en CI/CD**  
  - Ejecutar `npm run build:ssr` en pipelines.  
  - Validar Core Web Vitals con Lighthouse en cada despliegue.  
  - Incluir pruebas E2E que verifiquen que el HTML inicial contiene el contenido esperado.  

- **Fallbacks inteligentes**  
  Configurar `PrerenderFallback` para rutas dinámicas no prerenderizadas, eligiendo entre SSR, CSR o “none” según el caso.  

### 9.10.3. Monitorización y observabilidad

- **Angular DevTools + Chrome Profiler**: medir ciclos de cambio, tiempos de hydration y correlación con métricas de navegador.  
- **Logs de servidor**: registrar tiempos de renderizado SSR y errores de hydration.  
- **Alertas en producción**: detectar caídas de rendimiento o fallos de renderizado antes de que impacten a usuarios.

