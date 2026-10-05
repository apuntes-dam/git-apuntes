# Chuleta de comandos

## Configuración

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@example.com"
git config --global init.defaultBranch main
```

## Día a día

| Quiero... | Comando |
|---|---|
| Crear un repositorio | `git init` |
| Ver el estado | `git status` |
| Preparar cambios | `git add archivo` · `git add .` |
| Confirmar | `git commit -m "mensaje"` |
| Ver el historial | `git log --oneline --graph --all` |
| Ver cambios | `git diff` · `git diff --cached` |
| Quitar del staging | `git restore --staged archivo` |
| Descartar cambios | `git restore archivo` |
| Deshacer un commit (seguro) | `git revert <hash>` |
| Corregir el último mensaje | `git commit --amend -m "..."` |

## Ramas

| Quiero... | Comando |
|---|---|
| Crear y cambiar | `git switch -c nombre` |
| Cambiar | `git switch nombre` |
| Listar | `git branch` |
| Fusionar | `git merge nombre` |
| Cancelar fusión | `git merge --abort` |
| Borrar (ya fusionada) | `git branch -d nombre` |
| Guardar a medias | `git stash push -m "..."` · `git stash pop` |
| Rebasar | `git rebase main` · `git rebase --continue` · `git rebase --abort` |

## Remotos

| Quiero... | Comando |
|---|---|
| Copiar un repositorio | `git clone <url>` |
| Añadir un remoto | `git remote add origin <url>` |
| Ver remotos | `git remote -v` |
| Subir | `git push` · `git push -u origin main` (primera vez) |
| Traer sin aplicar | `git fetch` |
| Traer y fusionar | `git pull` |
| Cambiar a SSH | `git remote set-url origin git@github.com:usuario/repo.git` |

## SSH

```bash
ssh-keygen -t ed25519 -C "tu-correo@example.com"
cat ~/.ssh/id_ed25519.pub      # esta es la que se copia a GitHub
ssh -T git@github.com
```
