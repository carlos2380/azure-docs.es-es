---
title: Rendimiento reducido de las aplicaciones web en App Service | Microsoft Docs
description: "Este artículo lo ayuda a solucionar los problemas de rendimiento reducido en las aplicaciones web del Servicio de aplicaciones de Azure."
services: app-service\web
documentationcenter: 
author: cephalin
manager: erikre
editor: 
tags: top-support-issue
keywords: "rendimiento de aplicaciones web, aplicación lenta, aplicaciones lentas"
ms.assetid: b8783c10-3a4a-4dd6-af8c-856baafbdde5
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2016
ms.author: cephalin
translationtype: Human Translation
ms.sourcegitcommit: 5ea043ce3bcd0f500fd765f13764ea3ee83e1ba9
ms.openlocfilehash: 83c3592014c73c0cf36d371d2752bc76b7c8a4e8
ms.lasthandoff: 02/16/2017


---
# <a name="troubleshoot-slow-web-app-performance-issues-in-azure-app-service"></a>Solucionar los problemas de rendimiento reducido de aplicaciones web en el Servicio de aplicaciones de Azure
Este artículo lo ayuda a solucionar los problemas de rendimiento reducido en las aplicaciones web del [Servicio de aplicaciones de Azure](http://go.microsoft.com/fwlink/?LinkId=529714).

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y haga clic en **Obtener soporte técnico**.

## <a name="symptom"></a>Síntoma
Al examinar la aplicación web, las páginas se cargan lentamente y, en ocasiones, se agota el tiempo de espera.

## <a name="cause"></a>Causa
Con frecuencia este problema se debe a problemas en el nivel de la aplicación, como, por ejemplo:

* Solicitudes que tardan mucho tiempo
* Aplicaciones que hacen un uso elevado de memoria y CPU
* Aplicaciones que se bloquean debido a una excepción.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
El procedimiento de solución de problemas se puede dividir en tres tareas distintas, en orden secuencial:

1. [Observación y supervisión del comportamiento de la aplicación](#observe)
2. [Recopilación de datos](#collect)
3. [Mitigación del problema](#mitigate)

[Aplicaciones web del Servicio de aplicaciones](/services/app-service/web/) ofrece diversas opciones en cada paso.

<a name="observe" />

### <a name="1-observe-and-monitor-application-behavior"></a>1. Observación y supervisión del comportamiento de la aplicación
#### <a name="track-service-health"></a>Seguimiento del estado del servicio
Cada vez que hay una interrupción del servicio o una degradación del rendimiento, Microsoft Azure lo anuncia. Puede realizar un seguimiento del estado del servicio en el [Portal de Azure](https://portal.azure.com/). Para más información, consulte [Track service health](../monitoring-and-diagnostics/insights-service-health.md) (Seguimiento del estado del servicio).

#### <a name="monitor-your-web-app"></a>Supervisión de la aplicación web
Esta opción le permite averiguar si la aplicación tiene problemas. En la hoja de la aplicación web, haga clic en el icono **Solicitudes y errores** . La hoja **Métrica** mostrará todas las métricas que puede agregar.

Algunas de las métricas que podría querer supervisar para su aplicación web son:

* Espacio de trabajo de memoria promedio
* Tiempo medio de respuesta
* Tiempo de CPU
* Espacio de trabajo de memoria
* Solicitudes

![supervisar el rendimiento de aplicaciones web](./media/app-service-web-troubleshoot-performance-degradation/1-monitor-metrics.png)

Para más información, consulte:

* [Supervisión de Aplicaciones web en Servicio de aplicaciones de Azure](web-sites-monitor.md)
* [Recibir notificaciones de alerta](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)

#### <a name="monitor-web-endpoint-status"></a>estado de extremo web
Si ejecuta la aplicación web en el plan de tarifa **Estándar** , el servicio Aplicaciones web le permite supervisar 2 puntos de conexión de 3 ubicaciones geográficas.

La supervisión de extremo configura pruebas web desde ubicaciones distribuidas geográficamente que prueban el tiempo de respuesta y el tiempo activo de direcciones URL web. La prueba realiza una operación HTTP Get en la dirección URL web para determinar el tiempo de respuesta y el tiempo de actividad desde cada ubicación. Cada ubicación configurada ejecuta una prueba cada cinco minutos.

El tiempo activo se supervisa a través de códigos de respuesta de HTTP y el tiempo de respuesta se mide en milisegundos. Una prueba de supervisión da error si el código de respuesta HTTP es mayor o igual que 400 o si la respuesta demora más de 30 segundos. Un extremo se considera disponible si sus pruebas de supervisión se realizan correctamente desde todas las ubicaciones especificadas.

Para configurarlo, consulte [Supervisión de Aplicaciones en Azure App Service](web-sites-monitor.md).

Consulte también [Mantenimiento de Sitios web de Azure activos y supervisión de puntos de conexión: con Stefan Schackow](https://channel9.msdn.com/Shows/Azure-Friday/Keeping-Azure-Web-Sites-up-plus-Endpoint-Monitoring-with-Stefan-Schackow) para ver un vídeo sobre la supervisión de los puntos de conexión.

#### <a name="application-performance-monitoring-using-extensions"></a>Supervisión del rendimiento de la aplicación mediante el uso de extensiones
También puede supervisar el rendimiento de la aplicación aprovechando las *extensiones de sitio*.

Cada aplicación web del Servicio de aplicaciones proporciona un punto de conexión de administración extensible que le permite aprovechar un eficaz conjunto de herramientas implementadas como extensiones del sitio. Estas herramientas van desde editores de código fuente, como [Visual Studio Team Services](https://www.visualstudio.com/products/what-is-visual-studio-online-vs.aspx) hasta herramientas de administración de recursos conectados, como una base de datos MySQL conectada a una aplicación web.

[Azure Application Insights](/services/application-insights/) y [New Relic](/marketplace/partners/newrelic/newrelic/) son dos de las extensiones de sitio de supervisión del rendimiento que se encuentran disponibles. Para usar New Relic, instale a un agente en tiempo de ejecución. Para usar Application Insights de Azure, vuelva a compilar el código con un SDK; también puede instalar una extensión que proporciona acceso a datos adicionales. El SDK permite escribir código para supervisar el uso y el rendimiento de la aplicación con más detalle.

Para usar Application Insights, consulte [Supervisión del rendimiento en las aplicaciones web](../application-insights/app-insights-web-monitor-performance.md).

Para usar New Relic, consulte [Administración del rendimiento de las aplicaciones de New Relic en Azure](../store-new-relic-cloud-services-dotnet-application-performance-management.md).

<a name="collect" />

### <a name="2-collect-data"></a>2. Recopilación de datos
#### <a name="enable-diagnostics-logging-for-your-web-app"></a>Habilitación del registro de diagnóstico para una aplicación web
El entorno de Aplicaciones web proporciona funciones de diagnóstico para registrar información del servidor web y de la aplicación web. De forma lógica, estos diagnósticos se dividen en diagnósticos del servidor web y diagnóstico de aplicaciones.

##### <a name="web-server-diagnostics"></a>Diagnósticos del servidor web
Puede habilitar o deshabilitar los siguientes tipos de registros:

* **Registro de errores detallado** : registra información detallada de errores para códigos de estado HTTP que indican un problema (código de estado 400 o superior). Puede contener información que puede ayudar a determinar por qué el servidor ha devuelto el código de error.
* **Seguimiento de solicitudes con error** : registra información detallada acerca de solicitudes con error, incluido un seguimiento de los componentes de IIS usados para procesar la solicitud y el tiempo dedicado a cada componente. Esto puede resultar útil si está intentando mejorar el rendimiento de las aplicaciones web o de aislar la causa de un error HTTP determinado.
* **Registro del servidor web** : registra todas las transacciones HTTP con el formato de archivo de registro extendido de W3C. Este informe resulta útil para determinar las métricas totales de la aplicación web, como el número de solicitudes tramitadas o cuántas solicitudes proceden de una dirección IP específica.

##### <a name="application-diagnostics"></a>Diagnósticos de aplicaciones
El diagnóstico de aplicaciones le permite capturar información generada por una aplicación web. Las aplicaciones de ASP.NET pueden usar la clase `System.Diagnostics.Trace` para registrar información en el registro de diagnóstico de aplicaciones.

Para obtener instrucciones detalladas sobre cómo configurar su aplicación para el registro, consulte [Habilitación del registro de diagnóstico para aplicaciones web en el Servicio de aplicaciones de Azure](web-sites-enable-diagnostic-log.md).

#### <a name="use-remote-profiling"></a>Uso de la generación remota de perfiles
En el Servicio de aplicaciones, Aplicaciones web, Aplicaciones de API y WebJobs de Azure se pueden generar perfiles de forma remota. Si el proceso se ejecuta más lento de lo esperado o la latencia de las solicitudes HTTP es superior a la normal y el uso de CPU del proceso es también elevado, puede generar un perfil para su proceso de forma remota y obtener las pilas de llamada de muestreo de CPU para analizar la actividad del proceso y las rutas de acceso activas del código.

Para obtener más información, consulte [Compatibilidad con la generación remota de perfiles en el Servicio de aplicaciones de Azure](https://azure.microsoft.com/blog/remote-profiling-support-in-azure-app-service).

#### <a name="use-the-azure-app-service-support-portal"></a>Uso del Portal de soporte técnico del Servicio de aplicaciones de Azure
El servicio Aplicaciones web ofrece la posibilidad de solucionar los problemas relacionados con su aplicación web con solo examinar los registros HTTP, los registros de eventos, los volcados de proceso, etc. Puede tener acceso a toda esta información con nuestro Portal de soporte técnico en **http://&lt;Nombre de la aplicación>.scm.azurewebsites.net/Support**.

El Portal de soporte técnico del Servicio de aplicaciones de Azure le proporciona tres pestañas distintas correspondientes a los tres pasos de un escenario común de solución de problemas:

1. Observación del comportamiento actual
2. Análisis mediante la recopilación de información de diagnóstico y la ejecución de los analizadores integrados
3. Mitigación

Si el problema está sucediendo justo ahora, haga clic en **Analizar** > **Diagnósticos** > **Diagnosticar ahora** para crear una sesión de diagnóstico, que recopilará registros HTTP, registros del visor de eventos, volcados de memoria, registros de errores PHP e informes de procesos PHP.

Después de la recopilación de los datos, también se ejecuta un análisis de ellos y se proporciona un informe HTML.

En caso de que quiera descargar los datos, de forma predeterminada se almacenarían en la carpeta D:\home\data\DaaS.

Para obtener más información sobre el Portal de soporte técnico del Servicio de aplicaciones de Azure, consulte [Nuevas actualizaciones de la extensión de sitios de soporte técnico para Sitios web de Azure](https://azure.microsoft.com/blog/new-updates-to-support-site-extension-for-azure-websites).

#### <a name="use-the-kudu-debug-console"></a>Uso de la consola de depuración Kudu
El servicio Aplicaciones web incluye una consola de depuración que puede usar para depurar, explorar o cargar archivos, e incluye también puntos de conexión JSON para obtener información sobre su entorno. A esto se le denomina *Consola Kudu* o *Panel SCM* para la aplicación web.

Puede tener acceso a este panel en el vínculo **https://&lt;Nombre de la aplicación>.scm.azurewebsites.net/**.

Algunas de las cosas que proporciona Kudu son:

* Configuración del entorno de la aplicación
* transmisión de registro
* Volcado de diagnóstico
* Depuración de la consola en la que puede ejecutar cmdlets de Powershell y comandos básicos de DOS.

Otra característica útil de Kudu es que, en caso de que la aplicación inicie excepciones de primera oportunidad, puede usar Kudu y la herramienta Procdump de SysInternals para crear volcados de memoria. Estos volcados de memoria son instantáneas del proceso y a menudo pueden ayudarle solucionar problemas más complicados de su aplicación web.

Para obtener más información sobre las características disponibles en Kudu, consulte [Herramientas de Azure Websites Team Services que debe conocer](https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/).

<a name="mitigate" />

### <a name="3-mitigate-the-issue"></a>3. Mitigación del problema
#### <a name="scale-the-web-app"></a>Escalado de la aplicación web
En Azure App Service, puede ajustar la escala en la que se ejecuta la aplicación para aumentar el rendimiento y la capacidad de proceso. El escalado vertical de una aplicación web implica dos acciones relacionadas: cambiar el plan del Servicio de aplicaciones por un plan de tarifa más alto y configurar determinados valores después de haber cambiado a ese plan de tarifa más alto.

Para obtener más información sobre el escalado, consulte [Escalado de una aplicación web en el Servicio de aplicaciones de Azure](web-sites-scale.md).

Además, puede elegir ejecutar la aplicación en más de una instancia. Esto no solo le proporciona una mayor capacidad de procesamiento, sino también algo de tolerancia a errores. Si el proceso se interrumpe en una instancia, la otra instancia seguirá atendiendo las solicitudes.

Puede establecer el escalado en Manual o Automático.

#### <a name="use-autoheal"></a>Uso de AutoHeal
AutoHeal recicla el proceso de trabajo para su aplicación en función de la configuración que elija (como cambios de configuración, solicitudes, límites de memoria o el tiempo necesario para ejecutar una solicitud). Casi siempre, el proceso de reciclaje es la forma más rápida de recuperarse de un problema. Aunque siempre puede reiniciar la aplicación web directamente en el Portal de Azure, AutoHeal lo hará automáticamente por usted. Todo lo que debe hacer es agregar algunos desencadenadores en web.config raíz de la aplicación web. Tenga en cuenta que esta configuración funcionaría igual incluso si la aplicación no fuera .Net.

Para obtener más información, consulte [Recuperación automática de Sitios web de Azure](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites/).

#### <a name="restart-the-web-app"></a>Reinicio de la aplicación web
Suele ser la manera más sencilla de recuperarse de problemas que solo tienen lugar una vez. En el [Portal de Azure](https://portal.azure.com/), en la hoja de la aplicación web, tiene las opciones para detener o reiniciar la aplicación.

 ![reiniciar las aplicaciones web para resolver los problemas de rendimiento](./media/app-service-web-troubleshoot-performance-degradation/2-restart.png)

También puede administrar la aplicación web con Azure Powershell. Para obtener más información, vea [Uso de Azure PowerShell con el Administrador de recursos de Azure](../powershell-azure-resource-manager.md).

