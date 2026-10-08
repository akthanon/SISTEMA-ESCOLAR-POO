# Actividad 28. Procesando formularios en C++

## Temas

- Body HTTP
    
- Parseo de datos
    

## Ejercicio

Modificar el servidor para leer el cuerpo de las peticiones POST.

Extraer:

- nombre
    
- edad
    
- matrícula
    

Mostrar los datos recibidos en la consola del servidor.

Como reto, responder con un mensaje personalizado utilizando esos datos.

---
# TEORÍA

---

## 1. Recordatorio: anatomía de una petición HTTP

- **Línea de petición:** `POST /api/alumnos HTTP/1.1\r\n`
- **Cabeceras:** pares `Clave: Valor\r\n`, una por línea.
  - `Host: localhost:8080`
  - `Content-Type: application/json`
  - `Content-Length: 47`
- **Línea vacía:** `\r\n` (marca el fin de las cabeceras).
- **Cuerpo:** los datos en sí (JSON, texto, binario).
- **Separador cabeceras/cuerpo:** la secuencia **`\r\n\r\n`**.

**Regla clave:** Todo lo que hay **antes** del primer `\r\n\r\n` son cabeceras; todo lo que hay **después** es el cuerpo.

---

## 2. El problema de leer el cuerpo con sockets

- **`recv` no lee "un mensaje":** Lee **bytes de un buffer TCP**. No hay garantía de que:
  - Llegue toda la petición en un solo `recv`.
  - El `recv` termine exactamente donde termina el cuerpo.
  - Puede llegar en **varios paquetes**, y puede haber más datos después.
- **Para GET:** El cuerpo está vacío, no hay problema.
- **Para POST:** Necesitas saber **cuándo terminar de leer**. La cabecera `Content-Length` te lo dice.

---

## 3. Estrategia de lectura del cuerpo

Pasos recomendados (en pseudoconcepto):

1. **Leer en un bucle** con `recv` hasta acumular al menos las cabeceras completas (hasta encontrar `\r\n\r\n`).
2. **Parsear `Content-Length`** de las cabeceras. Si no está, el cuerpo es 0 (o inválido para POST con cuerpo).
3. **Calcular cuántos bytes del cuerpo ya tienes** en el buffer acumulado.
4. **Si te faltan bytes,** seguir llamando a `recv` hasta acumular `Content-Length` bytes.
5. **Cortar el cuerpo** como el substring después del `\r\n\r\n`, de longitud exactamente `Content-Length`.

**Fragmendo suelto (idea del bucle):**
```cpp
string peticion;
char buffer[4096];
while (peticion.find("\r\n\r\n") == string::npos) {
    int bytes = recv(cliente, buffer, sizeof(buffer), 0);
    if (bytes <= 0) break;  // error o conexión cerrada
    peticion.append(buffer, bytes);
}
```

- **Tamaño del buffer:** 4096 suele bastar para las cabeceras. Para el cuerpo, puede que necesites más.
- **Alternativa simplificada:** Si el POST es pequeño (un JSON de formulario), suele llegar completo en uno o dos `recv`. Pero **no confíes en eso** en producción.

---

## 4. Localizar la separación cabeceras/cuerpo

- **Busca `\r\n\r\n` con `find`:**
  ```cpp
  size_t pos = peticion.find("\r\n\r\n");
  ```
- **Si no lo encuentras,** la petición está incompleta. Sigue leyendo o descarta.
- **`pos`** es el índice donde **empieza** la secuencia `\r\n\r\n`.
- **Cabeceras:** `peticion.substr(0, pos)`.
- **Cuerpo:** `peticion.substr(pos + 4)` (los 4 caracteres de `\r\n\r\n`).

---

## 5. Parsear las cabeceras

- **Separar cabeceras en líneas:** Busca cada `\r\n`.
- **Cada línea:** Separa por el primer `:` → clave y valor. Limpia espacios al inicio del valor.
- **Buscar específicamente `Content-Length`:**
  ```cpp
  size_t inicio = cabeceras.find("Content-Length:");
  ```
- **Extraer el número:** Toma desde después de los dos puntos hasta el siguiente `\r\n`.
- **Convertir a entero:** `stoi` (con `try-catch` por si el valor es inválido).
- **Buscar `Content-Type`:** Para saber si el cuerpo es JSON o urlencoded.

**Fragmento suelto (extracción):**
```cpp
size_t p = cabeceras.find("Content-Length:");
size_t fin = cabeceras.find("\r\n", p);
string valor = cabeceras.substr(p + 15, fin - (p + 15));
int contentLength = stoi(valor);
```

- **Cuidado con el caso:** Las cabeceras HTTP son case-insensitive en teoría, pero en la práctica casi siempre vienen con la primera letra mayúscula. Puedes buscar ambas variantes o normalizar.
- **Espacios después de los dos puntos:** Son opcionales. Trim siempre el valor.

---

## 6. Leer el cuerpo completo

- **Cuerpo ya en el buffer:** Si `peticion.size() - (pos + 4) >= contentLength`, ya tienes todo.
- **Falta leer:** Si no, sigue llamando a `recv` hasta acumular `contentLength` bytes.
- **Corta exactamente `contentLength` bytes:** No más, porque puede haber datos extra (por ejemplo, otra petición en keep-alive).

**Fragmento suelto (idea):**
```cpp
size_t bodyInicio = pos + 4;
size_t bytesLeidos = peticion.size() - bodyInicio;
while (bytesLeidos < (size_t)contentLength) {
    int bytes = recv(cliente, buffer, sizeof(buffer), 0);
    if (bytes <= 0) break;
    peticion.append(buffer, bytes);
    bytesLeidos = peticion.size() - bodyInicio;
}
string cuerpo = peticion.substr(bodyInicio, contentLength);
```

---

## 7. Parseo del cuerpo según el Content-Type

### A. Cuerpo como JSON (`application/json`)

- El cuerpo es un string con formato JSON: `{"nombre":"Ana","edad":20,"matricula":"2024001"}`.
- **Parseo manual (sin librerías externas):** Es tedioso y frágil. Puedes hacerlo con búsquedas de strings, pero se rompe fácilmente con comillas escapadas, espacios, saltos de línea, etc.
- **Estrategia artesanal** (para esta actividad):
  1. Buscar la clave entre comillas: `"nombre"`.
  2. Encontrar los dos puntos `:` después de la clave.
  3. Saltar espacios.
  4. Leer el valor: si empieza con `"`, leer hasta el siguiente `"`. Si es número, leer hasta la coma o `}`.
  5. Cuidado con `,` después del último campo y con espacios.
- **Alternativa más robusta:** Usar una librería JSON para C++ (nlohmann/json, RapidJSON, jsoncpp). Pero **requiere agregar dependencias** (header-only en el caso de nlohmann, muy fácil de integrar).

**Fragmento suelto (artesanal, idea):**
```cpp
size_t pos = cuerpo.find("\"nombre\"");
// avanzar hasta ':', luego hasta la comilla inicial del valor
// leer hasta la siguiente comilla
```

- **Recomendación:** Para esta actividad, si el JSON es siempre el mismo formato (tres campos), el parseo artesanal es aceptable. Si el proyecto crece, usa nlohmann/json (header único, sin dependencias externas).

### B. Cuerpo como urlencoded (`application/x-www-form-urlencoded`)

- Formato: `nombre=Ana&edad=20&matricula=2024001`.
- **Separar por `&`** para obtener los pares `clave=valor`.
- **Separar cada par por el primer `=`** para obtener clave y valor.
- **Decodificar** los valores: `+` → espacio, `%XX` → byte correspondiente.
- **Más sencillo de parsear** que JSON, pero pierde tipos (todo string).

**Fragmento suelto (idea):**
```cpp
size_t amp = 0;
while ((amp = cuerpo.find('&', amp)) != string::npos) {
    // separar par, luego clave/valor
}
```

- **Recomendación:** Si el frontend envía JSON, parsea JSON. Si envía urlencoded, parsea urlencoded. **No mezcles.**

---

## 8. Extraer los campos específicos

- **Objetivo:** Obtener `nombre`, `edad`, `matricula` como variables separadas.
- **Claves exactas:** Deben coincidir **caracter por caracter** con lo que envía el frontend. Un espacio de más, una tilde, o mayúsculas distintas → no lo encuentras.
- **Conversión de tipos:**
  - `nombre` y `matricula` → `string`.
  - `edad` → `int`, con `stoi`.
  - Envuelve `stoi` en `try-catch` por si el valor no es numérico.
- **Valores por defecto:** Si un campo no se encuentra, usa string vacío o 0, y considera responder 400.
- **Verificación de campos faltantes:** Después de parsear, verifica que todos los campos esperados tengan valor.

**Fragmento suelto (conversión segura):**
```cpp
int edad = 0;
try {
    edad = stoi(valorEdad);
} catch (...) {
    // campo edad inválido
}
```

---

## 9. Mostrar los datos en consola

- **Formato claro y legible:**
  ```
  --- POST recibido ---
  Nombre: Ana
  Edad: 20
  Matrícula: 2024001
  ---------------------
  ```
- **Utilidad:** Verificar en desarrollo que el parseo fue correcto.
- **No imprimas el cuerpo crudo** (puede tener caracteres de control). Imprime los campos ya extraídos.
- **No imprimas en producción** información sensible del usuario (aunque en este proyecto no lo es).

---

## 10. Responder con un mensaje personalizado

- **Objetivo:** Que el servidor use los datos recibidos para construir una respuesta única.
- **Ejemplos de respuesta:**
  - `"Alumno Ana (matrícula 2024001) registrado correctamente."`
  - `{"ok":true,"mensaje":"Bienvenido Ana, edad 20"}`
- **Formato:** Elige uno y sé consistente. Recomendado: **JSON** para APIs.
- **Content-Type:** `application/json; charset=utf-8` si respondes JSON; `text/plain; charset=utf-8` si respondes texto.
- **Content-Length:** Calcula con `.size()` del cuerpo construido.
- **Escapado de strings:** Si el nombre del usuario incluye `"` o `\`, debes escaparlos en el JSON. Para esta actividad, si controlas los datos, no es crítico, pero tenlo en cuenta.
- **Código de estado:** `200 OK` (éxito) o `201 Created` (si consideras que se creó un recurso). Para esta actividad, 200 es suficiente.

**Fragmento suelto (idea de respuesta):**
```cpp
string mensaje = "Bienvenido " + nombre + ", tienes " + to_string(edad) + " años.";
string respuesta = "HTTP/1.1 200 OK\r\n";
respuesta += "Content-Type: text/plain; charset=utf-8\r\n";
respuesta += "Content-Length: " + to_string(mensaje.size()) + "\r\n";
respuesta += "Connection: close\r\n\r\n";
respuesta += mensaje;
```

---

## 11. Validación de datos (aunque no se almacene todavía)

- **Campos obligatorios:** `nombre`, `edad`, `matricula`. Si falta alguno → 400 Bad Request.
- **Tipos:** `edad` debe ser número entero positivo.
- **Formato:** `matricula` puede tener un patrón específico (por ejemplo, 7 dígitos).
- **Longitudes:** `nombre` no vacío, no demasiado largo.
- **Validación defensiva:** No confíes en el cliente. Aunque el cliente valide, el servidor debe revalidar.
- **Respuesta en caso de error:**
  - Código `400 Bad Request`.
  - Mensaje indicando qué falló.
  - Content-Type consistente con el resto de las respuestas.

---

## 12. Manejo de casos especiales

- **Content-Length ausente en un POST:** La petición es inválida. Responde 400.
- **Content-Length mayor que lo que llega:** Sigue leyendo; si el cliente cierra antes, descarta.
- **Content-Type inesperado:** Si esperas JSON y llega urlencoded, responde 415 Unsupported Media Type (o intenta parsear como puedas).
- **Cuerpo vacío:** Responde 400.
- **Cuerpo con BOM UTF-8:** Raro en peticiones, pero posible. Considera quitar los primeros 3 bytes si empiezan con `\xEF\xBB\xBF`.
- **Cuerpo con saltos de línea:** El JSON puede venir con `\n` si el cliente lo formatea. No asumas que todo va en una línea.

---

## 13. Organización del código

- **Función `leerPeticionCompleta(int socket)`:** Devuelve el string con toda la petición (cabeceras + cuerpo) tras leer hasta completar `Content-Length`. Encapsula la lógica de recepción.
- **Función `parsearPeticion(const string& peticion)`:** Devuelve un struct con método, ruta, cabeceras y cuerpo.
- **Función `parsearCuerpoJSON(const string& cuerpo)`:** Devuelve un struct con los campos extraídos (`nombre`, `edad`, `matricula`).
- **Función `construirRespuesta(int codigo, const string& mime, const string& cuerpo)`:** Genera la respuesta HTTP completa.
- **Ventaja:** Cada función hace una cosa y es más fácil de depurar.

**Fragmento suelto (struct de datos):**
```cpp
struct DatosAlumno {
    string nombre;
    int edad;
    string matricula;
    bool valido;
};
```

---

## 14. DevTools y verificación cruzada

- **En el navegador (DevTools → Network):**
  - Envía el formulario.
  - Abre la petición POST.
  - Pestaña **Payload**: verifica el JSON enviado.
- **En la consola del servidor:**
  - Debe imprimir los mismos valores.
  - Si no coinciden, el parseo está mal.
- **En la respuesta (DevTools → Response):**
  - El mensaje personalizado debe reflejar los datos.
  - Si el JSON de respuesta se ve mal, revisa el escapado.

---

## 15. Buenas prácticas

- **No confíes en el `recv` único:** Siempre lee hasta completar `Content-Length`.
- **Valida el `Content-Length`:** Debe ser un número positivo y razonable (evita DoS).
- **Corta el cuerpo exactamente:** No incluyas bytes sobrantes.
- **Parseo robusto:** Considera usar una librería JSON si el proyecto crece.
- **Maneja errores explícitamente:** 400 para datos malos, 415 para Content-Type incorrecto, 500 para fallos internos.
- **Loguea en desarrollo, no en producción** información sensible.
- **Escapa el contenido** al construir JSON de respuesta.
- **Sé consistente con las claves** entre frontend y backend (`nombre`, no `name`).
- **Convierte tipos** una vez, al parsear, no cada vez que uses el valor.
- **Separa lectura, parseo y construcción de respuesta** en funciones distintas.
- **Documenta el formato esperado** en el README del proyecto.
- **Prueba con `curl`** antes de probar con el navegador:
  - `curl -X POST -H "Content-Type: application/json" -d '{"nombre":"Ana","edad":20,"matricula":"2024001"}' http://localhost:8080/api/alumnos`
- **Añade mensajes de log útiles** para depurar, pero desactívalos en producción.

---

## 16. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **El cuerpo llega cortado** | No leíste hasta completar `Content-Length`. Sigue leyendo con `recv`. |
| **Content-Length mal parseado** | El offset tras `"Content-Length:"` es fijo, pero cuida los espacios. Trim el valor. |
| **Cuerpo vacío aunque el POST tenía datos** | No estás buscando `\r\n\r\n` correctamente o cortaste mal. |
| **Cuerpo con cabeceras pegadas** | Te falta sumar los 4 bytes del `\r\n\r\n`. |
| **JSON parseado a mano da resultados raros** | El formato tiene comillas escapadas, espacios o saltos de línea. Usa una librería. |
| **`stoi` lanza excepción** | El valor no es numérico o está fuera de rango. Envuelve en `try-catch`. |
| **Campos con tilde no se encuentran** | El frontend envía `nombre` con tilde o el servidor no decodifica UTF-8. Verifica encoding. |
| **Content-Type incorrecto** | El frontend envía JSON pero declara urlencoded, o al revés. Alinea ambos lados. |
| **El servidor responde 404 al POST** | Falta agregar la ruta POST en el enrutador. No es una ruta GET. |
| **La respuesta no tiene Content-Length** | Calcúlalo siempre. El navegador puede cortar la lectura antes. |
| **La respuesta se ve con tildes rotas** | Falta `; charset=utf-8` en el Content-Type. |
| **El cliente recibe HTML en lugar de JSON** | El servidor responde con HTML de error por defecto. Asegúrate de responder JSON desde el handler del POST. |
| **Funciona una vez y luego deja de responder** | El servidor no está en bucle infinito aceptando conexiones. |
| **En `recv` te quedas bloqueado esperando más datos** | El cliente ya envió todo. `recv` bloquea hasta que lleguen datos o se cierre la conexión. Usa `Content-Length` para saber cuándo parar. |
| **No se imprimen los datos en consola** | El handler del POST no se está ejecutando. Verifica el enrutamiento. |

---

## 17. Resumen de conceptos clave

- **Cuerpo HTTP:** Datos después del `\r\n\r\n`. Puede ser JSON, urlencoded, binario.
- **`Content-Length`:** Número de bytes del cuerpo. Indispensable para saber cuándo parar de leer.
- **Lectura en bucle:** `recv` no garantiza leer todo de una vez. Acumula hasta completar cabeceras + cuerpo.
- **Separación cabeceras/cuerpo:** Busca `\r\n\r\n`.
- **Parseo del `Content-Length`:** Busca la cabecera, extrae el número con `stoi`.
- **Corte del cuerpo:** `substr(pos + 4, contentLength)`.
- **Parseo JSON (artesanal):** Búsqueda de claves y valores. Aceptable para formatos simples.
- **Parseo urlencoded:** Separar por `&` y `=`; decodificar con URL decode.
- **Extracción de campos:** `nombre` (string), `edad` (int con `stoi`), `matricula` (string).
- **Validación:** Campos obligatorios, tipos, formatos.
- **Respuesta personalizada:** Construir con los datos recibidos. `Content-Type` y `Content-Length` correctos.
- **Manejo de errores:** 400, 415, 500 según corresponda.
- **Logs:** Imprime en consola para verificar. Útil en desarrollo, discreto en producción.
- **DevTools:** Verifica Payload y Response. Cruza con la consola del servidor.
- **Buenas prácticas:** Funciones separadas, parseo robusto, escape al construir JSON, consistencia de claves.

---
# PASO A PASO

---
# Actividad 28 — Procesar formularios en C++

## Objetivo
Modificar el servidor para **leer el cuerpo** de un POST y extraer sus campos. Mostrarlos por consola. Como reto, responder personalizadamente.

## Anatomía de un POST

```
POST /api/alumnos HTTP/1.1\r\n
Host: localhost:8080\r\n
Content-Type: application/json\r\n
Content-Length: 47\r\n
\r\n
{"nombre":"Ana","edad":20,"matricula":"A001"}
```

- Todo lo anterior al primer `\r\n\r\n` son **cabeceras**.
- Todo lo posterior es el **cuerpo**.
- `Content-Length` dice cuántos bytes tiene el cuerpo.

---

## Paso 1 — Leer toda la petición

`recv` no garantiza leer la petición completa en una sola llamada. Lee en bucle hasta tener cabeceras + cuerpo.

Agrega esta función:

```cpp
string leerPeticionCompleta(socket_t cliente) {
    string peticion;
    char buffer[4096];

    // 1) Leer hasta tener cabeceras completas
    while (peticion.find("\r\n\r\n") == string::npos) {
        int bytes = recv(cliente, buffer, sizeof(buffer), 0);
        if (bytes <= 0) return peticion;
        peticion.append(buffer, bytes);
    }

    // 2) Averiguar Content-Length
    size_t posCabecerasFin = peticion.find("\r\n\r\n");
    string cabeceras = peticion.substr(0, posCabecerasFin);

    int contentLength = 0;
    size_t p = cabeceras.find("Content-Length:");
    if (p != string::npos) {
        size_t fin = cabeceras.find("\r\n", p);
        if (fin == string::npos) fin = cabeceras.size();
        string valor = cabeceras.substr(p + 15, fin - (p + 15));
        // Trim espacios iniciales
        size_t i = valor.find_first_not_of(" \t");
        if (i != string::npos) valor = valor.substr(i);
        try { contentLength = stoi(valor); } catch (...) { contentLength = 0; }
    }

    // 3) Asegurar que tenemos todo el cuerpo
    size_t bodyInicio = posCabecerasFin + 4;
    size_t bytesCuerpo = peticion.size() - bodyInicio;

    while (bytesCuerpo < (size_t)contentLength) {
        int bytes = recv(cliente, buffer, sizeof(buffer), 0);
        if (bytes <= 0) break;
        peticion.append(buffer, bytes);
        bytesCuerpo = peticion.size() - bodyInicio;
    }

    return peticion;
}
```

---

## Paso 2 — Separar cabeceras y cuerpo

```cpp
string extraerCuerpo(const string& peticion) {
    size_t pos = peticion.find("\r\n\r\n");
    if (pos == string::npos) return "";
    return peticion.substr(pos + 4);
}
```

---

## Paso 3 — Extraer la ruta y el método

Ya lo tienes de la actividad 24 con `extraerRuta`. Agrega también el método:

```cpp
string extraerMetodo(const string& peticion) {
    size_t finLinea = peticion.find("\r\n");
    string primera = peticion.substr(0, finLinea);
    size_t p = primera.find(' ');
    return (p != string::npos) ? primera.substr(0, p) : "";
}
```

---

## Paso 4 — Parsear JSON (artesanal)

Para JSON simple con 3 campos, una función por campo sirve:

```cpp
string extraerValorJSON(const string& cuerpo, const string& clave) {
    string buscar = "\"" + clave + "\"";
    size_t pos = cuerpo.find(buscar);
    if (pos == string::npos) return "";

    size_t dosPuntos = cuerpo.find(':', pos + buscar.size());
    if (dosPuntos == string::npos) return "";

    size_t i = dosPuntos + 1;
    while (i < cuerpo.size() && isspace((unsigned char)cuerpo[i])) i++;

    if (i < cuerpo.size() && cuerpo[i] == '"') {
        // Valor string: leer hasta la siguiente "
        size_t fin = cuerpo.find('"', i + 1);
        if (fin == string::npos) return "";
        return cuerpo.substr(i + 1, fin - i - 1);
    }

    // Valor numérico u otro: leer hasta , o }
    size_t fin = i;
    while (fin < cuerpo.size() && cuerpo[fin] != ',' && cuerpo[fin] != '}') fin++;
    string valor = cuerpo.substr(i, fin - i);
    // Trim
    while (!valor.empty() && isspace((unsigned char)valor.back())) valor.pop_back();
    return valor;
}
```

⚠️ Esta versión es suficiente para el formato exacto que enviamos. No maneja comillas escapadas ni objetos anidados.

---

## Paso 5 — Struct y extracción completa

```cpp
struct DatosAlumno {
    string nombre;
    int edad = 0;
    string matricula;
    bool valido = false;
};

DatosAlumno parsearAlumno(const string& cuerpo) {
    DatosAlumno d;
    d.nombre    = extraerValorJSON(cuerpo, "nombre");
    d.matricula = extraerValorJSON(cuerpo, "matricula");

    string edadStr = extraerValorJSON(cuerpo, "edad");
    try { d.edad = stoi(edadStr); } catch (...) { d.edad = 0; }

    d.valido = !d.nombre.empty() && !d.matricula.empty() && d.edad > 0;
    return d;
}
```

---

## Paso 6 — Handler de POST /api/alumnos

```cpp
if (ruta == "/api/alumnos" && metodo == "POST") {
    string cuerpo = extraerCuerpo(peticion);
    DatosAlumno d = parsearAlumno(cuerpo);

    if (!d.valido) {
        return construirRespuesta(400, "text/plain; charset=utf-8",
                                  "Datos inválidos");
    }

    cout << "\n--- POST recibido ---\n"
         << "Nombre:    " << d.nombre << "\n"
         << "Edad:      " << d.edad << "\n"
         << "Matrícula: " << d.matricula << "\n";

    string msg = "Bienvenido " + d.nombre + ", matrícula " + d.matricula;
    return construirRespuesta(200, "text/plain; charset=utf-8", msg);
}
```

---

## Paso 7 — Usar `leerPeticionCompleta`

En el bucle principal, reemplaza la lectura original:

```cpp
socket_t cliente = accept(servidor, (sockaddr*)&clienteDir, &tam);
string peticion = leerPeticionCompleta(cliente);
if (peticion.empty()) { CLOSE_SOCKET(cliente); continue; }

string metodo = extraerMetodo(peticion);
string ruta   = normalizarRuta(extraerRuta(peticion));
string cuerpo = extraerCuerpo(peticion);

cout << "[REQ] " << metodo << " " << ruta << endl;
```

---

## Paso 8 — Probar

1. Compila y arranca el servidor.
2. Abre `http://localhost:8080/`.
3. Rellena el formulario y envía.
4. En la consola del servidor:

```
[REQ] POST /api/alumnos
--- POST recibido ---
Nombre:    Ana
Edad:      20
Matrícula: A001
```

5. En el navegador ves `Bienvenido Ana, matrícula A001`.

---

## Paso 9 — Probar con curl

```bash
curl -i -X POST -H "Content-Type: application/json" \
  -d '{"nombre":"Luis","edad":22,"matricula":"A002"}' \
  http://localhost:8080/api/alumnos
```

Debe responder 200 con el mensaje personalizado.

---

## ✅ Checklist de la Actividad 28

- [ ] `leerPeticionCompleta` lee cabeceras + cuerpo según `Content-Length`.
- [ ] El struct `DatosAlumno` se llena con lo que envió el formulario.
- [ ] La consola muestra los tres campos.
- [ ] Los datos inválidos devuelven 400.
- [ ] El navegador muestra el mensaje personalizado.

## 🧠 Mini-reto
Si la matrícula ya existe en un `vector<DatosAlumno>` en memoria, responde 409 Conflict en lugar de 200.

---

