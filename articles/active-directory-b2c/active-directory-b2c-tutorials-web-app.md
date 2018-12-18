---
title: 教學課程 - 讓 Web 應用程式能夠使用 Azure Active Directory B2C 向帳戶進行驗證 | Microsoft Docs
description: 關於如何使用 Azure Active Directory B2C 為 ASP.NET Web 應用程式提供使用者登入的教學課程。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.author: davidmu
ms.date: 1/23/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.component: B2C
ms.openlocfilehash: e215577fdb39b3dc1a9c5ce641c44e3cdef8fb45
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604088"
---
# <a name="tutorial-enable-a-web-application-to-authenticate-with-accounts-using-azure-active-directory-b2c"></a>教學課程：讓 Web 應用程式能夠使用 Azure Active Directory B2C 向帳戶進行驗證

本教學課程將說明如何使用 Azure Active Directory (Azure AD) B2C 在 ASP.NET Web 應用程式中登入和註冊使用者。 Azure AD B2C 可讓您的應用程式使用開放式標準通訊協定向社交帳戶、企業帳戶和 Azure Active Directory 帳戶進行驗證。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在您的 Azure AD B2C 租用戶中註冊範例 ASP.NET Web 應用程式。
> * 建立使用者註冊、登入、編輯設定檔和密碼重設的原則。
> * 設定範例 Web 應用程式以使用您的 Azure AD B2C 租用戶。 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

* 建立您自己的 [Azure AD B2C 租用戶](active-directory-b2c-get-started.md)
* 安裝包含 **ASP.NET 和 Web 開發**工作負載的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。

## <a name="register-web-app"></a>註冊 Web 應用程式

應用程式必須[註冊](../active-directory/develop/developer-glossary.md#application-registration)於您的租用戶中，才能接收來自 Azure Active Directory 的[存取權杖](../active-directory/develop/developer-glossary.md#access-token)。 應用程式註冊會在您的租用戶中建立應用程式的[應用程式識別碼](../active-directory/develop/developer-glossary.md#application-id-client-id)。 

以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

[!INCLUDE [active-directory-b2c-switch-b2c-tenant](../../includes/active-directory-b2c-switch-b2c-tenant.md)]

1. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。 您現在應使用在前一個教學課程中建立的租用戶。 

2. 在 B2C 設定中按一下 [應用程式]，然後按一下 [新增]。 

    若要在您的租用戶中註冊範例 Web 應用程式，請使用下列設定：

    ![新增應用程式](media/active-directory-b2c-tutorials-web-app/web-app-registration.png)
    
    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | 我的範例 Web 應用程式 | 輸入向取用者描述您的應用程式的 [名稱]。 | 
    | **包含 Web 應用程式 / Web API** | 是 | 針對 Web 應用程式選取 [是]。 |
    | **允許隱含流程** | 是 | 請選取 [是]，因為應用程式使用 [OpenID Connect 登入](active-directory-b2c-reference-oidc.md)。 |
    | **回覆 URL** | `https://localhost:44316` | 回覆 URL 是 Azure AD B2C 傳回您應用程式要求之任何權杖的所在端點。 在本教學課程中，範例會在本機執行 (localhost)，並接聽連接埠 44316。 |
    | **包含原生用戶端** | 否 | 這是 Web 應用程式，而不是原生用戶端，因此請選取 [否]。 |
    
3. 按一下 [建立]  以註冊您的應用程式。

已註冊的應用程式會顯示在 Azure AD B2C 租用戶的應用程式清單中。 從清單中選取 Web 應用程式。 Web 應用程式的 [屬性] 窗格隨即顯示。

![Web 應用程式屬性](./media/active-directory-b2c-tutorials-web-app/b2c-web-app-properties.png)

記下 [應用程式識別碼]。 此識別碼可唯一識別應用程式，後續在本教學課程中設定應用程式時將會用到。

### <a name="create-a-client-password"></a>建立用戶端密碼

Azure AD B2C 會對[用戶端應用程式](../active-directory/develop/developer-glossary.md#client-application)使用 OAuth2 授權。 Web 應用程式屬於[機密用戶端](../active-directory/develop/developer-glossary.md#web-client)，需要用戶端識別碼或應用程式識別碼和用戶端秘密、用戶端密碼或應用程式金鑰。

1. 選取已註冊之 Web 應用程式的 [金鑰] 頁面，然後按一下 [產生金鑰]。

2. 按一下 [儲存] 以顯示應用程式金鑰。

    ![應用程式一般金鑰頁面](media/active-directory-b2c-tutorials-web-app/app-general-keys-page.png)

金鑰會在入口網站中顯示一次。 請務必複製並儲存金鑰值。 您在設定應用程式時將需要此值。 將確保金鑰的安全性。 不要公開共用金鑰。

## <a name="create-policies"></a>建立原則

Azure AD B2C 原則會定義使用者工作流程。 例如，登入、註冊、變更密碼以及編輯設定檔，都是常見的工作流程。

### <a name="create-a-sign-up-or-sign-in-policy"></a>建立註冊或登入原則

若要註冊要存取並登入 Web 應用程式的使用者，請建立**註冊或登入原則**。

1. 在 Azure AD B2C 入口網站頁面中選取 [註冊或登入原則]，然後按一下 [新增]。

    若要設定您的原則，請使用下列設定：

    ![新增註冊或登入原則](media/active-directory-b2c-tutorials-web-app/add-susi-policy.png)

    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | SiUpIn | 輸入原則的 [名稱]。 原則名稱前面會加上 **b2c_1_**。 您在範例程式碼中會使用完整的原則名稱 **b2c_1_SiUpIn**。 | 
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
    | **名稱** | SiPe | 輸入原則的 [名稱]。 原則名稱前面會加上 **b2c_1_**。 您在範例程式碼中會使用完整的原則名稱 **b2c_1_SiPe**。 | 
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
    | **名稱** | SSPR | 輸入原則的 [名稱]。 原則名稱前面會加上 **b2c_1_**。 您在範例程式碼中會使用完整的原則名稱 **b2c_1_SSPR**。 | 
    | **身分識別提供者** | 使用電子郵件地址重設密碼 | 這是用來唯一識別使用者的身分識別提供者。 |
    | **應用程式宣告** | 使用者的物件識別碼 | 選取在密碼重設成功後要包含在[存取權杖](../active-directory/develop/developer-glossary.md#access-token)中的[宣告](../active-directory/develop/developer-glossary.md#claim)。 |

2. 按一下 [建立] 以建立原則。 

## <a name="update-web-app-code"></a>更新 Web 應用程式的程式碼

現在，您已註冊 Web 應用程式並建立原則，接下來您必須設定應用程式以使用您的 Azure AD B2C 租用戶。 在本教學課程中，您會設定可從 GitHub 下載的範例 Web 應用程式。 

[下載 zip 檔案](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip)，或從 GitHub 複製範例 Web 應用程式。 請務必要擷取資料夾中的檔案範例，路徑的總字元長度少於 260。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

範例 ASP.NET Web 應用程式是用來建立和更新待辦事項清單的簡易工作清單應用程式。 應用程式會使用 [Microsoft OWIN 中介軟體元件](https://docs.microsoft.com/aspnet/aspnet/overview/owin-and-katana/)，讓使用者註冊並使用您 Azure AD B2C 租用戶中的應用程式。 藉由建立 Azure AD B2C 原則，使用者將可使用社交帳戶，或建立帳戶以作為其存取應用程式的身分識別。 

範例方案中有兩個專案：

**Web 應用程式範例應用程式 (TaskWebApp)：** 用來建立和編輯工作清單的 Web 應用程式。 此 Web 應用程式會使用**註冊或登入**原則來註冊或登入使用者。

**Web API 範例應用程式 (TaskService)：** 支援建立、讀取、更新和刪除工作清單功能的 Web API。 此 Web API 會受到 Azure AD B2C 的保護，且由 Web 應用程式呼叫。

您需要將應用程式變更為使用租用戶中的應用程式註冊，其包含您先前記錄下來的應用程式識別碼和金鑰。 您也需要設定您已建立的原則。 範例 Web 應用程式會在 Web.config 檔案中將組態值定義為應用程式設定。 若要變更應用程式設定：

1. 在 Visual Studio 中開啟 **B2C-WebAPI-DotNet** 方案。

2. 在 **TaskWebApp** Web 應用程式專案中開啟 **Web.config** 檔案。 將 `ida:Tenant` 的值更換為您所建立的租用戶名稱。 將 `ida:ClientId` 的值更換為您所記錄下來的應用程式識別碼。 將 `ida:ClientSecret` 的值更換為您所記錄下來的金鑰。

3. 在 **Web.config** 檔案中，將 `ida:SignUpSignInPolicyId` 的值更換為 `b2c_1_SiUpIn`。 將 `ida:EditProfilePolicyId` 的值更換為 `b2c_1_SiPe`。 將 `ida:ResetPasswordPolicyId` 的值更換為 `b2c_1_SSPR`。

## <a name="run-the-sample-web-app"></a>執行範例 Web 應用程式

在方案總管中，以滑鼠右鍵按一下 **TaskWebApp** 專案，然後按一下 [設為起始專案]。

按 **F5** 鍵以啟動 Web 應用程式。 預設瀏覽器隨即啟動，並顯示本機網站位址 `https://localhost:44316/`。 

範例應用程式支援註冊、登入、編輯設定檔和密碼重設。 本教學課程特別說明使用者如何以電子郵件地址註冊並使用應用程式。 您可以自行探索其他案例。

### <a name="sign-up-using-an-email-address"></a>使用電子郵件地址註冊

1. 按一下頂端橫幅中的 [註冊/登入] 連結，註冊成為 Web 應用程式的使用者。 這會使用您在上一個步驟中定義的 **b2c_1_SiUpIn** 原則。

2. Azure AD B2C 會顯示含有註冊連結的登入頁面。 由於您還沒有帳戶，因此請按一下 [立即註冊] 連結。 

3. 註冊工作流程會顯示一個使用電子郵件地址來收集並驗證使用者身分識別的頁面。 註冊工作流程也會收集使用者的密碼，以及在原則中定義的要求屬性。

    請使用有效的電子郵件地址，並使用驗證碼進行驗證。 設定密碼。 輸入要求的屬性值。 

    ![註冊工作流程](media/active-directory-b2c-tutorials-web-app/sign-up-workflow.png)

4. 按一下 [建立]，在 Azure AD B2C 租用戶中建立本機帳戶。

現在，使用者已可使用電子郵件地址登入並使用 Web 應用程式。

## <a name="clean-up-resources"></a>清除資源

如果您想要嘗試其他 Azure AD B2C 教學課程，您可以使用 Azure AD B2C 租用戶。 不再需要時，您可以[刪除您的 Azure AD B2C 租用戶](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何建立 Azure AD B2C 租用戶、建立原則，以及更新範例 Web 應用程式以使用您的 Azure AD B2C 租用戶。 請繼續執行下一個教學課程，以了解如何註冊、設定及呼叫受到 Azure AD B2C 租用戶保護的 ASP.NET Web API。

> [!div class="nextstepaction"]
> [教學課程：使用 Azure Active Directory B2C 保護 ASP.NET Web API](active-directory-b2c-tutorials-web-api.md)
