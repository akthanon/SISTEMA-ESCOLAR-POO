# Actividad 42. Refactorización general

## Temas

- Código limpio
    
- Mantenimiento
    
- Arquitectura
    

## Ejercicio

Revisar todo el proyecto construido durante el curso.

Realizar mejoras:

- Eliminar código repetido.
    
- Cambiar nombres poco claros.
    
- Separar responsabilidades.
    
- Mejorar estructura de carpetas.
    
- Revisar clases demasiado grandes.
    
- Agregar validaciones.
    

Comparar:

Versión inicial:

```text
servidor.cpp
```

Versión final:

```text
Proyecto/
│
├── src/
│   ├── Servidor.cpp
│   ├── Usuario.cpp
│   ├── BaseDatosTXT.cpp
│   ├── Router.cpp
│   └── HTTP.cpp
│
├── include/
│
├── public/
│
└── data/
```

El objetivo no es agregar funcionalidades, sino mejorar el diseño.

---
# TEORÍA

---

## 1. ¿Qué es refactorizar?

- **Definición:** Modificar la **estructura interna** del código **sin cambiar su comportamiento externo**.
- **Objetivo:** Mejorar legibilidad, mantenibilidad, testabilidad y extensibilidad.
- **Regla de oro:** Después de refactorizar, el sistema hace **exactamente lo mismo** que antes. Si algo cambia de comportamiento, no fue refactorización, fue una modificación.
- **Por qué hacerlo:** El código se escribe una vez y se lee muchas. Un código limpio reduce el coste de mantenimiento a largo plazo.
- **Cuándo hacerlo:**
  - Cuando cuesta entender una función o clase.
  - Cuando hay que tocar varias partes para hacer un cambio pequeño.
  - Cuando hay duplicación evidente.
  - Antes de añadir una funcionalidad nueva (para tener una base sana).

---

## 2. Código limpio: principios fundamentales

### A. Nombres significativos
- Los nombres deben **revelar la intención**.
- **Mal:** `d`, `x2`, `data`, `temp`, `func1`, `Manager`.
- **Bien:** `peticionHTTP`, `usuarioAutenticado`, `calcularPromedioFinal`, `obtenerMIME`.
- **Convenciones consistentes:**
  - Clases: `PascalCase` (`Servidor`, `BaseDatosTXT`).
  - Funciones y variables: `camelCase` (`procesarGET`, `rutaPublic`).
  - Constantes: `MAYUSCULAS_CON_GUION_BAJO` o `kPascalCase`.
  - Archivos: coincidir con la clase principal que contienen.

### B. Funciones pequeñas
- **Regla:** una función, una responsabilidad.
- Idealmente menos de **20-30 líneas**.
- Si la función tiene más de **3-4 parámetros**, probablemente debería recibir un struct o dividirse.
- Si el nombre incluye "y" (`leerYParsear`), son dos funciones.

### C. Comentarios útiles
- Los comentarios explican **por qué**, no **qué**.
- **Mal:** `// incrementa i` antes de `i++`.
- **Bien:** `// se usa 3.0 para forzar división decimal`.
- Prefiere código autoexplicativo a comentarios que compensen malos nombres.
- No dejes código comentado. Usa git para eso.

### D. Formato consistente
- Indentación uniforme (2 o 4 espacios, nunca mezclar).
- Líneas no muy largas (80-120 caracteres).
- Líneas en blanco para separar bloques lógicos.
- Llaves `{ }` en el mismo estilo en todo el proyecto.
- Un solo estilo de comentarios.

### E. Evitar la duplicación (DRY)
- **DRY:** Don't Repeat Yourself.
- Si el mismo código aparece **dos veces**, extráelo a una función.
- Si el mismo bloque lógico aparece en varias clases, considera una clase base o una función auxiliar.
- **Cuidado:** No caigas en sobre-abstracción. Tres líneas repetidas no siempre justifican una función.

### F. Manejo de errores consistente
- Elige un patrón y úsalo en todo el proyecto:
  - Códigos de retorno (`bool` + parámetro de salida).
  - Excepciones (`try-catch`).
  - Códigos de error enum.
- No mezcles patrones sin criterio.

---

## 3. Eliminar código repetido

### A. Identificar duplicación
- Busca bloques idénticos o muy similares.
- Busca validaciones repetidas.
- Busca construcciones de respuesta HTTP repetidas.
- Busca parseos de JSON repetidos.

### B. Estrategias
- **Extraer función:** El mismo bloque → una función con parámetros.
- **Extraer clase:** Varios métodos con lógica compartida → una clase auxiliar.
- **Clase base:** Varias clases con métodos idénticos → herencia.
- **Plantillas (templates):** Cuando el tipo varía pero la lógica es idéntica.
- **Constantes:** Valores mágicos repetidos → constantes con nombre.

**Fragmento suelto (idea de extracción):**
```cpp
// Antes: repetido en 5 handlers
// Después:
RespuestaHTTP responderError(int codigo, const string& mensaje) {
    return RespuestaHTTP(codigo, "application/json", armarJSONError(mensaje));
}
```

- **Ventaja:** Un solo lugar para cambiar el formato de errores.

---

## 4. Cambiar nombres poco claros

### A. Señales de malos nombres
- Nombres de una sola letra fuera de bucles cortos.
- Abreviaciones no estándar (`usr`, `cfg`, `hdl`).
- Nombres genéricos (`data`, `info`, `obj`, `item`).
- Nombres que no describen qué hace la función (`procesar`, `manejar`).
- Nombres que no describen qué contiene una variable (`temp`, `aux`).

### B. Proceso de renombrado
1. Identifica el propósito real del elemento.
2. Elige un nombre que revele la intención.
3. Renombra en **todos** los lugares (IDE o buscar-y-reemplazar).
4. Compila y prueba.
5. Commit.

### C. Reglas prácticas
- Verbos para funciones (`calcular`, `obtener`, `procesar`, `validar`).
- Sustantivos para variables y clases (`usuario`, `servidor`, `peticion`).
- Booleanos con prefijo `es`/`tiene`/`puede` (`esValido`, `tieneSesion`, `puedeAcceder`).
- Evita nombres que empiecen con `get` si no son getters reales.

---

## 5. Separar responsabilidades

### A. Principio de Responsabilidad Única (SRP)
- Cada función, clase y archivo debe tener **una única razón para cambiar**.
- Si una clase mezcla HTTP con SQL, son dos responsabilidades.

### B. Señales de mezcla
- Una clase con métodos muy distintos entre sí.
- Un método que hace varias cosas no relacionadas.
- Un archivo que contiene lógica de red, persistencia y presentación.

### C. Cómo separar
- Identifica las **capas** del sistema:
  - **Red:** sockets, lectura/escritura de bytes.
  - **HTTP:** parseo de petición, construcción de respuesta, MIME.
  - **Enrutamiento:** decidir qué handler corresponde.
  - **Aplicación:** handlers que implementan la lógica.
  - **Persistencia:** leer/escribir archivos o BD.
  - **Modelo:** clases de datos (`Usuario`, `Alumno`).
- Cada capa vive en su propio conjunto de funciones o clases.
- Las capas se comunican por interfaces claras.

### D. Ventajas
- Cambios localizados.
- Testeable por capas.
- Reutilizable (por ejemplo, la capa HTTP con otro servidor).

---

## 6. Estructura de carpetas

### A. Antes
```
Proyecto/
└── servidor.cpp   ← 1500+ líneas con todo
```

### B. Después
```
Proyecto/
├── src/
│   ├── Servidor.cpp
│   ├── Usuario.cpp
│   ├── BaseDatosTXT.cpp
│   ├── Router.cpp
│   └── HTTP.cpp
│
├── include/
│   ├── Servidor.h
│   ├── Usuario.h
│   ├── BaseDatosTXT.h
│   ├── Router.h
│   └── HTTP.h
│
├── public/
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── img/
│
├── data/
│   ├── usuarios.txt
│   └── uploads/
│
├── Makefile o CMakeLists.txt
└── README.md
```

### C. Convenciones
- **`src/`:** implementaciones (`.cpp`).
- **`include/`:** cabeceras (`.h`).
- **`public/`:** archivos estáticos servidos al cliente.
- **`data/`:** datos internos del backend (no servidos).
- **`tests/`:** pruebas (si existen).
- **`docs/`:** documentación (si existe).
- **`Makefile` o `CMakeLists.txt`:** compilación automatizada.
- **`README.md`:** descripción, instrucciones, endpoints.

### D. Reglas para archivos `.h` y `.cpp`
- Un `.h` por clase o módulo.
- El `.h` contiene la declaración (prototipos, structs, clases).
- El `.cpp` contiene la implementación (definiciones).
- **Guardas de inclusión** (`#pragma once` o `#ifndef`) en cada `.h`.
- Incluir solo lo necesario (`#include` mínimos).
- Forward declarations cuando sea posible (reduce dependencias).

---

## 7. Separar en múltiples archivos

### A. Por qué
- Un archivo de 1500 líneas es difícil de navegar.
- Compilación parcial: cambiar un archivo no recompila todo.
- Trabajo en equipo: menos conflictos de merge.
- Reutilización: módulos independientes.

### B. Cómo dividir
- **Por clase:** un `.h/.cpp` por clase principal.
- **Por capa:** un archivo por capa (por ejemplo, `HTTP.cpp` agrupa todo lo relacionado con HTTP).
- **Por módulo:** agrupa funciones afines (`Utils.cpp` para utilidades).
- **Regla práctica:** cuando un archivo pasa de **500-800 líneas**, considera dividirlo.

### C. Ejemplo de organización
- `Usuario.h/.cpp` → clase `Usuario` y derivadas.
- `BaseDatosTXT.h/.cpp` → persistencia en TXT.
- `HTTP.h/.cpp` → parseo de petición y construcción de respuesta.
- `Router.h/.cpp` → enrutamiento.
- `Servidor.h/.cpp` → clase principal que orquesta.
- `main.cpp` → solo instancia y arranca.

### D. Dependencias
- `Servidor` depende de `Router`, `HTTP`, `BaseDatosTXT`.
- `Router` depende de handlers (en `Servidor`).
- `BaseDatosTXT` depende de `Usuario`.
- **Regla:** las dependencias deben ir en una sola dirección. Sin ciclos.

---

## 8. Revisar clases demasiado grandes

### A. Señales de clase "Dios" (God Object)
- Más de 500 líneas.
- Muchos métodos no relacionados entre sí.
- Nombre genérico (`Manager`, `Controller`, `Helper`).
- Métodos que acceden a muchas partes del sistema.

### B. Cómo dividir
- **Por responsabilidad:** cada clase, una tarea.
- **Por entidad:** una clase por concepto del dominio.
- **Por capa:** una clase para red, otra para HTTP, otra para datos.

### C. Refactorización típica de `Servidor`
- El `Servidor` original probablemente tiene:
  - Inicialización de sockets.
  - Bucle de aceptación.
  - Parseo HTTP.
  - Enrutamiento.
  - Handlers de cada endpoint.
  - Persistencia.
  - Construcción de respuestas.
- **Dividir:**
  - `Servidor` → solo orquesta: sockets, aceptación, delegar al `Router`.
  - `Router` → decide qué handler corresponde.
  - `HTTP` → parseo y construcción de respuestas.
  - `BaseDatosTXT` → persistencia.
  - Handlers en clases específicas (`HandlersUsuarios`, `HandlersArchivos`) o como métodos de `Servidor`.

### D. Ejemplo conceptual
- `Servidor`:
  - `iniciar()`, `ejecutar()`, `detener()`.
  - Atributos: socket, puerto, `Router`.
- `Router`:
  - `despachar(peticion, cliente)`.
  - Decide entre API y archivos.
- `HTTP`:
  - `parsear(crudo)`, `serializar(respuesta)`.
  - `obtenerMIME(extension)`.

---

## 9. Agregar validaciones

### A. Dónde validar
- **Cliente:** UX inmediata (campos requeridos, formato).
- **Servidor:** seguridad e integridad. **Obligatorio.**
- **Base de datos:** constraints (PRIMARY KEY, NOT NULL, CHECK).
- **Capa de aplicación:** reglas de negocio (matrícula única, rol válido).

### B. Qué validar
- **Entrada:** tipos, longitudes, formatos, rangos.
- **Existencia:** ¿el recurso existe?
- **Unicidad:** ¿ya hay un usuario con ese nombre?
- **Permisos:** ¿el usuario puede hacer esto?
- **Estado:** ¿la operación es válida en este momento?
- **Integridad:** ¿los datos relacionados existen?

### C. Patrón de validación
- Función de validación que devuelve `bool` o un struct con error.
- Mensaje de error específico para el cliente.
- **Nunca** confíes en el cliente. Aunque valide, el servidor revalida.

**Fragmento suelto (idea):**
```cpp
bool esNombreValido(const string& n) {
    return !n.empty() && n.size() <= 100;
}
```

- **Ventaja:** Validación centralizada. Cambias la regla en un solo lugar.

---

## 10. Arquitectura: separación en capas

### A. Capas típicas
```
┌───────────────────────────────┐
│         Presentación          │  ← HTML/CSS/JS (frontend)
├───────────────────────────────┤
│            API                │  ← endpoints HTTP (JSON)
├───────────────────────────────┤
│         Aplicación            │  ← handlers, lógica de negocio
├───────────────────────────────┤
│          Dominio              │  ← clases de datos (Usuario, Alumno)
├───────────────────────────────┤
│         Persistencia          │  ← BaseDatosTXT, BaseDatosSQL
└───────────────────────────────┘
```

- **Presentación:** el navegador.
- **API:** contrato entre frontend y backend.
- **Aplicación:** handlers que reciben peticiones y orquestan.
- **Dominio:** clases de datos con lógica básica (validaciones).
- **Persistencia:** lectura/escritura en disco o BD.

### B. Reglas entre capas
- Cada capa solo habla con la inmediatamente inferior.
- No saltes capas.
- Las capas inferiores no conocen a las superiores.
- La comunicación es por interfaces bien definidas.

### C. Ejemplo en tu proyecto
- `Servidor` (red + HTTP) → `Router` → handlers → `BaseDatosTXT` → `Usuario`.
- `HTTP` es una utilidad transversal usada por `Servidor` y por el `Router`.

---

## 11. Herramientas de calidad

### A. Compilador
- `-Wall -Wextra -Wpedantic` → avisos que detectan problemas.
- `-Werror` → convierte avisos en errores. Estricto pero útil.

### B. Análisis estático
- `cppcheck` → detecta errores comunes sin ejecutar.
- `clang-tidy` → sugerencias de estilo y buenas prácticas.
- `sonar` → análisis profundo en proyectos grandes.

### C. Formateo automático
- `clang-format` → formatea según un archivo `.clang-format`.
- **Ventaja:** estilo consistente sin discusiones.

### D. Pruebas automatizadas
- Frameworks: `Catch2`, `GoogleTest`.
- Pruebas unitarias: cada clase y función por separado.
- Pruebas de integración: probar flujos completos.
- **Para esta actividad:** No es obligatorio, pero menciónalo como práctica.

### E. Control de versiones
- Git es imprescindible.
- Commits pequeños, frecuentes y con mensajes claros.
- Ramas para features; merge a main cuando funcione.
- **Durante la refactorización:** commits después de cada paso que funcione.

---

## 12. Proceso de refactorización seguro

### A. Antes de empezar
1. **Commit** del estado actual (punto de retorno).
2. **Tener pruebas** (manuales o automatizadas) que verifiquen el comportamiento.
3. **Planificar** qué se va a cambiar y en qué orden.

### B. Durante
1. **Un cambio a la vez.** No todo de golpe.
2. **Compilar y probar** después de cada cambio.
3. **Commit** cuando algo funciona.
4. **No añadir funcionalidad nueva.** Solo reorganizar.

### C. Después
1. **Verificar** que el comportamiento es idéntico.
2. **Actualizar documentación** si es necesario.
3. **Commit final.**

### D. Reglas
- Si algo se rompe, **revertir** al último commit bueno.
- No acumular cambios sin compilar.
- Refactorizar con calma; no hay prisa.

---

## 13. Refactorizaciones específicas para tu proyecto

### A. Extraer HTTP
- Todo el parseo de petición → `HTTP::parsear`.
- Toda la construcción de respuesta → `HTTP::serializar`.
- Tabla de MIME → `HTTP::obtenerMIME`.
- **Ventaja:** La capa HTTP es reutilizable.

### B. Extraer Router
- Toda la decisión "¿qué handler llamo?" → `Router::despachar`.
- Cada ruta mapea a una función o método.
- **Ventaja:** El enrutamiento centralizado. Añadir rutas es trivial.

### C. Extraer BaseDatos
- Todo el acceso a archivos de datos → `BaseDatosTXT` o `BaseDatosSQL`.
- Operaciones CRUD abstractas.
- **Ventaja:** Cambiar de TXT a SQL es reescribir una clase.

### D. Extraer Usuario
- Clase base `Usuario` con atributos comunes.
- Derivadas `Alumno`, `Maestro`, `Administrador`.
- Serialización JSON → método de la clase.
- **Ventaja:** Cohesión de datos. Fácil de extender.

### E. Servidor delgado
- `Servidor` solo sabe de red y delegación.
- Todo lo demás vive en otras clases.
- **Ventaja:** El bucle principal es claro y corto.

### F. `main` mínimo
- 3-5 líneas. Crear el servidor y arrancarlo.
- **Ventaja:** Se entiende de un vistazo.

---

## 14. Métricas de calidad

- **Líneas por archivo:** idealmente < 500.
- **Líneas por función:** idealmente < 30.
- **Líneas por clase:** idealmente < 300.
- **Parámetros por función:** idealmente ≤ 3-4.
- **Profundidad de anidamiento:** idealmente ≤ 3 niveles.
- **Complejidad ciclomática:** bajo (pocos `if`/`switch` por función).

No son reglas absolutas, pero son buenas guías. Cuando una supera el umbral, revisa si se puede dividir.

---

## 15. Documentación

### A. README
- Descripción del proyecto.
- Cómo compilar y ejecutar.
- Dependencias.
- Estructura de carpetas.
- Endpoints de la API (ruta, método, parámetros, respuesta).
- Formato de los archivos de datos.
- Ejemplos de uso.

### B. Comentarios en el código
- A nivel de archivo: qué contiene.
- A nivel de clase: qué representa y sus responsabilidades.
- A nivel de función: qué hace, qué recibe, qué devuelve.
- **No comentarios obvios.**

### C. Diagramas (opcional)
- Diagrama de clases (UML).
- Diagrama de flujo de una petición.
- Diagrama de capas.

### D. CHANGELOG (opcional)
- Registro de cambios por versión.

---

## 16. Buenas prácticas

- **Refactoriza con tests.** Aunque sean manuales.
- **Un cambio a la vez.** Compila y prueba entre cada uno.
- **Nombres claros.** Invierte tiempo en pensar nombres.
- **Funciones pequeñas.** Divide si crecen.
- **Una clase, una responsabilidad.**
- **Una capa, una carpeta.**
- **Reutiliza antes de duplicar.**
- **Evita sobre-abstracción.** No crees una clase para todo.
- **Constantes para valores mágicos.**
- **Manejo de errores consistente.**
- **No mezcles capas.**
- **Sin variables globales.**
- **Incluye solo lo necesario.** Forward declarations cuando puedas.
- **Usa `const` y `const&` correctamente.**
- **Compila con warnings activados.**
- **Formatea con una herramienta.**
- **Commit frecuente.**
- **Documenta decisiones importantes.** Comentarios de "por qué", no "qué".
- **Revisa el código después de un tiempo.** Verás cosas que antes no.
- **Pide revisiones de código.** Otra persona ve lo que tú no.

---

## 17. Errores comunes en refactorización

| Error | Solución |
|-------|----------|
| **Cambiar comportamiento accidentalmente** | Compila y prueba después de cada paso. |
| **Hacer todo de golpe** | Refactoriza incrementalmente. |
| **No tener punto de retorno** | Commit antes de empezar. |
| **Añadir features durante la refactorización** | Sé disciplinado. Solo reorganiza. |
| **Dividir en demasiados archivos** | Un archivo por clase o capa. No por función. |
| **Sobre-abstraer** | No crees clases para todo. La simplicidad es valiosa. |
| **Renombrar sin actualizar todos los usos** | Usa el IDE o buscar-y-reemplazar con cuidado. |
| **Olvidar actualizar includes** | Compila tras cada cambio para detectarlo. |
| **Duplicar en lugar de extraer** | Si aparece dos veces, extrae. |
| **Dejar código muerto** | Elimínalo. Git conserva el historial. |
| **Comentarios desactualizados** | Revísalos al cambiar el código. |
| **Inconsistencia de estilo** | Usa `clang-format`. |
| **Archivos con nombres confusos** | Un nombre por clase, y la clase describe su responsabilidad. |
| **Dependencias circulares** | Replantea la arquitectura. Las capas van en una sola dirección. |
| **`main` con lógica** | `main` solo instancia y arranca. |

---

## 18. Verificación de la refactorización

1. **Compila sin errores ni warnings.**
2. **Arranca el servidor.** Logs idénticos.
3. **Prueba cada endpoint con `curl`.**
4. **Prueba el flujo completo desde el navegador:**
   - Login.
   - Registrar, editar, eliminar usuarios.
   - Subir, listar, descargar archivos.
   - Logout.
5. **Prueba casos borde:** sin sesión, sesión inválida, sin permisos, datos inválidos.
6. **Verifica la persistencia:** reinicia el servidor, los datos siguen ahí.
7. **Revisa la estructura de carpetas:** debe coincidir con el diseño planificado.
8. **Revisa las métricas:** líneas por archivo, función, clase.
9. **Revisa el README:** debe estar actualizado.
10. **Revisa con `cppcheck`** (opcional).

Si todo funciona igual que antes, la refactorización fue exitosa.

---

## 19. Comparación antes/después

| Aspecto | Antes | Después |
|---------|-------|---------|
| Estructura | Un solo `servidor.cpp` | Varios archivos por responsabilidad |
| Carpetas | Todo mezclado | `src/`, `include/`, `public/`, `data/` |
| Clases | Pocas o ninguna | `Servidor`, `Router`, `HTTP`, `Usuario`, `BaseDatosTXT` |
| Responsabilidades | Mezcladas | Separadas por capa |
| Funciones | Largas y variadas | Cortas y específicas |
| Nombres | Algunos genéricos | Descriptivos y consistentes |
| Duplicación | Alta | Baja o nula |
| `main` | Largo, con lógica | 3-5 líneas |
| Validaciones | Dispersas | Centralizadas |
| Manejo de errores | Heterogéneo | Consistente |
| Documentación | Escasa | README completo |
| Compilación | Manual o un comando | Makefile/CMake |
| Colaboración | Difícil | Fácil (archivos separados) |
| Cambios localizados | No | Sí |

---

## 20. Resumen de conceptos clave

- **Refactorizar:** mejorar la estructura sin cambiar el comportamiento.
- **Código limpio:** nombres significativos, funciones pequeñas, sin duplicación, comentarios útiles, formato consistente.
- **DRY:** no repetir código. Extrae a funciones o clases.
- **SRP:** una clase, una responsabilidad.
- **Capas:** red, HTTP, enrutamiento, aplicación, dominio, persistencia.
- **Estructura de carpetas:** `src/`, `include/`, `public/`, `data/`.
- **Dividir archivos:** cuando pasan de 500-800 líneas.
- **Métricas:** líneas por archivo/función/clase, parámetros, anidamiento, complejidad.
- **Clases "Dios":** dividir si tienen demasiadas responsabilidades.
- **Validaciones:** en cliente (UX) y servidor (seguridad). Centralizadas.
- **Herramientas:** `-Wall -Wextra`, `cppcheck`, `clang-tidy`, `clang-format`, git.
- **Proceso seguro:** commit, cambio pequeño, compilar, probar, commit.
- **`main` mínimo:** solo instancia y arranca.
- **Documentación:** README completo, comentarios con el "por qué".
- **Verificación:** mismo comportamiento que antes.

---
# PASO A PASO

---
# Actividad 42 — Refactorización general

## Objetivo
Dividir el `servidor.cpp` monolítico en múltiples archivos por capa y responsabilidad. Sin cambiar el comportamiento. Todo debe seguir funcionando igual.

## Antes

```
Proyecto/
└── servidor.cpp   ← cientos de líneas con todo
```

## Después

```
Proyecto/
├── src/
│   ├── main.cpp
│   ├── Servidor.cpp
│   ├── Router.cpp
│   ├── HTTP.cpp
│   ├── Usuario.cpp
│   ├── BaseDatosSQL.cpp
│   ├── RepositorioArchivos.cpp
│   └── Sesiones.cpp
├── include/
│   ├── Servidor.h
│   ├── Router.h
│   ├── HTTP.h
│   ├── Usuario.h
│   ├── BaseDatosSQL.h
│   ├── RepositorioArchivos.h
│   └── Sesiones.h
├── public/
├── data/
└── Makefile
```

---

## Paso 1 — Antes de empezar

```bash
cp servidor.cpp servidor.cpp.bak
git add . && git commit -m "chore: snapshot antes de refactorizar"
```

Compila y prueba todo. Verifica que funciona. Este es tu punto de retorno.

---

## Paso 2 — `include/Usuario.h`

```cpp
#pragma once
#include <string>
using std::string;

enum class Rol { ALUMNO, MAESTRO, ADMIN };

class Usuario {
protected:
    string nombre;
    string usuario;
    string contrasena;
    string rol;

public:
    Usuario(string n, string u, string c, string r);
    virtual ~Usuario() {}

    string getNombre()     const { return nombre; }
    string getUsuario()    const { return usuario; }
    string getContrasena() const { return contrasena; }
    string getRol()        const { return rol; }

    void setNombre(string n)     { nombre = n; }
    void setUsuario(string u)    { usuario = u; }
    void setContrasena(string c) { contrasena = c; }
    void setRol(string r)        { rol = r; }

    virtual string obtenerMenu() const = 0;

    bool esValido() const;
};
```

## Paso 3 — `src/Usuario.cpp`

```cpp
#include "Usuario.h"

Usuario::Usuario(string n, string u, string c, string r)
    : nombre(n), usuario(u), contrasena(c), rol(r) {}

bool Usuario::esValido() const {
    return !nombre.empty() && !usuario.empty()
        && !contrasena.empty() && !rol.empty();
}
```

---

## Paso 4 — `include/HTTP.h`

```cpp
#pragma once
#include <string>
using std::string;

struct PeticionHTTP {
    string metodo;
    string ruta;
    string cabeceras;
    string cuerpo;

    static PeticionHTTP parsear(const string& crudo);
    string obtenerCookie(const string& nombre) const;
};

struct RespuestaHTTP {
    int    codigo = 200;
    string mime;
    string cuerpo;
    string setCookie;

    string serializar() const;
};

string obtenerMIME(const string& ruta);
```

## Paso 5 — `src/HTTP.cpp`

```cpp
#include "HTTP.h"
#include <map>
#include <sstream>

PeticionHTTP PeticionHTTP::parsear(const string& crudo) {
    PeticionHTTP p;

    size_t finLinea = crudo.find("\r\n");
    string primera = crudo.substr(0, finLinea);

    size_t p1 = primera.find(' ');
    size_t p2 = primera.find(' ', p1 + 1);
    p.metodo = primera.substr(0, p1);
    p.ruta   = primera.substr(p1 + 1, p2 - p1 - 1);

    size_t posCab = crudo.find("\r\n\r\n");
    if (posCab != string::npos) {
        p.cabeceras = crudo.substr(finLinea + 2, posCab - finLinea - 2);
        p.cuerpo    = crudo.substr(posCab + 4);
    }
    return p;
}

string PeticionHTTP::obtenerCookie(const string& nombre) const {
    size_t pos = cabeceras.find("Cookie:");
    if (pos == string::npos) return "";

    size_t fin = cabeceras.find("\r\n", pos);
    if (fin == string::npos) fin = cabeceras.size();

    std::stringstream ss(cabeceras.substr(pos + 7, fin - pos - 7));
    string par;
    while (std::getline(ss, par, ';')) {
        size_t i = par.find_first_not_of(" \t");
        if (i != string::npos) par = par.substr(i);

        size_t eq = par.find('=');
        if (eq == string::npos) continue;

        if (par.substr(0, eq) == nombre) return par.substr(eq + 1);
    }
    return "";
}

string RespuestaHTTP::serializar() const {
    string estado;
    switch (codigo) {
        case 200: estado = "200 OK"; break;
        case 201: estado = "201 Created"; break;
        case 400: estado = "400 Bad Request"; break;
        case 401: estado = "401 Unauthorized"; break;
        case 403: estado = "403 Forbidden"; break;
        case 404: estado = "404 Not Found"; break;
        case 405: estado = "405 Method Not Allowed"; break;
        case 409: estado = "409 Conflict"; break;
        case 413: estado = "413 Payload Too Large"; break;
        case 500: estado = "500 Internal Server Error"; break;
        default:  estado = "500 Internal Server Error";
    }

    string r = "HTTP/1.1 " + estado + "\r\n"
             + "Content-Type: " + mime + "\r\n"
             + "Content-Length: " + std::to_string(cuerpo.size()) + "\r\n"
             + "Connection: close\r\n";

    if (!setCookie.empty())
        r += "Set-Cookie: " + setCookie + "\r\n";

    r += "\r\n" + cuerpo;
    return r;
}

string obtenerMIME(const string& ruta) {
    static const std::map<string, string> m = {
        {".html", "text/html; charset=utf-8"},
        {".css",  "text/css; charset=utf-8"},
        {".js",   "application/javascript; charset=utf-8"},
        {".json", "application/json; charset=utf-8"},
        {".png",  "image/png"},
        {".jpg",  "image/jpeg"},
        {".svg",  "image/svg+xml"},
        {".ico",  "image/x-icon"},
    };
    size_t punto = ruta.find_last_of('.');
    if (punto == string::npos) return "application/octet-stream";
    auto it = m.find(ruta.substr(punto));
    return (it != m.end()) ? it->second : "application/octet-stream";
}
```

---

## Paso 6 — `include/BaseDatosSQL.h`

```cpp
#pragma once
#include <sqlite3.h>
#include <vector>
#include <string>
#include "Usuario.h"
using std::string;
using std::vector;

class BaseDatosSQL {
private:
    sqlite3* db = nullptr;
    string rutaArchivo;
    bool conectada = false;

public:
    explicit BaseDatosSQL(const string& ruta);
    ~BaseDatosSQL();

    bool conectar();
    void desconectar();

    bool    guardarUsuario(const Usuario& u);
    bool    actualizarUsuario(const Usuario& u);
    bool    eliminarUsuario(const string& usuario);
    Usuario buscarUsuario(const string& usuario);
    vector<Usuario> listarUsuarios();
    bool    existeUsuario(const string& usuario);
};
```

## Paso 7 — `src/BaseDatosSQL.cpp`

```cpp
#include "BaseDatosSQL.h"
#include <iostream>
using std::cerr;

BaseDatosSQL::BaseDatosSQL(const string& ruta) : rutaArchivo(ruta) {}
BaseDatosSQL::~BaseDatosSQL() { desconectar(); }

bool BaseDatosSQL::conectar() {
    if (sqlite3_open(rutaArchivo.c_str(), &db) != SQLITE_OK) return false;
    conectada = true;

    const char* sql =
        "CREATE TABLE IF NOT EXISTS usuarios ("
        " usuario TEXT PRIMARY KEY,"
        " nombre TEXT NOT NULL,"
        " contrasena TEXT NOT NULL,"
        " rol TEXT NOT NULL)";
    char* err = nullptr;
    int rc = sqlite3_exec(db, sql, nullptr, nullptr, &err);
    if (rc != SQLITE_OK) {
        cerr << "[DB] " << err << "\n";
        sqlite3_free(err);
        return false;
    }
    return true;
}

void BaseDatosSQL::desconectar() {
    if (db) { sqlite3_close(db); db = nullptr; conectada = false; }
}

bool BaseDatosSQL::guardarUsuario(const Usuario& u) {
    if (!conectada) return false;
    const char* sql = "INSERT INTO usuarios VALUES (?,?,?,?)";
    sqlite3_stmt* stmt = nullptr;
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return false;

    sqlite3_bind_text(stmt, 1, u.getUsuario().c_str(),    -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 2, u.getNombre().c_str(),     -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 3, u.getContrasena().c_str(), -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 4, u.getRol().c_str(),        -1, SQLITE_TRANSIENT);

    int rc = sqlite3_step(stmt);
    sqlite3_finalize(stmt);
    return rc == SQLITE_DONE;
}

// ... resto de métodos (buscarUsuario, listarUsuarios, actualizar, eliminar)
// Cópialos tal cual del servidor.cpp.bak, adaptando solo la sintaxis de la clase.
```

Sigue el mismo patrón para `actualizarUsuario`, `eliminarUsuario`, `buscarUsuario`, `listarUsuarios`. Son **copias literales** de lo que ya tenías, solo reubicadas.

---

## Paso 8 — `include/Router.h`

```cpp
#pragma once
#include <map>
#include <set>
#include <string>
#include "HTTP.h"
using std::string;

class Router {
private:
    using Clave = std::pair<string, string>;
    std::map<Clave, std::set<string>> permisos;

public:
    Router();
    bool autorizado(const string& ruta, const string& metodo, const string& rol) const;
    bool esRutaAPI(const string& ruta) const;
};
```

## Paso 9 — `src/Router.cpp`

```cpp
#include "Router.h"

Router::Router() {
    auto add = [this](const string& r, const string& m,
                      std::initializer_list<string> roles) {
        permisos[{r, m}] = std::set<string>(roles);
    };

    add("/api/perfil",      "GET",  {"admin","maestro","alumno"});
    add("/api/usuarios",    "GET",  {"admin"});
    add("/api/usuarios",    "POST", {"admin"});
    add("/api/calificaciones", "GET",  {"admin","maestro"});
    add("/api/calificaciones", "POST", {"admin","maestro"});
    add("/api/archivos",    "GET",  {"admin","maestro","alumno"});
    add("/api/subir",       "POST", {"admin","maestro"});
}

bool Router::autorizado(const string& ruta, const string& metodo,
                        const string& rol) const {
    auto it = permisos.find({ruta, metodo});
    if (it == permisos.end()) return false;
    return it->second.count(rol) > 0;
}

bool Router::esRutaAPI(const string& ruta) const {
    return ruta.rfind("/api/", 0) == 0;
}
```

---

## Paso 10 — `include/Servidor.h`

```cpp
#pragma once
#include "BaseDatosSQL.h"
#include "Router.h"
#include "RepositorioArchivos.h"
#include <unordered_map>
#include <ctime>

struct Sesion {
    string usuario;
    string rol;
    time_t creada;
    time_t expira;
};

class Servidor {
private:
    int puerto;
    int socketServidor = -1;
    string carpetaPublic;

    BaseDatosSQL           baseDatos;
    RepositorioArchivos    repoArchivos;
    Router                 router;

    std::unordered_map<string, Sesion> sesiones;

    static const int DURACION_SESION = 3600;

    int      crearSocket();
    string   leerPeticionCompleta(int cliente);
    void     enviarRespuesta(int cliente, const string& respuesta);
    string   generarSessionId();
    string   crearSesion(const string& usuario, const string& rol);
    bool     validarSesion(const string& id, Sesion& salida);
    string   servirArchivo(const string& ruta);
    string   manejarAPI(const PeticionHTTP& p, const Sesion& sesion);
    string   procesarLogin(const string& cuerpo);
    string   procesarPerfil(const Sesion& sesion);

public:
    Servidor(int puerto, string carpetaPublic, string rutaBD);
    ~Servidor();
    void ejecutar();
};
```

## Paso 11 — `src/Servidor.cpp`

Copia los métodos tal cual desde el servidor original. Solo cambia:
- `alumnos` → `baseDatos` (o elimina si ya no usas el vector en memoria).
- `permisos` → `router.autorizado(ruta, metodo, rol)`.
- La lógica de sesiones se mantiene igual.

```cpp
#include "Servidor.h"
#include <iostream>
#include <filesystem>
using std::cout;
using std::cerr;
namespace fs = std::filesystem;

Servidor::Servidor(int p, string cp, string rutaBD)
    : puerto(p), carpetaPublic(cp),
      baseDatos(rutaBD),
      repoArchivos("data/uploads/", "data/uploads/index.txt") {}

Servidor::~Servidor() {
    if (socketServidor != -1) CLOSE_SOCKET(socketServidor);
}

void Servidor::ejecutar() {
    if (!baseDatos.conectar()) {
        cerr << "No se pudo conectar a la BD\n";
        return;
    }

    if (crearSocket() == -1) return;
    cout << "Servidor en http://localhost:" << puerto << "\n";

    // ... bucle de accept (idéntico al original)
}
```

Los métodos `crearSocket`, `leerPeticionCompleta`, `enviarRespuesta`, `generarSessionId`, `crearSesion`, `validarSesion`, `servirArchivo`, `manejarAPI`, `procesarLogin`, `procesarPerfil` — todos son copias literales del servidor original, solo cambia el nombre de la función a método.

---

## Paso 12 — `src/main.cpp`

```cpp
#include "Servidor.h"
#include <iostream>

#ifdef _WIN32
  #include <winsock2.h>
  #pragma comment(lib, "ws2_32.lib")
#endif

int main() {
#ifdef _WIN32
    WSADATA wsa;
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) return 1;
#endif

    Servidor servidor(8080, "public/", "data/sistema.db");
    servidor.ejecutar();

#ifdef _WIN32
    WSACleanup();
#endif
    return 0;
}
```

---

## Paso 13 — `Makefile`

```makefile
CXX      = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -Iinclude
LIBS     = -lsqlite3
TARGET   = servidor
SRCS     = $(wildcard src/*.cpp)
OBJS     = $(SRCS:src/%.cpp=build/%.o)

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CXX) $(CXXFLAGS) -o $@ $^ $(LIBS)

build/%.o: src/%.cpp | build
	$(CXX) $(CXXFLAGS) -c $< -o $@

build:
	mkdir -p build

clean:
	rm -rf build $(TARGET)

.PHONY: all clean
```

Compilar:
```bash
make
```

---

## Paso 14 — Verificación (crítica)

1. `make` compila sin errores ni warnings.
2. `./servidor` arranca y loguea los mismos mensajes que antes.
3. Prueba con `curl` cada endpoint del `servidor.cpp.bak`:
   - `/api/hola`, `/api/hora`, `/api/perfil`
   - `/api/usuarios` (GET, POST)
   - Login + logout con cookie.
   - Subir archivo.
4. Abre el navegador y usa el panel admin.
5. Reinicia y verifica que `data/sistema.db` persiste.

Si algo difiere → revierte el último cambio.

---

## ✅ Checklist de la Actividad 42

- [ ] Existen `src/` e `include/`.
- [ ] Hay un `.h` por clase/módulo con `#pragma once`.
- [ ] Cada `.cpp` incluye solo lo necesario.
- [ ] `Makefile` compila todo con un comando.
- [ ] `main.cpp` tiene menos de 20 líneas.
- [ ] Comportamiento idéntico a `servidor.cpp.bak`.
- [ ] Compila con `-Wall -Wextra` sin warnings.

## 🧠 Mini-reto
Añade `include/Utils.h` con funciones auxiliares compartidas (`escaparJSON`, `trim`, `toLower`). Elimina duplicaciones que detectes.
