---
title: PowerShell 範例-異地複寫容錯移轉群組-單一 Azure SQL Database | Microsoft Docs
description: 為單一 Azure SQL Database 設定作用中異地複寫容錯移轉群組並且進行容錯移轉的 Azure PowerShell 範例指令碼。
services: sql-database
documentationcenter: sql-database
author: CarlRabeler
manager: craigg
editor: carlrab
tags: azure-service-management
ms.assetid: ''
ms.service: sql-database
ms.custom: business continuity, mvc
ms.devlang: PowerShell
ms.topic: sample
ms.tgt_pltfrm: sql-database
ms.workload: database
ms.date: 04/01/2018
ms.author: carlrab
ms.openlocfilehash: 1bb5cf1eeb744bacb0a0324923f3389affea9ee7
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34364211"
---
# <a name="use-powershell-to-configure-an-active-geo-replication-failover-group-for-a-single-azure-sql-database"></a>使用 PowerShell 為單一 Azure SQL Database 設定作用中異地複寫容錯移轉群組

此 PowerShell 指令碼範例設定單一 Azure SQL Database 的作用中異地複寫容錯移轉群組，並將其容錯移轉到 Azure SQL Database 的次要複本。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-scripts"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/sql-database/setup-geodr-and-failover-database/setup-geodr-and-failover-database-failover-group.ps1?highlight=19-22 "Set up failover group for single database")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $primaryresourcegroupname
Remove-AzureRmResourceGroup -ResourceGroupName $secondaryresourcegroupname
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmSqlServer](/powershell/module/azurerm.sql/new-azurermsqlserver) | 建立主機資料庫或彈性集區的邏輯伺服器。 |
| [New-AzureRmSqlElasticPool](/powershell/module/azurerm.sql/new-azurermsqlelasticpool) | 在邏輯伺服器內建立彈性集區。 |
| [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) | 更新資料庫屬性，或將資料庫移入、移出彈性集區或在彈性集區之間移動資料庫。 |
| [New-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/new-azurermsqldatabasesecondary)| 針對現有資料庫建立次要資料庫並開始資料複寫。 |
| [Get-AzureRmSqlDatabase](/powershell/module/azurerm.sql/get-azurermsqldatabase)| 取得一或多個資料庫。 |
| [Set-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/set-azurermsqldatabasesecondary)| 將次要資料庫切換為主要資料庫以開始容錯移轉。|
| [Get-AzureRmSqlDatabaseReplicationLink](/powershell/module/azurerm.sql/get-azurermsqldatabasereplicationlink) | 取得 Azure SQL Database 和資源群組或 SQL Server 之間的異地複寫連結。 |
| [Remove-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/remove-azurermsqldatabasesecondary) | 終止 SQL Database 和指定次要資料庫間的資料複寫。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |
| [New-AzureRMSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/new-azurermsqldatabasefailovergroup) | 為指定的伺服器建立新的 Azure SQL Database 容錯移轉群組。 |
| [Switch-AzureRMSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/switch-azurermsqldatabasefailovergroup) | 切換容錯移轉群組中的伺服器角色，並將所有次要資料庫切換成主要角色。 |
| [Get-AzureRMSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/get-azurermsqldatabasefailovergroup) | 取得特定的 Azure SQL Database 容錯移轉群組，或列出伺服器上的容錯移轉群組。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

其他的 SQL Database PowerShell 指令碼範例可於 [Azure SQL Database PowerShell 指令碼](../sql-database-powershell-samples.md)中找到。
