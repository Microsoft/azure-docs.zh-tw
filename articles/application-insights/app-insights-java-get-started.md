---
title: 使用 Azure Application Insights 的 Java Web 應用程式分析 | Microsoft Docs
description: '使用 Application Insights 針對 Java Web 應用程式進行應用程式效能監視。 '
services: application-insights
documentationcenter: java
author: lgayhardt
manager: carmonm
ms.assetid: 051d4285-f38a-45d8-ad8a-45c3be828d91
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 10/09/2018
ms.author: lagayhar
ms.openlocfilehash: 216cebe74b7661e0ca336480774a56ea953ddc75
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49069466"
---
# <a name="get-started-with-application-insights-in-a-java-web-project"></a>在 Java Web 專案中開始使用 Application Insights


[Application Insights](https://azure.microsoft.com/services/application-insights/) 是一項 Web 開發人員可延伸的分析服務，可幫助您了解即時應用程式的效能和使用情形。 使用它可[偵測及診斷效能問題和例外狀況](app-insights-detect-triage-diagnose.md)，以及[撰寫程式碼][api]來追蹤使用者對應用程式執行的動作。

![概觀範例資料的螢幕擷取畫面](./media/app-insights-java-get-started/overview-graphs.png)

Application Insights 支援 Linux、Unix 或 Windows 上執行的 Java 應用程式。

您需要：

* JRE 1.7 或 1.8 版
* [Microsoft Azure](https://azure.microsoft.com/)訂用帳戶。

如果您有使用中的 Web 應用程式，您可以依照替代的程序[在執行階段於 Web 伺服器中新增 SDK ](app-insights-java-live.md)。替代方法可避免重建程式碼，但您沒有選項可撰寫程式碼來追蹤使用者活動。

如果您偏好 Spring 架構，可嘗試[設定 Spring Boot 初始設定式應用程式來使用 Application Insights 指南](https://docs.microsoft.com/java/azure/spring-framework/configure-spring-boot-java-applicationinsights) \(英文\)。

## <a name="1-get-an-application-insights-instrumentation-key"></a>1.取得 Application Insights 檢測金鑰
1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。
2. 建立 Application Insights 資源。 將應用程式類型設定為 Java Web 應用程式。

    ![填寫名稱，選擇 [Java Web 應用程式]，然後按一下 [建立]](./media/app-insights-java-get-started/02-create.png)
3. 尋找新資源的檢測金鑰。 您很快需要將此金鑰貼到程式碼專案中。

    ![在新資源概觀中，按一下 [屬性] 並複製檢測金鑰](./media/app-insights-java-get-started/03-key.png)

## <a name="2-add-the-application-insights-sdk-for-java-to-your-project"></a>2.將 Java 適用的 Application Insights SDK 加入至專案
*選擇適合您的專案的方式。*

#### <a name="if-youre-using-maven-a-namemaven-setup-"></a>如果您使用 Maven... <a name="maven-setup" />
如果您的專案已設定為使用 Maven 來建置，請將下列程式碼合併至 pom.xml 檔案。

然後重新整理專案相依性，以下載程式庫。

```XML

    <repositories>
       <repository>
          <id>central</id>
          <name>Central</name>
          <url>http://repo1.maven.org/maven2</url>
       </repository>
    </repositories>

    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-web</artifactId>
        <!-- or applicationinsights-core for bare API -->
        <version>[2.0,)</version>
      </dependency>
    </dependencies>
```

* *建置或總和檢查碼驗證錯誤？* 嘗試使用特定版本，例如：`<version>2.0.n</version>`。 您可以在 [SDK 版本資訊](https://github.com/Microsoft/ApplicationInsights-Java#release-notes)或 [Maven 成品](http://search.maven.org/#search%7Cga%7C1%7Capplicationinsights)中找到最新版本。
* *需要更新為新的 SDK？* 請重新整理專案的相依項目。

#### <a name="if-youre-using-gradle-a-namegradle-setup-"></a>如果您使用 Gradle... <a name="gradle-setup" />
如果您的專案已設定為要使用 Gradle 建置，請將下列程式碼合併至 build.gradle 檔案。

然後重新整理專案相依性，以下載程式庫。

```gradle

    repositories {
      mavenCentral()
    }

    dependencies {
      compile group: 'com.microsoft.azure', name: 'applicationinsights-web', version: '2.+'
      // or applicationinsights-core for bare API
    }
```

#### <a name="if-youre-using-eclipse-to-create-a-dynamic-web-project-"></a>如果您使用 Eclipse 建立動態 Web 專案...
使用 [Java 適用的 Application Insights SDK 外掛程式][eclipse]。 注意：雖然使用此外掛程式可讓您較快啟動並執行 Application Insights (假設您未使用 Maven/Gradle)，但它並不是相依性管理系統。 因此，更新此外掛程式並不會自動更新專案中的 Application Insights 程式庫。

* *建置或總和檢查碼驗證錯誤？* 嘗試使用特定版本，例如：`version:'2.0.n'`。 您可以在 [SDK 版本資訊](https://github.com/Microsoft/ApplicationInsights-Java#release-notes)或 [Maven 成品](http://search.maven.org/#search%7Cga%7C1%7Capplicationinsights)中找到最新版本。
* *若要更新為新的 SDK* 請重新整理專案的相依項目。

#### <a name="otherwise-if-you-are-manually-managing-dependencies-"></a>否則，如果您手動管理相依項目...
下載[最新版本](https://github.com/Microsoft/ApplicationInsights-Java/releases/latest)並將所需的檔案複製到您的專案中，並取代任何先前的版本。

### <a name="questions"></a>問題...
* `-core` 與 `-web` 元件之間有何關係？
  * `applicationinsights-core` 會提供裸機 API。 您一定需要此元件。
  * `applicationinsights-web` 提供追蹤 HTTP 要求計數和回應時間的度量。 如果您不想自動收集此原則，您可以忽略這個。 例如，如果您想要自己撰寫。
  
* 如果將 SDK 升級為最新版本？
  * 如果您使用 Gradle 或 Maven...
    * 更新您的組建檔案，以指定最新版本或使用 Gradle/Maven 的萬用字元語法自動包含最新版本。 然後，重新整理專案的相依項目。 在上述範例中可以看到 [Gradle](#gradle-setup) 或 [Maven](#maven-setup) 的萬用字元語法。
  * 如果您手動管理相依項目...
    * 下載最新的 [Application Insights SDK for Java](https://github.com/Microsoft/ApplicationInsights-Java/releases/latest) 並取代舊的。 [SDK 版本資訊](https://github.com/Microsoft/ApplicationInsights-Java#release-notes)中會說明變更內容。

## <a name="3-add-an-applicationinsightsxml-file"></a>3.新增 ApplicationInsights.xml 檔案
在專案中的資源資料夾中加入 ApplicationInsights.xml，或確定已將其加入專案部署類別路徑。 將下列 XML 複製到其中。

替換為您從 Azure 入口網站取得的檢測金鑰。

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

(選擇性) 組態檔可以位於您的應用程式可存取的任何位置。  系統屬性 `-Dapplicationinsights.configurationDirectory` 會指定包含 ApplicationInsights.xml 的目錄。 例如，位於 `E:\myconfigs\appinsights\ApplicationInsights.xml` 的組態檔是使用屬性 `-Dapplicationinsights.configurationDirectory="E:\myconfigs\appinsights"` 來設定。

* 檢測金鑰會隨著遙測的每個項目傳送，並告知 Application Insights 在您的資源中顯示它。
* HTTP 要求元件是選用的。 它會自動將要求和回應時間的遙測傳送到入口網站。
* 事件相互關聯是 HTTP 要求元件的補充。 它會將識別碼指派給伺服器收到的每個要求，並將此識別碼加入為遙測的每個項目的屬性，作為 'Operation.Id' 屬性。 它可讓您相互關聯與每個要求關聯的遙測，方法是在[診斷搜尋][diagnostic]中設定篩選器。

### <a name="alternative-ways-to-set-the-instrumentation-key"></a>設定檢測金鑰的替代方法
Application Insights SDK 會依此順序尋找此金鑰︰

1. 系統屬性：-DAPPLICATION_INSIGHTS_IKEY=your_ikey
2. 環境變數：APPLICATION_INSIGHTS_IKEY
3. 組態檔︰ApplicationInsights.xml

您也可以 [在程式碼中設定](app-insights-api-custom-events-metrics.md#ikey)：

```Java
    TelemetryConfiguration.getActive().setInstrumentationKey(iKey);
```

## <a name="4-add-an-http-filter"></a>4.加入 HTTP 篩選器
上一個組態步驟可讓 HTTP 要求元件記錄每個 Web 要求。 (如果您只需要單純的 API，則非必要。)

### <a name="spring-boot-applications"></a>Spring Boot 應用程式
在您的組態類別中註冊 Application Insights `WebRequestTrackingFilter`：

```Java
package <yourpackagename>.configurations;

import javax.servlet.Filter;

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.core.Ordered;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import com.microsoft.applicationinsights.TelemetryConfiguration;
import com.microsoft.applicationinsights.web.internal.WebRequestTrackingFilter;

@Configuration
public class AppInsightsConfig {

    @Bean
    public String telemetryConfig() {
        String telemetryKey = System.getenv("<instrumentation key>");
        if (telemetryKey != null) {
            TelemetryConfiguration.getActive().setInstrumentationKey(telemetryKey);
        }
        return telemetryKey;
    }

    /**
     * Programmatically registers a FilterRegistrationBean to register WebRequestTrackingFilter
     * @param webRequestTrackingFilter
     * @return Bean of type {@link FilterRegistrationBean}
     */
    @Bean
    public FilterRegistrationBean webRequestTrackingFilterRegistrationBean(WebRequestTrackingFilter webRequestTrackingFilter) {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(webRequestTrackingFilter);
        registration.addUrlPatterns("/*");
        registration.setOrder(Ordered.HIGHEST_PRECEDENCE + 10);
        return registration;
    }


    /**
     * Creates bean of type WebRequestTrackingFilter for request tracking
     * @param applicationName Name of the application to bind filter to
     * @return {@link Bean} of type {@link WebRequestTrackingFilter}
     */
    @Bean
    @ConditionalOnMissingBean

    public WebRequestTrackingFilter webRequestTrackingFilter(@Value("${spring.application.name:application}") String applicationName) {
        return new WebRequestTrackingFilter(applicationName);
    }


}
```

> [!NOTE]
> 如果您使用 Spring Boot 1.3.8 或較舊的版本，請以下列的行取代 FilterRegistrationBean

```Java
    import org.springframework.boot.context.embedded.FilterRegistrationBean;
```

此類別會將 `WebRequestTrackingFilter` 設定為 http 篩選鏈結上的第一個篩選條件。 它也會從作業系統環境變數中提取可用的檢測金鑰。

> 由於這是 Spring Boot 應用程式，而且它有其本身的 Spring MVC 組態，因此我們會使用 web http 篩選組態，而不是 Spring MVC 組態。 請參閱以下幾節，以了解 Spring MVC 特定組態。

### <a name="applications-using-webxml"></a>使用 Web.xml 的應用程式
在您的專案中找到並開啟 web.xml 檔案，然後將下列程式碼合併至 Web 應用程式節點下，也就是應用程式篩選器設定的位置。

為獲得最準確的結果，應該在其他所有篩選器之前先對應此篩選器。

```XML

    <filter>
      <filter-name>ApplicationInsightsWebFilter</filter-name>
      <filter-class>
        com.microsoft.applicationinsights.web.internal.WebRequestTrackingFilter
      </filter-class>
    </filter>
    <filter-mapping>
       <filter-name>ApplicationInsightsWebFilter</filter-name>
       <url-pattern>/*</url-pattern>
    </filter-mapping>

   <!-- This listener handles shutting down the TelemetryClient when an application/servlet is undeployed. -->
    <listener>
      <listener-class>com.microsoft.applicationinsights.web.internal.ApplicationInsightsServletContextListener</listener-class>
    </listener>
```

#### <a name="if-youre-using-spring-web-mvc-31-or-later"></a>如果您使用 Spring Web MVC 3.1 或更新版本
在 *-servlet.xml 中編輯這些元素來併入 Application Insights 封裝：

```XML

    <context:component-scan base-package=" com.springapp.mvc, com.microsoft.applicationinsights.web.spring"/>

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean class="com.microsoft.applicationinsights.web.spring.RequestNameHandlerInterceptorAdapter" />
        </mvc:interceptor>
    </mvc:interceptors>
```

#### <a name="if-youre-using-struts-2"></a>如果您使用 Struts 2
將此項目加入到 Struts 組態檔案 (通常名稱為 struts.xml 或 struts-default.xml)：

```XML

     <interceptors>
       <interceptor name="ApplicationInsightsRequestNameInterceptor" class="com.microsoft.applicationinsights.web.struts.RequestNameInterceptor" />
     </interceptors>
     <default-interceptor-ref name="ApplicationInsightsRequestNameInterceptor" />
```

如果預設堆疊中定義了攔截器，可以將攔截器新增至該堆疊。

## <a name="5-run-your-application"></a>5.執行您的應用程式
在您的開發電腦上以偵錯模式執行應用程式，或發佈至您的伺服器。

## <a name="6-view-your-telemetry-in-application-insights"></a>6.在 Application Insights 中檢視遙測
返回 [Microsoft Azure 入口網站](https://portal.azure.com)中的 Application Insights 資源。

[概觀] 刀鋒視窗上會顯示 HTTP 要求資料。 (如果沒有出現，請稍等片刻，然後按一下 [重新整理]。)

![範例資料](./media/app-insights-java-get-started/5-results.png)

[深入了解度量。][metrics]

按一下任何圖表以查看詳細彙總度量。

![](./media/app-insights-java-get-started/6-barchart.png)

> Application Insights 假設 MVC 應用程式的 HTTP 要求的格式為： `VERB controller/action`。 例如，`GET Home/Product/f9anuh81`、`GET Home/Product/2dffwrf5` 和 `GET Home/Product/sdf96vws` 會分組至 `GET Home/Product`。 此分組方式可提供要求有意義的彙總，例如要求數量和要求的平均執行時間。
>
>

### <a name="instance-data"></a>執行個體資料
點選特定要求類型以查看個別執行個體。

Application Insights 中會顯示兩種類型的資料︰彙總資料 (儲存並顯示為平均值、計數和總和)；以及執行個體資料 (HTTP 要求、例外狀況、頁面檢視或自訂事件的個別報表)。

檢視要求的屬性時，您可以查看與它關聯的遙測事件，例如要求和例外狀況。

![](./media/app-insights-java-get-started/7-instance.png)

### <a name="analytics-powerful-query-language"></a>分析︰功能強大的查詢語言
當您累積更多資料時，您就可以執行查詢以彙總資料並找出個別執行個體。  [分析](app-insights-analytics.md) 是一項強大的工具，既可了解效能和使用情況，也可進行診斷。

![分析的範例](./media/app-insights-java-get-started/025.png)

## <a name="7-install-your-app-on-the-server"></a>7.在伺服器上安裝您的 App
現在將您的應用程式發佈至伺服器供人使用，然後查看入口網站顯示的遙測。

* 請確定您的防火牆允許應用程式將遙測傳送至這些連接埠：

  * dc.services.visualstudio.com:443
  * f5.services.visualstudio.com:443

* 如果連出流量必須透過防火牆路由傳送，定義系統屬性 `http.proxyHost` 和 `http.proxyPort`。

* 在 Windows 伺服器上，安裝：

  * [Microsoft Visual C++ 可轉散發套件](http://www.microsoft.com/download/details.aspx?id=40784)

    (此元件會啟用效能計數器。)


## <a name="exceptions-and-request-failures"></a>例外狀況與要求失敗
會自動收集未處理的例外狀況：

![開啟 [設定]、[失敗]](./media/app-insights-java-get-started/21-exceptions.png)

若要收集其他例外狀況的資料，您有兩個選項：

* [在您的程式碼中插入 trackException() 的呼叫][apiexceptions]。
* [在伺服器上安裝 Java 代理程式](app-insights-java-agent.md)。 指定您想要觀看的方法。

## <a name="monitor-method-calls-and-external-dependencies"></a>監視方法呼叫和外部相依性
[安裝 Java 代理程式](app-insights-java-agent.md) 以記錄指定的內部方法和透過 JDBC 發出的呼叫與計時資料。

## <a name="performance-counters"></a>效能計數器
開啟 [設定]、[伺服器]，可看到一些效能計數器。

![](./media/app-insights-java-get-started/11-perf-counters.png)

### <a name="customize-performance-counter-collection"></a>自訂效能計數器集合
若要停用效能計數器的一組標準集合，請將下列程式碼加入 ApplicationInsights.xml 檔案的根節點下：

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
* `type` (選用) - JMX 物件屬性的類型：
  * 預設值：簡易類型，例如 int 或 long。
  * `composite`：效能計數器資料的格式為 'Attribute.Data'
  * `tabular`：效能計數器資料的格式為資料表列

#### <a name="windows-performance-counters"></a>Windows 效能計數器
每個 [Windows 效能計數器](https://msdn.microsoft.com/library/windows/desktop/aa373083.aspx) 是類別的成員 (以欄位是類別成員的相同方式)。 類別可以是全域，或可以有一定數量或指定的執行個體。

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

您的效能計數器會在[計量瀏覽器][metrics]中以自訂度量的形式顯示。

![](./media/app-insights-java-get-started/12-custom-perfs.png)

### <a name="unix-performance-counters"></a>Unix 效能計數器
* [使用 Application Insights 外掛程式安裝 collectd](app-insights-java-collectd.md) ，來取得各種不同的系統和網路資料。

## <a name="local-forwarder"></a>本機轉送工具

[本機轉送工具](https://docs.microsoft.com/azure/application-insights/local-forwarder)是會從各種 SDK 和架構收集 Application Insights 或 [OpenCensus](https://opencensus.io/) 遙測資料的代理程式，並且會將這些資料路由至 Application Insights。 此工具能夠在 Windows 和 Linux 下執行。

```xml
<Channel type="com.microsoft.applicationinsights.channel.concrete.localforwarder.LocalForwarderTelemetryChannel">
<DeveloperMode>false</DeveloperMode>
<EndpointAddress><!-- put the hostname:port of your LocalForwarder instance here --></EndpointAddress>
<!-- The properties below are optional. The values shown are the defaults for each property -->
<FlushIntervalInSeconds>5</FlushIntervalInSeconds><!-- must be between [1, 500]. values outside the bound will be rounded to nearest bound -->
<MaxTelemetryBufferCapacity>500</MaxTelemetryBufferCapacity><!-- units=number of telemetry items; must be between [1, 1000] -->
</Channel>
```

如果您使用 SpringBoot 入門版，請將下列項目新增至組態檔 (application.properies)：

```yml
azure.application-insights.channel.local-forwarder.endpoint-address=<!--put the hostname:port of your LocalForwarder instance here-->
azure.application-insights.channel.local-forwarder.flush-interval-in-seconds=<!--optional-->
azure.application-insights.channel.local-forwarder.max-telemetry-buffer-capacity=<!--optional-->
```

SpringBoot application.properties 和 applicationinsights.xml 組態的預設值相同。

## <a name="get-user-and-session-data"></a>取得使用者與工作階段資料
好了，您現在正在從 Web 服務傳送遙測資料。 現在若要取得應用程式的完整 360 度檢視，可以加入多個監視：

* [將遙測加入至您的網頁][usage]，即可監視頁面檢視和使用者度量。
* [設定 Web 測試][availability]，以確認應用程式處於線上狀態且能夠回應。

## <a name="capture-log-traces"></a>擷取記錄追蹤
您可以使用 Application Insights 將來自 Log4J、Logback 或其他記錄架構的記錄分解。 您可以將記錄與 HTTP 要求和其他遙測相互關聯。 [了解作法][javalogs]。

## <a name="send-your-own-telemetry"></a>傳送您自己的遙測
既然您已安裝 SDK，您可以使用 API 來傳送自己的遙測。

* [追蹤自訂事件和度量][api]，以了解使用者正對應用程式進行的動作。
* [搜尋事件和記錄][diagnostic]以協助診斷問題。

## <a name="availability-web-tests"></a>可用性 Web 測試
Application Insights 可讓您定期測試網站，以檢查網站運作中且正常回應。 [若要設定][availability]，請按一下 Web 測試。

![按一下 [Web 測試]，然後按一下 [新增 Web 測試]](./media/app-insights-java-get-started/31-config-web-test.png)

您將取得回應時間的圖表，以及若網站關閉還會取得電子郵件通知。

![Web 測試範例](./media/app-insights-java-get-started/appinsights-10webtestresult.png)

[深入了解 Web 測試的可用性。][availability]

## <a name="questions-problems"></a>有疑問嗎？ 有問題嗎？
[疑難排解 Java](app-insights-java-troubleshoot.md)

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>後續步驟
* [監視相依性呼叫](app-insights-java-agent.md)
* [監視 Unix 效能計數器](app-insights-java-collectd.md)
* 新增[對網頁的監視](app-insights-javascript.md)，以監視頁面載入時間、AJAX 呼叫、瀏覽器例外狀況。
* 撰寫[自訂遙測](app-insights-api-custom-events-metrics.md)，以追蹤瀏覽器中或在伺服器上的使用情況。
* 建立[儀表板](app-insights-dashboards.md)，以結合重要圖表來監視您的系統。
* 使用[分析](app-insights-analytics.md)功能從您的應用程式透過遙測進行強大查詢
* 如需詳細資訊，請瀏覽[適用於 Java 開發人員的 Azure](/java/azure)。

<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[apiexceptions]: app-insights-api-custom-events-metrics.md#trackexception
[availability]: app-insights-monitor-web-app-availability.md
[diagnostic]: app-insights-diagnostic-search.md
[eclipse]: /app-insights-java-quick-start.md
[javalogs]: app-insights-java-trace-logs.md
[metrics]: app-insights-metrics-explorer.md
[usage]: app-insights-javascript.md
