---
title: 在 PIM 中探索要管理的 Azure 資源 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中探索要管理的 Azure 資源。
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
ms.date: 08/30/2018
ms.author: rolyon
ms.openlocfilehash: d9a6ab49d619e487eee6fb13abe128cfc167b560
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306684"
---
# <a name="discover-azure-resources-to-manage-in-pim"></a>在 PIM 中探索要管理的 Azure 資源

您可以使用 Azure AD Privileged Identity Management (PIM)，來改善您 Azure 資源的保護。 這有助於已經使用 PIM 的組織來保護 Azure AD 目錄角色，並管理想要保護生產資源的群組和訂用帳戶擁有者。

當您第一次設定適用於 Azure 資源的 PIM 時，必須探索並選取要使用 PIM 保護的資源。 您可以使用 PIM 管理的資源數目並無任何限制。 不過，我們建議從您最重要的 (生產) 資源開始。

## <a name="discover-resources"></a>探索資源

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure 資源]。

    如果這是您第一次使用適用於 Azure 資源的 PIM，您會看到 [探索資源] 窗格。

    ![探索資源 - 第一次](./media/pim-resource-roles-discover-resources/discover-resources-first-run.png)

    如果您組織中的其他資源或目錄系統管理員已經管理 PIM 中的 Azure 資源，您會看到將目前管理的資源列出的清單。

    ![[探索資源] 窗格](./media/pim-resource-roles-discover-resources/discover-resources.png)

1. 按一下 [探索資源] 啟動探索體驗。

    ![[探索] 窗格](./media/pim-resource-roles-discover-resources/discovery-pane.png)

1. 在 [探索] 窗格中，使用 [資源狀態篩選] 和 [選取資源類型] 篩選您擁有寫入權限的管理群組或訂用帳戶。 一開始可能最容易的做法是先使用 [全部]。

    您只能使用 PIM 來搜尋並選取要管理的管理群組或訂用帳戶資源。 當您在 PIM 中對管理群組或訂用帳戶進行管理時，也可以管理其中的子資源。

1. 勾選您想要管理的任何未受控資源。

    > [!NOTE]
    > 管理群組或訂用帳戶設定為受控後，則無法變為非受控。 這可防止其他資源管理員移除 PIM 設定。

    ![探索 - 管理資源](./media/pim-resource-roles-discover-resources/discovery-manage-resource.png)

1. 按一下 [管理資源] 開始管理選取的資源。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中設定 Azure 資源角色設定](pim-resource-roles-configure-role-settings.md)
- [在 PIM 中指派 Azure 資源角色](pim-resource-roles-assign-roles.md)
