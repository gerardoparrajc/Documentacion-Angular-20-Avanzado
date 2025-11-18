# 16. Buenas prácticas y mantenimiento en Angular 20

## 16.1. Uso de herramientas de IA (Copilot, Cursor) para generación de código y documentación

En el ecosistema actual de desarrollo, la **inteligencia artificial** se ha convertido en un aliado estratégico para los equipos de software. Herramientas como **Copilot** y **Cursor** permiten acelerar la escritura de código, mejorar la calidad de la documentación y mantener proyectos complejos como los de **Angular 20** de forma más eficiente.  
Lejos de sustituir al desarrollador, estas herramientas actúan como un **asistente inteligente**, capaz de sugerir soluciones, detectar patrones y proponer mejoras en tiempo real.

### 16.1.1. Generación de código asistida y buenas prácticas modernas

- **Sugerencias contextuales**: al escribir un componente, servicio o directiva, la IA puede autocompletar métodos, estructuras de datos o patrones comunes de Angular (ej. `signals`, `inject`, `standalone components`).  
- **Plantillas repetitivas**: creación rápida de formularios reactivos, interceptores HTTP, guards de rutas o configuraciones de SSR.  
- **Buenas prácticas integradas**: las sugerencias suelen estar alineadas con las convenciones modernas de Angular 20, evitando errores comunes.  

Ejemplo: al comenzar a escribir un formulario de login, la IA puede sugerir automáticamente la estructura completa usando signals y validaciones tipadas.

### 16.1.2. Documentación y comentarios

- **Generación automática de JSDoc**: a partir de la firma de un método, la IA puede proponer descripciones, parámetros y valores de retorno.  
- **Explicaciones pedagógicas**: útil en proyectos educativos o de onboarding, donde cada sección del código necesita contexto narrativo.  
- **Traducción y adaptación**: la IA puede generar documentación en varios idiomas, facilitando la colaboración en equipos internacionales.  

Ejemplo: al escribir un servicio de autenticación, la IA puede generar un bloque de documentación que explique el flujo de login, refresh token y logout.

### 16.1.3. Refactorización y mantenimiento

- **Detección de redundancias**: sugerencias para extraer lógica repetida en servicios compartidos.  
- **Migraciones de versión**: ayuda a adaptar código a nuevas APIs de Angular 20 (ej. migrar de `NgModules` a standalone components y signals).  
- **Optimización de rendimiento**: recomendaciones sobre lazy loading, signals, SSR/hydration y uso de `ChangeDetectionStrategy.OnPush`.  

### 16.1.4. Integración en el flujo de trabajo

- **Copilot**: integrado en editores como VS Code, ofrece sugerencias en tiempo real mientras se escribe.  
- **Cursor**: un editor potenciado por IA que permite conversaciones contextuales sobre el código, ideal para refactorizaciones profundas o generación de documentación narrativa.  
- **Complementariedad**: Copilot es excelente para productividad inmediata, mientras que Cursor destaca en exploración y documentación más extensa.  

### 16.1.5. Buenas prácticas al usar IA en proyectos Angular

- **Validar siempre las sugerencias**: la IA acelera, pero no sustituye el criterio del desarrollador.  
- **Usar la IA como apoyo pedagógico**: especialmente útil para explicar patrones avanzados a nuevos miembros del equipo.  
- **Mantener consistencia**: revisar que las sugerencias respeten el estilo de código y convenciones del proyecto.  
- **Combinar con testing**: toda generación automática debe estar respaldada por pruebas unitarias (Jest) e integradas (Cypress/Playwright).  

## 16.2. Seguridad en Angular: protección contra XSS, CSRF y vulnerabilidades comunes

Cuando aprendemos Angular 20, solemos centrarnos en la arquitectura, los componentes y la experiencia de usuario. Pero hay un aspecto que nunca podemos descuidar: **la seguridad**.  
Una aplicación insegura no solo falla técnicamente: puede exponer datos sensibles, comprometer la confianza de los usuarios y generar consecuencias legales y económicas.  

Angular nos ofrece una base sólida de seguridad, pero como desarrolladores debemos **entender los riesgos más comunes** y aplicar las medidas adecuadas. Hoy vamos a repasar tres grandes bloques: **XSS, CSRF y otras vulnerabilidades frecuentes**.

### 16.2.1. Protección contra XSS (Cross-Site Scripting)

#### ¿Qué es XSS?
El **XSS** ocurre cuando un atacante consigue inyectar código malicioso (normalmente JavaScript) en nuestra aplicación. Ese código se ejecuta en el navegador de la víctima, pudiendo robar cookies, tokens de sesión o manipular la interfaz.

Ejemplo clásico:  
Un campo de comentarios que permite introducir HTML sin validación.  
Si alguien escribe:

```html
<script>alert('Hackeado');</script>
```

y la aplicación lo muestra sin sanitizar, ese script se ejecutará en el navegador de todos los usuarios que vean el comentario.

#### Cómo protege Angular
- **Escapado automático**: Angular escapa por defecto las expresiones interpoladas en plantillas (`{{ valor }}`), evitando que se ejecuten como código.  
- **Sanitización de HTML**: si usamos `[innerHTML]`, Angular aplica un proceso de sanitización que elimina etiquetas peligrosas como `<script>`.  
- **DomSanitizer**: Angular ofrece este servicio para casos donde necesitamos mostrar HTML seguro, pero debemos usarlo con extrema precaución.

#### Buenas prácticas contra XSS
- **Evita `[innerHTML]` siempre que sea posible**. Prefiere bindings normales.  
- **Nunca uses `bypassSecurityTrust...` con datos de usuario o dinámicos**. Solo en casos muy controlados (ej. contenido estático de confianza).  
- **Valida y limpia datos en el backend**: Angular protege en el cliente, pero el servidor también debe filtrar entradas y validar tokens/permisos.  
- **Configura Content Security Policy (CSP)**: limita qué scripts pueden ejecutarse en tu aplicación. Usa librerías como `helmet` en el backend para reforzar cabeceras HTTP.  

### 16.2.2. Protección contra CSRF (Cross-Site Request Forgery)

#### ¿Qué es CSRF?
El **CSRF** ocurre cuando un atacante engaña al navegador de un usuario autenticado para que ejecute acciones en tu aplicación sin su consentimiento.  
Ejemplo: un usuario está logueado en su banco, y visita una página maliciosa que envía un `POST` oculto para transferir dinero.

#### Cómo protege Angular
Angular incluye soporte nativo para **XSRF/CSRF tokens** en `HttpClient`:
- Busca automáticamente una cookie llamada `XSRF-TOKEN`.  
- Envía su valor en cada petición como cabecera `X-XSRF-TOKEN`.  
- El servidor valida que el token recibido coincide con el de la cookie.  

#### Ejemplo de configuración

```ts
import { HttpClientXsrfModule } from '@angular/common/http';

bootstrapApplication(AppComponent, {
  providers: [
    importProvidersFrom(HttpClientXsrfModule.withOptions({
      cookieName: 'XSRF-TOKEN',
      headerName: 'X-XSRF-TOKEN'
    }))
  ]
});
```

#### Buenas prácticas contra CSRF
- **Configura el backend** para emitir y validar tokens CSRF.  
- **Usa cookies seguras** (`HttpOnly`, `SameSite=Strict`) para reducir riesgos.  
- **No confíes solo en Angular**: la validación final siempre debe estar en el servidor y debe validar tokens y permisos en cada petición.

### 16.2.3. Otras vulnerabilidades comunes

#### a) Inyección de dependencias inseguras
- No expongas servicios críticos en el `window` global.  
- Usa `inject()` y `providedIn: 'root'` para un control seguro.  

#### b) Exposición de datos sensibles
- Evita guardar tokens en `localStorage` o `sessionStorage`.  
- Prefiere **cookies seguras con HttpOnly** para credenciales.  

#### c) Clickjacking
- Configura cabeceras HTTP como:  
  - `X-Frame-Options: DENY`  
  - `Content-Security-Policy: frame-ancestors 'none'`  

#### d) Content Security Policy (CSP)
- Define una política estricta para controlar qué scripts, estilos e iframes pueden cargarse.  
- Ejemplo:  
  ```http
  Content-Security-Policy: default-src 'self'; script-src 'self'
  ```

#### e) Dependencias inseguras
- Mantén Angular y librerías actualizadas.  
- Usa `npm audit` y `npm outdated` regularmente.  

### 16.2.4. Herramientas de apoyo

- **Angular DevTools**: detecta malas prácticas en componentes.  
- **npm audit**: revisa vulnerabilidades en dependencias.  
- **OWASP ZAP / Burp Suite**: escáneres de seguridad para pruebas de penetración.  
- **ESLint con reglas de seguridad**: ayuda a detectar patrones inseguros en el código.  

### 16.2.5. Estrategia de seguridad en proyectos Enterprise

En proyectos grandes, la seguridad debe ser **un proceso continuo**:
- **Integrar auditorías automáticas** en la pipeline de CI/CD.  
- **Formar al equipo** en OWASP Top 10 y buenas prácticas de Angular.  
- **Revisar periódicamente** la configuración de CSP, cabeceras y almacenamiento de credenciales.  
- **Testing de seguridad**: incluir pruebas unitarias y E2E que validen que el cambio de idioma, la carga de datos y la navegación no introducen vulnerabilidades.  


## 16.3. Mejores prácticas de autenticación y autorización (JWT, OAuth2)

En cualquier aplicación moderna, la **autenticación** (saber quién eres) y la **autorización** (qué puedes hacer) son pilares fundamentales.  
En Angular 20, solemos trabajar con **JWT (JSON Web Tokens)** para sesiones basadas en tokens y con **OAuth2/OpenID Connect** cuando necesitamos integrarnos con proveedores externos (Google, Microsoft, GitHub, etc.).  
El reto no es solo implementar estas tecnologías, sino hacerlo de forma **segura, escalable y mantenible**.

### 16.3.1. Autenticación con JWT

#### ¿Qué es un JWT?
Un **JSON Web Token** es un objeto firmado digitalmente que contiene información sobre el usuario (claims).  
Ejemplo de payload:

```json
{
  "sub": "1234567890",
  "name": "Gerardo",
  "role": "admin",
  "exp": 1733427200
}
```

- `sub`: identificador del usuario.  
- `role`: rol o permisos.  
- `exp`: fecha de expiración.  

El servidor emite este token tras un login exitoso, y el cliente lo envía en cada petición.

#### Buenas prácticas con JWT en Angular
- **Almacenamiento seguro**:  
  - Evita `localStorage` o `sessionStorage` para tokens sensibles.  
  - Prefiere **cookies seguras con HttpOnly y SameSite=Strict**.  
- **Expiración corta**: los tokens deben expirar rápido (ej. 15 minutos).  
- **Refresh tokens**: usa un token de refresco (más largo) para obtener nuevos JWT sin pedir credenciales otra vez.  
- **Interceptors en Angular**: añade automáticamente el token a cada petición HTTP.  

Ejemplo de interceptor:

```ts
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  // Preferir cookies HttpOnly para tokens en producción
  // Solo usar localStorage en entornos no críticos
  const token = /* obtener token de cookie segura */;
  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }
  return next(req);
};
```

### 16.3.2. Autorización basada en roles y claims

Una vez autenticado el usuario, debemos controlar **qué puede hacer**:

- **Guardas de rutas** (`CanActivate`): restringen acceso a rutas según rol.  
- **Directivas personalizadas**: mostrar/ocultar elementos en la UI según permisos.  
- **Claims en el token**: usar `role`, `permissions` o `scopes` para decidir accesos.  

Ejemplo de guard:

```ts
import { CanActivateFn } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const adminGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  return auth.hasRole('admin');
};
```

### 16.3.3. OAuth2 y OpenID Connect

Cuando necesitamos integrar login con terceros (Google, Microsoft, GitHub), usamos **OAuth2** (autorización delegada) y **OpenID Connect** (capa de identidad sobre OAuth2).

#### Flujo típico
1. El usuario hace clic en “Iniciar sesión con Google”.  
2. Angular redirige al proveedor (Google).  
3. El usuario se autentica allí.  
4. El proveedor devuelve un **authorization code**.  
5. El backend intercambia ese código por un **access token** y un **ID token** (JWT con identidad del usuario).  
6. Angular recibe el token y lo usa en sus peticiones.

#### Buenas prácticas con OAuth2/OIDC
- **Usar librerías probadas**: por ejemplo, `angular-oauth2-oidc`.  
- **PKCE (Proof Key for Code Exchange)**: obligatorio para SPAs modernas, evita ataques de interceptación de tokens.  
- **Scopes mínimos**: pedir solo los permisos estrictamente necesarios.  
- **Cerrar sesión correctamente**: invalidar tokens en el servidor y limpiar estado en el cliente.  

### 16.3.4. Estrategias Enterprise

En proyectos grandes:
- **Centralizar la lógica de autenticación** en un `AuthService`.  
- **Separar autenticación de autorización**: primero saber quién eres, luego qué puedes hacer.  
- **Integrar con Identity Providers (IdPs)**: Azure AD, Keycloak, Auth0, Okta.  
- **Auditoría y logging**: registrar accesos y fallos de autenticación.  
- **Testing de seguridad**: pruebas unitarias y E2E que validen accesos restringidos.  

### 16.3.5. Errores comunes a evitar

- Guardar tokens en `localStorage` sin cifrado.  
- No validar la expiración del token en el cliente.  
- No refrescar tokens automáticamente.  
- Confiar solo en la validación del frontend (el backend siempre debe validar).  
- Pedir más scopes de los necesarios en OAuth2.  


## 16.4. Accesibilidad (a11y): uso de atributos ARIA y pruebas automatizadas con axe-core

La **accesibilidad (a11y)** no es un añadido opcional: es un requisito fundamental para garantizar que nuestras aplicaciones Angular 20 puedan ser utilizadas por todas las personas, incluidas aquellas con discapacidades visuales, auditivas, motoras o cognitivas.  
En este apartado veremos cómo usar correctamente los **atributos ARIA** para enriquecer la semántica de la interfaz y cómo integrar **axe-core** para realizar pruebas automatizadas de accesibilidad.

### 16.4.1. ¿Por qué accesibilidad en Angular?

- **Inclusión**: garantizar que cualquier persona pueda interactuar con la aplicación.  
- **Cumplimiento legal**: normativas como WCAG 2.1 o EN 301 549 en Europa.  
- **Mejor experiencia de usuario**: accesibilidad también significa usabilidad.  
- **SEO**: muchas prácticas de accesibilidad (semántica, etiquetas) mejoran la indexación.  

### 16.4.2. Uso de atributos ARIA en Angular

Los **atributos ARIA (Accessible Rich Internet Applications)** complementan el HTML semántico para describir mejor la interfaz a los lectores de pantalla.

#### Ejemplos prácticos

##### a) Roles
```html
<nav role="navigation" aria-label="Menú principal">
  <a routerLink="/home">Inicio</a>
  <a routerLink="/about">Acerca de</a>
</nav>
```

- `role="navigation"` indica que es un bloque de navegación.  
- `aria-label` da un nombre descriptivo al menú.

##### b) Estados y propiedades
```html
<button aria-expanded="false" aria-controls="menuOpciones">
  Opciones
</button>
<ul id="menuOpciones" hidden>
  <li><a href="#">Perfil</a></li>
  <li><a href="#">Configuración</a></li>
</ul>
```

- `aria-expanded` comunica si el menú está abierto o cerrado.  
- `aria-controls` vincula el botón con el elemento controlado.

##### c) Alertas y notificaciones
```html
<div role="alert">
  ¡Tu sesión está a punto de expirar!
</div>
```

- `role="alert"` asegura que el lector de pantalla anuncie el mensaje inmediatamente.

### 16.4.3. Buenas prácticas con ARIA

- **Usar primero HTML semántico** (`<button>`, `<nav>`, `<header>`) antes de recurrir a ARIA.  
- **No duplicar roles**: si un elemento ya tiene semántica nativa, no añadas un `role` redundante.  
- **Mantener sincronizados los estados** (`aria-expanded`, `aria-checked`) con el estado real del componente (usa signals para sincronización reactiva).  
- **Etiquetas descriptivas**: usar `aria-label` o `aria-labelledby` para dar contexto.  
- **Angular DevTools**: puede ayudar a detectar problemas de accesibilidad en componentes.

### 16.4.4. Pruebas automatizadas con axe-core

#### ¿Qué es axe-core?
Es una librería de **testing de accesibilidad** desarrollada por Deque Systems. Permite analizar automáticamente la aplicación y detectar problemas comunes de accesibilidad.

#### Integración en Angular
Podemos usar axe-core en pruebas unitarias o E2E.

##### Ejemplo con Jasmine/Karma
```ts
import 'axe-core';

it('la página debería ser accesible', async () => {
  const results = await (window as any).axe.run();
  expect(results.violations.length).toBe(0);
});
```

##### Ejemplo con Cypress
```js
import 'cypress-axe';

describe('Accesibilidad', () => {
  it('no debería tener violaciones de a11y', () => {
    cy.visit('/');
    cy.injectAxe();
    cy.checkA11y();
  });
});
```

Ejemplo de integración en CI/CD:

```yaml
- name: Pruebas de accesibilidad
  run: npm run test:a11y
```

### 16.4.5. Estrategia Enterprise de accesibilidad
- **Integrar axe-core en CI/CD**: cada build debe ejecutar pruebas de accesibilidad y auditoría de seguridad.  
- **Auditorías manuales**: combinar pruebas automáticas con revisiones manuales usando lectores de pantalla (NVDA, VoiceOver).  
- **Formación del equipo**: todos los desarrolladores deben conocer las pautas WCAG y el proceso de onboarding debe incluir accesibilidad y automatización.  
- **Diseño inclusivo desde el inicio**: no “parchear” accesibilidad al final, sino integrarla en el diseño y desarrollo.  

## 16.5. Estrategias de refactorización segura en aplicaciones grandes

En proyectos pequeños, refactorizar suele ser un proceso rápido: renombrar un componente, mover un servicio, reorganizar un módulo.  
Pero en **aplicaciones grandes y de larga vida**, la refactorización se convierte en un reto: múltiples equipos trabajando en paralelo, dependencias cruzadas, librerías compartidas y un alto riesgo de romper funcionalidades críticas.  
Por eso, en Angular 20 debemos aplicar **estrategias de refactorización segura**, que nos permitan mejorar el código sin comprometer la estabilidad del sistema.

### 16.5.1. Principios básicos de una refactorización segura

- **Cambiar sin romper**: el objetivo no es añadir nuevas features, sino mejorar la calidad interna del código manteniendo el mismo comportamiento externo.  
- **Iterativa y controlada**: refactorizar en pasos pequeños, no en grandes saltos.  
- **Respaldada por pruebas**: sin una buena base de tests, la refactorización es un salto al vacío.  
- **Documentada y comunicada**: en proyectos Enterprise, otros equipos deben entender qué se ha cambiado y por qué.  

### 16.5.2. Preparación antes de refactorizar

- **Auditoría de dependencias**: identificar módulos, servicios y componentes más acoplados.  
- **Cobertura de tests**: asegurar que las áreas críticas tienen pruebas unitarias e integradas.  
- **Feature flags**: activar cambios de forma progresiva en producción. Se pueden gestionar con librerías como `@ngxs-labs/feature-flags` o soluciones personalizadas usando signals.
- **Planificación por fases**: dividir la refactorización en entregas pequeñas y medibles.  

### 16.5.3. Estrategias prácticas en Angular 20

#### a) Migración progresiva a standalone
- No intentes eliminar todos los `NgModules` de golpe.  
- Convierte primero módulos aislados (ej. `SharedModule`) y valida que funcionan.  
- Usa `importProvidersFrom` para compatibilidad temporal.  

#### b) Refactorización de servicios
- Centraliza lógica duplicada en servicios compartidos.  
- Usa `inject()` en lugar de `constructor` para mayor flexibilidad.  
- Aplica **interfaces** para abstraer contratos y facilitar mocks en testing.  

#### c) Limpieza de estado y NGRX
- Elimina reducers y efectos no usados.  
- Divide el store en slices más pequeños y modulares.  
- Usa **Signals** para estados locales y reserva NGRX para estados globales.  

#### d) Refactorización de rutas
- Introduce **lazy loading** para módulos pesados.  
- Añade prefijos de idioma (`/es`, `/en`) si la aplicación es multilingüe.  
- Usa guards y resolvers para separar lógica de negocio de la configuración de rutas.  

### 16.5.4. Herramientas que ayudan

- **Angular DevTools**: detectar ciclos de cambio innecesarios y componentes pesados.  
- **ESLint + reglas personalizadas**: identificar patrones obsoletos o inseguros.  
- **Schematics**: automatizar migraciones repetitivas (ej. actualizar imports).  
- **nx / monorepos**: facilitan refactorizaciones en proyectos con múltiples librerías internas.  

### 16.5.5. Testing como red de seguridad

**Unit tests (Jest)**: aseguran que cada pieza sigue funcionando tras el cambio.  
**Integration tests**: validan la interacción entre componentes y servicios.  
**E2E tests (Cypress/Playwright)**: garantizan que los flujos críticos del usuario no se rompen.  
**Pruebas de regresión visual**: detectar cambios inesperados en la UI.  

Ejemplo: antes de refactorizar un `AuthService`, escribe tests que validen login, logout y refresh de tokens. Así, cualquier error se detectará inmediatamente.

## 16.6. Versionado semántico y gestión de dependencias con NPM y Renovate

En proyectos **Enterprise con Angular 20**, la gestión de dependencias es tan importante como la escritura del propio código. Una librería desactualizada puede introducir vulnerabilidades, y una actualización mal gestionada puede romper la aplicación en producción.  
Por eso, necesitamos dos pilares: **versionado semántico (SemVer)** para entender los cambios en las librerías, y **herramientas de automatización como Renovate** para mantener las dependencias al día de forma segura.

### 16.6.1. Versionado semántico (SemVer)

El **versionado semántico** sigue la convención `MAJOR.MINOR.PATCH`:

- **MAJOR**: cambios incompatibles (breaking changes).  
- **MINOR**: nuevas funcionalidades compatibles hacia atrás.  
- **PATCH**: correcciones de errores y parches de seguridad.  

Ejemplo:  
- `15.2.3` → Angular 15, segunda versión menor, tercer parche.  
- Si pasamos de `15.x` a `16.0.0`, debemos esperar cambios incompatibles.  

#### Buenas prácticas con SemVer
- **Respetar rangos en `package.json`**:  
  - `^15.2.3` → acepta actualizaciones menores y parches (`15.x.x`).  
  - `~15.2.3` → solo acepta parches (`15.2.x`).  
- **Leer changelogs**: antes de actualizar, revisar qué cambios introduce la librería.  
- **Bloquear versiones críticas** en entornos de producción (`package-lock.json` o `npm ci`).  

### 16.6.2. Gestión de dependencias con NPM

#### Comandos clave
- `npm outdated` → muestra qué dependencias están desactualizadas.  
- `npm audit` → detecta vulnerabilidades conocidas.  
- `npm ci` → instala dependencias exactamente como en `package-lock.json`.  

#### Buenas prácticas
- **Usar `npm ci` en CI/CD** para garantizar builds reproducibles.  
- **Actualizar regularmente** dependencias menores y parches.  
- **Evitar dependencias innecesarias**: cada paquete extra es un riesgo potencial.  

### 16.6.3. Automatización con Renovate

**Renovate** es una herramienta que automatiza la actualización de dependencias en proyectos NPM.  
Funciona creando **pull requests automáticas** cuando detecta nuevas versiones de librerías.

#### Ventajas
- **Automatización continua**: no dependemos de revisiones manuales.  
- **Configuración flexible**: podemos decidir qué paquetes actualizar automáticamente y cuáles requieren revisión manual.  
- **Agrupación de actualizaciones**: Renovate puede combinar varias dependencias en un solo PR.  
- **Compatibilidad con SemVer**: entiende los rangos de versión y respeta breaking changes.  

### 16.6.4. Ejemplo de configuración de Renovate

Archivo `renovate.json` (añade reglas para dependencias críticas como NGRX, Angular DevTools y librerías de seguridad):

```json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchPackagePatterns": ["^@angular/"],
      "groupName": "Angular framework",
      "schedule": ["before 6am on monday"]
    },
    {
      "matchPackagePatterns": ["rxjs"],
      "groupName": "RxJS updates",
      "automerge": true
    },
    {
      "matchPackagePatterns": ["@ngrx/*"],
      "groupName": "NGRX updates",
      "schedule": ["before 6am on tuesday"]
    },
    {
      "matchPackagePatterns": ["angular-devtools"],
      "enabled": true
    },
    {
      "matchPackagePatterns": ["helmet"],
      "enabled": true
    }
  ]
}
```

- Agrupa todas las actualizaciones de Angular en un solo PR semanal.  
- Permite que RxJS se actualice automáticamente en parches y versiones menores.  
- Agrupa actualizaciones de NGRX en un PR separado.  
- Habilita actualizaciones automáticas para Angular DevTools y helmet.

## 16.7. Integración de CI/CD con GitHub Actions y pipelines automatizados

En proyectos modernos de **Angular 20**, no basta con escribir buen código: necesitamos garantizar que cada cambio pase por un proceso de **integración continua (CI)** y que el despliegue a entornos de prueba o producción se realice de forma **automatizada y confiable (CD)**.  
GitHub Actions se ha convertido en una de las herramientas más utilizadas para implementar pipelines de CI/CD, gracias a su integración nativa con GitHub y su flexibilidad para definir flujos de trabajo en YAML.

### 16.7.1. ¿Por qué CI/CD en Angular?

- **Calidad continua**: cada commit se valida automáticamente con tests y linters.  
- **Despliegues confiables**: evitamos errores humanos en la publicación manual.  
- **Feedback rápido**: los desarrolladores reciben alertas inmediatas si algo falla.  
- **Escalabilidad**: en proyectos Enterprise, múltiples equipos pueden trabajar en paralelo sin romper la aplicación.  

### 16.7.2. Configuración básica de GitHub Actions

Los flujos de trabajo se definen en `.github/workflows/ci.yml`.  
Ejemplo de pipeline básico para Angular:

```yaml
name: CI Angular

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del repositorio
        uses: actions/checkout@v3

      - name: Configurar Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Instalar dependencias
        run: npm ci

      - name: Linter
        run: npm run lint

      - name: Tests unitarios
        run: npm run test -- --watch=false --browsers=ChromeHeadless

      - name: Build de producción
        run: npm run build -- --configuration=production
```

Este pipeline asegura que cada commit en `main` o cada PR pase por **linting, testing y build** antes de ser aceptado.

### 16.7.3. Despliegue automatizado (CD)

Podemos añadir un job de despliegue al pipeline.  
Ejemplo: despliegue a **GitHub Pages**:

```yaml
deploy:
  needs: build-and-test
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: 20
    - run: npm ci
    - run: npm run build -- --configuration=production --base-href "/mi-app/"
    - name: Deploy a GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist/mi-app
```

En entornos Enterprise, en lugar de GitHub Pages, se suele desplegar a **servidores cloud** (Azure, AWS, GCP) o a contenedores Docker en Kubernetes.

### 16.7.4. Buenas prácticas en pipelines

- **Separar jobs**: build, test, lint, deploy deben ser pasos independientes.  
- **Cache de dependencias**: usar `actions/cache` para acelerar builds y `actions/setup-node` con versiones LTS.  
- **Variables de entorno y secretos**: nunca exponer claves en el YAML, usar `secrets` de GitHub.  
- **Branch protection**: exigir que el pipeline pase antes de permitir merges a `main`.  
- **Testing E2E en CI**: integrar Cypress o Playwright para validar flujos críticos.  

## 16.8. Monitorización continua y logging en aplicaciones Angular productivas

### 16.8.1. Logging en Angular
El **logging** es el registro de eventos relevantes que ocurren en la aplicación.  
En Angular, podemos apoyarnos en el servicio `LoggerService` propio o en librerías externas para estructurar mejor los logs.

#### Buenas prácticas de logging
**Niveles de log**:  
  - `debug`: información detallada para desarrollo.  
  - `info`: eventos normales (ej. inicio de sesión).  
  - `warn`: situaciones inesperadas pero no críticas.  
  - `error`: fallos que requieren atención inmediata.  
**Logs estructurados**: usar objetos JSON en lugar de cadenas sueltas.  
**Contexto en los logs**: incluir información de usuario, módulo y acción.  
**Evitar datos sensibles**: nunca registrar contraseñas, tokens o información personal. Los logs deben anonimizar datos sensibles para cumplir con GDPR.  

#### Ejemplo de servicio de logging
```ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class LoggerService {
  log(level: 'debug' | 'info' | 'warn' | 'error', message: string, data?: any) {
    const logEntry = {
      level,
      message,
      data,
      timestamp: new Date().toISOString()
    };
    console[level === 'debug' ? 'log' : level](logEntry);
  }
}
```

### 16.8.2. Integración con herramientas externas
En entornos productivos, los logs deben enviarse a sistemas centralizados:

- **Sentry**: captura errores y stack traces en tiempo real.  
- **LogRocket**: combina logging con grabación de sesiones de usuario.  
- **Elastic Stack (ELK)**: centraliza logs en Elasticsearch y permite visualizarlos en Kibana.  
- **Azure Application Insights** o **Google Cloud Logging**: integraciones cloud para métricas y trazas.  

Ejemplo de integración con Sentry:
```ts
import * as Sentry from "@sentry/angular";

Sentry.init({
  dsn: "https://<your-dsn>.ingest.sentry.io/...",
  integrations: [new Sentry.BrowserTracing()],
  tracesSampleRate: 1.0,
});
```

### 16.8.3. Monitorización continua
El logging nos dice **qué pasó**. La monitorización nos dice **cómo está funcionando la aplicación en tiempo real**.

#### Métricas clave a monitorizar
- **Errores no capturados**: excepciones en Angular o en servicios externos.  
- **Rendimiento**: tiempos de carga, latencia de API, FPS en vistas críticas.  
- **Uso de recursos**: memoria, CPU en dispositivos móviles.  
- **Comportamiento de usuario**: rutas más visitadas, abandonos, conversiones.  

#### Herramientas recomendadas
- **Application Performance Monitoring (APM)**: Datadog, New Relic, AppDynamics.  
- **Angular + Web Vitals**: medir métricas como LCP, FID y CLS.  
- **Dashboards en tiempo real**: Kibana, Grafana o Azure Monitor.  

### 16.8.4. Estrategia Enterprise
En proyectos grandes:
- **Centralizar logs y métricas** en un único sistema accesible a todos los equipos.  
- **Alertas automáticas**: configurar notificaciones en Slack, Teams o email cuando se detecten errores críticos.  
- **Correlación de eventos**: vincular logs de frontend (Angular) con logs de backend para diagnósticos completos usando IDs de sesión o usuario.  
- **Pruebas de resiliencia**: simular fallos en producción controlada y validar que los logs/alertas funcionan.  
- **Retención y cumplimiento**: definir cuánto tiempo se guardan los logs y cumplir normativas (ej. GDPR).  

### 16.8.5. Ejemplo de pipeline de monitorización
1. Angular registra un error con `LoggerService`.  
2. El error se envía a Sentry con contexto (usuario, ruta, estado).  
3. Sentry dispara una alerta en Slack.  
4. El equipo revisa el dashboard de Kibana para correlacionar con logs de backend.  
5. Se crea automáticamente un ticket en Jira para seguimiento.  

## 16.9. Checklist de buenas prácticas para proyectos enterprise en Angular 20

En proyectos **Enterprise**, la complejidad técnica y organizativa exige un conjunto de buenas prácticas claras y compartidas por todo el equipo.  
Este checklist resume los puntos clave que hemos trabajado a lo largo del tema y que deben revisarse de forma continua en cualquier aplicación Angular 20.

### 16.9.1. Arquitectura y organización
- [ ] Usar **standalone components** en lugar de `NgModules` siempre que sea posible.  
- [ ] Modularizar la aplicación en **feature modules** o dominios bien definidos.  
- [ ] Centralizar la lógica transversal en servicios (`AuthService`, `LoggerService`, `LanguageService`).  
- [ ] Aplicar **lazy loading** en rutas para mejorar rendimiento.  
- [ ] Documentar decisiones arquitectónicas en un **ADR (Architecture Decision Record)**.  

### 16.9.2. Internacionalización (i18n)
- [ ] Incluir el **idioma en la URL** (`/es`, `/en`, `/fr`) para SEO y claridad.  
- [ ] Usar **ngx-translate** o el sistema nativo de Angular según el caso.  
- [ ] Automatizar la extracción de claves con `ngx-translate-extract`.  
- [ ] Configurar **fallback de idioma** para claves faltantes.  
- [ ] Integrar traducciones con plataformas externas (Crowdin, Lokalise, DeepL).  

### 16.9.3. Seguridad
- [ ] Evitar `innerHTML` y usar `DomSanitizer` solo en casos controlados.  
- [ ] Configurar **CSRF tokens** en Angular y en el backend.  
- [ ] Usar **cookies seguras (HttpOnly, SameSite)** para credenciales.  
- [ ] Configurar **Content Security Policy (CSP)** en el servidor.  
- [ ] Revisar dependencias con `npm audit` y mantener Angular actualizado.  

### 16.9.4. Autenticación y autorización
- [ ] Usar **JWT** con expiración corta y refresh tokens.  
- [ ] Implementar **guards de rutas** (`CanActivate`) para roles y permisos.  
- [ ] Integrar con **OAuth2/OpenID Connect** en caso de login externo.  
- [ ] Centralizar la lógica en un `AuthService`.  
- [ ] Validar siempre los permisos también en el backend.  

### 16.9.5. Accesibilidad (a11y)
- [ ] Usar HTML semántico antes de recurrir a ARIA.  
- [ ] Añadir atributos ARIA (`aria-label`, `aria-expanded`, `role`) cuando sea necesario.  
- [ ] Ejecutar pruebas automáticas con **axe-core** en CI/CD y auditoría de seguridad en cada release.  
- [ ] Validar manualmente con lectores de pantalla (NVDA, VoiceOver).  
- [ ] Incluir accesibilidad en las revisiones de código.  

### 16.9.8. CI/CD y mantenimiento
- [ ] Configurar pipelines en **GitHub Actions** o similar con build, lint, test y deploy.  
- [ ] Usar **Renovate** para mantener dependencias actualizadas y reglas para dependencias críticas.  
- [ ] Integrar auditorías de seguridad (`npm audit`) y accesibilidad en CI/CD.  
- [ ] Configurar **branch protection rules** en repositorios críticos.  
- [ ] Documentar y versionar con **SemVer** (`MAJOR.MINOR.PATCH`).  
- [ ] Documentar el proceso de onboarding para nuevos desarrolladores, incluyendo el uso de IA y automatización.

### 16.9.9. Monitorización y logging
- [ ] Implementar un `LoggerService` con niveles (`debug`, `info`, `warn`, `error`).  
- [ ] Centralizar logs en sistemas como **Sentry**, **Elastic Stack** o **Application Insights**.  
- [ ] Configurar alertas automáticas en caso de errores críticos.  
- [ ] Correlacionar logs de frontend y backend.  
- [ ] Revisar métricas de uso y rendimiento en dashboards (Grafana, Kibana).

