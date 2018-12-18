---
title: Azure Active Directory B2C 自訂原則 | Microsoft Docs
description: 了解 Azure Active Directory B2C 自訂原則。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/04/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5634c14ee2b25600d66ff0f2c7385b2aaa9f1810
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699493"
---
# <a name="custom-policies-in-azure-active-directory-b2c"></a>Azure Active Directory B2C 中的自訂原則

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

自訂原則是定義 Azure Active Directory (Azure AD) B2C 租用戶行為的組態檔。 對於最常用的身分識別工作，Azure AD B2C 入口網站中預先定義了內建原則。 身分識別開發人員可以完全編輯自訂原則，以完成許多不同的工作。

## <a name="comparing-built-in-policies-and-custom-policies"></a>比較內建原則和自訂原則

| | 內建原則 | 自訂原則 |
|-|-------------------|-----------------|
| 目標使用者 | 所有具備或不具備身分識別專業知識的應用程式開發人員。 | 身分識別專業人員、系統整合人員、顧問和內部身分識別小組。 他們熟悉 OpenIDConnect 流程，也了解身分識別提供者和宣告型驗證。 |
| 設定方法 | 有使用者介面 (UI) 方便使用的 Azure 入口網站。 | 直接編輯 XML 檔案，然後上傳至 Azure 入口網站。 |
| UI 自訂 | 完整的 UI 自訂，包括 HTML 和 CSS。<br><br>使用自訂字串支援多語言。 | 相同 |
| 屬性自訂 | 標準和自訂屬性。 | 相同 |
| 權杖和工作階段管理 | 自訂權杖和多個工作階段選項。 | 相同 |
| 識別提供者 | 預先定義的本機或社交提供者。 | 標準式 OIDC、OAUTH 和 SAML。 |
| 身分識別工作 | 使用本機或許多社交帳戶來註冊或登入。<br><br>自助式密碼重設。<br><br>設定檔編輯。<br><br>Multi-Factor Authentication。<br><br>自訂權杖和工作階段。<br><br>存取權杖流程。 | 使用自訂識別提供者完成與內建原則相同的工作，或使用自訂範圍。<br><br>註冊時將使用者帳戶佈建在另一個系統中。<br><br>使用您自己的電子郵件服務提供者傳送歡迎電子郵件。<br><br>使用 Azure AD B2C 外部的使用者存放區。<br><br>使用 API 向受信任的系統驗證使用者所提供的資訊。 |

## <a name="policy-files"></a>原則檔

所使用的三種原則檔案類型如下︰

- **基底檔案** - 包含大部分的定義。 建議您儘可能不要變更這個檔案，才有利於原則的疑難排解和長期維護。
- **擴充檔案** - 保存租用戶的唯一組態變更。
- **信賴憑證者 (RP) 檔案** - 以單一工作為主的檔案，直接由應用程式或服務 (也稱為信賴憑證者) 叫用。 每個唯一的工作需要自己的 RP，而根據品牌需求而定，數目可能是「應用程式總數 x 使用案例總數」。

Azure AD B2C 中的內建原則遵守上述的 3 檔案模式，但開發人員只能看到 RP 檔案，而 Azure 入口網站是在背景中變更擴充檔案。

## <a name="custom-policy-core-concepts"></a>自訂原則的核心概念

Azure 中的客戶身分識別和存取管理 (CIAM) 服務包括：

- 使用者目錄，可藉由使用 Microsoft Graph 來存取，並且會保存本機帳戶和同盟帳戶的使用者資料。
- 存取**識別體驗架構**，協調使用者與實體之間的信任，並在它們之間傳遞宣告，以完成身分識別或存取權管理工作。 
- Security Token Service (STS)，發出識別碼權杖、重新整理權杖和存取權杖 (及對等的 SAML 判斷提示)，並驗證它們以保護資源。

Azure AD B2C 會依序與身分識別提供者、使用者、其他系統和本機使用者目錄互動，以達成身分識別工作。 例如讓使用者登入、註冊新的使用者或重設密碼。 識別體驗架構和原則 (也稱為使用者旅程圖或信任架構原則) 會建立多方信任，並明確定義動作項目、動作、通訊協定及完成的步驟順序。

識別體驗架構是完全可設定、原則導向、雲端式的 Azure 平台，協調標準通訊協定格式 (例如 OpenIDConnect、OAuth、SAML、WSFed) 實體和少數非標準格式 (例如：以 REST API 為基礎的系統對系統宣告交換) 實體之間的信任。 此架構會建立方便使用、白色標籤體驗，支援 HTML 和 CSS。

自訂原則以一或多個 XML 格式的檔案表示，各檔案在階層鏈中彼此參考。 XML 元素會定義宣告結構描述、宣告轉換、內容定義、宣告提供者、技術設定檔和使用者旅程圖協調流程步驟等元素。 自訂原則可以一或多個 XML 檔案的形式存取，當信賴憑證者叫用時，會由識別體驗架構執行這些檔案。 設定自訂原則的開發人員必須仔細定義信任關係，以包含中繼資料端點、精確的宣告交換定義，並設定每個識別提供者所需的祕密、金鑰和憑證。

### <a name="inheritance-model"></a>繼承模型

當應用程式呼叫 RP 原則檔時，Azure AD B2C 中的識別體驗架構將先後從基底原則檔、擴充原則檔和 RP 原則檔，新增所有元素，以組合目前生效的原則。  RP 檔案會覆寫擴充原則檔中相同類型和名稱的元素，而擴充原則檔會覆寫基底原則檔。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [開始使用自訂原則](active-directory-b2c-get-started-custom.md)
