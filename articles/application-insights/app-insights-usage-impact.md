---
title: Azure Application Insights 使用情況影響 | Microsoft Docs
description: 分析不同的屬性可能會對您應用程式組件的轉換率產生哪些影響。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 01/25/2018
ms.reviewer: daviste
ms.author: mbullwin
ms.openlocfilehash: 9188776fdd213f01523069b08bd898f48bee57a4
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091476"
---
# <a name="impact-analysis-with-application-insights"></a>使用 Application Insights 進行影響分析

「影響」可分析載入時間和其他屬性會對您應用程式各個組件的轉換率產生哪些影響。 為了讓分析結果更準確，它會探索**分頁檢視**的**任何維度**、**自訂事件**或**要求**會對不同的**頁面檢視**或**自訂事件**的使用情況產生什麼樣的影響。 

![影響工具](./media/app-insights-usage-impact/0001-impact.png)

## <a name="still-not-sure-what-impact-does"></a>仍不確定「影響」的功能嗎？

您可以將「影響」想成是團隊中某個成員用於設定引數的終極工具，該引數則和您網站中一些緩慢執行層面會對使用者停留造成哪些影響有關。 使用者可能可以忍受某一個程度的緩慢執行速度，「影響」能夠提供深入解析，讓您了解如何在最佳化和效能之間取得平衡，以將使用者轉換最大化。

但是，分析效能只是「影響」的功能子集。 由於「影響」支援自訂事件和維度，因此只要按幾下就能解決像是使用者瀏覽器選擇與不同轉換率之間有哪些關聯的問題。

![依瀏覽器的轉換螢幕擷取畫面](./media/app-insights-usage-impact/0004-browsers.png)

> [!NOTE]
> 您的 Application Insights 資源必須包含頁面檢視或自訂事件，以便使用「影響」工具。 [了解如何設定應用程式以使用 Application Insights JavaScript SDK 自動收集分頁檢視](app-insights-javascript.md)。 也請記住，因為您是要分析相互關聯，所以樣本大小很重要。
>
>

## <a name="is-page-load-time-impacting-how-many-people-convert-on-my-page"></a>頁面載入時間是否會影響頁面上轉換的使用者人數？

若要開始使用「影響」工具回答問題，請選擇一個初始頁面檢視、自訂事件或要求。

![影響工具](./media/app-insights-usage-impact/0002-dropdown.png)

1. 從 [針對頁面檢視] 下拉式清單中選取頁面檢視。
2. 保留 [分析其] 下拉式清單中的預設選取項目 [持續時間] \(在此內容中，[持續時間] 為 [頁面載入時間] 的別名。)
3. 對於 [影響下列項目的使用量] 下拉式清單中，選取一個自訂事件。 此事件應與您在步驟 1 所選取頁面檢視上的 UI 元素相對應。

![結果的螢幕擷取畫面](./media/app-insights-usage-impact/0003-results.png)

在這個執行個體中，因為 [產品頁面] 載入時間增加了，所以 [購買按一下的產品] 的轉換率就降低了。 依據上述分配，可將最佳化頁面載入持續時間 3.5 秒作為目標，以達到 55% 轉換率。 進一步增強效能以將載入時間縮短至 3.5 秒以下的作法，目前無法提供額外的轉換優勢。

## <a name="what-if-im-tracking-page-views-or-load-times-in-custom-ways"></a>如果我用自訂方式追蹤頁面檢視或載入時間呢？

「影響」支援標準與自訂屬性和測量。 您可以自由使用。 不要使用持續時間，請改為在主要和次要事件上使用篩選以獲得更有用的資訊。

## <a name="do-users-from-different-countries-or-regions-convert-at-different-rates"></a>來自不同國家或地區的使用者，其轉換率會不一樣嗎？

1. 從 [針對頁面檢視] 下拉式清單中選取頁面檢視。
2. 在 [分析其] 下拉式清單中，選擇 [國家或地區]
3. 針對 [影響下列項目的使用量] 下拉式清單，請選取和您在步驟 1 中於頁面檢視上所選 UI 元素相對應的自訂事件。

在此案例中，結果已不再適用於它們在第一個範例中的連續 X 軸模型。 相反地，會顯示與區段漏斗圖類似的視覺效果。 依據 [使用量] 排序以檢視您的自訂事件依據國家所呈現的轉換變化。


## <a name="how-does-the-impact-tool-calculate-these-conversion-rates"></a>「影響」工具如何計算這些轉換率？

「影像」工具背後的運作原理是以[皮爾森相互關聯係數] (https://en.wikipedia.org/wiki/Pearson_correlation_coefficient) 為依據。 計算結果會介於-1 與 1 之間，-1 代表零關聯，1 則代表正向關聯。

「影響分析」運作方式的基本細節如下：

讓 _A_ = 您在第一個下拉式清單中選取的主要頁面檢視/自訂事件/要求。 ([針對頁面檢視])。

讓 _B_ = 您選取的次要頁面檢視/自訂事件 ([影響下列項目的使用量])。

「影響」會查看所選時間範圍中使用者所有工作階段的範例。 針對每個工作階段，它查看每個出現的 _A_。

然後工作階段會根據兩個條件的其中之一，分成兩種不同類型的_子工作階段_：

- 轉換的子工作階段，由以 _B_ 事件作為結尾的工作階段所組成，並包含在 _B_ 之前發生的所有 _A_ 事件。
- 當所有 _A_ 在沒有終端機 _B_ 的情況下發生時，就會發生未轉換的子工作階段。

「影響」最終計算方式會視我們依計量或維度進行分析而不同。 針對計量，會將子工作階段中的所有 _A_ 平均計算。 針對維度，則是每個 _A_ 的值會將 _1/N_ 提供給指派給 _B_ 的值，其中 _N_ 為子工作階段中 _A_ 的數目。

## <a name="next-steps"></a>後續步驟

- 若要啟用使用體驗，請開始傳送[「自訂事件」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) 或 [「頁面檢視」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views)。
- 如果您已傳送自訂事件或頁面檢視，請探索「使用量工具」，以了解使用者如何使用您的服務。
    - [漏斗圖](usage-funnels.md)
    - [保留](app-insights-usage-retention.md)
    - [使用者流程](app-insights-usage-flows.md)
    - [活頁簿](app-insights-usage-workbooks.md)
    - [新增使用者內容](app-insights-usage-send-user-context.md)