---
title: 使用 RBAC 共用 Azure 入口網站儀表板 | Microsoft Docs
description: 本文說明如何在 Azure 入口網站中使用角色型存取控制來共用儀表板。
services: azure-portal
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 8908a6ce-ae0c-4f60-a0c9-b3acfe823365
ms.service: azure-portal
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 08/01/2016
ms.author: tomfitz
ms.openlocfilehash: c07a9d92cac13d6325e66f44426f1a64e8ac53cb
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35766807"
---
# <a name="share-azure-dashboards-by-using-role-based-access-control"></a>使用角色型存取控制來共用 Azure 儀表板
設定儀表板之後，您可以將它發佈，並與組織中的其他使用者共用。 使用 Azure [角色型存取控制](../role-based-access-control/role-assignments-portal.md)，您可允許其他人檢視您的儀表板。 您可以將一名使用者或一群使用者指派給某個角色，該角色就會定義這些使用者可以檢視或修改已發佈的儀表板。 

所有已發佈的儀表板會實作為 Azure 資源，這表示它們是做為您訂用帳戶內可管理的項目而存在，並且會包含在資源群組中。  從存取控制的觀點來看，儀表板與其他資源 (例如虛擬機器或儲存體帳戶) 並無不同。

> [!TIP]
> 儀表板上的個別圖格會根據其顯示的資源，強制執行自己的存取控制需求。  因此，您可以設計一個廣泛共用且同時仍會保護個別圖格上資料的儀表板。
> 
> 

## <a name="understanding-access-control-for-dashboards"></a>了解儀表板的存取控制
若使用角色型存取控制 (RBAC)，您可以將使用者指派給三個不同範圍層級的角色︰

* 訂用帳戶
* 資源群組
* resource

您指派的權限會從訂用帳戶往下繼承到資源。 發佈的儀表板是一種資源。 因此，您可能已將使用者指派給訂用帳戶的角色，而這也對已發佈的儀表板有效。 

範例如下。  假設您有 Azure 訂用帳戶，而且已為小組各種成員指派訂用帳戶的**擁有者**、**參與者**或**讀取者**角色。 身為擁有者或參與者的使用者能夠列出、檢視、建立、修改或刪除訂用帳戶內的儀表板。  身為讀取者的使用者能夠列出和檢視儀表板，但無法進行修改或刪除。  具有讀取者存取權的使用者能夠對已發佈的儀表板進行本機編輯 (例如在針對問題進行疑難排解時)，但無法將這些變更發佈回伺服器。  他們會有為自己製作儀表板的私人複本的選項。

不過，您也可以指派權限給包含數個儀表板的資源群組或指派給個別儀表板。 例如，您可能會決定，一群使用者應具有有限的訂用帳戶權限，但對特定儀表板則具有更大範圍的存取權。 您可以將這些使用者指派給該儀表板的角色。 

## <a name="publish-dashboard"></a>發佈儀表板
讓我們假設您已完成設定想要與訂用帳戶中的一群使用者共用的儀表板。 下列步驟描述稱為「儲存體管理員」的自訂群組，但您可以將您的群組命名為任何您喜歡的名稱。 如需建立 Active Directory 群組和將使用者新增至該群組的相關資訊，請參閱 [在 Azure Active Directory 中管理群組](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。

1. 在儀表板中選取 [共用] 。
   
     ![選取共用](./media/azure-portal-dashboard-share-access/select-share.png)
2. 在指派存取權之前，您必須先發佈儀表板。 根據預設，儀表板會發佈到名為 **儀表板**的資源群組。 選取 [發佈] 。
   
     ![publish](./media/azure-portal-dashboard-share-access/publish.png)

儀表板現已發佈。 如果繼承自訂用帳戶的權限合適，則不需要再執行任何動作。 組織中的其他使用者將可以根據其訂用帳戶層級的角色來存取和修改儀表板。 不過，在本教學課程中，讓我們將一群使用者指派給該儀表板的角色。

## <a name="assign-access-to-a-dashboard"></a>指派儀表板存取權
1. 在發佈儀表板之後，選取 [管理使用者] 。
   
     ![管理使用者](./media/azure-portal-dashboard-share-access/manage-users.png)
2. 您會看到已對其指派此儀表板角色的現有使用者清單。 您的現有使用者清單會不同於下面的影像。 指派很可能是繼承自訂用帳戶。 若要新增使用者或群組，請選取 [新增] 。
   
     ![新增使用者](./media/azure-portal-dashboard-share-access/existing-users.png)
3. 選取代表您想要授與之權限的角色。 在此範例中，請選取 [參與者] 。
   
     ![選取角色](./media/azure-portal-dashboard-share-access/select-role.png)
4. 選取想要指派給角色的使用者或群組。 如果您在清單中沒有看見要尋找的使用者或群組，請使用搜尋方塊。 您的可用群組清單取決於您已在 Active Directory 中建立的群組。
   
     ![選取使用者](./media/azure-portal-dashboard-share-access/select-user.png) 
5. 使用者或群組新增完成時，請選取 [確定] 。 
6. 新的指派就會加入至使用者清單。 請注意，其**存取權**會列為 [已指派] 而非 [已繼承]。
   
     ![指派的角色](./media/azure-portal-dashboard-share-access/assigned-roles.png)

## <a name="next-steps"></a>後續步驟
* 如需角色清單，請參閱 [RBAC︰內建角色](../role-based-access-control/built-in-roles.md)。
* 若要了解如何管理資源，請參閱 [透過入口網站管理 Azure 資源](resource-group-portal.md)。

