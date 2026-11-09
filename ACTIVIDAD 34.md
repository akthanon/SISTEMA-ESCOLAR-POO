# Actividad 34. Primeras clases del servidor

## RECOMENDACIÓN, IMPORTANTE
```text
Para este ejercicio se recomienda explicitamente utilizar IA para resolverlo debido al aumento de dificultad de este ejercicio en específico ya que incluye temas de POO y uso de bases de datos además de que muy probablemente se estarán cargando errores de codigos anteriores.
```
## Temas

- Responsabilidad única
    
- Encapsulación
    

## Ejercicio

Crear las clases:

```text
Servidor

Usuario

BaseDatosSQL
```

Cada una tendrá responsabilidades específicas.

Servidor

- aceptar conexiones
    
- procesar peticiones
    
- enviar respuestas
    

Usuario

- nombre
    
- usuario
    
- contraseña
    
- rol
    

BaseDatosSQL

- leer archivos
    
- guardar archivos
    
- buscar usuarios
    

Modificar el programa para utilizar estas clases en SQL

---
# TEORÍA
---

## 1. Principio de Responsabilidad Única (SRP)

- **Definición:** Cada clase debe tener **una sola razón para cambiar**. Es decir, debe tener **una única responsabilidad** bien definida.
- **Origen:** Es el primer principio SOLID ("S" de SRP).
- **Beneficio:** Código más fácil de mantener, entender y extender. Cuando algo cambia, solo se modifica una clase.
- **Síntoma de violación:** La frase "esta clase hace X **y** Y" → probablemente debería dividirse en dos.
- **En esta actividad:**
  - `Servidor` → responsabilidad: **comunicación HTTP**.
  - `Usuario` → responsabilidad: **representar a un usuario**.
  - `BaseDatosSQL` → responsabilidad: **persistencia en base de datos**.

Cada clase tiene un motivo distinto para cambiar. Si cambia el protocolo HTTP, solo tocas `Servidor`. Si cambia la estructura de un usuario, solo tocas `Usuario`. Si cambias de SQLite a MySQL, solo tocas `BaseDatosSQL`.

---

## 2. Encapsulación (repaso aplicado)

- **Definición:** Ocultar los detalles internos de una clase y exponer solo una interfaz controlada.
- **Mecanismo en C++:** Modificadores `private`, `protected`, `public`.
- **Regla:** Atributos `private`, métodos `public` (solo los que forman la interfaz).
- **En esta actividad:**
  - `Usuario`: sus campos (`nombre`, `usuario`, `contraseña`, `rol`) son `private`. Se accede vía getters/setters.
  - `BaseDatosSQL`: la conexión a la BD, el archivo `.db`, y los detalles de SQLite/MySQL son `private`. Se expone una API de alto nivel (`conectar`, `guardar`, `buscar`).
  - `Servidor`: el socket, el estado, el bucle principal son `private`. Solo `ejecutar()` y `detener()` son públicos.
- **Ventaja:** Puedes cambiar la implementación interna sin afectar al resto del programa.

---

## 3. Clase `Usuario`

### A. Responsabilidad
Representar a un usuario del sistema con sus datos básicos. **Nada más.** No sabe de archivos, ni de HTTP, ni de SQL.

### B. Atributos
- `string nombre;` → nombre completo ("Ana López").
- `string usuario;` → nombre de usuario único ("ana_lopez").
- `string contrasena;` → contraseña (nunca en texto plano en producción).
- `string rol;` → por ejemplo "alumno", "maestro", "admin".

### C. Métodos
- **Constructores:** vacío y con parámetros.
- **Getters y setters:** para cada atributo.
- **`bool esValido() const;`** → verifica que los campos mínimos estén presentes.
- **`string aJSON() const;`** → serialización para respuestas HTTP.
- **`static Usuario desdeJSON(const string& json);`** → deserialización (opcional).

### D. Consideraciones
- **Contraseña:** En proyectos reales, **nunca** se guarda en texto plano. Se guarda un **hash** (bcrypt, argon2, sha256 con salt). Para esta actividad académica se puede guardar en texto plano, pero menciónalo como deuda técnica.
- **Rol:** Puede ser un `enum` (`Rol::ALUMNO`, `Rol::MAESTRO`, `Rol::ADMIN`) o un `string`. El enum es más seguro en tiempo de compilación, pero más rígido si necesitas roles dinámicos.
- **Sin lógica de negocio aquí:** `Usuario` no valida contra la base de datos, no se autentica, no se registra. Solo representa datos.

**Fragmento suelto (esqueleto):**
```cpp
class Usuario {
private:
    string nombre;
    string usuario;
    string contrasena;
    string rol;

public:
    Usuario();
    Usuario(string n, string u, string c, string r);
    // getters y setters
};
```

---

## 4. Clase `BaseDatosSQL`

### A. Responsabilidad
Gestionar la **persistencia** en una base de datos SQL. Ocultar los detalles de conexión, consultas y transacciones al resto del programa.

### B. Atributos
- `sqlite3* db;` (si usas SQLite) o el handle de conexión que corresponda.
- `string rutaArchivo;` → ruta al archivo `.db` (SQLite) o cadena de conexión (MySQL/PostgreSQL).
- `bool conectada;` → estado de la conexión.

### C. Métodos
- **`bool conectar();`** → abre la base de datos. Devuelve `true` si tuvo éxito.
- **`void desconectar();`** → cierra la conexión limpiamente.
- **`bool guardarUsuario(const Usuario& u);`** → INSERT o UPDATE.
- **`vector<Usuario> listarUsuarios();`** → SELECT todos.
- **`Usuario buscarUsuario(const string& usuario);`** → SELECT por clave.
- **`bool eliminarUsuario(const string& usuario);`** → DELETE.
- **`bool existeUsuario(const string& usuario);`** → SELECT COUNT.

### D. Patrón de uso
- El `Servidor` crea **una** instancia de `BaseDatosSQL` al arrancar.
- Todos los handlers que necesiten persistencia llaman a métodos de esa instancia.
- Al terminar, `Servidor` cierra la conexión (o el destructor de `BaseDatosSQL` lo hace).

### E. Lo que NO debe hacer `BaseDatosSQL`
- No sabe de HTTP.
- No sabe de sockets.
- No conoce la clase `Servidor`.
- No construye respuestas.
- No valida reglas de negocio (por ejemplo, "no se puede registrar un usuario sin rol"). Eso es responsabilidad del `Servidor` o de una capa de servicios.

---

## 5. Clase `Servidor`

### A. Responsabilidad
Aceptar conexiones, procesar peticiones HTTP y enviar respuestas. Es la **capa de comunicación**.

### B. Atributos
- `int puerto;`
- `int socketServidor;`
- `string rutaPublic;`
- `BaseDatosSQL baseDatos;` → composición. El servidor **tiene una** base de datos.
- (Opcional) `bool activo;`

### C. Métodos
- **Públicos:**
  - `Servidor(int puerto, string rutaPublic, string rutaBD);`
  - `void ejecutar();` → bucle principal.
  - `void detener();`
- **Privados:**
  - `int aceptarConexion();`
  - `string leerPeticion(int cliente);`
  - `void enviarRespuesta(int cliente, const string& respuesta);`
  - `void procesarGET(...);`
  - `void procesarPOST(...);`
  - `void procesarPUT(...);`
  - `void procesarDELETE(...);`
  - `void manejarAPI(...);`
  - `void servirArchivo(...);`
  - `string obtenerMIME(...);`
  - `string construirRespuesta(...);`

### D. Composición
- `Servidor` **tiene un** `BaseDatosSQL`. No lo hereda, lo contiene.
- Esto es composición (relación "has-a"). El servidor delega la persistencia a la base de datos.
- **Ventaja:** Puedes cambiar `BaseDatosSQL` por otra implementación (por ejemplo, `BaseDatosEnMemoria`) sin tocar el servidor.

**Fragmento suelto (idea de composición):**
```cpp
class Servidor {
private:
    BaseDatosSQL baseDatos;
    // ...
};
```

---

## 6. Introducción a SQL

### A. ¿Qué es SQL?
- **SQL (Structured Query Language):** Lenguaje estándar para gestionar bases de datos relacionales.
- **Función:** Crear, leer, actualizar y eliminar datos (CRUD) en tablas.
- **Ventaja sobre archivos TXT:** Consultas rápidas, integridad referencial, transacciones, concurrencia, escalabilidad.
- **En esta actividad:** Se introduce para reemplazar el almacenamiento en TXT por una base de datos.

### B. Conceptos básicos
- **Tabla:** Equivalente a un arreglo o archivo. Tiene filas y columnas.
- **Fila (registro):** Un elemento (un usuario).
- **Columna (campo):** Un atributo del registro (`nombre`, `usuario`, etc.).
- **Clave primaria (PRIMARY KEY):** Columna que identifica unívocamente cada fila. En `Usuario`, sería `usuario`.
- **Consulta:** Instrucción SQL. Devuelve un conjunto de filas o modifica la base.
- **Transacción:** Conjunto de operaciones que se ejecutan como una sola. O todas o ninguna.
- **Índice:** Estructura auxiliar que acelera búsquedas.

### C. Comandos principales

| Comando | Propósito |
|---------|-----------|
| `CREATE TABLE` | Crear una tabla |
| `INSERT INTO` | Insertar filas |
| `SELECT` | Leer filas |
| `UPDATE` | Actualizar filas |
| `DELETE FROM` | Eliminar filas |
| `WHERE` | Filtrar filas (condición) |
| `ORDER BY` | Ordenar resultados |
| `DROP TABLE` | Eliminar una tabla completa |

**Fragmentos sueltos (ejemplos de sintaxis, no código completo):**
```sql
CREATE TABLE usuarios (
    usuario TEXT PRIMARY KEY,
    nombre TEXT NOT NULL,
    contrasena TEXT NOT NULL,
    rol TEXT NOT NULL
);

INSERT INTO usuarios (usuario, nombre, contrasena, rol)
VALUES ('ana_lopez', 'Ana López', 'hash', 'alumno');

SELECT nombre, rol FROM usuarios WHERE usuario = 'ana_lopez';

UPDATE usuarios SET rol = 'admin' WHERE usuario = 'ana_lopez';

DELETE FROM usuarios WHERE usuario = 'ana_lopez';
```

### D. Consultas preparadas (prepared statements)
- **Problema:** Concatenar strings para formar SQL abre la puerta a **inyección SQL**.
  - Ejemplo de ataque: si el usuario escribe `' OR '1'='1` como nombre de usuario, la consulta se rompe.
- **Solución:** Usar **consultas preparadas** con parámetros (`?`).
  - El motor SQL distingue entre el SQL y los datos. Los datos nunca se interpretan como código.
- **Sintaxis general (SQLite):** `prepare`, `bind`, `step`, `finalize`.
- **Regla de oro:** **Nunca concatenes strings para formar SQL.** Usa siempre consultas preparadas.

**Fragmento suelto (conceptual):**
```cpp
sqlite3_prepare_v2(db, "SELECT * FROM usuarios WHERE usuario = ?", -1, &stmt, nullptr);
sqlite3_bind_text(stmt, 1, usuario.c_str(), -1, SQLITE_STATIC);
sqlite3_step(stmt);
```

---

## 7. Opciones de bases de datos SQL para C++

### A. SQLite (recomendado para esta actividad)
- **Ventajas:**
  - Sin servidor: la base de datos es un archivo `.db`.
  - Librería ligera y portátil.
  - Ideal para proyectos pequeños y embebidos.
  - Sintaxis SQL estándar.
- **Integración:** Incluir `sqlite3.h` y enlazar con `-lsqlite3`.
- **Uso típico:** Una sola conexión, archivo local, sin configuración de red.
- **Perfecto para aprender SQL sin complejidad de servidores.**

### B. MySQL / MariaDB
- **Ventajas:** Robusto, usado en producción, escalable.
- **Desventajas:** Requiere servidor MySQL corriendo, configuración de usuarios, permisos, conexión por red.
- **Integración:** Librería `libmysqlclient` o `mysql-connector-c++`.
- **Uso:** Proyectos grandes.

### C. PostgreSQL
- **Ventajas:** Muy potente, estándar SQL completo, extensiones avanzadas.
- **Desventajas:** Similar a MySQL en complejidad de integración.
- **Integración:** `libpq` o `libpqxx` (wrapper C++).

### D. Recomendación
- **Para esta actividad:** SQLite. Cero configuración de servidor. La base de datos es un archivo.
- **Para el proyecto final:** SQLite sigue siendo suficiente para un sistema escolar.
- **Cambiar de SQLite a MySQL** más adelante solo requiere reescribir `BaseDatosSQL` internamente. El resto del código no cambia (por el encapsulamiento).

---

## 8. Tabla `usuarios` en SQL

### A. Estructura sugerida
- **`usuario`** TEXT PRIMARY KEY (clave primaria).
- **`nombre`** TEXT NOT NULL.
- **`contrasena`** TEXT NOT NULL.
- **`rol`** TEXT NOT NULL.

### B. Consideraciones
- **PRIMARY KEY:** Garantiza unicidad. Intentar insertar un usuario duplicado da error (que debes manejar como `409 Conflict`).
- **NOT NULL:** Evita campos vacíos.
- **CHECK:** Puedes añadir validaciones como `CHECK (rol IN ('alumno', 'maestro', 'admin'))`.
- **AUTOINCREMENT:** No necesario aquí, porque `usuario` ya es único.
- **Timestamps:** `creado_en TEXT DEFAULT CURRENT_TIMESTAMP` (opcional, útil para auditoría).
- **Contraseñas:** Nunca en texto plano. Almacena un hash. La lógica de hash vive fuera de `BaseDatosSQL` (por ejemplo, en una clase `Autenticacion`).

### C. Creación de la tabla
- Se hace **una vez** al inicializar la base de datos. Puedes:
  - Ejecutarlo al `conectar()` si la tabla no existe (`CREATE TABLE IF NOT EXISTS`).
  - O un script SQL separado ejecutado por el administrador.

---

## 9. Integración: cómo se comunican las tres clases

### A. Flujo de un POST `/api/usuarios`
1. El navegador envía POST con JSON: `{"nombre":"Ana","usuario":"ana_lopez","contrasena":"...","rol":"alumno"}`.
2. `Servidor` recibe, parsea y llama a `procesarPOST`.
3. `procesarPOST` parsea el JSON a un objeto `Usuario`.
4. `procesarPOST` valida los datos (no vacíos, rol correcto, etc.).
5. `procesarPOST` llama a `baseDatos.guardarUsuario(u)`.
6. `BaseDatosSQL` ejecuta un `INSERT` con consulta preparada.
7. `BaseDatosSQL` devuelve `true` o `false` (o lanza excepción).
8. `Servidor` construye la respuesta (`201 Created` o `409 Conflict`).
9. `Servidor` envía la respuesta al cliente.

### B. Flujo de un GET `/api/usuarios`
1. `Servidor` recibe GET.
2. Llama a `baseDatos.listarUsuarios()`.
3. `BaseDatosSQL` ejecuta `SELECT * FROM usuarios`.
4. `BaseDatosSQL` construye un `vector<Usuario>`.
5. `Servidor` serializa el vector a JSON.
6. `Servidor` envía la respuesta.

### C. Diagrama conceptual de dependencias
```
   main
     │
     ▼
  Servidor ────────► BaseDatosSQL ─────► SQLite (archivo .db)
     │                    │
     │                    │
     └─ usa ─► Usuario ◄──┘
```

- **Observa:**
  - `Servidor` depende de `BaseDatosSQL` (composición).
  - `BaseDatosSQL` maneja objetos `Usuario`.
  - `Usuario` es una clase de datos pura. No depende de nadie.
  - `main` solo instancia el servidor.

---

## 10. Ventajas del diseño con estas tres clases

### A. Cambio de motor de BD
- Si mañana decides pasar de SQLite a MySQL, **solo** reescribes `BaseDatosSQL`. `Servidor` y `Usuario` no cambian (gracias al encapsulamiento).
- **Requisito:** Que `BaseDatosSQL` exponga una **interfaz estable** (mismos métodos con la misma firma).
- **Mejora futura:** Definir una interfaz abstracta (`IBaseDatos`) y tener varias implementaciones. Se verá con polimorfismo.

### B. Test unitario
- Puedes crear un `BaseDatosSQL` de prueba (por ejemplo, en memoria con SQLite) y probar el servidor sin tocar el archivo real.
- `Usuario` se prueba con constructores y setters.

### C. Separación de responsabilidades
- Cambio en la estructura del usuario (añadir un campo `email`): solo tocas `Usuario` y su tabla SQL, no el `Servidor`.
- Cambio en el protocolo HTTP (por ejemplo, adoptar HTTP/2): solo tocas `Servidor`.

### D. Reutilización
- `BaseDatosSQL` podría usarse desde otra aplicación (una CLI, un script de migración).
- `Usuario` podría serializarse a JSON, XML, CSV sin tocar el servidor.

---

## 11. Estructura de archivos sugerida

### A. Todo en un archivo (simple, para esta actividad)
```
Proyecto/
├── servidor.cpp     ← incluye las tres clases + main
├── public/
└── data/
    └── sistema.db
```

### B. Separado en archivos (cuando crezca)
```
Proyecto/
├── main.cpp
├── servidor.h / servidor.cpp
├── usuario.h / usuario.cpp
├── baseDatosSQL.h / baseDatosSQL.cpp
├── public/
└── data/
    └── sistema.db
```

- **Para esta actividad:** Un solo archivo es suficiente. La separación es un paso posterior si el proyecto crece.
- **Regla práctica:** Cuando un archivo pase de ~800-1000 líneas, sepáralo.

---

## 12. Inicialización de SQLite

### A. Incluir la librería
- Header: `#include <sqlite3.h>`.
- Enlace: añadir `-lsqlite3` al compilar.
- En Windows con MinGW: verificar que `sqlite3.h` y `libsqlite3.a` estén en las rutas de include y lib.

### B. Abrir la conexión
- `sqlite3_open(ruta.c_str(), &db)` → abre o crea el archivo si no existe.
- Devuelve un código de error. `SQLITE_OK` significa éxito.
- **Verificar siempre:** si falla, loguear y salir o trabajar sin BD (según diseño).

### C. Ejecutar SQL
- **Consultas que no devuelven filas** (CREATE, INSERT, UPDATE, DELETE):
  - `sqlite3_exec(db, sql, nullptr, nullptr, &errmsg)`.
  - Devuelve `SQLITE_OK` o un error.
- **Consultas que devuelven filas** (SELECT):
  - `sqlite3_prepare_v2` → preparar la consulta.
  - `sqlite3_bind_*` → asociar parámetros.
  - `sqlite3_step` → ejecutar fila por fila (`SQLITE_ROW` mientras haya filas, `SQLITE_DONE` al terminar).
  - `sqlite3_column_*` → leer valores de la fila actual.
  - `sqlite3_finalize` → liberar recursos.

### D. Cerrar la conexión
- `sqlite3_close(db)` al terminar. Preferiblemente en el destructor de `BaseDatosSQL`.

### E. Errores comunes
- No verificar el resultado de `sqlite3_open` → trabajar con un puntero nulo.
- Olvidar `finalize` → fuga de recursos.
- No usar consultas preparadas → riesgo de inyección SQL.

**Fragmento suelto (esqueleto de conexión):**
```cpp
sqlite3* db = nullptr;
int rc = sqlite3_open("data/sistema.db", &db);
if (rc != SQLITE_OK) { /* error */ }
```

---

## 13. Consultas preparadas: patrón general

- **Preparar:** `sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr)`.
- **Bind:** `sqlite3_bind_text(stmt, posicion, valor.c_str(), -1, SQLITE_STATIC)` para strings, `sqlite3_bind_int` para enteros.
- **Ejecutar:** `sqlite3_step(stmt)`. En SELECT, se llama repetidamente hasta `SQLITE_DONE`.
- **Leer columnas:** `sqlite3_column_text(stmt, i)` para strings, `sqlite3_column_int(stmt, i)` para enteros.
- **Liberar:** `sqlite3_finalize(stmt)` siempre, incluso si hubo error.
- **Ventaja:** Los parámetros se escapan automáticamente. Imposible inyección SQL.

**Fragmento suelto (INSERT conceptual):**
```cpp
sqlite3_prepare_v2(db, "INSERT INTO usuarios VALUES (?, ?, ?, ?)", -1, &stmt, nullptr);
sqlite3_bind_text(stmt, 1, u.getUsuario().c_str(), -1, SQLITE_STATIC);
sqlite3_bind_text(stmt, 2, u.getNombre().c_str(), -1, SQLITE_STATIC);
sqlite3_bind_text(stmt, 3, u.getContrasena().c_str(), -1, SQLITE_STATIC);
sqlite3_bind_text(stmt, 4, u.getRol().c_str(), -1, SQLITE_STATIC);
int rc = sqlite3_step(stmt);
sqlite3_finalize(stmt);
```

- **`SQLITE_STATIC`** indica que el string no se modifica durante la ejecución. Si el string puede liberarse antes, usa `SQLITE_TRANSIENT` (SQLite copia el valor).
- **`sqlite3_step`** devuelve `SQLITE_DONE` si el INSERT tuvo éxito.

---

## 14. Manejo de errores en `BaseDatosSQL`

- **`conectar`:** Devuelve `bool`. Si falla, `Servidor` decide qué hacer (loguear, trabajar sin persistencia, abortar).
- **Operaciones CRUD:** Devuelven `bool` o un enum de estado. `Servidor` traduce a códigos HTTP:
  - Éxito → 200/201.
  - Duplicado → 409.
  - No encontrado → 404.
  - Error de BD → 500.
- **Excepciones:** Alternativa a los códigos de retorno. Útil si quieres diferenciar tipos de error.
- **Consistencia:** Elige un patrón y aplícalo en toda la clase. No mezcles.
- **Logs:** Loguea el código de error de SQLite y el mensaje para depurar. No expongas estos detalles al cliente.
- **Rollback:** Si usas transacciones (`BEGIN`, `COMMIT`, `ROLLBACK`), asegúrate de hacer rollback si algo falla a mitad.

---

## 15. Transacciones (introducción)

- **Definición:** Conjunto de operaciones SQL que se ejecutan como una unidad atómica.
- **Comandos:** `BEGIN TRANSACTION`, `COMMIT`, `ROLLBACK`.
- **Uso:** Cuando varias operaciones deben tener éxito o fallar juntas.
- **Ejemplo:** Transferir saldo entre dos cuentas. Si una actualización falla, no debe quedar la otra aplicada.
- **En esta actividad:** No es imprescindible. Cada operación CRUD es individual. Pero tenlo en cuenta si añades operaciones compuestas.

---

## 16. Seguridad: contraseñas

- **Nunca en texto plano.** Aunque en esta actividad académica se permita, en producción es inaceptable.
- **Hash:** Algoritmo de una sola dirección (bcrypt, argon2, PBKDF2, sha256 con salt).
- **Salt:** Valor aleatorio añadido a la contraseña antes de hashear. Evita ataques de diccionario.
- **Flujo:**
  1. Usuario se registra con contraseña `p`.
  2. Se genera un salt aleatorio.
  3. Se calcula `hash = H(p + salt)`.
  4. Se guardan `hash` y `salt` en la BD (no `p`).
  5. Al autenticar, se recalcula el hash con el salt guardado y se compara con el almacenado.
- **En C++:** Bibliotecas como `libsodium`, `bcrypt`, `argon2`.
- **Para esta actividad:** Guardar en texto plano es aceptable didácticamente, pero **menciónalo** como deuda técnica.

---

## 17. Buenas prácticas

- **SRP:** Una clase, una razón para cambiar. `Usuario` no toca archivos. `BaseDatosSQL` no toca sockets.
- **Encapsulación:** Atributos `private`, métodos de interfaz `public`.
- **Nombres en PascalCase** para clases (`Servidor`, `Usuario`, `BaseDatosSQL`).
- **Nombres en camelCase** para métodos (`guardarUsuario`, `buscarUsuario`).
- **Interfaz mínima:** Expón solo lo necesario. Cada método `public` es un contrato que no puedes romper.
- **Consultas preparadas siempre.** Nunca concatenes strings.
- **Verifica el resultado de cada operación de BD.** `sqlite3_open`, `sqlite3_step`, etc.
- **Libera recursos:** `sqlite3_finalize` en cada `prepare`, `sqlite3_close` al terminar.
- **Un solo objeto `BaseDatosSQL`** en el servidor. No abras la conexión en cada petición.
- **Cierra la conexión** en el destructor de `BaseDatosSQL`.
- **Errores consistentes:** `bool` de retorno o excepciones. No mezcles.
- **Loguea errores de BD en consola** para desarrollo. No los expongas al cliente.
- **Nunca guardes contraseñas en texto plano** en producción.
- **Considera separar `Usuario` en dos clases** si crece: `Usuario` (datos) y `Autenticacion` (lógica de login/hash).
- **Documenta el esquema de la tabla** en un comentario o en el README.
- **Versiona el esquema:** si cambia la tabla, considera migraciones.
- **Compila con `-Wall -Wextra`.** El compilador detecta problemas antes que tú.
- **Commits pequeños.** Cada clase funcional es un commit.

---

## 18. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`Servidor` accede directamente a la BD sin pasar por `BaseDatosSQL`** | Encapsula. `Servidor` solo llama métodos de `BaseDatosSQL`. |
| **`Usuario` lee/escribe archivos** | Rompe SRP. La persistencia es de `BaseDatosSQL`. |
| **`BaseDatosSQL` construye respuestas HTTP** | Rompe SRP. Eso es de `Servidor`. |
| **Concatenar strings para SQL** | Usa consultas preparadas. |
| **`sqlite3_open` falla y no se verifica** | Verifica siempre el código de retorno. |
| **Olvidar `sqlite3_finalize`** | Fuga de recursos y bloqueos. Libera siempre. |
| **Abrir y cerrar la BD en cada petición** | Abre una vez al arrancar, cierra al terminar. |
| **Contraseñas en texto plano** | Hash con salt. En producción, obligatorio. |
| **No verificar duplicados antes de INSERT** | SQLite lo hace por PRIMARY KEY. Captura el error. |
| **Mezclar código de red con SQL en un mismo método** | Divide en capas. `Servidor` llama a `BaseDatosSQL`. |
| **Métodos con demasiados parámetros** | Agrupa en objetos (`Usuario` en lugar de strings sueltos). |
| **`Usuario` con atributos públicos** | Encapsula con `private` + getters/setters. |
| **No loguear errores de BD** | Loguea para depurar. Pero no expongas al cliente. |
| **`SQLITE_STATIC` sobre un string temporal** | Usa `SQLITE_TRANSIENT` si el string puede liberarse antes del `step`. |
| **No cerrar la conexión al terminar** | El destructor de `BaseDatosSQL` debe cerrarla. |
| **Asumir que el INSERT funciona siempre** | Verifica `sqlite3_step` y maneja errores. |
| **Inyección SQL por concatenación** | Consultas preparadas. Es la única solución correcta. |

---

## 19. Verificación

1. **Compila con `-lsqlite3`.** Sin errores ni warnings.
2. **Ejecuta el servidor.** Debe crear o abrir `data/sistema.db` automáticamente.
3. **Registra un usuario** desde el navegador. Verifica que aparece en la base.
4. **Abre la BD con `sqlite3` CLI** o un visor gráfico (DB Browser for SQLite) para inspeccionar la tabla.
5. **Registra un usuario duplicado.** Debe responder `409 Conflict`.
6. **Lista los usuarios.** GET `/api/usuarios` debe devolver el JSON con todos.
7. **Reinicia el servidor.** Los usuarios deben persistir (están en el archivo `.db`).
8. **Elimina un usuario.** Verifica que ya no aparece.
9. **Actualiza un usuario.** Verifica el cambio.
10. **Prueba inyección SQL:** envía un nombre como `' OR '1'='1` y verifica que se guarda como texto literal, no como SQL.

---

## 20. Resumen de conceptos clave

- **SRP:** Cada clase, una responsabilidad. `Servidor` (HTTP), `Usuario` (datos), `BaseDatosSQL` (persistencia).
- **Encapsulación:** Atributos `private`, interfaz `public`.
- **Composición:** `Servidor` **tiene** un `BaseDatosSQL`. Relación "has-a".
- **SQL:** Lenguaje de consulta para bases de datos relacionales. CREATE, INSERT, SELECT, UPDATE, DELETE.
- **SQLite:** Base de datos ligera, sin servidor, ideal para proyectos pequeños y aprendizaje.
- **Consultas preparadas:** Evitan inyección SQL. Obligatorias en producción.
- **Tabla `usuarios`:** Columnas `usuario` (PK), `nombre`, `contrasena`, `rol`.
- **Ciclo de vida de la BD:** Abrir al arrancar, cerrar al terminar.
- **Interfaz de `BaseDatosSQL`:** `conectar`, `desconectar`, `guardarUsuario`, `buscarUsuario`, `listarUsuarios`, `eliminarUsuario`.
- **Contraseñas:** Nunca en texto plano en producción. Hash con salt.
- **Ventajas del diseño:** Cambio de motor SQL sin tocar el servidor, testeo individual, mantenibilidad.
- **Buenas prácticas:** SRP, encapsulación, consultas preparadas, verificar errores, liberar recursos, logs.

---
# PASO A PASO

---

# Actividad 34 — Primeras clases del servidor (SQLite)

## Objetivo
Crear tres clases con responsabilidades separadas:

- `Servidor`: aceptar conexiones, procesar HTTP.
- `Usuario`: representar datos del usuario.
- `BaseDatosSQL`: persistencia en SQLite.

Reemplazar el `vector<Alumno>` y el `usuarios.txt` por una **base de datos SQLite**.

## Antes de empezar

Instala SQLite:
- **Linux:** `sudo apt install libsqlite3-dev`
- **macOS:** `brew install sqlite3`
- **Windows (MinGW):** descarga `sqlite3.h` y `libsqlite3.a`.

Compila con:
```bash
g++ servidor.cpp -o servidor -lsqlite3 -std=c++17
```

---

## Paso 1 — Estructura de archivos

```
Proyecto/
├── servidor.cpp
├── public/
└── data/
    └── sistema.db   (se crea solo)
```

⚠️ **Crea la carpeta `data/` a mano.** El archivo `.db` lo crea SQLite.

---

## Paso 2 — Clase `Usuario`

Al inicio del archivo, antes de `Servidor`:

```cpp
class Usuario {
private:
    string nombre;
    string usuario;
    string contrasena;
    string rol;

public:
    Usuario() = default;

    Usuario(string n, string u, string c, string r)
        : nombre(n), usuario(u), contrasena(c), rol(r) {}

    // Getters
    string getNombre()     const { return nombre; }
    string getUsuario()    const { return usuario; }
    string getContrasena() const { return contrasena; }
    string getRol()        const { return rol; }

    // Setters
    void setNombre(string n)     { nombre = n; }
    void setUsuario(string u)    { usuario = u; }
    void setContrasena(string c) { contrasena = c; }
    void setRol(string r)        { rol = r; }

    bool esValido() const {
        return !nombre.empty() && !usuario.empty()
            && !contrasena.empty() && !rol.empty();
    }
};
```

📌 Observa:
- Atributos `private`.
- Getters/setters `public`.
- Sin lógica de archivos, sin lógica HTTP. Solo representa datos.

---

## Paso 3 — Clase `BaseDatosSQL`: cabecera

```cpp
#include <sqlite3.h>

class BaseDatosSQL {
private:
    sqlite3* db;
    string rutaArchivo;
    bool conectada;

public:
    BaseDatosSQL(const string& ruta) : rutaArchivo(ruta), db(nullptr), conectada(false) {}
    ~BaseDatosSQL();

    bool conectar();
    void desconectar();

    bool guardarUsuario(const Usuario& u);
    bool actualizarUsuario(const Usuario& u);
    bool eliminarUsuario(const string& usuario);
    Usuario buscarUsuario(const string& usuario);
    vector<Usuario> listarUsuarios();
    bool existeUsuario(const string& usuario);
};
```

- El **estado interno** (`sqlite3*`, `rutaArchivo`) es `private`.
- La **interfaz pública** son los métodos CRUD.

---

## Paso 4 — Constructor y destructor

```cpp
BaseDatosSQL::~BaseDatosSQL() {
    desconectar();
}

bool BaseDatosSQL::conectar() {
    int rc = sqlite3_open(rutaArchivo.c_str(), &db);
    if (rc != SQLITE_OK) {
        cerr << "[DB] Error al abrir: " << sqlite3_errmsg(db) << "\n";
        return false;
    }
    conectada = true;

    // Crear tabla si no existe
    const char* sql =
        "CREATE TABLE IF NOT EXISTS usuarios ("
        " usuario TEXT PRIMARY KEY,"
        " nombre TEXT NOT NULL,"
        " contrasena TEXT NOT NULL,"
        " rol TEXT NOT NULL)";

    char* err = nullptr;
    rc = sqlite3_exec(db, sql, nullptr, nullptr, &err);
    if (rc != SQLITE_OK) {
        cerr << "[DB] Error al crear tabla: " << err << "\n";
        sqlite3_free(err);
        return false;
    }

    return true;
}

void BaseDatosSQL::desconectar() {
    if (db) {
        sqlite3_close(db);
        db = nullptr;
        conectada = false;
    }
}
```

---

## Paso 5 — `guardarUsuario` con consulta preparada

⚠️ **Nunca** concatenes strings para SQL. Usa `prepare` + `bind`:

```cpp
bool BaseDatosSQL::guardarUsuario(const Usuario& u) {
    if (!conectada) return false;

    const char* sql =
        "INSERT INTO usuarios (usuario, nombre, contrasena, rol) VALUES (?, ?, ?, ?)";

    sqlite3_stmt* stmt = nullptr;
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) {
        cerr << "[DB] Error prepare: " << sqlite3_errmsg(db) << "\n";
        return false;
    }

    sqlite3_bind_text(stmt, 1, u.getUsuario().c_str(),    -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 2, u.getNombre().c_str(),     -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 3, u.getContrasena().c_str(), -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 4, u.getRol().c_str(),        -1, SQLITE_TRANSIENT);

    int rc = sqlite3_step(stmt);
    sqlite3_finalize(stmt);

    if (rc != SQLITE_DONE) {
        cerr << "[DB] Error al insertar: " << sqlite3_errmsg(db) << "\n";
        return false;
    }
    return true;
}
```

🎯 `SQLITE_TRANSIENT` copia el string al vuelo. Es lo que quieres si el `Usuario` se destruye después.

---

## Paso 6 — `buscarUsuario` y `listarUsuarios`

```cpp
Usuario BaseDatosSQL::buscarUsuario(const string& usuario) {
    Usuario vacio;
    if (!conectada) return vacio;

    const char* sql = "SELECT nombre, usuario, contrasena, rol FROM usuarios WHERE usuario = ?";
    sqlite3_stmt* stmt = nullptr;
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return vacio;

    sqlite3_bind_text(stmt, 1, usuario.c_str(), -1, SQLITE_TRANSIENT);

    Usuario resultado;
    if (sqlite3_step(stmt) == SQLITE_ROW) {
        resultado.setNombre   ((const char*)sqlite3_column_text(stmt, 0));
        resultado.setUsuario  ((const char*)sqlite3_column_text(stmt, 1));
        resultado.setContrasena((const char*)sqlite3_column_text(stmt, 2));
        resultado.setRol      ((const char*)sqlite3_column_text(stmt, 3));
    }

    sqlite3_finalize(stmt);
    return resultado;
}

vector<Usuario> BaseDatosSQL::listarUsuarios() {
    vector<Usuario> lista;
    if (!conectada) return lista;

    const char* sql = "SELECT nombre, usuario, contrasena, rol FROM usuarios";
    sqlite3_stmt* stmt = nullptr;
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return lista;

    while (sqlite3_step(stmt) == SQLITE_ROW) {
        Usuario u;
        u.setNombre   ((const char*)sqlite3_column_text(stmt, 0));
        u.setUsuario  ((const char*)sqlite3_column_text(stmt, 1));
        u.setContrasena((const char*)sqlite3_column_text(stmt, 2));
        u.setRol      ((const char*)sqlite3_column_text(stmt, 3));
        lista.push_back(u);
    }

    sqlite3_finalize(stmt);
    return lista;
}
```

---

## Paso 7 — Composición: `Servidor` tiene una `BaseDatosSQL`

```cpp
class Servidor {
private:
    int puerto;
    socket_t socketServidor;
    string carpetaPublic;
    BaseDatosSQL baseDatos;   // ← composición

    // ... resto de métodos privados

public:
    Servidor(int puerto, string carpetaPublic, string rutaBD)
        : puerto(puerto), socketServidor(INVALID_SOCKET),
          carpetaPublic(carpetaPublic), baseDatos(rutaBD) { }

    void ejecutar() {
        if (!baseDatos.conectar()) {
            cerr << "No se pudo conectar a la BD\n";
            return;
        }
        // ... crearSocket, bucle de accept
    }
};
```

📌 El `Servidor` **no sabe SQL**. Solo llama a `baseDatos.guardarUsuario(u)`, `baseDatos.listarUsuarios()`, etc.

---

## Paso 8 — Handlers usando la BD

El handler de POST `/api/usuarios` (o `/api/alumnos`, como lo tenías):

```cpp
if (ruta == "/api/usuarios" && metodo == "POST") {
    // extraer campos del JSON
    Usuario u(
        extraerValorJSON(cuerpo, "nombre"),
        extraerValorJSON(cuerpo, "usuario"),
        extraerValorJSON(cuerpo, "contrasena"),
        extraerValorJSON(cuerpo, "rol")
    );

    if (!u.esValido()) {
        return construirRespuesta(400, "application/json", "{\"error\":\"Datos inválidos\"}");
    }

    if (baseDatos.existeUsuario(u.getUsuario())) {
        return construirRespuesta(409, "application/json", "{\"error\":\"Usuario ya existe\"}");
    }

    if (!baseDatos.guardarUsuario(u)) {
        return construirRespuesta(500, "application/json", "{\"error\":\"Error de BD\"}");
    }

    return construirRespuesta(201, "application/json", "{\"ok\":true}");
}
```

GET `/api/usuarios`:

```cpp
if (ruta == "/api/usuarios" && metodo == "GET") {
    vector<Usuario> lista = baseDatos.listarUsuarios();

    string json = "[";
    for (size_t i = 0; i < lista.size(); i++) {
        if (i > 0) json += ",";
        json += "{\"nombre\":\""    + escaparJSON(lista[i].getNombre())    + "\",";
        json += "\"usuario\":\""    + escaparJSON(lista[i].getUsuario())   + "\",";
        json += "\"rol\":\""        + escaparJSON(lista[i].getRol())       + "\"}";
        // No devolvemos la contraseña nunca
    }
    json += "]";

    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

⚠️ **Nunca** devuelvas la contraseña ni el hash en la respuesta HTTP.

---

## Paso 9 — Verificar

1. Compila con `-lsqlite3 -std=c++17`.
2. Arranca el servidor. Debe crear `data/sistema.db` automáticamente.
3. Registra un usuario desde el navegador.
4. Inspecciona la BD con `sqlite3`:
   ```bash
   sqlite3 data/sistema.db "SELECT * FROM usuarios;"
   ```
5. Registra un usuario duplicado → debe devolver `409 Conflict`.
6. Reinicia el servidor → los usuarios siguen.
7. Prueba inyección SQL: registra un usuario con nombre `' OR '1'='1`. Debe guardarse como texto literal.

---

## ✅ Checklist de la Actividad 34

- [ ] Existen las clases `Usuario`, `BaseDatosSQL`, `Servidor`.
- [ ] `Usuario` solo tiene datos y getters/setters.
- [ ] `BaseDatosSQL` usa consultas preparadas en todas las operaciones.
- [ ] `Servidor` **no** ejecuta SQL directamente (delega en `BaseDatosSQL`).
- [ ] La tabla `usuarios` se crea automáticamente.
- [ ] La respuesta HTTP nunca expone la contraseña.
- [ ] `data/sistema.db` persiste tras reiniciar el servidor.

## 🧠 Mini-reto
Añade el método `bool actualizarUsuario(const Usuario& u)` con `UPDATE ... WHERE usuario = ?` y un endpoint PUT que lo use.