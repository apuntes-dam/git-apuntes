# A3.2 Quién, cuándo y dónde: blame, bisect y grep

Esta página usa la misma tienda de la [anterior](01-log.md):

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

## `blame`: quién escribió cada línea

**`git blame archivo`** muestra el archivo **línea a línea**, y delante de cada una, **el commit, la persona y la fecha** del último cambio que la tocó. Se limita a unas líneas con `-L` (`-L 1,2` = de la 1 a la 2):

```console
$ git blame -L 1,2 config.txt
73be57a (Ana Ejemplo 2025-03-01 10:06:00 +0000 1) precio=100
d293cb8 (Luis Prueba 2025-03-01 10:05:00 +0000 2) moneda=USD
```

Se lee así: la línea 1 (`precio=100`) es del commit más reciente que la tocó (el del precio) y **no de quien la creó**, porque el `blame` enseña **el último cambio**; la línea 2 (`moneda=USD`) es de Luis. Para ver **la historia completa de una línea** y no solo el último cambio, está `log -L`:

```console
$ git log -L1,1:config.txt --oneline
73be57a Ajusta el precio
diff --git a/config.txt b/config.txt
index 2a31dbd..edb7a43 100644
--- a/config.txt
+++ b/config.txt
@@ -1,1 +1,1 @@
-precio=10
+precio=100
7b9750a Añade la configuración
diff --git a/config.txt b/config.txt
new file mode 100644
index 0000000..c5dc659
--- /dev/null
+++ b/config.txt
@@ -0,0 +1,1 @@
+precio=10
```

Muestra **cada cambio que ha sufrido la línea 1**, con su diff: primero se creó (`precio=10`) y luego se cambió a `precio=100`.

!!! tip "Blame no es para culpar"
    El nombre viene de «culpar», pero su uso útil es **entender por qué** una línea está como está: el commit que la cambió trae un **mensaje que lo explica**. Y a veces quien lo cambió ya no recuerda por qué... pero el mensaje, sí.

## `bisect`: encontrar el commit que rompió algo

El precio de la tienda era `10` y ahora es `100`. **¿En qué commit se rompió?** Se podría mirar los ocho uno a uno, pero **con mil commits** no. **`git bisect`** hace una **búsqueda binaria**: coge el punto **del medio** entre uno **bueno** y uno **malo**, te pregunta si ahí funciona, y **descarta la mitad** cada vez. Con 1000 commits, en **unos 10 pasos** lo encuentra.

Hay que decirle un commit **malo** (el actual) y uno **bueno** (por ejemplo, el primero que sabes que funcionaba). En cada paso, git se coloca en un commit intermedio y **tú lo pruebas**:

```console
$ git bisect start
status: waiting for both 'good' and 'bad' commits
$ git bisect bad
status: waiting for 'good' commit(s), 'bad' commit known
$ git bisect good HEAD~7
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[25219e7d9bad0ebb28b93cbe4b7c9058f67793ea] Añade descuentos
$ grep -q 'precio=10$' config.txt && git bisect good || git bisect bad
Bisecting: 1 revision left to test after this (roughly 1 step)
[73be57a998627c60efe696c063201fce8563e7ec] Ajusta el precio
$ grep -q 'precio=10$' config.txt && git bisect good || git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[d293cb8723fc0105d9ef7f254ed7568e26751f80] Cambia la moneda
$ grep -q 'precio=10$' config.txt && git bisect good || git bisect bad
73be57a998627c60efe696c063201fce8563e7ec is the first 'bad' commit
commit 73be57a998627c60efe696c063201fce8563e7ec
Author: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:06:00 2025 +0000

    Ajusta el precio

 config.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git bisect reset
Previous HEAD position was d293cb8 Cambia la moneda
Switched to branch 'main'
```

En cada paso, `Bisecting: N revisions left to test` dice **cuántos commits quedan por revisar**. La «prueba» aquí es un `grep` que comprueba si `config.txt` tiene `precio=10`; si lo tiene, se marca `good`, y si no, `bad`. Al final, git dice **«<commit> is the first 'bad' commit»** (es el primer commit «malo») y enseña **el commit culpable** con su autor y su mensaje. `git bisect reset` **te devuelve** a donde estabas.

### Automático: `bisect run`

Si la prueba se puede escribir como **un comando que devuelve éxito o fracaso** (como un `grep` o un script de pruebas), **`git bisect run`** hace **todos los pasos solo**:

```console
$ git bisect start HEAD HEAD~7
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[25219e7d9bad0ebb28b93cbe4b7c9058f67793ea] Añade descuentos
$ git bisect run sh -c "grep -q 'precio=10$' config.txt"
running 'sh' '-c' 'grep -q '\''precio=10$'\'' config.txt'
Bisecting: 1 revision left to test after this (roughly 1 step)
[73be57a998627c60efe696c063201fce8563e7ec] Ajusta el precio
running 'sh' '-c' 'grep -q '\''precio=10$'\'' config.txt'
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[d293cb8723fc0105d9ef7f254ed7568e26751f80] Cambia la moneda
running 'sh' '-c' 'grep -q '\''precio=10$'\'' config.txt'
73be57a998627c60efe696c063201fce8563e7ec is the first 'bad' commit
commit 73be57a998627c60efe696c063201fce8563e7ec
Author: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:06:00 2025 +0000

    Ajusta el precio

 config.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
bisect found first 'bad' commit
$ git bisect reset
Previous HEAD position was d293cb8 Cambia la moneda
Switched to branch 'main'
```

Un único comando recorre toda la búsqueda y señala el commit. En un proyecto real, ese comando sería **tu conjunto de pruebas** (`./gradlew test`, `pytest`, `dart test`...): **una prueba que falla hoy encuentra sola el commit que la rompió**.

!!! note "Qué comando devuelve qué"
    Un comando **bueno** devuelve el código `0`; **malo**, de `1` a `127` (menos el `125`, que significa «no se puede probar este commit, sáltalo»). `grep -q` devuelve `0` si encuentra el texto y `1` si no, y por eso sirve.

## `grep`: buscar dentro del repositorio

**`git grep texto`** busca **en los archivos del proyecto**, mucho más rápido que `grep -r` porque **solo mira lo que git controla** (se salta lo que está en `.gitignore`). Con `-n` enseña el número de línea:

```console
$ git grep -n precio
config.txt:1:precio=100
```

Y se puede buscar **en una versión antigua**, sin cambiar de commit, poniendo el commit detrás:

```console
$ git grep -n precio HEAD~3
HEAD~3:config.txt:1:precio=10
```

En `HEAD~3` (tres commits atrás, antes de «romper» el precio) la línea era `precio=10`.

## Resumen: ¿qué herramienta uso?

| Pregunta | Herramienta |
|---|---|
| ¿Qué se hizo y cuándo? | `git log` con filtros |
| ¿Cuándo apareció o desapareció este texto? | `git log -S'texto'` |
| ¿Quién escribió esta línea y por qué? | `git blame -L` y `git log -L` |
| ¿Qué commit rompió algo? | `git bisect` (o `bisect run`) |
| ¿Dónde se usa este texto? | `git grep` |
| ¿Qué aporta una rama? | `git diff main...rama` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Hacer cambios en medio de un `bisect` | Mientras dura, **no toques nada**; al acabar, `git bisect reset` |
| Marcar mal un paso del `bisect` | Si dudas, `git bisect log` guarda lo hecho y `git bisect reset` empieza de nuevo |
| Creer que `blame` da quién creó la línea | Da **quién la cambió por última vez**; para el origen, `log -L` |
| Una «prueba» de `bisect run` que depende de algo fuera del commit | Debe depender **solo del código de ese commit** |
| Olvidar `git bisect reset` | Te deja en un commit antiguo, con `HEAD` suelto |

## Para practicar

Los ejercicios GA3.3 y GA3.4 de [GA3 · Ejercicios](ejercicios.md) practican `blame` y `bisect run`.
