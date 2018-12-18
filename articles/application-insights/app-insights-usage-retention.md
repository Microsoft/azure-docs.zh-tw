---
title: 使用 Azure Application Insights 進行 Web 應用程式的使用者保留期分析 | Microsoft Docs
description: 有多少使用者會回來使用您的應用程式？
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 05/03/2017
ms.author: mbullwin
ms.openlocfilehash: 6e042034d52551d58e290b54ca3547dedbc89c47
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096933"
---
# <a name="user-retention-analysis-for-web-applications-with-application-insights"></a>使用 Application Insights 進行 Web 應用程式的使用者保留期分析

[Azure Application Insights](app-insights-overview.md) 中的保留功能可協助您分析回來使用您應用程式的使用者人數，以及他們執行特定工作或達成目標的頻率。 例如，如果您執行遊戲網站，您可以比較使用者在遊戲落敗與遊戲獲勝後，回到網站的數目。 這項知識可以協助您改善使用者體驗和商務策略。

## <a name="get-started"></a>開始使用

如果您在 Application Insights 入口網站中尚未看到保留工具中的資料，請[了解如何開始使用使用量工具](app-insights-usage-overview.md)。

## <a name="the-retention-tool"></a>保留期工具

![保留期工具](./media/app-insights-usage-retention/retention.png)

1. 工具列可讓使用者建立新的保留報告、開啟現有的保留報告、儲存目前保留報告或另存新檔、還原對已儲存之報表所做的變更、重新整理報表上的資料、透過電子郵件或直接連結共用報表，以及存取文件頁面。 
2. 根據預設，保留期會顯示執行任何動作然後回來，並且在一段期間內未執行任何其他動作的所有使用者。 您可以選取不同組合的事件，以縮小特定使用者活動的關注範圍。
3. 在屬性新增一或多個篩選條件。 例如，您可以將重點放在特定國家或區域中的使用者。 設定篩選條件後，按一下 [更新]。 
4. 整體保留圖表顯示所選期間內的使用者保留摘要。 
5. 此格線根據 #2 中的查詢產生器顯示保留的使用者數目。 每個資料列代表在所顯示的時間週期內執行任何事件的同群使用者。 資料列中的每個資料格會顯示該同群使用者中在稍後期間內至少回來使用一次的數目。 有些使用者可能會在不只一個期間內回來使用。 
6. Insights 卡顯示前五個起始事件，以及前五個傳回的事件，讓使用者深入了解其保留報表。 

![保留滑鼠暫留](./media/app-insights-usage-retention/hover.png)

使用者可以將滑鼠停留在保留工具上的資料格，來存取會說明資料格所代表之意義的分析按鈕和工具提示。 [分析] 按鈕會帶領使用者前往具有預先填入查詢的分析工具，以從儲存格產生使用者。 

## <a name="use-business-events-to-track-retention"></a>使用商務事件來追蹤保留期

若要取得最有用的保留期分析，請測量代表重要商務活動的事件。 

例如，許多在應用程式中開啟頁面的使用者可能不會玩它所顯示的遊戲。 因此，僅追蹤頁面檢視所提供的先前玩過遊戲後再回來玩的人數估計可能會有誤。 若要取得舊玩家的明確內容，您的應用程式應該在使用者實際玩過遊戲時才傳送自訂事件。  

最佳做法是將代表重要商務動作的自訂事件編碼，並使用這些來進行保留期分析。 若要擷取遊戲的結果，您需要撰寫一行程式碼，將自訂事件傳送至 Application Insights。 如果您是以網頁程式碼或 Node.JS 來撰寫，它看起來會像這樣︰

```JavaScript
    appinsights.trackEvent("won game");
```

或是以 ASP.NET 伺服器程式碼撰寫：

```csharp
   telemetry.TrackEvent("won game");
```

[深入了解撰寫自訂事件](app-insights-api-custom-events-metrics.md#trackevent)。


## <a name="next-steps"></a>後續步驟
- 若要啟用使用體驗，請開始傳送[「自訂事件」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) 或 [「頁面檢視」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views)。
- 如果您已傳送自訂事件或頁面檢視，請探索「使用量工具」，以了解使用者如何使用您的服務。
    - [使用者、工作階段、事件](app-insights-usage-segmentation.md)
    - [漏斗圖](usage-funnels.md)
    - [使用者流程](app-insights-usage-flows.md)
    - [活頁簿](app-insights-usage-workbooks.md)
    - [新增使用者內容](app-insights-usage-send-user-context.md)


