---
title: Azure Application Insights 遙測資料模型 - 要求遙測 | Microsoft Docs
description: 要求遙測的 Application Insights 資料模型
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
ms.openlocfilehash: 6cee6db66fc4146e9c799394e40c72ab2ce665dc
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47092930"
---
# <a name="request-telemetry-application-insights-data-model"></a>要求遙測：Application Insights 資料模型

要求遙測項目 (在 [Application Insights](app-insights-overview.md) 中) 代表外部要求對應用程式所觸發的執行邏輯順序。 每個要求執行都是由包含所有執行參數的唯一 `ID` 和 `url` 所識別。 您可以將這些要求依邏輯 `name` 群組，並定義這項要求的 `source`。 程式碼執行可能會導致 `success` 或 `fail`，並且有特定 `duration`。 成功和失敗的執行都可以進一步以 `resultCode` 分組。 信封層級上定義之要求遙測的開始時間。

要求遙測會使用自訂 `properties` 和 `measurements`支援標準的擴充性模型。

## <a name="name"></a>名稱

要求的名稱代表處理要求所採用的程式碼路徑。 較低的基數值可使群組要求更妥善。 針對 HTTP 要求，它代表 HTTP 方法和 URL 路徑範本，例如無實際 `id` 值的 `GET /values/{id}`。

Application Insights web SDK 會將要求名稱依「現狀」傳送 (考量字母大小寫)。 UI 上的群組會區分大小寫，因此 `GET /Home/Index` 會與 `GET /home/INDEX` 分開計算，即使它們通常會產生相同的控制器和動作執行。 原因是 URL 通常會[區分大小寫](http://www.w3.org/TR/WD-html40-970708/htmlweb.html)。 您可能要查看 URL 出現的所有 `404` 是否都以大寫輸入。 您可以在[部落格文章](http://apmtips.com/blog/2015/02/23/request-name-and-url/)閱讀更多由 ASP.Net Web SDK 要求名稱集合的相關資訊。

最大長度︰1024 個字元

## <a name="id"></a>ID

要求呼叫執行個體的識別碼。 用於要求和其他遙測項目之間的相互關聯。 識別碼必須是全域唯一的。 如需詳細資訊，請參閱[相互關聯](application-insights-correlation.md)頁面。

最大長度︰128 個字元

## <a name="url"></a>Url

包含所有查詢字串參數的要求 URL。

最大長度︰2048 個字元

## <a name="source"></a>來源

要求的來源。 範例包括呼叫端的檢測金鑰或呼叫端的 IP 位址。 如需詳細資訊，請參閱[相互關聯](application-insights-correlation.md)頁面。

最大長度︰1024 個字元

## <a name="duration"></a>Duration

要求持續時間格式為︰`DD.HH:MM:SS.MMMMMM`。 必須是正數且小於 `1000` 天。 這是必要欄位，因為要求遙測代表開頭與結尾的作業。

## <a name="response-code"></a>Response code

要求執行的結果。 HTTP 要求的 HTTP 狀態碼。 可能是 `HRESULT` 值或其他要求類型的例外狀況類型。

最大長度︰1024 個字元

## <a name="success"></a>成功

表示成功或失敗的呼叫。 這是必填欄位。 當未明確設定為 `false` 時 - 要求會視為成功。 如果作業是例外狀況而中斷或傳回錯誤結果碼，將此值設定為 `false`。

針對 Web 應用程式，當回應碼小於 `400` 或等於 `401` 時，Application Insights 會將要求定義為失敗。 不過有可能會發生這個預設對應與應用程式之語意不相符的情況。 回應碼 `404` 可能表示「沒有記錄」，這可能是一般流程的一部分。 它也可能表示中斷的連結。 針對中斷連結，您甚至可以實作更進階的邏輯。 只有當中斷的連結藉由分析 URL 查閱者位於相同網站時，您才可以將這些連結標示為失敗。 或是從公司的行動應用程式存取時，才可以將它們標示為失敗。 同樣地，從不支援重新導向的用戶端存取時，`301` 和 `302` 會表示失敗。

部分接受的內容 `206` 可能表示整體要求失敗。 例如，Application Insights 端點會接收一批遙測項目作為單一要求。 當批次中的某些項目未處理成功時，它會傳回 `206`。 `206` 的速率遞增表示有需要調查的問題。 類似的邏輯也適用於 `207` 多狀態，當中成功可能是不同回應代碼的最差狀況。

您可以在[部落格文章](http://apmtips.com/blog/2016/12/03/request-success-and-response-code/)中閱讀更多要求結果碼和狀態碼的相關資訊。

## <a name="custom-properties"></a>自訂屬性

[!INCLUDE [application-insights-data-model-properties](../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>自訂度量

[!INCLUDE [application-insights-data-model-measurements](../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>後續步驟

- [撰寫自訂要求遙測](app-insights-api-custom-events-metrics.md#trackrequest)
- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- 了解如何使用 Application Insights [設定 ASP.NET Core](app-insights-asp-net.md) 應用程式。
- 查看 Application Insights 支援的[平台](app-insights-platforms.md)。
