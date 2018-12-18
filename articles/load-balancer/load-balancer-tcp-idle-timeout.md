---
title: 設定負載平衡器 TCP 閒置逾時 | Microsoft Docs
description: 設定負載平衡器 TCP 閒置逾時
services: load-balancer
documentationcenter: na
author: kumudd
manager: timlt
ms.assetid: 4625c6a8-5725-47ce-81db-4fa3bd055891
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: f19ac77f7c7f7d4ab8909d628f9dcce08c07c928
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23020903"
---
# <a name="configure-tcp-idle-timeout-settings-for-azure-load-balancer"></a>設定 Azure Load Balancer 的 TCP 閒置逾時設定

[!INCLUDE [load-balancer-basic-sku-include.md](../../includes/load-balancer-basic-sku-include.md)]

在預設組態中，Azure Load Balancer 的閒置逾時設定是 4 分鐘。 如果閒置期間超過逾時值，即無法保證仍能維持用戶端與雲端服務之間的 TCP 或 HTTP 工作階段。

當連線關閉時，用戶端應用程式可能會收到以下錯誤訊息：「基礎連線已關閉：應該保持運作的連接卻被伺服器關閉。」

常見作法是使用 TCP Keep-Alive。 此作法可讓連線保持長時間連線。 如需詳細資訊，請參閱這些 [.NET 文章](https://msdn.microsoft.com/library/system.net.servicepoint.settcpkeepalive.aspx)。 啟用 Keep-Alive 之後，就會在連線無活動期間傳送封包。 這些 Keep-Alive 封包可確保永遠不會達到閒置逾時值，因此可以長期維持連線。

此設定僅適用於輸入連線。 若要避免連線中斷，您必須使用比閒置逾時設定短的間隔來設定 TCP Keep-Alive，或增加閒置逾時值。 為了支援這類情況，我們新增了可設定閒置逾時的支援。 您現在可以設定 4 至 30 分鐘的持續時間。

TCP Keep-Alive 非常適合用於電池使用時間不受約束的情況。 不建議用於行動應用程式。 在行動裝置應用程式中使用 TCP Keep-Alive 可能會更快耗盡裝置電池電力。

![TCP 逾時](./media/load-balancer-tcp-idle-timeout/image1.png)

下列各節說明如何在虛擬機器和雲端服務中變更閒置逾時設定。

## <a name="configure-the-tcp-timeout-for-your-instance-level-public-ip-to-15-minutes"></a>將執行個體層級公用 IP 的 TCP 逾時值設定為 15 分鐘

```powershell
Set-AzurePublicIP -PublicIPName webip -VM MyVM -IdleTimeoutInMinutes 15
```

`IdleTimeoutInMinutes` 為選擇性。 若未設定，則預設的逾時為 4 分鐘。 可接受的逾時範圍為 4 到 30 分鐘。

## <a name="set-the-idle-timeout-when-creating-an-azure-endpoint-on-a-virtual-machine"></a>在虛擬機器上建立 Azure 端點時設定閒置逾時

若要變更端點的逾時設定，請使用以下方法：

```powershell
Get-AzureVM -ServiceName "mySvc" -Name "MyVM1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 8080 -IdleTimeoutInMinutes 15| Update-AzureVM
```

若要抓取閒置逾時組態，請使用以下命令：

    PS C:\> Get-AzureVM -ServiceName "MyService" -Name "MyVM" | Get-AzureEndpoint
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

## <a name="set-the-tcp-timeout-on-a-load-balanced-endpoint-set"></a>在負載平衡端點集上設定 TCP 逾時

如果端點是負載平衡端點集的一部分，就必須在負載平衡端點集上設定 TCP 逾時。 例如︰

```powershell
Set-AzureLoadBalancedEndpoint -ServiceName "MyService" -LBSetName "LBSet1" -Protocol tcp -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 -IdleTimeoutInMinutes 15
```

## <a name="change-timeout-settings-for-cloud-services"></a>變更雲端服務的逾時設定

您可以使用 Azure SDK 來更新雲端服務。 您需在 .csdef 檔案中進行雲端服務的端點設定。 若要更新雲端服務部署的 TCP 逾時，就必須進行部署升級。 如果只針對公用 IP 指定 TCP 逾時，則為例外狀況。 公用 IP 設定是在 .cscfg 中，而您可以透過更新和升級部署來更新這些設定。

端點設定的 .csdef 變更如下：

```xml
<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" idleTimeoutInMinutes="tcp-timeout" />
    </Endpoints>
</WorkerRole>
```

公用 IP 上逾時設定的 .cscfg 變更如下：

```xml
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

## <a name="rest-api-example"></a>REST API 範例

您可以使用服務管理 API 來設定 TCP 閒置逾時。 請確定 `x-ms-version` 標頭已設定為 `2014-06-01` 或更新版本。 在部署中的所有虛擬機器上，更新指定負載平衡輸入端點的組態。

### <a name="request"></a>要求

    POST https://management.core.windows.net/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>

### <a name="response"></a>Response

```xml
<LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
    <InputEndpoint>
    <LoadBalancedEndpointSetName>endpoint-set-name</LoadBalancedEndpointSetName>
    <LocalPort>local-port-number</LocalPort>
    <Port>external-port-number</Port>
    <LoadBalancerProbe>
        <Path>path-of-probe</Path>
        <Port>port-assigned-to-probe</Port>
        <Protocol>probe-protocol</Protocol>
        <IntervalInSeconds>interval-of-probe</IntervalInSeconds>
        <TimeoutInSeconds>timeout-for-probe</TimeoutInSeconds>
    </LoadBalancerProbe>
    <LoadBalancerName>name-of-internal-loadbalancer</LoadBalancerName>
    <Protocol>endpoint-protocol</Protocol>
    <IdleTimeoutInMinutes>15</IdleTimeoutInMinutes>
    <EnableDirectServerReturn>enable-direct-server-return</EnableDirectServerReturn>
    <EndpointACL>
        <Rules>
        <Rule>
            <Order>priority-of-the-rule</Order>
            <Action>permit-rule</Action>
            <RemoteSubnet>subnet-of-the-rule</RemoteSubnet>
            <Description>description-of-the-rule</Description>
        </Rule>
        </Rules>
    </EndpointACL>
    </InputEndpoint>
</LoadBalancedEndpointList>
```

## <a name="next-steps"></a>後續步驟

[內部負載平衡器概觀](load-balancer-internal-overview.md)

[開始設定網際網路對向負載平衡器](load-balancer-get-started-internet-arm-ps.md)

[設定負載平衡器分配模式](load-balancer-distribution-mode.md)
