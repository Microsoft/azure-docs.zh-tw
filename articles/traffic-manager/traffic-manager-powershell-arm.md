---
title: 使用 PowerShell 管理 Azure 中的流量管理員 | Microsoft Docs
description: 使用 PowerShell 透過 Azure Resource Manager 執行流量管理員
services: traffic-manager
documentationcenter: na
author: kumudd
manager: timlt
ms.assetid: bc247448-1d2e-4104-ac03-42b59ebde065
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: kumud
ms.openlocfilehash: 951e845e23a1ed0cbdc83fc24a97a545f00c52ad
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31526636"
---
# <a name="using-powershell-to-manage-traffic-manager"></a>使用 PowerShell 管理流量管理員

Azure Resource Manager 是 Azure 中慣用的服務管理介面。 您可以使用以 Azure Resource Manager 為基礎的 API 和工具來管理 Azure 流量管理員設定檔。

## <a name="resource-model"></a>資源模型

Azure 流量管理員使用名為「流量管理員設定檔」的設定集合進行設定。 這個設定檔包含 DNS 設定、流量路由設定、端點監視設定，以及流量路由的目標服務端點清單。

每個流量管理員設定檔都以 'TrafficManagerProfiles' 類型的資源表示。 在 REST API 層級中，每個設定檔的 URI 如下：

    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}

## <a name="setting-up-azure-powershell"></a>設定 Azure PowerShell

這些指示使用 Microsoft Azure PowerShell。 下列文章說明如何安裝和設定 Azure PowerShell。

* [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)

這篇文章中的範例假設您有現有的資源群組。 您可以使用下列命令建立資源群組：

```powershell
New-AzureRmResourceGroup -Name MyRG -Location "West US"
```

> [!NOTE]
> Azure Resource Manager 規定所有資源群組都要有位置。 此位置作為該資源群組中建立之資源的預設位置。 然而，因為流量管理員設定檔的資源是全域而非區域，資源群組位置的選擇不會影響 Azure 流量管理員。

## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔

若要建立流量管理員設定檔，請使用 `New-AzureRmTrafficManagerProfile`Cmdlet：

```powershell
$profile = New-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

下表描述參數：

| 參數 | 說明 |
| --- | --- |
| Name |流量管理員設定檔資源的資源名稱。 相同資源群組中的設定檔必須有唯一的名稱。 這個名稱是從用於 DNS 查詢的 DNS 名稱分割而來。 |
| resourceGroupName |包含設定檔資源的資源群組名稱。 |
| TrafficRoutingMethod |指定流量路由方法，用來決定回應 DNS 查詢時傳回哪一個端點。 可能的值為 'Performance'、'Weighted' 或 'Priority'。 |
| RelativeDnsName |指定此流量管理員設定檔所提供之 DNS 名稱的主機名稱部分。 這個值會與由 Azure 流量管理員使用的 DNS 網域名稱結合，形成設定檔的完整網域名稱 (FQDN)。 例如，值設為 'contoso' 會變成 'contoso.trafficmanager.net'。 |
| TTL |指定 DNS 存留時間 (TTL)，以秒為單位。 此 TTL 會通知本機 DNS 解析程式和 DNS 用戶端，以多長時間來快取此流量管理員設定檔的 DNS 回應。 |
| MonitorProtocol |指定用來監視端點健康情況的通訊協定。 可能的值為 'HTTP' 和 'HTTPS'。 |
| MonitorPort |指定用來監視端點健康情況的 TCP 連接埠。 |
| MonitorPath |指定用來探查端點健康情況之端點網域名稱的相對路徑。 |

此 Cmdlet 會在 Azure 中建立 Azure 流量管理員設定檔，並將對應的設定檔物件傳回至 PowerShell。 此時，設定檔不含任何端點。 如需將端點新增至流量管理員設定檔的詳細資訊，請參閱[新增流量管理員端點](#adding-traffic-manager-endpoints)。

## <a name="get-a-traffic-manager-profile"></a>取得流量管理員設定檔

若要擷取現有的流量管理員設定檔物件，請使用 `Get-AzureRmTrafficManagerProfle` Cmdlet：

```powershell
$profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
```

此 Cmdlet 會傳回流量管理員設定檔物件。

## <a name="update-a-traffic-manager-profile"></a>更新流量管理員設定檔

修改流量管理員設定檔需要 3 個步驟︰

1. 使用 `Get-AzureRmTrafficManagerProfile` 擷取設定檔，或使用 `New-AzureRmTrafficManagerProfile` 所傳回的設定檔。
2. 修改設定檔。 您可以新增和移除端點，也可以變更端點或設定檔參數。 這些變更是離線作業。 您只是變更記憶體中代表設定檔的本機物件。
3. 使用 `Set-AzureRmTrafficManagerProfile` Cmdlet 來認可您所做的變更。

設定檔的 RelativeDnsName 除外，其他所有設定檔屬性都可變更。 若要變更 RelativeDnsName，您必須刪除設定檔，再以新名稱建立新的設定檔。

下列範例示範如何變更設定檔的 TTL：

```powershell
$profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
$profile.Ttl = 300
Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile
```

流量管理員端點有三種類型：

1. **Azure 端點**是 Azure 中託管的服務。
2. **外部端點**是 Azure 外部託管的服務。
3. **巢狀端點**用來建構流量管理員設定檔的巢狀階層。 巢狀端點可讓複雜的應用程式使用進階的流量路由設定。

不論是這三種端點中的哪一種，您都可以透過兩種方式來新增端點：

1. 使用先前所述的 3 步驟程序。 這個方法的優點是更新一次就可以完成多項端點變更。
2. 使用 New-AzureRmTrafficManagerEndpoint Cmdlet。 此 Cmdlet 會在單一作業中將端點新增至現有的流量管理員設定檔。

## <a name="adding-azure-endpoints"></a>新增 Azure 端點

Azure 端點會參考 Azure 中託管的服務。 支援兩種 Azure 端點：

1. Azure Web Apps 
2. Azure PublicIpAddress 資源 (可附加至負載平衡器或虛擬機器 NIC)。 PublicIpAddress 必須已獲指派 DNS 名稱，才能在流量管理員中使用。

不論是上述哪一種，都必須注意以下事項：

* 使用 `Add-AzureRmTrafficManagerEndpointConfig` 或 `New-AzureRmTrafficManagerEndpoint` 的 'targetResourceId' 參數指定服務。
* TargetResourceId 隱含 'Target' 和 'EndpointLocation'。
* 您可以選擇是否指定 'Weight'。 只有在設定檔已設定為使用「加權」流量路由方法時，才會使用加權， 否則會予以忽略。 如果指定，則值必須是 1 到 1000 之間的數字。 預設值為 '1'。
* 您可以選擇是否指定 'Priority'。 只有在設定檔已設定為使用「優先順序」流量路由方法時，才會使用優先順序， 否則會予以忽略。 有效值從 1 到 1000，值越小代表優先順序越高。 如果對某個端點指定值，則所有端點也都必須進行指定。 如果省略，則會依端點列出順序，從 '1' 開始套用預設值。

### <a name="example-1-adding-web-app-endpoints-using-add-azurermtrafficmanagerendpointconfig"></a>範例 1︰使用 `Add-AzureRmTrafficManagerEndpointConfig` 新增 Web 應用程式端點

在此範例中，我們使用 `Add-AzureRmTrafficManagerEndpointConfig` Cmdlet 建立流量管理員設定檔，並新增兩個 Web 應用程式端點。

```powershell
$profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$webapp1 = Get-AzureRMWebApp -Name webapp1
Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
$webapp2 = Get-AzureRMWebApp -Name webapp2
Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile
```
### <a name="example-2-adding-a-publicipaddress-endpoint-using-new-azurermtrafficmanagerendpoint"></a>範例 2︰使用 `New-AzureRmTrafficManagerEndpoint` 新增 publicIpAddress 端點

此範例會在流量管理員設定檔中新增公用 IP 位址資源。 公用 IP 位址必須設定 DNS 名稱，而且可以繫結至 VM 的 NIC 或繫結至負載平衡器。

```powershell
$ip = Get-AzureRmPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
New-AzureRmTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="adding-external-endpoints"></a>新增外部端點

流量管理員使用外部端點將流量導向至在 Azure 之外所託管的服務。 如同 Azure 端點一樣，使用 `Add-AzureRmTrafficManagerEndpointConfig` 再接著使用 `Set-AzureRmTrafficManagerProfile`，或使用 `New-AzureRMTrafficManagerEndpoint`，都可以新增外部端點。

指定外部端點時：

* 必須使用 'Target' 參數指定端點網域名稱
* 如果使用「效能」流量路由方法，則需要 'EndpointLocation'， 否則為選擇性。 值必須是[有效的 Azure 區域名稱](https://azure.microsoft.com/regions/)。
* 'Weight' 和 'Priority' 是選擇性。

### <a name="example-1-adding-external-endpoints-using-add-azurermtrafficmanagerendpointconfig-and-set-azurermtrafficmanagerprofile"></a>範例 1︰使用 `Add-AzureRmTrafficManagerEndpointConfig` 和 `Set-AzureRmTrafficManagerProfile` 新增外部端點

在此範例中，我們會建立流量管理員設定檔、新增兩個外部端點，並認可變更。

```powershell
$profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzureRmTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointLocation "North Europe" -EndpointStatus Enabled
Add-AzureRmTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointLocation "Central US" -EndpointStatus Enabled
Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile
```

### <a name="example-2-adding-external-endpoints-using-new-azurermtrafficmanagerendpoint"></a>範例 2︰使用 `New-AzureRmTrafficManagerEndpoint` 新增外部端點

在此範例中，我們將外部端點新增至現有的設定檔。 使用設定檔和資源群組名稱來指定設定檔。

```powershell
New-AzureRmTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
```

## <a name="adding-nested-endpoints"></a>新增「巢狀」端點

每個「流量管理員」設定檔皆指定一個流量路由方法。 不過，有些情況需要的流量路由，比單一流量管理員設定檔所提供的路由更複雜。 您可以將流量管理員設定檔巢狀化，以結合多個流量路由方法的優點。 巢狀設定檔可讓您覆寫預設流量管理員行為，以支援更大和更複雜的應用程式部署。 如需詳細範例，請參閱[巢狀流量管理員設定檔](traffic-manager-nested-profiles.md)。

巢狀端點是在父設定檔中使用特定的端點類型 ('NestedEndpoints') 來設定。 指定巢狀端點時︰

* 必須使用 'targetResourceId' 參數指定端點
* 如果使用「效能」流量路由方法，則需要 'EndpointLocation'， 否則為選擇性。 值必須是[有效的 Azure 區域名稱](http://azure.microsoft.com/regions/)。
* 如同 Azure 端點一樣，'Weight' 和 'Priority' 是選擇性。
* 'MinChildEndpoints' 參數是選擇性。 預設值為 '1'。 如果可用的端點數目低於此臨界值，父設定檔會將子設定檔視為「已降級」，並將流量導向父設定檔中的其他端點。

### <a name="example-1-adding-nested-endpoints-using-add-azurermtrafficmanagerendpointconfig-and-set-azurermtrafficmanagerprofile"></a>範例 1︰使用 `Add-AzureRmTrafficManagerEndpointConfig` 和 `Set-AzureRmTrafficManagerProfile` 新增巢狀端點

在此範例中，我們會建立新的流量管理員子設定檔和父設定檔、將子設定檔新增至父設定檔中做為巢狀端點，並認可變更。

```powershell
$child = New-AzureRmTrafficManagerProfile -Name child -ResourceGroupName MyRG -TrafficRoutingMethod Priority -RelativeDnsName child -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$parent = New-AzureRmTrafficManagerProfile -Name parent -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName parent -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzureRmTrafficManagerEndpointConfig -EndpointName child-endpoint -TrafficManagerProfile $parent -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile
```

為了簡潔起見，在此範例中，我們並未將任何其他端點新增至子設定檔或父設定檔。

### <a name="example-2-adding-nested-endpoints-using-new-azurermtrafficmanagerendpoint"></a>範例 2︰使用 `New-AzureRmTrafficManagerEndpoint` 新增巢狀端點

在此範例中，我們將現有的子設定檔新增至現有的父設定檔中做為巢狀端點。 使用設定檔和資源群組名稱來指定設定檔。

```powershell
$child = Get-AzureRmTrafficManagerEndpoint -Name child -ResourceGroupName MyRG
New-AzureRmTrafficManagerEndpoint -Name child-endpoint -ProfileName parent -ResourceGroupName MyRG -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
```

## <a name="adding-endpoints-from-another-subscription"></a>從另一個訂用帳戶新增端點

流量管理員可以處理不同訂用帳戶的端點。 利用您想要新增的端點來切換至訂用帳戶，以擷取流量管理員所需的輸入。 然後，您需要透過流量管理員 設定檔切換至訂用帳戶，接下來將端點新增至這個帳戶。 下列範例會示範如何透過一個公用 IP 位址來完成此操作。

```powershell
Set-AzureRmContext -SubscriptionId $EndpointSubscription
$ip = Get-AzureRmPublicIpAddress -Name $IpAddresName -ResourceGroupName $EndpointRG

Set-AzureRmContext -SubscriptionId $trafficmanagerSubscription
New-AzureRmTrafficManagerEndpoint -Name $EndpointName -ProfileName $ProfileName -ResourceGroupName $TrafficManagerRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="update-a-traffic-manager-endpoint"></a>更新流量管理員端點

有兩種方式可以更新現有流量管理員端點：

1. 使用 `Get-AzureRmTrafficManagerProfile` 取得流量管理員設定檔、更新設定檔內的端點屬性，並使用 `Set-AzureRmTrafficManagerProfile` 認可變更。 這個方法的優點是能夠在單一作業中更新多個端點。
2. 使用 `Get-AzureRmTrafficManagerEndpoint` 取得流量管理員端點、更新端點屬性，並使用 `Set-AzureRmTrafficManagerEndpoint` 認可變更。 由於不需要在設定檔中編製索引為端點陣列，這個方法會比較簡單。

### <a name="example-1-updating-endpoints-using-get-azurermtrafficmanagerprofile-and-set-azurermtrafficmanagerprofile"></a>範例 1︰使用 `Get-AzureRmTrafficManagerProfile` 和 `Set-AzureRmTrafficManagerProfile` 更新端點

在此範例中，我們會修改現有設定檔中兩個端點的優先順序。

```powershell
$profile = Get-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG
$profile.Endpoints[0].Priority = 2
$profile.Endpoints[1].Priority = 1
Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile
```

### <a name="example-2-updating-an-endpoint-using-get-azurermtrafficmanagerendpoint-and-set-azurermtrafficmanagerendpoint"></a>範例 2︰使用 `Get-AzureRmTrafficManagerEndpoint` 和 `Set-AzureRmTrafficManagerEndpoint` 更新端點

在此範例中，我們會修改現有設定檔中單一端點的加權。

```powershell
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName MyRG -Type ExternalEndpoints
$endpoint.Weight = 20
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

## <a name="enabling-and-disabling-endpoints-and-profiles"></a>啟用和停用端點和設定檔

流量管理員可允許啟用和停用個別端點，以及允許啟用和停用全部設定檔。
透過取得/更新/設定端點或設定檔資源，即可進行這些變更。 為了簡化這些常見作業，系統也支援透過專用 Cmdlet 來執行這些作業。

### <a name="example-1-enabling-and-disabling-a-traffic-manager-profile"></a>範例 1：啟用和停用流量管理員設定檔

若要啟用流量管理員設定檔，請使用 `Enable-AzureRmTrafficManagerProfile`。 您可以使用設定檔物件來指定設定檔。 您可以透過管線或使用 '-TrafficManagerProfile' 參數傳遞設定檔物件。 在此範例中，我們會以設定檔和資源群組名稱來指定設定檔。

```powershell
Enable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

若要停用流量管理員設定檔：

```powershell
Disable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

Disable-AzureRmTrafficManagerProfile Cmdlet 會提示您確認。 使用 '-Force' 參數可以抑制此提示。

### <a name="example-2-enabling-and-disabling-a-traffic-manager-endpoint"></a>範例 2：啟用和停用流量管理員端點

若要啟用流量管理員端點，請使用 `Enable-AzureRmTrafficManagerEndpoint`。 有兩種方式可指定端點

1. 使用透過管線或利用 '-TrafficManagerEndpoint' 參數傳遞的 TrafficManagerEndpoint 物件
2. 使用端點名稱、端點類型、設定檔名稱和資源群組名稱︰

```powershell
Enable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

同樣地，若要停用流量管理員端點：

```powershell
Disable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG -Force
```

如同 `Disable-AzureRmTrafficManagerProfile`，`Disable-AzureRmTrafficManagerEndpoint` Cmdlet 會提示您確認。 使用 '-Force' 參數可以抑制此提示。

## <a name="delete-a-traffic-manager-endpoint"></a>停用流量管理員端點

若要移除個別的端點，請使用 `Remove-AzureRmTrafficManagerEndpoint` Cmdlet︰

```powershell
Remove-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

這個 Cmdlet 會提示您確認。 使用 '-Force' 參數可以抑制此提示。

## <a name="delete-a-traffic-manager-profile"></a>刪除流量管理員設定檔

若要刪除流量管理員設定檔，請使用 `Remove-AzureRmTrafficManagerProfile` Cmdlet，並指定設定檔和資源群組名稱：

```powershell
Remove-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG [-Force]
```

這個 Cmdlet 會提示您確認。 使用 '-Force' 參數可以抑制此提示。

要刪除的設定檔也可以使用設定檔物件來指定：

```powershell
$profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
Remove-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile [-Force]
```

也可輸送以下順序：

```powershell
Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG | Remove-AzureRmTrafficManagerProfile [-Force]
```

## <a name="next-steps"></a>後續步驟

[流量管理員監視](traffic-manager-monitoring.md)

[流量管理員的效能考量](traffic-manager-performance-considerations.md)
