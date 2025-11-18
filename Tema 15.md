# 15. Creación de librerías y documentación avanzada en Angular 20

## 15.1. Creación de librerías con Angular CLI y estructura recomendada

### ¿Qué es una librería en Angular?

Una **librería Angular** es un conjunto de funcionalidades empaquetadas (componentes, directivas, pipes, servicios, etc.) que pueden ser reutilizadas en una o varias aplicaciones Angular. Estas librerías se pueden consumir localmente dentro del mismo workspace o publicarse en un registro como npm para ser utilizadas externamente.

En Angular 20, las librerías deben construirse con componentes **standalone**, lo que significa que no dependen de `NgModules`. Esto simplifica la arquitectura, mejora el rendimiento y facilita la integración.

### Paso a paso: Creación de una librería standalone con Angular CLI

#### 1. Crear un workspace multiproyecto

Angular CLI permite crear un entorno que soporte múltiples proyectos (apps y librerías). Para ello, ejecutamos:

```bash
ng new angular-libs-workspace --create-application=false
cd angular-libs-workspace
```

Este comando genera un workspace vacío, sin aplicación inicial, ideal para alojar librerías y apps de demostración.

#### 2. Crear la librería

Una vez dentro del workspace, creamos la librería con:

```bash
ng generate library ui-components
```

Esto genera una estructura como la siguiente:

    projects/
    └── ui-components/
        ├── src/
        │   ├── lib/
        │   │   └── ui-components.component.ts
        │   └── public-api.ts
        ├── package.json
        └── tsconfig.lib.json

La librería ya está configurada para compilarse de forma independiente y puede ser usada en cualquier aplicación dentro del workspace.

### Estructura recomendada para librerías standalone

A medida que la librería crece, es fundamental organizar el código de forma clara y escalable. Aquí tienes una estructura recomendada:

    projects/ui-components/
    ├── src/
    │   ├── lib/
    │   │   ├── components/
    │   │   │   ├── button/
    │   │   │   │   ├── button.component.ts
    │   │   │   │   ├── button.component.html
    │   │   │   │   └── button.component.scss
    │   │   ├── directives/
    │   │   ├── pipes/
    │   │   ├── services/
    │   │   │   └── theme.service.ts
    │   └── public-api.ts

#### Detalles clave:

*   **components/**: contiene componentes standalone organizados por funcionalidad.
*   **services/**: servicios reutilizables, como gestión de temas, autenticación, etc.
*   **directives/** y **pipes/**: si se usan, deben ser también standalone.
*   **public-api.ts**: define qué elementos se exportan y están disponibles para el consumidor de la librería.

### Configuración del `public-api.ts`

Este archivo define qué partes de la librería se exponen al consumidor. Ejemplo:

```ts
export * from './lib/components/button/button.component';
export * from './lib/services/theme.service';
```

Esto permite que otros proyectos importen directamente `ButtonComponent` o `ThemeService`.

### Uso de la librería en una aplicación del mismo workspace

#### 1. Crear una aplicación de demostración

```bash
ng generate application demo-app
```

#### 2. Usar el componente en `app.component.ts`

```ts
import { Component } from '@angular/core';
import { ButtonComponent } from 'ui-components';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ButtonComponent],
  template: `<lib-button></lib-button>`
})
export class AppComponent {}
```

Angular CLI gestiona los paths automáticamente gracias a `tsconfig.base.json`, por lo que `ui-components` se resuelve correctamente.


## 15.2. Configuración de un Angular Workspace para librerías reutilizables

### **1. ¿Qué es un Workspace en Angular?**

Un *workspace* es el entorno raíz que agrupa aplicaciones y librerías bajo una misma configuración. Para proyectos que incluyen librerías reutilizables, el workspace permite:

*   **Consistencia** en dependencias y configuración.
*   **Facilidad de mantenimiento** y publicación.
*   **Integración con herramientas de documentación y testing**.

***

### **2. Creación del Workspace**

Comando recomendado:

```bash
ng new nombre-workspace --create-application=false --standalone
```

**Explicación de flags:**

*   `--create-application=false`: evita crear una app inicial, dejando el workspace limpio para librerías.
*   `--standalone`: habilita el modo standalone, eliminando la dependencia de NgModules.

### **3. Estructura del Workspace**

Después de crear el workspace, la estructura típica será:

    nombre-workspace/
     ├─ projects/
     │   └─ mi-libreria/
     ├─ angular.json
     ├─ tsconfig.json
     ├─ package.json

**Puntos clave:**

*   `projects/` contendrá las librerías.
*   `angular.json` define *build targets* y configuración global.
*   `tsconfig.json` gestiona paths para importar librerías fácilmente.

### **4. Configuración de Paths para Reutilización**

En `tsconfig.json`:

```json
{
  "compilerOptions": {
    "paths": {
      "@mi-org/mi-libreria": ["projects/mi-libreria/src/public-api.ts"]
    }
  }
}
```

Esto permite importar la librería como:

```typescript
import { MiComponente } from '@mi-org/mi-libreria';
```

### **5. Añadir la Librería**

Comando:

```bash
ng generate library mi-libreria
```

**Características:**

*   Genera la librería en `projects/mi-libreria`.
*   Incluye `public-api.ts` para exportar componentes, servicios y utilidades.

### **6. Configuración para Publicación**

En `projects/mi-libreria/ng-package.json`:

```json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/mi-libreria",
  "lib": {
    "entryFile": "src/public-api.ts"
  }
}
```

Para compilar y empaquetar:

```bash
ng build mi-libreria
```

## 15.3 Definición de APIs públicas y privadas en librerías Angular

### **1. ¿Qué es una API en el contexto de una librería Angular?**

La *API pública* de una librería define **qué partes del código son accesibles para los consumidores** (otras aplicaciones o librerías).  
La *API privada* incluye **implementaciones internas** que no deben ser expuestas fuera del paquete.


### **2. ¿Por qué es importante definir APIs?**

*   **Encapsulación**: evita que los consumidores dependan de detalles internos.
*   **Mantenibilidad**: permite evolucionar la librería sin romper contratos públicos.
*   **Seguridad y rendimiento**: reduce el riesgo de uso indebido y mejora la carga.

### **3. API Pública en Angular**

Se define en el archivo `public-api.ts` generado por Angular CLI:

```typescript
// projects/mi-libreria/src/public-api.ts
export * from './lib/componentes/mi-componente';
export * from './lib/servicios/mi-servicio';
```

**Reglas:**

*   Solo exportar **componentes, directivas, pipes y servicios** que sean parte del contrato público.
*   Evitar exportar clases internas, helpers o configuraciones internas.

### **4. API Privada**

Todo lo que **no se exporta en `public-api.ts`** se considera privado.  
Ejemplo:

```typescript
// projects/mi-libreria/src/lib/utils/interno.ts
export function helperInterno() { ... }
```

Este archivo **no debe aparecer en `public-api.ts`**, por lo que no será accesible desde el paquete publicado.

### **5. Buenas Prácticas para APIs**

*   **Principio de mínimo conocimiento**: expón solo lo necesario.
*   **Documentación clara**: indica qué es público y qué es interno.
*   **Versionado semántico**: cambios en la API pública implican *major version bump*.

### **6. Herramientas para Validar la API**

*   **Compodoc**: genera documentación y permite marcar elementos públicos/privados.
*   **ng-packagr**: asegura que solo lo definido en `public-api.ts` se incluya en el bundle.

### **7. Ejemplo Completo**

```typescript
// public-api.ts
export * from './lib/componentes/boton';
export * from './lib/componentes/input';
export * from './lib/servicios/formulario.service';
```

Archivos internos:

    lib/
     ├─ componentes/
     │   ├─ boton.ts
     │   ├─ input.ts
     ├─ servicios/
     │   ├─ formulario.service.ts
     ├─ utils/
     │   ├─ validaciones-internas.ts  // NO exportado

## 15.4. Empaquetado y optimización para publicación

### **1. ¿Por qué es importante el empaquetado?**

El empaquetado convierte tu librería en un formato listo para distribución, asegurando:

*   **Compatibilidad** con proyectos externos.
*   **Optimización** para reducir tamaño y mejorar rendimiento.
*   **Cumplimiento** con estándares de publicación en npm.

### **2. Herramienta principal: ng-packagr**

Angular utiliza **ng-packagr** para generar paquetes en formato **Angular Package Format (APF)**. Este formato incluye:

*   Código **ESM** (ECMAScript Modules).
*   Tipos TypeScript (`.d.ts`).
*   Archivos optimizados para **tree-shaking**.


### **3. Configuración del empaquetado**

En `projects/mi-libreria/ng-package.json`:

```json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/mi-libreria",
  "lib": {
    "entryFile": "src/public-api.ts"
  },
  "allowedNonPeerDependencies": []
}
```

**Claves:**

*   `dest`: carpeta donde se genera el paquete.
*   `entryFile`: punto de entrada público.
*   `allowedNonPeerDependencies`: lista de dependencias que no son peer (evitar inflar el bundle).


### **4. Comando para empaquetar**

```bash
ng build mi-libreria
```

Esto genera:

    dist/mi-libreria/
     ├─ fesm2022/
     ├─ esm2022/
     ├─ mi-libreria.d.ts
     ├─ package.json


### **5. Optimización del paquete**

*   **Tree-shaking**: asegurarse de exportar solo lo necesario en `public-api.ts`.
*   **SideEffects**: en `package.json` del paquete:

```json
{
  "sideEffects": false
}
```

Esto indica que el código es seguro para eliminación de imports no usados.

*   **Minificación**: Angular CLI y ng-packagr ya generan código optimizado, pero se puede integrar **Terser** para pasos adicionales.

### **6. Publicación en npm**

Pasos:

1.  **Actualizar versión** en `projects/mi-libreria/package.json`.
2.  **Login en npm**:
    ```bash
    npm login
    ```
3.  **Publicar**:
    ```bash
    cd dist/mi-libreria
    npm publish --access public
    ```

## 15.5 Publicación de librerías en NPM y registros privados (GitHub Packages, Azure Artifacts)

### **1. Introducción**

Publicar librerías en **NPM** o en registros privados permite distribuir componentes reutilizables de manera segura y controlada. Esto es esencial para:

*   **Equipos grandes** que comparten librerías internas.
*   **Proyectos open source** que requieren visibilidad pública.
*   **Entornos corporativos** con políticas de seguridad y compliance.

### **2. Preparación antes de publicar**

Antes de publicar, asegúrate de:

*   **Compilar la librería** con `ng build mi-libreria`.
*   Verificar que el contenido en `dist/mi-libreria` incluye:
    *   `package.json` con metadatos correctos.
    *   `README.md` y documentación.
    *   Archivos `.d.ts` para tipado.
*   Configurar **versionado semántico** (SemVer):
    *   `major.minor.patch` → Ejemplo: `1.2.3`.

### **3. Publicación en NPM**

#### **a) Autenticación**

Inicia sesión en NPM:

```bash
npm login
```

Proporciona:

*   **Username**
*   **Password**
*   **Email**

#### **b) Publicación**

Desde la carpeta `dist/mi-libreria`:

```bash
npm publish --access public
```

Para paquetes privados:

```bash
npm publish --access restricted
```

#### **c) Buenas prácticas**

*   Usa `npm pack` para generar un tarball y verificar el contenido antes de publicar.
*   Añade `files` en `package.json` para controlar qué se incluye.

### **4. Publicación en GitHub Packages**

#### **a) Configuración del registro**

En `~/.npmrc`:

    @mi-org:registry=https://npm.pkg.github.com
    //npm.pkg.github.com/:_authToken=TOKEN_PERSONAL

#### **b) Publicación**

Desde `dist/mi-libreria`:

```bash
npm publish --registry=https://npm.pkg.github.com
```

**Notas importantes:**

*   El `scope` (`@mi-org`) debe coincidir con la organización en GitHub.
*   El token debe tener permisos `write:packages`.

### **5. Publicación en Azure Artifacts**

#### **a) Configuración del feed**

En `~/.npmrc`:

    registry=https://pkgs.dev.azure.com/ORGANIZACION/_packaging/NOMBRE_FEED/npm/registry/
    //pkgs.dev.azure.com/ORGANIZACION/_packaging/NOMBRE_FEED/npm/registry/:username=ORGANIZACION
    //pkgs.dev.azure.com/ORGANIZACION/_packaging/NOMBRE_FEED/npm/registry/:_password=TOKEN_BASE64
    //pkgs.dev.azure.com/ORGANIZACION/_packaging/NOMBRE_FEED/npm/registry/:email=EMAIL

#### **b) Publicación**

```bash
npm publish --registry=https://pkgs.dev.azure.com/ORGANIZACION/_packaging/NOMBRE_FEED/npm/registry/
```

### **6. Estrategias avanzadas**

*   **Automatización con CI/CD**:
    *   GitHub Actions o Azure Pipelines para publicar automáticamente en cada release.
*   **Control de versiones**:
    *   Integrar `semantic-release` para generar changelogs y tags.
*   **Firma y verificación**:
    *   Usar `npm sign` y `npm verify` para garantizar integridad.
*   **Monorepos**:
    *   Herramientas como `Nx` para gestionar múltiples librerías y publicación coordinada.

### **7. Seguridad y compliance**

*   **Tokens seguros**: nunca los incluyas en repositorios, usa variables de entorno.
*   **Auditoría**: ejecuta `npm audit` antes de publicar.
*   **Licencias**: define claramente en `package.json` (`"license": "MIT"` o la que corresponda).

## 15.6. Estrategias de versionado semántico y control de dependencias en librerías

### **1. ¿Por qué es importante el versionado semántico?**

El versionado semántico (SemVer) permite comunicar claramente los cambios en una librería:

*   **MAJOR**: cambios incompatibles (breaking changes).
*   **MINOR**: nuevas funcionalidades compatibles.
*   **PATCH**: correcciones sin afectar la API.

Esto es crítico para librerías reutilizables porque:

*   Garantiza estabilidad en proyectos que dependen de ellas.
*   Facilita la automatización en pipelines de CI/CD.

### **2. Configuración del Versionado Semántico**

Herramientas recomendadas:

*   **semantic-release**: automatiza el versionado y publicación en npm.
*   **Conventional Commits**: define un estándar para mensajes de commit.

Ejemplo de instalación:

```bash
npm install --save-dev semantic-release @semantic-release/npm @semantic-release/git
```

Archivo `.releaserc.json`:

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/npm",
    "@semantic-release/git"
  ]
}
```

### **3. Control de Dependencias**

Para evitar conflictos y garantizar compatibilidad:

*   **Usar peerDependencies** para dependencias que deben ser provistas por el consumidor (ej. Angular core).
*   **Usar devDependencies** para herramientas internas (ej. testing, build).
*   **Evitar dependencias innecesarias** que inflen el bundle.

Ejemplo en `package.json`:

```json
{
  "peerDependencies": {
    "@angular/core": "^20.0.0",
    "@angular/common": "^20.0.0"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

### **4. Estrategias Avanzadas**

*   **Bloqueo de versiones críticas**: usar `^` o `~` según el nivel de tolerancia.
*   **Automatización con Renovate o Dependabot**: para mantener dependencias actualizadas.
*   **Validación en CI**: ejecutar `npm audit` y pruebas de compatibilidad antes de publicar.

### **5. Flujo de Publicación Automatizado**

1.  **Commit con convención**: `feat: añade soporte para X`.
2.  **Pipeline CI/CD**:
    *   Ejecuta tests.
    *   Genera changelog.
    *   Publica en npm con la versión calculada por semantic-release.


## 15.7. Documentación de librerías y proyectos con Compodoc

### 1. ¿Qué es Compodoc?

**Compodoc** es una herramienta de documentación para aplicaciones Angular. Genera documentación estática en formato HTML a partir del código fuente, incluyendo:

*   Componentes, servicios, directivas, pipes, interfaces, clases, etc.
*   Diagrama de dependencias entre módulos.
*   Documentación de rutas.
*   Documentación de pruebas unitarias.
*   Comentarios JSDoc.

**Ventajas:**

*   Fácil de usar.
*   Compatible con Angular CLI.
*   Genera documentación visual y navegable.
*   Ideal para equipos grandes y librerías compartidas.

### 2. Instalación de Compodoc

#### Requisitos previos

*   Angular CLI instalado.
*   Proyecto Angular 20 ya creado (aplicación o librería).

#### Instalación

```bash
npm install -g @compodoc/compodoc
```

O como dependencia de desarrollo:

```bash
npm install --save-dev @compodoc/compodoc
```

### 3. Generar documentación básica

#### Ejemplo práctico

Supongamos que tienes una librería llamada `awesome-ui-lib`.

##### Paso 1: Generar la documentación

```bash
npx compodoc -p projects/awesome-ui-lib/tsconfig.lib.json
```

*   `-p`: Ruta al archivo `tsconfig` del proyecto o librería.

##### Paso 2: Servir la documentación

```bash
npx compodoc -s
```

Esto abrirá un servidor local en `http://localhost:8080` con la documentación navegable.

### 4. Documentar correctamente con JSDoc

Compodoc utiliza los comentarios **JSDoc** para enriquecer la documentación.

#### Ejemplo:

```ts
/**
 * Componente de botón reutilizable.
 *
 * @example
 * <aui-button [label]="'Guardar'" (click)="guardar()"></aui-button>
 */
@Component({
  selector: 'aui-button',
  templateUrl: './button.component.html'
})
export class ButtonComponent {
  /**
   * Etiqueta del botón.
   */
  @Input() label: string = 'Botón';

  /**
   * Evento emitido al hacer clic.
   */
  @Output() click = new EventEmitter<void>();
}
```

### 5. Personalización de la documentación

Puedes personalizar el tema, título, logotipo y más:

```bash
npx compodoc -p tsconfig.json -d docs -t my-theme --name "Awesome UI Lib" --logo src/assets/logo.png
```

*   `-d`: Carpeta de salida.
*   `--name`: Nombre del proyecto.
*   `--logo`: Ruta al logotipo.
*   `-t`: Tema (por ejemplo: `default`, `postmark`, `vagrant`, etc.).

### 6. Documentar rutas y navegación

Compodoc también genera documentación de rutas si usas el módulo de enrutamiento de Angular.

#### Requisitos:

*   Usar `RouterModule.forRoot()` o `forChild()` correctamente.
*   Definir rutas con `path`, `component`, `loadChildren`, etc.

#### Ejemplo:

```ts
const routes: Routes = [
  { path: 'inicio', component: HomeComponent },
  { path: 'perfil', loadChildren: () => import('./perfil/perfil.module').then(m => m.PerfilModule) }
];
```

Compodoc generará un **diagrama de rutas** automáticamente.

### 7. Integración con librerías standalone

En Angular 20, muchas librerías se desarrollan como **standalone**. Compodoc también las documenta correctamente si están bien estructuradas.

#### Ejemplo de componente standalone:

```ts
/**
 * Componente de alerta standalone.
 */
@Component({
  standalone: true,
  selector: 'aui-alert',
  templateUrl: './alert.component.html',
  imports: [CommonModule]
})
export class AlertComponent {
  @Input() type: 'success' | 'error' = 'success';
}
```

Compodoc detectará este componente y lo incluirá en la documentación.

### 8. Automatización con scripts

Puedes agregar scripts en `package.json` para facilitar su uso:

```json
"scripts": {
  "docs": "compodoc -p tsconfig.json -d docs",
  "docs:serve": "compodoc -s -d docs"
}
```

### 9. Publicar la documentación

Puedes subir la carpeta generada (`docs/`) a:

*   GitHub Pages
*   Servidores estáticos (Netlify, Vercel, etc.)
*   Incluirla como parte del portal de documentación de tu empresa

### 10. Buenas prácticas

*   Documenta todos los `@Input`, `@Output`, métodos públicos y clases.
*   Usa ejemplos en los comentarios.
*   Mantén la documentación actualizada con cada release.
*   Automatiza la generación en pipelines CI/CD.

## 15.8. Integración de Storybook para documentar componentes UI

### ¿Qué es Storybook?

**Storybook** es una herramienta de desarrollo de interfaces de usuario que permite construir, probar y documentar componentes de forma aislada. Es especialmente útil en proyectos Angular donde los componentes son piezas reutilizables y modulares.

**Ventajas de usar Storybook:**

*   Permite visualizar componentes de forma independiente del resto de la aplicación.
*   Facilita el desarrollo basado en componentes (Component-Driven Development).
*   Mejora la colaboración entre desarrolladores y diseñadores.
*   Permite generar documentación interactiva.
*   Se integra fácilmente con herramientas de testing visual y de regresión visual (como Chromatic).

### Instalación de Storybook en un proyecto Angular

Para integrar Storybook en un proyecto Angular (incluyendo librerías standalone), sigue estos pasos:

#### 1. Instalar Storybook

Desde la raíz del proyecto:

```bash
npx storybook@latest init
```

Este comando detectará automáticamente que estás usando Angular y configurará Storybook con los ajustes necesarios.

#### 2. Verifica la estructura generada

Se creará una carpeta `.storybook/` con los siguientes archivos:

*   `main.ts`: configuración principal de Storybook.
*   `preview.ts`: configuración global de estilos y decoradores.
*   `tsconfig.json`: configuración de TypeScript para Storybook.

También se generará una historia de ejemplo en `src/stories`.

### Crear historias para tus componentes

Una **historia** en Storybook representa un estado de un componente. Para cada componente UI, puedes crear un archivo `.stories.ts` que defina sus diferentes variantes.

#### Ejemplo: `button.component.stories.ts`

```ts
import { Meta, Story } from '@storybook/angular';
import { MyButtonComponent } from './my-button.component';

export default {
  title: 'Components/MyButton',
  component: MyButtonComponent,
  tags: ['autodocs'],
} as Meta<MyButtonComponent>;

const Template: Story<MyButtonComponent> = (args: MyButtonComponent) => ({
  props: args,
});

export const Primary = Template.bind({});
Primary.args = {
  label: 'Botón Primario',
  type: 'primary',
};

export const Secondary = Template.bind({});
Secondary.args = {
  label: 'Botón Secundario',
  type: 'secondary',
};
```

### Documentación automática con `autodocs`

Desde Storybook 7, puedes usar la etiqueta `tags: ['autodocs']` para generar documentación automática a partir de los metadatos del componente Angular (inputs, outputs, etc.).

Asegúrate de tener habilitado el addon de documentación en `main.ts`:

```ts
export default {
  stories: ['../src/**/*.stories.@(js|ts)'],
  addons: ['@storybook/addon-docs'],
  framework: {
    name: '@storybook/angular',
    options: {},
  },
};
```

### Integración con librerías standalone

Si estás documentando componentes dentro de una **librería Angular standalone**, ten en cuenta lo siguiente:

1.  **Importación de módulos**: Asegúrate de importar los módulos necesarios en cada historia.
2.  **Uso de `standalone: true`**: Si tus componentes son standalone, puedes usarlos directamente sin necesidad de declarar un módulo.

#### Ejemplo:

```ts
import { Meta, Story } from '@storybook/angular';
import { MyStandaloneComponent } from './my-standalone.component';

export default {
  title: 'Standalone/MyComponent',
  component: MyStandaloneComponent,
  standalone: true,
} as Meta;

const Template: Story = (args) => ({
  props: args,
});

export const Default = Template.bind({});
Default.args = {
  title: 'Hola desde Storybook',
};
```

### Personalización y configuración avanzada

Puedes personalizar Storybook para adaptarlo a tus necesidades:

*   **Temas personalizados** con `@storybook/theming`.
*   **Decoradores globales** para aplicar estilos o wrappers comunes.
*   **Addons útiles**:
    *   `@storybook/addon-controls`: para manipular props en tiempo real.
    *   `@storybook/addon-a11y`: para accesibilidad.
    *   `@storybook/addon-interactions`: para pruebas de interacción.

### Integración con CI/CD

Puedes automatizar la generación y despliegue de la documentación de Storybook en tu pipeline de CI/CD. Algunas opciones:

*   **Publicar en GitHub Pages**.
*   **Integración con Chromatic** para pruebas visuales y hosting.
*   **Generar documentación estática**:

```bash
npm run build-storybook
```

Esto generará una carpeta `storybook-static/` que puedes desplegar en cualquier servidor web.

### Buenas prácticas

*   **Una historia por caso de uso**: documenta variantes significativas.
*   **Usa controles para inputs**: mejora la experiencia interactiva.
*   **Documenta los inputs y outputs** con JSDoc.
*   **Agrupa componentes por dominio o funcionalidad** en la jerarquía de Storybook.


## 15.9. Automatización de documentación con IA (Copilot, Cursor, WindsurfAI, ChatGPT)

La documentación técnica es una parte esencial del desarrollo de software, especialmente en proyectos complejos como los desarrollados con Angular. Sin embargo, escribir y mantener documentación puede ser una tarea repetitiva y propensa a errores. Aquí es donde las herramientas de inteligencia artificial (IA) pueden marcar una gran diferencia, automatizando partes del proceso y mejorando la calidad y consistencia de la documentación.

En este apartado exploraremos cómo aprovechar herramientas basadas en IA como **GitHub Copilot**, **Cursor**, **WindsurfAI** y **ChatGPT** para automatizar y enriquecer la documentación de librerías y proyectos Angular.

### 1. GitHub Copilot: Generación de comentarios y documentación en tiempo real

**GitHub Copilot**, desarrollado por GitHub y OpenAI, es un asistente de codificación que sugiere líneas de código y funciones completas en tiempo real. Además de ayudar a escribir código, también puede generar comentarios y documentación básica.

#### Casos de uso en documentación:

*   **Generación automática de comentarios JSDoc**:
    Al escribir una función o clase, Copilot puede sugerir automáticamente un bloque de documentación JSDoc con descripciones de parámetros y valores de retorno.

    ```ts
    /**
     * Calcula el total con IVA incluido.
     * @param precioBase El precio sin IVA.
     * @param iva Porcentaje de IVA.
     * @returns El precio total con IVA.
     */
    function calcularTotal(precioBase: number, iva: number): number {
      return precioBase * (1 + iva / 100);
    }
    ```

*   **Sugerencias de nombres y descripciones**:
    Copilot puede ayudarte a encontrar nombres más descriptivos para funciones, variables o componentes, lo que mejora la legibilidad y la documentación implícita del código.

#### Limitaciones:

*   No siempre genera descripciones precisas: es importante revisar y ajustar las sugerencias.
*   No reemplaza la documentación estructurada como la que genera Compodoc.


### 2. Cursor: IDE con IA integrada para documentación contextual

**Cursor** es un editor de código basado en VS Code que integra IA de forma nativa. A diferencia de Copilot, Cursor permite interactuar con la IA directamente sobre el código, haciendo preguntas o solicitando tareas específicas.

#### Casos de uso en documentación:

*   **Generación de documentación a partir de código existente**:
    Puedes seleccionar una función o componente y pedirle a Cursor que genere una descripción detallada de su funcionamiento.

*   **Refactorización con documentación**:
    Al refactorizar código, Cursor puede ayudarte a actualizar automáticamente los comentarios y la documentación asociada.

*   **Explicación de código heredado**:
    Ideal para entender código legado y generar documentación explicativa para nuevos miembros del equipo.

#### Ejemplo de uso:

Seleccionas un componente Angular y escribes:

> "Explica qué hace este componente y genera un resumen para la documentación técnica."

Cursor responde con una descripción detallada que puedes copiar directamente a tu documentación.


### 3. WindsurfAI: Documentación técnica asistida por IA

**WindsurfAI** es una herramienta especializada en generar documentación técnica a partir de código fuente. A diferencia de Copilot o Cursor, WindsurfAI está orientada específicamente a la **documentación estructurada**, lo que la hace ideal para proyectos grandes o librerías compartidas.

#### Características destacadas:

*   **Análisis semántico del código Angular**: reconoce estructuras como componentes, servicios, directivas, etc.
*   **Generación de documentación Markdown o HTML**: ideal para integrarse con Compodoc o Storybook.
*   **Soporte para múltiples lenguajes y frameworks**.

#### Flujo de trabajo sugerido:

1.  Subes tu repositorio o conectas tu cuenta de GitHub.
2.  WindsurfAI analiza el código y genera documentación técnica.
3.  Puedes editar, aprobar y exportar la documentación generada.


### 4. ChatGPT: Generación y revisión de documentación personalizada

**ChatGPT** es una herramienta versátil que puede ayudarte a:

*   **Redactar documentación técnica a partir de descripciones o código**.
*   **Revisar y mejorar documentación existente**.
*   **Traducir documentación técnica a otros idiomas**.
*   **Generar ejemplos de uso para componentes o servicios Angular**.

#### Ejemplo práctico:

Puedes copiar el código de un componente Angular y pedir:

> "Genera una sección de documentación para este componente, incluyendo descripción, inputs, outputs y un ejemplo de uso."

ChatGPT puede devolverte algo como:

````markdown
### Componente: `UserCardComponent`

Este componente muestra la información de un usuario en formato de tarjeta.

#### Inputs:
- `user: User` – Objeto con los datos del usuario.

#### Outputs:
- `onSelect: EventEmitter<User>` – Se emite cuando se selecciona la tarjeta.

#### Ejemplo de uso:

```html
<app-user-card [user]="usuario" (onSelect)="seleccionarUsuario($event)"></app-user-card>
```
```` 

### Recomendaciones para integrar IA en tu flujo de documentación

- **Combina herramientas**: Usa Copilot para comentarios rápidos, Cursor para explicaciones contextuales, WindsurfAI para documentación estructurada y ChatGPT para redacción avanzada.
- **Revisión humana**: Siempre revisa y ajusta la documentación generada por IA.
- **Automatiza en CI/CD**: Puedes integrar herramientas como WindsurfAI o scripts con ChatGPT API para generar documentación automáticamente en cada build.


## 15.10. Buenas prácticas para librerías internas (enterprise) vs open source

Al desarrollar librerías en Angular, es fundamental adaptar las prácticas de documentación y mantenimiento según el contexto en el que se utilizarán: **proyectos internos (enterprise)** o **proyectos de código abierto (open source)**. Aunque ambos comparten principios comunes, existen diferencias clave en sus objetivos, audiencias y requisitos de documentación.


### Librerías Internas (Enterprise)

Estas librerías están destinadas a ser utilizadas dentro de una organización, por equipos internos. Su objetivo principal es **estandarizar componentes, servicios o utilidades** para mejorar la productividad y la coherencia entre proyectos.

#### Buenas prácticas:

1.  **Documentación centrada en el equipo**:
    *   Prioriza la claridad y la utilidad para los desarrolladores internos.
    *   Incluye convenciones internas, decisiones de diseño y ejemplos de uso específicos del negocio.

2.  **Integración con herramientas internas**:
    *   Automatiza la generación de documentación con herramientas como **Compodoc** en pipelines de CI/CD.
    *   Usa **Storybook** para documentar componentes visuales y facilitar su reutilización.

3.  **Control de versiones y compatibilidad**:
    *   Mantén un changelog claro y versiones semánticas (semver).
    *   Documenta los cambios que afectan a otros equipos (breaking changes, nuevas APIs, deprecaciones).

4.  **Privacidad y seguridad**:
    *   Evita exponer información sensible en la documentación.
    *   Usa herramientas de control de acceso si la documentación se publica en servidores internos.

5.  **Automatización con IA**:
    *   Usa **Copilot** o **Cursor** para generar documentación técnica rápida.
    *   Integra **ChatGPT** o **WindsurfAI** para generar documentación contextual a partir de código y commits.

6.  **Documentación viva**:
    *   Fomenta la actualización continua de la documentación como parte del flujo de trabajo.
    *   Usa pull requests para revisar y validar cambios en la documentación.

### Librerías Open Source

Las librerías open source están destinadas a una audiencia más amplia y diversa. La documentación es clave para la adopción, el soporte comunitario y la colaboración.

#### Buenas prácticas:

1.  **Documentación pública y accesible**:
    *   Publica la documentación en sitios como GitHub Pages, Netlify o Vercel.
    *   Usa herramientas como **Compodoc**, **Typedoc** o **Docusaurus** para generar documentación navegable.

2.  **Guías de inicio rápido y ejemplos claros**:
    *   Incluye una sección de *Getting Started* con ejemplos mínimos funcionales.
    *   Proporciona ejemplos de integración en proyectos reales.

3.  **Storybook como documentación interactiva**:
    *   Usa **Storybook** para mostrar componentes en acción.
    *   Añade controles y documentación embebida para facilitar la exploración.

4.  **Internacionalización de la documentación**:
    *   Considera traducir la documentación a varios idiomas si la comunidad lo requiere.
    *   Usa herramientas como **Crowdin** o **i18next** para gestionar traducciones.

5.  **Contribución abierta y mantenibilidad**:
    *   Incluye un archivo `CONTRIBUTING.md` con pautas claras para contribuir.
    *   Usa etiquetas como `good first issue` para fomentar la participación.

6.  **Automatización con IA para escalar**:
    *   Usa **ChatGPT** para generar borradores de documentación a partir de issues o pull requests.
    *   Automatiza la generación de changelogs y notas de versión con herramientas como **semantic-release**.

### Comparativa rápida

| Aspecto                   | Librerías Internas (Enterprise)  | Librerías Open Source           |
| ------------------------- | -------------------------------- | ------------------------------- |
| Audiencia                 | Equipos internos                 | Comunidad global                |
| Nivel de detalle          | Alto, con foco en contexto local | Alto, con foco en claridad      |
| Herramientas recomendadas | Compodoc, Storybook, IA local    | Compodoc, Storybook, Docusaurus |
| Automatización con IA     | En flujos CI/CD internos         | Para escalar documentación      |
| Seguridad                 | Alta (datos sensibles)           | Pública, sin datos privados     |
| Estilo de documentación   | Técnica y específica             | Accesible y pedagógica          |
