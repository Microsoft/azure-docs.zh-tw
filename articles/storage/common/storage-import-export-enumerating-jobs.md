---
title: 所有 Azure 匯入/匯出作業的清單 | MicrosoftDocs
description: 了解如何列出訂用帳戶中所有「Azure 匯入/匯出」服務作業的清單。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 69daac71b69969a7ad9acfeb7095053f8138bf53
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39520875"
---
# <a name="enumerating-jobs-in-the-azure-importexport-service"></a>列舉 Azure 匯入/匯出服務中的作業
若要列舉訂用帳戶中的所有作業，請呼叫 [List Jobs](/rest/api/storageimportexport/jobs#Jobs_List)作業。 `List Jobs` 會傳回一份作業以及下列屬性︰

-   作業的類型 (匯入或匯出)

-   目前的作業狀態

-   作業的相關聯儲存體帳戶

## <a name="next-steps"></a>後續步驟

* [使用匯入/匯出服務 REST API](storage-import-export-using-the-rest-api.md)
