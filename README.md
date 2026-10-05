# Git y GitHub

🌐 **Web: https://apuntes-dam.github.io/git-apuntes/**

Apuntes y prácticas de Git y GitHub: ramas, SSH, rebase y Pull Requests.

Forma parte de [Practica los Lenguajes de Programación para 1º DAM y 2º DAM](https://apuntes-dam.github.io/apuntes-lenguajes/).

## Cómo está hecho

Web estática con [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) publicada con GitHub Pages: el workflow `.github/workflows/pages.yml` construye y despliega al hacer push a `main`. En local:

```bash
pip install mkdocs-material
mkdocs serve
```
