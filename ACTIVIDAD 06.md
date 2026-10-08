# Actividad 6. Arreglos y búsqueda

**Temas**

- Arrays
    
- string
    

**Ejercicio**

Agregar:

```
Buscar alumno por nombre
```

```
Eliminar alumno
```

```
Modificar alumno
```

---
# TEORÍA

---

## 1. Arreglos de tipo `string`

- **Recordatorio:** Para usar `string` necesitas `#include <string>`.
- Los arreglos de `string` se declaran igual que otros tipos: `string nombres[MAX];`
- **Comparación de strings:** Se usa el operador `==` (no `strcmp` como en C). Ejemplo: `if (nombres[i] == "Ana")`
- **Asignación:** `nombres[i] = "Carlos";` es válido y reemplaza el contenido.

---

## 2. Búsqueda lineal (secuencial)

- **Propósito:** Encontrar la posición (índice) de un elemento dentro del arreglo.
- **Algoritmo:** Recorrer el arreglo desde el inicio hasta el final (o hasta encontrar el elemento).
- **Estructura típica:**
  1. Pedir el nombre a buscar.
  2. Inicializar un índice de resultado (ej. `pos = -1` para indicar "no encontrado").
  3. Recorrer con un bucle (for o while) desde `0` hasta `cantidad-1`.
  4. Comparar cada `nombres[i]` con el nombre buscado.
  5. Si coinciden, guardar `i` en `pos` y salir del bucle (usando `break` o cambiando la condición).
  6. Después del bucle, verificar si `pos != -1` para actuar en consecuencia.

**Fragmentos sueltos:**
```cpp
string buscar;
cin >> buscar;
int pos = -1;
for (int i = 0; i < cantidad; i++) {
    if (nombres[i] == buscar) {
        pos = i;
        break;
    }
}
if (pos != -1) {
    // encontrado, hacer algo con pos
} else {
    // no encontrado
}
```

---

## 3. Buscar y mostrar alumno (opción del menú)

- Similar a la búsqueda: encontrar la posición y luego mostrar todos los datos del alumno en esa posición (edad, promedio, etc.).
- Puedes reutilizar la lógica de búsqueda dentro de una función que devuelva el índice (o -1 si no existe).

---

## 4. Eliminar un alumno

- **Eliminación lógica vs. física:**
  - *Lógica:* Marcar el elemento como "eliminado" (ej. con un arreglo booleano) para ignorarlo en futuras operaciones. No se recomienda para este ejercicio.
  - *Física (desplazamiento):* Mover todos los elementos siguientes una posición hacia la izquierda para "tapar" el hueco. Luego decrementar el contador `cantidad`.
- **Pasos para eliminar físicamente:**
  1. Buscar el índice del alumno (por nombre).
  2. Si no se encuentra, mostrar mensaje de error.
  3. Si se encuentra, **desplazar** todos los elementos desde `pos+1` hasta `cantidad-1` una posición a la izquierda:
     - `nombres[i] = nombres[i+1];`
     - `edades[i] = edades[i+1];`
     - `promedios[i] = promedios[i+1];`
  4. Decrementar `cantidad` en 1.
- **Importante:** El orden de desplazamiento debe ser de izquierda a derecha (desde `pos+1` hasta `cantidad-1`).

**Fragmento suelto de desplazamiento:**
```cpp
for (int i = pos; i < cantidad - 1; i++) {
    nombres[i] = nombres[i+1];
    edades[i] = edades[i+1];
    promedios[i] = promedios[i+1];
}
cantidad--;
```

---

## 5. Modificar un alumno

- **Propósito:** Cambiar los datos de un alumno existente.
- **Pasos:**
  1. Buscar el índice por nombre (igual que antes).
  2. Si se encuentra, pedir los nuevos datos (nombre, edad, promedio, según corresponda) y **asignarlos** directamente en las posiciones del arreglo.
  3. Si no se encuentra, mostrar mensaje de error.
- **Consideración:** Si permites modificar el nombre, asegúrate de que el nuevo nombre no entre en conflicto con otro alumno (opcional, pero buena práctica).

---

## 6. Validaciones y casos especiales

- **Búsqueda sin resultados:** Siempre manejar el caso de "no encontrado" con un mensaje claro.
- **Eliminar cuando el arreglo está vacío:** Verificar que `cantidad > 0` antes de intentar buscar/eliminar.
- **Duplicados:** El ejercicio no especifica qué hacer con nombres repetidos. Puedes buscar solo la primera coincidencia o preguntar al usuario cuál eliminar/modificar (si hay varios). Para simplificar, asume que los nombres son únicos o que solo eliminas/modificas la primera coincidencia.
- **Índices válidos:** Al acceder a los arreglos, siempre usa índices entre `0` y `cantidad-1`.

---

## 7. Funciones recomendadas para esta actividad

Puedes crear funciones específicas para estas nuevas opciones (o ampliar las existentes):

- `int buscarAlumno(string nombre);` → Devuelve el índice o -1.
- `void mostrarAlumnoPorNombre();` → Pide nombre, busca y muestra.
- `void eliminarAlumno();` → Pide nombre, busca y elimina (si existe).
- `void modificarAlumno();` → Pide nombre, busca y permite modificar campos.

Estas funciones pueden usar variables globales (`nombres`, `edades`, `promedios`, `cantidad`) o recibirlas por referencia si prefieres no usar globales.

---

## 8. Manejo de strings adicional

- **Comparación insensible a mayúsculas/minúsculas:** Por defecto, `==` es sensible. Si quieres ignorar diferencias, necesitas convertir ambos a minúsculas (usando `tolower` en un bucle) o usar funciones de la biblioteca `<algorithm>` (como `std::transform`). Para esta actividad, puedes asumir que el usuario escribe exactamente como se registró.
- **Lectura con espacios:** Recuerda usar `getline` para nombres con espacios, y `cin.ignore()` para limpiar el buffer después de `cin >>`.

---

## 9. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Desplazamiento incorrecto al eliminar** | Asegúrate de recorrer desde `pos` hasta `cantidad-2` (o `i < cantidad-1`) y copiar `i+1` a `i`. |
| **Olvidar decrementar `cantidad`** | Siempre actualiza el contador después de eliminar. |
| **No validar que `cantidad > 0`** | Antes de buscar, muestra un mensaje si no hay alumnos registrados. |
| **Modificar sin buscar primero** | Siempre busca el índice antes de asignar nuevos valores. |
| **Confundir posición con nombre** | La búsqueda devuelve índice, luego usas ese índice para acceder a los demás arreglos. |

---

## 10. Resumen de operaciones con arreglos

- **Registrar:** Asignar en la posición `cantidad` e incrementar.
- **Mostrar todos:** Recorrer de `0` a `cantidad-1`.
- **Buscar:** Recorrer y comparar hasta encontrar o terminar.
- **Eliminar:** Desplazar desde `pos+1` hacia la izquierda y decrementar `cantidad`.
- **Modificar:** Buscar y luego reasignar en la posición encontrada.

---



