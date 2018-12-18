---
title: 將 Azure 活動記錄檔串流至事件中樞
description: 了解如何將 Azure 活動記錄檔串流至事件中樞。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 07/25/2018
ms.author: johnkem
ms.component: activitylog
ms.openlocfilehash: 7a5372174fcc7cd9552c00c9d283772c9863b815
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39257993"
---
# <a name="stream-the-azure-activity-log-to-event-hubs"></a>將 Azure 活動記錄檔串流至事件中樞
您可以藉由下列任一方式，將 [Azure 活動記錄](monitoring-overview-activity-logs.md)近乎即時地串流至任何應用程式：

* 使用入口網站中內建的 [匯出] 選項
* 透過 Azure PowerShell Cmdlet 或 Azure CLI 在記錄設定檔中啟用「Azure 服務匯流排」規則識別碼

## <a name="what-you-can-do-with-the-activity-log-and-event-hubs"></a>您可以使用活動記錄檔與事件中樞執行的項目
以下是兩種您可以將串流功能用於「活動記錄」的方式：

* **串流至協力廠商記錄與遙測系統** – 隨著時間進展，「Azure 事件中樞」串流會成為將「活動記錄」輸送到協力廠商 SIEM 與記錄分析解決方案的機制。
* **建置自訂遙測與記錄平台** – 如果您已經有自訂建置的遙測平台或正考慮建置一個，「事件中樞」所具備的高度可調整發佈訂閱特質將可讓您靈活擷取活動記錄。 如需詳細資訊，請觀看 [Dan Rosanova 有關在全球級遙測平台中使用事件中樞的影片](https://azure.microsoft.com/documentation/videos/build-2015-designing-and-sizing-a-global-scale-telemetry-platform-on-azure-event-Hubs/) \(英文\)。

## <a name="enable-streaming-of-the-activity-log"></a>啟用活動記錄檔的串流功能
您可以以程式控制的方式，或透過入口網站來啟用活動記錄檔的串流功能。 不論使用哪一種方式，您都需挑選一個「事件中樞」命名空間，以及該命名空間的共用存取原則。 在第一個新的「活動記錄」事件發生時，會在該命名空間中建立名稱為 insights-logs-operationallogs 的事件中樞。 

如果您沒有「事件中樞」命名空間，就必須先建立一個。 如果您先前已將「活動記錄」事件串流至此「事件中樞」命名空間，則會重複使用先前建立的事件中樞。 

共用存取原則會定義串流機制具有的權限。 目前，串流至「事件中樞」需要**管理**、**傳送**及**接聽**權限。 您可以在 Azure 入口網站中您「事件中樞」命名空間的 [設定] 索引標籤下，為「事件中樞」命名空間建立或修改共用存取原則。 

若要更新「活動記錄」記錄設定檔以包含串流，進行變更的使用者必須擁有該「事件中樞」授權規則的 ListKey 權限。 事件中樞命名空間不一定要和發出記錄檔的訂用帳戶屬於相同的訂用帳戶，只要進行設定的使用者有這兩個訂用帳戶的適當 RBAC 存取權，而且這兩個訂用帳戶都屬於同一個 ADD 租用戶。

### <a name="via-the-azure-portal"></a>透過 Azure 入口網站
1. 藉由使用入口網站左側的 [所有服務] 搜尋，瀏覽至 [活動記錄] 區段。
   
   ![從入口網站的服務清單中選取 [活動記錄]](./media/monitoring-stream-activity-logs-event-hubs/activity.png)
2. 選取記錄檔頂端的 [匯出] 按鈕。
   
   ![入口網站中的 [匯出] 按鈕](./media/monitoring-stream-activity-logs-event-hubs/export.png)

   請注意，您在上一個檢視中檢視「活動記錄」時所套用的篩選設定，對您的匯出設定沒有任何影響。 這些僅用來篩選您在入口網站中瀏覽「活動記錄」時所看到的內容。
3. 在出現的區段中，選取 [所有區域]。 請勿選取特定的區域。
   
   ![[匯出] 區段](./media/monitoring-stream-activity-logs-event-hubs/export-audit.png)

   > [!WARNING]  
   > 如果您選取 [所有區域] 以外的選項，則會遺漏您預期接收的重要事件。 「活動記錄」是一個全域 (非區域性) 記錄，因此大多數事件都沒有關聯的區域。 
   >

4. 按一下 [Azure 事件中樞]  選項，然後選取中樞命名空間來將記錄檔傳送給它，然後按一下 [確定]。
5. 選取 [儲存] 以儲存這些設定。 設定會立即套用至您的訂用帳戶。
6. 如果您有數個訂用帳戶，請重複執行此動作，並將所有資料傳送給相同的事件中樞。

### <a name="via-powershell-cmdlets"></a>透過 PowerShell Cmdlet
如果記錄設定檔已經存在，您必須先移除現有的記錄設定檔，然後再建立新的記錄設定檔。

1. 使用 `Get-AzureRmLogProfile` 來識別記錄設定檔是否存在。  如果記錄設定檔存在，請找出 name 屬性。
2. 使用 `Remove-AzureRmLogProfile` 透過 name 屬性中的值來移除記錄設定檔。

    ```powershell
    # For example, if the log profile name is 'default'
    Remove-AzureRmLogProfile -Name "default"
    ```
3. 使用 `Add-AzureRmLogProfile` 來建立新的記錄設定檔：

   ```powershell
   # Settings needed for the new log profile
   $logProfileName = "default"
   $locations = (Get-AzureRmLocation).Location
   $locations += "global"
   $subscriptionId = "<your Azure subscription Id>"
   $resourceGroupName = "<resource group name your event hub belongs to>"
   $eventHubNamespace = "<event hub namespace>"

   # Build the service bus rule Id from the settings above
   $serviceBusRuleId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.EventHub/namespaces/$eventHubNamespace/authorizationrules/RootManageSharedAccessKey"

   Add-AzureRmLogProfile -Name $logProfileName -Location $locations -ServiceBusRuleId $serviceBusRuleId
   ```

### <a name="via-azure-cli"></a>透過 Azure CLI
如果記錄設定檔已經存在，您必須先移除現有的記錄設定檔，然後再建立新的記錄設定檔。

1. 使用 `az monitor log-profiles list` 來識別記錄設定檔是否存在。
2. 使用 `az monitor log-profiles delete --name "<log profile name>` 透過 name 屬性中的值來移除記錄設定檔。
3. 使用 `az monitor log-profiles create` 來建立新的記錄設定檔：

   ```azurecli-interactive
   az monitor log-profiles create --name "default" --location null --locations "global" "eastus" "westus" --categories "Delete" "Write" "Action"  --enabled false --days 0 --service-bus-rule-id "/subscriptions/<YOUR SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUB NAME SPACE>/authorizationrules/RootManageSharedAccessKey"
   ```

## <a name="consume-the-log-data-from-event-hubs"></a>從事件中樞取用記錄資料
如需了解「活動記錄」的結構描述，請參閱[使用 Azure 活動記錄監視訂用帳戶活動](monitoring-overview-activity-logs.md)。 每個事件都位於名為 *records* 的 JSON Blob 陣列中。

## <a name="next-steps"></a>後續步驟
* [將活動記錄檔封存至儲存體帳戶](monitoring-archive-activity-log.md)
* [閱讀 Azure 活動記錄檔的概觀](monitoring-overview-activity-logs.md)
* [根據活動記錄檔事件設定警示](insights-auditlog-to-webhook-email.md)

