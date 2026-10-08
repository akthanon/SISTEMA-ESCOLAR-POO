# Actividad 13. Polimorfismo

## Temas

- virtual
    
- override
    

## Ejercicio

Crear un método

```cpp
mostrarInformacion()
```

Cada clase mostrará información diferente.

Ejemplo:

Alumno

```text
Alumno
Nombre
Edad
Matrícula
```

Maestro

```text
Maestro
Nombre
Especialidad
```

Administrador

```text
Administrador
Departamento
```

El menú llamará siempre al mismo método.

---
# TEORÍA

---

## 1. ¿Qué es el polimorfismo?

- **Definición:** Es la capacidad de que un mismo método tenga **comportamientos diferentes** según el tipo real del objeto que lo invoca.
- **En C++:** Se logra mediante **funciones virtuales**. Cuando un método se declara `virtual` en la clase base y se sobrescribe en las derivadas, el compilador decide en tiempo de ejecución qué versión ejecutar (esto se llama **enlace dinámico** o **late binding**).
- **Relación con la herencia:** El polimorfismo **necesita herencia**. Sin una jerarquía de clases, no tiene sentido.
- **Ventaja principal:** Permite tratar objetos de distintas clases derivadas de forma uniforme a través de punteros o referencias a la base.

---

## 2. Función virtual (`virtual`)

- **Declaración en la base:**
  ```cpp
  virtual void mostrarInformacion() const;
  ```
- **Características:**
  - Se declara con la palabra clave `virtual` **solo en la clase base**.
  - La clase derivada puede (y debe) sobrescribirla.
  - Si la base declara el método `virtual`, el enlace es dinámico: se ejecuta la versión del **tipo real** del objeto, no la del tipo del puntero/referencia.
  - Si **no** es `virtual`, se ejecuta la versión del **tipo estático** (el tipo del puntero), lo cual es un error común.
- **Regla importante:** Si vas a manejar objetos derivados a través de punteros o referencias a la base, el método debe ser `virtual`.

---

## 3. Sobrescritura (`override`)

- **Declaración en la derivada:**
  ```cpp
  void mostrarInformacion() const override;
  ```
- **`override` (C++11):** Es una palabra clave opcional pero muy recomendable. Le dice al compilador: "este método debe estar sobrescribiendo un método virtual de la base".
  - Si no coincide exactamente con la firma del método de la base (mismo nombre, mismos parámetros, mismo `const`), el compilador da error.
  - Evita errores sutiles como olvidar el `const` o escribir mal el nombre.
- **`virtual` en la derivada:** Es opcional. Una vez que un método es virtual en la base, sigue siendo virtual en todas las derivadas aunque no lo repitas. Puedes ponerlo por claridad, pero no es obligatorio.
- **Firma idéntica:** La firma (nombre, parámetros, `const`) debe coincidir exactamente con la de la base. Si difiere, no es sobrescritura sino ocultamiento, y `override` lo detecta.

---

## 4. Enlace dinámico vs. enlace estático

- **Enlace estático (early binding):** Ocurre cuando el método **no es virtual**. El compilador decide qué función llamar basándose en el **tipo del puntero o referencia**, no en el tipo real del objeto.
- **Enlace dinámico (late binding):** Ocurre cuando el método **es virtual** y se llama a través de un puntero o referencia. El compilador genera código que consulta la tabla virtual del objeto en tiempo de ejecución para decidir qué versión ejecutar.

**Fragmento suelto (diferencia clave):**
```cpp
Persona* p = new Alumno(...);
p->mostrarInformacion();  // si es virtual: llama a Alumno::mostrarInformacion
                          // si NO es virtual: llama a Persona::mostrarInformacion
```

- **Importante:** El enlace dinámico solo funciona con **punteros o referencias**. Si usas un objeto por valor (no puntero ni referencia), se produce **slicing** (el objeto derivado se "corta" y solo se copia la parte de la base).

---

## 5. Método `mostrarInformacion()` en cada clase

- **Clase base `Persona`:**
  - Declara el método como `virtual`.
  - Puede tener una implementación por defecto (mostrar nombre y edad) o ser **virtual puro** (ver sección 6).
- **Clases derivadas:**
  - Sobrescriben el método con `override`.
  - Cada una muestra su información específica.
  - Pueden llamar a la versión de la base para reutilizar la impresión de nombre y edad:
    ```cpp
    void mostrarInformacion() const override {
        Persona::mostrarInformacion();  // imprime nombre y edad
        // luego imprime lo propio
    }
    ```

**Estructura típica de salida por clase:**
- `Alumno`: título "Alumno", nombre, edad, matrícula.
- `Maestro`: título "Maestro", nombre, edad, especialidad.
- `Administrador`: título "Administrador", nombre, edad, departamento.

---

## 6. Función virtual pura (clase abstracta)

- **Definición:** Una función virtual **sin implementación** en la base, marcada con `= 0`.
  ```cpp
  virtual void mostrarInformacion() const = 0;
  ```
- **Consecuencia:** La clase que contiene al menos una función virtual pura se convierte en **clase abstracta**: **no se pueden crear objetos de ella**, solo punteros o referencias.
- **Uso:** Cuando la base no tiene sentido por sí sola (una `Persona` genérica no se registra, solo sus derivadas). Es una buena práctica para forzar a las derivadas a implementar el método.
- **Decisión para esta actividad:**
  - Si quieres que `Persona` sea abstracta → usa `= 0`.
  - Si quieres poder crear `Persona` genérica → dale una implementación por defecto.
  - Para el ejercicio, tiene sentido hacerla **virtual pura** si no vas a registrar personas genéricas.

---

## 7. Uso en el menú

- **Arreglo de punteros a la base:**
  ```cpp
  Persona* personas[100];
  int numPersonas = 0;
  ```
- **Registrar:** Se crea el objeto del tipo específico con `new` y se guarda el puntero en el arreglo.
  ```cpp
  personas[numPersonas] = new Alumno(...);
  numPersonas++;
  ```
- **Mostrar todos:** Un solo bucle que llama al mismo método para todos:
  ```cpp
  for (int i = 0; i < numPersonas; i++) {
      personas[i]->mostrarInformacion();  // polimorfismo en acción
  }
  ```
- **El menú no necesita saber el tipo real** de cada objeto; simplemente llama a `mostrarInformacion()` y el polimorfismo se encarga de ejecutar la versión correcta.
- **Liberar memoria:** Al salir, recorrer el arreglo y hacer `delete personas[i];`. El destructor de la base debe ser **virtual** para que se llame al destructor correcto de cada derivada.

---

## 8. Destructor virtual

- **Regla:** Si vas a usar punteros a la base para manejar objetos derivados, el destructor de la base **debe ser virtual**.
- **Sintaxis:**
  ```cpp
  virtual ~Persona() { /* ... */ }
  ```
- **Consecuencia de no hacerlo:** Al hacer `delete` sobre un puntero a `Persona` que apunta a un `Alumno`, solo se llamaría al destructor de `Persona`, no al de `Alumno`, produciendo fugas de memoria si `Alumno` reservó recursos.
- **Buena práctica:** Si una clase tiene al menos una función virtual, su destructor también debe ser virtual.

---

## 9. Interacción con actividades anteriores

- **Composición (Actividad 11):** `Alumno` sigue teniendo `Materia` y `Calificacion`. Al destruir un `Alumno` (vía `delete` sobre puntero a `Persona`), se destruyen también sus materias y calificaciones.
- **Encapsulación (Actividad 10):** Los atributos siguen siendo `private` o `protected`; el método `mostrarInformacion()` es `public`.
- **Constructores (Actividad 9):** Se siguen llamando en orden (base → derivada) y los destructores en orden inverso (derivada → base).
- **Herencia (Actividad 12):** El polimorfismo es el siguiente paso natural después de la herencia.

---

## 10. Buenas prácticas

- **Marca `virtual` solo en la base.** En las derivadas, usa `override` (y opcionalmente `virtual`, pero no es necesario).
- **Usa `override` siempre.** Detecta errores de firma en tiempo de compilación.
- **Destructor virtual en la base** si hay polimorfismo.
- **Evita llamar a métodos virtuales desde constructores o destructores.** En esos momentos, el objeto aún no está completamente construido o ya se está destruyendo, y el enlace dinámico no funciona como esperas.
- **`const` correcto:** Si el método no modifica el objeto, decláralo `const` en la base y en las derivadas.
- **Documenta el comportamiento esperado** de cada versión de `mostrarInformacion()`.
- **No abuses del polimorfismo:** Si no necesitas tratar objetos de distintas clases de forma uniforme, no hace falta.

---

## 11. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `virtual` en la base** | El enlace es estático; se llama al método de la base, no al de la derivada. |
| **Olvidar `override` en la derivada** | El compilador no verifica la firma; puedes estar ocultando en lugar de sobrescribiendo. |
| **Firma distinta entre base y derivada** | Debe coincidir exactamente (nombre, parámetros, `const`). Usa `override` para detectarlo. |
| **No usar punteros o referencias** | Si usas objetos por valor, se produce slicing y se pierde el polimorfismo. |
| **Destructor de la base no virtual** | Al hacer `delete` sobre puntero a la base, no se llama al destructor de la derivada. |
| **Llamar a método virtual en constructor/destructor** | El enlace dinámico no funciona; se llama a la versión de la clase actual. |
| **Olvidar `= 0` en virtual pura** | La clase no es abstracta y se puede instanciar, lo cual puede no ser deseado. |
| **Confundir sobrecarga con sobrescritura** | Sobrecarga: mismo nombre, distintos parámetros, misma clase. Sobrescritura: misma firma, distinta clase. |

---

## 12. Resumen de conceptos clave

- **Polimorfismo:** Un mismo método, distintos comportamientos según el tipo real del objeto.
- **`virtual`:** Habilita el enlace dinámico. Se declara en la base.
- **`override`:** Verifica que estás sobrescribiendo correctamente. Se usa en las derivadas.
- **Enlace dinámico:** Ocurre con punteros o referencias a la base cuando el método es virtual.
- **Función virtual pura (`= 0`):** Convierte la clase en abstracta; las derivadas deben implementarla.
- **Destructor virtual:** Obligatorio si vas a hacer `delete` sobre punteros a la base.
- **Arreglo de punteros a la base:** Permite almacenar objetos de distintas derivadas y llamarlos uniformemente.
- **`mostrarInformacion()`:** Método virtual que cada clase implementa a su manera.
- **El menú llama siempre al mismo método**, sin preocuparse por el tipo real del objeto.

---
