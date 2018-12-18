---
title: Log Analytics 中的搜尋查詢 | Microsoft Docs
description: 本文提供教學課程來說明如何在 Log Analytics 中開始撰寫搜尋查詢。
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
ms.date: 08/06/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 250eddb043ccf9fa0b1bb92a298900f8ad820140
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992264"
---
# <a name="search-queries-in-log-analytics"></a>Log Analytics 中的搜尋查詢

> [!NOTE]
> 請先完成[在 Log Analytics 中開始使用查詢](get-started-queries.md)再完成本課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

Azure Log Analytics 查詢可以透過資料表名稱或 search 命令來開始。 本教學課程涵蓋以搜尋為基礎的查詢。 每種方法各有其優點。

以資料表為基礎的查詢一開始會界定查詢範圍，因此，通常會比搜尋查詢更有效率。 搜尋查詢的結構較為鬆散，因此更適合用於搜尋資料行或資料表中的特定值。 **search** 可以掃描指定資料表或所有資料表中的所有資料行，查看其中是否有指定值。 所要處理的資料量可能會很大，因此這些查詢可能需要較長的時間才能完成，而且可能會傳回非常大型的結果集。

## <a name="search-a-term"></a>搜尋字詞
**search** 命令通常用來搜尋特定字詞。 在下列範例中，所有資料表中的所有資料行都會進行掃描，查看其中是否有「error」字詞：

```Kusto
search "error"
| take 100
```

未界定範圍的查詢 (如上述查詢) 雖然容易使用，但效率卻很差，並且可能會傳回許多不相關的結果。 比較好的做法是在相關資料表或甚至特定資料行中進行搜尋。

### <a name="table-scoping"></a>界定資料表範圍
若要在特定資料表中搜尋字詞，請在 **search** 運算子正後方新增 `in (table-name)`：

```Kusto
search in (Event) "error"
| take 100
```

或者，在多個資料表中：
```Kusto
search in (Event, SecurityEvent) "error"
| take 100
```

### <a name="table-and-column-scoping"></a>界定資料表和資料行的範圍
根據預設，**search** 會評估資料集內的所有資料行。 若只想搜尋特定資料行，請使用此語法：

```Kusto
search in (Event) Source:"error"
| take 100
```

> [!TIP]
> 如果您使用 `==` 而非 `:`，結果中會包含 *Source* 資料行具有確切值「error」、且在這個確切案例中的記錄。 使用 ':' 則不會包含 Source 具有「error code 404」或「Error」等值的記錄。

## <a name="case-sensitivity"></a>區分大小寫
根據預設，字詞搜尋不會區分大小寫，因此，搜尋 "dns" 可能會產生 "DNS"、"dns" 或 "Dns" 等結果。 若要讓搜尋區分大小寫，請使用 `kind` 選項：

```Kusto
search kind=case_sensitive in (Event) "DNS"
| take 100
```

## <a name="use-wild-cards"></a>使用萬用字元
**search** 命令支援在字詞開頭、結尾或中間使用萬用字元。

若要搜尋開頭為 "win" 的字詞：
```Kusto
search in (Event) "win*"
| take 100
```

若要搜尋結尾為 ".com" 的字詞：
```Kusto
search in (Event) "*.com"
| take 100
```

若要搜尋包含 "www" 的字詞：
```Kusto
search in (Event) "*www*"
| take 100
```

若要搜尋開頭為 "corp"、結尾為 ".com" 的字詞，例如 "corp.mydomain.com""

```Kusto
search in (Event) "corp*.com"
| take 100
```

您也可以只使用萬用字元來取得資料表中的所有項目：`search in (Event) *`，但其結果和純粹撰寫 `Event` 相同。

> [!TIP]
> 雖然您可以使用 `search *` 來取得每個資料表中的每個資料行，但建議您一律將查詢範圍界定在特定資料表。 未界定範圍的查詢可能需要一段時間才能完成，而且可能會傳回太多結果。

## <a name="add-and--or-to-search-queries"></a>對搜尋查詢新增 and / or
使用 **and** 可搜尋包含多個字詞的記錄：

```Kusto
search in (Event) "error" and "register"
| take 100
```

使用 **or** 可取得包含至少一個字詞的記錄：

```Kusto
search in (Event) "error" or "register"
| take 100
```

如果您有多個搜尋條件，則可以使用括號將其合併到相同查詢：

```Kusto
search in (Event) "error" and ("register" or "marshal*")
| take 100
```

此範例的結果中所含有的記錄，會包含「error」字詞且同時包含「register」或以「marshal」開頭的字詞。

## <a name="pipe-search-queries"></a>垂直線搜尋查詢
和任何其他命令一樣，**search** 也可以使用垂直線，以便對搜尋結果進行篩選、排序和彙總。 例如，若要取得包含「win」的 Event 記錄數目：

```Kusto
search in (Event) "win"
| count
```




## <a name="next-steps"></a>後續步驟

- 請參閱關於 [Log Analytics 查詢語言網站](https://aka.ms/LogAnalyticsLanguage)的進階教學課程。