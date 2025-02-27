---
title: "07.4: Implementación de RAG con LangChain y el vector de búsqueda de Azure\_Cosmos\_DB for NoSQL"
lab:
  title: "07.4: Implementación de RAG con LangChain y el vector de búsqueda de Azure\_Cosmos\_DB for NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 13
parent: Python SDK labs
---

# Implementación de RAG con LangChain y el vector de búsqueda de Azure Cosmos DB for NoSQL

Las funcionalidades de orquestación de LangChain aportan una gran cantidad de ventajas sobre la implementación de la integración del LLM de copiloto mediante el cliente de Azure OpenAI directamente. LangChain permite una integración más fluida con varios orígenes de datos, incluido Azure Cosmos DB, lo que permite un vector de búsqueda eficaz que mejora el proceso de recuperación. LangChain ofrece herramientas sólidas para administrar y optimizar flujos de trabajo, lo facilitando la creación de aplicaciones complejas con componentes modulares y reutilizables. Esta flexibilidad no solo simplifica el desarrollo, sino que también garantiza la escalabilidad y el mantenimiento.

En este laboratorio, mejorarás el copiloto mediante la transición del punto de conexión `/chat` de la API desde el uso del cliente de Azure OpenAI para aprovechar las eficaces funcionalidades de orquestación de LangChain. Este cambio permitirá una recuperación de datos más eficaz y un rendimiento mejorado mediante la integración de la funcionalidad del vector de búsqueda con Azure Cosmos DB for NoSQL. Tanto si quieres optimizar el proceso de recuperación de información de la aplicación como explorar simplemente el potencial de RAG, este módulo te guiará a través de la conversión fluida, mostrando cómo LangChain puede simplificar y elevar las funcionalidades de la aplicación. Vamos a embarcarnos en este viaje para desbloquear nuevas eficiencias e información con LangChain y Azure Cosmos DB.

> &#128721; Los ejercicios anteriores de este módulo son un requisito previo para este laboratorio. Si todavía necesitas completar cualquiera de esos ejercicios, hazlos antes de continuar, ya que proporcionan la infraestructura necesaria y el código de inicio para este laboratorio.

## Instalación de las bibliotecas LangChain

1. Con Visual Studio Code, abre la carpeta en la que has clonado el repositorio de código de laboratorio para el módulo de aprendizaje **Compilación de copilotos con Azure Cosmos DB**.

2. En el panel **Explorer** de Visual Studio Code, ve a la carpeta **python/07-build-copilot** y abre el archivo `requirements.txt` que se encuentra en ella.

3. Actualiza el archivo `requirements.txt` para incluir las bibliotecas LangChain necesarias:

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. Inicia una nueva ventana de terminal integrado en Visual Studio Code y cambia los directorios a `python/07-build-copilot`.

5. Asegúrate de que la ventana del terminal integrado se ejecuta en el entorno virtual de Python activándola mediante el comando adecuado para el sistema operativo y el shell de la tabla siguiente:

    | Plataforma | Shell | Comando para activar el entorno virtual |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. Actualiza el entorno virtual con las bibliotecas LangChain ejecutando el siguiente comando en el símbolo del sistema del terminal integrado:

   ```bash
   pip install -r requirements.txt
   ```

7. Cierra el terminal integrado.

## Actualización de la API de back-end

En el laboratorio anterior, ejecutaste un patrón RAG mediante el cliente de Azure OpenAI y los datos de Azure Cosmos DB. Ahora, actualizarás la API de back-end para usar un agente LangChain con herramientas para realizar las mismas acciones.

El uso de LangChain para interactuar con los modelos de lenguaje implementados en Azure OpenAI Service es algo más simple desde el punto de vista del código...

1. Quita la instrucción de importación `from openai import AzureOpenAI` en la parte superior del archivo `main.py`. Esa biblioteca cliente ya no es necesaria, ya que todas las interacciones con Azure OpenAI pasarán por las clases proporcionadas por LangChain.

2. Elimina las siguientes instrucciones de importación en la parte superior del archivo `main.py`, ya que ya no serán necesarias:

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### Actualización del punto de conexión de incrustación

1. Importa la clase `AzureOpenAIEmbeddings` de la biblioteca `langchain_openai` agregando la siguiente instrucción de importación en la parte superior del archivo `main.py`:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. Busca el método `generate_embeddings` en el archivo y sobrescribe con lo siguiente, que usa la clase `AzureOpenAIEmbeddings` para controlar las interacciones con Azure OpenAI:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    La clase `AzureOpenAIEmbeddings` proporciona una interfaz para interactuar con la API de incrustaciones de Azure OpenAI y devolver un objeto de respuesta simplificado que contiene solo el vector generado.

### Actualización del punto de conexión de chat

1. Actualiza la instrucción de importación `lanchain_openai` para anexar la clase `AzureChatOpenAI`:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. Importa los siguientes objetos LangChain adicionales que se usarán al compilar el punto de conexión `/chat` revisado:

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. El historial de chats se insertará en la conversación de copiloto de forma diferente mediante un agente de LangChain, por lo que eliminarás las líneas de código inmediatamente después de la definición `system_prompt`. La línea que debes eliminar es:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. En lugar del código que acabas de eliminar, define un objeto `prompt` mediante la clase `ChatPromptTemplate` de LangChain:

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    `ChatPromptTemplate` se está creando con varios componentes en un orden específico. Así es como encajan esas piezas:

    - **Mensaje del sistema**: usa `system_prompt` para proporcionar un rol al copiloto, proporcionando instrucciones sobre cómo el asistente debe comportarse e interactuar con los usuarios.
    - **Historial de chats**: permite que `chat_history`, que contiene una lista de mensajes anteriores en la conversación, se incorporen al contexto en el que funciona el LLM.
    - **Entrada de usuario**: mensaje de usuario actual.
    - **Agente Scratchpad**: permite notas intermedias o pasos realizados por el agente.

    La indicación resultante proporciona una entrada estructurada para el agente de IA conversacional, lo que le ayuda a generar una respuesta basada en el contexto especificado.

1. A continuación, reemplaza la definición de matriz `tools` por la siguiente, que usa la clase `StructuredTool` de LangChain para extraer definiciones de función en el formato adecuado:

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    El método `StructuredTool.from_function` de LangChain crea una herramienta a partir de una función determinada mediante los parámetros de entrada y la descripción docstring de la función. Para usarlo con métodos asincrónicos, específica pasar el nombre de la función al parámetro de entrada `coroutine`.

    En Python, docstring (abreviatura para cadena de documentación) es un tipo especial de cadena que se usa para documentar una función, un método, una clase o un módulo. Proporciona una manera cómoda de asociar documentación con código de Python y normalmente se escribe entre comillas triples (""" o '''). Los documentos se colocan inmediatamente después de la definición de la función (o método, clase o módulo) que documentan.

    El uso de esta función automatiza la creación de las definiciones de función JSON que has tenido que crear manualmente mediante el cliente de Azure OpenAI, lo que simplifica el proceso de llamada a funciones.

1. Elimina todo el código entre la definición de matriz `tools` que completaste anteriormente y la instrucción `return` al final de la función. Con el cliente de Azure OpenAI, tenías que realizar dos llamadas al modelo de lenguaje. La primera, para permitir que determine qué llamadas de función, si las hay, debe realizar para aumentar la solicitud y la segunda para solicitar una finalización de RAG. Entre ellas, tenías que usar código para inspeccionar la respuesta de la primera llamada para determinar si se requerían llamadas de función y luego escribir código para "controlar" las llamadas de esas funciones. A continuación, tenías que insertar la salida de esas llamadas de función en los mensajes que se envían al LLM, por lo que podría tener la indicación enriquecida para razonar al formular una respuesta de finalización. LangChain simplifica considerablemente el proceso de llamar a un LLM mediante un patrón RAG, como verás a continuación. Este es el código que debes quitar:

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. Trabajando desde justo debajo de la definición de matriz `tools`, crea una referencia a la API de Azure OpenAI mediante la clase `AzureChatOpenAI` en LangChain:

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. Para permitir que el agente de LangChain interactúe con las funciones que has definido, crearás un agente mediante el método `create_openai_functions_agent`, al que proporcionarás el objeto `AzureChatOpenAI`, la matriz `tools` y el objeto `ChatPromptTemplate`:

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    La función `create_openai_functions_agent` de LangChain crea un agente que puede llamar a funciones externas para realizar tareas mediante un modelo de lenguaje y herramientas especificados. Esto permite la integración de varios servicios y funcionalidades en el flujo de trabajo del agente, lo que proporciona flexibilidad y funcionalidades mejoradas.

1. En LangChain, se usa la clase `AgentExecutor` para administrar el flujo de ejecución de los agentes, como el que creaste con el método `create_openai_functions_agent`. Controla el procesamiento de entradas, la invocación de herramientas o modelos y el control de salidas. Usa el código siguiente para crear un ejecutor de agente para el agente:

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    `AgentExecutor` garantiza que todos los pasos necesarios para generar una respuesta se ejecuten en el orden correcto. Abstrae las complejidades de ejecución de los agentes, lo que proporciona una capa adicional de funcionalidad y estructura, y facilita la compilación, la administración y el escalado de agentes sofisticados.

1. Usarás el método `invoke` del ejecutor del agente para enviar el mensaje de usuario entrante al LLM. También incluirás el historial de chats. Inserta el código siguiente justo debajo de la definición `agent_executor`:

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   Los tokens `input` y `chat_history` se definieron en el objeto prompt creado mediante `ChatPromptTemplate`. Con el método `invoke`, estos se insertarán en la indicación, lo que permite al LLM usar esa información al crear una respuesta.

1. Por último, actualiza la instrucción "return" para usar `output` del objeto de finalización del agente:

   ```python
   return completion["output"]
   ```

1. Guarda el archivo `main.py`. La función de punto de conexión `/chat` actualizada debe tener este aspecto:

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## Inicio de las aplicaciones de API e interfaz de usuario

1. Para empezar, abre una nueva ventana de terminal integrado en Visual Studio Code.

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

1. Abre una nueva ventana de terminal integrado, cambia los directorios a `python/07-build-copilot` para activar el entorno de Python y luego cambia los directorios a la carpeta `ui` y ejecuta lo siguiente para iniciar la aplicación de interfaz de usuario:

   ```bash
   python -m streamlit run index.py
   ```

1. Si la interfaz de usuario no se abre automáticamente en una ventana del explorador, inicia una nueva pestaña o ventana del explorador y ve a <http://localhost:8501> para abrir la interfaz de usuario.

## Prueba del copiloto

1. Antes de enviar mensajes a la interfaz de usuario, vuelve a Visual Studio Code y selecciona la ventana de terminal integrado asociada a la aplicación de API. En esta ventana, verás la salida "detallada" generada por el ejecutor del agente LangChain, que proporciona información sobre cómo LangChain controla las solicitudes que envías. Presta atención a la salida de esta ventana cuando envíes las solicitudes siguientes y vuelve a revisar después de cada llamada.

1. En la indicación de chat de la interfaz de usuario, escribe "Apply a discount" y envía el mensaje.

    Debes recibir una respuesta que solicite el porcentaje de descuento que te gustaría hacer y para qué categoría de producto.

1. Respuesta, "Guantes".

    Recibirás una respuesta en la que se te preguntará qué porcentaje de descuento deseas aplicar a la categoría "Guantes".

1. Envía un mensaje de "25 %".

    Deberías obtener una respuesta de "Se ha aplicado correctamente un descuento del 25 % a todos los productos de la categoría "Guantes"".

1. Pide al copiloto: "muéstrame todos los guantes".

    En la respuesta, deberías ver una lista de todos los guantes en la base de datos, que incluirá el precio de descuento del 25 %.

1. Por último, pregunta: "¿Qué guantes son los mejores para el frío?" para realizar una búsqueda vectorial. Esto implica una llamada de función al método `get_similar_items`, que luego llama al método `generate_embeddings` que actualizaste para usar una implementación de LangChain y la función `vector_search`.

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.
