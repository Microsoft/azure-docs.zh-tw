---
title: Azure Active Directory B2C 中開發人員的自訂原則使用注意事項 | Microsoft Docs
description: 開發人員在使用自訂原則來設定和維護 Azure AD B2C 時的注意事項。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/13/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 31ca79976ec2ecf2755b975749e9c9150c4a49e8
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441068"
---
# <a name="release-notes-for-azure-active-directory-b2c-custom-policy-public-preview"></a>Azure Active Directory B2C 自訂原則公開預覽版的版本資訊
所有 Azure Active Directory B2C (Azure AD B2C) 客戶現在已可評估自訂原則功能集 (公開預覽狀態)。 此功能集的適用對象是正在建置最複雜身分識別解決方案的進階身分識別開發人員。  

目前，此功能集需要開發人員直接透過 XML 檔案編輯來設定識別體驗架構。 用這種方式進行設定的效果最好，卻也複雜。 使用識別體驗架構的進階身分識別開發人員應該要規劃投入時間，以完成逐步解說課程和閱讀參考文件。 

## <a name="features-included-in-this-public-preview"></a>此公開預覽版包含的功能
透過公開預覽中所引進的新功能，開發人員可以執行下列工作：<br>

* 使用自訂原則來撰寫和上傳自訂驗證使用者旅程。 
   * 逐步將使用者旅程描述為宣告提供者之間的交換。 
   * 定義使用者旅程中的條件式分支。 
* 將啟用 REST API 功能的服務整合到您的自訂驗證使用者旅程。  
* 新增符合 OpenIDConnect 標準的識別提供者同盟。 <br>
* 新增遵守 SAML 2.0 通訊協定的識別提供者同盟。 

## <a name="terms-of-the-public-preview"></a>公開預覽版的條款

* 建議您只有在進行評估時才使用新功能。<br>
* 新功能不適合在生產環境中使用。<br>
* 服務等級協定 (SLA) 不適用於新功能。 <br>
* 您可以透過標準支援管道來提出支援要求。 <br>
* 正式運作的日期還未確定。<br>
* 若案例和使用者旅程超過 Azure AD B2C 產品可作為客戶身分識別與存取管理 (CIAM) 平台的許可範圍，Microsoft 會自行判斷 (以及因為任何原因) 而將其加上旗標並加以拒絕或施加限制。

## <a name="responsibilities-of-custom-policy-feature-set-developers"></a>自訂原則功能集開發人員的責任
手動設定原則會對 Azure AD B2C 基礎平台授與較低層級的存取權，並建立可完全自訂的唯一信任架構。 自訂識別提供者、信任關係、與外部服務的整合以及逐步工作流程會有各種可能的排列組合，因此使用這些組合的進階開發人員會遇到更嚴格的要求。

為了充分獲得公開預覽版的好處，建議使用自訂原則功能集的開發人員遵守下列方針：
* 熟悉識別體驗架構 (IEEE) 的設定語言和金鑰/密碼管理。
* 完整掌控案例和自訂整合。
* 有系統地執行案例測試。
* 對至少一個開發和測試環境以及一個生產環境遵循軟體開發和分段進行的最佳做法。
* 掌握與您整合之識別提供者和服務的最新開發資訊。 例如，追蹤密碼的變更，以及服務的排程和未排程變更。
* 設定主動式監控，並監視生產環境的回應能力。
* 在 Azure 訂用帳戶中保有最新的連絡人電子郵件地址，並持續回應 Microsoft 即時網站小組的電子郵件。
* 在 Microsoft 即時網站小組的建議下及時採取動作。 

## <a name="features-by-stage-and-known-issues"></a>依階段和已知問題的功能
自訂原則/識別體驗架構功能正在進行持續不斷且快速的開發。  此資料表是功能/元件可用性的索引。

將問題張貼在 Stack Overflow，網址為 [https://aka.ms/aadb2cso](https://aka.ms/aadb2cso)


### <a name="identity-providers-tokens-protocols"></a>識別提供者、權杖、通訊協定
具有外部元件和應用程式的介面

| 功能 | 開發 | 預覽 | GA | 注意 |
|---------------------------------------------|-------------|---------|----|-------|
| IDP-OpenIDConnect |  | x |  | 例如，Google+ |
| IDP-OAUTH2 |  | x |  | 例如，Facebook  |
| IDP-OAUTH1 |  | x |  | 例如，Twitter |
| IDP-SAML |  | x |  | 例如，Salesforce、ADFS |
| IDP-WSFED | x |  |  |  |
| 信賴憑證者 OAUTH |  | x |  |  |
| 信賴憑證者 OIDC |  | x |  |  |
| 信賴憑證者 SAML | x |  |  |  |
| 信賴憑證者 WSFED | x |  |  |  |
| 具有基本和憑證驗證的 REST API |  | x |  | 例如，Azure Functions |


### <a name="component-support"></a>元件支援


| 功能 | 開發 | 預覽 | GA | 注意 |
|-------------------------------------------|-------------|---------|----|-------|
| Azure Multi Factor Authentication |  | x |  |  |
| 作為本機目錄的 Azure Active Directory |  | x |  |  |
| 2FA 的 Azure 電子郵件子系統 |  | x |  |  |
| 多語言支援|  | x |  |  |
| 密碼複雜度 | x |  |  |  |


### <a name="content-definition"></a>內容定義

| 功能 | 開發 | 預覽 | GA | 注意 |
|-----------------------------------------------------------------------------|-------------|---------|----|-------|
|   錯誤頁面，api.error |  | x |  |  |
|   IDP 選取項目頁面，api.idpselections |  | x |  |  |
|   IDP 註冊選擇，api.idpselections.signup |  | x |  |  |
|   忘了密碼，api.localaccountpasswordreset |  | x |  |  |
|   本機帳戶登入，api.localaccountsignin |  | x |  |  |
|   本機帳戶登入，api.localaccountsignup |  | x |  |  |
|   MFA 頁面，api.phonefactor |  | x |  |  |
|   自我判斷 - 例如社交帳戶註冊，api.selfasserted |  | x |  |  |
|   自我判斷設定檔更新，api.selfasserted.profileupdate |  | x |  |  |
|   統一的註冊或登入頁面，api.signuporsignin |  | x |  |  |


### <a name="app-ief-integration"></a>App-IEF 整合
| 功能 | 開發 | 預覽 | GA | 注意 |
|--------------------------------------------------|-------------|---------|----|-------------------------------------------------|
| 查詢字串參數 id_token_hint | x |  |  |  |
| 查詢字串參數 domain_hint |  | x |  | 可作為宣告，可以傳遞至 IDP |
| 查詢字串參數 login_hint |  | x |  | 可作為宣告，可以傳遞至 IDP |
| 透過 client_assertion 將 JSON 插入至 UserJourney | x |  |  | 將被取代 |
| 將 JSON 插入至 UserJourney 作為 id_token_hint | x |  |  | 傳遞 JSON 的前進方法 |


### <a name="session-management"></a>工作階段管理

| 功能 | 開發 | 預覽 | GA | 注意 |
|---------------------------------|-------------|---------|----|-------|
| SSO 工作階段提供者 |  | x |  |  |
| 外部登入工作階段提供者 |  | x |  |  |
| SAML SSO 工作階段提供者 |  | x |  |  |


### <a name="security"></a>安全性
| 功能 | 開發 | 預覽 | GA | 注意 |
|---------------------------------------------|-------------|---------|----|-------|
| 原則機碼產生，手動上傳 |  | x |  |  |
| 原則機碼 - RSA/憑證，密碼 |  | x |  |  |


### <a name="developer-interface"></a>開發人員介面
| 功能 | 開發 | 預覽 | GA | 注意 |
|---------------------------------------------|-------------|---------|----|-------|
| Azure 入口網站 IEF UX |  | x |  |  |
| Application Insights UserJourney 記錄  |  | x |  |  |
| Application Insights 事件記錄 |x|  |  |  |



## <a name="next-steps"></a>後續步驟
[開始使用自訂原則](active-directory-b2c-get-started-custom.md)。
