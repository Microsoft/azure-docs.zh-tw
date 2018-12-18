---
title: 使用 Azure AD 管理對應用程式的存取 | Microsoft Docs
description: 描述 Azure Active Directory 如何讓組織可以指定每個使用者擁有存取權的應用程式。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/16/2017
ms.author: barbkess
ms.openlocfilehash: cec1b3089403966cbf738b372a4380bdacf264b7
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44718269"
---
# <a name="managing-access-to-apps"></a>管理應用程式的存取
在應用程式整合到您的組織的身分識別系統之後，進行中的存取管理、使用方式評估和報告持續成為一項挑戰。 在許多情況下，IT 系統管理員或技術服務人員都需要持續積極地為您管理應用程式存取權。 有時候，指派是由一般或分區 IT 小組執行。 通常，指派決策要委派給商務決策人員，在 IT 進行指派之前需要其核准。  其他組織投資於與現有的自動化身分識別與存取管理系統的整合，像是角色型存取控制 (RBAC) 或屬性型存取控制 (ABAC)。 整合與規則開發往往需要特製且所費不貲。 監視或報告任一管理方式是自己單獨、昂貴且複雜的投資。

## <a name="how-does-azure-active-directory-help"></a>Azure Active Directory 有何助益？
 Azure AD 對於已設定的應用程式支援廣泛的存取管理，讓組織得以透過委派並納入系統管理員管理，輕易地達成正確的存取原則，範圍包括自動、屬性式指派 (ABAC 或 RBAC 案例)。 有了 Azure AD，您可以輕鬆地達成複雜的原則，結合單一應用程式的多個管理模型並且甚至可以在具有相同對象的應用程式之間重覆使用管理規則。

* [加入新的或現有的應用程式](configure-single-sign-on-portal.md)

 Azure AD 的應用程式指派著重於兩種主要的指派模式：

* **個別指派** ：具備目錄全域管理員權限的 IT 系統管理員可以選取個別使用者帳戶，並授與它們應用程式存取權限。
* **群組指派 (僅限付費 Azure AD)** ：具備目錄全域管理員權限的 IT 系統管理員可以將群組指派給應用程式。 特定使用者的存取權決於在使用者嘗試存取應用程式時，使用者是否為群組的成員。 換句話說，系統管理員可以有效地建立指派規則，說明「獲指派群組的任何現有成員可存取應用程式」。 使用這個指派選項，系統管理員可以受益於 Azure AD 群組管理選項，包括 [屬性型動態群組](../fundamentals/active-directory-groups-create-azure-portal.md)、外部系統群組 (例如：內部部署 Active Directory 或工作日)、系統管理員管理或自助管理的群組。 單一群組可以輕鬆地指派給多個應用程式，確保與指派同質性的應用程式可以共用指派規則，降低整體管理複雜度。 請注意，目前對應用程式的群組式指派並不支援巢狀群組成員資格。

系統管理員可以使用這些兩種指派模式，達成任何需要的指派管理方法。

利用 Azure AD，使用方式和指派報告可完全整合，讓系統管理員輕鬆地報告指派狀態、指派錯誤和平均使用量。

## <a name="complex-application-assignment-with-azure-ad"></a>Azure AD 的複雜應用程式指派
考慮 Salesforce 之類的應用程式。 在許多組織中，Salesforce 主要是由行銷和銷售組織使用。 通常，行銷小組的成員對 Salesforce 具有高特殊權限存取權，而銷售小組的成員具有限制存取權。 在許多情況下，為數眾多的資訊工作者對應用程式的存取都會受到限制。 這些規則的例外狀況會使情況更加複雜。 它通常是行銷或銷售領導小組的特權，用來授與使用者存取或在這些一般規則以外個別變更其角色。

利用 Azure AD，可以預先設定 Salesforce 之類的應用程式使用單一登入 (SSO) 和自動化佈建。 一旦設定了應用程式，系統管理員可以採取一次性動作來建立及指派適當的群組。 在此範例中，系統管理員可以執行下列指派：

* [動態群組](../fundamentals/active-directory-groups-create-azure-portal.md) ，以自動代表行銷和銷售小組的所有成員使用部門或角色之類的屬性：
  
  * 行銷群組的所有成員會全部指派給 Salesforce 中的「行銷」角色
  * 銷售小組群組的所有成員會全部指派給 Salesforce 中的「銷售」角色。 進一步細分可使用多個群組，其代表指派到不同 Saleforce 角色的區域銷售小組。
* 若要啟用例外狀況機制，可以為每個角色建立自助群組。 例如，可以將「Salesforce 行銷例外狀況」群組建立為自助群組。 群組可以指派給 Salesforce 行銷角色，而行銷領導小組可成為擁有者。 行銷領導小組的成員可以加入或移除使用者、設定聯結原則或甚至核准或拒絕個別使用者加入的要求。 此機制透過不需要經過擁有者或成員特殊訓練的資訊工作者適當的體驗來支援。

在此情況下，所有指派的使用者會自動佈建至 Salesforce，因為當他們加入至不同群組時，他們的角色指派會在 Salesforce 中更新。 使用者就能夠透過 Microsoft 應用程式存取面板、Office Web 用戶端或甚至是藉由瀏覽至其組織的 Salesforce 登入頁面來探索及存取 Salesforce。 系統管理員能夠使用 Azure AD 報告輕鬆檢視使用方式和指派狀態。

系統管理員可以運用 [Azure AD 條件式存取](../active-directory-conditional-access-azure-portal.md) 為特定角色設定存取原則。 這些原則可以包括是否允許公司環境外部的存取，甚至是 Multi-Factor Authentication 或裝置需求，以在各種情況下達成存取。

## <a name="next-steps"></a>後續步驟
* [使用條件式存取來保護應用程式](../active-directory-conditional-access-azure-portal.md)
* [自助式群組管理/SSAA](../users-groups-roles/groups-self-service-management.md)
