---
title: Azure Web Apps 上節點應用程式的最佳作法和疑難排解指南
description: 了解 Azure Web Apps 上節點應用程式的最佳作法和疑難排解步驟。
services: app-service\web
documentationcenter: nodejs
author: ranjithr
manager: wadeh
editor: ''
ms.assetid: 387ea217-7910-4468-8987-9a1022a99bef
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 11/09/2017
ms.author: ranjithr
ms.openlocfilehash: 860874ed49056e6b4695c060b06bf061820c390e
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31789668"
---
# <a name="best-practices-and-troubleshooting-guide-for-node-applications-on-azure-web-apps"></a>Azure Web Apps 上節點應用程式的最佳作法和疑難排解指南

在本文中，您可以了解在 Azure Web Apps 上執行之[節點應用程式](app-service-web-get-started-nodejs.md)的最佳作法和疑難排解步驟 (透過 [iisnode](https://github.com/azure/iisnode))。

> [!WARNING]
> 請小心在您的生產網站上使用疑難排解步驟。 建議在非生產安裝 (例如您的預備位置) 上對應用程式進行疑難排解，而當問題修正後，請切換您的預備位置與生產位置。
>

## <a name="iisnode-configuration"></a>IISNODE 組態

此[結構描述檔案](https://github.com/Azure/iisnode/blob/master/src/config/iisnode_schema_x64.xml)會顯示可針對 iisnode 設定的所有設定。 適用於您的應用程式的一些設定如下︰

### <a name="nodeprocesscountperapplication"></a>nodeProcessCountPerApplication

此設定會控制每個 IIS 應用程式啟動的節點處理序數目。 預設值為 1。 您可以將此值變更為 0，啟動與您 VM vCPU 計數一樣多的 node.exe。 大部分應用程式的建議值為 0，以便您使用電腦上的所有 vCPU。 Node.exe 是單一執行緒，因此一個 node.exe 最多耗用 1 個 vCPU。 若要發揮節點應用程式的最大效能，建議使用所有 Vcpu。

### <a name="nodeprocesscommandline"></a>nodeProcessCommandLine

此設定會控制 node.exe 的路徑。 您可以設定此值以指向您的 node.exe 版本。

### <a name="maxconcurrentrequestsperprocess"></a>maxConcurrentRequestsPerProcess

此設定會控制 iisnode 傳送給每個 node.exe 的並行要求數目上限。 在 Azure Web Apps 上，預設值為 [無限]。 若不是裝載在 Azure Web Apps 上，則預設值為 1024。 您可以視您應用程式收到的要求數目以及您應用程式處理每個要求的速度而定，來設定此值。

### <a name="maxnamedpipeconnectionretry"></a>maxNamedPipeConnectionRetry

為了將要求傳送至 node.exe，此設定會控制 iisnode 在具名管道上重試連線的次數上限。 此設定結合 namedPipeConnectionRetryDelay 一起使用，可決定 iisnode 內每個要求的總逾時。 Azure Web Apps 上的預設值為 200。 總逾時 (秒) = (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay) / 1000

### <a name="namedpipeconnectionretrydelay"></a>namedPipeConnectionRetryDelay

此設定控制 iisnode 在每次重試透過具名管道將要求傳送到 node.exe 之間所要等待的時間量 (毫秒)。 預設值為 250 毫秒。
總逾時 (秒) = (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay) / 1000

在 Azure Web Apps 上，iisnode 的總逾時預設為 200 \* 250 毫秒 = 50 秒。

### <a name="logdirectory"></a>logDirectory

此設定會控制 iisnode 記錄 stdout/stderr 的目錄。 預設值是 iisnode，相對於主要指令碼目錄 (主要 server.js 所在的目錄)

### <a name="debuggerextensiondll"></a>debuggerExtensionDll

此設定會控制 iisnode 在進行節點應用程式偵錯時，會使用的節點偵測器版本。 iisnode-inspector-0.7.3.dll 和 iisnode-inspector.dll 目前是此設定僅有的兩個有效值。 預設值為 iisnode-inspector-0.7.3.dll。 iisnode-inspector-0.7.3.dll 版本使用 node-inspector-0.7.3，並使用 Web 通訊端。 請在 Azure Webapp 上啟用 Web 通訊端以使用此版本。 如需有關如何設定 iisnode 以使用新節點偵測器的詳細資訊，請參閱 <http://ranjithblogs.azurewebsites.net/?p=98>。

### <a name="flushresponse"></a>flushResponse

IIS 的預設行為是在排清之前或直到回應結束時 (取決於何者較早出現)，將回應資料緩衝至 4 MB。 iisnode 提供組態設定來覆寫這個行為︰若要在 iisnode 從 node.exe 收到回應實體內文的片段時就立刻排清，您需要將 web.config 中的 iisnode/@flushResponse 屬性設定為 'true'︰

```xml
<configuration>
    <system.webServer>
        <!-- ... -->
        <iisnode flushResponse="true" />
    </system.webServer>
</configuration>
```

啟用排清每個回應實體內文的片段會增加效能負擔，使系統的輸送量降低 ~5% (從 v0.1.13 開始)。 最好的方式是只對需要回應串流的端點套用此設定 (例如，使用 web.config 中的 `<location>` 元素)

除此之外，對於串流應用程式，您也必須將 iisnode 處理常式的 responseBufferLimit 設定為 0。

```xml
<handlers>
    <add name="iisnode" path="app.js" verb="\*" modules="iisnode" responseBufferLimit="0"/>
</handlers>
```

### <a name="watchedfiles"></a>watchedFiles

以分號分隔的檔案清單，系統會監看其變更。 任何檔案變更都會導致應用程式回收。 每個項目都包含選擇性目錄名稱，以及相對於主要應用程式進入點所在目錄的必要檔案名稱。 只有檔案名稱部分可以使用萬用字元。 預設值為 `*.js;web.config`

### <a name="recyclesignalenabled"></a>recycleSignalEnabled

預設值為 False。 若已啟用，節點應用程式可以連接至具名管道 (環境變數 IISNODE\_CONTROL\_PIPE) 並傳送「回收」訊息。 這會導致正常回收 w3wp。

### <a name="idlepageouttimeperiod"></a>idlePageOutTimePeriod

預設值為 0，這表示已停用此功能。 若設定為大於 0 的值，iisnode 會每隔 'idlePageOutTimePeriod' 毫秒將其所有的子處理序移出分頁。 若要了解移出分頁的意思，請參閱此[文件](https://msdn.microsoft.com/library/windows/desktop/ms682606.aspx)。 對於耗用大量記憶體和偶爾想要將記憶體移出至磁碟以釋放 RAM 的應用程式，此設定很有用。

> [!WARNING]
> 在生產應用程式上啟用下列組態設定時，請格外小心。 建議不要在實際生產應用程式上啟用這些設定。
>

### <a name="debugheaderenabled"></a>debugHeaderEnabled

預設值為 False。 如果設為 True，iisnode 會將 HTTP 回應標頭 `iisnode-debug` 新增至它所傳送的每個 HTTP 回應，而 `iisnode-debug` 標頭值是 URL。 查看 URL 片段即可取得個別的診斷資訊，但在瀏覽器中開啟 URL 可顯示視覺效果。

### <a name="loggingenabled"></a>loggingEnabled

此設定會控制 iisnode 記錄 stdout 和 stderr 的功能。 Iisnode 會從它所啟動的節點處理序擷取 stdout/stderr，並寫入至 'logDirectory' 設定中指定的目錄。 一旦啟用，您的應用程式會將記錄寫入至檔案系統，而視應用程式所完成的記錄量而定，可能會影響效能。

### <a name="deverrorsenabled"></a>devErrorsEnabled

預設值為 False。 若設為 True，iisnode 會在瀏覽器上顯示 HTTP 狀態碼和 Win32 錯誤碼。 在偵錯特定類型的問題時，Win32 程式碼很有幫助。

### <a name="debuggingenabled-do-not-enable-on-live-production-site"></a>debuggingEnabled (請勿在實際生產網站上啟用)

此設定會控制偵錯功能。 Iisnode 會與節點偵測器整合。 藉由啟用此設定，即可啟用節點應用程式的偵錯功能。 啟用此設定後，iisnode 會在對節點應用程式進行第一個偵錯要求時，在 'debuggerVirtualDir' 目錄中建立節點偵測器檔案。 您可以將要求傳送至 http://yoursite/server.js/debug，以載入節點偵測器。 您可以使用 'debuggerPathSegment' 設定來控制偵錯 URL 區段。 根據預設，debuggerPathSegment=’debug’。 例如您可以將 `debuggerPathSegment` 設定為 GUID，讓其他人更難以發現。

請閱讀[在 Windows 上對 node.js 應用程式進行偵錯](https://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html)，以取得有關偵錯的詳細資料。

## <a name="scenarios-and-recommendationstroubleshooting"></a>案例和建議/疑難排解

### <a name="my-node-application-is-making-excessive-outbound-calls"></a>我的節點應用程式進行太多的輸出呼叫

許多應用程式想要在其定期作業中進行輸出連線。 例如，當要求傳入時，節點應用程式會想連絡別處的 REST API，並取得一些資訊來處理要求。 您想要在進行 http 或 https 呼叫時使用保持連線代理程式。 您可以在進行這些輸出呼叫時，使用 agentkeepalive 模組作為您的保持連線代理程式。

agentkeepalive 模組可確保通訊端會在您的 Azure webapp VM 上重複使用。 在每一個輸出要求上建立新通訊端會增加應用程式的負擔。 讓應用程式重複使用輸出要求的通訊端，可確保您的應用程式不會超過每個 VM 配置的 maxSockets。 對於 Azure Web Apps 的建議是將 agentKeepAlive maxSockets 值設為每個 VM 總計有 160 個通訊端 (4 個 node.exe 執行個體 \* 40 個 maxSockets/執行個體)。

範例 [agentKeepALive](https://www.npmjs.com/package/agentkeepalive) 設定：

```nodejs
var keepaliveAgent = new Agent({
    maxSockets: 40,
    maxFreeSockets: 10,
    timeout: 60000,
    keepAliveTimeout: 300000
});
```

> [!IMPORTANT]
> 這個範例假設您有 4 個 node.exe 在 VM 上執行。 如果您有不同數目的 node.exe 在 VM 上執行，您必須據此修改 maxSockets 設定。
>

#### <a name="my-node-application-is-consuming-too-much-cpu"></a>我的節點應用程式目前耗用太多 CPU

您可能會在您的入口網站上收到 Azure Web Apps 建議的高 CPU 耗用量。 您也可以設定監視器以監看某些[計量](web-sites-monitor.md)。 在 [Azure 入口網站儀表板](../application-insights/app-insights-web-monitor-performance.md)上檢查 CPU 使用量時，請檢查 CPU 的 MAX 值，您才不會錯過尖峰值。
在您認為應用程式耗用太多 CPU，但您無法解釋原因的情況下，您可以剖析節點應用程式以找出原因。

#### <a name="profiling-your-node-application-on-azure-web-apps-with-v8-profiler"></a>在 Azure Web Apps 上使用 V8 分析工具剖析節點應用程式

例如，假設您擁有想要剖析的 hello world 應用程式，如下所示︰

```nodejs
var http = require('http');
function WriteConsoleLog() {
    for(var i=0;i<99999;++i) {
        console.log('hello world');
    }
}

function HandleRequest() {
    WriteConsoleLog();
}

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    HandleRequest();
    res.end('Hello world!');
}).listen(process.env.PORT);
```

移至偵錯主控台網站 https://yoursite.scm.azurewebsites.net/DebugConsole

進入 site/wwwroot 目錄。 您會看到如下列範例所示的命令提示字元：

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_install_v8.png)

執行命令 `npm install v8-profiler`。

此命令會在 node\_modules 目錄與其所有的相依項目之下安裝 v8 分析工具。
現在，編輯 server.js 以剖析您的應用程式。

```nodejs
var http = require('http');
var profiler = require('v8-profiler');
var fs = require('fs');

function WriteConsoleLog() {
    for(var i=0;i<99999;++i) {
        console.log('hello world');
    }
}

function HandleRequest() {
    profiler.startProfiling('HandleRequest');
    WriteConsoleLog();
    fs.writeFileSync('profile.cpuprofile', JSON.stringify(profiler.stopProfiling('HandleRequest')));
}

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    HandleRequest();
    res.end('Hello world!');
}).listen(process.env.PORT);
```

上述程式碼會剖析 WriteConsoleLog 函式，然後將設定檔輸出寫入至您的網站 wwwroot 下的 'profile.cpuprofile' 檔案。 將要求傳送至您的應用程式。 您會在您的網站 wwwroot 下看到建立的 'profile.cpuprofile' 檔案。

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_profile.cpuprofile.png)

下載此檔案，並使用 Chrome F12 工具開啟檔案。 在 Chrome 上按 F12，然後選擇 [設定檔] 索引標籤。選擇 [載入] 按鈕。 選取您下載的 profile.cpuprofile 檔案。 按一下您剛下載的設定檔

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/chrome_tools_view.png)

您會看到 WriteConsoleLog 函式已耗用 95%的時間。 輸出也會顯示造成此問題的確切行號和來源檔案。

### <a name="my-node-application-is-consuming-too-much-memory"></a>我的節點應用程式目前耗用太多記憶體

如果您的應用程式耗用太多記憶體，則會在入口網站上看到來自 Azure Web Apps 的通知，說明關於高記憶體的耗用。 您可以設定監視器以監看某些[計量](web-sites-monitor.md)。 在 [Azure 入口網站儀表板](../application-insights/app-insights-web-monitor-performance.md)上檢查記憶體使用量時，務必檢查記憶體的 MAX 值，才不會錯過尖峰值。

#### <a name="leak-detection-and-heap-diff-for-nodejs"></a>node.js 的流失偵測和堆積區分

您可以使用 [node-memwatch](https://github.com/lloyd/node-memwatch) 協助找出記憶體流失。
如同 v8 分析工具，您可以安裝 `memwatch` 並編輯您的程式碼來擷取和區分堆積，以找出您應用程式中的記憶體流失。

### <a name="my-nodeexes-are-getting-killed-randomly"></a>我的 node.exe 會隨機終止

有幾個原因會造成 node.exe 隨機關閉：

1. 您的應用程式擲回未攔截的例外狀況 - 請檢查 d:\\home\\LogFiles\\Application\\logging-errors.txt file 檔案，以取得所擲回例外狀況的詳細資料。 這個檔案有堆疊追蹤，可協助您對應用程式進行偵錯並加以修正。
2. 您的應用程式耗用太多記憶體，近而讓其他處理序無法開始執行。 如果 VM 記憶體總數接近 100%，則處理序管理員會終止 node.exe。 程序管理員會終止某些處理程序，讓其他處理程序有機會執行一些工作。 若要修正此問題，可分析您應用程式的記憶體流失。 如果您的應用程式需要大量記憶體，請相應增加至較大的 VM (這樣會增加 VM 的可用 RAM)。

### <a name="my-node-application-does-not-start"></a>我的節點應用程式並未啟動

如果您的應用程式在啟動時傳回 500 錯誤，可能有幾個原因︰

1. Node.exe 未出現在正確的位置。 檢查 nodeProcessCommandLine 設定。
2. 主要指令碼檔案未出現在正確的位置。 檢查 web.config，並確定處理常式區段中的主要指令碼檔案名稱符合主要指令碼檔案。
3. Web.config 組態不正確 – 檢查設定名稱/值。
4. 冷啟動 – 您的應用程式花太長時間啟動。 如果您的應用程式所花的時間大於 (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay) / 1000 秒，則 iisnode 會傳回 500 錯誤。 增加這些設定的值，以符合您的應用程式開始時間，可防止 iisnode 逾時並傳回 500 錯誤。

### <a name="my-node-application-crashed"></a>我的節點應用程式毀損

您的應用程式擲回未攔截的例外狀況 - 請檢查 `d:\\home\\LogFiles\\Application\\logging-errors.txt` 檔案，以取得所擲回例外狀況的詳細資料。 這個檔案有堆疊追蹤，可協助您診斷和修正應用程式。

### <a name="my-node-application-takes-too-much-time-to-start-cold-start"></a>我的節點應用程式花太多時間啟動 (冷啟動)

造成應用程式啟動時間過長的常見原因是 node\_modules 中有大量檔案。 應用程式會嘗試在啟動時載入大部分檔案。 根據預設，因為您的檔案會儲存在 Azure Web Apps 的網路共用上，所以載入太多檔案可能需要一些時間。
讓處理程序速度變快的解決方式如下︰

1. 藉由使用 npm3 來安裝您的模組，確定您有扁平相依性結構，而且沒有重複的相依性。
2. 試著延遲載入您的 node\_modules，而不要在應用程式啟動時載入所有的模組。 若要延遲載入模組，應在函數中確實需要模組時，於第一次執行模組程式碼之前執行 require('module') 呼叫。
3. Azure Web Apps 會提供稱為本機快取的功能。 這項功能會將您的內容從網路共用複製到 VM 上的本機磁碟。 因為這些檔案都在本機，node\_modules 的載入時間快很多。

## <a name="iisnode-http-status-and-substatus"></a>IISNODE http 狀態和子狀態

`cnodeconstants` [來源檔案](https://github.com/Azure/iisnode/blob/master/src/iisnode/cnodeconstants.h)會列出 iisnode 在發生錯誤時可傳回的所有可能狀態/子狀態組合。

為您的應用程式啟用 FREB 以查看 win32 錯誤碼 (基於效能考量，務必只在非生產網站上啟用 FREB)。

| Http 狀態 | Http 子狀態 | 可能的原因？ |
| --- | --- | --- |
| 500 |1000 |將要求分派至 IISNODE 時發生一些問題 – 檢查 node.exe 是否已啟動。 Node.exe 可能在啟動時損毀。 檢查 web.config 組態是否有錯誤。 |
| 500 |1001 |- Win32Error 0x2 - 應用程式並未回應 URL。 檢查 URL 重寫規則，或檢查您的快速應用程式是否已定義正確的路由。 - Win32Error 0x6d – 具名管道忙碌中 – Node.exe 不接受要求，因為管道忙碌中。 檢查高 CPU 使用量。 - 其他錯誤 – 檢查 node.exe 是否毀損。 |
| 500 |1002 |Node.exe 損毀 – 檢查 d:\\home\\LogFiles\\logging-errors.txt 中的堆疊追蹤。 |
| 500 |1003 |管道組態問題 – 具名管道的組態不正確。 |
| 500 |1004-1018 |傳送要求或處理 node.exe 的相關回應時發生一些錯誤。 檢查 node.exe 是否損毀。 檢查 d:\\home\\LogFiles\\logging-errors.txt 中的堆疊追蹤。 |
| 503 |1000 |記憶體不足，無法配置更多具名管道連線。 檢查您的應用程式為何使用這麼多的記憶體。 檢查 maxConcurrentRequestsPerProcess 設定值。 如果此值並非無限，而且您有許多要求，請增加此值來避免這個錯誤。 |
| 503 |1001 |無法將要求分派至 node.exe，因為應用程式正在回收處理。 回收應用程式之後，應該會正常提供要求。 |
| 503 |1002 |檢查 win32 錯誤碼的實際原因 – 無法將要求分派至 node.exe。 |
| 503 |1003 |具名管道太忙 – 檢查 node.exe 是否正耗用大量的 CPU |

NODE.exe 具有稱為 `NODE_PENDING_PIPE_INSTANCES` 的設定。 根據預設，如果不是部署在 Azure Web 應用程式，則這個值會是 4。 這表示該 node.exe 在具名管道上一次只能接受四個要求。 在 Azure Web Apps 上，此值設為 5000。 這個值應足以滿足大部分在 Azure Web Apps 上執行的節點應用程式。 您不應該在 Azure Web Apps 上看見 503.1003，因為 `NODE_PENDING_PIPE_INSTANCES` 的值較高

## <a name="more-resources"></a>其他資源

請遵循下列連結以深入了解 Azure App Service 上的 node.js 應用程式。

* [在 Azure App Service 中開始使用 Node.js Web 應用程式](app-service-web-get-started-nodejs.md)
* [如何在 Azure App Service 中偵錯 Node.js Web 應用程式](app-service-web-tutorial-nodejs-mongodb-app.md)
* [使用 Node.js 模組與 Azure 應用程式搭配](../nodejs-use-node-modules-azure-apps.md)
* [Azure App Service Web Apps：Node.js](https://blogs.msdn.microsoft.com/silverlining/2012/06/14/windows-azure-websites-node-js/)
* [Node.js 開發人員中心](../nodejs-use-node-modules-azure-apps.md)
* [探索神秘無比的 Kudu 偵錯主控台](https://azure.microsoft.com/documentation/videos/super-secret-kudu-debug-console-for-azure-web-sites/)