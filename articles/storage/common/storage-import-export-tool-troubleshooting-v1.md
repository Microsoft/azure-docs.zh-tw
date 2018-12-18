---
title: 針對 Azure 匯入/匯出工具進行疑難排解 | Microsoft Docs
description: 了解使用 Azure 匯入/匯出工具時見到的一些常見問題及其處理方式。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 58ba44488e8ef211e7c318fc9ba6497a5b1b69bb
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523269"
---
# <a name="troubleshooting-the-azure-importexport-tool"></a>針對 Azure 匯入/匯出工具進行疑難排解
如果發生問題，Microsoft Azure 匯入/匯出工具會傳回錯誤訊息。 本主題列出使用者可能會遇到的一些常見問題。  
  
## <a name="a-copy-session-fails-what-i-should-do"></a>複製工作階段失敗，我該怎麼做？  
 當複製工作階段失敗時，有兩個選項︰  
  
 如果錯誤可以重試 (例如，網路共用曾短暫離線，而現已上線)，您可以繼續複製工作階段。 如果錯誤不可重試 (例如，您在命令列參數中指定了錯誤的來源檔案目錄)，您需要中止複製工作階段。 如需有關繼續和中止複製工作階段的詳細資訊，請參閱[針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)。  
  
## <a name="i-cant-resume-or-abort-a-copy-session"></a>我無法繼續或中止複製工作階段。  
 如果複製工作階段是磁碟機的第一個複製工作階段，則錯誤訊息應如下所示：「第一個複製工作階段無法繼續或中止。」 在此情況下，您可以刪除舊的日誌檔，然後重新執行命令。  
  
 如果複製工作階段不是磁碟機的第一個複製工作階段，則可以永遠繼續或中止。  
  
## <a name="i-lost-the-journal-file-can-i-still-create-the-job"></a>我遺失了日誌檔，還可以建立作業嗎？  
 磁碟機的日誌檔包含將資料複製到這個磁碟機的完整資訊，而且必須將更多檔案新增到磁碟機並將用來建立匯入作業。 如果日誌檔案遺失，您必須針對磁碟機重做所有的複製工作階段。  
  
## <a name="next-steps"></a>後續步驟
 
* [設定 Azure 匯入/匯出工具](../storage-import-export-tool-setup-v1.md)   
* [針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [利用複製記錄檔檢閱作業狀態](../storage-import-export-tool-reviewing-job-status-v1.md)   
* [修復匯入作業](../storage-import-export-tool-repairing-an-import-job-v1.md)   
* [修復匯出作業](../storage-import-export-tool-repairing-an-export-job-v1.md)
