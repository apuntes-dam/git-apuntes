# 3.2 Autenticación SSH

Con SSH te identificas con un **par de claves**: la **pública** se registra en GitHub y la **privada** se queda en tu equipo. La privada nunca se comparte ni se sube a un repositorio.

## 1. Comprobar si ya tienes una clave

```bash
ls -al ~/.ssh
```

Si ves un archivo `id_ed25519.pub`, ya tienes una. Si no, genera una.

## 2. Generar la clave

```bash
ssh-keygen -t ed25519 -C "tu-correo@example.com"
```

* Acepta la ruta propuesta (`~/.ssh/id_ed25519`).
* Te pide una **frase de paso** (*passphrase*) para proteger la clave privada. En un equipo compartido, ponla. En tu equipo personal puedes dejarla vacía.

## 3. El agente SSH (opcional)

Con frase de paso, el **agente** guarda la clave desbloqueada para no escribirla en cada operación:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## 4. Registrar la clave pública en GitHub

1. Muestra la pública y cópiala **entera**:

```bash
cat ~/.ssh/id_ed25519.pub
```

2. En GitHub: **Settings → SSH and GPG keys → New SSH key**, pega la clave y ponle un nombre.

!!! danger "Pública sí, privada nunca"
    Copia siempre el archivo que termina en `.pub`. El que **no** lo lleva es la clave privada.

## 5. Probar la conexión

```bash
ssh -T git@github.com
```

La primera vez pregunta si confías en el servidor; responde `yes`. Debe contestar con un saludo con tu usuario.

## 6. Cambiar el remoto de HTTPS a SSH

```bash
git remote -v
git remote set-url origin git@github.com:usuario/mi-proyecto.git
git remote -v
git push
```

## Si tu red bloquea el puerto 22

Algunas redes (aulas, empresas) bloquean el puerto 22 de SSH. GitHub admite SSH por el **puerto 443**. En `~/.ssh/config`:

```text
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
```

y vuelve a probar `ssh -T git@github.com`.
