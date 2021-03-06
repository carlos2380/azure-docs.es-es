---
title: "Creación de una máquina virtual Linux en Azure con varias NIC | Microsoft Docs"
description: "Aprenda a crear una máquina virtual Linux con varias NIC conectadas a ella mediante la CLI de Azure o plantillas de Resource Manager."
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/09/2017
ms.author: iainfou
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 40911f9e663361dfb3a474970c727736cc0225b8
ms.lasthandoff: 04/03/2017


---
# <a name="create-a-linux-vm-with-multiple-nics-using-the-azure-cli-10"></a>Creación de una máquina virtual de Linux con varias NIC mediante la CLI de Azure 1.0
Puede crear una máquina virtual (VM) en Azure que tenga asociadas varias interfaces de red virtual (NIC). Un escenario común sería tener distintas subredes para la conectividad front-end y back-end o una red dedicada a una solución de supervisión o copia de seguridad. En este artículo se proporcionan comandos rápidos para crear una máquina virtual que tiene conectadas varias NIC. Para más información, lo que incluye cómo crear varias NIC dentro de sus propios scripts de Bash, lea más sobre la [implementación de máquinas virtuales con varias NIC](../../virtual-network/virtual-network-deploy-multinic-arm-cli.md). Diferentes [tamaños de máquina virtual](sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) admiten un número distinto de NIC, así que ajuste el tamaño de su máquina virtual teniendo esto en cuenta.

> [!WARNING]
> Cuando crea una máquina virtual, debe asociar varias NIC; no es posible agregar NIC a una máquina virtual existente. También puede [crear una máquina virtual en función de los discos virtuales originales](copy-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) y crear varias NIC mientras implementa la máquina virtual.


## <a name="cli-versions-to-complete-the-task"></a>Versiones de la CLI para completar la tarea
Puede completar la tarea mediante una de las siguientes versiones de la CLI:

- [CLI de Azure 1.0](#create-supporting-resources): la CLI para los modelos de implementación clásico y de Resource Manager (este artículo)
- [CLI de Azure 2.0](../windows/multiple-nics.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json): la CLI de última generación para el modelo de implementación de administración de recursos


## <a name="create-supporting-resources"></a>Creación de recursos de apoyo
Asegúrese de haber iniciado sesión en la [CLI de Azure](../../cli-install-nodejs.md) y que usa el modo de Resource Manager:

```azurecli
azure config mode arm
```

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Nombres de parámetros de ejemplo incluidos `myResourceGroup`, `mystorageaccount` y `myVM`.

En primer lugar, cree un grupo de recursos. En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup` en la ubicación `WestUS`:

```azurecli
azure group create myResourceGroup -l WestUS
```

Cree una cuenta de almacenamiento que contenga las máquinas virtuales: En el ejemplo siguiente se crea una cuenta de almacenamiento denominada `mystorageaccount`:

```azurecli
azure storage account create mystorageaccount -g myResourceGroup \
    -l WestUS --kind Storage --sku-name PLRS
```

Cree una red virtual para conectar las máquinas virtuales a: En el ejemplo siguiente se crea una red virtual denominada `myVnet` con un prefijo de dirección de `192.168.0.0/16`:

```azurecli
azure network vnet create -g myResourceGroup -l WestUS \
    -n myVnet -a 192.168.0.0/16
```

Cree dos subredes de red virtual: una para el tráfico front-end y otra para el tráfico back-end. En el ejemplo siguiente se crean dos subredes, denominadas `mySubnetFrontEnd` y `mySubnetBackEnd`:

```azurecli
azure network vnet subnet create -g myResourceGroup -e myVnet \
    -n mySubnetFrontEnd -a 192.168.1.0/24
azure network vnet subnet create -g myResourceGroup -e myVnet \
    -n mySubnetBackEnd -a 192.168.2.0/24
```

## <a name="create-and-configure-multiple-nics"></a>Creación y configuración de varias NIC
Puede leer información más detallada sobre la [implementación de varias NIC con la CLI de Azure](../../virtual-network/virtual-network-deploy-multinic-arm-cli.md), como los scripts del proceso de paso de bucle para crear todas las NIC.

En el ejemplo siguiente se crean dos NIC, denominadas `myNic1` y `myNic2`, una de las cuales se conecta con cada subred:

```azurecli
azure network nic create --resource-group myResourceGroup --location WestUS \
    -n myNic1 --subnet-vnet-name myVnet --subnet-name mySubnetFrontEnd
azure network nic create --resource-group myResourceGroup --location WestUS \
    -n myNic2 --subnet-vnet-name myVnet --subnet-name mySubnetBackEnd
```

Normalmente también crearía un [grupo de seguridad de red](../../virtual-network/virtual-networks-nsg.md) o [un equilibrador de carga](../../load-balancer/load-balancer-overview.md) para administrar y distribuir el tráfico entre las máquinas virtuales. En el ejemplo siguiente, se crea un grupo de seguridad de red denominado `myNetworkSecurityGroup`:

```azurecli
azure network nsg create --resource-group myResourceGroup --location WestUS \
    --name myNetworkSecurityGroup
```

Enlace las NIC al grupo de seguridad de red mediante `azure network nic set`. En el ejemplo siguiente, se enlaza `myNic1` y `myNic2` con `myNetworkSecurityGroup`:

```azurecli
azure network nic set --resource-group myResourceGroup --name myNic1 \
    --network-security-group-name myNetworkSecurityGroup
azure network nic set --resource-group myResourceGroup --name myNic2 \
    --network-security-group-name myNetworkSecurityGroup
```

## <a name="create-a-vm-and-attach-the-nics"></a>Creación de una máquina virtual y conexión de las NIC
Al crear la máquina virtual, ahora especifica varias NIC. En lugar de usar `--nic-name` para proporcionar una única NIC, usará `--nic-names` y proporcionará una lista de NIC separadas por coma. También debe tener cuidado al seleccionar el tamaño de la máquina virtual. Existen límites para el número total de NIC que se pueden agregar a una máquina virtual. Más información sobre los [tamaños de máquina virtual Linux](sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). En el ejemplo siguiente se muestra cómo especificar varias NIC y, luego, un tamaño de máquina virtual que admita el uso de varias NIC (`Standard_DS2_v2`):

```azurecli
azure vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location WestUS \
    --os-type linux \
    --nic-names myNic1,myNic2 \
    --vm-size Standard_DS2_v2 \
    --storage-account-name mystorageaccount \
    --image-urn UbuntuLTS \
    --admin-username azureuser \
    --ssh-publickey-file ~/.ssh/id_rsa.pub
```

## <a name="create-multiple-nics-using-resource-manager-templates"></a>Creación de varias NIC con plantillas de Resource Manager
Las plantillas de Azure Resource Manager emplean archivos JSON declarativos para definir el entorno. Puede leer la [introducción a Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md). Las plantillas de Resource Manager ofrecen una manera de crear varias instancias de un recurso durante la implementación; por ejemplo, se pueden crear varias NIC. Utilizará el comando *copy* para especificar el número de instancias que se crearán:

```json
"copy": {
    "name": "multiplenics"
    "count": "[parameters('count')]"
}
```

Más información sobre la [creación de varias instancias mediante *copia*](../../resource-group-create-multiple.md). 

También puede utilizar `copyIndex()` para anexar un número a un nombre de recurso, lo que le permite crear `myNic1`, `myNic2`, etc. A continuación se muestra un ejemplo de cómo anexar el valor de índice:

```json
"name": "[concat('myNic', copyIndex())]", 
```

Puede leer un ejemplo completo de [cómo crear varias NIC con plantillas de Resource Manager](../../virtual-network/virtual-network-deploy-multinic-arm-template.md).

## <a name="next-steps"></a>Pasos siguientes
Asegúrese de revisar los [tamaños de máquina virtual Linux](sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) al intentar crear una máquina virtual con varias NIC. Preste atención al número máximo de NIC que admite cada tamaño de máquina virtual. 

Recuerde que no se pueden agregar NIC adicionales a una máquina virtual existente; debe crear todas las NIC al implementar la máquina virtual. Tenga cuidado al planear las implementaciones para asegurarse de que dispone de toda la conectividad de red necesaria desde el principio.


