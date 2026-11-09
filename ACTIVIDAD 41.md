# Actividad 41. Gestión de archivos

## Temas

- Archivos HTTP
    
- Transferencia de datos
    
- Formularios multipart (simplificado)
    

## Ejercicio

Agregar soporte para subir y descargar archivos.

Crear una sección:

```text
archivos.html
```

Permitir:

- Subir documentos.
    
- Listar archivos disponibles.
    
- Descargar archivos.
    

Crear una carpeta:

```text
data/
└── uploads/
```

Agregar rutas:

```text
GET  /api/archivos

POST /api/subir

GET  /archivo/nombreArchivo
```

El servidor deberá:

- recibir el archivo;
    
- almacenarlo;
    
- mostrar la lista disponible;
    
- permitir descargarlo nuevamente.
    

---
# TEORÍA

---

## 1. ¿Qué es la gestión de archivos vía HTTP?

- **Definición:** Permitir que los clientes suban archivos al servidor, los listen y los descarguen.
- **Diferencia con enviar texto:** Los archivos pueden ser binarios (imágenes, PDFs, ZIPs) y de tamaño variable. No se manejan como JSON ni como texto plano.
- **En el proyecto:** Un administrador o usuario podrá subir documentos (PDFs, imágenes, etc.), ver la lista de archivos disponibles y descargarlos.
- **Casos de uso típicos:** Subir tareas, compartir documentos, avatares de usuario, reportes exportados.

---

## 2. Tipos de contenido: texto vs. binario

### A. Texto (`text/*`, `application/json`)
- Se puede leer y manipular como string.
- Codificación UTF-8 típica.
- Ejemplos: HTML, CSS, JS, JSON, XML, CSV.

### B. Binario (`image/*`, `application/pdf`, `application/zip`, `application/octet-stream`)
- **No** se puede tratar como texto sin corromperlo.
- Cada byte tiene significado propio.
- Ejemplos: imágenes, PDFs, ejecutables, archivos comprimidos.
- **Importante:** Al leer/escribir en C++, usa el modo **binario** (`ios::binary`).

### C. Regla práctica
- Si el archivo es texto → puedes leerlo con `getline` o como string.
- Si es binario → léelo **byte a byte** o en bloques, sin conversiones.

---

## 3. Formularios multipart (`multipart/form-data`)

### A. ¿Qué es?
- Formato especial de codificación para enviar **archivos y campos juntos** en una petición POST.
- **Content-Type:** `multipart/form-data; boundary=<algo>`.
- **Boundary:** Cadena única que separa las partes del formulario.
- **Cada parte** tiene sus propias cabeceras (nombre del campo, nombre del archivo, tipo MIME) y su contenido.

### B. Estructura conceptual
```
--BOUNDARY\r\n
Content-Disposition: form-data; name="nombreCampo"\r\n
\r\n
valor del campo\r\n
--BOUNDARY\r\n
Content-Disposition: form-data; name="archivo"; filename="documento.pdf"\r\n
Content-Type: application/pdf\r\n
\r\n
<bytes binarios del archivo>\r\n
--BOUNDARY--\r\n
```

- **Delimitador final:** `--BOUNDARY--` (con dos guiones al final).
- **Cada parte** empieza con `--BOUNDARY\r\n`.
- **Separación cabeceras/contenido de la parte:** `\r\n\r\n`.
- **Después del contenido:** `\r\n` antes del siguiente `--BOUNDARY`.

### C. Diferencia con urlencoded
- **urlencoded:** solo texto, campos codificados como `clave=valor&clave2=valor2`.
- **multipart:** archivos binarios + campos de texto, con boundaries separadores.
- **urlencoded** es más simple pero no permite archivos.

### D. Cuándo se usa multipart
- **Siempre** que se suben archivos desde un formulario HTML.
- Cuando hay muchos campos y archivos mezclados.
- Para subir archivos desde `fetch` con `FormData`.

### E. `FormData` en JavaScript
- Objeto del navegador que facilita construir peticiones multipart.
- Se puede añadir strings y archivos (`File`) con `.append(clave, valor)`.
- Al pasar `FormData` como `body` del `fetch`, el navegador genera automáticamente el `Content-Type: multipart/form-data` con el boundary correcto.
- **Importante:** **No** especifiques manualmente el `Content-Type` en el `fetch` cuando usas `FormData`; el navegador lo hace por ti.

**Fragmento suelto (idea):**
```js
const formData = new FormData();
formData.append("archivo", input.files[0]);
formData.append("descripcion", "documento");
fetch("/api/subir", { method: "POST", body: formData });
```

- **Observa:** No hay `headers: { "Content-Type": ... }`. El navegador lo añade.
- **Observa:** No hay `JSON.stringify`. El body es `FormData` directamente.

---

## 4. Subida de archivos: cabeceras HTTP

### A. Content-Type
- **Con multipart:** `multipart/form-data; boundary=<algo>`.
- **El boundary** es generado por el cliente. El servidor debe extraerlo.
- **Cuidado:** El boundary puede contener caracteres especiales. No lo asumas fijo.

### B. Content-Length
- **Obligatorio.** Indica el tamaño total del cuerpo (incluyendo boundaries y cabeceras de parte).
- Con archivos grandes, es fundamental para saber cuándo parar de leer.

### C. Nombre del archivo
- Viene en la cabecera `Content-Disposition` de la parte correspondiente:
  ```
  Content-Disposition: form-data; name="archivo"; filename="documento.pdf"
  ```
- **Cuidado:** El nombre puede venir con comillas, caracteres especiales o caracteres no ASCII. Hay que sanitizarlo.

### D. Tipo MIME del archivo
- Viene en `Content-Type` de la parte (si el cliente lo envía):
  ```
  Content-Type: application/pdf
  ```
- **No confíes ciegamente** en él. Verifica la extensión y, si puedes, el contenido (magic bytes).

---

## 5. Parseo del cuerpo multipart en C++

### A. Estrategia general
1. Extraer el `boundary` de la cabecera `Content-Type`.
2. Dividir el cuerpo por el boundary.
3. Cada parte tiene:
   - Sus cabeceras (Content-Disposition, Content-Type).
   - Una línea vacía.
   - El contenido (binario o texto).
4. Extraer nombre del campo, nombre del archivo y el contenido de cada parte.

### B. Extraer el boundary
- Busca `boundary=` en la cabecera `Content-Type`.
- El valor va hasta el final de la línea (`\r\n`) o hasta el primer `;`.
- Puede estar entre comillas o no. Considera ambos casos.

**Fragmento suelto (idea):**
```cpp
size_t p = contentType.find("boundary=");
string boundary = contentType.substr(p + 9);
if (boundary.front() == '"') boundary = boundary.substr(1, boundary.size() - 2);
```

### C. Dividir por el boundary
- El boundary real en el cuerpo viene precedido por `--`:
  ```
  --BOUNDARY
  ```
- Busca todas las ocurrencias de `--BOUNDARY` en el cuerpo.
- Cada par consecutivo delimita una parte.
- **Cuidado:** El contenido del archivo puede contener el boundary por casualidad (raro pero posible). Por eso se usa un boundary **aleatorio** y **largo**.

### D. Parseo de cada parte
- Después de `--BOUNDARY\r\n`, vienen las cabeceras de la parte.
- Busca `\r\n\r\n` para separar cabeceras de contenido.
- El contenido termina justo antes del siguiente `\r\n--BOUNDARY`.
- **Importante:** El contenido puede ser binario. Debes preservar **todos los bytes** desde después del `\r\n\r\n` hasta antes del siguiente `\r\n--BOUNDARY`.

### E. Casos especiales
- **Última parte:** Termina con `--BOUNDARY--\r\n`.
- **Nombres con comillas:** `filename="documento.pdf"` → extraer el valor entre comillas.
- **Sin filename:** Es un campo de texto normal (`name="descripcion"`).
- **Contenido vacío:** Puede haber partes sin contenido (raro pero posible).

### F. Alternativa: bibliotecas multipart
- Parsear multipart manualmente es tedioso y propenso a errores.
- Existen librerías en C++ que lo manejan (por ejemplo, `cpp-httplib`, `multipart-parser-cpp`).
- **Para esta actividad:** Si el parseo manual se complica, considera una librería. Pero entender el formato es el objetivo didáctico.

---

## 6. Almacenamiento de archivos

### A. Carpeta de destino
- **Creada previamente:** `data/uploads/`.
- **No se sirve como estático.** No está en `public/`.
- **Razón:** Los archivos subidos por usuarios no deben exponerse directamente; deben pasar por validación y control de acceso.

### B. Nombre del archivo en disco
- **Nunca uses el nombre original tal cual.** Riesgo de colisiones, caracteres peligrosos, path traversal.
- **Estrategia 1: Timestamp + nombre original sanitizado.**
  - `1682345678_documento.pdf`.
- **Estrategia 2: UUID o hash.**
  - `a1b2c3d4e5f6.pdf`.
  - **Ventaja:** Sin colisiones, sin nombres peligrosos.
- **Estrategia 3: Contador incremental.**
  - Simple pero requiere gestionar el contador.

- **Recomendación:** Timestamp + nombre sanitizado. Preserva algo de legibilidad.

### C. Sanitizar el nombre
- **Eliminar** rutas (`../`, `/`, `\`).
- **Reemplazar** caracteres problemáticos (espacios → `_`, tildes → sin tildes o codificadas).
- **Limitar** la longitud (por ejemplo, 100 caracteres).
- **Conservar** la extensión (necesaria para el MIME).

**Fragmento suelto (idea):**
```cpp
string sanitizar(const string& nombre) {
    // eliminar rutas, reemplazar caracteres, limitar longitud
}
```

### D. Guardar en modo binario
- **Imprescindible** para no corromper archivos binarios.
- `ofstream archivo(ruta, ios::binary);`
- Escribe el contenido tal cual, sin `<<` con formato.

**Fragmento suelto:**
```cpp
ofstream out(rutaFinal, ios::binary);
out.write(contenido.data(), contenido.size());
out.close();
```

- **`.data()`** devuelve el puntero a los bytes del string. **`.size()`** la cantidad.
- **Nunca uses `<<`** para binario: interpreta el contenido como texto.

### E. Nombre original vs. nombre en disco
- **Guarda ambos:** el archivo en disco con nombre sanitizado, y una referencia en un índice (archivo de texto o BD) con el nombre original.
- **Motivo:** Al descargar, el usuario espera su nombre original.

**Índice conceptual:**
```
id|nombreOriginal|nombreDisco|mime|tamano|fecha
```

- **Ubicación:** `data/uploads/index.txt` o una tabla en la BD.
- **Ventaja:** Puedes listar, buscar y descargar archivos por su nombre original.

---

## 7. Listar archivos: `GET /api/archivos`

### A. Fuente de datos
- **Opción 1:** Leer la carpeta `data/uploads/` con `std::filesystem::directory_iterator`.
- **Opción 2:** Leer el índice (`index.txt`) que registra los archivos subidos.
- **Recomendación:** Opción 2. El índice tiene el nombre original, el MIME y metadatos. La carpeta solo tiene nombres sanitizados.

### B. Respuesta
- **Content-Type:** `application/json`.
- **Cuerpo:** Arreglo JSON con la lista de archivos.
  ```json
  [
    {"id":"1","nombre":"tarea1.pdf","tamano":12345,"fecha":"2025-01-15"},
    {"id":"2","nombre":"foto.png","tamano":54321,"fecha":"2025-01-16"}
  ]
  ```

- **Campos a incluir:**
  - `id`: identificador para descargar.
  - `nombre`: nombre original.
  - `tamano`: en bytes.
  - `fecha`: timestamp.
  - `mime`: tipo MIME.

- **Rol que puede listar:** según tu diseño. Probablemente admin o cualquier usuario autenticado.

---

## 8. Descarga de archivos: `GET /archivo/{nombre}`

### A. Ruta
- **Ejemplo:** `GET /archivo/documento.pdf`.
- **El nombre** es el original (o el id). El servidor lo busca en el índice y localiza el archivo en disco.
- **Nota:** Se prefija con `/archivo/` para no confundir con `/api/`.

### B. Búsqueda del archivo
- Recibir el nombre o id de la URL.
- Buscar en el índice:
  - Si es por nombre → buscar coincidencia exacta.
  - Si es por id → buscar por id.
- Si no existe → `404 Not Found`.
- Si existe → leer el archivo de disco.

### C. Cabeceras de respuesta
- **`Content-Type`:** el MIME del archivo (del índice).
- **`Content-Length`:** tamaño en bytes.
- **`Content-Disposition`:** fundamental para forzar descarga.
  - `attachment; filename="documento.pdf"` → el navegador descarga.
  - `inline; filename="documento.pdf"` → el navegador intenta mostrarlo (útil para PDFs o imágenes).
- **`Connection: close`** o `keep-alive`.

**Fragmento suelto (cabecera):**
```
Content-Disposition: attachment; filename="documento.pdf"
```

- **`attachment`** → descarga directa.
- **`inline`** → mostrar en el navegador si puede.
- **El nombre** debe ir entre comillas y puede necesitar codificación si tiene caracteres no ASCII (RFC 5987).

### D. Envío del contenido
- **Leer en modo binario.**
- **Construir la respuesta** concatenando cabeceras + `\r\n\r\n` + bytes del archivo.
- **Enviar con `send`.** Puede que necesites múltiples llamadas para archivos grandes.
- **No uses operador `<<`** con strings si contienen bytes nulos (`\0`). Usa `send` con el buffer crudo.

**Fragmento suelto (envío de binario):**
```cpp
send(cliente, cabeceras.c_str(), cabeceras.size(), 0);
send(cliente, contenido.data(), contenido.size(), 0);
```

- **Dos llamadas:** una para cabeceras, otra para el cuerpo. Más claro y eficiente que concatenar todo.
- **Alternativa:** concatenar en un solo buffer y enviar de una vez. Solo si el archivo es pequeño.

---

## 9. Subida: `POST /api/subir`

### A. Flujo del handler
1. Verificar método POST y Content-Type `multipart/form-data`.
2. Extraer el boundary.
3. Leer el cuerpo completo (respetando `Content-Length`).
4. Parsear las partes.
5. Identificar la parte con el archivo (`filename` presente).
6. Sanitizar el nombre.
7. Generar nombre en disco único.
8. Guardar el archivo en `data/uploads/`.
9. Registrar la entrada en el índice.
10. Responder con éxito y los datos del archivo subido.

### B. Tamaño máximo
- **Definir un límite** (por ejemplo, 10 MB).
- Verificar el `Content-Length` antes de leer. Si excede → `413 Payload Too Large`.
- **Motivo:** Evitar que un cliente malicioso llene el disco o la memoria.
- **En producción:** Límite estricto y configuración en el servidor.

### C. Validaciones
- **Content-Type:** debe ser `multipart/form-data`.
- **Tamaño:** no exceder el límite.
- **Extensión:** permitir solo ciertas extensiones (`.pdf`, `.png`, `.jpg`, `.txt`, etc.).
- **Tipo MIME:** verificar contra la extensión.
- **Nombre:** sanitizar.
- **Contenido:** verificar magic bytes (avanzado, opcional).
- **Autenticación:** solo usuarios autenticados pueden subir.
- **Autorización:** según tu diseño, ciertos roles.

### D. Respuesta
- **`201 Created`** con el objeto del archivo subido.
- **Cuerpo:** JSON con `id`, `nombre`, `tamano`, `fecha`.
- **En caso de error:**
  - `400` → datos mal formados.
  - `401` → sin sesión.
  - `403` → sin permiso.
  - `413` → archivo demasiado grande.
  - `415` → tipo no soportado.
  - `500` → error de escritura.

### E. Logging
- Loguea: usuario, nombre original, nombre en disco, tamaño, resultado.
- **No loguees el contenido** del archivo.

---

## 10. Seguridad en la gestión de archivos

### A. Path traversal
- **Ataque:** Enviar `filename="../../../etc/passwd"` para sobrescribir archivos del sistema.
- **Defensa:** **Sanitizar siempre.** Eliminar rutas, `..`, `/`, `\`, caracteres de control.
- **Nunca** uses el nombre original directamente como nombre en disco.

### B. Sobrescritura de archivos
- **Ataque:** Enviar un archivo con nombre de otro existente para sobrescribirlo.
- **Defensa:** Generar nombres únicos (timestamp + sanitizado o UUID). Nunca sobrescribir.

### C. Ejecución de archivos subidos
- **Ataque:** Subir un script `.php`, `.exe`, `.sh` y ejecutarlo desde el navegador.
- **Defensa:**
  - **Nunca** guardes en `public/`. Guarda en `data/uploads/`.
  - **Nunca** sirvas archivos con MIME "ejecutable".
  - Fuerza `Content-Disposition: attachment` para forzar descarga.
  - Valida extensiones permitidas.

### D. Contenido malicioso
- **Ataque:** Subir un archivo con MIME tipo imagen pero contenido malicioso.
- **Defensa:**
  - Verificar magic bytes (firma del archivo).
  - Usar bibliotecas de validación.
  - Si el archivo es una imagen, reprocesarla (redimensionar, recomprimir) antes de servirla.

### E. Agotamiento de disco
- **Ataque:** Subir muchos archivos grandes para llenar el disco.
- **Defensa:**
  - Límite de tamaño por archivo.
  - Límite de archivos por usuario.
  - Cuotas de disco.
  - Limpieza periódica.

### F. XSS al listar archivos
- **Ataque:** Subir un archivo llamado `<script>alert(1)</script>.pdf` y mostrarlo sin escapar en el HTML.
- **Defensa:** Escapa el nombre al mostrarlo en la tabla del frontend. Usa `textContent`, no `innerHTML`.

### G. Fugas de información
- **Ataque:** Acceder a archivos de otros usuarios adivinando nombres.
- **Defensa:**
  - IDs largos y aleatorios.
  - Verificar permisos de acceso por archivo.
  - No exponer listados públicos sin autenticación.

### H. Symlinks
- **Ataque:** Subir un archivo que sea un enlace simbólico.
- **Defensa:** Verifica que sea un archivo regular (`is_regular_file`). No sigas enlaces.

---

## 11. Frontend: `archivos.html` + `archivos.js`

### A. Estructura de la página
- **Header:** título, nombre del usuario, botón "Cerrar sesión".
- **Sección de subida:**
  - `<input type="file" id="archivo">`.
  - Opcional: campo de descripción.
  - Botón "Subir".
  - Barra de progreso (opcional, avanzada).
- **Sección de listado:**
  - Tabla con columnas: nombre, tamaño, fecha, acciones.
  - Acciones: botón "Descargar", botón "Eliminar" (opcional).
- **Área de mensajes:** éxito, error, en progreso.

### B. Lógica de subida
1. Usuario selecciona archivo y pulsa "Subir".
2. Validar en cliente: extensión permitida, tamaño máximo.
3. Construir `FormData` y añadir el archivo.
4. `fetch("/api/subir", { method: "POST", body: formData })`.
5. **No especificar `Content-Type`.** El navegador lo hace.
6. Verificar `res.ok`.
7. En éxito: refrescar listado, limpiar input, mostrar mensaje.
8. En error: mostrar mensaje del servidor.

**Fragmento suelto (patrón):**
```js
const formData = new FormData();
formData.append("archivo", input.files[0]);

const res = await fetch("/api/subir", {
    method: "POST",
    body: formData
});
```

### C. Lógica de listado
1. `fetch("/api/archivos")`.
2. Parsear JSON.
3. Renderizar la tabla dinámicamente.
4. Cada fila tiene un enlace de descarga:
   - `<a href="/archivo/{id}" download>` o
   - `<a href="/archivo/{id}" target="_blank">`.
5. El atributo `download` fuerza la descarga. Pero el servidor también lo puede forzar con `Content-Disposition`.

### D. Lógica de descarga
- **Forma 1:** Enlace directo (`<a href="/archivo/{id}">`).
  - El navegador hace la petición y sigue las cabeceras.
  - Si el servidor envía `Content-Disposition: attachment`, se descarga.
  - Si envía `inline`, se muestra en el navegador (útil para PDFs e imágenes).
- **Forma 2:** `fetch` con blob.
  - Más complejo. Útil si necesitas manejar el progreso o autenticación especial.
  - Crea un `Blob` desde la respuesta y un enlace temporal para descargarlo.

- **Recomendación para esta actividad:** Forma 1. Es lo más simple y el navegador lo hace todo.

### E. Progreso de subida (avanzado)
- **`XMLHttpRequest`** permite escuchar el evento `progress`.
- **`fetch`** no permite (todavía) monitorear el progreso de subida de forma nativa.
- **Para esta actividad:** No es necesario. Opcional.

---

## 12. Estructura de la carpeta `data/`

```
data/
├── usuarios.txt
├── uploads/
│   ├── 1682345678_tarea1.pdf
│   ├── 1682345700_foto.png
│   └── index.txt          ← registro de archivos subidos
```

- **`uploads/`:** almacena los archivos físicos con nombres únicos.
- **`index.txt`:** registro con la metadata.
- **Formato del índice:** un archivo por línea.
  ```
  id|nombreOriginal|nombreDisco|mime|tamano|fecha
  ```
- **Ventaja:** El servidor lee el índice para listar; el disco solo tiene los bytes.

### Alternativa con SQL
- **Tabla `archivos`:** `id`, `nombre`, `nombre_disco`, `mime`, `tamano`, `fecha`, `usuario_id`.
- **Ventaja:** Consultas rápidas, integridad referencial, fácil de filtrar por usuario.
- **Para esta actividad:** TXT está bien; SQL es una mejora.

---

## 13. MIME types para descarga

- Reutiliza la función `obtenerMIME(extension)` que ya tienes.
- **Casos especiales:**
  - `application/octet-stream` → tipo genérico si no reconoces la extensión. Fuerza descarga.
  - `application/pdf` → se puede mostrar en el navegador.
  - `image/png`, `image/jpeg` → se muestran inline.
  - `text/plain` → se muestra o descarga según el navegador.
- **Regla:** Para archivos subidos por usuarios, prefiere `application/octet-stream` **o** fuerza `Content-Disposition: attachment`.

---

## 14. Tamaño máximo y streaming

### A. Para esta actividad
- Leer el archivo completo en memoria es aceptable para archivos pequeños (< 10 MB).
- **Límite explícito:** rechaza archivos más grandes.

### B. Para archivos grandes (futuro)
- **Streaming:** Leer el archivo en trozos y enviarlos al cliente sin cargar todo en memoria.
- **Chunked transfer encoding:** alternativa a `Content-Length`.
- **Range requests:** permitir descargar partes específicas (para reanudar descargas).
- **Complejo.** No es necesario para esta actividad, pero tenlo en cuenta.

---

## 15. Integración con sesiones y roles

- **Subir archivos:** requiere sesión válida. Opcionalmente, un rol específico (admin, maestro).
- **Listar archivos:** según diseño. Puede ser:
  - Todos ven todos los archivos.
  - Cada usuario ve los suyos.
  - Solo admin ve todos.
- **Descargar:** verificar permisos si los archivos son privados.
- **Eliminar:** solo el propietario o un admin.

- **Registro del propietario:** el índice puede incluir la columna `usuario`. Así filtras por usuario al listar.

---

## 16. Buenas prácticas

- **Modo binario siempre** al leer/escribir archivos binarios. `ios::binary`.
- **Sanitizar nombres** de archivo antes de usar en disco.
- **Nombres únicos en disco** (timestamp + sanitizado o UUID).
- **Nunca** uses el nombre original directamente para guardar.
- **Nunca** guardes archivos subidos en `public/`.
- **Siempre** valida tamaño, extensión y MIME.
- **Siempre** verifica autenticación y autorización.
- **Content-Disposition: attachment** para forzar descarga cuando quieras.
- **Content-Type correcto** según extensión.
- **Content-Length correcto** (bytes, no caracteres).
- **Streaming** para archivos grandes (futuro).
- **Límite de tamaño** explícito.
- **Índice separado** de los archivos físicos.
- **Escapa nombres** al mostrarlos en HTML (XSS).
- **No expongas rutas del sistema** en los mensajes de error.
- **Loguea** subidas y descargas (usuario, archivo, resultado).
- **No loguees** el contenido de los archivos.
- **Verifica que sea archivo regular**, no directorio ni symlink.
- **Cierra archivos** tras leerlos/escribirlos.
- **Prueba con archivos de distintos tipos:** texto, imagen, PDF, binario.
- **Prueba con archivos con nombres raros:** espacios, tildes, comillas, caracteres especiales.
- **Prueba con archivos grandes** justo por encima y por debajo del límite.
- **Considera cuotas por usuario** en producción.

---

## 17. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **El archivo se corrompe al subir** | No usaste modo binario al leer/escribir. Añade `ios::binary`. |
| **El archivo se descarga sin extensión** | No incluiste el nombre original en `Content-Disposition`. |
| **El navegador muestra el PDF en lugar de descargarlo** | `Content-Disposition: inline`. Cambia a `attachment`. |
| **Los bytes nulos cortan el envío** | Usa `send` con `.data()` y `.size()`, no `<<` con strings. |
| **Path traversal (`../../etc/passwd`)** | Sanitiza el nombre. Rechaza `..`, `/`, `\`. |
| **El archivo se guarda con nombre raro** | Estás usando el nombre original. Genera uno único. |
| **Sobrescritura de archivos existentes** | Genera nombres únicos. No confíes en el nombre del cliente. |
| **Colisión de archivos en el mismo segundo** | Añade un contador o componente aleatorio al nombre único. |
| **MIME incorrecto al descargar** | Usa la tabla de extensiones. No confíes solo en el Content-Type del cliente. |
| **Archivos enormes tumban el servidor** | Límite de tamaño y rechazo con `413`. |
| **El parseo multipart falla con cierto boundary** | El boundary debe extraerse correctamente. Considera librería. |
| **El nombre del archivo viene con comillas** | Extrae el valor entre comillas del `Content-Disposition`. |
| **El nombre del archivo tiene caracteres no ASCII** | Codifica el nombre en `Content-Disposition` con RFC 5987. |
| **El archivo aparece como texto plano en el navegador** | Content-Type incorrecto. Ajusta según extensión. |
| **Los acentos del nombre se ven mal** | Problema de encoding. Usa UTF-8 y codificación correcta en cabeceras. |
| **No se puede subir archivos con tildes en el nombre** | Sanitiza el nombre al guardar, pero preserva el original en el índice. |
| **`fetch` con `FormData` no añade Content-Type** | Correcto. El navegador lo añade automáticamente con el boundary. No lo especifiques. |
| **`fetch` con `FormData` y `Content-Type` manual falla** | No especifiques `Content-Type`. Deja que el navegador lo haga. |
| **El archivo subido se puede ejecutar desde el navegador** | Guárdalo fuera de `public/`. Fuerza `attachment`. |
| **`recv` no lee todo el archivo** | Sigue leyendo con `recv` hasta completar `Content-Length`. |
| **El servidor se queda sin memoria con archivos grandes** | Límite de tamaño. Streaming en el futuro. |
| **No aparece la lista de archivos** | Verifica el endpoint `/api/archivos` y el índice. |
| **El archivo descargado es más pequeño que el original** | Perdiste bytes al leer o escribir. Modo binario y `send` con `.size()`. |

---

## 18. Verificación

1. **Abre `archivos.html`** con sesión de usuario autorizado.
2. **Selecciona un archivo pequeño** (por ejemplo, un TXT).
3. **Súbelo.** Verifica en consola del servidor que se recibió.
4. **Verifica en disco** que el archivo está en `data/uploads/` con un nombre único.
5. **Verifica el índice** `data/uploads/index.txt`: debe tener una línea nueva.
6. **El listado debe mostrar el archivo.**
7. **Descarga el archivo.** Verifica que el contenido es idéntico al original (usa `diff` o `md5sum`).
8. **Repite con una imagen PNG.** Verifica que se muestra o descarga correctamente.
9. **Repite con un PDF.** Verifica que el navegador lo abre o descarga.
10. **Intenta subir un archivo con nombre `../hack.txt`.** Debe guardarse sanitizado.
11. **Intenta subir un archivo con nombre `<script>alert(1)</script>.txt`.** No debe ejecutar nada; debe mostrarse escapado en la tabla.
12. **Intenta subir un archivo de más de X MB.** Debe rechazarse con `413`.
13. **Prueba subir sin sesión.** Debe rechazarse con `401`.
14. **Prueba descargar un archivo inexistente.** Debe responder `404`.
15. **Reinicia el servidor.** El listado debe seguir mostrando los archivos (el índice persiste).

---

## 19. Resumen de conceptos clave

- **Gestión de archivos:** subir, listar y descargar archivos vía HTTP.
- **Multipart/form-data:** formato para enviar archivos + campos juntos. Con boundary único.
- **FormData en JS:** construye el cuerpo multipart automáticamente. No especifiques `Content-Type`.
- **Boundary:** cadena que separa partes. Se extrae del `Content-Type`.
- **Parseo multipart:** dividir por boundary, extraer cabeceras de parte y contenido. Preservar bytes binarios.
- **Modo binario:** `ios::binary` para leer/escribir archivos. `send` con `.data()` y `.size()`.
- **Sanitización:** eliminar rutas y caracteres peligrosos del nombre del archivo.
- **Nombre único:** timestamp + sanitizado, o UUID. Nunca el original.
- **Almacenamiento:** `data/uploads/`, no en `public/`.
- **Índice:** registro de archivos (nombre original, nombre en disco, MIME, tamaño, fecha, usuario).
- **Listar:** leer el índice, devolver JSON.
- **Descargar:** buscar en el índice, leer el archivo, enviar con cabeceras correctas.
- **Content-Disposition:** `attachment` para forzar descarga; `inline` para mostrar en el navegador.
- **Content-Type:** MIME correcto según extensión.
- **Content-Length:** tamaño en bytes del archivo.
- **Seguridad:** path traversal, sobrescritura, ejecución, agotamiento de disco, XSS, fugas.
- **Validaciones:** tamaño, extensión, MIME, sesión, permisos.
- **Errores HTTP:** 201 éxito, 400 mal formado, 401 sin sesión, 403 sin permiso, 404 no encontrado, 413 demasiado grande, 415 tipo no soportado, 500 error interno.

---
# PASO A PASO

---
# Actividad 41 — Gestión de archivos

## Objetivo
Soportar subida, listado y descarga de archivos con `multipart/form-data`. Guardar en `data/uploads/` con nombres únicos. Servir con las cabeceras correctas.

## Estructura

```
data/
└── uploads/
    ├── index.txt             ← metadatos de cada archivo
    └── 1682345678_tarea.pdf
```

Crea `data/uploads/` a mano la primera vez.

---

## Paso 1 — Estructura del índice

`data/uploads/index.txt`, un archivo por línea:

```
id|nombreOriginal|nombreDisco|mime|tamano|fecha|usuario
```

Ejemplo:

```
1|Tarea 1.pdf|1682345678_Tarea_1.pdf|application/pdf|12345|2025-01-15|ana_admin
2|foto.png|1682345700_foto.png|image/png|54321|2025-01-16|luis_alumno
```

---

## Paso 2 — Clase `RepositorioArchivos`

Añade al proyecto:

```cpp
struct InfoArchivo {
    int id;
    string nombreOriginal;
    string nombreDisco;
    string mime;
    long tamano;
    string fecha;
    string usuario;
};

class RepositorioArchivos {
private:
    string carpeta;        // "data/uploads/"
    string rutaIndice;     // "data/uploads/index.txt"
    int siguienteId = 1;

public:
    RepositorioArchivos(string carpeta, string indice)
        : carpeta(carpeta), rutaIndice(indice) {}

    bool guardarArchivo(const string& nombreOriginal,
                        const string& mime,
                        const string& contenido,
                        const string& usuario,
                        InfoArchivo& out);

    vector<InfoArchivo> listar() const;
    bool buscar(int id, InfoArchivo& out) const;
    string leerContenido(int id) const;
};
```

---

## Paso 3 — Cargar índice y calcular siguiente ID

```cpp
#include <filesystem>
namespace fs = std::filesystem;

void RepositorioArchivos::cargarIndice() {
    ifstream in(rutaIndice);
    if (!in.is_open()) return;

    string linea;
    int maxId = 0;
    while (getline(in, linea)) {
        if (linea.empty()) continue;
        int id = 0;
        try { id = stoi(linea.substr(0, linea.find('|'))); } catch (...) {}
        if (id > maxId) maxId = id;
    }
    siguienteId = maxId + 1;
}
```

Llamar en el constructor.

---

## Paso 4 — Guardar archivo

```cpp
string sanitizarNombre(const string& nombre) {
    string r;
    for (char c : nombre) {
        if (c == '/' || c == '\\' || c == ':' || c == '*' ||
            c == '?' || c == '"' || c == '<' || c == '>' || c == '|') {
            r += '_';
        } else if (c == ' ') {
            r += '_';
        } else {
            r += c;
        }
    }
    if (r.empty()) r = "archivo";
    if (r.size() > 100) r = r.substr(0, 100);
    return r;
}

bool RepositorioArchivos::guardarArchivo(const string& nombreOriginal,
                                          const string& mime,
                                          const string& contenido,
                                          const string& usuario,
                                          InfoArchivo& out) {
    string nombreSanit = sanitizarNombre(nombreOriginal);
    string nombreDisco = to_string(time(nullptr)) + "_" + nombreSanit;
    string rutaFinal   = carpeta + nombreDisco;

    // Escribir binario
    ofstream out(rutaFinal, ios::binary);
    if (!out.is_open()) return false;
    out.write(contenido.data(), contenido.size());
    out.close();

    // Registrar en índice
    ofstream idx(rutaIndice, ios::app);
    if (!idx.is_open()) return false;

    time_t ahora = time(nullptr);
    char fecha[32];
    strftime(fecha, sizeof(fecha), "%Y-%m-%d", localtime(&ahora));

    out.id             = siguienteId++;
    out.nombreOriginal = nombreOriginal;
    out.nombreDisco    = nombreDisco;
    out.mime           = mime;
    out.tamano         = contenido.size();
    out.fecha          = fecha;
    out.usuario        = usuario;

    idx << out.id << "|" << out.nombreOriginal << "|" << out.nombreDisco
        << "|" << out.mime << "|" << out.tamano << "|" << out.fecha
        << "|" << out.usuario << "\n";
    idx.close();

    return true;
}
```

---

## Paso 5 — Listar y buscar

```cpp
vector<InfoArchivo> RepositorioArchivos::listar() const {
    vector<InfoArchivo> lista;
    ifstream in(rutaIndice);
    if (!in.is_open()) return lista;

    string linea;
    while (getline(in, linea)) {
        if (linea.empty()) continue;
        stringstream ss(linea);
        InfoArchivo a;
        string id, tam;
        getline(ss, id, '|');
        getline(ss, a.nombreOriginal, '|');
        getline(ss, a.nombreDisco, '|');
        getline(ss, a.mime, '|');
        getline(ss, tam, '|');
        getline(ss, a.fecha, '|');
        getline(ss, a.usuario, '|');

        try { a.id = stoi(id); a.tamano = stol(tam); } catch (...) { continue; }
        lista.push_back(a);
    }
    return lista;
}

bool RepositorioArchivos::buscar(int id, InfoArchivo& out) const {
    for (const auto& a : listar()) {
        if (a.id == id) { out = a; return true; }
    }
    return false;
}

string RepositorioArchivos::leerContenido(int id) const {
    InfoArchivo a;
    if (!buscar(id, a)) return "";

    ifstream in(carpeta + a.nombreDisco, ios::binary);
    if (!in.is_open()) return "";
    return string((istreambuf_iterator<char>(in)),
                  istreambuf_iterator<char>());
}
```

---

## Paso 6 — Parser multipart (simplificado)

```cpp
string extraerBoundary(const string& peticion) {
    size_t pos = peticion.find("boundary=");
    if (pos == string::npos) return "";

    size_t ini = pos + 9;
    size_t fin = peticion.find("\r\n", ini);
    if (fin == string::npos) fin = peticion.size();

    string boundary = peticion.substr(ini, fin - ini);
    if (boundary.front() == '"') {
        boundary = boundary.substr(1, boundary.size() - 2);
    }
    return boundary;
}

// Devuelve (nombreArchivo, mime, contenido)
bool parsearMultipart(const string& cuerpo, const string& boundary,
                      string& nombreArchivo, string& mime, string& contenido) {
    string delim = "--" + boundary;

    size_t pos = 0;
    while ((pos = cuerpo.find(delim, pos)) != string::npos) {
        pos += delim.size();
        if (pos + 2 <= cuerpo.size() && cuerpo.substr(pos, 2) == "--") break;
        if (cuerpo.substr(pos, 2) == "\r\n") pos += 2;

        size_t finCab = cuerpo.find("\r\n\r\n", pos);
        if (finCab == string::npos) return false;

        string cabecera = cuerpo.substr(pos, finCab - pos);
        size_t iniCont  = finCab + 4;

        size_t finCont = cuerpo.find("\r\n" + delim, iniCont);
        if (finCont == string::npos) return false;

        if (cabecera.find("filename=") != string::npos) {
            // Extraer filename
            size_t p = cabecera.find("filename=");
            size_t ini = p + 9;
            if (cabecera[ini] == '"') ini++;
            size_t fin = cabecera.find('"', ini);
            nombreArchivo = cabecera.substr(ini, fin - ini);

            // Extraer Content-Type
            size_t pc = cabecera.find("Content-Type:");
            if (pc != string::npos) {
                size_t finMime = cabecera.find("\r\n", pc);
                mime = cabecera.substr(pc + 14, finMime - pc - 14);
                size_t i = mime.find_first_not_of(" \t");
                if (i != string::npos) mime = mime.substr(i);
            } else {
                mime = "application/octet-stream";
            }

            contenido = cuerpo.substr(iniCont, finCont - iniCont);
            return true;
        }

        pos = finCont + 2;
    }
    return false;
}
```

---

## Paso 7 — Endpoints

### A. `POST /api/subir`

```cpp
if (ruta == "/api/subir" && metodo == "POST") {
    string boundary = extraerBoundary(peticion);
    if (boundary.empty()) {
        return construirRespuesta(400, "application/json; charset=utf-8",
            "{\"error\":\"Se esperaba multipart/form-data\"}");
    }

    string nombre, mime, contenido;
    if (!parsearMultipart(cuerpo, boundary, nombre, mime, contenido)) {
        return construirRespuesta(400, "application/json; charset=utf-8",
            "{\"error\":\"No se encontró archivo\"}");
    }

    // Límite de 10 MB
    if (contenido.size() > 10 * 1024 * 1024) {
        return construirRespuesta(413, "application/json; charset=utf-8",
            "{\"error\":\"Archivo demasiado grande (máx 10 MB)\"}");
    }

    InfoArchivo info;
    if (!repoArchivos.guardarArchivo(nombre, mime, contenido,
                                     sesion.usuario, info)) {
        return construirRespuesta(500, "application/json; charset=utf-8",
            "{\"error\":\"Error al guardar\"}");
    }

    string json = "{\"id\":" + to_string(info.id) +
                  ",\"nombre\":\"" + escaparJSON(info.nombreOriginal) + "\"," +
                  "\"tamano\":" + to_string(info.tamano) + "}";
    return construirRespuesta(201, "application/json; charset=utf-8", json);
}
```

### B. `GET /api/archivos`

```cpp
if (ruta == "/api/archivos" && metodo == "GET") {
    auto lista = repoArchivos.listar();
    string json = "[";
    for (size_t i = 0; i < lista.size(); i++) {
        if (i > 0) json += ",";
        json += "{\"id\":" + to_string(lista[i].id) +
                ",\"nombre\":\"" + escaparJSON(lista[i].nombreOriginal) + "\"," +
                "\"tamano\":" + to_string(lista[i].tamano) + "," +
                "\"fecha\":\"" + escaparJSON(lista[i].fecha) + "\"}";
    }
    json += "]";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

### C. `GET /archivo/{id}` (fuera de `/api/`)

```cpp
if (ruta.rfind("/archivo/", 0) == 0 && metodo == "GET") {
    int id = 0;
    try { id = stoi(ruta.substr(9)); } catch (...) {
        return construirRespuesta(400, "text/plain; charset=utf-8", "ID inválido");
    }

    InfoArchivo info;
    if (!repoArchivos.buscar(id, info)) {
        return construirRespuesta(404, "text/plain; charset=utf-8", "No encontrado");
    }

    string contenido = repoArchivos.leerContenido(id);

    string cabeceras =
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: " + info.mime + "\r\n"
        "Content-Length: " + to_string(contenido.size()) + "\r\n"
        "Content-Disposition: attachment; filename=\"" + info.nombreOriginal + "\"\r\n"
        "Connection: close\r\n\r\n";

    // Enviar dos veces: cabeceras y cuerpo
    send(cliente, cabeceras.c_str(), cabeceras.size(), 0);
    send(cliente, contenido.data(), contenido.size(), 0);
    return "";   // ya se envió
}
```

🎯 **Dos `send`:** uno para cabeceras, otro para cuerpo binario. No concatenar porque puede haber bytes nulos.

---

## Paso 8 — Frontend `archivos.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Archivos</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <header>
    <img src="/img/logo.png" alt="Logo" width="40">
    <h1>Gestión de archivos</h1>
    <button id="btn-logout">Cerrar sesión</button>
  </header>

  <main>
    <section>
      <h2>Subir archivo</h2>
      <form id="form-subir">
        <input type="file" id="input-archivo" required>
        <button type="submit">Subir</button>
      </form>
      <p id="subir-mensaje"></p>
    </section>

    <section>
      <h2>Archivos disponibles</h2>
      <button id="btn-actualizar">Actualizar</button>
      <table id="tabla-archivos">
        <thead>
          <tr>
            <th>Nombre</th>
            <th>Tamaño</th>
            <th>Fecha</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </section>
  </main>

  <script src="/js/archivos.js"></script>
</body>
</html>
```

---

## Paso 9 — Frontend `archivos.js`

```js
'use strict';

const tbody = document.querySelector('#tabla-archivos tbody');

document.addEventListener('DOMContentLoaded', () => {
  document.querySelector('#form-subir').addEventListener('submit', subirArchivo);
  document.querySelector('#btn-actualizar').addEventListener('click', cargarArchivos);
  document.querySelector('#btn-logout').addEventListener('click', async () => {
    await fetch('/api/logout', { method: 'POST' });
    window.location.href = '/login.html';
  });
  cargarArchivos();
});

async function subirArchivo(e) {
  e.preventDefault();
  const input = document.querySelector('#input-archivo');
  const mensaje = document.querySelector('#subir-mensaje');

  if (!input.files[0]) {
    mensaje.textContent = 'Selecciona un archivo';
    return;
  }
  if (input.files[0].size > 10 * 1024 * 1024) {
    mensaje.textContent = 'Máximo 10 MB';
    return;
  }

  const formData = new FormData();
  formData.append('archivo', input.files[0]);

  mensaje.textContent = 'Subiendo...';

  try {
    const res = await fetch('/api/subir', {
      method: 'POST',
      body: formData
      // ⚠️ NO especificar Content-Type. El navegador lo hace.
    });

    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    mensaje.textContent = 'Subido correctamente';
    input.value = '';
    cargarArchivos();
  } catch (err) {
    mensaje.textContent = `Error: ${err.message}`;
  }
}

async function cargarArchivos() {
  tbody.innerHTML = '';

  try {
    const res = await fetch('/api/archivos');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const archivos = await res.json();

    if (archivos.length === 0) {
      const tr = document.createElement('tr');
      const td = document.createElement('td');
      td.colSpan = 4;
      td.textContent = 'Sin archivos';
      tr.appendChild(td);
      tbody.appendChild(tr);
      return;
    }

    archivos.forEach(a => {
      const tr = document.createElement('tr');

      const tdNombre = document.createElement('td');
      tdNombre.textContent = a.nombre;

      const tdTamano = document.createElement('td');
      tdTamano.textContent = (a.tamano / 1024).toFixed(1) + ' KB';

      const tdFecha = document.createElement('td');
      tdFecha.textContent = a.fecha;

      const tdAcc = document.createElement('td');
      const enlace = document.createElement('a');
      enlace.href = `/archivo/${a.id}`;
      enlace.textContent = 'Descargar';
      enlace.setAttribute('download', '');
      tdAcc.appendChild(enlace);

      tr.appendChild(tdNombre);
      tr.appendChild(tdTamano);
      tr.appendChild(tdFecha);
      tr.appendChild(tdAcc);
      tbody.appendChild(tr);
    });
  } catch (err) {
    console.error(err);
  }
}
```

---

## Paso 10 — Verificar

1. Sube un archivo TXT. Verifica que `data/uploads/` tiene un archivo con nombre único.
2. Abre `data/uploads/index.txt`, debe haber una línea nueva.
3. Sube una imagen PNG. Descárgala. Compara con `md5sum` o `diff`.
4. Intenta subir `../hack.txt` → debe guardarse como `_hack.txt`.
5. Sube un archivo > 10 MB → debe rechazarse con 413.
6. Descarga un ID inexistente → 404.
7. Reinicia el servidor. La lista sigue.

---

## ✅ Checklist de la Actividad 41

- [ ] `data/uploads/` y `index.txt` existen y se actualizan.
- [ ] Los archivos se guardan con nombres sanitizados y únicos.
- [ ] El parseo multipart funciona con archivos reales.
- [ ] Los binarios se leen/escriben con `ios::binary`.
- [ ] `Content-Disposition: attachment` fuerza descarga.
- [ ] Se rechazan archivos > límite con 413.
- [ ] Nombres sanitizados evitan path traversal.

## 🧠 Mini-reto
Añade `DELETE /api/archivos/{id}` que elimine el archivo del disco y la entrada del índice. Solo admin.
