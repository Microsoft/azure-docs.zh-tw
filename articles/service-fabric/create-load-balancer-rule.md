---
title: 建立叢集的 Azure Load Balancer 規則
description: 設定 Azure Load Balancer 來開啟 Azure Service Fabric 叢集的連接埠。
services: service-fabric
documentationcenter: na
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/06/2017
ms.author: ryanwi
ms.openlocfilehash: e4c3bf627c4a5e01c4d9001fcbb0feed0b92209f
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008037"
---
# <a name="open-ports-for-a-service-fabric-cluster"></a>開啟 Service Fabric 叢集的連接埠

此負載平衡器會隨 Azure Service Fabric 叢集一起部署，以將流量導向至節點上所執行的應用程式。 如果您變更應用程式以使用不同的連接埠，您必須在 Azure Load Balancer 中公開該連接埠 (或路由不同的連接埠)。

當您將 Service Fabric 叢集部署至 Azure 時，系統會自動為您建立負載平衡器。 如果您沒有負載平衡器，請參閱[設定網際網路面向的負載平衡器](..\load-balancer\load-balancer-get-started-internet-portal.md)。

## <a name="configure-service-fabric"></a>設定 Service Fabric

您的 Service Fabric 應用程式 **ServiceManifest.xml** 設定檔會定義應用程式預期使用的端點。 在更新設定檔以定義端點之後，必須更新負載平衡器以公開該連接埠 (或不同的連接埠)。 如需如何建立 Service Fabric 端點的詳細資訊，請參閱[設定端點](service-fabric-service-manifest-resources.md)。

## <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則會開啟網際網路面向的連接埠，並將流量轉送至您的應用程式所使用的內部節點連接埠。 如果您沒有負載平衡器，請參閱[設定網際網路面向的負載平衡器](..\load-balancer\load-balancer-get-started-internet-portal.md)。

若要建立負載平衡器規則，您需要收集下列資訊：

- 負載平衡器名稱。
- 負載平衡器和 Service Fabric 叢集的資源群組。
- 外部連接埠。
- 內部連接埠。

## <a name="azure-cli"></a>Azure CLI
使用 **Azure CLI**，只要一個命令就能建立負載平衡器規則。 您只需要知道建立新規則之負載平衡器和資源群組的名稱。

>[!NOTE]
>如果您需要決定負載平衡器的名稱，請使用此命令，以便快速取得一份所有負載平衡器和相關聯資源群組的清單。
>
>`az network lb list --query "[].{ResourceGroup: resourceGroup, Name: name}"`
>


```azurecli
az network lb rule create --backend-port 40000 --frontend-port 39999 --protocol Tcp --lb-name LB-svcfab3 -g svcfab_cli -n my-app-rule
```

Azure CLI 命令有幾個參數，如下表所述：

| 參數 | 說明 |
| --------- | ----------- |
| `--backend-port`  | Service Fabric 應用程式正在接聽的連接埠。 |
| `--frontend-port` | 負載平衡器公開給外部連線的連接埠。 |
| `-lb-name` | 所要變更的負載平衡器名稱。 |
| `-g`       | 包含負載平衡器和 Service Fabric 叢集的資源群組。 |
| `-n`       | 需要的規則名稱。 |


>[!NOTE]
>如需如何使用 Azure CLI 建立負載平衡器的詳細資訊，請參閱[使用 Azure CLI 建立負載平衡器](..\load-balancer\load-balancer-get-started-internet-arm-cli.md)。

## <a name="powershell"></a>PowerShell

PowerShell 比 Azure CLI 稍微複雜一點。 請遵循這些概念步驟來建立規則：

1. 從 Azure 取得負載平衡器。
2. 建立規則。
3. 將規則新增至負載平衡器。
4. 更新負載平衡器。

>[!NOTE]
>如果您需要決定負載平衡器的名稱，請使用此命令，以便快速取得一份所有負載平衡器和相關聯資源群組的清單。
>
>`Get-AzureRmLoadBalancer | Select Name, ResourceGroupName`

```powershell
# Get the load balancer
$lb = Get-AzureRmLoadBalancer -Name LB-svcfab3 -ResourceGroupName svcfab_cli

# Create the rule based on information from the load balancer.
$lbrule = New-AzureRmLoadBalancerRuleConfig -Name my-app-rule7 -Protocol Tcp -FrontendPort 39990 -BackendPort 40009 `
                                            -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
                                            -BackendAddressPool  $lb.BackendAddressPools[0] `
                                            -Probe $lb.Probes[0]

# Add the rule to the load balancer
$lb.LoadBalancingRules.Add($lbrule)

# Update the load balancer on Azure
$lb | Set-AzureRmLoadBalancer
```

至於 `New-AzureRmLoadBalancerRuleConfig` 命令，`-FrontendPort` 代表負載平衡器公開給外部連線的連接埠，而 `-BackendPort` 代表 Service Fabric 應用程式正在接聽的連接埠。

>[!NOTE]
>如需如何使用 PowerShell 建立負載平衡器的詳細資訊，請參閱[使用 PowerShell 建立負載平衡器](..\load-balancer\load-balancer-get-started-internet-arm-ps.md)。

## <a name="next-steps"></a>後續步驟

深入了解 [Service Fabric 中的網路功能](service-fabric-patterns-networking.md)。