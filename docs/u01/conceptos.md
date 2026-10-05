# 1.1 Conceptos

**Git** es un sistema de **control de versiones**: guarda el historial de un proyecto para poder ver qué cambió, cuándo y por qué, volver a un estado anterior y trabajar varias personas sin pisarse.

!!! warning "Git no es una copia en la nube"
    Git registra estados de tu proyecto **en tu ordenador**. Subirlos a internet es otro paso, y lo hace un servicio como **GitHub** (unidad 3).

## Vocabulario básico

| Término | Significa |
|---|---|
| **Repositorio** | La carpeta del proyecto más su historial (carpeta oculta `.git`) |
| **Commit** | Una "foto" del proyecto con un mensaje que explica el cambio |
| **Rama** (*branch*) | Una línea de trabajo independiente; la principal suele llamarse `main` |
| **Remoto** (*remote*) | Una copia del repositorio en otro sitio, como GitHub |
| **HEAD** | El commit (o rama) en el que estás ahora mismo |

## Las tres áreas de trabajo

```text
Directorio de trabajo  --git add-->  Área de preparación  --git commit-->  Repositorio
 (archivos que editas)               (staging / index)                     (commits, historial)
```

* **Directorio de trabajo**: los archivos tal como los editas.
* **Área de preparación** (*staging*): lo que has elegido incluir en el **próximo** commit.
* **Repositorio**: los commits ya confirmados.

!!! tip "Por qué existe el staging"
    Te deja elegir qué cambios van en cada commit. Puedes editar tres archivos y hacer dos commits distintos, cada uno con un objetivo claro.
