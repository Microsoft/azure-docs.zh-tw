---
title: Azure Service Fabric CLI - sfctl node | Microsoft Docs
description: 描述 Service Fabric CLI sfctl node 命令。
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
ms.openlocfilehash: e68a258c8e323b62f85219648c011ce1e661ee0d
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494533"
---
# <a name="sfctl-node"></a>sfctl node
管理形成叢集的節點。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| disable | 停用有指定停用意圖的 Service Fabric 叢集節點。 |
| enable | 啟用目前停用的 Service Fabric 叢集節點。 |
| health | 取得 Service Fabric 節點的健康情況。 |
| info | 取得 Service Fabric 叢集中特定節點的相關資訊。 |
| list | 取得 Service Fabric 叢集中的節點清單。 |
| load | 取得 Service Fabric 節點的負載資訊。 |
| remove-state | 通知 Service Fabric 某個節點上的永續性狀態已永久移除或遺失。 |
| report-health | 傳送 Service Fabric 節點的健康情況報告。 |
| restart | 重新啟動 Service Fabric 叢集節點。 |
| transition | 啟動或停止叢集節點。 |
| transition-status | 取得使用 StartNodeTransition 啟動的作業進度。 |

## <a name="sfctl-node-disable"></a>sfctl node disable
停用有指定停用意圖的 Service Fabric 叢集節點。

停用有指定停用意圖的 Service Fabric 叢集節點。 一旦正在進行停用，停用意圖便只能增加，而無法降低 (例如，使用 Pause 意圖停用的節點可以進一步使用 Restart 來停用，但無法反向操作)。 節點可在停用之後，隨時使用啟用節點的作業重新啟用。 如果未完成停用，則會取消停用。 節點如果在停用時停止運作後又恢復運作，將仍然需要重新啟用，才能將服務放在該節點上。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --deactivation-intent | 描述停用節點的意圖或理由。 可能的值如下。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-enable"></a>sfctl node enable
啟用目前停用的 Service Fabric 叢集節點。

啟用目前停用的 Service Fabric 叢集節點。 節點在啟用之後，將再次成為可供放置新複本的目標，且留在節點上的所有已停用複本都會重新啟用。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-health"></a>sfctl node health
取得 Service Fabric 節點的健康情況。

取得 Service Fabric 節點的健康情況。 使用 EventsHealthStateFilter 可根據健康情況狀態，篩選針對節點所報告的健康情況事件集合。 如果您依名稱指定的節點不在健康狀態資料存放區中，則會傳回錯誤。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-info"></a>sfctl node info
取得 Service Fabric 叢集中特定節點的相關資訊。

回應包括名稱、狀態、識別碼、健康情況、執行時間，以及與節點有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-list"></a>sfctl node list
取得 Service Fabric 叢集中的節點清單。

回應包括名稱、狀態、識別碼、健康情況、執行時間，以及與節點有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --max-results | 分頁式查詢時傳回的最大結果數目。 此參數定義傳回結果數目的上限。 根據設定中所定義的訊息大小限制上限，如果所傳回的結果無法放入訊息中，則它們可以小於指定的結果上限。 如果此參數為零或未指定，則分頁式查詢會在傳回訊息中盡可能包含較多結果。 |
| --node-status-filter | 可根據 NodeStatus 篩選節點。 只會傳回符合所指定篩選值的節點。 篩選值可以是下列其中之一。  預設值：default。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-load"></a>sfctl node load
取得 Service Fabric 節點的負載資訊。

針對已定義負載或容量的所有計量，擷取 Service Fabric 節點的負載資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-remove-state"></a>sfctl node remove-state
通知 Service Fabric 某個節點上的永續性狀態已永久移除或遺失。

這意謂著無法復原該節點的永續性狀態。 通常如果已將硬碟抹除乾淨或硬碟損毀，就會發生這種情況。 節點必須停止運作，此作業才能成功。 此作業可讓 Service Fabric 知道該節點上的複本已不存在，而 Service Fabric 應該停止等候這些複本恢復運作。 如果節點上的狀態尚未移除，而節點能夠在其狀態保持不變的情況下恢復運作，則請勿執行此 Cmdlet。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-report-health"></a>sfctl node report-health
傳送 Service Fabric 節點的健康情況報告。

回報所指定 Service Fabric 節點的健康情況狀態。 此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道節點，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請檢查此報告是否出現在 HealthEvents 區段中。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --health-property [必要] | 健康情況資訊的屬性。 <br><br> 實體可以有不同屬性的健康情況報告。 屬性是一個字串而不是固定的列舉，以便讓報告程式在分類觸發報告的狀態條件時較有彈性。 例如，SourceId 為 "LocalWatchdog" 的報告程式可以監視節點上可用磁碟的狀態，因此可以針對該節點回報 "AvailableDisk" 屬性。 該相同報告程式還可以監視節點連線能力，因此可以針對該相同節點回報 "Connectivity" 屬性。 在健康狀態資料存放區中，會將這些報告視為所指定節點的個別健康情況事件。 與 SourceId 搭配使用時，此屬性可唯一識別健康情況資訊。 |
| --health-state    [必要] | 可能的值包括：'Invalid'、'Ok'、'Warning'、'Error'、'Unknown'。 |
| --node-name [必要] | 報告所要針對的節點名稱。 |
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

## <a name="sfctl-node-restart"></a>sfctl node restart
重新啟動 Service Fabric 叢集節點。

重新啟動已啟動的 Service Fabric 叢集節點。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --create-fabric-dump | 指定 True 以建立網狀架構節點處理程序的傾印。 這區分大小寫。  預設值：False。 |
| --node-instance-id | 目標節點的執行個體識別碼。 如果指定執行個體識別碼，則只有當識別碼符合節點目前的執行個體時，才會重新啟動節點。 預設值 "0" 會比對出所有執行個體識別碼。 您可以使用取得節點查詢來取得執行個體識別碼。  預設值：0。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-transition"></a>sfctl node transition
啟動或停止叢集節點。

啟動或停止叢集節點。  叢集節點是一種處理程序，而不是作業系統執行個體本身。  若要啟動節點，請為 NodeTransitionType 參數傳入 "Start"。 若要停止節點，請為 NodeTransitionType 參數傳入 "Stop"。 此 API 會在 API 傳回節點可能尚未完成轉換時啟動作業。 呼叫有相同 OperationId 的 GetNodeTransitionProgress 可取得作業的進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-instance-id [必要] | 目標節點的節點執行個體識別碼。 這可透過 GetNodeInfo API 來決定。 |
| --node-name [必要] | 節點的名稱。 |
| --node-transition-type     [必要] | 表示要執行的轉換類型。  NodeTransitionType.Start 會啟動已停止的節點。 NodeTransitionType.Stop 會停止已啟動的節點。 |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --stop-duration-in-seconds [必要] | 讓節點維持停止的持續時間，以秒為單位。  最小值是 600，最大值是 14400。  在這段時間到期後，節點將會自動恢復運作。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-node-transition-status"></a>sfctl node transition-status
取得使用 StartNodeTransition 啟動的作業進度。

針對以 StartNodeTransition 啟動的作業，使用所提供的 OperationId 來取得作業的進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

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