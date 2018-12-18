---
title: 監視應用程式閘道的存取記錄、效能記錄、後端健康情況及計量
description: 了解如何啟用和管理應用程式閘道的存取記錄和效能記錄
services: application-gateway
author: amitsriva
manager: rossort
tags: azure-resource-manager
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 6/20/2018
ms.author: amitsriva
ms.openlocfilehash: 563194ea0b3e4bda2021c75c544d068f00d74ba7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963827"
---
# <a name="back-end-health-diagnostic-logs-and-metrics-for-application-gateway"></a>應用程式閘道的後端健康情況、診斷記錄和計量

使用 Azure 應用程式閘道，您可以下列方式監視資源：

* [後端健康情況](#back-end-health)：應用程式閘道提供透過 Azure 入口網站和 PowerShell 監視後端集區中伺服器健康情況的功能。 您也可以透過效能診斷記錄找到後端集區的健康情況。

* [記錄](#diagnostic-logging)：記錄能夠儲存效能、存取和其他資料，或從資源取用記錄以便進行監視。

* [計量](#metrics)：應用程式閘道目前有七個計量，可用來檢視效能計數器。

## <a name="back-end-health"></a>後端健康情況

應用程式閘道提供透過入口網站、PowerShell 及命令列界面 (CLI) 監視後端集區中個別成員健康情況的功能。 您也可以透過效能診斷記錄找到彙總的後端集區健康情況摘要。 

後端健康情況報表會將應用程式閘道健康情況探查的輸出反映到後端執行個體。 當探查成功且後端可以接收流量，則視為狀況良好。 否則，視為狀況不良。

> [!IMPORTANT]
> 如果應用程式閘道子網路上有網路安全性群組 (NSG)，請在應用程式閘道子網路上開啟連接埠範圍 65503-65534，供輸入流量使用。 Azure 基礎結構通訊需要此連接埠範圍。 它們受到 Azure 憑證的保護 (鎖定)。 若沒有適當的憑證，外部實體 (包括這些閘道的客戶) 將無法對這些端點起始任何變更。


### <a name="view-back-end-health-through-the-portal"></a>透過入口網站檢視後端健康情況

入口網站中會自動提供後端的健康情況。 在現有的應用程式閘道中，選取 [監視]  >  [後端健康情況]。 

後端集區中的每個成員均會列於此頁面中 (無論是 NIC、IP 或 FQDN)。 後端集區名稱、連接埠、後端 HTTP 設定名稱和健康情況均會顯示。 健康情況的有效值為「狀況良好」、「狀況不良」和「未知」。

> [!NOTE]
> 如果您看到後端健康情況為 [未知]，請確定 NSG 規則、使用者定義的路由 (UDR) 或虛擬網路中的自訂 DNS 並未封鎖對後端的存取。

![後端健康情況][10]

### <a name="view-back-end-health-through-powershell"></a>透過 PowerShell 檢視後端健康情況

下列 PowerShell 程式碼示範如何使用 `Get-AzureRmApplicationGatewayBackendHealth` Cmdlet 檢視後端健康情況：

```powershell
Get-AzureRmApplicationGatewayBackendHealth -Name ApplicationGateway1 -ResourceGroupName Contoso
```

### <a name="view-back-end-health-through-azure-cli"></a>透過 Azure CLI 檢視後端健康情況

```azurecli
az network application-gateway show-backend-health --resource-group AdatumAppGatewayRG --name AdatumAppGateway
```

### <a name="results"></a>結果

以下是回應範例：

```json
{
"BackendAddressPool": {
    "Id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/appGatewayBackendPool"
},
"BackendHttpSettingsCollection": [
    {
    "BackendHttpSettings": {
        "Id": "/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
    },
    "Servers": [
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        },
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        }
    ]
    }
]
}
```

## <a name="diagnostic-logging"></a>診斷記錄

您可以在 Azure 中使用不同類型的記錄檔來管理和針對應用程式閘道進行疑難排解。 您可以透過入口網站存取其中一些記錄。 可以從 Azure Blob 儲存體擷取所有記錄，並且在不同的工具中進行檢視 (例如 [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md)、Excel、PowerBI)。 您可以從下列清單進一步了解不同類型的記錄檔：

* **活動記錄**：您可以使用 [Azure 活動記錄](../monitoring-and-diagnostics/insights-debugging-with-events.md) (之前稱為「作業記錄和稽核記錄」) 來檢視提交至您的 Azure 訂用帳戶的所有作業及其狀態。 預設會收集活動記錄，您可在 Azure 入口網站中檢視它們。
* **存取記錄**：您可以使用此記錄來檢視應用程式閘道存取模式及分析重要資訊，包括呼叫端的 IP、要求的 URL、回應延遲、傳回碼、輸入和輸出位元組。每隔 300 秒會收集一次存取記錄。 此記錄檔包含每個應用程式閘道執行個體的一筆記錄。 應用程式閘道執行個體可以由 instanceId 屬性識別。
* **效能記錄**：您可以使用此記錄來檢視應用程式閘道執行個體的執行情況。 此記錄會擷取每個執行個體的效能資訊，包括提供的要求總數、輸送量 (以位元組為單位)、提供的總要求數、失敗的要求計數、狀況良好和狀況不良的後端執行個體計數。 每隔 60 秒會收集一次效能記錄。
* **防火牆記錄**：您可以使用此記錄，檢視透過應用程式閘道的偵測或防止模式 (依 Web 應用程式防火牆的設定) 所記錄的要求。

> [!NOTE]
> 記錄僅適用於在 Azure Resource Manager 部署模型中部署的資源。 您無法將記錄檔使用於傳統部署模型中的資源。 若要深入了解這兩個模型，請參閱[了解 Resource Manager 部署和傳統部署](../azure-resource-manager/resource-manager-deployment-model.md)一文。

您有三個選項可用來排序您的記錄：

* **儲存體帳戶**：如果記錄會儲存一段較長的持續期間，並在需要時加以檢閱，則最好針對記錄使用儲存體帳戶。
* **事件中樞**：如果要整合其他安全性資訊和事件管理 (SEIM) 工具以便在資源上取得警示，則事件中樞是絕佳的選項。
* **Log Analytics**：Log Analytics 最適合用來進行應用程式的一般即時監視，或查看趨勢。

### <a name="enable-logging-through-powershell"></a>透過 PowerShell 啟用記錄功能

每個 Resource Manager 資源都會自動啟用活動記錄功能。 您必須啟用存取和效能記錄功能，才能開始收集可透過這些記錄檔取得的資料。 使用下列步驟啟用記錄：

1. 請記下您的儲存體帳戶的資源識別碼 (記錄資料的儲存之處)。 此值的形式為：/subscriptions/\<subscriptionId\>/resourceGroups/\<資源群組名稱\>/providers/Microsoft.Storage/storageAccounts/\<儲存體帳戶名稱\>。 您可以使用訂用帳戶中的所有儲存體帳戶。 您可以使用 Azure 入口網站來尋找此資訊。

    ![入口網站：儲存體帳戶的資源識別碼](./media/application-gateway-diagnostics/diagnostics1.png)

2. 請記下您的應用程式閘道的資源識別碼 (將為其啟用記錄功能)。 此值的形式為：/subscriptions/\<subscriptionId\>/resourceGroups/\<資源群組名稱\>/providers/Microsoft.Network/applicationGateways/\<應用程式閘道名稱\>。 您可以使用入口網站來尋找此資訊。

    ![入口網站：應用程式閘道的資源識別碼](./media/application-gateway-diagnostics/diagnostics2.png)

3. 使用下列 Powershell Cmdlet 啟用診斷記錄功能：

    ```powershell
    Set-AzureRmDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name> -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> -Enabled $true     
    ```
    
> [!TIP] 
>活動記錄檔不需要個別的儲存體帳戶。 將儲存體用於記錄存取和效能會產生服務費用。

### <a name="enable-logging-through-the-azure-portal"></a>透過 Azure 入口網站啟用記錄功能

1. 在 Azure 入口網站中，找到您的資源並按一下 [診斷記錄]。

   應用程式閘道有三個記錄：

   * 存取記錄檔
   * 效能記錄檔
   * 防火牆記錄檔

2. 若要開始收集資料，請按一下 [開啟診斷]。

   ![開啟診斷][1]

3. [診斷設定] 刀鋒視窗中提供診斷記錄的設定。 在此範例中，Log Analytics 會儲存記錄。 按一下 [Log Analytics] 底下的 [設定] 來設定您的工作區。 您也可以使用事件中樞和儲存體帳戶來儲存診斷記錄。

   ![啟動設定程序][2]

4. 選擇現有的 Log Analytics 工作區，或建立新的工作區。 這個範例使用現有工作區。

   ![Log Analytics 工作區的選項][3]

5. 確認設定並按一下 [儲存]。

   ![診斷設定刀鋒視窗與選項][4]

### <a name="activity-log"></a>活動記錄檔

根據預設，Azure 會產生活動記錄。 記錄會在 Azure 的事件記錄存放區中保留 90 天。 閱讀[檢視事件和活動記錄](../monitoring-and-diagnostics/insights-debugging-with-events.md)一文，深入了解這些記錄。

### <a name="access-log"></a>存取記錄檔

只有當您如上述步驟所述，在每個應用程式閘道上啟用存取記錄，才會產生存取記錄。 資料會儲存在您啟用記錄功能時指定的儲存體帳戶中。 應用程式閘道的每次存取會以 JSON 格式記錄下來，如下列範例所示：


|值  |說明  |
|---------|---------|
|instanceId     | 處理要求的應用程式閘道執行個體。        |
|clientIP     | 要求的原始 IP。        |
|clientPort     | 要求的原始連接埠。       |
|httpMethod     | 要求使用的 HTTP 方法。       |
|requestUri     | 接收之要求的 URI。        |
|RequestQuery     | **Server-Routed**：傳送要求的後端集區執行個體。</br>**X-AzureApplicationGateway-LOG-ID**：要求所使用的相互關聯識別碼。 它可以用來針對後端伺服器上的流量問題進行疑難排解。 </br>**SERVER-STATUS**：應用程式閘道從後端收到的 HTTP 回應碼。       |
|UserAgent     | HTTP 要求標頭中的使用者代理程式。        |
|httpStatus     | 應用程式閘道傳回用戶端的 HTTP 狀態碼。       |
|httpVersion     | 要求的 HTTP 版本。        |
|receivedBytes     | 接收的封包大小，單位為位元組。        |
|sentBytes| 傳送的封包大小，單位為位元組。|
|timeTaken| 處理要求並傳送其回應所花費的時間長度，單位為毫秒。 算法是從應用程式閘道收到 HTTP 要求的回應第一個位元組的時間，到回應傳送作業完成時的時間間隔。 請務必注意，timeTaken 欄位通常包含要求和回應封包在網路上傳輸的時間。 |
|sslEnabled| 與後端集區的通訊是否使用 SSL。 有效值為 on 和 off。|
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "ApplicationGatewayRole_IN_0",
        "clientIP": "191.96.249.97",
        "clientPort": 46886,
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "requestQuery": "X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.4.0.4&X-AzureApplicationGateway-LOG-ID=874f1f0f-6807-41c9-b7bc-f3cfa74aa0b1&SERVER-STATUS=404",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off"
    }
}
```

### <a name="performance-log"></a>效能記錄檔

只有當您如上述步驟所述，在每個應用程式閘道上啟用效能記錄，才會產生效能記錄。 資料會儲存在您啟用記錄功能時指定的儲存體帳戶中。 產生效能記錄資料的時間間隔為 1 分鐘。 會記錄下列資料：


|值  |說明  |
|---------|---------|
|instanceId     |  將產生此應用程式閘道執行個體的效能資料。 應用程式閘道若有多個執行個體，則是一個執行個體一行資料。        |
|healthyHostCount     | 後端集區中狀況良好主機的數目。        |
|unHealthyHostCount     | 後端集區中狀況不良主機的數目。        |
|requestCount     | 處理的要求數目。        |
|latency | 從執行個體到處理要求的後端之間的要求延遲，單位為毫秒。 |
|failedRequestCount| 失敗的要求數目。|
|throughput| 自最後一個記錄以來的平均輸送量，測量單位為每秒位元組。|

```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayPerformance",
    "time": "2016-04-09T00:00:00Z",
    "category": "ApplicationGatewayPerformanceLog",
    "properties":
    {
        "instanceId":"ApplicationGatewayRole_IN_1",
        "healthyHostCount":"4",
        "unHealthyHostCount":"0",
        "requestCount":"185",
        "latency":"0",
        "failedRequestCount":"0",
        "throughput":"119427"
    }
}
```

> [!NOTE]
> 延遲的計算是從收到 HTTP 要求的第一個位元組時算起，到送出 HTTP 回應的最後一個位元組時結束。 是應用程式閘道處理時間，加上網路傳送到後端的時間，再加上後端處理要求所花時間的總和。

### <a name="firewall-log"></a>防火牆記錄檔

只有當您如上述步驟所述，在每個應用程式閘道上啟用防火牆記錄，才會產生防火牆記錄。 此記錄也需要在應用程式閘道上設定該 Web 應用程式防火牆。 資料會儲存在您啟用記錄功能時指定的儲存體帳戶中。 會記錄下列資料：


|值  |說明  |
|---------|---------|
|instanceId     | 將產生此應用程式閘道執行個體的防火牆資料。 應用程式閘道若有多個執行個體，則是一個執行個體一行資料。         |
|clientIp     |   要求的原始 IP。      |
|clientPort     |  要求的原始連接埠。       |
|requestUri     | 接收之要求的 URL。       |
|ruleSetType     | 規則集類型。 可用的值是 OWASP。        |
|ruleSetVersion     | 規則集版本。 可用值為 2.2.9 和 3.0。     |
|ruleId     | 觸發事件的規則識別碼。        |
|Message     | 方便使用的觸發事件訊息。 詳細資料區段中會提供詳細資料。        |
|動作     |  對要求採取的動作。 可用的值為 Blocked 和 Allowed。      |
|site     | 將產生此網站的記錄。 目前只列出 Global，因為規則為全域。|
|詳細資料     | 觸發事件的詳細資料。        |
|details.message     | 規則的描述。        |
|details.data     | 在符合規則之要求中找到的特定資料。         |
|details.file     | 包含規則的組態檔。        |
|details.line     | 觸發事件的組態檔中的行號。       |

```json
{
  "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
  "operationName": "ApplicationGatewayFirewall",
  "time": "2017-03-20T15:52:09.1494499Z",
  "category": "ApplicationGatewayFirewallLog",
  "properties": {
    "instanceId": "ApplicationGatewayRole_IN_0",
    "clientIp": "104.210.252.3",
    "clientPort": "4835",
    "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "ruleId": "941320",
    "message": "Possible XSS Attack Detected - HTML Tag Handler",
    "action": "Blocked",
    "site": "Global",
    "details": {
      "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
      "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
      "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
      "line": "865"
    }
  }
} 

```

### <a name="view-and-analyze-the-activity-log"></a>檢視和分析活動記錄檔

您可以使用下列任何方法，檢視和分析活動記錄資料：

* **Azure 工具**：透過 Azure PowerShell、Azure CLI、Azure REST API 或 Azure 入口網站，從活動記錄擷取資訊。 [活動作業與 Resource Manager](../azure-resource-manager/resource-group-audit.md) 一文會詳述每個方法的逐步指示。
* **Power BI**：如果還沒有 [Power BI](https://powerbi.microsoft.com/pricing) 帳戶，您可以免費試用。 使用 [Power BI 的 Azure 活動記錄內容套件](https://powerbi.microsoft.com/en-us/documentation/powerbi-content-pack-azure-audit-logs/)，您可以使用預先設定的儀表板 (可按原樣使用或加以自訂) 來分析資料。

### <a name="view-and-analyze-the-access-performance-and-firewall-logs"></a>檢視及分析存取、效能和防火牆記錄

Azure [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md) 可以從您的 Blob 儲存體帳戶收集計數器和事件記錄檔。 它也納入了視覺效果和強大的搜尋功能來分析您的記錄。

您也可以連接到儲存體帳戶並擷取存取和效能記錄檔的 JSON 記錄檔項目。 下載 JSON 檔案後，可以將它們轉換成 CSV，並在 Excel、PowerBI 或任何其他資料視覺化工具中檢視它們。

> [!TIP]
> 如果您熟悉 Visual Studio 以及在 C# 中變更常數和變數值的基本概念，您可以使用 GitHub 所提供的[記錄檔轉換器工具 (英文)](https://github.com/Azure-Samples/networking-dotnet-log-converter)。
> 
> 

#### <a name="analyzing-access-logs-through-goaccess"></a>透過 GoAccess 分析存取記錄

我們已發佈會安裝並執行常用 [GoAccess](https://goaccess.io/) 記錄分析器的 Resource Manager 範本，該分析器適用於應用程式閘道存取記錄。 GoAccess 提供實用的 HTTP 流量統計資料，例如非重複訪客、要求的檔案、主機、作業系統、瀏覽器、HTTP 狀態碼等等。 如需詳細資訊，請參閱 [GitHub 中 Resource Manager 範本資料夾中的讀我檔案](https://aka.ms/appgwgoaccessreadme) \(英文\)。

## <a name="metrics"></a>計量

計量是某些 Azure 資源的功能，可供您在入口網站中檢視效能計數器。 應用程式閘道可使用下列計量：

- **目前的連線數**
- **失敗的要求**
- **狀況良好的主機計數**

   您可以根據每個後端集區進行篩選，以顯示特定後端集區中狀況良好/狀況不良的主機。


- **回應狀態**

   回應狀態碼發佈可以進一步分類，以顯示回應 2xx、3xx、4xx 和 5xx 分類中的回應。

- **輸送量**
- **要求總數**
- **狀況不良的主機計數**

   您可以根據每個後端集區進行篩選，以顯示特定後端集區中狀況良好/狀況不良的主機。

瀏覽至應用程式閘道，在 [監視] 之下按一下 [計量]。 若要檢視可用的值，請選取 [計量] 下拉式清單。

下圖中的範例顯示了最近 30 分鐘內的三項計量：

[![](media/application-gateway-diagnostics/figure5.png "計量檢視")](media/application-gateway-diagnostics/figure5-lb.png#lightbox)

若要查看最新的度量清單，請參閱[支援 Azure Monitor 的計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md)。

### <a name="alert-rules"></a>警示規則

您可以根據資源的計量來啟動警示規則。 例如，如果應用程式閘道的輸送量高於、低於或等於臨界值達到一段指定時間，警示便可以呼叫 Webhook 或寄送電子郵件給系統管理員。

下列範例會逐步引導您建立警示規則，以在輸送量達到臨界值之後傳送電子郵件給系統管理員：

1. 按一下 [新增計量警示] 以開啟 [新增規則] 刀鋒視窗。 您也可以透過計量刀鋒視窗來到達此刀鋒視窗。

   ![新增計量警示按鈕][6]

2. 在 [新增規則] 刀鋒視窗中，填入名稱、條件、通知等區段，然後按一下 [確定]。

   * 在 [條件] 選取器中，選取這 4 個值之一：[大於]、[大於或等於]、[小於] 或 [小於或等於]。

   * 在 [期間] 選取器中，選取 5 分鐘到 6 小時的期間。

   * 如果選取 [傳送電子郵件給擁有者、參與者和讀者]，便可根據可存取該資源的使用者動態傳送電子郵件。 否則，您可以在 [其他系統管理員電子郵件] 方塊中提供以逗號分隔的使用者清單。

   ![新增規則刀鋒視窗][7]

如果達到臨界值，送達的電子郵件會類似下圖︰

![違反臨界值的電子郵件][8]

建立計量警示之後，就會出現警示的清單， 其中提供所有警示規則的概觀。

![警示和規則的清單][9]

若要深入了解警示通知，請參閱[接收警示通知](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)。

若要深入了解 Webhook 以及其如何與警示搭配使用，請造訪[針對 Azure 計量警示設定 Webhook](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。

## <a name="next-steps"></a>後續步驟

* 利用 [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md) 將計數器和事件記錄視覺化。
* [使用 Power BI 將您的 Azure 活動記錄視覺化](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx)部落格文章。
* [在 Power BI 和其他工具中檢視和分析 Azure 活動記錄](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)部落格文章。

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png
