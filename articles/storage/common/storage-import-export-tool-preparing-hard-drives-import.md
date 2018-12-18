---
title: 準備 Azure 匯入/匯出匯入工作的硬碟 |Microsoft Docs
description: 了解如何使用 WAImportExport 工具準備硬碟，以建立 Azure 匯入/匯出服務的匯入工作。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 06/29/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 9d8509e97ad83dd636f0a1b1892a2fa67c69e0b7
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521790"
---
# <a name="preparing-hard-drives-for-an-import-job"></a>準備匯入工作的硬碟

WAImportExport 工具是磁碟機準備及修復工具，可搭配 [Microsoft Azure 匯入/匯出服務](../storage-import-export-service.md)使用。 您可以使用此工具，將資料複製到要寄送至 Azure 資料中心的硬碟。 匯入工作完成後，您可以使用此工具來修復損毀、遺漏或與其他 Blob 衝突的任何 Blob。 當您收到已完成的匯出工作中的磁碟機後，您可以使用此工具來修復磁碟機上損毀或遺漏的任何檔案。 我們會在本文中介紹此工具的使用方式。

## <a name="prerequisites"></a>必要條件

### <a name="requirements-for-waimportexportexe"></a>WAImportExport.exe 的需求

- **電腦組態**
  - Windows 7、Windows Server 2008 R2 或更新版本的 Windows 作業系統
  - 必須安裝 .NET framework 4。 請參閱[常見問題集](#faq)，了解如何檢查電腦上是否已安裝 .Net Framework。
- **儲存體帳戶金鑰** - 您需要至少一個儲存體帳戶的帳戶金鑰。

### <a name="preparing-disk-for-import-job"></a>準備匯入工作的磁碟

- **BitLocker -** 執行 WAImportExport 工具的電腦上必須啟用 BitLocker。 如需如何啟用 BitLocker 的詳細資訊，請參閱[常見問題集](#faq)。
- **磁碟**必須可從執行 WAImportExport 工具的電腦上存取。 如需磁碟規格的詳細資訊，請參閱[常見問題集](#faq)。
- **來源檔案** - 您想匯入的檔案必須可從複製電腦上存取，無論它們是在網路共用或本機硬碟上。

### <a name="repairing-a-partially-failed-import-job"></a>修復部分失敗的匯入工作

- **複製記錄檔**，是 Azure 匯入/匯出服務在儲存體帳戶和磁碟之間複製資料時產生的。 此檔案位於您的目標儲存體帳戶。

### <a name="repairing-a-partially-failed-export-job"></a>修復部分失敗的匯出工作

- **複製記錄檔**，是 Azure 匯入/匯出服務在儲存體帳戶和磁碟之間複製資料時產生的。 此檔案位於您的來源儲存體帳戶。
- **資訊清單檔案** - [選擇性] 位於 Microsoft 寄回的匯出磁碟機上。

## <a name="download-and-install-waimportexport"></a>下載和安裝 WAImportExport

下載[最新版的 WAImportExport.exe](https://www.microsoft.com/download/details.aspx?id=55280)。 將壓縮的內容解壓縮至您電腦上的目錄。

您的下一個工作是建立 CSV 檔案。

## <a name="prepare-the-dataset-csv-file"></a>準備資料集 CSV 檔案

### <a name="what-is-dataset-csv"></a>什麼是資料集 CSV

資料集 CSV 檔案是 /dataset 旗標的值，此 CSV 檔案包含要複製到目標磁碟機的目錄清單和/或檔案清單。 建立匯入工作的第一步是判斷您要匯入哪些目錄和檔案。 可以是目錄清單、唯一檔案的清單或兩者的組合。 包含目錄時，目錄及其子目錄中的所有檔案都是匯入工作的一部分。

針對要匯入的每個目錄或檔案，您必須指定目的地虛擬目錄或 Azure Blob 服務中的 Blob。 您將使用這些目標做為 WAImportExport 工具的輸入。 目錄應使用斜線字元 "/" 分隔。

下表顯示一些 Blob 目標範例：

| 來源檔案或目錄 | 目的地 Blob 或虛擬目錄 |
| --- | --- |
| H:\Video | https://mystorageaccount.blob.core.windows.net/video |
| H:\Photo | https://mystorageaccount.blob.core.windows.net/photo |
| K:\Temp\FavoriteVideo.ISO | https://mystorageaccount.blob.core.windows.net/favorite/FavoriteVideo.ISO |
| \\myshare\john\music | https://mystorageaccount.blob.core.windows.net/music |

### <a name="sample-datasetcsv"></a>範例 dataset.csv

```
BasePath,DstBlobPathOrPrefix,BlobType,Disposition,MetadataFile,PropertiesFile
"F:\50M_original\100M_1.csv.txt","containername/100M_1.csv.txt",BlockBlob,rename,"None",None
"F:\50M_original\","containername/",BlockBlob,rename,"None",None
```

### <a name="dataset-csv-file-fields"></a>資料集 CSV 檔案欄位

| 欄位 | 說明 |
| --- | --- |
| 基本路徑 | **[必要]**<br/>此參數的值代表要匯入資料所在的來源。 此工具將以遞迴方式複製位於此路徑下的所有資料。<br><br/>**允許值**︰這必須是本機電腦上的有效路徑或有效的共用路徑，而且應可供使用者存取。 目錄路徑必須是絕對路徑 (而非相對路徑)。 如果路徑的結尾為 "\\"，即代表目錄，而結尾不是 "\\" 的路徑則代表檔案。<br/>此欄位中不允許 Regex。 如果路徑包含空格，請使用 "" 括住。<br><br/>**範例**："c:\Directory\c\Directory\File.txt"<br>"\\\\FBaseFilesharePath.domain.net\sharename\directory\"  |
| DstBlobPathOrPrefix | **[必要]**<br/> Microsoft Azure 儲存體帳戶中的目的地虛擬目錄路徑。 虛擬目錄可能已存在或可能不存在。 如果不存在，匯入/匯出服務將會建立一個。<br/><br/>指定目的地虛擬目錄或 blob 時，請確定使用有效的容器名稱。 請記住容器名稱必須是小寫。 關於容器命名規則，請參閱[命名和參考容器、Blob 及中繼資料](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。 如果只指定根，即會在目的地 Blob 容器中複寫來源的目錄結構。 如果所需的目錄結構不在來源中，CSV 中的多個對應資料列<br/><br/>您可以指定容器或 blob 首碼，如 music/70s/。 目的地目錄必須以容器名稱開頭，後面接著正斜線 "/"，並可選擇性地包含結尾是 "/" 的虛擬 Blob 目錄。<br/><br/>目的地容器為根容器時，您必須明確指定根容器 (包括正斜線)，例如 $root /。 由於根容器下的 Blob 名稱中不能包含 "/"，當目的地目錄是根容器時，將不會複製來源目錄中的任何子目錄。<br/><br/>**範例**<br/>如果目的地 Blob 路徑是 https://mystorageaccount.blob.core.windows.net/video，這個欄位的值可以是 video/  |
| BlobType | **[選用]** block &#124; page<br/>目前匯入/匯出服務支援兩種 Blob。 分頁 blob 和區塊 Blob，所有檔案預設會匯入為區塊 Blob。 \*.vhd 和 \*.vhdx 會匯入為分頁 Blob。區塊 Blob 和分頁 Blob 允許的大小有限。 如需詳細資訊，請參閱[儲存體延展性目標](storage-scalability-targets.md) (英文)。  |
| Disposition | **[選用]** rename &#124; no-overwrite &#124; overwrite <br/> 此欄位會指定匯入期間的複製行為，也就是說 正在從磁碟將資料上傳至儲存體帳戶時。 可用的選項為：rename&#124;overwite&#124;no-overwrite。若未指定任何項目，預設值為 "rename"。 <br/><br/>**Rename**︰如果已經有同名的物件，則在目的地建立複本。<br/>覆寫︰以較新的檔案覆寫檔案。 上次修改 wins 的檔案。<br/>**不要覆寫**︰如已存在則略過覆寫檔案。|
| MetadataFile | **[選用]** <br/>此欄位的值為中繼資料檔案，如果需要保留物件的中繼資料或提供自訂中繼資料，則可提供這個欄位的值。 目的地 Blob 的中繼資料檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。 |
| PropertiesFile | **[選用]** <br/>目的地 Blob 的屬性檔案路徑。 如需詳細資訊，請參閱[匯入/匯出服務中繼資料和屬性檔案格式](../storage-import-export-file-format-metadata-and-properties.md)。 |

## <a name="prepare-initialdriveset-or-additionaldriveset-csv-file"></a>準備 InitialDriveSet 或 AdditionalDriveSet CSV 檔案

### <a name="what-is-driveset-csv"></a>什麼是磁碟機集 CSV

/InitialDriveSet 或 /AdditionalDriveSet 旗標的值為 CSV 檔案，其中包含磁碟機代號對應的磁碟清單，使工具可以正確挑選要準備的磁碟清單。 如果資料大小超過單一磁碟大小，WAImportExport 工具會將資料以最佳化方式分散在此 CSV 檔案中列示的多個磁碟。

單一工作階段中可寫入資料的磁碟數目沒有限制。 此工具會根據磁碟大小和資料夾大小來分配資料。 它將選取最適合物件大小的磁碟。 資料上傳至儲存體帳戶時將併回資料集檔案中指定的目錄結構。 若要建立磁碟機集 CSV，請遵循下列步驟。

### <a name="create-basic-volume-and-assign-drive-letter"></a>建立基本磁碟區並指派磁碟機代號

若要建立基本磁碟區並指派磁碟機代號，請依照[磁碟管理概觀](https://technet.microsoft.com/library/cc754936.aspx)中「建立較簡易的磁碟分割」的指示進行。

### <a name="sample-initialdriveset-and-additionaldriveset-csv-file"></a>範例 InitialDriveSet 與 AdditionalDriveSet CSV 檔案

```
DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631
H,Format,SilentMode,Encrypt,
```

### <a name="driveset-csv-file-fields"></a>磁碟機集 CSV 檔案欄位

| 欄位 | 值 |
| --- | --- |
| DriveLetter | **[必要]**<br/> 每個提供給工具做為目的地的磁碟機上都必須有一個簡單的 NTFS 磁碟區，並已指派磁碟機代號。<br/> <br/>**範例**R 或 r |
| FormatOption | **[必要]** Format &#124; AlreadyFormatted<br/><br/> **Format**︰指定這個將格式化磁碟上的所有資料。 <br/>**AlreadyFormatted**︰指定此值時，工具會略過格式化。 |
| SilentOrPromptOnFormat | **[必要]** SilentMode &#124; PromptOnFormat<br/><br/>**SilentMode**︰提供這個值可讓使用者以無訊息模式執行工具。 <br/>**PromptOnFormat**︰工具會提示使用者確認針對每種格式的動作是否是想要的。<br/><br/>如果未設定，命令將會中止並顯示錯誤訊息："Incorrect value for SilentOrPromptOnFormat: none" \(SilentOrPromptOnFormat 的值不正確：無\) |
| 加密 | **[必要]** Encrypt &#124; AlreadyEncrypted<br/> 這個欄位的值決定要加密和不加密的磁碟。 <br/><br/>**Encrypt**︰工具將格式化磁碟機。 如果 "FormatOption" 欄位的值是 "Format"，則此值需為 "Encrypt"。 如果在此情況下指定 "AlreadyEncrypted"，則會造成下列錯誤：「指定 Format 時，也必須指定 Encrypt」。<br/>**AlreadyEncrypted**︰工具將使用 "ExistingBitLockerKey" 欄位中提供的 BitLockerKey 解密磁碟機。 如果 "FormatOption" 欄位的值是 "AlreadyFormatted"，這個值可以是 "Encrypt" 或 "AlreadyEncrypted" |
| ExistingBitLockerKey | **[必要]** 如果 "Encryption" 欄位的值是 "AlreadyEncrypted"<br/> 這個欄位的值是與特定磁碟相關聯的 BitLocker 金鑰。 <br/><br/>如果 "Encryption" 欄位的值為 "Encrypt"，此欄位應保留空白。  如果在此情況下指定 BitLocker 金鑰，則會造成下列錯誤：「不應指定 Bitlocker 金鑰」。<br/>  **範例**：060456-014509-132033-080300-252615-584177-672089-411631|

##  <a name="preparing-disk-for-import-job"></a>準備匯入工作的磁碟

如果要準備匯入工作的磁碟機，請使用 **PrepImport** 命令呼叫 WAImportExport 工具。 您包含的參數取決於這是第一個複製工作階段或後續複製工作階段。

### <a name="first-session"></a>第一個工作階段

將單一/多重目錄複製到單一/多重磁碟的第一個複製工作階段 (取決於在 CSV 檔案中指定的項目) WAImportExport 工具使用於第一個複製工作階段的 PrepImport 命令，會以新的複製工作階段複製目錄和/或檔案︰

```
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>
```

**範例：**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:\*\*\*\*\*\*\*\*\*\*\*\*\* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs
```

### <a name="add-data-in-subsequent-session"></a>在後續工作階段中加入資料

在後續的複製工作階段中，您不需要指定初始參數。 您必須使用相同的日誌檔案，讓工具能記得前一個工作階段的進度。 複製工作階段的狀態會寫入日誌檔案。 以下是複製其他目錄和/或檔案的後續複製工作階段的語法︰

```
WAImportExport.exe PrepImport /j:<SameJournalFile> /id:<DifferentSessionId>  [DataSet:<differentdataset.csv>]
```

**範例：**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

### <a name="add-drives-to-latest-session"></a>將磁碟機加入最近的工作階段

如果資料無法容納於 InitialDriveset 中指定的磁碟機，可以使用工具將其他磁碟機加入同一個複製工作階段。 

>[!NOTE] 
>工作階段識別碼應符合前一個工作階段識別碼。日誌檔案應符合前一個工作階段中指定的日誌檔案。
>
```
WAImportExport.exe PrepImport /j:<SameJournalFile> /id:<SameSessionId> /AdditionalDriveSet:<newdriveset.csv>
```

**範例：**

```
WAImportExport.exe PrepImport /j:SameJournalTest.jrn /id:session#2  /AdditionalDriveSet:driveset-2.csv
```

### <a name="abort-the-latest-session"></a>中止最近的工作階段︰

如果複製工作階段中斷且無法繼續 (例如來源目錄已證實無法存取)，您必須中止目前的工作階段，使其可以回復並開始新的複製工作階段︰

```
WAImportExport.exe PrepImport /j:<SameJournalFile> /id:<SameSessionId> /AbortSession
```

**範例：**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /AbortSession
```

如果是異常終止，只能中止最後一個複製工作階段。 請注意，您無法中止磁碟機的第一個複製工作階段。 而是必須使用新的日誌檔案重新開始複製工作階段。

### <a name="resume-a-latest-interrupted-session"></a>繼續最近中斷的工作階段

如果複製工作階段因任何原因中斷，只要指定日誌檔案，就可以執行工具來繼續︰

```
WAImportExport.exe PrepImport /j:<SameJournalFile> /id:<SameSessionId> /ResumeSession
```

**範例：**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2 /ResumeSession
```

> [!IMPORTANT] 
> 當您繼續複製工作階段時，請勿以加入或移除檔案的方式修改來源資料檔案和目錄。

## <a name="waimportexport-parameters"></a>WAImportExport 參數

| 參數 | 說明 |
| --- | --- |
|     /j:&lt;JournalFile&gt;  | **必要**<br/> 日誌檔案的路徑。 日誌檔案會追蹤一組磁碟機，並記錄準備這些磁碟機的進度。 日誌檔案一律需要指定。  |
|     /logdir:&lt;LogDirectory&gt;  | **選擇性**。 記錄檔目錄。<br/> 詳細資訊記錄檔及某些暫存檔案會寫入至這個目錄。 如未指定，將使用目前的目錄做為記錄檔目錄。 同一日誌檔案只能指定一次記錄檔目錄。  |
|     /id:&lt;SessionId&gt;  | **必要**<br/> 工作階段識別碼用於識別複製工作階段。 其用來確保正確復原中斷的複製工作階段。  |
|     /ResumeSession  | 選用。 如果最後一個複製工作階段異常終止，可以指定此參數以繼續工作階段。   |
|     /AbortSession  | 選用。 如果最後一個複製工作階段異常終止，可以指定此參數以中止工作階段。  |
|     /sn:&lt;StorageAccountName&gt;  | **必要**<br/> 僅適用於 RepairImport 和 RepairExport。 儲存體帳戶的名稱。  |
|     /sk:&lt;StorageAccountKey&gt;  | **必要**<br/> 儲存體帳戶的金鑰。 |
|     /InitialDriveSet:&lt;driveset.csv&gt;  | **必要** 執行第一個複製工作階段時<br/> CSV 檔案，包含要準備的磁碟機清單。  |
|     /AdditionalDriveSet:&lt;driveset.csv&gt; | **必要**。 將磁碟機加入目前的複製工作階段時。 <br/> CSV 檔案，包含要加入的其他磁碟機的清單。  |
|      /r:&lt;RepairFile&gt; | **必要** 僅適用於 RepairImport 和 RepairExport。<br/> 追蹤修復進度之檔案的路徑。 每個磁碟機需要一個修復檔案，而且只能有一個。  |
|     /d:&lt;TargetDirectories&gt; | **必要**。 僅適用於 RepairImport 和 RepairExport。 用於 RepairImport 時，是要修復的一或多個目錄 (以分號分隔)；用於 RepairExport 時，是要修復的目錄，例如磁碟機根目錄。  |
|     /CopyLogFile:&lt;DriveCopyLogFile&gt; | **必要** 僅適用於 RepairImport 和 RepairExport。 磁碟機複製記錄檔 (詳細資訊或錯誤) 的路徑。  |
|     /ManifestFile:&lt;DriveManifestFile&gt; | **必要** 僅適用於 RepairExport。<br/> 磁碟機資訊清單檔案的路徑。  |
|     /PathMapFile:&lt;DrivePathMapFile&gt; | **選擇性**。 僅適用於 RepairImport。<br/> 包含檔案路徑對應 (相對於實際檔案位置的磁碟機根目錄) 之檔案的路徑 (以 tab 分隔)。 第一次指定時，它會填入空白目標的檔案路徑，表示在 TargetDirectories 中找不到它們、存取被拒、具有無效名稱，或它們存在多個目錄中。 路徑對應檔案可以手動編輯以包含正確的目標路徑，並可再次指定，使工具可以正確解析檔案路徑。  |
|     /ExportBlobListFile:&lt;ExportBlobListFile&gt; | **必要**。 僅適用於 PreviewExport。<br/> XML 檔案的路徑，此檔案包含要匯出的 Blob 的Blob 路徑清單或 Blob 路徑前置詞。 此檔案格式與匯入/匯出服務 REST API 的 Put Job 作業中的 Blob 清單 Blob 格式相同。  |
|     /DriveSize:&lt;DriveSize&gt; | **必要**。 僅適用於 PreviewExport。<br/>  要用於匯出的磁碟機大小。 例如，500 GB、1.5 TB。 注意：1 GB = 1,000,000,000 個位元組 1 TB = 1,000,000,000,000 個位元組  |
|     /DataSet:&lt;dataset.csv&gt; | **必要**<br/> CSV 檔案，包含要複製到目標磁碟機的目錄清單和/或檔案清單。  |
|     /silentmode  | **選擇性**。<br/> 如未指定，它會提醒您磁碟機的需求，並需要您確認才能繼續。  |

## <a name="tool-output"></a>工具輸出

### <a name="sample-drive-manifest-file"></a>範例磁碟機資訊清單檔案

```xml
<?xml version="1.0" encoding="UTF-8"?>
<DriveManifest Version="2011-MM-DD">
   <Drive>
      <DriveId>drive-id</DriveId>
      <StorageAccountKey>storage-account-key</StorageAccountKey>
      <ClientCreator>client-creator</ClientCreator>
      <!-- First Blob List -->
      <BlobList Id="session#1-0">
         <!-- Global properties and metadata that applies to all blobs -->
         <MetadataPath Hash="md5-hash">global-metadata-file-path</MetadataPath>
         <PropertiesPath Hash="md5-hash">global-properties-file-path</PropertiesPath>
         <!-- First Blob -->
         <Blob>
            <BlobPath>blob-path-relative-to-account</BlobPath>
            <FilePath>file-path-relative-to-transfer-disk</FilePath>
            <ClientData>client-data</ClientData>
            <Length>content-length</Length>
            <ImportDisposition>import-disposition</ImportDisposition>
            <!-- page-range-list-or-block-list -->
            <!-- page-range-list -->
            <PageRangeList>
               <PageRange Offset="1073741824" Length="512" Hash="md5-hash" />
               <PageRange Offset="1073741824" Length="512" Hash="md5-hash" />
            </PageRangeList>
            <!-- block-list -->
            <BlockList>
               <Block Offset="1073741824" Length="4194304" Id="block-id" Hash="md5-hash" />
               <Block Offset="1073741824" Length="4194304" Id="block-id" Hash="md5-hash" />
            </BlockList>
            <MetadataPath Hash="md5-hash">metadata-file-path</MetadataPath>
            <PropertiesPath Hash="md5-hash">properties-file-path</PropertiesPath>
         </Blob>
      </BlobList>
   </Drive>
</DriveManifest>
```

### <a name="sample-journal-file-xml-for-each-drive"></a>每個磁碟機的範例日誌檔案 (XML)

```xml
[BeginUpdateRecord][2016/11/01 21:22:25.379][Type:ActivityRecord]
ActivityId: DriveInfo
DriveState: [BeginValue]
<?xml version="1.0" encoding="UTF-8"?>
<Drive>
   <DriveId>drive-id</DriveId>
   <BitLockerKey>*******</BitLockerKey>
   <ManifestFile>\DriveManifest.xml</ManifestFile>
   <ManifestHash>D863FE44F861AE0DA4DCEAEEFFCCCE68</ManifestHash> </Drive>
[EndValue]
SaveCommandOutput: Completed
[EndUpdateRecord]
```

### <a name="sample-journal-file-jrn-for-session-that-records-the-trail-of-sessions"></a>工作階段的範例日誌檔案 (JRN)，此檔案會記錄工作階段的軌跡

```
[BeginUpdateRecord][2016/11/02 18:24:14.735][Type:NewJournalFile]
VocabularyVersion: 2013-02-01
[EndUpdateRecord]
[BeginUpdateRecord][2016/11/02 18:24:14.749][Type:ActivityRecord]
ActivityId: PrepImportDriveCommandContext
LogDirectory: F:\logs
[EndUpdateRecord]
[BeginUpdateRecord][2016/11/02 18:24:14.754][Type:ActivityRecord]
ActivityId: PrepImportDriveCommandContext
StorageAccountKey: *******
[EndUpdateRecord]
```

## <a name="faq"></a>常見問題集

### <a name="general"></a>一般

#### <a name="what-is-waimportexport-tool"></a>什麼是 WAImportExport 工具？

WAImportExport 工具是磁碟機準備及修復工具，可搭配 Microsoft Azure 匯入/匯出服務使用。 您可以使用此工具，將資料複製到要寄送至 Azure 資料中心的硬碟。 匯入工作完成後，您可以使用此工具來修復損毀、遺漏或與其他 Blob 衝突的任何 Blob。 當您收到已完成的匯出工作中的磁碟機後，您可以使用此工具來修復磁碟機上損毀或遺漏的任何檔案。

#### <a name="how-does-the-waimportexport-tool-work-on-multiple-source-dir-and-disks"></a>WAImportExport 工具如何在多個來源目錄和磁碟上運作？

如果資料大小超過磁碟大小，WAImportExport 工具會將資料以最佳化方式分散在多個磁碟。 資料可以平行或循序方式複製到多個磁碟。 同時寫入資料的磁碟數目沒有限制。 此工具會根據磁碟大小和資料夾大小來分配資料。 它將選取最適合物件大小的磁碟。 資料上傳至儲存體帳戶時將併回指定的目錄結構。

#### <a name="where-can-i-find-previous-version-of-waimportexport-tool"></a>哪裡可以找到舊版的 WAImportExport 工具？

WAImportExport 工具擁有 WAImportExport V1 工具的所有功能。 WAImportExport 工具可讓使用者指定多個來源並寫入多個磁碟機。 此外，使用者可以輕鬆地管理多個來源位置 (資料必須從這些來源位置複製到單一 CSV 檔案)。 不過，萬一您需要 SAS 支援或想要將單一來源複製到單一磁碟，您可以 [下載 WAImportExport V1 工具] (http://go.microsoft.com/fwlink/?LinkID=301900&amp;clcid=0x409)，並參閱 [WAImportExport V1 參考](storage-import-export-tool-how-to-v1.md) 以取得 WAImportExport V1 使用方式的說明。

#### <a name="what-is-a-session-id"></a>什麼是工作階段識別碼？

如果目的是要將資料分散到多個磁碟，此工具會預期複製工作階段 (/id) 參數應相同。 維持相同的複製工作階段名稱可讓使用者將資料從一或多個來源位置複製到一或多個目的地磁碟/目錄。 維持相同的工作階段識別碼可讓工具重新找到上次處理的檔案複本。

不過，相同的複製工作階段不能用於匯入資料到不同的儲存體帳戶。

複製工作階段名稱在工具多次執行之間都相同時，記錄檔 (/logdir) 和儲存體帳戶金鑰 (/sk) 也應該相同。

工作階段識別碼可以包含字母、0~9、底線 (\_)、虛線 (-) 或井字鍵 (#)，且其長度必須是 3~30。

例如︰session-1 或 session#1 或 session\_1

#### <a name="what-is-a-journal-file"></a>什麼是日誌檔案？

每次執行 WAImportExport 工具以將檔案複製到硬碟時，工具會建立一個複製工作階段。 複製工作階段的狀態會寫入日誌檔案。 如果複製工作階段中斷 (例如，因為系統電源斷電)，可以再次執行工具並在命令列指定日誌檔案以繼續。

對於您使用 Azure 匯入/匯出工具準備的每個硬碟，此工具會建立名為 "&lt;DriveID&gt;.xml" 的單一日誌檔案，其中的磁碟機識別碼是與工具從磁碟讀取的磁碟機相關聯的序號。 所有磁碟機都將需要日誌檔案，才能建立匯入工作。 如果工具中斷，日誌檔案也可用來繼續磁碟機準備。

#### <a name="what-is-a-log-directory"></a>什麼是記錄檔目錄？

記錄檔目錄指定要用來儲存詳細資訊記錄檔和暫存資訊清單檔案的目錄。 如未指定，將使用目前的目錄做為記錄檔目錄。 記錄是詳細資訊記錄。

### <a name="prerequisites"></a>必要條件

#### <a name="what-are-the-specifications-of-my-disk"></a>什麼是磁碟的規格？

一或多個連接至複製電腦的空白 2.5 英吋或 3.5 英吋 SATAII 或 III 或 SSD 硬碟。

#### <a name="how-can-i-enable-bitlocker-on-my-machine"></a>如何啟用我電腦上的 BitLocker？

簡易的檢查方式是以滑鼠右鍵按一下系統磁碟機上。 它會顯示 Bitlocker 的選項 (如果此功能已開啟)。 如果此功能已關閉，您就不會看到它。

![檢查 BitLocker](./media/storage-import-export-tool-preparing-hard-drives-import/BitLocker.png)

這裡有關於[如何啟用 BitLocker](https://technet.microsoft.com/library/cc766295.aspx)的文章

您的電腦很可能沒有 TPM 晶片。 如果您使用 tpm.msc 並未取得輸出，請查看下一個常見問題集。

#### <a name="how-to-disable-trusted-platform-module-tpm-in-bitlocker"></a>如何停用 BitLocker 中受信任的平台模組 (TPM)？
> [!NOTE]
> 只有當他們的伺服器中沒有 TPM 時，您才需要停用 TPM 原則。 如果使用者的伺服器中具有受信任的 TPM，就不需要停用 TPM。 
> 

若要停用 BitLocker 中的 TPM，請遵循下列步驟︰<br/>
1. 在命令提示字元中輸入 gpedit.msc 以啟動**群組原則編輯器**。 如果**群組原則編輯器**似乎無法使用，請先啟用 BitLocker。 請參閱先前的常見問題集。
2. 開啟 [本機電腦原則] &gt;[電腦設定] &gt; [系統管理範本] &gt; [Windows 元件] &gt; [BitLocker 磁碟機加密] &gt; [作業系統磁碟機]。
3. 編輯 [啟動時需要其他驗證] 原則。
4. 將原則設為 [啟用]，確定已核取 [在不含相容 TPM 的情形下允許使用 BitLocker]。

####  <a name="how-to-check-if-net-4-or-higher-version-is-installed-on-my-machine"></a>如何檢查電腦上是否已安裝 .NET 4 或更新版本？

所有 Microsoft .NET Framework 版本均會安裝在下列目錄︰%windir%\Microsoft.NET\Framework\

在您的目標電腦上，瀏覽至執行工具所需的上述組件所在的位置。 尋找開頭為 "v4" 的資料夾名稱。 如果此目錄不存在，表示您的電腦上並未安裝 .NET 4。 您可以使用 [Microsoft .NET Framework 4 (Web 安裝程式)](https://www.microsoft.com/download/details.aspx?id=17851) 在您的電腦上下載 .Net 4。

### <a name="limits"></a>限制

#### <a name="how-many-drives-can-i-preparesend-at-the-same-time"></a>一次可以準備/傳送多少個磁碟機？

此工具可以準備的磁碟數目沒有限制。 不過，此工具需要磁碟機代號做為輸入。 因此可同時準備的磁碟限制為 25 個。 單一工作一次最多可處理 10 個磁碟。 如果您有 10 個以上的磁碟都對應同一儲存體帳戶，磁碟可以分散到多個工作。

#### <a name="can-i-target-more-than-one-storage-account"></a>可以對應一個以上的儲存體帳戶嗎？

每個工作只能提交一個儲存體帳戶，且一個儲存體帳戶只能用於單一複製工作階段。

### <a name="capabilities"></a>功能

#### <a name="does-waimportexportexe-encrypt-my-data"></a>WAImportExport.exe 是否會加密資料？

是。 此程序會啟用必須的 BitLocker 加密。

#### <a name="what-will-be-the-hierarchy-of-my-data-when-it-appears-in-the-storage-account"></a>資料在儲存體帳戶中會以何種階層顯示？

雖然資料分散到多個磁碟，但上傳至儲存體帳戶時將併回在資料集 CSV 檔案中指定的目錄結構。

#### <a name="how-many-of-the-input-disks-will-have-active-io-in-parallel-when-copy-is-in-progress"></a>當複製進行時，將有多少輸入磁碟會平行擁有作用中 IO？

工具會依據輸入檔案的大小，將資料分散到多個輸入磁碟。 也就是說，平行的作用中磁碟數完全取決於輸入資料的性質。 根據輸入資料集中個別檔案的大小，一或多個磁碟可能會平行顯示作用中 IO。 如需詳細資料，請參閱下一個問題。

#### <a name="how-does-the-tool-distribute-the-files-across-the-disks"></a>工具如何將檔案分散於多個磁碟？

WAImportExport 工具會以批次方式讀取和寫入檔案，一個批次最多包含 100000 個檔案。 這表示最多可以平行寫入 100000 個檔案。 如果將這 100000 個檔案分散到多個磁碟機，即會同時寫入到多個磁碟。 不過，工具要同時寫入到多個磁碟或單一磁碟，取決於批次的累計大小。 例如，當檔案較小時，如果這 10,0000 個檔案可以全部容納於單一磁碟機，工具在此批次的處理期間將只會寫入一個磁碟。

### <a name="waimportexport-output"></a>WAImportExport 輸出

#### <a name="there-are-two-journal-files-which-one-should-i-upload-to-azure-portal"></a>有兩個日誌檔案，我應該將哪一個上傳至 Azure 入口網站？

**.xml** - 對於您使用 WAImportExport 工具準備的每個硬碟，此工具會建立名為 `<DriveID>.xml` 的單一日誌檔案，其中的磁碟機識別碼是與工具從磁碟讀取的磁碟機相關聯的序號。 所有磁碟機都將需要日誌檔案，才能在 Azure 入口網站中建立匯入工作。 如果工具中斷，此日誌檔案也可用來繼續磁碟機準備。

**.jrn** - 後置詞為 `.jrn` 的日誌檔案包含硬碟所有複製工作階段的狀態。 它也包含建立匯入工作所需的資訊。 執行 WAImportExport 工具時，永遠必須指定日誌檔案，以及複製工作階段識別碼。

## <a name="next-steps"></a>後續步驟

* [設定 Azure 匯入/匯出工具](storage-import-export-tool-setup.md)
* [在匯入程序期間設定屬性和中繼資料](storage-import-export-tool-setting-properties-metadata-import.md)
* [針對匯入作業準備硬碟的簡單工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow.md)
* [常用命令快速參考](storage-import-export-tool-quick-reference.md) 
* [利用複製記錄檔檢閱作業狀態](storage-import-export-tool-reviewing-job-status-v1.md)
* [修復匯入作業](storage-import-export-tool-repairing-an-import-job-v1.md)
* [修復匯出作業](storage-import-export-tool-repairing-an-export-job-v1.md)
* [針對 Azure 匯入/匯出工具進行疑難排解](storage-import-export-tool-troubleshooting-v1.md)
