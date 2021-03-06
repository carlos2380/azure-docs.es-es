# Información general
## [¿Qué es Resource Manager?](resource-group-overview.md)
## [Versiones de API, regiones y servicios admitidos](resource-manager-supported-services.md)
## [Descripción de la implementación de Resource Manager y la implementación clásica](resource-manager-deployment-model.md)
## [Gobierno de suscripción prescriptivo](resource-manager-subscription-governance.md)
## [Ejemplos de gobierno para empresas](resource-manager-subscription-examples.md)

# Introducción
## [Exportación de la plantilla](resource-manager-export-template.md)
## [Creación de su primera plantilla](resource-manager-create-first-template.md)
## [Visual Studio con Resource Manager](vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)

# Muestras
## PowerShell
### [Implementación de la plantilla](resource-manager-samples-powershell-deploy.md)
## CLI de Azure
### [Implementación de la plantilla](resource-manager-samples-cli-deploy.md)

# Procedimientos
## Crear plantillas
### [Procedimientos recomendados para las plantillas](resource-manager-template-best-practices.md)
### [Secciones de plantilla](resource-group-authoring-templates.md)
### [Vínculo a otras plantillas](resource-group-linked-templates.md)
### [Definición de las dependencias entre recursos](resource-group-define-dependencies.md)
### Copia del bucle para crear varias instancias
#### [Sintaxis básica](resource-group-create-multiple.md)
#### [Bucle secuencial](resource-manager-sequential-loop.md)
#### [Copia de propiedades](resource-manager-property-copy.md)
### [Establecimiento de la ubicación](resource-manager-template-location.md)
### [Asignación de etiquetas](resource-manager-template-tags.md)
### [Establecimiento del nombre y tipo del recurso secundario](resource-manager-template-child-resource.md)
### [Actualización de recursos](resource-manager-update.md)
### [Uso de objetos para parámetros](resource-manager-objects-as-parameters.md)
### [Compartición del estado entre plantillas vinculadas](best-practices-resource-manager-state.md)
### [Patrones para diseñar plantillas](best-practices-resource-manager-design-templates.md)
## Implementación
### PowerShell
#### [Implementación de la plantilla](resource-group-template-deploy.md)
#### [Implementación de una plantilla privada con el token de SAS](resource-manager-powershell-sas-token.md)
#### [Exportación de la plantilla y reimplementación](resource-manager-export-template-powershell.md)
### CLI de Azure
#### [Implementación de la plantilla](resource-group-template-deploy-cli.md)
#### [Implementación de una plantilla privada con el token de SAS](resource-manager-cli-sas-token.md)
#### [Exportación de la plantilla y reimplementación](resource-manager-export-template-cli.md)
### [Portal](resource-group-template-deploy-portal.md)
### [API DE REST](resource-group-template-deploy-rest.md)
### [Integración continua con Visual Studio Team Services](../vs-azure-tools-resource-groups-ci-in-vsts.md?toc=%2fazure%2fazure-resource-manager%2ftoc.json)
### [Paso de valores seguros durante la implementación](resource-manager-keyvault-parameter.md)
## Administrar
### [PowerShell](powershell-azure-resource-manager.md)
### [CLI de Azure](xplat-cli-azure-resource-manager.md)
### [Portal](resource-group-portal.md)
### [API de REST](resource-manager-rest-api.md)
### [Uso de etiquetas para organizar los recursos](resource-group-using-tags.md)
### [Traslado de recursos a una suscripción o grupo nuevo](resource-group-move-resources.md)
## Control de acceso
### [Creación de una entidad de servicio con PowerShell](resource-group-authenticate-service-principal.md)
### [Creación de una entidad de servicio con la CLI de Azure 2.0](/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)
### [Creación de una entidad de servicio con la CLI de Azure 1.0](resource-group-authenticate-service-principal-cli.md)
### [Creación de una entidad de servicio con el portal](resource-group-create-service-principal-portal.md)
### [API de autenticación para acceder a las suscripciones](resource-manager-api-authentication.md)
### [Bloqueo de recursos](resource-group-lock-resources.md)
### [Consideraciones sobre la seguridad](best-practices-resource-manager-security.md)
## Establecimiento de directivas de recursos
### [¿Qué son las directivas de recursos?](resource-manager-policy.md)
### [Asignación de directivas de portal](resource-manager-policy-portal.md)
### [Asignación de directivas de script](resource-manager-policy-create-assign.md)
### [Directivas de etiquetas de recursos](resource-manager-policy-tags.md)
### [Directivas de almacenamiento](resource-manager-policy-storage.md)
### [Directivas de máquinas virtuales Linux](../virtual-machines/linux/policy.md?toc=%2fazure%2fazure-resource-manager%2ftoc.json)
### [Directivas de máquinas virtuales Windows](../virtual-machines/windows/policy.md?toc=%2fazure%2fazure-resource-manager%2ftoc.json)
## Auditoría y solución de problemas
### [Solución de errores de implementación comunes](resource-manager-common-deployment-errors.md)
### [Visualización de registros de actividad](resource-group-audit.md)
### [Ver operaciones de implementación](resource-manager-deployment-operations.md)

# Referencia
## [Funciones de plantillas](resource-group-template-functions.md)
### [Funciones de matriz y objeto](resource-group-template-functions-array.md)
### [Funciones de comparación](resource-group-template-functions-comparison.md)
### [Funciones de implementación](resource-group-template-functions-deployment.md)
### [Funciones numéricas](resource-group-template-functions-numeric.md)
### [Funciones de recursos](resource-group-template-functions-resource.md)
### [Funciones de cadena](resource-group-template-functions-string.md)
## [PowerShell](/powershell/module/azurerm.resources)
## [CLI de Azure 2.0](/cli/azure/resource)
## [.NET](/dotnet/api/microsoft.azure.management.resourcemanager)
## [Java](/java/api/com.microsoft.azure.management.resources)
## [Python](http://azure-sdk-for-python.readthedocs.io/en/latest/resourcemanagement.html)
## [Formato de plantilla](/azure/templates/)
## [REST](/rest/api/resources/)

# Recursos
## [Solicitudes de limitación](resource-manager-request-limits.md)
## [Seguimiento de operaciones asincrónicas](resource-manager-async-operations.md)
## [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-resource-manager)
## [Vídeos](https://azure.microsoft.com/documentation/videos/index/?services=azure-resource-manager)
## [Actualizaciones del servicio](https://azure.microsoft.com/updates/?product=azure-resource-manager)
