# Actividad 16. Primera página web

## Temas

- HTML
    
- Etiquetas
    
- Formularios
    

## Ejercicio

Crear la carpeta

```text
public/
```

Crear:

```text
index.html
```

Mostrar:

- título
    
- logo
    
- menú
    
- formulario para registrar alumno
    

(No funciona todavía.)

Es solamente la interfaz.

---
# TEORÍA

---

## 1. ¿Qué es HTML?

- **Definición:** HTML (HyperText Markup Language) es el lenguaje de **marcado** que define la **estructura** y el **contenido** de una página web.
- **No es un lenguaje de programación:** No tiene variables, bucles ni funciones. Solo describe qué elementos componen la página.
- **Función en el proyecto:** Será la interfaz visual que verá el usuario. Posteriormente se conectará con un backend en C++ para hacerla funcional.
- **Relación con CSS y JavaScript:**
  - **HTML:** Estructura y contenido (esqueleto).
  - **CSS:** Presentación y estilos (ropa, colores, tamaños).
  - **JavaScript:** Comportamiento e interactividad (músculos, acciones).
- **Para esta actividad:** Solo HTML. Nada de CSS ni JS todavía (o solo lo mínimo indispensable).

---

## 2. Estructura básica de un documento HTML

Todo archivo HTML debe seguir una estructura mínima:

- **`<!DOCTYPE html>`:** Declara que es un documento HTML5.
- **`<html>`:** Elemento raíz que contiene todo el documento.
- **`<head>`:** Contiene metadatos (título, codificación, enlaces a CSS, etc.). No se muestra al usuario.
- **`<body>`:** Contiene todo el contenido visible de la página.

**Fragmento suelto (esqueleto):**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Título de la pestaña</title>
</head>
<body>
    <!-- aquí va el contenido visible -->
</body>
</html>
```

- **`lang="es"`:** Indica el idioma del documento (útil para accesibilidad y SEO).
- **`<meta charset="UTF-8">`:** Permite caracteres especiales (tildes, ñ, emojis).
- **`<title>`:** Aparece en la pestaña del navegador, no en la página.

---

## 3. Etiquetas (tags)

- **Definición:** Son las unidades básicas de HTML. Se escriben entre `<` y `>`.
- **Tipos:**
  - **De apertura y cierre:** `<p> ... </p>` (la mayoría).
  - **Auto-cerradas (vacías):** `<img>`, `<br>`, `<hr>`, `<input>`, `<meta>`, `<link>`.
- **Atributos:** Información adicional dentro de la etiqueta de apertura.
  - **Sintaxis:** `<etiqueta atributo="valor">`
  - **Ejemplos:** `id`, `class`, `src`, `href`, `type`, `name`, `placeholder`, `required`.

**Fragmento suelto (etiquetas comunes):**
```html
<h1>Título principal</h1>
<p>Un párrafo de texto.</p>
<img src="logo.png" alt="Logo">
<a href="https://ejemplo.com">Enlace</a>
<br>
<hr>
```

### Etiquetas semánticas útiles para esta actividad

| Etiqueta | Propósito |
|----------|-----------|
| `<header>` | Encabezado de la página (título, logo). |
| `<nav>` | Menú de navegación. |
| `<main>` | Contenido principal. |
| `<section>` | Sección temática. |
| `<footer>` | Pie de página. |
| `<h1>` a `<h6>` | Títulos jerárquicos (h1 es el más importante). |
| `<p>` | Párrafo. |
| `<ul>` / `<ol>` / `<li>` | Listas desordenadas / ordenadas / elementos de lista. |
| `<img>` | Imagen (auto-cerrada). |
| `<a>` | Enlace. |
| `<form>` | Formulario. |
| `<input>` | Campo de entrada (auto-cerrado). |
| `<label>` | Etiqueta descriptiva para un campo. |
| `<button>` | Botón. |
| `<div>` | Contenedor genérico (bloque). |
| `<span>` | Contenedor genérico (en línea). |

- **`<div>` vs semánticas:** `<div>` no tiene significado; úsalo solo cuando ninguna etiqueta semántica aplique. Prefiere `<header>`, `<nav>`, `<main>`, `<section>`, `<footer>` cuando tenga sentido.

---

## 4. Estructura de la carpeta `public/`

- **Propósito:** Carpeta que contiene todos los archivos estáticos del frontend (HTML, CSS, JS, imágenes).
- **Convención:** Se llama `public` porque son archivos "públicos" accesibles desde el navegador.
- **Contenido mínimo:**
  ```
  public/
  └── index.html
  ```
- **`index.html`:** Es el archivo por defecto que los servidores web cargan cuando alguien visita la raíz del sitio. Siempre debe llamarse así (o `index.htm`) para que se cargue automáticamente.

---

## 5. Construcción de la página: secciones

### A. Título

- Usa `<h1>` para el título principal (solo uno por página, por semántica).
- Puede ir dentro de `<header>`.

### B. Logo

- Usa `<img>` con `src` apuntando a la imagen y `alt` descriptivo.
- **`alt`:** Texto alternativo que se muestra si la imagen no carga (también importante para accesibilidad).
- **Ruta:** Si el logo está en `public/`, la ruta es relativa (`src="logo.png"`). Si está en subcarpeta, `src="img/logo.png"`.

### C. Menú

- Usa `<nav>` con una lista `<ul>` de enlaces `<a>`.
- Cada `<li>` contiene un `<a>`.
- Los enlaces pueden apuntar a otras páginas (que aún no existen) o a anclas dentro de la misma página (`#seccion`).

**Fragmento suelto (menú):**
```html
<nav>
    <ul>
        <li><a href="#">Inicio</a></li>
        <li><a href="#">Alumnos</a></li>
        <li><a href="#">Maestros</a></li>
    </ul>
</nav>
```

- **`href="#"`:** Enlace vacío (no navega a ningún lado). Útil como placeholder.

### D. Formulario para registrar alumno

- **Etiqueta `<form>`:** Contenedor del formulario.
  - **`action`:** URL a la que se envían los datos (en esta actividad, vacío o `#`).
  - **`method`:** `GET` o `POST` (en esta actividad, da igual, no funciona todavía).
- **Campos `<input>`:** Cada campo con su `type`, `name`, `id` y `placeholder`.
  - **`type`:** `text`, `number`, `email`, `password`, `date`, etc.
  - **`name`:** Nombre del campo (se usa al enviar los datos al backend).
  - **`id`:** Identificador único (útil para `<label for="...">` y JavaScript).
  - **`placeholder`:** Texto de ayuda que desaparece al escribir.
  - **`required`:** Campo obligatorio.
- **`<label>`:** Describe el campo. Se asocia con `for="idDelInput"`.
- **`<button type="submit">`:** Botón para enviar el formulario.

**Fragmento suelto (formulario):**
```html
<form action="#" method="post">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre" placeholder="Ingresa tu nombre" required>
    
    <label for="edad">Edad:</label>
    <input type="number" id="edad" name="edad" min="0" required>
    
    <button type="submit">Registrar</button>
</form>
```

- **`min`, `max`, `step`:** Validaciones nativas del navegador para campos numéricos.
- **`type="submit"`:** Envía el formulario. **`type="button"`:** Solo es un botón sin acción (útil con JS).

---

## 6. Comentarios en HTML

- **Sintaxis:** `<!-- comentario -->`
- **Uso:** Documentar secciones, dejar notas, comentar código temporalmente.
- **No se muestran** al usuario en la página (sí en el código fuente).

---

## 7. Buenas prácticas

- **Siempre incluye `<!DOCTYPE html>`, `<html>`, `<head>` y `<body>`.**
- **Indenta el código** para que sea legible (2 o 4 espacios).
- **Usa etiquetas semánticas** en lugar de `<div>` genéricos cuando sea posible.
- **Siempre incluye `alt` en las imágenes** (accesibilidad).
- **Asocia `<label>` con `<input>`** usando `for` e `id` (accesibilidad y usabilidad).
- **Usa `name` en los campos del formulario** (necesario para enviar datos al backend).
- **Cierra todas las etiquetas** que tengan cierre (aunque el navegador sea tolerante, es buena práctica).
- **Un solo `<h1>` por página** (jerarquía semántica).
- **Usa rutas relativas** para archivos dentro de `public/` (portabilidad).
- **Codificación UTF-8** siempre (`<meta charset="UTF-8">`).
- **Valida tu HTML** con herramientas como el validador de W3C (opcional pero recomendable).

---

## 8. Cómo visualizar la página

- **Opción 1:** Abrir el archivo `index.html` directamente en el navegador (doble clic o arrastrar).
- **Opción 2:** Usar la extensión **Live Server** de VSCode (recarga automática al guardar).
- **Opción 3:** Usar un servidor local simple (por ejemplo, `python -m http.server` en la carpeta `public/`).
- **Para esta actividad:** Cualquiera de las tres funciona. Live Server es cómodo para desarrollo.

---

## 9. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Olvidar `<!DOCTYPE html>`** | El navegador puede entrar en "quirks mode" y comportarse raro. |
| **No cerrar etiquetas** | Aunque el navegador lo tolere, puede causar problemas. Ciérralas. |
| **No incluir `alt` en imágenes** | Mala accesibilidad. Siempre incluye `alt`. |
| **Olvidar `name` en inputs** | Sin `name`, el campo no se envía al backend. |
| **`<label>` sin `for`** | El label no se asocia al input. Usa `for="id"`. |
| **Ruta de imagen incorrecta** | Verifica que la ruta sea relativa a `index.html`. |
| **Codificación incorrecta** | Sin `<meta charset="UTF-8">`, las tildes se ven mal. |
| **Abrir el archivo en el lugar equivocado** | Asegúrate de abrir `index.html`, no otro archivo. |
| **Confundir `<head>` con `<header>`** | `<head>` es metadatos (no visible); `<header>` es encabezado visible. |

---

## 10. Resumen de conceptos clave

- **HTML:** Lenguaje de marcado para estructurar contenido web.
- **Estructura básica:** `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`.
- **Etiquetas:** Unidades básicas; pueden tener atributos.
- **Semántica:** Usa `<header>`, `<nav>`, `<main>`, `<section>`, `<footer>` en lugar de `<div>` cuando aplique.
- **Formularios:** `<form>`, `<input>`, `<label>`, `<button>`. Cada input necesita `name` para enviarse al backend.
- **Carpeta `public/`:** Contiene archivos estáticos; `index.html` es el punto de entrada.
- **Sin backend:** Esta actividad es solo la interfaz; los datos no se guardan ni procesan todavía.
- **Buenas prácticas:** Indentación, `alt`, `label for`, `name`, `UTF-8`, semántica.

---
# PASO A PASO

---
# Actividad 16 — HTML: estructura y formulario

## Objetivo
Tener la **interfaz** del sistema escolar (sin estilos todavía).
Al terminar: abres `index.html` en el navegador y ves header, menú, formulario, tabla vacía y tarjetas.

## Estructura de carpetas
Crea dentro de tu proyecto:

```
public/
├── index.html
└── img/
    └── logo.png   (cualquier imagen cuadrada pequeña)
```

---

## Paso 1 — Esqueleto base

Crea `public/index.html` con esto:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistema Escolar</title>
</head>
<body>

  <!-- aquí va todo el contenido -->

</body>
</html>
```

✅ Abre el archivo con doble clic. Debe verse una página en blanco con el título "Sistema Escolar" en la pestaña.

---

## Paso 2 — Header con logo y título

Dentro de `<body>`, agrega:

```html
<header>
  <img src="img/logo.png" alt="Logo de la escuela" width="60">
  <h1>Sistema Escolar</h1>
</header>
```

⚠️ `alt` es **obligatorio** (accesibilidad). `width="60"` lo pone pequeño provisionalmente; luego lo quitará el CSS.

---

## Paso 3 — Menú de navegación

Justo debajo del header:

```html
<nav>
  <ul>
    <li><a href="#inicio">Inicio</a></li>
    <li><a href="#registro">Registrar alumno</a></li>
    <li><a href="#lista">Lista de alumnos</a></li>
    <li><a href="#resumen">Resumen</a></li>
  </ul>
</nav>
```

`href="#algo"` sirve para saltar a un `id="algo"` dentro de la misma página.

---

## Paso 4 — Layout principal (aside + main)

Envuelve el contenido de aquí en adelante así:

```html
<div class="layout">

  <aside class="sidebar">
    <h2>Panel</h2>
    <ul>
      <li><a href="#">Alumnos</a></li>
      <li><a href="#">Maestros</a></li>
      <li><a href="#">Administradores</a></li>
    </ul>
  </aside>

  <main class="contenido">

    <!-- AQUÍ van las secciones siguientes -->

  </main>

</div>
```

---

## Paso 5 — Formulario de registro

Dentro de `<main>`, agrega:

```html
<section id="registro">
  <h2>Registrar alumno</h2>

  <form id="form-alumno" action="#" method="post">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre" placeholder="Ingresa el nombre" required>

    <label for="matricula">Matrícula:</label>
    <input type="text" id="matricula" name="matricula" placeholder="Ej. A001" required>

    <label for="edad">Edad:</label>
    <input type="number" id="edad" name="edad" min="0" max="120" required>

    <label for="correo">Correo:</label>
    <input type="email" id="correo" name="correo" placeholder="alumno@escuela.com" required>

    <button type="submit">Registrar</button>
    <button type="reset" id="btn-limpiar">Limpiar</button>
  </form>
</section>
```

🔍 Fíjate en:
- `id` de cada input = `for` del label (accesibilidad).
- `name` de cada input: lo usará el backend en C++ más adelante.
- El form tiene `id="form-alumno"` — lo vamos a usar en JS.

---

## Paso 6 — Tabla de alumnos (vacía por ahora)

Debajo del formulario:

```html
<section id="lista">
  <h2>Lista de alumnos</h2>

  <table id="tabla-alumnos">
    <thead>
      <tr>
        <th>Matrícula</th>
        <th>Nombre</th>
        <th>Edad</th>
        <th>Correo</th>
        <th>Acciones</th>
      </tr>
    </thead>
    <tbody>
      <!-- se llenará con JS en la Actividad 19 -->
    </tbody>
  </table>
</section>
```

---

## Paso 7 — Tarjetas de resumen

Debajo de la tabla:

```html
<section id="resumen" class="tarjetas">
  <div class="card">
    <h3>Total alumnos</h3>
    <p id="total-alumnos">0</p>
  </div>

  <div class="card">
    <h3>Promedio general</h3>
    <p id="promedio-general">—</p>
  </div>

  <div class="card">
    <h3>Último registrado</h3>
    <p id="ultimo-alumno">—</p>
  </div>
</section>
```

---

## Paso 8 — Footer

Cierra la estructura:

```html
<footer>
  <p>&copy; 2025 Sistema Escolar — Actividad 16</p>
</footer>
```

---

## ✅ Checklist de la Actividad 16

- [ ] Existe `public/index.html`.
- [ ] Hay un solo `<h1>` en la página.
- [ ] Todos los `<input>` tienen `id` y `name`.
- [ ] Todos los `<label>` tienen `for` que coincide con un `id`.
- [ ] La imagen tiene `alt`.
- [ ] El HTML abre sin errores en el navegador.

## 🧠 Mini-reto
Agrega un input adicional `type="date"` para la "fecha de inscripción". No olvides su `<label for="...">`.

---
