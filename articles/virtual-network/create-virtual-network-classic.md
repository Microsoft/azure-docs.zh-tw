---
title: 建立具有多個子網路的 Azure 虛擬網路 (傳統) | Microsoft Docs
description: 了解如何在 Azure 中建立具有多個子網路的虛擬網路 (傳統)。
services: virtual-network
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/15/2018
ms.author: genli
ms.custom: ''
ms.openlocfilehash: bede78782154e014d4ffad949f9ebd1a44eb2c4c
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35767166"
---
# <a name="create-a-virtual-network-classic-with-multiple-subnets"></a>建立有多個子網路的虛擬網路 (傳統)

> [!IMPORTANT]
> Azure 有二種建立和處理資源的[不同部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)：Resource Manager 和傳統模型。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議透過 [Resource Manager](quick-create-portal.md) 部署模型建立最新的虛擬網路。

本教學課程可供了解如何建立具有不同公用和私人子網路的基本 Azure 虛擬網路 (傳統)。 您可以在子網路中建立 Azure 資源，像是虛擬網路、雲端服務。 在虛擬網路 (傳統) 中建立的資源可以互相通訊，也可以和連線到虛擬網路之其他網路中的資源通訊。

深入了解所有[虛擬網路](manage-virtual-network.md)和[子網路](virtual-network-manage-subnet.md)設定。

> [!WARNING]
> 當[訂用帳戶停用](../billing/billing-subscription-become-disable.md?toc=%2fazure%2fvirtual-network%2ftoc.json#you-reached-your-spending-limit)時，Azure 會立即刪除虛擬網路 (傳統)。 系統會刪除虛擬網路 (傳統),不論虛擬網路中是否有資源存在。 如果您稍後重新啟用訂用帳戶，就必須重新建立虛擬網路中存在的資源。

您可以使用 [Azure 入口網站](#portal)、[Azure 命令列介面 (CLI) 1.0](#azure-cli) 或 [PowerShell](#powershell) 來建立虛擬網路 (傳統)。

## <a name="portal"></a>入口網站

1. 在網際網路瀏覽器中，移至 [Azure 入口網站](https://portal.azure.com)。 使用您的 [Azure 帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#account)登入。 如果您沒有 Azure 帳戶，您可以註冊[免費試用](https://azure.microsoft.com/offers/ms-azr-0044p)。
2. 在 Azure 入口網站中按一下 [建立資源]。
3. 在出現的 [新增] 窗格頂端的 [搜尋市集] 方塊中，輸入 [虛擬網路]。 當搜尋結果中出現虛擬網路時，按一下 [虛擬網路]。
4. 在出現的 [虛擬網路] 窗格中，於 [選取部署模型] 方塊中選取 [傳統]，然後按一下 [建立]。 
5. 在 [建立虛擬網路 (傳統)] 窗格中輸入下列值，然後按一下 [建立]：

    |設定|值|
    |---|---|
    |名稱|myVnet|
    |位址空間|10.0.0.0/16|
    |子網路名稱|公開|
    |子網路位址範圍|10.0.0.0/24|
    |資源群組|讓 [新建] 保持選取狀態，然後輸入 **myResourceGroup**。|
    |訂用帳戶和位置|選取您的訂用帳戶和位置。

    如果您不熟悉 Azure，請深入了解[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)、[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)和[位置](https://azure.microsoft.com/regions) (也稱為「區域」)。
4. 當您在入口網站中建立虛擬網路時，您只能建立一個子網路。 在本教學課程中，您會在建立虛擬網路後建立第二個子網路。 稍後您可能會在**公用**子網路中建立可從網際網路存取的資源。 您也可能會在**私人**子網路中建立無法從網際網路存取的資源。 若要建立第二個子網路，在入口網站頂端的 [搜尋資源] 方塊中輸入 **myVnet**。 當 myVnet 出現在搜尋結果中時，按一下 [myVnet]。
5. 在出現的 [建立虛擬網路 (傳統)] 窗格中，按一下 [設定] 區段中的 [子網路]。
6. 在出現的 [myVnet - 子網路] 窗格中，按一下 [+ 新增]。
7. 在 [新增子網路] 窗格中的 [名稱] 輸入**私人**。 在 [位址範圍] 輸入 **10.0.1.0/24**。  按一下 [確定]。
8. 您就會在 [myVnet - 子網路] 窗格中看到您建立的 [公用] 和 [私人] 子網路。
9. **選用**：當您完成本教學課程後，可能會想刪除您所建立的資源，以免產生使用費：
    - 按一下 [myVnet] 窗格上的 [概觀]。
    - 按一下 [myVnet] 窗格上的 [刪除] 圖示。
    - 若要確認刪除，請在 [刪除虛擬網路] 方塊中，按一下 [是]。

## <a name="azure-cli"></a>Azure CLI

1. 您可以[安裝及設定 Azure CLI](../cli-install-nodejs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，或在 Azure 雲端命令列介面中使用 CLI。 Azure Cloud Shell 是免費的 Bash Shell，您可以直接在 Azure 入口網站內執行。 它具有預先安裝和設定的 Azure CLI，可與您的帳戶搭配使用。 若要取得 CLI 命令的說明，請輸入 `azure <command> --help`。 
2. 在 CLI 工作階段中，使用下的命令登入 Azure。 如果您按一下方塊下方的 [試試看]，系統會開啟雲端命令列介面。 您可以登入您的 Azure 訂用帳戶，而不需需入下列命令：

    ```azurecli-interactive
    azure login
    ```

3. 若要確保 CLI 為服務管理模式，則輸入下列命令：

    ```azurecli-interactive
    azure config mode asm
    ```

4. 建立具有私人子網路的虛擬網路：

    ```azurecli-interactive
    azure network vnet create --vnet myVnet --address-space 10.0.0.0 --cidr 16  --subnet-name Private --subnet-start-ip 10.0.0.0 --subnet-cidr 24 --location "East US"
    ```

5. 在虛擬網路中建立公用子網路：

    ```azurecli-interactive
    azure network vnet subnet create --name Public --vnet-name myVnet --address-prefix 10.0.1.0/24
    ```    
    
6. 檢閱虛擬網路和子網路：

    ```azurecli-interactive
    azure network vnet show --vnet myVnet
    ```

7. **選用**：當您完成本教學課程後，可能會想刪除您所建立的資源，以免產生使用費：

    ```azurecli-interactive
    azure network vnet delete --vnet myVnet --quiet
    ```

> [!NOTE]
> 雖然您無法使用 CLI 指定要建立虛擬網路 (傳統) 的資源群組，Azure 會在名為 Default-Networking 的資源群組中建立虛擬網路。

## <a name="powershell"></a>PowerShell

1. 安裝最新版的 PowerShell [Azure](https://www.powershellgallery.com/packages/Azure) 模組。 如果您不熟悉 Azure PowerShell，請參閱 [Azure PowerShell 概觀](/powershell/azure/overview?toc=%2fazure%2fvirtual-network%2ftoc.json)。
2. 啟動 PowerShell 工作階段。
3. 在 PowerShell 中，輸入 `Add-AzureAccount` 命令來登入 Azure。
4. 視需要變更下列的路徑和檔名，然後再匯出您現有的網路組態檔：

    ```powershell
    Get-AzureVNetConfig -ExportToFile c:\azure\NetworkConfig.xml
    ```

5. 若要建立具有公用和私人子網路的虛擬網路，請使用任何文字編輯器在網路組態檔中新增如下的**VirtualNetworkSite** 元素。

    ```xml
    <VirtualNetworkSite name="myVnet" Location="East US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Private">
            <AddressPrefix>10.0.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Public">
            <AddressPrefix>10.0.1.0/24</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    ```

    檢閱完整的[網路組態檔結構描述](https://msdn.microsoft.com/library/azure/jj157100.aspx)。

6. 匯入網路組態檔：

    ```powershell
    Set-AzureVNetConfig -ConfigurationPath c:\azure\NetworkConfig.xml
    ```

    > [!WARNING]
    > 匯入變更過的網路組態檔會導致您訂用帳戶中現有的虛擬網路 (傳統) 發生變更。 請確定您只新增先前的虛擬網路，並且未變更或移除您訂用帳戶中任何現有的虛擬網路。 

7. 檢閱虛擬網路和子網路：

    ```powershell
    Get-AzureVNetSite -VNetName "myVnet"
    ```

8. **選用**：當您完成本教學課程後，可能會想刪除您所建立的資源，以免產生使用費。 若要刪除虛擬網路，再次執行步驟 4-6，這次移除您在步驟 5 中加入的 **VirtualNetworkSite** 元素。
 
> [!NOTE]
> 雖然您無法使用 PowerShell 指定要建立虛擬網路 (傳統) 的資源群組，Azure 會在名為 Default-Networking 的資源群組中建立虛擬網路。

---

## <a name="next-steps"></a>後續步驟

- 若要了解所有虛擬網路和子網路的設定，請參閱[管理虛擬網路](manage-virtual-network.md)和[管理虛擬網路子網路](virtual-network-manage-subnet.md)。 您有各種選項可在生產環境中使用虛擬網路和子網路，以便符合不同的需求。
- 建立 [Windows](../virtual-machines/windows/classic/createportal-classic.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/classic/createportal-classic.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虛擬機器，然後將它連線至現有的虛擬網路。
- 若要將同一個 Azure 位置中的兩個虛擬網路連線，請在兩個虛擬網路之間建立[虛擬網路對等互連](create-peering-different-deployment-models.md)。 您可以對等互連虛擬網路 (Resource Manager) 與虛擬網路 (傳統)，但無法對等互連兩個虛擬網路 (傳統)。
- 使用 [VPN 閘道](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或 [Azure ExpressRoute](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 線路將虛擬網路連線至內部部署網路。
