---
title: 在 Azure Service Fabric 中了解定期備份設定 | Microsoft Docs
description: 使用 Service Fabric 的定期備份與還原功能，啟用應用程式資料的定期資料備份。
services: service-fabric
documentationcenter: .net
author: hrushib
manager: timlt
editor: hrushib
ms.assetid: FAA45B4A-0258-4CB3-A825-7E8F70F28401
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/01/2018
ms.author: hrushib
ms.openlocfilehash: eeaa0e9a940f16c2416418959c98cd17e4816afc
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387628"
---
# <a name="understanding-periodic-backup-configuration-in-azure-service-fabric"></a>在 Azure Service Fabric 中了解定期備份設定

為可靠的具狀態服務或 Reliable Actors 設定定期備份，包含下列步驟：

1. **建立備份原則**：在此步驟中，視需求建立一或多個備份原則。

2. **啟用備份**：在此步驟中，您會讓**步驟 1** 中建立的備份原則與必要實體、應用程式、服務或分割區產生關聯。

## <a name="create-backup-policy"></a>建立備份原則

備份原則由下列設定組成：

* **在資料遺失時自動還原**：指定是否要在分割區發生資料遺失事件時，使用最新可用的備份來自動觸發還原。

* **最大增量備份**：定義要在兩個完整備份之間採用的最大增量備份。 最大增量備份可指定上限。 在下列其中一種情況下完成指定的增量備份數目之前，可以採取完整備份：

    1. 複本在變成主要複本之後從未執行完整備份。

    2. 自上次備份被截斷之後記錄了部分記錄檔。

    3. 複本超出 MaxAccumulatedBackupLogSizeInMB 限制。

* **備份排程**：進行定期備份的時間或頻率。 使用者可以將備份排定在指定的間隔重複執行，或在每日 / 每週的固定時間重複執行。

    1. **以頻率為基礎的備份排程**：如果需要在固定間隔進行資料備份，就應該使用此排程類型。 使用 ISO8601 格式，定義兩個連續備份之間所需的時間間隔。 以頻率為基礎的備份排程支援以分鐘為單位的間隔解析。
        ```json
        {
            "ScheduleKind": "FrequencyBased",
            "Interval": "PT10M"
        }
        ```

    2. **以時間為基礎的備份排程**：如果需要在當天或當週的特定時間進行資料備份，就應該使用此排程類型。 排程頻率類型可以是每天或每週。
        1. **每日以時間為基礎的備份排程**：如果需要在當天的特定時間進行資料備份，就應該使用此排程類型。 若要指定此類型，請將 `ScheduleFrequencyType` 設定為 [Daily]；並將 `RunTimes` 設定為一天當中所需的時間 (採用 ISO8601 格式) 清單，將會忽略與時間一起指定的日期。 例如，`0001-01-01T18:00:00` 代表每天「下午 6:00」，並忽略日期部份 0001-01-01。 下面的範例說明要在每天「上午 9:00」和「下午 6:00」觸發每日備份的組態。

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Daily",
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

        2. **每週以時間為基礎的備份排程**：如果需要在當天的特定時間進行資料備份，就應該使用此排程類型。 若要指定此類型，請將 `ScheduleFrequencyType` 設定為 [Weekly]；並將 `RunDays` 設定為一週當中需要觸發備份的星期幾清單，以及將 `RunTimes` 設定為一天當中所需的時間 (採用 ISO8601 格式) 清單，將會忽略與時間一起指定的日期。 一周當中要觸發定期備份的星期幾清單。 下面的範例說明要在週一至週五期間「上午 9:00」和「下午 6:00」觸發每日備份的組態。

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Weekly",
                "RunDays": [
                   "Monday",
                   "Tuesday",
                   "Wednesday",
                   "Thursday",
                   "Friday"
                ],
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

* **備份儲存體**：指定要上傳備份的位置。 儲存體可以是 Azure blob 存放區或檔案共用。
    1. **Azure blob 存放區**：需要在 Azure 中儲存所產生的備份時，就應該選取此儲存體類型。 「獨立」和「以 Azure 為基礎」的叢集都可以使用此儲存體類型。 此儲存體類型的描述需要連接字串和必須上傳備份的容器名稱。 如果沒有具指定名稱的容器，則會在上傳備份期間加以建立。
        ```json
        {
            "StorageKind": "AzureBlobStore",
            "FriendlyName": "Azure_storagesample",
            "ConnectionString": "<Put your Azure blob store connection string here>",
            "ContainerName": "BackupContainer"
        }
        ```

    2. **檔案共用**：需要在內部部署環境中儲存資料備份時，就應該針對「獨立」叢集選取此儲存體類型。 此儲存體類型的描述需要必須上傳備份的檔案共用路徑。 您可以使用下列其中一個選項，設定檔案共用的存取權：
        1. 整合式 Windows 驗證，可對屬於 Service Fabric 叢集的所有電腦提供檔案共用的存取權。 在此情況下，設定下列欄位，以設定以「檔案共用」為基礎的備份儲存體。

            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore"
            }
            ```

        2. 使用使用者名稱和密碼保護檔案共用，可對特定使用者提供檔案共用的存取權。 檔案共用儲存體規格也可供指定次要使用者名稱和次要密碼，以在使用主要使用者名稱和主要密碼驗證失敗的情況下，提供後援認證。 在此情況下，設定下列欄位，以設定以「檔案共用」為基礎的備份儲存體。
            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore",
                "PrimaryUserName": "backupaccount",
                "PrimaryPassword": "<Password for backupaccount>",
                "SecondaryUserName": "backupaccount2",
                "SecondaryPassword": "<Password for backupaccount2>"
            }
            ```

> [!NOTE]
> 確保儲存體可靠性符合或超過備份資料的可靠性需求。
>

## <a name="enable-periodic-backup"></a>啟用定期備份
定義可滿足資料備份需求的備份原則之後，應該將該備份原則與應用程式、服務或分割區建立適當關聯。

### <a name="hierarchical-propagation-of-backup-policy"></a>備份原則的階層式傳播
在 Service Fabric 中，應用程式、服務和分割區之間具有階層式關聯性，如[應用程式模型](./service-fabric-application-model.md)中所說明。 備份原則可以與階層中的應用程式、服務或分割區建立關聯。 備份原則會以階層方式傳播到下一個層級。 假設只建立了一個備份原則並與應用程式相關聯，則會使用備份原則來備份屬於該應用程式的所有可靠具狀態服務和 Reliable Actors的所有具狀態分割區。 而如果備份原則與可靠具狀態服務相關聯，則會使用備份原則來備份其所有分割區。

### <a name="overriding-backup-policy"></a>覆寫備份原則
有可能出現以下情節：應用程式的所有服務都需要採用相同備份排程的資料備份，但是需要使用較高頻率排程來備份資料或將資料備份到不同儲存體帳戶或檔案共用的特定服務除外。 為了解決這類案例，備份還原服務會提供設施，以覆寫在服務和分割區範圍內傳播的原則。 如果備份原則與服務或分割區相關聯，它就會覆寫傳播的備份原則 (如果有的話)。

### <a name="example"></a>範例

此範例使用具有兩個應用程式的安裝方式：_MyApp_A_ 和 _MyApp_B_。 應用程式 _MyApp_A_ 包含兩個可靠具狀態服務 _SvcA1_ & _SvcA3_，以及一個 Reliable Actor 服務 _ActorA2_。 _SvcA1_ 包含三個分割區，而 _ActorA2_ 和 _SvcA3_ 各自包含兩個分割區。  應用程式 _MyApp_B_ 包含三個可靠具狀態服務 _SvcB1_、_SvcB2_ 及 _SvcB3_。 _SvcB1_ 和 _SvcB2_ 各自包含兩個分割區，而 _SvcB3_ 包含三個分割區。

假設這些應用程式的資料備份需求如下所示：

1. MyApp_A
    1. 針對屬於該應用程式的所有可靠具狀態服務和 _Reliable Actors_ 的所有分割區，建立每日資料備份。 將備份資料上傳到 _BackupStore1_ 位置。

    2. 其中一項服務 _SvcA3_ 需要每小時備份資料。

    3. 分割區 _SvcA1_P2_ 的資料大小超過預期，而且其備份資料應儲存到不同的儲存體位置 _BackupStore2_。

2. MyApp_B
    1. 針對 _SvcB1_ 服務的所有分割區，建立在每星期日上午 8:00 的資料備份。 將備份資料上傳到 _BackupStore1_ 位置。

    2. 針對 _SvcB2_P1_ 分割區，建立在每天上午 8:00 的資料備份。 將備份資料上傳到 _BackupStore1_ 位置。

為了解決這些資料備份需求，則會建立備份原則 BP_1 至 BP_5 並啟用備份，如下所示。
1. MyApp_A
    1. 建立備份原則 _BP_1_，採用以頻率為基礎的備份排程，其頻率設定為 24 小時。 而且備份儲存體設定為使用儲存體位置 _BackupStore1_。 使用[啟用應用程式備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-enableapplicationbackup) API，針對應用程式 _MyApp_A_ 啟用此原則。 此動作可啟用資料備份，做法是針對屬於應用程式 _MyApp_A_ 的可靠具狀態服務和 Reliable Actors 的所有分割區，使用備份原則 _BP_1_。

    2. 建立備份原則 _BP_2_，採用以頻率為基礎的備份排程，其頻率設定為 1 小時。 而且備份儲存體設定為使用儲存體位置 _BackupStore1_。 使用[啟用服務備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-enableservicebackup) API，針對服務 _SvcA3_ 啟用此原則。 對於 _SvcA3_ 服務的所有分割區，此動作會以明確啟用的備份原則 _BP_2_ 覆寫傳播的原則 _BP_1_，進而導致使用這些分割區的備份原則 _BP_2_ 來備份資料。

    3. 建立備份原則 _BP_3_，採用以頻率為基礎的備份排程，其頻率設定為 24 小時。 而且備份儲存體設定為使用儲存體位置 _BackupStore2_。 使用[啟用分割區備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-enablepartitionbackup) API，針對分割區 _SvcA1_P2_ 啟用此原則。 對於分割區 _SvcA1_P2_，此動作會以明確啟用的備份原則 _BP_3_ 覆寫傳播的原則 _BP_1_。

2. MyApp_B
    1. 建立備份原則 _BP_4_，採用以時間為基礎的備份排程，其排程頻率類型設定為每週，執行日設定為星期日，而執行時間設定為上午 8:00。 備份儲存體設定為使用儲存體位置 _BackupStore1_。 使用[啟用服務備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-enableservicebackup) API，針對服務 _SvcB1_ 啟用此原則。 此動作會使用備份原則 _BP_4_，對 _SvcB1_ 服務的所有分割區啟用資料備份。

    2. 建立備份原則 _BP_5_，採用以時間為基礎的備份排程，其排程頻率類型設定為每日且執行時間設定為上午 8:00。 備份儲存體設定為使用儲存體位置 _BackupStore1_。 使用[啟用分割區備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-enablepartitionbackup) API，針對分割區 _SvcB2_P1_ 啟用此原則。 此動作會使用備份原則 _BP_5_，對分割區 _SvcB2_P1_ 啟用資料備份。

下圖描述明確啟用的備份原則和傳播的備份原則。

![Service Fabric 應用程式階層][0]

## <a name="disable-backup"></a>停用備份
不需要備份資料時，可以停用備份原則。 在應用程式啟用的備份原則只能使用[停用應用程式備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-disableapplicationbackup) API 在相同的應用程式停用，在服務啟用的備份原則可以使用[停用服務備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-disableservicebackup) API 在相同的服務停用，而在分割區啟用的備份原則可以使用[停用分割區備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-disablepartitionbackup) API 在相同的分割區停用。

* 停用應用程式的備份原則會阻止所有的定期資料備份發生，這是因為備份原則傳播到可靠具狀態服務分割區或 Reliable Actors 分割區。

* 停用服務的備份原則會阻止所有的定期資料備份發生，這是因為此備份原則傳播到服務的分割區。

* 停用分割區的備份原則會阻止所有的定期資料備份發生，這是由於備份原則屬於分割區範圍。

## <a name="suspend--resume-backup"></a>暫止與繼續備份
有些情況可能會要求暫時擱置資料的定期備份。 在這種情況下，根據需求而定，可能會在應用程式、服務或分割區使用暫止備份 API。 定期備份擱置可從其套用點透過應用程式階層的樹狀子目錄轉移。 

* 使用[暫止應用程式備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-suspendapplicationbackup) API 在應用程式套用擱置時，此應用程式之下的所有服務和分割區都會暫止資料的定期備份。

* 使用[暫止服務備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-suspendservicebackup) API 在服務套用擱置時，此服務之下的所有分割區都會暫止資料的定期備份。

* 使用[暫止分割區備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-suspendpartitionbackup) API 在分割區套用擱置時，此服務之下的分割區都會暫止資料的定期備份。

擱置需求結束後，則可使用各自的繼續備份 API 來還原定期資料備份。 定期備份必須在暫止時的相同應用程式、服務或_分割區_上繼續進行。

* 如果在應用程式套用擱置，則應該使用[繼續應用程式備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-resumeapplicationbackup) API 繼續進行。 

* 如果在服務套用擱置，則應該使用[繼續服務備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-resumeservicebackup) API 繼續進行。

* 如果在分割區套用擱置，則應該使用[繼續分割區備份](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-resumepartitionbackup) API 繼續進行。

## <a name="auto-restore-on-data-loss"></a>在資料遺失時自動還原
服務分割區可能因為非預期的失敗而遺失資料。 例如，分割區的三分之二複本 (包括主要複本) 的磁碟損毀或抹除。

當 Service Fabric 偵測到分割區遺失資料時，它會在分割區上叫用 `OnDataLossAsync` 介面方法，並預期分割區會採取必要的動作來擺脫資料遺失。 在此情況下，如果分割區的有效備份原則將其 `AutoRestoreOnDataLoss` 旗標設為 `true`，則會使用此分割區的最新可用備份自動觸發還原。

## <a name="get-backup-configuration"></a>取得備份組態
已提供不同的 API，以便取得應用程式、服務和分割區範圍的備份組態資訊。 這些 API 分別是[取得應用程式備份組態資訊](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getapplicationbackupconfigurationinfo)、[取得服務備份組態資訊](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getservicebackupconfigurationinfo)，以及[取得分割區備份組態資訊](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackupconfigurationinfo)。 這些 API 主要傳回適用的備份原則，此備份原則的套用範圍，以及備份擱置詳細資料。 以下是這些 API 所傳回結果的簡短說明。

- 應用程式備份組態資訊：提供在應用程式套用的備份原則詳細資料，以及在屬於該應用程式的服務和分割區覆寫的所有原則。 它也包含應用程式及其服務和分割區的擱置資訊。

- 服務備份組態資訊：提供服務的有效備份原則詳細資料，以及此原則的套用範圍與在其分割區覆寫的所有原則。 它也包含服務及其分割區的擱置資訊。

- 分割區備份組態資訊：提供分割區的有效備份原則詳細資料，以及此原則的套用範圍。 它也包含分割區的擱置資訊。

## <a name="list-available-backups"></a>列出可用的備份

使用 [取得備份清單] API 可以列出可用的備份。 API 呼叫的結果包含在備份儲存體 (設定於適用的備份原則中) 可用的所有備份的備份資訊項目。 此 API 的不同變體可供列出屬於應用程式、服務或分割區的可用備份。 這些 API 支援取得所有適用分割區的「最新」可用備份，或根據「開始日期」和「結束日期」篩選備份。

這些 API 也支援在 MaxResults 參數設為非零正整數時將結果分頁，則 API 會傳回最多 MaxResults 備份資訊項目。 萬一可用的備份資訊項目超過 MaxResults 值，則會傳回接續 Token。 有效的接續 Token 參數可用來取得下一組結果。 當有效的接續 Token 值傳遞至下一個 API 呼叫時，API 會傳回下一組結果。 傳回所有可用的結果時，回應中就不會包含接續 Token。

以下是關於支援變體的簡短資訊。

- [取得應用程式備份清單](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getapplicationbackuplist)：傳回屬於所指定 Service Fabric 應用程式的每個分割區可用的備份清單。

- [取得服務備份清單](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getservicebackuplist)：傳回屬於所指定 Service Fabric 服務的每個分割區可用的備份清單。
 
- [取得分割區備份清單](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackuplist)：傳回指定的分割區可用的備份清單。

## <a name="next-steps"></a>後續步驟
- [備份還原 REST API 參考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/BackupPolicyAssociationExample.png