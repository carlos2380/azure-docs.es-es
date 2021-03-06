

## <a name="memory-preserving-updates"></a>Actualizaciones de conservación de memoria
En el caso de una clase de actualizaciones de Microsoft Azure, los clientes no ven que sus máquinas virtuales en ejecución resulten afectadas en absoluto. Muchas de estas actualizaciones son componentes o servicios que se pueden actualizar sin interferir con la instancia en ejecución. Algunas de estas actualizaciones son actualizaciones de la infraestructura de la plataforma en el sistema operativo host, que se pueden aplicar sin necesidad de un reinicio completo de las máquinas virtuales.

Estas actualizaciones se realizan con la tecnología que permite la migración en vivo in situ, también llamada actualización de "conservación de memoria". Al actualizar, la máquina virtual se pone en un estado de "pausa" y conserva la memoria RAM, mientras que el sistema operativo host subyacente recibe las actualizaciones y revisiones necesarias. La máquina virtual se reanudará en menos de 30 segundos después de haberse puesto en pausa. Una vez reanudada, se sincronizará automáticamente el reloj de la máquina virtual.

No todas las actualizaciones pueden implementarse con este mecanismo, pero gracias a su breve período de pausa, implementar las actualizaciones de este modo reduce en gran medida el impacto en las máquinas virtuales.

Se aplican actualizaciones de instancias múltiples (parar máquinas virtuales en un conjunto de disponibilidad) a un dominio de actualización a la vez.  

## <a name="virtual-machine-configurations"></a>Configuraciones de máquinas virtuales
Hay dos tipos de configuraciones de máquinas virtuales: instancias múltiples y una sola instancia. En una configuración de instancias múltiples, las máquinas virtuales similares se colocan en un conjunto de disponibilidad.

La configuración de instancias múltiples proporciona redundancia entre equipos físicos, potencia y red, y se recomienda para garantizar la disponibilidad de la aplicación. Todas las máquinas virtuales del conjunto de disponibilidad deben prestar el mismo servicio a la aplicación.

Para más información acerca de cómo configurar máquinas virtuales para conseguir alta disponibilidad, consulte [Administración de la disponibilidad de las máquinas virtuales Windows ](../articles/virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) o [Administración de la disponibilidad de las máquinas virtuales Linux](../articles/virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Por el contrario, se usa una configuración de una sola instancia se usa para máquinas virtuales independientes no colocadas en un conjunto de disponibilidad. Estas máquinas virtuales no cumplen los requisitos para el Acuerdo de Nivel de Servicio (SLA), que requiere que se implementen dos o más máquinas virtuales en el mismo conjunto de disponibilidad.

Para más información acerca los acuerdos de Nivel de Servicio, consulte las secciones "Servicios en la nube y Máquinas virtuales" de [Contratos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/).

## <a name="multi-instance-configuration-updates"></a>Actualizaciones de una configuración de instancias múltiples
Durante el mantenimiento planeado, la plataforma Azure actualizará primero el conjunto de máquinas virtuales que se hospedan en una configuración de instancias múltiples. La actualización hace que estas máquinas virtuales se reinicien con unos 15 minutos de inactividad.

En las actualizaciones de la configuración de varias instancias se asume que cada máquina virtual realiza una función similar a los demás del conjunto de disponibilidad. En esta configuración, las máquinas virtuales se actualizan de forma tal que la disponibilidad se pierde durante el proceso.

La plataforma Azure subyacente asigna a cada máquina virtual de un conjunto de disponibilidad un dominio de actualización y un dominio de error. Cada dominio de actualización es un grupo de máquinas virtuales que se reiniciará en la misma ventana de tiempo. Cada dominio de error es un grupo de máquinas virtuales que comparten una fuente de alimentación común y un conmutador de red.


Para obtener más información sobre dominios de actualización y dominios de error, vea [Configuración de varias máquinas virtuales en un conjunto de disponibilidad para redundancia](../articles/virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).

Para mantener la disponibilidad a través de una actualización, Azure realiza el mantenimiento por dominio de actualización y actualiza los dominios de uno en uno. El mantenimiento de un dominio de actualización consta de las siguientes operaciones: apagar todas las máquinas virtuales del dominio, aplicar la actualización a las máquinas host y, después, reiniciar las máquinas virtuales. Cuando se completa el mantenimiento del dominio, Azure repite el proceso con el siguiente dominio de actualización y continúa hasta que todos los dominios estén actualizados.

El orden en el que los dominios de actualización se reinician no puede llevarse a cabo secuencialmente durante un mantenimiento planeado, sino que solamente se reiniciará un dominio de actualización simultáneamente. Actualmente, Azure ofrece una notificación de mantenimiento planeado para máquinas virtuales con una semana de antelación en la configuración de instancias múltiples.

Después de restaurar una máquina virtual, aquí se muestra un ejemplo de lo que puede mostrar el Visor de eventos de Windows:

<!--Image reference-->
![][image2]


Use el visor para indicar las máquinas virtuales que están configuradas en una configuración de instancias múltiples mediante Azure Portal, Azure PowerShell o la CLI de Azure. Por ejemplo, mediante Azure Portal, se puede agregar el _conjunto de disponibilidad_ al cuadro de diálogo de examen de **máquinas virtuales (clásico)**. Las máquinas virtuales que indican el mismo conjunto de disponibilidad forman parte de una configuración de instancias múltiples. En el ejemplo siguiente, la configuración de instancias múltiples consta de las máquinas virtuales SQLContoso01 y SQLContoso02.

<!--Image reference-->
  ![Vista de máquinas virtuales (clásico) desde Azure Portal][image4]

## <a name="single-instance-configuration-updates"></a>Actualizaciones de una configuración de una sola instancia
Una vez que se completa la configuración de las instancias múltiples, Azure realiza las actualizaciones de la configuración de las instancias múltiples. Estas actualizaciones también provocan que se reinicien las máquinas virtuales que no se ejecutan en conjuntos de disponibilidad.

> [!NOTE]
> Si en un conjunto de disponibilidad se ejecuta una sola una instancia de máquina virtual, la plataforma Azure la trata como una actualización de la configuración de instancias múltiples.
>

El mantenimiento de una configuración de instancia única consta de las siguientes operaciones: cerrar todas las máquinas virtuales que se ejecutan en un equipo host, actualizar el equipo host y, después, reiniciar las máquinas virtuales. El mantenimiento requiere aproximadamente 15 minutos de inactividad. El evento de mantenimiento planeado se ejecuta en todas las máquinas virtuales de una región en una sola ventana de mantenimiento.


Los eventos de mantenimiento planeados afectan a la disponibilidad de la aplicación en las configuraciones de instancia única. Azure ofrece una notificación del mantenimiento planeado de máquinas virtuales con una semana de antelación en las configuraciones de instancia única.

## <a name="email-notification"></a>Notificación por correo electrónico
Solo en el caso de las configuraciones de máquinas virtuales de instancias únicas o de instancias múltiples, Azure envía una alerta por correo electrónico para avisarle del próximo mantenimiento planeado (con una semana de antelación). Dicho correo electrónico se envía a las cuentas de correo electrónico del administrador y coadministrador de la suscripción. A continuación se muestra un ejemplo de este tipo de correo electrónico:

<!--Image reference-->
![][image1]

## <a name="region-pairs"></a>Pares de región

Al ejecutar el mantenimiento, Azure solo actualiza las instancias de máquina virtual en una sola región de su par. Por ejemplo, al actualizar las máquinas virtuales de la zona centro-norte de EE. UU., Azure no actualizará las máquinas virtuales de centro-sur de EE. UU. al mismo tiempo. Se programarán a una hora independiente, lo que permitirá la conmutación por error o el equilibrio de carga entre regiones. Sin embargo, otras regiones como Europa del Norte pueden estar en mantenimiento al mismo tiempo que el Este de EE. UU.

Consulte en la tabla siguiente los pares de regiones actuales:

| Región 1 | Región 2 |
|:--- | ---:|
| Este de EE. UU. |Oeste de EE. UU. |
| Este de EE. UU. 2 |Central EE. UU.: |
| Centro-Norte de EE. UU |Centro-Sur de EE. UU |
| Centro occidental de EE.UU. |Oeste de EE. UU. 2 |
| Este de Canadá |Centro de Canadá |
| Sur de Brasil |Centro-Sur de EE. UU |
| Gobierno de EE. UU. - Iowa |Gobierno de EE. UU. - Virginia |
| Departamento de Defensa de EE. UU. Este |Departamento de Defensa de EE. UU. Centro |
| Europa del Norte |Europa occidental |
| Oeste de Reino Unido |Sur del Reino Unido |
| Centro de Alemania |Noreste de Alemania |
| Sudeste de Asia |Asia oriental |
| Sudeste de Australia |Australia Oriental |
| India central |Sur de India |
| India occidental |Sur de India |
| Este de Japón |Oeste de Japón |
| Corea Central |Corea del Sur |
| Este de China |Norte de China |


<!--Anchors-->
[image1]: ./media/virtual-machines-common-planned-maintenance/vmplanned1.png
[image2]: ./media/virtual-machines-common-planned-maintenance/EventViewerPostReboot.png
[image3]: ./media/virtual-machines-planned-maintenance/RegionPairs.PNG
[image4]: ./media/virtual-machines-common-planned-maintenance/availabilitysetexample.png


<!--Link references-->
[Virtual Machines Manage Availability]: ../articles/virtual-machines/virtual-machines-windows-hero-tutorial.md

[Understand planned versus unplanned maintenance]: ../articles/virtual-machines/windows/manage-availability.md#Understand-planned-versus-unplanned-maintenance/
