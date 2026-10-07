# A5.1 Hooks, alias y configuración

!!! info "Para quién es esto"
    Es material **avanzado**. Las sesiones son **reales** y los números de commit serán **distintos en tu equipo**. Los ejemplos parten de un repositorio con un solo commit:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
```

## Hooks: programas que Git ejecuta por ti

Un **hook** es un **script** que Git ejecuta **automáticamente en un momento concreto**: antes de hacer un commit, al comprobar su mensaje, antes de subir... Si el script **termina con error** (código distinto de 0), **Git cancela la operación**. Así se puede **impedir** que se cuelen cosas que no deben.

Los hooks viven en la carpeta **`.git/hooks/`**: un archivo **ejecutable** cuyo **nombre es el del momento**. Los más útiles:

| Hook | Cuándo se ejecuta | Para qué se usa |
|---|---|---|
| `pre-commit` | **Antes** de crear el commit | Pasar el formateador, el analizador o las pruebas rápidas; rechazar `TODO`, claves o `console.log` |
| `commit-msg` | Tras escribir el mensaje, **antes** de aceptar el commit | Exigir un **formato** de mensaje |
| `pre-push` | **Antes** de subir | Pasar las pruebas completas |
| `post-merge` | Tras una fusión | Reinstalar dependencias si cambiaron |

### `pre-commit`: rechazar un TODO

Este hook **mira lo que se va a confirmar** (`git diff --cached`) y, si encuentra la palabra `TODO` en una línea nueva, **cancela el commit**:

```console
$ cat > .git/hooks/pre-commit <<'EOF'
#!/bin/sh
# Impide hacer commit si algo de lo que se va a confirmar contiene la palabra TODO
if git diff --cached | grep -q '^+.*TODO'; then
  echo "Commit rechazado: hay un TODO sin resolver."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
$ echo 'TODO: arreglar esto' > notas.txt && git add notas.txt && git commit -m 'Añade notas'
Commit rechazado: hay un TODO sin resolver.
$ echo 'Notas del proyecto' > notas.txt && git add notas.txt && git commit -m 'Añade notas'
[main c9967fa] Añade notas
 1 file changed, 1 insertion(+)
 create mode 100644 notas.txt
$ git log --oneline
c9967fa Añade notas
398526b Crea el proyecto
```

El primer commit **se rechaza** con el mensaje del hook y **no se crea**; al quitar el `TODO`, el hook no protesta y el commit se hace. Si en algún caso de verdad quieres **saltarte** el hook, existe `git commit --no-verify`, **pero no debería ser lo habitual**:

```console
$ cat > .git/hooks/pre-commit <<'EOF'
#!/bin/sh
# Impide hacer commit si algo de lo que se va a confirmar contiene la palabra TODO
if git diff --cached | grep -q '^+.*TODO'; then
  echo "Commit rechazado: hay un TODO sin resolver."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
$ echo 'TODO: arreglar esto' > notas.txt && git add notas.txt && git commit -q --no-verify -m 'Añade notas con un TODO'
$ git log --oneline
a6484fd Añade notas con un TODO
398526b Crea el proyecto
```

### `commit-msg`: exigir un formato de mensaje

Este hook recibe en `$1` **la ruta del archivo con el mensaje** y comprueba, con una expresión regular, que **empieza por un tipo** (`feat:`, `fix:`...), es decir, que sigue los *Conventional Commits* de la [unidad 4](../../u04/commits.md):

```console
$ cat > .git/hooks/commit-msg <<'EOF'
#!/bin/sh
# El mensaje debe empezar por un tipo: feat, fix, docs, refactor, test o chore
if ! grep -qE '^(feat|fix|docs|refactor|test|chore)(\(.+\))?: .+' "$1"; then
  echo "Mensaje no válido. Usa  tipo: descripción  (feat, fix, docs, refactor, test, chore)."
  exit 1
fi
EOF
chmod +x .git/hooks/commit-msg
$ git commit --allow-empty -m 'cambios'
Mensaje no válido. Usa  tipo: descripción  (feat, fix, docs, refactor, test, chore).
$ git commit --allow-empty -m 'feat: añade la lista'
[main 9a4c7f1] feat: añade la lista
$ git log --format=%s
feat: añade la lista
Crea el proyecto
```

Un mensaje **como `cambios`** se rechaza con una explicación y **uno con formato** pasa.

### Los hooks no viajan con el repositorio

Una limitación importante: la carpeta `.git/` **no se sube** con `push` ni se copia con `clone`. Tus hooks **solo existen en tu equipo**: **cada persona del equipo tendría que instalar los suyos**. La solución es guardarlos **dentro del repositorio** (por ejemplo en `.githooks/`) y decirle a Git **dónde están**, con **`core.hooksPath`**:

```console
$ mkdir .githooks && cat > .githooks/pre-commit <<'EOF'
#!/bin/sh
# Un hook compartido: está dentro del repositorio, así que viaja con él
echo "Comprobando antes de confirmar..."
EOF
chmod +x .githooks/pre-commit
$ git config core.hooksPath .githooks
$ git config --get core.hooksPath
.githooks
$ echo nuevo > nuevo.txt && git add nuevo.txt && git commit -m 'Añade un archivo'
[main 01f5d27] Añade un archivo
 1 file changed, 1 insertion(+)
 create mode 100644 nuevo.txt
Comprobando antes de confirmar...
```

El hook de `.githooks/` se ha ejecutado (`Comprobando antes de confirmar...`). Como esa carpeta **va en el repositorio**, cada persona solo tiene que ejecutar **una vez** `git config core.hooksPath .githooks` tras clonar. Para algo más serio hay herramientas que lo gestionan todo (**pre-commit**, **Husky**, **Lefthook**).

!!! warning "Los hooks son una ayuda, no una garantía"
    Cualquiera puede saltárselos (`--no-verify`) o no tenerlos instalados. Lo que **de verdad tiene que cumplirse** debe comprobarse también **en el servidor** (integración continua, [página siguiente](02-limpiar-ci.md)), que nadie puede esquivar.

## Alias: abreviar lo que repites

Un **alias** da un nombre corto a un comando largo. Se guardan en la configuración de git:

```console
$ git config alias.lg 'log --oneline --graph --decorate --all'
$ git config alias.ult 'log -1 --format=%s'
$ git lg
* 398526b (HEAD -> main) Crea el proyecto
$ git ult
Crea el proyecto
$ git config --get-regexp '^alias'
alias.lg log --oneline --graph --decorate --all
alias.ult log -1 --format=%s
```

Desde ese momento, `git lg` es el `log` con grafo y `git ult` muestra el mensaje del último commit. Algunos alias muy usados:

| Alias | Comando | Para qué |
|---|---|---|
| `git st` | `status -sb` | Estado corto con la rama |
| `git lg` | `log --oneline --graph --decorate --all` | Ver el árbol de ramas |
| `git co` | `switch` | Cambiar de rama |
| `git undo` | `reset --soft HEAD~1` | Deshacer el último commit conservando los cambios |
| `git last` | `log -1 HEAD --stat` | Ver el último commit |

## Los tres niveles de configuración

Git guarda su configuración en **tres sitios**, de **más general a más específico**; **el más específico gana**:

| Nivel | Opción | Dónde se guarda | Afecta a |
|---|---|---|---|
| **Sistema** | `--system` | Un archivo de la instalación de Git | **Todas las personas** del ordenador |
| **Global** | `--global` | `~/.gitconfig` (tu carpeta personal) | **Todos tus repositorios** |
| **Local** | `--local` (por defecto) | `.git/config` del repositorio | **Solo ese repositorio** |

Por eso `user.name` y `user.email` se ponen **globales** (los mismos siempre), pero puedes **cambiarlos en un repositorio concreto** (por ejemplo, tu correo del trabajo) con `--local`. Se ve **de qué nivel viene** cada valor con `--show-scope`:

```console
$ git config --global user.name 'Ana Ejemplo'
$ git config --local user.name 'Ana (trabajo)'
$ git config --local alias.st 'status -sb'
$ git config --show-scope --get-regexp '^(user|alias)'
global	user.name Ana Ejemplo
local	user.name Ana (trabajo)
local	alias.st status -sb
$ git config user.name
Ana (trabajo)
```

Hay dos valores para `user.name`, uno global y otro local, y **gana el local**: `git config user.name` responde `Ana (trabajo)`.

## `.gitattributes`: reglas por tipo de archivo

El archivo **`.gitattributes`** le dice a Git **cómo tratar ciertos archivos**. El uso más habitual es el de los **saltos de línea**: Windows termina cada línea con dos caracteres (`CRLF`) y Linux y Mac con uno (`LF`), y en un equipo mixto eso provoca **diferencias fantasma** y archivos «modificados» que no lo están. Para ver cómo ha guardado Git un archivo, `git ls-files --eol` (**`i/`** = lo que hay en el índice, **`w/`** = lo que hay en tu carpeta):

```console
$ printf 'línea\r\n' > windows.txt
$ git add windows.txt
$ git ls-files --eol windows.txt
i/crlf  w/crlf  attr/                 	windows.txt
$ echo '*.txt text eol=lf' > .gitattributes
$ git add --renormalize windows.txt
$ git ls-files --eol windows.txt
i/lf    w/crlf  attr/text eol=lf      	windows.txt
```

Al principio, el archivo se guardó con `CRLF` (`i/crlf`). Con la regla `*.txt text eol=lf`, **al normalizarlo** se guarda con `LF` (`i/lf`) y se **recuerda la regla** (`attr/text eol=lf`), de forma que **todo el equipo** guarda lo mismo, sea cual sea su sistema.

| Regla de ejemplo | Qué hace |
|---|---|
| `* text=auto` | Git decide qué archivos son de texto y **normaliza** sus saltos de línea |
| `*.sh text eol=lf` | Los `.sh` **siempre con `LF`** (con `CRLF` no se ejecutan en Linux) |
| `*.bat text eol=crlf` | Los `.bat` **siempre con `CRLF`** |
| `*.png binary` | Son **binarios**: no se tocan ni se intentan mostrar como texto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Un hook sin permiso de ejecución | `chmod +x .git/hooks/pre-commit`: sin eso, Git lo **ignora** |
| Un `pre-commit` lento | Que solo compruebe lo **rápido**; lo pesado, en `pre-push` o en el servidor |
| Dar por hecho que el equipo tiene tus hooks | `.githooks/` en el repositorio y `core.hooksPath` |
| Poner `--global` cuando querías algo solo de este proyecto | Sin opción (o `--local`) se guarda en el repositorio |
| Líneas «modificadas» en todo el archivo al cambiar de sistema | Un `.gitattributes` con `* text=auto` |

## Para practicar

Los ejercicios GA5.1 y GA5.2 de [GA5 · Ejercicios](ejercicios.md) piden un hook `commit-msg` y un alias.
