---
title: 使用 Microsoft Azure 流量管理員增加 Language Understanding (LUIS) 中的端點配額
titleSuffix: Azure Cognitive Services
description: Language Understanding (LUIS) 可讓您增加端點要求配額而超過單一金鑰的配額。 如果要這樣做，請在 [發佈] 頁面的 [資源和金鑰] 區段中，為 LUIS 建立多個金鑰，然後將它們加入 LUIS 應用程式。
author: diberry
manager: cgronlun
services: cognitive-services
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 28fc0d0061d1826f0e17c26325ea227e001dccda
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042171"
---
# <a name="use-microsoft-azure-traffic-manager-to-manage-endpoint-quota-across-keys"></a>使用 Microsoft Azure 流量管理員管理幾個金鑰之間的端點配額
Language Understanding (LUIS) 可讓您增加端點要求配額而超過單一金鑰的配額。 如果要這樣做，請在 [發佈] 頁面的 [資源和金鑰] 區段中，為 LUIS 建立多個金鑰，然後將它們加入 LUIS 應用程式。 

用戶端應用程式必須管理這些金鑰之間的流量。 LUIS 不會執行此工作。 

本文說明如何使用 Azure [流量管理員][traffic-manager-marketing]管理這些金鑰之間的流量。 您必須有已定型和發佈的 LUIS 應用程式。 如果還沒有，請遵循預先建置的領域[快速入門](luis-get-started-create-app.md)。 

## <a name="connect-to-powershell-in-the-azure-portal"></a>在 Azure 入口網站中連線到 PowerShell
在 [Azure][azure-portal] 入口網站中，開啟 PowerShell 視窗。 PowerShell 視窗的圖示是上方導覽列中的 **>_**。 從入口網站使用 PowerShell，可以取得最新的 PowerShell 版本，而且您會經過驗證。 入口網站中的 PowerShell 需要 [Azure 儲存體](https://azure.microsoft.com/services/storage/)帳戶。 

![Azure 入口網站中開啟 Powershell 視窗的螢幕擷取畫面](./media/traffic-manager/azure-portal-powershell.png)

以下各節使用[流量管理員 PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/?view=azurermps-6.2.0#traffic_manager)。

## <a name="create-azure-resource-group-with-powershell"></a>使用 PowerShell 建立 Azure 資源群組
建立 Azure 資源之前，請先建立要包含所有資源的資源群組。 將資源群組命名為 `luis-traffic-manager`，並使用 `West US` 的區域。 資源群組的區域會儲存與群組相關的中繼資料。 如果您的資源在另一個區域中，這樣不會降低資源的速度。 

使用 **[New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup?view=azurermps-6.2.0)** Cmdlet 建立資源群組：

```PowerShell
New-AzureRmResourceGroup -Name luis-traffic-manager -Location "West US"
```

## <a name="create-luis-keys-to-increase-total-endpoint-quota"></a>建立 LUIS 金鑰以增加端點總配額
1. 在 Azure 入口網站中，建立兩個 **Language Understanding** 金鑰，一個在 `West US`，另一個在 `East US`。 使用在上一節建立、名為 `luis-traffic-manager` 的現有資源群組。 

    ![Azure 入口網站的 luis-traffic-manager 資源群組中有兩個 LUIS 金鑰的螢幕擷取畫面](./media/traffic-manager/luis-keys.png)

2. 在 [LUIS][LUIS] 網站的 [管理] 區段中，請在 [金鑰和端點] 頁面上，將金鑰指派給應用程式，然後選取右上方功能表中的 [發佈] 按鈕來重新發佈應用程式。 

    [端點] 欄中的範例 URL 使用 GET 要求搭配端點金鑰作為查詢參數。 複製兩個新的金鑰端點 URL。 本文稍後的流量管理員設定會用到這些 URL。

## <a name="manage-luis-endpoint-requests-across-keys-with-traffic-manager"></a>使用流量管理員管理金鑰之間的 LUIS 端點要求
流量管理員會為您的端點建立新的 DNS 存取點。 這個存取點不會做為閘道或 Proxy，只會在 DNS 層級。 此範例不會變更任何 DNS 記錄。 它會使用 DNS 程式庫與流量管理員通訊，為該特定要求取得正確的端點。 _每個_給 LUIS 的要求一開始都需要一個流量管理員要求，以判斷要使用哪一個 LUIS 端點。 

### <a name="polling-uses-luis-endpoint"></a>輪詢使用 LUIS 端點
流量管理員會定期輪詢端點，確定端點仍然可用。 輪詢的流量管理員 URL 必須可使用 GET 要求存取，並傳回 200。 [發佈] 頁面上的端點 URL 就是如此。 因為每個端點金鑰都有不同的路由和查詢字串參數，所以每個端點金鑰都需要不同的輪詢路徑。 每當流量管理員輪詢時，就會花費配額要求。 LUIS 端點的查詢字串參數 **q** 是傳送至 LUIS 的語句。 此參數不會傳送語句，而是用來在 LUIS 端點記錄中加入流量管理員的輪詢，做為設定流量管理員時的一種偵錯技巧。

因為每個 LUIS 端點都需要自己的路徑，所以需要自己的流量管理員設定檔。 若要管理這些設定檔，請建立一個[_巢狀_流量管理員](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-nested-profiles)架構。 一個父系設定檔指向子系設定檔，並管理它們之間的流量。

設定好流量管理員之後，請記得變更路徑以使用 logging=false 查詢字串參數，讓您的記錄不會被輪詢填滿。

## <a name="configure-traffic-manager-with-nested-profiles"></a>使用巢狀設定檔設定流量管理員
以下各節會建立兩個子系設定檔，一個用於東部 LUIS 金鑰，另一個用於西部 LUIS 金鑰。 接著，建立一個父系設定檔，並在這個父系設定檔中加入兩個子系設定檔。 

### <a name="create-the-east-us-traffic-manager-profile-with-powershell"></a>使用 PowerShell 建立美國東部流量管理員設定檔
若要建立美國東部流量管理員設定檔，有幾個步驟：建立設定檔、新增端點和設定端點。 流量管理員設定檔可以有多個端點，但是每個端點都有相同的驗證路徑。 東部和西部訂用帳戶的 LUIS 端點 URL 會因為區域和端點金鑰而不同，因此每個 LUIS 端點在設定檔中都必須是單一端點。 

1. 使用 **[New-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/new-azurermtrafficmanagerprofile?view=azurermps-6.2.0)** Cmdlet 建立設定檔

    使用下列 Cmdlet 建立設定檔。 請務必變更 `appIdLuis` 和 `subscriptionKeyLuis`。 subscriptionKey 適用於美國東部 LUIS 金鑰。 如果路徑不正確 (包括 LUIS 應用程式識別碼和端點金鑰)，流量管理員輪詢會是 `degraded` 的狀態，因為流量管理員無法成功要求 LUIS 端點。 確定 `q` 的值是 `traffic-manager-east`，以便您可以在 LUIS 端點記錄中看到這個值。

    ```PowerShell
    $eastprofile = New-AzureRmTrafficManagerProfile -Name luis-profile-eastus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-eastus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appID>?subscription-key=<subscriptionKey>&q=traffic-manager-east"
    ```
    
    下表說明 Cmdlet 中的每個變數：
    
    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-Name|luis-profile-eastus|Azure 入口網站中的流量管理員名稱|
    |-ResourceGroupName|luis-traffic-manager|已在上一節中建立|
    |-TrafficRoutingMethod|效能|如需詳細資訊，請參閱[流量管理員路由方法][routing-methods]。 如果使用 performance，對流量管理員的 URL 要求必須來自使用者的區域。 如果經過 Chatbot 或其他應用程式，Chatbot 必須負責在對流量管理員的呼叫中模擬區域。 |
    |-RelativeDnsName|luis-dns-eastus|這是服務的子網域：luis-dns-eastus.trafficmanager.net|
    |-Ttl|30|輪詢間隔，30 秒|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|LUIS 的連接埠和通訊協定是 HTTPS/443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-east`|以您自己的值取代 <appIdLuis> 和 <subscriptionKeyLuis>。|
    
    成功的要求沒有回應。

2. 使用 **[Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/add-azurermtrafficmanagerendpointconfig?view=azurermps-6.2.0)** Cmdlet 新增美國東部端點

    ```PowerShell
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName luis-east-endpoint -TrafficManagerProfile $eastprofile -Type ExternalEndpoints -Target eastus.api.cognitive.microsoft.com -EndpointLocation "eastus" -EndpointStatus Enabled
    ```
    下表說明 Cmdlet 中的每個變數：

    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-EndpointName|luis-east-endpoint|設定檔下顯示的端點名稱|
    |-TrafficManagerProfile|$eastprofile|使用在步驟 1 建立的設定檔物件|
    |-Type|ExternalEndpoints|如需詳細資訊，請參閱[流量管理員端點][traffic-manager-endpoints] |
    |-Target|eastus.api.cognitive.microsoft.com|這是 LUIS 端點的網域。|
    |-EndpointLocation|"eastus"|端點的區域|
    |-EndpointStatus|已啟用|在端點建立時啟用端點|

    成功的回應看起來會像這樣：

    ```cmd
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-eastus
    Name                             : luis-profile-eastus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-eastus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/<luis-app-id>?subscription-key=f0517d185bcf467cba5147d6260bb868&q=traffic-manager-east
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-east-endpoint}
    ```

3. 使用 **[Set-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/set-azurermtrafficmanagerprofile?view=azurermps-6.2.0)** Cmdlet 設定美國東部端點

    ```PowerShell
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $eastprofile
    ```

    成功的回應會與步驟 2 的回應相同。

### <a name="create-the-west-us-traffic-manager-profile-with-powershell"></a>使用 PowerShell 建立美國西部流量管理員設定檔
若要建立美國西部流量管理員設定檔，請依照相同步驟：建立設定檔、新增端點和設定端點。

1. 使用 **[New-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/New-AzureRmTrafficManagerProfile?view=azurermps-6.2.0)** Cmdlet 建立設定檔

    使用下列 Cmdlet 建立設定檔。 請務必變更 `appIdLuis` 和 `subscriptionKeyLuis`。 subscriptionKey 適用於美國東部 LUIS 金鑰。 如果路徑不正確 (包括 LUIS 應用程式識別碼和端點金鑰)，流量管理員輪詢會是 `degraded` 的狀態，因為流量管理員無法成功要求 LUIS 端點。 確定 `q` 的值是 `traffic-manager-west`，以便您可以在 LUIS 端點記錄中看到這個值。

    ```PowerShell
    $westprofile = New-AzureRmTrafficManagerProfile -Name luis-profile-westus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-westus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west"
    ```
    
    下表說明 Cmdlet 中的每個變數：
    
    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-Name|luis-profile-westus|Azure 入口網站中的流量管理員名稱|
    |-ResourceGroupName|luis-traffic-manager|已在上一節中建立|
    |-TrafficRoutingMethod|效能|如需詳細資訊，請參閱[流量管理員路由方法][routing-methods]。 如果使用 performance，對流量管理員的 URL 要求必須來自使用者的區域。 如果經過 Chatbot 或其他應用程式，Chatbot 必須負責在對流量管理員的呼叫中模擬區域。 |
    |-RelativeDnsName|luis-dns-westus|這是服務的子網域：luis-dns-westus.trafficmanager.net|
    |-Ttl|30|輪詢間隔，30 秒|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|LUIS 的連接埠和通訊協定是 HTTPS/443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west`|以您自己的值取代 <appId> 和 <subscriptionKey>。 請記住，此端點金鑰不同於東部端點金鑰|
    
    成功的要求沒有回應。

2. 使用 **[Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Add-AzureRmTrafficManagerEndpointConfig?view=azurermps-6.2.0)** Cmdlet 新增美國西部端點

    ```PowerShell
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName luis-west-endpoint -TrafficManagerProfile $westprofile -Type ExternalEndpoints -Target westus.api.cognitive.microsoft.com -EndpointLocation "westus" -EndpointStatus Enabled
    ```

    下表說明 Cmdlet 中的每個變數：

    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-EndpointName|luis-west-endpoint|設定檔下顯示的端點名稱|
    |-TrafficManagerProfile|$westprofile|使用在步驟 1 建立的設定檔物件|
    |-Type|ExternalEndpoints|如需詳細資訊，請參閱[流量管理員端點][traffic-manager-endpoints] |
    |-Target|westus.api.cognitive.microsoft.com|這是 LUIS 端點的網域。|
    |-EndpointLocation|"westus"|端點的區域|
    |-EndpointStatus|已啟用|在端點建立時啟用端點|

    成功的回應看起來會像這樣：

    ```cmd
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-westus
    Name                             : luis-profile-westus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-westus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/c3fc5d1e-5187-40cc-af0f-fbde328aa16b?subscription-key=e3605f07e3cc4bedb7e02698a54c19cc&q=traffic-manager-west
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-west-endpoint}
    ```

3. 使用 **[Set-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Set-AzureRmTrafficManagerProfile?view=azurermps-6.2.0)** Cmdlet 設定美國西部端點

    ```PowerShell
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $westprofile
    ```

    成功的回應會與步驟 2 的回應相同。

### <a name="create-parent-traffic-manager-profile"></a>建立父系流量管理員設定檔
建立父系流量管理員設定檔，並將兩個子系流量管理員設定檔連結到父系。

1. 使用 **[New-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/New-AzureRmTrafficManagerProfile?view=azurermps-6.2.0)** Cmdlet 建立父系設定檔

    ```PowerShell
    $parentprofile = New-AzureRmTrafficManagerProfile -Name luis-profile-parent -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-parent -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/"
    ```

    下表說明 Cmdlet 中的每個變數：

    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-Name|luis-profile-parent|Azure 入口網站中的流量管理員名稱|
    |-ResourceGroupName|luis-traffic-manager|已在上一節中建立|
    |-TrafficRoutingMethod|效能|如需詳細資訊，請參閱[流量管理員路由方法][routing-methods]。 如果使用 performance，對流量管理員的 URL 要求必須來自使用者的區域。 如果經過 Chatbot 或其他應用程式，Chatbot 必須負責在對流量管理員的呼叫中模擬區域。 |
    |-RelativeDnsName|luis-dns-parent|這是服務的子網域：luis-dns-parent.trafficmanager.net|
    |-Ttl|30|輪詢間隔，30 秒|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|LUIS 的連接埠和通訊協定是 HTTPS/443|
    |-MonitorPath|`/`|此路徑不重要，因為會改為使用子端點路徑。|

    成功的要求沒有回應。

2. 使用 **[Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Add-AzureRmTrafficManagerEndpointConfig?view=azurermps-6.2.0)** 和 **NestedEndpoints** 類型，將美國東部子系設定檔新增至父系

    ```PowerShell
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName child-endpoint-useast -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $eastprofile.Id -EndpointStatus Enabled -EndpointLocation "eastus" -MinChildEndpoints 1
    ```

    下表說明 Cmdlet 中的每個變數：

    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-EndpointName|child-endpoint-useast|東部設定檔|
    |-TrafficManagerProfile|$parentprofile|要將此端點指派至的設定檔|
    |-Type|NestedEndpoints|如需詳細資訊，請參閱 [Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/Add-AzureRmTrafficManagerEndpointConfig?view=azurermps-6.2.0)。 |
    |-TargetResourceId|$eastprofile.Id|子系設定檔的識別碼|
    |-EndpointStatus|已啟用|新增至父系之後的端點狀態|
    |-EndpointLocation|"eastus"|資源的 [Azure 區域名稱](https://azure.microsoft.com/global-infrastructure/regions/)|
    |-MinChildEndpoints|1|子端點的數目下限|

    成功的回應看起來會像下面這樣，而且包括新的 `child-endpoint-useast` 端點：    

    ```cmd
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast}
    ```

3. 使用 **[Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Add-AzureRmTrafficManagerEndpointConfig?view=azurermps-6.2.0)** Cmdlet 和 **NestedEndpoints** 類型，將美國西部子系設定檔新增至父系

    ```PowerShell
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName child-endpoint-uswest -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $westprofile.Id -EndpointStatus Enabled -EndpointLocation "westus" -MinChildEndpoints 1
    ```

    下表說明 Cmdlet 中的每個變數：

    |設定參數|變數名稱或值|目的|
    |--|--|--|
    |-EndpointName|child-endpoint-uswest|西部設定檔|
    |-TrafficManagerProfile|$parentprofile|要將此端點指派至的設定檔|
    |-Type|NestedEndpoints|如需詳細資訊，請參閱 [Add-AzureRmTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager/Add-AzureRmTrafficManagerEndpointConfig?view=azurermps-6.2.0)。 |
    |-TargetResourceId|$westprofile.Id|子系設定檔的識別碼|
    |-EndpointStatus|已啟用|新增至父系之後的端點狀態|
    |-EndpointLocation|"westus"|資源的 [Azure 區域名稱](https://azure.microsoft.com/global-infrastructure/regions/)|
    |-MinChildEndpoints|1|子端點的數目下限|

    成功的回應會像下面這樣，並且包含先前的 `child-endpoint-useast` 端點和新的 `child-endpoint-uswest` 端點兩者：

    ```cmd
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast, child-endpoint-uswest}
    ```

4. 使用 **[Set-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Set-AzureRmTrafficManagerProfile?view=azurermps-6.2.0)** Cmdlet 設定端點 

    ```PowerShell
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $parentprofile
    ```

    成功的回應會與步驟 3 的回應相同。

### <a name="powershell-variables"></a>PowerShell 變數
在先前各節中，我們建立了三個 PowerShell 變數：`$eastprofile`、`$westprofile`、`$parentprofile`。 這些變數會一直用到流量管理員設定的最後。 如果您選擇不建立變數、忘記建立，或者 PowerShell 視窗逾時，則可以使用 PowerShell Cmdlet **[Get-AzureRmTrafficManagerProfile](https://docs.microsoft.com/powershell/module/AzureRM.TrafficManager/Get-AzureRmTrafficManagerProfile?view=azurermps-6.2.0)** 再次取得設定檔，並將它指派給變數。 

針對您需要的三個設定檔，將角括弧 `<>` 中的項目取代為正確值。 

```PowerShell
$<variable-name> = Get-AzureRmTrafficManagerProfile -Name <profile-name> -ResourceGroupName luis-traffic-manager
```

## <a name="verify-traffic-manager-works"></a>確認流量管理員可運作
若要確認流量管理員設定檔可運作，設定檔必須處於 `Online` 的狀態，這個狀態是以端點的輪詢路徑為基礎。 

### <a name="view-new-profiles-in-the-azure-portal"></a>在 Azure 入口網站中檢視新的設定檔
您可以查看 `luis-traffic-manager` 資源群組中的資源，確認所有三個設定檔都已建立。

![Azure 資源群組 luis-traffic-manager 的螢幕擷取畫面](./media/traffic-manager/traffic-manager-profiles.png)

### <a name="verify-the-profile-status-is-online"></a>確認設定檔狀態為 Online
流量管理員會輪詢每個端點的路徑，確定端點已在線上。 如果在線上，子系設定檔的狀態會是 `Online`。 這會顯示在每個設定檔的 [概觀] 頁面上。 

![Azure 流量管理員設定檔的 [概觀] 顯示 Online 監視狀態的螢幕擷取畫面](./media/traffic-manager/profile-status-online.png)

### <a name="validate-traffic-manager-polling-works"></a>驗證流量管理員輪詢可運作
另一個驗證流量管理員輪詢可運作的方法是使用 LUIS 端點記錄。 在 [LUIS][LUIS] 網站的應用程式清單頁面上，匯出應用程式的端點記錄。 因為流量管理員會經常輪詢兩個端點，所以記錄中會有項目 (即使只存在幾分鐘的時間)。 請記得尋找查詢以 `traffic-manager-` 開始的項目。

```text
traffic-manager-west    6/7/2018 19:19  {"query":"traffic-manager-west","intents":[{"intent":"None","score":0.944767}],"entities":[]}
traffic-manager-east    6/7/2018 19:20  {"query":"traffic-manager-east","intents":[{"intent":"None","score":0.944767}],"entities":[]}
```

### <a name="validate-dns-response-from-traffic-manager-works"></a>驗證來自流量管理員的 DNS 回應可運作
若要驗證 DNS 回應傳回 LUIS 端點，請使用 DNS 用戶端程式庫要求流量管理父系設定檔 DNS。 父系設定檔的 DNS 名稱是 `luis-dns-parent.trafficmanager.net`。

下列 Node.js 程式碼向父系設定檔提出要求，並傳回 LUIS 端點：

```javascript
const dns = require('dns');

dns.resolveAny('luis-dns-parent.trafficmanager.net', (err, ret) => {
  console.log('ret', ret);
});
```

LUIS 端點成功的回應為：

```json
[
    {
        value: 'westus.api.cognitive.microsoft.com', 
        type: 'CNAME'
    }
]
```

## <a name="use-the-traffic-manager-parent-profile"></a>使用流量管理員父系設定檔
若要管理端點之間的流量，您必須插入對流量管理員 DNS 的呼叫以尋找 LUIS 端點。 這個呼叫是針對每個 LUIS 端點要求所發出，而且必須模擬 LUIS 用戶端應用程式的使用者地理位置。 在您的 LUIS 用戶端應用程式與向 LUIS 取得端點預測的要求之間新增 DNS 回應碼。 


## <a name="clean-up"></a>清除
移除這兩個 LUIS 端點金鑰、這三個流量管理員設定檔，以及包含這五個資源的資源群組。 這些工作是從 Azure 入口網站進行。 您要從資源清單刪除五個資源。 然後刪除資源群組。 

## <a name="next-steps"></a>後續步驟

檢閱 BotFramework v4 中的[中介軟體](https://docs.microsoft.com/azure/bot-service/bot-builder-create-middleware?view=azure-bot-service-4.0&tabs=csaddmiddleware%2Ccsetagoverwrite%2Ccsmiddlewareshortcircuit%2Ccsfallback%2Ccsactivityhandler)選項，了解如何在 BotFramework Bot 中新增此流量管理程式碼。 

[traffic-manager-marketing]: https://azure.microsoft.com/services/traffic-manager/
[traffic-manager-docs]: https://docs.microsoft.com/azure/traffic-manager/
[LUIS]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions#luis-website
[azure-portal]: https://portal.azure.com/
[azure-storage]: https://azure.microsoft.com/services/storage/
[routing-methods]: https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods
[traffic-manager-endpoints]: https://docs.microsoft.com/azure/traffic-manager/traffic-manager-endpoint-types
