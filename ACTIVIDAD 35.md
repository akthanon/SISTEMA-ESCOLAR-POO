# Actividad 35. Composición

## Temas

- Composición
    
- Relaciones entre objetos
    

## Ejercicio

Relacionar las clases.

Ejemplo:

```text
Servidor
│
├── BaseDatosSQL
│
└── Usuario
```

El servidor utilizará un objeto `BaseDatosSQL para acceder a los archivos.

Analizar por qué esta relación es composición y no herencia.

---
# TEORÍA

---

## 1. ¿Qué es una relación entre clases?

- **Definición:** Cuando dos o más clases colaboran para lograr un objetivo, existe una **relación** entre ellas.
- **Tipos principales:**
  - **Herencia ("es un" / is-a):** Una clase **es una** versión especializada de otra. Ej: `Alumno` **es una** `Persona`.
  - **Composición ("tiene un" / has-a):** Una clase **contiene** objetos de otra clase. Ej: `Servidor` **tiene una** `BaseDatosSQL`.
  - **Agregación:** Variante de composición donde el ciclo de vida del contenido no depende del contenedor. Ej: `Universidad` tiene `Profesor`es, pero los profesores existen aunque la universidad cierre.
  - **Asociación:** Relación más débil, un objeto usa a otro sin contenerlo. Ej: `Servidor` **usa** un `Logger` externo.
  - **Dependencia:** Relación aún más débil, un objeto usa a otro temporalmente (por ejemplo, como parámetro de un método).

- **En esta actividad:** La relación entre `Servidor` y `BaseDatosSQL` es de **composición**.

---

## 2. Composición vs. Herencia: la diferencia conceptual

### A. Herencia ("es un" / is-a)
- Una clase **extiende** a otra.
- La clase derivada **es** una versión de la base.
- Ejemplos clásicos: `Perro` es un `Animal`, `Alumno` es una `Persona`, `Círculo` es una `Figura`.
- **Sintaxis:** `class Derivada : public Base`.
- **Cuándo usarla:** Cuando existe una relación taxonómica clara y la clase derivada puede sustituir a la base sin sorpresas.
- **Riesgos:** Jerarquías rígidas, acoplamiento fuerte, difícil de cambiar.

### B. Composición ("tiene un" / has-a)
- Una clase **contiene** un objeto de otra clase como atributo.
- La clase contenedora **tiene** al contenido.
- Ejemplos clásicos: `Coche` tiene un `Motor`, `Servidor` tiene una `BaseDatosSQL`, `Alumno` tiene una `Materia`.
- **Sintaxis:** atributo del tipo contenido dentro de la clase.
- **Cuándo usarla:** Cuando una clase **necesita** a otra para funcionar, pero no **es** una versión de ella.
- **Ventajas:** Flexible, bajo acoplamiento, fácil de sustituir la parte contenida.

### C. La regla mnemotécnica
- **"Es un" → Herencia.**
- **"Tiene un" → Composición.**
- **En caso de duda:** Prefiere composición. Es más flexible y menos problemática.

---

## 3. Por qué `Servidor` y `BaseDatosSQL` es composición

Analicemos la relación:

### A. ¿Es `Servidor` una `BaseDatosSQL`?
- **No.** Un servidor no **es** una base de datos. Un servidor es un programa que acepta conexiones y sirve contenido. La base de datos es un componente que el servidor **usa** para persistir datos.
- **Si fuera herencia:** `class Servidor : public BaseDatosSQL` → el servidor heredaría métodos como `insertar`, `buscar`, `conectar`. Pero esos no son comportamientos del servidor. **No tiene sentido.**

### B. ¿Tiene `Servidor` una `BaseDatosSQL`?
- **Sí.** El servidor **necesita** una base de datos para guardar y consultar usuarios.
- **Si es composición:** `Servidor` tiene un atributo de tipo `BaseDatosSQL`. Le delega las operaciones de persistencia.
- **Tiene sentido:** El servidor usa la BD como un servicio interno.

### C. Conclusión
- La relación es **composición** porque:
  1. `Servidor` **no es** una `BaseDatosSQL`.
  2. `Servidor` **tiene** una `BaseDatosSQL`.
  3. La BD es un componente del servidor, no una especialización.
  4. Si mañana cambias la BD por otro motor, el servidor sigue siendo un servidor.

---

## 4. Por qué `Servidor` y `Usuario` NO es composición directa

- **`Servidor` no contiene `Usuario`s** como atributos fijos.
- Los usuarios se **obtienen** de la base de datos, se **procesan** en la petición y se **envían** al cliente.
- La relación entre `Servidor` y `Usuario` es de **uso temporal** (dependencia) o **asociación**, no composición.
- **Matiz:** Si el servidor tuviera un `vector<Usuario> usuarios;` en memoria, entonces **sí** sería composición (el servidor contiene usuarios).
- **En el diseño de la Actividad 34:** Los usuarios viven en la base de datos, no en el servidor. El servidor los pide cuando los necesita.

### A. Relación real
```
Servidor ──usa──► Usuario (crea, procesa, envía)
Servidor ──tiene──► BaseDatosSQL
BaseDatosSQL ──almacena──► Usuario
```

- **`Servidor` tiene una `BaseDatosSQL`** (composición).
- **`BaseDatosSQL` maneja `Usuario`s** (dependencia en métodos, no composición).
- **`Servidor` usa `Usuario`s** al procesar peticiones (dependencia temporal).

---

## 5. Tipos de relaciones en detalle

### A. Composición fuerte ("owns-a")
- El contenedor **posee** al contenido. El ciclo de vida del contenido depende del contenedor.
- Si el contenedor se destruye, el contenido también.
- **Ejemplo en C++:** Atributo por valor. `class Servidor { BaseDatosSQL db; };`
- **En el proyecto:** `Servidor` y `BaseDatosSQL`.

### B. Agregación ("has-a" débil)
- El contenedor **usa** al contenido, pero el contenido puede existir independientemente.
- El ciclo de vida no está atado.
- **Ejemplo en C++:** Puntero o referencia al contenido. `class Universidad { vector<Profesor*> profesores; };`
- **En el proyecto:** No aplica directamente.

### C. Asociación
- Relación estructural entre objetos, sin propiedad.
- Un objeto conoce a otro y puede invocar sus métodos.
- **Ejemplo en C++:** Un atributo puntero o referencia, sin ownership claro.
- **En el proyecto:** Podría aplicarse si el servidor guardara una referencia a un `Logger` externo.

### D. Dependencia
- Relación más débil: un objeto **usa** a otro momentáneamente.
- **Ejemplo en C++:** Un parámetro de método. `void procesar(const Usuario& u);`
- **En el proyecto:** `Servidor` depende de `Usuario` al procesar peticiones.

---

## 6. Diagramas UML: cómo representar relaciones

### A. Herencia
- Línea con **flecha triangular hueca** apuntando a la base.
- Se lee: "Derivada **es una** Base".

### B. Composición fuerte
- Línea con **rombo relleno** en el lado del contenedor.
- Se lee: "Contenedor **tiene un** contenido".
- El rombo está relleno porque el contenedor **posee** al contenido.

### C. Agregación
- Línea con **rombo hueco** en el lado del contenedor.
- Se lee: "Contenedor **tiene** contenidos, pero no los posee".

### D. Asociación
- Línea simple entre dos clases, con o sin flecha.
- Se lee: "A **conoce** a B".

### E. Dependencia
- Línea **punteada** con flecha.
- Se lee: "A **usa** a B temporalmente".

### F. Aplicación al proyecto
```
┌────────────┐        ◆─────────┐
│  Servidor  │──────────────────│ BaseDatosSQL │
└────────────┘                  └──────────────┘
       │                                 │
       │ usa                             │ maneja
       ▼                                 ▼
┌────────────┐                  ┌──────────────┐
│  Usuario   │◄─────────────────│   (SQL)      │
└────────────┘                  └──────────────┘
```

- **Rombo relleno (◆)** entre `Servidor` y `BaseDatosSQL` → composición.
- **Línea discontinua** entre `Servidor` y `Usuario` → dependencia.
- **BaseDatosSQL maneja Usuario** → dependencia en métodos.

---

## 7. Implementación de la composición en C++

### A. Tres formas de componer

**1. Por valor (composición fuerte):**
- El objeto contenido es un atributo.
- Se construye y destruye con el contenedor.
- Más simple, sin gestión manual de memoria.
- **Fragmento suelto:** `class Servidor { BaseDatosSQL baseDatos; };`

**2. Por puntero (composición con ownership):**
- El contenedor tiene un puntero y es responsable de destruirlo.
- Útil si el contenido es polimórfico.
- Requiere `new`/`delete` o `unique_ptr`.
- **Fragmento suelto:** `class Servidor { unique_ptr<BaseDatosSQL> db; };`

**3. Por referencia (asociación):**
- El contenedor guarda una referencia a un objeto externo.
- No es dueño del objeto.
- **Fragmento suelto:** `class Servidor { BaseDatosSQL& db; };`

### B. Cuál usar
- **Por valor:** Cuando no necesitas polimorfismo y el contenido se crea con el contenedor. **Es el caso más simple y recomendado para esta actividad.**
- **Por `unique_ptr`:** Cuando necesitas polimorfismo (por ejemplo, `IBaseDatos` con SQLite y MySQL).
- **Por referencia:** Cuando el objeto se crea fuera y se pasa al contenedor. Menos común.

### C. Inicialización en el constructor
- Los atributos por valor se inicializan en la **lista de inicialización** del constructor.
- **Fragmento suelto:**
  ```cpp
  Servidor::Servidor(int puerto, string rutaBD)
      : puerto(puerto), baseDatos(rutaBD) { }
  ```

- **Orden de inicialización:** El orden real es el orden de **declaración** en la clase, no el orden en la lista.
- **Advertencia:** Si un atributo depende de otro, declara primero el que se necesita.

---

## 8. Delegación: el patrón que hace funcionar la composición

- **Definición:** El contenedor **delega** las operaciones al objeto contenido.
- **Ejemplo:** `Servidor::procesarPOST` no implementa SQL directamente. Llama a `baseDatos.guardarUsuario(u)` y deja que la BD haga el trabajo.
- **Ventajas:**
  - `Servidor` no necesita saber SQL.
  - `BaseDatosSQL` no necesita saber HTTP.
  - Cada clase hace lo suyo.
- **Regla:** Si el contenedor empieza a hacer cosas que son responsabilidad del contenido, la composición está mal diseñada.

**Fragmento suelto (delegación):**
```cpp
void Servidor::procesarPOST(const Usuario& u) {
    if (baseDatos.guardarUsuario(u)) {
        // responder 201
    } else {
        // responder 500
    }
}
```

- **Observa:** `Servidor` no ejecuta SQL. Solo llama al método de `BaseDatosSQL`.

---

## 9. Composición y ciclo de vida

### A. Orden de construcción
1. Se construyen los atributos en el orden de declaración.
2. Se ejecuta el cuerpo del constructor.

### B. Orden de destrucción
- Inverso al de construcción.
- Los atributos se destruyen automáticamente cuando el objeto contenedor se destruye.

### C. Ejemplo en el proyecto
- Al crear un `Servidor`, se crea automáticamente su `BaseDatosSQL`.
- Al destruir el `Servidor`, se destruye su `BaseDatosSQL` (y su destructor cierra la conexión a la BD).

### D. Implicación
- **No necesitas crear ni destruir la BD manualmente.** El ciclo de vida está atado.
- **Ventaja:** Menos errores. No olvidas cerrar la BD.

---

## 10. Composición y polimorfismo

### A. El problema
- Si en el futuro quieres soportar múltiples motores de BD (SQLite, MySQL, PostgreSQL), necesitas poder cambiar la implementación sin tocar `Servidor`.
- **Solución:** Definir una interfaz abstracta `IBaseDatos` con métodos virtuales puros.

### B. Patrón
- `IBaseDatos` es la interfaz (métodos virtuales puros).
- `BaseDatosSQLite`, `BaseDatosMySQL`, `BaseDatosPostgres` son implementaciones concretas.
- `Servidor` tiene un `unique_ptr<IBaseDatos>` y trabaja contra la interfaz.
- En tiempo de ejecución, se inyecta la implementación concreta.

### C. Fragmento suelto (esqueleto)
```cpp
class IBaseDatos {
public:
    virtual bool guardarUsuario(const Usuario&) = 0;
    virtual vector<Usuario> listarUsuarios() = 0;
    virtual ~IBaseDatos() {}
};

class Servidor {
    unique_ptr<IBaseDatos> baseDatos;
public:
    Servidor(unique_ptr<IBaseDatos> bd) : baseDatos(std::move(bd)) {}
};
```

- **Ventaja:** `Servidor` no depende de un motor concreto.
- **Desventaja:** Más complejidad. Solo vale la pena si realmente vas a cambiar de motor.
- **Para esta actividad:** Composición directa con `BaseDatosSQL` es suficiente. La interfaz abstracta es un paso posterior.

---

## 11. Inyección de dependencias (introducción)

- **Definición:** En lugar de que `Servidor` cree su `BaseDatosSQL` internamente, se la pasan desde fuera (por el constructor).
- **Ventaja 1:** Testabilidad. Puedes inyectar una BD falsa (mock) en pruebas.
- **Ventaja 2:** Flexibilidad. Puedes cambiar la implementación sin tocar `Servidor`.
- **Ventaja 3:** Claridad. Las dependencias del `Servidor` son explícitas.

### A. Sin inyección (acoplamiento fuerte)
- `Servidor` crea internamente `BaseDatosSQL db("data/sistema.db");`
- `Servidor` decide la ruta y el tipo de BD.
- Difícil de testear.

### B. Con inyección (acoplamiento débil)
- `Servidor` recibe la BD por el constructor.
- `main` crea la BD y la pasa.
- Fácil de testear y de cambiar.

**Fragmento suelto (con inyección):**
```cpp
Servidor::Servidor(int puerto, BaseDatosSQL& bd)
    : puerto(puerto), baseDatos(bd) { }
```

- **Referencia vs. valor:** Con inyección, se suele usar referencia o `unique_ptr`.
- **Para esta actividad:** Puedes hacerlo sin inyección (más simple) o con inyección (más limpio). Decide según el objetivo.

---

## 12. Composición vs. herencia: cuándo usar cada una

| Criterio | Herencia | Composición |
|----------|----------|-------------|
| Relación | "Es un" | "Tiene un" |
| Acoplamiento | Fuerte | Débil |
| Flexibilidad | Baja (jerarquía rígida) | Alta (fácil de cambiar) |
| Reutilización | Por especialización | Por delegación |
| Cambio en tiempo de ejecución | No (el tipo es fijo) | Sí (puedes cambiar el objeto contenido) |
| Complejidad del diseño | Aumenta con la profundidad | Se mantiene plana |
| Cuándo usar | Relación taxonómica clara | Relación de uso |
| Ejemplo en el proyecto | `Alumno` es `Persona` | `Servidor` tiene `BaseDatosSQL` |

### A. Regla de oro
- **Prefiere composición sobre herencia.** Es más flexible, menos acoplada y más fácil de modificar.
- **Usa herencia solo cuando:**
  - La relación "es un" es **real y evidente**.
  - La clase derivada puede sustituir a la base (principio de sustitución de Liskov).
  - No hay otra forma más simple de modelar la relación.
- **Usa composición cuando:**
  - La relación es "tiene un" o "usa un".
  - Necesitas cambiar la implementación en tiempo de ejecución.
  - Quieres evitar jerarquías profundas.

---

## 13. Ventajas de usar composición en este proyecto

### A. Separación de responsabilidades
- `Servidor` gestiona HTTP.
- `BaseDatosSQL` gestiona la persistencia.
- `Usuario` representa datos.
- Cada clase puede cambiar sin afectar a las demás.

### B. Reemplazabilidad
- Puedes cambiar SQLite por MySQL sin tocar `Servidor`.
- Solo reescribes `BaseDatosSQL` (o inyectas otra implementación).

### C. Testabilidad
- Puedes crear un `BaseDatosSQL` de prueba (por ejemplo, en memoria) y probar el servidor sin tocar la BD real.

### D. Reutilización
- `BaseDatosSQL` puede usarse en otras aplicaciones (una CLI, un script de migración).

### E. Ciclo de vida claro
- La BD se crea con el servidor y se destruye con él. No hay fugas de conexión.

### F. Sin "clases gigantes"
- Si `Servidor` heredara de `BaseDatosSQL`, tendría métodos de HTTP **y** de SQL mezclados. Un desastre.
- Con composición, cada clase hace lo suyo.

---

## 14. Errores comunes al relacionar clases

### A. Usar herencia para "reutilizar código"
- **Error:** "Como quiero usar los métodos de `BaseDatosSQL` en `Servidor`, hago que herede de ella."
- **Problema:** Rompe el principio "es un". `Servidor` no es una BD.
- **Solución:** Composición. `Servidor` **tiene** una `BaseDatosSQL`.

### B. Herencia múltiple innecesaria
- **Error:** Hacer que una clase herede de varias para "tener todo".
- **Problema:** Complejidad, ambigüedad, colisiones de nombres.
- **Solución:** Composición. Contiene los objetos que necesita.

### C. Acoplamiento fuerte entre clases
- **Error:** `Servidor` accede directamente a los atributos internos de `BaseDatosSQL`.
- **Problema:** Cambiar `BaseDatosSQL` obliga a tocar `Servidor`.
- **Solución:** Encapsulación. `Servidor` solo usa la interfaz pública de `BaseDatosSQL`.

### D. Responsabilidades mezcladas
- **Error:** `Servidor` hace SQL directamente, o `BaseDatosSQL` construye respuestas HTTP.
- **Problema:** Rompe SRP.
- **Solución:** Cada clase hace lo suyo. `Servidor` delega en `BaseDatosSQL`.

### E. Ciclo de vida mal gestionado
- **Error:** `Servidor` abre y cierra la BD en cada petición.
- **Problema:** Rendimiento, riesgo de bloqueos.
- **Solución:** La BD se crea con el servidor y vive durante toda su vida.

### F. Dependencias circulares
- **Error:** `Servidor` tiene `BaseDatosSQL` y `BaseDatosSQL` tiene `Servidor`.
- **Problema:** Imposible de compilar, diseño confuso.
- **Solución:** Replantear. `BaseDatosSQL` no necesita saber de `Servidor`.

---

## 15. Buenas prácticas

- **Prefiere composición sobre herencia.** Es la regla #1 del diseño OO moderno.
- **"Es un" → herencia. "Tiene un" → composición.** En caso de duda, composición.
- **Una clase, una responsabilidad.** Si una clase hace dos cosas, divídela.
- **Delega en lugar de duplicar.** El contenedor no reimplementa lo que el contenido ya hace.
- **Encapsula.** El contenedor accede al contenido solo por su interfaz pública.
- **Composición por valor** cuando no necesitas polimorfismo ni inyección.
- **Composición por `unique_ptr`** cuando necesitas polimorfismo.
- **Inyección de dependencias** para testeo y flexibilidad.
- **Ciclo de vida claro.** El contenedor es responsable del contenido.
- **Sin dependencias circulares.** Si A tiene B, B no debe tener A.
- **Documenta las relaciones** en un diagrama (UML o ASCII) para no perderlas de vista.
- **Nombra las clases según su responsabilidad.** `BaseDatosSQL` dice qué hace. `Manager` no dice nada.
- **Revisa el diseño periódicamente.** Cuando el proyecto crece, las relaciones se enredan si no se cuidan.
- **No abuses de la herencia.** Jerarquías profundas son difíciles de mantener.
- **Usa interfaces abstractas** cuando necesites varias implementaciones intercambiables.

---

## 16. Errores comunes y soluciones (tabla)

| Error | Solución |
|-------|----------|
| **Hacer que `Servidor` herede de `BaseDatosSQL`** | Composición. `Servidor` tiene una BD, no es una. |
| **`BaseDatosSQL` conoce a `Servidor`** | Rompe SRP. La BD no debe saber quién la usa. |
| **`Servidor` accede a atributos internos de la BD** | Encapsula. Solo usa métodos públicos. |
| **Pasar el objeto por valor cuando es grande** | Pasa por `const&` o `&`. |
| **Dependencias circulares** | Replantea el diseño. Si A tiene B, B no tiene A. |
| **Abrir/cerrar la BD en cada petición** | Una conexión por instancia, creada al inicio. |
| **`Servidor` con `vector<Usuario>` interno en lugar de la BD** | Decide: ¿los usuarios viven en memoria o en la BD? Si es BD, no dupliques. |
| **Herencia para "reutilizar código"** | Composición. La herencia es para "es un". |
| **Clases que saben demasiado** | Reduce responsabilidades. Divide en clases más pequeñas. |
| **UML desactualizado** | Actualízalo al cambiar las clases. |
| **`Usuario` con métodos de persistencia** | La persistencia es de `BaseDatosSQL`. `Usuario` solo representa datos. |
| **`BaseDatosSQL` con métodos HTTP** | El HTTP es de `Servidor`. La BD solo persiste. |
| **`main` con lógica de creación de BD** | `main` solo instancia y llama. La creación es del constructor. |

---

## 17. Refactorización sugerida

Si vienes de la Actividad 34 y quieres formalizar la composición:

1. **Verifica que `BaseDatosSQL` tiene un constructor** que recibe la ruta del archivo de BD.
2. **Añade un atributo `BaseDatosSQL baseDatos;`** a `Servidor` (o `unique_ptr` si prefieres).
3. **Inicializa el atributo en la lista de inicialización** del constructor de `Servidor`.
4. **Sustituye las llamadas directas a SQLite** en `Servidor` por llamadas a `baseDatos.metodo(...)`.
5. **Verifica que el destructor de `BaseDatosSQL`** cierra la conexión.
6. **Prueba** que todo funciona igual.
7. **Dibuja el diagrama UML** de las clases y sus relaciones.

**Regla:** No añadas funcionalidad nueva. Solo formaliza la composición.

---

## 18. Composición en el contexto del proyecto

Visualicemos el diseño final:

```
┌──────────────────────────┐
│         Servidor         │
│──────────────────────────│
│ - puerto: int            │
│ - socketServidor: int    │
│ - rutaPublic: string     │
│ - baseDatos: BaseDatosSQL│  ← composición
│──────────────────────────│
│ + ejecutar()             │
│ + detener()              │
│ - procesarGET()          │
│ - procesarPOST()         │
│ - enviarRespuesta()      │
└────────────┬─────────────┘
             │ ◆
             │ tiene
             ▼
┌──────────────────────────┐
│      BaseDatosSQL        │
│──────────────────────────│
│ - db: sqlite3*           │
│ - rutaArchivo: string    │
│──────────────────────────│
│ + conectar()             │
│ + desconectar()          │
│ + guardarUsuario(Usuario)│
│ + buscarUsuario(string)  │
│ + listarUsuarios()       │
└──────────────────────────┘
             │ usa
             ▼
┌──────────────────────────┐
│        Usuario           │
│──────────────────────────│
│ - nombre: string         │
│ - usuario: string        │
│ - contrasena: string     │
│ - rol: string            │
│──────────────────────────│
│ + getters/setters        │
└──────────────────────────┘
```

- **Rombo relleno (◆)** entre `Servidor` y `BaseDatosSQL` → composición fuerte.
- **Flecha simple** entre `BaseDatosSQL` y `Usuario` → dependencia (los métodos de BD trabajan con `Usuario`, pero no lo contienen).
- `Servidor` **no** contiene `Usuario`s como atributo. Solo los usa temporalmente en sus métodos.

---

## 19. Verificación del diseño

Pregúntate al revisar las relaciones:

1. **¿`Servidor` es una `BaseDatosSQL`?** No → no es herencia.
2. **¿`Servidor` tiene una `BaseDatosSQL`?** Sí → composición.
3. **¿Si el `Servidor` se destruye, la `BaseDatosSQL` también?** Sí → composición fuerte.
4. **¿`BaseDatosSQL` sabe de `Servidor`?** No → sin dependencia circular.
5. **¿`Usuario` depende de otras clases?** No → es una clase de datos pura.
6. **¿`Servidor` accede a la BD directamente (SQLite)?** No → solo por la interfaz de `BaseDatosSQL`.
7. **¿`BaseDatosSQL` construye respuestas HTTP?** No → eso es de `Servidor`.
8. **¿El diseño respeta SRP?** Sí → cada clase tiene una responsabilidad clara.

Si todas las respuestas son correctas, la composición está bien aplicada.

---

## 20. Resumen de conceptos clave

- **Relaciones entre clases:** Herencia ("es un"), composición ("tiene un"), agregación ("tiene débil"), asociación ("conoce"), dependencia ("usa").
- **Composición vs. herencia:**
  - "Es un" → herencia.
  - "Tiene un" → composición.
  - **En caso de duda, composición.**
- **Composición en el proyecto:** `Servidor` tiene `BaseDatosSQL`. La BD es un componente del servidor, no una especialización.
- **`Servidor` y `Usuario`:** Relación de uso temporal, no composición.
- **`BaseDatosSQL` y `Usuario`:** Dependencia (los métodos trabajan con `Usuario`).
- **Ciclo de vida:** El contenedor crea y destruye el contenido.
- **Delegación:** El contenedor delega en el contenido en lugar de reimplementar.
- **Implementación:** Por valor (simple), por `unique_ptr` (polimorfismo) o por referencia (asociación).
- **UML:** Rombo relleno para composición, flecha triangular para herencia, flecha punteada para dependencia.
- **Ventajas:** SRP, flexibilidad, testabilidad, reutilización, bajo acoplamiento.
- **Inyección de dependencias:** Alternativa más flexible a crear el contenido internamente.
- **Errores comunes:** Usar herencia para reutilizar, acoplamiento fuerte, dependencias circulares, responsabilidades mezcladas.

---
# PASO A PASO

---
# Actividad 35 — Composición

## Objetivo
Formalizar la relación entre `Servidor` y `BaseDatosSQL` como **composición**. Analizar por qué es composición y no herencia. Verificar que el diseño respeta SRP.

## Regla de oro
- **"Es un"** → herencia.
- **"Tiene un"** → composición.
- **En caso de duda:** composición.

---

## Paso 1 — Verificar la composición existente

En la actividad 34 ya introdujimos la composición:

```cpp
class Servidor {
private:
    BaseDatosSQL baseDatos;   // atributo por valor
    // ...
};
```

Esto es **composición fuerte**: el ciclo de vida de `baseDatos` está atado al del `Servidor`. Si el `Servidor` se destruye, `baseDatos` también (y su destructor cierra la conexión SQLite).

✅ Ya está hecho. Ahora vamos a **verificar y documentar** por qué es la relación correcta.

---

## Paso 2 — Justificar la decisión

Responde por escrito estas preguntas (agrega un comentario en el código o un archivo `docs/diseño.md`):

### A. ¿Es `Servidor` una `BaseDatosSQL`?
**No.** Un servidor **no es** una base de datos. Un servidor es un programa que acepta conexiones HTTP; la BD es un componente que usa para persistir.

Si hiciéramos `class Servidor : public BaseDatosSQL`, el servidor heredaría `conectar()`, `guardarUsuario()`, etc. Pero esos **no son comportamientos del servidor**. No tiene sentido.

### B. ¿Tiene `Servidor` una `BaseDatosSQL`?
**Sí.** El servidor **necesita** una base de datos para guardar usuarios. Es un componente suyo.

### C. ¿Si `Servidor` se destruye, `BaseDatosSQL` también?
**Sí** (composición por valor). El `~BaseDatosSQL` cierra la conexión automáticamente.

### D. ¿Sabe `BaseDatosSQL` que existe `Servidor`?
**No.** La BD no necesita saber quién la usa. Esa es la marca de un buen diseño.

---

## Paso 3 — Diagrama de relaciones

Dibuja (o escribe en un comentario) el diagrama:

```
┌──────────────────────────┐
│         Servidor         │
│──────────────────────────│
│ - puerto                 │
│ - socketServidor         │
│ - carpetaPublic          │
│ - baseDatos: BaseDatosSQL│ ◆ (composición)
│──────────────────────────│
│ + ejecutar()             │
│ - procesarGET()          │
│ - procesarPOST()         │
└────────────┬─────────────┘
             │
             │ tiene
             ▼
┌──────────────────────────┐
│       BaseDatosSQL       │
│──────────────────────────│
│ - db: sqlite3*           │
│ - rutaArchivo            │
│──────────────────────────│
│ + conectar()             │
│ + guardarUsuario()       │
│ + listarUsuarios()       │
└────────────┬─────────────┘
             │
             │ usa (dependencia)
             ▼
┌──────────────────────────┐
│        Usuario           │
│──────────────────────────│
│ - nombre                 │
│ - usuario                │
│ - contrasena             │
│ - rol                    │
│──────────────────────────│
│ + getters/setters        │
└──────────────────────────┘
```

- **Rombo relleno (◆)** entre `Servidor` y `BaseDatosSQL` → composición.
- **Flecha simple** entre `BaseDatosSQL` y `Usuario` → dependencia (los métodos de BD reciben `Usuario` como parámetro, no lo contienen).
- `Servidor` y `Usuario`: dependencia temporal (uso en handlers), **no** composición.

---

## Paso 4 — Delegación: la clave de la composición

Verifica que `Servidor` **delega** y no reimplementa:

```cpp
// ✅ BIEN: Servidor delega
void Servidor::procesarPOST(const Usuario& u) {
    if (baseDatos.guardarUsuario(u)) {
        // 201
    } else {
        // 500
    }
}

// ❌ MAL: Servidor hace SQL directamente
void Servidor::procesarPOST(const Usuario& u) {
    const char* sql = "INSERT INTO usuarios ...";  // ← ¡NO!
    sqlite3_exec(...);
}
```

Regla: si el `Servidor` empieza a hacer cosas que son responsabilidad de `BaseDatosSQL`, la composición está mal aplicada.

---

## Paso 5 — Encapsulación: qué expone cada clase

Revisa tus clases y verifica:

| Clase | `private` | `public` |
|-------|-----------|----------|
| `Servidor` | `puerto`, `socketServidor`, `baseDatos`, handlers | Constructor, `ejecutar()` |
| `BaseDatosSQL` | `db`, `rutaArchivo`, `conectada` | `conectar`, `desconectar`, CRUD |
| `Usuario` | `nombre`, `usuario`, `contrasena`, `rol` | Getters, setters |

⚠️ Si algo interno está `public`, es una fuga de encapsulación. Cámbialo a `private`.

---

## Paso 6 — Sin dependencias circulares

Verifica:
- `BaseDatosSQL` **no** incluye `Servidor`.
- `Usuario` **no** incluye `Servidor` ni `BaseDatosSQL`.
- Solo `Servidor` conoce a ambos.

```cpp
// ✅ BIEN
class Servidor {
    BaseDatosSQL baseDatos;
};

// ❌ MAL
class BaseDatosSQL {
    Servidor* servidor;  // ← dependencia circular
};
```

---

## Paso 7 — Inyección de dependencias (opcional pero recomendado)

En la actividad 34, el `Servidor` **crea** su `BaseDatosSQL` internamente. Una mejora es **inyectarla** desde fuera:

### Antes (acoplamiento fuerte)
```cpp
Servidor::Servidor(int puerto, string rutaBD)
    : puerto(puerto), baseDatos(rutaBD) { }
```

### Después (inyección)
```cpp
class Servidor {
private:
    BaseDatosSQL& baseDatos;   // referencia externa

public:
    Servidor(int puerto, BaseDatosSQL& bd)
        : puerto(puerto), baseDatos(bd) { }
};
```

En `main`:
```cpp
BaseDatosSQL bd("data/sistema.db");
bd.conectar();

Servidor servidor(8080, bd);
servidor.ejecutar();
```

**Ventajas:**
- Puedes inyectar una BD de prueba (por ejemplo, en memoria).
- `main` decide qué BD usa el servidor.
- El `Servidor` no depende de `sqlite3`.

**Desventaja:**
- Gestión del ciclo de vida fuera del servidor.

Para esta actividad es **opcional**. La composición por valor que ya tienes funciona perfectamente.

---

## Paso 8 — Verificar el diseño

Responde estas preguntas. Si todas son correctas, tu diseño está bien:

1. ¿`Servidor` es una `BaseDatosSQL`? → **No** (no hay herencia).
2. ¿`Servidor` tiene una `BaseDatosSQL`? → **Sí** (composición).
3. ¿Si `Servidor` se destruye, `BaseDatosSQL` también? → **Sí**.
4. ¿`BaseDatosSQL` sabe de `Servidor`? → **No**.
5. ¿`Usuario` sabe de otras clases? → **No** (clase de datos pura).
6. ¿`Servidor` accede directamente a `sqlite3*`? → **No**.
7. ¿`BaseDatosSQL` construye respuestas HTTP? → **No**.
8. ¿Respeta SRP? → **Sí** (una razón de cambio por clase).

---

## Paso 9 — Documentar el diseño en el README

Crea (o actualiza) `README.md` del proyecto con esta sección:

```markdown
## Arquitectura

El servidor sigue el principio de Responsabilidad Única (SRP).

- **Servidor**: protocolo HTTP + sockets. NO hace SQL.
- **BaseDatosSQL**: persistencia SQLite. NO hace HTTP.
- **Usuario**: modelo de datos. NO hace HTTP ni SQL.

### Relación

Servidor ◆— BaseDatosSQL —▷ Usuario

`Servidor` **tiene una** `BaseDatosSQL` (composición).
`BaseDatosSQL` **usa** `Usuario` en sus métodos (dependencia).

No se usa herencia porque:
- Un servidor NO es una base de datos.
- La herencia acoplaría los métodos de BD al servidor sin motivo.
- Composición permite cambiar `BaseDatosSQL` por otra implementación
  sin tocar `Servidor`.
```

---

## ✅ Checklist de la Actividad 35

- [ ] `BaseDatosSQL` es un atributo de `Servidor` (composición).
- [ ] `Servidor` no incluye `<sqlite3.h>` (solo lo usa `BaseDatosSQL`).
- [ ] `Usuario` no tiene lógica de archivos ni de HTTP.
- [ ] Los métodos de `Servidor` delegan en `BaseDatosSQL`.
- [ ] No hay dependencias circulares.
- [ ] Todo lo interno de cada clase es `private`.
- [ ] El README documenta el diseño.

## 🧠 Mini-reto
Define una interfaz abstracta `IBaseDatos` con los métodos virtuales puros (`guardarUsuario`, `listarUsuarios`, etc.). Haz que `BaseDatosSQL` la implemente. Cambia `Servidor` para que use `unique_ptr<IBaseDatos>`. Ahora puedes crear `BaseDatosEnMemoria` para tests.