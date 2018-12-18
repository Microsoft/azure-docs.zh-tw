---
title: Azure Service Fabric CLI - sfctl cluster | Microsoft Docs
description: 描述 Service Fabric CLI sfctl cluster 命令。
services: service-fabric
documentationcenter: na
author: Christina-Kang
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: cli
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: 4b0491d59e4ac495750a338ad743aab69ff47a4e
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494238"
---
# <a name="sfctl-cluster"></a>sfctl cluster
選取、管理和操作 Service Fabric 叢集。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| code-versions | 取得在 Service Fabric 叢集中佈建的網狀架構程式碼版本清單。 |
| config-versions | 取得在 Service Fabric 叢集中佈建的網狀架構組態版本清單。 |
| health | 取得 Service Fabric 叢集的健康情況。 |
| manifest | 取得 Service Fabric 叢集資訊清單。 |
| operation-cancel | 取消使用者引起的錯誤作業。 |
| operation-list | 取得依提供的輸入所篩選之使用者引起的錯誤作業清單。 |
| provision | 佈建 Service Fabric 叢集的程式碼或組態封裝。 |
| recover-system | 指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的系統服務。 |
| report-health | 傳送 Service Fabric 叢集的健康情況報告。 |
| 選取 | 連線到 Service Fabric 叢集端點。 |
| show-connection | 顯示這個 sfctl 執行個體連線到哪個 Service Fabric 叢集。 |
| unprovision | 解除佈建 Service Fabric 叢集的程式碼或組態封裝。 |
| 升級 | 開始升級 Service Fabric 叢集的程式碼或組態版本。 |
| upgrade-resume | 將叢集升級移至下一個升級網域。 |
| upgrade-rollback | 復原 Service Fabric 叢集的升級。 |
| upgrade-status | 取得目前叢集升級的進度。 |
| upgrade-update | 更新 Service Fabric 叢集升級的升級參數。 |

## <a name="sfctl-cluster-code-versions"></a>sfctl cluster code-versions
取得在 Service Fabric 叢集中佈建的網狀架構程式碼版本清單。

針對在叢集中佈建的網狀架構程式碼版本，取得相關資訊的清單。 參數 CodeVersion 可用來視需要將輸出篩選成僅限該特定版本。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --code-version | Service Fabric 的產品版本。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-config-versions"></a>sfctl cluster config-versions
取得在 Service Fabric 叢集中佈建的網狀架構組態版本清單。

針對在叢集中佈建的網狀架構組態版本，取得相關資訊的清單。 參數 ConfigVersion 可用來視需要將輸出篩選成僅限該特定版本。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --config-version | Service Fabric 的組態版本。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-health"></a>sfctl cluster health
取得 Service Fabric 叢集的健康情況。

使用 EventsHealthStateFilter 可根據健康情況狀態，篩選針對叢集所報告的健康情況事件集合。 同樣地，使用 NodesHealthStateFilter 和 ApplicationsHealthStateFilter 可根據彙總的健康情況狀態，篩選傳回的節點和應用程式集合。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --applications-health-state-filter | 允許根據健康情況狀態，篩選叢集健康情況查詢結果中所傳回的應用程式健康情況狀態物件。 此參數的可能值包括從 HealthStateFilter 列舉成員或對這些成員的位元運算取得的整數值。 只會傳回符合篩選條件的應用程式。 所有應用程式都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的應用程式健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --exclude-health-statistics | 指出是否應該在查詢結果中一併傳回健康情況統計資料。 預設為 False。 統計資料會顯示健康情況狀態為 Ok、Warning 及 Error 的子實體數目。 |
| --include-system-application-health-statistics | 指出健康情況統計資料是否應該包含 fabric\:/System 應用程式健康情況統計資料。 預設為 False。 如果將 IncludeSystemApplicationHealthStatistics 設定為 true，健康情況統計資料就會包含屬於 fabric\:/System 應用程式的實體。 否則，查詢結果只會包含使用者應用程式的健康情況統計資料。 健康情況統計資料必須包含在查詢結果中，才能套用此參數。 |
| --nodes-health-state-filter | 允許根據健康情況狀態，篩選叢集健康情況查詢結果中所傳回的節點健康情況狀態物件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的節點。 所有節點都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的節點健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 為完整偵錯記錄。 |

## <a name="sfctl-cluster-manifest"></a>sfctl cluster manifest
取得 Service Fabric 叢集資訊清單。

取得 Service Fabric 叢集資訊清單。 叢集資訊清單包含叢集的屬性，其中包括叢集上各種不同的節點類型、安全性設定、錯誤和升級網域拓撲等等。部署獨立叢集時，會在 ClusterConfig.JSON 檔案中一併指定這些屬性。 不過，叢集資訊清單中大部分的資訊都會由服務網狀架構在於其他部署案例 (例如使用 Azure 入口網站時) 中進行叢集部署的期間，從內部產生。 叢集資訊清單的內容僅供參考之用，使用者不應該依賴檔案內容或其解釋的格式。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-operation-cancel"></a>sfctl cluster operation-cancel
取消使用者引起的錯誤作業。

下列 API 啟動的錯誤作業可藉由使用 CancelOperation 來取消：StartDataLoss、StartQuorumLoss、StartPartitionRestart、StartNodeTransition。 如果 force 為 false，將會以正常方式停止並清除指定的使用者引發作業。  如果 force 為 true，則會中止命令，而可能遺留某些內部狀態。  將 force 指定為 true 應該謹慎使用。 必須已經在相同的測試命令上先搭配將 force 設定為 false 來呼叫此 API，或除非測試命令的 OperationState 已經是 OperationState.RollingBack，才允許搭配將 force 設定為 true 來呼叫此 API。 

 說明：OperationState.RollingBack 表示系統將會/正在清除執行命令所造成的內部系統狀態。 如果測試命令是用來引發資料遺失，則它將不會還原資料。  例如，如果您呼叫 StartDataLoss，然後呼叫此 API，系統將只會清除從執行命令產生的內部狀態。 如果命令已進展到足以造成資料遺失的地步，則它將不會還原目標分割區的資料。 

> [!NOTE]
> 如果搭配 force==true 來叫用此 API，可能會遺留內部狀態。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --force | 指出是否要以正常方式復原並清除執行使用者引發作業所修改的內部系統狀態。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-operation-list"></a>sfctl cluster operation-list
取得依提供的輸入所篩選之使用者引起的錯誤作業清單。

依所提供的輸入篩選之後，取得使用者引發的錯誤作業清單。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --state-filter | 針對使用者引發的作業，用來依據 OperationState 進行篩選。 <br> 65535 - 全選 <br> 1 - 選取 Running <br> 2 - 選取 RollingBack <br>8 - 選取 Completed <br>16 - 選取 Faulted <br>32 - 選取 Cancelled <br>64 - 選取 ForceCancelled。  <br>預設值\:65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --type-filter | 針對使用者引發的作業，用來依據 OperationType 進行篩選。 <br> 65535 - 全選 <br> 1 - 選取 PartitionDataLoss。 <br> 2 - 選取 PartitionQuorumLoss。 <br> 4 - 選取 PartitionRestart。 <br> 8 - 選取 NodeTransition。  <br> 預設值\:65535。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-provision"></a>sfctl cluster provision
佈建 Service Fabric 叢集的程式碼或組態封裝。

驗證並佈建 Service Fabric 叢集的程式碼或組態封裝。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --cluster-manifest-file-path | 叢集資訊清單檔案路徑。 |
| --code-file-path | 叢集程式碼封裝檔案路徑。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-recover-system"></a>sfctl cluster recover-system
指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的系統服務。

指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的系統服務。 只有當已知無法復原已關閉的複本時，才能執行這項作業。 不正確使用這個 API，可能會造成資料遺失。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-report-health"></a>sfctl cluster report-health
傳送 Service Fabric 叢集的健康情況報告。

此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道節點，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請檢查此報告是否出現在叢集的 HealthEvents 中。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --health-property [必要] | 健康情況資訊的屬性。 <br><br> 實體可以有不同屬性的健康情況報告。 屬性是一個字串而不是固定的列舉，以便讓報告程式在分類觸發報告的狀態條件時較有彈性。 例如，SourceId 為 "LocalWatchdog" 的報告程式可以監視節點上可用磁碟的狀態，因此可以針對該節點回報 "AvailableDisk" 屬性。 該相同報告程式還可以監視節點連線能力，因此可以針對該相同節點回報 "Connectivity" 屬性。 在健康狀態資料存放區中，會將這些報告視為所指定節點的個別健康情況事件。 與 SourceId 搭配使用時，此屬性可唯一識別健康情況資訊。 |
| --health-state    [必要] | 可能的值包括：'Invalid'、'Ok'、'Warning'、'Error'、'Unknown'。 |
| --source-id       [必要] | 針對產生健康情況資訊的用戶端/看門狗/系統元件，用來識別的來源名稱。 |
| --description | 健康情況資訊的描述。 <br><br> 這代表任意格式的文字，可用來新增人類可讀的報告相關資訊。 描述的字串長度上限為 4096 個字元。 如果提供的字串超出此長度，將會自動截斷。 截斷時，描述的最後字元會包含 "[已截斷]" 標記，而總字串大小會是 4096 個字元。 此標記的存在可向使用者指出已發生截斷情況。 請注意，截斷時，描述的字元數會少於來自原始字串的 4096 個字元。 |
| --immediate | 指出是否應該立即傳送報告的旗標。 <br><br> 健康情況報告會傳送給 Service Fabric 閘道應用程式，再由其轉送給健康狀態資料存放區。 如果將 Immediate 設定為 true，則不論「HTTP 閘道應用程式」使用什麼網狀架構用戶端設定，都會從「HTTP 閘道」立即將報告傳送給健康狀態資料存放區。 這對於應該儘快傳送的重要報告來說，相當有用。 視時機和其他條件而定，傳送報告時仍可能失敗，例如在「HTTP 閘道」關閉或訊息未觸達閘道的情況下。 如果將 Immediate 設定為 false，就會根據來自「HTTP 閘道」的健康情況用戶端設定來傳送報告。 因此，會根據 HealthReportSendInterval 設定進行批次處理。 這是建議的設定，因為這除了可讓健康情況用戶端將健康情況報告處理最佳化之外，也可將傳送給健康狀態資料存放區的健康情況報告訊息最佳化。 預設不會立即傳送報告。 |
| --remove-when-expired | 指出是否要在報告到期時將報告自健康狀態資料存放區中移除的值。 <br><br> 如果設定為 true，就會在報告到期後，將報告自健康狀態資料存放區中移除。 如果設定為 false，則會在報告到期後，將報告視為錯誤。 此屬性的值預設為 false。 當用戶端會定期回報時，應該將 RemoveWhenExpired 設定為 false (預設值)。 如此一來，如果報告程式發生問題 (例如死結) 而無法回報，在健康情況報告到期時，系統就會將實體評估為錯誤。 這會將實體標幟為處於「錯誤」健康情況狀態。 |
| --sequence-number | 此健康情況報告的序號 (以數值字串表示)。 <br><br> 健康狀態資料存放區會使用報告序號來偵測過時的報告。 如果未指定，就會在新增報告時，由健康情況用戶端自動產生序號。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --ttl | 此健康情況報告的有效持續時間。 此欄位使用 ISO8601 格式來指定持續時間。 <br><br> 當用戶端會定期回報時，其傳送報告的頻率應該高於存留時間。 如果用戶端會針對轉換進行回報，則可以將存留時間設定為無限。 如果 RemoveWhenExpired 為 true，當存留時間到期時，系統會將包含健康情況資訊的健康情況事件自健康狀態資料存放區中移除，如果 RemoveWhenExpired 為 false，則會將該事件評估為錯誤。 如果未指定，存留時間會預設為 infinite 值。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-select"></a>sfctl cluster select
連線到 Service Fabric 叢集端點。

如果連線到安全的叢集，請指定憑證 (.crt) 和金鑰檔案 (.key) 的絕對路徑，或包含兩者之單一檔案 (.pem) 的絕對路徑。 請勿指定兩者。 您也可以在連線到安全的叢集時，視需要一併指定 CA 組合檔案的絕對路徑，或是受信任 CA 憑證目錄的絕對路徑。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --endpoint [必要] | 叢集端點 URL，包括連接埠和 HTTP 或 HTTPS 前置詞。 |
| --aad | 使用 Azure Active Directory 進行驗證。 |
| --ca | 要視為有效 CA 憑證目錄的絕對路徑，或 CA 組合檔案的絕對路徑。 |
| --cert | 用戶端憑證檔案的絕對路徑。 |
| --key | 用戶端憑證金鑰檔案的絕對路徑。 |
| --no-verify | 使用 HTTPS 時停用憑證的驗證。請注意：這是不安全的選項，因此不應該用於生產環境。 |
| --pem | 用戶端憑證 (.pem 檔案) 的絕對路徑。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-show-connection"></a>sfctl 叢集 show-connection
顯示這個 sfctl 執行個體連線到哪個 Service Fabric 叢集。

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-unprovision"></a>sfctl cluster unprovision
解除佈建 Service Fabric 叢集的程式碼或組態封裝。

支援將程式碼和設定個別解除佈建。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --code-version | 叢集程式碼封裝版本。 |
| --config-version | 叢集資訊清單版本。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-upgrade"></a>sfctl cluster upgrade
開始升級 Service Fabric 叢集的程式碼或組態版本。

驗證提供的升級參數，如果參數有效，則開始升級 Service Fabric 叢集的程式碼或組態版本。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --app-health-map | 引發錯誤之前，應用程式名稱和健康情況不良最大百分比組合的 JSON 編碼目錄。 |
| --app-type-health-map | 引發錯誤之前，應用程式類型名稱和健康情況不良最大百分比組合的 JSON 編碼目錄。 |
| --code-version | 叢集程式碼版本。 |
| --config-version | 叢集組態版本。 |
| --delta-health-evaluation | 允許在完成每個升級網域之後進行差異健康狀態評估，而不是絕對健康狀態評估。 |
| --delta-unhealthy-nodes | 在叢集升級期間允許的節點健康情況衰退百分比上限。  預設值\:10。 <br><br> 差異是測量升級開始時節點的狀態和健康情況評估時節點的狀態之間的差異。 每次升級網域升級完成後都會執行檢查，以確保叢集的全域狀態處於容許的限制範圍內。 |
| --failure-action | 可能的值包括：'Invalid'、'Rollback'、'Manual'。 |
| --force-restart | 強制重新啟動。 |
| --health-check-retry | 健康情況檢查重試逾時 (毫秒)。 |
| --health-check-stable | 健康情況檢查穩定持續時間 (毫秒)。 |
| --health-check-wait | 健康情況檢查等候持續時間 (毫秒)。 |
| --replica-set-check-timeout | 升級複本設定檢查逾時 (秒)。 |
| --rolling-upgrade-mode | 可能的值包括：'Invalid'、'UnmonitoredAuto'、'UnmonitoredManual'、'Monitored'。  預設值\:UnmonitoredAuto。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --unhealthy-applications | 在報告錯誤之前，允許健康情況不良應用程式的最大百分比。 <br><br> 例如，若要允許 10% 的應用程式健康情況不良，這個值會是 10。 百分比表示在系統將叢集視為錯誤之前，容許應用程式健康情況不良的最大百分比。 如果未到達此百分比，但至少有一個健康情況不良的應用程式，則健康情況會評估為 Warning。 此計算是將健康情況不良的應用程式數目除以叢集中應用程式執行個體的總數而得，但不包括 ApplicationTypeHealthPolicyMap 所包含之應用程式類型的應用程式。 針對少量的應用程式數目，計算會四捨五入以容許一個失敗。 |
| --unhealthy-nodes | 在報告錯誤之前，允許健康情況不良節點的最大百分比。 <br><br> 例如，若要允許 10% 的節點健康情況不良，這個值會是 10。 百分比表示在叢集被視為處於錯誤狀態之前，容許節點健康情況不良的最大百分比。 如果未到達此百分比，但至少有一個健康情況不良的節點，則健康情況會評估為 Warning。 百分比是將健康情況不良節點數目除以叢集中的節點總數計算而得。 針對較少的節點數目，計算會四捨五入以容許一個失敗。 在大型叢集中，永遠都有一些節點會關閉或需要修復，因此應設定此百分比來容許這種情形。 |
| --upgrade-domain-delta-unhealthy-nodes | 在叢集升級期間允許的升級網域節點健康情況衰退百分比上限。  預設值\:15。 <br><br> 差異是測量升級開始時升級網域節點的狀態和健康情況評估時升級網域節點的狀態之間的差異。 每次所有已完成升級網域的升級網域升級完成後都會執行檢查，以確保升級網域的狀態處於容許的限制範圍內。 |
| --upgrade-domain-timeout | 升級網域逾時 (毫秒)。 |
| --upgrade-timeout | 升級逾時 (毫秒)。 |
| --warning-as-error | 具有相同嚴重性的警告會視為錯誤。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-upgrade-resume"></a>sfctl cluster upgrade-resume
將叢集升級移至下一個升級網域。

讓叢集程式碼或設定升級繼續進行至下一個升級網域 (如果適用)。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --upgrade-domain [必要] | 此叢集升級的下一個升級網域。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-upgrade-rollback"></a>sfctl cluster upgrade-rollback
復原 Service Fabric 叢集的升級。

復原 Service Fabric 叢集的程式碼或設定升級。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-upgrade-status"></a>sfctl cluster upgrade-status
取得目前叢集升級的進度。

取得進行中叢集升級的目前進度。 如果目前沒有任何進行中的升級，則取得前一個叢集升級的最後狀態。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-cluster-upgrade-update"></a>sfctl cluster upgrade-update
更新 Service Fabric 叢集升級的升級參數。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --app-health-map | 引發錯誤之前，應用程式名稱和健康情況不良最大百分比組合的 JSON 編碼目錄。 |
| --app-type-health-map | 引發錯誤之前，應用程式類型名稱和健康情況不良最大百分比組合的 JSON 編碼目錄。 |
| --delta-health-evaluation | 允許在完成每個升級網域之後進行差異健康狀態評估，而不是絕對健康狀態評估。 |
| --delta-unhealthy-nodes | 在叢集升級期間允許的節點健康情況衰退百分比上限。  預設值\:10。 <br><br> 差異是測量升級開始時節點的狀態和健康情況評估時節點的狀態之間的差異。 每次升級網域升級完成後都會執行檢查，以確保叢集的全域狀態處於容許的限制範圍內。 |
| --failure-action | 可能的值包括：'Invalid'、'Rollback'、'Manual'。 |
| --force-restart | 強制重新啟動。 |
| --health-check-retry | 健康情況檢查重試逾時 (毫秒)。 |
| --health-check-stable | 健康情況檢查穩定持續時間 (毫秒)。 |
| --health-check-wait | 健康情況檢查等候持續時間 (毫秒)。 |
| --replica-set-check-timeout | 升級複本設定檢查逾時 (秒)。 |
| --rolling-upgrade-mode | 可能的值包括：'Invalid'、'UnmonitoredAuto'、'UnmonitoredManual'、'Monitored'。  預設值\:UnmonitoredAuto。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --unhealthy-applications | 在報告錯誤之前，允許健康情況不良應用程式的最大百分比。 <br><br> 例如，若要允許 10% 的應用程式健康情況不良，這個值會是 10。 百分比表示在系統將叢集視為錯誤之前，容許應用程式健康情況不良的最大百分比。 如果未到達此百分比，但至少有一個健康情況不良的應用程式，則健康情況會評估為 Warning。 此計算是將健康情況不良的應用程式數目除以叢集中應用程式執行個體的總數而得，但不包括 ApplicationTypeHealthPolicyMap 所包含之應用程式類型的應用程式。 針對少量的應用程式數目，計算會四捨五入以容許一個失敗。 |
| --unhealthy-nodes | 在報告錯誤之前，允許健康情況不良節點的最大百分比。 <br><br> 例如，若要允許 10% 的節點健康情況不良，這個值會是 10。 百分比表示在叢集被視為處於錯誤狀態之前，容許節點健康情況不良的最大百分比。 如果未到達此百分比，但至少有一個健康情況不良的節點，則健康情況會評估為 Warning。 百分比是將健康情況不良節點數目除以叢集中的節點總數計算而得。 針對較少的節點數目，計算會四捨五入以容許一個失敗。 在大型叢集中，永遠都有一些節點會關閉或需要修復，因此應設定此百分比來容許這種情形。 |
| --upgrade-domain-delta-unhealthy-nodes | 在叢集升級期間允許的升級網域節點健康情況衰退百分比上限。  預設值\:15。 <br><br> 差異是測量升級開始時升級網域節點的狀態和健康情況評估時升級網域節點的狀態之間的差異。 每次所有已完成升級網域的升級網域升級完成後都會執行檢查，以確保升級網域的狀態處於容許的限制範圍內。 |
| --upgrade-domain-timeout | 升級網域逾時 (毫秒)。 |
| --upgrade-kind | 可能的值包括：'Invalid'、'Rolling'、'Rolling_ForceRestart'。  預設值\:Rolling。 |
| --upgrade-timeout | 升級逾時 (毫秒)。 |
| --warning-as-error | 具有相同嚴重性的警告會視為錯誤。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="next-steps"></a>後續步驟
- [設定](service-fabric-cli.md) Service Fabric CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。