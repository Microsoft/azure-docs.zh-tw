---
title: Azure Service Fabric CLI - sfctl replica | Microsoft Docs
description: 描述 Service Fabric CLI sfctl replica 命令。
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
ms.openlocfilehash: 6c16cd95fce7d3f367f0ded73c3635d8cefea7a0
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39493982"
---
# <a name="sfctl-replica"></a>sfctl replica
管理屬於服務分割區的複本。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| deployed | 取得在 Service Fabric 節點上部署的複本詳細資料。 |
| deployed-list | 取得在 Service Fabric 節點上部署的複本清單。 |
| health | 取得 Service Fabric 具狀態服務複本或無狀態服務執行個體的健康情況。 |
| info | 取得 Service Fabric 分割區複本的相關資訊。 |
| list | 取得 Service Fabric 服務分割區複本的相關資訊。 |
| remove | 移除在節點上執行的服務複本。 |
| report-health | 傳送 Service Fabric 複本的健康情況報告。 |
| restart | 重新啟動節點上執行之持續性服務的服務複本。 |

## <a name="sfctl-replica-deployed"></a>sfctl replica deployed
取得在 Service Fabric 節點上部署的複本詳細資料。

取得在 Service Fabric 節點上部署的複本詳細資料。 此資訊包含服務類型、服務名稱、目前的服務作業、目前的服務作業開始日期時間、分割區識別碼、複本/執行個體識別碼、回報的負載，以及其他資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 複本的識別碼。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-replica-deployed-list"></a>sfctl replica deployed-list
取得在 Service Fabric 節點上部署的複本清單。

針對 Service Fabric 節點上所部署的複本，取得包含相關資訊的清單。 此資訊包含分割區識別碼、複本識別碼、複本的狀態、服務的名稱、服務類型的名稱，以及其他資訊。 您可以使用 PartitionId 或 ServiceManifestName 查詢參數，以傳回符合這些參數指定值的已部署複本相關資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name      [必要] | 節點的名稱。 |
| --partition-id | 分割區的識別。 |
| --service-manifest-name | 在 Service Fabric 叢集中，註冊為應用程式類型中一部分的服務資訊清單名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-replica-health"></a>sfctl replica health
取得 Service Fabric 具狀態服務複本或無狀態服務執行個體的健康情況。

取得 Service Fabric 複本的健康情況。 使用 EventsHealthStateFilter 可根據健康情況狀態，篩選針對複本所報告的健康情況事件集合。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 複本的識別碼。 |
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

## <a name="sfctl-replica-info"></a>sfctl replica info
取得 Service Fabric 分割區複本的相關資訊。

回應會包含識別碼、角色、狀態、健康情況、節點名稱、執行時間，以及與複本有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 複本的識別碼。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-replica-list"></a>sfctl replica list
取得 Service Fabric 服務分割區複本的相關資訊。

GetReplicas 端點會傳回指定分割區複本的相關資訊。 回應會包含識別碼、角色、狀態、健康情況、節點名稱、執行時間，以及與複本有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-replica-remove"></a>sfctl replica remove
移除在節點上執行的服務複本。

此 API 會從 Service Fabric 叢集移除複本，來模擬 Service Fabric 複本失敗。 移除作業會關閉複本，將複本轉換成 None 角色，然後從叢集移除複本所有的狀態資訊。 此 API 會測試複本狀態移除路徑，並透過用戶端 API 模擬報告錯誤永久路徑。 警告：使用此 API 時，不會執行任何安全性檢查。 不正確使用此 API 可能會導致具狀態服務遺失資料。 此外，forceRemove 旗標會影響裝載於相同處理序中的所有其他複本。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 複本的識別碼。 |
| --force-remove | 強制移除 Service Fabric 應用程式或服務，而不經過正常關機順序。 當因服務程式碼中有導致無法正常關閉複本的問題，而使得應用程式或服務的刪除逾時時，可使用此參數來強制刪除應用程式或服務。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-replica-report-health"></a>sfctl replica report-health
傳送 Service Fabric 複本的健康情況報告。

回報所指定 Service Fabric 複本的健康情況狀態。 此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道複本，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請檢查此報告是否出現在事件區段中。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --health-property [必要] | 健康情況資訊的屬性。 <br><br> 實體可以有不同屬性的健康情況報告。 屬性是一個字串而不是固定的列舉，以便讓報告程式在分類觸發報告的狀態條件時較有彈性。 例如，SourceId 為 "LocalWatchdog" 的報告程式可以監視節點上可用磁碟的狀態，因此可以針對該節點回報 "AvailableDisk" 屬性。 該相同報告程式還可以監視節點連線能力，因此可以針對該相同節點回報 "Connectivity" 屬性。 在健康狀態資料存放區中，會將這些報告視為所指定節點的個別健康情況事件。 與 SourceId 搭配使用時，此屬性可唯一識別健康情況資訊。 |
| --health-state    [必要] | 可能的值包括：'Invalid'、'Ok'、'Warning'、'Error'、'Unknown'。 |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 分割區的識別。 |
| --source-id       [必要] | 針對產生健康情況資訊的用戶端/看門狗/系統元件，用來識別的來源名稱。 |
| --description | 健康情況資訊的描述。 <br><br> 這代表任意格式的文字，可用來新增人類可讀的報告相關資訊。 描述的字串長度上限為 4096 個字元。 如果提供的字串超出此長度，將會自動截斷。 截斷時，描述的最後字元會包含 "[已截斷]" 標記，而總字串大小會是 4096 個字元。 此標記的存在可向使用者指出已發生截斷情況。 請注意，截斷時，描述的字元數會少於來自原始字串的 4096 個字元。 |
| --immediate | 指出是否應該立即傳送報告的旗標。 <br><br> 健康情況報告會傳送給 Service Fabric 閘道應用程式，再由其轉送給健康狀態資料存放區。 如果將 Immediate 設定為 true，則不論「HTTP 閘道應用程式」使用什麼網狀架構用戶端設定，都會從「HTTP 閘道」立即將報告傳送給健康狀態資料存放區。 這對於應該儘快傳送的重要報告來說，相當有用。 視時機和其他條件而定，傳送報告時仍可能失敗，例如在「HTTP 閘道」關閉或訊息未觸達閘道的情況下。 如果將 Immediate 設定為 false，就會根據來自「HTTP 閘道」的健康情況用戶端設定來傳送報告。 因此，會根據 HealthReportSendInterval 設定進行批次處理。 這是建議的設定，因為這除了可讓健康情況用戶端將健康情況報告處理最佳化之外，也可將傳送給健康狀態資料存放區的健康情況報告訊息最佳化。 預設不會立即傳送報告。 |
| --remove-when-expired | 指出是否要在報告到期時將報告自健康狀態資料存放區中移除的值。 <br><br> 如果設定為 true，就會在報告到期後，將報告自健康狀態資料存放區中移除。 如果設定為 false，則會在報告到期後，將報告視為錯誤。 此屬性的值預設為 false。 當用戶端會定期回報時，應該將 RemoveWhenExpired 設定為 false (預設值)。 如此一來，如果報告程式發生問題 (例如死結) 而無法回報，在健康情況報告到期時，系統就會將實體評估為錯誤。 這會將實體標幟為處於「錯誤」健康情況狀態。 |
| --sequence-number | 此健康情況報告的序號 (以數值字串表示)。 <br><br> 健康狀態資料存放區會使用報告序號來偵測過時的報告。 如果未指定，就會在新增報告時，由健康情況用戶端自動產生序號。 |
| --service-kind | 正在回報健康情況的服務複本類型 (無狀態或具狀態)。 以下是可能的值：'Stateless'、'Stateful'。  預設值：Stateful。 |
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

## <a name="sfctl-replica-restart"></a>sfctl replica restart
重新啟動節點上執行之持續性服務的服務複本。

重新啟動節點上執行之持續性服務的服務複本。 警告：使用此 API 時，不會執行任何安全性檢查。 不正確使用此 API 可能會導致具狀態服務無法使用。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --partition-id [必要] | 分割區的識別。 |
| --replica-id [必要] | 複本的識別碼。 |
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
