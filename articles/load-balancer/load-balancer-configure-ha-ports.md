---
title: 設定 Azure Load Balancer 的高可用性連接埠| Microsoft Docs
description: 了解如何使用高可用性連接埠來對所有連接埠上的內部流量進行負載平衡
services: load-balancer
documentationcenter: na
author: rdhillon
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/21/2018
ms.author: kumud
ms.openlocfilehash: 9661722c5d35e4336d5e42374a1444cf50734fba
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998333"
---
# <a name="configure-high-availability-ports-for-an-internal-load-balancer"></a>設定內部負載平衡器的高可用性連接埠

本文提供如何在內部負載平衡器上部署高可用性連接埠的範例。 如需有關網路虛擬裝置 (NVA) 特有組態的詳細資訊，請參閱對應的提供者網站。

>[!NOTE]
>Azure Load Balancer 支援兩種不同類型：基本和標準。 本文討論標準 Load Balancer。 如需基本 Load Balancer 的詳細資訊，請參閱 [Load Balancer 概觀](load-balancer-overview.md)。

此圖表說明本文所述部署範例的下列組態：

- NVA 部署在高可用性連接埠組態背後之內部負載平衡器的後端集區中。 
- 套用在 DMZ 子網路的使用者定義路由 (UDR) 會將下一個躍點設為內部負載平衡器的虛擬 IP，以將所有流量路由傳送至 NVA。 
- 內部負載平衡器會根據負載平衡器演算法，將流量分散到其中一個作用中的 NVA。
- NVA 處理流量，並將流量轉送到後端子網路中的原始目的地。
- 如果在後端子網路設定了對應的 UDR，則傳回路徑則可採用相同的路由。 

![高可用性連接埠範例部署](./media/load-balancer-configure-ha-ports/haports.png)



## <a name="configure-high-availability-ports"></a>設定高可用性連接埠

若要設定高可用性連接埠，請在後端集區中設定具有 NVA 的內部負載平衡器。 設定對應的負載平衡器健全狀況探查設定，以偵測高可用性連接埠的 NVA 健全狀況和負載平衡器規則。 [開始使用](load-balancer-get-started-ilb-arm-portal.md)中會說明一般負載平衡器的相關組態。 本文主要在說明高可用性連接埠組態。

設定過程基本上涉及將前端連接埠和後端連接埠值設定為 **0**。 將通訊協定值設定為 **All**。 此文章說明如何使用 Azure 入口網站、PowerShell 與 Azure CLI 來設定高可用性連接埠。

### <a name="configure-a-high-availability-ports-load-balancer-rule-with-the-azure-portal"></a>使用 Azure 入口網站設定高可用性連接埠負載平衡器規則

若要使用 Azure 入口網站來設定高可用性連接埠，請選取 [HA 連接埠] 核取方塊。 選取此選項時，系統會自動填入相關的連接埠和通訊協定組態。 

![透過 Azure 入口網站的高可用性連接埠設定](./media/load-balancer-configure-ha-ports/haports-portal.png)


### <a name="configure-a-high-availability-ports-load-balancing-rule-via-the-resource-manager-template"></a>透過 Resource Manager 範本設定高可用性連接埠負載平衡器規則

您可以使用適用於 Load Balancer 資源中之 Microsoft.Network/loadBalancers 的 2017-08-01 API 版本，來設定高可用性連接埠。 下列 JSON 程式碼片段示範透過 REST API 來變更高可用性連接埠的負載平衡器設定：

```json
    {
        "apiVersion": "2017-08-01",
        "type": "Microsoft.Network/loadBalancers",
        ...
        "sku":
        {
            "name": "Standard"
        },
        ...
        "properties": {
            "frontendIpConfigurations": [...],
            "backendAddressPools": [...],
            "probes": [...],
            "loadBalancingRules": [
             {
                "properties": {
                    ...
                    "protocol": "All",
                    "frontendPort": 0,
                    "backendPort": 0
                }
             }
            ],
       ...
       }
    }
```

### <a name="configure-a-high-availability-ports-load-balancer-rule-with-powershell"></a>使用 PowerShell 設定高可用性連接埠負載平衡器規則

在使用 PowerShell 建立內部負載平衡器時，透過下列命令來建立高可用性連接埠負載平衡器規則：

```powershell
lbrule = New-AzureRmLoadBalancerRuleConfig -Name "HAPortsRule" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "All" -FrontendPort 0 -BackendPort 0
```

### <a name="configure-a-high-availability-ports-load-balancer-rule-with-azure-cli"></a>使用 Azure CLI 設定高可用性連接埠負載平衡器規則

在[建立內部負載平衡器集合](load-balancer-get-started-ilb-arm-cli.md)的步驟 4 中，使用下列命令來建立高可用性連接埠負載平衡器規則：

```azurecli
azure network lb rule create --resource-group contoso-rg --lb-name contoso-ilb --name haportsrule --protocol all --frontend-port 0 --backend-port 0 --frontend-ip-name feilb --backend-address-pool-name beilb
```

## <a name="next-steps"></a>後續步驟

深入了解[高可用性連接埠](load-balancer-ha-ports-overview.md)。
