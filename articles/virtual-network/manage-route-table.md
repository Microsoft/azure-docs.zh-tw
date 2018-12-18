---
title: 建立、變更或刪除 Azure 路由表 | Microsoft Docs
description: 了解如何建立、變更或刪除路由表。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/09/2018
ms.author: jdial
ms.openlocfilehash: 00a15d42084b660ee2ec4db2b012a5e47fc02b41
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988440"
---
# <a name="create-change-or-delete-a-route-table"></a>建立、變更或刪除路由表

Azure 會自動路由傳送 Azure 子網路、虛擬網路及內部部署網路之間的流量。 如果您想要變更任何 Azure 的預設路由，您可藉由建立路由表來執行此動作。 如果您不熟悉虛擬網路中的路由，您可以在[路由概觀](virtual-networks-udr-overview.md)中或透過完成[教學課程](tutorial-create-route-table-portal.md)來深入了解。

## <a name="before-you-begin"></a>開始之前

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://portal.azure.com，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.31 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

您登入或連線到 Azure 的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派[權限](#permissions)中所列適當動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="create-a-route-table"></a>建立路由表

每個 Azure 位置和訂用帳戶可以建立的路由表數目有所限制。 如需詳細資訊，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)。

1. 在入口網站的左上角，選取 [+ 建立資源]。
2. 選取 [網路]，然後選取 [路由表]。
3. 輸入路由表的 [名稱]，選取您的 [訂用帳戶]，建立新的 [資源群組] 或選取現有的資源群組，選取 [位置]，然後選取 [建立]。 如果您打算透過 VPN 閘道將路由表關聯至虛擬網路中連線到內部部署網路的子網路，而且停用 [BGP 路由傳播]，您的內部部署路由並不會傳播至子網路中的網路介面。

**命令**

- Azure CLI：[az network route-table create](/cli/azure/network/route-table/route#az_network_route_table_create)
- PowerShell：[New-AzureRmRouteTable](/powershell/module/azurerm.network/new-azurermroutetable)

## <a name="view-route-tables"></a>檢視路由表

在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。 列出的是存在訂用帳戶的路由表。

**命令**

- Azure CLI：[az network route-table list](/cli/azure/network/route-table/route#az_network_route_table_list)
- PowerShell：[Get-AzureRmRouteTable](/powershell/module/azurerm.network/get-azurermroutetable)

## <a name="view-details-of-a-route-table"></a>檢視路由表的詳細資料

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取清單中您想要檢視其詳細資料的路由表。 在 [設定] 下，您可以檢視路由表中的 [路由] 以及與路由表建立關聯的 [子網路]。
3. 若要深入了解 Azure 的一般設定，請參閱下列資訊：
    *   [活動記錄檔](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#activity-logs)
    *   [存取控制 (IAM)](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#access-control)
    *   [標記](../azure-resource-manager/resource-group-using-tags.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    *   [鎖定](../azure-resource-manager/resource-group-lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    *   [自動化指令碼](../azure-resource-manager/resource-manager-export-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json#export-the-template-from-resource-group)

**命令**

- Azure CLI：[az network route-table show](/cli/azure/network/route-table/route#az_network_route_table_show)
- PowerShell：[Get-AzureRmRouteTable](/powershell/module/azurerm.network/get-azurermroutetable)

## <a name="change-a-route-table"></a>變更路由表

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取您想要變更的路由表。 [新增](#create-a-route)或[移除](#delete-a-route)路由以及將子網路和路由表[建立關聯](#associate-a-route-table-to-a-subnet)或從子網路與路由表[中斷關聯](#dissociate-a-route-table-from-a-subnet)，都是最常見的變更。

**命令**

- Azure CLI：[az network route-table update](/cli/azure/network/route-table/route#az_network_route_table_update)
- PowerShell：[Set-AzureRmRouteTable](/powershell/module/azurerm.network/set-azurermroutetable)

## <a name="associate-a-route-table-to-a-subnet"></a>建立路由表與子網路的關聯

子網路可以有零個或一個與其相關聯的路由表。 路由表可以與零個或多個子網路建立關聯。 由於路由表並未與虛擬網路建立關聯，所以您必須和想要路由表與之相關聯的每個子網路建立關聯。 如果虛擬網路連線到 Azure 虛擬網路閘道 (ExpressRoute 或在利用 VPN 閘道使用 BGP 時為 VPN)，離開子網路的所有流量都會根據您在路由表內建立的路由、[預設路由](virtual-networks-udr-overview.md#default)以及從內部部署網路傳播的路由來路由傳送。 您只能在以路由表形式存在於相同的 Azure 位置和訂用帳戶的虛擬網路中，將路由表與子網路建立關聯。

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 在清單中所選取的虛擬網路須包含要和路由表建立關聯的子網路。
3. 在 [設定] 底下，選取 [子網路]。
4. 選取您要和路由表建立關聯的子網路。
5. 選取 [路由表]，選取您要和子網路建立關聯的路由表，然後選取 [儲存]。

如果虛擬網路連線至 Azure VPN 閘道，請勿將路由表關聯至所含路由的目的地為 0.0.0.0/0 的[閘道子網路](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub)。 這麼做會讓閘道無法正常運作。 如需關於在路由中使用 0.0.0.0/0 的詳細資訊，請參閱[虛擬網路流量路由](virtual-networks-udr-overview.md#default-route)。

**命令**

- Azure CLI：[az network vnet subnet update](/cli/azure/network/vnet/subnet?view=azure-cli-latest#az_network_vnet_subnet_update)
- PowerShell：[Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig)

## <a name="dissociate-a-route-table-from-a-subnet"></a>從子網路中斷與路由表的關聯

當您從子網路中斷與路由表的關聯時，Azure 會根據其[預設路由](virtual-networks-udr-overview.md#default)來路由傳送流量。

1. 在入口網站頂端的搜尋方塊中輸入「虛擬網路」。 當搜尋結果中出現**虛擬網路**時加以選取。
2. 選取的虛擬網路須包含要從中與路由表中斷關聯的子網路。
3. 在 [設定] 底下，選取 [子網路]。
4. 選取您要從中與路由表中斷關聯的子網路。
5. 選取 [路由表]，選取 [無]，然後選取 [儲存]。

**命令**

- Azure CLI：[az network vnet subnet update](/cli/azure/network/vnet/subnet?view=azure-cli-latest#az_network_vnet_subnet_update)
- PowerShell：[Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig) 

## <a name="delete-a-route-table"></a>刪除路由表

若有任何子網路與路由表相關聯，則無法刪除該路由表。 請從所有子網路與路由表[中斷關聯](#dissociate-a-route-table-from-a-subnet)之後，再嘗試予以刪除。

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取您想要刪除的路由表右側的 [...]。
3. 選取 [刪除]，然後選取 [是]。

**命令**

- Azure CLI：[az network route-table delete](/cli/azure/network/route-table/route#az_network_route_table_delete)
- PowerShell：[Remove-AzureRmRouteTable](https://docs.microsoft.com/powershell/module/azurerm.network/remove-azurermroutetable?view=azurermps-6.8.1) 

## <a name="create-a-route"></a>建立路由

每個 Azure 位置和訂用帳戶可在每個路由表建立的路由數目有所限制。 如需詳細資訊，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)。

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 從清單中選取您想要新增路由的路由表。
3. 在 [設定] 底下，選取 [路由]。
4. 選取 [+ 新增] 。
5. 為路由表內的路由輸入唯一的**名稱**。
6. 以 CIDR 標記法輸入您要路由傳送流量的**位址首碼**。 雖然首碼可以在另一個首碼內，但不可在路由表內的多個路由中重複。 例如，如果您將 10.0.0.0/16 定義為某路由的首碼，您仍能以 10.0.0.0/24 位址首碼定義另一個路由。 Azure 會根據最長首碼比對來為流量選取路由。 若要深入了解 Azure 選取路由的方式，請參閱[路由概觀](virtual-networks-udr-overview.md#how-azure-selects-a-route)。
7. 選取 [下一個躍點類型]。 如需所有下一個躍點類型的詳細說明，請參閱[路由概觀](virtual-networks-udr-overview.md)。
8. 輸入 [下一個躍點位址] 的 IP 位址。 如果您選取的 [下一個躍點類型] 為「虛擬設備」，您只能輸入一個位址。
9. 選取 [確定] 。

**命令**

- Azure CLI：[az network route-table route create](/cli/azure/network/route-table/route?view=azure-cli-latest#az_network_route_table_route_create)
- PowerShell：[New-AzureRmRouteConfig](/powershell/module/azurerm.network/new-azurermrouteconfig)

## <a name="view-routes"></a>檢視路由

路由表包含零個或多個路由。 若要深入了解檢視路由時列出的資訊，請參閱[路由概觀](virtual-networks-udr-overview.md)。

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 從清單中選取您想要檢視路由的路由表。
3. 在 [設定] 底下，選取 [路由]。

**命令**

- Azure CLI：[az network route-table route list](/cli/azure/network/route-table/route?view=azure-cli-latest#az_network_route_table_route_list)
- PowerShell：[Get-AzureRmRouteConfig](/powershell/module/azurerm.network/get-azurermrouteconfig)

## <a name="view-details-of-a-route"></a>檢視路由的詳細資料

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取您想要檢視其路由詳細資料的路由表。
3. 選取 [路由]。
4. 選取您想要檢視其詳細資料的路由。

**命令**

- Azure CLI：[az network route-table route show](/cli/azure/network/route-table/route?view=azure-cli-latest#az_network_route_table_route_show)
- PowerShell：[Get-AzureRmRouteConfig](/powershell/module/azurerm.network/get-azurermrouteconfig)

## <a name="change-a-route"></a>變更路由

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取您想要變更其路由的路由表。
3. 選取 [路由]。
4. 選取您想要變更的路由。
5. 將現有的設定變更為新設定，然後選取 [儲存]。

**命令**

- Azure CLI：[az network route-table route update](/cli/azure/network/route-table/route?view=azure-cli-latest#az_network_route_table_route_update)
- PowerShell：[Set-AzureRmRouteConfig](/powershell/module/azurerm.network/set-azurermrouteconfig)

## <a name="delete-a-route"></a>刪除路由

1. 在入口網站頂端的搜尋方塊中，輸入「路由表」。 當搜尋結果中出現**路由表**時加以選取。
2. 選取您想要刪除其路由的路由表。
3. 選取 [路由]。
4. 從路由清單中選取您想要刪除的路由右側的 [...]。
5. 選取 [刪除]，然後選取 [是]。

**命令**

- Azure CLI：[az network route-table route delete](/cli/azure/network/route-table/route?view=azure-cli-latest#az_network_route_table_route_delete)
- PowerShell：[Remove-AzureRmRouteConfig](/powershell/module/azurerm.network/remove-azurermrouteconfig)

## <a name="view-effective-routes"></a>檢視有效的路由

結合您建立的路由表、Azure 的預設路由以及經由 Azure 虛擬網路閘道透過 BGP 從內部部署網路傳播的任何路由，即形成連結至虛擬機器之每個網路介面的有效路由。 了解網路介面的有效路由，在對路由問題進行疑難排解時會很有幫助。 您可以檢視連結至執行中虛擬機器之任何網路介面的有效路由。

1. 在入口網站頂端的搜尋方塊中，輸入您要檢視其有效路由的虛擬機器名稱。 如果您不知道虛擬機器的名稱，請在搜尋方塊中輸入「虛擬機器」。 當搜尋結果中出現**虛擬機器**時加以選取，然後從清單中選取虛擬機器。
2. 在 [設定] 底下，選取 [網路]。
3. 選取網路介面的名稱。
4. 在 [支援 + 疑難排解] 底下，選取 [有效路由]。
5. 檢閱有效路由的清單，以判斷您要路由傳送流量的目標是否有正確的路由存在。 請參閱[路由概觀](virtual-networks-udr-overview.md)，深入了解您在此清單中看到的下一個躍點類型。

**命令**

- Azure CLI：[az network nic show-effective-route-table](/cli/azure/network/nic?view=azure-cli-latest#az_network_nic_show_effective_route_table)
- PowerShell：[Get-AzureRmEffectiveRouteTable](/powershell/module/azurerm.network/get-azurermeffectiveroutetable) 

## <a name="validate-routing-between-two-endpoints"></a>驗證兩個端點之間的路由

您可以判斷虛擬機器與另一個 Azure 資源的 IP 位址、內部部署資源或網路網路上資源之間的下一個躍點類型。 對路由問題進行疑難排解時，判斷 Azure 的路由會很有幫助。 您必須要有現有的網路監看員，才能完成此工作。 如果您沒有現有的網路監看員，請建立一個網路監看員，方法是完成[建立網路監看員執行個體](../network-watcher/network-watcher-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json)中的各步驟。

1. 在入口網站頂端的搜尋方塊中輸入「網路監看員」。 當搜尋結果中出現**網路監看員**時加以選取。
2. 在 [網路診斷工具] 底下，選取 [下一個躍點]。
3. 選取您要從其驗證路由的來源虛擬機器的 [訂用帳戶] 與 [資源群組]。
4. 選取 [虛擬機器]、連結至虛擬機器的 [網路介面]，以及指派給您要從其驗證路由之網路介面的 [來源 IP 位址]。
5. 輸入您要對其驗證路由的 [目的地 IP 位址]。
6. 選取 [下一個躍點]。
7. 短暫等候之後，傳回的資訊會告訴您路由傳送流量的下一個躍點類型和路由識別碼。 請參閱[路由概觀](virtual-networks-udr-overview.md)，深入了解您看到傳回的下一個躍點類型。

**命令**

- Azure CLI：[az network watcher show-next-hop](/cli/azure/network/watcher?view=azure-cli-latest#az_network_watcher_show_next_hop)
- PowerShell：[Get-AzureRmNetworkWatcherNextHop](/powershell/module/azurerm.network/get-azurermnetworkwatchernexthop) 

## <a name="permissions"></a>權限

若要針對路由表與路由執行工作，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表中所列適當動作的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                                          |   名稱                                                  |
|--------------------------------------------------------------   |   -------------------------------------------           |
| Microsoft.Network/routeTables/read                              |   讀取路由表                                    |
| Microsoft.Network/routeTables/write                             |   建立或更新路由表                        |
| Microsoft.Network/routeTables/delete                            |   刪除路由表                                  |
| Microsoft.Network/routeTables/join/action                       |   建立路由表與子網路的關聯                   |
| Microsoft.Network/routeTables/routes/read                       |   讀取路由                                          |
| Microsoft.Network/routeTables/routes/write                      |   建立或更新路由                              |
| Microsoft.Network/routeTables/routes/delete                     |   刪除路由                                        |
| Microsoft.Network/networkInterfaces/effectiveRouteTable/action  |   取得網路介面的有效路由表 |
| Microsoft.Network/networkWatchers/nextHop/action                |   從 VM 取得下一個躍點                           |

## <a name="next-steps"></a>後續步驟

- 使用 [PowerShell](powershell-samples.md) 或 [Azure CLI](cli-samples.md) 範例指令碼，或使用 Azure [Resource Manager 範本](template-samples.md)建立路由表
- 為虛擬網路建立及套用 [Azure 原則](policy-samples.md)
