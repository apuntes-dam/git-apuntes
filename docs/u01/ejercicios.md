# G1 · Ejercicios de primeros pasos

<div class="ej-gate" data-unit="u01" data-nombre="U1 · Primeros pasos con Git"></div>

Haz los ejercicios en una carpeta de pruebas, con datos de ejemplo. No incluyas contraseñas ni datos personales.

## Ejercicio G1.1 · Identidad

Configura tu nombre, tu correo y la rama principal por defecto. Comprueba la configuración con `git config --global --list` y copia la salida (sin mostrar nada privado).

## Ejercicio G1.2 · Primer repositorio

Crea una carpeta `mi-proyecto`, inicialízala con Git y crea dos archivos (`notas.txt` y `lista.txt`). Haz **dos commits**: uno por archivo, con mensajes que expliquen el cambio. Muestra `git log --oneline`.

*Debes ver:* dos commits, cada uno con un mensaje claro.

## Ejercicio G1.3 · `.gitignore`

Crea una carpeta `build/` con un archivo dentro y un archivo `secreto.env`. Añade un `.gitignore` para que Git los ignore y comprueba con `git status` que no aparecen. Confirma el `.gitignore`.

## Ejercicio G1.4 · Ver y descartar cambios

Modifica `notas.txt`. Observa el cambio con `git diff`, prepáralo con `git add` y compruébalo con `git diff --cached`. Después quítalo del staging con `git restore --staged notas.txt` y descártalo con `git restore notas.txt`. Explica la diferencia entre ambos comandos.

## Ejercicio G1.5 · Deshacer con `revert`

Haz un commit con un cambio erróneo. Deshazlo con `git revert` y comprueba en `git log --oneline` que el historial **no se reescribe**: aparece un commit nuevo que invierte el anterior.

## Ejercicio G1.6 · Las tres áreas

Dibuja (a mano o con texto) las tres áreas de Git e indica en cuál está un archivo tras: crearlo, `git add`, `git commit` y volver a editarlo.

## Ejercicio G1.7 · Reto: historial legible

Haz cinco commits pequeños sobre un mismo proyecto (por ejemplo, una página con tres secciones) y muestra el historial con `git log --oneline --graph --all`. Reescribe, con `--amend`, el mensaje del último commit.
