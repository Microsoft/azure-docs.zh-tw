---
title: 搭配使用 Azure 部署管理員與 Resource Manager 範本 | Microsoft Docs
description: 使用 Resource Manager 範本與 Azure 部署管理員來部署 Azure 資源。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/04/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 5d18a1f86e1d870db64199c575450dd475590b55
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394447"
---
# <a name="tutorial-use-azure-deployment-manager-with-resource-manager-templates-private-preview"></a>教學課程：使用 Azure Deployment Manager 搭配 Resource Manager 範本 (個人預覽版)

了解如何使用 [Azure 部署管理員](./deployment-manager-overview.md)跨多個區域部署您的應用程式。 若要使用部署管理員，您必須建立兩個範本：

* **拓撲範本**：說明構成應用程式的 Azure 資源及其部署位置。
* **首度發行範本**：說明部署應用程式時所應採取的步驟。

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 了解案例
> * 下載教學課程檔案
> * 準備成品
> * 建立使用者定義的受控識別
> * 建立服務拓撲範本
> * 建立首度發行範本
> * 部署範本
> * 驗證部署
> * 部署較新版本
> * 清除資源

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

若要完成本文，您需要：

* 開發 [Azure Resource Manager 範本](./resource-group-overview.md)的某些體驗。
* Azure Deployment Manager 為個人預覽版。 若要使用 Azure 部署管理員進行註冊，請填寫[註冊表](https://aka.ms/admsignup)。 
* Azure PowerShell。 如需詳細資訊，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。
* 部署管理員 Cmdlet。 若要安裝這些發行前版本的 Cmdlet，您需要最新版的 PowerShellGet。 若要取得最新版本，請參閱[安裝 PowerShellGet](/powershell/gallery/installing-psget)。 安裝 PowerShellGet 之後，請關閉 PowerShell 視窗。 開啟新的 PowerShell 視窗，並使用下列命令：

    ```
    Install-Module -Name AzureRM.DeploymentManager -AllowPrerelease
    ```
* [Microsoft Azure 儲存體總管](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409)。 您不一定要使用 Azure 儲存體總管，但若使用操作會更輕鬆。

## <a name="understand-the-scenario"></a>了解案例

服務拓撲範本會說明構成服務的 Azure 資源及其部署位置。 服務拓撲定義具有下列階層：

* 服務拓撲
    * 服務
        * 服務單位

下圖說明本教學課程中使用的服務拓撲：

![Azure 部署管理員教學課程案例圖表](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-scenario-diagram.png)

美國西部和美國東部位置中配置了兩項服務。  每項服務分別有兩個服務單位 - Web 應用程式前端，以及作為後端的儲存體帳戶。 服務單位定義包含用來建立 Web 應用程式和儲存體帳戶的範本和參數檔案的連結。

## <a name="download-the-tutorial-files"></a>下載教學課程檔案

1. 下載本教學課程所使用的[範本和成品](https://armtutorials.blob.core.windows.net/admtutorial/ADMTutorial.zip)。
2. 將檔案解壓縮到您所在地點的電腦。

根資料夾下有兩個資料夾：

- **ADMTemplates**：包含部署管理員範本，其中含有：
    - CreateADMServiceTopology.json
    - CreateADMServiceTopology.Parameters.json
    - CreateADMRollout.json
    - CreateADMRollout.Parameters.json
- **ArtifactStore**：包含範本成品和二進位成品。 請參閱[準備成品](#prepare-the-artifacts)。

請注意，範本共有兩組。  一組是用來部署服務拓撲和首度發行的部署管理員範本，另一組則是從服務單位呼叫、用以建立 Web 服務和儲存體帳戶的範本。

## <a name="prepare-the-artifacts"></a>準備成品

下載的 ArtifactStore 資料夾包含兩個資料夾：

![Azure 部署管理員教學課程成品來源圖表](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-artifact-source-diagram.png)

- **templates** 資料夾：包含範本成品。 **1.0.0.0** 和 **1.0.0.1** 分別代表兩個版本的二進位成品。 在每個版本中，兩項服務 (美國東部服務和美國西部服務) 並沒有個別的資料夾。 每個服務都有一組用來建立儲存體帳戶的範本和參數檔案，和一組用來建立 Web 應用程式的範本和參數檔案。 Web 應用程式範本會呼叫壓縮套件，其中包含 Web 應用程式檔案。 壓縮檔案是儲存在二進位資料夾中的二進位成品。
- **binaries** 資料夾：包含二進位成品。 **1.0.0.0** 和 **1.0.0.1** 分別代表兩個版本的二進位成品。 在每個版本中，都會有一個 zip 檔案用來建立美國西部位置的 Web 應用程式，和另一個 zip 檔案用來建立美國東部位置的 Web 應用程式。

兩個版本 (1.0.0.0 和 1.0.0.1) 會用於[修訂部署](#deploy-the-revision)。 雖然範本成品和二進位成品都有兩個版本，但只有二進位成品的兩個版本會有不同。 在實務上，二進位成品會比較範本成品更頻繁地更新。

1. 在文字編輯器中開啟 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateStorageAccount.json**。 這是用來建立儲存體帳戶的基本範本。  
2. 開啟 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplication.json**。 

    ![Azure 部署管理員教學課程建立 Web 應用程式範本](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-packageuri.png)

    範本會呼叫部署套件，其中包含 Web 應用程式的檔案。 在本教學課程中，壓縮套件僅包含 index.html 檔案。
3. 開啟 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplicationParameters.json**。 

    ![Azure 部署管理員教學課程建立 Web 應用程式範本參數 containerRoot](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-parameters-deploypackageuri.png)

    deployPackageUri 的值是部署套件的路徑。 此參數包含 **$containerRoot** 變數。 $containerRoot 的值會藉由串連成品來源 SAS 位置、成品根目錄和 deployPackageUri 提供於[首度發行範本](#create-the-rollout-template)中。
4. 開啟 **\ArtifactStore\binaries\1.0.0.0\helloWorldWebAppWUS.zip\index.html**。  

    ```html
    <html>
      <head>
        <title>Azure Deployment Manager tutorial</title>
      </head>
      <body>
        <p>Hello world from west U.S.!</p>
        <p>Version 1.0.0.0</p>
      </body>
    </html>
    ```
    HTML 中顯示位置和版本資訊。 1.0.0.1 資料夾中的二進位檔案會顯示「1.0.0.1 版」。 在部署服務之後，您可以瀏覽這些頁面。
5. 查看其他成品檔案。 這有助於您進一步了解案例。

範本成品由服務拓撲範本使用，二進位成品則由首度發行範本使用。 拓撲範本和首度發行範本都會定義成品來源 Azure 資源，此資源會用來將 Resource Manager 指向部署中使用的範本和二進位成品。 為了簡化本教學課程，我們使用一個儲存體帳戶來儲存範本成品和二進位成品。 這兩個成品來源指向相同的儲存體帳戶。

1. 建立 Azure 儲存體帳戶。 如需相關指示，請參閱[快速入門：使用 Azure 入口網站上傳、下載及列出 Blob](../storage/blobs/storage-quickstart-blobs-portal.md)。
2. 在儲存體帳戶中建立 Blob 容器。
3. 將兩個資料夾 (binaries 和 templates) 和兩個資料夾的內容複製到 Blob 容器。 [Microsoft Azure 儲存體總管](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409)支援拖放功能。
4. 使用下列指示取得容器的 SAS 位置：

    1. 在 Azure 儲存體總管中，瀏覽至 Blob 容器。
    2. 在左窗格中以滑鼠右鍵按一下 Blob 容器，然後選取 [取得共用存取簽章]。
    3. 設定 [開始時間] 和 [到期時間]。
    4. 選取 [建立] 。
    5. 複製 URL。 此 URL 必須填入下列兩個參數檔案的欄位中：[拓樸參數檔案](#topology-parameters-file)和[首度發行參數檔案](#rollout-parameters-file)。

## <a name="create-the-user-assigned-managed-identity"></a>建立使用者指派的受控識別

在教學課程的後續內容中，您會部署首度發行。 必須要有使用者指派的受控識別，才能執行部署動作 (例如，部署 Web 應用程式和儲存體帳戶)。 此身分識別必須有權存取服務要部署到的 Azure 訂用帳戶，並且有足夠的權限可完成成品部署。

您必須建立使用者指派的受控識別，並為您的訂用帳戶設定存取控制。

> [!IMPORTANT]
> 使用者指派的受控識別必須位於與[首度發行](#create-the-rollout-template)相同的位置。 目前，部署管理員資源 (包括首度發行) 只能在美國中部或美國東部 2 建立。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 建立[使用者指派的受控識別](../active-directory/managed-identities-azure-resources/overview.md)。
3. 在入口網站中，從左側功能表中選取 [訂用帳戶]，然後選取您的訂用帳戶。
4. 選取 [存取控制 (IAM)]，然後選取 [新增]
5. 輸入或選取下列值：

    ![Azure 部署管理員教學課程指派使用者的受控識別存取控制](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-access-control.png)

    - **角色**：授與足夠的權限以完成成品部署 (Web 應用程式和儲存體帳戶)。 在本教學課程中請選取 [參與者]。 在實務上，您可以限定為最低權限。
    - **存取權指派對象**：選取 [使用者指派的受控識別]。
    - 選取您在先前本教學課程中建立的使用者指派受控識別。
6. 選取 [ **儲存**]。

## <a name="create-the-service-topology-template"></a>建立服務拓撲範本

開啟 **\ADMTemplates\CreateADMServiceTopology.json**。

### <a name="the-parameters"></a>參數

範本包含下列參數：

![Azure 部署管理員教學課程拓撲範本參數](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-parameters.png)

- **namePrefix**：此前置詞用來建立部署管理員資源的名稱。 例如，若使用 "jdoe" 前置詞，服務拓撲名稱將是 **jdoe**ServiceTopology。  資源名稱會定義在此範本的變數區段中。
- **azureResourcelocation**：為了簡化本教學課程，所有資源都會共用此位置，除非另有指定。 目前，Azure 部署管理員資源只能在**美國中部**或**美國東部 2** 建立。
- **artifactSourceSASLocation**：部署的服務單位範本和參數檔案儲存所在 Blob 容器的 SAS URI。  請參閱[準備成品](#prepare-the-artifacts)。
- **templateArtifactRoot**：範本和參數儲存所在 Blob 容器的位移路徑。 預設值為 **templates/1.0.0.0**。 除非您要變更[準備成品](#prepare-the-artifacts)中說明的資料夾結構，否則請勿變更此值。 本教學課程會使用相對路徑。  完整路徑由 **artifactSourceSASLocation**、**templateArtifactRoot** 和 **templateArtifactSourceRelativePath** (或 **parametersArtifactSourceRelativePath**) 串連建構而成。
- **targetSubscriptionID**：要部署並計費的部署管理員資源的訂用帳戶識別碼。 在本教學課程中請使用您的訂用帳戶識別碼。

### <a name="the-variables"></a>變數

變數區段會定義資源的名稱、兩項服務 (**服務 WUS** 和**服務 EUS**) 的 Azure 位置，以及成品路徑：

![Azure 部署管理員教學課程拓撲範本變數](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-variables.png)

比較成品路徑與您上傳至儲存體帳戶的資料夾結構。 請注意，成品路徑是相對路徑。 完整路徑由 **artifactSourceSASLocation**、**templateArtifactRoot** 和 **templateArtifactSourceRelativePath** (或 **parametersArtifactSourceRelativePath**) 串連建構而成。

### <a name="the-resources"></a>資源

在根層級上有兩個已定義的資源：*成品來源*和*服務拓撲*。

成品來源定義為：

![Azure 部署管理員教學課程拓撲範本資源成品來源](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-artifact-source.png)

下列螢幕擷取畫面僅顯示服務拓撲、服務和服務單位定義的某些部分：

![Azure 部署管理員教學課程拓撲範本資源服務拓撲](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-service-topology.png)

- **artifactSourceId** 用來建立成品來源資源與服務拓撲資源的關聯。
- **dependsOn**：所有的服務拓撲資源皆相依於成品來源資源。
- **artifacts** 指向範本成品。  本文使用相對路徑。 完整路徑由 artifactSourceSASLocation (定義於成品來源中)、artifactRoot (定義於成品來源中) 和 templateArtifactSourceRelativePath (或 parametersArtifactSourceRelativePath) 串連建構而成。

### <a name="topology-parameters-file"></a>拓撲參數檔案

您必須建立用於拓樸範本的參數檔案。

1. 在 Visual Studio Code 或任何文字編輯器中開啟 **\ADMTemplates\CreateADMServiceTopology.Parameters**。
2. 填入參數值：

    - **namePrefix**：輸入含有 4-5 個字元的字串。 此前置詞用來建立唯一的 Azure 資源名稱。
    - **azureResourceLocation**：如果您不確定 Azure 位置，在本教學課程中請使用 **centralus**。
    - **artifactSourceSASLocation**：輸入部署的服務單位範本和參數檔案儲存所在的根目錄 (Blob 容器) 的 SAS URI。  請參閱[準備成品](#prepare-the-artifacts)。
    - **templateArtifactRoot**：除非您變更了成品的資料夾結構，否則在本教學課程中請使用 **templates/1.0.0.0**。
    - **tragetScriptionID**：輸入您的 Azure 訂用帳戶識別碼。

> [!IMPORTANT]
> 拓撲範本和首度發行範本會共用一些通用的參數。 這些參數必須具有相同的值。 這些參數是：**namePrefix**、**azureResourceLocation** 和 **artifactSourceSASLocation** (在本教學課程中，兩個成品來源會共用相同的儲存體帳戶)。

## <a name="create-the-rollout-template"></a>建立首度發行範本

開啟 **\ADMTemplates\CreateADMRollout.json**。

### <a name="the-parameters"></a>參數

範本包含下列參數：

![Azure 部署管理員教學課程首度發行範本參數](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-parameters.png)

- **namePrefix**：此前置詞用來建立部署管理員資源的名稱。 例如，若使用 "jdoe" 前置詞，首度發行檔案名稱將是 **jdoe**Rollout。  名稱會定義在範本的變數區段中。
- **azureResourcelocation**：為了簡化本教學課程，所有部署管理員資源都會共用此位置，除非另有指定。 目前，Azure 部署管理員資源只能在**美國中部**或**美國東部 2** 建立。
- **artifactSourceSASLocation**：部署的服務單位範本和參數檔案儲存所在的根目錄 (Blob 容器) 的 SAS URI。  請參閱[準備成品](#prepare-the-artifacts)。
- **binaryArtifactRoot**：預設值是為 **binaries/1.0.0.0**。 除非您要變更[準備成品](#prepare-the-artifacts)中說明的資料夾結構，否則請勿變更此值。 本教學課程會使用相對路徑。  完整路徑由 CreateWebApplicationParameters.json 中指定的 **artifactSourceSASLocation**、**binaryArtifactRoot** 和 **deployPackageUri** 串連建構而成。  請參閱[準備成品](#prepare-the-artifacts)。
- **managedIdentityID**：執行部署動作的使用者指派受控識別。 請參閱[建立使用者指派的受控識別](#create-the-user-assigned-managed-identity)。

### <a name="the-variables"></a>變數

變數區段會定義資源的名稱。 請確定服務拓撲名稱、服務名稱和服務單位名稱均符合[拓樸範本](#create-the-service-topology-template)中定義的名稱。

![Azure 部署管理員教學課程首度發行範本變數](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-variables.png)

### <a name="the-resources"></a>資源

在根層級上有三個已定義的資源：成品來源、步驟和首度發行。

成品來源定義與拓樸範本中定義的完全相同。  如需詳細資訊，請參閱[建立服務拓撲範本](#create-the-service-topology-tempate)。

下列螢幕擷取畫面顯示等候步驟定義：

![Azure 部署管理員教學課程首度發行範本資源等候步驟](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-wait-step.png)

持續時間採用 [ISO 8601 標準](https://en.wikipedia.org/wiki/ISO_8601#Durations)。 **PT1M** (必須使用大寫字母) 是等候 1 分鐘的範例。 

下列螢幕擷取畫面僅顯示首度發行定義的某些部分：

![Azure 部署管理員教學課程首度發行範本資源首度發行](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-rollout.png)

- **dependsOn**：首度發行資源相依於成品來源資源，和任何已定義的步驟。
- **artifactSourceId**：用來建立成品來源資源與首度發行資源的關聯。
- **targetServiceTopologyId**：用來建立服務拓撲資源與首度發行資源的關聯。
- **deploymentTargetId**：這是服務拓撲資源的服務單位資源識別碼。
- **preDeploymentSteps** 和 **postDeploymentSteps**：包含首度發行步驟。 在範本中會呼叫等候步驟。
- **dependsOnStepGroups**：設定步驟群組之間的相依性。

### <a name="rollout-parameters-file"></a>首度發行參數檔案

您必須建立用於首度發行範本的參數檔案。

1. 在 Visual Studio Code 或任何文字編輯器中開啟 **\ADMTemplates\CreateADMRollout.Parameters**。
2. 填入參數值：

    - **namePrefix**：輸入含有 4-5 個字元的字串。 此前置詞用來建立唯一的 Azure 資源名稱。
    - **azureResourceLocation**：目前，Azure 部署管理員資源只能在**美國中部**或**美國東部 2** 建立。
    - **artifactSourceSASLocation**：輸入部署的服務單位範本和參數檔案儲存所在的根目錄 (Blob 容器) 的 SAS URI。  請參閱[準備成品](#prepare-the-artifacts)。
    - **binaryArtifactRoot**：除非您變更了成品的資料夾結構，否則在本教學課程中請使用 **binaries/1.0.0.0**。
    - **managedIdentityID**：輸入使用者指派的受控識別。 請參閱[建立使用者指派的受控識別](#create-the-user-assigned-managed-identity)。 語法為：

        ```
        "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userassignedidentities/<ManagedIdentityName>"
        ```

> [!IMPORTANT]
> 拓撲範本和首度發行範本會共用一些通用的參數。 這些參數必須具有相同的值。 這些參數是：**namePrefix**、**azureResourceLocation** 和 **artifactSourceSASLocation** (在本教學課程中，兩個成品來源會共用相同的儲存體帳戶)。

## <a name="deploy-the-templates"></a>部署範本

Azure PowerShell 可用來部署範本。 

1. 執行部署服務拓撲的指令碼。

    ```powershell
    $deploymentName = "<Enter a Deployment Name>"
    $resourceGroupName = "<Enter a Resource Group Name>"
    $location = "Central US"  
    $filePath = "<Enter the File Path to the Downloaded Tutorial Files>"
    
    # Create a resource group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
    
    # Create the service topology
    New-AzureRmResourceGroupDeployment `
        -Name $deploymentName `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
    ```

2. 使用 Azure 入口網站確認服務拓撲和基礎資源均已成功建立：

    ![Azure 部署管理員教學課程已部署的服務拓撲資源](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

    必須選取 [顯示隱藏的類型] 才能檢視資源。

3. 部署首度發行範本：

    ```powershell
    # Create the rollout
    New-AzureRmResourceGroupDeployment `
        -Name $deploymentName `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json"
    ```

4. 使用下列 PowerShell 指令碼查看首度發行進度：

    ```powershell
    # Get the rollout status
    $rolloutname = "<Enter the Rollout Name>"
    Get-AzureRmDeploymentManagerRollout `
        -ResourceGroupName $resourceGroupName `
        -Name $rolloutName
    ```

    必須安裝部署管理員 PowerShell Cmdlet，才能執行此 Cmdlet。 請參閱[必要條件](#prerequisite)。

    下列範例顯示執行中狀態：
    
    ```
    ResourceGroupName       : adm0925rg
    BuildVersion            : 1.0.0.0
    ArtifactSourceId        : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    TargetServiceTopologyId : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/serviceTopologies/adm0925ServiceTopology
    Status                  : Running
    TotalRetryAttempts      : 0
    OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
    Services                : {adm0925ServiceEUS, adm0925ServiceWUS}
    Name                    : adm0925Rollout
    Type                    : Microsoft.DeploymentManager/rollouts
    Location                : centralus
    Id                      : /subscriptions/<SubscriptionID>/resourcegroups/adm0925rg/providers/Microsoft.DeploymentManager/rollouts/adm0925Rollout
    Tags                    :
    ```

    首度發行成功部署後，您應該會看到系統又建立了兩個資源群組，每項服務各一個。

## <a name="verify-the-deployment"></a>驗證部署

1. 開啟 [Azure 入口網站](https://portal.azure.com)。
2. 在首度發行部署所建立的新資源群組下，瀏覽至新建立的 Web 應用程式。
3. 在網頁瀏覽器中開啟 Web 應用程式。 確認 index.html 檔案的位置和版本。

## <a name="deploy-the-revision"></a>部署修訂

當 Web 應用程式有新版本 (1.0.0.1) 時， 您可以使用下列程序重新部署 Web 應用程式。

1. 開啟 CreateADMRollout.Parameters.json。
2. 將 **binaryArtifactRoot** 更新為 **binaries/1.0.0.1**。
3. 依照[部署範本](#deploy-the-templates)中的指示重新部署首度發行。
4. 依照[驗證部署](#verify-the-deployment)中的指示驗證部署。 網頁應該會顯示 1.0.0.1 版本。

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可藉由刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 使用 [依名稱篩選] 欄位，縮減在本教學課程中建立的資源群組數目。 應該會有 3-4 個：

    - **&lt;namePrefix>rg**：包含部署管理員資源。
    - **&lt;namePrefix>ServiceWUSrg**：包含 ServiceWUS 所定義的資源。
    - **&lt;namePrefix>ServiceEUSrg**：包含 ServiceEUS 所定義的資源。
    - 使用者定義受控識別的資源群組。
3. 選取資源群組名稱。  
4. 從頂端功能表中選取 [刪除資源群組]。
5. 重複最後兩個步驟，刪除本教學課程所建立的其他資源群組。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Azure 部署管理員。 若要深入了解，請參閱 [Azure Resource Manager 文件](/azure/azure-resource-manager/)。