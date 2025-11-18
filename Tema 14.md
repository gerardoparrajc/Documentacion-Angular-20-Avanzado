# 14. Internacionalización (i18n) en Angular 20

## 14.1. Configuración inicial de internacionalización (i18n) en Angular

En un mundo globalizado, las aplicaciones modernas deben ser capaces de adaptarse al idioma y la cultura de sus usuarios. Esto no solo implica traducir textos, sino también ajustar formatos de fecha, moneda, números y reglas gramaticales como pluralización o género.  
En Angular 20, aunque existe soporte nativo para i18n mediante compilación por idioma, en muchos proyectos se prefiere una solución más **dinámica y flexible**: **ngx-translate**.  

Esta librería se ha convertido en un estándar de facto porque permite:  
- Cambiar de idioma en tiempo real.  
- Cargar traducciones desde archivos JSON o APIs externas.  
- Integrarse fácilmente con componentes, servicios y pipes.  
- Mantener una arquitectura más ágil en proyectos multilingües.  

### 14.1.1. Instalación de ngx-translate

Para comenzar, instalamos la librería principal y el loader de archivos:

```bash
npm install @ngx-translate/core @ngx-translate/http-loader
```

### 14.1.2. Configuración en proyectos con módulos (AppModule)

En aplicaciones que aún usan módulos, la configuración se hace en `AppModule`:

```ts
import { HttpClient } from '@angular/common/http';
import { TranslateLoader, TranslateModule } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

export function HttpLoaderFactory(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

@NgModule({
  imports: [
    HttpClientModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient]
      }
    })
  ]
})
export class AppModule {}
```

Esto permite cargar archivos de traducción desde `assets/i18n/`.

### 14.1.3. Configuración en proyectos standalone (Angular 20 sin módulos)

En Angular 20, el enfoque por defecto es **standalone-first**, sin `AppModule`. Aquí usamos `ApplicationConfig` y `provideTranslate`:

**app.config.ts**

```ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, HttpClient } from '@angular/common/http';
import { provideTranslate, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

export function HttpLoaderFactory(http: HttpClient): TranslateLoader {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    provideTranslate({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient]
      }
    })
  ]
};
```

**main.ts**

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

En los componentes standalone, basta con importar `TranslateModule` en el `imports`.

### 14.1.4. Archivos de traducción

En `assets/i18n/` se definen los archivos JSON:

**es.json**
```json
{
  "WELCOME": "Bienvenido a la aplicación",
  "LOGIN": "Iniciar sesión",
  "LOGOUT": "Cerrar sesión"
}
```

**en.json**
```json
{
  "WELCOME": "Welcome to the application",
  "LOGIN": "Log in",
  "LOGOUT": "Log out"
}
```

### 14.1.5. Uso en plantillas

Con el pipe `translate`:

```html
<h1>{{ 'WELCOME' | translate }}</h1>
<button>{{ 'LOGIN' | translate }}</button>
```

### 14.1.6. Uso en componentes

En TypeScript, se inyecta `TranslateService`:

```ts
import { TranslateService } from '@ngx-translate/core';

export class LoginComponent {
  constructor(private translate: TranslateService) {
    translate.addLangs(['en', 'es']);
    translate.setDefaultLang('es');
    translate.use('es');
  }

  switchLang(lang: string) {
    this.translate.use(lang);
  }
}
```

### 14.1.7. Cambio de idioma en tiempo real

El idioma puede cambiarse dinámicamente:

```ts
this.translate.use('en');
```

Incluso detectar el idioma del navegador:

```ts
const browserLang = this.translate.getBrowserLang();
this.translate.use(browserLang.match(/en|es/) ? browserLang : 'en');
```

### 14.1.8. Funcionalidades avanzadas

- **Interpolación de variables**  
  ```json
  { "GREETING": "Hola, {{name}}" }
  ```
  ```ts
  this.translate.instant('GREETING', { name: 'Gerardo' });
  ```

- **Pluralización**  
  ```json
  { "MESSAGES": "Tienes {{count}} mensaje(s)" }
  ```

- **Contexto y organización**  
  Usar claves jerárquicas:  
  ```json
  {
    "LOGIN": {
      "TITLE": "Iniciar sesión",
      "BUTTON": "Entrar"
    }
  }
  ```

### 14.1.9. Buenas prácticas

- Mantener consistencia en las claves (`feature.key`).  
- Evitar textos literales como claves.  
- Validar que todos los idiomas tengan las mismas claves.  
- Usar interpolación en lugar de concatenar strings.  
- Integrar con formularios, validaciones y mensajes de error.  


La internacionalización no es solo traducir palabras: es **adaptar la experiencia al contexto cultural del usuario**.  
Con **ngx-translate**, Angular 20 ofrece una solución dinámica y flexible que permite cambiar de idioma en tiempo real, cargar traducciones desde archivos o APIs y mantener una arquitectura limpia.  
En proyectos modernos, especialmente standalone, la configuración con `provideTranslate` encaja perfectamente con la filosofía de Angular 20: aplicaciones más ligeras, explícitas y modulares.  


## 14.2. Generación de archivos de traducción con Angular CLI

Cuando trabajamos con **ngx-translate**, las traducciones se gestionan en archivos JSON. Sin embargo, en proyectos grandes es fácil que se pierdan claves, se dupliquen o se olviden textos sin traducir.  
Aquí es donde entra **ngx-translate-extract**, una herramienta de línea de comandos que analiza el código de Angular y **extrae automáticamente todas las claves de traducción** usadas en plantillas y componentes, generando o actualizando los archivos JSON de traducción.

De esta forma, el equipo siempre dispone de un inventario actualizado de las claves, lo que facilita la colaboración con traductores y evita inconsistencias.

### 14.2.1. Instalación

Se instala como dependencia de desarrollo:

```bash
npm install @biesbjerg/ngx-translate-extract --save-dev
```

### 14.2.2. Uso básico

El comando principal es:

```bash
ngx-translate-extract -i src -o src/assets/i18n/{en,es}.json
```

- `-i src`: indica la carpeta de entrada (el código fuente).  
- `-o`: define los archivos de salida. En este ejemplo, se generan/actualizan `en.json` y `es.json` en la carpeta `assets/i18n/`.  
- `{en,es}`: permite generar varios idiomas en una sola ejecución.  

El resultado es que todos los textos encontrados en el código se añaden a los archivos de traducción, manteniendo las claves existentes.

### 14.2.3. Ejemplo de flujo de trabajo

1. **Usar claves en plantillas y componentes**  
   ```html
   <h1>{{ 'WELCOME' | translate }}</h1>
   <button>{{ 'LOGIN' | translate }}</button>
   ```

2. **Ejecutar el extractor**  
   ```bash
   npx ngx-translate-extract -i ./src -o ./src/assets/i18n/{en,es}.json -f json
   ```

3. **Archivos generados**  
   **en.json**
   ```json
   {
     "WELCOME": "",
     "LOGIN": ""
   }
   ```
   **es.json**
   ```json
   {
     "WELCOME": "",
     "LOGIN": ""
   }
   ```

   Los traductores completan los valores en cada idioma.

### 14.2.4. Opciones útiles

- `--clean`: elimina claves no utilizadas en el código.  
- `--sort`: ordena las claves alfabéticamente.  
- `--format`: permite elegir el formato de salida (`json`, `namespaced-json`, `pot`).  
- `--marker`: define un marcador especial para extraer claves usadas en TypeScript (ej. `marker('KEY')`).  

Ejemplo con limpieza y orden:

```bash
npx ngx-translate-extract -i ./src -o ./src/assets/i18n/{en,es}.json --clean --sort
```

### 14.2.4. Integración en package.json

Para simplificar, se puede añadir un script en `package.json`:

```json
"scripts": {
  "i18n:extract": "ngx-translate-extract -i ./src -o ./src/assets/i18n/{en,es}.json --clean --sort --format=json"
}
```

Ahora basta con ejecutar:

```bash
npm run i18n:extract
```

### 14.2.5. Buenas prácticas

- Ejecutar el extractor regularmente (ej. antes de cada release).  
- Usar `--clean` con cuidado: elimina claves no detectadas en el código.  
- Mantener un archivo por idioma (`en.json`, `es.json`, `fr.json`).  
- Versionar los archivos de traducción para evitar pérdidas.  
- Usar claves semánticas (`auth.login.title`) en lugar de textos literales.  

Con **ngx-translate-extract**, la internacionalización en Angular se vuelve mucho más **mantenible y escalable**.  
Ya no dependemos de recordar manualmente qué claves deben añadirse: la herramienta garantiza que todos los textos usados en la aplicación estén reflejados en los archivos de traducción.  
Esto no solo ahorra tiempo, sino que también reduce errores y mejora la colaboración entre desarrolladores y traductores.


## 14.3. Uso de etiquetas y marcadores para traducir en plantillas y componentes

En proyectos con **ngx-translate**, la traducción no se limita a cargar archivos JSON: también debemos **marcar y referenciar las claves de traducción** en el código de manera clara y consistente.  
Esto se logra mediante dos mecanismos principales:

1. **Etiquetas en plantillas**: pipes y directivas que aplican traducciones directamente en el HTML.  
2. **Marcadores en componentes**: funciones o utilidades que aseguran que las claves usadas en TypeScript también sean detectadas por herramientas como **ngx-translate-extract**.

### 14.3.1. Uso de etiquetas en plantillas

La forma más común de traducir en Angular es mediante el **pipe `translate`**:

```html
<h1>{{ 'WELCOME' | translate }}</h1>
<button>{{ 'LOGIN' | translate }}</button>
```

- `WELCOME` y `LOGIN` son claves definidas en los archivos JSON.  
- El pipe se encarga de mostrar el valor traducido según el idioma activo.  

#### Interpolación de variables

Podemos pasar parámetros dinámicos:

**es.json**
```json
{ "GREETING": "Hola, {{name}}" }
```

**HTML**
```html
<p>{{ 'GREETING' | translate:{ name: userName } }}</p>
```

Resultado: `Hola, Gerardo`.

#### Directiva `translate`

Otra opción es usar la directiva `translate`:

```html
<p translate>WELCOME</p>
```

Esto es útil cuando queremos traducir el contenido de un elemento sin usar el pipe.

### 14.3.2. Uso de marcadores en componentes

Cuando usamos claves de traducción en **TypeScript**, por ejemplo con `translate.instant()` o `translate.get()`, esas claves no siempre son detectadas automáticamente por los extractores.  
Para resolverlo, se utilizan **marcadores**.

#### Ejemplo con `marker`

```ts
import { marker } from '@biesbjerg/ngx-translate-extract-marker';

const loginKey = marker('LOGIN');
```

- La función `marker()` no traduce nada en tiempo de ejecución.  
- Su único propósito es **marcar la clave** para que `ngx-translate-extract` la detecte al analizar el código.  

Luego podemos usar esa clave con `TranslateService`:

```ts
this.translate.instant(loginKey);
```

#### Ejemplo práctico

```ts
import { TranslateService } from '@ngx-translate/core';
import { marker } from '@biesbjerg/ngx-translate-extract-marker';

const errorKey = marker('ERROR.SERVER');

@Component({...})
export class LoginComponent {
  constructor(private translate: TranslateService) {}

  showError() {
    alert(this.translate.instant(errorKey));
  }
}
```

De esta forma, la clave `ERROR.SERVER` se incluye en los archivos de traducción aunque solo aparezca en TypeScript.

### 14.3.3. Combinando etiquetas y marcadores

- **Plantillas**: usar siempre el pipe `translate` o la directiva `translate`.  
- **Componentes**: usar `marker()` para asegurar que las claves se extraigan, y luego `translate.instant()` o `translate.get()` para obtener el valor traducido.  

Esto garantiza que **todas las claves** estén presentes en los archivos JSON, evitando errores de traducción en producción.

### 14.3.4. Buenas prácticas

- Usar **claves semánticas** (`auth.login.title`) en lugar de textos literales.  
- Marcar siempre las claves en TypeScript con `marker()`.  
- Evitar concatenar strings para formar claves dinámicas (difícil de extraer).  
- Centralizar las claves en constantes si se usan en múltiples lugares.  
- Revisar periódicamente con `ngx-translate-extract` para detectar claves faltantes o huérfanas.  

El uso de **etiquetas y marcadores** en ngx-translate no es un detalle menor: es la base para una internacionalización **robusta, mantenible y escalable**.  
Las etiquetas en plantillas permiten traducir de forma declarativa y clara, mientras que los marcadores en componentes aseguran que ninguna clave quede fuera del radar de los extractores.  
En conjunto, estos mecanismos convierten la traducción en un proceso **sistemático y confiable**, incluso en aplicaciones grandes y multilingües.


## 14.4. Cambio dinámico de idioma en una aplicación Angular

Una de las grandes ventajas de usar **ngx-translate** frente al sistema nativo de i18n de Angular es la posibilidad de **cambiar de idioma en tiempo real**, sin necesidad de recompilar ni recargar la aplicación.  
Esto permite que el usuario seleccione su idioma preferido desde un menú o que la aplicación detecte automáticamente el idioma del navegador y lo aplique al instante.

### 14.4.1. Configuración inicial

En la inicialización de la aplicación, configuramos `TranslateService` para definir los idiomas disponibles, el idioma por defecto y el idioma activo:

```ts
constructor(private translate: TranslateService) {
  translate.addLangs(['en', 'es', 'fr']);
  translate.setDefaultLang('es');

  const browserLang = translate.getBrowserLang();
  translate.use(browserLang?.match(/en|es|fr/) ? browserLang : 'es');
}
```

- `addLangs`: registra los idiomas soportados.  
- `setDefaultLang`: define el idioma por defecto.  
- `use`: activa un idioma específico.  
- `getBrowsefrLang`: detecta el idioma del navegador.  

### 14.4.2. Cambio de idioma desde la interfaz

Podemos crear un **selector de idioma** en la interfaz:

```html
<select #langSelect (change)="switchLang(langSelect.value)">
  <option *ngFor="let lang of languages" [value]="lang">{{ lang }}</option>
</select>
```

En el componente:

```ts
languages = ['en', 'es', 'fr'];

switchLang(lang: string) {
  this.translate.use(lang);
}
```

Cada vez que el usuario selecciona un idioma, la aplicación cambia inmediatamente todos los textos traducidos.

### 14.4.3. Ejemplo en un componente standalone

En Angular 20 (standalone):

```ts
import { Component } from '@angular/core';
import { TranslateService, TranslateModule } from '@ngx-translate/core';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TranslateModule],
  template: `
    <h1>{{ 'WELCOME' | translate }}</h1>
    <button (click)="switchLang('en')">English</button>
    <button (click)="switchLang('es')">Español</button>
    <button (click)="switchLang('fr')">Français</button>
  `
})
export class AppComponent {
  constructor(private translate: TranslateService) {
    translate.addLangs(['en', 'es', 'fr']);
    translate.setDefaultLang('es');
    translate.use('es');
  }

  switchLang(lang: string) {
    this.translate.use(lang);
  }
}
```

### 14.4.4. Persistencia de la preferencia de idioma

Para mejorar la experiencia, podemos guardar la elección del usuario en `localStorage`:

```ts
switchLang(lang: string) {
  this.translate.use(lang);
  localStorage.setItem('lang', lang);
}

ngOnInit() {
  const savedLang = localStorage.getItem('lang');
  this.translate.use(savedLang || 'es');
}
```

De esta forma, al volver a abrir la aplicación, se mantiene el idioma elegido.

### 14.4.5. Cambio de idioma en servicios y lógica de negocio

Además de las plantillas, también podemos traducir mensajes en servicios:

```ts
this.translate.get('ERROR.SERVER').subscribe(msg => {
  this.notificationService.showError(msg);
});
```

Esto asegura que incluso los mensajes de error o notificaciones internas se adapten al idioma activo.

### 14.4.6. Buenas prácticas

- **Centralizar la lógica de idioma** en un servicio (`LanguageService`) para no repetir código.  
- **Persistir la preferencia** del usuario en `localStorage` o en el backend.  
- **Evitar recargar la app**: ngx-translate actualiza dinámicamente los textos.  
- **Ofrecer siempre un idioma por defecto** para evitar pantallas vacías.  
- **Mantener consistencia**: todos los textos deben provenir de archivos de traducción, no hardcodeados.  

El **cambio dinámico de idioma** convierte a una aplicación Angular en una experiencia verdaderamente global.  
Gracias a **ngx-translate**, no solo podemos traducir textos, sino también adaptarnos a las preferencias del usuario en tiempo real, sin interrupciones ni recargas.  
En entornos empresariales o educativos, esta capacidad es clave para garantizar accesibilidad, inclusión y una experiencia de usuario de calidad.


## 14.5. Estrategias para gestionar múltiples idiomas en proyectos enterprise

En aplicaciones pequeñas, traducir unos cuantos textos con **ngx-translate** puede ser suficiente.  
Pero en proyectos **Enterprise**, donde existen decenas de módulos, equipos distribuidos y usuarios en distintos países, la gestión de múltiples idiomas se convierte en un desafío estratégico.  
No se trata solo de traducir, sino de **organizar, versionar, mantener y desplegar** traducciones de forma eficiente y escalable.

### 14.5.1. Organización de archivos de traducción

En proyectos grandes, es recomendable **modularizar las traducciones** en lugar de tener un único archivo JSON gigante.

Ejemplo de estructura:

```
assets/i18n/
  en/
    auth.json
    dashboard.json
    errors.json
  es/
    auth.json
    dashboard.json
    errors.json
```

- Cada dominio o feature tiene su propio archivo.  
- Esto facilita que los equipos trabajen en paralelo y que las traducciones se mantengan organizadas.  
- En tiempo de ejecución, se pueden **fusionar** los archivos para formar el diccionario completo.

### 14.5.2. Uso de claves semánticas y jerárquicas

En lugar de claves planas como `"LOGIN"`, usar claves jerárquicas:

```json
{
  "auth": {
    "login": {
      "title": "Iniciar sesión",
      "button": "Entrar"
    }
  }
}
```

Ventajas:  
- Mayor claridad semántica.  
- Evita colisiones de claves.  
- Facilita la búsqueda y el mantenimiento.  

### 14.5.3. Automatización con ngx-translate-extract

En proyectos Enterprise, es inviable mantener manualmente todas las claves.  
Con **ngx-translate-extract** se pueden:

- Extraer automáticamente las claves usadas en plantillas y componentes.  
- Generar archivos de traducción base para todos los idiomas.  
- Detectar claves huérfanas o no utilizadas.  

Ejemplo de script en `package.json`:

```json
"scripts": {
  "i18n:extract": "ngx-translate-extract -i ./src -o ./src/assets/i18n/{en,es,fr}.json --clean --sort --format=json"
}
```

### 14.5.4. Estrategias de carga de traducciones

En aplicaciones grandes, no conviene cargar todas las traducciones de golpe.  
Opciones:

- **Carga estática global**: todos los idiomas se cargan al inicio (simple, pero pesado).  
- **Carga diferida por módulo**: cada módulo carga sus traducciones al ser usado, usando `TranslateModule.forChild({ extend: true })`.  
- **Carga dinámica desde backend**: las traducciones se almacenan en una API y se descargan bajo demanda.  

Ejemplo de carga modular:

```ts
TranslateModule.forChild({
  extend: true
})
```

Esto permite que cada feature module aporte sus propias traducciones.

### 14.5.5. Integración con procesos de traducción

En entornos Enterprise, los traductores suelen trabajar con herramientas profesionales (Crowdin, Lokalise, Transifex).  
Buenas prácticas:

- Exportar los archivos JSON a la plataforma de traducción.  
- Mantener sincronización automática (CI/CD).  
- Validar que todas las claves estén traducidas antes de desplegar.  

### 14.5.6. Persistencia y preferencia de idioma

En aplicaciones corporativas, el idioma del usuario debe persistir entre sesiones y dispositivos:

- Guardar la preferencia en `localStorage` o `IndexedDB`.  
- Sincronizar con el backend (perfil de usuario).  
- Detectar idioma del navegador solo como fallback inicial.  

### 14.5.7. Estrategias de fallback

Siempre debe existir un idioma de respaldo (generalmente inglés o español corporativo).  
Si una clave no está traducida en el idioma actual, ngx-translate mostrará la del idioma por defecto.  
Esto evita errores en producción y asegura consistencia.

Gestionar múltiples idiomas en proyectos Enterprise no es solo un reto técnico, sino también **organizativo**.  
La clave está en combinar buenas prácticas de arquitectura (modularización, carga diferida), con procesos de automatización (extractores, CI/CD) y con integración en el flujo de trabajo de traductores profesionales.  
De esta forma, la internacionalización deja de ser un obstáculo y se convierte en una **ventaja competitiva**, permitiendo que la aplicación llegue a más usuarios de forma inclusiva y eficiente.


## 14.6. Implementación de selección de idioma mediante formularios y Signals

En aplicaciones multilingües, no basta con ofrecer traducciones: es fundamental que el usuario pueda **elegir su idioma preferido** de forma sencilla y que la aplicación reaccione inmediatamente a ese cambio.  
En Angular 20, podemos combinar **formularios reactivos** con **Signals** para crear un selector de idioma elegante, declarativo y fácil de mantener.

### 14.6.1. Preparar los idiomas disponibles

En un servicio centralizado (ej. `LanguageService`), definimos los idiomas soportados y gestionamos la lógica de cambio:

```ts
import { Injectable, signal } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';

@Injectable({ providedIn: 'root' })
export class LanguageService {
  languages = ['en', 'es', 'fr'];
  currentLang = signal('es');

  constructor(private translate: TranslateService) {
    translate.addLangs(this.languages);
    translate.setDefaultLang('es');
    this.use(this.currentLang());
  }

  use(lang: string) {
    this.translate.use(lang);
    this.currentLang.set(lang);
    localStorage.setItem('lang', lang);
  }

  loadSaved() {
    const saved = localStorage.getItem('lang');
    if (saved && this.languages.includes(saved)) {
      this.use(saved);
    }
  }
}
```

- `currentLang` es un **Signal** que refleja el idioma actual.  
- `use()` cambia el idioma en ngx-translate y actualiza el Signal.  
- Se guarda la preferencia en `localStorage` para persistencia.  

### 14.6.2. Crear un formulario reactivo para el selector

```ts
import { ChangeDetectionStrategy, Component, OnInit, inject } from '@angular/core';
import { ReactiveFormsModule, FormControl } from '@angular/forms';
import { TranslateModule } from '@ngx-translate/core';
import { LanguageService } from './language.service';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-language-selector',
  standalone: true,
  imports: [ReactiveFormsModule, TranslateModule],
  template: `
    <form>
      <label for="lang">{{ 'SELECT_LANGUAGE' | translate }}</label>
      <select [formControl]="langControl" id="lang">
        <option *ngFor="let lang of languageService.languages" [value]="lang">
          {{ lang | uppercase }}
        </option>
      </select>
    </form>

    <p>{{ 'WELCOME' | translate }}</p>
    <p>Idioma actual: {{ languageService.currentLang() | uppercase }}</p>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LanguageSelectorComponent implements OnInit {
  languageService = inject(LanguageService);
  langControl = new FormControl(this.languageService.currentLang());

  ngOnInit() {
    // Sincronizar formulario con Signal
    this.langControl.valueChanges.subscribe(lang => {
      if (lang) this.languageService.use(lang);
    });

    // Inicializar con idioma guardado
    this.languageService.loadSaved();
    this.langControl.setValue(this.languageService.currentLang(), { emitEvent: false });
  }
}
```

- El formulario (`FormControl`) refleja el idioma actual.  
- Al cambiar el valor del select, se actualiza el idioma en tiempo real.  
- El Signal `currentLang` mantiene sincronizado el estado global del idioma.  

### 14.6.3. Integración de Signals en la UI

Podemos usar directamente el Signal en la plantilla para mostrar el idioma activo:

```html
<p>Idioma actual: {{ languageService.currentLang() | uppercase }}</p>
```

Cada vez que el usuario cambie el idioma, el Signal se actualizará y la UI reaccionará automáticamente.

### 14.6.4. Beneficios de esta estrategia

- **Reactividad inmediata**: los Signals actualizan la UI sin necesidad de suscripciones manuales.  
- **Persistencia**: el idioma elegido se guarda y se recupera en cada sesión.  
- **Escalabilidad**: el `LanguageService` centraliza la lógica, facilitando su uso en toda la aplicación.  
- **Integración con formularios**: el selector se implementa con un `FormControl`, lo que permite validaciones, estilos y extensiones.  

La combinación de **formularios reactivos** y **Signals** en Angular 20 ofrece una forma moderna y declarativa de implementar un selector de idioma.  
El usuario puede cambiar de idioma en tiempo real, la aplicación reacciona automáticamente y la preferencia se conserva entre sesiones.  
En proyectos Enterprise, esta estrategia asegura una experiencia multilingüe fluida, escalable y alineada con las mejores prácticas de Angular moderno.


## 14.7. Integración de i18n con SSR para aplicaciones multilingües optimizadas para SEO

En Angular 20, el **Server-Side Rendering (SSR)** ya no es un añadido externo como lo era Angular Universal: ahora forma parte del framework y se activa al crear el proyecto con la opción `--ssr`.  
Esto simplifica la configuración y abre la puerta a aplicaciones **multilingües optimizadas para SEO**, donde cada idioma se renderiza en el servidor y se entrega al usuario (y a los buscadores) ya traducido.

### 14.7.1. Creación de un proyecto con SSR activado

Al generar un nuevo proyecto:

```bash
ng new mi-proyecto --ssr
```

Esto configura automáticamente el renderizado en servidor y cliente, sin necesidad de instalar paquetes adicionales.

### 14.7.2. ¿Por qué combinar i18n con SSR?

- **SEO optimizado**: los buscadores reciben HTML ya traducido.  
- **Mejor rendimiento inicial**: el usuario ve contenido traducido desde el primer render.  
- **Indexación por idioma**: cada idioma tiene su propia URL y contenido.  
- **Accesibilidad global**: usuarios de distintas regiones reciben la versión adecuada de inmediato.  

### 14.7.3. Estrategia de rutas multilingües: incluir el idioma en la URL

> Recomendación Angular 20+: Sincroniza el idioma de la URL con signals en el servicio de idioma para que la UI reaccione automáticamente al cambio de idioma, tanto en SSR como en cliente.

#### a) Definir rutas con prefijo de idioma

La práctica más robusta es incluir el idioma en la URL. Ejemplo:

- Español: `https://miapp.com/es/dashboard`  
- Inglés: `https://miapp.com/en/dashboard`  
- Francés: `https://miapp.com/fr/dashboard`  

En el enrutador:

```ts
import { Routes } from '@angular/router';
import { DashboardComponent } from './dashboard/dashboard.component';

export const routes: Routes = [
  {
    path: ':lang',
    children: [
      { path: 'dashboard', component: DashboardComponent },
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' }
    ]
  },
  { path: '', redirectTo: '/es', pathMatch: 'full' } // idioma por defecto
];
```

#### b) Detectar el idioma desde la URL

Creamos un servicio para sincronizar el idioma de la URL con **ngx-translate**:

```ts
import { Injectable, signal } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';

@Injectable({ providedIn: 'root' })
export class LanguageService {
  currentLang = signal('es');
  supportedLangs = ['es', 'en', 'fr'];

  constructor(private translate: TranslateService) {
    translate.addLangs(this.supportedLangs);
    translate.setDefaultLang('es');
  }

  setLanguage(lang: string) {
    if (!this.supportedLangs.includes(lang)) {
      lang = 'es';
    }
    this.translate.use(lang);
    this.currentLang.set(lang);
  }
}
```

#### c) Sincronizar idioma con el enrutador

En el `AppComponent`:

```ts
import { Component, OnInit, inject } from '@angular/core';
import { Router, NavigationEnd, ActivatedRoute } from '@angular/router';
import { filter } from 'rxjs';
import { LanguageService } from './language.service';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `<router-outlet></router-outlet>`
})
export class AppComponent implements OnInit {
  private router = inject(Router);
  private route = inject(ActivatedRoute);
  private languageService = inject(LanguageService);

  ngOnInit() {
    this.router.events
      .pipe(filter(event => event instanceof NavigationEnd))
      .subscribe(() => {
        const lang = this.route.root.firstChild?.snapshot.params['lang'];
        if (lang) {
          this.languageService.setLanguage(lang);
        }
      });
  }
}
```

#### d) Generar enlaces con idioma incluido

En plantillas:

```html
<a [routerLink]="['/', languageService.currentLang(), 'dashboard']">
  {{ 'DASHBOARD' | translate }}
</a>
```

Si el idioma actual es `en`, el enlace será `/en/dashboard`.

### 14.7.4. Integración con SSR y SEO

Con SSR activado, el servidor renderiza directamente la versión traducida según la URL (`/es`, `/en`, `/fr`).  
Esto permite:

- Que Google indexe cada idioma como una página distinta.  
- Añadir etiquetas `hreflang` en el `<head>`:  

```html
<link rel="alternate" hreflang="es" href="https://miapp.com/es/" />
<link rel="alternate" hreflang="en" href="https://miapp.com/en/" />
<link rel="alternate" hreflang="fr" href="https://miapp.com/fr/" />
```

- Generar un **sitemap multilingüe** con todas las rutas.  

### 14.7.5. Buenas prácticas

- Usar siempre **prefijos de idioma en la URL**.  
- Definir un idioma por defecto (`/es`).  
- Mantener consistencia en todas las rutas.  
- Añadir `hreflang` y sitemap multilingüe para SEO.  
- Evitar contenido duplicado: cada idioma debe tener traducciones reales.  

Incluir el idioma en la URL y combinarlo con **SSR nativo de Angular 20** es la estrategia más sólida para aplicaciones multilingües.  
El usuario recibe contenido traducido desde el primer render, los buscadores indexan cada idioma como una página independiente y la aplicación escala fácilmente a nuevos mercados.  
En proyectos Enterprise, esta práctica es esencial para garantizar **visibilidad global, rendimiento y SEO competitivo**.

## 14.8. Automatización de traducciones con servicios externos (Google Translate API, DeepL)

En proyectos pequeños, traducir manualmente los archivos JSON de **ngx-translate** puede ser suficiente.  
Pero en aplicaciones **Enterprise**, con decenas de módulos y múltiples idiomas, mantener las traducciones de forma manual se convierte en un cuello de botella.  
Aquí es donde entran en juego los **servicios externos de traducción automática**, como **Google Translate API** o **DeepL API**, que permiten **automatizar la generación inicial de traducciones** y acelerar el trabajo de los equipos de localización.

### 14.8.1. ¿Por qué usar traducción automática?

- **Escalabilidad**: generar traducciones iniciales para cientos de claves en múltiples idiomas.  
- **Velocidad**: reducir el tiempo de entrega de nuevas features multilingües.  
- **Consistencia**: mantener sincronizados los archivos de traducción en todos los idiomas.  
- **Colaboración**: los traductores humanos pueden refinar sobre una base ya generada.  

La traducción automática no sustituye al trabajo humano en contextos críticos (marketing, legal, cultural), pero sí es un **primer paso eficiente**.

### 14.8.2. Flujo de trabajo típico con ngx-translate

1. **Definir idioma base** (ej. `es.json`).  
2. **Extraer claves** con `ngx-translate-extract`.  
3. **Enviar claves a un servicio externo** (Google Translate o DeepL).  
4. **Generar archivos JSON traducidos** (`en.json`, `fr.json`, `de.json`).  
5. **Revisión humana opcional** para asegurar calidad.  
6. **Integración continua (CI/CD)**: automatizar el proceso en cada release.  

### 14.8.3. Uso de Google Translate API

Google ofrece la **Cloud Translation API**, que permite traducir texto mediante peticiones HTTP.

Ejemplo de uso en Node.js para generar traducciones:

```ts
import { TranslationServiceClient } from '@google-cloud/translate';

const client = new TranslationServiceClient();

async function translateText(text: string, targetLang: string) {
  const [response] = await client.translateText({
    parent: `projects/YOUR_PROJECT_ID/locations/global`,
    contents: [text],
    mimeType: 'text/plain',
    targetLanguageCode: targetLang,
    sourceLanguageCode: 'es'
  });

  return response.translations?.[0].translatedText;
}
```

Este script puede recorrer las claves de `es.json` y generar automáticamente `en.json`, `fr.json`, etc.

### 14.8.4. Uso de DeepL API

DeepL es conocido por ofrecer traducciones de mayor calidad en muchos idiomas europeos.  
Su API también permite traducir texto mediante peticiones HTTP.

Ejemplo con `fetch`:

```ts
async function translateWithDeepL(text: string, targetLang: string) {
  const response = await fetch('https://api.deepl.com/v2/translate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      auth_key: 'YOUR_DEEPL_API_KEY',
      text,
      source_lang: 'ES',
      target_lang: targetLang
    })
  });

  const data = await response.json();
  return data.translations[0].text;
}
```

### 14.8.5. Integración con CI/CD

En proyectos Enterprise, lo ideal es integrar este proceso en la **pipeline de CI/CD**:

- Paso 1: Ejecutar `ngx-translate-extract` para generar/actualizar claves.  
- Paso 2: Ejecutar script de traducción automática (Google o DeepL).  
- Paso 3: Guardar los archivos JSON traducidos en el repositorio.  
- Paso 4: Desplegar la aplicación con todos los idiomas actualizados.  

Esto asegura que cada release incluya traducciones actualizadas sin intervención manual.

### 14.8.6. Buenas prácticas

- **Idioma base único**: mantener un idioma de referencia (ej. español o inglés).  
- **Revisión humana**: usar traducción automática como borrador, no como versión final en contextos sensibles.  
- **Control de calidad**: validar que todas las claves estén presentes en todos los idiomas.  
- **Versionado**: mantener los archivos de traducción bajo control de versiones (Git).  
- **Automatización parcial**: usar traducción automática solo para nuevas claves, no para sobrescribir traducciones ya revisadas.  

La **automatización de traducciones** con servicios externos como Google Translate o DeepL es un **acelerador clave** en proyectos multilingües de gran escala.  
Permite mantener sincronizados los idiomas, reducir tiempos de entrega y liberar a los traductores humanos para tareas de revisión y adaptación cultural.  
En combinación con **ngx-translate** y herramientas como **ngx-translate-extract**, se logra un flujo de trabajo **ágil, escalable y sostenible** para aplicaciones Angular 20 orientadas a audiencias globales.


## 14.9. Testing de aplicaciones con soporte multi-idioma

En una aplicación multilingüe, el testing no se limita a verificar que los componentes funcionen: también debemos asegurar que las **traducciones se carguen correctamente**, que el **cambio de idioma sea dinámico**, que las **rutas multilingües funcionen** y que el **SSR entregue contenido traducido**.  
Un error en este aspecto puede afectar tanto la experiencia del usuario como la indexación SEO.

### 14.9.1. Tipos de pruebas necesarias

- **Unit tests (Jasmine/Karma o Jest)**  
  Validan que los servicios de traducción y los componentes funcionen correctamente con distintos idiomas.  

- **Integration tests (TestBed)**  
  Comprueban que los componentes rendericen las traducciones esperadas al cambiar de idioma.  

- **E2E tests (Cypress, Playwright o Protractor)**  
  Simulan la interacción del usuario: seleccionar idioma, navegar entre rutas multilingües y verificar que el contenido cambie dinámicamente.  

- **SSR tests**  
  Verifican que el servidor entregue HTML ya traducido en cada idioma, asegurando compatibilidad con SEO.  

### 14.9.2. Unit tests con ngx-translate

Podemos mockear el `TranslateService` para probar componentes sin necesidad de cargar archivos JSON:

```ts
import { TestBed } from '@angular/core/testing';
import { TranslateService } from '@ngx-translate/core';
import { of } from 'rxjs';
import { HeaderComponent } from './header.component';

describe('HeaderComponent', () => {
  let translateServiceStub: Partial<TranslateService>;

  beforeEach(() => {
    translateServiceStub = {
      get: (key: any) => of(key === 'WELCOME' ? 'Bienvenido' : key),
      use: () => {}
    };

    TestBed.configureTestingModule({
      imports: [HeaderComponent],
      providers: [{ provide: TranslateService, useValue: translateServiceStub }]
    });
  });

  it('debería mostrar el texto traducido', () => {
    const fixture = TestBed.createComponent(HeaderComponent);
    fixture.detectChanges();
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('h1')?.textContent).toContain('Bienvenido');
  });
});
```

### 14.9.3. Integration tests con cambio de idioma

Podemos simular el cambio de idioma y verificar que el texto se actualiza:

```ts
it('debería cambiar el idioma dinámicamente', () => {
  const translate = TestBed.inject(TranslateService);
  translate.use('en');
  const fixture = TestBed.createComponent(HeaderComponent);
  fixture.detectChanges();
  const compiled = fixture.nativeElement as HTMLElement;
  expect(compiled.querySelector('h1')?.textContent).toContain('Welcome');
});
```

### 14.9.4. E2E tests con Cypress

En pruebas end-to-end, verificamos que el selector de idioma funciona y que las rutas multilingües se comportan correctamente:

```js
describe('Selector de idioma', () => {
  it('cambia a inglés y muestra el texto traducido', () => {
    cy.visit('/es');
    cy.get('h1').should('contain', 'Bienvenido');
    cy.get('select#lang').select('en');
    cy.get('h1').should('contain', 'Welcome');
    cy.url().should('include', '/en');
  });
});
```

### 14.9.5. Testing con SSR

Para validar SSR, podemos hacer peticiones HTTP directamente al servidor y comprobar que el HTML ya contiene el texto traducido:

```bash
curl http://localhost:4000/es | grep "Bienvenido"
curl http://localhost:4000/en | grep "Welcome"
```

Esto asegura que el contenido traducido está presente en el HTML inicial, lo que es clave para SEO.

### 14.9.6. Buenas prácticas

- **Mockear traducciones en unit tests** para no depender de archivos externos.  
- **Probar todos los idiomas soportados** en al menos un flujo crítico.  
- **Verificar persistencia de idioma** (ej. guardado en `localStorage`).  
- **Incluir pruebas de rutas multilingües** (`/es`, `/en`, `/fr`).  
- **Automatizar pruebas SSR** en la pipeline de CI/CD.  
- **Validar fallback**: si falta una clave en un idioma, debe mostrarse la del idioma por defecto.  
- **Automatizar la validación de claves** con extractores y scripts en CI/CD para evitar duplicados y claves huérfanas.

El testing en aplicaciones multilingües no es un añadido opcional: es la garantía de que la experiencia del usuario será consistente en todos los idiomas y que la aplicación cumplirá con los requisitos de SEO y accesibilidad global.  
Con una estrategia que combine **unit tests, integration tests, E2E y SSR tests**, podemos asegurar que la internacionalización no se convierta en un punto débil, sino en una fortaleza de la aplicación.


## 14.10. Buenas prácticas y recomendaciones para proyectos globales

Cuando una aplicación Angular evoluciona hacia un producto global, la internacionalización deja de ser un detalle técnico y se convierte en un **pilar estratégico**.  
No basta con traducir textos: hay que pensar en **procesos, arquitectura, rendimiento, SEO y colaboración entre equipos**.  
A continuación, se presentan las mejores prácticas y recomendaciones para garantizar que un proyecto multilingüe sea **escalable, mantenible y competitivo**.

### 14.10.1. Arquitectura y organización del proyecto

- **Modularizar las traducciones**: dividir los archivos por dominio o feature (`auth.json`, `dashboard.json`, `errors.json`) en lugar de un único archivo gigante.  
- **Claves semánticas y jerárquicas**: usar estructuras como `auth.login.title` en lugar de claves planas.  
- **Centralizar la lógica de idioma** en un servicio (`LanguageService`) que gestione idioma actual, persistencia y sincronización con rutas y signals.  
- **Prefijos de idioma en la URL** (`/es`, `/en`, `/fr`) para SEO y claridad.  
- **Evitar duplicados y claves huérfanas**: automatizar la validación con extractores y scripts en CI/CD.

### 14.10.2. Procesos de traducción

- **Idioma base único**: definir un idioma de referencia (ej. inglés o español corporativo).  
- **Automatización con extractores**: usar `ngx-translate-extract` para detectar claves nuevas y eliminar las obsoletas.  
- **Integración con servicios externos**: Google Translate o DeepL para generar traducciones iniciales, con revisión humana posterior.  
- **Plataformas de localización**: integrar con herramientas como Crowdin, Lokalise o Transifex para colaboración entre desarrolladores y traductores.  

### 14.10.3. Rendimiento y carga de traducciones

- **Carga diferida (lazy loading)**: cargar solo las traducciones necesarias por módulo o feature.  
- **SSR multilingüe**: entregar HTML ya traducido desde el servidor para mejorar SEO y rendimiento inicial.  
- **Fallback robusto**: siempre definir un idioma por defecto para evitar pantallas vacías si falta una clave.  
- **Memoización y Signals**: usar Signals para reflejar el idioma actual en la UI sin recalcular innecesariamente.  

### 14.10.4. Testing y calidad

- **Unit tests**: mockear `TranslateService` para validar componentes en distintos idiomas.  
- **E2E tests**: simular cambio de idioma y navegación en rutas multilingües.  
- **SSR tests**: verificar que el HTML inicial ya contiene el contenido traducido.  
- **Validación de claves**: scripts automáticos que aseguren que todos los idiomas tienen las mismas claves.  

### 14.10.5. SEO y accesibilidad

- **URLs claras y consistentes**: cada idioma debe tener su propia ruta.  
- **Etiquetas hreflang**: indicar a los buscadores las variantes de idioma.  
- **Sitemaps multilingües**: incluir todas las rutas traducidas.  
- **Accesibilidad lingüística**: asegurar que lectores de pantalla y herramientas de accesibilidad funcionen correctamente con textos traducidos.  

### 14.10.6. Colaboración y gobernanza

- **Definir un flujo de trabajo claro** entre desarrolladores, traductores y QA.  
- **Versionado de traducciones**: mantener los archivos bajo control de versiones (Git).  
- **Documentación interna**: guías de estilo para claves, convenciones de nombres y procesos de traducción.  
- **Revisión continua**: auditar periódicamente la calidad de las traducciones y la coherencia terminológica.  

Un proyecto global no se mide solo por su código, sino por su capacidad de **adaptarse cultural y lingüísticamente** a distintos mercados.  
Angular 20, junto con **ngx-translate**, SSR nativo y Signals, ofrece todas las herramientas necesarias para construir aplicaciones **multilingües, rápidas y optimizadas para SEO**.  
La clave está en combinar **buenas prácticas técnicas** con **procesos organizativos sólidos**, logrando que la internacionalización sea un **motor de crecimiento global** y no un obstáculo.

