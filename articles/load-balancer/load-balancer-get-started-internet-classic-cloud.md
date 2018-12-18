---
title: 建立適用於 Azure 雲端服務的網際網路對向負載平衡器 | Microsoft Docs
description: 了解如何針對雲端服務在傳統部署模型中建立網際網路面向的負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
tags: azure-service-management
ms.assetid: 0bb16f96-56a6-429f-88f5-0de2d0136756
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2017
ms.author: genli
ms.openlocfilehash: b292739f5f4b7184a24ac9b14d40ed98d4921bde
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34067481"
---
# <a name="get-started-creating-an-internet-facing-load-balancer-for-cloud-services"></a>開始為雲端服務建立網際網路面向的負載平衡器

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-internet-classic-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-internet-classic-cli.md)
> * [Azure 雲端服務](../load-balancer/load-balancer-get-started-internet-classic-cloud.md)

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

> [!IMPORTANT]
> 使用 Azure 資源之前，請務必了解 Azure 目前有 Azure Resource Manager 和「傳統」兩種部署模型。 在使用任何 Azure 資源之前，請先確認您了解 [部署模型和工具](../azure-classic-rm.md) 。 您可以按一下本文頂端的索引標籤，檢視不同工具的文件。 本文涵蓋之內容包括傳統部署模型。 您也可以 [了解如何使用 Azure 資源管理員建立網際網路面向的負載平衡器](load-balancer-get-started-internet-arm-ps.md)。

雲端服務是使用負載平衡器自動設定，並可透過服務模型加以自訂。

## <a name="create-a-load-balancer-using-the-service-definition-file"></a>使用服務定義檔案建立負載平衡器

您可以利用 Azure SDK for .NET 2.5 來更新雲端服務。 雲端服務的端點設定設於 [service definition](https://msdn.microsoft.com/library/azure/gg557553.aspx).csdef 檔案中。

下列範例示範如何設定雲端部署的 servicedefinition.csdef 檔案：

檢查雲端部署所產生之 .csdef 檔的片段，您可以看到已設定為在連接埠 10000、10001 和 10002 上使用連接埠 HTTP 的外部端點。

```xml
<ServiceDefinition name="Tenant">
    <WorkerRole name="FERole" vmsize="Small">
        <Endpoints>
            <InputEndpoint name="FE_External_Http" protocol="http" port="10000" />
            <InputEndpoint name="FE_External_Tcp"  protocol="tcp"  port="10001" />
            <InputEndpoint name="FE_External_Udp"  protocol="udp"  port="10002" />

            <InputEndpoint name="HTTP_Probe" protocol="http" port="80" loadBalancerProbe="MyProbe" />

            <InstanceInputEndpoint name="InstanceEP" protocol="tcp" localPort="80">
                <AllocatePublicPortFrom>
                    <FixedPortRange min="10110" max="10120"  />
                </AllocatePublicPortFrom>
            </InstanceInputEndpoint>
            <InternalEndpoint name="FE_InternalEP_Tcp" protocol="tcp" />
        </Endpoints>
    </WorkerRole>
</ServiceDefinition>
```

## <a name="check-load-balancer-health-status-for-cloud-services"></a>檢查雲端服務的負載平衡器健全狀態

以下是健全狀態探查的範例：

```xml
<LoadBalancerProbes>
    <LoadBalancerProbe name="MyProbe" protocol="http" path="Probe.aspx" intervalInSeconds="5" timeoutInSeconds="100" />
</LoadBalancerProbes>
```

負載平衡器會合併端點的資訊和探查的資訊，以建立形式為 `http://{DIP of VM}:80/Probe.aspx` 的 URL，該 URL 可用來查詢服務的健康狀態。

該服務偵測到來自相同 IP 位址的定期探查。 這是來自虛擬機器執行所在之節點主機的健全狀況探查要求。 服務必須以負載平衡器的 HTTP 200 狀態碼回應，假設服務的狀況良好。 任何其他 HTTP 狀態碼 (例如 503) 都會直接讓虛擬機器脫離循環。

探查定義也會控制探查的頻率。 在上述案例中，負載平衡器每隔 5 秒探查端點一次。 如果長達 10 秒 (兩個探查間隔) 未收到正面回應，則會認為探查已關閉，而虛擬機器會脫離循環。 同樣地，如果服務已脫離循環並收到正面回應，則會立即將服務放回循環。 如果服務在狀況良好和狀況不良之間變動，則負載平衡器可以決定延遲將循環重新帶回服務，直到有探查數次的狀況皆為良好為止。

如需詳細資訊，請查看服務定義結構描述中的 [健全狀況探查](https://msdn.microsoft.com/library/azure/jj151530.aspx) 。

## <a name="next-steps"></a>後續步驟

[開始設定內部負載平衡器](load-balancer-get-started-ilb-arm-ps.md)

[設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)

