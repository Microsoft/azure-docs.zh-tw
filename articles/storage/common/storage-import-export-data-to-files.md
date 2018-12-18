---
title: 使用 Azure 匯入/匯出將資料轉送至 Azure 檔案服務 | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立匯入作業，以將資料轉送至 Azure 檔案服務。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 09/10/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: 1be7dd9ce537d0999ace01eafa46318e42df9f20
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45732442"
---
# <a name="use-azure-importexport-service-to-import-data-to-azure-files"></a>使用 Azure 匯入/匯出服務將資料匯入 Azure 檔案服務

本文提供的逐步指示會說明如何使用 Azure 匯入/匯出服務，安全地將大量資料匯入 Azure 檔案服務。 若要將匯入資料，服務會要求您將包含資料的可支援磁碟機寄送到 Azure 資料中心。  

匯入/匯出服務僅支援將 Azure 檔案服務匯入到 Azure 儲存體。 不支援將 Azure 檔案服務匯出。

## <a name="prerequisites"></a>必要條件

在建立匯入作業來將資料傳入 Azure 檔案服務之前，請仔細檢閱並完成下列必要條件清單。 您必須：

- 具有可與匯入/匯出服務搭配使用的有效 Azure 訂用帳戶。
- 具有至少一個 Azure 儲存體帳戶。 請參閱[匯入/匯出服務支援的儲存體帳戶和儲存體類型](storage-import-export-requirements.md)清單。 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](storage-quickstart-create-account.md)(英文)。
- 具有屬於[支援類型](storage-import-export-requirements.md#supported-disks)的磁碟，且數量足夠。 
- 具有執行[受支援 OS 版本](storage-import-export-requirements.md#supported-operating-systems) 的 Windows 系統。
- 請在 Windows 系統上[下載 WAImportExport 第 2 版](https://www.microsoft.com/download/details.aspx?id=55280)。 將檔案解壓縮至預設資料夾 `waimportexport`。 例如： `C:\WaImportExport`。
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

1. 透過 SATA 連接器將我們的磁碟機連線到 Windows 系統。
2. 在每個磁碟機上建立單一 NTFS 磁碟區。 指派磁碟機代號給磁碟區。 請勿使用掛接點。
3. 在工具所在的根資料夾中，修改 dataset.csv 檔案。 根據您要匯入的是檔案、資料夾或兩者，將項目加入 dataset.csv 檔案，類似下列範例。  

    - **若要匯入檔案**：在下列範例中，要複製的資料會位於 C: 磁碟機。 您的檔案 MyFile1.txt 會複製到 MyAzureFileshare1 的根目錄。 如果 MyAzureFileshare1 不存在，則會建立在 Azure 儲存體帳戶中。 資料夾結構會保留。

        ```
            BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
            "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None
    
        ```
    - **若要匯入資料夾**：MyFolder2 底下的所有檔案和資料夾會以遞迴方式複製到檔案共用。 資料夾結構會保留。

        ```
            "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None 
            
        ```
    您可以在對應至所匯入資料夾或檔案的相同檔案中製造多個項目。 

        ```
            "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None
            "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None 
                        
        ```
    深入了解[準備資料集 CSV 檔案](storage-import-export-tool-preparing-hard-drives-import.md#prepare-the-dataset-csv-file)。
    

4. 在工具所在的根資料夾中，修改 driveset.csv 檔案。 將項目新增至 driveset.csv 檔案，類似下列範例。 磁碟機集檔案有磁碟機清單和對應的磁碟機代號，因此工具可以正確地挑選要準備的磁碟清單。

    此範例假設已連結兩個磁碟，並已建立基本 NTFS 磁碟區 G:\ 和 H:\。 當 G: 已加密時，H:\ 就不會加密。 此工具只會格式化和加密裝載 H:\ (不是 G:\)) 的磁碟。

    - **針對未加密的磁碟**：指定「Encrypt」，以在磁碟上啟用 BitLocker 加密。

        ```
        DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
        H,Format,SilentMode,Encrypt,
        ```
    
    - **針對已加密的磁碟**：指定「AlreadyEncrypted」，並提供 BitLocker 金鑰。

        ```
        DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
        G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631
        ```

    您可以在對應至多個磁碟機的相同檔案中製造多個項目。 深入了解[準備磁碟機集 CSV 檔案](storage-import-export-tool-preparing-hard-drives-import.md#prepare-initialdriveset-or-additionaldriveset-csv-file)。 

5.  使用 `PrepImport` 選項可將資料複製到磁碟機並準備這些資料。 針對以新複製工作階段來複製目錄和/或檔案的第一複製工作階段，請執行下列命令：

        ```
        .\WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>
        ```

    匯入範例如下所示。
  
        ```
        .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
        ```
 
6. 每次執行命令列時，都會使用您以 `/j:` 參數提供的名稱來建立日誌檔案。 您準備的每個磁碟機有日誌檔案，您必須在建立匯入作業時將其上傳。 沒有日誌檔案的磁碟機將無法處理。

    > [!IMPORTANT]
    > - 完成磁碟準備工作之後，請勿修改磁碟機上的資料或日誌檔案。

如需其他範例，請移至[日誌檔案的範例](#samples-for-journal-files)。

## <a name="step-2-create-an-import-job"></a>步驟 2：建立匯入作業 

在 Azure 入口網站中執行下列步驟，以建立匯入作業。
1. 登入 https://portal.azure.com/。
2. 移至 [所有服務] > [儲存體] > [匯入/匯出作業]。 

    ![移至匯入/匯出](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. 按一下 [建立匯入/匯出作業]。

    ![按一下 [匯入/匯出作業]](./media/storage-import-export-data-to-blobs/import-to-blob2.png)

4. 在 [基本] 中：

    - 選取 [匯入至 Azure]。
    - 輸入匯入作業的描述性名稱。 當作業正在進行中和一旦完成後，使用此名稱來追蹤您的作業。
        -  此名稱只能包含小寫英文字母、數字、連字號和底線。
        -  名稱必須以字母開頭，並且不能包含空格。 
    - 選取一個訂用帳戶。
    - 選取資源群組。 

        ![建立匯入作業 - 步驟 1](./media/storage-import-export-data-to-blobs/import-to-blob3.png)

3. 在 [作業詳細資料] 中：
    
    - 上傳您在前面[步驟 1：準備磁碟機](#step-1-prepare-the-drives)中建立的日誌檔。 
    - 選取將匯入資料的儲存體帳戶。 
    - 系統會根據所選儲存體帳戶的區域，自動填入「放置」位置。
   
       ![建立匯入作業 - 步驟 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

4. 在 [寄返資訊] 中：

    - 從下拉式清單中選取貨運公司。
    - 輸入您在該貨運公司中建立的有效貨運帳戶號碼。 當匯入作業完成時，Microsoft 會透過此帳戶將磁碟機寄還給您。 
    - 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。

        > [!TIP] 
        > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。

       ![建立匯入工作 - 步驟 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)

   
5. 在 [摘要] 中：

    - 提供 Azure 資料中心寄送地址，以便將磁碟機寄回 Azure。 請確認出貨標籤上有提及作業名稱和完整的地址。
    - 按一下 [確定]，完成建立匯入作業。

        ![建立匯入作業 - 步驟 4](./media/storage-import-export-data-to-blobs/import-to-blob6.png)

## <a name="step-3-ship-the-drives-to-the-azure-datacenter"></a>步驟 3：將磁碟機寄送至 Azure 資料中心 

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]

## <a name="step-4-update-the-job-with-tracking-information"></a>步驟 4：使用追蹤資訊更新作業

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>步驟 5：確認資料上傳至 Azure

追蹤作業到完成為止。 作業完成之後，請確認您的資料已上傳至 Azure。 確認上傳成功之後才刪除內部部署的資料。

## <a name="samples-for-journal-files"></a>日誌檔案的範例

若要**新增更多磁碟機**，可以建立新的磁碟機集檔案並執行命令，如下所示。 

對於後續複製到不同磁碟機的工作階段 (不是 InitialDriveset.csv 檔案中指定的磁碟機)，請指定新的磁碟機集 .csv 檔案，並以值形式提供給參數 `AdditionalDriveSet`。 使用**相同的日誌檔**名稱，並提供**新的工作階段識別碼**。 AdditionalDriveset CSV 檔案的格式與 InitialDriveSet 格式相同 。

    ```
    WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
    ```

匯入範例如下所示。

    ```
    WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
    ```


若要將其他資料新增至相同磁碟機集，請對後續複製工作階段使用 PrepImport 命令，以複製其他檔案/目錄。

若要對 InitialDriveset.csv 檔案中指定的同個硬碟進行後續複製工作階段，請指定**相同的日誌檔案**名稱並提供**新的工作階段識別碼**；不需要再次提供儲存體帳戶金鑰。

    ```
    WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
    ```

匯入範例如下所示。

    ```
    WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
    ```

## <a name="next-steps"></a>後續步驟

* [檢視作業和磁碟機狀態](storage-import-export-view-drive-status.md)
* [檢閱匯入/匯出的需求](storage-import-export-requirements.md)


