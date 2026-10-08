# Actividad 3. Menús

**Temas**

- if
    
- else
    
- switch
    

**Ejercicio**

Convertir el programa anterior en un menú.

```
===== MENU =====

1 Registrar alumno

2 Mostrar alumno

3 Calcular promedio

4 Salir
```

---
# TEORÍA

---

## 1. La estructura `if`

- **Propósito:** Ejecutar un bloque de código **solo si** una condición (expresión booleana) es verdadera.
- **Sintaxis básica:**
  ```cpp
  if (condicion) {
      // código que se ejecuta si condicion es true
  }
  ```
- La condición puede ser cualquier expresión que devuelva `bool` o un valor convertible a `bool` (0 = false, distinto de 0 = true).
- Si el bloque tiene una sola línea, las llaves `{ }` son opcionales (pero se recomienda usarlas siempre para legibilidad).

---

## 2. La estructura `if-else`

- **Propósito:** Elegir entre dos caminos: uno para cuando la condición es verdadera y otro para cuando es falsa.
- **Sintaxis:**
  ```cpp
  if (condicion) {
      // código si es true
  } else {
      // código si es false
  }
  ```
- Solo se ejecuta **uno** de los dos bloques.

---

## 3. Encadenamiento `else if`

- **Propósito:** Evaluar múltiples condiciones en secuencia.
- **Sintaxis:**
  ```cpp
  if (cond1) {
      // ...
  } else if (cond2) {
      // ...
  } else if (cond3) {
      // ...
  } else {
      // (opcional) si ninguna condicion anterior fue verdadera
  }
  ```
- Las condiciones se evalúan en orden; al encontrar la primera verdadera, se ejecuta su bloque y se sale del resto.

---

## 4. La estructura `switch`

- **Propósito:** Seleccionar entre múltiples opciones basadas en el valor de una **expresión entera** (o de tipo `char`, `enum`, o `int`). No funciona con `float`, `double` ni `string`.
- **Sintaxis:**
  ```cpp
  switch (expresion) {
      case valor1:
          // código
          break;
      case valor2:
          // código
          break;
      // ...
      default:
          // código opcional si ningún case coincide
  }
  ```
- **Reglas importantes:**
  - `expresion` debe ser de tipo entero o `char` (o enumerado).
  - Cada `case` debe tener un valor constante (literal o constante `const`).
  - **`break`** es obligatorio al final de cada `case` para evitar que la ejecución "caiga" al siguiente caso (fall-through). Sin `break`, se ejecutan los casos siguientes hasta encontrar un `break` o el final del `switch`.
  - `default` es opcional y se ejecuta si ningún `case` coincide.

---

## 5. Comparación `if-else` vs `switch`

| `if-else` | `switch` |
|-----------|----------|
| Útil para condiciones complejas (rangos, operadores lógicos, comparaciones no exactas). | Útil para igualdad exacta contra varios valores discretos. |
| Puede evaluar cualquier tipo (incluyendo `bool`, `string`, `float`). | Solo evalúa tipos enteros o `char`. |
| Puede anidarse sin restricciones. | No se pueden anidar fácilmente (aunque se puede). |
| Más flexible pero puede ser menos legible con muchas opciones. | Más legible cuando hay muchas opciones fijas. |

---

## 6. Expresiones booleanas y operadores de comparación

- Las condiciones se construyen con:
  - **Relacionales:** `==`, `!=`, `<`, `>`, `<=`, `>=`.
  - **Lógicos:** `&&` (Y), `||` (O), `!` (NO).
- Ejemplo de condición compuesta:
  ```cpp
  if (edad >= 18 && edad <= 65) { ... }
  ```
- Cuidado con confundir `=` (asignación) con `==` (comparación). El compilador puede aceptar `if (x = 5)` pero siempre será verdadero (asigna 5 y evalúa a 5 → true), un error lógico.

---

## 7. Ámbito de variables (scope)

- Las variables declaradas **dentro** de un bloque `{ }` solo existen dentro de ese bloque.
- No se puede usar una variable declarada dentro de un `if` fuera de él.
- Para compartir una variable entre diferentes ramas, declárala **antes** del `if`/`switch`.

**Ejemplo (solo fragmento conceptual):**
```cpp
int opcion;
cin >> opcion;

if (opcion == 1) {
    string nombre;  // solo visible dentro de este if
    cin >> nombre;
}
// aqui 'nombre' NO es accesible
```

---

## 8. Buenas prácticas para el menú

- Usa una variable entera (ej. `opcion`) para almacenar la elección del usuario.
- Puedes usar `if-else if` o `switch` para manejar las opciones.
- Para opciones numéricas (1, 2, 3, 4), `switch` es muy adecuado.
- No olvides incluir un `default` o `else` para manejar entradas inválidas (mostrar un mensaje de error).
- Es común envolver el menú en un bucle (que verás en la siguiente actividad), pero para esta actividad no es necesario; solo se pide la estructura condicional.

---

## 9. Casos de uso típicos en esta actividad

- **Registrar alumno:** pedir nombre, edad, carrera (con `cin` y `getline`).
- **Mostrar alumno:** mostrar los datos previamente guardados (necesitas variables que persistan; decláralas antes del menú).
- **Calcular promedio:** pedir tres calificaciones, calcular y mostrar.
- **Salir:** finalizar el programa con `return 0;` o simplemente terminar.

La lógica del menú puede implementarse con un `switch` donde cada `case` corresponda a una opción. Recuerda que las variables que almacenan los datos del alumno deben declararse **fuera** del `switch` para que estén disponibles en todas las opciones.

---
