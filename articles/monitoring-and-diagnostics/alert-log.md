---
title: 使用 Azure 監視器來建立、檢視及管理記錄警示
description: 使用 Azure 監視器來撰寫、檢視及管理 Azure 中的記錄警示規則。
author: msvijayn
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/15/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: 627c0ee5a2fab333b8a120867ab54ea76bc66823
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46976834"
---
# <a name="create-view-and-manage-log-alerts-using-azure-monitor"></a>使用 Azure 監視器來建立、檢視及管理記錄警示  

## <a name="overview"></a>概觀
本文會示範如何使用 Azure 入口網站內的警示介面來設定記錄警示。 警示規則的定義分成三個部分：
- 目標：要監視的特定 Azure 資源
- 條件：訊號中所示且會觸發動作的特定條件或邏輯
- 動作：傳送至通知 (電子郵件、SMS、Webhook 等) 接收者的特定呼叫。

使用**記錄警示**一詞來描述符合下列條件的警示：其訊號是以 [Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 或 [Application Insights](../application-insights/app-insights-analytics.md) 為基礎的自訂查詢。 閱讀[記錄警示 - 概觀](monitor-alerts-unified-log.md)以深入了解其功能、術語和類型。

> [!NOTE]
> [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 中常用的記錄資料現在也會在 Azure 監視器的計量平台上提供。 如需詳細資料檢視，請參閱[記錄的計量警示](monitoring-metric-alerts-logs.md)

## <a name="managing-log-alerts-from-the-azure-portal"></a>從 Azure 入口網站管理記錄警示

詳細的下一步是將使用 Azure 入口網站介面來使用記錄警示的逐步指南。

### <a name="create-a-log-alert-rule-with-the-azure-portal"></a>使用 Azure 入口網站建立記錄警示規則
1. 在[入口網站](https://portal.azure.com/)中選取 [監視]，然後在 [監視] 區段下選擇 [警示]。  
    ![監視](./media/monitor-alerts-unified/AlertsPreviewMenu.png)

1. 選取 [新增警示規則] 按鈕，以在 Azure 中建立新的警示。
    ![新增警示](./media/monitor-alerts-unified/AlertsPreviewOption.png)

1. [建立警示] 區段會以三個部分顯示：「定義警示條件」、「定義警示詳細資料」和「定義動作群組」。

    ![建立規則](./media/monitor-alerts-unified/AlertsPreviewAdd.png)

1.  使用 [選取資源] 連結並透過選取資源來指定目標，即可定義警示條件。 選擇 [訂閱]、[資源類型] 和 [資源] (必選)，即可進行篩選。 

    >[!NOTE]

    > 若要建立記錄警示 - 請先確認選取的資源可使用 **log** 訊號，再繼續下面的步驟。
    ![選取資源](./media/monitor-alerts-unified/Alert-SelectResourceLog.png)

 
1. 記錄警示：請確認 [資源類型] 是分析來源 (例如「Log Analytics」或「Application Insights」) 且訊號類型為 [記錄]，然後在選擇適當的 [資源] 後，按一下 [完成]。 接著使用 [新增準則] 按鈕，檢視資源可用的訊號選項清單，然後從訊號清單中選取所選記錄監視服務 (例如 *Log Analytics* 或 *Application Insights*) 的 [自訂記錄搜尋] 選項。

   ![選取資源 - 自訂記錄搜尋](./media/monitor-alerts-unified/AlertsPreviewResourceSelectionLog.png)

   > [!NOTE]

   > [警示] 清單可匯入分析查詢作為訊號類型 - [記錄 (已儲存的查詢)]，如上圖所示。 因此使用者可在 Analytics 中完成理想的查詢後，將其儲存以供日後在警示中使用 - 如需使用儲存查詢的詳細資料，請參閱[在記錄分析中使用記錄搜尋](../log-analytics/log-analytics-log-searches.md)或[在 Application Insights 分析中的共用的查詢](../log-analytics/log-analytics-overview.md)。 

1.  *記錄警示*：選好後，即可在 [搜尋查詢] 欄位中指定警示的查詢；如果查詢語法不正確，欄位會以紅色顯示錯誤。 如果查詢語法正確 - 指定查詢的記錄資料會顯示為圖形以供參考，以及顯示調整時間範圍的選項 (從過去六小時到過去一週)。

 ![設定警示規則](./media/monitor-alerts-unified/AlertsPreviewAlertLog.png)

 > [!NOTE]

    > 記錄資料的視覺效果只會在查詢結果具有時間詳細資料時才會顯示。 如果您的查詢結果的是摘要的資料或特定資料行的值，則會以單一繪圖顯示相同資料。

    >  若使用 Application Insights 的記錄警示類型是「計量測量」，您可以使用 [彙總依據] 選項來指定要群組資料的特定變數，如下所示：

    ![[依據彙總] 選項](./media/monitor-alerts-unified/aggregate-on.png)

1.  記錄警示：當視覺效果就緒後，您就可以從所示的條件、彙總和閾值 (最後選取) 選項來選取**警示邏輯**。 最後，使用 [期間] 選項，在邏輯中指定用來評估指定條件的時間。 以及，透過選取 [頻率]，來指定警示執行的頻率。
針對**記錄警示**，可以根據：
   - 記錄數目：如果查詢所傳回的記錄數目大於或小於所提供的值，則會建立警示。
   - 計量測量：如果結果中的每個「彙總值」超過提供的閾值，而且彙總值依據選擇的值來「分組」，則會建立警示。 警示的違規數是所選時段中超過閾值的次數。 您可以跨結果集針對任何違規組合指定 [違規數總計]，或指定 [連續違規] 以要求違規必須發生於連續取樣中。 深入了解[記錄警示和其類型](monitor-alerts-unified-log.md)。


1. 第二個步驟是在 [警示規則名稱] 欄位中定義警示的名稱，以及在 [描述] 中具體描述警示，並從提供的選項中選取 [嚴重性] 的值。 這些詳細資料會在所有由 Azure 監視器完成的警示電子郵件、通知或推送中重複使用。 此外，使用者可以透過適當地切換 [在建立時啟用規則] 選項，選擇在建立警示規則後立即啟用警示。

    警示詳細資料中有一些額外的功能可使用 (僅限於**記錄警示**)：

    - **隱藏警示**：當您開啟警示規則的隱藏功能時，此規則的動作會在建立新警示後停用並持續一段您所定義的時間。 此規則仍在執行中，並且會在符合準則時建立警示記錄。 讓您有時間更正問題，而不需執行重複的動作。

        ![記錄警示的隱藏警示](./media/monitor-alerts-unified/AlertsPreviewSuppress.png)

        > [!TIP]
        > 指定大於警示頻率的隱藏警示值，可確保通知會停止而不重疊

1. 第三個步驟 (也就是最後一個步驟) 是指定當警示條件符合時，是否有任何**動作群組**必須根據警示規則來觸發。 您可以選擇任何具有警示的現有動作群組，或建立新的動作群組。 根據選取的動作群組，當警示觸發時，Azure 將會：傳送電子郵件、傳送 SMS、呼叫 Webhook、使用 Azure Runbook 進行補救，發送到您的 ITSM 工具等等。 深入了解[動作群組](monitoring-action-groups.md)。

    > [!NOTE]
    > 請參閱 [Azure 訂用帳戶服務限制](../azure-subscription-service-limits.md)，了解透過 Azure 動作群組針對記錄警示觸發 Runbook 承載的限制

    針對**記錄警示**，有一些額外的功能可用來覆寫預設的動作：

    - **電子郵件通知**：覆寫透過「動作群組」傳送的電子郵件「電子郵件主旨」；如果前述的「動作群組」中存在一或多個電子郵件動作。 您無法修改電子郵件的本文，且此欄位**不是**用於電子郵件地址。
    - **包括自訂 JSON 承載**：覆寫「動作群組」所使用的 Webhook JSON；如果前述的「動作群組」中存在一或多個 Webhook 動作。 使用者可以指定要用於相關聯動作群組中設定的所有 Webhook JSON 格式；如需有關 Webhook 格式的詳細資訊，請參閱[記錄警示的 Webhook 動作](monitor-alerts-unified-log-webhook.md)。 提供的 [檢視 Webhook] 選項可用來檢查使用 JSON 資料範例的格式。

        ![記錄警示的動作覆寫](./media/monitor-alerts-unified/AlertsPreviewOverrideLog.png)


1. 如果所有欄位都有效，並且有綠色勾號，您就可以按一下 [建立警示規則] 按鈕，在 Azure [監視器 - 警示] 中建立警示。 您可以從「警示儀表板」中檢視所有警示。

    ![建立規則](./media/monitor-alerts-unified/AlertsPreviewCreate.png)

    在幾分鐘之內，警示會開始作用，且先前所述觸發。

使用者可以也會在 [Azure 入口網站中的 Logs Analytics 頁面](../log-analytics/log-analytics-log-search-portals.md#log-analytics-page
)完成分析查詢，然後透過「設定警示」按鈕推送它以建立警示，接著按照上述教學中步驟 6 開始往下操作指示。

 ![Log Analytics - 設定警示](./media/monitor-alerts-unified/AlertsAnalyticsCreate.png)

### <a name="view--manage-log-alerts-in-azure-portal"></a>在 Azure 入口網站中檢視和管理記錄警示

1. 在[入口網站](https://portal.azure.com/)中選取 [監視]，然後在 [監視] 區段下選擇 [警示]。  

1. [警示儀表板] 隨即顯示 - 其中所有 Azure 警示 (包括記錄警示) 都會顯示在單一面板上，包括引發記錄警示規則時的每個執行個體。 若要深入了解，請參閱[警示管理](https://aka.ms/managealertinstances)。
    > [!NOTE]
    > 記錄警示規則由使用者提供的自訂查詢邏輯所組成，因此沒有已解決的狀態。 所以只要符合記錄警示規則中所指定的條件時，就會引發它。 


1. 在頂端的工具列上選取 [管理規則]，即可瀏覽到規則管理區段 - 其中會列出所有已建立的警示規則 (包括已停用的警示)。
    ![管理警示規則](./media/monitoring-activity-log-alerts-new-experience/manage-alert-rules.png)

## <a name="managing-log-alerts-using-azure-resource-template"></a>使用 Azure 資源範本管理記錄警示
目前可以使用兩個不同的資源範本建立記錄警示，取決於警示的分析平台，也就是 Log Analytics 或 Application Insights。

因此下一節提供如何在兩個分析平台上為記錄警示使用資源範本的詳細資料。

### <a name="azure-resource-template-for-log-analytics"></a>適用於 Log Analytics 的 Azure 資源範本
Log Analytics 的記錄警示是由警示規則所建立，這些規則會定期執行已儲存的搜尋。 如果查詢的結果符合指定的準則，就會建立警示記錄，並執行一或多個動作。 

您可以在文件的 Log Analytics 一節中，取得記錄分析儲存的搜尋和記錄分析警示資源範本。 若要深入了解，請參閱[新增 Log Analytics 的已儲存搜尋和警示](../operations-management-suite/operations-management-suite-solutions-resources-searches-alerts.md)；其中包含說明範例與結構描述詳細資料。

### <a name="azure-resource-template-for-application-insights"></a>適用於 Application Insights 的 Azure 資源範本
Application Insights 資源的記錄警示有一個類型是 `Microsoft.Insights/scheduledQueryRules/`。 如需有關此資源類型的詳細資訊，請參閱 [Azure 監視器 - 排程的查詢規則 API 參考](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/)。

以下結構用於根據資源範本[建立排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/createorupdate)，其中使用資料集範例作為變數。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0", 
    "parameters": {      
    },   
    "variables": {
    "alertLocation": "southcentralus",
    "alertName": "samplelogalert",
    "alertTag": "hidden-link:/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/components/sampleAIapplication",
    "alertDesription": "Sample log search alert",
    "alertStatus": "true",
    "alertSource":{
        "Query":"requests",
        "SourceId": "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/components/sampleAIapplication",
        "Type":"ResultCount"
         },
     "alertSchedule":{
         "Frequency": 15,
         "Time": 60
         },
     "alertActions":{
         "SeverityLevel": "4"
         },
      "alertTrigger":{
        "Operator":"GreaterThan",
        "Threshold":"1"
         },
       "actionGrp":{
        "ActionGroup": "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/actiongroups/sampleAG",
        "Subject": "Customized Email Header",
        "Webhook": "{ \"alertname\":\"#alertrulename\", \"IncludeSearchResults\":true }"           
         }
  },
  "resources":[ {
    "name":"[variables('alertName')]",
    "type":"Microsoft.Insights/scheduledQueryRules",
    "apiVersion": "2018-04-16",
    "location": "[variables('alertLocation')]",
    "tags":{"[variables('alertTag')]": "Resource"},
    "properties":{
       "description": "[variables('alertDesription')]",
       "enabled": "[variables('alertStatus')]",
       "source": {
           "query": "[variables('alertSource').Query]",
           "dataSourceId": "[variables('alertSource').SourceId]",
           "queryType":"[variables('alertSource').Type]"
       },
      "schedule":{
           "frequencyInMinutes": "[variables('alertSchedule').Frequency]",
           "timeWindowInMinutes": "[variables('alertSchedule').Time]"    
       },
      "action":{
           "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.AlertingAction",
           "severity":"[variables('alertActions').SeverityLevel]",
           "aznsAction":{
               "actionGroup":"[array(variables('actionGrp').ActionGroup)]",
               "emailSubject":"[variables('actionGrp').Subject]",
               "customWebhookPayload":"[variables('actionGrp').Webhook]"
           },
       "trigger":{
               "thresholdOperator":"[variables('alertTrigger').Operator]",
               "threshold":"[variables('alertTrigger').Threshold]"
           }
       }
     }
   }
 ]
}
```
> [!IMPORTANT]
> [排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/) API 呼叫或資源範本中，一定會使用具有目標資源隱藏連結的標籤欄位。 

基於本逐步解說的目的，上述 json 範例可儲存為 (假設) sampleScheduledQueryRule.json，而且可以使用 [Azure 入口網站中的 Azure Resource Manager](../azure-resource-manager/resource-group-template-deploy-portal.md#deploy-resources-from-custom-template) 進行部署。


## <a name="managing-log-alerts-using-powershell-cli-or-api"></a>使用 PowerShell、CLI 或 API 管理記錄警示
目前可以使用兩個不同的與 Resource Manager 相容的 API 建立記錄警示，取決於警示的分析平台，也就是 Log Analytics 或 Application Insights。

因此下一節提供如何在兩個分析平台上為記錄警示透過 Powershell 或 CLI 使用 API 的詳細資料。

### <a name="powershell-cli-or-api-for-log-analytics"></a>適用於 Log Analytics 的 PowerShell、CLI 或 API
Log Analytics 警示 REST API 是 RESTful，可透過 Azure Resource Manager REST API 來存取。 因此您可以從 PowerShell 命令列存取 API，API 會以 JSON 格式向您輸出搜尋結果，讓您以程式設計方式透過許多不同的方法使用結果。

深入了解[使用 REST API 建立及管理 Log Analytics 中的警示規則](../log-analytics/log-analytics-api-alerts.md)；包括從 Powershell 存取 API 的範例。

### <a name="powershell-cli-or-api-for-application-insights"></a>適用於 Application Insights 的 PowerShell、CLI 或 API
[Azure 監視器 - 已排程的查詢規則 API](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/) 是 REST API，並且可與 Azure Resource Manager REST API 完全相容。 因此它可以透過 Powershell 和 Azure CLI 搭配資源管理員 Cmdlet 來使用。

以下說明透過 Azure Resource Manager PowerShell Cmdlet 使用上述[資源範本一節](#azure-resource-template-for-application-insights)中資源範本範例 (sampleScheduledQueryRule.json) 的方式：
```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile "D:\Azure\Templates\sampleScheduledQueryRule.json"
```
以下說明透過 Azure CLI 中的 Azure Resource Manager 命令使用上述[資源範本一節](#azure-resource-template-for-application-insights)中資源範本範例 (sampleScheduledQueryRule.json) 的方式：

```azurecli
az group deployment create --resource-group myRG --template-file sampleScheduledQueryRule.json
```
成功完成作業後，系統會傳回 201，說明新警示規則的建立，或是傳回 200，表示現有的警示規則已修改。


  
## <a name="next-steps"></a>後續步驟

* 了解 [Azure 警示中的記錄警示](monitor-alerts-unified-log.md)
* 了解 [Webhook 動作記錄警示](monitor-alerts-unified-log-webhook.md)
* 深入了解 [Application Insights](../application-insights/app-insights-analytics.md)
* 深入了解 [Log Analytics](../log-analytics/log-analytics-overview.md)。 

