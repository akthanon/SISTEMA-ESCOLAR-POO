# Actividad 14. Sobrecarga

## Temas

- Sobrecarga de funciones
    
- Sobrecarga de operadores
    

## Ejercicio

Sobrecargar

```cpp
registrar()
```

para registrar:

- Alumno
    
- Maestro
    
- Administrador
    

Sobrecargar

```cpp
operator<<
```

para imprimir cualquier persona usando:

```cpp
cout << alumno;
```

Sobrecargar `==` para comparar personas por matrícula o ID.

---
# TEORÍA

---

## 1. ¿Qué es la sobrecarga?

- **Definición:** Es la capacidad de tener **varias funciones con el mismo nombre** pero **distinta firma** (diferente número o tipo de parámetros), o **varios operadores con el mismo símbolo** pero distinto comportamiento según los operandos.
- **Requisito:** El compilador debe poder distinguir cuál usar en cada llamada, basándose en los argumentos.
- **Diferencia con sobrescritura (override):**
  - **Sobrecarga (overload):** Mismo nombre, distintos parámetros, **misma clase o ámbito**.
  - **Sobrescritura (override):** Misma firma, **distinta clase** (herencia + virtual).

---

## 2. Sobrecarga de funciones

- **Reglas:**
  - Las funciones deben diferir en el **número** o **tipo** de parámetros.
  - **No** pueden diferir solo en el tipo de retorno.
  - Pueden estar en la misma clase o en el mismo ámbito global.
- **Resolución:** El compilador elige la versión que mejor coincida con los argumentos proporcionados (mediante conversiones implícitas si es necesario).
- **Aplicación en esta actividad:** Sobrecargar `registrar()` para que acepte un `Alumno`, un `Maestro` o un `Administrador`.

**Fragmento suelto (prototipos):**
```cpp
void registrar(Alumno a);
void registrar(Maestro m);
void registrar(Administrador adm);
```

- **Ventaja:** El mismo nombre `registrar` se usa para los tres tipos, lo cual es intuitivo. El compilador decide cuál llamar según el argumento.
- **Consideración:** Si los tres tipos derivan de `Persona`, podrías pensar en sobrecargar solo con `Persona`, pero entonces perderías los atributos específicos de cada derivada. La sobrecarga con los tres tipos concretos es más explícita.
- **Alternativa (polimorfismo):** Si ya tienes un arreglo de punteros a `Persona`, podrías tener una sola función `registrar(Persona* p)` y crear el objeto antes de llamarla. Pero el ejercicio pide específicamente sobrecargar `registrar()`, así que usa las tres versiones.

---

## 3. Sobrecarga de operadores

- **Definición:** Permite redefinir el comportamiento de un operador (`+`, `-`, `==`, `<<`, `>>`, `[]`, etc.) para tipos definidos por el usuario (clases).
- **Sintaxis general:**
  ```cpp
  tipoRetorno operator simbolo(parametros) { ... }
  ```
- **Operadores que se pueden sobrecargar:** La mayoría, excepto `.`, `::`, `?:`, `sizeof`, `.*`.
- **Formas de sobrecarga:**
  - **Como método de la clase:** El operador es un método miembro. El primer operando es el objeto implícito (`this`).
  - **Como función libre (no miembro):** Se define fuera de la clase. Útil cuando el primer operando no es de la clase (por ejemplo, `cout << objeto`).
- **Amistad (`friend`):** Si necesitas acceder a atributos privados desde una función libre, declara esa función como `friend` dentro de la clase.

---

## 4. Sobrecarga de `operator<<` para `cout`

- **Propósito:** Permitir `cout << objeto;` imprimiendo el objeto de forma personalizada.
- **Forma:** **Función libre** (no miembro), porque el primer operando es `ostream` (no el objeto).
- **Firma típica:**
  ```cpp
  ostream& operator<<(ostream& os, const Clase& obj);
  ```
  - Devuelve una referencia a `ostream` para permitir encadenamiento (`cout << a << b`).
  - Recibe `ostream&` (el flujo de salida) y `const Clase&` (el objeto a imprimir).
- **Acceso a atributos privados:** Si la función necesita acceder a atributos `private` o `protected`, declárala como `friend` dentro de la clase.
  ```cpp
  friend ostream& operator<<(ostream& os, const Persona& p);
  ```
- **Polimorfismo con `<<`:** Si quieres que `cout << *persona` llame a la versión correcta según el tipo real, el operador debe delegar en un método virtual (por ejemplo, `mostrarInformacion()`).
  - La función libre `operator<<` no puede ser virtual.
  - Solución: `operator<<` llama a un método virtual `imprimir(os)` que cada clase sobrescribe.
- **Aplicación en esta actividad:** Sobrecargar `<<` para `Persona` (y que funcione para `Alumno`, `Maestro`, `Administrador` gracias al polimorfismo).

**Fragmento suelto (esqueleto):**
```cpp
ostream& operator<<(ostream& os, const Persona& p) {
    p.imprimir(os);   // método virtual
    return os;
}
```

- **`imprimir`** sería un método virtual en `Persona` que cada derivada sobrescribe, recibiendo `ostream&`.

---

## 5. Sobrecarga de `operator==`

- **Propósito:** Comparar dos objetos con `==`. En esta actividad, comparar personas por matrícula o ID.
- **Forma:** Puede ser **método de la clase** o **función libre**.
- **Como método:**
  ```cpp
  bool operator==(const Clase& otro) const;
  ```
  - El primer operando es `*this`; el segundo, `otro`.
  - Devuelve `bool`.
- **Como función libre:**
  ```cpp
  bool operator==(const Clase& a, const Clase& b);
  ```
  - Necesita ser `friend` si accede a atributos privados.
- **Aplicación:** Comparar por `matricula` (Alumno) o por `id` (Maestro, Administrador). Como `Persona` no tiene matrícula, se puede:
  - Definir un atributo común en `Persona` (por ejemplo, `id`) que todas las derivadas hereden.
  - O sobrecargar `==` en cada clase derivada comparando su propio identificador.
  - O definir `==` en `Persona` comparando `nombre` + `edad` (menos específico).
- **Recomendación:** Agrega un atributo `id` (o `clave`) en `Persona` para que todas las derivadas tengan un identificador común. Así `operator==` en `Persona` puede comparar por `id`, y funciona para todas las derivadas.

**Fragmento suelto (esqueleto):**
```cpp
bool operator==(const Persona& otra) const {
    return this->id == otra.id;
}
```

---

## 6. Sobrecarga vs. polimorfismo

- **Sobrecarga:** Se resuelve en **tiempo de compilación** (enlace estático). El compilador decide qué versión usar según los tipos de los argumentos.
- **Polimorfismo:** Se resuelve en **tiempo de ejecución** (enlace dinámico) cuando hay `virtual` y punteros/referencias.
- **En esta actividad:**
  - `registrar()` sobrecargado → resolución en tiempo de compilación.
  - `operator<<` y `operator==` → resolución en tiempo de compilación, pero pueden **delegar** en métodos virtuales para aprovechar el polimorfismo.
- **Combinación:** Es común sobrecargar operadores que internamente llaman a métodos virtuales. Por ejemplo, `operator<<` llama a `imprimir()` (virtual), logrando que `cout << *persona` funcione polimórficamente.

---

## 7. Buenas prácticas

- **Sobrecarga de funciones:**
  - Usa nombres descriptivos; no abuses de la sobrecarga si hace el código confuso.
  - Documenta qué hace cada versión.
  - Evita ambigüedades que el compilador no pueda resolver.
- **Sobrecarga de operadores:**
  - Sobrecarga solo cuando el comportamiento sea **intuitivo** y **consistente** con el operador original.
  - `==` debe devolver `bool` y ser **reflexivo, simétrico y transitivo**.
  - `<<` debe devolver `ostream&` para permitir encadenamiento.
  - Usa `const` correctamente: el objeto a imprimir o comparar no debe modificarse.
  - Si el operador no modifica el objeto, declara el parámetro como `const &` y el método como `const`.
- **Amistad (`friend`):**
  - Úsala solo cuando sea necesaria (acceso a atributos privados desde funciones libres).
  - No abuses; prefiere getters públicos si es posible.
- **Documentación:**
  - Explica qué criterio usa `==` (matrícula, ID, nombre, etc.).
  - Explica el formato de salida de `<<`.
- **Evita duplicación:** Si `operator<<` y `mostrarInformacion()` hacen lo mismo, haz que uno llame al otro.

---

## 8. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Sobrecargar solo por tipo de retorno** | No es válido; debe diferir en parámetros. |
| **`operator<<` como método miembro** | Debe ser función libre (el primer operando es `ostream`). |
| **No devolver `ostream&` en `<<`** | Rompe el encadenamiento (`cout << a << b`). |
| **No usar `const` en `operator<<`** | El objeto no debe modificarse al imprimirlo. |
| **`operator==` no simétrico** | Asegúrate de que `a == b` y `b == a` den lo mismo. |
| **Olvidar `friend` si accedes a privados** | Declara la función como `friend` dentro de la clase. |
| **Confundir sobrecarga con sobrescritura** | Sobrecarga: mismos nombre, distintos parámetros, misma clase. Sobrescritura: misma firma, distinta clase, `virtual`. |
| **`operator<<` que no imprime polimórficamente** | Delega en un método virtual (`imprimir(os)`) para que cada derivada imprima lo suyo. |
| **Ambigüedad en la sobrecarga de `registrar()`** | Si los tipos son convertibles entre sí, el compilador puede no saber cuál usar. Evita conversiones implícitas entre `Alumno`, `Maestro` y `Administrador`. |

---

## 9. Resumen de conceptos clave

- **Sobrecarga de funciones:** Mismo nombre, distintos parámetros, misma clase/ámbito. Resolución en tiempo de compilación.
- **Sobrecarga de operadores:** Redefinir el comportamiento de un operador para una clase.
  - **`operator<<`:** Función libre, devuelve `ostream&`, recibe `ostream&` y `const Clase&`. Puede delegar en un método virtual para polimorfismo.
  - **`operator==`:** Método o función libre, devuelve `bool`, compara por un criterio definido (ID, matrícula, etc.).
- **`friend`:** Permite a funciones libres acceder a miembros privados.
- **Combinación con polimorfismo:** Los operadores sobrecargados pueden llamar a métodos virtuales para comportarse polimórficamente.
- **Buenas prácticas:** Sobrecarga intuitiva, `const` correcto, simetría en `==`, encadenamiento en `<<`.

---