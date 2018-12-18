---
title: 診斷虛擬機器網路流量篩選問題 - 快速入門 - Azure CLI | Microsoft Docs
description: 在本快速入門中，您將了解如何使用 Azure 網路監看員的 IP 流量驗證功能來診斷虛擬機器網路流量篩選問題。
services: network-watcher
documentationcenter: network-watcher
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 4be1fac519ee0a7bcd61bd4cced4d829c275679d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990105"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-cli"></a>快速入門：診斷虛擬機器網路流量篩選問題 - Azure CLI

本快速入門中，您可部署虛擬機器 (VM)，然後檢查送至 IP 位址和 URL 以及來自 IP 位址的通訊。 您可判斷通訊失敗的原因及其解決方式。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找已安裝的版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 驗證 CLI 版本之後，請執行 `az login` 以建立與 Azure 的連線。 本快速入門的 CLI 命令會經過格式化以在 Bash 殼層中執行。

## <a name="create-a-vm"></a>建立 VM

您必須先建立資源群組來包含 VM，才能建立 VM。 使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。 下列範例會建立名為 myVm 的 VM：

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 在建立 VM 且 CLI 傳回輸出之前，請勿繼續進行其餘步驟。

## <a name="test-network-communication"></a>測試網路通訊

若要測試網路監看員的網路通訊，您必須先在欲測試的 VM 所在地區啟用網路監看員，然後使用網路監看員的 IP 流量驗證功能來測試通訊。

### <a name="enable-network-watcher"></a>啟用網路監看員

如果您已在美國東部地區啟用網路監看員，請跳至[使用 IP 流程驗證](#use-ip-flow-verify)。 使用 [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) 命令在美國東部地區建立網路監看員：

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-ip-flow-verify"></a>使用 IP 流量驗證

根據預設，當您建立 VM 時，Azure 會允許及拒絕 VM 的雙向網路流量。 您稍後可能會覆寫 Azure 的預設值，並允許或拒絕其他類型的流量。 若要測試允許或拒絕送至不同目的地的流量和來自來源 IP 位址的流量，請使用 [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow) 命令。

測試從 VM 輸出至 www.bing.com 的其中一個 IP 位址的通訊：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

幾秒之後，傳回的結果會通知您，名為 **AllowInternetOutbound** 的安全性規則允許存取。

測試從 VM 輸出至 172.31.0.100 的通訊：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

傳回的結果會通知您，名為 **DefaultOutboundDenyAll** 的安全性規則拒絕存取。

測試從 172.31.0.100 輸入至 VM 的通訊：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

傳回的結果會通知您，因為名為 **DefaultInboundDenyAll** 的安全性規則而拒絕存取。 既然您已知道哪些安全性規則允許或拒絕 VM 的流量，即可判斷如何解決問題。

## <a name="view-details-of-a-security-rule"></a>檢視安全性規則的詳細資料

若要判斷為何[使用 IP 流量驗證](#use-ip-flow-verify)中的規則允許或禁止通訊，請使用 [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) 命令檢閱網路介面的有效安全性規則：

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

對於在[使用 IP 流量驗證](#use-ip-flow-verify) 底下的先前步驟中允許對外存取 www.bing.com 的 **AllowInternetOutbound** 規則，傳回的輸出包含下列文字：

```azurecli
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

您可以在先前的輸出中看到 **destinationAddressPrefix** 為 **Internet**。 雖然並不清楚 13.107.21.200 與 **Internet** 有何關連。 您會看到有幾個位址首碼列在 **expandedDestinationAddressPrefix** 底下。 清單中有一個前置詞是 **12.0.0.0/6**，其中包含 IP 位址的 12.0.0.1-15.255.255.254 範圍。 因為 13.107.21.200 在該位址範圍內，所以 **AllowInternetOutBound** 規則允許輸出流量。 此外，先前的輸出中並未顯示任何會覆寫此規則的較高優先順序 (數字較小) 規則。 若要拒絕輸出至某個 IP 位址的通訊，您可以新增具有較高優先順序的安全性規則，以拒絕連接埠 80 輸出至此 IP 位址。

當您在[使用 IP 流量驗證](#use-ip-flow-verify)中執行 `az network watcher test-ip-flow` 命令來測試輸出至 172.131.0.100 的通訊時，輸出會告知您 **DefaultOutboundDenyAll** 規則拒絕通訊。 **DefaultOutboundDenyAll** 規則等同於 `az network nic list-effective-nsg` 命令的下列輸出中所列的 **DenyAllOutBound** 規則：

```azurecli
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

此規則列出 **0.0.0.0/0** 作為 **destinationAddressPrefix**。 此規則會拒絕輸出至 172.131.0.100 的通訊，因為此位址不在 `az network nic list-effective-nsg` 命令輸出中任何其他輸出規則的 **destinationAddressPrefix** 內。 若要允許輸出通訊，您可以新增較高優先順序的安全性規則，以允許輸出至位於 172.131.0.100 之通訊埠 80 的流量。

當您在[使用 IP 流量驗證](#use-ip-flow-verify)中執行 `az network watcher test-ip-flow` 命令來測試從 172.131.0.100 輸入的通訊時，輸出會告知您 **DefaultInboundDenyAll** 規則拒絕通訊。 **DefaultInboundDenyAll** 規則等同於 `az network nic list-effective-nsg` 命令的下列輸出中所列的 **DenyAllInBound** 規則：

```azurecli
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

如輸出所示，系統會套用 **DenyAllInBound** 規則，因為 `az network nic list-effective-nsg` 命令的輸出中不存在其他較高優先順序的規則，可允許連接埠 80 從 172.131.0.100 輸入至 VM。 若要允許輸入通訊，您可以新增較高優先順序的安全性規則，以允許通訊埠 80 從 172.131.0.100 輸入。

本快速入門中的檢查已測試 Azure 組態。 如果檢查傳回預期的結果，但仍有網路問題，請確定您的 VM 與您正在通訊的端點之間沒有防火牆，而且您的 VM 中的作業系統沒有允許或拒絕通訊的防火牆。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [az group delete](/cli/azure/group#az_group_delete) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立 VM 並診斷出輸入和輸出網路流量篩選條件。 您已了解網路安全性群組規則允許或拒絕 VM 的雙向流量。 深入了解[安全性規則](../virtual-network/security-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)以及如何[建立安全性規則](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule)。

即使採用適當的網路流量篩選條件，VM 的通訊仍可能因為路由組態而失敗。 若要了解如何診斷 VM 網路路由問題，請參閱[診斷 VM 路由問題](diagnose-vm-network-routing-problem-cli.md)，或若要透過一項工具來診斷輸出路由、延遲和流量篩選問題，請參閱[連線疑難排解](network-watcher-connectivity-cli.md)。