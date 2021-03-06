---
title: "Creación de centro de eventos de Azure | Microsoft Docs"
description: "Creación de un espacio de nombres de Azure Event Hubs y un centro de eventos con Azure Portal"
services: event-hubs
documentationcenter: na
author: jtaubensee
manager: timlt
editor: 
ms.assetid: ff99e327-c8db-4354-9040-9c60c51a2191
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/25/2017
ms.author: jotaub
translationtype: Human Translation
ms.sourcegitcommit: db7cb109a0131beee9beae4958232e1ec5a1d730
ms.openlocfilehash: 23df3a3a11d8f065d6ce2a4f14ba175d6c781ee9
ms.lasthandoff: 04/18/2017

---

# <a name="create-an-event-hubs-namespace-and-an-event-hub-using-the-azure-portal"></a>Creación de un espacio de nombres de Event Hubs y un centro de eventos con Azure Portal

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
1. Inicie sesión en [Azure Portal][Azure portal] y haga clic en **Nuevo** en la parte superior izquierda de la pantalla.
1. Haga clic en **Internet de las cosas** y, luego, en **Event Hubs**.
   
    ![](./media/event-hubs-create/create-event-hub9.png)
1. En la hoja **Crear espacio de nombres** , especifique el nombre del espacio de nombres. El sistema realiza la comprobación automáticamente para ver si el nombre está disponible.
   
    ![](./media/event-hubs-create/create-event-hub1.png)
1. Después de asegurarse de que el nombre del espacio de nombres está disponible, elija el plan de tarifa (Básico o Estándar). Elija también una suscripción de Azure, un grupo de recursos y la ubicación en la que se va a crear el recurso. 
1. Haga clic en **Crear** para crear el espacio de nombres.
1. En la lista de espacios de nombres de los Centros de eventos, haga clic en el espacio de nombres recién creado.      
   
    ![](./media/event-hubs-create/create-event-hub2.png)
1. En la hoja del espacio de nombres, haga clic en **Centros de eventos**.
   
    ![](./media/event-hubs-create/create-event-hub3.png)

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

1. En la parte superior de la hoja, haga clic en **Agregar Centro de eventos**.
   
    ![](./media/event-hubs-create/create-event-hub4.png)
1. Escriba el nombre del centro de eventos y, luego, haga clic en **Crear**.
   
    ![](./media/event-hubs-create/create-event-hub5.png)
1. En la lista de centros de eventos, haga clic en el nombre del centro de eventos recién creado. 
    
    ![](./media/event-hubs-create/create-event-hub6.png)
1. En la hoja del espacio de nombres (no en la hoja del centro de eventos específico), haga clic en **Directivas de acceso compartido** y, luego, en **RootManageSharedAccessKey**.
    
    ![](./media/event-hubs-create/create-event-hub7.png)
1. Haga clic en el botón Copiar para copiar la cadena de conexión **RootManageSharedAccessKey** al Portapapeles. Guarde esta cadena de conexión para usarla más adelante en el tutorial.
    
    ![](./media/event-hubs-create/create-event-hub8.png)

Ya se ha creado el centro de eventos y cuenta con las cadenas de conexión que necesita para enviar y recibir eventos.

## <a name="next-steps"></a>Pasos siguientes
Para aprender más acerca de Event Hubs, consulte estos vínculos:

* [Información general de Event Hubs](event-hubs-overview.md)
* [Información general de la API de Event Hubs](event-hubs-api-overview.md)

[Azure portal]: https://portal.azure.com/
