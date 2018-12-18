---
title: 在 Azure Active Directory B2C 中針對自訂原則進行疑難排解 | Microsoft Docs
description: 了解在 Azure Active Directory B2C 中使用自訂原則時，解決錯誤的方法。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 61f2dfcada0f7e03d8043d9b8f1b6dadf7027c8b
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441160"
---
# <a name="troubleshoot-azure-ad-b2c-custom-policies-and-identity-experience-framework"></a>針對 Azure AD B2C 自訂原則和身分識別體驗架構進行疑難排解

如果您使用 Azure Active Directory B2C (Azure AD B2C) 自訂原則，當您以原則語言 XML 格式來設定身分識別體驗架構時，您可能會遇到挑戰。  學習撰寫自訂原則就如同學習新的語言。 在本文中，我們將說明可協助您快速找出並解決問題的工具和提示。 

> [!NOTE]
> 本文著重在針對 Azure AD B2C 自訂原則設定進行疑難排解。 其中不討論信賴憑證者應用程式或其身分識別程式庫。

## <a name="xml-editing"></a>XML 編輯

設定自訂原則時，最常見的錯誤是 XML 格式不正確。 良好的 XML 編輯器幾乎不可或缺。 良好的 XML 編輯器會顯示 XML 原貌、為內容加上色彩、預先填入常用字詞、保持編製 XML 元素的索引，而且可以根據結構描述進行驗證。 以下是我們最喜愛的兩個 XML 編輯器：

* [Visual Studio Code](https://code.visualstudio.com/)
* [Notepad++](https://notepad-plus-plus.org/)

在您上傳 XML 檔案之前，XML 結構描述驗證會識別錯誤。 在入門套件的根資料夾中，取得 XML 結構描述定義 TrustFrameworkPolicy_0.3.0.0.xsd。 如需詳細資訊，請在 XML 編輯器的文件中尋找「XML 工具」和「XML 驗證」。

您可能會發現檢閱 XML 規則很有幫助。 Azure AD B2C 會拒絕它所偵測到的任何 XML 格式錯誤。 格式錯誤的 XML 有時可能會產生令人誤解的錯誤訊息。

## <a name="upload-policies-and-policy-validation"></a>上傳原則和原則驗證

 XML 檔案上傳驗證是自動執行。 大部分的錯誤會導致上傳失敗。 驗證包括您要上傳的原則檔。 也包括上傳檔案所參考的一連串檔案 (信賴憑證者原則檔、擴充檔案和基底檔案)。 
 
 常見的驗證錯誤包括下列幾個。

錯誤程式碼片段︰`... makes a reference to ClaimType with id "displaName" but neither the policy nor any of its base policies contain such an element`
* ClaimType 值可能拼字錯誤，或不存在於結構描述中。
* ClaimType 值至少必須定義於原則內的其中一個檔案中。 
    例如：` <ClaimType Id="socialIdpUserId">`
* 如果 ClaimType 定義於擴充檔案中，但也用於基底檔案的 TechnichalProfile 值中，則上傳基底檔案會導致錯誤。

錯誤程式碼片段︰`...makes a reference to a ClaimsTransformation with id...`
* 此錯誤的原因可能與 ClaimType 錯誤一樣。

錯誤程式碼片段︰`Reason: User is currently logged as a user of 'yourtenant.onmicrosoft.com' tenant. In order to manage 'yourtenant.onmicrosoft.com', please login as a user of 'yourtenant.onmicrosoft.com' tenant`
* 檢查 **\<TrustFrameworkPolicy\>** 和 **\<BasePolicy\>** 元素中的 TenantId 值符合目標 Azure AD B2C 租用戶。  

## <a name="troubleshoot-the-runtime"></a>針對執行階段進行疑難排解

* 使用 `Run Now` 和 `https://jwt.io`，在不受 Web 或行動應用程式的影響下測試您的原則。 此網站的作用就像信賴憑證者應用程式。 它會顯示 Azure AD B2C 原則所產生的 JSON Web 權杖 (JWT) 內容。 若要在身分識別體驗架構中建立測試應用程式，請使用下列值：
    * 名稱︰TestApp
    * Web 應用程式/Web API：無
    * 原生用戶端︰否

* 若要追蹤用戶端瀏覽器和 Azure AD B2C 之間的訊息交換，請使用 [Fiddler](http://www.telerik.com/fiddler)。 它可協助您了解使用者旅程圖在協調流程步驟中的何處失敗。

* 在**開發模式**中，請使用 **Application Insights** 來追蹤身分識別體驗架構使用者旅程圖的活動。 在**開發模式**中，您可以觀察在身分識別體驗架構和各種宣告提供者 (由技術設定檔定義) 之間的宣告交換，這些提供者包括識別提供者、API 型服務、Azure AD B2C 使用者目錄和其他服務 (例如 Azure Multi-Factor-Authentication)。  

## <a name="recommended-practices"></a>建議的做法

**為您的情節保留多個版本。將它們和您的應用程式一起放在一個專案中。** 基底、擴充和信賴憑證者檔案直接相互依存。 將它們儲存成一個群組。 有新功能新增至您的原則時，保留個別的工作版本。 將工作版本和其所互動的應用程式程式碼，分階段放入您自己的檔案系統中。  您的應用程式可能會在一個租用戶中叫用許多不同的信賴憑證者原則。 它們可能會變成相依於您的 Azure AD B2C 原則可能提供的宣告。

**使用已知的使用者旅程圖來開發和測試技術設定檔。** 使用已測試的入門套件原則來設定您的技術設定檔。 將它們併入您自己的使用者旅程圖之前，先個別進行測試。

**使用已測試的技術設定檔來開發和測試使用者旅程圖。** 累加地變更使用者旅程圖的協調流程步驟。 漸進地建置您想要的情節。

## <a name="next-steps"></a>後續步驟

* 在 GitHub 中，下載 [active-directory-b2c-custom-policy-starterpack] (https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) .zip 檔案。
