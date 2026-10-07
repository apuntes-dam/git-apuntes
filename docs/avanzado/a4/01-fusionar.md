# A4.1 Fusionar, conflictos y etiquetas

!!! info "Para quién es esto"
    Es material **avanzado**. Las sesiones son **reales** y los números de commit serán **distintos en tu equipo**.

## Una app con una rama

Un repositorio con `main` y una rama `feature` con **dos commits** (`login` y `logout`). Para reproducirlo:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

Hay **tres formas de traer `feature` a `main`**, y dejan un historial distinto. Elegir bien importa, porque el historial **se lee durante años**.

## 1. Fast-forward: cuando `main` no se ha movido

Si `main` **no ha cambiado** desde que salió la rama, no hay nada que «mezclar»: basta con **mover `main` hacia delante**. Es un *fast-forward* y **no crea ningún commit nuevo**:

```console
$ git merge feature
Updating 3e5728b..2b5bd7c
Fast-forward
 login.txt  | 1 +
 logout.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git log --oneline --graph --decorate
* 2b5bd7c (HEAD -> main, feature) Añade el logout
* a19be50 Añade el login
* 3e5728b Crea el proyecto
```

El historial queda **recto**, como si todo se hubiera hecho en `main`. No hay forma de saber que existió una rama.

## 2. Commit de fusión: `--no-ff`

Con **`--no-ff`** (*no fast-forward*), Git **crea un commit de fusión** aunque no hiciera falta. Así **queda constancia** de que esos dos commits formaban **un grupo de trabajo**:

```console
$ git merge --no-ff feature
Merge made by the 'ort' strategy.
 login.txt  | 1 +
 logout.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git log --oneline --graph --decorate
*   e13b0b7 (HEAD -> main) Merge branch 'feature'
|\  
| * 2b5bd7c (feature) Añade el logout
| * a19be50 Añade el login
|/  
* 3e5728b Crea el proyecto
```

El grafo tiene una «burbuja»: el **commit de fusión** (`Merge branch 'feature'`) junta las dos líneas. Si mañana hay que **deshacer toda la funcionalidad**, basta con **revertir ese commit**.

Cuando `main` **sí se ha movido** (alguien más ha añadido algo), el commit de fusión **es inevitable**, con o sin `--no-ff`:

```console
$ git merge feature
Merge made by the 'ort' strategy.
 login.txt  | 1 +
 logout.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git log --oneline --graph --decorate
*   f505973 (HEAD -> main) Merge branch 'feature'
|\  
| * 2b5bd7c (feature) Añade el logout
| * a19be50 Añade el login
* | 72aaf3b Añade la ayuda
|/  
* 3e5728b Crea el proyecto
```

## 3. Squash: todo en un solo commit

Con **`--squash`**, Git toma **todos los cambios** de la rama y los deja **preparados**, **sin hacer commit ni registrar la fusión**. Tú haces **un solo commit** con el mensaje que quieras:

```console
$ git merge --squash feature
Updating 3e5728b..2b5bd7c
Fast-forward
Squash commit -- not updating HEAD
 login.txt  | 1 +
 logout.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git status -s
A  login.txt
A  logout.txt
$ git commit -m 'Añade el login y el logout'
[main c1ec6a9] Añade el login y el logout
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git log --oneline --graph --decorate
* c1ec6a9 (HEAD -> main) Añade el login y el logout
* 3e5728b Crea el proyecto
```

El historial queda **recto y con un único commit** para toda la funcionalidad: muy limpio. Pero tiene una contrapartida: **Git no sabe que `feature` se fusionó**. Se nota al intentar borrar la rama:

```console
$ git merge --squash feature
Updating 3e5728b..2b5bd7c
Fast-forward
Squash commit -- not updating HEAD
 login.txt  | 1 +
 logout.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 login.txt
 create mode 100644 logout.txt
$ git commit -q -m 'Añade el login y el logout'
$ git branch --merged
* main
$ git branch -d feature
error: the branch 'feature' is not fully merged
hint: If you are sure you want to delete it, run 'git branch -D feature'
hint: Disable this message with "git config set advice.forceDeleteBranch false"
```

`git branch --merged` **no menciona `feature`**, y `git branch -d` **se niega a borrarla** (`not fully merged`), porque para Git sus commits **no están en `main`**: lo que hay es **una copia** en un commit distinto. Si estás seguro, se fuerza con `git branch -D feature`.

## ¿Cuál elijo?

| Forma | Historial | Ventaja | Inconveniente |
|---|---|---|---|
| **Fast-forward** | Recto, sin rastro de la rama | Muy limpio | Pierdes el agrupamiento de la rama |
| **`--no-ff`** | Con commit de fusión | Se ve **qué commits eran una funcionalidad** y se puede **revertir de una vez** | Más grafo |
| **`--squash`** | Recto, un commit por funcionalidad | Muy limpio, **descarta el ruido** (`wip`, arreglos) | **Pierdes los commits intermedios** y Git no marca la rama como fusionada |
| **`rebase`** + fast-forward | Recto, con los commits **originales** | Lineal y detallado | Reescribe los commits ([A2](../a2/index.md)) |

En GitHub, los tres botones del **Pull Request** son justo estas formas: **«Create a merge commit»** (`--no-ff`), **«Squash and merge»** (`--squash`) y **«Rebase and merge»** (rebase). **El equipo debe elegir una** y usarla siempre.

## Conflictos: resolver con criterio

Dos ramas cambiaron **la misma línea** de `nota.txt`:

```bash
echo 'línea original' > nota.txt && git add nota.txt && git commit -m 'Añade la nota'
git switch -c rama
echo 'versión de la rama' > nota.txt && git commit -am 'Cambia la nota en la rama'
git switch main
echo 'versión de main' > nota.txt && git commit -am 'Cambia la nota en main'
```

```console
$ git merge rama
Auto-merging nota.txt
CONFLICT (content): Merge conflict in nota.txt
Automatic merge failed; fix conflicts and then commit the result.
$ git status -s
UU nota.txt
$ cat nota.txt
<<<<<<< HEAD
versión de main
=======
versión de la rama
>>>>>>> rama
```

`UU` significa «**modificado por los dos**». En las marcas, entre `<<<<<<< HEAD` y `=======` está la versión de **la rama en la que estás** (`main`) y, entre `=======` y `>>>>>>>`, la de **la que traes** (`rama`). Se puede resolver **a mano** (editar el archivo y dejar lo que corresponda) o **elegir un lado entero**:

| Comando | Qué se queda |
|---|---|
| `git checkout --ours archivo` | La versión de **tu** rama (la actual) |
| `git checkout --theirs archivo` | La versión de la rama que **traes** |

Aquí se elige la de la rama y se termina la fusión:

```console
$ git merge rama
Auto-merging nota.txt
CONFLICT (content): Merge conflict in nota.txt
Automatic merge failed; fix conflicts and then commit the result.
$ git checkout --theirs nota.txt
Updated 1 path from the index
$ git add nota.txt
$ git commit --no-edit
[main b613589] Merge branch 'rama'
$ cat nota.txt
versión de la rama
$ git log --oneline --graph --decorate
*   b613589 (HEAD -> main) Merge branch 'rama'
|\  
| * 5d537f2 (rama) Cambia la nota en la rama
* | eeeaf7b Cambia la nota en main
|/  
* 32b33c1 Añade la nota
* 3e5728b Crea el proyecto
```

!!! warning "`ours` y `theirs` se invierten en un rebase"
    En un **merge**, `--ours` es tu rama y `--theirs` la que traes. En un **rebase** es **al revés** ([A2.2](../a2/02-onto-conflictos.md)), porque Git va colocando **tus** commits sobre la otra rama. Ante la duda, **abre el archivo y mira las marcas**.

Y si en medio del conflicto prefieres **no seguir**, `git merge --abort` **deja todo como estaba antes** de empezar.

## Etiquetas: marcar versiones

Una **etiqueta** (*tag*) es un **nombre fijo para un commit concreto**, como `v1.0`. A diferencia de una rama, **no se mueve**. Se usan para marcar **versiones publicadas**. Hay dos tipos:

* **Ligera**: solo el nombre (`git tag v0.1`).
* **Anotada**: con **mensaje, autor y fecha** (`git tag -a v1.0 -m "..."`). **Es la que debes usar para versiones**.

```console
$ git tag v0.1
$ git tag -a v1.0 -m 'Primera versión estable'
$ git tag
v0.1
v1.0
$ git tag -n1
v0.1            Crea el proyecto
v1.0            Primera versión estable
$ git show v1.0 --stat
tag v1.0
Tagger: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:07:00 2025 +0000

Primera versión estable

commit 3e5728b93879fc65872b50ba7f0fc6bc656abc2d
Author: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:01:00 2025 +0000

    Crea el proyecto

 README.md | 1 +
 1 file changed, 1 insertion(+)
```

`git tag -n1` muestra cada etiqueta con su mensaje: la **anotada** (`v1.0`) enseña el suyo, y la **ligera** (`v0.1`), que no tiene mensaje propio, enseña **el del commit al que apunta**. Y `git show v1.0` enseña **quién la creó, cuándo y con qué mensaje**, y después el commit.

Para **saber en qué punto estás respecto a la última versión**, `git describe` dice cuántos commits hay desde la última etiqueta:

```console
$ git tag -a v1.0 -m 'Primera versión estable'
$ echo ayuda > ayuda.txt && git add ayuda.txt && git commit -q -m 'Añade la ayuda'
$ git describe --tags
v1.0-1-gb1a66cf
```

`v1.0-1-g...` significa «**1 commit después de `v1.0`**, y el commit actual empieza por `g...`» (la `g` es por *git*).

Las etiquetas **no se suben solas** con `git push`. Hay que subirlas **a propósito**. En el servidor, una etiqueta anotada aparece **en dos líneas**: la etiqueta en sí (que es un objeto propio) y, con `^{}`, **el commit al que apunta**:

```console
$ git init -q --bare ../origen.git
$ git remote add origin ../origen.git
$ git push -q origin main
$ git tag -a v1.0 -m 'Primera versión estable'
$ git push -q origin v1.0
$ git ls-remote --tags origin
df0ebc5bd86f0ac2e585d55587280448d6bac9fb	refs/tags/v1.0
3e5728b93879fc65872b50ba7f0fc6bc656abc2d	refs/tags/v1.0^{}
```

### Numerar versiones: SemVer

La convención más extendida es el **versionado semántico**: **`MAJOR.MINOR.PATCH`**, por ejemplo `2.4.1`.

| Número | Se sube cuando... | Ejemplo |
|---|---|---|
| **MAJOR** | Hay cambios que **rompen la compatibilidad** | `1.9.3` → `2.0.0` |
| **MINOR** | Se **añade una función** sin romper nada | `2.3.0` → `2.4.0` |
| **PATCH** | Se **arregla un error** sin cambiar nada más | `2.4.0` → `2.4.1` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `--squash` y luego intentar `branch -d` | Esperado: usa `-D` cuando estés seguro de que está integrada |
| Resolver un conflicto sin leer **los dos lados** | Entiende qué quería cada persona antes de elegir |
| Mezclar formas de fusionar en un equipo | Elegid **una** (y configurad GitHub para permitir solo esa) |
| Una etiqueta ligera para una versión | Usa **`-a`**: deja constancia de quién y cuándo |
| Olvidar subir las etiquetas | `git push origin v1.0` (o `git push --tags` para todas) |

## Para practicar

Los ejercicios GA4.1, GA4.2 y GA4.3 de [GA4 · Ejercicios](ejercicios.md) practican `--no-ff`, `--squash` y las etiquetas anotadas.
