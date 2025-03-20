---
lab:
  title: "01: Conexión a Azure Cosmos\_DB for NoSQL con el SDK"
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# Conexión a Azure Cosmos DB for NoSQL con el SDK

El SDK de Azure para Python es un conjunto de bibliotecas cliente que proporciona una interfaz de desarrollador coherente para interactuar con muchos servicios de Azure. Las bibliotecas cliente son paquetes que se usarían para consumir estos recursos e interactuar con ellos.

En este laboratorio, te conectarás a una cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para Python.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#install-the-azure-cosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Instalación de la biblioteca azure-cosmos

La biblioteca **azure-cosmos** está disponible en **PyPI** para facilitar la instalación en los proyectos de Python.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/01-sdk-connect**.

1. Abre el menú contextual de la carpeta **python/01-sdk-connect** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **python/01-sdk-connect**.

1. Creación y activación de un entorno virtual para administrar las dependencias:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Ejecuta el comando siguiente para instalar el paquete [azure-cosmos][pypi.org/project/azure-cosmos]:

   ```bash
   pip install azure-cosmos
   ```

1. Instala la biblioteca [azure-identity][pypi.org/project/azure-identity], que nos permite usar la autenticación de Azure para conectarnos al área de trabajo de Azure Cosmos DB mediante el siguiente comando:

   ```bash
   pip install azure-identity
   ```

1. Cierra el terminal integrado.

## Uso de la biblioteca azure-cosmos

Una vez importada la biblioteca de Azure Cosmos DB del SDK de Azure para Python, puedes usar inmediatamente sus clases para conectarte a una cuenta de Azure Cosmos DB for NoSQL. La clase **CosmosClient** es la clase principal que se usa para establecer la conexión inicial a una cuenta de Azure Cosmos DB for NoSQL.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **python/01-sdk-connect**.

1. Abre el archivo Python en blanco llamado **script.py**.

1. Agrega la siguiente instrucción `import` para importar las clases **CosmosClient** y **DefaultAzureCredential**:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. Agrega las variables denominadas **endpoint** y **credential** y establece el valor de **endpoint** en el **punto de conexión** de la cuenta de Azure Cosmos DB que creaste anteriormente. La variable **credential** debe establecerse en una nueva instancia de la clase **DefaultAzureCredential**:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https://dp420.documents.azure.com:443/**, la instrucción sería: **endpoint = "https://dp420.documents.azure.com:443/".**

1. Agrega una nueva variable denominada **client** e inicialízala como una nueva instancia de la clase **CosmosClient** mediante las variables **endpoint** y **credential**:

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. Agrega una función denominada **main** para leer e imprimir las propiedades de la cuenta:

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. Ahora, el aspecto del archivo **script.py** será parecido a este:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **Guarda** el archivo **script.py**.

## Prueba del script

Ahora que el código de Python para conectarse a la cuenta de Azure Cosmos DB for NoSQL está completo, puedes probar el script. Este script imprimirá el nivel de coherencia predeterminado y el nombre de la primera región grabable. Al crear la cuenta, especificaste una ubicación y deberías esperar ver ese mismo valor de ubicación impreso como resultado de este script.

1. En **Visual Studio Code**, abre el menú contextual de la carpeta **ython/01-sdk-connect** y luego selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

1. Antes de ejecutar el script, debes iniciar sesión en Azure mediante el comando `az login`. En la ventana de terminal, ejecuta lo siguiente:

   ```bash
   az login
   ```

1. Ejecuta el script mediante el comando `python`:

   ```bash
   python script.py
   ```

1. El script generará ahora el nivel de coherencia predeterminado y la primera región grabable. Por ejemplo, si el nivel de coherencia predeterminado para la cuenta es **Sesión** y la primera región grabable es **Este de EE. UU.**, el script generaría:

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
