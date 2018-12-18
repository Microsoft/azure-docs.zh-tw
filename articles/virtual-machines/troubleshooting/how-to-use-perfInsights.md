---
title: 如何在 Microsoft Azure 中使用 PerfInsights | Microsoft Docs
description: 了解如何使用 PerfInsights 針對 Windows VM 效能問題進行疑難排解。
services: virtual-machines-windows'
documentationcenter: ''
author: anandhms
manager: cshepard
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: a6077b3e10562b4aee313cccd86256b8f00084ec
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411853"
---
# <a name="how-to-use-perfinsights"></a>如何使用 PerfInsights

[PerfInsights](http://aka.ms/perfinsightsdownload) 是一種自助診斷工具，此工具會收集和分析診斷資料，並提供報表來協助進行 Azure 中 Windows 虛擬機器效能問題的疑難排解。 您可以在虛擬機器上將 PerfInsights 當作獨立工具執行、使用 [Azure 虛擬機器的效能診斷](performance-diagnostics.md)直接從入口網站執行，或藉由安裝 [Azure 效能診斷 VM 擴充功能](performance-diagnostics-vm-extension.md)來執行。

如果您遇到虛擬機器的效能問題，在連絡支援人員之前，請先執行這個工具。

## <a name="supported-troubleshooting-scenarios"></a>支援的疑難排解案例

PerfInsights 可以收集並分析多種資訊。 下列幾節會說明常見案例。

### <a name="quick-performance-analysis"></a>快速效能分析

此案例會收集磁碟設定和其他重要資訊，包括：

-   事件記錄檔

-   所有傳入和傳出連線的網路狀態

-   網路和防火牆組態設定

-   目前正在系統上執行之所有應用程式的工作清單

-   Microsoft SQL Server 資料庫組態設定 (如果 VM 被識別為執行 SQL Server 的伺服器)

-   儲存體可靠性計數器

-   重要的 Windows Hotfix

-   已安裝的篩選器驅動程式

這是被動的資訊集合，應該不會影響系統。 

>[!Note]
>下列每一個案例都會自動包含此案例：

### <a name="benchmarking"></a>效能評定

此案例會為連接至 VM 的所有磁碟機執行 [Diskspd](https://github.com/Microsoft/diskspd) 基準測試 (IOPS 和 MBPS)。 

> [!Note]
> 此案例會影響系統，所以不應該在實際的生產系統上執行。 如有必要，請在專用的維護視窗中執行此案例，以免發生任何問題。 因追蹤或基準測試所增加的工作負載，可能會對 VM 的效能造成負面影響。
>

### <a name="performance-analysis"></a>效能分析

此案例會使用於 RuleEngineConfig.json 檔案中指定的計數器來執行[效能計數器](https://msdn.microsoft.com/library/windows/desktop/aa373083(v=vs.85).aspx) \(英文\) 追蹤。 如果將 VM 識別為執行 SQL Server 的伺服器，就會執行效能計數器追蹤。 它會使用可在 RuleEngineConfig.json 檔案中找到的計數器來執行此動作。 此案例也會包含效能診斷資料。

### <a name="azure-files-analysis"></a>Azure 檔案分析

此案例會同時執行特殊的效能計數器擷取與網路追蹤。 擷取包含所有伺服器訊息區塊 (SMB) 用戶端共用計數器。 以下是屬於擷取的一些重要 SMB 用戶端共用效能計數器：

| **類型**     | **SMB 用戶端共用計數器** |
|--------------|-------------------------------|
| IOPS         | 資料要求/秒             |
|              | 讀取要求/秒             |
|              | 寫入要求/秒            |
| Latency      | 平均秒/資料要求         |
|              | 平均秒/讀取                 |
|              | 平均秒/寫入                |
| IO 大小      | Avg.位元組/資料要求       |
|              | Avg.位元組/讀取               |
|              | Avg.位元組/寫入              |
| Throughput   | 資料位元組/秒                |
|              | 讀取位元組/秒                |
|              | 寫入位元組/秒               |
| 佇列長度 | Avg.讀取佇列長度        |
|              | Avg.寫入佇列長度       |
|              | Avg.資料佇列長度        |

### <a name="advanced-performance-analysis"></a>進階效能分析

當您執行進階效能分析時，需選取要平行執行的追蹤。 您可以視需要全部執行 (效能計數器、Xperf、網路和 StorPort)。  

> [!Note]
> 此案例會影響系統，所以不應該在實際的生產系統上執行。 如有必要，請在專用的維護視窗中執行此案例，以免發生任何問題。 因追蹤或基準測試所增加的工作負載，可能會對 VM 的效能造成負面影響。
>

## <a name="what-kind-of-information-is-collected-by-perfinsights"></a>PerfInsights 收集何種資訊？

收集的資訊包括 Windows VM、磁碟或儲存集區設定、效能計數器、記錄和各種追蹤。 這取決於您所使用的效能案例。 下表提供詳細資料：

|收集的資料                              |  |  | 效能測試案例 |  |  | |
|----------------------------------|----------------------------|------------------------------------|--------------------------|--------------------------------|----------------------|----------------------|
|                               | 快速效能分析 | 效能評定 | 效能分析 | Azure 檔案分析 | 進階效能分析 |
| 來自事件記錄的資訊       | 是                        | yes                                | yes                      | yes                  | 是                  |
| 系統資訊                | 是                        | yes                                | yes                      | yes                  | 是                  |
| 磁碟區對應                        | 是                        | yes                                | yes                      | yes                  | 是                  |
| 磁碟對應                          | 是                        | yes                                | yes                      | yes                  | 是                  |
| 正在執行的工作                     | 是                        | yes                                | yes                      | yes                  | 是                  |
| 儲存體可靠性計數器      | 是                        | yes                                | yes                      | yes                  | 是                  |
| 儲存體資訊               | 是                        | yes                                | yes                      | yes                  | 是                  |
| Fsutil 輸出                     | 是                        | yes                                | yes                      | yes                  | 是                  |
| 篩選器驅動程式資訊                | 是                        | yes                                | yes                      | yes                  | 是                  |
| Netstat 輸出                    | 是                        | yes                                | yes                      | yes                  | 是                  |
| 網路組態             | 是                        | yes                                | yes                      | yes                  | 是                  |
| 防火牆設定            | 是                        | yes                                | yes                      | yes                  | 是                  |
| SQL Server 設定          | 是                        | yes                                | yes                      | yes                  | 是                  |
| 效能診斷追蹤 *  | 是                        | yes                                | yes                      | yes                  | 是                  |
| 效能計數器追蹤 **      |                            |                                    | 是                      |                      | 是                  |
| SMB 計數器追蹤 **              |                            |                                    |                          | 是                  |                      |
| SQL Server 計數器追蹤 **       |                            |                                    | 是                      |                      | 是                  |
| XPerf 追蹤                       |                            |                                    |                          |                      | 是                  |
| StorPort 追蹤                    |                            |                                    |                          |                      | 是                  |
| 網路追蹤                     |                            |                                    |                          | 是                  | 是                  |
| Diskspd 基準追蹤 ***       |                            | 是                                |                          |                      |                      |
|       |                            |                         |                                                   |                      |                      |

### <a name="performance-diagnostics-trace-"></a>效能診斷追蹤 (*)

在背景中執行規則引擎來收集資料並診斷進行中的效能問題。 目前支援下列規則：

- HighCpuUsage 規則：偵測高 CPU 使用量時段，並顯示在這些時段內前幾名的 CPU 使用量取用者。
- HighDiskUsage 規則：偵測實體磁碟的高磁碟使用量時段，並顯示在這些時段內前幾名的磁碟使用量取用者。
- HighResolutionDiskMetric 規則：顯示每個實體磁碟每 50 毫秒的 IOPS、輸送量和 I/O 延遲計量。 它有助於快速找出磁碟節流時段。
- HighMemoryUsage 規則：偵測高記憶體使用量時段，並顯示在這些時段內前幾名的記憶體使用量取用者。

> [!NOTE] 
> 目前支援包含 .NET Framework 4.5 或更新版本的 Windows 版本。

### <a name="performance-counter-trace-"></a>效能計數器追蹤 (\*\*)

收集下列效能計數器︰

- \Process、\Processor、\Memory、\Thread、\PhysicalDisk 和 \LogicalDisk
- \Cache\Dirty Pages、\Cache\Lazy Write Flushes/sec、\Server\Pool Nonpaged、Failures 和 \Server\Pool Paged Failures
- \Network Interface、\IPv4\Datagrams、\IPv6\Datagrams、\TCPv4\Segments、\TCPv6\Segments、\Network Adapter、\WFPv4\Packets、\WFPv6\Packets、\UDPv4\Datagrams、\UDPv6\Datagrams、\TCPv4\Connection、\TCPv6\Connection、\Network QoS Policy\Packets、\Per Processor Network Interface Card Activity 和 \Microsoft Winsock BSP 下的已選取計數器

#### <a name="for-sql-server-instances"></a>SQL Server 執行個體
- \SQL Server:Buffer Manager、\SQLServer:Resource Pool Stats 和 \SQLServer:SQL Statistics\
- \SQLServer:Locks、\SQLServer:General、Statistics
- \SQLServer:Access 方法

#### <a name="for-azure-files"></a>Azure 檔案
\SMB 用戶端共用

### <a name="diskspd-benchmark-trace-"></a>Diskspd 基準追蹤 (***)
Diskspd I/O 工作負載測試 (OS 磁碟 [寫入] 和集區磁碟 [讀取/寫入])

## <a name="run-the-perfinsights-tool-on-your-vm"></a>在 VM 上執行 PerfInsights 工具

### <a name="what-do-i-have-to-know-before-i-run-the-tool"></a>執行此工具之前應該知道什麼？ 

#### <a name="tool-requirements"></a>工具需求

-  此工具必須在發生效能問題的 VM 上執行。 

-  支援下列作業系統：Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 和 Windows Server 2016；Windows 8.1 和 Windows 10。

#### <a name="possible-problems-when-you-run-the-tool-on-production-vms"></a>當您在生產 VM 上執行工具時可能發生的問題

-  對於設定為使用 Xperf 或 Diskspd 的基準案例或「進階效能分析」情節，此工具可能會對 VM 效能產生不良的影響。 這些案例不應在生產環境中執行。

-  對於設定為使用 Diskspd 的基準情節或「進階效能分析」情節，請確定 I/O 工作負載不受其他背景活動干擾。

-  根據預設，工具會使用暫存磁碟機來收集資料。 如果追蹤長時間保持啟用，則收集的資料量會有所影響。 這會降低暫存磁碟的可用空間，從而影響依賴此磁碟機的任何應用程式。

### <a name="how-do-i-run-perfinsights"></a>如何執行 PerfInsights？ 

藉由安裝 [Azure 效能診斷 VM 擴充功能](performance-diagnostics-vm-extension.md)，您可以在虛擬機器上執行 PerfInsights。 您也可以將它當作獨立的工具執行。 

**從 Azure 入口網站安裝及執行 PerfInsights**

如需此選項的詳細資訊，請參閱[安裝 Azure 效能診斷 VM 擴充功能](performance-diagnostics-vm-extension.md#install-the-extension)。  

**在獨立模式中執行 PerfInsights**

若要執行 PerfInsights 工具，請遵循下列步驟：


1. 下載 [PerfInsights.zip](http://aka.ms/perfinsightsdownload)。

2. 解除封鎖 PerfInsights.zip 檔案。 若要這樣做，請以滑鼠右鍵按一下 PerfInsights.zip 檔案，然後選取 [屬性]。 在 [一般] 索引標籤上，選取 [解除封鎖]，然後選取 [確定]。 如此可確保工具執行時不會出現任何額外的安全性提示。  

    ![PerfInsights 屬性的螢幕擷取畫面 (已醒目提示 [解除封鎖])](media/how-to-use-perfInsights/unlock-file.png)

3.  將 PerfInsights.zip 檔案解壓縮至暫存磁碟機 (根據預設，這通常為 D 磁碟機)。 

4.  以系統管理員身分開啟 Windows 命令提示字元，然後執行 PerfInsights.exe 來檢視可用的命令列參數。

    ```
    cd <the path of PerfInsights folder>
    PerfInsights
    ```
    ![PerfInsights 命令列輸出的螢幕擷取畫面](media/how-to-use-perfInsights/PerfInsightsCommandline.png)
    
    執行 PerfInsights 案例的基本語法如下：
    
    ```
    PerfInsights /run <ScenarioName> [AdditionalOptions]
    ```

    您可以使用下列範例來執行為時 5 分鐘的效能分析案例：
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    您可以使用下列範例來搭配 Xperf 和效能計數器追蹤執行為時 5 分鐘的進階案例：
    
    ```
    PerfInsights /run advanced xp /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    您可以使用下列範例來執行為時 5 分鐘的效能分析案例，然後將結果 ZIP 檔案上傳至儲存體帳戶：
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics /sa <StorageAccountName> /sk <StorageAccountKey>
    ```

    您可以使用 **/list** 命令，來查詢所有可用的案例和選項：
    
    ```
    PerfInsights /list
    ```

    >[!Note]
    >執行案例之前，PerfInsights 會提示使用者同意共用診斷資訊並同意 EULA。 使用 **/AcceptDisclaimerAndShareDiagnostics** 選項以略過這些提示。 
    >
    >如果您擁有 Microsoft 的作用中支援票證，並正在依照協助您之支援工程師的要求執行 PerfInsights，請務必使用 **/sr** 選項來提供支援票證號碼。
    >
    >根據預設，PerfInsights 會嘗試將自己更新為最新版本 (如果可以的話)。 使用 **/SkipAutoUpdate** 或 **/sau** 參數以略過自動更新。  
    >
    >如果未指定持續時間參數 **/d**，PerfInsights 會在您執行 vmslow、Azure 檔案和進階案例時提示您重新產生問題。 

當追蹤或作業都完成之後，與 PerfInsights 相同的資料夾中會出現一個新檔案。 檔案名稱為 **PerformanceDiagnostics\_yyyy-MM-dd\_hh-mm-ss-fff.zip。** 您可以將此檔案傳送給支援專員進行分析，或開啟 zip 檔案內的報表來檢閱結果和建議。

## <a name="review-the-diagnostics-report"></a>檢閱診斷報告

在 **PerformanceDiagnostics\_yyyy-MM-dd\_hh-mm-ss-fff.zip** 檔案內，您可以找到詳細說明 PerfInsights 結果的 HTML 報告。 若要檢閱報告，請展開 **PerformanceDiagnostics\_yyyy-MM-dd\_hh-mm-ss-fff.zip** 檔案，然後開啟 **PerfInsights Report.html** 檔案。

選取 [Findings] \(結果) 索引標籤。

![PerfInsights 報告的螢幕擷取畫面](media/how-to-use-perfInsights/findingtab.png)
![PerfInsights 報告的螢幕擷取畫面](media/how-to-use-perfInsights/findings.PNG)

> [!NOTE] 
> 分類為「高」的結果是可能會造成效能問題的已知問題。 分類為「中」的結果表示不一定會造成效能問題的非最佳化設定。 分類為「低」的結果是僅用來參考的資訊說明。

檢閱所有「高」和「中」結果的建議和連結。 了解這些結果如何影響效能，以及效能最佳化組態的最佳做法。

### <a name="storage-tab"></a>[儲存體] 索引標籤

**結果**區段會顯示有關儲存體的各種結果與建議。

[DiskMap] 和 [VolumeMap] 區段說明邏輯磁碟區和實體磁碟如何彼此相關。

在實體磁碟檢視方塊 (磁碟對應) 中，資料表會顯示在磁碟上執行的所有邏輯磁碟區。 在下列範例中，**PhysicalDrive2** 執行在多個分割區建立的 2 個邏輯磁碟區 (J 和 H)：

![磁碟索引標籤的螢幕擷取畫面](media/how-to-use-perfInsights/disktab.png)

在磁碟區檢視方塊中 (磁碟區對應)，資料表會顯示每個邏輯磁碟區下的所有實體磁碟。 請注意，對於 RAID/動態磁碟，您可能會在多個實體磁碟上執行邏輯磁碟區。 在下列範例中，*C:\\mount* 是在實體磁碟 2 和 3 上設定為 SpannedDisk 的掛接點：

![磁碟區索引標籤的螢幕擷取畫面](media/how-to-use-perfInsights/volumetab.png)

### <a name="sql-tab"></a>[SQL] 索引標籤

如果目標 VM 裝載任何 SQL Server 執行個體，您將在報表中看到名為 **SQL** 的其他索引標籤：

![SQL 索引標籤的螢幕擷取畫面](media/how-to-use-perfInsights/sqltab.png)

本區段包含 [結果] 索引標籤，以及裝載在虛擬機器上的每個 SQL Server 執行個體的其他索引標籤。

[結果] 索引標籤包含所有發現的 SQL 效能相關問題清單及建議事項。

下列範例中會顯示 **PhysicalDrive0** (執行 C 磁碟機)。 這是因為 **modeldev** 和 **modellog** 檔案都位於 C 磁碟機上，且為不同的類型 (例如它們分別為資料檔和交易記錄)。

![記錄資訊的螢幕擷取畫面](media/how-to-use-perfInsights/loginfo.png)

SQL Server 的特定執行個體索引標籤包含一般區段，其中顯示所選執行個體的基本資訊。 這些索引標籤也包含進階資訊的其他區段，包括設定、組態和使用者選項。

### <a name="diagnostic-tab"></a>[診斷] 索引標籤
[診斷] 索引標籤包含 PerfInsights 執行期間電腦上的最上層 CPU、磁碟和記憶體取用相關資訊。 您也可以找到以下相關資訊：可能缺少系統的重大修補程式、工作清單和重要系統事件。 

## <a name="references-to-the-external-tools-used"></a>所用外部工具的參考

### <a name="diskspd"></a>Diskspd

Diskspd 是來自 Microsoft 的儲存體負載產生器和效能測試工具。 如需詳細資訊，請參閱 [Diskspd](https://github.com/Microsoft/diskspd)。

### <a name="xperf"></a>Xperf

Xperf 是命令列工具，會從 Windows 效能工具組擷取追蹤。 如需詳細資訊，請參閱 [Windows 效能工具組 – Xperf](https://blogs.msdn.microsoft.com/ntdebugging/2008/04/03/windows-performance-toolkit-xperf/)。

## <a name="next-steps"></a>後續步驟

您可以將診斷記錄和報告上傳至 Microsoft 支援服務，以供進一步檢閱。 支援服務人員可能會要求您傳送 PerfInsights 產生的輸出，以協助進行疑難排解程序。

下列螢幕擷取畫面會顯示類似您可能收到的一則訊息：

![來自 Microsoft 支援服務之範例訊息的螢幕擷取畫面](media/how-to-use-perfInsights/supportemail.png)

請依照訊息中的指示來存取檔案傳輸工作區。 為增加安全性，您必須在第一次使用時變更密碼。

登入之後，您會發現一個對話方塊，可上傳 PerfInsights 收集的 **PerformanceDiagnostics\_yyyy-MM-dd\_hh-mm-ss-fff.zip** 檔案。

