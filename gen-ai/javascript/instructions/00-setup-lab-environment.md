---
lab:
  title: Configuración del entorno de laboratorio
  module: Setup
---

# Configuración del entorno de laboratorio local

Lo ideal es completar estos laboratorios en un entorno de laboratorio hospedado. Si deseas completarlos en tu propio equipo, puedes hacerlo instalando el software siguiente. Puedes experimentar diálogos y comportamientos inesperados al usar tu propio entorno. Debido a la amplia variedad de configuraciones locales posibles, el equipo del curso no puede cubrir problemas que puedan surgir en tu propio entorno.

## Herramientas de línea de comandos de Azure

1. [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) o [Azure Cloud Shell](https://shell.azure.com): instala si deseas ejecutar comandos a través de la CLI en lugar de Azure Portal.

## Node.js

1. Descarga e instala Node.js v18.0.0 o posterior desde [nodejs.org/en/download].

1. Descarga e instala NPM v10.2.3 o posterior desde [npmjs.com/get-npm].

Esta es la manera recomendada de instalar la versión más reciente de NPM y node.js en Windows:

- Instala NVM desde [github.com/coreybutler/nvm-windows]
- Ejecuta nvm install latest
- Ejecuta nvm list (para ver las versiones de NPM/node.js disponibles)
- Ejecute nvm use latest (para usar la versión más reciente disponible)

### Git

1. Descarga e instala desde [git-scm.com/downloads].

    - Usa las opciones predeterminadas del instalador.

### Visual Studio Code (y extensiones)

1. Descarga e instala desde [code.visualstudio.com/download].

    - Usa las opciones predeterminadas del instalador.

1. Después de la instalación, inicia Visual Studio Code.

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
[nodejs.org/en/download]: https://nodejs.org/en/download
[npmjs.com/get-npm]: https://npmjs.com/get-npm
[github.com/coreybutler/nvm-windows]: https://github.com/coreybutler/nvm-windows
