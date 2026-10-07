# GA5 · Ejercicios de automatizar y configurar

<div class="ej-gate" data-unit="a5" data-nombre="A5 · Automatizar y configurar"></div>

Cada ejercicio parte de un repositorio que se prepara con unos comandos. Hazlos en una carpeta de pruebas, **nunca en un proyecto real**. Al final se comprueba con un comando y se muestra el resultado esperado.

## Ejercicio GA5.1 · Un hook que exige formato

Crea un hook `commit-msg` que **rechace** los mensajes que no empiecen por `feat:`, `fix:` o `docs:`. Comprueba que `git commit --allow-empty -m 'cambios'` **falla**, haz después `git commit --allow-empty -m 'feat: añade la lista'` y comprueba el historial.

Prepara el repositorio con:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
feat: añade la lista
Crea el proyecto
```

??? tip "Pista"
    El hook recibe en `$1` el archivo del mensaje; `exit 1` cancela el commit. Y tiene que ser ejecutable.

## Ejercicio GA5.2 · Un alias

Crea el alias `ult` para que `git ult` muestre **solo el mensaje del último commit**. Pruébalo.

Prepara el repositorio con:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
```

Al terminar, ejecuta `git ult`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Crea el proyecto
```

??? tip "Pista"
    `git config alias.nombre 'comando'`, y el comando es `log -1 --format=%s`.

## Ejercicio GA5.3 · Dejar de seguir un archivo

Por error se subió `.env` junto a `app.txt`. **Deja de seguir `.env`** (sin borrarlo de tu carpeta), **ignóralo** para que no vuelva a entrar y haz un commit. Después, `git ls-files` debe mostrar solo `.gitignore` y `app.txt`.

Prepara el repositorio con:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo 'sesion=1' > .env && echo 'app' > app.txt && git add .env app.txt && git commit -m 'Añade la app y la configuración local'
```

Al terminar, ejecuta `git ls-files`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
.gitignore
README.md
app.txt
```

??? tip "Pista"
    `git rm --cached` saca el archivo del seguimiento; luego, `.gitignore`.

## Ejercicio GA5.4 · Limpiar lo que sobra

Hay un archivo sin seguimiento (`temporal.tmp`) y una carpeta con otro (`carpeta/otra.tmp`). **Primero comprueba qué se borraría** y después **bórralo todo**. Al final, `ls` debe mostrar solo `README.md`.

Prepara el repositorio con:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo basura > temporal.tmp
mkdir carpeta && echo otra > carpeta/otra.tmp
```

Al terminar, ejecuta `ls`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
README.md
```

??? tip "Pista"
    Primero `git clean -nd` (en seco) y luego `git clean -fd`.

## Ejercicio GA5.5 · Exportar sin lo ignorado

El proyecto tiene `README.md`, `datos.txt` y un `.env` **ignorado**. **Exporta** el último commit como un archivo `tar` y **lista su contenido** en la misma orden: debe salir sin `.env` y sin la carpeta `.git`.

Prepara el repositorio con:

```bash
mkdir proyecto && cd proyecto
git init
echo '# Proyecto' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo '.env' > .gitignore && echo 'secreto' > .env && echo 'datos' > datos.txt
git add .gitignore datos.txt && git commit -m 'Añade datos'
```

Al terminar, ejecuta `git archive --format=tar HEAD | tar -t`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
.gitignore
README.md
datos.txt
```

??? tip "Pista"
    `git archive --format=tar HEAD` escribe el archivo en la salida; `tar -t` lista su contenido.
