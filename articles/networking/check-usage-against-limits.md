---
title: 根據限制檢查 Azure 資源使用量 | Microsoft Docs
description: 瞭解如何根據 Azure 訂用帳戶限制來檢查您的 Azure 資源使用量。
services: networking
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: networking
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/05/2018
ms.author: jdial
ms.openlocfilehash: d57f34da261d51707caf499bfdf1fffa344b72a9
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385510"
---
# <a name="check-resource-usage-against-limits"></a>根據限制檢查資源使用量

在本文中，您將學習如何查看訂用帳戶中已部署之各種網路資源類型的數量，以及您的[訂用帳戶限制](../azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits)為何。 如果能根據限制查看資源使用量，便有助於追蹤目前的使用量以及規劃未來的用途。 您可以使用 [Azure 入口網站](#azure-portal)、[PowerShell](#powershell) 或 [Azure CLI](#azure-cli) 來追蹤使用量。

## <a name="azure-portal"></a>Azure 入口網站

1. 登入 Azure [入口網站](https://portal.azure.com)。
2. 在 Azure 入口網站的左上角，選取 [所有服務]。
3. 在 [篩選條件] 方塊中輸入「訂用帳戶」。 當**訂用帳戶**出現在搜尋結果中時，選取它。
4. 選取您想要查看其使用量資訊的訂用帳戶名稱。
5. 在 [設定] 下方，選取 [使用量 + 配額]。
6. 您可以選取下列選項：
    - **資源類型**：您可以選取所有的資源類型，或選取您想要查看的具體資源類型。
    - **提供者**：您可以選取所有資源提供者，或選取 [計算]、[網路] 或 [儲存體]。
    - **位置**：您可以選取所有的 Azure 位置，或選取特定的位置。
    - 您可以選擇顯示所有的資源，或者顯示的資源中，至少有一個被部署在某處。

    下圖中的範例所顯示的所有網路資源，至少有一個是部署在美國東部：

        ![View usage data](./media/check-usage-against-limits/view-usage.png)

    您可以選取資料行標題來排序資料行。 顯示的限制是訂用帳戶的限制。 如果您要增加預設限制，請選取 [要求增加]，然後填寫並提交支援要求。 所有資源都有一個最大限制，分別列示在 Azure [限制](../azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits)中。 如果您目前的限制已經是最大值，則無法提高限制。

## <a name="powershell"></a>PowerShell

您可以執行 [Azure Cloud Shell](https://shell.azure.com/powershell) 中採用的命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式殼層。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 如果您是從電腦執行 PowerShell，便需要 *AzureRM* PowerShell 模組 6.0.1 版或更新的版本。 請在您的電腦上執行 `Get-Module -ListAvailable AzureRM`，以尋找已安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，也需要執行 `Login-AzureRmAccount` 來登入 Azure。

使用 [Get AzureRmNetworkUsage](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermnetworkusage?view=azurermps-6.8.0) 來根據限制查看您的使用量。 下列範例所取得的資源使用量中，至少有一個資源是部署在美國東部地區：

```azurepowershell-interactive
Get-AzureRmNetworkUsage `
  -Location eastus `
  | Where-Object {$_.CurrentValue -gt 0} `
  | Format-Table ResourceType, CurrentValue, Limit
```

您收到的輸出，其格式會與下面的範例輸出相同：

```powershell
ResourceType            CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
```

## <a name="azure-cli"></a>Azure CLI

如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本文需要 Azure CLI 2.0.32 版或更新的版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您是在本機執行 Azure CLI，也需要執行 `az login` 來登入 Azure。

使用 [az network list-usages](/cli/azure/network?view=azure-cli-latest#az-network-list-usages) 來根據限制查看您的使用量。 下列範例會取得美國東部地區的資源使用量：

```azurecli-interactive
az network list-usages \
  --location eastus \
  --out table
```

您收到的輸出，其格式會與下面的範例輸出相同：

```azurecli
Name                    CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
Load Balancers                     0   100
Application Gateways               0    50
```
