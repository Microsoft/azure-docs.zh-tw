---
title: 在 Azure Active Directory 中設定自助式服務應用程式存取管理 | Microsoft Docs
description: 在 Azure Active Directory 中建立及管理安全性群組或 Office 365 群組，並要求安全性群組或 Office 365 群組成員資格
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: get-started-article
ms.date: 09/11/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 7966705df2a352f2011d5f7611277e92d8e2d11a
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294232"
---
# <a name="set-up-azure-active-directory-for-self-service-group-management"></a>設定 Azure Active Directory 進行自助服務群組管理
使用者可以在 Azure Active Directory (Azure AD) 中建立及管理自己的安全性群組或 Office 365 群組。 使用者也可以要求安全性群組或 Office 365 群組成員資格，然後群組擁有者可以管理或拒絕成員資格。 您可以將群組成員資格的日常控制權委派給了解該成員資格的商務內容的人員。 自助式群組管理功能僅供安全性群組和 Office 365 群組使用，但具備郵件功能的安全性群組或通訊群組清單無法使用此功能。

自助式群組管理目前提供兩個重要案例：委派的群組管理和自助式群組管理。

* **委派的群組管理** ：例如，管理對系統管理員公司所使用之 SaaS 應用程式存取權的系統管理員。 管理這些存取權限會變得很麻煩，所以此系統管理員會要求企業業主建立新的群組。 系統管理員會將應用程式的存取權指派給新群組，並新增至所有人員已存取該應用程式的群組。 接著，企業業主可以新增更多使用者，而且這些使用者會自動佈建到應用程式。 企業業主不需要等待系統管理員管理使用者的存取權。 如果系統管理員將相同的權限授與不同業務群組中的管理員，則該人員也可以管理自己的使用者的存取權限。 企業業主或管理員都不可以檢視或管理彼此的使用者。 系統管理員仍然可以看到可以存取應用程式的所有使用者，並視需要封鎖存取權。
* **自助式群組管理** ：此案例的其中一個範例是都已獨立設定 SharePoint Online 網站的兩個使用者。 他們想要賦與彼此團隊各自網站的存取權。 為了達成此目的，他們可以在 Azure AD 中建立一個群組，並各自在 SharePoint Online 中選取該群組來提供對他們網站的存取權。 當有人想要存取權時，他們會從 [存取面板] 中要求，而且在核准之後，他們就可以自動取得這兩個 SharePoint Online 網站的存取權。 之後，其中一個人會決定存取網站的所有人員也可以存取特定的 SaaS 應用程式。 SaaS 應用程式的系統管理員可以將此應用程式的存取權限新增到 SharePoint Online 網站。 從那時起，任何獲得核准的要求都可以存取兩個 SharePoint Online 網站，也可以存取此 SaaS 應用程式。

## <a name="make-a-group-available-for-user-self-service"></a>提供可供使用者自助服務的群組
1. 使用具備目錄全域管理員身分的帳戶來登入 [Azure AD 管理中心](https://aad.portal.azure.com)。
2. 選取 [使用者和群組]，然後選取 [群組設定]。
3. 將 [啟用自助式群組管理] 設定為 [是]。
4. 將 [使用者可以建立安全性群組] 或 [使用者可以建立 Office 365 群組] 設定為 [是]。
  * 若已啟用這些設定，目錄中的所有使用者都可以建立新的安全性群組，並將成員新增至這些群組。 這些新的群組也會顯示在其他所有使用者的 [存取面板] 中。 如果群組的原則設定允許，其他使用者可以建立加入這些群組的要求。 
  * 若已停用這些設定，使用者就無法建立群組，也無法變更其擁有的現有群組。 不過，他們仍然可以管理這些群組的成員資格，以及核准其他使用者加入其群組的要求。

您也可以使用 [可管理安全性群組的使用者]  和 [可管理 Office 365 群組的使用者]，對您使用者的自助式群組管理功能實現更精細的存取控制。 當 [使用者可以建立群組]  啟用時，租用戶中的所有使用者都可以建立新群組，並將成員新增至這些群組。 將他們設定為 [部分]，表示您要將群組管理限制為只有一組有限的使用者。 當這個參數設定為 [部分] 時，您必須先將使用者新增至 SSGMSecurityGroupsUsers 群組，他們才能建立新的群組並在其中新增成員。 將 [可為安全性群組使用自助服務的使用者]  和 [可管理 Office 365 群組的使用者] 設定為 [全部]，表示您允許租用戶中的所有使用者建立新群組。

您也可以使用 [可管理安全性群組的群組]  或 [可管理 Office 365 群組的群組]，指定其成員可使用自助服務的單一群組。

## <a name="next-steps"></a>後續步驟
這些文章提供有關 Azure Active Directory 的其他資訊。

* [使用 Azure Active Directory 群組來管理資源的存取權](../fundamentals/active-directory-manage-groups.md)
* [設定群組設定的 Azure Active Directory Cmdlet](groups-settings-cmdlets.md)
* [Azure Active Directory 中的應用程式管理](../manage-apps/what-is-application-management.md)
* [什麼是 Azure Active Directory？](../fundamentals/active-directory-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
