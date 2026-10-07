# GA4 · Ejercicios de ramas y trabajo en equipo

<div class="ej-gate" data-unit="a4" data-nombre="A4 · Ramas y trabajo en equipo"></div>

Cada ejercicio parte de un repositorio que se prepara con unos comandos. Hazlos en una carpeta de pruebas, **nunca en un proyecto real**. Al final se comprueba con un comando y se muestra el resultado esperado.

## Ejercicio GA4.1 · Fusionar dejando constancia

Trae la rama `feature` a `main` **con un commit de fusión**, aunque se pudiera hacer sin él. Después, comprueba que existe un commit de fusión en el historial.

Prepara el repositorio con:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

Al terminar, ejecuta `git log --merges --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Merge branch 'feature'
```

??? tip "Pista"
    `git merge --no-ff feature`.

## Ejercicio GA4.2 · Todo en un solo commit

Trae la rama `feature` a `main` **en un único commit** llamado `Añade el acceso` (sin conservar los dos commits de la rama).

Prepara el repositorio con:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade el acceso
Crea el proyecto
```

??? tip "Pista"
    `git merge --squash feature` y después `git commit`.

## Ejercicio GA4.3 · Una etiqueta de versión

Marca el estado actual de `main` como la versión `v1.0`, con una **etiqueta anotada** cuyo mensaje sea `Primera versión`. Comprueba con `git tag -n1`.

Prepara el repositorio con:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

Al terminar, ejecuta `git tag -n1`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
v1.0            Primera versión
```

??? tip "Pista"
    `git tag -a nombre -m "mensaje"`.

## Ejercicio GA4.4 · Guardar un trabajo a medias

Hay un cambio sin terminar en `README.md` y un archivo nuevo `borrador.txt`. **Guarda los dos** con el mensaje `Trabajo a medias` para dejar la carpeta limpia, y comprueba que `git status -s` sale vacío.

Prepara el repositorio con:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
echo cambio >> README.md
echo borrador > borrador.txt
```

Al terminar, ejecuta `git status -s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text

```

??? tip "Pista"
    El archivo nuevo necesita la opción `-u`.

## Ejercicio GA4.5 · Dos carpetas, un repositorio

Crea un `worktree` en una carpeta hermana llamada `app-urgente` con una rama nueva `urgente`. Comprueba que ahora `git worktree list` muestra **dos líneas** (cuéntalas con `wc -l`).

Prepara el repositorio con:

```bash
mkdir app && cd app
git init
echo '# App' > README.md && git add README.md && git commit -m 'Crea el proyecto'
git switch -c feature
echo login > login.txt && git add login.txt && git commit -m 'Añade el login'
echo logout > logout.txt && git add logout.txt && git commit -m 'Añade el logout'
git switch main
```

Al terminar, ejecuta `git worktree list | wc -l`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
2
```

??? tip "Pista"
    `git worktree add -b rama ../carpeta`.
