---
title: Azure Application Insights 遙測資料模型 - 例外狀況遙測 | Microsoft Docs
description: 例外狀況遙測的 Application Insights 資料模型
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
ms.openlocfilehash: dc5480b90ef6b5520f47c51f0c105202d7071089
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47093588"
---
# <a name="exception-telemetry-application-insights-data-model"></a>例外狀況遙測：Application Insights 資料模型

在 [Application Insights](app-insights-overview.md) 中，例外狀況的執行個體代表受監視應用程式執行期間所發生的已處理或未處理例外狀況。

## <a name="problem-id"></a>問題識別碼

例外狀況在程式碼中擲回位置的識別項。 用於例外狀況群組。 通常為例外狀況類型和呼叫堆疊中之函式的組合。

最大長度︰1024 個字元

## <a name="severity-level"></a>嚴重性層級

追蹤嚴重性層級。 值可為 `Verbose`、`Information`、`Warning`、`Error`、`Critical`。

## <a name="exception-details"></a>例外狀況詳細資料

(等候進一步擴充)

## <a name="custom-properties"></a>自訂屬性

[!INCLUDE [application-insights-data-model-properties](../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>自訂度量

[!INCLUDE [application-insights-data-model-measurements](../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>後續步驟

- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- 了解如何[使用 Application Insights 在 Web Apps 中診斷例外狀況](app-insights-asp-net-exceptions.md)。
- 查看 Application Insights 支援的[平台](app-insights-platforms.md)。
