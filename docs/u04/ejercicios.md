# G4 · Ejercicios de flujo profesional

<div class="ej-gate" data-unit="u04" data-nombre="U4 · Flujo de trabajo profesional"></div>

## Ejercicio G4.1 · Mensajes claros

Reescribe estos mensajes malos como Conventional Commits y explica qué mejora en cada uno: `cambios`, `arreglo`, `final2`, `subo lo de ayer`.

## Ejercicio G4.2 · Commits pequeños

Parte de un cambio grande (por ejemplo, añadir una página con cabecera, contenido y pie). Divídelo en **tres commits**, uno por parte, con `git add` selectivo y mensajes `feat:` distintos.

## Ejercicio G4.3 · `rebase`

Crea una rama `feat/a` desde `main`, haz dos commits y, mientras tanto, añade un commit nuevo en `main`. Rebasa `feat/a` sobre `main` y muestra `git log --oneline --graph --all` antes y después. Explica la diferencia con un `merge`.

## Ejercicio G4.4 · Limpiar con `rebase -i`

Haz cuatro commits pequeños seguidos (el último, un arreglo de un detalle del anterior). Con `git rebase -i HEAD~4`, une el arreglo con su commit (`squash`) y renombra otro (`reword`). Muestra el historial resultante.

## Ejercicio G4.5 · Pull Request

En tu repositorio de GitHub, crea una rama, sube un cambio y abre un **Pull Request** hacia `main` con título y descripción. Fusiónalo desde la web y actualiza tu copia local con `git pull`. Reto: haz el mismo ejercicio con un *fork* de otro repositorio de pruebas.
