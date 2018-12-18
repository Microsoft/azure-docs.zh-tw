---
title: 修復 Azure 匯入/匯出匯出作業 - v1 | Microsoft Docs
description: 了解如何修復使用 Azure 匯入/匯出服務建立和執行的匯出作業。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: ef5a5f81c5eb3994f62469139c6e835bd802eaa9
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522718"
---
# <a name="repairing-an-export-job"></a>修復匯出作業
完成匯出作業之後，您可以在內部部署上執行 Microsoft Azure 匯入/匯出工具，以便：  
  
1.  下載 Azure 匯入/匯出服務無法匯出的檔案。  
  
2.  驗證磁碟機上已正確匯出的檔案。  
  
您必須能夠連線到 Azure 儲存體，才能使用這項功能。  
  
用於修復匯出作業的命令是 **RepairExport**。

## <a name="repairexport-parameters"></a>RepairExport 參數

您可以搭配 **RepairExport** 指定下列參數：  
  
|參數|說明|  
|---------------|-----------------|  
|**/r:<RepairFile\>**|必要。 修復檔案的路徑，可追蹤修復進度，並可讓您繼續中斷的修復。 每個磁碟機必須只能有一個修復檔案。 當您啟動指定磁碟機的修復時，您將會傳入尚不存在之修復檔的路徑。 若要恢復中斷的修復，您應傳入現有修復檔案名稱。 一律必須指定對應於目標磁碟機的修復檔。|  
|**/logdir:\><LogDirectory**|選用。 記錄檔目錄。 詳細資訊記錄檔會寫入至這個目錄。 如未指定記錄檔目錄，則會使用目前的目錄做為記錄檔目錄。|  
|**/d:<TargetDirectory\>**|必要。 要驗證及修復的目錄。 這通常是匯出磁碟機的根目錄，但也可能是含有匯出檔案複本的網路檔案共用。|  
|**/bk:<BitLockerKey\>**|選用。 如果您希望該工具解除鎖定存放匯出檔案的加密磁碟機，您應指定 BitLocker 金鑰。|  
|**/sn:<StorageAccountName\>**|必要。 匯出作業的儲存體帳戶名稱。|  
|**/sk:<StorageAccountKey\>**|如果未指定 (且只有在未指定) 容器 SAS 時，才是**必要**參數。 匯出作業之儲存體帳戶的帳戶金鑰。|  
|**/csas:<ContainerSas\>**|如果未指定 (且只有在未指定) 儲存體帳戶金鑰時，才是**必要**參數。 存取與匯出作業相關聯的 Blob 所用的容器 SAS。|  
|**/CopyLogFile:<DriveCopyLogFile\>**|必要。 磁碟機複製記錄檔的路徑。 此檔案是由 Windows Azure 匯入/匯出服務所產生，您可以從與作業相關聯的 blob 儲存體下載。 複製記錄檔包含所要修復之失敗 blob 或檔案的相關資訊。|  
|**/ManifestFile:<DriveManifestFile\>**|選用。 匯出磁碟機的資訊清單檔案路徑。 此檔案是由 Windows Azure 匯入/匯出服務所產生並儲存在匯出磁碟機上，並選擇性地儲存在與作業相關聯之儲存體帳戶的 blob 中。<br /><br /> 工具將會使用此檔案內含的 MD5 雜湊，驗證匯出磁碟機上的檔案內容。 任何被斷定為損毀的檔案都將會下載並重新寫入至目標目錄。|  
  
## <a name="using-repairexport-mode-to-correct-failed-exports"></a>使用 RepairExport 模式來更正失敗的匯出  
您可以使用 Azure 匯入/匯出工具來下載無法匯出的檔案。 複製記錄檔會包含無法匯出的檔案清單。  
  
匯出失敗的原因包括下列可能性︰  
  
-   損壞的磁碟機  
  
-   儲存體帳戶金鑰會在移轉過程中變更  
  
若要在 **RepairExport** 模式中執行此工具，您必須先將含有匯出檔案的磁碟機連接到您的電腦。 接下來，執行 Azure 匯入/匯出工具，並使用 `/d` 參數指定該磁碟機的路徑。 您也必須指定您下載之磁碟機複製記錄檔的路徑。 下列命令行範例會執行工具，以修復任何無法匯出的檔案︰  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log  
```  
  
以下的複製記錄檔範例顯示 blob 中無法匯出的一個區塊︰  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
  <DriveId>9WM35C2V</DriveId>  
  <Blob Status="CompletedWithErrors">  
    <BlobPath>pictures/wild/desert.jpg</BlobPath>  
    <FilePath>\pictures\wild\desert.jpg</FilePath>  
    <LastModified>2012-09-18T23:47:08Z</LastModified>  
    <Length>163840</Length>  
    <BlockList>  
      <Block Offset="65536" Length="65536" Id="AQAAAA==" Status="Failed" />  
    </BlockList>  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```  
  
此複製記錄檔指出當 Windows Azure 匯入/匯出服務將其中一個 blob 區塊下載至匯出磁碟機上的檔案時所發生的錯誤。 已成功下載檔案的其他元件，並已正確設定檔案長度。 在此情況下，工具會開啟磁碟機上的這個檔案，從儲存體帳戶下載此區塊，並將它寫入至從位移 65536 開始且長度為 65536 的檔案範圍。  
  
## <a name="using-repairexport-to-validate-drive-contents"></a>使用 RepairExport 驗證磁碟機內容  
您也可以使用 Azure 匯入/匯出服務搭配 **RepairExport** 選項，以驗證磁碟機上的內容是否正確。 每個匯出磁碟機上的資訊清單檔案都包含磁碟機內容適用的 MD5。  
  
Azure 匯入/匯出服務也可以在匯出期間將資訊清單檔案儲存到儲存體帳戶。 當作業完成時，可透過 [Get Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業，取得資訊清單檔案的位置。 如需磁碟機資訊清單檔案的格式，請參閱[匯入/匯出服務資訊清單檔案格式](storage-import-export-file-format-metadata-and-properties.md)。  
  
下列範例示範如何搭配 **/ManifestFile** 和 **/CopyLogFile** 參數執行 Azure 匯入/匯出工具：  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log /ManifestFile:G:\9WM35C3U.manifest  
```  
  
以下是資訊清單檔案的範例：  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveManifest Version="2011-10-01">  
  <Drive>  
    <DriveId>9WM35C3U</DriveId>  
    <ClientCreator>Windows Azure Import/Export service</ClientCreator>  
    <BlobList>
      <Blob>  
        <BlobPath>pictures/city/redmond.jpg</BlobPath>  
        <FilePath>\pictures\city\redmond.jpg</FilePath>  
        <Length>15360</Length>  
        <PageRangeList>  
          <PageRange Offset="0" Length="3584" Hash="72FC55ED9AFDD40A0C8D5C4193208416" />  
          <PageRange Offset="3584" Length="3584" Hash="68B28A561B73D1DA769D4C24AA427DB8" />  
          <PageRange Offset="7168" Length="512" Hash="F521DF2F50C46BC5F9EA9FB787A23EED" />  
        </PageRangeList>  
        <PropertiesPath Hash="E72A22EA959566066AD89E3B49020C0A">\pictures\city\redmond.jpg.properties</PropertiesPath>  
      </Blob>  
      <Blob>  
        <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
        <FilePath>\pictures\wild\canyon.jpg</FilePath>  
        <Length>10884</Length>  
        <BlockList>  
          <Block Offset="0" Length="2721" Id="AAAAAA==" Hash="263DC9C4B99C2177769C5EBE04787037" />  
          <Block Offset="2721" Length="2721" Id="AQAAAA==" Hash="0C52BAE2CC20EFEC15CC1E3045517AA6" />  
          <Block Offset="5442" Length="2721" Id="AgAAAA==" Hash="73D1CB62CB426230C34C9F57B7148F10" />  
          <Block Offset="8163" Length="2721" Id="AwAAAA==" Hash="11210E665C5F8E7E4F136D053B243E6A" />  
        </BlockList>  
        <PropertiesPath Hash="81D7F81B2C29F10D6E123D386C3A4D5A">\pictures\wild\canyon.jpg.properties</PropertiesPath>  
      </Blob> 
    </BlobList>  
 </Drive>  
</DriveManifest>  
``` 
  
完成修復程序之後，工具會讀取資訊清單檔案中參考的每個檔案，並以 MD5 雜湊驗證檔案的完整性。 在上述資訊清單中，將會逐步進行下列元件。  

```  
G:\pictures\city\redmond.jpg, offset 0, length 3584  
  
G:\pictures\city\redmond.jpg, offset 3584, length 3584  
  
G:\pictures\city\redmond.jpg, offset 7168, length 3584  
  
G:\pictures\city\redmond.jpg.properties  
  
G:\pictures\wild\canyon.jpg, offset 0, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 2721, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 5442, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 8163, length 2721  
  
G:\pictures\wild\canyon.jpg.properties  
```

工具將會下載驗證失敗的所有元件，並將它們重新寫入至磁碟機上的相同檔案。  
  
## <a name="next-steps"></a>後續步驟
 
* [設定 Azure 匯入/匯出工具](storage-import-export-tool-setup-v1.md)   
* [針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [利用複製記錄檔檢閱作業狀態](storage-import-export-tool-reviewing-job-status-v1.md)   
* [修復匯入作業](storage-import-export-tool-repairing-an-import-job-v1.md)   
* [針對 Azure 匯入/匯出工具進行疑難排解](storage-import-export-tool-troubleshooting-v1.md)
