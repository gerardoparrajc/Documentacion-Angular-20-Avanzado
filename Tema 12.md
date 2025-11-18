# 12. PWAs y APIs web avanzadas

## 12.1 Introducción y fundamentos de las Progressive Web Apps (PWAs) en Angular

Las **Progressive Web Apps (PWAs)** son una de las grandes apuestas de la web moderna: aplicaciones que combinan la **universalidad de la web** con la **experiencia de las apps nativas**.  
En lugar de obligar a los usuarios a descargar desde una tienda, una PWA se instala directamente desde el navegador, funciona offline, envía notificaciones push y se comporta como una aplicación instalada en el dispositivo.

Angular, gracias a su ecosistema y al soporte oficial de `@angular/service-worker`, ofrece un camino claro para transformar una SPA en una PWA robusta y lista para producción.

### 12.1.1 ¿Qué define a una PWA?

Una aplicación web se considera PWA cuando cumple con estas características:

- **Instalable**: el navegador ofrece añadirla a la pantalla de inicio o escritorio.  
- **Offline-first**: funciona sin conexión gracias a un Service Worker que cachea recursos.  
- **Segura**: se sirve siempre bajo HTTPS.  
- **Responsive**: se adapta a cualquier dispositivo.  
- **Enganchable**: puede enviar notificaciones push y actualizarse en segundo plano.  
- **Rápida**: carga instantánea tras la primera visita, gracias al caching inteligente.

### 12.1.2. El papel del Service Worker

El **Service Worker** es el motor que hace posible la experiencia PWA.  
Se trata de un script que corre en segundo plano y que puede:

- Interceptar peticiones de red y servir recursos desde cache.  
- Gestionar actualizaciones de la aplicación.  
- Recibir notificaciones push.  
- Sincronizar datos en segundo plano.  

En Angular, este comportamiento se configura de forma declarativa en el archivo `ngsw-config.json`.

### 12.1.3. Activar PWA en Angular

Angular CLI simplifica el proceso con un solo comando:

```bash
ng add @angular/pwa
```

Este comando añade:

- El archivo `ngsw-config.json` con la estrategia de caching.  
- El `manifest.webmanifest` con los metadatos de instalación.  
- El registro del Service Worker en `main.ts`.  
- Iconos y assets para instalación en dispositivos.  

Al compilar en producción (`ng build --configuration production`), Angular genera el Service Worker y lo integra en el bundle.

---

### Ejemplo de configuración básica (`ngsw-config.json`)

> Nota: Evita cachear datos sensibles (tokens, información privada) en el Service Worker. Usa la caché solo para recursos estáticos y públicos.

```json
{
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "resources": {
        "files": ["/assets/**"]
      }
    }
  ]
}
```

- `installMode: prefetch` → cachea en la instalación inicial.  
- `installMode: lazy` → cachea bajo demanda.  

### 12.1.4. Manifest y experiencia de instalación

El archivo `manifest.webmanifest` define cómo se verá la app al instalarse:

```json
{
  "name": "Mi Angular PWA",
  "short_name": "AngularPWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "icons": [
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

Esto permite que la aplicación se abra en modo standalone, sin barra de navegador, como si fuera una app nativa.

### 12.1.5. Ventajas de usar PWAs en Angular

- **Experiencia de usuario mejorada**: rápida, confiable, offline.  
- **Mayor alcance**: accesible desde cualquier navegador moderno.  
- **Menor coste de mantenimiento**: una sola base de código para web y “app”.  
- **Ideal para entornos Enterprise**: apps internas, CRM, logística, administración.  
- **Compatibilidad con SSR y Signals**: Angular 20 permite combinar PWA con renderizado optimizado y estado reactivo.  

### 12.1.6. Limitaciones y consideraciones

- El Service Worker solo se activa en **modo producción**.  
- Las actualizaciones no son inmediatas: el nuevo SW se instala en segundo plano.  
- Algunas APIs (Push, Background Sync) requieren permisos y no están disponibles en todos los navegadores.  
- En iOS, el soporte de PWA es más limitado que en Android.  


## 12.2. Configuración de Service Workers y estrategias de caché para offline-first

El **Service Worker** es el núcleo de cualquier PWA: un script que se ejecuta en segundo plano, intercepta peticiones de red y decide cómo responder a ellas (desde la red, desde la caché o combinando ambas).  
En Angular, el paquete `@angular/service-worker` abstrae gran parte de la complejidad y permite configurar el comportamiento de la aplicación mediante un archivo declarativo: **`ngsw-config.json`**.

### 12.2.1. Activación del Service Worker en Angular

1. Instalar soporte PWA:  
   ```bash
   ng add @angular/pwa
   ```

2. Angular genera automáticamente:  
   - `ngsw-config.json` → reglas de caché.  
   - `manifest.webmanifest` → metadatos de instalación.  
   - Registro del Service Worker en `main.ts`.  

3. El Service Worker solo se activa en **builds de producción**:  
   ```bash
   ng build --configuration production
   ```

### 12.2.2. Anatomía de `ngsw-config.json`

Ejemplo básico:

```json
{
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "resources": {
        "files": ["/assets/**"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api-data",
      "urls": ["/api/**"],
      "cacheConfig": {
        "strategy": "freshness",
        "maxSize": 100,
        "maxAge": "1h",
        "timeout": "10s"
      }
    }
  ]
}
```

- **assetGroups**: recursos estáticos (HTML, CSS, JS, imágenes).  
  - `installMode: prefetch` → se cachean en la instalación inicial.  
  - `installMode: lazy` → se cachean bajo demanda.  
- **dataGroups**: peticiones dinámicas (APIs, JSON).  
  - `strategy: freshness` → primero intenta red, si falla usa caché.  
  - `strategy: performance` → primero caché, luego red.  

### 12.2.3. Estrategias de caché para offline-first

El objetivo de **offline-first** es que la aplicación siga siendo usable incluso sin conexión. Para ello, combinamos distintas estrategias:

- **App Shell + Prefetch**  
  - Cachear el *shell* de la aplicación (HTML, CSS, JS principales) en la instalación inicial.  
  - Garantiza que la app siempre arranca, incluso sin red.  

- **Lazy caching para assets pesados**  
  - Imágenes, fuentes o recursos secundarios se cachean solo cuando se usan.  
  - Evita descargas innecesarias en la primera carga.  

- **Data caching con Freshness**  
  - Para APIs críticas: se intenta obtener datos frescos, pero si no hay red se sirve la última versión cacheada.  
  - Ejemplo: catálogo de productos, noticias, listados.  

- **Data caching con Performance**  
  - Para datos que cambian poco: se sirve primero la caché y luego se actualiza en segundo plano.  
  - Ejemplo: configuración de usuario, tablas de referencia.  

- **Fallbacks offline**  
  - Definir páginas de fallback (`offline.html`) o imágenes por defecto cuando no hay conexión.  

### 12.2.4. Ejemplo de flujo offline-first

1. El usuario abre la app por primera vez → el Service Worker cachea el *shell*.  
2. En la segunda visita, incluso sin conexión, la app arranca desde la caché.  
3. Las llamadas a la API usan estrategia **freshness**: si hay red, se actualizan; si no, se sirven los datos cacheados.  
4. El usuario puede navegar, ver contenido y realizar acciones básicas sin conexión.  
5. Cuando la red vuelve, el Service Worker sincroniza datos y actualiza la caché.  

### 12.2.5. Buenas prácticas

- **Limitar el tamaño de la caché** (`maxSize`, `maxAge`) para evitar saturar el almacenamiento.  
- **Versionar assets** (hash en nombres de archivos) para que el SW detecte cambios y actualice.  
- **Notificar al usuario** cuando haya una nueva versión disponible.  
- **Probar en condiciones reales**: simular offline en DevTools y validar la experiencia.  
- **Combinar con Background Sync** para enviar acciones pendientes cuando vuelva la conexión.  


## 12.3. Implementación de Web Workers para tareas pesadas y paralelización

En una aplicación Angular, todo el código de la interfaz (renderizado, eventos, lógica) se ejecuta en un único hilo: el **main thread** del navegador.  
Esto significa que si realizamos operaciones costosas —como cálculos matemáticos complejos, procesamiento de imágenes, parsers de JSON muy grandes o transformaciones de datos masivos—, la interfaz puede **bloquearse**, generando una experiencia lenta y poco fluida.

Los **Web Workers** son la solución: permiten ejecutar código en **hilos separados**, liberando el hilo principal y manteniendo la UI siempre responsiva.  
En Angular, la CLI ofrece soporte nativo para crearlos e integrarlos de forma sencilla.

### 12.3.1. ¿Qué es un Web Worker?

Un **Web Worker** es un script que corre en paralelo al hilo principal del navegador.  
Características clave:

- No tiene acceso directo al DOM (no puede manipular elementos de la interfaz).  
- Se comunica con el hilo principal mediante **mensajes** (`postMessage` y `onmessage`).  
- Es ideal para tareas de **CPU intensiva** que no dependen de la UI.  

### 12.3.2. Crear un Web Worker en Angular

Angular CLI simplifica la creación:

```bash
ng generate web-worker app
```

Esto genera un archivo `app.worker.ts` y configura el build para soportar workers.

### 12.3.3. Ejemplo: cálculo intensivo en un Worker

#### `app.worker.ts`

```ts
/// <reference lib="webworker" />

addEventListener('message', ({ data }) => {
  const result = heavyComputation(data);
  postMessage(result);
});

function heavyComputation(input: number): number {
  let total = 0;
  for (let i = 1; i < 1e8; i++) {
    total += (i * input) % 7;
  }
  return total;
}

```

#### Uso en un componente

```ts
import { ChangeDetectionStrategy, Component, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [],
  template: `
    <button (click)="startTask()">Iniciar cálculo</button>
    @if (result()) {
      <p>Resultado: {{ result() }}</p>
    }
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class App {
  result = signal<number | undefined>(undefined);

  startTask() {
    if (typeof Worker !== 'undefined') {
      const worker = new Worker(new URL('./workers/app.worker', import.meta.url));
      worker.onmessage = ({ data }) => {
        this.result.set(data);
      };
      worker.postMessage(45); // Enviar dato al worker
    } else {
      console.warn('Web Workers no soportados en este navegador');
    }
  }
}



```

En este ejemplo, el cálculo pesado se ejecuta en el worker, y la UI sigue siendo fluida.

### 12.3.4. Estrategias de paralelización

- **Dividir tareas en chunks**: en lugar de un cálculo enorme, dividir en partes y procesarlas en paralelo con varios workers.  
- **Pool de workers**: mantener un conjunto de workers activos para distribuir tareas (similar a un thread pool en backend).  
- **Workers especializados**: un worker para cálculos matemáticos, otro para parsing de datos, etc.  
- **Transferencia de datos eficiente**: usar `Transferable Objects` (ej. `ArrayBuffer`) para evitar copias innecesarias.  

### 12.3.5. Casos de uso en Angular

- Procesamiento de imágenes (filtros, compresión).  
- Cálculos científicos o financieros.  
- Parsing de grandes ficheros JSON/CSV.  
- Validaciones complejas en formularios masivos.  
- Algoritmos de machine learning en el cliente.  

### 12.3.6. Buenas prácticas

- **No abusar**: crear demasiados workers puede saturar la memoria.  
- **Cerrar workers** cuando ya no se necesiten (`worker.terminate()`).  
- **Diseñar comunicación clara**: definir protocolos de mensajes (ej. `{ type, payload }`).  
- **Testear en distintos navegadores**: soporte generalizado, pero con matices en Safari/iOS.  
- **Combinar con RxJS**: envolver la comunicación en Observables para integrarlo con el flujo reactivo de Angular.  


## 12.4. Registro y comunicación con Web Workers en aplicaciones Angular

Los **Web Workers** permiten ejecutar tareas pesadas en segundo plano, pero para que sean útiles necesitamos **registrarlos correctamente** y establecer un **canal de comunicación bidireccional** con la aplicación Angular.  
Este proceso se basa en el intercambio de mensajes: el hilo principal envía datos al worker, el worker los procesa y devuelve resultados. Angular CLI facilita este flujo con soporte nativo.

### 12.4.1. Registro de un Web Worker en Angular

1. **Generar un worker con Angular CLI**  
   ```bash
   ng generate web-worker app
   ```
   Esto crea un archivo `app.worker.ts` y configura el build para soportar workers.

2. **Registro automático**  
   En el componente donde se use, Angular permite instanciar el worker con:  
   ```ts
   const worker = new Worker(new URL('./app.worker', import.meta.url));
   ```

3. **Compatibilidad**  
   Antes de usarlo, conviene verificar si el navegador soporta workers:  
   ```ts
   if (typeof Worker !== 'undefined') {
     // Worker disponible
   }
   ```

### 12.4.2. Comunicación entre Angular y el Worker

La comunicación se realiza mediante **mensajes**:

- **Enviar datos al worker**: `worker.postMessage(data)`  
- **Recibir respuesta**: `worker.onmessage = (event) => { ... }`  

### Ejemplo básico

#### `app.worker.ts`

```ts
/// <reference lib="webworker" />

addEventListener('message', ({ data }) => {
  const result = data * 2; // Procesamiento simple
  postMessage(result);
});
```

#### Uso en un componente Angular

```ts
@Component({
  selector: 'app-root',
  template: `
    <button (click)="calcular()">Calcular</button>
    <p *ngIf="resultado">Resultado: {{ resultado }}</p>
  `
})
export class AppComponent {
  resultado?: number;

  calcular() {
    if (typeof Worker !== 'undefined') {
      const worker = new Worker(new URL('./app.worker', import.meta.url));
      worker.onmessage = ({ data }) => {
        this.resultado = data;
        worker.terminate(); // Cerrar cuando ya no se use
      };
      worker.postMessage(21); // Enviar dato al worker
    }
  }
}
```

### 12.4.3. Patrones de comunicación

En aplicaciones más complejas, conviene estructurar los mensajes:

- **Mensajes tipados**: usar interfaces para definir el contrato de comunicación.  
  ```ts
  interface WorkerMessage {
    type: 'CALCULAR' | 'RESULTADO';
    payload: any;
  }
  ```
- **Protocolos simples**: enviar `{ type, payload }` en lugar de datos sueltos.  
- **RxJS + Observables**: envolver la comunicación en un `Subject` para integrarlo con el flujo reactivo de Angular.  

Ejemplo con protocolo:

```ts
// Enviar
worker.postMessage({ type: 'CALCULAR', payload: 42 });

// Recibir
worker.onmessage = ({ data }) => {
  if (data.type === 'RESULTADO') {
    this.resultado = data.payload;
  }
};
```

### 12.4.4. Buenas prácticas

- **Cerrar workers** con `terminate()` cuando ya no se necesiten.  
- **Evitar transferencias pesadas** de objetos grandes: usar `Transferable Objects` (`ArrayBuffer`) para mejorar rendimiento.  
- **Centralizar la lógica de comunicación** en un servicio Angular, usando RxJS o signals para integración reactiva.  
- **Testear en navegadores reales**: aunque el soporte es amplio, hay diferencias en Safari/iOS.  
- **Evitar acceso directo a APIs del navegador en SSR/PWA**: usa comprobaciones de entorno (`isPlatformBrowser`).


## 12.5. Estrategias de compatibilidad en navegadores modernos y móviles

Cuando hablamos de **PWAs y APIs web avanzadas**, uno de los mayores retos no es tanto la implementación técnica en Angular, sino garantizar que la aplicación funcione de manera consistente en la enorme diversidad de navegadores y dispositivos que existen hoy en día. La web es, por naturaleza, un entorno heterogéneo: no todos los usuarios utilizan la misma versión de Chrome, ni todos los móviles tienen la misma potencia, ni todos los sistemas operativos ofrecen el mismo nivel de soporte para las APIs modernas.  

Por eso, más allá de escribir código correcto, el verdadero desafío es **diseñar con compatibilidad en mente**. Esto significa aceptar que no todos los usuarios tendrán acceso a todas las funcionalidades, y que debemos construir aplicaciones que se adapten al contexto de cada dispositivo y navegador.  

### 12.5.1. La filosofía del *progressive enhancement*

El concepto de *progressive enhancement* (mejora progresiva) es la base de cualquier estrategia de compatibilidad moderna. La idea es sencilla:  
1. Primero se construye una experiencia básica, accesible y funcional, que pueda ejecutarse en cualquier navegador, incluso en aquellos con soporte limitado.  
2. Sobre esa base, se van añadiendo capas de funcionalidades avanzadas que se activan únicamente si el navegador las soporta.  

Por ejemplo, una aplicación Angular puede funcionar perfectamente como una SPA tradicional en cualquier navegador moderno. Si detectamos que el navegador soporta **Service Workers**, entonces activamos el modo offline y el caching avanzado. Si además soporta **Push Notifications**, podemos ofrecer al usuario la posibilidad de recibir alertas en tiempo real. Y si el dispositivo permite **Background Sync**, podemos añadir sincronización automática de datos cuando la red vuelva a estar disponible.  

De esta manera, la aplicación nunca se rompe: simplemente ofrece más o menos funcionalidades según el entorno.  

### 12.5.2. Detección de capacidades frente a detección de navegador

Un error común en el pasado era basar la compatibilidad en la detección del *user agent* (es decir, identificar el navegador y su versión para decidir qué hacer). Este enfoque es frágil, porque los navegadores cambian constantemente y los *user agents* pueden falsificarse.  

La estrategia moderna es la **detección de capacidades** (*feature detection*). En lugar de preguntar “¿Es Chrome 120?”, preguntamos “¿Existe `serviceWorker` en `navigator`?”. Si la respuesta es sí, activamos la funcionalidad; si no, la ignoramos o mostramos una alternativa.  

Ejemplo en Angular:  

```ts
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/ngsw-worker.js');
}
```

De esta forma, la aplicación se adapta dinámicamente al entorno, sin depender de suposiciones sobre el navegador.  

### 12.5.3. Compatibilidad de PWAs en distintos navegadores

Aquí es donde las diferencias se hacen más evidentes.  
- **En Chrome, Edge y Firefox**, el soporte de PWAs es muy completo: Service Workers, Web App Manifest, Push Notifications y Background Sync funcionan de manera estable.  
- **En Safari (iOS)**, el soporte es más limitado. Aunque los Service Workers están disponibles, se reinician con frecuencia para ahorrar batería, lo que puede afectar la persistencia de datos offline. Además, las notificaciones push llegaron tarde a iOS y aún presentan restricciones. El Web App Manifest también se interpreta de forma parcial: algunas propiedades, como `display: standalone`, funcionan, pero otras se ignoran.  

Esto significa que, al diseñar una PWA en Angular, debemos asumir que la experiencia en iOS será más limitada. La clave está en **no romper la aplicación**: si no hay soporte para notificaciones, simplemente no se ofrecen; si el Service Worker se reinicia, la aplicación debe ser capaz de recuperarse sin problemas.  

### 12.5.4. Compatibilidad de Web Workers

Los **Web Workers** tienen un soporte mucho más homogéneo en navegadores modernos, tanto en desktop como en móviles. Sin embargo, hay matices:  
- En iOS, los workers pueden pausarse cuando la aplicación pasa a segundo plano, lo que significa que no podemos confiar en ellos para tareas de larga duración si el usuario cambia de app.  
- En general, los workers no tienen acceso al DOM, lo que obliga a diseñar protocolos de comunicación claros entre el hilo principal y el worker.  

La estrategia aquí es siempre **verificar disponibilidad** (`if (typeof Worker !== 'undefined')`) y diseñar la aplicación para que, si no hay soporte, la tarea se ejecute en el hilo principal, aunque con menor rendimiento.  

### 12.5.5. Estrategias prácticas en Angular

Angular ofrece varias herramientas para gestionar compatibilidad:  
- **Polyfills**: en el archivo `polyfills.ts` podemos incluir librerías que simulan funcionalidades modernas en navegadores antiguos. Aunque hoy en día su uso es menor, siguen siendo útiles para garantizar soporte en entornos corporativos con navegadores desactualizados.  
- **Condicionales en runtime**: registrar Service Workers, Web Workers o APIs avanzadas solo si están disponibles.  
- **Fallbacks de UI**: mostrar mensajes claros cuando una funcionalidad no esté disponible. Por ejemplo: “Las notificaciones no están soportadas en este navegador” en lugar de dejar un botón inservible.  
- **Testing cross-browser**: no basta con probar en Chrome. Es necesario validar en Safari, Firefox y navegadores móviles. Herramientas como BrowserStack o dispositivos físicos ayudan a detectar problemas reales.  

### 12.5.6. Compatibilidad en móviles: retos adicionales

Los dispositivos móviles añaden otra capa de complejidad:  
- **Limitaciones de hardware**: menos CPU y memoria que un desktop → conviene usar Web Workers para tareas pesadas y optimizar el consumo de recursos.  
- **Batería**: los navegadores móviles son agresivos en suspender procesos en segundo plano para ahorrar energía. Esto afecta a Service Workers y Background Sync.  
- **Almacenamiento**: los navegadores móviles limitan el espacio de caché. Por eso es importante configurar `maxSize` y `maxAge` en `ngsw-config.json` para evitar que la aplicación ocupe demasiado espacio.  
- **Experiencia de instalación**: en Android, las PWAs se instalan con un clic; en iOS, el proceso es más manual y menos visible para el usuario.  


## 12.6. Uso de Push Notifications API en Angular (registro y envío de notificaciones)

Las **notificaciones push** son una de las capacidades más potentes que ofrecen las **Progressive Web Apps (PWAs)**. Permiten que una aplicación web se comunique con el usuario incluso cuando no está abierta en el navegador, enviando mensajes relevantes directamente al dispositivo.  
En entornos **Enterprise**, esta funcionalidad es clave para casos como: alertas de seguridad, actualizaciones de estado en tiempo real, recordatorios de tareas o notificaciones de nuevos mensajes.  

Angular, gracias a su integración con **Service Workers** y la **Push API**, ofrece un camino claro para implementar esta característica, aunque requiere comprender bien el flujo de registro, permisos y envío de mensajes.

### 12.6.1. ¿Cómo funcionan las notificaciones push?

El flujo básico de las notificaciones push en una PWA Angular es el siguiente:

1. **El navegador solicita permiso al usuario** para mostrar notificaciones.  
2. **La aplicación se suscribe** a un servicio de mensajería push a través del Service Worker.  
3. **El servidor de la aplicación guarda la suscripción** (un objeto con claves públicas y endpoint).  
4. **El servidor envía un mensaje push** al endpoint del navegador, usando protocolos de Web Push.  
5. **El Service Worker recibe el mensaje** y muestra la notificación en el dispositivo, incluso si la app está cerrada.  

Este flujo combina tres piezas: el cliente Angular, el Service Worker y un servidor que actúa como emisor de notificaciones.

### 12.6.2. Registro de notificaciones en Angular

Para habilitar notificaciones push en Angular, necesitamos:

1. **Activar el Service Worker** con `@angular/pwa`.  
2. **Configurar el módulo de notificaciones push** en Angular.  
3. **Solicitar permiso al usuario** y registrar la suscripción.  

Ejemplo de servicio Angular para gestionar la suscripción:

```ts
import { Injectable } from '@angular/core';
import { SwPush } from '@angular/service-worker';

@Injectable({ providedIn: 'root' })
export class PushService {
  readonly VAPID_PUBLIC_KEY = 'TU_CLAVE_PUBLICA_VAPID';

  constructor(private swPush: SwPush) {}

  subscribeToNotifications() {
    if (!this.swPush.isEnabled) {
      console.warn('Push Notifications no están habilitadas');
      return;
    }

    this.swPush.requestSubscription({
      serverPublicKey: this.VAPID_PUBLIC_KEY
    })
    .then(sub => {
      // Aquí enviaríamos la suscripción al backend
      console.log('Suscripción creada:', sub);
    })
    .catch(err => console.error('Error en la suscripción', err));
  }
}
```

- `SwPush` es el servicio de Angular que abstrae la API de Push.  
- `VAPID_PUBLIC_KEY` es la clave pública generada para autenticar el envío de notificaciones (parte del estándar Web Push).  

### 12.6.3. Recepción de notificaciones en el Service Worker

El Service Worker intercepta los mensajes push y muestra la notificación:

```ts
// ngsw-worker.js o un service worker personalizado
self.addEventListener('push', (event) => {
  const data = event.data?.json() || {};
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.message,
      icon: '/assets/icons/icon-192x192.png'
    })
  );
});
```

Esto asegura que, aunque la aplicación no esté abierta, el usuario reciba la notificación.

### 12.6.4. Envío de notificaciones desde el servidor

El backend es responsable de enviar las notificaciones a los navegadores suscritos.  
Para ello se utiliza el protocolo **Web Push**, que requiere:

- La **clave privada VAPID** (complementaria a la pública usada en Angular).  
- Una librería en el servidor que implemente Web Push (ej. `web-push` en Node.js).  

Ejemplo en Node.js:

```js
const webpush = require('web-push');

const vapidKeys = {
  publicKey: 'TU_CLAVE_PUBLICA_VAPID',
  privateKey: 'TU_CLAVE_PRIVADA_VAPID'
};

webpush.setVapidDetails(
  'mailto:admin@miapp.com',
  vapidKeys.publicKey,
  vapidKeys.privateKey
);

// Suscripción guardada en la base de datos
const subscription = { /* objeto enviado desde Angular */ };

webpush.sendNotification(subscription, JSON.stringify({
  title: 'Nueva alerta',
  message: 'Tienes un nuevo mensaje en tu bandeja'
}));
```

### 12.6.5. Consideraciones de compatibilidad

- **Android + Chrome/Edge/Firefox**: soporte completo de notificaciones push.  
- **iOS (Safari)**: soporte más limitado y reciente; requiere iOS 16.4+ y aún con restricciones.  
- **Desktop**: funciona en la mayoría de navegadores modernos, aunque algunos requieren que la app esté instalada como PWA.  

Por eso, siempre conviene **detectar soporte** y ofrecer alternativas (ej. notificaciones in-app) cuando no esté disponible.

### 12.6.6. Buenas prácticas

- **Solicitar permisos en el momento adecuado**: nunca al cargar la app, sino cuando el usuario realiza una acción que justifique la notificación.  
- **Ser relevante y no intrusivo**: notificaciones excesivas generan rechazo y desinstalación.  
- **Gestionar la suscripción en backend**: guardar y actualizar las suscripciones de cada usuario.  
- **Personalizar la experiencia**: incluir iconos, acciones rápidas y deep links en las notificaciones.  
- **Combinar con analítica**: medir cuántos usuarios aceptan notificaciones y cuántos interactúan con ellas.  
- **Centralizar la lógica de notificaciones en servicios Angular** y validar siempre en backend.


## 12.7. Integración de PWAs con Angular CLI: instalación y pruebas en dispositivos

Uno de los grandes atractivos de Angular es que ofrece soporte **nativo** para convertir una aplicación en **Progressive Web App (PWA)** sin necesidad de configurar manualmente todos los detalles del Service Worker o del manifest.  
La **Angular CLI** simplifica este proceso, permitiendo que en pocos pasos una SPA tradicional se transforme en una aplicación instalable, con soporte offline y lista para probar en dispositivos móviles.

### 12.7.1. De SPA a PWA con Angular CLI

El primer paso para integrar una PWA en un proyecto Angular existente es ejecutar:

```bash
ng add @angular/pwa
```

Este comando realiza varias tareas automáticamente:

- Añade la dependencia `@angular/service-worker`.  
- Configura el `angular.json` para incluir el Service Worker en builds de producción.  
- Genera el archivo `ngsw-config.json`, donde se definen las estrategias de caché.  
- Crea el `manifest.webmanifest`, con los metadatos de instalación (nombre, iconos, colores, etc.).  
- Añade iconos en diferentes resoluciones dentro de la carpeta `assets/icons`.  
- Modifica `app.module.ts` para registrar el Service Worker cuando la aplicación se ejecute en producción.

De esta forma, la aplicación queda preparada para comportarse como una PWA sin necesidad de configuraciones manuales complejas.

### 12.7.2. Instalación en dispositivos: el flujo del usuario

Una vez compilada la aplicación en modo producción (`ng build --configuration production`) y desplegada en un servidor **HTTPS**, los navegadores modernos detectan automáticamente que se trata de una PWA.

- **En Chrome/Edge (desktop y Android)**: aparece un botón o banner de “Instalar aplicación” en la barra de direcciones o en el menú. Al instalarla, la aplicación se abre en modo standalone, sin barra de navegación, como si fuera una app nativa.  
- **En Safari (iOS)**: el proceso es más manual. El usuario debe pulsar el botón de compartir y seleccionar “Añadir a pantalla de inicio”. Aunque el soporte es más limitado, la aplicación se comporta como una app instalada.  

Este flujo de instalación es clave para la experiencia de usuario: la aplicación deja de ser “una pestaña más” y pasa a formar parte del ecosistema del dispositivo.

### 12.7.3. Pruebas en dispositivos reales

Probar una PWA en el navegador de escritorio es útil, pero no suficiente. La verdadera validación ocurre en dispositivos móviles, donde entran en juego factores como la conectividad intermitente, el almacenamiento limitado y las restricciones de batería.

#### Estrategias de prueba

1. **Pruebas en local con HTTPS**  
   - Angular CLI permite servir la aplicación con HTTPS en local:  
     ```bash
     ng serve --configuration production --ssl true
     ```  
   - Esto es necesario porque los Service Workers solo funcionan bajo HTTPS (excepto en `localhost`).  

2. **Uso de DevTools en Chrome**  
   - En la pestaña *Application* se puede inspeccionar el Service Worker, el manifest y la caché.  
   - Permite simular condiciones offline y comprobar que la aplicación sigue funcionando.  

3. **Pruebas en dispositivos físicos**  
   - Instalar la aplicación en un móvil Android y verificar la experiencia de instalación, el arranque offline y la persistencia de datos.  
   - En iOS, comprobar el flujo de “Añadir a pantalla de inicio” y validar las limitaciones (ej. reinicio frecuente del Service Worker).  

4. **Pruebas de actualización**  
   - Desplegar una nueva versión y comprobar cómo el Service Worker gestiona la actualización.  
   - Angular ofrece un mecanismo automático: el nuevo SW se instala en segundo plano y se activa al recargar la aplicación.  

### 12.7.4. Buenas prácticas en la integración

- **Versionar assets**: Angular ya genera nombres con hash para detectar cambios y actualizar la caché.  
- **Notificar al usuario de nuevas versiones**: mostrar un banner cuando el Service Worker detecta una actualización disponible.  
- **Optimizar el manifest**: personalizar iconos, colores y nombre para que la app instalada tenga identidad propia.  
- **Probar en condiciones reales**: simular redes lentas, desconexiones y cambios de versión para garantizar una experiencia robusta.  


## 12.8. Configuración de Web Manifest y mejora de experiencia de usuario

El **Web App Manifest** es un archivo JSON que actúa como la “tarjeta de identidad” de una Progressive Web App. Define cómo se presentará la aplicación cuando el usuario la instale en su dispositivo: su nombre, iconos, colores, orientación de pantalla y comportamiento al abrirse.  
Aunque pueda parecer un detalle menor, el manifest es clave para que la aplicación **se sienta como una app nativa** y no como una simple página web. En entornos Enterprise, donde la experiencia de usuario es un factor de adopción, una configuración cuidada del manifest puede marcar la diferencia entre una aplicación que los usuarios perciben como profesional y otra que se siente improvisada.

### 12.8.1. ¿Qué es el Web Manifest?

El manifest es un archivo llamado normalmente `manifest.webmanifest` que se incluye en el `index.html` mediante una etiqueta `<link rel="manifest" href="manifest.webmanifest">`.  
Este archivo contiene metadatos que los navegadores utilizan para:

- Mostrar el nombre y el icono de la aplicación en la pantalla de inicio o escritorio.  
- Definir los colores de fondo y de tema que se aplican en la barra de estado o en el splash screen.  
- Determinar cómo se abre la aplicación (en pestaña del navegador, en ventana standalone, en pantalla completa).  
- Especificar la orientación preferida (portrait, landscape).  
- Indicar la URL inicial al abrir la aplicación instalada.  

En resumen, el manifest es el puente entre la web y la experiencia de instalación en el dispositivo.

### 12.8.2. Ejemplo de manifest en Angular

Cuando ejecutamos `ng add @angular/pwa`, Angular genera automáticamente un archivo `manifest.webmanifest` con una configuración básica. Un ejemplo típico sería:

```json
{
  "name": "Mi Angular PWA",
  "short_name": "AngularPWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "orientation": "portrait",
  "icons": [
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

Cada propiedad tiene un impacto directo en la experiencia del usuario:

- **name / short_name**: el nombre completo aparece en el splash screen, mientras que el corto se usa en el icono de la pantalla de inicio.  
- **start_url**: define la ruta inicial al abrir la app instalada (por ejemplo, `/dashboard` en lugar de `/`).  
- **display**: `standalone` elimina la barra de navegación del navegador, haciendo que la app se vea como nativa.  
- **background_color** y **theme_color**: se usan en el splash screen y en la barra de estado, reforzando la identidad visual.  
- **icons**: deben incluirse en varias resoluciones para adaptarse a distintos dispositivos y densidades de pantalla.  

### 12.8.3. Mejora de la experiencia de usuario

Configurar el manifest no es solo un requisito técnico, sino una oportunidad para **reforzar la identidad de marca y la usabilidad**:

- **Consistencia visual**: usar los mismos colores y logotipos que en la versión nativa o en la identidad corporativa.  
- **Splash screen atractivo**: elegir un color de fondo y un icono que transmitan profesionalidad mientras la app carga.  
- **Nombre claro y corto**: evitar nombres largos que se corten en la pantalla de inicio.  
- **Orientación adecuada**: en apps de lectura o formularios, `portrait`; en apps de mapas o dashboards, `landscape`.  
- **Start URL estratégica**: dirigir al usuario a la sección más relevante (ej. un panel de control) en lugar de la home genérica.  

### 12.8.4. Consideraciones de compatibilidad

- **Android y navegadores basados en Chromium**: interpretan casi todas las propiedades del manifest y ofrecen instalación fluida.  
- **iOS (Safari)**: no interpreta el manifest completo, pero respeta algunas propiedades como iconos y colores. Requiere configuraciones adicionales en `apple-touch-icon` y `meta tags` para personalizar la experiencia.  
- **Desktop**: navegadores como Chrome y Edge permiten instalar la PWA en el escritorio, usando los metadatos del manifest.  

Esto significa que, aunque el manifest es estándar, conviene **probar en distintos dispositivos** para ajustar la experiencia.

### 12.8.5. Buenas prácticas

- Incluir iconos en múltiples tamaños: 72, 96, 128, 192, 256, 384 y 512 px.  
- Usar colores de tema que contrasten bien con el contenido.  
- Mantener el manifest sincronizado con la identidad visual de la organización.  
- Validar el manifest con herramientas como **Lighthouse** para detectar errores o propiedades faltantes.  
- Probar la instalación en Android, iOS y desktop para asegurar consistencia.  
- **Evitar acceso directo a APIs del navegador en SSR/PWA**: usa comprobaciones de entorno (`isPlatformBrowser`).


## 12.9. Implementación de lógica avanzada en notificaciones push (acciones y respuestas)

Las notificaciones push no tienen por qué limitarse a mostrar un título y un texto. En aplicaciones modernas, especialmente en entornos **Enterprise**, se busca que las notificaciones sean **interactivas**: que permitan al usuario **responder directamente** o **ejecutar acciones rápidas** sin necesidad de abrir la aplicación completa.  
Esto convierte a las notificaciones en un canal de interacción bidireccional, no solo en un mecanismo de alerta.

### 12.9.1. Acciones en notificaciones: más que un mensaje

La **Notification API** permite añadir botones de acción a una notificación. Cada acción puede tener un identificador y un título, y opcionalmente un icono.  
Ejemplo: una notificación de un sistema de mensajería puede incluir botones como **“Responder”** o **“Marcar como leído”**.

En Angular, estas acciones se definen en el **Service Worker**, que es quien finalmente muestra la notificación al usuario.

### 12.9.2. Ejemplo de notificación con acciones

En el Service Worker (`ngsw-worker.js` o un worker personalizado):

```js
self.addEventListener('push', (event) => {
  const data = event.data?.json() || {};
  const options = {
    body: data.message,
    icon: '/assets/icons/icon-192x192.png',
    actions: [
      { action: 'reply', title: 'Responder' },
      { action: 'mark-read', title: 'Marcar como leído' }
    ]
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});
```

Aquí hemos definido dos acciones: **reply** y **mark-read**.

### 12.9.3. Manejo de respuestas a acciones

Cuando el usuario pulsa un botón de acción, el Service Worker recibe un evento `notificationclick`.  
Podemos capturar este evento y decidir qué hacer:

```js
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'reply') {
    // Abrir la app en la pantalla de respuesta
    event.waitUntil(
      clients.openWindow('/chat/reply')
    );
  } else if (event.action === 'mark-read') {
    // Llamar a una API para marcar el mensaje como leído
    event.waitUntil(
      fetch('/api/messages/mark-read', { method: 'POST' })
    );
  } else {
    // Si el usuario hace clic en la notificación sin elegir acción
    event.waitUntil(
      clients.openWindow('/')
    );
  }
});
```

De esta forma, la notificación no solo informa, sino que **desencadena lógica de negocio** directamente desde el Service Worker.

### 12.9.4. Notificaciones con entrada de texto (respuestas directas)

Algunos navegadores y sistemas operativos permiten notificaciones con **respuestas directas** (ej. responder a un mensaje desde la propia notificación).  
Aunque el soporte aún es limitado y depende del sistema, la API de notificaciones soporta el campo `type: 'text'` en las acciones:

```js
actions: [
  {
    action: 'reply',
    type: 'text',
    title: 'Responder'
  }
]
```

En este caso, el `notificationclick` recibe también el texto introducido por el usuario, que puede enviarse al servidor.

### 12.9.5. Integración con Angular y backend

Para que estas acciones tengan sentido, deben integrarse con la lógica de la aplicación:

- **En Angular**: al abrir la app desde una notificación, podemos leer parámetros de la URL (`/chat/reply`) y redirigir al usuario a la vista adecuada.  
- **En el backend**: las acciones como “marcar como leído” o “aceptar invitación” deben traducirse en llamadas a APIs REST o GraphQL que actualicen el estado en el servidor.  
- **En el Service Worker**: se pueden realizar llamadas `fetch` directamente, incluso sin abrir la app, lo que permite ejecutar lógica en segundo plano.  

### 12.9.6. Buenas prácticas en notificaciones interactivas

- **Relevancia**: no añadir acciones innecesarias; cada botón debe tener un propósito claro.  
- **Simplicidad**: máximo 2–3 acciones, para no saturar al usuario.  
- **Consistencia**: usar siempre los mismos identificadores de acción y mantener coherencia en la UI.  
- **Seguridad**: validar en el backend cualquier acción recibida desde una notificación (no confiar solo en el cliente).  
- **Experiencia fluida**: si el usuario responde desde la notificación, la app debe reflejar ese cambio inmediatamente al abrirse.  


## 12.10. Buenas prácticas en la construcción de PWAs seguras, accesibles y rápidas

Construir una **Progressive Web App (PWA)** no se trata únicamente de añadir un Service Worker y un manifest. Para que una PWA sea realmente útil y confiable, debe cumplir con tres pilares fundamentales: **seguridad, accesibilidad y rendimiento**.  
Estos aspectos no son opcionales: son la base de la confianza del usuario, de la adopción en entornos corporativos y de la percepción de calidad. Una PWA que falla en cualquiera de estos puntos puede ser técnicamente correcta, pero difícilmente será aceptada por los usuarios.

### 12.10.1. Seguridad: proteger al usuario y a la aplicación

La seguridad en una PWA no es negociable. Al ser aplicaciones que se instalan y permanecen en el dispositivo del usuario, deben transmitir confianza y garantizar que los datos están protegidos.

- **HTTPS obligatorio**: los Service Workers solo funcionan bajo HTTPS (excepto en `localhost`). Esto asegura que la comunicación entre cliente y servidor esté cifrada, evitando ataques de intermediarios.  
- **Gestión de permisos responsable**: las PWAs pueden solicitar permisos sensibles (notificaciones, geolocalización, cámara). Es fundamental pedirlos solo cuando el usuario entienda el valor de concederlos, nunca al inicio de la aplicación.  
- **Validación en backend**: aunque el Service Worker pueda interceptar peticiones, la lógica crítica debe validarse siempre en el servidor. Nunca confiar en que el cliente es seguro.  
- **Actualizaciones seguras**: Angular Service Worker gestiona versiones automáticamente, pero conviene notificar al usuario cuando hay una nueva versión disponible, para evitar inconsistencias.  
- **Protección de datos en caché**: no cachear información sensible (tokens, datos personales) en el Service Worker. El almacenamiento debe usarse para recursos estáticos o datos públicos.  

En entornos Enterprise, estas prácticas son esenciales para cumplir con normativas de seguridad y privacidad.

### 12.10.2. Accesibilidad: una PWA para todos

Una aplicación que no es accesible excluye a una parte de los usuarios. La accesibilidad no es solo un requisito legal en muchos países, sino también una muestra de calidad y empatía en el diseño.

- **Compatibilidad con lectores de pantalla**: usar etiquetas semánticas en HTML y atributos ARIA cuando sea necesario. Angular facilita esto con buenas prácticas en componentes.  
- **Contraste y colores adecuados**: garantizar que los textos sean legibles en cualquier dispositivo y bajo distintas condiciones de luz.  
- **Navegación por teclado**: todos los elementos interactivos deben ser accesibles sin ratón.  
- **Mensajes claros en offline**: si la aplicación no puede cargar datos, mostrar un mensaje accesible y comprensible, no un error técnico.  
- **Pruebas con herramientas automáticas**: Lighthouse y axe-core permiten detectar problemas de accesibilidad de forma rápida.  
- **Lazy loading de imágenes y módulos** para mejorar accesibilidad y rendimiento.

Una PWA accesible no solo cumple con estándares, sino que amplía su alcance y mejora la experiencia de todos los usuarios.

### 12.10.3. Rendimiento: la clave de la percepción de calidad

La rapidez es uno de los factores más determinantes en la experiencia de usuario. Una PWA lenta pierde su propósito: debe ser **rápida desde la primera carga y consistente en el uso offline**.

- **App Shell + caching inteligente**: precargar el esqueleto de la aplicación (HTML, CSS, JS principales) para que la app arranque instantáneamente, incluso sin conexión.  
- **Lazy loading de módulos**: cargar solo lo necesario en cada ruta, reduciendo el tiempo de arranque inicial.  
- **Optimización de imágenes**: usar formatos modernos (WebP, AVIF), tamaños adaptativos y carga diferida (`lazy loading`).  
- **Minificación y tree-shaking**: Angular CLI ya aplica estas optimizaciones en producción, eliminando código no usado.  
- **Medición continua**: usar herramientas como Lighthouse, WebPageTest o el panel de rendimiento de Chrome DevTools para identificar cuellos de botella.  
- **Background Sync**: permitir que las acciones del usuario (ej. enviar un formulario) se guarden y se sincronicen automáticamente cuando vuelva la conexión, evitando frustraciones.  

El rendimiento no es un lujo: es la base de la confianza. Una aplicación que responde rápido transmite profesionalidad y fiabilidad.

