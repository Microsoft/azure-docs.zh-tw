---
title: Azure 匯入/匯出記錄檔格式 | Microsoft Docs
description: 了解執行匯入/匯出服務作業的步驟時所建立的記錄檔格式。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: b842a80762989c34ae278a397cc49c088ff77fb2
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525513"
---
# <a name="azure-importexport-service-log-file-format"></a>Azure 匯入/匯出服務記錄檔格式
當 Microsoft Azure 匯入/匯出服務於匯入工作或匯出工作期間在磁碟機上執行動作時，記錄檔會寫入至與該工作相關聯的儲存體帳戶中的區塊 Blob。  
  
匯入/匯出服務可覆寫兩個記錄檔︰  
  
-   發生錯誤時，一定會產生錯誤記錄檔。  
  
-   詳細資訊記錄檔預設未啟用，但可透過在 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 或 [Update Job Properties](/rest/api/storageimportexport/jobs#Jobs_Update) 作業上設定 `EnableVerboseLog` 屬性來啟用。  
  
## <a name="log-file-location"></a>記錄檔位置  
記錄檔會寫入至容器中的區塊 Blob 或 `ImportExportStatesPath` 設定指定的虛擬目錄 (您可以在 `Put Job` 作業上設定) 中。 寫入記錄檔的位置取決於為工作指定的驗證方式，以及為 `ImportExportStatesPath` 指定的值。 您可以透過儲存體帳戶金鑰或容器 SAS (共用存取簽章) 指定工作的驗證方式。  
  
容器或虛擬目錄的名稱可以是 `waimportexport` 的預設名稱，或是您指定的其他容器或虛擬目錄名稱。  
  
下表顯示可能的選項︰  
  
|驗證方法|`ImportExportStatesPath` 元素的值|記錄檔 Blob 的位置|  
|---------------------------|----------------------------------------------|---------------------------|  
|儲存體帳戶金鑰|預設值|名為 `waimportexport` 的容器，這是預設容器。 例如︰<br /><br /> `https://myaccount.blob.core.windows.net/waimportexport`|  
|儲存體帳戶金鑰|使用者指定值|使用者命名的容器。 例如︰<br /><br /> `https://myaccount.blob.core.windows.net/mylogcontainer`|  
|容器 SAS|預設值|名為 `waimportexport` 的虛擬目錄，這是預設名稱，位於 SAS 中指定的容器底下。<br /><br /> 例如，如果為工作指定的 SAS 是 `https://myaccount.blob.core.windows.net/mylogcontainer?sv=2012-02-12&se=2015-05-22T06%3A54%3A55Z&sr=c&sp=wl&sig=sigvalue`，記錄檔位置會是 `https://myaccount.blob.core.windows.net/mylogcontainer/waimportexport`|  
|容器 SAS|使用者指定值|使用者命名的虛擬目錄，位於 SAS 中指定的容器底下。<br /><br /> 例如，如果為工作指定的 SAS 是 `https://myaccount.blob.core.windows.net/mylogcontainer?sv=2012-02-12&se=2015-05-22T06%3A54%3A55Z&sr=c&sp=wl&sig=sigvalue`，且指定的虛擬目錄名為 `mylogblobs`，記錄檔位置會是 `https://myaccount.blob.core.windows.net/mylogcontainer/waimportexport/mylogblobs`。|  
  
您可以透過呼叫 [Get Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業來擷取錯誤和詳細資訊記錄檔的 URL。 磁碟機處理完成後會產生記錄檔。  
  
## <a name="log-file-format"></a>記錄檔格式  
兩個記錄檔的格式同為 Blob，其中包含在硬碟和客戶的帳戶之間複製 Blob 時所發生事件的 XML 描述。  
  
詳細資訊記錄檔包含與每個 Blob (匯入工作) 或檔案 (匯出工作) 的複製作業狀態相關的完整資訊，錯誤記錄檔則只包含匯入或匯出工作期間發生錯誤的 Blob 或檔案的資訊。  
  
詳細資訊記錄檔格式顯示如下。 錯誤記錄檔的結構相同，但會篩選出成功的作業。  

```xml
<DriveLog Version="2014-11-01">  
  <DriveId>drive-id</DriveId>  
  [<Blob Status="blob-status">  
   <BlobPath>blob-path</BlobPath>  
   <FilePath>file-path</FilePath>  
   [<Snapshot>snapshot</Snapshot>]  
   <Length>length</Length>  
   [<LastModified>last-modified</LastModified>]  
   [<ImportDisposition Status="import-disposition-status">import-disposition</ImportDisposition>]  
   [page-range-list-or-block-list]  
   [metadata-status]  
   [properties-status]  
  </Blob>]  
  [<Blob>  
    . . .  
  </Blob>]  
  <Status>drive-status</Status>  
</DriveLog>  
  
page-range-list-or-block-list ::= 
  page-range-list | block-list  
  
page-range-list ::=   
<PageRangeList>  
      [<PageRange Offset="page-range-offset" Length="page-range-length"   
       [Hash="md5-hash"] Status="page-range-status"/>]  
      [<PageRange Offset="page-range-offset" Length="page-range-length"   
       [Hash="md5-hash"] Status="page-range-status"/>]  
</PageRangeList>  
  
block-list ::=  
<BlockList>  
      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]  
       [Hash="md5-hash"] Status="block-status"/>]  
      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]   
       [Hash="md5-hash"] Status="block-status"/>]  
</BlockList>  
  
metadata-status ::=  
<Metadata Status="metadata-status">  
   [<GlobalPath Hash="md5-hash">global-metadata-file-path</GlobalPath>]  
   [<Path Hash="md5-hash">metadata-file-path</Path>]  
</Metadata>  
  
properties-status ::=  
<Properties Status="properties-status">  
   [<GlobalPath Hash="md5-hash">global-properties-file-path</GlobalPath>]  
   [<Path Hash="md5-hash">properties-file-path</Path>]  
</Properties>  
```

下表說明記錄檔的元素：  
  
|XML 元素|類型|說明|  
|-----------------|----------|-----------------|  
|`DriveLog`|XML 元素|代表磁碟機記錄檔。|  
|`Version`|屬性、字串|記錄檔格式的版本。|  
|`DriveId`|字串|磁碟機的硬體序號。|  
|`Status`|字串|磁碟機處理狀態。 如需詳細資訊，請參閱以下的 `Drive Status Codes` 表格。|  
|`Blob`|巢狀的 XML 元素|代表 Blob。|  
|`Blob/BlobPath`|字串|Blob 的 URI。|  
|`Blob/FilePath`|字串|磁碟機上檔案的相對路徑。|  
|`Blob/Snapshot`|Datetime|Blob 快照集版本 (限匯出工作)。|  
|`Blob/Length`|整數 |Blob 總長度 (以位元組為單位)。|  
|`Blob/LastModified`|Datetime|上次修改 Blob 的日期/時間 (限匯出工作)。|  
|`Blob/ImportDisposition`|字串|Blob 的匯入配置 (限匯入工作)。|  
|`Blob/ImportDisposition/@Status`|屬性、字串|匯入配置狀態。|  
|`PageRangeList`|巢狀的 XML 元素|代表分頁 Blob 的頁面範圍清單。|  
|`PageRange`|XML 元素|代表頁面範圍。|  
|`PageRange/@Offset`|屬性、整數|Blob 中的頁面範圍起始位移。|  
|`PageRange/@Length`|屬性、整數|頁面範圍的長度 (以位元組為單位)。|  
|`PageRange/@Hash`|屬性、字串|頁面範圍的 Base16 式編碼的 MD5 雜湊。|  
|`PageRange/@Status`|屬性、字串|頁面範圍處理狀態。|  
|`BlockList`|巢狀的 XML 元素|代表區塊 Blob 的區塊清單。|  
|`Block`|XML 元素|代表區塊。|  
|`Block/@Offset`|屬性、整數|Blob 中的區塊起始位移。|  
|`Block/@Length`|屬性、整數|區塊的長度 (以位元組為單位)。|  
|`Block/@Id`|屬性、字串|區塊識別碼。|  
|`Block/@Hash`|屬性、字串|區塊的 Base16 式編碼的 MD5 雜湊。|  
|`Block/@Status`|屬性、字串|區塊處理狀態。|  
|`Metadata`|巢狀的 XML 元素|代表 Blob 的中繼資料。|  
|`Metadata/@Status`|屬性、字串|Blob 中繼資料處理狀態。|  
|`Metadata/GlobalPath`|字串|全域中繼資料檔案的相對路徑。|  
|`Metadata/GlobalPath/@Hash`|屬性、字串|全域中繼資料檔案的 Base16 式編碼的 MD5 雜湊。|  
|`Metadata/Path`|字串|中繼資料檔案的相對路徑。|  
|`Metadata/Path/@Hash`|屬性、字串|中繼資料檔案的 Base16 式編碼的 MD5 雜湊。|  
|`Properties`|巢狀的 XML 元素|表示 Blob 屬性。|  
|`Properties/@Status`|屬性、字串|Blob 屬性處理狀態，例如，找不到檔案、已完成。|  
|`Properties/GlobalPath`|字串|全域屬性檔案的相對路徑。|  
|`Properties/GlobalPath/@Hash`|屬性、字串|全域屬性檔案的 Base16 式編碼的 MD5 雜湊。|  
|`Properties/Path`|字串|屬性檔案的相對路徑。|  
|`Properties/Path/@Hash`|屬性、字串|屬性檔案的 Base16 式編碼的 MD5 雜湊。|  
|`Blob/Status`|字串|Blob 處理狀態。|  
  
# <a name="drive-status-codes"></a>磁碟機狀態碼  
下表列出磁碟機處理的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Completed`|磁碟機已完成處理，沒有任何錯誤。|  
|`CompletedWithWarnings`|磁碟機已完成處理，並根據為 Blob 指定的匯入配置在一或多個 Blob 中發出警告。|  
|`CompletedWithErrors`|磁碟機已完成，一或多個 Blob 或區塊中發生錯誤。|  
|`DiskNotFound`|在磁碟機上找不到磁碟。|  
|`VolumeNotNtfs`|磁碟上的第一個資料磁碟區不是 NTFS 格式。|  
|`DiskOperationFailed`|在磁碟機上執行作業時發生未知的失敗。|  
|`BitLockerVolumeNotFound`|找不到 BitLocker 可加密磁碟區。|  
|`BitLockerNotActivated`|磁碟區上未啟用 BitLocker。|  
|`BitLockerProtectorNotFound`|數字密碼金鑰保護裝置不存在磁碟區上。|  
|`BitLockerKeyInvalid`|提供的數字密碼無法解除鎖定磁碟區。|  
|`BitLockerUnlockVolumeFailed`|嘗試解除鎖定磁碟區時發生未知的失敗。|  
|`BitLockerFailed`|執行 BitLocker 作業時發生未知的失敗。|  
|`ManifestNameInvalid`|資訊清單檔案名稱無效。|  
|`ManifestNameTooLong`|資訊清單檔案名稱太長。|  
|`ManifestNotFound`|找不到資訊清單檔案。|  
|`ManifestAccessDenied`|拒絕存取資訊清單檔案。|  
|`ManifestCorrupted`|資訊清單檔案已損毀 (內容不符合其雜湊)。|  
|`ManifestFormatInvalid`|資訊清單內容不符合所需格式。|  
|`ManifestDriveIdMismatch`|資訊清單檔案中的磁碟機識別碼與從磁碟機讀取的不符。|  
|`ReadManifestFailed`|讀取資訊清單時發生磁碟 I/O 失敗。|  
|`BlobListFormatInvalid`|匯出 Blob 清單 Blob 不符合所需格式。|  
|`BlobRequestForbidden`|禁止存取儲存體帳戶中的 Blob。 這可能是因為儲存體帳戶金鑰或容器 SAS 無效。|  
|`InternalError`|處理磁碟機時發生內部錯誤。|  
  
## <a name="blob-status-codes"></a>Blob 狀態碼  
下表列出 Blob 處理的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Completed`|Blob 已完成處理，沒有任何錯誤。|  
|`CompletedWithErrors`|Blob 已完成處理，一或多個頁面範圍或區塊、 中繼資料或屬性中發生錯誤。|  
|`FileNameInvalid`|檔案名稱無效。|  
|`FileNameTooLong`|檔案名稱太長。|  
|`FileNotFound`|找不到檔案。|  
|`FileAccessDenied`|拒絕存取檔案。|  
|`BlobRequestFailed`|存取 Blob 的 Blob 服務要求失敗。|  
|`BlobRequestForbidden`|禁止存取 Blob 的 Blob 服務要求。 這可能是因為儲存體帳戶金鑰或容器 SAS 無效。|  
|`RenameFailed`|無法重新命名 Blob (匯入工作) 或檔案 (匯出工作)。|  
|`BlobUnexpectedChange`|Blob (匯出工作) 發生未預期的變更。|  
|`LeasePresent`|Blob 上有一個租約。|  
|`IOFailed`|處理 Blob 時發生磁碟或網路 I/O 失敗。|  
|`Failed`|處理 Blob 時發生未知的失敗。|  
  
## <a name="import-disposition-status-codes"></a>匯入配置狀態碼  
下表列出解決匯入配置的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Created`|Blob 已建立。|  
|`Renamed`|Blob 已依照重新命名匯入配置重新命名。 `Blob/BlobPath` 元素包含已重新命名的 Blob 的 URI。|  
|`Skipped`|已根據 `no-overwrite` 匯入配置略過 Blob。|  
|`Overwritten`|Blob 已根據 `overwrite` 匯入配置覆寫現有的 Blob。|  
|`Cancelled`|先前的失敗已停止進一步處理匯入配置。|  
  
## <a name="page-rangeblock-status-codes"></a>頁面範圍/區塊狀態碼  
下表列出頁面範圍或區塊處理的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Completed`|頁面範圍或區塊已完成處理，沒有任何錯誤。|  
|`Committed`|區塊已認可，但不在完整區塊清單中，因為其他區塊失敗，或是放置完整區塊清單本身失敗。|  
|`Uncommitted`|區塊已上傳但尚未認可。|  
|`Corrupted`|頁面範圍或區塊已損毀 (內容不符合其雜湊)。|  
|`FileUnexpectedEnd`|發現非預期的檔案結尾。|  
|`BlobUnexpectedEnd`|發現非預期的 Blob 結尾。|  
|`BlobRequestFailed`|存取頁面範圍或區塊的 Blob 服務要求失敗。|  
|`IOFailed`|處理頁面範圍或區塊時發生磁碟或網路 I/O 失敗。|  
|`Failed`|處理頁面範圍或區塊時發生未知的失敗。|  
|`Cancelled`|先前的失敗已停止進一步處理頁面範圍或區塊。|  
  
## <a name="metadata-status-codes"></a>中繼資料狀態碼  
下表列出 Blob 中繼資料處理的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Completed`|中繼資料已完成處理，沒有任何錯誤。|  
|`FileNameInvalid`|中繼資料檔案名稱無效。|  
|`FileNameTooLong`|中繼資料檔案名稱太長。|  
|`FileNotFound`|找不到中繼資料檔案。|  
|`FileAccessDenied`|拒絕存取中繼資料檔案。|  
|`Corrupted`|中繼資料檔案已損毀 (內容不符合其雜湊)。|  
|`XmlReadFailed`|中繼資料內容不符合所需格式。|  
|`XmlWriteFailed`|寫入中繼資料 XML 失敗。|  
|`BlobRequestFailed`|存取中繼資料的 Blob 服務要求失敗。|  
|`IOFailed`|處理中繼資料時發生磁碟或網路 I/O 失敗。|  
|`Failed`|處理中繼資料時發生未知的失敗。|  
|`Cancelled`|先前的失敗已停止進一步處理中繼資料。|  
  
## <a name="properties-status-codes"></a>屬性狀態碼  
下表列出 Blob 屬性處理的狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|`Completed`|屬性已完成處理，沒有任何錯誤。|  
|`FileNameInvalid`|屬性檔案名稱無效。|  
|`FileNameTooLong`|屬性檔案名稱太長。|  
|`FileNotFound`|找不到屬性檔案。|  
|`FileAccessDenied`|拒絕存取屬性檔案。|  
|`Corrupted`|屬性檔案已損毀 (內容不符合其雜湊)。|  
|`XmlReadFailed`|屬性內容不符合所需格式。|  
|`XmlWriteFailed`|寫入屬性 XML 失敗。|  
|`BlobRequestFailed`|存取屬性的 Blob 服務要求失敗。|  
|`IOFailed`|處理屬性時發生磁碟或網路 I/O 失敗。|  
|`Failed`|處理屬性時發生未知的失敗。|  
|`Cancelled`|先前的失敗已停止進一步處理屬性。|  
  
## <a name="sample-logs"></a>範例記錄  
以下是詳細資訊記錄檔的範例。  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<DriveLog Version="2014-11-01">  
    <DriveId>WD-WMATV123456</DriveId>  
    <Blob Status="Completed">  
       <BlobPath>pictures/bob/wild/desert.jpg</BlobPath>  
       <FilePath>\Users\bob\Pictures\wild\desert.jpg</FilePath>  
       <Length>98304</Length>  
       <ImportDisposition Status="Created">overwrite</ImportDisposition>  
       <BlockList>  
          <Block Offset="0" Length="65536" Id="AAAAAA==" Hash=" 9C8AE14A55241F98533C4D80D85CDC68" Status="Completed"/>  
          <Block Offset="65536" Length="32768" Id="AQAAAA==" Hash=" DF54C531C9B3CA2570FDDDB3BCD0E27D" Status="Completed"/>  
       </BlockList>  
       <Metadata Status="Completed">  
          <GlobalPath Hash=" E34F54B7086BCF4EC1601D056F4C7E37">\Users\bob\Pictures\wild\metadata.xml</GlobalPath>  
       </Metadata>  
    </Blob>  
    <Blob Status="CompletedWithErrors">  
       <BlobPath>pictures/bob/animals/koala.jpg</BlobPath>  
       <FilePath>\Users\bob\Pictures\animals\koala.jpg</FilePath>  
       <Length>163840</Length>  
       <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
       <PageRangeList>  
          <PageRange Offset="0" Length="65536" Hash="19701B8877418393CB3CB567F53EE225" Status="Completed"/>  
          <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted"/>  
          <PageRange Offset="131072" Length="4096" Hash="9BA552E1C3EEAFFC91B42B979900A996" Status="Completed"/>  
       </PageRangeList>  
       <Properties Status="Completed">  
          <Path Hash="38D7AE80653F47F63C0222FEE90EC4E7">\Users\bob\Pictures\animals\koala.jpg.properties</Path>  
       </Properties>  
    </Blob>  
    <Status>CompletedWithErrors</Status>  
</DriveLog>  
```  
  
對應的錯誤記錄檔顯示如下：  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<DriveLog Version="2014-11-01">  
    <DriveId>WD-WMATV6965824</DriveId>  
    <Blob Status="CompletedWithErrors">  
       <BlobPath>pictures/bob/animals/koala.jpg</BlobPath>  
       <FilePath>\Users\bob\Pictures\animals\koala.jpg</FilePath>  
       <Length>163840</Length>  
       <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
       <PageRangeList>  
          <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted"/>  
       </PageRangeList>  
    </Blob>  
    <Status>CompletedWithErrors</Status>  
</DriveLog>  
```

 匯入工作的下列錯誤記錄檔包含在匯入磁碟機上找不到檔案的相關錯誤。 請注意，後續元件的狀態是 `Cancelled`。  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog Version="2014-11-01">  
  <DriveId>9WM35C2V</DriveId>  
  <Blob Status="FileNotFound">  
    <BlobPath>pictures/animals/koala.jpg</BlobPath>  
    <FilePath>\animals\koala.jpg</FilePath>  
    <Length>30310</Length>  
    <ImportDisposition Status="Cancelled">rename</ImportDisposition>  
    <BlockList>  
      <Block Offset="0" Length="6062" Id="MD5/cAzn4h7VVSWXf696qp5Uaw==" Hash="700CE7E21ED55525977FAF7AAA9E546B" Status="Cancelled" />  
      <Block Offset="6062" Length="6062" Id="MD5/PEnGwYOI8LPLNYdfKr7kAg==" Hash="3C49C6C18388F0B3CB35875F2ABEE402" Status="Cancelled" />  
      <Block Offset="12124" Length="6062" Id="MD5/FG4WxqfZKuUWZ2nGTU2qVA==" Hash="146E16C6A7D92AE5166769C64D4DAA54" Status="Cancelled" />  
      <Block Offset="18186" Length="6062" Id="MD5/ZzibNDzr3IRBQENRyegeXQ==" Hash="67389B343CEBDC8441404351C9E81E5D" Status="Cancelled" />  
      <Block Offset="24248" Length="6062" Id="MD5/ZzibNDzr3IRBQENRyegeXQ==" Hash="67389B343CEBDC8441404351C9E81E5D" Status="Cancelled" />  
    </BlockList>  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```

匯出工作的下列錯誤記錄檔指出 Blob 內容已順利寫入至磁碟機，但匯出 Blob 的屬性時發生錯誤。  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog Version="2014-11-01">  
  <DriveId>9WM35C3U</DriveId>  
  <Blob Status="CompletedWithErrors">  
    <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
    <FilePath>\pictures\wild\canyon.jpg</FilePath>  
    <LastModified>2012-09-18T23:47:08Z</LastModified>  
    <Length>163840</Length>  
    <BlockList />  
    <Properties Status="Failed" />  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```
  
## <a name="next-steps"></a>後續步驟
 
* [儲存體匯入/匯出 REST API](/rest/api/storageimportexport/)
