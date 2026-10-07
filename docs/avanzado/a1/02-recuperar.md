# A1.2 Recuperar y mover commits

Esta página usa el mismo repositorio de prueba de la [anterior](01-deshacer.md) (tres commits con una lista de la compra):

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
```

## `reflog`: el diario de lo que has hecho

Git guarda un **diario** de **todas las posiciones por las que ha pasado `HEAD`**: cada commit, cada cambio de rama, cada `reset`. Se ve con **`git reflog`**. Es la red de seguridad de Git: aunque un commit **ya no salga en `git log`**, mientras no se haya limpiado, **sigue existiendo**.

Para comprobarlo, se destruyen dos commits **a propósito** con `reset --hard` y se recuperan:

```console
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
$ git reset --hard HEAD~2
HEAD is now at d373f4a Crea el proyecto
$ git log --oneline
d373f4a Crea el proyecto
$ git reflog
d373f4a HEAD@{0}: reset: moving to HEAD~2
c7355aa HEAD@{1}: commit: Añade leche
c510b48 HEAD@{2}: commit: Añade la lista
d373f4a HEAD@{3}: commit (initial): Crea el proyecto
$ git reset --hard HEAD@{1}
HEAD is now at c7355aa Añade leche
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
```

Qué ha pasado:

1. El `reset --hard HEAD~2` **hace desaparecer** de `git log` los dos últimos commits (`Añade la lista` y `Añade leche`).
2. Pero **`git reflog`** los recuerda: cada línea es un movimiento de `HEAD`. La primera es el `reset` que acabas de hacer; la segunda, el último commit antes de él.
3. `HEAD@{1}` significa **«donde estaba `HEAD` hace un movimiento»**. Hacer `reset --hard HEAD@{1}` te lleva **de vuelta** al estado anterior al desastre.

!!! tip "El reflog es local y caduca"
    El reflog es **solo tuyo** (no se sube con `push`) y las entradas se borran **a los 90 días**, aproximadamente. Sirve para recuperar errores **recientes**, no para guardar historia. Y solo recupera **commits**: lo que nunca llegó a un commit, no está.

## `cherry-pick`: copiar un commit a otra rama

A veces necesitas **un solo commit** de otra rama, no todo lo que hay en ella: por ejemplo, un arreglo urgente que se hizo en una rama de pruebas. **`git cherry-pick <commit>`** **copia** ese commit sobre la rama en la que estás.

```console
$ git switch -q -c arreglo
$ echo 'huevos' >> lista.txt && git commit -qam 'Añade huevos'
$ echo 'queso' >> lista.txt && git commit -qam 'Añade queso'
$ git switch -q main
$ git log --oneline
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
$ git cherry-pick arreglo~1
[main 1c3e828] Añade huevos
 Date: Sat Mar 1 10:05:00 2025 +0000
 1 file changed, 1 insertion(+)
$ git log --oneline
1c3e828 Añade huevos
c7355aa Añade leche
c510b48 Añade la lista
d373f4a Crea el proyecto
$ cat lista.txt
pan
leche
huevos
```

En la rama `arreglo` hay dos commits (`Añade huevos` y `Añade queso`) y se copia **solo el penúltimo** (`arreglo~1`) a `main`. En `main` aparece un commit `Añade huevos` con un **número distinto**: es una **copia**, no el mismo commit. `lista.txt` tiene `huevos`, pero **no `queso`**.

Se puede dejar constancia de **de dónde** viene la copia con la opción `-x`, que añade al mensaje una línea con el original:

```console
$ git switch -q -c arreglo
$ echo 'huevos' >> lista.txt && git commit -qam 'Añade huevos'
$ git switch -q main
$ git cherry-pick -x arreglo
[main c64f220] Añade huevos
 Date: Sat Mar 1 10:05:00 2025 +0000
 1 file changed, 1 insertion(+)
$ git log -1 --format=%B
Añade huevos

(cherry picked from commit 1a6a70f53a8ec3a4009e450fcf3e45e70629cf01)
```

!!! warning "Cuidado con duplicar"
    Una copia con `cherry-pick` **no sabe que es una copia**. Si después **fusionas** la rama original, Git verá los mismos cambios dos veces (con commits distintos) y, en el peor caso, dará **conflictos** absurdos. Úsalo para llevar **un arreglo suelto**, no como sustituto de fusionar una rama.

## Publicar una historia que has cambiado

Todo lo anterior (`amend`, `reset`, `rebase`) **cambia los números de los commits**. Si esos commits **ya estaban en el servidor**, un `git push` normal **se rechaza**: Git ve que tu historia **ya no continúa** la que hay allí. Se ve con un servidor de prueba (un repositorio `--bare` en una carpeta, que hace de GitHub):

```console
$ git init -q --bare ../origen.git
$ git remote add origin ../origen.git
$ git push -q -u origin main
$ git commit --amend -q -m 'Añade leche (corregido)'
$ git push
To ../origen.git
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to '../origen.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

El `push` normal **se rechaza** (`non-fast-forward`). Si de verdad quieres sustituir lo que hay en el servidor, hay que **forzar**. Pero **nunca con `--force` a secas**, sino con **`--force-with-lease`**:

```console
$ git init -q --bare ../origen.git
$ git remote add origin ../origen.git
$ git push -q -u origin main
$ git commit --amend -q -m 'Añade leche (corregido)'
$ git push --force-with-lease
To ../origen.git
 + c7355aa...e581454 main -> main (forced update)
$ git log --oneline
e581454 Añade leche (corregido)
c510b48 Añade la lista
d373f4a Crea el proyecto
```

`--force-with-lease` significa: **«fuerza, pero solo si el servidor sigue como yo lo conocía»**. Es mucho más seguro que `--force`, que **machaca lo que haya**, sea lo que sea. La diferencia se ve cuando **otra persona ha subido algo mientras tanto**:

```console
$ git init -q --bare ../origen.git
$ git remote add origin ../origen.git
$ git push -q -u origin main
$ git clone -q ../origen.git ../copia
$ cd ../copia && echo 'pan integral' >> lista.txt && git commit -qam 'Cambio de otra persona' && git push -q
$ git commit --amend -q -m 'Añade leche (corregido)'
$ git push --force-with-lease
To ../origen.git
 ! [rejected]        main -> main (stale info)
error: failed to push some refs to '../origen.git'
```

La otra persona ha subido un commit que **tú todavía no habías descargado**. Con `--force-with-lease`, Git lo detecta (`stale info`, información desfasada) y **se niega**: un `--force` normal habría **borrado su trabajo sin avisar**. Lo correcto ahora es traer lo suyo (`git pull --rebase`) y volver a intentarlo.

!!! warning "La regla de oro"
    **No reescribas historia que otras personas ya tienen.** `amend`, `reset` y `rebase` son seguros en **tu rama local**, antes de subirla. Una vez publicada, si hay que deshacer algo, usa **`revert`**. Y si en un equipo hace falta reescribir una rama compartida, **avisa** antes.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Entrar en pánico tras un `reset --hard` | Mira `git reflog`: casi siempre el commit sigue ahí |
| `git push --force` | Usa **`--force-with-lease`** |
| Reescribir una rama que otros usan | Avisa, o usa `revert` |
| `cherry-pick` de varios commits de una rama y luego fusionar la rama | Genera commits duplicados: elige una de las dos formas |
| Buscar en `reflog` algo que nunca se confirmó | `reflog` solo recuerda **commits**; lo que no estaba en un commit, no |

## Para practicar

Los ejercicios GA1.4 y GA1.5 de [GA1 · Ejercicios](ejercicios.md) practican `reflog` y `cherry-pick`.
