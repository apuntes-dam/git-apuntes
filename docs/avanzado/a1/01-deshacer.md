# A1.1 Deshacer: amend, reset y revert

!!! info "Para quién es esto"
    Es material **avanzado**. Da por sabido hacer commits y mirar el historial de la [unidad 1](../../u01/index.md). Todas las sesiones de esta página son **reales**: se han ejecutado con git en un repositorio de prueba. Los **números de commit** (como `3f3a1c5`) serán **distintos en tu equipo**, porque dependen de la fecha y de quién hace el commit.

## Un repositorio de prueba

Todos los ejemplos parten de un repositorio con **tres commits** (una lista de la compra). Para reproducirlos:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
```

## La idea clave: un commit no se «edita»

Un commit es **inmutable**: su número (el *hash*) se calcula a partir de su contenido, su autor, su fecha y **el commit anterior**. Por eso, «corregir» un commit en realidad **crea otro nuevo** y deja el viejo sin usar. Casi todo lo de esta página consiste en **elegir qué commits quedan** en una rama, y no cambia las **copias de los demás**.

## `commit --amend`: enmendar el último commit

El caso más común: el mensaje del último commit tiene **una errata**, o **se te olvidó un archivo**. `--amend` **sustituye** el último commit por uno nuevo.

**Una errata en el mensaje:**

```console
$ echo huevos >> lista.txt
$ git commit -am 'Añade hueos'
[main bcb01de] Añade hueos
 1 file changed, 1 insertion(+)
$ git log --oneline
bcb01de Añade hueos
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
$ git commit --amend -m 'Añade huevos'
[main 0948520] Añade huevos
 Date: Sat Mar 1 10:05:00 2025 +0000
 1 file changed, 1 insertion(+)
$ git log --oneline
0948520 Añade huevos
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
```

Fíjate en que el commit **cambia de número** (`Añade hueos` y `Añade huevos` no comparten *hash*): es un commit distinto que ha **reemplazado** al anterior. El historial queda limpio, con un solo commit.

**Un archivo olvidado:** se añade al área de preparación y se enmienda **sin tocar el mensaje** (`--no-edit`):

```console
$ echo 'Notas de la tienda' > notas.txt
$ git commit --amend --no-edit
[main 2d3e197] Añade leche
 Date: Sat Mar 1 10:03:00 2025 +0000
 1 file changed, 1 insertion(+)
$ git status -s
?? notas.txt
```

!!! warning "Solo con commits que no has publicado"
    Enmendar **cambia el número del commit**. Si ya lo habías subido (`push`) y otra persona lo había descargado, ahora tenéis **dos versiones distintas** del mismo commit y los conflictos están asegurados. Se ve con detalle en la [página siguiente](02-recuperar.md).

## `reset`: mover la rama hacia atrás

`git reset <commit>` **mueve la rama** a otro commit (normalmente uno anterior), y **qué pasa con tus cambios** depende de una opción. Son **tres modos** y conviene distinguirlos bien. Se parte siempre del mismo estado: tres commits y se hace un cuarto (`Añade huevos`).

| Modo | Mueve la rama | Área de preparación (*staging*) | Archivos de tu carpeta |
|---|---|---|---|
| `--soft` | **Sí** | Se **conserva** (los cambios siguen preparados) | Se **conservan** |
| `--mixed` (el de por defecto) | **Sí** | Se **vacía** (los cambios dejan de estar preparados) | Se **conservan** |
| `--hard` | **Sí** | Se **vacía** | Se **pierden**: vuelven al estado de ese commit |

Un cuarto commit, y después se deshace con cada modo.

**`--soft`**: el commit desaparece, pero **sus cambios quedan preparados**, listos para volver a hacer commit (por ejemplo, con otro mensaje o juntos con otro):

```console
$ echo huevos >> lista.txt && git commit -qam 'Añade huevos'
$ git reset --soft HEAD~1
$ git status -s
M  lista.txt
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
```

La `M` en la **primera** columna significa **«preparado»**: el cambio de `lista.txt` sigue ahí, preparado.

**`--mixed`** (lo que haces si no pones nada): los cambios se conservan, pero **ya no están preparados**:

```console
$ echo huevos >> lista.txt && git commit -qam 'Añade huevos'
$ git reset HEAD~1
Unstaged changes after reset:
M	lista.txt
$ git status -s
 M lista.txt
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
```

Ahora el `M` está en la **segunda** columna: el archivo está modificado, pero **sin preparar** (hay que volver a hacer `git add`).

**`--hard`**: se pierde **todo**, también los cambios de tu carpeta:

```console
$ echo huevos >> lista.txt && git commit -qam 'Añade huevos'
$ git reset --hard HEAD~1
HEAD is now at c7355aa Añade leche
$ git status -s
$ cat lista.txt
pan
leche
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
```

El `git status -s` sale **vacío** y el archivo ha **vuelto a su contenido anterior** (sin `huevos`). Es el modo más peligroso, porque los cambios **sin commit** no se pueden recuperar. Los **commits** sí, como se verá más abajo con `reflog`.

!!! tip "`HEAD~1`"
    `HEAD` es el commit en el que estás y `HEAD~1` es **el anterior**; `HEAD~2`, el anterior a ese. Es la forma de decir «uno atrás» sin copiar números.

## `revert`: deshacer sin reescribir

`reset` **reescribe** la historia (hace como si algo nunca hubiera pasado). `git revert` hace lo contrario: **añade un commit nuevo** que **deshace los cambios** de otro. La historia **no se toca**: queda constancia del error y de su corrección.

```console
$ git revert HEAD --no-edit
[main 8c72e87] Revert "Añade leche"
 Date: Sat Mar 1 10:04:00 2025 +0000
 1 file changed, 1 deletion(-)
$ git log --oneline
8c72e87 Revert "Añade leche"
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
$ cat lista.txt
pan
```

Se ha creado un commit `Revert "Añade leche"` y `lista.txt` ha vuelto a lo que era, **sin borrar nada del historial**.

| | `reset` | `revert` |
|---|---|---|
| ¿Cambia la historia? | **Sí**: quita commits | **No**: añade uno |
| ¿Seguro con commits **ya publicados**? | **No** | **Sí** |
| ¿Cuándo? | En tu **rama local**, antes de publicar | Para deshacer algo **que ya está subido** |

## ¿Qué uso?

| Quiero... | Comando |
|---|---|
| Corregir el **mensaje** del último commit | `git commit --amend -m "..."` |
| Añadir algo **olvidado** al último commit | `git add ...` y `git commit --amend --no-edit` |
| Quitar el último commit **conservando los cambios preparados** | `git reset --soft HEAD~1` |
| Quitar el último commit **conservando los cambios sin preparar** | `git reset HEAD~1` |
| Quitar el último commit **y tirar sus cambios** | `git reset --hard HEAD~1` |
| Deshacer un commit **ya publicado** | `git revert <commit>` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `reset --hard` con cambios sin guardar | Antes, `git status`: lo que no está en un commit **no tiene vuelta atrás** |
| Enmendar un commit ya subido | Si ya está publicado, usa `revert` |
| Confundir `reset` (mueve la rama) con `restore` (recupera archivos) | `reset` es sobre **commits**; `git restore archivo` es sobre **archivos** |
| Creer que `revert` borra el commit original | Lo **deja** y añade su inverso |

## Para practicar

Los ejercicios GA1.1 a GA1.3 de [GA1 · Ejercicios](ejercicios.md) practican `amend`, `reset --soft` y `revert`.
