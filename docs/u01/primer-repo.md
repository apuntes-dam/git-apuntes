# 1.3 Tu primer repositorio

## Crear el repositorio y el primer commit

```bash
mkdir mi-proyecto
cd mi-proyecto
git init
git status
```

`git init` crea la carpeta oculta `.git`. Crea un archivo `hola.txt` con cualquier editor y observa el estado:

```bash
git status
```

Verás `hola.txt` como **archivo sin seguimiento** (*untracked*). Pásalo al área de preparación y confírmalo:

```bash
git add hola.txt
git status
git commit -m "Añade el archivo de saludo"
git status
```

## El ciclo de trabajo

```text
editar archivos -> git status -> git add <archivos> -> git commit -m "mensaje" -> repetir
```

!!! tip "Revisa antes de añadir"
    `git add .` añade todo lo de la carpeta. Úsalo solo después de mirar `git status`, para no subir archivos que no querías.

## Consultar el historial

```bash
git log
git log --oneline
git log --oneline --graph --all
```

Para ver **qué cambió** sin confirmar todavía:

```bash
git diff              # cambios sin preparar
git diff --cached     # cambios ya preparados (staging)
git show <hash>       # un commit concreto
```

## Ignorar archivos: `.gitignore`

Algunos archivos no deben versionarse: carpetas de compilación, configuración del IDE, dependencias o **secretos**. Crea un archivo `.gitignore` en la raíz:

```text
# Ejemplos
.idea/
.vscode/
build/
node_modules/
__pycache__/
*.log
.env
```

Añádelo y confírmalo como cualquier otro archivo.

!!! danger "Nunca subas secretos"
    Contraseñas, claves, *tokens* y archivos `.env` no deben estar en un repositorio. Si un secreto llega a un commit, borrarlo después **no lo elimina del historial**: hay que cambiar la contraseña.

## Deshacer con seguridad

| Situación | Comando |
|---|---|
| Quitar un archivo del staging (sin borrar su contenido) | `git restore --staged archivo` |
| Descartar los cambios de un archivo sin confirmar | `git restore archivo` (**irreversible**) |
| Deshacer un commit ya hecho sin reescribir la historia | `git revert <hash>` (crea un commit inverso) |
| Mirar un commit antiguo sin mover nada | `git switch --detach <hash>` y volver con `git switch main` |

!!! warning "Comandos destructivos"
    `git reset --hard` y `git checkout .` pueden **borrar cambios sin confirmar** sin preguntar. Mientras aprendes, haz un commit antes de experimentar.

## Arreglar el último mensaje

Si el último commit **todavía no está subido**, puedes corregir su mensaje:

```bash
git commit --amend -m "Mensaje corregido"
```

No lo hagas con commits que ya hayas publicado.
