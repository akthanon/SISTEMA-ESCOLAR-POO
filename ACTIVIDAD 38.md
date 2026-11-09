# Actividad 38. Sesiones mediante cookies

## Temas

- Cookies
    
- Sesiones
    

## Ejercicio

Después del inicio de sesión exitoso:

- generar un identificador de sesión;
    
- enviarlo mediante una cookie;
    
- reconocer automáticamente al usuario en las siguientes peticiones.
    

Agregar una opción para cerrar sesión eliminando la cookie.

---
# TEORÍA
---

## 1. El problema: HTTP es sin estado

- **HTTP es stateless:** Cada petición es independiente. El servidor no recuerda nada entre peticiones.
- **Consecuencia:** Después del login, la siguiente petición (por ejemplo, `GET /api/alumnos`) no sabe quién la hace.
- **Solución tradicional:** Cookies y sesiones.

**Idea conceptual:**
- En el login, el servidor genera un **identificador único** y lo envía al cliente.
- El cliente lo guarda (como cookie).
- En cada petición posterior, el cliente envía automáticamente ese identificador.
- El servidor lo busca en su tabla de sesiones y sabe quién es el usuario.

---

## 2. ¿Qué es una cookie?

- **Definición:** Un pequeño fragmento de texto que el servidor envía al navegador y que este **almacena** y **reenvía** automáticamente en cada petición al mismo dominio.
- **Estructura:** Par clave-valor + atributos opcionales.
- **Envío desde el servidor:** Cabecera `Set-Cookie` en la respuesta HTTP.
- **Envío desde el cliente:** Cabecera `Cookie` en la petición HTTP.
- **Almacenamiento:** El navegador las guarda en disco y las gestiona automáticamente.
- **Alcance:** Se envían solo al dominio que las estableció (y, opcionalmente, al path configurado).

**Analogía:** La cookie es como una pulsera que te dan al entrar a un evento. Al pasar por cada puerta, la muestras y te reconocen sin pedirte tus datos otra vez.

---

## 3. Cabeceras HTTP relacionadas

### A. `Set-Cookie` (servidor → cliente)
- Se envía en la **respuesta** del servidor.
- Establece una cookie en el navegador.
- **Formato básico:**
  ```
  Set-Cookie: nombre=valor; atributo1; atributo2; ...
  ```
- **Ejemplo conceptual:**
  ```
  Set-Cookie: sessionId=abc123; Path=/; HttpOnly; Max-Age=3600
  ```
- **Puede haber varias:** Múltiples cabeceras `Set-Cookie`, una por cookie.

### B. `Cookie` (cliente → servidor)
- Se envía en la **petición** del cliente.
- Contiene todas las cookies del dominio separadas por `; `.
- **Formato:**
  ```
  Cookie: nombre1=valor1; nombre2=valor2; ...
  ```
- **Ejemplo conceptual:**
  ```
  Cookie: sessionId=abc123
  ```
- **El navegador la envía automáticamente.** No tienes que programarla desde JavaScript.

---

## 4. Atributos de las cookies

| Atributo | Función |
|----------|---------|
| `Path=/` | La cookie se envía a todas las rutas del dominio. Si pones `Path=/api`, solo a `/api/*`. |
| `Domain=ejemplo.com` | Dominio al que se envía. Por defecto, el dominio que la estableció. |
| `Max-Age=3600` | Duración en segundos. Si se omite, es cookie de sesión (se borra al cerrar el navegador). |
| `Expires=Fecha` | Alternativa a `Max-Age`. Formato de fecha HTTP. |
| `Secure` | Solo se envía por HTTPS. Imprescindible en producción. |
| `HttpOnly` | No accesible desde JavaScript. **Previene XSS.** Muy recomendado para cookies de sesión. |
| `SameSite=Strict/Lax/None` | Controla el envío en peticiones cross-site. **Previene CSRF.** Valores: `Strict` (más seguro), `Lax` (equilibrio), `None` (requiere `Secure`). |

**Recomendación para cookies de sesión:**
```
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; SameSite=Lax; Max-Age=3600
```

- **`HttpOnly`:** Vital. Sin esto, un ataque XSS puede robar la cookie.
- **`SameSite=Lax`:** Buen equilibrio entre seguridad y usabilidad.
- **`Secure`:** Obligatorio en producción con HTTPS.

---

## 5. ¿Qué es una sesión?

- **Definición:** Un estado asociado a un usuario autenticado que persiste entre varias peticiones HTTP.
- **Contenido típico:**
  - Identificador de sesión (ID).
  - Usuario asociado.
  - Fecha de creación.
  - Fecha de última actividad.
  - Fecha de expiración.
  - (Opcional) Datos adicionales (rol, permisos).

- **Dónde vive:** En **memoria del servidor** (típicamente un `map`).
  - Clave: ID de sesión.
  - Valor: struct con los datos del usuario y metadatos.

- **Qué viaja al cliente:** Solo el **ID de sesión** (en una cookie). El resto vive en el servidor.

**Ventaja:** El cliente no puede manipular los datos de sesión. Solo puede enviar su ID; si lo inventa, no coincide con ninguna sesión válida.

---

## 6. Flujo del login

1. El usuario envía POST `/api/login` con `usuario` y `contrasena`.
2. El servidor valida las credenciales contra la BD.
3. Si son válidas:
   - **Genera un ID de sesión único** (aleatorio, largo).
   - **Guarda la sesión** en el `map` del servidor: `id → { usuario, ..., expira }`.
   - **Envía la cookie** `Set-Cookie: sessionId=<id>; HttpOnly; ...` en la respuesta.
   - Responde con éxito (por ejemplo, `200 OK` con datos del usuario).
4. Si no son válidas:
   - Responde `401 Unauthorized` sin cookie.

**Punto clave:** El cliente **nunca ve** el ID de sesión como dato del cuerpo. Solo lo recibe en la cookie.

---

## 7. Flujo de peticiones posteriores

1. El cliente hace una petición (por ejemplo, `GET /api/alumnos`).
2. **El navegador añade automáticamente** la cabecera `Cookie: sessionId=abc123`.
3. El servidor **lee la cabecera `Cookie`**, extrae el ID.
4. El servidor busca el ID en el `map` de sesiones.
5. **Si existe y no ha expirado:**
   - El servidor sabe quién es el usuario.
   - Procesa la petición con esa información.
6. **Si no existe o expiró:**
   - El servidor responde `401 Unauthorized`.
   - El cliente redirige a `login.html`.

**Observación importante:** Toda la seguridad recae en:
- Que el ID sea **imposible de adivinar**.
- Que el ID esté marcado como `HttpOnly` y `Secure` (en producción).
- Que el servidor **valide el ID** en cada petición protegida.

---

## 8. Generación del ID de sesión

- **Requisito:** Debe ser **único**, **aleatorio** e **impredecible**.
- **Longitud recomendada:** Al menos 128 bits (32 caracteres hex o 22 en base64).
- **Fuentes de aleatoriedad:**
  - `std::random_device` (C++11+).
  - `std::mt19937_64` (Mersenne Twister) sembrado con `random_device`.
- **Nunca uses:** `rand()`, `srand(time(0))`, o el reloj del sistema como única fuente. Son predecibles.
- **Alternativa:** Bibliotecas como `libsodium` ofrecen generadores criptográficamente seguros.

**Fragmento suelto (idea de generación):**
```cpp
random_device rd;
mt19937_64 gen(rd());
uniform_int_distribution<uint64_t> dist;
uint64_t parte1 = dist(gen);
uint64_t parte2 = dist(gen);
// concatenar en un string hexadecimal
```

- **Ventaja:** `random_device` es criptográficamente fuerte en la mayoría de las plataformas.
- **Desventaja:** Puede ser lento; úsalo solo para generar IDs, no en bucles.

---

## 9. Almacenamiento de sesiones

### A. Estructura del `map`
- **Contenedor:** `map<string, Sesion>` o `unordered_map<string, Sesion>`.
  - `map`: ordenado, búsqueda O(log n).
  - `unordered_map`: sin orden, búsqueda O(1). **Preferible aquí.**
- **Clave:** ID de sesión.
- **Valor:** struct con la información.

### B. Struct `Sesion`
- `string usuario;` → nombre de usuario asociado.
- `string rol;` → rol para autorización.
- `time_t creada;` → timestamp de creación.
- `time_t ultimaActividad;` → para implementar expiración por inactividad.
- `time_t expira;` → timestamp absoluto de expiración.

**Fragmento suelto (idea):**
```cpp
struct Sesion {
    string usuario;
    string rol;
    time_t creada;
    time_t ultimaActividad;
    time_t expira;
};

unordered_map<string, Sesion> sesiones;
```

- **Un solo `map`** en el servidor, compartido por todas las peticiones.
- **Debe ser accesible** desde cualquier handler que necesite saber quién es el usuario.

### C. Dónde vive el `map`
- **En el `Servidor`** como atributo privado. Compartido por todos los métodos.
- **Ventaja:** Encapsulado, no hay variables globales.

---

## 10. Validación de la sesión en cada petición

- **Función típica:** `bool validarSesion(const string& sessionId, Sesion& salida);`
  - Busca el ID en el `map`.
  - Si no existe, devuelve `false`.
  - Si existe, verifica que no haya expirado.
  - Si expiró, elimina la entrada y devuelve `false`.
  - Si es válida, actualiza `ultimaActividad` y devuelve `true` con la sesión en `salida`.

- **Uso típico en un handler:**
  1. Extraer el ID de la cookie.
  2. Validar la sesión.
  3. Si falla → `401 Unauthorized`.
  4. Si tiene éxito → procesar la petición con el usuario.

- **Rutas que requieren sesión:** Todas las `/api/` excepto `/api/login`.
- **Rutas públicas:** `/api/login`, servir archivos estáticos (`/index.html`, `/css/`, `/js/`, `/img/`).
- **Rutas protegidas:** `/api/alumnos`, `/api/usuarios`, etc.

---

## 11. Parsear la cabecera `Cookie`

- **Formato:** `Cookie: nombre1=valor1; nombre2=valor2; ...`
- **Objetivo:** Extraer el valor de `sessionId`.
- **Pasos:**
  1. Buscar la cabecera `Cookie:` en las cabeceras (case-insensitive).
  2. Extraer el valor completo hasta el `\r\n`.
  3. Separar por `;`.
  4. Para cada par, separar por el primer `=`.
  5. Si el nombre es `sessionId`, guardar el valor.
- **Consideración:** Los valores pueden tener espacios alrededor (trim). El navegador normalmente no los añade, pero conviene ser defensivo.

**Fragmento suelto (idea):**
```cpp
size_t pos = cabeceras.find("Cookie:");
if (pos != string::npos) {
    size_t fin = cabeceras.find("\r\n", pos);
    string cookies = cabeceras.substr(pos + 7, fin - pos - 7);
    // separar por ';', buscar sessionId
}
```

- **Solo hay una cabecera `Cookie`** en la petición (el navegador concatena todas las cookies). No busques múltiples `Cookie:`.
- **Los atributos de la cookie** (`Path`, `HttpOnly`, etc.) no se envían al servidor. Solo el par `nombre=valor`.

---

## 12. Establecer la cookie en la respuesta

- **Cabecera:** `Set-Cookie: sessionId=<valor>; atributos`.
- **Ubicación:** Entre las cabeceras, antes del `\r\n\r\n`.
- **Puede haber varias:** una cabecera `Set-Cookie` por cada cookie.
- **Atributos recomendados:**
  - `Path=/` → la cookie se envía a todo el sitio.
  - `HttpOnly` → no accesible desde JavaScript.
  - `SameSite=Lax` → mitigación de CSRF.
  - `Max-Age=3600` → expira en 1 hora (por ejemplo).
  - `Secure` → solo HTTPS (en producción).

**Fragmento suelto (cabecera):**
```
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; SameSite=Lax; Max-Age=3600
```

- **`Max-Age=0`** → el navegador borra la cookie inmediatamente. Útil para logout.
- **`Expires`** es la alternativa clásica a `Max-Age`. `Max-Age` es más moderno.
- **Un solo `Set-Cookie` por cookie.** No se pueden combinar varias en una línea.

---

## 13. Cierre de sesión (logout)

- **Ruta típica:** `POST /api/logout`.
- **Pasos:**
  1. Leer el ID de la cookie.
  2. **Eliminar la sesión** del `map` del servidor.
  3. **Enviar una cookie con `Max-Age=0`** para que el navegador la borre.
  4. Responder con éxito.
- **Efecto:** Aunque alguien tenga el ID (por ejemplo, en un log antiguo), ya no vale, porque fue eliminado del servidor.
- **No basta con borrar la cookie del navegador:** El servidor debe invalidar la sesión para que el ID no sea reutilizable.

**Fragmento suelto (cabecera de borrado):**
```
Set-Cookie: sessionId=; Path=/; HttpOnly; Max-Age=0
```

- **El valor puede ser vacío.** Lo importante es `Max-Age=0`.
- **Opcional:** Incluir los mismos atributos (`Path`, `HttpOnly`, `SameSite`) para que coincidan con la cookie original.

---

## 14. Expiración de sesiones

### A. Expiración absoluta
- La sesión expira tras un tiempo fijo desde su creación (por ejemplo, 8 horas).
- `expira = creada + 8*3600;`
- Independiente de la actividad del usuario.

### B. Expiración por inactividad
- La sesión expira si el usuario no hace peticiones durante X tiempo (por ejemplo, 30 minutos).
- Se actualiza `ultimaActividad` en cada petición válida.
- Si `time(0) - ultimaActividad > 30*60`, la sesión se invalida.

### C. Combinación
- Lo más común es combinar ambas: expira a las 8 horas **o** tras 30 minutos de inactividad, lo que ocurra primero.

### D. Limpieza de sesiones expiradas
- **Opción 1:** Validar y eliminar en cada petición (ya visto).
- **Opción 2:** Un hilo o timer que recorra el `map` periódicamente y elimine las expiradas.
- **Opción 3:** Ninguna; las expiradas se van eliminando al detectarlas.

**Recomendación para esta actividad:** Validación en cada petición. Es suficiente y simple.

---

## 15. Trabajar con cookies desde JavaScript (opcional)

- **`document.cookie`:** Devuelve un string con todas las cookies del dominio accesibles desde JS (excluye las `HttpOnly`).
- **Formato:** `"cookie1=valor1; cookie2=valor2"`.
- **Limitación:** No puedes leer cookies `HttpOnly`. **Eso es una protección, no un problema.**
- **Para el cliente HTTP:** El navegador envía y recibe cookies automáticamente. **No tienes que hacer nada especial en `fetch`.**
- **`credentials`:** Por defecto, `fetch` **no envía cookies** en peticiones cross-origin. Para el mismo origen (como en tu caso), sí las envía por defecto.
  - Si tuvieras frontend y backend en distintos orígenes: `fetch(url, { credentials: "include" })`.
- **Para tu proyecto:** Frontend y backend en el mismo puerto → **las cookies viajan automáticamente** con cada `fetch`. No tienes que configurar nada.

**Nota:** No puedes leer la cookie `sessionId` desde JavaScript si tiene `HttpOnly`. **Y eso es lo deseado.** El navegador la envía automáticamente al servidor; JS no necesita verla.

---

## 16. Seguridad: ataques comunes y mitigaciones

### A. XSS (Cross-Site Scripting)
- **Ataque:** Un atacante inyecta JavaScript malicioso en tu página. Ese JS puede leer `document.cookie` y robar la cookie de sesión.
- **Mitigación:**
  - **`HttpOnly`** en la cookie → JS no puede leerla.
  - **Sanear** todo lo que el usuario ingresa y se muestra en HTML (usa `textContent`, no `innerHTML`).
  - **CSP (Content Security Policy)** en producción.

### B. CSRF (Cross-Site Request Forgery)
- **Ataque:** Un sitio malicioso hace peticiones al tuyo (con las cookies del usuario) sin que este lo sepa.
- **Mitigación:**
  - **`SameSite=Lax`** o `Strict` en la cookie.
  - **Token CSRF** en formularios (patrón adicional).
  - **Verificar el header `Origin` / `Referer`** en el servidor.

### C. Session Fixation
- **Ataque:** El atacante fija un ID de sesión conocido y engaña al usuario para que use ese ID. Cuando el usuario se loguea, el atacante ya conoce el ID.
- **Mitigación:** **Regenerar el ID de sesión** al hacer login (descartar el anterior y crear uno nuevo).

### D. Session Hijacking
- **Ataque:** El atacante roba el ID de sesión (por ejemplo, en una red insegura).
- **Mitigación:**
  - **`Secure`** → la cookie solo viaja por HTTPS.
  - **HTTPS** en producción.
  - **Expiración corta** de las sesiones.

### E. Adivinación del ID
- **Ataque:** El atacante intenta adivinar el ID por fuerza bruta.
- **Mitigación:** IDs largos y aleatorios (mínimo 128 bits).

### F. Timing attacks
- **Ataque:** Medir tiempos de respuesta para inferir información.
- **Mitigación:** Comparaciones de tiempo constante en validaciones sensibles (avanzado, no crítico aquí).

---

## 17. Contraseñas: repaso

- **Nunca en texto plano.** Aunque la BD actual las tenga así, en producción se guardan como hash.
- **Algoritmos recomendados:** bcrypt, argon2, PBKDF2, o sha256 con salt.
- **Salt:** Valor aleatorio único por usuario, añadido antes de hashear. Evita ataques de diccionario.
- **Comparación:** Compara el hash de la contraseña ingresada + salt con el hash almacenado.
- **No reveles** si el error fue "usuario no existe" o "contraseña incorrecta". Muestra un mensaje genérico: "Credenciales inválidas".
  - **Motivo:** No dar pistas al atacante sobre qué usuarios existen.

---

## 18. Estructura del servidor con sesiones

### A. Nuevos atributos del `Servidor`
- `unordered_map<string, Sesion> sesiones;`
- `time_t duracionSesion;` (opcional, para configurar la expiración).

### B. Nuevos métodos
- `string generarSessionId();` → devuelve un ID único.
- `void crearSesion(const string& usuario, const string& rol, string& sessionIdOut);` → genera ID, guarda en el map, devuelve el ID.
- `bool validarSesion(const string& sessionId, Sesion& salida);` → busca y valida.
- `void eliminarSesion(const string& sessionId);` → elimina del map.
- `string extraerSessionId(const PeticionHTTP& p);` → parsea la cabecera Cookie.
- `void establecerCookie(RespuestaHTTP& r, const string& nombre, const string& valor, int maxAge, bool httpOnly);` → añade la cabecera `Set-Cookie`.

### C. Modificaciones en `RespuestaHTTP`
- Un atributo `vector<string> setCookies;` o similar.
- Al serializar, emitir una cabecera `Set-Cookie` por cada cookie.
- **Motivo:** Puede haber varias cookies; no puedes concatenarlas.

### D. Modificaciones en `PeticionHTTP`
- Un método `string obtenerCookie(const string& nombre) const;` que parsea la cabecera `Cookie` y devuelve el valor.

---

## 19. Autenticación vs. autorización

- **Autenticación:** Verificar **quién** es el usuario (login con credenciales).
- **Autorización:** Verificar **qué puede hacer** el usuario (según su rol).
- **En esta actividad:** Se trata la autenticación (login + sesión).
- **Autorización vendrá después:** Por ejemplo, solo un `Administrador` puede eliminar usuarios. Se implementa comprobando el `rol` de la sesión antes de procesar la petición.
- **Regla:** Autentica primero, autoriza después. Nunca autorices sin autenticar.

---

## 20. Buenas prácticas

- **IDs de sesión largos y aleatorios** (mínimo 128 bits).
- **`HttpOnly`** siempre en cookies de sesión.
- **`Secure`** siempre en producción (HTTPS).
- **`SameSite=Lax`** como mínimo. `Strict` si tu app no necesita cross-site.
- **Expiración** de sesiones (absoluta y por inactividad).
- **Regenerar el ID de sesión** al hacer login (prevenir session fixation).
- **Invalidar la sesión** en el servidor al hacer logout, no solo borrar la cookie.
- **Nunca** guardes datos sensibles en la cookie (solo el ID).
- **Nunca** confíes en cookies que el cliente "podría haber modificado": valida siempre contra el `map`.
- **No expongas** si el usuario existe o no en los mensajes de error.
- **Limpia** sesiones expiradas periódicamente o al detectarlas.
- **Centraliza** la lógica de sesiones en funciones del `Servidor`.
- **Rutas públicas vs. protegidas:** Deja claro qué endpoints requieren sesión y cuáles no.
- **Log de actividad:** Registra logins y logouts exitosos y fallidos (útil para auditoría).
- **Considera cifrar la cookie** si guardas datos en ella (no lo necesitas si solo guardas el ID).
- **No compartas sesiones entre usuarios** en el mismo navegador: cada login crea una sesión nueva.
- **Actualiza `ultimaActividad`** en cada petición válida si usas expiración por inactividad.

---

## 21. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **La cookie no se envía al servidor** | El navegador la envía automáticamente. Verifica que el `Path` y el `Domain` permitan enviarla a esa ruta. |
| **La cookie se pierde tras cerrar el navegador** | Es una cookie de sesión (sin `Max-Age` ni `Expires`). Añade `Max-Age` para persistirla. |
| **No se envía la cookie en `fetch` cross-origin** | Añade `credentials: "include"` en `fetch` y configura CORS con `Access-Control-Allow-Credentials: true`. |
| **La cookie se ve en `document.cookie`** | No tiene `HttpOnly`. Añádelo. |
| **El ID de sesión es predecible** | Usa `random_device` + `mt19937_64` y 128+ bits. Nunca `rand()`. |
| **La sesión sigue válida tras logout** | No la eliminaste del `map`. Borra la entrada. |
| **Sesiones que crecen indefinidamente en memoria** | Implementa expiración y limpieza. |
| **`Set-Cookie` no aparece en la respuesta** | Revisa la serialización de `RespuestaHTTP`. Cada `Set-Cookie` es una cabecera separada. |
| **`Max-Age=0` no borra la cookie** | Asegúrate de enviar los mismos atributos (`Path`, `Domain`) que la cookie original. |
| **El cliente puede modificar el ID y hacerse pasar por otro** | Solo si el ID está bien generado y validado, no debería. Verifica la lógica. |
| **Sesiones compartidas entre usuarios** | Cada login debe crear un ID nuevo. Nunca reutilices uno existente. |
| **Falta `SameSite`** | Añádelo. `Lax` es un buen default. |
| **Contraseña en texto plano** | En producción, usa hash con salt. |
| **`401` vs `403`** | 401 = no autenticado. 403 = autenticado pero sin permiso. |
| **La cookie no se borra al logout** | Verifica que envías `Set-Cookie` con `Max-Age=0` y el mismo `Path`. |
| **Sesión muy larga (días)** | Acorta `Max-Age`. Cuanto más corto, más seguro. |
| **Reutilizar IDs de sesión tras expirar** | No reutilices. Genera uno nuevo en cada login. |

---

## 22. Verificación

Después de implementar sesiones:

1. **Login exitoso:** el servidor responde `200` con `Set-Cookie: sessionId=...`.
2. **DevTools → Application → Cookies:** verifica que la cookie aparece, con `HttpOnly` marcado.
3. **DevTools → Network:** verifica que en las siguientes peticiones el navegador envía `Cookie: sessionId=...`.
4. **Petición sin cookie:** el servidor responde `401 Unauthorized`.
5. **Petición con cookie inválida:** el servidor responde `401`.
6. **Petición con cookie válida:** el servidor responde con los datos.
7. **Logout:** el servidor responde `200` con `Set-Cookie` de borrado. La cookie desaparece del navegador.
8. **Petición después del logout:** el servidor responde `401`.
9. **Reinicio del servidor:** las sesiones en memoria se pierden (los usuarios deben volver a loguearse). Tenlo en cuenta.
10. **Expiración:** espera a que expire la sesión (o configura un tiempo muy corto) y verifica que responde `401`.

---

## 23. Resumen de conceptos clave

- **HTTP sin estado:** Cada petición es independiente. Cookies y sesiones resuelven esto.
- **Cookie:** Par clave-valor enviado por el servidor (`Set-Cookie`) y devuelto por el navegador (`Cookie`).
- **Sesión:** Estado en el servidor asociado a un ID. El ID viaja en la cookie.
- **Login:** Valida credenciales → genera ID → guarda sesión → envía `Set-Cookie`.
- **Peticiones posteriores:** El navegador envía la cookie → el servidor busca el ID → identifica al usuario.
- **Logout:** Elimina la sesión del servidor + borra la cookie con `Max-Age=0`.
- **Atributos de cookie:** `Path`, `Max-Age`, `Secure`, `HttpOnly`, `SameSite`.
- **Generación de ID:** 128+ bits, aleatorio, con `random_device` + `mt19937_64`.
- **Almacenamiento:** `unordered_map<string, Sesion>` en el servidor.
- **Validación:** Buscar en el map, verificar expiración.
- **Seguridad:** `HttpOnly` contra XSS, `SameSite` contra CSRF, `Secure` contra sniffing, regeneración contra fixation.
- **Frontend:** El navegador gestiona las cookies automáticamente. `fetch` del mismo origen las envía sin configuración.
- **No guardes datos sensibles en la cookie.** Solo el ID.
- **Contraseñas:** Nunca en texto plano en producción. Hash + salt.

---
# PASO A PASO

---
# Actividad 38 — Sesiones mediante cookies

## Objetivo
Después del login, el servidor genera un **ID de sesión**, lo envía en una cookie, y reconoce automáticamente al usuario en peticiones posteriores. Añadir logout.

---

## Paso 1 — Struct `Sesion` y contenedor en `Servidor`

Antes de la clase `Servidor`:

```cpp
#include <unordered_map>
#include <random>
#include <ctime>

struct Sesion {
    string usuario;
    string rol;
    time_t creada;
    time_t ultimaActividad;
    time_t expira;
};
```

Dentro de `Servidor`, agrega:

```cpp
private:
    std::unordered_map<string, Sesion> sesiones;
    static const int DURACION_SESION = 3600;   // 1 hora
```

---

## Paso 2 — Generar ID de sesión

```cpp
string Servidor::generarSessionId() {
    static std::random_device rd;
    static std::mt19937_64 gen(rd());
    static std::uniform_int_distribution<uint64_t> dist;

    uint64_t a = dist(gen);
    uint64_t b = dist(gen);

    char buf[33];
    snprintf(buf, sizeof(buf), "%016llx%016llx",
             (unsigned long long)a, (unsigned long long)b);
    return string(buf);
}
```

🎯 128 bits de aleatoriedad criptográfica. **Nunca uses `rand()` ni `time(0)` solos.**

---

## Paso 3 — Crear sesión

```cpp
string Servidor::crearSesion(const string& usuario, const string& rol) {
    string id = generarSessionId();
    time_t ahora = time(nullptr);

    sesiones[id] = Sesion{
        usuario,
        rol,
        ahora,
        ahora,
        ahora + DURACION_SESION
    };

    return id;
}
```

---

## Paso 4 — Validar sesión

```cpp
bool Servidor::validarSesion(const string& id, Sesion& salida) {
    auto it = sesiones.find(id);
    if (it == sesiones.end()) return false;

    time_t ahora = time(nullptr);
    if (ahora > it->second.expira) {
        sesiones.erase(it);
        return false;
    }

    it->second.ultimaActividad = ahora;
    salida = it->second;
    return true;
}
```

---

## Paso 5 — Parsear la cabecera `Cookie`

```cpp
string Servidor::extraerCookie(const string& peticion, const string& nombre) {
    size_t pos = peticion.find("Cookie:");
    if (pos == string::npos) return "";

    size_t fin = peticion.find("\r\n", pos);
    if (fin == string::npos) fin = peticion.size();

    string cookies = peticion.substr(pos + 7, fin - pos - 7);

    stringstream ss(cookies);
    string par;
    while (getline(ss, par, ';')) {
        // trim
        size_t i = par.find_first_not_of(" \t");
        if (i != string::npos) par = par.substr(i);

        size_t igual = par.find('=');
        if (igual == string::npos) continue;

        string key   = par.substr(0, igual);
        string value = par.substr(igual + 1);

        if (key == nombre) return value;
    }
    return "";
}
```

---

## Paso 6 — Cabecera `Set-Cookie` en la respuesta

Necesitamos poder añadir cabeceras extra a la respuesta. Cambia la firma de `construirRespuesta` (o añade una sobrecarga):

```cpp
string construirRespuestaConCookie(int codigo, const string& mime,
                                   const string& cuerpo, const string& setCookie) {
    string estado = (codigo == 200) ? "200 OK"
                  : (codigo == 401) ? "401 Unauthorized"
                  : "500 Internal Server Error";

    string r = "HTTP/1.1 " + estado + "\r\n"
             + "Content-Type: " + mime + "\r\n"
             + "Content-Length: " + to_string(cuerpo.size()) + "\r\n"
             + "Connection: close\r\n";

    if (!setCookie.empty()) {
        r += "Set-Cookie: " + setCookie + "\r\n";
    }

    r += "\r\n" + cuerpo;
    return r;
}
```

⚠️ **Cada `Set-Cookie` es su propia cabecera.** No se pueden concatenar varias cookies en una sola cabecera `Set-Cookie`.

---

## Paso 7 — Modificar `/api/login` para crear sesión

```cpp
if (ruta == "/api/login" && metodo == "POST") {
    Usuario u = parsearUsuarioLogin(cuerpo);
    Usuario encontrado = baseDatos.buscarUsuario(u.getUsuario());

    if (encontrado.getUsuario().empty() ||
        encontrado.getContrasena() != u.getContrasena()) {
        return construirRespuesta(401, "application/json; charset=utf-8",
                                  "{\"ok\":false,\"error\":\"Credenciales inválidas\"}");
    }

    string sessionId = crearSesion(encontrado.getUsuario(), encontrado.getRol());

    string setCookie = "sessionId=" + sessionId + "; Path=/; HttpOnly; SameSite=Lax; Max-Age="
                     + to_string(DURACION_SESION);

    string json = "{\"ok\":true,"
                  "\"nombre\":\"" + escaparJSON(encontrado.getNombre()) + "\","
                  "\"rol\":\""    + escaparJSON(encontrado.getRol())    + "\"}";

    cout << "[LOGIN OK] " << encontrado.getUsuario()
         << " sesión=" << sessionId.substr(0, 8) << "...\n";

    return construirRespuestaConCookie(200, "application/json; charset=utf-8", json, setCookie);
}
```

📌 Observa los atributos de la cookie:
- **`HttpOnly`** → no accesible desde JS (protección XSS).
- **`SameSite=Lax`** → mitigación CSRF.
- **`Max-Age`** → duración.
- **`Path=/`** → envío a todo el sitio.
- *(Producción: añadir `Secure` para que solo viaje por HTTPS.)*

---

## Paso 8 — Proteger rutas

Antes de manejar `/api/alumnos`, `/api/usuarios`, etc., verifica la sesión:

```cpp
// Al inicio de manejarAPI, para rutas protegidas
bool rutaProtegida = (ruta == "/api/alumnos" || ruta == "/api/usuarios");

if (rutaProtegida && ruta != "/api/login" && ruta != "/api/logout") {
    string sessionId = extraerCookie(peticion, "sessionId");
    Sesion s;

    if (sessionId.empty() || !validarSesion(sessionId, s)) {
        return construirRespuesta(401, "application/json; charset=utf-8",
                                  "{\"ok\":false,\"error\":\"No autenticado\"}");
    }

    // Guarda s.usuario y s.rol si el handler los necesita
}
```

⚠️ Para que `manejarAPI` tenga acceso a `peticion` (necesita la cabecera `Cookie`), pásala como parámetro:

```cpp
string manejarAPI(const string& peticion, const string& ruta,
                  const string& metodo, const string& cuerpo);
```

O extrae la cookie antes y pasa el `Sesion` ya validado.

---

## Paso 9 — Endpoint `POST /api/logout`

```cpp
if (ruta == "/api/logout" && metodo == "POST") {
    string sessionId = extraerCookie(peticion, "sessionId");
    if (!sessionId.empty()) {
        sesiones.erase(sessionId);
        cout << "[LOGOUT] sesión " << sessionId.substr(0, 8) << "...\n";
    }

    // Cookie con Max-Age=0 → el navegador la borra
    string setCookie = "sessionId=; Path=/; HttpOnly; SameSite=Lax; Max-Age=0";
    return construirRespuestaConCookie(200, "application/json; charset=utf-8",
                                       "{\"ok\":true}", setCookie);
}
```

📌 **No basta con borrar la cookie del navegador.** También hay que eliminar la sesión del `map` del servidor, o el ID seguiría siendo válido si alguien lo copia.

---

## Paso 10 — Verificar quién es el usuario en peticiones protegidas

Ejemplo: `GET /api/perfil` devuelve los datos de la sesión:

```cpp
if (ruta == "/api/perfil" && metodo == "GET") {
    string sessionId = extraerCookie(peticion, "sessionId");
    Sesion s;
    if (!validarSesion(sessionId, s)) {
        return construirRespuesta(401, "application/json; charset=utf-8",
                                  "{\"error\":\"No autenticado\"}");
    }

    string json = "{\"usuario\":\"" + escaparJSON(s.usuario) + "\","
                  "\"rol\":\""     + escaparJSON(s.rol)     + "\"}";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

🎯 Esta es la prueba definitiva de que la sesión funciona.

---

## Paso 11 — Frontend: el navegador envía la cookie automáticamente

En `app.js`, **no hace falta nada especial**. El navegador adjunta automáticamente la cookie `sessionId` a cada `fetch` al mismo origen.

```js
// Ejemplo: pedir el perfil al cargar el panel
fetch('/api/perfil')
  .then(res => {
    if (res.status === 401) {
      window.location.href = '/login.html';
      return;
    }
    return res.json();
  })
  .then(data => {
    document.querySelector('#usuario-nombre').textContent = data.usuario;
  });
```

⚠️ Si el frontend y el backend estuvieran en orígenes distintos:
```js
fetch(url, { credentials: 'include' })
```
Pero en este proyecto, frontend y backend comparten puerto → **nada especial**.

---

## Paso 12 — Botón de logout

En `panel-alumno.html` (o cualquier panel):

```html
<button id="btn-logout">Cerrar sesión</button>
```

Y en `app.js`:

```js
document.querySelector('#btn-logout')?.addEventListener('click', async () => {
  await fetch('/api/logout', { method: 'POST' });
  window.location.href = '/login.html';
});
```

---

## Paso 13 — Verificar en DevTools

1. **Login** en `http://localhost:8080/login.html` con credenciales válidas.
2. **Application → Cookies → http://localhost:8080**: debe aparecer `sessionId`, con `HttpOnly` marcado. ✅ **No** aparece en `document.cookie`.
3. **Network**: en cualquier petición posterior a `/api/`, verifica `Cookie: sessionId=...` en Request Headers.
4. **Logout**: la cookie desaparece del navegador, y `POST /api/logout` la elimina del `map`.
5. Prueba `GET /api/perfil` tras logout → `401`.

Prueba con `curl`:

```bash
# Login (guarda cookie en archivo)
curl -i -c cookies.txt -X POST -H "Content-Type: application/json" \
  -d '{"usuario":"ana_lopez","contrasena":"secreta123"}' \
  http://localhost:8080/api/login

# Usar la cookie
curl -i -b cookies.txt http://localhost:8080/api/perfil

# Logout
curl -i -b cookies.txt -X POST http://localhost:8080/api/logout

# Después del logout → 401
curl -i -b cookies.txt http://localhost:8080/api/perfil
```

---

## Paso 14 — Seguridad: ya aplicada vs. pendiente

| Aspecto | Estado | Nota |
|---------|--------|------|
| ID aleatorio 128 bits | ✅ | `random_device` + `mt19937_64` |
| `HttpOnly` | ✅ | Cookie no accesible desde JS |
| `SameSite=Lax` | ✅ | Mitiga CSRF básico |
| `Secure` | ❌ | Requiere HTTPS (producción) |
| Expiración | ✅ | 1 hora |
| Regenerar ID al login | ✅ | Nuevo ID en cada login |
| Invalidar sesión al logout | ✅ | Se borra del `map` |
| HTTPS | ❌ | Deuda técnica para producción |
| Tokens CSRF explícitos | ❌ | `SameSite=Lax` ayuda, pero no es completo |
| Hash de contraseña | ❌ | Deuda técnica |

---

## Paso 15 — Nota sobre reinicios del servidor

Las sesiones viven en **memoria del servidor**. Al reiniciar:
- Todas las sesiones se pierden.
- Los usuarios deben volver a loguearse.
- Es lo **esperado** en esta actividad. 

Para persistir sesiones, se usaría una tabla `sesiones` en SQLite o Redis. Es trabajo futuro.

---

## ✅ Checklist de la Actividad 38

- [ ] Login genera `sessionId` y lo envía en `Set-Cookie`.
- [ ] La cookie tiene `HttpOnly` y `SameSite=Lax`.
- [ ] Las rutas protegidas validan la sesión.
- [ ] `GET /api/perfil` devuelve el usuario autenticado.
- [ ] `POST /api/logout` elimina la sesión del `map` **y** envía `Max-Age=0`.
- [ ] Tras logout, las peticiones protegidas devuelven `401`.
- [ ] En DevTools se ve la cookie con `HttpOnly` ✅.
- [ ] Se loguea el login/logout en consola del servidor (sin la contraseña).

## 🧠 Mini-reto
Implementa **expiración por inactividad**: si pasan 30 minutos sin peticiones, la sesión se invalida aunque no haya llegado su `expira` absoluto. Compara `time(nullptr) - ultimaActividad > 1800`.
