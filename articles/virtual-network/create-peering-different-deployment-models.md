---
title: 建立 Azure 虛擬網路對等互連 - 不同部署模型 - 相同訂用帳戶 |Microsoft Docs
description: 了解如何在透過不同 Azure 部署模型建立且存在於相同 Azure 訂用帳戶中的虛擬網路之間，建立虛擬網路對等互連。
services: virtual-network
documentationcenter: ''
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: jdial;anavin
ms.openlocfilehash: 44cc582bfa0a6940de7eeea9b54e3979735c07e2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998219"
---
# <a name="create-a-virtual-network-peering---different-deployment-models-same-subscription"></a>建立虛擬網路對等互連 - 不同部署模型、相同訂用帳戶

在本教學課程中，您會了解如何在透過不同部署模型建立的虛擬網路之間，建立虛擬網路對等互連。 這兩個虛擬網路均存在於相同的訂用帳戶中。 對等互連兩個虛擬網路，可讓不同虛擬網路中的資源彼此通訊，且通訊時會有相同的頻寬和延遲，彷彿這些資源是位於相同的虛擬網路中。 深入了解[虛擬網路對等互連](virtual-network-peering-overview.md)。

建立虛擬網路對等互連的步驟會因一些因素而有所不同，這取決於虛擬網路是位於相同還是不同的訂用帳戶中，以及是透過哪一個 [Azure 部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)建立虛擬網路。 請按一下下表中的案例，以了解如何在其他案例中建立虛擬網路對等互連：

|Azure 部署模型  | Azure 訂用帳戶  |
|--------- |---------|
|[兩者皆使用 Resource Manager](tutorial-connect-virtual-networks-portal.md) |相同|
|[兩者皆使用 Resource Manager](create-peering-different-subscriptions.md) |不同|
|[一個使用 Resource Manager、一個使用傳統部署模型](create-peering-different-deployment-models-subscriptions.md) |不同|

虛擬網路對等互連無法在透過傳統部署模型建立的兩個虛擬網路之間建立。 如果您需要將兩個都是透過傳統部署模型建立的虛擬網路連接，可以使用 Azure [VPN 閘道](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)來連接這些虛擬網路。

此教學課程將同一個區域中的虛擬網路視為對等。 您也可以針對不同[支援區域](virtual-network-manage-peering.md#cross-region)中的虛擬網路進行對等互連。 建議您在對等互連虛擬網路之前，先熟悉[對等互連的需求和限制條件](virtual-network-manage-peering.md#requirements-and-constraints)。

您可以使用 [Azure 入口網站](#portal)、Azure [命令列介面](#cli) (CLI)、Azure [PowerShell](#powershell) 或 [Azure Resource Manager 範本](#template)，來建立虛擬網路對等互連。 按一下任何先前的工具連結，直接前往使用您所選工具建立虛擬網路對等互連的步驟。

## <a name="create-peering---azure-portal"></a>建立對等互連 - Azure 入口網站

1. 登入 [Azure 入口網站](https://portal.azure.com)。 您登入時使用的帳戶必須擁有必要的權限，才能建立虛擬網路對等互連。 如需權限清單，請參閱[虛擬網路對等互連權限](virtual-network-manage-peering.md#requirements-and-constraints)。
2. 依序按一下 [新增]、[網路] 及 [虛擬網路]。
3. 在 [建立虛擬網路] 刀鋒視窗上，輸入或選取下列設定的值，然後按一下 [建立]：
    - **名稱**：*myVnet1*
    - **位址空間**：*10.0.0.0/16*
    - **子網路名稱**：*預設值*
    - **子網路位址範圍**：*10.0.0.0/24*
    - **訂用帳戶**︰選取您的訂用帳戶
    - **資源群組**：選取 [新建] 並輸入 *myResourceGroup*
    - **位置**：*美國東部*
4. 按一下 [+ 新增]。 在 [搜尋 Marketplace] 方塊中，輸入*虛擬網路*。 當搜尋結果中出現虛擬網路時，按一下 [虛擬網路]。 
5. 在 [虛擬網路] 刀鋒視窗中，於 [選取部署模型] 方塊中選取 [傳統]，然後按一下 [建立]。
6. 在 [建立虛擬網路] 刀鋒視窗上，輸入或選取下列設定的值，然後按一下 [建立]：
    - **名稱**： *myVnet2*
    - **位址空間**：*10.1.0.0/16*
    - **子網路名稱**：*預設值*
    - **子網路位址範圍**：*10.1.0.0/24*
    - **訂用帳戶**︰選取您的訂用帳戶
    - **資源群組**：選取 [使用現有] 並選取 *myResourceGroup*
    - **位置**：*美國東部*
7. 在入口網站頂端的 [搜尋資源] 方塊中，輸入 *myResourceGroup*。 當搜尋結果中出現 **MyResourceGroup** 時，按一下該項目。 **myresourcegroup** 資源群組的刀鋒視窗隨即出現。 資源群組包含在前述步驟中建立的兩個虛擬網路。
8. 按一下 [myVNet1]。
9. 在顯示的 [myVnet1] 刀鋒視窗，從刀鋒視窗左側的垂直選項清單中按一下 [對等互連]。
10. 在顯示的 [myVnet1 - 對等互連] 刀鋒視窗中，按一下 [+ 新增]
11. 在顯示的 [新增對等互連] 刀鋒視窗中，輸入或選取下列選項，然後按一下 [確定]：
     - **名稱**：*myVnet1ToMyVnet2*
     - **虛擬網路部署模型**︰選取 [傳統]。 
     - **訂用帳戶**︰選取您的訂用帳戶
     - **虛擬網路**：按一下 [選擇虛擬網路]，然後按一下 [myVnet2]。
     - **允許虛擬網路存取**：確定已選取 [啟用]。
    本教學課程中不會使用其他設定。 若要了解所有對等互連設定，請閱讀[管理虛擬網路對等互連](virtual-network-manage-peering.md#create-a-peering)。
12. 按一下上一個步驟中的 [確定] 後，[新增對等互連] 刀鋒視窗隨即關閉，而且您會再次看到 [myVnet1 - 對等互連] 刀鋒視窗。 幾秒之後，您建立的對等互連會出現在刀鋒視窗中。 您所建立之 **myVnet1ToMyVnet2** 對等互連的 [對等互連狀態] 資料行中會列出 [已連線]。

    現在已建立對等互連。 您在任何一個虛擬網路中建立的任何 Azure 資源現在能夠透過其 IP 位址彼此通訊。 如果您使用虛擬網路的預設 Azure 名稱解析，則虛擬網路中的資源無法跨虛擬網路解析名稱。 如果您想要跨對等互連中的虛擬網路解析名稱，您必須建立自己的 DNS 伺服器。 了解如何設定[使用自己的 DNS 伺服器進行名稱解析](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)。
13. **選擇性**：雖然本教學課程未涵蓋建立虛擬機器，但您可以在每個虛擬網路中建立一部虛擬機器，並從一部虛擬機器連線至另一部來驗證連線。
14. **選擇性︰** 若要刪除您在本教學課程中建立的資源，請完成本文之[刪除資源](#delete-portal)一節中的步驟。

## <a name="cli"></a>建立對等互連 - Azure CLI

使用 Azure 傳統 CLI 與 Azure CLI 完成下列步驟。 您可以從 Azure Cloud Shell 完成這些步驟，只需在下列任一步驟中選取 [試用] 按鈕，或安裝[傳統 CLI](/cli/azure/install-cli-version-1.0.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 與 [CLI](/cli/azure/install-azure-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 並在您的本機電腦上執行命令。

1. 如果使用 Cloud Shell，請跳至步驟 2，因為 Cloud Shel l會自動將您登入 Azure。 開啟命令工作階段，然後使用 `azure login` 命令來登入 Azure。
2. 輸入 `azure config mode asm` 命令來以「服務管理」模式執行 CLI。
3. 輸入下列命令來建立虛擬網路 (傳統)：

   ```azurecli-interactive
   azure network vnet create --vnet myVnet2 --address-space 10.1.0.0 --cidr 16 --location "East US"
   ```

4. 使用 CLI 執行下列 bash CLI 指令碼，而不是傳統 CLI。 如需在 Windows 電腦上執行 bash CLI 指令碼的選項，請參閱[在 Windows 上安裝 Azure CLI](/cli/azure/install-azure-cli-windows)。

   ```azurecli-interactive
   #!/bin/bash

   # Create a resource group.
   az group create \
     --name myResourceGroup \
     --location eastus

   # Create the virtual network (Resource Manager).
   az network vnet create \
     --name myVnet1 \
     --resource-group myResourceGroup \
     --location eastus \
     --address-prefix 10.0.0.0/16
   ```

5. 在透過不同部署模型建立的兩個虛擬網路之間，使用 CLI 建立虛擬網路對等互連。 將下列指令碼複製到您電腦上的文字編輯器中。 使用您的訂用帳戶 ID 來取代 `<subscription id>` 。如果您不知道您的訂用帳戶 ID，請輸入 `az account show` 命令。 輸出中的 **id** 值就是您的訂用帳戶 ID。請將修改過的指令碼貼到您的 CLI 工作階段中，然後按 `Enter`。

   ```azurecli-interactive
   # Get the id for VNet1.
   vnet1Id=$(az network vnet show \
     --resource-group myResourceGroup \
     --name myVnet1 \
     --query id --out tsv)

   # Peer VNet1 to VNet2.
   az network vnet peering create \
     --name myVnet1ToMyVnet2 \
     --resource-group myResourceGroup \
     --vnet-name myVnet1 \
     --remote-vnet-id /subscriptions/<subscription id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnet2 \
     --allow-vnet-access
   ```

6. 在指令碼執行之後，檢閱虛擬網路 (Resource Manager) 的對等互連。 請複製下列命令並貼到您的 CLI 工作階段中，然後按 `Enter`：

   ```azurecli-interactive
   az network vnet peering list \
     --resource-group myResourceGroup \
     --vnet-name myVnet1 \
     --output table
   ```

   輸出會在 **PeeringState** 資料行中顯示 **Connected**。

   您在任何一個虛擬網路中建立的任何 Azure 資源現在能夠透過其 IP 位址彼此通訊。 如果您使用虛擬網路的預設 Azure 名稱解析，則虛擬網路中的資源無法跨虛擬網路解析名稱。 如果您想要跨對等互連中的虛擬網路解析名稱，您必須建立自己的 DNS 伺服器。 了解如何設定[使用自己的 DNS 伺服器進行名稱解析](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)。
7. **選擇性**：雖然本教學課程未涵蓋建立虛擬機器，但您可以在每個虛擬網路中建立一部虛擬機器，並從一部虛擬機器連線至另一部來驗證連線。
8. **選擇性︰** 若要刪除您在本教學課程中所建立的資源，請完成本文之[刪除資源](#delete-cli)中的步驟。

## <a name="powershell"></a>建立對等互連 - PowerShell

1. 安裝最新版的 PowerShell [Azure](https://www.powershellgallery.com/packages/Azure) 和 [AzureRm](https://www.powershellgallery.com/packages/AzureRM/) 模組。 如果您不熟悉 Azure PowerShell，請參閱 [Azure PowerShell 概觀](/powershell/azure/overview?toc=%2fazure%2fvirtual-network%2ftoc.json)。
2. 啟動 PowerShell 工作階段。
3. 在 PowerShell 中，輸入 `Add-AzureAccount` 命令來登入 Azure。 您登入時使用的帳戶必須擁有必要的權限，才能建立虛擬網路對等互連。 如需權限清單，請參閱[虛擬網路對等互連權限](virtual-network-manage-peering.md#requirements-and-constraints)。
4. 若要使用 PowerShell 來建立虛擬網路 (傳統)，您必須建立一個新的或修改現有的網路設定檔。 了解如何[匯出、更新及匯入網路設定檔](virtual-networks-using-network-configuration-file.md)。 就本教學課程中使用的虛擬網路而言，此檔案應該包含下列 **VirtualNetworkSite** 元素：

    ```xml
    <VirtualNetworkSite name="myVnet2" Location="East US">
      <AddressSpace>
        <AddressPrefix>10.1.0.0/16</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="default">
          <AddressPrefix>10.1.0.0/24</AddressPrefix>
        </Subnet>
      </Subnets>
    </VirtualNetworkSite>
    ```

    > [!WARNING]
    > 匯入變更過的網路設定檔會導致您訂用帳戶中現有的虛擬網路 (傳統) 發生變更。 請確定您只新增先前的虛擬網路，並且未變更或移除您訂用帳戶中任何現有的虛擬網路。 
5. 輸入 `Connect-AzureRmAccount` 命令來登入 Azure 以建立虛擬網路 (Resource Manager)。 您登入時使用的帳戶必須擁有必要的權限，才能建立虛擬網路對等互連。 如需權限清單，請參閱[虛擬網路對等互連權限](virtual-network-manage-peering.md#requirements-and-constraints)。
6. 建立資源群組和虛擬網路 (Resource Manager)。 請複製指令碼並貼到 PowerShell 中，然後按 `Enter`。

    ```powershell
    # Create a resource group.
      New-AzureRmResourceGroup -Name myResourceGroup -Location eastus

    # Create the virtual network (Resource Manager).
      $vnet1 = New-AzureRmVirtualNetwork `
      -ResourceGroupName myResourceGroup `
      -Name 'myVnet1' `
      -AddressPrefix '10.0.0.0/16' `
      -Location eastus
    ```

7. 在透過不同部署模型建立的兩個虛擬網路之間，建立虛擬網路對等互連。 將下列指令碼複製到您電腦上的文字編輯器中。 使用您的訂用帳戶 ID 來取代 `<subscription id>` 。如果您不知道您的訂用帳戶 ID，請輸入 `Get-AzureRmSubscription` 命令來檢視它。 傳回之輸出中的 **Id** 值就是您的訂用帳戶 ID。 若要執行指令碼，請從您的文字編輯器中複製修改過的指令碼，接著在 PowerShell 工作階段中按一下滑鼠右鍵，然後按 `Enter`。

    ```powershell
    # Peer VNet1 to VNet2.
    Add-AzureRmVirtualNetworkPeering `
      -Name myVnet1ToMyVnet2 `
      -VirtualNetwork $vnet1 `
      -RemoteVirtualNetworkId /subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnet2
    ```

8. 在指令碼執行之後，檢閱虛擬網路 (Resource Manager) 的對等互連。 請複製下列命令並貼到您的 PowerShell 工作階段中，然後按 `Enter`：

    ```powershell
    Get-AzureRmVirtualNetworkPeering `
      -ResourceGroupName myResourceGroup `
      -VirtualNetworkName myVnet1 `
      | Format-Table VirtualNetworkName, PeeringState
    ```

    輸出會在 **PeeringState** 資料行中顯示 **Connected**。

    您在任何一個虛擬網路中建立的任何 Azure 資源現在能夠透過其 IP 位址彼此通訊。 如果您使用虛擬網路的預設 Azure 名稱解析，則虛擬網路中的資源無法跨虛擬網路解析名稱。 如果您想要跨對等互連中的虛擬網路解析名稱，您必須建立自己的 DNS 伺服器。 了解如何設定[使用自己的 DNS 伺服器進行名稱解析](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)。

9. **選擇性**：雖然本教學課程未涵蓋建立虛擬機器，但您可以在每個虛擬網路中建立一部虛擬機器，並從一部虛擬機器連線至另一部來驗證連線。
10. **選擇性︰** 若要刪除您在本教學課程中所建立的資源，請完成本文之[刪除資源](#delete-powershell)中的步驟。
 
## <a name="delete"></a>刪除資源
當您完成本教學課程時，您可能會想刪除您在教學課程中建立的資源，以免產生使用費。 刪除資源群組同時會刪除其內含的所有資源。

### <a name="delete-portal"></a>Azure 入口網站

1. 在入口網站的搜尋方塊中，輸入 **myResourceGroup**。 在搜尋結果中按一下 [myResourceGroup]。
2. 在 [myResourceGroup] 刀鋒視窗中，按一下 [刪除] 圖示。
3. 若要確認刪除動作，請在 [輸入資源群組名稱] 方塊中輸入 **myResourceGroup**，然後按一下 [刪除]。

### <a name="delete-cli"></a>Azure CLI

1. 使用 Azure CLI 來以下列命令刪除虛擬網路 (Resource Manager)：

    ```azurecli-interactive
    az group delete --name myResourceGroup --yes
    ```

2. 使用傳統 CLI 來以下列命令刪除虛擬網路 (傳統)：

    ```azurecli-interactive
    azure config mode asm

    azure network vnet delete --vnet myVnet2 --quiet
    ```

### <a name="delete-powershell"></a>PowerShell

1. 輸入下列命令來刪除虛擬網路 (Resource Manager)：

    ```powershell
    Remove-AzureRmResourceGroup -Name myResourceGroup -Force
    ```

2. 若要使用 PowerShell 來刪除虛擬網路 (傳統)，您必須修改現有的網路設定檔。 了解如何[匯出、更新及匯入網路設定檔](virtual-networks-using-network-configuration-file.md)。 針對本教學課程中使用的虛擬網路，請移除下列 VirtualNetworkSite 元素：

    ```xml
    <VirtualNetworkSite name="myVnet2" Location="East US">
      <AddressSpace>
        <AddressPrefix>10.1.0.0/16</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="default">
          <AddressPrefix>10.1.0.0/24</AddressPrefix>
        </Subnet>
      </Subnets>
    </VirtualNetworkSite>
    ```

    > [!WARNING]
    > 匯入變更過的網路設定檔會導致您訂用帳戶中現有的虛擬網路 (傳統) 發生變更。 請確定您只移除先前的虛擬網路，並且未變更或移除您訂用帳戶中任何其他現有的虛擬網路。 

## <a name="next-steps"></a>後續步驟

- 請先徹底熟悉重要[虛擬網路對等互連的條件約束和行為](virtual-network-manage-peering.md#requirements-and-constraints)，再建立虛擬網路對等互連以供生產環境使用。
- 了解所有[虛擬網路對等互連設定](virtual-network-manage-peering.md#create-a-peering)。
- 了解如何透過虛擬網路對等互連來[建立中樞和輪輻網路拓撲](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#vnet-peering)。