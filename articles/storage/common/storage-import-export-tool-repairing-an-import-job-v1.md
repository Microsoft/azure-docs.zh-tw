---
title: 修復 Azure 匯入/匯出匯入作業 - v1 | Microsoft Docs
description: 了解如何修復使用 Azure 匯入/匯出服務建立和執行的匯入作業。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: e2eb580df0a90e07e79c7f080ba31e5418fc5956
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523779"
---
# <a name="repairing-an-import-job"></a>修復匯入作業
Microsoft Azure 匯入/匯出服務可能無法將某些檔案或某個檔案的部分複製到 Windows Azure Blob 服務。 部分的失敗原因包括︰  
  
-   損毀的檔案  
  
-   損壞的磁碟機  
  
-   儲存體帳戶金鑰在傳輸檔案時變更。  
  
您可以使用匯入作業的複製記錄檔執行 Microsoft Azure 匯入/匯出工具，而此工具會將遺漏的檔案 (或檔案的部分) 上傳到您的 Windows Azure 儲存體帳戶，以完成匯入作業。  
  
## <a name="repairimport-parameters"></a>RepairImport 參數

您可以搭配 **RepairImport** 指定下列參數： 
  
|||  
|-|-|  
|**/r:**<RepairFile\>|**必要。** 修復檔案的路徑，可追蹤修復進度，並可讓您繼續中斷的修復。 每個磁碟機需要一個修復檔案，而且只能有一個。 當您啟動指定磁碟機的修復時，請傳入尚不存在之修復檔的路徑。 若要恢復中斷的修復，您應傳入現有修復檔案名稱。 一律必須指定對應於目標磁碟機的修復檔。|  
|**/logdir:**<LogDirectory\>|**選用。** 記錄檔目錄。 詳細資訊記錄檔會寫入至這個目錄。 如未指定記錄檔目錄，則會使用目前的目錄作為記錄檔目錄。|  
|**/d:**<TargetDirectories\>|**必要。** 一或多個以分號分隔的目錄，其中包含所匯入的原始檔。 也可以使用匯入磁碟機，但如果有原始檔的替代位置可用，則非必要。|  
|**/bk:**<BitLockerKey\>|**選用。** 如果您希望該工具解除鎖定原始檔案所在的加密磁碟機，您應指定 BitLocker 金鑰。|  
|**/sn:**<StorageAccountName\>|**必要。** 匯入作業的儲存體帳戶名稱。|  
|**/sk:**<StorageAccountKey\>|如果未指定 (且只有在未指定) 容器 SAS 時，才是**必要**參數。 匯入作業之儲存體帳戶的帳戶金鑰。|  
|**/csas:**<ContainerSas\>|如果未指定 (且只有在未指定) 儲存體帳戶金鑰時，才是**必要**參數。 存取與匯入作業相關聯的 Blob 所用的容器 SAS。|  
|**/CopyLogFile:**<DriveCopyLogFile\>|**必要。** 磁碟機複製記錄檔 (詳細資訊記錄檔或錯誤記錄檔) 的路徑。 此檔案是由 Windows Azure 匯入/匯出服務所產生，您可以從與作業相關聯的 blob 儲存體下載。 複製記錄檔包含所要修復之失敗 blob 或檔案的相關資訊。|  
|**/PathMapFile:**<DrivePathMapFile\>|**選用。** 如果您在相同作業中匯入多個同名的檔案，則為可用來解決模稜兩可情形之文字檔的路徑。 第一次執行此工具時，它可以在此檔案中填入所有模稜兩可的名稱。 後續執行此工具時，會使用此檔案來解決模稜兩可情形。|  
  
## <a name="using-the-repairimport-command"></a>使用 RepairImport 命令  
若要透過網路串流資料來修復匯入資料，您必須指定內含您使用 `/d` 參數匯入之原始檔的目錄。 您也必須指定您從儲存體帳戶下載的複製記錄檔。 用於修復部分失敗之匯入作業的典型命令行如下所示︰  
  
```  
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log  
```  
  
在以下的複製記錄檔範例中，針對匯入作業送交的磁碟機上有一個 64K 的檔案片段已損毀。 這是唯一的失敗處，所以已順利匯入作業中其餘的 blob。  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
 <DriveId>9WM35C2V</DriveId>  
 <Blob Status="CompletedWithErrors">  
 <BlobPath>pictures/animals/koala.jpg</BlobPath>  
 <FilePath>\animals\koala.jpg</FilePath>  
 <Length>163840</Length>  
 <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
 <PageRangeList>  
  <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted" />  
 </PageRangeList>  
 </Blob>  
 <Status>CompletedWithErrors</Status>  
</DriveLog>  
```
  
將此複製記錄檔傳遞至 Azure 匯入/匯出工具時，此工具會嘗試透過網路複製遺漏的內容來完成此檔案匯入。 在上述範例之後，此工具會在 `C:\Users\bob\Pictures` 和 `X:\BobBackup\photos` 兩個目錄中尋找原始檔案 `\animals\koala.jpg`。 如果檔案 `C:\Users\bob\Pictures\animals\koala.jpg` 存在，則 Azure 匯入/匯出工具會將遺漏的資料範圍複製到對應的 blob `http://bobmediaaccount.blob.core.windows.net/pictures/animals/koala.jpg`。  
  
## <a name="resolving-conflicts-when-using-repairimport"></a>解決使用 RepairImport 時的衝突  
在某些情況下，基於下列原因之一，此工具可能無法找到或開啟所需的檔案︰找不到檔案、無法存取檔案、檔案名稱模稜兩可，或檔案的內容已不正確。  
  
如果此工具嘗試找出 `\animals\koala.jpg`，而 `C:\Users\bob\pictures` 和 `X:\BobBackup\photos` 之下同時有該名稱的檔案，則可能會發生模稜兩可錯誤。 也就是說，`C:\Users\bob\pictures\animals\koala.jpg` 和 `X:\BobBackup\photos\animals\koala.jpg` 兩者都存在於匯入作業磁碟機上。  
  
`/PathMapFile` 選項可讓您解決這些錯誤。 您可以指定檔案名稱，其中內含工具無法正確識別的檔案清單。 下列命令列範例會填入 `9WM35C2V_pathmap.txt`：  
  
```
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log /PathMapFile:C:\WAImportExport\9WM35C2V_pathmap.txt  
```
  
工具會接著將有問題的檔案路徑寫入至 `9WM35C2V_pathmap.txt`，一行一個。 比方說，此檔案在執行命令後可能包含下列項目︰  
 
```
\animals\koala.jpg  
\animals\kangaroo.jpg  
```
  
 對於清單中的每個檔案，您應該嘗試找出並開啟檔案，以確保其可供工具使用。 如果您想明確地告訴工具何處尋找檔案，您可以修改路徑對應檔並將路徑新增至同一行上的每個檔案 (以 Tab 字元分隔)︰  
  
```
\animals\koala.jpg           C:\Users\bob\Pictures\animals\koala.jpg  
\animals\kangaroo.jpg        X:\BobBackup\photos\animals\kangaroo.jpg  
```
  
讓必要檔案可供工具使用，或更新路徑對應檔之後，您可以重新執行工具，以完成匯入程序。  
  
## <a name="next-steps"></a>後續步驟
 
* [設定 Azure 匯入/匯出工具](storage-import-export-tool-setup-v1.md)   
* [針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [利用複製記錄檔檢閱作業狀態](storage-import-export-tool-reviewing-job-status-v1.md)   
* [修復匯出作業](../storage-import-export-tool-repairing-an-export-job-v1.md)   
* [針對 Azure 匯入/匯出工具進行疑難排解](storage-import-export-tool-troubleshooting-v1.md)
