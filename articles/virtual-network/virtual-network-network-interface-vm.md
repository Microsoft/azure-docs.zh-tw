---
title: 在 Azure 虛擬機器中新增或移除網路介面 | Microsoft Docs
description: 了解如何在虛擬機器中新增網路介面或移除網路介面。
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
ms.date: 12/15/2017
ms.author: jdial
ms.openlocfilehash: 7a981297c3b0d958761f0c802bbe9a75dc0220b4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987758"
---
# <a name="add-network-interfaces-to-or-remove-network-interfaces-from-virtual-machines"></a>在虛擬機器中新增網路介面或移除網路介面

了解如何在建立 Azure 虛擬機器 (VM) 時新增現有的網路介面，或從處於已停止 (已取消配置) 狀態的現有 VM 新增或移除網路介面。 網路介面可讓 Azure 虛擬機器與網際網路、Azure 以及內部部署資源進行通訊。 一個 VM 可以有一或多個網路介面。 

如果您需要新增、變更或移除網路介面的 IP 位址，請參閱[管理網路介面 IP 位址](virtual-network-network-interface-addresses.md)。 如果您需要建立、變更或刪除網路介面，請參閱[管理網路介面](virtual-network-network-interface.md)。

## <a name="before-you-begin"></a>開始之前

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://portal.azure.com，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 Azure PowerShell 模組 5.2.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.26 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

## <a name="add-existing-network-interfaces-to-a-new-vm"></a>將現有的網路介面新增至新的 VM

當您透過入口網站建立虛擬機器時，入口網站會以預設設定為您建立網路介面，並將其連結至該 VM。 使用 Azure 入口網站時，無法將現有的網路介面新增至新的 VM，也無法建立具有多個網路介面的 VM。 這兩項作業都可使用 CLI 或 PowerShell 來進行，但請務必熟悉[條件約束](#constraints)。 如果您建立具有多個網路介面的 VM，則也必須設定作業系統，才能在建立 VM 之後正常使用它們。 了解如何設定 [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) 或 [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) 以供使用多個網路介面。

### <a name="commands"></a>命令

建立 VM 之前，請使用[建立網路介面](virtual-network-network-interface.md#create-a-network-interface)中的步驟來建立網路介面。

|工具|命令|
|---|---|
|CLI|[az vm create](/cli/azure/vm?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_create)|
|PowerShell|[New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="vm-add-nic"></a>將網路介面新增至現有的 VM

1. 登入 Azure 入口網站。
2. 在入口網站頂端的搜尋方塊中，輸入您想要新增網路介面的 VM 名稱，或選取 [所有服務]，然後選取 [虛擬機器] 來瀏覽 VM。 找到 VM 之後，請加以選取。 VM 必須支援您想要新增的網路介面數目。 若要知道每個 VM 大小支援多少個網路介面，請參閱 [Azure 中的 Linux 虛擬機器大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或 [Azure 中的 Windows 虛擬機器大小](../virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。  
3. 在 [設定] 底下，選取 [概觀]。 選取 [停止]，然後等到 VM 的 [狀態] 變成 [已停止 (已取消配置)] 為止。 
4. 在 [設定] 底下，選取 [網路]。
5. 選取 [連結網路介面]。 從目前未連結至其他 VM 的網路介面清單中，選取您想要連結的網路介面。 

    >[!NOTE]
    您選取的網路介面不得啟用加速網路、不得指派為 IPv6 位址，而且必須和目前已與 VM 連結的網路介面存在於相同的虛擬網路中。 

    如果您沒有現有的網路介面，則必須先建立一個。 若要這樣做，請選取 [建立網路介面]。 若要深入了解如何建立網路介面的詳細資訊，請參閱[建立網路介面](virtual-network-network-interface.md#create-a-network-interface)。 若要深入了解將網路介面新增至虛擬機器時的其他條件約束，請參閱[條件約束](#constraints)。

6. 選取 [確定] 。
7. 在 [設定] 底下，選取 [概觀]，然後選取 [啟動] 以啟動虛擬機器。
8. 設定 VM 作業系統以正確使用多個網路介面。 了解如何設定 [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) 或 [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) 以供使用多個網路介面。

|工具|命令|
|---|---|
|CLI|[az vm nic add](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_nic_add) (參考) 或[詳細步驟](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-a-vm)|
|PowerShell|[Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (參考) 或[詳細步驟](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-an-existing-vm)|

## <a name="view-network-interfaces-for-a-vm"></a>檢視 VM 網路介面

您可以檢視目前連接至 VM 的網路介面，以了解每個網路介面的組態，以及指派給每個網路介面的 IP 位址。 

1. 使用具備您訂用帳戶擁有者、參與者或網路參與者角色的帳戶來登入 [Azure 入口網站](https://portal.azure.com)。 若要深入了解如何將角色指派給帳戶的詳細資訊，請參閱 [Azure 角色型存取控制的內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)。
2. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「虛擬機器」。 當搜尋結果中出現**虛擬機器**時加以選取。
3. 選取您想要檢視網路介面的 VM 名稱。
4. 在您所選取 VM 的 [設定] 區段中，選取 [網路]。 若要了解網路介面設定以及如何變更它們，請參閱[管理網路介面](virtual-network-network-interface.md)。 若要了解如何新增、變更或移除指派給網路介面的 IP 位址，請參閱[管理網路介面 IP 位址](virtual-network-network-interface-addresses.md)。

### <a name="commands"></a>命令

|工具|命令|
|---|---|
|CLI|[az vm show](/cli/azure/vm?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_show)|
|PowerShell|[Get-AzureRmVM](/powershell/module/azurerm.compute/get-azurermvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="remove-a-network-interface-from-a-vm"></a>從 VM 移除網路介面

1. 登入 Azure 入口網站。
2. 在入口網站頂端的搜尋方塊中，搜尋您想要從中移除 (中斷連結) 網路介面的 VM 名稱，或選取 [所有服務]，然後選取 [虛擬機器] 來瀏覽 VM。 找到 VM 之後，請加以選取。
3. 在 [設定] 底下，選取 [概觀]，然後選取 [停止]。 等到 VM 的 [狀態] 變成 [已停止 (已取消配置)] 為止。 
4. 在 [設定] 底下，選取 [網路]。
5. 選取 [將網路介面中斷連結]。 從目前連結至虛擬機器的網路介面清單中，選取您想要與之中斷連結的網路介面。 

    >[!NOTE]
    如果只列出一個網路介面，則無法加以中斷連結，因為虛擬機器一律至少必須有一個連結的網路介面。
6. 選取 [確定] 。

### <a name="commands"></a>命令

|工具|命令|
|---|---|
|CLI|[az vm nic remove](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_nic_remove) (參考) 或[詳細步驟](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-a-vm)|
|PowerShell|[Remove-AzureRMVMNetworkInterface](/powershell/module/azurerm.compute/remove-azurermvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (參考) 或[詳細步驟](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-an-existing-vm)|

## <a name="constraints"></a>條件約束

- VM 至少必須擁有一個連結的網路介面。
- VM 可連結的網路介面數量受限於 VM 大小所支援的數量。 若要深入了解每個 VM 大小支援多少個網路介面，請參閱 [Azure 中的 Linux 虛擬機器大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或 [Azure 中的 Windows 虛擬機器大小](../virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 所有大小都至少支援兩個網路介面。
- 您在某個 VM 中新增的網路介面目前無法連接至另一個 VM。 若要深入了解如何建立網路介面的詳細資訊，請參閱[建立網路介面](virtual-network-network-interface.md#create-a-network-interface)。
- 在過去，網路介面僅可新增至支援多個網路介面且至少以兩個網路介面建立的 VM。 您無法將網路介面新增至使用一個網路介面建立的 VM，即使該 VM 大小支援多個網路介面也一樣。 相反地，由於以至少兩個網路介面所建立的 VM 一定要有至少兩個網路介面，因此您只能從至少具有三個網路介面的 VM 中移除網路介面。 以上這些限制已不再適用。 您現在可以建立具有數量不拘 (以 VM 大小可支援的數目為限) 網路介面的 VM。
- 根據預設，連結至 VM 的第一個網路介面會定義為「主要」網路介面。 VM 中的所有其他網路介面均為「次要」網路介面。
- 雖然您可以控制要將輸出流量傳送至哪個網路介面，但根據預設，來自 VM 的所有輸出流量都會送出指派給主要網路介面之主要 IP 設定的 IP 位址。
- 在過去，相同可用性設定組中的所有 VM 都必須具有單一或多個網路介面。 具有任意多個 (最多可達 VM 大小所支援的數目) 網路介面的 VM 現在可存在於相同的可用性設定組中。 您只有在建立 VM 時，才能將 VM 新增到可用性設定組。 若要深入了解可用性設定組，請參閱[在 Azure 中管理 VM 的可用性](../virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)。
- 雖然相同 VM 中的網路介面可以連線至虛擬網路中不同的子網路，但這些網路介面全都必須連線至相同的虛擬網路。
- 您可以將任何主要或次要網路介面之任何 IP 組態的任何 IP 位址新增至 Azure Load Balancer 後端集區。 在過去，只能將主要網路介面的主要 IP 位址新增到後端集區。 若要深入了解 IP 位址和設定，請參閱[新增、變更或移除 IP 位址](virtual-network-network-interface-addresses.md)。
- 刪除 VM 並不會刪除與其連接的網路介面。 當您刪除 VM 時，就會中斷網路介面與該 VM 的連結。 您可以將網路介面新增至不同的 VM，也可以刪除網路介面。
- 如果為網路介面指派了私人 IPv6 位址，則您必須在建立 VM 時將該網路介面新增 (連結) 至 VM。 在您建立 VM 之後，就無法將指派了 IPv6 位址的網路介面新增至 VM。 如果您在建立虛擬機器時新增指派了私人 IPv6 位址的網路介面，則不論該虛擬機器大小支援多少個網路介面，您都只能將該網路介面新增到虛擬機器。 若要深入了解如何將 IP 位址指派給網路介面，請參閱[管理網路介面 IP 位址](virtual-network-network-interface-addresses.md)。
- 如同 IPv6，您無法在建立 VM 之後，將啟用加速網路的網路介面與 VM 連結。 此外，若要利用加速網路，您也必須在 VM 作業系統內完成一些步驟。 深入了解加速網路，以及在 [Windows](create-vm-accelerated-networking-powershell.md) 或 [Linux](create-vm-accelerated-networking-cli.md) 虛擬機器使用時的其他條件約束。

## <a name="next-steps"></a>後續步驟
若要建立具有多個網路介面或 IP 位址的 VM，請閱讀下列文章：

### <a name="commands"></a>命令

|Task|工具|
|---|---|
|建立具有多個 NIC 的 VM|[CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
|建立具有多個 IPv4 位址的單一 NIC VM|[CLI](virtual-network-multiple-ip-addresses-cli.md)、[PowerShell](virtual-network-multiple-ip-addresses-powershell.md)|
|建立具有私人 IPv6 位址的單一 NIC VM (在 Azure Load Balancer 後端)|[CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[Azure Resource Manager 範本](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|


