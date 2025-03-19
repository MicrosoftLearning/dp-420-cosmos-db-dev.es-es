---
lab:
  title: 02 - Configuración del SDK de JavaScript de Azure Cosmos DB para el desarrollo sin conexión
  module: Configure the Azure Cosmos DB for NoSQL SDK
---

# Configuración del SDK de Python de Azure Cosmos DB para el desarrollo sin conexión

El emulador de Azure Cosmos DB es una herramienta local que emula el servicio Azure Cosmos DB para desarrollo y pruebas. El emulador admite la API de NoSQL y se puede usar en lugar del servicio en la nube al desarrollar código mediante el SDK de Azure para Python.

En este laboratorio, te conectarás al emulador de Azure Cosmos DB desde el SDK de Azure para Python.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio para **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Inicio del emulador de Azure Cosmos DB

Si usas un entorno de laboratorio hospedado, ya deberías tener instalado el emulador. Si no es así, consulta las [instrucciones de instalación](https://docs.microsoft.com/azure/cosmos-db/local-emulator) para instalar el emulador de Azure Cosmos DB. Una vez iniciado el emulador, puedes recuperar la cadena de conexión y usarla para conectarte al emulador mediante el SDK de Azure para Python.

> &#128161; Opcionalmente, puedes instalar el [nuevo emulador de Azure Cosmos DB basado en Linux (en versión preliminar)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux), que está disponible como contenedor Docker. Admite la ejecución en una amplia variedad de procesadores y sistemas operativos.

1. Inicia el **emulador de Azure Cosmos DB**.

    > 💡 Si usas Windows, el emulador de Azure Cosmos DB está anclado tanto a la barra de tareas de Windows como al menú Inicio. Si el emulador no se inicia desde los iconos anclados, intenta abrirlo haciendo doble clic en el archivo **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe**.

1. Espera a que el emulador abra automáticamente el explorador predeterminado y ve a la página de aterrizaje **https://localhost:8081/_explorer/index.html**.

1. En el panel **Inicio rápido**, anota la **cadena de conexión principal**. Necesitarás esta cadena de conexión más adelante.

> &#128221; A veces, la página de aterrizaje no se carga correctamente, aunque el emulador se esté ejecutando. Si esto sucede, puedes usar la cadena de conexión conocida para conectarte al emulador. La cadena de conexión conocida es:`AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Instalación de la biblioteca azure-cosmos

La biblioteca **azure-cosmos** está disponible en **PyPI** para facilitar la instalación en los proyectos de Python.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/02-sdk-offline**.

1. Abre el menú contextual de la carpeta **python/02-sdk-offline** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/02-sdk-offline**.

1. Creación y activación de un entorno virtual para administrar las dependencias:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Ejecuta el comando siguiente para instalar el paquete [azure-cosmos][pypi.org/project/azure-cosmos]:

   ```bash
   pip install azure-cosmos
   ```

## Conexión al emulador desde el SDK de Python

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/02-sdk-offline**.

1. Abre el archivo Python en blanco llamado **script.py**.

1. Agrega el código siguiente para conectarte al emulador, crea una base de datos e imprime su id.:

   ```python
   from azure.cosmos import CosmosClient, PartitionKey
   
   # Connection string for the Azure Cosmos DB Emulator
   connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
    
   # Initialize the Cosmos client
   client = CosmosClient.from_connection_string(connection_string)
    
   # Create a database
   database_name = "cosmicworks"
   database = client.create_database_if_not_exists(id=database_name)
    
   # Print the database ID
   print(f"New Database: Id: {database.id}")
   ```

1. **Guarda** el archivo **script.py**.

## Ejecuta el script.

1. Usa la misma ventana de terminal en **Visual Studio Code** que usaste para configurar el entorno de Python para este laboratorio. Abre el menú contextual de la carpeta **python/02-sdk-offline** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

1. Ejecuta el script mediante el comando `python`:

   ```bash
   python script.py
   ```

1. El script crea una base de datos denominada `cosmicworks` en el emulador. Deberías ver un resultado similar al siguiente:

   ```text
   New Database: Id: cosmicworks
   ```

## Creación y visualización de un nuevo contenedor

Puedes ampliar el script para crear un contenedor dentro de la base de datos.

### Código actualizado

1. Modifica el archivo `script.py` para agregar el código siguiente en la parte inferior del archivo para crear un contenedor:

   ```python
   # Create a container
   container_name = "products"
   partition_key_path = "/categoryId"
   throughput = 400
    
   container = database.create_container_if_not_exists(
       id=container_name,
       partition_key=PartitionKey(path=partition_key_path),
       offer_throughput=throughput
   )
    
   # Print the container ID
   print(f"New Container: Id: {container.id}")
   ```

### Ejecución del script actualizado

1. Ejecuta el script actualizado con el siguiente comando:

   ```bash
   python script.py
   ```

1. El script crea un contenedor denominado `products` en el emulador. Deberías ver un resultado similar al siguiente:

   ```text
   New Container: Id: products
   ```

### Verificación de los resultados

1. Cambia al navegador donde está abierto la instancia de Data Explorer del emulador.

1. Actualiza la **API de NoSQL** para observar la nueva base de datos **cosmicworks** y el contenedor de **products**.

## Parada del emulador de Azure Cosmos DB

Es importante detener el emulador cuando hayas terminado de usarlo para liberar los recursos el sistema. Sigue los pasos siguientes en base a tu sistema operativo:

### En macOS o Linux:

Si has iniciado el emulador en una ventana de terminal, sigue estos pasos:

1. Busca la ventana de terminal donde se ejecuta el emulador.

1. Presiona `Ctrl + C` para finalizar el proceso del emulador.

Como alternativa, si necesitas detener manualmente el proceso del emulador:

1. Abra una nueva ventana de terminal.

1. Para encontrar el proceso del emulador, usa el comando siguiente:

   ```bash
   ps aux | grep CosmosDB.Emulator
   ```

Identifica el **PID** (id. de proceso) del proceso del emulador en la salida. Usa el comando kill para finalizar el proceso del emulador:

```bash
kill <PID>
```

### En Windows:

1. Busca el icono del emulador de Azure Cosmos DB en la bandeja del sistema de Windows (cerca del reloj de la barra de tareas).

1. Haz clic con el botón derecho en el icono del emulador para abrir el menú contextual.

1. Selecciona **Salir** para apagar el emulador.

> 💡 Todas las instancias del emulador pueden tardar un minuto en salir.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
