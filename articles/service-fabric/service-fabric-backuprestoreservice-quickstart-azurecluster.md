---
title: 在 Azure Service Fabric 中定期備份和還原
description: 使用 Service Fabric 的定期備份與還原功能，啟用應用程式資料的定期資料備份。
ms.topic: conceptual
ms.date: 5/24/2019
ms.openlocfilehash: 2d167b261f9b5915a970b4c219113f0765c039cb
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98927996"
---
# <a name="periodic-backup-and-restore-in-an-azure-service-fabric-cluster"></a>Azure Service Fabric 叢集中的定期備份和還原
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

Service Fabric 中的備份與還原服務可讓您輕鬆自動備份具狀態服務中儲存的資訊。 定期備份應用程式資料是防止資料遺失和服務無法使用的基本原則。 Service Fabric 提供一項選用的備份與還原服務，可讓您無須撰寫任何額外的程式碼，即可設定具狀態 Reliable Services (包括「動作項目服務」) 的定期備份。 它也可以協助還原先前建立的備份。 


Service Fabric 提供一組 API，可實現下列和定期備份與復原功能相關的功能：

- 排定可靠具狀態服務和 Reliable Actors 的定期備份，並支援將備份上傳至 (外部) 儲存體位置。 支援的儲存位置
    - Azure 儲存體
    - 檔案共用 (內部部署)
- 列舉備份
- 觸發特定磁碟分割的臨機操作備份
- 使用先前的備份來還原分割區
- 暫時暫停備份
- 備份的保留管理 (即將推出)

## <a name="prerequisites"></a>先決條件
* 具有網狀架構6.4 版或更新版本的 Service Fabric 叢集。 如需了解使用 Azure 資源範本來建立 Service Fabric 叢集的步驟，請參閱這篇[文章](service-fabric-cluster-creation-via-arm.md)。
* 用於加密祕密 (連線至儲存體以儲存備份時所需) 的 X.509 憑證。 若要了解如何取得或建立 X.509 憑證，請參閱這篇[文章](service-fabric-cluster-creation-via-arm.md)。
* 使用 Service Fabric SDK 3.0 版或更新版本來建置的 Service Fabric 可靠具狀態應用程式。 針對以 .NET Core 2.0 為目標的應用程式，應該使用 Service Fabric SDK 3.1 版或更新版本來建立應用程式。
* 建立 Azure 儲存體帳戶來儲存應用程式備份。
* 安裝 ServiceFabric， (預覽) 進行設定呼叫。

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

> [!NOTE]
> 如果 PowerShellGet 版本小於1.6.0，您將需要更新以新增 *-AllowPrerelease* 旗標的支援：
>
> `Install-Module -Name PowerShellGet -Force`

* 使用 ServiceFabric 進行任何設定要求之前，請先使用命令來確定叢集已連線 `Connect-SFCluster` 。

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>啟用備份與還原服務

### <a name="using-azure-portal"></a>使用 Azure 入口網站

在 [] 索引標籤底下的 [啟用 `Include backup restore service` ] 核取方塊 `+ Show optional settings` `Cluster Configuration` 。

![使用入口網站啟用備份還原服務][1]


### <a name="using-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本
首先，您必須在叢集啟用「備份與還原服務」。 取得您想要部署之叢集的範本。 您可以使用[範例範本](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype)或建立 Resource Manager 範本。 請使用下列步驟來啟用「備份與還原服務」：

1. 檢查 `apiversion` 是否已 **`2018-02-01`** 為資源設定為 `Microsoft.ServiceFabric/clusters` ，如果不是，請更新它，如下列程式碼片段所示：

    ```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. 現在，在 `properties` 區段底下新增下列 `addonFeatures` 區段來啟用「備份與還原服務」，如下列程式碼片段所示： 

    ```json
        "properties": {
            ...
            "addonFeatures":  ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```
3. 設定用於加密認證的 X.509 憑證。 這很重要，可確保所提供來連線至儲存體的認證會先經過加密再進行保存。 在 `fabricSettings` 區段底下新增下列 `BackupRestoreService` 區段來設定加密憑證，如下列程式碼片段所示： 

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

4. 在您更新叢集範本以反映先前的變更之後，請套用它們，然後讓部署/升級完成。 完成之後，「備份與還原服務」就會開始在您的叢集中執行。 此服務的 URI 是 `fabric:/System/BackupRestoreService`，此服務可能位於 Service Fabric 總管中的系統服務區段底下。 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>啟用可靠具狀態服務和 Reliable Actors 的定期備份
以下步驟將逐步解說如何啟用可靠具狀態服務和 Reliable Actors 的定期備份。 這些步驟假設
- 已使用 X.509 安全性為叢集設定「備份與還原服務」。
- 叢集上已部署可靠具狀態服務。 基於本快速入門指南的目的，應用程式 URI 是 `fabric:/SampleApp`，而屬於此應用程式的可靠具狀態服務 URI 是 `fabric:/SampleApp/MyStatefulService`。 已為此服務部署單一分割區，而分割區識別碼是 `974bd92a-b395-4631-8a7f-53bd4ae9cf22`。
- 在將可供叫用下面指令碼的電腦上，具備系統管理員角色的用戶端憑證已安裝於 _CurrentUser_ 憑證存放區位置的 _My_ (_Personal_) 存放區名稱中。 此範例使用 `1b7ebe2174649c45474a4819dafae956712c31d3` 作為此憑證的指紋。 如需有關用戶端憑證的詳細資訊，請參閱[角色型存取控制 (適用於 Service Fabric 用戶端)](service-fabric-cluster-security-roles.md)。

### <a name="create-backup-policy"></a>建立備份原則

第一步是建立備份原則來描述備份排程、備份資料的目標儲存體、原則名稱，以及觸發備份儲存體的完整備份和保留原則之前所允許的增量備份上限。 

針對備份儲存體，請使用上面建立的 Azure 儲存體帳戶。 容器 `backup-container` 已設定為儲存備份。 如果此容器尚未存在，便會在備份上傳期間，建立具有此名稱的容器。 在 `ConnectionString` 中填入 Azure 儲存體帳戶的有效連接字串，然後使用您的儲存體帳戶名稱來取代 `account-name`，並使用您的儲存體帳戶金鑰來取代 `account-key`。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>使用 ServiceFabric 的 PowerShell 模組

執行下列 PowerShell Cmdlet 來建立新的備份原則。 請使用您的儲存體帳戶名稱來取代 `account-name`，並使用您的儲存體帳戶金鑰來取代 `account-key`。

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container' -Basic -RetentionDuration '10.00:00:00'

```

#### <a name="rest-call-using-powershell"></a>使用 PowerShell 進行 Rest 呼叫

執行下列 PowerShell 指令碼來叫用必要的 REST API 以建立新原則。 請使用您的儲存體帳戶名稱來取代 `account-name`，並使用您的儲存體帳戶金鑰來取代 `account-key`。

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
}

$RetentionPolicy = @{ 
    RetentionPolicyType = 'Basic'
    RetentionDuration =  'P10D'
}

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
    RetentionPolicy = $RetentionPolicy
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

```

#### <a name="using-service-fabric-explorer"></a>使用 Service Fabric Explorer

1. 在 Service Fabric Explorer 中，流覽至 [備份] 索引標籤，然後選取 [動作] > 建立備份原則]。

    ![建立備份原則][6]

2. 填寫資訊。 針對 Azure 叢集，應該選取 [AzureBlobStore]。

    ![建立備份原則 Azure Blob 儲存體][7]

### <a name="enable-periodic-backup"></a>啟用定期備份
定義可滿足應用程式資料保護需求的備份原則之後，應該將該備份原則與應用程式建立關聯。 視需求而定，備份原則可以與應用程式、服務或分割區建立關聯。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>使用 ServiceFabric 的 PowerShell 模組

```powershell

Enable-SFApplicationBackup -ApplicationId 'SampleApp' -BackupPolicyName 'BackupPolicy1'

```
#### <a name="rest-call-using-powershell"></a>使用 PowerShell 進行 Rest 呼叫

請執行下列 PowerShell 指令碼來叫用必要的 REST API，以將在上述步驟中所建立名為 `BackupPolicy1` 的備份原則與應用程式 `SampleApp` 建立關聯。

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
``` 

#### <a name="using-service-fabric-explorer"></a>使用 Service Fabric Explorer

1. 選取應用程式並移至 [動作]。 按一下 [啟用/更新應用程式備份]。

    ![啟用應用程式備份][3]

2. 最後，選取所需的原則，然後按一下 [啟用備份]。

    ![選取原則][4]


### <a name="verify-that-periodic-backups-are-working"></a>確認定期備份能夠運作

啟用應用程式層級的備份之後，所有屬於應用程式底下可靠具狀態服務和 Reliable Actors 的分割區，就會依據關聯的備份原則開始定期進行備份。 

![分割區備份健康情況事件][0]

### <a name="list-backups"></a>列出備份

您可以使用 _GetBackups_ API，以列舉屬於應用程式可靠具狀態服務和 Reliable Actors 的所有分割區相關備份。 您可以列舉應用程式、服務或分割區的備份。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>使用 ServiceFabric 的 PowerShell 模組

```powershell
    
Get-SFApplicationBackupList -ApplicationId WordCount
```

#### <a name="rest-call-using-powershell"></a>使用 PowerShell 進行 Rest 呼叫

請執行下列 PowerShell 指令碼來叫用 HTTP API，以列舉針對 `SampleApp` 應用程式內所有分割區建立的備份。

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

上述執行的範例輸出：

```
BackupId                : b9577400-1131-4f88-b309-2bb1e943322c
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 20.55.16.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3334
CreationTimeUtc         : 2018-04-06T20:55:16Z
FailureError            : 

BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            : 

BackupId                : 69436834-c810-4163-9386-a7a800f78359
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.25.36.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3764
CreationTimeUtc         : 2018-04-06T21:25:36Z
FailureError            : 
```

#### <a name="using-service-fabric-explorer"></a>使用 Service Fabric Explorer

若要在 Service Fabric Explorer 中查看備份，請流覽至分割區，然後選取 [備份] 索引標籤。

![列舉備份][5]

## <a name="limitation-caveats"></a>限制 / 注意事項
- Service Fabric PowerShell Cmdlet 處於預覽模式。
- 不支援 Linux 上的 Service Fabric 叢集。

## <a name="next-steps"></a>後續步驟
- [了解定期備份組態](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [備份還原 REST API 參考](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/partition-backedup-health-event-azure.png
[1]: ./media/service-fabric-backuprestoreservice/enable-backup-restore-service-with-portal.png
[3]: ./media/service-fabric-backuprestoreservice/enable-app-backup.png
[4]: ./media/service-fabric-backuprestoreservice/enable-application-backup.png
[5]: ./media/service-fabric-backuprestoreservice/backup-enumeration.png
[6]: ./media/service-fabric-backuprestoreservice/create-bp.png
[7]: ./media/service-fabric-backuprestoreservice/creation-bp.png
