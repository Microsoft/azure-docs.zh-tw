---
title: Azure 匯入/匯出工具匯入作業命令的快速參考 - v1 | Microsoft Docs
description: 適用於常用匯入作業命令的 Azure 匯入/匯出工具命令參考。 這是指 v1 的匯入/匯出工具。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 16e0fa6f7336b39f63b00564d37c1be308bebb16
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526278"
---
# <a name="quick-reference-for-frequently-used-commands-for-import-jobs"></a>匯入作業的常用命令快速參考
本章節提供一些常用命令的快速參考。 如需詳細使用方式，請參閱[針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)。  

## <a name="prepare-a-hard-drive-when-data-has-already-been-copied-to-the-hard-drive"></a>當資料已複製到硬碟時，讓硬碟做好準備
 下列命令會在資料已複製到硬碟，但尚未以 BitLocker 加密時，讓硬碟做好準備：  
  
```  
  WAImportExport.exe PrepImport /j:9WM35C2V.jrn /id:session#1 /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /t:d /encrypt /srcdir:d:\movies\drama /dstdir:movies/drama/ /skipwrite
```    

## <a name="copy-a-single-directory-to-a-hard-drive"></a>將單一目錄複製到硬碟機  
 下列命令會將單一來源目錄複製到尚未以 BitLocker 加密的硬碟︰  
  
```  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:movies /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:d:\Movies /dstdir:entertainment/movies/  
```  
  
## <a name="copy-two-directories-to-a-hard-drive"></a>將兩個目錄複製到硬碟  
 若要將兩個來源目錄複製到硬碟，請使用下列命令：  
  
 第一個命令會指定記錄目錄、儲存體帳戶金鑰、目標磁碟機代號和 `format/encrypt` 需求以及一般參數︰  
  
```  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:movies /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:d:\Movies /dstdir:entertainment/movies/  
```  
  
 第二個命令會指定日誌檔案、新的工作階段識別碼，以及來源和目的地位置︰  
  
```  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:music /srcdir:d:\Music /dstdir:entertainment/music/  
```  
  
## <a name="copy-a-large-file-to-a-hard-drive-in-a-second-copy-session"></a>在第二個複製工作階段中，將大型檔案複製到硬碟機  
 下列命令會將單一大型檔案複製到已在前一個複製工作階段中備妥的硬碟︰  
  
```  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:dvd /srcfile:d:\dvd\favoritemovie.vhd /dstblob:dvd/favoritemovie.vhd  
```  
  
## <a name="next-steps"></a>後續步驟

* [針對匯入作業準備硬碟的簡單工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow-v1.md)
