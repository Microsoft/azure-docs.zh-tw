---
title: 在 PIM 中起始 Azure 資源角色的存取權檢閱 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中起始 Azure 資源角色的存取權檢閱。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: pim
ms.date: 04/02/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 9a35d32d89931a03b33f232ba4f79226fc3f57e5
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189169"
---
# <a name="start-an-access-review-for-azure-resource-roles-in-pim"></a>在 PIM 中起始 Azure 資源角色的存取權檢閱
當使用者擁有不再需要的特殊存取權時，角色指派就會變成「過時」。 若要降低與這些過時角色指派相關聯的風險，特殊權限角色管理員應該定期檢閱角色。 本文件涵蓋在適用於 Azure 資源的 Privileged Identity Management (PIM) 中開始進行存取權檢閱的步驟。

從 PIM 應用程式主頁面移至：

* [存取權檢閱] > [新增]

![新增存取權檢閱](media/azure-pim-resource-rbac/rbac-access-review-home.png)

選取 [新增] 按鈕時，隨即會顯示 [建立存取權檢閱] 刀鋒視窗。 在此刀鋒視窗上，為檢閱設定名稱和時間限制、選擇要檢閱的角色，然後決定要由誰執行檢閱。

![建立存取權檢閱](media/azure-pim-resource-rbac/rbac-create-access-review.png)

### <a name="configure-the-review"></a>設定檢閱
若要建立存取權檢閱，請先為它命名，然後設定開始和結束日期。

![設定檢閱 - 螢幕擷取畫面](media/azure-pim-resource-rbac/rbac-access-review-setting-1.png)

為檢閱設定足夠的時間長度，以便讓使用者能夠完成檢閱。 如果在結束日期之前即完成檢閱，一律可以提早停止檢閱。

### <a name="choose-a-role-to-review"></a>選取要檢閱的角色
每個檢閱只針對一個角色。 除非您是從特定的角色刀鋒視窗開始存取權檢閱，否則您現在必須選擇一個角色。

1. 移至 [檢閱角色成員資格]
   
    ![檢閱角色成員資格 - 螢幕擷取畫面](media/azure-pim-resource-rbac/rbac-access-review-setting-2.png)
2. 從清單中選擇一個角色。

### <a name="decide-who-will-perform-the-review"></a>決定將由誰執行檢閱
執行檢閱的選項有三個。 您可以將檢閱指派給其他人來完成、可以自行進行檢閱，也可以由每個使用者檢閱自己的存取權。

1. 選擇其中一個選項：
   
   * **選取的使用者**︰如果您不知道誰需要存取權，請使用此選項。 使用此選項，您可以指派資源擁有者或群組管理員完成檢閱。
   * **指派 (自我)**：若要讓使用者檢閱自己的角色指派，請使用此選項。
   
2. 移至 [選取檢閱者]。
   
    ![選取檢閱者 - 螢幕擷取畫面](media/azure-pim-resource-rbac/rbac-access-review-setting-3.png)

### <a name="start-the-review"></a>開始檢閱
最後，您可以要求使用者提供核准存取權的原因。 依喜好新增檢閱描述。 然後，選取 [開始]。

請確定讓使用者知道有等待他們執行的存取權檢閱，並示範[如何執行存取權檢閱](pim-resource-roles-perform-access-review.md)。

## <a name="manage-the-access-review"></a>管理存取權檢閱
當檢閱者完成其檢閱時，您可以在 [PIM Azure 資源] 儀表板追蹤進度。 在[完成檢閱](pim-resource-roles-complete-access-review.md)之前，不會變更目錄中的任何存取權限。

在檢閱期間結束之前，您都可以提醒使用者完成其檢閱，或是從 [存取權檢閱] 區段提前停止檢閱。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中完成 Azure 資源角色的存取權檢閱](pim-resource-roles-complete-access-review.md)
- [在 PIM 中執行 Azure 資源角色的存取權檢閱](pim-resource-roles-perform-access-review.md)
- [在 PIM 中起始 Azure AD 目錄角色的存取權檢閱](pim-how-to-start-security-review.md)
