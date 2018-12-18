---
title: 將應用程式新增至 Azure Active Directory 的方式和原因
description: 對於應用程式來說，新增至 Azure AD 有何意義，以及要如何新增？
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 3321d130-f2a8-4e38-b35e-0959693f3576
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/18/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: elisol, lenalepa
ms.openlocfilehash: fb2bfc89322d81833b1961bfb866a773c5d1d475
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39577225"
---
# <a name="how-and-why-applications-are-added-to-azure-ad"></a>將應用程式加入至 Azure AD 的方式和原因

應用程式在 Azure AD 中有兩種表示方式： 
* [應用程式物件](app-objects-and-service-principals.md#application-object) - 雖有[例外狀況](#notes-and-exceptions)，但可以將應用程式物件視為應用程式的定義。
* [服務主體](app-objects-and-service-principals.md#service-principal-object) - 可以將這些主體視為應用程式的執行個體。 服務主體通常會參考應用程式物件，而且一個應用程式物件可供不同目錄的多個服務主體來參考。

## <a name="what-are-application-objects-and-where-do-they-come-from"></a>什麼是應用程式物件，其來自何處？
您可以在 Azure 入口網站中透過[應用程式註冊](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade)體驗來管理[應用程式物件](app-objects-and-service-principals.md#application-object)。 應用程式物件會向 Azure AD 描述應用程式，而且您可以將其視為應用程式的定義，其可讓服務了解如何根據其設定對應用程式核發權杖。 應用程式物件只存在於其主目錄中，即使其為在其他目錄中支援服務主體的多租用戶應用程式也是如此。 應用程式物件可能包含下列任何項目 (以及此處未提及的其他資訊)：
* 名稱、標誌和發行者
* 回覆 URL
* 密碼 (用來驗證應用程式的對稱和/或非對稱金鑰)
* API 相依性 (OAuth)
* 已發佈的 API/資源/範圍 (OAuth)
* 應用程式角色 (RBAC)
* SSO 中繼資料和組態
* 使用者佈建中繼資料和組態
* Proxy 中繼資料和組態

您可以透過多個路徑建立應用程式物件，包括：
* Azure 入口網站中的應用程式註冊
* 使用 Visual Studio 建立新的應用程式，並將它設定為使用 Azure AD 驗證
* 當系統管理員從應用程式庫新增應用程式時 (這也會建立服務主體)
* 使用 Microsoft Graph API、Azure AD Graph API 或 PowerShell 建立新的應用程式
* 許多其他路徑，包括 Azure 中的各種開發人員經驗，以及在跨開發人員中心的 API 總管體驗

## <a name="what-are-service-principals-and-where-do-they-come-from"></a>什麼是服務主體，其來自何處？
您可以在 Azure 入口網站中透過[企業應用程式](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/)體驗來管理[服務主體](app-objects-and-service-principals.md#service-principal-object)。 服務主體可控管連線至 Azure AD 的應用程式，您可以將其視為目錄中應用程式的執行個體。 任何給定的應用程式最多可以有一個應用程式物件 (註冊於「主」目錄)，以及一或多個服務主體物件 (代表應用程式作用所在每個目錄中應用程式的執行個體)。 

服務主體可包含：

* 透過應用程式識別碼屬性對應用程式物件的反向參考
* 本機使用者和群組應用程式角色指派的記錄
* 針對已授與應用程式的本機使用者和系統管理員權限所做的記錄
  * 例如：應用程式用來存取特定使用者電子郵件的權限
* 本機原則的記錄，包括條件式存取原則
* 應用程式替代本機設定的記錄
  * 宣告轉換規則
  * 屬性對應 (使用者佈建)
  * 目錄專屬的應用程式角色 (如果應用程式支援自訂角色)
  * 目錄專屬的名稱或標誌

和應用程式物件一樣，您也可以透過多個路徑來建立服務主體，包括：

* 當使用者登入與 Azure AD 整合的第三方應用程式時
  * 登入期間，系統會要求使用者為應用程式提供可用來存取其設定檔的權限以及其他權限。 第一個同意的人就會將代表應用程式的服務主體新增至目錄。
* 當使用者登入 Microsoft 線上服務時，例如 [Office 365](http://products.office.com/)
  * 訂閱或開始試用 Office 365 時，目錄中會建立一或多個服務主體，代表用來傳遞所有 Office 365 相關功能的各種服務。
  * 某些 Office 365 服務(如 SharePoint) 會持續建立服務主體，以允許元件之間的安全通訊，包括工作流程。
* 當系統管理員從應用程式庫新增應用程式時 (這也會建立基礎應用程式物件)
* 新增應用程式來使用 [Azure AD 應用程式 Proxy](https://msdn.microsoft.com/library/azure/dn768219.aspx)
* 連接應用程式以便啟用 SAML 單一登入或密碼單一登入 (SSO)
* 以程式設計方式透過 Azure AD Graph API 或 PowerShell 來建立

## <a name="how-are-application-objects-and-service-principals-related-to-each-other"></a>應用程式物件和服務主體彼此之間如何建立關聯性？
應用程式在其主目錄中有一個應用程式物件，可供其運作所在每個目錄 (包括應用程式的主目錄) 中的一或多個服務主體來參考。
![說明應用程式物件和服務主體在 Azure AD 執行個體中如何互動的圖表。][apps_service_principals_directory]

在上圖中，Microsoft 會在內部維護兩個用來發佈應用程式的目錄 (顯示在左)：

* 一個用於 Microsoft 應用程式 (Microsoft 服務目錄)
* 一個用於預先整合的第三方應用程式 (應用程式庫目錄)

與 Azure AD 整合的應用程式發行者/廠商必須具有發佈目錄 (顯示在右的「部分 SaaS 目錄」)。

您自行新增的應用程式 (以圖中的**應用程式 (您的)** 來代表) 包括：

* 您所開發的應用程式 (與 Azure AD 整合)
* 為單一登入而進行連線的應用程式
* 使用 Azure AD 應用程式 Proxy 所發佈的應用程式

### <a name="notes-and-exceptions"></a>附註及例外狀況
* 並非所有服務主體都會往回指向應用程式物件。 原先建置 Azure AD 時提供給應用程式的服務會有更多限制，而且服務主體就足以建立應用程式識別碼。 原先服務主體的功能狀況接近 Windows Server Active Directory 服務帳戶。 基於這個原因，您還是可以透過不同路徑來建立服務主體 (例如使用 Azure AD PowerShell)，而無需先建立應用程式物件。 Azure AD Graph API 需要應用程式物件才能建立服務主體。
* 並非所有上述資訊目前都處於以程式設計方式公開的狀態。 以下功能僅適用於 UI：
  * 宣告轉換規則
  * 屬性對應 (使用者佈建)
* 如需服務主體與應用程式物件的詳細資訊，請參閱 Azure AD Graph REST API 參考文件：
  * [應用程式](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#application-entity)
  * [服務主體](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#serviceprincipal-entity)

## <a name="why-do-applications-integrate-with-azure-ad"></a>應用程式為何要與 Azure AD 整合？
將應用程式新增至 Azure AD，即可利用一或多個其所提供的服務：

* 應用程式驗證和授權
* 使用者驗證和授權
* 使用同盟或密碼的 SSO
* 使用者佈建與同步處理
* 角色型存取控制 - 使用目錄來定義應用程式角色，才能在應用程式中執行角色型授權檢查
* OAuth 授權服務 - Office 365 和其他 Microsoft 應用程式會用它來授與對 API/資源的存取權限
* 應用程式發佈與 Proxy - 將應用程式從私人網路發佈到網際網路

## <a name="who-has-permission-to-add-applications-to-my-azure-ad-instance"></a>誰有權將應用程式加入我的 Azure AD 執行個體？
雖然仍有一些工作只能由全域管理員進行 (例如，從應用程式庫新增應用程式，以及將應用程式設定為使用應用程式 Proxy)，但根據預設，目錄中的所有使用者都有權註冊其所開發的應用程式，並可自行決定哪些應用程式可在經過同意後共用/存取其組織資料。 如果某人是您目錄中第一位登入應用程式並授與同意的使用者，便會在您的租用戶中建立服務主體；否則，就會將同意授與資訊儲存在現有的服務主體內。

剛聽到要允許使用者註冊並同意應用程式可能會讓您有些疑慮，但請記住下列要點：


* 在無需在目錄中註冊或記錄應用程式的情況下，應用程式能夠利用 Windows Server Active Directory 進行使用者驗證已有數年的時間。 現在組織已提高可見度，您可以知道究竟有多少個應用程式正在使用目錄以及使用目的為何。
* 對使用者委派這些責任，就不必建立由系統管理員驅動的應用程式註冊和發行程序。 有了 Active Directory Federation Services (ADFS)，系統管理員很有可能必須代表其開發人員，將應用程式新增為信賴憑證者。 現在開發人員可以自助。
* 基於商業用途，使用者使用其組織帳戶登入應用程式是件好事。 如果使用者之後離開組織，他們就會自動失去其在先前所用應用程式中帳戶的存取權。
* 記錄哪些資料與哪些應用程式共用是件好事。 資料比以往更容易進行傳輸，且清楚記錄哪些人員與哪些應用程式共用哪些資料會很有用。
* 使用 Azure AD for OAuth 的 API 擁有者可確切決定使用者可以授與應用程式哪些權限，以及哪些權限需要系統管理員的同意。 唯有系統管理員可以同意較大範圍且更重要的權限，使用者的同意範圍則僅限於使用者自己的資料和功能。
* 當使用者新增應用程式並允許應用程式存取其資料時，事件將可供稽核，讓您可在 Azure 入口網站中檢視稽核報告，從而確定應用程式是如何新增到目錄的。

如果您仍想讓目錄中的使用者在未獲得系統管理員核准時，就無法註冊應用程式和登入應用程式，您可以變更兩項設定以關閉這些功能：

* 若要防止使用者同意應用程式代表使用者自己行事：
  1. 在 Azure 入口網站中，移至企業應用程式下的[使用者設定](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/)區段。
  2. 將 [使用者可同意應用程式代表自己存取公司資料] 變更為 [否]。 
      > [!NOTE]
      > 如果您決定關閉使用者同意功能，使用者需要使用的任何新應用程式就必須由系統管理員同意。    
* 若要防止使用者註冊自己的應用程式：
  1. 在 Azure 入口網站中，移至 Azure Active Directory 下的[使用者設定](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/UserSettings)區段
  2. 將 [使用者可以註冊應用程式] 變更為 [否]。

> [!NOTE]
> 對於能夠註冊應用程式並同意應用程式代表自己的使用者，Microsoft 本身會使用預設組態。

<!--Image references-->
[apps_service_principals_directory]:../media/active-directory-how-applications-are-added/HowAppsAreAddedToAAD.jpg

