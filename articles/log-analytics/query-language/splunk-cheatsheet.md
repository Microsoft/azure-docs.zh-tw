---
title: 從 Splunk 到 Azure Log Analytics | Microsoft Docs
description: 協助熟悉 Splunk 的使用者學習 Log Analytics 查詢語言。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/21/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 5ea9790695b8afe7bd42b98b071869756b301350
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42447049"
---
# <a name="splunk-to-log-analytics"></a>從 Splunk 到 Log Analytics

本文旨在協助熟悉 Splunk 的使用者學習 Log Analytics 查詢語言。 我們會直接比較這兩者的各項內容，以了解主要差異及相似之處，以便您運用現有的知識。

## <a name="structure-and-concepts"></a>結構和概念

下表比較 Splunk 與 Log Analytics 之間的概念和資料結構。

 | 概念  | Splunk | Log Analytics |  註解
 | --- | --- | --- | ---
 | 部署單位  | 叢集 |  叢集 |  Log Analytics 可允許任意跨叢集的查詢。 Splunk 則無法。 |
 | 資料快取 |  貯體  |  快取與保留原則 |  控制資料的期間和快取層級。 此設定會直接影響查詢效能和部署成本。 |
 | 資料的邏輯分割區  |  index  |  資料庫  |  可允許資料的邏輯分隔。 兩種實作皆允許分割區的集合聯集和聯結。 |
 | 結構化的事件中繼資料 | N/A | 資料表 |  Splunk 沒有事件中繼資料的搜尋語言概念。 Log Analytics 有資料表的概念，且資料表具有資料行。 每個事件執行個體會對應至一個資料列。 |
 | 資料記錄 | 事件 | 資料列 |  僅限詞彙變更。 |
 | 資料記錄屬性 | field |  資料行 |  在 Log Analytics 中，這已預先定義為資料表結構的一部分。 在 Splunk 中，每個事件都有自己的欄位集。 |
 | 類型 | datatype |  datatype |  Log Analytics 資料類型在資料行上設定時更明確。 兩者都能夠以動態方式使用資料類型，且擁有大致相當的資料類型集，包括 JSON 支援。 |
 | 查詢和搜尋  | 搜尋 | query |  Log Analytics 與 Splunk 兩者的概念基本上相同。 |
 | 事件擷取時間 | 系統時間 | ingestion_time() |  在 Splunk 中，每個事件都會取得事件編製索引時間的系統時間戳記。 在 Log Analytics 中，您可以定義稱為 ingestion_time 的原則，其會公開可透過 ingestion_time() 函式參考的系統資料行。 |

## <a name="functions"></a>Functions

下表指定 Log Analytics 中與 Splunk 函式相等的函式。

|Splunk | Log Analytics |註解
|---|---|---
|strcat | strcat()| (1) |
|split  | split() | (1) |
|if     | iff()   | (1) |
|tonumber | todouble()<br>tolong()<br>toint() | (1) |
|upper<br>lower |toupper()<br>tolower()|(1) |
| 取代 | replace() | (1)<br> 也請注意，雖然 `replace()` 在兩個產品中都採用三個參數，但參數是不同的。 |
| substr | substring() | (1)<br>也請注意，Splunk 使用以一為基底的索引。 Log Analytics 則是以零為基底的索引。 |
| tolower |  tolower() | (1) |
| toupper | toupper() | (1) |
| match | 符合 RegEx |  (2)  |
| RegEx | 符合 RegEx | 在 Splunk 中，`regex` 是運算子。 在 Log Analytics 中則是關係運算子。 |
| searchmatch | == | 在 Splunk 中，`searchmatch` 可允許搜尋完全相符的字串。
| 隨機 | rand()<br>rand(n) | Splunk 的函式會傳回 0 到 2<sup>31</sup>-1 的數字。 Log Analytics 的函式會傳回介於 0.0 和 1.0 之間的數字，或如果提供參數，則介於 0 到 n-1 之間。
| now | now() | (1)
| relative_time | totimespan() | (1)<br>在 Log Analytics 中，Splunk 的 relative_time(datetimeVal, offsetVal) 對等項目為 datetimeVal + totimespan(offsetVal)。<br>例如，<code>search &#124; eval n=relative_time(now(), "-1d@d")</code> 會變成 <code>...  &#124; extend myTime = now() - totimespan("1d")</code>。

(1) 在 Splunk 中，會使用 `eval` 運算子叫用函式。 在 Log Analytics 中，它會用做 `extend` 或 `project` 的一部分。<br>(2) 在 Splunk 中，會使用 `eval` 運算子叫用函式。 在 Log Analytics 中，它可以搭配 `where` 運算子使用。


## <a name="operators"></a>運算子

下列章節會提供使用 Splunk 與 Log Analytics 之間不同運算子的範例。

> [!NOTE]
> 針對下列範例目的，Splunk 欄位 _rule_ 對應至 Azure Log Analytics 中的資料表，而 Splunk 的預設時間戳記對應至 Log Analytics _ingestion_time()_ 資料行。

### <a name="search"></a>Search
在 Splunk 中，您可以省略 `search` 關鍵字，並指定不具引號的字串。 在 Azure Log Analytics 中，您必須使用 `find` 啟動每個搜尋，不具引號的字串是資料行名稱，且查閱值必須是加上引號的字串。 

| |  | |
|:---|:---|:---|
| Splunk | **search** | <code>search Session.Id="c8894ffd-e684-43c9-9125-42adc25cd3fc" earliest=-24h</code> |
| Log Analytics | **find** | <code>find Session.Id=="c8894ffd-e684-43c9-9125-42adc25cd3fc" and ingestion_time()> ago(24h)</code> |
| | |

### <a name="filter"></a>Filter
Azure Log Analytics 查詢會從篩選所在的表格式結果集開始。 Splunk 的篩選則是在目前索引上的預設作業。 您也可以使用 Splunk 中的 `where` 運算子，但我們不建議這樣做。

| |  | |
|:---|:---|:---|
| Splunk | **search** | <code>Event.Rule="330009.2" Session.Id="c8894ffd-e684-43c9-9125-42adc25cd3fc" _indextime>-24h</code> |
| Log Analytics | **where** | <code>Office_Hub_OHubBGTaskError<br>&#124; where Session_Id == "c8894ffd-e684-43c9-9125-42adc25cd3fc" and ingestion_time() > ago(24h)</code> |
| | |


### <a name="getting-n-eventsrows-for-inspection"></a>取得 n 個事件/資料列以進行檢查 
Azure Log Analytics 也支援以 `take` 作為 `limit` 別名。 在 Splunk 中，如果結果已進行排序，`head` 會傳回前 n 個結果。 在 Azure Log Analytics 中，限制不會排序，但會傳回找到的前 n 個資料列。

| |  | |
|:---|:---|:---|
| Splunk | **head** | <code>Event.Rule=330009.2<br>&#124; head 100</code> |
| Log Analytics | **limit** | <code>Office_Hub_OHubBGTaskError<br>&#124; limit 100</code> |
| | |



### <a name="getting-the-first-n-eventsrows-ordered-by-a-fieldcolumn"></a>取得依欄位/資料行排序的前 n 個事件/資料列
對於底部結果，Splunk 中可以使用 `tail`。 在 Azure Log Analytics 中，您可以使用 `asc` 指定排序方向。

| |  | |
|:---|:---|:---|
| Splunk | **head** |  <code>Event.Rule="330009.2"<br>&#124; sort Event.Sequence<br>&#124; head 20</code> |
| Log Analytics | **top** | <code>Office_Hub_OHubBGTaskError<br>&#124; top 20 by Event_Sequence</code> |
| | |




### <a name="extending-the-result-set-with-new-fieldscolumns"></a>以新的欄位/資料行擴充結果集
Splunk 也有 `eval` 函式，其無法與 `eval` 運算子相比較。 Splunk 的 `eval` 運算子和 Azure Log Analytics 的 `extend` 運算子皆只支援純量值函式和算術運算子。

| |  | |
|:---|:---|:---|
| Splunk | **eval** |  <code>Event.Rule=330009.2<br>&#124; eval state= if(Data.Exception = "0", "success", "error")</code> |
| Log Analytics | **extend** | <code>Office_Hub_OHubBGTaskError<br>&#124; extend state = iif(Data_Exception == 0,"success" ,"error")</code> |
| | |


### <a name="rename"></a>重新命名 
Azure Log Analytics 使用相同的運算子重新命名及建立新的欄位。 Splunk 有兩個不同的運算子，`eval` 和 `rename`。

| |  | |
|:---|:---|:---|
| Splunk | **rename** |  <code>Event.Rule=330009.2<br>&#124; rename Date.Exception as execption</code> |
| Log Analytics | **extend** | <code>Office_Hub_OHubBGTaskError<br>&#124; extend exception = Date_Exception</code> |
| | |




### <a name="format-resultsprojection"></a>格式結果/投影
Splunk 似乎沒有類似 `project-away` 的運算子。 您可以使用 UI 篩選欄位。

| |  | |
|:---|:---|:---|
| Splunk | **資料表** |  <code>Event.Rule=330009.2<br>&#124; table rule, state</code> |
| Log Analytics | **project**<br>**project-away** | <code>Office_Hub_OHubBGTaskError<br>&#124; project exception, state</code> |
| | |



### <a name="aggregation"></a>彙總
請參閱 [Log Analytics 查詢中的彙總](aggregations.md)，以了解不同的彙總函式。

| |  | |
|:---|:---|:---|
| Splunk | **stats** |  <code>search (Rule=120502.*)<br>&#124; stats count by OSEnv, Audience</code> |
| Log Analytics | **summarize** | <code>Office_Hub_OHubBGTaskError<br>&#124; summarize count() by App_Platform, Release_Audience</code> |
| | |



### <a name="join"></a>Join
Splunk 中的聯結具有重大限制。 子查詢的限制為 10000 筆結果 (設定在部署組態檔中)，且聯結類別的數目有限。

| |  | |
|:---|:---|:---|
| Splunk | **join** |  <code>Event.Rule=120103* &#124; stats by Client.Id, Data.Alias | join Client.Id max=0 [search earliest=-24h Event.Rule="150310.0" Data.Hresult=-2147221040]</code> |
| Log Analytics | **join** | <code>cluster("OAriaPPT").database("Office PowerPoint").Office_PowerPoint_PPT_Exceptions<br>&#124; where  Data_Hresult== -2147221040<br>&#124; join kind = inner (Office_System_SystemHealthMetadata<br>&#124; summarize by Client_Id, Data_Alias)on Client_Id</code>   |
| | |



### <a name="sort"></a>排序
在 Splunk 中，若要以遞增順序排序，您必須使用 `reverse` 運算子。 Azure Log Analytics 也支援定義 Null 的放置位置 (置於開頭或結尾)。

| |  | |
|:---|:---|:---|
| Splunk | **sort** |  <code>Event.Rule=120103<br>&#124; sort Data.Hresult<br>&#124; reverse</code> |
| Log Analytics | **排序依據** | <code>Office_Hub_OHubBGTaskError<br>&#124; order by Data_Hresult,  desc</code> |
| | |



### <a name="multivalue-expand"></a>展開多重值
這個運算子在 Splunk 和 Log Analytics 中很類似。

| |  | |
|:---|:---|:---|
| Splunk | **mvexpand** |  `mvexpand foo` |
| Log Analytics | **mvexpand** | `mvexpand foo` |
| | |




### <a name="results-facets-interesting-fields"></a>結果 Facet，關鍵值組欄位
在 Log Analytics 入口網站中，只會公開第一個資料行。 所有資料行可透過 API 提供。

| |  | |
|:---|:---|:---|
| Splunk | **fields** |  <code>Event.Rule=330009.2<br>&#124; fields App.Version, App.Platform</code> |
| Log Analytics | **facets** | <code>Office_Excel_BI_PivotTableCreate<br>&#124; facet by App_Branch, App_Version</code> |
| | |




### <a name="de-duplicate"></a>De-duplicate
您可以改用 `summarize arg_min()`，反轉已選擇的記錄順序。

| |  | |
|:---|:---|:---|
| Splunk | **dedup** |  <code>Event.Rule=330009.2<br>&#124; dedup device_id sortby -batterylife</code> |
| Log Analytics | **summarize arg_max()** | <code>Office_Excel_BI_PivotTableCreate<br>&#124; summarize arg_max(batterylife, *) by device_id</code> |
| | |




## <a name="next-steps"></a>後續步驟

- 進行[在 Log Analytics 中撰寫查詢](get-started-queries.md)課程。