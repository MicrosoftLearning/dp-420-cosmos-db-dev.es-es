---
lab:
  title: Habilitación de proveedores de recursos
  module: Setup
---

# Habilitación de proveedores de recursos de Azure

Hay algunos proveedores de recursos que se deben estar registrados en la suscripción de Azure. Siga estos pasos para asegurarse de que están registrados.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. En la página **Inicio**, seleccione **Suscripciones**.

    > &#128161; Como alternativa, expanda el menú **&#8801;**, seleccione **Todos los servicios** y, en la categoría **Todos**, seleccione **Suscripciones**.

1. Seleccione la suscripción a Azure.

1. En la hoja de la suscripción, en la sección **Configuración**, seleccione **Proveedores de recursos**.

1. En la lista de proveedores de recursos, asegúrese de que están registrados los proveedores siguientes:
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; Si un proveedor no está registrado, selecciónelo y luego seleccione **Registrar**.

1. Cierre la ventana o pestaña del explorador web.

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
