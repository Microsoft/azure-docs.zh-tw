---
title: 在 JAVA web 應用程式中篩選 Azure 應用程式見解遙測
description: 篩選出您不需要監視的事件，以減少遙測流量。
ms.topic: conceptual
ms.date: 3/14/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 71858be97404344bad88ea20e31b17fa44f669a2
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98131180"
---
# <a name="filter-telemetry-in-your-java-web-app"></a>在 Java Web 應用程式中篩選遙測

> [!IMPORTANT]
> 監視 JAVA 應用程式的建議方法是使用自動檢測，而不需要變更程式碼。 請遵循 [Application Insights JAVA 3.0 代理程式](./java-in-process-agent.md)的指導方針。

篩選器可供選取 [Java Web 應用程式傳送至 Application Insights](java-get-started.md) 的遙測。 您可以使用一些現成可用的篩選器，也可以撰寫自己的自訂篩選器。

現成可用的篩選器包括︰

* 追蹤嚴重性層級
* 特定 URL、關鍵字或回應碼
* 快速回應 - 也就是，要求您的應用程式快速回應
* 特定事件名稱

> [!NOTE]
> 篩選器會扭曲您應用程式的計量。 例如，您可能會決定，為了診斷緩慢回應，您將設定一個篩選器來捨棄快速回應時間。 但是，您必須留意，Application Insights 所回報的平均回應時間會比真實的速度慢，而且要求計數會小於實際計數。
> 如果有此疑慮，請改用[取樣](./sampling.md)。

## <a name="setting-filters"></a>設定篩選器

在 ApplicationInsights.xml，新增如下面範例所示的 `TelemetryProcessors` 區段：


```XML

    <ApplicationInsights>
      <TelemetryProcessors>

        <BuiltInProcessors>
           <Processor type="TraceTelemetryFilter">
                  <Add name="FromSeverityLevel" value="ERROR"/>
           </Processor>

           <Processor type="RequestTelemetryFilter">
                  <Add name="MinimumDurationInMS" value="100"/>
                  <Add name="NotNeededResponseCodes" value="200-400"/>
           </Processor>

           <Processor type="PageViewTelemetryFilter">
                  <Add name="DurationThresholdInMS" value="100"/>
                  <Add name="NotNeededNames" value="home,index"/>
                  <Add name="NotNeededUrls" value=".jpg,.css"/>
           </Processor>

           <Processor type="TelemetryEventFilter">
                  <!-- Names of events we don't want to see -->
                  <Add name="NotNeededNames" value="Start,Stop,Pause"/>
           </Processor>

           <!-- Exclude telemetry from availability tests and bots -->
           <Processor type="SyntheticSourceFilter">
                <!-- Optional: specify which synthetic sources,
                     comma-separated
                     - default is all synthetics -->
                <Add name="NotNeededSources" value="Application Insights Availability Monitoring,BingPreview"
           </Processor>

        </BuiltInProcessors>

        <CustomProcessors>
          <Processor type="com.fabrikam.MyFilter">
            <Add name="Successful" value="false"/>
          </Processor>
        </CustomProcessors>

      </TelemetryProcessors>
    </ApplicationInsights>

```

[檢查整組的內建處理器](https://github.com/microsoft/ApplicationInsights-Java/tree/master/core/src/main/java/com/microsoft/applicationinsights/internal)。

## <a name="built-in-filters"></a>內建篩選器

### <a name="metric-telemetry-filter"></a>計量遙測篩選器

```XML

           <Processor type="MetricTelemetryFilter">
                  <Add name="NotNeeded" value="metric1,metric2"/>
           </Processor>
```

* `NotNeeded` - 自訂計量名稱的逗號分隔清單。


### <a name="page-view-telemetry-filter"></a>頁面檢視遙測篩選器

```XML

           <Processor type="PageViewTelemetryFilter">
                  <Add name="DurationThresholdInMS" value="500"/>
                  <Add name="NotNeededNames" value="page1,page2"/>
                  <Add name="NotNeededUrls" value="url1,url2"/>
           </Processor>
```

* `DurationThresholdInMS` - Duration 是指載入網頁所花費的時間。 若已設定此屬性，則不會報告載入速度比此時間快的頁面。
* `NotNeededNames` - 頁面名稱的逗號分隔清單。
* `NotNeededUrls` - URL 片段的逗號分隔清單。 例如，`"home"` 可篩選出在 URL 中包含 "home" 的所有頁面。


### <a name="request-telemetry-filter"></a>要求遙測篩選器


```XML

           <Processor type="RequestTelemetryFilter">
                  <Add name="MinimumDurationInMS" value="500"/>
                  <Add name="NotNeededResponseCodes" value="page1,page2"/>
                  <Add name="NotNeededUrls" value="url1,url2"/>
           </Processor>
```



### <a name="synthetic-source-filter"></a>綜合來源篩選器

篩選出 SyntheticSource 屬性中具有值的所有遙測。 這些包括來自 Bot、編目程式和可用性測試的要求。

篩選出所有綜合要求的遙測︰


```XML

           <Processor type="SyntheticSourceFilter" />
```

篩選出特定綜合來源的遙測︰


```XML

           <Processor type="SyntheticSourceFilter" >
                  <Add name="NotNeeded" value="source1,source2"/>
           </Processor>
```

* `NotNeeded` - 綜合來源名稱的逗號分隔清單。

### <a name="telemetry-event-filter"></a>遙測事件篩選器

篩選自訂事件 (使用 [TrackEvent()](./api-custom-events-metrics.md#trackevent) 記錄)。


```XML

           <Processor type="TelemetryEventFilter" >
                  <Add name="NotNeededNames" value="event1, event2"/>
           </Processor>
```


* `NotNeededNames` - 事件名稱的逗號分隔清單。


### <a name="trace-telemetry-filter"></a>追蹤遙測篩選器

篩選記錄檔追蹤 (使用 [TrackTrace()](./api-custom-events-metrics.md#tracktrace) 或[紀錄架構收集器](java-trace-logs.md)記錄)。

```XML

           <Processor type="TraceTelemetryFilter">
                  <Add name="FromSeverityLevel" value="ERROR"/>
           </Processor>
```

* `FromSeverityLevel` 有效值為︰
  *  OFF             - 篩選出所有追蹤
  *  TRACE           - 不篩選。 等於追蹤層級
  *  INFO            - 篩選出 TRACE 層級
  *  WARN            - 篩選出 TRACE 和 INFO
  *  ERROR           - 篩選出 WARN、INFO、TRACE
  *  CRITICAL        - 篩選出 CRITICAL 以外的所有項目


## <a name="custom-filters"></a>自訂篩選器

### <a name="1-code-your-filter"></a>1. 撰寫篩選器的程式碼

在您的程式碼中，建立類別來實作 `TelemetryProcessor`:

```Java

    package com.fabrikam.MyFilter;
    import com.microsoft.applicationinsights.extensibility.TelemetryProcessor;
    import com.microsoft.applicationinsights.telemetry.Telemetry;

    public class SuccessFilter implements TelemetryProcessor {

        /* Any parameters that are required to support the filter.*/
        private final String successful;

        /* Initializers for the parameters, named "setParameterName" */
        public void setNotNeeded(String successful)
        {
            this.successful = successful;
        }

        /* This method is called for each item of telemetry to be sent.
           Return false to discard it.
           Return true to allow other processors to inspect it. */
        @Override
        public boolean process(Telemetry telemetry) {
            if (telemetry == null) { return true; }
            if (telemetry instanceof RequestTelemetry)
            {
                RequestTelemetry requestTelemetry = (RequestTelemetry)    telemetry;
                return request.getSuccess() == successful;
            }
            return true;
        }
    }

```

### <a name="2-invoke-your-filter-in-the-configuration-file"></a>2. 在設定檔中叫用您的篩選

在 ApplicationInsights.xml 中：

```XML


    <ApplicationInsights>
      <TelemetryProcessors>
        <CustomProcessors>
          <Processor type="com.fabrikam.SuccessFilter">
            <Add name="Successful" value="false"/>
          </Processor>
        </CustomProcessors>
      </TelemetryProcessors>
    </ApplicationInsights>

```

### <a name="3-invoke-your-filter-java-spring"></a>3. (JAVA 春季叫用您的篩選) 

針對以春季架構為基礎的應用程式，自訂遙測處理器必須在您的主要應用程式類別中註冊為 bean。 當應用程式啟動時，它們就會 autowired。

```Java
@Bean
public TelemetryProcessor successFilter() {
      return new SuccessFilter();
}
```

您必須在中建立自己的篩選參數 `application.properties` ，並利用彈簧開機的外部化設定架構，將這些參數傳遞至您的自訂篩選準則。 


## <a name="troubleshooting"></a>疑難排解

我的篩選器無法運作。

* 請檢查您已提供有效的參數值。 例如，持續時間應該是整數。 無效的值會導致篩選器被忽略。 如果您的自訂篩選器從建構函式或 set 方法擲回例外狀況，該篩選器將被忽略。

## <a name="next-steps"></a>後續步驟

* [取樣](./sampling.md) - 請考慮以取樣做為替代方式，因為它不會扭曲計量。

