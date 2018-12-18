---
title: 在 Azure Active Directory B2C 的自訂原則中定義 SAML 技術設定檔 | Microsoft Docs
description: 在 Azure Active Directory B2C 的自訂原則中定義 SAML 技術設定檔。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 301ae251413cc174f115479e9ebef2310aa83ba7
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162437"
---
# <a name="define-a-saml-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>在 Azure Active Directory B2C 自訂原則中定義 SAML 技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory (Azure AD) B2C 提供對 SAML 2.0 識別提供者的支援。 此文章說明技術設定檔的詳細規格，可用來與支援此標準化通訊協定的宣告提供者互動。 使用 SAML 技術設定檔，您可以與 SAML 式識別提供者 (例如 AD-FS 與 Salesforce) 同盟，讓您的使用者夠使用其現有的社交或企業識別來登入。

## <a name="metadata-exchange"></a>中繼資料交換

中繼資料是在 SAML 通訊協定中用來公開 SAML 合作對象 (例如，服務提供者或識別提供者) 設定的資訊。 中繼資料會定義服務的位置，例如，登入和登出、憑證、登入方法，以及其他項目。 識別提供者會使用中繼資料來了解如何與 Azure AD B2C 通訊。 中繼資料會以 XML 格式來設定，並且可能使用數位簽章來簽署，如此一來，另一個合作對象能夠驗證中繼資料的完整性。 當 Azure AD B2C 與 SAML 識別提供者同盟時，它會做為服務提供者來起始 SAML 要求，並等候 SAML 回應。 此外，在某些情況下 (不包括未經要求的 SAML 驗證)，也稱為識別提供者起始。 

在這兩個合作對象中，可將中繼資料設定為「靜態中繼資料」或「動態中繼資料」。 在靜態模式中，您會從一個合作對象中複製完整的中繼資料，並在另一個合作對象中設定它。 在動態模式中，您會將 URL 設定為中繼資料，而另一個合作對象會以動態方式讀取該設定。 準則都一樣，您會在識別提供者中設定 Azure AD B2C 技術設定檔的中繼資料，並在 Azure AD B2C 中設定識別提供者的中繼資料。

每個 SAML 識別提供者都會使用不同的步驟來公開和設定服務提供者 (在此案例中為 Azure AD B2C)，以及在識別提供者中設定 Azure AD B2C 中繼資料。 請查看您的識別提供者文件，以取得如何執行此操作的指導方針。

下列範例顯示 Azure AD B2C 技術設定檔 SAML 中繼資料的 URL 位址：

```
https://login.microsoftonline.com/te/your-tenant/your-policy/samlp/metadata?idptp=your-technical-profile
```

取代下列值：

- 將 **your-tenant** 取代為您的租用戶名稱，例如 your-tenant.onmicrosoft.com
- 將 **your-policy** 取代為您的原則名稱。 使用您用來設定 SAML 提供者技術設定檔的原則，或繼承自該原則的原則。
- 將 **your-technical-profile** 取代為您的 SAML 識別提供者技術設定檔名稱

## <a name="digital-signing-certificates-exchange"></a>數位簽署憑證交換

若要在 Azure AD B2C 與您的 SAML 識別提供者之間建置信任，您必須提供具備私密金鑰的有效 X509 憑證。 您要將具備私密金鑰的憑證 (.pfx 檔案) 上傳到 Azure AD B2C 原則金鑰存放區。 Azure AD B2C 會使用您提供的憑證，以數位方式簽署 SAML 登入要求。 

憑證的使用方式如下：

- Azure AD B2C 會使用憑證的 Azure AD B2C 私密金鑰，來產生並簽署 SAML 要求。 SAML 要求會傳送到識別提供者，其會使用憑證的 Azure AD B2C 公開金鑰來驗證要求。 Azure AD B2C 公開憑證可透過技術設定檔中繼資料來存取。 或者，您可以手動將 .cer 檔案上傳到您的 SAML 識別提供者。
- 識別提供者會使用識別提供者的憑證私密金鑰，來簽署傳送到 Azure AD B2C 的資料。 Azure AD B2C 會使用識別提供者的公開憑證來驗證資料。 每個識別提供者都會使用不同的步驟進行安裝，請查看您的識別提供者文件，以取得如何執行此操作的指導方針。 在 Azure AD B2C 中，您的原則需要使用識別提供者的中繼資料來存取憑證公開金鑰。

在大部分案例中都可接受自我簽署憑證。 針對生產環境，建議使用憑證授權單位所發出的 X509 憑證。 此外，如此文件稍後所述，針對非生產環境，您可以在這兩端停用 SAML 簽署。

下圖顯示中繼資料與憑證交換：

![中繼資料與憑證交換](media/saml-technical-profile/technical-profile-idp-saml-metadata.png)


## <a name="digital-encryption"></a>數位加密

為了加密 SAML 回應判斷提示，識別提供者一律會在 Azure AD B2C 技術設定檔中使用加密憑證的公開金鑰。 當 Azure AD B2C 需要將資料解密時，它會使用加密憑證的私密部分。

加密 SAML 回應判斷提示：

1. 將具備私密金鑰的有效 X509 憑證 (.pfx 檔案) 上傳到 Azure AD B2C 原則金鑰存放區。
2. 將識別碼為 `SamlAssertionDecryption` 的 **CryptographicKey** 元素新增至技術設定檔 **CryptographicKeys** 集合。 將 **StorageReferenceId** 設定為您在步驟 1 中建立的原則金鑰名稱。
3. 將技術設定檔中繼資料 **WantsEncryptedAssertions** 設定為 `true`。
4. 使用新的 Azure AD B2C 技術設定檔中繼資料來更新識別提供者。 您應該會看到已將 **use** 屬性設定為 `encryption` 的 **KeyDescriptor**，其中包含憑證的公開金鑰。

下列範例顯示中繼資料的 Azure AD B2C 技術設定檔加密區段：

```XML
<KeyDescriptor use="encryption">
  <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>valid certificate</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

## <a name="identity-provider-initiated-flow"></a>識別提供者起始的流程

在 IDP 所起始的單一登入工作階段 (未經要求的要求) 中，會將未經要求的 SAML 回應傳送到服務提供者，在此案例中為 Azure AD B2C 技術設定檔。 在此流程中，使用者不會先經過 Web 應用程式，但會導向至識別提供者。 傳送要求時，驗證頁面會由識別提供者提供給使用者。 使用者會完成登入，然後使用包含判斷提示的 SAML 回應來將要求重新導向至 Azure AD B2C。 Azure AD B2C 會讀取判斷提示並發出新的 SAML 權杖，然後將使用者重新導向回到信賴憑證者應用程式。 重新導向會透過 **AssertionConsumerService** 元素的 **Location** 屬性來完成。


![SAML IDP 起始](media/saml-technical-profile/technical-profile-idp-saml-idp-initiated.png) 

建立識別提供者起始的流程時，請考慮下列原則需求：

- 第一個協調流程步驟必須是指向 SAML 技術設定檔的單一宣告交換。
- 技術設定檔必須將名為 **IdpInitiatedProfileEnabled** 的中繼資料項目設定為 `true`。
- 信賴憑證者原則必須是 SAML 信賴憑證者。
- 信賴憑證者原則必須將名為 **IdpInitiatedProfileEnabled** 的中繼資料項目設定為 `true`。
- 未經要求的回應必須傳送到 `/your-tenant/your-policy/samlp/sso/assertionconsumer` 端點。 回應中所包含的任何轉送狀態都會轉送到信賴憑證者。 取代下列值：將 **your-tenant** 取代為您的租用戶名稱。 將 **your-policy** 取代為您的信賴憑證者原則名稱。
    
## <a name="protocol"></a>通訊協定

Protocol 元素的 **Name** 屬性必須設定為 `SAML2`。 

## <a name="metadata"></a>中繼資料

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| PartnerEntity | 是 | SAML 識別提供者的中繼資料 URL。 複製識別提供者中繼資料，並在 CDATA 元素 `<![CDATA[Your IDP metadata]]>` 內新增它 |
| WantsSignedRequests | 否 | 指出技術設定檔是否需要所有連出驗證要求都經過簽署。 可能的值：`true` 或 `false`。 預設值為 `true`。 將值設定為 `true` 時，**SamlMessageSigning** 密碼編譯金鑰必須經過簽署，而所有連出驗證要求也都要經過簽署。 如果將值設定為 `false`，則會略過要求中的 **SigAlg** 和 **Signature** 參數 (查詢字串或張貼參數)。 此中繼資料也會控制中繼資料 **AuthnRequestsSigned** 屬性，其為要與識別提供者共用之 Azure AD B2C 技術設定檔中繼資料內的輸出。 |
| XmlSignatureAlgorithm | 否 | Azure AD B2C 用來簽署 SAML 要求的方法。 此中繼資料會控制 SAML 要求中 **SigAlg** 參數 (查詢字串或張貼參數) 的值。 可能的值：`Sha256`、`Sha384`、`Sha512` 或 `Sha1`。 請確定您會使用相同的值來設定這兩端的簽章演算法。 僅使用您憑證支援的演算法。 | 
| WantsSignedAssertions | 否 | 指出技術設定檔是否需要所有傳入的判斷提示都要經過簽署。 可能的值：`true` 或 `false`。 預設值為 `true`。 如果將值設定為 `true`，由識別提供者傳送到 Azure AD B2C 的所有判斷提示區段 `saml:Assertion` 都必須經過簽署。 如果將值設定為 `false`，則識別提供者不應該簽署判斷提示，但即使是這樣，Azure AD B2C 還是不會驗證簽章。 此中繼資料也會控制中繼資料旗標 **WantsAssertionsSigned**，其為要與識別提供者共用之 Azure AD B2C 技術設定檔中繼資料內的輸出。 如果您停用判斷提示驗證，您可能也想要停用回應簽章驗證 (如需詳細資訊，請參閱 **ResponsesSigned**)。 |
| ResponsesSigned | 否 | 可能的值：`true` 或 `false`。 預設值為 `true`。 如果將值設定為 `false`，則識別提供者不應該簽署 SAML 回應，但即使是這樣，Azure AD B2C 還是不會驗證簽章。 如果將值設定為 `true`，由識別提供者傳送到 Azure AD B2C 的 SAML 回應會經過簽署且必須進行驗證。 如果您停用 SAML 回應驗證，您可能也想要停用判斷提示簽章驗證 (如需詳細資訊，請參閱 **WantsSignedAssertions**)。 |
| WantsEncryptedAssertions | 否 | 指出技術設定檔是否需要所有傳入的判斷提示都要加密。 可能的值：`true` 或 `false`。 預設值為 `false`。 如果將值設定為 `true`，由識別提供者傳送到 Azure AD B2C 的判斷提示都必須經過簽署，而且必須指定 **SamlAssertionDecryption** 密碼編譯金鑰。 如果將值設定為 `true`，Azure AD B2C 技術設定檔的中繼資料就會包含 [加密] 區段。 識別提供者會讀取中繼資料，並使用 Azure AD B2C 技術設定檔中繼資料內提供的公開金鑰來加密 SAML 回應判斷提示。 如果您啟用判斷提示加密，您可能也需要停用回應簽章驗證 (如需詳細資訊，請參閱 **ResponsesSigned**)。 | 
| IdpInitiatedProfileEnabled | 否 |指出由 SAML 識別提供者設定檔所起始的單一登入工作階段設定檔是否已啟用。 可能的值：`true` 或 `false`。 預設值為 `false`。 在識別提供者所起始的流程中，使用者會在外部進行驗證，而未經要求的回應會傳送到 Azure AD B2C，其接著會取用權杖、執行協調流程步驟，然後將回應傳送到信賴憑證者應用程式。 |

## <a name="cryptographic-keys"></a>密碼編譯金鑰

**CryptographicKeys** 元素包含下列屬性：

| 屬性 |必要 | 說明 |
| --------- | ----------- | ----------- |
| SamlMessageSigning |是 | 用來簽署 SAML 訊息的 X509 憑證 (RSA 金鑰組)。 Azure AD B2C 會使用此金鑰來簽署要求，並將它們傳送到識別提供者。 |
| SamlAssertionDecryption |是 | 用來將 SAML 訊息解密的 X509 憑證 (RSA 金鑰組)。 此憑證應該由識別提供者來提供。 Azure AD B2C 會使用此憑證來將識別提供者所傳送的資料解密。 |
| MetadataSigning |否 | 用來簽署 SAML 中繼資料的 X509 憑證 (RSA 金鑰組)。 Azure AD B2C 會使用此金鑰來簽署中繼資料。  |

## <a name="examples"></a>範例

- [使用自訂原則新增 ADFS 作為 SAML 識別提供者](active-directory-b2c-custom-setup-adfs2016-idp.md)
- [使用 Salesforce 帳戶透過 SAML 來登入](active-directory-b2c-setup-sf-app-custom.md)













