# Actividad 5. Modularizar

**Temas**

- Funciones
    

**Ejercicio**

Mover el código del menú a funciones.

Ejemplo:

```cpp
registrarAlumno();

mostrarAlumno();

mostrarTodos();

calcularPromedio();

menu();
```

---
# TEORÍA

---

## 1. ¿Qué es una función?

- **Definición:** Una función es un bloque de código con un nombre, que realiza una tarea específica. Se puede llamar (invocar) desde cualquier parte del programa.
- **Propósito:** Modularizar el código, evitar repetición, facilitar mantenimiento y lectura.
- **Estructura básica:**  
  - **Prototipo (declaración):** Dice al compilador "existe una función con este nombre, tipo de retorno y parámetros".  
    `tipo_retorno nombre(tipo1 param1, tipo2 param2);`  
  - **Definición (implementación):** Contiene el cuerpo con las instrucciones.  
    `tipo_retorno nombre(tipo1 param1, tipo2 param2) { /* cuerpo */ }`

---

## 2. Tipos de retorno

- **`void`:** La función **no devuelve ningún valor**. Solo ejecuta acciones (ej. imprimir, modificar variables globales).  
  - Puede tener un `return;` opcional para salir anticipadamente.
- **Tipos concretos (`int`, `double`, `string`, etc.):** La función **devuelve** un valor de ese tipo, que puede ser asignado o usado en expresiones.  
  - Debe tener un `return valor;` al final (o en cualquier lugar donde termine la ejecución).

**Fragmentos sueltos:**
```cpp
void mostrarMenu();                      // no retorna nada
int sumar(int a, int b);                 // retorna un entero
string obtenerNombre();                  // retorna una cadena
```

---

## 3. Parámetros: paso por valor vs. referencia

- **Paso por valor (por defecto):** Se pasa una **copia** del argumento. La función trabaja con esa copia; los cambios no afectan a la variable original.  
  - Ejemplo: `void duplicar(int x) { x = x * 2; }` → no modifica el original.
- **Paso por referencia (usando `&`):** Se pasa la **dirección de memoria** del argumento. La función trabaja directamente con la variable original; cualquier modificación **sí afecta** a la variable externa.  
  - Ejemplo: `void duplicar(int &x) { x = x * 2; }` → modifica la original.
- **¿Cuándo usar cada uno?**  
  - Usa **paso por valor** para parámetros de entrada (solo lectura).  
  - Usa **paso por referencia** cuando necesites modificar la variable original o evitar copias de objetos grandes (como `string` o arreglos).

**Fragmentos sueltos:**
```cpp
void leerDatos(string &nombre, int &edad);   // modifica las variables pasadas
double calcularPromedio(double a, double b, double c); // solo lee valores
```

---

## 4. Ámbito (scope) y variables

- **Variables locales:** Declaradas dentro de una función o bloque `{}`. Solo existen mientras se ejecuta ese bloque. No son accesibles desde otras funciones.
- **Variables globales:** Declaradas fuera de todas las funciones (al inicio del archivo). Son accesibles desde cualquier función, pero su uso excesivo es mala práctica porque dificulta el seguimiento y el mantenimiento.
- **Recomendación:** Para esta actividad, puedes usar variables globales para los arreglos y el contador, simplificando el paso de parámetros. Sin embargo, en proyectos más grandes, es mejor pasarlos como parámetros (por referencia si se modifican, por valor si solo se leen).

**Fragmentos sueltos (globales):**
```cpp
const int MAX = 10;
string nombres[MAX];
int edades[MAX];
double promedios[MAX];
int cantidad = 0;   // cuántos alumnos hay registrados
```

---

## 5. Organización del código

- **Prototipos (declaraciones):** Se colocan al inicio del archivo (después de `#include` y antes de `main`) para que el compilador conozca las funciones antes de usarlas en `main`.
- **Definiciones:** Pueden ir después de `main` o antes. Si van después, los prototipos son obligatorios. Si van antes de `main`, no se necesitan prototipos (pero es común usar prototipos para tener una estructura clara).
- **Estructura recomendada:**
  1. `#include`
  2. `using namespace std;`
  3. Variables globales (opcional).
  4. Prototipos de funciones.
  5. `main()` (que llama a las funciones).
  6. Definiciones de funciones (implementación).

---

## 6. Funciones típicas para este ejercicio

- `void registrarAlumno();` → Pide nombre, edad y (opcionalmente) promedio; almacena en los arreglos globales; incrementa el contador.
- `void mostrarAlumno();` → Pide un índice (o busca por nombre) y muestra los datos de un alumno específico.
- `void mostrarTodos();` → Recorre los arreglos y muestra los datos de todos los alumnos registrados.
- `void calcularPromedio();` → Pide las tres calificaciones y calcula el promedio (puede guardarlo en el arreglo de promedios o mostrarlo en pantalla).
- `void menu();` → Muestra las opciones y gestiona la selección del usuario (con un `do-while` y `switch`). Dentro de este menú se llaman las otras funciones según la opción elegida.

**Fragmentos sueltos de prototipos:**
```cpp
void registrarAlumno();
void mostrarAlumno();
void mostrarTodos();
void calcularPromedio();
void menu();
```

---

## 7. Buenas prácticas

- **Nombres descriptivos:** Usa verbos en infinitivo o imperativo (`registrar`, `mostrar`, `calcular`).
- **Una función, una tarea:** Cada función debe hacer una sola cosa. Si una función crece mucho, divídela en varias más pequeñas.
- **Comentarios:** Explica qué hace cada función, qué parámetros recibe (si los tiene) y qué devuelve.
- **Evita variables globales innecesarias:** Prefiere pasar parámetros, aunque para este ejercicio es aceptable usar globales para simplificar.

---

## 8. Posibles errores y cómo evitarlos

| Error | Solución |
|-------|----------|
| **Olvidar el prototipo** antes de `main` | Siempre declara el prototipo de cada función que definas después de `main`. |
| **Confundir paso por valor y referencia** | Si necesitas modificar la variable original, usa `&`. Si solo lees, no uses `&`. |
| **No declarar variables globales antes de las funciones** | Las globales deben estar definidas antes de cualquier función que las use. |
| **Olvidar el `return` en funciones no `void`** | Agrega `return valor;` con el tipo correcto. |
| **Llamar a una función con argumentos incorrectos** | Verifica que el número y tipo de argumentos coincidan con los parámetros declarados. |
| **Funciones que no modifican globales pero tienen efectos laterales** | Revisa que las funciones solo modifiquen lo que deben modificar. |

---

## 9. Resumen de conceptos clave para esta actividad

- **Modularización:** Divide el programa en funciones con responsabilidades claras.
- **Paso de parámetros:** Decide si pasar por valor o por referencia según necesites modificar o no las variables originales.
- **Ámbito:** Las variables globales son accesibles desde todas las funciones; las locales solo dentro de su función.
- **Prototipos:** Necesarios si defines la función después de `main`.
- **Retorno:** Usa `void` si no devuelves nada; de lo contrario, especifica el tipo y usa `return`.

---
