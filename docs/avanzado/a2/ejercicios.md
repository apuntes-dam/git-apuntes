# GA2 · Ejercicios de rebase interactivo

<div class="ej-gate" data-unit="a2" data-nombre="A2 · Rebase interactivo y limpiar la historia"></div>

Cada ejercicio parte de un repositorio que se prepara con unos comandos. Hazlos en una carpeta de pruebas, **nunca en un proyecto real**. Al final se comprueba con un comando y se muestra el resultado esperado.

## Ejercicio GA2.1 · Juntar un arreglo con su commit

En la web de ejemplo, `arreglo cabecera` es una corrección de `Añade la cabecera`. **Fúndelo** con ese commit (que no quede como commit aparte y se pierda su mensaje) y **mantén el resto del historial**. Usa `git rebase -i HEAD~5`.

Prepara el repositorio con:

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

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade contacto
wip
Añade el texto
Añade la cabecera
Crea el proyecto
```

??? tip "Pista"
    En la lista del editor, **sube** la línea de `arreglo cabecera` justo debajo de la de `Añade la cabecera` y cámbiale `pick` por `fixup`.

## Ejercicio GA2.2 · Dar nombre a un wip

El commit `wip` es el que añade `pie.html`. **Cámbiale el mensaje** a `Añade el pie de página` con un rebase interactivo, sin tocar nada más.

Prepara el repositorio con:

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

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade contacto
Añade el pie de página
arreglo cabecera
Añade el texto
Añade la cabecera
Crea el proyecto
```

??? tip "Pista"
    `git rebase -i HEAD~2` y `reword` en la línea de `wip`.

## Ejercicio GA2.3 · Un solo commit para el final

`wip` y `Añade contacto` son una sola idea. **Júntalos** en un único commit con el mensaje `Añade pie y contacto`, con `squash`.

Prepara el repositorio con:

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

Al terminar, ejecuta `git log --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Añade pie y contacto
arreglo cabecera
Añade el texto
Añade la cabecera
Crea el proyecto
```

??? tip "Pista"
    La segunda línea de la lista (`Añade contacto`) pasa de `pick` a `squash`.

## Ejercicio GA2.4 · Trasplantar una rama con --onto

La rama `funcion` se creó **desde `experimento`** y arrastra su commit. Haz que `funcion` salga **directamente de `main`** y lleve **solo su propio commit** (`Función nueva`). Después, `git log --format=%s main..funcion` debe mostrar un único commit.

Prepara el repositorio con:

```bash
mkdir web && cd web
git init
echo '# Mi web' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo '<h1>Hola</h1>' > header.html && git add header.html && git commit -m 'Añade la cabecera'
echo '<p>Bienvenida</p>' > texto.html && git add texto.html && git commit -m 'Añade el texto'
sed -i 's/Hola/Hola, mundo/' header.html && git commit -am 'arreglo cabecera'
echo '<footer>2025</footer>' > pie.html && git add pie.html && git commit -m 'wip'
echo '<p>Contacto</p>' > contacto.html && git add contacto.html && git commit -m 'Añade contacto'
git switch -c experimento
echo exp > experimento.txt && git add experimento.txt && git commit -m 'Prueba de experimento'
git switch -c funcion
echo f > funcion.txt && git add funcion.txt && git commit -m 'Función nueva'
```

Al terminar, ejecuta `git log --format=%s main..funcion`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Función nueva
```

??? tip "Pista"
    `git rebase --onto main experimento funcion`: «los commits de `funcion` que no están en `experimento`, encima de `main`».

## Ejercicio GA2.5 · Resolver un conflicto en un rebase

`rama` y `main` han cambiado **la misma línea** de `nota.txt`. Pon `rama` encima de `main` con `git rebase main`, **resuelve el conflicto** dejando en `nota.txt` la línea `versión de las dos` y **termina el rebase**. Al final, `cat nota.txt` debe mostrar esa línea y el historial debe ser lineal.

Prepara el repositorio con:

```bash
mkdir web && cd web
git init
echo '# Mi web' > README.md && git add README.md && git commit -m 'Crea el proyecto'
echo '<h1>Hola</h1>' > header.html && git add header.html && git commit -m 'Añade la cabecera'
echo '<p>Bienvenida</p>' > texto.html && git add texto.html && git commit -m 'Añade el texto'
sed -i 's/Hola/Hola, mundo/' header.html && git commit -am 'arreglo cabecera'
echo '<footer>2025</footer>' > pie.html && git add pie.html && git commit -m 'wip'
echo '<p>Contacto</p>' > contacto.html && git add contacto.html && git commit -m 'Añade contacto'
echo 'línea original' > nota.txt && git add nota.txt && git commit -m 'Añade la nota'
git switch -c rama
echo 'versión de la rama' > nota.txt && git commit -am 'Cambia la nota en la rama'
git switch main
echo 'versión de main' > nota.txt && git commit -am 'Cambia la nota en main'
git switch rama
```

Al terminar, ejecuta `cat nota.txt`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
versión de las dos
```

??? tip "Pista"
    Tras el conflicto: edita el archivo, `git add nota.txt` y `git rebase --continue`.
