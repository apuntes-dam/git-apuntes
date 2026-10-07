# A3.1 Preguntarle al historial

!!! info "Para quién es esto"
    Es material **avanzado**. Las sesiones son **reales** y los números de commit serán **distintos en tu equipo**.

## Una tienda con historia

Un repositorio con **ocho commits** de **dos personas** (Ana y Luis). En el sexto, alguien cambió el precio de `10` a `100`: un error que más adelante habrá que encontrar.

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

## `log` con filtros

`git log` admite muchos filtros. Los más útiles, con los resultados reales de este repositorio. **Quién hizo qué**, con un formato propio (`%h` el número corto, `%an` el autor, `%s` el mensaje):

```console
$ git log --format='%h  %an  %s'
09b973a  Ana Ejemplo  Actualiza el README
8a75cbf  Ana Ejemplo  Añade el pie
73be57a  Ana Ejemplo  Ajusta el precio
d293cb8  Luis Prueba  Cambia la moneda
25219e7  Luis Prueba  Añade descuentos
9fa8389  Ana Ejemplo  Añade la lista de productos
7b9750a  Ana Ejemplo  Añade la configuración
d373f4a  Ana Ejemplo  Crea el proyecto
```

**Solo los commits de una persona**:

```console
$ git log --oneline --author=Luis
d293cb8 Cambia la moneda
25219e7 Añade descuentos
```

**Solo los que tocan un archivo** (se pone el archivo después de `--`, que separa las opciones de las rutas):

```console
$ git log --oneline -- config.txt
73be57a Ajusta el precio
d293cb8 Cambia la moneda
7b9750a Añade la configuración
```

**Por el texto del mensaje** (`--grep`), sin distinguir mayúsculas (`-i`):

```console
$ git log --oneline --grep='añade' -i
8a75cbf Añade el pie
25219e7 Añade descuentos
9fa8389 Añade la lista de productos
7b9750a Añade la configuración
```

| Quiero ver... | Opción |
|---|---|
| Los últimos `n` commits | `git log -n 3` (o `-3`) |
| Los de una persona | `--author=nombre` |
| Los que tocan un archivo o carpeta | `-- ruta` |
| Los que mencionan un texto en el mensaje | `--grep=texto` |
| Los de un rango de fechas | `--since="2025-03-01"` y `--until="2025-03-15"` (también valen `"2 weeks ago"`) |
| Qué archivos tocó cada uno | `--stat` |
| El cambio completo de cada uno | `-p` (o `--patch`) |
| El grafo de ramas | `--graph --oneline --all --decorate` |
| Solo las fusiones, o sin ellas | `--merges` / `--no-merges` |

## ¿Cuándo apareció este texto? El «pickaxe»

El filtro más potente y menos conocido es **`-S`** (el «pico»): **muestra los commits que han cambiado el número de veces que aparece un texto**, es decir, **el commit que lo introdujo o lo eliminó**. Responde a «¿cuándo apareció `precio=100`?»:

```console
$ git log --oneline -S'precio=100'
73be57a Ajusta el precio
```

Y para ver **qué cambió exactamente**, se añade `-p` (y la ruta, para no ver otros archivos):

```console
$ git log -p -S'precio=100' -- config.txt
commit 73be57a998627c60efe696c063201fce8563e7ec
Author: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:06:00 2025 +0000

    Ajusta el precio

diff --git a/config.txt b/config.txt
index 2a31dbd..edb7a43 100644
--- a/config.txt
+++ b/config.txt
@@ -1,2 +1,2 @@
-precio=10
+precio=100
 moneda=USD
```

Hay un hermano, **`-G`**, que en lugar de contar apariciones busca **un patrón (expresión regular) en las líneas cambiadas**, útil cuando buscas «cualquier línea que cambió y contiene `precio`»:

```console
$ git log --oneline -G'^precio='
73be57a Ajusta el precio
7b9750a Añade la configuración
```

## Resumir: `shortlog`

**`git shortlog -sn`** cuenta **los commits de cada persona** (`-s` = resumen, `-n` = de más a menos). Aquí se escribe con `HEAD` al final: en una terminal normal no hace falta, pero **sin un commit y fuera de una terminal** (en un script) `shortlog` se queda **esperando datos por la entrada**:

```console
$ git shortlog -sn HEAD
     6	Ana Ejemplo
     2	Luis Prueba
```

## Ver un commit: `show`

`git show` muestra **un commit entero** (autor, fecha, mensaje y cambios). Con `--stat`, solo **los archivos** que tocó:

```console
$ git show --stat HEAD~2
commit 73be57a998627c60efe696c063201fce8563e7ec
Author: Ana Ejemplo <ana@ejemplo.com>
Date:   Sat Mar 1 10:06:00 2025 +0000

    Ajusta el precio

 config.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## Comparar: `..` frente a `...`

Entre dos ramas, **dos puntos** y **tres puntos** significan cosas distintas, y se confunden mucho. Se crea una rama `rama` desde un commit antiguo, y se le añade un commit; `main` ha seguido avanzando:

```console
$ git switch -q -c rama main~3
$ echo x > nuevo.txt && git add nuevo.txt && git commit -q -m 'Trabajo en la rama'
$ git log --oneline main..rama
427f5a6 Trabajo en la rama
$ git log --oneline rama..main
09b973a Actualiza el README
8a75cbf Añade el pie
73be57a Ajusta el precio
```

* **`main..rama`** = «los commits que **están en `rama` y no en `main`**»: aquí, solo el trabajo de la rama.
* **`rama..main`** = al revés: lo que `main` tiene y `rama` no.

Y con `diff`, los **dos puntos** comparan **las puntas** de las dos ramas, mientras que los **tres puntos** comparan **la rama contra el punto donde se separó**, es decir, **solo lo que ha hecho la rama**:

```console
$ git switch -q -c rama main~3
$ echo x > nuevo.txt && git add nuevo.txt && git commit -q -m 'Trabajo en la rama'
$ git diff --stat main..rama
 README.md  | 1 -
 config.txt | 2 +-
 nuevo.txt  | 1 +
 pie.txt    | 1 -
 4 files changed, 2 insertions(+), 3 deletions(-)
$ git diff --stat main...rama
 nuevo.txt | 1 +
 1 file changed, 1 insertion(+)
```

Con `main..rama` aparecen **también los cambios de `main` al revés** (los archivos que `main` tiene y la rama no, como si la rama los hubiera borrado). Con `main...rama` solo sale **`nuevo.txt`**, lo que hizo la rama. Para revisar **qué aporta una rama**, casi siempre quieres **tres puntos**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar `--` antes de la ruta de un archivo | Sin él, git puede confundir el archivo con una rama |
| Buscar con `--grep` algo que está **en el código** | `--grep` mira el **mensaje**; para el código es `-S` o `-G` |
| Confundir `..` y `...` | Dos puntos: lo que está en una y no en la otra. Tres: lo de **cada lado** desde el punto común (en `diff`, lo que hizo la rama) |
| Un historial enorme sin filtrar | Combina filtros: `--author`, `--since`, `--`, `-S` |

## Para practicar

Los ejercicios GA3.1, GA3.2 y GA3.5 de [GA3 · Ejercicios](ejercicios.md) practican `--author`, el *pickaxe* y `shortlog`.
