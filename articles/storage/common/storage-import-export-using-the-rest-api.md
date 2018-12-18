---
title: 使用 Azure 匯入/匯出服務 REST API | Microsoft Docs
description: 了解可以使用 Azure 匯入/匯出服務 REST API 在何處尋找資源，包括使用說明及參考資料。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 3775a77a6dfc590e79e785e1604226c1187952de
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528403"
---
# <a name="using-the-azure-importexport-service-rest-api"></a>了解如何使用 Azure 匯入/匯出服務 REST API

Microsoft Azure 匯入/匯出服務會公開 REST API，以啟用程式設計方式控制匯入/匯出作業。 您可以使用 REST API 來執行您可以使用 [Azure 入口網站](https://portal.azure.com/)執行的所有匯入/匯出作業。 此外，您可以使用 REST API 來執行某些細微作業，例如查詢作業的完成百分比，此作業目前在 Azure 入口網站中無法使用。

請參閱[使用 Microsoft Azure 匯入/匯出服務將資料移轉至 Blob 儲存體](../storage-import-export-service.md)以取得匯入/匯出服務概觀及示範如何使用入口網站建立和管理匯入與匯出作業的教學課程。

## <a name="service-endpoints"></a>服務端點

Azure 匯入/匯出服務是 Azure Resource Manager 的資源提供者，並在下列 HTTPS 端點提供一組 REST API 以管理匯入/匯出作業︰

```
https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ImportExport/jobs/<job-name>
```

## <a name="versioning"></a>版本控制

匯入/匯出服務的要求必須指定 `api-version` 參數並將其值設定為 `2016-11-01`。

## <a name="importexport-service-operations"></a>匯入/匯出服務作業

[建立匯入作業](../storage-import-export-creating-an-import-job.md)

[建立匯出作業](../storage-import-export-creating-an-export-job.md)

[擷取作業的狀態資訊](storage-import-export-retrieving-state-info-for-a-job.md)

[列舉作業](../storage-import-export-enumerating-jobs.md)

[取消和刪除作業](storage-import-export-cancelling-and-deleting-jobs.md)

[備份磁碟機資訊清單](../storage-import-export-backing-up-drive-manifests.md)

[匯入/匯出作業的診斷和錯誤復原](../storage-import-export-diagnostics-and-error-recovery.md)

## <a name="next-steps"></a>後續步驟

* [儲存體匯入/匯出 REST](/rest/api/storageimportexport)
