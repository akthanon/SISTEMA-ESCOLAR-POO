# Actividad 17. CSS

## Temas

- Selectores
    
- Colores
    
- Layout
    

## Ejercicio

Diseñar el sistema escolar.

Agregar:

- menú lateral
    
- tabla
    
- formulario
    
- botones
    
- tarjetas
    

El objetivo es que el proyecto deje de verse como HTML sin estilo.

---
# TEORÍA

---

## 1. ¿Qué es CSS?

- **Definición:** CSS (Cascading Style Sheets) es el lenguaje que define la **presentación visual** de un documento HTML: colores, tamaños, tipografías, espaciados, posiciones, animaciones.
- **Función en el proyecto:** Transformar el HTML "pelado" en una interfaz agradable, usable y coherente.
- **Separación de responsabilidades:**
  - **HTML:** estructura y contenido.
  - **CSS:** apariencia.
  - **JavaScript:** comportamiento.
- **Ventaja:** Un mismo HTML puede verse distinto según el CSS, sin tocar la estructura.

---

## 2. Formas de incluir CSS

- **CSS externo (recomendado):** Un archivo `.css` enlazado desde el `<head>` del HTML.
  ```html
  <link rel="stylesheet" href="styles.css">
  ```
  - Ventajas: reutilizable en varias páginas, caché del navegador, HTML limpio.
- **CSS interno:** Dentro de una etiqueta `<style>` en el `<head>`. Útil para pruebas rápidas.
- **CSS en línea (inline):** Directamente en el atributo `style` de una etiqueta. Evítalo salvo casos excepcionales (emails, por ejemplo).

**Recomendación para esta actividad:** Crea `public/styles.css` y enlázalo desde `index.html`.

**Estructura de carpeta sugerida:**
```
public/
├── index.html
├── styles.css
└── img/
    └── logo.png
```

---

## 3. Sintaxis básica

- **Regla CSS:**
  ```
  selector {
      propiedad: valor;
      otra-propiedad: valor;
  }
  ```
- **Selectores:** Indican a qué elementos se aplica.
- **Propiedades:** Qué se modifica (color, tamaño, margen…).
- **Valores:** Cuánto o cómo se modifica.
- **Comentarios:** `/* comentario */`
- **Punto y coma:** Obligatorio al final de cada declaración (excepto la última, pero se recomienda ponerla siempre).

---

## 4. Selectores

### A. Selectores básicos

| Selector | Ejemplo | Aplica a… |
|----------|---------|-----------|
| Etiqueta | `p { }` | Todos los `<p>` |
| Clase | `.card { }` | Elementos con `class="card"` |
| ID | `#menu { }` | El elemento con `id="menu"` (único) |
| Universal | `* { }` | Todos los elementos (cuidado, afecta rendimiento) |
| Atributo | `input[type="text"] { }` | Inputs de tipo texto |

- **Clases:** Se pueden reutilizar en varios elementos. En HTML: `class="card destacada"` (varias clases separadas por espacio).
- **ID:** Único por página. Úsalo con moderación (prefiere clases).
- **Convención de nombres:** kebab-case (`menu-lateral`, `btn-primario`).

### B. Selectores combinados

| Combinador | Ejemplo | Significado |
|------------|---------|-------------|
| Descendiente (espacio) | `nav a { }` | Todos los `<a>` dentro de `<nav>` |
| Hijo directo (`>`) | `ul > li { }` | Solo los `<li>` hijos directos de `<ul>` |
| Hermano adyacente (`+`) | `h1 + p { }` | El `<p>` inmediatamente después de un `<h1>` |
| Hermano general (`~`) | `h1 ~ p { }` | Todos los `<p>` que siguen a un `<h1>` |

### C. Pseudo-clases y pseudo-elementos

- **Pseudo-clases:** Estados del elemento.
  - `:hover` → al pasar el mouse.
  - `:focus` → al enfocar (input activo).
  - `:active` → al hacer clic.
  - `:first-child`, `:last-child`, `:nth-child(n)`.
- **Pseudo-elementos:** Partes del elemento.
  - `::before`, `::after` → insertan contenido antes/después.
  - `::placeholder` → texto de placeholder de un input.

**Fragmentos sueltos:**
```css
a:hover { color: red; }
input:focus { border-color: blue; }
tr:nth-child(even) { background: #f2f2f2; }
```

### D. Especificidad (importante)

Orden de prioridad (de menor a mayor):
1. Selector de etiqueta (`p`).
2. Selector de clase (`.card`).
3. Selector de ID (`#menu`).
4. Estilo en línea (`style="..."`).
5. `!important` (evítalo salvo emergencia).

- Si dos reglas chocan, gana la más específica.
- Si tienen la misma especificidad, gana la **última declarada** (orden en el archivo).

---

## 5. Colores

- **Formatos:**
  - **Nombres:** `red`, `blue`, `tomato`, `steelblue`. Limitados pero cómodos.
  - **HEX:** `#RRGGBB` o `#RGB`. Ej: `#1e90ff`.
  - **RGB:** `rgb(30, 144, 255)`.
  - **RGBA:** `rgba(30, 144, 255, 0.5)` → con transparencia (0 = transparente, 1 = opaco).
  - **HSL:** `hsl(210, 100%, 56%)` → tono, saturación, luminosidad. Muy intuitivo para paletas.
- **Propiedades de color:**
  - `color` → color del texto.
  - `background-color` → color de fondo.
  - `border-color` → color del borde.
- **Paleta recomendada:** Define **variables CSS** (custom properties) para mantener coherencia.
  ```css
  :root {
      --color-primario: #1e90ff;
      --color-secundario: #333;
      --color-fondo: #f5f5f5;
  }
  ```
  Y luego las usas con `var(--color-primario)`.

---

## 6. Layout: modelo de caja (box model)

- **Todo elemento HTML es una caja** con cuatro capas:
  1. **Content** (contenido).
  2. **Padding** (relleno interno).
  3. **Border** (borde).
  4. **Margin** (margen externo).
- **`box-sizing: border-box;`** → el padding y border se incluyen en el ancho/alto declarado. **Muy recomendado** aplicarlo globalmente.
  ```css
  * { box-sizing: border-box; }
  ```

- **Propiedades clave:**
  - `width`, `height` → tamaño.
  - `padding` → espacio interno (`padding: 10px;` o `padding: 10px 20px;`).
  - `margin` → espacio externo.
  - `border` → `border: 1px solid #ccc;`
  - `border-radius` → esquinas redondeadas.
  - `overflow` → qué hacer si el contenido desborda (`hidden`, `auto`, `scroll`).

---

## 7. Layout: `display` y posicionamiento

### A. `display`

- **`block`:** Ocupa todo el ancho disponible, salta de línea (div, p, h1, section).
- **`inline`:** Ocupa solo el ancho de su contenido, no salta (span, a, strong).
- **`inline-block`:** Como inline, pero acepta width/height.
- **`none`:** No se muestra (desaparece del layout).
- **`flex`:** Activa Flexbox (ver siguiente sección).
- **`grid`:** Activa CSS Grid (ver siguiente sección).

### B. Flexbox (muy usado para menús, barras, tarjetas)

- **Contenedor flex:** `display: flex;` en el padre.
- **Dirección:** `flex-direction: row | column;`
- **Alineación horizontal:** `justify-content: flex-start | center | space-between | space-around | space-evenly;`
- **Alineación vertical:** `align-items: flex-start | center | flex-end | stretch;`
- **Espacio entre hijos:** `gap: 10px;`
- **Hijos flexibles:** `flex: 1;` (crece para ocupar espacio disponible).

**Uso típico:** Barra lateral + contenido principal, tarjetas en fila, centrado de un formulario.

### C. CSS Grid (para layouts 2D)

- **Contenedor grid:** `display: grid;`
- **Columnas:** `grid-template-columns: 250px 1fr;` (barra lateral + contenido).
- **Filas:** `grid-template-rows: auto 1fr auto;`
- **Gap:** `gap: 20px;`
- **Uso típico:** Layout general de la página (header, sidebar, main, footer).

### D. Posicionamiento

- **`position: static`** → por defecto.
- **`position: relative`** → relativo a su posición normal (útil como referencia para hijos absolutos).
- **`position: absolute`** → relativo al ancestro posicionado más cercano.
- **`position: fixed`** → fijo respecto a la ventana (útil para menús que no se mueven al hacer scroll).
- **`position: sticky`** → se comporta como relativo hasta que alcanza cierto umbral, luego se fija (útil para encabezados).

---

## 8. Tipografía y espaciado

- **Fuente:** `font-family: 'Roboto', sans-serif;` (define fallbacks).
- **Tamaño:** `font-size: 16px;` o `1rem` (relativo al root).
- **Peso:** `font-weight: bold | 400 | 700;`
- **Alineación:** `text-align: left | center | right | justify;`
- **Altura de línea:** `line-height: 1.5;`
- **Espaciado entre letras:** `letter-spacing: 1px;`
- **Google Fonts:** Puedes importar fuentes desde `<link>` en el HTML.

**Recomendación:** Usa `rem` para tamaños de fuente y espaciados coherentes. Define `html { font-size: 16px; }` como base.

---

## 9. Elementos específicos del ejercicio

### A. Menú lateral (sidebar)

- **Contenedor:** `<nav>` o `<aside>` con `display: flex; flex-direction: column;`
- **Ancho fijo:** `width: 250px;`
- **Altura completa:** `height: 100vh;` (100% de la ventana).
- **Fondo oscuro:** `background-color: #2c3e50;`
- **Enlaces:** Colores claros, padding cómodo, hover que cambie el fondo.
- **Estado activo:** Añade una clase `.activo` con un borde o fondo distinto.

### B. Tabla

- **Estructura HTML:** `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`.
- **Estilo:**
  - `border-collapse: collapse;` → bordes limpios.
  - `width: 100%;` → ocupa todo el ancho.
  - `padding` en celdas.
  - Filas alternas: `tr:nth-child(even) { background: #f9f9f9; }`
  - Hover: `tbody tr:hover { background: #eef; }`
  - Encabezado destacado: `thead { background: #1e90ff; color: white; }`

### C. Formulario

- **Contenedor:** `display: flex; flex-direction: column; gap: 15px; max-width: 500px;`
- **Campos:** Padding cómodo, borde suave, focus con borde de color.
- **Labels:** Block o arriba del input.
- **Botón:** Fondo de color primario, texto blanco, padding generoso, cursor pointer, hover más oscuro.

### D. Botones

- **Base:** padding, border-radius, sin borde por defecto, cursor pointer.
- **Variantes:** `.btn-primario`, `.btn-secundario`, `.btn-peligro` (colores distintos).
- **Estados:** `:hover` (fondo más oscuro), `:active` (un poco más pequeño o sombra interna), `:disabled` (opacidad baja, cursor no permitido).
- **Transiciones:** `transition: background 0.3s ease;` → cambios suaves.

### E. Tarjetas (cards)

- **Estructura HTML:** Un `<div class="card">` con título, contenido y quizá un botón.
- **Estilo:**
  - `background: white;`
  - `border-radius: 8px;`
  - `box-shadow: 0 2px 8px rgba(0,0,0,0.1);` → sombra suave.
  - `padding: 20px;`
  - Hover: `transform: translateY(-3px);` + sombra más marcada, con `transition`.
- **Grid de tarjetas:** `display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px;`

---

## 10. Responsive design (introducción)

- **Viewport:** Incluye `<meta name="viewport" content="width=device-width, initial-scale=1.0">` en el `<head>`.
- **Unidades relativas:** `%`, `rem`, `em`, `vw`, `vh` en lugar de píxeles fijos.
- **Media queries:**
  ```css
  @media (max-width: 768px) {
      .sidebar { width: 100%; height: auto; }
  }
  ```
- **Mobile-first:** Diseña primero para móvil y luego amplía. Aunque para esta actividad no es obligatorio, tenlo en cuenta.

---

## 11. Buenas prácticas

- **Un archivo CSS externo** (`styles.css`) enlazado desde el HTML.
- **Organiza el CSS por secciones** con comentarios: reset, variables, layout, componentes, utilidades.
- **Usa variables CSS** (`:root`) para colores, tipografías y espaciados recurrentes.
- **Consistencia:** mismos paddings, mismos radios, misma paleta.
- **Evita `!important`** salvo casos extremos.
- **Evita selectores demasiado específicos** (dificultan el mantenimiento).
- **Nombra clases con kebab-case** y significado semántico (`card-alumno`, no `.c1`).
- **Reutiliza clases** en lugar de duplicar reglas.
- **`box-sizing: border-box` global** al inicio.
- **Reset o normalize:** Aplica un reset mínimo (margins y paddings a cero) para evitar diferencias entre navegadores.
- **Comenta secciones** para que el CSS sea navegable.
- **Verifica en varios navegadores** (Chrome, Firefox, Edge).

---

## 12. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **No enlazar el CSS** | Verifica la ruta en `<link rel="stylesheet" href="...">`. |
| **Selector mal escrito** | Revisa mayúsculas y puntos: `.card` ≠ `.Card`. |
| **Olvidar `;`** | Cada declaración termina con `;`. |
| **Estilos que no se aplican** | Puede ser especificidad o caché. Recarga con Ctrl+F5. |
| **Elemento no se centra** | Usa Flexbox: `display:flex; justify-content:center; align-items:center;`. |
| **Layout roto al reducir la ventana** | Usa unidades relativas y media queries. |
| **Tabla con bordes dobles** | Aplica `border-collapse: collapse;`. |
| **Formulario con campos desalineados** | Usa `display: flex; flex-direction: column; gap: ...;`. |
| **Botón sin efecto hover** | Recuerda `cursor: pointer` y `transition`. |
| **Tarjetas sin sombra o planas** | Añade `box-shadow` y `border-radius`. |
| **Margen que colapsa** | Usa `padding` en el padre o `display: flex` para evitar el colapso de márgenes. |

---

## 13. Resumen de conceptos clave

- **CSS:** Lenguaje de presentación para HTML.
- **Inclusión:** Externo (recomendado), interno o en línea.
- **Selectores:** etiqueta, clase, id, atributo, combinadores, pseudo-clases.
- **Especificidad:** id > clase > etiqueta; `!important` como último recurso.
- **Colores:** nombres, HEX, RGB, RGBA, HSL; variables CSS para coherencia.
- **Box model:** content + padding + border + margin; `box-sizing: border-box`.
- **Layout:** `display: flex` para filas y columnas; `display: grid` para layouts 2D.
- **Componentes del ejercicio:**
  - **Sidebar:** `<nav>` vertical, ancho fijo, fondo oscuro.
  - **Tabla:** `border-collapse`, filas alternas, hover.
  - **Formulario:** Flex column, gap, focus visible.
  - **Botones:** variantes, hover, transición, cursor pointer.
  - **Tarjetas:** fondo blanco, sombra, border-radius, hover elevado.
- **Responsive:** viewport, unidades relativas, media queries.
- **Buenas prácticas:** organizar, reutilizar, comentar, consistencia visual.

---
# PASO A PASO

---
# Actividad 17 — CSS: layout, colores y componentes

## Objetivo
Convertir la página "pelada" del paso anterior en una interfaz usable:
sidebar lateral, tabla legible, formulario ordenado, botones y tarjetas.

## Archivo nuevo
Crea `public/styles.css`.

En `index.html`, dentro de `<head>`, agrega:

```html
<link rel="stylesheet" href="styles.css">
```

⚠️ Verifica mayúsculas y ruta: `styles.css` ≠ `Styles.css`.

---

## Paso 1 — Reset y variables

Pega al inicio de `styles.css`:

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --color-primario: #2563eb;
  --color-primario-oscuro: #1e40af;
  --color-peligro: #dc2626;
  --color-fondo: #f3f4f6;
  --color-sidebar: #1f2937;
  --color-texto: #111827;
  --color-borde: #d1d5db;
  --radio: 8px;
  --sombra: 0 2px 8px rgba(0, 0, 0, 0.08);
}

html {
  font-size: 16px;
}

body {
  font-family: system-ui, -apple-system, sans-serif;
  background: var(--color-fondo);
  color: var(--color-texto);
  line-height: 1.5;
}
```

✅ Recarga. Deben desaparecer los márgenes por defecto.

---

## Paso 2 — Header

```css
header {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem 2rem;
  background: white;
  border-bottom: 1px solid var(--color-borde);
}

header img {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  object-fit: cover;
}

header h1 {
  font-size: 1.5rem;
  color: var(--color-primario);
}
```

---

## Paso 3 — Nav horizontal

```css
nav ul {
  display: flex;
  gap: 1rem;
  list-style: none;
  padding: 0.75rem 2rem;
  background: white;
  border-bottom: 1px solid var(--color-borde);
}

nav a {
  text-decoration: none;
  color: var(--color-texto);
  padding: 0.4rem 0.8rem;
  border-radius: var(--radio);
  transition: background 0.2s ease;
}

nav a:hover {
  background: var(--color-primario);
  color: white;
}
```

---

## Paso 4 — Layout con CSS Grid (sidebar + contenido)

```css
.layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 1.5rem;
  padding: 1.5rem 2rem;
  min-height: 70vh;
}
```

---

## Paso 5 — Sidebar

```css
.sidebar {
  background: var(--color-sidebar);
  color: white;
  padding: 1.5rem 1rem;
  border-radius: var(--radio);
  height: fit-content;
}

.sidebar h2 {
  font-size: 1.1rem;
  margin-bottom: 1rem;
  border-bottom: 1px solid rgba(255,255,255,0.15);
  padding-bottom: 0.5rem;
}

.sidebar ul {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.sidebar a {
  color: #e5e7eb;
  text-decoration: none;
  display: block;
  padding: 0.5rem 0.75rem;
  border-radius: var(--radio);
  transition: background 0.2s ease;
}

.sidebar a:hover {
  background: rgba(255,255,255,0.1);
  color: white;
}
```

---

## Paso 6 — Secciones y títulos

```css
.contenido section {
  background: white;
  padding: 1.5rem;
  border-radius: var(--radio);
  box-shadow: var(--sombra);
  margin-bottom: 1.5rem;
}

.contenido section h2 {
  font-size: 1.25rem;
  margin-bottom: 1rem;
  color: var(--color-primario);
}
```

---

## Paso 7 — Formulario

```css
#form-alumno {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-width: 480px;
}

#form-alumno label {
  font-weight: 600;
  font-size: 0.9rem;
}

#form-alumno input {
  padding: 0.6rem 0.8rem;
  border: 1px solid var(--color-borde);
  border-radius: var(--radio);
  font-size: 1rem;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

#form-alumno input:focus {
  outline: none;
  border-color: var(--color-primario);
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);
}
```

---

## Paso 8 — Botones

```css
button {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: var(--radio);
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.05s ease;
}

button[type="submit"] {
  background: var(--color-primario);
  color: white;
}

button[type="submit"]:hover {
  background: var(--color-primario-oscuro);
}

button[type="reset"] {
  background: var(--color-borde);
  color: var(--color-texto);
}

button[type="reset"]:hover {
  background: #9ca3af;
}

button:active {
  transform: scale(0.98);
}
```

---

## Paso 9 — Tabla

```css
table {
  width: 100%;
  border-collapse: collapse;
  overflow: hidden;
  border-radius: var(--radio);
}

thead {
  background: var(--color-primario);
  color: white;
}

th, td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid var(--color-borde);
}

tbody tr:nth-child(even) {
  background: #f9fafb;
}

tbody tr:hover {
  background: #eff6ff;
}
```

---

## Paso 10 — Tarjetas

```css
.tarjetas {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
  background: transparent !important;
  box-shadow: none !important;
  padding: 0 !important;
}

.card {
  background: white;
  padding: 1.25rem;
  border-radius: var(--radio);
  box-shadow: var(--sombra);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0,0,0,0.12);
}

.card h3 {
  font-size: 0.85rem;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.5rem;
}

.card p {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-primario);
}
```

---

## Paso 11 — Footer

```css
footer {
  text-align: center;
  padding: 1.5rem;
  color: #6b7280;
  font-size: 0.85rem;
}
```

---

## Paso 12 — Responsive básico

```css
@media (max-width: 768px) {
  .layout {
    grid-template-columns: 1fr;
    padding: 1rem;
  }

  nav ul {
    flex-wrap: wrap;
    padding: 0.5rem 1rem;
  }

  header {
    padding: 1rem;
  }
}
```

---

## ✅ Checklist de la Actividad 17

- [ ] El CSS está en archivo externo y enlazado.
- [ ] No hay colores sueltos: todo usa variables `var(--...)`.
- [ ] El sidebar se ve oscuro a la izquierda.
- [ ] La tabla tiene encabezado azul y filas alternas.
- [ ] Los botones cambian al pasar el mouse.
- [ ] Al reducir la ventana (<768px), el sidebar se va arriba.

## 🧠 Mini-reto
Agrega `.btn-peligro` con fondo rojo y úsala en un botón de "Cancelar" dentro del formulario.