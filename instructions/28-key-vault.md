---
lab:
  title: "Almacenamiento de claves de cuenta de Azure Cosmos\_DB for NoSQL en Azure Key Vault"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Almacenamiento de claves de cuenta de Azure Cosmos DB for NoSQL en Azure Key Vault

Agregar un código de conexión de cuenta de Azure Cosmos DB a la aplicación es tan sencillo como proporcionar el URI y las claves de la cuenta. Esta información de seguridad a veces puede codificarse de forma rígida en el código de la aplicación. Sin embargo, si la aplicación se implementa en Azure App Service, puede guardar la información de conexión cifrada en Azure Key Vault.

En este laboratorio, cifraremos y almacenaremos la cadena de conexión de la cuenta de Azure Cosmos DB en Azure Key Vault. A continuación, crearemos una aplicación web de Azure App Service que recuperará esas credenciales de Azure Key Vault. La aplicación usará estas credenciales y se conectará a la cuenta de Azure Cosmos DB. A continuación, la aplicación creará algunos documentos en los contenedores de cuentas de Azure Cosmos DB y devolverá su estado a una página web.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code] [code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, ***CIERRE*** *Visual Studio Code*. Más adelante lo abriremos apuntando directamente a la carpeta **28-key-vault**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccione cuál de las API quiere que admita la cuenta (por ejemplo, **API Mongo** o **API NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave. Use el punto de conexión y la clave para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante programación. Use el punto de conexión y la clave en las cadenas de conexión del SDK de Azure para .NET o cualquier otro SDK.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicia sesión en el portal con las credenciales de Microsoft asociadas a tu suscripción.

1. Selecciona **+ Crear un recurso**, busca *Cosmos DB* y, a continuación, crea un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Tipo de carga de trabajo** | **Aprendizaje** |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Selecciona un grupo de recursos ya existente o crea un nuevo* |
    | **Nombre de cuenta** | *Escribe un nombre único global*. |
    | **Ubicación** | *Selecciona cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que te impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espera a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Cosmos DB** recién creado y, después, ve al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente el campo **CADENA DE CONEXIÓN PRINCIPAL**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

## Creación de una instancia de Azure Key Vault y almacenamiento de las credenciales de la cuenta de Azure Cosmos DB como un secreto

Antes de crear nuestra aplicación web, protegeremos la cadena de conexión de la cuenta de Azure Cosmos DB copiándolas en un *secreto* cifrado de *Azure Key Vault*. Vamos a hacerlo.

1. En una nueva pestaña del explorador, vaya a Azure Portal y abra la página **Almacenes de claves**.

1. Agregue un almacén seleccionando el botón ***+ Crear*** y rellene el almacén con la siguiente configuración, *dejando todas las opciones restantes en sus valores predeterminados* y, a continuación, seleccione para crear el almacén:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree un nuevo* |
    | **Nombre del almacén de claves** | *Escriba un nombre único global*. |
    | **Región** | *seleccione cualquier región disponible* |
    | **Configuración de acceso/modelo de permisos** | *Directiva de acceso de Vault* |
    | **Configuración de acceso/Directivas de acceso** | *Active la casilla nombre de usuario actual* |

    > &#128221; Tenga en cuenta que, en un entorno de producción, es probable que seleccione el control RBAC en lugar de la directiva de acceso al almacén y el administrador probablemente le asignará el rol RBAC adecuado para limitar el acceso a Key Vault.

1. Una vez creado el almacén, vaya al almacén.

1. En la sección *Objetos*, seleccione **Secretos**.

1. Seleccione **+ Generar/Importar** para cifrar la cadena de conexión de credenciales y rellene los valores de *secreto* con la siguiente configuración, *dejando todas las opciones restantes en sus valores predeterminados* y, a continuación, seleccione para crear el secreto:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Opciones de carga** | *Manual* |
    | **Nombre** | *El nombre con el que etiquetará el secreto* |
    | **Valor** | *Este campo es el campo más importante que se va a rellenar. Copie este valor de la cadena de conexión principal de la sección clave de la cuenta de Azure Cosmos DB. Este valor se convertirá en un secreto.* |
    | **Habilitado** | *Sí* |
 
1. En Secretos, ahora debería ver el nuevo secreto en la lista. Necesitamos obtener el *identificador secreto* que agregaremos al código de nuestra aplicación web. Seleccione el **secreto** que creó.

1. Azure Key Vault permite crear varias versiones del secreto, pero para este laboratorio solo necesitamos una versión. Seleccione la **versión actual**.

1. Registre el valor del campo **Identificador secreto**. Este valor es lo que usaremos en el código de la aplicación para obtener el secreto de Key Vault.  Observe que este valor es una dirección URL. Hay un paso más que necesitamos para que este secreto funcione correctamente, pero lo haremos un poco más tarde.

## Creación de una aplicación web en Azure App Service

Crearemos una aplicación web que se conectará a la cuenta de Azure Cosmos DB y crearemos algunos contenedores y documentos. No codificaremos de forma rígida las *credenciales* de Azure Cosmos DB en esta aplicación, sino que codificaremos de forma rígida el **identificador secreto** del almacén de claves. Veremos cómo este identificador es inútil sin los derechos adecuados asignados a la aplicación web en la capa de Azure. Comencemos a codificar.



1. Abra **Visual Studio Code**.  Abra la carpeta **28-key-vault**; para ello, seleccione Archivo >Abrir carpeta y vaya a la carpeta **28-key-vault**.

    > &#128221; Tenga en cuenta que solo debería ver la carpeta **28-key-vault** y sus archivos y subcarpetas en el árbol del **Explorador**. Si puede ver todo el repositorio de GitHub que clonamos anteriormente, ***cierre Visual Studio Code*** y vuelva a abrirlo directamente en la carpeta **28-key-vault**.  La aplicación web no funcionará correctamente si ese directorio no es el directorio raíz de los proyectos, por lo que asegúrese de que solo puede ver la carpeta **28-key-vault** y sus archivos y subcarpetas en el árbol del **Explorador**.

1. Abra el menú contextual de la carpeta **28-key-vault** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **28-key-vault**.

1. Vamos a crear un shell de aplicación web MVC. Reemplazaremos un par de archivos generados en un momento. Ejecute el siguiente comando para crear la aplicación web:

    ```
    dotnet new mvc
    ```


    > &#128221;Este comando creó el shell de una aplicación web, por lo que agregó varios archivos y directorios. Ya tenemos un par de archivos con todo el código que necesitamos. 

1. Reemplace los archivos **.\Controllers\HomeController.cs** y **.\Views\Home\Index.cshtml** por sus respectivos archivos del directorio **.\KeyvaultFiles**.

1. Una vez que reemplace los archivos ***ELIMINE*** el directorio **.\KeyvaultFiles**.

## Importación de varias bibliotecas que faltan en el script de .NET

La CLI de .NET incluye un comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar paquetes desde una fuente de paquetes preconfigurada. Una instalación de .NET usa NuGet como fuente de paquetes predeterminada.

1. Agrega el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Agrega el paquete [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.3] de NuGet mediante el siguiente comando:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.3
    ```

1. Agregue el paquete [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] de NuGet mediante el siguiente comando:

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. Agregue el paquete [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.6.2
    ```

## Adición del identificador secreto a la aplicación web

1. En Visual Studio, abra el archivo `.\Controllers\HomeControler.cs`

1. La función definida por el usuario **GetKeyVaultSecret** obtendrá el secreto de la cuenta de Azure Cosmos DB. La función se inicia en *la línea 98* y debe tener un aspecto similar al siguiente script.

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. Vamos a revisar las llamadas importantes que realiza esta función.

    - En *la línea 100*, definimos el token de la aplicación web actual. Este token se proporcionará a Azure Key Vault para identificar qué aplicación intenta acceder al almacén. 
    - En *la línea 104-105*, preparamos el *cliente de Key Vault* que se conectará a Azure Key Vault. Observe que se envía el token de aplicación web como parámetro. 
    - En *las líneas 107-108*, proporcionamos al cliente de Key Vault la dirección URL de nuestro **identificador secreto** que devolvería el secreto almacenado en ese almacén de claves. 

1.  Para poder implementar nuestra aplicación web, todavía es necesario enviar la dirección URL del **identificador secreto**.  En *la línea 107*, reemplace la cadena ***<Key Vault Secret Identifier>*** por la dirección URL del **identificador secreto** que registramos en la sección *secreta* y guarde el archivo.

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## (Opcional) Instalación de la extensión de Azure App Services

En Visual Studio, si abre la paleta de comandos (**CTRL+MAYÚS+P**) y no devuelve nada al buscar comandos de Recursos de aplicación de Azure, es necesario instalar la extensión.

1. En el menú izquierdo de Visual Studio Code, seleccione la opción **Extensiones**.

1. En la barra de búsqueda, busque Azure App Service y selecciónelo.

1. Seleccione el botón Instalar para instalarlo.

1. Cierre la pestaña **Extensiones** y vuelva al código.

## Implemente su aplicación en Azure App Services

El resto del código es sencillo, obtenga la cadena de conexión, conéctese a Azure Cosmos DB y agregue algunos documentos. La aplicación también debe proporcionarnos algunos comentarios sobre cualquier problema. No es necesario realizar más cambios después de implementar la aplicación. Empecemos. 

> &#128221; La mayoría de los pasos siguientes se ejecutarán en la paleta de comandos (**CTRL+MAYÚS+P**) en el centro superior de la pantalla de Visual Studio. 

1. En Visual Studio Code, abra la paleta de comandos y busque ***Azure App Service: Crear nueva Aplicación web… (Avanzado)***

1. Seleccione ***Iniciar sesión en Azure…***. Esta opción abrirá una ventana del explorador web, seguirá el proceso de inicio de sesión y cerrará el explorador cuando haya terminado y vuelva a Visual Studio Code.

1. (Opcional) Si solicita la suscripción, seleccione la suscripción.

1. Escriba un nombre único global para la aplicación web.

1. Seleccione un grupo de recursos existente o cree uno si lo necesita.

1. Selecciona **.NET 8 (LTS)**.

1. Seleccione **Windows**.

1. Seleccione una ubicación disponible.

1. Seleccione  **+ Crear un nuevo plan de App Service**.

1. Acepte el nombre predeterminado del plan de App Service (debe ser el mismo que el nombre de la aplicación web) o elija un nombre nuevo.

1. Seleccione **Gratis (F1) Pruebe Azure sin costo alguno**.

1. Seleccione **Omitir por ahora** para Application Insights.

1. La implementación debería ejecutarse ahora con una barra de estado en la esquina inferior derecha. 

1. Cuando se le solicite, seleccione **Implementar**.

1. Seleccione **Examinar** y debe estar dentro de la carpeta **28-key-vault**, seleccione esa carpeta.

1. Aparecerá un elemento emergente con el mensaje **Configuración necesaria para implementar en "28-key-vault"**, seleccione el botón Agregar configuración.  Esta opción creará la carpeta `.vscode` que falta.

    > &#128221; Muy importante, si este elemento emergente no aparece en la primera implementación de la aplicación, la carga en Azure App Services estará incompleta. La implementación se realizará correctamente, pero el sitio web siempre devolverá el mensaje *No tiene permiso para ver este directorio o página.* Lo más probable es que Visual Studio Code se abrió en el repositorio clonado de GitHub en lugar de solo la  carpeta **28-key-vault**.

1. Seleccione **Sí** cuando se le pida que implemente siempre en esa área de trabajo.

1. Seleccione **Examinar sitio web** cuando se le solicite.  Como alternativa, abra un explorador y vaya a **`https://<yourwebappname>.azurewebsites.net`**. En cualquier caso, tenemos un problema. Verá un mensaje definido por el usuario en nuestra página web. El mensaje debe ser, **No se pudo acceder a Key Vault** con un mensaje de error extendido. Vamos a solucionarlo.

## Permitir que nuestra aplicación use una identidad administrada

El primer problema que necesitamos corregir es permitir que nuestra aplicación use una identidad administrada. El uso de una identidad administrada permitirá a nuestra aplicación usar servicios de Azure como Azure Key Vault.

1. Abra el explorador e inicie sesión en Azure Portal.

1. Abra la página **App Services**. Debe aparecer el nombre de la aplicación web, selecciónelo.

1. En la sección *Configuración*, seleccione **Identidad**.

1. En Estado, seleccione **Activado** y **Guardar**.  Seleccione **Sí** si se le pide que habilite la *identidad administrada asignada*.

1. Probemos la aplicación web de nuevo.  En el explorador, vaya a **`https://<yourwebappname>.azurewebsites.net`**.

1. Todavía hay un problema. Aunque el primer mensaje es un mensaje definido por el usuario que nuestro programa está enviando, el segundo es un sistema generado. Lo que significa el segundo mensaje es que se ha concedido acceso a la conexión al almacén de claves, pero no se ha concedido acceso para ver el secreto dentro del almacén.  Vamos a establecer una configuración final para corregir este problema.

## Conceder a nuestra aplicación web una directiva de acceso a los secretos de Key Vault

El objetivo original de este laboratorio era evitar que nuestras cuentas de Azure Cosmos DB se codifiquen de forma rígida en nuestras aplicaciones. Sin embargo, hemos codificado de forma rígida nuestra dirección URL de **identificador secreto** que cualquiera puede ver. ¿Cómo podemos proteger nuestras credenciales? La buena noticia es que el identificador secreto por sí mismo es inútil. El **identificador secreto** solo le lleva a la puerta de Azure Key Vault, pero el almacén decidirá quién entra y quién permanece en la puerta. Esto significa que necesitaremos crear una directiva de acceso de Key Vault para nuestra aplicación para que pueda ver los secretos en ese almacén. Echemos un vistazo a esa solución.

1. (Opcional) Antes de crear la directiva, vamos a revisar el contenido actual de nuestra base de datos de Azure Cosmos DB.  En Azure Portal, vaya a la cuenta de Azure Cosmos DB, ¿existe una base de datos **GlobalCustomers**? Si no está allí, se creará mediante la ejecución correcta de la aplicación web. Si está allí, revise el número de elementos de la base de datos y la ejecución correcta de la aplicación web agregará más elementos.

1. En Azure Portal, vaya al almacén de claves que hemos creado anteriormente.

1. En la sección *Configuración*, seleccione **Configuración de acceso**.

1. Asegúrese de que **la directiva de acceso del almacén** está seleccionada y, a continuación, seleccione **Ir a las directivas de acceso**.

1. Seleccione **+ Create** (+ Crear).

1. En la pestaña **Permisos**, active la casilla **Obtener** **permisos de clave** y **permisos secretos** y, a continuación, seleccione **Siguiente**.

1. En la pestaña **Entidad de seguridad**, en el cuadro de búsqueda, escriba el nombre que asignó a App Service, selecciónelo en la lista y, a continuación, seleccione **Siguiente**.

1. En la pestaña **Aplicación (opcional)**, seleccione **Siguiente**.
    
1. En la pestaña **Revisar y crear**, seleccione **Crear**.

1. Probemos la aplicación web de nuevo.  En el explorador, vaya a **`https://<yourwebappname>.azurewebsites.net`**.

1. ¡Hecho! Nuestra página web debe indicar que insertamos nuevos elementos en el contenedor del cliente. También podemos ver que se muestra el secreto real.

    > &#128221; En un entorno de producción **nunca** se muestra el secreto, esto se acaba de hacer con fines ilustrativos.


1. Vaya a la cuenta de Azure Cosmos DB y compruebe que tiene una nueva base de datos **GlobalCustomers** con datos en ella o, si la base de datos ya existía, si ahora hay más elementos en la base de datos.

Ahora hemos usado correctamente Azure Key Vault para proteger las claves de la cuenta de Azure Cosmos DB.
