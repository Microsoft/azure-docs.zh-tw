---
title: 使用 JavaScript API 與報告進行互動 | Microsoft Docs
description: Power BI JavaScript API 可讓您輕鬆地將 Power BI 報告內嵌到您的應用程式中。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: bdd885d3-1b00-4dcf-bdff-531eb1f97bfb
ms.service: power-bi-embedded
ms.topic: conceptual
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 32b72444c620b542262db322c5af94e69c49521e
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048668"
---
# <a name="interact-with-power-bi-reports-using-the-javascript-api"></a>使用 JavaScript API 與 Power BI 報告進行互動

Power BI JavaScript API 可讓您輕鬆地將 Power BI 報告內嵌到您的應用程式中。 使用此 API，您的應用程式可以程式設計方式與不同的報告元素 (例如頁面和篩選器) 互動。 此互動功能可讓 Power BI 報告更能夠融合到您的應用程式中。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

您可使用應用程式中裝載的 iframe，在應用程式中內嵌 Power BI 報告。 如下圖所示，iframe 可做為應用程式與報告之間的界限：

![Power BI 工作區集合 iframe (沒有 Javascript API)](media/interact-with-reports/iframe-without-javacript.png)

iframe 可以讓內嵌程序變得簡單多了，但是若沒有 JavaScript API，報告和應用程式便無法彼此互動。 欠缺這種互動會讓您覺得報告其實不是應用程式的一部分。 報告和應用程式真的需要往來通訊，如下圖所示：

![Power BI 工作區集合 iframe (有 Javascript API)](media/interact-with-reports/iframe-with-javascript.png)

Power BI JavaScript API 可讓您撰寫可安全地通過 iframe 界限的程式碼。 這可讓應用程式以程式設計方式在報告中執行動作，以及從使用者在報告中所進行的動作接聽事件。

## <a name="what-can-you-do-with-the-power-bi-javascript-api"></a>您可以使用 Power BI JavaScript API 來做什麼？

利用 JavaScript API，您可以管理報告、瀏覽至報告中的頁面、篩選報告，以及處理內嵌事件。 下圖顯示 API 的結構。

![Power BI JavaScript API 圖表](media/interact-with-reports/javascript-api-diagram.png)

### <a name="manage-reports"></a>管理報告
Javascript API 可讓您管理報告和頁面層級的行為︰

* 在應用程式中安全地內嵌特定 Power BI 報告 - 試用 [內嵌示範應用程式](http://azure-samples.github.io/powerbi-angular-client/#/scenario1)
  * 設定存取權杖
* 設定報告
  * 啟用和停用篩選器窗格和頁面導覽窗格 - 試用 [更新設定示範應用程式](http://azure-samples.github.io/powerbi-angular-client/#/scenario6)
  * 設定頁面和篩選器的預設值 - 試用 [預設值示範](http://azure-samples.github.io/powerbi-angular-client/#/scenario5)
* 進入或離開全螢幕模式

[深入了解內嵌報告](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embedding-Basics)

### <a name="navigate-to-pages-in-a-report"></a>瀏覽至報告中的頁面
JavaScript API enbales 可讓您探索報告中的所有頁面，以及設定目前的頁面。 試用 [瀏覽示範應用程式](http://azure-samples.github.io/powerbi-angular-client/#/scenario3)。

[深入了解頁面瀏覽](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Page-Navigation)

### <a name="filter-a-report"></a>篩選報告
JavaScript API 會提供內嵌報告和報告頁面的基本和進階篩選功能。 試用 [篩選示範應用程式](http://azure-samples.github.io/powerbi-angular-client/#/scenario4)，並檢閱以下一些入門程式碼。

#### <a name="basic-filters"></a>基本篩選器
基本篩選器置於資料行或階層層級，而且包含一份要包含或排除的值清單。

```
const basicFilter: pbi.models.IBasicFilter = {
  $schema: "http://powerbi.com/product/schema#basic",
  target: {
    table: "Store",
    column: "Count"
  },
  operator: "In",
  values: [1,2,3,4]
}
```

#### <a name="advanced-filters"></a>進階篩選器
進階篩選器使用邏輯運算子 AND 或 OR，並接受一或兩個條件，而每個條件都有自己的運算子和值。 支援的條件如下︰

* None
* LessThan
* LessThanOrEqual
* GreaterThan
* GreaterThanOrEqual
* Contains
* DoesNotContain
* StartsWith
* DoesNotStartWith
* Is
* IsNot
* IsBlank
* IsNotBlank

```
const advancedFilter: pbi.models.IAdvancedFilter = {
  $schema: "http://powerbi.com/product/schema#advanced",
  target: {
    table: "Store",
    column: "Name"
  },
  logicalOperator: "Or",
  conditions: [
    {
      operator: "Contains",
      value: "Wash"
    },
    {
      operator: "Contains",
      value: "Park"
    }
  ]
}
```

[深入了解篩選](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Filters)

### <a name="handling-events"></a>錯誤事件

除了將資訊傳送到 iframe，您的應用程式也可以接收來自 iframe 的下列事件相關資訊︰

* 內嵌
  * 已載入
  * 錯誤
* 報告
  * pageChanged
  * dataSelected (敬請期待)

[深入了解處理事件](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Handling-Events)

## <a name="next-steps"></a>後續步驟

如需 Power BI JavaScript API 的詳細資訊，請查看下列連結：

* [JavaScript API Wiki](https://github.com/Microsoft/PowerBI-JavaScript/wiki)
* [物件模型參考](https://microsoft.github.io/powerbi-models/modules/_models_.html)
* [實況示範](https://microsoft.github.io/PowerBI-JavaScript/demo/)