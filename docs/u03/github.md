# 3.1 GitHub y remotos

**GitHub** aloja repositorios Git en internet, para tener una copia de seguridad, compartir y colaborar. No es lo mismo que Git: Git es la herramienta; GitHub, un servicio.

## Conectar tu repositorio local con GitHub

1. En GitHub, crea un repositorio nuevo **vacío** (sin README ni `.gitignore`, si ya tienes el tuyo).
2. En tu carpeta local:

```bash
git remote add origin https://github.com/usuario/mi-proyecto.git
git branch -M main
git push -u origin main
```

* `origin` es el nombre habitual del remoto principal.
* `-u` enlaza tu rama local con la remota; a partir de ahora basta `git push` y `git pull`.

## Los comandos de sincronización

| Comando | Qué hace |
|---|---|
| `git clone <url>` | Copia un repositorio remoto a tu equipo |
| `git push` | Sube tus commits al remoto |
| `git fetch` | Descarga los cambios del remoto **sin** aplicarlos |
| `git pull` | Descarga **y** fusiona (`fetch` + `merge`) |
| `git remote -v` | Muestra los remotos configurados |

```text
tu equipo (local)  --git push-->  GitHub (remoto)
tu equipo (local)  <--git pull--  GitHub (remoto)
```

## HTTPS o SSH

Un remoto se puede escribir de dos formas:

```text
HTTPS:  https://github.com/usuario/mi-proyecto.git
SSH:    git@github.com:usuario/mi-proyecto.git
```

GitHub ya no admite la contraseña de la cuenta para HTTPS: pide un *token* personal. Por eso se recomienda [SSH](ssh.md), que usa un par de claves.

!!! danger "Cuida los tokens"
    Un *token* o una clave privada equivale a tu contraseña. No los pegues en el código, en el chat ni en un commit.

## Fork y colaboración

Un **fork** es tu copia de un repositorio ajeno dentro de tu cuenta. Sirve para proponer cambios sin tener permiso de escritura en el original (ver [Pull Requests](../u04/rebase-pr.md)).
