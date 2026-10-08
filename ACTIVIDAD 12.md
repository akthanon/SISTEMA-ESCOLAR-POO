# Actividad 12. Herencia

## Temas

- Clase base
    
- Clase derivada
    

## Ejercicio

Crear:

```text
Persona
├── Alumno
├── Maestro
└── Administrador
```

Cada uno tendrá:

- nombre
    
- edad
    

Y atributos propios.

Alumno

- matrícula
    

Maestro

- especialidad
    

Administrador

- departamento
    

Agregar una opción para registrar cualquiera de los tres.

---
# TEORÍA

---

## 1. ¿Qué es la herencia?

- **Definición:** Es un mecanismo de la POO que permite crear una **clase derivada** (hija) a partir de una **clase base** (padre). La clase derivada **hereda** los atributos y métodos de la base, y puede añadir los suyos propios o modificar los heredados.
- **Relación:** "es un" (is-a). Un `Alumno` **es una** `Persona`; un `Maestro` **es una** `Persona`; un `Administrador` **es una** `Persona`.
- **Ventajas:**
  - Reutilización de código (los atributos y métodos comunes se escriben una sola vez).
  - Jerarquías claras que reflejan el mundo real.
  - Facilita la extensibilidad (agregar nuevas clases derivadas sin tocar la base).

---

## 2. Sintaxis de la herencia

- **Declaración:**
  ```cpp
  class Derivada : public Base {
      // ...
  };
  ```
- **Tipos de herencia:** `public`, `protected`, `private`. Para este ejercicio usa **`public`**, que es la más común: los miembros `public` de la base siguen siendo `public` en la derivada; los `protected` siguen siendo `protected`; los `private` no son accesibles directamente desde la derivada.
- **Acceso a miembros de la base desde la derivada:**
  - Si son `public` o `protected`, se accede directamente por nombre.
  - Si son `private`, no se puede acceder directamente; se usan getters/setters `public` o `protected` de la base.

---

## 3. Diseño de la jerarquía

- **Clase base `Persona`:**
  - Atributos comunes: `nombre`, `edad`.
  - Métodos: constructores, destructor, getters, setters, `mostrar()`.
  - Estos miembros son heredados por todas las clases derivadas.
- **Clases derivadas:**
  - `Alumno` añade `matricula` (y todo lo que ya tenía de actividades anteriores: materias, calificaciones, etc.).
  - `Maestro` añade `especialidad`.
  - `Administrador` añade `departamento`.
- **Cada derivada** debe tener sus propios constructores (vacío y con parámetros), destructor, getters/setters y `mostrar()` (que puede reutilizar el de `Persona` y añadir lo propio).

**Fragmento suelto (esqueleto):**
```cpp
class Persona {
protected:      // protected para que las derivadas accedan directamente
    string nombre;
    int edad;

public:
    // constructores, destructor, getters, setters, mostrar()
};

class Alumno : public Persona {
private:
    string matricula;

public:
    // constructores, destructor, getters, setters, mostrar()
};
```

---

## 4. Modificador `protected`

- **¿Por qué `protected` y no `private` en la base?**
  - Con `private`, las clases derivadas **no pueden acceder** directamente a los atributos heredados; tendrían que usar getters/setters públicos de la base.
  - Con `protected`, las clases derivadas **sí pueden acceder** directamente, pero el exterior sigue sin poder hacerlo.
- **Regla práctica:** Usa `protected` para atributos que las clases derivadas necesiten manipular directamente. Usa `private` si prefieres forzar el uso de getters/setters incluso dentro de la jerarquía.

---

## 5. Constructores en herencia

- **Orden de construcción:** Primero se construye la **base**, luego los atributos de la derivada, luego el cuerpo del constructor de la derivada.
- **Llamada al constructor de la base:** Si la base no tiene constructor por defecto (o quieres usar uno con parámetros), debes invocarlo explícitamente en la **lista de inicialización** del constructor de la derivada.
- **Sintaxis:**
  ```cpp
  Alumno(string n, int e, string m) : Persona(n, e), matricula(m) {
      // cuerpo
  }
  ```
- **Si no se especifica**, se llama al constructor por defecto de la base. Si la base no tiene constructor por defecto, el compilador da error.
- **Recomendación:** Define siempre un constructor por defecto en la base para evitar problemas al crear arreglos de objetos derivados.

---

## 6. Destructores en herencia

- **Orden de destrucción:** Inverso al de construcción: primero se destruye la derivada, luego la base.
- **El destructor de la base debe ser `virtual`** si vas a usar punteros a `Persona` apuntando a objetos derivados y quieres que se llame al destructor correcto. Para esta actividad, si no usas polimorfismo con punteros, no es estrictamente necesario, pero es buena práctica declararlo `virtual` en la base.
- **Sintaxis:**
  ```cpp
  virtual ~Persona() { /* ... */ }
  ```

---

## 7. Reutilización de métodos: `mostrar()`

- La clase derivada puede **sobrescribir** (override) el método `mostrar()` de la base para añadir su propia información.
- **Opción 1:** Llamar al `mostrar()` de la base y luego añadir lo propio:
  ```cpp
  void mostrar() {
      Persona::mostrar();   // llama al mostrar de la base
      cout << "Matricula: " << matricula << endl;
  }
  ```
- **Opción 2:** Reescribir todo desde cero (menos recomendable por duplicación).
- **Polimorfismo (avanzado):** Si quieres que un puntero a `Persona` llame al `mostrar()` correcto según el tipo real del objeto, declara `mostrar()` como `virtual` en la base. Esto se verá más adelante, pero tenlo en mente.

---

## 8. Arreglos de objetos derivados vs. arreglos de punteros

- **Arreglo de objetos derivados:** `Alumno alumnos[100];` funciona igual que antes. Cada elemento es un `Alumno` completo.
- **Arreglo de punteros a la base:** `Persona* personas[100];` permite almacenar objetos de **cualquier clase derivada** (Alumno, Maestro, Administrador) usando punteros. Es la forma de tener un arreglo heterogéneo.
  - Para crear: `personas[i] = new Alumno(...);`
  - Para destruir: `delete personas[i];` (por eso el destructor de la base debe ser `virtual`).
- **Para esta actividad:** Si solo quieres registrar "cualquiera de los tres", puedes usar arreglos separados para cada tipo, o un arreglo de punteros a `Persona`. El arreglo de punteros es más elegante pero requiere manejo de memoria dinámica.

**Fragmento suelto (arreglo de punteros):**
```cpp
Persona* personas[100];
int numPersonas = 0;

// registrar un alumno:
personas[numPersonas] = new Alumno(...);
numPersonas++;

// al salir:
for (int i = 0; i < numPersonas; i++) {
    delete personas[i];
}
```

---

## 9. Menú actualizado

- **Nueva opción:** "Registrar persona" (o "Registrar alumno/maestro/administrador").
- **Submenú:** Preguntar qué tipo de persona registrar:
  - 1 → Alumno (pide matrícula, materias, etc.)
  - 2 → Maestro (pide especialidad)
  - 3 → Administrador (pide departamento)
- **Mostrar:** Puedes tener opciones separadas para mostrar cada tipo, o una sola que recorra el arreglo de punteros y llame a `mostrar()` (aprovechando el polimorfismo si `mostrar()` es `virtual`).

---

## 10. Interacción con composición (Actividad 11)

- **`Alumno` ahora hereda de `Persona`** y sigue teniendo composición con `Materia` y `Calificacion`.
- **`Maestro` y `Administrador`** no necesitan materias ni calificaciones, solo sus atributos propios.
- **Orden de declaración:**
  1. `Materia`
  2. `Calificacion`
  3. `Persona` (base)
  4. `Alumno`, `Maestro`, `Administrador` (derivadas)

---

## 11. Buenas prácticas

- **Herencia solo cuando hay relación "es un".** No uses herencia para reutilizar código si no hay una relación conceptual clara.
- **Base con destructor virtual** si vas a usar punteros a la base.
- **Constructores con lista de inicialización** para llamar al constructor de la base.
- **Evita atributos `protected`** si puedes usar `private` + getters/setters. `protected` es útil cuando las derivadas necesitan acceso directo.
- **Sobrescritura con `override`** (C++11 en adelante): marca los métodos que sobrescriben con la palabra clave `override` para que el compilador verifique que realmente están sobrescribiendo algo.
  ```cpp
  void mostrar() override { ... }
  ```
- **Documenta la jerarquía:** un diagrama simple ayuda a visualizar las relaciones.
- **No abuses de la herencia profunda:** jerarquías de más de 3-4 niveles suelen ser difíciles de mantener. Prefiere composición cuando sea posible.

---

## 12. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `public` en la herencia** | `class Alumno : public Persona` (sin `public`, la herencia es `private` por defecto). |
| **No llamar al constructor de la base con parámetros** | Usa la lista de inicialización: `Alumno(...) : Persona(...) { ... }`. |
| **Base sin constructor por defecto y derivada sin lista de inicialización** | Define un constructor por defecto en la base o llama explícitamente al constructor con parámetros. |
| **Destructor de la base no virtual con punteros** | Declara `virtual ~Persona()` para evitar fugas de memoria. |
| **Acceder a atributos `private` de la base desde la derivada** | Usa `protected` o getters/setters públicos. |
| **Olvidar el `;` al final de cada clase** | Cierra con `};`. |
| **Confundir composición con herencia** | "Tiene un" → composición; "es un" → herencia. |
| **Duplicar atributos comunes en cada derivada** | Muévelos a la base `Persona`. |

---

## 13. Resumen de conceptos clave

- **Herencia:** Mecanismo para crear clases derivadas a partir de una base (relación "es un").
- **Sintaxis:** `class Derivada : public Base { ... };`
- **`protected`:** Accesible desde la clase y sus derivadas, no desde fuera.
- **Constructores:** Se llama primero al de la base (explícitamente o por defecto).
- **Destructores:** Orden inverso; el de la base debe ser `virtual` si hay punteros.
- **Sobrescritura:** La derivada puede redefinir métodos de la base (marca con `override`).
- **Reutilización:** Llama a `Base::metodo()` desde la derivada para aprovechar la lógica existente.
- **Arreglos de punteros:** Permiten almacenar objetos de distintos tipos derivados en una sola colección.
- **Jerarquía:** `Persona` → `Alumno`, `Maestro`, `Administrador`.

---
