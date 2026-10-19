# Macrodocumento de Estudio y Examen: Git, GitHub Pages y GitHub Actions
> **Módulo:** Despliegue de Aplicaciones Web (2º DAW)  
> **Formato:** Cortito, al pie, ultra-estructurado y con bloques de código listos para copiar y pegar.

---

## Índice
1. [Las 5 Reglas de Oro y Configuración Inicial](#1-las-5-reglas-de-oro-y-configuración-inicial)
2. [Terminal y Editores de Rescate](#2-terminal-y-editores-de-rescate)
3. [Git Local: Ciclo de Vida, Zonas y Deshacer](#3-git-local-ciclo-de-vida-zonas-y-deshacer)
4. [Ramas, Fusiones y Resolución de Conflictos](#4-ramas-fusiones-y-resolución-de-conflictos)
5. [Tags (Etiquetas) y Entregas](#5-tags-etiquetas-y-entregas)
6. [Remotos, Conexión con GitHub, Fork y Pull Requests](#6-remotos-conexión-con-github-fork-y-pull-requests)
7. [GitHub Pages: Despliegue de Webs](#7-github-pages-despliegue-de-webs)
8. [Teoría de SCV y Metodología Gitflow](#8-teoría-de-scv-y-metodología-gitflow)
9. [MACRO-BLOQUE: GitHub Actions al Detalle](#9-macro-bloque-github-actions-al-detalle)
   - 9.1 [Anatomía y Jerarquía de un Workflow](#91-anatomía-y-jerarquía-de-un-workflow)
   - 9.2 [Disparadores (`on`), Filtros y Cron](#92-disparadores-on-filtros-y-cron)
   - 9.3 [Variables de Entorno, Secretos y Contextos](#93-variables-de-entorno-secretos-y-contextos)
   - 9.4 [Dependencias (`needs`), Artefactos y Matrices (`matrix`)](#94-dependencias-needs-artefactos-y-matrices-matrix)
   - 9.5 [Plantilla Examen 1: Multi-SO y Artefactos (PyInstaller)](#95-plantilla-examen-1-multi-so-y-artefactos-pyinstaller)
   - 9.6 [Plantilla Examen 2: Modificaciones euro.py y Versión Pro Multirama](#96-plantilla-examen-2-modificaciones-europy-y-versión-pro-multirama)
   - 9.7 [Custom Actions (Docker, JS y Composite)](#97-custom-actions-docker-js-y-composite)
10. [Tabla Rápida: «Si el enunciado pide... tú ejecutas...»](#10-tabla-rápida-si-el-enunciado-pide-tú-ejecutas)
11. [Tabla de Errores Típicos y Solución Instantánea](#11-tabla-de-errores-típicos-y-solución-instantánea)
12. [Checklist Final antes de Entregar](#12-checklist-final-antes-de-entregar)

---

## 1. Las 5 Reglas de Oro y Configuración Inicial

### Las 5 reglas sagradas de examen
1. **Antes de tocar nada:** Ejecuta `git status` y `git branch` (¿en qué rama estás y qué tienes pendiente?).
2. **Sintaxis exacta:** Todo comando empieza por `git`. Los flags no llevan espacios (`--soft`, `--hard`, `--staged`).
3. **Cambios reales:** Para simular cambios, usa `echo "texto" >> archivo` o edita con `nano`. Un comando `touch` sobre un archivo existente no modifica contenido ni genera commit nuevo.
4. **Los tags no van solos:** Un `git push` normal **NUNCA** sube tags. Hay que subirlos explícitamente (`git push origin <tag>` o `git push origin --tags`).
5. **Comprobación final:** Verifica siempre el histórico con `git log --oneline --graph --all --decorate`.

### Configuración de identidad en la máquina del examen
```bash
# Nombre y correo (fija tu autoría en los commits)
git config --global user.name "TuNombre Apellido"
git config --global user.email "tu_correo@ejemplo.com"

# Editor por defecto (evita atascarse con Vim)
git config --global core.editor nano

# Recordar credenciales para no meter el token en cada push
git config --global credential.helper store

# Comprobar qué hay guardado
git config --list
```

### El Token de Acceso Personal (PAT) de GitHub
GitHub no admite tu contraseña habitual en la terminal. Usa un token clásico:
1. Web GitHub → Clic en tu avatar → **Settings** → **Developer Settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token**.
2. Marca permisos: `repo` (completo) y `workflow` (vital para subir Actions).
3. **Copia el token** en un bloc de notas temporal.
4. Cuando la terminal pida:
   - `Username`: Tu usuario de GitHub.
   - `Password`: Pega el Token (no saldrán asteriscos ni letras al pegar, es normal).

---

## 2. Terminal y Editores de Rescate

### Comandos de navegación y archivos
```bash
pwd                     # Dónde estoy
ls -la                  # Listar todo (incluye .git y archivos ocultos)
mkdir -p a/b/c          # Crear carpetas anidadas de golpe
cp origen destino       # Copiar
mv origen destino       # Mover o renombrar
rm archivo              # Borrar archivo
rm -rf carpeta          # Borrar carpeta y su contenido
chmod +x script.sh      # Dar permisos de ejecución imprescindible en Actions
```

### Escribir bloques de texto sin abrir editor (Heredoc)
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Web</title></head>
<body><h1>Hola Mundo</h1></body>
</html>
EOF
```

### Salir de los editores si te atrapan
- **Nano:** `Ctrl + O` luego `Enter` (guardar) → `Ctrl + X` (salir). Buscar: `Ctrl + W`.
- **Vim / Vi:** Pulsa `Esc` varias veces → escribe `:wq` y `Enter` (guardar y salir) o `:q!` y `Enter` (salir sin guardar).
- **Git log bloqueado:** Pulsa la tecla `q`.

---

## 3. Git Local: Ciclo de Vida, Zonas y Deshacer

### Las 3 zonas de Git
```
+---------------------+     git add     +---------------------+    git commit    +---------------------+
|    Working Tree     | --------------> |    Staging Area     | ---------------> |  Local Repository   |
| (archivos editados) |                 | (índice preparado)  |                  |    (.git / commits) |
+---------------------+                 +---------------------+                  +---------------------+
```

### Ciclo estándar de trabajo
```bash
# 1. Crear repositorio
git init

# 2. Ver estado
git status

# 3. Preparar cambios
git add index.html          # Uno solo
git add .                   # Todos los archivos de la carpeta
git add -u                  # Solo los archivos ya rastreados

# 4. Confirmar en el historial
git commit -m "Añade página principal"

# Atajo (add + commit a la vez; OJO: solo para archivos que YA estaban bajo seguimiento)
git commit -am "Actualiza estilos de la cabecera"

# Ver diferencias
git diff                    # Cambios modificados vs staging (antes de git add)
git diff --staged           # Cambios en staging vs último commit
git diff rama1 rama2        # Diferencias entre dos ramas
```

### Archivo `.gitignore`
Evita subir basura o secretos al repositorio.
```gitignore
# Archivo .gitignore de ejemplo
*.log
node_modules/
dist/
build/
.env
__pycache__/
venv/
!importante.log             # Excepción: este log sí se sube
```
> **Atención:** Si ya hiciste commit de un archivo y luego lo añades al `.gitignore`, Git lo seguirá rastreando. Para desvincularlo sin borrarlo del disco:
```bash
git rm --cached archivo.log
git commit -m "Deja de rastrear archivo.log"
```

### Deshacer cambios
```bash
# Sacar un archivo de staging sin perder cambios
git restore --staged archivo.txt      # o: git reset archivo.txt

# Descartar cambios de un archivo modificado (volver al último commit)
git restore archivo.txt              # o: git checkout -- archivo.txt

# Modificar el mensaje del último commit (o añadir archivos olvidados)
git add archivo_olvidado.txt
git commit --amend -m "Nuevo mensaje corregido" --no-edit

# Deshacer commits (HEAD~1 = el último commit)
git reset --soft HEAD~1              # Borra el commit; deja los cambios en STAGING
git reset HEAD~1                     # (Mixed) Borra el commit; deja cambios en WORKING
git reset --hard HEAD~1              # PELIGRO: Borra el commit y destruye los cambios

# Deshacer un commit creando uno nuevo inverso (recomendado para repos remotos)
git revert <HASH_DEL_COMMIT>
```

---

## 4. Ramas, Fusiones y Resolución de Conflictos

### Comandos esenciales de ramas
```bash
git branch                           # Listar ramas locales (* indica la actual)
git branch --show-current            # Imprime solo el nombre de la rama activa
git branch -m nueva-rama             # Renombrar la rama en la que estás (ej: cambiar master a main)

# Crear y moverse
git switch -c feature-contacto       # Moderno (o: git checkout -b feature-contacto)
git switch master                    # Volver a master (o: git checkout master)

# Borrar ramas
git branch -d feature-contacto       # Borra la rama (solo si ya está fusionada)
git branch -D feature-contacto       # Fuerza el borrado aunque no esté fusionada
```

### Fusión (*Merge*): Fast-Forward vs 3 Vías
1. **Fast-Forward:** La rama base no se ha movido. Git solo desliza el puntero hacia adelante. No crea commit de merge.
2. **3 Vías (No Fast-Forward):** Ambas ramas tienen commits nuevos separados. Git crea automáticamente un commit de fusión.
```bash
# Para fusionar feature en master:
git switch master
git merge feature-contacto           # Realiza la fusión
git merge --no-ff feature-contacto   # Fuerza commit de merge aunque pudiera ser fast-forward
```

### Resolución de conflictos paso a paso
Si dos ramas tocaron la misma línea del mismo archivo, `git merge` se detendrá con `CONFLICT (content)`.
1. Ejecuta `git status` para ver qué archivo está en conflicto (`both modified`).
2. Abre el archivo con `nano`:
```html
<<<<<<< HEAD
<h1>Título en rama master</h1>
=======
<h1>Título en rama feature</h1>
>>>>>>> feature-contacto
```
3. Borra las marcas (`<<<<<<<`, `=======`, `>>>>>>>`) y deja el contenido definitivo.
4. Concluye el merge:
```bash
git add archivo_arreglado.html
git commit -m "Resuelve conflicto entre master y feature-contacto"

# Si quieres abortar el merge y dejar todo como estaba antes de empezar:
git merge --abort
```

---

## 5. Tags (Etiquetas) y Entregas

Los tags marcan hitos estables (versiones de entrega).
- **Ligero:** Solo un nombre apuntando al commit.
- **Anotado (el exigido en examen):** Guarda autor, fecha y mensaje propio.

```bash
# Crear tag anotado en el commit actual
git tag -a entrega-v1.0 -m "Entrega práctica módulo 1 - TuNombre"

# Crear tag sobre un commit antiguo (etiquetado tardío)
git log --oneline                    # Copias el hash de 7 caracteres
git tag -a v0.9 -m "Versión previa" a1b2c3d

# Ver datos y autoría del tag
git show entrega-v1.0

# Subir tags a GitHub (¡VITAL!)
git push origin entrega-v1.0         # Sube este tag específico
git push origin --tags               # Sube todos los tags locales

# Borrar tags
git tag -d v0.9                      # Borrar en local
git push origin --delete v0.9        # Borrar en GitHub
```

---

## 6. Remotos, Conexión con GitHub, Fork y Pull Requests

### Conectar un repositorio local a GitHub
```bash
# 1. Vincular la URL de tu repositorio remoto
git remote add origin https://github.com/TU_USUARIO/NOMBRE_REPO.git

# 2. Si te equivocaste de URL, corrígela con set-url (NUNCA hagas add otra vez):
git remote set-url origin https://github.com/TU_USUARIO/NOMBRE_CORRECTO.git

# 3. Comprobar a qué URL apunta
git remote -v

# 4. Primer push (vincula rama local con rama remota con -u)
git push -u origin master            # O 'main', según indique git branch

# 5. En los siguientes commits basta con:
git push
```

### Recibir cambios del remoto
```bash
git pull origin master               # Descarga y fusiona en tu rama actual (fetch + merge)
git fetch origin                     # Solo descarga referencias a origin/master sin mezclar
```

### Flujo de trabajo con Fork y Pull Request (PR)
1. En GitHub, pulsa el botón **Fork** del repositorio original para copiarlo a tu cuenta.
2. Clona **tu fork** en tu ordenador:
```bash
git clone https://github.com/TU_USUARIO/repo-fork.git && cd repo-fork
```
3. Añade el repositorio original como remoto `upstream` para mantenerte al día:
```bash
git remote add upstream https://github.com/PROFE/repo-original.git
git fetch upstream
git merge upstream/main
```
4. Crea una rama de trabajo, haz commits y súbela a tu fork:
```bash
git switch -c mejora-rendimiento
# Realiza cambios...
git add . && git commit -m "Añade optimizaciones"
git push -u origin mejora-rendimiento
```
5. En la interfaz web de GitHub: Pulsa **Contribute** → **Open Pull Request** → Selecciona tu rama contra la rama base del repositorio original → **Create Pull Request**.

---

## 7. GitHub Pages: Despliegue de Webs

Existen dos modalidades para publicar webs en GitHub Pages:

### Modalidad A: Repositorio de Usuario (`TUUSUARIO.github.io`)
- **Regla estricta:** El nombre del repositorio debe ser exactamente `<tu-usuario>.github.io`.
- Los archivos (`index.html`, `css/`, `js/`) deben estar en la **raíz** de la carpeta, no metidos en subdirectorios extras.
```bash
mkdir ~/web && cd ~/web
# Descomprimir o crear los archivos web aquí directamente
git init
git add .
git commit -m "Primer despliegue web"
git remote add origin https://github.com/TU_USUARIO/TU_USUARIO.github.io.git
git push -u origin master
```
- Ir a GitHub → **Settings** → **Pages** → Source: **Deploy from a branch** → Rama `master` / `/ (root)` → **Save**.
- URL generada: `https://TU_USUARIO.github.io/`

### Modalidad B: Repositorio de Proyecto (Rama `gh-pages`)
- Para repositorios con nombres comunes (ej: `tienda-online`):
```bash
git switch -c gh-pages
git add .
git commit -m "Publica en gh-pages"
git push -u origin gh-pages
```
- En **Settings** → **Pages**, selecciona la rama `gh-pages`.
- URL generada: `https://TU_USUARIO.github.io/tienda-online/`

---

## 8. Teoría de SCV y Metodología Gitflow

### Comparativa de Sistemas de Control de Versiones
| Tipo | Funcionamiento | Ventajas / Inconvenientes | Ejemplos |
|---|---|---|---|
| **Local** | Base de datos simple con cambios de archivos en el propio PC. | Fácil, pero sin colaboración y si se rompe el disco se pierde todo. | RCS |
| **Centralizado** | Un único servidor central contiene todos los archivos y versiones; los clientes descargan solo una instantánea. | Colaboración fácil; pero punto único de fallo (si cae el servidor, nadie commitea). | CVS, Subversion (SVN) |
| **Distribuido** | Cada cliente clona el historial íntegro del repositorio en su disco local. | Trabajo offline completo, redundancia total, tolerancia a fallos. | Git, Mercurial |

*Diferencia clave Git vs SVN:* SVN almacena diferencias incrementales entre versiones. Git almacena un conjunto de **instantáneas de archivos íntegros** (*snapshots*) y garantiza la integridad mediante hashes **SHA-1** de 40 dígitos hexadecimales.

---

### Metodología Gitflow
Divide el desarrollo en ramas con propósitos específicos:

```
master/main  -----------------------------------------● (v1.0) -----------● (v1.1) (Producción)
                                                     /                    /
release      ------------------------------●--------/                    /
                                          /                             /
develop      ---●------------●-----------●-----------------------------●------------- (Desarrollo)
                 \          /
feature           ●--------● (feature/login)
```

| Rama | Propósito | Nace de | Se fusiona en | Convención de nombre |
|---|---|---|---|---|
| `master` / `main` | Código en producción (siempre estable). | — | — | `master` / `main` |
| `develop` | Desarrollo de la próxima versión estable. | `master` | — | `develop` |
| `feature` | Nuevas funcionalidades independientes. | `develop` | `develop` | `feature/<nombre>` |
| `release` | Preparar y pulir una versión (solo bugs). | `develop` | `develop` y `master` (+ tag) | `release/<versión>` |
| `hotfix` | Arreglo crítico urgente en producción. | `master` | `develop` y `master` (+ tag) | `hotfix/<versión>` |

#### Comandos con el plugin `git-flow` vs Git Estándar
```bash
# Inicializar
git flow init                  # O bien: git checkout -b develop master

# Crear y terminar una feature
git flow feature start login   # O bien: git switch -c feature/login develop
# ...hacer commits...
git flow feature finish login  # O bien:
                               # git switch develop
                               # git merge --no-ff feature/login
                               # git branch -d feature/login

# Publicar feature en el remoto para compartirla con un compañero
git flow feature publish login # O bien: git push origin feature/login

# Crear y terminar una release
git flow release start 1.0.0   # O bien: git switch -c release/1.0.0 develop
# ...ajustes de última hora...
git flow release finish 1.0.0  # Fusiona en master, crea tag '1.0.0', fusiona en develop y borra rama

# Hotfix urgente de producción
git flow hotfix start 1.0.1    # O bien: git switch -c hotfix/1.0.1 master
git flow hotfix finish 1.0.1   # Fusiona en master con tag '1.0.1' y en develop
```

---

## 9. MACRO-BLOQUE: GitHub Actions al Detalle

### 9.1 Anatomía y Jerarquía de un Workflow
- **Ubicación obligatoria:** `.github/workflows/<nombre>.yml` (**con 's' final en workflows**, si no, GitHub lo ignora por completo).
- **Formato:** YAML estricto. **2 espacios de sangría por nivel, jamás usar el tabulador (Tab)**. Poner siempre un espacio tras dos puntos (`clave: valor`).

```
Workflow (fichero .yml disparado por un evento)
   └── Job 1 (ejecutado en un Runner aislado: ubuntu, windows, macos)
         ├── Step 1 (uses: acción comunitaria / propia)
         ├── Step 2 (run: comando de terminal)
   └── Job 2 (corre en paralelo a Job 1, salvo que use needs: Job 1)
```

#### Regla de oro para el examen: ¿`uses` o `run`?
- Si el enunciado te da una ruta tipo `actions/checkout@v4` o `actions/setup-python@v5`, usa **`uses`**.
- Si el enunciado te da un comando de terminal como `pip install pyinstaller` o `python main.py`, usa **`run`**.
- Las versiones de acciones (`@v4`, `@v5`) y de lenguajes (`'3.12'`) **se copian exactamente como las pide el enunciado**, no se inventan.

---

### 9.2 Disparadores (`on`), Filtros y Cron

```yaml
name: Ejemplo de Eventos y Filtros

# Disparar en push y pull_request simultáneamente
on:
  push:
    branches:
      - main
      - 'releases/**'            # Coincide con releases/v1, releases/v1/beta...
      - '!releases/**-alpha'     # La exclamación (!) excluye esa coincidencia
    paths:
      - 'src/**'                 # Solo si cambian archivos dentro de src/
      - 'requirements.txt'
    tags:
      - 'v*'                     # Solo al subir tags que empiecen por v (ej: v1.0)
  pull_request:
    branches:
      - main
  # Disparo manual con un botón en la interfaz de GitHub
  workflow_dispatch:
  # Programado periódicamente (sintaxis cron en UTC: min hora día mes díaSemana)
  schedule:
    - cron: '30 7 * * 1-5'       # Lunes a viernes a las 07:30 UTC
```

---

### 9.3 Variables de Entorno, Secretos y Contextos

#### Jerarquía de variables de entorno (`env`)
Se pueden declarar a nivel de:
1. **Workflow:** Disponible para todos los jobs.
2. **Job:** Disponible para todos los steps de ese job.
3. **Step:** Solo visible dentro de ese comando. Gana siempre la más específica.

#### Contextos más utilizados
- `${{ github.ref_name }}`: Nombre de la rama o tag que disparó la acción (ej: `main`, `test`).
- `${{ github.sha }}`: Hash del commit actual.
- `${{ github.actor }}`: Usuario que provocó el evento.
- `${{ runner.os }}`: Sistema operativo del runner (`Linux`, `Windows`, `macOS`).
- `${{ secrets.MI_SECRETO }}`: Secretos guardados en el repo (**Settings** → **Secrets and variables** → **Actions**). En los logs saldrá enmascarado como `***`.

```yaml
env:
  GLOBAL_APP: "MiApp"

jobs:
  test-env:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: "Nivel Job"
    steps:
      - name: Imprime variables
        env:
          STEP_VAR: "Nivel Step"
        run: |
          echo "Global: $GLOBAL_APP"
          echo "Job: $JOB_VAR"
          echo "Step: $STEP_VAR"
          echo "Rama activa: ${{ github.ref_name }}"
```

---

### 9.4 Dependencias (`needs`), Artefactos y Matrices (`matrix`)

#### Dependencias entre Jobs (`needs`)
Por defecto, todos los jobs dentro de `jobs:` se ejecutan **en paralelo**. Si un job necesita que otro termine con éxito previamente:
```yaml
jobs:
  compilar:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Compilando..."

  probar:
    needs: compilar            # Espera a que termine 'compilar'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Probando..."

  desplegar:
    needs: [compilar, probar]  # Espera a que terminen ambos jobs
    runs-on: ubuntu-latest
    steps:
      - run: echo "Desplegando..."
```

#### Artefactos (Compartir ficheros entre jobs)
Cada job arranca en una máquina virtual limpia e independiente. Los archivos creados en el Job A **no existen** en el Job B a menos que se suban como artefactos compartidos.
- **Subir archivo (upload):**
```yaml
- name: Subir artefacto
  uses: actions/upload-artifact@v4
  with:
    name: mi-binario           # Identificador del artefacto
    path: dist/ejecutable       # Ruta del archivo o carpeta a guardar
```
- **Descargar archivo (download):**
```yaml
- name: Descargar artefacto
  uses: actions/download-artifact@v4
  with:
    name: mi-binario           # Debe coincidir exactamente con el nombre en upload
```

#### Estrategia Matricial (`strategy.matrix`)
Permite multiplicar un mismo job ejecutándolo contra varias versiones de un lenguaje o varios sistemas operativos:
```yaml
jobs:
  matriz-pruebas:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false         # Si una versión falla, no cancela las demás
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ['3.11', '3.12']
        include:               # Añadir una combinación específica
          - os: macos-latest
            python-version: '3.12'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: python --version
```

---

### 9.5 Plantilla Examen 1: Multi-SO y Artefactos (PyInstaller)
*Este es el ejercicio clásico del examen: compilar `comprova_desplegaments.py` en 3 sistemas operativos y validar el ejecutable resultante en un job dependiente.*

Ruta: `.github/workflows/empaquetar.yml`
```yaml
name: Empaquetat

on:
  push:
    branches:
      - master                 # OJO: Pon la rama exacta que marque tu terminal (master o main)

jobs:
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del repositorio
        uses: actions/checkout@v4

      - name: Configurar entorno Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Instalar PyInstaller
        run: pip install pyinstaller

      - name: Compilar ejecutable binario
        run: pyinstaller --onefile comprova_desplegaments.py

      - name: Guardar ejecutable como artefacto
        uses: actions/upload-artifact@v4
        with:
          name: executable-linux
          path: dist/comprova_desplegaments

  build-macos:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install pyinstaller
      - run: pyinstaller --onefile comprova_desplegaments.py

  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install pyinstaller
      - run: pyinstaller --onefile comprova_desplegaments.py

  comprova:
    needs: build-linux         # Espera a que termine la compilación de Linux
    runs-on: ubuntu-latest
    steps:
      - name: Descargar ejecutable generado en Linux
        uses: actions/download-artifact@v4
        with:
          name: executable-linux

      - name: Dar permisos de ejecucion
        run: chmod +x comprova_desplegaments

      - name: Ejecutar la comprobacion
        run: ./comprova_desplegaments example.com
```

---

### 9.6 Plantilla Examen 2: Modificaciones euro.py y Versión Pro Multirama
*Basado en la práctica de `titansmc/actions` (script `euro.py`). Las 5 tareas requeridas paso a paso:*

#### Tarea 1 y 2: Clonar y enlazar a tu propio repositorio
```bash
git clone https://github.com/titansmc/actions.git && cd actions
# Cambiar el remoto para subirlo a tu cuenta (USA set-url, NO add):
git remote set-url origin https://github.com/TU_USUARIO/actions.git
git push -u origin main
```

#### Tarea 3: Instalar paquetes usando `requirements.txt`
1. Crear el archivo `requirements.txt`:
```text
requests
pandas
```
2. Modificar en el YAML el paso de instalación:
```yaml
      - name: Instalar dependencias
        run: pip install -r requirements.txt
```
3. Subir los cambios:
```bash
git add requirements.txt .github/workflows/actions.yml
git commit -m "Usa requirements.txt para dependencias"
git push
```

#### Tarea 4: Rama `test` con Python 3.12
```bash
git switch -c test
# Modificar en .github/workflows/actions.yml:
# python-version: '3.12'
# Asegurar que 'test' esté en on: push: branches: [ main, test ]
git add .github/workflows/actions.yml
git commit -m "Configura Python 3.12 en rama test"
git push -u origin test
```

#### Tarea 5: Rama `development` ejecutando solo con parámetro USD
```bash
git switch main                # Volver a la base limpia
git switch -c development
# Modificar en el YAML el comando de ejecución:
# run: python ./euro.py USD
# Asegurar que 'development' esté en on: push: branches: [ main, test, development ]
git add .github/workflows/actions.yml
git commit -m "Ejecuta euro.py solo con USD en development"
git push -u origin development
```

---

#### 🌟 Versión PRO: Un solo YAML universal válido para todas las ramas
En vez de mantener 3 archivos YAML desincronizados, este workflow inteligente usa condicionales inline y directivas `if:` para adaptarse automáticamente a cada rama:

```yaml
name: Monedas-Universal

on:
  push:
    branches:
      - main
      - test
      - development

jobs:
  create-list:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Configurar Python dinámico
        uses: actions/setup-python@v5
        with:
          # Si estamos en rama test usa 3.12, si no usa 3.11 (operador ternario)
          python-version: ${{ github.ref_name == 'test' && '3.12' || '3.11' }}

      - name: Instalar paquetes
        run: pip install -r requirements.txt

      - name: Ejecucion para main y test (todas las divisas)
        if: github.ref_name != 'development'
        run: python ./euro.py

      - name: Ejecucion para development (solo USD)
        if: github.ref_name == 'development'
        run: python ./euro.py USD
```

---

### 9.7 Custom Actions (Docker, JS y Composite)

Una Custom Action es un componente reutilizable empaquetado por ti.

#### Comparativa de tipos
| Tipo | Velocidad | Compatibilidad de SO | Gestión de dependencias |
|---|---|---|---|
| **Docker** | Lenta (descarga y compila imagen) | **Solo Linux** | Dentro del contenedor Docker |
| **JavaScript** | Rápida | Linux, Windows, macOS | Debe compilarse con `@vercel/ncc` |
| **Composite** | Inmediata | Según lo que use cada step | Steps normales encadenados |

#### Ejemplo de Custom Action con Docker
Estructura de archivos:
```
mi-accion/
├── action.yml
├── Dockerfile
└── entrypoint.sh
```

**1. `action.yml` (Manifiesto de la acción):**
```yaml
name: 'Mi Accion Personalizada'
description: 'Saluda al usuario y devuelve la hora'
author: 'TuNombre'

inputs:
  who-to-greet:
    description: 'Persona a saludar'
    required: true
    default: 'Mundo'

outputs:
  time:
    description: 'Hora en que se completó'

runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

**2. `Dockerfile`:**
```dockerfile
FROM alpine:3.18
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**3. `entrypoint.sh`:**
```bash
#!/bin/sh -l
echo "Hola $1!"
# Sintaxis moderna para definir outputs en GitHub Actions:
echo "time=$(date)" >> $GITHUB_OUTPUT
```

**4. Cómo consumir esta acción local en un workflow:**
```yaml
jobs:
  saludo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4        # OBLIGATORIO para descargar tu propio repo
      - name: Ejecuta la custom action
        id: saluda-step
        uses: ./mi-accion                # Ruta donde vive el action.yml
        with:
          who-to-greet: 'Valen'
      - name: Muestra el output obtenido
        run: echo "La hora fue: ${{ steps.saluda-step.outputs.time }}"
```

---

## 10. Tabla Rápida: «Si el enunciado pide... tú ejecutas...»

| Si el enunciado dice... | Comando / Sintaxis exacta |
|---|---|
| **Configurar identidad** | `git config --global user.name "Nombre"` y `git config --global user.email "email"` |
| **Crear repositorio nuevo** | `mkdir carpeta && cd carpeta && git init` |
| **Comprobar en qué rama estás** | `git branch` o `git branch --show-current` |
| **Renombrar rama a master o main** | `git branch -m master` |
| **Crear una rama y cambiarte a ella** | `git switch -c nombre-rama` (o `git checkout -b nombre-rama`) |
| **Guardar cambios de archivos nuevos** | `git add . && git commit -m "Mensaje"` |
| **Deshacer el último commit sin perder archivos** | `git reset --soft HEAD~1` |
| **Fusionar una rama en master** | `git switch master && git merge nombre-rama` |
| **Crear tag de entrega del examen** | `git tag -a entrega-tu_nombre -m "Entrega examen"` |
| **Subir la rama al repositorio remoto** | `git push -u origin <nombre-rama>` |
| **Subir el tag al repositorio remoto** | `git push origin entrega-tu_nombre` |
| **Corregir la URL remota errónea** | `git remote set-url origin https://github.com/USUARIO/REPO.git` |
| **Workflow en Actions: solo en una rama** | `on: push: branches: [ nombre-rama ]` |
| **Workflow: instalar dependencias Python** | `run: pip install -r requirements.txt` |
| **Workflow: versión de Python específica** | `uses: actions/setup-python@v5` con `with: python-version: '3.12'` |
| **Workflow: pasar un ejecutable a otro job** | `upload-artifact@v4` en el 1º, `needs:` y `download-artifact@v4` en el 2º |
| **Permisos de ejecución en Linux** | `run: chmod +x ./nombre_binario` |

---

## 11. Tabla de Errores Típicos y Solución Instantánea

| Mensaje de Error / Síntoma | Causa Real | Solución Inmediata |
|---|---|---|
| `fatal: not a git repository` | No estás en la carpeta o falta inicializar. | Haz `cd` a la carpeta correcta o ejecuta `git init`. |
| `fatal: remote origin already exists` | Ya agregaste el remoto con `add`. | Usa `git remote set-url origin <URL>` para actualizarlo. |
| `error: src refspec master does not match any` | Tu rama local se llama `main` o aún no has hecho ningún commit. | Haz el primer commit o renómbrala con `git branch -m master`. |
| `fatal: Authentication failed` / Pide contraseña | GitHub no acepta contraseñas normales. | Genera un Personal Access Token clásico y pégalo como contraseña. |
| `refusing to allow a Personal Access Token to create or update workflow` | Tu token no tiene permisos de Actions. | En GitHub genera otro token marcando la casilla **`workflow`**. |
| `[rejected - non-fast-forward]` | El repo en GitHub se creó con README y tiene commits que tú no tienes. | Crea el repo de GitHub completamente vacío, o haz `git pull origin master --allow-unrelated-histories`. |
| `error: branch 'X' is not fully merged` | Intentas borrar una rama con `-d` sin fusionar. | Si estás seguro de querer destruirla, usa `git branch -D X`. |
| En GitHub Actions la pestaña está vacía | El archivo no está en la ruta exacta. | Comprueba que la carpeta sea `.github/workflows/` (**con s final**). |
| Workflow no se ejecuta al hacer push | El filtro `branches:` no incluye la rama a la que subiste. | Añade tu rama a `branches: [ ... ]` en el YAML. |
| `Invalid workflow file (yaml syntax error)` | Errores de sangría o tabuladores. | Sustituye tabuladores por 2 espacios y pon espacio tras cada dos puntos `:`. |
| El step de Action dice "file not found" | Olvidaste clonar el repositorio dentro del runner. | Añade `- uses: actions/checkout@v4` como primer paso del job. |
| `Permission denied` en Linux runner | El script o binario no tiene permisos de ejecución. | Ejecuta `run: chmod +x ./mi_script.sh` antes de invocarlo. |
| Los tags no aparecen en GitHub | Se hizo un push simple sin tags. | Ejecuta `git push origin --tags`. |

---

## 12. Checklist Final antes de Entregar

Antes de avisar al examinador o dar por concluido el examen, revisa punto por punto:

- [ ] **1. `git status` limpio:** No hay archivos en rojo sin commitear ni cambios pendientes.
- [ ] **2. En la rama adecuada:** Ejecuta `git branch` para confirmar que estás en la rama exigida por el enunciado.
- [ ] **3. Histórico limpio y decorado:** Ejecuta `git log --oneline --graph --all --decorate` y comprueba que las fusiones y los tags coinciden con el esquema del examen.
- [ ] **4. Ramas temporales eliminadas:** Si el enunciado pedía borrar las ramas secundarias tras el merge, verifica con `git branch` que solo quedan las requeridas.
- [ ] **5. Tags subidos:** Comprueba en la web de GitHub (pestaña **Tags** del repositorio) que tu tag de entrega está visible.
- [ ] **6. Pestaña Actions en verde:** Entra en GitHub → Pestaña **Actions**. Debe haber una ejecución para tu último commit y todos los jobs deben mostrar el tick verde. Si hay una cruz roja, entra en el job, lee el paso en rojo y arréglalo.
- [ ] **7. Si era Pages:** Abre `https://<usuario>.github.io/<repo>/` en modo incógnito (o recarga con `Ctrl + F5`) y confirma que la página carga sin errores 404.
