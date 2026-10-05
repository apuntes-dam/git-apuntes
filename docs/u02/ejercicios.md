# G2 · Ejercicios de ramas

<div class="ej-gate" data-unit="u02" data-nombre="U2 · Ramas"></div>

Parte del repositorio de la unidad 1 (o crea uno nuevo con un par de commits).

## Ejercicio G2.1 · Tu primera rama

Crea la rama `desarrollo`, añade un archivo y haz un commit. Vuelve a `main` y comprueba que el archivo **no** está. Vuelve a `desarrollo` y comprueba que sí.

## Ejercicio G2.2 · Fusión sin conflicto

Fusiona `desarrollo` en `main` y muestra `git log --oneline --graph --all`. Borra la rama con `git branch -d`.

## Ejercicio G2.3 · Provoca y resuelve un conflicto

Crea un archivo `saludo.txt` en `main`. Crea una rama, cambia **la misma línea** en la rama y en `main`, haz un commit en cada una e intenta fusionar. Resuelve el conflicto quedándote con una versión que combine las dos.

*Debes ver:* las marcas `<<<<<<<` dentro del archivo antes de resolver y un commit de fusión después.

## Ejercicio G2.4 · Cancelar una fusión

Repite el conflicto del ejercicio anterior, pero esta vez **cancela** la fusión con `git merge --abort`. Comprueba con `git status` que todo ha vuelto al estado anterior.

## Ejercicio G2.5 · `stash`

Modifica un archivo sin confirmar. Guarda el cambio con `git stash push -m "..."`, comprueba que el directorio queda limpio y recupera el trabajo con `git stash pop`.

## Ejercicio G2.6 · Reto: tres ramas

Crea tres ramas (`ajustes`, `estilos`, `textos`) que modifiquen **archivos distintos** y fusiónalas una a una en `main`. Después repite cambiando la misma línea en dos de ellas y resuelve el conflicto que aparezca.
