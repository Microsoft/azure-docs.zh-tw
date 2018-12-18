---
title: 建立 Azure 雲端服務的內部負載平衡器 | Microsoft Docs
description: 了解如何在傳統部署模型中使用 PowerShell 建立內部負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
tags: azure-service-management
ms.assetid: 57966056-0f46-4f95-a295-483ca1ad135d
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2017
ms.author: genli
ms.openlocfilehash: 96a7c5b21d78dcc84a32fbfc8835542d64a5d7af
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34067753"
---
# <a name="get-started-creating-an-internal-load-balancer-classic-for-cloud-services"></a>開始為雲端服務建立內部負載平衡器 (傳統)

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-classic-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-ilb-classic-cli.md)
> * [雲端服務](../load-balancer/load-balancer-get-started-ilb-classic-cloud.md)

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何[使用 Resource Manager 模型執行這些步驟](load-balancer-get-started-ilb-arm-ps.md)。

## <a name="configure-internal-load-balancer-for-cloud-services"></a>設定雲端服務的內部負載平衡器

虛擬機器和雲端服務都支援內部負載平衡器。 在區域虛擬網路外的雲端服務中建立的內部負載平衡器端點，將只能在雲端服務內存取。

內部負載平衡器組態必須在於雲端服務中建立第一個部署的期間進行設定，如以下範例所示。

> [!IMPORTANT]
> 執行下列步驟的必要條件是已經為雲端部署建立虛擬網路。 您需要虛擬網路名稱和子網路名才能建立內部負載平衡。

### <a name="step-1"></a>步驟 1

在 Visual Studio 中開啟雲端部署的服務組態檔 (.cscfg) 並新增下列區段，以在網路組態的最後一個 "`</Role>`" 項目下建立內部負載平衡。

```xml
<NetworkConfiguration>
    <LoadBalancers>
    <LoadBalancer name="name of the load balancer">
        <FrontendIPConfiguration type="private" subnet="subnet-name" staticVirtualNetworkIPAddress="static-IP-address"/>
    </LoadBalancer>
    </LoadBalancers>
</NetworkConfiguration>
```

讓我們新增網路組態檔的值，以顯示看起來如何。 在此範例中，假設您利用稱為 test_subnet 的子網路 10.0.0.0/24 和靜態 IP 10.0.0.4 建立稱為 "test_vnet" 的 VNet。 負載平衡器將會命名為 testLB。

```xml
<NetworkConfiguration>
    <LoadBalancers>
    <LoadBalancer name="testLB">
        <FrontendIPConfiguration type="private" subnet="test_subnet" staticVirtualNetworkIPAddress="10.0.0.4"/>
    </LoadBalancer>
    </LoadBalancers>
</NetworkConfiguration>
```

如需關於負載平衡器結構描述的詳細資訊，請參閱 [新增負載平衡器](https://msdn.microsoft.com/library/azure/dn722411.aspx)

### <a name="step-2"></a>步驟 2

變更服務定義 (.csdef) 檔案，以將端點新增至內部負載平衡。 建立角色執行個體時，服務定義檔會將角色執行個體新增至內部負載平衡。

```xml
<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" loadBalancer="load-balancer-name" />
    </Endpoints>
</WorkerRole>
```

遵循與上述範例相同的值，讓我們將值新增至服務定義檔。

```xml
<WorkerRole name="WorkerRole1" vmsize="A7" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="endpoint1" protocol="http" localPort="80" port="80" loadBalancer="testLB" />
    </Endpoints>
</WorkerRole>
```

網路流量會使用 testLB 負載平衡器進行負載平衡，使用連接埠 80 進行連入要求，也在連接埠 80 上傳送背景工作角色執行個體。

## <a name="next-steps"></a>後續步驟

[使用來源 IP 同質性設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)

