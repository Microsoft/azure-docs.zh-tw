---
title: Azure 資料箱磁碟限制 | Microsoft Docs
description: 說明 Microsoft Azure 資料箱磁碟的系統限制和建議大小。
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: article
ms.date: 09/04/2018
ms.author: alkohli
ms.openlocfilehash: 1a4fe30881f06d8af851a67f389a6faafbe3dfef
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389457"
---
# <a name="azure-data-box-disk-limits-preview"></a>Azure 資料箱磁碟限制 (預覽)


當您部署和操作 Microsoft Azure 資料箱磁碟解決方案時，請考慮這些限制。 

> [!IMPORTANT] 
> Azure 資料箱磁碟處於預覽狀態。 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 


## <a name="data-box-service-limits"></a>資料箱服務限制

 - 在 Azure 公用雲端的所有 Azure 區域中，資料箱服務僅適用於美國、歐盟、加拿大和澳洲。
 - 資料箱磁碟支援單一儲存體帳戶。

## <a name="data-box-disk-performance"></a>資料箱磁碟效能

使用透過 USB 3.0 連線的磁碟進行測試時，磁碟效能可達 430 MB/s。 實際數字會隨使用的檔案大小而不同。 如果檔案較小，您可能會看到較低的效能。

## <a name="azure-storage-limits"></a>Azure 儲存體限制

本節說明 Azure 儲存體服務的限制，以及適用於資料箱服務的 Azure 檔案、Azure 區塊 Blob 和 Azure 分頁 Blob 所需的命名慣例。 請仔細檢閱儲存體限制，並遵循所有的建議。

如需與 Azure 儲存體服務限制以及命名共用、容器和檔案的最佳作法有關的最新資訊，請移至：

- [命名和參考容器](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)
- [命名和參考共用](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [區塊 Blob 和分頁 Blob 慣例](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

> [!IMPORTANT]
> 如果有任何檔案或目錄超出 Azure 儲存體服務限制，或不符合 Azure 檔案/Blob 命名慣例，則不會透過資料箱服務將這些檔案或目錄擷取到 Azure 儲存體中。

## <a name="data-upload-caveats"></a>資料上傳注意事項

- 請勿直接將資料複製到磁碟中。 請將資料複製到預先建立的 *BlockBlob* 和 *PageBlob* 資料夾中。
- *BlockBlob* 和 *PageBlob* 下的資料夾是容器。 例如，容器會建立為 *BlockBlob/container* 和 *PageBlob/container*。
- 如果您在雲端中現有的 Azure 物件 (例如 Blob) 與要複製的物件同名，資料箱磁碟將會覆寫雲端中的檔案。
- 寫入 *BlockBlob* 和 *PageBlob* 共用中的每個檔案，分別會以區塊 Blob 和分頁 Blob 的形式上傳。
- 在 *BlockBlob* 和 *PageBlob* 資料夾下建立的任何空目錄階層 (不含任何檔案) 則不會上傳。
- 如果將資料上傳至 Azure 時發生任何錯誤，則會在目標儲存體帳戶中建立錯誤記錄。 在上傳完成後，入口網站中會提供此錯誤記錄的路徑，而您可以檢閱記錄以執行更正動作。 若未事先確認已上傳的資料，請勿從來源刪除資料。

## <a name="azure-storage-account-size-limits"></a>Azure 儲存體帳戶大小限制

以下是複製到儲存體帳戶中的資料在大小方面的限制。 請確定您上傳的資料符合這些限制。 如需關於這些限制的最新資訊，請移至 [Azure Blob 儲存體調整目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets#azure-blob-storage-scale-targets)和 [Azure 檔案調整目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets#azure-files-scale-targets)。

| 複製到 Azure 儲存體帳戶中的資料大小                      | 預設限制          |
|---------------------------------------------------------------------|------------------------|
| 區塊 Blob 和分頁 Blob                                            | 每一儲存體帳戶 500 TB。 <br> 其中包含來自所有來源 (包括資料箱磁碟) 的資料。|


## <a name="azure-object-size-limits"></a>Azure 物件大小限制

以下是可寫入的 Azure 物件大小。 請確定所有上傳的檔案均符合這些限制。

| Azure 物件類型 | 預設限制                                             |
|-------------------|-----------------------------------------------------------|
| 區塊 Blob        | ~ 8 TB                                                 |
| 分頁 Blob         | 1 TB <br> (以分頁 Blob 格式上傳的每個檔案都必須是 512 位元組規格 (整數倍數)，否則上傳會失敗。 <br> VHD 和 VHDX 為 512 位元組規格。) |


## <a name="azure-block-blob-and-page-blob-naming-conventions"></a>Azure 區塊 Blob 和分頁 Blob 的命名慣例

| 實體                                       | 慣例                                                                                                                                                                                                                                                                                                               |
|----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 區塊 Blob 和分頁 Blob 的容器名稱 | 必須是長度介於 3 到 63 個字元的有效 DNS 名稱。 <br>  必須以字母或數字開頭。 <br> 只能包含小寫字母、數字和連字號 (-)。 <br> 每個連字號 (-) 的前後都必須緊鄰字母或數字。 <br> 名稱中不允許連續的連字號。 |
| 區塊 Blob 和分頁 Blob 的 Blob 名稱      | Blob 名稱會區分大小寫，而且可以包含字元的任意組合。 <br> Blob 名稱長度必須介於 1 到 1,024 個字元之間。 <br> 保留的 URL 字元必須正確逸出。 <br>構成 Blob 名稱的路徑區段數目不可超過 254 個。 路徑線段是連續分隔符號字元 (例如，正斜線 '/') 之間的字串，會對應到虛擬目錄的名稱。 |


## <a name="next-steps"></a>後續步驟
* 檢閱[資料箱系統需求](data-box-system-requirements.md)
