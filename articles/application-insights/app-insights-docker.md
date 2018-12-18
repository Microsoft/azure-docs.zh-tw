---
title: 在 Azure Application Insights 中監視 Docker 應用程式 | Microsoft Docs
description: Docker 效能計數器、事件和例外狀況可以與來自容器化應用程式的遙測一起顯示在 Application Insights 上。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 27a3083d-d67f-4a07-8f3c-4edb65a0a685
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/14/2017
ms.author: mbullwin
ms.openlocfilehash: 53ade76b9dbdc27df90da1f7e197464816529d1d
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295756"
---
# <a name="monitor-docker-applications-in-application-insights"></a>在 Application Insights 中監視 Docker 應用程式
[Docker](https://www.docker.com/) 容器的週期事件和效能計數器可以在 Application Insights 上繪製成圖表。 在您的主機的容器中安裝 [Application Insights](https://hub.docker.com/r/microsoft/applicationinsights/) 映像，它會顯示主機及其他映像的效能計數器。

使用 Docker，您可以在完成所有相依性的輕量級容器中散發應用程式。 它們會在執行 Docker 引擎的任何主機電腦上執行。

當您在 Docker 主機上執行 [Application Insights 映像 (英文)](https://hub.docker.com/r/microsoft/applicationinsights/) 時，可為您提供下列好處：

* 主機上執行之所有容器的相關週期遙測 - 啟動、停止等。
* 所有容器的效能計數器。 CPU、記憶體、網路使用量等。
* 如果您在於容器中執行的應用程式中[安裝了 Application Insights SDK for Java](app-insights-java-live.md)，則這些應用程式的所有遙測將會有可識別容器和主機電腦的額外屬性。 例如，如果您在多部主機上執行某個應用程式的多個執行個體，您可以輕鬆地依主機來篩選應用程式遙測。

![範例](./media/app-insights-docker/00.png)

## <a name="set-up-your-application-insights-resource"></a>設定您的 Application Insights 資源
1. 登入 [Microsoft Azure 入口網站](https://azure.com)，然後開啟您應用程式的 Application Insights 資源；或[建立新的資源](app-insights-create-new-resource.md)。 
   
    *我應該使用哪種資源？* 如果您在主機上執行的應用程式是由其他人所開發，則您需要[建立新的 Application Insights 資源](app-insights-create-new-resource.md)。 這是您檢視及分析遙測的位置 (針對應用程式類型選取 [一般])。
   
    但如果您是應用程式的開發人員，我們希望您 [將 Application Insights SDK 加入](app-insights-java-live.md) 每個應用程式中。 如果這些應用程式其實全部都是單一商務應用程式的元件，則您可能會設定所有應用程式將遙測資料傳送至一個資源，再使用該相同的資源來顯示 Docker 週期和效能資料。 
   
    第三種情況是您已開發大部分應用程式，但想要使用不同的資源來顯示其遙測。 在這種情況下，您可能也需要為 Docker 資料建立不同的資源。 
2. 新增 [Docker] 磚：選擇 [新增磚]，從資源庫拖曳 [Docker] 磚，然後按一下 [完成]。 
   
    ![範例](./media/app-insights-docker/03.png)

> [!NOTE]
> Application Insights 中的概觀窗格現已鎖定，且不允許從資源庫新增磚。 您仍然可以透過 Azure 儀表板介面來新增 Docker 磚，如上面所述。

3. 按一下 [程式集]  下拉式清單，然後複製檢測金鑰。 您將會使用這個項目以告知 SDK 傳送遙測的位置。

    ![範例](./media/app-insights-docker/02-props.png)

保持該瀏覽器視窗就緒，因為您稍後即會返回以查看您的遙測。

## <a name="run-the-application-insights-monitor-on-your-host"></a>在您的主機上執行 Application Insights 監視器
當您有其他位置可顯示遙測之後，您可以設定收集和傳送遙測的容器化應用程式。

1. 連接到您的 Docker 主機。 
2. 在這個命令中編輯您的檢測金鑰，然後執行命令：
   
   ```
   
   docker run -v /var/run/docker.sock:/docker.sock -d microsoft/applicationinsights ikey=000000-1111-2222-3333-444444444
   ```

每部 Docker 主機只需要一個 Application Insights 映像。 如果您的應用程式部署在多部 Docker 主機上，請在每部主機上重複執行命令。

## <a name="update-your-app"></a>更新應用程式
如果使用 [Application Insights SDK for Java](app-insights-java-get-started.md) 檢測您的應用程式，請將下行新增到您專案之 ApplicationInsights.xml 檔案中的 `<TelemetryInitializers>` 元素底下：

```xml

    <Add type="com.microsoft.applicationinsights.extensibility.initializer.docker.DockerContextInitializer"/> 
```

這會將容器和主機 ID 等 Docker 資訊，加入從您的應用程式送出的每個遙測項目中。

## <a name="view-your-telemetry"></a>檢視遙測
返回 Azure 入口網站中的 Application Insights 資源。

點選 [Docker] 磚。

您很快就會看到來自 Docker 應用程式的資料抵達，尤其是如果您在 Docker 引擎上有其他容器執行。

以下是一些您可以取得的檢視。

### <a name="perf-counters-by-host-activity-by-image"></a>依據主機的效能計數器，依據映像的活動
![範例](./media/app-insights-docker/10.png)

![範例](./media/app-insights-docker/11.png)

按一下任何主機或映像名稱以取得更多詳細資料。

若要自訂檢視，按一下任何圖表、方格標題，或使用 [新增圖表]。 

[深入了解計量瀏覽器](app-insights-metrics-explorer.md)。

### <a name="docker-container-events"></a>Docker 容器事件
![範例](./media/app-insights-docker/13.png)

若要調查個別事件，請按一下 [搜尋](app-insights-diagnostic-search.md)。 搜尋和篩選以尋找您想要的事件。 按一下任何事件以查看詳細資料。

### <a name="exceptions-by-container-name"></a>依據容器名稱的例外狀況
![範例](./media/app-insights-docker/14.png)

### <a name="docker-context-added-to-app-telemetry"></a>已加入應用程式遙測中的 Docker 內容
從使用 AI SDK 檢測之應用程式送出的要求遙測，並以 Docker 內容擴充：

![範例](./media/app-insights-docker/16.png)

處理器時間和可用記憶體效能計數器，並依 Docker 容器名稱擴充和分組：

![範例](./media/app-insights-docker/15.png)

## <a name="q--a"></a>問答集
*Application Insights 可以給我哪些無法從 Docker 取得的功能？*

* 依據容器和映像的效能計數器的詳細分解圖。
* 將容器和應用程式資料整合在一個儀表板中。
* [匯出遙測](app-insights-export-telemetry.md) 以進行資料庫、Power BI 或其他儀表板的進一步分析。

*如何從應用程式本身取得遙測？*

* 在應用程式中安裝 Application Insights SDK。 針對下列項目深入了解：[Java Web 應用程式](app-insights-java-get-started.md)、[Windows Web 應用程式](app-insights-asp-net.md)。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>後續步驟

* [適用於 Java 的 Application Insights](app-insights-java-get-started.md)
* [適用於 Node.js 的 Application Insights](app-insights-nodejs.md)
* [適用於 ASP.NET 的 Application Insights](app-insights-asp-net.md)
