# 4.2 Rebase y Pull Requests

## `rebase`: reescribir dónde empieza tu rama

`merge` une dos historias con un commit de fusión. `rebase` **recoloca** tus commits sobre la punta de otra rama, dejando un historial lineal.

```text
Antes:                    Después de  git rebase main  (estando en tu-rama):

main:   A - B - C         main:   A - B - C
             \                                \
tu-rama:      D - E       tu-rama:             D' - E'
```

```bash
git switch tu-rama
git rebase main
```

Si hay conflictos, resuélvelos y continúa:

```bash
git add archivo
git rebase --continue
# o cancelar:
git rebase --abort
```

!!! danger "Regla de oro del rebase"
    **No reescribas commits que otras personas ya tienen** (ya publicados en una rama compartida). `rebase` cambia los identificadores de los commits; si los habías subido, tendrías que forzar el `push` y complicarías el trabajo de los demás. Úsalo en tus ramas locales o privadas.

## `rebase -i`: limpiar tus commits

El modo interactivo te permite reordenar, unir (`squash`) o renombrar tus últimos commits antes de compartirlos:

```bash
git rebase -i HEAD~3
```

Se abre un editor con una lista; cambia `pick` por `squash` (o `reword`, `drop`) y guarda.

## Pull Request (PR)

Un **Pull Request** es una petición para integrar los cambios de una rama en otra, con revisión. Flujo habitual:

1. Crea una rama a partir de `main`: `git switch -c feat/mi-cambio`.
2. Haz tus commits y súbela: `git push -u origin feat/mi-cambio`.
3. En GitHub aparece el aviso **Compare & pull request**. Ponle título y descripción claros.
4. Otra persona revisa, comenta y aprueba.
5. Se fusiona (*merge*) y se borra la rama.

## Colaborar con un *fork*

Si el repositorio no es tuyo:

1. Pulsa **Fork** en GitHub (crea tu copia).
2. Clona **tu** fork y crea una rama.
3. Sube la rama a tu fork y abre el PR hacia el repositorio original.
4. Mantén tu fork al día añadiendo el original como remoto:

```bash
git remote add upstream https://github.com/original/proyecto.git
git fetch upstream
git switch main
git merge upstream/main
```

## Deshacer una fusión ya publicada

Para revertir un commit de **fusión** se indica qué lado conservar:

```bash
git revert -m 1 <hash-del-merge>
```
