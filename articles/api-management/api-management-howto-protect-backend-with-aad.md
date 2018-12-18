---
title: 使用 OAuth 2.0 搭配 Azure Active Directory 與 API 管理來保護 API | Microsoft Docs
description: 了解如何使用 Azure Active Directory 與 API 管理保護 Web API 後端。
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2018
ms.author: apimpm
ms.openlocfilehash: 06350d30999cb056babbd001f98a6c3a5fdbac6c
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39576989"
---
# <a name="protect-an-api-by-using-oauth-20-with-azure-active-directory-and-api-management"></a>使用 OAuth 2.0 搭配 Azure Active Directory 與 API 管理來保護 API

本指南說明如何使用 OAuth 2.0 通訊協定與 Azure Active Directory (Azure AD)，設定 Azure API 管理執行個體來保護 API。 

## <a name="prerequisites"></a>必要條件
若要依照本文中的步驟進行，您必須有：
* API 管理執行個體
* 正在使用 API 管理執行個體發佈的 API
* Azure AD 租用戶

## <a name="overview"></a>概觀

以下為步驟概述：

1. 在 Azure AD 中註冊應用程式 (後端應用程式) 來代表 API。
2. 在 Azure AD 中註冊另一個應用程式 (用戶端應用程式) 來代表需要呼叫 API 的用戶端應用程式。
3. 在 Azure AD 中授與權限，允許用戶端應用程式呼叫後端應用程式。
4. 設定讓開發人員主控台使用 OAuth 2.0 使用者授權。
5. 新增**驗證 JWT** 原則，以驗證每個連入要求的 OAuth 權杖。

## <a name="register-an-application-in-azure-ad-to-represent-the-api"></a>在 Azure AD 中註冊應用程式來代表 API

若要使用 Azure AD 保護 API，第一個步驟是在 Azure AD 中註冊一個應用程式來代表 API。 

1. 瀏覽至您的 Azure AD 租用戶，然後瀏覽至 [應用程式註冊]。

2. 選取 [新增應用程式註冊]。 

3. 提供應用程式的名稱。 (此範例中，名稱為 `backend-app`。)  

4. 選擇 [Web 應用程式/API] 作為 [應用程式類型]。 

5. 在 [登入 URL] 中，您可以使用 `https://localhost` 作為預留位置。

6. 選取 [建立] 。

應用程式建立後，請記下**應用程式識別碼**，以供後續步驟使用。 

## <a name="register-another-application-in-azure-ad-to-represent-a-client-application"></a>在 Azure AD 中註冊另一個應用程式來代表用戶端應用程式

每個呼叫 API 的用戶端應用程式也必須在 Azure AD 中註冊為應用程式。 在此範例中，範例用戶端應用程式是 API 管理開發人員入口網站中的開發人員主控台。 以下說明如何在 Azure AD 中註冊另一個應用程式，來代表開發人員主控台。

1. 選取 [新增應用程式註冊]。 

2. 提供應用程式的名稱。 (此範例中，名稱為 `client-app`。)

3. 選擇 [Web 應用程式/API] 作為 [應用程式類型]。  

4. 在 [登入 URL] 中，您可以使用 `https://localhost` 作為預留位置，或使用 API 管理執行個體的登入 URL。 (此範例中，URL 為 `https://contoso5.portal.azure-api.net/signin`。)

5. 選取 [建立] 。

應用程式建立後，請記下**應用程式識別碼**，以供後續步驟使用。 

現在，建立此應用程式的用戶端密碼，以供後續步驟使用。

1. 再次選取 [設定]，然後移至 [金鑰]。

2. 在 [密碼] 之下，提供 [金鑰描述]。 選擇金鑰應過期的時間，然後選取 [儲存]。

記下金鑰的值。 

## <a name="grant-permissions-in-azure-ad"></a>在 Azure AD 中授與權限

現在您已註冊兩個應用程式來代表 API 和開發人員主控台，您需要授與權限，允許用戶端應用程式呼叫後端應用程式。  

1. 瀏覽至 [應用程式註冊]。 

2. 選取 `client-app` 並移至 [設定]。

3. 選取 [必要權限] > [新增]。

4. 選取 [選取 API]，然後搜尋 `backend-app`。

5. 在 [委派的權限] 之下，選取 `Access backend-app`。 

6. 選取 [選取]，然後選取 [完成]。 

> [!NOTE]
> 如果 [Azure Active Directory] 並未列在其他應用程式的權限之下，請選取 [新增] 從清單將其新增。
> 
> 

## <a name="enable-oauth-20-user-authorization-in-the-developer-console"></a>在開發人員主控台中啟用 OAuth 2.0 使用者授權

此時，您已在 Azure AD 中建立我們的應用程式，並已授與適當的權限，讓用戶端應用程式可以呼叫後端應用程式。 

在此範例中，開發人員主控台是用戶端應用程式。 下列步驟說明如何在開發人員主控台中啟用 OAuth 2.0 使用者授權。 

1. 瀏覽至您的 API 管理執行個體。

2. 選取 [OAuth 2.0] > [新增]。

3. 提供[顯示名稱] 和 [描述]。

4. 針對 [用戶端註冊頁面 URL]，輸入預留位置值，例如 `http://localhost`。 [用戶端註冊頁面 URL] 指向的頁面可供使用者用來針對支援此功能的 OAuth 2.0 提供者建立和設定其專屬帳戶。 在此範例中，使用者沒有建立和設定自己的帳戶，因此您可改使預留位置。

5. 針對 [授權授與類型]，選取 [授權碼]。

6. 指定 [授權端點 URL] 和 [權杖端點 URL]。 從 Azure AD 租用戶中的 [端點] 頁面擷取這些值。 瀏覽至 [應用程式註冊] 頁面，然後選取 [端點]。

7. 複製 [OAuth 2.0 授權端點] 並貼到 [授權端點 URL] 文字方塊中。

8. 複製 [OAuth 2.0 權杖端點] 並貼到 [權杖端點 URL] 文字方塊中。 除了貼入權杖端點以外，新增名為 **resource** 主體參數。 針對此參數的值，使用後端應用程式的 [應用程式識別碼]。

9. 接著，指定用戶端認證。 這些是用戶端應用程式的認證。

10. 針對 [用戶端識別碼]，使用用戶端應用程式的 [應用程式識別碼]。

11. [用戶端密碼] 請使用您稍早為用戶端應用程式建立的金鑰。 

12. 緊接在用戶端密碼後面的是授權碼授與類型的 [redirect_url]。 記下此 URL。

13. 選取 [建立] 。

14. 回到用戶端應用程式的 [設定] 頁面。

15. 選取 [回覆 URL]，並在第一行貼上 **redirect_url**。 在此範例中，您已將第一行中的 `https://localhost` 取代為此 URL。  

現在您已經設定 OAuth 2.0 授權伺服器，開發人員主控台可以從 Azure AD 取得存取權杖。 

下一個步驟是要讓您的 API 能夠使用 OAuth 2.0 使用者授權。 這可讓開發人員主控台知道它需要先代表使用者取得存取權杖，然後才能呼叫我們的 API。

1. 瀏覽至您的 API 管理執行個體，然後移至 [API]。

2. 選取您要保護的 API。 在此範例中，您會使用 `Echo API`。

3. 移至 [設定] 。

4. 在 [安全性] 之下，選擇 [OAuth 2.0]，然後選取您先前設定的 OAuth 2.0 伺服器。 

5. 選取 [ **儲存**]。

## <a name="successfully-call-the-api-from-the-developer-portal"></a>從開發人員入口網站成功呼叫 API

現在 `Echo API` 已啟用 OAuth 2.0 使用者授權，開發人員主控台會先代表使用者取得存取權杖，然後才呼叫 API。

1. 瀏覽至開發人員入口網站中 `Echo API` 之下的任何作業，然後選取 [試試看]。 這將帶您前往 [開發人員主控台]。

2. 請注意，在 [授權] 區段中有一個新項目對應到您剛才新增的授權伺服器。

3. 從授權下拉式清單中選取 [授權碼]，系統會提示您登入 Azure AD 租用戶。 如果您已經使用帳戶登入，就不會提示您。

4. 成功登入之後，要求中就會新增 `Authorization` 標頭，以及來自 Azure AD 的存取權杖。 以下是範例權杖 (Base64 編碼)：

   ```
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCIsImtpZCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCJ9.eyJhdWQiOiIxYzg2ZWVmNC1jMjZkLTRiNGUtODEzNy0wYjBiZTEyM2NhMGMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC80NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgvIiwiaWF0IjoxNTIxMTUyNjMzLCJuYmYiOjE1MjExNTI2MzMsImV4cCI6MTUyMTE1NjUzMywiYWNyIjoiMSIsImFpbyI6IkFWUUFxLzhHQUFBQUptVzkzTFd6dVArcGF4ZzJPeGE1cGp2V1NXV1ZSVnd1ZXZ5QU5yMlNkc0tkQmFWNnNjcHZsbUpmT1dDOThscUJJMDhXdlB6cDdlenpJdzJLai9MdWdXWWdydHhkM1lmaDlYSGpXeFVaWk9JPSIsImFtciI6WyJyc2EiXSwiYXBwaWQiOiJhYTY5ODM1OC0yMWEzLTRhYTQtYjI3OC1mMzI2NTMzMDUzZTkiLCJhcHBpZGFjciI6IjEiLCJlbWFpbCI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiSmlhbmciLCJnaXZlbl9uYW1lIjoiTWlhbyIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpcGFkZHIiOiIxMzEuMTA3LjE3NC4xNDAiLCJuYW1lIjoiTWlhbyBKaWFuZyIsIm9pZCI6IjhiMTU4ZDEwLWVmZGItNDUxMS1iOTQzLTczOWZkYjMxNzAyZSIsInNjcCI6InVzZXJfaW1wZXJzb25hdGlvbiIsInN1YiI6IkFGaWtvWFk1TEV1LTNkbk1pa3Z3MUJzQUx4SGIybV9IaVJjaHVfSEM1aGciLCJ0aWQiOiI0NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgiLCJ1bmlxdWVfbmFtZSI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsInV0aSI6ImFQaTJxOVZ6ODBXdHNsYjRBMzBCQUEiLCJ2ZXIiOiIxLjAifQ.agGfaegYRnGj6DM_-N_eYulnQdXHhrsus45QDuApirETDR2P2aMRxRioOCR2YVwn8pmpQ1LoAhddcYMWisrw_qhaQr0AYsDPWRtJ6x0hDk5teUgbix3gazb7F-TVcC1gXpc9y7j77Ujxcq9z0r5lF65Y9bpNSefn9Te6GZYG7BgKEixqC4W6LqjtcjuOuW-ouy6LSSox71Fj4Ni3zkGfxX1T_jiOvQTd6BBltSrShDm0bTMefoyX8oqfMEA2ziKjwvBFrOjO0uK4rJLgLYH4qvkR0bdF9etdstqKMo5gecarWHNzWi_tghQu9aE3Z3EZdYNI_ZGM-Bbe3pkCfvEOyA
   ```

5. 選取 [傳送]，而您可以成功呼叫 API。


## <a name="configure-a-jwt-validation-policy-to-pre-authorize-requests"></a>設定 JWT 驗證原則來預先授權要求

此時，當使用者嘗試從開發人員主控台進行呼叫時，系統會提示使用者登入。 開發人員主控台會代表使用者取得存取權杖。

但如果有人沒用權杖或用無效的權杖呼叫 API 會怎麼樣？ 例如，即使您刪除 `Authorization`標頭，仍然可以呼叫 API。 這是因為 API 管理此時不會驗證存取權杖。 它只會將 `Authorization` 標頭傳遞至後端 API。

您可以使用[驗證 JWT](api-management-access-restriction-policies.md#ValidateJWT) 原則，藉由驗證每個傳入要求的存取權杖來預先授權 API 管理中的要求。 如果要求沒有有效的權杖，則 API 管理會封鎖該要求。 例如，您可以將下列原則新增至 `Echo API` 的 `<inbound>` 原則區段。 它會檢查存取權杖中的 audience 宣告，並在權杖無效時傳回錯誤訊息。 如需如何設定原則的資訊，請參閱[設定或編輯原則](set-edit-policies.md)。

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/{aad-tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud">
            <value>{Application ID of backend-app}</value>
        </claim>
    </required-claims>
</validate-jwt>
```

## <a name="build-an-application-to-call-the-api"></a>建置應用程式來呼叫 API

在本指南中，您使用了 API 管理中的開發人員主控台作為範例用戶端應用程式，以呼叫 OAuth 2.0 所保護的 `Echo API`。 若要深入了解如何建置應用程式及實作 OAuth 2.0，請參閱 [Azure Active Directory 程式碼範例](../active-directory/develop/sample-v1-code.md)。

## <a name="next-steps"></a>後續步驟
* 深入了解 [Azure Active Directory 和 OAuth2.0](../active-directory/develop/authentication-scenarios.md)。
* 查看更多有關 API 管理的 [視訊](https://azure.microsoft.com/documentation/videos/index/?services=api-management) 。
* 如需其他保護後端服務的方式，請參閱[相互憑證驗證](api-management-howto-mutual-certificates.md)。

* [建立 API 管理服務執行個體](get-started-create-service-instance.md)。

* [管理第一個 API](import-and-publish.md)。
