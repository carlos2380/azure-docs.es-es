---
title: "Administración de discos de Azure con Azure PowerShell | Microsoft Docs"
description: "Tutorial: Administración de discos de Azure con Azure PowerShell"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: neilpeterson
manager: timlt
editor: tysonn
tags: azure-service-management
ms.assetid: 
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/21/2017
ms.author: nepeters
translationtype: Human Translation
ms.sourcegitcommit: 1cc1ee946d8eb2214fd05701b495bbce6d471a49
ms.openlocfilehash: 6bc28290f5ae0466b8e1a260d411b7eab6f12439
ms.lasthandoff: 04/26/2017

---

# <a name="manage-azure-disks-with-powershell"></a>Administración de discos de Azure con PowerShell

En este tutorial, conocerá los diferentes tipos de discos de máquina virtual, aprenderá a seleccionar una configuración de disco, y sabrá cómo crear y conectar discos a máquinas virtuales de Azure. Este tutorial también explica cómo tomar instantáneas de discos.  

Se pueden completar los pasos de este tutorial con la versión más reciente del módulo [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/).

## <a name="default-azure-disks"></a>Discos de Azure predeterminados

Cuando se crea una máquina virtual de Azure, se conectan automáticamente dos discos a la máquina virtual. 

**Disco del sistema operativo**: se puede cambiar el tamaño de los discos del sistema operativo hasta 1 terabyte.Estos hospedan el sistema operativo de las máquinas virtuales.  Al disco del sistema operativo se le asigna una letra de unidad de `c:` de forma predeterminada. La configuración de almacenamiento en caché en disco del disco del sistema operativo está optimizada teniendo en cuenta el rendimiento del SO. El disco del sistema operativo **no debería** hospedar aplicaciones o datos. Para aplicaciones y datos, use un disco de datos. Explicamos esto más adelante en este artículo.

**Disco temporal**: los discos temporales utilizan una unidad de estado sólida que se encuentra en el mismo host de Azure que la máquina virtual. Además, son muy eficaces y se pueden usar para operaciones como el procesamiento de datos temporales. Sin embargo, si la máquina virtual se mueve a un nuevo host, se quitará cualquier dato almacenado en un disco temporal. El tamaño del disco temporal se determina según el tamaño de la máquina virtual. A los discos temporales se les asigna una letra de unidad de `d:` de forma predeterminada.

### <a name="temporary-disk-sizes"></a>Tamaños de los discos temporales

| Tipo | Tamaño de VM | Tamaño máximo de los discos temporales |
|----|----|----|
| [Uso general](sizes-general.md) | Series A y D | 800 |
| [Proceso optimizado](sizes-compute.md) | Serie F | 800 |
| [Memoria optimizada](../virtual-machines-windows-sizes-memory.md) | Series D y G | 6144 |
| [Almacenamiento optimizado](../virtual-machines-windows-sizes-storage.md) | Serie L | 5630 |
| [GPU](sizes-gpu.md) | Serie N | 1440 |
| [Alto rendimiento](sizes-hpc.md) | Series A y H | 2000 |

## <a name="azure-data-disks"></a>Discos de datos de Azure

Pueden agregarse más discos de datos para instalar aplicaciones y almacenar datos. Los discos de datos deben usarse en cualquier situación donde desee un almacenamiento de datos duradero y con capacidad de respuesta. Cada disco de datos tiene una capacidad máxima de 1 terabyte. El tamaño de la máquina virtual determina cuántos discos de datos se pueden conectar a una máquina virtual. Para cada núcleo de la máquina virtual, se pueden conectar dos discos de datos. 

### <a name="max-data-disks-per-vm"></a>Discos de datos máximos por máquina virtual

| Tipo | Tamaño de VM | Discos de datos máximos por máquina virtual |
|----|----|----|
| [Uso general](sizes-general.md) | Series A y D | 32 |
| [Proceso optimizado](sizes-compute.md) | Serie F | 32 |
| [Memoria optimizada](../virtual-machines-windows-sizes-memory.md) | Series D y G | 64 |
| [Almacenamiento optimizado](../virtual-machines-windows-sizes-storage.md) | Serie L | 64 |
| [GPU](sizes-gpu.md) | Serie N | 48 |
| [Alto rendimiento](sizes-hpc.md) | Series A y H | 32 |

## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual

Azure proporciona dos tipos de disco.

### <a name="standard-disk"></a>Disco estándar

El almacenamiento estándar está respaldado por unidades de disco duro y ofrece un almacenamiento rentable al mismo tiempo que tiene un rendimiento superior. Los discos estándar son ideales para cargas de trabajo de desarrollo y prueba rentables.

### <a name="premium-disk"></a>Disco Premium

Los discos Premium están respaldados por un disco de latencia reducida y alto rendimiento basado en SSD. Es perfecto para máquinas virtuales que ejecutan cargas de trabajo de producción. Premium Storage es compatible con las máquinas virtuales de las series DS, DSv2, GS y FS. Los discos Premium incluyen 3 tipos (P10, P20 y P30); el tamaño del disco determina el tipo de disco. Al seleccionar el tamaño de un disco, el valor se redondea al siguiente tipo. Por ejemplo, si el tamaño es inferior a 128 GB, el tipo de disco será P10, entre 129 y 512 P20, y más de 512 P30. 

### <a name="premium-disk-performance"></a>Rendimiento de los discos Premium

|Tipo de disco de Premium Storage | P10 | P20 | P30 |
| --- | --- | --- | --- |
| Tamaño del disco (redondeo hacia arriba) | 128 GB | 512 GB | 1024 GB (1 TB) |
| IOPS por disco | 500 | 2,300 | 5.000 |
Rendimiento de disco | 100 MB/s | 150 MB/s | 200 MB/s |

Aunque la tabla anterior identifica las IOPS máximas por disco, se puede obtener un mayor nivel de rendimiento dividiendo varios discos de datos. Por ejemplo, 64 discos de datos pueden conectarse a la máquina virtual Standard_GS5. Si cada uno de estos discos tienen un tamaño de P30, se puede lograr un máximo de 80 000 IOPS. Para obtener información detallada sobre las IOPS máximas por máquina virtual, consulte el artículo sobre [tamaños de máquinas virtuales Linux](./sizes.md).

## <a name="create-and-attach-disks"></a>Creación y conexión de discos

Para completar el ejemplo de este tutorial, debe tener una máquina virtual. Si es necesario, este [script de ejemplo](../scripts/virtual-machines-windows-powershell-sample-create-vm.md) puede crear una. Al trabajar en el tutorial, reemplace los nombres de grupo de recursos y máquina virtual cuando proceda.

Cree la configuración inicial con [New-AzureRmDiskConfig](/powershell/module/azurerm.compute/new-azurermdiskconfig). En el ejemplo siguiente se crea un disco denominado con un tamaño de 128 gigabytes.

```powershell
$diskConfig = New-AzureRmDiskConfig -Location westus -CreateOption Empty -DiskSizeGB 128
```

Cree el disco de datos con el comando [New-AzureRmDisk](/powershell/module/azurerm.compute/new-azurermdisk).

```powershell
$dataDisk = New-AzureRmDisk -ResourceGroupName myResourceGroup -DiskName myDataDisk -Disk $diskConfig
```

Obtenga la máquina virtual que desea agregar al disco de datos con el comando [Get-AzureRmVM](/powershell/module/azurerm.compute/get-azurermvm).

```powershell
$vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM
```

Agregue el disco de datos a la configuración de máquina virtual con el comando [Add-AzureRmVMDataDisk](/powershell/module/azurerm.compute/add-azurermvmdatadisk).

```powershell
$vm = Add-AzureRmVMDataDisk -VM $vm -Name myDataDisk -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 1
```

Actualice la máquina virtual con el comando [Update-AzureRmVM](/powershell/module/azurerm.compute/add-azurermvmdatadisk).

```powershell
Update-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm
```

## <a name="prepare-data-disks"></a>Preparación de los discos de datos

Cuando se haya conectado un disco a la máquina virtual, el sistema operativo debe configurarse para utilizar el disco. En el ejemplo siguiente se muestra cómo configurar manualmente el primer disco que se agrega a la máquina virtual. Este proceso también se puede automatizar utilizando la [extensión de script personalizado](./tutorial-automate-vm-deployment.md).

### <a name="manual-configuration"></a>Configuración manual

Crear una conexión RDP con la máquina virtual. Abra PowerShell y ejecute este script.

```powershell
Get-Disk | Where partitionstyle -eq 'raw' | `
Initialize-Disk -PartitionStyle MBR -PassThru | `
New-Partition -AssignDriveLetter -UseMaximumSize | `
Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información sobre los discos de máquina virtual. Avanzar hasta el siguiente tutorial para obtener información sobre cómo automatizar la configuración de máquina virtual.

[Automatización de la configuración de máquinas virtuales](./tutorial-automate-vm-deployment.md)
