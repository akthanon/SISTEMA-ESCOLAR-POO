# Actividad 2. Primer menú

**Temas**

- Variables
    
- Tipos de datos
    
- cin
    
- cout
    
- Operadores
    

**Ejercicio**

Crear un menú que solicite:

- Nombre
    
- Edad
    
- Carrera
    

y después imprima algo como:

```
Bienvenido Jorge

Edad: 33

Carrera: TI
```

Agregar una opción para calcular el promedio de tres materias.

---
# TEORÍA

---

## 1. Variables en C++

- **Definición:** Una variable es un espacio en memoria con un nombre simbólico que almacena un valor que puede cambiar durante la ejecución.
- **Reglas de nombres (identificadores):**
  - Solo letras, dígitos y guión bajo (`_`).
  - No puede empezar con dígito.
  - Sensible a mayúsculas/minúsculas (`Edad` ≠ `edad`).
  - No puede ser una palabra reservada (`int`, `return`, `if`, etc.).
- **Declaración:** `tipo nombre;`
- **Inicialización (al declarar):** `tipo nombre = valor;` o `tipo nombre(valor);`
- **Asignación:** `nombre = nuevoValor;` (una vez declarada).

---

## 2. Tipos de datos básicos

| Tipo       | Tamaño (aprox.) | Rango / uso                          |
|------------|----------------|--------------------------------------|
| `int`      | 4 bytes        | Números enteros (sin decimales).     |
| `float`    | 4 bytes        | Números con decimales (precisión simple). |
| `double`   | 8 bytes        | Números con decimales (precisión doble). |
| `char`     | 1 byte         | Un solo carácter (entre comillas simples `'a'`). |
| `bool`     | 1 byte         | `true` o `false` (valores lógicos).  |
| `string`   | variable       | Cadena de texto (requiere `#include <string>`). |

**Observaciones:**
- Para usar `string` debes incluir la biblioteca: `#include <string>`.
- El tipo `string` se escribe con comillas dobles: `"Hola"`.

---

## 3. Entrada de datos con `cin`

- **`cin`** (character input) lee desde el teclado.
- Sintaxis: `cin >> variable;`
- Puede encadenarse: `cin >> var1 >> var2;`
- **Problema común:** `cin >>` con `string` se detiene en el primer espacio en blanco (no lee frases completas). Para leer líneas enteras (con espacios) usa `getline(cin, variable);`
- **Mezcla de `cin >>` y `getline`:** Después de un `cin >>`, el salto de línea (`\n`) queda en el buffer. Para limpiarlo, usa `cin.ignore();` antes del `getline`.

**Fragmento suelto (solo sintaxis):**
```cpp
int edad;
cin >> edad;

string nombreCompleto;
cin.ignore();
getline(cin, nombreCompleto);
```

---

## 4. Salida de datos con `cout`

- **`cout`** (character output) envía texto a la consola.
- Sintaxis: `cout << expresión;`
- Puede encadenarse: `cout << "Edad: " << edad << endl;`
- **`endl`** inserta un salto de línea y vacía el búfer de salida.
- También puedes usar `'\n'` para salto de línea (más ligero).

**Fragmento suelto:**
```cpp
cout << "Bienvenido " << nombre;
cout << "\nEdad: " << edad;
```

---

## 5. Operadores en C++

### A. Operadores aritméticos (para calcular el promedio)
- `+` suma
- `-` resta
- `*` multiplicación
- `/` división (cuidado: si ambos operandos son enteros, da división entera)
- `%` módulo (resto de división, solo con enteros)

**Para promedio:**  
Suma las tres calificaciones y divide entre 3.  
Si usas `int`, el resultado será entero (trunca). Para obtener decimales, convierte al menos un operando a `float` o `double` (ej. usando `3.0` en lugar de `3`).

### B. Operadores de asignación
- `=` asigna un valor.
- `+=`, `-=`, `*=`, `/=` (ej. `x += 5` equivale a `x = x + 5`).

### C. Operadores relacionales (devuelven `bool`)
- `==` igual a
- `!=` diferente de
- `<`, `>`, `<=`, `>=`

### D. Operadores lógicos
- `&&` (Y lógico)
- `||` (O lógico)
- `!` (NO lógico)

### E. Operadores de incremento/decremento
- `++` (incrementa en 1), `--` (decrementa en 1). Pueden ser prefijo (`++x`) o sufijo (`x++`).

### F. Precedencia (orden de evaluación)
- `( )` (paréntesis) se evalúa primero.
- Multiplicación, división y módulo antes que suma y resta.
- En caso de duda, usa paréntesis para hacer explícito el orden.

---

## 6. Buenas prácticas para esta actividad

- Incluye siempre `#include <iostream>` y `#include <string>`.
- Usa `using namespace std;` para no escribir `std::` cada vez (o escribe `std::cout` si prefieres).
- Declara las variables al inicio del bloque o justo antes de usarlas.
- Para mostrar el mensaje final, usa `cout` con los valores leídos.
- Para el promedio, declara variables para las tres calificaciones, lee sus valores con `cin`, calcula el promedio y muéstralo con `cout`.

---

## 7. Estructura típica del programa (sin código completo)

La idea es:

1. **Declarar variables** para nombre (string), edad (int), carrera (string), y las tres calificaciones (double o float).
2. **Leer** los datos con `cin` (recuerda el `cin.ignore()` si usas `getline` para el nombre).
3. **Calcular** el promedio: suma de calificaciones dividida entre 3.0.
4. **Mostrar** el mensaje de bienvenida con los datos ingresados y el promedio calculado.

---

## 8. Posibles errores y cómo evitarlos

- **Olvidar `#include <string>`** → error al usar `string`.
- **No limpiar el buffer** después de `cin >>` y antes de `getline` → `getline` leerá un salto de línea vacío.
- **División entera** en promedio: si usas `int` para calificaciones, el promedio será entero. Usa `double` o divide entre `3.0`.
- **No usar `endl` o `'\n'`** → la salida se queda pegada en la misma línea.

---

