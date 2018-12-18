---
title: Azure Service Fabric CLI- sfctl application| Microsoft Docs
description: 描述 Service Fabric CLI sfctl application 命令。
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
ms.openlocfilehash: 40ec204f105b32c8b7d9e2dda6f6f3c3023b2d44
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495453"
---
# <a name="sfctl-application"></a>sfctl application
建立、刪除與管理應用程式和應用程式類型。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| create | 使用指定的描述來建立 Service Fabric 應用程式。 |
| delete | 刪除現有 Service Fabric 應用程式。 |
| deployed | 取得 Service Fabric 節點上所部署應用程式的相關資訊。 |
| deployed-health | 取得 Service Fabric 節點上所部署應用程式健康情況的相關資訊。 |
| deployed-list | 取得 Service Fabric 節點上所部署的應用程式清單。 |
| health | 取得 Service Fabric 應用程式的健康情況。 |
| info | 取得 Service Fabric 應用程式的相關資訊。 |
| list | 取得在 Service Fabric 叢集中建立且符合所指定篩選條件的應用程式清單。 |
| load | 取得 Service Fabric 應用程式的載入資訊。 |
| manifest | 取得可描述應用程式類型的資訊清單。 |
| provision | 使用外部存放區中的 .sfpkg 套件或使用映像存放區中的應用程式套件，向叢集佈建或註冊 Service Fabric 應用程式類型。 |
| report-health | 傳送 Service Fabric 應用程式的健康情況報告。 |
| type | 取得 Service Fabric 叢集中完全符合指定名稱的應用程式類型清單。 |
| type-list | 取得 Service Fabric 叢集中的應用程式類型清單。 |
| unprovision | 從叢集移除或取消註冊 Service Fabric 應用程式類型。 |
| 升級 | 開始在 Service Fabric 叢集中升級應用程式。 |
| upgrade-resume | 繼續在 Service Fabric 叢集中升級應用程式。 |
| upgrade-rollback | 開始復原目前正在 Service Fabric 叢集中進行的應用程式升級。 |
| upgrade-status | 取得在這個應用程式上執行之最新升級的詳細資料。 |
| upload | 將 Service Fabric 應用程式套件複製到映像存放區。 |

## <a name="sfctl-application-create"></a>sfctl application create
使用指定的描述來建立 Service Fabric 應用程式。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --app-name    [必要] | 應用程式的名稱，包含 'fabric\:' URI 配置。 |
| --app-type    [必要] | 在應用程式資訊清單中找到的應用程式類型名稱。 |
| --app-version [必要] | 應用程式資訊清單中所定義的應用程式類型版本。 |
| --max-node-count | Service Fabric 將為此應用程式保留容量的節點數目上限。 請注意，這並不表示會將此應用程式的服務放在所有這些節點上。 |
| --metrics | 應用程式容量計量描述的 JSON 編碼清單。 計量就定義而言是一個名稱，與應用程式所在每個節點的一組功能相關聯。 |
| --min-node-count | Service Fabric 將為此應用程式保留容量的節點數目下限。 請注意，這並不表示會將此應用程式的服務放在所有這些節點上。 |
| --parameters | 要在建立應用程式時套用的應用程式參數覆寫 JSON 編碼清單。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-delete"></a>sfctl application delete
刪除現有 Service Fabric 應用程式。

必須先建立應用程式，才能將它刪除。 刪除應用程式時，會刪除屬於該應用程式的所有服務。 Service Fabric 預設會嘗試以正常方式關閉服務複本，然後再刪除該服務。 不過，如果服務在以正常方式關閉複本時發生問題，則刪除作業可能會花費很長的時間或停滯。 使用選擇性 ForceRemove 旗標來略過正常關閉順序，並強制刪除應用程式和其所有服務。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
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

## <a name="sfctl-application-deployed"></a>sfctl application deployed
取得 Service Fabric 節點上所部署應用程式的相關資訊。

如果所提供的應用程式識別碼適用於系統應用程式，則此查詢會傳回系統應用程式資訊。 結果包含作用中、正在啟用以及下載中狀態的已部署應用程式。 要使用這項查詢，節點名稱就必須對應到叢集上的節點。 如果所提供的節點名稱未指向叢集上任何作用中的 Service Fabric 節點，則查詢會失敗。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name      [必要] | 節點的名稱。 |
| --include-health-state | 包含實體的健康情況狀態。 如果此參數為 false 或未指定，則傳回的健康情況狀態會是 "Unknown"。 設定為 true 時，會先將查詢以平行方式傳送到節點和健康情況系統服務，然後再合併結果。 因此，查詢會較耗費資源且可能花費較長的時間。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-deployed-health"></a>sfctl application deployed-health
取得 Service Fabric 節點上所部署應用程式健康情況的相關資訊。

取得 Service Fabric 節點上所部署應用程式健康情況的相關資訊。 使用 EventsHealthStateFilter 可視需要根據健康情況狀態，篩選針對所部署應用程式回報的 HealthEvent 物件集合。 使用 DeployedServicePackagesHealthStateFilter 可視需要根據健康情況狀態，篩選 DeployedServicePackageHealth 子系。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id                     [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name                          [必要] | 節點的名稱。 |
| --deployed-service-packages-health-state-filter | 允許根據健康情況狀態，篩選已部署應用程式健康情況查詢結果中所傳回的已部署服務套件健康情況狀態物件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的已部署服務套件。 所有已部署服務套件都會用來評估已部署應用程式的彙總健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的服務套件健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --exclude-health-statistics | 指出是否應該在查詢結果中一併傳回健康情況統計資料。 預設為 False。 統計資料會顯示健康情況狀態為 Ok、Warning 及 Error 的子實體數目。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-deployed-list"></a>sfctl application deployed-list
取得 Service Fabric 節點上所部署的應用程式清單。

取得 Service Fabric 節點上所部署的應用程式清單。 結果不會包含所部署系統應用程式的相關資訊，除非依識別碼明確進行查詢。 結果包含作用中、正在啟用以及下載中狀態的已部署應用程式。 要使用這項查詢，節點名稱就必須對應到叢集上的節點。 如果所提供的節點名稱未指向叢集上任何作用中的 Service Fabric 節點，則查詢會失敗。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --node-name [必要] | 節點的名稱。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --include-health-state | 包含實體的健康情況狀態。 如果此參數為 false 或未指定，則傳回的健康情況狀態會是 "Unknown"。 設定為 true 時，會先將查詢以平行方式傳送到節點和健康情況系統服務，然後再合併結果。 因此，查詢會較耗費資源且可能花費較長的時間。 |
| --max-results | 分頁式查詢時傳回的最大結果數目。 此參數定義傳回結果數目的上限。 根據設定中所定義的訊息大小限制上限，如果所傳回的結果無法放入訊息中，則它們可以小於指定的結果上限。 如果此參數為零或未指定，則分頁式查詢會在傳回訊息中盡可能包含較多結果。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-health"></a>sfctl application health
取得 Service Fabric 應用程式的健康情況。

傳回 Service Fabric 應用程式的健康情況狀態。 回應會報告 Ok、Error 或 Warning 健康情況狀態。 如果在健康狀態資料存放區中找不到實體，則會傳回 Error。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id                 [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --deployed-applications-health-state-filter | 可根據健康情況狀態來篩選應用程式健康情況查詢結果中所傳回的已部署應用程式健康情況狀態物件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的已部署應用程式。 所有已部署應用程式都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的已部署應用程式健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --exclude-health-statistics | 指出是否應該在查詢結果中一併傳回健康情況統計資料。 預設為 False。 統計資料會顯示健康情況狀態為 Ok、Warning 及 Error 的子實體數目。 |
| --services-health-state-filter | 可根據健康情況狀態來篩選服務健康情況查詢結果中所傳回的服務健康情況狀態物件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的服務。 所有服務都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的服務健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-info"></a>sfctl application info
取得 Service Fabric 應用程式的相關資訊。

傳回 Service Fabric 叢集中已建立或正在建立且名稱符合指定為參數之名稱的應用程式相關資訊。 回應包括名稱、類型、狀態、參數，以及應用程式的其他詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id      [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --exclude-application-parameters | 指定是否將從結果中排除應用程式參數的旗標。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-list"></a>sfctl application list
取得在 Service Fabric 叢集中建立且符合所指定篩選條件的應用程式清單。

取得 Service Fabric 叢集中已建立或正在建立且符合指定篩選條件的應用程式相關資訊。 回應包括名稱、類型、狀態、參數，以及應用程式的其他詳細資料。 如果應用程式無法放入一頁，則會傳回一頁結果，以及可用來取得下一頁的接續權杖。 無法同時指定篩選條件 ApplicationTypeName 和 ApplicationDefinitionKindFilter。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-definition-kind-filter | 用來依據 ApplicationDefinitionKind 進行篩選，這是用來定義 Service Fabric 應用程式的機制。  <br> - Default - 預設值，所執行的功能和選取 "All" 時相同。 值為 0。  <br> - All - 符合具有任何 ApplicationDefinitionKind 值之輸入的篩選條件。 值為 65535。  <br> - ServiceFabricApplicationDescription - 符合具有 ApplicationDefinitionKind 值 ServiceFabricApplicationDescription 之輸入的篩選條件。 值為 1。  <br> - Compose - 符合具有 ApplicationDefinitionKind 值 Compose 之輸入的篩選條件。 值為 2。 |
| --application-type-name | 用來篩選要查詢之應用程式的應用程式類型名稱。 此值不應該包含應用程式類型版本。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --exclude-application-parameters | 指定是否將從結果中排除應用程式參數的旗標。 |
| --max-results | 分頁式查詢時傳回的最大結果數目。 此參數定義傳回結果數目的上限。 根據設定中所定義的訊息大小限制上限，如果所傳回的結果無法放入訊息中，則它們可以小於指定的結果上限。 如果此參數為零或未指定，則分頁式查詢會在傳回訊息中盡可能包含較多結果。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-load"></a>sfctl application load
取得 Service Fabric 應用程式的載入資訊。

傳回 Service Fabric 叢集中已建立或正在建立且名稱符合指定為參數之名稱的應用程式相關負載資訊。 回應會包含名稱、最小節點數、最大節點數、應用程式目前佔用的節點數，以及應用程式的相關應用程式負載計量資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-manifest"></a>sfctl application manifest
取得可描述應用程式類型的資訊清單。

回應會將應用程式資訊清單 XML 包含為字串。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-type-name    [必要] | 應用程式類型的名稱。 |
| --application-type-version [必要] | 應用程式類型的版本。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-provision"></a>sfctl application provision
使用外部存放區中的 .sfpkg 套件或使用映像存放區中的應用程式套件，向叢集佈建或註冊 Service Fabric 應用程式類型。

向叢集佈建 Service Fabric 應用程式類型。 需要有這個項目，才能具現化任何新的應用程式。 佈建作業可以在 relativePathInImageStore 所指定的應用程式套件上執行，也可以使用外部 .sfpkg 的 URI 來執行。 除非已設定 --external-provision，否則此命令將會預期要有映像存放區佈建。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-package-download-uri | '.sfpkg' 應用程式套件的路徑，使用 HTTP 或 HTTPS 通訊協定可以從這個位置下載應用程式套件。 <br><br> 僅適用於佈建類型外部存放區。 應用程式套件可以儲存在提供 GET 作業來下載檔案的外部存放區中。 支援的通訊協定為 HTTP 和 HTTPS，而且路徑必須允許 READ 存取。 |
| --application-type-build-path | 僅適用於佈建類型映像存放區。 先前上傳作業期間所指定映像存放區中的應用程式套件相對路徑。 |
| --application-type-name | 僅適用於佈建類型外部存放區。 應用程式類型名稱代表在應用程式資訊清單中所找到應用程式類型的名稱。 |
| --application-type-version | 僅適用於佈建類型外部存放區。 應用程式類型版本代表在應用程式資訊清單中找到之應用程式類型的版本。 |
| --external-provision | 可供註冊或佈建應用程式套件的位置。 表示佈建是適用於先前上傳到外部存放區的應用程式套件。 此應用程式套件的結尾是 *.sfpkg 副檔名。 |
| --no-wait | 表示佈建是否應該以非同步方式發生。 <br><br> 設為 true 時，如果系統接受要求，佈建作業就會回歸，且佈建作業會繼續而沒有任何逾時限制。 預設值為 False。 針對大型應用程式套件，建議您將此值設定為 true。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-report-health"></a>sfctl application report-health
傳送 Service Fabric 應用程式的健康情況報告。

回報所指定 Service Fabric 應用程式的健康情況狀態。 此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道應用程式，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請取得應用程式健康情況，並檢查是否有此報告。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id  [必要] | 應用程式的身分識別。 <br><br> 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 '\~' 字元分隔。 例如，如果應用程式名稱是 'fabric\:/myapp/app1'，則應用程式識別在 6.0+ 中會是 'myapp\~app1'，而在舊版中會是 'myapp/app1'。 |
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

## <a name="sfctl-application-type"></a>sfctl application type
取得 Service Fabric 叢集中完全符合指定名稱的應用程式類型清單。

傳回 Service Fabric 叢集中已佈建或正在佈建之應用程式類型的相關資訊。 這些結果會是名稱完全符合指定為參數之名稱且符合所指定查詢參數的應用程式類型。 會傳回符合應用程式類型名稱之應用程式類型的所有版本，而且每一個版本都會傳回為一個應用程式類型。 回應包括名稱、版本、狀態，以及應用程式類型的其他詳細資料。 這是分頁式查詢，表示如果並非所有應用程式類型都可以放入一頁，則會傳回一頁結果，以及可用來取得下一頁的接續權杖。 例如，如果有 10 個應用程式類型，但一頁只能容納前 3 個應用程式類型，或結果數目上限已設定為 3，則會傳回 3 個。 若要存取其餘的結果，請在下一個查詢中使用所傳回的接續權杖來取出後續頁面。 如果沒有後續頁面，則會傳回空的接續權杖。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-type-name [必要] | 應用程式類型的名稱。 |
| --application-type-version | 應用程式類型的版本。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --exclude-application-parameters | 指定是否將從結果中排除應用程式參數的旗標。 |
| --max-results | 分頁式查詢時傳回的最大結果數目。 此參數定義傳回結果數目的上限。 根據設定中所定義的訊息大小限制上限，如果所傳回的結果無法放入訊息中，則它們可以小於指定的結果上限。 如果此參數為零或未指定，則分頁式查詢會在傳回訊息中盡可能包含較多結果。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-type-list"></a>sfctl application type-list
取得 Service Fabric 叢集中的應用程式類型清單。

傳回 Service Fabric 叢集中已佈建或正在佈建之應用程式類型的相關資訊。 每個應用程式類型版本都會傳回成一個應用程式類型。 回應包括名稱、版本、狀態，以及應用程式類型的其他詳細資料。 這是分頁式查詢，表示如果並非所有應用程式類型都可以放入一頁，則會傳回一頁結果，以及可用來取得下一頁的接續權杖。 例如，如果有 10 個應用程式類型，但一頁只能容納前 3 個應用程式類型，或結果數目上限已設定為 3，則會傳回 3 個。 若要存取其餘的結果，請在下一個查詢中使用所傳回的接續權杖來取出後續頁面。 如果沒有後續頁面，則會傳回空的接續權杖。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-type-definition-kind-filter | 用來依據 ApplicationTypeDefinitionKind 進行篩選，這是用來定義 Service Fabric 應用程式類型的機制。  <br> - Default - 預設值，所執行的功能和選取 "All" 時相同。 值為 0。  <br> - All - 比對輸入具有任何 ApplicationTypeDefinitionKind 值的篩選條件。 值為 65535。  <br> - ServiceFabricApplicationPackage - 比對輸入具有 ApplicationTypeDefinitionKind 值 ServiceFabricApplicationPackage 的篩選條件。 值為 1。  <br> - Compose - 比對輸入具有 ApplicationTypeDefinitionKind 值 Compose 的篩選條件。 值為 2。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --exclude-application-parameters | 指定是否將從結果中排除應用程式參數的旗標。 |
| --max-results | 分頁式查詢時傳回的最大結果數目。 此參數定義傳回結果數目的上限。 根據設定中所定義的訊息大小限制上限，如果所傳回的結果無法放入訊息中，則它們可以小於指定的結果上限。 如果此參數為零或未指定，則分頁式查詢會在傳回訊息中盡可能包含較多結果。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-unprovision"></a>sfctl application unprovision
從叢集移除或取消註冊 Service Fabric 應用程式類型。

只有在已刪除應用程式類型的所有應用程式執行個體時，才能執行這項作業。 將應用程式類型取消註冊之後，就無法為此特定應用程式類型建立新的應用程式執行個體。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-type-name    [必要] | 應用程式類型的名稱。 |
| --application-type-version [必要] | 應用程式資訊清單中所定義的應用程式類型版本。 |
| --async-parameter | 指出解除佈建是否應該以非同步方式發生的旗標。 設為 true 時，如果系統接受要求，解除佈建作業就會回歸，且解除佈建作業會繼續而沒有任何逾時限制。 預設值為 False。 不過，針對已佈建的大型應用程式套件，建議您將它設定為 true。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-upgrade"></a>sfctl application upgrade
開始在 Service Fabric 叢集中升級應用程式。

驗證提供的應用程式升級參數，並在參數有效時開始升級應用程式。 請注意，升級描述會取代現有的應用程式描述。 這表示，如果未指定參數，則會將應用程式上的現有參數覆寫為空的參數清單。 這會導致應用程式使用應用程式資訊清單中參數的預設值。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id      [必要] | 應用程式的身分識別。 <br><br> 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 '\~' 字元分隔。 例如，如果應用程式名稱是 'fabric\:/myapp/app1'，則應用程式識別在 6.0+ 中會是 'myapp\~app1'，而在舊版中會是 'myapp/app1'。 |
| --application-version [必要] | 目標應用程式版本。 |
| --parameters          [必要] | 要在升級應用程式時套用的應用程式參數覆寫 JSON 編碼清單。 |
| --default-service-health-policy | 預設用來評估服務類型健康情況的健康原則 JSON 編碼規格。 |
| --failure-action | 受監視的升級遇到監視原則或健康原則違規時，所要執行的動作。 |
| --force-restart | 即使程式碼版本尚未變更，仍然在升級期間強制重新啟動處理序。 |
| --health-check-retry-timeout | 當應用程式或叢集狀況不良時，在執行失敗動作之前，重試健康情況評估的時間長度。 測量單位：秒。  預設值：PT0H10M0S。 |
| --health-check-stable-duration | 在升級繼續進入下一個升級網域之前，應用程式或叢集必須維持狀況良好的時間長度。 測量單位：秒。  預設值：PT0H2M0S。 |
| --health-check-wait-duration | 完成升級網域之後，在套用健康原則之前，要等待的時間長度。 測量單位：秒。  預設值：0。 |
| --max-unhealthy-apps | 允許的狀況不良已部署應用程式百分比上限。 以介於 0 到 100 之間的數字來表示。 |
| --mode | 在輪流升級期間用來監視健康情況的模式。  預設值\:UnmonitoredAuto。 |
| --replica-set-check-timeout | 發生非預期問題時，封鎖處理升級網域並防止遺失可用性的時間長度上限。 測量單位：秒。 |
| --service-health-policy | 每個服務類型名稱相對於服務類型健康原則的 JSON 編碼對應。 對應預設是空的。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |
| --upgrade-domain-timeout | 執行 FailureAction 之前，每個升級網域必須完成的時間長度。 測量單位：秒。  預設值：P10675199DT02H48M05.4775807S。 |
| --upgrade-timeout | 執行 FailureAction 之前，整體升級必須完成的時間長度。 測量單位：秒。  預設值：P10675199DT02H48M05.4775807S。 |
| --warning-as-error | 將健康情況評估警告的嚴重性視為與錯誤相同。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-upgrade-resume"></a>sfctl application upgrade-resume
繼續在 Service Fabric 叢集中升級應用程式。

繼續執行未受監視的手動 Service Fabric 應用程式升級。 Service Fabric 升級會一次升級一個升級網域。 針對未受監視的手動升級，Service Fabric 會在完成一個升級網域之後，等候您呼叫此 API，再繼續進行下一個升級網域。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id      [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --upgrade-domain-name [必要] | 要在其中繼續執行升級的升級網域名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-upgrade-rollback"></a>sfctl application upgrade-rollback
開始復原目前正在 Service Fabric 叢集中進行的應用程式升級。

開始將目前的應用程式升級復原至前一個版本。 此 API 只能用來復原目前正在向前復原至新版本的升級。 如果目前沒有在升級應用程式，請使用 StartApplicationUpgrade API 將它升級至所需的版本，包括復原至前一個版本。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-upgrade-status"></a>sfctl application upgrade-status
取得在這個應用程式上執行之最新升級的詳細資料。

傳回最新應用程式升級狀態的相關資訊，以及有助於進行應用程式健康情況問題偵錯的詳細資料。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-application-upload"></a>sfctl application upload
將 Service Fabric 應用程式套件複製到映像存放區。

選擇性地顯示套件中每個檔案的上傳進度。 上傳進度會傳送至 `stderr`。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --path   [必要] | 本機應用程式套件的路徑。 |
| --imagestore-string | 將應用程式套件上傳至其中的目的地映像存放區。  預設值：fabric\:ImageStore。 |
| --show-progress | 顯示大型套件的檔案上傳進度。 |

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
