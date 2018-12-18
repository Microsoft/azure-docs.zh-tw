---
title: 了解如何提供選擇性宣告給 Azure AD 應用程式 | Microsoft Docs
description: 將自訂或額外宣告新增至 Azure Active Directory 所簽發之 SAML 2.0 和 JSON Web Token (JWT) 權杖的指南。
documentationcenter: na
author: CelesteDG
services: active-directory
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/12/2018
ms.author: celested
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: d924c1fc9697bff77f12f7f0bf33a1654d1e7d6e
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39597968"
---
# <a name="optional-claims-in-azure-ad-preview"></a>Azure AD 中的選擇性宣告 (預覽)

應用程式開發人員可以使用此功能來指定要包含在傳送給其應用程式之權杖中的宣告。 您可以使用選擇性宣告來：
-   選取要包含在應用程式之權杖中的額外宣告。
-   變更 Azure AD 在權杖中傳回之特定宣告的行為。
-   新增和存取應用程式的自訂宣告。 

> [!Note]
> 這項功能目前為公開預覽版。 您應做好將任何變更還原或移除的準備。 在公開預覽版期間，所有 Azure AD 訂用帳戶中都有提供此功能。 不過，當此功能變成正式運作版時，可能需要 Azure AD Premium 訂用帳戶，才能使用此功能的某些層面。

如需標準宣告的清單及其在權杖中的使用方式，請參閱 [Azure AD 所簽發權杖的基本概念](v1-id-and-access-tokens.md)。 

[v2.0 Azure AD 端點](active-directory-appmodel-v2-overview.md)的其中一個目標是縮小權杖大小，以確保用戶端獲得最佳效能。  因此，數個先前包含在存取和識別碼權杖中的宣告在 v2.0 權杖中已不再提供，而必須依據個別應用程式明確提出要求才會提供。  

**表 1：適用性**

| 帳戶類型 | V1.0 端點                      | V2.0 端點  |
|--------------|------------------------------------|----------------|
| 個人 Microsoft 帳戶  | N/A - 改用 RPS 票證 | 即將支援 |
| Azure AD 帳戶          | 支援                          | 支援      |

## <a name="standard-optional-claims-set"></a>標準選擇性宣告集
以下列出預設可供應用程式使用的一組選擇性宣告。  若要為您的應用程式新增自訂選擇性宣告，請參閱下方的[目錄延伸模組](active-directory-optional-claims.md#Configuring-custom-claims-via-directory-extensions)。 

> [!Note]
>這些宣告中大多數都可包含在 v1.0 和 v2.0 權杖的 JWT 中，但不可包含在 SAML 權杖中 (「權杖類型」欄中已註明者除外)。  此外，雖然目前僅針對 AAD 使用者支援選擇性宣告，但已正在新增 MSA 支援。  當 MSA 在 v2.0 端點上具有選擇性宣告支援時，「使用者類型」欄會指出宣告可供 AAD 還是 MSA 使用者使用。  

**表 2：標準選擇性宣告集**

| Name                        | 說明   | 權杖類型 | 使用者類型 | 注意  |
|-----------------------------|----------------|------------|-----------|--------|
| `auth_time`                | 上次驗證使用者的時間。  請參閱 OpenID Connect 規格。| JWT        |           |  |
| `tenant_region_scope`      | 資源租用戶的區域 | JWT        |           | |
| `signin_state`             | 登入狀態宣告   | JWT        |           | 6 個以旗標形式表示的傳回值：<br> "dvc_mngd"：裝置為受控裝置<br> "dvc_cmp"：裝置符合規範<br> "dvc_dmjd"：裝置已加入網域<br> "dvc_mngd_app"：裝置是透過 MDM 管理的裝置<br> "inknownntwk"：裝置位於已知的網路內。<br> "kmsi"：已使用 [讓我保持登入]。 <br> |
| `controls`                 | 包含「條件式存取」原則所強制執行之工作階段控制項的多值宣告。  | JWT        |           | 3 個值：<br> "app_res"：應用程式必須強制執行更細微的限制。 <br> "ca_enf"：「條件式存取」強制執行已延遲但仍需要。 <br> "no_cookie"：此權杖不足以在瀏覽器中交換 Cookie。 <br>  |
| `home_oid`                 | 就來賓使用者而言，是使用者主租用戶中的使用者物件識別碼。| JWT        |           | |
| `sid`                      | 工作階段識別碼，用於個別工作階段使用者登出。 | JWT        |           |         |
| `platf`                    | 裝置平台    | JWT        |           | 限制為可驗證裝置類型的受控裝置。|
| `verified_primary_email`   | 以使用者的 PrimaryAuthoritativeEmail 為來源      | JWT        |           |         |
| `verified_secondary_email` | 以使用者的 SecondaryAuthoritativeEmail 為來源   | JWT        |           |        |
| `enfpolids`                | 強制執行的原則識別碼。 已針對目前使用者評估的原則識別碼清單。  | JWT |  |  |
| `vnet`                     | VNET 規範資訊。    | JWT        |           |      |
| `fwd`                      | IP 位址。| JWT    |   | 新增發出要求之用戶端的原始 IPv4 位址 (位於 VNET 內部時) |
| `ctry`                     | 使用者的國家/地區 | JWT |           | 如果出現且宣告值是兩個字母的國家/地區碼 (例如 FR、JP、SZ 等等)，則 Azure AD 會傳回 `ctry` 選擇性宣告。 |
| `tenant_ctry`              | 資源租用戶的國家/地區 | JWT | | |
| `xms_pdl`          | 慣用資料位置   | JWT | | 針對多地理位置租用戶，這是 3 個字母的代碼，用以顯示使用者位於哪一個地理區域。  如需詳細資訊，請參閱[關於慣用資料位置的 Azure AD Connect 文件](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-feature-preferreddatalocation)。 <br> 例如：`APC` 是指亞太地區。 |
| `xms_pl`                   | 使用者慣用語言  | JWT ||如果設定，則為使用者的慣用語言。  在來賓存取案例中，來源是其主租用戶。  格式化 LL-CC (“en-us”)。 |
| `xms_tpl`                  | 租用戶慣用語言| JWT | | 如果設定，則為資源租用戶的慣用語言。  格式化 LL (“en”)。 |
| `ztdid`                    | 全自動部署識別碼 | JWT | | 裝置身分識別，用於 [Windows AutoPilot](https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-10-autopilot) |
| `acct`             | 租用戶中的使用者帳戶狀態。   | JWT、SAML | | 如果使用者是租用戶的成員，則值為 `0`。  如果是來賓使用者，則值為 `1`。  |
| `upn`                      | UserPrincipalName 宣告。  | JWT、SAML  |           | 雖然會自動包含此宣告，但在來賓使用者案例中，您可以將它指定為選擇性宣告來附加額外屬性，以修改其行為。  <br> 額外屬性： <br> `include_externally_authenticated_upn` <br> `include_externally_authenticated_upn_without_hash` |

### <a name="v20-optional-claims"></a>V2.0 選擇性宣告
在 v1.0 權杖中一律會包含這些宣告，但在 v2.0 權杖中除非提出要求，否則不會包含。  這些宣告僅適用於 JWT (識別碼權杖和存取權杖)。  

**表 3：僅適用於 V2.0 的選擇性宣告**

| JWT 宣告     | Name                            | 說明                                                                                                                    | 注意 |
|---------------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------|-------|
| `ipaddr`      | IP 位址                      | 用戶端的登入來源 IP 位址。                                                                                      |       |
| `onprem_sid`  | 內部部署安全性識別碼 |                                                                                                                                |       |
| `pwd_exp`     | 密碼到期時間        | 密碼到時的日期時間。                                                                                    |       |
| `pwd_url`     | 變更密碼 URL             | 使用者可以瀏覽來變更其密碼的 URL。                                                                        |       |
| `in_corp`     | 公司網路內部        | 指出用戶端是否是從公司網路登入的。 如果不是，就不會包含此宣告                     |       |
| `nickname`    | 暱稱                        | 使用者的額外名稱 (有別於名字或姓氏)。                                                             |       |                                                                                                                |       |
| `family_name` | 姓氏                       | 提供使用者的姓氏 (如 Azure AD 使用者物件中所定義)。 <br>"family_name":"Miller" |       |
| `given_name`  | 名字                      | 提供使用者的名字 (如 Azure AD 使用者物件上所設定)。<br>"given_name": "Frank"                   |       |

### <a name="additional-properties-of-optional-claims"></a>選擇性宣告的額外屬性

有些選擇性宣告可經由設定來變更傳回宣告的方式。  這些額外的屬性大多數用來協助移轉對資料有不同要求的內部部署應用程式 (例如 `include_externally_authenticated_upn_without_hash` 可協助無法處理 UPN 中井號 (`#`) 的用戶端)

**表 4：用來設定標準選擇性宣告的值**

| 屬性名稱                                     | 額外屬性名稱                                                                                                             | 說明 |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|-------------|
| `upn`                                                 |                                                                                                                                      |  可用於 SAML 和 JWT 回應。            |
| | `include_externally_authenticated_upn`              | 包含儲存在資源租用戶中的來賓 UPN。  例如， `foo_hometenant.com#EXT#@resourcetenant.com`                            |             
| | `include_externally_authenticated_upn_without_hash` | 同上，除了將井號 (`#`) 取代成底線 (`_`) 之外，例如 `foo_hometenant.com_EXT_@resourcetenant.com` |             

> [!Note]
>如果指定 UPN 選擇性宣告但未指定額外屬性，將不會變更任何行為 – 為了在權杖中看到簽發新屬性，必須至少新增其中一個額外屬性。 


#### <a name="additional-properties-example"></a>額外屬性範例：

```json
 "optionalClaims": 
   {
       "idToken": [ 
             { 
                "name": "upn", 
            "essential": false,
                "additionalProperties": [ "include_externally_authenticated_upn"]  
              }
        ]
}
```

這個 OptionalClaims 物件會導致傳回給用戶端的識別碼權杖包含另一個 UPN，該 UPN 含有額外的主租用戶和資源租用戶資訊。  如果使用者是租用戶中的來賓 (使用不同的 IDP 進行驗證)，則這樣只會變更權杖中的 `upn` 宣告。 

## <a name="configuring-optional-claims"></a>設定選擇性宣告

您可以藉由修改應用程式資訊清單，來為應用程式設定選擇性宣告 (請參閱以下範例)。 如需詳細資訊，請參閱[了解 Azure AD 應用程式資訊清單](reference-app-manifest.md)一文。

**範例結構描述：**

```json
"optionalClaims":  
   {
       "idToken": [
             { 
                   "name": "auth_time", 
                   "essential": false
              }
        ],
 "accessToken": [ 
             {
                    "name": "ipaddr", 
                    "essential": false
              }
        ],
"saml2Token": [ 
              { 
                    "name": "upn", 
                    "essential": true
               },
               { 
                    "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                    "source": "user", 
                    "essential": true
               }
       ]
   }
```

### <a name="optionalclaims-type"></a>OptionalClaims 類型

宣告應用程式所要求的選擇性宣告。 應用程式可以針對它可從安全性權杖服務接收的三種權杖 (識別碼權杖、存取權杖、SAML 2 權杖)，設定要在這每一種權杖中傳回的選擇性宣告。 應用程式可以設定一組要在每個權杖類型中傳回的不同選擇性宣告。 「應用程式」實體的 OptionalClaims 屬性是一個 OptionalClaims 物件。

**表 5：OptionalClaims 類型屬性**

| Name        | 類型                       | 說明                                           |
|-------------|----------------------------|-------------------------------------------------------|
| `idToken`     | 集合 (OptionalClaim) | 在 JWT 識別碼權杖中傳回的選擇性宣告。     |
| `accessToken` | 集合 (OptionalClaim) | 在 JWT 存取權杖中傳回的選擇性宣告。 |
| `saml2Token`  | 集合 (OptionalClaim) | 在 SAML 權杖中傳回的選擇性宣告。       |


### <a name="optionalclaim-type"></a>OptionalClaim 類型

包含與應用程式或服務主體關聯的選擇性宣告。 [OptionalClaims](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#optionalclaims-type) 類型的 idToken、accessToken 及 saml2Token 屬性是 OptionalClaim 的集合。
如果特定宣告可支援，您也可以使用 AdditionalProperties 欄位來修改 OptionalClaim 的行為。

**表 6：OptionalClaim 類型屬性**

| Name                 | 類型                    | 說明                                                                                                                                                                                                                                                                                                   |
|----------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                 | Edm.String              | 選擇性宣告的名稱。                                                                                                                                                                                                                                                                               |
| `source`               | Edm.String              | 宣告的來源 (目錄物件)。 有來自延伸模組屬性的預先定義宣告和使用者定義宣告。 如果來源值為 null，宣告便是預先定義的選擇性宣告。 如果來源值為 user，名稱屬性中的值即為來自使用者物件的延伸模組屬性。 |
| `essential`            | Edm.Boolean             | 如果值為 true，就必須要有用戶端指定的宣告，才能確保使用者所要求之特定工作的授權體驗順暢。 預設值為 False。                                                                                                                 |
| `additionalProperties` | 集合 (Edm.String) | 宣告的額外屬性。 如果屬性存在於此集合中，它就會修改名稱屬性中所指定之選擇性宣告的行為。                                                                                                                                                   |

## <a name="configuring-custom-claims-via-directory-extensions"></a>透過目錄延伸模組設定自訂宣告

除了標準選擇性宣告集之外，您也可以設定讓權杖包含目錄結構描述延伸模組 (如需詳細資訊，請參閱[目錄結構描述延伸模組](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions) \(機器翻譯\) 一文)。  此功能可用來附加應用程式可使用的額外使用者資訊 – 例如，使用者已設定的額外識別碼或重要設定選項。 

> [!Note]
> 目錄結構描述延伸模組是僅適用於 AAD 的功能，因此如果您的應用程式資訊清單要求某個自訂延伸模組，而登入您應用程式的是 MSA 使用者，系統就不會傳回這些延伸模組。 

### <a name="values-for-configuring-additional-optional-claims"></a>用來設定額外選擇性宣告的值 

針對延伸模組屬性，請在應用程式資訊清單中使用延伸模組的全名 (格式為：`extension_<appid>_<attributename>`)。 `<appid>` 必須符合要求宣告之應用程式的識別碼。 

在 JWT 內，這些宣告將會以下列名稱格式發出：`extn.<attributename>`。

在 SAML 權杖內，這些宣告則會以下列 URI 格式發出：`http://schemas.microsoft.com/identity/claims/extn.<attributename>`

## <a name="optional-claims-example"></a>選擇性宣告範例

在本節中，您可以逐步進行案例，以了解如何針對應用程式使用選擇性宣告功能。
有多個選項可用來更新應用程式身分識別設定上的屬性，以啟用和設定選擇性宣告：
-   您可以修改應用程式資訊清單。 以下範例將使用此方法來執行設定。 請先閱讀介紹資訊清單的[了解 Azure AD 應用程式資訊清單](https://docs.microsoft.com/azure/active-directory/develop/active-directory-application-manifest)文件。
-   您也可以撰寫使用[圖形 API](https://docs.microsoft.com/azure/active-directory/develop/active-directory-graph-api)來更新您應用程式的應用程式。 「圖形 API」參考指南中的[實體和複雜類型參考](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#optionalclaims-type) \(英文\) 可協助您設定選擇性宣告。

**範例：** 在以下範例中，您將修改應用程式的資訊清單，以將宣告新增至要用於應用程式的存取權杖、識別碼權杖及 SAML 權杖中。
1.  登入 [Azure 入口網站](https://portal.azure.com)。
2.  驗證後，在頁面右上角選取您的 Azure AD 租用戶。
3.  從左側導覽面板中選取 [Azure AD 延伸模組]，然後按一下 [應用程式註冊]。
4.  從清單中找出您要為其設定選擇性宣告的應用程式，然後按一下該應用程式。
5.  按一下應用程式頁面中的 [資訊清單]，以開啟內嵌資訊清單編輯器。 
6.  您可以使用此編輯器直接編輯資訊清單。 資訊清單會依循[應用程式實體](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity)的結構描述，並在儲存後會自動設定資訊清單的格式。 新元素會新增至 `OptionalClaims` 屬性。

      ```json
      "optionalClaims": 
      {
            "idToken": [ 
                  { 
                        "name": "upn", 
                        "essential": false, 
                        "additionalProperties": [ "include_externally_authenticated_upn"]  
                  }
            ],
      "accessToken": [ 
                  {
                        "name": "auth_time", 
                        "essential": false
                  }
            ],
      "saml2Token": [ 
                  { 
                        "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                        "source": "user", 
                        "essential": true
                  }
            ]
      }
      ```
      在此案例中，已將不同的選擇性宣告新增至應用程式可接收的每一種權杖。 識別碼權杖現在將會在完整格式 (`<upn>_<homedomain>#EXT#@<resourcedomain>`) 中包含同盟使用者的 UPN。 存取權杖現在將會接收 auth_time 宣告。 SAML 權杖現在將會包含 skypeId 目錄結構描述延伸模組 (在此範例中，此應用程式的應用程式識別碼是 ab603c56068041afb2f6832e2a17e237)。  SAML 權杖會將 Skype 識別碼公開為 `extension_skypeId`。

7.  完成資訊清單更新時，按一下 [儲存] 以儲存資訊清單


## <a name="related-content"></a>相關內容
* 深入了解 Azure AD 所提供的[標準宣告](v1-id-and-access-tokens.md)。 
