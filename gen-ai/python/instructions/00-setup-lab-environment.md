---
title: Configuración del entorno de laboratorio
lab:
  title: Configuración del entorno de laboratorio
  module: Setup
layout: default
nav_order: 2
parent: Python SDK labs
---

# Configuración del entorno de laboratorio local

Lo ideal es completar estos laboratorios en un entorno de laboratorio hospedado. Si deseas completarlos en tu propio equipo, puedes hacerlo instalando el software siguiente. Puedes experimentar diálogos y comportamientos inesperados al usar tu propio entorno. Debido a la amplia variedad de configuraciones locales posibles, el equipo del curso no puede cubrir problemas que puedan surgir en tu propio entorno.

## Herramientas de línea de comandos de Azure

1. [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) o [Azure Cloud Shell](https://shell.azure.com): instala si deseas ejecutar comandos a través de la CLI en lugar de Azure Portal.

## Python

1. Descarga e instala Python 3.11+ desde [python.org/downloads] con \<install location\>\Python36 y \<install location>\Python36\Scripts agregados a la ruta de acceso.

    - Usa las opciones predeterminadas del instalador.

## Git

1. Descarga e instala desde [git-scm.com/downloads].

    - Usa las opciones predeterminadas del instalador.

## Visual Studio Code (y extensiones)

1. Descarga e instala desde [code.visualstudio.com/download].

    - Usa las opciones predeterminadas del instalador.

1. Después de la instalación, inicia Visual Studio Code.

1. En el menú **Extensiones**, busca e instala las siguientes extensiones de Microsoft:

    - [Extensión de Python para Visual Studio Code][marketplace.visualstudio.com/mms-python.python]

### Emulador de Azure Cosmos DB

1. Descarga e instala desde [docs.microsoft.com/azure/cosmos-db/local-emulator].
    - Usa las opciones predeterminadas del instalador.

### Clonación del repositorio del laboratorio

Si aún no has clonado el repositorio de código del laboratorio para **Compilación de copilotos con Azure Cosmos DB** en el entorno en el que estás trabajando en este laboratorio, sigue estos pasos para hacerlo. De lo contrario, abre la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicia **Visual Studio Code**.

    > &#128221; Si aún no estás familiarizado con la interfaz de Visual Studio Code, consulta la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abre la paleta de comandos y ejecuta **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` en una carpeta local de tu elección.

    > &#128161; Puedes usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abre la carpeta local que seleccionaste en **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[python.org/downloads]: https://www.python.org/downloads/
[marketplace.visualstudio.com/mms-python.python]: https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview
