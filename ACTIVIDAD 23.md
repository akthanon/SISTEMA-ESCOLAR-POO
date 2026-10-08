# Actividad 23. Nuestro primer formulario

## Temas

- Formularios HTML
    
- GET
    
- POST (conceptualmente)
    

## Ejercicio

Crear una página:

```text
registro.html
```

con un formulario para registrar alumnos.

Campos:

- Nombre
    
- Edad
    
- Matrícula
    

Al enviar el formulario, observar con las herramientas del navegador qué información se envía al servidor.

Todavía no se procesa la información.

---
# TEORÍA

---

## 1. ¿Qué es un formulario HTML?

- **Definición:** Un formulario es un conjunto de controles (inputs, selects, checkboxes, etc.) que el usuario rellena y envía al servidor para que los procese.
- **Etiqueta contenedora:** `<form>`. Todo control debe estar dentro para ser enviado.
- **Flujo básico:**
  1. El usuario rellena los campos.
  2. Pulsa un botón de envío (`submit`).
  3. El navegador **empaqueta los datos** de los campos con `name` en un formato específico.
  4. Envía una **petición HTTP** al servidor con esos datos (en la URL o en el cuerpo).
  5. El servidor responde (todavía no procesa nada en esta actividad, solo observamos qué llega).

---

## 2. Estructura básica de un formulario

- **`<form>`:** Contenedor. Atributos clave:
  - **`action`:** URL a la que se envían los datos (por ejemplo, `/registrar` o `#`).
  - **`method`:** `GET` o `POST`. Determina **cómo** viajan los datos.
  - **`enctype`:** Cómo se codifica el cuerpo (relevante solo en POST).
  - **`target`:** Dónde mostrar la respuesta (`_self`, `_blank`, etc.).
  - **`novalidate`:** Desactiva la validación nativa del navegador.

- **Controles:** Cada campo debe tener un atributo **`name`**. Sin `name`, el campo **no se envía**.
  - `<input>`, `<textarea>`, `<select>`, `<button>`.

- **Botón de envío:** `<button type="submit">` o `<input type="submit">`. Es lo que dispara el envío.

**Fragmento suelto (esqueleto):**
```html
<form action="/registrar" method="post">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre">
    <button type="submit">Enviar</button>
</form>
```

---

## 3. Atributos importantes de los `<input>`

| Atributo | Función |
|----------|---------|
| `type` | Tipo de campo: `text`, `number`, `email`, `password`, `date`, `checkbox`, `radio`, `file`, `hidden`, etc. |
| `name` | **Obligatorio** para que el campo se envíe. Identifica el dato. |
| `id` | Identificador único. Se usa con `<label for="...">` y con JS. |
| `value` | Valor inicial o valor enviado (en botones y hidden). |
| `placeholder` | Texto de ayuda dentro del campo. |
| `required` | El navegador no permite enviar si está vacío. |
| `min`, `max`, `step` | Validaciones para números/fechas. |
| `maxlength`, `minlength` | Longitud de texto. |
| `pattern` | Expresión regular para validar el formato. |
| `disabled` | El campo **no se envía**. |
| `readonly` | El campo **sí se envía**, pero no se puede editar. |
| `checked`, `selected` | Estado inicial de checkboxes/radios/opciones. |

- **Diferencia clave `disabled` vs `readonly`:** Un campo `disabled` no viaja en la petición; uno `readonly` sí.
- **`name` duplicados:** Si dos campos tienen el mismo `name`, ambos valores viajan (útil para checkboxes agrupados o radios).

---

## 4. GET: datos en la URL

- **Definición:** El método GET envía los datos del formulario **como parte de la URL**, en la **query string**.
- **Formato:** `accion?clave1=valor1&clave2=valor2&...`
  - `clave` es el `name` del campo.
  - Los pares se separan con `&`.
  - Los caracteres especiales se codifican (URL encoding): espacio → `%20` o `+`, `ñ` → `%C3%B1`, `&` → `%26`, etc.
- **Ejemplo conceptual:**
  ```
  GET /registrar?nombre=Ana+Lopez&edad=20&matricula=2024001 HTTP/1.1
  ```

- **Ventajas:**
  - Simple, fácil de probar (puedes escribir la URL a mano).
  - Los parámetros son visibles (útil para búsquedas compartibles).
  - Se puede guardar en marcadores.

- **Desventajas:**
  - **Sin privacidad:** Cualquiera que vea la URL ve los datos (historial, logs, referrers).
  - **Longitud limitada:** Las URLs tienen un límite (típicamente 2000-8000 caracteres).
  - No apto para contraseñas, archivos ni grandes volúmenes de datos.
  - **No debe usarse para modificar estado** (crear, actualizar, borrar). GET debe ser **idempotente y seguro**.

- **Cuándo usar GET:** Búsquedas, filtros, paginación, cualquier consulta que no modifique nada.

---

## 5. POST: datos en el cuerpo

- **Definición:** El método POST envía los datos del formulario **en el cuerpo (body)** de la petición HTTP, no en la URL.
- **Formato:** Los datos se codifican de manera similar a GET, pero en el cuerpo:
  ```
  nombre=Ana+Lopez&edad=20&matricula=2024001
  ```
- **Cabeceras asociadas:**
  - `Content-Type: application/x-www-form-urlencoded` (por defecto).
  - `Content-Length: <número de bytes del cuerpo>`.
- **Alternativa moderna:** `Content-Type: multipart/form-data` (para subir archivos) o `application/json` (para APIs).
- **Ejemplo conceptual de petición:**
  ```
  POST /registrar HTTP/1.1
  Host: localhost:8080
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 48
  
  nombre=Ana+Lopez&edad=20&matricula=2024001
  ```

- **Ventajas:**
  - Los datos **no son visibles** en la URL.
  - Sin límite práctico de tamaño.
  - Ideal para datos sensibles y para modificar estado.

- **Desventajas:**
  - No se puede guardar en marcadores.
  - No se puede recargar sin reenviar (aparece el típico aviso del navegador).

- **Cuándo usar POST:** Registro, login, subir archivos, cualquier operación que **modifique** estado en el servidor.

---

## 6. Comparación GET vs. POST

| Aspecto | GET | POST |
|---------|-----|------|
| Dónde van los datos | URL (query string) | Cuerpo de la petición |
| Visible en la URL | Sí | No |
| Longitud | Limitada | Sin límite práctico |
| Idempotente | Sí (no modifica) | No (puede modificar) |
| Cacheable | Sí | No por defecto |
| Marcable / compartible | Sí | No |
| Seguridad | Baja (visible) | Media (no visible en URL, pero sin HTTPS sigue siendo inseguro) |
| Uso típico | Búsquedas, filtros | Envíos, modificaciones |
| Content-Type | No aplica | `application/x-www-form-urlencoded` |

- **Regla mnemotécnica:** GET = **leer**, POST = **enviar/modificar**.
- **Ninguno es "seguro" sin HTTPS:** POST oculta los datos de la URL, pero un atacante en la red puede leerlos igual si no hay TLS.

---

## 7. Envío del formulario: cómo funciona por dentro

Cuando el usuario pulsa el botón de envío:

1. **Validación nativa:** El navegador verifica `required`, `type`, `pattern`, etc. Si falla, no envía y muestra mensajes.
2. **Recolección de datos:** Recoge el `name` y `value` de cada control **habilitado**.
   - Los campos `disabled` se omiten.
   - Los checkboxes/radios no marcados no se envían.
3. **Codificación:** Aplica URL encoding a los valores (espacios → `+`, caracteres especiales → `%XX`).
4. **Construcción de la petición:**
   - **GET:** Añade la query string a la URL.
   - **POST:** Escribe el cuerpo con el formato codificado.
5. **Envío HTTP:** Abre (o reutiliza) la conexión TCP y envía la petición.
6. **Espera la respuesta:** El navegador muestra el contenido de la respuesta según `action` y `target`.
7. **Recarga/reenvío:** Si recargas la página después de un POST, el navegador avisa que reenviará los datos.

---

## 8. Cómo observar lo que se envía (herramientas del navegador)

- **Abrir DevTools:** F12 o Ctrl+Shift+I (Cmd+Option+I en Mac).
- **Pestaña "Network" / "Red":**
  - Marca la casilla **"Preserve log"** para no perder las peticiones al navegar.
  - Filtra por **"Fetch/XHR"** o **"Doc"** para ver las peticiones del formulario.
  - Recarga con la pestaña abierta.
- **Al enviar el formulario:**
  - Aparece una nueva petición con el método (`GET` o `POST`) y la URL.
  - **Haz clic en la petición** para ver el detalle:
    - **Headers:** Método, URL, cabeceras (Host, Content-Type, Content-Length).
    - **Payload / Request:** Los datos enviados.
      - En GET: aparecen como **Query String Parameters**.
      - En POST: aparecen como **Form Data**.
    - **Response:** Lo que devolvió el servidor.
- **Ver la URL completa en GET:** Copia la URL de la barra de direcciones; verás los parámetros.
- **Ver el cuerpo en POST:** En la pestaña "Payload" o "Request" de DevTools.

**Consejo:** Es útil tener DevTools abierto en la pestaña Network **antes** de enviar el formulario, para capturar la petición.

---

## 9. URL Encoding (percent-encoding)

- **Por qué:** Las URLs solo admiten ciertos caracteres ASCII. Los demás deben codificarse.
- **Reglas básicas:**
  - Espacio → `%20` (o `+` en `application/x-www-form-urlencoded`).
  - Letras, dígitos, `-`, `_`, `.`, `~` → sin cambios.
  - Caracteres reservados (`&`, `=`, `?`, `/`, `#`, `%`) → codificados como `%XX`.
  - Tildes, ñ y caracteres no ASCII → codificados en UTF-8 byte a byte.
    - `ñ` → `%C3%B1`
    - `á` → `%C3%A1`
- **Ejemplo:**
  - Valor original: `Ana López`
  - Codificado en query: `Ana+L%C3%B3pez` o `Ana%20L%C3%B3pez`.

- **En el servidor:** Deberás **decodificar** los valores recibidos para trabajar con el texto original. En C++, esto requiere implementar una función de decodificación (buscar `%XX` y convertirlo al byte correspondiente, y `+` → espacio).

---

## 10. Validación de formularios

- **Nativa (HTML5):** El navegador valida automáticamente con:
  - `required`, `type="email"`, `type="number"`, `min`, `max`, `pattern`, `maxlength`.
  - Muestra mensajes de error al usuario si algo falla.
  - **Ventajas:** Sin JS, sencilla, accesible.
  - **Desventajas:** Limitada, mensajes no personalizables, se puede desactivar con `novalidate`.

- **Con JavaScript:** Puedes interceptar el evento `submit`, validar y llamar a `preventDefault()` si hay errores.
  - **Ventajas:** Control total, mensajes personalizados, validaciones cruzadas.
  - **Desventajas:** Requiere JS, puede ser más código.

- **En el servidor:** **Siempre valida en el servidor**, aunque valides en el cliente. El cliente puede mentir o estar deshabilitado.
  - Verifica tipos, rangos, longitudes, formato.
  - Rechaza datos inválidos con códigos HTTP apropiados (400).

**Regla de oro:** Nunca confíes en datos que vengan del cliente. Valida siempre en el servidor.

---

## 11. Cuándo el formulario "no procesa nada" (esta actividad)

- **Objetivo:** Solo **observar** lo que envía el navegador.
- **Cómo lograr que no rompa:**
  - Si el `action` apunta a una URL que no existe en tu servidor, este responderá 404 y verás la petición en DevTools con todo lo enviado. Perfecto para inspeccionar.
  - Alternativa: usar `action="#"` o `action=""` para que el navegador envíe a la misma página. Al recargarse, verás la URL con la query string (en GET) o perderás los datos (en POST).
  - **Mejor opción para observar sin efectos:** Apunta a una URL concreta como `/registrar` y observa la petición en DevTools; el 404 es irrelevante, solo te interesa la petición.
- **Recomendación:** En esta actividad, prueba primero con GET (más fácil de visualizar en la URL) y luego con POST para comparar. En DevTools verás claramente la diferencia.

---

## 12. Buenas prácticas

- **`name` en todos los campos.** Sin `name`, no se envía.
- **`id` + `<label for>`** para accesibilidad.
- **`type` adecuado** para cada dato (`number`, `email`, `date`), para aprovechar la validación nativa.
- **`required` en campos obligatorios** para UX inmediata.
- **`autocomplete`** (opcional): `on` o `off`, y valores específicos como `name`, `email`, etc.
- **Agrupa campos relacionados** con `<fieldset>` y `<legend>`.
- **Usa POST para modificar estado**, nunca GET.
- **Nunca envíes contraseñas por GET.**
- **HTTPS siempre en producción** (aunque no lo veas en este curso, es la regla).
- **Valida en cliente Y servidor.**
- **Mensajes de error claros** si usas JS para validar.
- **Botón de envío claro** (`<button type="submit">Enviar</button>`).
- **Buen `action`:** Apunta a una URL específica, no dejes el default si vas a procesar.
- **`enctype="multipart/form-data"`** si vas a subir archivos.
- **No uses `placeholder` como sustituto del `<label>`:** el placeholder desaparece al escribir y no es accesible.

---

## 13. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Los datos no llegan al servidor** | Verifica que cada `<input>` tenga `name`. Sin `name`, no viaja. |
| **El formulario recarga la página al enviar** | Es el comportamiento por defecto. Usa `event.preventDefault()` en JS si quieres manejarlo manualmente. |
| **`method="post"` pero los datos aparecen en la URL** | Verifica que no haya `method="get"` en otro lugar o que el navegador no esté reescribiendo. Revisa DevTools. |
| **`Content-Length` no coincide** | Si construyes la petición a mano, cuenta los bytes exactos del cuerpo. |
| **Los caracteres especiales se ven raros** | Falta URL encoding al enviar, o decodificación al recibir. |
| **Los checkboxes no se envían cuando no están marcados** | Es el comportamiento esperado. Solo se envía si están marcados. |
| **Los radios con mismo `name` permiten seleccionar varios** | Deben compartir el mismo `name` para agruparse. |
| **El campo con `disabled` no aparece en la petición** | Es correcto: `disabled` no se envía. Usa `readonly` si quieres enviarlo. |
| **No veo la petición en DevTools** | Abre DevTools antes de enviar y marca "Preserve log". |
| **POST recarga y reenvía los datos** | Es normal. Para evitarlo, responde con una redirección 302 (patrón POST/Redirect/GET) — lo verás más adelante. |
| **`action` vacío envía a la misma URL** | Útil para pruebas, pero no es lo ideal en producción. |
| **No se ejecuta la validación nativa** | Verifica que no haya `novalidate` en el `<form>`. |

---

## 14. Resumen de conceptos clave

- **Formulario HTML:** Conjunto de controles `<input>`, `<select>`, `<textarea>` dentro de `<form>`.
- **`name`:** Obligatorio para que un campo se envíe.
- **GET:** Datos en la query string de la URL. Visible, limitado, para consultas.
- **POST:** Datos en el cuerpo de la petición. Oculto en URL, sin límite, para modificaciones.
- **`action`:** URL a la que se envían los datos.
- **`method`:** GET o POST.
- **URL encoding:** Codificación de caracteres especiales para que viajen seguros.
- **Validación nativa:** HTML5 valida `required`, `type`, `min`, `pattern`, etc.
- **DevTools → Network:** Herramienta para inspeccionar peticiones y ver exactamente qué se envía.
- **Payload / Request:** En GET se ve como query string; en POST, como form data.
- **Sin procesamiento en esta actividad:** Solo observar la petición HTTP.
- **Regla:** GET para leer, POST para modificar. Validar siempre en el servidor.

---
# PASO A PASO

---
# Actividad 23 — Formulario HTML: observar qué envía el navegador

## Objetivo
Crear `registro.html` con un formulario de alumno. Al enviarlo, **observar en DevTools** qué manda el navegador. Todavía no lo procesamos en el servidor.

---

## Paso 1 — Crear la página

Crea `public/registro.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Registro de alumno</title>
</head>
<body>
  <h1>Registrar alumno</h1>

  <form action="/registrar" method="GET">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre" required>

    <label for="edad">Edad:</label>
    <input type="number" id="edad" name="edad" min="0" max="120" required>

    <label for="matricula">Matrícula:</label>
    <input type="text" id="matricula" name="matricula" required>

    <button type="submit">Enviar</button>
  </form>
</body>
</html>
```

⚠️ Cada `<input>` tiene `name`. **Sin `name`, el campo no se envía.**

---

## Paso 2 — Probar GET

Abre `http://localhost:8080/registro.html`.

Rellena los campos y pulsa Enviar. El navegador irá a:

```
http://localhost:8080/registrar?nombre=Ana&edad=20&matricula=A001
```

🎯 En GET, los datos van **en la URL**, visibles.

Como no tenemos handler en `/registrar`, el servidor devolverá 404. Eso está bien: lo importante es ver la URL.

---

## Paso 3 — Observar la petición en DevTools

Abre DevTools (F12) → pestaña **Network** → marca **Preserve log**.

Envía el formulario. Verás una fila con la petición a `/registrar`:

- **Headers** → método `GET`, URL con los parámetros.
- **Payload** → sección **Query String Parameters** con los tres campos.
- **Response** → 404 (esperado).

---

## Paso 4 — Cambiar a POST

Modifica el `<form>`:

```html
<form action="/registrar" method="POST">
```

Recarga y envía otra vez. Esta vez:

- **La URL ya no tiene parámetros**: `/registrar`.
- En DevTools → **Payload** verás **Form Data** con los tres campos.
- Los datos van en el **cuerpo** de la petición, no en la URL.

---

## Paso 5 — Ver el cuerpo crudo

En DevTools → Network → haz clic en la petición POST → pestaña **Headers** → sección **Request Headers**. Busca:

```
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
```

Y en **Payload** (o **Request**):

```
nombre=Ana&edad=20&matricula=A001
```

🎯 `application/x-www-form-urlencoded` es el formato por defecto de un formulario HTML. Codifica espacios como `+` y caracteres especiales como `%XX`.

---

## Paso 6 — Comparar GET vs POST

Prueba con el nombre "Ana López" (con tilde):

- **GET**: URL → `?nombre=Ana+L%C3%B3pez&...`
- **POST**: cuerpo → `nombre=Ana+L%C3%B3pez&...`

En ambos casos se aplica URL encoding. La diferencia es **dónde** viaja el dato.

---

## Paso 7 — Añadir el enlace en index.html

En tu `public/index.html`, agrega un enlace:

```html
<a href="/registro.html">Registrar alumno</a>
```

Así accedes fácil al formulario desde la página principal.

---

## ✅ Checklist de la Actividad 23

- [ ] `registro.html` carga correctamente desde `http://localhost:8080/registro.html`.
- [ ] Con `method="GET"` la URL muestra los parámetros.
- [ ] Con `method="POST"` la URL no cambia y los datos van en el cuerpo.
- [ ] Viste en DevTools → Network la sección **Payload** con los campos.
- [ ] Todos los inputs tienen `name` y `id`.

## 🧠 Mini-reto
Agrega un `<input type="hidden" name="origen" value="web">` y verifica en DevTools que viaja junto con los demás campos aunque el usuario no lo vea.

---
