# Actividad 7. Paso por referencia

**Temas**

- &
    
- punteros básicos
    

**Ejercicio**

Modificar todas las funciones para que reciban los arreglos por referencia.

Por ejemplo:

```cpp
registrarAlumno(
    nombres,
    edades,
    promedios
);
```

Agregar una función

```
Intercambiar dos alumnos
```

para entender referencias.

---
# TEORÍA

---

## 1. Repaso: paso por valor vs. paso por referencia

- **Paso por valor:** La función recibe una **copia** del argumento. Modificar el parámetro dentro de la función **no afecta** a la variable original.
- **Paso por referencia (con `&`):** La función recibe un **alias** (otro nombre) de la variable original. Modificar el parámetro **sí afecta** a la variable original.

**Fragmento suelto (diferencia clave):**
```cpp
void porValor(int x)    { x = 100; }   // no modifica el original
void porReferencia(int &x) { x = 100; } // sí modifica el original
```

---

## 2. ¿Por qué pasar arreglos por referencia?

- **Los arreglos en C++ decaen a punteros** cuando se pasan a una función. Es decir, **por defecto ya se pasan "por referencia"** (se pasa la dirección del primer elemento).
- Sin embargo, cuando declaras un parámetro como `tipo nombre[]`, el compilador lo interpreta como `tipo *nombre` (un puntero al primer elemento). Esto significa que **los cambios en los elementos del arreglo sí se reflejan** en el arreglo original.
- **Entonces, ¿por qué usar `&`?** Porque en C++ moderno puedes pasar arreglos por referencia usando plantillas o `std::array`, pero con arreglos estilo C (como los que usas), el `&` no se aplica al arreglo en sí, sino a **elementos individuales** o a **variables sueltas**.
- **Aclaración importante:** Cuando escribes `void funcion(int arr[])`, técnicamente estás pasando un puntero, y los cambios en los elementos se reflejan. El `&` explícito se usa para **variables simples** (como `int`, `string`, `double`) que quieres modificar dentro de la función.

**Fragmento suelto (arreglo como parámetro):**
```cpp
void mostrarTodos(string nombres[], int edades[], double promedios[], int cantidad);
```

- En este caso, `nombres`, `edades` y `promedios` se pasan como punteros al primer elemento. **No necesitas `&`** porque los arreglos ya se comportan como referencias al pasarlos.
- **La variable `cantidad` sí necesita `&`** si quieres modificarla dentro de la función (por ejemplo, al registrar o eliminar).

**Fragmento suelto (cantidad por referencia):**
```cpp
void registrarAlumno(string nombres[], int edades[], double promedios[], int &cantidad);
```

---

## 3. Paso por referencia de variables simples

- Para modificar una variable simple (como `int cantidad`) dentro de una función, debes declarar el parámetro con `&`:
  ```cpp
  void incrementar(int &numero) { numero++; }
  ```
- Al llamar la función, pasas la variable directamente: `incrementar(cantidad);` — no necesitas `&` en la llamada.

---

## 4. Prototipos con arreglos y referencias

- Los prototipos deben coincidir en tipo y número de parámetros con la definición.
- Ejemplo de prototipo que recibe arreglos y una referencia:
  ```cpp
  void registrarAlumno(string nombres[], int edades[], double promedios[], int &cantidad);
  ```
- **Nota:** El tamaño del arreglo no se especifica en el parámetro (o se especifica pero se ignora). Por eso siempre se pasa también la `cantidad` de elementos válidos.

---

## 5. Punteros básicos (introducción)

- **Definición:** Un puntero es una variable que almacena una **dirección de memoria**.
- **Declaración:** `tipo *nombre;` (ej. `int *p;`).
- **Operador `&` (dirección de):** Devuelve la dirección de memoria de una variable. `int x = 5; int *p = &x;`
- **Operador `*` (desreferencia):** Accede al valor almacenado en la dirección apuntada. `cout << *p;` imprime 5.
- **Relación con arreglos:** El nombre de un arreglo es equivalente a un puntero a su primer elemento. `nombres` es lo mismo que `&nombres[0]`.
- **Aritmética de punteros:** `*(nombres + i)` es equivalente a `nombres[i]`.

**Fragmentos sueltos:**
```cpp
int x = 10;
int *p = &x;      // p apunta a x
*p = 20;          // modifica x a través del puntero
cout << x;        // imprime 20

int arr[3] = {1, 2, 3};
int *q = arr;     // q apunta al primer elemento
cout << *(q + 1); // imprime 2 (equivalente a arr[1])
```

- **Importante:** Los punteros son la base del paso por referencia "manual". Cuando pasas un arreglo a una función, en realidad pasas un puntero; por eso los cambios se reflejan.

---

## 6. Función "Intercambiar dos alumnos"

- **Propósito:** Intercambiar **todos los datos** de dos alumnos (nombre, edad, promedio) usando paso por referencia.
- **Estrategia:** Puedes hacerlo de dos formas:
  1. **Intercambiar elemento por elemento** usando variables temporales para cada campo.
  2. **Intercambiar los índices** en el arreglo (más simple si solo quieres reordenar, pero aquí se pide intercambiar los datos).
- **Paso por referencia:** La función debe recibir los elementos individuales por referencia (o el arreglo y los dos índices).
- **Fragmento suelto (intercambio de variables simples):**
  ```cpp
  void intercambiar(int &a, int &b) {
      int temp = a;
      a = b;
      b = temp;
  }
  ```
- Para strings y doubles, el mismo patrón aplica.
- **Firma posible para la función de intercambio de alumnos:**
  ```cpp
  void intercambiarAlumnos(string &nombre1, int &edad1, double &prom1,
                           string &nombre2, int &edad2, double &prom2);
  ```
  O bien, pasar el arreglo completo y los dos índices:
  ```cpp
  void intercambiarAlumnos(string nombres[], int edades[], double promedios[],
                           int i, int j);
  ```
  En este segundo caso, dentro de la función se intercambian `nombres[i]` con `nombres[j]`, etc.

---

## 7. Modificar todas las funciones para recibir arreglos por referencia

- **Firmas típicas (prototipos):**
  ```cpp
  void registrarAlumno(string nombres[], int edades[], double promedios[], int &cantidad);
  void mostrarTodos(string nombres[], int edades[], double promedios[], int cantidad);
  int buscarAlumno(string nombres[], int cantidad, string nombre);
  void eliminarAlumno(string nombres[], int edades[], double promedios[], int &cantidad);
  void modificarAlumno(string nombres[], int edades[], double promedios[], int cantidad);
  void calcularPromedio(double promedios[], int &cantidad); // si aplica
  void intercambiarAlumnos(string nombres[], int edades[], double promedios[], int i, int j);
  ```
- **Observa:**
  - Los arreglos se pasan como `tipo nombre[]` (sin `&` explícito, porque ya son punteros).
  - La variable `cantidad` se pasa como `int &cantidad` si la función la modifica (registrar, eliminar).
  - Si la función solo lee la cantidad, se pasa por valor: `int cantidad`.
- **Llamada desde `main`:** Se pasan los arreglos directamente (sin `&`), y la variable `cantidad` también directamente (el `&` está en la declaración de la función, no en la llamada).

**Fragmento suelto de llamada:**
```cpp

void intercambiarAlumnos(int indice, string nombres[], int edades[], double promedios[], int a, int b)
{

    cout <<"Se intercambiaran "<< a <<" con "<< b <<"\n";

    string nombre_temp;

    int edad_temp;
    double promedio_temp;
            
    nombre_temp=nombres[a];
    edad_temp=edades[a];
    promedio_temp=promedios[a];

    nombres[a]=nombres[b];
    edades[a]=edades[b];
    promedios[a]=promedios[b];

    nombres[b]=nombre_temp;
    edades[b]=edad_temp;
    promedios[b]=promedio_temp;

    cout << "Alumnos Intercambiados\n";
}

intercambiarAlumnos(TOT_ALUMNOS, nombres, edades, promedios, indice1, indice2);
```

---

## 8. Buenas prácticas

- **Sé consistente:** Si una función modifica un arreglo, los parámetros deben reflejarlo (aunque no lleven `&`, es implícito). Documenta qué modifica cada función.
- **Pasa `cantidad` por referencia solo cuando se modifique.** Si solo se lee, pásala por valor.
- **Evita punteros crudos** cuando puedas usar referencias. Los punteros son poderosos pero propensos a errores (fugas de memoria, accesos inválidos).
- **Nombra claramente:** `intercambiarAlumnos` es más descriptivo que `swap`.
- **Valida índices:** En `intercambiarAlumnos`, verifica que `i` y `j` estén dentro del rango válido (`0 <= i < cantidad`, `0 <= j < cantidad`).

---

## 9. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `&` en `cantidad`** al registrar/eliminar | Si la función modifica `cantidad`, debe recibirla por referencia. |
| **Pensar que los arreglos se pasan por valor** | Los arreglos decaen a punteros; los cambios en sus elementos se reflejan. |
| **Usar `&` en la llamada a la función** | El `&` solo va en la declaración/definición, no en la llamada. |
| **Confundir `*` con `&`** | `&` obtiene la dirección; `*` desreferencia (accede al valor). |
| **Intercambiar solo un campo** | Asegúrate de intercambiar **todos** los campos del alumno (nombre, edad, promedio). |
| **Índices fuera de rango en intercambio** | Valida antes de acceder a los elementos. |

---

## 10. Resumen de conceptos clave

- **Paso por referencia (`&`):** Permite modificar la variable original desde la función.
- **Arreglos como parámetros:** Se pasan como punteros; los cambios en sus elementos se reflejan.
- **`cantidad` por referencia:** Necesario si la función registra o elimina alumnos.
- **Punteros básicos:** Variables que almacenan direcciones; `&` para obtener dirección, `*` para acceder al valor.
- **Intercambio:** Usa variables temporales y paso por referencia para intercambiar datos entre dos alumnos.

---

