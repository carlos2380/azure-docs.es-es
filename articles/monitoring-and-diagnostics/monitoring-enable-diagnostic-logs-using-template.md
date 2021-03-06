---
title: "Habilitación automática de Configuración de diagnóstico con una plantilla de Resource Manager | Microsoft Docs"
description: "Aprenda a usar una plantilla de Resource Manager para crear una configuración de diagnóstico que le permitirá transmitir los registros de diagnóstico a centros de eventos o almacenarlos en una cuenta de almacenamiento."
author: johnkemnetz
manager: rboucher
editor: 
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: a8a88a8c-4a48-4df6-8f7e-d90634d39c57
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/14/2017
ms.author: johnkem
translationtype: Human Translation
ms.sourcegitcommit: 197ebd6e37066cb4463d540284ec3f3b074d95e1
ms.openlocfilehash: 87403d68bfb57645417d6255329af7fd0d757f50
ms.lasthandoff: 03/31/2017


---
# <a name="automatically-enable-diagnostic-settings-at-resource-creation-using-a-resource-manager-template"></a>Habilitación automática de Configuración de diagnóstico al crear recursos con una plantilla de Resource Manager
En este artículo se muestra cómo usar una [plantilla de Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md) para establecer Configuración de diagnóstico en un recurso cuando se crea. Esto permite empezar automáticamente a transmitir las métricas y los registros de diagnóstico a Event Hubs, a archivarlos en una cuenta de almacenamiento o a enviarlos a Log Analytics cuando se crea un recurso.

El método para habilitar registros de diagnóstico mediante una plantilla de Resource Manager depende del tipo de recurso.

* **no de proceso** (por ejemplo, Grupos de seguridad de red, Logic Apps, Automatización) usan [Configuración de diagnóstico como se describe en este artículo](monitoring-overview-of-diagnostic-logs.md#diagnostic-settings).
* **de proceso** (basados en WAD/LAD) usan el [archivo de configuración de WAD/LAD descrito en este artículo](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md).

En este artículo se describe cómo configurar diagnósticos mediante cualquiera de estos métodos.

Los pasos básicos son los siguientes:

1. Cree una plantilla como archivo JSON que describa cómo crear el recurso y habilitar los diagnósticos.
2. [Implemente la plantilla mediante cualquier método de implementación](../azure-resource-manager/resource-group-template-deploy.md).

A continuación, se ofrece un ejemplo del archivo JSON de plantilla que debe generar para los recursos de proceso y los que no lo son.

## <a name="non-compute-resource-template"></a>Plantilla para recursos no de proceso
Para recursos no de proceso, debe hacer dos cosas:

1. Agregue parámetros al blob de parámetros para el nombre de cuenta de almacenamiento, el identificador de regla de Service Bus o el identificador del área de trabajo de Log Analytics de OMS (esto habilita el archivado de registros de diagnóstico en una cuenta de almacenamiento, el streaming de registros a Event Hubs o el envío de registros a Log Analytics).
   
    ```json
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Storage Account in which Diagnostic Logs should be saved."
      }
    },
    "serviceBusRuleId": {
      "type": "string",
      "metadata": {
        "description": "Service Bus Rule Id for the Service Bus Namespace in which the Event Hub should be created or streamed to."
      }
    },
    "workspaceId":{
      "type": "string",
      "metadata": {
        "description": "Log Analytics workspace ID for the Log Analytics workspace to which logs will be sent."
      }
    }
    ```
2. En la matriz de recursos del recurso para el que desea habilitar los registros de diagnóstico, agregue un recurso de tipo `[resource namespace]/providers/diagnosticSettings`.
   
    ```json
    "resources": [
      {
        "type": "providers/diagnosticSettings",
        "name": "Microsoft.Insights/service",
        "dependsOn": [
          "[/*resource Id for which Diagnostic Logs will be enabled>*/]"
        ],
        "apiVersion": "2015-07-01",
        "properties": {
          "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
          "serviceBusRuleId": "[parameters('serviceBusRuleId')]",
          "workspaceId": "[parameters('workspaceId')]",
          "logs": [ 
            {
              "category": "/* log category name */",
              "enabled": true,
              "retentionPolicy": {
                "days": 0,
                "enabled": false
              }
            }
          ],
          "metrics": [
            {
              "timeGrain": "PT1M",
              "enabled": true,
              "retentionPolicy": {
                "enabled": false,
                "days": 0
              }
            }
          ]
        }
      }
    ]
    ```

El blob de propiedades para la configuración de diagnóstico sigue [el formato descrito en este artículo](https://msdn.microsoft.com/library/azure/dn931931.aspx). Al agregar la propiedad `metrics`, también podrá enviar métricas de recursos para estos mismos resultados, siempre y cuando [los recursos admitan las métricas de Azure Monitor](monitoring-supported-metrics.md).

Este es un ejemplo completo en el que se crea una aplicación lógica y se activa la transmisión a Event Hubs y el almacenamiento en una cuenta de almacenamiento.

```json

{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "logicAppName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Logic App that will be created."
      }
    },
    "testUri": {
      "type": "string",
      "defaultValue": "http://azure.microsoft.com/en-us/status/feed/"
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Storage Account in which Diagnostic Logs should be saved."
      }
    },
    "serviceBusRuleId": {
      "type": "string",
      "metadata": {
        "description": "Service Bus Rule Id for the Service Bus Namespace in which the Event Hub should be created or streamed to."
      }
    },
    "workspaceId": {
      "type": "string",
      "metadata": {
        "description": "Log Analytics workspace ID for the Log Analytics workspace to which logs will be sent."
      }
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Logic/workflows",
      "name": "[parameters('logicAppName')]",
      "apiVersion": "2016-06-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "definition": {
          "$schema": "http://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "testURI": {
              "type": "string",
              "defaultValue": "[parameters('testUri')]"
            }
          },
          "triggers": {
            "recurrence": {
              "type": "recurrence",
              "recurrence": {
                "frequency": "Hour",
                "interval": 1
              }
            }
          },
          "actions": {
            "http": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('testUri')"
              },
              "runAfter": {}
            }
          },
          "outputs": {}
        },
        "parameters": {}
      },
      "resources": [
        {
          "type": "providers/diagnosticSettings",
          "name": "Microsoft.Insights/service",
          "dependsOn": [
            "[resourceId('Microsoft.Logic/workflows', parameters('logicAppName'))]"
          ],
          "apiVersion": "2015-07-01",
          "properties": {
            "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
            "serviceBusRuleId": "[parameters('serviceBusRuleId')]",
            "workspaceId": "[parameters('workspaceId')]",
            "logs": [
              {
                "category": "WorkflowRuntime",
                "enabled": true,
                "retentionPolicy": {
                  "days": 0,
                  "enabled": false
                }
              }
            ],
            "metrics": [
              {
                "timeGrain": "PT1M",
                "enabled": true,
                "retentionPolicy": {
                  "enabled": false,
                  "days": 0
                }
              }
            ]
          }
        }
      ],
      "dependsOn": []
    }
  ]
}

```

## <a name="compute-resource-template"></a>Plantilla para recursos de proceso
Para habilitar los diagnósticos en un recurso de proceso, por ejemplo, un clúster de Service Fabric o de máquinas virtuales, necesitará hacer lo siguiente:

1. Agregue la extensión de Diagnósticos de Azure a la definición de recurso de máquina virtual.
2. Especifique una cuenta de almacenamiento y/o un centro de eventos como parámetro.
3. Agregue el contenido del archivo XML de WADCfg a la propiedad XMLCfg y dé el formato de escape correcto a todos los caracteres XML.

> [!WARNING]
> Este último paso puede ser complicado de realizar correctamente. [Consulte este artículo](../virtual-machines/windows/extensions-diagnostics-template.md#diagnostics-configuration-variables) para ver un ejemplo que divide el esquema de configuración de diagnóstico en variables con el formato y el escape correctos.
> 
> 

Se describe el proceso completo, con ejemplos, [en este documento](../virtual-machines/windows/extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="next-steps"></a>Pasos siguientes
* [Más información sobre los registros de Diagnósticos de Azure](monitoring-overview-of-diagnostic-logs.md)
* [Transmita registros de diagnóstico de Azure a centros de eventos](monitoring-stream-diagnostic-logs-to-event-hubs.md)


