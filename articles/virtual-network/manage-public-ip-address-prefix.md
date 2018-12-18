---
title: 建立、變更或刪除 Azure 公用 IP 位址首碼 | Microsoft Docs
description: 了解如何建立、變更或刪除公用 IP 位址首碼。
services: virtual-network
documentationcenter: na
author: anavinahar
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: anavin
ms.openlocfilehash: 71dee9f36a3e8e11cc23f966c9157f6409a3b5e0
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405728"
---
# <a name="create-change-or-delete-a-public-ip-address-prefix"></a>建立、變更或刪除公用 IP 位址首碼

了解公用 IP 位址首碼，以及如何建立、變更和刪除公用 IP 位址首碼。 公用 IP 位址首碼是根據您所指定之公用 IP 位址數目的連續位址範圍。 這些位址會指派給您的訂用帳戶。 當您建立公用 IP 位址資源時，可以從首碼指派靜態公用 IP 位址，並關聯至虛擬機器、負載平衡器或其他資源，以啟用網際網路連線。 如果您不熟悉公用 IP 位址首碼，請參閱[公用 IP 位址首碼概觀](public-ip-address-prefix.md)

## <a name="before-you-begin"></a>開始之前

> [!IMPORTANT]
> 公用 IP 位址首碼在有限的區域內為公開預覽。 您可以[了解預覽的意義](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 公用 IP 位址首碼目前已於下列區域提供：美國中西部、美國西部、美國西部 2、美國中部、北歐、西歐和東南亞。 如需區域的更新清單，請前往 [Azure 更新](https://azure.microsoft.com/updates/?product=virtual-network)。

在完成本文任一節的步驟之前，請先完成下列工作︰

- 如果您還沒有 Azure 帳戶，請註冊[免費試用帳戶](https://azure.microsoft.com/free)。
- 如果使用入口網站，請開啟 https://aka.ms/publicipprefixportal，並使用您的 Azure 帳戶來登入。
- 如果使用 PowerShell 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/powershell) \(英文\) 中執行命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 本教學課程需要 AzureRm.Network PowerShell 模組 6.3.1 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM.Network` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](https://github.com/Azure/azure-powershell/releases/tag/AzureRm.Network.6.3.1)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。
- 如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本教學課程需要 Azure CLI 2.0.41 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

您登入或連線到 Azure 的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派[權限](#permissions)中所列適當動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

公用 IP 位址首碼需要付費。 如需詳細資料，請參閱[定價](https://azure.microsoft.com/pricing/details/ip-addresses)。

## <a name="create-a-public-ip-address-prefix"></a>建立公用 IP 位址首碼

1. 在入口網站的左上角，選取 [+ 建立資源]。
2. 在 [搜尋 Marketplace] 方塊中，輸入「公用 IP 位址首碼」。 當 [公用 IP 位址首碼] 出現於搜尋結果時，將其選取。
3. 在 [公用 IP 位址首碼] 下方，選取 [建立]。
4. 在 [建立公用 IP 位址首碼] 下方，輸入或選取下列設定的值，然後選取 [建立]：

   |設定|必要？|詳細資料|
   |---|---|---|
   |訂用帳戶|是|所在的[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)必須與您想要與公用 IP 位址建立關聯的資源相同。|
   |資源群組|是|所在的[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)可以與您想要與公用 IP 位址建立關聯的資源相同或不同。|
   |名稱|是|名稱必須是您選取的資源群組中唯一的名稱。|
   |區域|是|必須與您從範圍指派位址的公用 IP 位址存在於相同[區域](https://azure.microsoft.com/regions)。 首碼目前在下列區域已供預覽：美國中西部、美國西部、美國西部 2、美國中部、北歐、西歐和東南亞。|
   |首碼大小|是| 您需要的首碼大小。 /28 或 16 個 IP 位址為預設值。 

**命令**


|工具|命令|
|---|---|
|CLI|[az network public-ip prefix create](/cli/azure/network/public-ip/prefix/az-network-public-ip-prefix-create)|
|PowerShell|[New-AzureRmPublicIpAddressPrefix](/powershell/module/azurerm.network/new-azurermpublicipaddressprefix)|

## <a name="create-a-static-public-ip-address-from-a-prefix"></a>從首碼建立靜態公用 IP 位址
建立首碼之後，您必須從首碼建立靜態 IP 位址。 若要這樣做，請遵循下面的步驟。

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，鍵入「公用 IP 位址首碼」。 當 [公用 IP 位址首碼] 出現於搜尋結果時，將其選取。
2. 選取您要從中建立公用 IP 的首碼。
3. 當它出現在搜尋結果時，請選取它，然後按一下 [概觀] 區段中的 [+ 新增 IP 位址]。 如果您沒有看到它，請確定您使用正確的連結進行預覽： https://aka.ms/publicipprefixportal
4. 在 [建立公用 IP 位址] 下方，輸入或選取下列設定的值。 由於首碼適用於標準 SKU、IPv4 和靜態，因此您只需要提供下列資訊：

   |設定|必要？|詳細資料|
    |---|---|---|
    |名稱|是|公用 IP 位址名稱在您選取的資源群組中必須是唯一。|
   |閒置逾時 (分鐘)|否|不需依賴用戶端傳送保持連線訊息，讓 TCP 或 HTTP 連線保持開啟的分鐘數。 |
   |DNS 名稱標籤|否|在您建立名稱的 Azure 區域 (跨越所有訂用帳戶和所有客戶) 中必須是唯一。 Azure 會在其 DNS 中自動登錄名稱和 IP 位址，以便您連線至具有此名稱的資源。 Azure 會將 *location.cloudapp.azure.com* (其中 location 是您選取的位置) 之類的預設子網路附加至您提供的名稱，以建立完整的 DNS 名稱。如需詳細資訊，請參閱[使用具有 Azure 公用 IP 位址的 Azure DNS](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address)。|

## <a name="view-or-delete-a-prefix"></a>檢視或刪除首碼

1. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，鍵入「公用 IP 位址首碼」。 當 [公用 IP 位址首碼] 出現於搜尋結果時，將其選取。
2. 選取您要檢視、變更設定，或從清單中刪除的公用 IP 位址首碼名稱。
3. 根據您要檢視、刪除或變更公用 IP 位址首碼，完成下列其中一個選項。
    - **檢視**：[概觀] 區段會顯示公用 IP 位址首碼的索引鍵設定，例如首碼。
    - **刪除**：若要刪除公用 IP 位址首碼，請在 [概觀] 區段中選取 [刪除]。 如果首碼內的位址與公用 IP 位址資源相關聯，則您必須先刪除公用 IP 位址資源。 請參閱[刪除公用 IP 位址](virtual-network-public-ip-address.md#view-change-settings-for-or-delete-a-public-ip-address)。

**命令**

|工具|命令|
|---|---|
|CLI|[az network public-ip prefix list](/cli/azure/network/public-ip/prefix/az-network-public-ip-prefix-list) 可列出公用 IP 位址、[az network public-ip prefix show](/cli/azure/network/public-ip/prefix/az-network-public-ip-prefix-show) 可顯示設定；[az network public-ip prefix update](/cli/azure/network/public-ip-prefix#az-network-public-ip-prefix-update) 可進行更新；[az network public-ip prefix delete](/cli/azure/network/public-ip/prefix/az-network-public-ip-prefix-delete) 可進行刪除|
|PowerShell|[Get-AzureRmPublicIpAddressPrefix](/powershell/module/azurerm.network/get-azurermpublicipaddressprefix) 可擷取公用 IP 位址物件並檢視其設定、[Set-AzureRmPublicIpAddressPrefix](/powershell/module/azurerm.network/set-azurermpublicipaddressprefix) 可更新設定；[Remove-AzureRmPublicIpAddressPrefix](/powershell/module/azurerm.network/remove-azurermpublicipaddressprefix) 可進行刪除|

## <a name="permissions"></a>權限

若要針對公用 IP 位址首碼執行工作，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表所列適當動作的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                                                   | 名稱                                                           |
| ---------                                                                | -------------                                                  |
| Microsoft.Network/publicIPPrefixes/read                           | 讀取公用 IP 位址首碼                                |
| Microsoft.Network/publicIPPrefixes/write                          | 建立或更新公用 IP 位址首碼                    |
| Microsoft.Network/publicIPPrefixes/delete                         | 刪除公用 IP 位址首碼                              |
|Microsoft.Network/publicIPPrefixes/join/action | 從首碼建立公用 IP 位址 |

## <a name="next-steps"></a>後續步驟

- 了解使用[公用 IP 首碼](public-ip-address-prefix.md)的案例和優點
