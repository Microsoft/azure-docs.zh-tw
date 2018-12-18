---
title: SQL Server VM 的自動修補 (Resource Manager) | Microsoft Docs
description: 說明在使用 Resource Manager 的 Azure 虛擬機器中執行的 SQL Server 自動修補功能。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
editor: ''
tags: azure-resource-manager
ms.assetid: 58232e92-318f-456b-8f0a-2201a541e08d
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: jroth
ms.openlocfilehash: 398e682db6c42bd7f4864113ddf10a6a75e2b65b
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29850611"
---
# <a name="automated-patching-for-sql-server-in-azure-virtual-machines-resource-manager"></a>Azure 虛擬機器的 SQL Server 自動修補 (Resource Manager)
> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-sql-automated-patching.md)
> * [傳統](../sqlclassic/virtual-machines-windows-classic-sql-automated-patching.md)

自動修補會針對執行 SQL Server 的 Azure 虛擬機器建立維護時間範圍。 自動更新只能在此維護時間範圍內安裝。 對 SQL Server 來說，這項限制可確保系統更新及任何關聯的重新啟動都會在對資料庫而言最佳的時機發生。 

> [!IMPORTANT]
> 只會安裝標示為 [重要] 的 Windows 更新。 其他 SQL Server 更新 (例如累計更新) 必須以手動方式安裝。 

自動修補相依於 [SQL Server IaaS 代理程式擴充](virtual-machines-windows-sql-server-agent-extension.md)。

## <a name="prerequisites"></a>先決條件
若要使用自動修補，請考慮下列必要條件︰

**作業系統**：

* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

**SQL Server 版本**：

* SQL Server 2012
* SQL Server 2014
* SQL Server 2016

**Azure PowerShell**：

* [安裝最新的 Azure PowerShell 命令](/powershell/azure/overview) 。

> [!NOTE]
> 自動修補相依於 SQL Server IaaS 代理程式擴充。 目前的 SQL 虛擬機器資源庫映像預設會新增這項擴充。 如需詳細資訊，請參閱 [SQL Server IaaS 代理程式擴充](virtual-machines-windows-sql-server-agent-extension.md)。
> 
> 

## <a name="settings"></a>設定
下表說明可以為自動修補設定的選項。 實際的設定步驟會依據您是使用 Azure 入口網站或 Azure Windows PowerShell 命令而有所不同。

| 設定 | 可能的值 | 說明 |
| --- | --- | --- |
| **自動修補** |啟用/停用 (已停用) |啟用或停用 Azure 虛擬機器的自動修補。 |
| **維護排程** |每天、星期一、星期二、星期三、星期四、星期五、星期六、星期日 |虛擬機器的 Windows、SQL Server 和 Microsoft 更新的下載及安裝排程。 |
| **維護開始時間** |0-24 |更新虛擬機器的當地開始時間。 |
| **維護時間範圍** |30-180 |允許完成下載和安裝更新的分鐘數。 |
| **PATCH 類別** |重要事項 | 要下載並安裝之 Windows 更新的類別。|

## <a name="configuration-in-the-portal"></a>入口網站的組態
您可以在佈建期間或針對現有的 VM，使用 Azure 入口網站來設定「自動修補」。

### <a name="new-vms"></a>新的 VM
在 Resource Manager 部署模型中建立新的「SQL Server 虛擬機器」時，請使用 Azure 入口網站來設定「自動修補」。

在 [SQL Server 設定] 刀鋒視窗中選取 [自動修補]。 下列的 Azure 入口網站螢幕擷取畫面顯示 [SQL 自動修補]  刀鋒視窗。

![Azure 入口網站中的 SQL 自動修補](./media/virtual-machines-windows-sql-automated-patching/azure-sql-arm-patching.png)

如需相關內容，請參閱 [在 Azure 中佈建 SQL Server 虛擬機器](virtual-machines-windows-portal-sql-server-provision.md)完整主題。

### <a name="existing-vms"></a>現有的 VM
如果是現有的 SQL Server 虛擬機器，請選取您的 SQL Server 虛擬機器。 然後選取 [設定] 刀鋒視窗的 [SQL Server 組態] 區段。

![現有 VM 的 SQL 自動修補](./media/virtual-machines-windows-sql-automated-patching/azure-sql-rm-patching-existing-vms.png)

在 [SQL Server 組態] 刀鋒視窗中，按一下 [自動修補] 區段中的 [編輯] 按鈕。

![設定現有 VM 的 SQL 自動修補](./media/virtual-machines-windows-sql-automated-patching/azure-sql-rm-patching-configuration.png)

完成時，按一下 [SQL Server 組態] 刀鋒視窗底部的 [確定] 按鈕，以儲存您的變更。

如果這是您第一次啟用「自動修補」，Azure 就會在背景中設定 SQL Server IaaS Agent。 在此期間，Azure 入口網站可能不會顯示已設定自動修補。 請等候幾分鐘的時間來安裝及設定代理程式。 在那之後，Azure 入口網站就會反映新的設定。

> [!NOTE]
> 您也可以使用範本來設定「自動修補」。 如需詳細資訊，請參閱 [適用於自動修補的 Azure 快速入門範本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-autopatching-update)。
> 
> 

## <a name="configuration-with-powershell"></a>使用 PowerShell 進行設定
佈建 SQL VM 之後，請使用 PowerShell 設定自動修補。

在下列範例中，會使用 PowerShell 在現有的 SQL Server VM 上設定自動修補。 **AzureRM.Compute\New-AzureRmVMSqlServerAutoPatchingConfig** 命令會為自動更新設定新的維護時間範圍。

    $vmname = "vmname"
    $resourcegroupname = "resourcegroupname"
    $aps = AzureRM.Compute\New-AzureRmVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"

    Set-AzureRmVMSqlServerExtension -AutoPatchingSettings $aps -VMName $vmname -ResourceGroupName $resourcegroupname

> [!IMPORTANT]
> 如果擴充功能尚未安裝，安裝此擴充功能時 SQL Server 服務會重新啟動。

下表會根據此範例來描述對目標 Azure VM 的實際效果：

| 參數 | 效果 |
| --- | --- |
| **DayOfWeek** |在每個星期四安裝修補程式。 |
| **MaintenanceWindowStartingHour** |在上午 11:00 開始更新。 |
| **MaintenanceWindowsDuration** |必須在 120 分鐘內安裝修補程式。 根據開始時間，其必須在下午 1:00 之前完成。 |
| **PatchCategory** |此參數的唯一可能設定是 **Important**。 這會安裝標示為 [重要] 的 Windows 更新；它不會安裝未包含於此類別中的任何 SQL Server 更新。 |

可能需要幾分鐘的時間來安裝及設定 SQL Server IaaS 代理程式。

若要停用自動修補，請執行相同的指令碼，但不要對 **AzureRM.Compute\New-AzureRmVMSqlServerAutoPatchingConfig** 使用 **-Enable** 參數。 沒有 **-Enable** 參數時，即表示通知命令停用此功能。

## <a name="next-steps"></a>後續步驟
如需有關其他可用之自動化工作的資訊，請參閱 [SQL Server IaaS 代理程式擴充功能](virtual-machines-windows-sql-server-agent-extension.md)。

如需有關在 Azure VM 上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)。

