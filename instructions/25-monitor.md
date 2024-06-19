---
lab:
  title: "Uso de Azure Monitor para analizar una cuenta de Azure Cosmos\_DB for NoSQL"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Uso de Azure Monitor para analizar una cuenta de Azure Cosmos DB for NoSQL

Azure Monitor es un servicio de supervisión de pila completo en Azure que proporciona un conjunto completo de características para supervisar los recursos de Azure.  Azure Cosmos DB crea datos de supervisión mediante Azure Monitor.  Azure Monitor captura datos de telemetría y métricas de Cosmos DB.

En este laboratorio, ejecutará una carga de trabajo simulada en contenedores de Azure Cosmos DB y analizará cómo afecta esa carga de trabajo a la cuenta de Azure Cosmos DB.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccione cuál de las API quiere que admita la cuenta (por ejemplo, **API Mongo** o **API NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave. Use el punto de conexión y la clave para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante programación. Use el punto de conexión y la clave en las cadenas de conexión del SDK de Azure para .NET o cualquier otro SDK.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree un nuevo* |
    | **Account Name** | *Escriba un nombre único global*. |
    | **Ubicación** | *seleccione cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |
    | **Limitar la cantidad total de rendimiento que se puede aprovisionar en esta cuenta** | *Desactivar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **punto de conexión** más adelante en este ejercicio.

    1. Observe el campo **CLAVE PRINCIPAL**. Usará este valor de **clave** más adelante en este ejercicio.

1. Minimice, pero no cierre, la ventana del explorador. Volveremos a Azure Portal unos minutos después de iniciar una carga de trabajo en segundo plano en los pasos siguientes.


## Importación de las bibliotecas Microsoft.Azure.Cosmos y Newtonsoft.Json en un script de .NET

La CLI de .NET incluye un comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar paquetes desde una fuente de paquetes preconfigurada. Una instalación de .NET usa NuGet como fuente de paquetes predeterminada.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **25-monitor**.

1. Abra el menú contextual de la carpeta **25-monitor** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **25-monitor**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Agregue el paquete [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] desde NuGet mediante el siguiente comando:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

## Ejecución de un script para crear los contenedores y la carga de trabajo

Ahora estamos listos para ejecutar una carga de trabajo para supervisar su uso de la cuenta de Azure Cosmos DB.  El script se ejecutará en segundo plano. Este script creará tres contenedores y cargará algunos datos en esos contenedores. Después, el script ejecutará algunas consultas SQL aleatoriamente para emular varias aplicaciones de usuario que acceden a la cuenta de Azure Cosmos DB. 

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **25-monitor**.

1. Abra el archivo de código **Program.cs**.

1. Actualice la variable existente denominada **endpoint** con su valor establecido en el **punto de conexión** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, la instrucción C# sería: **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Actualice la variable existente denominada **key** con su valor establecido en la **clave** de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Guarde el archivo **Program.cs**.

1. Vuelva al *terminal integrado*.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```
    > &#128221; La primera parte de este script creará los tres contenedores y cargará los datos en ellos; esta acción tardará unos 2 minutos. Para emular algunos eventos de limitación de velocidad, el script establece el rendimiento aprovisionado en 400 RU/s. A continuación, debería obtener un mensaje parecido a ***Creación de una carga de trabajo en segundo plano simulada, espere entre 5 y 10 minutos y vaya al siguiente paso del ejercicio***. Dado que los recursos de Azure cargan datos de supervisión en Azure Monitor de forma asincrónica, es necesario esperar un breve período de tiempo para empezar a obtener algunos datos de diagnóstico en Métricas e Información de Azure Monitor. Una vez transcurridos de 5 a 10 minutos, vaya al paso siguiente. Si lo desea, para recopilar datos de diagnóstico adicionales, no tiene que detener el script después de 5 a 10 minutos y puede esperar hasta el final del laboratorio para hacerlo.

    > &#128221; Observará un par de advertencias en amarillo, ya que el compilador detecta que el script ejecuta muchas operaciones de forma sincrónica y no espera una respuesta de las operaciones. Puede omitir esta advertencia, ya que es el comportamiento esperado para ejecutar varios scripts SQL simultáneamente.

## Uso de Azure Monitor para analizar el uso de la cuenta de Azure Cosmos DB

En esta parte del ejercicio, volveremos al explorador y revisaremos algunos de los informes de Información y Métricas de Azure Monitor.

### Informes de métricas de Azure Monitor

1. Vuelva a la ventana del explorador abierta que hemos minimizado anteriormente. Si la ha cerrado, abra una nueva y vaya a la página de la cuenta de Azure Cosmos DB en portal.azure.com.

1. En el menú izquierdo de Azure Cosmos DB, en *Supervisión*, seleccione **Métricas**. Observará que los campos de **Ámbito** y **Espacio de nombres de métricas** se rellenan previamente con la información correcta. En los pasos siguientes, echaremos un vistazo a algunas opciones de **Métrica** y a las características de *Agregar filtro* y *Aplicar división*.

1. De forma predeterminada, la sección *Métricas* nos mostrará información de diagnóstico de las últimas 24 horas. Es necesario tener más precisión para examinar las métricas durante la carga de trabajo que hemos creado en el paso anterior. En la esquina superior derecha, seleccione el botón etiquetado ***Hora local: Últimas 24 horas (automático)***, obtendremos una ventana con varias opciones de intervalo de tiempo de tipo botón de radio.  Elija el botón de radio etiquetado ***Últimos 30 minutos*** y seleccione el botón **Aplicar**. Si es necesario, puede obtener más precisión eligiendo el botón de radio *Personalizado* y seleccionando una fecha y hora de inicio y finalización. 

1. Ahora que tenemos un buen intervalo de tiempo para nuestros gráficos de diagnóstico, echemos un vistazo a algunas métricas. Comenzaremos con una métrica común. En la lista desplegable *Métrica*, elija **Total de unidades de solicitud**. De forma predeterminada, esta métrica se mostrará como la suma total de unidades de solicitud (RU). O bien, puede cambiar la lista desplegable Agregación al valor de promedio o al valor máximo. Una vez que desactive esas dos agregaciones, establézcalo en *Suma* para los pasos siguientes.

1. Esta métrica nos da una buena idea de cuántas unidades de solicitudes se han usado en nuestra cuenta de Azure Cosmos DB. Sin embargo, nuestro gráfico actual podría no ayudarnos a centrarnos en un problema cuando tenemos varias bases de datos o contenedores en nuestra cuenta. Vamos a cambiarlo, revisaremos cómo se ha realizado nuestro consumo de RU por Base de datos. En el menú bajo el título char, seleccione **Aplicar división**, en la lista desplegable **Valores** elija **DatabaseName** y seleccione cualquier lugar del gráfico para aceptar los cambios. Ahora se colocará un botón **Dividir por = DatabaseName** justo encima del gráfico. 

1. Es más, ahora sabemos qué base de datos está haciendo la mayor parte del trabajo. Aunque esta información es buena, no sabemos qué contenedor está haciendo todo el trabajo.  Seleccione el botón **Dividir por = DatabaseName** para cambiar la condición Dividir y elija **CollectionName** en la lista desplegable *Valores*. Perfecto, ahora deberíamos tener datos para nuestras colecciones **customer** y **salesOrder**. Solo hay un problema con este gráfico, la colección **salesOrder** existe en dos bases de datos, **database-v2** y **database-v3**, por lo que este valor es una agregación de ese nombre de colección en ambas bases de datos.

1. Debe ser una corrección sencilla, seleccione el botón **Agregar filtro**, en la lista desplegable *Propiedad* elija **DatabaseName** y en *Valores*, elija **database-V3**.

1. Echemos un vistazo a dos métricas más. Editaremos el gráfico existente, o también puede crear un nuevo gráfico si lo desea. Encima del gráfico, seleccione el botón con el *nombre de la cuenta de Azure Cosmos DB* y la etiqueta **Unidad de solicitud total**. Elija **Total de solicitudes** en la lista desplegable *Métrica*, observe que la única agregación disponible es *Recuento*.

1. Dos filtros clave aquí pueden ayudarnos a solucionar diferentes tipos de problemas. Vamos a agregar un filtro con la propiedad **StatusCode** (observe que un filtro similar con un tipo de detalle diferente sería **Status**) y para *Valores*, elija **200** y **429**. Cambie Dividir para usar StatusCode. Observe que hay una gran cantidad de 429, o solicitudes de limitación de velocidad en comparación con el estado 200, solicitudes correctas. Las 429 excepciones se produjeron porque el script envía miles de solicitudes por segundo mientras establecemos el rendimiento aprovisionado en 400 RU/s. *Este gran número de 429 excepciones, en comparación con la solicitud correcta, no debería ser normal en un entorno de producción. En un entorno de producción, las 429 excepciones deben producirse con poca frecuencia en una cuenta de Azure Cosmos DB correcta*.  También puede usar las *Propiedades* **StatusCode** o **Status** de forma similar a **Total de unidades de solicitud**

1. Vamos a examinar la **Solicitud total**, pero cambiaremos la división a **OperationType**.  Esta propiedad nos ayudará a determinar qué operaciones de lectura o escritura están realizando la mayor parte del trabajo. De nuevo, esta propiedad también se podría usar de forma similar en **Total de unidades de solicitud**

1. Al igual que hicimos con el **Total de unidades de solicitud**, experimente eligiendo diferentes filtros y opciones de división. 

1. La métrica final que veremos en este ejercicio es la métrica de **Consumo de RU normalizado**. Cambie la división a **PartitionKeyRangeId**. Esta métrica nos ayuda a identificar qué uso del intervalo de claves de partición es más adecuado. La métrica permite sesgar el rendimiento hacia un intervalo de claves de partición. Continúe y elija esa métrica en la lista desplegable *Métrica*. Este gráfico debería mostrarnos ahora un sistema muy incorrecto, alcanzando un 100 % constante de **Consumo normalizado de RU**.

> &#128221; Si desea ver más de un gráfico a la vez, haga clic en la opción **+ Nuevo gráfico** encima del nombre del gráfico. 

> &#128221; Aunque no podemos guardar directamente nuestras métricas, puede crear o usar un panel existente y agregar este gráfico haciendo clic en el botón **Anclar al panel** de la esquina superior derecha del gráfico.  Haga clic en el botón y elija la pestaña **Crear nuevo**, asígnele el nombre *Laboratorios DP-420* y haga clic en **Crear y anclar**. Para ver los paneles privados, debe ir al menú Portal en la esquina superior izquierda y elegir Panel en las opciones de Recursos de Azure. La primera vez, el panel puede tardar unos minutos en aparecer.

> &#128221; Una forma más de compartir el gráfico es hacer clic en la lista desplegable Compartir y descargarlo como un archivo de Excel o la opción Copiar vínculo.

### Informes de Información de Azure Monitor

Es posible que tengamos que dedicar algún tiempo a ajustar los informes de diagnóstico de Métricas de Azure Monitor.  Información de Cosmos DB ofrece una vista general del rendimiento, los errores y el mantenimiento operativo de todos los recursos de Azure Cosmos DB. Estos gráficos de Información serán gráficos pregenerados similares a los de Métricas. Echemos un vistazo a algunos de ellos.

1. En el menú izquierdo de Azure Cosmos DB, en *Supervisión*, seleccione **Información**. Observará que hay varias pestañas de Información general a Opciones de administración. Veremos algunos de estos gráficos de **Información**. La primera pestaña, la pestaña Información general, proporciona un resumen de los gráficos más comunes que puede usar. Por ejemplo, gráficos como los de Solicitud total, Uso de datos e índices, 429 excepciones y Consumo normalizado de RU.  Hemos visto la mayoría de estos gráficos en la sección anterior.

1. Observe que en la parte superior de los gráficos podemos alojar el **Intervalo de tiempo**, por lo que seleccione *15* o *30* minutos para evaluar la carga de trabajo en este ejercicio.

1. En la esquina superior derecha de *cada* gráfico, observará una opción para ***Abrir el explorador de métricas***. Continúe y seleccione la opción **Abrir el explorador de métricas** para el gráfico **Total de solicitudes**. Observará que al seleccionar esta opción irá a los informes de métricas que revisamos anteriormente. La ventaja de abrir el explorador de métricas es que ya se ha creado una buena parte del gráfico.

1. Para volver a la página Información, seleccione la **X** en la esquina superior derecha del gráfico de Métricas.

1. Seleccione la pestaña Rendimiento. Estos gráficos son buenos para identificar problemas de rendimiento.  Preste mucha atención al gráfico **Consumo de RU normalizado (%) por PartitionKeyRangeID**, que se puede usar para detectar particiones activas.

1. Seleccione la pestaña Solicitudes. Estos gráficos son excelentes para analizar el número de eventos de limitación que experimenta la cuenta (429 frente a 200) o para revisar el número de solicitudes por tipo de operación.  

1. Seleccione la pestaña Almacenamiento. Estos gráficos nos muestran tanto el crecimiento de nuestras colecciones como el uso de datos e índices.  

1. Seleccione la pestaña Sistema. Si la aplicación estaba creando, eliminando o consultando los metadatos de las cuentas con frecuencia, es posible tener 429 excepciones.  Estos gráficos nos ayudan a determinar si ese acceso frecuente a metadatos es la causa de nuestras 429 excepciones. Además, podemos determinar el estado de nuestras solicitudes de metadatos.  

### Informes de Información de Azure Monitor

1. Si el programa sigue en ejecución, vuelva al terminal de comandos de Visual Studio Code.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
