# A2.1 Rebase interactivo

!!! info "Para quién es esto"
    Es material **avanzado**. Da por sabido `rebase` de la [unidad 4](../../u04/rebase-pr.md) y la regla de oro de la [A1](../a1/02-recuperar.md): **solo reescribas commits que no has publicado**. Las sesiones de esta página son **reales** y los números de commit serán **distintos en tu equipo**.

## Un historial con ruido

Mientras trabajas, el historial se llena de commits que **no cuentan nada** (`wip`, `arreglo`) o que **corrigen otro anterior**. Antes de enseñárselo a nadie conviene limpiarlo. Este es el repositorio de ejemplo, una web en construcción:

```bash
mkdir web && cd web
git init
echo '# Mi web' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo '<h1>Hola</h1>' > header.html && git add header.html && git commit -m 'Añade la cabecera'
echo '<p>Bienvenida</p>' > texto.html && git add texto.html && git commit -m 'Añade el texto'
sed -i 's/Hola/Hola, mundo/' header.html && git commit -am 'arreglo cabecera'
echo '<footer>2025</footer>' > pie.html && git add pie.html && git commit -m 'wip'
echo '<p>Contacto</p>' > contacto.html && git add contacto.html && git commit -m 'Añade contacto'
```

Su historial, del más reciente al más antiguo:

```console
$ git log --oneline
3e3d877 Añade contacto
bdc2bff wip
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
```

Tiene tres problemas: `arreglo cabecera` corrige un commit anterior, `wip` no dice qué es, y el orden mezcla cosas.

## El editor del rebase interactivo

`git rebase -i HEAD~5` vuelve a aplicar **los últimos 5 commits** y antes **abre un editor** con la **lista de lo que va a hacer**, un commit por línea, **del más antiguo al más reciente** (al revés que `git log`). Tú cambias **la palabra del principio** y/o **el orden de las líneas**; al guardar y cerrar, Git ejecuta el plan.

| Palabra | Qué hace con ese commit |
|---|---|
| `pick` | Lo **deja** como está |
| `reword` | Lo deja, pero te deja **cambiar su mensaje** |
| `edit` | Se **detiene** en él para que lo modifiques (cambiar archivos, partirlo en dos...) |
| `squash` | Lo **junta con el anterior** y te deja **editar el mensaje** de los dos |
| `fixup` | Lo junta con el anterior **descartando su mensaje** |
| `drop` | Lo **elimina** (también vale borrar la línea) |
| `exec` | Ejecuta un **comando** de terminal en ese punto (por ejemplo, las pruebas) |

!!! note "Cómo se ven las sesiones de esta página"
    El editor es interactivo, así que en los ejemplos se muestra **lo que se abre** (`# el editor se abre con:`) y **cómo se deja** (`# y se deja así:`). Es exactamente lo que verías en tu editor. Fíjate en que la lista **sale del más antiguo al más reciente**.

## Reordenar y juntar con `fixup`

`arreglo cabecera` corrige el commit `Añade la cabecera`. Se **sube** justo debajo y se marca como `fixup`: **desaparece su mensaje y sus cambios se funden** con los de `Añade la cabecera`.

```console
$ git rebase -i HEAD~5
# el editor se abre con:
  pick 5fe2266 # Añade la cabecera
  pick b41a3e1 # Añade el texto
  pick 40ef271 # arreglo cabecera
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
# y se deja así:
  pick 5fe2266 # Añade la cabecera
  fixup 40ef271 # arreglo cabecera
  pick b41a3e1 # Añade el texto
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
Rebasing (2/5)
Rebasing (3/5)
Rebasing (4/5)
Rebasing (5/5)
Successfully rebased and updated refs/heads/main.
$ git log --oneline
9362238 Añade contacto
bf4dfc9 wip
98b4c57 Añade el texto
bb617aa Añade la cabecera
215e952 Crea el proyecto
```

Ahora el historial tiene **un commit menos** y la cabecera ya incluye su arreglo. Mira también que **todos los commits posteriores han cambiado de número**: cada uno tiene un padre nuevo, así que **es un commit nuevo**.

## Dar un nombre a un `wip`: `reword`

`wip` no dice qué es. Se marca con `reword` y, cuando Git se detenga, se escribe el mensaje nuevo:

```console
$ git rebase -i HEAD~2
# el editor se abre con:
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
# y se deja así:
  reword bdc2bff # wip
  pick 3e3d877 # Añade contacto
[detached HEAD a4ad0cb] Añade el pie de página
 Date: Sat Mar 1 10:05:00 2025 +0000
 1 file changed, 1 insertion(+)
 create mode 100644 pie.html
Rebasing (1/2)
Rebasing (2/2)
Successfully rebased and updated refs/heads/main.
$ git log --oneline
03b5baf Añade contacto
a4ad0cb Añade el pie de página
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
```

## Juntar varios commits en uno: `squash`

`squash` junta un commit **con el anterior** y deja escribir un mensaje que sirva para los dos. Aquí se unen `wip` y `Añade contacto`, que forman una sola idea («el final de la página»):

```console
$ git rebase -i HEAD~2
# el editor se abre con:
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
# y se deja así:
  pick bdc2bff # wip
  squash 3e3d877 # Añade contacto
[detached HEAD a8368b3] Añade pie y contacto
 Date: Sat Mar 1 10:05:00 2025 +0000
 2 files changed, 2 insertions(+)
 create mode 100644 contacto.html
 create mode 100644 pie.html
Rebasing (2/2)
Successfully rebased and updated refs/heads/main.
$ git log --oneline
a8368b3 Añade pie y contacto
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
```

`squash` y `fixup` hacen lo mismo con los **cambios**; la diferencia es el **mensaje**: `squash` te lo deja **redactar** y `fixup` **descarta el del commit que se une**.

## Quitar un commit: `drop`

```console
$ git rebase -i HEAD~1
# el editor se abre con:
  pick 3e3d877 # Añade contacto
# y se deja así:
  drop 3e3d877 # Añade contacto
Successfully rebased and updated refs/heads/main.
$ git log --oneline
bdc2bff wip
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
$ ls
README.md
header.html
pie.html
texto.html
```

`Añade contacto` ha **desaparecido del historial** y, con él, su archivo (`contacto.html` ya no está en la carpeta).

## Probar cada commit: `exec`

Un buen historial es uno **donde cada commit funciona**. Con la opción `--exec` se ejecuta un comando **después de cada commit**; si **falla**, el rebase **se detiene** justo ahí. Aquí el «comando de pruebas» comprueba que **no exista** `pie.html`, y falla en el commit que lo crea:

```console
$ git rebase -i --exec 'test ! -f pie.html' HEAD~3
# el editor se abre con:
  pick 40ef271 # arreglo cabecera
  exec test ! -f pie.html
  pick bdc2bff # wip
  exec test ! -f pie.html
  pick 3e3d877 # Añade contacto
  exec test ! -f pie.html
# y se deja así:
  pick 40ef271 # arreglo cabecera
  exec test ! -f pie.html
  pick bdc2bff # wip
  exec test ! -f pie.html
  pick 3e3d877 # Añade contacto
  exec test ! -f pie.html
Rebasing (2/6)
Executing: test ! -f pie.html
Rebasing (3/6)
Rebasing (4/6)
Executing: test ! -f pie.html
warning: execution failed: test ! -f pie.html
You can fix the problem, and then run

  git rebase --continue
$ git status -sb
## HEAD (no branch)
$ git rebase --abort
$ git log --oneline
3e3d877 Añade contacto
bdc2bff wip
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
```

El rebase **se detiene en `wip`**, el commit que no pasa la comprobación. Ahí se puede **arreglar el commit** (`edit`) y seguir con `git rebase --continue`, o **cancelar** todo con `git rebase --abort`, que deja la rama **exactamente como estaba**. Sirve, sobre todo, para encontrar **qué commit rompe las pruebas** antes de subirlos.

## Cancelar siempre es posible

En cualquier momento de un rebase, `git rebase --abort` **vuelve al estado de antes de empezar**. Es el botón de seguridad: **mientras no hayas terminado, no hay nada que perder**. Y si ya terminaste y te arrepientes, `git reflog` (de la [A1](../a1/02-recuperar.md)) te devuelve a la rama como estaba.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Hacer rebase de commits **ya publicados** | Solo los que no has subido (o avisa si es una rama compartida) |
| Reordenar commits que **dependen unos de otros** | Dará conflictos: reordena solo los independientes |
| Borrar por error una línea de la lista | Una línea que falta es un commit que se **elimina** |
| Poner `squash` o `fixup` en la **primera** línea | No hay un commit anterior con el que juntarlo: da error |
| Asustarse con un conflicto en medio | `git rebase --abort` lo deshace todo |

## Para practicar

Los ejercicios GA2.1 a GA2.3 de [GA2 · Ejercicios](ejercicios.md) practican `fixup`, `reword` y `squash`.
