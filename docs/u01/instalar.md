# 1.2 Instalar y configurar

## Instalar Git

* **Windows:** descarga el instalador de [git-scm.com](https://git-scm.com/) y deja las opciones por defecto. Incluye **Git Bash**, una terminal con comandos de Linux.
* **macOS:** `git --version` (propone instalarlo) o con Homebrew: `brew install git`.
* **Linux:** `sudo apt install git` (Debian/Ubuntu).

Comprueba la instalación:

```bash
git --version
```

## Configurar tu identidad

Cada commit lleva nombre y correo. Configúralos una vez:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@example.com"
git config --global init.defaultBranch main
git config --global --list
```

!!! note "Correo y privacidad"
    Si vas a usar GitHub, GitHub ofrece un correo anónimo (`...@users.noreply.github.com`) por si no quieres publicar el tuyo.

## Lo mínimo de la terminal

| Quiero... | Linux / macOS / Git Bash | PowerShell |
|---|---|---|
| Ver dónde estoy | `pwd` | `pwd` |
| Listar archivos | `ls -al` | `ls` / `dir` |
| Entrar en una carpeta | `cd carpeta` | `cd carpeta` |
| Crear una carpeta | `mkdir carpeta` | `mkdir carpeta` |
| Ver un archivo | `cat archivo` | `type archivo` |

!!! warning "Cuidado al borrar"
    Mientras aprendes, evita comandos que borren carpetas (`rm -rf`). Trabaja en una carpeta de pruebas, por ejemplo `Documentos/pruebas-git`.
