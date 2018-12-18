---
title: 在 PIM 中設定 Azure 資源角色的安全性警示 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中設定 Azure 資源角色的安全性警示。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: pim
ms.date: 04/02/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 2fa63cf2fa05f2cde4558f0bea38bfd7f17df3ae
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43342057"
---
# <a name="configure-security-alerts-for-azure-resource-roles-in-pim"></a>在 PIM 中設定 Azure 資源角色的安全性警示
當您的環境中有可疑或不安全的活動時，適用於 Azure 資源的 Privileged Identity Management (PIM) 就會產生警示。 觸發後的警示會顯示在 [警示] 頁面上。 

![警示頁面](media/azure-pim-resource-rbac/RBAC-alerts-home.png)

## <a name="review-alerts"></a>檢閱警示
選取警示，以查看列出觸發警示的使用者或角色以及補救建議的報告。

![警示報告](media/azure-pim-resource-rbac/rbac-alert-info.png)

## <a name="alerts"></a>警示
| 警示 | 嚴重性 | 觸發程序 | 建議 |
| --- | --- | --- | --- |
| **指派了太多的擁有者到資源** |中 |太多使用者具有擁有者角色。 |檢閱清單中的使用者，然後將部分使用者重新指派給較少特殊權限的角色。 |
| **指派太多永久擁有者過到資源** |中 |太多使用者永久指派給某個角色。 |檢閱清單中的使用者，然後重新指派部分使用者以要求啟用角色。 |
| **建立了重複的角色** |中 |多個角色具有相同的準則。 |僅使用其中一個角色。 |


### <a name="severity"></a>嚴重性
* **高**：因為發生原則違規而需要立即採取行動。 
* **中**：不需要立即採取行動，但表示可能發生原則違規。
* **低**：不需要立即採取行動，但建議進行慣用的原則變更。

## <a name="configure-security-alert-settings"></a>設定安全性警示設定
從 [警示] 頁面中移至 [設定]。
![設定](media/azure-pim-resource-rbac/rbac-navigate-settings.png)

自訂不同警示的設定，以便合您的環境和安全性目標。
![自訂設定](media/azure-pim-resource-rbac/rbac-alert-settings.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中設定 Azure 資源角色的安全性警示](pim-resource-roles-configure-alerts.md)
