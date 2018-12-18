---
title: 檢視 Azure 虛擬網路拓撲 | Microsoft Docs
description: 了解如何檢視虛擬網路中的資源，以及資源之間的關聯性。
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2018
ms.author: jdial
ms.openlocfilehash: e5e9901d6265b48a7b57cdf2c146ebb623ad5c3d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992197"
---
# <a name="view-the-topology-of-an-azure-virtual-network"></a>檢視 Azure 虛擬網路的拓撲

在本文中，您將了解如何檢視 Microsoft Azure 虛擬網路中的資源，以及資源之間的關聯性。 例如，虛擬網路包含子網路。 子網路包含資源，例如 Azure 虛擬機器 (VM)。 VM 可以有一或多個網路介面。 每個子網路都可以有安全性群組群組及其相關聯的路由表。 Azure 網路監看員的拓撲功能可讓您檢視虛擬網路中的資源、與虛擬網路中的資源相關聯的資源，以及資源之間的關聯性。

您可以使用 [Azure 入口網站](#azure-portal)、[Azure CLI](#azure-cli) 或 [PowerShell](#powershell) 來檢視拓撲。

## <a name = "azure-portal"></a>檢視拓撲 - Azure 入口網站

1. 使用具有必要[權限](required-rbac-permissions.md)的帳戶登入 [Azure 入口網站](https://portal.azure.com)。
2. 在入口網站的左上角，選取 [所有服務]。
3. 在 [所有服務] 篩選方塊中，輸入 [網路監看員]。 當結果中出現**網路監看員**時，請加以選取。
4. 選取 [拓撲]。 若要產生拓撲，在您要產生拓撲的虛擬網路所在的相同區域中必須要有網路監看員。 如果您未在要產生拓撲的虛擬網路所在的相同區域中啟用網路監看員，系統將在所有區域中自動為您建立網路監看員。 網路監看員會建立在名為 NetworkWatcherRG 的資源群組中。
5. 依序選取訂用帳戶、您想要檢視拓撲之虛擬網路的資源群組，以及虛擬網路。 下圖會在名為 *MyResourceGroup* 的資源群組中，針對名為 *MyVnet* 的虛擬網路顯示一個拓撲：

    ![檢視拓撲](./media/view-network-topology/view-topology.png)

    您可以在上圖中看到，虛擬網路包含三個子網路。 一個子網路中部署了 VM。 VM 有一個連結的網路介面，和一個相關聯的公用 IP 位址。 其他兩個子網路各有一個相關聯的路由表。 每個路由表分別包含兩個路由。 一個子網路具有相關聯的網路安全性群組。 系統只會顯示以下資源的拓撲資訊：
    
    - 位於與 *myVent* 虛擬網路相同的資源群組和區域中。 例如，存在於 *MyResourceGroup* 以外的資源群組中的網路安全性群組並不會顯示，即使該網路安全性群組與 *MyVnet* 虛擬網路中的子網路相關聯，仍是如此。
    - 位於 *myVnet* 虛擬網路內，或與該網路內的資源相關聯。 例如，未與 *MyVnet* 虛擬網路中的子網路或網路介面相關聯的網路安全性群組並不會顯示，即使該網路安全性群組位於 *MyResourceGroup* 資源群組中，仍是如此。

  圖中顯示的拓撲屬於部署**透過網路虛擬設備路由流量的指令碼範例**後所建立的虛擬網路；您可以使用 [Azure CLI](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) 或 [PowerShell](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) 部署該指令碼範例。

6. 選取 [下載拓撲]，以下載 svg 格式的影像作為可編輯的檔案。

途中顯示的資源是虛擬網路中的網路元件子集。 例如，當網路安全性群組顯示時，其中的安全性規則並不會顯示於圖表中。 雖然在此圖中並未區分，但直線代表兩個關聯性的其中之一：*內含項目*或*相關聯*。 若要查看虛擬網路中的完整資源清單以及資源之間的關聯性類型，請使用 [PowerShell](#powershell) 或 [Azure CLI](#azure-cli) 產生拓撲。

## <a name = "azure-cli"></a>檢視拓撲 - Azure CLI

您可以在下列步驟中執行命令：
- 在 Azure Cloud Shell 中，選取位於任何命令右上方的**試試看**。 Azure Cloud Shell 是一個免費的互動式殼層，具有預先安裝和設定的通用 Azure 工具，可與您的帳戶搭配使用。
- 從您的電腦執行 CLI。 如果您從電腦執行 CLI，則必須要使用 Azure CLI 2.0.31 版或更新版本，才能執行本文中的步驟。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

您所使用的帳戶必須具有必要的[權限](required-rbac-permissions.md)。

1. 如果您在要建立拓撲的虛擬網路所在的相同區域中已有網路監看員，請跳至步驟 3。 使用 [az group create](/cli/azure/group#az_group_create) 建立包含網路監看員的資源群組： 下列範例會在 *eastus* 區域中建立資源群組：

    ```azurecli-interactive
    az group create --name NetworkWatcherRG --location eastus
    ```

2. 使用 [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) 建立網路監看員。 下列範例會在 *eastus* 區域中建立網路監看員：

    ```azurecli-interactive
    az network watcher configure \
      --resource-group NetworkWatcherRG \
      --location eastus \
      --enabled true
    ```

3. 使用 [az network watcher show-topology](/cli/azure/network/watcher#az-network-watcher-show-topology) 檢視拓撲。 下列範例會檢視資源群組 *MyResourceGroup* 的拓撲：

    ```azurecli-interactive
    az network watcher show-topology --resource-group MyResourceGroup
    ```

    只有與 *MyResourceGroup* 資源群組位於相同資源群組內的資源，以及與網路監看員位於相同區域的資源，才會傳回拓撲資訊。 例如，存在於 *MyResourceGroup* 以外的資源群組中的網路安全性群組並不會顯示，即使該網路安全性群組與 *MyVnet* 虛擬網路中的子網路相關聯，仍是如此。

  深入了解傳回的輸出中的[關聯性](#relationhips)和[屬性](#properties)。 如果您沒有可檢視拓撲的現有虛擬網路，您可以使用[透過網路虛擬設備來路由傳送流量](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)指令碼範例建立一個。 若要檢視拓撲的圖表，並以可編輯檔案的形式加以下載，請使用[入口網站](#azure-portal)。

## <a name = "powershell"></a>檢視拓撲 - PowerShell

您可以在下列步驟中執行命令：
- 在 Azure Cloud Shell 中，選取位於任何命令右上方的**試試看**。 Azure Cloud Shell 是一個免費的互動式殼層，具有預先安裝和設定的通用 Azure 工具，可與您的帳戶搭配使用。
- 從您的電腦執行 PowerShell。 如果您從電腦執行 PowerShell，則必須要使用 AzureRm 模組的 5.7.0 版或更新版本，才能執行本文中的步驟。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

您所使用的帳戶必須具有必要的[權限](required-rbac-permissions.md)。

1. 如果您在要建立拓撲的虛擬網路所在的相同區域中已有網路監看員，請跳至步驟 3。 使用 [New-AzureRmResourceGroup](/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup) 建立包含網路監看員的資源群組。 下列範例會在 *eastus* 區域中建立資源群組：

    ```azurepowershell-interactive
    New-AzureRmResourceGroup -Name NetworkWatcherRG -Location EastUS
    ```

2. 使用 [New-AzureRmNetworkWatcher](/powershell/module/azurerm.network/new-azurermnetworkwatcher) 建立網路監看員。 下列範例會在 eastus 區域中建立網路監看員：

    ```azurepowershell-interactive
    New-AzureRmNetworkWatcher `
      -Name NetworkWatcher_eastus `
      -ResourceGroupName NetworkWatcherRG
    ```

3. 使用 [Get-AzureRmNetworkWatcher](/powershell/module/azurerm.network/get-azurermnetworkwatcher) 擷取網路監看員執行個體。 下列範例會擷取美國東部區域中的網路監看員：

    ```azurepowershell-interactive
    $nw = Get-AzurermResource `
      | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "EastUS" }
    $networkWatcher = Get-AzureRmNetworkWatcher `
      -Name $nw.Name `
      -ResourceGroupName $nw.ResourceGroupName
    ```

4. 使用 [Get-AzureRmNetworkWatcherTopology](/powershell/module/azurerm.network/get-azurermnetworkwatchertopology) 擷取拓撲。 下列範例會擷取資源群組 *MyResourceGroup* 中的虛擬網路的拓撲：

    ```azurepowershell-interactive
    Get-AzureRmNetworkWatcherTopology `
      -NetworkWatcher $networkWatcher `
      -TargetResourceGroupName MyResourceGroup
    ```

   只有與 *MyResourceGroup* 資源群組位於相同資源群組內的資源，以及與網路監看員位於相同區域的資源，才會傳回拓撲資訊。 例如，存在於 *MyResourceGroup* 以外的資源群組中的網路安全性群組並不會顯示，即使該網路安全性群組與 *MyVnet* 虛擬網路中的子網路相關聯，仍是如此。

  深入了解傳回的輸出中的[關聯性](#relationhips)和[屬性](#properties)。 如果您沒有可檢視拓撲的現有虛擬網路，您可以使用[透過網路虛擬設備來路由傳送流量](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)指令碼範例建立一個。 若要檢視拓撲的圖表，並以可編輯檔案的形式加以下載，請使用[入口網站](#azure-portal)。

## <a name="relationships"></a>關聯性

在拓撲中傳回的所有資源，都會具有對另一項資源的下列其中一種關聯性：

| 關聯性類型 | 範例                                                                                                |
| ---               | ---                                                                                                    |
| 內含項目       | 虛擬網路包含子網路。 子網路包含網路介面。                            |
| 相關聯        | 網路介面與 VM 相關聯。 與網路介面相關聯的公用 IP 位址。 |

## <a name="properties"></a>properties

在拓撲中傳回的所有資源都具有下列屬性：

- **名稱**：資源的名稱
- **識別碼**：資源的 URI。
- **位置**：資源所在的 Azure 區域。
- **關聯性**：與所參考物件之關聯性的清單。 每個關聯性都具有下列屬性：
    - **AssociationType**：參考子物件與父系之間的關聯性。 有效值為 [包含] 或 [相關聯]。
    - **名稱**：所參考資源的名稱。
    - **ResourceId**：關聯性中參考之資源的 URI。

## <a name="next-steps"></a>後續步驟

- 了解如何使用網路監看員的 IP 流量驗證功能[診斷 VM 輸入或輸出的網路流量篩選問題](diagnose-vm-network-traffic-filtering-problem.md)
- 了解如何使用網路監看員的下一個躍點功能[診斷 VM 輸出的網路流量路由問題](diagnose-vm-network-routing-problem.md)
