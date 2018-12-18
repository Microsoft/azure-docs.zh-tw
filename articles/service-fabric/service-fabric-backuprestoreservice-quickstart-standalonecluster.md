---
title: 快速入門 - Azure Service Fabric 中的定期備份與還原 (預覽) | Microsoft Docs
description: 使用 Service Fabric 的定期備份與還原功能，啟用應用程式資料的定期資料備份。
services: service-fabric
documentationcenter: .net
author: hrushib
manager: timlt
editor: hrushib
ms.assetid: FAADBCAB-F0CF-4CBC-B663-4A6DCCB4DEE1
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/04/2018
ms.author: hrushib
ms.openlocfilehash: 5666ed4226104104b91b6f964abeb0490ef80866
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38990739"
---
# <a name="quickstart-periodic-backup-and-restore-in-azure-service-fabric-preview"></a>快速入門：Azure Service Fabric 中的定期備份與還原 (預覽)
> [!div class="op_single_selector"]
> * [Azure 上的叢集](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [獨立叢集](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service Fabric 是一個分散式系統平台，可讓您輕鬆開發及管理可靠的分散式微服務型雲端應用程式。 它可同時允許執行無狀態及具狀態微服務。 具狀態服務可維護要求與回應或完整交易以外的可變動授權狀態。 如果具狀態服務長時間停止運作，或因為災害而遺失資訊，可能就需要將它還原至其某個最近的狀態備份，才能在恢復運作後繼續提供服務。

Service Fabric 會將狀態複寫至多個節點，以確保服務具有高可用性。 即使叢集內的一個節點失敗，服務仍繼續可供使用。 不過，在某些情況下，仍然建議您讓服務資料能夠穩定可靠地面對更廣泛的失敗狀況。
 
例如，服務可以備份其資料，以針對下列情況提供防護：
- 發生整個 Service Fabric 叢集永久遺失的情況。
- 永久遺失多數的服務分割區複本
- 不小心刪除或損毀狀態的系統管理錯誤。 例如，具備足夠權限的系統管理員錯誤地刪除服務。
- 服務中造成資料損毀的錯誤。 例如，當服務程式碼升級而開始將錯誤資料寫入「可靠的集合」時，就可能發生此情況。 在這種情況下，可能必須將程式碼和資料還原成先前的狀態。
- 離線資料處理。 對於獨立於服務來產生資料的商業智慧，離線處理資料相當方便。

Service Fabric 提供內建的 API 來執行時間點[備份與還原](service-fabric-reliable-services-backup-restore.md)。 應用程式開發人員可以使用這些 API 來定期備份服務的狀態。 此外，如果服務管理員想要在特定時間 (例如在升級應用程式之前) 從服務外部觸發備份，開發人員就必須從服務將備份 (和還原) 公開為 API。 維護備份是這之外的一筆額外成本。 例如，您可以每半小時建立 5 個增量備份，接著再建立一個完整備份。 在建立完整備份之後，您便可以刪除先前的增量備份。 這個方法需要額外的程式碼，而會造成在應用程式開發期間產生額外的成本。

對於管理分散式應用程式，以及防止資料遺失或服務長期中斷來說，定期備份應用程式資料是一項基本需求。 Service Fabric 提供一項選用的備份與還原服務，可讓您無須撰寫任何額外的程式碼，即可設定具狀態 Reliable Services (包括「動作項目服務」) 的定期備份。 它也可以協助還原先前建立的備份。 

> [!NOTE]
> 定期備份與還原功能目前為**預覽版**，不支援用於生產環境工作負載。 
>

Service Fabric 提供一組 API，可實現下列和定期備份與復原功能相關的功能：

- 排定可靠具狀態服務和 Reliable Actors 的定期備份，並支援將備份上傳至 (外部) 儲存體位置。 支援的儲存位置
    - Azure 儲存體
    - 檔案共用 (內部部署)
- 列舉備份
- 觸發分割區的臨機操作備份
- 使用先前的備份來還原分割區
- 暫時暫停備份
- 備份的保留管理 (即將推出)

## <a name="prerequisites"></a>必要條件
* 具有 Fabric 6.2 版和更新版本的 Service Fabric 叢集。 應該在 Windows Server 上設定叢集。 如需了解下載所需套件的步驟，請參閱這篇[文章](service-fabric-cluster-creation-for-windows-server.md)。
* 用於加密祕密 (連線至儲存體以儲存備份時所需) 的 X.509 憑證。 若要了解如何取得或建立自我簽署的 X.509 憑證，請參閱這篇[文章](service-fabric-windows-cluster-x509-security.md)。
* 使用 Service Fabric SDK 3.0 版或更新版本來建置的 Service Fabric 可靠具狀態應用程式。 針對以 .Net Core 2.0 為目標的應用程式，則應該使用 Service Fabric SDK 3.1 版或更新版本來建置應用程式。

## <a name="enabling-backup-and-restore-service"></a>啟用備份與還原服務
首先，您必須在叢集啟用「備份與還原服務」。 取得您想要部署之叢集的範本。 您可以使用[範例範本](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)。 請使用下列步驟來啟用「備份與還原服務」：

1. 檢查叢集設定檔中的 `apiversion` 是否已設定為 `10-2017`，如果不是，請更新它，如下列程式碼片段所示：

    ```json
    {
        "apiVersion": "10-2017",
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        ...
    }
    ```

2. 現在，在 `properties` 區段底下新增下列 `addonFeatures` 區段來啟用「備份與還原服務」，如下列程式碼片段所示： 

    ```json
        "properties": {
            ...
            "addonFeatures": ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```

3. 設定用於加密認證的 X.509 憑證。 這很重要，可確保所提供來連線至儲存體的認證 (如果有的話) 會先經過加密再進行保存。 在 `fabricSettings` 區段底下新增下列 `BackupRestoreService` 區段來設定加密憑證，如下列程式碼片段所示： 

    ```json
    "properties": {
        ...
        "addonFeatures": ["BackupRestoreService"],
        "fabricSettings": [{
            "name": "BackupRestoreService",
            "parameters":  [{
                "name": "SecretEncryptionCertThumbprint",
                "value": "[Thumbprint]"
            }]
        }
        ...
    }
    ```

4. 在您更新叢集設定檔以反映先前的變更之後，請套用它們，然後讓部署/升級完成。 完成之後，「備份與還原服務」就會開始在您的叢集中執行。 此服務的 URI 是 `fabric:/System/BackupRestoreService`，此服務可能位於 Service Fabric 總管中的系統服務區段底下。 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>啟用可靠具狀態服務和 Reliable Actors 的定期備份
以下步驟將逐步解說如何啟用可靠具狀態服務和 Reliable Actors 的定期備份。 這些步驟假設
- 已為叢集設定「備份與還原服務」。
- 叢集上已部署可靠具狀態服務。 基於本快速入門指南的目的，應用程式 URI 是 `fabric:/SampleApp`，而屬於此應用程式的可靠具狀態服務 URI 是 `fabric:/SampleApp/MyStatefulService`。 已為此服務部署單一分割區，而分割區識別碼是 `23aebc1e-e9ea-4e16-9d5c-e91a614fefa7`。  

### <a name="create-backup-policy"></a>建立備份原則

第一步是建立備份原則來描述備份排程、備份資料的目標儲存體、原則名稱，以及觸發完整備份之前所允許的增量備份上限。 

針對備份儲存體，請建立檔案共用，然後將此檔案共用的 ReadWrite 存取權授與所有 Service Fabric 節點電腦。 此範例假設 `StorageServer` 上有名為 `BackupStore` 的共用。

執行下列 PowerShell 指令碼來叫用必要的 REST API 以建立新原則。

```powershell
$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
}   

$StorageInfo = @{
    Path = '\\StorageServer\BackupStore'
    StorageKind = 'FileShare'
}

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "http://localhost:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.2-preview"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
```

### <a name="enable-periodic-backup"></a>啟用定期備份
定義可滿足應用程式資料保護需求的原則之後，應該將該備份原則與應用程式建立關聯。 視需求而定，備份原則可以與應用程式、服務或分割區建立關聯。

請執行下列 PowerShell 指令碼來叫用必要的 REST API，以將在上述步驟中所建立名為 `BackupPolicy1` 的備份原則與應用程式 `SampleApp` 建立關聯。

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "http://localhost:19080/Applications/SampleApp/$/EnableBackup?api-version=6.2-preview"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
``` 

### <a name="verify-that-periodic-backups-are-working"></a>確認定期備份能夠運作

啟用應用程式的備份之後，所有屬於應用程式底下可靠具狀態服務和 Reliable Actors 的分割區，就會依據關聯的備份原則開始定期進行備份。 

![分割區備份健康情況事件][0]

### <a name="list-backups"></a>列出備份

您可以使用 _GetBackups_ API，以列舉屬於應用程式可靠具狀態服務和 Reliable Actors 的所有分割區相關備份。 視需求而定，可以列舉應用程式、服務或分割區的備份。

請執行下列 PowerShell 指令碼來叫用 HTTP API，以列舉針對 `SampleApp` 應用程式內所有分割區建立的備份。

```powershell
$url = "http://localhost:19080/Applications/SampleApp/$/GetBackups?api-version=6.2-preview"

$response = Invoke-WebRequest -Uri $url -Method Get

$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```
上述執行的範例輸出：

```
BackupId                : d7e4038e-2c46-47c6-9549-10698766e714
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.39.40.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2058
CreationTimeUtc         : 2018-04-01T19:39:40Z
FailureError            : 

BackupId                : 8c21398a-2141-4133-b4d7-e1a35f0d7aac
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.54.38.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2237
CreationTimeUtc         : 2018-04-01T19:54:38Z
FailureError            : 

BackupId                : fc75bd4c-798c-4c9a-beee-e725321f73b2
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 20.09.44.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2437
CreationTimeUtc         : 2018-04-01T20:09:44Z
FailureError            : 
```

## <a name="preview-limitation-caveats"></a>預覽版限制/注意事項
- 沒有任何 Service Fabric 內建 PowerShell Cmdlet。
- 不支援 Service Fabric CLI。
-  不支援自動化備份清除。 參考[備份保留指令碼](https://github.com/Microsoft/service-fabric-scripts-and-templates/tree/master/scripts/BackupRetentionScript)可設定以指令碼為基礎的外部自動化，以供清除備份。
- 不支援 Linux 上的 Service Fabric 叢集。

## <a name="next-steps"></a>後續步驟
- [了解定期備份組態](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [備份還原 REST API 參考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/PartitionBackedUpHealthEvent.png

