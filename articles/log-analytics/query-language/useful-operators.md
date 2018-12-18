---
title: Azure Log Analytics 查詢中的實用運算子 | Microsoft Docs
description: 在 Log Analytics 查詢中用於不同案例的常見函式。
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
ms.openlocfilehash: 0b14c13462f15dd90285ed9e37080487324a4d85
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831274"
---
# <a name="useful-operators-in-log-analytics-queries"></a>Log Analytics 查詢中的實用運算子

下表提供在 Log Analytics 查詢中用於不同案例的一些常見函式。

## <a name="useful-operators"></a>實用運算子

類別                                |相關分析函式
----------------------------------------|----------------------------------------
選取項目和資料行別名            |`project`、`project-away`, `extend`
暫存資料表和常數          |`let scalar_alias_name = …;` <br> `let table_alias_name =  …  …  … ;`| 
比較和字串運算子         |`startswith`、`!startswith`、`has`、`!has` <br> `contains`、`!contains`, `containscs` <br> `hasprefix`、`!hasprefix`、`hassuffix`、`!hassuffix`、`in`、`!in` <br> `matches regex` <br> `==`、`=~`、`!=`、`!~`
常見字串函式                 |`strcat()`、`replace()`、`tolower()`、`toupper()`、`substring()`、`strlen()`
常見數學函式                   |`sqrt()`、`abs()` <br> `exp()`、`exp2()`、`exp10()`、`log()`、`log2()`、`log10()`、`pow()` <br> `gamma()`、`gammaln()`
剖析文字                            |`extract()`、`extractjson()`、`parse`、`split()`
限制輸出                         |`take`、`limit`、`top`、`sample`
日期函式                          |`now()`、`ago()` <br> `datetime()`、`datepart()`, `timespan` <br> `startofday()`、`startofweek()`、`startofmonth()`、`startofyear()` <br> `endofday()`、`endofweek()`、`endofmonth()`、`endofyear()` <br> `dayofweek()`、`dayofmonth()`, `dayofyear()` <br> `getmonth()`、`getyear()`、`weekofyear()`、`monthofyear()`
群組和彙總                |`summarize by` <br> `max()`、`min()`、`count()`、`dcount()`、`avg()`、`sum()` <br> `stddev()`、`countif()`、`dcountif()`、`argmax()`、`argmin()` <br> `percentiles()`、`percentile_array()`
聯結和集合聯集                        |`join kind=leftouter`、`inner`、`rightouter`、`fullouter`、`leftanti` <br> `union`
排序，次序                             |`sort`、`order` 
動態物件 (JSON 和陣列)         |`parsejson()` <br> `makeset()`、`makelist()` <br> `split()`、`arraylength()` <br> `zip()`、`pack()`
邏輯運算子                       |`and`、`or`, `iff(condition, value_t, value_f)` <br> `binary_and()`、`binary_or()`、`binary_not()`、`binary_xor()`
機器學習服務                        |`evaluate autocluster`、`basket`、`diffpatterns`、`extractcolumns`


## <a name="next-steps"></a>後續步驟

- 進行[在 Log Analytics 中撰寫查詢](get-started-queries.md)課程。
