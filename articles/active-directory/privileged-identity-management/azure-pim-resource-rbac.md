---
title: 在 PIM 中檢視具有 Azure 資源角色的人員 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中檢視具有 Azure 資源角色的人員。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: pim
ms.date: 03/30/2018
ms.author: rolyon
ms.openlocfilehash: ce7c96d92938c4e3b4cc0b53271df48350083754
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465226"
---
# <a name="view-who-has-azure-resource-roles-in-pim"></a>在 PIM 中檢視具有 Azure 資源角色的人員

有了 Azure Active Directory Privileged Identity Management (PIM)，您便可以管理、控制及監視對您組織內 Azure 資源的存取。 適用範圍包括訂用帳戶、資源群組甚至是虛擬機器。 所有在 Azure 入口網站內利用 Azure 角色型存取控制 (RBAC) 功能的資源，都可以運用 Azure AD PIM 中的安全性和生命週期管理功能。 

## <a name="pim-for-azure-resources-helps-resource-administrators"></a>適用於 Azure 資源的 PIM 可協助資源系統管理員

- 查看您所管理 Azure 資源的角色指派到哪些使用者和群組
- 視需要啟用「及時」存取以管理資源，例如「訂用帳戶」、「資源群組」以及其他
- 使用新的時間界限指派設定，讓已指派使用者/群組資源的存取自動到期
- 在快速工作和待命排程上指派暫時資源存取
- 在任何內建或自訂角色上強制執行 Multi-Factor Authentication 以存取資源 
- 在使用者作用中工作階段期間，取得資源存取相互關聯資源活動的報告
- 在新使用者或群組獲指派資源存取，以及啟用符合資格的指派時取得警示

## <a name="view-activation-and-azure-resource-activity"></a>檢視啟用與 Azure 資源活動

如果您需要查看特定使用者對各種資源所採取的動作，您可以檢閱與特定啟用期間相關聯的 Azure 資源活動 (適用於符合資格的使用者)。 若要開始，請從 [成員] 檢視或從特定角色的成員成員中選取使用者。 結果會以圖表形式顯示，包含使用者在 Azure 資源上的動作 (按日期排序)，以及相同時間內最近的角色啟用。

![](media/azure-pim-resource-rbac/user-details.png)

選取特定角色啟用會顯示角色啟用的詳細資料，以及使用者作用中時發生的相應的 Azure Resource 活動。

![](media/azure-pim-resource-rbac/audits.png)

## <a name="review-who-has-access-in-a-subscription"></a>檢閱訂用帳戶中具有存取權的使用者

若要檢閱訂用帳戶中的角色指派，請從左側導覽中，選取 [成員] 索引標籤或選取角色，並選擇特定角色以檢視成員。 

從動作列選取 [檢閱] 以檢閱現有的存取權檢閱，然後選取 [新增] 以建立新檢閱。

![](media/azure-pim-resource-rbac/owner.png)

[深入了解存取權檢閱](pim-how-to-perform-security-review.md)

>[!NOTE]
訂用帳戶資源類型僅在此時支援 [檢閱]。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中指派 Azure 資源角色](pim-resource-roles-assign-roles.md)
- [在 PIM 中核准或拒絕 Azure 資源角色的要求](pim-resource-roles-approval-workflow.md)
- [Azure 中的內建角色](../../role-based-access-control/built-in-roles.md)
