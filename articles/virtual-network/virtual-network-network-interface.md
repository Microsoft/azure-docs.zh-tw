---
title: 建立、變更或刪除 Azure 網路介面 | Microsoft Docs
description: 了解何謂網路介面，以及要如何建立網路介面、變更其設定和刪除網路介面。
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
ms.date: 07/24/2017
ms.author: jdial
ms.openlocfilehash: 78578197c5f764c8e197d3426506cb1eb13b838f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956790"
---
# <a name="create-change-or-delete-a-network-interface"></a>建立、變更或刪除網路介面

了解如何建立網路介面、變更其設定和刪除網路介面。 網路介面可讓 Azure 虛擬機器與網際網路、Azure 以及內部部署資源進行通訊。 在使用 Azure 入口網站建立虛擬機器時，入口網站會以預設設定為您建立一個網路介面。 您可以改為選擇使用自訂設定建立網路介面，並在建立虛擬機器時新增一或多個網路介面。 您可能也想變更現有網路介面的預設網路介面設定。 本文說明如何以自訂設定建立網路介面、變更現有的設定 (例如，網路篩選 (網路安全性群組) 指派、子網路指派、DNS 伺服器設定和 IP 轉送)，以及刪除網路介面。

如果您需要新增、變更或移除網路介面的 IP 位址，請參閱[管理 IP 位址](virtual-network-network-interface-addresses.md)。 如果您需要新增或移除虛擬機器的網路介面，請參閱[新增或移除網路介面](virtual-network-network-interface-vm.md)。

## <a name="before-you-begin"></a>開始之前

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://portal.azure.com，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 Azure PowerShell 模組 5.4.1 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.28 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

您登入或連線到 Azure 的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派[權限](#permissions)中所列適當動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="create-a-network-interface"></a>建立網路介面

在使用 Azure 入口網站建立虛擬機器時，入口網站會以預設設定為您建立一個網路介面。 如果您想要指定所有網路介面設定，可以使用自訂設定建立網路介面，並在建立虛擬機器時將該網路介面連接到虛擬機器 (使用 PowerShell 或 Azure CLI)。 您也可以建立網路介面，並將它新增至現有的虛擬機器 (使用 PowerShell 或 Azure CLI)。 若要了解如何使用現有的網路介面建立虛擬機器，或如何新增或移除現有虛擬機器的網路介面，請參閱[新增或移除網路介面](virtual-network-network-interface-vm.md)。 在建立網路介面之前，用來建立網路介面的相同位置和訂用帳戶中必須已有既存的[虛擬網路](manage-virtual-network.md#create-a-virtual-network)。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 選取 [網路介面] 底下的 [+ 新增]。
3. 輸入或選取下列設定的值，然後選取 [建立]：

    |設定|必要？|詳細資料|
    |---|---|---|
    |名稱|是|名稱必須是您選取的資源群組中唯一的名稱。 在經過一段時間後，您的 Azure 訂用帳戶中可能會有好幾個網路介面。 如需建立命名慣例以便更輕鬆管理數個網路介面時的建議，請參閱[命名慣例](/azure/architecture/best-practices/naming-conventions?toc=%2fazure%2fvirtual-network%2ftoc.json#naming-rules-and-restrictions)。 建立網路介面之後，便無法變更名稱。|
    |虛擬網路|是|選取網路介面的虛擬網路。 您只能將網路介面指派給和網路介面的訂用帳戶和位置相同的虛擬網路。 網路介面建立後，即無法變更為它指派的虛擬網路。 新增網路介面的虛擬機器必須也位於和網路介面相同的位置和訂用帳戶。|
    |子網路|是|選取您所選虛擬網路內的子網路。 在網路介面建立後，您可以變更為它指派的子網路。|
    |私人 IP 位址指派|是| 在此設定中，您選擇的是 IPv4 位址指派方法。 選擇下列任一指派方法︰**動態︰** 若選取此選項，Azure 會自動指派您所選子網路位址空間中的下一個可用位址。 **靜態︰** 若選取此選項，您必須手動指派所選子網路位址空間中的可用 IP 位址。 直到您變更靜態和動態位址或刪除網路介面，位址才會變更。 您可以在建立網路介面後變更指派方法。 Azure DHCP 伺服器會將此位址指派給虛擬機器作業系統內的網路介面。|
    |網路安全性群組|否| 保持設為 [無]、選取現有的[網路安全性群組](security-overview.md)或[建立網路安全性群組](tutorial-filter-network-traffic.md)。 網路安全性群組可讓您篩選進出網路介面的網路流量。 您可以將網路安全性群組套用至網路介面或不套用。 您也可以將網路安全性群組套用至網路介面的目的地子網路或不套用。 當網路安全性群組套用至網路介面和網路介面的目的地子網路後，有時會發生非預期結果。 若要為套用至網路介面和子網路的網路安全性群組進行疑難排解，請參閱[針對網路安全性群組進行疑難排解](diagnose-network-traffic-filter-problem.md)。|
    |訂用帳戶|是|選取其中一個 Azure [訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)。 連接網路介面的虛擬機器和虛擬機器連線的虛擬網路，必須存在於相同的訂用帳戶。|
    |私人 IP 位址 (IPv6)|否| 如果您選取此核取方塊，除了指派給網路介面的 IPv4 位址以外，IPv6 位址也會指派給網路介面。 如需使用 IPv6 搭配網路介面的重要資訊，請參閱本文的 [IPv6](#IPv6) 一節。 您無法選取 IPv6 位址的指派方法。 如果您選擇要指派 IPv6 位址，會使用動態方法指派。
    |IPv6 名稱 (只有在核取 [私人 IP 位址 (IPv6)] 核取方塊時才會顯示) |是，如果核取 [私人 IP 位址 (IPv6)] 核取方塊。| 這個名稱會指派給網路介面的次要 IP 組態。 若要深入了解 IP 設定，請參閱[檢視網路介面設定](#view-network-interface-settings)。|
    |資源群組|是|選取現有的[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)，或建立一個群組。 網路介面可以位於與其連結的虛擬機器或虛擬機器所連線的虛擬網路相同或不同的資源群組中。|
    |位置|是|連接網路介面的虛擬機器和虛擬機器連線的虛擬網路，必須存在於相同的[地區](https://azure.microsoft.com/regions) (亦稱為區域)。|

當您建立網路介面時，入口網站並未提供將公用 IP 位址指派給網路介面的選項，然而在使用入口網站建立虛擬機器時，入口網站會建立公用 IP 位址，並將它指派給網路介面。 若要了解如何在建立網路介面之後對其新增公用 IP 位址，請參閱[管理 IP 位址](virtual-network-network-interface-addresses.md)。 如果您想要建立具有公用 IP 位址的網路介面，就必須使用 CLI 或 PowerShell 來建立網路介面。

入口網站不會提供建立網路介面時將網路介面指派給應用程式安全性群組的選項，但是 Azure CLI 和 PowerShell 會提供。 只要網路介面有連接至虛擬機器，您就可以使用入口網站，將現有的網路介面指派給應用程式安全性群組。 若要了解如何將網路介面指派給應用程式安全性群組，請參閱[新增到應用程式安全性群組或從中移除](#add-to-or-remove-from-application-security-groups)。

>[!Note]
> 在網路介面連接至虛擬機器和虛擬機器第一次啟動後，Azure 才會指派 MAC 位址給網路介面。 您無法指定 Azure 指派給網路介面的 MAC 位址。 在網路介面遭到刪除或指派給主要網路介面之主要 IP 組態的私人 IP 位址遭到變更之前，MAC 位址會保持指派給網路介面。 若要深入了解 IP 位址和 IP 設定，請參閱[管理 IP 位址](virtual-network-network-interface-addresses.md)。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic create](/cli/azure/network/nic#az_network_nic_create)|
|PowerShell|[New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface#create)|

## <a name="view-network-interface-settings"></a>檢視網路介面設定

您可以在網路介面建立後變更其大部分的設定。 入口網站不會顯示網路介面的 DNS 尾碼或應用程式安全性群組成員資格。 您可以使用 PowerShell 或 Azure CLI [命令](#view-settings-commands)來檢視 DNS 尾碼和應用程式安全性群組成員資格。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 從清單中選取您要檢視或變更設定的網路介面。
3. 隨即會針對您選取的網路介面列出下列項目：
    - **概觀：** 提供網路介面相關資訊︰例如，為它指派的 IP 位址、為網路介面指派的虛擬網路/子網路，以及網路介面連接的虛擬機器 (如有連接)。 下圖顯示名為 **mywebserver256** 之網路介面的概觀設定：![網路介面概觀](./media/virtual-network-network-interface/nic-overview.png) 您可以選取**資源群組**或**訂用帳戶名稱**旁邊的 (**變更**)，以將網路介面移至不同的資源群組或訂用帳戶。 如果您移動網路介面，則必須移動和網路介面相關的所有資源。 例如，如果網路介面連接至虛擬機器，您也必須移動虛擬機器和其他虛擬機器相關資源。 若要移動網路介面，請參閱[將資源移至新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json#use-portal)。 該文會列出必要條件，以及如何使用 Azure 入口網站、PowerShell 和 Azure CLI 來移動資源。
    - **IP 組態︰** 此處列出指派給 IP 組態的公用與私人 IPv4 和 IPv6 位址。 如果指派給 IP 組態的是 IPv6 位址，則不會顯示位址。 若要深入了解 IP 設定以及如何新增和移除 IP 位址，請參閱[設定 Azure 網路介面的 IP 位址](virtual-network-network-interface-addresses.md)。 IP 轉送和子網路指派也會在這一節設定。 若要深入了解這些設定，請參閱[啟用或停用 IP 轉送](#enable-or-disable-ip-forwarding)和[變更子網路指派](#change-subnet-assignment)。
    - **DNS 伺服器︰** 您可以指定由 Azure DHCP 伺服器指派給網路介面的 DNS 伺服器。 網路介面可以繼承為它指派之虛擬網路中的設定，或可擁有覆寫此設定的自訂設定。 若要修改所顯示的內容，請參閱[變更 DNS 伺服器](#change-dns-servers)。
    - **網路安全性群組 (NSG)：** 顯示與網路介面相關聯的 NSG (如果有的話)。 NSG 包含輸入和輸出規則，可篩選網路介面的網路流量。 如果 NSG 與網路介面相關聯，則會顯示相關聯 NSG 的名稱。 若要修改顯示的內容，請參閱[建立或取消與網路安全性群組的關聯](#associate-or-dissociate-a-network-security-group)。
    - **屬性︰** 顯示有關網路介面的重要設定，包括其 MAC 位址 (如果網路介面未連結至虛擬機器，則位址會空白) 以及其所在的訂用帳戶。
    - **有效安全性規則︰** 如果網路介面已連結至執行中的虛擬機器，而且 NSG 與網路介面、為它指派的子網路或兩者相關聯，則會列出安全性規則。 若要深入了解顯示的內容，請參閱[檢視有效的安全性規則](#view-effective-security-rules)。 若要深入了解 NSG，請參閱[網路安全性群組](security-overview.md)。
    - **有效路由︰** 如果網路介面已連結至執行中的虛擬機器，則會列出路由。 路由是下列各項的組合：Azure 預設路由、任何使用者定義的路由，以及為網路介面所指派子網路的任何可能 BGP 路由。 若要深入了解所顯示的內容，請參閱[檢視有效的路由](#view-effective-routes)。 若要深入了解 Azure 預設路由和使用者定義的路由，請閱讀[路由概觀](virtual-networks-udr-overview.md)。
    - **一般 Azure Resource Manager 設定：** 若要深入了解一般 Azure Resource Manager 設定，請參閱[活動記錄](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#activity-logs)、[存取控制 (IAM)](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#access-control)、[標記](../azure-resource-manager/resource-group-using-tags.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[鎖定](../azure-resource-manager/resource-group-lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)和[自動化指令碼](../azure-resource-manager/resource-manager-export-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json#export-the-template-from-resource-group)。

<a name="view-settings-commands"></a>**命令**

如果指派給網路介面的是 IPv6 位址，PowerShell 輸出會傳回已指派位址的事實，但不會傳回指派的位址。 同樣地，CLI 在其輸出中會傳回已指派位址的事實，但位址則為 *null*。

|工具|命令|
|---|---|
|CLI|[az network nic list](/cli/azure/network/nic#az_network_nic_list) 用以檢視訂用帳戶中的網路介面；[az network nic show](/cli/azure/network/nic#az_network_nic_show) 用以檢視網路介面的設定|
|PowerShell|[Get-AzureRmNetworkInterface](/powershell/module/azurerm.network/get-azurermnetworkinterface) 用以檢視訂用帳戶中的網路介面；或檢視網路介面的設定|

## <a name="change-dns-servers"></a>變更 DNS 伺服器

DNS 伺服器是由 Azure DHCP 伺服器指派給虛擬機器作業系統內的網路介面。 指派的 DNS 伺服器就是網路介面的任何 DNS 伺服器設定。 若要深入了解網路介面的名稱解析設定，請參閱[虛擬機器的名稱解析](virtual-networks-name-resolution-for-vms-and-role-instances.md)。 網路介面可以從虛擬網路繼承設定，或使用它自己的唯一設定來覆寫虛擬網路的設定。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 從清單中選取您要變更 DNS 伺服器的網路介面。
3. 選取 [設定] 底下的 [DNS 伺服器]。
4. 選取任一個選項：
    - **繼承自虛擬網路**︰選擇此選項，可針對網路介面所指派的虛擬網路，繼承為虛擬網路所定義的 DNS 伺服器設定。 在虛擬網路層級，可定義自訂 DNS 伺服器或 Azure 提供的 DNS 伺服器。 Azure 提供的 DNS 伺服器可為指派給相同虛擬網路的資源解析其主機名稱。 對於指派給不同虛擬網路的資源，則必須使用 FQDN 來解析。
    - **自訂**︰您可以設定自己的 DNS 伺服器，以解析跨多個虛擬網路的名稱。 輸入您要做為 DNS 伺服器之伺服器的 IP 位址。 您指定的 DNS 伺服器位址只會指派給此網路介面，並會覆寫為此網路介面指派的虛擬網路的任何 DNS 設定。
5. 選取 [ **儲存**]。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic#az_network_nic_update)|
|PowerShell|[Set-AzureRmNetworkInterface](/powershell/module/azurerm.network/set-azurermnetworkinterface)|

## <a name="enable-or-disable-ip-forwarding"></a>啟用或停用 IP 轉送

IP 轉送讓網路介面連接的虛擬機器能夠：
- 接收以下網路流量：其目的地不是指派給任何 IP 組態 (已指派給網路介面) 的其中一個 IP 位址。
- 傳送以下網路流量：其來源 IP 位址不同於指派給其中一個網路介面之 IP 組態的 IP 位址。

對於接收需轉送之流量的虛擬機器，其所連接的每個網路介面，都必須啟用此設定。 無論虛擬機器是連接多個網路介面或連接單一網路介面，都可以轉送流量。 IP 轉送雖然是 Azure 設定，但虛擬機器也必須執行能夠轉送流量的應用程式，例如防火牆、WAN 最佳化及負載平衡應用程式。 當虛擬機器執行網路應用程式時，虛擬機器通常稱為網路虛擬設備。 您可以檢視 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances) 中可立即部署的網路虛擬設備清單。 IP 轉送通常使用於使用者定義的路由。 若要深入了解使用者定義的路由，請參閱[使用者定義的路由](virtual-networks-udr-overview.md)。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 選取您要啟用或停用 IP 轉送的網路介面。
3. 選取 [設定] 區段中的 [IP 設定]。
4. 選取 [已啟用] 或 [已停用] (預設設定) 來變更設定。
5. 選取 [ **儲存**]。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic#az_network_nic_update)|
|PowerShell|[Set-AzureRmNetworkInterface](/powershell/module/azurerm.network/set-azurermnetworkinterface)|

## <a name="change-subnet-assignment"></a>變更子網路指派

您可以變更為網路介面指派的子網路 (而非虛擬網路)。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 從清單中選取您要變更子網路指派的網路介面。
3. 選取 [設定] 底下的 [IP 設定]。 如果所列 IP 組態的私人 IP 位址旁邊有 **(靜態)** 字樣，您必須完成下列步驟以將 IP 位址的指派方法變更為動態。 您必須使用動態指派方法來指派所有的私人 IP 位址，以便變更網路介面的子網路指派。 如果您使用動態方法來指派位址，請繼續執行步驟 5。 如果您使用靜態指派方法指派了任何 IPv4 位址，請完成下列步驟來將指派方法變更為動態︰
    - 從 IP 設定清單中，選取您要變更 IPv4 位址指派方法的 IP 設定。
    - 針對私人 IP 位址**指派**方法選取 [動態]。 您無法使用靜態指派方法指派 IPv6 位址。
    - 選取 [ **儲存**]。
4. 從 [子網路] 下拉式清單中，選取要將網路介面移至其中的子網路。
5. 選取 [ **儲存**]。 系統就會從新的子網路位址範圍指派新的動態位址。 在對新的子網路指派網路介面之後，您可以從新的子網路位址範圍指派靜態 IPv4 位址 (如果您做此選擇的話)。 若要深入了解如何為網路介面新增、變更和移除 IP 位址，請參閱[管理 IP 位址](virtual-network-network-interface-addresses.md)。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic ip-config update](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_update)|
|PowerShell|[Set-AzureRmNetworkInterfaceIpConfig](/powershell/module/azurerm.network/set-azurermnetworkinterfaceipconfig)|

## <a name="add-to-or-remove-from-application-security-groups"></a>新增到應用程式安全性群組或從中移除

如果網路介面是連接至虛擬機器，您只能使用入口網站在應用程式安全性群組中新增或移除網路介面。 無論網路介面是否連接至虛擬機器，您都可以使用 PowerShell 或 Azure CLI 在應用程式安全性群組中新增或移除網路介面。 深入了解[應用程式安全性群組](security-overview.md#application-security-groups)和如何[建立應用程式安全性群組](manage-network-security-group.md#create-an-application-security-group)。

1. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，開始輸入您想要在應用程式安全性群組中新增或移除其網路介面的虛擬機器名稱。 當 VM 的名稱出現在搜尋結果中時，請加以選取。
2. 在 [設定] 底下，選取 [網路]。  選取 [設定應用程式安全性群組]，選取您想要新增網路介面的應用程式安全性群組，或取消選取您想要移除網路介面的應用程式安全性群組，然後選取 [儲存]。 只有同一個虛擬網路中存在的網路介面可新增到同一個應用程式安全性群組。 應用程式安全性群組必須存在於網路介面所在的同一個位置。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic#az_network_nic_update)|
|PowerShell|[Set-AzureRmNetworkInterface](/powershell/module/azurerm.network/set-azurermnetworkinterface)|

## <a name="associate-or-dissociate-a-network-security-group"></a>建立或取消與網路安全性群組的關聯

1. 在入口網站頂端的搜尋方塊中輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 在清單中選取您要與網路安全性群組建立關聯 (或取消關聯) 的網路介面。
3. 選取 [設定] 下的 [網路安全性群組]。
4. 選取 [編輯]。
5. 選取 [網路安全性群組]，然後選取您要與網路介面建立關聯的網路安全性群組，或是選取 [無]，以取消與網路安全性群組的關聯。
6. 選取 [ **儲存**]。

**命令**

- Azure CLI：[az network nic update](/cli/azure/network/nic#az-network-nic-update)
- PowerShell：[Set-AzureRmNetworkInterface](/powershell/module/azurerm.network/set-azurermnetworkinterface)

## <a name="delete-a-network-interface"></a>刪除網路介面

只要網路介面未連接至虛擬機器，您便可將它刪除。 如果網路介面已連接至虛擬機器，您必須先讓虛擬機器進入已停止 (已解除配置) 狀態，接著再中斷連結網路介面與虛擬機器。 若要讓網路介面與虛擬機器中斷連結，請完成[讓網路介面與虛擬機器中斷連結](virtual-network-network-interface-vm.md#remove-a-network-interface-from-a-vm)中的步驟。 但是，如果該網路介面是連接至虛擬機器的唯一網路介面，則無法將它從虛擬機器中斷連結。 虛擬機器必須一律至少有一個連接的網路介面。 刪除虛擬機器會中斷連結所有已連接的網路介面，但不會刪除網路介面。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「網路介面」。 當**網路介面**出現於搜尋結果時，請選取它。
2. 選取要從網路介面清單中刪除之網路介面右邊的 [...]。
3. 選取 [刪除] 。
4. 選取 [是] 以確認刪除網路介面。

當您刪除網路介面時，會釋出指派給它的所有 MAC 或 IP 位址。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic delete](/cli/azure/network/nic#az_network_nic_delete)|
|PowerShell|[Remove-AzureRmNetworkInterface](/powershell/module/azurerm.network/remove-azurermnetworkinterface)|

## <a name="resolve-connectivity-issues"></a>解決連線問題

如果您無法與虛擬機器進行通訊，則網路安全性群組安全規則或對網路介面有效的路由可能會導致此問題。 您有下列選項來協助您解決問題：

### <a name="view-effective-security-rules"></a>檢視有效的安全性規則

連接至虛擬機器的每個網路介面的有效安全性規則，是您在網路安全性群組和[預設安全性規則](security-overview.md#default-security-rules)中建立的規則組合。 了解網路介面的有效安全性規則，可協助您判斷為什麼無法與虛擬機器進行通訊的原因。 您可以檢視連結至執行中虛擬機器之任何網路介面的有效規則。

1. 在入口網站頂端的搜尋方塊中，輸入您要檢視其有效安全性規則的虛擬機器名稱。 如果您不知道虛擬機器的名稱，請在搜尋方塊中輸入「虛擬機器」。 當搜尋結果中出現**虛擬機器**時加以選取，然後從清單中選取虛擬機器。
2. 在 [設定] 底下，選取 [網路]。
3. 選取網路介面的名稱。
4. 在 [支援 + 疑難排解] 底下，選取 [有效的安全性規則]。
5. 檢閱有效的安全性規則清單，以判斷您所需的輸入和輸出通訊是否存在正確的規則。 請參閱[網路安全性群組概觀](security-overview.md)，深入了解您在此清單中看到的內容。

Azure 網路監看員的 IP 流程驗證功能，也可以協助您判斷安全規則是否阻止虛擬機器和端點之間的通訊。 若要深入了解，請參閱 [IP 流程驗證](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

**命令**

- Azure CLI：[az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg)
- PowerShell：[Get-AzureRmEffectiveNetworkSecurityGroup](/powershell/module/azurerm.network/get-azurermeffectivenetworksecuritygroup) 

### <a name="view-effective-routes"></a>檢視有效的路由

連接至虛擬機器網路介面的有效路由是下列路由的組合：預設路由、您所建立的任何路由，以及透過 BGP 經由 Azure 虛擬網路閘道從內部部署網路傳播的任何路由。 了解網路介面的有效路由，可協助您判斷為什麼無法與虛擬機器進行通訊的原因。 您可以檢視連結至執行中虛擬機器之任何網路介面的有效路由。

1. 在入口網站頂端的搜尋方塊中，輸入您要檢視其有效安全性規則的虛擬機器名稱。 如果您不知道虛擬機器的名稱，請在搜尋方塊中輸入「虛擬機器」。 當搜尋結果中出現**虛擬機器**時加以選取，然後從清單中選取虛擬機器。
2. 在 [設定] 底下，選取 [網路]。
3. 選取網路介面的名稱。
4. 在 [支援 + 疑難排解] 底下，選取 [有效路由]。
5. 檢閱有效的路由清單，以判斷您所需的輸入和輸出通訊是否存在正確的路由。 請參閱[路由概觀](virtual-networks-udr-overview.md)，深入了解您在此清單中看到的內容。

Azure 網路監看員的下一個躍點功能，也可協助您判斷路由是否阻止虛擬機器和端點之間的通訊。 如需深入了解，請參閱[下一個躍點](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

**命令**

- Azure CLI：[az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table)
- PowerShell：[Get-AzureRmEffectiveRouteTable](/powershell/module/azurerm.network/get-azurermeffectiveroutetable)

## <a name="permissions"></a>權限

若要針對網路介面執行工作，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表所列適當權限的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                                                     | 名稱                                                      |
| ---------                                                                  | -------------                                             |
| Microsoft.Network/networkInterfaces/read                                   | 取得網路介面                                     |
| Microsoft.Network/networkInterfaces/write                                  | 建立或更新網路介面                        |
| Microsoft.Network/networkInterfaces/join/action                            | 將網路介面連結至虛擬機器           |
| Microsoft.Network/networkInterfaces/delete                                 | 刪除網路介面                                  |
| Microsoft.Network/networkInterfaces/joinViaPrivateIp/action                | 透過服務將資源加入網路介面     |
| Microsoft.Network/networkInterfaces/effectiveRouteTable/action             | 取得網路介面的有效路由表               |
| Microsoft.Network/networkInterfaces/effectiveNetworkSecurityGroups/action  | 取得網路介面的有效安全性群組           |
| Microsoft.Network/networkInterfaces/loadBalancers/read                     | 取得網路介面負載平衡器                      |
| Microsoft.Network/networkInterfaces/serviceAssociations/read               | 取得服務關聯                                   |
| Microsoft.Network/networkInterfaces/serviceAssociations/write              | 建立或更新服務關聯                    |
| Microsoft.Network/networkInterfaces/serviceAssociations/delete             | 刪除服務關聯                                |
| Microsoft.Network/networkInterfaces/serviceAssociations/validate/action    | 驗證服務關聯                              |
| Microsoft.Network/networkInterfaces/ipconfigurations/read                  | 取得網路介面 IP 設定                    |

## <a name="next-steps"></a>後續步驟

- 使用 [Azure CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 建立具有多個 NIC 的 VM
- 使用 [Azure CLI](virtual-network-multiple-ip-addresses-cli.md) 或 [PowerShell](virtual-network-multiple-ip-addresses-powershell.md) 建立具有多個 IPv4 位址的單一 NIC VM
- 使用 [Azure CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Azure Resource Manager 範本](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)建立具有私人 IPv6 位址的單一 NIC VM (位於 Azure Load Balancer 後方)
- 使用 [PowerShell](powershell-samples.md) 或 [Azure CLI](cli-samples.md) 範例指令碼，或使用 Azure [Resource Manager 範本](template-samples.md)建立網路介面
- 為虛擬網路建立及套用 [Azure 原則](policy-samples.md)