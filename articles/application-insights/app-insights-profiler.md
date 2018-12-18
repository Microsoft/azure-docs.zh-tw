---
title: 使用 Application Insights Profiler 來分析 Azure 上的即時 Web 應用程式 | Microsoft Docs
description: 使用低資源使用量的分析工具來找出 Web 伺服器程式碼中的最忙碌路徑。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.reviewer: cawa
ms.date: 08/06/2018
ms.author: mbullwin
ms.openlocfilehash: 839e462522be4f492010ca1c22631cb4dd6affe4
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064419"
---
# <a name="profile-live-azure-web-apps-with-application-insights"></a>使用 Application Insights 來分析即時 Azure Web 應用程式

Azure Application Insights 的這項功能在 Azure App Service 的 Web Apps 功能和 Azure 計算資源中已是正式運作版。 您可在這裡找到有關[在內部部署伺服器上啟用 Profiler](https://docs.microsoft.com/azure/application-insights/enable-profiler-compute#enable-profiler-on-on-premises-servers) 的資訊。

本文討論當您使用 [Application Insights](app-insights-overview.md) 時，即時 Web 應用程式的各個方法所花費的時間量。 Application Insights Profiler 工具會針對由您的應用程式所處理的即時要求來顯示詳細設定檔。 Profiler 會醒目提示使用最多時間的最忙碌路徑。 系統會根據取樣來分析具有不同回應時間的要求。 您可以使用各種技術來將應用程式相關的額外負荷降到最低。

Profiler 目前適用於在 Web Apps 上執行的 ASP.NET 和 ASP.NET Core Web 應用程式。 需要基本或更高的服務層，才可使用 Profiler。

## <a id="installation"></a> 為您的 Web Apps 啟用 Profiler

一旦您部署 Web 應用程式後，不論您是否在原始程式碼中包含 App Insights SDK，請執行下列作業：

1. 移至 Azure 入口網站中的 [App Service] 窗格。
1. 巡覽至 [設定 > 監視] 窗格。

   ![在 App Service 入口網站上啟用 Application Insights](./media/app-insights-profiler/AppInsights-AppServices.png)

1. 遵循窗格上的指示以建立新資源，或選取現有 App Insights 資源以監視您的 Web 應用程式。 接受所有預設選項。 預設會開啟 [程式碼層級診斷]，並啟用 Profiler。

   ![新增 App Insights 網站延伸模組][Enablement UI]

1. Profiler 現在會隨 App Insights 網站延伸模組而安裝，並使用應用程式服務應用程式設定來啟用。

    ![Profiler 的應用程式設定][profiler-app-setting]

### <a name="enable-profiler-for-azure-compute-resources"></a>啟用 Azure 計算資源的 Profiler

如需詳細資訊，請參閱 [Azure 計算資源的 Profiler 版本](https://go.microsoft.com/fwlink/?linkid=848155)。

## <a name="view-profiler-data"></a>檢視分析工具資料

確定應用程式正在接收流量。 如果您要進行實驗，則可以使用 [Application Insights 效能測試](https://docs.microsoft.com/vsts/load-test/app-service-web-app-performance-test)來產生 Web 應用程式的要求。 如果您剛啟用 Profiler，則可以執行大約 15 分鐘的簡短負載測試，這應該會產生分析工具追蹤。 如果您啟用 Profiler 已有一段時間，請記住，Profiler 會每個小時隨機執行兩次，每次執行兩分鐘。 建議您先執行一小時的負載測試，以確實取得分析工具追蹤樣本。

應用程式接收到流量之後，請移至 [效能] 窗格，選取 [採取動作] 以檢視分析工具追蹤，然後選取 [分析工具追蹤]。

![Application Insights [效能] 窗格預覽分析工具追蹤][performance-blade-v2-examples]

選取某個範例就會顯示要求在執行時所用時間的程式碼層級細目。

![Application Insights 追蹤總管][trace-explorer]

追蹤總管會顯示下列資訊：

* [顯示最忙碌路徑]：會開啟最大的分葉節點，或至少是類似的節點。 在大部分情況下，這個節點會接近效能瓶頸。
* **標籤**︰函式或事件的名稱。 樹狀結構會混合顯示程式碼和發生的事件 (例如 SQL 和 HTTP 事件)。 最上層事件代表要求整體的持續時間。
* **經過時間**︰作業開始和作業結束之間的時間間隔。
* **何時**︰顯示函式或事件相對於其他函式的執行時間。

## <a name="how-to-read-performance-data"></a>如何讀取效能資料

Microsoft 服務分析工具會合併使用取樣方法和檢測功能，來分析應用程式的效能。 當系統在進行詳細的收集作業時，服務分析工具會對電腦的每個 CPU 指令指標每毫秒進行取樣一次。 每次取樣都會擷取目前執行中執行緒的完整呼叫堆疊。 它會提供有關該執行緒所執行工作的詳細資訊，不論是概括還是詳細的抽象概念都有提供。 服務分析工具也會收集其他事件來追蹤活動的相互關聯和因果關係，包括內容切換事件、「工作平行程式庫」(TPL) 事件及執行緒集區事件。

時間表檢視中顯示的呼叫堆疊是取樣和檢測的結果。 由於每個範例都會擷取執行緒的完整呼叫堆疊，因此它會包含來自 Microsoft .NET Framework 和您所參考之其他架構的程式碼。

### <a id="jitnewobj"></a>物件配置 (clr!JIT\_New 或 clr!JIT\_Newarr1)

**clr!JIT\_New** 和 **clr!JIT\_Newarr1** 是 .NET Framework 內的 Helper 函式，可從受控堆積配置記憶體。 配置物件時，會叫用 **clr!JIT\_New**。 配置物件陣列時，會叫用 **clr!JIT\_Newarr1**。 這兩個函式的執行速度通常很快，花費的時間相當少。 如果您在時間表中看到 **clr!JIT\_New** 或 **clr!JIT\_Newarr1** 花費相當長的時間，即表示程式碼可能配置許多物件，而耗用大量的記憶體。

### <a id="theprestub"></a>載入程式碼 (clr!ThePreStub)

**clr!ThePreStub**是 .NET Framework 內的 Helper 函式，可讓程式碼準備好進行第一次的執行。 這通常包括但不限於 Just-In-Time (JIT) 編譯。 針對每個 C# 方法，在程序的存留期內，應該最多只會叫用 **clr!ThePreStub**一次。

如果 **clr!ThePreStub** 針對某個要求花費相當長的時間，即表示該要求是第一個執行該方法的要求。 .NET Framework 執行階段載入第一個方法的時間相當重要。 您可以考慮先使用熱身程序來執行該部分的程式碼，再讓使用者存取該程式碼，或考慮在您的組件上執行 Native Image Generator (ngen.exe)。

### <a id="lockcontention"></a>鎖定爭用 (clr!JITutil\_MonContention 或 clr!JITutil\_MonEnterWorker)

**clr!JITutil\_MonContention** 或 **clr!JITutil\_MonEnterWorker** 表示目前的執行緒正在等候鎖定被釋放。 通常在執行 C# **LOCK** 陳述式、叫用 **Monitor.Enter** 方法，或叫用含有 **MethodImplOptions.Synchronized** 屬性的方法時，會顯示此文字。 當執行緒 _A_ 已取得鎖定，但執行緒 _B_ 在執行緒 _A_ 釋放鎖定之前就嘗試取得同一鎖定，通常就會發生鎖定爭用。

### <a id="ngencold"></a>載入程式碼 ([COLD])

如果方法名稱包含 **[COLD]** (例如 **mscorlib.ni![COLD]System.Reflection.CustomAttribute.IsDefined**)，即表示 .NET Framework 執行階段是第一次執行程式碼，而該程式碼尚未由<a href="https://msdn.microsoft.com/library/e7k32f4k.aspx">特性指引最佳化</a>進行最佳化。 每個方法最多只應該會在程序的存留期內顯示一次這個名稱。

如果某個要求在載入程式碼時花費相當長的時間，即表示該要求是第一個執行該方法中未最佳化部分的要求。 請考慮先使用熱身程序來執行該部分的程式碼，再讓使用者存取該程式碼。

### <a id="httpclientsend"></a>傳送 HTTP 要求

**HttpClient.Send** 之類的方法表示程式碼正在等候 HTTP 要求完成。

### <a id="sqlcommand"></a>資料庫作業

**SqlCommand.Execute** 之類的方法表示程式碼正在等候資料庫作業完成。

### <a id="await"></a>等候 (AWAIT\_TIME)

**AWAIT\_TIME** 表示程式碼正在等候另一個工作完成。 這通常是發生在 C# **AWAIT** 陳述式。 當程式碼執行 C# **AWAIT** 時，執行緒會將控制權回溯並交還給執行緒集區，而且不會有任何執行緒被封鎖以等候 **AWAIT** 完成。 不過，就邏輯而言，執行 **AWAIT** 的執行緒會被「封鎖」並等候作業完成。 **AWAIT\_TIME** 陳述式表示為了等候工作完成而封鎖的時間。

### <a id="block"></a>封鎖時間

**BLOCKED_TIME** 表示程式碼正在等候另一個資源可供使用。 例如，它可能正在等候同步處理物件、等候執行緒可供使用，或等候要求完成。

### <a id="cpu"></a>CPU 時間

CPU 正忙於執行指令。

### <a id="disk"></a>磁碟時間

應用程式正在執行磁碟作業。

### <a id="network"></a>網路時間

應用程式正在執行網路作業。

### <a id="when"></a>何時資料行

[何時] 資料行會以視覺方式呈現針對節點收集的 INCLUSIVE 樣本如何隨著時間變化。 要求的總範圍會分成 32 個時間貯體。 該節點的內含範例會在這 32 個貯體中累積。 每個貯體都以一個長條表示。 長條的高度代表換算值。 節點如果標示為 **CPU_TIME** 或 **BLOCKED_TIME**，或是與耗用資源 (例如，CPU、磁碟或執行緒) 有明顯的關聯性，長條便代表在該貯體的期間內耗用這其中一個資源。 如果耗用多個資源，這些計量的值便有可能大於 100%。 例如，如果您在一段間隔內平均使用兩個 CPU，值就會是 200%。

## <a name="limitations"></a>限制

預設資料保留期為 5 天。 每天的內嵌資料上限為 10 GB。

使用 Profiler 服務為免費。 若要使用 Profiler 服務，您的 Web 應用程式必須至少裝載在 Web Apps 的「基本」層。

## <a name="overhead-and-sampling-algorithm"></a>額外負荷和取樣演算法

Profiler 會在裝載已啟用 Profiler 之應用程式的每部虛擬機器上，每小時隨機執行 2 分鐘來擷取追蹤。 Profiler 在執行時會對伺服器增加 5% 到 15% 的 CPU 額外負荷。

可供裝載應用程式的伺服器越多，Profiler 對整體應用程式效能的影響就越小。 這是因為取樣演算法會使得 Profiler 在任何時間都只在 5% 的伺服器上執行。 可服務 Web 要求的伺服器越多，就越能抵消執行 Profiler 所造成的伺服器額外負荷。

## <a name="disable-profiler"></a>停用 Profiler

若要停止或重新啟動個別 Web 應用程式執行個體的 Profiler，請在 [Web 作業] 底下，移至該 Web Apps 資源。 若要刪除 Profiler，請移至 [擴充功能]。

![停用 Web 作業的 Profiler][disable-profiler-webjob]

建議您在所有 Web 應用程式上都啟用 Profiler，以儘早找出任何效能問題。

如果您使用 WebDeploy 來部署對 Web 應用程式所做的變更，請確定您已排除 [App_Data] 資料夾，以免系統在部署期間刪除它。 否則，下次您將 Web 應用程式部署到 Azure 時，系統就會刪除 Profiler 擴充功能的檔案。


## <a id="troubleshooting"></a>疑難排解

### <a name="too-many-active-profiling-sessions"></a>太多個使用中分析工作階段

目前，您最多可以在於相同服務方案中執行的 4 個 Azure Web 應用程式及部署位置上啟用 Profiler。 如果 Profiler Web 作業回報太多作用中分析工作階段，請將一些 Web 應用程式移至不同的服務方案。

### <a name="how-do-i-determine-whether-application-insights-profiler-is-running"></a>如何判斷 Application Insights Profiler 是否有在執行？

Profiler 會在 Web 應用程式中以連續性 Web 作業的形式執行。 您可以在 [Azure 入口網站](https://portal.azure.com)中開啟 Web 應用程式資源。 在 [WebJobs] 窗格中，檢查 **ApplicationInsightsProfiler** 的狀態。 如果它並沒有在執行，請開啟 [記錄] 以取得詳細資訊。

### <a name="why-cant-i-find-any-stack-examples-even-though-profiler-is-running"></a>為什麼 Profiler 明明已在執行，我卻找不到任何堆疊範例？

以下是您可以檢查的幾點：

* 確定 Web 應用程式服務方案至少是「基本」層。
* 確定 Web 應用程式已啟用 Application Insights SDK 2.2 Beta 或更新版本。
* 確定 Web 應用程式的 **APPINSIGHTS_INSTRUMENTATIONKEY** 設定所擁有的檢測金鑰與 Application Insights SDK 所使用的金鑰相同。
* 確定 Web 應用程式是在 .NET Framework 4.6 上執行。
* 如果 Web 應用程式是 ASP.NET Core 應用程式，則必須至少執行 ASP.NET Core 2.0。

在啟動 Profiler 之後，會有一小段熱身時間，Profiler 會利用這段時間積極收集幾項效能追蹤資料。 之後，Profiler 會每隔一小時花兩分鐘時間來收集效能追蹤資料。

> [!NOTE]
> 分析工具代理程式中有一個錯誤，使其無法上傳從在 ASP.NET Core 2.1 上執行的應用程式取得的追蹤資料。 我們正努力修正錯誤，盡快讓它準備就緒。

### <a name="i-was-using-azure-service-profiler-what-happened-to-it"></a>我之前使用的是 Azure Service 分析工具。 它有什麼改變？

當您啟用 Application Insights Profiler 時，系統會將 Azure Service 分析工具代理程式停用。

### <a id="double-counting"></a>平行執行緒重複計算

在某些情況下，堆疊檢視器中的總時間計量會比要求的持續時間還長。

當要求有兩個或更多個關聯的執行緒，且這些執行緒以平行方式運作時，就會發生這種情形。 在此情況下，執行緒的總時間就會超出經過時間。 一個執行緒可能會等候另一個執行緒完成。 檢視器會嘗試偵測這個狀況並省略不重要的等候，但它會寧可顯示過多資訊，以免省略可能重要的資訊。

當您在追蹤資料內看到平行執行緒時，請判斷正在等候的執行緒，以便確定要求的關鍵路徑。 在大部分情況下，快速進入等候狀態的執行緒就只是在等候其他執行緒。 請專注在其他執行緒上，並忽略等候中執行緒的時間。

### <a id="issue-loading-trace-in-viewer"></a>沒有分析資料

以下是您可以檢查的幾點：

* 如果您嘗試檢視的資料已存在好幾週，請試著限縮時間篩選器，然後再試一次。
* 確定系統未阻止 Proxy 或防火牆存取 https://gateway.azureserviceprofiler.net。
* 確定您在應用程式中使用的 Application Insights 檢測金鑰，與您用來啟用分析功能的 Application Insights 資源相同。 此金鑰通常位於 ApplicationInsights.config 檔案中，但也可能在 web.config 或 app.config 檔案中。

### <a name="error-report-in-the-profiling-viewer"></a>分析檢視器的錯誤報表

您可以在入口網站中提交支援票證。 請務必包含錯誤訊息中的相互關聯識別碼。

### <a name="deployment-error-directory-not-empty-dhomesitewwwrootappdatajobs"></a>部署錯誤：目錄尚有資料 'D:\\home\\site\\wwwroot\\App_Data\\jobs'

如果您要將 Web 應用程式重新部署到已啟用 Profiler 的 Web Apps 資源，您可能會看到類似以下的訊息：

目錄尚有資料 'D:\\home\\site\\wwwroot\\App_Data\\jobs'

如果您從指令碼或從 Azure DevOps 部署管線執行 Web Deploy，就會發生此錯誤。 解決方法是將下列額外的部署參數新增至 Web Deploy 工作：

```
-skip:Directory='.*\\App_Data\\jobs\\continuous\\ApplicationInsightsProfiler.*' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs\\continuous$' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs$'  -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data$'
```

這些參數會刪除 Application Insights Profiler 所使用的資料夾，並將重新部署程序解除封鎖。 它們並不會影響目前正在執行的 Profiler 執行個體。

## <a name="manual-installation"></a>手動安裝

當您設定 Profiler 時，系統會對 Web 應用程式的設定進行更新。 您可以視環境需要，手動套用這些更新。 可能的範例為您的應用程式在 PowerApps 的 Web Apps 環境中執行。

1. 在 [Web 應用程式控制項] 窗格中，開啟 [設定]。
1. 將 [.Net Framework 版本] 設定為 [v4.6]。
1. 將 [一律開啟] 設定為 [開啟]。
1. 新增 **APPINSIGHTS_INSTRUMENTATIONKEY** 應用程式設定，並將值設定為與 SDK 所用相同的檢測金鑰。
1. 開啟 [進階工具]。
1. 選取 [執行] 以開啟 Kudu 網站。
1. 在 Kudu 網站上，選取 [網站延伸模組]。
1. 從 Azure Web Apps 資源庫安裝 [Application Insights]。
1. 重新啟動 Web 應用程式。

## <a id="profileondemand"></a>手動觸發 Profiler

按一個按鈕即可手動觸發 Profiler。 假設您正在執行 Web 效能測試。 您需要進行追蹤，以利於了解如何在負載下執行 Web 應用程式。 在擷取追蹤時擁有控制權十分重要，因為您知道何時會執行負載測試，但隨機取樣間隔可能會錯過這項資訊。
下列步驟說明此案例的運作方式：

### <a name="optional-step-1-generate-traffic-to-your-web-app-by-starting-a-web-performance-test"></a>(選擇性) 步驟 1：啟動 Web 效能測試來產生 Web 應用程式的流量

如果您的 Web 應用程式已經有連入流量，或您只想要以手動方式產生流量，請略過本節，並繼續執行步驟 2。

巡覽至 Application Insights 入口網站中的 [設定 > 效能測試]。 按一下 [新增] 按鈕來啟動新的效能測試。
![建立新的效能測試][create-performance-test]

在 [新增效能測試] 窗格中，設定測試目標 URL。 接受所有的預設設定，並開始執行負載測試。

![設定負載測試][configure-performance-test]

您會看到新的測試會先排入佇列，隨後接著進行中的狀態。

![系統隨即會提交負載測試並排入佇列][load-test-queued]

![負載測試執行正在進行中][load-test-in-progress]

### <a name="step-2-start-profiler-on-demand"></a>步驟 2：啟動隨選 Profiler

一旦負載測試執行後，我們就可以啟動 Profiler，在其接收負載時，擷取 Web 應用程式上的追蹤。
巡覽至 [設定 Profiler] 窗格：

![設定 Profiler 窗格項目][configure-profiler-entry]

在 [設定 Profiler] 窗格中，已連結 Web 應用程式的所有執行個體上，都會有一個用以觸發 Profiler 的 [立即分析] 按鈕。 此外，您還可以看見 Profiler 過去的執行。

![隨選 Profiler][profiler-on-demand]

您會看到 Profiler 執行狀態上的通知和狀態變更。

### <a name="step-3-view-traces"></a>步驟 3：檢視追蹤

一旦 Profiler 完成執行後，請遵循通知中的指示，移至 [效能] 刀鋒視窗並檢視追蹤。

### <a name="troubleshooting-on-demand-profiler"></a>針對隨選 Profiler 進行疑難排解

有時候，您可能會在隨選工作階段之後看到 Profiler 逾時錯誤訊息：

![Profiler 逾時錯誤][profiler-timeout]

您會看到此錯誤的原因可能有兩個：

1. 隨選 Profiler 工作階段已成功，但 Application Insights 花費了較長時間來處理所收集的資料。 如果資料未在 15 分鐘內完成處理，入口網站就會顯示逾時訊息。 但一段時間之後，會顯示 Profiler 追蹤。 如果發生這種情況，目前只需忽略錯誤訊息。 我們正積極修正

1. 您的 Web 應用程式中，Profiler 代理程式版本較舊且沒有隨選功能。 如果您先前已啟用 Application Insights 設定檔，則您可能必須更新您的 Profiler 代理程式，才能開始使用隨選功能。
  
請遵循下列步驟來檢查並安裝最新的 Profiler：

1. 移至 [應用程式服務應用程式設定]，並檢查是否已設定下列設定：
    * **APPINSIGHTS_INSTRUMENTATIONKEY**：取代為 Application Insights 的適當檢測金鑰。
    * **APPINSIGHTS_PORTALINFO**：ASP.NET
    * **APPINSIGHTS_PROFILERFEATURE_VERSION**：1.0.0 如果未設定這些設定，請移至 Application Insights 啟用窗格，來安裝最新的網站延伸模組。

1. 請移至應用程式服務入口網站中的 Application Insights 窗格。

    ![從應用程式服務入口網站啟用 Application Insights][enable-app-insights]

1. 如果您在下列頁面中看到 [更新] 按鈕，請按一下以更新 Application Insights 網站延伸模組，這會安裝最新的 Profiler 代理程式。
![更新網站延伸模組][update-site-extension]

1. 然後按一下 [變更]，來確定 Profiler 已開啟，並選取 [確定] 以儲存變更。

    ![變更並儲存 App Insights][change-and-save-appinsights]

1. 返回 App Service 的 [應用程式設定] 索引標籤，再次檢查已設定下列應用程式設定項目：
    * **APPINSIGHTS_INSTRUMENTATIONKEY**：取代為 Application Insights 的適當檢測金鑰。
    * **APPINSIGHTS_PORTALINFO**：ASP.NET
    * **APPINSIGHTS_PROFILERFEATURE_VERSION**：1.0.0

    ![Profiler 的應用程式設定][app-settings-for-profiler]

1. (選擇性) 請檢查延伸模組版本，並確定沒有可用的更新。

    ![檢查延伸模組更新][check-for-extension-update]

## <a name="next-steps"></a>後續步驟

* [在 Visual Studio 中使用 Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-visual-studio)

[appinsights-in-appservices]:./media/app-insights-profiler/AppInsights-AppServices.png
[Enablement UI]: ./media/app-insights-profiler/Enablement_UI.png
[profiler-app-setting]:./media/app-insights-profiler/profiler-app-setting.png
[performance-blade]: ./media/app-insights-profiler/performance-blade.png
[performance-blade-examples]: ./media/app-insights-profiler/performance-blade-examples.png
[performance-blade-v2-examples]:./media/app-insights-profiler/performance-blade-v2-examples.png
[trace-explorer]: ./media/app-insights-profiler/trace-explorer.png
[trace-explorer-toolbar]: ./media/app-insights-profiler/trace-explorer-toolbar.png
[trace-explorer-hint-tip]: ./media/app-insights-profiler/trace-explorer-hint-tip.png
[trace-explorer-hot-path]: ./media/app-insights-profiler/trace-explorer-hot-path.png
[enable-profiler-banner]: ./media/app-insights-profiler/enable-profiler-banner.png
[disable-profiler-webjob]: ./media/app-insights-profiler/disable-profiler-webjob.png
[linked app services]: ./media/app-insights-profiler/linked-app-services.png
[create-performance-test]: ./media/app-insights-profiler/new-performance-test.png
[configure-performance-test]: ./media/app-insights-profiler/configure-performance-test.png
[load-test-queued]: ./media/app-insights-profiler/load-test-queued.png
[load-test-in-progress]: ./media/app-insights-profiler/load-test-inprogress.png
[profiler-on-demand]: ./media/app-insights-profiler/Profiler-on-demand.png
[configure-profiler-entry]: ./media/app-insights-profiler/configure-profiler-entry.png
[enable-app-insights]: ./media/app-insights-profiler/enable-app-insights-blade-01.png
[update-site-extension]: ./media/app-insights-profiler/update-site-extension-01.png
[change-and-save-appinsights]: ./media/app-insights-profiler/change-and-save-appinsights-01.png
[app-settings-for-profiler]: ./media/app-insights-profiler/appsettings-for-profiler-01.png
[check-for-extension-update]: ./media/app-insights-profiler/check-extension-update-01.png
[profiler-timeout]: ./media/app-insights-profiler/profiler-timeout.png
