# Actividad 11. Composición

## Temas

- Composición
    
- Objetos dentro de objetos
    

## Ejercicio

Crear las clases:

- `Materia`
    
- `Calificacion`
    

y agregarlas a la clase `Alumno`.

Ejemplo:

```text
Alumno
 ├── nombre
 ├── edad
 ├── Materia[10]
 └── Calificacion[10]
```

Agregar opciones al menú:

- Agregar materia
    
- Registrar calificación
    
- Mostrar historial académico
    

---
# TEORÍA

---

## 1. ¿Qué es la composición?

- **Definición:** La composición es una relación entre clases donde una clase **contiene** objetos de otra clase como parte de sus atributos. Es una relación "tiene un" (has-a).
- **Diferencia con herencia:** La herencia es "es un" (is-a), la composición es "tiene un" (has-a).
- **Ejemplo:** Un `Alumno` **tiene** un nombre, una edad, y **tiene** materias y calificaciones. Las materias y calificaciones no son tipos de alumno, sino partes que lo componen.
- **Ventaja:** Permite modelar relaciones del mundo real de forma más natural y reutilizar clases.

---

## 2. Clases involucradas

### A. Clase `Materia`

- **Propósito:** Representar una asignatura (por ejemplo, "Matemáticas", "Programación").
- **Atributos típicos:**
  - `string nombre;` (nombre de la materia)
  - `string clave;` (opcional, identificador único)
  - `int creditos;` (opcional)
- **Métodos típicos:**
  - Getters y setters para cada atributo.
  - Constructor vacío y con parámetros.
  - `mostrar()` para imprimir la información.

### B. Clase `Calificacion`

- **Propósito:** Representar la calificación obtenida en una materia.
- **Atributos típicos:**
  - `string claveMateria;` (a qué materia corresponde)
  - `float nota;` (la calificación obtenida)
  - `string periodo;` (opcional, por ejemplo "2024-1")
- **Métodos típicos:**
  - Getters y setters.
  - Constructores.
  - `mostrar()`.

### C. Clase `Alumno` (modificada)

- **Nuevos atributos:**
  - `Materia materias[10];` → arreglo de objetos `Materia`.
  - `Calificacion calificaciones[10];` → arreglo de objetos `Calificacion`.
  - `int numMaterias;` → contador de materias registradas.
  - `int numCalificaciones;` → contador de calificaciones registradas.
- **Observación:** Los arreglos de objetos se construyen **automáticamente** al construir el `Alumno`. Es decir, al crear un `Alumno`, se crean también sus 10 materias y 10 calificaciones (con sus constructores por defecto). Esto es importante para el ciclo de vida.

**Fragmento suelto (esqueleto):**
```cpp
class Alumno {
private:
    string nombre;
    int edad;
    Materia materias[10];
    Calificacion calificaciones[10];
    int numMaterias;
    int numCalificaciones;

public:
    // constructores, destructor, getters, setters, métodos...
};
```

---

## 3. Orden de declaración de clases

- **Regla importante:** Para que una clase pueda contener objetos de otra, la **clase contenida debe estar declarada antes** que la clase contenedora.
- **Orden recomendado:**
  1. `Materia`
  2. `Calificacion`
  3. `Alumno`
- Si necesitas referencias cruzadas (por ejemplo, `Materia` necesita saber de `Alumno`), se usan **forward declarations** (`class Alumno;`), pero en este ejercicio no es necesario.

---

## 4. Constructores y composición

- **Construcción automática:** Cuando se construye un `Alumno`, se construyen automáticamente sus arreglos de `Materia` y `Calificacion`. Cada uno de esos objetos llama a su constructor por defecto.
- **Lista de inicialización:** Si quieres inicializar los objetos contenidos con valores específicos, puedes usar la lista de inicialización del constructor del contenedor.
- **Mensajes de traza:** Si pusiste mensajes en los constructores de `Materia` y `Calificacion`, verás muchos mensajes al crear un `Alumno`. Ten cuidado con la saturación de la consola.

**Fragmento suelto (constructor de Alumno con lista de inicialización):**
```cpp
Alumno() : numMaterias(0), numCalificaciones(0) {
    // el resto de la inicialización
}
```

- **Orden de construcción:** Los atributos se construyen en el orden en que están declarados en la clase, no en el orden de la lista de inicialización. Tenlo en cuenta si hay dependencias.

---

## 5. Destructores y composición

- **Destrucción automática:** Cuando se destruye un `Alumno`, se destruyen automáticamente sus arreglos de `Materia` y `Calificacion`, llamando a sus destructores.
- **Orden de destrucción:** Inverso al de construcción (el último construido se destruye primero).
- **Mensajes de traza:** Si pusiste mensajes en los destructores, verás muchos mensajes al destruir un `Alumno`.

---

## 6. Funciones para gestionar materias y calificaciones

### A. Agregar materia

- **Propósito:** Añadir una nueva materia al arreglo `materias` del alumno.
- **Pasos:**
  1. Verificar que `numMaterias < 10` (hay espacio).
  2. Pedir los datos de la materia (nombre, clave, etc.).
  3. Asignar en `materias[numMaterias]` usando los setters de `Materia`.
  4. Incrementar `numMaterias`.
- **Función típica:** `void agregarMateria();` o `void agregarMateria(Alumno &a);` (según dónde la coloques).

### B. Registrar calificación

- **Propósito:** Añadir una calificación al arreglo `calificaciones` del alumno.
- **Pasos:**
  1. Verificar que `numCalificaciones < 10`.
  2. Pedir la clave de la materia (o seleccionarla de las ya registradas) y la nota.
  3. Asignar en `calificaciones[numCalificaciones]`.
  4. Incrementar `numCalificaciones`.
- **Validación:** Verificar que la materia exista (opcional pero recomendable). Puedes usar una búsqueda lineal sobre `materias`.
- **Función típica:** `void registrarCalificacion();`

### C. Mostrar historial académico

- **Propósito:** Mostrar todas las materias con sus calificaciones correspondientes.
- **Pasos:**
  1. Recorrer `materias` desde `0` hasta `numMaterias-1`.
  2. Para cada materia, buscar su calificación en `calificaciones` (comparando `claveMateria`).
  3. Mostrar el nombre de la materia y la nota obtenida.
- **Función típica:** `void mostrarHistorial();`

---

## 7. Búsqueda de materias y calificaciones

- **Búsqueda por clave:** Similar a la búsqueda de alumnos por nombre. Recorres el arreglo y comparas `materias[i].getClave() == claveBuscada`.
- **Búsqueda por nombre:** Igual pero comparando `getNombre()`.
- **Uso:** Útil para registrar una calificación (verificar que la materia exista) o para mostrar una materia específica.

**Fragmento suelto (búsqueda):**
```cpp
int buscarMateria(string clave) {
    for (int i = 0; i < numMaterias; i++) {
        if (materias[i].getClave() == clave) {
            return i;
        }
    }
    return -1;
}
```

---

## 8. Menú actualizado

- **Opciones nuevas:**
  - Agregar materia
  - Registrar calificación
  - Mostrar historial académico
- **Opciones existentes:** Registrar alumno, mostrar alumno, calcular promedio, eliminar alumno, modificar alumno, intercambiar alumnos, mostrar todos, salir.
- **Estructura:** El menú puede crecer mucho. Considera agrupar opciones en submenús (por ejemplo, un submenú de "Gestión académica" con materias y calificaciones).

---

## 9. Interacción con encapsulación

- **Acceso a materias y calificaciones:** Como son atributos privados del `Alumno`, necesitas métodos públicos para manipularlos:
  - `void agregarMateria(Materia m);` → añade una materia al arreglo.
  - `Materia getMateria(int i) const;` → devuelve una materia por índice.
  - `int getNumMaterias() const;` → devuelve el contador.
  - Similar para calificaciones.
- **Alternativa:** Si las funciones de gestión están dentro de la clase `Alumno`, pueden acceder directamente a los atributos privados sin necesidad de getters/setters.
- **Recomendación:** Coloca las operaciones de gestión **dentro de la clase `Alumno`** como métodos. Así aprovechas el encapsulamiento y evitas exponer los arreglos internos.

**Fragmento suelto (método dentro de Alumno):**
```cpp
void agregarMateria(Materia m) {
    if (numMaterias < 10) {
        materias[numMaterias] = m;
        numMaterias++;
    }
}
```

---

## 10. Buenas prácticas

- **Una clase, una responsabilidad:** `Materia` representa una asignatura; `Calificacion` representa una nota. No mezcles responsabilidades.
- **Encapsulamiento:** Atributos `private`, métodos `public`.
- **Constructores:** Define constructor vacío y con parámetros para cada clase.
- **Validaciones:** Verifica límites de arreglos (`numMaterias < 10`) y valida datos (nota entre 0 y 10, nombre no vacío).
- **Reutilización:** Si necesitas la misma lógica de búsqueda en varias funciones, extráela a un método auxiliar privado.
- **Documentación:** Explica qué hace cada método y qué parámetros recibe.
- **Evita arreglos paralelos:** Con composición, cada materia y calificación es un objeto, lo cual es más limpio que tener arreglos paralelos de strings y floats.

---

## 11. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Declarar `Materia materias[10];` sin haber definido `Materia` antes** | Define las clases en orden: `Materia`, `Calificacion`, `Alumno`. |
| **Olvidar incrementar `numMaterias` al agregar** | Siempre actualiza el contador después de añadir. |
| **Acceder a `materias[i]` con `i >= numMaterias`** | Recorre solo hasta `numMaterias-1`. |
| **No validar que la materia exista antes de registrar calificación** | Busca la materia por clave antes de añadir la calificación. |
| **Confundir el índice de materia con el de calificación** | Son arreglos independientes; usa claves para relacionarlos. |
| **Saturar la consola con mensajes de constructores** | Comenta los mensajes de traza en clases que se crean muchas veces (como `Materia` y `Calificacion` dentro de `Alumno`). |
| **Olvidar el `;` al final de cada clase** | Cierra cada clase con `};`. |

---

## 12. Resumen de conceptos clave

- **Composición:** Una clase contiene objetos de otra clase como atributos (relación "tiene un").
- **Orden de declaración:** Las clases contenidas deben declararse antes que la contenedora.
- **Construcción/destrucción automática:** Los objetos contenidos se construyen y destruyen junto con el contenedor.
- **Arreglos de objetos:** Permiten almacenar múltiples materias y calificaciones dentro de un alumno.
- **Métodos de gestión:** Agregar materia, registrar calificación, mostrar historial.
- **Encapsulamiento:** Los arreglos internos son `private`; se accede a ellos mediante métodos `public`.
- **Búsqueda:** Localizar materias por clave o nombre para relacionarlas con calificaciones.

---