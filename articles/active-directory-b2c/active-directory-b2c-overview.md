---
title: 什麼是 Azure Active Directory B2C？ | Microsoft Docs
description: 了解如何使用 Azure Active Directory B2C 建立和管理應用程式登入體驗。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 04/05/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6949ab89cf806818783c86199e6df334e263b046
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440876"
---
# <a name="what-is-azure-active-directory-b2c"></a>什麼是 Azure Active Directory B2C？

Azure Active Directory (Azure AD) B2C 是一項身分識別管理服務，可讓您自訂和控制客戶在使用您的應用程式時如何註冊、登入及管理其設定檔。 其中包括針對 iOS、Android 和 .NET 等平台開發的應用程式。 Azure AD B2C 可讓您的客戶執行這些動作，同時保護其身分識別。

您可以設定對 Azure AD B2C 註冊的應用程式，以執行各種不同的身分識別管理動作。 部分範例如下：

- 讓客戶註冊，以使用您已註冊的應用程式
- 讓已註冊的客戶登入，並開始使用您的應用程式
- 讓已註冊的客戶編輯其設定檔
- 在您的應用程式中啟用 Multi-Factor Authentication
- 讓客戶使用特定的身分識別提供者註冊並登入
- 授與從您的應用程式對您建置的 API 進行存取的權限
- 自訂註冊和登入體驗的外觀與風格
- 管理應用程式的單一登入工作階段

## <a name="what-do-i-need-to-think-about-before-using-azure-ad-b2c"></a>使用 Azure AD B2C 之前需要先考量哪些事項？

- 我應該讓客戶如何與我的應用程式互動？
- 我想要為客戶提供怎樣的使用者介面 (UI) 體驗？
- 我要讓客戶從我的應用程式中選擇哪些身分識別提供者？
- 我的登入程序是否需要執行其他 API？

### <a name="customer-interaction"></a>客戶互動

Azure AD B2C 支援將 [OpenID Connect](https://openid.net/connect/) 用於所有客戶體驗。 在 Azure AD B2C 的 OpenID Connect 實作中，您的應用程式會向 Azure AD B2C 發出驗證要求，以起始此使用者旅程圖。 要求的結果是 `id_token`。 此安全性權杖代表客戶的身分識別。

每個使用 Azure AD B2C 的應用程式都必須使用 Azure 入口網站在 Azure AD B2C 租用戶中註冊。 註冊程序會收集和指派一些值給您的應用程式。 這些值包括可唯一識別應用程式的應用程式識別碼，以及可用來將回應傳回給應用程式的重新導向 URI。

每個應用程式的互動都遵循類似的高階模式：

1. 應用程式會引導客戶執行原則。
2. 客戶完成根據原則定義完成原則。
3. 應用程式接收安全性權杖。
4. 應用程式使用安全性權杖嘗試存取受保護的資源。
5. 資源伺服器會驗證安全性權杖，以確認可以授與存取權。
6. 應用程式定期重新整理安全性權杖。

根據您所建置的應用程式類型，這些步驟可能稍有不同。

Azure AD B2C 會依序與身分識別提供者、客戶、其他系統和本機目錄互動，以完成身分識別工作。 例如登入客戶、註冊新的客戶、重設密碼。 建立多方信任並完成這些步驟的基礎平台，稱為身分識別體驗架構。 此架構和原則 (也稱為使用者旅程圖或信任架構原則) 會明確定義動作項目、動作、通訊協定和要完成步驟的順序。

Azure AD B2C 可透過多種方式防止對您的應用程式發動的拒絕服務和密碼攻擊。 Azure AD B2C 使用偵測和降低風險的技術 (例如 SYN Cookie) 及速率和連線限制來保護資源，避免遭受拒絕服務的攻擊。 此外也包含風險降低機制，以防範暴力密碼破解攻擊和字典密碼破解攻擊。

#### <a name="built-in-policies"></a>內建原則

每個傳送至 Azure AD B2C 的要求都會指定一個原則。 原則會控制您的應用程式與 Azure AD B2C 互動的行為模式。 對於最常用的身分識別工作則預先定義了內建原則，這些工作包括註冊、登入和設定檔編輯。  比方說，註冊原則可讓進行下列設定來控制行為：

- 客戶可用來註冊應用程式的社交帳戶
- 對客戶收集的資料，例如名字或郵遞區號
- Multi-Factor Authentication
- 所有註冊頁面的外觀與風格
- 傳回至應用程式的資訊

#### <a name="custom-policies"></a>自訂原則 

[自訂原則](active-directory-b2c-overview-custom.md)是一種組態檔，可定義 Azure AD B2C 租用戶中的身分識別體驗架構的行為。 自訂原則可完整編輯以完成多項工作。 自訂原則以一或多個 XML 格式的檔案表示，各檔案在階層鏈中彼此參考。 

您可以視需要在 Azure AD B2C 租用戶中使用不同類型的多個自訂原則，並且可以跨應用程式重複使用。 此一彈性可讓您在稍微變更或完全不變更程式碼的情況下，定義及修改客戶的身分識別體驗。 您可以在 HTTP 驗證要求中新增特定查詢參數，以使用原則。

自訂原則可透過下列方式用來控制使用者旅程圖：

- 定義如何與 API 互動以擷取其他資訊、驗證客戶提供的宣告，或觸發外部程序。
- 根據 API 中的宣告或目錄 (例如 *migrationStatus*) 中的宣告變更行為。
- 內建原則未涵蓋的任何工作流程。 例如，在登入體驗期間對客戶收集更多資訊，以及執行存取資源的授權檢查。

### <a name="identity-providers"></a>識別提供者

身分識別提供者是會驗證客戶身分識別並簽發安全性權杖的服務。 在 Azure AD B2C 中，您可以在租用戶中設定多個身分識別提供者，例如 Microsoft 帳戶、Facebook 或 Amazon 等。 

若要在您的 Azure AD B2C 租用戶中設定身分識別提供者，您必須從您建立的身分識別提供者應用程式，記下應用程式識別碼或用戶端識別碼以及密碼或用戶端密碼。 此識別碼和密碼後續將用來設定您的應用程式。

### <a name="user-interface-experience"></a>使用者介面體驗

對客戶呈現的多數 HTML 和 CSS 內容都是可控制的。 您可以使用頁面 UI 自訂功能，自訂任何原則的外觀與風格。 您也可以維護應用程式與 Azure AD B2C 之間的品牌和視覺一致性。

Azure AD B2C 會在客戶的瀏覽器中執行程式碼，並使用名為「跨原始資源共用 (CORS)」的新式方法。 首先，您必須在原則中使用自訂 HTML 內容指定 URL。 Azure AD B2C 會合併 UI 元素與從您 URL 載入的 HTML 內容，然後對客戶顯示此頁面。

您可以在查詢字串中將參數傳送至 Azure AD B2C。 將參數傳遞至您的 HTML 端點，即可動態變更網頁內容。 例如，視您從 Web 或行動裝置應用程式傳遞的參數而定，您可以變更 Azure AD B2C 註冊或登入背景影像。

## <a name="how-do-i-get-started-with-azure-ad-b2c"></a>如何開始使用 Azure AD B2C？

在 Azure AD B2C 中，租用戶代表您的組織，而實際上是一個使用者目錄。 每個 Azure AD B2C 租用戶都不相同，並與其他 Azure AD B2C 租用戶分開。 租用戶包含已註冊要使用您的應用程式之客戶的相關資訊。 例如，密碼、設定檔資料和權限。

您必須將您的 Azure AD B2C 租用戶連結至您的 Azure 訂用帳戶，才能啟用所有功能，並支付使用費用。 若要允許 Azure AD B2C 客戶登入您的應用程式，您必須在 Azure AD B2C 租用戶中註冊您的應用程式。

在設定應用程式以使用 Azure AD B2C 之前，您必須先建立 Azure AD B2C 租用戶，並註冊您的應用程式。 若要註冊您的應用程式，請完成[教學課程：註冊應用程式以啟用使用 Azure AD B2C 的註冊和登入](tutorial-register-applications.md)中的步驟。
  
如果您是 ASP.NET Web 應用程式開發人員，請使用[教學課程：使用 Azure AD B2C 啟用透過帳戶的 Web 應用程式驗證](active-directory-b2c-tutorials-web-app.md)中的步驟設定您的應用程式，以進行帳戶驗證。

如果您是桌面應用程式開發人員，請使用[教學課程：使用 Azure AD B2C 啟用透過帳戶的桌面應用程式驗證](active-directory-b2c-tutorials-desktop-app.md)中的步驟設定您的應用程式，以進行帳戶驗證。

如果您是使用 Node.js 的單一頁面應用程式開發人員，請使用[教學課程：使用 Azure AD B2C 啟用透過帳戶的單一頁面應用程式驗證](active-directory-b2c-tutorials-spa.md)中的步驟設定您的應用程式，以進行帳戶驗證。

## <a name="next-steps"></a>後續步驟

繼續進行教學課程，開始為您的應用程式進行註冊和登入體驗的設定。

> [!div class="nextstepaction"]
> [教學課程：註冊應用程式以啟用使用 Azure AD B2C 的註冊和登入](tutorial-register-applications.md)
