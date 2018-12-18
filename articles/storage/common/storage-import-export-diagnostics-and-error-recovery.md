---
title: Azure 匯入匯出作業的診斷和錯誤復原 | Microsoft Docs
description: 了解如何啟用 Microsoft Azure 匯入/匯出服務作業的詳細資訊記錄。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 2a54752f933b91265d0aa8add61ca0707615931b
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526322"
---
# <a name="diagnostics-and-error-recovery-for-azure-importexport-jobs"></a>Azure 匯入/匯出作業的診斷和錯誤復原
針對每個已處理的磁碟機，Azure 匯入/匯出服務會在相關聯的儲存體帳戶中建立錯誤記錄檔。 您也可以在呼叫 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 或 [Update Job Properties](/rest/api/storageimportexport/jobs#Jobs_Update) 作業時，將 `LogLevel` 屬性設定為 `Verbose` 來啟用詳細資訊記錄。

 根據預設，記錄會寫入名為 `waimportexport` 的容器。 您可以在呼叫 `Put Job` 或 `Update Job Properties` 作業時，設定 `DiagnosticsPath` 屬性以指定不同的名稱。 記錄會使用下列的命名慣例儲存為區塊 blob︰`waies/jobname_driveid_timestamp_logtype.xml`。

 您可以藉由呼叫 [Get Job](/rest/api/storageimportexport/jobs#Jobs_Get) 作業來擷取工作的記錄 URI。 會針對每個磁碟機在 `VerboseLogUri` 屬性中傳回詳細資訊記錄檔的 URI，而在 `ErrorLogUri` 屬性中會傳回錯誤記錄檔的 URI。

您可以使用記錄資料來識別下列問題。

## <a name="drive-errors"></a>磁碟機錯誤

下列項目會歸類為磁碟機錯誤：

-   存取或讀取資訊清單檔案中的錯誤

-   不正確的 BitLocker 金鑰

-   磁碟機讀取/寫入錯誤

## <a name="blob-errors"></a>Blob 錯誤

下列項目會歸類為 blob 錯誤：

-   不正確或衝突的 blob 或名稱

-   遺失的檔案

-   找不到 Blob

-   截斷的檔案 (磁碟上的檔案小於資訊清單中所指定的)

-   損毀的檔案內容 (適用於匯入工作，偵測到與 MD5 總和檢查碼不符)

-   損毀的 blob 中繼資料和屬性檔 (偵測到與 MD5 總和檢查碼不符)

-   blob 屬性及/或中繼資料檔案的結構描述不正確

在某些情況下，匯入或匯出作業的某些部分無法順利完成，而整個工作仍然會完成。 在此情況下，您可以透過網路上傳或下載遺漏的資料片段，或可以建立新的作業來傳送資料。 請參閱 [Azure 匯入/匯出工具參考](storage-import-export-tool-how-to-v1.md)，以了解如何透過網路修復資料。

## <a name="next-steps"></a>後續步驟

* [使用匯入/匯出服務 REST API](storage-import-export-using-the-rest-api.md)
