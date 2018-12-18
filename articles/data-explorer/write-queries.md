---
title: 撰寫 Azure 資料總管查詢
description: 在此操作說明中，您將學到如何執行 Azure 資料總管的基本與進階查詢。
services: data-explorer
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 952d8801c189322161bbf8b795676af48b92c29f
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394322"
---
# <a name="write-queries-for-azure-data-explorer"></a>撰寫 Azure 資料總管查詢

在本文中，您將學到如何在 Azure 資料總管中使用查詢語言，以運用最常見的運算子執行基本查詢。 您也會了解一些該語言更進階的功能。

## <a name="prerequisites"></a>必要條件

有兩種方式可執行本文中的查詢：

- 在 Azure 資料總管中，使用我們已設定協助學習的*說明叢集*。
    請使用為 Azure Active Directory 成員的組織電子郵件帳戶[登入叢集](https://dataexplorer.azure.com/clusters/help/databases/samples)。

- 在您自己包含 StormEvents 範例資料的叢集上。 如需詳細資訊。請參閱[快速入門：建立 Azure 資料總管叢集與資料庫](create-cluster-database-portal.md)及[將範例資料內嵌至 Azure 資料總管](ingest-sample-data.md)。

StormEvents 範例資料集包含來自[美國國家環境資訊中心](https://www.ncdc.noaa.gov/stormevents/)的氣象相關資料。

## <a name="overview-of-the-query-language"></a>查詢語言概觀

Azure 資料總管中的查詢語言是一種負責處理資料並傳回結果的唯讀要求。 該要求是採用針對讓語法易於理解、撰寫及自動執行所設計的資料流程模型，並採用純文字加以陳述。 此資料所使用的結構描述實體其組織架構類似於 SQL 的階層：資料庫、資料表與資料欄。

此查詢包含一系列以分號 (`;`) 分隔的查詢陳述式，其中至少有一個陳述式為表格式運算陳述式，這種陳述式所產生的資料會以類似表格之網格的資料行與資料列方式排列。 此查詢的表格式運算陳述式會產生查詢的結果。

此表格式運算陳述式語法中的表格式資料會從某個表格式查詢運算子流到另一個運算子，先從資料來源開始 (例如，資料庫中的資料表，或是會產生資料的運算子)，然後流經一組使用管線運算子 (`|`) 分隔符號繫結在一起的資料轉換運算子。

例如，下列查詢會有一個陳述式，亦即表格式運算陳述式。 此陳述式一開始會先參照名為 `StormEvents` 的資料表 (裝載此資料表的資料庫在此是隱含的，且為連接資訊的一部分)。 接著會篩選資料表的資料 (資料列)，先依 `StartTime` 資料行的值篩選，再以 `State` 資料行的值篩選。 然後查詢會傳回「仍正常運作」的資料列計數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVWws1VISSxJLQGyNYwMDMx1DQ11DQw1FRLzUpBU2aArMgIpQjGvJFXB1lZByc3HP8jTxVFJQQEkm5xfmlcCAHoR9euCAAAA)**\]**

```Kusto
StormEvents
| where StartTime >= datetime(2007-11-01) and StartTime < datetime(2007-12-01)
| where State == "FLORIDA"  
| count
```

在此案例中，結果是：

|Count|
|-----|
|   23|
| |

如需詳細資訊，請參閱 [查詢語言參考資料](https://aka.ms/kustolangref) (英文)。

## <a name="most-common-operators"></a>常見運算子

本節所述的運算子是建置組塊，可用來了解 Azure 資料總管中的查詢。 您所撰寫的查詢大部分將包含以下數種運算子。

若要在說明叢集上執行查詢：請選取每個查詢上方的 [按一下以執行查詢]。

若要在您自己的叢集上執行查詢：

1. 將每個查詢複製到以網路為基礎的查詢應用程式，然後選取該查詢，或將游標放在查詢中。

1. 選取應用程式頂端的 [執行]。

### <a name="count"></a>count

[**count**](https://docs.microsoft.com/azure/kusto/query/countoperator)：傳回資料表中的資料列計數。

下列查詢會傳回 StormEvents 資料表中的資料列計數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRSM4vzSsBALU2eHsTAAAA)**\]**

```Kusto
StormEvents | count
```

### <a name="take"></a>take

[**take**](https://docs.microsoft.com/azure/kusto/query/takeoperator)：最多傳回所指定資料列數目的資料。

下列查詢會傳回 StormEvents 資料表中的五個資料列。 關鍵字 *limit* 是 *take* 的別名。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRKEnMTlUwBQDEz2b8FAAAAA%3d%3d)**\]**

```Kusto
StormEvents | take 5
```

> [!TIP]
> 除非來源資料有排序，否則不保證會傳回哪些記錄。

### <a name="project"></a>project

[**project**](https://docs.microsoft.com/azure/kusto/query/projectoperator)：選取資料行的子集。

下列查詢會傳回一組特定的資料行。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUShJzE5VMAWxCorys1KTSxSCSxKLSkIyc1N1FFzzUiAMoFgJiA%2fSFlJZAGS6JOYmpqcGFOUXpBaVVAKlCjKL81NS%2fRKLihJLMstSAY%2buIINnAAAA)**\]**

```Kusto
StormEvents
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="where"></a>其中

[**where**](https://docs.microsoft.com/azure/kusto/query/whereoperator)：篩選資料表以建立滿足述詞的資料列子集。

下列查詢依 `EventType` 與 `State` 篩選資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAEWMPQvCMBCGd8H%2fcFuWro4dBOvHkgoJOB%2fm0KjJhetRKfjjNe3g9n49r1OW1I2UdVivPvC%2bkxDM3k%2bFoG3B7F%2fMwQDmAE5Rl%2fCydceTPfjemsopPgk2VRXhB121TkV9TNRAl8MiZrz53zeww4Q3OgsXEp1%2bVYkDB7IoghpH%2bgI9OH8WnwAAAA%3d%3d)**\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="sort"></a>sort

[**sort**](https://docs.microsoft.com/azure/kusto/query/sortoperator)：按照一個或多個資料行的順序，排序輸入資料表的資料列。

下列查詢會依照 `DamageProperty` 的遞減順序排序資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2NPQvCMBCGd8H%2fcFuXrI4dBOvHEoUGnM%2fm0KjphctRKfjjNe0guL0fvM%2fbKktsBuo1LxdveN1ICCbvxkRQ11Btn8y%2bAuw9tIo6h%2bd1uz%2fYnTvaquwyi8JlhA1GvNJJOJHoCJ5yV2rFB8GqqCR8p04LSdSFSAaa3s9iopvfu%2fnDfasUMnuyKIIaBvoAtvGMsb4AAAA%3d)**\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| sort by DamageProperty desc
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

> [!NOTE]
> 作業的順序很重要。 請嘗試將 `take 5` 放在 `sort by` 之前。 是否要取得不同的結果？

### <a name="top"></a>top

[**top**](https://docs.microsoft.com/azure/kusto/query/topoperator)：傳回按指定資料行排序的前 *N* 個記錄。

下列查詢會以少一個運算子的方式，傳回與上方相同的結果。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2NOwvCMBSFd8H%2fcLcsWR07CNbHkgoJOMfmohGTG24vlYA%2fXtsOgtt5cL5jhTi1I2YZ1qs3vO7ICLN3tSA0Daj9kygo8DmAFS9LeNna48kcXGfUtBMqsIFrhZ1P%2foZnpoIsFQIO%2fdQXpgf2MgFYXEyooc1hETNU%2f071H%2bRblThQQOOZvcQRP1rSng21AAAA)**\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| top 5 by DamageProperty desc
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="extend"></a>extend

[**extend**](https://docs.microsoft.com/azure/kusto/query/extendoperator)：計算衍生的資料行。

下列查詢會藉由計算每個資料列中的值，來建立新的資料行。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2OvQ7CMAyEdyTewVuWMDJ2QGr5WQJSKzGHxoIiEkeuKVTi4WmooBKbfXeffaUQ%2b6LDIO189oLHBRnhs1d9RMgyUOsbkVNgg4NSrIzicVVud2ZT7Y1KnFCEJZx6yK23ZzwwRWTpwWFbJx%2bfggOf39lKQwEyKIKrGo%2bwSEdZ0pyCkemKtUyi%2fib1j9ZjDz311H9%2fBys2LTk0lhPT4RvwA3pn6AAAAA%3d%3d)**\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| top 5 by DamageProperty desc
| extend Duration = EndTime - StartTime
| project StartTime, EndTime, Duration, State, EventType, DamageProperty, EpisodeNarrative
```

運算式可以包含所有常見的運算子 (+、-、*、/、%)，而且有一系列您可呼叫的實用函式。

### <a name="summarize"></a>summarize

[**summarize**](https://docs.microsoft.com/azure/kusto/query/summarizeoperator)：彙總資料列群組。

下列查詢會依 `State` 傳回事件的計數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIBYnFJ%2beX5pUo2CqAaQ1NhaRKheCSxJJUAB%2fedDI3AAAA)**\]**

```Kusto
StormEvents
| summarize event_count = count() by State
```

**summarize** 運算子會將在 **by** 子句中有相同值的資料列分組，然後使用彙總函式 (例如 **count**) 將每個群組結合成一個資料列。 因此在這種狀況下，每個狀態都會有一個資料列，且會有一個資料行顯示該狀態中的資料列計數。

有一些 彙總函式，您可以在一個 **summarize** 運算子中使用其中幾個函式，來產生數個計算資料行。 例如，您可以取得每個狀態中的 storm 計數，以及各狀態其 storm 的唯一編號，然後使用 **top** 取得最受 storm 影響的狀態。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlUIBkk455fmlSjYKiSDaA1NHYWQyoJU%2fzSwXDFQPAUiAdYPktJUSKoE6kwsSQUZVpJfoGAKEYGblZJanAwAgbFb73QAAAA%3d)**\]**

```Kusto
StormEvents
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State
| top 5 by StormCount desc
```

**summarize** 作業的結果有：

- 以 **by** 命名的每個資料行

- 每個已計算運算式的一個資料行

- 每個依值組合的一個資料列

### <a name="render"></a>render

[**render**](https://docs.microsoft.com/azure/kusto/query/renderoperator)：以圖形化輸出呈現結果。

下列查詢會顯示為直條圖。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWMsQ7CQAxDdyT%2bIWMrdSgbSxmQ2Nj6Aei4Ru0hkqA0VwTi49uUBRZL9rPdmiidJmQbt5sPjJkoaHojoGeXKJmtWbUoK6DUQQNh6osj9onPwUq4vqC1YLjORc2Dpef2OaD%2bPcEBdvu6dvZQuWG077b6LTlV5A4VotwzcRyC2gxU6ktSqQAAAA%3d%3d)**\]**

```Kusto
StormEvents
| summarize event_count=count(), mid = avg(BeginLat) by State
| sort by mid
| where event_count > 1800
| project State, event_count
| render columnchart
```

下列查詢會顯示簡單的時間圖表。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIBYnFJ%2beX5pXYgkkNTYWkSoWkzDyN4JLEopKQzNxUHQXDFE2QtqLUvJTUIoUSoFhyBlASAAyXWQJWAAAA)**\]**

```Kusto
StormEvents
| summarize event_count=count() by bin(StartTime, 1d)
| render timechart
```

下列查詢會依時間模數計算一天內的事件計數 (以小時為單位分組)，並顯示時間圖表。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEADWNQQqDMBRE90LvMBtBwY0HcNkT2L2k8UuEJh9%2bfqSWHt4k4GZghpk3s7L450FB46P5g75KYYXjJJiwfZilm9WIvnZPaDGuGDC6vnRj8t7I%2fiNQ2S%2bWU9CpatfjfVZKLbLo7WGiLZnkGxJoxlqX%2bRf81ZbyiAAAAA%3d%3d)**\]**

```Kusto
StormEvents
| extend hour = floor(StartTime % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
| render timechart
```

下列查詢會比較時間圖表上的多個每日數列。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEACWPSwvCMBCE74L%2fYSgIFXrpD%2bihaKzxkUBTXyeputKCbSCmvvDHm9TL7gwzsN8qq03DHtTa%2b3DwBb0stRdUujMJrjetTQhlS2OLuiGMEF8QIa7GvvusyJBPLaFuEQbZZjWDnGHN9nwigyhYp1wwt7c8z7jgqZM7riZSKC6cFjIv5pimS1n4SLAdFixX7OCMzFkmRdAfundNU5r6QyAPejzrrrVJP8MxTu8eN%2fqT%2bL5xL5CBdcjnyrH%2fALPTSKnkAAAA)**\]**

```Kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render timechart
```

> [!NOTE]
> **render** 運算子是用戶端功能，而非引擎的一部分。 它會整合到該語言中以方便使用。 Web 應用程式支援下列選項：長條圖、直條圖、圓形圖、時間圖表及折線圖。 

## <a name="scalar-operators"></a>純量運算子

本節說明一些最重要的純量運算子。

### <a name="bin"></a>bin()

[**bin()**](https://docs.microsoft.com/azure/kusto/query/binfunction)：將值捨入為指定 bin 大小的整數倍數。

下列查詢會計算一天當中為貯體大小的計數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVWwU0hJLEktATI1jAwMzHUNjHQNTTQVEvNSkBTZYCoyMtQEGVdcmpubWJRZlaqQCrIiPjm%2fNK9EwVYBTGtoKiRVKiRl5mnAjdJRMEzRBABIhjnmkwAAAA%3d%3d)**\]**

```Kusto
StormEvents
| where StartTime > datetime(2007-02-14) and StartTime < datetime(2007-02-21)
| summarize event_count = count() by bin(StartTime, 1d)
```

### <a name="case"></a>case()

[**case()**](https://docs.microsoft.com/azure/kusto/query/casefunction)：評估述詞的清單，並傳回第一個其述詞得到滿足的結果運算式，或傳回最後一個 **else** 運算式。 您可以使用此運算子將資料分類或群組：

下列查詢會傳回新的資料行 `deaths_bucket`，並依數目將 death 分組。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAGWOwQrCQAxE74X%2bQ9hTCwX14FFBaK9e%2bgGS7gZdbFrYZEXFj7dbqgfNbfJmhml1DNzcaFDJsxdIZMbgnwSOUC8Cu%2fQq6lnUPpDVEroHtIpKKUB3pcEt7lMX7ZV0ClkUgiLPYLqlaQ%2fbdQWmx3AmU%2f2gTUJMzkf%2bYwkJY99%2fiDmuDqac545Bv3MAxb4Bic1Oy88AAAA%3d)**\]**

```Kusto
StormEvents
| summarize deaths = sum(DeathsDirect) by State
| extend deaths_bucket = case (
    deaths > 50, "large",
    deaths > 10, "medium",
    deaths > 0, "small",
    "N/A")
| sort by State asc
```

### <a name="extract"></a>extract()

[**extract()**](https://docs.microsoft.com/azure/kusto/query/extractfunction)：從文字字串取得規則運算式的相符項目。

下列查詢會從追蹤中擷取特定的屬性值。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAE2OwQrCMBBE74X%2bw9BTojHYagSVHJRevXkrHqJdpVBbSVew4McbFYungeXtvKmJsetzxw4WZQh2x5og9t6daIWOfdVcJIpkY1OFrc0U8rt3XLWNTbOZnhultU4UfoD5A4zRmVkovInDOo6%2bojh6gh5MTTmQwR0uQckiGb5FMZ0s9WEsQ3uo%2fixSccT9jdqz8ORqKTECV1cSaSdfq2k6L8oAAAA%3d)**\]**

```Kusto
let MyData = datatable (Trace: string) ["A=1, B=2, Duration=123.45,...", "A=1, B=5, Duration=55.256, ..."];
MyData
| extend Duration = extract("Duration=([0-9.]+)", 1, Trace, typeof(real)) * time(1s)
```

此查詢使用 **let** 陳述式，這會將名稱 (在此狀況中為 `MyData`) 繫結至運算式。 對於剩餘的範圍，在此會顯示 **let** 陳述式 (全域範圍或在函式主體範圍中)，名稱可用於參照其繫結的值。

### <a name="parsejson"></a>parse_json()

[**parse_json()**](https://docs.microsoft.com/azure/kusto/query/parsejsonfunction)：將字串解譯為 JSON 值，並以動態形式傳回值。 當您需要從 JSON 複合物件擷取多個元素時，使用 **extractjson()** 函式會更合適。

下列查詢會擷取陣列中的 JSON 元素。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAHWPQQuCQBCF74L%2fYdmLBSJ6EGKjU17r1E0kJh1C2XZlHc0w%2f3ur1s1O896bB%2fONRGKnVwIE7MAKOwhuEtnmYiBHwRoypbpvXSf1Bl60BqjUiot04B3IFrmIol0Q%2bpPLdauIi3iyj9KWojCcNfRWx7NuqEiw48KaMRu9bO86y3HXeTPsCVXBzvg8amlpajANXqtGq4VmO5VqoyvM6dsKfkhpmAUzkf9nM9OtLi3reg79ar788AEVX8GkOAEAAA%3d%3d)**\]**

```Kusto
let MyData = datatable (Trace: string)
['{"duration":[{"value":118.0,"valcount":5.0,"min":100.0,"max":150.0,"stdDev":0.0}]}'];
MyData
| extend NewCol = parse_json(Trace)
| project NewCol.duration[0].value, NewCol.duration[0].valcount, NewCol.duration[0].min, NewCol.duration[0].max, NewCol.duration[0].stdDev
```

下列查詢會擷取 JSON 元素。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAE2OwQqCQBCG74LvsOzFBBE9CLHRKa916hYRkw6RbLuyO5pRvXvrGtZpvn9m4P8kEts%2bSiBga1a7QXCWyBZ7AxUKZslc1SVmh%2bjJe5AdcpHnyzRLxlTpThEXxRhvV%2bVOWeYZBseFZ0t1iT0XLryj4yoMprIweDEcCFXNdnjfaOnaWzAWT43VamqPx6fW6AYr%2bn6l3iH5S95hXjiLH8Mw82TxAQvJEB%2fsAAAA)**\]**

```Kusto
let MyData = datatable (Trace: string) ['{"value":118.0,"valcount":5.0,"min":100.0,"max":150.0,"stdDev":0.0}'];
MyData
| extend NewCol = parse_json(Trace)
| project NewCol.value, NewCol.valcount, NewCol.min, NewCol.max, NewCol.stdDev
```

下列查詢會擷取類型為動態資料的 JSON 元素。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAD2NMQvCMBBG90D%2bw5GphVLSoSARt65ubuJwJjdU0lZiWlrU%2f25MotO9x8H7LHk4bh16hAOYcDxeLUFxcqhJgdlGHHpdcnbOWDzFgnYmoZpmV8tK6GkePTmh2q8N%2fRg%2bUkbGNXAb%2beFNR4tQQd7lZc9ZGuXsBXc33Uh7iJN1jFdZcvunIf5HXCvOEqf2BwXmDCnKAAAA)**\]**

```Kusto
let MyData = datatable (Trace: dynamic)
[dynamic({"value":118.0,"counter":5.0,"min":100.0,"max":150.0,"stdDev":0.0})];
MyData
| project Trace.value, Trace.counter, Trace.min, Trace.max, Trace.stdDev
```

### <a name="ago"></a>ago()

[**ago()**](https://docs.microsoft.com/azure/kusto/query/agofunction)：從目前的 UTC 時鐘時間減去指定的時間範圍。

下列查詢會傳回過去 12 小時的資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA1WOQQ6CQAxF9yTc4S8hQcmQuNSNR4ALTKQyJDAlnSIuPLwzJGrctM3v+7+t684R7qMEhW6MafQUMJAnsUoIdl4mQm/VVrC+h0Z6shFOINZAIc/qOql24KIEL8nIAuWYohC6sfQB9yjtPtPA8SrhmGeLjF7RjTO1Gu+cIdYPVHjeisOpLyukKTbjYml5piuvXknwIU1lGlPm2Qvzg55L+u+b9udIyOZI6LfHZf/YNK58Ay2HrbAEAQAA)**\]**

```Kusto
//The first two lines generate sample data, and the last line uses
//the ago() operator to get records for last 12 hours.
print TimeStamp= range(now(-5d), now(), 1h), SomeCounter = range(1,121)
| mvexpand TimeStamp, SomeCounter
| where TimeStamp > ago(12h)
```

### <a name="startofweek"></a>startofweek()

[**startofweek()**](https://docs.microsoft.com/azure/kusto/query/startofweekfunction)：傳回包含該日期的當週開始日期，若有位移的話則移位

下列查詢會以不同位移傳回該週開始日期。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEACtKzEtPVchPSytOLVFIK8rPVdA1VCjJVzBUKC5JLVAw5OWqUSgoys9KTS5RKE9NzQ4uSSwqUbAFygLp%2fDSQkEZefrmGpg7UEE0dCA0AdE3lv1kAAAA%3d)**\]**

```Kusto
range offset from -1 to 1 step 1
| project weekStart = startofweek(now(), offset),offset
```

此查詢使用 **range** 運算子，產生單資料行資料表的資料。 亦請參閱：[**startofday()**](https://docs.microsoft.com/azure/kusto/query/startofdayfunction)[**startofweek()**](https://docs.microsoft.com/azure/kusto/query/startofweekfunction)[**startofyear()**](https://docs.microsoft.com/azure/kusto/query/startofyearfunction))、[**startofmonth()**](https://docs.microsoft.com/azure/kusto/query/startofmonthfunction)、[**endofday()**](https://docs.microsoft.com/azure/kusto/query/endofdayfunction)、[**endofweek()**](https://docs.microsoft.com/azure/kusto/query/endofweekfunction)[**endofmonth()**](https://docs.microsoft.com/azure/kusto/query/endofmonthfunction) 及 [**endofyear()**](https://docs.microsoft.com/azure/kusto/query/endofyearfunction)。

### <a name="between"></a>between()

[**between()**](https://docs.microsoft.com/azure/kusto/query/betweenoperator)：比對內含範圍內的輸入。

下列查詢會依指定的日期範圍篩選資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVVISi0pT03NU9BISSxJLQGKaBgZGJjrApGRuaaCnp4ChrixgaYmyKTk%2fNK8EgBluyagXgAAAA%3d%3d)**\]**

```Kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. datetime(2007-07-30))
| count
```

下列查詢會依照指定的日期範圍篩選資料，但自開始日起的三天 (`3d`) 會略有變化。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVVISi0pT03NU9BISSxJLQGKaBgZGJjrApGRuaaCnp6CcYomSF9yfmleCQCGAqjRTAAAAA%3d%3d)**\]**

```Kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. 3d)
| count
```

## <a name="tabular-operators"></a>表格式運算子

Kusto 有表格式運算子，有一部分會在本文的其他小節中說明。 本節將著重說明 **parse**。 

### <a name="parse"></a>parse

[**parse**](https://docs.microsoft.com/azure/kusto/query/parseoperator)：評估字串運算式，並將其值剖析至一或多個計算的資料行。 有三種剖析方式：簡易 (預設值)、Regex 及寬鬆。

下列查詢會使用預設的簡易剖析，來剖析追蹤與擷取相關的值。 運算式 (稱為 StringConstant) 是規則字串值，採嚴格比對：擴充的資料行必須符合所需的類型。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2UTU%2fDMAyG75X6H6xcxlCkpRlsUNQjN6gQ2wnEoevMFsiaKk2HJvHjabqvlI91l11QLrH12vETW5Zo4H411kmKEME0MdWZSISz2yVmpvaHhdEim3V979n3OrU%2fhFgZ8boaSZHiI0pMiipEY6FKnWKcLDB6EDlKkeEoneO0lKgpGGUSWYcUER9SKOw1LhcT1BHvU5AqfR%2bLKpbxXjDscRYMgF2FFyxkwRMFvX7ngCLXuBSqLO5%2bT9S%2ftrJuh54OI7g8iMFaMdhxGOy0GJz9i25w%2fjdG0IoRHNWNNe1ph2pwEKNlqI7HsEPley83vrfZCL73CXmiq%2fr32wA%2bhJnDOZAGEQHXBNIEIq4VSpXNbAIXkbjAO8UOmuz4bWoXlrhWWO0vqyA2%2bAcw2f7B1rORd60calat3jA1TRbq1A6NxsC%2bLdCoCuj3p74AKTs4pmcFAAA%3d)**\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse EventTrace with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previouLockTime
```

下列查詢會使用 `kind = regex` 剖析追蹤與擷取相關的值。 StringConstant 可以是規則運算式。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2UQU%2fCQBCF7036HyZ7gWKRbVHQmgY9eNPGCCcoh9KOsLK0ZLtFMf54l6LQBgUuXEyTTbP7pt3vvclwlPC47IkgRHAhCqR6Rhyher%2fAWOb7TioFi8eGrg10rZLvO%2bAlkr0su5yF%2bIwcg1SVCEyTTIToBTN0n9gcOYuxG04wyjgKE2QiA56XpK7dNiFdvXrZbITCtZsm8CSc9piqpXbDajdsarWAXjkX1KFW3wSx%2fs8exVzggiVZ%2bvD7h5rXK5lRMU%2bHYV3uxaAHMehxGPS0GDb9F2nY9t8Y1kEM66g01rSnbarWXowDTXU8xqqpdG14o2vfE0HXPmEeCHX%2fKYsjNR8EjvEdtqMB3picAKme1zrGIKh%2f3NX7w5pLoEgLt6SM56c1PzpTq6oqYpIitMOTeAxAlKb6c3Wjs3GBbAzJJUV8UjQjP91BJztuOGryKbHvGwQgxxbJK4ayTFKKBbahQCkA2DX7C29veJJmBQAA)**\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse kind = regex EventTrace with "(.*?)[a-zA-Z]*=" resourceName @", totalSlices=\s*\d+\s*.*?sliceNumber=" sliceNumber:long  ".*?(previous)?lockTime=" lockTime ".*?releaseTime=" releaseTime ".*?previousLockTime=" previousLockTime:date "\\)"  
| project resourceName , sliceNumber , lockTime , releaseTime , previousLockTime
```

下列查詢會使用 `kind = relaxed` 剖析追蹤與擷取相關的值。 StringConstant 是規則字串值，採寬鬆比對：擴充的資料行可以部分符合所需的類型。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2US0%2fCQBDH7036HSZ7wZpN2BYFrenRGzZG4KLxUNoRVpYu2W5REj%2b83fKw9QE1kYvppTOZx%2f%2b3MxmBGm5WQxXFCAEkkS6%2bsUA4uV5iqku%2fn2nF04ljWw%2b21Sr9PoRS86fVQPAY71BglBUpCjOZqxjDaI7BLV%2bg4CkO4ikmuUBFQUsdiTIlC7wehcz8hvl8jCrwOhSEjGdDXuQyr%2b322h5zu8Au%2fDPmM%2feeglr32ROxULjkMs%2f63xfqXJowp0WPh%2bGe78VgBzFYMwx2XAyP%2fYtpeN7PGO5BDLfRNNa0x12q7l6MA0vVHMMslW09XtnW5iLY1hssIlXon%2fE0CYom0SsmQP6IMxz1%2b7%2b7AnXQdX6TNXMIvHA9hVMgNYEEqiaQuj5StXwh04kpUNVLqup3ETsCsoMxpavSSdXyi7NrIohJ%2foJDtoRbzybcMeFQjkjJZ4x1nYVWtEPtleHjjaGmCujnVu%2fWU75tHgYAAA%3d%3d)**\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse kind=relaxed "Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=NULL, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=NULL, previousLockTime=02/17/2016 08:39:01)" with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previousLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

## <a name="time-series-analysis"></a>時間序列分析

### <a name="make-series"></a>make-series

[**make-series**](https://docs.microsoft.com/azure/kusto/query/make-seriesoperator)：將如 [summarize](https://docs.microsoft.com/azure/kusto/query/summarizeoperator) 的資料列群組彙總在一起，但對 by 值的每個組合產生 (time) 序列向量。

下列查詢會針對每天的 storm 事件計數傳回一組時間序列。 查詢包含每個狀態的三個月期限，以常數 0 填入遺漏的 bin：

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUchNzE7VLU4tykwtVsizTc4vzSvR0FRISU1LLM0psTVQyM9TCC5JLCoJycxNVcjMUyhKzEtP1UhJLEktAYpoGBkYmOsaGAKRpo4CmqixrjFI1DBFUyGpEmRKSSoAazsM0n0AAAA%3d)**\]**

```Kusto
StormEvents
| make-series n=count() default=0 on StartTime in range(datetime(2007-01-01), datetime(2007-03-31), 1d) by State
```

在您產生一組 (time) 序列後，您可以套用 series 函式，以偵測異常的形狀、季節性模式等等。

下列查詢會擷取在特定一天中具有最多事件的前三名狀態：

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2OsQoCMRBEe8F%2f2DIBAzmvsLrSLzj7EC%2brBs3mSPbkBD%2feLDYibPVmZmdGziUdn0hct5s3JH9HU7FErEDDlBdipSHgxS8PHixkgpF94VNMCJGgeLqiCp6RG1F7aw%2fGdu30Dv5ob3qhXdBwfskXRmnElZECfDtdbbgq0qJwnqEX76%2fmyCW%2ftkV1Ek9pWSwgNdOt7foAJIuybs8AAAA%3d)**\]**

```Kusto
StormEvents
| make-series n=count() default=0 on StartTime in range(datetime(2007-01-01), datetime(2007-03-31), 1d) by State
| extend series_stats(n)
| top 3 by series_stats_n_max desc
| render timechart
```

如需詳細資訊，請檢閱 [series 函式](https://docs.microsoft.com/azure/kusto/query/scalarfunctions#series-processing-functions) (英文)。

## <a name="advanced-aggregations"></a>進階彙總

本文稍早已討論如 **count** 與 **summarize** 等基本彙總。 本節將介紹更進階的選項。

### <a name="top-nested"></a>top-nested

[**top-nested**](https://docs.microsoft.com/azure/kusto/query/topnestedoperator)：產生階層式的首要結果，其中可根據上一層的值向下切入每一層。

此運算子適用於儀表板視覺效果情節，或必須回答如下的問題時：「尋找 K1 的前 N 個值 (使用某個彙總函式)；對於每一個值尋找 K2 的前 M 個值 (使用其他彙總函式)；...」

下列查詢會傳回頂層為 `State`，接著為 `Sources` 的階層資料表。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjJL9DNSy0uSU1RMFLIT1MILkksSVVIqlQoLs3VcEpNz8zzSSzR1OHlQlJoDFaYX1qUTEilIUila16KT35yYklmfh6GcgDrXwk5jgAAAA%3d%3d)**\]**

```Kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
top-nested 3 of Source by sum(BeginLat),
top-nested 1 of EndLocation by sum(BeginLat)
```

### <a name="pivot-plugin"></a>pivot() plugin

[**pivot() plugin**](https://docs.microsoft.com/azure/kusto/query/pivotplugin)：透過將輸入資料表中某資料行的唯一值，轉變為輸出資料表中的多個資料行，來旋轉資料表。 此運算子所執行的彙整必須在最終輸出中任何剩餘的資料行值上。

以下查詢會套用篩選條件，並將資料列樞紐轉動成資料行。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSgoys9KTS5RCC5JLEnVUQBLhFQWpILkyjNSi1IhMgrFJYlFJcXlmSUZCkqOPkoIabgOhYzEYgWl8My8FLBsalliTilIZ0FmWX6JBtgUTQDlv21NfQAAAA%3d%3d)**\]**

```Kusto
StormEvents
| project State, EventType
| where State startswith "AL"
| where EventType has "Wind"
| evaluate pivot(State)
```

### <a name="dcount"></a>dcount()

[**dcount()**](https://docs.microsoft.com/azure/kusto/query/dcount-aggfunction)：傳回群組中運算式之相異值數目的估計值。 使用 [**count()**](https://docs.microsoft.com/azure/kusto/query/countoperator) 可計數所有的值。

下列查詢會依 `State` 計數相異的 `Source`。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlUIzi8tSk4tVrBVSEnOL80r0YAIaCokVSoElySWpAIAFKgSBDoAAAA%3d)**\]**

```Kusto
StormEvents
| summarize Sources = dcount(Source) by State
```

### <a name="dcountif"></a>dcountif()

[**dcountif()**](https://docs.microsoft.com/azure/kusto/query/dcountif-aggfunction)：傳回其述詞估計為 true 之資料列運算式其相異值數目的估計值。

下列查詢會計數 `Source` 的相異值，其中 `DamageProperty < 5000`。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRKEnMTlUwNDDg5apRKC7NzU0syqxKVQjOLy1KTi1WsFVISc4vzSvJTNOACOkouCTmJqanBhTlF6QWlVQq2CiYGhgYaCokVSoElySWpAIAuk%2fTX14AAAA%3d)**\]**

```Kusto
StormEvents 
| take 100
| summarize Sources = dcountif(Source, DamageProperty < 5000) by State
```

### <a name="dcounthll"></a>dcount_hll()

[**dcount_hll()**](https://docs.microsoft.com/azure/kusto/query/dcount-hllfunction)：從 HyperLogLog 結果計算 **dcount** (由 [**hll**](https://docs.microsoft.com/azure/kusto/query/hll-aggfunction) 或 [**hll_merge**](https://docs.microsoft.com/azure/kusto/query/hll-merge-aggfunction) 產生)。

下列查詢會使用 HLL 演算法產生計數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlXIyMkJSi1WsAUxNFwScxPTUwOK8gtSi0oqNRWSKhWSMvM0gksSi0pCMnNTdQwNcjUx9PumFqWnpkCMiM8FcTQgpoKVFhTlZ6UmlyikJOeX5pXEg6yB69EEAKm9wyCXAAAA)**\]**

```Kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

### <a name="argmax"></a>arg_max()

[**arg_max()**](https://docs.microsoft.com/azure/kusto/query/arg-max-aggfunction)：尋找群組中將運算式最大化的資料列，並傳回另一個運算式的值 (或使用 * 傳回整個資料列)。

以下查詢會傳回每個狀態中最後流量報告的時間。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQDzQyoLUhVsbRWU3HLy81OUQLLFpbm5iUWZVakKiUXp8bmJFRrBJYlFJSGZuak6ClqaCkmVCkCBklSQ2oKi%2fKzU5BKIgI4CkkLXvBQoA2YNAHO1S0OFAAAA)**\]**

```Kusto
StormEvents
| where EventType == "Flood"
| summarize arg_max(StartTime, *) by State
| project State, StartTime, EndTime, EventType
```

### <a name="makeset"></a>makeset()

[**makeset()**](https://docs.microsoft.com/azure/kusto/query/makeset-aggfunction)：傳回一組相異值的動態 (JSON) 陣列，這些是運算式在群組中取得的值。

下列查詢會傳回何時依每個狀態回報流量的所有時間，並從一組相異值建立陣列。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWLQQ6CQBAE7yb8ocNJE76wR3mA8IEFOxF1mM3siIHweAVPHqsq1bianCeOnovDiveNRuzczokIAWX9VL2WW80vkWjDQuzuwqTmGQESH8z0Y%2bPRvB2EJ3QzvuTcvmR6Z%2b8%2fUf3NH6ZkMFeAAAAA)**\]**

```Kusto
StormEvents
| where EventType == "Flood"
| summarize FloodReports = makeset(StartTime) by State
| project State, FloodReports
```

### <a name="mvexpand"></a>mvexpand

[**mvexpand**](https://docs.microsoft.com/azure/kusto/query/mvexpandoperator)：從動態輸入的資料行展開多值的集合，讓集合中的每個值取得不同的資料列。 所展開資料列中的其他所有資料行則會重複。 並且與 makelist 相反。

下列查詢會先建立一個集合，然後利用該集合來示範 **mvexpand** 功能，藉此產生範例資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWOQQ6CQAxF9yTcoWGliTcws1MPIFygyk9EKTPpVBTj4Z2BjSz%2f738v7WF06r1vD2xcp%2bCoNq9yHDFYLIsvvW5Q0JybKYCco2omqnyNTxHW7oPFckbwajFZhB%2bIsE1trNZ0gi1dpuRmQ%2baC%2bjuuthS7Fbwvi%2f%2bP8lpGvAMP7Wr3A6BceSu7AAAA)**\]**

```Kusto
let FloodDataSet = StormEvents
| where EventType == "Flood"
| summarize FloodReports = makeset(StartTime) by State
| project State, FloodReports;
FloodDataSet
| mvexpand FloodReports
```

### <a name="percentiles"></a>percentiles()

[**percentiles()**](https://docs.microsoft.com/azure/kusto/query/percentiles-aggfunction)：針對由運算式定義的母體擴展，傳回該母體擴展之[**最近序數百分位數**](https://docs.microsoft.com/azure/kusto/query/percentiles-aggfunction)的估計值。 其精確度取決於百分位數區域中的母體密度。 僅用於 [**summarize**](https://docs.microsoft.com/azure/kusto/query/summarizeoperator) 內部彙總的內容。

下列查詢會計算 storm 持續時間的百分位數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUUitKEnNS1FIKS1KLMnMz1OwVXDNSwnJzE1V0FUILkksKgGxQQrLM1KLUhHq7BQMirEI2ygYZ4CEi0tzcxOLMqtSFQpSi5KBlmXmpBZrwJTpKJjqKBgZACkgtgBiS1NNAEC7XiaYAAAA)**\]**

```Kusto
StormEvents
| extend duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize percentiles(duration, 5, 20, 50, 80, 95)
```

下列查詢會依狀態計算 storm 持續時間的百分位數，並以五分鐘的 bin (`5m`) 標準化資料。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1NSwrCMBTcC95hli1EKEpBQd31BHUvafOgAZNI8uIPD28SEBVcDDMM8%2bnZedNdyHKYz56gG5NVUNFL1s5ih86qgzaEBXqWnrPOwetEnj65PZrwx95iNWU7RGOk1w8C5avj6KLlNF64qjHcMWhbvXsCralFPmT6rZ%2fJj2lAnyh8pwWWTaKEdcKmLYul%2fgLODFs%2b4AAAAA%3d%3d)**\]**

```Kusto
StormEvents
| extend duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count() by bin(duration, 5m), State
| summarize percentiles(duration, 5, 20, 50, 80, 95) by State
```

### <a name="cross-dataset"></a>跨資料集

本節說明一些可讓您建立更複雜查詢、跨資料表聯結資料，以及跨資料庫與叢集查詢的元素。

### <a name="let"></a>let

[**let**](https://docs.microsoft.com/azure/kusto/query/letstatement)：改善模組化並重複使用。 **let** 陳述式可讓您將可能複雜的運算式分解成數個部分，每個部分皆繫結至一個名稱，也允許您將這些部分組合在一起。 **let** 陳述式也可用於建立使用者定義的函式與檢視 (其結果看起來像新資料表之資料表上方的運算式)。 由 **let** 陳述式繫結的運算式可以是純量類型、表格式類型，或使用者定義的函式 (lambdas)。

下列範例會建立表格式類型變數，並在後續的運算式中使用該變數。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAMtJLVHwyUzPKMnLzEsPLskvyi1WsOXlArNcy1LzSop5uWoUyjNSi1IVwPyQyoJUBVtbBSW4LiVrXq4coDGOZYk5iXnJGakkGQPXBTIGzSUgPVn5mXkKGmhmayrk5ykElySWpIKUpGQWl2TmJZdARACul3kY0gAAAA%3d%3d)**\]**

```Kusto
let LightningStorms =
StormEvents
| where EventType == "Lightning";
let AvalancheStorms =
StormEvents
| where EventType == "Avalanche";
LightningStorms
| join (AvalancheStorms) on State
| distinct State
```

### <a name="join"></a>Join

[**join**](https://docs.microsoft.com/azure/kusto/query/joinoperator)：透過比對每個資料表中所指定資料行的值，來合併兩個資料表的資料列，以形成新的資料表。 Kusto 支援全系列的聯結類型：**fullouter** **inner**、**innerunique** **leftanti** **leftantisemi** **leftouter**、**leftsemi**、**rightanti**、**rightantisemi** **rightouter** **rightsemi**。

下列範例使用內部聯結來聯結兩個資料表。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA8tJLVGIULBVSEksAcKknFQN79RKq+KSosy8dB2FsMSc0lRDq5z8vHRNXq5oXi4FIFBPVNcx1IGyk9R1jJDYxjB2srqOCS9XrDUvVw7Qhkj8Nhih2wA0ydAAySgjZI4xnJMCtMQAYkuEQo1CVn5mnkJ2Zl6KbWZeXmoR0Nb8PAWgZQAFPLdO5AAAAA==)**\]**

```Kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X 
| join kind=inner Y on Key
```

> [!TIP]
> 在聯結之前，使用 **where** 與 **project** 運算子來減少輸入資料表中的資料列與資料行數目。 如果某個資料表一定會比另一個資料表還小，請將它做為聯結的左側 (管線)。 聯結相符項目的資料行必須具有相同名稱。 如果必須重新命名其中一個資料表中的資料行，請使用 **project** 運算子。

### <a name="serialize"></a>serialize

[**serialize**](https://docs.microsoft.com/azure/kusto/query/serializeoperator)：將資料列集合序列化，讓您可以使用需要序列化資料的函式，例如 **row_number()**。

下列查詢會成功，因為資料已序列化。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIzi%2fNK9HQVEiqVAguSSxJBcumFmUm5gBlQZzUipLUvBSFovzy%2bLzS3KTUIgVbJI6GJgB4pV4NWgAAAA%3d%3d)**\]**

```Kusto
StormEvents
| summarize count() by State
| serialize
| extend row_number = row_number()
```

資料列集合如果是以下運算子的結果，則也視為佇列化：**sort** **top** 或 **range** 運算子，後面可選擇性加上 **project** **project-away** **extend**、**where** **parse** **mvexpand** 或 **take** 運算子。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIzi%2fNK9HQVEiqVAguSSxJBcvmF5XABRQSi5NBgqkVJal5KQpF%2beXxeaW5SalFCrZIHA1NAEGimf5iAAAA)**\]**

```Kusto
StormEvents
| summarize count() by State
| sort by State asc
| extend row_number = row_number()
```

### <a name="cross-database-and-cross-cluster-queries"></a>跨資料庫與跨叢集查詢

[跨資料庫與跨資料集查詢](https://docs.microsoft.com/azure/kusto/query/cross-cluster-or-database-queries)：您可以查詢相同叢集上的資料庫，做法是將該資料庫稱做 `database("MyDatabase").MyTable`。 您可以查詢遠端叢集上的資料庫，做法是將該資料庫稱做 `cluster("MyCluster").database("MyDatabase").MyTable`。

下列查詢是從一個叢集呼叫的，並會查詢 `MyCluster` 叢集中的資料。 若要執行此查詢，請使用您自己的叢集名稱和資料庫名稱。

```Kusto
cluster("MyCluster").database("Wiki").PageViews
| where Views < 2000
| take 1000;
```

### <a name="user-analytics"></a>使用者分析

本節包含的元素與查詢可示範在 Kusto 中執行使用者行為分析是多麼的容易。

### <a name="activitycountsmetrics-plugin"></a>activity_counts_metrics plugin

[**activity_counts_metrics plugin**](https://docs.microsoft.com/azure/kusto/query/activity-counts-metrics-plugin)：計算有用的活動計量 (總計數值、相異計算值、新值的相異計數，以及彙總的相異計數)。 系統會計算每個時間範圍的計量，然後比較計量，再將計量與所有先前的時間範圍彙總在一起。

下列查詢會藉由計算每日活動計數來分析使用者的採用狀況。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAJXSPQvCMBAG4L3Q%2f5CtFlLoFyiVDn4M6mqdREpsggTaKs1VEfzxXm0LDiEimcJz3CW8VwogClgDKWcgQFZiEvrB1PNnnh%2b4c9sqsUDUXMPxyA9Z8%2bsjDfhwz0hKsBzPuRSTgxLNlicKGllfKMmwBw6sbsnY0bWto205C4cS3Rso2tpgO4MtDbbSWvixzGD6eb1ttBYZev42%2fbzI8L%2fe9n9b3NkJQ8xs60XEnZUt1hBWgLxLeObFta1B5ZXAKAs1BPuVKO03iXb7gp36tXDfExVB%2f2ICAAA%3d)**\]**

```Kusto
let start=datetime(2017-08-01);
let end=datetime(2017-08-04);
let window=1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'A', datetime(2017-08-01),
'D', datetime(2017-08-01),
'J', datetime(2017-08-01),
'B', datetime(2017-08-01),
'C', datetime(2017-08-02),
'T', datetime(2017-08-02),
'J', datetime(2017-08-02),
'H', datetime(2017-08-03),
'T', datetime(2017-08-03),
'T', datetime(2017-08-03),
'J', datetime(2017-08-03),
'B', datetime(2017-08-03),
'S', datetime(2017-08-03),
'S', datetime(2017-08-04),
];
T
| evaluate activity_counts_metrics(UserId, Timestamp, start, end,
window)
```

### <a name="activityengagement-plugin"></a>activity_engagement plugin

[**activity_engagement plugin**](https://docs.microsoft.com/azure/kusto/query/activity-engagement-plugin)：依據 ID 資料列來計算活動參與率隨滑動時間軸時段的變化。 **activity_engagement plugin** 可用於計算 DAU、WAU 及 MAU (每日、每週與每月作用中使用者)。

下列查詢會傳回在行進中的七天時間範圍中，將每日使用應用程式的相異使用者總數，比較每週使用應用程式的相異使用者總數，所得到的比率。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1RQWrDMBC8G%2fyHvUVOHGy1lByKD6GBviDkUIoR1tpVsS0jr0MCeXxXiigpVAiBVjOzM6uigHcc0SlCcGrUdgCtSIFtYZnRgWrInA0ZnNOkR4J6JuUIKo9CMgOKp1LutqXknb1GDI76P8RzQHCXDqHW6gqt43ZRkeydNxNOIHWa3AAv5Ctei2xvx06IQNtGTlZInT0AHQN9BpFt5EO59kHmKvQVUUivX8q1y3L4c9%2fIks%2bt5LoMwsMZLxMrgtHVXcb7pOuEthWemEFvBkPARL%2fSpCjgTfXN0vuBHvbH4rQ%2fsikyNjg6q37xL3GsV47cqQ4HHEl8rIxefeZhNHmMmIehsB2dp8nunnZy9hsbiriDWuqTWqpfxdBsLb2ZGzhm8y%2f6b2i%2bWO8HLFcMGe8BAAA%3d)**\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-01-31);
range _day from _start to _end step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1)
| mvexpand id=_users to typeof(long) limit 1000000
// Calculate DAU/WAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 7d)
| project _day, Dau_Wau=activity_ratio*100
| render timechart
```

> [!TIP]
> 計算時 DAU/MAU，請變更結束資料和行進中時間範圍 (OuterActivityWindow)。

### <a name="activitymetrics-plugin"></a>activity_metrics plugin

[**activity_metrics plugin**](https://docs.microsoft.com/azure/kusto/query/activity-metrics-plugin)：根據目前期間與上個期間的比較，計算有用的活動計量 (相異計數值、新值的相異計數、保留率及流失率)。

下列查詢會計算所指定資料集的流失率與保留速率。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG2SwW7CMAyG70i8g2%2bk0KoNE%2bIwscsOe4hpqqLGQFjaVKkLVNrDLw7RxjRyqBTr%2fz%2f3t1OW8IYdekUIXnXataAVKXB7GAf0oBoyZ0MGh%2fnMIkE9kPIEO1YhmRbFupLbopJFtc6ekwY7%2fV%2bxKZ4kK0KXA0Kt1QR7H9olIrmbbyDsQer57AvwSlxhFjnruoMQ0VYkT1ZKnd0JfRByBpGt5F255iDDLvYVCaSXm2rpsxz%2b3FfrKnwLGeoygtszXvtABKN3Nwz%2fJ009ur1gYwbWtIZAVvGw53JEn%2fK9PJwSi3rvTthQlOWBPp%2bVJbwq24yWN3FB%2fLQTeAwByLgOeD8x0lnZkRVpL1PdInnTDOJ9YfTiI0%2fE24DyONIctvpB0x94zfBlSJBDcxz97509PgDCM%2bAMzTEgvwEO44wSMAIAAA%3d%3d)**\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1)
| mvexpand id=_users to typeof(long) limit 1000000
| where _day > datetime(2017-01-02)
| project _day, id
// Calculate weekly retention rate
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, retention_rate*100, churn_rate*100
| render timechart
```

### <a name="newactivitymetrics-plugin"></a>new_activity_metrics plugin

[**new_activity_metrics plugin**](https://docs.microsoft.com/azure/kusto/query/new-activity-metrics-plugin)：針對新使用者的世代，計算有用的活動計量 (相異計數值、新值的相異計數、保留率及流失率)。 此外掛程式的概念類似 [**activity_metrics plugin**](https://docs.microsoft.com/azure/kusto/query/activity-metrics-plugin)，但側重於新使用者。

下列查詢會針對新使用者世代 (本週已抵達的使用者) 逐週計算保留率與流失率。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1Ry27DIBC8W%2fI%2f7C04wbJJFeVQ5VapP9BbVVnIrGMaGyy8eVjqxxcwh1QqBx7LzCwzVBW8o0EnCcFJo%2bwISpIE28F1RgeyJX3TpHHOswEJmpmkIzgFFJIeke1rcSzrQ1mL4jVh0Kj%2fEC8R4bucEd7kAp3z3ZIg2ZU2E04gVJ79AD4oVIIU2cGaM2OBVSZKUQlVPOGcxwUHrNiJp3ITbMyn2JUlHbU91FtXcPhz3u1rP5fC10UUHm%2f4mLwiaHVaZcIzaZnQdiwQCxj0qAlEHUeeVRV8yAuCNcMC1CN02s0Ed8QLtLa33igbpK9M0skRCd3q4CaHa%2fgBg%2fcmJb40%2ft7pdmafG602XzxExpN3HsPicFQ8z1IcQWhy9htbisk2EU92XZ1vZkhb04Sv5tD2V7fufwFYtolnAgIAAA%3d%3d)**\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1)
| mvexpand id=_users to typeof(long) limit 1000000
// Take only the first week cohort (last parameter)
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d, _start)
| project from_Day, to_Day, retention_rate, churn_rate
```

### <a name="sessioncount-plugin"></a>session_count plugin

[**session_count plugin**](https://docs.microsoft.com/azure/kusto/query/session-count-plugin)：根據 ID 資料行，計算隨時間軸變化的工作階段計數。

下列查詢會傳回工作階段的計數。 如果使用者識別碼在有 100 個時段的時間範圍內至少出現一次，則該工作階段視為作用中，而工作階段回顧時間範圍則有 41 個時段。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWPQYvCQAyF74X%2bh3dZUCjYgfUgMkcP3r2XoZPqaM3INK4u7I%2ffzOwiNQRC8pKPl5EEnXfiYJEcHwmHcKUxMGFI8QoDidhoYBK6wdTVD%2bgpxB5dd6FvPSuzcwyMS2BvAzMlLP5gez%2fDrNt%2fCN4Z1iwRua5Kk2GPE6WZkY%2bMsRZt1m4pnqmXl9qouK2r1Qo75cUB5RlPQ%2bAgoWDzpPj%2bcuPdCWGiaVKp6%2bOdZbH3zYxmNFuNUhp8mmU%2bTWpWv8or%2fckl%2bQXutT48NwEAAA%3d%3d)**\]**

```Kusto
let _data = range Timeline from 1 to 9999 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// End of data definition
_data
| evaluate session_count(Id, Timeline, 1, 10000, 100, 41)
| render linechart
```

### <a name="funnelsequence-plugin"></a>funnel_sequence plugin

[**funnel_sequence plugin**](https://docs.microsoft.com/azure/kusto/query/funnel-sequence-plugin)：計算已採用一系列狀態的使用者相異計數；顯示在序列之前的上個狀態與下個狀態的分布情形。

下列查詢會顯示 2007 年在所有龍捲風事件之前與之後發生的事件。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAGWOPYvCQBCG%2b0D%2bw3QmEEmieIqNVQrBRgxYiMhcdqKLyWzcnQiCP95V70DuYIrh5Xk%2f0hRWxpw1H8EwbMTYtrgSiwMnKNqJrtw8DNIU1vkcticUOGHXETv4ptpYgtJYRmWAnrbFGx39QbEWsv%2fIj7YwuHsZmx6FoO6ZqTk4uvTEFUVFp51RtFSJH4hWSt1SAsqj4r9olGXTYZb7i5Mw%2bJRnvzLkKhl%2fTXzAq668dc%2bAG2Orq2g3%2bBk22MfxA23MLGQQAQAA)**\]**

```Kusto
// Looking on StormEvents statistics:
// Q1: What happens before Tornado event?
// Q2: What happens after Tornado event?
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, dynamic(['Tornado']))
```

### <a name="funnelsequencecompletion-plugin"></a>funnel_sequence_completion plugin

[**funnel_sequence_completion plugin**](https://docs.microsoft.com/azure/kusto/query/funnel-sequence-completion-plugin)：計算在不同時段內已完成序列步驟的漏斗圖。

下列查詢會檢查序列的完成漏斗圖：在一小時、四小時及一天 (`[1h, 4h, 1d]`)「整體」時間中的 `Hail -> Tornado -> Thunderstorm -> Wind`。

**\[**[**按一下以執行查詢**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA12QTYvCMBCG74L/YW6tkIV2XT9g8SjsnlvwICKhM9JAOqlJqrj4402CW0RIIB/PPLwzmjwcnZfWwwZQevKqo/yzKFYfRRnW7Hs60ZEhxjdi/UZcFaO5VuqPAjhfLvD/w9F5IG7iM95YdqrJ99mPVDoTkNXGskSTju3ASNZ5Y7t43wVhdhj9PVll0L1aylbAV9glJqyKldsLsXfTyR3oIvUQAsNpYCY95jg2puuDUhnOt71yBukXBVRxCnVoTjwnIlLX4rUzAUlf3/pEPYViDDd7AOyqowFQAQAA)**\]**

```Kusto
let _start = datetime(2007-01-01);
let _end = datetime(2008-01-01);
let _windowSize = 365d;
let _sequence = dynamic(['Hail', 'Tornado', 'Thunderstorm', 'Wind']);
let _periods = dynamic([1h, 4h, 1d]);
StormEvents
| evaluate funnel_sequence_completion(EpisodeId, StartTime, _start, _end, _windowSize, EventType, _sequence, _periods)
```

## <a name="functions"></a>Functions

本節說明[**函式**](https://docs.microsoft.com/azure/kusto/query/functions)：儲存在伺服器上可重複使用的查詢。 函式可由查詢或其他函式叫用 (不支援遞迴函式)。

> [!NOTE]
> 您無法在說明叢集上建立函式，因為該叢集是唯讀的。 針對此組件使用您自己的測試叢集。

下列範例會建立一個採用狀態名稱 (`MyState`) 作為引數的函式。

```Kusto
.create function with (folder="Demo")
MyFunction (MyState: string)
{
StormEvents
| where State =~ MyState
}
```

下列範例會呼叫一個會取得德州資料的函式。

```Kusto
MyFunction ("Texas")
| summarize count()
```

下列範例會刪除在第一個步驟中建立的函式。

```Kusto
.drop function MyFunction
```

## <a name="next-steps"></a>後續步驟

[Kusto 查詢語言參考](https://aka.ms/kustolangref) (英文)
