---
title: 使用角色型存取控制來管理 Azure Site Recovery | Microsoft Docs
description: 本文說明如何套用與使用角色型存取控制 (RBAC) 來管理 Azure Site Recovery 部署
ms.service: site-recovery
ms.workload: backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/16/2018
author: mayurigupta13
ms.topic: conceptual
ms.author: mayg
ms.openlocfilehash: 9ec9acab4ad83af22504058c21b102df94b8692d
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353232"
---
# <a name="use-role-based-access-control-to-manage-site-recovery-access"></a>使用角色型存取控制管理 Site Recovery 存取

Azure 角色型存取控制 (RBAC) 可以對 Azure 進行更細緻的存取權管理。 您可以使用 RBAC 劃分小組責任，並只將特定存取權限授與需要執行特定工作的使用者。

Azure Site Recovery 提供 3 種內建角色，以控制 Site Recovery 管理作業。 深入了解 [Azure RBAC 內建角色](../role-based-access-control/built-in-roles.md)

* [Site Recovery 參與者](../role-based-access-control/built-in-roles.md#site-recovery-contributor)：此角色具有在復原服務保存庫中管理 Azure Site Recovery 作業所需的所有權限。 不過，具有此角色的使用者無法建立或刪除復原服務保存庫，也無法為其他使用者指派存取權限。 此角色最適合災害復原系統管理員，他們可以為應用程式或整個組織 (視情況而定) 啟用和管理災害復原。
* [Site Recovery 操作員](../role-based-access-control/built-in-roles.md#site-recovery-operator) - 此角色具有執行和管理容錯移轉和容錯回復作業的權限。 具有此角色的使用者無法啟用或停用複寫、建立或刪除保存庫、註冊新的基礎結構，也無法為其他使用者指派存取權限。 此角色最適合災害復原操作員，當應用程式擁有者和 IT 系統管理員在實際或模擬災害情況 (例如災害復原演習) 中指示時，操作員可以對虛擬機器或應用程式進行容錯移轉。 災害解決後，災害復原操作員可以重新保護和容錯回復虛擬機器。
* [Site Recovery 讀者](../role-based-access-control/built-in-roles.md#site-recovery-reader)：此角色擁有可檢視所有 Site Recovery 管理作業的權限。 此角色最適合 IT 監督主管，以便監控目前的保護狀態，並在需要時提出支援票證。

如果您想要定義自己的角色以獲得更進一步控制，請參閱如何在 Azure 中[建立自訂角色](../role-based-access-control/custom-roles.md)。

## <a name="permissions-required-to-enable-replication-for-new-virtual-machines"></a>啟用新虛擬機器複寫所需的權限
當使用 Azure Site Recovery 將新的虛擬機器複寫至 Azure 時，系統會驗證相關聯使用者的存取層級，以確定使用者擁有使用提供給 Site Recovery 的 Azure 資源所需的權限。

若要啟用新虛擬機器的複寫，使用者必須擁有：
* 在所選資源群組中建立虛擬機器的權限
* 在所選虛擬網路中建立虛擬機器的權限
* 寫入所選儲存體帳戶的權限

使用者需要下列權限才能完成新虛擬機器的複寫。

> [!IMPORTANT]
>確定為每個用於部署資源的部署模型 (Resource Manager/傳統) 新增相關的權限。

| **資源類型** | **部署模型** | **權限** |
| --- | --- | --- |
| 計算 | Resource Manager | Microsoft.Compute/availabilitySets/read |
|  |  | Microsoft.Compute/virtualMachines/read |
|  |  | Microsoft.Compute/virtualMachines/write |
|  |  | Microsoft.Compute/virtualMachines/delete |
|  | 傳統 | Microsoft.ClassicCompute/domainNames/read |
|  |  | Microsoft.ClassicCompute/domainNames/write |
|  |  | Microsoft.ClassicCompute/domainNames/delete |
|  |  | Microsoft.ClassicCompute/virtualMachines/read |
|  |  | Microsoft.ClassicCompute/virtualMachines/write |
|  |  | Microsoft.ClassicCompute/virtualMachines/delete |
| 網路 | Resource Manager | Microsoft.Network/networkInterfaces/read |
|  |  | Microsoft.Network/networkInterfaces/write |
|  |  | Microsoft.Network/networkInterfaces/delete |
|  |  | Microsoft.Network/networkInterfaces/join/action |
|  |  | Microsoft.Network/virtualNetworks/read |
|  |  | Microsoft.Network/virtualNetworks/subnets/read |
|  |  | Microsoft.Network/virtualNetworks/subnets/join/action |
|  | 傳統 | Microsoft.ClassicNetwork/virtualNetworks/read |
|  |  | Microsoft.ClassicNetwork/virtualNetworks/join/action |
| 儲存體 | Resource Manager | Microsoft.Storage/storageAccounts/read |
|  |  | Microsoft.Storage/storageAccounts/listkeys/action |
|  | 傳統 | Microsoft.ClassicStorage/storageAccounts/read |
|  |  | Microsoft.ClassicStorage/storageAccounts/listKeys/action |
| 資源群組 | Resource Manager | Microsoft.Resources/deployments/* |
|  |  | Microsoft.Resources/subscriptions/resourceGroups/read |

請考慮分別為 Resource Manager 與傳統部署模型使用「虛擬機器參與者」與「傳統虛擬機器參與者」[內建角色](../role-based-access-control/built-in-roles.md)。

## <a name="next-steps"></a>後續步驟
* [角色型存取控制](../role-based-access-control/role-assignments-portal.md)：開始在 Azure 入口網站中使用 RBAC。
* 了解如何使用下列各項管理存取權：
  * [PowerShell](../role-based-access-control/role-assignments-powershell.md)
  * [Azure CLI](../role-based-access-control/role-assignments-cli.md)
  * [REST API](../role-based-access-control/role-assignments-rest.md)
* [角色型存取控制疑難排解](../role-based-access-control/troubleshooting.md)︰取得修正常見問題的建議。
