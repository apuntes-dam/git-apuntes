# GA3 · Ejercicios de investigar la historia

<div class="ej-gate" data-unit="a3" data-nombre="A3 · Investigar la historia"></div>

Todos los ejercicios usan **la misma tienda de las páginas A3.1 y A3.2** (se prepara con los comandos de cada ejercicio). Hazlos en una carpeta de pruebas. Al final se comprueba con un comando y se muestra el resultado esperado.

## Ejercicio GA3.1 · Los commits de una persona

Muestra **solo los mensajes** de los commits que ha hecho **Luis**, del más reciente al más antiguo.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
printf 'precio=10\nmoneda=EUR\n' > config.txt && git add config.txt && git commit -m 'Añade la configuración'
printf 'pan\nleche\nhuevos\n' > productos.txt && git add productos.txt && git commit -m 'Añade la lista de productos'
echo 'descuento=5' > descuentos.txt && git add descuentos.txt && git commit --author='Luis Prueba <luis@ejemplo.com>' -m 'Añade descuentos'
sed -i 's/EUR/USD/' config.txt && git commit -a --author='Luis Prueba <luis@ejemplo.com>' -m 'Cambia la moneda'
sed -i 's/precio=10/precio=100/' config.txt && git commit -am 'Ajusta el precio'
echo '--- fin ---' > pie.txt && git add pie.txt && git commit -m 'Añade el pie'
echo 'Tienda de ejemplo' >> README.md && git commit -am 'Actualiza el README'
```

Al terminar, ejecuta `git log --author=Luis --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Cambia la moneda
Añade descuentos
```

??? tip "Pista"
    `--author` filtra por autor y `--format=%s` deja solo el mensaje.

## Ejercicio GA3.2 · ¿Cuándo apareció el precio roto?

El precio pasó de `10` a `100`. Averigua **qué commit introdujo el texto `precio=100`** y muestra **solo su mensaje**.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
printf 'precio=10\nmoneda=EUR\n' > config.txt && git add config.txt && git commit -m 'Añade la configuración'
printf 'pan\nleche\nhuevos\n' > productos.txt && git add productos.txt && git commit -m 'Añade la lista de productos'
echo 'descuento=5' > descuentos.txt && git add descuentos.txt && git commit --author='Luis Prueba <luis@ejemplo.com>' -m 'Añade descuentos'
sed -i 's/EUR/USD/' config.txt && git commit -a --author='Luis Prueba <luis@ejemplo.com>' -m 'Cambia la moneda'
sed -i 's/precio=10/precio=100/' config.txt && git commit -am 'Ajusta el precio'
echo '--- fin ---' > pie.txt && git add pie.txt && git commit -m 'Añade el pie'
echo 'Tienda de ejemplo' >> README.md && git commit -am 'Actualiza el README'
```

Al terminar, ejecuta `git log -S'precio=100' --format=%s`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Ajusta el precio
```

??? tip "Pista"
    La opción `-S'texto'` busca el commit que hizo aparecer o desaparecer ese texto.

## Ejercicio GA3.3 · Quién escribió una línea

Averigua **quién cambió por última vez** la línea 2 de `config.txt` (la de la moneda). Muestra solo la línea con el autor, usando `blame` en el formato `--line-porcelain` y filtrando con `grep '^author '`.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
printf 'precio=10\nmoneda=EUR\n' > config.txt && git add config.txt && git commit -m 'Añade la configuración'
printf 'pan\nleche\nhuevos\n' > productos.txt && git add productos.txt && git commit -m 'Añade la lista de productos'
echo 'descuento=5' > descuentos.txt && git add descuentos.txt && git commit --author='Luis Prueba <luis@ejemplo.com>' -m 'Añade descuentos'
sed -i 's/EUR/USD/' config.txt && git commit -a --author='Luis Prueba <luis@ejemplo.com>' -m 'Cambia la moneda'
sed -i 's/precio=10/precio=100/' config.txt && git commit -am 'Ajusta el precio'
echo '--- fin ---' > pie.txt && git add pie.txt && git commit -m 'Añade el pie'
echo 'Tienda de ejemplo' >> README.md && git commit -am 'Actualiza el README'
```

Al terminar, ejecuta `git blame --line-porcelain -L 2,2 config.txt | grep '^author '`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
author Luis Prueba
```

??? tip "Pista"
    `git blame -L 2,2` limita a la línea 2; `--line-porcelain` da los datos en líneas `autor ...`, `fecha ...`.

## Ejercicio GA3.4 · Encontrar el commit culpable con bisect

Con `git bisect run`, encuentra **el primer commit en que `config.txt` deja de contener `precio=10`** (comprueba con `grep -q 'precio=10$' config.txt`); usa `HEAD` como malo y `HEAD~7` como bueno. Termina con `git bisect reset`. Para comprobarlo, **antes** de hacer `reset`, muestra el mensaje del commit marcado como malo.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
printf 'precio=10\nmoneda=EUR\n' > config.txt && git add config.txt && git commit -m 'Añade la configuración'
printf 'pan\nleche\nhuevos\n' > productos.txt && git add productos.txt && git commit -m 'Añade la lista de productos'
echo 'descuento=5' > descuentos.txt && git add descuentos.txt && git commit --author='Luis Prueba <luis@ejemplo.com>' -m 'Añade descuentos'
sed -i 's/EUR/USD/' config.txt && git commit -a --author='Luis Prueba <luis@ejemplo.com>' -m 'Cambia la moneda'
sed -i 's/precio=10/precio=100/' config.txt && git commit -am 'Ajusta el precio'
echo '--- fin ---' > pie.txt && git add pie.txt && git commit -m 'Añade el pie'
echo 'Tienda de ejemplo' >> README.md && git commit -am 'Actualiza el README'
```

Al terminar, ejecuta `git log -1 --format=%s refs/bisect/bad`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
Ajusta el precio
```

??? tip "Pista"
    Tras el `bisect run`, la referencia `refs/bisect/bad` apunta al primer commit malo.

## Ejercicio GA3.5 · Cuántos commits ha hecho cada uno

Muestra, con una sola orden, **cuántos commits ha hecho cada persona**, de la que más a la que menos.

Prepara el repositorio con:

```bash
mkdir tienda && cd tienda
git init
echo '# Tienda' > README.md && git add README.md && git commit -m 'Crea el proyecto'
printf 'precio=10\nmoneda=EUR\n' > config.txt && git add config.txt && git commit -m 'Añade la configuración'
printf 'pan\nleche\nhuevos\n' > productos.txt && git add productos.txt && git commit -m 'Añade la lista de productos'
echo 'descuento=5' > descuentos.txt && git add descuentos.txt && git commit --author='Luis Prueba <luis@ejemplo.com>' -m 'Añade descuentos'
sed -i 's/EUR/USD/' config.txt && git commit -a --author='Luis Prueba <luis@ejemplo.com>' -m 'Cambia la moneda'
sed -i 's/precio=10/precio=100/' config.txt && git commit -am 'Ajusta el precio'
echo '--- fin ---' > pie.txt && git add pie.txt && git commit -m 'Añade el pie'
echo 'Tienda de ejemplo' >> README.md && git commit -am 'Actualiza el README'
```

Al terminar, ejecuta `git shortlog -sn HEAD`. Debe salir **exactamente** esto (si salen números de commit, los tuyos serán distintos):

```text
     6	Ana Ejemplo
     2	Luis Prueba
```

??? tip "Pista"
    `git shortlog` con las opciones `-s` (resumen) y `-n` (ordenar por número); en el ejercicio, añade `HEAD` al final.
