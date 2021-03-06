## <a name="prepare-to-authenticate-azure-resource-manager-requests"></a>Prepararse para autenticar solicitudes de Azure Resource Manager
Debe autenticar todas las operaciones que se realizan en los recursos mediante [Azure Resource Manager][lnk-authenticate-arm] con Azure Active Directory (AD). La manera más sencilla de configurar esto es usar PowerShell o CLI de Azure.

Debe instalar [Azure PowerShell 1.0][lnk-powershell-install] o una versión posterior antes de continuar.

En los pasos siguientes se muestra cómo configurar la autenticación de contraseña para una aplicación de AD mediante PowerShell. Puede ejecutar estos comandos en una sesión de PowerShell estándar.

1. Inicie sesión en su suscripción de Azure con el siguiente comando:
   
    ```
    Login-AzureRmAccount
    ```
2. Anote los valores de **TenantId** y **SubscriptionId**. Las necesitará más adelante.
3. Cree una nueva aplicación de Azure Active Directory con el siguiente comando, reemplazando los marcadores de posición:
   
   * **{Nombre para mostrar}**: nombre para mostrar de la aplicación, por ejemplo, **MySampleApp**.
   * **{Dirección URL de la página principal}**: la dirección URL de la página principal de la aplicación, por ejemplo, **http://mysampleapp/home**. Esta dirección URL no tiene que señalar a una aplicación real.
   * **{Identificador de aplicación}**: identificador único, por ejemplo, **http://mysampleapp**. Esta dirección URL no tiene que señalar a una aplicación real.
   * **{Contraseña}:** una contraseña que se usará para autenticar con la aplicación.
     
     ```
     New-AzureRmADApplication -DisplayName {Display name} -HomePage {Home page URL} -IdentifierUris {Application identifier} -Password {Password}
     ```
4. Anote el **ApplicationId** de la aplicación que ha creado. Lo necesitará más adelante.
5. Cree una nueva entidad de servicio con el comando siguiente, reemplazando **{MyApplicationId}** por el valor de **ApplicationId** del paso anterior:
   
    ```
    New-AzureRmADServicePrincipal -ApplicationId {MyApplicationId}
    ```
6. Configure una asignación de rol con el comando siguiente, reemplazando **{MyApplicationId}** por su valor de **ApplicationId**.
   
    ```
    New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName {MyApplicationId}
    ```

Ahora ha terminado de crear la aplicación de Azure AD que le permitirá autenticar desde su aplicación personalizada de C#. En este tutorial necesitará los recursos siguientes:

* TenantId
* SubscriptionId
* ApplicationId
* Password

[lnk-authenticate-arm]: https://msdn.microsoft.com/library/azure/dn790557.aspx
[lnk-powershell-install]: /powershell/azureps-cmdlets-docs
