---
title: Azure 監視器的 Log Analytics 語言參考 | Microsoft Docs
description: Log Analytics 所使用的 Kusto 語言參考資訊。 包含 Log Analytics 專用的其他元素和 Log Analytics 查詢中不支援的元素。
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
ms.topic: article
ms.date: 09/25/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 5173790436a29fa9947346d711da1a2ddb32bf62
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451063"
---
# <a name="log-analytics-query-language-reference"></a>Log Analytics 查詢語言參考
[Log Analytics 查詢](../log-analytics-queries.md)使用的查詢語言和引擎與 [Azure 資料總管](/azure/data-explorer/)所用的相同。 您可以從下列位置存取語言參考和語言的其他詳細資料：[Kusto 語言參考](/azure/kusto/query)



## <a name="kusto-elements-not-support-in-log-analytics"></a>Log Analytics 中不支援的 Kusto 元素
雖然 Log Analytics 查詢會使用 Kusto 的實作，但有一些 Kusto 元素是不受支援的，如下列各節所述。

### <a name="statements-not-supported-in-log-analytics"></a>Log Analytics 中不支援的陳述式

* [別名](/kusto/query/aliasstatement)
* [查詢參數](/azure/kusto/query/queryparametersstatement)

### <a name="functions-not-supported-in-log-analytics"></a>Log Analytics 中不支援的函式

* [cluster()](/azure/kusto/query/clusterfunction)
* [cursor_after()](/azure/kusto/query/cursorafterfunction)
* [cursor_before_or_at()](/azure/kusto/query/cursorbeforeoratfunction)
* [cursor_current()、current_cursor()](/azure/kusto/query/cursorcurrent)
* [database()](/azure/kusto/query/databasefunction)
* [current_principal()](/azure/kusto/query/current-principalfunction)
* [extent_id()](/azure/kusto/query/extentidfunction)
* [extent_tags()](/azure/kusto/query/extenttagsfunction)

### <a name="operators-not-supported-in-log-analytics"></a>Log Analytics 中不支援的運算子

* [跨叢集聯結](/azure/kusto/query/joincrosscluster)
* [externaldata 運算子](/azure/kusto/query/externaldata-operator)

### <a name="plugins-not-supported-in-log-analytics"></a>Log Analytics 中不支援的外掛程式

* [sql_request 外掛程式](/azure/kusto/query/sqlrequestplugin)


## <a name="additional-operators-in-log-analytics"></a>Log Analytics 中的其他運算子
為了支援特定 Log Analytics 功能，下列所提供的額外 Kusto 運算子無法在 Log Analytics 以外的地方使用。 

* [app()](app-expression.md)
* [workspace()](workspace-expression.md)

## <a name="next-steps"></a>後續步驟

- 了解 [Log Analytics](../log-analytics-queries.md) 中的查詢。
- 逐步完成撰寫 [Log Analytics 查詢](/log-analytics/query-language/get-started-queries.md)的課程。
- 存取完整的 [Kusto 參考文件](/azure/kusto/query/)。