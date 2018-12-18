---
title: 診斷虛擬機器網路流量篩選問題 | Microsoft Docs
description: 了解如何藉由檢視虛擬機器的有效安全性規則來診斷虛擬機器網路流量篩選問題。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: a54feccf-0123-4e49-a743-eb8d0bdd1ebc
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2018
ms.author: jdial
ms.openlocfilehash: 366ff0b59835ca3a28cafd5de77c0bd645ff58c5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984223"
---
# <a name="diagnose-a-virtual-machine-network-traffic-filter-problem"></a>診斷虛擬機器網路流量篩選問題

在本文中，您將了解如何藉由檢視對虛擬機器 (VM) 有效的網路安全性群組 (NSG) 安全性規則來診斷網路流量篩選問題。

NSG 可讓您針對流入和流出 VM 的流量，控制流量的類型。 您可以將 NSG 關聯至 Azure 虛擬網路中的子網路、附加至 VM 的網路介面，或是上述兩者。 套用至網路介面的有效安全性規則，乃是針對網路介面及該網路介面所在子網路關聯的 NSG，存在其中的規則彙總。 不同 NSG 中的規則有時會互相衝突，並影響 VM 的網路連線能力。 您可以從 VM 網路介面上套用的 NSG 中檢視所有的有效安全性規則。 如果您不熟悉虛擬網路、網路介面或 NSG 概念，請參閱[虛擬網路概觀](virtual-networks-overview.md)、[網路介面](virtual-network-network-interface.md)及[網路安全性群組概觀](security-overview.md)。

## <a name="scenario"></a>案例

您嘗試從網際網路透過連接埠 80 連線到 VM，但連線失敗。 若要判斷為何無法從網際網路存取連接埠 80，您可以使用 Azure [入口網站](#diagnose-using-azure-portal)、[PowerShell](#diagnose-using-powershell) 或 [Azure CLI](#diagnose-using-azure-cli) 來檢視網路介面的有效安全性規則。

下列步驟假設您具有可檢視有效安全性規則的現有 VM。 如果您沒有現有的 VM，請先部署 [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) VM，以用來完成本文中的工作。 本文中的範例適用於名為 *myVM* 的 VM，且該 VM 具有名為 *myVMVMNic* 的網路介面。 VM 和網路介面皆位於名為 *myResourceGroup* 的資源群組，且位於「美國東部」區域。 請針對您要診斷問題的 VM，適當地變更步驟中的值。

## <a name="diagnose-using-azure-portal"></a>使用 Azure 入口網站進行診斷

1. 使用具有[必要權限](virtual-network-network-interface.md#permissions)的 Azure 帳戶登入 Azure [入口網站](https://portal.azure.com)。
2. 在 Azure 入口網站頂端的搜尋方塊中輸入 VM 的名稱。 當 VM 的名稱出現在搜尋結果中時，請選取它。
3. 在 [設定] 下方，選取 [網路]，如下圖所示：

   ![檢視安全性規則](./media/diagnose-network-traffic-filter-problem/view-security-rules.png)

   上圖所列的規則適用於名為 **myVMVMNic** 的網路介面。 其中有來自兩個不同網路安全性群組的網路介面 [輸入連接埠規則]：
   
   - **mySubnetNSG**：關聯至網路介面所在的子網路。
   - **myVMNSG**：關聯至名為 **myVMVMNic** 的 VM 中網路介面。

   名為 **DenyAllInBound** 的規則是防止從網際網路透過連接埠 80 與 VM 進行輸入通訊的規則，如[案例](#scenario)中所述。 此規則針對 [來源] 列出 *0.0.0.0/0*，這包括網際網路。 沒有其他具較高優先順序 (較低數字) 的規則允許透過連接埠 80 進行輸入。 若要允許從網際網路透過連接埠 80 對 VM 進行輸入，請參閱[解決問題](#resolve-a-problem)。 若要深入了解安全性規則及 Azure 套用它們的方式，請參閱[網路安全性群組](security-overview.md)。

   您也會在圖片底部看到 [輸出連接埠規則]。 位於其下方的是適用於網路介面的輸出連接埠規則。 雖然圖片針對每個 NSG 只有顯示四個輸入規則，您的 NSG 可能會具有四個以上的規則。 在圖中，您會在 [來源] 和 [目的地] 下方看到 [VirtualNetwork]，並在 [來源] 下方看到 [AzureLoadBalancer]。 [VirtualNetwork] 和 [AzureLoadBalancer] 為[服務標籤](security-overview.md#service-tags)。 服務標籤代表一組 IP 位址前置詞，有助於降低建立安全性規則的複雜性。

4. 確認 VM 處於執行中狀態，然後選取 [有效的安全性規則] (如上圖所示)，以查看有效的安全性規則 (如下圖所示)：

   ![檢視有效的安全性規則](./media/diagnose-network-traffic-filter-problem/view-effective-security-rules.png)

   列出的規則與您在步驟 3 中看到的一樣，不過針對關聯至網路介面和子網路的 NSG 則有不同的索引標籤。 如同您可以在圖中看到的，系統只會顯示前 50 個的規則。 若要下載包含所有規則的 .csv 檔案，請選取 [下載]。

   若要查看每個服務標籤所代表的前置詞，請選取其中一個規則 (例如名為 **AllowAzureLoadBalancerInbound** 的規則)。 下圖顯示適用於 **AzureLoadBalancer** 服務標籤的前置詞：

   ![檢視有效的安全性規則](./media/diagnose-network-traffic-filter-problem/address-prefixes.png)

   雖然 **AzureLoadBalancer** 服務標籤只代表一個前置詞，其他服務標籤可能會代表數個前置詞。

5. 先前的步驟示範 **myVMVMNic** 網路介面的安全性規則，但您也在先前某些圖片中看過 **myVMVMNic2** 的網路介面。 此範例中的 VM 具有兩個附加的網路介面。 針對每個網路介面的有效安全性規則可能會不同。

   若要查看適用於 **myVMVMNic2** 網路介面的規則，請選取它。 如下圖所示，與該網路介面的子網路關聯規則與 **myVMVMNic** 網路介面相同，因為這兩個網路介面都位於相同的子網路。 當您將 NSG 關聯至子網路時，系統會將其規則套用至子網路中的所有網路介面。

   ![檢視安全性規則](./media/diagnose-network-traffic-filter-problem/view-security-rules2.png)

   不同於 **myVMVMNic** 網路介面，**myVMVMNic2** 網路介面沒有與它相關聯的網路安全性群組。 每個網路介面和子網路皆可有零個或一個與它相關聯的 NSG。 與每個網路介面或子網路相關聯的 NSG 可以是相同或不同的。 您可以將相同的網路安全性群組關聯至任意數目的網路介面和子網路。

雖然可以透過 VM 檢視有效的安全性規則，您也可以透過下列個別的項目來檢視有效的安全性規則：
- **網路介面**：了解如何[檢視網路介面](virtual-network-network-interface.md#view-network-interface-settings)。
- **NSG**：了解如何[檢視 NSG](manage-network-security-group.md#view-details-of-a-network-security-group)。

## <a name="diagnose-using-powershell"></a>使用 PowerShell 進行診斷

您可以執行 [Azure Cloud Shell](https://shell.azure.com/powershell) 中採用的命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式殼層。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 如果您是從電腦執行 PowerShell，便需要 *AzureRM* PowerShell 模組 6.0.1 版或更新的版本。 請在您的電腦上執行 `Get-Module -ListAvailable AzureRM`，以尋找已安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，還需要執行 `Login-AzureRmAccount` 以使用具有[必要權限](virtual-network-network-interface.md#permissions)的帳戶登入 Azure。

使用 [Get-AzureRmEffectiveNetworkSecurityGroup](/powershell/module/azurerm.network/get-azurermeffectivenetworksecuritygroup) 來取得網路介面的有效安全性規則。 下列範例會針對名為 *myVMVMNic* 的網路介面取得有效的安全性規則，該介面位於名為 *myResourceGroup* 的資源群組中：

```azurepowershell-interactive
Get-AzureRmEffectiveNetworkSecurityGroup `
  -NetworkInterfaceName myVMVMNic interface `
  -ResourceGroupName myResourceGroup
```

輸出會以 json 格式傳回。 若要了解輸出，請參閱[解譯命令輸出](#interpret-command-output)。
只有在 NSG 與網路介面、與網路介面所在的子網路，或是上述兩者相關聯時，才會傳回輸出。 VM 必須處於執行中狀態。 一個 VM 可能有多個套用不同 NSG 的網路介面。 進行疑難排解時，請針對每個網路介面執行命令。

如果您仍然有連線問題，請參閱[其他診斷](#additional-diagnosis)和[考量](#considerations)。

如果您不知道網路介面的名稱，但知道該網路介面所附加至的 VM 名稱，則下列命令會針對所有附加至 VM 的網路介面，傳回其識別碼：

```azurepowershell-interactive
$VM = Get-AzureRmVM -Name myVM -ResourceGroupName myResourceGroup
$VM.NetworkProfile
```

您會收到類似於下列範例的輸出：

```powershell
NetworkInterfaces
-----------------
{/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic
```

在上述輸出中，網路介面名稱是 *myVMVMNic*。

## <a name="diagnose-using-azure-cli"></a>使用 Azure CLI 進行診斷

如果使用命令列介面 (CLI) 命令來完成這篇文章中的工作，請在 [Azure Cloud Shell](https://shell.azure.com/bash) \(英文\) 中執行命令，或從您的電腦執行 CLI。 本文需要 Azure CLI 2.0.32 版或更新的版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，還需要執行 `az login` 並使用具有[必要權限](virtual-network-network-interface.md#permissions)的帳戶登入 Azure。

使用 [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) 來取得網路介面的有效安全性規則。 下列範例會取得 *myResourceGroup* 資源群組中 *myVMVMNic* 網路介面的有效安全性規則：

```azurecli-interactive
az network nic list-effective-nsg \
  --name myVMVMNic \
  --resource-group myResourceGroup
```

輸出會以 json 格式傳回。 若要了解輸出，請參閱[解譯命令輸出](#interpret-command-output)。
只有在 NSG 與網路介面、與網路介面所在的子網路，或是上述兩者相關聯時，才會傳回輸出。 VM 必須處於執行中狀態。 一個 VM 可能有多個套用不同 NSG 的網路介面。 進行疑難排解時，請針對每個網路介面執行命令。

如果您仍然有連線問題，請參閱[其他診斷](#additional-diagnosis)和[考量](#considerations)。

如果您不知道網路介面的名稱，但知道該網路介面所附加至的 VM 名稱，則下列命令會針對所有附加至 VM 的網路介面，傳回其識別碼：

```azurecli-interactive
az vm show \
  --name myVM \
  --resource-group myResourceGroup
```

在傳回的輸出內，您會看到類似下列範例的資訊：

```azurecli
"networkProfile": {
    "additionalProperties": {},
    "networkInterfaces": [
      {
        "additionalProperties": {},
        "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
        "primary": true,
        "resourceGroup": "myResourceGroup"
      },
```

在上述輸出中，網路介面名稱是 *myVMVMNic interface*。

## <a name="interpret-command-output"></a>解譯命令輸出

不論您是使用 [PowerShell](#diagnose-using-powershell) 或 [Azure CLI](#diagnose-using-azure-cli) 來診斷問題，都會收到包含下列資訊的輸出：

- **NetworkSecurityGroup**：網路安全性群組的識別碼。
- **Association**：網路安全性群組是否關聯至 *NetworkInterface* 或 *Subnet*。 如果 NSG 同時關聯至兩者，則會針對每個 NSG 以 **NetworkSecurityGroup**、**Association** 和 **EffectiveSecurityRules** 傳回輸出。 如果在執行命令以檢視有效安全性規則之前建立/解除 NSG 的關聯，您可能需要等候幾秒鐘的時間，變更才會反映在命令輸出中。
- **EffectiveSecurityRules**：[建立安全性規則](manage-network-security-group.md#create-a-security-rule)中會詳述每個屬性的說明。 前面加上 *defaultSecurityRules/* 的規則名稱是存在於每個 NSG 中的預設安全性規則。 前面加上 *securityRules/* 的規則名稱是您已建立的規則。 針對 **destinationAddressPrefix** 或 **sourceAddressPrefix** 屬性指定[服務標籤](security-overview.md#service-tags) (例如 **Internet**、**VirtualNetwork** 和 **AzureLoadBalancer**) 的規則，也會具有 **expandedDestinationAddressPrefix** 屬性值。 **expandedDestinationAddressPrefix** 屬性會列出服務標籤所代表的所有位址前置詞。

如果您看到輸出中有列出重複的規則，那是因為有某個 NSG 同時關聯至網路介面和子網路。 這兩個 NSG 都具有相同的預設規則，且如果您在這兩個 NSG 中自行建立相同的規則，則這兩個 NSG 可能會有其他重複的規則。

名為 **defaultSecurityRules/DenyAllInBound** 的規則是防止從網際網路透過連接埠 80 與 VM 進行輸入通訊的規則，如[案例](#scenario)中所述。 沒有其他具較高優先順序 (較低數字) 的規則允許從網際網路透過連接埠 80 進行輸入。

## <a name="resolve-a-problem"></a>解決問題

無論您是使用 Azure [入口網站](#diagnose-using-azure-portal)、[PowerShell](#diagnose-using-powershell) 或 [Azure CLI](#diagnose-using-azure-cli) 來診斷本文[案例](#scenario)中所呈現的問題，其解決方案都是使用下列屬性來建立網路安全性規則：

| 屬性                | 值                                                                              |
|---------                |---------                                                                           |
| 來源                  | 任意                                                                                |
| 來源連接埠範圍      | 任意                                                                                |
| 目的地             | VM 的 IP 位址、IP 位址範圍，或是子網路中的所有位址。 |
| 目的地連接埠範圍 | 80                                                                                 |
| 通訊協定                | TCP                                                                                |
| 動作                  | 允許                                                                              |
| 優先順序                | 100                                                                                |
| 名稱                    | Allow-HTTP-All                                                                     |

當您建立規則之後，系統就會允許從網際網路經由連接埠 80 進行輸入，因為該規則的優先順序高於名為 *DenyAllInBound* 且會拒絕該流量的預設安全性規則。 了解如何[建立安全性規則](manage-network-security-group.md#create-a-security-rule)。 如果有不同的 NSG 同時關聯至網路介面和子網路，您就必須在那兩個 NSG 中建立相同的規則。

當 Azure 處理輸入流量時，會先處理與子網路關聯 NSG 中的規則 (如果有相關聯的 NSG 的話)，然後再處理與網路介面相關聯 NSG 中的規則。 如果有某個 NSG 同時關聯至網路介面和子網路，就必須同時在那兩個 NSG 中開啟該連接埠來使流量能抵達 VM。 為了簡化管理和通訊問題，我們建議您將 NSG 關聯至子網路，而不是個別的網路介面。 如果子網路內的 VM 需要不同的安全性規則，您可以使網路介面成為應用程式安全性群組 (ASG) 的成員，並將 ASG 指定為安全性規則的來源和目的地。 深入了解[應用程式安全性群組](security-overview.md#application-security-groups)。

如果仍有通訊問題，請參閱[考量](#considerations)和[其他診斷](#additional-dignosis)。

## <a name="considerations"></a>考量

對連線問題進行疑難排解時，請考量下列幾點︰

* 預設的安全性規則會封鎖來自網際網路的輸入存取，並且只允許來自虛擬網路的輸入流量。 若要允許來自網際網路的輸入流量，請新增優先順序高於預設規則的安全性規則。 深入了解[預設安全性規則](security-overview.md#default-security-rules)，或如何[新增安全性規則](manage-network-security-group.md#create-a-security-rule)。
* 如果您有對等互連的虛擬網路，**VIRTUAL_NETWORK** 服務標籤預設會自動展開以包含適用於對等互連虛擬網路的前置詞。 若要對任何與虛擬網路對等互連相關的問題進行疑難排解，您可以在 **ExpandedAddressPrefix** 清單中檢視前置詞。 深入了解[虛擬網路對等互連](virtual-network-peering-overview.md)和[服務標籤](security-overview.md#service-tags)。
* 有效安全性規則只會在有 NSG 已關聯至 VM 的網路介面和/或子網路，且 VM 處於執行中狀態的情況下顯示。
* 如果沒有 NSG 關聯至網路介面或子網路，而且您將[公用 IP 位址](virtual-network-public-ip-address.md)指派給 VM，則系統會開啟所有連接埠，並可從任何地方對它進行輸入和輸出存取。 如果 VM 具有公用 IP 位址，我們建議將 NSG 套用至網路介面所在的子網路。

## <a name="additional-diagnosis"></a>其他診斷

* 若要執行快速測試來判斷系統是否允許流量進出 VM，請使用 Azure 網路監看員的 [IP 流程驗證](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md)功能。 IP 流程驗證會告訴您流量是否被允許或拒絕。 如果被拒絕，IP 流程驗證會告訴您流量是被哪一個安全性規則拒絕。
* 如果沒有任何安全性規則導致 VM 的網路連線能力失敗，則問題可能來自於：
  * VM 作業系統內執行的防火牆軟體
  * 為虛擬設備或內部部署流量設定的路由。 網際網路流量可以透過[強制通道](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)重新導向至您的內部部署網路。 如果您透過強制通道將網際網路流量重新導向至虛擬設備或內部部署，便可能會無法從網際網路連線至 VM。 若要了解如何診斷可能會妨礙從 VM 流出流量的路由問題，請參閱[診斷虛擬機器網路流量路由問題](diagnose-network-routing-problem.md)。

## <a name="next-steps"></a>後續步驟

- 了解適用於[網路安全性群組](manage-network-security-group.md#work-with-network-security-groups)和[安全性規則](manage-network-security-group.md#work-with-security-rules)的所有工作、屬性和設定。
- 了解適用於 VM 的[預設安全性規則](security-overview.md#default-security-rules)、[服務標籤](security-overview.md#service-tags)，以及 [Azure 如何處理輸入和輸出流量的安全性規則](security-overview.md#network-security-groups)。
