# Actividad 22. Servidor de archivos estáticos

## Temas

- Rutas
    
- Archivos
    
- MIME Types
    

## Ejercicio

Agregar nuevos archivos dentro de `public` sin modificar el código del servidor.

Por ejemplo:

```text
public/
├── index.html
├── login.html
├── alumnos.html
├── materias.html
├── css/
├── js/
└── img/
```

Verificar que todos funcionen automáticamente.

Como reto, agregar un favicon y algunas imágenes.

---
# TEORÍA

---

## 1. ¿Qué es un servidor de archivos estáticos?

- **Definición:** Un servidor que entrega archivos tal cual están en disco (HTML, CSS, JS, imágenes, fuentes, JSON, etc.) sin procesarlos ni transformarlos.
- **Diferencia con un servidor de aplicaciones:** Un servidor de aplicaciones ejecuta lógica (por ejemplo, consultar una base de datos, generar HTML dinámico). Un servidor estático solo lee y envía bytes.
- **Ventaja:** Es genérico, rápido, simple. Añadir archivos no requiere tocar el código.
- **En esta actividad:** Tu servidor de la Actividad 21 deja de tener rutas "hardcodeadas" y pasa a mapear **cualquier ruta** a un archivo dentro de `public/`.

---

## 2. El concepto de "ruta"

- **Ruta URL (request URI):** Lo que el navegador envía en la primera línea de la petición. Ejemplos:
  - `/` → raíz del sitio.
  - `/index.html`
  - `/login.html`
  - `/css/styles.css`
  - `/js/app.js`
  - `/img/logo.png`
  - `/favicon.ico`

- **Ruta en disco:** La ubicación real del archivo en el sistema de archivos. Ejemplo:
  - URL `/css/styles.css` → disco `public/css/styles.css`.

- **Mapeo:** El servidor traduce la ruta URL a una ruta en disco:
  ```
  rutaDisco = carpetaRaiz + rutaURL
  ```
  donde `carpetaRaiz` es `public/`.

- **Ruta relativa vs. absoluta:**
  - La URL siempre empieza con `/` (es relativa a la raíz del sitio).
  - En disco, la ruta es relativa al directorio desde donde se ejecuta el servidor.
  - **Recomendación:** Combina `public/` + rutaURL, eliminando el `/` inicial si es necesario.

---

## 3. Normalización de la ruta

Antes de buscar el archivo en disco, la ruta URL debe **normalizarse** para:

- **Eliminar query strings:** Si la URL es `/buscar?nombre=Ana`, solo te interesa `/buscar`.
  - Detecta el primer `?` y corta la cadena ahí.
- **Eliminar fragmentos:** Si la URL es `/login#seccion`, corta en el `#`.
- **Decodificar caracteres especiales:** El navegador envía URLs codificadas (URL encoding). Por ejemplo:
  - `%20` → espacio.
  - `%C3%B1` → `ñ`.
  - `%2F` → `/`.
  - Puedes implementar una función `urlDecode` o, para esta actividad, ignorarlo si no usas caracteres especiales.
- **Ruta raíz:** Si la ruta es `/` o está vacía, se convierte en `/index.html` (recurso por defecto).
- **Ruta de directorio:** Si la ruta termina en `/` (ej. `/css/`), normalmente se sirve `index.html` dentro de ese directorio o se genera un listado. Para esta actividad, es aceptable devolver 404 si no hay `index.html`.

**Fragmentos sueltos:**
```cpp
size_t pos = ruta.find('?');
if (pos != string::npos) ruta = ruta.substr(0, pos);

if (ruta == "/" || ruta.empty()) ruta = "/index.html";
```

---

## 4. Seguridad: evitar Directory Traversal

- **Problema:** Un atacante podría pedir `/../../etc/passwd` para salir de `public/` y leer archivos del sistema.
- **Solución:** Rechazar cualquier ruta que contenga `..` como segmento.
- **Estrategia:**
  - Verificar que la ruta no contenga `..`.
  - Verificar que la ruta final (normalizada) siga estando dentro de `public/`.
  - Verificar que no empiece con `/` absoluto que apunte fuera de `public/`.
- **Respuesta:** Si la ruta es sospechosa, responde con `400 Bad Request` o `403 Forbidden`.

**Fragmento suelto:**
```cpp
if (ruta.find("..") != string::npos) {
    // rechazar
}
```

- **Mejora avanzada:** Usar `realpath` (POSIX) o `GetFullPathName` (Windows) para verificar que la ruta resuelta esté dentro de `public/`.

---

## 5. Detección de archivos y directorios

- **¿Existe el archivo?** Antes de leerlo, verifica su existencia.
  - POSIX: `stat(ruta)` o `access(ruta, F_OK)`.
  - C++17: `std::filesystem::exists(ruta)` (recomendado si tienes compilador moderno).
  - Alternativa: intentar abrirlo con `ifstream` y verificar `.is_open()`.
- **¿Es un archivo o un directorio?** Si es un directorio, puedes:
  - Devolver 404 (simple).
  - Servir su `index.html` si existe.
  - Generar un listado de archivos (avanzado).
- **Recomendación para esta actividad:** Verifica existencia y que sea archivo regular. Si no existe, 404.

**Fragmento suelto (con `std::filesystem`):**
```cpp
#include <filesystem>
namespace fs = std::filesystem;

if (fs::exists(rutaDisco) && fs::is_regular_file(rutaDisco)) {
    // servir
} else {
    // 404
}
```

---

## 6. Lectura del archivo

- **Modo binario:** Abre el archivo con `ios::binary` para evitar conversiones de fin de línea (`\n` ↔ `\r\n`).
- **Lectura completa:** Lee todo el contenido en un solo string usando iteradores de flujo.
  ```cpp
  ifstream archivo(rutaDisco, ios::binary);
  string contenido((istreambuf_iterator<char>(archivo)), istreambuf_iterator<char>());
  ```
- **Tamaño del archivo:** Puedes obtenerlo con `fs::file_size(ruta)` o usando `.seekg`/`.tellg` en el stream.
- **Errores:** Verifica que el archivo se abrió correctamente antes de leerlo.
- **Memoria:** Para archivos grandes (por ejemplo, videos), cargar todo en memoria no es eficiente. Se haría en chunks. Para esta actividad, los archivos son pequeños, así que leer completo está bien.

---

## 7. MIME Types (Content-Type)

- **Definición:** El MIME Type le dice al navegador **qué tipo de contenido** está recibiendo, para que sepa cómo interpretarlo.
- **Consecuencias de un MIME incorrecto:**
  - HTML servido como `text/plain` → el navegador lo muestra como texto plano en lugar de renderizarlo.
  - CSS servido como `text/html` → el navegador no lo aplica como estilo.
  - PNG servido como `application/octet-stream` → el navegador lo descarga en lugar de mostrarlo.

- **Tabla de extensiones → MIME (comunes):**

| Extensión | MIME Type |
|-----------|-----------|
| `.html`, `.htm` | `text/html` |
| `.css` | `text/css` |
| `.js` | `application/javascript` |
| `.json` | `application/json` |
| `.xml` | `application/xml` |
| `.txt` | `text/plain` |
| `.png` | `image/png` |
| `.jpg`, `.jpeg` | `image/jpeg` |
| `.gif` | `image/gif` |
| `.svg` | `image/svg+xml` |
| `.ico` | `image/x-icon` |
| `.webp` | `image/webp` |
| `.woff` | `font/woff` |
| `.woff2` | `font/woff2` |
| `.ttf` | `font/ttf` |
| `.pdf` | `application/pdf` |
| `.zip` | `application/zip` |
| `.mp3` | `audio/mpeg` |
| `.mp4` | `video/mp4` |

- **Cómo obtener la extensión:** Busca el último `.` en la ruta. Cuidado con archivos sin extensión (por ejemplo, `favicon.ico` sí tiene, pero `/api/alumnos` no).
  ```cpp
  size_t punto = ruta.find_last_of('.');
  string extension = (punto != string::npos) ? ruta.substr(punto) : "";
  ```
- **Valor por defecto:** Si la extensión no se reconoce, usa `application/octet-stream` (fuerza descarga en lugar de mostrarlo).
- **Implementación:** Una función que reciba la extensión y devuelva el MIME correspondiente usando un `if-else` o un `map<string, string>`.

**Fragmento suelto (con `map`):**
```cpp
#include <map>
map<string, string> mimes = {
    {".html", "text/html"},
    {".css", "text/css"},
    {".js", "application/javascript"}
};
string mime = mimes.count(ext) ? mimes[ext] : "application/octet-stream";
```

- **Consideración de `charset`:** Para texto (`text/html`, `text/css`, `application/javascript`), añade `; charset=utf-8` al final del Content-Type para evitar problemas con tildes.

---

## 8. Cabeceras de la respuesta

- **Content-Type:** Obligatorio. Determina cómo interpreta el navegador el cuerpo.
- **Content-Length:** Obligatorio en HTTP/1.1 si no usas transfer-encoding chunked. Debe ser el tamaño **en bytes** del cuerpo.
- **Connection:** `close` si vas a cerrar la conexión después de la respuesta. `keep-alive` si vas a reutilizarla.
- **Cache-Control (opcional):** Indica al navegador cuánto puede cachear el recurso.
  - `no-cache` → siempre revalidar.
  - `max-age=3600` → cachear por 1 hora.
  - Útil para imágenes y CSS/JS estáticos.
- **ETag / Last-Modified (opcional, avanzado):** Permiten cacheo condicional (el navegador pregunta si cambió). No es necesario para esta actividad.

---

## 9. Estructura de `public/`

- **Raíz del sitio:** Todo lo que esté dentro de `public/` es accesible desde el navegador.
- **Estructura típica:**
  ```
  public/
  ├── index.html          → http://localhost:8080/
  ├── login.html          → http://localhost:8080/login.html
  ├── alumnos.html        → http://localhost:8080/alumnos.html
  ├── materias.html       → http://localhost:8080/materias.html
  ├── favicon.ico         → http://localhost:8080/favicon.ico
  ├── css/
  │   └── styles.css      → http://localhost:8080/css/styles.css
  ├── js/
  │   └── app.js          → http://localhost:8080/js/app.js
  └── img/
      ├── logo.png        → http://localhost:8080/img/logo.png
      └── banner.jpg
  ```
- **Regla de oro:** Cualquier ruta URL corresponde a un archivo dentro de `public/`. El servidor no necesita saber qué archivos existen: simplemente intenta leerlos.
- **Añadir archivos:** No requiere recompilar ni reiniciar el servidor. Solo colocar el archivo en `public/` y acceder a su URL.

---

## 10. Enlaces relativos en el HTML

- **Rutas relativas:** Si un HTML está en la raíz, `css/styles.css` y `/css/styles.css` funcionan igual.
- **Rutas absolutas:** Siempre empiezan con `/` y son relativas a la **raíz del sitio** (no del sistema de archivos).
  - `/css/styles.css` → siempre `public/css/styles.css`, sin importar desde qué página se llame.
- **Rutas relativas al documento:** No empiezan con `/`. Se resuelven desde la ubicación del HTML actual.
  - Desde `public/alumnos.html`, `css/styles.css` → `public/css/styles.css`.
  - Pero desde `public/sub/seccion.html`, `css/styles.css` → `public/sub/css/styles.css` (probablemente no lo que quieres).
- **Recomendación:** Usa **rutas absolutas** (`/css/styles.css`) para evitar ambigüedades cuando el proyecto crezca.
- **Favicon:** El navegador pide automáticamente `/favicon.ico` en cada visita. Si no existe, verás un 404 en la consola del navegador (no rompe nada, pero es feo). Añádelo con `<link rel="icon" href="/favicon.ico">` o simplemente coloca el archivo en `public/favicon.ico`.

---

## 11. Favicon e imágenes (reto)

- **Favicon:**
  - Formato tradicional: `.ico` con múltiples tamaños (16×16, 32×32).
  - Formatos modernos: `.png`, `.svg`. Puedes enlazarlos con `<link rel="icon" type="image/png" href="/img/favicon.png">`.
  - Ubicación: `public/favicon.ico` es la ubicación por defecto que el navegador busca si no se especifica en el HTML.
- **Imágenes en HTML:**
  - `<img src="/img/logo.png" alt="Logo">`
  - Verifica que el MIME sea el correcto (`image/png`, `image/jpeg`, etc.), o el navegador no las mostrará.
- **Optimización (opcional):** Comprime imágenes antes de subirlas. Usa formatos modernos (WebP, SVG) cuando sea posible.
- **`alt`:** Siempre incluye texto alternativo por accesibilidad.

---

## 12. Logging y depuración

- **Imprime cada petición:** Método + ruta. Útil para ver qué recursos pide el navegador y en qué orden.
  ```
  GET /index.html
  GET /css/styles.css
  GET /js/app.js
  GET /img/logo.png
  GET /favicon.ico
  ```
- **Imprime el código de estado:** 200, 404, etc. Así sabes qué archivos no se encontraron.
- **Imprime el MIME asignado:** Si algo no se renderiza, verifica el Content-Type.
- **No imprimas todo el buffer recibido** (contiene cabeceras y puede ser verboso). Solo la primera línea es suficiente.

---

## 13. Buenas prácticas

- **Carpeta raíz configurable:** Define `public/` como constante al inicio para cambiarla fácilmente.
- **Funciones separadas:**
  - `obtenerMIME(extension)`
  - `leerArchivo(ruta)`
  - `construirRespuesta(estado, mime, contenido)`
  - `parsearPeticion(buffer)`
- **Tabla de MIME centralizada:** En un `map` o `switch`.
- **Rutas absolutas en HTML:** Evitan errores al mover archivos.
- **Seguridad:** Rechaza `..`, verifica que la ruta no salga de `public/`.
- **404 con HTML:** Devuelve una página 404 amigable en lugar de un texto plano.
- **`Content-Length` correcto:** Siempre igual al tamaño en bytes del cuerpo.
- **`Connection: close`:** Para esta actividad, cierra la conexión tras cada respuesta. Es más simple que implementar keep-alive.
- **No reinventar la rueda:** Si el proyecto crece, considera usar una librería HTTP (Crow, cpp-httplib, Boost.Beast) en lugar de sockets crudos.
- **Sin dependencias externas en esta actividad:** Todo con `ifstream`, `filesystem`, sockets y `map`.
- **Documenta el MIME por defecto:** Deja claro qué se devuelve cuando la extensión no se reconoce.

---

## 14. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **CSS/JS no se cargan** | Verifica el MIME correcto y que el servidor atiende múltiples peticiones. |
| **Imágenes no se muestran** | MIME incorrecto o ruta mal escrita. Verifica en la pestaña Network del navegador. |
| **Favicon da 404** | Agrega `public/favicon.ico` o un `<link rel="icon" href="...">` en el HTML. |
| **Rutas con `/` final dan 404** | Normaliza `/` a `/index.html` o sirve el `index.html` del directorio. |
| **Ruta con `?` o `#` da 404** | Corta la cadena antes de buscar el archivo. |
| **Se sirven archivos fuera de `public/`** | Rechaza `..` en la ruta. |
| **HTML se muestra como texto** | Falta `Content-Type: text/html`. |
| **El servidor responde solo una vez y muere** | `accept` debe estar en un bucle infinito. |
| **El navegador no recarga cambios** | El navegador cachea. Usa Ctrl+F5 o abre en modo incógnito. |
| **Content-Length mal calculado** | Usa `.size()` del string de contenido (bytes). |
| **Encoding de tildes roto** | Añade `; charset=utf-8` al Content-Type y guarda los HTML en UTF-8. |
| **Windows no encuentra `std::filesystem`** | Usa C++17 (`-std=c++17`) y GCC ≥ 8 o MSVC 2017+. |

---

## 15. Cómo verificar que todo funciona

1. **Navega a `http://localhost:8080/`** → debe servir `index.html`.
2. **Navega a `http://localhost:8080/login.html`** → debe servir `login.html`.
3. **Navega a `http://localhost:8080/css/styles.css`** → debe devolver el CSS con MIME `text/css`.
4. **Navega a `http://localhost:8080/js/app.js`** → MIME `application/javascript`.
5. **Abre la consola del navegador (F12) → pestaña Network.** Verifica:
   - Todas las peticiones devuelven 200.
   - Los `Content-Type` son correctos.
   - No hay 404 inesperados.
6. **Agrega una imagen nueva a `public/img/`** y accede a ella desde el navegador **sin reiniciar el servidor.** Debe funcionar.
7. **Agrega un `favicon.ico`** y recarga. El icono aparece en la pestaña del navegador.
8. **Intenta acceder a `http://localhost:8080/../servidor.cpp`** → debe devolver 400 o 404 (seguridad).
9. **Intenta acceder a `http://localhost:8080/noexiste.html`** → debe devolver 404.

---

## 16. Resumen de conceptos clave

- **Servidor estático:** Entrega archivos tal cual están en `public/`, sin lógica de negocio.
- **Mapeo de rutas:** URL `/x/y.z` → disco `public/x/y.z`.
- **Normalización:** Quitar `?`, `#`, decodificar URL, tratar `/` como `/index.html`.
- **Seguridad:** Rechazar `..` para evitar traversal.
- **Verificación:** `std::filesystem::exists` y `is_regular_file`.
- **Lectura:** `ifstream` en modo binario, leer completo con iteradores.
- **MIME Types:** Tabla extensión → tipo. `Content-Type` correcto es crítico.
- **Content-Length:** Tamaño en bytes del cuerpo.
- **Cabeceras HTTP:** Content-Type, Content-Length, Connection.
- **Rutas absolutas en HTML:** Evitan problemas al mover archivos.
- **Favicon:** `public/favicon.ico` o `<link rel="icon">`.
- **Añadir archivos no requiere tocar el código:** esa es la gracia del servidor estático.
- **Logging:** Imprime método y ruta de cada petición para depurar.

---
# PASO A PASO

---
# Actividad 22 — Servidor de archivos estáticos

## Objetivo
Convertir el servidor de la actividad 21 en un servidor **genérico**: cualquier archivo dentro de `public/` se sirve automáticamente. Sin rutas hardcodeadas.

## Regla de oro
`rutaURL` → `public/rutaURL`. Punto.

Si pides `/css/styles.css`, el servidor lee `public/css/styles.css`. Si pides `/img/logo.png`, lee `public/img/logo.png`. Añadir archivos nuevos **no requiere tocar el código**.

---

## Paso 1 — Nuevos includes

Arriba de `servidor.cpp`:

```cpp
#include <map>
#include <filesystem>
namespace fs = std::filesystem;
```

⚠️ `std::filesystem` requiere C++17. Compila con `-std=c++17`.

---

## Paso 2 — Tabla de MIME types

Antes de `main`:

```cpp
string obtenerMIME(const string& ruta) {
    size_t punto = ruta.find_last_of('.');
    if (punto == string::npos) return "application/octet-stream";

    string ext = ruta.substr(punto);

    static const map<string, string> mimes = {
        {".html", "text/html; charset=utf-8"},
        {".htm",  "text/html; charset=utf-8"},
        {".css",  "text/css; charset=utf-8"},
        {".js",   "application/javascript; charset=utf-8"},
        {".json", "application/json; charset=utf-8"},
        {".txt",  "text/plain; charset=utf-8"},
        {".xml",  "application/xml; charset=utf-8"},
        {".png",  "image/png"},
        {".jpg",  "image/jpeg"},
        {".jpeg", "image/jpeg"},
        {".gif",  "image/gif"},
        {".svg",  "image/svg+xml"},
        {".webp", "image/webp"},
        {".ico",  "image/x-icon"},
        {".woff", "font/woff"},
        {".woff2","font/woff2"},
        {".ttf",  "font/ttf"},
        {".pdf",  "application/pdf"},
        {".zip",  "application/zip"},
        {".mp3",  "audio/mpeg"},
        {".mp4",  "video/mp4"},
    };

    auto it = mimes.find(ext);
    return (it != mimes.end()) ? it->second : "application/octet-stream";
}
```

🎯 Sin el MIME correcto, el CSS no se aplica y las imágenes se descargan en vez de mostrarse.

---

## Paso 3 — Función para construir respuestas

```cpp
string construirRespuesta(int codigo, const string& mime, const string& cuerpo) {
    string estado;
    switch (codigo) {
        case 200: estado = "200 OK"; break;
        case 400: estado = "400 Bad Request"; break;
        case 403: estado = "403 Forbidden"; break;
        case 404: estado = "404 Not Found"; break;
        case 405: estado = "405 Method Not Allowed"; break;
        case 500: estado = "500 Internal Server Error"; break;
        default:  estado = "500 Internal Server Error";
    }

    return "HTTP/1.1 " + estado + "\r\n"
         + "Content-Type: " + mime + "\r\n"
         + "Content-Length: " + to_string(cuerpo.size()) + "\r\n"
         + "Connection: close\r\n"
         + "\r\n"
         + cuerpo;
}
```

Reutilizable por todas las respuestas a partir de ahora.

---

## Paso 4 — Parsear la ruta de la primera línea

Antes del bucle, agrega esta función:

```cpp
string extraerRuta(const string& peticion) {
    // Primera línea: "GET /ruta HTTP/1.1"
    size_t finLinea = peticion.find("\r\n");
    string primera = peticion.substr(0, finLinea);

    size_t p1 = primera.find(' ');
    if (p1 == string::npos) return "/";

    size_t p2 = primera.find(' ', p1 + 1);
    if (p2 == string::npos) return "/";

    return primera.substr(p1 + 1, p2 - p1 - 1);
}
```

---

## Paso 5 — Normalizar la ruta

```cpp
string normalizarRuta(string ruta) {
    // Quitar query string
    size_t q = ruta.find('?');
    if (q != string::npos) ruta = ruta.substr(0, q);

    // Quitar fragmento
    size_t h = ruta.find('#');
    if (h != string::npos) ruta = ruta.substr(0, h);

    // Ruta raíz → index.html
    if (ruta == "/" || ruta.empty()) ruta = "/index.html";

    return ruta;
}
```

🎯 Si el navegador pide `/buscar?q=ana`, solo nos interesa `/buscar`.

---

## Paso 6 — Seguridad: rechazar `..`

Antes de buscar en disco:

```cpp
bool rutaSegura(const string& ruta) {
    return ruta.find("..") == string::npos;
}
```

Si `rutaSegura` devuelve `false`, respondemos con `400 Bad Request`. Esto evita que alguien pida `/../../etc/passwd`.

---

## Paso 7 — Servir el archivo

Reemplaza el bloque del paso 9 de la actividad 21 por:

```cpp
string metodo = ...; // extraído también de la primera línea
string ruta   = extraerRuta(peticion);
ruta = normalizarRuta(ruta);

if (!rutaSegura(ruta)) {
    string cuerpo = "Ruta no permitida";
    string resp = construirRespuesta(400, "text/plain; charset=utf-8", cuerpo);
    send(cliente, resp.c_str(), resp.size(), 0);
    CLOSE_SOCKET(cliente);
    continue;
}

string rutaDisco = CARPETA_PUBLICA + ruta;

if (!fs::exists(rutaDisco) || !fs::is_regular_file(rutaDisco)) {
    string cuerpo = "<h1>404 Not Found</h1><p>Recurso no encontrado</p>";
    string resp = construirRespuesta(404, "text/html; charset=utf-8", cuerpo);
    send(cliente, resp.c_str(), resp.size(), 0);
    CLOSE_SOCKET(cliente);
    continue;
}

string contenido = leerArchivo(rutaDisco);
string mime = obtenerMIME(rutaDisco);

string resp = construirRespuesta(200, mime, contenido);
send(cliente, resp.c_str(), resp.size(), 0);

cout << "[200] " << ruta << " (" << mime << ")" << endl;
CLOSE_SOCKET(cliente);
```

📌 Observa el orden: se decide qué hacer **antes** de leer el archivo.

---

## Paso 8 — Probar con CSS, JS e imágenes

En `public/`, crea:

```
public/
├── index.html
├── styles.css
├── app.js
└── img/
    └── logo.png
```

En `index.html`:
```html
<link rel="stylesheet" href="/styles.css">
<script src="/app.js"></script>
<img src="/img/logo.png" alt="Logo">
```

Recarga `http://localhost:8080`. Todo debe cargar.

🎯 Abre DevTools → Network y verifica:
- `/` → 200 con `text/html`
- `/styles.css` → 200 con `text/css`
- `/app.js` → 200 con `application/javascript`
- `/img/logo.png` → 200 con `image/png`

---

## Paso 9 — Añadir un favicon

Crea `public/favicon.ico` (cualquier imagen cuadrada convertida a .ico). El navegador lo pedirá automáticamente y ya no dará 404.

---

## ✅ Checklist de la Actividad 22

- [ ] Se sirven HTML, CSS, JS e imágenes sin cambios en el código.
- [ ] Los `Content-Type` son correctos (verificado en DevTools).
- [ ] Al agregar un archivo nuevo a `public/`, se sirve sin recompilar.
- [ ] Pedir `/../servidor.cpp` devuelve 400 o 404.
- [ ] Pedir `/noexiste.html` devuelve 404.
- [ ] El favicon ya no da error.

## 🧠 Mini-reto
Sirve `/css/` (con barra al final) como `public/css/index.html` si existe; si no, responde 404.

---

