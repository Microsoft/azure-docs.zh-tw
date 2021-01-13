---
title: 快速入門：使用 Azure 應用程式 Insights 的 JAVA web 應用程式分析
description: '使用 Application Insights 針對 Java Web 應用程式進行應用程式效能監視。 '
ms.topic: conceptual
ms.date: 11/22/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 115e1ec347cdcd80904b47a0c8798206360d0dad
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98131775"
---
# <a name="quickstart-get-started-with-application-insights-in-a-java-web-project"></a>快速入門：開始在 Java Web 專案中使用 Application Insights (機器翻譯)


> [!CAUTION]
> 自2020年11月起，為了監視 JAVA 應用程式，我們建議使用 Azure 監視器 Application Insights JAVA 3.0 代理程式來進行自動檢測。 如需如何開始使用的詳細資訊，請參閱 [Application Insights JAVA 3.0 代理程式](./java-in-process-agent.md)。

在本快速入門中，您會使用 Application Insights SDK 來檢測要求、追蹤相依性，以及收集效能計數器、診斷效能問題和例外狀況，以及撰寫程式碼來追蹤使用者對應用程式執行的動作。

Application Insights 是一項 Web 開發人員可延伸的分析服務，可幫助您了解即時應用程式的效能和使用情形。 Application Insights 支援 Linux、Unix 或 Windows 上執行的 Java 應用程式。

## <a name="prerequisites"></a>Prerequisites

* 具有有效訂用帳戶的 Azure 帳戶。 [免費建立帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
* 運作正常的 JAVA 應用程式。

## <a name="get-an-application-insights-instrumentation-key"></a>取得 Application Insights 檢測金鑰

> [!IMPORTANT]
> 新的 Azure 區域 **需要** 使用連接字串，而不是檢測金鑰。 [連接字串](./sdk-connection-string.md?tabs=java) 會識別您想要與遙測資料相關聯的資源。 它也可讓您修改您的資源將用來做為遙測目的地的端點。 您必須複製連接字串，並將它加入應用程式的程式碼或加入環境變數。
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 Azure 入口網站中，建立 Application Insights 資源。 將應用程式類型設定為 Java Web 應用程式。

3. 尋找新資源的檢測金鑰。 您很快需要將此金鑰貼到程式碼專案中。

    ![在新資源概觀中，按一下 [屬性] 並複製檢測金鑰](./media/java-get-started/instrumentation-key-001.png)

## <a name="add-the-application-insights-sdk-for-java-to-your-project"></a>將 Java 適用的 Application Insights SDK 加入至專案

*選擇您的專案類型。*

# <a name="maven"></a>[Maven](#tab/maven)

如果您的專案已設定為使用 Maven 進行組建，請將下列程式碼合併至您的 *pom.xml* 檔案。

然後重新整理專案相依性，以下載程式庫。

```XML
    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-web-auto</artifactId>
        <!-- or applicationinsights-web for manual web filter registration -->
        <!-- or applicationinsights-core for bare API -->
        <version>2.6.2</version>
      </dependency>
    </dependencies>
```

# <a name="gradle"></a>[Gradle](#tab/gradle)

如果您的專案已設定為使用組建的 Gradle，請將下列程式碼合併至 *Gradle* 檔案。

然後重新整理專案相依性，以下載程式庫。

```gradle
    dependencies {
      compile group: 'com.microsoft.azure', name: 'applicationinsights-web-auto', version: '2.6.2'
      // or applicationinsights-web for manual web filter registration
      // or applicationinsights-core for bare API
    }
```

---

### <a name="questions"></a>問題
* *和元件之間的關係為何 `-web-auto` `-web` `-core` ？*
  * `applicationinsights-web-auto` 藉由在執行時間自動註冊 Application Insights servlet 篩選器，提供計量來追蹤 HTTP servlet 要求計數和回應時間。
  * `applicationinsights-web` 也提供追蹤 HTTP servlet 要求計數和回應時間的計量，但需要在您的應用程式中手動註冊 Application Insights servlet 篩選器。
  * `applicationinsights-core` 僅提供您的裸機 API，例如，如果您的應用程式不是以 servlet 為基礎。
  
* 如果將 SDK 升級為最新版本？
  * 自2020年11月起，為了監視 JAVA 應用程式，我們建議使用 Azure 監視器 Application Insights JAVA 3.0 代理程式來進行自動檢測。 如需如何開始使用的詳細資訊，請參閱 [Application Insights JAVA 3.0 代理程式](./java-in-process-agent.md)。

## <a name="add-an-applicationinsightsxml-file"></a>新增 *ApplicationInsights.xml* 檔案
將 *ApplicationInsights.xml* 新增至專案中的 [資源] 資料夾，或確定它已新增至專案的部署類別路徑。 將下列 XML 複製到其中。

使用您從 Azure 入口網站取得的檢測金鑰來取代它。

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">

   <!-- The key from the portal: -->
   <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>

   <!-- HTTP request component (not required for bare API) -->
   <TelemetryModules>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
   </TelemetryModules>

   <!-- Events correlation (not required for bare API) -->
   <!-- These initializers add context data to each event -->
   <TelemetryInitializers>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
   </TelemetryInitializers>

</ApplicationInsights>
```

（選擇性）設定檔可位於您的應用程式可存取的任何位置。  System 屬性 `-Dapplicationinsights.configurationDirectory` 指定包含 *ApplicationInsights.xml* 的目錄。 例如，位於 `E:\myconfigs\appinsights\ApplicationInsights.xml` 的組態檔是使用屬性 `-Dapplicationinsights.configurationDirectory="E:\myconfigs\appinsights"` 來設定。

* 檢測金鑰會隨著遙測的每個項目傳送，並告知 Application Insights 在您的資源中顯示它。
* HTTP 要求元件是選用的。 它會自動將要求和回應時間的遙測傳送到入口網站。
* 事件相互關聯是 HTTP 要求元件的補充。 它會將識別碼指派給伺服器收到的每個要求。 然後，它會將此識別碼做為屬性加入至每個遙測專案的屬性 ' Operation.Id '。 它可讓您相互關聯與每個要求關聯的遙測，方法是在 [診斷搜尋][diagnostic]中設定篩選器。

### <a name="alternative-ways-to-set-the-instrumentation-key"></a>設定檢測金鑰的替代方法
Application Insights SDK 會依此順序尋找此金鑰︰

1. 系統屬性：-DAPPINSIGHTS_INSTRUMENTATIONKEY = your_ikey
2. 環境變數： APPINSIGHTS_INSTRUMENTATIONKEY
3. 設定檔： *ApplicationInsights.xml*

您也可以 [在程式碼中設定](./api-custom-events-metrics.md#ikey)：

```java
    String instrumentationKey = "00000000-0000-0000-0000-000000000000";

    if (instrumentationKey != null)
    {
        TelemetryConfiguration.getActive().setInstrumentationKey(instrumentationKey);
    }
```

## <a name="add-agent"></a>新增代理程式

[安裝 JAVA 代理程式](java-agent.md) 來捕捉傳出 HTTP 呼叫、JDBC 查詢、應用程式記錄，以及更好的作業命名。

## <a name="run-your-application"></a>執行您的應用程式
在您的開發電腦上以偵錯模式執行應用程式，或發佈至您的伺服器。

## <a name="view-your-telemetry-in-application-insights"></a>在 Application Insights 中檢視遙測
返回 [Microsoft Azure 入口網站](https://portal.azure.com)中的 Application Insights 資源。

[概觀] 刀鋒視窗上會顯示 HTTP 要求資料。 (如果沒有出現，請稍等片刻，然後按一下 [重新整理]。)

![概觀範例資料的螢幕擷取畫面](./media/java-get-started/overview-graphs.png)

[深入瞭解計量。][metrics]

按一下任何圖表以查看詳細彙總度量。

![具有圖表的 Application Insights 失敗窗格](./media/java-get-started/006-barcharts.png)

### <a name="instance-data"></a>執行個體資料
點選特定要求類型以查看個別執行個體。

![深入瞭解特定的範例視圖](./media/java-get-started/007-instance.png)

### <a name="analytics-powerful-query-language"></a>分析︰功能強大的查詢語言
當您累積更多資料時，您就可以執行查詢以彙總資料並找出個別執行個體。  [分析](../log-query/log-query-overview.md) 是一項強大的工具，既可了解效能和使用情況，也可進行診斷。

![分析的範例](./media/java-get-started/0025.png)

## <a name="install-your-app-on-the-server"></a>在伺服器上安裝您的 App
現在將您的應用程式發佈至伺服器供人使用，然後查看入口網站顯示的遙測。

* 請確定您的防火牆允許應用程式將遙測傳送至這些連接埠：

  * dc.services.visualstudio.com:443
  * f5.services.visualstudio.com:443

* 如果連出流量必須透過防火牆路由傳送，定義系統屬性 `http.proxyHost` 和 `http.proxyPort`。

* 在 Windows 伺服器上，安裝：

  * [Microsoft Visual C++ 可轉散發套件](https://www.microsoft.com/download/details.aspx?id=40784)

    (此元件會啟用效能計數器。)

## <a name="azure-app-service-aks-vms-config"></a>Azure App Service，AKS，Vm 設定

若要監視 Azure 資源提供者上執行的應用程式，最簡單且最簡單的方法是透過 [JAVA 3.0 代理](./java-in-process-agent.md)程式使用 Application Insights 自動檢測。


## <a name="exceptions-and-request-failures"></a>例外狀況與要求失敗
Application Insights web 篩選器會自動收集未處理的例外狀況和要求失敗。

若要收集其他例外狀況的資料，您可以 [在程式碼中插入 trackException ( # A1 的呼叫][apiexceptions]。

## <a name="monitor-method-calls-and-external-dependencies"></a>監視方法呼叫和外部相依性
[安裝 Java 代理程式](java-agent.md) 以記錄指定的內部方法和透過 JDBC 發出的呼叫與計時資料。

以及用於自動操作的命名。

## <a name="w3c-distributed-tracing"></a>W3C 分散式追蹤

Application Insights Java SDK 現在支援 [W3C 分散式追蹤](https://w3c.github.io/trace-context/)。

我們的[相互關聯](correlation.md)文章會進一步說明內送 SDK 組態。

外寄 SDK 組態則定義於 [AI-Agent.xml](java-agent.md) 檔案中。

## <a name="performance-counters"></a>效能計數器
開啟 **調查**、 **計量**，以查看一系列的效能計數器。

![已選取進程私用位元組的 [計量] 窗格螢幕擷取畫面](./media/java-get-started/011-perf-counters.png)

### <a name="customize-performance-counter-collection"></a>自訂效能計數器集合
若要停用一組標準效能計數器的收集，請在 *ApplicationInsights.xml* 檔案的根節點下新增下列程式碼：

```XML
    <PerformanceCounters>
       <UseBuiltIn>False</UseBuiltIn>
    </PerformanceCounters>
```

### <a name="collect-additional-performance-counters"></a>收集其他效能計數器
您可以指定要收集的其他效能計數器。

#### <a name="jmx-counters-exposed-by-the-java-virtual-machine"></a>JMX 計數器 (由虛擬機器公開)

```XML
    <PerformanceCounters>
      <Jmx>
        <Add objectName="java.lang:type=ClassLoading" attribute="TotalLoadedClassCount" displayName="Loaded Class Count"/>
        <Add objectName="java.lang:type=Memory" attribute="HeapMemoryUsage.used" displayName="Heap Memory Usage-used" type="composite"/>
      </Jmx>
    </PerformanceCounters>
```

* `displayName` – Application Insights 入口網站中顯示的名稱。
* `objectName` – JMX 物件名稱。
* `attribute` – 要提取的 JMX 物件名稱的屬性
* `type` (選擇性) -JMX 物件的屬性類型：
  * 預設值：簡易類型，例如 int 或 long。
  * `composite`：效能計數器資料的格式為 'Attribute.Data'
  * `tabular`：效能計數器資料的格式為資料表列

#### <a name="windows-performance-counters"></a>Windows 效能計數器
每個 [Windows 效能計數器](/windows/win32/perfctrs/performance-counters-portal) 是類別的成員 (以欄位是類別成員的相同方式)。 類別可以是全域，或可以有一定數量或指定的執行個體。

```XML
    <PerformanceCounters>
      <Windows>
        <Add displayName="Process User Time" categoryName="Process" counterName="%User Time" instanceName="__SELF__" />
        <Add displayName="Bytes Printed per Second" categoryName="Print Queue" counterName="Bytes Printed/sec" instanceName="Fax" />
      </Windows>
    </PerformanceCounters>
```

* displayName – Application Insights 入口網站中顯示的名稱。
* categoryName – 與此效能計數器有關聯的效能計數器類別 (效能物件)。
* counterName – 效能計數器的名稱。
* instanceName – 效能計數器類別執行個體的名稱，或如果類別包含單一執行個體，則為空白字串 ("")。 如果 categoryName 為 Process，而您要收集的效能計數器來自應用程式執行所在的目前 JVM 處理程序，請指定 `"__SELF__"`。

### <a name="unix-performance-counters"></a>Unix 效能計數器
* [使用 Application Insights 外掛程式安裝 collectd](java-collectd.md) ，來取得各種不同的系統和網路資料。

## <a name="get-user-and-session-data"></a>取得使用者與工作階段資料
好了，您現在正在從 Web 服務傳送遙測資料。 現在若要取得應用程式的完整 360 度檢視，可以加入多個監視：

* [將遙測加入至您的網頁][usage] ，即可監視頁面檢視和使用者度量。
* [設定 Web 測試][availability] ，以確認應用程式處於線上狀態且能夠回應。

## <a name="send-your-own-telemetry"></a>傳送您自己的遙測
既然您已安裝 SDK，您可以使用 API 來傳送自己的遙測。

* [追蹤自訂事件和度量][api]，以了解使用者正對應用程式進行的動作。
* [搜尋事件和記錄][diagnostic]以協助診斷問題。

## <a name="availability-web-tests"></a>可用性 Web 測試
Application Insights 可讓您定期測試網站，以檢查網站運作中且正常回應。

[深入瞭解如何設定可用性 web 測試。][availability]

## <a name="questions-problems"></a>有問題嗎？ 有問題嗎？
[疑難排解 Java](java-troubleshoot.md)

## <a name="next-steps"></a>後續步驟
* [監視相依性呼叫](java-agent.md)
* [監視 Unix 效能計數器](java-collectd.md)
* 新增[對網頁的監視](javascript.md)，以監視頁面載入時間、AJAX 呼叫、瀏覽器例外狀況。
* 撰寫[自訂遙測](./api-custom-events-metrics.md)，以追蹤瀏覽器中或在伺服器上的使用情況。
* 使用  [分析](../log-query/log-query-overview.md) 從您的應用程式透過遙測進行強大的查詢
* 如需詳細資訊，請瀏覽[適用於 Java 開發人員的 Azure](/java/azure)。

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#trackexception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[javalogs]: java-trace-logs.md
[metrics]: ../platform/metrics-charts.md
[usage]: javascript.md