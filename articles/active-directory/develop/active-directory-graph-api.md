---
title: Azure Active Directory Graph API | Microsoft Docs
description: Azure AD 圖形 API 的概觀和快取入門指南，可讓您以程式設計方式透過 REST API 端點存取 Azure AD。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.assetid: 5471ad74-20b3-44df-a2b5-43cde2c0a045
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: dkershaw, sureshja
ms.custom: aaddev
ms.openlocfilehash: bcac650e4ce4ed3260920feec5687c8265ce0c13
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977514"
---
# <a name="azure-active-directory-graph-api"></a>Azure Active Directory 圖形 API

> [!IMPORTANT]
> 強烈建議您使用 [Microsoft Graph](https://graph.microsoft.io/) 取代 Azure AD Graph API 來存取 Azure Active Directory 資源。 我們的開發工作現在是針對 Microsoft Graph，並沒有針對 Azure AD Graph API 規劃的進一步增強功能 。 有極少數的案例可能仍適用 Azure AD Graph API；如需詳細資訊，請參閱 Office 開發人員中心的 [Microsoft Graph 或 Azure AD Graph](https://dev.office.com/blogs/microsoft-graph-or-azure-ad-graph) 部落格文章。

Azure Active Directory 圖形 API 支援以程式設計方式透過 REST API 端點存取 Azure AD。 應用程式可以使用 Azure AD 圖形 API 來執行有關目錄資料和物件的建立、讀取、更新及刪除 (CRUD) 作業。 例如，Azure AD 圖形 API 支援對使用者物件執行下列常見的作業：

* 在目錄中建立新的使用者
* 取得使用者的詳細屬性，例如其群組
* 更新使用者的屬性 (例如其位置和電話號碼)，或變更其密碼
* 檢查使用者在角色型存取方面的群組成員資格
* 停用使用者帳戶或完全刪除

此外，您也可以在其他物件上執行類似的作業，例如群組和應用程式。 若要在目錄上呼叫 Azure AD 圖形 API，應用程式必須向 Azure AD 註冊。 您還必須對應用程式授與 Azure AD 圖形 API 的存取權。 此存取權通常是透過使用者或系統管理員同意流程來實現。

若要開始使用 Azure Active Directory Graph API，請參閱 [Azure AD Graph API 快速入門指南](active-directory-graph-api-quickstart.md)，或檢視[互動式 Azure AD Graph API 參考文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。

## <a name="features"></a>特性

Azure AD 圖形 API 提供下列功能：

* **REST API 端點**：Azure AD 圖形 API 是 RESTful 服務，由使用標準 HTTP 要求存取的端點所組成。 Azure AD 圖形 API 支援要求和回應的 XML 或 Javascript 物件標記法 (JSON) 內容類型。 如需詳細資訊，請參閱 [Azure AD Graph REST API 參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。
* **向 Azure AD 驗證**：必須在要求的 Authorization 標頭中附加 JSON Web Token (JWT)，以驗證 Azure AD 圖形 API 的每個要求。 您可以向 Azure AD 的權杖端點提出要求並提供有效的認證，以取得此權杖。 您可以使用 OAuth 2.0 用戶端認證流程或授權碼授與流程，以取得權杖來呼叫 Graph。 如需詳細資訊，請參閱 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/library/azure/dn645545.aspx)。
* **以角色為基礎的授權 (RBAC)**：在 Azure AD 圖形 API 中使用安全性群組執行 RBAC。 例如，如果您想要判斷使用者是否能夠存取特定的資源，應用程式可以呼叫 [檢查群組成員資格 (可移轉)](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/functions-and-actions#checkMemberGroups) 作業，它會傳回 true 或 false。
* **差異查詢**：差異查詢可讓您追蹤兩段時間之間的目錄變更，而不必對 Azure AD 圖形 API 進行頻繁的查詢。 此類型的要求只會傳回前一個差異查詢要求與目前要求之間所做的變更。 如需詳細資訊，請參閱 [Azure AD Graph API 差異查詢](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-differential-query)。
* **目錄擴充**：您可以將自訂屬性新增至目錄物件，而不需要用到外部資料存放區。 例如，如果應用程式需要每個使用者都有 Skype ID 屬性，您可以在目錄中註冊新的屬性，此屬性就可供每個使用者物件使用。 如需詳細資訊，請參閱 [Azure AD Graph API 目錄結構描述延伸模組](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions)。
* **受權限範圍保護**：Azure AD 圖形 API 會公開權限範圍，以供使用 OAuth 2.0 安全存取 Azure AD 資料。 其支援各種用戶端應用程式類型，包括︰
  
  * 透過登入使用者 (委派) 授權而獲得資料委派存取權的使用者介面
  * 未顯示登入使用者而在背景中作業，並使用應用程式所定義的角色型存取控制服務/精靈應用程式
    
    委派和應用程式權限都代表 Azure AD 圖形 API 公開的權限，而且用戶端應用程式可以透過應用程式註冊權限要求它們 ([Azure 入口網站中的功能](https://portal.azure.com))。 [Azure AD Graph API 權限範圍](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)會提供可供用戶端應用程式使用的權限資訊。

## <a name="scenarios"></a>案例

Azure AD 圖形 API 支援許多應用程式案例。 以下是最常見的案例：

* **企業營運 (單一租用戶) 應用程式**：在此案例中，企業開發人員任職於具有 Office 365 訂用帳戶的組織中。 開發人員正在建置的 Web 應用程式會與 Azure AD 互動來執行工作，例如指派授權給使用者。 這項工作需要存取 Azure AD 圖形 API，所以開發人員在 Azure Ad 中註冊單一租用戶應用程式，並設定 Azure AD 圖形 API 的讀取和寫入權限。 然後，將應用程式設定為使用它自己的認證，或目前登入的使用者認證，以取得權杖來呼叫 Azure AD 圖形 API。
* **軟體即服務應用程式 (多租用戶)**：在此案例中，獨立軟體廠商 (ISV) 正在開發託管的多租用戶 Web 應用程式，目的是為使用 Azure AD 的其他組織提供使用者管理功能。 這些功能需要存取目錄物件，所以此應用程式需要呼叫 Azure AD 圖形 API。 開發人員在 Azure AD 中註冊此應用程式，將它設定為需要 Azure AD 圖形 API 的讀取和寫入權限，然後啟用外部存取，讓其他組織同意在其目錄中使用此應用程式。 當另一個組織中的使用者第一次向應用程式驗證時，就會出現同意對話方塊及此應用程式所要求的權限。 同意之後，就會給予所要求的權限，讓應用程式在使用者的目錄中存取 Azure AD 圖形 API。 如需同意架構的詳細資訊，請參閱 [同意架構的概觀](consent-framework.md)。

## <a name="next-steps"></a>後續步驟

若要開始使用 Azure Active Directory Graph API，請參閱下列主題：

* [Azure AD Graph API 快速入門指南](active-directory-graph-api-quickstart.md)
* [Azure AD Graph REST 文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)
