# Guía Completa de Comandos de Git

Esta guía detalla los comandos esenciales de Git, organizados por categorías, y explica cómo usarlos para gestionar proyectos de manera eficiente. También incluye mejores prácticas para escribir mensajes de commit basadas en la especificación de [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). La información se basa en los archivos proporcionados y ha sido verificada con fuentes confiables.

## 1. Introducción a Git

Git es un sistema de control de versiones distribuido que permite rastrear cambios en el código fuente, colaborar en equipo y mantener un historial organizado. Es ampliamente utilizado por su flexibilidad y capacidad para manejar proyectos de cualquier tamaño.

### Beneficios de Git

- **Rastreo de cambios**: Registra cada modificación en el código.
- **Colaboración**: Permite que múltiples desarrolladores trabajen simultáneamente.
- **Recuperación**: Facilita revertir cambios o recuperar versiones anteriores.
- **Ramificación**: Soporta flujos de trabajo paralelos mediante ramas.

## 2. Configuración Inicial de Git

Antes de usar Git, configura tu identidad para asociar tus commits.

| **Comando**                                             | **Descripción**                                |
| ------------------------------------------------------- | ---------------------------------------------- |
| `git config --global user.name "Tu Nombre"`             | Establece el nombre de usuario globalmente.    |
| `git config --global user.email "tu.email@example.com"` | Establece el correo electrónico globalmente.   |
| `git config --list`                                     | Muestra la configuración actual.               |
| `git version`                                           | Muestra la versión instalada de Git.           |
| `git help <comando>`                                    | Proporciona ayuda sobre un comando específico. |

## 3. Creando un Nuevo Repositorio

Para comenzar un proyecto desde cero o trabajar con uno existente:

| **Comando**       | **Descripción**                                          |
| ----------------- | -------------------------------------------------------- |
| `git init`        | Inicializa un nuevo repositorio en el directorio actual. |
| `git clone <URL>` | Clona un repositorio remoto a tu máquina local.          |

## 4. Trabajando con Archivos

Git gestiona los cambios en tres etapas: modificados, preparados (staged) y confirmados (committed).

| **Comando**                      | **Descripción**                                                   |
| -------------------------------- | ----------------------------------------------------------------- |
| `git status`                     | Muestra el estado del directorio de trabajo y el área de staging. |
| `git status -s`                  | Muestra una versión concisa del estado.                           |
| `git add <archivo>`              | Agrega un archivo específico al área de staging.                  |
| `git add .`                      | Agrega todos los cambios al área de staging.                      |
| `git add -f <archivo_o_carpeta>` | Fuerza la adición de archivos ignorados.                          |
| `git commit -m "mensaje"`        | Confirma los cambios preparados con un mensaje descriptivo.       |

## 5. Repositorios Remotos

Los repositorios remotos permiten colaborar con otros desarrolladores.

| **Comando**                   | **Descripción**                                                            |
| ----------------------------- | -------------------------------------------------------------------------- |
| `git remote add origin <URL>` | Añade un repositorio remoto con el nombre "origin".                        |
| `git remote -v`               | Muestra las URLs de los repositorios remotos.                              |
| `git push origin <rama>`      | Envía los cambios locales a la rama especificada en el repositorio remoto. |
| `git push`                    | Envía los cambios a la rama remota predeterminada.                         |
| `git push -f`                 | Fuerza el envío, sobrescribiendo cambios remotos (usar con precaución).    |
| `git pull`                    | Obtiene y fusiona cambios del repositorio remoto.                          |

## 6. Branching

Las ramas permiten trabajar en diferentes líneas de desarrollo.

| **Comando**                                     | **Descripción**                                            |
| ----------------------------------------------- | ---------------------------------------------------------- |
| `git branch`                                    | Lista todas las ramas locales.                             |
| `git branch <nombre>`                           | Crea una nueva rama sin cambiar a ella.                    |
| `git checkout -b <nombre>`                      | Crea y cambia a una nueva rama.                            |
| `git checkout <nombre>`                         | Cambia a una rama existente.                               |
| `git checkout <commit_hash>`                    | Cambia al estado de un commit específico.                  |
| `git branch -m <nombre_antiguo> <nombre_nuevo>` | Renombra una rama.                                         |
| `git merge <rama>`                              | Fusiona la rama especificada en la rama actual.            |
| `git rebase <rama_base>`                        | Reaplica los cambios de la rama actual sobre la rama base. |
| `git rebase --continue`                         | Continúa un rebase tras resolver conflictos.               |

## 7. Inspeccionando un Repositorio

Estos comandos ayudan a explorar el historial y los cambios.

| **Comando**                 | **Descripción**                                                  |
| --------------------------- | ---------------------------------------------------------------- |
| `git log`                   | Muestra el historial completo de commits.                        |
| `git log --oneline`         | Muestra el historial en una línea por commit.                    |
| `git log --oneline --graph` | Muestra el historial con un gráfico de ramas.                    |
| `git reflog`                | Muestra un registro de todas las actualizaciones de referencias. |

## 8. Deshaciendo Cambios

Git permite revertir cambios en diferentes etapas.

| **Comando**                      | **Descripción**                                                                         |
| -------------------------------- | --------------------------------------------------------------------------------------- |
| `git reset <commit_hash>`        | Restablece el repositorio a un commit, manteniendo cambios en el directorio de trabajo. |
| `git reset --hard <commit_hash>` | Restablece el repositorio a un commit, descartando todos los cambios posteriores.       |
| `git revert <commit_hash>`       | Crea un nuevo commit que deshace los cambios de un commit específico.                   |
| `git clean -fd`                  | Elimina archivos y directorios no rastreados.                                           |

## 9. Tags

Los tags marcan puntos específicos en el historial, como versiones de lanzamiento.

| **Comando**                     | **Descripción**                             |
| ------------------------------- | ------------------------------------------- |
| `git tag <nombre> -m "mensaje"` | Crea un tag con un mensaje descriptivo.     |
| `git push --tags`               | Envía todos los tags al repositorio remoto. |

## 10. Mejores Prácticas: Conventional Commits

La especificación de [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) estandariza los mensajes de commit para mejorar la legibilidad y compatibilidad con herramientas automáticas.

### Estructura

```
<tipo>(<alcanze>): <descripción corta>

<cuerpo del commit>

<descripción footer>
```

- **tipo**: Indica el tipo de cambio (e.g., `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`).
- **alcanze**: Especifica el módulo o componente afectado (opcional).
- **descripción corta**: Resumen breve del cambio.
- **cuerpo del commit**: Detalles adicionales (opcional).
- **descripción footer**: Notas como `BREAKING CHANGE` o referencias a issues (opcional).

### Tipos de Cambios

| **Tipo**   | **Descripción**                                                       |
| ---------- | --------------------------------------------------------------------- |
| `feat`     | Introduce una nueva funcionalidad (correlaciona con MINOR en SemVer). |
| `fix`      | Corrige un error (correlaciona con PATCH en SemVer).                  |
| `docs`     | Cambios en la documentación.                                          |
| `style`    | Cambios de formato o estilo (e.g., CSS, sangrías).                    |
| `refactor` | Mejoras en el código sin alterar funcionalidad.                       |
| `perf`     | Mejoras de rendimiento.                                               |
| `test`     | Añade o modifica pruebas.                                             |
| `build`    | Cambios en el sistema de compilación o dependencias.                  |
| `ci`       | Cambios en integración continua o scripts.                            |
| `chore`    | Cambios misceláneos (e.g., actualizar `.gitignore`).                  |

### Cambios de Ruptura

Los cambios que rompen compatibilidad se indican con `!` después del tipo y un footer con `BREAKING CHANGE: descripción`.

### Ejemplos

- Nueva funcionalidad:
  
  ```
  feat(login): agregar autenticación con Google
  ```
- Corrección de error:
  
  ```
  fix(api): corregir error en endpoint de usuarios
  ```
- Cambio de ruptura:
  
  ```
  feat(api)!: cambiar endpoint de autenticación
  BREAKING CHANGE: Nuevo endpoint es /auth/new
  ```

## 11. Comandos Útiles Adicionales

| **Comando**          | **Descripción**                                                        |
| -------------------- | ---------------------------------------------------------------------- |
| `git version`        | Muestra la versión de Git instalada.                                   |
| `git help <comando>` | Proporciona ayuda sobre un comando.                                    |
| `git reflog`         | Muestra un registro de referencias para recuperar commits perdidos.    |
| `git push -f`        | Fuerza un push, sobrescribiendo cambios remotos (usar con precaución). |

## Resumen

Esta guía cubre los comandos esenciales de Git para inicializar repositorios, gestionar cambios, colaborar en equipo y mantener un historial claro. Al adoptar Conventional Commits, los mensajes de commit se vuelven más estructurados, facilitando la colaboración y la automatización. Usa comandos como `git reset --hard` o `git push -f` con cuidado, ya que pueden alterar el historial de manera irreversible.