# Actividad 37. Autenticación

## Temas

- Login
    
- Validación
    

## Ejercicio

Agregar una página:

```text
login.html
```

Implementar:

```text
POST /api/login
```

El servidor validará usuario y contraseña consultando el archivo de usuarios.

Responder indicando si las credenciales son válidas.

---
# TEORÍA

---

## 1. ¿Qué es la autenticación?

- **Definición:** Proceso de verificar que un usuario **es quien dice ser**. Se basa en la presentación de credenciales (usuario + contraseña, tokens, certificados).
- **Diferencia con autorización:** La autenticación verifica **quién eres**; la autorización verifica **qué puedes hacer**. Primero te autenticas, después el sistema te autoriza.
- **En el proyecto:** Login con usuario y contraseña. Si las credenciales coinciden con un registro en la BD, el usuario está autenticado.
- **Importante:** Autenticación ≠ cifrado. La autenticación valida identidad; el cifrado (HTTPS) protege la transmisión. Sin HTTPS, las credenciales viajan en claro por la red. En un proyecto real, HTTPS es obligatorio.

---

## 2. Flujo completo del login

1. El usuario abre `login.html`.
2. Rellena usuario y contraseña.
3. El navegador envía `POST /api/login` con las credenciales en JSON.
4. El servidor recibe la petición, parsea el JSON.
5. Busca el usuario en la BD por nombre de usuario.
6. Compara la contraseña recibida con la almacenada.
7. Responde indicando éxito o fallo.
8. El frontend reacciona: redirige al panel o muestra error.

**Observación clave:** El servidor **no** autentica al usuario enviando su contraseña de vuelta. Solo confirma o niega la validez. Y en proyectos reales, además, emite un **token** para que las siguientes peticiones se identifiquen sin reenviar la contraseña.

---

## 3. La página `login.html`

### A. Estructura
- Formulario con dos campos: `usuario` y `contrasena`.
- Botón "Iniciar sesión".
- Área para mostrar mensajes (éxito o error).
- Opcional: enlace a "Registrarse" o "Olvidé mi contraseña".

### B. Atributos importantes
- **`<input type="text" name="usuario">`**: campo de usuario.
- **`<input type="password" name="contrasena">`**: campo de contraseña (oculta lo escrito).
- **`autocomplete="username"`** y **`autocomplete="current-password"`**: ayudan a los gestores de contraseñas.
- **`required`**: validación nativa.
- **`id`** en cada campo y `for` en el `<label>`: accesibilidad.

### C. Consideraciones UX
- No reveles si el error fue en el usuario o en la contraseña. Un mensaje genérico "Credenciales inválidas" evita filtrar información.
- Deshabilita el botón mientras se procesa la petición.
- Muestra "Cargando..." si la petición tarda.
- Limpia el campo de contraseña tras un fallo (opcional, pero común).

---

## 4. Ruta `POST /api/login`

### A. Por qué POST
- Las credenciales **no** deben ir en la URL. GET las expondría en logs, historial e intermediarios.
- POST envía los datos en el cuerpo, no en la URL.

### B. Estructura del cuerpo
- JSON con los campos mínimos:
  ```
  {"usuario":"ana_lopez","contrasena":"secreta123"}
  ```
- **No** incluyas campos innecesarios. El servidor solo necesita esos dos.

### C. Content-Type
- `application/json`. El servidor lo verifica y rechaza con `415 Unsupported Media Type` si no coincide.

### D. Enrutamiento
- Añade un caso en `manejarAPI` para `/api/login` con método POST.
- Como es una ruta fija, `if (ruta == "/api/login" && metodo == "POST")` es suficiente.

---

## 5. Validación en el servidor

### A. Pasos
1. Leer el cuerpo completo.
2. Parsear el JSON.
3. Extraer `usuario` y `contrasena`.
4. Verificar que no estén vacíos.
5. Buscar el usuario en la BD.
6. Comparar la contraseña.
7. Responder.

### B. Validaciones obligatorias
- **Campos presentes:** si falta alguno → `400 Bad Request`.
- **Tipos correctos:** ambos deben ser strings no vacíos.
- **Longitudes razonables:** evita procesar contraseñas de 10MB.
- **Content-Type correcto:** `application/json`.
- **Método correcto:** POST. Si es GET → `405 Method Not Allowed`.

### C. Lo que NO debes hacer
- **No** revelar si el usuario existe o no. Siempre "Credenciales inválidas".
- **No** loguear la contraseña recibida en consola. Loguea solo el intento y el resultado.
- **No** comparar con `==` si usas hashes. Usa comparación de tiempo constante (ver sección 8).
- **No** devolver la contraseña ni el hash en la respuesta.

---

## 6. Respuesta del servidor

### A. Casos posibles

| Caso | Código HTTP | Cuerpo |
|------|-------------|--------|
| Credenciales válidas | 200 OK | `{"ok":true,"mensaje":"Bienvenido Ana"}` |
| Credenciales inválidas | 401 Unauthorized | `{"ok":false,"error":"Credenciales inválidas"}` |
| Datos mal formados | 400 Bad Request | `{"ok":false,"error":"Datos incompletos"}` |
| Content-Type incorrecto | 415 Unsupported | `{"ok":false,"error":"Formato no soportado"}` |
| Método incorrecto | 405 | `{"ok":false,"error":"Método no permitido"}` |
| Error interno | 500 | `{"ok":false,"error":"Error del servidor"}` |

### B. Códigos clave
- **200 OK:** Todo bien, el usuario está autenticado.
- **401 Unauthorized:** Credenciales incorrectas. **Es el código correcto** para este caso, no 400 ni 403.
- **403 Forbidden:** Reservado para "autenticado pero sin permisos". No lo uses aquí.

### C. Contenido de la respuesta
- **Éxito:** mensaje de bienvenida con el nombre del usuario (opcional), y el `rol` si lo vas a usar en el frontend para redirigir al panel correcto.
- **Fallo:** mensaje genérico. Nunca detalles internos.

**Fragmento suelto (respuesta de éxito conceptual):**
```json
{
  "ok": true,
  "usuario": "ana_lopez",
  "nombre": "Ana López",
  "rol": "alumno"
}
```

- **Utilidad del `rol`:** El frontend puede redirigir a `/panel-alumno.html`, `/panel-maestro.html` o `/panel-admin.html` según el rol.

---

## 7. Contraseñas: el problema de guardarlas

### A. Nunca en texto plano
- Guardar contraseñas en texto plano es una **mala práctica grave**. Si la BD se filtra, todas las cuentas quedan comprometidas.
- Aunque en esta actividad didáctica se tolere, menciónalo como deuda técnica.

### B. Hashing
- **Definición:** Función de una sola dirección. Dado un input, produce un output fijo. No se puede revertir.
- **Algoritmos recomendados:** bcrypt, argon2, scrypt. **Evita** MD5 y SHA-1 (obsoletos) y SHA-256 solo (sin salt) para contraseñas.
- **Salt:** Valor aleatorio único por usuario, añadido a la contraseña antes de hashear. Evita ataques de diccionario y rainbow tables.
- **Almacenamiento:** Se guardan el `hash` y el `salt` (o un string combinado tipo `$2b$10$...` en bcrypt).

### C. Flujo de autenticación con hash
1. Al registrar: generar salt, calcular `hash = H(contrasena + salt)`, guardar `hash` y `salt`.
2. Al hacer login: recuperar `hash` y `salt` del usuario, calcular `H(contrasenaRecibida + salt)`, comparar con el `hash` almacenado.
3. Comparación con tiempo constante para evitar ataques de temporización.

### D. Comparación de tiempo constante
- **Problema:** Comparar strings con `==` puede filtrar información por el tiempo que tarda en detectar la diferencia.
- **Solución:** Comparar byte a byte sin cortocircuito. Bibliotecas como `libsodium` ofrecen `crypto_pwhash_str_verify` y funciones similares.
- **Para esta actividad:** Si usas texto plano, compara con `==`. Si usas hash, usa una función de comparación de tiempo constante.

### E. Para esta actividad
- Si el proyecto es pequeño y académico, guardar en texto plano es aceptable, **documentándolo como deuda técnica**.
- Si quieres hacerlo bien desde el principio, integra una biblioteca de hashing. bcrypt es relativamente fácil de añadir.

---

## 8. Búsqueda del usuario en la BD

- **Método:** `buscarUsuario(const string& usuario)` en `BaseDatosSQL`.
- **Consulta preparada:** `SELECT * FROM usuarios WHERE usuario = ?`.
- **Resultado:** Devuelve un `Usuario` o un indicador de "no encontrado".
- **Uso:**
  ```cpp
  Usuario u = baseDatos.buscarUsuario(usuarioRecibido);
  if (u.getUsuario().empty()) {
      // no existe → 401
  } else if (u.getContrasena() == contrasenaRecibida) {
      // login OK
  } else {
      // contraseña incorrecta → 401
  }
  ```
- **Indistinción de errores:** Ambos casos (usuario no existe / contraseña incorrecta) devuelven **el mismo mensaje** al cliente.

---

## 9. Lado cliente: enviar el login con `fetch`

- **Interceptar el `submit` del formulario** con `e.preventDefault()`.
- **Leer los valores** de los inputs.
- **Construir el objeto** con `usuario` y `contrasena`.
- **Serializar con `JSON.stringify`.**
- **`fetch` con POST**, `Content-Type: application/json`, body serializado.
- **Leer la respuesta con `.json()`** porque el servidor responde JSON.
- **Verificar `response.ok`** y actuar en consecuencia:
  - Si `ok`: redirigir o mostrar mensaje de bienvenida.
  - Si `401`: mostrar "Credenciales inválidas".
  - Si otro error: mensaje genérico.
- **Deshabilitar el botón** mientras la petición está en curso.

**Fragmento suelto (patrón):**
```js
formulario.addEventListener("submit", async (e) => {
    e.preventDefault();
    const datos = {
        usuario: document.querySelector("#usuario").value,
        contrasena: document.querySelector("#contrasena").value
    };
    // fetch POST
});
```

---

## 10. Redirección tras login exitoso

- **Opción 1: Redirigir según el rol.**
  - Si `rol === "alumno"` → `window.location.href = "/panel-alumno.html";`
  - Si `rol === "maestro"` → `/panel-maestro.html`.
  - Si `rol === "admin"` → `/panel-admin.html`.
- **Opción 2: Redirigir a una sola página** y que esa página consulte el rol para mostrar el contenido adecuado.
- **Consideración importante:** Redirigir en el cliente **no autentica** nada. Cualquier usuario puede escribir la URL directamente. La autenticación real ocurre en el servidor en **cada petición posterior**. Este es el gran tema de la sesión y los tokens (ver sección 12).

---

## 11. Manejo de errores en el cliente

- **Credenciales inválidas (401):** Mensaje genérico "Usuario o contraseña incorrectos". No detalles.
- **Campos vacíos:** Validación previa en JS antes de enviar. Muestra mensaje.
- **Servidor caído:** `fetch` rechaza la promesa → "No se pudo conectar con el servidor".
- **Error del servidor (500):** Mensaje genérico "Error del servidor. Intenta de nuevo".
- **Nunca muestres el mensaje crudo** del servidor al usuario. Traduce a lenguaje amigable.
- **Nunca muestres detalles técnicos** (códigos de error de SQLite, rutas de archivos).

---

## 12. Sesiones y tokens (introducción)

- **Problema:** Después del login, ¿cómo sabe el servidor que la siguiente petición viene del mismo usuario?
- **HTTP es sin estado:** Cada petición es independiente. No hay memoria entre ellas por defecto.
- **Soluciones comunes:**
  - **Cookies de sesión:** El servidor crea una sesión, guarda el ID en una cookie, el navegador la envía en cada petición.
  - **Tokens (JWT u otros):** El servidor emite un token firmado tras el login. El cliente lo guarda (memoria o `localStorage`) y lo envía en la cabecera `Authorization: Bearer <token>` en cada petición.
- **Para esta actividad:** No es obligatorio implementar sesiones. El login solo verifica credenciales. Las demás rutas siguen siendo públicas. Documenta esto como "trabajo futuro".
- **Nota de seguridad:** `localStorage` es vulnerable a XSS. Para tokens sensibles, se prefieren cookies `HttpOnly` + `Secure` + `SameSite`.

---

## 13. HTTPS: la capa que falta

- **Recordatorio:** Sin HTTPS, las credenciales viajan en claro por la red. Cualquiera en la misma red puede leerlas.
- **En desarrollo:** `http://localhost` es tolerable porque el tráfico no sale de tu máquina.
- **En producción:** HTTPS con TLS es **obligatorio**. Se obtiene con Let's Encrypt o certificados comerciales.
- **Implementar HTTPS en C++ crudo** es complejo (OpenSSL, mbedTLS). Para el proyecto académico, no es necesario, pero **menciónalo** en el análisis.
- **Alternativa moderna:** Poner un proxy inverso (nginx, Caddy) delante del servidor C++ que gestione TLS.

---

## 14. Ataques comunes y defensas

### A. Fuerza bruta
- **Ataque:** Probar muchas combinaciones de usuario/contraseña.
- **Defensa:** Rate limiting (limitar intentos por IP), bloqueo temporal tras N intentos fallidos, CAPTCHA.
- **Para esta actividad:** No lo implementes, pero menciónalo.

### B. Inyección SQL
- **Ataque:** Enviar `' OR '1'='1` como usuario o contraseña.
- **Defensa:** Consultas preparadas. **Siempre.** Nunca concatenes strings para formar SQL.

### C. Enumeración de usuarios
- **Ataque:** Diferenciar si el usuario existe por el mensaje de error.
- **Defensa:** Mensaje genérico "Credenciales inválidas" en ambos casos.

### D. Timing attacks
- **Ataque:** Medir el tiempo de respuesta para inferir información.
- **Defensa:** Comparación de tiempo constante para contraseñas hasheadas.

### E. XSS
- **Ataque:** Inyectar scripts maliciosos en la página.
- **Defensa:** Escapar la salida. En el servidor, escapa strings al construir JSON. En el cliente, usa `textContent` en lugar de `innerHTML`.

### F. CSRF
- **Ataque:** Forzar al navegador a enviar una petición autenticada sin el consentimiento del usuario.
- **Defensa:** Tokens CSRF, cookies `SameSite=Strict`.

### G. Fuerza bruta sobre hashes
- **Ataque:** Con el hash y el salt, probar contraseñas comunes offline.
- **Defensa:** Contraseñas largas y complejas, hashes lentos (bcrypt, argon2).

---

## 15. Validación en el cliente y en el servidor

- **Cliente:** Validación de campos no vacíos, longitud mínima. Mejora la UX.
- **Servidor:** Validación **obligatoria**. Nunca confíes en el cliente. Un atacante puede enviar lo que quiera con `curl`.
- **Regla de oro:** Si el cliente lo valida, es por UX. Si el servidor lo valida, es por seguridad.

---

## 16. Logging y auditoría

- **Qué loguear:**
  - Intentos de login (usuario, resultado, timestamp).
  - IP de origen (si está disponible).
  - No loguees la contraseña ni el hash.
- **Para qué:** Detectar ataques, auditar accesos, depurar.
- **En el proyecto:** Un log en consola basta. En producción, un archivo de log estructurado (JSON) o un sistema de logs centralizado (ELK, Loki).

---

## 17. Buenas prácticas

- **POST para login, nunca GET.**
- **Credenciales en el cuerpo, nunca en la URL.**
- **Mensaje genérico en errores de credenciales.** No reveles si el usuario existe.
- **Nunca guardes contraseñas en texto plano en producción.** Usa hash + salt.
- **Consultas preparadas.** Evita inyección SQL.
- **Verifica el Content-Type** y rechaza con 415 si no es `application/json`.
- **Responde 401 para credenciales inválidas**, no 400 ni 403.
- **No expongas detalles internos** en las respuestas.
- **No loguees la contraseña.**
- **Rate limiting en producción.** Aunque no lo implementes en el proyecto, menciónalo.
- **HTTPS en producción.** Crítico.
- **Valida en el cliente y en el servidor.** El cliente para UX; el servidor por seguridad.
- **Redirige después del login** según el rol, pero recuerda que la autenticación real ocurre en cada petición posterior.
- **Considera sesiones o tokens** para peticiones posteriores. En esta actividad, documéntalo como siguiente paso.
- **No confíes en el cliente para nada crítico.** Ni para validar, ni para decidir permisos.
- **Prueba casos borde:** campos vacíos, usuario inexistente, contraseña incorrecta, mayúsculas/minúsculas en el usuario (decide si es case-insensitive y sé consistente).
- **Documenta el comportamiento** esperado del endpoint en el README.

---

## 18. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Contraseñas en texto plano** | Usa hash + salt en producción. Documenta la deuda técnica si es académico. |
| **Concatenar strings para SQL** | Consultas preparadas. Siempre. |
| **Mensaje distinto si el usuario existe vs. si la contraseña es incorrecta** | Mismo mensaje genérico en ambos casos. Evita enumeración. |
| **Loguear la contraseña** | Nunca. Loguea solo el intento y el resultado. |
| **Login por GET** | Las credenciales en la URL quedan en logs e historial. Usa POST. |
| **Redirigir sin autenticar en cada petición** | El login solo verifica credenciales. Cada ruta protegida debe verificar sesión o token. |
| **Devolver 200 con `ok:false`** | Usa 401 para credenciales inválidas. Los

---
# PASO A PASO

---
# Actividad 37 — Autenticación (login)

## Objetivo
Crear `login.html` y `POST /api/login`. El servidor valida usuario y contraseña contra SQLite. Responde si las credenciales son válidas o no.

---

## Paso 1 — Página `login.html`

Crea `public/login.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Iniciar sesión</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <main class="login-container">
    <h1>Iniciar sesión</h1>

    <form id="form-login">
      <label for="usuario">Usuario:</label>
      <input type="text" id="usuario" name="usuario"
             autocomplete="username" required>

      <label for="contrasena">Contraseña:</label>
      <input type="password" id="contrasena" name="contrasena"
             autocomplete="current-password" required>

      <button type="submit" id="btn-login">Iniciar sesión</button>
    </form>

    <p id="login-mensaje" class="mensaje"></p>
  </main>

  <script src="/app.js"></script>
</body>
</html>
```

⚠️ `type="password"` oculta lo escrito. `autocomplete` ayuda a los gestores de contraseñas.

---

## Paso 2 — Endpoint `POST /api/login` en el servidor

Agrega dentro de `manejarAPI`:

```cpp
if (ruta == "/api/login" && metodo == "POST") {
    Usuario u = parsearUsuarioLogin(cuerpo);

    if (u.getUsuario().empty() || u.getContrasena().empty()) {
        return construirRespuesta(400, "application/json; charset=utf-8",
                                  "{\"ok\":false,\"error\":\"Datos incompletos\"}");
    }

    Usuario encontrado = baseDatos.buscarUsuario(u.getUsuario());

    // ⚠️ Importante: el mismo mensaje para ambos casos (usuario no existe / contraseña mal)
    if (encontrado.getUsuario().empty() ||
        encontrado.getContrasena() != u.getContrasena()) {
        cout << "[LOGIN FALLIDO] " << u.getUsuario() << "\n";
        return construirRespuesta(401, "application/json; charset=utf-8",
                                  "{\"ok\":false,\"error\":\"Credenciales inválidas\"}");
    }

    cout << "[LOGIN OK] " << u.getUsuario() << " (" << encontrado.getRol() << ")\n";

    // No devolvemos la contraseña; solo lo necesario
    string json = "{\"ok\":true,"
                  "\"usuario\":\"" + escaparJSON(encontrado.getUsuario()) + "\","
                  "\"nombre\":\""  + escaparJSON(encontrado.getNombre())  + "\","
                  "\"rol\":\""     + escaparJSON(encontrado.getRol())     + "\"}";

    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

📌 `401 Unauthorized` es el código correcto para credenciales inválidas. **No** `400` ni `403`.

---

## Paso 3 — Parsear el JSON del cuerpo

Usa la función `extraerValorJSON` que ya tienes:

```cpp
Usuario parsearUsuarioLogin(const string& cuerpo) {
    return Usuario(
        "",   // nombre no viene en el login
        extraerValorJSON(cuerpo, "usuario"),
        extraerValorJSON(cuerpo, "contrasena"),
        ""    // rol no viene
    );
}
```

---

## Paso 4 — Frontend: enviar el login con fetch

Agrega a `app.js`:

```js
const formLogin = document.querySelector('#form-login');
if (formLogin) {
  formLogin.addEventListener('submit', async (e) => {
    e.preventDefault();

    const mensaje = document.querySelector('#login-mensaje');
    const boton   = document.querySelector('#btn-login');
    boton.disabled = true;
    mensaje.textContent = 'Verificando...';

    const datos = {
      usuario:    document.querySelector('#usuario').value.trim(),
      contrasena: document.querySelector('#contrasena').value
    };

    try {
      const res = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(datos)
      });

      const respuesta = await res.json();

      if (!res.ok) {
        mensaje.textContent = respuesta.error || 'Credenciales inválidas';
        mensaje.className = 'mensaje error';
        document.querySelector('#contrasena').value = '';
        return;
      }

      // Login exitoso
      mensaje.textContent = `Bienvenido, ${respuesta.nombre}`;
      mensaje.className = 'mensaje exito';

      // Redirigir según el rol
      setTimeout(() => {
        if (respuesta.rol === 'admin') {
          window.location.href = '/panel-admin.html';
        } else if (respuesta.rol === 'maestro') {
          window.location.href = '/panel-maestro.html';
        } else {
          window.location.href = '/panel-alumno.html';
        }
      }, 800);

    } catch (err) {
      mensaje.textContent = 'No se pudo conectar con el servidor';
      mensaje.className = 'mensaje error';
    } finally {
      boton.disabled = false;
    }
  });
}
```

---

## Paso 5 — CSS básico

En `styles.css`:

```css
.login-container {
  max-width: 380px;
  margin: 4rem auto;
  padding: 2rem;
  background: white;
  border-radius: var(--radio);
  box-shadow: var(--sombra);
}

#form-login {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.mensaje {
  margin-top: 1rem;
  padding: 0.75rem;
  border-radius: var(--radio);
  text-align: center;
}

.mensaje.error {
  background: #fee2e2;
  color: #991b1b;
}

.mensaje.exito {
  background: #dcfce7;
  color: #166534;
}
```

---

## Paso 6 — Probar

1. Abre `http://localhost:8080/login.html`.
2. Ingresa credenciales **incorrectas** → debe aparecer "Credenciales inválidas".
3. Ingresa credenciales **correctas** → debe aparecer "Bienvenido, [nombre]" y redirigir.
4. DevTools → Network → filtro Fetch/XHR: verifica que `POST /api/login` devuelve `200` o `401`.

---

## Paso 7 — Verificar casos borde

| Caso | Resultado esperado |
|------|---------------------|
| Campos vacíos | `400 Bad Request` |
| Usuario no existe | `401 Unauthorized` (mismo mensaje que contraseña mal) |
| Contraseña incorrecta | `401 Unauthorized` (mismo mensaje) |
| Credenciales OK | `200 OK` con `usuario`, `nombre`, `rol` |
| Método GET | `405 Method Not Allowed` |

Prueba con `curl`:

```bash
# Login fallido
curl -i -X POST -H "Content-Type: application/json" \
  -d '{"usuario":"noexist","contrasena":"x"}' \
  http://localhost:8080/api/login

# Login exitoso
curl -i -X POST -H "Content-Type: application/json" \
  -d '{"usuario":"ana_lopez","contrasena":"secreta123"}' \
  http://localhost:8080/api/login
```

---

## Paso 8 — Seguridad: qué tener en cuenta

Ya mencionado en la teoría; ahora en la práctica:

| Aspecto | Estado en la actividad | Producción |
|---------|-------------------------|------------|
| Contraseña en texto plano | ⚠️ Deuda técnica | Hash + salt (bcrypt/argon2) |
| HTTPS | ❌ | Obligatorio |
| Rate limiting | ❌ | Recomendado |
| Sesiones | ❌ (próxima actividad) | Obligatorio |
| Log de intentos | ✅ Básico | Estructurado + alertas |
| Mensaje genérico en fallos | ✅ | ✅ |

Para documentar la deuda técnica, agrega un comentario en el código:

```cpp
// TODO (deuda técnica): almacenar contraseñas con hash + salt (bcrypt).
// Aceptable en entorno académico; inaceptable en producción.
```

---

## ✅ Checklist de la Actividad 37

- [ ] `login.html` se sirve correctamente.
- [ ] `POST /api/login` valida usuario y contraseña.
- [ ] Se responde `401` con mensaje genérico en fallos.
- [ ] Se responde `200` con `nombre` y `rol` en éxito.
- [ ] La contraseña **nunca** se devuelve en la respuesta.
- [ ] La consola del servidor loguea `[LOGIN OK]` / `[LOGIN FALLIDO]` (sin la contraseña).
- [ ] El frontend muestra mensajes claros al usuario.

## 🧠 Mini-reto
Añade un contador en el servidor: si un mismo usuario falla el login 3 veces seguidas, responde `429 Too Many Requests` durante 5 minutos. Lleva la cuenta en un `map<string, int>`.
