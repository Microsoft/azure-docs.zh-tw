---
title: 建立 Azure 匯入/匯出的匯出作業 | Microsoft Docs
description: 了解如何建立 Microsoft Azure 匯入/匯出服務的匯出作業。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 0066ec20c68d70fac9a6529f8f5b85855d6c5349
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666819"
---
# <a name="creating-an-export-job-for-the-azure-importexport-service"></a>建立 Azure 匯入/匯出服務的匯出作業
使用 REST API 建立 Microsoft Azure 匯入/匯出服務的匯出作業包含下列步驟︰

-   選取要匯出的 blob。

-   取得寄送位置。

-   建立匯出作業。

-   透過支援的貨運服務將您的空磁碟機寄送給 Microsoft。

-   使用封裝資訊更新匯出作業。

-   從 Microsoft 接收磁碟機。

 請參閱[使用 Windows Azure 匯入/匯出服務將資料移轉至 Blob 儲存體](storage-import-export-service.md)以取得匯入/匯出服務概觀及示範如何使用 [Azure 入口網站](https://portal.azure.com/)建立和管理匯入與匯出作業的教學課程。

## <a name="selecting-blobs-to-export"></a>選取要匯出的 blob
 若要建立匯出作業，您必須提供您要從儲存體帳戶匯出的 blob 清單。 有幾種方式可選取要匯出的 blob：

-   您可以使用相對 blob 路徑選取單一 blob 和其所有快照。

-   您可以使用相對 blob 路徑選取單一 blob，但其快照除外。

-   您可以使用相對 blob 路徑和快照時間選取單一快照。

-   您可以使用 blob 首碼選取所有的 blob 和具有指定前置詞的快照。

-   您可以匯出儲存體帳戶中所有的 blob 和快照。

 如需指定要匯出之 blob 的詳細資訊，請參閱 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業。

## <a name="obtaining-your-shipping-location"></a>取得寄送位置
在建立匯出作業之前，您需要藉由呼叫 [Get Location](https://portal.azure.com) 或 [List Locations](https://docs.microsoft.com/rest/api/storageimportexport/locations/list) 作業來取得寄送位置名稱和地址。 `List Locations` 會傳回位置及其郵寄地址的清單。 您可以從傳回的清單中選取位置，並將您的硬碟寄送至該地址。 您也可以使用 `Get Location` 作業來直接取得特定位置的寄送地址。

請遵循下列步驟來取得寄送位置︰

-   識別儲存體帳戶位置的名稱。 您可以在 Azure 入口網站中儲存體帳戶「儀表板」上的 [位置] 欄位下找到此值，或使用服務管理 API 作業[取得儲存體帳戶屬性](/rest/api/storagerp/storageaccounts#StorageAccounts_GetProperties)來查詢此值。

-   藉由呼叫 `Get Location` 作業來擷取可用來處理此儲存體帳戶的位置。

-   如果位置的 `AlternateLocations` 屬性包含位置本身，則可以使用此位置。 否則，使用其中一個替代位置再次呼叫 `Get Location` 作業。 原始位置可能暫時關閉進行維護。

## <a name="creating-the-export-job"></a>建立匯出作業
 若要建立匯出作業，請呼叫 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業。 您必須提供下列資訊：

-   作業的名稱。

-   儲存體帳戶名稱。

-   在上一個步驟中取得的寄送位置名稱。

-   作業類型 (匯出)。

-   在匯出作業完成之後要傳送磁碟機的傳回位址。

-   要匯出的 blob (或 blob 前置詞) 清單。

## <a name="shipping-your-drives"></a>寄送您的磁碟機
 接下來，根據您選取要匯出的 blob 和磁碟機大小，使用 Azure 匯入/匯出工具來判斷需要傳送的磁碟機數目。 如需詳細資訊，請參閱 [Azure 匯入/匯出工具參考](storage-import-export-tool-how-to-v1.md)。

 將磁碟機封裝在單一包裹中，並將它們寄送至先前步驟中所取得的地址。 下一個步驟中請注意您的包裹追蹤號碼。

> [!NOTE]
>  您必須透過支援的貨運服務公司 (會提供您的包裹追蹤號碼) 寄送您的磁碟機。

## <a name="updating-the-export-job-with-your-package-information"></a>使用包裹資訊更新匯出作業
 在您取得追蹤號碼之後，請呼叫 [Update Job Properties](/rest/api/storageimportexport/jobs#Jobs_Update) 作業以更新貨運公司名稱和作業的追蹤號碼。 您可以選擇性指定磁碟機數目、寄件者地址及寄送日期。

## <a name="receiving-the-package"></a>接收包裹
 匯出作業處理完成之後，會將您的磁碟機退回給您，並包含您的加密資料。 您也可以藉由呼叫 [Get Job](/rest/api/storageimportexport/jobs#Jobs_Get) 作業來擷取每個磁碟機的 BitLocker 金鑰。 然後您就可以使用金鑰來解除鎖定磁碟機。 每個磁碟機上的磁碟機資訊清單檔案包含磁碟機上的檔案清單，以及每個檔案的原始 blob 位址。

[!INCLUDE [storage-import-export-delete-personal-info.md](../../../includes/storage-import-export-delete-personal-info.md)]

## <a name="next-steps"></a>後續步驟

* [使用匯入/匯出服務 REST API](storage-import-export-using-the-rest-api.md)
