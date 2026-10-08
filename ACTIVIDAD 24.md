# Actividad 24. Creando nuestra primera API

## Temas

- Rutas dinámicas
    
- Backend
    

## Ejercicio

Agregar soporte para rutas:

```text
/api/hola
/api/version
/api/hora
/api/status
```

Cada ruta deberá devolver texto diferente.

Ejemplo:

```text
/api/hola
```

↓

```text
Hola desde C++
```

Explicar la diferencia entre servir archivos y generar respuestas dinámicamente.

---
# TEORÍA

---

## 1. ¿Qué es una API?

- **Definición:** API (Application Programming Interface) es un conjunto de **rutas y reglas** que un servidor expone para que otros programas (clientes) puedan **solicitar datos o ejecutar acciones**.
- **En el contexto web:** Una API HTTP es un conjunto de endpoints (URLs) que devuelven datos (típicamente JSON) en lugar de HTML renderizado.
- **Diferencia con un sitio web tradicional:**
  - **Sitio web:** Devuelve HTML, CSS, JS para que el navegador lo renderice para humanos.
  - **API:** Devuelve datos estructurados (JSON, XML, texto plano) para que otro programa los consuma.
- **En esta actividad:** Crearás endpoints simples que devuelven texto, para entender el mecanismo antes de complicarlo con JSON o lógica de negocio.

---

## 2. Servir archivos vs. generar respuestas dinámicas

Esta es la diferencia conceptual más importante de la actividad.

### A. Servir archivos (estático)

- **Qué hace:** Lee un archivo del disco y envía su contenido tal cual.
- **Origen del contenido:** Disco (`public/`).
- **Cuándo cambia:** Solo cuando cambias el archivo en disco.
- **Ejemplos:** `/index.html`, `/css/styles.css`, `/img/logo.png`.
- **Ventajas:** Simple, rápido, cacheable.
- **Desventajas:** No puede responder a datos del usuario, ni calcular nada, ni consultar una base de datos.

### B. Generar respuestas dinámicas (API)

- **Qué hace:** Construye el contenido **en el momento**, en memoria, según la petición, el estado del servidor o datos externos.
- **Origen del contenido:** El código C++ (variables, cálculos, archivos leídos, bases de datos).
- **Cuándo cambia:** Cada petición puede devolver algo diferente.
- **Ejemplos:** `/api/hora` (hora actual), `/api/version` (versión del servidor), `/api/alumnos` (lista desde archivo).
- **Ventajas:** Flexible, personalizable, puede combinar datos.
- **Desventajas:** No cacheable por defecto, más costoso de generar, requiere lógica.

### C. Comparación

| Aspecto | Archivo estático | Respuesta dinámica |
|---------|------------------|---------------------|
| Origen | Disco | Memoria (código) |
| Contenido | Fijo | Variable |
| Cambia | Al modificar archivo | En cada petición (si aplica) |
| Velocidad | Muy rápido | Depende de la lógica |
| Cacheable | Sí | No por defecto |
| Requiere lógica | No | Sí |

### D. Cómo conviven en el mismo servidor

- El servidor **decide** qué hacer según la ruta:
  - Si empieza por `/api/` → genera respuesta dinámica.
  - Si no → sirve archivo estático.
- Esta decisión se toma **antes** de intentar leer el archivo.

**Fragmento suelto (idea conceptual):**
```cpp
if (ruta.rfind("/api/", 0) == 0) {
    // manejar API
} else {
    // servir archivo
}
```

- **`rfind(..., 0) == 0`:** Verifica que la cadena empiece con `/api/`. Equivalente a `startsWith`.

---

## 3. Rutas dinámicas

- **Definición:** Rutas que no corresponden a un archivo en disco, sino a una **acción o recurso lógico** que el servidor resuelve con código.
- **Nomenclatura común:** Se prefijan con `/api/` para distinguirlas de los archivos estáticos.
- **Ejemplos de esta actividad:**
  - `/api/hola` → devuelve un saludo.
  - `/api/version` → devuelve la versión del servidor.
  - `/api/hora` → devuelve la hora actual.
  - `/api/status` → devuelve el estado del servidor.
- **Ventaja:** Puedes añadir rutas nuevas agregando un `if` o un `switch`. No dependen del sistema de archivos.

---

## 4. Enrutamiento (routing)

- **Definición:** El proceso de decidir qué código ejecutar según la ruta solicitada.
- **Enfoque simple (if-else):**
  - Comparas la ruta con cada endpoint conocido.
  - Si coincide, ejecutas su lógica.
  - Si no coincide ninguna, devuelves 404.
- **Enfoque con `switch`:** No funciona con strings directamente en C++. Puedes usar `if-else` o un `map<string, function>`.
- **Enfoque con `map` (más escalable):**
  - Asocias cada ruta a una función que genera la respuesta.
  - Buscas la ruta en el mapa; si existe, llamas a la función; si no, 404.
  - Ventaja: añadir endpoints es solo agregar una entrada al mapa.

**Fragmentos sueltos:**
```cpp
// if-else
if (ruta == "/api/hola") {
    // ...
} else if (ruta == "/api/version") {
    // ...
} else {
    // 404
}
```

```cpp
// map de funciones
map<string, function<string()>> rutas;
rutas["/api/hola"] = []() { return string("Hola desde C++"); };
```

- **Recomendación:** Para 4 rutas, `if-else` es claro y suficiente. Si el número crece, migra a un `map`.

---

## 5. Métodos HTTP en una API

- **`GET`:** Obtener datos. Es el método por defecto para las rutas de esta actividad.
- **`POST`:** Crear recursos. Lo usarás para registrar alumnos desde el formulario (actividades posteriores).
- **`PUT` / `PATCH`:** Actualizar recursos.
- **`DELETE`:** Eliminar recursos.

- **Para esta actividad:** Solo necesitas manejar `GET`. Si llega otro método a una ruta `/api/`, puedes:
  - Ignorarlo (responder igual).
  - Devolver `405 Method Not Allowed` (más correcto).

---

## 6. Construcción de la respuesta dinámica

- **Estructura:** Igual que la respuesta estática: línea de estado + cabeceras + `\r\n\r\n` + cuerpo.
- **Cabeceras clave:**
  - **`Content-Type`:** Depende del contenido:
    - Texto plano: `text/plain; charset=utf-8`
    - HTML generado: `text/html; charset=utf-8`
    - JSON: `application/json; charset=utf-8`
  - **`Content-Length`:** Tamaño en bytes del cuerpo generado.
  - **`Connection: close`** (o `keep-alive`).
- **Códigos de estado comunes en una API:**
  - `200 OK` → Respuesta exitosa.
  - `201 Created` → Recurso creado (POST).
  - `400 Bad Request` → Datos inválidos.
  - `404 Not Found` → Ruta no existe.
  - `405 Method Not Allowed` → Método incorrecto.
  - `500 Internal Server Error` → Error del servidor.

**Fragmento suelto (esqueleto):**
```cpp
string cuerpo = "Hola desde C++";
string respuesta = "HTTP/1.1 200 OK\r\n";
respuesta += "Content-Type: text/plain; charset=utf-8\r\n";
respuesta += "Content-Length: " + to_string(cuerpo.size()) + "\r\n";
respuesta += "Connection: close\r\n";
respuesta += "\r\n";
respuesta += cuerpo;
send(cliente, respuesta.c_str(), respuesta.size(), 0);
```

- **Concatenación:** Construye toda la respuesta en un solo `string` y envía de una vez (una sola llamada a `send`).
- **`\r\n`:** Fin de línea en HTTP. No uses solo `\n`.
- **`to_string(...)`:** Convierte el tamaño (size_t) a string para la cabecera.

---

## 7. Respuestas de cada endpoint

### A. `/api/hola`
- **Devuelve:** Un saludo, por ejemplo `Hola desde C++`.
- **Content-Type:** `text/plain`.
- **Sirve para:** Probar que el enrutamiento funciona.

### B. `/api/version`
- **Devuelve:** La versión del servidor, por ejemplo `Servidor v1.0.0`.
- **Content-Type:** `text/plain`.
- **Sirve para:** Reportar la versión del backend, útil en producción.

### C. `/api/hora`
- **Devuelve:** La hora actual del servidor.
- **Content-Type:** `text/plain`.
- **Cómo obtenerla:** Usa `<ctime>` con `time()` y `localtime()` / `strftime`.
  - `time(nullptr)` → tiempo actual (epoch).
  - `localtime(&t)` → estructura `tm` con año, mes, día, hora, minuto, segundo.
  - `strftime(buffer, tamaño, formato, &tm)` → formatea la fecha como string.
- **Fragmentos sueltos:**
  ```cpp
  time_t ahora = time(nullptr);
  tm* local = localtime(&ahora);
  char buffer[64];
  strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", local);
  string horaStr(buffer);
  ```
- **Sirve para:** Verificar que el servidor responde con datos dinámicos reales.

### D. `/api/status`
- **Devuelve:** El estado del servidor (siempre OK en esta actividad). Por ejemplo: `OK`.
- **Content-Type:** `text/plain`.
- **Uso real:** Los sistemas de monitoreo lo consultan periódicamente para saber si el servidor sigue vivo.
- **Sirve para:** Aprender el patrón de un endpoint "health check".

---

## 8. Respuesta por defecto para rutas `/api/` desconocidas

- Si la ruta empieza con `/api/` pero no coincide con ninguna conocida → **404 Not Found**.
- **Content-Type:** `text/plain` (o JSON si quieres más adelante).
- **Cuerpo:** Un mensaje como `Endpoint no encontrado`.
- **No servir archivos** para rutas `/api/` desconocidas: una API **nunca** devuelve HTML de error de disco.

**Fragmento suelto:**
```cpp
// dentro del bloque de /api/
} else {
    respuesta = "HTTP/1.1 404 Not Found\r\n";
    // ...
    cuerpo = "Endpoint no encontrado";
}
```

---

## 9. Organización del código

- **Función separada para API:** `string manejarAPI(const string& ruta, const string& metodo);`
  - Recibe la ruta y el método.
  - Devuelve la respuesta HTTP completa (o solo el cuerpo + un código de estado).
- **Ventajas:**
  - Separa la lógica de la API de la del servidor de archivos.
  - `main` solo decide si llamar a `manejarAPI` o `servirArchivo`.
- **Función separada para construir respuestas:** Evita repetir la construcción de cabeceras en cada endpoint.
  - `string construirRespuesta(int codigo, const string& mime, const string& cuerpo);`

**Fragmento suelto (estructura):**
```cpp
string manejarAPI(const string& ruta, const string& metodo) {
    if (ruta == "/api/hola") { return construirRespuesta(200, "text/plain", "Hola desde C++"); }
    if (ruta == "/api/version") { return construirRespuesta(200, "text/plain", "Servidor v1.0.0"); }
    // ...
    return construirRespuesta(404, "text/plain", "Endpoint no encontrado");
}
```

- **Regla:** Cada endpoint debe devolver **respuesta completa** (con cabeceras). Así `send` recibe una sola cadena lista.

---

## 10. Pruebas

- **En el navegador:** Puedes escribir `http://localhost:8080/api/hola` directamente en la barra de direcciones. El navegador envía `GET` y muestra el texto.
- **En la terminal:**
  - Linux/macOS: `curl http://localhost:8080/api/hola`
  - Windows: `curl http://localhost:8080/api/hola` (PowerShell tiene su propio `curl` con otro comportamiento; usa `curl.exe` si es necesario).
- **`curl -i`** para ver cabeceras completas.
- **Postman / Insomnia:** Herramientas gráficas para probar APIs (opcional).
- **DevTools → Network:** Al abrir la ruta desde el navegador, verás la petición y la respuesta en la pestaña Network.

**Consejo:** Usa `curl -i` desde terminal para verificar el código de estado, Content-Type y Content-Length. Es más rápido que el navegador.

---

## 11. Diferencia entre HTML de error y API

- **Archivo estático no encontrado:** El servidor devuelve un HTML de error (por ejemplo, una página `404 Not Found` con diseño).
- **Endpoint API no encontrado:** El servidor devuelve **texto plano** (o JSON) con un mensaje corto, porque el cliente de la API no renderiza HTML.
- **Regla:** El cliente de una API espera datos, no presentación.

---

## 12. Buenas prácticas

- **Prefijo `/api/`:** Diferencia claramente las rutas dinámicas de los archivos.
- **Content-Type correcto:** `text/plain` para texto, `application/json` para JSON.
- **`charset=utf-8`:** Siempre en tipos de texto.
- **Content-Length correcto:** Tamaño en bytes del cuerpo generado (`.size()`).
- **Códigos HTTP apropiados:** 200, 404, 405, 500 según corresponda.
- **Respuestas concisas:** Un endpoint, una responsabilidad.
- **Nunca expongas detalles internos** (rutas de disco, versiones de librerías, stack traces).
- **Log:** Imprime en consola cada llamada a `/api/` para depurar.
- **Construye la respuesta en una sola cadena** y envía con un solo `send`.
- **Valida el método:** Si solo aceptas GET, responde 405 para POST/PUT/DELETE.
- **No confundas errores de API con errores de archivo:** cada uno tiene su formato.
- **Documenta los endpoints** en el README: ruta, método, parámetros, respuesta.

---

## 13. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Devuelve HTML en lugar de texto** | Verifica `Content-Type: text/plain` para APIs simples. |
| **Content-Length no coincide con el cuerpo** | Usa `.size()` del string del cuerpo. |
| **La hora se ve rara** | Formatea con `strftime` correctamente. Añade `charset=utf-8` si hay caracteres especiales. |
| **404 confundido con error del servidor** | Verifica el código en la línea de estado y en el log. |
| **El navegador muestra todo en una línea** | Es normal si el Content-Type es `text/plain`. El navegador no respeta `\n` en texto plano. Usa `text/html` con `<br>` o `<pre>` si quieres saltos visibles. |
| **`curl` muestra "connection refused"** | El servidor no está corriendo, o estás en otro puerto. |
| **El endpoint devuelve 404 para todas las rutas** | Revisa el `if`/`else if` o el mapa de rutas. Cuidado con mayúsculas y con el `/` inicial. |
| **Comparación de strings fallida** | Usa `==`, no `=` (asignación). Cuida los espacios. |
| **El servidor deja de responder tras la primera petición API** | El `accept` debe seguir en bucle infinito. |
| **El navegador añade `/favicon.ico` automáticamente** | Es normal. Puedes servir un favicon o devolver 404 silenciosamente. |
| **Se sirve un archivo llamado `api`** | Verifica el orden de las comprobaciones: primero `/api/`, luego archivos. |

---

## 14. Resumen de conceptos clave

- **API:** Conjunto de endpoints que devuelven datos para otros programas.
- **Servir archivos vs. generar respuestas:**
  - Estático: lee de disco y envía tal cual.
  - Dinámico: construye en memoria según la lógica.
- **Rutas dinámicas:** No corresponden a archivos, se resuelven con código.
- **Prefijo `/api/`:** Convención para distinguir endpoints.
- **Enrutamiento:** `if-else` o `map<string, function>` para decidir qué ejecutar.
- **Construcción de respuesta:** Línea de estado + cabeceras + `\r\n\r\n` + cuerpo.
- **Content-Type:** `text/plain` o `application/json` según corresponda.
- **Content-Length:** Tamaño en bytes del cuerpo.
- **Códigos HTTP:** 200, 404, 405, 500.
- **Endpoints de esta actividad:** `/api/hola`, `/api/version`, `/api/hora`, `/api/status`.
- **Hora del servidor:** `time()` + `localtime()` + `strftime()`.
- **Fallback:** Ruta `/api/` desconocida → 404 con texto plano (no HTML).
- **Separación de código:** Función `manejarAPI` independiente de `servirArchivo`.
- **Pruebas:** Navegador, `curl -i`, DevTools → Network.

---
# PASO A PASO

---
# Actividad 24 — Primera API: rutas dinámicas

## Objetivo
Agregar endpoints `/api/...` que devuelven texto **generado en C++**, no leído de disco. Aprender a enrutar según la ruta y el método.

## Idea clave
- Si la ruta empieza con `/api/` → se genera respuesta.
- Si no → se sirve archivo estático (lo de la actividad 22).

---

## Paso 1 — Separar el manejo de API

Añade antes de `main`:

```cpp
#include <ctime>

string manejarAPI(const string& ruta, const string& metodo) {
    // Solo aceptamos GET por ahora
    if (metodo != "GET") {
        return construirRespuesta(405, "text/plain; charset=utf-8",
                                  "Método no permitido");
    }

    if (ruta == "/api/hola") {
        return construirRespuesta(200, "text/plain; charset=utf-8",
                                  "Hola desde C++");
    }

    if (ruta == "/api/version") {
        return construirRespuesta(200, "text/plain; charset=utf-8",
                                  "Servidor v1.0.0");
    }

    if (ruta == "/api/hora") {
        time_t ahora = time(nullptr);
        tm* local = localtime(&ahora);
        char buffer[64];
        strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", local);
        return construirRespuesta(200, "text/plain; charset=utf-8",
                                  string(buffer));
    }

    if (ruta == "/api/status") {
        return construirRespuesta(200, "text/plain; charset=utf-8", "OK");
    }

    return construirRespuesta(404, "text/plain; charset=utf-8",
                              "Endpoint no encontrado");
}
```

📌 Todos los endpoints usan `text/plain; charset=utf-8`. En la actividad 30 verás cómo devolver JSON.

---

## Paso 2 — Decidir en el bucle principal

Dentro del bucle, después de extraer `ruta` y `metodo`:

```cpp
bool esAPI = (ruta.rfind("/api/", 0) == 0);

if (esAPI) {
    string respuesta = manejarAPI(ruta, metodo);
    send(cliente, respuesta.c_str(), respuesta.size(), 0);
    CLOSE_SOCKET(cliente);
    continue;
}

// si no es API → sirve archivo estático (código de actividad 22)
```

🎯 `ruta.rfind("/api/", 0) == 0` significa "la ruta empieza con `/api/`". `rfind(pat, 0)` busca desde la posición 0, lo que en la práctica es un `startsWith`.

---

## Paso 3 — Probar en el navegador

Abre directamente en la barra:

- `http://localhost:8080/api/hola` → `Hola desde C++`
- `http://localhost:8080/api/version` → `Servidor v1.0.0`
- `http://localhost:8080/api/hora` → `2025-01-15 10:30:00`
- `http://localhost:8080/api/status` → `OK`
- `http://localhost:8080/api/pepito` → 404 con `Endpoint no encontrado`

---

## Paso 4 — Probar con curl

Desde terminal:

```bash
curl -i http://localhost:8080/api/hola
```

Salida esperada:

```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 15
Connection: close

Hola desde C++
```

`-i` incluye las cabeceras en la salida. Útil para verificar el código de estado y el MIME.

---

## Paso 5 — Log de peticiones API

Agrega un log dentro de `manejarAPI` o en el bucle:

```cpp
if (esAPI) {
    cout << "[API] " << metodo << " " << ruta << endl;
    // ...
}
```

---

## Paso 6 — Diferencia estático vs dinámico (para tenerlo claro)

| Aspecto           | Estático (public/)            | Dinámico (/api/)            |
|-------------------|--------------------------------|-----------------------------|
| Origen del cuerpo | Archivo en disco               | Código C++                  |
| Cambia sin reiniciar | Sí (solo reemplaza archivo)  | No (hay que recompilar)     |
| Content-Type      | Según extensión                | Elegido por el programador  |
| Cacheable         | Sí                             | No por defecto              |
| Caso de uso       | HTML, CSS, JS, imágenes        | Datos, acciones             |

---

## ✅ Checklist de la Actividad 24

- [ ] Los 4 endpoints responden correctamente.
- [ ] Un endpoint desconocido devuelve 404 con texto plano.
- [ ] Los métodos distintos de GET reciben 405.
- [ ] La consola del servidor muestra los logs `[API] GET /api/...`.
- [ ] Los archivos estáticos siguen funcionando.

## 🧠 Mini-reto
Agrega `/api/eco?msg=hola` que devuelva el contenido del parámetro `msg`. Pista: extrae la parte después de `?` con `find` y `substr`.

---
