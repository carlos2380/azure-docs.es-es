---
title: "Migración de VM de AWS a Azure| Microsoft Docs"
description: Migrar una instancia EC2 de Amazon Web Services (AWS) a Azure Virtual Machines. Este escenario utiliza Managed Disks para simplificar el almacenamiento en nube.
services: virtual-machines-windows
documentationcenter: 
author: cynthn
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 02/08/2017
ms.author: cynthn
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: a6549999003d4b1c8e2b8a8e2a2fafef942bce43
ms.lasthandoff: 04/27/2017


---

# <a name="migrate-from-amazon-web-services-aws-to-azure-managed-disks"></a>Migración de Amazon Web Services (AWS) a Azure Managed Disks

Puede migrar una instancia EC2 de Amazon Web Services (AWS) a Azure cargando el VHD. Si desea crear varias VM en Azure a partir de la misma imagen, primero debe generalizar la VM y, a continuación, exportar el VHD generalizado en un directorio local. Una vez cargado el VHD, puede crear una nueva VM de Azure que usa [Managed Disks](../../storage/storage-managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para el almacenamiento. Azure Managed Disks elimina la necesidad de administrar las cuentas de almacenamiento para las VM IaaS de Azure. Solo debe especificar el tipo (Premium o Estándar) y el tamaño del disco que necesita, y Azure creará y administrará el disco automáticamente. 

Antes de iniciar este proceso, asegúrese de revisar [Plan for the migration to Managed Disks](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks) (Planificación de la migración a Managed Disks).

Antes de cargar los VHD en Azure, debe consultar [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="before-you-begin"></a>Antes de empezar
Si usa PowerShell, asegúrese de que tiene la versión más reciente del módulo de PowerShell AzureRM.Compute. Ejecute el siguiente comando para instalarla.

```powershell
Install-Module AzureRM.Compute -MinimumVersion 2.6.0
```
Para más información, consulte [Azure PowerShell Versioning](/powershell/azure/overview) (Control de versiones de Azure PowerShell).


## <a name="generalize-the-windows-vm-using-sysprep"></a>Generalización de VM con Windows mediante Sysprep

Al generalizar una VM con Sysprep, se elimina cualquier información específica del equipo y la información de la cuenta personal del VHD y, además, se prepara el equipo que se usará como una imagen. Para obtener más información sobre Sysprep, vea [Uso de Sysprep: Introducción](http://technet.microsoft.com/library/bb457073.aspx).

Asegúrese de que los roles de servidor que se ejecutan en la máquina sean compatibles con Sysprep. Para más información, consulte [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> Si ejecuta Sysprep antes de cargar el VHD en Azure por primera vez, asegúrese de que tiene [preparada la máquina virtual](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) antes de ejecutar Sysprep. 
> 
> 

1. Inicie sesión en la máquina virtual de Windows.
2. Abra una ventana del símbolo del sistema como administrador. Cambie el directorio a **%windir%\system32\sysprep** y, después, ejecute `sysprep.exe`.
3. En **Herramienta de preparación del sistema**, seleccione **Iniciar la Configuración rápida (OOBE)** y asegúrese de que la casilla **Generalizar** está activada.
4. En **Opciones de apagado**, seleccione **Apagar**.
5. Haga clic en **Aceptar**.
   
    ![Iniciar Sysprep](./media/aws-to-azure/sysprepgeneral.png)
6. Cuando Sysprep finaliza, apaga la máquina virtual. No reinicie la VM.



## <a name="export-the-vhd-from-an-ec2-instance"></a>Exportación del VHD desde una instancia EC2

1.    Si usa Amazon Web Services (AWS), exporte la instancia EC2 a un VHD en un depósito de Amazon S3. Siga los pasos que se describen en la documentación de Azure para exportar instancias de Amazon EC2 para instalar la herramienta de interfaz de la línea de comandos (CLI) de Amazon EC2 y ejecute el comando create-instance-export-task para exportar la instancia de EC2 a un archivo VHD. Procure usar VHD para la variable DISK_IMAGE_FORMAT cuando ejecute el comando create-instance-export-task. El archivo VHD exportado se guarda en el depósito de Amazon S3 que haya especificado durante el proceso.

    ```
    aws ec2 create-instance-export-task --instance-id ID --target-environment TARGET_ENVIRONMENT '
    --export-to-s3-task DiskImageFormat=DISK_IMAGE_FORMAT,ContainerFormat=ova,S3Bucket=BUCKET,S3Prefix=PREFIX
    ```

2.    Descargue el archivo VHD desde el depósito de S3. Seleccione el archivo VHD y después **Acciones** > **Descargar**.



## <a name="log-in-to-azure"></a>Inicie sesión en Azure.
Si todavía no tiene la versión de PowerShell instalada, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/overview).

1. Abra Azure PowerShell e inicie sesión en la cuenta de Azure. Se abrirá una ventana emergente para que escriba sus credenciales de cuenta de Azure.
   
    ```powershell
    Login-AzureRmAccount
    ```
2. Obtenga los identificadores de suscripción para las suscripciones disponibles.
   
    ```powershell
    Get-AzureRmSubscription
    ```
3. Establezca la suscripción correcta con el identificador de suscripción. Reemplace `<subscriptionID>` con el identificador de la suscripción correcta.
   
    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"
    ```

## <a name="get-the-storage-account"></a>Obtención de la cuenta de almacenamiento
Necesita una cuenta de almacenamiento de Azure para almacenar la imagen de VM cargada. Puede usar una cuenta de almacenamiento existente o crear una nueva. 

Si va a usar el VHD para crear un disco administrado para una VM, la ubicación de la cuenta de almacenamiento debe ser la misma ubicación donde va a crear la VM.

Para mostrar las cuentas de almacenamiento disponibles, escriba:

```powershell
Get-AzureRmStorageAccount
```

Si desea utilizar una cuenta de almacenamiento existente, puede continuar con la sección [Carga de la imagen de VM en la cuenta de almacenamiento](#upload-the-vm-vhd-to-your-storage-account).

Si necesita crear una nueva cuenta de almacenamiento, siga estos pasos:

1. Necesita el nombre del grupo de recursos donde se debe crear la cuenta de almacenamiento. Para averiguar todos los grupos de recursos que están en la suscripción, escriba:
   
    ```powershell
    Get-AzureRmResourceGroup
    ```

    Para crear un grupo de recursos denominado **myResourceGroup** en la región **Oeste de EE. UU.**, escriba:

    ```powershell
    New-AzureRmResourceGroup -Name myResourceGroup -Location "West US"
    ```

2. Cree una cuenta de almacenamiento denominada **mystorageaccount** en este grupo de recursos con el cmdlet [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount):
   
    ```powershell
    New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "West US" `
        -SkuName "Standard_LRS" -Kind "Storage"
    ```
   
    Los valores válidos para - SkuName son los siguientes:
   
   * **Standard_LRS**: Almacenamiento con redundancia local. 
   * **Standard_ZRS**: Almacenamiento con redundancia de zona.
   * **Standard_GRS**: Almacenamiento con redundancia geográfica. 
   * **Standard_RAGRS**: Almacenamiento con redundancia geográfica con acceso de lectura. 
   * **Premium_LRS**: Almacenamiento con redundancia local premium. 

## <a name="upload-the-vhd-to-your-storage-account"></a>Carga del VHD en la cuenta de almacenamiento

Use el cmdlet [Add-AzureRmVhd](/powershell/module/azurerm.compute/add-azurermvhd) para cargar el VHD en un contenedor de su cuenta de almacenamiento. En este ejemplo se carga el archivo **myVHD.vhd** de `"C:\Users\Public\Documents\Virtual hard disks\"` para una cuenta de almacenamiento denominada **mystorageaccount** en el grupo de recursos **myResourceGroup**. El archivo se colocará en el contenedor llamado **mycontainer** y el nuevo nombre de archivo será **myUploadedVHD.vhd**.

```powershell
$rgName = "myResourceGroup"
$urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd"
Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
    -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
```


Si se realiza correctamente, obtendrá una respuesta similar a la siguiente:

```powershell
MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
MD5 hash calculation is completed.
Elapsed time for the operation: 00:03:35
Creating new page blob of size 53687091712...
Elapsed time for upload: 01:12:49

LocalFilePath           DestinationUri
-------------           --------------
C:\Users\Public\Doc...  https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd
```

Dependiendo de la conexión de red y del tamaño del archivo VHD, este comando tardará algún tiempo en completarse.

Guarde la ruta de acceso del **URI de destino** para usarla más adelante si va a crear un disco administrado o una nueva VM usando el VHD cargado.

### <a name="other-options-for-uploading-a-vhd"></a>Otras opciones para cargar un disco duro virtual

También puede cargar un VHD en la cuenta de almacenamiento mediante uno de los siguientes medios:

-   [API de tipo Copy Blob de Almacenamiento Azure](https://msdn.microsoft.com/library/azure/dd894037.aspx)

-   [Blobs de carga de Azure Storage Explorer](https://azurestorageexplorer.codeplex.com/)

-   [Referencia de la API de REST de servicios de importación y exportación de almacenamiento](https://msdn.microsoft.com/library/dn529096.aspx)

    Se recomienda utilizar el servicio Import/Export si se calcula que el tiempo de carga va a ser superior a siete días. Puede usar [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) para calcular el tiempo a partir del tamaño de los datos y la unidad de transferencia. 

    Import/Export se puede usar para realizar copias en una cuenta de almacenamiento estándar. Para copiar desde el almacenamiento estándar a una cuenta de Almacenamiento premium, use una herramienta como AzCopy.

## <a name="create-a-managed-image-from-the-uploaded-vhd"></a>Creación de una imagen administrada desde el VHD cargado 

Cree una imagen administrada con el VHD del SO generalizado.


1.  En primer lugar, establezca los parámetros comunes:

    ```powershell
    $rgName = "myResourceGroupName"
    $vmName = "myVM"
    $location = "West Central US" 
    $imageName = "yourImageName"
    $osVhdUri = "https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd"
    ```

4.  Cree la imagen con el VHD del SO generalizado.

    ```powershell
    $imageConfig = New-AzureRmImageConfig -Location $location
    $imageConfig = Set-AzureRmImageOsDisk -Image $imageConfig -OsType Windows -OsState Generalized -BlobUri $osVhdUri
    $image = New-AzureRmImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ```

## <a name="setup-some-variables-for-the-image"></a>Configuración de algunas variables para la imagen

Primero se debe recopilar información básica sobre la imagen y crear una variable para la imagen. En este ejemplo se usa una imagen de VM administrada denominada **myImage** que se encuentra en el grupo de recursos **myResourceGroup** en la ubicación **Centro occidental de EE. UU.** 

```powershell
$rgName = "myResourceGroup"
$location = "West Central US"
$imageName = "myImage"
$image = Get-AzureRMImage -ImageName $imageName -ResourceGroupName $rgName
```

## <a name="create-a-virtual-network"></a>Crear una red virtual
Cree la red virtual y la subred de la [red virtual](../../virtual-network/virtual-networks-overview.md).

1. Cree la subred. Este ejemplo crea una subred denominada **mySubnet** con el prefijo de dirección **10.0.0.0/24**.  
   
    ```powershell
    $subnetName = "mySubnet"
    $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
    ```
2. Creación de la red virtual. Este ejemplo crea una red virtual denominada **myVnet** con el prefijo de dirección **10.0.0.0/16**.  
   
    ```powershell
    $vnetName = "myVnet"
    $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
        -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
    ```    

## <a name="create-a-public-ip-address-and-network-interface"></a>Creación de una dirección IP y una interfaz de red

Para permitir la comunicación con la máquina virtual en la red virtual, necesitará una [dirección IP pública](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) y una interfaz de red.

1. Cree una dirección IP pública. Este ejemplo crea una dirección IP pública denominada "**myPip**". 
   
    ```powershell
    $ipName = "myPip"
    $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
        -AllocationMethod Dynamic
    ```       
2. Cree la NIC. Este ejemplo crea una NIC denominada "**myNic**". 
   
    ```powershell
    $nicName = "myNic"
    $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location `
        -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
    ```

## <a name="create-the-network-security-group-and-an-rdp-rule"></a>Creación del grupo de seguridad de red y las reglas de RDP

Para poder iniciar sesión en la VM mediante RDP, debe tener una regla de seguridad de red (NSG) que permita el acceso RDP en el puerto 3389. 

Este ejemplo crea un NSG denominado"**myNsg**" que contiene una regla llamada "**myRdpRule**" que permita el tráfico RDP en el puerto 3389. Para obtener más información sobre NSG, consulte [Apertura de puertos para una máquina virtual en Azure mediante PowerShell](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

```powershell
$nsgName = "myNsg"
$ruleName = "myRdpRule"
$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name $ruleName -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389

$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name $nsgName -SecurityRules $rdpRule
```


## <a name="create-a-variable-for-the-virtual-network"></a>Creación de una variable para la red virtual

Cree una variable para la red virtual completada. 

```powershell
$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName

```

## <a name="get-the-credentials-for-the-vm"></a>Obtención de las credenciales para la VM

El siguiente cmdlet abrirá una ventana en la que escribirá un nuevo nombre de usuario y una contraseña que se usará como la cuenta de administrador local para tener acceso remoto a la VM. 

```powershell
$cred = Get-Credential
```

## <a name="set-variables-for-the-vm-name-computer-name-and-the-size-of-the-vm"></a>Definición de las variables para el nombre de VM, el nombre del equipo y el tamaño de la VM

1. Cree variables para el nombre de VM y el nombre del equipo. Este ejemplo establece el nombre de VM como **myVM** y el nombre del equipo como **myComputer**.

    ```powershell
    $vmName = "myVM"
    $computerName = "myComputer"
    ```
2. Establezca el tamaño de la máquina virtual. Este ejemplo crea la VM con el tamaño **Standard_DS1_v2**. Consulte la documentación sobre los [tamaños de VM](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/) para más información.

    ```powershell
    $vmSize = "Standard_DS1_v2"
    ```

3. Agregue el nombre y el tamaño de la VM a la configuración de la VM.

```powershell
$vm = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize
```

## <a name="set-the-vm-image-as-source-image-for-the-new-vm"></a>Establecimiento de la imagen de VM como imagen de origen para la nueva VM

Establezca la imagen de origen con el identificador de la imagen de VM administrada.

```powershell
$vm = Set-AzureRmVMSourceImage -VM $vm -Id $image.Id
```

## <a name="set-the-os-configuration-and-add-the-nic"></a>Establezca la configuración del SO y agregue la NIC.

Escriba el tipo de almacenamiento (PremiumLRS o StandardLRS) y el tamaño del disco del SO. Este ejemplo establece el tipo de cuenta en **PremiumLRS**, el tamaño del disco en **128 GB** y el almacenamiento en caché en disco en **ReadWrite**.

```powershell
$vm = Set-AzureRmVMOSDisk -VM $vm  -ManagedDiskStorageAccountType PremiumLRS -DiskSizeInGB 128 `
-CreateOption FromImage -Caching ReadWrite

$vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName $computerName `
-Credential $cred -ProvisionVMAgent -EnableAutoUpdate

$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
```

## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree la nueva VM utilizando la configuración creada y almacenada en la variable **$vm**.

```powershell
New-AzureRmVM -VM $vm -ResourceGroupName $rgName -Location $location
```

## <a name="verify-that-the-vm-was-created"></a>Comprobación de que se creó la máquina virtual
Cuando finalice, debería ver la máquina virtual recién creada en el [Azure Portal](https://portal.azure.com), en **Examinar** > **Máquinas virtuales** o mediante los comandos de PowerShell siguientes:

```powershell
    $vmList = Get-AzureRmVM -ResourceGroupName $rgName
    $vmList.Name
```

## <a name="next-steps"></a>Pasos siguientes

Para iniciar sesión en la nueva máquina virtual, examine la máquina virtual en el [Portal](https://portal.azure.com), haga clic en **Conectar**y abra el archivo RDP del Escritorio remoto. Utilice las credenciales de cuenta de la máquina virtual original para iniciar sesión en la nueva máquina virtual. Para más información, consulte [Conexión a una máquina virtual de Azure donde se ejecuta Windows Server e inicio de sesión en ella](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 
 

