---
title: "Preguntas más frecuentes sobre integración de registro de Azure | Microsoft Docs"
description: "Estas preguntas más frecuentes responden a las preguntas sobre la integración de registro de Azure."
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TerryLanfear
ms.assetid: d06d1ac5-5c3b-49de-800e-4d54b3064c64
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/07/2017
ms.author: TomSh
translationtype: Human Translation
ms.sourcegitcommit: 197ebd6e37066cb4463d540284ec3f3b074d95e1
ms.openlocfilehash: f539fc2945b9c6646660d50713d11dd7d822d06f
ms.lasthandoff: 03/31/2017


---
# <a name="azure-log-integration-frequently-asked-questions-faq"></a>Preguntas más frecuentes sobre la integración de registro de Azure (P+F)
Estas P+F responden a preguntas acerca de la integración de registro de Azure, un servicio que le permite integrar los registros sin procesar de los recursos de Azure en los sistemas locales de administración de eventos e información de seguridad (SIEM). Esta integración proporciona un panel unificado de todos los recursos, locales o en la nube, para que pueda agregar, correlacionar, analizar y alertar de eventos de seguridad asociados a las aplicaciones.

## <a name="is-the-azure-log-integration-software-free"></a>¿Es el software Integración de registro de Azure gratuito?
Sí. No hay ningún cargo por el software Integración de registro de Azure.

## <a name="where-is-azure-log-integration-available"></a>¿Dónde se encuentra disponible la integración de registro de Azure?

Está actualmente disponible en la versión comercial de Azure y en Azure Government. No se encuentra disponible ni en China ni en Alemania.

## <a name="how-can-i-see-the-storage-accounts-from-which-azure-log-integration-is-pulling-azure-vm-logs-from"></a>¿Cómo puedo ver las cuentas de almacenamiento desde las que el servicio de integración de registro de Azure extrae los registros de máquinas virtuales de Azure?
Ejecute el comando **azlog source list**.

## <a name="how-can-i-update-the-proxy-configuration"></a>¿Cómo se puede actualizar la configuración de proxy?
Si la configuración de proxy no permite el acceso a Azure Storage directamente, abra el archivo **AZLOG.EXE.CONFIG** en **c:\Program Files\Microsoft Azure Log Integration**. Actualice el archivo para que incluya la sección **defaultProxy** con la dirección del proxy de su organización. Después realizar la actualización, detenga e inicie el servicio mediante los comandos **net stop azlog** y **net start azlog**.

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.net>
        <connectionManagement>
          <add address="*" maxconnection="400" />
        </connectionManagement>
        <defaultProxy>
          <proxy usesystemdefault="true"
          proxyaddress=http://127.0.0.1:8888
          bypassonlocal="true" />
        </defaultProxy>
      </system.net>
      <system.diagnostics>
        <performanceCounters filemappingsize="20971520" />
      </system.diagnostics>   

## <a name="how-can-i-see-the-subscription-information-in-windows-events"></a>¿Cómo se puede ver la información de suscripción en los eventos de Windows?
Anexe el **subscriptionid** al nombre descriptivo al agregar el origen.

    Azlog source add <sourcefriendlyname>.<subscription id> <StorageName> <StorageKey>  
El archivo XML de eventos presenta los metadatos como se muestra a continuación, incluido el identificador de suscripción.

![XML de eventos][1]

## <a name="error-messages"></a>mensajes de error
### <a name="when-running-command-azlog-createazureid-why-do-i-get-the-following-error"></a>Cuando ejecuto el comando **azlog createazureid**, ¿por qué obtengo el siguiente error?
Error:

  *Failed to create AAD Application (Error al crear la aplicación de AAD): Inquilino 72f988bf-86f1-41af-91ab-2d7cd011db37 - Reason = 'Forbidden' - Message = 'No tiene privilegios suficientes para completar la operación'.*

**Azlog createazureid** intenta crear una entidad de servicio en todos los inquilinos de Azure AD de las suscripciones a las que el inicio de sesión de Azure tiene acceso. Si el inicio de sesión de Azure es solo el de un usuario invitado en ese inquilino de Azure AD, el comando generará el error 'No tiene privilegios suficientes para completar la operación'. Solicite al administrador de inquilinos que agregue su cuenta como una cuenta de usuario en el inquilino.

### <a name="when-running-command-azlog-authorize-why-do-i-get-the-following-error"></a>Cuando ejecuto el comando **azlog authorize**, ¿por qué obtengo el siguiente error?
Error:

  *Advertencia de creación de la asignación de rol - AuthorizationFailed: el cliente janedo@microsoft.com con objeto de identificador fe9e03e4-4dad-4328-910f-fd24a9660bd2 no tiene autorización para realizar la acción Microsoft.Authorization/roleAssignments/write en el ámbito /subscriptions/70d95299-d689-4c97-b971-0d8ff0000000.*

El comando **Azlog authorize** asigna el rol Lector a la entidad de servicio de Azure AD (creada con **Azlog createazureid**) para las suscripciones proporcionadas. Si el inicio de sesión de Azure no se realiza mediante una cuenta de coadministrador o de propietario de la suscripción, se producirá el error ‘Error de autorización’. Se necesita el control de acceso basado en roles de Azure (RBAC) de una cuenta de coadministrador o de propietario para completar esta acción.

## <a name="where-can-i-find-the-definition-of-the-properties-in-audit-log"></a>¿Dónde puedo encontrar la definición de las propiedades de registro de auditoría?
Consulte:

* [Operaciones de auditoría con Resource Manager](../azure-resource-manager/resource-group-audit.md)
* [List the management events in a subscription in Azure Monitor REST API](https://msdn.microsoft.com/library/azure/dn931934.aspx) (Lista de los eventos de administración de una suscripción en la API de REST de Azure Monitor)

## <a name="where-can-i-find-details-on-azure-security-center-alerts"></a>¿Dónde puedo encontrar detalles sobre las alertas de Azure Security Center?
Consulte [Administración y respuesta a las alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md).

## <a name="how-can-i-modify-what-is-collected-with-vm-diagnostics"></a>¿Cómo puedo modificar la información recopilada mediante el diagnóstico de máquinas virtuales?
Consulte [Uso de PowerShell para habilitar Diagnósticos de Azure en una máquina virtual con Windows](../virtual-machines/windows/ps-extensions-diagnostics.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para más información acerca de cómo obtener, modificar y configurar Diagnósticos de Azure en Windows *(WAD)* . A continuación se muestra un ejemplo:

### <a name="get-the-wad-config"></a>Obtención de la configuración de WAD
    -AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient
    $publicsettings = (Get-AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient).PublicSettings
    $encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
    $xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
    Write-Host $xmlconfig

    $xmlconfig | Out-File -Encoding utf8 -FilePath "d:\WADConfig.xml"

### <a name="modify-the-wad-config"></a>Modificación de la configuración de WAD
El ejemplo siguiente muestra una configuración en la que se recopilan solo los eventos EventID 4624 y EventId 4625 desde el registro de eventos de seguridad. Los eventos de Microsoft Antimalware se recopilan desde el registro de eventos del sistema. Consulte [Consuming Events] (https://msdn.microsoft.com/library/windows/desktop/dd996910(v=vs.85) para obtener más información sobre el uso de expresiones de XPath.

    <WindowsEventLog scheduledTransferPeriod="PT1M">
        <DataSource name="Security!*[System[(EventID=4624 or EventID=4625)]]" />
        <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>
    </WindowsEventLog>

### <a name="set-the-wad-configuration"></a>Configuración de WAD
    $diagnosticsconfig_path = "d:\WADConfig.xml"
    Set-AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName log3121 -StorageAccountKey <storage key>

Después de realizar cambios, compruebe la cuenta de almacenamiento para asegurarse de que se recopilan los eventos correctos.

Si experimenta problemas durante la instalación y la configuración, abra una [solicitud de soporte técnico](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request) y seleccione **Integración de registros** como el servicio para el que está solicitando soporte técnico.


<!--Image references-->
[1]: ./media/security-azure-log-integration-faq/event-xml.png

