# Actividad 39. Roles y autorización

## Temas

- Autorización
    
- Roles
    

## Ejercicio

Crear tres tipos de usuario.

```text
Administrador

Maestro

Alumno
```

Controlar el acceso a las rutas.

Ejemplo:

Administrador

```text
/api/eliminarUsuario
```

Maestro

```text
/api/calificaciones
```

Alumno

```text
/api/perfil
```

Mostrar un mensaje de acceso denegado cuando corresponda.

---
# TEORÍA

---

## 1. Autenticación vs. Autorización

| Concepto | Pregunta que responde | Cuándo ocurre |
|----------|----------------------|----------------|
| **Autenticación** | ¿Quién eres? | En el login |
| **Autorización** | ¿Qué puedes hacer? | En cada petición protegida |

- **Autenticación** verifica identidad (usuario + contraseña).
- **Autorización** verifica permisos (según el rol o los privilegios del usuario).
- **Regla de oro:** **Autentica primero, autoriza después.** Nunca autorices sin haber autenticado.
- **En esta actividad:** Después de validar la sesión, el servidor comprueba el `rol` del usuario antes de procesar la petición.

---

## 2. ¿Qué es un rol?

- **Definición:** Una etiqueta que agrupa un conjunto de permisos. Los usuarios que comparten un rol pueden realizar las mismas acciones.
- **Ventaja:** No asignas permisos usuario por usuario; los asignas por rol. Menos trabajo, más coherente.
- **En el proyecto:**
  - `alumno` → acceso a su perfil, ver sus calificaciones, ver su horario.
  - `maestro` → registrar calificaciones, ver sus grupos.
  - `admin` → gestionar usuarios, eliminar cuentas, ver reportes globales.
- **Los roles son datos, no código:** En la BD, la columna `rol` de la tabla `usuarios` guarda el rol. El servidor lo lee desde la sesión o desde la BD.

### A. Rol vs. permisos
- **Rol:** Una etiqueta ("admin", "alumno"). Simple, rígido.
- **Permisos:** Lista detallada de acciones ("crear_usuario", "eliminar_usuario", "ver_reporte"). Flexible, granular.
- **Para esta actividad:** Roles simples. Los permisos granulares son una evolución posterior (RBAC con permisos, ACL).

### B. Jerarquía de roles (opcional)
- Un rol puede "contener" a otro. Ejemplo:
  - `admin` > `maestro` > `alumno`.
  - Un `admin` puede hacer todo lo que un `maestro` puede hacer, y más.
- **Ventaja:** Evita duplicar reglas.
- **Desventaja:** Más complejo. Requiere definir qué roles incluyen a cuáles.
- **Para esta actividad:** No es necesario. Roles planos son suficientes.

---

## 3. Modelo de autorización basado en roles (RBAC)

- **RBAC:** Role-Based Access Control. Es el modelo estándar en la industria.
- **Componentes:**
  - **Usuarios:** Personas o entidades que acceden al sistema.
  - **Roles:** Etiquetas que agrupan permisos.
  - **Permisos/Acciones:** Lo que se puede hacer.
  - **Asignaciones:** Qué roles tiene cada usuario y qué permisos tiene cada rol.
- **Flujo:** Usuario → tiene roles → cada rol tiene permisos → cada permiso autoriza una acción.
- **En el proyecto:** El rol vive en la sesión (y en la BD). El servidor lo consulta antes de cada acción protegida.

---

## 4. Estructura de los roles en el código

### A. Como strings
- `string rol = "alumno";`
- **Ventaja:** Simple, flexible, viene directo de la BD.
- **Desventaja:** Errores tipográficos no detectados en compilación.

### B. Como `enum`
- `enum class Rol { ALUMNO, MAESTRO, ADMIN };`
- **Ventaja:** Tipo seguro, el compilador detecta errores, mejor autocompletado.
- **Desventaja:** Al guardar en BD hay que convertir a string y viceversa.
- **Para esta actividad:** El `enum` es más limpio, pero requiere conversión al leer/escribir de la BD. Los strings son más directos.

### C. Constantes
- `const string ROL_ADMIN = "admin";`
- **Ventaja:** Evita strings mágicos repetidos.
- **Desventaja:** No previene todos los errores.
- **Recomendación:** Si usas strings, centraliza los valores en constantes.

**Fragmento suelto (enum):**
```cpp
enum class Rol {
    ALUMNO,
    MAESTRO,
    ADMIN
};
```

- **Ventaja:** En un `switch`, el compilador te avisa si olvidas un caso.
- **Conversión:** `toString(Rol)` y `parseRol(string)` como funciones auxiliares.

---

## 5. La sesión como fuente del rol

- **Recordatorio (Actividad 38):** La sesión guarda datos del usuario autenticado, entre ellos el `rol`.
- **Ventaja:** No tienes que consultar la BD en cada petición. La sesión ya tiene el rol.
- **Actualización del rol:** Si el rol del usuario cambia (por ejemplo, lo ascienden a admin), la sesión actual no lo refleja hasta que vuelva a loguearse o hasta que se actualice la sesión. Decide tu política:
  - **Simple:** El rol se congela al iniciar sesión.
  - **Robusto:** Consultar el rol actual en la BD en cada petición.

**Fragmento suelto (struct Sesion):**
```cpp
struct Sesion {
    string usuario;
    string rol;       // rol del usuario
    time_t creada;
    time_t expira;
};
```

- **Alternativa:** Guardar un `enum class Rol` en lugar de string.

---

## 6. Middleware de autorización

- **Definición:** Una función o paso intermedio que se ejecuta **antes** del handler de la ruta. Verifica si el usuario tiene permiso para acceder.
- **Ventaja:** Centraliza la lógica de autorización. No la repites en cada handler.
- **Flujo típico:**
  1. Extraer `sessionId` de la cookie.
  2. Validar sesión → obtener `Sesion`.
  3. Si la sesión es inválida → `401 Unauthorized`.
  4. Verificar si el rol del usuario está autorizado para esa ruta.
  5. Si no está autorizado → `403 Forbidden`.
  6. Si está autorizado → procesar la petición.

**Fragmento suelto (esqueleto conceptual):**
```cpp
bool autorizado(const string& ruta, const string& metodo, const string& rol) {
    // devolver true/false según la ruta y el rol
}
```

- **Uso:** En `manejarAPI` (o donde enrutés), llamar a `autorizado(...)` antes de despachar al handler.
- **Ventaja:** Añadir una ruta protegida es solo añadir una regla.

---

## 7. Matriz de permisos (rol × recurso)

Una forma clara de documentar y centralizar la autorización:

| Ruta | Método | Admin | Maestro | Alumno |
|------|--------|:-----:|:-------:|:------:|
| `/api/usuarios` | GET | ✅ | ❌ | ❌ |
| `/api/usuarios` | POST | ✅ | ❌ | ❌ |
| `/api/usuarios/{id}` | DELETE | ✅ | ❌ | ❌ |
| `/api/calificaciones` | GET | ✅ | ✅ | ✅ (solo propias) |
| `/api/calificaciones` | POST | ✅ | ✅ | ❌ |
| `/api/perfil` | GET | ✅ | ✅ | ✅ |
| `/api/alumnos` | GET | ✅ | ✅ | ❌ |

- **Ventaja:** Documento único. Fácil de revisar y mantener.
- **En el código:** Puedes tener esta matriz como un `map` o como un conjunto de reglas.
- **Cuidado con recursos "propios":** Un alumno solo ve **sus** calificaciones. Eso es autorización de recurso, no solo de ruta. Ver sección 11.

**Fragmento suelto (idea de mapa):**
```cpp
// ruta + método -> conjunto de roles permitidos
map<pair<string,string>, vector<string>> permisos;
permisos[{"/api/eliminarUsuario", "DELETE"}] = {"admin"};
permisos[{"/api/calificaciones", "POST"}] = {"admin", "maestro"};
permisos[{"/api/perfil", "GET"}] = {"admin", "maestro", "alumno"};
```

- **Ventaja:** Añadir un permiso es añadir una entrada.
- **Desventaja:** No maneja reglas complejas (por ejemplo, "solo el propio usuario").

---

## 8. Respuestas HTTP para autorización

| Situación | Código | Motivo |
|-----------|--------|--------|
| No hay sesión válida | `401 Unauthorized` | El usuario no está autenticado |
| Sesión válida pero sin permiso | `403 Forbidden` | El usuario está autenticado pero no puede acceder |
| Recurso no existe | `404 Not Found` | La ruta o el recurso no existe |
| Método incorrecto | `405 Method Not Allowed` | La ruta existe pero no acepta ese método |

### A. 401 vs. 403
- **401:** "No sé quién eres. Inicia sesión." El cliente debería redirigir a login.
- **403:** "Sé quién eres, pero no puedes acceder." No tiene sentido redirigir a login.

### B. Contenido de la respuesta
- **Mensaje genérico al cliente:** `{"ok":false,"error":"Acceso denegado"}`.
- **No reveles detalles:** No digas "solo administradores pueden acceder" o "te falta el rol X". El atacante aprende de esa información.
- **Log interno:** Sí puedes loguear en consola el motivo: "Usuario ana_lopez (alumno) intentó acceder a /api/eliminarUsuario".
- **Redirección:** El frontend puede redirigir a login en caso de 401, o mostrar "no tienes permisos" en caso de 403.

---

## 9. Frontend: adaptar la UI según el rol

- **Después del login exitoso:** El servidor puede devolver el `rol` en la respuesta. El frontend lo guarda (en memoria o `sessionStorage`).
- **Uso:**
  - Mostrar u ocultar botones según el rol.
  - Redirigir al panel correspondiente (`/panel-admin.html`, `/panel-alumno.html`, etc.).
  - Adaptar el menú.
- **Regla de oro:** **Ocultar en el frontend NO es seguridad.** Es UX.
  - Un usuario malicioso puede abrir DevTools y enviar la petición con `curl`.
  - El servidor **siempre** debe validar el rol. La UI solo mejora la experiencia.
  - **No confíes en el frontend para autorizar.**
- **Guardar el rol:** Puedes guardarlo en `sessionStorage` (se borra al cerrar la pestaña) o en una cookie no `HttpOnly`. **No lo guardes en `localStorage`** si es sensible.

---

## 10. Autorización por ruta vs. por recurso

### A. Autorización por ruta (simple)
- La regla se basa solo en la ruta y el método.
- **Ejemplo:** "Solo admins pueden acceder a `DELETE /api/usuarios`."
- **Suficiente** para esta actividad.

### B. Autorización por recurso (fina)
- La regla depende del **recurso específico**. No basta con saber el rol; hay que saber a qué recurso se accede.
- **Ejemplo:** "Un alumno solo puede ver **sus** calificaciones, no las de otros."
- **Ejemplo:** "Un maestro solo puede modificar calificaciones de **sus** alumnos."
- **Implementación:** Además de verificar el rol, verificar que el recurso pertenece al usuario o está dentro de su ámbito.
- **Esto es más complejo.** Requiere que cada handler verifique el recurso.
- **Para esta actividad:** No es obligatorio, pero menciónalo como una mejora.

**Fragmento suelto (idea):**
```cpp
// Un alumno solo puede consultar su propio perfil
if (rol == "alumno" && usuarioSesion != usuarioSolicitado) {
    responder403();
}
```

---

## 11. Ejemplo: la matriz en la práctica

Pensemos en tres rutas de ejemplo:

### A. `DELETE /api/eliminarUsuario`
- **Permitido solo a:** `admin`.
- **Si viene un `alumno` o `maestro`:** → `403 Forbidden`.
- **Si no hay sesión válida:** → `401 Unauthorized`.

### B. `POST /api/calificaciones`
- **Permitido a:** `admin`, `maestro`.
- **Si viene un `alumno`:** → `403`.
- **Si no hay sesión:** → `401`.

### C. `GET /api/perfil`
- **Permitido a:** `admin`, `maestro`, `alumno`.
- **Si no hay sesión:** → `401`.
- **Nota:** Como autorización fina, un alumno solo debería poder acceder a su propio perfil.

---

## 12. Centralización de la autorización

### A. En una función
- `bool autorizado(const string& ruta, const string& metodo, const string& rol);`
- **Ventaja:** Un solo lugar donde viven las reglas.
- **Desventaja:** Puede crecer mucho si hay muchas rutas.

### B. En un mapa de permisos
- `map<pair<string,string>, vector<string>> permisos;`
- **Ventaja:** Declarativo, fácil de leer.
- **Desventaja:** Menos flexible para reglas complejas.

### C. En cada handler
- Cada handler verifica el rol al inicio.
- **Ventaja:** Máxima flexibilidad.
- **Desventaja:** Duplicación, fácil de olvidar en una ruta nueva.
- **Recomendación:** Evítalo si puedes. Centraliza.

### D. Middleware genérico
- Una función que envuelve al handler y verifica antes de llamarlo.
- **Ventaja:** Transparente, no hay que recordar verificar.
- **Desventaja:** Más complejo de implementar en C++ sin frameworks.
- **Para esta actividad:** Función `autorizado` + llamada explícita en `manejarAPI` es suficiente.

---

## 13. Logging de autorización

- **Qué loguear:**
  - Intentos de acceso denegados: usuario, rol, ruta, timestamp.
  - **Por qué:** Detectar patrones sospechosos (un alumno intentando acceder a rutas de admin repetidamente).
- **Qué NO loguear:**
  - Contraseñas.
  - IDs de sesión completos (son credenciales en sí mismos).
- **Formato:** Una línea por evento. Fácil de parsear.
- **En producción:** Enviar los logs a un sistema centralizado (ELK, Loki, etc.).

---

## 14. Seguridad adicional

### A. No confíes en el cliente
- El `rol` que muestra la UI puede ser manipulado.
- El servidor **siempre** consulta el rol de la **sesión** o de la **BD**, no de un parámetro del cliente.
- **Nunca** aceptes el rol como dato del POST o de la URL para decidir permisos. El cliente solo envía su `sessionId`; el servidor resuelve el resto.

### B. Principio de menor privilegio
- Cada rol debe tener **el mínimo de permisos necesarios** para cumplir su función.
- Un alumno no necesita ver la lista completa de usuarios. Un maestro no necesita eliminar cuentas.
- **Revisa los permisos periódicamente.** Los roles tienden a acumular permisos con el tiempo.

### C. Defensa en profundidad
- No confíes solo en una capa. Aplica múltiples:
  - Validación en el middleware de autorización.
  - Validación dentro del handler (para casos especiales).
  - Validación en la BD (por ejemplo, roles válidos con `CHECK`).
  - Validación en el frontend (UX).
- **Aunque el frontend oculte un botón, el servidor debe rechazar la petición.**

### D. Auditoría
- Registra quién accedió a qué y cuándo. Especialmente para acciones sensibles (eliminar usuarios, modificar calificaciones).
- **Motivo:** Detectar abusos, cumplir con normativas, investigar incidentes.

---

## 15. Casos especiales

### A. Un usuario sin rol
- **Escenario:** Un usuario autenticado pero sin rol asignado. Es un error de datos.
- **Decisión:** O se le deniega el acceso a todo (excepto cerrar sesión), o se le asigna un rol por defecto ("invitado").
- **Recomendación:** Rechaza el login si el usuario no tiene rol válido. No deberías tener usuarios sin rol.

### B. Un rol no reconocido
- **Escenario:** En la BD hay un rol como `"superadmin"` que tu código no maneja.
- **Decisión:** Tratar como no autorizado (denegar). Nunca asumir un permiso por defecto.
- **Recomendación:** Valida los roles en la BD con un `CHECK` o una tabla separada.

### C. Cambio de rol durante la sesión
- **Escenario:** El usuario es ascendido a admin mientras tiene una sesión activa.
- **Decisión:**
  - **Simple:** La sesión guarda el rol antiguo hasta que se cierre sesión y vuelva a entrar.
  - **Robusto:** Consultar la BD en cada petición para obtener el rol actual.
- **Recomendación:** Documenta la política. Para esta actividad, congelar el rol en la sesión es suficiente.

### D. Multi-rol
- **Escenario:** Un usuario podría ser maestro y administrador a la vez.
- **Decisión:** Modelar como `vector<string> roles` en lugar de un solo rol.
- **Complejidad:** Añade complejidad a las comprobaciones.
- **Para esta actividad:** Un solo rol por usuario. Multi-rol es una mejora futura.

---

## 16. Pruebas

### A. Desde el navegador (con sesión iniciada)
1. Inicia sesión como **admin**. Verifica que puedes acceder a todas las rutas.
2. Inicia sesión como **maestro**. Verifica que accedes a las rutas de maestro y no a las de admin.
3. Inicia sesión como **alumno**. Verifica que solo accedes a las rutas de alumno.
4. Sin sesión (cerrando sesión o borrando la cookie), verifica que todas las rutas protegidas devuelven `401`.

### B. Con `curl`
- Prueba con y sin cookie. Copia el `sessionId` de DevTools → Application → Cookies.
- Comando conceptual:
  ```
  curl -X DELETE -H "Cookie: sessionId=<id>" http://localhost:8080/api/eliminarUsuario
  ```
- **Casos a cubrir:**
  - Sin cookie → `401`.
  - Cookie inválida → `401`.
  - Cookie de alumno intentando ruta de admin → `403`.
  - Cookie de admin accediendo a ruta de admin → `200`.

### C. Desde el frontend
- Con cada rol, verifica que el menú muestra solo las opciones permitidas.
- Verifica que al intentar acceder manualmente a una URL no permitida, el servidor responde `403`.

---

## 17. Estructura de código sugerida

### A. Atributos y métodos en `Servidor`
- **Atributo:** `map<pair<string,string>, vector<string>> permisos;` (o la estructura que elijas).
- **Métodos:**
  - `bool autorizado(const string& ruta, const string& metodo, const string& rol);`
  - `void cargarPermisos();` → inicializa el mapa. Se llama en el constructor.
  - `bool esRutaPublica(const string& ruta);` → rutas que no requieren sesión (`/api/login`, archivos estáticos).

### B. Flujo en `manejarAPI` (o equivalente)
1. Si la ruta es `/api/login` → procesar login (sin verificar sesión).
2. Para el resto:
   - Extraer `sessionId` de la cookie.
   - Validar sesión.
   - Si falla → `401`.
   - Verificar autorización: `autorizado(ruta, metodo, sesion.rol)`.
   - Si falla → `403`.
   - Procesar el handler correspondiente.

### C. `Sesion` como fuente del rol
- El rol viene de la sesión, no del cliente.
- El cliente no envía su rol; solo envía el `sessionId`.

---

## 18. Buenas prácticas

- **Autentica primero, autoriza después.** Nunca al revés.
- **Deniega por defecto.** Si una ruta no tiene una regla explícita, rechazarla. Es más seguro que autorizar por defecto.
- **Centraliza las reglas.** Un solo lugar donde se definen los permisos.
- **Usa códigos correctos:** 401 para no autenticado, 403 para no autorizado.
- **No reveles detalles** al cliente sobre por qué se denegó el acceso.
- **Loguea los intentos denegados** internamente.
- **No confíes en el cliente.** El rol viene de la sesión, no del cuerpo ni de la URL.
- **Principio de menor privilegio.** Cada rol, el mínimo de permisos.
- **Revisa los permisos periódicamente.** Los roles acumulan permisos con el tiempo.
- **Documenta la matriz de permisos.** Facilita el mantenimiento y las auditorías.
- **Adapta la UI al rol, pero no como seguridad.** Es solo UX.
- **Sé consistente con la nomenclatura.** Si usas "admin" en la BD, usa "admin" en el código. No mezcles "administrador" y "admin".
- **Valida los roles en la BD.** Un `CHECK` o tabla separada evita valores inválidos.
- **Considera multi-rol si el proyecto crece.** Pero no lo hagas hasta que sea necesario.
- **Prueba cada rol con cada ruta.** Es la única forma de verificar la matriz completa.
- **Audita acciones sensibles.** Eliminar usuarios, modificar notas, cambiar roles. Todo debe quedar registrado.

---

## 19. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Confundir autenticación con autorización** | Autenticación: ¿quién eres? Autorización: ¿qué puedes hacer? Son pasos distintos. |
| **Responder 401 cuando el usuario está autenticado pero sin permiso** | Usa 403. 401 es solo cuando no hay sesión válida. |
| **Responder 403 cuando no hay sesión** | Usa 401. 403 implica que ya sabes quién es. |
| **Confiar en el rol que envía el cliente** | El rol siempre viene de la sesión o de la BD. Nunca del cliente. |
| **Ocultar el botón en el frontend y creer que ya está seguro** | El servidor debe verificar siempre. Ocultar es solo UX. |
| **Autorizar por defecto cuando no hay regla** | Deniega por defecto. Es más seguro. |
| **Repetir la lógica de autorización en cada handler** | Centraliza en una función o middleware. |
| **No loguear intentos denegados** | Loguea para auditoría y detección de ataques. |
| **Dar demasiados permisos a un rol** | Principio de menor privilegio. Revisa y reduce. |
| **Roles sin validación en la BD** | `CHECK (rol IN ('alumno','maestro','admin'))` o tabla de roles. |
| **Cambiar el rol en la BD y esperar que se refleje sin re-login** | Documenta la política. Congela el rol en la sesión o consulta la BD en cada petición. |
| **No verificar el rol en rutas sensibles** | Cualquier ruta con efecto (DELETE, POST con datos) debe tener verificación explícita. |
| **Mensajes de error que revelan la política** | "Acceso denegado" genérico. Los detalles van al log del servidor, no al cliente. |
| **Permitir que un usuario se autoasigne un rol** | El rol solo lo cambia un admin a través de una ruta específica y protegida. |
| **Olvidar proteger las rutas nuevas** | Al añadir una ruta, revisa la matriz y añade su regla. Deniega por defecto hasta que la definas. |
| **Asumir que el frontend puede hacer cumplir los roles** | El frontend es un cliente más. Puede mentir. Solo el servidor autoriza. |

---

## 20. Verificación

1. **Crea usuarios con cada rol** en la BD (o modifica el rol de uno existente).
2. **Inicia sesión con cada uno** y verifica que la sesión guarda el rol correcto.
3. **Prueba cada ruta con cada rol**:
   - `DELETE /api/eliminarUsuario`: admin ✅, maestro ❌ 403, alumno ❌ 403.
   - `POST /api/calificaciones`: admin ✅, maestro ✅, alumno ❌ 403.
   - `GET /api/perfil`: los tres ✅.
4. **Prueba sin sesión** en todas las rutas protegidas → `401`.
5. **Prueba con sesión inválida** → `401`.
6. **Prueba intentando cambiar el rol en la cookie o en el cuerpo** → el servidor debe ignorarlo (el rol viene de la sesión).
7. **Revisa los logs** del servidor. Deben aparecer los intentos denegados con usuario, rol y ruta.
8. **Revisa DevTools → Network** para verificar los códigos de estado correctos.
9. **Verifica la UI:** con cada rol, el menú debe mostrar solo lo permitido.
10. **Prueba manualmente en la barra de direcciones** una ruta no permitida para tu rol. El servidor debe responder con `403` (y el frontend con un mensaje de "Acceso denegado").

---

## 21. Resumen de conceptos clave

- **Autenticación vs. autorización:** Quién eres vs. qué puedes hacer.
- **Rol:** Etiqueta que agrupa permisos. Ejemplos: alumno, maestro, admin.
- **RBAC:** Modelo estándar de autorización basada en roles.
- **Sesión:** Guarda el rol del usuario. Fuente del rol en cada petición.
- **Middleware de autorización:** Verifica el rol antes del handler.
- **Matriz de permisos:** Documenta qué rol puede acceder a qué ruta.
- **Códigos HTTP:** `401` (sin autenticar), `403` (autenticado sin permiso).
- **Denegar por defecto:** Si no hay regla explícita, rechazar.
- **Principio de menor privilegio:** Mínimo necesario para cada rol.
- **Centralización:** Un solo lugar donde viven las reglas.
- **Logging:** Registra intentos denegados para auditoría.
- **Frontend:** Adapta la UI, pero no como seguridad. El servidor siempre valida.
- **No confiar en el cliente:** El rol viene de la sesión, no del cuerpo ni de la URL.
- **Autorización por ruta vs. por recurso:** La primera es simple; la segunda es más fina y más compleja.
- **Defensa en profundidad:** Múltiples capas de verificación.
- **Auditoría:** Registra quién hizo qué y cuándo.

---
# PASO A PASO

---
# Actividad 39 — Roles y autorización

## Objetivo
Implementar autorización por roles. Cada ruta de la API solo será accesible para ciertos roles. Tras verificar la sesión (actividad 38), el servidor comprueba el rol antes de ejecutar el handler.

## Concepto clave
- **Autenticación** = ¿quién eres? (login, sesión).
- **Autorización** = ¿qué puedes hacer? (rol, permisos).
- **`401 Unauthorized`** = sin sesión válida.
- **`403 Forbidden`** = sesión válida pero sin permiso.

---

## Paso 1 — Atributo `permisos` en `Servidor`

Añade un `map` con la matriz de permisos:

```cpp
#include <set>

class Servidor {
private:
    // ruta + método → conjunto de roles permitidos
    std::map<std::pair<std::string, std::string>, std::set<std::string>> permisos;

    void cargarPermisos() {
        auto add = [this](const string& ruta, const string& metodo,
                          std::initializer_list<string> roles) {
            permisos[{ruta, metodo}] = std::set<string>(roles);
        };

        // Rutas específicas del ejercicio
        add("/api/eliminarUsuario", "DELETE", {"admin"});
        add("/api/calificaciones",  "GET",    {"admin", "maestro"});
        add("/api/calificaciones",  "POST",   {"admin", "maestro"});
        add("/api/perfil",          "GET",    {"admin", "maestro", "alumno"});
        add("/api/usuarios",        "GET",    {"admin"});
        add("/api/usuarios",        "POST",   {"admin"});
        add("/api/usuarios",        "PUT",    {"admin"});
        add("/api/usuarios",        "DELETE", {"admin"});
        add("/api/archivos",        "GET",    {"admin", "maestro", "alumno"});
        add("/api/subir",           "POST",   {"admin", "maestro"});
    }
};
```

Llamar `cargarPermisos()` en el constructor.

---

## Paso 2 — Función `autorizado`

```cpp
bool Servidor::autorizado(const string& ruta, const string& metodo,
                          const string& rol) const {
    auto it = permisos.find({ruta, metodo});
    if (it == permisos.end()) {
        // Por defecto, denegar. Es más seguro.
        return false;
    }
    return it->second.count(rol) > 0;
}
```

🎯 **Denegar por defecto.** Si una ruta no está en la tabla, se rechaza. Es más seguro que autorizar implícitamente.

---

## Paso 3 — Aplicar en `manejarAPI`

Modifica el inicio de `manejarAPI` para verificar la sesión y el rol antes de procesar cualquier handler:

```cpp
string Servidor::manejarAPI(const string& peticion, const string& ruta,
                            const string& metodo, const string& cuerpo) {
    // 1) Rutas públicas: login (no requiere sesión)
    if (ruta == "/api/login" && metodo == "POST") {
        return procesarLogin(cuerpo);
    }

    // 2) Rutas que requieren sesión
    string sessionId = extraerCookie(peticion, "sessionId");
    Sesion sesion;
    if (sessionId.empty() || !validarSesion(sessionId, sesion)) {
        return construirRespuesta(401, "application/json; charset=utf-8",
            "{\"ok\":false,\"error\":\"No autenticado\"}");
    }

    // 3) Logout no requiere rol específico, solo sesión
    if (ruta == "/api/logout" && metodo == "POST") {
        sesiones.erase(sessionId);
        string setCookie = "sessionId=; Path=/; HttpOnly; SameSite=Lax; Max-Age=0";
        return construirRespuestaConCookie(200,
            "application/json; charset=utf-8",
            "{\"ok\":true}", setCookie);
    }

    // 4) Autorización: ¿el rol permite esta ruta/método?
    if (!autorizado(ruta, metodo, sesion.rol)) {
        cout << "[403] " << sesion.usuario << " (" << sesion.rol
             << ") intentó acceder a " << metodo << " " << ruta << "\n";
        return construirRespuesta(403, "application/json; charset=utf-8",
            "{\"ok\":false,\"error\":\"Acceso denegado\"}");
    }

    // 5) Handlers
    if (ruta == "/api/perfil" && metodo == "GET") {
        string json = "{\"usuario\":\"" + escaparJSON(sesion.usuario) + "\","
                      "\"rol\":\""     + escaparJSON(sesion.rol)     + "\"}";
        return construirRespuesta(200, "application/json; charset=utf-8", json);
    }

    if (ruta == "/api/eliminarUsuario" && metodo == "DELETE") {
        return procesarEliminarUsuario(peticion, cuerpo);
    }

    if (ruta == "/api/calificaciones" && metodo == "GET") {
        return procesarListaCalificaciones(sesion);
    }

    // ... resto de handlers (usuarios, archivos, etc.)

    return construirRespuesta(404, "application/json; charset=utf-8",
        "{\"ok\":false,\"error\":\"Ruta no encontrada\"}");
}
```

📌 **Observa el orden:**
1. ¿Es pública? → procesar.
2. ¿Tiene sesión válida? → si no, 401.
3. ¿El rol permite esta ruta? → si no, 403.
4. Solo entonces: procesar el handler.

---

## Paso 4 — Endpoints de ejemplo

### A. `/api/perfil` (los tres roles)

```cpp
// Ya está en el bloque de arriba
```

### B. `/api/calificaciones` (admin y maestro)

```cpp
string Servidor::procesarListaCalificaciones(const Sesion& s) {
    // Si es alumno, más adelante filtramos por "sus" calificaciones
    // Por ahora devolvemos lista vacía como ejemplo
    string json = "[]";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

### C. `/api/eliminarUsuario` (solo admin)

```cpp
string Servidor::procesarEliminarUsuario(const string& peticion, const string& cuerpo) {
    string usuario = extraerValorJSON(cuerpo, "usuario");
    if (usuario.empty()) {
        return construirRespuesta(400, "application/json; charset=utf-8",
            "{\"ok\":false,\"error\":\"Falta el usuario\"}");
    }

    if (!baseDatos.eliminarUsuario(usuario)) {
        return construirRespuesta(404, "application/json; charset=utf-8",
            "{\"ok\":false,\"error\":\"Usuario no encontrado\"}");
    }
    return construirRespuesta(200, "application/json; charset=utf-8",
        "{\"ok\":true}");
}
```

---

## Paso 5 — Frontend: adaptar la UI según el rol

Al login exitoso, guardar el rol:

```js
// En login.js, tras login exitoso
sessionStorage.setItem('usuario', respuesta.nombre);
sessionStorage.setItem('rol', respuesta.rol);
```

Al cargar cualquier panel:

```js
const rol = sessionStorage.getItem('rol');
if (!rol) {
  window.location.href = '/login.html';
}

// Ocultar elementos según el rol
document.querySelectorAll('[data-rol]').forEach(el => {
  const permitidos = el.dataset.rol.split(',');
  if (!permitidos.includes(rol)) {
    el.style.display = 'none';
  }
});
```

En el HTML:

```html
<button data-rol="admin" id="btn-eliminar">Eliminar usuario</button>
<button data-rol="admin,maestro" id="btn-calificar">Calificar</button>
<a data-rol="admin,maestro,alumno" href="/perfil.html">Mi perfil</a>
```

⚠️ **Esto es solo UX.** El servidor **siempre** valida el rol. Aunque alguien fuerce el botón visible, el servidor responde 403.

---

## Paso 6 — Probar con `curl`

```bash
# Login como admin → guarda cookie
curl -i -c admin.txt -X POST -H "Content-Type: application/json" \
  -d '{"usuario":"ana_admin","contrasena":"secreta"}' \
  http://localhost:8080/api/login

# Admin accede a ruta de admin → 200
curl -i -b admin.txt -X DELETE -H "Content-Type: application/json" \
  -d '{"usuario":"pepe"}' \
  http://localhost:8080/api/eliminarUsuario

# Login como alumno
curl -i -c alumno.txt -X POST -H "Content-Type: application/json" \
  -d '{"usuario":"luis_alumno","contrasena":"secreta"}' \
  http://localhost:8080/api/login

# Alumno intenta ruta de admin → 403
curl -i -b alumno.txt -X DELETE -H "Content-Type: application/json" \
  -d '{"usuario":"pepe"}' \
  http://localhost:8080/api/eliminarUsuario

# Sin cookie → 401
curl -i -X DELETE -H "Content-Type: application/json" \
  -d '{"usuario":"pepe"}' \
  http://localhost:8080/api/eliminarUsuario
```

---

## Paso 7 — Log de autorización

Cada vez que se deniega, imprime en consola:

```
[403] luis_alumno (alumno) intentó acceder a DELETE /api/eliminarUsuario
```

Nunca loguear contraseñas. Nunca loguear IDs completos de sesión.

---

## Paso 8 — Casos borde

- **Rol no reconocido** en la tabla de permisos → `403` (denegar por defecto).
- **Cambio de rol durante sesión activa:** el rol se congela en la sesión. Si se cambia en BD, el usuario debe re-loguearse.
- **Multi-rol:** si más adelante un usuario tiene varios roles, la sesión guarda un `set<string>` en lugar de un string, y `autorizado` verifica intersección.

---

## ✅ Checklist de la Actividad 39

- [ ] Existe `map<pair<string,string>, set<string>> permisos`.
- [ ] `autorizado()` deniega por defecto rutas no registradas.
- [ ] Rutas protegidas verifican sesión → 401, luego rol → 403.
- [ ] Se loguean los intentos denegados.
- [ ] El frontend oculta botones según el rol (solo UX).
- [ ] Probaste con `curl` los tres roles contra varias rutas.

## 🧠 Mini-reto
Implementa **autorización por recurso**: un alumno solo puede acceder a `GET /api/calificaciones?alumno={suUsuario}`. Si intenta consultar otro alumno, responde 403.