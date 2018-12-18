---
title: 在 PIM 中檢視 Azure 資源角色的稽核記錄 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中檢視 Azure 資源角色的稽核記錄。
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
ms.openlocfilehash: c0536423e9640f78149b612ec66b0a07cdcf24bb
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189306"
---
# <a name="view-audit-history-for-azure-resource-roles-in-pim"></a>在 PIM 中檢視 Azure 資源角色的稽核記錄

資源稽核可讓您檢視資源的所有角色活動。 您可以使用預先定義的日期或自訂範圍來篩選資訊。
![篩選資訊](media/azure-pim-resource-rbac/rbac-resource-audit.png)

資源稽核也可讓使用者快速存取使用者的活動詳細資料。 在 [稽核類型] 下選取 [啟用]。 選取 [(活動)] 可在 Azure 資源上看到該使用者的動作。
![活動詳細資料](media/azure-pim-resource-rbac/rbac-audit-activity.png)

![更多活動詳細資料](media/azure-pim-resource-rbac/rbac-audit-activity-details.png)

## <a name="my-audit"></a>我的稽核

「我的稽核」可讓您檢視使用者的個人角色活動。 您可以使用預先定義的日期或自訂範圍來篩選資訊。
![個人角色活動](media/azure-pim-resource-rbac/my-audit-time.png)

## <a name="view-activation-and-azure-resource-activity"></a>檢視啟用與 Azure 資源活動

若要查看各種資源中特定使用者採取了哪些動作，您可以檢閱與指定啟用期間相關聯的 Azure 資源活動。 從 [成員] 檢視或從特定角色的成員清單中選取使用者來開始。 結果會依日期顯示使用者在 Azure 資源中所採取動作的圖形化檢視。 它也會顯示同一段時間週期內最新的角色啟用。

![使用者詳細資料](media/azure-pim-resource-rbac/rbac-user-details.png)

選取特定的角色啟用會顯示角色啟用的詳細資料，以及該使用者作用中時所發生的對應 Azure 資源活動。

![選取角色啟用](media/azure-pim-resource-rbac/rbac-user-resource-activity.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中檢視 Azure AD 目錄角色的稽核記錄](pim-how-to-use-audit-log.md)
