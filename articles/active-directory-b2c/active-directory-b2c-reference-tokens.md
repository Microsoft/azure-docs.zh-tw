---
title: Azure Active Directory B2C 中的權杖參考 | Microsoft Docs
description: 在 Azure Active Directory B2C 中簽發的權杖類型
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5ff4ddee3d8af15caf082be56a51b1aa0d36f02a
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43339972"
---
# <a name="azure-ad-b2c-token-reference"></a>Azure AD B2C：權杖參考

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) 會在處理每個[驗證流程](active-directory-b2c-apps.md)時發出數種安全性權杖。 本文件說明每種權杖的格式、安全性特性和內容。

## <a name="types-of-tokens"></a>權杖的類型
Azure AD B2C 支援 [OAuth 2.0 授權通訊協定](active-directory-b2c-reference-protocols.md)，該通訊協定會使用存取權杖與重新整理權杖。 它也支援透過 [OpenID Connect](active-directory-b2c-reference-protocols.md)進行驗證和登入，這引進第三種類型的權杖：ID 權杖。 每個權杖都是以持有人權杖表示。

持有人權杖是輕巧型的安全性權杖，授權「持有人」存取受保護的資源。 持有人是可出示權杖的任何一方。 Azure AD B2C 必須先驗證合作對象，才能接收持有人權杖。 但若傳輸和儲存時未採取必要的步驟來保護權杖，它可能會被非預期的一方攔截和使用。 某些安全性權杖有內建的機制，可防止未授權的合作對象使用它們，但持有人權杖沒有這項機制。 它們必須在安全的通道 (例如傳輸層安全性 (HTTPS)) 中傳輸。

如果持有人權杖是在安全通道外部進行傳輸，則惡意人士就能使用攔截式攻擊來取得權杖，未經授權地使用該權杖來存取受保護的資源。 儲存或快取持有人權杖供以後使用時，也適用相同的安全性原則。 務必確定您的應用程式以安全的方式傳輸和儲存持有人權杖。

如需持有人權杖的其他安全性考量，請參閱 [RFC 6750 第 5 節](http://tools.ietf.org/html/rfc6750)。

許多由 Azure AD B2C 所簽發的權杖都實作為 JSON Web 權杖 (JWT)。 JWT 是一種精簡的 URL 安全方法，可在兩方之間傳輸資訊。 JWT 包含稱為宣告的資訊。 這些是權杖持有人及主體相關資訊的判斷提示。 JWT 中的宣告是為了傳輸而編碼和序列化的 JSON 物件。 因為 Azure AD B2C 所簽發的 JWT 已簽署但未加密，所以您可以輕鬆地檢查 JWT 的內容以便偵錯。 有一些工具可以這樣做，包括 [jwt.ms](https://jwt.ms)。 如需 JWT 的詳細資訊，請參閱 [JWT 規格](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。

### <a name="id-tokens"></a>ID 權杖

ID 權杖是您的應用程式從 Azure AD B2C `/authorize` 和 `/token` 端點接收的一種安全性權杖。 ID 權杖以 [JWT](#types-of-tokens)表示，而且包含宣告讓您用來識別應用程式中的使用者。 從 `/authorize` 端點取得 ID 權杖時，它們已經使用[隱含流程](active-directory-b2c-reference-spa.md)來完成，這通常適用於登入以 javascript 作為基礎之 web 應用程式的使用者。 從 `/token` 端點取得 ID 權杖時，它們已經使用[機密程式碼流程](active-directory-b2c-reference-oidc.md)來完成，這會使權杖保持規避瀏覽器。 這樣可在 HTTP 要求中安全地傳送權杖，從而在相同應用程式或服務的兩個元件之間進行通訊。 您可以依需要在 ID 權杖中使用宣告。 它們通常用來顯示帳戶資訊，或決定應用程式中的存取控制。  

識別碼權杖已簽署，但目前未加密。 當您的應用程式或 API 收到 ID 權杖時，它必須 [驗證簽章](#token-validation) 以證明權杖的真實性。 您的應用程式或 API 也必須驗證權杖中的幾個宣告，以證明它有效。 應用程式所驗證的宣告會視案例需求而有所不同，但您的應用程式必須在每一種案例中執行一些 [常見的宣告驗證](#token-validation) 。

#### <a name="sample-id-token"></a>範例 ID 權杖
```
// Line breaks for display purposes only

eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IklkVG9rZW5TaWduaW5nS2V5Q29udGFpbmVyIn0.
eyJleHAiOjE0NDIzNjAwMzQsIm5iZiI6MTQ0MjM1NjQzNCwidmVyIjoiMS4wIiwiaXNzIjoiaHR0cHM6Ly9s
b2dpbi5taWNyb3NvZnRvbmxpbmUuY29tLzc3NTUyN2ZmLTlhMzctNDMwNy04YjNkLWNjMzExZjU4ZDkyNS92
Mi4wLyIsImFjciI6ImIyY18xX3NpZ25faW5fc3RvY2siLCJzdWIiOiJOb3Qgc3VwcG9ydGVkIGN1cnJlbnRs
eS4gVXNlIG9pZCBjbGFpbS4iLCJhdWQiOiI5MGMwZmU2My1iY2YyLTQ0ZDUtOGZiNy1iOGJiYzBiMjlkYzYi
LCJpYXQiOjE0NDIzNTY0MzQsImF1dGhfdGltZSI6MTQ0MjM1NjQzNCwiaWRwIjoiZmFjZWJvb2suY29tIn0.
h-uiKcrT882pSKUtWCpj-_3b3vPs3bOWsESAhPMrL-iIIacKc6_uZrWxaWvIYkLra5czBcGKWrYwrAC8ZvQe
DJWZ50WXQrZYODEW1OUwzaD_I1f1HE0c2uvaWdGXBpDEVdsD3ExKaFlKGjFR2V7F-fPThkVDdKmkUDQX3bVc
yyj2V2nlCQ9jd7aGnokTPfLfpOjuIrTsAdPcGpe5hfSEuwYDmqOJjGs9Jp1f-eSNEiCDQOaTBSvr479L5ptP
XWeQZyX2SypN05Rjr05bjZh3j70ZUimiocfJzjibeoDCaQTz907yAg91WYuFOrQxb-5BaUoR7K-O7vxr2M-_
CQhoFA

```

### <a name="access-tokens"></a>存取權杖

存取權杖也是您的應用程式從 Azure AD B2C `/authorize` 和 `/token` 端點接收的一種安全性權杖。 存取權杖也會以 [JWT](#types-of-tokens) 表示，而且其中包含宣告以供您用來識別對您 API 所授與的權限。 存取權杖已簽署，但目前未加密。 您應該使用存取權杖來提供伺服器 API 和資源的存取權。 深入了解如何[使用存取權杖](active-directory-b2c-access-tokens.md)。 

當您的 API 收到存取權杖時，它必須 [驗證簽章](#token-validation) 以證明權杖的真實性。 您的 API 也必須驗證權杖中的幾個宣告，以證明它有效。 應用程式所驗證的宣告會視案例需求而有所不同，但您的應用程式必須在每一種案例中執行一些 [常見的宣告驗證](#token-validation) 。

### <a name="claims-in-id-and-access-tokens"></a>識別碼和存取權杖中的宣告

當您使用 Azure AD B2C 時，您可以精確控制權杖的內容。 您可以設定 [原則](active-directory-b2c-reference-policies.md) ，以在宣告中傳送您的應用程式運作時所需的幾組特定的使用者資料。 這些宣告可以包含標準的屬性，例如使用者的 `displayName` 和 `emailAddress`。 其中也可以包含您在 B2C 目錄中可定義的 [自訂使用者屬性](active-directory-b2c-reference-custom-attr.md) 。 您收到的每個識別碼權杖和存取權杖都會包含一組特定的安全性相關宣告。 您的應用程式可以使用這些宣告來安全地驗證使用者和要求。

請注意，ID 權杖中的宣告不依任何特定順序傳回。 此外，隨時都可以在 ID 權杖中加入新的宣告。 加入新的宣告時，您的應用程式不會損壞。 以下是 Azure AD B2C 所簽發的識別碼權杖和存取權杖中應該會有的宣告。 其他任何宣告都由原則決定。 練習時，請試著將範例識別碼權杖中的宣告貼入 [jwt.ms](https://jwt.ms) 中進行檢查。 在 [OpenID Connect 規格](http://openid.net/specs/openid-connect-core-1_0.html)中可找到進一步的詳細資料。

| 名稱 | 宣告 | 範例值 | 說明 |
| --- | --- | --- | --- |
| 對象 |`aud` |`90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` |對象宣告識別權杖的預定接收者。 在 Azure AD B2C 中，對象是在應用程式註冊入口網站中指派給您應用程式的應用程式識別碼。 您的應用程式應驗證此值並拒絕不相符的權杖。 對象是資源的同義詞。 |
| 簽發者 |`iss` |`https://{tenantname}.b2clogin.com/775527ff-9a37-4307-8b3d-cc311f58d925/v2.0/` |此宣告會識別負責建構並傳回權杖的 Security Token Service (STS)。 它也會識別用於驗證使用者的 Azure AD 目錄。 您的應用程式應該驗證簽發者宣告，以確保權杖來自 Azure Active Directory v2.0 端點。 |
| 發出時間 |`iat` |`1438535543` |此宣告是簽發權杖的時間，以新紀元時間表示。 |
| 到期時間 |`exp` |`1438539443` |此到期時間宣告是權杖失效的時間，以新紀元時間表示。 您的應用程式應使用此宣告來驗證權杖存留期的有效性。 |
| 生效時間 |`nbf` |`1438535543` |此宣告是權杖生效的時間，以新紀元時間表示。 這通常與權杖的簽發時間相同。 您的應用程式應使用此宣告來驗證權杖存留期的有效性。 |
| 版本 |`ver` |`1.0` |這是 Azure AD 所定義的 ID 權杖版本。 |
| 代碼雜湊 |`c_hash` |`SGCPtt01wxwfgnYZy2VJtQ` |只有當 ID 權杖與 OAuth 2.0 授權碼一起簽發時，權杖才會包含代碼雜湊。 代碼雜湊可用來驗證授權碼的真實性。 如需如何執行此驗證的詳細資訊，請參閱 [OpenID Connect 規格](http://openid.net/specs/openid-connect-core-1_0.html) \(英文\)。  |
| 存取權杖雜湊 |`at_hash` |`SGCPtt01wxwfgnYZy2VJtQ` |只有當 ID 權杖與 OAuth 2.0 存取權杖一起簽發時，權杖才會包含存取權杖雜湊。 存取權杖雜湊可用來驗證存取權杖的真實性。 如需如何執行此驗證的詳細資訊，請參閱 [OpenID Connect 規格](http://openid.net/specs/openid-connect-core-1_0.html) \(英文\)。  |
| Nonce |`nonce` |`12345` |Nonce 是用來緩和權杖重新執行攻擊的策略。 您的應用程式可以使用 `nonce` 查詢參數，在授權要求中指定 Nonce。 您在要求中提供的值將只會在 ID 權杖的 `nonce` 宣告中發出 (未經修改)。 這可讓您的應用程式根據在要求上指定的值驗證此值，使應用程式的工作階段與給定的 ID 權杖產生關聯。 您的應用程式應在 ID 權杖驗證程序中執行這項驗證。 |
| 主體 |`sub` |`884408e1-2918-4cz0-b12d-3aa027d7563b` |這是權杖聲稱資訊時所針對的主體，例如應用程式的使用者。 這個值不可變，而且無法重新指派或重複使用。 它可用來安全地執行授權檢查，例如當權杖用於存取資源時。 根據預設，主體宣告會填入目錄中使用者的物件識別碼。 若要深入了解，請參閱 [Azure Active Directory B2C︰權杖、工作階段及單一登入設定](active-directory-b2c-token-session-sso.md)。 |
| 驗證內容類別參考 |`acr` |不適用 |目前未使用 (較舊的原則除外)。 若要深入了解，請參閱 [Azure Active Directory B2C︰權杖、工作階段及單一登入設定](active-directory-b2c-token-session-sso.md)。 |
| 信任架構原則 |`tfp` |`b2c_1_sign_in` |這是用來取得 ID 權杖的原則名稱。 |
| 驗證期間 |`auth_time` |`1438535543` |此宣告是使用者上次輸入認證的時間，以新紀元時間表示。 |

### <a name="refresh-tokens"></a>重新整理權杖
重新整理權杖是可供您的應用程式在 OAuth 2.0 流程中取得新的 ID 權杖和存取權杖的安全性權杖。 它們可讓您的應用程式代表使用者來長期存取資源，而不需與使用者互動。

若要在權杖回應中接收重新整理權杖，您的應用程式必須要求 `offline_acesss` 範圍。 若要深入了解 `offline_access` 範圍，請參閱 [Azure AD B2C 通訊協定參考](active-directory-b2c-reference-protocols.md)。

重新整理權杖永遠對您的應用程式完全不透明。 它們是由 Azure AD 所簽發，只能由 Azure AD 檢查和解譯。 它們屬於長效權杖，但在撰寫您的應用程式時，不應該期待重新整理權杖會持續一段特定時間。 重新整理權杖可能會因為各種原因而隨時失效。 讓您的應用程式知道重新整理權杖是否有效的唯一方式，就是對 Azure AD 提出權杖要求以嘗試兌換。

當您以重新整理權杖來兌換新的權杖時 (而且如果您的應用程式已獲得 `offline_access` 範圍)，您會在權杖回應中收到新的重新整理權杖。 您應儲存新簽發的重新整理權杖。 它應該取代您先前使用於要求中的重新整理權杖。 這有助於保證您的重新整理權杖盡可能長期保持有效。

## <a name="token-validation"></a>權杖驗證
若要驗證權杖，您的應用程式應該檢查權杖的簽章和宣告。

視您偏好的語言而定，有許多開放原始碼程式庫可用來驗證 JWT。 我們建議您探索這些選項，而不是實作您自己的驗證邏輯。 本指南中的資訊有助於您了解如何適當使用這些程式庫。

### <a name="validate-the-signature"></a>驗證簽章
JWT 包含三個區段 (以 `.` 字元分隔)。 第一個區段是*標頭*，第二個是*主體*，而第三個是*簽章*。 簽章區段可用來驗證權杖的真實性，您的應用程式才得以信任。

Azure AD B2C 權杖是經由業界標準非對稱式加密演算法 (例如 RSA 256) 進行簽署。 權杖標頭包含用來簽署權杖的金鑰和加密方法相關資訊：

```
{
        "typ": "JWT",
        "alg": "RS256",
        "kid": "GvnPApfWMdLRi8PDmisFn7bprKg"
}
```

`alg` 宣告指出用來簽署權杖的演算法。 `kid` 宣告指出用來簽署權杖的特定公開金鑰。

無論何時，Azure AD 都能使用任何一組特定的公開-私密金鑰組來簽署權杖。 Azure AD 會定期替換一組可能的金鑰，所以應將您的應用程式撰寫成自動處理這些金鑰變更。 檢查 Azure AD 所用公開金鑰的更新的合理頻率為每 24 小時。

Azure AD B2C 具有 OpenID Connect 中繼資料端點。 這可讓應用程式在執行階段擷取 Azure AD B2C 的相關資訊。 這項資訊包括端點、權杖內容和權杖簽署金鑰。 您的 B2C 目錄包含每個原則的 JSON 中繼資料文件。 例如，`fabrikamb2c.onmicrosoft.com` 中適用於 `b2c_1_sign_in` 原則的中繼資料文件位於：

```
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=b2c_1_sign_in
```

`fabrikamb2c.onmicrosoft.com` 是用來驗證使用者的 B2C 目錄，而 `b2c_1_sign_in` 是用來取得權杖的原則。 若要判斷哪個原則用來簽署權杖 (以及何處可擷取中繼資料)，您有兩個選項。 首先，原則名稱包含在權杖的 `acr` 宣告中。 您可以將 JWT 主體進行 base 64 解碼，並將產生的 JSON 字串還原序列化，以剖析 JWT 主體中的宣告。 `acr` 宣告會是用來簽發權杖的原則名稱。  另一個選項是當您發出要求時在 `state` 參數的值中將原則編碼，然後將它解碼以判斷使用了哪個原則。 任一種方法都有效。

中繼資料文件是包含幾項實用資訊的 JSON 物件。 其中包括執行 OpenID Connect 驗證時所需端點的位置。 它們還包含 `jwks_uri`，指出用來簽署權杖的公用金鑰組的位置。 這裡提供該位置，但最好使用中繼資料文件並剖析 `jwks_uri`來動態擷取該位置：

```
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/discovery/v2.0/keys?p=b2c_1_sign_in
```

位於此 URL 的 JSON 文件包含特定時刻使用的所有公開金鑰資訊。 您的應用程式可以使用 JWT 標頭中的 `kid` 宣告，以選取 JSON 文件中用來簽署特定權杖的公開金鑰。 接著可以使用正確的公開金鑰和指定的演算法來執行簽章驗證。

描述如何執行簽章驗證已超出本文的範圍。 如果需要的話，有許多開放原始碼程式庫可協助您處理這方面。

### <a name="validate-the-claims"></a>驗證宣告
當您的應用程式或 API 收到 ID 權杖時，還應該對 ID 權杖中的宣告執行幾項檢查。 包含但不限於：

* **對象** 宣告：這會確認 ID 權杖預定要提供給您的應用程式。
* **生效時間**和**到期時間**宣告：這些會確認 ID 權杖尚未過期。
* **簽發者** 宣告：這會確認權杖已由 Azure AD 簽發給您的應用程式。
* **Nonce**：這是用來緩和權杖重新執行攻擊的策略。

如需您應用程式應執行之驗證的完整清單，請參閱 [OpenID Connect 規格](https://openid.net) \(英文\)。 先前的 [權杖一節](#types-of-tokens)包含這些宣告的預期值的詳細資料。  

## <a name="token-lifetimes"></a>權杖存留期
下列權杖存留期讓您了解更透徹。 當您開發及偵錯應用程式時，它們可以協助您。 請注意，撰寫您的應用程式時，請不要認為任何存留期會維持不變。 它們會變更。 請閱讀更多在 Azure AD B2C 中[自訂權杖存留期](active-directory-b2c-token-session-sso.md)的相關資訊。

| token | 存留期 | 說明 |
| --- | --- | --- |
| ID 權杖 |一小時 |ID 權杖的有效期通常為 1 小時。 Web 應用程式可以使用此存留期來維持它自己與使用者之間的工作階段 (建議選項)。 您也可以選擇不同的工作階段存留期。 如果您的應用程式需要取得新的 ID 權杖，它只需要對 Azure AD 提出新的登入要求。 如果使用者與 Azure AD 之間存在有效的瀏覽器工作階段，該使用者可能不需要再次輸入認證。 |
| 重新整理權杖 |最多 14 天 |單一重新整理權杖的有效期最多 14 天。 不過，重新整理權杖可能會基於許多因素而隨時失效。 您的應用程式應繼續嘗試使用重新整理權杖，直到要求失敗，或您的應用程式更換新的重新整理權杖為止。 在使用者上次輸入認證之後經過 90 天，重新整理權杖也會失效。 |
| 授權碼 |五分鐘 |授權碼是刻意設計成短期。 它們應在收到時立即兌換成存取權杖、ID 權杖或重新整理權杖。 |

