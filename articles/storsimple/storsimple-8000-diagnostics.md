---
title: 針對 StorSimple 8000 裝置進行疑難排解的診斷工具 | Microsoft Docs
description: 描述 StorSimple 裝置模式並說明如何使用 Windows PowerShell for StorSimple 來變更裝置的模式。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: 7199009553eb7aae31db3f913fe4de87e03d74ba
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/10/2018
ms.locfileid: "27744107"
---
# <a name="use-the-storsimple-diagnostics-tool-to-troubleshoot-8000-series-device-issues"></a>使用 StorSimple 診斷工具針對 8000 系列裝置問題進行疑難排解

## <a name="overview"></a>概觀

StorSimple 診斷工具可診斷 StorSimple 裝置的系統、效能、網路和硬體元件健全狀況的相關問題。 診斷工具可用在各種情況。 這些情況包括工作負載規劃、部署 StorSimple 裝置、評估網路環境，以及判斷作業裝置的效能。 本文提供診斷工具的概觀，並說明如何使用此工具來診斷 StorSimple 裝置。

診斷工具主要用於 StorSimple 8000 系列內部部署裝置 (8100 和 8600)。

## <a name="run-diagnostics-tool"></a>執行診斷工具

您可以透過 StorSimple 裝置的 Windows PowerShell 介面執行此工具。 有兩種方式可存取裝置的本機介面︰

* [使用 PuTTY 連線至裝置序列主控台](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)
* [透過適用於 StorSimple 的 Windows PowerShell 從遠端存取工具](storsimple-8000-remote-connect.md)。

在本文中，我們假設您已透過 PuTTY 連線至裝置序列主控台。

#### <a name="to-run-the-diagnostics-tool"></a>執行診斷工具

連線至裝置的 Windows PowerShell 介面之後，執行下列步驟來執行 Cmdlet。
1. 遵循 [使用 PuTTY 來連接到裝置序列主控台](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)中的步驟，登入裝置序列主控台。

2. 輸入以下命令：

    `Invoke-HcsDiagnostics`

    如果未指定 scope 參數，Cmdlet 會執行所有診斷測試。 這些測試包括系統、硬體元件健全狀況、網路和效能。 
    
    若只要執行特定測試，請指定 scope 參數。 例如，若只要執行網路測試，請輸入

    `Invoke-HcsDiagnostics -Scope Network`

3. 選取 PuTTY 視窗中的輸出，並複製到文字檔以進一步分析。

## <a name="scenarios-to-use-the-diagnostics-tool"></a>使用診斷工具的情況

使用診斷工具針對網路、效能、系統和系統的硬體健全狀況進行疑難排解。 以下是可能的情況：

* **裝置離線** - StorSimple 8000 系列裝置已離線。 但從 Windows PowerShell 介面中查看，兩個控制器似乎都正常運作。
    * 您可以使用此工具判斷網路狀態。
         
         > [!NOTE]
         > 在註冊 (或透過安裝精靈進行設定) 裝置之前，請勿使用此工具來評估裝置的效能和網路設定。 在安裝精靈和註冊期間會指派有效的 IP 給裝置。 在未註冊的裝置上，您可以執行此 Cmdlet 診斷硬體健全狀況和系統。 使用 scope 參數，例如︰
         >
         > `Invoke-HcsDiagnostics -Scope Hardware`
         >
         > `Invoke-HcsDiagnostics -Scope System`

* **持續的裝置問題** - 發生的裝置問題似乎持續不斷。 比方說，註冊失敗。 在裝置成功註冊並運作一段時間之後，也可能會發生裝置問題。
    * 在此情況下，請使用此工具進行初步疑難排解，再向 Microsoft 支援服務登記服務要求。 我們建議您執行這項工具，並擷取這項工具的輸出。 然後，您可以提供此輸出給支援服務，以加速疑難排解。
    * 如果發生任何硬體元件或叢集失敗，您應該登記支援要求。

* **裝置效能低落** - StorSimple 裝置變慢。
    * 在此情況下，請執行此 Cmdlet，並將 scope 參數設為 performance。 分析輸出。 您會知道雲端讀寫延遲。 請使用報告的延遲當做為可達成的最高目標，考量內部資料處理時的一些額外負荷，然後在系統上部署工作負載。 如需詳細資訊，請參閱[使用網路測試針對裝置效能進行疑難排解](#network-test)。


## <a name="diagnostics-test-and-sample-outputs"></a>診斷測試和輸出範例

### <a name="hardware-test"></a>硬體測試

這項測試會判斷系統上執行的硬體元件、USM 韌體和磁碟韌體的狀態。

* 報告的硬體元件是未通過測試或不存在系統中的元件。
* 報告的 USM 韌體和磁碟韌體版本是指系統中的控制器 0、控制器 1 和共用元件。 如需硬體元件的完整清單，請移至︰

    * [主要機箱中的元件](storsimple-8000-monitor-hardware-status.md#component-list-for-primary-enclosure-of-storsimple-device)
    * [EBOD 機箱中的元件](storsimple-8000-monitor-hardware-status.md#component-list-for-ebod-enclosure-of-storsimple-device)

> [!NOTE]
> 如果硬體測試報告失敗的元件，請[向 Microsoft 支援服務登記服務要求](storsimple-8000-contact-microsoft-support.md)。

#### <a name="sample-output-of-hardware-test-run-on-an-8100-device"></a>在 8100 裝置上執行硬體測試的輸出範例

以下是 StorSimple 8100 裝置的輸出範例。 8100 機型裝置中沒有 EBOD 機箱。 因此，不會報告 EBOD 控制器元件。

```
Controller0>Invoke-HcsDiagnostics -Scope Hardware
Running hardware diagnostics ...
--------------------------------------------------
Hardware components failed or not present
----------------------

           Type           State      Controller           Index     EnclosureId
           ----           -----      ----------           -----     -----------
...rVipResource      NotPresent            None               1            None
...rVipResource      NotPresent            None               2            None
...rVipResource      NotPresent            None               3            None
...rVipResource      NotPresent            None               4            None
...rVipResource      NotPresent            None               5            None
...rVipResource      NotPresent            None               6            None
...rVipResource      NotPresent            None               7            None
...rVipResource      NotPresent            None               8            None
...rVipResource      NotPresent            None               9            None
...rVipResource      NotPresent            None              10            None
...rVipResource      NotPresent            None              11            None

Firmware information
----------------------
TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

--------------------------------------------------
```

### <a name="system-test"></a>系統測試

這項測試會報告系統資訊、可用的更新、叢集資訊，以及裝置的服務資訊。

* 系統資訊包括機型、裝置序號、時區、控制器狀態，以及系統上執行的詳細軟體版本。 若要了解輸出所報告的各種系統參數，請參閱[解譯系統資訊](#appendix-interpreting-system-information)。

* 更新可用性報告定期和維護模式是否可用及其相關聯的套件名稱。 如果 `RegularUpdates` 和 `MaintenanceModeUpdates` 是 `false`，這表示沒有可用的更新。 您的裝置已是最新狀態。
* 叢集資訊包含所有 HCS 叢集群組的各種邏輯元件及其個別狀態的相關資訊。 如果您在此報告區段看到離線叢集群組，請[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。
* 服務資訊包含在您的裝置上執行的所有 HCS 和 CiS 服務的名稱和狀態。 這項資訊可協助 Microsoft 支援服務針對裝置問題進行疑難排解。

#### <a name="sample-output-of-system-test-run-on-an-8100-device"></a>在 8100 裝置上執行系統測試的輸出範例

以下是在 8100 裝置上執行系統測試的輸出範例。

```
Controller0>Invoke-HcsDiagnostics -Scope System
Running system diagnostics ...
--------------------------------------------------

System information
----------------------
Controller0:

InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                : (UTC-08:00) Pacific Time (US & Canada)
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : Disabled
FipsMode                : Enabled

Controller1:
InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                :
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : HttpsAndHttpEnabled
FipsMode                : Enabled

Update availability
----------------------
RegularUpdates              : False
MaintenanceModeUpdates      : False
RegularUpdatesTitle         : {}
MaintenanceModeUpdatesTitle : {}

Cluster information
----------------------
Name                          State OwnerGroup
----                          ----- ----------
ApplicationHostRLUA           Online HCS Cluster Group
Data0v4                       Online HCS Cluster Group
HCS Vnic Resource             Online HCS Cluster Group
hcs_cloud_connectivity_...    Online HCS Cluster Group
hcs_controller_replacement    Online HCS Cluster Group
hcs_datapath_service          Online HCS Cluster Group
hcs_management_servic         Online HCS Cluster Group
hcs_nvram_service             Online HCS Cluster Group
hcs_passive_datapath          Online HCS Passive Cluster Group
hcs_platform_service          Online HCS Cluster Group
hcs_saas_agent_service        Online HCS Cluster Group
HddDataClusterDisk            Online HCS Cluster Group
HddMgmtClusterDisk            Online HCS Cluster Group
HddReplClusterDisk            Online HCS Cluster Group
iSCSI Target Server           Online HCS Cluster Group
NvramClusterDisk              Online HCS Cluster Group
SSAdminRLUA                   Online HCS Cluster Group
SsdDataClusterDisk            Online HCS Cluster Group
SsdNvramClusterDisk           Online HCS Cluster Group

Service information
----------------------
Name                                          Status DisplayName
----                                          ------ -----------
CiSAgentSvc                                   Stopped CiS Service Agent
hcs_cloud_connectivity_...                    Running hcs_cloud_connectivity...
hcs_controller_replacement                    Running HCS Controller Replace...
hcs_datapath_service                          Running HCS Datapath Service
hcs_management_service                        Running HCS Management Service
hcs_minishell                                 Running hcs_minishell
HCS_NVRAM_Service                             Running HCS NVRAM Service
hcs_passive_datapath                          Stopped HCS Passive Datapath S...
hcs_platform_service                          Running HCS Platform Monitor S...
hcs_saas_agent_service                        Running hcs_saas_agent_service
hcs_startup                                   Stopped hcs_startup
--------------------------------------------------
```

### <a name="network-test"></a>網路測試

這項測試會驗證網路介面、連接埠、DNS 和 NTP 伺服器連線的狀態、SSL 憑證、儲存體帳戶認證、更新伺服器的連線能力，以及 StorSimple 裝置上的 Web Proxy 連線能力。

#### <a name="sample-output-of-network-test-when-only-data0-is-enabled"></a>僅啟用 DATA0 時的網路測試輸出範例

以下是 8100 裝置的輸出範例。 您可以在輸出中看到︰
* 只有啟用和設定 DATA 0 和 DATA 1 網路介面。
* 入口網站中未啟用 DATA 2 - 5。
* DNS 伺服器組態無效，裝置可以透過 DNS 伺服器來連線。
* NTP 伺服器連線能力也正常。
* 連接埠 80 和 443 已開啟。 但已封鎖連接埠 9354。 根據[系統網路需求](storsimple-system-requirements.md)，您必須開啟此連接埠，服務匯流排通訊才能進行。
* SSL 憑證有效。
* 裝置可以連線至儲存體帳戶︰_myss8000storageacct_。
* 更新伺服器的連線能力有效。
* 此裝置上未設定 Web Proxy。

#### <a name="sample-output-of-network-test-when-data0-and-data1-are-enabled"></a>啟用 DATA0 和 DATA1 時的網路測試輸出範例

```
Controller0>Invoke-HcsDiagnostics -Scope Network
Running network diagnostics ....
--------------------------------------------------
Validating networks .....
Name                Entity              Result              Details
----                ------              ------              -------
Network interface   Data0               Valid
Network interface   Data1               Valid
Network interface   Data2               Not enabled
Network interface   Data3               Not enabled
Network interface   Data4               Not enabled
Network interface   Data5               Not enabled
DNS                 10.222.118.154      Valid
NTP                 time.windows.com    Valid
Port                80                  Open
Port                443                 Open
Port                9354                Blocked
SSL certificate     https://myss8000... Valid
Storage account ... myss8000storageacct Valid
URL                 http://download.... Valid
URL                 http://download.... Valid
Web proxy                               Not enabled         Web proxy is not...
--------------------------------------------------
```

### <a name="performance-test"></a>效能測試

這項測試會透過裝置的雲端讀寫延遲來報告雲端效能。 這項工具可以用來建立 StorSimple 可達到的雲端效能基準線。 此工具會報告連線可達成的最大效能 (讀寫延遲的最佳案例)。

由於此工具會報告可達成的最大效能，我們可以在部署工作負載時，將所報告的讀寫延遲當做目標。

測試會模擬裝置上不同磁碟區類型相關聯的 blob 大小。 固定在本機之磁碟區的一般分層和備份使用 64 KB blob 大小。 勾選封存選項的分層磁碟區使用 512 KB blob 資料大小。 如果您的裝置已設定分層和固定在本機的磁碟區，則只會執行對應至 64 KB blob 資料大小的測試。

若要使用此工具，請執行下列步驟：

1.  首先，混合建立分層磁碟區和勾選封存選項的分層磁碟區。 此動作可確保工具會同時對 64 KB 和 512 KB blob 大小執行測試。

2. 建立並設定磁碟區之後，執行此 Cmdlet。 輸入：

    `Invoke-HcsDiagnostics -Scope Performance`

3. 記下工具所報告的讀寫延遲。 這項測試可能需要執行幾分鐘之後才會報告結果。

4. 如果連線延遲全部在預期的範圍內，則工具所報告的延遲，可當做部署工作負載時可達成的最高目標。 考量內部資料處理時的一些額外負荷。

    如果診斷工具所報告的讀寫延遲很高︰

    1. 設定 blob 服務的儲存體分析，然後分析輸出，以了解 Azure 儲存體帳戶的延遲。 如需詳細指示，請參閱[啟用和設定儲存體分析](../storage/common/storage-enable-and-view-metrics.md)。 如果這些延遲也很高，而且與您從 StorSimple 診斷工具收到的數字齊鼓相當，則您需要向 Azure 儲存體登記服務要求。

    2. 如果儲存體帳戶延遲較低，請連絡網路系統管理員來調查網路中的延遲問題。

#### <a name="sample-output-of-performance-test-run-on-an-8100-device"></a>在 8100 裝置上執行效能測試的輸出範例

```
Controller0>Invoke-HcsDiagnostics -Scope Performance
Running performance diagnostics...
--------------------------------------------------
Cloud performance: writing blobs
Cloud write latency: 194 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: reading blobs..
Cloud read latency: 544 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: writing blobs.
Cloud write latency: 369 ms using credential 'myss8000storageacct', blob size '512KB'
Cloud performance: reading blobs...
Cloud read latency: 4924 ms using credential 'myss8000storageacct', blob size '512KB'
--------------------------------------------------
Controller0>
```

## <a name="appendix-interpreting-system-information"></a>附錄︰解譯系統資訊

下表描述系統資訊中各種 Windows PowerShell 參數對應的意義。 

| PowerShell 參數    | 說明  |
|-------------------------|------------------|
| 執行個體識別碼             | 每個控制器都有相關聯的唯一識別碼或 GUID。|
| Name                    | 在裝置部署期間，透過 Azure 入口網站設定的裝置易記名稱。 預設的易記名稱是裝置序號。 |
| 模型                   | StorSimple 8000 系列裝置的機型。 機型可以是 8100 或 8600。|
| SerialNumber            | 裝置序號是在工廠裡指派，長度為 15 個字元。 例如，8600-SHX0991003G44HT 表示：<br> 8600 – 裝置機型。<br>SHX – 製造場所。<br> 0991003 – 特定產品。 <br> G44HT– 最後 5 位數會遞增以產生唯一序號。 這可能不是連續的組合。|
| TimeZone                | 裝置時區是在裝置部署期間於 Azure 入口網站中設定。|
| CurrentController       | 您透過 StorSimple 裝置的 Windows PowerShell 介面連接的控制器。|
| ActiveController        | 在裝置上運作的控制器，控制所有網路和磁碟作業。 這可以是控制器 0 或控制器 1。  |
| Controller0Status       | 裝置上控制器 0 的狀態。 控制器狀態可以是正常、復原模式或無法存取。|
| Controller1Status       | 裝置上控制器 1 的狀態。  控制器狀態可以是正常、復原模式或無法存取。|
| SystemMode              | StorSimple 裝置的整體狀態。 裝置狀態可以是正常、維護或已解除 (對應至 Azure 入口網站中的已停用）。|
| FriendlySoftwareVersion | 對應至裝置軟體版本的易記字串。 如果是執行 Update 4 的系統，好易軟體版本會是 StorSimple 8000 Series Update 4.0。|
| HcsSoftwareVersion      | 裝置上執行的 HCS 軟體版本。 例如，對應至 StorSimple 8000 Series Update 4.0 的 HCS 軟體版本是 6.3.9600.17820。 |
| ApiVersion              | HCS 裝置的 Windows PowerShell API 軟體版本。|
| VhdVersion              | 裝置所隨附之原廠映像的軟體版本。 如果您將裝置重設為原廠預設值，它會執行此軟體版本。|
| OSVersion               | 裝置上執行之 Windows Server 作業系統的軟體版本。 StorSimple 裝置是根據對應至 6.3.9600 的 Windows Server 2012 R2。|
| CisAgentVersion         | StorSimple 裝置上執行的 Ci 代理程式版本。 此代理程式可協助與 Azure 中執行的 StorSimple Manager 服務進行通訊。|
| MdsAgentVersion         | 與 StorSimple 裝置上執行的 Mds 代理程式相對應的版本。 此代理程式會將資料移至的監視與診斷服務 (MDS)。|
| Lsisas2Version          | 與 StorSimple 裝置上的 LSI 驅動程式相對應的版本。|
| Capacity                | 裝置的總容量 (以位元組為單位)。|
| RemoteManagementMode    | 指出是否可以透過 Windows PowerShell 介面從遠端管理裝置。 |
| FipsMode                | 指出您的裝置上是否啟用美國聯邦資訊處理標準 (FIPS) 模式。 FIPS 140 標準定義核准美國聯邦政府電腦系統所使用的密碼編譯演算法來保護機密資料。 如果是執行 Update 4 或更新版本的裝置，預設會啟用 FIPS 模式。 |

## <a name="next-steps"></a>後續步驟

* 了解 [Invoke-HcsDiagnostics Cmdlet 的語法](https://technet.microsoft.com/library/mt795371.aspx)。

* 深入了解如何在 StorSimple 裝置上[針對部署問題進行疑難排解](storsimple-troubleshoot-deployment.md)。
