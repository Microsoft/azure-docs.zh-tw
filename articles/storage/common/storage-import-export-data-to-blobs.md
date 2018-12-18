---
title: 使用 Azure 匯入/匯出將資料轉送至 Azure Blob | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立匯入和匯出作業，將資料傳入和傳出 Blob 儲存體。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 07/17/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: 39287e4d90e4f6f42dd1f79d5457853e253bd736
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45737051"
---
# <a name="use-the-azure-importexport-service-to-import-data-to-azure-blob-storage"></a>使用 Azure 匯入/匯出服務將資料匯入 Azure Blob 儲存體

本文提供的逐步指示會說明如何使用 Azure 匯入/匯出服務，安全地將大量資料匯入 Azure Blob 儲存體。 若要將資料匯入到 Azure Blob，服務會要求您將包含資料的加密磁碟機寄送到 Azure 資料中心。  

## <a name="prerequisites"></a>必要條件

在建立匯入作業來將資料傳入 Azure Blob 儲存體之前，請仔細檢閱並完成此服務的下列必要條件清單。 您必須：

- 具有可用於匯入/匯出服務的有效 Azure 訂用帳戶。
- 至少具有一個包含儲存體容器的 Azure 儲存體帳戶。 請參閱[匯入/匯出服務支援的儲存體帳戶和儲存體類型](storage-import-export-requirements.md)清單。 
    - 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](storage-quickstart-create-account.md)(英文)。 
    - 如需有關儲存體容器的資訊，請移至[建立儲存體容器](../blobs/storage-quickstart-blobs-portal.md#create-a-container)。
- 具有屬於[支援類型](storage-import-export-requirements.md#supported-disks)的磁碟，且數量足夠。 
- 具有執行[受支援 OS 版本](storage-import-export-requirements.md#supported-operating-systems) 的 Windows 系統。 
- 在 Windows 系統上啟用 BitLocker。 請參閱[如何啟用 BitLocker](http://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/)。
- 請在 Windows 系統上[下載 WAImportExport 第 1 版](https://www.microsoft.com/en-us/download/details.aspx?id=42659)。 將檔案解壓縮至預設資料夾 `waimportexportv1`。 例如： `C:\WaImportExportV1`。
- 擁有 FedEx/DHL 帳戶。  
    - 帳戶必須是有效的、需要有餘額，且必須有退貨運送功能。
    - 產生匯出作業的追蹤號碼。
    - 每個作業都應該具有個別的追蹤號碼。 不支援多個作業使用相同的追蹤號碼。
    - 如果您沒有貨運公司帳戶，請移至：
        - [建立 FedEX 帳戶](https://www.fedex.com/en-us/create-account.html) \(英文\)，或 
        - [建立 DHL 帳戶](http://www.dhl-usa.com/en/express/shipping/open_account.html) \(英文\)。

## <a name="step-1-prepare-the-drives"></a>步驟 1：準備磁碟機

此步驟會產生日誌檔案。 日誌檔案會儲存磁碟機序號、加密金鑰及儲存體帳戶詳細資料等基本資訊。 

執行下列步驟來準備磁碟機。

1.  透過 SATA 連接器將磁碟機連線到 Windows 系統。
1.  在每個磁碟機上建立單一 NTFS 磁碟區。 指派磁碟機代號給磁碟區。 請勿使用掛接點。
2.  在 NTFS 磁碟區上啟用 BitLocker 加密。 如果使用 Windows Server 系統，請使用[如何在 Windows Server 2012 R2 上啟用 BitLocker](http://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/) 中的指示。
3.  將資料複製到加密磁碟區。 使用拖放或 Robocopy，或是任何這類的複製工具。
4.  以系統管理權限開啟 PowerShell 或命令列視窗。 若要將目錄變更為解壓縮的資料夾，請執行下列命令：
    
    `cd C:\WaImportExportV1`
5.  若要取得磁碟機的 BitLocker 金鑰，請執行下列命令：
    
    ` manage-bde -protectors -get <DriveLetter>: `
6.  若要準備磁碟，請執行下列命令。 **視資料大小而定，這可能需要數小時到數天的時間。** 

    ```
    ./WAImportExport.exe PrepImport /j:<journal file name> /id:session#<session number> /sk:<Storage account key> /t:<Drive letter> /bk:<BitLocker key> /srcdir:<Drive letter>:\ /dstdir:<Container name>/ /skipwrite 
    ```
    日誌檔案會建立在執行此工具的相同資料夾中。 還會建立其他兩個檔案 - .xml 檔案 (您執行工具的資料夾) 和 drive-manifest.xml 檔案 (資料所在的資料夾)。
    
    下表會說明使用的參數：

    |選項  |說明  |
    |---------|---------|
    |/j:     |日誌檔案的名稱 (具有 .jrn 副檔名)。 每個磁碟機都會產生日誌檔案。 我們建議您使用磁碟序號作為日誌檔案名稱。         |
    |/id:     |工作階段識別碼。 針對命令的每個執行個體使用唯一的工作階段號碼。      |
    |/sk:     |Azure 儲存體帳戶金鑰。         |
    |/t:     |要寄送之磁碟的磁碟機代號。 例如，磁碟機 `D`。         |
    |/bk:     |磁碟機的 BitLocker 金鑰。 其數字密碼來自 ` manage-bde -protectors -get D: ` 的輸出      |
    |/srcdir:     |要寄送之磁碟的磁碟機代號，其後緊接著 `:\`。 例如： `D:\`。         |
    |/dstdir:     |Azure 儲存體中目的地容器的名稱。         |
    |/skipwrite：     |此選項表示不需要複製新資料，且即將準備磁碟上的現有資料。          |
7. 為每個要寄送的磁碟重複上述步驟。 每次執行命令列時，都會使用提供的名稱來建立日誌檔案。
    
    > [!IMPORTANT]
    > - 與日誌檔案一起，`<Journal file name>_DriveInfo_<Drive serial ID>.xml` 檔案也會建立在工具所在的相同資料夾中。 如果日誌檔案太大，建立作業時會使用 .xml 檔案代替日誌檔案。 

## <a name="step-2-create-an-import-job"></a>步驟 2：建立匯入作業

在 Azure 入口網站中執行下列步驟，以建立匯入作業。

1. 登入 https://portal.azure.com/。
2. 移至 [所有服務] > [儲存體] > [匯入/匯出作業]。 
    
    ![移至匯入/匯出作業](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. 按一下 [建立匯入/匯出作業]。

    ![按一下 [建立匯入/匯出作業]](./media/storage-import-export-data-to-blobs/import-to-blob2.png)

4. 在 [基本] 中：

    - 選取 [匯入至 Azure]。
    - 輸入匯入作業的描述性名稱。 使用此名稱來追蹤作業進度。
        - 名稱只能包含小寫英文字母、數字、連字號和底線。
        - 名稱必須以字母開頭，並且不能包含空格。
    - 選取一個訂用帳戶。
    - 輸入或選取資源群組。  

    ![建立匯入作業 - 步驟 1](./media/storage-import-export-data-to-blobs/import-to-blob3.png)

3. 在 [作業詳細資料] 中：

    - 上傳在磁碟機準備步驟中取得的磁碟機日誌檔案。 如果已使用 `waimportexport.exe version1`，您需要針對已備妥的每個磁碟機上傳一個檔案。 如果日誌檔案大小超過 2 MB，則您可以使用與日誌檔案一起建立的 `<Journal file name>_DriveInfo_<Drive serial ID>.xml`。 
    - 選取將存放資料的目的地儲存體帳戶。 
    - 系統會根據所選儲存體帳戶的區域，自動填入「放置」位置。
   
   ![建立匯入作業 - 步驟 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

4. 在 [寄返資訊] 中：

    - 從下拉式清單中選取貨運公司。
    - 輸入您在該貨運公司中建立的有效貨運帳戶號碼。 當匯入作業完成時，Microsoft 會透過此帳戶將磁碟機寄還給您。 如果您沒有帳戶號碼，請建立 [FedEx](http://www.fedex.com/us/oadr/) 或 [DHL](http://www.dhl.com/) 貨運帳戶。
    - 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。 
        
        > [!TIP] 
        > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。

    ![建立匯入工作 - 步驟 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)
   
5. 在 [摘要] 中：

    - 檢閱摘要中提供的作業資訊。 記下作業名稱和 Azure 資料中心寄送地址，以將磁碟寄回 Azure。 這項資訊稍後會用在出貨標籤上。
    - 按一下 [確定] 以完成建立匯入作業。

    ![建立匯入作業 - 步驟 4](./media/storage-import-export-data-to-blobs/import-to-blob6.png)

## <a name="step-3-ship-the-drives"></a>步驟 3：寄送磁碟機 

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]


## <a name="step-4-update-the-job-with-tracking-information"></a>步驟 4：使用追蹤資訊更新作業

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>步驟 5：確認資料上傳至 Azure

追蹤作業到完成為止。 作業完成之後，請確認您的資料已上傳至 Azure。 確認上傳成功之後才刪除內部部署的資料。

## <a name="next-steps"></a>後續步驟

* [檢視作業和磁碟機狀態](storage-import-export-view-drive-status.md)
* [檢閱匯入/匯出的需求](storage-import-export-requirements.md)


