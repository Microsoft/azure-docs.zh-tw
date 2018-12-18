---
title: Azure Application Insights 中的使用者、工作階段和事件分析 | Microsoft Docs
description: 您 Web 應用程式的使用者人口統計分析。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 01/24/2018
ms.reviewer: daviste
ms.author: mbullwin
ms.openlocfilehash: 2bc10a292855832b7ddb9b8e3a59fbe0f17d8dc6
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47093803"
---
# <a name="users-sessions-and-events-analysis-in-application-insights"></a>Application Insights 中的使用者、工作階段和事件分析

了解人們在使用您的 Web 應用程式時，他們最感興趣的網頁、使用者的所在位置，以及他們使用何種瀏覽器與作業系統。 使用 [Azure Application Insights](app-insights-overview.md) 來分析商務和使用量遙測。

![Application Insights 使用者的螢幕擷取畫面](./media/app-insights-usage-segmentation/0001-users.png)

## <a name="get-started"></a>開始使用

如果您在 Application Insights 入口網站中尚未看到 [使用者]、[工作階段] 或 [事件] 刀鋒視窗中的資料，[請了解如何開始使用使用量工具](app-insights-usage-overview.md)。

## <a name="the-users-sessions-and-events-segmentation-tool"></a>「使用者」、「工作階段」和「事件」分割工具

三個 [使用量] 刀鋒視窗會使用相同的工具，從三個方面將您 Web 應用程式的遙測進行交叉分析。 透過將資料進行篩選及分割，您會發現關於不同頁面和功能之相對使用方式的深入解析。

* **使用者工具**︰使用您應用程式的使用者數目和應用程式的功能。  會使用儲存在瀏覽器 Cookie 中的匿名識別碼來計算使用者。 使用不同瀏覽器或電腦的單一使用者將會計算為多個使用者。
* **工作階段工具**︰已包含應用程式的特定頁面和功能之使用者活動的工作階段數目。 在使用者閒置半小時或連續使用 24 小時之後，系統就會計算工作階段。
* **事件工具**︰您應用程式的特定頁面與功能之使用頻率。 當瀏覽器從您的應用程式載入頁面時 (假如您已[將它進行檢測](app-insights-javascript.md)) 就會計算頁面檢視。 

    自訂事件代表您應用程式中的事情發生一次，通常是諸如按一下按鈕或完成某些工作等使用者互動。 將程式碼插入您的應用程式可[產生自訂事件](app-insights-api-custom-events-metrics.md#trackevent)。

## <a name="querying-for-certain-users"></a>查詢特定使用者

瀏覽不同群組的使用者，方法是調整 [使用者] 工具頂端的查詢選項︰

* 顯示：選擇要分析之使用者的世代。
* 已使用者︰選擇自訂事件和頁面檢視。
* 期間︰選擇時間範圍。
* 方法︰選擇將資料貯體的方式，可依一段時間或依另一個屬性 (例如瀏覽器或城市)。
* 分割方法︰選擇要用來分割或區隔資料的屬性。 
* 新增篩選條件︰將查詢根據其屬性 (例如瀏覽器或城市) 限制為特定使用者、工作階段或事件。 
 
## <a name="saving-and-sharing-reports"></a>儲存和共用報告 
您可以透過下列方式來儲存使用者報告：儲存在 [我的報告] 區段中，僅供您自己私人使用；或儲存在 [共用的報告] 區段中，與擁有 Application Insights 資源存取權的其他所有人共用。

若要共用使用者、工作階段或事件報告的連結，請按一下工具列中的 [共用]，然後複製連結。

若要共用使用者、工作階段或事件報告的資料複本，請按一下工具列中的 [共用]，然後按一下 [Word] 圖示以建立具有該資料的 Word 文件。 或者，按一下主要圖表上方的 [Word] 圖示。

## <a name="meet-your-users"></a>會見使用者

[會見使用者] 區段會顯示目前查詢所比對出之五位範例使用者的相關資訊。 除了彙總之外，考量並瀏覽個人行為可深入解析使用者實際使用您應用程式的方式。

## <a name="next-steps"></a>後續步驟

- 若要啟用使用體驗，請開始傳送[「自訂事件」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) 或 [「頁面檢視」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views)。
- 如果您已傳送自訂事件或頁面檢視，請探索「使用量工具」，以了解使用者如何使用您的服務。
    - [漏斗圖](usage-funnels.md)
    - [保留](app-insights-usage-retention.md)
    - [使用者流程](app-insights-usage-flows.md)
    - [活頁簿](app-insights-usage-workbooks.md)
    - [新增使用者內容](app-insights-usage-send-user-context.md)