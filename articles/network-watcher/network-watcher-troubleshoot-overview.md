---
title: Azure 網路監看員中的資源疑難排解簡介 | Microsoft Docs
description: 本頁提供網路監看員資源疑難排解功能的概觀
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.assetid: c1145cd6-d1cf-4770-b1cc-eaf0464cc315
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2017
ms.author: jdial
ms.openlocfilehash: 2f8a41834c1451d80c53cfed4bae3b7e36281702
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32779255"
---
# <a name="introduction-to-resource-troubleshooting-in-azure-network-watcher"></a>Azure 網路監看員中的資源疑難排解簡介

虛擬網路閘道可提供 Azure 中內部部署資源與其他虛擬網路之間的連線。 監視閘道器及其連線，對於確保通訊不中斷至關重要。 網路監看員可提供針對閘道和連線進行疑難排解的功能。 此功能可透過入口網站、PowerShell、Azure CLI 或 REST API 呼叫。 一經呼叫，網路監看員就會診斷閘道或連線的健康情況，並傳回相關結果。 此要求是長時間執行的交易。 完成診斷之後，會傳回結果。

![入口網站][2]

## <a name="results"></a>結果

傳回的初步結果會提供資源的整體健康情況。 可以針對資源提供如下一節所示的更深入資訊︰

下列清單是透過疑難排解 API 傳回的值︰

* **startTime** - 此值是疑難排解 API 呼叫的開始時間。
* **endTime** - 此值是疑難排解結束時的時間。
* **code** - 如果有單一診斷失敗，此值為 UnHealthy。
* **results** - 結果是在連線或虛擬網路閘道上傳回的結果集合。
    * **id** - 此值是錯誤類型。
    * **summary** - 此值是錯誤摘要。
    * **detailed** - 此值提供錯誤的詳細說明。
    * **recommendedActions** - 此屬性是建議採取的動作集合。
      * **actionText** - 此值包含用於描述所要採取之動作的文字。
      * **actionUri** - 此值提供如何採取行動的文件URI。
      * **actionUriText** - 此值是動作文字的簡短說明。

下表顯示可用的不同錯誤類型 (上述清單中 results 底下的 id) 以及該錯誤是否會建立記錄檔。

### <a name="gateway"></a>閘道器

| 錯誤類型 | 原因 | 記錄檔|
|---|---|---|
| NoFault | 未偵測到任何錯誤時 |yes|
| GatewayNotFound | 找不到閘道或閘道尚未佈建 |否|
| PlannedMaintenance |  閘道執行個體正在進行維護  |否|
| UserDrivenUpdate | 當正在更新使用者時，會發生此錯誤。 此更新可能是調整大小作業。 | 否 |
| VipUnResponsive | 由於健康情況探查失敗而無法連線到閘道的主要執行個體時，會發生此錯誤。 | 否 |
| PlatformInActive | 平台發生問題。 | 否|
| ServiceNotRunning | 基礎服務並未執行。 | 否|
| NoConnectionsFoundForGateway | 閘道上沒有任何連線存在。 此錯誤只是警告。| 否|
| ConnectionsNotConnected | 未建立連線。 此錯誤只是警告。| yes|
| GatewayCPUUsageExceeded | 目前的閘道 CPU 使用量 > 95%。 | yes |

### <a name="connection"></a>連線

| 錯誤類型 | 原因 | 記錄檔|
|---|---|---|
| NoFault | 未偵測到任何錯誤時 |yes|
| GatewayNotFound | 找不到閘道或閘道尚未佈建 |否|
| PlannedMaintenance | 閘道執行個體正在進行維護  |否|
| UserDrivenUpdate | 當正在更新使用者時，會發生此錯誤。 此更新可能是調整大小作業。  | 否 |
| VipUnResponsive | 由於健康情況探查失敗而無法連線到閘道的主要執行個體時，會發生此錯誤。 | 否 |
| ConnectionEntityNotFound | 缺少連線組態 | 否 |
| ConnectionIsMarkedDisconnected | 連線標記為「已中斷連線」 |否|
| ConnectionNotConfiguredOnGateway | 基礎服務未設定連線。 | yes |
| ConnectionMarkedStandy | 基礎服務標記為「待命」。| yes|
| 驗證 | 預先共用的金鑰不相符 | yes|
| PeerReachability | 無法連線到對等閘道。 | yes|
| IkePolicyMismatch | 對等閘道的 IKE 原則不受 Azure 支援。 | yes|
| WfpParse 錯誤 | 剖析 WFP 記錄時發生錯誤。 |yes|

## <a name="supported-gateway-types"></a>支援的閘道類型

下表列出網路監看員疑難排解所支援的閘道和連線：

|  |  |
|---------|---------|
|**閘道類型**   |         |
|VPN      | 支援        |
|ExpressRoute | 不支援 |
|**VPN 類型** | |
|路由式 | 支援|
|原則式 | 不支援|
|**連線類型**||
|IPsec| 支援|
|VNet2Vnet| 支援|
|ExpressRoute| 不支援|
|VPNClient| 不支援|

## <a name="log-files"></a>記錄檔

在資源疑難排解完成之後，資源疑難排解記錄檔會儲存在儲存體帳戶中。 下圖顯示造成錯誤的呼叫內容範例。

![zip 檔案][1]

> [!NOTE]
> 在某些情況下，只有部分的記錄檔會寫入至儲存體。

如需從 Azure 儲存體帳戶下載檔案的指示，請參閱[以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 另一項可用工具為儲存體總管。 有關儲存體總管的詳細資訊可以在下列連結找到︰[儲存體總管](http://storageexplorer.com/)

### <a name="connectionstatstxt"></a>ConnectionStats.txt

**ConnectionStats.txt** 檔案包含連線的整體統計資料，包括輸入和輸出位元組、連線狀態，以及連線的建立時間。

> [!NOTE]
> 如果疑難排解 API 的呼叫傳回狀況良好，則 zip 檔案中傳回的唯一項目是 **ConnectionStats.txt** 檔案。

此檔案的內容類似於下列範例：

```
Connectivity State : Connected
Remote Tunnel Endpoint :
Ingress Bytes (since last connected) : 288 B
Egress Bytes (Since last connected) : 288 B
Connected Since : 2/1/2017 8:22:06 PM
```

### <a name="cpustatstxt"></a>CPUStats.txt

**CPUStats.txt** 檔案包含測試階段可用的 CPU 使用量與記憶體。  此檔案的內容類似於下列範例：

```
Current CPU Usage : 0 % Current Memory Available : 641 MBs
```

### <a name="ikeerrorstxt"></a>IKEErrors.txt

**IKEErrors.txt** 檔案包含在監視期間找到的任何 IKE 錯誤。

下列範例顯示 IKEErrors.txt 檔案的內容。 您的錯誤可能因問題而有所不同。

```
Error: Authentication failed. Check shared key. Check crypto. Check lifetimes. 
     based on log : Peer failed with Windows error 13801(ERROR_IPSEC_IKE_AUTH_FAIL)
Error: On-prem device sent invalid payload. 
     based on log : IkeFindPayloadInPacket failed with Windows error 13843(ERROR_IPSEC_IKE_INVALID_PAYLOAD)
```

### <a name="scrubbed-wfpdiagtxt"></a>Scrubbed-wfpdiag.txt

**Scrubbed-wfpdiag.txt** 記錄檔包含 wfp 記錄。 此記錄包含套件置放和 IKE/AuthIP 失敗的記錄。

下列範例顯示 Scrubbed-wfpdiag.txt 檔案的內容。 在此範例中，連線的共用金鑰不正確 (可以從底部算起的第三行看出來)。 下列範例是只是整個記錄的某個片段，因為視問題而定，記錄可能很冗長。

```
...
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Deleted ICookie from the high priority thread pool list
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|IKE diagnostic event:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Event Header:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Timestamp: 1601-01-01T00:00:00.000Z
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Flags: 0x00000106
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    Local address field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    Remote address field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    IP version field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  IP version: IPv4
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  IP protocol: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Local address: 13.78.238.92
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Remote address: 52.161.24.36
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Local Port: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Remote Port: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Application ID:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  User SID: <invalid>
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Failure type: IKE/Authip Main Mode Failure
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Type specific info:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Failure error code:0x000035e9
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    IKE authentication credentials are unacceptable
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Failure point: Remote
...
```

### <a name="wfpdiagtxtsum"></a>wfpdiag.txt.sum

**wfpdiag.txt.sum** 檔案是顯示已處理緩衝區和事件的記錄。

下列範例是 wfpdiag.txt.sum 檔案的內容。
```
Files Processed:
    C:\Resources\directory\924336c47dd045d5a246c349b8ae57f2.GatewayTenantWorker.DiagnosticsStorage\2017-02-02T17-34-23\wfpdiag.etl
Total Buffers Processed 8
Total Events  Processed 2169
Total Events  Lost      0
Total Format  Errors    0
Total Formats Unknown   486
Elapsed Time            330 sec
+-----------------------------------------------------------------------------------+
|EventCount    EventName            EventType   TMF                                 |
+-----------------------------------------------------------------------------------+
|        36    ikeext               ike_addr_utils_c844  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|        12    ikeext               ike_addr_utils_c857  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|        96    ikeext               ike_addr_utils_c832  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|         6    ikeext               ike_bfe_callbacks_c133  1dc2d67f-8381-6303-e314-6c1452eeb529|
|         6    ikeext               ike_bfe_callbacks_c61  1dc2d67f-8381-6303-e314-6c1452eeb529|
|        12    ikeext               ike_sa_management_c5698  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|         6    ikeext               ike_sa_management_c8447  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c494  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c642  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|         6    ikeext               ike_sa_management_c3162  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c3307  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
```

## <a name="next-steps"></a>後續步驟

若要了解如何診斷閘道或閘道連線的問題，請參閱[診斷網路間的通訊問題](diagnose-communication-problem-between-networks.md)。
<!--Image references-->

[1]: ./media/network-watcher-troubleshoot-overview/GatewayTenantWorkerLogs.png
[2]: ./media/network-watcher-troubleshoot-overview/portal.png
