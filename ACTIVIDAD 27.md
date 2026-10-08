# Actividad 27. Enviando datos al servidor

## Temas

- fetch()
    
- POST
    

## Ejercicio

Tomar el formulario de registro.

Enviar sus datos mediante:

```javascript
fetch(...)
```

en formato JSON.

Verificar que el servidor reciba correctamente la petición.

Todavía no se almacena información.

---
# TEORÍA

---

## 1. ¿Por qué POST y no GET?

- **GET:** Los datos viajan en la URL (query string). Visibles, limitados, no aptos para modificar estado.
- **POST:** Los datos viajan en el **cuerpo** de la petición. Ocultos en la URL, sin límite práctico, aptos para crear/modificar recursos.
- **Regla:** Si el envío **crea o modifica** algo en el servidor, usa **POST**.
- **En esta actividad:** Estás registrando un alumno → POST.

---

## 2. Formato de los datos: JSON vs. formulario tradicional

### A. Formato tradicional (`application/x-www-form-urlencoded`)

- Es el formato por defecto de un `<form>` HTML.
- Los datos van como `clave1=valor1&clave2=valor2`.
- Cómodo si el navegador envía el formulario directamente (sin JavaScript).
- **Desventajas:** Todo son strings, sin tipos, sin estructuras anidadas.

### B. Formato JSON (`application/json`)

- Los datos van como un string JSON: `{"nombre":"Ana","edad":20}`.
- **Ventajas:**
  - Tipos nativos (números, booleanos, null).
  - Estructuras anidadas (objetos dentro de objetos, arrays).
  - Estándar en APIs modernas.
- **Desventajas:** Requiere serializar con `JSON.stringify` y parsear en el servidor.
- **En esta actividad:** Usarás JSON porque es el formato dominante en APIs y porque en el backend ya conoces cómo parsearlo.

---

## 3. `fetch()` con POST: configuración

- **Firma:** `fetch(url, opciones)`
- **`opciones` para POST:**
  - **`method: "POST"`:** Obligatorio. Sin esto, `fetch` usa GET por defecto.
  - **`headers`:** Al menos `Content-Type: application/json` para que el servidor sepa qué formato viene.
  - **`body`:** Los datos serializados. Para JSON, `JSON.stringify(objeto)`.
- **Sin `body`, el POST no envía nada.**

**Fragmento suelto (esqueleto):**
```js
fetch("/api/alumnos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(datos)
});
```

- **`headers` también puede incluir** otros como `Accept: application/json` (indica al servidor qué formato de respuesta prefieres).
- **No necesitas poner `Content-Length`:** El navegador lo calcula automáticamente.
- **No necesitas `Host`:** El navegador lo añade.

---

## 4. Serialización del formulario

- **Objetivo:** Convertir los campos del formulario HTML en un objeto JS y luego en JSON.
- **Fuente de los datos:**
  - Leer cada input con `document.querySelector("#id").value`.
  - O usar `new FormData(formulario)` y convertirlo a objeto.
- **Consideración de tipos:** Los valores de `<input>` son **siempre strings**. Si un campo es numérico (edad), conviértelo con `Number()` antes de meterlo al objeto.
- **Construcción del objeto:**
  - Crea un objeto plano con las claves que espera el servidor.
  - Las claves del objeto deben coincidir con las que el backend espera (por ejemplo, `nombre`, `edad`, `matricula`).

**Fragmentos sueltos:**
```js
const datos = {
    nombre: document.querySelector("#nombre").value,
    edad: Number(document.querySelector("#edad").value),
    matricula: document.querySelector("#matricula").value
};
```

- **Con FormData (alternativa):**
  ```js
  const formData = new FormData(formulario);
  const datos = Object.fromEntries(formData.entries());
  ```
  - **Ojo:** `Object.fromEntries` no convierte tipos; todo queda como string.
  - `FormData` es más útil cuando hay archivos (`<input type="file">`).

---

## 5. Interceptar el envío del formulario

- **Problema:** Por defecto, un formulario HTML al enviarse **recarga la página** y envía los datos como `application/x-www-form-urlencoded`.
- **Solución:** Interceptar el evento `submit`, llamar a `event.preventDefault()` y manejar el envío manualmente con `fetch`.

**Fragmento suelto (patrón):**
```js
formulario.addEventListener("submit", async (e) => {
    e.preventDefault();
    // construir datos
    // fetch POST
});
```

- **Escuchar `submit` en el `<form>`**, no `click` en el botón. Así funciona también si el usuario pulsa Enter.
- **`e.preventDefault()`** debe ir **al inicio** del listener.

---

## 6. Envío y manejo de la respuesta

- **Servidor responde:** Después del POST, el servidor devuelve una respuesta. Puede ser:
  - **200 OK** con un mensaje de éxito.
  - **201 Created** si se creó un recurso.
  - **400 Bad Request** si los datos están mal.
  - **500 Internal Server Error** si algo falló.
- **Verifica `response.ok`** antes de leer el cuerpo.
- **Lee la respuesta según el Content-Type:**
  - Si es JSON, `await response.json()`.
  - Si es texto, `await response.text()`.

**Fragmento suelto (patrón):**
```js
const res = await fetch("/api/alumnos", { ... });
if (!res.ok) {
    // error HTTP
    throw new Error(`HTTP ${res.status}`);
}
const resultado = await res.json();
```

- **Actualiza el DOM** con el mensaje de éxito o error.
- **Maneja errores con `try-catch`** (red, parseo, lógica propia).
- **Deshabilita el botón** mientras se envía, para evitar dobles envíos.

---

## 7. Manejo de errores y validación

- **Validación en cliente (JS):**
  - Verificar campos vacíos antes de enviar.
  - Verificar tipos (que edad sea número).
  - Mostrar mensajes al usuario.
- **Validación en servidor (C++):**
  - **Obligatoria.** El cliente puede mentir o estar manipulado.
  - Verificar que los campos esperados existan y tengan el tipo correcto.
  - Responder 400 si algo falta o es inválido.
- **Errores a manejar en JS:**
  - **Red:** `fetch` rechaza la promesa (servidor caído, sin conexión).
  - **HTTP:** `response.ok === false`.
  - **JSON malformado:** `response.json()` lanza error.
  - **Falta de campos:** El backend puede responder 400 con detalle.

**Fragmento suelto (estructura de manejo de errores):**
```js
try {
    const res = await fetch(...);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    // éxito
} catch (err) {
    // error de red, HTTP o parseo
}
```

---

## 8. Servidor C++: recepción del POST

Esto es clave: tu servidor **aún no procesa** los datos (eso viene en la próxima actividad), pero sí necesita **leerlos** para verificar que llegaron. Conceptos:

- **`recv()`:** Lee bytes del socket. Puede que el POST llegue en **varios paquetes**. Necesitas leer hasta tener todas las cabeceras y el cuerpo completo.
- **Detección del fin de cabeceras:** Busca la secuencia `\r\n\r\n`. Todo lo que hay **después** es el cuerpo.
- **`Content-Length`:** Cabecera que indica cuántos bytes tiene el cuerpo. Sin ella, no sabes cuándo terminar de leer.
- **Estrategia de lectura:**
  1. Leer con `recv` en un bucle hasta encontrar `\r\n\r\n`.
  2. Parsear el `Content-Length` de las cabeceras.
  3. Seguir leyendo hasta acumular `Content-Length` bytes de cuerpo.
  4. Concatenar el cuerpo en un string.
- **Para esta actividad:** El cuerpo es un JSON pequeño, así que probablemente llega en una sola llamada a `recv`. Aun así, debes verificar que lo recibido incluya el cuerpo completo.

**Fragmentos sueltos (conceptuales):**
```cpp
size_t pos = peticion.find("\r\n\r\n");
if (pos != string::npos) {
    string cabeceras = peticion.substr(0, pos);
    string cuerpo = peticion.substr(pos + 4);
}
```

- **Parsear `Content-Length`:**
  - Busca `"Content-Length: "` en las cabeceras.
  - Toma el número hasta el final de la línea.
  - Conviértelo a `int` con `stoi`.
- **Cuerpo JSON:** Es un string en formato JSON. Aún no lo parseas (eso será en la próxima actividad); solo lo recibes y verificas que no esté vacío.
- **Log de depuración:** Imprime en consola el cuerpo recibido para confirmar que llegó completo y bien formado.

**Consideración sobre el Content-Type:**
- El servidor debería verificar que `Content-Type: application/json` está presente. Si no, podría rechazar con 415 Unsupported Media Type.
- En esta actividad, para simplificar, puedes aceptar cualquier Content-Type pero **leer el cuerpo** de todas formas.

---

## 9. Respuestas del servidor

- **Éxito:**
  - Código `200 OK` o `201 Created`.
  - `Content-Type: application/json` (más moderno) o `text/plain`.
  - Cuerpo: un mensaje o un JSON de confirmación.
  - Ejemplo conceptual: `{"ok":true,"mensaje":"Alumno recibido"}`.
- **Error de cliente:**
  - Código `400 Bad Request` si falta algún campo o el JSON está malformado.
  - Cuerpo: `{"ok":false,"error":"Falta el campo nombre"}`.
- **Error de servidor:**
  - Código `500 Internal Server Error` si algo falla internamente.
- **CORS:** No aplica en esta actividad porque frontend y backend comparten origen.

---

## 10. DevTools → Network: verificar la petición

- **Abrir DevTools** antes de enviar el formulario.
- **Pestaña Network**, filtro "Fetch/XHR".
- Al enviar, aparece la petición POST. Haz clic para ver:
  - **Headers:** Método POST, URL, `Content-Type: application/json`.
  - **Payload / Request:** El cuerpo enviado (el JSON serializado).
  - **Response:** Lo que respondió el servidor.
  - **Status:** 200, 400, 500...
- **Console:** Errores de red o excepciones del `try-catch`.
- **Server logs:** En la terminal donde corre el servidor C++, verifica que el cuerpo llegó completo.

**Verificación cruzada:** Lo que ves en `Payload` de DevTools debe coincidir con lo que imprime el servidor en consola.

---

## 11. Diferencias con la Actividad 23 (formulario HTML tradicional)

| Aspecto | Formulario HTML tradicional | `fetch` con POST JSON |
|---------|----------------------------|------------------------|
| Recarga la página | Sí | No |
| Formato de envío | `application/x-www-form-urlencoded` | `application/json` |
| Tipos de datos | Todo string | Números, booleanos, anidados |
| Control del JS | Mínimo | Total |
| Manejo de respuesta | El navegador reemplaza la página | El JS decide qué hacer |
| Validación | Nativa HTML | Nativa + JS |
| Errores | Difíciles de manejar | try-catch, response.ok |
| UX | Recarga completa | Sin recarga, fluido |

- **Ventaja de `fetch`:** Control total, sin recargas, feedback inmediato, mejor UX.
- **Ventaja del tradicional:** Menos JavaScript, más simple si solo necesitas enviar y recargar.

---

## 12. Buenas prácticas

- **Escucha `submit`, no `click`**, para capturar también Enter.
- **`e.preventDefault()`** siempre al inicio.
- **Serializa con `JSON.stringify`**, no envíes objetos directamente.
- **`Content-Type: application/json`** en las cabeceras del `fetch`.
- **Convierte tipos numéricos** en el cliente antes de enviar (o valida en el servidor).
- **Valida en cliente Y servidor.** El cliente para UX, el servidor para seguridad.
- **Maneja errores con `try-catch`** y verifica `response.ok`.
- **Feedback visual:** "Enviando...", "Éxito", "Error". Nunca dejes al usuario sin saber qué pasó.
- **Deshabilita el botón** durante el envío. Evita duplicados.
- **Limpia el formulario** tras un envío exitoso (opcional pero buena UX).
- **No muestres mensajes del backend directamente con `innerHTML`.** Usa `textContent` para evitar XSS.
- **Log en el servidor:** Imprime el cuerpo recibido para verificar en desarrollo.
- **Verifica en DevTools** que el Payload coincide con lo que construiste.
- **Documenta el formato esperado** por el backend (qué claves, qué tipos) en el README.
- **Sé consistente:** si el backend espera `matricula`, no envíes `matrícula` ni `mat`.
- **Rechaza `Content-Type` incorrecto** en el servidor (opcional pero recomendable).

---

## 13. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **La página se recarga al enviar** | Falta `e.preventDefault()` en el listener `submit`. |
| **El servidor no recibe el cuerpo** | Falta `body: JSON.stringify(datos)` en el `fetch`. |
| **El servidor recibe `[object Object]`** | Enviaste el objeto sin serializar. Usa `JSON.stringify`. |
| **El servidor no sabe qué formato viene** | Falta `Content-Type: application/json`. |
| **El cuerpo llega cortado** | El servidor no leyó hasta completar `Content-Length`. Recibe en bucle. |
| **`response.json()` falla** | El servidor no devolvió JSON válido. Usa `.text()` si es texto plano. |
| **Solo leo el cuerpo la mitad** | El `recv` no incluyó todo. Necesitas leer hasta el `Content-Length`. |
| **Todo string en el servidor** | El cliente envió números como string o el servidor no parsea el JSON (eso viene después). |
| **CORS blocked** | Estás sirviendo el frontend desde otro puerto. Accede por el mismo backend. |
| **"Body already read"** | Solo puedes leer `response` una vez. Guarda el resultado. |
| **`await` fuera de función async** | Envuelve el listener en `async` o usa `.then`. |
| **Payload en DevTools no coincide con lo que esperaba** | Revisa cómo construiste el objeto antes de serializar. |
| **El servidor imprime todo el POST como headers** | No buscaste `\r\n\r\n` para separar cabeceras del cuerpo. |
| **Falta una cabecera en el servidor** | La parseaste mal. Cuidado con `\r\n` vs `\n`. |
| **Error 400 inesperado** | El servidor valida campos que no enviaste, o espera nombres distintos. |

---

## 14. Flujo completo

1. Usuario rellena el formulario.
2. Pulsa "Enviar".
3. Se dispara el evento `submit`.
4. `e.preventDefault()` evita la recarga.
5. JS lee los valores de los inputs.
6. Construye un objeto JS con las claves que espera el backend.
7. `JSON.stringify` lo convierte a string.
8. `fetch` con POST + headers + body envía la petición.
9. El navegador abre TCP al `localhost:8080`.
10. Envía la petición HTTP con las cabeceras y el cuerpo.
11. El servidor C++ recibe, parsea, y (por ahora) loguea el cuerpo.
12. El servidor responde con un mensaje de confirmación.
13. JS recibe la respuesta, verifica `ok`, lee el cuerpo.
14. JS actualiza el DOM con el mensaje.
15. Opcionalmente limpia el formulario y reactiva el botón.

---

## 15. Resumen de conceptos clave

- **POST:** Método para crear/modificar. Datos en el cuerpo.
- **JSON:** Formato de intercambio. Serializar con `JSON.stringify`.
- **`fetch` POST:** `method`, `headers` (`Content-Type: application/json`), `body`.
- **Interceptar submit:** `addEventListener("submit", ...)` + `e.preventDefault()`.
- **Construir datos:** Leer inputs, convertir tipos, construir objeto.
- **Respuesta:** Verificar `response.ok`, leer con `.json()` o `.text()`.
- **Errores:** `try-catch`, manejo de red y HTTP.
- **Servidor C++:** Leer hasta `\r\n\r\n`, parsear `Content-Length`, leer el cuerpo hasta completar. Loguear para verificar.
- **DevTools Network:** Verificar Payload, Response y Status.
- **Validación:** En cliente y servidor. Nunca confíes en el cliente.
- **Feedback visual:** Cargando, éxito, error. Deshabilitar botón.
- **Aún no se almacena:** Solo se verifica que el servidor reciba el cuerpo. La persistencia viene después.

---
# PASO A PASO

---
# Actividad 27 — Enviando datos al servidor (POST con fetch)

## Objetivo
Enviar el formulario de registro al servidor con `fetch()` y formato **JSON**, sin que la página se recargue.

## Concepto clave
En vez del envío tradicional (`application/x-www-form-urlencoded` que recarga la página), usamos `fetch` con `Content-Type: application/json` y `JSON.stringify`.

---

## Paso 1 — Ajustar el formulario

En `public/index.html`, el form de registro ya debe existir. Verifica que tenga un `id`:

```html
<form id="form-alumno">
  <label for="nombre">Nombre:</label>
  <input type="text" id="nombre" name="nombre" required>

  <label for="edad">Edad:</label>
  <input type="number" id="edad" name="edad" min="0" max="120" required>

  <label for="matricula">Matrícula:</label>
  <input type="text" id="matricula" name="matricula" required>

  <button type="submit">Registrar</button>
</form>
```

⚠️ Fíjate: **sin `action` ni `method`**. Los maneja JS.

---

## Paso 2 — Interceptar el envío

En `app.js`:

```js
const formAlumno = document.querySelector('#form-alumno');
const apiResultado = document.querySelector('#api-resultado');

formAlumno.addEventListener('submit', async (e) => {
  e.preventDefault();   // evita la recarga

  const datos = {
    nombre:    document.querySelector('#nombre').value.trim(),
    edad:      Number(document.querySelector('#edad').value),
    matricula: document.querySelector('#matricula').value.trim()
  };

  // Validación mínima en cliente
  if (!datos.nombre || !datos.edad || !datos.matricula) {
    apiResultado.textContent = 'Completa todos los campos.';
    apiResultado.classList.add('error');
    return;
  }

  try {
    const res = await fetch('/api/alumnos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(datos)
    });

    if (!res.ok) throw new Error(`HTTP ${res.status}`);

    const texto = await res.text();
    apiResultado.textContent = `Servidor respondió: ${texto}`;
    apiResultado.classList.remove('error');

    formAlumno.reset();
  } catch (err) {
    apiResultado.textContent = `Error: ${err.message}`;
    apiResultado.classList.add('error');
  }
});
```

🎯 **Tres piezas obligatorias del POST:**
1. `method: 'POST'`
2. `headers: { 'Content-Type': 'application/json' }`
3. `body: JSON.stringify(datos)`

---

## Paso 3 — Ruta temporal en el servidor

El servidor de la actividad 24 no tiene `/api/alumnos`. Para probar sin modificar el servidor, agrega temporalmente un endpoint que **solo devuelva OK** (en la actividad 28 lo haremos de verdad):

```cpp
if (ruta == "/api/alumnos" && metodo == "POST") {
    return construirRespuesta(200, "text/plain; charset=utf-8",
                              "Alumno recibido (pero no procesado)");
}
```

---

## Paso 4 — Probar

1. Abre `http://localhost:8080/`.
2. Rellena el formulario.
3. Pulsa **Registrar**.
4. Debe aparecer: `Servidor respondió: Alumno recibido (pero no procesado)`.
5. El formulario se limpia.
6. **La página no se recarga.**

---

## Paso 5 — Verificar en DevTools

DevTools → Network → filtro Fetch/XHR.

- Aparece `POST /api/alumnos`.
- **Headers** → `Content-Type: application/json`.
- **Payload** → el JSON enviado:
  ```json
  {"nombre":"Ana","edad":20,"matricula":"A001"}
  ```
- **Response** → el mensaje del servidor.

🎯 Compara con la actividad 23: allí el cuerpo era `nombre=Ana&edad=20&...` (urlencoded). Aquí es JSON.

---

## Paso 6 — Manejo de errores y limpieza

```js
finally {
  document.querySelector('#btn-registrar').disabled = false;
}
```

Cambia el botón para tener `id`:

```html
<button type="submit" id="btn-registrar">Registrar</button>
```

Y al inicio del listener:

```js
const boton = document.querySelector('#btn-registrar');
boton.disabled = true;
```

---

## ✅ Checklist de la Actividad 27

- [ ] El formulario NO recarga la página al enviarse.
- [ ] En DevTools aparece `POST /api/alumnos` con `Content-Type: application/json`.
- [ ] En Payload se ve el JSON con los datos.
- [ ] Tras éxito, el formulario se limpia.
- [ ] Los campos vacíos muestran un mensaje sin llegar al servidor.

## 🧠 Mini-reto
Añade un campo "correo" y envíalo junto con los demás. Actualiza el objeto `datos` y prueba que viaje en el Payload.

---

