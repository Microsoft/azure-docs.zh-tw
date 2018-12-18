---
title: 了解 Azure 中的自動調整設定
description: 自動調整規模設定及其運作方式的詳細解說。 適用於虛擬機器、雲端服務、Web Apps
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 12/18/2017
ms.author: ancav
ms.component: autoscale
ms.openlocfilehash: 982bc43fd86a808da07833d77bde17e17789b2d6
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264991"
---
# <a name="understand-autoscale-settings"></a>了解自動調整設定
自動調整設定可協助您確保執行中的資源數量正確，能夠處理變動的應用程式負載。 您可以設定根據指出負載或效能的計量，或是在排定的日期和時間，觸發自動調整設定。 本文將詳細探討自動調整設定的剖析。 本文開頭是設定的結構描述和屬性，然後逐步說明可以設定的各種設定檔類型。 最後，本文會討論 Azure 中的自動調整功能如何評估在任何指定的時間要執行哪個設定檔。

## <a name="autoscale-setting-schema"></a>自動調整規模設定結構描述
為了說明自動調整設定結構描述，將會使用以下自動調整設定。 請務必注意，此自動調整設定具有：
- 一個設定檔。 
- 此設定檔中有兩個計量規則；一個用於相應放大，一個用於相應縮小。
  - 當虛擬機器擴展集的平均「百分比 CPU」計量在過去 10 分鐘大於 85% 時，就會觸發相應放大規則。
  - 當虛擬機器擴展集的平均值在過去 1 分鐘小於 60% 時，就會觸發相應縮小規則。

> [!NOTE]
> 一項設定可以有多個設定檔。 若要深入了解，請參閱[設定檔](#autoscale-profiles)一節。 一個設定檔也可以定義多個相應放大規則和相應縮小規則。 若要查看這些規則的評估方式，請參閱[評估](#autoscale-evaluation)一節。

```JSON
{
  "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/autoscalesettings/setting1",
  "name": "setting1",
  "type": "Microsoft.Insights/autoscaleSettings",
  "location": "East US",
  "properties": {
    "enabled": true,
    "targetResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
    "profiles": [
      {
        "name": "mainProfile",
        "capacity": {
          "minimum": "1",
          "maximum": "4",
          "default": "1"
        },
        "rules": [
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
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
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
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
      }
    ]
  }
}
```

| 區段 | 元素名稱 | 說明 |
| --- | --- | --- |
| 設定 | ID | 自動調整規模設定的資源識別碼。 自動調整規模設定是 Azure Resource Manager 資源。 |
| 設定 | name | 自動調整規模設定名稱。 |
| 設定 | location | 自動調整規模設定的位置。 此位置可以與要調整規模的資源位置不同。 |
| properties | targetResourceUri | 要調整規模之資源的資源識別碼。 每個資源都只能有一個自動調整規模設定。 |
| properties | 設定檔 | 自動調整規模設定是由一或多個設定檔所組成。 每次自動調整規模引擎執行時，都會執行一個設定檔。 |
| 設定檔 | name | 設定檔的名稱。 您可以選擇任何有助於識別設定檔的名稱。 |
| 設定檔 | Capacity.maximum | 允許的最大容量。 執行此設定檔時，可確保自動調整不會將資源的規模調整到大於此數字。 |
| 設定檔 | Capacity.minimum | 允許的最小容量。 執行此設定檔時，可確保自動調整規模不會將資源的規模調整到小於此數字。 |
| 設定檔 | Capacity.default | 如果在讀取資源計量 (在此案例中為 “vmss1” 的 CPU) 時發生問題，而目前的容量低於預設值，則自動調整會相應放大為預設值。 這是為了確保資源的可用性。 如果目前的容量已超過預設容量，自動調整不會進行相應縮小。 |
| 設定檔 | 規則 | 自動調整會使用設定檔中的規則，在最大容量與最小容量之間進行調整。 一個設定檔中可以有多個規則。 通常有兩個規則：一個用於決定何時要相應放大，另一個用於決定何時要相應縮小。 |
| 規則 | metricTrigger | 定義規則的計量條件。 |
| metricTrigger | metricName | 計量的名稱。 |
| metricTrigger |  metricResourceUri | 發出計量之資源的資源識別碼。 在大多數情況下，會與要調整規模的資源相同。 在某些情況下，則可能不同。 例如，您可以根據儲存體佇列中的訊息數目來調整虛擬機器擴展集的規模。 |
| metricTrigger | timeGrain | 計量取樣持續時間。 例如 **TimeGrain = “PT1M”** 表示應該使用 statistic 元素中所指定的彙總方法，每分鐘彙總一次計量。 |
| metricTrigger | statistic | timeGrain 期間內的彙總方法。 例如 **statistic = “Average”** 和 **timeGrain = “PT1M”** 表示應該每分鐘計算一次計量平均值來彙總計量。 此屬性會指定進行計量取樣的方式。 |
| metricTrigger | timeWindow | 回顧計量的時間長度。 例如 **timeWindow = “PT10M”** 表示每次執行自動調整時，都會查詢過去 10 分鐘的計量。 時間範圍可讓您的計量正規化，而避免對暫時性尖峰做出反應。 |
| metricTrigger | timeAggregation | 用來彙總所取樣計量的彙總方法。 例如 **TimeAggregation = “Average”** 應該會透過計算平均值來彙總所取樣的計量。 在前述案例中會採用 10 個 1 分鐘樣本，然後計算其平均值。 |
| 規則 | scaleAction | 觸發 metricTrigger 時要採取的動作。 |
| scaleAction | direction | "Increase" 用於相應放大，或 "Decrease" 用於相應縮小。|
| scaleAction | value | 要增加或減少多少資源容量。 |
| scaleAction | cooldown | 在進行調整作業之後、再次調整之前，所要等待的時間長度。 例如，如果 **cooldown = “PT10M”**，則自動調整在接下來 10 分鐘內不會再次嘗試進行調整。 cooldown 是用來在新增或移除執行個體之後，讓計量穩定。 |

## <a name="autoscale-profiles"></a>自動調整規模設定檔

自動調整規模設定檔有三種類型：

- **一般設定檔：** 最常見的設定檔。 如果您不需要根據星期幾或特定日子調整資源規模，您可以使用一般設定檔。 接著，可以為此設定檔設定計量規則，以指定何時要相應放大及何時要相應縮小。 您應該只定義一個一般設定檔。

    本文中稍早使用的範例設定檔是一般設定檔範例。 請注意，您也可以將設定檔設定成調整至資源的靜態執行個體計數。

- **固定日期設定檔：** 這個設定檔用於特殊案例。 例如，假設您有一個在 2017 年 12 月 26 日 (PST) 發生的重要事件。 您希望當天資源的最小和最大容量不同，但仍會依據相同的計量進行調整。 在此情況下，您應該在設定的設定檔清單中新增一個固定日期設定檔。 此設定檔會設定為只在事件當天執行。 在任何其他日子，自動調整會使用一般設定檔。

    ``` JSON
    "profiles": [{
    "name": " regularProfile",
    "capacity": {
    ...
    },
    "rules": [{
    ...
    },
    {
    ...
    }]
    },
    {
    "name": "eventProfile",
    "capacity": {
    ...
    },
    "rules": [{
    ...
    }, {
    ...
    }],
    "fixedDate": {
        "timeZone": "Pacific Standard Time",
               "start": "2017-12-26T00:00:00",
               "end": "2017-12-26T23:59:00"
    }}
    ]
    ```
    
- **週期設定檔：** 此類型的設定檔可讓您確保在特定的星期幾一律使用此設定檔。 週期設定檔只有開始時間。 它們會一直執行，直到所設定的下一個週期設定檔或固定日期設定檔開始為止。 如果自動調整設定只有一個週期設定檔，則即使在相同設定中已定義一般設定檔，也會執行該週期設定檔。 下列兩個範例說明如何使用此設定檔：

    **範例 1：工作日與週末**
    
    假設在週末，您希望最大容量為 4。 在工作日，預期會有更多負載，所以您希望最大容量為 10。 在此情況下，您的設定會包含兩個週期設定檔，一個用來在週末執行，另一個在工作日執行。
    設定如下所示：

    ``` JSON
    "profiles": [
    {
    "name": "weekdayProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }}
    },
    {
    "name": "weekendProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Saturday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```

    上述設定顯示每個週期設定檔都有一個排程。 此排程決定設定檔何時開始執行。 此設定檔會在該執行另一個設定檔時停止。

    例如，在上述設定中，“weekdayProfile” 已設定為在星期一上午 12 點開始。 這表示此設定檔會在星期一上午 12 點開始執行。 它會持續執行到星球六上午 12 點為止，亦即已排定的 “weekendProfile” 開始執行時間。

    **範例 2：上班時間**
    
    例如，假設您想要有一個適用於上班時間 (上午 9 點到下午 5 點) 的計量閾值，以及另一個適用於所有其他時間的計量閾值。 此設定看起來會像這樣：
    
    ``` JSON
    "profiles": [
    {
    "name": "businessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                9
            ],
            "minutes": [
                0
            ]
        }
    }
    },
    {
    "name": "nonBusinessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                17
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```
    
    前述設定顯示 “businessHoursProfile” 會在星期一上午 9 點開始執行，並持續執行到下午 5 點。 這是 “nonBusinessHoursProfile” 開始執行的時間。 “nonBusinessHoursProfile” 會執行到星期二上午 9 點為止，然後 “businessHoursProfile” 又接著執行。 這會重複直到星期五下午 5 點為止。 屆時 “nonBusinessHoursProfile” 會一直執行到星期一上午 9 點。
    
> [!Note]
> Azure 入口網站中的自動調整使用者介面會對週期設定檔強制執行結束時間，然後在週期設定檔之間開始執行自動調整設定的預設設定檔。
    
## <a name="autoscale-evaluation"></a>自動調整評估
由於自動調整設定可以有多個設定檔，且每個設定檔可以有多個計量規則，因此了解如何評估自動調整設定相當重要。 每次執行自動調整作業時，就會開始選擇適用的設定檔。 自動調整會評估最小值和最大值，以及設定檔中的任何計量規則，然後決定是否需要調整動作。

### <a name="which-profile-will-autoscale-pick"></a>自動調整規模會挑選哪個設定檔？

自動調整會使用下列順序來挑選設定檔：
1. 首先會尋找任何設定為在現在執行的固定日期設定檔。 如果有，自動調整就會執行它。 如果有多個應該執行的固定日期設定檔，自動調整會選取第一個。
2. 如果沒有任何固定日期設定檔，自動調整就會尋找週期設定檔。 如果找到週期設定檔，就會執行它。
3. 如果沒有任何固定日期或週期設定檔，則自動調整會執行一般設定檔。

### <a name="how-does-autoscale-evaluate-multiple-rules"></a>自動調整規模如何評估多個規則？

在自動調整判斷應該執行哪個設定檔之後，它會評估設定檔中的所有相應放大規則 (**direction = “Increase”** 的規則)。

如果觸發的相應放大規則有一或多個，自動調整就會計算出由這每一個規則的 **scaleAction** 所決定的新容量。 接著，它會相應放大至這些容量的最大值，以確保服務可用性。

例如，假設有目前容量為 10 的虛擬機器擴展集。 相應放大規則有兩個：一個會增加 10% 的容量，而另一個則會使容量的計數增加 3。 第一個規則會產生新容量 11，而第二個規則則會產生容量 13。 為了確保服務可用性，自動調整會選擇產生最大容量的動作，因此會選擇第二個規則。

如果未觸發任何相應放大規則，自動調整就會評估所有相應縮小規則 (**direction = “Decrease”** 的規則)。 如果觸發所有相應縮小規則，則自動調整規模將只會執行相應縮小動作。

自動調整會計算出由這每一個規則的 **scaleAction** 所決定的新容量。 接著，它會選擇產生這些容量最大值的規模調整動作，以確保服務可用性。

例如，假設有目前容量為 10 的虛擬機器擴展集。 相應縮小規則有兩個：一個會減少 50% 的容量，而另一個則會使容量的計數減少 3。 第一個規則會產生新容量 5，而第二個規則則會產生容量 7。 為了確保服務可用性，自動調整會選擇產生最大容量的動作，因此會選擇第二個規則。

## <a name="next-steps"></a>後續步驟
請參考下列各項，以深入了解自動調整︰

* [自動調整規模的概觀](monitoring-overview-autoscale.md)
* [Azure 監視器自動調整的常用度量](insights-autoscale-common-metrics.md)
* [Azure 監視器自動調整的最佳作法](insights-autoscale-best-practices.md)
* [使用自動調整動作傳送電子郵件和 Webhook 警示通知](insights-autoscale-to-webhook-email.md)
* [自動調整 REST API](https://msdn.microsoft.com/library/dn931953.aspx)
