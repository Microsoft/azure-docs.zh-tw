---
title: 預覽 Azure 匯入/匯出作業的磁碟機使用量 - v1 | Microsoft Docs
description: 了解如何預覽您已針對 Azure 匯入/匯出服務中的匯出作業選取的 blob 清單。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 21c0fd9b258100e769172332713769024fb12969
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39520566"
---
# <a name="previewing-drive-usage-for-an-export-job"></a>預覽匯出作業的磁碟機使用量
建立匯出作業之前，您必須先選擇一組要匯出的 blob。 Microsoft Azure 匯入/匯出服務可讓您使用一份 blob 路徑或 blob 前置詞清單來代表您已選取的 blob。  
  
接著，必須判斷您需要傳送多少個磁碟機。 匯入/匯出工具提供的 `PreviewExport` 命令可根據您要使用的磁碟機大小，預覽所選取 Blob 的磁碟機使用量。

## <a name="command-line-parameters"></a>命令列參數

使用匯入/匯出工具的 `PreviewExport` 命令時，您可以使用下列參數。

|命令列參數|說明|  
|--------------------------|-----------------|  
|**/logdir:**<LogDirectory\>|選用。 記錄檔目錄。 詳細資訊記錄檔會寫入至這個目錄。 如未指定記錄檔目錄，則會使用目前的目錄做為記錄檔目錄。|  
|**/sn:**<StorageAccountName\>|必要。 匯出作業的儲存體帳戶名稱。|  
|**/sk:**<StorageAccountKey\>|如果未指定 (且只有在未指定) 容器 SAS 時，才是必要參數。 匯出作業之儲存體帳戶的帳戶金鑰。|  
|**/csas:**<ContainerSas\>|如果未指定 (且只有在未指定) 儲存體帳戶金鑰時，才是必要參數。 容器 SAS，可供列出要在匯出作業中匯出的 blob。|  
|**/ExportBlobListFile:**<ExportBlobListFile\>|必要。 XML 檔案的路徑，此檔案包含要匯出的 Blob 的Blob 路徑清單或 Blob 路徑前置詞。 匯入/匯出服務 REST API 的 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業中 `BlobListBlobPath` 元素中所使用的檔案格式。|  
|**/DriveSize:**<DriveSize\>|必要。 要用於匯出作業的磁碟機大小，例如 500GB、1.5TB。|  

## <a name="command-line-example"></a>命令列範例

下列範例示範 `PreviewExport` 命令：  
  
```  
WAImportExport.exe PreviewExport /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /ExportBlobListFile:C:\WAImportExport\mybloblist.xml /DriveSize:500GB    
```  
  
匯出 blob 清單檔案可能包含 blob 名稱和 blob 前置詞，如下所示︰  
  
```xml 
<?xml version="1.0" encoding="utf-8"?>  
<BlobList>  
<BlobPath>pictures/animals/koala.jpg</BlobPath>  
<BlobPathPrefix>/vhds/</BlobPathPrefix>  
<BlobPathPrefix>/movies/</BlobPathPrefix>  
</BlobList>  
```

Azure 匯入/匯出工具會列出所有要匯出的 blob 並計算如何將它們封裝到指定大小的磁碟機，並考量任何必要的額外負荷，然後估計保留 blob 和磁碟機使用量資訊所需的磁碟機數目。  
  
以下輸出範例，以中省略資訊記錄檔︰  
  
```  
Number of unique blob paths/prefixes:   3  
Number of duplicate blob paths/prefixes:        0  
Number of nonexistent blob paths/prefixes:      1  
  
Drive size:     500.00 GB  
Number of blobs that can be exported:   6  
Number of blobs that cannot be exported:        2  
Number of drives needed:        3  
        Drive #1:       blobs = 1, occupied space = 454.74 GB  
        Drive #2:       blobs = 3, occupied space = 441.37 GB  
        Drive #3:       blobs = 2, occupied space = 131.28 GB    
```  
  
## <a name="next-steps"></a>後續步驟

* [Azure 匯入/匯出工具參考](../storage-import-export-tool-how-to-v1.md)
