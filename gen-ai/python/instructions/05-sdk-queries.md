---
title: "05: Ejecución de una consulta con el SDK de Azure Cosmos\_DB for NoSQL"
lab:
  title: "05: Ejecución de una consulta con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: Python SDK labs
---

# Ejecución de una consulta con el SDK de Azure Cosmos DB for NoSQL

La versión más reciente del SDK de Python para Azure Cosmos DB for NoSQL simplifica la consulta de un contenedor y la iteración a través de conjuntos de resultados mediante las características modernas de Python.

La biblioteca `azure-cosmos` tiene una funcionalidad integrada para que la consulta de Azure Cosmos DB sea eficaz y sencilla.

En este laboratorio, usarás un iterador para procesar un conjunto de resultados grande devuelto de Azure Cosmos DB for NoSQL. Usarás el SDK de Python para consultar e iterar los resultados.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio para **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para la **Compilación de copilotos con laboratorios de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#create-azure-cosmos-db-database-and-container-with-sample-data). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Creación de un contenedor y una base de datos de Azure Cosmos DB con datos de ejemplo

Si ya has creado una base de datos de Azure Cosmos DB denominada **cosmicworks-full** y un contenedor dentro de ella denominada **products**, que se ha cargado previamente con datos de ejemplo, puedes usarla para este laboratorio y pasar a la [sección siguiente](#install-the-azure-cosmos-library). De lo contrario, sigue los pasos siguientes para crear una nueva base de datos y un contenedor de ejemplo.

<details markdown=1>
<summary markdown="span"><strong>Haz clic para expandir o contraer los pasos para crear una base de datos y un contenedor con los datos de ejemplo.</strong></summary>

1. En el recurso de cuenta de **Azure Cosmos DB** recién creado, ve al panel **Data Explorer**.

1. En **Data Explorer**, selecciona **Iniciar inicio rápido** en la página principal.

1. En la ventana **Nuevo contenedor**, escribe los siguientes valores:

    - **Id. de base de datos**: `cosmicworks-full`
    - **Id. de contenedor**: `products`
    - **Clave de partición**: `/categoryId`
    - **Almacén analítico**: `Off`

1. Selecciona **Aceptar** para crear el contenedor nuevo. Este proceso tardará uno o dos minutos mientras crea los recursos y carga previamente el contenedor con datos de producto de ejemplo.

1. Mantén abierta la pestaña del explorador, ya que volveremos a ella más adelante.

1. Vuelve a **Visual Studio Code**.

</details>

## Instalación de la biblioteca azure-cosmos

La biblioteca **azure-cosmos** está disponible en **PyPI** para facilitar la instalación en los proyectos de Python.

1. En **Visual Studio Code**, en el panel **Explorer**, explora la carpeta **python/05-sdk-queries**.

1. Abre el menú contextual de la carpeta **python/05-sdk-queries** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/05-sdk-queries**.

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

## Iteración de los resultados de una consulta SQL mediante el SDK

Con las credenciales de la cuenta recién creada, te conectarás con las clases del SDK y te conectarás a la base de datos y al contenedor que aprovisionaste en un paso anterior. Además, iterarás los resultados de una consulta SQL mediante el SDK.

Ahora usarás un iterador para crear un bucle sencillo a través de los resultados paginados de Azure Cosmos DB. En segundo plano, el SDK administrará el iterador de fuente y se asegurará de que las solicitudes posteriores se invoquen correctamente.

1. En **Visual Studio Code**, en el panel **Explorer**, explora la carpeta **python/05-sdk-queries**.

1. Abre el archivo Python en blanco llamado **script.py**.

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

1. Agrega el código siguiente para conectarte a la base de datos y al contenedor que creaste anteriormente:

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. Crea una variable de cadena de consulta denominada `sql` con el valor de `SELECT * FROM products p`.

   ```python
   sql = "SELECT * FROM products p"
   ```

1. Invoca el método [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) con la variable `sql` como parámetro del constructor.

   ```python
   result_iterator = container.query_items(
       query=sql
   )
   ```

1. El método **query_items** devolvió un iterador asincrónico que almacenamos en una variable denominada `result_iterator`. Esto significa que cada objeto del iterador es un objeto que se puede esperar y que aún no contiene el resultado de la consulta. Agrega el código siguiente para crear un bucle asincrónico **for** para esperar cada resultado de consulta mientras procesas iteraciones en el iterador asincrónico e imprimes los objetos `id`, `name` y `price` de cada elemento.

   ```python
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Debajo del método `main`, agrega el código siguiente para ejecutar el método `main` mediante la biblioteca `asyncio`:

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. Ahora, el aspecto del archivo **script.py** será parecido a este:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           result_iterator = container.query_items(
               query=sql
           )
            
           # Perform the query asynchronously
           async for item in result_iterator:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

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

1. El script generará ahora todos los productos del contenedor.

## Realización de una consulta dentro de una partición lógica

En la sección anterior, consultaste todos los elementos del contenedor. De forma predeterminada, **CosmosClient** asincrónico realiza consultas entre particiones. Debido a esto, la consulta que ejecutaste (`"SELECT * FROM products p"`) hizo que el motor de consultas examinara todas las particiones del contenedor. Como procedimiento recomendado, siempre debes consultar dentro de una partición lógica para evitar consultas entre particiones. Si lo haces, a la larga, te ahorrarás dinero y mejorarás el rendimiento.

En esta sección, realizarás una consulta dentro de una partición lógica mediante la inclusión de la clave de partición en la consulta.

1. Vuelve a la pestaña Editor del archivo de código **script.py**.

1. Elimina las siguientes líneas de código:

   ```python
   result_iterator = container.query_items(
       query=sql
   )
    
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Modifica el script para crear una variable **partition_key** para almacenar el valor de Id. de categoría para jerséis. Agrega **partition_key** como un parámetro al método **query_items**. Esto garantiza que la consulta se ejecute dentro de la partición lógica de la categoría jerséis.

   ```python
   partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

   result_iterator = container.query_items(
       query=sql,
       partition_key=partition_key
   )
   ```

1. En la sección anterior, realizaste un bucle asincrónico for directamente en el iterador asincrónico (`async for item in result_iterator:`). Esta vez, crearás de forma asincrónica una lista completa de los resultados reales de la consulta. Este código realiza la misma acción que el ejemplo de bucle for que usaste anteriormente. Agrega las siguientes líneas de código para crear una lista de resultados e imprimirlos:

   ```python
   item_list = [item async for item in result_iterator]

   for item in item_list:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Ahora, el aspecto del archivo **script.py** será parecido a este:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

           result_iterator = container.query_items(
               query=sql,
               partition_key=partition_key
           )
    
           # Perform the query asynchronously
           item_list = [item async for item in result_iterator]
    
           for item in item_list:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Guarda** el archivo **script.py**.

1. Ejecuta el script para crear la base de datos y el contenedor:

   ```bash
   python script.py
   ```

1. El script generará ahora todos los productos dentro de la categoría jersey, realizando eficazmente una consulta en partición.

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
