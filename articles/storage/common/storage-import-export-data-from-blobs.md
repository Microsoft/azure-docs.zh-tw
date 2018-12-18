---
title: 使用 Azure 匯入/匯出來匯出 Azure Blob 中的資料 | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立匯出作業，以轉送 Azure Blob 中的資料。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 07/17/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: a85fb6ce211e6089f67cb2e2f265fbda06a25b10
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45735078"
---
# <a name="use-the-azure-importexport-service-to-export-data-from-azure-blob-storage"></a>使用 Azure 匯入/匯出服務匯出 Azure Blob 儲存體中的資料
本文提供的逐步指示會說明如何使用 Azure 匯入/匯出服務，安全地從 Azure Blob 儲存體匯出大量資料。 此服務需要您將空磁碟機寄送至 Azure 資料中心。 此服務會將您儲存體帳戶的資料匯出至磁碟機，然後將磁碟機寄回給您。

## <a name="prerequisites"></a>必要條件

在建立匯出作業來轉送 Azure Blob 儲存體中的資料之前，請仔細檢閱並完成此服務的下列必要條件清單。 您必須：

- 具有可用於匯入/匯出服務的有效 Azure 訂用帳戶。
- 具有至少一個 Azure 儲存體帳戶。 請參閱[匯入/匯出服務支援的儲存體帳戶和儲存體類型](storage-import-export-requirements.md)清單。 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](storage-quickstart-create-account.md)(英文)。
- 具有屬於[支援類型](storage-import-export-requirements.md#supported-disks)的磁碟，且數量足夠。
- 擁有 FedEx/DHL 帳戶。  
    - 帳戶必須是有效的、需要有餘額，且必須有退貨運送功能。
    - 產生匯出作業的追蹤號碼。
    - 每個作業都應該具有個別的追蹤號碼。 不支援多個作業使用相同的追蹤號碼。 
    - 如果您沒有貨運公司帳戶，請移至：
        - [建立 FedEX 帳戶](https://www.fedex.com/en-us/create-account.html) \(英文\)，或 
        - [建立 DHL 帳戶](http://www.dhl-usa.com/en/express/shipping/open_account.html) \(英文\)。

## <a name="step-1-create-an-export-job"></a>步驟 1：建立匯出作業

在 Azure 入口網站中執行下列步驟，以建立匯出作業。

1. 登入 https://portal.azure.com/。
2. 移至 [所有服務] > [儲存體] > [匯入/匯出作業]。 

    ![移至匯入/匯出作業](./media/storage-import-export-data-from-blobs/export-from-blob1.png)

3. 按一下 [建立匯入/匯出作業]。

    ![按一下 [匯入/匯出作業]](./media/storage-import-export-data-from-blobs/export-from-blob2.png)

4. 在 [基本] 中：
    
    - 選取 [從 Azure 匯出]。 
    - 輸入匯出作業的描述性名稱。 使用您選擇的名稱來追蹤作業進度。 
        - 名稱只能包含小寫英文字母、數字、連字號和底線。
        - 名稱必須以字母開頭，並且不能包含空格。 
    - 選取一個訂用帳戶。
    - 輸入或選取資源群組。

        ![基本概念](./media/storage-import-export-data-from-blobs/export-from-blob3.png) 
    
3. 在 [作業詳細資料] 中：

    - 選取要從中匯出資料的儲存體帳戶。 使用接近您所在位置的儲存體帳戶。
    - 系統會根據所選儲存體帳戶的區域，自動填入「放置」位置。 
    - 指定您要從儲存體帳戶匯出至空白磁碟機的 Blob 資料。 
    - 選擇 [全部匯出]，可匯出儲存體帳戶中的所有 Blob 資料。
    
         ![全部匯出](./media/storage-import-export-data-from-blobs/export-from-blob4.png) 

    - 您可以指定要匯出的容器和 Blob。
        - **若要指定要匯出的 Blob**：使用「**等於**」選取器。 指定 Blob 的相對路徑，開頭為容器名稱。 使用 *$root* 指定根容器。
        - **若要為所有 Blob 指定開頭的前置詞**：使用「**開始值**」選取器。 指定前置詞，以正斜線 '/' 開頭。 此首碼可以是容器名稱的首碼、完整容器名稱，或是後面接著 Blob 名稱首碼的完整容器名稱。 您必須提供有效格式的 Blob 路徑，以避免在處理期間發生錯誤，如以下螢幕擷取畫面所示。 如需詳細資訊，請參閱[有效 Blob 路徑的範例](#examples-of-valid-blob-paths)。 
   
           ![匯出選取的容器和 Blob](./media/storage-import-export-data-from-blobs/export-from-blob5.png) 

    - 您可以從 Blob 清單檔案匯出。

        ![從 Blob 清單檔案匯出](./media/storage-import-export-data-from-blobs/export-from-blob6.png)  
   
   > [!NOTE]
   > 如果要匯出的 Blob 在資料複製期間為使用中狀態，則 Azure 匯入/匯出服務會擷取 Blob 的快照集，並複製此快照集。
 

4. 在 [寄返資訊] 中：

    - 從下拉式清單中選取貨運公司。
    - 輸入您在該貨運公司中建立的有效貨運帳戶號碼。 當匯入作業完成時，Microsoft 會透過此帳戶將磁碟機寄還給您。 
    - 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。

        > [!TIP] 
        > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。
   
5. 在 [摘要] 中：

    - 檢閱作業的詳細資料。
    - 請記下作業名稱及所提供的 Azure 資料中心寄送地址，以將磁碟機寄送至 Azure。 

        > [!NOTE] 
        > 請一律將磁碟機寄送到在 Azure 入口網站中記下的資料中心。 如果磁碟機寄送到錯誤的資料中心，作業將不會被處理。

    - 按一下 [確定]，完成建立匯出作業。

## <a name="step-2-ship-the-drives"></a>步驟 2：寄送磁碟機

如果您不知道您需要多少磁碟機，請移至[確認磁碟機數目](#check-the-number-of-drives)。 如果您知道磁碟機數目，請直接寄送磁碟機。

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]

## <a name="step-3-update-the-job-with-tracking-information"></a>步驟 3：使用追蹤資訊更新作業

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]


## <a name="step-4-receive-the-disks"></a>步驟 4：收到磁碟
當儀表板報告該作業已完成時，磁碟機已寄送給您，您可以在入口網站上取得貨運的追蹤號碼。

1. 當您收到具有匯出資料的磁碟機之後，您需要取得 BitLocker 金鑰才能對磁碟機解除鎖定。 移至 Azure 入口網站中的匯出作業。 按一下 [匯入/匯出] 索引標籤。 
2. 在清單中選取並按一下您的匯出作業。 移至 [BitLocker 金鑰] 並複製金鑰。
   
   ![檢視匯出工作的 BitLocker 金鑰](./media/storage-import-export-service/export-job-bitlocker-keys.png)

3. 使用 BitLocker 金鑰來對磁碟機解除鎖定。

匯出已完成。 此時，您可以刪除作業，或是讓其在 90 天後自動刪除。


## <a name="check-the-number-of-drives"></a>確認磁碟機數目

此「選擇性」步驟可協助您判斷匯出作業所需的磁碟機數目。 執行此步驟的 Windows 系統必須執行[受支援的 OS 版本](storage-import-export-requirements.md#supported-operating-systems)。

1. 請在 Windows 系統上[下載 WAImportExport 第 1 版](https://www.microsoft.com/en-us/download/details.aspx?id=42659)。 
2. 將檔案解壓縮至預設資料夾 `waimportexportv1`。 例如： `C:\WaImportExportV1`。
3. 以系統管理權限開啟 PowerShell 或命令列視窗。 若要將目錄變更為解壓縮的資料夾，請執行下列命令：
    
    `cd C:\WaImportExportV1`

4. 若要為所選 Blob 確認所需的磁碟機數目，請執行下列命令：

    `WAImportExport.exe PreviewExport /sn:<Storage account name> /sk:<Storage account key> /ExportBlobListFile:<Path to XML blob list file> /DriveSize:<Size of drives used>`

    下表會說明這些參數：
    
    |命令列參數|說明|  
    |--------------------------|-----------------|  
    |**/logdir:**|選用。 記錄檔目錄。 詳細資訊記錄檔會寫入至這個目錄。 如未指定，將使用目前的目錄做為記錄目錄。|  
    |**/sn:**|必要。 匯出作業的儲存體帳戶名稱。|  
    |**/sk:**|如果未指定容器 SAS 時，才是必要參數。 匯出作業之儲存體帳戶的帳戶金鑰。|  
    |**/csas:**|如果未指定儲存體帳戶金鑰時，才是必要參數。 容器 SAS，可供列出要在匯出作業中匯出的 blob。|  
    |**/ExportBlobListFile:**|必要。 XML 檔案的路徑，此檔案包含要匯出的 Blob 的Blob 路徑清單或 Blob 路徑前置詞。 匯入/匯出服務 REST API 的 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業中 `BlobListBlobPath` 元素中所使用的檔案格式。|  
    |**/DriveSize:**|必要。 要用於匯出作業的磁碟機大小，例如 500 GB、1.5 TB。|  

    請參閱 [PreviewExport 命令的範例](#example-of-previewexport-command)。
 
5. 請確定您可以讀取/寫入為匯出作業而寄送的磁碟機。

### <a name="example-of-previewexport-command"></a>PreviewExport 命令的範例

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

## <a name="examples-of-valid-blob-paths"></a>有效 Blob 路徑的範例

下表顯示有效 Blob 路徑範例：
   
   | 選取器 | Blob 路徑 | 說明 |
   | --- | --- | --- |
   | 開頭為 |/ |匯出儲存體帳戶中的所有 Blob |
   | 開頭為 |/$root/ |匯出根容器中的所有 Blob |
   | 開頭為 |/book |匯出任何容器中以首碼 **book** |
   | 開頭為 |/music/ |匯出容器 **music** |
   | 開頭為 |/music/love |匯出容器 **music** 中以首碼 **love** 開頭的所有 Blob |
   | 等於 |$root/logo.bmp |匯出根容器中的 Blob **logo.bmp** |
   | 等於 |videos/story.mp4 |匯出容器 **videos** 中的 Blob **story.mp4** |

## <a name="next-steps"></a>後續步驟

* [檢視作業和磁碟機狀態](storage-import-export-view-drive-status.md)
* [檢閱匯入/匯出的需求](storage-import-export-requirements.md)


