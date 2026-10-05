# 4.1 Commits profesionales

El historial es documentación: dentro de seis meses (o para un compañero) debe contar **qué se cambió y por qué**.

## Reglas de un buen commit

* **Pequeño y con un solo objetivo.** Un commit = un cambio lógico.
* **Que funcione.** No dejes el proyecto roto entre commits.
* **Mensaje claro**, en modo imperativo y en pocas palabras.

## Estructura del mensaje

```text
tipo: resumen corto (hasta ~50 caracteres)

Explicación opcional de qué y por qué (no del cómo),
en líneas de hasta ~72 caracteres.
```

Una convención muy extendida son los **Conventional Commits**:

| Tipo | Úsalo para |
|---|---|
| `feat` | una nueva funcionalidad |
| `fix` | corregir un error |
| `docs` | cambios en documentación |
| `style` | formato, sin cambiar el comportamiento |
| `refactor` | reorganizar código sin cambiar lo que hace |
| `test` | añadir o corregir pruebas |
| `chore` | tareas de mantenimiento |

Ejemplos:

```text
feat: añade validación de edad en el formulario
fix: corrige la división por cero en la media
docs: explica cómo ejecutar los tests
```

!!! warning "Mensajes que no ayudan"
    `arreglos`, `cambios`, `asdf`, `final2`… no dicen nada. Evítalos.

## Ramas con nombre

Usa nombres que digan el propósito: `feat/formulario-login`, `fix/redondeo-notas`.

## Qué no debe entrar

Archivos generados, dependencias, configuración personal del IDE y **secretos** (ver `.gitignore`).
