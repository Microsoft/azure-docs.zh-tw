---
title: 監視 Linux - Azure 上的 Java Web 應用程式效能 | Microsoft Docs
description: 使用 Application Insights 的 CollectD 外掛程式擴充您的 Java 網站的應用程式效能監視功能。
services: application-insights
documentationcenter: java
author: mrbullwinkle
manager: carmonm
ms.assetid: 40c68f45-197a-4624-bf89-541eb7323002
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 08/24/2016
ms.author: mbullwin
ms.openlocfilehash: fd8c45bf3d7fd9e392d55a82001aa8768fcce8ba
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096594"
---
# <a name="collectd-linux-performance-metrics-in-application-insights"></a>collectd：Application Insights 中的 Linux 效能計量


若要在 [Application Insights](app-insights-overview.md) 中瀏覽 Linux 系統效能度量，請安裝 [collectd](http://collectd.org/) 以及其 Application Insights 外掛程式。 這個開放原始碼解決方案會收集各種系統和網路統計資料。

如果您已[使用 Application Insights 檢測您的 Java Web 服務][java]，通常您會使用 collectd。 提供給您更多資料來幫助您增強應用程式的效能或診斷問題。 

![範例圖表](./media/app-insights-java-collectd/sample.png)

## <a name="get-your-instrumentation-key"></a>取得檢測金鑰
在 [Microsoft Azure 入口網站](https://portal.azure.com)中，開啟您要顯示資料的 [Application Insights](app-insights-overview.md) 資源。 (或[建立新的資源](app-insights-create-new-resource.md)。)

取得一份可識別資源的檢測金鑰。

![瀏覽全部，開啟您的資源，然後在 [Essentials] 下拉式清單中，選取並複製檢測金鑰](./media/app-insights-java-collectd/02-props.png)

## <a name="install-collectd-and-the-plug-in"></a>安裝 collectd 和外掛程式
在您的 Linux 伺服器機器上：

1. 安裝 [collectd](http://collectd.org/) 5.4.0 版或更新版本。
2. 下載 [Application Insights collectd 寫入器外掛程式](https://aka.ms/aijavasdk)。 記下版本號碼。
3. 將外掛程式 JAR 複製到 `/usr/share/collectd/java`。
4. 編輯 `/etc/collectd/collectd.conf`：
   * 確定 [Java 外掛程式](https://collectd.org/wiki/index.php/Plugin:Java) 已啟用。
   * 更新 java.class.path 的 JVMArg 以包括下列 JAR。 更新版本號碼以符合您所下載的版本：
   * `/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar`
   * 使用來自您的資源的檢測金鑰，加入此程式碼片段：

```XML

     LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"
     <Plugin ApplicationInsightsWriter>
        InstrumentationKey "Your key"
     </Plugin>
```

以下是範例組態檔的一部分：

```XML

    ...
    # collectd plugins
    LoadPlugin cpu
    LoadPlugin disk
    LoadPlugin load
    ...

    # Enable Java Plugin
    LoadPlugin "java"

    # Configure Java Plugin
    <Plugin "java">
      JVMArg "-verbose:jni"
      JVMArg "-Djava.class.path=/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar:/usr/share/collectd/java/collectd-api.jar"

      # Enabling Application Insights plugin
      LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"

      # Configuring Application Insights plugin
      <Plugin ApplicationInsightsWriter>
        InstrumentationKey "12345678-1234-1234-1234-123456781234"
      </Plugin>

      # Other plugin configurations ...
      ...
    </Plugin>
    ...
```

設定其他 [collectd 外掛程式](https://collectd.org/wiki/index.php/Table_of_Plugins)，它可以從不同來源收集各種資料。

根據其 [手冊](https://collectd.org/wiki/index.php/First_steps)重新啟動 collectd。

## <a name="view-the-data-in-application-insights"></a>在 Application Insights 中檢視資料
在您的 Application Insights 資源中，開啟[計量瀏覽器並加入圖表][metrics]，從 [自訂] 類別選取您想要查看的度量。

![](./media/app-insights-java-collectd/result.png)

根據預設，會對收集度量來源的所有主機電腦彙總度量。 若要檢視每一主機的度量，在圖表的 [詳細資料] 刀鋒視窗中，開啟 [群組]，然後選擇依 CollectD-Host 群組。

## <a name="to-exclude-upload-of-specific-statistics"></a>排除特定統計資料的上傳
根據預設，Application Insights 外掛程式會傳送所有啟用的 collectd 'read' 外掛程式所收集的所有資料。 

若要排除特定外掛程式或資料來源的資料：

* 編輯組態檔。 
* 在 `<Plugin ApplicationInsightsWriter>`中，加入指示詞行，如下所示：

| 指示詞 | 效果 |
| --- | --- |
| `Exclude disk` |排除 `disk` 外掛程式所收集的所有資料 |
| `Exclude disk:read,write` |排除來自 `disk` 外掛程式名為 `read` 和 `write` 的來源。 |

以新行分隔個別指示詞。

## <a name="problems"></a>有問題嗎？
*我在入口網站中看不到任何資料*

* 開啟[搜尋][diagnostic]以查看原始事件是否已抵達。 有時需要較長的時間才會在計量瀏覽器中顯示。
* 您可能需要 [設定輸出資料的防火牆例外狀況](app-insights-ip-addresses.md)
* 在 Application Insights 外掛程式中啟用追蹤。 在 `<Plugin ApplicationInsightsWriter>`內加入這一行：
  * `SDKLogger true`
* 開啟終端機，並以詳細資訊模式啟動 collectd，以查看所報告的任何問題：
  * `sudo collectd -f`

## <a name="known-issue"></a>已知問題

Application Insights 的「寫入」外掛程式與某些「讀取」外掛程式不相容。 有些外掛程式有時會在 Application Insights 外掛程式預期要有浮點數的位置傳送 "NaN"。

徵狀：collectd 記錄檔會顯示包含下列資訊的錯誤：「AI: ...SyntaxError: 未預期的權杖 N」。

因應措施：排除有問題的「寫入」外掛程式所收集的資料。 

<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[apiexceptions]: app-insights-api-custom-events-metrics.md#track-exception
[availability]: app-insights-monitor-web-app-availability.md
[diagnostic]: app-insights-diagnostic-search.md
[eclipse]: app-insights-java-eclipse.md
[java]: app-insights-java-get-started.md
[javalogs]: app-insights-java-trace-logs.md
[metrics]: app-insights-metrics-explorer.md


