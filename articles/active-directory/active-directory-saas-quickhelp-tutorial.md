---
title: "Tutorial: Integración de Azure Active Directory con QuickHelp | Microsoft Docs"
description: "Aprenda a configurar el inicio de sesión único entre Azure Active Directory y QuickHelp."
services: active-directory
documentationcenter: 
author: jeevansd
manager: femila
editor: 
ms.assetid: 655c9ad3-2076-4e2c-8e47-9ed3bf04be56
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/22/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 0837cb33bf438fb7fd9665d21d411f0170cdd393
ms.openlocfilehash: f84a5a9e40f6dd1c98fd73ba38a1b5cae9d9cba4
ms.lasthandoff: 02/23/2017


---
# <a name="tutorial-azure-active-directory-integration-with-quickhelp"></a>Tutorial: Integración de Azure Active Directory con QuickHelp
El objetivo de este tutorial es mostrar cómo integrar QuickHelp con Azure Active Directory (Azure AD).  
Integrar QuickHelp con Azure AD proporciona las siguientes ventajas: 

* Puede controlar en Azure AD quién tiene acceso a QuickHelp. 
* Puede permitir que los usuarios inicien sesión automáticamente en QuickHelp (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: el Portal de Azure clásico.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Requisitos previos
Para configurar la integración de Azure AD con QuickHelp, necesita los siguientes elementos:

* Una suscripción de Azure AD
* Una suscripción habilitada para el inicio de sesión único en QuickHelp

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción.
> 
> 

Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

* No debe usar el entorno de producción, a menos que sea necesario.
* Si no dispone de un entorno de prueba de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/). 

## <a name="scenario-description"></a>Descripción del escenario
El objetivo de este tutorial es permitirle probar el inicio de sesión único de Azure AD en un entorno de prueba.  
La situación descrita en este tutorial consta de dos bloques de creación principales:

1. Agregar QuickHelp desde la galería 
2. Configuración y comprobación del inicio de sesión único de Azure AD

## <a name="adding-quickhelp-from-the-gallery"></a>Agregar QuickHelp desde la galería
Para configurar la integración de QuickHelp en Azure AD, deberá agregar QuickHelp desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar QuickHelp desde la galería, realice los pasos siguientes:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**. 
   
    ![Active Directory][1]
2. En la lista **Directory** , seleccione el directorio cuya integración desee habilitar.
3. Para abrir la vista de aplicaciones, haga clic en **Applications** , en el menú superior de la vista de directorios.
   
    ![Applications][2]
4. Haga clic en **Agregar** en la parte inferior de la página.
   
    ![Aplicaciones][3]
5. En el cuadro de diálogo **¿Qué desea hacer?**, haga clic en **Agregar una aplicación de la galería**.
   
    ![Aplicaciones][4]
6. En el cuadro de búsqueda, escriba **QuickHelp**.
   
    ![Aplicaciones][5]
7. En el panel de resultados, seleccione **QuickHelp** y, después, haga clic en **Completar** para agregar la aplicación.
   
    ![Aplicaciones][500]

## <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuración y comprobación del inicio de sesión único de Azure AD
El objetivo de esta sección es mostrar cómo configurar y probar el inicio de sesión único de Azure AD con QuickHelp con un usuario de prueba llamado "Britta Simon".

Para configurar y probar el inicio de sesión único de Azure AD con QuickHelp, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-single-sign-on)** : para permitir a los usuarios usar esta característica.
2. **[Creación de un usuario de prueba de Azure AD](#creating-an-azure-ad-test-user)** : para probar el inicio de sesión único de Azure AD con Britta Simon.
3. **[Creación de un usuario de prueba de QuickHelp](#creating-a-quickhelp-test-user)** : para tener un homólogo de Britta Simon en QuickHelp que esté vinculado a la representación de ella en Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assigning-the-azure-ad-test-user)** : para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Prueba del inicio de sesión único](#testing-single-sign-on)** : para comprobar si funciona la configuración.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD
El objetivo de esta sección es habilitar el inicio de sesión único de Azure AD en el Portal de Azure clásico y configurar el inicio de sesión único en la aplicación QuickHelp.

**Para configurar el inicio de sesión único de Azure AD con QuickHelp, realice los pasos siguientes:**

1. En el Portal de Azure clásico, en la página de integración de aplicaciones de **QuickHelp**, haga clic en **Configurar inicio de sesión único** para abrir el diálogo **Configurar inicio de sesión único**.
   
    ![Configurar inicio de sesión único][6] 
2. En la página **¿Cómo quiere que los usuarios inicien sesión en QuickHelp?**, seleccione **Inicio de sesión único de Azure AD** y, después, haga clic en **Siguiente**.
   
    ![Inicio de sesión único de Azure AD ][7] 
3. En la página de diálogo **Configurar las opciones de la aplicación** , realice los pasos siguientes:
   
    ![Configurar las opciones de la aplicación][8] 
   
    a. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL que los usuarios usan para iniciar sesión en el sitio de QuickHelp (por ejemplo: *https://quickhelp.com/bsiazure/*).
   
    > [!NOTE]
    > Póngase en contacto con el equipo de soporte de QuickHelp si no conoce el valor de la URL de inicio de sesión.
    > 
    > 
   
    b. Haga clic en **Siguiente**.
4. En la página **Configurar inicio de sesión único en QuickHelp**, siga este procedimiento: haga clic en **Descargar metadatos** y, luego, guarde el archivo de metadatos localmente en el equipo.
   
    ![Qué es Azure AD Connect][9] 
5. Inicie sesión en el sitio de la empresa de QuickHelp como administrador.
6. En el menú de la parte superior, haga clic en **Administrador**.
   
    ![Configurar inicio de sesión único][21]
7. En el menú **Administrador de QuickHelp**, haga clic en **Configuración**.
   
    ![Configurar inicio de sesión único][22]
8. Haga clic en **Configuración de autenticación**.
9. En la página **Configuración de autenticación** , realice los siguientes pasos.
   
    ![Configurar inicio de sesión único][23]
   
    a. En **Tipo de SSO**, seleccione **WSFederation**.
   
    b. Para cargar el archivo de metadatos de Azure descargado, haga clic en **Examinar**, vaya hasta el archivo y, luego, haga clic en **Cargar metadatos**.
   
    c. En el cuadro de texto **Correo electrónico**, escriba **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress**.
   
    d. En el cuadro de texto **Nombre**, escriba **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname**.
   
    e. En el cuadro de texto **Apellidos**, escriba **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname**.
   
    f. En la **Barra de acciones**, haga clic en **Guardar**.
10. En el Portal de Azure clásico, seleccione la confirmación de la configuración de inicio de sesión único y haga clic en **Siguiente**. 
    
     ![Qué es Azure AD Connect][10]
11. En la página **Confirmación del inicio de sesión único**, haga clic en **Completar**.  
    
     ![Qué es Azure AD Connect][11]

### <a name="creating-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD
El objetivo de esta sección es crear un usuario de prueba en el Portal de Azure clásico llamado Britta Simon.  
En la lista Usuarios, seleccione **Britta Simon**.

![Creación de un usuario de Azure AD][20]

**Siga estos pasos para crear un usuario de prueba en Azure AD:**

1. En el panel de navegación izquierdo del **Portal de Azure clásico**, haga clic en **Active Directory**.
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_02.png) 
2. En la lista **Directory** , seleccione el directorio cuya integración desee habilitar.
3. Para mostrar la lista de usuarios, en el menú de la parte superior, haga clic en **Usuarios**.
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_03.png) 
4. Para abrir el cuadro de diálogo **Agregar usuario**, en la barra de herramientas de la parte inferior, haga clic en **Agregar usuario**. 
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_04.png) 
5. En la página de diálogo **Proporcione información sobre este usuario** , realice los pasos siguientes: 
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_05.png) 
   
    a. En Tipo de usuario, seleccione Nuevo usuario de la organización.
   
    b. En el cuadro de texto **Nombre de usuario**, escriba**BrittaSimon**.
   
    c. Haga clic en **Siguiente**.
6. En la página de diálogo **Perfil de usuario** , realice los pasos siguientes: 
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_06.png) 
   
    a. En el cuadro de texto **Nombre**, escriba **Britta**.  
   
    b. En el cuadro de texto **Apellidos**, escriba **Simon**.
   
    c. En el cuadro de texto **Nombre para mostrar**, escriba **Britta Simon**.
   
    d. En la lista **Rol**, seleccione **Usuario**.
    e. Haga clic en **Siguiente**.
7. En el cuadro de diálogo **Obtener contraseña temporal**, haga clic en **Crear**.

    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_07.png) 

1. En la página de diálogo **Obtener contraseña temporal** , realice los pasos siguientes:
   
    ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-quickhelp-tutorial/create_aaduser_08.png) 
   
    a. Anote el valor del campo **Nueva contraseña**.
   
    b. Haga clic en **Completo**.   

### <a name="creating-a-quickhelp-test-user"></a>Creación de un usuario de prueba de QuickHelp
El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en QuickHelp.
Para que el inicio de sesión único funcione, Azure AD debe saber cuál es el usuario homólogo de QuickHelp para un usuario de Azure AD. Es decir, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de QuickHelp.

QuickHelp admite el aprovisionamiento Just-In-Time. Esto significa que, si fuera necesario, se creará automáticamente una cuenta de usuario en QuickHelp y la cuenta se vinculará a la cuenta de Azure AD.

No hay ningún elemento de acción para usted en esta sección.

### <a name="assigning-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD
El objetivo de esta sección es permitir que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a QuickHelp.

![Asignar usuario][200] 

**Para asignar a Britta Simon a QuickHelp, realice los pasos siguientes:**

1. En el Portal de Azure clásico, para abrir la vista de aplicaciones, en la vista del directorio, haga clic en **Aplicaciones** , que se encuentra en el menú superior.
   
    ![Asignar usuario][201] 
2. En la lista de aplicaciones, seleccione **QuickHelp**.
   
    ![Asignar usuario][202] 
3. En el menú de la parte superior, haga clic en **Usuarios**.
   
    ![Asignar usuario][203] 
4. En la lista Usuarios, seleccione **Britta Simon**.
5. En la barra de herramientas de la parte inferior, haga clic en **Asignar**.
   
    ![Asignar usuario][205]

### <a name="testing-single-sign-on"></a>Prueba del inicio de sesión único 
El objetivo de esta sección es probar la configuración del inicio de sesión único de Azure AD mediante el panel de acceso.  
Al hacer clic en el icono de QuickHelp en el panel de acceso, debería iniciar sesión automáticamente en la aplicación.

## <a name="additional-resources"></a>Recursos adicionales
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](active-directory-saas-tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_04.png
[5]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_01.png
[500]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_14.png


[6]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_05.png
[7]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_02.png
[8]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_03.png
[9]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_04.png
[10]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_100.png
[21]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_05.png
[22]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_06.png
[23]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_07.png
[24]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_08.png
[25]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_09.png
[26]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_10.png
[27]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_11.png
[28]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_12.png

[200]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_quickhelp_13.png
[203]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-quickhelp-tutorial/tutorial_general_205.png


[400]: ./media/active-directory-saas-QuickHelp-tutorial/tutorial_QuickHelp_400.png
[401]: ./media/active-directory-saas-QuickHelp-tutorial/tutorial_QuickHelp_401.png
[402]: ./media/active-directory-saas-QuickHelp-tutorial/tutorial_QuickHelp_402.png





