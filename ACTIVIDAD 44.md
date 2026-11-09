# Actividad 44. Control de versiones con GitHub

## Temas

- Git
    
- Trabajo colaborativo
    

## Ejercicio

Subir el proyecto a GitHub.

Crear:

```text
main
develop
feature/*
```

Realizar flujo de trabajo:

1. Crear una rama para una nueva función.
    
2. Realizar cambios.
    
3. Crear commit.
    
4. Abrir Pull Request.
    
5. Revisar cambios.
    
6. Fusionar a la rama principal.
    

Cada equipo deberá mantener historial del desarrollo.

Ejemplo:

```text
feat: agregar autenticación

fix: corregir lectura de archivos

refactor: separar clase servidor
```

---
# TEORÍA

---

## 1. ¿Qué es el control de versiones?

- **Definición:** Sistema que registra los cambios realizados sobre un conjunto de archivos a lo largo del tiempo, permitiendo recuperar versiones anteriores, trabajar en paralelo y colaborar sin pisarse.
- **Problema que resuelve:**
  - "Se me borró el código y no tengo copia."
  - "Trabajé con un compañero y sobrescribimos cambios."
  - "No sé qué cambió entre la versión que funciona y la que falla."
- **Ventajas:**
  - Historial completo de cambios.
  - Posibilidad de revertir a versiones anteriores.
  - Trabajo en paralelo con ramas.
  - Colaboración sin conflictos.
  - Auditoría de quién cambió qué y cuándo.

---

## 2. Git vs. GitHub

- **Git:** Sistema de control de versiones **distribuido**. Se ejecuta localmente. Cada desarrollador tiene una copia completa del repositorio con su historial.
- **GitHub:** Plataforma en la nube que hospeda repositorios Git. Añade funcionalidades sociales y de colaboración: Pull Requests, Issues, Actions, Wikis.
- **Diferencia clave:**
  - **Git** es la herramienta.
  - **GitHub** es el servicio donde se aloja el repositorio remoto.
- **Alternativas a GitHub:** GitLab, Bitbucket, Gitea. Los conceptos son idénticos; cambia la interfaz.

---

## 3. Conceptos fundamentales de Git

### A. Repositorio
- Carpeta gestionada por Git. Contiene el proyecto y un subdirectorio oculto `.git/` con todo el historial.
- **Local:** en tu máquina.
- **Remoto:** en GitHub u otro servidor.

### B. Commit
- **Definición:** Fotografía del estado del proyecto en un momento dado.
- **Contiene:**
  - Los cambios realizados.
  - Un mensaje descriptivo.
  - Autor, fecha, hash único.
- **Hash:** Identificador único del commit (por ejemplo, `a1b2c3d`). Imposible de modificar sin alterar el historial.

### C. Rama (branch)
- **Definición:** Línea de desarrollo independiente. Permite trabajar en algo sin afectar a la línea principal.
- **Rama por defecto:** `main` (antes `master`).
- **Uso:** Una rama por feature, fix o experimento.

### D. HEAD
- Puntero al commit actual en el que estás posicionado.
- Normalmente apunta a la última confirmación de tu rama actual.

### E. Working directory, staging area, repository
- **Working directory:** Tu carpeta de trabajo con los archivos tal cual están.
- **Staging area (índice):** Zona intermedia donde preparas los cambios antes de confirmarlos.
- **Repository:** Historial de commits confirmados.
- **Flujo:** working → `git add` → staging → `git commit` → repository.

### F. Remoto (remote)
- Repositorio en un servidor (GitHub). Se le da un nombre, normalmente `origin`.
- Operaciones: `push` (enviar cambios), `pull` (traer cambios), `fetch` (descargar sin fusionar).

---

## 4. Estados de un archivo en Git

| Estado | Descripción |
|--------|-------------|
| **Untracked** | Archivo nuevo que Git no conoce aún. |
| **Modified** | Archivo rastreado que ha cambiado desde el último commit. |
| **Staged** | Cambio preparado para el próximo commit. |
| **Committed** | Cambio guardado en el historial. |
| **Ignored** | Archivo excluido por `.gitignore`. |

- **`.gitignore`:** Archivo que lista patrones de archivos que Git debe ignorar (binarios, logs, archivos temporales, `.env` con secretos, carpetas de compilación).

**Fragmento suelto (idea de `.gitignore`):**
```
*.o
*.exe
build/
data/*.db
.env
```

---

## 5. Flujo básico de Git

1. **Configurar identidad** (una sola vez):
   - `git config --global user.name "Tu Nombre"`
   - `git config --global user.email "tu@correo.com"`
2. **Inicializar repositorio** en la carpeta del proyecto: `git init`.
3. **Ver estado** de los archivos: `git status`.
4. **Añadir archivos** al staging: `git add <archivo>` o `git add .` para todos.
5. **Confirmar cambios:** `git commit -m "mensaje descriptivo"`.
6. **Conectar con remoto:** `git remote add origin <url>`.
7. **Subir cambios:** `git push -u origin main` (la primera vez), luego `git push`.
8. **Traer cambios:** `git pull` (descarga y fusiona).
9. **Clonar un repositorio existente:** `git clone <url>`.

---

## 6. Ramas (branches)

### A. Concepto
- Permiten trabajar en paralelo sin afectar a la rama principal.
- Cada rama es una línea de desarrollo independiente.
- Al terminar, se fusiona (merge) con la rama destino.

### B. Operaciones básicas
- **Crear rama:** `git branch <nombre>`.
- **Cambiar de rama:** `git checkout <nombre>` o `git switch <nombre>`.
- **Crear y cambiar:** `git checkout -b <nombre>` o `git switch -c <nombre>`.
- **Listar ramas:** `git branch` (locales), `git branch -a` (incluye remotas).
- **Fusionar:** `git merge <rama>` (desde la rama destino).
- **Eliminar rama local:** `git branch -d <nombre>`.
- **Eliminar rama remota:** `git push origin --delete <nombre>`.

### C. Convenciones de nombres
- `main` o `master`: rama principal, estable.
- `develop`: rama de integración, donde se juntan las features antes de pasar a main.
- `feature/<nombre>`: nuevas funcionalidades.
- `fix/<nombre>` o `bugfix/<nombre>`: correcciones.
- `hotfix/<nombre>`: correcciones urgentes en producción.
- `refactor/<nombre>`: reorganización de código.
- `docs/<nombre>`: documentación.

**Fragmento suelto (idea):**
```
feature/login
feature/gestion-archivos
fix/lectura-usuarios
refactor/servidor-clases
```

- **Ventaja:** Los nombres describen la intención. Se sabe qué hace cada rama sin abrirla.

---

## 7. Modelo de ramas (Git Flow simplificado)

Un modelo de ramas común en proyectos colaborativos:

- **`main`:** Solo contiene código estable y probado. Cada commit aquí es una versión entregable.
- **`develop`:** Rama de integración. Aquí se fusionan las features terminadas.
- **`feature/*`:** Ramas temporales para cada nueva funcionalidad. Nacen de `develop` y mueren en `develop`.
- **`fix/*`:** Ramas para correcciones. Pueden nacer de `develop` o `main` según la gravedad.
- **`hotfix/*`:** Correcciones urgentes en producción. Nacen de `main` y se fusionan tanto en `main` como en `develop`.

### Flujo típico
1. Actualizas `develop`: `git checkout develop && git pull`.
2. Creas tu rama: `git checkout -b feature/nueva-funcion`.
3. Trabajas, commiteas, pusheas la rama.
4. Abres un Pull Request de `feature/nueva-funcion` hacia `develop`.
5. Revisión de código por otro miembro del equipo.
6. Se aprueba y se fusiona.
7. Periódicamente, `develop` se fusiona a `main` cuando hay una versión estable.

### Variantes
- **Trunk-based development:** Todo se trabaja sobre `main` con ramas muy cortas. Requiere mucha disciplina y buenas pruebas automatizadas.
- **GitHub Flow:** Solo `main` + ramas cortas por feature. Más simple que Git Flow.
- **Elección:** Para proyectos académicos o pequeños, GitHub Flow es suficiente. Para proyectos grandes, Git Flow.

---

## 8. Commits: mensajes y buenas prácticas

### A. Conventional Commits
- **Convención:** `<tipo>(<alcance>): <descripción>`.
- **Tipos comunes:**
  - `feat`: nueva funcionalidad.
  - `fix`: corrección de bug.
  - `refactor`: reorganización sin cambiar comportamiento.
  - `docs`: cambios en documentación.
  - `style`: formato, espacios, sin cambios funcionales.
  - `test`: añadir o modificar pruebas.
  - `chore`: tareas de mantenimiento (dependencias, configuración).
  - `perf`: mejoras de rendimiento.
  - `build`: cambios en el sistema de build.
  - `ci`: cambios en integración continua.

**Fragmento suelto (ejemplos de mensajes):**
```
feat: agregar autenticación con sesiones
fix: corregir lectura de archivos con tildes
refactor: separar clase Servidor en módulos
docs: actualizar README con endpoints
chore: añadir .gitignore para binarios
```

### B. Reglas para buenos mensajes
- **Imperativo presente:** "agregar", no "agregado" ni "agrega".
- **Primera línea ≤ 50 caracteres.**
- **Descripción completa si hace falta:** deja una línea en blanco y explica el "por qué" en el cuerpo.
- **Un commit, un cambio lógico.** No mezcles varias cosas no relacionadas.
- **Evita mensajes genéricos:** "cambios", "update", "fix", "WIP".

**Fragmento suelto (commit con cuerpo):**
```
feat: agregar autenticación con sesiones

Se implementa login con cookies HttpOnly. La sesión
se guarda en memoria del servidor. Se añade ruta
/api/logout para invalidar la sesión.
```

- **Ventaja:** El historial es autoexplicativo. Se puede entender qué pasó sin abrir cada commit.

---

## 9. Pull Requests (PR)

### A. ¿Qué es?
- **Definición:** Propuesta de fusionar una rama (por ejemplo, `feature/login`) en otra (por ejemplo, `develop`).
- **Plataforma:** Se hace en GitHub, no en Git.
- **Función:** Revisión de código antes de fusionar.

### B. Flujo de un Pull Request
1. Pushear la rama al remoto: `git push -u origin feature/login`.
2. En GitHub, ir al repositorio y pulsar "New Pull Request".
3. Seleccionar la rama origen (feature) y la destino (develop).
4. Escribir un título y descripción:
   - Qué hace el cambio.
   - Por qué es necesario.
   - Cómo probarlo.
   - Capturas si aplica.
5. Asignar revisores.
6. Revisores leen el código, comentan, piden cambios.
7. Se hacen los ajustes en la rama (más commits).
8. Se aprueba el PR.
9. Se fusiona (merge) a la rama destino.
10. Se elimina la rama de feature si ya no se necesita.

### C. Revisión de código
- **Qué revisar:**
  - ¿El código hace lo que dice?
  - ¿Sigue las convenciones del proyecto?
  - ¿Tiene duplicación?
  - ¿Está bien nombrado?
  - ¿Tiene manejo de errores?
  - ¿Tiene validaciones?
  - ¿Hay tests?
- **Cómo revisar:**
  - Comentarios constructivos, no ataques.
  - Sugerir mejoras, no imponer.
  - Aprobar cuando esté listo. No bloquear por detalles menores.

### D. Ventajas
- Detecta bugs antes de llegar a main.
- Comparte conocimiento entre el equipo.
- Mantiene un estándar de calidad.
- Documenta qué cambió y por qué.

---

## 10. Conflictos de merge

- **Definición:** Ocurre cuando dos ramas modifican la **misma línea** del mismo archivo de formas distintas.
- **Síntomas:** `git merge` avisa de conflictos. El archivo contiene marcas de conflicto.
- **Marcas de conflicto:**
  - `<<<<<<< HEAD` → versión de tu rama.
  - `=======` → separador.
  - `>>>>>>> rama` → versión de la otra rama.
- **Resolución:**
  1. Abrir el archivo afectado.
  2. Decidir qué versión conservar (o combinar ambas).
  3. Eliminar las marcas.
  4. `git add` del archivo resuelto.
  5. `git commit` para completar el merge.
- **Prevención:**
  - Ramas cortas.
  - Pull frecuente de la rama base.
  - Commits pequeños.
  - Comunicación con el equipo.

---

## 11. Sincronización con el remoto

- **`git fetch`:** Descarga los cambios del remoto sin fusionarlos. Solo actualiza las referencias.
- **`git pull`:** `fetch` + `merge`. Descarga y fusiona en tu rama actual.
- **`git push`:** Sube tus commits al remoto.
- **`git push --force`:** Sobrescribe el historial remoto. **Peligroso.** Solo si sabes lo que haces y estás seguro de que nadie más está trabajando en esa rama.
- **`git push --force-with-lease`:** Variante más segura. Falla si el remoto cambió desde tu último fetch.
- **Antes de hacer push:** Siempre `pull` para evitar rechazos y conflictos.

---

## 12. Comandos útiles adicionales

- **`git log`:** Ver el historial de commits.
  - `git log --oneline` → versión compacta.
  - `git log --graph --oneline --all` → ver ramas visualmente.
- **`git diff`:** Ver cambios no confirmados.
  - `git diff` → working vs. staged.
  - `git diff --staged` → staged vs. último commit.
- **`git show <commit>`:** Ver detalle de un commit.
- **`git blame <archivo>`:** Ver quién escribió cada línea.
- **`git stash`:** Guardar cambios sin confirmar para cambiar de rama.
  - `git stash` → guardar.
  - `git stash pop` → recuperar.
- **`git reset`:** Deshacer cambios.
  - `git reset --soft HEAD~1` → deshacer último commit, mantiene cambios en staging.
  - `git reset --mixed HEAD~1` → deshacer último commit, mantiene cambios en working.
  - `git reset --hard HEAD~1` → deshacer último commit y **borrar cambios**. **Peligroso.**
- **`git revert <commit>`:** Crea un commit nuevo que deshace los cambios de otro. Más seguro que `reset` en ramas compartidas.
- **`git cherry-pick <commit>`:** Aplicar un commit específico de otra rama.

---

## 13. GitHub: funcionalidades adicionales

### A. Issues
- Sistema de tickets para reportar bugs, sugerir features, discutir ideas.
- Se pueden etiquetar (`bug`, `enhancement`, `documentation`).
- Se asignan a personas.
- Se cierran con commits (`fix: ... (#12)`).

### B. Projects
- Tableros tipo Kanban para organizar el trabajo.
- Columnas: To Do, In Progress, Done.

### C. Actions (CI/CD)
- Automatización: ejecutar tests, compilar, desplegar.
- Se configura con archivos YAML en `.github/workflows/`.
- **Para esta actividad:** No es obligatorio, pero es un plus.

### D. Wiki
- Documentación del proyecto separada del código.

### E. Releases
- Versiones etiquetadas del proyecto.
- Se asocian a tags de Git (`v1.0.0`, `v1.1.0`).

### F. Fork
- Copia de un repositorio ajeno en tu cuenta.
- Se usa para contribuir a proyectos open source sin tener permisos de escritura directos.
- Flujo: fork → clonar → rama → commits → push → Pull Request al repo original.

---

## 14. Trabajo colaborativo

### A. Reglas de convivencia
- **Nunca** hagas push directo a `main`. Siempre por PR.
- **Siempre** pull antes de empezar a trabajar.
- **Siempre** crea una rama para cada feature.
- **Commits pequeños y frecuentes.**
- **Mensajes claros.**
- **Revisa PRs de tus compañeros.**
- **Comunica** si vas a tocar zonas conflictivas.
- **No** reescribas historia compartida (`push --force` en ramas compartidas).

### B. División del trabajo
- Cada quien en su rama.
- Zonas distintas del código para evitar conflictos.
- Integración frecuente en `develop` para detectar problemas pronto.

### C. Revisión entre pares
- Todo PR necesita al menos un revisor.
- El revisor aprueba, comenta o pide cambios.
- El autor responde a los comentarios.
- Se fusiona cuando hay acuerdo.

---

## 15. Estructura de un repositorio

```
Proyecto/
├── .git/                  ← gestionado por Git, no se toca
├── .gitignore             ← archivos a ignorar
├── README.md              ← documentación principal
├── LICENSE                ← licencia (opcional)
├── CHANGELOG.md           ← historial de versiones (opcional)
├── src/                   ← código fuente
├── include/               ← cabeceras
├── public/                ← frontend
├── data/                  ← datos (excepto .gitignore)
├── tests/                 ← pruebas
├── docs/                  ← documentación adicional
└── Makefile / CMakeLists.txt
```

- **`.gitignore`:** crucial. Debe excluir binarios, archivos generados, datos sensibles, configuraciones locales.
- **README.md:** la primera impresión del proyecto. Debe explicar qué es, cómo compilar y cómo ejecutar.
- **LICENSE:** define los permisos de uso. Si es open source, MIT o GPL. Si es académico, quizá no haga falta.
- **`data/`:** normalmente se ignora el contenido generado, pero se pueden incluir archivos de ejemplo.

---

## 16. Buenas prácticas

- **Commits atómicos.** Un cambio lógico por commit.
- **Mensajes en imperativo y descriptivos.** Convención `tipo: descripción`.
- **Ramas cortas.** Cuanto más larga, más difícil el merge.
- **Pull frecuente.** Evita conflictos grandes.
- **Push frecuente.** No acumules trabajo local.
- **Nunca** commits con contraseñas, tokens ni claves. Usa variables de entorno.
- **`.gitignore` desde el primer commit.** Incluye binarios, datos generados, `.env`.
- **README desde el primer commit.** Explica el proyecto.
- **Pull Requests pequeños.** Fáciles de revisar.
- **Revisión de código.** Siempre por PR, aunque trabajes solo (te ayuda a ver el diff).
- **Etiqueta versiones** (`v1.0.0`, `v1.1.0`) con `git tag`.
- **Mantén `main` siempre estable.** Nunca rompas la rama principal.
- **Documenta el flujo de trabajo** en el README o CONTRIBUTING.md.
- **No mezcles cambios no relacionados** en un mismo commit o PR.
- **Borra las ramas** después de fusionar. Evita acumulación.
- **Configura tu identidad** en Git antes de empezar.
- **Aprende a resolver conflictos.** Ocurrirán tarde o temprano.
- **Usa `git log`** para entender la historia del proyecto.
- **No uses `git push --force`** en ramas compartidas.
- **Revisa `git status`** antes de cada commit.
- **Haz backups.** GitHub es un backup, pero no el único.

---

## 17. Errores comunes y soluciones

| Error | Solución |
|-------|----------|
| **Commit con contraseña o token** | Elimínalo con `git filter-repo` o `BFG`. Cambia la contraseña. Prevenlo con `.gitignore`. |
| **Push rechazado** | El remoto tiene cambios que no tienes. Haz `git pull` y resuelve. |
| **Conflicto de merge** | Edita el archivo, elimina marcas, `git add`, `git commit`. |
| **Commit en la rama equivocada** | `git cherry-pick` para moverlo, `git reset` para deshacerlo, o crear la rama correcta con el commit. |
| **Olvidé crear una rama antes de trabajar** | `git stash` los cambios, crear rama, `git stash pop`. |
| **Hice push --force y perdí cambios ajenos** | Recupera del reflog si es posible. Evita `--force` en ramas compartidas. |
| **Trabajé directamente en main** | Crea una rama a partir de tu estado actual, mueve main atrás, sigue en la rama. |
| **Historial con commits "WIP" o "fix"** | Rebase interactivo (`git rebase -i`) para limpiar antes del push. |
| **No sé qué cambió** | `git log --oneline`, `git diff`, `git show <commit>`. |
| **Eliminé una rama por error** | `git reflog` para localizar el commit y recuperarla. |
| **Archivo enorme subido a Git** | Elimínalo del historial con `git filter-repo` o `BFG`. Añádelo a `.gitignore`. |
| **`git pull` falla con "unrelated histories"** | `git pull --allow-unrelated-histories`. O verifica que estás en el repo correcto. |
| **Trabajo perdido tras reset --hard** | `git reflog`, encuentra el commit anterior, `git reset --hard <hash>`. |
| **No puedo hacer push a `main`** | Está protegida. Crea un PR desde tu rama. |
| **Commit con archivo ignorado** | Ya estaba rastreado. `git rm --cached <archivo>`. |
| **Conflicto recurrente en el mismo archivo** | Coordina con quien lo edita. Fusiona frecuentemente. |
| **Mensajes de commit incoherentes** | Establece convención (`Conventional Commits`). Revísalo en el PR. |
| **Repositorio clonado con submódulos** | `git submodule update --init --recursive`. |
| **Cambiar la URL del remoto** | `git remote set-url origin <nueva-url>`. |
| **Ver el remoto actual** | `git remote -v`. |

---

## 18. Flujo completo para esta actividad

1. **Verificar Git instalado:** `git --version`.
2. **Configurar identidad:** `git config --global user.name` y `user.email`.
3. **Inicializar el repositorio:** `git init` en la raíz del proyecto.
4. **Crear `.gitignore`** con binarios, `data/*.db`, `.env`, `build/`.
5. **Crear `README.md`** con la descripción del proyecto.
6. **Primer commit:** `git add .` + `git commit -m "chore: inicializar repositorio"`.
7. **Crear repositorio en GitHub** (vacío, sin README ni .gitignore).
8. **Conectar remoto:** `git remote add origin <url>`.
9. **Subir `main`:** `git push -u origin main`.
10. **Crear rama `develop`:** `git checkout -b develop` + `git push -u origin develop`.
11. **Para cada feature:**
    - `git checkout develop`
    - `git pull`
    - `git checkout -b feature/nombre`
    - Trabajar, commitear.
    - `git push -u origin feature/nombre`
    - Abrir PR en GitHub hacia `develop`.
    - Revisar, aprobar, fusionar.
    - `git checkout develop` + `git pull`.
    - Borrar la rama local y remota.
12. **Periódicamente:** PR de `develop` hacia `main` cuando haya versión estable.

---

## 19. Verificación

1. **El repositorio está en GitHub** y es accesible.
2. **`git log --oneline --graph`** muestra un historial limpio.
3. **`main` está protegida.** No se aceptan pushes directos; solo PRs.
4. **`develop` existe** y contiene las features integradas.
5. **Cada feature tiene su rama** con commits descriptivos.
6. **Los PRs** tienen descripción, revisores y discusión.
7. **El README** explica el proyecto y cómo usarlo.
8. **`.gitignore`** excluye binarios y datos sensibles.
9. **No hay commits con contraseñas** ni información sensible.
10. **El equipo** puede clonar el repositorio y empezar a trabajar.

---

## 20. Resumen de conceptos clave

- **Git:** sistema de control de versiones distribuido. Local.
- **GitHub:** plataforma para hospedar repositorios Git. Remoto.
- **Commit:** fotografía del estado del proyecto. Hash único.
- **Rama:** línea de desarrollo independiente. `main`, `develop`, `feature/*`.
- **Staging:** zona intermedia entre working y commit.
- **Remoto:** repositorio en el servidor (`origin`).
- **Push/pull/fetch:** sincronización con el remoto.
- **Conventional Commits:** `tipo: descripción` (`feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`).
- **Pull Request:** propuesta de fusionar una rama. Revisión de código.
- **Conflicto:** dos ramas modifican lo mismo. Se resuelve manualmente.
- **`.gitignore`:** archivos que Git ignora.
- **Flujo:** `main` estable, `develop` integración, `feature/*` por cada cambio.
- **Buenas prácticas:** commits atómicos, ramas cortas, pull frecuente, revisión por pares.
- **Errores comunes:** push --force, commits con secretos, trabajar en main.
- **Verificación:** historial limpio, PRs revisados, README, .gitignore, ramas organizadas.

---
# PASO A PASO

---
# Actividad 44 — Control de versiones con GitHub

## Objetivo
Subir el proyecto a GitHub. Organizar el trabajo con ramas `main`, `develop` y `feature/*`. Usar Pull Requests para integrar cambios.

## Modelo de ramas

```
main       ← solo código estable, releases
  │
  └── develop       ← integración de features
        │
        ├── feature/login
        ├── feature/panel-admin
        ├── feature/archivos
        └── fix/parseo-cookies
```

---

## Paso 1 — Inicializar repositorio local

```bash
git init
git config user.name "Tu Nombre"
git config user.email "tu@correo.com"
```

---

## Paso 2 — Crear `.gitignore`

```gitignore
# Compilados
build/
*.o
*.exe
servidor
servidor.exe

# Base de datos y archivos subidos
data/sistema.db
data/uploads/

# Configuración local
.env
*.local

# IDs de editor
.vscode/
.idea/
*.swp

# Sistema
.DS_Store
Thumbs.db
```

⚠️ **No** ignores `data/` completo si quieres conservar los archivos de ejemplo. Solo ignora los generados.

---

## Paso 3 — Primer commit

```bash
git add .
git status                    # verificar qué se va a commitear
git commit -m "chore: inicializar repositorio con estructura base"
```

---

## Paso 4 — Crear repositorio en GitHub

1. Ve a <https://github.com/new>.
2. Nombre: `sistema-escolar`.
3. **No** marques "Initialize with README" ni ".gitignore" (ya los tienes).
4. Público o privado.
5. Crear.

Copia la URL del repositorio.

---

## Paso 5 — Conectar remoto y subir

```bash
git remote add origin https://github.com/tu-usuario/sistema-escolar.git
git branch -M main
git push -u origin main
```

Verifica en GitHub que aparecen tus archivos.

---

## Paso 6 — Crear rama `develop`

```bash
git checkout -b develop
git push -u origin develop
```

---

## Paso 7 — Flujo por cada feature

Vamos a crear la rama para una nueva funcionalidad.

### A. Crear la rama

```bash
git checkout develop
git pull                                  # asegurar que está al día
git checkout -b feature/login-mejorado
```

### B. Trabajar y commitear

Haz cambios en el código. Commit atómico por cada avance lógico:

```bash
git add public/login.html
git commit -m "feat: agregar validación de campos vacíos en login"

git add src/Servidor.cpp
git commit -m "fix: corregir comparación de contraseña con trim"

git add public/js/login.js
git commit -m "feat: mostrar mensaje de error específico por código HTTP"
```

### C. Subir la rama

```bash
git push -u origin feature/login-mejorado
```

### D. Abrir Pull Request

1. Ve a GitHub → tu repositorio.
2. Aparecerá un banner "Compare & pull request". Haz clic.
3. **Base:** `develop`. **Compare:** `feature/login-mejorado`.
4. Título claro y descripción:
   ```markdown
   ## Qué hace este PR
   - Agrega validación cliente al formulario de login.
   - Corrige un bug donde la contraseña con espacios al final fallaba.
   - Muestra mensajes de error específicos por código HTTP.

   ## Cómo probarlo
   1. Abrir `/login.html`.
   2. Probar con campos vacíos → mensaje de validación.
   3. Probar con contraseña incorrecta → 401.
   4. Probar con credenciales correctas → redirige.

   ## Checklist
   - [x] Compila sin warnings.
   - [x] Probado manualmente.
   - [x] README actualizado si aplica.
   ```
5. Asignar revisores (compañeros).
6. Crear PR.

### E. Revisión

El revisor:
- Lee el diff.
- Comenta sugerencias.
- Aprueba o pide cambios.

Si pide cambios:

```bash
# Hacer los ajustes en la misma rama
git add .
git commit -m "fix: atender comentarios del code review"
git push
```

El PR se actualiza automáticamente.

### F. Fusionar

Cuando se apruebe:
- En GitHub, botón **"Squash and merge"** (o "Merge pull request" según preferencia).
- Eliminar la rama remota.
- Localmente:

```bash
git checkout develop
git pull
git branch -d feature/login-mejorado
```

---

## Paso 8 — Mensajes de commit: Convención

Formato: `<tipo>: <descripción imperativa>`

| Tipo | Uso |
|------|-----|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `refactor` | Reorganización sin cambiar comportamiento |
| `docs` | Cambios en documentación |
| `style` | Formato (espacios, comas, sin cambio funcional) |
| `test` | Añadir o modificar pruebas |
| `chore` | Tareas de mantenimiento (deps, config, build) |
| `perf` | Mejora de rendimiento |

**Ejemplos válidos:**

```
feat: agregar autenticación con sesiones
fix: corregir lectura de archivos con tildes
refactor: separar clase Servidor en módulos
docs: actualizar README con endpoints
chore: añadir .gitignore para binarios
```

**Ejemplos inválidos:**

```
update
fix stuff
WIP
cambios varios
```

---

## Paso 9 — Trabajo colaborativo

### A. Antes de empezar a trabajar

```bash
git checkout develop
git pull
git checkout -b feature/mi-tarea
```

**Nunca** empieces sobre una rama desactualizada.

### B. Durante

- Commits pequeños y frecuentes.
- Push regular a tu rama remota.
- Pull de `develop` si otros están trabajando en paralelo:

```bash
git checkout develop
git pull
git checkout feature/mi-tarea
git merge develop     # o git rebase develop (según preferencia del equipo)
```

### C. Al terminar

PR → revisión → merge. **Nunca** push directo a `main` ni a `develop` (protege ambas ramas en GitHub).

---

## Paso 10 — Proteger `main` en GitHub

En GitHub → Settings → Branches:

- Add rule para `main`:
  - ✅ Require a pull request before merging.
  - ✅ Require approvals (1).
  - ✅ Dismiss stale PR approvals when new commits are pushed.
  - ✅ Require status checks (si añades CI).
- Hacer lo mismo para `develop`.

Esto garantiza que nadie haga push directo a esas ramas.

---

## Paso 11 — Etiquetar versiones (releases)

Cuando `develop` esté estable y quieras pasar a `main`:

```bash
# PR de develop → main
git checkout main
git pull
git merge develop
git tag -a v1.0.0 -m "Primera versión funcional completa"
git push origin main
git push --tags
```

En GitHub → Releases → crear release asociada al tag `v1.0.0` con notas del CHANGELOG.

---

## Paso 12 — Comandos de rescate útiles

| Situación | Comando |
|-----------|---------|
| Ver estado actual | `git status` |
| Ver historial compacto | `git log --oneline --graph --all` |
| Ver cambios no commiteados | `git diff` |
| Ver quién cambió cada línea | `git blame archivo.cpp` |
| Deshacer último commit (mantener cambios) | `git reset --soft HEAD~1` |
| Deshacer último commit (borrar cambios) ⚠️ | `git reset --hard HEAD~1` |
| Revertir commit en rama compartida | `git revert <hash>` |
| Recuperar commit perdido | `git reflog` |
| Guardar cambios temporalmente | `git stash` |
| Recuperar del stash | `git stash pop` |
| Mover un commit de otra rama | `git cherry-pick <hash>` |
| Ver remotos configurados | `git remote -v` |
| Cambiar URL del remoto | `git remote set-url origin <url>` |

---

## Paso 13 — Verificación

1. El repositorio está en GitHub y es accesible.
2. `main` y `develop` están **protegidas** en GitHub.
3. `git log --oneline --graph --all` muestra un historial limpio.
4. Los commits tienen mensajes descriptivos en formato `tipo: descripción`.
5. `.gitignore` excluye binarios y `data/sistema.db`.
6. Hay al menos un PR fusionado.
7. El README está actualizado.

---

## ✅ Checklist de la Actividad 44

- [ ] Repositorio en GitHub con `main` y `develop`.
- [ ] `.gitignore` configurado.
- [ ] Rama `feature/*` por cada funcionalidad.
- [ ] Mensajes de commit en formato `tipo: descripción`.
- [ ] Al menos un PR abierto, revisado y fusionado.
- [ ] `main` y `develop` protegidas.
- [ ] Al menos un tag/release etiquetado.

## 🧠 Mini-reto
Configura **GitHub Actions** en `.github/workflows/build.yml` para compilar con `make` en cada push a `develop`. Si la compilación falla, el PR se marca como no aprobable.