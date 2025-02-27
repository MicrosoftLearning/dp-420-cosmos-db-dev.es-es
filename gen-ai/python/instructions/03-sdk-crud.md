---
title: '03: Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL'
lab:
  title: 03 - Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: Python SDK labs
---

# Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL

La biblioteca `azure-cosmos` incluye métodos para crear, recuperar, actualizar y eliminar elementos (CRUD) dentro de un contenedor de Azure Cosmos DB for NoSQL. Juntos, estos métodos realizan algunas de las operaciones "CRUD" más comunes en varios elementos dentro de los contenedores de la API NoSQL.

En este laboratorio, utilizarás el SDK de Python para realizar operaciones CRUD cotidianas en un elemento dentro de un contenedor de Azure Cosmos DB for NoSQL.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos con Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#install-the-azure-cosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Instalación de la biblioteca azure-cosmos

La biblioteca **azure-cosmos** está disponible en **PyPI** para facilitar la instalación en los proyectos de Python.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/03-sdk-crud**.

1. Abre el menú contextual de la carpeta **python/03-sdk-crud** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/03-sdk-crud**.

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

1. La versión asincrónica del SDK también requiere la biblioteca `aiohttp`. Instálalo con el comando siguiente:

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
       asyncio.run(query_items_async())
   ```

1. Ahora, el aspecto del archivo **script.py** será parecido a este:

   ```python
   from azure.cosmos import PartitionKey
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

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, observa el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

## Realización de operaciones de creación y lectura de puntos en elementos con el SDK

Ahora usarás el conjunto de métodos asincrónicos en la clase **ContainerProxy** para realizar operaciones comunes en elementos dentro de un contenedor de API NoSQL.

1. Vuelve a **Visual Studio Code**. Si aún no está abierto, abre el archivo de código **script.py** dentro de la carpeta **python/03-sdk-crud**.

1. Crea un nuevo elemento de producto y asígnalo a una variable denominada **saddle** con las siguientes propiedades:

    | Propiedad | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Sillín para carretera* |
    | **price** | *45.99d* |
    | **tags** | *{ tan, new, crisp }* |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. Invoca el método [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) de la variable **container**; para ello, pasa la variable **saddle** como el parámetro de método:

   ```python
   await container.create_item(body=saddle)
   ```

1. Una vez que hayas terminado, el archivo de código debería incluir:
  
   ```python
   from azure.cosmos import PartitionKey
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
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Vuelve a **guardar** y ejecutar el script:

   ```bash
   python script.py
   ```

1. Observa el nuevo elemento en **Data Explorer**.

1. Vuelve a **Visual Studio Code**.

1. Vuelve a la pestaña Editor del archivo de código **script.py**.

1. Elimina las siguientes líneas de código:

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. Crea una variable de cadena denominada **item_id** con un valor de **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. Crea una variable de cadena denominada **partition_key** con un valor de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Invoca el método [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item) de la variable **container** pasando las variables **item_id** y **partition_key** como parámetros de método:

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; El método `read_item` permite realizar una operación de lectura puntual en un elemento del contenedor. El método requiere los parámetros `item_id` y `partition_key` para identificar el elemento que se va a leer. En lugar de ejecutar una consulta mediante el lenguaje de consulta SQL de Cosmos DB para buscar el único elemento, el método `read_item` es más eficaz y rentable para recuperar un solo elemento. Las lecturas puntuales pueden leer los datos directamente y no requieren que el motor de consultas procese la solicitud.

1. Imprime el objeto saddle mediante una cadena de salida con formato:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Una vez que hayas terminado, el archivo de código debería incluir:

   ```python
   from azure.cosmos import PartitionKey
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
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Vuelve a **guardar** y ejecutar el script:

   ```bash
   python script.py
   ```

1. Observa la salida del terminal. En concreto, observa el texto de salida con formato con el identificador, el nombre y el precio del elemento.

## Realización de operaciones de actualización y eliminación de puntos con el SDK

Al aprender el SDK, no es raro usar una cuenta de Azure Cosmos DB en línea o el emulador para actualizar un elemento y alternar entre Data Explorer y el IDE que prefieras mientras realizas una operación y compruebas si se ha aplicado el cambio. Aquí, lo harás al actualizar y eliminar un elemento mediante el SDK.

1. Vuelve a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, expande el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Selecciona el nodo **Items**. Selecciona el único artículo dentro del contenedor y observa los valores de las propiedades **name** y **price** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *Sillín para carretera* |
    | **Precio** | *45,99 $* |

    > &#128221; En este momento, estos valores no deben haberse cambiado desde que creaste el artículo. Estos valores se cambiarán en este ejercicio.

1. Vuelve a **Visual Studio Code**. Vuelve a la pestaña Editor del archivo de código **script.py**.

1. Elimina la siguiente línea de código:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Cambia la variable **saddle** estableciendo el valor de la propiedad price en **32.55**:

   ```python
   saddle["price"] = 32.55
   ```

1. Modifica de nuevo la variable **saddle** estableciendo el valor de la propiedad **name** en **Sillín para carretera LL**:

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. Invoca el método [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) de la variable **container**; para ello, pasa las variables **item_id** y **saddle** como parámetros del método:

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Una vez que haya terminado, el archivo de código debería incluir:

   ```python
   from azure.cosmos import PartitionKey
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
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. Vuelve a **guardar** y ejecutar el script:

   ```bash
   python script.py
   ```

1. Vuelve a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, expande el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Selecciona el nodo **Items**. Selecciona el único artículo dentro del contenedor y observa los valores de las propiedades **name** y **price** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Name** | *Road LL Saddle* |
    | **Price** | *$32.55* |

    > &#128221; En este momento, estos valores deben haberse cambiado desde que has observado el elemento.

1. Vuelve a **Visual Studio Code**. Vuelve a la pestaña Editor del archivo de código **script.py**.

1. Elimina las siguientes líneas de código:

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Invoca el método [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) de la variable **container**; para ello, pasa las variables **item_id** y **partition_key** como parámetros del método:

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. Guarda y vuelve a ejecutar el script

   ```bash
   python script.py
   ```

1. Cierra el terminal integrado.

1. Vuelve a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, expande el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Selecciona el nodo **Items**. Observa que la lista de elementos ahora está vacía.

1. Cierra la ventana o pestaña del explorador web.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
