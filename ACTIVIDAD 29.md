# Actividad 29. Persistencia con archivos

## Temas

- Archivos TXT
    
- Persistencia
    

## Ejercicio

Crear:

```text
data/
└── usuarios.txt
```

Cada nuevo registro recibido por el servidor deberá almacenarse automáticamente.

Definir un formato sencillo, por ejemplo:

```text
Juan|20|A001
María|21|A002
Luis|19|A003
```

Comprobar que los datos permanecen después de cerrar y volver a abrir el servidor.

---
# TEORÍA

---

## 1. ¿Por qué persistir datos?

- **Problema:** Todo lo que vive en memoria (variables, structs, arreglos) se pierde al cerrar el servidor.
- **Solución:** Escribir los datos en disco para que estén disponibles en la próxima ejecución.
- **Formatos posibles:**
  - **Texto plano (TXT):** legible por humanos, simple de parsear, ideal para aprender.
  - **CSV:** texto plano con comas, exportable a Excel.
  - **JSON:** estructurado, cómodo para anidar datos.
  - **Binario:** eficiente pero no legible.
  - **Base de datos (SQLite, PostgreSQL):** para proyectos grandes.
- **En esta actividad:** TXT con delimitador `|`, un registro por línea.

---

## 2. Estructura del proyecto

```
Proyecto/
├── servidor.cpp
├── public/
│   └── ...
└── data/
    └── usuarios.txt
```

- **`data/`:** Carpeta interna del backend. **No se sirve al navegador** (no está en `public/`).
- **`usuarios.txt`:** Archivo de texto con un alumno por línea.
- **Ubicación:** Relativa al directorio desde donde se ejecuta el servidor. Asegúrate de lanzarlo desde la raíz del proyecto para que `data/usuarios.txt` se resuelva correctamente.
- **Creación automática:** Si el archivo no existe, debe crearse al primer `ofstream` en modo escritura. Si la carpeta `data/` no existe, **no se crea sola**; debes crearla a mano o con `std::filesystem::create_directories`.

---

## 3. Formato del archivo

- **Un registro por línea:** Cada línea representa un alumno.
- **Campos separados por `|`:** Delimitador elegido porque:
  - No aparece en nombres normales.
  - Es fácil de parsear (un solo carácter).
  - Es legible a simple vista.
- **Orden de campos fijo:** `nombre|edad|matricula`.
- **Ejemplo:**
  ```
  Juan|20|A001
  María|21|A002
  Luis|19|A003
  ```
- **Fin de línea:** `\n` (una sola). No uses `\r\n` explícito; el sistema operativo lo maneja o lo lees igual.
- **Sin cabecera:** Aunque podrías añadir una línea de encabezado (`nombre|edad|matricula`), para simplicidad la omitimos. Si la añades, recuerda saltarla al leer.
- **Sin líneas vacías:** Cada línea debe tener los tres campos. Las líneas vacías o incompletas se ignoran al leer.
- **UTF-8:** Guarda los archivos en UTF-8 para que las tildes se vean correctamente.

---

## 4. Escritura: agregar registros

- **Modo append:** Abre el archivo con `ios::app` para **añadir al final sin borrar** lo existente.
- **Modo por defecto de `ofstream`:** Sobrescribe. **Cuidado:** si olvidas `ios::app`, borras todo.
- **Verificación de apertura:** Siempre verifica con `.is_open()` antes de escribir.
- **Escritura por líneas:** Un `<<` por campo, con `|` entre ellos, y `"\n"` al final.
- **Cierre:** Llama a `.close()` al terminar. Aunque el destructor lo hace solo, ser explícito evita olvidos.
- **Concurrencia:** Si el servidor atiende una sola petición a la vez (como en tu caso), no hay problema. Con múltiples hilos o procesos, necesitarías bloqueos de archivo.

**Fragmentos sueltos:**
```cpp
ofstream archivo("data/usuarios.txt", ios::app);
if (!archivo.is_open()) { /* error */ }
archivo << nombre << "|" << edad << "|" << matricula << "\n";
archivo.close();
```

- **`ios::app`:** Append. Si el archivo no existe, lo crea.
- **`"\n"` vs `endl`:** `endl` vacía el búfer y es más lento. `"\n"` es suficiente.
- **Validación previa:** Antes de escribir, valida los campos (no vacíos, edad numérica). No guardes basura.

---

## 5. Lectura: cargar registros al iniciar

- **Modo lectura:** `ifstream` en modo por defecto (o `ios::in`).
- **Verificación:** Si el archivo no existe, no es un error crítico. Simplemente significa "no hay datos aún". Maneja ese caso y arranca con un arreglo vacío.
- **Lectura línea por línea:** `getline(archivo, linea)`.
- **Parseo de cada línea:**
  - Usa `stringstream` + `getline(ss, campo, '|')` para separar por el delimitador.
  - O bien busca manualmente con `find('|')` y `substr`.
- **Conversión de tipos:** `edad` con `stoi`, envuelto en `try-catch`.
- **Validación:** Verifica que la línea tenga exactamente tres campos. Si no, ignórala y loguea una advertencia.
- **Almacenamiento:** Guarda cada registro en tu estructura de datos en memoria (arreglo, vector, struct).

**Fragmento suelto (con stringstream):**
```cpp
ifstream archivo("data/usuarios.txt");
string linea;
while (getline(archivo, linea)) {
    stringstream ss(linea);
    string campo;
    while (getline(ss, campo, '|')) {
        // procesar campo
    }
}
```

- **`getline(ss, campo, '|')`:** Lee hasta el `|` y lo descarta. El último campo se lee sin delimitador.
- **Verificar apertura antes:** `if (!archivo.is_open()) { /* no hay datos previos, OK */ }`
- **Cierre:** `archivo.close();` al terminar.

---

## 6. Momento de cargar y guardar

### A. Carga al iniciar
- **Cuándo:** Al arrancar el servidor, **antes** de empezar a aceptar conexiones.
- **Función típica:** `cargarUsuarios()` que llena el arreglo/vector en memoria.
- **Por qué:** Los datos están listos para consultarse en cualquier endpoint.
- **Dónde:** En `main()` antes del `while(true)` de `accept`.

### B. Guardado tras cada POST
- **Cuándo:** Inmediatamente después de validar los datos recibidos en el handler del POST.
- **Función típica:** `guardarUsuario(const DatosAlumno& d)` que hace append al archivo.
- **Por qué:** Garantiza que cada registro nuevo se persiste de inmediato, sin esperar al cierre del servidor.
- **Ventaja:** Si el servidor se cae, los datos ya guardados no se pierden.
- **Desventaja:** Abrir y cerrar el archivo por cada POST es lento si hay muchas peticiones. Para esta actividad está bien.

### C. Guardado al salir (opcional)
- **Cuándo:** Si prefieres escribir todo el archivo de una vez al cerrar el servidor.
- **Ventaja:** Menos accesos a disco.
- **Desventaja:** Si el servidor se cae sin cerrar correctamente, se pierden los datos no guardados.
- **Recomendación:** Para esta actividad, **guarda al recibir cada POST**. Es más seguro y didáctico.

### Comparación

| Estrategia | Ventaja | Desventaja |
|------------|---------|------------|
| Guardar en cada POST (append) | Persistencia inmediata, segura ante caídas | Muchas operaciones de apertura/cierre |
| Guardar todo al salir | Menos accesos a disco | Se pierden datos si el servidor crashea |
| Reescritura completa tras cada POST | Estado del archivo siempre consistente | Costoso con muchos registros |

---

## 7. Alternativa: reescritura completa

- **En vez de hacer append,** cada vez que cambie el arreglo (tras un POST), se reescribe el archivo completo desde el arreglo en memoria.
- **Ventaja:** El archivo siempre refleja exactamente el estado en memoria. No hay duplicados ni inconsistencias.
- **Desventaja:** Costoso si hay muchos registros. Además, si el servidor se cae a mitad de escritura, el archivo queda corrupto.
- **Mitigación:** Escribir primero en un archivo temporal (`usuarios.tmp`) y luego renombrarlo sobre el original (operación atómica en muchos sistemas).
- **Para esta actividad:** Con pocos registros, cualquiera de las dos estrategias funciona. **Append es más simple** y suficiente.

---

## 8. Validación y sanitización

- **Antes de escribir:**
  - `nombre` no vacío.
  - `edad` numérica y dentro de un rango razonable (por ejemplo, 0-120).
  - `matricula` no vacía y con formato esperado.
  - Ningún campo debe contener el delimitador `|` (rompería el formato). Si puede contenerlo, escápalo o cambia el delimitador.
- **Al leer:**
  - Verifica que cada línea tenga 3 campos.
  - Verifica que la edad sea convertible a entero.
  - Ignora líneas vacías o con formato inválido, logueando advertencias.
- **Concurrencia:** No es un problema en un servidor secuencial. Con múltiples hilos, protege el archivo con un mutex.

**Fragmento suelto (idea):**
```cpp
if (nombre.empty() || nombre.find('|') != string::npos) {
    // rechazar
}
```

---

## 9. Manejo de errores de archivo

- **Archivo no se puede abrir al escribir:**
  - Verifica permisos y que la carpeta `data/` exista.
  - Responde con `500 Internal Server Error` al cliente.
  - Loguea el error en consola.
- **Archivo no se puede abrir al leer:**
  - Si no existe, no es error: significa "primer arranque".
  - Si existe pero no se puede abrir (permisos), loguea y arranca vacío.
- **Disco lleno:** El `ofstream` puede fallar silenciosamente. Verifica con `.fail()` o `.bad()` después de escribir.
- **Archivo corrupto (líneas mal formadas):** Ignora las líneas inválidas y continúa. No abortes el arranque.
- **Carpeta `data/` no existe:**
  - Opción 1: Crearla a mano al preparar el proyecto (recomendado).
  - Opción 2: Crearla en código con `std::filesystem::create_directories("data")` al inicio.

---

## 10. Integración con el handler del POST

Pasos concretos al recibir un POST en `/api/alumnos`:

1. Leer el cuerpo completo (Actividad 28).
2. Parsear el JSON para extraer `nombre`, `edad`, `matricula`.
3. Validar los datos.
4. **Guardar el registro** con `guardarUsuario(datos)`.
   - Abre el archivo con `ios::app`.
   - Escribe `nombre|edad|matricula\n`.
   - Cierra.
5. Agregar el registro también a la estructura en memoria (para poder consultarlo sin releer el archivo).
6. Responder al cliente con mensaje de éxito y, opcionalmente, el ID asignado.
7. Si algo falla, responder con 400 o 500 y **no** escribir al archivo.

**Punto clave:** El orden importa. **Valida antes de escribir.** Nunca escribas datos inválidos al archivo.

---

## 11. Cargar al iniciar: integración en `main`

Orden recomendado al arrancar:

1. Inicializar sockets (Winsock en Windows).
2. Crear el socket del servidor.
3. `bind` y `listen`.
4. **Cargar los usuarios desde `data/usuarios.txt`** al arreglo/vector en memoria.
5. Loguear cuántos registros se cargaron.
6. Entrar al bucle `while(true)` de `accept`.

- **Importante:** La carga ocurre **antes** de aceptar conexiones. Así cualquier petición que llegue ya encuentra los datos disponibles.
- **Si el archivo no existe:** Simplemente arranca con cero usuarios, sin error.

---

## 12. Verificación de persistencia

Cómo comprobar que todo funciona:

1. Levanta el servidor.
2. Envía un formulario desde el navegador (`POST /api/alumnos` con nombre, edad, matrícula).
3. Verifica en consola del servidor el mensaje del handler.
4. **Abre el archivo `data/usuarios.txt`** con un editor de texto. Debe contener la nueva línea.
5. **Detén el servidor** (Ctrl+C).
6. **Vuelve a abrirlo.** Debe loguear "N usuarios cargados".
7. Consulta (si ya tienes endpoint `/api/alumnos` GET) o imprime en consola el contenido del arreglo cargado.
8. Envía otro POST. El nuevo registro se añade **después** del anterior (no se borra).
9. Repite el ciclo varias veces para confirmar que el archivo acumula correctamente.

**Señales de alarma:**
- El archivo se vacía tras cada POST → olvidaste `ios::app`.
- El archivo no crece → el handler del POST no llama a `guardarUsuario`.
- El archivo tiene caracteres raros → problema de encoding o de delimitador en los datos.
- Tras reiniciar, no hay datos → la función de carga no se ejecuta o falla silenciosamente.

---

## 13. Formato alternativo: CSV

- **Delimitador `,`** en lugar de `|`.
- **Ventaja:** Compatible con Excel, Google Sheets.
- **Desventaja:** Los nombres con comas rompen el formato. Requiere entrecomillar los campos que contengan comas.
- **Para esta actividad:** `|` es más seguro y sencillo. Reserva CSV para cuando necesites exportar a otros sistemas.

---

## 14. Buenas prácticas

- **Carpeta `data/` separada de `public/`.** Los datos no se sirven al navegador.
- **Nombre de archivo descriptivo:** `usuarios.txt`, `alumnos.txt`, `maestros.txt`.
- **Un archivo por tipo de entidad.** Facilita la carga y el mantenimiento.
- **Delimitador seguro:** `|` es buena elección. Evita `,` si los datos pueden contenerlo.
- **Un registro por línea.** Sin líneas vacías, sin cabeceras (o salta la cabecera al leer).
- **Escritura append:** `ios::app` para no borrar lo existente.
- **Cierre explícito:** `.close()` tras cada operación.
- **Verificación de apertura:** `if (!archivo.is_open())` antes de leer/escribir.
- **Validación antes de escribir:** No guardes datos inválidos.
- **UTF-8:** Codificación consistente entre frontend, backend y archivo.
- **Logs:** Imprime cuántos registros se cargaron y cada vez que se guarda uno nuevo.
- **Manejo de errores:** Si falla la escritura, responde 500 y no confirmes al cliente.
- **Funciones separadas:** `cargarUsuarios`, `guardarUsuario`, `parsearLinea`. Una responsabilidad por función.
- **Documentación:** Explica en el README el formato del archivo y qué campos contiene.
- **No mezclar datos de distintas entidades** en un mismo archivo. Usa uno por tipo.
- **Considera migrar a SQLite** cuando el proyecto crezca y necesites búsquedas rápidas y concurrencia.

---

## 15. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **El archivo se sobrescribe en cada POST** | Falta `ios::app` al abrir el `ofstream`. |
| **El archivo no se crea** | La carpeta `data/` no existe. Créala a mano o con `std::filesystem`. |
| **El archivo se crea pero está vacío** | No se llama a `guardarUsuario` desde el handler del POST. |
| **Los datos se corrompen con tildes** | Codificación inconsistente. Guarda y lee siempre en UTF-8. |
| **El delimitador `|` aparece en los datos** | Valida y rechaza campos que contengan `|`. |
| **Líneas vacías al final del archivo** | Normal al hacer append si el archivo terminaba sin `\n`. Ignóralas al leer. |
| **Última línea sin `\n`** | Algunos editores no añaden `\n` final. Al hacer append, tu nueva línea se pega a la anterior. Verifica o añade `\n` tras escribir. |
| **El servidor no carga los datos al reiniciar** | La función `cargarUsuarios` no se llama en `main`, o el archivo tiene formato inválido. |
| **`stoi` lanza excepción al leer** | Alguna línea tiene edad no numérica. Envuelve en `try-catch` e ignora esa línea. |
| **Se pierden datos al cerrar el servidor** | Estás guardando solo al salir y el servidor no cierra correctamente. Guarda tras cada POST. |
| **Se guardan registros duplicados** | Doble envío desde el frontend o doble llamada a `guardarUsuario`. Verifica el flujo. |
| **Concurrencia: el archivo se corrompe con múltiples clientes** | Servidor secuencial: no hay problema. Con hilos: usa mutex. |
| **El archivo crece indefinidamente** | Es lo normal con append. Rota o comprime cuando sea necesario. |
| **No encuentro el archivo** | Ruta relativa al directorio de ejecución. Usa la ruta absoluta o ejecuta desde la raíz del proyecto. |
| **Acentos rotos al abrir el archivo en otro editor** | El archivo está en UTF-8. Asegúrate de que el editor lo interprete así. |

---

## 16. Resumen de conceptos clave

- **Persistencia:** Guardar datos en disco para que sobrevivan al cierre del servidor.
- **Carpeta `data/`:** Separada de `public/`, no se sirve al navegador.
- **Formato TXT con `|`:** Un registro por línea, tres campos (`nombre|edad|matricula`).
- **Escritura append:** `ofstream` con `ios::app` para añadir sin borrar.
- **Lectura:** `ifstream` + `getline` + parseo con `stringstream` + `getline(ss, campo, '|')`.
- **Conversión:** `stoi` para edad, con `try-catch`.
- **Momento de guardar:** Tras cada POST, inmediatamente después de validar.
- **Momento de cargar:** Al arrancar el servidor, antes de aceptar conexiones.
- **Validación:** Antes de escribir, rechaza datos inválidos.
- **Manejo de errores:** Archivo no se abre → 500. Carpeta no existe → crearla. Formato roto → ignorar línea.
- **Verificación:** Enviar POST → mirar el archivo → reiniciar servidor → verificar que los datos siguen.
- **Buenas prácticas:** Un archivo por entidad, funciones separadas, cierre explícito, logs, UTF-8.
- **Futuro:** Cuando el proyecto crezca, considera migrar a SQLite o similar.

---
# PASO A PASO

---
# Actividad 29 — Persistencia con archivos

## Objetivo
Guardar cada alumno recibido en `data/usuarios.txt`. Los datos deben sobrevivir al reinicio del servidor.

## Formato

```
Juan|20|A001
María|21|A002
Luis|19|A003
```

Un registro por línea, campos separados por `|`.

---

## Paso 1 — Estructura de carpetas

```
Proyecto/
├── servidor.cpp
├── public/
└── data/
    └── usuarios.txt   (se crea solo al primer POST)
```

⚠️ **Crea la carpeta `data/` a mano** la primera vez.

---

## Paso 2 — Modelo en memoria

Al inicio del archivo, fuera de `main`:

```cpp
struct Alumno {
    string nombre;
    int edad;
    string matricula;
};

vector<Alumno> alumnos;
const string ARCHIVO_DATOS = "data/usuarios.txt";
```

Necesitas:

```cpp
#include <vector>
```

---

## Paso 3 — Función: cargar alumnos del archivo

```cpp
void cargarAlumnos() {
    alumnos.clear();

    ifstream archivo(ARCHIVO_DATOS);
    if (!archivo.is_open()) {
        cout << "[DATA] No hay archivo previo. Arrancando vacío.\n";
        return;
    }

    string linea;
    while (getline(archivo, linea)) {
        if (linea.empty()) continue;

        stringstream ss(linea);
        string nombre, edadStr, matricula;

        getline(ss, nombre,    '|');
        getline(ss, edadStr,   '|');
        getline(ss, matricula, '|');

        if (nombre.empty() || edadStr.empty() || matricula.empty()) {
            cerr << "[DATA] Línea inválida ignorada: " << linea << "\n";
            continue;
        }

        try {
            int edad = stoi(edadStr);
            alumnos.push_back({nombre, edad, matricula});
        } catch (...) {
            cerr << "[DATA] Edad no numérica en: " << linea << "\n";
        }
    }

    archivo.close();
    cout << "[DATA] " << alumnos.size() << " alumnos cargados.\n";
}
```

---

## Paso 4 — Función: guardar todos los alumnos

```cpp
bool guardarAlumnos() {
    ofstream archivo(ARCHIVO_DATOS);   // sin ios::app → trunca
    if (!archivo.is_open()) {
        cerr << "[DATA] No se pudo abrir " << ARCHIVO_DATOS << " para escritura\n";
        return false;
    }

    for (const auto& a : alumnos) {
        archivo << a.nombre << "|" << a.edad << "|" << a.matricula << "\n";
    }

    archivo.close();
    return true;
}
```

🎯 Sin `ios::app`, `ofstream` **trunca** el archivo. Es lo que queremos al reescribir.

---

## Paso 5 — Llamar a `cargarAlumnos()` en `main`

Antes del `while (true)`:

```cpp
cargarAlumnos();
cout << "Servidor escuchando en http://localhost:" << PUERTO << endl;
```

---

## Paso 6 — Guardar tras cada POST

Modifica el handler de `/api/alumnos`:

```cpp
if (ruta == "/api/alumnos" && metodo == "POST") {
    string cuerpo = extraerCuerpo(peticion);
    DatosAlumno d = parsearAlumno(cuerpo);

    if (!d.valido) {
        return construirRespuesta(400, "text/plain; charset=utf-8",
                                  "Datos inválidos");
    }

    // Verificar duplicado
    for (const auto& a : alumnos) {
        if (a.matricula == d.matricula) {
            return construirRespuesta(409, "text/plain; charset=utf-8",
                                      "Matrícula duplicada");
        }
    }

    // 1) Modificar memoria
    alumnos.push_back({d.nombre, d.edad, d.matricula});

    // 2) Persistir
    if (!guardarAlumnos()) {
        // Revertir en memoria para no quedar desincronizado
        alumnos.pop_back();
        return construirRespuesta(500, "text/plain; charset=utf-8",
                                  "Error al guardar");
    }

    cout << "[POST] Alumno guardado: " << d.nombre << " (" << d.matricula << ")\n";

    string msg = "Alumno " + d.nombre + " registrado.";
    return construirRespuesta(200, "text/plain; charset=utf-8", msg);
}
```

🎯 **Orden correcto:** valida → modifica memoria → persiste → responde. Si falla la persistencia, revierte memoria.

---

## Paso 7 — Verificar la persistencia

1. Levanta el servidor.
2. Envía un formulario.
3. Abre `data/usuarios.txt` con un editor: debe contener `Ana|20|A001`.
4. Detén el servidor (Ctrl+C) y vuelve a arrancarlo.
5. La consola debe decir `[DATA] 1 alumnos cargados.`
6. Envía otro formulario. El archivo ahora tiene dos líneas.

---

## Paso 8 — Errores comunes

| Error                              | Causa                                         |
|------------------------------------|-----------------------------------------------|
| El archivo se sobrescribe siempre  | Olvidaste el modelo en memoria (solo append) |
| El archivo no se crea              | No existe la carpeta `data/`                 |
| Tildes se ven mal                  | Encoding del archivo ≠ UTF-8                 |
| Se duplican líneas al reiniciar    | `cargarAlumnos()` no se llama en `main`     |

---

## ✅ Checklist de la Actividad 29

- [ ] Existe `data/usuarios.txt` con un alumno por línea.
- [ ] Los datos sobreviven al reinicio del servidor.
- [ ] Se rechazan matrículas duplicadas con 409.
- [ ] El log del servidor dice cuántos alumnos cargó al inicio.
- [ ] Se usa `ios::app` nunca; siempre se reescribe el archivo completo.

## 🧠 Mini-reto
Añade un campo "correo" al struct `Alumno`, al archivo y a la serialización. Actualiza también el parseo.

---
