---
title: 開始使用 PIM - Azure | Microsoft Docs
description: 了解如何在 Azure 入口網站中開始使用 Azure AD Privileged Identity Management (PIM)。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: pim
ms.topic: conceptual
ms.workload: identity
ms.date: 08/27/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 5b3bff27821964648713b02589c941c99e3eb03d
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190084"
---
# <a name="start-using-pim"></a>開始使用 PIM

您可以使用 Azure Active Directory (Azure AD) Privileged Identity Management (PIM) 來管理、控制和監視組織內的存取。 此範圍包括存取 Azure 資源、Azure AD 和其他 Microsoft 線上服務 (例如 Office 365 或 Microsoft Intune)。

本文會告訴您如何將 Azure AD PIM 應用程式加入 Azure 入口網站儀表板中。

## <a name="first-person-to-use-pim"></a>使用 PIM 的第一個人

如果您是目錄中使用 PIM 的第一人，則會自動獲指派目錄中的「安全性系統管理員」[](../users-groups-roles/directory-assign-admin-roles.md#security-administrator)和「特殊權限角色管理員」[](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator)角色。 只有特殊權限角色管理員才能管理使用者的 Azure AD Directory 角色指派。 此外，您還可以選擇執行[安全性精靈](pim-security-wizard.md) 引導您完成初始探索和指派體驗。

## <a name="add-pim-tile-to-the-dashboard"></a>將 PIM 圖格新增至儀表板

若要更輕鬆地開啟 PIM，您應將 PIM 圖格新增至 Azure 入口網站儀表板。

1. 以目錄的全域系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

1. 按一下 [所有服務]，並尋找 [Azure AD Privileged Identity Management] 服務。

    ![所有服務中的 Azure AD Privileged Identity Management](./media/pim-getting-started/pim-all-services-find.png)

1. 按一下以開啟 PIM 快速入門。

1. 請參閱**將刀鋒視窗釘選到儀表板**，將 [PIM 快速入門] 刀鋒視窗釘選到儀表板上。

    ![將刀鋒視窗釘選到儀表板](./media/pim-getting-started/pim-quickstart-pin-to-dashboard.png)

    在 Azure 儀表板上，您會看到像這樣的圖格：

    ![PIM 快速入門圖格](./media/pim-getting-started/pim-quickstart-dashboard-tile.png)

## <a name="navigate-to-your-tasks"></a>瀏覽至您的工作

PIM 設定完成後，您可以使用此刀鋒視窗來完成身分識別管理工作。

![PIM 的最上層工作 - 螢幕擷取畫面](./media/pim-getting-started/pim-quickstart-tasks.png)

| 工作 + 管理 | 說明 |
| --- | --- |
| **我的角色**  | 顯示一個清單，列出已指派給您的合格和使用中角色。 您可以在這裡啟動任何指派的合格角色。 |
| **我的要求** | 顯示您啟動合格角色指派的任何擱置要求。 |
| **應用程式存取** | 可讓您減少潛在的延遲，並在啟用後立即使用角色。 |
| **核准要求** | 顯示一個清單，列出您目錄中的使用者為了啟動合格角色所提出的要求，而此目錄是指定給您以進行核准的目錄。 |
| **檢閱存取** | 列出指派給您完成的使用中存取權檢閱 (無論您是在檢閱自己還是他人的存取權)。 |
| **Azure AD 目錄角色** | 顯示儀表板和設定，讓特殊權限角色管理員可管理 Azure AD Directory 角色指派。 對於任何不是特殊權限角色管理員的人員，系統會停用此儀表板。 這些使用者可以存取標題為 [我的檢視] 的特殊儀表板。 [我的檢視] 儀表板只會顯示存取儀表板的使用者，而非整個租用戶的相關資訊。 |
| **Azure 資源** | 顯示儀表板和設定，讓特殊權限角色管理員可管理 Azure 資源角色指派。 對於任何不是特殊權限角色管理員的人員，系統會停用此儀表板。 這些使用者可以存取標題為 [我的檢視] 的特殊儀表板。 [我的檢視] 儀表板只會顯示存取儀表板的使用者，而非整個租用戶的相關資訊。 |

## <a name="next-steps"></a>後續步驟

- [在 PIM 中啟用我的 Azure AD 目錄角色](pim-how-to-activate-role.md)
- [在 PIM 中啟用我的 Azure 資源角色](pim-resource-roles-activate-your-roles.md)
