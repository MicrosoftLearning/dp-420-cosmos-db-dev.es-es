---
lab:
  title: Crear grupo de recursos de laboratorio
  module: Setup
---

# Creación de un grupo de recursos de Azure para laboratorio

Antes de completar este laboratorio, debe crear un nuevo [grupo de recursos][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal] en el que colocar el recurso de Azure recién implementado.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. En la página de **Inicio**, seleccione **Grupos de recursos**.

    > &#128161; Como alternativa, expanda el menú **&#8801;**, seleccione **Todos los servicios** y, en la categoría **Todos**, seleccione **Grupos de recursos**.

1. Seleccione **+ Create** (+ Crear).

1. En el emergente **Crear un grupo de recursos**, cree un grupo de recursos con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Asigne un nombre único al grupo de recursos* |
    | **Región** | *seleccione cualquier región disponible* |

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal
