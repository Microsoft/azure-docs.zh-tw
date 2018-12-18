---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e021a1b109f735dee16d75c05c26ab35e599a3d9
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197698"
---
### <a name="to-view-local-network-gateways"></a>檢視區域網路閘道

若要檢視區域網路閘道清單，請使用 [az network local-gateway list](https://docs.microsoft.com/cli/azure/network/local-gateway#az_network_local_gateway_list) 命令。

```azurecli
az network local-gateway list --resource-group TestRG1
```

[!INCLUDE [modify-prefix](vpn-gateway-modify-ip-prefix-cli-include.md)]

[!INCLUDE [modify-gateway-IP](vpn-gateway-modify-lng-gateway-ip-cli-include.md)]

### <a name="to-verify-the-shared-key-values"></a>確認共用金鑰值

請確認共用金鑰值和您用於 VPN 裝置組態的值相同。 如果不同，請使用裝置的值再次執行連線，或是以傳回的值將裝置更新。 這兩個值必須相符。 若要檢視共用金鑰，請使用 [az network vpn-connection-list](https://docs.microsoft.com/cli/azure/network/vpn-connection#az_network_vpn_connection_list)。

```azurecli
az network vpn-connection shared-key show --connection-name VNet1toSite2 --resource-group TestRG1
```
### <a name="to-view-the-vpn-gateway-public-ip-address"></a>若要檢視 VPN 閘道公用 IP 位址：

若要尋找虛擬網路閘道的公用 IP 位址，請使用 [az network public-ip list](https://docs.microsoft.com/cli/azure/network/public-ip#az_network_public_ip_list) 命令。 為了方便閱讀，系統會將此範例的輸出格式化為以資料表格式顯示公用 IP 的清單。

```azurecli
az network public-ip list --resource-group TestRG1 --output table
```
