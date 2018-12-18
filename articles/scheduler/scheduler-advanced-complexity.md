---
title: 建置進階的作業排程和週期 - Azure 排程器
description: 了解如何在 Azure 排程器中建立作業的進階排程和週期
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam
ms.suite: infrastructure-services
ms.assetid: 5c124986-9f29-4cbc-ad5a-c667b37fbe5a
ms.topic: article
ms.date: 08/18/2016
ms.openlocfilehash: f5a8b929cf5af6e4e43c6003e6b622d04a50b93e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980935"
---
# <a name="build-advanced-schedules-and-recurrences-for-jobs-in-azure-scheduler"></a>在 Azure 排程器中建置作業的進階排程和週期

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 會取代 Azure 排程器，Azure 排程器之後將無法使用。 若要排程作業，請[改為試用 Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md)。 

在 [Azure 排程器](../scheduler/scheduler-intro.md)作業內，排程為決定排程器服務何時及如何執行作業的核心。 您可以使用排程器，為作業設定多個單次和週期性排程。 單次排程只會在指定的時間執行一次，而且基本上是僅執行一次的週期性排程。 週期性排程會以指定的頻率來執行。 由於這種彈性，您可以將排程器用於各種商務案例，例如：

* **定期清除資料**：建立每日作業，以刪除所有超過三個月的推文。

* **封存資料**：建立每月作業，將發票歷程記錄推送到備份服務。

* **要求外部資料**：建立作業，每隔 15 分鐘執行一次，並從 NOAA 提取新氣象報告。

* **處理影像**：建立工作日作業，在離峰時間執行，並使用雲端運算來壓縮當天上傳的影像。

本文說明您可以使用排程器和 [Azure 排程器 REST API](https://docs.microsoft.com/rest/api/schedule) 建立的範例作業，並包含每個排程的 JavaScript 物件標記法 (JSON) 定義。 

## <a name="supported-scenarios"></a>支援的案例

這些範例示範 Azure 排程器支援的各種案例，以及如何為各種不同行為模式建立排程，例如：

* 於特定日期和時間單次執行。
* 執行並重複特定次數。
* 立即執行並重複。
* 執行並從特定時間開始，每隔 *n* 分鐘、小時、天、週或月重複執行一次。
* 執行且每週或每月重複，但僅於當週的特定天，或當月的特定天。
* 執行並針對特定的期間重複多次。 例如，在每月的最後一個星期五和星期一執行，或是在每天的上午 5:15 和下午 5:15 執行。

本文稍後將更詳細地說明這些案例。

<a name="create-scedule"></a>

## <a name="create-schedule-with-rest-api"></a>使用 REST API 建立排程

若要使用 [Azure 排程器 REST API](https://docs.microsoft.com/rest/api/schedule) 建立基本排程，請遵循下列步驟：

1. 使用[註冊操作 - Resource Manager REST API](https://docs.microsoft.com/rest/api/resources/providers#Providers_Register) 來向資源提供者註冊您的 Azure 訂用帳戶。 Azure 排程器服務的提供者名稱是 **Microsoft.Scheduler**。 

1. 在排程器 REST API 中，使用[適用於作業集合的建立或更新操作](https://docs.microsoft.com/rest/api/scheduler/jobcollections#JobCollections_CreateOrUpdate)來建立作業集合。 

1. 使用[適用於作業的建立或更新操作](https://docs.microsoft.com/rest/api/scheduler/jobs/createorupdate)來建立作業。 

## <a name="job-schema-elements"></a>作業結構描述元素

此表格提供您在設定作業的週期和排程時可使用之主要 JSON 元素的高階概觀。 

| 元素 | 必要 | 說明 | 
|---------|----------|-------------|
| **startTime** | 否 | [ISO 8601 格式](http://en.wikipedia.org/wiki/ISO_8601) \(英文\) 的日期時間字串值，會在基本排程中指定作業第一次啟動的時機。 <p>針對複雜的排程，作業一到 **startTime** 就會啟動。 | 
| **recurrence** | 否 | 適用於作業執行時機的週期規則。 **recurrence** 物件支援下列元素：**frequency**、**interval**、**schedule**、**count** 及 **endTime**。 <p>如果您使用 **recurrence** 元素，也必須使用 **frequency** 元素，而其他 **recurrence** 元素均為選擇性的。 |
| **frequency** | 是，當您使用 **recurrence** 時 | 發生次數之間的時間單位，並支援下列值："Minute"、"Hour"、"Day"、"Week"、"Month" 及 "Year" | 
| **interval** | 否 | 正整數，根據 **frequency** 來決定發生次數之間的時間單位數。 <p>例如，如果 **interval** 為 10 且 **frequency** 為 "Week"，則作業每隔 10 週就會重複執行一次。 <p>以下是適用於每個頻率的間隔最大數目： <p>- 18 個月 <br>- 78 週 <br>- 548 天 <br>- 若為小時和分鐘，則範圍是 1 <= <*interval*> <= 1000。 | 
| **schedule** | 否 | 根據分鐘標記、小時標記、星期幾和每月執行日來定義週期的變更 | 
| **count** | 否 | 正整數，指定作業完成之前執行的次數。 <p>例如，若將每日作業的 **count** 設為 7，而且開始日期為星期一，作業就會在星期日完成執行。 如果開始日期已經過去，則會從建立時間計算第一次執行時間。 <p>如果沒有 **endTime** 或 **count**，作業就會無限期執行。 您無法在同一個作業中同時使用**count** 和 **endTime**，但會接受先完成的規則。 | 
| **endTime** | 否 | [ISO 8601 格式](http://en.wikipedia.org/wiki/ISO_8601) \(英文\) 的日期或日期時間字串值，會指定作業停止執行的時機。 您可以將 **endTime** 的值設定為過去的時間。 <p>如果沒有 **endTime** 或 **count**，作業就會無限期執行。 您無法在同一個作業中同時使用**count** 和 **endTime**，但會接受先完成的規則。 |
|||| 

例如，此 JSON 結構描述說明作業的基本排程和週期： 

```json
"properties": {
   "startTime": "2012-08-04T00:00Z", 
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "weekDays": ["Monday", "Wednesday", "Friday"],
         "hours": [10, 22]                      
      },
      "count": 10,       
      "endTime": "2012-11-04"
   },
},
``` 

日期和日期時間值

* 排程器作業中的日期只包含日期並遵循 [ISO 8601 規格](http://en.wikipedia.org/wiki/ISO_8601) \(英文\)。

* 排程器作業中的日期時間包含日期和時間、遵循 [ISO 8601 規格](http://en.wikipedia.org/wiki/ISO_8601) \(英文\)，並在未指定任何 UTC 時差時假設為 UTC。 

如需詳細資訊，請參閱[概念、術語及實體](../scheduler/scheduler-concepts-terms.md)。

<a name="start-time"></a>

## <a name="details-starttime"></a>詳細資料：startTime

下表說明 **startTime** 如何控制作業執行的方式：

| startTime | 無週期 | 週期、無排程 | 週期性有排程 |
|-----------|---------------|-------------------------|--------------------------|
| **没有開始時間** | 立即執行一次。 | 立即執行一次。 根據上次執行時間算出的時間執行後續的執行作業。 | 立即執行一次。 根據週期排程執行後續的執行作業。 | 
| **過去的開始時間** | 立即執行一次。 | 計算開始時間之後的第一個未來執行時間，並在該時間執行。 <p>根據上次執行時間算出的時間執行後續的執行作業。 <p>請參閱這個表格後面的範例。 | 「一到」指定的開始時間就啟動作業。 第一次執行是根據從開始時間計算的排程。 <p>根據週期排程執行後續的執行作業。 | 
| **在未來或目前時間的開始時間** | 在指定的開始時間執行一次。 | 在指定的開始時間執行一次。 <p>根據上次執行時間算出的時間執行後續的執行作業。 | 「一到」指定的開始時間就啟動作業。 第一次發生是根據排程，從開始時間計算。 <p>根據週期排程執行後續的執行作業。 |
||||| 

假設這個範例具備下列條件：過去的開始時間，有週期但無排程。

```json
"properties": {
   "startTime": "2015-04-07T14:00Z", 
   "recurrence": {
      "frequency": "Day",
      "interval": 2
   }
}
```

* 目前的日期和時間是 "2015-04-08 13:00"。

* 開始日期和時間是 "2015年-04-07 14:00"，其在目前的日期和時間之前。

* 週期是每隔兩天。

1. 根據這些條件，第一次執行是在 2015-04-09 的 14:00。 

   排程器會根據開始時間計算執行出現次數、捨棄過去的任何執行個體，並在未來使用下一個執行個體。 
   在此案例中，**startTime** 是在 2015-04-07 下午 2:00，所以下一個執行個體是在該時間的二天後，即 2015-04-09 下午 2:00。

   不論 **startTime** 為 2015-04-05 14:00 或 2015-04-01 14:00，第一次執行都一樣。 第一次執行之後，就會根據排程來計算出後續的執行時間。 
   
1. 執行接著會遵循下列順序： 
   
   1. 2015-04-11 下午 2:00
   1. 2015-04-13 下午 2:00 
   1. 2015-04-15 下午 2:00
   1. 依此類推...

1. 最後，當作業具備排程但未指定小時和分鐘時，這些值會分別預設為第一次執行的小時和分鐘。

<a name="schedule"></a>

## <a name="details-schedule"></a>詳細資料：schedule

您可以使用 **schedule** 來*限制*作業執行的數目。 例如，如果作業的 **frequency** 為 "month" 且具有只在 31 日執行的排程，則該作業只在具有第三十一天的月份執行。

您也可以使用 **schedule** 來*擴充*作業執行的數目。 例如，如果作業的 **frequency** 為 "month" 且具有在當月 1 日和 2 日執行的排程，則該作業會在當月的第一天和第二天執行，而不是一個月只執行一次。

如果您指定了多個 schedule 元素，則評估的順序是從最大到最小：週數、月日、工作日、小時和分鐘。

下表詳細說明 schedule 元素：

| JSON 名稱 | 說明 | 有效值 |
|:--- |:--- |:--- |
| **minutes** |作業在該小時的第幾分鐘執行。 |整數陣列。 |
| **hours** |作業在該日的第幾小時執行。 |整數陣列。 |
| **weekDays** |作業在該週的第幾天執行。 只能搭配 weekly 頻率指定。 |包含任何下列值的陣列 (最大的陣列大小為 7)：<br />- "Monday"<br />- "Tuesday"<br />- "Wednesday"<br />- "Thursday"<br />- "Friday"<br />- "Saturday"<br />- "Sunday"<br /><br />不區分大小寫。 |
| **monthlyOccurrences** |決定將在當月哪幾天執行工作。 只能搭配 monthly 頻率指定。 |**monthlyOccurrences** 物件的陣列：<br /> `{ "day": day, "occurrence": occurrence}`<br /><br /> **day** 是作業在當週的第幾天執行。 例如，*{Sunday}* 是當月的每個星期日。 必要。<br /><br />**occurrence** 是當月期間的第幾天發生。 例如，*{Sunday, -1}* 是當月的最後一個星期日。 選用。 |
| **monthDays** |作業將在當月的哪一日執行。 只能搭配 monthly 頻率指定。 |包含下列值的陣列：<br />- <= -1 且 >= -31 的任何值<br />- >= 1 且 <= 31 的任何值|

## <a name="examples-recurrence-schedules"></a>範例：週期排程

下列範例顯示各種循環排程。 該範例著重於 schedule 物件和其子元素。

排程假設 **interval** 設定為 1\. 此範例也會假設 **schedule** 中值的 **frequency** 值是正確的。 例如，在 **schedule** 中具有 **monthDays** 修改的情況下，您無法將 **frequency** 設為 "day" 。 我們稍早在文章中已描述這些限制。

| 範例 | 說明 |
|:--- |:--- |
| `{"hours":[5]}` |在每天的上午 5 點執行。<br /><br />排程器會將 "hours" 中的每個值與 "minutes" 中的每個值逐一比對，以建立一個清單，列出將執行工作的所有時間。 |
| `{"minutes":[15], "hours":[5]}` |在每天的上午 5:15 執行。 |
| `{"minutes":[15], "hours":[5,17]}` |在每天的上午 5:15 和下午 5:15 執行。 |
| `{"minutes":[15,45], "hours":[5,17]}` |在每天的上午 5:15、上午 5:45、下午 5:15 及下午 5:45 執行。 |
| `{"minutes":[0,15,30,45]}` |每隔 15 分鐘執行一次。 |
| `{hours":[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]}` |每小時執行一次。<br /><br />這項工作每小時執行一次。 分鐘是由 **startTime** 的值控制 (若有指定)。 如果未指定 **startTime** 值，則分鐘是由建立時間控制。 比方說，如果開始時間或建立時間 (無論適用哪一個) 為下午 12:25，則工作將在 00:25、01:25、02:25、…、23:25 執行。<br /><br />該排程相當於具有 **frequency** 為 "hour"、**interval** 為 1 且沒有 **schedule** 值的作業。 差別在於，您可以使用此排程搭配不同的 **frequency** 和 **interval** 值來建立其他作業。 例如，如果 **frequency** 為 "month"，則該排程只會每個月執行一次，而不是每天 (如果 **frequency** 為 "day")。 |
| `{minutes:[0]}` |在每小時整點執行。<br /><br />此作業也會每小時執行，但會在整點執行 (上午 12 點、上午 1 點、上午 2 點，依此類推)。 這相當於具有 **frequency** 為 "hour"、**startTime** 值為零分鐘且沒有 **schedule** 的作業 (如果 frequency 為 "day")。 不過，如果 **frequency** 為 "week" 或 "month"，則排程分別只會一週或一個月執行一天。 |
| `{"minutes":[15]}` |在每小時整點過 15 分鐘後執行。<br /><br />每小時執行，從上午 00:15 開始，然後在上午 1:15、上午 2:15，依此類推。 它結束於下午 11:15。 |
| `{"hours":[17], "weekDays":["saturday"]}` |在每週於星期六的下午 5 點執行。 |
| `{hours":[17], "weekDays":["monday", "wednesday", "friday"]}` |在每週於星期一、星期三及星期五的下午 5 點執行。 |
| `{"minutes":[15,45], "hours":[17], "weekDays":["monday", "wednesday", "friday"]}` |在每週星期一、星期三及星期五的下午 5:15 和 5:45 執行。 |
| `{"hours":[5,17], "weekDays":["monday", "wednesday", "friday"]}` |在每週於星期一、星期三及星期五的上午 5 點和下午 5 點執行。 |
| `{"minutes":[15,45], "hours":[5,17], "weekDays":["monday", "wednesday", "friday"]}` |在每週於星期一、星期三及星期五的上午 5:15、上午 5:45、下午 5:15 和上午 5:45 執行。 |
| `{"minutes":[0,15,30,45], "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` |在工作日每隔 15 分鐘執行一次。 |
| `{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` |在工作日的上午 9 點與下午 4:45 之間每隔 15 分鐘執行一次。 |
| `{"weekDays":["sunday"]}` |在星期日於開始時間執行。 |
| `{"weekDays":["tuesday", "thursday"]}` |在星期二和星期四於開始時間執行。 |
| `{"minutes":[0], "hours":[6], "monthDays":[28]}` |在每個月的第二十八天於上午 6 點執行 (假設 **frequency** 為 "month")。 |
| `{"minutes":[0], "hours":[6], "monthDays":[-1]}` |在當月最後一天的上午 6 點執行。<br /><br />如果您要在當月最後一天執行工作，請使用 -1，而不是 28、29、30 或 31。 |
| `{"minutes":[0], "hours":[6], "monthDays":[1,-1]}` |在每個月第一天和最後一天的上午 6 點執行。 |
| `{monthDays":[1,-1]}` |在每個月第一天和最後一天於開始時間執行。 |
| `{monthDays":[1,14]}` |在每個月第一天和第十四天於開始時間執行。 |
| `{monthDays":[2]}` |在當月第二天於開始時間執行。 |
| `{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` |在每個月第一個星期五的上午 5 點執行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` |在每個月第一個星期五於開始時間執行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":-3}]}` |在每個月倒數第三個星期五於開始時間執行。 |
| `{"minutes":[15], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` |在每個月第一個和最後一個星期五的上午 5:15 執行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` |在每個月第一個和最後一個星期五於開始時間執行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":5}]}` |在每個月第五個星期五於開始時間執行。<br /><br />如果當月沒有第五個星期五，該作業就不會執行。 如果您想要在當月最後一個出現的星期五執行作業，則可以考慮針對 occurrence 使用 -1，而不是 5。 |
| `{"minutes":[0,15,30,45], "monthlyOccurrences":[{"day":"friday", "occurrence":-1}]}` |在月份中最後一個星期五每隔 15 分鐘執行一次。 |
| `{"minutes":[15,45], "hours":[5,17], "monthlyOccurrences":[{"day":"wednesday", "occurrence":3}]}` |在每個月第三個星期三的上午 5:15、上午 5:45、下午 5:15 及下午 5:45 執行。 |

## <a name="see-also"></a>另請參閱

* [什麼是 Azure 排程器？](scheduler-intro.md)
* [Azure 排程器概念、術語及實體階層](scheduler-concepts-terms.md)
* [Azure 排程器限制、預設值和錯誤碼](scheduler-limits-defaults-errors.md)