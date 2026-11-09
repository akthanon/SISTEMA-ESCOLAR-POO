# Actividad 32. Organizando el servidor

## Temas

- Modularización
    
- Funciones
    

## Ejercicio

Tomar el `servidor.cpp` construido hasta ahora.

Separar el código en funciones.

Por ejemplo:

```cpp
leerPeticion();

obtenerRuta();

procesarGET();

procesarPOST();

leerArchivo();

guardarArchivo();

obtenerMime();

enviarRespuesta();
```

El programa debe seguir funcionando exactamente igual.

El objetivo es mejorar únicamente la organización del código.

---
# TEORÍA

---

## 1. ¿Por qué modularizar?

- **Problema:** Un `main` gigante con todo el código mezclado es difícil de leer, mantener, depurar y extender.
- **Solución:** Dividir el programa en **funciones pequeñas**, cada una con **una responsabilidad única**.
- **Beneficios:**
  - **Legibilidad:** Se entiende el flujo general leyendo `main`, y los detalles en cada función.
  - **Reutilización:** Funciones como `obtenerMime` o `leerArchivo` se usan en varios lugares.
  - **Mantenimiento:** Cambiar una parte no afecta a las demás.
  - **Depuración:** Aíslas el problema en una función concreta.
  - **Pruebas:** Puedes probar cada función por separado.
  - **Trabajo en equipo:** Varias personas pueden trabajar en funciones distintas.

---

## 2. Principios de diseño

- **Una función, una tarea.** Si una función hace dos cosas, divídela.
- **Nombres descriptivos.** `procesarGET` dice qué hace; `funcion1` no.
- **Funciones cortas.** Idealmente menos de 30-40 líneas. Si es más, probablemente hace demasiado.
- **Bajo acoplamiento.** Una función no debería depender de variables globales innecesarias.
- **Alta cohesión.** Todo lo que está dentro de una función debe estar relacionado con su tarea.
- **Parámetros claros.** Pasa lo que la función necesita, no más.
- **Retorno claro.** Lo que la función produce debe ser evidente por su tipo de retorno.

---

## 3. Estructura recomendada del archivo

Orden típico en un `servidor.cpp` modularizado:

1. **Includes** (`<iostream>`, `<fstream>`, `<sstream>`, `<string>`, `<vector>`, sockets, etc.).
2. **`using namespace std;`** (opcional, pero común en proyectos pequeños).
3. **Constantes globales** (puerto, rutas de archivos, carpeta raíz).
4. **Structs** (`Alumno`, `Peticion`, `Respuesta`, etc.).
5. **Prototipos de funciones.** Permiten que `main` las use aunque estén definidas después.
6. **`main`.** Solo orquesta: inicializa, carga datos, entra al bucle, acepta, delega.
7. **Definiciones de funciones.** Implementación de cada una.

- **Ventaja de los prototipos:** Puedes leer `main` primero y entender el flujo sin distraerte con detalles.

**Fragmento suelto (esqueleto):**
```cpp
// includes
#include <iostream>
// ...

// constantes
const int PUERTO = 8080;
const string RUTA_PUBLIC = "public/";
const string RUTA_DATOS = "data/usuarios.txt";

// structs
struct Alumno { /* ... */ };
struct Peticion { /* ... */ };
struct Respuesta { /* ... */ };

// prototipos
string leerPeticion(int cliente);
Peticion parsearPeticion(const string& cruda);
void procesarGET(/* ... */);
// ...

int main() {
    // orquesta
}

// definiciones
string leerPeticion(int cliente) { /* ... */ }
// ...
```

---

## 4. Funciones sugeridas y su responsabilidad

| Función | Responsabilidad |
|---------|-----------------|
| `leerPeticion(int cliente)` | Leer del socket hasta completar cabeceras + cuerpo. Devuelve el string crudo. |
| `parsearPeticion(const string& cruda)` | Separar método, ruta, cabeceras, cuerpo en un struct `Peticion`. |
| `obtenerRuta(const Peticion& p)` | Normalizar la ruta: quitar query string, decodificar, mapear `/` a `/index.html`. |
| `obtenerMetodo(const Peticion& p)` | Extraer el método HTTP (`GET`, `POST`, etc.). |
| `procesarGET(/* ... */)` | Manejar peticiones GET: rutas `/api/` → API, resto → archivos. |
| `procesarPOST(/* ... */)` | Manejar peticiones POST: parsear cuerpo, validar, guardar, responder. |
| `procesarPUT(/* ... */)` | Actualizar un recurso existente. |
| `procesarDELETE(/* ... */)` | Eliminar un recurso. |
| `leerArchivo(const string& ruta)` | Leer un archivo de disco. Devuelve contenido + éxito/error. |
| `guardarArchivo(const string& ruta, const string& contenido)` | Escribir contenido en un archivo (sobrescribir). |
| `agregarLinea(const string& ruta, const string& linea)` | Añadir una línea al final (append). |
| `obtenerMime(const string& extension)` | Traducir extensión a MIME Type. |
| `enviarRespuesta(int cliente, const Respuesta& r)` | Construir la respuesta HTTP completa y enviarla con `send`. |
| `construirRespuesta(int codigo, const string& mime, const string& cuerpo)` | Generar el string HTTP completo. |
| `cargarUsuarios()` | Cargar `usuarios.txt` al arrancar. |
| `manejarAPI(const Peticion& p)` | Enrutar rutas `/api/...` a su handler correspondiente. |
| `escaparJSON(const string& s)` | Escapar caracteres especiales para JSON. |
| `parsearJSON(const string& cuerpo)` | Extraer campos de un JSON simple (artesanal). |
| `leerSockets` / `inicializarSockets()` | Abstraer la inicialización (Windows vs. POSIX). |

- **No todas son necesarias si el proyecto es pequeño**, pero la lista te da una idea de cómo descomponer.
- **Agrupa funciones relacionadas** con comentarios de sección: `// --- API ---`, `// --- Archivos ---`, etc.

---

## 5. `main` como orquestador

- **`main` no debe contener lógica de negocio.** Solo debe:
  1. Inicializar (sockets, carga de datos).
  2. Crear el socket servidor, `bind`, `listen`.
  3. Bucle `while(true)`:
     - `accept`.
     - Leer la petición.
     - Parsearla.
     - Decidir si es GET, POST, etc.
     - Llamar al handler correspondiente.
     - Cerrar el socket del cliente.
- **Consejo:** Si `main` tiene más de 40-50 líneas, probablemente hay lógica que debería estar en funciones.

**Fragmento suelto (idea de `main` orquestador):**
```cpp
int main() {
    inicializarSockets();
    int servidor = crearSocketServidor(PUERTO);
    cargarUsuarios();

    while (true) {
        int cliente = accept(servidor, nullptr, nullptr);
        if (cliente < 0) continue;

        string cruda = leerPeticion(cliente);
        Peticion p = parsearPeticion(cruda);
        Respuesta r = enrutar(p);  // decide GET/POST/etc.
        enviarRespuesta(cliente, r);

        close(cliente);
    }
}
```

- **`enrutar`** puede ser una función que mira `p.metodo` y delega a `procesarGET`, `procesarPOST`, etc.
- **Observa:** No hay lógica de archivos ni de parseo dentro de `main`. Solo orquesta.

---

## 6. Structs para agrupar datos relacionados

- **`Peticion`:** método, ruta, cabeceras, cuerpo. Agrupa todo lo que viene del cliente.
- **`Respuesta`:** código de estado, Content-Type, cuerpo. Agrupa todo lo que vas a devolver.
- **`Alumno`:** nombre, edad, matrícula. Agrupa los datos de un alumno.
- **Ventaja:** Pasar un solo struct es más limpio que pasar múltiples parámetros. Y es extensible: si mañana añades una cabecera, solo cambias el struct.

**Fragmento suelto (structs):**
```cpp
struct Peticion {
    string metodo;
    string ruta;
    string cuerpo;
    // cabeceras si las necesitas
};

struct Respuesta {
    int codigo;
    string mime;
    string cuerpo;
};
```

- **Construir la respuesta HTTP completa** ocurre al final, en `enviarRespuesta`, a partir del struct `Respuesta`. Así los handlers solo piensan en "qué responder", no en "cómo formatearlo".

---

## 7. Separación por capas

Una buena modularización separa el código en capas lógicas:

### A. Capa de red (sockets)
- Inicialización (`WSAStartup`, o nada en POSIX).
- Creación, bind, listen, accept.
- Lectura y escritura de bytes.
- Funciones: `crearSocketServidor`, `aceptarConexion`, `leerPeticion`, `enviarRespuesta`.

### B. Capa HTTP (protocolo)
- Parseo de la petición (método, ruta, cabeceras, cuerpo).
- Construcción de la respuesta (estado, cabeceras, cuerpo).
- Funciones: `parsearPeticion`, `construirRespuesta`, `obtenerMime`.

### C. Capa de aplicación (rutas y lógica)
- Enrutamiento: decidir qué handler corresponde.
- Handlers: `procesarGET`, `procesarPOST`, `manejarAPI`.
- Funciones: `manejarAPI`, `procesarGET`, `procesarPOST`.

### D. Capa de datos (persistencia)
- Lectura/escritura de archivos.
- Parseo/serialización de líneas y JSON.
- Funciones: `leerArchivo`, `guardarArchivo`, `cargarUsuarios`, `escaparJSON`.

- **Ventaja:** Puedes cambiar una capa sin tocar las demás. Por ejemplo, migrar de TXT a SQLite solo afecta a la capa de datos.

---

## 8. Manejo de errores en cada capa

- **Capa de red:** `socket`, `bind`, `listen`, `accept`, `recv`, `send` pueden fallar. Cada uno verifica su resultado.
- **Capa HTTP:** Parseo puede fallar (petición mal formada). Devuelve un error claro (`bool ok` o código de estado).
- **Capa de aplicación:** Valida los datos del cliente. Responde 400 si son inválidos.
- **Capa de datos:** Verifica apertura de archivos. Devuelve `bool` o lanza excepciones.
- **Regla:** Cada función debe **fallar de forma controlada** (devolver un estado de error, no abortar el programa).
- **`try-catch`** en los puntos donde puede lanzarse una excepción (por ejemplo, `stoi`).

---

## 9. Comunicación entre funciones

- **Por valor:** Para datos pequeños y de solo lectura (`int`, `string`, structs pequeños).
- **Por referencia (`&`):** Para modificar o evitar copias costosas (`vector`, `string` grandes, structs).
- **Por referencia constante (`const&`):** Para solo lectura y evitar copias.
- **Retorno:** Devuelve el resultado (struct, string, bool). Evita depender de variables globales para "devolver" valores.
- **Variables globales:** Solo para constantes (puerto, rutas) o para el estado que debe compartirse ampliamente (por ejemplo, el vector de alumnos en memoria). Prefiere parámetros cuando sea posible.

---

## 10. Refactorización paso a paso

Refactorizar significa **cambiar la estructura sin cambiar el comportamiento**. Pasos seguros:

1. **Antes de tocar nada, haz una copia de seguridad** o commitea en git.
2. **Identifica bloques de código** que hagan una tarea concreta.
3. **Extrae uno a uno** a funciones. No todo de golpe.
4. **Compila y prueba después de cada extracción.** Así detectas errores al instante.
5. **Renombra variables** si es necesario para que el código sea más claro.
6. **Verifica que el comportamiento es idéntico** al final (mismas respuestas, mismos logs).
7. **Commit intermedio** cuando algo funcione. No esperes al final.

**Regla de oro:** **No añadas funcionalidad nueva durante la refactorización.** Solo reorganiza. Si algo no funciona después, sabes que fue por la reorganización, no por features nuevas.

---

## 11. Beneficios de los prototipos

- **Prototipos** al inicio del archivo permiten que `main` y otras funciones usen funciones definidas más abajo.
- **Sintaxis:** `tipoRetorno nombreFuncion(parametros);`
- **Ventaja:** Puedes leer el archivo de arriba a abajo: prototipos, `main`, y luego las implementaciones.
- **Alternativa:** Definir todas las funciones antes de `main`. Funciona, pero `main` queda al final y es menos legible.
- **Recomendación:** Prototipos + `main` arriba, implementaciones abajo.

---

## 12. División en múltiples archivos (avanzado, opcional)

- Cuando el proyecto crece mucho, se divide en `.h` (declaraciones) y `.cpp` (implementaciones):
  - `sockets.h` / `sockets.cpp`
  - `http.h` / `http.cpp`
  - `api.h` / `api.cpp`
  - `archivos.h` / `archivos.cpp`
  - `main.cpp`
- **Para esta actividad:** **No es necesario.** Un solo `servidor.cpp` modularizado con funciones es suficiente. La división en archivos es un paso posterior si el proyecto sigue creciendo.
- **Regla práctica:** Empieza con un archivo. Cuando pase de ~800-1000 líneas o cueste encontrar cosas, divídelo.

---

## 13. Comentarios y organización visual

- **Comentarios de sección:** Separadores visuales para agrupar funciones relacionadas.
  ```cpp
  // ==================== CAPA DE RED ====================
  // ==================== CAPA HTTP ====================
  // ==================== CAPA DE DATOS ====================
  ```
- **Comentario antes de cada función:** Breve descripción de qué hace, qué recibe, qué devuelve.
- **No comentes lo obvio.** `// incrementa i` antes de `i++` es ruido. Comenta el **por qué**, no el **qué**.
- **Indentación consistente:** 2 o 4 espacios. Elige y no mezcles.
- **Líneas en blanco** para separar bloques lógicos dentro de una función.

---

## 14. Buenas prácticas

- **Una función, una responsabilidad.** Si el nombre incluye "y" (por ejemplo, `leerYParsear`), probablemente son dos funciones.
- **Funciones cortas.** 20-40 líneas máximo. Si es más, divide.
- **Nombres en camelCase** (`leerPeticion`, `obtenerMime`, `enviarRespuesta`).
- **Parámetros mínimos.** Una función con 6 parámetros probablemente debería recibir un struct.
- **Evita la duplicación.** Si dos funciones repiten código, extráelo a una tercera.
- **No mezcles capas.** La capa de red no debe leer archivos. La capa de datos no debe construir respuestas HTTP.
- **Manejo de errores consistente.** Elige un patrón (`bool` + parámetro de salida, excepciones, código de retorno) y aplícalo en todas partes.
- **Constantes para valores mágicos.** `const int PUERTO = 8080;` en lugar de `8080` disperso por el código.
- **Sin `using namespace std;` en headers** (aquí no hay headers, pero tenlo en mente).
- **Compila con `-Wall -Wextra`** para que el compilador te avise de posibles errores.
- **Prueba después de cada cambio.** La refactorización sin pruebas es un salto al vacío.
- **Git es tu amigo.** Commits pequeños y frecuentes durante la refactorización. Si algo se rompe, puedes volver atrás.

---

## 15. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`main` sigue siendo gigante** | Extrae más bloques a funciones. `main` solo debe orquestar. |
| **Funciones que hacen demasiado** | Divide por responsabilidad. Si mezcla parseo y escritura, sepáralas. |
| **Pasar 10 parámetros a una función** | Crea un struct que los agrupe y pásalo. |
| **Variables globales por todos lados** | Reduce a las imprescindibles (constantes, estado compartido). Pasa el resto por parámetro. |
| **Olvidar prototipos y compilar antes de tiempo** | Define los prototipos antes de `main` o las funciones antes de usarlas. |
| **Cambiar comportamiento al refactorizar** | Prueba cada endpoint después de cada extracción. Compara respuestas. |
| **Errores de compilación en cascada** | Refactoriza una función a la vez. No toques todo de golpe. |
| **Duplicación de código entre GET y POST** | Extrae la parte común a una función auxiliar. |
| **Mezclar capas** | Aplica la regla: red ↔ HTTP ↔ aplicación ↔ datos. Si una función mezcla dos capas, divídela. |
| **Perder el manejo de errores al refactorizar** | Cada función debe seguir devolviendo error de la misma forma que antes. Prueba casos de fallo. |
| **Funciones con nombres confusos** | Renombra para que el nombre describa la acción. `hacer()` no dice nada; `procesarGET()` sí. |
| **No cerrar archivos o sockets** | Cada función que abre algo debe cerrarlo. O usar RAII (destructores). |

---

## 16. Verificación de la refactorización

Después de modularizar, verifica que **el comportamiento es idéntico**:

1. **Arranca el servidor.** Debe loguear lo mismo que antes (puerto, usuarios cargados).
2. **Prueba cada endpoint con `curl`**:
   - `curl http://localhost:8080/api/hola`
   - `curl http://localhost:8080/api/hora`
   - `curl http://localhost:8080/api/version`
   - `curl http://localhost:8080/api/status`
   - `curl http://localhost:8080/api/alumnos`
3. **Prueba un POST con datos válidos.** Verifica que se guarda en el archivo y que la respuesta es la misma.
4. **Prueba un POST con datos inválidos.** Verifica que responde 400 con el mismo mensaje.
5. **Prueba servir archivos estáticos** (`/index.html`, `/css/styles.css`).
6. **Prueba el flujo completo desde el navegador:** registrar, listar, editar, eliminar.
7. **Mira los logs en consola.** Deben ser los mismos que antes.
8. **Reinicia el servidor.** Los datos persistidos deben seguir ahí.

- **Si algo difiere, la refactorización cambió el comportamiento.** Revisa el último cambio y corrígelo.

---

## 17. Ejemplo conceptual de flujo modularizado

Cuando llega una petición:

1. `accept` acepta la conexión.
2. `leerPeticion` lee del socket y devuelve el string crudo.
3. `parsearPeticion` separa método, ruta y cuerpo en un struct `Peticion`.
4. `obtenerRuta` normaliza la ruta.
5. `enrutar` decide si es GET, POST, PUT o DELETE según `p.metodo`.
6. Según la decisión:
   - `procesarGET` maneja rutas `/api/` y archivos estáticos.
   - `procesarPOST`, `procesarPUT`, `procesarDELETE` manejan sus respectivas operaciones.
7. Dentro de cada handler, se llaman funciones auxiliares:
   - `leerArchivo`, `guardarArchivo`, `agregarLinea` (capa datos).
   - `obtenerMime` (capa HTTP).
   - `construirRespuesta` (capa HTTP).
8. `enviarRespuesta` envía la respuesta al cliente.
9. `close` cierra el socket del cliente.

Cada paso es una función. `main` solo encadena las llamadas.

---

## 18. Resumen de conceptos clave

- **Modularizar:** Dividir el código en funciones con una responsabilidad cada una.
- **Beneficios:** Legibilidad, mantenimiento, reutilización, pruebas.
- **`main` orquestador:** Solo inicializa y encadena llamadas. Nada de lógica compleja.
- **Prototipos:** Al inicio del archivo para que `main` use funciones definidas después.
- **Structs:** Agrupan datos relacionados (`Peticion`, `Respuesta`, `Alumno`).
- **Capas:** Red, HTTP, aplicación, datos. No mezclar.
- **Funciones sugeridas:** `leerPeticion`, `parsearPeticion`, `procesarGET`, `procesarPOST`, `leerArchivo`, `guardarArchivo`, `obtenerMime`, `enviarRespuesta`, etc.
- **Manejo de errores consistente** en todas las capas.
- **Refactorización segura:** Cambios pequeños, compilar y probar tras cada uno.
- **No añadir funcionalidad** durante la refactorización.
- **Verificación:** Comportamiento idéntico al final.
- **Comentarios de sección** para agrupar funciones por capa.
- **Nombres descriptivos** y **funciones cortas**.
- **Un archivo es suficiente** para este proyecto; dividir en `.h`/`.cpp` es un paso posterior.

---
# PASO A PASO

---
# Actividad 32 — Organizando el servidor (modularización)

## Objetivo
Tomar el `servidor.cpp` que ya tienes (con todo en `main` y funciones sueltas) y **reorganizarlo en funciones pequeñas**, cada una con una responsabilidad. El comportamiento debe ser idéntico. Solo cambia la estructura.

## Antes de empezar
Haz una copia de seguridad:
```bash
cp servidor.cpp servidor.cpp.bak
```

Al final comparas el comportamiento entre `servidor.cpp.bak` y el refactorizado. Deben responder exactamente igual.

---

## Paso 1 — Identificar los bloques del `main`

Abre tu `servidor.cpp` actual y localiza los bloques dentro de `main`. Anota en papel (o en un comentario) qué hace cada uno:

```
main()
├── inicializar Winsock (Windows)
├── crear socket
├── bind
├── listen
├── cargar alumnos del archivo
└── while (true)
    ├── accept
    ├── leer petición
    ├── parsear método y ruta
    ├── decidir si es API o archivo
    │   ├── manejarAPI (a su vez, varios if)
    │   └── servir archivo
    ├── leer archivo
    ├── obtener MIME
    ├── construir respuesta
    └── enviar respuesta + cerrar cliente
```

Ese es el mapa de funciones que vas a extraer.

---

## Paso 2 — Prototipos arriba del archivo

Justo después de los `#include` y constantes, declara los prototipos. Aún no los implementas:

```cpp
// ============ PROTOTIPOS ============

// Capa de red
socket_t crearSocketServidor(int puerto);
string   leerPeticionCompleta(socket_t cliente);
void     enviarRespuesta(socket_t cliente, const string& respuesta);

// Capa HTTP
string   extraerMetodo(const string& peticion);
string   extraerRuta(const string& peticion);
string   extraerCuerpo(const string& peticion);
string   normalizarRuta(string ruta);
string   obtenerMIME(const string& ruta);
string   construirRespuesta(int codigo, const string& mime, const string& cuerpo);

// Capa de aplicación
string   manejarAPI(const string& ruta, const string& metodo, const string& cuerpo);
string   servirArchivo(const string& ruta);

// Capa de datos
string   leerArchivo(const string& ruta);
bool     guardarArchivo(const string& ruta, const string& contenido);
```

- **Ventaja:** Puedes leer los prototipos y ya entiendes la estructura del programa.
- **Compila:** Solo con prototipos compila (aunque falle al enlazar).

---

## Paso 3 — Extraer `crearSocketServidor`

Mueve toda la secuencia `socket` + `bind` + `listen` a su función:

```cpp
socket_t crearSocketServidor(int puerto) {
    socket_t servidor = socket(AF_INET, SOCK_STREAM, 0);
    if (servidor == INVALID_SOCKET) {
        cerr << "Error al crear socket\n";
        return INVALID_SOCKET;
    }

    int opt = 1;
    setsockopt(servidor, SOL_SOCKET, SO_REUSEADDR,
               (const char*)&opt, sizeof(opt));

    sockaddr_in dir{};
    dir.sin_family = AF_INET;
    dir.sin_addr.s_addr = INADDR_ANY;
    dir.sin_port = htons(puerto);

    if (bind(servidor, (sockaddr*)&dir, sizeof(dir)) != 0) {
        cerr << "Error en bind\n";
        CLOSE_SOCKET(servidor);
        return INVALID_SOCKET;
    }

    if (listen(servidor, 10) != 0) {
        cerr << "Error en listen\n";
        CLOSE_SOCKET(servidor);
        return INVALID_SOCKET;
    }

    return servidor;
}
```

En `main`:
```cpp
socket_t servidor = crearSocketServidor(PUERTO);
if (servidor == INVALID_SOCKET) return 1;
```

✅ Compila y prueba: `curl http://localhost:8080/` debe seguir funcionando.

---

## Paso 4 — Extraer `leerPeticionCompleta`

Ya la tienes de la actividad 28. Muévela tal cual (si no la tenías separada, sácala ahora del bucle):

```cpp
string leerPeticionCompleta(socket_t cliente) {
    string peticion;
    char buffer[4096];

    while (peticion.find("\r\n\r\n") == string::npos) {
        int n = recv(cliente, buffer, sizeof(buffer), 0);
        if (n <= 0) return peticion;
        peticion.append(buffer, n);
    }

    size_t posFin = peticion.find("\r\n\r\n");
    string cabeceras = peticion.substr(0, posFin);

    int contentLength = 0;
    size_t p = cabeceras.find("Content-Length:");
    if (p != string::npos) {
        size_t fin = cabeceras.find("\r\n", p);
        if (fin == string::npos) fin = cabeceras.size();
        string valor = cabeceras.substr(p + 15, fin - (p + 15));
        size_t i = valor.find_first_not_of(" \t");
        if (i != string::npos) valor = valor.substr(i);
        try { contentLength = stoi(valor); } catch (...) {}
    }

    size_t bodyInicio = posFin + 4;
    while (peticion.size() - bodyInicio < (size_t)contentLength) {
        int n = recv(cliente, buffer, sizeof(buffer), 0);
        if (n <= 0) break;
        peticion.append(buffer, n);
    }

    return peticion;
}
```

---

## Paso 5 — Extraer `construirRespuesta`

```cpp
string construirRespuesta(int codigo, const string& mime, const string& cuerpo) {
    string estado;
    switch (codigo) {
        case 200: estado = "200 OK"; break;
        case 201: estado = "201 Created"; break;
        case 400: estado = "400 Bad Request"; break;
        case 401: estado = "401 Unauthorized"; break;
        case 404: estado = "404 Not Found"; break;
        case 405: estado = "405 Method Not Allowed"; break;
        case 409: estado = "409 Conflict"; break;
        case 415: estado = "415 Unsupported Media Type"; break;
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

---

## Paso 6 — Extraer `extraerMetodo`, `extraerRuta`, `extraerCuerpo`

```cpp
string extraerMetodo(const string& peticion) {
    size_t fin = peticion.find("\r\n");
    string primera = peticion.substr(0, fin);
    size_t p = primera.find(' ');
    return (p != string::npos) ? primera.substr(0, p) : "";
}

string extraerRuta(const string& peticion) {
    size_t fin = peticion.find("\r\n");
    string primera = peticion.substr(0, fin);
    size_t p1 = primera.find(' ');
    if (p1 == string::npos) return "/";
    size_t p2 = primera.find(' ', p1 + 1);
    if (p2 == string::npos) return "/";
    return primera.substr(p1 + 1, p2 - p1 - 1);
}

string extraerCuerpo(const string& peticion) {
    size_t pos = peticion.find("\r\n\r\n");
    return (pos != string::npos) ? peticion.substr(pos + 4) : "";
}
```

---

## Paso 7 — Extraer `normalizarRuta` y `servirArchivo`

```cpp
string normalizarRuta(string ruta) {
    size_t q = ruta.find('?');
    if (q != string::npos) ruta = ruta.substr(0, q);
    size_t h = ruta.find('#');
    if (h != string::npos) ruta = ruta.substr(0, h);
    if (ruta == "/" || ruta.empty()) ruta = "/index.html";
    return ruta;
}

string servirArchivo(const string& ruta) {
    if (ruta.find("..") != string::npos) {
        return construirRespuesta(400, "text/plain; charset=utf-8",
                                  "Ruta no permitida");
    }

    string rutaDisco = CARPETA_PUBLICA + ruta;
    if (!fs::exists(rutaDisco) || !fs::is_regular_file(rutaDisco)) {
        string cuerpo = "<h1>404 Not Found</h1>";
        return construirRespuesta(404, "text/html; charset=utf-8", cuerpo);
    }

    string contenido = leerArchivo(rutaDisco);
    string mime = obtenerMIME(rutaDisco);
    return construirRespuesta(200, mime, contenido);
}
```

---

## Paso 8 — Nuevo `main` (orquestador)

```cpp
int main() {
#ifdef _WIN32
    WSADATA wsa;
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        cerr << "Error al inicializar Winsock\n";
        return 1;
    }
#endif

    cargarAlumnos();   // desde data/usuarios.txt

    socket_t servidor = crearSocketServidor(PUERTO);
    if (servidor == INVALID_SOCKET) return 1;

    cout << "Servidor escuchando en http://localhost:" << PUERTO << "\n";

    while (true) {
        sockaddr_in clienteDir{};
#ifdef _WIN32
        int tam = sizeof(clienteDir);
#else
        socklen_t tam = sizeof(clienteDir);
#endif
        socket_t cliente = accept(servidor, (sockaddr*)&clienteDir, &tam);
        if (cliente == INVALID_SOCKET) continue;

        string peticion = leerPeticionCompleta(cliente);
        if (peticion.empty()) {
            CLOSE_SOCKET(cliente);
            continue;
        }

        string metodo = extraerMetodo(peticion);
        string ruta   = normalizarRuta(extraerRuta(peticion));
        string cuerpo = extraerCuerpo(peticion);

        cout << "[REQ] " << metodo << " " << ruta << "\n";

        string respuesta;
        if (ruta.rfind("/api/", 0) == 0) {
            respuesta = manejarAPI(ruta, metodo, cuerpo);
        } else {
            respuesta = servirArchivo(ruta);
        }

        enviarRespuesta(cliente, respuesta);
        CLOSE_SOCKET(cliente);
    }

#ifdef _WIN32
    WSACleanup();
#endif
    return 0;
}
```

🎯 `main` pasó de cientos de líneas a ~40. Solo orquesta.

---

## Paso 9 — Verificar

1. Compila sin errores.
2. Levanta el servidor. Los logs deben ser idénticos a antes.
3. Prueba cada endpoint:
   ```bash
   curl -i http://localhost:8080/
   curl -i http://localhost:8080/api/hola
   curl -i http://localhost:8080/api/alumnos
   curl -i http://localhost:8080/noexiste.html
   ```
4. Registra un alumno con `POST`.
5. Abre `data/usuarios.txt` y verifica que se guardó.

Si algo difiere del `servidor.cpp.bak`, la refactorización cambió comportamiento. Revisa.

---

## Paso 10 — Comentarios de sección

Añade separadores para agrupar las funciones visualmente:

```cpp
// ==================== CAPA DE RED ====================
// crearSocketServidor, leerPeticionCompleta, enviarRespuesta

// ==================== CAPA HTTP ====================
// extraerMetodo, extraerRuta, normalizarRuta, construirRespuesta, obtenerMIME

// ==================== CAPA DE APLICACIÓN ====================
// manejarAPI, servirArchivo

// ==================== CAPA DE DATOS ====================
// cargarAlumnos, guardarAlumnos, leerArchivo, guardarArchivo
```

---

## ✅ Checklist de la Actividad 32

- [ ] `main` tiene menos de 50 líneas.
- [ ] Cada función hace una sola cosa.
- [ ] Hay prototipos arriba.
- [ ] Las funciones están agrupadas por capa con comentarios.
- [ ] Comparaste comportamiento con `servidor.cpp.bak` y es idéntico.
- [ ] Todos los endpoints siguen funcionando.

## 🧠 Mini-reto
Divide `manejarAPI` en funciones por endpoint (`manejarHola`, `manejarHora`, `manejarAlumnosGET`, `manejarAlumnosPOST`). La función `manejarAPI` solo debe enrutar.