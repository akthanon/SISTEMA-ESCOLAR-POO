# Actividad 9. Constructores

Agregar:

Constructor vacío.

Constructor con parámetros.

Destructor.

Que al crear un alumno aparezca:

```
Alumno creado
```

y al terminar el programa:

```
Alumno destruido
```

Solo para visualizar el ciclo de vida.

---
# TEORÍA

---

## 1. Ciclo de vida de un objeto

Todo objeto en C++ pasa por tres etapas:

1. **Creación (construcción):** Se reserva memoria y se inicializan los atributos. Aquí interviene el **constructor**.
2. **Uso:** El objeto existe y sus métodos/atributos pueden ser invocados.
3. **Destrucción:** Cuando el objeto deja de existir (sale de ámbito o se libera la memoria), se llama al **destructor**.

---

## 2. Constructores

- **Definición:** Es un método especial que se ejecuta **automáticamente** cuando se crea un objeto.
- **Características:**
  - Tiene el **mismo nombre que la clase**.
  - **No tiene tipo de retorno** (ni siquiera `void`).
  - **No se llama explícitamente**; el compilador lo invoca al crear el objeto.
  - Puede tener múltiples versiones (**sobrecarga**), siempre que difieran en número o tipo de parámetros.

### A. Constructor vacío (por defecto)

- **Propósito:** Inicializar los atributos a valores predeterminados (o dejarlos listos para ser asignados después).
- **Sin parámetros:** Se invoca automáticamente al declarar un objeto sin argumentos.
- **Si no defines ningún constructor**, C++ genera uno por defecto que no inicializa los atributos (quedan con valores basura). Por eso conviene definirlo explícitamente.
- **Fragmento suelto:**
  ```cpp
  Alumno() {
      nombre = "";
      edad = 0;
      promedio = 0.0;
      // aquí puedes imprimir "Alumno creado"
  }
  ```

### B. Constructor con parámetros

- **Propósito:** Inicializar los atributos con valores específicos en el momento de la creación.
- **Se invoca** pasando argumentos al declarar el objeto.
- **Fragmento suelto:**
  ```cpp
  Alumno(string n, int e, float p) {
      nombre = n;
      edad = e;
      promedio = p;
      // aquí también puedes imprimir "Alumno creado"
  }
  ```
- **Uso:** `Alumno a("Ana", 20, 9.5);` crea un alumno con esos valores.

### C. Lista de inicialización (alternativa más eficiente)

- **Propósito:** Inicializar atributos **antes** de que el cuerpo del constructor se ejecute. Es más eficiente, especialmente para objetos (como `string`).
- **Sintaxis:** Se coloca después de los dos puntos `:` tras la lista de parámetros.
- **Fragmento suelto:**
  ```cpp
  Alumno(string n, int e, float p) : nombre(n), edad(e), promedio(p) {
      // cuerpo (puede estar vacío o imprimir "Alumno creado")
  }
  ```
- **Ventaja:** Evita una asignación extra (primero se construye con valor por defecto y luego se asigna). Con la lista, se construye directamente con el valor deseado.

---

## 3. Destructor

- **Definición:** Es un método especial que se ejecuta **automáticamente** cuando un objeto se destruye (sale de ámbito, se libera memoria, o termina el programa para objetos globales/estáticos).
- **Características:**
  - Tiene el **mismo nombre que la clase** pero precedido por una **virgulilla `~`**.
  - **No tiene parámetros** ni tipo de retorno.
  - **No se puede sobrecargar** (solo hay un destructor por clase).
  - Se usa para liberar recursos (memoria dinámica, archivos abiertos, etc.). En esta actividad, solo lo usaremos para imprimir un mensaje.
- **Fragmento suelto:**
  ```cpp
  ~Alumno() {
      // aquí puedes imprimir "Alumno destruido"
  }
  ```
- **Cuándo se llama:**
  - Al salir del bloque `{ }` donde se declaró el objeto local.
  - Al finalizar el programa, para objetos globales o estáticos.
  - Al eliminar un objeto creado con `new` usando `delete`.
  - Para arreglos locales, se llama al destructor de **cada elemento** cuando el arreglo sale de ámbito.

---

## 4. Orden de llamadas en un arreglo de objetos

- Al declarar `Alumno alumnos[10];`:
  - Se llama al **constructor por defecto** (vacío) **10 veces**, una por cada objeto del arreglo.
  - Si defines un constructor con parámetros y **no** defines el vacío, el compilador **no** generará el vacío automáticamente, y `Alumno alumnos[10];` dará error (no hay constructor sin argumentos). Por eso debes definir **ambos**.
- Al salir del ámbito (por ejemplo, al terminar `main`):
  - Se llama al **destructor** de cada elemento, en orden **inverso** al de construcción (el último creado se destruye primero).

**Fragmento suelto (observación del orden):**
- Al ejecutar el programa verás 10 mensajes de "Alumno creado" al inicio y 10 de "Alumno destruido" al final.

---

## 5. Constructores y destructores: resumen de reglas

| Aspecto | Constructor | Destructor |
|---------|-------------|------------|
| Nombre | Igual que la clase | `~` + nombre de la clase |
| Parámetros | Puede tener (sobrecarga) | Ninguno |
| Tipo de retorno | Ninguno | Ninguno |
| Se llama | Al crear el objeto | Al destruir el objeto |
| Sobrecarga | Sí | No |
| Puede ser virtual | Sí | Sí (útil en herencia) |
| Generado por defecto | Sí, si no defines ninguno | Sí, si no defines ninguno |

---

## 6. Constructores y destructores con mensajes de traza

- **Propósito didáctico:** Visualizar el ciclo de vida de los objetos.
- **Recomendación:** Coloca los mensajes (`cout`) al inicio del constructor y del destructor.
- **Cuidado:** Si creas muchos objetos (por ejemplo, 100 alumnos en un arreglo), verás muchos mensajes repetidos. Para esta actividad, puedes reducir el tamaño del arreglo (por ejemplo, a 3) para observar el ciclo sin saturar la consola, o comentar los mensajes cuando ya no los necesites.
- **Fragmento suelto (constructor vacío con mensaje):**
  ```cpp
  Alumno() {
      nombre = "";
      edad = 0;
      promedio = 0.0;
      cout << "Alumno creado" << endl;
  }
  ```

---

## 7. Interacción con la migración de la Actividad 8

- **Si ya tienes un arreglo `Alumno alumnos[100];`**, al añadir los constructores y destructores verás 100 mensajes al inicio y 100 al final. Para evitar esto, puedes:
  - Reducir el tamaño a un número pequeño (ej. 3) solo para probar.
  - Usar un arreglo dinámico (con `new` y `delete`) para controlar cuándo se crean y destruyen los objetos.
  - Aceptar los 100 mensajes y observar el orden.
- **Si usas constructores con parámetros** para crear objetos individuales (por ejemplo, al registrar un alumno), el mensaje "Alumno creado" se imprimirá cada vez que crees uno nuevo. Pero si tu arreglo está declarado como `Alumno alumnos[100];`, esos 100 ya se crearon al inicio (con el constructor vacío). Registrar un alumno en ese arreglo **no crea un nuevo objeto**, solo modifica uno existente. Para ver el mensaje "Alumno creado" al registrar, tendrías que crear un objeto temporal y luego copiarlo al arreglo, o usar un arreglo dinámico.

**Reflexión importante:** El ejercicio dice "que al crear un alumno aparezca 'Alumno creado'". Si usas un arreglo estático, todos los alumnos se crean al inicio, no al registrarlos. Si quieres que aparezca al registrar, necesitas crear el objeto en ese momento (con `new` o con un arreglo dinámico). Decide qué enfoque usar según lo que pida tu profesor.

---

## 8. Buenas prácticas

- **Siempre define el constructor vacío** si vas a declarar arreglos de objetos o si tu clase tiene otros constructores.
- **Usa la lista de inicialización** para atributos que son objetos (como `string`), es más eficiente.
- **No pongas lógica compleja en el destructor:** debe ser rápido y no lanzar excepciones.
- **Libera recursos en el destructor:** si usas memoria dinámica (`new`), libera con `delete`. En esta actividad no es necesario, pero tenlo en mente.
- **Documenta el ciclo de vida:** los mensajes de traza son útiles para depurar, pero recuerda quitarlos o comentarlos en producción.

---

## 9. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar el `~` en el destructor** | El destructor siempre lleva `~` antes del nombre. |
| **Poner parámetros al destructor** | El destructor no acepta parámetros. |
| **Definir solo constructor con parámetros** | Si vas a declarar arreglos, necesitas también el constructor vacío. |
| **Olvidar el `;` al final de la clase** | Cierra con `};`. |
| **Llamar explícitamente al constructor/destructor** | No se llaman manualmente (salvo casos especiales con placement new). |
| **Confundir constructor con método normal** | El constructor no tiene tipo de retorno y se llama igual que la clase. |
| **Esperar que el mensaje "Alumno creado" aparezca al registrar** | Con arreglo estático, los objetos ya se crearon al inicio. Usa arreglo dinámico si quieres ese comportamiento. |

---

## 10. Resumen de conceptos clave

- **Constructor:** Método especial que inicializa el objeto al crearlo.
  - **Vacío:** Sin parámetros, se invoca al declarar sin argumentos.
  - **Con parámetros:** Permite inicializar con valores específicos.
  - **Lista de inicialización:** Forma eficiente de inicializar atributos.
- **Destructor:** Método especial que se ejecuta al destruir el objeto (`~Clase()`).
- **Ciclo de vida:** Construcción → uso → destrucción.
- **Arreglos de objetos:** Se construyen todos al declarar el arreglo y se destruyen todos al salir de ámbito.
- **Mensajes de traza:** Útiles para visualizar el ciclo de vida, pero pueden saturar la consola con arreglos grandes.

---
