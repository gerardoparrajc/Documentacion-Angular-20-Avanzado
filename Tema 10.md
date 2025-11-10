# 10. Testing moderno en Angular 20

## 10.1. Introducción a las estrategias de testing modernas en Angular

El testing en Angular ha evolucionado notablemente en las últimas versiones. Con Angular 20, disponemos de herramientas y patrones que permiten escribir pruebas **más rápidas, legibles, mantenibles y cercanas a la experiencia real del usuario**.

En esta introducción veremos **qué ha cambiado**, **por qué es importante** y **qué estrategias modernas** deberíamos adoptar para asegurar la calidad de nuestras aplicaciones.

### 10.1.1. ¿Por qué hablar de "testing moderno"?

En el pasado, las pruebas en Angular se centraban mucho en **detalles internos** (componentes, métodos privados, mocks excesivos).  
Hoy, el enfoque ha cambiado hacia **pruebas más cercanas al comportamiento real del usuario** y a la **integración fluida con el ecosistema**.

**Claves del testing moderno:**

- **Menos pruebas frágiles:** Evitar tests que fallen por cambios internos irrelevantes.
- **Más enfoque en la experiencia de usuario:** Validar que la aplicación funciona como se espera desde la perspectiva del usuario.
- **Automatización en CI/CD:** Integrar las pruebas en pipelines para detectar errores antes de llegar a producción.
- **Uso de herramientas modernas:** Harnesses, TestBed simplificado, Playwright, Cypress, Jest, etc.

### 10.1.2. Cambios y mejoras en Angular 20 para testing

| Característica | Descripción | Beneficio |
|----------------|-------------|-----------|
| **TestBed simplificado** | Menos configuración repetitiva, inicialización más rápida. | Tests más cortos y claros. |
| **Component Harnesses** | API oficial para interactuar con componentes de forma estable. | Menos dependencia de selectores frágiles. |
| **Mejor soporte para Standalone Components** | Configuración directa sin módulos de prueba. | Menos boilerplate. |
| **Compatibilidad con Jest y Vitest** | Ejecución más rápida y feedback inmediato. | Mejora la productividad. |
| **Integración con Playwright** | E2E más estables y rápidos. | Cobertura de escenarios reales. |

### 10.1.3. Tipos de pruebas en un enfoque moderno

1. **Unitarias (Unit Tests)**  
   - Validan piezas pequeñas (funciones, servicios, pipes).
   - Rápidas y fáciles de mantener.
   - Ejemplo: comprobar que un pipe formatea correctamente una fecha.

2. **De integración (Integration Tests)**  
   - Verifican que varios componentes/servicios funcionan juntos.
   - Útiles para detectar problemas de comunicación entre piezas.

3. **End-to-End (E2E)**  
   - Simulan la interacción real del usuario en el navegador.
   - Detectan problemas que no aparecen en pruebas unitarias.
   - Herramientas recomendadas: Playwright, Cypress.

4. **Pruebas de accesibilidad (A11y Testing)**  
   - Validan que la aplicación sea usable por personas con discapacidad.
   - Herramientas: axe-core, pa11y.

### 10.1.4. Principios de las estrategias modernas

En el testing moderno de Angular 20, no se trata solo de “probar por probar”, sino de **probar de forma inteligente**, optimizando el tiempo de desarrollo y asegurando que las pruebas aporten valor real al producto.

### 1️⃣ Testea comportamientos, no implementaciones internas
- **Qué significa:** Valida la funcionalidad visible o el contrato público, no métodos privados.
- **Por qué:** Cambios internos no deberían romper tests si el comportamiento externo es el mismo.
- **Ejemplo:**
  ```ts
  // ❌ Frágil
  expect(component.calculateTotal()).toBe(100);

  // ✅ Robusto
  component.addItem({ price: 50 });
  component.addItem({ price: 50 });
  fixture.detectChanges();
  expect(fixture.nativeElement.querySelector('.total').textContent).toContain('100');
  ```

### 2️⃣ Usa APIs oficiales de testing (Harnesses, TestBed)
- **Qué significa:** Aprovecha herramientas como Component Harnesses para interactuar con componentes.
- **Beneficio:** Menos dependencia de selectores CSS frágiles.
- **Ejemplo:**
  ```ts
  const button = await loader.getHarness(MatButtonHarness.with({ text: /Guardar/i }));
  await button.click();
  ```

### 3️⃣ Minimiza mocks innecesarios
- **Qué significa:** Simula solo lo imprescindible (APIs externas, recursos costosos).
- **Por qué:** Demasiados mocks pueden ocultar errores reales.

### 4️⃣ Automatiza en CI/CD
- **Qué significa:** Ejecuta tests automáticamente en cada commit o PR.
- **Beneficio:** Detecta errores antes de producción.
- **Tip:** Separa suites rápidas (unitarias) y lentas (E2E).

### 5️⃣ Mide cobertura, pero prioriza calidad
- **Qué significa:** La cobertura es un indicador, no un fin.
- **Ejemplo:** Evita tests triviales solo para subir el porcentaje.

### 6️⃣ Incluye pruebas de accesibilidad desde el inicio
- **Qué significa:** Garantiza que la app sea usable por todos.
- **Herramientas:** axe-core, assertions de accesibilidad en Playwright.

### 7️⃣ Mantén los tests independientes y reproducibles
- **Qué significa:** Cada test debe poder ejecutarse solo.
- **Tip:** Resetea estado en `beforeEach` y usa datos consistentes.


### 10.1.5. Ejemplo introductorio

**Componente a probar:**
```ts
import { ChangeDetectionStrategy, Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h1>{{ count() }}</h1>
    <button (click)="inc()">Incrementar</button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class CounterComponent {
  count = signal(0);
  inc() { this.count.update(v => v + 1); }
}
```

**Test moderno con TestBed simplificado:**
```ts
import { TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';
import { By } from '@angular/platform-browser';

describe('CounterComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [CounterComponent]
    });
  });

  it('debe incrementar el contador al hacer click', () => {
    const fixture = TestBed.createComponent(CounterComponent);
    fixture.detectChanges();

    const button = fixture.debugElement.query(By.css('button')).nativeElement;
    button.click();
    fixture.detectChanges();

    const value = fixture.debugElement.query(By.css('h1')).nativeElement.textContent;
    expect(value).toBe('1');
  });
});
```

### 10.1.6. Buenas prácticas iniciales

- **Nombrar claramente** los tests: que el nombre explique el comportamiento esperado.
- **Mantener independencia**: cada test debe poder ejecutarse solo.
- **Evitar dependencias externas reales**: usar mocks para APIs externas.
- **Revisar y refactorizar** tests junto con el código de producción.
- **Adoptar la pirámide de testing**: más unitarios, menos E2E, pero bien equilibrados.

## 10.2. Unit testing con Jest: configuración y ventajas frente a Karma/Jasmine

En Angular, el entorno de testing por defecto ha sido tradicionalmente **Karma** (runner) junto con **Jasmine** (framework de aserciones).  
En proyectos modernos, **Jest** se ha convertido en una alternativa muy popular gracias a su **velocidad**, **simplicidad** y **mejor experiencia de desarrollo**.

### Ventajas de Jest frente a Karma/Jasmine

| Característica | Karma/Jasmine | Jest |
|----------------|---------------|------|
| **Velocidad** | Más lento, ejecuta en navegador real | Muy rápido, ejecuta en Node.js |
| **Configuración** | Más compleja, varios archivos | Sencilla, un solo archivo `jest.config.js` |
| **Feedback** | Menos inmediato | Feedback rápido y claro en consola |
| **Cobertura** | Requiere configuración extra | Integrada por defecto (`--coverage`) |
| **Mocks** | Manuales o librerías externas | Sistema de mocks integrado |
| **Snapshots** | No disponible | Integrado, útil para componentes y objetos |
| **Paralelismo** | Limitado | Corre tests en paralelo por defecto |

### Configuración de Jest en Angular 20

### 1️⃣ Instalar dependencias

```bash
npm install --save-dev jest jest-preset-angular @types/jest ts-jest jest-environment-jsdom  @angular/platform-browser-dynamic --legacy-peer-deps
```

- **jest**: runner y framework de testing.
- **jest-preset-angular**: configuración específica para Angular.
- **@types/jest**: tipado para TypeScript.

---

### 2️⃣ Archivo de configuración

Crea `jest.config.js` en la raíz del proyecto:

```js
module.exports = {
  preset: 'jest-preset-angular',
  setupFilesAfterEnv: ['<rootDir>/setup-jest.ts'],
  testEnvironment: 'jsdom',
  moduleFileExtensions: ['ts', 'html', 'js', 'json'],
  coverageDirectory: 'coverage/jest',
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/main.ts',
    '!src/polyfills.ts'
  ]
};
```

---

### 3️⃣ Archivo de setup

Crea `setup-jest.ts`:

```ts
import { setupZoneTestEnv } from 'jest-preset-angular/setup-env/zone';
setupZoneTestEnv();
```

---

### 4️⃣ Ajustar `tsconfig.spec.json`

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "types": ["jest"]
  },
  "files": ["src/polyfills.ts"],
  "include": ["src/**/*.spec.ts", "src/**/*.d.ts"]
}
```

---

### 5️⃣ Scripts en `package.json`

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### Ejemplo básico de test con Jest

**Componente:**
```ts
import { ChangeDetectionStrategy, Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>{{ greet('Gerardo') }}</h1>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class HelloComponent {
  greet(name: string) {
    return `Hola, ${name}!`;
  }
}
```

**Test:**
```ts
import { HelloComponent } from './hello.component';

describe('HelloComponent', () => {
  it('debe saludar correctamente', () => {
    const comp = new HelloComponent();
    expect(comp.greet('Gerardo')).toBe('Hola, Gerardo!');
  });
});
```

### Buenas prácticas con Jest en Angular

- Usar `jest-preset-angular` para simplificar la configuración.
- Aprovechar snapshots para componentes con HTML estable.
- Configurar paths en `tsconfig` y `jest.config.js` para imports limpios.
- Ejecutar en CI con `jest --ci` para resultados consistentes.
- Combinar con TestBed para pruebas de componentes Angular.

## 10.3. Harnesses del Angular CDK para pruebas aisladas de componentes

Cuando escribimos pruebas de componentes en Angular, uno de los retos más comunes es **evitar que los tests se rompan por cambios internos en la plantilla**.  
Si tu test depende de un selector CSS concreto o de la estructura HTML exacta, cualquier cambio en el marcado —aunque no afecte al comportamiento— puede provocar fallos innecesarios. Esto genera frustración y pérdida de tiempo.

Para resolver este problema, Angular introdujo los **Component Harnesses** dentro del **CDK (Component Dev Kit)**.  
Un Harness es, en esencia, una **capa de abstracción** que nos permite interactuar con un componente como si fuera una “caja negra”: no nos importa cómo está construido por dentro, solo nos interesa lo que hace y cómo responde a nuestras acciones.

### ¿Qué es exactamente un Harness?

Podemos imaginarlo como un **control remoto** para un componente.  
En lugar de “meter la mano” dentro del HTML y buscar elementos con `querySelector`, usamos métodos de alto nivel que el propio Harness nos proporciona: `clickSaveButton()`, `getValue()`, `isDisabled()`, etc.

Esto tiene varias ventajas:

- **Aislamiento total**: el test no depende de la estructura interna del componente.
- **Estabilidad**: cambios en el HTML no rompen el test mientras el comportamiento se mantenga.
- **Reutilización**: el mismo Harness puede usarse en pruebas unitarias y en pruebas E2E.
- **Legibilidad**: el código del test describe la intención, no los detalles técnicos.

### Cómo funcionan en la práctica

Los Harnesses se basan en tres conceptos clave:

1. **Host Selector**: indica qué elemento del DOM representa el componente.
2. **Locators**: funciones para encontrar elementos internos de forma segura.
3. **Predicados**: filtros para localizar componentes que cumplan ciertas condiciones (por ejemplo, un botón con un texto específico).

Cuando usamos Harnesses oficiales (como los de Angular Material), estos ya vienen con métodos listos para interactuar con el componente.  
Si el componente es nuestro, podemos crear un Harness personalizado que exponga las acciones y lecturas que necesitamos.

### Ejemplo con un Harness oficial

Supongamos que tenemos un componente con botones de Angular Material:

```ts
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-actions',
  standalone: true,
  imports: [MatButtonModule],
  template: `
    <button mat-button (click)="save()">Guardar</button>
    <button mat-button color="warn" (click)="delete()">Eliminar</button>
  `
})
export class ActionsComponent {
  save() { console.log('Guardado'); }
  delete() { console.log('Eliminado'); }
}
```

En un test tradicional, buscaríamos el botón con `By.css('button')` y haríamos click.  
Pero si mañana cambiamos el orden de los botones o añadimos un icono, el selector podría dejar de funcionar.

Con un Harness, el test se vuelve más robusto:

```ts
import { TestBed } from '@angular/core/testing';
import { ActionsComponent } from './actions.component';
import { MatButtonHarness } from '@angular/material/button/testing';
import { TestbedHarnessEnvironment } from '@angular/cdk/testing/testbed';

describe('ActionsComponent con Harnesses', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [ActionsComponent]
    });
  });

  it('debe encontrar y hacer click en el botón Guardar', async () => {
    const fixture = TestBed.createComponent(ActionsComponent);
    fixture.detectChanges();

    const loader = TestbedHarnessEnvironment.loader(fixture);
    const saveButton = await loader.getHarness(MatButtonHarness.with({ text: 'Guardar' }));

    await saveButton.click();

    // Aquí podríamos verificar efectos secundarios del click
    expect(saveButton).toBeTruthy();
  });
});
```

Aquí no nos importa si el botón tiene un icono, un `span` interno o cambia de color: mientras el texto siga siendo “Guardar”, el Harness lo encontrará.

### Creando un Harness personalizado

Cuando el componente es nuestro y no existe un Harness oficial, podemos crearlo.  
Esto es especialmente útil para componentes complejos con mucha interacción.

**Componente:**
```ts
import { ChangeDetectionStrategy, Component, Input } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h1>{{ value }}</h1>
    <button (click)="increment()">+1</button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class CounterComponent {
  @Input() value = 0;
  increment() { this.value++; }
}
```

**Harness personalizado:**
```ts
import { ComponentHarness } from '@angular/cdk/testing';

export class CounterHarness extends ComponentHarness {
  static hostSelector = 'app-counter';

  async getValue(): Promise<number> {
    const text = await (await this.locatorFor('h1')()).text();
    return Number(text);
  }

  async clickIncrement(): Promise<void> {
    return (await this.locatorFor('button')()).click();
  }
}
```

**Test usando el Harness en app.spect.ts:**
```ts
it('debe incrementar el valor al hacer click', async () => {
    const fixture = TestBed.createComponent(App);
    fixture.detectChanges();

    const loader = TestbedHarnessEnvironment.loader(fixture);
    const counter = await loader.getHarness(CounterHarness);

    expect(await counter.getValue()).toBe(0);
    await counter.clickIncrement();
    expect(await counter.getValue()).toBe(1);
  });
```

### Buenas prácticas y recomendaciones

- **Usar Harnesses oficiales** siempre que estén disponibles: ahorran tiempo y están bien mantenidos.
- **Nombrar métodos de forma descriptiva**: que el nombre explique la acción (`clickSaveButton()` es mejor que `clickButton()`).
- **Evitar lógica compleja dentro del Harness**: debe ser una capa fina de interacción, no un lugar para procesar datos.
- **Reutilizar Harnesses** en pruebas unitarias y E2E para coherencia.
- **Combinar con predicados** para localizar elementos específicos sin depender de selectores frágiles.

### Cuándo usar Harnesses

- **Componentes con HTML complejo**: evitan que los tests dependan de la estructura interna.
- **Bibliotecas de componentes**: facilitan que otros equipos prueben tus componentes sin conocer su implementación.
- **Pruebas de regresión**: si el HTML cambia pero la funcionalidad sigue igual, el test no se rompe.
- **Integración con E2E**: puedes usar el mismo Harness en pruebas end-to-end para mantener consistencia.


## 10.4. Mocking avanzado con `TestBed.inject()` y Signals

En pruebas unitarias y de integración, el **mocking** es la técnica que nos permite sustituir dependencias reales por versiones simuladas.  
Esto es fundamental para aislar el comportamiento de un componente o servicio y evitar efectos secundarios como llamadas HTTP reales, acceso a bases de datos o manipulación de estado global.

En Angular 20, el uso combinado de **`TestBed.inject()`** y **Signals** abre nuevas posibilidades para crear mocks más potentes, reactivos y fáciles de mantener.

### ¿Por qué usar `TestBed.inject()` para mocking?

Tradicionalmente, en Angular se han usado patrones como `providers: [{ provide: Servicio, useValue: mock }]` dentro de `TestBed.configureTestingModule()` para sustituir dependencias.  
Esto sigue siendo válido, pero `TestBed.inject()` nos da una forma más directa y flexible de **obtener instancias de servicios** ya configurados en el entorno de pruebas.

Ventajas:

- **Acceso inmediato** a la instancia mockeada sin necesidad de inyectarla en el constructor del componente.
- **Control total** sobre el estado del mock durante la prueba.
- **Compatibilidad** con servicios que usan Signals para su estado interno.

### ¿Por qué combinarlo con Signals?

Las **Signals** permiten que el estado del mock sea **reactivo** y que los componentes bajo prueba respondan automáticamente a los cambios que hagamos en el mock.  
Esto es especialmente útil cuando queremos simular flujos dinámicos: cambios de datos, actualizaciones periódicas, eventos de usuario, etc.

### Ejemplo práctico: servicio con Signals

Supongamos que tenemos un servicio que gestiona el estado de usuario:

```ts
import { Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class UserService {
  user = signal<{ name: string; role: string } | null>(null);

  setUser(name: string, role: string) {
    this.user.set({ name, role });
  }

  clearUser() {
    this.user.set(null);
  }
}
```

### Mocking avanzado con `TestBed.inject()` y Signals

Queremos probar un componente que muestra el nombre del usuario y su rol.  
En lugar de usar el servicio real, crearemos un mock que también use Signals para simular cambios de estado.

**Mock del servicio:**
```ts
import { signal } from '@angular/core';

export class MockUserService {
  user = signal<{ name: string; role: string } | null>(null);

  setUser(name: string, role: string) {
    this.user.set({ name, role });
  }

  clearUser() {
    this.user.set(null);
  }
}
```

**Componente bajo prueba:**
```ts
import { Component } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-info',
  standalone: true,
  template: `
    @if (userService.user()) {
      <p>{{ userService.user()?.name }} - {{ userService.user()?.role }}</p>
    } @else {
      <p>No hay usuario</p>
    }
  `
})
export class UserInfoComponent {
  constructor(public userService: UserService) {}
}
```

**Test con `TestBed.inject()` y Signals:**
```ts
import { TestBed } from '@angular/core/testing';
import { UserInfoComponent } from './user-info.component';
import { UserService } from './user.service';
import { MockUserService } from './mock-user.service';

describe('UserInfoComponent con mocking avanzado', () => {
  let mockService: MockUserService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [UserInfoComponent],
      providers: [
        { provide: UserService, useClass: MockUserService }
      ]
    });

    // Obtenemos la instancia mock directamente
    mockService = TestBed.inject(UserService) as unknown as MockUserService;
  });

  it('debe mostrar el usuario cuando el mock lo establece', () => {
    const fixture = TestBed.createComponent(UserInfoComponent);
    fixture.detectChanges();

    // Inicialmente no hay usuario
    expect(fixture.nativeElement.textContent).toContain('No hay usuario');

    // Simulamos login
    mockService.setUser('Gerardo', 'Admin');
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('Gerardo - Admin');
  });

  it('debe mostrar mensaje vacío cuando se limpia el usuario', () => {
    const fixture = TestBed.createComponent(UserInfoComponent);
    fixture.detectChanges();

    mockService.setUser('Ana', 'User');
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('Ana - User');

    mockService.clearUser();
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('No hay usuario');
  });
});
```

### Explicación paso a paso

1. **Creamos un mock del servicio** que mantiene la misma API que el original, pero sin lógica real.
2. **Usamos Signals en el mock** para que el componente reaccione automáticamente a los cambios.
3. **Configuramos `TestBed`** para que inyecte nuestro mock en lugar del servicio real.
4. **Obtenemos la instancia mock con `TestBed.inject()`** para manipular su estado directamente en el test.
5. **Simulamos escenarios**: usuario presente, usuario ausente, cambios de rol, etc.
6. **Verificamos la UI** después de cada cambio para asegurar que el componente responde correctamente.

### Buenas prácticas

- **Mantener la API del mock idéntica** a la del servicio real para evitar errores de tipado.
- **Usar Signals en mocks** cuando el servicio real también las use, para mantener coherencia.
- **Evitar lógica compleja en el mock**: debe ser simple y predecible.
- **Centralizar mocks reutilizables** en una carpeta `testing/mocks` para no duplicar código.
- **Combinar con Harnesses** si el componente tiene interacciones complejas.


## 10.5. Testing de formularios basados en Signals y validaciones dinámicas

Los formularios en Angular han dado un salto importante con la llegada de **Signals**. Ahora es posible que el estado del formulario, sus valores y sus validaciones reaccionen de forma inmediata a cambios en tiempo real, sin depender de `valueChanges` o suscripciones manuales.  
Esto abre la puerta a patrones de validación más expresivos, pero también plantea nuevos retos a la hora de probarlos.

Cuando un formulario depende de señales, el test no solo debe verificar el valor final, sino también **cómo evoluciona el estado** a medida que cambian las condiciones. Esto es especialmente relevante en validaciones dinámicas: reglas que se activan o desactivan según otros campos, datos externos o incluso el rol del usuario.

Imaginemos un escenario: un formulario de registro donde el campo *"Código de invitación"* solo es obligatorio si el usuario selecciona el rol *"Beta Tester"*.  
En un enfoque tradicional, esto implicaría suscribirse a cambios y añadir o quitar validadores manualmente. Con Signals, podemos expresar esta lógica de forma declarativa y reactiva.

```ts
import { Component, signal, computed } from '@angular/core';
import { FormControl, FormGroup, Validators, NonNullableFormBuilder } from '@angular/forms';

@Component({
  selector: 'app-register-form',
  standalone: true,
  template: `
    <form [formGroup]="form">
      <label>Rol:
        <select formControlName="role">
          <option value="user">Usuario</option>
          <option value="beta">Beta Tester</option>
        </select>
      </label>

      <label>Código de invitación:
        <input type="text" formControlName="inviteCode">
      </label>

      <p *ngIf="inviteCodeRequired()">Este campo es obligatorio para Beta Testers</p>
    </form>
  `
})
export class RegisterFormComponent {
  form: FormGroup;
  role = signal<'user' | 'beta'>('user');
  inviteCodeRequired = computed(() => this.role() === 'beta');

  constructor(private fb: NonNullableFormBuilder) {
    this.form = this.fb.group({
      role: this.fb.control(this.role()),
      inviteCode: this.fb.control('')
    });

    // Vinculamos la señal con el control
    this.form.controls.role.valueChanges.subscribe(value => this.role.set(value));

    // Validación dinámica reactiva
    this.inviteCodeRequired.subscribe(required => {
      const ctrl = this.form.controls.inviteCode;
      ctrl.clearValidators();
      if (required) {
        ctrl.addValidators(Validators.required);
      }
      ctrl.updateValueAndValidity({ emitEvent: false });
    });
  }
}
```

Probar este tipo de formularios requiere algo más que verificar un valor final. Queremos asegurarnos de que:

- La validación se activa y desactiva en el momento correcto.
- El mensaje de error aparece o desaparece según el estado.
- El formulario refleja los cambios de la señal sin necesidad de interacción manual.

Un test bien planteado podría verse así:

```ts
import { TestBed } from '@angular/core/testing';
import { RegisterFormComponent } from './register-form.component';
import { ReactiveFormsModule } from '@angular/forms';

describe('RegisterFormComponent con Signals y validaciones dinámicas', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [ReactiveFormsModule, RegisterFormComponent]
    });
  });

  it('debe exigir código de invitación solo para Beta Testers', () => {
    const fixture = TestBed.createComponent(RegisterFormComponent);
    fixture.detectChanges();

    const comp = fixture.componentInstance;
    const inviteCtrl = comp.form.controls.inviteCode;

    // Estado inicial: no requerido
    expect(inviteCtrl.hasError('required')).toBeFalse();

    // Cambiamos el rol a beta
    comp.role.set('beta');
    fixture.detectChanges();

    expect(inviteCtrl.hasError('required')).toBeTrue();

    // Introducimos un valor válido
    inviteCtrl.setValue('ABC123');
    fixture.detectChanges();

    expect(inviteCtrl.valid).toBeTrue();

    // Volvemos a rol usuario
    comp.role.set('user');
    fixture.detectChanges();

    expect(inviteCtrl.hasError('required')).toBeFalse();
  });
});
```

Lo interesante aquí es que no necesitamos simular eventos de cambio en el `<select>` para probar la lógica: al manipular directamente la señal `role`, el formulario reacciona igual que lo haría en producción. Esto hace que las pruebas sean más **predecibles** y **rápidas**, porque eliminamos la dependencia de la capa de DOM cuando no es necesaria.

Además, este patrón nos permite aislar la validación dinámica y probarla como una unidad independiente. Podríamos incluso extraer la lógica de validación a un servicio con Signals y mockearlo en otros tests, manteniendo la coherencia en toda la aplicación.

En validaciones más complejas —por ejemplo, comprobaciones asíncronas contra un backend—, las Signals siguen siendo útiles: podemos simular la respuesta del servidor actualizando la señal que representa el estado de validación, sin necesidad de esperar a que se resuelva un `Observable`. Esto reduce drásticamente el tiempo de ejecución de las pruebas y nos da control total sobre los escenarios.

En resumen, probar formularios basados en Signals no es solo cuestión de verificar valores: es validar **transiciones de estado** y **reacciones inmediatas**. Y cuando las validaciones son dinámicas, este enfoque nos permite cubrir casos que antes requerían tests más frágiles y lentos.


## 10.6. Estrategias de testing en aplicaciones con SSR e Hydration

Probar una aplicación Angular que utiliza **Server-Side Rendering (SSR)** y **Hydration** no es lo mismo que probar una SPA tradicional.  
En una SPA, el navegador construye la interfaz desde cero en el cliente. En SSR, la primera vista llega ya renderizada desde el servidor, y la **hidratación** es el proceso que conecta ese HTML estático con el código interactivo del cliente, sin volver a renderizarlo por completo.

Esto introduce un matiz importante: **hay dos fases críticas que debemos validar**.  
Primero, que el HTML generado en el servidor sea correcto y contenga todo lo necesario para que el usuario vea la página incluso antes de que el JavaScript se ejecute.  
Segundo, que la hidratación en el cliente se realice sin errores y que la aplicación quede plenamente interactiva.

### El reto de las pruebas en SSR + Hydration

En un entorno SSR, los fallos pueden aparecer en lugares que no existen en una SPA pura:

- **Divergencias entre servidor y cliente**: si el HTML inicial no coincide con lo que Angular espera en el cliente, la hidratación puede fallar o provocar un re-render completo.
- **Acceso a APIs del navegador en el servidor**: llamadas a `window`, `document` o APIs de DOM que no existen en Node.js.
- **Datos iniciales inconsistentes**: si el servidor y el cliente no comparten la misma fuente de datos o el mismo estado inicial, el usuario puede ver un “parpadeo” o contenido duplicado.
- **Comportamiento condicional**: lógica que depende del entorno (`isPlatformBrowser` / `isPlatformServer`) y que puede comportarse de forma distinta en cada fase.

### Cómo enfocar el testing

En lugar de pensar en “un test para todo”, conviene separar las pruebas en **tres capas complementarias**:

1. **Pruebas de renderizado en servidor**  
   Ejecutar el renderizado SSR en un entorno de test (Node) y verificar que el HTML resultante contiene lo esperado: metadatos, contenido crítico, atributos de accesibilidad, datos precargados.  
   Aquí no hay interacción, solo validación del HTML inicial.

2. **Pruebas de hidratación en cliente**  
   Simular la carga de la aplicación en un navegador real o en un entorno como JSDOM, partiendo del HTML generado por el servidor. El objetivo es detectar errores de hidratación y asegurar que los componentes quedan interactivos sin re-render innecesario.

3. **Pruebas de integración end-to-end**  
   Lanzar la aplicación completa con SSR y navegar como lo haría un usuario real, validando que la experiencia es fluida desde la primera pintura hasta la interacción final.

### Un ejemplo de validación de HTML SSR

Podemos montar un test que invoque el motor de SSR y analice el HTML resultante:

```ts
import { renderApplication } from '@angular/platform-server';
import { AppComponent } from './app/app.component';

describe('SSR Render', () => {
  it('debe generar HTML con el título correcto', async () => {
    const html = await renderApplication(AppComponent, {
      appId: 'serverApp',
      document: '<app-root></app-root>',
      url: '/'
    });

    expect(html).toContain('<title>Mi Aplicación</title>');
    expect(html).toContain('Bienvenido');
  });
});
```

Este tipo de prueba nos da confianza en que el servidor está entregando contenido útil incluso antes de que el cliente lo hidrate.

### Detectando problemas de hidratación

Para probar la hidratación, necesitamos partir del HTML SSR y cargarlo en un entorno que simule el navegador. Con herramientas como **Playwright** o **Puppeteer**, podemos:

- Cargar la página con JavaScript deshabilitado y verificar que el contenido inicial es correcto.
- Activar JavaScript y comprobar que los elementos se vuelven interactivos sin cambios visuales bruscos.
- Capturar errores de consola durante la hidratación (indicadores de divergencias).

Ejemplo conceptual con Playwright:

```ts
import { test, expect } from '@playwright/test';

test('hidratación sin errores', async ({ page }) => {
  page.on('console', msg => {
    if (msg.type() === 'error') throw new Error(`Error en consola: ${msg.text()}`);
  });

  await page.goto('http://localhost:4000', { waitUntil: 'domcontentloaded' });

  // Verificar contenido inicial
  const title = await page.textContent('h1');
  expect(title).toBe('Bienvenido');

  // Interactuar y comprobar que responde
  await page.click('button#increment');
  const value = await page.textContent('#counter');
  expect(value).toBe('1');
});
```

### Validaciones específicas para SSR + Hydration

- **Integridad del HTML inicial**: que incluya datos críticos, atributos ARIA, metadatos y contenido visible.
- **Ausencia de parpadeos**: el contenido no debe cambiar drásticamente tras la hidratación.
- **Sin errores de consola**: divergencias de HTML o problemas de serialización suelen aparecer aquí.
- **Datos precargados coherentes**: si usas resolvers o inyección de datos en SSR, deben coincidir con lo que el cliente espera.
- **Compatibilidad con entornos sin JS**: aunque no sea un requisito funcional, es un buen indicador de robustez.

### Consejos prácticos

- **Aísla la lógica dependiente de plataforma**: usa `isPlatformBrowser` y `isPlatformServer` para evitar errores en SSR.
- **Prueba rutas críticas**: páginas de inicio, landing pages y cualquier vista indexada por buscadores.
- **Automatiza en CI**: ejecuta tests SSR y de hidratación en cada build para detectar regresiones pronto.
- **Mide métricas de experiencia**: LCP (Largest Contentful Paint) y TTI (Time to Interactive) son especialmente relevantes en SSR.

En definitiva, el testing en aplicaciones con SSR e Hydration no se limita a “ver si funciona”: se trata de **garantizar que la experiencia inicial del usuario es rápida, estable y sin sorpresas**.  
Separar las pruebas por fases y usar herramientas adecuadas para cada una nos permite cubrir desde la generación del HTML hasta la interacción final, asegurando que el salto del servidor al cliente sea invisible para quien usa la aplicación.


## 10.7. End-to-End testing (e2e) con Cypress: casos de uso y configuración

Las pruebas **End-to-End** son el último eslabón de la cadena de testing: simulan la experiencia de un usuario real, recorriendo la aplicación desde la primera carga hasta la interacción final.  
En Angular 20, combinadas con herramientas modernas como **Cypress**, se convierten en una forma poderosa de validar que todo —desde el frontend hasta el backend— funciona como se espera.

Cypress ha ganado popularidad porque rompe con la complejidad de los frameworks E2E tradicionales: no requiere Selenium, se ejecuta directamente en el navegador, ofrece una API muy expresiva y un panel visual que permite ver cada paso de la prueba como si fuera un vídeo.

### ¿Por qué Cypress?

Más allá de la velocidad y la facilidad de uso, Cypress ofrece algo que los desarrolladores valoran mucho: **confianza visual**.  
Mientras la prueba se ejecuta, puedes ver la aplicación en tiempo real, inspeccionar el DOM en cada paso y entender exactamente qué ocurrió antes de un fallo. Esto reduce el tiempo de diagnóstico y evita la sensación de “caja negra” que a veces tienen otras herramientas.

Además, Cypress integra de forma natural:

- **Esperas automáticas**: no necesitas añadir `sleep()` o `wait()` para que los elementos aparezcan; Cypress espera de forma inteligente.
- **Snapshots y timetravel**: puedes retroceder a cualquier paso y ver el estado de la aplicación en ese momento.
- **Mocks y stubs**: intercepta peticiones HTTP para simular respuestas del servidor.
- **Integración con CI/CD**: ejecuta las pruebas en pipelines sin configuración complicada.

### Casos de uso en Angular 20

En una aplicación Angular moderna, Cypress puede cubrir escenarios como:

- **Flujos críticos de negocio**: registro de usuario, login, compra, checkout.
- **Validaciones de UI**: que los elementos aparezcan y se comporten como se espera.
- **Integración con backend**: verificar que las llamadas HTTP se realizan y responden correctamente.
- **Pruebas de accesibilidad**: usando librerías como `cypress-axe` para detectar problemas.
- **SSR + Hydration**: comprobar que la aplicación renderizada en servidor se hidrata sin errores y es interactiva desde el primer momento.
- **Formularios complejos**: validar que las reglas dinámicas y las señales se aplican correctamente en un entorno real.

### Configuración básica de Cypress en Angular 20

Aunque Angular CLI incluye soporte para Protractor (ya en desuso), la integración con Cypress es directa y más sencilla.

1. **Instalar Cypress**  
   ```bash
   npm install --save-dev cypress
   ```

2. **Inicializar configuración**  
   ```bash
   npx cypress open
   ```
   Esto crea la carpeta `cypress/` con ejemplos y un archivo `cypress.config.ts` (o `.js`).

3. **Configurar baseUrl**  
   En `cypress.config.ts`:
   ```ts
   import { defineConfig } from 'cypress';

   export default defineConfig({
     e2e: {
       baseUrl: 'http://localhost:4200',
       supportFile: 'cypress/support/e2e.ts'
     }
   });
   ```

4. **Añadir script en `package.json`**  
   ```json
   {
     "scripts": {
       "e2e": "cypress open",
       "e2e:run": "cypress run"
     }
   }
   ```

5. **Ejecutar la aplicación y Cypress**  
   En dos terminales:
   ```bash
   npm start
   npm run e2e
   ```

### Un ejemplo de prueba E2E

Supongamos que queremos validar el flujo de login:

```ts
describe('Flujo de login', () => {
  it('debe permitir iniciar sesión con credenciales válidas', () => {
    cy.visit('/login');

    cy.get('input[name="email"]').type('usuario@example.com');
    cy.get('input[name="password"]').type('123456');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Bienvenido').should('be.visible');
  });

  it('debe mostrar error con credenciales inválidas', () => {
    cy.visit('/login');

    cy.get('input[name="email"]').type('usuario@example.com');
    cy.get('input[name="password"]').type('wrongpass');
    cy.get('button[type="submit"]').click();

    cy.contains('Credenciales incorrectas').should('be.visible');
  });
});
```

Aquí vemos cómo Cypress interactúa con la aplicación como lo haría un usuario: escribe en campos, hace clic en botones y verifica que la URL y el contenido cambian según lo esperado.

### Consejos para un E2E eficaz

- **Prioriza flujos críticos**: no intentes cubrir todo; céntrate en lo que rompería la aplicación si fallara.
- **Usa datos de prueba controlados**: evita depender de datos reales que puedan cambiar.
- **Intercepta peticiones HTTP**: simula respuestas para escenarios difíciles de reproducir.
- **Mantén las pruebas independientes**: cada test debe poder ejecutarse solo.
- **Integra en CI/CD**: ejecuta E2E en cada despliegue para detectar regresiones.

### Más allá de lo básico

Cypress permite combinar pruebas E2E con validaciones de rendimiento, accesibilidad y seguridad.  
Por ejemplo, puedes integrar `cypress-audit` para medir métricas de Lighthouse, o `cypress-axe` para comprobar accesibilidad automáticamente en cada vista.

En aplicaciones Angular con SSR, puedes usar Cypress para cargar la página con JavaScript deshabilitado, validar el HTML inicial y luego habilitarlo para comprobar la hidratación, uniendo así lo aprendido en la sección anterior.

En definitiva, Cypress no es solo una herramienta para “hacer clics” en la aplicación: es un entorno de pruebas que te permite **ver, entender y controlar** cada paso del flujo de usuario.  
En Angular 20, su integración es directa y su potencial enorme, especialmente si lo combinas con las estrategias de mocking, Signals y Harnesses que ya hemos visto.


## 10.8. Integración de Jest + Cypress en pipelines de CI/CD

En un proyecto Angular 20 bien cuidado, las pruebas no deberían ser algo que se ejecuta “cuando nos acordamos”, sino una parte natural y obligatoria del ciclo de entrega.  
La combinación de **Jest** (para unitarias e integración) y **Cypress** (para E2E) cubre prácticamente todo el espectro de testing, y llevar ambas a un **pipeline de CI/CD** garantiza que cada cambio pase por un filtro de calidad antes de llegar a producción.

Integrar estas herramientas en un pipeline no es solo cuestión de “poner comandos en un YAML”: implica pensar en **cuándo** y **cómo** se ejecutan, **qué feedback** queremos obtener y **cómo optimizar tiempos** para no ralentizar el flujo de desarrollo.

### El flujo ideal

En un pipeline típico, las pruebas se distribuyen en fases:

1. **Instalación y build**  
   - Se instala el proyecto y sus dependencias.
   - Se compila para asegurar que no hay errores de tipado o build rotos.

2. **Pruebas unitarias e integración (Jest)**  
   - Rápidas, se ejecutan en Node.
   - Validan la lógica interna y la interacción entre piezas pequeñas.
   - Dan feedback inmediato: si algo falla aquí, no tiene sentido seguir.

3. **Pruebas E2E (Cypress)**  
   - Más lentas, pero validan el flujo real de usuario.
   - Se ejecutan contra una instancia real de la aplicación (SSR o SPA).
   - Pueden correr en paralelo o en entornos dedicados para no bloquear.

4. **Reporte y artefactos**  
   - Resultados de cobertura de Jest.
   - Capturas de pantalla y vídeos de Cypress en caso de fallo.
   - Publicación de reportes en el sistema de CI.

### Ejemplo con GitHub Actions

Un pipeline sencillo que combina Jest y Cypress podría verse así:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout código
        uses: actions/checkout@v4

      - name: Configurar Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Instalar dependencias
        run: npm ci

      - name: Compilar aplicación
        run: npm run build --if-present

      - name: Ejecutar pruebas unitarias (Jest)
        run: npm run test -- --ci --coverage

      - name: Iniciar servidor en segundo plano
        run: npm start &
      
      - name: Esperar a que el servidor esté listo
        run: npx wait-on http://localhost:4200

      - name: Ejecutar pruebas E2E (Cypress)
        run: npm run e2e:run
```

Este flujo:

- **Se detiene rápido** si Jest falla, ahorrando tiempo y recursos.
- **Levanta la app** solo si las unitarias pasan.
- **Usa `wait-on`** para asegurarse de que Cypress no arranca antes de que la app esté lista.
- **Genera cobertura** con Jest y puede subirla a servicios como Codecov o SonarQube.

### Optimización y buenas prácticas

- **Paralelizar**: en pipelines más grandes, separar Jest y Cypress en jobs distintos que corran en paralelo.
- **Caché de dependencias**: usar cache de `node_modules` o de `npm ci` para acelerar builds.
- **Datos de prueba controlados**: en E2E, usar entornos o bases de datos específicas para evitar interferencias.
- **Ejecución selectiva**: correr Cypress solo en ramas principales o antes de un despliegue, y Jest en cada commit/pull request.
- **Reportes legibles**: integrar con Slack, Teams o email para avisar de fallos con enlaces a logs y capturas.

### Integración con otros servicios

- **Code coverage**: subir la cobertura de Jest a Codecov o SonarQube para seguimiento histórico.
- **Dashboard de Cypress**: usar el servicio oficial para ver vídeos, capturas y métricas de ejecución.
- **Análisis de performance**: añadir pasos opcionales con Lighthouse o cypress-audit para medir métricas clave.

En resumen, la integración de Jest y Cypress en CI/CD no es solo una cuestión técnica: es una **decisión estratégica** que convierte las pruebas en un guardián automático de la calidad.  
Cuando el pipeline está bien diseñado, cada commit es una oportunidad de validar que la aplicación sigue funcionando como debe, sin sorpresas en producción.


## 10.8. Buenas prácticas para asegurar cobertura y fiabilidad en proyectos enterprise

En entornos Enterprise, el testing deja de ser una actividad aislada y se convierte en un **pilar de gobernanza técnica**.  
Aquí no basta con “que los tests pasen”: necesitamos que sean **representativos**, **consistentes** y que se ejecuten de forma **predecible** en cualquier entorno, desde el portátil de un desarrollador hasta el pipeline de producción.

La cobertura y la fiabilidad son dos caras de la misma moneda:  
- **Cobertura**: qué porcentaje del código está protegido por pruebas.  
- **Fiabilidad**: que esas pruebas realmente detecten errores y no den falsos positivos o negativos.

### Pensar en cobertura como un indicador, no como un objetivo ciego

En proyectos grandes, es habitual fijar umbrales de cobertura (por ejemplo, 80% global, 100% en módulos críticos).  
Pero perseguir el número sin criterio puede llevar a tests triviales que no aportan valor.

**Enfoque recomendado:**
- Identificar **zonas críticas** (procesos de negocio, seguridad, integraciones externas) y exigir cobertura máxima ahí.
- Aceptar menor cobertura en código de bajo riesgo o altamente volátil, para evitar tests frágiles.
- Revisar la cobertura por **líneas, ramas y funciones**: un 90% de líneas cubiertas no significa que todas las rutas lógicas estén probadas.

### Fiabilidad: el enemigo son los tests frágiles

Un test frágil es aquel que falla por cambios irrelevantes o por condiciones externas no controladas.  
En un entorno Enterprise, esto es costoso: bloquea pipelines, genera ruido y erosiona la confianza del equipo.

**Cómo evitarlos:**
- Usar **Harnesses** para desacoplar la prueba de la estructura interna del componente.
- Mockear dependencias externas y controlar datos de prueba.
- Evitar dependencias en el orden de ejecución o en el estado dejado por otros tests.
- Simular tiempos y respuestas en lugar de depender de APIs reales.

### Integrar la pirámide de testing en la arquitectura

En proyectos grandes, la **pirámide de testing** no es solo teoría: es una herramienta de planificación.

- **Base amplia**: pruebas unitarias con Jest, rápidas y abundantes.
- **Capa intermedia**: pruebas de integración que validen módulos completos.
- **Cima**: E2E con Cypress, centradas en flujos críticos.

Esto asegura que la mayoría de los errores se detecten en fases rápidas y baratas, y que las pruebas más costosas se reserven para validar la experiencia completa.

### Automatización y control de calidad continuo

- **CI/CD obligatorio**: ningún cambio se integra sin pasar por el pipeline de pruebas.
- **Ejecución paralela**: separar unitarias e integración de E2E para optimizar tiempos.
- **Reportes automáticos**: cobertura, métricas de rendimiento y accesibilidad en cada build.
- **Alertas tempranas**: notificaciones en Slack/Teams cuando un test crítico falla.

### Cultura de testing en el equipo

La fiabilidad no se consigue solo con herramientas: requiere una **cultura compartida**.

- Revisar tests en code reviews con el mismo rigor que el código de producción.
- Documentar patrones de prueba y ejemplos de mocks, Harnesses y uso de Signals.
- Formar a nuevos miembros en las prácticas y estándares del equipo.
- Mantener una **biblioteca de utilidades de testing** para evitar duplicación y fomentar consistencia.

### Checklist para proyectos Enterprise

1. **Cobertura mínima definida** y revisada por módulo.
2. **Tests críticos etiquetados** para ejecución prioritaria.
3. **Mocks centralizados** y reutilizables.
4. **Harnesses** para componentes complejos.
5. **Validaciones de accesibilidad** integradas en E2E.
6. **Pipeline con gates** que bloqueen despliegues si fallan pruebas clave.
7. **Revisión periódica** de la suite para eliminar tests obsoletos o redundantes.

En un proyecto Enterprise, la meta no es solo “tener tests”, sino **tener confianza** en que esos tests reflejan la realidad y protegen el negocio.  
Cuando la cobertura está bien distribuida y la fiabilidad es alta, el equipo puede desplegar con seguridad, incluso en ciclos rápidos, sabiendo que el sistema de pruebas actúa como un escudo contra errores y regresiones.


