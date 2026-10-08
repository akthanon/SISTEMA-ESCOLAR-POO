# Actividad 31. CRUD completo

## Temas

- CRUD
    
- Persistencia
    

## Ejercicio

Agregar las operaciones completas para administrar alumnos.

Implementar las rutas:

```text
GET    /api/alumnos
POST   /api/alumnos
PUT    /api/alumnos
DELETE /api/alumnos
```

Guardar todos los cambios en `usuarios.txt`.

La página web deberá permitir:

- Registrar alumnos.
    
- Editar alumnos.
    
- Eliminar alumnos.
    
- Mostrar alumnos.
    

---
# TEORÍA

---

## 1. ¿Qué es CRUD?

- **Acrónimo:** Create, Read, Update, Delete.
- **Correspondencia HTTP:**

| Operación | Verbo HTTP | Semántica | Idempotente |
|-----------|------------|-----------|-------------|
| Create | POST | Crear un nuevo recurso | No |
| Read | GET | Leer uno o varios recursos | Sí |
| Update | PUT / PATCH | Actualizar un recurso existente | Sí |
| Delete | DELETE | Eliminar un recurso | Sí |

- **Idempotente:** Repetir la operación produce el mismo estado final. POST no lo es (dos POST crean dos recursos). PUT y DELETE sí (dos PUT con los mismos datos dejan el mismo estado).
- **Filosofía REST:** Cada URL representa un **recurso** y el método HTTP indica la **acción**.
  - `/api/alumnos` → la colección de alumnos.
  - `/api/alumnos/{id}` → un alumno específico.

- **En esta actividad:** Se pide implementar los cuatro verbos sobre `/api/alumnos`. Como identificar alumnos por índice puede ser problemático, es común usar la **matrícula** como identificador único.

---

## 2. Identificación única de recursos

- **Problema:** Para editar o eliminar un alumno, hay que poder referirse a él de forma única.
- **Opciones:**
  - **Por matrícula:** Ya es un campo del alumno. Se usa como clave natural. Ej. `PUT /api/alumnos/A001`.
  - **Por índice:** Posición en el vector. Frágil (cambia si se eliminan elementos).
  - **Por ID autogenerado:** Un campo extra que el servidor asigna. Más robusto, pero más trabajo.
- **Recomendación para esta actividad:** Usa la **matrícula** como identificador. Es lo que ya tiene el alumno y es único por diseño.
- **Ruta resultante:**
  - `PUT /api/alumnos/{matricula}`
  - `DELETE /api/alumnos/{matricula}`
- **Alternativa simple:** Enviar la matrícula en el cuerpo del PUT/DELETE. Funciona, pero no es tan RESTful.

---

## 3. Enrutamiento con parámetros de ruta

- **Problema:** Las rutas `/api/alumnos/A001` cambian según el recurso. No puedes comparar con `==` directamente.
- **Solución:** Detectar el prefijo y extraer el parámetro.
  - Verifica si la ruta **empieza con** `/api/alumnos/`.
  - El resto de la ruta es el identificador.
- **Fragmento suelto (idea):**
  ```cpp
  if (ruta.rfind("/api/alumnos/", 0) == 0) {
      string id = ruta.substr(13); // longitud de "/api/alumnos/"
  }
  ```
- **`rfind(str, 0) == 0`** equivale a "empieza con" (no hay `startsWith` en C++ estándar).
- **Cuidado con rutas exactas:** `/api/alumnos` (sin barra al final) es la colección; `/api/alumnos/X` es un recurso concreto. Ambos casos deben manejarse.
- **Combinación con el método HTTP:** El mismo path (`/api/alumnos`) puede recibir GET, POST, PUT, DELETE. Debes enrutar **por método y por ruta**.

**Estructura típica de enrutamiento:**
```cpp
if (ruta == "/api/alumnos" && metodo == "GET") { /* listar */ }
else if (ruta == "/api/alumnos" && metodo == "POST") { /* crear */ }
else if (ruta.rfind("/api/alumnos/", 0) == 0 && metodo == "PUT") { /* actualizar */ }
else if (ruta.rfind("/api/alumnos/", 0) == 0 && metodo == "DELETE") { /* eliminar */ }
else { /* 404 */ }
```

- **Alternativa:** Enrutar primero por ruta y luego por método (switch interno).
- **Recomendación:** Con 4 métodos y 2 variantes de ruta, `if-else` es claro. Si crece, migra a un mapa de handlers.

---

## 4. Modelo en memoria

- **Fuente de verdad:** El vector de alumnos en memoria.
- **Sincronización:** El archivo en disco **debe reflejar siempre** el estado del vector.
- **Flujo de cada operación:**
  1. Recibir petición.
  2. Modificar el vector en memoria.
  3. Reescribir el archivo completo desde el vector.
  4. Responder al cliente.
- **Ventaja de reescribir todo:** El archivo queda siempre consistente. No hay duplicados, no hay registros huérfanos.
- **Desventaja:** Si el archivo es grande, reescribir es costoso. Para esta actividad, con decenas o cientos de registros, es perfectamente aceptable.
- **Alternativa (append):** Solo válida para Create. Para Update y Delete, es inevitable reescribir.

**Regla de oro:** Guarda el archivo **después de cada operación que modifique el estado**, no al cerrar el servidor. Así, si el servidor se cae, los datos ya están persistidos.

---

## 5. CREATE — POST /api/alumnos

- **Request:** POST con JSON en el cuerpo: `{"nombre":"Ana","edad":20,"matricula":"A001"}`.
- **Pasos:**
  1. Leer el cuerpo.
  2. Parsear el JSON.
  3. Validar campos (no vacíos, edad numérica, matrícula no duplicada).
  4. **Verificar que la matrícula no exista ya** (es la clave única).
  5. Añadir el alumno al vector.
  6. Reescribir el archivo completo.
  7. Responder con `201 Created` y el recurso creado.
- **Códigos de estado:**
  - `201 Created` → éxito.
  - `400 Bad Request` → datos inválidos.
  - `409 Conflict` → matrícula duplicada (opcional pero correcto).
  - `500` → error de escritura.
- **Respuesta típica:** El objeto creado con todos sus campos.
- **Verificación de duplicados:** Recorrer el vector buscando la matrícula antes de insertar.

---

## 6. READ — GET /api/alumnos

- **Request:** GET sin cuerpo.
- **Pasos:**
  1. Leer el archivo o usar el vector en memoria (más rápido).
  2. Serializar el vector completo a JSON.
  3. Responder con `200 OK` y el arreglo.
- **Variantes:**
  - `GET /api/alumnos` → lista completa.
  - `GET /api/alumnos/{matricula}` → un solo alumno (opcional).
- **Códigos de estado:**
  - `200 OK` → éxito (incluso si el arreglo está vacío).
  - `404` → solo si pides un alumno específico que no existe.
- **Recomendación:** Mantén el vector en memoria y responde desde ahí. Es mucho más rápido que leer el archivo en cada petición.

---

## 7. UPDATE — PUT /api/alumnos/{matricula}

- **Request:** PUT con JSON en el cuerpo: `{"nombre":"Ana López","edad":21,"matricula":"A001"}`.
- **El parámetro `{matricula}`** es la clave del alumno a actualizar.
- **Semántica PUT vs PATCH:**
  - **PUT:** Reemplaza el recurso completo. Envías todos los campos.
  - **PATCH:** Actualiza solo los campos enviados. Más complejo de implementar.
  - **Para esta actividad:** Usa PUT y envía todos los campos.
- **Pasos:**
  1. Extraer la matrícula de la ruta.
  2. Leer y parsear el cuerpo.
  3. Validar los campos nuevos.
  4. **Buscar el alumno en el vector** por matrícula.
  5. Si no existe → `404 Not Found`.
  6. Si existe → actualizar sus campos.
  7. Reescribir el archivo completo.
  8. Responder con `200 OK` y el recurso actualizado.
- **Consideración:** ¿Puede cambiar la matrícula? Si sí, hay que verificar que la nueva no esté duplicada. Si no, la matrícula es inmutable y solo se actualizan nombre y edad.
- **Recomendación:** Mantén la matrícula inmutable. Es la clave del recurso.

---

## 8. DELETE — DELETE /api/alumnos/{matricula}

- **Request:** DELETE sin cuerpo. La matrícula va en la ruta.
- **Pasos:**
  1. Extraer la matrícula de la ruta.
  2. Buscar el alumno en el vector por matrícula.
  3. Si no existe → `404 Not Found`.
  4. Si existe → eliminarlo del vector.
  5. Reescribir el archivo completo.
  6. Responder con `200 OK` o `204 No Content`.
- **Eliminación del vector:**
  - Con `std::vector`, usa `erase` con el iterador del elemento encontrado.
  - El vector se desplaza automáticamente; no hay huecos.
- **Respuesta:** Puede ser un mensaje de confirmación o simplemente sin cuerpo (204).
- **Códigos de estado:**
  - `200 OK` con mensaje.
  - `204 No Content` si no devuelves cuerpo.
  - `404` si el recurso no existe.

---

## 9. Persistencia: reescribir el archivo completo

- **Cuándo:** Después de cada operación que cambia el estado (POST, PUT, DELETE).
- **Cómo:**
  1. Abrir el archivo con `ofstream` en modo **trunc** (por defecto).
  2. Recorrer el vector en memoria.
  3. Escribir una línea por alumno: `nombre|edad|matricula\n`.
  4. Cerrar el archivo.
- **Verificación:** Tras cerrar, verifica con `.fail()` que no hubo error.
- **Cuidado:** Abrir con `ofstream` sin `ios::app` **trunca el archivo**. Eso es exactamente lo que quieres: reescribir desde cero.
- **Orden de operaciones:** Modifica el vector **primero**, luego reescribe. Si el archivo falla, al menos el estado en memoria está correcto para el resto de la sesión.

**Fragmento suelto (idea):**
```cpp
ofstream archivo("data/usuarios.txt");
for (const auto& a : alumnos) {
    archivo << a.nombre << "|" << a.edad << "|" << a.matricula << "\n";
}
archivo.close();
```

- **Sin append:** No uses `ios::app` aquí. Necesitas truncar.
- **Alternativa segura:** Escribir a `usuarios.tmp` y luego `rename` sobre `usuarios.txt`. Si algo falla a mitad, el original no se corrompe.
- **Para esta actividad:** Escribir directamente es suficiente.

---

## 10. Funciones auxiliares recomendadas

- **`cargarAlumnos()`:** Lee el archivo y llena el vector. Se llama una vez al arrancar.
- **`guardarAlumnos()`:** Reescribe el archivo completo desde el vector. Se llama tras cada operación que modifica el estado.
- **`buscarPorMatricula(const string& matricula)`:** Devuelve el índice del alumno en el vector, o -1 si no existe.
- **`existeMatricula(const string& matricula)`:** Retorna bool. Útil antes de insertar.
- **`parsearAlumnoJSON(const string& cuerpo)`:** Devuelve un struct Alumno con los campos extraídos.
- **`alumnoAJSON(const Alumno& a)`:** Serializa un alumno a objeto JSON.
- **`vectorAJSON(const vector<Alumno>& v)`:** Serializa todo el vector a un arreglo JSON.
- **`responder(cliente, codigo, mime, cuerpo)`:** Construye y envía una respuesta HTTP.
- **Ventaja:** Cada función hace una cosa. El código queda legible y reutilizable.

---

## 11. Códigos de estado HTTP completos

| Código | Significado | Cuándo usarlo |
|--------|-------------|----------------|
| 200 | OK | GET, PUT, DELETE con cuerpo de respuesta |
| 201 | Created | POST exitoso |
| 204 | No Content | DELETE exitoso sin cuerpo |
| 400 | Bad Request | Datos inválidos, JSON mal formado |
| 404 | Not Found | Recurso no existe |
| 405 | Method Not Allowed | Método no soportado para esa ruta |
| 409 | Conflict | Matrícula duplicada |
| 415 | Unsupported Media Type | Content-Type incorrecto |
| 500 | Internal Server Error | Error de archivo o excepción |

- **Sé consistente:** Un mismo tipo de error siempre debe devolver el mismo código.
- **Cuerpo de error:** Envía un JSON con `{"error":"..."}` para que el cliente pueda mostrar el mensaje.

---

## 12. Lado cliente: la interfaz CRUD

### A. Registrar (Create)
- Formulario con campos nombre, edad, matrícula.
- Botón "Registrar" → `fetch` POST con JSON.
- Tras éxito: refrescar la tabla automáticamente.

### B. Mostrar (Read)
- Botón "Actualizar" y/o carga automática al abrir la página.
- `fetch` GET `/api/alumnos` → renderizar tabla.

### C. Editar (Update)
- Cada fila tiene un botón "Editar".
- Al pulsarlo:
  - Opción 1: Rellenar el formulario con los datos del alumno → el usuario modifica → "Guardar cambios" envía PUT.
  - Opción 2: Usar `prompt()` para cada campo (simple pero poco elegante).
  - Opción 3: Convertir la fila en editable con `contenteditable` o inputs inline.
- **Recomendación para esta actividad:** Usar la opción 1 (formulario reutilizado). Es la más limpia y práctica.

### D. Eliminar (Delete)
- Cada fila tiene un botón "Eliminar".
- Al pulsarlo:
  - `confirm("¿Eliminar a X?")` para confirmar.
  - Si acepta → `fetch` DELETE `/api/alumnos/{matricula}`.
  - Tras éxito: refrescar la tabla.

- **Cómo saber qué fila es cuál:** Cada fila lleva un `data-matricula` o un `data-id`. Al pulsar un botón, se lee ese atributo para saber qué recurso modificar o eliminar.
- **Delegación de eventos:** Escucha el click en el `<tbody>` y detecta qué botón se pulsó. Funciona aunque las filas se regeneren.

---

## 13. Reutilizar el formulario para editar

- **Modo "crear":** El formulario está vacío y el botón dice "Registrar". Al enviar → POST.
- **Modo "editar":** El formulario está prellenado con los datos del alumno. El botón dice "Guardar cambios". Al enviar → PUT.
- **Cómo saber en qué modo está:** Una variable de estado (`modoEdicion = true/false`) o un campo oculto en el formulario con la matrícula original.
- **Cambio de modo:**
  - Al pulsar "Editar" en una fila → `modoEdicion = true`, rellenar campos, cambiar texto del botón.
  - Al terminar la edición o pulsar "Cancelar" → `modoEdicion = false`, limpiar formulario, restaurar el botón.
- **Ventaja:** Un solo formulario maneja ambos casos. Menos HTML, menos duplicación.

**Fragmento suelto (idea de estado):**
```js
let modoEdicion = false;
let matriculaEditando = null;
```

---

## 14. Confirmaciones y feedback

- **Antes de eliminar:** `confirm()` para evitar borrados accidentales. Es feo pero funcional. Alternativa: un modal personalizado.
- **Durante la operación:** Deshabilitar el botón, mostrar "Guardando..." o "Eliminando...".
- **Tras éxito:**
  - Mensaje visual ("Alumno registrado", "Cambios guardados", "Alumno eliminado").
  - Refrescar la tabla con `cargarAlumnos()`.
  - Si estabas editando, salir del modo edición.
- **Tras error:**
  - Mostrar el mensaje del servidor (si viene en JSON).
  - No limpies el formulario para que el usuario corrija.
  - Log en consola para depurar.

---

## 15. Refrescar la tabla tras cada operación

- **Patrón:** Tras cualquier POST, PUT o DELETE exitoso, llamar a `cargarAlumnos()`.
- **Ventaja:** El frontend siempre muestra el estado real del servidor. No hay que sincronizar manualmente.
- **Desventaja:** Una petición extra al servidor por cada operación. Aceptable en este proyecto.
- **Optimización posible:** Actualizar solo la fila afectada. Más complejo y propenso a errores. **No lo hagas en esta actividad.**

---

## 16. Validaciones en cliente y servidor

### A. Cliente (JS)
- Campos no vacíos.
- Edad numérica y positiva.
- Matrícula no vacía.
- Mensajes de error visibles al usuario.
- **Objetivo:** Buena UX, evitar peticiones innecesarias.

### B. Servidor (C++)
- **Obligatorio.** Nunca confíes en el cliente.
- Verifica que los campos existan y tengan el tipo correcto.
- Verifica que la matrícula no esté duplicada en POST.
- Verifica que el recurso exista en PUT y DELETE.
- Verifica el `Content-Type` en POST y PUT.
- Responde con el código HTTP adecuado.

**Regla de oro:** El cliente valida para el usuario; el servidor valida para la integridad.

---

## 17. Consistencia del archivo

- **Después de cada operación,** el archivo debe reflejar exactamente el vector en memoria.
- **Si el archivo se escribe parcialmente** (por crash en medio de la escritura), queda corrupto.
- **Mitigación:** Escribir en archivo temporal y renombrar.
  - `usuarios.tmp` → se escribe completo → `rename` a `usuarios.txt`.
  - El `rename` es atómico en la mayoría de los sistemas operativos.
- **Para esta actividad:** Escribir directamente es aceptable. La mitigación es un plus si quieres robustez.
- **Verificación:** Tras cada operación, abre el archivo y comprueba que refleja los cambios.

---

## 18. Concurrencia (nota)

- **Servidor secuencial:** Atiende una petición a la vez. No hay problema de concurrencia.
- **Servidor con hilos:** Múltiples peticiones pueden modificar el vector simultáneamente → condiciones de carrera.
- **Solución:** Mutex (`std::mutex`) protegiendo el vector y el archivo.
- **Para esta actividad:** Se asume servidor secuencial. No te preocupes por concurrencia. Pero tenlo en cuenta si el proyecto crece.

---

## 19. Pruebas

- **Con `curl`:**
  - Crear: `curl -X POST -H "Content-Type: application/json" -d '{"nombre":"Ana","edad":20,"matricula":"A001"}' http://localhost:8080/api/alumnos`
  - Listar: `curl http://localhost:8080/api/alumnos`
  - Actualizar: `curl -X PUT -H "Content-Type: application/json" -d '{"nombre":"Ana López","edad":21,"matricula":"A001"}' http://localhost:8080/api/alumnos/A001`
  - Eliminar: `curl -X DELETE http://localhost:8080/api/alumnos/A001`
- **Con navegador + DevTools:** Verifica cada operación en Network, revisa Payload y Response.
- **Casos a probar:**
  - Crear con matrícula nueva → 201.
  - Crear con matrícula existente → 409.
  - Crear con campos vacíos → 400.
  - Listar con arreglo vacío → 200 con `[]`.
  - Actualizar existente → 200.
  - Actualizar inexistente → 404.
  - Eliminar existente → 200 o 204.
  - Eliminar inexistente → 404.
  - Método no soportado → 405.
- **Persistencia:** Tras cada operación, abre `usuarios.txt` y verifica que refleja el cambio. Reinicia el servidor y confirma que los datos se cargan correctamente.

---

## 20. Buenas prácticas

- **Usa la matrícula como identificador único** en las rutas PUT y DELETE.
- **Verifica duplicados antes de insertar.**
- **Reescribe el archivo tras cada operación que modifique el estado.**
- **Separa las funciones:** leer, escribir, buscar, parsear, serializar, responder.
- **Un handler por combinación método+ruta.** 

---
# PASO A PASO

---
# Actividad 31 — CRUD completo de alumnos

## Objetivo
Implementar **Create, Read, Update, Delete** sobre `/api/alumnos` en C++ y consumirlos desde el frontend. Todos los cambios se persisten en `usuarios.txt`.

## Correspondencia HTTP

| Operación | Verbo HTTP | Ruta                        |
|-----------|-----------|------------------------------|
| Create    | POST      | `/api/alumnos`              |
| Read      | GET       | `/api/alumnos`              |
| Update    | PUT       | `/api/alumnos/{matricula}`  |
| Delete    | DELETE    | `/api/alumnos/{matricula}`  |

Usamos la **matrícula** como identificador único (es clave natural del alumno).

---

## Paso 1 — Función: buscar por matrícula

```cpp
int buscarPorMatricula(const string& matricula) {
    for (size_t i = 0; i < alumnos.size(); i++) {
        if (alumnos[i].matricula == matricula) return (int)i;
    }
    return -1;
}
```

---

## Paso 2 — CREATE (ya lo tienes de la actividad 29)

El handler de `POST /api/alumnos`:
1. Parsea el cuerpo.
2. Valida campos.
3. Rechaza matrícula duplicada con 409.
4. Agrega al vector.
5. Guarda el archivo.
6. Responde 201 Created.

Ajuste: cambia el 200 por **201** y devuelve el recurso creado en JSON:

```cpp
if (ruta == "/api/alumnos" && metodo == "POST") {
    string cuerpo = extraerCuerpo(peticion);
    DatosAlumno d = parsearAlumno(cuerpo);

    if (!d.valido) {
        return construirRespuesta(400, "application/json; charset=utf-8",
                                  "{\"error\":\"Datos inválidos\"}");
    }

    if (buscarPorMatricula(d.matricula) != -1) {
        return construirRespuesta(409, "application/json; charset=utf-8",
                                  "{\"error\":\"Matrícula duplicada\"}");
    }

    Alumno nuevo{d.nombre, d.edad, d.matricula};
    alumnos.push_back(nuevo);

    if (!guardarAlumnos()) {
        alumnos.pop_back();
        return construirRespuesta(500, "application/json; charset=utf-8",
                                  "{\"error\":\"Error al guardar\"}");
    }

    string json = "{\"nombre\":\""    + escaparJSON(nuevo.nombre)    + "\","
                + "\"edad\":"        + to_string(nuevo.edad)        + ","
                + "\"matricula\":\"" + escaparJSON(nuevo.matricula) + "\"}";
    return construirRespuesta(201, "application/json; charset=utf-8", json);
}
```

---

## Paso 3 — READ (ya lo tienes de la actividad 30)

Si quieres `GET /api/alumnos/{matricula}` para un alumno individual:

```cpp
if (ruta.rfind("/api/alumnos/", 0) == 0 && metodo == "GET") {
    string matricula = ruta.substr(13);   // longitud de "/api/alumnos/"
    int idx = buscarPorMatricula(matricula);
    if (idx == -1) {
        return construirRespuesta(404, "application/json; charset=utf-8",
                                  "{\"error\":\"No encontrado\"}");
    }
    const auto& a = alumnos[idx];
    string json = "{\"nombre\":\""    + escaparJSON(a.nombre)    + "\","
                + "\"edad\":"        + to_string(a.edad)        + ","
                + "\"matricula\":\"" + escaparJSON(a.matricula) + "\"}";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

---

## Paso 4 — UPDATE

```cpp
if (ruta.rfind("/api/alumnos/", 0) == 0 && metodo == "PUT") {
    string matricula = ruta.substr(13);
    int idx = buscarPorMatricula(matricula);

    if (idx == -1) {
        return construirRespuesta(404, "application/json; charset=utf-8",
                                  "{\"error\":\"Alumno no encontrado\"}");
    }

    string cuerpo = extraerCuerpo(peticion);
    DatosAlumno d = parsearAlumno(cuerpo);

    if (!d.valido) {
        return construirRespuesta(400, "application/json; charset=utf-8",
                                  "{\"error\":\"Datos inválidos\"}");
    }

    // Actualizar (matrícula inmutable)
    Alumno respaldo = alumnos[idx];
    alumnos[idx].nombre = d.nombre;
    alumnos[idx].edad   = d.edad;

    if (!guardarAlumnos()) {
        alumnos[idx] = respaldo;
        return construirRespuesta(500, "application/json; charset=utf-8",
                                  "{\"error\":\"Error al guardar\"}");
    }

    string json = "{\"nombre\":\""    + escaparJSON(alumnos[idx].nombre)    + "\","
                + "\"edad\":"        + to_string(alumnos[idx].edad)         + ","
                + "\"matricula\":\"" + escaparJSON(alumnos[idx].matricula)  + "\"}";
    return construirRespuesta(200, "application/json; charset=utf-8", json);
}
```

---

## Paso 5 — DELETE

```cpp
if (ruta.rfind("/api/alumnos/", 0) == 0 && metodo == "DELETE") {
    string matricula = ruta.substr(13);
    int idx = buscarPorMatricula(matricula);

    if (idx == -1) {
        return construirRespuesta(404, "application/json; charset=utf-8",
                                  "{\"error\":\"Alumno no encontrado\"}");
    }

    Alumno respaldo = alumnos[idx];
    alumnos.erase(alumnos.begin() + idx);

    if (!guardarAlumnos()) {
        alumnos.insert(alumnos.begin() + idx, respaldo);
        return construirRespuesta(500, "application/json; charset=utf-8",
                                  "{\"error\":\"Error al guardar\"}");
    }

    return construirRespuesta(200, "application/json; charset=utf-8",
                              "{\"ok\":true}");
}
```

🎯 El respaldo + reversión mantiene la coherencia entre memoria y archivo si la escritura falla.

---

## Paso 6 — Columna de acciones en la tabla

Actualiza el `<thead>`:

```html
<tr>
  <th>Matrícula</th>
  <th>Nombre</th>
  <th>Edad</th>
  <th>Acciones</th>
</tr>
```

Y en `renderizarTabla`, añade la celda con botones:

```js
const tdAcciones = document.createElement('td');
tdAcciones.className = 'acciones';

const btnEditar = document.createElement('button');
btnEditar.textContent = 'Editar';
btnEditar.dataset.accion = 'editar';
btnEditar.dataset.matricula = a.matricula;

const btnEliminar = document.createElement('button');
btnEliminar.textContent = 'Eliminar';
btnEliminar.dataset.accion = 'eliminar';
btnEliminar.dataset.matricula = a.matricula;

tdAcciones.appendChild(btnEditar);
tdAcciones.appendChild(btnEliminar);

tr.appendChild(tdAcciones);
```

---

## Paso 7 — Delegación de eventos en la tabla

```js
tbody.addEventListener('click', (e) => {
  const boton = e.target.closest('button[data-accion]');
  if (!boton) return;

  const matricula = boton.dataset.matricula;
  const accion    = boton.dataset.accion;

  if (accion === 'editar')   editarAlumno(matricula);
  if (accion === 'eliminar') eliminarAlumno(matricula);
});
```

---

## Paso 8 — Modo edición en el formulario

Reutilizamos el mismo formulario. Variable de estado:

```js
let modoEdicion = false;
let matriculaEditando = null;
```

**Editar** → rellenar campos y marcar modo:

```js
function editarAlumno(matricula) {
  // Buscar en la tabla actual
  const fila = [...tbody.querySelectorAll('tr')].find(
    tr => tr.children[0]?.textContent === matricula
  );
  if (!fila) return;

  document.querySelector('#nombre').value    = fila.children[1].textContent;
  document.querySelector('#edad').value      = fila.children[2].textContent;
  document.querySelector('#matricula').value = matricula;

  modoEdicion = true;
  matriculaEditando = matricula;

  const btn = document.querySelector('#form-alumno button[type="submit"]');
  btn.textContent = 'Guardar cambios';

  // Bloquear el campo matrícula
  document.querySelector('#matricula').readOnly = true;
}
```

Ajusta el listener del `submit` para decidir POST o PUT:

```js
formAlumno.addEventListener('submit', async (e) => {
  e.preventDefault();

  const datos = {
    nombre:    document.querySelector('#nombre').value.trim(),
    edad:      Number(document.querySelector('#edad').value),
    matricula: document.querySelector('#matricula').value.trim()
  };

  const url    = modoEdicion
    ? `/api/alumnos/${matriculaEditando}`
    : '/api/alumnos';
  const method = modoEdicion ? 'PUT' : 'POST';

  try {
    const res = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(datos)
    });

    if (!res.ok) throw new Error(`HTTP ${res.status}`);

    salirModoEdicion();
    formAlumno.reset();
    await cargarAlumnos();
  } catch (err) {
    console.error(err);
    tablaEstado.textContent = `Error: ${err.message}`;
  }
});

function salirModoEdicion() {
  modoEdicion = false;
  matriculaEditando = null;
  document.querySelector('#matricula').readOnly = false;
  document.querySelector('#form-alumno button[type="submit"]').textContent = 'Registrar';
}
```

---

## Paso 9 — Eliminar

```js
async function eliminarAlumno(matricula) {
  if (!confirm(`¿Eliminar al alumno con matrícula ${matricula}?`)) return;

  try {
    const res = await fetch(`/api/alumnos/${matricula}`, { method: 'DELETE' });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    await cargarAlumnos();
  } catch (err) {
    console.error(err);
    tablaEstado.textContent = `Error al eliminar: ${err.message}`;
  }
}
```

---

## Paso 10 — Probar el ciclo completo

**Create**
1. Rellena el formulario.
2. Envía → 201.
3. La tabla se refresca.

**Read**
4. Recarga la página → los datos siguen.

**Update**
5. Pulsa **Editar** → el formulario se prellena con los datos.
6. Cambia el nombre y guarda → 200.
7. La tabla refleja el cambio.

**Delete**
8. Pulsa **Eliminar** → confirmación → 200.
9. La fila desaparece.
10. Reinicia el servidor → el archivo `usuarios.txt` refleja el cambio.

---

## Paso 11 — Probar con curl (opcional)

```bash
# Crear
curl -i -X POST -H "Content-Type: application/json" \
  -d '{"nombre":"Ana","edad":20,"matricula":"A001"}' \
  http://localhost:8080/api/alumnos

# Listar
curl -i http://localhost:8080/api/alumnos

# Actualizar
curl -i -X PUT -H "Content-Type: application/json" \
  -d '{"nombre":"Ana López","edad":21,"matricula":"A001"}' \
  http://localhost:8080/api/alumnos/A001

# Eliminar
curl -i -X DELETE http://localhost:8080/api/alumnos/A001
```

Verifica en la consola del servidor y en `usuarios.txt` que todo se aplicó.

---

## ✅ Checklist de la Actividad 31

- [ ] POST responde 201 con el recurso creado.
- [ ] POST con matrícula duplicada responde 409.
- [ ] GET `/api/alumnos` responde el arreglo completo.
- [ ] PUT sobre matrícula existente responde 200 y actualiza.
- [ ] PUT sobre matrícula inexistente responde 404.
- [ ] DELETE sobre existente responde 200.
- [ ] DELETE sobre inexistente responde 404.
- [ ] `usuarios.txt` refleja cada operación.
- [ ] El botón Editar prellena el formulario y envía PUT.
- [ ] El botón Eliminar pide confirmación antes de borrar.
- [ ] Tras cualquier operación exitosa, la tabla se refresca.

## 🧠 Mini-reto
Añade un botón **Cancelar** al lado de "Guardar cambios" cuando estés en modo edición. Al pulsarlo, limpia el formulario y vuelve al modo "Registrar".