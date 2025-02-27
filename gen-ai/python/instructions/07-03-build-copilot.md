---
title: '07.3: Compilación de un copiloto con Python y Azure Cosmos DB for NoSQL'
lab:
  title: '07.3: Compilación de un copiloto con Python y Azure Cosmos DB for NoSQL'
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 12
parent: Python SDK labs
---

# Compilación de un copiloto con Python y Azure Cosmos DB for NoSQL

Mediante el uso de las versátiles funcionalidades de programación de Python y las funcionalidades escalables de vector de búsqueda y bases de datos de NoSQL de Azure Cosmos DB, puedes crear copilotos de IA eficaces y eficientes, lo que simplifica los flujos de trabajo complejos.

En este laboratorio, crearás un copiloto mediante Python y Azure Cosmos DB for NoSQL, lo que creará una API de back-end que proporcionará los puntos de conexión necesarios para interactuar con los servicios de Azure (Azure OpenAI y Azure Cosmos DB) y una interfaz de usuario de front-end para facilitar la interacción del usuario con el copiloto. El copiloto servirá como asistente para ayudar a los usuarios de Cosmic Works a administrar y encontrar productos relacionados con las bicicletas. En concreto, el copiloto permitirá a los usuarios aplicar y quitar descuentos de categorías de productos, buscar categorías de productos para ayudar a informar a los usuarios de qué tipos de productos están disponibles y usar el vector de búsqueda para realizar búsquedas de similitud para los productos.

![Diagrama de arquitectura del copiloto de alto nivel, que muestra una interfaz de usuario desarrollada en Python mediante Streamlit, una API de back-end escrita en Python e interacciones con Azure Cosmos DB y Azure OpenAI.](media/07-copilot-high-level-architecture-diagram.png)

La separación de la funcionalidad de la aplicación en una interfaz de usuario dedicada y la API de back-end al crear un copiloto en Python ofrece varias ventajas. En primer lugar, mejora la modularidad y el mantenimiento, lo que te permite actualizar la interfaz de usuario o el back-end de forma independiente sin interrumpir el otro. Streamlit proporciona una interfaz intuitiva e interactiva que simplifica las interacciones del usuario, mientras que FastAPI garantiza un alto rendimiento, el control asincrónico de solicitudes y el procesamiento de datos. Esta separación también promueve la escalabilidad, ya que se pueden implementar distintos componentes en varios servidores, lo que optimiza el uso de recursos. Además, permite mejores prácticas de seguridad, ya que la API de back-end puede controlar los datos confidenciales y la autenticación por separado, reduciendo el riesgo de exponer vulnerabilidades en la capa de interfaz de usuario. Este enfoque conduce a una aplicación más sólida, eficaz y fácil de usar.

> &#128721; Los ejercicios anteriores de este módulo son un requisito previo para este laboratorio. Si todavía necesitas completar cualquiera de esos ejercicios, hazlos antes de continuar, ya que proporcionan la infraestructura necesaria y el código de inicio para este laboratorio.

## Construcción de una API de back-end

La API de back-end para copiloto enriquece sus capacidades para controlar datos complejos, proporcionar información en tiempo real y conectarse sin problemas con diversos servicios, lo que hace que las interacciones sean más dinámicas e informativas. Para compilar la API de su copiloto, usarás la biblioteca de Python de FastAPI. FastAPI es un marco web moderno y de alto rendimiento diseñado para permitirte compilar las API con Python en función de sugerencias de tipo de Python estándar. Al desacoplar el copiloto desde el back-end mediante este enfoque, se garantiza una mayor flexibilidad, capacidad de mantenimiento y escalabilidad, lo que permite que el copiloto evolucione independientemente de los cambios de back-end.

> &#128721; La API de back-end se basa en el código que agregaste al archivo `main.py` en la carpeta `python/07-build-copilot/api/app` del ejercicio anterior. Si aún no has terminado el ejercicio anterior, es preciso completarlo antes de continuar.

1. Con Visual Studio Code, abre la carpeta en la que has clonado el repositorio de código de laboratorio para el módulo de aprendizaje **Compilación de copilotos con Azure Cosmos DB**.

1. En el panel **Explorer** de Visual Studio Code, ve a la carpeta **python/07-build-copilot/api/app** y abre el archivo `main.py` que se encuentra en ella.

1. Agrega las siguientes líneas de código debajo de las instrucciones existentes `import` en la parte superior del archivo `main.py` para incorporar las bibliotecas que se usarán para realizar acciones asincrónicas mediante FastAPI:

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. Para habilitar el punto de conexión `/chat` que crearás para recibir datos en el cuerpo de la solicitud, pasarás contenido a través de un objeto `CompletionRequest` definido en el módulo de *modelos* de proyectos. Actualiza la instrucción de importación `from models import Product` en la parte superior del archivo para incluir la clase `CompletionRequest` del módulo `models`. La instrucción de importación debería tener ahora este aspecto:

   ```python
   from models import Product, CompletionRequest
   ```

1. Necesitarás el nombre de implementación del modelo de finalización de chat que creaste en Azure OpenAI Service. Crea una variable en la parte inferior del bloque de variables de configuración de Azure OpenAI para proporcionar lo siguiente:

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    Si el nombre de implementación de finalización difiere, actualiza el valor asignado a la variable en consecuencia.

1. Los SDK de Identidad y de Azure Cosmos DB proporcionan métodos asincrónicos para trabajar con esos servicios. Cada una de estas clases se usará en varias funciones de la API, por lo que crearás instancias globales de cada una, lo que permite que el mismo cliente se comparta entre métodos. Inserta las siguientes declaraciones de variable global debajo del bloque variables de configuración de Cosmos DB:

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. Elimina las siguientes líneas de código del archivo, ya que la funcionalidad proporcionada se moverá a la función `lifespan` que definirás en el paso siguiente:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. Para crear instancias singleton de las clases `CosmosClient` y `DefaultAzureCredentail`, aprovecharás el objeto `lifespan` en FastAPI. Este método administra esas clases a través del ciclo de vida de la aplicación de API. Inserta el código siguiente para definir `lifespan`:

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   En FastAPI, los eventos de duración son operaciones especiales que se ejecutan al principio y al final del ciclo de vida de la aplicación. Estas operaciones se ejecutan antes de que la aplicación empiece a controlar las solicitudes y después de que se detenga, lo que les hace ideales para inicializar y limpiar los recursos que se usan en toda la aplicación y que se comparten entre las solicitudes. Este enfoque garantiza que la configuración necesaria se complete antes de que se procesen las solicitudes y que los recursos se administren correctamente al apagarse.

1. Crea una instancia de la clase FastAPI mediante el código siguiente. Se debe insertar debajo de la función `lifespan`:

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   Al llamar a `FastAPI()`, vas a inicializar una nueva instancia de la aplicación FastAPI. Esta instancia, denominada `app`, servirá como punto de entrada principal para la aplicación web. Al pasar `lifespan` el control, se adjunta el controlador de eventos de duración a la aplicación.

1. A continuación, realiza el código auxiliar de los puntos de conexión de la API. El método `api_status` se adjunta a la dirección URL raíz de la API y actúa como mensaje de estado para mostrar que la API está en funcionamiento correctamente. Compilarás el punto de conexión `/chat` más adelante en este ejercicio. Inserta el código siguiente debajo del código para crear el cliente, la base de datos y el contenedor de Cosmos DB:

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. Sobrescribe el bloque main guard en la parte inferior del archivo para iniciar el servidor web `uvicorn` ASGI (interfaz de puerta de enlace de servidor asincrónico) cuando se ejecuta el archivo desde la línea de comandos:

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Guarda el archivo `main.py`. Ahora debería tener un aspecto similar al siguiente, incluidos los métodos `generate_embeddings` y `upsert_product` que agregaste en el ejercicio anterior:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
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
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Para probar rápidamente la API, abre una nueva ventana de terminal integrado en Visual Studio Code.

1. Asegúrate de que has iniciado sesión en Azure con el comando `az login`. Ejecuta lo siguiente en el símbolo del sistema del terminal:

   ```bash
   az login
   ```

1. Completa el proceso de inicio de sesión en tu navegador.

1. Cambia los directorios a `python/07-build-copilot` en el símbolo del sistema del terminal.

1. Asegúrate de que la ventana de terminal integrado se ejecuta en el entorno virtual de Python activándola mediante un comando de la tabla siguiente y seleccionando el comando adecuado para el sistema operativo y el shell.

    | Plataforma | Shell | Comando para activar el entorno virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. En el símbolo del sistema del terminal, cambia los directorios a `api/app` y ejecuta el siguiente comando para ejecutar la aplicación web FastAPI:

   ```bash
   uvicorn main:app
   ```

1. Si no se abre automáticamente, inicia una nueva ventana o pestaña del explorador web y ve a <http://127.0.0.1:8000>.

    Un mensaje de `{"status":"ready"}` en la ventana del explorador indica que la API se está ejecutando.

1. Ve a la interfaz de usuario de Swagger para la API anexando `/docs` al final de la dirección URL: <http://127.0.0.1:8000/docs>.

    > &#128221; La interfaz de usuario de Swagger es una interfaz interactiva basada en web para explorar y probar puntos de conexión de API generados a partir de las especificaciones de OpenAPI. Permite a los desarrolladores y usuarios visualizar, interactuar con y depurar llamadas API en tiempo real, lo que mejora la facilidad de uso y la documentación.

1. Vuelve a Visual Studio Code y detén la aplicación de API presionando **CTRL+C** en la ventana de terminal integrado asociada.

## Incorporación de los datos de productos de Azure Cosmos DB

Al aprovechar los datos de Azure Cosmos DB, el copiloto puede simplificar los flujos de trabajo complejos y ayudar a los usuarios a completar tareas de forma eficaz. El copiloto puede actualizar registros y recuperar valores de búsqueda en tiempo real, lo que garantiza información precisa y oportuna. Esta funcionalidad permite al copiloto proporcionar interacciones avanzadas, mejorando la capacidad de los usuarios para navegar y completar tareas de forma rápida y precisa.

Las funciones permitirán que el copiloto de administración de productos aplique descuentos a los productos de una categoría. Estas funciones serán el mecanismo a través del cual el copiloto recupera e interactúa con los datos de productos de Cosmic Works de Azure Cosmos DB.

1. El copiloto usará una función asincrónica denominada `apply_discount` para agregar y quitar descuentos y precios de venta en productos de una categoría especificada. Inserta el código de función siguiente debajo de la función `upsert_product` cerca de la parte inferior del archivo `main.py`:

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    Esta función realiza una búsqueda en Azure Cosmos DB para extraer todos los productos de una categoría y aplicar el descuento solicitado a esos productos. También calcula el precio de venta del artículo utilizando el descuento especificado y lo inserta en la base de datos.

2. A continuación, agregarás una segunda función denominada `get_category_names`, que el copiloto llamará para ayudarle a saber qué categorías de productos están disponibles al aplicar o quitar los descuentos de los productos. Agrega el método siguiente debajo de la función `apply_discount` en el archivo:

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    La función `get_category_names` consulta el contenedor `Products` para recuperar de la base de datos una lista de nombres de categoría distintos.

3. Guarda el archivo `main.py`.

## Implementación del punto de conexión de chat

El punto de conexión `/chat` de la API de back-end actúa como la interfaz a través de la cual la interfaz de usuario de front-end interactúa con los modelos de Azure OpenAI y los datos internos del producto de Cosmic Works. Este punto de conexión actúa como puente de comunicación, lo que permite que la entrada de la interfaz de usuario se envíe a Azure OpenAI Service, que luego procesa estas entradas mediante modelos de lenguaje sofisticados. A continuación, los resultados se devuelven al front-end, permitiendo así conversaciones inteligentes en tiempo real. Al aprovechar esta configuración, los desarrolladores pueden garantizar una experiencia de usuario fluida y con capacidad de respuesta, mientras que el back-end controla la tarea compleja de procesar lenguaje natural y generar respuestas adecuadas. Este enfoque también admite la escalabilidad y el mantenimiento al desacoplar el front-end de la infraestructura de IA subyacente.

1. Busca el código auxiliar de punto de conexión `/chat` que agregaste anteriormente en el archivo `main.py`.

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    La función acepta `CompletionRequest` como parámetro. El uso de una clase para el parámetro de entrada permite pasar varias propiedades al punto de conexión de API en el cuerpo de la solicitud. La clase `CompletionRequest` se define dentro del módulo de *modelos* e incluye las propiedades de historial máximo, historial de chats y mensajes de usuario. El historial de chats permite al copiloto hacer referencia a aspectos anteriores de la conversación con el usuario, por lo que mantiene el conocimiento del contexto de toda la discusión. La propiedad `max_history` permite definir el número de mensajes de historial que se deben pasar al contexto del LLM. Esto te permite controlar los usos de tokens para el mensaje y evitar los límites de TPM en las solicitudes.

2. Para empezar, elimina la línea `raise NotImplementedError("The chat endpoint is not implemented yet.")` de la función a medida que se inicia el proceso de implementación del punto de conexión.

3. Lo primero que harás dentro del método de punto de conexión de chat es proporcionar un aviso del sistema. Este mensaje define el "rol" de los copilotos, dictando cómo debe interactuar el copiloto con los usuarios, responder a preguntas y aprovechar las funciones disponibles para realizar acciones.

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. A continuación, crea una matriz de mensajes para enviar al LLM, agregando el aviso del sistema, los mensajes del historial de chats y el mensaje de usuario entrante. Este código debe ir directamente debajo de la declaración del aviso del sistema en la función:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    La propiedad `messages` encapsula el historial de la conversación en curso. Incluye toda la secuencia de entradas de usuario y las respuestas de la IA, lo que ayuda al modelo a mantener el contexto. Al hacer referencia a este historial, la IA puede generar respuestas coherentes y contextualmente relevantes, lo que garantiza que las interacciones sigan siendo fluidas y dinámicas. Esta propiedad es fundamental para permitir que la IA comprenda el flujo y los matices de la conversación a medida que avanza.

5. Para permitir que el copiloto use las funciones que definiste anteriormente para interactuar con los datos de Azure Cosmos DB, debes definir una colección de "herramientas". LLM llamará a estas herramientas como parte de su ejecución. Azure OpenAI usa definiciones de funciones para habilitar interacciones estructuradas entre la IA y varias herramientas o API. Cuando se define una función, describe las operaciones que puede realizar, los parámetros necesarios y las entradas necesarias. Para crear una matriz de `tools`, proporciona el código siguiente que contiene definiciones de función para los métodos `apply_discount` y `get_category_names` que definiste anteriormente:

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    Mediante el uso de definiciones de función, Azure OpenAI garantiza que las interacciones entre la IA y los sistemas externos estén bien organizadas, seguras y eficientes. Este enfoque estructurado permite a la IA realizar tareas complejas sin problemas y de forma confiable, mejorando sus funcionalidades generales y la experiencia del usuario.

6. Crea un cliente de Azure OpenAI asincrónico para realizar solicitudes al modelo de finalización de chat:

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. El punto de conexión de chat realizará dos llamadas a Azure OpenAI para aprovechar las llamadas a funciones. La primera proporciona al cliente de Azure OpenAI acceso a las herramientas:

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. La respuesta de esta primera llamada contiene información del LLM sobre las herramientas o funciones que ha determinado que son necesarias para responder a la solicitud. Debes incluir código para procesar las salidas de la llamada de función, insertarlas en el historial de conversaciones de modo que el LLM pueda usarlas para formular una respuesta sobre los datos que contienen esas salidas:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    La llamada a funciones en Azure OpenAI permite la integración sin problemas de las API o herramientas externas directamente en la salida del modelo. Cuando el modelo detecta una solicitud pertinente, construye un objeto JSON con los parámetros necesarios, que después se ejecuta. El resultado se devuelve al modelo, lo que le permite ofrecer una respuesta final completa y enriquecida con datos externos.

9. Para completar la solicitud con los datos enriquecidos de Azure Cosmos DB, debe enviar una segunda solicitud a Azure OpenAI para generar una finalización:

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. Por último, devuelve la respuesta de finalización a la interfaz de usuario:

   ```python
   return final_response.choices[0].message.content
   ```

11. Guarda el archivo `main.py`. El método `generate_chat_completion` del punto de conexión `/chat` debe tener este aspecto:

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## Creación de una interfaz de usuario de chat sencilla

La interfaz de usuario de Streamlit proporciona una interfaz para que los usuarios interactúen con su copiloto.

1. La interfaz de usuario se definirá mediante el archivo `index.py` ubicado en la carpeta `python/07-build-copilot/ui`.

2. Abre el archivo `index.py` y agrega las siguientes instrucciones import en la parte superior del archivo para empezar:

   ```python
   import streamlit as st
   import requests
   ```

3. Configura la página Streamlit definida en el archivo `index.py` agregando la siguiente línea debajo de las instrucciones `import`:

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. La interfaz de usuario interactuará con la API de back-end mediante la biblioteca `requests` para realizar llamadas al punto de conexión `/chat` que definiste en la API. Puedes encapsular la llamada API en un método que espera el mensaje de usuario actual y una lista de mensajes del historial de chats.

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. Define la función `main`, que es el punto de entrada para las llamadas en la aplicación.

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. Por último, agrega un bloque **main guard** al final del archivo:

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. Guarda el archivo `index.py`.

## Prueba del copiloto a través de la interfaz de usuario

1. Vuelve a la ventana de terminal integrado que abriste en Visual Studio Code para el proyecto de API y escribe lo siguiente para iniciar la aplicación de API:

   ```bash
   uvicorn main:app
   ```

2. Abre una nueva ventana de terminal integrado, cambia los directorios a `python/07-build-copilot` para activar el entorno de Python y luego cambia los directorios a la carpeta `ui` y ejecuta lo siguiente para iniciar la aplicación de interfaz de usuario:

   ```bash
   python -m streamlit run index.py
   ```

3. Si la interfaz de usuario no se abre automáticamente en una ventana del explorador, inicia una nueva pestaña o ventana del explorador y ve a <http://localhost:8501> para abrir la interfaz de usuario.

4. En la indicación de chat de la interfaz de usuario, escribe "Aplicar descuento" y envía el mensaje.

    Dado que necesitas proporcionar al copiloto más detalles para actuar, la respuesta debe ser una solicitud para obtener más información, como proporcionar el porcentaje de descuento que te gustaría aplicar y la categoría de productos a los que se debe aplicar el descuento.

5. Para comprender qué categorías están disponibles, pide al copiloto que te proporcione una lista de categorías de productos.

    El copiloto realizará una llamada de función mediante la función `get_category_names` y enriquecerás los mensajes de conversación con esas categorías para que pueda responder en consecuencia.

6. También puedes solicitar un conjunto más específico de categorías, como "Proporcionarme una lista de categorías relacionadas con la ropa".

7. A continuación, pide al copiloto que aplique un descuento del 15 % a todos los productos textiles.

8. Para comprobar que se aplicó el descuento de precio, abre la cuenta de Azure Cosmos DB en Azure Portal, selecciona **Data Explorer** y ejecuta una consulta en el contenedor `Products` para ver todos los productos de la categoría "clothing", como por ejemplo:

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    Observa que cada elemento de los resultados de la consulta tiene un valor `discount` de `0.15` y `sale_price` debe ser un 15 % menor que el original`price`.

9. Vuelve a Visual Studio Code y detén la aplicación de API presionando **CTRL+C** en la ventana del terminal que ejecuta la aplicación. Puedes dejar la interfaz de usuario funcionando.

## Integración del vector de búsqueda

Hasta ahora, has dado al copiloto la capacidad de realizar acciones para aplicar descuentos a los productos, pero todavía no tiene conocimiento de los productos almacenados en la base de datos. En esta tarea, agregarás funcionalidades de vector de búsqueda que te permitirán pedir productos con ciertas cualidades y buscar productos similares en la base de datos.

1. Vuelva al archivo `main.py` de la carpeta `api/app` y proporciona un método para realizar las búsquedas vectoriales en el contenedor `Products` de la cuenta de Azure Cosmos DB. Puedes insertar este método debajo de las funciones existentes cerca de la parte inferior del archivo.

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. A continuación, crea un método denominado `get_similar_products` que servirá como la función usada por el LLM para realizar búsquedas vectoriales en la base de datos:

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    La función `get_similar_products` realiza llamadas asincrónicas a la función `vector_search` que definiste anteriormente, así como la función `generate_embeddings` que creaste en el ejercicio anterior. Las incrustaciones se generan en el mensaje de usuario entrante para permitir que se compare con los vectores almacenados en la base de datos mediante la función `VectorDistance` integrada en Cosmos DB.

3. Para permitir que el LLM use las nuevas funciones, debes actualizar la matriz `tools` que creaste anteriormente, agregando una definición de función para el método `get_similar_products`:

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. También debes agregar código para controlar la salida de la nueva función. Agrega la siguiente condición `elif` al bloque de código que controla las salidas de la llamada de función:

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    El bloque completado con ahora tiene el siguiente aspecto:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. Por último, debes actualizar la definición del aviso del sistema para proporcionar instrucciones sobre cómo realizar búsquedas vectoriales. Inserta lo siguiente al final de `system_prompt`:

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    El aviso del sistema actualizado será similar al siguiente:

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. Guarde el archivo `main.py`.

## Prueba de la característica de vector de búsqueda

1. Reinicia la aplicación de API ejecutando lo siguiente en la ventana de terminal integrado abierta para esa aplicación en Visual Studio Code:

   ```bash
   uvicorn main:app
   ```

2. La interfaz de usuario debe seguir ejecutándose, pero si la detuviste, vuelve a la ventana del terminal integrado para ella y ejecuta:

   ```bash
   python -m streamlit run index.py
   ```

3. Vuelve a la ventana del navegador que ejecuta la interfaz de usuario y, en la indicación de chat, escribe lo siguiente:

   ```bash
   Tell me about the mountain bikes in stock
   ```

    Esta pregunta devolverá algunos productos que coincidan con la búsqueda.

4. Prueba algunas otras búsquedas, como "Show me durable pedals", "Provide a list of 5 stylish jerseys" y "Give me details about all gloves suitable for warm weather riding".

    Para las dos últimas consultas, observa que los productos contienen el 15 % de descuento y el precio de venta que aplicaste anteriormente.
