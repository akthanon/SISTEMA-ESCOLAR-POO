# Actividad 19. DOM

## Temas

- document
    
- querySelector
    
- innerHTML
    

## Ejercicio

Mostrar los alumnos directamente en la página.

Crear una tabla dinámica.

Agregar:

- insertar filas
    
- eliminar filas
    
- modificar datos
    

Todavía no existe comunicación con C++.

Todo vive únicamente en JavaScript.

---
# TEORÍA

---

## 1. ¿Qué es el DOM?

- **Definición:** El DOM (Document Object Model) es la representación en memoria del HTML como un **árbol de nodos**. Cada etiqueta, atributo y texto es un nodo.
- **Función:** JavaScript puede **leer, crear, modificar y eliminar** nodos del DOM, y esos cambios se reflejan **inmediatamente** en la página, sin recargar.
- **Objeto raíz:** `document` es el punto de entrada al árbol. Representa todo el documento HTML.
- **Diferencia con el HTML fuente:** El HTML es el texto inicial; el DOM es el estado **actual** de la página (puede diferir del HTML original si JS lo ha modificado).
- **En esta actividad:** Toda la gestión de alumnos (agregar, eliminar, modificar) ocurre **solo en el DOM**, sin backend. Al recargar la página, todo se pierde.

---

## 2. El objeto `document`

- **Propiedades y métodos más usados:**
  - `document.querySelector(selector)` → primer elemento que coincide.
  - `document.querySelectorAll(selector)` → todos los que coinciden (NodeList).
  - `document.getElementById(id)` → elemento con ese id.
  - `document.createElement(tag)` → crea un nuevo nodo elemento.
  - `document.createTextNode(texto)` → crea un nodo de texto (menos usado que `textContent`).
  - `document.body` → el `<body>`.
  - `document.title` → el título de la pestaña.
- **`document`** está disponible globalmente en el navegador; no necesitas importarlo.

---

## 3. Selección de elementos

- **`querySelector(selector)`:** Devuelve el **primer** elemento que coincide con el selector CSS. Devuelve `null` si no encuentra nada.
- **`querySelectorAll(selector)`:** Devuelve una **NodeList** (parecida a un arreglo, pero no lo es del todo). Se puede recorrer con `forEach`.
- **Ventaja de `querySelector`:** Acepta cualquier selector CSS (etiqueta, clase, id, combinadores, pseudo-clases).
- **Verificación obligatoria:** Siempre verifica que el resultado no sea `null` antes de usarlo.
- **Búsqueda dentro de un elemento:** Puedes llamar `querySelector` sobre un elemento ya seleccionado para buscar solo dentro de él.
  ```js
  const tabla = document.querySelector("#tabla-alumnos");
  const fila = tabla.querySelector("tr");  // solo dentro de la tabla
  ```

**Fragmentos sueltos:**
```js
const boton = document.querySelector("#btn-agregar");
const filas = document.querySelectorAll("tbody tr");
```

---

## 4. Creación de elementos

- **`document.createElement(tag)`:** Crea un nodo elemento vacío. Aún no está en la página.
- **`elemento.textContent = "..."`:** Asigna texto plano al elemento (seguro, no interpreta HTML).
- **`elemento.innerHTML = "..."`:** Asigna HTML interno (puede interpretar etiquetas). **Cuidado:** si el contenido viene del usuario, es un riesgo de **XSS** (inyección de código). Prefiere `textContent` cuando solo sea texto.
- **`elemento.setAttribute(nombre, valor)`:** Añade o modifica un atributo.
- **`elemento.classList.add("clase")`:** Añade una clase CSS.
- **`elemento.value = "..."`:** Asigna valor a un input.
- **Un nodo recién creado no aparece en la página** hasta que se inserta en el DOM.

**Fragmentos sueltos:**
```js
const fila = document.createElement("tr");
const celda = document.createElement("td");
celda.textContent = "Ana";
fila.appendChild(celda);
```

---

## 5. Inserción de elementos en el DOM

- **`padre.appendChild(hijo)`:** Añade el hijo **al final** del padre.
- **`padre.insertBefore(nuevo, referencia)`:** Inserta el nuevo antes del nodo de referencia.
- **`padre.prepend(hijo)`:** Inserta al **inicio**.
- **`elemento.insertAdjacentElement(posicion, nuevo)`:** Posiciones: `'beforebegin'`, `'afterbegin'`, `'beforeend'`, `'afterend'`.
- **`elemento.append(...nodos)`:** Añade uno o varios nodos (o strings) al final. Más flexible que `appendChild`.
- **`elemento.innerHTML += "..."`:** Añade HTML al final (pero **destruye y recrea** todo el contenido, perdiendo listeners). **Evítalo** si ya tienes elementos con eventos.

**Fragmento suelto:**
```js
tbody.appendChild(fila);         // añade al final
tbody.prepend(fila);             // añade al inicio
tbody.insertBefore(fila, tbody.firstChild); // equivalente a prepend
```

- **Orden de creación antes de insertar:** Crea la fila, las celdas, asígnales contenido, y **al final** inserta la fila en el tbody. Así solo se hace una operación de inserción en el DOM.

---

## 6. Eliminación de elementos

- **`elemento.remove()`:** Elimina el elemento del DOM (método moderno y directo).
- **`padre.removeChild(hijo)`:** Método clásico; requiere referencia al padre.
- **`elemento.innerHTML = ""`:** Vacía el contenido del elemento (elimina todos los hijos). Útil para "refrescar" una tabla.
- **Eliminar desde un evento:** Usualmente se hace desde un botón dentro de la fila, subiendo al padre con `elemento.closest("tr")` o `elemento.parentElement`.
  ```js
  botonEliminar.addEventListener("click", (e) => {
      const fila = e.target.closest("tr");
      fila.remove();
  });
  ```
- **Cuidado con los índices:** Si eliminas filas del DOM, los índices cambian. Si tu modelo de datos es un arreglo de objetos en JS, sincroniza: elimina también del arreglo.

---

## 7. Modificación de datos

- **Modificar texto de una celda:** `celda.textContent = "nuevo valor";`
- **Modificar HTML interno:** `celda.innerHTML = "<strong>nuevo</strong>";`
- **Modificar atributos:** `elemento.setAttribute("data-id", "5");`
- **Modificar estilos:** `elemento.style.color = "red";` (mejor usar clases).
- **Alternar clases:** `elemento.classList.toggle("activo");`
- **Edición en línea:**
  - Opción 1: Usar `prompt()` para pedir el nuevo valor (simple, poco elegante).
  - Opción 2: Convertir la celda en `<input>` editable temporalmente.
  - Opción 3: Usar el atributo `contenteditable="true"` en la celda (permite editar directamente).
- **Sincronización con el modelo:** Cada modificación en el DOM debería reflejarse en el arreglo de objetos (el "modelo de datos"). Si no, se desincroniza.

---

## 8. Tabla dinámica: estructura

- **HTML base:** La tabla existe en el HTML con `<thead>` y un `<tbody>` vacío.
- **JS:** Cada alumno es un objeto. Al agregar, se crea una fila `<tr>` con celdas `<td>` (una por campo) y se inserta en `<tbody>`.
- **Botones por fila:** Cada fila puede tener una última celda con botones "Editar" y "Eliminar".
- **Identificador de fila:** Asigna un `data-id` único a cada fila o a los botones, para saber a qué alumno corresponden.
  ```js
  fila.dataset.id = "1";
  ```
- **Búsqueda por `data-id`:** Puedes localizar la fila con `querySelector(`[data-id="${id}"]`)`.

**Fragmentos sueltos (atributo data):**
```js
boton.dataset.id = alumno.id;              // asigna
const id = boton.dataset.id;               // lee
const fila = document.querySelector(`[data-id="${id}"]`); // busca
```

---

## 9. Patrón "modelo + vista"

- **Modelo:** Un arreglo de objetos en JS que representa los datos (`let alumnos = [];`).
- **Vista:** El DOM, que refleja el modelo (la tabla).
- **Función de renderizado:** Una función que **reconstruye** la tabla a partir del modelo.
  - Recibe el arreglo.
  - Vacía el `<tbody>`.
  - Recorre el arreglo y crea una fila por cada alumno.
  - Inserta todas las filas.
- **Ventaja:** El modelo es la fuente de verdad. Al agregar/eliminar/modificar, actualizas el modelo y **vuelves a renderizar**. Así el DOM siempre refleja el estado real.
- **Alternativa (más eficiente):** Actualizar solo el DOM afectado (añadir una fila, eliminar una fila específica). Pero es más propenso a errores de sincronización.
- **Recomendación para esta actividad:** Usa el patrón modelo + render para simplificar.

**Fragmento suelto (patrón):**
```js
let alumnos = [];

function renderizar() {
    const tbody = document.querySelector("#tabla-alumnos tbody");
    tbody.innerHTML = ""; // vaciar
    alumnos.forEach(alumno => {
        // crear fila y celdas, insertar en tbody
    });
}
```

---

## 10. Operaciones CRUD en la tabla

### A. Insertar fila (Create)
1. Leer los valores del formulario.
2. Crear un objeto alumno.
3. Añadirlo al arreglo `alumnos`.
4. Volver a renderizar (o insertar la fila directamente).

### B. Eliminar fila (Delete)
1. Identificar el alumno a eliminar (por índice, por `data-id`, o por nombre).
2. Eliminarlo del arreglo `alumnos`.
3. Volver a renderizar.

### C. Modificar fila (Update)
1. Identificar el alumno a modificar.
2. Pedir los nuevos valores (prompt, inputs, o contenteditable).
3. Actualizar el objeto en el arreglo `alumnos`.
4. Volver a renderizar.

### D. Leer fila (Read)
- Ya se hace al recorrer el arreglo para renderizar.

---

## 11. Manejo de eventos en elementos dinámicos

- **Problema:** Si creas botones dinámicamente y les asignas `addEventListener` **después** de insertarlos, funciona. Pero si los creas con `innerHTML`, los listeners se pierden.
- **Solución:** Asigna los listeners a cada botón antes de insertarlo, o usa **delegación de eventos**.
- **Delegación de eventos:** Escucha el evento en un **padre** (por ejemplo, el `<tbody>`) y en el listener verifica `event.target` para saber qué botón se pulsó.
  ```js
  tbody.addEventListener("click", (e) => {
      if (e.target.classList.contains("btn-eliminar")) {
          const fila = e.target.closest("tr");
          // eliminar fila
      }
  });
  ```
- **Ventaja:** Funciona aunque se creen y eliminen filas dinámicamente, porque el listener está en el padre (que existe siempre).

---

## 12. Buenas prácticas

- **Verifica que los elementos existan** antes de usarlos (`if (elemento) { ... }`).
- **Usa `textContent` en lugar de `innerHTML`** para texto proveniente del usuario (evita XSS).
- **Construye los nodos antes de insertarlos** en el DOM: menos reflows y mejor rendimiento.
- **Usa `documentFragment`** si vas a insertar muchas filas: crea el fragmento, añade todos los nodos, y luego inserta el fragmento una sola vez.
  ```js
  const fragment = document.createDocumentFragment();
  // añadir filas al fragment
  tbody.appendChild(fragment); // una sola operación
  ```
- **Delegación de eventos** para listas/tablas dinámicas.
- **Separa modelo y vista:** no mezcles lógica de datos con manipulación del DOM.
- **Nombra tus funciones con verbos** (`agregarFila`, `eliminarFila`, `renderizarTabla`).
- **Evita `innerHTML +=`** porque destruye y recrea todo.
- **No abuses de `innerHTML`:** si necesitas crear nodos con estructura, usa `createElement` + `appendChild`.
- **Sincroniza modelo y vista** siempre.
- **Documenta cada operación** con comentarios explicando el flujo.
- **Maneja casos límite:** tabla vacía, formulario incompleto, eliminación de fila inexistente.

---

## 13. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`querySelector` devuelve `null`** | Verifica el selector y el orden de carga (usa `defer`). |
| **Nodo creado no aparece** | Falta insertarlo con `appendChild` o similar. |
| **Listeners se pierden al renderizar** | Usa delegación de eventos en el padre. |
| **`innerHTML +=` rompe los eventos** | Reconstruye todo con `createElement` y `appendChild`. |
| **Índices de fila desincronizados con el modelo** | Usa `data-id` o sincroniza siempre con el arreglo. |
| **XSS al usar `innerHTML` con input del usuario** | Usa `textContent` para texto del usuario. |
| **`tbody` no existe** | Asegúrate de tener `<tbody>` explícito en la tabla (el navegador no siempre lo crea). |
| **Eliminar una fila y no actualizar el modelo** | Elimina también del arreglo, o volver a renderizar desde el modelo. |
| **Insertar filas una por una (muchos reflows)** | Usa `documentFragment` o construye antes de insertar. |
| **Confundir `closest` con `parentElement`** | `closest("tr")` sube hasta encontrar un `tr` ancestro; `parentElement` es el padre directo. |

---

## 14. Resumen de conceptos clave

- **DOM:** Árbol de nodos que representa el HTML. JS lo manipula para cambiar la página.
- **Selección:** `querySelector`, `querySelectorAll`.
- **Creación:** `createElement`, `textContent`, `setAttribute`, `classList`.
- **Inserción:** `appendChild`, `prepend`, `insertBefore`, `insertAdjacentElement`, `documentFragment`.
- **Eliminación:** `remove()`, `removeChild()`, `innerHTML = ""`.
- **Modificación:** `textContent`, `innerHTML`, atributos, estilos, clases.
- **Tabla dinámica:** Crear `<tr>` con `<td>` por cada alumno e insertar en `<tbody>`.
- **Modelo + vista:** Arreglo de objetos como fuente de verdad; renderizar reconstruye la tabla.
- **CRUD:** Agregar, eliminar, modificar, leer sobre el modelo y reflejar en el DOM.
- **Delegación de eventos:** Escuchar en el padre para manejar botones de filas dinámicas.
- **`data-id`:** Identificador único por fila para localizar y operar sobre ella.
- **Buenas prácticas:** `textContent` > `innerHTML`, construir antes de insertar, sincronizar modelo y vista.

---
# PASO A PASO

---
# Actividad 19 — DOM: tabla dinámica con CRUD

## Objetivo
Que el formulario **agregue filas** a la tabla, que se puedan **eliminar** y **editar**, todo en JS sin backend. Al recargar, se pierde (aún no hay C++).

## Patrón que vamos a usar: modelo + render

- **Modelo**: un arreglo de objetos `alumnos = []`.
- **Vista**: la tabla en el DOM.
- **Render**: una función que reconstruye la tabla a partir del arreglo.

Esta es la forma más simple y evita desincronizaciones.

---

## Paso 1 — Modelo y array

En `app.js`, arriba de todo después de `'use strict';`:

```js
let alumnos = [];   // nuestro "modelo"
let siguienteId = 1;
```

---

## Paso 2 — Función render

```js
function render() {
  const tbody = document.querySelector('#tabla-alumnos tbody');
  tbody.innerHTML = '';  // vaciar

  if (alumnos.length === 0) {
    const filaVacia = document.createElement('tr');
    filaVacia.innerHTML = `<td colspan="5" class="vacio">Sin alumnos registrados</td>`;
    tbody.appendChild(filaVacia);
    actualizarResumen();
    return;
  }

  alumnos.forEach((alumno) => {
    const fila = document.createElement('tr');
    fila.dataset.id = alumno.id;

    fila.innerHTML = `
      <td>${alumno.matricula}</td>
      <td>${alumno.nombre}</td>
      <td>${alumno.edad}</td>
      <td>${alumno.correo}</td>
      <td class="acciones">
        <button class="btn-editar"   data-accion="editar">Editar</button>
        <button class="btn-eliminar" data-accion="eliminar">Eliminar</button>
      </td>
    `;

    tbody.appendChild(fila);
  });

  actualizarResumen();
}
```

⚠️ Nota sobre `${...}`: estamos usando template literals. Los datos vienen del propio usuario, así que hay un riesgo teórico de XSS; en la actividad 21 verás la versión con `createElement + textContent` que es la recomendada. Por ahora es aceptable para ver el flujo.

CSS extra:

```css
td.vacio {
  text-align: center;
  padding: 2rem;
  color: #9ca3af;
  font-style: italic;
}

.acciones {
  display: flex;
  gap: 0.5rem;
}

.btn-editar {
  background: #f59e0b;
  color: white;
}

.btn-eliminar {
  background: var(--color-peligro);
  color: white;
}
```

---

## Paso 3 — Actualizar resumen

```js
function actualizarResumen() {
  document.querySelector('#total-alumnos').textContent = alumnos.length;

  const ultimo = alumnos[alumnos.length - 1];
  document.querySelector('#ultimo-alumno').textContent =
    ultimo ? ultimo.nombre : '—';
}
```

---

## Paso 4 — Insertar (Create)

Modifica el listener del submit:

```js
formAlumno.addEventListener('submit', (e) => {
  e.preventDefault();

  const nombre    = document.querySelector('#nombre').value.trim();
  const matricula = document.querySelector('#matricula').value.trim();
  const edad      = Number(document.querySelector('#edad').value);
  const correo    = document.querySelector('#correo').value.trim();

  if (!nombre || !matricula || !edad || !correo) {
    mostrarResultado('Completa todos los campos.', 'error');
    return;
  }

  const nuevoAlumno = {
    id: siguienteId++,
    nombre,
    matricula,
    edad,
    correo,
  };

  alumnos.push(nuevoAlumno);   // 1) modifico el modelo
  render();                    // 2) redibujo la tabla
  formAlumno.reset();          // 3) limpio el form
  mostrarResultado(`Alumno "${nombre}" registrado.`);
});
```

---

## Paso 5 — Eliminar y editar con delegación de eventos

**No** asignamos listeners a cada botón. Escuchamos en el `<tbody>` (el padre) y verificamos qué botón fue.

```js
document.querySelector('#tabla-alumnos tbody').addEventListener('click', (e) => {
  const boton = e.target.closest('button[data-accion]');
  if (!boton) return;

  const fila = boton.closest('tr');
  const id   = Number(fila.dataset.id);
  const accion = boton.dataset.accion;

  if (accion === 'eliminar') {
    eliminarAlumno(id);
  } else if (accion === 'editar') {
    editarAlumno(id);
  }
});

function eliminarAlumno(id) {
  if (!confirm('¿Eliminar este alumno?')) return;
  alumnos = alumnos.filter(a => a.id !== id);
  render();
}

function editarAlumno(id) {
  const alumno = alumnos.find(a => a.id === id);
  if (!alumno) return;

  const nuevoNombre = prompt('Nuevo nombre:', alumno.nombre);
  if (nuevoNombre === null) return;         // canceló

  const nuevaEdad = prompt('Nueva edad:', alumno.edad);
  if (nuevaEdad === null) return;

  alumno.nombre = nuevoNombre.trim() || alumno.nombre;
  alumno.edad   = Number(nuevaEdad) || alumno.edad;

  render();
}
```

🎯 **Clave didáctica**: `closest('tr')` sube desde el botón hasta la fila. `dataset.id` lee el `data-id` que pusimos en `render()`.

---

## Paso 6 — Arranque

Al final de `app.js`:

```js
render();   // pinta la tabla vacía al cargar
```

---

## Paso 7 — Probar

1. Registra 2 alumnos. Deben aparecer en la tabla.
2. Click en "Eliminar" → confirmar → desaparece.
3. Click en "Editar" → prompt con valores actuales → cambias → se actualiza.
4. Recarga la página → todo se borra. **Es lo esperado** (no hay persistencia aún).

---

## ✅ Checklist de la Actividad 19

- [ ] Existe un array `alumnos` como fuente de verdad.
- [ ] `render()` reconstruye la tabla completa.
- [ ] Cada fila tiene `data-id`.
- [ ] Un **solo** listener en el `<tbody>` maneja todos los botones.
- [ ] Eliminar y editar modifican el **array**, no el DOM directamente.
- [ ] El resumen (`#total-alumnos`, `#ultimo-alumno`) se actualiza al renderizar.

## 🧠 Mini-reto
Agrega la columna "Promedio" a la tabla. Cada alumno tendrá `calificaciones: [c1, c2, c3]`. Muestra el promedio con `toFixed(2)`.
