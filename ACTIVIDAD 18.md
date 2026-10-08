# Actividad 18. JavaScript

## Temas

- Variables
    
- Funciones
    
- Eventos
    

## Ejercicio

Agregar interactividad.

Ejemplos:

Botón

```text
Mostrar mensaje
```

Botón

```text
Limpiar formulario
```

Botón

```text
Calcular promedio
```

Todo ocurre únicamente en el navegador.

---
# TEORÍA

---

## 1. ¿Qué es JavaScript?

- **Definición:** JavaScript (JS) es un lenguaje de programación que se ejecuta en el **navegador** (y también en servidores con Node.js). Permite **dar comportamiento e interactividad** a una página web.
- **Diferencia con HTML y CSS:**
  - **HTML:** estructura y contenido.
  - **CSS:** presentación y estilo.
  - **JavaScript:** comportamiento, lógica, respuestas a eventos del usuario.
- **Función en el proyecto:** Hacer que los botones hagan algo (mostrar mensajes, limpiar formularios, calcular promedios) **sin recargar la página** y **sin backend todavía**.
- **Ejecución:** Corre en el navegador del usuario. No puede leer/escribir archivos directamente (eso lo hará el backend en C++ más adelante).

---

## 2. Formas de incluir JavaScript

- **JS externo (recomendado):** Archivo `.js` enlazado desde el HTML.
  ```html
  <script src="app.js"></script>
  ```
  - Se coloca **antes de cerrar `</body>`** para que el HTML ya esté cargado cuando el JS se ejecute.
  - O bien en el `<head>` con el atributo `defer`: `<script src="app.js" defer></script>`.
- **JS interno:** Dentro de `<script> ... </script>` en el HTML. Útil para pruebas rápidas.
- **JS en línea (inline):** En atributos como `onclick="..."`. **Evítalo**, mezcla HTML con lógica y dificulta el mantenimiento.

**Estructura sugerida:**
```
public/
├── index.html
├── styles.css
└── app.js
```

---

## 3. Sintaxis básica

- **Declaración de variables:** `let`, `const`, `var` (evita `var`).
- **Funciones:** bloques reutilizables de código.
- **Sentencias:** terminan con `;` (opcional pero recomendado).
- **Comentarios:** `// una línea` o `/* varias líneas */`.
- **Sensible a mayúsculas:** `nombre` ≠ `Nombre`.
- **Tipado dinámico:** No declaras el tipo; el motor lo infiere.

---

## 4. Variables

- **`let`:** Variable que puede cambiar su valor. Úsala por defecto cuando necesites reasignar.
- **`const`:** Variable que **no se reasigna** (aunque los objetos y arreglos que contiene sí pueden modificarse internamente). Úsala cuando el valor no cambie.
- **`var`:** Forma antigua, con problemas de ámbito. **No la uses.**
- **Ámbito:** Las variables declaradas con `let` y `const` tienen **ámbito de bloque** (`{ }`). Fuera del bloque, no existen.
- **Hoisting:** Las declaraciones se "elevan" al inicio del ámbito, pero `let` y `const` no se pueden usar antes de declararlas (dan error). `var` sí, pero da `undefined`.

**Fragmentos sueltos:**
```js
let contador = 0;
const PI = 3.1416;
let nombre = "Ana";
```

---

## 5. Tipos de datos en JavaScript

| Tipo | Ejemplo | Notas |
|------|---------|-------|
| `number` | `42`, `3.14` | No distingue int y float. |
| `string` | `"hola"`, `'mundo'` | Comillas simples o dobles (o backticks `` ` `` para templates). |
| `boolean` | `true`, `false` | |
| `null` | `null` | Ausencia intencional de valor. |
| `undefined` | `undefined` | Variable declarada sin asignar. |
| `object` | `{ nombre: "Ana", edad: 20 }` | Objetos literales. |
| `array` | `[1, 2, 3]` | En JS, los arreglos son objetos. |
| `function` | `function() {}` | Las funciones son ciudadanos de primera clase. |

- **Template literals:** `` `Hola ${nombre}, tienes ${edad} años` `` → interpolación con `${}`.
- **Conversión de tipos:**
  - `Number("42")` → 42
  - `String(42)` → "42"
  - `parseInt("42px")` → 42 (extrae número)
  - `parseFloat("3.14")` → 3.14

---

## 6. Operadores

- **Aritméticos:** `+ - * / % **`
- **Asignación:** `= += -= *= /=`
- **Comparación:**
  - `==` → comparación **débil** (convierte tipos). **Evítala.**
  - `===` → comparación **estricta** (tipo y valor). **Úsala siempre.**
  - `!=`, `!==`, `<`, `>`, `<=`, `>=`
- **Lógicos:** `&&` (Y), `||` (O), `!` (NO).
- **Ternario:** `condicion ? valorSiTrue : valorSiFalse`
- **Incremento/decremento:** `++`, `--`

---

## 7. Funciones

- **Declaración:**
  ```js
  function sumar(a, b) {
      return a + b;
  }
  ```
- **Expresión (arrow function):** Más compacta, común en JS moderno.
  ```js
  const sumar = (a, b) => a + b;
  ```
- **Parámetros:** Pueden tener valores por defecto (`function f(x = 0)`).
- **Retorno:** Si no hay `return`, devuelve `undefined`.
- **Funciones anónimas:** Sin nombre, útiles como callbacks.
- **Callbacks:** Pasar una función como argumento a otra.

**Diferencias clave arrow vs. function tradicional:**
- Arrow no tiene su propio `this` (hereda el del ámbito donde se define).
- Arrow no se puede usar como constructor.
- Arrow no tiene objeto `arguments`.
- Para esta actividad, las arrow functions son cómodas y suficientes.

---

## 8. El DOM (Document Object Model)

- **Definición:** Es la representación en memoria del HTML como un árbol de objetos. JavaScript puede leerlo y modificarlo.
- **Objeto global `document`:** Punto de entrada al DOM.

### Seleccionar elementos

| Método | Devuelve |
|--------|----------|
| `document.getElementById("id")` | El elemento con ese id (o `null`). |
| `document.querySelector("selector")` | El **primer** elemento que coincide con el selector CSS. |
| `document.querySelectorAll("selector")` | Todos los que coinciden (NodeList). |
| `document.getElementsByClassName("clase")` | Colección viva por clase. |
| `document.getElementsByTagName("etiqueta")` | Colección viva por etiqueta. |

**Recomendación:** Usa `querySelector` y `querySelectorAll` (más flexibles, aceptan cualquier selector CSS).

### Modificar elementos

- **Texto:** `elemento.textContent = "nuevo texto";`
- **HTML interno:** `elemento.innerHTML = "<strong>texto</strong>";` (cuidado con seguridad si el contenido viene del usuario).
- **Atributos:** `elemento.setAttribute("href", "url");` o `elemento.href = "url";`
- **Estilos:** `elemento.style.color = "red";` (o mejor, agregar/quitar clases).
- **Clases:**
  - `elemento.classList.add("activo");`
  - `elemento.classList.remove("activo");`
  - `elemento.classList.toggle("activo");`
  - `elemento.classList.contains("activo");` → devuelve true/false.
- **Valor de un input:** `input.value` (leer o escribir).

---

## 9. Eventos

- **Definición:** Acciones del usuario o del sistema que JavaScript puede detectar y manejar (clic, tecla, envío de formulario, carga de página).
- **Formas de escuchar eventos:**
  1. **`addEventListener` (recomendado):** Permite múltiples listeners, más limpio.
     ```js
     boton.addEventListener("click", function() {
         // qué hacer al hacer clic
     });
     ```
  2. **Propiedad `onclick` (menos recomendado):** Solo un listener, sobrescribe.
  3. **Atributo HTML `onclick="..."` (evítalo):** Mezcla HTML y JS.

### Eventos comunes

| Evento | Se dispara cuando… |
|--------|--------------------|
| `click` | El usuario hace clic. |
| `submit` | Se envía un formulario. |
| `input` | El valor de un input cambia (mientras escribe). |
| `change` | El valor cambia y pierde el foco. |
| `keydown` / `keyup` | Se presiona/suelta una tecla. |
| `mouseover` / `mouseout` | El mouse entra/sale del elemento. |
| `DOMContentLoaded` | El HTML terminó de cargarse. |

### Objeto `event`

- Se recibe como parámetro de la función listener.
- **`event.preventDefault()`:** Evita el comportamiento por defecto (por ejemplo, que un formulario recargue la página al enviarse).
- **`event.target`:** El elemento que disparó el evento.

**Fragmento suelto (prevenir recarga al enviar):**
```js
formulario.addEventListener("submit", (e) => {
    e.preventDefault();
    // procesar los datos
});
```

---

## 10. Interacción con formularios

- **Leer valores:** `document.getElementById("nombre").value` o `document.querySelector("#nombre").value`.
- **Los valores de input siempre son strings.** Convertir con `Number()` o `parseFloat()` cuando necesites operar.
- **Escribir valores:** `input.value = "nuevo";`
- **Limpiar formulario:**
  - Opción 1: `formulario.reset();`
  - Opción 2: Asignar `""` a cada campo individualmente.
- **Validaciones:** Puedes validar manualmente o apoyarte en las nativas de HTML (`required`, `min`, `max`, `type`).

---

## 11. Mensajes al usuario

- **`alert(mensaje);`** → Muestra un cuadro de diálogo modal. **Bloquea la ejecución.** Úsalo solo para pruebas.
- **`console.log(mensaje);`** → Escribe en la consola del navegador (F12 → Console). Ideal para depurar.
- **`confirm(mensaje);`** → Devuelve `true` o `false` según la elección del usuario.
- **`prompt(mensaje);`** → Pide un valor al usuario (string o `null` si cancela).
- **En el DOM (recomendado para UX):** Crear/modificar un `<div>` o `<span>` con el mensaje y mostrarlo en la página.
  ```js
  document.getElementById("resultado").textContent = "Promedio: " + promedio;
  ```

---

## 12. Aplicación a los botones del ejercicio

### A. Botón "Mostrar mensaje"
- Escuchar el evento `click`.
- Mostrar un mensaje en un `<div>` de la página (mejor que `alert`).
- Alternar visibilidad con `classList.toggle("oculto")` si quieres mostrar/ocultar.

### B. Botón "Limpiar formulario"
- Escuchar el evento `click`.
- Llamar a `formulario.reset()` o vaciar cada campo.
- Opcionalmente, mostrar un mensaje de confirmación.

### C. Botón "Calcular promedio"
- Escuchar el evento `click`.
- Leer los valores de los inputs de calificaciones (string → number).
- Calcular el promedio.
- Mostrar el resultado en un `<div>` o `<span>`.
- **Validaciones:** Verificar que los campos tengan valores numéricos válidos antes de calcular.

**Fragmento suelto (leer, convertir, calcular, mostrar):**
```js
const c1 = Number(document.querySelector("#cal1").value);
const c2 = Number(document.querySelector("#cal2").value);
const c3 = Number(document.querySelector("#cal3").value);
const promedio = (c1 + c2 + c3) / 3;
document.querySelector("#resultado").textContent = `Promedio: ${promedio.toFixed(2)}`;
```

- **`toFixed(2)`:** Redondea a 2 decimales y devuelve string.

---

## 13. Buenas prácticas

- **Separa HTML, CSS y JS** en archivos distintos.
- **Carga el JS al final del body** o usa `defer` para que el DOM esté listo.
- **Usa `addEventListener`**, no atributos `onclick` en HTML.
- **Usa `const` por defecto**, `let` solo cuando necesites reasignar.
- **Usa `===` y `!==`**, no `==` ni `!=`.
- **Nombra las funciones con verbos** (`mostrarMensaje`, `limpiarFormulario`, `calcularPromedio`).
- **Nombra las variables en camelCase** (`nombreCompleto`, `promedioFinal`).
- **Verifica que los elementos existan** antes de usarlos (si `querySelector` devuelve `null`, lanzará error al acceder a propiedades).
- **Usa `console.log` para depurar**, no `alert`.
- **Comenta el código** explicando la lógica de cada bloque.
- **Maneja errores:** Si el usuario ingresa un valor no numérico, muestra un mensaje en lugar de fallar.
- **Evita variables globales:** Envuelve tu código en funciones o en un IIFE (función autoejecutable).
- **`'use strict';`** al inicio del archivo activa el modo estricto (buena práctica).

---

## 14. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`querySelector` devuelve `null`** | Verifica el selector y que el elemento exista en el HTML. Carga el JS después del HTML o usa `defer`. |
| **Olvidar `e.preventDefault()` en submit** | El formulario recarga la página. Llama a `e.preventDefault()` al inicio del listener. |
| **Usar `==` en lugar de `===`** | Comportamiento inesperado por conversión de tipos. Usa `===`. |
| **Concatenar strings con `+` y números** | `"5" + 3` da `"53"`, no `8`. Convierte con `Number()` antes de operar. |
| **Modificar el DOM antes de que esté listo** | El script corre antes de que existan los elementos. Usa `defer` o coloca el script al final. |
| **`document.getElementById` con id mal escrito** | JS es sensible a mayúsculas. Verifica el id. |
| **No manejar el caso de input vacío** | Valida antes de calcular. `Number("")` da `0`, pero `parseFloat("")` da `NaN`. |
| **Confundir `innerHTML` con `textContent`** | `innerHTML` interpreta HTML; `textContent` lo muestra como texto. Usa `textContent` salvo que necesites HTML. |
| **Sobrescribir listeners** | `elemento.onclick = f` sobrescribe; `addEventListener` acumula. Usa siempre `addEventListener`. |
| **`NaN` en cálculos** | Algún valor no es numérico. Verifica con `isNaN(valor)`. |

---

## 15. Resumen de conceptos clave

- **JavaScript:** Lenguaje de comportamiento en el navegador.
- **Inclusión:** Archivo externo `.js` enlazado con `<script src="...">`.
- **Variables:** `const` por defecto, `let` si reasignas, nunca `var`.
- **Tipos:** number, string, boolean, null, undefined, object, array, function.
- **Comparación:** Usa `===` y `!==`.
- **Funciones:** Declaración tradicional o arrow functions.
- **DOM:** `document.querySelector`, `textContent`, `value`, `classList`.
- **Eventos:** `addEventListener`, `event.preventDefault()`.
- **Formularios:** Leer con `.value`, convertir con `Number()`, limpiar con `.reset()`.
- **Mensajes:** `console.log` para depurar; modificar el DOM para mostrar al usuario.
- **Aplicación:** Los tres botones (mostrar, limpiar, calcular) se implementan con listeners sobre `click` o `submit`.
- **Todo ocurre en el navegador**, sin backend todavía.

---
# PASO A PASO

---
# Actividad 18 — JavaScript: eventos y botones

## Objetivo
Que la interfaz **reaccione**: mostrar mensajes, limpiar el formulario y calcular promedios. Todo en el navegador, sin backend.

## Archivo nuevo
Crea `public/app.js`.

Antes de cerrar `</body>` en `index.html`:

```html
<script src="app.js"></script>
```

⚠️ Va **antes** de `</body>`, no en `<head>`, para que el HTML ya exista cuando el JS se ejecute.

---

## Paso 1 — Modo estricto y verificación

Al inicio de `app.js`:

```js
'use strict';

console.log('app.js cargado correctamente');
```

✅ Abre la consola (F12 → Console). Debe aparecer el mensaje.

---

## Paso 2 — Botones extra en el HTML

Dentro de la sección `#resumen`, agrega un bloque de acciones:

```html
<div class="acciones-js">
  <button type="button" id="btn-mensaje">Mostrar mensaje</button>
  <button type="button" id="btn-promedio">Calcular promedio</button>
</div>

<p id="resultado" class="resultado"></p>
```

⚠️ `type="button"` para que no envíen el form.

---

## Paso 3 — Seleccionar elementos

En `app.js`, después del console.log:

```js
const btnMensaje   = document.querySelector('#btn-mensaje');
const btnPromedio  = document.querySelector('#btn-promedio');
const resultado    = document.querySelector('#resultado');
const formAlumno   = document.querySelector('#form-alumno');
```

✅ En la consola escribe `btnMensaje` y presiona Enter. Debe devolver el `<button>`.

---

## Paso 4 — Botón "Mostrar mensaje"

```js
btnMensaje.addEventListener('click', () => {
  resultado.textContent = '¡Hola! Este mensaje vive solo en el navegador.';
  resultado.classList.add('visible');
});
```

Agrega en `styles.css`:

```css
.resultado {
  margin-top: 1rem;
  padding: 0.75rem 1rem;
  border-radius: var(--radio);
  background: #dcfce7;
  color: #166534;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.resultado.visible {
  opacity: 1;
}
```

---

## Paso 5 — Evitar que el formulario recargue

```js
formAlumno.addEventListener('submit', (e) => {
  e.preventDefault();   // sin esto, la página se recarga

  const nombre = document.querySelector('#nombre').value.trim();

  if (nombre === '') {
    resultado.textContent = 'El nombre no puede estar vacío.';
    return;
  }

  resultado.textContent = `Alumno "${nombre}" listo para registrar (todavía sin backend).`;
  resultado.classList.add('visible');
});
```

📌 Prueba: escribe un nombre y presiona "Registrar". NO debe recargarse la página.

---

## Paso 6 — Botón "Limpiar"

El botón `type="reset"` ya limpia el form solo, pero si quieres hacerlo con JS:

```js
document.querySelector('#btn-limpiar').addEventListener('click', (e) => {
  e.preventDefault();
  formAlumno.reset();
  resultado.textContent = 'Formulario limpio.';
  resultado.classList.add('visible');
});
```

---

## Paso 7 — Calcular promedio (con 3 calificaciones)

Agrega al HTML, dentro del formulario, un bloque de calificaciones:

```html
<fieldset>
  <legend>Calificaciones</legend>

  <label for="cal1">Calificación 1:</label>
  <input type="number" id="cal1" min="0" max="10" step="0.1">

  <label for="cal2">Calificación 2:</label>
  <input type="number" id="cal2" min="0" max="10" step="0.1">

  <label for="cal3">Calificación 3:</label>
  <input type="number" id="cal3" min="0" max="10" step="0.1">
</fieldset>
```

En `app.js`:

```js
btnPromedio.addEventListener('click', () => {
  const c1 = Number(document.querySelector('#cal1').value);
  const c2 = Number(document.querySelector('#cal2').value);
  const c3 = Number(document.querySelector('#cal3').value);

  if (isNaN(c1) || isNaN(c2) || isNaN(c3)) {
    resultado.textContent = 'Ingresa las 3 calificaciones antes de calcular.';
    resultado.classList.add('visible');
    return;
  }

  const promedio = (c1 + c2 + c3) / 3;
  resultado.textContent = `Promedio: ${promedio.toFixed(2)}`;
  resultado.classList.add('visible');
});
```

📌 `Number("")` da `0`, por eso usamos `isNaN` para detectar vacíos: `Number("")` → 0, pero si el campo está vacío queremos avisar. Truco: valida primero `if (valor === '')`.

### Versión más segura del chequeo:

```js
const v1 = document.querySelector('#cal1').value;
const v2 = document.querySelector('#cal2').value;
const v3 = document.querySelector('#cal3').value;

if (v1 === '' || v2 === '' || v3 === '') {
  resultado.textContent = 'Faltan calificaciones.';
  return;
}

const promedio = (Number(v1) + Number(v2) + Number(v3)) / 3;
```

---

## Paso 8 — Función reutilizable para mostrar resultados

Refactoriza:

```js
function mostrarResultado(mensaje, tipo = 'ok') {
  resultado.textContent = mensaje;
  resultado.classList.remove('ok', 'error');
  resultado.classList.add('visible', tipo);
}
```

Y en `styles.css`:

```css
.resultado.ok    { background: #dcfce7; color: #166534; }
.resultado.error { background: #fee2e2; color: #991b1b; }
```

Ahora en vez de `resultado.textContent = ...` usas `mostrarResultado('...')`.

---

## ✅ Checklist de la Actividad 18

- [ ] `app.js` está enlazado antes de `</body>`.
- [ ] El botón "Mostrar mensaje" cambia el texto de `#resultado`.
- [ ] El formulario no recarga la página al enviarse.
- [ ] El botón "Calcular promedio" valida y muestra el resultado.
- [ ] Usas `===` en vez de `==` en las comparaciones.
- [ ] Usas `const` por defecto y `let` solo cuando reasignas.

## 🧠 Mini-reto
Haz que el botón "Mostrar mensaje" **alterne** el mensaje (si ya está visible, que lo oculte). Pista: `classList.toggle('visible')`.