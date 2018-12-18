---
title: 在 Azure Stack 中建立和發佈 Marketplace 項目 | Microsoft Azure
description: 在 Azure Stack 中建立和發佈 Marketplace 項目。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/14/2018
ms.author: sethm
ms.reviewer: jeffgo
ms.openlocfilehash: 9e579123124615df83483e244ef11810ca590844
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45633958"
---
# <a name="create-and-publish-a-marketplace-item"></a>建立及發行 Marketplace 項目

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="create-a-marketplace-item"></a>建立 Marketplace 項目
1. [下載](http://www.aka.ms/azurestackmarketplaceitem) Azure Gallery Packager 工具和範例 Azure Stack Marketplace 項目。
2. 開啟範例 Marketplace 項目，並重新命名 **SimpleVMTemplate** 資料夾。 (使用與您的 Marketplace 項目相同的名稱 - 例如 **Contoso.TodoList**。)此資料夾包含：
   
       /Contoso.TodoList/
       /Contoso.TodoList/Manifest.json
       /Contoso.TodoList/UIDefinition.json
       /Contoso.TodoList/Icons/
       /Contoso.TodoList/Strings/
       /Contoso.TodoList/DeploymentTemplates/
3. [建立 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)，或從 GitHub 選擇範本。 Marketplace 項目會使用此範本來建立資源。

    > [!Note]  
    > 絕對不要硬式編碼任何祕密 (例如 Azure Resource Manager 範本中的產品金鑰、密碼或任何客戶識別資訊)。 範本 JSON 檔案只要發佈至資源庫，就可供存取，而且不需要驗證。  將所有祕密都儲存至 [Key Vault](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-keyvault-parameter)，並從範本呼叫它們。

4. 若要確定可以成功部署資源，請使用 Microsoft Azure Stack API 來測試範本。
5. 如果您的範本依賴於虛擬機器映像，請遵循指示來[將虛擬機器映像新增至 Azure Stack](azure-stack-add-vm-image.md)。
6. 將 Azure Resource Manager 範本儲存在 **/Contoso.TodoList/DeploymentTemplates/** 資料夾中。
7. 選擇您的 Marketplace 項目的圖示和文字。 將圖示新增至 **Icons** 資料夾，以及將文字新增至 **Strings** 資料夾的 **resources** 檔案中。 使用 Small、Medium、Large 和 Wide 圖示命名慣例。 如需詳細的說明，請參閱 [Marketplace 項目 UI 參考](#reference-marketplace-item-ui)。
   
   > [!NOTE]
   > 需要全部四個圖示大小 (小、中、大、寬) 才能正確建立 Marketplace 項目。
   > 
   > 
8. 在 **manifest.json** 檔案中，將 [名稱] 變更為您的 Marketplace 項目的名稱。 同時將 [發行者] 變更為您的姓名或公司。
9. 在 [構件] 下，將 [名稱] 和 [路徑] 變更為您所併入 Azure Resource Manager 範本的正確資訊。
   
         "artifacts": [
            {
                "name": "Type your template name",
                "type": "Template",
                "path": "DeploymentTemplates\\Type your path",
                "isDefault": true
            }
10. 將**我的 Marketplace 項目**取代為您的 Marketplace 項目應出現位置的目錄清單。
    
             "categories":[
                 "My Marketplace Items"
              ],
11. 若要對 manifest.json 進行任何進一步的編輯，請參閱[參考：Marketplace 項目 manifest.json](#reference-marketplace-item-manifestjson)。
12. 若要將資料夾封裝到 .azpkg 檔案中，請開啟命令提示字元，然後執行下列命令：
    
        AzureGalleryPackager.exe package –m <path to manifest.json> -o <output location for the package>
    
    > [!NOTE]
    > 輸出套件的完整路徑必須存在。 例如，如果輸出路徑是 C:\MarketPlaceItem\yourpackage.azpkg，資料夾 C:\MarketPlaceItem 必須存在。
    > 
    > 

## <a name="publish-a-marketplace-item"></a>發佈 Marketplace 項目
1. 使用 PowerShell 或 Azure 儲存體總管來將您的 Marketplace 項目 (.azpkg) 上傳至 Azure Blob 儲存體。 您可以上傳至本機 Azure Stack 儲存體，或上傳至 Azure 儲存體。 (它是套件的暫存位置。)請確定 Blob 可公開存取。
2. 在 Microsoft Azure Stack 環境中的用戶端虛擬機器上，確定您的 PowerShell 工作階段已設有您服務系統管理員的認證。 您可以在[使用 PowerShell 部署範本](user/azure-stack-deploy-template-powershell.md)中找到如何在 Azure Stack 中驗證 PowerShell 的指示。
3. 當您使用 [PowerShell 1.3.0]( azure-stack-powershell-install.md) 或更新版本時，可以使用 **Add-AzsGalleryItem** PowerShell Cmdlet 將 Marketplace 項目發佈至 Azure Stack。 在使用 PowerShell 1.3.0 之前，使用 **Add-AzureRMGalleryitem** Cmdlet 代替 **Add-AzsGalleryItem**。  例如，當您使用 PowerShell 1.3.0 或更新版本時：
   
       Add-AzsGalleryItem -GalleryItemUri `
       https://sample.blob.core.windows.net/gallerypackages/Microsoft.SimpleTemplate.1.0.0.azpkg –Verbose
   
   | 參數 | 說明 |
   | --- | --- |
   | SubscriptionID |管理訂用帳戶 ID。 您可以使用 PowerShell 來擷取它。 如果您偏好在入口網站中取得它，請移至提供者的訂用帳戶，並複製該訂用帳戶識別碼。 |
   | GalleryItemUri |已上傳至儲存體的您的資源庫套件的 Blob URI。 |
   | Apiversion |設定為 **2015-04-01**。 |
4. 移至入口網站。 身為操作員或使用者，現在可以在入口網站中看到 Marketplace 項目。
   
   > [!NOTE]
   > 套件可能需要幾分鐘才會出現。
   > 
   > 
5. 您的 Marketplace 項目現在已儲存至 Azure Stack Marketplace。 您可以選擇將它從您的 Blob 儲存體位置刪除。
    > [!Caution]  
    > 現在，透過下列 URL，可以未經驗證存取所有預設資源庫成品和您的自訂資源庫成品：  
`https://adminportal.[Region].[external FQDN]:30015/artifact/20161101/[Template Name]/DeploymentTemplates/Template.json`  
`https://portal.[Region].[external FQDN]:30015/artifact/20161101/[Template Name]/DeploymentTemplates/Template.json`  
`https://systemgallery.blob.[Region].[external FQDN]/dev20161101-microsoft-windowsazure-gallery/[Template Name]/UiDefinition.json`

6. 使用 **Remove-AzureRMGalleryItem** Cmdlet 可移除 Marketplace 項目。 範例：
   
        Remove-AzsGalleryItem -Name Microsoft.SimpleTemplate.1.0.0  –Verbose
   
   > [!NOTE]
   > 移除項目之後，Marketplace UI 可能會顯示錯誤。 若要修正錯誤，請按一下入口網站中的 [設定]。 然後，在 [入口網站自訂] 之下，選取 [捨棄修改]。
   > 
   > 

## <a name="reference-marketplace-item-manifestjson"></a>參考：Marketplace 項目 manifest.json
### <a name="identity-information"></a>身分識別資訊
| 名稱 | 必要 | 類型 | 條件約束 | 說明 |
| --- | --- | --- | --- | --- |
| 名稱 |X |字串 |[A-a-za-z0-9] + | |
| 發行者 |X |字串 |[A-a-za-z0-9] + | |
| 版本 |X |字串 |[SemVer v2](http://semver.org/) | |

### <a name="metadata"></a>中繼資料
| 名稱 | 必要 | 類型 | 條件約束 | 說明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |字串 |建議 80 個字元 |如果超過 80 個字元，入口網站可能無法正常顯示項目名稱。 |
| PublisherDisplayName |X |字串 |建議 30 個字元 |如果超過 30 個字元，入口網站可能無法正常顯示發行者名稱。 |
| PublisherLegalName |X |字串 |上限 256 個字元 | |
| 總結 |X |字串 |60 到 100 個字元 | |
| LongSummary |X |字串 |140 到 256 個字元 |目前在 Azure Stack 中不適用。 |
| 說明 |X |[HTML](https://auxdocs.azurewebsites.net/en-us/documentation/articles/gallery-metadata#html-sanitization) |500 到 5,000 個字元 | |

### <a name="images"></a>映像
Marketplace 會使用下列圖示：

| 名稱 | 寬度 | 高度 | 注意 |
| --- | --- | --- | --- |
| 寬 |255 像素 |115 像素 |一律需要 |
| 大型 |115 像素 |115 像素 |一律需要 |
| 中 |90 像素 |90 像素 |一律需要 |
| 小型 |40 像素 |40 像素 |一律需要 |
| 螢幕擷取畫面 |533 像素 |32 像素 |選用 |

### <a name="categories"></a>類別
每個 Marketplace 項目應該以可識別項目出現在入口網站 UI 上位置的類別加以標記。 您可以在 Azure Stack 中選擇其中一個現有的類別 (運算、資料 + 儲存體等等) 或選擇新的類別。

### <a name="links"></a>連結
每個 Marketplace 項目可以包含各種其他內容的連結。 連結已指定為名稱與 URI 的清單。

| 名稱 | 必要 | 類型 | 條件約束 | 說明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |字串 |上限 64 個字元 | |
| Uri |X |URI | | |

### <a name="additional-properties"></a>其他屬性
除了前述中繼資料，Marketplace 作者可以下列形式提供自訂的成對索引鍵/值資料：

| 名稱 | 必要 | 類型 | 條件約束 | 說明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |字串 |上限 25 個字元 | |
| 值 |X |字串 |上限 30 個字元 | |

### <a name="html-sanitization"></a>HTML 病毒掃描
任何可使用 HTML 的欄位，都可以使用下列項目與屬性：

h1、h2、h3、h4、h5、p、ol、ul、li、a[target|href]、br、strong、em、b、i

## <a name="reference-marketplace-item-ui"></a>參考：Marketplace 項目 UI
在 Azure Stack 入口網站中所看見 Marketplace 項目的圖示和文字如下所示。

### <a name="create-blade"></a>建立刀鋒視窗
![建立刀鋒視窗](media/azure-stack-marketplace-item-ui-reference/image1.png)

### <a name="marketplace-item-details-blade"></a>Marketplace 項目詳細資料刀鋒視窗
![Marketplace 項目詳細資料刀鋒視窗](media/azure-stack-marketplace-item-ui-reference/image3.png)

