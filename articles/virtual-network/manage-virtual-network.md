---
title: 建立、變更或刪除 Azure 虛擬網路 | Microsoft Docs
description: 了解如何在 Azure 中建立、變更或刪除虛擬網路。
services: virtual-network
documentationcenter: na
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
ms.date: 02/09/2018
ms.author: jdial
ms.openlocfilehash: d72faa99c15fdbebb299e416fd902bae261f31f9
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221174"
---
# <a name="create-change-or-delete-a-virtual-network"></a>建立、變更或刪除虛擬網路

了解如何建立和刪除虛擬網路以及變更現有虛擬網路的設定，例如 DNS 伺服器和 IP 位址空間。 如果您不熟悉虛擬網路，則可以在[虛擬網路概觀](virtual-networks-overview.md)中或透過完成[教學課程](quick-create-portal.md)來深入了解。 虛擬網路包含子網路。 若要了解如何建立、變更和刪除子網路，請參閱[管理子網路](virtual-network-manage-subnet.md)。

## <a name="before-you-begin"></a>開始之前

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://portal.azure.com，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.31 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。
- 您登入或連線到 Azure 的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派[權限](#permissions)中所列適當動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="create-a-virtual-network"></a>建立虛擬網路

1. 選取 [+ 建立資源] > [網路] > [虛擬網路]。
2. 輸入或選取下列設定的值，然後選取 [建立]：
    - **名稱**：您選取用來建立虛擬網路的[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)名稱必須是唯一的。 虛擬網路建立後，就不能再變更其名稱。 您可以隨著時間建立多個虛擬網路。 如需命名建議，請參閱[命名慣例](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions)。 下列命名慣例有協助於輕鬆管理多個虛擬網路。
    - **位址空間**：虛擬網路的位址空間由一或多個以 CIDR 標記法指定的非重疊位址範圍所組成。 您可以定義公用或私人的位址範圍 (RFC 1918)。 不論您定義的是公用或私人的位址範圍，您都只能從虛擬網路、互連的虛擬網路，以及任何已連線到虛擬網路的內部部署網路來連線到位址空間。 您無法新增下列位址範圍︰
        - 224.0.0.0/4 (多點傳送)
        - 255.255.255.255/32 (廣播)
        - 127.0.0.0/8 (回送)
        - 169.254.0.0/16 (連結-本機)
        - 168.63.129.16/32 (內部 DNS)

      雖然在建立虛擬網路時，只能定義一個位址範圍，但您可以在建立虛擬網路後，新增更多位址範圍至位址空間。 若要了解如何將位址範圍新增至現有的虛擬網路，請參閱[新增或移除位址範圍](#add-or-remove-an-address-range)。

      >[!WARNING]
      >如果虛擬網路具有與另一個虛擬網路或內部部署網路重疊的位址範圍，就無法連線這兩個網路。 在您定義位址範圍之前，請考慮未來是否想要將該虛擬網路連線到其他虛擬網路或內部部署網路。
      >
      >

    - **子網路名稱**：子網路名稱在虛擬網路內必須是唯一的。 子網路建立後，就不能再變更子網路名稱。 當您在建立虛擬網路時，即使虛擬網路不一定要擁有任何子網路，但入口網站仍會要求您定義一個子網路。 當您在入口網站中建立虛擬網路時，您只能定義一個子網路。 建立虛擬網路之後，您可以將更多子網路新增至虛擬網路。 若要將子網路新增至虛擬網路，請參閱[管理子網路](virtual-network-manage-subnet.md)。 您可以使用 Azure CLI 或 PowerShell，建立具有多個子網路的虛擬網路。

      >[!TIP]
      >有時候，系統管理員會建立不同的子網路來篩選或控制在子網路之間傳送的流量。 在定義子網路之前，請先考慮您想要如何篩選和路由傳送子網路之間的流量。 若要深入了解如何篩選子網路之間的流量，請參閱[網路安全性群組](security-overview.md)。 Azure 會自動路由傳送子網路之間的流量，但您可以覆寫 Azure 預設路由。 若要深入了解 Azures 預設子網路流量路由，請參閱[路由概觀](virtual-networks-udr-overview.md)。
      >

    - **子網路位址範圍**︰此範圍必須位於您針對虛擬網路輸入的位址空間內。 您可以指定的最小範圍是 /29，此範圍可提供八個 IP 位址供子網路使用。 為了符合通訊協定的規定，Azure 會保留每個子網路中的第一個和最後一個位址。 Azure 還會保留三個位址供 Azure 服務使用。 因此，使用 /29 子網路位址範圍的虛擬網路只有三個可用的 IP 位址。 如果您打算將虛擬網路連線至 VPN 閘道，則必須建立一個閘道子網路。 深入了解[閘道子網路位址範圍的具體考量](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub)。 若符合特定條件，您可以在子網路建立好之後變更其位址範圍。 若要了解如何變更子網路的位址範圍，請參閱[管理子網路](virtual-network-manage-subnet.md)。
    - **訂用帳戶**︰選取[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)。 您無法在多個 Azure 訂用帳戶中使用相同的虛擬網路。 不過，您可以利用[虛擬網路對等互連](virtual-network-peering-overview.md)，將某個訂用帳戶中的虛擬網路連線至其他訂用帳戶中的虛擬網路。 您連線到虛擬網路的任何 Azure 資源必須與虛擬網路位於相同的訂用帳戶中。
    - **資源群組**：選取現有[資源群組](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-groups)或建立新的資源群組。 您連線到虛擬網路的 Azure 資源，可以位於與虛擬網路相同或不同的資源群組中。
    - **位置**︰選取 Azure [位置](https://azure.microsoft.com/regions/) (也稱為區域)。 虛擬網路只能位於一個 Azure 位置。 不過，您可以使用 VPN 閘道，將某個位置的虛擬網路連線至其他位置的虛擬網路。 您連線到虛擬網路的任何 Azure 資源必須與虛擬網路位於相同的位置。

**命令**

- Azure CLI：[az network vnet create](/cli/azure/network/vnet)
- PowerShell：[New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork)

## <a name="view-virtual-networks-and-settings"></a>檢視虛擬網路與設定

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 在虛擬網路清單中，選取您想要檢視設定的虛擬網路。
3. 您所選虛擬網路會列出如下設定︰
    - **概觀**︰提供虛擬網路的相關資訊，包括位址空間和 DNS 伺服器。 下列螢幕擷取畫面顯示名為 **MyVNet** 之虛擬網路的概觀設定：

        ![網路介面概觀](./media/manage-virtual-network/vnet-overview.png)

      您可以選取 [資源群組] 或 [訂用帳戶名稱] 旁的 [變更]，將虛擬網路移至不同的訂用帳戶或資源群組。 若要了解如何移動虛擬網路，請參閱[將資源移到不同的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 該文會列出必要條件，以及如何使用 Azure 入口網站、PowerShell 和 Azure CLI 來移動資源。 所有連線至虛擬網路的資源都必須隨著虛擬網路移動。
    - **位址空間**︰這裡會列出指派給虛擬網路的位址空間。 若要了解如何將位址範圍新增至位址空間和從中移除，請完成[新增或移除位址範圍](#add-or-remove-an-address-range)中的步驟。
    - **已連線的裝置**︰這裡會列出任何與虛擬網路連線的資源。 在上述螢幕擷取畫面中，有三個網路介面和一個負載平衡器連線到虛擬網路。 您建立並連線到虛擬網路的任何新資源都會在此列出。 如果您刪除已連線到虛擬網路的資源，清單中就不會再顯示該資源。
    - **子網路**︰虛擬網路中現存的子網路清單會在此顯示。 若要了解如何新增和移除子網路，請參閱[管理子網路](virtual-network-manage-subnet.md)。
    - **DNS 伺服器**︰您可以指定由 Azure 的內部 DNS 伺服器或自訂 DNS 伺服器來為連線到虛擬網路的裝置提供名稱解析。 在使用 Azure 入口網站建立虛擬網路時，虛擬網路預設會使用 Azure 的 DNS 伺服器來解析名稱。 若要修改 DNS 伺服器，請完成本文[變更 DNS 伺服器](#change-dns-servers)中的步驟。
    - **對等互連**︰如果訂用帳戶中已有對等互連，這裡便會列出。 您可以檢視現有對等互連的設定，也可以建立、變更或刪除對等互連。 若要深入了解對等互連，請參閱[虛擬網路對等互連](virtual-network-peering-overview.md)。
    - **屬性**︰顯示虛擬網路的相關設定，包括虛擬網路的資源識別碼及其所在的訂用帳戶。
    - **圖表**︰此圖表會以視覺方式來呈現所有連線到虛擬網路的裝置。 此圖表包含有關裝置的一些重要資訊。 若要在此檢視中管理裝置，請在圖表中選取裝置。
    - **一般 Azure 設定**：若要深入了解 Azure 的一般設定，請參閱下列資訊：
        *   [活動記錄檔](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)
        *   [存取控制 (IAM)](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#access-control)
        *   [標記](../azure-resource-manager/resource-group-using-tags.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
        *   [鎖定](../azure-resource-manager/resource-group-lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
        *   [自動化指令碼](../azure-resource-manager/resource-manager-export-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json#export-the-template-from-resource-group)

**命令**

- Azure CLI：[az network vnet show](/cli/azure/network/vnet#az_network_vnet_show)
- PowerShell：[Get-AzureRmVirtualNetwork](/powershell/module/azurerm.network/get-azurermvirtualnetwork)

## <a name="add-or-remove-an-address-range"></a>新增或移除位址範圍

您可以新增和移除虛擬網路的位址範圍。 位址範圍必須以 CIDR 標記法來指定，而且同一個虛擬網路中的位址範圍不能重疊。 您可以定義公用或私人的位址範圍 (RFC 1918)。 不論您定義的是公用或私人的位址範圍，您都只能從虛擬網路、互連的虛擬網路，以及任何已連線到虛擬網路的內部部署網路來連線到位址空間。 您無法新增下列位址範圍︰

- 224.0.0.0/4 (多點傳送)
- 255.255.255.255/32 (廣播)
- 127.0.0.0/8 (回送)
- 169.254.0.0/16 (連結-本機)
- 168.63.129.16/32 (內部 DNS)

新增或移除位址範圍：

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 從虛擬網路清單中，選取您要新增或移除位址範圍的虛擬網路。
3. 在 [設定] 底下，選取 [位址空間]。
4. 完成下列其中一個選項：
    - **新增位址範圍**︰輸入新的位址範圍。 此位址範圍不能與虛擬網路已定義的現有位址範圍重疊。
    - **移除位址範圍**：在您想要移除的位址範圍的右側，選取 [...]，然後選取 [移除]。 如果位址範圍有現存的子網路，您就無法移除該位址範圍。 若要移除位址範圍，您必須先刪除位址範圍中存在的任何子網路 (以及子網路中的任何資源)。
5. 選取 [ **儲存**]。

**命令**

- Azure CLI：[az network vnet update](/cli/azure/network/vnet#az_network_vnet_update)
- PowerShell：[Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork)

## <a name="change-dns-servers"></a>變更 DNS 伺服器

所有連線至虛擬網路的 VM 會向您為虛擬網路指定的 DNS 伺服器註冊。 他們也可使用指定的 DNS 伺服器進行名稱解析。 VM 中的每個網路介面 (NIC) 都可以有自己的 DNS 伺服器設定。 如果 NIC 有自己的 DNS 伺服器設定，其設定就會覆寫虛擬網路的 DNS 伺服器設定。 若要深入了解 NIC DNS 設定，請參閱[網路介面工作和設定](virtual-network-network-interface.md#change-dns-servers)。 若要深入了解 Azure 雲端服務中 VM 和角色執行個體的名稱解析，請參閱 [VM 和角色執行個體的名稱解析](virtual-networks-name-resolution-for-vms-and-role-instances.md)。 若要新增、變更或移除 DNS 伺服器：

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 從虛擬網路清單中，選取您要變更 DNS 伺服器的虛擬網路。
3.  在 [設定] 底下，選取 [DNS 伺服器]。
4. 選取下列其中一個選項：
    - **預設值 (由 Azure 提供)**︰所有資源名稱和私人 IP 位址都會自動向 Azure DNS 伺服器進行註冊。 您可以在連線到相同虛擬網路的任何資源之間解析名稱。 您無法使用此選項來跨虛擬網路解析名稱。 若要跨虛擬網路解析名稱，您必須使用自訂 DNS 伺服器。
    - **自訂**︰您可以為虛擬網路新增一或多部伺服器，數量可達 Azure 的限制。 若要深入了解 DNS 伺服器限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-networking-limits-classic)。 您有下列選擇：
        - **新增地址**︰將伺服器新增至虛擬網路 DNS 伺服器清單。 此選項也會向 Azure 註冊 DNS 伺服器。 如果您已向 Azure 註冊 DNS 伺服器，即可在清單中選取該 DNS 伺服器。
        - **移除位址**︰在您想要移除的伺服器旁，選取 [...]，然後選取 [移除]。 刪除伺服器只會從此虛擬網路清單中移除伺服器。 DNS 伺服器會在 Azure 中保持登錄狀態，以便您其他的虛擬網路使用。
        - **將 DNS 伺服器位址重新排序**︰請務必要確認，您為環境所列出的 DNS 伺服器順序是正確的。 DNS 伺服器清單會依其指定的順序來使用。 它們不會當作循環配置資源設定運作。 如果清單上的第一部 DNS 伺服器是可以連線的，用戶端就會使用該 DNS 伺服器，而不管該 DNS 伺服器是否有正常運作。 請移除所有列出的 DNS 伺服器，再以您想要的順序重新新增。
        - **變更位址**：將清單中的 DNS 伺服器醒目提示，然後輸入新位址。
5. 選取 [ **儲存**]。
6. 將連線到虛擬網路的 VM 重新啟動，讓這些 VM 獲得新的 DNS 伺服器設定。 這些 VM 會繼續使用其目前的 DNS 設定，直到您將其重新啟動。

**命令**

- Azure CLI：[az network vnet update](/cli/azure/network/vnet#az_network_vnet_update)
- PowerShell：[Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork)

## <a name="delete-a-virtual-network"></a>刪除虛擬網路

您只能刪除未連線任何資源的虛擬網路。 如果有資源連線至虛擬網路中的任何子網路，您必須先刪除虛擬網路中所有子網路的已連線資源。 不同資源的資源刪除步驟也各異。 若要了解如何刪除已連線至子網路的資源，請閱讀您想要刪除之各個資源類型的適用文件。 若要刪除虛擬網路：

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 從虛擬網路清單中，選取您要刪除的虛擬網路。
3. 請在 [設定] 底下，選取 [連接的裝置]，確認沒有任何裝置連線到虛擬網路。 如果有已連線的裝置，您必須先將其刪除，才能刪除虛擬網路。 如果沒有連接的裝置，請選取 [概觀]。
4. 選取 [刪除] 。
5. 若要確認刪除虛擬網路，請選取 [是]。

**命令**

- Azure CLI：[azure network vnet delete](/cli/azure/network/vnet#az_network_vnet_delete)
- PowerShell：[Remove-AzureRmVirtualNetwork](/powershell/module/azurerm.network/remove-azurermvirtualnetwork)

## <a name="permissions"></a>權限

若要針對虛擬網路執行工作，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表中所列適當動作的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                  |   名稱                                |
|---------------------------------------- |   --------------------------------    |
|Microsoft.Network/virtualNetworks/read   |   讀取虛擬網路              |
|Microsoft.Network/virtualNetworks/write  |   建立或更新虛擬網路  |
|Microsoft.Network/virtualNetworks/delete |   刪除虛擬網路            |

## <a name="next-steps"></a>後續步驟

- 使用 [PowerShell](powershell-samples.md) 或 [Azure CLI](cli-samples.md) 範例指令碼，或使用 Azure [Resource Manager 範本](template-samples.md)建立虛擬網路
- 為虛擬網路建立及套用 [Azure 原則](policy-samples.md)