---
title: 使用 Azure Application Insights 進行使用量分析 | Microsoft Docs
description: 了解您的使用者，以及他們如何運用您的應用程式。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 10/10/2017
ms.author: mbullwin
ms.openlocfilehash: 1217083980355c69fa51d7a6983c71295684f5b1
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633534"
---
# <a name="usage-analysis-with-application-insights"></a>使用 Application Insights 進行使用量分析

Web 或行動應用程式的哪些功能最受歡迎？ 您的使用者是否利用您的應用程式達到其目標呢？ 他們會在特定點退出，並在稍後返回嗎？  [Azure Application Insights](app-insights-overview.md) 可協助您深入了解使用者如何使用應用程式。 每次您更新應用程式時，都可以評估它適用於使用者的程度。 您可以透過了解這些來制定下一個開發週期的相關資料導向決策。

## <a name="send-telemetry-from-your-app"></a>傳送來自您應用程式的遙測

若要獲得最佳體驗，請同時在您的應用程式伺服器程式碼和網頁中安裝 Application Insights。 您應用程式的用戶端和伺服器元件會將遙測資料傳送回 Azure 入口網站以供分析。

1. **伺服器程式碼：** 為您的 [ASP.NET](app-insights-asp-net.md)、[Azure](app-insights-azure.md)、[Java](app-insights-java-get-started.md)、[Node.js](app-insights-nodejs.md) 或[其他](app-insights-platforms.md)應用程式安裝適當的模組。

    * *不想安裝伺服器程式碼嗎？請直接[建立 Azure Application Insights 資源](app-insights-create-new-resource.md)。*

2. **網頁程式碼：** 開啟 [Azure 入口網站](https://portal.azure.com)、開啟您應用程式的 Application Insights 資源，然後開啟 [快速入門] > [監視及診斷用戶端應用程式]。 

    ![將指令碼複製到您主版頁面的標頭。](./media/app-insights-usage-overview/02-monitor-web-page.png)

3. **行動應用程式程式碼：** 使用 App Center SDK 從應用程式收集事件，然後將這些事件的複本傳送至 Application Insights，[按照此指南](app-insights-mobile-center-quickstart.md)的指示進行分析。

4. **取得遙測資料：** 以偵錯模式執行您的專案幾分鐘，然後在 Application Insights 的 [概觀] 刀鋒視窗中尋找結果。

    發佈您的應用程式以監視應用程式的效能，並了解使用者如何利用您的應用程式。

## <a name="include-user-and-session-id-in-your-telemetry"></a>將使用者與工作階段識別碼加入您的遙測
若要追蹤使用者一段時間，Application Insights 需要識別他們的方式。 「事件工具」是唯一不需要使用者識別碼或工作階段識別碼的「使用量工具」。

使用[此流程](https://docs.microsoft.com/azure/application-insights/app-insights-usage-send-user-context)開始傳送使用者與工作階段識別碼。

## <a name="explore-usage-demographics-and-statistics"></a>探索使用量人口統計和統計資料
了解人們在使用您的應用程式時，他們最感興趣的網頁、使用者的所在位置，以及他們使用何種瀏覽器與作業系統。 

使用者與工作階段報告會依頁面或自訂事件來篩選資料，並透過諸如位置、環境及頁面等屬性，將這些頁面或自訂事件進行區隔。 您也可以新增自己的篩選條件。

![使用者](./media/app-insights-usage-overview/users.png)  

右方情資指出資料集內的有趣模式。  

* **使用者**報告會在您所選擇的時間週期內，計算存取您網頁的唯一使用者數目。 對於 Web 應用程式，會使用 Cookie 來計算使用者。 如果有人使用不同的瀏覽器或用戶端電腦來存取您的網站，或清除其 Cookie，系統就會將他們計算為多次。
* **工作階段**報告會計算存取您網站之使用者工作階段的數目。 工作階段是使用者活動的一段時間，在閒置時間超過半小時後就會加以終止。

[進一步了解「使用者」、「工作階段」和「事件」工具](app-insights-usage-segmentation.md)  

## <a name="page-views"></a>頁面檢視

在 [使用量] 刀鋒視窗中，按一下 [頁面檢視] 圖格可取得最常用頁面的明細：

![從 [概觀] 刀鋒視窗，按一下 [頁面檢視] 圖表](./media/app-insights-usage-overview/05-games.png)

上述範例來自遊戲網站。 從這些圖表，我們可以立即看到：

* 在上一週使用量未改善。 也許我們應該考慮搜尋引擎最佳化？
* 網球是最受歡迎的遊戲頁面。 讓我們將焦點放在此頁面的進一步改善上。
* 平均而言，使用者大約每週瀏覽網球頁面三次。 (與使用者相比，工作階段數大約多出三倍)。
* 大多數使用者都是在美國工作週且在工作時間瀏覽此網站。 也許我們應該在網頁上提供一個 [快速隱藏] 按鈕。
* 圖表上的[註解](app-insights-annotations.md)會顯示新版網站的部署時間。 沒有任何一個最近的部署對使用量有明顯的影響。

如果您要更詳細調查網站的流量，像是依您的網站在其頁面檢視遙測中所傳送之自訂屬性進行分割，該怎麼做？

1. 在 [Application Insights 資源] 功能表中，開啟「事件」工具。 此工具可讓您根據各種的篩選、同群使用者及區隔等選項，將應用程式所傳送出的頁面檢視和自訂事件數目進行分析。
2. 在 [已使用人員] 下拉式清單中，選取 [任何頁面檢視]。
3. 在 [分割者] 下拉式清單中，選取要用來分割您頁面檢視遙測的屬性。

## <a name="retention---how-many-users-come-back"></a>保留期 - 回來使用的使用者人數？

保留期可根據同群使用者在特定時間貯體期間執行的某些商務動作，協助您了解使用者返回使用其應用程式的頻率。 

- 了解相較於其他功能，哪些特定功能會讓使用者回來使用 
- 根據實際使用者資料的表單假設 
- 判斷保留期是否為您產品的問題 

![保留](./media/app-insights-usage-overview/retention.png) 

在最上層的保留期控制項可讓您定義用來計算保留期的特定事件和時間範圍。 中間的圖表會依指定的時間範圍提供整體保留期百分比的視覺表示法。 底部圖表代表指定時間內的個別保留期。 此詳細資料等級會使用更詳細的資料粒度，讓您了解使用者在做什麼，以及可能會影響舊有使用者的因素。  

[深入了解保留期工具](app-insights-usage-retention.md)

## <a name="custom-business-events"></a>自訂商務事件

若要清楚了解使用者會使用您的應用程式來做什麼，插入程式碼行來記錄自訂事件會很有用。 這些事件可以追蹤任何項目，從詳細的使用者動作 (例如按一下特定按鈕)，到更重要的商務事件 (例如購物或遊戲獲勝)。 

雖然在某些情況下，頁面檢視可以代表有用的事件，但一般而言它並不正確。 使用者可以開啟產品網頁而無需購買產品。 

您可以使用特定商務事件，透過網站將使用者的進度製作成圖表。 您可以找出他們對不同選項的偏好，以及它們退出或遇到困難之處。 您可以透過了解這些對開發待處理項目的優先順序做出明智決策。

事件可透過應用程式的用戶端來記錄下來：

```JavaScript

    appInsights.trackEvent("ExpandDetailTab", {DetailTab: tabName});
```

或者，也可以透過伺服器端：

```csharp
    var tc = new Microsoft.ApplicationInsights.TelemetryClient();
    tc.TrackEvent("CreatedAccount", new Dictionary<string,string> {"AccountType":account.Type}, null);
    ...
    tc.TrackEvent("AddedItemToCart", new Dictionary<string,string> {"Item":item.Name}, null);
    ...
    tc.TrackEvent("CompletedPurchase");
```

您可以將屬性值附加至這些事件，當您在入口網站中檢查它們時，就可以將事件進行篩選或分割。 此外，會將一組標準的屬性附加至每個事件，例如匿名使用者識別碼，這可讓您追蹤個別使用者的活動順序。

深入了解[自訂事件](app-insights-api-custom-events-metrics.md#trackevent)和[屬性](app-insights-api-custom-events-metrics.md#properties)。

### <a name="slice-and-dice-events"></a>將事件進行交叉分析

在「使用者」、「工作階段」和「事件」工具中，您可以依使用者、事件名稱和屬性將自訂事件進行交叉分析。
![使用者](./media/app-insights-usage-overview/users.png)  
  
## <a name="design-the-telemetry-with-the-app"></a>使用應用程式來設計遙測

當您在設計您應用程式的每項功能時，請考慮要如何利用使用者來測量它的成功。 決定您需要記錄哪些商務事件，並從開頭將這些事件的追蹤呼叫編碼至應用程式。

## <a name="a--b-testing"></a>A | B 測試
如果您不知道哪個功能的變數將更可能成功，請兩者都釋出，讓兩者都可供不同使用者存取。 測量每一個的成功，然後移至整合的版本。

針對此技術，您會將獨特的屬性值附加到每個應用程式版本所傳送的所有遙測。 您可以在作用中 TelemetryContext 中定義屬性來執行該動作。 這些預設屬性會加入到應用程式傳送的每個遙測訊息 - 不只是您的自訂訊息，還有標準遙測。

在 Application Insights 入口網站中，將資料依屬性值篩選並分割，以便比較不同版本。

若要這樣做，[請設定遙測初始設定式](app-insights-api-filtering-sampling.md##add-properties-itelemetryinitializer)：

```csharp


    // Telemetry initializer class
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize (ITelemetry telemetry)
        {
            telemetry.Properties["AppVersion"] = "v2.1";
        }
    }
```

在 Web 應用程式初始設定式例如 Global.asax.cs 中：

```csharp

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers
        .Add(new MyTelemetryInitializer());
    }
```

所有新的 TelemetryClients 都會自動新增您指定的屬性值。 個別的遙測事件可以覆寫預設值。

## <a name="next-steps"></a>後續步驟
   - [使用者、工作階段、事件](app-insights-usage-segmentation.md)
   - [漏斗圖](usage-funnels.md)
   - [保留](app-insights-usage-retention.md)
   - [使用者流程](app-insights-usage-flows.md)
   - [活頁簿](app-insights-usage-workbooks.md)
   - [新增使用者內容](app-insights-usage-send-user-context.md)
