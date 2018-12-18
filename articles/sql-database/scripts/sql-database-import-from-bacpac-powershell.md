---
title: PowerShell 範例：將 BACPAC 檔案匯入 Azure SQL Database | Microsoft Docs
description: 將 BASPAC 檔案匯入 SQL Database 的 Azure PowerShell 範例指令碼
services: sql-database
documentationcenter: sql-database
author: CarlRabeler
manager: craigg
editor: carlrab
tags: azure-service-management
ms.assetid: ''
ms.service: sql-database
ms.custom: load & move data, mvc
ms.devlang: PowerShell
ms.topic: sample
ms.tgt_pltfrm: sql-database
ms.workload: database
ms.date: 07/24/2018
ms.author: carlrab
ms.openlocfilehash: 86042b631fd1f621f3db84bed981a744cebff522
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39262884"
---
# <a name="use-powershell-to-import-a-bacpac-file-into-an-azure-sql-database"></a>使用 PowerShell 將 BACPAC 檔案匯入 Azure SQL Database

此 PowerShell 指令碼範例會從 BACPAC 檔案將資料庫匯入 Azure SQL Database。  

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/sql-database/import-from-bacpac/import-from-bacpac.ps1?highlight=18-19 "Create SQL Database")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmSqlServer](/powershell/module/azurerm.sql/new-azurermsqlserver) | 建立主控 SQL Database 的邏輯伺服器。 |
| [New-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/new-azurermsqlserverfirewallrule) | 建立防火牆規則以允許從輸入的 IP 位址範圍存取伺服器上的所有 SQL Database。 |
| [New-AzureRmSqlDatabaseImport (英文)](/powershell/module/azurerm.sql/new-azurermsqldatabaseimport) | 在伺服器上匯入 BACPAC 檔案並建立新的資料庫。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

其他的 SQL Database PowerShell 指令碼範例可於 [Azure SQL Database PowerShell 指令碼](../sql-database-powershell-samples.md)中找到。
