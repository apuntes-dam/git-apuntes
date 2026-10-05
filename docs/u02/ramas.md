# 2.1 Ramas, fusiones y stash

Una **rama** es una línea de trabajo independiente. Permite probar algo nuevo sin tocar `main` hasta que esté listo.

```text
main:     A --- B --------- M
                 \         /
rama:             C --- D
```

## Crear y cambiar de rama

```bash
git status                       # comprueba que el árbol está limpio
git switch -c nueva-funcion      # crea la rama y se cambia a ella
git branch                       # lista las ramas (la actual lleva *)
git switch main                  # vuelve a main
```

Los archivos creados en una rama **no aparecen** en la otra hasta que las fusionas.

## Fusionar (`merge`)

Para traer a `main` el trabajo de `nueva-funcion`:

```bash
git switch main
git merge nueva-funcion
git log --oneline --graph --all
git branch -d nueva-funcion      # borra la rama ya fusionada
```

## Conflictos

Si las dos ramas cambiaron **la misma parte** de un archivo, Git no sabe cuál elegir y marca el conflicto dentro del archivo:

```text
<<<<<<< HEAD
versión de main
=======
versión de la otra rama
>>>>>>> nueva-funcion
```

Para resolverlo:

1. Abre el archivo y decide qué contenido queda. Borra las marcas `<<<<<<<`, `=======` y `>>>>>>>`.
2. `git add archivo` para marcarlo como resuelto.
3. `git commit` para terminar la fusión.

Si prefieres cancelar la fusión antes de resolverla:

```bash
git merge --abort
```

!!! tip "No borres cambios a ciegas"
    Lee las dos versiones. A veces hay que quedarse con partes de ambas.

## Guardar cambios a medias: `stash`

Si tienes cambios sin terminar y necesitas cambiar de rama o limpiar el directorio:

```bash
git stash push -m "trabajo a medias"   # guarda y deja el directorio limpio
git stash list                          # lista lo guardado
git stash apply                         # recupera los cambios (los deja guardados)
git stash pop                           # recupera y borra el guardado
```

```text
cambios sin confirmar -> git stash push -> directorio limpio -> otra tarea -> git stash pop -> sigo donde lo dejé
```
