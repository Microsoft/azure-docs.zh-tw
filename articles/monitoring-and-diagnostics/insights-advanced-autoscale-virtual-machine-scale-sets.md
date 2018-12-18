---
title: 使用 Azure 虛擬機器的進階自動調整
description: 使用 Resource Manager 和 VM 擴展集，搭配多個規則與設定檔，以傳送電子郵件並使用調整動作來呼叫 Webhook URL。
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 02/22/2016
ms.author: ancav
ms.component: autoscale
ms.openlocfilehash: 78e3bec0d00336ce7cedc1434bf6ad7c65435969
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978177"
---
# <a name="advanced-autoscale-configuration-using-resource-manager-templates-for-vm-scale-sets"></a>針對 VM 擴展集使用 Resource Manager 範本的進階自動調整設定
您可以根據效能標準臨界值、循環排程或特定日期，針對虛擬機器擴展集進行相應縮小和放大。 您也可以針對調整動作設定電子郵件和 webhook 通知。 本逐步解說會示範在 VM 擴展集上使用 Resource Manager 範本設定所有這些物件。

> [!NOTE]
> 雖然此逐步解說會說明 VM 擴展集的各步驟，相同的資訊也適用於自動調整[雲端服務](https://azure.microsoft.com/services/cloud-services/)、[App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) 與 [API 管理服務](https://docs.microsoft.com/azure/api-management/api-management-key-concepts) 如需 VM 擴展集上根據簡單效能計量 (例如 CPU) 的簡單相應縮小/放大設定，請參閱 [Linux](../virtual-machine-scale-sets/virtual-machine-scale-sets-linux-autoscale.md) 與 [Windows](../virtual-machine-scale-sets/virtual-machine-scale-sets-windows-autoscale.md) 文件
>
>

## <a name="walkthrough"></a>逐步介紹
在本逐步解說中，我們會使用 [Azure 資源總管](https://resources.azure.com/)來設定並更新擴展集的自動調整設定。 Azure 資源總管是透過 Resource Manager 範本輕鬆管理 Azure 資源的方式。 如果您還不熟悉 Azure 資源總管、工具，請參閱[本簡介](https://azure.microsoft.com/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/)。

1. 使用基本自動調整設定部署新的擴展集。 這篇文章會使用 Azure 快速入門資源庫中擁有基本自動調整範本的 Windows 擴展集設定。 Linux 擴展集的使用方式相同。
2. 建立擴展集之後，請透過 Azure 資源總管瀏覽到擴展集資源。 您會在 Microsoft.Insights 節點下看見下列內容。

    ![Azure Explorer](./media/insights-advanced-autoscale-vmss/azure_explorer_navigate.png)

    範本的執行已建立了具有名稱為 **'autoscalewad'** 的預設自動調整設定。 您可以在右手邊檢視此自動調整設定的完整定義。 在此情況下，預設自動調整設定具有以 CPU% 為基礎的相應放大和相應縮小規則。  

3. 您現在可以根據排程或特定需求新增更多設定檔和規則。 我們會建立具有三個設定檔的自動調整設定。 若要了解自動調整中的設定檔和規則，請檢閱[自動調整最佳做法](insights-autoscale-best-practices.md)。  

    | 設定檔與規則 | 說明 |
    |--- | --- |
    | **設定檔** |**以效能/計量為基礎** |
    | 規則 |服務匯流排佇列訊息計數 > x |
    | 規則 |服務匯流排佇列訊息計數 < y |
    | 規則 |CPU% > n |
    | 規則 |CPU% < p |
    | **設定檔** |**工作日早上時間 (無規則)** |
    | **設定檔** |**產品發行日 (無規則)** |

4. 以下是我們用於此逐步解說的虛構調整案例。

    * **以負載為基礎** - 我想要根據裝載於擴展集的應用程式的負載來相應放大或縮小。*
    * **訊息佇列大小** - 我使用服務匯流排佇列來存放傳入應用程式的訊息。 我使用佇列的訊息計數和 CPU%，並設定預設設定檔，在訊息計數或 CPU 達到臨界值時觸發調整動作。
    * **每週和每日時間** - 我想要一個以每週一次「當天時間」為基礎的設定檔，稱為「工作日早上時間」。 根據歷史資料，我明白這段期間，最好要有一定數量的 VM 執行個體來處理應用程式的負載。*
    * **特殊日期** - 我已新增「產品發行日」設定檔。 我提前針對特定日期做好規劃，讓應用程式能夠因應由於行銷發表或當我們將新產品置入應用程式時所引起的負載。
    * *最後兩個設定檔也可以具有以其他效能標準為基礎的規則。在此案例中，我決定不那麼做，而是依賴以預設效能標準為基礎的規則。針對週期性和日期式設定檔的規則為選擇性。*

    自動調整引擎針對設定檔和規則的優先順序，也已在[自動調整最佳做法](insights-autoscale-best-practices.md)一文中說明。
    如需自動調整的常見計量清單，請參閱[自動調整的常用計量](insights-autoscale-common-metrics.md)

5. 請確定您已處於資源總管的**讀寫**模式

    ![Autoscalewad, default autoscale setting, 自動調整, 預設自動調整設定](./media/insights-advanced-autoscale-vmss/autoscalewad.png)

6. 按一下 [編輯]。 使用下列組態**取代**自動調整設定中的 'profiles' 元素：

    ![設定檔](./media/insights-advanced-autoscale-vmss/profiles.png)

    ```
    {
            "name": "Perf_Based_Scale",
            "capacity": {
              "minimum": "2",
              "maximum": "12",
              "default": "2"
            },
            "rules": [
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 10
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 3
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 85
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 60
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              }
            ]
          },
          {
            "name": "Weekday_Morning_Hours_Scale",
            "capacity": {
              "minimum": "4",
              "maximum": "12",
              "default": "4"
            },
            "rules": [],
            "recurrence": {
              "frequency": "Week",
              "schedule": {
                "timeZone": "Pacific Standard Time",
                "days": [
                  "Monday",
                  "Tuesday",
                  "Wednesday",
                  "Thursday",
                  "Friday"
                ],
                "hours": [
                  6
                ],
                "minutes": [
                  0
                ]
              }
            }
          },
          {
            "name": "Product_Launch_Day",
            "capacity": {
              "minimum": "6",
              "maximum": "20",
              "default": "6"
            },
            "rules": [],
            "fixedDate": {
              "timeZone": "Pacific Standard Time",
              "start": "2016-06-20T00:06:00Z",
              "end": "2016-06-21T23:59:00Z"
            }
          }
    ```
    如需支援的欄位和其值，請參閱[自動調整 REST API 文件](https://msdn.microsoft.com/library/azure/dn931928.aspx)。 現在，您的自動調整設定已包含前面所述的三個設定檔。

7. 最後，我們來看一下自動調整的**通知**區段。 自動調整通知能讓您在相應放大或縮小動作成功觸發時執行三件事。
   - 通知系統管理員和共同管理員有關您訂用帳戶的狀況
   - 傳送電子郵件給一組使用者
   - 觸發 webhook 呼叫。 引發時，此 webhook 會傳送有關自動調整狀態和擴展集資源的中繼資料。 若要深入了解自動調整 webhook 的承載，請參閱[針對自動調整設定 Webhook 與電子郵件通知](insights-autoscale-to-webhook-email.md)。

   新增下列內容到自動調整設定，並取代其值為 null 的 **notification** 元素

   ```
   "notifications": [
      {
        "operation": "Scale",
        "email": {
          "sendToSubscriptionAdministrator": true,
          "sendToSubscriptionCoAdministrators": false,
          "customEmails": [
              "user1@mycompany.com",
              "user2@mycompany.com"
              ]
        },
        "webhooks": [
          {
            "serviceUri": "https://foo.webhook.example.com?token=abcd1234",
            "properties": {
              "optional_key1": "optional_value1",
              "optional_key2": "optional_value2"
            }
          }
        ]
      }
    ]

   ```

   按一下資源總管中的 [放置] 按鈕以更新自動調整設定。

您已將 VM 擴展集上的自動調整設定更新為包含多個調整設定檔和調整通知。

## <a name="next-steps"></a>後續步驟
請使用下列連結來深入了解自動調整。

[針對使用虛擬機器擴展集的自動調整進行疑難排解](../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)

[自動調整的常用計量](insights-autoscale-common-metrics.md)

[Azure 自動調整的最佳做法](insights-autoscale-best-practices.md)

[使用 PowerShell 管理自動調整](insights-powershell-samples.md#create-and-manage-autoscale-settings)

[使用 CLI 管理自動調整](insights-cli-samples.md#autoscale)

[針對自動調整設定 Webhook 與電子郵件通知](insights-autoscale-to-webhook-email.md)
