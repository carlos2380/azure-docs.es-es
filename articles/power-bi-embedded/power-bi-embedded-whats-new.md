---
title: Novedades en Power BI Embedded
description: "Obtenga la información más reciente sobre las novedades en Power BI Embedded."
services: power-bi-embedded
documentationcenter: 
author: guyinacube
manager: erikre
editor: 
tags: 
ms.assetid: 2794ae98-b9a7-45df-b6e1-962a395b91fa
ms.service: power-bi-embedded
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: powerbi
ms.date: 03/11/2017
ms.author: asaxton
translationtype: Human Translation
ms.sourcegitcommit: c1cd1450d5921cf51f720017b746ff9498e85537
ms.openlocfilehash: 07c53a5d6b1881a4c207a2aefed9fcede0fa069e
ms.lasthandoff: 03/14/2017

---
# <a name="whats-new-in-power-bi-embedded"></a>Novedades en Power BI Embedded

De forma periódica se publican actualizaciones a **Power BI Embedded** . Sin embargo, no todas las versiones incluirán nuevas características de cara al usuario; algunas se centran en las funcionalidades del servicio back-end. Analizaremos aquí las nuevas funcionalidades para el usuario. Asegúrese de revisarlas periódicamente.

## <a name="march-2017"></a>Marzo de 2017

<iframe width="640" height="360" src="https://www.youtube.com/embed/ibuN4DzCl5c?showinfo=0" frameborder="0" allowfullscreen></iframe>

**Funcionalidades de autoservicio**

* [Creación de un informe nuevo](power-bi-embedded-create-report-from-dataset.md)
* [Informe SaveAs](power-bi-embedded-save-reports.md)
* Insertar informe en modo de lectura/edición/creación de nuevo 
* [Toggle report between edit/read modes](power-bi-embedded-toggle-mode.md) (Alternancia de informe entre los modos de edición/lectura)

**Conectividad a datos con las API de REST**

* [Creación de un conjunto de datos](https://msdn.microsoft.com/library/azure/mt778875.aspx)
* Inserción de datos 

**API de administración**

* Clonación de informe y conjunto de datos
* Enlace de un informe a otro conjunto de datos

**Ejemplos**

* Actualización del [ejemplo de inserción de informe de JavaScript](https://microsoft.github.io/PowerBI-JavaScript/demo)

## <a name="december-2016"></a>Diciembre de 2016

* [Nuevo ejemplo de inserción de JavaScript](https://microsoft.github.io/PowerBI-JavaScript/demo/)

## <a name="october-2016"></a>Octubre de 2016

* [Advanced Analytics con Power BI Embedded y R](https://powerbi.microsoft.com/blog/r-in-pbie/)

## <a name="august-31st-2016"></a>31 de agosto de 2016
En esta versión se incluyen:

* Todos los nuevos SDK de JavaScript que admita el [filtro avanzado y la exploración de páginas](power-bi-embedded-interact-with-reports.md).
* Power BI Embedded ahora es compatible con el centro de datos de Canadá Central. Compruebe el [estado del centro de datos](https://azure.microsoft.com/status/).

## <a name="july-11th-2016"></a>11 de julio de 2016
En esta versión se incluyen:

* **Buenas noticias** El servicio Power BI Embedded ya no está en versión preliminar: ahora ya está disponible de manera general.  
* Todas las API de REST han pasado de **/beta** a **/v1.0**.
* Los SDK de JavaScript y .NET se han actualizado a **v1.0**.
* Ahora se pueden autenticar las llamadas de API de BI Power directamente mediante claves de API. Los tokens de aplicación solo se necesitan para la inserción. Como parte de esto, los tokens de aprovisionamiento y desarrollo han quedado en desuso en la versión v1.0 API, pero seguirán funcionando en la versión beta hasta el 30 de diciembre de 2016. Para obtener más información, consulte [Acerca del flujo del token de aplicación en Power BI Embedded](power-bi-embedded-app-token-flow.md).
* Compatibilidad de la seguridad de nivel de fila (RLS) con tokens de aplicación e informes insertados. Para obtener más información, consulte [Seguridad de nivel de fila con Power BI Embedded](power-bi-embedded-rls.md).
* Aplicación de ejemplo actualizada para todas las llamadas de API **v1.0** .
* Compatibilidad de Power BI Embedded con SDK de Azure, PowerShell y CLI.
* Los usuarios pueden exportar datos de visualización a un archivo **.csv**.
* Power BI Embedded ahora es compatible con los mismos idiomas o configuraciones regionales que Microsoft Azure. Para obtener más información, consulte [Azure - Idiomas](http://social.technet.microsoft.com/wiki/contents/articles/4234.windows-azure-extent-of-localization.aspx).


