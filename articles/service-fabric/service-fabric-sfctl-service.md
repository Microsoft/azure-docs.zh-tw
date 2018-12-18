---
title: Azure Service Fabric CLI- sfctl service | Microsoft Docs
description: 描述 Service Fabric CLI sfctl service 命令。
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
ms.openlocfilehash: 84c2faaf137e19d78e7e17527feb50baebf8041b
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494569"
---
# <a name="sfctl-service"></a>sfctl service
建立、刪除與管理服務、服務類型和服務套件。

## <a name="commands"></a>命令

|命令|說明|
| --- | --- |
| app-name | 取得服務的 Service Fabric 應用程式名稱。 |
| code-package-list | 取得 Service Fabric 節點上所部署的程式碼套件清單。 |
| create | 建立指定的 Service Fabric 服務。 |
| delete | 刪除現有 Service Fabric 服務。 |
| deployed-type | 針對 Service Fabric 叢集節點上所部署的應用程式，取得其指定服務類型的相關資訊。 |
| deployed-type-list | 從 Service Fabric 叢集節點上所部署的應用程式，取得包含服務類型相關資訊的清單。 |
| 說明 | 取得現有 Service Fabric 服務的描述。 |
| get-container-logs | 取得在 Service Fabric 節點上部署之容器的容器記錄。 |
| health | 取得所指定 Service Fabric 服務的健康情況。 |
| info | 取得屬於 Service Fabric 應用程式的特定服務相關資訊。 |
| list | 取得屬於應用程式識別碼所指定應用程式之所有服務的相關資訊。 |
| manifest | 取得可描述服務類型的資訊清單。 |
| package-deploy | 將與所指定服務資訊清單相關聯的套件下載至所指定節點上的映像快取。 |
| package-health | 針對 Service Fabric 節點和應用程式所部署的特定應用程式，取得服務套件健康情況的相關資訊。 |
| package-info | 取得 Service Fabric 節點上所部署且完全符合指定名稱的服務套件清單。 |
| package-list | 取得 Service Fabric 節點上所部署的服務套件清單。 |
| recover | 指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的指定服務。 |
| report-health | 傳送 Service Fabric 服務的健康情況報告。 |
| resolve | 解析 Service Fabric 資料分割。 |
| type-list | 針對 Service Fabric 叢集中已佈建應用程式類型所支援的服務類型，取得包含服務類型相關資訊的清單。 |
| update | 使用指定的更新描述來更新指定的服務。 |

## <a name="sfctl-service-app-name"></a>sfctl service app-name
取得服務的 Service Fabric 應用程式名稱。

取得所指定服務的應用程式名稱。 如果沒有具備所提供服務識別碼的服務存在，就會傳回 404 FABRIC_E_SERVICE_DOES_NOT_EXIST 錯誤。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
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

## <a name="sfctl-service-code-package-list"></a>sfctl service code-package-list
取得 Service Fabric 節點上所部署的程式碼套件清單。

取得 Service Fabric 節點上針對指定應用程式部署的程式碼套件清單。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name      [必要] | 節點的名稱。 |
| --code-package-name | 服務資訊清單中所指定的程式碼套件名稱，此資訊清單已註冊為 Service Fabric 叢集中應用程式類型的一部分。 |
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

## <a name="sfctl-service-create"></a>sfctl service create
建立指定的 Service Fabric 服務。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --app-id       [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 '\~' 字元分隔。 例如，如果應用程式名稱是 'fabric\:/myapp/app1'，則應用程式識別在 6.0+ 中會是 'myapp\~app1'，而在舊版中會是 'myapp/app1'。 |
| --name         [必要] | 服務的名稱。 這應該是應用程式識別碼的子系。這是包含 `fabric\:` URI 的完整名稱。 例如，服務 `fabric\:/A/B` 是應用程式 `fabric\:/A` 的子系。 |
| --service-type [必要] | 服務類型的名稱。 |
| --activation-mode | 服務套件的啟用模式。 |
| --constraints | 以字串表示的放置限制式。 放置條件約束是節點屬性上的布林運算式，可允許根據服務需求將服務限制在特定節點。 例如，若要在 NodeType 為 blue 的節點上放置服務，請指定："NodeColor == blue"。 |
| --correlated-service | 要與之建立相互關聯的目標服務名稱。 |
| --correlation | 使用對齊親和性將服務與現有服務相互關聯。 |
| --dns-name | 要建立之服務的 DNS 名稱。 必須啟用 Service Fabric DNS 系統服務，才能進行此設定。 |
| --instance-count | 執行個體計數。 這只適用於無狀態服務。 |
| --int-scheme | 指出應該在某範圍的不帶正負號整數之間一致地分割服務。 |
| --int-scheme-count | 使用統一整數分割區配置時，整數索引鍵範圍內要建立的分割區數目。 |
| --int-scheme-high | 使用統一整數分割區配置時，索引鍵整數範圍的結尾。 |
| --int-scheme-low | 使用統一整數分割區配置時，索引鍵整數範圍的開頭。 |
| --load-metrics | 在節點之間進行服務負載平衡時，所使用的計量 JSON 編碼清單。 |
| --min-replica-set-size | 以數字表示的最小複本集大小。 這只適用於具狀態服務。 |
| --move-cost | 指定服務的移動成本。 可能的值為：'Zero'、'Low'、'Medium'、'High'。 |
| --named-scheme | 指出服務應該有多個具名資料分割。 |
| --named-scheme-list | 使用具名分割區配置時，要在其間分割服務的名稱 JSON 編碼清單。 |
| --no-persisted-state | 如果為 true，即表示服務不會在本機磁碟上儲存任何持續性狀態，或只會將狀態儲存在記憶體中。 |
| --placement-policy-list | 服務的放置原則 JSON 編碼清單，以及任何相關聯的網域名稱。 原則可以是下列其中一或多個原則：`NonPartiallyPlaceService`、`PreferPrimaryDomain`、`RequireDomain`、`RequireDomainDistribution`。 |
| --quorum-loss-wait | 允許分割區處於仲裁遺失狀態的持續時間上限 (秒)。 這只適用於具狀態服務。 |
| --replica-restart-wait | 關閉複本與建立新複本之間的持續時間 (秒)。 這只適用於具狀態服務。 |
| --scaling-policies | 此服務的規模調整原則 JSON 編碼清單。 |
| --singleton-scheme | 指出服務應該有單一分割區，或應該是非分割服務。 |
| --stand-by-replica-keep | StandBy 複本在被移除前的保留持續時間上限 (秒)。 這只適用於具狀態服務。 |
| --stateful | 指出服務是具狀態服務。 |
| --stateless | 指出服務是具狀態服務。 |
| --target-replica-set-size | 以數字表示的目標複本集大小。 這只適用於具狀態服務。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-delete"></a>sfctl service delete
刪除現有 Service Fabric 服務。

必須先建立服務，才能將它刪除。 Service Fabric 預設會嘗試以正常方式關閉服務複本，然後再刪除該服務。 不過，如果服務在以正常方式關閉複本時發生問題，則刪除作業可能會花費很長的時間或停滯。 使用選擇性 ForceRemove 旗標來略過正常關閉順序，並強制刪除服務。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --service-id [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
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

## <a name="sfctl-service-deployed-type"></a>sfctl service deployed-type
針對 Service Fabric 叢集節點上所部署的應用程式，取得其指定服務類型的相關資訊。

從 Service Fabric 叢集節點上所部署的應用程式，取得包含特定服務類型相關資訊的清單。 回應會包含服務類型的名稱、其註冊狀態、註冊它的程式碼套件，以及服務套件的啟用識別碼。 每個項目皆代表一個服務類型啟用，以啟用識別碼來區分。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id    [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name         [必要] | 節點的名稱。 |
| --service-type-name [必要] | 指定 Service Fabric 服務類型的名稱。 |
| --service-manifest-name | 用來篩選所部署服務類型資訊清單的服務資訊清單名稱。 如果指定，回應將只會包含此服務資訊清單中所定義服務類型的相關資訊。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-deployed-type-list"></a>sfctl service deployed-type-list
從 Service Fabric 叢集節點上所部署的應用程式，取得包含服務類型相關資訊的清單。

從 Service Fabric 叢集節點上所部署的應用程式，取得包含服務類型相關資訊的清單。 回應會包含服務類型的名稱、其註冊狀態、註冊它的程式碼套件，以及服務套件的啟用識別碼。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name      [必要] | 節點的名稱。 |
| --service-manifest-name | 用來篩選所部署服務類型資訊清單的服務資訊清單名稱。 如果指定，回應將只會包含此服務資訊清單中所定義服務類型的相關資訊。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-description"></a>sfctl service description
取得現有 Service Fabric 服務的描述。

取得現有 Service Fabric 服務的描述。 必須先建立服務，才能取得其描述。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
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

## <a name="sfctl-service-get-container-logs"></a>sfctl service get-container-logs
取得在 Service Fabric 節點上部署之容器的容器記錄。

針對指定程式碼套件，取得 Service Fabric 節點上所部署容器的容器記錄。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id        [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --code-package-name     [必要] | 服務資訊清單中所指定的程式碼套件名稱，此資訊清單已註冊為 Service Fabric 叢集中應用程式類型的一部分。 |
| --node-name             [必要] | 節點的名稱。 |
| --service-manifest-name [必要] | 在 Service Fabric 叢集中，註冊為應用程式類型中一部分的服務資訊清單名稱。 |
| --previous | 指定是否要從程式碼套件執行個體的已結束/無作用容器取得容器記錄。 |
| --tail | 要從記錄結尾顯示的行數。 預設值為 100。 'all' 表示顯示完整記錄。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-health"></a>sfctl service health
取得所指定 Service Fabric 服務的健康情況。

取得所指定服務的健康情況資訊。 使用 EventsHealthStateFilter 可根據健康情況狀態，篩選針對服務所報告的健康情況事件集合。 使用 PartitionsHealthStateFilter 來篩選所傳回資料分割的集合。 如果您指定的服務不在健康狀態資料存放區中，此要求就會傳回錯誤。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --service-id          [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --events-health-state-filter | 可根據健康情況狀態來篩選所傳回的 HealthEvent 物件集合。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的事件。 所有事件都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因而此值可以是使用位元 'OR' 運算子所取得這些值的組合。 例如，如果提供的值為 6，則會傳回 HealthState 值為 OK (2) 和 Warning (4) 的所有事件。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --exclude-health-statistics | 指出是否應該在查詢結果中一併傳回健康情況統計資料。 預設為 False。 統計資料會顯示健康情況狀態為 Ok、Warning 及 Error 的子實體數目。 |
| --partitions-health-state-filter | 允許根據健康情況狀態，篩選服務健康情況查詢結果中所傳回的分割區健康情況狀態物件。 此參數的可能值包括下列其中一個健康情況狀態的整數值。 只會傳回符合篩選條件的分割區。 所有分割區都會用來評估彙總的健康情況狀態。 如果未指定，則會傳回所有項目。 狀態值是以旗標為基礎的列舉，因此值可以是使用位元 'OR' 運算子取得的這些值的組合。 例如，如果提供的值為 6，將會傳回 HealthState 值為 OK (2) 和 Warning (4) 的分割區健康情況狀態。  <br> - Default - 預設值。 符合任何 HealthState。 值為零。  <br> - None - 不符合任何 HealthState 值的篩選條件。 用來在指定狀態集合沒有任何結果時傳回。 值為 1。  <br> - Ok - 符合輸入含 HealthState 值 Ok 的篩選條件。 值為 2。  <br> - Warning - 符合輸入含 HealthState 值 Warning 的篩選條件。 值為 4。  <br> - Error - 符合輸入含 HealthState 值 Error 的篩選條件。 值為 8。  <br> - All - 符合輸入含任何 HealthState 值的篩選條件。 值為 65535。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-info"></a>sfctl service info
取得屬於 Service Fabric 應用程式的特定服務相關資訊。

傳回屬於所指定 Service Fabric 應用程式的指定服務相關資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
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

## <a name="sfctl-service-list"></a>sfctl service list
取得屬於應用程式識別碼所指定應用程式之所有服務的相關資訊。

傳回屬於應用程式識別碼所指定應用程式之所有服務的相關資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --continuation-token | 接續權杖參數可用來取得下一組結果。 當來自系統的結果無法放入單一回應中時，API 的回應中會包含具有非空白值的接續權杖。 當此值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 如果沒有任何進一步的結果，接續權杖就不會包含值。 此參數的值不能經過 URL 編碼。 |
| --service-type-name | 用來篩選要查詢之服務的服務類型名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-manifest"></a>sfctl service manifest
取得可描述服務類型的資訊清單。

取得可描述服務類型的資訊清單。 回應會將服務資訊清單 XML 包含為字串。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-type-name    [必要] | 應用程式類型的名稱。 |
| --application-type-version [必要] | 應用程式類型的版本。 |
| --service-manifest-name    [必要] | 在 Service Fabric 叢集中，註冊為應用程式類型中一部分的服務資訊清單名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-package-deploy"></a>sfctl service package-deploy
將與所指定服務資訊清單相關聯的套件下載至所指定節點上的映像快取。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --app-type-name         [必要] | 對應所要求服務資訊清單的應用程式資訊清單名稱。 |
| --app-type-version      [必要] | 對應所要求服務資訊清單的應用程式資訊清單版本。 |
| --node-name             [必要] | 節點的名稱。 |
| --service-manifest-name [必要] | 與所要下載套件相關聯的服務資訊清單名稱。 |
| --share-policy | 共用原則的 JSON 編碼清單。 每個共用原則元素皆由「名稱」和「範圍」所組成。 名稱會與要共用的程式碼、設定或資料套件的名稱對應。 範圍可以是 'None'、'All'、'Code'、'Config' 或 'Data'。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-package-health"></a>sfctl service package-health
針對 Service Fabric 節點和應用程式所部署的特定應用程式，取得服務套件健康情況的相關資訊。

取得 Service Fabric 節點上所部署特定應用程式的服務套件健康情況相關資訊。 使用 EventsHealthStateFilter 可視需要根據健康情況狀態，篩選針對所部署服務套件回報的 HealthEvent 物件集合。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id       [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name            [必要] | 節點的名稱。 |
| --service-package-name [必要] | 服務套件的名稱。 |
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

## <a name="sfctl-service-package-info"></a>sfctl service package-info
取得 Service Fabric 節點上所部署且完全符合指定名稱的服務套件清單。

針對指定應用程式，傳回 Service Fabric 節點上所部署服務套件的相關資訊。 針對名稱與參數所指定服務套件名稱完全相符的服務套件，這些是其結果。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id       [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name            [必要] | 節點的名稱。 |
| --service-package-name [必要] | 服務套件的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-package-list"></a>sfctl service package-list
取得 Service Fabric 節點上所部署的服務套件清單。

針對指定應用程式，傳回 Service Fabric 節點上所部署服務套件的相關資訊。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --application-id [必要] | 應用程式的身分識別。 這通常是不含 'fabric\:' URI 配置的完整應用程式名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果應用程式名稱是 "fabric\:/myapp/app1"，則應用程式識別在 6.0+ 中會是 "myapp\~app1"，而在舊版中會是 "myapp/app1"。 |
| --node-name      [必要] | 節點的名稱。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-recover"></a>sfctl service recover
指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的指定服務。

指示 Service Fabric 叢集應該嘗試復原目前停留在仲裁遺失狀態的指定服務。 只有當已知無法復原已關閉的複本時，才能執行這項作業。 不正確使用這個 API，可能會造成資料遺失。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
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

## <a name="sfctl-service-report-health"></a>sfctl service report-health
傳送 Service Fabric 服務的健康情況報告。

回報所指定 Service Fabric 服務的健康情況狀態。 此報告必須包含健康情況報告來源及報告所針對屬性的相關資訊。 此報告會傳送給 Service Fabric 閘道服務，再由其轉送給健康狀態資料存放區。 閘道可能會接受此報告，但健康狀態資料存放區可能在進行額外驗證後會予以拒絕。 例如，健康狀態資料存放區可能會因為參數無效 (例如序號過時) 而拒絕此報告。 若要查看健康狀態資料存放區中是否已套用此報告，請檢查此報告是否出現在服務的健康情況事件中。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --health-property [必要] | 健康情況資訊的屬性。 <br><br> 實體可以有不同屬性的健康情況報告。 屬性是一個字串而不是固定的列舉，以便讓報告程式在分類觸發報告的狀態條件時較有彈性。 例如，SourceId 為 "LocalWatchdog" 的報告程式可以監視節點上可用磁碟的狀態，因此可以針對該節點回報 "AvailableDisk" 屬性。 該相同報告程式還可以監視節點連線能力，因此可以針對該相同節點回報 "Connectivity" 屬性。 在健康狀態資料存放區中，會將這些報告視為所指定節點的個別健康情況事件。 與 SourceId 搭配使用時，此屬性可唯一識別健康情況資訊。 |
| --health-state    [必要] | 可能的值包括：'Invalid'、'Ok'、'Warning'、'Error'、'Unknown'。 |
| --service-id      [必要] | 服務的身分識別。 <br><br> 這通常是不含 'fabric\:' URI 配置的完整服務名稱。 從 6.0 版開始，階層的名稱會以 '\~' 字元分隔。 例如，如果服務名稱是 'fabric\:/myapp/app1/svc1'，則服務識別在 6.0+ 中會是 'myapp\~app1\~svc1'，而在舊版中會是 'myapp/app1/svc1'。 |
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

## <a name="sfctl-service-resolve"></a>sfctl service resolve
解析 Service Fabric 資料分割。

解析 Service Fabric 服務分割區，以取得服務複本的端點。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --service-id [必要] | 服務的身分識別。 此識別碼通常是不含 'fabric\:' URI 配置的服務完整名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 "fabric\:/myapp/app1/svc1"，則服務識別在 6.0+ 中會是 "myapp\~app1\~svc1"，而在舊版中會是 "myapp/app1/svc1"。 |
| --partition-key-type | 資料分割的索引鍵類型。 如果服務的分割區配置是 Int64Range 或 Named，就必須設定此參數。 可能的值如下。 - None (1) - 指出未指定 PartitionKeyValue 參數。 這適用於分割區配置為 Singleton 的分割區。 這是預設值。 值為 1。 - Int64Range (2) - 指出 PartitionKeyValue 參數是 int64 分割區索引鍵。 這適用於分割區配置為 Int64Range 的分割區。 值為 2。 - Named (3) - 指出 PartitionKeyValue 參數是分割區名稱。 這適用於分割區配置為 Named 的分割區。 值為 3。 |
| --partition-key-value | 資料分割索引鍵。 如果服務的分割區配置是 Int64Range 或 Named，就必須設定此項目。 |
| --previous-rsp-version | 先前所收到回應中 [版本] 欄位的值。 如果使用者知道先前取得的結果已過時，就必須設定此項目。 |
| --timeout -t | 伺服器逾時 (秒)。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|說明|
| --- | --- |
| --debug | 增加記錄詳細程度以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細程度。 使用 --debug 取得完整偵錯記錄。 |

## <a name="sfctl-service-type-list"></a>sfctl service type-list
針對 Service Fabric 叢集中已佈建應用程式類型所支援的服務類型，取得包含服務類型相關資訊的清單。

針對 Service Fabric 叢集中已佈建應用程式類型所支援的服務類型，取得包含服務類型相關資訊的清單。 提供的應用程式類型必須存在。 否則，會傳回 404 狀態。

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

## <a name="sfctl-service-update"></a>sfctl service update
使用指定的更新描述來更新指定的服務。

### <a name="arguments"></a>引數

|引數|說明|
| --- | --- |
| --service-id   [必要] | 服務的身分識別。 這通常是不含 'fabric\:' URI 配置的完整服務名稱。 從 6.0 版開始，階層的名稱會以 "\~" 字元分隔。 例如，如果服務名稱是 'fabric\:/myapp/app1/svc1'，則服務識別在 6.0+ 中會是 'myapp\~app1\~svc1'，而在舊版中會是 'myapp/app1/svc1'。 |
| --constraints | 以字串表示的放置限制式。 放置條件約束是節點屬性上的布林運算式，可允許根據服務需求將服務限制在特定節點。 例如，若要在 NodeType 為 blue 的節點上放置服務，請指定："NodeColor == blue"。 |
| --correlated-service | 要與之建立相互關聯的目標服務名稱。 |
| --correlation | 使用對齊親和性將服務與現有服務相互關聯。 |
| --instance-count | 執行個體計數。 這只適用於無狀態服務。 |
| --load-metrics | 在節點之間進行負載平衡時所使用計量的 JSON 編碼清單。 |
| --min-replica-set-size | 以數字表示的最小複本集大小。 這只適用於具狀態服務。 |
| --move-cost | 指定服務的移動成本。 可能的值為：'Zero'、'Low'、'Medium'、'High'。 |
| --placement-policy-list | 服務的放置原則 JSON 編碼清單，以及任何相關聯的網域名稱。 原則可以是下列其中一或多個原則：`NonPartiallyPlaceService`、`PreferPrimaryDomain`、`RequireDomain`、`RequireDomainDistribution`。 |
| --quorum-loss-wait | 允許分割區處於仲裁遺失狀態的持續時間上限 (秒)。 這只適用於具狀態服務。 |
| --replica-restart-wait | 關閉複本與建立新複本之間的持續時間 (秒)。 這只適用於具狀態服務。 |
| --scaling-policies | 此服務的規模調整原則 JSON 編碼清單。 |
| --stand-by-replica-keep | StandBy 複本在被移除前的保留持續時間上限 (秒)。 這只適用於具狀態服務。 |
| --stateful | 指出目標服務是具狀態服務。 |
| --stateless | 指出目標服務是無狀態服務。 |
| --target-replica-set-size | 以數字表示的目標複本集大小。 這只適用於具狀態服務。 |
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
