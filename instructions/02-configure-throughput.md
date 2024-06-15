---
lab:
  title: Configuración del rendimiento de Azure Cosmos DB para NoSQL con Azure Portal
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Configuración del rendimiento de Azure Cosmos DB for NoSQL con Azure Portal

Uno de los aspectos más importantes que hay que tener en cuenta es la configuración del rendimiento en Azure Cosmos DB for NoSQL. Para crear un contenedor de Azure Cosmos DB for NoSQL, primero debe crear una cuenta y, a continuación, una base de datos; en ese orden.

En este laboratorio, aprovisionará el rendimiento mediante varios métodos en Data Explorer. Aprovisionará el rendimiento manualmente o mediante la escalabilidad automática, en la base de datos y en el nivel de contenedor.

## Creación de una cuenta sin servidor

Empecemos de forma sencilla creando una cuenta sin servidor. No hay mucho que configurar aquí ya que todo es sin servidor. Cuando creamos nuestra base de datos y nuestro contenedor, no tenemos que aprovisionar rendimiento en absoluto. Verá todo esto a medida que vayamos creando esta cuenta.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. En la categoría **Servicios de Azure**, seleccione **Crear un recurso** y, a continuación, seleccione **Azure Cosmos DB**.

    > &#128161; Como alternativa; expanda el menú **&#8801;**, seleccione **Todos los servicios**, en la categoría **Bases de datos**, seleccione **Azure Cosmos DB** y después **Crear**.

1. En el panel de **opción Seleccionar API**, seleccione la opción **Crear** en la sección **Azure Cosmos DB for NoSQL**.

1. En el panel **Crear cuenta de Azure Cosmos DB**, observe la pestaña **Aspectos básicos**.

1. En la pestaña **Aspectos básicos**, escriba los valores siguientes para cada opción:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Suscripción** | **Use la suscripción de Azure existente.** *Todos los recursos deben pertenecer a un grupo de recursos. Cada grupo de recursos debe pertenecer a una suscripción.* |
    | **Grupo de recursos** | **Use un grupo de recursos existente o cree uno nuevo.** *Todos los recursos deben pertenecer a un grupo de recursos.*|
    | **Account Name** |  **Escriba un nombre único global.** *Nombre de cuenta único global. Este nombre se usará como parte de la dirección DNS para las solicitudes.  El portal comprobará el nombre en tiempo real.* |
    | **Ubicación** | **Seleccione cualquier región disponible.** *Seleccione la región geográfica en la que se hospedará inicialmente la base de datos.* |
    | **Capacity mode (Modo de capacidad)** | **Seleccione Sin servidor** |

1. Seleccione **Revisar y crear** para ir a la pestaña **Revisar y crear** y después seleccione **Crear**.

    > &#128221; La cuenta de Azure Cosmos DB for NoSQL puede tardar entre 10 y 15 minutos en estar lista para su uso.

1. Observe el panel **Implementación**. Una vez que se complete la implementación, el panel se actualizará con un mensaje **Implementación correcta**.

1. Aún dentro del panel **Implementación**, seleccione **Ir a recursos**.

1. En el panel **cuenta de Azure Cosmos DB**, seleccione **Data Explorer** en el menú de recursos.

1. En el panel **Data Explorer**, expanda **Nuevo contenedor** y, a continuación, seleccione **Nueva base de datos**.

1. En la ventana emergente **Nueva base de datos**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *`cosmicworks`* |

1. De nuevo en el panel **Data Explorer**, observe el nodo de base de datos **cosmicworks** dentro de la jerarquía.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *`products`* |
    | **Clave de partición** | *`/categoryId`* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

1. Vuelva al **Inicio** de Azure Portal.

## Creación de una cuenta aprovisionada

Ahora vamos a crear una cuenta de rendimiento aprovisionada con opciones de configuración más tradicionales. Este tipo de cuenta nos abrirá un mundo de opciones de configuración que puede resultar un poco abrumador. Vamos a ver algunos ejemplos de emparejamientos de bases de datos y contenedores que son posibles aquí.

1. En la categoría **Servicios de Azure**, seleccione **Crear un recurso** y, a continuación, seleccione **Azure Cosmos DB**.

    > &#128161; Como alternativa; expanda el menú **&#8801;**, seleccione **Todos los servicios**, en la categoría **Bases de datos**, seleccione **Azure Cosmos DB** y después **Crear**.

1. En el panel de **opción Seleccionar API**, seleccione la opción **Crear** en la sección **Azure Cosmos DB for NoSQL**.

1. En el panel **Crear cuenta de Azure Cosmos DB**, observe la pestaña **Aspectos básicos**.

1. En la pestaña **Aspectos básicos**, escriba los valores siguientes para cada opción:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Suscripción** | **Use la suscripción de Azure existente.** *Todos los recursos deben pertenecer a un grupo de recursos. Cada grupo de recursos debe pertenecer a una suscripción.* |
    | **Grupo de recursos** | **Use un grupo de recursos existente o cree uno nuevo.** *Todos los recursos deben pertenecer a un grupo de recursos.*|
    | **Account Name** |  **Escriba un nombre único global.** *Nombre de cuenta único global. Este nombre se usará como parte de la dirección DNS para las solicitudes.  El portal comprobará el nombre en tiempo real.* |
    | **Ubicación** | **Seleccione cualquier región disponible.** *Seleccione la región geográfica en la que se hospedará inicialmente la base de datos.* |
    | **Capacity mode (Modo de capacidad)** | **Rendimiento aprovisionado** |
    | **Aplicación de descuento por nivel Gratis** | **No aplicar** |
    | **Limitar la cantidad total de rendimiento que se puede aprovisionar en esta cuenta** | **Desactivado** |

1. Seleccione **Revisar y crear** para ir a la pestaña **Revisar y crear** y después seleccione **Crear**.

    > &#128221; La cuenta de Azure Cosmos DB for NoSQL puede tardar entre 10 y 15 minutos en estar lista para su uso.

1. Observe el panel **Implementación**. Una vez que se complete la implementación, el panel se actualizará con un mensaje **Implementación correcta**.

1. Aún dentro del panel **Implementación**, seleccione **Ir a recursos**.

1. En el panel **cuenta de Azure Cosmos DB**, seleccione **Data Explorer** en el menú de recursos.

1. En el panel **Data Explorer**, expanda **Nuevo contenedor** y, a continuación, seleccione **Nueva base de datos**.

1. En la ventana emergente **Nueva base de datos**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *`nothroughputdb`* |
    | **Aprovisionar rendimiento** | *Desactivado* |

1. De nuevo en el panel **Data Explorer**, observe el nodo de base de datos **nothroughputdb** dentro de la jerarquía.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *nothroughputdb* |
    | **Id. de contenedor** | *`requiredthroughputcontainer`* |
    | **Clave de partición** | *`/primarykey`* |
    | **Rendimiento del contenedor** | *Manual* |
    | **RU/s** | *`400`* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **nothroughputdb** y observe el nodo contenedor **requiredthroughputcontainer** dentro de la jerarquía.

1. En el panel **Data Explorer**, expanda **Nuevo contenedor** y, a continuación, seleccione **Nueva base de datos**.

1. En la ventana emergente **Nueva base de datos**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *`manualthroughputdb`* |
    | **Aprovisionar rendimiento** | *Activada* |
    | **Rendimiento de base de datos** | *Manual* |
    | **RU/s** | *`400`* |

1. De nuevo en el panel **Data Explorer**, observe el nodo de base de datos **manualthroughputdb** dentro de la jerarquía.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *manualthroughputdb* |
    | **Id. de contenedor** | *`childcontainer`* |
    | **Clave de partición** | *`/primarykey`* |
    | **Aprovisionar rendimiento dedicado para este contenedor** | *Activada* |
    | **Rendimiento del contenedor** | *Manual* |
    | **RU/s** | *`1000`* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **manualthroughputdb** y observe el nodo contenedor **childcontainer** dentro de la jerarquía.
