# Actividad 20. Clases en JavaScript

## Temas

- class
    
- constructor
    
- métodos
    

## Ejercicio

Crear en JavaScript las mismas clases existentes en C++.

```text
Persona

Alumno

Maestro

Administrador
```

Crear objetos desde el navegador.

Comparar:

```cpp
Alumno alumno(...);
```

vs

```javascript
const alumno = new Alumno(...);
```

Mostrar cómo ambos lenguajes implementan prácticamente los mismos conceptos de POO, aunque con una sintaxis diferente.

---
# TEORÍA

---

## 1. ¿Qué son las clases en JavaScript?

- **Definición:** Una clase es una **plantilla** para crear objetos con atributos (propiedades) y comportamientos (métodos). Introducidas en ES6 (2015) como **azúcar sintáctico** sobre el sistema de prototipos que JS ya tenía.
- **Diferencia con C++:** En C++, las clases son tipos estáticos definidos en tiempo de compilación. En JS, las clases son **funciones especiales** que se evalúan en tiempo de ejecución; son "objetos" ellos mismos.
- **Paralelismo:** Los conceptos son los mismos: clase, objeto, atributos, métodos, constructor, herencia, polimorfismo. Cambia la sintaxis y algunas reglas.
- **En esta actividad:** Replicarás `Persona`, `Alumno`, `Maestro`, `Administrador` en JS para ver cómo ambos lenguajes expresan la misma POO.

---

## 2. Sintaxis básica de una clase

- **Declaración:**
  ```js
  class NombreClase {
      // cuerpo
  }
  ```
- **Convención:** Nombre en **PascalCase** (igual que en C++).
- **No lleva `;` al final** (a diferencia de C++).
- **Las clases no se "hoistean"** como las funciones: debes declararlas **antes** de usarlas.

---

## 3. El constructor

- **Definición:** Método especial que se ejecuta automáticamente al crear un objeto con `new`.
- **Nombre fijo:** Se llama **`constructor`** (no el nombre de la clase, a diferencia de C++).
- **Solo puede haber uno por clase** (no hay sobrecarga de constructores como en C++).
- **Sin tipo de retorno** (ni siquiera se declara).
- **Parámetros:** Puede recibir tantos como necesites; puedes usar valores por defecto.
- **Inicialización de atributos:** Dentro del constructor se asignan las propiedades del objeto usando `this`.

**Fragmento suelto:**
```js
class Persona {
    constructor(nombre, edad) {
        this.nombre = nombre;
        this.edad = edad;
    }
}
```

- **`this`:** Referencia al objeto que se está construyendo. En C++ el equivalente sería el objeto implícito al que accedes directamente por nombre de atributo.

### Diferencia con C++

| Aspecto | C++ | JavaScript |
|---------|-----|------------|
| Nombre | Igual que la clase | `constructor` |
| Sobrecarga | Sí (varios constructores) | No (solo uno, con valores por defecto) |
| Lista de inicialización | Sí (`: atributo(valor)`) | No, se asigna en el cuerpo |
| Llamada a la base | En lista de inicialización | Con `super(...)` al inicio |

---

## 4. Propiedades (atributos)

- **Declaración:** En JS moderno, se declaran **dentro del constructor** con `this.propiedad = valor`.
- **Declaración de campos de clase (ES2022):** También puedes declarar propiedades directamente en el cuerpo de la clase, fuera del constructor:
  ```js
  class Persona {
      nombre = "";     // campo de clase con valor por defecto
      edad = 0;
  }
  ```
- **Sin modificadores de acceso:** JS **no tiene `public`, `private`, `protected`** de forma tradicional. Existen:
  - **Campos privados (ES2022):** Se prefijan con `#` (ej. `#nombre`). Solo accesibles desde dentro de la clase.
  - **Convención `_nombre`:** Prefijo guion bajo para indicar "privado" (no lo hace privado realmente, es solo convención).
- **Diferencia con C++:** En C++ declaras atributos con tipo y visibilidad. En JS no hay tipos ni visibilidad real (excepto campos privados con `#`).

**Fragmento suelto (campo privado):**
```js
class Alumno {
    #matricula;  // campo privado
    constructor(matricula) {
        this.#matricula = matricula;
    }
}
```

---

## 5. Métodos

- **Definición:** Funciones definidas dentro de la clase que operan sobre el objeto (`this`).
- **Sintaxis:** Se declaran con nombre y parámetros, **sin la palabra `function`**.
  ```js
  class Persona {
      mostrar() {
          console.log(this.nombre);
      }
  }
  ```
- **No llevan tipo de retorno** ni modificadores de acceso.
- **Métodos estáticos:** Se prefijan con `static`. Pertenecen a la clase, no a las instancias.
  ```js
  static crearDesdeTexto(texto) { ... }
  ```
- **Getters y setters:** Se declaran con `get` y `set` delante del nombre.
  ```js
  get nombre() { return this._nombre; }
  set nombre(valor) { this._nombre = valor; }
  ```
  Se usan **como propiedades**, no como métodos: `obj.nombre` en lugar de `obj.nombre()`.
- **`this` en métodos:** Apunta al objeto sobre el que se llama. Si pasas un método como callback, `this` puede perderse (se soluciona con arrow functions o `.bind`).

---

## 6. Creación de objetos

- **Sintaxis:** `const obj = new Clase(args);`
- **Diferencia con C++:**
  - **C++:** `Alumno alumno("Ana", 20);` (objeto en el stack).
  - **C++ dinámico:** `Alumno* alumno = new Alumno("Ana", 20);` (objeto en el heap, requiere `delete`).
  - **JS:** `const alumno = new Alumno("Ana", 20);` → siempre crea un objeto en memoria gestionada automáticamente (no hay `delete`).
- **En JS no hay distinción stack/heap** visible para el programador. El recolector de basura libera la memoria automáticamente.
- **Todo objeto en JS es una referencia.** Asignar un objeto a otra variable **no copia**; ambas apuntan al mismo objeto. Para copiar, hay que hacerlo explícitamente.
  ```js
  const a = new Alumno(...);
  const b = a;         // b y a apuntan al mismo objeto
  b.nombre = "Otro";   // afecta también a a
  ```

---

## 7. Herencia

- **Sintaxis:** `class Derivada extends Base { ... }`
- **`super`:** Se usa para llamar al constructor de la base. **Obligatorio** llamarlo antes de usar `this` en el constructor derivado.
  ```js
  class Alumno extends Persona {
      constructor(nombre, edad, matricula) {
          super(nombre, edad);  // llama al constructor de Persona
          this.matricula = matricula;
      }
  }
  ```
- **`super.metodo()`:** También se puede usar para llamar a métodos de la base.
- **Sobrescritura:** La derivada puede redefinir métodos de la base con el mismo nombre.
- **`extends` equivale a la herencia pública de C++** (`: public Base`).
- **Diferencia con C++:** En JS no hay herencia múltiple (solo una clase base). En C++ sí existe.

---

## 8. Polimorfismo

- **Definición:** Un mismo método puede comportarse diferente según el tipo real del objeto.
- **En JS:** Es **automático**. No hace falta declarar `virtual`. Si una clase derivada sobrescribe un método, al llamarlo sobre una instancia de la derivada se ejecuta el de la derivada.
- **Diferencia con C++:** En C++ debes declarar `virtual` en la base y usar punteros/referencias. En JS **todo método es polimórfico por defecto** y **todo objeto es por referencia**.
- **Sin `override`:** No existe la palabra clave. Simplemente redefines el método con el mismo nombre en la clase derivada.
- **Acceso a la versión de la base:** Con `super.metodo()`.

---

## 9. `class` vs. función constructora (contexto histórico)

- **Antes de ES6:** Se usaban funciones constructoras y el objeto `prototype` para simular clases.
  ```js
  function Persona(nombre) { this.nombre = nombre; }
  Persona.prototype.mostrar = function() { ... };
  ```
- **Desde ES6:** La sintaxis `class` es más clara y familiar para quienes vienen de lenguajes como C++ o Java.
- **Internamente:** Sigue siendo el mismo sistema de prototipos; `class` es azúcar sintáctico.
- **Recomendación:** Usa `class` en proyectos modernos.

---

## 10. Comparación directa C++ ↔ JavaScript

| Concepto | C++ | JavaScript |
|----------|-----|------------|
| Declaración | `class Alumno { ... };` | `class Alumno { ... }` |
| Constructor | Nombre = clase, sin retorno | Método `constructor(...)` |
| Sobrecarga de constructores | Sí | No (valores por defecto) |
| Atributos | Declarados con tipo | Asignados con `this.x = ...` |
| Visibilidad | `public`, `private`, `protected` | Solo `#` para privados |
| Métodos | `tipo nombre(params) { }` | `nombre(params) { }` |
| `this` | Puntero implícito al objeto | Referencia al objeto |
| Creación | `Alumno a(...);` o `new Alumno(...)` | `new Alumno(...)` |
| Herencia | `class D : public B` | `class D extends B` |
| Llamada a base | Lista de inicialización | `super(...)` |
| Polimorfismo | `virtual` + punteros/referencias | Automático |
| Destructor | `~Clase()` | No hay (recolector de basura) |
| Memoria | Manual (`new`/`delete`) | Automática (GC) |
| Copia de objetos | Copia por valor por defecto | Copia por referencia |
| Tipado | Estático y fuerte | Dinámico y débil |
| Sobrecarga de operadores | Sí | No |

---

## 11. Ciclo de vida de un objeto en JS

- **Creación:** `new Clase(...)` → se ejecuta el constructor.
- **Uso:** Acceso a propiedades y métodos.
- **Destrucción:** No hay destructor explícito. El **recolector de basura** libera la memoria cuando **ya no hay referencias** al objeto.
- **Diferencia con C++:** En C++ el destructor se llama determinísticamente al salir del ámbito o al hacer `delete`. En JS no puedes saber cuándo se libera la memoria.
- **Implicación práctica:** Si quieres "limpiar" algo al eliminar un objeto, no puedes confiar en un destructor. Debes hacerlo manualmente (por ejemplo, con un método `destroy()` que llames tú).
- **Mensajes de traza:** Puedes imprimir "objeto creado" en el constructor, pero no puedes imprimir "objeto destruido" de forma fiable.

---

## 12. Buenas prácticas

- **Nombres de clase en PascalCase** (`Alumno`, `Administrador`).
- **Nombres de métodos y propiedades en camelCase** (`mostrarInfo`, `nombreCompleto`).
- **Usa campos privados (`#`)** para encapsular datos sensibles.
- **Usa getters y setters** para validaciones y control de acceso.
- **Usa `const` para variables que no reasignas** (aunque el objeto en sí pueda modificarse).
- **No abuses de la herencia:** prefiere composición cuando no haya relación "es un".
- **Llama a `super(...)` siempre** en el constructor de la derivada, antes de usar `this`.
- **Documenta cada clase** con un comentario que describa su propósito y sus métodos principales.
- **Evita modificar prototipos de clases nativas** (Array, Object, etc.).
- **No uses `var`** dentro de clases; usa `let` o `const` si necesitas variables temporales.
- **Separa responsabilidades:** una clase, una tarea.

---

## 13. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Usar la clase antes de declararla** | JS no hoistea clases; declara primero. |
| **Olvidar `new` al crear el objeto** | Sin `new`, el constructor no se ejecuta correctamente. |
| **Usar `this` en el constructor derivado antes de `super`** | Error en tiempo de ejecución. Llama a `super(...)` primero. |
| **No llamar a `super` en la derivada** | Si el constructor derivado no llama a `super`, error. |
| **Confundir propiedad con método** | `obj.nombre` (propiedad) vs. `obj.mostrar()` (método). |
| **Perder `this` en callbacks** | Usa arrow functions o `.bind(this)`. |
| **Modificar un objeto copiado pensando que es independiente** | En JS los objetos se comparten por referencia. Copia explícitamente si necesitas independencia. |
| **Esperar destructores** | JS no los tiene. Usa el recolector de basura y limpia manualmente si es necesario. |
| **Declarar atributos con tipo** | En JS no se declaran tipos. Solo `this.x = valor`. |
| **Usar `==` para comparar objetos** | `==` compara referencias, no contenido. Para comparar contenido, define un método o compara campo por campo. |

---

## 14. Reflexión: mismos conceptos, distinta sintaxis

- **Clase:** Plantilla para crear objetos. Igual en ambos lenguajes.
- **Objeto:** Instancia concreta. En C++ puede vivir en stack o heap; en JS siempre en memoria gestionada por GC.
- **Constructor:** Inicializa el objeto. C++ tiene sobrecarga; JS solo uno con valores por defecto.
- **Métodos:** Comportamiento del objeto. Sintaxis casi idéntica, sin tipos en JS.
- **Herencia:** `extends` (JS) vs. `: public Base` (C++). Mismo concepto.
- **Polimorfismo:** Automático en JS; explícito con `virtual` en C++.
- **Encapsulación:** `private`/`protected` en C++; `#` en JS (limitado).
- **Ciclo de vida:** Destructores deterministas en C++; recolección de basura no determinista en JS.

**Conclusión:** La POO es un paradigma, no un lenguaje. Los conceptos son los mismos; la sintaxis y las reglas cambian. Comprender ambos te permite elegir la herramienta adecuada y trasladar conocimiento entre lenguajes.

---

## 15. Resumen de conceptos clave

- **`class`:** Azúcar sintáctico sobre prototipos. PascalCase, sin `;` final.
- **`constructor`:** Método especial, uno por clase, sin retorno. Inicializa con `this`.
- **Propiedades:** Se asignan con `this.x = ...`. Pueden ser privadas con `#`.
- **Métodos:** Sin `function`, sin tipos, con `this` implícito.
- **Getters/setters:** Palabras clave `get`/`set`; se usan como propiedades.
- **`new`:** Crea la instancia y ejecuta el constructor.
- **`extends` + `super`:** Herencia y llamada a la base.
- **Polimorfismo:** Automático en JS, sin `virtual`.
- **Sin destructores:** Recolector de basura, no determinista.
- **Objetos por referencia:** Asignar no copia; hay que copiar explícitamente.
- **Mismos conceptos que C++, distinta sintaxis y reglas.**

---
# PASO A PASO

---
# Actividad 20 — Clases en JavaScript (POO)

## Objetivo
Reemplazar los objetos "planos" por instancias de clases reales: `Persona`, `Alumno`, `Maestro`, `Administrador`. Comparar con C++.

## Comparación rápida

| Concepto        | C++                              | JavaScript                  |
|-----------------|----------------------------------|-----------------------------|
| Declaración     | `class Alumno { ... };`          | `class Alumno { ... }`      |
| Constructor     | mismo nombre de la clase         | `constructor(...)`          |
| Herencia        | `class A : public B`             | `class A extends B`         |
| Llamar a base   | lista de inicialización          | `super(...)`                |
| Crear objeto    | `Alumno a("Ana", 20);`           | `const a = new Alumno(...)` |
| Destructor      | sí, determinista                 | no, garbage collector       |
| Polimorfismo    | `virtual` + punteros             | automático                  |

---

## Paso 1 — Clase base `Persona`

Al inicio de `app.js`, **antes** de `let alumnos = []`:

```js
class Persona {
  #id;              // campo privado (ES2022)

  constructor(nombre, edad) {
    this.nombre = nombre;
    this.edad = edad;
    this.#id = Persona.#generarId();
  }

  static #contador = 0;
  static #generarId() {
    return ++Persona.#contador;
  }

  get id() {
    return this.#id;
  }

  // Método polimórfico: las clases hijas lo pueden sobrescribir
  rol() {
    return 'Persona';
  }

  mostrarInfo() {
    return `${this.rol()}: ${this.nombre} (${this.edad} años)`;
  }

  // Método "toString" de JS
  toString() {
    return this.mostrarInfo();
  }
}
```

💡 Fíjate:
- `#id` es privado de verdad (no accesible desde fuera).
- `static #contador` es un contador compartido por todas las instancias.
- `get id()` se usa como propiedad: `persona.id`, no `persona.id()`.

---

## Paso 2 — Clase `Alumno`

Debajo de `Persona`:

```js
class Alumno extends Persona {
  constructor(nombre, edad, matricula, correo) {
    super(nombre, edad);          // ⚠️ antes de usar this
    this.matricula = matricula;
    this.correo = correo;
    this.calificaciones = [];
  }

  rol() {
    return 'Alumno';
  }

  agregarCalificacion(nota) {
    if (typeof nota !== 'number' || nota < 0 || nota > 10) {
      throw new Error('Calificación inválida');
    }
    this.calificaciones.push(nota);
  }

  get promedio() {
    if (this.calificaciones.length === 0) return 0;
    const suma = this.calificaciones.reduce((a, b) => a + b, 0);
    return suma / this.calificaciones.length;
  }

  mostrarInfo() {
    return `${super.mostrarInfo()} — Matrícula: ${this.matricula}`;
  }
}
```

---

## Paso 3 — Clase `Maestro`

```js
class Maestro extends Persona {
  constructor(nombre, edad, empleadoId, materia) {
    super(nombre, edad);
    this.empleadoId = empleadoId;
    this.materia = materia;
  }

  rol() {
    return 'Maestro';
  }

  mostrarInfo() {
    return `${super.mostrarInfo()} — Materia: ${this.materia}`;
  }
}
```

---

## Paso 4 — Clase `Administrador`

```js
class Administrador extends Persona {
  constructor(nombre, edad, departamento) {
    super(nombre, edad);
    this.departamento = departamento;
  }

  rol() {
    return 'Administrador';
  }
}
```

---

## Paso 5 — Probar en la consola

Recarga la página y en la consola del navegador (F12):

```js
const ana   = new Alumno('Ana', 20, 'A001', 'ana@escuela.com');
const luis  = new Maestro('Luis', 40, 'M001', 'Cálculo');
const admin = new Administrador('Sofía', 35, 'Control Escolar');

console.log(ana.mostrarInfo());
console.log(luis.mostrarInfo());
console.log(admin.mostrarInfo());

ana.agregarCalificacion(9);
ana.agregarCalificacion(8);
ana.agregarCalificacion(10);
console.log('Promedio:', ana.promedio.toFixed(2));
```

Salida esperada:

```
Alumno: Ana (20 años) — Matrícula: A001
Maestro: Luis (40 años) — Materia: Cálculo
Administrador: Sofía (35 años)
Promedio: 9.00
```

🎯 **Observa el polimorfismo**: `mostrarInfo()` se comporta distinto según la clase, sin declarar nada especial.

---

## Paso 6 — Equivalencia con C++

| En C++ escribirías...                                    | En JS escribes...                                  |
|----------------------------------------------------------|----------------------------------------------------|
| `class Persona { protected: string nombre; ... };`       | `class Persona { constructor(nombre) {...} }`      |
| `Alumno(string n, int e) : Persona(n) { ... }`           | `constructor(n, e) { super(n); ... }`              |
| `virtual string rol() { return "Alumno"; }`              | `rol() { return "Alumno"; }`                       |
| `Alumno* a = new Alumno("Ana", 20);`                     | `const a = new Alumno("Ana", 20);`                 |
| `delete a;`                                              | (nada: lo libera el GC)                            |

---

## Paso 7 — Error típico y cómo se ve

Prueba esto en la consola para **ver el error**:

```js
const x = Alumno('sin new', 18, 'X', 'x@x.com');
// ❌ TypeError: Class constructor Alumno cannot be invoked without 'new'
```

Y esto otro:

```js
class Hijo extends Persona {
  constructor() {
    this.x = 1;        // ⚠️ antes de super
    super('x', 1);
  }
}
// ❌ ReferenceError: Must call super constructor ... before accessing 'this'
```

**Regla**: en un constructor derivado, `super()` va **primero**.

---

## ✅ Checklist de la Actividad 20

- [ ] Existen las 4 clases: `Persona`, `Alumno`, `Maestro`, `Administrador`.
- [ ] `Alumno`, `Maestro`, `Administrador` heredan de `Persona`.
- [ ] Todas las derivadas llaman a `super(...)` en su constructor.
- [ ] Existe al menos un campo privado con `#`.
- [ ] `rol()` está sobrescrito en cada hija (polimorfismo).
- [ ] Probaste crear objetos en la consola con `new`.

## 🧠 Mini-reto
Agrega una clase `Grupo` que contenga `alumnos = []` y tenga métodos `agregar(alumno)`, `promedioGrupal()`.

# Integración final — Sistema Escolar completo

## Objetivo
Unir las 5 actividades: el formulario crea **instancias de `Alumno`**, la tabla las muestra, se pueden editar/eliminar y el resumen se calcula con los **getters** de las clases.

---

## Paso 1 — Adaptar el modelo

En `app.js`, cambia:

```js
let alumnos = [];
let siguienteId = 1;
```

por:

```js
let alumnos = [];   // ahora contendrá instancias de Alumno
```

Elimina el `siguienteId` (los IDs ya los maneja `Persona` con su contador estático).

---

## Paso 2 — Reemplazar `id` por el getter

En `render()`, cambia:

```js
fila.dataset.id = alumno.id;
```

Esto ya funciona ✅ porque `Alumno` hereda el getter `id` de `Persona`.

En el listener de delegación:

```js
const id = Number(fila.dataset.id);
```

También funciona sin cambios.

---

## Paso 3 — Submit con clase `Alumno`

Reemplaza el listener del submit:

```js
formAlumno.addEventListener('submit', (e) => {
  e.preventDefault();

  const nombre    = document.querySelector('#nombre').value.trim();
  const matricula = document.querySelector('#matricula').value.trim();
  const edad      = Number(document.querySelector('#edad').value);
  const correo    = document.querySelector('#correo').value.trim();
  const c1        = document.querySelector('#cal1').value;
  const c2        = document.querySelector('#cal2').value;
  const c3        = document.querySelector('#cal3').value;

  if (!nombre || !matricula || !edad || !correo) {
    mostrarResultado('Completa todos los campos.', 'error');
    return;
  }

  const nuevoAlumno = new Alumno(nombre, edad, matricula, correo);

  [c1, c2, c3].forEach((valor) => {
    if (valor !== '') {
      try {
        nuevoAlumno.agregarCalificacion(Number(valor));
      } catch (err) {
        mostrarResultado(err.message, 'error');
      }
    }
  });

  alumnos.push(nuevoAlumno);
  render();
  formAlumno.reset();
  mostrarResultado(`Alumno "${nombre}" registrado.`);
});
```

---

## Paso 4 — Columna "Promedio" en la tabla

Cambia el `<thead>` en `index.html`:

```html
<tr>
  <th>Matrícula</th>
  <th>Nombre</th>
  <th>Edad</th>
  <th>Correo</th>
  <th>Promedio</th>
  <th>Acciones</th>
</tr>
```

Y actualiza `render()` para incluir la celda:

```js
fila.innerHTML = `
  <td>${alumno.matricula}</td>
  <td>${alumno.nombre}</td>
  <td>${alumno.edad}</td>
  <td>${alumno.correo}</td>
  <td>${alumno.promedio.toFixed(2)}</td>
  <td class="acciones">
    <button class="btn-editar"   data-accion="editar">Editar</button>
    <button class="btn-eliminar" data-accion="eliminar">Eliminar</button>
  </td>
`;
```

Y actualiza el `colspan` del mensaje de tabla vacía:

```js
filaVacia.innerHTML = `<td colspan="6" class="vacio">Sin alumnos registrados</td>`;
```

---

## Paso 5 — Resumen con promedio general

Reemplaza `actualizarResumen()`:

```js
function actualizarResumen() {
  document.querySelector('#total-alumnos').textContent = alumnos.length;

  if (alumnos.length === 0) {
    document.querySelector('#promedio-general').textContent = '—';
    document.querySelector('#ultimo-alumno').textContent = '—';
    return;
  }

  const sumaPromedios = alumnos.reduce((acc, a) => acc + a.promedio, 0);
  const promedioGeneral = sumaPromedios / alumnos.length;

  document.querySelector('#promedio-general').textContent = promedioGeneral.toFixed(2);
  document.querySelector('#ultimo-alumno').textContent = alumnos[alumnos.length - 1].nombre;
}
```

---

## Paso 6 — Editar con `prompt` (mejorado)

```js
function editarAlumno(id) {
  const alumno = alumnos.find(a => a.id === id);
  if (!alumno) return;

  const nuevoNombre = prompt('Nombre:', alumno.nombre);
  if (nuevoNombre === null) return;

  const nuevaEdad = prompt('Edad:', alumno.edad);
  if (nuevaEdad === null) return;

  const nuevoCorreo = prompt('Correo:', alumno.correo);
  if (nuevoCorreo === null) return;

  if (nuevoNombre.trim()) alumno.nombre = nuevoNombre.trim();
  if (Number(nuevaEdad) > 0) alumno.edad = Number(nuevaEdad);
  if (nuevoCorreo.trim()) alumno.correo = nuevoCorreo.trim();

  render();
  mostrarResultado('Alumno actualizado.');
}
```

---

## Paso 7 — Probar el flujo completo

1. Registra 3 alumnos con calificaciones.
2. Verifica que el "Promedio" de cada uno se calcula.
3. Verifica que el "Promedio general" del resumen coincide.
4. Edita uno: cambia el nombre. Se actualiza la tabla y el resumen.
5. Elimina uno: baja el contador y se recalcula el promedio.
6. Abre la consola y escribe `alumnos[0].mostrarInfo()` → debe funcionar porque es una instancia real de `Alumno`.

---

## Paso 8 — Estructura final del proyecto

```
public/
├── index.html
├── styles.css
├── app.js
└── img/
    └── logo.png
```

Y `app.js` queda organizado así:

```
1. 'use strict';
2. Clases: Persona, Alumno, Maestro, Administrador
3. Modelo: let alumnos = []
4. Utilidades: mostrarResultado()
5. Render: render(), actualizarResumen()
6. CRUD: eliminarAlumno(), editarAlumno()
7. Eventos: submit, delegación en tbody, botones
8. Arranque: render();
```

---

## Paso 9 — Mejora opcional: `createElement` en vez de `innerHTML`

La versión "profesional" (evita XSS y es la buena práctica):

```js
function crearFila(alumno) {
  const fila = document.createElement('tr');
  fila.dataset.id = alumno.id;

  const celdas = [
    alumno.matricula,
    alumno.nombre,
    alumno.edad,
    alumno.correo,
    alumno.promedio.toFixed(2),
  ];

  celdas.forEach(texto => {
    const td = document.createElement('td');
    td.textContent = texto;   // seguro contra XSS
    fila.appendChild(td);
  });

  const tdAcciones = document.createElement('td');
  tdAcciones.className = 'acciones';
  tdAcciones.innerHTML = `
    <button class="btn-editar"   data-accion="editar">Editar</button>
    <button class="btn-eliminar" data-accion="eliminar">Eliminar</button>
  `;
  fila.appendChild(tdAcciones);

  return fila;
}

// Usa dentro de render():
// tbody.appendChild(crearFila(alumno));
```

---

## ✅ Checklist final del proyecto

**HTML**
- [ ] Un solo `<h1>`.
- [ ] Todos los labels asociados con `for`/`id`.
- [ ] `alt` en imágenes.
- [ ] `name` en todos los inputs (para el futuro backend).

**CSS**
- [ ] Variables CSS en `:root`.
- [ ] `box-sizing: border-box` global.
- [ ] Layout con Grid (`.layout`).
- [ ] Media query para móvil.

**JS**
- [ ] Clases con herencia y `super()`.
- [ ] `addEventListener` en todos los eventos (nada de `onclick=` en HTML).
- [ ] `===` en comparaciones.
- [ ] Delegación de eventos en `tbody`.
- [ ] Modelo como fuente de verdad.

**Comportamiento**
- [ ] Registrar, editar, eliminar funcionan.
- [ ] El promedio se calcula con el getter de `Alumno`.
- [ ] El resumen se sincroniza.
- [ ] Al recargar, se pierde todo (es lo correcto: aún no hay backend).

---

## 🚀 ¿Y el backend en C++?

Esta interfaz está lista para conectarse. Los siguientes pasos serían:

1. Cambiar `action="#"` del form por la URL de tu servidor C++ (`http://localhost:8080/api/alumnos`).
2. Usar `fetch()` en JS para enviar los datos.
3. Recibir la respuesta JSON y actualizar el modelo.

Pero eso ya no es parte de estas actividades 🎓. Lo veremos en las siguientes, vamos a crear un servidor web 💀💀💀.

---
