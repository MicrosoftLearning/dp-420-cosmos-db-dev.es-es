---
lab:
  title: Recuperación de una base de datos o un contenedor a partir de un punto de recuperación
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Recuperación de una base de datos o un contenedor a partir de un punto de recuperación 

Azure realiza automáticamente copias de seguridad cifradas de los datos. Estas copias de seguridad se realizan en dos modos, los modos de copia de seguridad **Periódico** y **Continuo**.

En este laboratorio, hará **copias de seguridad** y **restauraciones** mediante el modo de copia de seguridad continua. En primer lugar, creará una cuenta de Azure Cosmos DB. A continuación, creará dos contenedores y agregará algunos documentos a ellos. A continuación, actualizará un par de documentos en esos contenedores. Por último, creará restauraciones de la cuenta a un punto antes de cada eliminación.

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
    | Pestaña**Distribución global** | Deshabilitar las escrituras en varias regiones |

    > &#128221; Tenga en cuenta que puede habilitar el modo **Continuo** (7 días) durante la creación de la cuenta de Azure Cosmos DB; para ello, selecciónelo en la pestaña **Directiva de copia de seguridad**. En este laboratorio, tiene la opción de habilitar esta característica durante la creación de la cuenta o después de crearla en la sección opcional siguiente. **Sin embargo, habilitar la característica <ins>*después*</ins> de crear la cuenta *podría tardar más de 5 minutos*.**

    > &#128221; Tenga en cuenta que *[Actualmente no se admiten las cuentas de escritura en varias regiones para copias de seguridad continuas][/azure/cosmos-db/continuous-backup-restore-introduction]*.

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

## Agregar una base de datos y dos contenedores a la cuenta

Vamos a crear una base de datos y un par de contenedores.

1. En Azure Portal, vaya a la página de la cuenta de Azure Cosmos DB.

1. En **Explorador de datos**, agregue un nuevo contenedor con la siguiente configuración

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Id. de base de datos** | *Cree un nuevo* nombre: *`Sales`* |
    | **Uso compartido del rendimiento entre contenedores** | *No seleccionar* |
    | **Id. de contenedor** | *`customer`* |
    | **Clave de partición** | *`/id`* |
    | **Rendimiento del contenedor (400: RU/s ilimitados)** | Rendimiento *manual*: *400*|

1. En **Explorador de datos**, agregue un nuevo contenedor con la siguiente configuración

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Id. de base de datos** | *Use el nombre existente*: *Ventas* |
    | **Id. de contenedor** | *`salesOrder`* |
    | **Clave de partición** | *`/id`* |
    | **Rendimiento del contenedor (400: RU/s ilimitados)** | Rendimiento *manual*: *400*|

## Agregar elementos a los contenedores

Vamos a agregar algunos documentos a esos contenedores.

1. En Azure Portal, vaya a la página de la cuenta de Azure Cosmos DB.

1. En el **Explorador de datos**, agregue los dos documentos siguientes al contenedor del **cliente**.

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. En el **Explorador de datos**, agregue los tres documentos siguientes al contenedor **salesOrder**.

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## Cambie el modo de copia de seguridad predeterminado a continuo (opcional si la característica no está habilitada durante la creación de la cuenta)

*Si no habilitó la característica durante la creación de la cuenta de Azure Cosmos DB, deberá hacerlo ahora.*  Cambiar el modo de copia de seguridad es sencillo, todo lo que se necesita es cambiar un valor a **Activado**. Vamos a cambiarlo ahora.

1. En Azure Portal, vaya a la página de la cuenta de Azure Cosmos DB.

1. En la sección **Configuración**, seleccione **Copia de seguridad y restauración**.

1. Seleccione **Cambiar** junto al **Modo de directiva de copia de seguridad**; en la pantalla, seleccione la opción **Continua (7 días)** y, a continuación, seleccione **Guardar**. ***La habilitación de esta característica puede tardar más de cinco minutos***.

    > &#128221; Tenga en cuenta que *[Actualmente no se admiten las cuentas de escritura en varias regiones para copias de seguridad continuas][/azure/cosmos-db/continuous-backup-restore-introduction]*. Si no ha deshabilitado las escrituras en varias regiones al crear la cuenta de Azure Cosmos DB, tendrá que hacerlo ahora o se producirá un error al habilitar la característica de copia de seguridad continua.  Puede deshabilitar las escrituras en varias regiones en la sección **Replicación de datos globalmente** *Configuración*.

## Eliminar uno de los documentos salesOrder

1. En el **Explorador de datos**, ejecute la consulta siguiente para obtener la fecha y hora actuales. Copie esa marca de tiempo en el Bloc de notas. Esta marca de tiempo debe estar en UTC.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. En el **Explorador de datos**, busque el documento **salesOrder** con el **id.** `0019092E-BD25-48F5-8050-7051B2655BC5`. Elimine el documento y compruebe que el documento ya no esté ahí.

## Restaurar la base de datos al punto antes de eliminar el documento salesOrder

1. En Azure Portal, vaya a la página de la cuenta de Azure Cosmos DB.

1. En la sección *Configuración*, seleccione **Restauración a un momento dado**. Use la configuración siguiente:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Punto de restauración (UTC)** | Convierta la fecha y la hora adecuadamente. La hora tendrá que estar en formato a. m./p. m.|
    | **Ubicación** | *Seleccionar una ubicación disponible* |
    | **Seleccionar recursos que desea restaurar** | *Base de datos o contenedores seleccionados* |
    | **Restaurar recurso** | *salesOrder* |
    | **Restaurar cuenta de destino** | *elija un* ***nuevo*** *nombre de cuenta de Azure Cosmos DB* |

    > &#128221; En el caso de las restauraciones de Azure Cosmos DB, ***nunca*** se restaura encima de una cuenta *existente* y siempre tendrá que crear una nueva cuenta de Azure Cosmos DB.

    > &#128221; Aunque podría haber elegido restaurar toda la base de datos o incluso toda la cuenta, en un entorno de producción real, las bases de datos podrían ser enormes. En muchos escenarios, podría ser más rápido restaurar solo los contenedores o las bases de datos necesarias.

1. Esta restauración puede tardar 15 minutos o más; vaya a la sección siguiente y deje que esta restauración se ejecute en segundo plano.

## Eliminar el contenedor de cliente

1. En el **Explorador de datos**, ejecute la consulta siguiente para obtener la fecha y hora actuales. Copie esa marca de tiempo en el Bloc de notas.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Elimine el contenedor de **cliente**.

## Restaurar la base de datos al punto antes de eliminar el documento salesOrder

1. En Azure Portal, vaya a la página de la cuenta de Azure Cosmos DB.

1. En la sección *Configuración*, seleccione **Restauración a un momento dado**. Use la configuración siguiente:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Ubicación** | *Seleccionar una ubicación disponible* |
    | **Punto de restauración (UTC)** | Convierta la fecha y la hora adecuadamente. La hora tendrá que estar en formato a. m./p. m.|
    | **Seleccionar recursos que desea restaurar** | *Base de datos o contenedores seleccionados* |
    | **Restaurar recurso** | *`customer`* |
    | **Restaurar cuenta de destino** | *elija un* ***nuevo*** *nombre de cuenta de Azure Cosmos DB* |

    > &#128221; En el caso de las restauraciones de Azure Cosmos DB, ***nunca*** se restaura encima de una cuenta *existente* y siempre tendrá que crear una nueva cuenta de Azure Cosmos DB.

    > &#128221; Aunque podría haber elegido restaurar toda la base de datos o incluso toda la cuenta, en un entorno de producción real, las bases de datos podrían ser enormes. En muchos escenarios, podría ser más rápido restaurar solo los contenedores o las bases de datos necesarias.

1. Esta restauración puede tardar 15 minutos o más; vaya a la sección siguiente y deje que esta restauración se ejecute en segundo plano.

## Revisar los datos restaurados

Las restauraciones pueden tardar mucho tiempo en función del tamaño de la base de datos y de otros factores. Una vez finalizadas las restauraciones de la cuenta de Azure Cosmos DB:

1. Para nuestra primera restauración, asegúrese de que se ha recuperado el tercer documento.

1. Para la segunda restauración, deberíamos haber restaurado la tabla del cliente.

## Limpieza

1. Elimine las dos nuevas cuentas de Azure Cosmos DB creadas por las restauraciones de la cuenta.

1. Elimine la base de datos Sales y, si es necesario, elimine la cuenta original de Azure Cosmos DB.

[/azure/cosmos-db/continuous-backup-restore-introduction]:https://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction

