---
title: Azure Service Fabric CLI - sfctl partition | Microsoft Docs
description: 描述 Service Fabric CLI sfctl partition 命令。
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
ms.openlocfilehash: 93478e5d13ef649b86ebc047f4e53f1486e2ff68
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39493948"
---
# <a name="sfctl-partition"></a>sfctl partition
查詢和管理任何服務的資料分割。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| data-loss | 此 API 會引發所指定分割區的資料遺失。 |
| data-loss-status | 針對使用 StartDataLoss API 啟動的分割區資料遺失作業，取得作業的進度。 |
| health | 取得指定 Service Fabric 分割區的健康情況。 |
| info | 取得 Service Fabric 分割區的相關資訊。 |
| list | 取得 Service Fabric 服務的分割區清單。 |
| load | 取得所指定 Service Fabric 分割區的負載資訊。 |
| load-reset | 重設 Service Fabric 分割區目前的負載。 |
| quorum-loss | 導致指定具狀態服務分割區的仲裁遺失。 |
| quorum-loss-status | 針對在分割區上使用 StartQuorumLoss API 啟動的仲裁遺失作業，取得作業的進度。 |
| recover | 表示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失的特定分割區。 |
| recover-all | 指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的所有服務 (包括系統服務)。 |
| report-health | 傳送 Service Fabric 分割區的健康情況報告。 |
| restart | 此 API 會重新啟動所指定分割區的部分或全部複本或執行個體。 |
| restart-status | 針對使用 StartPartitionRestart 啟動的 PartitionRestart 作業，取得作業的進度。 |
| svc-name | 取得分割區的 Service Fabric 服務名稱。 |

## <a name="sfctl-partition-data-loss"></a>sfctl partition data-loss
此 API 會引發所指定分割區的資料遺失。

這會觸發對分割區 OnDataLossAsync API 的呼叫。  此 API 會引發所指定分割區的資料遺失。 這會觸發對分割區 OnDataLoss API 的呼叫。 實際的資料遺失將取決於指定的 DataLossMode。 <br> PartialDataLoss - 只會移除複本仲裁，並且會為分割區觸發 OnDataLoss，但實際資料遺失取決於是否有進行中的複寫存在。 <br>FullDataLoss - 將會移除所有複本，因此會遺失所有資料，並且會觸發 OnDataLoss。 <br>應該只使用具狀態服務作為目標來呼叫此 API。 不建議使用系統服務作為目標來呼叫此 API。 
> [!NOTE]
> 一旦呼叫此 API，即無法回復。 呼叫 CancelOperation 只會停止執行並清除內部系統狀態。 如果命令已進展到足以造成資料遺失的地步，則它將不會還原資料。 您可以使用相同的 OperationId 來呼叫 GetDataLossProgress API，以傳回使用此 API 所啟動作業的相關資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --data-loss-mode [必要] | 此列舉會傳遞給 StartDataLoss API 以指出要引發哪一種類型的資料遺失。 |
| --operation-id   [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id   [必要] | 分割區的識別。 |
| --service-id     [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-data-loss-status"></a>sfctl partition data-loss-status
針對使用 StartDataLoss API 啟動的分割區資料遺失作業，取得作業的進度。

針對使用 StartDataLoss API 啟動的資料遺失作業，以 OperationId 來取得作業的進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id [必要] | 分割區的識別。 |
| --service-id   [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-health"></a>sfctl partition health
取得指定 Service Fabric 分割區的健康情況。

使用 EventsHealthStateFilter 可根據健康情況狀態，篩選針對服務所報告的健康情況事件集合。 使用 ReplicasHealthStateFilter 可篩選分割區上 ReplicaHealthState 物件的集合。 如果您指定的分割區不在健康狀態資料存放區中，此要求就會傳回錯誤。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --exclude-health-statistics | 指出是否應該在查詢結果中一併傳回健康情況統計資料。 預設為 False。 統計資料會顯示健康情況狀態為 Ok、Warning 及 Error 的子實體數目。 |
| --replicas-health-state-filter | 允許篩選分割區上的 ReplicaHealthState 物件集合。 可以從 HealthStateFilter 成員或對這些成員的位元運算取得值。 只會傳回符合篩選條件的複本。 所有複本都會用來評估彙總的健康情況狀態。 若未指定，則會傳回所有項目。狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-info"></a>sfctl partition info
取得 Service Fabric 分割區的相關資訊。

取得所指定分割區的相關資訊。 回應包括分割區識別碼、資料分割配置資訊、分割區支援的索引鍵、狀態、健康情況，以及與分割區有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-list"></a>sfctl partition list
取得 Service Fabric 服務的分割區清單。

回應包括分割區識別碼、資料分割配置資訊、分割區支援的索引鍵、狀態、健康情況，以及與分割區有關的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --service-id [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
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

## <a name="sfctl-partition-load"></a>sfctl partition load
取得所指定 Service Fabric 分割區的負載資訊。

傳回所指定分割區的負載相關資訊。 回應會包含 Service Fabric 分割區的負載報告清單。 每個報告都包含負載計量名稱、值及上次報告時間 (UTC)。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-load-reset"></a>sfctl partition load-reset
重設 Service Fabric 分割區目前的負載。

將 Service Fabric 分割區目前的負載重設為服務的預設負載。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-quorum-loss"></a>sfctl partition quorum-loss
導致指定具狀態服務分割區的仲裁遺失。

此 API 對於您服務上的暫時性仲裁遺失情況來說，相當有用。 您可以使用相同的 OperationId 來呼叫 GetQuorumLossProgress API，以傳回使用此 API 所啟動作業的相關資訊。 只能在具狀態持續性 (HasPersistedState==true) 服務上呼叫此 API。  請勿在無狀態服務或僅限記憶體內的具狀態服務上使用此 API。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id         [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id         [必要] | 分割區的識別。 |
| --quorum-loss-duration [必要] | 分割區將維持在仲裁遺失狀態的時間長度。  必須以秒為單位來指定此值。 |
| --quorum-loss-mode     [必要] | 此列舉會傳遞給 StartQuorumLoss API 以指出要引發哪一種類型的仲裁遺失。 |
| --service-id           [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-quorum-loss-status"></a>sfctl partition quorum-loss-status
針對在分割區上使用 StartQuorumLoss API 啟動的仲裁遺失作業，取得作業的進度。

針對使用 StartQuorumLoss 啟動的仲裁遺失作業，使用所提供的 OperationId 來取得作業的進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id [必要] | 分割區的識別。 |
| --service-id   [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-recover"></a>sfctl partition recover
表示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失的特定分割區。

只有當已知無法復原已關閉的複本時，才能執行這項作業。 不正確使用這個 API，可能會造成資料遺失。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-recover-all"></a>sfctl partition recover-all
指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的所有服務 (包括系統服務)。

只有當已知無法復原已關閉的複本時，才能執行這項作業。 不正確使用這個 API，可能會造成資料遺失。

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

## <a name="sfctl-partition-report-health"></a>sfctl partition report-health
傳送 Service Fabric 分割區的健康情況報告。

回報所指定 Service Fabric 分割區的健康情況狀態。 此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道分割區，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請檢查此報告是否出現在事件區段中。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --health-property [必要] | 健康情況資訊的屬性。 <br><br> 實體可以有不同屬性的健康情況報告。 屬性是一個字串而不是固定的列舉，以便讓報告程式在分類觸發報告的狀態條件時較有彈性。 例如，SourceId 為 "LocalWatchdog" 的報告程式可以監視節點上可用磁碟的狀態，因此可以針對該節點回報 "AvailableDisk" 屬性。 該相同報告程式還可以監視節點連線能力，因此可以針對該相同節點回報 "Connectivity" 屬性。 在健康狀態資料存放區中，會將這些報告視為所指定節點的個別健康情況事件。 與 SourceId 搭配使用時，此屬性可唯一識別健康情況資訊。 |
| --health-state    [必要] | 可能的值包括：'Invalid'、'Ok'、'Warning'、'Error'、'Unknown'。 |
| --partition-id [必要] | 分割區的識別。 |
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

## <a name="sfctl-partition-restart"></a>sfctl partition restart
此 API 會重新啟動所指定分割區的部分或全部複本或執行個體。

此 API 在測試容錯移轉時很有用。 如果用來針對無狀態服務的分割區，RestartPartitionMode 必須是 AllReplicasOrInstances。 使用相同的 OperationId 呼叫 GetPartitionRestartProgress API 可取得進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id [必要] | 分割區的識別。 |
| --restart-partition-mode [必要] | 描述要重新啟動哪個分割區。 |
| --service-id [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-restart-status"></a>sfctl partition restart-status
針對使用 StartPartitionRestart 啟動的 PartitionRestart 作業，取得作業的進度。

針對使用 StartPartitionRestart 啟動的 PartitionRestart，使用所提供的 OperationId 來取得其進度。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --operation-id [必要] | 識別此 API 呼叫的 GUID。  這會傳遞至對應的 GetProgress API。 |
| --partition-id [必要] | 分割區的識別。 |
| --service-id   [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-partition-svc-name"></a>sfctl partition svc-name
取得分割區的 Service Fabric 服務名稱。

取得所指定分割區的服務名稱。 如果叢集中沒有該分割區識別碼，則會傳回 404 錯誤。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --partition-id [必要] | 分割區的識別。 |
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
