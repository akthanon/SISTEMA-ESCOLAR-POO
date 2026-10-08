# Actividad 15. Persistencia

## Temas

- fstream
    
- Archivos TXT
    

## Ejercicio

Guardar automáticamente:

```text
alumnos.txt

maestros.txt

administradores.txt
```

Al iniciar el programa:

- cargar datos
    

Al salir:

- guardar datos
    

Agregar opciones:

- Exportar
    
- Importar
    

---
# TEORÍA

---

## 1. ¿Qué es la persistencia?

- **Definición:** Es la capacidad de un programa de **guardar datos** en un medio permanente (como un archivo) para que estén disponibles en futuras ejecuciones, y **recuperarlos** al iniciar.
- **Sin persistencia:** Los datos viven solo en memoria RAM y se pierden al cerrar el programa.
- **Con persistencia:** Los datos se escriben en disco (archivos de texto, binarios, bases de datos) y se leen al volver a abrir el programa.
- **En esta actividad:** Usar archivos de texto plano (`.txt`) para simplicidad y legibilidad humana.

---

## 2. La biblioteca `<fstream>`

Para trabajar con archivos necesitas incluir la biblioteca:

```cpp
#include <fstream>
```

Esta biblioteca proporciona tres clases principales:

| Clase | Propósito | Operación típica |
|-------|-----------|------------------|
| `ofstream` | Escritura (output) | Guardar datos en archivo |
| `ifstream` | Lectura (input) | Leer datos desde archivo |
| `fstream` | Lectura y escritura | Ambos |

- **Modos de apertura:** Se pueden combinar con el operador `|`:
  - `ios::in` → lectura
  - `ios::out` → escritura (sobrescribe por defecto)
  - `ios::app` → añadir al final (append)
  - `ios::trunc` → truncar (borrar contenido) — es el modo por defecto de `out`
  - `ios::binary` → modo binario
- **Para esta actividad:** Usarás `ofstream` para guardar y `ifstream` para leer. El modo por defecto de `ofstream` ya sobrescribe, lo cual está bien si guardas todo de nuevo cada vez.

**Fragmentos sueltos:**
```cpp
ofstream archivo("datos.txt");          // escritura (sobrescribe)
ifstream archivo("datos.txt");          // lectura
ofstream archivo("datos.txt", ios::app); // añadir al final
```

---

## 3. Abrir y cerrar archivos

- **Apertura:** Se puede hacer en el constructor o con el método `.open()`.
- **Verificación:** Siempre verifica que el archivo se abrió correctamente con `.is_open()` o `.fail()`.
- **Cierre:** Se hace con `.close()`. Aunque el destructor cierra automáticamente al salir de ámbito, es buena práctica cerrarlo explícitamente después de usarlo.
- **Importante:** Si el archivo no existe al leer, `ifstream` falla. Verifica antes de intentar leer.

**Fragmento suelto:**
```cpp
ifstream archivo("alumnos.txt");
if (!archivo.is_open()) {
    // el archivo no existe o no se pudo abrir
}
// ... leer ...
archivo.close();
```

---

## 4. Escritura en archivos de texto

- **Operador `<<`:** Funciona igual que con `cout`, pero escribe en el archivo en lugar de la consola.
- **Formato:** Puedes escribir campos separados por espacios, comas, punto y coma, o saltos de línea. Elige un formato consistente y fácil de parsear.
- **Recomendación para esta actividad:** Un registro por línea, con campos separados por un delimitador claro (por ejemplo, `|` o `;`). Evita usar espacios si algún campo puede contenerlos (como nombres completos).
- **Ejemplo de formato conceptual:**
  - Cada línea: `campo1|campo2|campo3|...`
  - El delimitador debe ser un carácter que no aparezca en los datos.

**Fragmento suelto:**
```cpp
ofstream archivo("alumnos.txt");
archivo << nombre << "|" << edad << "|" << matricula << "\n";
archivo.close();
```

- **`"\n"` vs `endl`:** `endl` vacía el búfer y es más lento. Para archivos, `"\n"` es suficiente y más eficiente.
- **Encadenamiento:** Puedes encadenar varios `<<` en una sola línea.

---

## 5. Lectura de archivos de texto

- **Operador `>>`:** Lee palabra por palabra (delimitada por espacios, tabs o saltos de línea).
- **`getline(archivo, variable)`:** Lee una línea completa hasta el salto de línea.
- **`getline(archivo, variable, delimitador)`:** Lee hasta el delimitador especificado (por ejemplo, `'|'`).
- **Estrategia recomendada:**
  1. Leer una línea completa con `getline(archivo, linea)`.
  2. Parsear la línea usando el delimitador (con `stringstream` o `find` + `substr`).
  3. Convertir los campos numéricos con `stoi`, `stof`, `stod`, etc.
- **Ventaja:** Este enfoque es robusto ante campos con espacios (como nombres completos).

**Fragmento suelto (parseo con stringstream):**
```cpp
#include <sstream>
string linea;
while (getline(archivo, linea)) {
    stringstream ss(linea);
    string campo;
    while (getline(ss, campo, '|')) {
        // procesar cada campo
    }
}
```

- **Convertir string a número:**
  - `stoi(string)` → `int`
  - `stof(string)` → `float`
  - `stod(string)` → `double`
- **Convertir número a string:** `to_string(numero)`

---

## 6. Estructura de los archivos

- **`alumnos.txt`:** Un alumno por línea. Campos: nombre, edad, matrícula, y opcionalmente materias y calificaciones (o guardarlas en archivos separados).
- **`maestros.txt`:** Un maestro por línea. Campos: nombre, edad, especialidad, ID.
- **`administradores.txt`:** Un administrador por línea. Campos: nombre, edad, departamento, ID.
- **Consideración sobre composición (Actividad 11):** Si el `Alumno` tiene materias y calificaciones, puedes:
  - Guardarlas en el mismo archivo con un formato más complejo (por ejemplo, varios campos por materia).
  - Guardarlas en archivos separados (`materias.txt`, `calificaciones.txt`) y relacionarlas por matrícula.
  - Simplificar: guardar solo los datos básicos del alumno y no las materias/calificaciones (menos completo pero más sencillo).
- **Recomendación:** Para esta actividad, guarda al menos los datos básicos de cada persona. Si quieres incluir materias y calificaciones, usa un formato claro y consistente.

---

## 7. Carga automática al iniciar

- **Propósito:** Al arrancar el programa, leer los archivos y reconstruir los objetos en memoria.
- **Pasos:**
  1. Abrir el archivo con `ifstream`.
  2. Verificar si existe. Si no existe, no hacer nada (el programa arranca vacío).
  3. Leer línea por línea, parsear cada una y crear el objeto correspondiente.
  4. Añadir el objeto al arreglo (o al arreglo de punteros si usas polimorfismo).
  5. Cerrar el archivo.
- **Momento:** Puedes hacerlo en `main()` antes de mostrar el menú, o en una función `cargarDatos()` llamada al inicio.
- **Con polimorfismo (Actividad 13):** Si usas `Persona* personas[100]`, al leer cada línea debes saber qué tipo crear. Puedes:
  - Usar archivos separados por tipo (uno para alumnos, otro para maestros, otro para administradores).
  - O incluir un campo "tipo" al inicio de cada línea y decidir con un `switch`.

---

## 8. Guardado automático al salir

- **Propósito:** Al terminar el programa, escribir todos los objetos en memoria a los archivos correspondientes.
- **Pasos:**
  1. Abrir el archivo con `ofstream` (modo por defecto, sobrescribe).
  2. Recorrer el arreglo de objetos.
  3. Para cada objeto, escribir sus campos separados por el delimitador y con salto de línea.
  4. Cerrar el archivo.
- **Momento:** Puedes hacerlo antes de `return 0;` en `main()`, o en una función `guardarDatos()` llamada justo antes de salir.
- **Con polimorfismo:** Si tienes un arreglo de punteros a `Persona`, necesitas distinguir el tipo real para saber en qué archivo guardarlo. Opciones:
  - Usar `dynamic_cast` para comprobar el tipo.
  - Guardar un campo "tipo" y usar `switch`.
  - Tener arreglos separados por tipo (más simple pero menos flexible).

---

## 9. Opciones de Exportar e Importar

- **Diferencia con guardar/cargar automático:**
  - **Guardar/Cargar automático:** Usa archivos fijos (`alumnos.txt`, etc.) y ocurre siempre al iniciar/salir.
  - **Exportar/Importar:** El usuario elige el **nombre del archivo** y el momento. Útil para hacer copias de seguridad o transferir datos.
- **Exportar:** Similar a guardar, pero pidiendo al usuario el nombre del archivo destino.
- **Importar:** Similar a cargar, pero pidiendo al usuario el nombre del archivo origen. Considera si importar **reemplaza** o **añade** a los datos existentes.
- **Validaciones:**
  - Verificar que el archivo exista al importar.
  - Verificar que el usuario tenga permisos de escritura al exportar.
  - Manejar errores (archivo corrupto, formato incorrecto).

**Fragmento suelto (pedir nombre de archivo):**
```cpp
string nombreArchivo;
cout << "Nombre del archivo: ";
cin >> nombreArchivo;
ofstream archivo(nombreArchivo);
```

---

## 10. Manejo de errores

- **Archivo no existe al leer:** `ifstream` falla. Verifica con `.is_open()` o `.fail()`.
- **Archivo no se puede crear al escribir:** Verifica permisos y ruta.
- **Formato incorrecto:** Al parsear, pueden fallar las conversiones (`stoi` lanza excepción si el string no es numérico). Usa `try-catch` o validaciones previas.
- **Líneas vacías o incompletas:** Ignóralas o repórtalas.
- **Campos faltantes:** Verifica que la línea tenga el número esperado de campos antes de parsear.

**Fragmento suelto (try-catch):**
```cpp
try {
    int edad = stoi(campo);
} catch (const invalid_argument& e) {
    // el campo no es un número válido
} catch (const out_of_range& e) {
    // el número está fuera del rango de int
}
```

---

## 11. Buenas prácticas

- **Formato consistente:** Define un delimitador y úsalo siempre. Documenta el formato en un comentario o en el README.
- **Un archivo por tipo:** `alumnos.txt`, `maestros.txt`, `administradores.txt`. Facilita la carga y el guardado.
- **Cerrar archivos:** Siempre cierra con `.close()` después de usarlos.
- **Verificar apertura:** Nunca asumas que un archivo se abrió correctamente.
- **Manejo de errores:** Usa `try-catch` para conversiones y verifica estados de los flujos.
- **No guardes datos sensibles en texto plano** si el proyecto crece (contraseñas, información personal). Para esta actividad académica no hay problema.
- **Versionado:** Si el formato cambia, considera incluir un número de versión al inicio del archivo.
- **Respaldo:** Antes de sobrescribir, considera hacer una copia de seguridad (por ejemplo, renombrando el archivo anterior).
- **Rutas:** Usa rutas relativas al ejecutable para portabilidad. Evita rutas absolutas específicas de tu máquina.
- **Documentación:** Explica en el README el formato de cada archivo y cómo se cargan/guardan.

---

## 12. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `#include <fstream>`** | Necesario para usar `ofstream` e `ifstream`. |
| **No verificar si el archivo se abrió** | Usa `.is_open()` antes de leer/escribir. |
| **No cerrar el archivo** | Llama a `.close()` explícitamente. |
| **Usar `>>` para leer nombres con espacios** | Usa `getline` con delimitador. |
| **Mezclar `>>` y `getline` sin limpiar el buffer** | Usa `cin.ignore()` o `archivo.ignore()`. |
| **No convertir strings a números** | Usa `stoi`, `stof`, `stod` según corresponda. |
| **Asumir que el archivo existe** | Verifica con `.is_open()` y maneja el caso de archivo inexistente. |
| **Sobrescribir sin querer** | Usa `ios::app` si quieres añadir; el modo por defecto sobrescribe. |
| **No validar conversiones** | Usa `try-catch` o verifica que el string sea numérico antes de convertir. |
| **Guardar objetos derivados en el archivo equivocado** | Usa archivos separados por tipo o incluye un campo "tipo" y decide con `switch`. |
| **No manejar el polimorfismo al cargar** | Si usas `Persona*`, necesitas saber el tipo real al leer. Usa archivos separados o campo "tipo". |

---

## 13. Resumen de conceptos clave

- **Persistencia:** Guardar y recuperar datos entre ejecuciones.
- **`<fstream>`:** Biblioteca para manejo de archivos.
  - **`ofstream`:** Escritura.
  - **`ifstream`:** Lectura.
- **Modos:** `ios::in`, `ios::out`, `ios::app`, `ios::trunc`, `ios::binary`.
- **Escritura:** Operador `<<`, similar a `cout`.
- **Lectura:** `getline` + parseo con `stringstream` + conversiones (`stoi`, `stof`).
- **Verificación:** `.is_open()`, `.fail()`, `.close()`.
- **Carga automática:** Leer archivos al iniciar el programa.
- **Guardado automático:** Escribir archivos al salir del programa.
- **Exportar/Importar:** Igual que guardar/cargar, pero con nombre de archivo elegido por el usuario.
- **Manejo de errores:** `try-catch`, validaciones, verificación de apertura.
- **Buenas prácticas:** Formato consistente, cerrar archivos, verificar apertura, documentar.

---
