---
title: 建立、變更或刪除 Azure 公用 IP 位置 | Microsoft Docs
description: 了解如何建立、變更或刪除公用 IP 位址。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: bb71abaf-b2d9-4147-b607-38067a10caf6
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: jdial
ms.openlocfilehash: b9ab32764e6a5e780625618d0efa3d2a9c19cd35
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983526"
---
# <a name="create-change-or-delete-a-public-ip-address"></a>建立、變更或刪除公用 IP 位址

了解公用 IP 位址，以及如何建立、變更和刪除公用 IP 位址。 公用 IP 位址是可加以設定的資源。 將公用 IP 位址指派給支援公用 IP 位址的 Azure 資源，可以：
- 啟用從網路網路到資源的輸入通訊；資源包括 Azure 虛擬機器 (VM)、Azure 應用程式閘道、Azure Load Balancer、Azure VPN 閘道等等。 即使未將公用 IP 位址指派給該虛擬機器，只要虛擬機器是負載平衡器後端集區的一部分，且已將公用 IP 位址指派給負載平衡器，就可以讓您與來自網際網路的虛擬機器等資源進行通訊。 若要判斷資源是否可以將公用 IP 位址指派給特定的 Azure 服務，或者是否可以透過不同 Azure 資源的公用 IP 位址與其進行通訊，請參閱服務的說明文件。 
- 使用可預測 IP 位址對網際網路進行輸出連線。 例如，虛擬機器不需有指派的公用 IP 位址，即可對網際網路進行輸出通訊，但其位址是由 Azure 轉譯而成的網路位址 (依預設是無法預測的公用位址)。 指派公用 IP 位址給資源，可讓您知道輸出連線所使用的 IP 位址。 雖然可預測，但位址可能根據選擇的指派方法而有所變更。 如需詳細資訊，請參閱＜[建立公用 IP 位址](#create-a-public-ip-address)＞。 若要深入了解 Azure 資源的輸出連線，請參閱[了解輸出連線](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="before-you-begin"></a>開始之前

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://portal.azure.com，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.31 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

您登入或連線到 Azure 的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派[權限](#permissions)中所列適當動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

公用 IP 位址需要少許費用。 若要檢視價格，請閱讀 [IP 位址價格](https://azure.microsoft.com/pricing/details/ip-addresses)頁面。 

## <a name="create-a-public-ip-address"></a>建立公用 IP 位址

1. 在入口網站的左上角，選取 [+ 建立資源]。
2. 在 [搜尋 Marketplace] 方塊中，輸入「公用 IP 位址」。 當「公用 IP 位址」出現於搜尋結果時，將其選取。
3. 在 [公用 IP 位址] 下方，選取 [建立]。
4. 在 [建立公用 IP 位址] 下方，輸入或選取下列設定的值，然後選取 [建立]：

    |設定|必要？|詳細資料|
    |---|---|---|
    |名稱|是|名稱必須是您選取的資源群組中唯一的名稱。|
    |SKU|是|在 SKU 推出之前所建立的公用 IP 位址全都是**基本** SKU 的公用 IP 位址。  建立公用 IP 位址之後，即無法變更 SKU。 獨立虛擬機器、可用性設定組內的虛擬機器，或虛擬機器擴展集，可以使用基本或標準 SKU。  不允許混用可用性設定組或擴展集內虛擬機器之間的 SKU。 **基本** SKU：如果您要在支援可用性區域的區域中建立公用 IP 位址，**可用性區域**設定依預設會設為「無」。 您可以選擇選取可用性區域，以確保您公用 IP 位址的特定區域。 **標準** SKU：標準 SKU 公用 IP 可與虛擬機器或負載平衡器前端建立關聯。 如果您要在支援可用性區域的區域中建立公用 IP 位址，**可用性區域**設定依預設會設為「區域備援」。 如需可用性區域的詳細資訊，請參閱**可用性區域**設定。 如果您要將位址與標準負載平衡器建立關聯，則需要標準 SKU。 若要深入了解標準負載平衡器，請參閱 [Azure 負載平衡器標準 SKU](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 當您將標準 SKU 的公用 IP 位址指派給虛擬機器的網路介面時，必須使用[網路安全性群組](security-overview.md#network-security-groups)明確地允許預定的流量。 在建立和關聯網路安全性群組並明確地允許所要流量前，與資源進行的通訊都會失敗。|
    |IP 版本|是| 選取 IPv4 或 IPv6。 公用的 IPv4 位址可指派給數個 Azure 資源，而 IPv6 公用 IP 位址只可指派給網際網路面向的負載平衡器。 負載平衡器可將 IPv6 的流量負載分散到 Azure 虛擬機器。 深入了解[將 IPv6 流量負載分散到虛擬機器](../load-balancer/load-balancer-ipv6-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 如果您選取**標準 SKU**，則無法選擇 *IPv6*。 使用**標準 SKU** 時，您只能建立 IPv4 位址。|
    |IP 位址指派|是|**動態︰** 只有在公用 IP 位址與 Azure 資源相關聯，且第一次啟動資源之後，才會指派動態位址。 動態位址指派給資源時可以變更 (例如指派給虛擬機器，而虛擬機器停止 (解除配置)，然後再重新開機)。 如果虛擬機器已重新開機或停止 (但未解除配置)，則位址維持不變。 當公用 IP 位址資源與其所關聯的資源中斷關聯時，便會釋放動態位址。 **靜態︰** 建立公用 IP 位址時會指派靜態位址。 刪除公用 IP 位址資源之前，不會釋出靜態位址。 如果位址與資源沒有關聯，您可以在位址建立後變更指派方法。 如果位址與資源相關聯，您可能無法變更指派方法。 如果您選取 [IPv6] 作為 [IP 版本]，則指派方法為「動態」。 如果您為 [SKU] 選取 [標準]，則指派方法為「靜態」。|
    |閒置逾時 (分鐘)|否|不需依賴用戶端傳送保持連線訊息，讓 TCP 或 HTTP 連線保持開啟的分鐘數。 如果您選取 IPv6 作為 **IP 版本**，則無法變更此值。 |
    |DNS 名稱標籤|否|在您建立名稱的 Azure 位置 (跨越所有訂用帳戶和所有位置) 中必須是唯一的。 Azure 會在其 DNS 中自動登錄名稱和 IP 位址，以便您連線至具有此名稱的資源。 Azure 會將 *location.cloudapp.azure.com* (其中 location 是您選取的位置) 之類的預設子網路附加至您提供的名稱 ，以建立完整的 DNS 名稱。 如果您選擇兩個位址版本都建立，則會指派相同的 DNS 名稱給 IPv4 和 IPv6 位址。 Azure 預設 DNS 包含 IPv4 A 和 IPv6 AAAA 名稱記錄，並且會在查詢 DNS 名稱時回應這兩個記錄。 用戶端選擇要與哪一個位址 (IPv4 或 IPv6) 通訊。 可改為 (或同時) 使用具有預設尾碼的 DNS 名稱標籤，您可以使用 Azure DNS 服務來設定 DNS 名稱，其具有解析為公用 IP 位址的自訂尾碼。 如需詳細資訊，請參閱[使用具有 Azure 公用 IP 位址的 Azure DNS](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address)。|
    |建立 IPv6 (或 IPv4) 位址|否| 顯示 IPv6 或 IPv4 取決於您選取的 **IP 版本**。 例如，如果您選取 **IPv4** 作為 **IP 版本**，則此處會顯示 **IPv6**。 如果您為 **SKU** 選取「標準」，則無法建立 IPv6 位址。
    |名稱 (僅在您核取 [建立 IPv6 (或 IPv4) 位址] 核取方塊時顯示)|是 (如果您選取 [建立 IPv6] \(或 IPv4) 核取方塊)。|該名稱必須與此清單中的第一個**名稱**不同。 如果您選擇同時建立 IPv4 和 IPv6 位址，則入口網站會建立兩個個別的公用 IP 位址資源，並各指派一個 IP 位址版本。|
    |IP 位址指派 (僅在您核取 [建立 IPv6 (或 IPv4) 位址] 核取方塊時顯示)|是 (如果您選取 [建立 IPv6] \(或 IPv4) 核取方塊)。|如果核取方塊顯示**建立 IPv4 位址**，表示您可以選擇指派方法。 如果核取方塊顯示**建立 IPv6 位址**，表示您無法選擇指派方法，因為指派方法必須為**動態**。|
    |訂用帳戶|是|所在的[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)必須與您想要與公用 IP 位址建立關聯的資源相同。|
    |資源群組|是|所在的[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)可以與您想要與公用 IP 位址建立關聯的資源相同或不同。|
    |位置|是|所在的[位置](https://azure.microsoft.com/regions) (也稱為區域) 必須與您想要與公用 IP 位址建立關聯的資源相同。|
    |可用性區域| 否 | 只有在您選取受支援的位置時，才會出現此設定。 如需受支援位置的清單，請參閱[可用性區域概觀](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 如果您選取**基本** SKU，則會自動為您選取「無」。 如果您想要保證特定區域，可選取特定區域。 或選擇非區域備援。 如果您選取**標準** SKU：會自動為您選取區域備援，並針對區域失敗進行資料路徑復原。 如果您希望保證對區域失敗無法復原的特定區域，則可選取特定區域。

**命令**

雖然入口網站提供建立兩個公用 IP 位址資源 (一個 IPv4 和一個 IPv6) 的選項，但下列的 CLI 和 PowerShell 命令則是會以其中一個 IP 版本的位址建立一個資源。 如果您想要兩個公用 IP 位址資源 (每個 IP 版本各一個)，您必須執行該命令兩次，並針對公用 IP 位址資源指定不同名稱和版本。

|工具|命令|
|---|---|
|CLI|[az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create)|
|PowerShell|[New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress)|

## <a name="view-change-settings-for-or-delete-a-public-ip-address"></a>檢視、變更公用 IP 位址的設定，或刪除公用 IP 位址

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入「公用 ip 位址」。 當「公用 IP 位址」出現於搜尋結果時，將其選取。
2. 選取您要檢視、變更設定，或從清單中刪除的公用 IP 位址名稱。
3. 根據您要檢視、刪除或變更公用 IP 位址，完成下列其中一個選項。
    - **檢視**：[概觀] 區段會顯示公用 IP 位址的主要設定，例如和該位址關聯的網路介面 (如果位址與網路介面關聯)。 入口網站不會顯示位址版本 (IPv4 或 IPv6)。 若要檢視版本資訊，請使用 PowerShell 或 CLI 命令來檢視公用 IP 位址。 如果 IP 位址版本是 IPv6 時，指派的位址不會顯示在入口網站、PowerShell 或 CLI 。
    - **刪除**：若要刪除公用 IP 位址，請在 [概觀] 區段中選取 [刪除]。 如果位址目前與 IP 組態相關聯，則無法加以刪除。 如果位址目前與組態相關聯，請選取 [解除關聯] 來解除位址與 IP 組態的關聯。
    - **變更**：選取 [組態]。 使用[建立公用 IP 位址](#create-a-public-ip-address)中步驟 4 的資訊來變更設定。 若要將 IPv4 位址的指派從靜態變更為動態，您必須先解除公用 IPv4 位址與相關聯 IP 組態的關聯。 您可以接著將指派方法變更為動態，然後選取 [關聯] 讓 IP 位址與相同 IP 組態、不同組態建立關聯，您也可以讓它解除關聯。 若要解除公用 IP 位址的關聯，請在 [概觀] 區段中，選取 [解除關聯]。

    >[!WARNING]
    >當您將指派方法從靜態變更為動態時，您會遺失已指派給公用 IP 位址的 IP 位址。 雖然 Azure 公用 DNS 伺服器會維護靜態或動態位址與任何 DNS 名稱標籤 (如果您定義一個位置) 之間的對應，但是動態 IP 位址可能會在虛擬機器處於停止 (已解除配置) 狀態後啟動時變更。 若要防止位址變更，請指派靜態 IP 位址。

**命令**

|工具|命令|
|---|---|
|CLI|[az network public-ip list](/cli/azure/network/public-ip#az-network-public-ip-list) 可列出公用 IP 位址、[az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) 可顯示設定；[az network public-ip update](/cli/azure/network/public-ip#az-network-public-ip-update) 可進行更新；[az network public-ip delete](/cli/azure/network/public-ip#az-network-public-ip-delete) 可進行刪除|
|PowerShell|[Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 可擷取公用 IP 位址物件並檢視其設定、[Set-AzureRmPublicIpAddress](/powershell/module/azurerm.network/set-azurermpublicipaddress) 可更新設定；[Remove-AzureRmPublicIpAddress](/powershell/module/azurerm.network/remove-azurermpublicipaddress) 可進行刪除|

## <a name="assign-a-public-ip-address"></a>指派公用 IP 位址

了解如何將公用 IP 位址指派給下列資源：

- [Windows](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) VM (建立期間)，或指派給[現有的 VM](virtual-network-network-interface-addresses.md#add-ip-addresses)
- [網際網路對應負載平衡器](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Azure 應用程式閘道](../application-gateway/application-gateway-create-gateway-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [使用 Azure VPN 閘道的站對站連線](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Azure 虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-portal-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

## <a name="permissions"></a>權限

若要針對公用 IP 位址執行工作，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表所列適當動作的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                                             | 名稱                                                           |
| ---------                                                          | -------------                                                  |
| Microsoft.Network/publicIPAddresses/read                           | 讀取公用 IP 位址                                          |
| Microsoft.Network/publicIPAddresses/write                          | 建立或更新公用 IP 位址                           |
| Microsoft.Network/publicIPAddresses/delete                         | 刪除公用 IP 位址                                     |
| Microsoft.Network/publicIPAddresses/join/action                    | 將公用 IP 位址與資源建立關聯                    |

## <a name="next-steps"></a>後續步驟

- 使用 [PowerShell](powershell-samples.md) 或 [Azure CLI](cli-samples.md) 範例指令碼，或使用 Azure [Resource Manager 範本](template-samples.md)建立公用 IP 位址
- 為公用 IP 位址建立及套用 [Azure 原則](policy-samples.md)
