---
title: Azure PowerShell 指令碼 - Azure Cosmos DB 的多區域複寫 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - Azure Cosmos DB 的多區域複寫
services: cosmos-db
documentationcenter: cosmosdb
author: SnehaGunda
manager: kfile
tags: azure-service-management
ms.service: cosmos-db
ms.custom: mvc
ms.devlang: PowerShell
ms.topic: sample
ms.tgt_pltfrm: cosmosdb
ms.workload: database
ms.date: 05/10/2017
ms.author: sngun
ms.openlocfilehash: 99d6a0852fbdb031b34abca433569f9297ecd740
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41919487"
---
# <a name="replicate-an-azure-cosmos-db-database-account-in-multiple-regions-and-configure-failover-priorities-using-powershell"></a>使用 PowerShell 複寫多個區域中的 Azure Cosmos DB 資料庫帳戶和設定容錯移轉優先順序

此範例會使用 PowerShell 複寫多個區域中的任何一種 Azure Cosmos DB 資料庫帳戶，並設定容錯移轉優先順序。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/cosmosdb/replicate-database-multiple-regions/replicate-database-multiple-regions.ps1?highlight=37-44,47-48,51-55 "Replicate an Azure Cosmos DB account across multiple regions")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmResource](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresource?view=azurermps-3.8.0) | 建立主機資料庫或彈性集區的邏輯伺服器。 |
| [Set-AzureRMResource](https://docs.microsoft.com/powershell/module/azurerm.resources/set-azurermresource?view=azurermps-3.8.0) | 修改資料庫帳戶。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure Cosmos DB PowerShell 指令碼](../powershell-samples.md)中找到其他 Azure Cosmos DB PowerShell 指令碼範例。