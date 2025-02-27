---
title: '07.2: Generación de incrustaciones de vectores con Azure OpenAI y almacenaje en Azure Cosmos DB for NoSQL'
lab:
  title: '07.2: Generación de incrustaciones de vectores con Azure OpenAI y almacenaje en Azure Cosmos DB for NoSQL'
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 11
parent: Python SDK labs
---

# Generación de incrustaciones de vectores con Azure OpenAI y almacenaje en Azure Cosmos DB for NoSQL

Azure OpenAI proporciona acceso a los modelos de lenguaje avanzado de OpenAI, incluidos los modelos `text-embedding-ada-002`, `text-embedding-3-small` y `text-embedding-3-large`. Al aprovechar uno de estos modelos, puedes generar representaciones vectoriales de datos textuales, que se pueden almacenar en un almacén vectorial como Azure Cosmos DB for NoSQL. Esto facilita búsquedas de similitud eficaces y precisas, lo que mejora significativamente la capacidad de un copiloto para recuperar información relevante y proporcionar interacciones contextualmente enriquecidas.

En este laboratorio, crearás un Azure OpenAI Service e implementarás un modelo de incrustación. Después, usarás código de Python para crear clientes de Azure OpenAI y Cosmos DB mediante sus respectivos SDK de Python para generar representaciones vectoriales de descripciones de productos y escribirlas en la base de datos.

> &#128721; El ejercicio anterior de este módulo es un requisito previo para este laboratorio. Si aún necesitas completar ese ejercicio, hágalo antes de continuar, ya que proporciona la infraestructura necesaria para este laboratorio.

## Creación de un Azure OpenAI Service

Azure OpenAI proporciona acceso a la API de REST a los modelos de lenguaje eficaces de OpenAI. Estos modelos se pueden adaptar fácilmente a su tarea específica, entre las que se incluyen, entre otras, la generación de contenido, el resumen, el reconocimiento de imágenes, la búsqueda semántica y la traducción de lenguaje natural a código.

1. Ve a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

2. Inicia sesión en el portal con las credenciales de Microsoft asociadas a tu suscripción.

3. Selecciona **+ Crear un recurso**, busca *Azure OpenAI* y luego crea un nuevo recurso de **Azure OpenAI** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | Configuración | Valor |
    | ------- | ----- |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Selecciona un grupo de recursos ya existente o crea un nuevo* |
    | **Región** | *Elige una región disponible que admita el modelo `text-embedding-3-small`* en la [lista de regiones admimtidas](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings). |
    | **Nombre** | *Escribe un nombre único global*. |
    | **Plan de tarifa** | *Elige estándar 0* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que te impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

4. Espera a que se complete la tarea de implementación antes de continuar con la tarea siguiente.

## Implementación de un modelo de incrustación

Para usar Azure OpenAI para generar incrustaciones, primero debes implementar una instancia del modelo de incrustación deseado en el servicio.

1. Ve al Azure OpenAI Service recién creado en Azure Portal (``portal.azure.com``).

2. En la página **Información general** de Azure OpenAI Service, inicia **Fundición de IA de Azure**. Para ello, selecciona el vínculo ** Ir al Portal de la Fundición de IA de Azure** en la barra de herramientas.

3. En Fundición de IA de Azure, selecciona **Implementaciones** en el menú izquierdo.

4. En la página **Implementaciones de modelos**, selecciona **Implementar modelo** y selecciona **Implementar modelo base** en el elemento desplegable.

5. Selecciona `text-embedding-3-small` en la lista de modelos.

    > &#128161; Puedes filtrar la lista para mostrar solo los modelos de *incrustación* mediante el filtro de tareas de inferencia.

    > &#128221; Si no ves el modelo `text-embedding-3-small`, es posible que hayas seleccionado una región de Azure que no admita ese modelo actualmente. En este caso, puedes usar el modelo `text-embedding-ada-002` para este laboratorio. Ambos modelos generan vectores con 1536 dimensiones, por lo que no se requieren cambios en la directiva de vectores de contenedor definida en el contenedor `Products` de Azure Cosmos DB.

6. Selecciona **Confirmar** para implementar el modelo.

7. En la página **Implementaciones de modelos** de Fundición de IA de Azure, anota el **nombre** de la implementación de modelo `text-embedding-3-small`, ya que lo necesitarás más adelante en este ejercicio.

## Implementación de un modelo de finalización de chat

Además del modelo de incrustación, necesitarás un modelo de finalización de chat para el copiloto. Usarás el modelo de lenguaje grande `gpt-4o` de OpenAI para generar respuestas de tu copiloto.

1. Mientras sigues en la página **Implementaciones de modelos** en Fundición de IA de Azure, vuelve a seleccionar el botón **Implementar modelo** y elige **Implementar modelo base** en el elemento desplegable.

2. Selecciona el modelo de finalización de chat **gpt-4o** de la lista.

3. Selecciona **Confirmar** para implementar el modelo.

4. En la página **Implementaciones de modelos** de Fundición de IA de Azure, anota el **nombre** de la implementación de modelo `gpt-4o`, ya que lo necesitarás más adelante en este ejercicio.

## Asignación del rol RBAC de usuario de OpenAI de Cognitive Services

Para permitir que la identidad de usuario interactúe con Azure OpenAI Service, puedes asignar a tu cuenta el rol de **usuario de OpenAI de Cognitive Services**. Azure OpenAI Service admite el control de acceso basado en rol de Azure (RBAC de Azure), un sistema de autorización que permite administrar el acceso individual a los recursos de Azure. Con RBAC de Azure, se asignan diferentes miembros del equipo a distintos niveles de permisos en función de sus necesidades relacionadas con un proyecto dado.

> &#128221; El control de acceso basado en rol (RBAC) de Microsoft Entra ID para autenticarse en los servicios de Azure, como Azure OpenAI, mejora la seguridad mediante controles de acceso precisos adaptados a roles de usuario, lo que reduce eficazmente los riesgos de acceso no autorizado. La optimización de la administración de acceso seguro mediante RBAC de Entra ID crea una solución más eficaz y escalable para aprovechar los servicios de Azure.

1. En Azure Portal (``portal.azure.com``), ve a tu recurso de Azure OpenAI.

2. Selecciona **Control de acceso (IAM)** en el panel de navegación izquierdo.

3. Selecciona **Agregar** y, luego, **Agregar asignación de roles**.

4. En la pestaña **Rol**, selecciona el rol de **usuario de OpenAI de Cognitive Services** y, después, selecciona **Siguiente**.

5. En la pestaña **Miembros**, selecciona Asignar acceso a un usuario, grupo o entidad de servicio y, después, selecciona **Seleccionar miembros**.

6. En el cuadro de diálogo **Seleccionar miembros**, busca tu nombre o dirección de correo electrónico y selecciona tu cuenta.

7. En la pestaña **Revisión y asignación**, selecciona **Revisión y asignación** para asignar el rol.

## Creación de un entorno virtual de Python

Los entornos virtuales de Python son esenciales para mantener un espacio de desarrollo limpio y organizado, lo que permite que los proyectos individuales tengan su propio conjunto de dependencias, aislados de otros. Esto evita conflictos entre diferentes proyectos y garantiza la coherencia en el flujo de trabajo de desarrollo. Mediante el uso de entornos virtuales, puedes administrar fácilmente las versiones de paquetes, evitar conflictos de dependencia y mantener los proyectos se ejecuten sin problemas. Es un procedimiento recomendado que mantiene el entorno de codificación estable y confiable, lo que hace que el proceso de desarrollo sea más eficaz y menos propenso a problemas.

1. Con Visual Studio Code, abre la carpeta en la que has clonado el repositorio de código de laboratorio para el módulo de aprendizaje **Compilación de copilotos con Azure Cosmos DB**.

2. En Visual Studio Code, abre una nueva ventana de terminal y cambia los directorios a la carpeta `python/07-build-copilot`.

3. Para crear un entorno virtual denominado `.venv`, ejecuta el siguiente comando en el símbolo del sistema del terminal:

    ```bash
    python -m venv .venv 
    ```

    El comando anterior creará una carpeta `.venv` en la carpeta `07-build-copilot`, que proporcionará un entorno de Python dedicado para los ejercicios de este laboratorio.

4. Activa el entorno virtual seleccionando el comando adecuado para el sistema operativo y el shell en la tabla siguiente y ejecutándolo en el símbolo del sistema del terminal.

    | Plataforma | Shell | Comando para activar el entorno virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. Instala las bibliotecas definidas en `requirements.txt`:

    ```bash
    pip install -r requirements.txt
    ```

    El archivo `requirements.txt` contiene un conjunto de bibliotecas de Python que usarás en este laboratorio.

    | Biblioteca | Versión | Descripción |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | SDK de Azure Cosmos DB para Python: biblioteca cliente |
    | `azure-identity` | 1.19.0 | SDK de Identidad de Azure para Python |
    | `fastapi` | 0.115.5 | Marco web para compilar las API con Python |
    | `openai` | 1.55.2 | Proporciona acceso a la API de REST de Azure OpenAI desde aplicaciones de Python. |
    | `pydantic` | 2.10.2 | Validación de datos mediante sugerencias de tipo de Python. |
    | `requests` | 2.32.3 | Enviar solicitudes HTTP |
    | `streamlit` | 1.40.2 | Transforma los scripts de Python en aplicaciones web interactivas. |
    | `uvicorn` | 0.32.1 | Una implementación del servidor web ASGI para Python. |
    | `httpx` | 0.27.2 | Un cliente HTTP de última generación para Python. |

## Adición de una función de Python para vectorizar texto

El SDK de Python para Azure OpenAI proporciona acceso a clases síncronas y asíncronas que se pueden usar para crear inserciones para datos textuales. Esta funcionalidad se puede encapsular en una función en el código de Python.

1. En el panel **Explorer** de Visual Studio Code, ve a la carpeta `python/07-build-copilot/api/app` y abre el archivo `main.py` ubicado allí.

    > &#128221; Este archivo servirá como punto de entrada a una API de Python de back-end que compilarás en el ejercicio siguiente. En este ejercicio, proporcionarás una serie de funciones asíncronas que se pueden usar para importar datos con inserciones en Azure Cosmos DB que la API aprovechará.

2. Para usar el SDK asíncrono de Azure OpenAI para Python, importa la biblioteca agregando el código siguiente a la parte superior del archivo `main.py`:

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. Accederás de forma asíncrona a Azure OpenAI y Cosmos DB mediante la autenticación de Azure y los roles de RBAC de Entra ID que asignaste previamente a la identidad de tu usuario. Agrega la siguiente línea debajo de la instrucción `openai` import de la parte superior del archivo para importar las clases necesarias de la biblioteca `azure-identity`:

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; Para asegurarte de que puedes interactuar de forma segura con los servicios de Azure desde la API, usarás el SDK de identidad de Azure para Python. Este enfoque te permite evitar tener que almacenar o interactuar con claves del código, en lugar de aprovechar los roles de RBAC que asignaste a tu cuenta para acceder a Azure Cosmos DB y Azure OpenAI en los ejercicios anteriores.

4. Crea variables para almacenar la versión y el punto de conexión de la API de Azure OpenAI, reemplazando el token `<AZURE_OPENAI_ENDPOINT>` por el valor del punto de conexión para Azure OpenAI Service. Además, crea una variable para el nombre de la implementación del modelo de inserción. Inserta el código siguiente debajo de las instrucciones `import` en el archivo:

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    Si el nombre de implementación de inserción es diferente, actualiza el valor asignado a la variable en consecuencia.

    > &#128161; La versión de la API de `2024-10-21` era la versión más reciente de disponibilidad general a partir del momento de redacción de este documento. Puedes usar esa o una nueva versión, si hay una disponible. La documentación de las especificaciones de API contiene una [tabla con las versiones de API más recientes](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs).

    > &#128221; El objeto `EMBEDDING_DEPLOYMENT_NAME` es el valor **Name** que anotaste después de implementar el modelo `text-embedding-3-small` en Fundición de IA de Azure. Si necesitas recuperarla, inicia Fundición de IA de Azure, ve a la página **Implementaciones** y busca la implementación cuyo **nombre del modelo** sea `text-embedding-3-small` A continuación, copia el valor del campo **Nombre** de ese elemento. Si has implementado el modelo `text-embedding-ada-002`, usa el nombre de esa implementación.

5. Usa la clase del SDK de Identidad de Azure para la clase `DefaultAzureCredential` de Python para crear una credencial asíncrona y acceder a Azure OpenAI y Azure Cosmos DB mediante la autenticación RBAC de Microsoft Entra ID al insertar el código siguiente debajo de las declaraciones de variables:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. Para controlar la creación de inserciones, inserta lo siguiente, que agrega una función para generar inserciones mediante un cliente de Azure OpenAI:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

    La creación del cliente de Azure OpenAI no requiere el valor `api_key` porque está recuperando un token de portador mediante la clase `get_bearer_token_provider` del SDK de Identidad de Azure.

7. El archivo `main.py` debería tener ahora un aspecto similar a los siguiente:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

8. Guarde el archivo `main.py`.

## Prueba de la función de inserción

Para asegurarte de que la función `generate_embeddings` del archivo `main.py` funciona correctamente, agregarás algunas líneas de código en la parte inferior del archivo para permitir que se ejecute directamente. Estas líneas permiten ejecutar la función `generate_embeddings` desde la línea de comandos, pasando el texto que se va a insertar.

1. Agrega un bloque **main guard** que contenga una llamada a `generate_embeddings` en la parte inferior del archivo `main.py`:

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; El bloque `if __name__ == "__main__":` se conoce normalmente como **main guard** o **entry point** en Python. Garantiza que cierto código solo se ejecuta cuando el script se ejecuta directamente y no cuando se importa como módulo en otro script. Esta práctica ayuda a organizar el código y hace que sea más reutilizable y modular.

2. Guarda el archivo `main.py`, que ahora debería tener este aspecto:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. En Visual Studio Code, abre una nueva ventana de terminal integrada.

4. Antes de ejecutar la API, que enviará solicitudes a Azure OpenAI, debes iniciar sesión en Azure mediante el comando `az login`. En la ventana de terminal, ejecuta lo siguiente:

   ```bash
   az login
   ```

5. Completa el proceso de inicio de sesión en tu navegador.

6. Cuando el terminal te lo pida, cambia el directorio a `python/07-build-copilot`.

7. Asegúrate de que la ventana del terminal integrado se esté ejecutando dentro de tu entorno virtual Python. Para ello, activa el entorno virtual mediante un comando de la tabla que aparece a continuación y selecciona el comando adecuado para tu sistema operativo y shell.

    | Plataforma | Shell | Comando para activar el entorno virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. En el símbolo del sistema del terminal, cambia al directorio `api/app` y luego ejecuta el siguiente comando:

   ```python
   python main.py "Hello, world!"
   ```

9. Observa el resultado en la ventana del terminal. Deberías ver una matriz de números de punto flotante, que es la representación vectorial de la cadena "Hello, world!". cadena. Debería tener un aspecto similar al siguiente resultado abreviado:

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## Crea una función para escribir datos en Azure Cosmos DB

Con el SDK de Azure Cosmos DB para Python, puedes crear una función que permita insertar y actualizar documentos en tu base de datos. Una operación upsert actualizará un registro si se encuentra una coincidencia e insertará un nuevo registro si no la hay.

1. Vuelve al archivo `main.py` abierto en Visual Studio Code e importa la clase `CosmosClient` asíncrona del SDK de Azure Cosmos DB para Python. Para ello, inserta la siguiente línea justo debajo de las instrucciones `import` que ya están en el archivo:

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. Agrega otra sentencia import para hacer referencia a la clase `Product` del módulo *models* en la carpeta `api/app`. La clase `Product` define la forma de los productos en el conjunto de datos de Cosmic Works.

   ```python
   from models import Product
   ```

3. Crea un nuevo grupo de variables que contenga valores de configuración asociados con Azure Cosmos DB y agrégalos al archivo `main.py` debajo de las variables de Azure OpenAI que insertaste anteriormente. Asegúrate de reemplazar el token `<AZURE_COSMOSDB_ENDPOINT>` con el punto de conexión de tu cuenta de Azure Cosmos DB.

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. Agrega una función llamada `upsert_product` para actualizar o insertar documentos en Cosmos DB. Para ello, inserta el siguiente código debajo de la función `generate_embeddings` en el archivo `main.py`:

   ```python
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
   ```

5. Guarda el archivo `main.py`, que ahora debería tener este aspecto:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## Vectorizar datos de ejemplo

Ahora estás listo para probar las funciones `generate_embeddings` y `upsert_document` juntas. Para ello, sobrescribirás el bloque de protección principal `if __name__ == "__main__"` con un código que descarga un archivo de datos de ejemplo que contiene información de productos de Cosmic Works desde GitHub, luego vectoriza el campo `description` de cada producto y actualiza o inserta los documentos en el contenedor `Products` de la base de datos Azure Cosmos DB.

> &#128221; Este enfoque se utiliza para demostrar las técnicas de generación con Azure OpenAI y almacenamiento de inserciones en Azure Cosmos DB. Sin embargo, en un escenario real, un enfoque más robusto, como el uso de una función de Azure activada por la fuente de cambios de Azure Cosmos DB, sería más apropiado para gestionar la agregación de inserciones a documentos nuevos y existentes.

1. En el archivo `main.py`, sobrescribe el bloque de código `if __name__ == "__main__":` con lo siguiente:

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. En el símbolo del sistema del terminal integrado abierto en Visual Studio Code, ejecuta el archivo `main.py` de nuevo mediante el comando:

   ```python
   python main.py
   ```

3. Espera a que se complete la ejecución del código, que se indicará mediante un mensaje que especifica que todos los productos con descripciones vectorizadas se han subido al contenedor de Cosmos DB. El proceso de vectorización y actualización de datos puede tardar entre 10 y 15 minutos en completarse para los 295 registros del conjunto de datos de productos. Si no se han insertado todos los productos, puedes volver a ejecutar `main.py` mediante el comando anterior para agregar los productos restantes.

## Revisa los datos de ejemplo insertados y actualizados en Cosmos DB

1. Vuelve a Azure Portal (``portal.azure.com``) y navega hasta tu cuenta de Azure Cosmos DB.

2. Selecciona la instancia de **Data Explorer** para el menú de navegación izquierdo.

3. Expande la base de datos **CosmicWorks** y el contenedor **Products**, luego selecciona **Items** debajo del contenedor.

4. Selecciona varios documentos aleatorios dentro del contenedor y asegúrate de que el campo `embedding` se rellene con una matriz vectorial.
