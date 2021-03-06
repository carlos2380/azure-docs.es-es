---
title: "Instalación de paquetes de aplicaciones en nodos de proceso - Azure Batch | Microsoft Docs"
description: "Utilice la característica paquetes de aplicación de Lote de Azure para administrar fácilmente varias aplicaciones y versiones para la instalación en nodos de proceso de Lote."
services: batch
documentationcenter: .net
author: tamram
manager: timlt
editor: 
ms.assetid: 3b6044b7-5f65-4a27-9d43-71e1863d16cf
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 02/27/2017
ms.author: tamram
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 6b6c548ca1001587e2b40bbe9ee2fcb298f40d72
ms.openlocfilehash: 9c7073e55b98406fc8f9db9a40bf1a6ffc626f47
ms.lasthandoff: 02/28/2017


---
# <a name="deploy-applications-to-compute-nodes-with-batch-application-packages"></a>Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch

La característica de paquetes de aplicación de Lote de Azure permite administrar e implementar fácilmente aplicaciones de tareas en los nodos de proceso de un grupo. Con paquetes de aplicación, puede cargar y administrar fácilmente varias versiones de las aplicaciones que las tareas ejecutan, incluidos los archivos auxiliares. A continuación, se pueden implementar automáticamente una o varias de estas aplicaciones en los nodos de proceso del grupo.

En este artículo, aprenderá cómo cargar y administrar paquetes de aplicación en el Portal de Azure. A continuación, aprenderá a instalarlos en nodos de proceso de un grupo mediante la biblioteca de [.NET para Batch][api_net].

> [!NOTE]
> La característica paquetes de aplicación que se describe aquí reemplaza a la característica "aplicaciones de Lote" disponible en las versiones anteriores del servicio.
> 
> 

## <a name="application-package-requirements"></a>Requisitos de los paquetes de aplicación
Para utilizar paquetes de aplicación, primero se debe [vincular una cuenta de Almacenamiento de Azure](#link-a-storage-account) a su cuenta de Lote.

La característica de paquetes de aplicación que se explica en este artículo *solo* es compatible con los grupos de Lote creados después del 10 de marzo de 2016. Los paquetes de aplicación no se implementarán en los nodos de proceso de los grupos creados antes de esta fecha.

Esta característica se introdujo en la [API de REST de Batch][api_rest] versión 2015-12-01.2.2, y la correspondiente biblioteca de [.NET para Batch][api_net], versión 3.1.0. Si se trabaja con Lote, se recomienda utilizar siempre la versión más reciente de la API.

> [!IMPORTANT]
> Actualmente, solo los grupos *CloudServiceConfiguration* admiten paquetes de aplicación. No puede usar paquetes de aplicación en grupos creados con imágenes VirtualMachineConfiguration. Consulte la sección [Configuración de la máquina virtual](batch-linux-nodes.md#virtual-machine-configuration) de [Aprovisionamiento de nodos de proceso de Linux en grupos del servicio Lote de Azure](batch-linux-nodes.md) para más información sobre las dos configuraciones.
> 
> 

## <a name="about-applications-and-application-packages"></a>Acerca de las aplicaciones y los paquetes de aplicación
En Lote de Azure, una *aplicación* hace referencia a un conjunto de archivos binarios con versiones que se pueden descargar automáticamente en los nodos de proceso del grupo. Un *paquete de aplicación* hace referencia a un *conjunto específico* de dichos archivos binarios y representa una *versión* determinada de la aplicación.

![Diagrama de alto nivel de aplicaciones y paquetes de aplicación][1]

### <a name="applications"></a>Aplicaciones
Una aplicación en Lote contiene uno o más paquetes de aplicación y especifica las opciones de configuración de la aplicación. Por ejemplo, una aplicación puede especificar la versión predeterminada del paquete de aplicación que se instala en los nodos de proceso y si sus paquetes se pueden actualizar o eliminar.

### <a name="application-packages"></a>Paquetes de aplicación
Un paquete de aplicación es un archivo .zip que contiene los archivos binarios de la aplicación y los archivos auxiliares que se requieren para que las tareas los ejecuten. Cada paquete de aplicación representa una versión específica de la aplicación.

Los paquetes de aplicaciones se pueden especificar a nivel de grupo y de tarea. Puede especificar uno o varios de estos paquetes y (opcionalmente) una versión cuando crea un grupo o tarea.

* **paquetes de aplicación del grupo** se implementan en *cada* nodo del grupo. Las aplicaciones se implementan cuando un nodo se une a un grupo y cuando se reinicia o se restablece la imagen inicial.
  
    Los paquetes de aplicación de grupo son adecuados cuando todos los nodos de un grupo ejecutan las tareas de un trabajo. Puede especificar uno o más paquetes de aplicación cuando se crea un grupo y puede agregar o actualizar los paquetes de un grupo ya existente. Si actualiza los paquetes de aplicaciones de un grupo existente, debe reiniciar los nodos para instalar el nuevo paquete.
* **paquetes de aplicación de tareas** solo se implementan en un nodo de proceso programado para ejecutar una tarea, justo antes de ejecutar la línea de comandos de la tarea. Si el paquete de aplicación y la versión especificados ya están en el nodo, este no se volverá a implementar y se utilizará el paquete existente.
  
    Los paquetes de aplicaciones de tareas son útiles en entornos de grupo compartido, donde los distintos trabajos se ejecutan en un grupo que no se elimina cuando se completa un trabajo. Si el trabajo tiene menos tareas que nodos en el grupo, los paquetes de aplicación de las tareas pueden minimizar la transferencia de datos, ya que la aplicación se implementa solo en los nodos que ejecutan tareas.
  
    Otros escenarios que pueden beneficiarse de los paquetes de aplicación de tareas son los trabajos que usan una aplicación de gran tamaño, pero solo para un pequeño número de tareas. Por ejemplo, una fase previa al procesamiento o una tarea de combinación, donde la aplicación de procesamiento previo o combinación es pesada.

> [!IMPORTANT]
> Hay restricciones en el número de aplicaciones y paquetes de aplicación que puede haber en una cuenta de Lote, así como en el tamaño máximo del paquete de aplicación. Para más información sobre estos límites, consulte [Cuotas y límites del servicio de Lote de Azure](batch-quota-limit.md) .
> 
> 

### <a name="benefits-of-application-packages"></a>Ventajas de los paquetes de aplicación
Los paquetes de aplicación pueden simplificar el código de su solución de Lote y reducir la sobrecarga requerida para administrar las aplicaciones que ejecutan las tareas.

La tarea de inicio del grupo no tiene que especificar una larga lista de archivos de recursos individuales que se deben instalar en los nodos. No es preciso administrar manualmente varias versiones de los archivos de la aplicación en Almacenamiento de Azure ni en los nodos. Y tampoco es preciso preocuparse de generar [direcciones URL de SAS](../storage/storage-dotnet-shared-access-signature-part-1.md) para proporcionar acceso a los archivos de su cuenta de Almacenamiento. Lote funciona en segundo plano con el Almacenamiento de Azure para almacenar paquetes de aplicación e implementarlos en los nodos de proceso.

## <a name="upload-and-manage-applications"></a>Carga y administración de aplicaciones
Puede usar el [Azure Portal][portal] o la biblioteca de [Batch Management .NET](batch-management-dotnet.md) para administrar los paquetes de aplicaciones en la cuenta de Batch. En las siguientes secciones, se vinculará primero una cuenta de almacenamiento y, después, se analizará la incorporación de paquetes y aplicaciones y su administración con el portal.

### <a name="link-a-storage-account"></a>Vínculo a una cuenta de Almacenamiento
Para utilizar paquetes de aplicación, primero se debe vincular una cuenta de Almacenamiento de Azure a su cuenta de Lote. Si aún no ha configurado ninguna cuenta de almacenamiento para su cuenta de Batch, Azure Portal mostrará una advertencia la primera vez que haga clic en el icono **Aplicaciones** en la hoja **Cuenta de Batch**.

> [!IMPORTANT]
> Actualmente, Batch *solo* admite el tipo de cuenta de almacenamiento **de uso general**, tal y como se describe en el paso 5, [Crear una cuenta de almacenamiento](../storage/storage-create-storage-account.md#create-a-storage-account), del artículo [Acerca de las cuentas de Azure Storage](../storage/storage-create-storage-account.md). Cuando vincula una cuenta de Almacenamiento de Azure a su cuenta de Lote, *solo* se vincula una cuenta de almacenamiento de **Uso general** .
> 
> 

![Advertencia de que no hay cuentas de almacenamiento almacenadas en Azure Portal][9]

El servicio de Lote utiliza la cuenta de Almacenamiento asociada para el almacenamiento y la recuperación de paquetes de aplicación. Una vez que haya vinculado las dos cuentas, Lote puede implementar automáticamente los paquetes almacenados en la cuenta de Almacenamiento vinculada en los nodos de proceso. Haga clic en **Configuración de cuenta de almacenamiento** en la hoja **Advertencia**; después, en **Cuenta de almacenamiento** en la hoja **Cuenta de almacenamiento** para vincular una cuenta de almacenamiento a su cuenta de Batch.

![Hoja Elegir de cuenta de almacenamiento en Azure Portal][10]

Se recomienda crear una cuenta de almacenamiento *específicamente* para su uso con la cuenta de Lote y seleccionarla aquí. Para información más detallada sobre cómo crear una cuenta de almacenamiento, consulte la sección "Crear una cuenta de almacenamiento" en [Acerca de las cuentas de almacenamiento de Azure](../storage/storage-create-storage-account.md). Una vez que haya creado una cuenta de Almacenamiento, puede vincularla a su cuenta de Lote mediante la hoja **Cuenta de almacenamiento** .

> [!WARNING]
> Dado que Batch almacena los paquetes de aplicación con Azure Storage, se aplican [cargos normales][storage_pricing] a los datos de blob en bloques. Asegúrese de considerar el tamaño y número de los paquetes de aplicación y elimine periódicamente los paquetes en desuso para minimizar el costo.
> 
> 

### <a name="view-current-applications"></a>Visualización de las aplicaciones actuales
Para ver las aplicaciones en la cuenta de Batch, haga clic en el elemento de menú **Aplicaciones** situado en el menú de la izquierda de la hoja **Cuenta de Batch**.

![Icono Aplicaciones][2]

Se abre la hoja **Aplicaciones** :

![Lista de aplicaciones][3]

La hoja **Aplicaciones** muestra el identificador de cada aplicación de su cuenta y las siguientes propiedades:

* **Paquetes**: el número de versiones asociadas a la aplicación.
* **Versión predeterminada**: la versión que se instala si no especifica ninguna versión al configurar la aplicación para un grupo. Esta configuración es opcional.
* **Permitir actualizaciones**: el valor que especifica si se permiten las actualizaciones, eliminaciones y adiciones en el paquete. Si se establece en **No**, las actualizaciones y eliminaciones se deshabilitan para la aplicación. Solo se pueden agregar versiones nuevas del paquete de aplicación. El valor predeterminado es **Sí**.

### <a name="view-application-details"></a>Visualización de los detalles de una aplicación
Haga clic en una aplicación en la hoja **Aplicaciones** para abrir la hoja que incluye los detalles de la aplicación.

![Detalles de la aplicación][4]

En la hoja de detalles de la aplicación, puede configurar los siguientes valores de la aplicación.

* **Permitir actualizaciones**: especifica si se pueden actualizar o eliminar sus paquetes de aplicación. Consulte "Actualización o eliminación de un paquete de aplicación" más adelante en este artículo.
* **Versión predeterminada**: especifique el paquete de aplicación predeterminado que se implementa en los nodos de proceso.
* **Nombre para mostrar**: especifica un nombre "descriptivo" que la solución de Lote puede usar al mostrar información de la aplicación, como en la interfaz de usuario de un servicio que proporciona a los clientes a través de Lote.

### <a name="add-a-new-application"></a>Adición de una nueva aplicación
Para crear una aplicación, agregue un paquete de aplicación y especifique un identificador de aplicación nuevo y exclusivo. El primer paquete de aplicación que agregue con el nuevo identificador de aplicación también creará la nueva aplicación.

Haga clic en **Agregar** en la hoja **Aplicaciones** para abrir la hoja **Nueva aplicación**.

![Hoja Nueva aplicación en Azure Portal][5]

La hoja **Nueva aplicación** proporciona los siguientes campos para especificar la configuración de la nueva aplicación y del paquete de aplicación.

**Id. de la aplicación**

Este campo especifica el identificador de la nueva aplicación, que está sujeto a las reglas de validación estándar de identificadores de Lote de Azure:

* Pueden contener cualquier combinación de caracteres alfanuméricos, incluidos los guiones y caracteres de subrayado.
* No pueden contener más de 64 caracteres.
* Deben ser único en la cuenta de Lote.
* Conserva las mayúsculas y minúsculas y no distingue mayúsculas de minúsculas.

**Versión**

Especifica la versión del paquete de aplicación que se carga. Las cadenas de la versión están sujetas a las siguientes reglas de validación:

* Pueden contener cualquier combinación de caracteres alfanuméricos, incluidos guiones, caracteres de subrayado y puntos.
* No pueden contener más de 64 caracteres.
* Deben ser únicas en la aplicación.
* Conservación de mayúsculas y minúsculas y no distingue mayúsculas de minúsculas.

**Paquete de aplicación**

Este campo especifica el archivo .zip que contiene los archivos binarios y los archivos auxiliares necesarios para ejecutar la aplicación. Haga clic en el cuadro **Seleccionar un archivo** o en el icono de la carpeta a la que desee desplazarse y seleccione un archivo .zip que contenga los archivos de la aplicación.

Una vez que haya seleccionado un archivo, haga clic en **Aceptar** para iniciar la carga en Almacenamiento de Azure. Cuando se complete la operación de carga, recibirá una notificación y se cerrará la hoja. En función del tamaño del archivo que se va a cargar y de la velocidad de la conexión de red, esta operación puede tardar un tiempo.

> [!WARNING]
> No cierre la hoja **Nueva aplicación** antes de que se complete la operación de carga. Si lo hace, se detendrá el proceso de carga.
> 
> 

### <a name="add-a-new-application-package"></a>Adición de un nuevo paquete de aplicación
Para agregar una nueva versión del paquete de aplicación para una aplicación existente, seleccione una aplicación en la hoja **Aplicaciones**, haga clic en **Paquetes** y en **Agregar** para abrir la hoja **Agregar paquete**.

![Hoja Agregar paquete de aplicación en Azure Portal][8]

Como puede ver, los campos coinciden con los de la hoja **Nueva aplicación**, excepto el cuadro de texto **Id. de aplicación**, que está deshabilitado. Como hizo para la nueva aplicación, especifique la **Versión** del paquete nuevo, vaya al archivo .zip de su **Paquete de aplicación** y haga clic en **Aceptar** para cargar el paquete.

### <a name="update-or-delete-an-application-package"></a>Actualización o eliminación de un paquete de aplicación
Para actualizar o eliminar un paquete de aplicación existente, abra la hoja de detalles de la aplicación, haga clic en **Paquetes** para abrir la hoja **Paquetes** y en los **puntos suspensivos** de la fila del paquete de aplicación que desee modificar, y seleccione la acción que desee realizar.

![Actualizar o eliminar paquete en Azure Portal][7]

**Actualización**

Al hacer clic en **Actualizar**, se muestra la hoja *Actualizar paquete*. Esta hoja es similar a la hoja *New application package* (Nuevo paquete de aplicación). Sin embargo, solo está habilitado el campo de selección de paquete, lo que permite especificar un nuevo archivo ZIP para cargarlo.

![Hoja Actualizar paquete en Azure Portal][11]

**Eliminar**

Al hacer clic en **Eliminar**, se le pedirá que confirme la eliminación de la versión del paquete y Lote eliminará el paquete de Almacenamiento de Azure. Si elimina la versión predeterminada de una aplicación, la configuración de la **Versión predeterminada** se quita de la aplicación.

![Eliminar aplicación][12]

## <a name="install-applications-on-compute-nodes"></a>Instalación de aplicaciones en nodos de proceso
Ahora que ha visto cómo administrar paquetes de aplicación con el Portal de Azure, podemos analizar cómo implementarlos en los nodos de proceso y ejecutarlos con las tareas de Lote.

### <a name="install-pool-application-packages"></a>Instalación de paquetes de aplicación de grupos
Para instalar un paquete de aplicación en todos los nodos de proceso de un grupo, especifique una o varias *referencias* de paquetes de aplicación para el grupo. Los paquetes de aplicación que especifique para un grupo se instalan en cada nodo de proceso cuando este se una al grupo, además de cuando se reinicie o se restablezca la imagen inicial.

En el entorno de .NET para Batch, especifique una o varias [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref] al crear el grupo o para un grupo existente. La clase [ApplicationPackageReference][net_pkgref] especifica una versión y un identificador de la aplicación que se va a instalar en los nodos de proceso de un grupo.

```csharp
// Create the unbound CloudPool
CloudPool myCloudPool =
    batchClient.PoolOperations.CreatePool(
        poolId: "myPool",
        targetDedicated: 1,
        virtualMachineSize: "small",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "4"));

// Specify the application and version to install on the compute nodes
myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "litware",
        Version = "1.1001.2b" }
};

// Commit the pool so that it's created in the Batch service. As the nodes join
// the pool, the specified application package will be installed on each.
await myCloudPool.CommitAsync();
```

> [!IMPORTANT]
> Si, por cualquier motivo, se produce un error en una implementación del paquete de aplicación, el servicio Batch marca el nodo como [inutilizable][net_nodestate] y no se programarán tareas de ejecución en ese nodo. En este caso, debería **reiniciar** el nodo para reiniciar la implementación del paquete. Al reiniciar el nodo, también se volverá a habilitar la programación de tareas en el nodo.
> 
> 

### <a name="install-task-application-packages"></a>Instalación de paquetes de aplicación de tareas
De forma similar a un grupo, puede especificar las *referencias* de un paquete de aplicación para una tarea. Cuando una tarea está programada para ejecutarse en un nodo, el paquete se descarga y extrae justo antes de ejecutar la línea de comandos de la tarea. Si el paquete y versión especificados ya están instalados en el nodo, el paquete no se descarga y se utiliza el paquete existente.

Para instalar un paquete de aplicación de tareas, configure la propiedad [CloudTask][net_cloudtask].[ApplicationPackageReferences][net_cloudtask_pkgref]:

```csharp
CloudTask task =
    new CloudTask(
        "litwaretask001",
        "cmd /c %AZ_BATCH_APP_PACKAGE_LITWARE%\\litware.exe -args -here");

task.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference
    {
        ApplicationId = "litware",
        Version = "1.1001.2b"
    }
};
```

## <a name="execute-the-installed-applications"></a>Ejecución de las aplicaciones instaladas
Los paquetes que ha especificado para un grupo o tarea se descargan y extraen en un directorio con nombre dentro del nodo `AZ_BATCH_ROOT_DIR` . Lote también permite crear una variable de entorno que contiene la ruta de acceso al directorio con nombre. Las líneas de comando de la tarea usan esta variable de entorno al hacer referencia a la aplicación en el nodo. La variable está en el formato siguiente:

`AZ_BATCH_APP_PACKAGE_APPLICATIONID#version`

`APPLICATIONID` y `version` son los valores que corresponden a la versión de la aplicación y del paquete que ha especificado para la implementación. Por ejemplo, si especificó que se debía instalar la versión 2.7 de la aplicación *blender* , las líneas de comando de la tarea usarán esta variable de entorno para tener acceso a sus archivos:

`AZ_BATCH_APP_PACKAGE_BLENDER#2.7`

Cuando carga un paquete de aplicación, puede especificar una versión predeterminada que implementar en los nodos de proceso. Si ha especificado una versión predeterminada para una aplicación, puede omitir el sufijo de versión al hacer referencia a ella. Puede especificar la versión predeterminada de la aplicación en Azure Portal, en la hoja Aplicaciones, como se muestra en [Carga y administración de aplicaciones](#upload-and-manage-applications).

Por ejemplo, si especifica la versión 2.7 como la versión predeterminada de la aplicación *blender*, las tareas pueden hacer referencia a la siguiente variable de entorno y ejecutarán la versión 2.7:

`AZ_BATCH_APP_PACKAGE_BLENDER`

El siguiente fragmento de código muestra una línea de comandos de una tarea de ejemplo que inicia la versión predeterminada de la aplicación *blender* :

```csharp
string taskId = "blendertask01";
string commandLine =
    @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -args -here";
CloudTask blenderTask = new CloudTask(taskId, commandLine);
```

> [!TIP]
> Para más información sobre la configuración del entorno del nodo de proceso, consulte el apartado [Configuración del entorno para las tareas](batch-api-basics.md#environment-settings-for-tasks) de [Información general de las características de Batch](batch-api-basics.md).
> 
> 

## <a name="update-a-pools-application-packages"></a>Actualización de los paquetes de aplicación de un grupo
Si un grupo existente ya se ha configurado con un paquete de aplicación, se puede especificar un paquete nuevo para el grupo. Si especifica una nueva referencia de paquete para un grupo, se aplica lo siguiente:

* Todos los nodos nuevos que se unen al grupo y cualquier nodo existente que se reinicie o cuya imagen inicial se restablezca instalarán el paquete recién especificado.
* Los nodos de proceso que ya estén en el grupo cuando se actualicen las referencias del paquete no instalan automáticamente el paquete de aplicación nuevo. Estos nodos de proceso deben reiniciarse o se debe restablecer su imagen inicial para recibir el nuevo paquete.
* Cuando se implementa un nuevo paquete, las variables de entorno creadas reflejan las nuevas referencias del paquete de aplicación.

En este ejemplo, el grupo existente tiene la versión 2.7 de la aplicación *blender* configurada como una de sus propiedades [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]. Para actualizar los nodos del grupo con la versión 2.76b, especifique una nueva clase [ApplicationPackageReference][net_pkgref] con la nueva versión y confirme el cambio.

```csharp
string newVersion = "2.76b";
CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "blender",
        Version = newVersion }
};
await boundPool.CommitAsync();
```

Ahora que se ha configurado la nueva versión, todos los nodos *nuevos* que se unan al grupo tendrán la versión 2.76b implementada. Para instalar la versión 2.76b en los nodos que *ya* están en el grupo, reinícielos o restablezca su imagen inicial. Tenga en cuenta que los nodos reiniciados conservarán los archivos de las anteriores implementaciones del paquete.

## <a name="list-the-applications-in-a-batch-account"></a>Enumeración de las aplicaciones en una cuenta de Lote
Puede enumerar las aplicaciones y sus paquetes en una cuenta de Batch mediante el método [ApplicationOperations][net_appops].[ListApplicationSummaries][net_appops_listappsummaries].

```csharp
// List the applications and their application packages in the Batch account.
List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
foreach (ApplicationSummary app in applications)
{
    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);

    foreach (string version in app.Versions)
    {
        Console.WriteLine("  {0}", version);
    }
}
```

## <a name="wrap-up"></a>Encapsulado
Con los paquetes de aplicación puede ayudar a los clientes a seleccionar las aplicaciones para sus trabajos y especificar la versión exacta que deben usar al procesar los trabajos con su servicio con Lote habilitado. También puede proporcionar a los clientes la capacidad de cargar y hacer un seguimiento de sus propias aplicaciones en su servicio.

## <a name="next-steps"></a>Pasos siguientes
* La [API de REST de Batch][api_rest] también proporciona compatibilidad para trabajar con paquetes de aplicación. Por ejemplo, consulte el elemento [applicationPackageReferences][rest_add_pool_with_packages] de [Agregar un grupo a una cuenta][rest_add_pool] para especificar los paquetes que se instalan mediante la API de REST. Para ver detalles sobre cómo obtener información de la aplicación mediante la API de REST de Batch, consulte [Aplicaciones][rest_applications].
* Aprenda a [administrar mediante programación cuentas y cuotas de Lote de Azure con .NET de administración de lotes](batch-management-dotnet.md). La biblioteca de [Batch Management .NET][api_net_mgmt] puede habilitar las características de creación y eliminación de cuentas de una aplicación o servicio de Batch.

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_mgmt]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[github_samples]: https://github.com/Azure/azure-batch-samples
[storage_pricing]: https://azure.microsoft.com/pricing/details/storage/
[net_appops]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.aspx
[net_appops_listappsummaries]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.listapplicationsummaries.aspx
[net_cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_cloudpool_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.applicationpackagereferences.aspx
[net_cloudtask]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudtask.aspx
[net_cloudtask_pkgref]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudtask.applicationpackagereferences.aspx
[net_nodestate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.state.aspx
[net_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationpackagereference.aspx
[portal]: https://portal.azure.com
[rest_applications]: https://msdn.microsoft.com/library/azure/mt643945.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[rest_add_pool_with_packages]: https://msdn.microsoft.com/library/azure/dn820174.aspx#bk_apkgreference

[1]: ./media/batch-application-packages/app_pkg_01.png "Diagrama de alto nivel de paquetes de aplicación"
[2]: ./media/batch-application-packages/app_pkg_02.png "Icono Aplicaciones en Azure Portal"
[3]: ./media/batch-application-packages/app_pkg_03.png "Hoja Aplicaciones en Azure Portal"
[4]: ./media/batch-application-packages/app_pkg_04.png "Hoja Detalles de aplicación en Azure Portal"
[5]: ./media/batch-application-packages/app_pkg_05.png "Hoja Nueva aplicación en Azure Portal"
[7]: ./media/batch-application-packages/app_pkg_07.png "Lista desplegable Actualizar o eliminar paquetes en Azure Portal"
[8]: ./media/batch-application-packages/app_pkg_08.png "Hoja Nuevo paquete de aplicación en Azure Portal"
[9]: ./media/batch-application-packages/app_pkg_09.png "Alerta de ausencia de cuenta de almacenamiento vinculada"
[10]: ./media/batch-application-packages/app_pkg_10.png "Hoja Elegir de cuenta de almacenamiento en Azure Portal"
[11]: ./media/batch-application-packages/app_pkg_11.png "Hoja Actualizar paquete en Azure Portal"
[12]: ./media/batch-application-packages/app_pkg_12.png "Cuadro de diálogo de confirmación de eliminación de paquetes en Azure Portal"

