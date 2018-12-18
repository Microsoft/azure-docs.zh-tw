---
title: 如何從適用於 VM 的 Azure 監視器查詢記錄 | Microsoft Docs
description: 適用於 VM 的 Azure 監視器解決方案會將計量和記錄資料轉送到 Log Analytics，本文將說明記錄並提供範例查詢。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: magoedte
ms.openlocfilehash: 446268f28e7c87196023636889f03be2da92ecfd
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967637"
---
# <a name="how-to-query-logs-from-azure-monitor-for-vms"></a>如何從適用於 VM 的 Azure 監視器查詢記錄
適用於 VM 的 Azure 監視器會收集效能和連線計量、電腦和處理序清查資料，以及健康狀態資訊，並將它轉送到 Azure 監視器中的 Log Analytics 資料存放區。  此資料可用來在 Log Analytics 中進行[搜尋](../log-analytics/log-analytics-log-searches.md)。 您可以將此資料套用至各種案例，包括移轉規劃、容量分析、探索和隨選效能疑難排解。

## <a name="map-records"></a>對應記錄
除了當處理序或電腦啟動或是上線到適用於 VM 的 Azure 監視器對應功能時所產生的記錄外，每小時還會為每個唯一的電腦和處理序產生一筆記錄。 這些記錄具有下表中的屬性。 ServiceMapComputer_CL 事件中的欄位和值對應到 ServiceMap Azure Resource Manager API 中的機器資源欄位。 ServiceMapProcess_CL 事件中的欄位和值對應到 ServiceMap Azure Resource Manager API 中的處理序資源欄位。 ResourceName_s 欄位會符合對應 Resource Manager 資源中的名稱欄位。 

有可用來識別唯一處理程序和電腦的內部產生屬性︰

- 電腦：使用 *ResourceId* 或 *ResourceName_s* 來唯一識別 Log Analytics 工作區中的電腦。
- 處理序：使用 *ResourceId* 來唯一識別 Log Analytics 工作區中的處理序。 *ResourceName_s* 在執行處理序的機器 (MachineResourceName_s) 環境中是唯一的 

因為在指定時間範圍內可以有多筆指定處理序和電腦的記錄，針對相同電腦或處理序的查詢可能會傳回多筆記錄。 若只要包含最新的記錄，請在查詢中加入 "| dedup ResourceId"。

### <a name="connections"></a>連線
連線計量會寫入到 Log Analytics 中的新資料表：VMConnection。 這個資料表會提供機器連線 (輸入和輸出) 的相關資訊。 連線計量也會透過 API 來公開，這類 API 會提供方法來取得某個時間範圍內的特定計量。  因為在接聽通訊端上「接受」而產生的 TCP 連線是輸入，因為「連線」到指定 IP 和連接埠而建立的連線則為輸出。 連線的方向會透過 Direction 屬性來表示，此屬性可設為 **inbound** 或 **outbound**。 

這些資料表中的記錄都是從 Dependency Agent 所報告的資料產生的。 每筆記錄均代表在一分鐘時間間隔內的觀測。 TimeGenerated 屬性表示時間間隔的開始時間。 每筆記錄均包含資訊來識別個別的實體 (也就是連線或連接埠)，以及與該實體相關聯的計量。 目前只會報告透過 IPv4 使用 TCP 而發生的網路活動。

為了管理成本和複雜度，連線記錄不代表個別的實體網路連線。 將多個實體網路連線群組為一個邏輯連線，其接著會反映於各自的資料表中。  這表示，*VMConnection* 資料表中的記錄代表一個邏輯群組，而非觀測到的個別實體連線。 在指定的一分鐘時間間隔內，共用下列屬性相同值的實體網路連線會彙總為 *VMConnection* 中的單一邏輯記錄。 

| 屬性 | 說明 |
|:--|:--|
|方向 |連線的方向，值為 *inbound* 或 *outbound* |
|機器 |電腦 FQDN |
|Process |處理序或處理序群組的身分識別，會起始/接受連線 |
|SourceIp |來源的 IP 位址 |
|DestinationIp |目的地的 IP 位址 |
|DestinationPort |目的地的連接埠號碼 |
|通訊協定 |用於連線的通訊協定。  值為 *tcp*。 |

為了說明群組的影響，會在記錄的下列屬性中提供群組實體連線數目的相關資訊：

| 屬性 | 說明 |
|:--|:--|
|LinksEstablished |已在報告時間範圍內建立的實體網路連線數目 |
|LinksTerminated |已在報告時間範圍內終止的實體網路連線數目 |
|LinksFailed |在報告時間範圍內失敗的實體網路連線數目。 此資訊目前僅適用於輸出連線。 |
|LinksLive |已在報告時間範圍結束時開啟的實體網路連線數目|

#### <a name="metrics"></a>度量

除了連線計數計量，在指定邏輯連線或網路連接埠上傳送與接收的資料量相關資訊也會包含於記錄的下列屬性中：

| 屬性 | 說明 |
|:--|:--|
|BytesSent |已在報告時間範圍內傳送的位元組總數 |
|BytesReceived |已在報告時間範圍內接收的位元組總數 |
|回應 |已在報告時間範圍內觀測到的回應數目。 
|ResponseTimeMax |已在報告時間範圍內觀測到的最大回應時間 (毫秒)。  如果沒有值，則此屬性為空白。|
|ResponseTimeMin |已在報告時間範圍內觀測到的最小回應時間 (毫秒)。  如果沒有值，則此屬性為空白。|
|ResponseTimeSum |已在報告時間範圍內觀測到的所有回應時間總和 (毫秒)。  如果沒有值，則此屬性為空白|

所報告的第三個資料類型是回應時間：呼叫端需要花費多久時間來等候透過連線傳送的要求，此要求會由遠端端點來處理及回應。 所報告的回應時間是基礎應用程式通訊協定的真正回應時間估計值。 它會使用啟發學習法，根據實體網路連線來源和目的端之間資料流程的觀測計算而來的。 概念上，它是要求的最後一個位元組離開傳送端的時間，以及回應的最後一個位元組傳回給它的到達時間之間的差異。 這兩個時間戳記可用來描述指定實體連線上的要求和回應事件。 它們之間的差異代表單一要求的回應時間。 

在此功能的第一個版本中，我們的演算法是一個近似值，根據針對指定網路連線所使用的實際應用程式通訊協定，成功的程度可能各有不同。 例如，目前的方法非常適用以要求-回應為基礎的通訊協定 (例如 HTTP (S))，但不適用單向或以訊息佇列為基礎的通訊協定。

此處有一些需要考慮的重要事項：

1. 如果處理序會在同一個 IP 位址但透過多個網路介面來接受連線，則將會報告每個介面的個別記錄。 
2. 使用萬用字元 IP 的記錄將不會包含任何活動。 它們會包含在其中，以代表會針對輸入流量開啟電腦上的連接埠。
3. 若要減少詳細資訊和資料量，如果有具特定 IP 位址的相符記錄 (適用於相同的處理序、連接埠和通訊協定)，將略過具有萬用字元 IP 的記錄。 略過萬用字元 IP 記錄時，具有特定 IP 位址的 IsWildcardBind 記錄屬性將設為 "True"，以代表連接埠會透過報告機器的每個介面來公開。
4. 只在特定介面上繫結的連接埠會將 IsWildcardBind 設為 "False"。

#### <a name="naming-and-classification"></a>命名和分類
為了方便起見，RemoteIp 屬性中會包含連線遠端的 IP 位址。 如果是輸入連線，RemoteIp 相當於 SourceIp，如果是連出連線，則它相當於 DestinationIp。 RemoteDnsCanonicalNames 屬性代表適用於 RemoteIp 的機器所報告的 DNS 標準名稱。 RemoteDnsQuestions 和 RemoteClassification 屬性均會保留，以供日後使用。 

#### <a name="geolocation"></a>地理位置
*VMConnection* 也會在記錄的下列屬性中，包含每個連線記錄遠端的地理位置資訊： 

| 屬性 | 說明 |
|:--|:--|
|RemoteCountry |裝載 RemoteIp 的國家/地區名稱。  例如，*United States* |
|RemoteLatitude |地理位置緯度。  例如，*47.68* |
|RemoteLongitude |地理位置經度。  例如：*-122.12* |

#### <a name="malicious-ip"></a>惡意 IP
*VMConnection* 資料表中的每個 RemoteIp 屬性均會根據一組具有已知惡意活動的 IP 進行檢查。 如果 RemoteIp 被識別為惡意的，將在記錄的下列屬性中填入下列屬性 (如果 IP 被視為不是惡意的，則它們是空的)：

| 屬性 | 說明 |
|:--|:--|
|MaliciousIP |RemoteIp 位址 |
|IndicatorThreadType | |
|說明 | |
|TLPLevel | |
|信賴度 | |
|嚴重性 | |
|FirstReportedDateTime | |
|LastReportedDateTime | |
|IsActive | |
|ReportReferenceLink | |
|AdditionalInformation | |

### <a name="servicemapcomputercl-records"></a>ServiceMapComputer_CL 記錄
類型為 *ServiceMapComputer_CL* 的記錄會有伺服器 (具有 Dependency Agent) 的清查資料。 這些記錄具有下表中的屬性：

| 屬性 | 說明 |
|:--|:--|
| 類型 | *ServiceMapComputer_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | 工作區中機器的唯一識別碼 |
| ResourceName_s | 工作區中機器的唯一識別碼 |
| ComputerName_s | 電腦 FQDN |
| Ipv4Addresses_s | 伺服器的 IPv4 位址清單 |
| Ipv6Addresses_s | 伺服器的 IPv6 位址清單 |
| DnsNames_s | DNS 名稱的陣列 |
| OperatingSystemFamily_s | Windows 或 Linux |
| OperatingSystemFullName_s | 作業系統的完整名稱  |
| Bitness_s | 機器的運算位元數 (32 位元或 64 位元)  |
| PhysicalMemory_d | 實體記憶體 (MB) |
| Cpus_d | CPU 數目 |
| CpuSpeed_d | CPU 速度 (MHz)|
| VirtualizationState_s | *unknown**physical**virtual* *hypervisor* |
| VirtualMachineType_s | *hyperv*、*vmware* 等等 |
| VirtualMachineNativeMachineId_g | VM 識別碼 (由其 Hypervisor 指派) |
| VirtualMachineName_s | VM 的名稱 |
| BootTime_t | 開機時間 |

### <a name="servicemapprocesscl-type-records"></a>ServiceMapProcess_CL 類型記錄
類型為 *ServiceMapProcess_CL* 的記錄會有伺服器 (具有 Dependency Agent) 上 TCP 連線處理序的清查資料。 這些記錄具有下表中的屬性：

| 屬性 | 說明 |
|:--|:--|
| 類型 | *ServiceMapProcess_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | 工作區中處理序的唯一識別碼 |
| ResourceName_s | 在執行處理序的機器上，處理序的唯一識別碼|
| MachineResourceName_s | 機器的資源名稱 |
| ExecutableName_s | 處理序可執行檔的名稱 |
| StartTime_t | 處理序集區的開始時間 |
| FirstPid_d | 處理序集區中的第一個 PID |
| Description_s | 處理序的描述 |
| CompanyName_s | 公司的名稱 |
| InternalName_s | 內部名稱 |
| ProductName_s | 產品的名稱 |
| ProductVersion_s | 產品版本 |
| FileVersion_s | 檔案版本 |
| CommandLine_s | 命令列 |
| ExecutablePath _s | 可執行檔的路徑 |
| WorkingDirectory_s | 工作目錄 |
| UserName | 執行處理序的帳戶 |
| UserDomain | 執行處理序的網域 |

## <a name="sample-log-searches"></a>記錄搜尋範例

### <a name="list-all-known-machines"></a>列出所有已知的機器
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>列出所有受控電腦的實體記憶體容量。
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project PhysicalMemory_d, ComputerName_s

### <a name="list-computer-name-dns-ip-and-os"></a>列出電腦名稱、DNS、IP 和 OS。
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project ComputerName_s, OperatingSystemFullName_s, DnsNames_s, Ipv4Addresses_s

### <a name="find-all-processes-with-sql-in-the-command-line"></a>在命令列中尋找具有「sql」的所有處理程序
ServiceMapProcess_CL | where CommandLine_s contains_cs "sql" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>以資源名稱尋找機器 (最新的記錄)
search in (ServiceMapComputer_CL) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>以 IP 位址尋找機器 (最新的記錄)
search in (ServiceMapComputer_CL) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-all-known-processes-on-a-specified-machine"></a>列出指定機器上的所有已知處理序
ServiceMapProcess_CL | where MachineResourceName_s == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-all-computers-running-sql"></a>列出所有執行 SQL 的電腦
ServiceMapComputer_CL | where ResourceName_s in ((search in (ServiceMapProcess_CL) "\*sql\*" | distinct MachineResourceName_s)) | distinct ComputerName_s

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>列出資料中心內所有唯一 curl 產品版本
ServiceMapProcess_CL | where ExecutableName_s == "curl" | distinct ProductVersion_s

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>為所有執行 CentOS 的電腦建立電腦群組
ServiceMapComputer_CL | where OperatingSystemFullName_s contains_cs "CentOS" | distinct ComputerName_s

### <a name="summarize-the-outbound-connections-from-a-group-of-machines"></a>摘要說明來自機器群組的輸出連線
```
// the machines of interest
let machines = datatable(m: string) ["m-82412a7a-6a32-45a9-a8d6-538354224a25"];
// map of ip to monitored machine in the environment
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
// all connections to/from the machines of interest
let out=materialize(VMConnection
| where Machine in (machines)
| summarize arg_max(TimeGenerated, *) by ConnectionId);
// connections to localhost augmented with RemoteMachine
let local=out
| where RemoteIp startswith "127."
| project ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=Machine;
// connections not to localhost augmented with RemoteMachine
let remote=materialize(out
| where RemoteIp !startswith "127."
| join kind=leftouter (ips) on $left.RemoteIp == $right.ips
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=MonitoredMachine);
// the remote machines to/from which we have connections
let remoteMachines = remote | summarize by RemoteMachine;
// all augmented connections
(local)
| union (remote)
//Take all outbound records but only inbound records that come from either //unmonitored machines or monitored machines not in the set for which we are computing dependencies.
| where Direction == 'outbound' or (Direction == 'inbound' and RemoteMachine !in (machines))
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine
// identify the remote port
| extend RemotePort=iff(Direction == 'outbound', DestinationPort, 0)
// construct the join key we'll use to find a matching port
| extend JoinKey=strcat_delim(':', RemoteMachine, RemoteIp, RemotePort, Protocol)
// find a matching port
| join kind=leftouter (VMBoundPort 
| where Machine in (remoteMachines) 
| summarize arg_max(TimeGenerated, *) by PortId 
| extend JoinKey=strcat_delim(':', Machine, Ip, Port, Protocol)) on JoinKey
// aggregate the remote information
| summarize Remote=makeset(iff(isempty(RemoteMachine), todynamic('{}'), pack('Machine', RemoteMachine, 'Process', Process1, 'ProcessName', ProcessName1))) by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol
```

## <a name="next-steps"></a>後續步驟
* 如果您不熟悉如何在 Log Analytics 中撰寫查詢，請檢閱 Azure 入口網站中的[如何使用 Log Analytics 頁面](../log-analytics/query-language/get-started-analytics-portal.md)以撰寫 Log Analytics 查詢。
* 了解如何[撰寫搜尋查詢](../log-analytics/query-language/search-queries.md)。