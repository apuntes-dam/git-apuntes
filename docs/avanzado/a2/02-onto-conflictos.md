# A2.2 Autosquash, --onto y conflictos

Esta página usa la misma web de ejemplo de la [anterior](01-rebase-interactivo.md):

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

## `--autosquash`: arreglar un commit antiguo sin dolor

Descubres un error en un commit **de hace tres**. Si haces un commit normal, el arreglo queda **lejos** de lo que arregla. Con **`git commit --fixup <commit>`** creas un commit especial cuyo mensaje empieza por `fixup!` y **nombra al commit al que pertenece**. Después, **`git rebase -i --autosquash`** **coloca cada `fixup!` justo debajo de su commit y lo marca como `fixup` automáticamente**. Tú solo guardas la lista:

```console
$ sed -i 's/Bienvenida/Bienvenidos/' texto.html
$ git commit -a --fixup ':/Añade el texto'
[main 1ac833c] fixup! Añade el texto
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git log --oneline
1ac833c fixup! Añade el texto
3e3d877 Añade contacto
bdc2bff wip
40ef271 arreglo cabecera
b41a3e1 Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
$ git rebase -i --autosquash HEAD~6
# el editor se abre con:
  pick 5fe2266 # Añade la cabecera
  pick b41a3e1 # Añade el texto
  fixup 1ac833c # fixup! Añade el texto
  pick 40ef271 # arreglo cabecera
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
# y se deja así:
  pick 5fe2266 # Añade la cabecera
  pick b41a3e1 # Añade el texto
  fixup 1ac833c # fixup! Añade el texto
  pick 40ef271 # arreglo cabecera
  pick bdc2bff # wip
  pick 3e3d877 # Añade contacto
Rebasing (3/6)
Rebasing (4/6)
Rebasing (5/6)
Rebasing (6/6)
Successfully rebased and updated refs/heads/main.
$ git log --oneline
2b1e1d8 Añade contacto
1b3a872 wip
cc05aa1 arreglo cabecera
389815c Añade el texto
5fe2266 Añade la cabecera
215e952 Crea el proyecto
```

Hay dos cosas que mirar:

1. El commit `fixup! Añade el texto` **aparecía el último**, y en la lista del editor **ya está colocado debajo de `Añade el texto`**, con `fixup` puesto: Git lo ha hecho solo.
2. Al terminar, **el commit del arreglo ha desaparecido** y el texto de `texto.html` ya incluye la corrección.

`:/Añade el texto` es una forma de decir «**el commit más reciente cuyo mensaje contiene este texto**», para no copiar números. Y si quieres que `rebase -i` use siempre el autosquash, `git config --global rebase.autoSquash true`.

## `--onto`: cambiar la base de una rama

`rebase` normal pone tu rama **encima de otra**. A veces hace falta algo más fino: **trasplantar solo unos commits** a otro sitio. Imagina que has creado `funcion` **a partir de `experimento`** por error, cuando tenía que salir de `main`. Con `experimento` quieres seguir **sin llevarte** su commit:

```console
$ git switch -q -c experimento
$ echo exp > experimento.txt && git add experimento.txt && git commit -q -m 'Prueba de experimento'
$ git switch -q -c funcion
$ echo f > funcion.txt && git add funcion.txt && git commit -q -m 'Función nueva'
$ git log --oneline --graph --all --decorate
* 81ccd9a (HEAD -> funcion) Función nueva
* 2291439 (experimento) Prueba de experimento
* 3e3d877 (main) Añade contacto
* bdc2bff wip
* 40ef271 arreglo cabecera
* b41a3e1 Añade el texto
* 5fe2266 Añade la cabecera
* 215e952 Crea el proyecto
```

`funcion` está **encima de `experimento`**, y por eso arrastra su commit `Prueba de experimento`. Se corrige con:

```console
$ git switch -q -c experimento
$ echo exp > experimento.txt && git add experimento.txt && git commit -q -m 'Prueba de experimento'
$ git switch -q -c funcion
$ echo f > funcion.txt && git add funcion.txt && git commit -q -m 'Función nueva'
$ git rebase --onto main experimento funcion
Rebasing (1/1)
Successfully rebased and updated refs/heads/funcion.
$ git log --oneline --graph --all --decorate
* a4b8b41 (HEAD -> funcion) Función nueva
| * 2291439 (experimento) Prueba de experimento
|/  
* 3e3d877 (main) Añade contacto
* bdc2bff wip
* 40ef271 arreglo cabecera
* b41a3e1 Añade el texto
* 5fe2266 Añade la cabecera
* 215e952 Crea el proyecto
```

Se lee: **«los commits de `funcion` que no están en `experimento` (`experimento..funcion`), ponlos encima de `main`»**. Ahora `funcion` sale **directamente de `main`**, y `experimento` y `funcion` son ramas **hermanas**, independientes.

| Orden | Significado |
|---|---|
| `git rebase main` | Pon **toda mi rama** encima de `main` |
| `git rebase --onto A B C` | Toma los commits de **C que no están en B** y ponlos encima de **A** |

## Conflictos en medio de un rebase

Un rebase aplica tus commits **uno a uno**. Si uno **choca** con lo que hay debajo, **se detiene** y te pide que lo arregles. Se parte de dos ramas que cambian **la misma línea** de `nota.txt`:

```bash
echo 'línea original' > nota.txt && git add nota.txt && git commit -m 'Añade la nota'
git switch -c rama
echo 'versión de la rama' > nota.txt && git commit -am 'Cambia la nota en la rama'
git switch main
echo 'versión de main' > nota.txt && git commit -am 'Cambia la nota en main'
git switch rama
```

Se intenta poner `rama` encima de `main`:

```console
$ git rebase main
Auto-merging nota.txt
CONFLICT (content): Merge conflict in nota.txt
Rebasing (1/1)
error: could not apply cf1264a... Cambia la nota en la rama
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config set advice.mergeConflict false"
Could not apply cf1264a... # Cambia la nota en la rama
$ git status -s
UU nota.txt
$ cat nota.txt
<<<<<<< HEAD
versión de main
=======
versión de la rama
>>>>>>> cf1264a (Cambia la nota en la rama)
```

El rebase **se ha parado** en el commit `Cambia la nota en la rama`. `git status -s` da `UU nota.txt`: **«modificado por los dos»**. Y el archivo trae **las marcas de conflicto**: entre `<<<<<<<` y `=======` está la versión de `main` (`HEAD`, que durante un rebase es **la base sobre la que se está colocando**) y, entre `=======` y `>>>>>>>`, la de **tu commit**. Se **decide** el contenido final (aquí, una línea que reúne las dos), se **marca como resuelto** con `git add` y se **continúa**:

```console
$ git rebase main
Auto-merging nota.txt
CONFLICT (content): Merge conflict in nota.txt
Rebasing (1/1)
error: could not apply cf1264a... Cambia la nota en la rama
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config set advice.mergeConflict false"
Could not apply cf1264a... # Cambia la nota en la rama
$ echo 'versión de las dos' > nota.txt
$ git add nota.txt
$ git rebase --continue
[detached HEAD d3ac8aa] Cambia la nota en la rama
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/rama.
$ git log --oneline --graph --decorate
* d3ac8aa (HEAD -> rama) Cambia la nota en la rama
* 1f4a530 (main) Cambia la nota en main
* b5cabe1 Añade la nota
* 3e3d877 Añade contacto
* bdc2bff wip
* 40ef271 arreglo cabecera
* b41a3e1 Añade el texto
* 5fe2266 Añade la cabecera
* 215e952 Crea el proyecto
$ cat nota.txt
versión de las dos
```

El historial queda **lineal**, con el commit de la rama **encima** del de `main`, y el archivo con lo que se decidió. Si hubiera **varios commits** con conflictos, el rebase se pararía en **cada uno**.

Y si en medio del lío prefieres **no seguir**:

```console
$ git rebase main
Auto-merging nota.txt
CONFLICT (content): Merge conflict in nota.txt
Rebasing (1/1)
error: could not apply cf1264a... Cambia la nota en la rama
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config set advice.mergeConflict false"
Could not apply cf1264a... # Cambia la nota en la rama
$ git rebase --abort
$ git status -s
$ git log --oneline --graph --all --decorate
* 1f4a530 (main) Cambia la nota en main
| * cf1264a (HEAD -> rama) Cambia la nota en la rama
|/  
* b5cabe1 Añade la nota
* 3e3d877 Añade contacto
* bdc2bff wip
* 40ef271 arreglo cabecera
* b41a3e1 Añade el texto
* 5fe2266 Añade la cabecera
* 215e952 Crea el proyecto
```

`git rebase --abort` **deshace todo** y la rama queda **como estaba antes**: no ha pasado nada. (`git status -s` sale vacío.)

!!! tip "Tres comandos de un rebase parado"
    | Comando | Qué hace |
    |---|---|
    | `git rebase --continue` | Sigue, tras resolver el conflicto y hacer `git add` |
    | `git rebase --skip` | **Descarta** el commit que da el problema y sigue con el siguiente |
    | `git rebase --abort` | **Cancela** todo y deja la rama como estaba |

## `pull --rebase`: no ensuciar el historial al descargar

Cuando **otra persona ha subido algo** y tú también has hecho commits, un `git pull` normal los une con un **commit de fusión** que no aporta nada. Con **`git pull --rebase`**, tus commits se **colocan encima** de lo descargado, y el historial queda **recto**. Se ve con un servidor de prueba:

```console
$ git init -q --bare ../origen.git
$ git remote add origin ../origen.git
$ git push -q -u origin main
$ git clone -q ../origen.git ../copia
$ cd ../copia && echo otra > otra.txt && git add otra.txt && git commit -q -m 'Cambio de otra persona' && git push -q
$ echo mio > mio.txt && git add mio.txt && git commit -q -m 'Mi cambio'
$ git push
To ../origen.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to '../origen.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally. This is usually caused by another repository pushing to
hint: the same ref. If you want to integrate the remote changes, use
hint: 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
$ git pull --rebase
From ../origen
   3e3d877..7192bad  main       -> origin/main
Rebasing (1/1)
Successfully rebased and updated refs/heads/main.
$ git log --oneline --graph --decorate
* e1284ea (HEAD -> main) Mi cambio
* 7192bad (origin/main, origin/HEAD) Cambio de otra persona
* 3e3d877 Añade contacto
* bdc2bff wip
* 40ef271 arreglo cabecera
* b41a3e1 Añade el texto
* 5fe2266 Añade la cabecera
* 215e952 Crea el proyecto
$ git push -q
```

El primer `push` **se rechaza** (la otra persona ya había subido), `git pull --rebase` coloca `Mi cambio` **encima** de `Cambio de otra persona`, y el segundo `push` funciona. Sin `--rebase` habría un **commit de fusión** extra. Para hacerlo siempre así: `git config --global pull.rebase true`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Resolver un conflicto y hacer `git commit` en un rebase | Se hace **`git add`** y **`git rebase --continue`**, no `commit` |
| Confundir qué es «tuyo» y qué es «de ellos» en las marcas | En un **rebase** es al revés que en un merge: `HEAD` es la **base**, y lo de abajo, **tu commit** |
| Usar `--onto` sin entender el rango | Léelo en voz alta: «los commits de C que no están en B» |
| Seguir adelante en un rebase confuso | `git rebase --abort`: vuelves al principio sin perder nada |
| Hacer `pull --rebase` con commits **ya publicados** que otra persona tiene | Solo reescribe los que **aún no has subido** |

## Para practicar

Los ejercicios GA2.4 y GA2.5 de [GA2 · Ejercicios](ejercicios.md) practican `--onto` y los conflictos en un rebase.
