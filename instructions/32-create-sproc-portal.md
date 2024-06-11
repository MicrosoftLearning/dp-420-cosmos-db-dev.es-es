---
lab:
  title: Creación de un procedimiento almacenado con Azure Portal
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Creación de un procedimiento almacenado con Azure Portal

Los procedimientos almacenados son una de las formas en que puede ejecutar el lado servidor de lógica de negocios en Azure Cosmos DB. Con un procedimiento almacenado, puede realizar operaciones CRUD (crear, leer, actualizar, eliminar) básicas con un contenedor en varios documentos dentro de un único ámbito transaccional.

En este laboratorio, originará un procedimiento almacenado que crea un documento dentro del contenedor. A continuación, usará una consulta SQL para validar los resultados del procedimiento almacenado.

## Creación de un procedimiento almacenado

Los procedimientos almacenados se crean en JavaScript integrado en lenguaje y admiten la ejecución de operaciones CRUD básicas dentro del motor de base de datos. JavaScript que se ejecuta en el motor de base de datos se hace posible mediante el SDK de JavaScript del lado servidor para Azure Cosmos DB y una serie de métodos auxiliares.

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

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, seleccione **Nuevo contenedor** y, a continuación, cree un contenedor con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Id. de base de datos** | *Crear nuevo* &vert; *``cosmicworks``* |
    | **Uso compartido del rendimiento entre contenedores** | *Seleccione esta opción* |
    | **Rendimiento de base de datos** | *Manual* &vert; *400* |
    | **Id. de contenedor** | *``products``* |
    | **Indexación** | *Automática* |
    | **Clave de partición** | *``/categoryId``* |

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, seleccione el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione **Nuevo procedimiento almacenado**.

1. En el campo **Id. de procedimiento almacenado**, escriba el valor **createDoc**.

1. Elimine el contenido del área del editor.

1. Cree una nueva función de JavaScript denominada **createDoc** sin parámetros de entrada:

    ```
    function createDoc() {
        
    }
    ```

1. En la función **createDoc**, invoque el método [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] integrado y almacene el resultado en una variable denominada **context**:

    ```
    var context = getContext();
    ```

1. Invoque el método [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] del objeto de contexto y almacene el resultado en una variable denominada **container**:

    ```
    var container = context.getCollection();
    ```

1. Cree un nuevo objeto denominado **doc** con dos propiedades:

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *primer documento* |
    | **Identificador de categoría** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. Invoque el método **createDocument** del objeto contenedor pasando el resultado de invocar el método **getSelfLink** del objeto contenedor y el nuevo documento como parámetros:

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. Una vez que haya terminado, el código del procedimiento almacenado debería incluir:

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. Seleccione **Guardar** para conservar los cambios en el procedimiento almacenado.

1. Seleccione **Ejecutar** y, a continuación, ejecute el procedimiento almacenado mediante los siguientes parámetros de entrada:

    | **Configuración** | **Clave** | **Valor** |
    | ---: | :--- | :--- |
    | **Valor de clave de partición** | *String* | *demo* |

1. Observe el resultado vacío. Aunque el procedimiento almacenado se ejecutó correctamente, el código JavaScript nunca devolvió una respuesta legible.

## Implementación de procedimientos recomendados para un procedimiento almacenado

Aunque el procedimiento almacenado creado anteriormente en este laboratorio tiene funcionalidad básica, también faltan algunas técnicas comunes de control de errores que se deben implementar en todos los procedimientos almacenados. En primer lugar, el procedimiento almacenado presupone que siempre tendrá tiempo para completar la operación y no comprueba el valor devuelto del método **createDocument** para asegurarse de que tiene suficiente tiempo. En segundo lugar, el procedimiento almacenado presupone que todos los documentos se insertan correctamente sin comprobar ni producir ningún mensaje de error potencial. Por último, el procedimiento almacenado no devuelve el documento recién creado como respuesta HTTP para la solicitud que invocó originalmente el procedimiento almacenado. Realizará estos tres cambios en el procedimiento almacenado para implementar procedimientos recomendados comunes.

1. Vuelva al editor del procedimiento almacenado **createDoc**.

1. Busque la línea 1 en el código que define la función **createDoc**:

    ```
    function createDoc() {
    ```

    y actualice la línea de código para incluir un parámetro denominado **title**:

    ```
    function createDoc(title) {
    ```

1. Busque la línea 5 en el código que establece la propiedad **name** del objeto **doc**:

    ```
    name: 'first document',
    ```

    y actualice la línea de código para usar el valor del parámetro **title**:

    ```
    name: title,
    ```

1. Busque la línea 8 en el código que invoca el método **createDocument**:

    ```
    container.createDocument(
    ```

    y actualice la línea de código para almacenar el resultado de la invocación del método en una variable denominada **accepted**

    ```
    var accepted = container.createDocument(
    ```

1. Agregue una nueva línea de código después de la invocación del método **createDocument** para comprobar el valor de la variable **accepted** y devolver el método si no es true:

    ```
    if (!accepted) return;
    ```

1. Por último, agregue un tercer parámetro a la invocación de método **createDocument** que es una función que adopta dos parámetros denominados **error** y **newDoc**, comprueba si el error es nulo y, a continuación, establece newDoc en el cuerpo de respuesta del procedimiento almacenado:

    ```
    ,
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. Una vez que haya terminado, el código del procedimiento almacenado debería incluir:

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. Seleccione **Actualizar** para conservar los cambios en el procedimiento almacenado.

1. Seleccione **Ejecutar** y, a continuación, ejecute el procedimiento almacenado mediante los siguientes parámetros de entrada:

    | **Configuración** | **Clave** | **Valor** |
    | ---: | :--- | :--- |
    | **Valor de clave de partición** | *String* | *demo* |
    | **Parámetros de entrada** | *String* | *segundo documento* |

1. Observe el resultado JSON. Una vez que el procedimiento almacenado se ejecutó correctamente, el documento recién creado se devolvió como respuesta para la solicitud HTTP original.

## Consulta de documentos

Para ir cerrando aspectos, usará el Explorador de datos para emitir una consulta SQL que devolverá los dos documentos creados en este laboratorio.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos en los que **categoryId** equivalga a **demo**:

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los dos documentos que creó en este laboratorio como los resultados de la ejecución de esta consulta.

1. Cierre la ventana o pestaña del explorador web.

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
