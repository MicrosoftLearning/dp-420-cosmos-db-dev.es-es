---
title: "01: Conexión a Azure Cosmos\_DB for NoSQL con el SDK"
lab:
  title: "01: Conexión a Azure Cosmos\_DB for NoSQL con el SDK"
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: JavaScript SDK labs
---

# Conexión a Azure Cosmos DB for NoSQL con el SDK

El SDK de Azure para JavaScript (Node.js y explorador) es un conjunto de bibliotecas cliente que proporciona una interfaz de desarrollador coherente para interactuar con muchos servicios de Azure. Las bibliotecas cliente son paquetes que se usarían para consumir estos recursos e interactuar con ellos.

En este laboratorio, conectarás a una cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para JavaScript.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya creaste una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos con Azure Cosmos DB** de este sitio, puedes usarla para este laboratorio y pasar a la [siguiente sección](#import-the-azurecosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Importa la biblioteca @azure/cosmos

La biblioteca **@azure/cosmos** está disponible en **npm** para facilitar su instalación en los proyectos JavaScript.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/01-sdk-connect**.

1. Abre el menú contextual de la carpeta **javascript/01-sdk-connect** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **javascript/01-sdk-connect**.

1. Inicia un nuevo proyecto Node.js:

    ```bash
    npm init -y
    ```

1. Instala el paquete [@azure/cosmos][npmjs.com/package/@azure/cosmos] ejecutando el siguiente comando:

    ```bash
    npm install @azure/cosmos
    ```

1. Instala la biblioteca [@azure/identity][npmjs.com/package/@azure/identity], que permite utilizar la autenticación de Azure para conectarse al espacio de trabajo de Azure Cosmos DB, mediante el siguiente comando:

    ```bash
    npm install @azure/identity
    ```

## Usa la biblioteca @azure/cosmos

Una vez importada la biblioteca de Azure Cosmos DB del SDK de Azure para JavaScript, puedes utilizar inmediatamente sus clases para conectarte a una cuenta de Azure Cosmos DB for NoSQL. La clase **CosmosClient** es la clase principal que se usa para establecer la conexión inicial a una cuenta de Azure Cosmos DB for NoSQL.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/01-sdk-connect**.

1. Abre el archivo JavaScript vacío llamado **script.js**.

1. Agrega las siguientes instrucciones `require` para importar las bibliotecas **@azure/cosmos** y **@azure/identity**:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. Agrega las variables denominadas **endpoint** y **credential** y establece el valor de **endpoint** en el **punto de conexión** de la cuenta de Azure Cosmos DB que creaste anteriormente. La variable **credential** debe establecerse en una nueva instancia de la clase **DefaultAzureCredential**:

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; Por ejemplo, si el punto final es **https://dp420.documents.azure.com:443/**, la instrucción sería **const endpoint = "https://dp420.documents.azure.com:443/";**.

1. Agrega una nueva variable denominada **client** e inicialízala como una nueva instancia de la clase **CosmosClient** mediante las variables **endpoint** y **credential**:

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. Agrega una función `async` denominada **main** para leer e imprimir las propiedades de la cuenta:

    ```javascript
    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }
    ```

1. Invoca la función **main**:

    ```javascript
    main().catch((error) => console.error(error));
    ```

1. Tu archivo **script.js** debería tener ahora este aspecto:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }

    main().catch((error) => console.error(error));
    ```

1. **Guarda** el archivo **script.js**.

## Prueba del script

Ahora que el código de JavaScript para conectarse a la cuenta de Azure Cosmos DB for NoSQL está completo, puedes probar el script. Este script imprimirá el nivel de coherencia predeterminado y el nombre de la primera región grabable. Al crear la cuenta, especificaste una ubicación y deberías esperar ver ese mismo valor de ubicación impreso como resultado de este script.

1. En **Visual Studio Code**, abre el menú contextual de la carpeta **javascript/01-sdk-connect** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

1. Antes de ejecutar el script, debes iniciar sesión en Azure mediante el comando `az login`. En la ventana de terminal, ejecuta lo siguiente:

    ```bash
    az login
    ```

1. Ejecuta el script mediante el comando `node`:

    ```bash
    node script.js
    ```

1. El script ahora generará el nivel de coherencia predeterminado de la cuenta y la primera región grabable. Por ejemplo, si el nivel de coherencia predeterminado para la cuenta es **Sesión** y la primera región grabable es **Este de EE. UU.**, el script generaría:

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
