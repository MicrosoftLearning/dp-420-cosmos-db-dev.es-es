---
title: "06: Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos\_DB for NoSQL"
lab:
  title: "06: Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Author complex queries with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 9
parent: Python SDK labs
---

# Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos DB for NoSQL

En Azure Cosmos DB, las consultas tendrán normalmente varias páginas de resultados. La paginación se realiza automáticamente en el servidor cuando Azure Cosmos DB no puede devolver todos los resultados de la consulta en una sola ejecución. En muchas aplicaciones, querrás escribir código mediante el SDK para procesar los resultados de la consulta por lotes de una manera eficaz.

En este laboratorio, crearás un iterador de fuente que se puede usar en un bucle para recorrer en iteración todo el conjunto de resultados.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

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

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/06-sdk-pagination**.

1. Abre el menú contextual de la carpeta **python/06-sdk-pagination** y, a continuación, selecciona **Open in Integraged Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/06-sdk-pagination**.

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

## Paginar a través de pequeños conjuntos de resultados de una consulta SQL mediante el SDK

Al procesar los resultados de la consulta, debes asegurarte de que el código avanza a través de todas las páginas de resultados y comprobaciones para ver si quedan más páginas antes de realizar solicitudes posteriores.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/06-sdk-pagination**.

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

1. Crea una variable denominada **sql** de tipo *string* con un valor de **SELECT * FROM products WHERE products.price > 500**:

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. Invoca el método [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) con la variable `sql` como parámetro del constructor. Establece el objeto `max_item_count` en `50` para limitar el número de elementos devueltos en cada página.

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. Crea un bucle **for** asíncrono que invoque de forma asíncrona el método [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) en el objeto iterador. Este método devuelve una página de resultados cada vez que se llama.

   ```python
   async for page in iterator.by_page():
   ```

1. Dentro del bucle **for** asíncrono, procesa una iteración de manera asíncrona sobre los resultados paginados e imprime los objetos `id`, `name` y `price` de cada elemento.

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
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
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. El script ahora generará páginas de 50 elementos a la vez.

    > &#128161; La consulta coincidirá con cientos de elementos del contenedor de productos.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
