# Actividad 36. Herencia y polimorfismo

## Temas

- Herencia
    
- virtual
    
- override
    

## Ejercicio

Crear la jerarquía:

```text
Usuario
├── Alumno
├── Maestro
└── Administrador
```

Cada tipo de usuario implementará su propia versión de:

```cpp
mostrarMenu();
```

El servidor utilizará punteros o referencias a `Usuario` para trabajar con cualquier tipo de usuario.

---
# TEORÍA

---

## 1. Repaso: herencia y polimorfismo en el contexto del servidor

- **Herencia:** Mecanismo para crear clases derivadas a partir de una base, compartiendo atributos y métodos comunes.
- **Polimorfismo:** Capacidad de que un mismo método tenga comportamientos distintos según el tipo real del objeto.
- **Aplicado al proyecto:**
  - `Usuario` es la clase base con atributos comunes (`nombre`, `usuario`, `contrasena`, `rol`).
  - `Alumno`, `Maestro` y `Administrador` **son** `Usuario`s especializados.
  - Cada uno implementa su versión de `mostrarMenu()`.
  - El servidor maneja punteros o referencias a `Usuario` y, gracias al polimorfismo, llama a la versión correcta.

---

## 2. Diseño de la jerarquía

### A. Clase base `Usuario`
- **Atributos comunes:** `nombre`, `usuario`, `contrasena`, `rol`.
- **Métodos comunes:** getters, setters, `aJSON()`, `esValido()`.
- **Método polimórfico:** `virtual void mostrarMenu() const;` → cada derivada lo sobrescribe.
- **Destructor:** `virtual ~Usuario();` → obligatorio si vas a hacer `delete` sobre punteros a `Usuario`.

### B. Clases derivadas
- **`Alumno`:** Añade `matricula`, `carrera`, (opcionalmente materias y calificaciones si vienes de la Actividad 11). Sobrescribe `mostrarMenu()` con opciones de alumno.
- **`Maestro`:** Añade `especialidad`, (opcionalmente materias que imparte). Sobrescribe `mostrarMenu()` con opciones de maestro.
- **`Administrador`:** Añade `departamento`. Sobrescribe `mostrarMenu()` con opciones de administrador.

### C. Atributos específicos por tipo
- **Alumno:** `matricula`, `carrera`, `semestre`.
- **Maestro:** `especialidad`, `numeroEmpleado`.
- **Administrador:** `departamento`, `nivelAcceso`.

- **Diseño:** Los atributos comunes en la base; los específicos en cada derivada.
- **Regla:** Si un atributo es común a todos, súbelo a la base. Si es específico, déjalo en la derivada.

---

## 3. Modificadores de acceso en la base

- **`protected`:** Permite que las clases derivadas accedan directamente a los atributos heredados sin exponerlos al exterior.
- **`private` con getters/setters:** Alternativa más estricta. Las derivadas usan `getNombre()`, `setNombre()`, etc.
- **Recomendación para esta actividad:** Usa `protected` para `nombre`, `usuario`, `contrasena`, `rol`. Así las derivadas pueden leerlos y modificarlos directamente si lo necesitan, pero el exterior sigue usando getters/setters.
- **`private` puro:** Si prefieres forzar el uso de la interfaz pública incluso dentro de la jerarquía, usa `private` y getters/setters `protected`.

**Fragmento suelto (esqueleto):**
```cpp
class Usuario {
protected:
    string nombre;
    string usuario;
    string contrasena;
    string rol;

public:
    Usuario(string n, string u, string c, string r);
    virtual ~Usuario();
    virtual void mostrarMenu() const;
    // getters, setters, aJSON...
};
```

---

## 4. Función virtual (`virtual`)

- **Declaración en la base:** `virtual void mostrarMenu() const;`
- **Efecto:** Cuando se llama a `mostrarMenu()` a través de un puntero o referencia a `Usuario`, se ejecuta la versión del **tipo real** del objeto (no la del puntero).
- **Sin `virtual`:** Se ejecuta la versión del tipo del puntero (enlace estático). Error clásico.
- **Regla:** Si vas a manejar objetos derivados a través de punteros o referencias a la base, el método debe ser `virtual`.

**Cuándo se activa el polimorfismo:**
- **Sí:** punteros (`Usuario*`) o referencias (`Usuario&`).
- **No:** objetos por valor (`Usuario u = alumno;` → slicing, se pierde la parte derivada).

---

## 5. Sobrescritura (`override`)

- **Declaración en la derivada:**
  ```cpp
  void mostrarMenu() const override;
  ```
- **`override` (C++11):** Palabra clave opcional pero muy recomendable. El compilador verifica que realmente estás sobrescribiendo un método virtual de la base.
- **Errores que detecta:**
  - Firma distinta (por ejemplo, olvidar el `const`).
  - Nombre mal escrito.
  - La base no tiene ese método virtual.
- **`virtual` en la derivada:** Opcional. Una vez virtual en la base, sigue siéndolo en las derivadas.
- **Regla:** Usa siempre `override` en las derivadas. Es gratis y evita errores.

---

## 6. `mostrarMenu()`: comportamiento por tipo

- **Clase base `Usuario`:**
  - Puede tener una implementación por defecto (menú genérico) o ser **virtual pura** (cada derivada obligada a implementarlo).
  - **Recomendación:** Virtual pura (`= 0`) si no existe un "menú genérico" con sentido. Así obligas a cada derivada a implementar el suyo.

- **`Alumno::mostrarMenu()`:**
  - Opciones típicas: ver calificaciones, ver materias, ver horario.
- **`Maestro::mostrarMenu()`:**
  - Opciones típicas: ver materias que imparte, registrar calificaciones, ver alumnos.
- **`Administrador::mostrarMenu()`:**
  - Opciones típicas: gestionar usuarios, ver reportes, configuración del sistema.

- **Formato:** Cada versión imprime (o devuelve) las opciones correspondientes. Podría devolver un string, un vector de strings, o simplemente imprimir por consola.

**Nota conceptual:** En un servidor HTTP, `mostrarMenu()` puede:
- Devolver un string con las opciones (que el servidor incluye en la respuesta HTTP).
- O devolver un JSON con las opciones (más estructurado).
- O simplemente imprimir en consola (menos útil en un servidor).

**Recomendación:** Haz que devuelva un `string` o un vector de strings para que el servidor lo pueda usar en la respuesta al cliente.

**Fragmento suelto (firma alternativa):**
```cpp
virtual string obtenerMenu() const = 0;
```

- **Ventaja:** El servidor recibe el menú como dato y decide cómo enviarlo (HTML, JSON, texto).
- **Desventaja:** Menos directo si lo único que quieres es imprimir.

---

## 7. Destructor virtual

- **Declaración en la base:** `virtual ~Usuario();`
- **Cuándo es obligatorio:** Si vas a hacer `delete` sobre un puntero a `Usuario` que apunta a un objeto derivado.
- **Qué pasa si no lo haces:** Solo se llama al destructor de `Usuario`, no al de la derivada. Fuga de memoria si la derivada reservó recursos.
- **Regla:** Si una clase tiene al menos un método virtual, su destructor también debe ser virtual.

**Fragmento suelto:**
```cpp
class Usuario {
public:
    virtual ~Usuario() {}
};
```

- **Implementación vacía:** Aceptable si no hay nada que liberar. Pero el `virtual` es obligatorio.

---

## 8. Clase abstracta vs. concreta

### A. Clase abstracta
- Tiene al menos una **función virtual pura** (`= 0`).
- **No se pueden crear objetos** de ella, solo punteros o referencias.
- **Cuándo usarla:** Cuando la base no tiene sentido por sí sola. En este proyecto, `Usuario` genérico no se registra; solo se registran alumnos, maestros y administradores.

### B. Clase concreta
- Todos sus métodos virtuales tienen implementación.
- Se pueden crear objetos.

### C. Decisión para esta actividad
- **Opción A:** `Usuario` con `mostrarMenu()` virtual pura → abstracta. **Recomendada.**
- **Opción B:** `Usuario` con implementación por defecto de `mostrarMenu()` → concreta pero rara vez instanciada directamente.
- **Recomendación:** Opción A. Más limpia y refleja que solo existen tipos concretos de usuario.

**Fragmento suelto (virtual pura):**
```cpp
virtual string obtenerMenu() const = 0;
```

---

## 9. Manejo de punteros a `Usuario`

### A. Contenedor polimórfico
- Para almacenar objetos de distintos tipos derivados en una sola colección, usa **punteros a la base**.
- **Contenedor típico:** `vector<Usuario*> usuarios;` o `vector<unique_ptr<Usuario>> usuarios;`.
- **Recomendación:** `vector<unique_ptr<Usuario>>` (C++11+) gestiona la memoria automáticamente. No tienes que hacer `delete` manualmente.

### B. Creación
- Se crea el objeto derivado con `new` y se guarda el puntero en el vector.
- **Con `unique_ptr`:** `usuarios.push_back(make_unique<Alumno>(...));`

### C. Iteración polimórfica
- Se recorre el vector y se llama al método virtual.
- El compilador genera el código que consulta la tabla virtual del objeto para decidir qué versión ejecutar.

**Fragmento suelto (patrón):**
```cpp
for (const auto& u : usuarios) {
    string menu = u->obtenerMenu();  // polimorfismo en acción
    // usar menu
}
```

- **Observa:** `u` es un `unique_ptr<Usuario>`. `u->` accede al objeto apuntado. El método virtual se resuelve en runtime.

---

## 10. Alternativas a `unique_ptr`

### A. Punteros crudos (`Usuario*`)
- **Ventaja:** Simple, explícito.
- **Desventaja:** Debes gestionar `delete` manualmente. Fácil olvidarse → fuga de memoria.
- **Cuándo usar:** Proyectos pequeños o donde ya controlas el ciclo de vida cuidadosamente.

### B. `unique_ptr<Usuario>`
- **Ventaja:** Propiedad exclusiva, liberación automática, sin fugas.
- **Desventaja:** No se puede copiar (solo mover). Requiere `std::move` al pasarlo.
- **Recomendación:** **Usa esta opción.** Es el estándar moderno.

### C. `shared_ptr<Usuario>`
- **Ventaja:** Propiedad compartida, liberación cuando el último puntero desaparece.
- **Desventaja:** Coste de conteo de referencias, ciclos posibles.
- **Cuándo usar:** Cuando varios objetos comparten la propiedad de otro. No es el caso típico aquí.

### D. Referencias (`Usuario&`)
- **Ventaja:** Sintaxis limpia, sin gestión de memoria.
- **Desventaja:** No se pueden almacenar en un `vector` (no son copiables ni asignables).
- **Cuándo usar:** Como parámetros de funciones, no como miembros de un contenedor.

---

## 11. Slicing: el error clásico

- **Definición:** Cuando copias un objeto derivado en una variable del tipo base, se "corta" la parte derivada.
- **Ejemplo:**
  ```cpp
  Alumno a(...);
  Usuario u = a;  // slicing: u solo tiene la parte Usuario
  ```
- **Consecuencia:** `u.obtenerMenu()` llama a la versión de `Usuario` (o error si es virtual pura). El polimorfismo se pierde.
- **Solución:** Usa punteros o referencias a la base, nunca objetos por valor.
- **Regla:** El polimorfismo solo funciona con punteros o referencias.

---

## 12. Serialización polimórfica: reto práctico

- **Problema:** El servidor guarda usuarios en la BD. Necesita saber qué tipo reconstruir al leer.
- **Estrategias:**
  - **Campo `rol`:** Cada objeto tiene `rol` = "alumno", "maestro" o "admin". Al leer, se hace `switch` sobre el rol y se crea la instancia correcta.
  - **Campo `tipo`:** Similar, pero separado del rol.
  - **Tablas separadas:** Una tabla por tipo (`alumnos`, `maestros`, `administradores`). Más limpio pero duplica estructura.
  - **Tabla única con campos opcionales:** Una tabla `usuarios` con columnas para todos los campos posibles; algunas quedan `NULL` según el tipo. Simple pero desperdicia espacio.
- **Recomendación para esta actividad:** Usa el campo `rol` con `switch` al leer de la BD. Es lo más simple y ya tienes `rol` en `Usuario`.

**Fragmento suelto (idea):**
```cpp
unique_ptr<Usuario> crearUsuarioDesdeFila(...) {
    string rol = ...;  // leer de la fila
    if (rol == "alumno") return make_unique<Alumno>(...);
    if (rol == "maestro") return make_unique<Maestro>(...);
    if (rol == "admin") return make_unique<Administrador>(...);
    return nullptr;
}
```

---

## 13. Interacción con `BaseDatosSQL`

- **`BaseDatosSQL` maneja `Usuario`** (o mejor dicho, `unique_ptr<Usuario>`).
- **Al guardar:** Recibe un `const Usuario&`. Puede necesitar saber el tipo real para serializar los campos específicos.
  - **Solución:** `dynamic_cast` o un método virtual `obtenerCamposEspecificos()`.
- **Al leer:** Construye el objeto derivado correcto según el `rol`.
- **Ventaja del polimorfismo:** `BaseDatosSQL` puede trabajar contra la interfaz `Usuario` sin conocer los detalles de cada derivada, salvo en la construcción.

**Nota:** Aquí es donde el polimorfismo y la persistencia se encuentran. Es un tema avanzado. Para esta actividad, quizá simplifiques guardando solo los campos comunes y dejando los específicos para una próxima iteración.

---

## 14. `dynamic_cast` y `typeid` (introducción)

### A. `dynamic_cast`
- Convierte un puntero o referencia a la base en uno a la derivada, verificando en runtime si el objeto es realmente de ese tipo.
- **Uso:** `Alumno* a = dynamic_cast<Alumno*>(usuario);` → devuelve `nullptr` si no es un `Alumno`.
- **Requisito:** La base debe tener al menos un método virtual.
- **Cuándo usar:** Cuando necesitas acceder a métodos específicos de una derivada a través de un puntero a la base.
- **Recomendación:** Úsalo con moderación. Si abusas, probablemente tu diseño no es lo bastante polimórfico.

### B. `typeid`
- Devuelve información del tipo real en runtime.
- `typeid(*usuario).name()` → nombre del tipo (formato dependiente del compilador).
- **Cuándo usar:** Para depurar o para lógica muy específica.
- **Recomendación:** Prefiere métodos virtuales sobre `typeid`.

**Fragmento suelto (uso de `dynamic_cast`):**
```cpp
if (auto a = dynamic_cast<Alumno*>(usuario.get())) {
    // es un Alumno, usar a->getMatricula()
}
```

- **Alternativa mejor:** Añade un método virtual en `Usuario` para lo que necesites. Así evitas el `dynamic_cast`.

---

## 15. Interacción con el servidor

### A. Flujo de un POST `/api/usuarios`
1. El cliente envía JSON con un campo `rol` ("alumno", "maestro", "admin").
2. `Servidor` parsea el JSON.
3. Según el `rol`, crea la instancia adecuada con `make_unique<Alumno>(...)`, `make_unique<Maestro>(...)`, etc.
4. Llama a `baseDatos.guardarUsuario(*usuario)`.
5. Responde al cliente.

### B. Flujo de un GET `/api/menu/{usuario}`
1. `Servidor` busca el usuario en la BD.
2. `BaseDatosSQL` devuelve un `unique_ptr<Usuario>` (o `Usuario*`).
3. `Servidor` llama a `usuario->obtenerMenu()`.
4. El polimorfismo decide qué menú mostrar según el tipo real.
5. `Servidor` envía el menú al cliente.

**Observa:** El servidor **no necesita saber** si el usuario es alumno, maestro o admin. Solo llama al método virtual y confía en el polimorfismo.

---

## 16. Ventajas del polimorfismo en este proyecto

### A. Código más limpio
- Un solo bucle sirve para procesar cualquier tipo de usuario.
- Sin `switch` gigantes para decidir qué hacer según el tipo.

### B. Extensibilidad
- Añadir un nuevo tipo de usuario (por ejemplo, `Bibliotecario`) solo requiere:
  1. Crear la clase derivada.
  2. Sobrescribir `obtenerMenu()` y otros métodos virtuales.
  3. Añadir un caso al `switch` de creación desde la BD.
- El resto del código no cambia.

### C. Separación de responsabilidades
- Cada tipo de usuario sabe qué menú ofrecer.
- El servidor no necesita esa lógica.

### D. Testabilidad
- Puedes crear objetos de cada tipo y probar sus menús por separado.
- Puedes sustituir con mocks fácilmente.

### E. Reutilización
- Los métodos comunes (getters, `aJSON()`) están en la base y se heredan.

---

## 17. Buenas prácticas

- **Marca `virtual` solo en la base.** En las derivadas, usa `override`.
- **Usa `override` siempre.** Detecta errores de firma en tiempo de compilación.
- **Destructor virtual en la base.** Obligatorio si hay polimorfismo con punteros.
- **Prefiere `unique_ptr` sobre punteros crudos.** Gestión automática de memoria.
- **No copies objetos derivados por valor.** Slicing.
- **Prefiere composición sobre herencia** cuando no haya relación "es un" clara.
- **No abuses de `dynamic_cast`.** Si lo necesitas mucho, replantea el diseño.
- **Evita llamar a métodos virtuales en constructores/destructores.** El enlace dinámico no funciona como esperas durante la construcción/destrucción.
- **Considera funciones virtuales puras** si la base no tiene sentido por sí sola.
- **Marca los métodos `const`** si no modifican el objeto.
- **Documenta cada derivada** con un comentario describiendo sus particularidades.
- **Serializa el `rol`** en el JSON y en la BD para poder reconstruir el tipo correcto.
- **Actualiza el diagrama UML** con la jerarquía y las relaciones.
- **Prueba con punteros a la base,** no con objetos concretos, para verificar el polimorfismo.

---

## 18. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `virtual` en la base** | El enlace es estático. Se llama al método de la base, no al de la derivada. |
| **Olvidar `override` en la derivada** | El compilador no verifica la firma. Puedes estar ocultando en lugar de sobrescribiendo. |
| **Firma distinta entre base y derivada** | Debe coincidir exactamente (nombre, parámetros, `const`). Usa `override`. |
| **Destructor de la base no virtual** | Al hacer `delete` sobre puntero a la base, no se llama al destructor de la derivada. Fuga de memoria. |
| **Slicing: copiar derivada en variable base** | Usa punteros o referencias. Nunca objetos por valor. |
| **Llamar a método virtual en constructor/destructor** | El enlace dinámico no funciona. Llama a la versión de la clase actual. |
| **`dynamic_cast` sobre una base sin métodos virtuales** | No compila. La base debe tener al menos un método virtual. |
| **Olvidar `= 0` en virtual pura** | La clase no es abstracta y se puede instanciar, lo cual no deseas. |
| **`vector<Usuario>` en lugar de `vector<Usuario*>`** | Slicing garantizado. Usa punteros o `unique_ptr`. |
| **Confundir sobrecarga con sobrescritura** | Sobrecarga: mismo nombre, distintos parámetros, misma clase. Sobrescritura: misma firma, distinta clase. |
| **No actualizar el `switch` de creación al añadir un tipo** | Es el coste de usar `switch` en lugar de un patrón más extensible (factory). |
| **`Usuario` abstracta pero intentas instanciarla** | Error de compilación. Solo puedes crear punteros o referencias. |
| **No cerrar correctamente al destruir usuarios polimórficos** | Asegúrate de que el destructor virtual se llama. Usa `unique_ptr` para automatizarlo. |

---

## 19. Verificación

Después de implementar la jerarquía:

1. **Compila** sin errores ni warnings.
2. **Crea un objeto de cada tipo** (`Alumno`, `Maestro`, `Administrador`).
3. **Llama a `obtenerMenu()` (o `mostrarMenu()`)** desde un puntero a `Usuario`. Verifica que cada uno devuelve/imprime su menú específico.
4. **Registra cada tipo de usuario** desde el navegador (POST con distintos `rol`).
5. **Guarda los usuarios en la BD.** Verifica que se almacenan correctamente.
6. **Lee los usuarios de la BD.** Verifica que se reconstruyen como el tipo correcto.
7. **Llama a `obtenerMenu()`** sobre cada uno y verifica el resultado.
8. **Añade un nuevo tipo** (por ejemplo, `Bibliotecario`). Verifica que solo necesitas crear la clase y añadir un caso al `switch`.
9. **Elimina un usuario polimórfico.** Verifica que no hay fugas (con valgrind en Linux o el monitor de memoria en Windows).
10. **Prueba el polimorfismo con un `vector<unique_ptr<Usuario>>`** que contenga los tres tipos. Recórrelo y llama a `obtenerMenu()`.

---

## 20. Resumen de conceptos clave

- **Herencia:** `Alumno`, `Maestro`, `Administrador` **son** `Usuario`.
- **Atributos comunes en la base:** `nombre`, `usuario`, `contrasena`, `rol`.
- **Atributos específicos en las derivadas:** matrícula, especialidad, departamento, etc.
- **`protected`:** Accesible desde la clase y sus derivadas.
- **`virtual`:** Habilita el polimorfismo en la base.
- **`override`:** Verifica la sobrescritura en las derivadas.
- **Destructor virtual:** Obligatorio para liberar correctamente objetos polimórficos.
- **Virtual pura (`= 0`):** Convierte la base en abstracta. Cada derivada debe implementar el método.
- **Punteros o referencias:** Únicos medios para que funcione el polimorfismo. Los objetos por valor producen slicing.
- **`unique_ptr<Usuario>`:** Contenedor polimórfico recomendado. Gestión automática de memoria.
- **`mostrarMenu()` / `obtenerMenu()`:** Método virtual que cada derivada implementa a su manera.
- **El servidor usa la base:** No necesita saber el tipo real para llamar al método virtual.
- **Serialización:** Usa el campo `rol` para reconstruir el tipo al leer de la BD.
- **`dynamic_cast`:** Útil pero con moderación. Prefiere métodos virtuales.
- **Ventajas:** Código limpio, extensible, testable, con responsabilidades bien separadas.

---
# PASO A PASO

---
# Actividad 36 — Herencia y polimorfismo

## Objetivo
Crear la jerarquía `Usuario → Alumno, Maestro, Administrador`. Cada tipo implementa su propio `mostrarMenu()` / `obtenerMenu()`. El servidor trabaja con punteros a `Usuario` y no necesita saber el tipo real.

## Antes
```cpp
class Usuario { ... };  // todos los campos en uno
```

## Después
```cpp
class Usuario { ... virtual string obtenerMenu() const = 0; };
class Alumno    : public Usuario { ... };
class Maestro   : public Usuario { ... };
class Administrador : public Usuario { ... };
```

---

## Paso 1 — `Usuario` se vuelve abstracta

Modifica `Usuario` para que `obtenerMenu()` sea **virtual pura**:

```cpp
class Usuario {
protected:
    string nombre;
    string usuario;
    string contrasena;
    string rol;

public:
    Usuario(string n, string u, string c, string r)
        : nombre(n), usuario(u), contrasena(c), rol(r) {}

    virtual ~Usuario() {}   // ⚠️ destructor virtual OBLIGATORIO

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

    // Método virtual puro → cada derivada lo implementa
    virtual string obtenerMenu() const = 0;

    bool esValido() const {
        return !nombre.empty() && !usuario.empty()
            && !contrasena.empty() && !rol.empty();
    }
};
```

📌 Puntos clave:
- `protected` → las derivadas acceden a `nombre`, `usuario`, etc.
- `virtual ~Usuario()` → **obligatorio** si vas a usar `delete` sobre punteros a `Usuario`.
- `= 0` en `obtenerMenu` → **virtual pura**. `Usuario` es ahora **abstracta**: no puedes hacer `Usuario u(...)`.

---

## Paso 2 — Clase `Alumno`

```cpp
class Alumno : public Usuario {
private:
    string matricula;
    string carrera;

public:
    Alumno(string n, string u, string c, string mat, string car)
        : Usuario(n, u, c, "alumno"),
          matricula(mat), carrera(car) {}

    string getMatricula() const { return matricula; }
    string getCarrera()   const { return carrera; }

    string obtenerMenu() const override {
        return "[Menú Alumno]\n"
               "  1. Ver calificaciones\n"
               "  2. Ver materias\n"
               "  3. Ver horario\n";
    }
};
```

---

## Paso 3 — Clase `Maestro`

```cpp
class Maestro : public Usuario {
private:
    string especialidad;
    string numeroEmpleado;

public:
    Maestro(string n, string u, string c, string esp, string emp)
        : Usuario(n, u, c, "maestro"),
          especialidad(esp), numeroEmpleado(emp) {}

    string getEspecialidad()   const { return especialidad; }
    string getNumeroEmpleado() const { return numeroEmpleado; }

    string obtenerMenu() const override {
        return "[Menú Maestro]\n"
               "  1. Ver mis materias\n"
               "  2. Registrar calificaciones\n"
               "  3. Ver lista de alumnos\n";
    }
};
```

---

## Paso 4 — Clase `Administrador`

```cpp
class Administrador : public Usuario {
private:
    string departamento;

public:
    Administrador(string n, string u, string c, string dep)
        : Usuario(n, u, c, "admin"),
          departamento(dep) {}

    string getDepartamento() const { return departamento; }

    string obtenerMenu() const override {
        return "[Menú Administrador]\n"
               "  1. Gestionar usuarios\n"
               "  2. Ver reportes\n"
               "  3. Configuración del sistema\n";
    }
};
```

---

## Paso 5 — Fábrica de usuarios desde la BD

Ahora `BaseDatosSQL` guarda el `rol` en la columna `rol`. Al leer, hay que **reconstruir el tipo correcto**.

Agrega esta función (puede ir en `BaseDatosSQL` o como helper en el archivo):

```cpp
#include <memory>
using std::unique_ptr;
using std::make_unique;

unique_ptr<Usuario> crearUsuarioDesdeFila(sqlite3_stmt* stmt) {
    string nombre   ((const char*)sqlite3_column_text(stmt, 0));
    string usuario  ((const char*)sqlite3_column_text(stmt, 1));
    string contra   ((const char*)sqlite3_column_text(stmt, 2));
    string rol      ((const char*)sqlite3_column_text(stmt, 3));

    // En una BD real, habría más columnas para los campos específicos.
    // Aquí las dejamos vacías para simplificar.
    if (rol == "alumno")
        return make_unique<Alumno>(nombre, usuario, contra, "", "");
    if (rol == "maestro")
        return make_unique<Maestro>(nombre, usuario, contra, "", "");
    if (rol == "admin")
        return make_unique<Administrador>(nombre, usuario, contra, "");

    return nullptr;
}
```

⚠️ En una implementación completa, la tabla tendría columnas como `matricula`, `carrera`, etc. Para esta actividad, se pueden dejar vacías y rellenar en una próxima iteración.

---

## Paso 6 — `listarUsuarios` con polimorfismo

Cambia el tipo de retorno:

```cpp
vector<unique_ptr<Usuario>> BaseDatosSQL::listarUsuarios() {
    vector<unique_ptr<Usuario>> lista;
    if (!conectada) return lista;

    const char* sql = "SELECT nombre, usuario, contrasena, rol FROM usuarios";
    sqlite3_stmt* stmt = nullptr;
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return lista;

    while (sqlite3_step(stmt) == SQLITE_ROW) {
        if (auto u = crearUsuarioDesdeFila(stmt)) {
            lista.push_back(std::move(u));
        }
    }

    sqlite3_finalize(stmt);
    return lista;
}
```

🎯 Ahora la lista contiene punteros a `Usuario`, pero cada uno apunta a un objeto del tipo real (`Alumno`, `Maestro`, `Administrador`).

---

## Paso 7 — Uso polimórfico en el servidor

Endpoint `GET /api/menus` que devuelve el menú de cada usuario:

```cpp
if (ruta == "/api/menus" && metodo == "GET") {
    auto usuarios = baseDatos.listarUsuarios();

    string json = "[\n";
    for (size_t i = 0; i < usuarios.size(); i++) {
        if (i > 0) json += ",\n";
        json += "  {\"usuario\":\"" + escaparJSON(usuarios[i]->getUsuario()) + "\",";
        json += "\"menu\":\""      + escaparJSON(usuarios[i]->obtenerMenu()) + "\"}";
    }
    json += "\n]";

    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

🎯 **`usuarios[i]->obtenerMenu()`** es la magia. Aunque `usuarios[i]` es un `unique_ptr<Usuario>`, el método virtual se resuelve al tipo **real** (`Alumno`, `Maestro`, `Administrador`).

---

## Paso 8 — Probar el polimorfismo

En la consola del servidor, tras el arranque, prueba:

```cpp
// Dentro de ejecutar(), después de conectar():
auto usuarios = baseDatos.listarUsuarios();
for (const auto& u : usuarios) {
    cout << u->getUsuario() << " →\n" << u->obtenerMenu() << "\n\n";
}
```

Salida esperada (si tienes 3 tipos):

```
ana_lopez →
[Menú Alumno]
  1. Ver calificaciones
  ...

luis_garcia →
[Menú Maestro]
  1. Ver mis materias
  ...

sofia_admin →
[Menú Administrador]
  1. Gestionar usuarios
  ...
```

✅ El servidor no sabe en tiempo de compilación qué tipo es cada uno. En tiempo de ejecución, cada uno ejecuta **su** versión.

---

## Paso 9 — Slicing: por qué punteros y no valores

Si intentas esto, perderás la parte derivada:

```cpp
Alumno a("Ana", "ana", "x", "A001", "TI");
Usuario u = a;   // ❌ slicing: u solo tiene la parte Usuario
u.obtenerMenu(); // ni siquiera compila (virtual pura)
```

**Regla:** Con polimorfismo, usa **punteros o referencias**. Nunca objetos por valor.

| Forma | Polimorfismo | Ciclo de vida |
|-------|--------------|---------------|
| `Usuario*` | ✅ | manual (`delete`) |
| `unique_ptr<Usuario>` | ✅ | automático |
| `Usuario&` | ✅ | no copiable |
| `Usuario` (valor) | ❌ slicing | automático |

---

## Paso 10 — Verificar

1. Compila con `-std=c++17 -Wall -Wextra`.
2. Registra 3 usuarios con distintos `rol` desde el navegador.
3. Abre `http://localhost:8080/api/menus`. Debe mostrar tres menús distintos.
4. En la consola del servidor, recorre `listarUsuarios()` y verifica que cada uno devuelve su propio menú.
5. Si usas `valgrind` (Linux): `valgrind ./servidor`. No debe haber fugas.

---

## ✅ Checklist de la Actividad 36

- [ ] `Usuario` es abstracta (tiene al menos un método virtual puro).
- [ ] `Usuario` tiene destructor virtual.
- [ ] `Alumno`, `Maestro`, `Administrador` heredan y usan `override`.
- [ ] Cada derivada implementa `obtenerMenu()`.
- [ ] `listarUsuarios()` devuelve `vector<unique_ptr<Usuario>>`.
- [ ] El servidor llama `u->obtenerMenu()` sin `if (rol == ...)`.
- [ ] Reconstruir desde la BD usa `rol` para decidir el tipo.

## 🧠 Mini-reto
Añade un nuevo tipo `Bibliotecario` con su propio `obtenerMenu()`. Verifica que solo necesitas crear la clase y añadir un caso en `crearUsuarioDesdeFila`.