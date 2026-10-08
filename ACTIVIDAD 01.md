# Actividad 1. Preparando el laboratorio

**Objetivo**

- Instalar VSCode.
    
- Instalar compilador de C++.
    
- Crear la estructura del proyecto.
    
- Aprender a compilar y ejecutar.
    

**Ejercicio**

Crear el proyecto:

```python
ProyectoPOO/
│
├── main.cpp
└── README.md
```

Mostrar:

```
Bienvenido al Sistema Escolar UTC
```

---
# TEORÍA

---

## 1. Entornos de Desarrollo (IDE) y Editores de Código

Un **IDE** (Entorno de Desarrollo Integrado) es un software que reúne todas las herramientas necesarias para programar: editor de texto, compilador, depurador y gestor de archivos. 

**Visual Studio Code (VSCode)** no es un IDE pesado, sino un **editor de código fuente** altamente personalizable. Para que funcione como un IDE de C++, necesita extensiones (como la de C/C++ de Microsoft) y un compilador instalado en tu sistema operativo.

---

## 2. El Compilador de C++

El código que escribes (en lenguaje humano, C++) no es entendible por la máquina (ceros y unos). El **compilador** es el programa encargado de traducir tu código fuente a un archivo ejecutable (`.exe` en Windows o sin extensión en Linux/Mac).

Los compiladores más usados son:

- **Windows:** `MinGW` (Minimalist GNU for Windows) que incluye `g++` (el compilador de GNU para C++).
- **Linux:** `g++` (viene instalado por defecto o se instala con `sudo apt install g++`).
- **macOS:** `clang++` (viene con Xcode Command Line Tools).

**Verifica tu compilador:**
Abre una terminal (CMD, PowerShell o Bash) y escribe:
```bash
g++ --version
```
Si aparece la versión, está listo. Si no, debes instalarlo y agregarlo a las variables de entorno (PATH).

---

## 3. Estructura de un Proyecto en C++

Un proyecto no es solo código; es un conjunto de archivos ordenados. Para esta actividad usaremos:

```
ProyectoPOO/
│
├── main.cpp       # Archivo que contiene el código fuente (el programa).
└── README.md      # Archivo de texto (formato Markdown) con la descripción del proyecto.
```

- **`main.cpp`**: Es el archivo principal. Por convención, `main` es el nombre de la función por donde el sistema operativo comienza a ejecutar el programa.
- **`README.md`**: Es la "tarjeta de presentación" del proyecto. Se escribe en formato Markdown (usa `#` para títulos, `*` para listas, etc.) y se visualiza bonito en plataformas como GitHub.

---

## 4. Sintaxis Básica de C++ para tu Programa

Para mostrar el mensaje "Bienvenido al Sistema Escolar UTC", necesitas entender estas líneas:

```cpp
#include <iostream>   // 1. Directiva de preprocesador
using namespace std;  // 2. Espacio de nombres

int main() {          // 3. Función principal
    // 4. Cuerpo del programa
    cout << "Bienvenido al Sistema Escolar UTC" << endl;
    
    return 0;         // 5. Indicador de éxito
}
```

**Explicación teoría:**

1. **`#include <iostream>`**: Inserta el contenido de la biblioteca `iostream` (Input/Output Stream). Necesaria para usar `cout` (salida en consola).
2. **`using namespace std;`**: Permite usar `cout` sin tener que escribir `std::cout` cada vez. `std` es el espacio de nombres estándar de C++.
3. **`int main()`**: Es la función principal. Todo programa en C++ debe tener una función llamada `main`. El `int` significa que devuelve un número entero al sistema operativo.
4. **`cout << "..."`**: `cout` significa "character output". El operador `<<` envía el texto al flujo de salida (la consola). `endl` es un "salto de línea" (enter).
5. **`return 0;`**: Devuelve `0` al sistema operativo para indicar que el programa terminó sin errores.

---

## 5. Proceso de Compilación y Ejecución

Para convertir tu `main.cpp` en un programa que corra, debes pasar por dos fases:

### A. Compilación (Traducción)
El compilador lee el archivo `.cpp` y genera un archivo objeto (`.o` o `.obj`) y luego lo enlaza con las bibliotecas necesarias para crear el ejecutable.

**Comando en terminal:**
```bash
g++ main.cpp -o main.exe   # En Windows
g++ main.cpp -o main       # En Linux/Mac
```
- `g++`: llama al compilador de C++.
- `main.cpp`: archivo fuente.
- `-o main`: indica el nombre del archivo de salida (ejecutable).

### B. Ejecución (Correr el programa)
Una vez generado el ejecutable, lo corres:

- **Windows:** `main.exe` o `.\main.exe`
- **Linux/Mac:** `./main`

---

## 6. Uso de VSCode para esta Actividad

1. **Abrir la carpeta:** En VSCode, ve a `File > Open Folder...` y selecciona la carpeta donde crearás `ProyectoPOO`.
2. **Crear archivos:** Haz clic en el icono de "Nuevo Archivo" y crea `main.cpp` y `README.md`.
3. **Escribir el código:** Copia el código de C++ en `main.cpp`. En `README.md` escribe por ejemplo: `# Proyecto POO - Sistema Escolar UTC`.
4. **Ejecutar desde la terminal integrada:** En VSCode, abre la terminal (`` Ctrl+Ñ `` o `View > Terminal`). Asegúrate de estar en la carpeta `ProyectoPOO` y ejecuta los comandos de compilación y ejecución.
5. **Extensión recomendada:** Instala la extensión **"C/C++" de Microsoft** para obtener resaltado de sintaxis y sugerencias.

---

## Resumen de pasos para tu ejercicio:

| Paso | Acción |
| :--- | :--- |
| 1 | Instalar VSCode y el compilador `g++`. |
| 2 | Crear carpeta `ProyectoPOO`. |
| 3 | Dentro de ella, crear `main.cpp` y `README.md`. |
| 4 | Escribir el programa con `#include`, `main` y `cout`. |
| 5 | Abrir terminal en VSCode y compilar con `g++ main.cpp -o programa`. |
| 6 | Ejecutar con `./programa` (o `programa.exe`). |
| 7 | Verificar que en la consola aparezca el mensaje solicitado. |

---

**Consejo:** Si el sistema no reconoce `g++`, revisa que la carpeta `bin` del compilador esté agregada en las variables de entorno (PATH) de tu sistema. ¡Eso es todo lo que necesitas saber para esta primera actividad!

---

