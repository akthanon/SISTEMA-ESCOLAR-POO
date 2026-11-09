# Actividad 40. Panel administrativo

## Temas

- Integración completa
    

## Ejercicio

Crear una página web para administrar usuarios.

Permitir:

- Registrar usuarios.
    
- Modificar usuarios.
    
- Eliminar usuarios.
    
- Buscar usuarios.
    
- Mostrar usuarios.
    
- Cambiar roles.
    

Toda la información deberá almacenarse en archivos TXT y actualizarse dinámicamente desde JavaScript utilizando las rutas API implementadas anteriormente.

---
# TEORÍA

---

## 1. Visión general: la integración completa

Un panel administrativo es la **prueba de fuego** de todo el sistema. Debe funcionar como un cliente más de tu API, consumiendo todas las rutas que ya construiste.

### A. Componentes que se integran
- **Backend C++:** servidor HTTP, enrutamiento, sesiones, roles, BD.
- **API REST:** `/api/usuarios`, `/api/login`, `/api/logout`, `/api/perfil`, rutas específicas por rol.
- **Base de datos:** archivo TXT o SQLite (según hayas avanzado).
- **Frontend HTML/CSS/JS:** panel, formularios, tabla, búsqueda.
- **Autenticación y autorización:** solo `admin` puede acceder al panel.

### B. Flujo completo desde que el admin entra
1. El admin abre `login.html`, introduce credenciales.
2. `POST /api/login` → servidor valida → crea sesión → envía cookie.
3. El frontend recibe el rol en la respuesta → redirige a `admin.html`.
4. El servidor entrega `admin.html` solo si la sesión es válida y el rol es `admin`.
5. El panel carga los usuarios con `GET /api/usuarios`.
6. El admin puede registrar, editar, eliminar, buscar y cambiar roles.
7. Cada acción llama a la API correspondiente con `fetch`.
8. Tras cada acción, el panel refresca la tabla.

---

## 2. Estructura de la carpeta `public/`

```
public/
├── index.html
├── login.html
├── admin.html          ← panel administrativo
├── css/
│   ├── styles.css
│   └── admin.css       ← estilos específicos del panel
├── js/
│   ├── app.js
│   ├── login.js
│   └── admin.js        ← lógica del panel
└── img/
    └── logo.png
```

- **`admin.html`:** solo se sirve a usuarios con rol admin (validado en el servidor).
- **`admin.js`:** maneja todas las operaciones CRUD desde el navegador.
- **`admin.css`:** estilos específicos (sidebar, tabla, formulario de admin).
- **Separación de archivos:** mantiene el proyecto organizado. Un archivo por responsabilidad.

---

## 3. Estructura de `admin.html`

### A. Layout típico de panel administrativo
- **Header:** título del panel, nombre del admin, botón "Cerrar sesión".
- **Sidebar (menú lateral):** secciones del panel (Usuarios, Reportes, Configuración).
- **Contenido principal:** donde vive la tabla de usuarios y los formularios.
- **Área de mensajes:** para notificaciones de éxito/error.

### B. Sección principal: usuarios
- **Barra de búsqueda:** `<input>` para filtrar por nombre, usuario o rol.
- **Botón "Nuevo usuario":** abre el formulario (o lo muestra si estaba oculto).
- **Tabla:** columnas: nombre, usuario, rol, acciones.
- **Formulario de registro/edición:** campos nombre, usuario, contraseña, rol. Botones "Guardar" y "Cancelar".
- **Confirmaciones:** modales o `confirm()` para eliminar.

### C. Semántica HTML
- `<header>`, `<nav>`, `<main>`, `<section>` en lugar de `<div>` genéricos.
- `<table>` con `<thead>` y `<tbody>`.
- `<form>` con `<label>` asociados por `for`/`id`.

---

## 4. Lógica del panel: `admin.js`

### A. Responsabilidades del módulo
- Verificar que la sesión es válida y el rol es admin.
- Cargar la lista de usuarios.
- Renderizar la tabla dinámicamente.
- Manejar el formulario de registro/edición.
- Manejar búsqueda en tiempo real.
- Manejar cambio de roles.
- Manejar eliminación con confirmación.
- Mostrar feedback al usuario.

### B. Estado en el cliente
- **Arreglo de usuarios:** guardado tras el `fetch` inicial. Se usa para búsqueda local.
- **Modo del formulario:** `"crear"` o `"editar"`.
- **Usuario en edición:** referencia al usuario que se está modificando.
- **Filtro de búsqueda:** texto actual del input.

**Fragmento suelto (estado):**
```js
let usuarios = [];
let modoFormulario = "crear";
let usuarioEditando = null;
let filtro = "";
```

- **Ventaja:** Todo el estado vive en un solo lugar. Se puede reconstruir la vista desde él.

---

## 5. Operación: Mostrar usuarios (Read)

- **Al cargar la página:** llamar a `cargarUsuarios()` en `DOMContentLoaded`.
- **`fetch` GET** `/api/usuarios` con `credentials: "include"` si fuera cross-origin (mismo origen, no hace falta, pero es buena práctica tenerlo en cuenta).
- **Verificar `res.ok`:**
  - `200` → parsear y renderizar.
  - `401` → redirigir a login.
  - `403` → mostrar "Acceso denegado".
- **Renderizar la tabla:**
  - Vaciar el `<tbody>`.
  - Recorrer el arreglo `usuarios`.
  - Por cada uno, crear un `<tr>` con `<td>` para cada campo y un `<td>` con botones "Editar" y "Eliminar".
  - Insertar los `<tr>` en el `<tbody>`.
- **Asignar `data-usuario`** a cada fila para saber a quién corresponde.

**Fragmento suelto (idea):**
```js
tr.dataset.usuario = u.usuario;
```

- **Caso arreglo vacío:** mostrar una fila con "No hay usuarios registrados" o un mensaje fuera de la tabla.

---

## 6. Operación: Registrar usuarios (Create)

- **Botón "Nuevo usuario"** → limpia el formulario, `modoFormulario = "crear"`.
- **Submit del formulario:**
  1. `e.preventDefault()`.
  2. Leer los valores del formulario.
  3. Validar en cliente: campos no vacíos, contraseña mínima, usuario sin espacios.
  4. Construir el objeto.
  5. `fetch` POST `/api/usuarios` con JSON.
  6. Verificar `res.ok`.
  7. En éxito: cerrar formulario, `cargarUsuarios()`, mostrar mensaje.
  8. En error: mostrar mensaje del servidor.

- **Casos de error a manejar:**
  - `400` → datos inválidos.
  - `409` → usuario ya existe.
  - `401/403` → sin permiso, redirigir o mensaje.

---

## 7. Operación: Editar usuarios (Update)

- **Botón "Editar"** en cada fila:
  1. Leer `data-usuario` de la fila.
  2. Buscar el usuario en el arreglo local.
  3. Rellenar el formulario con sus datos.
  4. `modoFormulario = "editar"`.
  5. `usuarioEditando = u.usuario`.
  6. Cambiar el texto del botón de "Registrar" a "Guardar cambios".

- **Submit del formulario:**
  - Si `modoFormulario === "editar"` → `fetch` PUT `/api/usuarios/{usuario}`.
  - Si `modoFormulario === "crear"` → `fetch` POST `/api/usuarios`.

- **Consideración sobre contraseña:**
  - En edición, si el campo contraseña está vacío, no se actualiza.
  - Si tiene valor, se actualiza.
  - El servidor debe respetar esta regla.

- **Cancelar edición:** botón que limpia el formulario y vuelve a modo "crear".

---

## 8. Operación: Eliminar usuarios (Delete)

- **Botón "Eliminar"** en cada fila:
  1. Leer `data-usuario`.
  2. Buscar el usuario en el arreglo local.
  3. Mostrar confirmación: `confirm("¿Eliminar a " + u.nombre + "?")`.
  4. Si el usuario acepta → `fetch` DELETE `/api/usuarios/{usuario}`.
  5. Si el usuario cancela → no hacer nada.

- **Reglas importantes:**
  - **No permitir eliminarse a sí mismo.** El servidor debe verificarlo. El cliente solo muestra el botón deshabilitado como UX.
  - **Confirmación siempre.** Nunca borrar sin preguntar.
  - **Feedback tras éxito:** mensaje y refresco de la tabla.

- **Códigos de error:**
  - `404` → usuario ya no existe (quizá otro admin lo eliminó).
  - `403` → sin permiso.

---

## 9. Operación: Buscar usuarios

### A. Búsqueda en el cliente (recomendada para esta actividad)
- **Ventaja:** Instantánea, sin peticiones al servidor.
- **Implementación:** Filtrar el arreglo `usuarios` con `filter` según el texto del input.
- **Búsqueda por:** nombre, usuario, rol.
- **Case-insensitive:** Convertir ambos a minúsculas con `toLowerCase()`.
- **Evento `input`:** Se dispara con cada tecla. Re-renderiza la tabla con el arreglo filtrado.

**Fragmento suelto (idea):**
```js
const filtrados = usuarios.filter(u =>
    u.nombre.toLowerCase().includes(filtro.toLowerCase()) ||
    u.usuario.toLowerCase().includes(filtro.toLowerCase())
);
```

- **Ventaja:** La tabla se actualiza sin peticiones al servidor.
- **Desventaja:** Si hay miles de usuarios, no es escalable. Para esta actividad, es suficiente.

### B. Búsqueda en el servidor (alternativa escalable)
- **Ruta:** `GET /api/usuarios?q=<texto>`.
- **El servidor** filtra en SQL con `LIKE`.
- **El cliente** solo envía la query y renderiza el resultado.
- **Ventaja:** Escalable. La BD hace el trabajo pesado.
- **Desventaja:** Una petición por cada tecla (o usar debounce).

**Debounce:** Esperar X ms antes de enviar la petición, para no saturar el servidor con cada tecla.

---

## 10. Operación: Cambiar roles

- **Dos enfoques:**

### A. Cambio inline en la tabla
- En la fila, en lugar de mostrar el rol como texto, mostrar un `<select>` con las opciones (alumno, maestro, admin).
- Al cambiar la selección → `fetch` PUT `/api/usuarios/{usuario}/rol` con el nuevo rol.
- **Ventaja:** Rápido, intuitivo.
- **Desventaja:** Fácil de cambiar por error. Requiere confirmación.

### B. A través del formulario de edición
- Editar el usuario y cambiar el rol en el formulario.
- Se guarda como parte del PUT general.
- **Ventaja:** Un solo flujo de edición.
- **Desventaja:** Más pasos para cambiar solo el rol.

- **Recomendación para esta actividad:** Enfoque A con confirmación.

- **Reglas:**
  - Solo un admin puede cambiar roles.
  - No puedes cambiarte tu propio rol (para no quedarte sin admin por error).
  - El cambio se refleja en la próxima sesión del usuario afectado (si congelaste el rol en sesión) o inmediatamente (si consultas la BD por petición).

---

## 11. Feedback y mensajes al usuario

- **Notificaciones de éxito:** "Usuario registrado", "Cambios guardados", "Usuario eliminado".
- **Notificaciones de error:** "No se pudo guardar", "Usuario ya existe", "Sin conexión".
- **Ubicación:** Un `<div>` de alertas en la parte superior del panel, o `toast` que desaparece tras unos segundos.
- **Tipos visuales:**
  - Verde para éxito.
  - Rojo para error.
  - Amarillo para advertencia.
- **Auto-ocultar** tras 3-5 segundos.
- **Evitar `alert()`** salvo para confirmaciones críticas. Mejor toasts o banners.

**Fragmento suelto (idea):**
```js
function mostrarMensaje(texto, tipo) {
    // actualizar un div con class "alerta alerta-exito" o "alerta alerta-error"
    // programar su desaparición
}
```

---

## 12. Confirmaciones antes de acciones destructivas

- **Eliminar:** `confirm()` o un modal propio.
- **Cambiar rol a admin:** Confirmación especialmente importante. Escalar privilegios es sensible.
- **Cambiar rol desde admin:** Confirmar con aviso de que el usuario perderá privilegios.
- **Regla:** Cualquier acción irreversible debe confirmarse.

---

## 13. Autorización en el panel

### A. Verificación en el servidor (imprescindible)
- **Al servir `admin.html`:** el servidor verifica sesión y rol. Si no es admin → redirige a `login.html` o devuelve `403`.
- **En cada API call:** el servidor verifica sesión y rol antes de procesar.
- **Nunca confíes en el cliente** para decidir si el usuario es admin.

### B. Verificación en el cliente (UX)
- **Al cargar `admin.js`:**
  1. `fetch` a `/api/perfil`.
  2. Si `401` → `window.location.href = "/login.html"`.
  3. Si `403` o rol ≠ admin → `window.location.href = "/index.html"`.
  4. Si es admin → continuar cargando el panel.
- **Ventaja:** Redirige rápido si no corresponde. **No es seguridad, es UX.**
- **La seguridad real está en el servidor.** Aunque alguien abra `admin.html` directamente, todas las API calls fallarán con 403.

---

## 14. Sincronización entre pestañas (opcional)

- **Escenario:** Un admin elimina un usuario en una pestaña, y otra pestaña muestra el estado antiguo.
- **Solución 1:** Botón "Actualizar" manual. Simple.
- **Solución 2:** Polling cada X segundos. Recarga la lista automáticamente.
- **Solución 3:** `BroadcastChannel` API. Sincroniza pestañas del mismo origen.
- **Para esta actividad:** Botón "Actualizar" + recarga tras cada acción es suficiente.

---

## 15. Persistencia: TXT vs. SQL

Aunque la actividad menciona "archivos TXT", es probable que ya hayas migrado a SQL en actividades anteriores. Mencionamos ambas opciones:

### A. Si usas TXT
- Todas las operaciones CRUD reescriben `usuarios.txt` completo.
- **Ventaja:** Simple.
- **Desventaja:** No escala, no hay concurrencia segura, búsquedas lentas.

### B. Si usas SQL (recomendado)
- Las operaciones CRUD usan consultas preparadas.
- **Ventaja:** Escalable, concurrente, rápido.
- **Desventaja:** Requiere integrar SQLite.
- **La interfaz del panel no cambia** entre TXT y SQL. El frontend no sabe qué usa el backend.
- **Ventaja del encapsulamiento (Actividad 34):** `BaseDatosSQL` oculta si es TXT o SQL. El servidor no cambia.

### C. Decisión para esta actividad
- **Si ya tienes SQL:** úsalo. La actividad se centra en el panel, no en cambiar la persistencia.
- **Si sigues con TXT:** está bien para un proyecto académico. Documenta la deuda técnica.

---

## 16. Rutas API necesarias (repaso)

Para que el panel funcione, estas rutas deben existir:

| Ruta | Método | Propósito | Rol |
|------|--------|-----------|-----|
| `/api/login` | POST | Autenticar | público |
| `/api/logout` | POST | Cerrar sesión | cualquier sesión |
| `/api/perfil` | GET | Datos del usuario actual | cualquier sesión |
| `/api/usuarios` | GET | Listar usuarios | admin |
| `/api/usuarios` | POST | Crear usuario | admin |
| `/api/usuarios/{usuario}` | PUT | Actualizar usuario | admin |
| `/api/usuarios/{usuario}` | DELETE | Eliminar usuario | admin |
| `/api/usuarios/{usuario}/rol` | PUT | Cambiar rol | admin |

- **Verifica que todas estén implementadas** antes de construir el panel.
- **Prueba cada una con `curl`** antes de consumirlas desde el frontend.
- **Documenta cada ruta** en el README con parámetros y respuestas.

---

## 17. Estructura de `admin.js`

Un módulo bien organizado suele tener estas funciones:

| Función | Responsabilidad |
|---------|-----------------|
| `verificarSesion()` | Comprueba que hay sesión y es admin. Redirige si no. |
| `cargarUsuarios()` | `fetch` GET → actualiza el arreglo → renderiza. |
| `renderizarTabla()` | Reconstruye el `<tbody>` con el arreglo filtrado. |
| `filtrarUsuarios()` | Aplica el filtro de búsqueda al arreglo. |
| `abrirFormularioNuevo()` | Limpia el form y lo pone en modo "crear". |
| `abrirFormularioEdicion(usuario)` | Rellena el form con datos y lo pone en modo "editar". |
| `cerrarFormulario()` | Limpia y oculta. |
| `guardarUsuario(e)` | Submit: decide entre POST y PUT. |
| `eliminarUsuario(usuario)` | Confirmación + DELETE. |
| `cambiarRol(usuario, nuevoRol)` | PUT al endpoint de rol. |
| `cerrarSesion()` | POST logout + redirección. |
| `mostrarMensaje(texto, tipo)` | Feedback visual. |
| `manejarError(res)` | Traduce códigos HTTP a mensajes. |

- **Ventaja:** Cada función hace una cosa. Fácil de leer y mantener.
- **Regla:** Ninguna función debería exceder 30-40 líneas.

---

## 18. Ciclo de vida del panel

1. **Carga inicial:**
   - Verificar sesión y rol.
   - Cargar lista de usuarios.
   - Renderizar tabla.
   - Asignar listeners (form submit, búsqueda, botones).
2. **Interacción del usuario:**
   - Cada acción llama a una función específica.
   - Cada función hace su `fetch`, maneja errores, actualiza estado y re-renderiza.
3. **Refresco:**
   - Tras cada operación, recargar la lista (`cargarUsuarios()`).
   - No manipular el DOM manualmente tras cada operación; reconstruir desde el modelo.
4. **Cierre:**
   - Logout limpia la sesión del servidor y la cookie.
   - Redirección a `login.html`.

---

## 19. Manejo de errores en el panel

- **`401 Unauthorized`:** La sesión expiró o es inválida. Redirigir a login.
- **`403 Forbidden`:** El usuario ya no es admin (o nunca lo fue). Redirigir a `index.html` o mostrar "Acceso denegado".
- **`404 Not Found`:** El recurso no existe. Puede ser que otro admin lo eliminó. Recargar la lista.
- **`409 Conflict`:** Usuario duplicado. Mostrar mensaje específico.
- **`400 Bad Request`:** Datos inválidos. Mostrar mensaje del servidor.
- **`500 Internal Server Error`:** Error del servidor. Mensaje genérico.
- **Error de red:** `fetch` rechaza la promesa. Mostrar "Sin conexión con el servidor".

### A. Patrón de manejo centralizado
- Una función `manejarRespuesta(res)` que:
  - Verifica `res.ok`.
  - Si `401` → redirige a login.
  - Si `403` → redirige a index.
  - Si otro error → lanza excepción con mensaje.
  - Si OK → devuelve `res.json()`.
- **Ventaja:** No repites la lógica en cada función.

---

## 20. UX/UI: hacer el panel usable

### A. Sidebar
- Fija, con las secciones principales.
- Íconos opcionales.
- Enlace activo resaltado.

### B. Tabla
- Filas alternas con color suave.
- Hover en las filas.
- Encabezados fijos (sticky) si la lista es larga.
- Ancho de columnas consistente.
- Botones de acción alineados.

### C. Formulario
- Modal o panel lateral.
- Campos con labels claros.
- Validación visual (borde rojo en errores).
- Botón "Guardar" y "Cancelar".
- Mensajes de error bajo el campo correspondiente.

### D. Búsqueda
- Input grande, con placeholder "Buscar por nombre, usuario o rol".
- Debounce en el evento `input` si filtras en servidor.
- Botón de limpiar (X) dentro del input.

### E. Feedback
- Toasts en la esquina superior derecha.
- Spinner o texto "Cargando..." durante peticiones.
- Botones deshabilitados mientras hay una operación en curso.

### F. Responsive
- En móvil, la sidebar puede colapsar o pasar a un menú hamburguesa.
- La tabla puede volverse scrollable horizontalmente.
- Los formularios ocupan el ancho completo.

---

## 21. Pruebas del panel

### A. Pruebas funcionales
1. **Login como admin.** Verificar que redirige a `admin.html`.
2. **Login como alumno.** Verificar que **no** accede a `admin.html`.
3. **Cargar lista.** Los usuarios aparecen correctamente.
4. **Registrar usuario.** Aparece en la tabla tras el POST.
5. **Registrar usuario duplicado.** Mensaje de error.
6. **Editar usuario.** Los cambios se reflejan.
7. **Eliminar usuario.** Confirmación y desaparición de la tabla.
8. **Buscar.** La tabla se filtra en tiempo real.
9. **Cambiar rol.** El cambio se refleja en la tabla y en la BD.
10. **Cerrar sesión.** Redirige a login y la cookie se borra.

### B. Pruebas de seguridad
1. **Abrir `admin.html` sin sesión.** El servidor redirige o devuelve 403.
2. **Cookie manipulada.** El servidor responde 401.
3. **Sesión de alumno en admin.html.** El servidor responde 403 en las API.
4. **Intento de eliminarse a sí mismo.** El servidor rechaza.
5. **Intento de cambiar el propio rol.** El servidor rechaza.
6. **DevTools → Network.** Todos los códigos HTTP correctos.

### C. Pruebas de persistencia
1. Registrar usuario → reiniciar servidor → usuario sigue ahí.
2. Editar usuario → reiniciar → cambio persiste.
3. Eliminar usuario → reiniciar → no reaparece.

---

## 22. Buenas prácticas

- **Separación de responsabilidades en el frontend:** un archivo JS por página, no mezclar lógica de login con admin.
- **Estado centralizado:** un arreglo `usuarios` como fuente de verdad; reconstruir la tabla desde él.
- **No manipular el DOM manualmente tras cada acción:** recargar desde el modelo.
- **Uso de `textContent`, no `innerHTML`,** para datos del servidor. Evita XSS.
- **Delegación de eventos** en el `<tbody>` para los botones de cada fila.
- **Verificar la sesión al cargar** el panel, no solo al hacer login.
- **Autorización en el servidor siempre.** El frontend solo mejora la UX.
- **Feedback visual** para toda acción: éxito, error, en curso.
- **Confirmaciones antes de acciones destructivas** (eliminar, cambiar rol a admin).
- **Botones deshabilitados** durante peticiones en curso.
- **Recargar la tabla** tras cada operación exitosa.
- **Manejo centralizado de errores HTTP** (401 → login, 403 → index, etc.).
- **Rutas relativas** para las peticiones a la API.
- **Sin `console.log`** en producción (o al menos mínimo). Los logs deben ir al servidor.
- **CSS consistente** con el resto del proyecto. Reutiliza variables (`:root`).
- **Responsive:** el panel debe funcionar en móvil y escritorio.
- **Accesibilidad:** labels asociados, `alt` en imágenes, contraste adecuado.
- **Documentar la matriz de permisos** en el README.
- **Prueba cada operación con cada rol** antes de dar por terminado el panel.
- **Considera añadir paginación** si esperas muchos usuarios. Para esta actividad, no es necesaria.

---

## 23. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`admin.html` accesible sin sesión** | El servidor debe verificar sesión y rol antes de servir el archivo. |
| **Las API no verifican el rol** | Cada handler debe verificar el rol. Denegar por defecto. |
| **El cliente decide si es admin** | El servidor siempre decide. El cliente solo adapta la UI. |
| **La tabla se duplica al refrescar** | Vaciar el `<tbody>` antes de re-renderizar. |
| **Los datos del usuario se muestran con `[object Object]`** | Usar `u.nombre`, no `u`. |
| **XSS al inyectar nombres de usuario con `innerHTML`** | Usar `textContent`. |
| **La búsqueda no filtra bien** | Convertir a minúsculas ambos lados. Revisar que el filtro incluya todos los campos deseados. |
| **Eliminar sin confirmar** | `confirm()` o modal antes del DELETE. |
| **El admin se elimina a sí mismo** | El servidor debe rechazarlo. El cliente debe deshabilitar el botón. |
| **El admin se cambia su propio rol y pierde acceso** | El servidor debe rechazar cambios de rol propios. |
| **El form no se limpia tras guardar** | Llamar a `form.reset()` y volver a modo "crear". |
| **Errores silenciosos** | Mostrar siempre un mensaje al usuario. Loguear en consola para depurar. |
| **No se refresca la tabla tras una operación** | Llamar a `cargarUsuarios()` tras cada éxito. |
| **Sesión expirada pero el panel sigue "funcionando"** | Manejar `401` en cada fetch, redirigir a login. |
| **CORS al probar desde otro puerto** | Servir el frontend desde el mismo backend. |
| **Contraseñas en texto plano en la BD** | Hash + salt en producción. Documenta deuda técnica si es académico. |
| **El panel no responde en móvil** | CSS responsive: media queries, unidades relativas. |
| **Los botones no hacen nada** | Verificar que los listeners estén asignados tras renderizar. Usar delegación de eventos. |
| **Al recargar tras editar, el form sigue en modo edición** | Resetear `modoFormulario` a `"crear"` tras guardar. |
| **Muchos `console.log` olvidados** | Limpiar antes de la entrega. |
| **Falta confirmación en cambio de rol** | Añadir `confirm()` para acciones sensibles. |

---

## 24. Verificación final

Cuando todo funcione:

1. **Login como admin** → accede al panel.
2. **Login como alumno** → no puede acceder al panel.
3. **Sin sesión** → no puede acceder a `admin.html` ni a las API.
4. **La tabla muestra los usuarios** al cargar.
5. **Registrar un usuario** → aparece en la tabla.
6. **Buscar un usuario** → la tabla se filtra.
7. **Editar un usuario** → los cambios se reflejan.
8. **Cambiar el rol de un usuario** → se actualiza.
9. **Eliminar un usuario** → desaparece de la tabla y de la BD.
10. **Cerrar sesión** → redirige a login.
11. **Reiniciar el servidor** → los datos persisten.
12. **Abrir DevTools → Network** → todos los códigos son correctos.
13. **Probar en móvil** → el panel es usable.
14. **Revisar el README** → la documentación está actualizada.

Si todo esto funciona, el panel administrativo está completo y la integración del sistema también.

---

## 25. Mejoras futuras (documentar como "siguiente iteración")

- **Paginación** de la tabla si hay muchos usuarios.
- **Ordenar** por cualquier columna.
- **Filtros avanzados:** por rol, por fecha de registro, etc.
- **Auditoría visual:** mostrar quién creó/modificó/eliminó cada usuario.
- **Cambio de contraseña obligatorio** en el primer login.
- **Reset de contraseña** por parte del admin.
- **Exportar usuarios** a CSV o JSON.
- **Importar usuarios** desde CSV.
- **Multi-rol** (un usuario con varios roles).
- **Permisos granulares** en lugar de roles fijos (RBAC con permisos).
- **Doble factor de autenticación (2FA).**
- **HTTPS** en producción.
- **Rate limiting** en login.
- **Sesiones persistidas** en BD para sobrevivir reinicios.

Documentar estas mejoras muestra madurez en el análisis y visión de futuro.

---

## 26. Resumen de conceptos clave

- **Panel administrativo:** interfaz de gestión completa, cliente de la API.
- **Integración:** unir backend, API, sesiones, roles y frontend en una sola aplicación funcional.
- **Estructura:** `admin.html`, `admin.js`, `admin.css` separados.
- **Estado en el cliente:** arreglo `usuarios`, modo de formulario, filtro. Reconstruir la vista desde el modelo.
- **Operaciones CRUD:** mostrar, registrar, editar, eliminar, buscar, cambiar roles.
- **Fetch:** cada operación llama a la API correspondiente con el método correcto.
- **Feedback:** mensajes visuales de éxito, error y en curso.
- **Confirmaciones:** antes de acciones destructivas o de escalada de privilegios.
- **Autorización en el servidor:** siempre. El cliente solo adapta la UI.
- **Manejo centralizado de errores:** 401 → login, 403 → index, otros → mensaje.
- **Refresco tras operación:** recargar la lista desde el servidor.
- **Seguridad:** `textContent`, sin `innerHTML` con datos del usuario, sin `console.log` en producción.
- **Persistencia:** TXT o SQL. El frontend no distingue.
- **Pruebas:** funcionales, de seguridad, de persistencia.
- **Mejoras futuras:** paginación, auditoría, 2FA, HTTPS, multi-rol.

---
# PASO A PASO

---
# Actividad 40 — Panel administrativo

## Objetivo
Crear `admin.html` que permita registrar, modificar, eliminar, buscar y listar usuarios, además de cambiar roles. Consume todas las rutas API construidas.

## Requisito previo
Todas estas rutas deben existir y funcionar:

| Ruta | Método | Rol |
|------|--------|-----|
| `/api/usuarios` | GET | admin |
| `/api/usuarios` | POST | admin |
| `/api/usuarios/{u}` | PUT | admin |
| `/api/usuarios/{u}` | DELETE | admin |
| `/api/usuarios/{u}/rol` | PUT | admin |
| `/api/perfil` | GET | cualquiera |
| `/api/logout` | POST | cualquiera |

---

## Paso 1 — Estructura de archivos

```
public/
├── admin.html
├── css/
│   └── admin.css
├── js/
│   └── admin.js
```

---

## Paso 2 — `admin.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Panel de administración</title>
  <link rel="stylesheet" href="/styles.css">
  <link rel="stylesheet" href="/css/admin.css">
</head>
<body>
  <header class="admin-header">
    <img src="/img/logo.png" alt="Logo" width="40">
    <h1>Panel administrativo</h1>
    <div class="admin-sesion">
      <span id="admin-nombre">—</span>
      <button id="btn-logout">Cerrar sesión</button>
    </div>
  </header>

  <div class="admin-layout">
    <aside class="admin-sidebar">
      <h2>Secciones</h2>
      <ul>
        <li><a href="#usuarios" class="activo">Usuarios</a></li>
        <li><a href="#archivos">Archivos</a></li>
        <li><a href="#reportes">Reportes</a></li>
      </ul>
    </aside>

    <main class="admin-main">
      <section id="usuarios">
        <div class="admin-toolbar">
          <input type="text" id="busqueda" placeholder="Buscar por nombre, usuario o rol...">
          <button id="btn-nuevo">Nuevo usuario</button>
        </div>

        <div id="alerta" class="alerta" hidden></div>

        <table id="tabla-usuarios">
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Usuario</th>
              <th>Rol</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>

        <form id="form-usuario" class="admin-form" hidden>
          <h3 id="form-titulo">Nuevo usuario</h3>

          <label for="form-nombre">Nombre:</label>
          <input type="text" id="form-nombre" required>

          <label for="form-usuario">Usuario:</label>
          <input type="text" id="form-usuario" required>

          <label for="form-contrasena">Contraseña:</label>
          <input type="password" id="form-contrasena"
                 placeholder="Dejar en blanco para no cambiar">

          <label for="form-rol">Rol:</label>
          <select id="form-rol" required>
            <option value="alumno">Alumno</option>
            <option value="maestro">Maestro</option>
            <option value="admin">Administrador</option>
          </select>

          <div class="admin-form-botones">
            <button type="submit">Guardar</button>
            <button type="button" id="btn-cancelar">Cancelar</button>
          </div>
        </form>
      </section>
    </main>
  </div>

  <script src="/js/admin.js"></script>
</body>
</html>
```

---

## Paso 3 — `admin.js`: estado y arranque

```js
'use strict';

const tbody       = document.querySelector('#tabla-usuarios tbody');
const formUsuario = document.querySelector('#form-usuario');
const alerta      = document.querySelector('#alerta');
const busqueda    = document.querySelector('#busqueda');

let usuarios        = [];
let modoFormulario  = 'crear';    // 'crear' | 'editar'
let usuarioEditando = null;
let filtro          = '';

document.addEventListener('DOMContentLoaded', async () => {
  const ok = await verificarSesion();
  if (!ok) return;

  await cargarUsuarios();
  conectarEventos();
});
```

---

## Paso 4 — Verificación de sesión y rol

```js
async function verificarSesion() {
  try {
    const res = await fetch('/api/perfil');
    if (res.status === 401) {
      window.location.href = '/login.html';
      return false;
    }
    const data = await res.json();
    if (data.rol !== 'admin') {
      window.location.href = '/index.html';
      return false;
    }
    document.querySelector('#admin-nombre').textContent = data.usuario;
    return true;
  } catch {
    window.location.href = '/login.html';
    return false;
  }
}
```

🎯 Si el rol no es admin, redirige. **No es seguridad**, es UX. La seguridad está en el servidor (actividad 39).

---

## Paso 5 — Cargar usuarios

```js
async function cargarUsuarios() {
  try {
    const res = await fetch('/api/usuarios');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    usuarios = await res.json();
    renderizarTabla();
  } catch (err) {
    mostrarAlerta(`No se pudieron cargar usuarios: ${err.message}`, 'error');
  }
}
```

---

## Paso 6 — Renderizar tabla

```js
function renderizarTabla() {
  tbody.innerHTML = '';

  const filtrados = usuarios.filter(u =>
    u.nombre.toLowerCase().includes(filtro.toLowerCase()) ||
    u.usuario.toLowerCase().includes(filtro.toLowerCase()) ||
    u.rol.toLowerCase().includes(filtro.toLowerCase())
  );

  if (filtrados.length === 0) {
    const tr = document.createElement('tr');
    const td = document.createElement('td');
    td.colSpan = 4;
    td.className = 'vacio';
    td.textContent = 'Sin usuarios';
    tr.appendChild(td);
    tbody.appendChild(tr);
    return;
  }

  filtrados.forEach(u => {
    const tr = document.createElement('tr');
    tr.dataset.usuario = u.usuario;

    const tdNombre = document.createElement('td');
    tdNombre.textContent = u.nombre;

    const tdUsuario = document.createElement('td');
    tdUsuario.textContent = u.usuario;

    const tdRol = document.createElement('td');
    const selectRol = document.createElement('select');
    ['alumno', 'maestro', 'admin'].forEach(r => {
      const opt = document.createElement('option');
      opt.value = r;
      opt.textContent = r;
      if (r === u.rol) opt.selected = true;
      selectRol.appendChild(opt);
    });
    selectRol.addEventListener('change', () =>
      cambiarRol(u.usuario, selectRol.value));
    tdRol.appendChild(selectRol);

    const tdAcc = document.createElement('td');
    tdAcc.className = 'acciones';

    const btnEditar = document.createElement('button');
    btnEditar.textContent = 'Editar';
    btnEditar.dataset.accion = 'editar';

    const btnEliminar = document.createElement('button');
    btnEliminar.textContent = 'Eliminar';
    btnEliminar.className = 'btn-peligro';
    btnEliminar.dataset.accion = 'eliminar';

    tdAcc.appendChild(btnEditar);
    tdAcc.appendChild(btnEliminar);

    tr.appendChild(tdNombre);
    tr.appendChild(tdUsuario);
    tr.appendChild(tdRol);
    tr.appendChild(tdAcc);
    tbody.appendChild(tr);
  });
}
```

⚠️ **`textContent`, no `innerHTML`.** Evita XSS si un nombre contiene `<script>`.

---

## Paso 7 — Delegación de eventos en la tabla

```js
function conectarEventos() {
  document.querySelector('#btn-logout').addEventListener('click', logout);
  document.querySelector('#btn-nuevo').addEventListener('click', abrirFormularioNuevo);
  document.querySelector('#btn-cancelar').addEventListener('click', cerrarFormulario);
  formUsuario.addEventListener('submit', guardarUsuario);
  busqueda.addEventListener('input', (e) => {
    filtro = e.target.value;
    renderizarTabla();
  });

  tbody.addEventListener('click', (e) => {
    const boton = e.target.closest('button[data-accion]');
    if (!boton) return;
    const fila = boton.closest('tr');
    const usuario = fila.dataset.usuario;
    if (boton.dataset.accion === 'editar')   abrirFormularioEdicion(usuario);
    if (boton.dataset.accion === 'eliminar') eliminarUsuario(usuario);
  });
}
```

---

## Paso 8 — Formulario: crear y editar

```js
function abrirFormularioNuevo() {
  modoFormulario = 'crear';
  usuarioEditando = null;
  formUsuario.reset();
  document.querySelector('#form-titulo').textContent = 'Nuevo usuario';
  document.querySelector('#form-usuario').readOnly = false;
  document.querySelector('#form-contrasena').required = true;
  formUsuario.hidden = false;
}

function abrirFormularioEdicion(usuario) {
  const u = usuarios.find(x => x.usuario === usuario);
  if (!u) return;

  modoFormulario = 'editar';
  usuarioEditando = usuario;

  document.querySelector('#form-nombre').value     = u.nombre;
  document.querySelector('#form-usuario').value    = u.usuario;
  document.querySelector('#form-contrasena').value = '';
  document.querySelector('#form-rol').value        = u.rol;

  document.querySelector('#form-titulo').textContent = 'Editar usuario';
  document.querySelector('#form-usuario').readOnly = true;
  document.querySelector('#form-contrasena').required = false;

  formUsuario.hidden = false;
}

function cerrarFormulario() {
  modoFormulario = 'crear';
  usuarioEditando = null;
  formUsuario.reset();
  formUsuario.hidden = true;
  document.querySelector('#form-usuario').readOnly = false;
  document.querySelector('#form-contrasena').required = true;
}
```

---

## Paso 9 — Guardar usuario (POST o PUT)

```js
async function guardarUsuario(e) {
  e.preventDefault();

  const datos = {
    nombre:     document.querySelector('#form-nombre').value.trim(),
    usuario:    document.querySelector('#form-usuario').value.trim(),
    contrasena: document.querySelector('#form-contrasena').value,
    rol:        document.querySelector('#form-rol').value
  };

  if (!datos.nombre || !datos.usuario || !datos.rol) {
    mostrarAlerta('Completa los campos obligatorios.', 'error');
    return;
  }

  if (modoFormulario === 'crear' && !datos.contrasena) {
    mostrarAlerta('La contraseña es obligatoria al crear.', 'error');
    return;
  }

  const url    = modoFormulario === 'editar'
    ? `/api/usuarios/${encodeURIComponent(usuarioEditando)}`
    : '/api/usuarios';
  const method = modoFormulario === 'editar' ? 'PUT' : 'POST';

  try {
    const res = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(datos)
    });

    const respuesta = await res.json().catch(() => ({}));

    if (!res.ok) {
      mostrarAlerta(respuesta.error || `Error ${res.status}`, 'error');
      return;
    }

    mostrarAlerta(
      modoFormulario === 'editar' ? 'Cambios guardados' : 'Usuario creado',
      'exito');
    cerrarFormulario();
    await cargarUsuarios();
  } catch (err) {
    mostrarAlerta(`Sin conexión: ${err.message}`, 'error');
  }
}
```

---

## Paso 10 — Eliminar con confirmación

```js
async function eliminarUsuario(usuario) {
  const u = usuarios.find(x => x.usuario === usuario);
  if (!u) return;

  if (!confirm(`¿Eliminar a ${u.nombre} (${u.usuario})?`)) return;

  try {
    const res = await fetch(`/api/usuarios/${encodeURIComponent(usuario)}`,
      { method: 'DELETE' });
    if (!res.ok) {
      mostrarAlerta(`Error ${res.status}`, 'error');
      return;
    }
    mostrarAlerta('Usuario eliminado', 'exito');
    await cargarUsuarios();
  } catch (err) {
    mostrarAlerta(`Sin conexión: ${err.message}`, 'error');
  }
}
```

---

## Paso 11 — Cambiar rol

```js
async function cambiarRol(usuario, nuevoRol) {
  if (!confirm(`¿Cambiar el rol de ${usuario} a "${nuevoRol}"?`)) {
    await cargarUsuarios();   // revertir el select
    return;
  }

  try {
    const res = await fetch(
      `/api/usuarios/${encodeURIComponent(usuario)}/rol`,
      {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ rol: nuevoRol })
      });

    if (!res.ok) {
      mostrarAlerta(`Error ${res.status}`, 'error');
      await cargarUsuarios();
      return;
    }
    mostrarAlerta('Rol actualizado', 'exito');
    await cargarUsuarios();
  } catch (err) {
    mostrarAlerta(`Sin conexión: ${err.message}`, 'error');
    await cargarUsuarios();
  }
}
```

---

## Paso 12 — Feedback visual

```js
let timeoutAlerta = null;

function mostrarAlerta(texto, tipo = 'info') {
  alerta.textContent = texto;
  alerta.className = `alerta alerta-${tipo}`;
  alerta.hidden = false;

  if (timeoutAlerta) clearTimeout(timeoutAlerta);
  timeoutAlerta = setTimeout(() => { alerta.hidden = true; }, 4000);
}
```

CSS en `css/admin.css`:

```css
.alerta {
  padding: 0.75rem 1rem;
  margin: 1rem 0;
  border-radius: var(--radio);
  font-weight: 600;
}
.alerta-exito { background: #dcfce7; color: #166534; }
.alerta-error { background: #fee2e2; color: #991b1b; }
.alerta-info  { background: #dbeafe; color: #1e40af; }

.admin-layout {
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 1.5rem;
  padding: 1.5rem;
}
.admin-sidebar { background: var(--color-sidebar); color: white; padding: 1rem; border-radius: var(--radio); }
.admin-sidebar a { color: #e5e7eb; text-decoration: none; display: block; padding: 0.5rem; }
.admin-sidebar a.activo { background: rgba(255,255,255,0.15); border-radius: var(--radio); }
.admin-toolbar { display: flex; gap: 0.75rem; margin-bottom: 1rem; }
.admin-toolbar input { flex: 1; padding: 0.5rem; border-radius: var(--radio); border: 1px solid var(--color-borde); }
.admin-form { display: flex; flex-direction: column; gap: 0.75rem; max-width: 420px; margin-top: 1.5rem; }
.admin-form-botones { display: flex; gap: 0.5rem; margin-top: 0.5rem; }
.btn-peligro { background: var(--color-peligro); color: white; }
```

---

## Paso 13 — Ruta `/api/usuarios/{u}/rol` en el servidor

Añádela en `manejarAPI`:

```cpp
if (ruta.rfind("/api/usuarios/", 0) == 0 &&
    ruta.size() > 17 && ruta.substr(ruta.size() - 4) == "/rol" &&
    metodo == "PUT") {

    // Extraer el usuario de /api/usuarios/{u}/rol
    string resto = ruta.substr(14);              // quitar "/api/usuarios/"
    size_t posRol = resto.find("/rol");
    string usuario = resto.substr(0, posRol);

    string nuevoRol = extraerValorJSON(cuerpo, "rol");
    if (nuevoRol != "alumno" && nuevoRol != "maestro" && nuevoRol != "admin") {
        return construirRespuesta(400, "application/json; charset=utf-8",
            "{\"error\":\"Rol inválido\"}");
    }

    Usuario u = baseDatos.buscarUsuario(usuario);
    if (u.getUsuario().empty()) {
        return construirRespuesta(404, "application/json; charset=utf-8",
            "{\"error\":\"Usuario no encontrado\"}");
    }

    u.setRol(nuevoRol);
    if (!baseDatos.actualizarUsuario(u)) {
        return construirRespuesta(500, "application/json; charset=utf-8",
            "{\"error\":\"Error al actualizar\"}");
    }
    return construirRespuesta(200, "application/json; charset=utf-8",
        "{\"ok\":true}");
}
```

---

## ✅ Checklist de la Actividad 40

- [ ] `admin.html` carga y verifica sesión al entrar.
- [ ] Un alumno que abra `admin.html` es redirigido.
- [ ] La tabla muestra los usuarios al cargar.
- [ ] Se puede registrar, editar y eliminar usuarios.
- [ ] Se puede cambiar el rol con un select en la tabla.
- [ ] La búsqueda filtra en tiempo real.
- [ ] Los datos persisten tras reiniciar el servidor.
- [ ] Se usa `textContent`, no `innerHTML`, para datos del servidor.

## 🧠 Mini-reto
Añade paginación: muestra 10 usuarios por página con botones "Anterior" / "Siguiente". Hazlo en cliente sobre el arreglo `usuarios`.
