# A4.2 Flujos de trabajo, stash y worktree

Esta página usa la misma app de la [anterior](01-fusionar.md):

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

## Flujos de trabajo en equipo

Git **no impone** cómo organizar las ramas: cada equipo acuerda una **convención**. Las tres más conocidas:

| Flujo | Cómo funciona | Cuándo encaja |
|---|---|---|
| **GitHub Flow** | `main` **siempre desplegable**. Cada cambio, en **una rama corta** que se une por **Pull Request** tras revisarla | Equipos pequeños y medianos, **despliegue continuo**, web |
| **Trunk-based** | **Todos** trabajan casi directamente en `main`, con ramas de **horas** o de un día, y funciones sin acabar **escondidas** tras interruptores | Equipos con **muchas pruebas automáticas** y entregas muy frecuentes |
| **GitFlow** | Ramas fijas: `main` (producción), `develop` (integración), `feature/*`, `release/*` y `hotfix/*` | Software con **versiones numeradas** y varias **versiones en mantenimiento a la vez** (apps, librerías) |

Regla práctica: **cuanto más simple, mejor**. Para empezar, **GitHub Flow**: `main` limpio, ramas pequeñas con nombre claro (`feat/login`, `fix/precio`) y Pull Requests. **GitFlow** añade mucho trabajo y solo compensa si de verdad mantienes varias versiones.

## `stash`: guardar un trabajo a medias

Estás en medio de algo y **hay que cambiar de tarea ya**, pero **no quieres hacer un commit de algo a medias**. `git stash` **guarda los cambios** en un cajón aparte y deja la carpeta **limpia**:

```console
$ echo 'cambio a medias' >> README.md
$ git status -s
 M README.md
$ git stash push -m 'Trabajo a medias'
Saved working directory and index state On main: Trabajo a medias
$ git status -s
$ git stash list
stash@{0}: On main: Trabajo a medias
```

Ahora la carpeta está limpia, y el trabajo, **guardado con un nombre** (`-m`). Para recuperarlo, **`git stash pop`** lo **devuelve y lo borra del cajón** (`git stash apply` lo devuelve **sin borrarlo**):

```console
$ echo 'cambio a medias' >> README.md
$ git stash push -q -m 'Trabajo a medias'
$ git stash show -p
diff --git a/README.md b/README.md
index 95f541a..695d493 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 # App
+cambio a medias
$ git stash pop
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (48ee0a9fe3b235e8c41ea580d9b9ab8b72aee44d)
$ git status -s
 M README.md
$ git stash list
```

`git stash show -p` enseña **qué hay guardado** antes de recuperarlo. Después del `pop`, el cambio vuelve a estar en `README.md` y la lista de `stash` queda **vacía**.

### Los archivos nuevos no se guardan solos

Por defecto, `stash` **solo guarda los archivos que git ya conocía**. Un archivo **nuevo** (sin seguimiento) se queda en la carpeta. Para guardarlo también hace falta **`-u`**:

```console
$ echo borrador > borrador.txt
$ echo cambio >> README.md
$ git stash push -q -m 'sin -u'
$ git status -s
?? borrador.txt
$ git stash pop -q
$ git stash push -q -u -m 'con -u'
$ git status -s
```

Con el `stash` normal, **`borrador.txt` sigue ahí** (`??` = sin seguimiento). Con **`-u`**, desaparece de la carpeta junto con el resto.

## `worktree`: dos carpetas, un repositorio

`stash` sirve para cambiar de tarea **un momento**. Pero si necesitas **trabajar en dos ramas a la vez** (arreglar algo urgente **mientras** tienes una tarea larga a medias, o probar una rama sin parar la otra), hay algo mejor: **`git worktree`**. Crea **otra carpeta** con **otra rama** abierta, **compartiendo el mismo repositorio** (sin clonar otra vez):

```console
$ git worktree add -b urgente ../app-urgente
HEAD is now at 3e5728b Crea el proyecto
Preparing worktree (new branch 'urgente')
$ git worktree list
/ruta/al/proyecto    3e5728b [main]
/ruta/app-urgente 3e5728b [urgente]
$ cd ../app-urgente && echo arreglo > arreglo.txt && git add arreglo.txt && git commit -q -m 'Arreglo urgente' && git branch --show-current
urgente
$ git log --oneline --all
6179caa Arreglo urgente
2b5bd7c Añade el logout
a19be50 Añade el login
3e5728b Crea el proyecto
$ git worktree remove ../app-urgente
$ git worktree list
/ruta/al/proyecto 3e5728b [main]
```

Qué ha pasado:

1. `git worktree add -b urgente ../app-urgente` crea la rama `urgente` **y una carpeta nueva** (`app-urgente`) con ella abierta.
2. `git worktree list` muestra **las dos carpetas**, cada una con su rama.
3. Se hace el commit **en la otra carpeta** y, desde la original, `git log --all` lo ve: **es el mismo repositorio**.
4. Con `git worktree remove` se **borra** la carpeta extra (la rama `urgente` y su commit **siguen existiendo**).

!!! tip "¿Stash o worktree?"
    **`stash`** para **cambiar un momento** y volver enseguida. **`worktree`** para **trabajar en paralelo** durante un rato largo, **sin** parar la otra tarea, **sin** compilar de nuevo todo al cambiar de rama y **sin** el riesgo de olvidar un `stash`.

## `restore` y `switch`: los comandos modernos

Antes, `git checkout` servía para **cambiar de rama** y para **recuperar archivos**: dos cosas muy distintas con el mismo nombre. Hoy cada una tiene su comando:

| Quiero... | Comando moderno | Antes |
|---|---|---|
| Cambiar de rama | `git switch rama` (`-c` para crearla) | `git checkout rama` |
| **Quitar un archivo del área de preparación** (sin perder el cambio) | `git restore --staged archivo` | `git reset archivo` |
| **Descartar** los cambios de un archivo (¡sin vuelta atrás!) | `git restore archivo` | `git checkout -- archivo` |
| **Recuperar un archivo de otra versión** | `git restore --source=HEAD~2 archivo` | `git checkout HEAD~2 -- archivo` |

```console
$ echo cambio >> README.md
$ git add README.md
$ git status -s
M  README.md
$ git restore --staged README.md
$ git status -s
 M README.md
$ git restore README.md
$ git status -s
```

Se prepara un cambio (`M ` en la **primera** columna), se **desprepara** (` M` en la segunda: el cambio sigue en el archivo) y por último se **descarta** (el estado sale **vacío**: el archivo ha vuelto a como estaba).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar un `stash` y perder la pista de un trabajo | `git stash list` de vez en cuando, y pon siempre **un mensaje** con `-m` |
| Creer que `stash` guarda los archivos nuevos | Hace falta **`-u`** |
| Hacer `git restore archivo` sin pensar | **Descarta** el cambio **sin copia**: antes, `git diff` |
| Borrar la carpeta de un `worktree` a mano | Usa `git worktree remove`; si ya la borraste, `git worktree prune` |
| Ramas de larga vida | Cuanto más tiempo viva una rama, **más dolor** al integrarla: ramas pequeñas y cortas |

## Para practicar

Los ejercicios GA4.4 y GA4.5 de [GA4 · Ejercicios](ejercicios.md) practican `stash` y `worktree`.
