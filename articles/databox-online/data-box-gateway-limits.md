---
title: Azure 資料箱閘道限制 | Microsoft Docs
description: 說明 Microsoft Azure 資料箱閘道的系統限制和建議大小。
services: databox-edge-gateway
documentationcenter: NA
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: databox-edge-gateway
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/24/2018
ms.author: alkohli
ms.custom: ''
ms.openlocfilehash: edb4995b626055be830a7accb74d99f1db3ef8d0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962174"
---
# <a name="azure-data-box-gateway-limits-preview"></a>Azure 資料箱閘道限制 (預覽)


當您部署和操作 Microsoft Azure 資料箱閘道解決方案時，請考慮這些限制。 

> [!IMPORTANT] 
> 資料箱閘道處於預覽狀態。 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 


## <a name="data-box-gateway-service-limits"></a>資料箱閘道服務限制

- 在此版本中，服務僅適用於美國、歐洲和亞太地區的特定區域。 如需詳細資訊，請移至[區域可用性](#data-box-gateway-overview#region-availability)。 儲存體帳戶應在實際距離上盡可能接近裝置部署所在的區域 (可與服務地區不同)。
- 目前不支援將資料箱閘道資源移至不同的訂用帳戶或資源群組。 如需詳細資訊，請移至[將資源移至新的資源群組或訂用帳戶](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)。

## <a name="data-box-gateway-device-limits"></a>資料箱閘道裝置限制

下表說明資料箱閘道裝置的限制。

| 說明 | 值 |
|---|---|
|否。 (每個裝置的檔案) |1 億 <br> 限制約為每 2 TB 的磁碟空間可以有 2 千 5 百萬個檔案，最大限制為 1 億個 |
|否。 (每個裝置的共用) |24 |
|寫入至共用的檔案大小上限|5 TB |

## <a name="azure-storage-limits"></a>Azure 儲存體限制

本節說明 Azure 儲存體服務的限制，以及適用於資料箱閘道/Data Box Edge 服務的 Azure 檔案、Azure 區塊 Blob 和 Azure 分頁 Blob 所需的命名慣例。 請仔細檢閱儲存體限制，並遵循所有的建議。

如需與 Azure 儲存體服務限制以及命名共用、容器和檔案的最佳作法有關的最新資訊，請移至：

- [命名和參考容器](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)
- [命名和參考共用](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [區塊 Blob 和分頁 Blob 慣例](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

> [!IMPORTANT]
> 如果有任何檔案或目錄超出 Azure 儲存體服務限制，或不符合 Azure 檔案/Blob 命名慣例，則不會透過資料箱閘道/Data Box Edge 服務將這些檔案或目錄擷取到 Azure 儲存體中。

## <a name="data-upload-caveats"></a>資料上傳注意事項

下列注意事項適用於正移至 Azure 的資料。

- 我們建議不要將多個裝置寫入至相同的容器。
- 如果雲端中現有的 Azure 物件 (例如 Blob 或檔案) 與要複製的物件同名，裝置將會覆寫雲端中的檔案。 
- 在共用資料夾下建立的空目錄階層 (不含任何檔案) 則不會上傳至 Blob 容器。


## <a name="azure-storage-account-size-and-object-size-limits"></a>Azure 儲存體帳戶大小和物件大小限制

以下是複製到儲存體帳戶中的資料在大小方面的限制。 請確定您上傳的資料符合這些限制。 如需關於這些限制的最新資訊，請移至 [Azure Blob 儲存體調整目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets#azure-blob-storage-scale-targets)和 [Azure 檔案調整目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets#azure-files-scale-targets)。

| 複製到 Azure 儲存體帳戶中的資料大小                      | 預設限制          |
|---------------------------------------------------------------------|------------------------|
| 區塊 Blob 和分頁 Blob                                            | 每一儲存體帳戶 500 TB|


## <a name="azure-object-size-limits"></a>Azure 物件大小限制

以下是可寫入的 Azure 物件大小。 請確定所有上傳的檔案均符合這些限制。

| Azure 物件類型 | 預設限制                                             |
|-------------------|-----------------------------------------------------------|
| 區塊 Blob        | ~ 8 TB                                                 |
| 分頁 Blob         | 1 TB <br> 以分頁 Blob 格式上傳的每個檔案都必須是 512 位元組規格 (整數倍數)，否則上傳會失敗。 <br> VHD 和 VHDX 為 512 位元組規格。 |


## <a name="next-steps"></a>後續步驟

- [準備部署 Azure 資料箱閘道](data-box-gateway-deploy-prep.md)
