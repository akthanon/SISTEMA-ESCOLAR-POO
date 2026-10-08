# Actividad 26. Primer fetch()

## Temas

- fetch()
    
- GET
    

## Ejercicio

Desde JavaScript consumir:

```text
/api/hola
```

Mostrar la respuesta dentro de la página web sin recargar el navegador.

Agregar otros botones para consultar:

- Hora
    
- Versión
    
- Estado del servidor
    

---
# TEORÍA

---

## 1. ¿Qué es `fetch()`?

- **Definición:** `fetch()` es una función **nativa del navegador** (no de JavaScript como lenguaje) que permite hacer **peticiones HTTP** desde JavaScript sin recargar la página.
- **Sustituye a:** `XMLHttpRequest` (la forma antigua, más verbosa).
- **Basada en promesas:** Devuelve una `Promise` que se resuelve con la respuesta del servidor.
- **Asíncrona por naturaleza:** No bloquea la ejecución del resto del código mientras espera la respuesta.
- **Función en el proyecto:** Permite al frontend consultar tu API en C++ (`/api/hola`, `/api/hora`, `/api/version`, `/api/status`) y mostrar el resultado en la página.

---

## 2. Asincronía: promesas y `async/await`

- **Problema:** Las peticiones de red tardan tiempo. Bloquear el navegador mientras esperas sería terrible.
- **Solución:** Código asíncrono. La petición se lanza, y cuando llega la respuesta, se ejecuta el código que la maneja.
- **Promise:** Objeto que representa un valor que **estará disponible en el futuro**. Estados:
  - **pending:** Aún no terminó.
  - **fulfilled:** Terminó con éxito (tiene un valor).
  - **rejected:** Terminó con error.
- **Métodos de una promesa:**
  - `.then(callback)` → se ejecuta si se cumple.
  - `.catch(callback)` → se ejecuta si falla.
  - `.finally(callback)` → se ejecuta siempre, al final.
- **`async/await`:** Azúcar sintáctico sobre promesas, más legible.
  - `async` antes de una función: la función devuelve una promesa.
  - `await` dentro de la función: pausa la ejecución hasta que la promesa se resuelva.
  - **Solo se puede usar `await` dentro de funciones `async`.**
  - Los errores se manejan con `try-catch` (en lugar de `.catch`).

**Fragmentos sueltos:**
```js
// Estilo promesas
fetch(url).then(res => res.text()).then(data => { /* usar data */ });

// Estilo async/await
async function cargar() {
    const res = await fetch(url);
    const data = await res.text();
    // usar data
}
```

**Recomendación:** Usa `async/await` para claridad.

---

## 3. Sintaxis básica de `fetch()`

- **Firma simplificada:** `fetch(url, opciones)`
- **Parámetros:**
  - `url`: ruta absoluta o relativa. En tu caso, rutas relativas como `/api/hola` funcionan porque el frontend y el backend están en el **mismo origen** (mismo host, mismo puerto).
  - `opciones` (opcional): objeto con método, cabeceras, cuerpo, etc. Si se omite, el método por defecto es **GET**.
- **Devuelve:** Una `Promise` que se resuelve con un objeto **Response**.
- **Importante:** `fetch` **no rechaza la promesa por errores HTTP** (404, 500, etc.). Solo rechaza por **errores de red** (sin conexión, DNS fallido). Para detectar errores HTTP, verifica `response.ok` o `response.status`.

---

## 4. El objeto `Response`

Una vez que `fetch` se resuelve, obtienes un objeto `Response` con:

- **Propiedades útiles:**
  - `response.ok` → `true` si el código está entre 200 y 299.
  - `response.status` → el código HTTP (200, 404, 500...).
  - `response.statusText` → texto del código ("OK", "Not Found").
  - `response.headers` → cabeceras de la respuesta (acceso tipo `Map`).
  - `response.url` → la URL final de la respuesta (por si hubo redirecciones).
- **Métodos para leer el cuerpo** (¡solo puedes leer el cuerpo **una vez**!):
  - `response.text()` → devuelve el cuerpo como string (Promise).
  - `response.json()` → parsea el cuerpo como JSON (Promise). **Falla si no es JSON válido.**
  - `response.blob()` → para archivos binarios.
  - `response.arrayBuffer()` → para datos binarios de bajo nivel.
  - `response.formData()` → para formularios multipart.

**Regla práctica:** Si tu API devuelve `text/plain`, usa `.text()`. Si devuelve `application/json`, usa `.json()`.

---

## 5. Métodos HTTP con `fetch()`

- **GET (por defecto):**
  - No requiere configuración. Basta con `fetch(url)`.
  - Los parámetros van en la URL: `/api/buscar?nombre=Ana`.
- **POST:**
  - Requiere especificar el método y el cuerpo.
  - El cuerpo debe ser un string (típicamente `JSON.stringify(datos)`).
  - Hay que indicar `Content-Type` en las cabeceras.
- **Otros métodos:** `PUT`, `PATCH`, `DELETE`. Misma idea, cambiar `method`.

**Fragmento suelto (POST conceptual):**
```js
fetch("/api/alumnos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(alumno)
});
```

- **Para esta actividad:** Solo necesitas **GET**, así que `fetch("/api/hola")` es suficiente.

---

## 6. Manejo de errores

- **Errores de red:** `fetch` rechaza la promesa (servidor caído, sin internet). Se manejan con `.catch` o `try-catch`.
- **Errores HTTP:** `fetch` **no rechaza**. La promesa se resuelve con `response.ok === false`. Debes verificarlo manualmente.
- **Errores de parseo:** Si llamas a `response.json()` sobre algo que no es JSON, lanza error.
- **Errores de lectura del cuerpo:** Solo puedes leer el cuerpo una vez. Si lo lees dos veces, error.
- **Errores de CORS:** Si el frontend y el backend están en **orígenes distintos**, el navegador bloquea la respuesta (ver sección 9).

**Patrón recomendado:**
```js
try {
    const res = await fetch(url);
    if (!res.ok) {
        // manejar error HTTP (404, 500...)
        throw new Error(`HTTP ${res.status}`);
    }
    const data = await res.text();
    // usar data
} catch (err) {
    // error de red o de la lógica anterior
}
```

---

## 7. Actualizar el DOM con la respuesta

- **Objetivo:** Mostrar el resultado de la petición en la página, **sin recargar**.
- **Herramientas:** Los mismos métodos del DOM que ya conoces:
  - `document.querySelector(...)` para seleccionar el elemento destino.
  - `elemento.textContent = data;` para texto plano.
  - `elemento.innerHTML = ...` si vas a inyectar HTML (cuidado con XSS).
- **Patrón típico:**
  1. El usuario pulsa un botón.
  2. Se lanza `fetch` a la API correspondiente.
  3. Cuando llega la respuesta, se actualiza el DOM con el contenido.
  4. Opcionalmente, se muestra un mensaje de "Cargando..." mientras tanto.

**Fragmento suelto (patrón):**
```js
boton.addEventListener("click", async () => {
    const res = await fetch("/api/hola");
    const texto = await res.text();
    document.querySelector("#resultado").textContent = texto;
});
```

- **Mensaje de carga:** Antes del `await`, escribe "Cargando..." en el DOM. Después, reemplázalo con la respuesta. Mejora la UX.
- **Deshabilitar el botón** durante la petición para evitar clics múltiples.

---

## 8. `fetch` y mismo origen

- **Mismo origen (same-origin):** Mismo protocolo, mismo host, mismo puerto. Ejemplo: frontend en `http://localhost:8080` y backend en `http://localhost:8080`. **Sin problemas.**
- **Orígenes distintos:** Frontend en `http://localhost:5500` (Live Server) y backend en `http://localhost:8080`. **El navegador aplica CORS** y bloquea por defecto.
- **En esta actividad:** Como el backend en C++ sirve **tanto el HTML como la API** en el mismo puerto (8080), **no hay problema de CORS**. El navegador considera que todo viene del mismo origen.
- **Recomendación:** Servir siempre el frontend desde el backend (como ya haces) para evitar CORS. Si usas Live Server de VSCode, **no** lo hagas en esta actividad; accede por `http://localhost:8080`.

---

## 9. CORS (Cross-Origin Resource Sharing)

- **Definición:** Política de seguridad del navegador que impide que una página de un origen haga peticiones a otro origen **sin permiso explícito** del servidor destino.
- **Cuándo aparece:** Frontend en `http://localhost:5500` → backend en `http://localhost:8080` (distinto puerto). También al abrir el HTML con doble clic (`file://`) y consultar a `http://localhost:8080`.
- **Síntomas:** En la consola del navegador verás un error tipo `Access to fetch at ... has been blocked by CORS policy`.
- **Solución en el servidor:** Añadir cabeceras a la respuesta:
  - `Access-Control-Allow-Origin: *` (o el origen específico).
  - `Access-Control-Allow-Methods: GET, POST, ...`
  - `Access-Control-Allow-Headers: Content-Type, ...`
- **Solución alternativa:** Servir el frontend desde el mismo backend (lo que ya haces). **Cero configuración CORS.**
- **Para esta actividad:** No deberías topar con CORS porque todo vive en `localhost:8080`. Si aparece, revisa que realmente estés accediendo por la URL del servidor C++ y no por `file://` o por Live Server.

---

## 10. DevTools → Network: observar las peticiones

- **Abrir:** F12 → pestaña **Network**.
- **Filtrar:** Por "Fetch/XHR" para ver solo las peticiones de `fetch`.
- **Al pulsar un botón:**
  - Aparece una nueva fila con la URL `/api/...`.
  - **Headers:** Método (GET), URL, cabeceras.
  - **Response:** El cuerpo devuelto por el servidor.
  - **Preview:** Vista previa si es JSON.
  - **Timing:** Cuánto tardó la petición.
- **Estados:**
  - Verde (200): OK.
  - Rojo (4xx, 5xx): Error. Haz clic para ver el detalle.
  - Sin conexión: `(failed)` o `(canceled)`.

**Consejo:** Ten DevTools abierto mientras pruebas los botones. Es la mejor forma de depurar.

---

## 11. Buenas prácticas

- **Usa `async/await`** en lugar de `.then` encadenados cuando la lógica crezca.
- **Verifica `response.ok`** antes de leer el cuerpo.
- **Maneja errores con `try-catch`** (o `.catch`).
- **Muestra feedback al usuario:** "Cargando...", errores, éxito.
- **Deshabilita el botón** durante la petición para evitar clics repetidos.
- **Nunca uses `innerHTML` con datos que vienen del servidor** sin sanear: riesgo de XSS. Prefiere `textContent`.
- **Usa rutas relativas** (`/api/hola`) cuando el frontend y backend comparten origen.
- **Un botón, una responsabilidad:** Cada botón llama a un endpoint distinto.
- **Centraliza la lógica:** Una función `consultarAPI(ruta, elementoDestino)` que reciba la ruta y dónde mostrar el resultado. Así no repites código para cada botón.
- **No abuses de `alert`** para mostrar respuestas; mejor en el DOM.
- **Sé específico con el `Content-Type`**: si tu API devuelve texto plano, lee con `.text()`; si devuelve JSON, con `.json()`.
- **Prueba los endpoints primero en el navegador** (`http://localhost:8080/api/hola`) antes de consumirlos con `fetch`. Si eso falla, `fetch` también fallará.
- **Documenta los endpoints** en el README: ruta, método, respuesta esperada.

---

## 12. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **CORS blocked** | Sirve el frontend desde el mismo backend. O añade cabeceras CORS en el servidor. |
| **"Failed to fetch"** | El servidor está caído, o el puerto no coincide. Verifica en el navegador directamente. |
| **`response.json()` falla** | El cuerpo no es JSON válido. Usa `.text()` si es texto plano. |
| **"Body already read"** | Solo puedes leer el cuerpo una vez. Guarda el resultado en una variable. |
| **`await` fuera de función async** | Envuelve el código en una función `async` o usa `.then`. |
| **La respuesta no se muestra** | Verifica el selector del DOM y que el listener esté bien asignado. |
| **El botón no hace nada** | Abre DevTools → Console y verifica errores de JavaScript. |
| **404 en `/api/hola`** | El endpoint no existe en tu servidor C++ o la ruta está mal escrita. |
| **El servidor no responde a la segunda petición** | El backend debe seguir en bucle aceptando conexiones. Cada `fetch` es una conexión nueva. |
| **`fetch` no espera y el código sigue** | Falta `await` delante del `fetch`. |
| **Muestro `[object Promise]`** | Olvidaste `await` al leer el cuerpo (`res.text()` o `res.json()`). |
| **Los acentos se ven mal** | El servidor debe enviar `charset=utf-8` en el `Content-Type`. |
| **CORS al abrir con doble clic** | Abre por `http://localhost:8080` en lugar de `file://`. |

---

## 13. Aplicación a los botones del ejercicio

- **Botón "Hola":** `fetch("/api/hola")` → `.text()` → mostrar en `<div>`.
- **Botón "Hora":** `fetch("/api/hora")` → `.text()` → mostrar en `<div>`.
- **Botón "Versión":** `fetch("/api/version")` → `.text()` → mostrar en `<div>`.
- **Botón "Estado":** `fetch("/api/status")` → `.text()` → mostrar en `<div>`.

- **Refactorización recomendada:** Una función `consultar(endpoint)` que:
  1. Muestra "Cargando..." en el área de resultados.
  2. Hace `fetch(endpoint)`.
  3. Verifica `response.ok`.
  4. Lee el cuerpo con `.text()`.
  5. Actualiza el DOM con el resultado.
  6. Captura errores y los muestra.
  
  Así cada botón solo llama `consultar("/api/hola")`, `consultar("/api/hora")`, etc.

- **Elemento de salida:** Un solo `<div id="resultado">` donde se reemplaza el contenido. O un `<div>` por endpoint. El primero es más simple.

---

## 14. Flujo completo de una petición con `fetch`

1. Usuario pulsa el botón "Hola".
2. Se ejecuta el listener `click`.
3. Se llama `fetch("/api/hola")`.
4. El navegador abre (o reutiliza) una conexión TCP al `localhost:8080`.
5. Envía `GET /api/hola HTTP/1.1`.
6. El servidor C++ recibe la petición, la rutea, construye la respuesta `Hola desde C++`.
7. El navegador recibe la respuesta.
8. La promesa se resuelve con el objeto `Response`.
9. Se llama a `response.text()` para leer el cuerpo (otra promesa).
10. Se obtiene el string `"Hola desde C++"`.
11. Se actualiza el DOM con ese texto.
12. El usuario ve el mensaje **sin que la página se recargue**.

---

## 15. Resumen de conceptos clave

- **`fetch()`:** API nativa del navegador para hacer peticiones HTTP. Asíncrona, basada en promesas.
- **`async/await`:** Sintaxis moderna y legible para manejar promesas.
- **`Response`:** Objeto con `ok`, `status`, `headers` y métodos `.text()`, `.json()`.
- **Errores:**
  - Red → rechazo de la promesa.
  - HTTP → la promesa se resuelve, hay que verificar `response.ok`.
- **GET:** Método por defecto de `fetch`. Sin configuración adicional.
- **Actualizar el DOM:** `textContent` o `innerHTML` con la respuesta. Prefiere `textContent`.
- **Mismo origen:** Frontend y backend en el mismo host y puerto → sin CORS.
- **CORS:** Bloquea peticiones entre orígenes distintos. Evítalo sirviendo el frontend desde el mismo backend.
- **DevTools → Network:** Imprescindible para depurar peticiones `fetch`.
- **Buenas prácticas:** Feedback al usuario, deshabilitar botones, `try-catch`, verificar `response.ok`, no usar `innerHTML` con datos externos.
- **Aplicación:** Un botón por endpoint, o una función genérica reutilizable.

---
# PASO A PASO

---
# Actividad 26 — Primer `fetch()`

## Objetivo
Desde JS, consultar los endpoints de la actividad 24 con `fetch()` y mostrar el resultado en la página **sin recargarla**.

## Requisito
Tener el servidor C++ de la actividad 24 corriendo, con `/api/hola`, `/api/hora`, `/api/version`, `/api/status`.

---

## Paso 1 — Agregar el HTML de la sección

En `public/index.html`, agrega:

```html
<section id="api-demo">
  <h2>Consultar API</h2>

  <div class="botones-api">
    <button type="button" data-endpoint="/api/hola">Hola</button>
    <button type="button" data-endpoint="/api/hora">Hora</button>
    <button type="button" data-endpoint="/api/version">Versión</button>
    <button type="button" data-endpoint="/api/status">Estado</button>
  </div>

  <p id="api-resultado">Pulsa un botón...</p>
</section>
```

🎯 Usamos `data-endpoint` para no duplicar cuatro listeners distintos.

---

## Paso 2 — Un listener genérico

En `app.js`:

```js
const apiResultado = document.querySelector('#api-resultado');
const botonesApi   = document.querySelectorAll('[data-endpoint]');

async function consultarAPI(endpoint) {
  apiResultado.textContent = 'Cargando...';
  apiResultado.classList.remove('error');

  try {
    const res = await fetch(endpoint);
    if (!res.ok) {
      throw new Error(`HTTP ${res.status}`);
    }
    const texto = await res.text();
    apiResultado.textContent = texto;
  } catch (err) {
    apiResultado.textContent = `Error: ${err.message}`;
    apiResultado.classList.add('error');
    console.error(err);
  }
}

botonesApi.forEach((boton) => {
  boton.addEventListener('click', () => {
    consultarAPI(boton.dataset.endpoint);
  });
});
```

---

## Paso 3 — Estilos mínimos

En `styles.css`:

```css
#api-resultado {
  margin-top: 1rem;
  padding: 0.75rem 1rem;
  background: #f0f9ff;
  border-left: 4px solid var(--color-primario);
  border-radius: 4px;
  font-family: monospace;
}

#api-resultado.error {
  background: #fee2e2;
  border-color: var(--color-peligro);
  color: #991b1b;
}
```

---

## Paso 4 — Probar

1. Abre `http://localhost:8080/`.
2. Pulsa **Hola** → aparece `Hola desde C++`.
3. Pulsa **Hora** → aparece la hora actual.
4. Pulsa **Versión** → aparece la versión.
5. Pulsa **Estado** → `OK`.

Todo sin recargar la página.

---

## Paso 5 — Ver qué pasa en DevTools

DevTools → **Network** → filtro **Fetch/XHR**. Cada clic lanza una petición:

- URL `/api/hola`, método `GET`, status `200`.
- **Response** → el cuerpo de la respuesta.

🎯 **Regla de oro de `fetch`:** La promesa se **rechaza** solo si hay error de red. Si el servidor responde 404 o 500, la promesa se **resuelve** y hay que verificar `res.ok`.

---

## Paso 6 — Probar un endpoint que no existe

Cambia temporalmente un `data-endpoint` a `/api/pepito` y pulsa. Verás:

```
Error: HTTP 404
```

---

## Paso 7 — Deshabilitar el botón mientras carga

```js
async function consultarAPI(endpoint, boton) {
  boton.disabled = true;
  apiResultado.textContent = 'Cargando...';

  try {
    const res = await fetch(endpoint);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    apiResultado.textContent = await res.text();
  } catch (err) {
    apiResultado.textContent = `Error: ${err.message}`;
  } finally {
    boton.disabled = false;
  }
}

botonesApi.forEach((boton) => {
  boton.addEventListener('click', () => {
    consultarAPI(boton.dataset.endpoint, boton);
  });
});
```

---

## Paso 8 — Sin CORS (recordatorio)

Como el frontend y el backend están en el **mismo origen** (`http://localhost:8080`), no hay CORS. Si abres el `index.html` con `file://` o desde Live Server, las peticiones fallarán.

✅ Correcto: `http://localhost:8080/`
❌ Incorrecto: `file:///.../index.html` o `http://localhost:5500/`

---

## ✅ Checklist de la Actividad 26

- [ ] Los 4 botones funcionan y muestran la respuesta sin recargar.
- [ ] Antes de responder se ve `Cargando...`.
- [ ] Un endpoint 404 muestra mensaje de error.
- [ ] En DevTools → Network se ven las peticiones `fetch`.
- [ ] El botón se deshabilita durante la petición.

## 🧠 Mini-reto
Haz que la hora se actualice automáticamente cada 5 segundos llamando a `/api/hora` con `setInterval`.

---
