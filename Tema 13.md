# 13. Estado global con NGRX en Angular 20

## 13.1. Introducción a NGRX y su relación con el patrón Redux

A medida que las aplicaciones Angular crecen en complejidad, uno de los retos más frecuentes es **gestionar el estado de forma coherente y predecible**.  
Cuando múltiples componentes necesitan acceder, modificar o reaccionar al mismo conjunto de datos —como el carrito de compras, el usuario autenticado, las preferencias de configuración o los resultados de búsqueda—, el uso de servicios compartidos y `BehaviorSubject`s puede volverse difícil de escalar y mantener.  

Aquí es donde entra **NGRX**: una librería inspirada en el patrón **Redux**, diseñada específicamente para Angular, que permite gestionar el estado global de la aplicación de forma **centralizada, reactiva y trazable**.

### 13.1.1 ¿Qué es Redux y por qué importa?

**Redux** es un patrón de arquitectura de estado que nació en el ecosistema de React, pero que se ha extendido a otros frameworks por su claridad conceptual.  
Su premisa es simple pero poderosa:

- El estado de la aplicación se guarda en un **único árbol de estado global**.  
- Las actualizaciones se realizan mediante **acciones** que describen qué ocurrió.  
- Un conjunto de **reducers** (funciones puras) recibe el estado actual y la acción, y devuelve el nuevo estado.  
- Todo el flujo es **unidireccional**: los componentes no modifican el estado directamente, sino que lo describen mediante acciones.

Este enfoque permite:

- **Predecibilidad**: siempre sabes cómo se transforma el estado.  
- **Debugging avanzado**: puedes registrar cada acción y ver cómo cambia el estado paso a paso.  
- **Escalabilidad**: múltiples equipos pueden trabajar sobre el mismo modelo sin interferencias.  
- **Integración con herramientas**: como el Redux DevTools, que permite inspeccionar el estado en tiempo real.

### 13.1.2. ¿Qué es NGRX?

**NGRX** es la implementación de Redux para Angular, pero con una integración profunda en el ecosistema del framework:

- Usa **RxJS** como motor de reactividad.  
- Se basa en **Actions**, **Reducers**, **Selectors** y **Effects**.  
- Permite que el estado global se almacene en un **Store** centralizado, accesible desde cualquier componente.  
- Ofrece herramientas como `createAction`, `createReducer`, `createSelector` y `createEffect` para escribir código más declarativo y mantenible.  
- Se integra con Angular DevTools y con el ecosistema de Signals en Angular 20.

En otras palabras, NGRX toma los principios de Redux y los adapta a la forma en que Angular piensa y trabaja.

### 13.1.3. ¿Por qué usar NGRX en Angular?

Aunque Angular ofrece múltiples formas de compartir estado (servicios, Subjects, Signals), NGRX se vuelve especialmente útil cuando:

- El estado es **global y compartido** por múltiples módulos o componentes.  
- Se requiere **trazabilidad**: saber qué acción disparó qué cambio.  
- Hay **efectos secundarios** complejos (ej. llamadas HTTP, sincronización con APIs externas).  
- Se necesita **persistencia** del estado entre sesiones o navegación.  
- Se busca una arquitectura **predecible y escalable**, especialmente en equipos grandes.

Ejemplo clásico: una aplicación de e‑commerce donde el carrito, el usuario, los productos y las promociones deben estar sincronizados en múltiples vistas, y donde cada acción (añadir producto, aplicar cupón, iniciar sesión) debe reflejarse en el estado global.

### 13.1.4. Relación entre NGRX y Redux: similitudes y diferencias

| Concepto        | Redux (original)                  | NGRX (Angular)                          |
|----------------|-----------------------------------|----------------------------------------|
| Store          | Árbol de estado global            | Árbol de estado global con RxJS        |
| Actions        | Objetos con `type` y `payload`    | `createAction` con tipado y metadata   |
| Reducers       | Funciones puras                   | `createReducer` con `on()` declarativo |
| Selectors      | Funciones para leer el estado     | `createSelector` con memoización       |
| Middleware     | Para efectos secundarios          | `Effects` con RxJS observables         |
| DevTools       | Redux DevTools                    | Angular DevTools + Redux DevTools      |

La diferencia clave es que NGRX está profundamente integrado con **RxJS**, lo que permite modelar efectos secundarios como flujos reactivos, y que su sintaxis moderna (desde Angular 14+) es mucho más declarativa y legible que en versiones anteriores.

---

NGRX no es obligatorio en Angular, pero cuando se usa con criterio, se convierte en una herramienta poderosa para **organizar el estado, separar responsabilidades y escalar la arquitectura**.  
Su relación con Redux no es solo técnica, sino filosófica: ambos promueven la idea de que el estado debe ser **predecible, trazable y centralizado**, especialmente cuando la aplicación crece en complejidad.

En Angular 20, con la llegada de Signals y mejoras en DevTools, NGRX se adapta aún más al paradigma reactivo moderno, permitiendo construir aplicaciones robustas, mantenibles y fáciles de depurar.


## 13.2. Ventajas y desventajas del uso de NGRX frente a Signals o servicios locales

En Angular 20, el ecosistema de gestión de estado se ha diversificado. Ya no estamos ante una única forma de compartir datos entre componentes, sino ante un abanico de opciones que van desde **servicios locales reactivos**, pasando por **Signals**, hasta llegar a soluciones más estructuradas como **NGRX**.  
Cada enfoque tiene sus fortalezas y sus límites, y elegir uno u otro depende del tipo de aplicación, del equipo de desarrollo y de los requisitos de escalabilidad, trazabilidad y complejidad.

### 13.2.1. ¿Qué ofrecen los servicios locales?

Los **servicios locales** son la forma más directa y tradicional de compartir estado en Angular. Se basan en la inyección de dependencias y en el uso de mecanismos como `BehaviorSubject`, `ReplaySubject`, `RxJS signals` o incluso propiedades simples.

Ventajas:

- Simples de implementar y entender.  
- No requieren librerías externas.  
- Perfectos para casos acotados: formularios, filtros, flags de UI.  
- Fáciles de testear en aislamiento.  

Desventajas:

- Escalabilidad limitada: cuando el estado crece, se vuelve difícil de mantener.  
- Trazabilidad pobre: no hay registro de qué acción modificó el estado.  
- Riesgo de efectos colaterales: múltiples componentes pueden modificar el estado sin control.  
- Difícil de sincronizar entre módulos o rutas.

### 13.2.2. ¿Qué aportan los Signals?

Los **Signals** son una de las grandes novedades de Angular 16+ y se consolidan en Angular 20 como una forma reactiva, declarativa y eficiente de manejar estado local y compartido.

Ventajas:

- Reactividad automática: los componentes se actualizan cuando el signal cambia.  
- Sintaxis clara y moderna: `signal()`, `computed()`, `effect()`.  
- Integración profunda con Angular: sin necesidad de RxJS ni Subjects.  
- Ideal para estado local o compartido entre pocos componentes.  
- Excelente rendimiento: actualizaciones precisas y sin sobrecarga.

Desventajas:

- No hay trazabilidad de acciones: no se sabe quién modificó el signal ni por qué.  
- No hay separación entre intención y efecto: el cambio de estado ocurre directamente.  
- No hay arquitectura de efectos secundarios: las llamadas HTTP o sincronizaciones deben manejarse manualmente.  
- No hay DevTools ni herramientas de inspección avanzadas (aunque están en desarrollo).

### 13.2.3. ¿Qué aporta NGRX?

**NGRX** es una solución estructurada para manejar estado global, inspirada en Redux, con integración completa en Angular.

Ventajas:

- Estado centralizado y predecible.  
- Trazabilidad completa: cada cambio de estado está asociado a una acción.  
- Separación clara de responsabilidades: acciones, reducers, selectors, effects.  
- Ideal para aplicaciones grandes, con múltiples módulos y equipos.  
- Integración con DevTools: inspección en tiempo real del estado y del flujo de acciones.  
- Efectos secundarios modelados como flujos reactivos (`createEffect`), lo que facilita la sincronización con APIs.

Desventajas:

- Curva de aprendizaje más pronunciada.  
- Verbosidad: requiere escribir más código para tareas simples.  
- Sobrecoste innecesario en apps pequeñas o medianas.  
- Puede generar rigidez si se aplica sin necesidad real.

### 13.2.4. Comparación narrativa: ¿cuándo usar cada uno?

Imaginemos tres escenarios:

1. **Una app de gestión de tareas personales**: pocos componentes, estado local, sin sincronización compleja.  
   → Aquí, los **Signals** o servicios locales son más que suficientes. NGRX sería excesivo.

2. **Una app de e‑commerce con carrito, usuario, catálogo, promociones y sincronización con backend**: múltiples módulos, estado compartido, efectos secundarios.  
   → En este caso, **NGRX** ofrece trazabilidad, escalabilidad y control. Signals pueden usarse en componentes aislados, pero no como solución global.

3. **Una app de dashboard administrativo con filtros, gráficos y configuraciones locales**: estado reactivo pero no global.  
   → **Signals** brillan aquí: permiten construir interfaces fluidas sin necesidad de una arquitectura pesada.

No hay una única respuesta correcta. La elección entre **NGRX**, **Signals** y **servicios locales** depende del contexto.  
Lo importante es entender que cada herramienta tiene un propósito:

- Los **servicios locales** son rápidos y útiles para casos simples.  
- Los **Signals** ofrecen reactividad moderna y rendimiento, ideales para estado local o compartido en pequeña escala.  
- **NGRX** es una solución arquitectónica para aplicaciones que necesitan trazabilidad, escalabilidad y separación de responsabilidades.

En Angular 20, el reto no es elegir “la mejor herramienta”, sino **usar la herramienta adecuada para el problema adecuado**.


## 13.3. Creación y configuración inicial del Store en un proyecto Angular

Una vez que comprendemos qué es NGRX y por qué puede ser útil en aplicaciones Angular complejas, el siguiente paso es **crear y configurar el Store**, que es el núcleo del estado global.  
El Store actúa como un contenedor reactivo de datos: cualquier componente puede leer desde él, y cualquier acción puede modificarlo de forma controlada.  
En Angular 20, con la sintaxis moderna basada en funciones (`createAction`, `createReducer`, `createSelector`), esta configuración es más clara, más declarativa y más fácil de mantener que en versiones anteriores.

### 13.3.1. Instalación de NGRX en el proyecto

Angular CLI ofrece un esquema para instalar NGRX de forma rápida:

```bash
ng add @ngrx/store
```

Este comando:

- Instala la librería `@ngrx/store`.  
- Añade el módulo `StoreModule` al `AppModule`.  
- Crea una carpeta `store/` con archivos base si se usa el esquema extendido.  

Opcionalmente, también puedes instalar:

```bash
ng add @ngrx/effects
ng add @ngrx/store-devtools
```

Esto añade soporte para efectos secundarios y herramientas de depuración.

### 13.3.2. Estructura inicial del Store

En NGRX moderno, el estado se organiza en **slices** o “rebanadas” funcionales. Cada slice representa un dominio de la aplicación: usuario, carrito, productos, configuración, etc.

Ejemplo: crear un slice para el estado del usuario.

#### 1. Definir el modelo de estado

```ts
export interface UserState {
  name: string;
  email: string;
  loggedIn: boolean;
}
```

#### 2. Crear acciones

```ts
import { createAction, props } from '@ngrx/store';

export const login = createAction('[User] Login', props<{ name: string; email: string }>());
export const logout = createAction('[User] Logout');
```

#### 3. Crear el reducer

```ts
import { createReducer, on } from '@ngrx/store';
import { login, logout } from './user.actions';
import { UserState } from './user.model';

export const initialState: UserState = {
  name: '',
  email: '',
  loggedIn: false
};

export const userReducer = createReducer(
  initialState,
  on(login, (state, { name, email }) => ({
    ...state,
    name,
    email,
    loggedIn: true
  })),
  on(logout, () => initialState)
);
```

#### 4. Registrar el reducer en el AppModule

```ts
import { StoreModule } from '@ngrx/store';
import { userReducer } from './store/user.reducer';

@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({ user: userReducer })
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

Esto crea un Store con una slice llamada `user`, accesible desde cualquier componente.

### 13.3.3. Lectura del estado con Selectors

Para acceder al estado de forma eficiente y desacoplada, se usan **selectors**:

```ts
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { UserState } from './user.model';

export const selectUserState = createFeatureSelector<UserState>('user');

export const selectUserName = createSelector(
  selectUserState,
  (state) => state.name
);
```

En el componente:

```ts
export class HeaderComponent {
  userName$ = this.store.select(selectUserName);

  constructor(private store: Store) {}
}
```

Esto permite que el componente se actualice automáticamente cuando el nombre del usuario cambie.

### 13.3.4. Desencadenar acciones desde componentes

Para modificar el estado, se despachan acciones:

```ts
this.store.dispatch(login({ name: 'Gerardo', email: 'gerardo@example.com' }));
```

Esto activa el reducer correspondiente y actualiza el estado global.

La **configuración inicial del Store** en Angular con NGRX no es compleja, pero sí exige claridad en la estructura: definir bien los modelos, las acciones y los reducers.  
Una vez configurado, el Store se convierte en el **sistema nervioso de la aplicación**, permitiendo que todos los componentes trabajen sobre un estado común, reactivo y trazable.  
En Angular 20, esta arquitectura se complementa perfectamente con Signals, Effects y DevTools, ofreciendo una experiencia de desarrollo moderna y robusta.


## 13.4. Definición y uso de Actions para describir eventos de la aplicación

En el patrón Redux —y por extensión en **NGRX**—, las **Actions** son el lenguaje con el que describimos lo que ocurre en la aplicación.  
Podemos pensar en ellas como **mensajes inmutables** que narran un evento: “el usuario inició sesión”, “se añadió un producto al carrito”, “la API devolvió un error”.  
No contienen lógica de negocio ni transforman el estado por sí mismas; su único propósito es **describir un hecho** de manera clara y estructurada.  

Este enfoque aporta una gran ventaja: el estado de la aplicación no cambia de forma arbitraria, sino siempre como respuesta a un evento explícito. Esto hace que el flujo de datos sea **predecible, trazable y fácil de depurar**.

### 13.4.1.  ¿Qué es una Action en NGRX?

Una Action es un objeto con al menos una propiedad obligatoria: **`type`**, que identifica el evento.  
En NGRX moderno, usamos la función `createAction` para definirlas de forma declarativa y tipada.

Ejemplo básico:

```ts
import { createAction, props } from '@ngrx/store';

export const login = createAction(
  '[Auth] Login',
  props<{ email: string; password: string }>()
);

export const loginSuccess = createAction(
  '[Auth] Login Success',
  props<{ userId: string; token: string }>()
);

export const loginFailure = createAction(
  '[Auth] Login Failure',
  props<{ error: string }>()
);
```

- El **prefijo entre corchetes** (`[Auth]`) indica el dominio o feature al que pertenece la acción.  
- El **nombre** describe el evento de forma clara.  
- `props` define la carga útil (payload) de datos que acompaña a la acción.  

### 13.4.2. El ciclo de vida de una Action

1. **Un componente o servicio despacha una acción** con `store.dispatch()`.  
   ```ts
   this.store.dispatch(login({ email: 'user@test.com', password: '1234' }));
   ```

2. **El reducer escucha esa acción** y actualiza el estado en consecuencia.  
   ```ts
   on(loginSuccess, (state, { userId, token }) => ({
     ...state,
     userId,
     token,
     loggedIn: true
   }))
   ```

3. **Los efectos (Effects)** pueden reaccionar a la acción para ejecutar lógica asíncrona, como llamadas HTTP.  
   ```ts
   loadUser$ = createEffect(() =>
     this.actions$.pipe(
       ofType(login),
       switchMap(action => this.authService.login(action.email, action.password)
         .pipe(
           map(user => loginSuccess({ userId: user.id, token: user.token })),
           catchError(err => of(loginFailure({ error: err.message })))
         ))
     )
   );
   ```

De esta manera, las Actions se convierten en el **punto de entrada único** para cualquier cambio de estado o efecto secundario.

### 13.4.3. Ventajas de usar Actions

- **Claridad semántica**: cada cambio en la aplicación está descrito por un evento con nombre explícito.  
- **Trazabilidad**: gracias a DevTools, podemos ver el historial de acciones y reproducir el flujo de la aplicación paso a paso.  
- **Desacoplamiento**: los componentes no modifican el estado directamente, solo emiten eventos. Reducers y Effects se encargan del resto.  
- **Escalabilidad**: en equipos grandes, las Actions sirven como contrato común entre frontend, backend y diseño de flujos.  

### 13.4.4. Buenas prácticas al definir Actions

- Usar **nombres consistentes**: `[Feature] Evento`.  
- Mantener las Actions **simples y descriptivas**: deben narrar lo que pasó, no cómo se resuelve.  
- Evitar sobrecargar las Actions con demasiados datos: solo incluir lo necesario.  
- Agrupar las Actions por dominio en archivos separados (`auth.actions.ts`, `cart.actions.ts`).  
- Usar `props` para tipar correctamente los datos y evitar errores en tiempo de compilación.  

Las **Actions** son el vocabulario con el que una aplicación Angular con NGRX cuenta su propia historia.  
Cada interacción del usuario, cada respuesta del servidor y cada cambio interno se traduce en un evento explícito.  
Esto no solo facilita la depuración y el mantenimiento, sino que también aporta una **narrativa clara** al flujo de datos: sabemos qué ocurrió, cuándo ocurrió y qué consecuencia tuvo en el estado global.


## 13.5. Implementación de Reducers para gestionar el estado global

En el patrón Redux —y en su adaptación a Angular con NGRX—, los **Reducers** son las funciones puras que definen cómo cambia el estado global de la aplicación en respuesta a las **Actions**.  
Podemos pensarlos como **intérpretes de eventos**: reciben el estado actual y una acción que describe lo ocurrido, y devuelven un nuevo estado actualizado.  

La clave está en que los reducers son **funciones puras**:  
- No modifican el estado original, sino que crean una copia inmutable con los cambios.  
- No tienen efectos secundarios (no hacen llamadas HTTP, no acceden al DOM, no generan logs).  
- Su única responsabilidad es **transformar el estado** de manera predecible.

### 13.5.1. Reducers en la práctica

Supongamos que tenemos un estado global para gestionar un carrito de compras. El modelo podría ser:

```ts
export interface CartState {
  items: { productId: string; quantity: number }[];
}
```

#### 1. Definir acciones

```ts
import { createAction, props } from '@ngrx/store';

export const addItem = createAction(
  '[Cart] Add Item',
  props<{ productId: string }>()
);

export const removeItem = createAction(
  '[Cart] Remove Item',
  props<{ productId: string }>()
);

export const clearCart = createAction('[Cart] Clear');
```

#### 2. Crear el reducer

```ts
import { createReducer, on } from '@ngrx/store';
import { addItem, removeItem, clearCart } from './cart.actions';
import { CartState } from './cart.model';

export const initialState: CartState = {
  items: []
};

export const cartReducer = createReducer(
  initialState,

  on(addItem, (state, { productId }) => {
    const existing = state.items.find(i => i.productId === productId);
    if (existing) {
      return {
        ...state,
        items: state.items.map(i =>
          i.productId === productId ? { ...i, quantity: i.quantity + 1 } : i
        )
      };
    }
    return {
      ...state,
      items: [...state.items, { productId, quantity: 1 }]
    };
  }),

  on(removeItem, (state, { productId }) => ({
    ...state,
    items: state.items.filter(i => i.productId !== productId)
  })),

  on(clearCart, () => initialState)
);
```

#### 3. Registrar el reducer en el Store

```ts
import { StoreModule } from '@ngrx/store';
import { cartReducer } from './store/cart.reducer';

@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({ cart: cartReducer })
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### 13.5.2. Principios clave de los Reducers

1. **Inmutabilidad**: nunca modificar el estado directamente (`state.items.push(...)` está prohibido). Siempre devolver un nuevo objeto.  
2. **Pureza**: no ejecutar lógica externa (fetch, logs, timers). Eso corresponde a los **Effects**.  
3. **Determinismo**: dado el mismo estado y la misma acción, el resultado debe ser siempre el mismo.  
4. **Simplicidad**: cada reducer debe encargarse de un dominio concreto (ej. `cartReducer`, `userReducer`).  

### 13.5.3. Reducers y escalabilidad

En aplicaciones grandes, el Store se divide en múltiples slices, cada uno con su propio reducer. Angular permite combinarlos fácilmente:

```ts
StoreModule.forRoot({
  cart: cartReducer,
  user: userReducer,
  products: productsReducer
})
```

Esto mantiene el estado organizado y facilita que distintos equipos trabajen en paralelo sobre diferentes dominios.

Los **Reducers** son el corazón de NGRX: convierten las acciones en cambios de estado de forma clara, predecible y trazable.  
Gracias a su naturaleza pura e inmutable, permiten que el estado global sea confiable y fácil de depurar.  
En combinación con **Actions** y **Selectors**, los reducers forman la base de una arquitectura sólida que escala sin perder claridad, incluso en aplicaciones empresariales de gran tamaño.


## 13.6. Uso de Selectors para obtener datos del Store en componentes Angular

En NGRX, los **Reducers** definen cómo cambia el estado y las **Actions** describen qué ocurrió. Pero falta un eslabón clave: **¿cómo acceden los componentes a ese estado global sin acoplarse a la estructura interna del Store?**  
La respuesta son los **Selectors**: funciones puras que permiten **consultar el estado** de forma eficiente, reutilizable y desacoplada.  

Podemos pensar en los Selectors como **consultas SQL para el Store**: no nos importa cómo están guardados los datos internamente, sino cómo obtener la información que necesitamos en cada componente.

### 13.6.1. ¿Qué es un Selector?

Un **Selector** es una función que recibe el estado global y devuelve un fragmento derivado de él.  
En NGRX moderno, se crean con `createSelector` o `createFeatureSelector`.  

Ejemplo básico:

```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { UserState } from './user.model';

export const selectUserState = createFeatureSelector<UserState>('user');

export const selectUserName = createSelector(
  selectUserState,
  (state) => state.name
);
```

- `createFeatureSelector` apunta a una “slice” del estado (`user`).  
- `createSelector` permite derivar datos específicos (ej. el nombre del usuario).  

### 13.6.2. Uso de Selectors en componentes Angular

En un componente, podemos suscribirnos al Store usando `store.select(selector)`:

```ts
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { selectUserName } from './store/user.selectors';

@Component({
  selector: 'app-header',
  template: `<h1>Bienvenido, {{ userName$ | async }}</h1>`
})
export class HeaderComponent {
  userName$ = this.store.select(selectUserName);

  constructor(private store: Store) {}
}
```

- `userName$` es un observable que emite el nombre del usuario cada vez que cambia en el Store.  
- El `async` pipe en la plantilla se encarga de la suscripción y desuscripción automática.  

De esta forma, el componente **no conoce la estructura interna del estado**, solo consume el selector.

### 13.6.3. Selectors derivados y memoización

Una de las grandes ventajas de los Selectors es que pueden **derivar datos** y están **memoizados**:  
- Si el estado no cambia, el selector devuelve el mismo resultado sin recalcular.  
- Esto mejora el rendimiento y evita renders innecesarios.  

Ejemplo: calcular el número total de productos en el carrito.

```ts
import { createSelector } from '@ngrx/store';
import { CartState } from './cart.model';

export const selectCartState = createFeatureSelector<CartState>('cart');

export const selectCartItems = createSelector(
  selectCartState,
  (state) => state.items
);

export const selectCartTotal = createSelector(
  selectCartItems,
  (items) => items.reduce((total, item) => total + item.quantity, 0)
);
```

En el componente:

```ts
cartTotal$ = this.store.select(selectCartTotal);
```

Así, el componente recibe directamente el dato procesado, sin necesidad de lógica adicional.

### 13.6.4. Beneficios de usar Selectors

- **Desacoplamiento**: los componentes no dependen de la estructura interna del Store.  
- **Reutilización**: un mismo selector puede usarse en múltiples componentes.  
- **Optimización**: gracias a la memoización, se evita recalcular datos innecesariamente.  
- **Legibilidad**: los Selectors actúan como una capa de consultas declarativas sobre el estado.  
- **Escalabilidad**: en aplicaciones grandes, permiten organizar el acceso al estado de forma clara y mantenible.  

### 13.6.5. Buenas prácticas

- Definir un **feature selector** por cada slice del estado (`user`, `cart`, `products`).  
- Crear **selectors derivados** para cálculos frecuentes (totales, filtros, estados combinados).  
- Mantener los Selectors en archivos separados (`user.selectors.ts`, `cart.selectors.ts`).  
- Evitar lógica compleja en los componentes: delegar siempre al selector.  
- Usar `async` pipe en plantillas para simplificar la suscripción.  

Los **Selectors** son la forma idiomática de acceder al estado en NGRX.  
Permiten que los componentes se mantengan simples, declarativos y enfocados en la presentación, mientras que la lógica de acceso y derivación de datos se concentra en funciones puras, reutilizables y optimizadas.  
En aplicaciones grandes, esta separación es lo que marca la diferencia entre un Store caótico y un Store mantenible y escalable.


## 13.7. Implementación de Effects para manejar efectos secundarios y peticiones HTTP

Hasta ahora hemos visto cómo las **Actions** describen lo que ocurre en la aplicación y cómo los **Reducers** transforman el estado de manera pura y predecible. Sin embargo, en una aplicación real, muchos eventos no se limitan a actualizar el estado local: necesitan **interactuar con el mundo exterior**.  
Ejemplos típicos son:  
- Llamadas HTTP a un backend.  
- Acceso a APIs del navegador.  
- Escritura en almacenamiento local.  
- Registro de métricas o logging.  

Estas operaciones se conocen como **efectos secundarios** (*side effects*), porque ocurren fuera del flujo puro de datos. En NGRX, la forma idiomática de manejarlos es mediante los **Effects**.

### 13.7.1. ¿Qué es un Effect?

Un **Effect** es un servicio especial que escucha las **Actions** que se despachan en la aplicación y, en respuesta, ejecuta lógica asíncrona o externa.  
Después de completar esa lógica, el Effect puede:  
- Despachar nuevas acciones (ej. `loadUsersSuccess`, `loadUsersFailure`).  
- O simplemente ejecutar la operación sin modificar el estado (ej. enviar un log).  

La clave es que los Effects **no mutan el estado directamente**: siempre lo hacen a través de nuevas acciones que los reducers interpretan.

### 13.7.2. Ejemplo: carga de usuarios desde una API

Supongamos que tenemos un módulo de usuarios y queremos cargar la lista desde un backend.

#### 1. Definir acciones

```ts
import { createAction, props } from '@ngrx/store';
import { User } from './user.model';

export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User] Load Users Failure',
  props<{ error: string }>()
);
```

#### 2. Crear el Effect

```ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { UserService } from '../services/user.service';
import { loadUsers, loadUsersSuccess, loadUsersFailure } from './user.actions';
import { catchError, map, mergeMap, of } from 'rxjs';

@Injectable()
export class UserEffects {
  constructor(private actions$: Actions, private userService: UserService) {}

  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers), // Escucha la acción
      mergeMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })), // Éxito
          catchError(error => of(loadUsersFailure({ error: error.message }))) // Error
        )
      )
    )
  );
}
```

- `Actions` es un observable que emite todas las acciones despachadas en la app.  
- `ofType(loadUsers)` filtra solo las que nos interesan.  
- `mergeMap` ejecuta la llamada HTTP y devuelve nuevas acciones según el resultado.  

#### 3. Registrar los Effects en el módulo

```ts
import { EffectsModule } from '@ngrx/effects';
import { UserEffects } from './store/user.effects';

@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({}),
    EffectsModule.forRoot([UserEffects])
  ]
})
export class AppModule {}
```

### 13.7.3. Patrones comunes en Effects

- **Cargar datos**: escuchar una acción de “load” y devolver `success` o `failure`.  
- **Encadenar acciones**: un Effect puede despachar otra acción que a su vez dispare otro Effect.  
- **No despachar acciones**: algunos Effects solo ejecutan lógica (ej. logging). En ese caso se configura con `{ dispatch: false }`.  

Ejemplo:

```ts
logActions$ = createEffect(() =>
  this.actions$.pipe(
    tap(action => console.log('Acción despachada:', action))
  ),
  { dispatch: false }
);
```

### 13.7.4. Beneficios de usar Effects

- **Separación de responsabilidades**: los reducers siguen siendo funciones puras, y la lógica asíncrona se concentra en los Effects.  
- **Escalabilidad**: cada dominio puede tener sus propios Effects, organizados y mantenibles.  
- **Reactividad**: al estar basados en RxJS, permiten componer flujos complejos de manera declarativa.  
- **Depuración**: junto con DevTools, es posible seguir el ciclo completo: acción → effect → nueva acción → reducer → nuevo estado.  

### 13.7.5. Buenas prácticas

- Mantener los Effects **enfocados en un dominio** (ej. `UserEffects`, `CartEffects`).  
- Usar operadores adecuados (`mergeMap`, `switchMap`, `concatMap`) según el caso de concurrencia.  
- Manejar siempre los errores con `catchError` para evitar que el flujo se rompa.  
- Evitar lógica de transformación compleja en los Effects: delegar al servicio o al reducer.  
- Documentar las acciones que dispara cada Effect para facilitar la trazabilidad.  

Los **Effects** son el puente entre el mundo puro y predecible del Store y la realidad asíncrona de las aplicaciones modernas.  
Gracias a ellos, Angular con NGRX puede manejar llamadas HTTP, sincronización con APIs y cualquier efecto secundario sin comprometer la claridad de la arquitectura.  
En aplicaciones grandes, los Effects son la clave para mantener un flujo de datos **estructurado, escalable y fácil de depurar**.


## 13.8. Estrategias para combinar NGRX con Signals en proyectos modernos

Con la llegada de **Signals** en Angular 16 y su consolidación en Angular 20, muchos equipos se preguntan: ¿qué lugar ocupa NGRX en este nuevo paradigma?  
La respuesta no es un reemplazo absoluto, sino una **convivencia estratégica**. Mientras NGRX sigue siendo la mejor opción para **estado global, trazabilidad y efectos secundarios complejos**, los Signals brillan en la **gestión de estado local, reactividad inmediata y simplicidad**.  
La clave está en **saber cuándo usar cada uno y cómo integrarlos de manera armónica**.

### 13.8.1. Diferencias de enfoque

- **NGRX**:  
  - Diseñado para **estado global y compartido**.  
  - Ofrece trazabilidad, DevTools, separación clara de responsabilidades.  
  - Ideal para aplicaciones grandes y equipos distribuidos.  

- **Signals**:  
  - Pensados para **estado local y reactivo**.  
  - Integración nativa con Angular, sin necesidad de RxJS.  
  - Perfectos para componentes, formularios y vistas que requieren reactividad inmediata.  

En lugar de verlos como rivales, podemos verlos como **capas complementarias**: NGRX gestiona el “sistema nervioso central” de la aplicación, mientras que los Signals se encargan de los reflejos locales y rápidos.

### 13.8.2. Estrategias de integración

> Recomendación Angular 20+: Convierte los observables del store en signals en la capa de presentación usando `toSignal` para aprovechar la reactividad moderna y simplificar la integración con la nueva API de Angular.

#### 1. Signals como capa de presentación sobre NGRX
Una práctica común es usar NGRX para almacenar el estado global y exponerlo a los componentes a través de **Selectors**.  
En el componente, ese observable puede convertirse en un Signal para integrarse mejor con la nueva API reactiva de Angular:

```ts
import { ChangeDetectionStrategy, Component } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { selectUserName } from './store/user.selectors';

@Component({
  selector: 'app-header',
  template: `<h1>Bienvenido, {{ userName() }}</h1>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class HeaderComponent {
  userName = toSignal(this.store.select(selectUserName));

  constructor(private store: Store) {}
}
```

De esta forma, el Store sigue siendo la fuente de verdad, pero el componente trabaja con Signals, lo que simplifica la reactividad.

#### 2. Signals para estado local, NGRX para estado global
En un formulario complejo, podemos usar Signals para manejar validaciones y cambios inmediatos de campos, mientras que el resultado final se despacha al Store con una acción:

```ts
formName = signal('');
formEmail = signal('');

save() {
  this.store.dispatch(saveUser({
    name: this.formName(),
    email: this.formEmail()
  }));
}
```

Así, evitamos sobrecargar el Store con estados efímeros que solo importan dentro del componente.

#### 3. Computed Signals como Selectors locales
Los **computed signals** permiten derivar datos de otros signals, de forma similar a los Selectors de NGRX.  
Esto es útil cuando necesitamos cálculos rápidos en la UI, sin necesidad de definir un selector global.

```ts
cartItems = toSignal(this.store.select(selectCartItems));
cartTotal = computed(() =>
  this.cartItems()?.reduce((t, i) => t + i.quantity, 0) ?? 0
);
```

Aquí, el Store provee los datos base, y los Signals derivan cálculos inmediatos para la vista.

#### 4. Effects + Signals para side effects locales
Los **Effects** de NGRX siguen siendo la mejor opción para lógica asíncrona global (llamadas HTTP, sincronización con APIs).  
Pero para efectos locales, como mostrar un mensaje temporal en un componente, los **effects de Signals** (`effect()`) son más ligeros y directos:

```ts
message = signal('');
effect(() => {
  if (this.message()) {
    console.log('Nuevo mensaje:', this.message());
  }
});
```

### 13.8.3. Buenas prácticas en la combinación

- **No duplicar responsabilidades**: el estado global debe estar en NGRX; el estado local, en Signals.  
- **Convertir observables a signals** en la capa de presentación, no en la lógica de negocio.  
- **Usar selectors de NGRX para datos compartidos** y signals para cálculos inmediatos en la UI.  
- **Mantener Effects en NGRX para side effects globales** y `effect()` de Signals para lógica local.  
- **Pensar en capas**: NGRX como la base estructural, Signals como la capa de interacción rápida.  
- **Utilizar Angular DevTools y Redux DevTools** para depuración y trazabilidad del estado y las acciones.

La combinación de **NGRX y Signals** no es un dilema, sino una oportunidad.  
NGRX sigue siendo la columna vertebral para aplicaciones grandes y distribuidas, mientras que Signals aporta fluidez y simplicidad en la capa de presentación.  
En Angular 20, la arquitectura moderna no consiste en elegir entre uno u otro, sino en **aprovechar lo mejor de ambos mundos**: la trazabilidad y escalabilidad de NGRX junto con la reactividad inmediata y ergonómica de Signals.

