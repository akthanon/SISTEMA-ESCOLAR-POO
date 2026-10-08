# Actividad 8. Primeras clases

En lugar de tener

```cpp
string nombres[100];

int edades[100];

float promedios[100];
```

crear

```cpp
class Alumno
```

con

```
nombre

edad

promedio
```

y reemplazar todo por

```cpp
Alumno alumnos[100];
```

---
# TEORÍA

---

## 1. ¿Qué es una clase?

- **Definición:** Una clase es un **molde** o **plantilla** que define las características (atributos) y comportamientos (métodos) que tendrán los objetos creados a partir de ella.
- **Objeto:** Es una **instancia** concreta de una clase. Mientras la clase es el plano, el objeto es la casa construida.
- **Diferencia con struct:** En C++, `class` y `struct` son casi idénticos. La única diferencia por defecto es la visibilidad: en `class` los miembros son **privados** por defecto; en `struct` son **públicos**. Por convención, se usa `class` cuando hay encapsulamiento y `struct` para agrupaciones simples de datos.

**Fragmento suelto (declaración):**
```cpp
class Alumno {
    // atributos y métodos
};
```

- **Importante:** No olvides el `;` al final de la llave de cierre de la clase.

---

## 2. Atributos (variables miembro)

- **Definición:** Son las variables que describen el estado del objeto.
- **Declaración:** Se declaran dentro de la clase, igual que cualquier variable.
- **Para esta actividad:**
  - `string nombre;`
  - `int edad;`
  - `float promedio;` (o `double`, según prefieras)

**Fragmento suelto:**
```cpp
class Alumno {
    string nombre;
    int edad;
    float promedio;
};
```

---

## 3. Modificadores de acceso

- **`private`:** Los miembros solo son accesibles desde dentro de la propia clase. Es el nivel por defecto en `class`.
- **`public`:** Los miembros son accesibles desde cualquier parte del programa (desde `main`, desde otras funciones, etc.).
- **`protected`:** Similar a `private`, pero accesible también desde clases derivadas (herencia). No lo necesitas en esta actividad.
- **Regla de oro del encapsulamiento:** Los **atributos** deben ser `private` y se accede a ellos mediante **métodos públicos** (getters y setters). Sin embargo, para esta actividad inicial, puedes hacerlos `public` para simplificar la migración. En la siguiente actividad probablemente se enseñe encapsulamiento.

**Fragmento suelto:**
```cpp
class Alumno {
public:
    string nombre;
    int edad;
    float promedio;
};
```

---

## 4. Objetos y arreglos de objetos

- **Declaración de un objeto:** `Alumno alumno1;` crea una instancia con atributos sin inicializar.
- **Arreglo de objetos:** `Alumno alumnos[100];` crea 100 objetos `Alumno`, cada uno con sus propios atributos.
- **Acceso a atributos de un objeto:** Se usa el operador **punto** `.`:
  - `alumnos[i].nombre`
  - `alumnos[i].edad`
  - `alumnos[i].promedio`

**Fragmento suelto (acceso):**
```cpp
alumnos[0].nombre = "Ana";
alumnos[0].edad = 20;
cout << alumnos[i].nombre;
```

- **Ventaja:** En lugar de tener tres arreglos paralelos (nombres, edades, promedios), tienes **un solo arreglo** donde cada elemento agrupa toda la información de un alumno. Esto reduce errores de sincronización (por ejemplo, olvidar mover los tres arreglos al eliminar).

---

## 5. Métodos (funciones miembro)

- **Definición:** Son funciones declaradas dentro de la clase. Operan sobre los atributos del objeto.
- **Declaración dentro de la clase:** Se pueden definir directamente o solo declarar el prototipo y definir fuera con el operador `::` (scope resolution).
- **Métodos útiles para esta actividad:**
  - `void mostrar();` → Imprime los datos del alumno.
  - `void capturar();` → Pide al usuario los datos del alumno.
- **Llamada a métodos:** Igual que los atributos, con el operador punto: `alumnos[i].mostrar();`

**Fragmento suelto (declaración y definición fuera):**
```cpp
class Alumno {
public:
    string nombre;
    int edad;
    float promedio;
    void mostrar();   // prototipo dentro de la clase
};

void Alumno::mostrar() {   // definición fuera, usando ::
    // cuerpo del método
}
```

- **Operador `::` (resolución de ámbito):** Indica que la función `mostrar` pertenece a la clase `Alumno`.

---

## 6. Constructores (introducción)

- **Definición:** Un constructor es un método especial que se ejecuta **automáticamente** al crear un objeto. Sirve para inicializar los atributos.
- **Características:**
  - Tiene el **mismo nombre que la clase**.
  - **No tiene tipo de retorno** (ni siquiera `void`).
  - Puede tener parámetros (constructor parametrizado) o ninguno (constructor por defecto).
- **Constructor por defecto:** Si no defines ningún constructor, C++ genera uno automáticamente que no inicializa los atributos (quedan con valores basura). Es recomendable definir uno propio.
- **Para esta actividad:** Puedes definir un constructor que inicialice los atributos a valores por defecto (por ejemplo, nombre vacío, edad 0, promedio 0.0) o simplemente no definirlo y asignar valores después.

**Fragmento suelto (constructor):**
```cpp
class Alumno {
public:
    string nombre;
    int edad;
    float promedio;
    
    Alumno() {   // constructor por defecto
        nombre = "";
        edad = 0;
        promedio = 0.0;
    }
};
```

---

## 7. Migración del código: qué cambia

- **Antes (tres arreglos paralelos):**
  - `string nombres[100];`
  - `int edades[100];`
  - `float promedios[100];`
- **Ahora (un arreglo de objetos):**
  - `Alumno alumnos[100];`
- **Cambios en las funciones:**
  - Los parámetros ya no son tres arreglos separados, sino **un solo arreglo de `Alumno`**.
  - Ejemplo de nueva firma: `void mostrarTodos(Alumno alumnos[], int cantidad);`
  - El acceso a los datos cambia: en lugar de `nombres[i]`, usas `alumnos[i].nombre`.
  - Al eliminar, ya no desplazas tres arreglos en paralelo, sino **un solo arreglo de objetos**: `alumnos[i] = alumnos[i+1];` (esto copia todos los atributos de golpe).
- **Ventaja:** El código es más limpio, coherente y menos propenso a errores.

---

## 8. Funciones auxiliares: ¿métodos o funciones externas?

- **Opción A: Métodos dentro de la clase.** Cada operación sobre un alumno (mostrar, capturar) es un método. Las operaciones sobre el arreglo completo (buscar, eliminar, mostrar todos) siguen siendo funciones externas que reciben el arreglo de alumnos.
- **Opción B: Todo como funciones externas.** La clase `Alumno` solo tiene atributos, y todas las operaciones se hacen desde fuera accediendo a `.nombre`, `.edad`, `.promedio`. Es menos "orientado a objetos" pero más rápido de migrar.
- **Recomendación:** Para esta actividad, usa métodos para las operaciones **de un solo alumno** (mostrar, capturar) y funciones externas para las operaciones **sobre el arreglo** (buscar, eliminar, mostrar todos, intercambiar). Así empiezas a aprovechar las clases sin reescribir todo.

---

## 9. Paso de objetos por referencia

- **Objetos como parámetros:** Se pueden pasar por valor (copia) o por referencia (`&`).
- **Paso por valor:** Copia todo el objeto. Costoso si el objeto es grande (como un `string` largo). Los cambios no afectan al original.
- **Paso por referencia:** `Alumno &a` → se trabaja con el objeto original. Los cambios sí se reflejan. Es lo que usarás para `intercambiarAlumnos`.
- **Paso por referencia constante:** `const Alumno &a` → se pasa por referencia (eficiente) pero no se puede modificar. Ideal para funciones que solo leen (como `mostrar`).

**Fragmentos sueltos:**
```cpp
void mostrar(const Alumno &a);          // solo lectura, eficiente
void modificar(Alumno &a);              // puede modificar
void intercambiar(Alumno &a, Alumno &b); // intercambia dos objetos
```

- **Intercambio de objetos:** Con la clase, el intercambio se simplifica enormemente:
  - `Alumno temp = a; a = b; b = temp;`
  - Esto funciona porque el compilador genera automáticamente el operador de asignación `=` para copiar todos los atributos.

---

## 10. Buenas prácticas

- **Nombres de clases:** En PascalCase o UpperCamelCase (`Alumno`, `Materia`, `Profesor`).
- **Nombres de atributos:** En camelCase o snake_case (`nombreCompleto`, `promedioFinal`).
- **Métodos:** Verbos en infinitivo o imperativo (`mostrar`, `capturar`, `calcularPromedio`).
- **Encapsulamiento:** Aunque en esta actividad uses atributos `public` para simplificar, en la próxima considera hacerlos `private` y agregar getters/setters.
- **Constructores:** Define al menos un constructor por defecto para evitar valores basura.
- **Reutilización:** Si varios métodos comparten lógica, extráela a métodos privados auxiliares.

---

## 11. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `;` al final de la clase** | Siempre cierra con `};` (llave + punto y coma). |
| **No declarar `public:` antes de los miembros** | Por defecto son `private`; si los usas desde `main`, deben ser `public`. |
| **Confundir el nombre de la clase con el de una variable** | `Alumno alumnos[100];` → `Alumno` es la clase, `alumnos` es el arreglo. |
| **Definir métodos fuera sin `::`** | Usa `tipo_retorno Clase::metodo() { ... }`. |
| **Pasar objetos por valor cuando son grandes** | Usa `const Clase &` para lectura o `Clase &` para modificar. |
| **No inicializar atributos** | Define un constructor por defecto o asigna valores antes de usar. |
| **Acceder con `->` en lugar de `.`** | Con objetos (no punteros) usa `.`; con punteros usa `->`. |

---

## 12. Resumen de conceptos clave

- **Clase:** Molde que agrupa atributos y métodos.
- **Objeto:** Instancia concreta de una clase.
- **Atributos:** Variables miembro que describen el estado.
- **Métodos:** Funciones miembro que definen el comportamiento.
- **Modificadores de acceso:** `public`, `private`, `protected`.
- **Arreglo de objetos:** `Alumno alumnos[100];` reemplaza a los arreglos paralelos.
- **Acceso:** Operador `.` para atributos y métodos.
- **Constructor:** Método especial que inicializa el objeto al crearlo.
- **Paso por referencia:** `Alumno &` para modificar, `const Alumno &` para solo leer.
- **Intercambio de objetos:** Se simplifica gracias al operador `=` generado automáticamente.

---
