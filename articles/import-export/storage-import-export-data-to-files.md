---
title: 使用 Azure 匯入/匯出將資料轉送至 Azure 檔案服務 | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立匯入作業，以將資料轉送至 Azure 檔案服務。
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/14/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: e038cdcb50c7ee15960c904c8e234d6917d02f3b
ms.sourcegitcommit: 75041f1bce98b1d20cd93945a7b3bd875e6999d0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98706376"
---
# <a name="use-azure-importexport-service-to-import-data-to-azure-files"></a>使用 Azure 匯入/匯出服務將資料匯入 Azure 檔案服務

本文提供的逐步指示會說明如何使用 Azure 匯入/匯出服務，安全地將大量資料匯入 Azure 檔案服務。 若要將匯入資料，服務會要求您將包含資料的可支援磁碟機寄送到 Azure 資料中心。

匯入/匯出服務僅支援將 Azure 檔案服務匯入到 Azure 儲存體。 不支援將 Azure 檔案服務匯出。

## <a name="prerequisites"></a>Prerequisites

在建立匯入作業來將資料傳入 Azure 檔案服務之前，請仔細檢閱並完成下列必要條件清單。 您必須：

- 具有可與匯入/匯出服務搭配使用的有效 Azure 訂用帳戶。
- 具有至少一個 Azure 儲存體帳戶。 請參閱[匯入/匯出服務支援的儲存體帳戶和儲存體類型](storage-import-export-requirements.md)清單。 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](../storage/common/storage-account-create.md)(英文)。
- 具有屬於[支援類型](storage-import-export-requirements.md#supported-disks)的磁碟，且數量足夠。
- 具有執行[受支援 OS 版本](storage-import-export-requirements.md#supported-operating-systems) 的 Windows 系統。
- 請在 Windows 系統上[下載 WAImportExport 第 2 版](https://aka.ms/waiev2)。 將檔案解壓縮至預設資料夾 `waimportexport`。 例如： `C:\WaImportExport` 。
- 擁有 FedEx/DHL 帳戶。 如果您想要使用 FedEx/DHL 以外的電訊廠商，請聯絡 Azure 資料箱營運團隊 `adbops@microsoft.com` 。
    - 帳戶必須是有效的、需要有餘額，且必須有退貨運送功能。
    - 產生匯出作業的追蹤號碼。
    - 每個作業都應該具有個別的追蹤號碼。 不支援多個作業使用相同的追蹤號碼。
    - 如果您沒有貨運公司帳戶，請移至：
        - [建立 FedEx 帳戶](https://www.fedex.com/en-us/create-account.html)，或
        - [建立 DHL 帳戶](http://www.dhl-usa.com/en/express/shipping/open_account.html) \(英文\)。



## <a name="step-1-prepare-the-drives"></a>步驟 1：準備磁碟機

此步驟會產生日誌檔案。 日誌檔案會儲存磁碟機序號、加密金鑰及儲存體帳戶詳細資料等基本資訊。

執行下列步驟來準備磁碟機。

1. 透過 SATA 連接器將我們的磁碟機連線到 Windows 系統。
2. 在每個磁碟機上建立單一 NTFS 磁碟區。 指派磁碟機代號給磁碟區。 請勿使用掛接點。
3. 在工具所在的根資料夾中，修改 dataset.csv 檔案。 根據您要匯入的是檔案、資料夾或兩者，將項目加入 dataset.csv 檔案，類似下列範例。

   - **若要匯入** 檔案：在下列範例中，要複製的資料位於 F：磁片磁碟機中。 您的檔案 MyFile1.txt 會複製到 MyAzureFileshare1 的根目錄。 如果 MyAzureFileshare1 不存在，則會建立在 Azure 儲存體帳戶中。 資料夾結構會保留。

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
     深入了解[準備資料集 CSV 檔案](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import)。


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

     您可以在對應至多個磁碟機的相同檔案中製造多個項目。 深入了解[準備磁碟機集 CSV 檔案](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import)。

5. 使用 `PrepImport` 選項可將資料複製到磁碟機並準備這些資料。 針對以新複製工作階段來複製目錄和/或檔案的第一複製工作階段，請執行下列命令：

    ```cmd
    .\WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>]/DataSet:<dataset.csv>
    ```

   匯入範例如下所示。

    ```cmd
    .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
    ```

6. 每次執行命令列時，都會使用您以 `/j:` 參數提供的名稱來建立日誌檔案。 您準備的每個磁碟機有日誌檔案，您必須在建立匯入作業時將其上傳。 沒有日誌檔案的磁碟機將無法處理。

    > [!IMPORTANT]
    > - 完成磁碟準備工作之後，請勿修改磁碟機上的資料或日誌檔案。

如需其他範例，請移至[日誌檔案的範例](#samples-for-journal-files)。

## <a name="step-2-create-an-import-job"></a>步驟 2：建立匯入作業

### <a name="portal"></a>[入口網站](#tab/azure-portal)

在 Azure 入口網站中執行下列步驟，以建立匯入作業。
1. 登入 https://portal.azure.com/。
2. 移至 [所有服務] > [儲存體] > [匯入/匯出作業]。

    ![移至匯入/匯出](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. 按一下 [ **建立匯入/匯出作業**]。

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
    - 系統會根據所選儲存體帳戶的區域，自動填入置放位置。

       ![建立匯入作業 - 步驟 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

4. 在 [寄返資訊] 中：

    - 從下拉式清單中選取貨運公司。 如果您想要使用 FedEx/DHL 以外的電訊廠商，請從下拉式清單中選擇現有的選項。 請與 `adbops@microsoft.com`  您打算使用的電訊廠商相關資訊，聯絡 Azure 資料箱營運團隊。
    - 輸入您在該貨運公司中建立的有效貨運帳戶號碼。 當匯入作業完成時，Microsoft 會透過此帳戶將磁碟機寄還給您。
    - 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。

        > [!TIP]
        > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。

       ![建立匯入工作 - 步驟 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)


5. 在 [摘要] 中：

    - 提供 Azure 資料中心寄送地址，以便將磁碟機寄回 Azure。 請確認出貨標籤上有提及作業名稱和完整的地址。
    - 按一下 [確定]，完成建立匯入作業。

        ![建立匯入作業 - 步驟 4](./media/storage-import-export-data-to-blobs/import-to-blob6.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

使用下列步驟，在 Azure CLI 中建立匯入作業。

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>建立作業

1. 使用 [az extension add](/cli/azure/extension#az_extension_add) 命令新增 [az 匯入匯出](/cli/azure/ext/import-export/import-export) 延伸模組：

    ```azurecli
    az extension add --name import-export
    ```

1. 您可以使用現有的資源群組，或建立一個群組。 執行 [az group create](/cli/azure/group#az_group_create) 命令以建立資源群組：

    ```azurecli
    az group create --name myierg --location "West US"
    ```

1. 您可以使用現有的儲存體帳戶或建立一個帳戶。 若要建立儲存體帳戶，請執行 [az storage account create](/cli/azure/storage/account#az_storage_account_create) 命令：

    ```azurecli
    az storage account create -resource-group myierg -name myssdocsstorage --https-only
    ```

1. 若要取得您可以寄送磁片的位置清單，請使用 az 匯 [入-匯出位置清單](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_list) 命令：

    ```azurecli
    az import-export location list
    ```

1. 使用 [az 匯入-匯出位置 show](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_show) 命令來取得您區域的位置：

    ```azurecli
    az import-export location show --location "West US"
    ```

1. 執行下列 [az import-export create](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_create) 命令來建立匯入作業：

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name MyIEjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --drive-list bit-locker-key=439675-460165-128202-905124-487224-524332-851649-442187 \
            drive-header-hash= drive-id=AZ31BGB1 manifest-file=\\DriveManifest.xml \
            manifest-hash=69512026C1E8D4401816A2E5B8D7420D \
        --type Import \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --return-shipping carrier-name=FedEx carrier-account-number=123456789 \
        --storage-account myssdocsstorage
    ```

   > [!TIP]
   > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。


1. 使用 [az 匯入-匯出清單](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_list) 命令來查看 myierg 資源群組的所有工作：

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. 若要更新您的作業或取消作業，請執行 [az 匯入-匯出更新](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_update) 命令：

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

使用下列步驟，在 Azure PowerShell 中建立匯入作業。

[!INCLUDE [azure-powershell-requirements-h3.md](../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> **Microsoft.importexport** PowerShell 模組目前為預覽狀態，您必須使用指令程式個別進行安裝 `Install-Module` 。 此 PowerShell 模組正式推出後，便會成為未來 Az PowerShell 模組版本的一部分，且預設可從 Azure Cloud Shell 內使用。

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>建立作業

1. 您可以使用現有的資源群組，或建立一個群組。 若要建立資源群組，請執行 [>new-azresourcegroup](/powershell/module/az.resources/new-azresourcegroup) Cmdlet：

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myierg -Location westus
   ```

1. 您可以使用現有的儲存體帳戶或建立一個帳戶。 若要建立儲存體帳戶，請執行 [new-azstorageaccount](/powershell/module/az.storage/new-azstorageaccount) Cmdlet：

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName myierg -AccountName myssdocsstorage -SkuName Standard_RAGRS -Location westus -EnableHttpsTrafficOnly $true
   ```

1. 若要取得您可以寄送磁片的位置清單，請使用 [AzImportExportLocation](/powershell/module/az.importexport/get-azimportexportlocation) Cmdlet：

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. 使用 `Get-AzImportExportLocation` Cmdlet 搭配 `Name` 參數，以取得您區域的位置：

   ```azurepowershell-interactive
   Get-AzImportExportLocation -Name westus
   ```

1. 執行下列 [AzImportExport](/powershell/module/az.importexport/new-azimportexport) 範例來建立匯入作業：

   ```azurepowershell-interactive
   $driveList = @(@{
     DriveId = '9CA995BA'
     BitLockerKey = '439675-460165-128202-905124-487224-524332-851649-442187'
     ManifestFile = '\\DriveManifest.xml'
     ManifestHash = '69512026C1E8D4401816A2E5B8D7420D'
     DriveHeaderHash = 'AZ31BGB1'
   })

   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'MyIEjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      DriveList = $driveList
      JobType = 'Import'
      LogLevel = 'Verbose'
      ShippingInformationRecipientName = 'Microsoft Azure Import/Export Service'
      ShippingInformationStreetAddress1 = '3020 Coronado'
      ShippingInformationCity = 'Santa Clara'
      ShippingInformationStateOrProvince = 'CA'
      ShippingInformationPostalCode = '98054'
      ShippingInformationCountryOrRegion = 'USA'
      ShippingInformationPhone = '4083527600'
      ReturnAddressRecipientName = 'Gus Poland'
      ReturnAddressStreetAddress1 = '1020 Enterprise way'
      ReturnAddressCity = 'Sunnyvale'
      ReturnAddressStateOrProvince = 'CA'
      ReturnAddressPostalCode = '94089'
      ReturnAddressCountryOrRegion = 'USA'
      ReturnAddressPhone = '4085555555'
      ReturnAddressEmail = 'gus@contoso.com'
      ReturnShippingCarrierName = 'FedEx'
      ReturnShippingCarrierAccountNumber = '123456789'
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

   > [!TIP]
   > 請提供群組電子郵件，而不是指定單一使用者的電子郵件地址。 這樣可以確保即使當系統管理員不在時，您也可以收到通知。

1. 使用 [AzImportExport 指令程式](/powershell/module/az.importexport/get-azimportexport) 可查看 myierg 資源群組的所有工作：

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. 若要更新您的作業或取消作業，請執行 [AzImportExport](/powershell/module/az.importexport/update-azimportexport) Cmdlet：

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

## <a name="step-3-ship-the-drives-to-the-azure-datacenter"></a>步驟 3：將磁碟機寄送至 Azure 資料中心

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-4-update-the-job-with-tracking-information"></a>步驟 4：使用追蹤資訊更新作業

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>步驟 5：確認資料上傳至 Azure

追蹤作業到完成為止。 作業完成之後，請確認您的資料已上傳至 Azure。 確認上傳成功之後才刪除內部部署的資料。

## <a name="samples-for-journal-files"></a>日誌檔案的範例

若要 **新增更多磁碟機**，可以建立新的磁碟機集檔案並執行命令，如下所示。

對於後續複製到不同磁碟機的工作階段 (不是 InitialDriveset.csv 檔案中指定的磁碟機)，請指定新的磁碟機集 .csv 檔案，並以值形式提供給參數 `AdditionalDriveSet`。 使用 **相同的日誌檔** 名稱，並提供 **新的工作階段識別碼**。 AdditionalDriveset CSV 檔案的格式與 InitialDriveSet 格式相同 。

```cmd
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
```

匯入範例如下所示。

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
```


若要將其他資料新增至相同磁碟機集，請對後續複製工作階段使用 PrepImport 命令，以複製其他檔案/目錄。

若要對 InitialDriveset.csv 檔案中指定的同個硬碟進行後續複製工作階段，請指定 **相同的日誌檔案** 名稱並提供 **新的工作階段識別碼**；不需要再次提供儲存體帳戶金鑰。

```cmd
WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
```

匯入範例如下所示。

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

## <a name="next-steps"></a>後續步驟

* [檢視作業和磁碟機狀態](storage-import-export-view-drive-status.md)
* [檢閱匯入/匯出的需求](storage-import-export-requirements.md)