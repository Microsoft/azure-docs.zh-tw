---
title: 在 PIM 中完成 Azure 資源角色的存取權檢閱 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中完成 Azure 資源角色的存取權檢閱。
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
ms.openlocfilehash: f998c509e9bea65980367690a5e9d03f579b8e98
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189122"
---
# <a name="complete-an-access-review-for-azure-resource-roles-in-pim"></a>在 PIM 中完成 Azure 資源角色的存取權檢閱
在[開始存取權檢閱](pim-resource-roles-start-access-review.md)之後，特殊權限角色管理員就可以檢閱特殊權限存取權。 適用於 Azure 資源的 Privileged Identity Management (PIM) 會自動傳送電子郵件，提示使用者檢閱其存取權。 如果使用者未收到電子郵件，您可以將[如何執行存取權檢閱](pim-resource-roles-perform-access-review.md)中的指示傳送給他們。

存取權檢閱期間結束後，或所有使用者都已完成其自我檢閱後，請遵循本文的步驟來管理檢閱並查看結果。

## <a name="manage-access-reviews"></a>管理存取權檢閱
1. 移至 [Azure 入口網站](https://portal.azure.com/)。 然後在儀表板中，選取 [Azure 資源] 應用程式。

2. 選取您的資源。

3. 選取儀表板的 [存取權檢閱]  區段。
![存取權檢閱](media/azure-pim-resource-rbac/rbac-access-review-home-list.png)

4. 選取您想要管理的存取權檢閱。

在存取權檢閱的詳細資料刀鋒視窗上，有一些可管理該檢閱的選項。 選項如下：

![管理檢閱的選項](media/azure-pim-resource-rbac/rbac-access-review-menu.png)

### <a name="stop"></a>Stop
所有的存取權檢閱都有結束日期，但是您可以使用 [停止]  按鈕來提早結束檢閱。 在您停止檢閱後，所有尚未在此時間內完成檢閱的使用者，將無法完成檢閱。 在停止檢閱之後，即無法重新開始檢閱。

### <a name="reset"></a>重設
您可以重設存取權檢閱，以移除在其上所做的所有決策。 重設存取權檢閱後，所有使用者都會再次標示為「未檢閱」。 

### <a name="apply"></a>套用
完成存取權檢閱後，請使用 [套用] 按鈕來實作檢閱的結果。 如果使用者的存取權在檢閱中遭拒，則此步驟就會將其角色指派移除。  

### <a name="delete"></a>刪除
如果對檢閱不再有任何興趣，請將其刪除。 [刪除]  按鈕會將檢閱從 PIM 應用程式中移除。

## <a name="results"></a>結果
在 [結果] 索引標籤上，檢視和下載檢閱結果清單。 
[結果]![](media/azure-pim-resource-rbac/rbac-access-review-results.png) 索引標籤

## <a name="reviewers"></a>檢閱者
檢視檢閱者並將其新增至現有的存取權檢閱。 提醒檢閱者完成其檢閱。
[新增檢閱者]![](media/azure-pim-resource-rbac/rbac-access-review-reviewers.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中起始 Azure 資源角色的存取權檢閱](pim-resource-roles-start-access-review.md)
- [在 PIM 中執行 Azure 資源角色的存取權檢閱](pim-resource-roles-perform-access-review.md)
