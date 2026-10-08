# Actividad 25. Introducción a JSON

## Temas

- Objetos
    
- JSON
    
- Serialización
    

## Ejercicio

Crear un objeto `Alumno` en JavaScript.

Convertirlo a JSON.

Mostrar el resultado en pantalla.

Después recibir un JSON y convertirlo nuevamente en un objeto.

Comparar un objeto JavaScript con su representación JSON.

---
# TEORÍA

---

## 1. ¿Qué es un objeto en JavaScript?

- **Definición:** Un objeto es una colección de **pares clave-valor**. Cada clave es un string (o símbolo), cada valor puede ser cualquier tipo: número, string, booleano, array, otro objeto, función, `null`.
- **Sintaxis literal:**
  ```js
  const alumno = {
      nombre: "Ana",
      edad: 20,
      matricula: "2024001"
  };
  ```
- **Acceso:**
  - Notación punto: `alumno.nombre`
  - Notación corchetes: `alumno["nombre"]` (útil cuando la clave es dinámica).
- **Modificación:**
  - `alumno.edad = 21;`
  - `alumno["matricula"] = "2024002";`
  - Añadir nueva propiedad: `alumno.carrera = "TI";`
  - Eliminar: `delete alumno.edad;`
- **Anidamiento:** Un objeto puede contener otros objetos y arrays.
  ```js
  const alumno = {
      nombre: "Ana",
      materias: ["Mate", "Prog"],
      direccion: { ciudad: "Mexicali", cp: "21000" }
  };
  ```
- **Diferencia con array:** El array es una colección ordenada indexada por números (0, 1, 2…). El objeto es una colección no ordenada indexada por claves.

---

## 2. ¿Qué es JSON?

- **Definición:** JSON (JavaScript Object Notation) es un **formato de texto** para representar datos estructurados. Es un **estándar independiente del lenguaje**, no un objeto de JavaScript.
- **Origen:** Derivado de la sintaxis de objetos de JavaScript, pero es más restrictivo.
- **Función:** Intercambiar datos entre sistemas (frontend ↔ backend, API ↔ app móvil, etc.). Es el formato dominante en APIs modernas.
- **Sintaxis:** Muy parecida a un objeto JS, pero con reglas estrictas.
  ```
  {
      "nombre": "Ana",
      "edad": 20,
      "matricula": "2024001"
  }
  ```

- **Reglas de JSON (más estrictas que JS):**
  - **Claves siempre entre comillas dobles** (`"clave"`). No se permiten comillas simples ni claves sin comillas.
  - **Strings siempre con comillas dobles.**
  - **No se permiten comentarios.**
  - **No se permite coma final** después del último elemento.
  - **No se permiten funciones, `undefined` ni símbolos.** Solo:
    - Strings: `"texto"`
    - Números: `42`, `3.14` (sin `NaN`, `Infinity`)
    - Booleanos: `true`, `false`
    - `null`
    - Arrays: `[ ... ]`
    - Objetos: `{ ... }`
  - **Los valores numéricos** no pueden tener ceros iniciales ni `+` explícito.

- **Valores permitidos en JSON:** string, number, boolean, null, array, object.
- **Valores NO permitidos:** `undefined`, funciones, `NaN`, `Infinity`, comentarios, fechas (se representan como string ISO 8601).

---

## 3. Comparación: objeto JavaScript vs. JSON

| Aspecto | Objeto JavaScript | JSON |
|---------|-------------------|------|
| Naturaleza | Estructura en memoria | Texto plano |
| Uso | Manipular datos en código | Intercambiar datos entre sistemas |
| Claves | Pueden ir sin comillas (si son identificadores válidos) | Siempre entre comillas dobles |
| Strings | Comillas simples o dobles o backticks | Solo comillas dobles |
| Comentarios | Permitidos | No permitidos |
| Coma final | Permitida en objetos/arrays modernos | No permitida |
| Funciones | Permitidas como valor | No permitidas |
| `undefined` | Permitido | No permitido |
| Fechas | Objeto `Date` | Solo como string (ISO 8601) |
| Tipo | `typeof obj === "object"` | `typeof json === "string"` |
| Acceso | `obj.nombre` | Requiere parseo primero |

**Observación clave:** JSON es **un string**, no un objeto. Para trabajar con él como objeto, hay que **parsearlo**.

---

## 4. Serialización: objeto → JSON

- **Definición:** Convertir un objeto (o cualquier valor JS) en un string con formato JSON.
- **Método:** `JSON.stringify(valor)`.
- **Uso típico:** Antes de enviar datos al servidor, guardarlos en `localStorage`, o mostrarlos por consola.
- **Resultado:** Un string con el objeto en formato JSON.

**Fragmentos sueltos:**
```js
const json = JSON.stringify(alumno);
console.log(json);          // '{"nombre":"Ana","edad":20}'
console.log(typeof json);   // "string"
```

- **Parámetros opcionales de `JSON.stringify`:**
  - **Replacer (función o array):** Filtra o transforma las propiedades incluidas.
    - Array de claves: solo incluye esas.
    - Función: decide qué hacer con cada clave/valor.
  - **Espaciado (número o string):** Añade indentación para legibilidad.
    ```js
    JSON.stringify(alumno, null, 2);   // 2 espacios de indentación
    JSON.stringify(alumno, null, "\t"); // tabulaciones
    ```

- **Qué se pierde al serializar:**
  - **Funciones:** Se omiten.
  - **`undefined`:** Se omiten (o se convierten en `null` dentro de arrays).
  - **`NaN`, `Infinity`, `-Infinity`:** Se convierten en `null`.
  - **Fechas:** Se convierten a string ISO (`"2024-01-15T10:30:00.000Z"`).
  - **Referencias circulares:** Lanzan error (`TypeError: Converting circular structure to JSON`).
  - **Símbolos:** Se omiten.
  - **Propiedades no enumerables:** Se omiten.

---

## 5. Deserialización: JSON → objeto

- **Definición:** Convertir un string con formato JSON en un objeto JavaScript utilizable.
- **Método:** `JSON.parse(string)`.
- **Uso típico:** Al recibir datos del servidor, al leer de `localStorage`, al leer de un archivo.
- **Resultado:** Un objeto (o array, o primitivo) de JavaScript.

**Fragmentos sueltos:**
```js
const json = '{"nombre":"Ana","edad":20}';
const alumno = JSON.parse(json);
console.log(alumno.nombre); // "Ana"
console.log(typeof alumno);  // "object"
```

- **Errores comunes:**
  - Si el string no es JSON válido, `JSON.parse` lanza `SyntaxError`.
  - Comillas simples en lugar de dobles → error.
  - Coma final → error.
  - Comentarios → error.
  - `undefined` dentro del JSON → error.

- **Manejo de errores:** Usa `try-catch` para parsear strings que pueden ser inválidos (por ejemplo, entrada del usuario o datos de red).
  ```js
  try {
      const obj = JSON.parse(texto);
  } catch (e) {
      console.error("JSON inválido", e);
  }
  ```

- **Reviver (segundo parámetro):** Una función que transforma los valores durante el parseo. Útil para convertir strings ISO a objetos `Date`.
  ```js
  JSON.parse(texto, (clave, valor) => {
      if (clave === "fechaNacimiento") return new Date(valor);
      return valor;
  });
  ```

---

## 6. Ciclo completo: objeto → JSON → objeto

1. **Objeto original:** En memoria, con métodos, referencias, etc.
2. **Serialización:** `JSON.stringify(obj)` → string JSON.
3. **Transporte/almacenamiento:** El string viaja por red o se guarda.
4. **Deserialización:** `JSON.parse(string)` → nuevo objeto.
5. **Objeto reconstruido:** En memoria otra vez, pero **sin las funciones ni la identidad del original**.

- **Importante:** El objeto reconstruido **no es el mismo objeto** que el original. Es una **copia estructural** con los mismos datos primitivos, pero:
  - Las funciones se perdieron.
  - Las fechas son strings (a menos que uses un reviver).
  - Las referencias compartidas se duplican (dos propiedades que apuntaban al mismo objeto ahora apuntan a dos objetos distintos).
  - Los métodos de la clase (si era instancia) se pierden: queda como objeto plano.

---

## 7. Serialización y clases

- **Con instancias de clases:**
  - `JSON.stringify` solo serializa las **propiedades enumerables propias**, no los métodos del prototipo.
  - Al parsear, el resultado es un **objeto plano**, no una instancia de la clase.
  - Los métodos `get`/`set` no se serializan.
  - Los campos privados (`#`) **no se serializan**.
- **Para preservar la clase al deserializar:** Necesitas reconstruir manualmente la instancia a partir de los datos (por ejemplo, llamando al constructor con los valores del objeto plano).
- **`toJSON()` (opcional):** Si un objeto tiene un método `toJSON`, `JSON.stringify` lo llamará automáticamente y usará su resultado.
  - Útil para personalizar qué se serializa y cómo.
  - Si devuelves un string, `stringify` lo envuelve con comillas.

---

## 8. Aplicaciones típicas de JSON

- **APIs:** Intercambio de datos entre frontend y backend (fetch, XMLHttpRequest).
- **`localStorage` / `sessionStorage`:** Solo guardan strings; hay que serializar antes y parsear después.
  ```js
  localStorage.setItem("alumno", JSON.stringify(alumno));
  const alumno = JSON.parse(localStorage.getItem("alumno"));
  ```
- **Archivos de configuración:** `package.json`, `tsconfig.json`, etc.
- **Logs estructurados:** Registros con campos consistentes.
- **Bases de datos NoSQL:** MongoDB, CouchDB almacenan documentos en formato BSON (similar a JSON).

---

## 9. Buenas prácticas

- **`JSON.stringify`** antes de enviar/guardar un objeto.
- **`JSON.parse`** después de recibir/leer.
- **Siempre `try-catch` al parsear** datos que vienen de fuera (usuario, red, archivos).
- **No uses `eval`** para parsear JSON; usa `JSON.parse`.
- **No confundas** un objeto JS con un string JSON. `typeof` te lo dice:
  - `typeof obj === "object"` → objeto.
  - `typeof json === "string"` → JSON.
- **Claves en camelCase o snake_case,** consistentes en todo el proyecto.
- **Indentación con `JSON.stringify(obj, null, 2)`** para logs legibles, sin indentación para transporte (ahorra bytes).
- **Cuida las fechas:** Se convierten a string ISO. Si necesitas `Date`, usa un reviver.
- **Cuida las referencias circulares:** Rompen `JSON.stringify`. Si tu estructura puede tenerlas, considera una librería externa o una serialización personalizada.
- **Valida la forma de los datos** al parsear (por ejemplo, que las propiedades esperadas existan), especialmente si vienen del usuario.
- **Evita serializar objetos enormes** si solo necesitas algunos campos; usa el replacer para filtrar.
- **No guardes datos sensibles** en JSON si va a viajar sin cifrado.

---

## 10. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **`JSON.parse` lanza `SyntaxError`** | El string no es JSON válido. Verifica comillas dobles, sin comas finales, sin comentarios. |
| **Claves sin comillas** | En JSON las claves siempre llevan comillas dobles. |
| **Comillas simples** | JSON solo acepta comillas dobles para strings y claves. |
| **Coma final** | JSON no permite coma después del último elemento. |
| **`undefined` en el JSON** | No es válido en JSON. Usa `null` o elimina la propiedad. |
| **`NaN`, `Infinity`** | Se convierten a `null` al serializar. No son válidos en JSON. |
| **Función dentro del objeto no aparece tras parsear** | Se perdió en la serialización. Las funciones no son serializables. |
| **`Date` pierde su tipo** | Se serializa como string ISO. Usa un reviver para reconstruirla. |
| **Objeto reconstruido no es instancia de la clase** | `JSON.parse` devuelve objetos planos. Reconstruye la instancia manualmente. |
| **`Converting circular structure to JSON`** | Hay una referencia circular. Elimínala o usa una librería que lo maneje. |
| **Objeto original y reconstruido no son `===`** | `===` compara referencias; son objetos distintos con los mismos datos. |
| **`localStorage` guarda `[object Object]`** | Falta `JSON.stringify` al guardar. |

---

## 11. Comparación: objeto original vs. objeto reconstruido

Después de `JSON.parse(JSON.stringify(obj))`:

| Aspecto | Objeto original | Objeto reconstruido |
|---------|-----------------|---------------------|
| Propiedades primitivas | Sí | Sí (copiadas) |
| Referencias a otros objetos | Sí | Se duplican (ya no comparte referencias) |
| Métodos propios | Sí | **Perdidos** |
| Métodos del prototipo (clase) | Sí | **Perdidos** (es objeto plano) |
| `instanceof Clase` | `true` | **`false`** |
| Fechas | Objeto `Date` | String ISO (a menos que uses reviver) |
| `undefined` | Posible | **Perdido o convertido a `null`** |
| Funciones | Posibles | **Perdidas** |
| Identidad (`===` con el original) | — | **`false`** |
| Comparación por contenido | Requiere método propio | Requiere comparar campo a campo |

**Conclusión:** JSON es un formato de **datos**, no de **comportamiento**. Serializar y deserializar preserva la información, pero no la identidad, las funciones ni el tipo exacto.

---

## 12. Cómo "comparar" un objeto con su JSON

- **Visualmente:** Muestra el objeto y su `JSON.stringify` con `console.log`.
  - El objeto aparece como `{ nombre: "Ana", edad: 20 }` (sin comillas en las claves).
  - El JSON aparece como `'{"nombre":"Ana","edad":20}'` (todo entre comillas, es un string).
- **Con `typeof`:** El objeto es `"object"`, el JSON es `"string"`.
- **Modificando uno:** Cambia una propiedad en el objeto original y verifica que el JSON **no cambia** (es una copia textual).
- **Con `JSON.parse`:** Recupera un nuevo objeto desde el JSON y compáralo con el original:
  - `obj1 === obj2` → `false` (son objetos distintos).
  - Compara campo a campo para verificar igualdad de contenido.

---

## 13. Resumen de conceptos clave

- **Objeto JS:** Estructura en memoria con pares clave-valor, permite funciones y tipos nativos.
- **JSON:** Formato de texto para intercambio de datos. Reglas estrictas (comillas dobles, sin comentarios, sin comas finales, sin funciones).
- **Serialización:** `JSON.stringify` → objeto a string JSON. Pierde funciones, `undefined`, tipos de fecha, identidad.
- **Deserialización:** `JSON.parse` → string JSON a objeto. Devuelve objetos planos, no instancias de clase.
- **Ciclo completo:** objeto → JSON → objeto. Copia estructural, no preserva identidad ni comportamiento.
- **Tipos válidos en JSON:** string, number, boolean, null, array, object.
- **Tipos NO válidos:** `undefined`, funciones, `NaN`, `Infinity`, comentarios, fechas (van como string ISO).
- **Replacer / Reviver:** Parámetros opcionales de `stringify`/`parse` para filtrar o transformar.
- **`toJSON()`:** Método opcional que personaliza la serialización de una clase.
- **`localStorage`:** Requiere `JSON.stringify` al guardar y `JSON.parse` al leer.
- **Buenas prácticas:** `try-catch` al parsear, cuidar fechas y referencias circulares, no confundir objeto con string.
- **Comparación:** `===` compara referencias, no contenido. JSON es un string, no un objeto.

---
# PASO A PASO

---
# Actividad 25 — Introducción a JSON en JavaScript

## Objetivo
Serializar objetos JS a JSON y deserializar JSON a objetos. Comparar ambos. Todo ocurre en el navegador, sin backend todavía.

## Idea clave
- **Objeto JS**: estructura en memoria (puede tener funciones, fechas, etc.).
- **JSON**: string con formato de texto, más restrictivo.

---

## Paso 1 — Abrir la consola del navegador

Con cualquier página abierta, presiona F12 → pestaña **Console**. Ahí puedes probar todo lo de esta actividad sin escribir archivos.

---

## Paso 2 — Crear un objeto alumno

```js
const alumno = {
  nombre: "Ana",
  edad: 20,
  matricula: "A001",
  materias: ["Cálculo", "Programación"],
  direccion: { ciudad: "Mexicali", cp: "21000" }
};

console.log(alumno);
console.log(typeof alumno);   // "object"
```

---

## Paso 3 — Serializar a JSON

```js
const json = JSON.stringify(alumno);
console.log(json);
console.log(typeof json);     // "string"
```

Verás algo como:
```
{"nombre":"Ana","edad":20,"matricula":"A001","materias":["Cálculo","Programación"],"direccion":{"ciudad":"Mexicali","cp":"21000"}}
```

⚠️ Fíjate: claves y strings van con **comillas dobles**. Es un string, no un objeto.

---

## Paso 4 — Versión legible

```js
console.log(JSON.stringify(alumno, null, 2));
```

Los parámetros son:
1. El valor a serializar.
2. Un **replacer** (aquí `null`, no filtra nada).
3. El **espaciado** (2 espacios de indentación).

---

## Paso 5 — Comprobar que son cosas distintas

```js
alumno.nombre = "Cambiado";
console.log(alumno.nombre);   // "Cambiado"
console.log(json);            // el string JSON sigue diciendo "Ana"
```

El JSON es una **copia textual** tomada en el momento del `stringify`. No se actualiza solo.

---

## Paso 6 — Deserializar

```js
const jsonStr = '{"nombre":"Luis","edad":22,"matricula":"A002"}';
const otro = JSON.parse(jsonStr);

console.log(otro);            // {nombre: 'Luis', edad: 22, matricula: 'A002'}
console.log(otro.nombre);     // "Luis"
console.log(typeof otro);     // "object"
```

---

## Paso 7 — Identidad vs contenido

```js
const copia = JSON.parse(JSON.stringify(alumno));
console.log(copia === alumno);              // false
console.log(copia.nombre === alumno.nombre); // true
```

`===` compara referencias, no contenido. `parse(stringify(x))` produce un **objeto nuevo**, sin funciones ni identidad.

---

## Paso 8 — Qué se pierde al serializar

```js
const obj = {
  nombre: "Ana",
  saludar: () => "hola",         // función
  indefinido: undefined,
  infinito: Infinity,
  fecha: new Date()
};

console.log(JSON.stringify(obj));
// {"nombre":"Ana","infinito":null,"fecha":"2025-01-15T..."}
```

Observaciones:
- La **función** desaparece.
- `undefined` desaparece.
- `Infinity` se convierte a `null`.
- La **fecha** se serializa como string ISO.

---

## Paso 9 — Manejo de errores al parsear

```js
try {
  JSON.parse("{nombre: 'sin comillas'}");   // inválido
} catch (e) {
  console.error("JSON inválido:", e.message);
}
```

Reglas que JSON **NO** permite:
- Comillas simples.
- Claves sin comillas.
- Coma final.
- Comentarios.
- `undefined`, `NaN`, `Infinity`.

---

## Paso 10 — Guardar en localStorage

`localStorage` solo guarda strings. Ejemplo:

```js
localStorage.setItem("alumno", JSON.stringify(alumno));

const recuperado = JSON.parse(localStorage.getItem("alumno"));
console.log(recuperado.nombre);
```

---

## ✅ Checklist de la Actividad 25

- [ ] `JSON.stringify` sobre un objeto devuelve un string.
- [ ] `typeof` del objeto es `"object"`; del JSON, `"string"`.
- [ ] Probaste un JSON inválido y capturaste el `SyntaxError`.
- [ ] Guardaste y leíste un objeto en `localStorage`.
- [ ] Viste desaparecer una función al serializar.

## 🧠 Mini-reto
Serializa un arreglo de objetos (3 alumnos) e impleméntalo con `JSON.stringify(alumnos, null, 2)`. Después parsea el resultado y recorre con `forEach`.

---

