---
title: Azure 防火牆記錄概觀
description: 本文將概略說明 Azure 防火牆的診斷記錄。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 9/24/2018
ms.author: victorh
ms.openlocfilehash: 6d8d0b2dc0f6baf48d2aacb9c7203937aef08d15
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957792"
---
# <a name="azure-firewall-logs"></a>Azure 防火牆記錄

您可以使用防火牆記錄來監視 Azure 防火牆。 您也可以使用活動記錄來稽核 Azure 防火牆資源上的作業。

您可以透過入口網站存取其中一些記錄。 您可以將記錄傳送到 [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md)、儲存體和事件中樞，並在 Log Analytics 中或透過不同的工具 (例如 Excel 和 Power BI) 來分析記錄。

## <a name="diagnostic-logs"></a>診斷記錄檔

 下列診斷記錄可供 Azure 防火牆使用：

* **應用程式規則記錄**

   只有當您針對每個 Azure 防火牆加以啟用時，應用程式規則記錄才會儲存到儲存體帳戶、串流至事件中樞及/或傳送至 Log Analytics。 符合其中一個已設定應用程式規則的每個新連線，都會產生接受/拒絕連線的記錄。 資料會以 JSON 格式記錄下來，如下列範例所示：

   ```
   Category: application rule logs.
   Time: log timestamp.
   Properties: currently contains the full message. 
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
   {
    "category": "AzureFirewallApplicationRule",
    "time": "2018-04-16T23:45:04.8295030Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallApplicationRuleLog",
    "properties": {
        "msg": "HTTPS request from 10.1.0.5:55640 to mydestination.com:443. Action: Allow. Rule Collection: collection1000. Rule: rule1002"
    }
   }
   ```

* **網路規則記錄**

   只有當您針對每個 Azure 防火牆加以啟用時，網路規則記錄才會儲存到儲存體帳戶、串流至事件中樞及/或傳送至 Log Analytics。 符合其中一個已設定網路規則的每個新連線，都會產生接受/拒絕連線的記錄。 資料會以 JSON 格式記錄下來，如下列範例所示：

   ```
   Category: network rule logs.
   Time: log timestamp.
   Properties: currently contains the full message. 
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
  {
    "category": "AzureFirewallNetworkRule",
    "time": "2018-06-14T23:44:11.0590400Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallNetworkRuleLog",
    "properties": {
        "msg": "TCP request from 111.35.136.173:12518 to 13.78.143.217:2323. Action: Deny"
    }
   }

   ```

您有三個選項可用來排序您的記錄：

* **儲存體帳戶**：如果記錄會儲存一段較長的持續期間，並在需要時加以檢閱，則最好針對記錄使用儲存體帳戶。
* **事件中樞**：如果要整合其他安全性資訊和事件管理 (SEIM) 工具以便在資源上取得警示，則事件中樞是絕佳的選項。
* **Log Analytics**：Log Analytics 最適合用來進行應用程式的一般即時監視，或查看趨勢。

## <a name="activity-logs"></a>活動記錄

   預設會收集活動記錄，您可在 Azure 入口網站中檢視它們。

   您可以使用 [Azure 活動記錄](../azure-resource-manager/resource-group-audit.md) (之前稱為「作業記錄和稽核記錄」) 來檢視提交至您 Azure 訂用帳戶的所有作業。


## <a name="next-steps"></a>後續步驟

若要了解如何監視 Azure 防火牆記錄和計量，請參閱[教學課程：監視 Azure 防火牆記錄](tutorial-diagnostics.md)。