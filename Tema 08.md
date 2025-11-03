# 8. Optimización de recursos e imágenes

## 8.1. Introducción al nuevo `NgOptimizedImage` en Angular 20

Las imágenes son uno de los recursos más pesados en la web: representan en promedio casi **1 MB por página** y suelen ser el **elemento LCP (Largest Contentful Paint)** en más del 70% de los sitios. Esto significa que optimizarlas no es un lujo, sino una necesidad para mejorar la velocidad de carga, la estabilidad visual y la experiencia de usuario.  

Con Angular 20, la directiva **`NgOptimizedImage`** se consolida como la forma recomendada de manejar imágenes, integrando prácticas modernas de optimización directamente en el framework.

### 8.1.1. ¿Qué es `NgOptimizedImage`?

- Es una **directiva nativa de Angular** que reemplaza el uso tradicional de `src` por `ngSrc`.  
- Aplica automáticamente **mejores prácticas de optimización de imágenes**:  
  - Preload inteligente de la imagen LCP.  
  - Lazy loading para imágenes no críticas.  
  - Reserva de espacio con `width` y `height` para evitar *layout shifts*.  
  - Soporte para *responsive images* (`sizes`, `srcset`).  
  - Placeholders de baja resolución (blur-up) cuando se usa un loader compatible.  

👉 En resumen: Angular se encarga de aplicar las técnicas que antes debíamos configurar manualmente.

### 8.1.2. Uso básico

```html
<img ngSrc="assets/banner.jpg" width="1200" height="600" priority />
```

- `ngSrc`: activa la directiva.  
- `width` y `height`: reservan espacio para evitar saltos de layout (CLS).  
- `priority`: marca la imagen como crítica (ej. el banner principal).  

### 8.1.3. Imágenes responsivas

Para imágenes que deben adaptarse al viewport:

```html
<img
  ngSrc="assets/product.jpg"
  width="800"
  height="600"
  sizes="(max-width: 600px) 100vw, 50vw"
/>
```

👉 Angular genera automáticamente los atributos `srcset` y selecciona la mejor versión según el dispositivo.

### 8.1.4. Modo *fill* (relleno de contenedor)

Cuando queremos que la imagen se comporte como un *background*:

```html
<div class="hero">
  <img ngSrc="assets/hero.jpg" fill priority />
</div>
```

- `fill`: hace que la imagen ocupe todo el contenedor padre.  
- Útil para *hero sections* o banners de fondo.  

### 8.1.5. Placeholders automáticos

Si usamos un **loader compatible con redimensionamiento** (ej. un CDN de imágenes), podemos añadir:

```html
<img ngSrc="assets/gallery.jpg" width="600" height="400" placeholder />
```

👉 Angular solicitará una versión reducida de la imagen y la mostrará difuminada mientras carga la versión final.

### 8.1.6. Beneficios principales

- **Mejora de Web Vitals**: optimiza LCP y CLS automáticamente.  
- **Menos configuración manual**: Angular aplica las mejores prácticas por defecto.  
- **Integración con SSR**: genera etiquetas `<link rel="preload">` en renderizado del lado del servidor.  
- **Compatibilidad progresiva**: se puede migrar gradualmente desde `src` a `ngSrc`.  

## 8.2. Configuración básica y avanzada de `NgOptimizedImage`

La directiva **`NgOptimizedImage`** no es solo un reemplazo de `src` por `ngSrc`. Es un **ecosistema de configuración** que permite a Angular aplicar automáticamente buenas prácticas de optimización de imágenes, pero también ofrece un nivel de personalización avanzado para escenarios complejos: desde catálogos de e‑commerce hasta aplicaciones con SSR y CDNs de imágenes.  

### 8.2.1. Configuración básica: lo mínimo necesario

En su forma más simple, basta con:  
- Usar `ngSrc` en lugar de `src`.  
- Definir `width` y `height`.  
- Marcar con `priority` la imagen principal de la página.  

Ejemplo:

```html
<img ngSrc="assets/banner.jpg" width="1200" height="600" priority />
```

Esto ya activa:  
- **Lazy loading automático** en imágenes no críticas.  
- **Preload inteligente** de la imagen marcada como `priority`.  
- **Reserva de espacio** para evitar *layout shifts* (CLS).  

👉 Con solo tres atributos, Angular aplica optimizaciones que antes requerían configuración manual.

### 8.2.2. Configuración intermedia: imágenes responsivas

En sitios modernos, una misma imagen debe adaptarse a distintos tamaños de pantalla. Con `NgOptimizedImage`, basta con añadir `sizes`:

```html
<img
  ngSrc="assets/product.jpg"
  width="800"
  height="600"
  sizes="(max-width: 600px) 100vw, 50vw"
/>
```

- Angular genera automáticamente el `srcset`.  
- El navegador selecciona la mejor versión según el ancho de pantalla.  
- Se reduce el consumo de datos en móviles y se mejora la calidad en pantallas grandes.  

👉 Esto elimina la necesidad de generar manualmente múltiples versiones de la imagen.

### 8.2.3. Configuración avanzada: modo *fill*

Cuando la imagen debe comportarse como un **background adaptable**:

```html
<div class="hero">
  <img ngSrc="assets/hero.jpg" fill priority />
</div>
```

- `fill` hace que la imagen ocupe todo el contenedor padre.  
- Se combina con CSS (`object-fit: cover;`) para mantener proporciones.  
- Ideal para *hero sections*, banners y cabeceras.  

### 8.2.4. Placeholders y carga progresiva

Con `placeholder`, Angular puede mostrar una versión reducida y difuminada de la imagen mientras carga la definitiva:

```html
<img ngSrc="assets/gallery.jpg" width="600" height="400" placeholder />
```

- Mejora la **percepción de velocidad**.  
- Evita pantallas en blanco en galerías o catálogos.  
- Requiere un **loader compatible** (ej. Cloudinary, ImageKit, Akamai).  

### 8.2.5. Configuración global con `IMAGE_CONFIG`

Para proyectos grandes, podemos definir reglas globales con el provider `IMAGE_CONFIG`:

```ts
import { IMAGE_CONFIG } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: IMAGE_CONFIG,
      useValue: {
        placeholderSize: 30, // tamaño en px del placeholder
        disableImageSizeWarning: false, // mostrar advertencias en dev
        breakpoints: [320, 640, 1024, 1440] // tamaños para srcset
      }
    }
  ]
});
```

Opciones comunes:  
- **`placeholderSize`**: controla la resolución del placeholder.  
- **`disableImageSizeWarning`**: útil en desarrollo para detectar imágenes sin `width`/`height`.  
- **`breakpoints`**: define los anchos que Angular usará para generar `srcset`.  

👉 Esto permite estandarizar la optimización en toda la aplicación.

### 8.2.6. Integración con loaders externos (CDNs de imágenes)

En entornos enterprise, lo habitual es usar un **CDN de imágenes** que genere versiones optimizadas al vuelo. Angular soporta loaders como **ImageKit**, **Cloudinary** o personalizados.  

Ejemplo con ImageKit:

```ts
import { provideImageKitLoader } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    provideImageKitLoader('https://ik.imagekit.io/myaccount/')
  ]
});
```

Ahora, al usar `ngSrc="product.jpg"`, Angular construye automáticamente la URL optimizada desde el CDN.  

### 8.2.7. Integración con SSR (Server-Side Rendering)

Cuando usamos Angular Universal:  
- `NgOptimizedImage` genera etiquetas `<link rel="preload">` para imágenes críticas.  
- Mejora el **LCP** en el primer renderizado.  
- Permite que los crawlers indexen imágenes optimizadas desde el inicio.  

👉 Esto impacta directamente en SEO y Core Web Vitals.  

### 8.2.8. Escenarios reales de uso

- **E‑commerce**: catálogos con cientos de imágenes → lazy loading + placeholders.  
- **Aplicaciones de contenido**: blogs o medios → responsive + breakpoints globales.  
- **Dashboards corporativos**: iconografía y gráficos → `priority` en elementos clave.  
- **Landing pages**: banners y hero sections → `fill` + preload.  

### 8.2.9. Buenas prácticas

- Siempre definir `width` y `height` (o `fill`).  
- Usar `priority` solo en la imagen principal (evitar abusar).  
- Configurar `sizes` en imágenes responsivas.  
- Aprovechar loaders externos para catálogos grandes.  
- Revisar advertencias en consola durante desarrollo (Angular avisa si falta configuración).  


## 8.3.Comparación con técnicas manuales de optimización de imágenes

Antes de la llegada de `NgOptimizedImage`, los desarrolladores tenían que aplicar manualmente una serie de prácticas para optimizar imágenes: generar múltiples versiones, configurar `srcset`, aplicar lazy loading, reservar espacio con CSS, etc. Angular 20 integra todo esto en una directiva única, reduciendo la complejidad y los errores humanos.  

### 8.3.1. Técnicas manuales tradicionales

#### 🔹 Generación manual de múltiples tamaños
- Crear varias versiones de la misma imagen (ej. 320px, 640px, 1280px).  
- Configurar manualmente el atributo `srcset` y `sizes`.  
- Problema: requiere procesos de *build* adicionales o un CDN configurado.  

#### 🔹 Lazy loading manual
- Añadir `loading="lazy"` en cada `<img>`.  
- Funciona, pero no distingue entre imágenes críticas (LCP) y no críticas.  

#### 🔹 Reserva de espacio con CSS
- Definir `width` y `height` en CSS o usar contenedores con `aspect-ratio`.  
- Si se omite, se producen *layout shifts* (CLS).  

#### 🔹 Placeholders personalizados
- Generar imágenes en baja resolución o usar SVGs difuminados como fondo.  
- Requiere lógica adicional en el frontend o en el pipeline de imágenes.  

#### 🔹 Preload manual
- Añadir `<link rel="preload">` en el `<head>` para imágenes críticas.  
- Difícil de mantener en aplicaciones dinámicas.  

### 8.3.2. Enfoque con `NgOptimizedImage`

La directiva automatiza todas estas tareas:  

- **`ngSrc`**: reemplaza `src` y activa la optimización.  
- **`priority`**: marca automáticamente la imagen LCP para preload.  
- **`width` y `height`**: obligatorios, garantizan reserva de espacio.  
- **`sizes`**: Angular genera `srcset` automáticamente.  
- **`placeholder`**: activa placeholders de baja resolución si hay un loader configurado.  
- **Integración con SSR**: genera `<link rel="preload">` en renderizado del servidor.  

👉 En lugar de múltiples configuraciones dispersas, todo se concentra en una directiva declarativa y coherente.  

### 8.3.3. Comparación directa

| Aspecto | Técnicas manuales | `NgOptimizedImage` |
|---------|------------------|--------------------|
| **Responsive (`srcset`)** | Configuración manual de múltiples tamaños | Generado automáticamente con `sizes` |
| **Lazy loading** | `loading="lazy"` en cada imagen | Automático, con distinción entre imágenes críticas y no críticas |
| **Preload de LCP** | `<link rel="preload">` manual en el `<head>` | Automático con `priority` |
| **Reserva de espacio** | CSS o `aspect-ratio` manual | Obligatorio con `width`/`height` o `fill` |
| **Placeholders** | Generados manualmente o con lógica extra | Automáticos con `placeholder` y loader |
| **SSR** | Preload manual y configuración extra | Integración nativa con Angular Universal |
| **Mantenibilidad** | Propenso a errores y repetición | Declarativo y centralizado |

### 8.3.4. Escenarios reales

- **Catálogo de e‑commerce**:  
  - Manual: generar decenas de versiones de cada producto y configurar `srcset`.  
  - Con `NgOptimizedImage`: basta con `ngSrc`, `width`, `height` y `sizes`.  

- **Landing page con hero image**:  
  - Manual: añadir preload en `<head>`, definir CSS para reservar espacio.  
  - Con `NgOptimizedImage`: `ngSrc="hero.jpg" fill priority`.  

- **Galería de imágenes**:  
  - Manual: placeholders en baja resolución generados en el pipeline.  
  - Con `NgOptimizedImage`: `placeholder` + loader externo (ej. Cloudinary).  


## 8.4. Carga diferida (lazy loading) y diferimiento condicional de recursos

La **carga diferida** es una técnica que retrasa la descarga o inicialización de recursos hasta que son realmente necesarios. En Angular, esto se aplica tanto a **módulos y componentes** como a **recursos estáticos** (imágenes, scripts, estilos). El **diferimiento condicional** va un paso más allá: decide dinámicamente qué recursos cargar en función del contexto (dispositivo, red, rol del usuario, etc.).  

### 8.4.1. Carga diferida en Angular

#### 🔹 Lazy loading de módulos y componentes
- **Módulos**: se cargan solo cuando el usuario navega a la ruta correspondiente (`loadChildren`).  
- **Componentes standalone**: se cargan bajo demanda con `loadComponent`.  

Ejemplo:

```ts
export const routes: Routes = [
  {
    path: 'reports',
    loadChildren: () =>
      import('./reports/reports.routes').then(m => m.REPORTS_ROUTES)
  },
  {
    path: 'settings',
    loadComponent: () =>
      import('./settings/settings.component').then(m => m.SettingsComponent)
  }
];
```

👉 Esto reduce el *bundle* inicial y acelera el arranque de la aplicación.  

### 8.4.2. Carga diferida de imágenes y medios

Con **`NgOptimizedImage`**, Angular aplica lazy loading automáticamente en imágenes no críticas.  
- Solo se cargan cuando están cerca del viewport.  
- Se pueden marcar imágenes críticas con `priority` para que se precarguen.  

Ejemplo:

```html
<img ngSrc="assets/gallery.jpg" width="600" height="400" />
```

👉 Angular añade `loading="lazy"` por defecto, optimizando el consumo de datos.  

### 8.4.3. Diferimiento condicional de recursos

El diferimiento condicional consiste en **cargar recursos solo si se cumplen ciertas condiciones**. Algunos ejemplos:  

- **Condiciones de red**:  
  - Si la conexión es lenta (`navigator.connection.effectiveType` = `2g` o `3g`), no precargar imágenes pesadas ni módulos secundarios.  
- **Dispositivo**:  
  - En móviles, cargar versiones reducidas de imágenes o desactivar animaciones pesadas.  
- **Rol del usuario**:  
  - Precargar el módulo de administración solo si el usuario es administrador.  
- **Interacción previa**:  
  - Precargar el módulo de checkout solo cuando el usuario añade un producto al carrito.  

Ejemplo con `canMatch` y carga condicional:

```ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes').then(m => m.ADMIN_ROUTES),
    canMatch: [() => inject(AuthService).isAdmin()]
  }
];
```

👉 Aquí, el módulo de administración solo se carga si el usuario tiene rol de administrador.  

### 8.4.4. Diferimiento de scripts y estilos

Además de Angular, podemos aplicar diferimiento en recursos externos:  
- **Scripts**: usar `defer` o `async` en `<script>`.  
- **Estilos**: cargar CSS no crítico de forma diferida (`media="print"` y luego cambiar a `all`).  
- **Fuentes web**: usar `font-display: swap` para evitar bloqueos en el renderizado.  

### 8.4.5. Beneficios combinados

- **Tiempo de carga inicial reducido**: solo se cargan los recursos esenciales.  
- **Uso eficiente de datos**: ideal para usuarios móviles o con planes limitados.  
- **Mejor experiencia de usuario**: la aplicación responde más rápido a las interacciones iniciales.  
- **Escalabilidad**: aplicaciones grandes pueden crecer sin penalizar el rendimiento.  

## 8.5. Uso de directivas para retrasar imágenes no visibles en pantalla

En aplicaciones con mucho contenido visual (catálogos, galerías, feeds sociales), cargar todas las imágenes de golpe puede saturar la red y ralentizar la experiencia inicial. La solución es **retrasar la carga de imágenes que aún no son visibles en el viewport**, técnica conocida como *lazy loading de imágenes*.  

Angular 20 facilita este patrón gracias a la directiva **`NgOptimizedImage`**, que ya incorpora lazy loading por defecto, y a la posibilidad de crear **directivas personalizadas** para escenarios más avanzados.

### 8.5.1. Lazy loading automático con `NgOptimizedImage`

Cuando usamos `ngSrc`, Angular aplica automáticamente `loading="lazy"` a las imágenes que no están marcadas como prioritarias:

```html
<img ngSrc="assets/gallery1.jpg" width="600" height="400" />
<img ngSrc="assets/gallery2.jpg" width="600" height="400" />
<img ngSrc="assets/gallery3.jpg" width="600" height="400" />
```

👉 Estas imágenes solo se descargan cuando el usuario hace scroll y se acercan al viewport.  
👉 Si una imagen es crítica (ej. el banner principal), se marca con `priority` para que se cargue inmediatamente.  

### 8.5.2. Directivas personalizadas con Intersection Observer

Para casos donde necesitamos un control más granular (ej. cargar imágenes solo tras cierto umbral de visibilidad), podemos crear una directiva que use la API **Intersection Observer**:

```ts
import { Directive, ElementRef, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[lazyLoad]'
})
export class LazyLoadDirective implements OnInit {
  @Input('lazyLoad') src!: string;

  constructor(private el: ElementRef<HTMLImageElement>) {}

  ngOnInit() {
    const observer = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.el.nativeElement.src = this.src;
          observer.unobserve(this.el.nativeElement);
        }
      });
    });
    observer.observe(this.el.nativeElement);
  }
}
```

Uso en plantilla:

```html
<img lazyLoad="assets/large-image.jpg" width="800" height="600" />
```

👉 La imagen no se carga hasta que entra en el viewport.  
👉 Esto es útil para imágenes muy pesadas o en listas infinitas.  

### 8.5.3. Diferimiento condicional de imágenes

Podemos combinar directivas con **Signals** o servicios para decidir dinámicamente si retrasar o no la carga:  
- **Conexión lenta** → retrasar imágenes no críticas.  
- **Modo ahorro de datos** → cargar solo miniaturas.  
- **Dispositivo móvil** → cargar versiones reducidas.  

Ejemplo con Signal:

```ts
const isSlowNetwork = signal(false);

effect(() => {
  if (isSlowNetwork()) {
    // cargar solo imágenes esenciales
  }
});
```

### 8.5.4. Beneficios de retrasar imágenes no visibles

- **Menor tiempo de carga inicial**: la página se muestra más rápido.  
- **Ahorro de datos**: ideal para usuarios móviles.  
- **Mejor experiencia de usuario**: las imágenes aparecen justo cuando se necesitan.  
- **Optimización de Core Web Vitals**: mejora LCP y CLS al evitar bloqueos innecesarios.  

### 8.5.5. Buenas prácticas

- Usar `priority` solo en la imagen principal (LCP).  
- Combinar `NgOptimizedImage` con directivas personalizadas para casos especiales.  
- Siempre definir `width` y `height` para evitar saltos de layout.  
- En galerías grandes, cargar primero miniaturas y diferir las versiones en alta resolución.  
- Monitorizar con herramientas como **Lighthouse** o **Angular DevTools** para validar mejoras.  


## 8.6. Mejores prácticas de imágenes responsive con Angular

El diseño responsive no se limita a rejillas y media queries: las **imágenes adaptativas** son un pilar fundamental. Una imagen mal optimizada puede arruinar la experiencia en móviles (descargas pesadas) o en pantallas grandes (baja calidad). Angular 20, con `NgOptimizedImage`, ofrece un enfoque moderno y declarativo para resolver este reto.  

### 8.6.1. Definir siempre `width` y `height`

- Angular exige declarar dimensiones para evitar *layout shifts* (CLS).  
- Para imágenes fijas: usar el tamaño renderizado deseado.  
- Para imágenes adaptables: usar el tamaño intrínseco del archivo.  

Ejemplo:

```html
<img ngSrc="assets/product.jpg" width="800" height="600" />
```

👉 Esto asegura que el navegador reserve espacio antes de cargar la imagen.  

### 8.6.2. Usar `sizes` para imágenes adaptativas

El atributo `sizes` permite indicar cómo debe comportarse la imagen en distintos anchos de pantalla. Angular genera automáticamente el `srcset` correspondiente.  

Ejemplo:

```html
<img
  ngSrc="assets/product.jpg"
  width="1200"
  height="800"
  sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

- En móviles: ocupa el 100% del ancho.  
- En tablets: ocupa el 50%.  
- En pantallas grandes: ocupa un tercio.  

### 8.6.3. Marcar la imagen LCP como `priority`

La imagen más importante de la página (ej. hero banner) debe cargarse primero.  

```html
<img ngSrc="assets/hero.jpg" width="1600" height="900" priority />
```

👉 Angular genera automáticamente un `<link rel="preload">` en SSR y fuerza su carga inmediata.  

### 8.6.4. Usar `fill` para imágenes de fondo

Cuando la imagen debe ocupar todo el contenedor:  

```html
<div class="hero">
  <img ngSrc="assets/hero-bg.jpg" fill priority />
</div>
```

- Se combina con CSS (`object-fit: cover;`).  
- Ideal para cabeceras, banners y secciones de impacto visual.  

### 8.6.5. Placeholders y carga progresiva

Para mejorar la percepción de velocidad:  

```html
<img ngSrc="assets/gallery.jpg" width="600" height="400" placeholder />
```

👉 Angular muestra un **placeholder difuminado** mientras carga la imagen final, siempre que se use un loader compatible (ej. Cloudinary, ImageKit).  

### 8.6.6. Breakpoints globales con `IMAGE_CONFIG`

En proyectos grandes, conviene definir breakpoints globales para generar `srcset` de forma consistente:  

```ts
import { IMAGE_CONFIG } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: IMAGE_CONFIG,
      useValue: {
        breakpoints: [320, 640, 1024, 1440]
      }
    }
  ]
});
```

👉 Esto asegura que todas las imágenes usen los mismos puntos de corte responsive.  

### 8.6.7. Buenas prácticas adicionales

- **Optimizar imágenes en origen**: usar formatos modernos (WebP, AVIF) cuando sea posible.  
- **Combinar con lazy loading**: cargar solo las imágenes visibles en pantalla.  
- **Testear en distintos dispositivos**: usar Chrome DevTools para simular breakpoints.  
- **No abusar de `priority`**: solo debe aplicarse a la imagen LCP.  
- **Usar loaders externos**: delegar en un CDN la generación de variantes optimizadas.  


## 8.7. Compresión y uso de formatos modernos (WebP, AVIF)

Las imágenes representan, en promedio, **casi la mitad del peso total de una página web**. Por ello, elegir el formato adecuado y aplicar compresión eficiente es fundamental para mejorar la velocidad de carga, reducir el consumo de datos y optimizar métricas de experiencia de usuario como **LCP (Largest Contentful Paint)** y **CLS (Cumulative Layout Shift)**.  

Los formatos tradicionales como **JPEG** y **PNG** siguen siendo ampliamente usados, pero presentan limitaciones:  
- JPEG: buena para fotografías, pero con compresión con pérdida y sin soporte de transparencia.  
- PNG: excelente para gráficos con transparencia, pero con archivos pesados.  
- GIF: útil para animaciones simples, pero limitado en colores y muy poco eficiente.  

Los formatos modernos **WebP** y **AVIF** surgen como alternativas superiores, combinando lo mejor de los anteriores con técnicas avanzadas de compresión.  

### 8.7.1. WebP: el estándar moderno de Google

- **Compresión eficiente**: hasta un **30% más pequeño que JPEG** y un **45% más pequeño que PNG** con calidad similar.  
- **Soporte de transparencia**: como PNG, pero con menor peso.  
- **Soporte de animaciones**: como GIF, pero más eficiente.  
- **Compatibilidad**: soportado por la mayoría de navegadores modernos.  
- **Versatilidad**: admite compresión con pérdida y sin pérdida.  

Ejemplo de uso con fallback:  

```html
<picture>
  <source srcset="imagen.webp" type="image/webp" />
  <img src="imagen.jpg" alt="Ejemplo de imagen optimizada" />
</picture>
```

👉 Si el navegador soporta WebP, cargará esa versión; de lo contrario, usará JPEG.  

### 8.7.2. AVIF: el futuro de la compresión de imágenes

- **Basado en el códec AV1** (usado en vídeo de alta eficiencia).  
- **Compresión superior**: entre un **20% y 50% más eficiente que WebP**.  
- **Mejor preservación de detalles** en altas tasas de compresión.  
- **Soporte para HDR y espacios de color amplios**.  
- **Características avanzadas**: transparencia, animaciones, profundidad de bits variable.  
- **Limitaciones actuales**:  
  - Codificación más lenta (requiere más recursos).  
  - Ecosistema menos maduro que WebP.  
  - Soporte en navegadores en crecimiento, pero aún no universal.  

Ejemplo de uso con fallback múltiple:  

```html
<picture>
  <source srcset="imagen.avif" type="image/avif" />
  <source srcset="imagen.webp" type="image/webp" />
  <img src="imagen.jpg" alt="Ejemplo de imagen optimizada con fallback" />
</picture>
```

👉 Aquí, el navegador intentará cargar AVIF; si no lo soporta, usará WebP; y como último recurso, JPEG.  

### 8.7.3. Estrategias de adopción en Angular 20

- **Nivel básico**: seguir usando JPEG/PNG optimizados para compatibilidad universal.  
- **Nivel intermedio**: usar WebP con fallback a JPEG/PNG.  
- **Nivel avanzado**: usar AVIF como primera opción, con fallback a WebP y luego a JPEG/PNG.  

En combinación con **`NgOptimizedImage`**, Angular puede gestionar automáticamente `srcset`, `sizes` y placeholders, facilitando la adopción de estos formatos modernos.  

### 8.7.4. Beneficios para Core Web Vitals

- **LCP más rápido**: imágenes críticas cargan antes gracias a menor peso.  
- **CLS reducido**: al definir `width` y `height`, se evita el reflujo de diseño.  
- **Mejor SEO**: Google premia sitios rápidos y penaliza los lentos.  
- **Menor consumo de datos**: ideal para usuarios móviles y conexiones limitadas.  

## 8.8. Estrategias de imágenes en SSR y PWAs

Las imágenes son, en la mayoría de los casos, el recurso más pesado de una aplicación web. En un mundo donde los usuarios esperan experiencias inmediatas, fluidas y disponibles en cualquier contexto —ya sea en un ordenador de escritorio con fibra óptica o en un móvil con conexión intermitente—, optimizar cómo se cargan y gestionan las imágenes se convierte en un factor decisivo.  

En Angular 20, el reto no es solo **mostrar imágenes**, sino hacerlo de manera **inteligente y adaptativa**. Aquí entran en juego dos escenarios clave: **SSR (Server-Side Rendering)** y **PWAs (Progressive Web Apps)**. Ambos persiguen un mismo objetivo: mejorar la percepción de velocidad y la experiencia de usuario, pero lo hacen desde ángulos distintos.  

### 8.8.1. Imágenes en SSR: la primera impresión cuenta

Cuando una aplicación se renderiza en el servidor, el usuario recibe un HTML ya procesado, con el contenido listo para mostrarse. Esto cambia radicalmente la forma en que se gestionan las imágenes:  

- **Preload de imágenes críticas**:  
  La imagen más importante de la página (generalmente el *hero banner* o la imagen LCP) debe estar disponible lo antes posible. Con `NgOptimizedImage`, basta con añadir `priority` para que Angular genere automáticamente un `<link rel="preload">` en el `<head>`. Esto significa que, incluso antes de que el navegador empiece a renderizar, ya sabe que esa imagen es esencial.  

- **SEO y accesibilidad**:  
  Al estar las imágenes incluidas en el HTML inicial, los motores de búsqueda pueden indexarlas sin depender de JavaScript. Esto mejora la visibilidad en buscadores y garantiza que las imágenes críticas estén disponibles para tecnologías de asistencia.  

- **Placeholders renderizados en servidor**:  
  Si usamos `placeholder`, el difuminado inicial se entrega ya en el HTML. El usuario nunca ve un espacio vacío: desde el primer instante percibe que la imagen está “ahí”, aunque todavía no se haya cargado en alta resolución.  

- **Optimización en rutas dinámicas**:  
  En aplicaciones con SSR, cada ruta puede tener imágenes distintas (ej. `/product/:id`). Aquí conviene precargar solo la imagen principal de cada vista y diferir el resto. Así, el usuario percibe inmediatez sin sobrecargar la red.  

**Ejemplo narrativo**: imagina un e‑commerce de moda. Cuando un usuario entra en la página de un producto, el servidor ya le entrega la foto principal del vestido en alta calidad, lista para ser renderizada. El resto de imágenes de la galería se cargan después, cuando el usuario empieza a interactuar. El resultado: una primera impresión impecable.  

### 8.8.2. Imágenes en PWAs: la experiencia continua

Las **Progressive Web Apps** buscan ofrecer una experiencia similar a una aplicación nativa: rápidas, confiables y disponibles incluso sin conexión. En este contexto, las imágenes juegan un papel crucial:  

- **Cacheo inteligente con Service Workers**:  
  El `ngsw-config.json` permite definir qué imágenes se cachean y cómo. Podemos optar por estrategias como:  
  - *Performance*: servir primero desde cache, ideal para iconos o logotipos.  
  - *Freshness*: verificar en red y actualizar cache, útil para catálogos que cambian con frecuencia.  

- **Soporte offline**:  
  Una PWA bien diseñada no debería mostrar “espacios rotos” cuando no hay conexión. Para ello, se pueden cachear versiones reducidas de las imágenes o mostrar placeholders cuando no se pueda acceder a la red.  

- **Carga diferida en scroll**:  
  En catálogos extensos, las imágenes deben cargarse solo cuando el usuario hace scroll. Esto reduce el consumo de datos y mejora la fluidez.  

- **Uso de formatos modernos**:  
  Al cachear imágenes en el dispositivo, cada kilobyte cuenta. Usar WebP o AVIF reduce el espacio ocupado y acelera la carga en visitas posteriores.  

**Ejemplo narrativo**: imagina una PWA de noticias. El usuario abre la app en el metro, sin conexión. Gracias al Service Worker, las imágenes de los artículos que ya había leído aparecen instantáneamente desde cache. Para los artículos nuevos, la app muestra placeholders hasta que la conexión se restablece. El usuario nunca percibe una “rotura” en la experiencia.  

### 8.8.3. Estrategias combinadas: SSR + PWA

Cuando una aplicación combina **SSR y PWA**, se obtiene lo mejor de ambos mundos:  

- **Velocidad inicial (SSR)**: el usuario recibe la primera vista renderizada con imágenes críticas ya precargadas.  
- **Experiencia continua (PWA)**: en visitas posteriores, las imágenes ya están en cache, listas para mostrarse incluso sin conexión.  
- **Fallbacks inteligentes**: si el usuario pierde la conexión, la app puede mostrar imágenes cacheadas o placeholders, manteniendo la coherencia visual.  

**Ejemplo narrativo**: un marketplace internacional.  
1. Primera visita → SSR entrega la página del producto con la imagen principal precargada.  
2. El Service Worker cachea las imágenes de la galería.  
3. En la segunda visita, aunque el usuario esté en un avión sin WiFi, la PWA muestra las imágenes desde cache. La experiencia es fluida, como si estuviera online.  

### 8.8.4. Buenas prácticas

- En **SSR**:  
  - Usar `priority` solo en la imagen LCP.  
  - Evitar precargar demasiadas imágenes desde el servidor.  
  - Aprovechar placeholders para mejorar la percepción de carga.  

- En **PWAs**:  
  - Cachear solo lo necesario (iconos, logotipos, imágenes críticas).  
  - Usar estrategias condicionales según red y dispositivo.  
  - Proveer imágenes fallback para modo offline.  

- En **SSR + PWA**:  
  - Combinar preload inicial con cache progresivo.  
  - Usar formatos modernos (WebP/AVIF) para reducir peso en cache.  
  - Monitorizar métricas de LCP y CLS con Lighthouse o Angular DevTools.  


## 8.9. Integración de `NgOptimizedImage` en proyectos enterprise grandes

En aplicaciones pequeñas, adoptar `NgOptimizedImage` puede ser tan sencillo como reemplazar `src` por `ngSrc` en unas pocas imágenes. Sin embargo, en proyectos **enterprise**, con decenas de equipos, miles de componentes y catálogos de imágenes dinámicos, la integración requiere una **estrategia global** que garantice consistencia, escalabilidad y control de rendimiento.  

### 8.9.1. Desafíos en proyectos enterprise

- **Volumen de imágenes**: catálogos de productos, galerías multimedia, dashboards con iconografía.  
- **Diversidad de fuentes**: imágenes locales, CDNs, servicios externos, APIs dinámicas.  
- **Consistencia visual**: mantener proporciones, evitar *layout shifts* y asegurar calidad en distintos dispositivos.  
- **Métricas de rendimiento**: cumplir con Core Web Vitals (LCP, CLS, FID) en todas las vistas.  
- **Colaboración entre equipos**: distintos equipos pueden tener criterios diferentes para optimizar imágenes.  

👉 Aquí es donde `NgOptimizedImage` se convierte en un **estándar corporativo** que unifica prácticas.  

### 8.9.2. Estrategia de adopción progresiva

En proyectos grandes, no es realista migrar todas las imágenes de golpe. Lo recomendable es una **adopción progresiva**:

1. **Identificar imágenes críticas**:  
   - Hero banners.  
   - Imágenes de producto en páginas de detalle.  
   - Elementos que impactan en LCP.  

2. **Migración de componentes compartidos**:  
   - Botones con iconos.  
   - Avatares de usuario.  
   - Cabeceras y footers.  

3. **Extensión a módulos funcionales**:  
   - Catálogo → `NgOptimizedImage` con `sizes` y `placeholder`.  
   - Dashboard → iconografía optimizada.  
   - Secciones de marketing → `priority` en imágenes clave.  

### 8.9.3. Configuración global con `IMAGE_CONFIG`

En proyectos enterprise, es fundamental **centralizar la configuración** para evitar inconsistencias.  

Ejemplo:

```ts
import { IMAGE_CONFIG } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: IMAGE_CONFIG,
      useValue: {
        breakpoints: [320, 640, 1024, 1440, 1920],
        placeholderSize: 40,
        disableImageSizeWarning: false
      }
    }
  ]
});
```

- **`breakpoints`**: asegura que todas las imágenes responsivas usen los mismos puntos de corte.  
- **`placeholderSize`**: define un estándar para placeholders en toda la app.  
- **`disableImageSizeWarning`**: se mantiene en `false` para que los equipos reciban advertencias en desarrollo.  

👉 Esto convierte a `NgOptimizedImage` en una **política corporativa de optimización**.  

### 8.9.4. Integración con CDNs y loaders externos

En entornos enterprise, lo habitual es usar un **CDN de imágenes** (Cloudinary, Akamai, ImageKit). Angular permite integrar loaders externos para que `NgOptimizedImage` genere automáticamente URLs optimizadas.  

Ejemplo con ImageKit:

```ts
import { provideImageKitLoader } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    provideImageKitLoader('https://ik.imagekit.io/empresa/')
  ]
});
```

👉 Esto asegura que todas las imágenes se sirvan optimizadas desde el CDN, sin que cada equipo tenga que preocuparse por generar variantes manualmente.  

### 8.9.5. SSR y PWA en proyectos enterprise

- **SSR (Server-Side Rendering)**:  
  - Preload automático de imágenes críticas.  
  - Mejora de SEO en catálogos masivos.  
  - Placeholders renderizados en servidor.  

- **PWA (Progressive Web App)**:  
  - Cacheo inteligente de imágenes en Service Workers.  
  - Experiencia offline en catálogos y dashboards.  
  - Reducción de peso en cache usando WebP/AVIF.  

👉 La combinación SSR + PWA + `NgOptimizedImage` garantiza **velocidad inicial + experiencia continua**.  

### 8.9.6. Gobernanza y buenas prácticas en enterprise

- **Definir guías de estilo**: documentar cómo y cuándo usar `priority`, `fill`, `sizes` y `placeholder`.  
- **Linting y auditorías**: crear reglas de lint que obliguen a usar `ngSrc` en lugar de `src`.  
- **Monitorización continua**: integrar métricas de LCP y CLS en pipelines de CI/CD.  
- **Capacitación de equipos**: formar a los desarrolladores en el uso correcto de la directiva.  
- **Migración progresiva**: priorizar imágenes críticas y luego extender al resto.  

## 8.10. Ejemplos de migración desde librerías de terceros hacia la solución nativa

Durante años, muchos proyectos Angular han dependido de librerías externas o soluciones caseras para optimizar imágenes: directivas personalizadas, wrappers sobre `<picture>`, integraciones con CDNs como Cloudinary o ImageKit, o incluso librerías de terceros como `ngx-lazy-load-image`. Con Angular 20, la directiva **`NgOptimizedImage`** ofrece una alternativa **nativa, integrada y mantenida por el propio framework**, lo que simplifica la arquitectura y reduce dependencias externas.  

### 8.10.1. Migración desde `ngx-lazy-load-image`

**Antes (con librería externa):**

```html
<img [defaultImage]="'assets/placeholder.jpg'"
     [lazyLoad]="'assets/product.jpg'"
     [errorImage]="'assets/error.jpg'" />
```

**Después (con `NgOptimizedImage`):**

```html
<img ngSrc="assets/product.jpg"
     width="800"
     height="600"
     placeholder />
```

- `placeholder` sustituye al `defaultImage`.  
- El lazy loading ya está integrado por defecto.  
- Se elimina la dependencia de una librería externa.  

### 8.10.2. Migración desde `<picture>` manual con WebP

**Antes (configuración manual):**

```html
<picture>
  <source srcset="assets/product.webp" type="image/webp" />
  <img src="assets/product.jpg" width="800" height="600" />
</picture>
```

**Después (con `NgOptimizedImage`):**

```html
<img ngSrc="assets/product.jpg"
     width="800"
     height="600"
     sizes="(max-width: 600px) 100vw, 50vw" />
```

- Angular genera automáticamente `srcset` y selecciona el mejor formato si usamos un loader con soporte WebP/AVIF.  
- Se simplifica el marcado y se centraliza la lógica.  

### 8.10.3. Migración desde loaders de CDNs personalizados

**Antes (con integración manual de Cloudinary):**

```html
<img src="https://res.cloudinary.com/demo/image/upload/w_800,h_600,c_fill/product.jpg" />
```

**Después (con `NgOptimizedImage` + loader oficial):**

```ts
import { provideCloudinaryLoader } from '@angular/common';

bootstrapApplication(AppComponent, {
  providers: [
    provideCloudinaryLoader('https://res.cloudinary.com/demo/')
  ]
});
```

```html
<img ngSrc="product.jpg" width="800" height="600" />
```

- Angular construye automáticamente la URL optimizada.  
- Se mantiene la integración con el CDN, pero con sintaxis declarativa y soporte oficial.  

### 8.10.4. Migración desde directivas personalizadas de lazy loading

Muchos equipos crearon sus propias directivas con **Intersection Observer** para retrasar la carga de imágenes.  

**Antes (directiva custom):**

```html
<img appLazyLoad="assets/gallery.jpg" width="600" height="400" />
```

**Después (con `NgOptimizedImage`):**

```html
<img ngSrc="assets/gallery.jpg" width="600" height="400" />
```

- Lazy loading ya está integrado.  
- Se elimina código duplicado y mantenimiento de directivas propias.  

### 8.10.5. Migración desde `background-image` en CSS

**Antes (con CSS):**

```html
<div class="hero"></div>
```

```css
.hero {
  background-image: url('assets/hero.jpg');
  background-size: cover;
}
```

**Después (con `NgOptimizedImage` y `fill`):**

```html
<div class="hero">
  <img ngSrc="assets/hero.jpg" fill priority />
</div>
```

- `fill` permite que la imagen ocupe todo el contenedor.  
- Angular puede optimizarla, precargarla y aplicar placeholders.  

### 8.10.6. Beneficios de la migración

- **Menos dependencias externas** → menos riesgo de incompatibilidades y vulnerabilidades.  
- **Soporte oficial** → mantenido por el equipo de Angular, alineado con el roadmap del framework.  
- **Mejor DX (Developer Experience)** → sintaxis declarativa, advertencias en desarrollo, integración con SSR y PWAs.  
- **Mejor UX (User Experience)** → optimización automática de LCP, CLS y lazy loading.  

