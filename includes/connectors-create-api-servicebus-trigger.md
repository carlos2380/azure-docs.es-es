Aquí mostraremos cómo usar el desencadenador **Service Bus - When a message is received in a queue** (Bus de servicio: cuando se recibe un mensaje en una cola) para iniciar un flujo de trabajo de aplicación lógica cuando se reciba un nuevo elemento en una cola del Bus de servicio.  

> [!NOTE]
> Si aún no ha creado una conexión a Bus de servicio, se le pedirá que inicie sesión con la cadena de conexión del Bus de servicio.  
> 
> 

1. En el cuadro de búsqueda del diseñador de aplicaciones lógicas, escriba **Bus de servicio**. Después, seleccione el desencadenador **Service Bus - When a message is received in a queue** (Bus de servicio: cuando se recibe un mensaje en una cola).  
   ![Imagen 1 de desencadenador del Bus de servicio](./media/connectors-create-api-servicebus/trigger-1.png)   
2. Se muestra el cuadro de diálogo **When a message is received in a queue** (Cuando se recibe un mensaje en una cola).  
   ![Imagen 2 de desencadenador del Bus de servicio](./media/connectors-create-api-servicebus/trigger-2.png)   
3. Escriba el nombre de la cola del Bus de servicio que quiere que supervise el desencadenador.   
   ![Imagen 3 de desencadenador del Bus de servicio](./media/connectors-create-api-servicebus/trigger-3.png)   

En este punto, la aplicación lógica está configurado con un desencadenador, que activará otros desencadenadores y acciones del flujo de trabajo cuando se reciba un nuevo elemento en la cola que ha seleccionado.    

