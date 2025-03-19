---
lab:
  title: "04: Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure Cosmos DB for NoSQL

El SDK de Python `azure-cosmos` proporciona el  método `execute_item_batch` para ejecutar varias operaciones de punto en un solo paso lógico. Esto permite a los desarrolladores agrupar de forma eficaz varias operaciones y determinar si se completaron correctamente en el lado servidor.

En este laboratorio, usarás el SDK de Python para realizar operaciones por lotes de dos elementos que muestran lotes transaccionales correctos y errantes.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#install-the-azure-cosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Instalación de la biblioteca azure-cosmos

La biblioteca **azure-cosmos** está disponible en **PyPI** para facilitar la instalación en los proyectos de Python.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/04-sdk-batch**.

1. Abre el menú contextual de la carpeta **python/04-sdk-batch** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/04-sdk-batch**.

1. Creación y activación de un entorno virtual para administrar las dependencias:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Ejecuta el comando siguiente para instalar el paquete [azure-cosmos][pypi.org/project/azure-cosmos]:

   ```bash
   pip install azure-cosmos
   ```

1. Puesto que usamos la versión asincrónica del SDK, también es necesario instalar la biblioteca `asyncio`:

   ```bash
   pip install asyncio
   ```

1. La versión asincrónica del SDK también requiere la biblioteca `aiohttp`. Instálala con el comando siguiente:

   ```bash
   pip install aiohttp
   ```

1. Instala la biblioteca [azure-identity][pypi.org/project/azure-identity], que nos permite usar la autenticación de Azure para conectarnos al área de trabajo de Azure Cosmos DB mediante el siguiente comando:

   ```bash
   pip install azure-identity
   ```

## Uso de la biblioteca azure-cosmos

Con las credenciales de la cuenta recién creada, te conectarás con las clases del SDK y crearás una nueva instancia de contenedor y una base de datos. A continuación, usarás Data Explorer para validar que las instancias existan en Azure Portal.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/03-sdk-crud**.

1. Abre el archivo Python en blanco llamado **script.py**.

1. Agrega la siguiente instrucción `import` para importar la clase **PartitionKey**:

   ```python
   from azure.cosmos import PartitionKey
   ```

1. Agrega las siguientes `import` instrucciones para importar la clase **CosmosClient** asincrónica, **la clase DefaultAzureCredential** y la biblioteca **asincrónica**:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. Agrega las variables denominadas **endpoint** y **credential** y establece el valor de **endpoint** en el **punto de conexión** de la cuenta de Azure Cosmos DB que creaste anteriormente. La variable **credential** debe establecerse en una nueva instancia de la clase **DefaultAzureCredential**:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https://dp420.documents.azure.com:443/**, la instrucción sería: **endpoint = "https://dp420.documents.azure.com:443/".**

1. La interacción con Cosmos DB comienza con una instancia de `CosmosClient`. Para usar el cliente asincrónico, es necesario usar las palabras clave async/await, que solo se pueden usar en métodos asincrónicos. Crea un nuevo método asincrónico denominado **main** y agrega el código siguiente para crear una nueva instancia de la clase **CosmosClient** asincrónica mediante las variables **endpoint** y **credential**:

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161; Puesto que estamos usando el cliente asincrónico **CosmosClient**, también tienes que prepararlo y cerrarlo para poder usarlo correctamente. Se recomienda usar las palabras clave `async with` como se muestra en el código anterior para iniciar los clientes: estas palabras clave crean un administrador de contexto que activa, inicializa y limpia automáticamente el cliente, para que no tengas que hacerlo tú.

1. Agrega el siguiente código para crear una base de datos y un contenedor si aún no existen:

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. Debajo del método `main`, agrega el código siguiente para ejecutar el método `main` mediante la biblioteca `asyncio`:

   ```python
   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Ahora, el aspecto del archivo **script.py** será parecido a este:

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Guarda** el archivo **script.py**.

1. Antes de ejecutar el script, debes iniciar sesión en Azure mediante el comando `az login`. En la ventana de terminal, ejecuta lo siguiente:

   ```bash
   az login
   ```

1. Ejecuta el script para crear la base de datos y el contenedor:

   ```bash
   python script.py
   ```

1. Cambia a la ventana del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, observa el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

## Creación de un lote transaccional

En primer lugar, vamos a crear un sencillo lote transaccional que hace dos productos ficticios. Este lote insertará un sillín usado y un manillar oxidado en el contenedor con el mismo identificador de categoría "used accessories". Ambos elementos tienen la misma clave de partición lógica, lo que garantiza que tendremos una operación por lotes correcta.

1. Vuelve a **Visual Studio Code**. Si aún no está abierto, abre el archivo de código **script.py** dentro de la carpeta **python/04-sdk-batch**.

1. Crea dos diccionarios que representen productos: una **sillín usado** y un **manillar oxidado**. Ambos elementos comparten el mismo valor de clave de partición de **"9603ca6c-9e28-4a02-9194-51cdb7fea816"**.

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. Define el valor de clave de partición.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Crea un lote que contenga los dos elementos.

   ```python
   batch = [saddle, handlebar]
   ```

1. Ejecuta el lote mediante el método `execute_item_batch` del objeto `container` e imprime la respuesta para cada elemento del lote.

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. Una vez que hayas terminado, el archivo de código debería incluir:
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Vuelve a **guardar** y ejecutar el script:

   ```bash
   python script.py
   ```

1. La salida debe indicar un código de estado correcto para cada operación.

## Creación de un lote transaccional errante

Ahora, vamos a crear un lote transaccional que producirá un error a propósito. Este lote intentará insertar dos elementos que tengan claves de partición lógica diferentes. Crearemos una luz estroboscópica parpadeante en la categoría "used accessories" y un nuevo casco en la categoría "pristine accessories". Por definición, debe ser una solicitud incorrecta y devolver un error al realizar esta transacción.

1. Vuelve a la pestaña Editor del archivo de código **script.py**.

1. Elimina las siguientes líneas de código:

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. Modifica el script para crear una nueva **luz estroboscópica parpadeante** y un **nuevo casco** con diferentes valores de clave de partición.

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. Define el valor de clave de partición para el lote.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Crea un nuevo lote que contenga los dos elementos.

   ```python
   batch = [light, helmet]
   ```

1. Una vez que hayas terminado, el archivo de código debería incluir:

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. Vuelve a **guardar** y ejecutar el script:

   ```bash
   python script.py
   ```

1. Observa la salida del terminal. El código de estado del segundo elemento (el "Nuevo casco") debe ser **400** para **Solicitud incorrecta**. Esto se ha producido porque todos los elementos de la transacción no comparten el mismo valor de clave de partición que el lote transaccional.

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
