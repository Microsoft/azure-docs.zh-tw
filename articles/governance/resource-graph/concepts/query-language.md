---
title: 了解 Azure Resource Graph 查詢語言
description: 說明 Azure Resource Graph 查詢語言的運作方式。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: resource-graph
manager: carmonm
ms.openlocfilehash: 3600fc47a0fb318a49c1b37722cb7fffa51ec6f2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951945"
---
# <a name="understanding-the-azure-resource-graph-query-language"></a>了解 Azure Resource Graph 查詢語言

Azure Resource Graph 查詢語言支援多個運算子與函式。 每個項目的運作和操作方式都類似於 Kusto 查詢語言 (KQL)。 不過，請務必了解雖然 Resource Graph 的查詢語言類似於 [KQL](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators)，但兩者並不相同。

> [!NOTE]
> KQL 文件的連結可能需要驗證。

了解 Resource Graph 所使用查詢語言的最佳方式是從 KQL 文件開始著手，了解如何建構該語言以及各種不同支援的運算子與函式如何一起運作。

## <a name="supported-tabular-operators"></a>支援的表格式運算子

以下是 Resource Graph 中支援的表格式運算子清單：

- [distinct](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/distinct-operator)
- [extend](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/extend-operator)
- [limit](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/limit-operator)
- [order by](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/order-operator)
- [project](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/project-operator)
- [project-away](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/project-away-operator)
- [sample](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/sample-operator)
- [sample-distinct](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/sample-distinct-operator)
- [sort by](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/sort-operator)
- [summarize](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/summarize-operator)
- [take](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/take-operator)
- [top](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/top-operator)
- [top-nested](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/top-nested-operator)
- [top-hitters](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/top-hitters-operator)
- [where](https://docs.loganalytics.io/docs/Language-Reference/Tabular-operators/where-operator)

## <a name="supported-functions"></a>支援的函式

以下是 Resource Graph 中支援的函式清單：

- [ago()](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/ago%28%29)
- [buldschema()](https://docs.loganalytics.io/docs/Language-Reference/Aggregation-functions/buildschema%28%29)
- [count()](https://docs.loganalytics.io/docs/Language-Reference/Aggregation-functions/count%28%29)
- [strcat()](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/strcat%28%29)
- [isnotempty()](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/isnotempty%28%29_-notempty%28%29)
- [tostring()](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/tostring%28%29)

## <a name="next-steps"></a>後續步驟

- 請參閱[入門查詢](../samples/starter.md)中使用的語言
- 請參閱[進階查詢](../samples/advanced.md)中的進階使用方式
- 了解[探索資源](explore-resources.md)