# Actividad 10. Encapsulación

Cambiar

```cpp
public:
```

por

```cpp
private:
```

Crear

```
setNombre()

getNombre()

setEdad()

getEdad()

setPromedio()

getPromedio()
```

---
# TEORÍA

---

## 1. ¿Qué es la encapsulación?

- **Definición:** Es el principio de la Programación Orientada a Objetos que consiste en **ocultar los detalles internos** de una clase y exponer solo lo necesario mediante una **interfaz pública** controlada.
- **Objetivo:**
  - Proteger los datos de modificaciones no controladas.
  - Permitir validaciones antes de asignar valores.
  - Facilitar el mantenimiento: si cambias la implementación interna, la interfaz pública no cambia.
- **Mecanismo en C++:** Los **modificadores de acceso** (`private`, `public`, `protected`).

---

## 2. Modificadores de acceso (repaso)

| Modificador | Accesible desde... |
|-------------|--------------------|
| `public` | Cualquier parte del programa. |
| `private` | Solo desde dentro de la propia clase (métodos propios). |
| `protected` | Desde la propia clase y clases derivadas (herencia). |

- **Regla de oro:** Los **atributos** deben ser `private`. Los **métodos** que forman la interfaz (getters, setters, operaciones) deben ser `public`.
- **Excepción:** Constantes o métodos auxiliares pueden ser `private` si solo se usan internamente.

---

## 3. Estructura típica de una clase encapsulada

- **Sección `private`:** Contiene los atributos (y posiblemente métodos auxiliares internos).
- **Sección `public`:** Contiene los métodos que el exterior puede usar (getters, setters, constructores, destructor, métodos de operación).

**Fragmento suelto (esqueleto):**
```cpp
class Alumno {
private:
    string nombre;
    int edad;
    float promedio;

public:
    // constructores, destructor, getters, setters, métodos...
};
```

---

## 4. Getters y Setters

### A. Getters (métodos de acceso o "consultores")

- **Propósito:** Devolver el valor de un atributo privado.
- **Convención de nombre:** `get` + NombreDelAtributo (con mayúscula inicial).
- **Tipo de retorno:** El mismo tipo del atributo (o `const &` para objetos grandes como `string`, para evitar copias).
- **Parámetros:** Ninguno.
- **No modifican el objeto**, por lo que es buena práctica declararlos como `const`.

**Fragmento suelto:**
```cpp
string getNombre() const {
    return nombre;
}

int getEdad() const {
    return edad;
}
```

- **El `const` al final** indica que el método no modifica ningún atributo de la clase. Si intentas modificar dentro, el compilador da error.

### B. Setters (métodos de modificación o "mutadores")

- **Propósito:** Asignar un valor a un atributo privado, posiblemente con validaciones.
- **Convención de nombre:** `set` + NombreDelAtributo.
- **Tipo de retorno:** `void` (no devuelven nada).
- **Parámetros:** Un parámetro del mismo tipo del atributo.
- **Pueden incluir validaciones** (por ejemplo, que la edad sea positiva, que el promedio esté entre 0 y 10).

**Fragmento suelto:**
```cpp
void setNombre(string n) {
    nombre = n;
}

void setEdad(int e) {
    if (e >= 0) {      // validación
        edad = e;
    }
}
```

- **Ventaja de las validaciones:** Centralizas las reglas de negocio en un solo lugar. Si cambias la regla, solo modificas el setter.

---

## 5. Interacción con constructores y destructores

- **Constructores:** Aunque los atributos sean `private`, los constructores **sí pueden acceder** a ellos porque son métodos de la propia clase. Puedes seguir usando la lista de inicialización o asignaciones dentro del cuerpo.
- **Destructor:** Igualmente, puede acceder a los atributos privados sin problema.
- **Métodos internos:** Cualquier método de la clase puede leer y escribir los atributos privados directamente (sin necesidad de getters/setters).

**Reflexión:** Dentro de la clase, no necesitas usar `getNombre()` para leer `nombre`; puedes usar `nombre` directamente. Los getters/setters son para **el exterior**.

---

## 6. Cambios necesarios en el resto del programa

- **Antes (atributos públicos):**
  - `alumnos[i].nombre = "Ana";`
  - `cout << alumnos[i].edad;`
- **Ahora (atributos privados):**
  - `alumnos[i].setNombre("Ana");`
  - `cout << alumnos[i].getEdad();`
- **Todas las funciones externas** (registrar, mostrar, buscar, eliminar, modificar, intercambiar) deben actualizarse para usar getters y setters en lugar de acceder directamente a los atributos.
- **La función `intercambiarAlumnos`** ya no puede hacer `temp = a; a = b; b = temp;` directamente si los atributos son privados, porque la asignación de objetos usa el operador `=` que copia atributos privados (esto **sí funciona** porque el operador `=` es generado por el compilador y tiene acceso a los miembros privados). Sin embargo, si quieres hacerlo manualmente, necesitas usar getters y setters:
  - Guardar los valores con getters en variables temporales.
  - Asignar con setters los valores del segundo al primero y los temporales al segundo.

**Fragmento suelto (intercambio manual con getters/setters):**
```cpp
string tempNombre = alumnos[i].getNombre();
int tempEdad = alumnos[i].getEdad();
float tempProm = alumnos[i].getPromedio();

alumnos[i].setNombre(alumnos[j].getNombre());
alumnos[i].setEdad(alumnos[j].getEdad());
alumnos[i].setPromedio(alumnos[j].getPromedio());

alumnos[j].setNombre(tempNombre);
alumnos[j].setEdad(tempEdad);
alumnos[j].setPromedio(tempProm);
```

- **Alternativa más simple:** Como el operador `=` de la clase funciona incluso con atributos privados (porque es un método generado por el compilador que tiene acceso), puedes seguir usando:
  ```cpp
  Alumno temp = alumnos[i];
  alumnos[i] = alumnos[j];
  alumnos[j] = temp;
  ```
  Esto es válido y más limpio. Solo ten en cuenta que el compilador genera este operador automáticamente si no lo defines tú.

---

## 7. Getters que devuelven referencias constantes

- **Para atributos de tipo `string` u objetos grandes**, devolver una copia es costoso. Se puede devolver una **referencia constante**:
  ```cpp
  const string& getNombre() const {
      return nombre;
  }
  ```
- **Ventaja:** Evita copias innecesarias.
- **Desventaja:** El exterior no puede modificar el atributo (porque es `const`), lo cual es correcto.
- **Para tipos simples** (`int`, `float`, `double`, `char`, `bool`), devolver por valor es perfectamente aceptable y más sencillo.

---

## 8. Buenas prácticas

- **Atributos siempre `private`** (o `protected` si hay herencia).
- **Getters y setters `public`** para los atributos que el exterior necesite consultar o modificar.
- **Métodos de solo lectura:** Declara los getters como `const` para que puedan llamarse sobre objetos constantes.
- **Validaciones en setters:** Aprovecha para centralizar reglas de negocio (edad no negativa, promedio entre 0 y 10, nombre no vacío, etc.).
- **No abuses de getters/setters:** Si un atributo solo se usa internamente, no necesita getter ni setter. Solo expón lo necesario.
- **Consistencia:** Usa la misma convención de nombres (`getX`, `setX`) en todas tus clases.
- **Documenta:** Explica qué valida cada setter y qué devuelve cada getter.

---

## 9. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `public:` antes de los getters/setters** | Sin `public:`, son `private` y no se pueden usar desde fuera. |
| **Intentar acceder a un atributo privado desde `main`** | Usa `getX()` o `setX()` en su lugar. |
| **Getter que modifica el objeto** | Declara el getter como `const` para evitar modificaciones accidentales. |
| **Setter sin validación** | Agrega validaciones para mantener la integridad de los datos. |
| **Devolver referencia a un atributo privado no constante** | Devuelve por valor o por `const &` para evitar modificaciones externas. |
| **Olvidar actualizar las funciones externas** | Revisa todas las funciones que accedían directamente a los atributos y cámbialas por getters/setters. |
| **Confundir `const` al final del método con `const` en el parámetro** | El `const` al final (`getX() const`) indica que no modifica el objeto; el `const` en el parámetro indica que no modifica el argumento. |

---

## 10. Resumen de conceptos clave

- **Encapsulación:** Ocultar los atributos (`private`) y exponer una interfaz controlada (`public`).
- **Getter:** Método `public` que devuelve el valor de un atributo. Convención: `getX()`. Suele ser `const`.
- **Setter:** Método `public` que asigna un valor a un atributo. Convención: `setX()`. Puede incluir validaciones.
- **Ventajas:**
  - Protección de datos.
  - Validaciones centralizadas.
  - Flexibilidad para cambiar la implementación interna.
- **Impacto en el resto del programa:** Todas las funciones externas deben usar getters/setters en lugar de acceder directamente a los atributos.
- **Intercambio de objetos:** Puede seguir usándose el operador `=` (generado automáticamente) o hacerse manualmente con getters/setters.

---
