---
title: 教學課程 - 使用 Azure Active Directory B2C 授與從 Web 應用程式存取 ASP.NET Web API 的權限 | Microsoft Docs
description: 關於如何使用 Active Directory B2C 來保護 ASP.NET Web API，以及如何從 ASP.NET Web 應用程式來呼叫它的教學課程。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.author: davidmu
ms.date: 01/23/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.component: B2C
ms.openlocfilehash: 2b70ed174331b88f9afc9aa30d14a585986496a5
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604335"
---
# <a name="tutorial-grant-access-to-an-aspnet-web-api-from-a-web-app-using-azure-active-directory-b2c"></a>教學課程 - 使用 Azure Active Directory B2C 授與從 Web 應用程式存取 ASP.NET Web API 的權限

此教學課程將說明如何從 ASP.NET Web 應用程式呼叫 Azure Active Directory (Azure AD) B2C 所保護的 Web API 資源。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 在您的 Azure AD B2C 租用戶中註冊 Web API
> * 定義及設定 Web API 的範圍
> * 為應用程式授與 Web API 的權限
> * 更新範例程式碼以使用 Azure AD B2C 來保護 Web API

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先決條件

* 完成[在 ASP.NET Web 應用程式中使用 Azure Active Directory B2C 進行使用者驗證的教學課程](active-directory-b2c-tutorials-web-app.md)。
* 安裝包含 **ASP.NET 和 Web 開發**工作負載的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。

## <a name="register-web-api"></a>註冊 Web API

Web API 資源必須先在您的租用戶中註冊，才能接受及回應受到[用戶端應用程式](../active-directory/develop/developer-glossary.md#client-application)保護而提供來自 Azure Active Directory 之[存取權杖](../active-directory/develop/developer-glossary.md#access-token)的[資源要求](../active-directory/develop/developer-glossary.md#resource-server)。 註冊會在您的租用戶中建立[應用程式與服務主體物件](../active-directory/develop/developer-glossary.md#application-object)。 

以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

[!INCLUDE [active-directory-b2c-switch-b2c-tenant](../../includes/active-directory-b2c-switch-b2c-tenant.md)]

1. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。 您現在應使用在前一個教學課程中建立的租用戶。

2. 選取 [應用程式]，然後選取 [新增]。

    若要在您的租用戶中註冊範例 Web API，請使用下列設定。
    
    ![新增 API](./media/active-directory-b2c-tutorials-web-api/web-api-registration.png)
    
    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | 我的範例 Web API | 輸入向開發人員描述您的 Web API 的 [名稱]。 |
    | **包含 Web 應用程式 / Web API** | 是 | 針對 Web API 選取 [是]。 |
    | **允許隱含流程** | 是 | 請選取 [是]，因為 API 使用 [OpenID Connect 登入](active-directory-b2c-reference-oidc.md)。 |
    | **回覆 URL** | `https://localhost:44332` | 回覆 URL 是 Azure AD B2C 傳回您 API 要求之任何權杖的所在端點。 在此教學課程中，範例 Web API 會在本機執行 (localhost)，並接聽連接埠 44332。 |
    | **應用程式識別碼 URI** | myAPISample | URI 可唯一識別租用戶中的 API。 這可讓您為每個租用戶註冊多個 API。 [範圍](../active-directory/develop/developer-glossary.md#scopes)可控管對受保護 API 資源的存取，並就個別的應用程式識別碼 URI 進行定義。 |
    | **原生用戶端** | 否 | 這是 Web API，而不是原生用戶端，因此請選取 [否]。 |
    
3. 按一下 [建立]  以註冊您的 API。

已註冊的 API 會顯示在 Azure AD B2C 租用戶的應用程式清單中。 從清單中選取 Web API。 Web API 的 [屬性] 窗格隨即顯示。

![Web API 屬性](./media/active-directory-b2c-tutorials-web-api/b2c-web-api-properties.png)

請記下 [應用程式用戶端識別碼]。 此識別碼可唯一識別 API，後續在此教學課程中設定 API 時將會用到。

將您的 Web API 註冊至 Azure AD B2C，可定義信任關係。 由於 API 已註冊至 B2C，因此 API 此時得以信任其接收自其他應用程式的 B2C 存取權杖。

## <a name="define-and-configure-scopes"></a>定義及設定範圍

[範圍](../active-directory/develop/developer-glossary.md#scopes)可用來控管對受保護資源的存取。 Web API 可使用範圍來實作以範圍為基礎的存取控制。 例如，Web API 的使用者可以同時具有讀取和寫入權限，Web API 的使用者也可能只具有讀取權限。 在此教學課程中，您會使用範圍來定義 Web API 的讀取和寫入權限。

### <a name="define-scopes-for-the-web-api"></a>定義 Web API 的範圍

已註冊的 API 會顯示在 Azure AD B2C 租用戶的應用程式清單中。 從清單中選取 Web API。 Web API 的 [屬性] 窗格隨即顯示。

按一下 [發佈的範圍 (預覽)]。

若要設定 API 的範圍，請新增下列項目。 

![Web API 中定義的範圍](media/active-directory-b2c-tutorials-web-api/scopes-defined-in-web-api.png)

| 設定      | 建議的值  | 說明                                        |
| ------------ | ------- | -------------------------------------------------- |
| **範圍** | Hello.Read | hello 的讀取權限 |
| **範圍** | Hello.Write | hello 的寫入權限 |

按一下 [檔案] 。

發佈的範圍可以用來為用戶端應用程式授與對 Web API 的權限。

### <a name="grant-app-permissions-to-web-api"></a>為應用程式授與 Web API 的權限

若要從應用程式呼叫受保護的 Web API，您必須為應用程式授與對 API 的權限。 在此教學課程中，使用[在 ASP.NET Web 應用程式中使用 Azure Active Directory B2C 進行使用者驗證](active-directory-b2c-tutorials-web-app.md)教學課程中建立的 Web 應用程式。 

1. 在 Azure 入口網站中，從服務清單中選取 [Azure AD B2C]，然後按一下 [應用程式] 以檢視已註冊的應用程式清單。

2. 從應用程式清單中選取 [我的範例 Web 應用程式]、按一下 [API 存取 (預覽)]，然後按一下 [新增]。

3. 在 [選取 API] 下拉式清單中，選取您已註冊的 Web API **我的範例 Web API**。

4. 在 [選取範圍] 下拉式清單中，選取您在 Web API 註冊中定義的範圍。

    ![選取應用程式的範圍](media/active-directory-b2c-tutorials-web-api/selecting-scopes-for-app.png)

5. 按一下 [確定]。

您的 [我的範例 Web 應用程式] 會進行註冊以呼叫受保護的 [我的範例 Web API]。 使用者會透過 Azure AD B2C 進行[驗證](../active-directory/develop/developer-glossary.md#authentication)以使用 Web 應用程式。 Web 應用程式會取得 Azure AD B2C 的[授權授與](../active-directory/develop/developer-glossary.md#authorization-grant)，以存取受保護的 Web API。

## <a name="update-code"></a>更新程式碼

現在，Web API 已註冊，且您已定義範圍，接下來您必須設定 Web API 程式碼以使用您的 Azure AD B2C 租用戶。 在此教學課程中，您會設定範例 Web API。 

您在必要的教學課程中下載的專案包含範例 Web API：[在 ASP.NET Web 應用程式中使用 Azure Active Directory B2C 進行使用者驗證的教學課程](active-directory-b2c-tutorials-web-app.md)。 如果您未完成必要的教學課程，請先完成再繼續。

範例方案中有兩個專案：

**Web 應用程式範例應用程式 (TaskWebApp)：** 用來建立和編輯工作清單的 Web 應用程式。 此 Web 應用程式會使用**註冊或登入**原則，以電子郵件地址來註冊或登入使用者。

**Web API 範例應用程式 (TaskService)：** 支援建立、讀取、更新和刪除工作清單功能的 Web API。 此 Web API 會受到 Azure AD B2C 的保護，且由 Web 應用程式呼叫。

範例 Web 應用程式和 Web API 會在每個專案的 Web.config 檔案中將組態值定義為應用程式設定。

在 Visual Studio 中開啟 **B2C-WebAPI-DotNet** 方案。

### <a name="configure-the-web-app"></a>設定 Web 應用程式

1. 開啟 **TaskWebApp** 專案中的 **Web.config**。

2. 若要在本機執行 API，請使用 **api:TaskServiceUrl** 的 localhost 設定。 變更 Web.config，如下所示： 

    ```C#
    <add key="api:TaskServiceUrl" value="https://localhost:44332/"/>
    ```

3. 設定 API 的 URI。 這是 Web 應用程式用來進行 API 要求的 URI。 此外，請設定要求的權限。

    ```C#
    <add key="api:ApiIdentifier" value="https://<Your tenant name>.onmicrosoft.com/myAPISample/" />
    <add key="api:ReadScope" value="Hello.Read" />
    <add key="api:WriteScope" value="Hello.Write" />
    ```

### <a name="configure-the-web-api"></a>設定 Web API

1. 開啟 **TaskService** 專案中的 **Web.config**。

2. 設定 API 以使用您的租用戶。

    ```C#
    <add key="ida:Tenant" value="<Your tenant name>.onmicrosoft.com" />
    ```

3. 將用戶端識別碼設為您的 API 已註冊的應用程式識別碼。

    ```C#
    <add key="ida:ClientId" value="<The Application ID for your web API obtained from the Azure portal>"/>
    ```

4. 使用您在建立註冊和登入原則時產生的名稱來更新原則設定。

    ```C#
    <add key="ida:SignUpSignInPolicyId" value="B2C_1_SiUpIn" />
    ```

5. 設定比對您在入口網站中建立的項目時所使用的範圍設定。

    ```C#
    <add key="api:ReadScope" value="Hello.Read" />
    <add key="api:WriteScope" value="Hello.Write" />
    ```

## <a name="run-the-sample"></a>執行範例

您必須同時執行 **TaskWebApp** 和 **TaskService** 專案。 

1. 在方案總管中，以滑鼠右鍵按一下方案，然後選取 [設定啟始專案]。 
2. 選取 [多個啟始專案] 選項按鈕。
3. 將兩個專案的 [動作] 變更為 [啟動]。
4. 按一下 [確定] 以儲存組態。
5. 按 **F5** 鍵執行兩個應用程式。 這兩個應用程式會分別在其本身的瀏覽器索引標籤中開啟。`https://localhost:44316/` 為 Web 應用程式。
    `https://localhost:44332/` 為 Web API。

6. 在 Web 應用程式中，按一下功能表橫幅中的註冊/登入連結，以註冊 Web 應用程式。 請使用您在 [Web 應用程式教學課程](active-directory-b2c-tutorials-web-app.md)中建立的帳戶。 
7. 登入之後，請按一下 [待辦事項清單] 連結，並建立待辦事項清單項目。

在您建立待辦事項清單項目時，Web 應用程式會對 Web API 提出要求，以產生待辦事項清單項目。 您受保護的 Web 應用程式會呼叫您的 Azure AD B2C 租用戶中受保護的 Web API。

## <a name="clean-up-resources"></a>清除資源

如果您想要嘗試其他 Azure AD B2C 教學課程，您可以使用 Azure AD B2C 租用戶。 不再需要時，您可以[刪除您的 Azure AD B2C 租用戶](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant)。

## <a name="next-steps"></a>後續步驟

此文章逐步說明如何在 Azure AD B2C 中註冊和定義範圍，以保護 ASP.NET Web API。 若要深入了解開發此案例的詳細資料 (包括程式碼逐步解說)，請繼續執行下一個教學課程。

> [!div class="nextstepaction"]
> [建立支援 Azure Active Directory B2C 註冊、登入、設定檔編輯及密碼重設的 ASP.NET Web 應用程式](active-directory-b2c-devquickstarts-web-dotnet-susi.md)