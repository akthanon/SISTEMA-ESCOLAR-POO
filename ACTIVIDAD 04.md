# Actividad 4. Múltiples alumnos

**Temas**

- while
    
- for
    
- do while
    

**Ejercicio**

Permitir registrar 10 alumnos usando arreglos.

```
Nombre[10]

Edad[10]

Promedio[10]
```

Agregar la opción

```
Mostrar todos los alumnos
```

---
# TEORÍA

---

## 1. ¿Por qué necesitamos bucles?

Los bucles permiten **repetir un bloque de código** varias veces sin escribirlo múltiples veces. En esta actividad, necesitarás:

- Repetir la lectura de datos para 10 alumnos.
- Repetir la muestra de los 10 alumnos registrados.
- Mantener el menú activo hasta que el usuario decida salir (aunque esto puede ser opcional en esta actividad).

---

## 2. El bucle `while`

- **Propósito:** Ejecuta un bloque **mientras** una condición sea verdadera. La condición se evalúa **antes** de cada iteración. Si es falsa al inicio, no se ejecuta ni una vez.
- **Sintaxis:**
  ```cpp
  while (condicion) {
      // código a repetir
  }
  ```
- **Uso típico:** Cuando no sabes cuántas iteraciones harás (ej. esperar una entrada válida, leer hasta fin de archivo).

**Fragmento suelto (solo sintaxis):**
```cpp
int i = 0;
while (i < 10) {
    // hacer algo
    i++;  // incremento para evitar bucle infinito
}
```

---

## 3. El bucle `for`

- **Propósito:** Ejecuta un bloque un número **determinado** de veces. Agrupa en una sola línea: **inicialización**, **condición** y **actualización** (incremento/decremento).
- **Sintaxis:**
  ```cpp
  for (inicializacion; condicion; actualizacion) {
      // código a repetir
  }
  ```
- **Uso típico:** Cuando sabes exactamente cuántas iteraciones necesitas (ej. recorrer un arreglo de tamaño fijo).

**Fragmento suelto:**
```cpp
for (int i = 0; i < 10; i++) {
    // i toma valores 0, 1, 2, ..., 9
    // usar i como índice
}
```

- Las tres partes son opcionales, pero los `;` son obligatorios. Si omites la condición, el bucle es infinito (a menos que uses `break`).

---

## 4. El bucle `do-while`

- **Propósito:** Similar a `while`, pero la condición se evalúa **después** de ejecutar el bloque. Así que el bloque se ejecuta **al menos una vez**.
- **Sintaxis:**
  ```cpp
  do {
      // código a repetir
  } while (condicion);
  ```
- **Uso típico:** Menús, donde quieres mostrar las opciones al menos una vez antes de preguntar si continuar.

**Fragmento suelto:**
```cpp
do {
    // mostrar menú y leer opción
} while (opcion != 4);
```

---

## 5. Arreglos (arrays) en C++

- **Definición:** Un arreglo es una colección de elementos del **mismo tipo**, almacenados en posiciones contiguas de memoria. Se accede a cada elemento mediante un índice.
- **Declaración:** `tipo nombre[tamaño];` donde `tamaño` debe ser una constante conocida en tiempo de compilación (entero literal o constante `const`).
- **Índices:** Comienzan en **0** y van hasta `tamaño-1`.
- **Inicialización:** Puedes hacerlo al declarar: `int edades[3] = {18, 20, 22};` o `int edades[10] = {0};` (inicializa todos a cero).
- **Acceso:** `nombre[indice]` (ej. `edades[0] = 25; cout << edades[i];`).
- **Riesgo común:** Acceder a un índice fuera del rango (ej. `edades[10]` si el tamaño es 10) produce **comportamiento indefinido** (puede corromper memoria o dar resultados erróneos). Siempre verifica que el índice esté entre `0` y `tamaño-1`.

**Fragmentos sueltos:**
```cpp
const int MAX_ALUMNOS = 10;
string nombres[MAX_ALUMNOS];
int edades[MAX_ALUMNOS];
double promedios[MAX_ALUMNOS];
```

```cpp
nombres[0] = "Ana";   // primer elemento
edades[1] = 20;       // segundo elemento
```

---

## 6. Relación con el ejercicio

- **Registrar 10 alumnos:** Usa un bucle `for` (o `while`) que itere de 0 a 9. En cada iteración, pide nombre, edad y (opcionalmente) promedio, y almacena cada valor en el arreglo correspondiente en la posición `i`.
- **Mostrar todos los alumnos:** Otro bucle `for` que recorra los arreglos y muestre cada campo con `cout`.
- **Estructura del menú:** Puedes usar un bucle `do-while` para mantener el menú activo hasta que el usuario elija salir. Dentro del menú, usa `switch` o `if-else` para manejar las opciones (como en la actividad anterior, pero ahora con la lógica de los bucles y arreglos).
- **Variables compartidas:** Los arreglos y el contador de alumnos registrados deben declararse **fuera** del menú (por ejemplo, al inicio de `main`), para que todas las opciones puedan acceder a ellos.

---

## 7. Consideraciones importantes

- **Tamaño fijo:** El ejercicio pide exactamente 10 alumnos, así que puedes definir el tamaño como una constante (`const int MAX = 10;`).
- **Control de índice:** Si decides permitir registrar menos de 10, necesitas un contador de alumnos registrados (por ejemplo, `int cantidad = 0;`). Al registrar, incrementas `cantidad`; al mostrar, recorres hasta `cantidad` (no hasta `MAX`).
- **Bucles anidados:** No es necesario aquí, pero ten en cuenta que puedes tener bucles dentro de bucles si fuera necesario.
- **Salir del bucle infinito:** Si usas `while(true)`, asegúrate de tener un `break` o una condición de salida para no quedarte atrapado.

---

## 8. Errores comunes y cómo evitarlos

| Error | Solución |
|-------|----------|
| **Acceder a `arreglo[10]`** (índice fuera de rango) | Recuerda que el índice máximo es `tamaño-1`. Usa `<` en la condición del bucle (`i < 10`). |
| **Bucle infinito** | Asegúrate de que la variable de control se actualice correctamente (`i++` en `while`). |
| **No inicializar arreglos** | Los arreglos locales no se inicializan automáticamente. Inicializa al declarar o asigna valores antes de usarlos. |
| **Confundir `=` con `==` en condiciones** | `if (i = 10)` asigna, no compara. Usa `==` para comparar. |
| **Olvidar `#include <string>`** | Necesario si usas `string` para los nombres. |

---

## 9. Buenas prácticas

- Usa nombres descriptivos para los arreglos (ej. `nombresAlumnos`, `edadesAlumnos`).
- Define el tamaño como una constante global o local al inicio (`const int MAX_ALUMNOS = 10;`).
- Comenta el propósito de cada bucle.
- Si usas `do-while` para el menú, coloca la condición al final para que se muestre al menos una vez.
- Siempre usa llaves `{}` aunque el bloque tenga una sola línea, para evitar errores al añadir más código después.

---

