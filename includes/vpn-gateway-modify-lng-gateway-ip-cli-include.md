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
ms.openlocfilehash: 865f4a963dfbaa7b8392865f8b0c468734673ba0
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197630"
---
### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>修改區域網路閘道 'gatewayIpAddress'

如果您想要連線的 VPN 裝置已變更其公用 IP 位址，您需要修改區域網路閘道，以反映該變更。 您可變更閘道 IP 位址，而不需移除現有的 VPN 閘道連線 (如果有的話)。 若要修改閘道 IP 位址，使用 [az network local-gateway update](https://docs.microsoft.com/cli/azure/network/local-gateway#az_network_local_gateway_update) 命令以您自己的值取代 'Site2' 和 'TestRG1'。

```azurecli
az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1
```

確認輸出中的 IP 位址正確無誤︰

```
"gatewayIpAddress": "23.99.222.170",
```