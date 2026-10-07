# GA1 · Ejercicios de reescribir y recuperar la historia

<div class="ej-gate" data-unit="a1" data-nombre="A1 · Reescribir y recuperar la historia"></div>

Cada ejercicio parte de un repositorio que se prepara con unos comandos. Hazlos en una carpeta de pruebas, **nunca en un proyecto real**. Al final se comprueba con un comando y se muestra el resultado esperado.

## Ejercicio GA1.1 · Corregir una errata

Se ha hecho un commit con una errata en el mensaje: `Añade huevso` en lugar de `Añade huevos`. **Corrígelo** sin crear un commit nuevo, de forma que el historial tenga **un solo** commit de huevos.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
echo huevos >> lista.txt && git commit -am 'Añade huevso'
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade huevos
Añade leche
Añade la lista
Crea el proyecto
```

??? tip "Pista"
    `git commit --amend -m "..."` sustituye el último commit.

## Ejercicio GA1.2 · Juntar dos commits con reset

Se han hecho dos commits seguidos, `Añade huevos` y `Añade queso`, que en realidad son una sola idea. **Quítalos** conservando sus cambios y haz **un único commit** llamado `Añade huevos y queso`. No uses `rebase`.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
echo huevos >> lista.txt && git commit -am 'Añade huevos'
echo queso >> lista.txt && git commit -am 'Añade queso'
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade huevos y queso
Añade leche
Añade la lista
Crea el proyecto
```

??? tip "Pista"
    `git reset --soft HEAD~2` quita dos commits dejando los cambios **preparados**.

## Ejercicio GA1.3 · Deshacer un commit ya publicado

El commit `Añade leche` **ya se ha subido** al servidor y resulta que no debía estar. **Deshazlo sin reescribir la historia**. El historial debe conservar el commit original y añadir otro que lo revierta (con el mensaje que Git propone).

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Revert "Añade leche"
Añade leche
Añade la lista
Crea el proyecto
```

??? tip "Pista"
    Un commit que ya está publicado no se borra: se **revierte**.

## Ejercicio GA1.4 · Rescatar commits perdidos

Por error se ha ejecutado `git reset --hard HEAD~2`, y `git log` ya no muestra `Añade la lista` ni `Añade leche`. **Recupéralos** y comprueba que el historial vuelve a tener los tres commits.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
git reset --hard HEAD~2
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade leche
Añade la lista
Crea el proyecto
```

??? tip "Pista"
    `git reflog` enseña dónde estaba `HEAD` antes del desastre; `HEAD@{1}` es «un movimiento atrás».

## Ejercicio GA1.5 · Copiar solo un commit

En la rama `arreglo` hay dos commits: `Añade huevos` y `Añade queso`. Estando en `main`, **trae solo `Añade huevos`** (el penúltimo de `arreglo`) y no el otro. Después, `lista.txt` en `main` debe tener `huevos` pero no `queso`.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo pan > lista.txt && git add lista.txt && git commit -m 'Añade la lista'
echo leche >> lista.txt && git commit -am 'Añade leche'
git switch -c arreglo
echo huevos >> lista.txt && git commit -am 'Añade huevos'
echo queso >> lista.txt && git commit -am 'Añade queso'
git switch main
```

Al terminar, ejecuta `cat lista.txt`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
pan
leche
huevos
```

??? tip "Pista"
    `git cherry-pick arreglo~1` copia el commit anterior al último de `arreglo`.
