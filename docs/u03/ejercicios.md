# G3 · Ejercicios de GitHub y SSH

<div class="ej-gate" data-unit="u03" data-nombre="U3 · GitHub y SSH"></div>

Usa un repositorio de pruebas. **No muestres ni copies tu clave privada** en ningún ejercicio.

## Ejercicio G3.1 · Subir un proyecto

Crea un repositorio vacío en GitHub y conecta tu proyecto local por HTTPS (`git remote add origin ...`). Sube `main` con `git push -u origin main` y comprueba en la web que están tus commits.

## Ejercicio G3.2 · `clone` y `pull`

Clona el repositorio en **otra carpeta**. En la copia original haz un commit y súbelo; en el clon, trae el cambio con `git pull`. Explica qué hace `fetch` frente a `pull`.

## Ejercicio G3.3 · Clave SSH

Genera una clave `ed25519` (con frase de paso si usas un equipo compartido), registra la **pública** en GitHub y comprueba la conexión con `ssh -T git@github.com`. Anota qué archivo has copiado y por qué no el otro.

## Ejercicio G3.4 · Cambiar a SSH

Cambia el remoto del ejercicio G3.1 de HTTPS a SSH con `git remote set-url`, haz un commit pequeño y súbelo con `git push`. Muestra `git remote -v` antes y después.

## Ejercicio G3.5 · Secretos fuera del repositorio

Crea un archivo `.env` con una contraseña falsa y añádelo a `.gitignore` **antes** del primer `git add`. Comprueba con `git status` que no aparece. Explica qué habría que hacer si lo hubieras subido por error.

## Ejercicio G3.6 · Reto: puerto 443

Configura `~/.ssh/config` para usar el puerto 443 de GitHub y comprueba la conexión. Explica en qué situación te hará falta.
