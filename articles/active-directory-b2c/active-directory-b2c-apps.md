---
title: 可以在 Azure Active Directory B2C 中使用的應用程式類型 | Microsoft Docs
description: 了解您可以在 Azure Active Directory B2C 中使用的應用程式類型。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/13/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e42bc63b0c2b6edf4dc0de204bbac5fe90071a67
ms.sourcegitcommit: fc5555a0250e3ef4914b077e017d30185b4a27e6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39480507"
---
# <a name="applications-types-that-can-be-used-in-active-directory-b2c"></a>可以在 Active Directory B2C 中使用的應用程式類型

Azure Active Directory (Azure AD) B2C 支援各種現代應用程式架構的驗證。 全部都以業界標準通訊協定 [OAuth 2.0](active-directory-b2c-reference-protocols.md) 或 [OpenID Connect](active-directory-b2c-reference-protocols.md) 為基礎。 本文件將說明您可以建置的應用程式類型，但不涉及您慣用的語言或平台。 在您開始建置應用程式之前，也可協助您先了解一些高階案例。

每個使用 Azure AD B2C 的應用程式都必須使用 [Azure 入口網站](https://portal.azure.com/)，在 [Azure AD B2C 租用戶](active-directory-b2c-get-started.md)中註冊。 應用程式註冊程序會收集和指派值，例如：

* 可唯一識別應用程式的**應用程式識別碼**。
* 可用來將回應導回應用程式的**重新導向 URI**。

每個傳送至 Azure AD B2C 的要求都指定一個 **原則**。 原則控制 Azure AD 的行為。 您也可以使用這些端點來建立一組可靈活自訂的使用者體驗。 常見的原則包括註冊、登入和設定檔編輯原則。 如果您不熟悉原則，在繼續之前，請參閱 Azure AD B2C 的 [可延伸原則架構](active-directory-b2c-reference-policies.md) 。

每個應用程式的互動都遵循類似的高階模式：

1. 應用程式會將使用者導向至 v2.0 端點以執行[原則](active-directory-b2c-reference-policies.md)。
2. 使用者完成根據原則定義來完成原則。
3. 應用程式會接收來自 v2.0 端點的安全性權杖。
4. 應用程式會使用此安全性權杖，以存取受保護的資訊或受保護的資源。
5. 資源伺服器會驗證安全性權杖，以確認可以授與存取權。
6. 應用程式定期重新整理安全性權杖。

根據您所建置的應用程式類型，這些步驟可能稍有不同。

## <a name="web-applications"></a>Web 應用程式

對於裝載於伺服器且透過瀏覽器存取的 Web 應用程式 (包括 .NET、PHP、Java、Ruby、Python 及 Node.js)，Azure AD B2C 支援使用 [OpenID Connect](active-directory-b2c-reference-protocols.md) 的所有使用者體驗。 這包括登入、註冊和設定檔管理。 在 OpenID Connect 的 Azure AD B2C 實作中，Web 應用程式會向 Azure AD 發出驗證要求，以起始這些使用者體驗。 要求的結果是 `id_token`。 這個安全性權杖代表使用者的身分識別。 它也以宣告形式提供使用者的相關資訊：

```
// Partial raw id_token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded id_token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)，以深入了解應用程式可用的權杖和宣告類型。

在 Web 應用程式中，每次執行[原則](active-directory-b2c-reference-policies.md)時均會採用下列高階步驟：

1. 使用者瀏覽至 web 應用程式。
2. Web 應用程式將使用者重新導向至 Azure AD B2C，以指出要執行的原則。
3. 使用者完成原則。
4. Azure AD B2C 會將 `id_token` 傳回給瀏覽器。
5. `id_token` 會張貼至重新導向 URI。
6. `id_token` 已經過驗證並已設定工作階段 cookie。
7. 系統會將安全的頁面傳回給使用者。

使用接收自 Azure AD 的公開簽署金鑰來驗證 `id_token` ，就足以驗證使用者的身分識別。 這也會設定工作階段 Cookie，在後續頁面要求上可用來識別使用者。

若要查看此案例的實際運作情形，請在[使用者入門](active-directory-b2c-overview.md)一節的 Web 應用程式登入程式碼範例中擇一試用。

除了讓登入更簡單，Web 伺服器應用程式可能也需要存取後端 Web 服務。 在此情況下，Web 應用程式可執行稍有不同的 [OpenID Connect 流程](active-directory-b2c-reference-oidc.md)，並使用授權碼和重新整理權杖來取得權杖。 以下 [Web API](#web-apis)一節描述此案例。

## <a name="web-apis"></a>Web API

您可以使用 Azure AD B2C 來保護 Web 服務，例如應用程式的 RESTful Web API。 Web API 可以使用 OAuth 2.0 保護其資料，並使用權杖來驗證傳入的 HTTP 要求。 Web API 的呼叫端會在 HTTP 要求的授權標頭中附加權杖：

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

然後，Web API 會使用這個權杖來驗證 API 呼叫端的身分識別，並從編碼在權杖中的宣告擷取呼叫端的相關資訊。 請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)，以深入了解應用程式可用的權杖和宣告類型。

> [!NOTE]
> Azure AD B2C 目前僅支援以各自已知的用戶端存取的 Web API。 例如，您的完整應用程式可能包括 iOS 應用程式、Android 應用程式和後端 Web API。 完全支援這種架構。 目前不支援允許夥伴用戶端 (例如另一個 iOS 應用程式) 存取相同的 Web API。 完整應用程式的所有元件都必須共用單一應用程式識別碼。
>
>

Web API 接收的權杖可以來自許多類型的用戶端，包括 Web 應用程式、桌面和行動應用程式、單一頁面應用程式、伺服器端精靈，以及其他 Web API。 以下是 Web 應用程式呼叫 Web API 的完整流程範例：

1. Web 應用程式會執行原則，而且使用者完成了使用者體驗。
2. Azure AD B2C 會將 `access_token` 和授權碼傳回給瀏覽器。
3. 瀏覽器會將 `access_token` 和授權碼張貼到重新導向 URI。
4. Web 伺服器會驗證 `access token` 並設定工作階段 cookie。
5. 系統會將 `access_token` 提供給 Azure AD B2C，其中包含授權碼、應用程式用戶端識別碼和認證。
6. 系統會將 `access_token` 和 `refresh_token` 傳回給 Web 伺服器。
7. 使用授權標頭中的 `access_token` 可呼叫 Web API。
8. Web API 會驗證此權杖。
9. 系統會將安全的資料傳回給 Web 伺服器。

若要深入了解授權碼、重新整理權杖和取得權杖的步驟，請參閱 [OAuth 2.0 通訊協定](active-directory-b2c-reference-oauth-code.md)。

若要了解如何使用 Azure AD B2C 保護 Web API，請查看 [使用者入門](active-directory-b2c-overview.md)一節的 Web API 教學課程。

## <a name="mobile-and-native-applications"></a>行動與原生應用程式

安裝在裝置上的應用程式 (例如行動和桌面應用程式) 通常需要代表使用者來存取後端服務或 Web API。 您可以將自訂的身分識別管理體驗新增至原生應用程式，並使用 Azure AD B2C 和 [OAuth 2.0 授權碼流程](active-directory-b2c-reference-oauth-code.md)以安全地呼叫後端服務。  

在此流程中，應用程式會執行[原則](active-directory-b2c-reference-policies.md)，並在使用者完成原則之後，從 Azure AD 接收 `authorization_code`。 `authorization_code` 代表應用程式有權代表目前登入的使用者來呼叫後端服務。 然後應用程式就可在背景中以 `authorization_code` 交換 `id_token` 和 `refresh_token`。  應用程式可以在 HTTP 要求中使用 `id_token` 來向後端 Web API 進行驗證。 它也可以使用 `refresh_token` 來取得新的 `id_token` (當舊的已過期時)。

> [!NOTE]
> Azure AD B2C 目前僅支援用來存取應用程式本身後端 Web 服務的權杖。 例如，您的完整應用程式可能包括 iOS 應用程式、Android 應用程式和後端 Web API。 完全支援這種架構。 目前不支援允許 iOS 應用程式使用 OAuth 2.0 存取權杖來存取夥伴 Web API。 完整應用程式的所有元件都必須共用單一應用程式識別碼。
>
>

## <a name="current-limitations"></a>目前的限制

Azure AD B2C 目前不支援下列類型的應用程式，但正在規劃中。 

### <a name="daemonsserver-side-applications"></a>精靈/伺服器端應用程式

如果應用程式含有長時間執行的處理序或不需要使用者操作，也仍然需要有存取受保護資源的方法，例如 Web API。 這些應用程式可以使用應用程式的身分識別 (而非使用者的委派身分識別)，以及使用 OAuth 2.0 用戶端認證流程，以驗證及取得權杖。 用戶端認證流程與代理流程不同，代理流程不應用於伺服器對伺服器的驗證。

雖然 Azure AD B2C 目前不支援用戶端認證流程，但您可以使用 Azure AD 來設定用戶端認證流程。 Azure AD B2C 租用戶會與 Azure AD 企業租用戶共用某些功能。  使用 Azure AD B2C 租用戶的 Azure AD 功能也可支援用戶端認證流程。 

若要設定用戶端認證流程，請參閱 [Azure Active Directory v2.0 和 OAuth 2.0 用戶端認證流程](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds)。 成功驗證後會收到已格式化的權杖，如此就可供 Azure AD 使用，如 [Azure AD 權杖參考](https://docs.microsoft.com/azure/active-directory/develop/active-directory-token-and-claims)中所述。


### <a name="web-api-chains-on-behalf-of-flow"></a>Web API 鏈結 (代理者流程)

許多架構中都有一個 Web API 需要呼叫另一個下游 Web API，而兩者都受 Azure AD B2C 保護。 此案例常見於有 Web API 後端的原生用戶端。 這接著會呼叫 Microsoft 線上服務，例如 Azure AD Graph API。

使用 OAuth 2.0 JWT 持有人認證授與可支援此鏈結的 Web API 案例，亦稱為代理者流程。  不過，Azure AD B2C 目前未實作代理者流程。
