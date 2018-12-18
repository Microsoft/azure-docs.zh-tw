---
title: Azure Application Insights 遙測資料模型 - 追蹤遙測 | Microsoft Docs
description: 追蹤遙測的 Application Insights 資料模型
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.author: mbullwin
ms.openlocfilehash: 2487df93baec80c0e4f7741aa7b052ac89459b3b
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633117"
---
# <a name="trace-telemetry-application-insights-data-model"></a>追蹤遙測：Application Insights 資料模型

追蹤遙測 (在 [Application Insights](app-insights-overview.md) 中) 代表以文字搜尋的 `printf` 樣式追蹤陳述式。 `Log4Net`、`NLog`和其他以文字為基礎的記錄檔項目會轉譯成此類型的執行個體。 追蹤沒有作為擴充性的度量。

## <a name="message"></a>訊息

追蹤訊息。

最大長度︰32768 個字元

## <a name="severity-level"></a>嚴重性層級

追蹤嚴重性層級。 值可為 `Verbose`、`Information`、`Warning`、`Error`、`Critical`。

## <a name="custom-properties"></a>自訂屬性

[!INCLUDE [application-insights-data-model-properties](../../includes/application-insights-data-model-properties.md)]

## <a name="next-steps"></a>後續步驟

- [在 Application Insights 中探索 .NET 追蹤記錄](app-insights-asp-net-trace-logs.md)。
- [在 Application Insights 中探索 Java 追蹤記錄](app-insights-java-trace-logs.md)。
- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- [撰寫自訂追蹤遙測](app-insights-api-custom-events-metrics.md#tracktrace)
- 查看 Application Insights 支援的[平台](app-insights-platforms.md)。
