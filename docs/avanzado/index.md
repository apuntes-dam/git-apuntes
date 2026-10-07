# Avanzado

Material para ir más allá de las cuatro unidades. Cada unidad tiene su teoría, con **sesiones de terminal reales** (se han ejecutado con git de verdad en un repositorio de prueba), y sus ejercicios con el resultado esperado.

!!! info "Es un modo aparte"
    Esta sección solo aparece en el menú cuando activas el interruptor **Avanzado** de la cabecera. Así, quien está empezando no ve nada que no necesite todavía. Tu elección se guarda en el navegador.

| Unidad | Contenido |
|---|---|
| [A1 · Reescribir y recuperar la historia](a1/index.md) | `commit --amend`, `reset` (soft, mixed, hard), `revert`, `reflog`, `cherry-pick` y `push --force-with-lease` |
| [A2 · Rebase interactivo y limpiar la historia](a2/index.md) | `rebase -i` (`fixup`, `squash`, `reword`, `drop`, `exec`), `--autosquash`, `--onto`, conflictos en un rebase y `pull --rebase` |
| [A3 · Investigar la historia](a3/index.md) | `log` con filtros, `-S` y `-G`, `shortlog`, `..` frente a `...`, `blame`, `log -L`, `bisect` (manual y `run`) y `grep` |
| [A4 · Ramas y trabajo en equipo](a4/index.md) | Fast-forward, `--no-ff`, `--squash`, conflictos, etiquetas y SemVer, flujos de trabajo, `stash`, `worktree` y `restore` |
| [A5 · Automatizar y configurar](a5/index.md) | Hooks (`pre-commit`, `commit-msg`), alias, niveles de configuración, `.gitattributes`, `rm --cached`, `clean`, `archive`, changelog e integración continua |

!!! warning "Practica en una carpeta de pruebas"
    Muchos comandos de esta sección **reescriben la historia** o **borran cambios**. Haz los ejemplos y los ejercicios en un repositorio de prueba, **nunca en un proyecto real**.

!!! note "Antes de empezar"
    Da por sabidas las cuatro unidades de la web, sobre todo la [U4 · Flujo de trabajo profesional](../u04/index.md).
