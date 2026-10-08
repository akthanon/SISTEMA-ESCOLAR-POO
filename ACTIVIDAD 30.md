# Actividad 30. Mostrar información almacenada

## Temas

- Lectura de archivos
    
- Generación de respuestas dinámicas
    

## Ejercicio

Crear la ruta:

```text
/api/alumnos
```

El servidor leerá `usuarios.txt` y devolverá la información en formato JSON.

Desde JavaScript utilizar `fetch()` para obtener esa lista y construir dinámicamente una tabla HTML con todos los alumnos registrados.

Agregar un botón **Actualizar** que vuelva a consultar el servidor sin recargar la página.

---
# TEORÍA

---

## 1. Visión general del flujo

1. El usuario abre la página `alumnos.html` (o pulsa un botón "Actualizar").
2. JavaScript llama a `fetch("/api/alumnos")`.
3. El servidor C++ recibe la petición GET.
4. El handler lee `data/usuarios.txt`.
5. Convierte cada línea a un objeto y todos a un arreglo JSON.
6. Responde con `Content-Type: application/json` y el arreglo serializado.
7. El JavaScript recibe la respuesta, la parsea con `.json()`.
8. Construye filas `<tr>` dinámicamente y las inserta en el `<tbody>`.
9. El usuario ve la tabla actualizada, sin recargar la página.

**Concepto clave:** Este es el **patrón API + cliente**: el backend expone datos, el frontend los consume y los presenta. La lógica de presentación vive en JS; la de datos, en C++.

---

## 2. Lado servidor: rutas y métodos

### A. Ruta `GET /api/alumnos`
- **Método:** GET (obtener un recurso).
- **Por qué GET y no POST:** No modifica estado, solo lee.
- **Ubicación:** Debe enrutarse dentro del bloque `/api/`, no como archivo estático.
- **Verificación del método:** Si el método no es GET, responder `405 Method Not Allowed`.

### B. Enrutamiento
- Dentro de la función `manejarAPI`, añade un caso para `/api/alumnos`.
- Como el proyecto puede tener ya otros endpoints (`/api/hola`, `/api/hora`, etc.), simplemente agrega otro `if` o entrada en el mapa.
- **Recomendación:** En este punto, migrar de `if-else` a un `map<string, function>` empieza a ser útil si crecen los endpoints. Pero si solo tienes 5, `if-else` sigue siendo claro.

---

## 3. Lado servidor: leer el archivo

- **Abrir `data/usuarios.txt` con `ifstream`.**
- **Verificar si existe:**
  - Si no existe → responder con un arreglo vacío `[]` (código 200), **no** 404. La ruta existe, simplemente no hay datos aún.
  - Si existe pero no se puede abrir → 500 Internal Server Error.
- **Leer línea por línea con `getline`.**
- **Ignorar líneas vacías** y **líneas mal formadas** (menos de 3 campos).
- **Parsear cada línea** con `stringstream` + `getline(ss, campo, '|')` (repaso de la Actividad 29).
- **Conversión de `edad` a int** con `try-catch`. Si falla, descarta esa línea.
- **Guardar cada alumno** en un `struct` o directamente en un vector de structs.

**Estructura sugerida:**
```cpp
struct Alumno {
    string nombre;
    int edad;
    string matricula;
};
```

- **Contenedor:** `vector<Alumno>` es lo más cómodo en C++ moderno.
- **Vector vacío:** Si no hay archivo o está vacío, el vector queda vacío y se serializa como `[]`.

**Fragmento suelto (parseo):**
```cpp
stringstream ss(linea);
string nombre, edadStr, matricula;
getline(ss, nombre, '|');
getline(ss, edadStr, '|');
getline(ss, matricula, '|');
```

---

## 4. Lado servidor: construir el JSON manualmente

### A. Estructura objetivo
- El JSON de respuesta será un **arreglo de objetos**:
  ```
  [
    {"nombre":"Juan","edad":20,"matricula":"A001"},
    {"nombre":"María","edad":21,"matricula":"A002"}
  ]
  ```
- Cada objeto tiene las claves **exactamente** como las espera el frontend (`nombre`, `edad`, `matricula`).

### B. Serialización manual
- **Recorre el vector** y para cada alumno añade un objeto JSON al string de salida.
- **Usa `to_string(edad)`** para convertir el int a string (sin comillas, es un número JSON).
- **Escapa strings** que puedan contener caracteres especiales:
  - `"` → `\"`
  - `\` → `\\`
  - `\n` → `\n`
  - `\t` → `\t`
  - `\r` → `\r`
  - Caracteres de control → `\uXXXX`
- **Comas entre objetos:** El último objeto **no lleva coma**. Es fácil olvidarlo y romper el JSON.

**Fragmento suelto (estructura del bucle):**
```cpp
string json = "[";
for (size_t i = 0; i < alumnos.size(); i++) {
    if (i > 0) json += ",";
    json += "{";
    json += "\"nombre\":\"" + escapar(alumnos[i].nombre) + "\",";
    json += "\"edad\":" + to_string(alumnos[i].edad) + ",";
    json += "\"matricula\":\"" + escapar(alumnos[i].matricula) + "\"";
    json += "}";
}
json += "]";
```

- **Sin espacios innecesarios:** El JSON no necesita indentación para ser válido. Menos bytes = más rápido.
- **Sin saltos de línea** salvo que quieras inspeccionarlo manualmente. Para el navegador, `[{"a":1},{"a":2}]` es perfectamente válido.

### C. Función `escapar`
- Recorre el string carácter por carácter.
- Si encuentra un carácter especial, lo reemplaza por su secuencia de escape.
- Devuelve el string escapado.
- **Importante:** Sin esto, un nombre con comillas rompe el JSON entero.

**Fragmento suelto (idea):**
```cpp
string escapar(const string& s) {
    string r;
    for (char c : s) {
        if (c == '"') r += "\\\"";
        else if (c == '\\') r += "\\\\";
        else if (c == '\n') r += "\\n";
        else r += c;
    }
    return r;
}
```

- **Alternativa robusta:** Usar una librería JSON (nlohmann/json). Para esta actividad, la función manual es suficiente.
- **Alternativa simple pero incompleta:** No escapar. Solo funciona si controlas los datos y sabes que no habrá comillas. **No recomendado.**

---

## 5. Lado servidor: construir la respuesta HTTP

- **Línea de estado:** `HTTP/1.1 200 OK\r\n`
- **Cabeceras obligatorias:**
  - `Content-Type: application/json; charset=utf-8\r\n` (el charset es importante por las tildes).
  - `Content-Length: <bytes>\r\n` (usa `.size()` del string JSON).
  - `Connection: close\r\n`
- **Línea vacía:** `\r\n`.
- **Cuerpo:** el JSON serializado.
- **Envío:** una sola llamada a `send` con toda la respuesta concatenada.

**Fragmento suelto (respuesta):**
```cpp
string respuesta = "HTTP/1.1 200 OK\r\n";
respuesta += "Content-Type: application/json; charset=utf-8\r\n";
respuesta += "Content-Length: " + to_string(json.size()) + "\r\n";
respuesta += "Connection: close\r\n\r\n";
respuesta += json;
send(cliente, respuesta.c_str(), respuesta.size(), 0);
```

- **Content-Length en bytes:** `json.size()` ya cuenta bytes si el string está en UTF-8 (que debería).
- **No añadas saltos de línea** después del cuerpo.

---

## 6. Casos especiales en el servidor

- **Archivo no existe:** Responder 200 con `[]`. No es un error que no haya datos aún.
- **Archivo vacío:** Mismo caso, `[]`.
- **Archivo con líneas inválidas:** Ignorarlas y responder solo con los válidos. No falles por una línea mala.
- **Error al abrir el archivo:** `500 Internal Server Error`, con cuerpo `{"error":"No se pudo leer el archivo"}`.
- **Método distinto de GET:** `405 Method Not Allowed`.
- **Content-Type esperado:** El navegador no envía Content-Type en un GET, no lo necesitas.

---

## 7. Lado cliente: `fetch` GET

- **Uso básico:**
  ```js
  const res = await fetch("/api/alumnos");
  ```
- **Es GET por defecto.** No necesitas pasar opciones.
- **Verifica `res.ok`** antes de leer el cuerpo:
  - Si `res.ok === false`, lanza error o muestra mensaje.
- **Lee el cuerpo con `.json()`** porque el servidor devuelve `application/json`.
  ```js
  const alumnos = await res.json();
  ```
- **Manejo de errores con `try-catch`**:
  - Error de red → `fetch` rechaza la promesa.
  - Error HTTP → `res.ok === false`.
  - Error de parseo → `.json()` lanza excepción si el cuerpo no es JSON válido.

**Fragmento suelto (patrón):**
```js
try {
    const res = await fetch("/api/alumnos");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const alumnos = await res.json();
    renderizarTabla(alumnos);
} catch (err) {
    mostrarError("No se pudieron cargar los alumnos");
}
```

- **Guardar el arreglo en una variable de módulo** si vas a necesitar manipularlo (por ejemplo, para eliminar un alumno). Así no tienes que volver a pedirlo al servidor.

---

## 8. Lado cliente: renderizar la tabla dinámicamente

- **Estructura HTML base:**
  - Un `<table>` con `<thead>` (encabezados fijos) y un `<tbody>` vacío.
  - El `<tbody>` es el destino de las filas dinámicas.
  - Un `<div>` o `<p>` para mostrar mensajes de error o "Cargando...".

- **Estrategia de renderizado (repaso de la Actividad 19):**
  1. **Vaciar el `<tbody>`:** `tbody.innerHTML = ""` o `tbody.replaceChildren()`.
  2. **Recorrer el arreglo** de alumnos.
  3. **Por cada alumno:**
     - Crear `<tr>`.
     - Crear `<td>` con nombre, otro con edad, otro con matrícula.
     - Asignar `textContent` (no `innerHTML`) para evitar XSS.
     - Opcionalmente, un `<td>` con botones "Editar" / "Eliminar" (aunque eso viene en actividades posteriores).
     - Añadir el `<tr>` al `<tbody>`.
  4. **Caso arreglo vacío:** Mostrar una fila con un mensaje "No hay alumnos registrados" o un mensaje fuera de la tabla.

- **`documentFragment`:** Si esperas muchos registros, usa un fragmento para acumular las filas y hacer una sola inserción al DOM. Menos reflows, mejor rendimiento.

**Fragmento suelto (patrón):**
```js
const tbody = document.querySelector("#tabla-alumnos tbody");
tbody.innerHTML = "";
alumnos.forEach(a => {
    const tr = document.createElement("tr");
    const tdNombre = document.createElement("td");
    tdNombre.textContent = a.nombre;
    tr.appendChild(tdNombre);
    // ... edad, matrícula ...
    tbody.appendChild(tr);
});
```

- **Alternativa con `innerHTML`:** Concatenar todo el HTML de las filas y asignarlo de una vez. **Cuidado con XSS** si los datos vienen del usuario: mejor `textContent`.
- **Alternativa con `insertAdjacentHTML`:** Más explícito que `innerHTML`.

---

## 9. Botón "Actualizar"

- **Propósito:** Volver a consultar el servidor sin recargar la página.
- **Implementación:**
  1. Un `<button id="btn-actualizar">Actualizar</button>` en el HTML.
  2. Un listener `click` que llama a la función `cargarAlumnos()`.
  3. La función hace `fetch`, parsea y renderiza.
- **Refactorización:** Tener una sola función `cargarAlumnos()` que:
  - Haga el `fetch`.
  - Maneje errores.
  - Llame a `renderizarTabla(alumnos)`.
  - Actualice el estado del botón (deshabilitado mientras carga).

- **Al cargar la página:** Puedes llamar automáticamente a `cargarAlumnos()` en el evento `DOMContentLoaded` para que la tabla aparezca sin necesidad de pulsar el botón.

**Fragmento suelto (patrón):**
```js
document.querySelector("#btn-actualizar")
    .addEventListener("click", cargarAlumnos);

document.addEventListener("DOMContentLoaded", cargarAlumnos);
```

- **Feedback visual:** Mientras la petición está en curso, muestra "Cargando..." o deshabilita el botón. Evita clics múltiples.
- **Después de un POST exitoso** (registrar alumno), puedes llamar a `cargarAlumnos()` para que la tabla refleje el nuevo registro inmediatamente.

---

## 10. Manejo de errores y estados

- **Estados de la UI:**
  - **Cargando:** Mensaje "Cargando..." o spinner.
  - **Éxito con datos:** Tabla con filas.
  - **Éxito sin datos:** Mensaje "No hay alumnos registrados".
  - **Error:** Mensaje "Error al cargar. Intenta de nuevo."
- **Errores del servidor:** 500 → mensaje genérico. 404 → probablemente un bug de enrutamiento, loguea en consola.
- **Errores de red:** El servidor no responde → "Sin conexión con el servidor".
- **Errores de parseo:** El cuerpo no es JSON válido → "Respuesta inválida del servidor".
- **Nunca dejes la UI en blanco** sin indicar qué pasó. El usuario debe saber si está cargando, si no hay datos, o si hubo error.

---

## 11. Patrón "modelo + vista" en el cliente

- **Modelo:** El arreglo `alumnos` en JS (obtenido del servidor).
- **Vista:** La tabla HTML.
- **Función de renderizado:** `renderizarTabla(alumnos)` reconstruye el DOM a partir del modelo.
- **Ventaja:** Cualquier cambio (agregar, eliminar, actualizar) se refleja llamando a `cargarAlumnos()` de nuevo, sin tocar la tabla manualmente.
- **Sincronización:** El modelo viene siempre del servidor. El frontend **no** mantiene estado propio que pueda desincronizarse.

---

## 12. DevTools: verificar la petición

- **Network → Fetch/XHR:**
  - Verifica que la petición a `/api/alumnos` devuelve 200.
  - Revisa la pestaña **Response** o **Preview**: debe mostrar el JSON.
  - Si el JSON tiene un error de sintaxis, el navegador no podrá parsearlo y `.json()` fallará.
- **Console:**
  - Errores de JavaScript durante el renderizado.
  - Errores de `fetch` o de parseo.
- **Elementos:**
  - Inspecciona el `<tbody>` tras el renderizado para verificar que las filas se insertaron correctamente.
- **Servidor:**
  - Log de cada GET `/api/alumnos` con el número de registros devueltos.

---

## 13. Verificación integral

1. Levanta el servidor.
2. Comprueba en consola el log de carga de usuarios.
3. Abre la página en el navegador.
4. Verifica que la tabla aparece con los datos de `usuarios.txt`.
5. Registra un nuevo alumno (POST).
6. Pulsa "Actualizar": el nuevo alumno debe aparecer.
7. Detén el servidor y vuelve a arrancarlo.
8. Recarga la página: los datos deben seguir ahí.
9. Edita manualmente `usuarios.txt` (añade o quita una línea).
10. Pulsa "Actualizar": la tabla refleja el cambio.

---

## 14. Buenas prácticas

- **Delimitador y estructura fijos** entre frontend y backend: las claves del JSON deben coincidir exactamente con lo que el frontend espera.
- **`Content-Type: application/json; charset=utf-8`** siempre que devuelvas JSON.
- **Escapado correcto** de strings en el JSON. Sin esto, un nombre con comillas rompe la respuesta.
- **Arreglo vacío `[]`** si no hay datos, no 404.
- **`500` solo cuando hay error real** de lectura o escritura, no cuando el archivo no existe.
- **Ignorar líneas inválidas** del archivo en lugar de abortar.
- **`textContent` en el cliente,** no `innerHTML`, para evitar XSS.
- **Vaciar el `<tbody>` antes de renderizar** para evitar duplicados.
- **Feedback visual** durante la carga y en errores.
- **Botón deshabilitado** mientras hay una petición en curso.
- **Función única `cargarAlumnos`** reutilizable desde distintos botones y eventos.
- **Llamada automática al cargar la página** (`DOMContentLoaded`) para mostrar datos sin acción del usuario.
- **Considera añadir paginación** si el archivo crece mucho (fuera del alcance de esta actividad).
- **Cache-Control: no-store** en la respuesta si quieres evitar que el navegador cachee la lista.

---

## 15. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`.json()` falla con "Unexpected token"** | El JSON devuelto está mal formado. Revisa comas, comillas y escapado. |
| **La tabla aparece vacía aunque hay datos** | El selector del `<tbody>` está mal, o el renderizado no se llama tras el `fetch`. |
| **Los datos tienen tildes rotas** | Falta `charset=utf-8` en `Content-Type`, o el archivo no está en UTF-8. |
| **La tabla muestra `[object Object]`** | Asignaste el objeto entero a `textContent` en lugar de un campo. Usa `alumno.nombre`. |
| **Se duplican las filas al pulsar "Actualizar"** | No vacías el `<tbody>` antes de renderizar. |
| **El navegador cachea la respuesta y no ves cambios** | Añade `Cache-Control: no-store` o fuerza recarga con Ctrl+F5. |
| **404 en `/api/alumnos`** | No añadiste el caso al enrutador o el método no es GET. |
| **500 al leer el archivo** | La carpeta `data/` no existe o no tiene permisos. |
| **Devuelve HTML en lugar de JSON** | Falta el caso `/api/alumnos` y cae en el handler de archivos estáticos. |
| **`fetch` falla con CORS** | Sirve el frontend desde el mismo backend (puerto 8080). |
| **La respuesta se corta** | `Content-Length` mal calculado. Usa `.size()` del string JSON. |
| **Coma extra al final del JSON** | Verifica que solo añades coma **entre** elementos. |
| **Último objeto sin cerrar** | Revisa el orden de `{`, `}`, `,`. |
| **`edad` sale entre comillas** | Estás envolviendo el int en comillas al construir el JSON. Debe ser `"edad":20`, no `"edad":"20"`. |
| **Error al recargar la página tras POST** | El navegador reenvía el POST. Ya no aplica si usas `fetch` (no recarga). |
| **No se actualiza tras registrar** | No llamas a `cargarAlumnos()` tras el POST exitoso. |

---

## 16. Resumen de conceptos clave

- **Flujo completo:** Archivo TXT → servidor C++ → JSON → respuesta HTTP → `fetch` → tabla HTML.
- **Ruta `GET /api/alumnos`:** Lee el archivo, construye arreglo JSON, responde.
- **Lectura:** `ifstream` + `getline` + `stringstream` + `getline(ss, campo, '|')`.
- **JSON manual:** Recorrer vector, escapar strings, comas entre objetos, sin coma final.
- **Escapado:** `"` → `\"`, `\` → `\\`, `\n` → `\n`. Fundamental para no romper el JSON.
- **Respuesta:** `Content-Type: application/json; charset=utf-8`, `Content-Length` correcto.
- **Cliente:** `fetch("/api/alumnos")` → `.json()` → renderizar tabla.
- **Renderizado:** Vaciar `<tbody>`, crear `<tr>`/`<td>`, `textContent`, insertar.
- **Botón "Actualizar":** Listener que llama a `cargarAlumnos()`.
- **Carga automática:** `DOMContentLoaded` → `cargarAlumnos()`.
- **Manejo de errores:** try-catch, `res.ok`, mensajes al usuario.
- **Patrón modelo+vista:** El modelo viene del servidor; la vista se reconstruye cada vez.
- **Arreglo vacío:** `[]` con 200, no 404.
- **Buenas prácticas:** UTF-8, `textContent`, feedback visual, escapado de JSON, función reutilizable.

---
# PASO A PASO

---
# Actividad 30 — Mostrar información almacenada

## Objetivo
Crear `GET /api/alumnos` que devuelva **JSON** con todos los alumnos. Desde el frontend, consumirlo con `fetch()` y construir una tabla HTML dinámicamente.

## Flujo completo

```
usuarios.txt → servidor C++ → JSON → respuesta HTTP → fetch → tabla HTML
```

---

## Paso 1 — Estructura del JSON de salida

```json
[
  {"nombre":"Juan","edad":20,"matricula":"A001"},
  {"nombre":"María","edad":21,"matricula":"A002"}
]
```

- Cada alumno = un objeto.
- Todo el arreglo = la respuesta.

---

## Paso 2 — Función: escapar strings para JSON

```cpp
string escaparJSON(const string& s) {
    string r;
    for (char c : s) {
        switch (c) {
            case '"':  r += "\\\""; break;
            case '\\': r += "\\\\"; break;
            case '\n': r += "\\n";  break;
            case '\r': r += "\\r";  break;
            case '\t': r += "\\t";  break;
            default:   r += c;
        }
    }
    return r;
}
```

🎯 Sin esto, un nombre con comillas rompe el JSON entero.

---

## Paso 3 — Función: vector → JSON

```cpp
string alumnosAJSON() {
    string json = "[";
    for (size_t i = 0; i < alumnos.size(); i++) {
        if (i > 0) json += ",";
        const auto& a = alumnos[i];
        json += "{";
        json += "\"nombre\":\""    + escaparJSON(a.nombre)    + "\",";
        json += "\"edad\":"        + to_string(a.edad)        + ",";
        json += "\"matricula\":\"" + escaparJSON(a.matricula) + "\"";
        json += "}";
    }
    json += "]";
    return json;
}
```

⚠️ Observa:
- `"edad":20` → **sin comillas** (es número).
- Sin coma tras el último objeto.

---

## Paso 4 — Handler GET /api/alumnos

```cpp
if (ruta == "/api/alumnos" && metodo == "GET") {
    string json = alumnosAJSON();
    cout << "[GET] /api/alumnos → " << alumnos.size() << " registros\n";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

⚠️ Si `alumnos` está vacío, el JSON es `[]`. Eso NO es un 404.

---

## Paso 5 — Tabla en el HTML

En `public/index.html`:

```html
<section id="lista">
  <h2>Alumnos registrados</h2>
  <button type="button" id="btn-actualizar">Actualizar</button>

  <table id="tabla-alumnos">
    <thead>
      <tr>
        <th>Matrícula</th>
        <th>Nombre</th>
        <th>Edad</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <p id="tabla-estado"></p>
</section>
```

---

## Paso 6 — Consumir con fetch

En `app.js`:

```js
const tbody        = document.querySelector('#tabla-alumnos tbody');
const tablaEstado  = document.querySelector('#tabla-estado');
const btnActualizar = document.querySelector('#btn-actualizar');

async function cargarAlumnos() {
  tablaEstado.textContent = 'Cargando...';
  btnActualizar.disabled = true;

  try {
    const res = await fetch('/api/alumnos');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);

    const alumnos = await res.json();
    renderizarTabla(alumnos);

    tablaEstado.textContent = `${alumnos.length} alumno(s)`;
  } catch (err) {
    tablaEstado.textContent = `Error: ${err.message}`;
    console.error(err);
  } finally {
    btnActualizar.disabled = false;
  }
}

function renderizarTabla(alumnos) {
  tbody.innerHTML = '';

  if (alumnos.length === 0) {
    const fila = document.createElement('tr');
    const td   = document.createElement('td');
    td.colSpan = 3;
    td.textContent = 'No hay alumnos registrados';
    td.className = 'vacio';
    fila.appendChild(td);
    tbody.appendChild(fila);
    return;
  }

  alumnos.forEach((a) => {
    const tr = document.createElement('tr');

    const tdMat = document.createElement('td');
    tdMat.textContent = a.matricula;

    const tdNom = document.createElement('td');
    tdNom.textContent = a.nombre;

    const tdEdad = document.createElement('td');
    tdEdad.textContent = a.edad;

    tr.appendChild(tdMat);
    tr.appendChild(tdNom);
    tr.appendChild(tdEdad);

    tbody.appendChild(tr);
  });
}

btnActualizar.addEventListener('click', cargarAlumnos);
document.addEventListener('DOMContentLoaded', cargarAlumnos);
```

🎯 `textContent` en lugar de `innerHTML` → evita XSS.

---

## Paso 7 — Refrescar tras un POST

En el listener del `submit` (actividad 27), después de éxito:

```js
formAlumno.reset();
await cargarAlumnos();   // ← refresca la tabla
```

---

## Paso 8 — Verificar todo

1. Levanta el servidor.
2. Abre `http://localhost:8080/`.
3. Al cargar, la tabla aparece con los datos de `usuarios.txt`.
4. Registra un alumno nuevo → la tabla se actualiza automáticamente.
5. Pulsa **Actualizar** → vuelve a pedir la lista al servidor.
6. Abre `usuarios.txt` y edítalo a mano (borra una línea). Pulsa Actualizar → la tabla refleja el cambio.
7. Reinicia el servidor → los datos siguen ahí.

---

## Paso 9 — Ver en DevTools

DevTools → Network → filtro Fetch/XHR:
- `GET /api/alumnos` → 200.
- Response → el JSON con los alumnos.

Y en **Preview** puedes ver el JSON en forma de árbol.

---

## ✅ Checklist de la Actividad 30

- [ ] `GET /api/alumnos` responde `application/json`.
- [ ] El JSON tiene la forma `[{"nombre":...,"edad":...,"matricula":...}, ...]`.
- [ ] La tabla se llena al abrir la página.
- [ ] El botón Actualizar refresca sin recargar.
- [ ] Registrar un alumno refresca la tabla automáticamente.
- [ ] Se usa `textContent` para insertar valores.

## 🧠 Mini-reto
Muestra en un `<div>` aparte el total de alumnos y el promedio de edades, calculados en el cliente con los datos del JSON.

---

