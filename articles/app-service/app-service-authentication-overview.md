---
title: Azure App Service 中的驗證和授權 | Microsoft Docs
description: Azure App Service 的驗證/授權功能的概念性參考和概觀
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''
ms.assetid: b7151b57-09e5-4c77-a10c-375a262f17e5
ms.service: app-service
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 08/24/2018
ms.author: mahender,cephalin
ms.openlocfilehash: 46f8602583329a0516edb9af59e53754ca349555
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43336799"
---
# <a name="authentication-and-authorization-in-azure-app-service"></a>Azure App Service 中的驗證與授權

Azure App Service 提供內建的驗證和授權支援，因此您在 Web 應用程式、API 和行動裝置後端以及 [Azure Functions](../azure-functions/functions-overview.md) 中幾乎不需要寫入或完全無需寫入程式碼，即可登入使用者及存取資料。 本文說明 App Service 如何協助您簡化應用程式的驗證和授權。 

安全的驗證和授權要求對安全性的深入了解，包括同盟、加密、[JSON Web 權杖 (JWT)](https://wikipedia.org/wiki/JSON_Web_Token) 管理、[授與類型](https://oauth.net/2/grant-types/)等等。 App Service 會提供這些公用程式，以便您可以將更多的時間和精力花在為客戶提供商務價值上。

> [!NOTE]
> 您不需要使用 App Service 來進行驗證和授權。 許多 Web 架構都會搭載安全性功能，您可以視需要加以使用。 如果您需要的彈性高於 App Service 所提供的彈性，也可以撰寫您自己的公用程式。  
>

如需原生行動應用程式的專屬資訊，請參閱 [Azure App Service 的行動應用程式使用者驗證和授權](../app-service-mobile/app-service-mobile-auth.md)。

## <a name="how-it-works"></a>運作方式

驗證和授權模組會在與應用程式程式碼相同的沙箱中執行。 此模組啟用時，每個連入的 HTTP 要求會先通過此模組，再由應用程式程式碼處理。

![](media/app-service-authentication-overview/architecture.png)

此模組會處理應用程式的幾件事：

- 向指定提供者驗證使用者
- 驗證、儲存及重新整理權杖
- 管理已驗證的工作階段
- 將身分識別資訊插入要求標頭中

此模組會與應用程式程式碼分開執行，並且會使用應用程式設定加以設定。 不需要任何 SDK、特定語言或對應用程式程式碼進行任何變更。 

### <a name="user-claims"></a>使用者宣告

在所有語言架構中，App Service 都會將使用者的宣告插入要求標頭內以供程式碼使用。 在 ASP.NET 4.6 應用程式中，App Service 會使用已驗證的使用者宣告填入 [ClaimsPrincipal.Current](/dotnet/api/system.security.claims.claimsprincipal.current)，因此您可以遵循標準的 .NET 程式碼模式，包括 `[Authorize]` 屬性。 同樣地，在 PHP 應用程式中，App Service 會填入 `_SERVER['REMOTE_USER']` 變數。

在 [Azure Functions](../azure-functions/functions-overview.md) 中，系統不會針對 .NET 程式碼將 `ClaimsPrincipal.Current` 序列化，不過您仍可以在要求標頭中找到使用者宣告。

如需詳細資訊，請參閱[存取使用者宣告](app-service-authentication-how-to.md#access-user-claims)。

### <a name="token-store"></a>權杖存放區

App Service 會提供內建的權杖存放區，也就是與 Web 應用程式、API 或原生行動應用程式相關聯的權杖存放庫。 當您使用任何提供者啟用驗證時，此權杖存放區就會立即供應用程式使用。 如果應用程式程式碼需要代表使用者存取這些提供者的資料，例如： 

- 張貼至已驗證使用者的 Facebook 時間軸
- 從 Azure Active Directory 圖形 API 或甚至 Microsoft Graph 讀取使用者的公司資料

系統會針對已驗證的工作階段快取識別碼權杖、存取權杖和重新整理權杖，且只有相關聯的使用者能存取這些權杖。  

您通常必須撰寫程式碼，才能在應用程式中收集、儲存及重新整理這些權杖。 使用權杖存放區時，您只有在需要權杖時才會[取出權杖](app-service-authentication-how-to.md#retrieve-tokens-in-app-code)，並在權杖失效時才會[告知 App Service 加以重新整理](app-service-authentication-how-to.md#refresh-access-tokens)。 

如果您不需要在應用程式中使用權杖，可以將權杖存放區停用。

### <a name="logging-and-tracing"></a>記錄和追蹤

如果您[啟用應用程式記錄](web-sites-enable-diagnostic-log.md)，則會直接在記錄檔中看到驗證和授權追蹤。 如果您看到非預期的驗證錯誤，可以在現有的應用程式記錄中查看，輕鬆地找到所有詳細資料。 如果您啟用[失敗要求追蹤](web-sites-enable-diagnostic-log.md)，就可以看到驗證和授權模組可能在失敗要求中所扮演的確切角色。 在追蹤記錄中，尋找名為 `EasyAuthModule_32/64` 的模組參考。 

## <a name="identity-providers"></a>識別提供者

App Service 使用[同盟身分識別](https://en.wikipedia.org/wiki/Federated_identity)，由第三方識別提供者為您管理使用者身分識別和驗證流程。 預設可用的識別提供者有五個： 

| 提供者 | 登入端點 |
| - | - |
| [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) | `/.auth/login/aad` |
| [Microsoft 帳戶](../active-directory/develop/active-directory-appmodel-v2-overview.md) | `/.auth/login/microsoftaccount` |
| [Facebook](https://developers.facebook.com/docs/facebook-login) | `/.auth/login/facebook` |
| [Google](https://developers.google.com/+/web/api/rest/oauth) | `/.auth/login/google` |
| [Twitter](https://developer.twitter.com/en/docs/basics/authentication) | `/.auth/login/twitter` |

當您利用上述其中一個提供者啟用驗證和授權時，其登入端點即可用來驗證使用者，以及用來驗證提供者的驗證權杖。 您可以輕鬆地為使用者提供任何數目的上述登入選項。 您也可以整合其他識別提供者或[您自己的自訂身分識別解決方案][custom-auth]。

## <a name="authentication-flow"></a>驗證流程

所有提供者的驗證流程皆相同，但會根據您是否要使用提供者的 SDK 登入而有所不同：

- 不使用提供者 SDK：應用程式會將同盟登入委派給 App Service。 瀏覽器應用程式通常是這種情況，可以向使用者顯示提供者的登入頁面。 伺服器程式碼會管理登入程序，因此也稱為「伺服器導向流程」或「伺服器流程」。 此案例適用於 Web 應用程式。 它也適用於使用 Mobile Apps 用戶端 SDK 將使用者登入的原生應用程式，因為 SDK 會開啟 Web 檢視，使用 App Service 驗證將使用者登入。 
- 使用提供者 SDK：應用程式會以手動方式將使用者登入，然後將驗證權杖提交給 App Service 進行驗證。 無瀏覽器應用程式通常是這種情況，無法向使用者顯示提供者的登入頁面。 應用程式程式碼會管理登入程序，因此也稱為「用戶端導向流程」或「用戶端流程」。 此案例適用於 REST API、[Azure Functions](../azure-functions/functions-overview.md)、JavaScript 瀏覽器用戶端，以及在登入程序中需要更多彈性的 Web 應用程式。 它也適用於使用提供者 SDK 將使用者登入的原生行動應用程式。

> [!NOTE]
> 您可以使用伺服器導向流程來驗證 App Service 中受信任瀏覽器應用程式的呼叫對 App Service 或 [Azure Functions](../azure-functions/functions-overview.md) 中另一個 REST API 的呼叫。 如需詳細資訊，請參閱[自訂 App Service 中的驗證與授權](app-service-authentication-how-to.md)。
>

下表顯示驗證流程的步驟。

| 步驟 | 不使用提供者 SDK | 使用提供者 SDK |
| - | - | - |
| 1.將使用者登入 | 將用戶端重新導向至 `/.auth/login/<provider>`。 | 用戶端程式碼會直接使用提供者的 SDK 將使用者登入，並接收驗證權杖。 如需詳細資訊，請參閱提供者的文件。 |
| 2.後續驗證 | 提供者會將用戶端重新導向至 `/.auth/login/<provider>/callback`。 | 用戶端程式碼會將提供者的權杖張貼至 `/.auth/login/<provider>` 以進行驗證。 |
| 3.建立已驗證的工作階段 | App Service 會將已驗證的 Cookie 新增至回應。 | App Service 會將自己的驗證權杖傳回至用戶端程式碼。 |
| 4.提供已驗證的內容 | 用戶端會在後續要求中包含驗證 Cookie (瀏覽器會自動處理)。 | 用戶端程式碼會在 `X-ZUMO-AUTH` 標頭中顯示驗證權杖 (Mobile Apps 用戶端 SDK 會自動處理)。 |

對於用戶端瀏覽器，App Service 可以自動將所有未經驗證的使用者導向至 `/.auth/login/<provider>`。 您也可以向使用者顯示一或多個 `/.auth/login/<provider>` 連結，讓其使用選擇的提供者登入您的應用程式。

<a name="authorization"></a>

## <a name="authorization-behavior"></a>授權行為

在 [Azure 入口網站](https://portal.azure.com)中，您可以使用一些行為來設定 App Service 授權。

![](media/app-service-authentication-overview/authorization-flow.png)

下列標題會說明可用選項。

### <a name="allow-all-requests-default"></a>允許所有要求 (預設值)

驗證和授權未受 App Service 管理 (關閉)。 

如果您不需要驗證和授權，或如果您要撰寫自己的驗證和授權程式碼，請選擇此選項。

### <a name="allow-only-authenticated-requests"></a>僅允許已驗證的要求

選項為 [使用 \<提供者> 登入]。 App Service 會將所有匿名要求重新導向至您所選提供者的 `/.auth/login/<provider>`。 如果匿名要求來自原生行動應用程式，則傳回的回應是 `HTTP 401 Unauthorized`。

使用此選項時，您不需要在應用程式中撰寫任何驗證程式碼。 您可以藉由檢查使用者的宣告來處理更精細的授權 (例如特定角色授權，請參閱[存取使用者宣告](app-service-authentication-how-to.md#access-user-claims))。

### <a name="allow-all-requests-but-validate-authenticated-requests"></a>允許所有要求，但要驗證已驗證的要求

選項為 [允許匿名要求]。 此選項會開啟 App Service 中的驗證和授權，但是會延遲對應用程式程式碼的授權決策。 對於已驗證的要求，App Service 也會在 HTTP 標頭中一起傳送驗證資訊。 

此選項會提供更大的彈性來處理匿名要求。 例如，它可讓您向使用者[顯示多個登入提供者](app-service-authentication-how-to.md#use-multiple-sign-in-providers)。 不過，您必須撰寫程式碼。 

## <a name="more-resources"></a>其他資源

[教學課程：在 Azure App Service 中端對端驗證和授權使用者 (Windows)](app-service-web-tutorial-auth-aad.md)  
[教學課程：在適用於 Linux 的 Azure App Service 中端對端驗證和授權使用者](containers/tutorial-auth-aad.md)  
[自訂 App Service 中的驗證與授權](app-service-authentication-how-to.md)

提供者專屬的使用說明指南：

* [如何設定 App 使用 Azure Active Directory 登入][AAD]
* [如何設定 App 使用 Facebook 登入][Facebook]
* [如何設定 App 使用 Google 登入][Google]
* [如何設定 App 使用 Microsoft 帳戶登入][MSA]
* [如何設定 App 使用 Twitter 登入][Twitter]
* [做法：針對應用程式使用自訂驗證][custom-auth]

[AAD]: app-service-mobile-how-to-configure-active-directory-authentication.md
[Facebook]: app-service-mobile-how-to-configure-facebook-authentication.md
[Google]: app-service-mobile-how-to-configure-google-authentication.md
[MSA]: app-service-mobile-how-to-configure-microsoft-authentication.md
[Twitter]: app-service-mobile-how-to-configure-twitter-authentication.md

[custom-auth]: ../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#custom-auth

[ADAL-Android]: ../app-service-mobile/app-service-mobile-android-how-to-use-client-library.md#adal
[ADAL-iOS]: ../app-service-mobile/app-service-mobile-ios-how-to-use-client-library.md#adal
[ADAL-dotnet]: ../app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library.md#adal
