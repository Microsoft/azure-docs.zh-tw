---
title: 從 Azure 下載市集項目 | Microsoft Docs
description: 雲端操作者可以將市集項目從 Azure 下載到我的 Azure Stack 部署。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/09/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: cf6bc980f6fd821056a987d0c830863bd15ba779
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902003"
---
# <a name="download-marketplace-items-from-azure-to-azure-stack"></a>將市集項目從 Azure 下載到 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

雲端操作者可從 Azure Marketplace 下載項目，並可在 Azure Stack 中取得。 您可以選擇的項目來自 Azure Marketplace 項目策劃清單，這些項目已預先測試並支援搭配 Azure Stack 運作。 其他項目會不斷新增到此清單中，因此請繼續回來查看是否有新內容。 

有兩個用於連線到 Azure Marketplace 的案例： 

- **已連線的案例** - Azure Stack 環境必須連線到網際網路。 您可使用 Azure Stack 入口網站來找出並下載項目。 
- **已中斷連線或部分連線的案例** - 您必須存取網際網路，才能使用工具摘要整合工具下載市集項目。 然後，將下載項目傳輸到已中斷連線的 Azure Stack 安裝。 此案例使用 PowerShell。

請參閱[適用於 Azure Stack 的 Azure Marketplace 項目](azure-stack-marketplace-azure-items.md)，以取得您可以下載的市集項目清單。


## <a name="connected-scenario"></a>已連線的案例
如果 Azure Stack 連線到網際網路，您可以使用管理入口網站來下載市集項目。

### <a name="prerequisites"></a>必要條件
您的 Azure Stack 部署必須具有網際網路連線能力，而且[已向 Azure 註冊](azure-stack-register.md)。

### <a name="use-the-portal-to-download-marketplace-items"></a>使用入口網站下載市集項目  
1. 登入 Azure Stack 系統管理員入口網站。

2.  在下載市集項目之前，檢閱可用的儲存空間。 稍後，當您選取要下載的項目時，您可以比較下載大小與您可用的儲存容量。 如果容量有限，請考慮[管理可用空間](azure-stack-manage-storage-shares.md#manage-available-space)選項。 

    若要檢閱可用空間，請在 [區域管理] 中選取您想要探索的區域，然後移至 [資源提供者] > [儲存體]。

    ![檢閱儲存空間](media/azure-stack-download-azure-marketplace-item/storage.png)  

    
3. 開啟 Azure Stack Marketplace 並連線至 Azure。 若要這樣做，請選取 [Marketplace 管理]，然後選取 [從 Azure 新增]。

    ![從 Azure 新增](media/azure-stack-download-azure-marketplace-item/marketplace.png)

    入口網站會顯示可從 Azure Marketplace 下載的項目清單。 您可以按一下每個項目來檢視其描述及其他相關資訊，包括其下載大小。 

    ![Marketplace 清單](media/azure-stack-download-azure-marketplace-item/image03.png)

4. 選取您想要的項目，然後選取 [下載]。 下載時間會有所不同。

    ![下載訊息](media/azure-stack-download-azure-marketplace-item/image04.png)

    下載完成之後，您可以用 Azure Stack 操作員或使用者身分來部署新市集項目。

5. 若要部署已下載的項目，請選取 [+ 建立資源]，然後在類別中搜尋新的市集項目。 接下來選取項目以開始部署程序。 此程序會因不同的市集項目而有所不同。 

## <a name="disconnected-or-a-partially-connected-scenario"></a>已中斷連線或部分連線的案例

如果 Azure Stack 處於已中斷連線的模式而且沒有網際網路連線能力，您可使用 PowerShell 和「市集摘要整合工具」，將市集項目下載至具有網際網路連線能力的電腦。 然後將這些項目傳輸至您的 Azure Stack 環境。 在已中斷連線的環境中，您無法透過 Azure Stack 入口網站下載市集項目。 

市集摘要整合工具也可使用於已連線的案例中。 

此案例分成兩部分：
- **第 1 部分：** 從 Azure Marketplace 下載。 在可存取網際網路的電腦上，設定 PowerShell、下載摘要整合工具，然後從 Azure Marketplace 下載項目。  
- **第 2 部分：** 上傳並發佈至 Azure Stack Marketplace。 將您下載的檔案移動到 Azure Stack 環境、匯入至 Azure Stack，再將它們發佈至 Azure Stack Marketplace。  


### <a name="prerequisites"></a>必要條件
- 您的 Azure Stack 部署必須[已向 Azure 註冊](azure-stack-register.md)。  

- 具有網際網路連線能力的電腦必須擁有 **Azure Stack PowerShell 模組 1.2.11 版**或更高版本。 如果不存在的話，請[安裝 Azure Stack 特定 PowerShell 模組](azure-stack-powershell-install.md)。  

- 若要能夠匯入已下載的市集項目，則必須[為 Azure Stack 操作者設定 PowerShell 環境](azure-stack-powershell-configure-admin.md)。  

- 您必須在 Azure Stack 中擁有具備可公開存取容器 (這是儲存體 blob) 的[儲存體帳戶](azure-stack-manage-storage-accounts.md)。 您可使用容器作為市集項目資源庫檔案的暫存儲存體。 如果您不熟悉儲存體帳戶和容器，請參閱在 Azure 文件中[使用 blob - Azure 入口網站](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)。

- 市集摘要整合工具會在第一個程序期間下載。 

### <a name="use-the-marketplace-syndication-tool-to-download-marketplace-items"></a>使用市集摘要整合工具來下載市集項目

1. 在具有網際網路連線的電腦上，以系統管理員身分開啟 PowerShell 主控台。

2. 新增您用來註冊 Azure Stack 的 Azure 帳戶。 若要新增帳戶，請在 PowerShell 中執行 `Add-AzureRmAccount` (不使用任何參數)。 當系統提示您輸入 Azure 帳戶認證時，您可能需要根據帳戶的組態使用雙因素驗證。

3. 如果您有多個訂用帳戶，請執行下列命令以選取您用於註冊的訂用帳戶：  

   ```PowerShell  
   Get-AzureRmSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzureRmSubscription
   $AzureContext = Get-AzureRmContext
   ```

4. 使用下列指令碼來下載最新版的市集摘要整合工具：  

   ```PowerShell
   # Download the tools archive.
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
   invoke-webrequest https://github.com/Azure/AzureStack-Tools/archive/master.zip `
     -OutFile master.zip

   # Expand the downloaded files.
   expand-archive master.zip `
     -DestinationPath . `
     -Force

   # Change to the tools directory.
   cd .\AzureStack-Tools-master

   ```

5. 匯入摘要整合模組，然後執行下列命令來啟動工具。 使用一個位置取代 `Destination folder path`，以儲存您從 Azure Marketplace 下載的檔案。   

   ```PowerShell  
   Import-Module .\Syndication\AzureStack.MarketplaceSyndication.psm1

   Sync-AzSOfflineMarketplaceItem 
      -Destination "Destination folder path in quotes" `
      -AzureTenantID $AzureContext.Tenant.TenantId `
      -AzureSubscriptionId $AzureContext.Subscription.Id 
   ```

6. 執行工具時，系統會提示您輸入 Azure 帳號認證。 登入您用來註冊 Azure Stack 的 Azure 帳戶。 登入成功之後，您應會看到如下圖所示的畫面，其中包含可用的市集項目清單。  

   ![Azure Marketplace 項目快顯視窗](media/azure-stack-download-azure-marketplace-item/image05.png)

7. 選取您想要下載的項目，並記下 version。 (您可以按住 Ctrl 鍵以選取多個映像。)您會在下一個程序中匯入項目時參考「版本」。 
   
   您也可以使用 [新增條件] 選項來篩選映像清單。

8. 選取 [確定]，然後檢閱並接受法律條款。 

9. 下載所需的時間取決於項目的大小。 下載完成之後，在您於指令碼中指定的資料夾中可取得項目。 下載包含 VHD 檔案 (用於虛擬機器) 或 .ZIP 檔案 (用於虛擬機器擴充功能)。 它也包含 .azpkg 格式的資源庫套件。 (.azpkg 套件是 .zip 檔案。)
 

### <a name="import-the-download-and-publish-to-azure-stack-marketplace"></a>匯入下載並發佈至 Azure Stack Marketplace
1. 您[先前下載的](#use-the-marketplace-syndication-tool-to-download-marketplace-items)虛擬機器映像檔案或解決方案範本檔案，必須可在本機提供給您的 Azure Stack 環境使用。  

2. 使用管理入口網站，將市集項目套件 (.azpkg 檔案) 和虛擬硬碟映像 (.vhd 檔案) 上傳至 Azure Stack Blob 儲存體。 上傳套件和磁碟檔案使其可供 Azure Stack 使用，以便您稍後將項目發佈至 Azure Stack Marketplace。

   上傳時，您需要有具備可公開存取容器的儲存體帳戶 (請參閱這個案例的先決條件)。  
   1. 在 Azure Stack 管理員入口網站中，移至 [所有服務]，然後在 [資料 + 儲存體] 類別下，選取 [儲存體帳戶]。  
   
   2. 從您的訂用帳戶選取儲存體帳戶，然後在 [Blob 服務] 之下選取 [容器]。  
      ![Blob 服務](media/azure-stack-download-azure-marketplace-item/blob-service.png)  
   
   3. 選取您想要使用的容器，然後選取 [上傳] 以開啟 [上傳 blob] 窗格。  
      ![容器](media/azure-stack-download-azure-marketplace-item/container.png)  
   
   4. 在 [上傳 Blob] 窗格上，瀏覽至要載入到儲存體的套件和磁碟檔案，然後選取 [上傳]。  
      ![upload](media/azure-stack-download-azure-marketplace-item/upload.png)  

   5. 您上傳的檔案會出現在 [容器] 窗格中。 選取檔案並從 [Blob 屬性] 窗格複製 URL。 當您將市集項目匯入 Azure Stack 時，您將在下一個步驟中使用此 URL。  在下圖中，容器是 blob-test-storage，而檔案是 Microsoft.WindowsServer2016DatacenterServerCore-ARM.1.0.801.azpkg。  檔案 URL 是 *https://testblobstorage1.blob.local.azurestack.external/blob-test-storage/Microsoft.WindowsServer2016DatacenterServerCore-ARM.1.0.801.azpkg*。  
      ![Blob 屬性](media/azure-stack-download-azure-marketplace-item/blob-storage.png)  

3. 使用 **Add-AzsPlatformimage** Cmdlet 將 VHD 映像匯入至 Azure Stack。 使用此 Cmdlet 時，請以您所匯入映像的值，取代 publisher、offer 及其他參數值。 

   您可以從與 AZPKG 檔案一起下載的文字檔案中，取得映像的 publisher、offer 和 sku 值。 文字檔案會儲存在目的地位置。 version 值是在上一個程序中從 Azure 下載項目時所記下的版本。 
 
   下列範例指令碼中會使用 Windows Server 2016 Datacenter - Server Core 虛擬機器的值。 -Osuri 的值是項目的 Blob 儲存體位置路徑範例。

   ```PowerShell  
   Add-AzsPlatformimage `
    -publisher "MicrosoftWindowsServer" `
    -offer "WindowsServer" `
    -sku "2016-Datacenter-Server-Core" `
    -osType Windows `
    -Version "2016.127.20171215" `
    -OsUri "https://mystorageaccount.blob.local.azurestack.external/cont1/Microsoft.WindowsServer2016DatacenterServerCore-ARM.1.0.801.vhd"  
   ```
   **關於解決方案範本：** 某些範本可以包含名稱為 **fixed3.vhd** 的小型 3 MB .VHD 檔案。 您不需要將該檔案匯入到 Azure Stack。 Fixed3.vhd。  這個檔案隨附於一些解決方案範本，以符合 Azure Marketplace 的發佈需求。

   檢閱範本描述並下載，然後匯入其他需求，例如使用解決方案範本所需的 VHD。  
   
   **關於擴充功能：** 當您使用虛擬機器映像擴充功能時，請使用下列參數：
   - *發行者*
   - *類型*
   - *版本*  

   您不會將 Offer 使用於擴充功能。   


4.  使用 **Add-AzsGalleryItem** Cmdlet，透過 PowerShell 將市集項目發佈至 Azure Stack。 例如︰  
    ```PowerShell  
    Add-AzsGalleryItem `
     -GalleryItemUri "https://mystorageaccount.blob.local.azurestack.external/cont1/Microsoft.WindowsServer2016DatacenterServerCore-ARM.1.0.801.azpkg" `
     –Verbose
    ```
5. 資源庫項目在發佈之後，現已可供使用。 若要確認是否已發佈資源庫項目，請移至 [所有服務]，然後在 [一般] 目錄下，選取 [Marketplace]。  如果您下載的是解決方案範本，請務必針對該解決方案範本新增任何相依的 VHD 映像。  
  ![檢視市集](media/azure-stack-download-azure-marketplace-item/view-marketplace.png)  

> [!NOTE]
> 隨著 Azure Stack PowerShell 1.3.0 的發行，您現在可以新增虛擬機器擴充功能。

例如︰

````PowerShell
Add-AzsVMExtension -Publisher "Microsoft" -Type "MicroExtension" -Version "0.1.0" -ComputeRole "IaaS" -SourceBlob "https://github.com/Microsoft/PowerShell-DSC-for-Linux/archive/v1.1.1-294.zip" -SupportMultipleExtensions -VmOsType "Linux"
````

## <a name="next-steps"></a>後續步驟
[建立及發佈 Marketplace 項目](azure-stack-create-and-publish-marketplace-item.md)