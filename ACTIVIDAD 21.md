# Actividad 21. Nuestro primer servidor web

## Temas

- Socket TCP
    
- HTTP básico
    
- Servidor web
    

## Ejercicio

Compilar y ejecutar un único `servidor.cpp`.

Crear la estructura:

```text
Proyecto/
├── servidor.cpp
├── public/
│   └── index.html
└── data/
```

Al abrir:

```text
http://localhost:8080
```

el servidor deberá mostrar automáticamente `index.html`.

Explicar cómo el navegador realiza una petición HTTP.

---
# TEORÍA

---

## 1. ¿Qué es un socket?

- **Definición:** Un socket es un **punto final de comunicación** entre dos programas a través de una red. Es la interfaz de programación que expone el sistema operativo para enviar y recibir datos por red.
- **Analogía:** Un socket es como un "teléfono" que tu programa usa para hablar con otro programa en la misma máquina o en otra.
- **Tipos principales:**
  - **TCP (SOCK_STREAM):** Orientado a conexión, confiable, los datos llegan en orden. Es lo que usa HTTP.
  - **UDP (SOCK_DGRAM):** Sin conexión, rápido pero sin garantías. No lo usaremos aquí.
- **En esta actividad:** Usarás TCP porque HTTP se transporta sobre TCP.

---

## 2. API de sockets en C++ (Berkeley sockets)

C++ no trae sockets en la biblioteca estándar. Se usan las **APIs del sistema operativo**:

- **Windows:** Winsock2 (`<winsock2.h>`, `<ws2tcpip.h>`), requiere inicializar con `WSAStartup` y enlazar con `-lws2_32`.
- **Linux/macOS:** POSIX sockets (`<sys/socket.h>`, `<netinet/in.h>`, `<arpa/inet.h>`, `<unistd.h>`).

**Diferencias clave entre plataformas:**

| Aspecto | Windows (Winsock) | Linux/macOS (POSIX) |
|---------|-------------------|---------------------|
| Inicialización | `WSAStartup(...)` | No requiere |
| Cierre de socket | `closesocket(s)` | `close(s)` |
| Enlace de librería | `-lws2_32` | Ninguno |
| Tipo de socket | `SOCKET` | `int` |
| Errores | `WSAGetLastError()` | `errno` |

- **Recomendación:** Usa `#ifdef _WIN32` para escribir código multiplataforma, o trabaja solo en tu sistema operativo objetivo (por ejemplo, Linux si el proyecto es para servidor).

---

## 3. Ciclo de vida de un socket servidor

Un servidor TCP pasa por esta secuencia de llamadas al sistema:

1. **Crear el socket:** `socket(AF_INET, SOCK_STREAM, 0)`.
   - `AF_INET`: IPv4.
   - `SOCK_STREAM`: TCP.
   - `0`: protocolo por defecto (TCP).
2. **Configurar la dirección:** Se llena una estructura `sockaddr_in` con:
   - `sin_family = AF_INET`
   - `sin_addr.s_addr = INADDR_ANY` (escuchar en todas las interfaces) o `inet_addr("127.0.0.1")` (solo local).
   - `sin_port = htons(PUERTO)` → `htons` convierte el puerto de orden de host a orden de red.
3. **Enlazar (bind):** `bind(sockfd, (sockaddr*)&direccion, sizeof(direccion))` → asocia el socket a un puerto.
4. **Escuchar (listen):** `listen(sockfd, BACKLOG)` → pone el socket en modo escucha.
   - `BACKLOG`: número máximo de conexiones pendientes en cola.
5. **Aceptar (accept):** `accept(sockfd, ...)` → **bloquea** hasta que llega una conexión. Devuelve un **nuevo socket** para esa conexión específica.
6. **Comunicarse:** `recv()` para leer, `send()` para escribir.
7. **Cerrar:** `close()` o `closesocket()`.

**Fragmentos sueltos (conceptuales):**
```cpp
int sockfd = socket(AF_INET, SOCK_STREAM, 0);

sockaddr_in dir;
dir.sin_family = AF_INET;
dir.sin_port = htons(8080);
dir.sin_addr.s_addr = INADDR_ANY;

bind(sockfd, (sockaddr*)&dir, sizeof(dir));
listen(sockfd, 10);

int cliente = accept(sockfd, nullptr, nullptr);

char buffer[4096];
int bytes = recv(cliente, buffer, sizeof(buffer), 0);
send(cliente, respuesta, strlen(respuesta), 0);

close(cliente);
close(sockfd);
```

- **Bucle infinito:** El servidor suele tener un `while(true)` alrededor de `accept()` para atender múltiples conexiones.
- **Un cliente a la vez:** Con este modelo (sin hilos), atiendes una conexión, la cierras, y vuelves a aceptar. Para múltiples clientes simultáneos se necesitan hilos o multiplexación (`select`, `poll`, `epoll`), que verás en actividades posteriores.

---

## 4. El navegador como cliente

- **El navegador es un cliente HTTP:** Cuando escribes `http://localhost:8080` y pulsas Enter, el navegador:
  1. Resuelve `localhost` a `127.0.0.1`.
  2. Abre una **conexión TCP** al puerto 8080.
  3. Envía una **petición HTTP** por esa conexión.
  4. Espera la **respuesta HTTP**.
  5. Cierra la conexión (o la reutiliza, según HTTP/1.1).
  6. Renderiza el contenido recibido.

- **Importante:** Tu servidor no necesita "conocer" el navegador. Solo necesita:
  - Escuchar en un puerto.
  - Leer bytes entrantes.
  - Interpretarlos como HTTP.
  - Responder con bytes en formato HTTP.

---

## 5. HTTP: el protocolo de la web

### A. ¿Qué es HTTP?

- **HTTP (HyperText Transfer Protocol):** Protocolo de capa de aplicación basado en **texto plano**, que define cómo los clientes (navegadores) y servidores intercambian información.
- **Modelo request-response:** El cliente envía una **petición**; el servidor envía una **respuesta**. Siempre.
- **Sin estado:** Cada petición es independiente (a menos que uses cookies/sesiones).
- **Sobre TCP:** Por defecto en el puerto 80 (HTTP) o 443 (HTTPS).

### B. Estructura de una petición HTTP

Una petición tiene tres partes:

1. **Línea de petición:**
   ```
   METODO RUTA VERSION
   ```
   Ejemplo: `GET /index.html HTTP/1.1`
   - **Métodos:** `GET` (obtener), `POST` (enviar datos), `PUT`, `DELETE`, `HEAD`, etc.
   - **Ruta:** Recurso solicitado (por ejemplo, `/`, `/index.html`, `/api/alumnos`).
   - **Versión:** Casi siempre `HTTP/1.1`.

2. **Cabeceras (headers):** Pares `Clave: Valor`, una por línea.
   - `Host: localhost:8080`
   - `User-Agent: Mozilla/5.0 ...`
   - `Accept: text/html,application/xhtml+xml,...`
   - `Content-Type` (solo en POST/PUT)
   - `Content-Length` (solo en POST/PUT)

3. **Cuerpo (body):** Opcional. Solo en métodos como `POST` o `PUT`. Separado de las cabeceras por una **línea vacía** (`\r\n\r\n`).

**Ejemplo conceptual de petición:**
```
GET /index.html HTTP/1.1\r\n
Host: localhost:8080\r\n
User-Agent: Mozilla/5.0\r\n
Accept: text/html\r\n
\r\n
```

- **Fin de cabeceras:** Se detecta por la **doble línea vacía** (`\r\n\r\n`).

### C. Estructura de una respuesta HTTP

Una respuesta también tiene tres partes:

1. **Línea de estado:**
   ```
   VERSION CODIGO DESCRIPCION
   ```
   Ejemplo: `HTTP/1.1 200 OK`
   - **Códigos comunes:**
     - `200 OK` → Todo bien.
     - `404 Not Found` → Recurso no existe.
     - `500 Internal Server Error` → Error del servidor.
     - `301/302` → Redirecciones.
     - `400 Bad Request` → Petición mal formada.

2. **Cabeceras:** Pares `Clave: Valor`.
   - `Content-Type: text/html` → tipo MIME del contenido.
   - `Content-Length: 1234` → tamaño del cuerpo en bytes.
   - `Connection: close` → cerrar tras la respuesta.

3. **Cuerpo:** El contenido real (HTML, CSS, JSON, imagen, etc.).

**Ejemplo conceptual de respuesta:**
```
HTTP/1.1 200 OK\r\n
Content-Type: text/html\r\n
Content-Length: 45\r\n
Connection: close\r\n
\r\n
<!DOCTYPE html><html>...</html>
```

- **Separación cabeceras/cuerpo:** Doble salto de línea (`\r\n\r\n`).
- **`\r\n`** es el fin de línea en HTTP (retorno de carro + nueva línea). **No uses solo `\n`.**

---

## 6. Tipos MIME (Content-Type)

- **Definición:** Identifican el tipo de contenido del cuerpo de la respuesta.
- **Necesarios para que el navegador sepa cómo interpretar los bytes.**
- **Tipos comunes:**
  - `.html` → `text/html`
  - `.css` → `text/css`
  - `.js` → `application/javascript`
  - `.json` → `application/json`
  - `.png` → `image/png`
  - `.jpg` / `.jpeg` → `image/jpeg`
  - `.svg` → `image/svg+xml`
  - `.ico` → `image/x-icon`
  - `.txt` → `text/plain`
- **Extensión → MIME:** Puedes implementar una función que reciba la extensión y devuelva el MIME correspondiente.

---

## 7. Servir archivos estáticos

- **Objetivo:** Cuando el navegador pide `/index.html`, el servidor lee el archivo de disco y lo envía.
- **Flujo típico:**
  1. Leer la petición con `recv`.
  2. Parsear la primera línea para obtener el método y la ruta.
  3. Si la ruta es `/`, convertirla en `/index.html` (recurso por defecto).
  4. Construir la ruta en disco: `public/` + ruta solicitada.
  5. Verificar que el archivo existe y se puede abrir.
  6. Leer el archivo completo (con `ifstream` en modo binario).
  7. Determinar el `Content-Type` por la extensión.
  8. Construir la respuesta HTTP con cabeceras + cuerpo.
  9. Enviar con `send`.
  10. Cerrar la conexión del cliente.

**Fragmentos sueltos:**
```cpp
ifstream archivo(rutaCompleta, ios::binary);
string contenido((istreambuf_iterator<char>(archivo)), istreambuf_iterator<char>());
```

- **Ruta base:** `public/` (carpeta del frontend). El servidor la conoce.
- **Seguridad:** Verifica que la ruta solicitada **no salga de `public/`** (evitar `../` para leer archivos fuera de la carpeta).
- **Archivo por defecto:** Si la ruta es `/`, sirve `index.html`.
- **404:** Si el archivo no existe, responde con `404 Not Found` y un HTML simple.

---

## 8. Parseo básico de la petición

- **Leer bytes con `recv`:** Puede que la petición llegue en varios paquetes. Para esta actividad, con una sola llamada suele bastar (las peticiones GET son pequeñas).
- **Parsear la primera línea:**
  - Buscar el primer `\n`.
  - Separar por espacios: `[método, ruta, versión]`.
- **Parsear cabeceras:** Opcional para esta actividad. Solo necesitas el método y la ruta.
- **Ignorar el cuerpo:** En GET no hay cuerpo. En POST sí, pero no lo necesitas todavía.

**Fragmentos sueltos (parseo):**
```cpp
string peticion(buffer, bytes);
size_t finLinea = peticion.find("\r\n");
string primeraLinea = peticion.substr(0, finLinea);

size_t p1 = primeraLinea.find(' ');
size_t p2 = primeraLinea.find(' ', p1 + 1);
string metodo = primeraLinea.substr(0, p1);
string ruta = primeraLinea.substr(p1 + 1, p2 - p1 - 1);
```

- **Método:** Casi siempre `GET`. Si es otro, puedes responder `405 Method Not Allowed` (opcional).
- **Ruta:** Puede incluir query string (`/buscar?nombre=Ana`). Para esta actividad, ignora lo que venga después de `?`.

---

## 9. Estructura del proyecto

```
Proyecto/
├── servidor.cpp
├── public/
│   └── index.html
└── data/
```

- **`servidor.cpp`:** Código del servidor en C++. Contiene:
  - Inicialización de sockets.
  - Configuración del puerto.
  - Bucle de aceptación.
  - Parseo de petición.
  - Lectura y envío de archivos.
- **`public/`:** Archivos estáticos servidos por el servidor (HTML, CSS, JS, imágenes). Es la raíz del sitio web.
- **`data/`:** Carpeta para archivos de datos (persistencia, archivos TXT, etc.). La usará el backend más adelante para guardar información, no para servir al navegador.

**Nota:** El servidor debe ejecutarse desde la raíz del proyecto, para que las rutas relativas (`public/index.html`) funcionen.

---

## 10. Compilación

- **Linux/macOS:**
  ```bash
  g++ servidor.cpp -o servidor
  ./servidor
  ```
- **Windows (MinGW):**
  ```bash
  g++ servidor.cpp -o servidor.exe -lws2_32
  servidor.exe
  ```
- **Windows (MSVC):** Se enlaza automáticamente con `Ws2_32.lib` si incluyes los headers correctos (o con `#pragma comment(lib, "ws2_32.lib")`).
- **Puerto:** Si el puerto 8080 está ocupado, el `bind` fallará. Verifica con `netstat -an | grep 8080` (Linux/Mac) o `netstat -an | findstr 8080` (Windows).
- **Permisos:** Puertos por debajo de 1024 requieren privilegios de administrador en Linux/Mac. Usa 8080 para evitar problemas.

---

## 11. Cómo el navegador realiza la petición HTTP

Flujo completo cuando escribes `http://localhost:8080` en el navegador:

1. **Parseo de la URL:**
   - Esquema: `http`
   - Host: `localhost`
   - Puerto: `8080` (por defecto, si no se especifica, sería 80)
   - Ruta: `/`

2. **Resolución DNS:**
   - `localhost` se resuelve a `127.0.0.1` (dirección de loopback).

3. **Conexión TCP:**
   - El navegador abre un socket TCP al `127.0.0.1:8080`.
   - Realiza el **three-way handshake**: SYN → SYN-ACK → ACK.

4. **Envío de la petición HTTP:**
   - Envía la primera línea: `GET / HTTP/1.1`
   - Cabeceras: `Host: localhost:8080`, `User-Agent`, `Accept`, etc.
   - Termina con `\r\n\r\n`.

5. **El servidor procesa:**
   - Recibe la petición.
   - Parsea la ruta `/` → sirve `public/index.html`.
   - Construye una respuesta HTTP con el archivo.
   - Envía la respuesta.

6. **El navegador recibe la respuesta:**
   - Parsea el estado (`200 OK`).
   - Lee las cabeceras (especialmente `Content-Type` y `Content-Length`).
   - Lee el cuerpo (HTML).

7. **Renderizado:**
   - El navegador parsea el HTML.
   - Descubre recursos adicionales (CSS, JS, imágenes) y **hace peticiones adicionales** por cada uno (por eso un servidor real debe atender varias peticiones seguidas).
   - Aplica estilos y ejecuta JavaScript.

8. **Cierre de conexión:**
   - Si el servidor envía `Connection: close`, cierra la conexión.
   - Si no, la mantiene abierta para reutilizarla (keep-alive).

**Observación importante:** Tu servidor debe poder atender **varias peticiones seguidas** porque el navegador pedirá `/index.html`, luego `/styles.css`, luego `/app.js`, etc. Si solo respondes a una y cierras el bucle, el CSS y el JS no se cargarán.

---

## 12. Consideraciones de diseño del servidor

- **Bucle infinito:** `while(true)` alrededor de `accept`.
- **Un cliente a la vez:** Simple pero funcional para desarrollo. Si dos clientes se conectan a la vez, uno espera al otro.
- **Manejo de errores:** Verifica el resultado de cada llamada al sistema (`socket`, `bind`, `listen`, `accept`, `recv`, `send`).
- **Lectura completa del archivo:** Usa el constructor de `string` con iteradores de `istreambuf_iterator` para leer todo de una vez.
- **Content-Length correcto:** Debe ser el tamaño **en bytes** del contenido. Usa `.size()` del string.
- **Cadena de respuesta:** Construye la respuesta como un solo string concatenando estado + cabeceras + `\r\n\r\n` + cuerpo. Una sola llamada a `send` es más eficiente.
- **Cierre:** Cierra el socket del cliente después de enviar la respuesta.
- **Cierre del servidor:** Ctrl+C en la terminal. No necesitas manejar la señal para esta actividad.

---

## 13. Buenas prácticas

- **Rutas relativas:** El servidor asume que se ejecuta desde la raíz del proyecto.
- **Verificar archivos:** Siempre comprueba que el archivo exista antes de leerlo.
- **Seguridad básica:** Rechaza rutas que contengan `..` para evitar traversal de directorios.
- **No exponer `data/`:** El servidor solo sirve archivos dentro de `public/`. Los datos (`data/`) son internos del backend.
- **Códigos HTTP correctos:** 200 para éxito, 404 para no encontrado, 500 para errores internos.
- **Cabeceras completas:** `Content-Type`, `Content-Length`, `Connection` son obligatorias para que el navegador se comporte bien.
- **Logging:** Imprime en consola cada petición recibida (método, ruta) para depurar.
- **Separación de responsabilidades:** El servidor se encarga solo de servir archivos estáticos en esta actividad. La lógica de negocio vendrá después, cuando integres el backend.
- **Documenta el puerto:** Deja claro en el README en qué puerto escucha el servidor.
- **Manejo de MIME:** Centraliza la tabla de extensiones → MIME en una función.

---

## 14. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`bind` falla: "Address already in use"** | El puerto está ocupado. Espera unos segundos o usa otro puerto. |
| **`bind` falla: "Permission denied"** | Estás usando un puerto < 1024. Usa 8080 o superior. |
| **El navegador muestra "Connection refused"** | El servidor no está corriendo, o escucha en otro puerto. |
| **El navegador muestra el HTML pero sin estilos** | El servidor no está sirviendo el CSS. Verifica que atiende varias peticiones y que el `Content-Type` es correcto. |
| **El HTML se muestra como texto** | Falta la cabecera `Content-Type: text/html`. |
| **La respuesta se corta** | `Content-Length` mal calculado (por ejemplo, contar caracteres en lugar de bytes). |
| **`recv` devuelve 0 o menos** | El cliente cerró la conexión o hubo error. Sal del bucle de esa conexión. |
| **Petición no se parsea bien** | El delimitador de línea es `\r\n`, no `\n`. Ajusta el parseo. |
| **Falta `WSAStartup` en Windows** | Winsock requiere inicialización antes de usar sockets. |
| **No se enlaza `-lws2_32` en Windows** | Error de "undefined reference". Agrega el flag al compilar. |
| **El servidor responde solo a la primera petición del navegador** | El bucle debe seguir aceptando conexiones. Cada recurso (CSS, JS, imagen) es una petición nueva. |
| **Se sirven archivos fuera de `public/`** | Riesgo de seguridad. Rechaza rutas con `..`. |
| **La consola se queda bloqueada** | `accept` bloquea hasta que llega una conexión. Es normal. Usa Ctrl+C para detener el servidor. |

---

## 15. Resumen de conceptos clave

- **Socket TCP:** Punto final de comunicación sobre el que se construye HTTP.
- **API de sockets:** `socket`, `bind`, `listen`, `accept`, `recv`, `send`, `close` (con variantes en Windows).
- **Servidor TCP:** Crea socket, enlaza a puerto, escucha, acepta conexiones en bucle.
- **HTTP:** Protocolo de texto, modelo request-response, sobre TCP.
- **Petición HTTP:** Línea de petición + cabeceras + `\r\n\r\n` + cuerpo opcional.
- **Respuesta HTTP:** Línea de estado + cabeceras + `\r\n\r\n` + cuerpo.
- **Tipos MIME:** Identifican el contenido del cuerpo (`text/html`, `text/css`, etc.).
- **Archivos estáticos:** Se sirven desde `public/`, con la ruta relativa al proyecto.
- **Estructura del proyecto:** `servidor.cpp`, `public/` (frontend), `data/` (datos internos).
- **Flujo del navegador:** URL → DNS → TCP → petición → respuesta → renderizado.
- **Múltiples peticiones:** El navegador pide HTML, luego CSS, luego JS; el servidor debe atenderlas en bucle.
- **Buenas prácticas:** Verificar errores, rutas seguras, códigos HTTP correctos, cabeceras completas, logging.

---
# PASO A PASO

---
# Actividad 21 — Nuestro primer servidor web (C++ + sockets)

## Objetivo
Compilar y correr `servidor.cpp`. Al abrir `http://localhost:8080` en el navegador, el servidor responde con `index.html`. Todo con sockets TCP crudos, sin librerías externas.

## Estructura del proyecto

```
Proyecto/
├── servidor.cpp
├── public/
│   └── index.html
└── data/
```

## Diferencias de plataforma (importante antes de escribir código)

| Aspecto              | Windows (Winsock)         | Linux/macOS (POSIX)   |
|----------------------|---------------------------|-----------------------|
| Headers              | `<winsock2.h>`, `<ws2tcpip.h>` | `<sys/socket.h>`, `<netinet/in.h>`, `<arpa/inet.h>`, `<unistd.h>` |
| Inicialización       | `WSAStartup(...)`         | no requiere           |
| Cerrar socket        | `closesocket(s)`          | `close(s)`            |
| Linker               | `-lws2_32`                | —                     |
| Tipo socket          | `SOCKET`                  | `int`                 |

Vamos a escribir código **multiplataforma** con `#ifdef _WIN32`.

---

## Paso 1 — Esqueleto con headers multiplataforma

Crea `servidor.cpp` con esto:

```cpp
#ifdef _WIN32
  #include <winsock2.h>
  #include <ws2tcpip.h>
  #pragma comment(lib, "ws2_32.lib")
  using socket_t = SOCKET;
  #define CLOSE_SOCKET closesocket
#else
  #include <sys/socket.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  #include <unistd.h>
  using socket_t = int;
  #define CLOSE_SOCKET close
  #define INVALID_SOCKET (-1)
#endif

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
using namespace std;

const int PUERTO = 8080;
const string CARPETA_PUBLICA = "public";
```

✅ Compila aunque todavía no hagas nada útil: `g++ servidor.cpp -o servidor`.

---

## Paso 2 — Inicializar Winsock en Windows

Dentro de `main`, al inicio:

```cpp
int main() {
#ifdef _WIN32
    WSADATA wsa;
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        cerr << "Error al inicializar Winsock" << endl;
        return 1;
    }
#endif

    // ... resto del código

#ifdef _WIN32
    WSACleanup();
#endif
    return 0;
}
```

---

## Paso 3 — Crear el socket del servidor

```cpp
socket_t servidor = socket(AF_INET, SOCK_STREAM, 0);
if (servidor == INVALID_SOCKET) {
    cerr << "Error al crear el socket" << endl;
    return 1;
}
```

- `AF_INET` → IPv4.
- `SOCK_STREAM` → TCP.
- `0` → protocolo por defecto.

---

## Paso 4 — bind al puerto

```cpp
sockaddr_in direccion{};
direccion.sin_family = AF_INET;
direccion.sin_addr.s_addr = INADDR_ANY;   // escuchar en todas las interfaces
direccion.sin_port = htons(PUERTO);       // htons: host → red

if (bind(servidor, (sockaddr*)&direccion, sizeof(direccion)) != 0) {
    cerr << "Error en bind (¿puerto ocupado?)" << endl;
    CLOSE_SOCKET(servidor);
    return 1;
}
```

📌 Si `bind` falla con "Address already in use", espera unos segundos y reintenta (TIME_WAIT del socket anterior).

---

## Paso 5 — listen

```cpp
if (listen(servidor, 10) != 0) {
    cerr << "Error en listen" << endl;
    return 1;
}
cout << "Servidor escuchando en http://localhost:" << PUERTO << endl;
```

`10` es el backlog: cuántas conexiones pendientes caben en la cola.

---

## Paso 6 — Bucle de accept

```cpp
while (true) {
    sockaddr_in clienteDir{};
#ifdef _WIN32
    int tam = sizeof(clienteDir);
#else
    socklen_t tam = sizeof(clienteDir);
#endif

    socket_t cliente = accept(servidor, (sockaddr*)&clienteDir, &tam);
    if (cliente == INVALID_SOCKET) {
        cerr << "Error en accept" << endl;
        continue;
    }

    // ... aquí leeremos la petición y responderemos
    CLOSE_SOCKET(cliente);
}
```

⚠️ Con este modelo **atendes una conexión a la vez**. Está bien para la actividad.

---

## Paso 7 — Leer la petición

```cpp
char buffer[4096] = {0};
int bytes = recv(cliente, buffer, sizeof(buffer) - 1, 0);
if (bytes <= 0) {
    CLOSE_SOCKET(cliente);
    continue;
}
string peticion(buffer, bytes);

// Loguear la primera línea (método + ruta)
size_t finLinea = peticion.find("\r\n");
string primeraLinea = (finLinea != string::npos)
                      ? peticion.substr(0, finLinea)
                      : peticion;
cout << "[REQ] " << primeraLinea << endl;
```

📌 El delimitador HTTP es `\r\n` (CRLF), no solo `\n`.

---

## Paso 8 — Función para leer archivos

Antes de `main`, agrega:

```cpp
string leerArchivo(const string& ruta) {
    ifstream archivo(ruta, ios::binary);
    if (!archivo.is_open()) return "";

    string contenido((istreambuf_iterator<char>(archivo)),
                     istreambuf_iterator<char>());
    return contenido;
}
```

---

## Paso 9 — Servir index.html (versión mínima)

Dentro del bucle, después de leer la petición:

```cpp
// Por ahora: siempre devolvemos index.html
string cuerpo = leerArchivo("public/index.html");

string respuesta;
if (cuerpo.empty()) {
    respuesta =
        "HTTP/1.1 404 Not Found\r\n"
        "Content-Type: text/plain; charset=utf-8\r\n"
        "Content-Length: 15\r\n"
        "Connection: close\r\n"
        "\r\n"
        "Archivo no hallado";
} else {
    respuesta =
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html; charset=utf-8\r\n"
        "Content-Length: " + to_string(cuerpo.size()) + "\r\n"
        "Connection: close\r\n"
        "\r\n" +
        cuerpo;
}

send(cliente, respuesta.c_str(), respuesta.size(), 0);
```

📌 **Tres reglas de oro de HTTP:**
1. Cada cabecera termina en `\r\n`.
2. La última cabecera va seguida de una línea vacía (`\r\n\r\n`).
3. `Content-Length` es el tamaño **en bytes** del cuerpo.

---

## Paso 10 — Compilar y probar

Linux/macOS:
```bash
g++ servidor.cpp -o servidor
./servidor
```

Windows (MinGW):
```bash
g++ servidor.cpp -o servidor.exe -lws2_32
servidor.exe
```

Abre `http://localhost:8080`. Debe mostrarse el HTML de `public/index.html`.

---

## Paso 11 — Ver la petición que envía el navegador

Deja el `cout` de la primera línea. Recarga y observa la consola del servidor:

```
[REQ] GET / HTTP/1.1
[REQ] GET /favicon.ico HTTP/1.1
```

🎯 Aunque no lo pidas, el navegador pide `favicon.ico`. En la actividad 22 lo serviremos correctamente.

---

## ✅ Checklist de la Actividad 21

- [ ] El proyecto compila sin errores.
- [ ] Al abrir `http://localhost:8080` se ve `index.html`.
- [ ] En la consola del servidor aparece al menos una línea `[REQ] GET / HTTP/1.1`.
- [ ] Se cierra el cliente después de cada respuesta.
- [ ] Usas `\r\n` en las respuestas, no `\n`.

## 🧠 Mini-reto
Si el archivo `public/index.html` no existe, responde un HTML amigable de "404 Not Found" en lugar de texto plano. Sigue usando `Content-Type: text/html`.

---
