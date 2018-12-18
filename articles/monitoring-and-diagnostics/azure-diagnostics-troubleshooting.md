---
title: 針對 Azure 診斷擴充功能疑難排解
description: 針對在 Azure 虛擬機器、Service Fabric 或 雲端服務中使用 Azure 診斷時出現的問題進行疑難排解。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 07/12/2017
ms.author: robb
ms.component: diagnostic-extension
ms.openlocfilehash: 6ea68bce81094f4745616e32c7434d6c833a45ee
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952574"
---
# <a name="azure-diagnostics-troubleshooting"></a>Azure 診斷疑難排解
本文說明有關使用 Azure 診斷的疑難排解資訊。 如需有關 Azure 診斷的詳細資訊，請參閱 [Azure 診斷概觀](azure-diagnostics.md)。

## <a name="logical-components"></a>邏輯元件
**診斷外掛程式啟動器 (DiagnosticsPluginLauncher.exe)**：啟動 Azure 診斷擴充功能。 可作為進入點程序。

**診斷外掛程式 (DiagnosticsPlugin.exe)**：設定、啟動及管理監視代理程式的存留期。 它是由啟動程式啟動的主要處理序。

**監視代理程式 (MonAgent\*.exe 處理序)**：監視、收集及傳輸診斷資料。  

## <a name="logartifact-paths"></a>記錄檔/構件路徑
以下是一些重要記錄檔和構件的路徑。 稍後在本文中我們會參考這些資訊。

### <a name="azure-cloud-services"></a>Azure 雲端服務
| 構件 | Path |
| --- | --- |
| **Azure 診斷組態檔** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\Config.txt |
| **記錄檔** | C:\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\ |
| **診斷資料的本機存放區** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Tables |
| **監視代理程式組態檔** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MaConfig.xml |
| **Azure 診斷擴充功能套件** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version> |
| **記錄集合公用程式路徑** | %SystemDrive%\Packages\GuestAgent\ |
| **MonAgentHost 記錄檔** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

### <a name="virtual-machines"></a>虛擬機器
| 構件 | Path |
| --- | --- |
| **Azure 診斷組態檔** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\RuntimeSettings |
| **記錄檔** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\ |
| **診斷資料的本機存放區** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Tables |
| **監視代理程式組態檔** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MaConfig.xml |
| **狀態檔案** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\Status |
| **Azure 診斷擴充功能套件** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>|
| **記錄集合公用程式路徑** | C:\WindowsAzure\Packages |
| **MonAgentHost 記錄檔** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

## <a name="metric-data-doesnt-appear-in-the-azure-portal"></a>計量資料沒有出現在 Azure 入口網站中
Azure 診斷會提供計量資料，這些資料可以在 Azure 入口網站中顯示。 如果您在入口網站中看不到資料，請檢查 Azure 診斷儲存體帳戶中的 WADMetrics\* 表格，以查看是否有對應的計量記錄。

此處的表格 **PartitionKey** 是資源識別碼、虛擬機器或虛擬機器擴展集。 **RowKey** 是度量的名稱 (也稱為效能計數器名稱)。

如果資源識別碼不正確，請檢查 [診斷組態] > [計量] > [資源識別碼]，以查看資源識別碼是否正確地設定。

如果沒有特定計量的資料，請檢查 [診斷組態] > [PerformanceCounter]，以查看是否包含計量 (效能計數器)。 我們預設會啟用下列計數器：
- \Processor(_Total)\% Processor Time
- \Memory\Available Bytes
- \ASP.NET Applications(__Total__)\Requests/Sec
- \ASP.NET Applications(__Total__)\Errors Total/Sec
- \ASP.NET\Requests Queued
- \ASP.NET\Requests Rejected
- \Processor(w3wp)\% Processor Time
- \Process(w3wp)\Private Bytes
- \Process(WaIISHost)\% Processor Time
- \Process(WaIISHost)\Private Bytes
- \Process(WaWorkerHost)\% Processor Time
- \Process(WaWorkerHost)\Private Bytes
- \Memory\Page Faults/sec
- \.NET CLR Memory(_Global_)\% Time in GC
- \LogicalDisk(C:)\Disk Write Bytes/sec
- \LogicalDisk(C:)\Disk Read Bytes/sec
- \LogicalDisk(D:)\Disk Write Bytes/sec
- \LogicalDisk(D:)\Disk Read Bytes/sec

如果設定都正確，但您仍然沒有看到計量資料，請使用下列指導方針來進行疑難排解。


## <a name="azure-diagnostics-isnt-starting"></a>Azure 診斷未啟動
如需 Azure 診斷為何無法啟動的相關詳細資訊，請參閱稍早所提供記錄檔位置中的 **DiagnosticsPluginLauncher.log** 和 **DiagnosticsPlugin.log** 檔案。

如果這些記錄指出 `Monitoring Agent not reporting success after launch`，則表示啟動 MonAgentHost.exe 失敗。 在前一節針對 `MonAgentHost log file` 所指示的位置中，查看這些記錄檔。

記錄檔的最後一行包含結束代碼。  

```
DiagnosticsPluginLauncher.exe Information: 0 : [4/16/2016 6:24:15 AM] DiagnosticPlugin exited with code 0
```
如果結束代碼為「負值」，請參閱[參考](#references)一節中的[結束代碼表格](#azure-diagnostics-plugin-exit-codes)。

## <a name="diagnostics-data-is-not-logged-to-azure-storage"></a>診斷資料未記錄至 Azure 儲存體
判斷是沒有出現任何資料，或是出現部分的資料。

### <a name="diagnostics-infrastructure-logs"></a>診斷基礎結構記錄檔
診斷會將所有錯誤記錄在診斷基礎結構記錄檔中。 請確定您已[在設定中啟用擷取診斷基礎結構記錄檔](#how-to-check-diagnostics-extension-configuration)。 之後，就可以在設定的儲存體帳戶中，快速尋找 `DiagnosticInfrastructureLogsTable` 表格中出現的任何相關錯誤。

### <a name="no-data-is-appearing"></a>沒有出現任何資料
如果完全沒有出現事件資料，最常見的原因便是未正確定義儲存體帳戶資訊。

解決方法：更正診斷組態，並重新安裝診斷。

如果已正確地設定儲存體帳戶，請從遠端存取該電腦，並確定 DiagnosticsPlugin.exe 和 MonAgentCore.exe 正在執行中。 如果這兩個執行檔並未執行，請依照 [Azure 診斷未啟動](#azure-diagnostics-is-not-starting)中的步驟操作。

如果處理序正在執行，請移至[是否正在本機擷取資料](#is-data-getting-captured-locally)，並依照該處的指示操作。

### <a name="part-of-the-data-is-missing"></a>遺漏部分資料
如果您取得部分資料，但不是全部資料，則表示資料收集/傳輸管線的設定正確。 請依照此處子小節的說明，以縮小問題範圍。

#### <a name="is-the-collection-configured"></a>是否已設定收集？
診斷組態會針對要收集的特定資料類型，包含相關的指示。 [檢閱您的設定](#how-to-check-diagnostics-extension-configuration)以確認您只尋找設定要收集的資料。

#### <a name="is-the-host-generating-data"></a>主機是否正在產生資料？
- **效能計數器**：開啟 Perfmon 並檢查計數器。

- **追蹤記錄檔**：從遠端存取 VM，並將 TextWriterTraceListener 新增至應用程式的組態檔。  請參閱 http://msdn.microsoft.com/library/sk36c28t.aspx 設定文字接聽程式。  確定 `<trace>` 元素具有 `<trace autoflush="true">`。<br />
如果未看見正在產生追蹤記錄檔，請參閱[關於遺漏追蹤記錄檔的詳細資訊](#more-about-trace-logs-missing)。

- **ETW 追蹤**：從遠端存取 VM 並安裝 PerfView。  在 PerfView 中執行 [File] \(檔案\) > [User Command] \(使用者命令\) > [Listen etwprovder1] \(接聽 etwprovder1\) > [etwprovider2]，依此類推。 **Listen** 命令會區分大小寫，而且在以逗號區隔的 ETW 提供者清單之間不能有空格。 如果命令執行失敗，您可以選取 Perfview 工具右下方的 [Log] \(記錄\) 按鈕，即可查看已嘗試執行的動作與執行結果。  如果輸入正確，就會跳出新的視窗。 在幾秒鐘內，就會開始看到 ETW 追蹤。

- **事件記錄檔**：從遠端存取 VM。 開啟 `Event Viewer`，然後確認事件存在。

#### <a name="is-data-getting-captured-locally"></a>是否正在本機擷取資料？
接下來，確定正在本機擷取資料。
資料儲存在[診斷資料的本機存放區](#log-artifacts-path)的本機 `*.tsf` 檔案中。 不同的 `.tsf` 檔案會收集不同類型的記錄。 名稱類似於 Azure 儲存體中的表格名稱。

例如，`Performance Counters` 收集在 `PerformanceCountersTable.tsf` 中。 事件記錄檔收集在 `WindowsEventLogsTable.tsf` 中。 請使用[本機記錄檔擷取](#local-log-extraction)一節中的指示，開啟本機收集檔案並確認這些檔案收集於磁碟上。

如果沒有看到本機收集的記錄，且已經確認主機正在產生資料，則可能是設定有問題。 請仔細檢閱您的設定。

同時也請檢閱針對 MonitoringAgent [MaConfig.xml](#log-artifacts-path) 所產生的組態。 確認有描述相關記錄來源的區段。 然後，確認它在診斷組態和監視代理程式組態之間正確地轉譯。

#### <a name="is-data-getting-transferred"></a>是否正在傳輸資料？
如果您已經確認正在本機擷取資料，但仍然沒有在儲存體帳戶中看到資料，請依照下列步驟執行：

- 確定您已經提供正確的儲存體帳戶，且未變換所提供儲存體帳戶的金鑰。 針對 Azure 雲端服務，我們發現使用者有時候並未更新 `useDevelopmentStorage=true`。

- 確認提供正確的儲存體帳戶。 請確認您的網路並未限制元件連線到公用儲存體端點。 確認的方法之一，便是從遠端存取電腦，然後嘗試寫入到相同的儲存體帳戶。

- 最後，您可以查看監視代理程式所報告的失敗項目。 監視代理程式會將其記錄檔寫入位於[診斷資料的本機存放區](#log-artifacts-path)的 `maeventtable.tsf` 中。 依照[本機記錄檔擷取](#local-log-extraction)一節中的指示來開啟這個檔案。 然後嘗試判斷，是否有指示讀取本機檔案寫入儲存體失敗的 `errors`。

### <a name="capturing-and-archiving-logs"></a>擷取和封存記錄檔
如果您想要連絡支援人員，支援人員首先可能會要求您收集電腦的記錄檔。 您可以先自行執行該作業以節省時間。 執行位在[記錄收集公用程式路徑](#log-artifacts-path)的 `CollectGuestLogs.exe`。 它會產生一個 .zip 檔案，其中所有相關 Azure 記錄檔都放在相同的資料夾中。

## <a name="diagnostics-data-tables-not-found"></a>找不到診斷資料表格
Azure 儲存體中保存 ETW 事件的表格使用以下程式碼來命名：

```csharp
        if (String.IsNullOrEmpty(eventDestination)) {
            if (e == "DefaultEvents")
                tableName = "WADDefault" + MD5(provider);
            else
                tableName = "WADEvent" + MD5(provider) + eventId;
        }
        else
            tableName = "WAD" + eventDestination;
```

下列是一個範例：

```XML
        <EtwEventSourceProviderConfiguration provider="prov1">
          <Event id="1" />
          <Event id="2" eventDestination="dest1" />
          <DefaultEvents />
        </EtwEventSourceProviderConfiguration>
        <EtwEventSourceProviderConfiguration provider="prov2">
          <DefaultEvents eventDestination="dest2" />
        </EtwEventSourceProviderConfiguration>
```
```JSON
"EtwEventSourceProviderConfiguration": [
    {
        "provider": "prov1",
        "Event": [
            {
                "id": 1
            },
            {
                "id": 2,
                "eventDestination": "dest1"
            }
        ],
        "DefaultEvents": {
            "eventDestination": "DefaultEventDestination",
            "sinks": ""
        }
    },
    {
        "provider": "prov2",
        "DefaultEvents": {
            "eventDestination": "dest2"
        }
    }
]
```
此程式碼會產生四個表格：

| Event | 資料表名稱 |
| --- | --- |
| provider=”prov1” &lt;Event id=”1” /&gt; |WADEvent+MD5(“prov1”)+”1” |
| provider=”prov1” &lt;Event id=”2” eventDestination=”dest1” /&gt; |WADdest1 |
| provider=”prov1” &lt;DefaultEvents /&gt; |WADDefault+MD5(“prov1”) |
| provider=”prov2” &lt;DefaultEvents eventDestination=”dest2” /&gt; |WADdest2 |

## <a name="references"></a>參考

### <a name="how-to-check-diagnostics-extension-configuration"></a>如何檢查診斷擴充功能組態
若要檢查擴充功能組態，最容易的方式就是移至 [Azure 資源總管](http://resources.azure.com)，然後移至 Azure 診斷擴充功能 (IaaSDiagnostics / PaaDiagnostics) 所在的虛擬機器或雲端服務。

或者，使用遠端桌面連線到電腦，然後查看[記錄檔構件路徑](#log-artifacts-path)一節中所描述的 Azure 診斷組態檔。

不論使用何種方式，都可以搜尋 **Microsoft.Azure.Diagnostics**，然後搜尋 **xmlCfg** 或 **WadCfg** 欄位。

如果是在虛擬機器上搜尋，且出現 **WadCfg** 欄位，表示組態為 JSON 格式。 如果出現 **xmlCfg** 欄位，表示組態為 XML 格式且以 base64 編碼。 您必須將它[解碼](http://www.bing.com/search?q=base64+decoder)，才能查看診斷載入的 XML。

針對雲端服務角色，如果是從磁碟挑選組態，則資料會以 base64 編碼，因此需要[將資料解碼](http://www.bing.com/search?q=base64+decoder)，才能查看診斷所載入的 XML。

### <a name="azure-diagnostics-plugin-exit-codes"></a>Azure 診斷外掛程式結束代碼
外掛程式會傳回下列結束代碼：

| 結束代碼 | 說明 |
| --- | --- |
| 0 |成功。 |
| -1 |一般錯誤。 |
| -2 |無法載入 rcf 檔。<p>只有在 VM 上不正確地手動叫用客體代理程式外掛程式啟動器時，才會發生此內部錯誤。 |
| -3 |無法載入診斷組態檔。<p><p>解決方法：因組態檔未通過結構描述驗證而造成。 解決方法是提供以結構描述編譯的組態檔。 |
| -4 |診斷之監視代理程式的另一個執行個體已在使用本機資源目錄。<p><p>解決方法：為 **LocalResourceDirectory** 指定其他值。 |
| -6 |客體代理程式外掛程式啟動器嘗試使用無效的命令列來啟動診斷。<p><p>只有在 VM 上不正確地手動叫用客體代理程式外掛程式啟動器時，才會發生此內部錯誤。 |
| -10 |診斷外掛程式結束並發生未處理的例外狀況。 |
| -11 |客體代理程式無法建立負責啟動及監視監視代理程式的處理序。<p><p>解決方法：確認有足夠的系統資源，可啟動新的處理序。<p> |
| -101 |呼叫診斷外掛程式時的引數無效。<p><p>只有在 VM 上不正確地手動叫用客體代理程式外掛程式啟動器時，才會發生此內部錯誤。 |
| -102 |外掛程式處理序無法對本身進行初始化。<p><p>解決方法：確認有足夠的系統資源，可啟動新的處理序。 |
| -103 |外掛程式處理序無法對本身進行初始化。 具體而言，即是無法建立記錄器物件。<p><p>解決方法：確認有足夠的系統資源，可啟動新的處理序。 |
| -104 |無法載入客體代理程式提供的 rcf 檔。<p><p>只有在 VM 上不正確地手動叫用客體代理程式外掛程式啟動器時，才會發生此內部錯誤。 |
| -105 |診斷外掛程式無法開啟診斷組態檔。<p><p>只有在 VM 上不正確地手動叫用診斷外掛程式時，才會發生此內部錯誤。 |
| -106 |無法讀取診斷組態檔。<p><p>原因是組態檔未通過結構描述驗證。 <br><br>解決方法：提供符合結構描述的組態檔。 如需詳細資訊，請參閱[如何檢查診斷擴充功能組態](#how-to-check-diagnostics-extension-configuration)。 |
| -107 |傳遞給監視代理程式的資源目錄無效。<p><p>只有在 VM 上不正確地手動叫用監視代理程式時，才會發生此內部錯誤。</p> |
| -108 |無法將診斷組態檔轉換成監視代理程式組態檔。<p><p>此內部錯誤應只有當使用無效的組態檔以手動方式叫用診斷外掛程式時才會發生。 |
| -110 |一般診斷組態錯誤。<p><p>此內部錯誤應只有當使用無效的組態檔以手動方式叫用診斷外掛程式時才會發生。 |
| -111 |無法啟動監視代理程式。<p><p>解決方法：確認有足夠的系統資源可用。 |
| -112 |一般錯誤 |

### <a name="local-log-extraction"></a>本機記錄檔擷取
監視代理程式會以 `.tsf` 檔案的方式收集記錄和構件。 您無法讀取 `.tsf` 檔案，但可以如下所示將它轉換成 `.csv`：

```
<Azure diagnostics extension package>\Monitor\x64\table2csv.exe <relevantLogFile>.tsf
```
在對應的 `.tsf` 檔案所在的相同路徑中，會建立一個名為 `<relevantLogFile>.csv` 的新檔案。

>[!NOTE]
> 您只需要針對主要的 tsf 檔案 (例如 PerformanceCountersTable.tsf) 來執行此公用程式。 系統會自動處理隨附的檔案 (例如，PerformanceCountersTables_\*\*001.tsf、PerformanceCountersTables_\*\*002.tsf 等等)。

### <a name="more-about-missing-trace-logs"></a>關於遺漏追蹤記錄檔的詳細資訊

>[!NOTE]
> 除非在 IaaS VM 上執行的應用程式已經設定 DiagnosticsMonitorTraceListener，否則以下資訊大部分適用於 Azure 雲端服務。

- 請確定已在 web.config 或 app.config 中設定 **DiagnosticMonitorTraceListener**。這是雲端服務專案的預設設定。 不過，有些客戶會使它成為註解，造成診斷不會收集追蹤陳述式。

- 如果未從 **OnStart** 或 **Run** 方法寫入記錄檔，請確定 app.config 中有 **DiagnosticMonitorTraceListener**。它預設是位在 web.config 中，但此預設值僅適用於 w3wp.exe 內執行的程式碼。 因此您需要將它放在 app.config 中，以擷取在 WaIISHost.exe 中執行的追蹤。

- 請確定您使用的是 **Diagnostics.Trace.TraceXXX**，而不是 **Diagnostics.Debug.WriteXXX**。 偵錯陳述式已從發行組建中移除。

- 請確定已編譯的程式碼確實具有 **Diagnostics.Trace** 行 (使用 Reflector、ildasm 或 ILSpy 來驗證)。 除非使用 TRACE 條件式編譯符號，否則 **Diagnostics.Trace** 命令已從編譯的二進位檔中移除。 當您使用 msbuild 來建置專案時，這是常見的問題。   

## <a name="known-issues-and-mitigations"></a>已知問題與緩解方式
以下是已知問題和緩解方式的清單：

**1. .NET 4.5 相依性**

Windows Azure 診斷擴充功能對於 .NET 4.5 Framework 或更新版本有執行階段相依性。 在本文撰寫之際，針對 Azure 雲端服務佈建的所有電腦，以及以 Azure 虛擬機器為基礎的所有官方映像，都已經安裝 .NET 4.5 或更新版本。

當您嘗試執行 Windows Azure 診斷擴充功能時，仍然可能會遇到電腦沒有 .NET 4.5 或更新版本的情況。 當您從舊映像或快照集建立電腦，或帶入自己的自訂磁碟時，就可能會發生這個問題。

執行 **DiagnosticsPluginLauncher.exe** 時，這通常會以結束代碼 **255** 來表示。 失敗原因為下列未處理的例外狀況：
```
System.IO.FileLoadException: Could not load file or assembly 'System.Threading.Tasks, Version=1.5.11.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies
```

**緩解方式：** 在電腦上安裝 .NET 4.5 或更新版本。

**2.儲存體中有效能計數器資料，但入口網站中沒有顯示**

虛擬機器中的入口網站體驗預設會顯示特定效能計數器。 如果您沒有看到效能計數器，但知道系統正在產生資料，因為儲存體中有資料，請檢查下列項目：

- 儲存體中的資料是否具有英文計數器名稱。 如果計數器名稱不是英文，入口網站計量圖表將無法辨識該計數器。 **緩解方式**：針對系統帳戶，將電腦的語言變更為英文。 若要執行上述方式，請選取 [控制台] > [地區] > [系統管理] > [複製設定]。 接下來，取消選取 [歡迎畫面及系統帳戶]，如此一來自訂語言就不會套用到系統帳戶。

- 如果您在效能計數器名稱中使用萬用字元 (\*)，在將效能計數器傳送至 Azure 儲存體接收時，入口網站將無法關聯設定和收集的計數器。 **風險降低**：若要確定您可以使用萬用字元，並讓入口網站展開 (\*)，請將效能計數器傳送至[「Azure 監視器」接收](azure-diagnostics-schema.md#diagnostics-extension-111)。
