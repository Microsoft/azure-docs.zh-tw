---
title: 準備 Azure 匯入/匯出匯入作業的硬碟 - v1 |Microsoft Docs
description: 了解如何使用 WAImportExport v1 工具準備硬碟，以建立 Azure 匯入/匯出服務的匯入作業。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 861b3302e065689a4ea9c0df0879f9c0df12e619
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526941"
---
# <a name="preparing-hard-drives-for-an-import-job"></a>針對匯入作業準備硬碟
若要準備匯入工作的一個或多個硬碟，請依照下列步驟執行︰

-   指定要匯入至 Blob 服務的資料

-   指定目標虛擬目錄和 Blob 服務中的 Blob

-   判斷需要多少個磁碟機

-   將資料複製到每個硬碟

 如需範例工作流程，請參閱[準備匯入工作的硬碟的範例工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow-v1.md)。

## <a name="identify-the-data-to-be-imported"></a>識別要匯入的資料
 建立匯入工作的第一步是判斷您要匯入哪些目錄和檔案。 可以是目錄清單、唯一檔案的清單或兩者的組合。 包含目錄時，目錄及其子目錄中的所有檔案都是匯入工作的一部分。

> [!NOTE]
>  因為包含父目錄時會以遞迴方式包含子目錄，因此只需指定父目錄。 請勿一併指定其任何子目錄。
>
>  目前 Microsoft Azure 匯入/匯出工具有下列限制︰如果目錄包含超過一個硬碟可以包含的資料，則目錄需分割成較小的目錄。 比方說，如果目錄包含 2.5 TB 的資料，而硬碟容量只有 2 TB，則您需要將 2.5 TB 的目錄分割成較小的目錄。 更新版的工具中將會解決這項限制。

## <a name="identify-the-destination-locations-in-the-blob-service"></a>識別 Blob 服務中的目的地位置
 針對要匯入的每個目錄或檔案，您需要指定目的地虛擬目錄或 Azure Blob 服務中的 Blob。 您將使用這些目標做為 Azure 匯入/匯出工具的輸入。 請注意，目錄應使用正斜線字元 "/" 分隔。

 下表顯示一些 Blob 目標範例：

|來源檔案或目錄|目的地 Blob 或虛擬目錄|
|------------------------------|-------------------------------------------|
|H:\Video|https://mystorageaccount.blob.core.windows.net/video|
|H:\Photo|https://mystorageaccount.blob.core.windows.net/photo|
|K:\Temp\FavoriteVideo.ISO|https://mystorageaccount.blob.core.windows.net/favorite/FavoriteVideo.ISO|
|\\\myshare\john\music|https://mystorageaccount.blob.core.windows.net/music|

## <a name="determine-how-many-drives-are-needed"></a>判斷需要多少個磁碟機
 接著，您必須判斷︰

-   儲存資料所需的硬碟數目。

-   將複製到每個硬碟的目錄和/或獨立檔案。

 確定您具有儲存傳輸的資料所需的硬碟數目。

## <a name="copy-data-to-your-hard-drive"></a>將資料複製到硬碟
 本節描述如何呼叫 Azure 匯入/匯出工具，以將資料複製到一或多個硬碟。 每次呼叫 Azure 匯入/匯出工具時，會建立新的「複製工作階段」。 您要為每個用於複製資料的磁碟機建立至少一個複製工作階段；在某些情況下，您可能需要多個複製工作階段才能將所有資料複製到單一磁碟機。 以下是一些您可能需要多個複製工作階段的原因︰

-   您必須為每個用於複製的磁碟機建立個別的複製工作階段。

-   複製工作階段可以將單一目錄或單一 Blob 複製到磁碟機。 如果您要複製多個目錄、多個 Blob 或兩者的組合，您必須建立多個複製工作階段。

-   您可以指定將在匯入工作匯入的 Blob 上設定的屬性和中繼資料。 您為複製工作階段指定的屬性或中繼資料，將套用至該複製工作階段指定的所有 Blob。 如果您想為一些 Blob 指定不同的屬性或中繼資料，您必須建立個別的複製工作階段。 如需詳細資訊，請參閱[在匯入程序期間設定屬性和中繼資料](storage-import-export-tool-setting-properties-metadata-import-v1.md)。

> [!NOTE]
>  如果您有多部機器符合[設定 Azure 匯入/匯出工具](storage-import-export-tool-setup-v1.md)中所描述的需求，您可以在每部機器上執行此工具的執行個體，以將資料平行複製到多個硬碟。

 針對您使用 Azure 匯入/匯出工具準備的每個硬碟，工具將會建立單一日誌檔案。 所有磁碟機都將需要日誌檔案，才能建立匯入工作。 如果工具中斷，日誌檔案也可用來繼續磁碟機準備。

### <a name="azure-importexport-tool-syntax-for-an-import-job"></a>適用於匯入作業的 Azure 匯入/匯出工具語法
 如果要準備匯入作業的磁碟機，請使用 **PrepImport** 命令呼叫 Azure 匯入/匯出工具。 您包含的參數取決於這是第一個複製工作階段或後續複製工作階段。

 磁碟機的第一個複製工作階段需要一些其他參數來指定儲存體帳戶金鑰、目標磁碟機代號、是否必須格式化磁碟機、是否必須加密磁碟機 (如果是，則需要 BitLocker 金鑰) 和記錄檔目錄。 以下是複製目錄或單一檔案的初始複製工作階段語法︰

 **複製單一目錄的第一個複製工作階段**

 `WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`

 **複製單一檔案的第一個複製工作階段**

 `WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcfile:<SourceFile> /dstblob:<DestinationBlobPath> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`

 在後續的複製工作階段中，您不需要指定初始參數。 以下是複製目錄或單一檔案的後續複製工作階段語法︰

 **複製單一目錄的後續複製工作階段**

 `WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`

 **複製單一檔案的後續複製工作階段**

 `WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /srcfile:<SourceFile> /dstblob:<DestinationBlobPath> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`

### <a name="parameters-for-the-first-copy-session-for-a-hard-drive"></a>適用於硬碟的第一個複製工作階段的參數
 每次執行 Azure 匯入/匯出工具以將檔案複製到硬碟時，工具會建立一個複製工作階段。 每個複製工作階段會將單一目錄或單一檔案複製到硬碟。 複製工作階段的狀態會寫入日誌檔案。 如果複製工作階段中斷 (例如，因為系統電源斷電)，可以再次執行工具並在命令列指定日誌檔案以繼續。

> [!WARNING]
>  如果您為第一個複製工作階段指定 **/format** 參數，將會格式化磁碟機並清除磁碟機上的所有資料。 建議您僅使用空白的磁碟機進行複製工作階段。

 每個磁碟機的第一個複製工作階段所用的命令需要的參數不同於後續複製工作階段的命令。 下表列出第一個複製工作階段可用的其他參數︰

|命令列參數|說明|
|-----------------------------|-----------------|
|**/sk:**<StorageAccountKey\>|`Optional.`將匯入資料的儲存體帳戶的儲存體帳戶金鑰。 您必須在命令中包含 **/sk:**<StorageAccountKey\> 或 **/csas:**<ContainerSas\>。|
|**/csas:**<ContainerSas\>|`Optional`。 用於將資料匯入儲存體帳戶的容器 SAS。 您必須在命令中包含 **/sk:**<StorageAccountKey\> 或 **/csas:**<ContainerSas\>。<br /><br /> 這個參數的值開頭必須是容器名稱，後面加上問號 (?) 和 SAS Token。 例如︰<br /><br /> `mycontainer?sv=2014-02-14&sr=c&si=abcde&sig=LiqEmV%2Fs1LF4loC%2FJs9ZM91%2FkqfqHKhnz0JM6bqIqN0%3D&se=2014-11-20T23%3A54%3A14Z&sp=rwdl`<br /><br /> 權限 (無論是在 URL 或預存存取原則中指定) 必須包含讀取、寫入和刪除匯入工作，以及讀取、寫入和列出匯出工作。<br /><br /> 指定這個參數時，要匯入或匯出的所有 Blob 必須位於共用存取簽章中指定的容器內。|
|**/t:**<TargetDriveLetter\>|`Required.`目前複製工作階段中目標硬碟的磁碟機代號，不包含結尾的冒號。|
|**/format**|`Optional.`當磁碟機需要進行格式化時請指定此參數；否則請省略。 此工具格式化磁碟機之前，會提示您從主控台進行確認。 若要隱藏確認，請指定 /silentmode 參數。|
|**/silentmode**|`Optional.`指定此參數可隱藏格式化目標磁碟機的確認。|
|**/encrypt**|`Optional.`當磁碟機尚未使用 BitLocker 加密，且需要透過工具加密時，請指定此參數。 若已使用 BitLocker 加密磁碟機，則省略此參數並指定 `/bk`參數，以提供現有的 BitLocker 金鑰。<br /><br /> 若您指定 `/format` 參數，則您也必須指定 `/encrypt` 參數。|
|**/bk:**<BitLockerKey\>|`Optional.`若指定 `/encrypt`，請省略這個參數。 若省略 `/encrypt`，您必須已使用 BitLocker 加密磁碟機。 使用此參數可指定 BitLocker 金鑰。 匯入工作的所有硬碟都需要 BitLocker 加密。|
|**/logdir:**<LogDirectory\>|`Optional.`記錄檔目錄指定要用來儲存詳細資訊記錄檔和暫存資訊清單檔案的目錄。 如未指定，將使用目前的目錄做為記錄檔目錄。|

### <a name="parameters-required-for-all-copy-sessions"></a>所有複製工作階段需要的參數
 日誌檔案包含硬碟的所有複製工作階段的狀態。 它也包含建立匯入工作所需的資訊。 執行 Azure 匯入/匯出工具時，一律必須指定日誌檔案及複製工作階段識別碼：

|||
|-|-|
|命令列參數|說明|
|**/j:**<JournalFile\>|`Required.`日誌檔案的路徑。 每個磁碟機必須只有一個日誌檔案。 請注意，日誌檔案不得位於目標磁碟機上。 日誌檔案的副檔名是 `.jrn`。|
|**/id:**<SessionId\>|`Required.`工作階段識別碼會識別複製工作階段。 其用來確保正確復原中斷的複製工作階段。 在複製工作階段中複製的檔案會儲存在以目標磁碟機上工作階段識別碼命名的目錄。|

### <a name="parameters-for-copying-a-single-directory"></a>用以複製單一目錄的參數
 複製單一目錄時，適用下列必要和選用參數︰

|命令列參數|說明|
|----------------------------|-----------------|
|**/srcdir:**<SourceDirectory\>|`Required.`包含要複製到目標磁碟機的檔案的來源目錄。 目錄路徑必須是絕對路徑 (而非相對路徑)。|
|**/dstdir:**<DestinationBlobVirtualDirectory\>|`Required.`Microsoft Azure 儲存體帳戶中的目的地虛擬目錄路徑。 虛擬目錄可能已存在或可能不存在。<br /><br /> 您可以指定容器或 Blob 前置詞，如 `music/70s/`。 目的地目錄必須以容器名稱開頭，後面接著正斜線 "/"，並可選擇性地包含結尾是 "/" 的虛擬 Blob 目錄。<br /><br /> 目的地容器為根容器時，您必須明確指定根容器 (包括正斜線)，例如 `$root/`。 由於根容器下的 Blob 名稱中不能包含 "/"，當目的地目錄是根容器時，將不會複製來源目錄中的任何子目錄。<br /><br /> 指定目的地虛擬目錄或 blob 時，請確定使用有效的容器名稱。 請記住容器名稱必須是小寫。 關於容器命名規則，請參閱[命名和參考容器、Blob 及中繼資料](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。|
|**/Disposition:**<rename&#124;no-overwrite&#124;overwrite>|`Optional.`指定具有指定位址的 Blob 已存在時的行為。 這個參數的有效值為︰`rename`、`no-overwrite`及 `overwrite`。 請注意，這些值區分大小寫。 如果未指定任何值，預設值為 `rename`。<br /><br /> 為此參數指定的值會影響 `/srcdir` 參數指定的目錄中的所有檔案。|
|**/BlobType:**<BlockBlob&#124;PageBlob>|`Optional.`指定目的地 Blob 的 Blob 類型。 有效值為：`BlockBlob`和 `PageBlob`。 請注意，這些值區分大小寫。 如果未指定任何值，預設值為 `BlockBlob`。<br /><br /> 在大部分情況下，建議使用 `BlockBlob`。 如果您指定 `PageBlob`，目錄中每個檔案的長度必須是 512 (分頁 Blob 的分頁大小) 的倍數。|
|**/PropertyFile:**<PropertyFile\>|`Optional.`目的地 Blob 的屬性檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。|
|**/MetadataFile:**<MetadataFile\>|`Optional.`目的地 Blob 的中繼資料檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。|

### <a name="parameters-for-copying-a-single-file"></a>用以複製單一檔案的參數
 複製單一檔案時，適用下列必要和選用參數︰

|命令列參數|說明|
|----------------------------|-----------------|
|**/srcfile:**<SourceFile\>|`Required.`要複製的檔案的完整路徑。 目錄路徑必須是絕對路徑 (而非相對路徑)。|
|**/dstblob:**<DestinationBlobPath\>|`Required.`Microsoft Azure 儲存體帳戶中目的地 Blob 的路徑。 Blob 可能已存在或可能不存在。<br /><br /> 指定開頭為容器名稱的 Blob 名稱。 Blob 名稱不得以 "/" 或儲存體帳戶名稱開頭。 關於 Blob 命名規則，請參閱[命名和參考容器、Blob 及中繼資料](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。<br /><br /> 目的地容器為根容器時，您必須明確指定 `$root` 做為容器，例如 `$root/sample.txt`。 請注意，根容器下的 Blob 名稱不能包含 "/"。|
|**/Disposition:**<rename&#124;no-overwrite&#124;overwrite>|`Optional.`指定具有指定位址的 Blob 已存在時的行為。 這個參數的有效值為︰`rename`、`no-overwrite`及 `overwrite`。 請注意，這些值區分大小寫。 如果未指定任何值，預設值為 `rename`。|
|**/BlobType:**<BlockBlob&#124;PageBlob>|`Optional.`指定目的地 Blob 的 Blob 類型。 有效值為：`BlockBlob`和 `PageBlob`。 請注意，這些值區分大小寫。 如果未指定任何值，預設值為 `BlockBlob`。<br /><br /> 在大部分情況下，建議使用 `BlockBlob`。 如果您指定 `PageBlob`，目錄中每個檔案的長度必須是 512 (分頁 Blob 的分頁大小) 的倍數。|
|**/PropertyFile:**<PropertyFile\>|`Optional.`目的地 Blob 的屬性檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。|
|**/MetadataFile:**<MetadataFile\>|`Optional.`目的地 Blob 的中繼資料檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。|

### <a name="resuming-an-interrupted-copy-session"></a>繼續中斷的複製工作階段
 如果複製工作階段因任何原因中斷，只要指定日誌檔案，就可以執行工具來繼續︰

```
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /ResumeSession
```

 如果是異常中止，只能繼續最近的複製工作階段。

> [!IMPORTANT]
>  當您繼續複製工作階段時，請勿以加入或移除檔案的方式修改來源資料檔案和目錄。

### <a name="aborting-an-interrupted-copy-session"></a>中止中斷的複製工作階段
 如果複製工作階段中斷且無法繼續 (例如來源目錄已證實無法存取)，您必須中止目前的工作階段，使其可以回復並開始新的複製工作階段︰

```
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AbortSession
```

 如果是異常終止，只能中止最後一個複製工作階段。 請注意，您無法中止磁碟機的第一個複製工作階段。 而是必須使用新的日誌檔案重新開始複製工作階段。

## <a name="next-steps"></a>後續步驟

* [設定 Azure 匯入/匯出工具](storage-import-export-tool-setup-v1.md)
* [在匯入程序期間設定屬性和中繼資料](storage-import-export-tool-setting-properties-metadata-import-v1.md)
* [針對匯入作業準備硬碟的簡單工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow-v1.md)
* [常用命令快速參考](storage-import-export-tool-quick-reference-v1.md) 
* [利用複製記錄檔檢閱作業狀態](storage-import-export-tool-reviewing-job-status-v1.md)
* [修復匯入作業](storage-import-export-tool-repairing-an-import-job-v1.md)
* [修復匯出作業](storage-import-export-tool-repairing-an-export-job-v1.md)
* [針對 Azure 匯入/匯出工具進行疑難排解](storage-import-export-tool-troubleshooting-v1.md)
