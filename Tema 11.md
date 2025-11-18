# 11. Arquitecturas escalables con Angular

## 11.1. Introducción a Angular Workspaces para proyectos multi-app

Cuando una aplicación Angular crece, no solo lo hace en tamaño de código, sino también en complejidad organizativa.  
En entornos profesionales —especialmente en proyectos Enterprise— es común que una misma base de código aloje **varias aplicaciones**, **librerías compartidas**, y **módulos reutilizables** que deben mantenerse de forma coherente y escalable.

Aquí es donde entran los **Angular Workspaces**: una estructura de proyecto que permite **agrupar múltiples apps y librerías en un solo repositorio**, con herramientas integradas para facilitar el desarrollo, el testing, el versionado y el despliegue.

### ¿Qué es un Workspace en Angular?

Un Workspace es un contenedor lógico que puede incluir:

- Una o más aplicaciones Angular (por ejemplo, una app pública, una app de administración, una app móvil).
- Librerías reutilizables (por ejemplo, componentes UI, servicios comunes, validadores, pipes).
- Herramientas de desarrollo compartidas (tipos, mocks, utilidades de testing).
- Configuración centralizada (tsconfig, linting, testing, CI/CD).

Todo esto vive bajo una misma raíz, con una estructura clara y mantenible.

### ¿Por qué usar Workspaces en proyectos multi-app?

La necesidad surge cuando:

- Tienes **varias aplicaciones** que comparten lógica o diseño.
- Quieres **reutilizar componentes** sin duplicar código.
- Necesitas **versionar librerías internas** de forma controlada.
- Buscas **consistencia en testing, linting y configuración** entre apps.
- Quieres **escalar el equipo** y dividir responsabilidades por dominio.

En lugar de tener múltiples repositorios aislados, un Workspace permite **centralizar el desarrollo** y mantener una arquitectura modular.

### Estructura típica de un Workspace

```plaintext
my-workspace/
├── projects/
│   ├── admin-app/
│   ├── public-app/
│   ├── shared-ui-lib/
│   └── auth-lib/
├── angular.json
├── package.json
├── tsconfig.json
└── README.md
```

- La carpeta `projects/` contiene todas las apps y librerías.
- Cada entrada tiene su propio `tsconfig`, `testing`, `assets`, etc.
- El archivo `angular.json` define cómo se compilan, sirven y testean cada uno.

### Crear un Workspace desde cero

Angular CLI permite crear un Workspace vacío y añadir apps o librerías según lo necesites.

```bash
ng new my-workspace --create-application=false
cd my-workspace
ng generate application public-app
ng generate application admin-app
ng generate library shared-ui-lib
```

Esto genera una estructura modular donde cada app y librería tiene su propio espacio, pero comparten configuración base.

### Cómo se comunican las piezas

Las librerías se importan como cualquier módulo Angular, pero con rutas relativas o alias definidos en `tsconfig.json`.  
Por ejemplo:

```ts
import { ButtonComponent } from '@shared-ui-lib/button';
```

Esto permite que los equipos trabajen en paralelo: mientras el equipo de UI mejora la librería de componentes, el equipo de producto puede seguir desarrollando la app sin duplicar código.

### Testing y mantenimiento en Workspaces

Cada app y librería puede tener su propia suite de pruebas (unitarias, integración, E2E), pero también puedes configurar testing compartido:

- **Mocks comunes** para servicios.
- **Harnesses reutilizables** para componentes.
- **Configuración de Jest o Cypress** centralizada.

Esto reduce duplicación y asegura que todos los módulos se testean bajo los mismos estándares.

### Escalabilidad organizativa

En proyectos grandes, los Workspaces permiten:

- **Separar dominios funcionales**: por ejemplo, `projects/catalog`, `projects/checkout`, `projects/user-profile`.
- **Asignar ownership por carpeta**: cada equipo mantiene su módulo.
- **Versionar librerías internas**: útil si usas herramientas como Nx o Lerna.
- **Desplegar apps por separado**: cada app puede tener su propio pipeline y entorno.

### Buenas prácticas iniciales

- Usa nombres claros y consistentes para apps y librerías.
- Define alias en `tsconfig.json` para evitar rutas largas.
- Centraliza configuración de linting, testing y estilos.
- Documenta la estructura y convenciones del Workspace.
- Mantén las librerías pequeñas y enfocadas: una responsabilidad por módulo.

### ¿Y si ya tengo una app monolítica?

Puedes migrar gradualmente:

1. Crear un Workspace vacío.
2. Mover tu app actual como `projects/main-app`.
3. Extraer partes comunes como librerías (`projects/shared`, `projects/core`).
4. Añadir nuevas apps como `admin-app`, `mobile-app`, etc.
5. Refactorizar imports y configuración.

Este enfoque permite escalar sin romper lo que ya funciona.


## 11.2. Uso de librerías compartidas entre diferentes aplicaciones Angular

En una arquitectura Angular escalable, especialmente dentro de un Workspace multi-app, las **librerías compartidas** son el mecanismo que permite **reutilizar código**, **mantener consistencia** y **reducir duplicación** entre aplicaciones que conviven en el mismo ecosistema.

No se trata solo de extraer componentes comunes: hablamos de **modularizar el conocimiento técnico y funcional** de la organización en piezas reutilizables que puedan evolucionar de forma independiente, pero mantenerse sincronizadas.

### ¿Qué tipo de código se comparte?

La idea no es compartir todo, sino identificar qué elementos tienen sentido como librería:

- **Componentes UI reutilizables**: botones, formularios, modales, layouts.
- **Servicios comunes**: autenticación, logging, almacenamiento local, internacionalización.
- **Pipes y directivas**: formateadores de fecha, moneda, validadores visuales.
- **Interfaces y modelos**: contratos de datos que deben ser consistentes entre apps.
- **Utilidades de testing**: mocks, Harnesses, helpers para pruebas unitarias o E2E.

Estas librerías pueden ser tan simples como un conjunto de componentes o tan complejas como un módulo de negocio completo (por ejemplo, `lib-catalogo`, `lib-usuarios`, `lib-pedidos`).

### Cómo se estructuran dentro del Workspace

En un Workspace Angular, las librerías se crean dentro de la carpeta `projects/`, igual que las aplicaciones.  
Cada una tiene su propio espacio, configuración y ciclo de vida.

Ejemplo:

```plaintext
projects/
├── public-app/
├── admin-app/
├── shared-ui/
├── core-services/
├── domain-catalog/
```

- `shared-ui`: componentes visuales reutilizables.
- `core-services`: lógica transversal (auth, storage, API).
- `domain-catalog`: lógica de negocio específica, pero compartida entre apps.

### Crear una librería compartida

Angular CLI lo hace sencillo:

```bash
ng generate library shared-ui
```

Esto genera:

```plaintext
projects/shared-ui/
├── src/
│   ├── lib/
│   │   └── shared-ui.module.ts
│   └── public-api.ts
├── package.json
├── tsconfig.lib.json
```

Puedes añadir componentes, servicios, pipes, etc., y exportarlos desde `public-api.ts` para que estén disponibles en otras apps.

### Importar una librería en una aplicación

Desde cualquier aplicación del Workspace, puedes importar módulos o elementos de la librería:

```ts
import { SharedUiModule } from 'shared-ui';
```

Esto funciona gracias a los alias definidos en `tsconfig.json`:

```json
{
  "compilerOptions": {
    "paths": {
      "shared-ui": ["projects/shared-ui/src/public-api"]
    }
  }
}
```

Este alias permite importar sin rutas relativas largas ni errores de resolución.

### Versionado y mantenimiento

Aunque las librerías viven en el mismo repositorio, conviene tratarlas como **entidades independientes**:

- **Documenta su API pública**: qué expone, qué espera, cómo se usa.
- **Evita dependencias circulares**: una librería no debe depender de otra que depende de ella.
- **Usa testing dedicado**: cada librería debe tener su propia suite de pruebas.
- **Controla los cambios**: si una librería cambia, asegúrate de que las apps que la usan se actualizan correctamente.

En proyectos grandes, puedes incluso versionar las librerías internamente (por ejemplo, `shared-ui@2.1.0`) y usar herramientas como Nx para gestionar dependencias y afectaciones.

### Buenas prácticas

- **Una responsabilidad por librería**: no mezcles UI con lógica de negocio.
- **Evita acoplamientos innecesarios**: no hagas que una librería dependa de una app concreta.
- **Centraliza contratos de datos**: interfaces y modelos deben vivir en librerías compartidas.
- **Reutiliza en pruebas**: mocks, Harnesses y helpers deben estar disponibles para todas las apps.
- **Mantén la API pública limpia**: exporta solo lo que realmente debe usarse fuera.

### Ejemplo realista

Supongamos que tienes dos aplicaciones: una pública (`public-app`) y una de administración (`admin-app`).  
Ambas necesitan:

- Mostrar productos en una tarjeta.
- Validar formularios de contacto.
- Acceder al estado de autenticación.

Puedes crear tres librerías:

- `shared-ui`: con `ProductCardComponent`, `ContactFormComponent` (preferentemente como standalone components y usando signals para estado).
- `core-auth`: con `AuthService`, `AuthGuard` (servicios standalone y signals para estado).
- `shared-forms`: con validadores reutilizables y formularios tipados.

Cada app importa solo lo que necesita usando alias definidos en `tsconfig.json`, y cualquier mejora en la librería beneficia a ambas sin duplicar esfuerzo.


## 11.3. Creación de Schematics personalizados para automatizar generación de código

En un proyecto Angular pequeño, escribir manualmente un componente, un servicio o un módulo no supone un gran esfuerzo. Pero en un **ecosistema Enterprise**, con múltiples aplicaciones, librerías compartidas y decenas de desarrolladores trabajando en paralelo, la repetición de tareas y la falta de consistencia se convierten en un problema real.  
Aquí es donde los **Schematics personalizados** se transforman en aliados estratégicos: permiten **convertir las convenciones de tu equipo en comandos CLI reutilizables**, asegurando que cada pieza de código se genera con la misma estructura, estilo y buenas prácticas.

Podemos pensar en los Schematics como **plantillas inteligentes**. No solo copian archivos, sino que **entienden el contexto del proyecto** y aplican reglas:  
- ¿Dónde debe ir este nuevo módulo?  
- ¿Qué configuración de routing necesita?  
- ¿Debe incluir un guard, un resolver o un servicio asociado?  
- ¿Qué estrategia de Change Detection debe aplicarse por defecto?  

En lugar de confiar en que cada desarrollador recuerde estas decisiones, el Schematic las automatiza. Así, la arquitectura deja de ser un documento en Confluence y se convierte en **código ejecutable**.

### Ejemplo completo: un Schematic para generar un componente con OnPush y test inicial

Para ilustrar el valor de los Schematics, veamos un ejemplo real: un generador de componentes que siempre crea la misma estructura, con `ChangeDetectionStrategy.OnPush`, un HTML base, un archivo de estilos y un test inicial en Jest.

#### Estructura del Schematic

```plaintext
my-schematics/
├── src/
│   └── component/
│       ├── index.ts
│       ├── schema.json
│       └── files/
│           ├── __name__.component.ts
│           ├── __name__.component.html
│           └── __name__.component.spec.ts
├── collection.json
├── package.json
```

#### `schema.json`

```json
{
  "$schema": "http://json-schema.org/schema",
  "id": "CustomComponentSchema",
  "title": "Custom Component Options",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Nombre del componente"
    },
    "path": {
      "type": "string",
      "description": "Ruta donde se generará el componente"
    }
  },
  "required": ["name"]
}
```

#### `index.ts`

```ts
import { Rule, SchematicContext, Tree, apply, url, template, move, mergeWith } from '@angular-devkit/schematics';
import { strings } from '@angular-devkit/core';

export function customComponent(options: any): Rule {
  return (tree: Tree, _context: SchematicContext) => {
    const sourceTemplates = apply(url('./files'), [
      template({
        ...options,
        ...strings
      }),
      move(options.path || 'src/app/components')
    ]);

    return mergeWith(sourceTemplates)(tree, _context);
  };
}
```

#### `collection.json`

```json
{
  "schematics": {
    "custom-component": {
      "description": "Genera un componente con OnPush y test inicial",
      "factory": "./src/component/index#customComponent",
      "schema": "./src/component/schema.json"
    }
  }
}
```

#### Plantillas en `files/`

##### `__name__.component.ts`

```ts
import { ChangeDetectionStrategy, Component } from '@angular/core';

@Component({
  selector: 'app-<%= dasherize(name) %>',
  templateUrl: './<%= dasherize(name) %>.component.html',
  styleUrls: ['./<%= dasherize(name) %>.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class <%= classify(name) %>Component {}
```

> Nota: Si el componente gestiona estado, se recomienda usar signals en vez de variables tradicionales.

##### `__name__.component.html`

```html
<p><%= dasherize(name) %> works!</p>
```

##### `__name__.component.spec.ts`

```ts
import { TestBed } from '@angular/core/testing';
import { <%= classify(name) %>Component } from './<%= dasherize(name) %>.component';

describe('<%= classify(name) %>Component', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [<%= classify(name) %>Component]
    });
  });

  it('should create', () => {
    const fixture = TestBed.createComponent(<%= classify(name) %>Component);
    const comp = fixture.componentInstance;
    expect(comp).toBeTruthy();
  });
});
```

### Ejecución del Schematic

Con todo configurado, basta con ejecutar:

```bash
ng generate my-schematics:custom-component --name=card --path=src/app/ui
```

Esto genera automáticamente:

```plaintext
src/app/ui/card/
├── card.component.ts
├── card.component.html
├── card.component.scss
└── card.component.spec.ts
```

#### Código generado

##### `card.component.ts`

```ts
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-card',
  templateUrl: './card.component.html',
  styleUrls: ['./card.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class CardComponent {}
```

##### `card.component.html`

```html
<p>card works!</p>
```

##### `card.component.scss`

```scss
:host {
  display: block;
}
```

##### `card.component.spec.ts`

```ts
import { TestBed } from '@angular/core/testing';
import { CardComponent } from './card.component';

describe('CardComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [CardComponent]
    });
  });

  it('should create', () => {
    const fixture = TestBed.createComponent(CardComponent);
    const comp = fixture.componentInstance;
    expect(comp).toBeTruthy();
  });
});
```


Este ejemplo muestra cómo un Schematic convierte una **convención de equipo** en un **comando reproducible**.  
En lugar de confiar en la memoria de cada desarrollador, el CLI se encarga de generar siempre la misma estructura, con las mismas buenas prácticas.  
En proyectos Enterprise, esto significa **consistencia, velocidad y reducción de errores humanos**.  


## 11.4. Estrategias de monorepos y multirepos en entornos enterprise

Cuando hablamos de **arquitecturas escalables con Angular** en entornos Enterprise, no basta con pensar en cómo organizar el código dentro de una aplicación: también debemos decidir **cómo organizar el código a nivel de repositorio**.  
La elección entre un **monorepo** (un único repositorio que contiene múltiples aplicaciones y librerías) y un **multirepo** (varios repositorios independientes, cada uno con su propio ciclo de vida) no es trivial.  
Ambas estrategias tienen ventajas y desventajas, y la decisión suele depender tanto de factores técnicos como de la cultura y estructura de la organización.

### El enfoque monorepo

En un monorepo, todo el código vive en un único repositorio: aplicaciones, librerías compartidas, utilidades, incluso microservicios front-end.  
Herramientas como **Nx** o **Turborepo** han popularizado este enfoque en el ecosistema Angular, ya que facilitan la gestión de dependencias, la ejecución selectiva de tests y la construcción incremental.

**Ventajas:**
- **Consistencia total**: un único set de reglas de linting, testing y build para todo el ecosistema.
- **Reutilización inmediata**: las librerías compartidas se consumen directamente, sin necesidad de publicarlas en un registry.
- **Refactors globales**: cambiar una interfaz o modelo se propaga a todas las apps en un solo commit.
- **Visibilidad completa**: todos los equipos ven el mismo código, lo que reduce duplicación.

**Desafíos:**
- **Escalabilidad del repositorio**: a medida que crece, los tiempos de CI/CD pueden aumentar si no se optimiza con caching y ejecución distribuida.
- **Gobernanza**: requiere disciplina para evitar que cualquier equipo modifique cualquier parte sin control.
- **Curva de aprendizaje**: herramientas como Nx añaden complejidad inicial.

### El enfoque multirepo

En un multirepo, cada aplicación o librería vive en su propio repositorio, con su ciclo de vida independiente.  
Este modelo es más tradicional y sigue siendo común en organizaciones con equipos muy autónomos o con políticas de seguridad estrictas.

**Ventajas:**
- **Aislamiento fuerte**: cada equipo controla su repo, sus releases y su pipeline.
- **Ciclos de vida independientes**: una app puede evolucionar sin esperar a que otra esté lista.
- **Simplicidad percibida**: no requiere herramientas adicionales para gestionar la complejidad interna.

**Desafíos:**
- **Duplicación de código**: sin un mecanismo claro de librerías compartidas, se tiende a repetir lógica.
- **Sincronización difícil**: actualizar una librería común implica publicar una nueva versión y actualizar manualmente en cada repo.
- **Visibilidad fragmentada**: es más difícil detectar inconsistencias o aplicar refactors globales.

### Estrategias híbridas en Enterprise

En la práctica, muchas organizaciones adoptan un modelo **híbrido**:

- **Monorepo para el frontend**: todas las apps Angular y librerías compartidas viven juntas, facilitando la coherencia visual y técnica.
- **Multirepo para backend y servicios externos**: cada microservicio tiene su propio ciclo de vida y despliegue independiente.
- **Librerías internas publicadas en un registry privado**: incluso dentro de un monorepo, algunas librerías se versionan y publican para garantizar estabilidad y control de dependencias.

Este enfoque equilibra la **velocidad de desarrollo** con la **gobernanza organizativa**.

### Factores de decisión en entornos Enterprise

Al elegir entre monorepo y multirepo, conviene analizar:

- **Tamaño del equipo**: equipos pequeños suelen beneficiarse de monorepo; equipos muy grandes y distribuidos pueden preferir multirepo.
- **Nivel de autonomía**: si cada equipo necesita releases independientes, multirepo puede ser más natural.
- **Necesidad de consistencia**: si la prioridad es mantener un diseño y arquitectura unificados, el monorepo es más adecuado.
- **Infraestructura de CI/CD**: un monorepo requiere pipelines más sofisticados (caching, ejecución paralela).
- **Cultura organizativa**: empresas con fuerte centralización tienden al monorepo; las más descentralizadas, al multirepo.

### Buenas prácticas

- En **monorepos**: usar herramientas como Nx para detectar qué proyectos se ven afectados por un cambio y ejecutar solo los tests/builds necesarios.  
- En **multirepos**: establecer un registry privado (por ejemplo, con npm o GitHub Packages) para publicar librerías compartidas y evitar duplicación.  
- En ambos casos: definir convenciones claras de versionado, testing y documentación para que los equipos trabajen de forma coherente.  

Por lo tanto, no existe una respuesta única: **el monorepo ofrece coherencia y velocidad de refactor, mientras que el multirepo ofrece autonomía y aislamiento**.  
En entornos Enterprise, lo más habitual es encontrar un **modelo híbrido**, donde el frontend se beneficia de la cohesión del monorepo y el backend mantiene la independencia del multirepo.  
Lo importante no es la etiqueta, sino que la estrategia elegida **responda a la escala, cultura y objetivos de la organización**.


## 11.5. Introducción a Microfrontends y cuándo aplicarlos

En los últimos años, el desarrollo frontend ha experimentado un crecimiento similar al que vivieron los backends con los **microservicios**.  
Las aplicaciones web han pasado de ser simples interfaces a convertirse en **plataformas complejas**, con múltiples equipos trabajando en paralelo, ciclos de despliegue independientes y necesidades de escalabilidad organizativa.  
En este contexto surge el concepto de **Microfrontends**: una forma de dividir una aplicación grande en **fragmentos autónomos**, desarrollados y desplegados de manera independiente, pero que se integran en una experiencia de usuario unificada.

### ¿Qué son los Microfrontends?

Un **Microfrontend** es, en esencia, una **aplicación frontend independiente** que representa una parte funcional de la interfaz global.  
Cada microfrontend puede estar desarrollado con su propio stack (aunque en entornos Enterprise suele estandarizarse en Angular, React o Vue), tener su propio ciclo de vida y ser desplegado de forma autónoma.

La idea es que, al igual que los microservicios en backend, cada equipo sea **dueño de un dominio funcional completo**: desde la base de datos hasta la interfaz de usuario que lo expone.

Ejemplo:  
- Un equipo desarrolla el módulo de **catálogo de productos**.  
- Otro se encarga del **checkout**.  
- Otro mantiene el **perfil de usuario**.  
Cada uno despliega su microfrontend, y el “shell” de la aplicación los integra en tiempo de ejecución.

### Motivaciones principales

¿Por qué optar por Microfrontends en lugar de un monolito frontend tradicional?

- **Escalabilidad organizativa**: equipos grandes pueden trabajar en paralelo sin bloquearse.  
- **Autonomía de despliegue**: cada microfrontend se publica de forma independiente, sin esperar a que toda la app esté lista.  
- **Evolución tecnológica**: permite migrar partes de la aplicación a nuevas versiones o frameworks sin reescribir todo.  
- **Aislamiento de fallos**: un error en un microfrontend no necesariamente tumba toda la aplicación.  
- **Reutilización**: un mismo microfrontend puede integrarse en varias aplicaciones (ej. un módulo de login común).

### Estrategias de integración

Existen varias formas de integrar microfrontends en Angular:

- **Module Federation (Webpack 5)**: muy popular en Angular 13+, permite cargar módulos remotos en tiempo de ejecución.  
- **iframes**: más simple, pero con limitaciones de comunicación y estilos.  
- **Build-time integration**: compilar varios proyectos en uno solo, perdiendo parte de la independencia.  
- **Single-SPA**: framework que orquesta microfrontends de distintos frameworks.  

En entornos Enterprise, **Module Federation** se ha convertido en el estándar de facto, ya que permite mantener la experiencia de usuario fluida y compartir dependencias comunes (Angular, librerías de UI, etc.).

### Ejemplo práctico: Microfrontends con Angular y Module Federation

Imaginemos una plataforma de e‑commerce dividida en tres dominios funcionales:

- **Shell (o host)**: la aplicación principal que orquesta la navegación y carga los microfrontends.  
- **Catálogo (remote)**: microfrontend encargado de mostrar productos.  
- **Checkout (remote)**: microfrontend que gestiona el proceso de compra.  

Cada microfrontend es una aplicación Angular independiente, con su propio ciclo de build y despliegue, pero integrados en tiempo de ejecución gracias a **Webpack Module Federation**.

#### Configuración del Shell (host)

```js
// webpack.config.js del shell
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        catalog: 'catalog@http://localhost:4201/remoteEntry.js',
        checkout: 'checkout@http://localhost:4202/remoteEntry.js'
      },
      shared: {
        '@angular/core': { singleton: true, strictVersion: true },
        '@angular/common': { singleton: true, strictVersion: true },
        '@angular/router': { singleton: true, strictVersion: true }
      }
    })
  ]
};
```

#### Configuración de un Remote (Catalog)

```js
// webpack.config.js del microfrontend catalog
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'catalog',
      filename: 'remoteEntry.js',
      exposes: {
        './Module': './src/app/catalog/catalog.module.ts'
      },
      shared: {
        '@angular/core': { singleton: true, strictVersion: true },
        '@angular/common': { singleton: true, strictVersion: true },
        '@angular/router': { singleton: true, strictVersion: true }
      }
    })
  ]
};
```

#### Integración en el Shell con Routing

```ts
// app-routing.module.ts del shell
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'catalog',
    loadChildren: () =>
      import('catalog/Module').then(m => m.CatalogModule)
  },
  {
    path: 'checkout',
    loadChildren: () =>
      import('checkout/Module').then(m => m.CheckoutModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

#### Comunicación entre Microfrontends

Aunque cada microfrontend es autónomo, a menudo necesitan comunicarse.  
Algunas estrategias comunes:

- **Servicios compartidos**: expuestos desde una librería común dentro del monorepo, preferentemente como servicios standalone y usando signals para estado.
- **Eventos globales**: usando RxJS Subjects o Signals compartidos.
- **State management centralizado**: un store común (NgRx, Signals Store) consumido por host y remotes.

Ejemplo simple con un servicio compartido:

```ts
import { Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CartService {
  private items = signal<string[]>([]);

  addItem(item: string) {
    this.items.update(list => [...list, item]);
  }

  getItems() {
    return this.items.asReadonly();
  }
}
```

Tanto `catalog` como `checkout` pueden inyectar este servicio y reaccionar a los cambios en el carrito.


### ¿Cuándo aplicarlos?

No todas las aplicaciones necesitan Microfrontends. De hecho, en proyectos pequeños o medianos pueden añadir complejidad innecesaria.  
Son especialmente útiles cuando:

- La aplicación es **muy grande** y debe ser mantenida por **múltiples equipos en paralelo**.  
- Existen **dominios funcionales bien definidos** (catálogo, pagos, usuarios, administración).  
- Se requiere **despliegue independiente** de cada módulo.  
- Hay necesidad de **evolución tecnológica gradual** (ej. migrar de AngularJS a Angular moderno por partes).  
- Se busca **reutilizar módulos** en diferentes aplicaciones (ej. un microfrontend de autenticación usado en varias plataformas internas).  

En cambio, si el equipo es pequeño, el dominio es acotado y los despliegues son centralizados, un monolito frontend bien estructurado suele ser más eficiente.

### Buenas prácticas iniciales

- **Definir boundaries claros**: cada microfrontend debe tener un dominio funcional propio, no dividirse arbitrariamente.  
- **Centralizar diseño y UX**: aunque los equipos sean independientes, la experiencia de usuario debe ser coherente.  
- **Gestionar dependencias compartidas**: evitar duplicar Angular, librerías de UI o utilidades comunes.  
- **Automatizar la integración**: pipelines de CI/CD que publiquen y consuman microfrontends sin intervención manual.  
- **Observar el rendimiento**: demasiados microfrontends mal orquestados pueden degradar la experiencia.  
- **Usar standalone components y signals** en microfrontends y librerías compartidas para aprovechar las ventajas de Angular 20+.


## 11.6. Estrategias de seguridad, despliegue y monitorización en arquitecturas distribuidas

Cuando una aplicación Angular evoluciona hacia una **arquitectura distribuida** —ya sea con **Workspaces multi-app**, **Microfrontends** o integraciones con múltiples servicios—, los retos ya no son solo de modularidad y escalabilidad.  
En entornos Enterprise, la **seguridad**, el **despliegue controlado** y la **monitorización continua** se convierten en pilares imprescindibles para garantizar que la plataforma no solo funciona, sino que lo hace de forma confiable, segura y observable.

### Seguridad en arquitecturas distribuidas

En un sistema compuesto por múltiples aplicaciones y microfrontends, la superficie de ataque aumenta. Algunas estrategias clave:

- **Autenticación y autorización centralizadas**  
  - Usar **OpenID Connect / OAuth2** con un Identity Provider (Keycloak, Auth0, Azure AD).  
  - Delegar la gestión de tokens y roles a un servicio común, consumido por todos los microfrontends.  

- **Comunicación segura entre microfrontends**  
  - Transmitir datos sensibles solo mediante **servicios compartidos** o **eventos cifrados**, nunca por `localStorage` sin protección.  
  - Usar **JWT firmados** y con expiración corta.  

- **Control de dependencias**  
  - Escanear librerías NPM con herramientas como **Snyk** o **npm audit**.  
  - Evitar dependencias duplicadas en Module Federation (riesgo de versiones vulnerables).  

- **Políticas de Content Security Policy (CSP)**  
  - Restringir orígenes de scripts y estilos.  
  - Evitar inyecciones de código en microfrontends cargados dinámicamente.  

> ⚠️ **Advertencia SSR/Microfrontends:** Evita acceder directamente a APIs del navegador (`window`, `document`, `navigator`) en servicios o componentes que puedan ejecutarse en SSR o microfrontends. Usa comprobaciones de entorno (`isPlatformBrowser`) para evitar errores y facilitar la integración y seguridad.

### Estrategias de despliegue

El despliegue en arquitecturas distribuidas debe equilibrar **autonomía de equipos** con **coherencia de plataforma**.

- **Pipelines CI/CD por microfrontend**  
  - Cada equipo despliega su remote de forma independiente.  
  - El shell consume la última versión estable publicada.  

- **Versionado controlado**  
  - Uso de **semver interno** para microfrontends y librerías compartidas.  
  - Estrategias de *rollback* rápidas en caso de fallo.  

- **Despliegues progresivos**  
  - **Canary releases**: liberar un microfrontend a un subconjunto de usuarios antes del despliegue global.  
  - **Feature flags**: activar/desactivar funcionalidades sin necesidad de redeploy.  

- **Infraestructura como código**  
  - Definir entornos (dev, staging, prod) con Terraform, Pulumi o ARM templates.  
  - Automatizar la creación de pipelines y entornos de prueba efímeros.  

### Monitorización y observabilidad

En arquitecturas distribuidas, el reto no es solo desplegar, sino **entender qué ocurre en producción**.

- **Logging centralizado**  
  - Enviar logs de cada microfrontend a un sistema común (Elastic Stack, Loki, Azure Monitor).  
  - Incluir *correlation IDs* para rastrear una petición a través de múltiples módulos.  

- **Métricas de rendimiento**  
  - Medir **LCP, FID, CLS** (Core Web Vitals) por microfrontend.  
  - Integrar con **Prometheus + Grafana** o servicios cloud (Azure Application Insights).  

- **Alertas proactivas**  
  - Configurar umbrales de error, tiempo de respuesta y consumo de recursos.  
  - Notificar en Slack/Teams cuando un microfrontend supera los límites.  

- **User Experience Monitoring (RUM)**  
  - Capturar métricas reales de usuarios (tiempo de carga, errores JS, fallos de red).  
  - Herramientas como Datadog, New Relic o Sentry ayudan a correlacionar errores con releases.  

### Buenas prácticas finales

- **Zero Trust**: cada microfrontend y servicio debe autenticarse, incluso dentro de la misma red.  
- **Automatización total**: desde el build hasta el despliegue y rollback.  
- **Observabilidad desde el diseño**: instrumentar la aplicación con métricas y trazas desde el inicio.  
- **Gobernanza clara**: definir qué equipo es responsable de cada microfrontend, librería o servicio.  
- **Pruebas en producción controladas**: usar *dark launches* y *A/B testing* para validar nuevas funcionalidades.

