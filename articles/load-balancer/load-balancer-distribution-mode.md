---
title: 設定 Azure Load Balancer 分配模式 | Microsoft Docs
description: 如何設定 Azure Load Balancer 分配模式來支援來源 IP 同質性。
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
ms.assetid: 7df27a4d-67a8-47d6-b73e-32c0c6206e6e
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: ae793bad9cef86158418eb87e0c38ee0370a6bd2
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30176970"
---
# <a name="configure-the-distribution-mode-for-azure-load-balancer"></a>設定 Azure Load Balancer 的分配模式

## <a name="hash-based-distribution-mode"></a>雜湊型分配模式

Azure Load Balancer 的預設分配模式是一個 5 Tuple 的雜湊。 此 Tuple 是由來源 IP、來源連接埠、目的地 IP、目的地連接埠及通訊協定類型所組成。 此雜湊可用來將流量對應至可用的伺服器，而演算法則只在傳輸工作階段內提供綁定。 系統會將相同工作階段中的封包，導向至負載平衡端點後的相同資料中心 IP (DIP) 執行個體。 當用戶端從相同的來源 IP 啟動新的工作階段時，來源連接埠便會變更，進而導致流量進入不同的 DIP 端點。

![5 Tuple 雜湊型分配模式](./media/load-balancer-distribution-mode/load-balancer-distribution.png)

## <a name="source-ip-affinity-mode"></a>來源 IP 同質性模式

您也可以使用來源 IP 同質性分配模式來設定 Load Balancer。 這個分配模式也稱為工作階段同質性或用戶端 IP 同質性。 此模式會使用 2 Tuple (來源 IP 和目的地 IP) 或 3 Tuple (來源 IP、目的地 IP 及通訊協定類型) 雜湊將流量對應至可用的伺服器。 藉由使用來源 IP 同質性，從相同用戶端電腦起始的連線就會前往相同的 DIP 端點。

下圖說明一個 2 Tuple 組態。 請注意此 2 Tuple 如何經由負載平衡器前往虛擬機器 1 (VM1)。 然後，VM2 和 VM3 會備份 VM1。

![2 Tuple 工作階段同質性分配模式](./media/load-balancer-distribution-mode/load-balancer-session-affinity.png)

來源 IP 同質性模式可解決 Azure Load Balancer 與「遠端桌面閘道」(RD 閘道) 之間的不相容性。 藉由使用此模式，您將可以在單一雲端服務中建置「RD 閘道」伺服器陣列。

另一個使用案例是媒體上傳。 這會透過 UDP 來上傳資料，但會透過 TCP 來存取控制層：

* 用戶端對負載平衡公用位址起始一個 TCP 工作階段，然後被導向到特定的 DIP。 通道會保持作用中來監視連線健康情況。
* 相同的用戶端電腦對相同的負載平衡公用端點起始一個新的 UDP 工作階段。 該連線會被導向到與先前 TCP 連線相同的 DIP 端點。 媒體上傳既能以高輸送量的方式執行，同時又可透過 TCP.維護控制通道。

> [!NOTE]
> 當藉由移除或新增虛擬機器來變更負載平衡集時，系統會重新計算用戶端要求的分配。 您無法確定來自現有用戶端的新連線最後都會抵達相同的伺服器。 此外，使用來源 IP 同質性分配模式可能會導致流量的分配不相等。 在 Proxy 後方執行的用戶端可能會被視為一個獨特的用戶端應用程式。

## <a name="configure-source-ip-affinity-settings"></a>設定來源 IP 同質性設定

針對虛擬機器，請使用 Azure PowerShell 來變更逾時設定。 將 Azure 端點新增到虛擬機器，然後設定負載平衡器分配模式：

```powershell
Get-AzureVM -ServiceName mySvc -Name MyVM1 | Add-AzureEndpoint -Name HttpIn -Protocol TCP -PublicPort 80 -LocalPort 8080 –LoadBalancerDistribution sourceIP | Update-AzureVM
```

請設定 `LoadBalancerDistribution` 元素的值來達到所需的負載平衡量。 如果是 2 Tuple (來源 IP 和目的地 IP) 負載平衡，請指定 sourceIP。 如果是 3 Tuple (來源 IP、目的地 IP 及通訊協定類型) 負載平衡，請指定 sourceIPProtocol。 如果是預設的 5 Tuple 負載平衡行為，請指定 none。

使用下列設定來擷取端點負載平衡器分配模式組態：

    PS C:\> Get-AzureVM –ServiceName MyService –Name MyVM | Get-AzureEndpoint

    VERBOSE: 6:43:50 PM - Completed Operation: Get Deployment
    LBSetName : MyLoadBalancedSet
    LocalPort : 80
    Name : HTTP
    Port : 80
    Protocol : tcp
    Vip : 65.52.xxx.xxx
    ProbePath :
    ProbePort : 80
    ProbeProtocol : tcp
    ProbeIntervalInSeconds : 15
    ProbeTimeoutInSeconds : 31
    EnableDirectServerReturn : False
    Acl : {}
    InternalLoadBalancerName :
    IdleTimeoutInMinutes : 15
    LoadBalancerDistribution : sourceIP

當 `LoadBalancerDistribution` 元素不存在時，Azure Load Balancer 會使用預設的 5 Tuple 演算法。

### <a name="configure-distribution-mode-on-load-balanced-endpoint-set"></a>在負載平衡端點集上設定分配模式

當端點是負載平衡端點集的一部分時，必須在負載平衡端點集上設定分配模式：

```powershell
Set-AzureLoadBalancedEndpoint -ServiceName MyService -LBSetName LBSet1 -Protocol TCP -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 –LoadBalancerDistribution sourceIP
```

### <a name="configure-distribution-mode-for-cloud-services-endpoints"></a>設定雲端服務端點的分配模式

使用「適用於 .NET 2.5 的 Azure SDK」來更新您的雲端服務。 「雲端服務」的端點設定是在 .csdef 檔案中進行的。 若要更新「雲端服務」部署的負載平衡器分配模式，必須進行部署升級。

適用於端點設定的 .csdef 變更範例如下：

```xml
<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" loadBalancerDistribution="sourceIP" />
    </Endpoints>
</WorkerRole>
<NetworkConfiguration>
    <VirtualNetworkSite name="VNet"/>
    <AddressAssignments>
<InstanceAddress roleName="VMRolePersisted">
    <PublicIPs>
    <PublicIP name="public-ip-name" idleTimeoutInMinutes="timeout-in-minutes"/>
    </PublicIPs>
</InstanceAddress>
    </AddressAssignments>
</NetworkConfiguration>
```

## <a name="api-example"></a>API 範例

下列範例說明如何針對某個部署中的指定負載平衡集，重新設定負載平衡器分配模式。 

### <a name="change-distribution-mode-for-deployed-load-balanced-set"></a>變更已部署之負載平衡集的分配模式

使用 Azure 傳統部署模型來變更現有的部署組態。 新增 `x-ms-version` 標頭，然後將值設定成 2014-09-01 版或更新版本。

#### <a name="request"></a>要求

    POST https://management.core.windows.net/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>?comp=UpdateLbSet   x-ms-version: 2014-09-01
    Content-Type: application/xml

    <LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
      <InputEndpoint>
        <LoadBalancedEndpointSetName> endpoint-set-name </LoadBalancedEndpointSetName>
        <LocalPort> local-port-number </LocalPort>
        <Port> external-port-number </Port>
        <LoadBalancerProbe>
          <Port> port-assigned-to-probe </Port>
          <Protocol> probe-protocol </Protocol>
          <IntervalInSeconds> interval-of-probe </IntervalInSeconds>
          <TimeoutInSeconds> timeout-for-probe </TimeoutInSeconds>
        </LoadBalancerProbe>
        <Protocol> endpoint-protocol </Protocol>
        <EnableDirectServerReturn> enable-direct-server-return </EnableDirectServerReturn>
        <IdleTimeoutInMinutes>idle-time-out</IdleTimeoutInMinutes>
        <LoadBalancerDistribution>sourceIP</LoadBalancerDistribution>
      </InputEndpoint>
    </LoadBalancedEndpointList>

如先前所述，針對 2 Tuple 同質性，請將 `LoadBalancerDistribution` 元素設定成 sourceIP，針對 3 Tuple 同質性，請設定成 sourceIPProtocol，或針對無同質性 (5 Tuple 同質性)，則設定成 none。

#### <a name="response"></a>Response

    HTTP/1.1 202 Accepted
    Cache-Control: no-cache
    Content-Length: 0
    Server: 1.0.6198.146 (rd_rdfe_stable.141015-1306) Microsoft-HTTPAPI/2.0
    x-ms-servedbyregion: ussouth2
    x-ms-request-id: 9c7bda3e67c621a6b57096323069f7af
    Date: Thu, 16 Oct 2014 22:49:21 GMT

## <a name="next-steps"></a>後續步驟

* [Azure 內部負載平衡器概觀](load-balancer-internal-overview.md)
* [開始設定網際網路對應負載平衡器](load-balancer-get-started-internet-arm-ps.md)
* [設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)
