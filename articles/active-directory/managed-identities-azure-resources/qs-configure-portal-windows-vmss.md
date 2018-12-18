---
title: 如何在虛擬機器擴展集上設定 Azure 資源受控識別
description: 使用 Azure 入口網站在虛擬機器擴展集上設定 Azure 資源受控識別的逐步指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/20/2018
ms.author: daveba
ms.openlocfilehash: 0a6440b7fffe1aec26ba4755f21fa2f56935887e
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44377576"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-virtual-machine-scale-set-using-the-azure-portal"></a>使用 Azure 入口網站在虛擬機器擴展集上設定 Azure 資源受控識別

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Azure 資源受控識別會在 Azure Active Directory 中為 Azure 服務提供自動受控識別。 您可以使用此身分識別來向任何支援 Azure AD 驗證的服務進行驗證，不需要任何您程式碼中的認證。 

在本文中，您將了解如何使用 PowerShell，在虛擬機器擴展集上執行下列 Azure 資源受控識別作業：

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 若要執行本文中的管理作業，您的帳戶需要下列 Azure 角色型存取控制指派：

    > [!NOTE]
    > 不需要其他 Azure AD 目錄角色指派。

    - [虛擬機器參與者](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor)，可從虛擬機器擴展集啟用和移除系統指派的受控識別。

## <a name="system-assigned-managed-identity"></a>系統指派的受控識別

在本節中，您會了解如何使用 Azure 入口網站，啟用和停用系統指派的受控識別。

### <a name="enable-system-assigned-managed-identity-during-creation-of-a-virtual-machine-scale-set"></a>在虛擬機器擴展集建立期間啟用系統指派的受控識別

目前，Azure 入口網站不支援在建立虛擬機器擴展集時啟用系統指派的受控識別。 相反地，請參閱下列虛擬機器擴展集建立快速入門文章，來先建立虛擬機器擴展集，然後繼續進行下一節，以了解如何在虛擬機器擴展集上啟用系統指派的受控識別：

- [在 Azure 入口網站建立虛擬機器擴展集](../../virtual-machine-scale-sets/quick-create-portal.md)  

### <a name="enable-system-assigned-managed-identity-on-an-existing-virtual-machine-scale-set"></a>在現有虛擬機器擴展集上啟用系統指派的受控識別

若要在原本佈建時沒有使用系統指派受控識別的虛擬機器擴展集上啟用該識別：

1. 使用與包含虛擬機器擴展集的 Azure 訂用帳戶相關聯的帳戶，登入 [Azure 入口網站](https://portal.azure.com)。

2. 瀏覽到需要的虛擬機器擴展集。

3. 在 [系統指派]、[狀態] 底下，選取 [開啟]，然後按一下 [儲存]：

   ![組態頁面螢幕擷取畫面](./media/msi-qs-configure-portal-windows-vmss/create-windows-vmss-portal-configuration-blade.png)](./media/msi-qs-configure-portal-windows-vmss/create-windows-vmss-portal-configuration-blade.png#lightbox)  

### <a name="remove-system-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>從虛擬機器擴展集移除系統指派的受控識別

如果您的虛擬機器擴展集不再需要系統指派的受控識別：

1. 使用與包含虛擬機器擴展集的 Azure 訂用帳戶相關聯的帳戶，登入 [Azure 入口網站](https://portal.azure.com)。 此外也請確定您的帳戶屬於在虛擬機器擴展集上具有寫入權限的角色。

2. 瀏覽到需要的虛擬機器擴展集。

3. 在 [系統指派]、[狀態] 底下，選取 [關閉]，然後按一下 [儲存]：

   ![組態頁面螢幕擷取畫面](./media/msi-qs-configure-portal-windows-vmss/disable-windows-vmss-portal-configuration-blade.png)

## <a name="user-assigned-managed-identity"></a>使用者指派的受控識別

在本節中，您會了解如何使用 Azure 入口網站，從虛擬機器擴展集新增和移除使用者指派的受控識別。

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-a-virtual-machine-scale-set"></a>在虛擬機器擴展集建立期間指派使用者指派的受控識別

目前，Azure 入口網站不支援在建立虛擬機器擴展集時指派使用者指派的受控識別。 相反地，請參閱下列虛擬機器擴展集建立快速入門文章，來先建立虛擬機器擴展集，然後繼續進行下一節，以了解如何對它指派使用者指派的受控識別：

- [在 Azure 入口網站建立虛擬機器擴展集](../../virtual-machine-scale-sets/quick-create-portal.md)

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-virtual-machine-scale-set"></a>將使用者指派的受控識別指派給現有的虛擬機器擴展集

1. 使用與包含虛擬機器擴展集的 Azure 訂用帳戶相關聯的帳戶，登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至所需虛擬機器擴展集，然後依序按一下 [身分識別]、[使用者指派] 和 [\+新增]。

   ![對 VMSS 新增使用者指派的身分識別](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vmss-screenshot1.png)

3. 按一下想要新增到虛擬機器擴展集的使用者指派身分識別，然後按一下 [新增]。
   
   ![對 VMSS 新增使用者指派的身分識別](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vm-screenshot2.png)

### <a name="remove-a-user-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>從虛擬機器擴展集移除使用者指派的受控識別

1. 使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶，登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至所需虛擬機器擴展集，按一下 [身分識別]、[使用者指派] 和想要刪除的使用者指派受控識別名稱，然後按一下 [移除] (按一下 [確認] 窗格中的 [是])。

   ![從 VMSS 中移除使用者指派的身分識別](./media/msi-qs-configure-portal-windows-vm/remove-user-assigned-identity-vmss-screenshot.png)


## <a name="next-steps"></a>後續步驟

- 使用 Azure 入口網站，提供虛擬機器擴展集的受控識別[存取權給另一個 Azure 資源](howto-assign-access-portal.md)。


