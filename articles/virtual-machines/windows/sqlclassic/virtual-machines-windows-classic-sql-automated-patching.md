---
title: SQL Server VM 的自動修補 (傳統) | Microsoft Docs
description: 針對 Azure 中以傳統部署模式執行的 SQL Server 虛擬機器，說明自動修補功能。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: 737b2f65-08b9-4f54-b867-e987730265a8
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: jroth
ms.openlocfilehash: 4bb13ec9d835959273801c2f53c8d736491080a6
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061330"
---
# <a name="automated-patching-for-sql-server-in-azure-virtual-machines-classic"></a>Azure 虛擬機器中的 SQL Server 自動修補 (傳統)
> [!div class="op_single_selector"]
> * [Resource Manager](../sql/virtual-machines-windows-sql-automated-patching.md)
> * [傳統](../classic/sql-automated-patching.md)
> 
> 

自動修補會針對執行 SQL Server 的 Azure 虛擬機器建立維護時間範圍。 自動更新只能在此維護時間範圍內安裝。 對於 SQL Server，這可以確保系統更新和任何相關聯的重新啟動會在對資料庫最好的時間發生。 

> [!IMPORTANT]
> 只會安裝標示為 [重要] 的 Windows 更新。 其他 SQL Server 更新 (例如累計更新) 必須以手動方式安裝。 

自動修補相依於 [SQL Server IaaS 代理程式擴充](../classic/sql-server-agent-extension.md)。

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 若要檢視這篇文章的 Resource Manager 版本，請參閱 [Automated Patching for SQL Server in Azure Virtual Machines Resource Manager](../sql/virtual-machines-windows-sql-automated-patching.md)(Azure 虛擬機器的 SQL Server 自動修補 (Resource Manager))。

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

* [安裝最新的 Azure PowerShell 命令](/powershell/azure/overview)。

**SQL Server IaaS 擴充功能**：

* [安裝 SQL Server IaaS 擴充功能](../classic/sql-server-agent-extension.md)。

## <a name="settings"></a>設定
下表說明可以為自動修補設定的選項。 針對傳統 VM，您必須使用 PowerShell 來設定這些設定。

| 設定 | 可能的值 | 說明 |
| --- | --- | --- |
| **自動修補** |啟用/停用 (已停用) |啟用或停用 Azure 虛擬機器的自動修補。 |
| **維護排程** |每天、星期一、星期二、星期三、星期四、星期五、星期六、星期日 |虛擬機器的 Windows、SQL Server 和 Microsoft 更新的下載及安裝排程。 |
| **維護開始時間** |0-24 |更新虛擬機器的當地開始時間。 |
| **維護時間範圍** |30-180 |允許完成下載和安裝更新的分鐘數。 |
| **PATCH 類別** |重要事項 |要下載並安裝的更新類別。 |

## <a name="configuration-with-powershell"></a>使用 PowerShell 進行設定
在下列範例中，會使用 PowerShell 在現有的 SQL Server VM 上設定自動修補。 **New-AzureVMSqlServerAutoPatchingConfig** 命令會設定自動更新的維護時間範圍。

    $aps = New-AzureVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoPatchingSettings $aps | Update-AzureVM

下表會根據此範例來描述對目標 Azure VM 的實際效果：

| 參數 | 效果 |
| --- | --- |
| **DayOfWeek** |在每個星期四安裝修補程式。 |
| **MaintenanceWindowStartingHour** |在上午 11:00 開始更新。 |
| **MaintenanceWindowDuration** |必須在 120 分鐘內安裝修補程式。 根據開始時間，其必須在下午 1:00 之前完成。 |
| **PatchCategory** |此參數唯一可能的設定是 "Important"。 |

可能需要幾分鐘的時間來安裝及設定 SQL Server IaaS 代理程式。

若要停用自動修補，請執行相同的指令碼，但不要對 New-AzureVMSqlServerAutoPatchingConfig 使用 -Enable 參數。 和安裝一樣，可能需要幾分鐘的時間來停用自動修補。

## <a name="next-steps"></a>後續步驟
如需有關其他可用之自動化工作的資訊，請參閱 [SQL Server IaaS 代理程式擴充功能](../classic/sql-server-agent-extension.md)。

如需有關在 Azure VM 上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](../sql/virtual-machines-windows-sql-server-iaas-overview.md)。

