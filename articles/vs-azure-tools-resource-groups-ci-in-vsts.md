---
title: 使用 Azure 資源群組專案在 Azure DevOps Services 中進行連續整合 | Microsoft Docs
description: 使用 Visual Studio 中的 Azure 資源群組部署專案在 Azure DevOps Services 中進行持續整合。
services: visual-studio-online
documentationcenter: na
author: mlearned
manager: erickson-doug
editor: ''
ms.assetid: b81c172a-be87-4adc-861e-d20b94be9e38
ms.service: azure-resource-manager
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/01/2016
ms.author: mlearned
ms.openlocfilehash: f44bb7bd95ef405c65bb259a6d104475c2e283bd
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297837"
---
# <a name="continuous-integration-in-azure-devops-services-using-azure-resource-group-deployment-projects"></a>使用 Azure 資源群組部署專案在 Azure DevOps Services 中進行持續整合
若要部署 Azure 範本，您需要執行各種階段的工作：組建、測試、複製到 Azure (也稱為「暫存」) 及部署範本。 有兩種不同的方式，可將範本部署到 Azure DevOps Services。 兩種方法所產生的結果都相同，因此請選擇最符合您工作流程的方法。

1. 在執行 PowerShell 指令碼的組建管線 (包含在 Azure 資源群組部署專案，Deploy-AzureResourceGroup.ps1) 中新增一個步驟。 指令碼會複製構件，接著部署範本。
2. 新增多個 Azure DevOps Services 組建步驟，每個都執行一個階段工作。

本文將示範兩個選項。 第一個選項的優點是能夠使用開發人員在 Visual Studio 中所使用的相同指令碼，並在整個生命週期中提供一致性。 第二個選項可提供內建指令碼的方便替代選項。 這兩個程序假設您已經在 Azure DevOps Services 中簽入 Visual Studio 部署專案。

## <a name="copy-artifacts-to-azure"></a>將構件複製到 Azure
無論何種情況，如果您有範本部署所需的任何構件，則必須提供存取權給 Azure Resource Manager。 這些構件包括下列檔案：

* 巢狀範本
* 組態指令碼及 DSC 指令碼
* 應用程式二進位檔

### <a name="nested-templates-and-configuration-scripts"></a>巢狀範本和組態指令碼
當您使用 Visual Studio (或以 Visual Studio 程式碼片段建置的) 提供的範本時，PowerShell 指令碼不但會暫存構件，也會參數化資源 URI 以進行不同的部署。 接著，指令碼會將構件複製到 Azure 中的安全容器，並為該容器建立 SaS 權杖，再將該資訊傳遞至範本部署。 請參閱 [建立範本部署](https://msdn.microsoft.com/library/azure/dn790564.aspx) 以深入了解巢狀範本。  在 Azure DevOps Services 中使用工作時，您必須針對範本部署選取適當的工作，且如果必要，必須從預備步驟將參數值傳遞至範本部署。

## <a name="set-up-continuous-deployment-in-azure-pipelines"></a>在 Azure Pipelines 中設定持續部署
若要在 Azure Pipelines 中呼叫 PowerShell 指令碼，請更新您的組建管線。 簡單來說，請執行下列步驟： 

1. 編輯組建管線。
2. 在 Azure Pipelines 中設定 Azure 授權。
3. 新增參考 Azure 資源群組部署專案中 PowerShell 指令碼的 Azure PowerShell 組建步驟。
4. 設定 *-ArtifactsStagingDirectory* 參數值，以與 Azure Pipelines 中建置的專案搭配使用。

### <a name="detailed-walkthrough-for-option-1"></a>選項 1 的詳細逐步解說
下列程序會使用單一工作在您的專案中執行 PowerShell 指令碼，逐步引導您進行在 Azure DevOps Services 中設定持續部署所需的步驟。 

1. 編輯您的 Azure DevOps Services 組建管線並新增 Azure PowerShell 組建步驟。 在 [組建管線] 類別下選擇組建管線，再選擇 [編輯] 連結。
   
   ![編輯組建管線][0]
2. 在組建管線中新增 [Azure PowerShell] 組建步驟，再選擇 [新增組建步驟...] 按鈕。
   
   ![新增組建步驟][1]
3. 選擇 [部署工作] 類別，選取 [Azure PowerShell] 工作，再選擇其 [新增] 按鈕。
   
   ![新增工作][2]
4. 選擇 [Azure PowerShell]  組建步驟，再填上其值。
   
   1. 如果您已將 Azure 服務端點新增至 Azure DevOps Services，請在 [Azure 訂用帳戶] 下拉式清單方塊中選擇訂用帳戶，並跳至下一節。 
      
      如果您的 Azure DevOps Services 中沒有 Azure 服務端點，請新增一個。 此訂用帳戶會帶您完成此程序。 如果您的 Azure 帳戶使用 Microsoft 帳戶 (例如 Hotmail)，您必須進行下列步驟以取得服務主體驗證。

   2. 選擇 [Azure 訂用帳戶] 下拉式清單方塊旁的 [管理] 連結。
      
      ![管理 Azure 訂用帳戶][3]
   3. 在 [新服務端點] 下拉式清單方塊中選擇 [Azure]。
      
      ![新增服務端點][4]
   4. 在 [新增 Azure 訂用帳戶] 對話方塊中，選取 [服務主體] 選項。
      
      ![服務主體選項][5]
   5. 在 [新增 Azure 訂用帳戶]  對話方塊中新增 Azure 訂用帳戶資訊。 您必須先提供下列項目：
      
      * 訂用帳戶識別碼
      * 訂用帳戶名稱
      * 服務主體識別碼
      * 服務主體金鑰
      * 租用戶識別碼
   6. 在 [訂用帳戶]  名稱方塊中新增您選擇的名稱。 此值稍後會出現在 Azure DevOps Services 的 [Azure 訂用帳戶] 下拉式清單中。 

   7. 如果您不知道 Azure 訂用帳戶識別碼，可以使用以下其中一個命令擷取。
      
      對於 Azure PowerShell 指令碼，請使用：
      
      `Get-AzureRmSubscription`
      
      對於 Azure CLI，請使用：
      
      `azure account show`
   8. 若要取得服務主體識別碼、服務主體金鑰及租用戶識別碼，請依照[使用入口網站建立 Active Directory 應用程式和服務主體](resource-group-create-service-principal-portal.md)或[以 Azure 資源管理員驗證服務主體](resource-group-authenticate-service-principal.md)中的程序。
   9. 將服務主體識別碼、服務主體金鑰，以及租用戶識別碼值新增至 [新增 Azure 訂用帳戶] 對話方塊，然後選擇 [確定] 按鈕。
      
      現在，您擁有可執行 Azure PowerShell 指令碼的有效服務主體。
5. 編輯組建管線並選擇 **Azure PowerShell** 組建步驟。 在 [Azure 訂用帳戶] 下拉式清單方塊中選取訂用帳戶。 (如果訂用帳戶未出現，請選擇 [管理] 連結旁的 [重新整理]按鈕。) 
   
   ![安裝並設定 Azure PowerShell 建置工作][8]
6. 提供 Deploy-AzureResourceGroup.ps1 PowerShell 指令碼的路徑。 若要這樣做，請選擇 [指令碼路徑] 方塊旁的省略符號 (…) 按鈕，瀏覽到您專案的 [指令碼] 資料夾中的 Deploy-AzureResourceGroup.ps1 PowerShell 指令碼，選取並選擇 [確定] 按鈕。    
   
   ![選取要編寫指令碼的路徑][9]
7. 選取指令碼之後，將路徑更新到該指令碼，以便從 Build.StagingDirectory 執行 (與 *ArtifactsLocation* 設定的目錄相同)。 您可以在指令碼路徑的開頭加入 “$(Build.StagingDirectory)/” 以執行此項作業。
   
    ![選取要編寫指令碼的路徑][10]
8. 在 [指令碼引數]  方塊中，輸入下列參數 (請輸入在同一行)。 當您在 Visual Studio 中執行指令碼時，可以在 [輸出]  視窗中看到 VS 如何使用參數。 您可以從這裡開始，在建置步驟中設定參數值。
   
   | 參數 | 說明 |
   | --- | --- |
   | -ResourceGroupLocation |資源群組所在的地理位置，例如 **eastus** 或**美國東部**。 (如果名稱中有空間，請加入單引號。)如需詳細資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。 |
   | -ResourceGroupName |此部署使用的資源群組名稱。 |
   | -UploadArtifacts |此參數出現時，可指定需要從本機系統上傳到 Azure 的構件。 您只需要在範本部署需要您希望暫存的其他構件時，使用 PowerShell 指令碼設定此轉換 (例如組態指令碼或巢狀範本) 即可。 |
   | -StorageAccountName |用來暫存此部署之構件的儲存體帳戶名稱。 僅在您執行部署的成品時才會使用這個參數。 如果提供這個參數，則如果指令碼在先前部署期間尚未建立儲存體帳戶，便會建立新的儲存體帳戶。 如果指定參數，則儲存體帳戶必須已經存在。 |
   | -StorageAccountResourceGroupName |與儲存體帳戶相關聯的資源群組名稱。 僅在您提供 StorageAccountName 參數的值時，才需要此參數。 |
   | -TemplateFile |Azure 資源群組部署專案中的範本檔案路徑。 為了提高彈性，請使用與 PowerShell 指令碼相關的參數路徑，不要使用絕對路徑。 |
   | -TemplateParametersFile |Azure 資源群組部署專案中的參數檔案路徑。 為了提高彈性，請使用與 PowerShell 指令碼相關的參數路徑，不要使用絕對路徑。 |
   | -ArtifactStagingDirectory |此參數可讓 PowerShell 指令碼從應該複製專案二進位檔的位置取得資料夾。 這個值會覆寫 PowerShell 指令碼使用的預設值。 若要供 Azure DevOps Services 使用，請將值設為：-ArtifactStagingDirectory $(Build.StagingDirectory) |
   
   以下是指令碼引數的範例 (斷行以方便閱讀)：
   
   ```    
   -ResourceGroupName 'MyGroup' -ResourceGroupLocation 'eastus' -TemplateFile '..\templates\azuredeploy.json' 
   -TemplateParametersFile '..\templates\azuredeploy.parameters.json' -UploadArtifacts -StorageAccountName 'mystorageacct' 
   –StorageAccountResourceGroupName 'Default-Storage-EastUS' -ArtifactStagingDirectory '$(Build.StagingDirectory)'    
   ```
   
   完成之後，[指令碼引數] 方塊應該如下清單所示：
   
   ![指令碼引數][11]
9. 將所有必要的項目都加入 Azure PowerShell 建置步驟之後，請選擇 [佇列]  建置按鈕以建置專案。 [建置]  畫面會顯示 PowerShell 指令碼的輸出。

### <a name="detailed-walkthrough-for-option-2"></a>選項 2 的詳細逐步解說
下列程序會使用內建工作，逐步引導您進行在 Azure DevOps Services 中設定持續部署所需的步驟。

1. 編輯您的 Azure DevOps Services 組建管線，以新增兩個新的組建步驟。 在 [組建定義] 類別下選擇組建管線，再選擇 [編輯] 連結。
   
   ![編輯組建定義][12]
2. 將新的組建步驟新增至組建管線，方法為使用 [新增組建步驟...] 按鈕。
   
   ![新增組建步驟][13]
3. 選擇 [部署工作] 類別，選取 [Azure 檔案複製] 工作，再選擇其 [新增] 按鈕。
   
   ![新增 Azure 檔案複製工作][14]
4. 選擇 [Azure 資源群組部署] 工作，然後選擇其 [新增] 按鈕，然後 [關閉] [工作目錄]。
   
   ![新增 [Azure 資源群組部署] 工作][15]
5. 選擇 **Azure 檔案複製**工作並填入其值。
   
   如果您已將 Azure 服務端點新增至 Azure DevOps Services，請在 [Azure 訂用帳戶] 下拉式清單方塊中選擇訂用帳戶。 如果您沒有訂用帳戶，請參閱[選項 1](#detailed-walkthrough-for-option-1)，以查看在 Azure DevOps Services 中設定訂用帳戶的指示。
   
   * 來源 - 輸入 **$(Build.StagingDirectory)**
   * Azure 連線類型 - 選取 **Azure Resource Manager**
   * Azure RM 訂用帳戶 - 在 [Azure 訂用帳戶] 下拉式清單方塊中，選取您想要使用之儲存體帳戶的訂用帳戶。 如果訂用帳戶未出現，請選擇 [管理] 連結旁的 [重新整理]按鈕。
   * 目的地類型 - 選取 **Azure Blob**
   * RM 儲存體帳戶 - 選取您想要針對預備構件使用的儲存體帳戶
   * 容器名稱 - 輸入您想要用於暫存的容器名稱，可以是任何有效的容器名稱，但請使用專用於此組建管線的名稱
   
   輸出值︰
   
   * 儲存體容器 URI - 輸入 **artifactsLocation**
   * 儲存體容器 SAS - 輸入 **artifactsLocationSasToken**
   
   ![設定 Azure 檔案複製工作][16]
6. 選擇 [Azure 資源群組部署]  組建步驟，再填上其值。
   
   * Azure 連線類型 - 選取 **Azure Resource Manager**
   * Azure RM 訂用帳戶 - 在 [Azure 訂用帳戶] 下拉式清單方塊中選取部署的訂用帳戶。 這通常與上一個步驟中使用的訂用帳戶相同
   * 動作 - 選取 **建立或更新資源群組**
   * 資源群組 - 選取資源群組或輸入部署的新資源群組名稱
   * 位置 - 選取資源群組的位置
   * 範本 - 輸入要部署之範本的名稱與路徑前面加上 **$(Build.StagingDirectory)**，例如︰**$(Build.StagingDirectory/DSC-CI/azuredeploy.json)**
   * 範本參數 - 輸入要使用之參數的名稱與路徑，前面加上 **$(Build.StagingDirectory)**，例如︰**$(Build.StagingDirectory/DSC-CI/azuredeploy.parameters.json)**
   * 覆寫範本參數 - 輸入或複製並貼上下列程式碼︰
     
     ```    
     -_artifactsLocation $(artifactsLocation) -_artifactsLocationSasToken (ConvertTo-SecureString -String "$(artifactsLocationSasToken)" -AsPlainText -Force)
     ```
     ![設定 [Azure 資源群組部署] 工作][17]
7. 在新增所有必要的項目之後，儲存組建管線然後在頂端選擇 [將新組建排入佇列]。

## <a name="next-steps"></a>後續步驟
如需 Azure Resource Manager 和 Azure 資源群組的詳細資訊的詳細資訊，請參閱 [Azure Resource Manager 概觀](azure-resource-manager/resource-group-overview.md) 。

[0]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough1.png
[1]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough2.png
[2]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough3.png
[3]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough4.png
[4]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough5.png
[5]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough6.png
[8]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough9.png
[9]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough10.png
[10]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough11b.png
[11]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough12.png
[12]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough13.png
[13]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough14.png
[14]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough15.png
[15]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough16.png
[16]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough17.png
[17]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough18.png
