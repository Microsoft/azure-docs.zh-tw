---
title: 教學課程 - 讓單頁應用程式能夠使用 Azure Active Directory B2C 向帳戶進行驗證 | Microsoft Docs
description: 關於如何使用 Azure Active Directory B2C 為單頁應用程式 (JavaScript) 提供使用者登入的教學課程。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.author: davidmu
ms.date: 3/02/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.component: B2C
ms.openlocfilehash: 47589b8cb8aa6e8d1cacaa028948242431f02c44
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604785"
---
# <a name="tutorial-enable-single-page-app-authentication-with-accounts-using-azure-active-directory-b2c"></a>教學課程 - 讓單頁應用程式能夠使用 Azure Active Directory B2C 向帳戶進行驗證

此教學課程將說明如何使用 Azure Active Directory (Azure AD) B2C 在單頁應用程式 (SPA) 中登入和註冊使用者。 Azure AD B2C 可讓您的應用程式使用開放式標準通訊協定向社交帳戶、企業帳戶和 Azure Active Directory 帳戶進行驗證。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 在您的 Azure AD B2C 目錄中，註冊範例單頁應用程式。
> * 建立使用者註冊、登入、編輯設定檔和密碼重設的原則。
> * 將範例應用程式設定為使用您的 Azure AD B2C 目錄。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先決條件

* 建立您自己的 [Azure AD B2C 目錄](active-directory-b2c-get-started.md)
* 安裝包含 **ASP.NET 和 Web 開發**工作負載的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。
* [.NET Core 2.0.0 SDK](https://www.microsoft.com/net/core) 或更新版本
* 安裝 [Node.js](https://nodejs.org/en/download/)

## <a name="register-single-page-app"></a>註冊單頁應用程式

應用程式必須[註冊](../active-directory/develop/developer-glossary.md#application-registration)於您的目錄中，才能接收來自 Azure Active Directory 的[存取權杖](../active-directory/develop/developer-glossary.md#access-token)。 應用程式註冊會在您的目錄中建立應用程式的[應用程式識別碼](../active-directory/develop/developer-glossary.md#application-id-client-id)。 

以 Azure AD B2C 目錄的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

[!INCLUDE [active-directory-b2c-switch-b2c-tenant](../../includes/active-directory-b2c-switch-b2c-tenant.md)]

1. 從 Azure 入口網站的服務清單中選取 [Azure AD B2C]。 

2. 在 B2C 設定中按一下 [應用程式]，然後按一下 [新增]。 

    若要在您的目錄中註冊範例 Web 應用程式，請使用下列設定：
    
    ![新增應用程式](media/active-directory-b2c-tutorials-spa/spa-registration.png)
    
    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | 我們範例單頁應用程式 | 輸入向取用者描述您的應用程式的 [名稱]。 | 
    | **包含 Web 應用程式 / Web API** | 是 | 針對單頁應用程式選取 [是]。 |
    | **允許隱含流程** | 是 | 請選取 [是]，因為應用程式使用 [OpenID Connect 登入](active-directory-b2c-reference-oidc.md)。 |
    | **回覆 URL** | `http://localhost:6420` | 回覆 URL 是 Azure AD B2C 傳回您應用程式要求之任何權杖的所在端點。 在此教學課程中，範例會在本機執行 (localhost)，並接聽連接埠 6420。 |
    | **包含原生用戶端** | 否 | 這是單頁應用程式，而不是原生用戶端，因此選取 [否]。 |
    
3. 按一下 [建立]  以註冊您的應用程式。

已註冊的應用程式會顯示在 Azure AD B2C 目錄的應用程式清單中。 從清單中選取單頁應用程式。 已註冊單頁應用程式的 [屬性] 窗格隨即顯示。

![單頁應用程式屬性](./media/active-directory-b2c-tutorials-spa/b2c-spa-properties.png)

請記下 [應用程式用戶端識別碼]。 此識別碼可唯一識別應用程式，後續在此教學課程中設定應用程式時將會用到。

## <a name="create-policies"></a>建立原則

Azure AD B2C 原則會定義使用者工作流程。 例如，登入、註冊、變更密碼以及編輯設定檔，都是常見的工作流程。

### <a name="create-a-sign-up-or-sign-in-policy"></a>建立註冊或登入原則

若要註冊要存取並登入 Web 應用程式的使用者，請建立**註冊或登入原則**。

1. 在 Azure AD B2C 入口網站頁面中選取 [註冊或登入原則]，然後按一下 [新增]。

    若要設定您的原則，請使用下列設定：

    ![新增註冊或登入原則](media/active-directory-b2c-tutorials-web-app/add-susi-policy.png)

    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | SiUpIn | 輸入原則的 [名稱]。 原則名稱前面會加上 **B2C_1_**。 您在範例程式碼中會使用完整的原則名稱 **B2C_1_SiUpIn**。 | 
    | **身分識別提供者** | 電子郵件註冊 | 用來唯一識別使用者的身分識別提供者。 |
    | **註冊屬性** | 顯示名稱和郵遞區號 | 選取在註冊期間要對使用者收集的屬性。 |
    | **應用程式宣告** | 顯示名稱、郵遞區號、使用者是新的、使用者的物件識別碼 | 選取要包含在[存取權杖](../active-directory/develop/developer-glossary.md#access-token)中的[宣告](../active-directory/develop/developer-glossary.md#claim)。 |

2. 按一下 [建立] 以建立原則。 

### <a name="create-a-profile-editing-policy"></a>建立設定檔編輯原則

若要允許使用者自行重設其使用者設定檔資訊，請建立**設定檔編輯原則**。

1. 在 Azure AD B2C 入口網站頁面中選取 [設定檔編輯原則]，然後按一下 [新增]。

    若要設定您的原則，請使用下列設定：

    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | SiPe | 輸入原則的 [名稱]。 原則名稱前面會加上 **B2C_1_**。 您在範例程式碼中會使用完整的原則名稱 **B2C_1_SiPe**。 | 
    | **身分識別提供者** | 本機帳戶登入 | 用來唯一識別使用者的身分識別提供者。 |
    | **設定檔屬性** | 顯示名稱和郵遞區號 | 選取使用者在設定檔編輯期間可修改的屬性。 |
    | **應用程式宣告** | 顯示名稱、郵遞區號、使用者的物件識別碼 | 選取在設定檔編輯成功後要包含在[存取權杖](../active-directory/develop/developer-glossary.md#access-token)中的[宣告](../active-directory/develop/developer-glossary.md#claim)。 |

2. 按一下 [建立] 以建立原則。 

### <a name="create-a-password-reset-policy"></a>建立密碼重設原則

若要在您的應用程式上啟用密碼重設，您必須建立**密碼重設原則**。 此原則會說明取用者在密碼重設期間的體驗，以及應用程式在重設成功時所將收到的權杖內容。

1. 在 Azure AD B2C 入口網站頁面中選取 [密碼重設原則]，然後按一下 [新增]。

    若要設定您的原則，請使用下列設定。

    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | SSPR | 輸入原則的 [名稱]。 原則名稱前面會加上 **B2C_1_**。 您在範例程式碼中會使用完整的原則名稱 **B2C_1_SSPR**。 | 
    | **身分識別提供者** | 使用電子郵件地址重設密碼 | 這是用來唯一識別使用者的身分識別提供者。 |
    | **應用程式宣告** | 使用者的物件識別碼 | 選取在密碼重設成功後要包含在[存取權杖](../active-directory/develop/developer-glossary.md#access-token)中的[宣告](../active-directory/develop/developer-glossary.md#claim)。 |

2. 按一下 [建立] 以建立原則。 

## <a name="update-single-page-app-code"></a>更新單頁應用程式程式碼

現在，您已註冊應用程式並建立原則，接下來您必須將應用程式設定為使用您的 Azure AD B2C 目錄。 在此教學課程中，您會設定可從 GitHub 下載的範例 SPA JavaScript 應用程式。 

[下載 zip 檔案](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip)，或從 GitHub 複製範例 Web 應用程式。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
```
範例應用程式會示範單頁應用程式如何使用 Azure AD B2C 進行使用者註冊、登入及呼叫受保護的 Web API。 您必須將此應用程式變更為使用您目錄中的應用程式註冊，以及設定您所建立的原則。 

若要變更應用程式設定：

1. 開啟 Node.js 單頁應用程式範例中的 `index.html` 檔案。
2. 使用 Azure AD B2C 目錄註冊資訊來設定此範例。 變更下列幾行程式碼 (請務必將其中的值，取代為您的目錄和 API 名稱)：

    ```javascript
    // The current application coordinates were pre-registered in a B2C directory.
    var applicationConfig = {
        clientID: '<Application ID for your SPA obtained from portal app registration>',
        authority: "https://fabrikamb2c.b2clogin.com/tfp/fabrikamb2c.onmicrosoft.com/B2C_1_<Sign-up or sign-in policy name>",
        b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/demoapi/demo.read"],
        webApi: 'https://fabrikamb2chello.azurewebsites.net/hello',
    };
    ```

    此教學課程中使用的原則名稱是 **B2C_1_SiUpIn**。 如果您使用不同的原則名稱，請在 `authority` 值中使用您的原則名稱。

## <a name="run-the-sample"></a>執行範例

1. 啟動 Node.js 命令提示字元。
2. 切換至包含 Node.js 範例的目錄。 例如 `cd c:\active-directory-b2c-javascript-msal-singlepageapp`
3. 執行下列命令：
    ```
    npm install && npm update
    node server.js
    ```

    主控台視窗會顯示裝載應用程式的連接埠號碼。
    
    ```
    Listening on port 6420...
    ```

4. 使用瀏覽器瀏覽至位址 `http://localhost:6420` 以檢視應用程式。

範例應用程式支援註冊、登入、編輯設定檔和密碼重設。 此教學課程特別說明使用者如何以電子郵件地址註冊並使用應用程式。 您可以自行探索其他案例。

### <a name="sign-up-using-an-email-address"></a>使用電子郵件地址註冊

1. 按一下 [登入] 以 SPA 應用程式的使用者身分登入。 這會使用您在上一個步驟中定義的 **B2C_1_SiUpIn** 原則。

2. Azure AD B2C 會顯示含有註冊連結的登入頁面。 由於您還沒有帳戶，因此請按一下 [立即註冊] 連結。 

3. 註冊工作流程會顯示一個使用電子郵件地址來收集並驗證使用者身分識別的頁面。 註冊工作流程也會收集使用者的密碼，以及在原則中定義的要求屬性。

    請使用有效的電子郵件地址，並使用驗證碼進行驗證。 設定密碼。 輸入要求的屬性值。 

    ![註冊工作流程](media/active-directory-b2c-tutorials-desktop-app/sign-up-workflow.png)

4. 按一下 [建立]，在 Azure AD B2C 目錄中建立本機帳戶。

現在，使用者已可使用電子郵件地址登入並使用 SPA 應用程式。

> [!NOTE]
> 登入之後，應用程式會顯示「權限不足」錯誤。 因為您嘗試從示範目錄存取資源，所以會收到這個錯誤。 因為您的存取權杖只對您的 Azure AD 目錄有效，所以此 API 呼叫未經授權。 繼續進行下一個教學課程，為您的目錄建立受保護的 Web API。 

## <a name="clean-up-resources"></a>清除資源

如果您想要嘗試其他 Azure AD B2C 教學課程，您可以使用 Azure AD B2C 目錄。 不再需要時，您可以[刪除您的 Azure AD B2C 目錄](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant)。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何建立 Azure AD B2C 目錄、建立原則，以及將範例單頁應用程式更新為使用您的 Azure AD B2C 目錄。 請繼續進行下一個教學課程，以了解如何從傳統型應用程式註冊、設定及呼叫受保護的 Web API。

> [!div class="nextstepaction"]
> 
