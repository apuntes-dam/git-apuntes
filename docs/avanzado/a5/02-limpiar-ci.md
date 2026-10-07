# A5.2 Limpiar, exportar y automatizar en el servidor

## Dejar de seguir un archivo: `git rm --cached`

Si un archivo **ya está en el repositorio** y luego lo añades a `.gitignore`, **no pasa nada**: `.gitignore` solo vale para archivos **que Git todavía no conoce**. Hay que **sacarlo del seguimiento** (sin borrarlo de tu carpeta) con `git rm --cached`. Pasa mucho con archivos de configuración local o claves que alguien subió **por error**:

```console
$ git ls-files
.env
README.md
app.txt
$ echo .env >> .gitignore
$ git status -s
?? .gitignore
$ git rm -q --cached .env
$ git add .gitignore
$ git commit -q -m 'Deja de seguir .env'
$ git ls-files
.gitignore
README.md
app.txt
$ ls -a
.
..
.env
.git
.gitignore
README.md
app.txt
```

Al principio, `.env` **estaba seguido** y `.gitignore` no lo remedia. Con `git rm --cached` **deja de estar en el repositorio** (ya no aparece en `git ls-files`), pero **sigue en tu carpeta** (`ls -a` lo muestra). Ojo: sigue estando **en los commits antiguos**, y si contenía una **clave o una contraseña**, hay que **cambiarla** igualmente, porque **ya se ha publicado** en la historia.

### ¿Por qué se ignora algo? `check-ignore`

Cuando un archivo «no se sube» y no sabes por qué, `git check-ignore -v` **te dice qué regla lo ignora**:

```console
$ echo '*.log' > .gitignore
$ echo error > depuracion.log
$ git check-ignore -v depuracion.log
.gitignore:1:*.log	depuracion.log
$ git status -s
?? .gitignore
```

Dice **el archivo y la línea** de la regla (`.gitignore:1:*.log`). Y `git status -s` no muestra `depuracion.log`, como corresponde.

## Limpiar lo que no está en el repositorio: `git clean`

Tras compilar o probar, la carpeta se llena de **archivos que git no conoce** (`??`). **`git clean`** los borra. Es **peligroso**, porque lo que borra **no se puede recuperar** (nunca estuvo en un commit), así que **siempre se prueba primero** con **`-n`** (*dry run*, «en seco»), que **solo enseña lo que borraría**:

```console
$ echo basura > temporal.tmp
$ mkdir carpeta && echo otra > carpeta/otra.tmp
$ git status -s
?? carpeta/
?? temporal.tmp
$ git clean -n
Would remove temporal.tmp
$ git clean -nd
Would remove carpeta/
Would remove temporal.tmp
$ git clean -fdq
$ git status -s
$ ls
README.md
```

`git clean -n` **no borra nada** y, sin `-d`, **ignora las carpetas**: por eso hace falta **`-nd`** para ver también `carpeta/`. Cuando el resultado es lo esperado, **`-fd`** lo borra de verdad (`-f` = forzar, obligatorio; `-d` = incluir carpetas). Por defecto **no toca los archivos ignorados**; con **`-x`** también borraría **esos**.

| Opción | Qué hace |
|---|---|
| `-n` | **Solo enseña** lo que borraría |
| `-f` | **Borra** (hay que ponerlo: es una protección) |
| `-d` | Incluye **carpetas** sin seguimiento |
| `-x` | Incluye **también los archivos ignorados** (por ejemplo, una compilación entera) |
| `-i` | Modo **interactivo**: te pregunta |

## Exportar el proyecto: `git archive`

Para **entregar el código sin la carpeta `.git`** ni los archivos ignorados (un `zip` para enviar, o para el servidor), **`git archive`** exporta **exactamente lo que está en un commit**:

```console
$ echo 'secreto' > .env
$ echo '.env' > .gitignore
$ echo 'datos' > datos.txt
$ git add .gitignore datos.txt
$ git commit -q -m 'Añade datos'
$ git archive --format=tar HEAD | tar -t
.gitignore
README.md
datos.txt
```

Solo salen los archivos **del repositorio**: **no está `.env`** (estaba ignorado) y **no hay carpeta `.git`**. Con `--format=zip -o proyecto.zip` se obtiene directamente un `.zip`, y con `HEAD` puedes poner **una etiqueta** (`v1.0`) para exportar **esa versión**.

## Un registro de cambios con `log`

Si los mensajes siguen un formato, **un registro de cambios** (*changelog*) sale casi solo. Con una etiqueta de la última versión, `log` lista **lo que ha pasado desde entonces**:

```console
$ git tag -a v1.0 -m 'Versión 1.0'
$ echo a > a.txt && git add a.txt && git commit -q -m 'feat: añade la lista de la compra'
$ echo b > b.txt && git add b.txt && git commit -q -m 'fix: corrige el total'
$ echo c > c.txt && git add c.txt && git commit -q -m 'docs: explica cómo instalarlo'
$ git log --format='- %s' v1.0..HEAD
- docs: explica cómo instalarlo
- fix: corrige el total
- feat: añade la lista de la compra
```

`v1.0..HEAD` significa «lo que hay desde `v1.0` hasta ahora», y `--format='- %s'` lo deja como **una lista de Markdown** lista para pegar en la descripción de una versión.

## Automatizar en el servidor: integración continua

Los hooks se ejecutan **en tu equipo** y se pueden esquivar. La **integración continua** (CI, *continuous integration*) ejecuta **las pruebas en un servidor, en cada `push` o Pull Request**, y **nadie puede saltársela**. En GitHub se llama **GitHub Actions**, y se configura con un archivo YAML en **`.github/workflows/`**. Un ejemplo mínimo que **pasa las pruebas de un proyecto Python** en cada subida:

```yaml
# .github/workflows/pruebas.yml
name: Pruebas
on: [push, pull_request]        # cuándo se ejecuta

jobs:
  pruebas:
    runs-on: ubuntu-latest       # en qué sistema
    steps:
      - uses: actions/checkout@v4          # descarga el código
      - uses: actions/setup-python@v5      # instala Python
        with:
          python-version: "3.13"
      - run: python -m unittest -v         # ejecuta las pruebas
```

!!! note "Este archivo no se ha ejecutado aquí"
    A diferencia del resto de ejemplos de esta unidad, **GitHub Actions necesita GitHub**: este YAML es una **estructura típica**, no una salida comprobada en esta página. Las versiones de las acciones (`@v4`, `@v5`) cambian con el tiempo: mira la documentación al usarlas.

Con ello, **un Pull Request que rompe las pruebas se marca en rojo** antes de que nadie lo fusione, y en la configuración del repositorio se puede **obligar** a que pasen para poder fusionar. Es el **complemento** de los hooks: los hooks avisan **rápido y en local**; la CI **garantiza** lo importante **en el servidor**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Añadir un archivo a `.gitignore` esperando que se deje de seguir | Si ya estaba, hace falta `git rm --cached` |
| Subir una **clave** y luego solo borrarla | Queda en la historia: **cámbiala** y trátala como filtrada |
| `git clean -f` sin probar antes | Siempre **`-n`** primero |
| Entregar el proyecto con la carpeta `.git` o con archivos ignorados | `git archive` |
| Confiar solo en los hooks locales | Repite lo importante en la **integración continua** |

## Para practicar

Los ejercicios GA5.3 a GA5.5 de [GA5 · Ejercicios](ejercicios.md) practican `git rm --cached`, `git clean` y `git archive`.
