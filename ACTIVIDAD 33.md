# Actividad 33. Del código procedural a la POO

## Temas

- Refactorización
    
- Clases
    

## Ejercicio

Transformar las funciones creadas anteriormente en clases.

El objetivo es mantener exactamente el mismo comportamiento del servidor.

Comparar:

Antes

```cpp
procesarPOST();
```

Después

```cpp
servidor.procesarPOST();
```

Analizar las ventajas obtenidas mediante encapsulación y reutilización.

---
# TEORÍA

---

## 1. Del procedural a la POO: el cambio de paradigma

### A. Enfoque procedural (lo que ya tienes)
- El estado (vector de alumnos, socket servidor, puerto) vive en **variables globales** o se pasa por parámetros entre funciones.
- Las funciones operan sobre esos datos externos.
- El flujo lo controla `main` llamando funciones en secuencia.
- **Ejemplo conceptual:** `procesarPOST(peticion, alumnos, socket)`

### B. Enfoque orientado a objetos (lo que vas a construir)
- El estado vive **dentro de un objeto** (atributos privados).
- Las funciones que operan sobre ese estado se convierten en **métodos** del objeto.
- El objeto **sabe cómo operar sobre sí mismo**.
- **Ejemplo conceptual:** `servidor.procesarPOST(peticion)`

### C. Diferencia clave
- **Procedural:** los datos van a las funciones.
- **POO:** las funciones van a los datos (viven dentro del objeto que los contiene).

---

## 2. ¿Por qué hacer este cambio?

- **Encapsulación:** El estado interno queda protegido. Nadie fuera de la clase puede modificar los alumnos o el socket sin pasar por los métodos.
- **Cohesión:** Todo lo relacionado con el servidor vive en un mismo lugar. El código es más fácil de leer.
- **Reutilización:** Puedes instanciar múltiples servidores (en distintos puertos, por ejemplo) sin colisión de estado global.
- **Extensibilidad:** Añadir funcionalidad (HTTPS, autenticación, logs) es más limpio cuando hay un objeto que la agrupa.
- **Testabilidad:** Puedes crear un objeto servidor y probar sus métodos individualmente.
- **Evitar variables globales:** Todo el estado queda contenido. Es especialmente importante en proyectos grandes.

---

## 3. Identificar las clases candidatas

Antes de escribir una sola línea, identifica **qué conceptos** de tu dominio se pueden agrupar:

| Concepto | Clase candidata | Qué contiene |
|----------|-----------------|--------------|
| El servidor | `Servidor` | socket, puerto, estado, alumnos, métodos de arranque y bucle |
| Una petición HTTP | `PeticionHTTP` | método, ruta, cabeceras, cuerpo, métodos de parseo |
| Una respuesta HTTP | `RespuestaHTTP` | código, mime, cuerpo, método para serializar |
| Un alumno | `Alumno` | nombre, edad, matrícula, getters/setters |
| La persistencia | `AlmacenAlumnos` o `RepositorioAlumnos` | archivo, operaciones CRUD, lectura/escritura |
| La capa de red | `SocketServidor` (opcional) | socket, bind, listen, accept, recv, send |
| MIME Types | `MimeTypes` (opcional) | tabla de extensiones → tipo |

- **No necesitas todas.** Para esta actividad, `Servidor`, `PeticionHTTP`, `RespuestaHTTP` y `Alumno` son suficientes. Puedes empezar por `Servidor` y refactorizar más adelante.
- **Regla:** Una clase = un concepto con estado propio y operaciones asociadas.

---

## 4. Clase `Servidor`: la pieza central

### A. Atributos
- **Puerto:** `int puerto;`
- **Socket del servidor:** `int socketServidor;` (o `SOCKET` en Windows).
- **Arreglo de alumnos:** `vector<Alumno> alumnos;`
- **Ruta del archivo de datos:** `string rutaDatos;`
- **Ruta del directorio público:** `string rutaPublic;`
- **Estado:** `bool activo;` (opcional, útil para detener el servidor).

### B. Métodos
- **Ciclo de vida:** `iniciar()`, `detener()`, `ejecutar()` (bucle principal).
- **Red:** `aceptarConexion()`, `cerrarConexion(int cliente)`.
- **HTTP:** `leerPeticion(int cliente)`, `parsearPeticion(...)`, `enviarRespuesta(...)`.
- **Enrutamiento:** `enrutar(...)`, `manejarAPI(...)`.
- **Handlers:** `procesarGET(...)`, `procesarPOST(...)`, `procesarPUT(...)`, `procesarDELETE(...)`.
- **Estáticos:** `servirArchivo(...)`, `obtenerMIME(...)`.
- **Datos:** `cargarAlumnos()`, `guardarAlumnos()`, `buscarPorMatricula(...)`.
- **Utilidades:** `construirRespuesta(...)`, `escaparJSON(...)`.

### C. Por qué todos estos métodos van aquí
- Comparten el mismo estado (alumnos, rutas, socket).
- Forman parte del comportamiento del servidor.
- Al agruparlos, puedes leer `servidor.ejecutar()` y entender qué hace.

**Fragmento suelto (esqueleto):**
```cpp
class Servidor {
private:
    int puerto;
    int socketServidor;
    vector<Alumno> alumnos;
    string rutaDatos;
    string rutaPublic;

    void procesarGET(...);
    void procesarPOST(...);
    // ...

public:
    Servidor(int puerto, string rutaPublic, string rutaDatos);
    void ejecutar();
    void detener();
};
```

- **Miembros privados:** Los que son detalles internos (socket, alumnos, handlers).
- **Miembros públicos:** Los que el "mundo exterior" necesita (constructor, `ejecutar`, `detener`).
- **Regla:** Todo lo que no necesite usarse desde fuera debe ser `private`.

---

## 5. Clase `PeticionHTTP`: encapsular la petición

### A. Atributos
- `string metodo;`
- `string ruta;`
- `map<string, string> cabeceras;` (o un vector de pares)
- `string cuerpo;`

### B. Métodos
- `bool parsear(const string& crudo);` → Extrae método, ruta, cabeceras, cuerpo.
- `string obtenerCabecera(const string& nombre) const;`
- Getters para método, ruta y cuerpo.

### C. Ventaja
- En vez de pasar tres o cuatro strings sueltos por todas las funciones, pasas un solo objeto `PeticionHTTP`.
- Si mañana necesitas añadir más información (por ejemplo, cookies), la añades al struct/clase sin cambiar todas las firmas.

**Fragmento suelto (uso):**
```cpp
PeticionHTTP p;
p.parsear(crudo);
string ruta = p.ruta;
```

---

## 6. Clase `RespuestaHTTP`: encapsular la respuesta

### A. Atributos
- `int codigo;`
- `string mime;`
- `string cuerpo;`
- Opcional: `map<string, string> cabeceras;`

### B. Métodos
- `string serializar() const;` → Convierte el objeto en el string HTTP completo.
- `void setCodigo(int c);`
- `void setCuerpo(const string& c, const string& m);`

### C. Ventaja
- Los handlers solo construyen un objeto `RespuestaHTTP` con código, mime y cuerpo.
- La serialización (formato HTTP) está en un solo lugar.
- Si cambias el formato (por ejemplo, para añadir `Date` o `Server`), lo haces en `serializar()`.

**Fragmento suelto (uso):**
```cpp
RespuestaHTTP r(200, "application/json", "{}");
enviarRespuesta(cliente, r.serializar());
```

---

## 7. Clase `Alumno`: ya la tienes, reutilízala

- Atributos: `nombre`, `edad`, `matricula`.
- Métodos: getters, setters, `aJSON()`, `desdeJSON()`.
- Los métodos de persistencia (`cargar`, `guardar`) pueden vivir en una clase aparte (`Repositorio`) o en el `Servidor`. **Recomendación:** deja la lógica de archivo en el `Servidor` para esta actividad; separar en `Repositorio` es un paso posterior.

---

## 8. Aplicación a los métodos del servidor

### A. Antes (procedural)
```cpp
void procesarPOST(const string& cuerpo, vector<Alumno>& alumnos, int cliente) {
    // ...
}

procesarPOST(cuerpo, alumnos, cliente);
```

### B. Después (POO)
```cpp
void Servidor::procesarPOST(const PeticionHTTP& p, int cliente) {
    // usa this->alumnos, this->rutaDatos, etc.
}

servidor.procesarPOST(peticion, cliente);
```

- **Observa:** Ya no pasas `alumnos` ni `rutaDatos` como parámetros. Están dentro del objeto (`this->alumnos`).
- **Observa:** Pasas `peticion` como objeto en lugar de strings sueltos.
- **Ventaja:** Las firmas de los métodos son más cortas. La lógica está más cohesionada.

---

## 9. El `main` se simplifica

### A. Antes (procedural)
```cpp
int main() {
    inicializarSockets();
    int servidor = crearSocketServidor(PUERTO);
    cargarUsuarios(alumnos, RUTA_DATOS);

    while (true) {
        int cliente = accept(servidor, nullptr, nullptr);
        string crudo = leerPeticion(cliente);
        PeticionHTTP p = parsearPeticion(crudo);
        // enrutamiento
        // ...
    }
}
```

### B. Después (POO)
```cpp
int main() {
    Servidor servidor(8080, "public/", "data/usuarios.txt");
    servidor.ejecutar();
    return 0;
}
```

- **`main` se convierte en 3 líneas.**
- Toda la complejidad vive dentro de `Servidor`.
- **Ventaja:** Se entiende el programa de un vistazo.

---

## 10. Constructor de `Servidor`

- **Responsabilidad:** Inicializar el estado del objeto.
- **Qué recibe:**
  - Puerto.
  - Ruta del directorio público.
  - Ruta del archivo de datos.
- **Qué hace:**
  - Guarda los parámetros en atributos.
  - Carga los alumnos desde el archivo.
  - Inicializa sockets (WSAStartup en Windows).
  - (Opcional) Crea el socket y hace bind/listen en `iniciar()`.
- **Fragmento suelto (idea):**
  ```cpp
  Servidor::Servidor(int p, string rp, string rd)
      : puerto(p), rutaPublic(rp), rutaDatos(rd) {
      cargarAlumnos();
  }
  ```

- **Uso de lista de inicialización:** Inicializa atributos directamente (más eficiente).

---

## 11. Encapsulación: qué poner en `private` y qué en `public`

### A. `private` (detalles internos)
- Socket del servidor.
- Puerto.
- Vector de alumnos.
- Rutas de archivos.
- Handlers específicos (`procesarGET`, `procesarPOST`, etc.).
- Funciones auxiliares (`escaparJSON`, `obtenerMIME`, `leerArchivo`).

### B. `public` (interfaz del objeto)
- Constructor.
- `ejecutar()` (bucle principal).
- `detener()` (para cerrar limpiamente).
- (Opcional) `agregarAlumno(...)`, `listarAlumnos()` si quieres una interfaz de programación externa.

### C. Regla de oro
- **Todo lo que pueda ser `private`, que lo sea.**
- Exponer solo lo estrictamente necesario. Esto reduce el acoplamiento y facilita el mantenimiento.
- **Test de decisión:** "¿Necesita alguien de fuera llamar esto?" Si no, `private`.

---

## 12. Encapsulación en la práctica

- **Antes:** Cualquier función podía modificar `alumnos` directamente. Fácil de introducir bugs.
- **Después:** Solo los métodos de `Servidor` pueden modificar `alumnos`. Todas las validaciones están centralizadas.
- **Ejemplo:** Si quieres garantizar que no hay matrículas duplicadas, lo verificas en `Servidor::agregarAlumno` y listo. Nadie puede saltarse esa validación.
- **Ventaja adicional:** Puedes cambiar la estructura interna de `alumnos` (de vector a lista, a mapa) sin que el resto del código se entere.

---

## 13. Reutilización: instanciar múltiples servidores

- **Antes (procedural):** Solo puedes tener un servidor, porque el estado es global.
- **Después (POO):** Puedes instanciar dos servidores en puertos distintos:
  ```cpp
  Servidor s1(8080, "public/", "data/usuarios1.txt");
  Servidor s2(8081, "public2/", "data/usuarios2.txt");
  ```
- **Ventaja:** Útil para pruebas, para ejecutar entornos paralelos, o para test automatizado.
- **Observación:** Cada objeto tiene su propio estado. No hay colisiones.

---

## 14. Cohesión y acoplamiento

- **Cohesión:** Qué tan relacionadas están las cosas dentro de una clase. **Alta cohesión es buena.** Todo lo del servidor vive en `Servidor`.
- **Acoplamiento:** Qué tan dependientes son dos clases entre sí. **Bajo acoplamiento es bueno.** `Servidor` usa `PeticionHTTP` y `RespuestaHTTP`, pero no conoce sus detalles internos, solo su interfaz pública.
- **Regla práctica:** Si al modificar una clase tienes que tocar otras cinco, el acoplamiento es alto. Revisa las interfaces.

---

## 15. Pasar la petición como objeto

### A. Antes (múltiples parámetros)
```cpp
procesarGET(metodo, ruta, cabeceras, cuerpo, cliente, alumnos, rutaPublic);
```

### B. Después (un objeto)
```cpp
servidor.procesarGET(peticion, cliente);
```

- **Ventaja 1:** Menos parámetros. La firma es más legible.
- **Ventaja 2:** Si añades un campo a `PeticionHTTP` (por ejemplo, `ipCliente`), no cambias la firma de todas las funciones.
- **Ventaja 3:** Los métodos que reciben `PeticionHTTP` pueden acceder a cualquier campo del objeto sin más cambios.

---

## 16. Métodos `const` para solo lectura

- **Un método `const`** no modifica el objeto. Es seguro llamarlo sobre objetos constantes.
- **Útil para:**
  - Getters.
  - Serializadores (`aJSON()`, `serializar()`).
  - Cálculos que no cambian el estado (por ejemplo, `obtenerMIME`).
- **No se puede** llamar a un método no-`const` desde un método `const`.
- **Regla:** Marca como `const` todo método que no modifique el objeto. El compilador te lo agradecerá.

**Fragmento suelto:**
```cpp
string Alumno::aJSON() const { /* no modifica el objeto */ }
int Servidor::obtenerPuerto() const { return puerto; }
```

---

## 17. Manejo de errores en POO

- **Excepciones:** Puedes lanzarlas desde los métodos y capturarlas en `ejecutar` o `main`. Útil para errores de archivo, parseo, red.
- **Códigos de retorno:** Alternativa clásica. Métodos devuelven `bool` o un enum de estado.
- **Consistencia:** Elige un patrón y aplícalo en toda la clase. No mezcles excepciones y códigos de retorno sin criterio.
- **Recomendación para esta actividad:** Códigos de retorno o excepciones en `iniciar()`, y luego un `try-catch` en `ejecutar()` para evitar que el servidor muera ante un error puntual.

---

## 18. Refactorización paso a paso (segura)

1. **Haz una copia de seguridad o commitea antes de empezar.**
2. **Crea la clase `Servidor` con el constructor vacío.** Sin métodos todavía.
3. **Mueve las variables globales a atributos privados** (`puerto`, `socketServidor`, `alumnos`, etc.).
4. **Mueve `cargarUsuarios` y `guardarAlumnos`** como métodos privados de `Servidor`. Compila y prueba.
5. **Mueve las funciones HTTP** (`parsearPeticion`, `construirRespuesta`) como métodos. Compila y prueba.
6. **Mueve los handlers** (`procesarGET`, `procesarPOST`, etc.) como métodos. Compila y prueba.
7. **Mueve el bucle principal a `Servidor::ejecutar()`.** El `main` solo instancia y llama. Compila y prueba.
8. **Crea la clase `PeticionHTTP`** y sustituye los parámetros sueltos por el objeto. Compila y prueba.
9. **Crea la clase `RespuestaHTTP`** (opcional pero recomendada). Compila y prueba.
10. **Verifica el comportamiento completo** con los tests de la Actividad 32.

**Regla de oro:** Refactoriza **una cosa a la vez**. Compila y prueba entre pasos. No hagas todo y luego te preguntes qué rompió.

---

## 19. Estructura sugerida del archivo

Un solo `servidor.cpp` con las clases declaradas y definidas en orden:

1. **Includes.**
2. **`using namespace std;`**
3. **Constantes globales** (mínimas: puerto por defecto, rutas por defecto).
4. **Clase `Alumno`** (si no la tienes ya, o si la reescribes).
5. **Clase `PeticionHTTP`.**
6. **Clase `RespuestaHTTP`.**
7. **Clase `Servidor`** (declaración con métodos; definiciones fuera con `::`).
8. **`main`** (3 líneas).

- **Alternativa con separación en archivos:** `Alumno.h/cpp`, `PeticionHTTP.h/cpp`, `Servidor.h/cpp`, `main.cpp`. Es el paso siguiente cuando el proyecto crezca más. Para esta actividad, un solo archivo es suficiente.

---

## 20. Comparación antes/después

| Aspecto | Antes (procedural) | Después (POO) |
|---------|---------------------|----------------|
| Estado | Variables globales | Atributos privados del objeto |
| Funciones | Libres en el archivo | Métodos dentro de la clase |
| Llamada | `procesarPOST(...)` | `servidor.procesarPOST(...)` |
| Parámetros | Muchos, incluyendo estado | Pocos, solo lo necesario |
| `main` | Orquesta todo el flujo | 3 líneas: crear y ejecutar |
| Encapsulación | Nula | Alta (private/public) |
| Reutilización | Solo un servidor | Múltiples instancias |
| Testeo | Difícil (depende de globales) | Fácil (instanciar y probar métodos) |
| Extensibilidad | Hay que tocar muchas funciones | Añades métodos a la clase |

---

## 21. Buenas prácticas

- **Una clase, una responsabilidad.** `Servidor` no debería saber cómo se parsea un JSON interno de `Alumno`. Delega.
- **Atributos `private`**, métodos de interfaz `public`.
- **Nombres en PascalCase** para clases (`Servidor`, `PeticionHTTP`), **camelCase** para métodos (`procesarPOST`, `obtenerMIME`).
- **Constructores explícitos** que dejen el objeto en un estado válido.
- **Sin variables globales** una vez refactorizado (excepto constantes).
- **Métodos `const`** cuando no modifican el objeto.
- **Evita métodos de más de 40-50 líneas.** Si los tienes, divídelos en privados auxiliares.
- **Pasa objetos por `const&`** cuando sean grandes y no los modifiques.
- **Usa `this->`** con moderación; solo cuando haya ambigüedad con parámetros.
- **Separa declaración y definición:** Puedes declarar la clase completa con métodos y luego definir los métodos fuera con `::`. Mejora la lectura.
- **Documenta la clase** con un comentario al inicio describiendo su propósito.
- **Commits pequeños** durante la refactorización. Cada paso que funcione es un commit.
- **Prueba después de cada paso.** No acumules cambios sin compilar.
- **No añadas funcionalidad** durante la refactorización. Solo reorganiza.
- **No pasa nada si dejas la clase en un solo archivo.** Dividir en `.h/.cpp` es opcional y posterior.

---

## 22. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `public:` en la clase** | Todo es `private` por defecto. Añade `public:` donde corresponda. |
| **Olvidar el `;` al final de la clase** | Cierra con `};`, no solo `}`. |
| **Definir métodos fuera sin `::`** | Usa `tipoRetorno Clase::metodo(...) { }`. |
| **Seguir usando variables globales** | Muévelas a atributos del objeto. Es el punto de la refactorización. |
| **Pasar `this` a funciones libres** | Si necesitas `this`, es porque el método debería ser miembro de la clase. |
| **`main` sigue siendo gigante** | La lógica debe estar en `Servidor::ejecutar()`. `main` solo instancia. |
| **Cambiar comportamiento al refactorizar** | Prueba después de cada paso. No acumules cambios. |
| **Miembros sin inicializar** | Usa el constructor y lista de inicialización. |
| **Métodos demasiado largos** | Divide en privados auxiliares. |
| **Mezclar capas dentro de la clase** | La clase puede tener métodos privados para cada capa, pero no mezcles lógica de red con lógica de datos en un mismo método. |
| **Abuso de `friend`** | Casi nunca es necesario. Si lo usas, replantea el diseño. |
| **Copiar objetos grandes por valor** | Usa `const&` para parámetros. |
| **Errores de compilación en cascada tras mover código** | Refactoriza una función/método a la vez. Compila tras cada movimiento. |
| **No cerrar sockets/archivos en el destructor** | El destructor debe limpiar los recursos del objeto. |

---

## 23. El destructor de `Servidor`

- **Propósito:** Liberar recursos al destruir el objeto.
- **Qué hacer:**
  - Cerrar el socket del servidor si está abierto.
  - En Windows, llamar a `WSACleanup()` si corresponde.
  - Guardar los alumnos pendientes (opcional; ya guardas en cada operación).
- **Cuándo se llama:** Al salir del ámbito donde se creó el `Servidor`, típicamente al terminar `main`.
- **Fragmento suelto (idea):**
  ```cpp
  Servidor::~Servidor() {
      if (socketServidor > 0) close(socketServidor);
  }
  ```

- **Ventaja:** Garantiza limpieza incluso si el programa termina por una excepción no capturada.

---

## 24. Verificación del comportamiento

Después de la refactorización, debes verificar que **todo funciona igual**:

1. **Compila sin errores ni warnings** (`-Wall -Wextra`).
2. **Arranca el servidor.** Logs idénticos a antes.
3. **Prueba cada endpoint con `curl`** (mismos resultados).
4. **Registra, lista, edita y elimina un alumno.** El archivo `usuarios.txt` debe reflejar los cambios.
5. **Reinicia el servidor.** Los datos persisten.
6. **Carga una página web desde el navegador.** Todo funciona igual.
7. **Prueba con múltiples peticiones seguidas.** El servidor sigue vivo.
8. **Verifica DevTools → Network.** Los códigos de estado son los mismos.
9. **Ejecuta pruebas de casos borde:** matrícula duplicada, campos vacíos, recurso no encontrado.

Si todo coincide con el comportamiento anterior, la refactorización fue exitosa.

---

## 25. Ventajas obtenidas (para el análisis solicitado)

El ejercicio pide "analizar las ventajas obtenidas". Estas son las principales:

### A. Encapsulación
- El estado queda protegido. Solo los métodos del objeto lo modifican.
- Se garantiza la integridad de los datos (validaciones centralizadas).
- El código externo no depende de la estructura interna.

### B. Reutilización
- Múltiples instancias del mismo objeto con estados independientes.
- Métodos reutilizables desde distintas partes del propio objeto.

### C. Cohesión y legibilidad
- Todo lo del servidor está en una sola clase.
- `main` se reduce a 3 líneas y describe la intención del programa.
- Los nombres de los métodos describen la acción (auto-documentación).

### D. Mantenibilidad
- Cambiar la implementación interna no afecta al resto del código.
- Añadir funcionalidad es añadir métodos a la clase.

### E. Testabilidad
- Cada método se puede probar por separado instanciando un objeto.
- Se eliminan las dependencias globales.

### F. Escalabilidad
- El proyecto puede crecer añadiendo nuevas clases (`Autenticacion`, `BaseDatos`, `Logger`) sin reescribir todo.
- Facilita el trabajo en equipo: cada clase es una unidad de trabajo.

### G. Abstracción
- `servidor.ejecutar()` oculta toda la complejidad interna. Es la interfaz pública.
- Desde fuera, no sabes (ni necesitas saber) cómo funciona por dentro.

---

## 26. Resumen de conceptos clave

- **Refactorización:** Cambiar la estructura sin cambiar el comportamiento.
- **Del procedural a POO:** Mover estado y funciones a clases.
- **Clases candidatas:** `Servidor`, `PeticionHTTP`, `RespuestaHTTP`, `Alumno`.
- **Encapsulación:** `private` para estado interno, `public` para interfaz.
- **`main` mínimo:** Instancia y llama a `ejecutar()`.
- **Atributos privados:** Puerto, socket, alumnos, rutas.
- **Métodos públicos:** Constructor, `ejecutar`, `detener`.
- **Métodos privados:** Handlers, helpers, persistencia.
- **Constructor:** Inicializa el objeto. Carga datos. Prepara sockets.
- **Destructor:** Libera recursos.
- **Pasar objetos:** `PeticionHTTP` en lugar de múltiples strings.
- **`const`:** Métodos que no modifican el objeto.
- **Ventajas:** Encapsulación, cohesión, reutilización, mantenibilidad, testabilidad, escalabilidad, abstracción.
- **Verificación:** Comportamiento idéntico al anterior.
- **Refactorización segura:** Un cambio a la vez, compilar y probar entre pasos.

---
# PASO A PASO

---
# Actividad 33 — Del código procedural a la POO

## Objetivo
Transformar las funciones sueltas (actividad 32) en **métodos de clase**. El servidor pasa a ser un objeto `Servidor` con estado propio. El comportamiento debe seguir siendo idéntico.

## Antes
```cpp
procesarPOST(peticion, alumnos, cliente);
```

## Después
```cpp
servidor.procesarPOST(peticion, cliente);
```

La diferencia clave: **el estado (alumnos, puerto, rutas) vive dentro del objeto**, no como variables globales ni parámetros.

---

## Paso 1 — Variables globales → atributos

Antes de crear la clase, mira qué variables globales tienes:

```cpp
int PUERTO = 8080;
string CARPETA_PUBLICA = "public";
vector<Alumno> alumnos;
const string ARCHIVO_DATOS = "data/usuarios.txt";
```

Esas se van a convertir en atributos de `Servidor`. Deja solo las verdaderas constantes globales (por ejemplo, la tabla de MIME puede ser una función estática).

---

## Paso 2 — Esqueleto de la clase `Servidor`

```cpp
class Servidor {
private:
    int puerto;
    socket_t socketServidor;
    string carpetaPublic;
    string archivoDatos;
    vector<Alumno> alumnos;

    // Capa de red
    socket_t crearSocket();
    string   leerPeticion(socket_t cliente);
    void     enviarRespuesta(socket_t cliente, const string& respuesta);

    // Capa HTTP
    string extraerMetodo(const string& peticion) const;
    string extraerRuta(const string& peticion) const;
    string extraerCuerpo(const string& peticion) const;
    string normalizarRuta(string ruta) const;
    string obtenerMIME(const string& ruta) const;
    string construirRespuesta(int codigo, const string& mime, const string& cuerpo) const;

    // Capa de aplicación
    string manejarAPI(const string& ruta, const string& metodo, const string& cuerpo);
    string servirArchivo(const string& ruta) const;

    // Capa de datos
    void   cargarAlumnos();
    bool   guardarAlumnos() const;
    string leerArchivo(const string& ruta) const;

public:
    Servidor(int puerto, string carpetaPublic, string archivoDatos);
    ~Servidor();

    void ejecutar();
};
```

⚠️ Observa:
- Atributos en `private`.
- Solo `ejecutar()` y el constructor son `public`.
- Fíjate en el `const` al final de algunos métodos: no modifican el objeto.

---

## Paso 3 — Constructor

```cpp
Servidor::Servidor(int p, string cp, string ad)
    : puerto(p), socketServidor(INVALID_SOCKET),
      carpetaPublic(cp), archivoDatos(ad) {
    cargarAlumnos();
}

Servidor::~Servidor() {
    if (socketServidor != INVALID_SOCKET) {
        CLOSE_SOCKET(socketServidor);
    }
}
```

- La lista de inicialización inicializa los atributos **antes** del cuerpo.
- El destructor cierra el socket. El `WSACleanup` va en `main` todavía.

---

## Paso 4 — Mover `crearSocketServidor` a método

Copia el cuerpo de la función de la actividad 32 tal cual, pero sustituye las variables globales por atributos (`this->puerto`):

```cpp
socket_t Servidor::crearSocket() {
    socketServidor = socket(AF_INET, SOCK_STREAM, 0);
    if (socketServidor == INVALID_SOCKET) return INVALID_SOCKET;

    int opt = 1;
    setsockopt(socketServidor, SOL_SOCKET, SO_REUSEADDR,
               (const char*)&opt, sizeof(opt));

    sockaddr_in dir{};
    dir.sin_family = AF_INET;
    dir.sin_addr.s_addr = INADDR_ANY;
    dir.sin_port = htons(puerto);

    if (bind(socketServidor, (sockaddr*)&dir, sizeof(dir)) != 0) {
        return INVALID_SOCKET;
    }
    if (listen(socketServidor, 10) != 0) {
        return INVALID_SOCKET;
    }
    return socketServidor;
}
```

✅ Compila igual. El método no necesita recibir `puerto` como parámetro: ya es un atributo.

---

## Paso 5 — Mover el resto de funciones

Mueve una a una las funciones de la actividad 32 a métodos, siguiendo el mismo patrón:

- Sustituye variables globales por atributos (`this->alumnos`, `this->carpetaPublic`, `this->archivoDatos`).
- Añade `const` al final si el método no modifica el objeto.

Ejemplo con `servirArchivo`:

```cpp
string Servidor::servirArchivo(const string& ruta) const {
    if (ruta.find("..") != string::npos) {
        return construirRespuesta(400, "text/plain; charset=utf-8",
                                  "Ruta no permitida");
    }

    string rutaDisco = carpetaPublic + ruta;  // ← atributo, no global
    if (!fs::exists(rutaDisco)) {
        return construirRespuesta(404, "text/html", "<h1>404</h1>");
    }

    string contenido = leerArchivo(rutaDisco);
    return construirRespuesta(200, obtenerMIME(rutaDisco), contenido);
}
```

📌 `const` al final indica que el método no modifica el objeto. `servirArchivo` no cambia `alumnos` ni el socket → puede ser `const`.

---

## Paso 6 — El bucle principal como método

```cpp
void Servidor::ejecutar() {
    if (crearSocket() == INVALID_SOCKET) {
        cerr << "No se pudo iniciar el servidor\n";
        return;
    }
    cout << "Servidor en http://localhost:" << puerto << "\n";

    while (true) {
        sockaddr_in clienteDir{};
#ifdef _WIN32
        int tam = sizeof(clienteDir);
#else
        socklen_t tam = sizeof(clienteDir);
#endif
        socket_t cliente = accept(socketServidor, (sockaddr*)&clienteDir, &tam);
        if (cliente == INVALID_SOCKET) continue;

        string peticion = leerPeticion(cliente);
        if (peticion.empty()) { CLOSE_SOCKET(cliente); continue; }

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
}
```

---

## Paso 7 — Nuevo `main`

```cpp
int main() {
#ifdef _WIN32
    WSADATA wsa;
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) return 1;
#endif

    Servidor servidor(8080, "public/", "data/usuarios.txt");
    servidor.ejecutar();

#ifdef _WIN32
    WSACleanup();
#endif
    return 0;
}
```

🎯 De cientos de líneas a **3** (más el setup de Winsock).

---

## Paso 8 — Ajustar `manejarAPI`

`manejarAPI` accedía a `alumnos` (vector global). Ahora es atributo:

```cpp
string Servidor::manejarAPI(const string& ruta, const string& metodo, const string& cuerpo) {
    // GET /api/hola, /api/hora, etc.
    if (ruta == "/api/alumnos" && metodo == "GET") {
        return construirRespuesta(200, "application/json; charset=utf-8",
                                  alumnosAJSON(alumnos));  // ← atributo
    }
    // POST /api/alumnos → usa alumnos, guardarAlumnos, etc.
    // ...
}
```

📌 `manejarAPI` **no puede ser `const`** porque modifica `alumnos` en POST.

---

## Paso 9 — Verificar (de nuevo, comportamiento idéntico)

1. Compila con `-std=c++17 -Wall -Wextra`.
2. Corre el servidor. Los logs iniciales deben ser los mismos (`X alumnos cargados`).
3. Prueba todos los endpoints con `curl`.
4. Registra, lista, edita, elimina. `data/usuarios.txt` debe seguir funcionando.
5. Abre el navegador. Todo debe verse igual.

---

## Paso 10 — Comparación antes / después

| Aspecto | Antes (procedural) | Después (POO) |
|---------|---------------------|----------------|
| Estado | Variables globales | Atributos privados |
| Funciones | Libres | Métodos de `Servidor` |
| Llamada | `procesarPOST(peticion, alumnos, cliente)` | `servidor.procesarPOST(peticion, cliente)` |
| `main` | 40+ líneas | 3 líneas (más Winsock) |
| Encapsulación | Nula | Alta |
| Reutilización | Un servidor global | Múltiples instancias |

---

## ✅ Checklist de la Actividad 33

- [ ] Existe la clase `Servidor` con atributos en `private`.
- [ ] El constructor recibe puerto, carpeta pública y archivo de datos.
- [ ] El destructor cierra el socket.
- [ ] `main` solo instancia y llama a `ejecutar()`.
- [ ] Comportamiento idéntico al de la actividad 32.
- [ ] Compila sin warnings (`-Wall -Wextra`).

## 🧠 Mini-reto
Instancia **dos** servidores en puertos distintos (8080 y 8081) en el mismo `main`, apuntando a dos archivos de datos distintos. Verifica que cada uno tiene su propio estado.