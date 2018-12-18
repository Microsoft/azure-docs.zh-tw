---
title: 使用 Azure Resource Manager 範本建立多個 VM 環境和 PaaS 資源 | Microsoft Docs
description: 了解如何從 Azure Resource Manager 範本在 Azure DevTest Labs 中建立多個 VM 環境和 PaaS 資源
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2018
ms.author: spelluru
ms.openlocfilehash: 4dc61258df2311c4e7ccd4c05ebe077b3e2343eb
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222279"
---
# <a name="create-multi-vm-environments-and-paas-resources-with-azure-resource-manager-templates"></a>使用 Azure Resource Manager 範本建立多個 VM 環境和 PaaS 資源

[Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)可讓您輕鬆地[一次新增一個 VM 至實驗室](https://docs.microsoft.com/azure/devtest-lab/devtest-lab-add-vm)。 不過，如果環境包含多個 VM，則必須分別建立每個 VM。 針對多層式 Web 應用程式或 SharePoint 伺服器陣列的情況，需要有機制以允許在單一步驟中建立多個 VM。 使用 Azure Resource Manager 範本，您現在可以定義 Azure 方案的基礎結構和組態，並在一致的狀態中重複部署多個 VM。 此功能可提供下列優點：

- 載入 Azure Resource Manager 範本時，會直接從您的原始檔控制存放庫 (GitHub 或 Azure DevOps Services Git) 載入。
- 一旦設定後，您的使用者就可以透過從 Azure 入口網站挑選 Azure Resource Manager 範本來建立環境，就像他們使用其他類型的 [VM 基底](./devtest-lab-comparing-vm-base-image-types.md)所做的一樣。
- Azure PaaS 資源除了 IaaS VM 之外，可以從 Azure Resource Manager 範本的環境中佈建。
- 除了其他類型基底所建立的個別 VM 之外，可在實驗室中追蹤環境的成本。
- PaaS 資源會建立並出現在成本追蹤中；不過，VM 自動關機不適用於 PaaS 資源。

深入了解許多[使用 Resource Manager 範本的優點](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#the-benefits-of-using-resource-manager)，以透過單一作業部署、更新或刪除所有實驗室資源。

> [!NOTE]
> 當您使用 Resource Manager 範本作為建立其他實驗室 VM 的基礎時，不論是建立多重 VM 還是單一 VM，都請記住一些差異。 [使用虛擬機器的 Azure Resource Manager 範本](devtest-lab-use-resource-manager-template.md)會更詳細地說明這些差異。
>

## <a name="devtest-labs-public-environments"></a>DevTest Labs 公用環境
Azure DevTest Labs 有 [Azure Resource Manager 範本的公用存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Environments)，您可以用來建立環境，而不需要自行連線到外部 GitHub 來源。 此存放庫包括常用範本，例如 Azure Web Apps、Service Fabric 叢集與開發 SharePoint 伺服器陣列環境。 此功能類似您建立之每個實驗室包含之成品的公用存放庫。 環境存放庫可讓您使用預先製作的環境範本在輸入參數最少的情況下開始，以在實驗室內為您提供順暢的 PaaS 資源開始使用體驗。 如需詳細資訊，請參閱[在 DevTest Labs 中設定及使用公用環境](devtest-lab-configure-use-public-environments.md)。

## <a name="configure-your-own-template-repositories"></a>設定您自己的範本存放庫
做為基礎結構即程式碼與組態即程式碼的其中一個最佳作法，應在原始檔控制中管理環境範本。 Azure DevTest Labs 依循此作法，而會直接從您的 GitHub 或 Azure DevOps Services Git 存放庫載入所有 Azure Resource Manager 範本。 因此，可以在測試環境到生產環境的整個發行週期使用 Resource Manager 範本。

請參閱[公用 GitHub 存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Environments)中由 DevTest Labs 小組所建立的範本。 在公用存放庫中，您可以檢視其他人分享的範本，並且可以直接使用這些範本，或根據您的需求來自訂這些範本。 建立範本之後，請將它儲存在這個存放庫以與他人分享。 您也可以使用可用來在雲端中設定環境的範本，來設定您自己的 Git 存放庫。 

您可以遵循幾個規則，以在存放庫中組織 Azure Resource Manager 範本︰

- 主要的範本檔案必須命名為 `azuredeploy.json`。 

    ![金鑰 Azure Resource Manager 範本檔案](./media/devtest-lab-create-environment-from-arm/master-template.png)

- 如果您想要使用參數檔案中定義的參數值，參數檔案必須命名為 `azuredeploy.parameters.json`。
- 您可以使用 `_artifactsLocation` 和 `_artifactsLocationSasToken` 參數來建構 parametersLink URI 值，讓 DevTest Labs 自動管理巢狀範本。 如需詳細資訊，請參閱 [How Azure DevTest Labs makes nested Resource Manager template deployments easier for testing environments](https://blogs.msdn.microsoft.com/devtestlab/2017/05/23/how-azure-devtest-labs-makes-nested-arm-template-deployments-easier-for-testing-environments/) (Azure DevTest Labs 如何簡化測試環境的巢狀 Resource Manager 範本部署)。
- 可以定義中繼資料來指定範本顯示名稱和描述。 此中繼資料必須在名為 `metadata.json` 的檔案中。 下列範例中繼資料檔案說明如何指定顯示名稱和描述︰ 

    ```json
    { 
        "itemDisplayName": "<your template name>", 
        "description": "<description of the template>" 
    }
    ```

下列步驟將引導您使用 Azure 入口網站將存放庫新增至您的實驗室。 

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
1. 從實驗室清單中，選取所需的實驗室。   
1. 在實驗室的 [概觀] 窗格中，選取 [設定與原則]。

    ![組態和原則](./media/devtest-lab-create-environment-from-arm/configuration-and-policies-menu.png)

1. 從 [組態和原則] 設定清單中，選取 [存放庫]。 [存放庫] 窗格會列出已新增至實驗室的存放庫。 會為所有實驗室自動產生名為 `Public Repo` 的存放庫，並連接到 [DevTest Labs GitHub 存放庫](https://github.com/Azure/azure-devtestlab)，其中包含數個 VM 構件供您使用。

    ![公用存放庫](./media/devtest-lab-create-environment-from-arm/public-repo.png)

1. 選取 [新增 +] 以新增您的 Azure Resource Manager 範本存放庫。
1. 當第二個 [存放庫] 窗格開啟時，輸入必要資訊，如下所示：
    - **名稱** - 輸入在實驗室中使用的存放庫名稱。
    - **Git 複製 URL** - 輸入來自 GitHub 或 Azure DevOps Services 的 GIT HTTPS 複製 URL。  
    - **分支** - 輸入分支的名稱，以存取您的 Azure Resource Manager 範本定義。 
    - **個人的存取權杖** - 個人的存取權杖用來安全地存取您的存放庫。 若要從 Azure DevOps Services 取得您的權杖，請選取 [&lt;您的名稱] > > [我的設定檔] > [安全性] > [公用存取權杖]。 若要從 GitHub 取得權杖，請選取 avatar 然後選取 [設定 > 公用存取權杖]。 
    - **資料夾路徑** - 使用兩個輸入欄位其中之一、輸入資料夾路徑，其開頭為正斜線 - / - 且與 Git 複製 URI 相關，至構件定義 (第一個輸入欄位) 或您的 Azure Resource Manager 範本定義。   
    
        ![公用存放庫](./media/devtest-lab-create-environment-from-arm/repo-values.png)


1. 一旦輸入所有必要的欄位並通過驗證後，請選取 [儲存]。

下一節將引導您從 Azure Resource Manager 範本建立環境。

## <a name="create-an-environment-from-a-resource-manager-template-using-the-azure-portal"></a>使用 Azure 入口網站從 Resource Manager 範本建立環境

一旦在實驗室中設定 Azure Resource Manager 範本存放庫之後，您實驗室的使用者可以使用 Azure 入口網站以下列步驟建立環境︰

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
1. 從實驗室清單中，選取所需的實驗室。   
1. 在實驗室的窗格中，選取 [新增+]。
1. [選擇基底] 窗格會顯示您可以與先列出的 Azure Resource Manager 範本搭配使用的基底映像。 選取所需的 Azure Resource Manager 範本。

    ![選擇基底](./media/devtest-lab-create-environment-from-arm/choose-a-base.png)
  
1. 在 [新增] 窗格中，輸入**環境名稱**值。 環境名稱是實驗室中向您使用者顯示的內容。 會在 Azure Resource Manager 範本中定義其餘的輸入欄位。 如果在範本中定義預設值或 `azuredeploy.parameter.json` 檔案存在，則預設值會顯示在這些輸入欄位中。 針對安全字串類型的參數，您可以使用儲存在 Azure 金鑰保存庫中的密碼。 若要深入了解如何在金鑰保存庫中儲存祕密以及在建立實驗室資源時使用它們，請參閱[在 Azure Key Vault 中儲存祕密](devtest-lab-store-secrets-in-key-vault.md)。  

    ![新增窗格](./media/devtest-lab-create-environment-from-arm/add.png)

    > [!NOTE]
    > 有幾個參數值 (即使指定) 會顯示為空值。 因此，如果使用者將那些值指派給 Azure Resource Manager 範本中的參數，DevTest Labs 就不會顯示值。 而是會顯示空白的輸入欄位，實驗室使用者必須在建立環境時於其中輸入值。
    > 
    > - GEN-UNIQUE
    > - GEN-UNIQUE-[N]
    > - GEN-SSH-PUB-KEY
    > - GEN-PASSWORD 
 
1. 選取 [新增] 以建立環境。 環境會立即啟動佈建，並在**我的虛擬機器**清單中有狀態顯示。 實驗室會自動建立新的資源群組，以佈建 Azure Resource Manager 範本中定義的所有資源。
1. 建立環境之後，在 [我的虛擬機器] 清單中選取環境，以開啟資源群組窗格，並瀏覽環境中佈建的所有資源。
    
    ![我的虛擬機器清單](./media/devtest-lab-create-environment-from-arm/all-environment-resources.png)
   
   您也可以展開環境，只檢視環境中所佈建的 VM 清單。
    
    ![我的虛擬機器清單](./media/devtest-lab-create-environment-from-arm/my-vm-list.png)

1. 按一下任何環境，以檢視可用的動作 - 例如套用構件、加入資料磁碟、變更自動關機時間等等。

    ![環境動作](./media/devtest-lab-create-environment-from-arm/environment-actions.png)

## <a name="deploy-a-resource-manager-template-to-create-a-vm"></a>部署 Resource Manager 範本以建立 VM
儲存 Resource Manager 範本並且針對需求自訂之後，您可以使用它來自動化 VM 建立。 
- [使用 Resource Manager 範本與 Azure PowerShell 來部署資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)說明如何使用 Azure PowerShell 與 Resource Manager 範本以將資源部署至 Azure。 
- [使用 Resource Manager 範本與 Azure CLI 來部署資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli)說明如何使用 Azure CLI 與 Resource Manager 範本以將資源部署至 Azure。

> [!NOTE]
> 只有具備實驗室擁有者權限的使用者才可以使用 Azure PowerShell 從 Resource Manager 範本建立 VM。 如果您想要使用 Resource Manager 範本自動化 VM 建立，而且您只有使用者權限，您可以使用 [CLI 中的 **az lab vm create** 命令](https://docs.microsoft.com/cli/azure/lab/vm#az-lab-vm-create)。

### <a name="general-limitations"></a>一般限制 

在 DevTest Labs 中使用 Resource Manager 範本時，請考慮下列限制：

- 您所建立的任何 Resource Manager 範本都無法參考現有的資源；它只能參考新的資源。 此外，如果您通常會將現有的 Resource Manager 範本部署於 DevTest Labs 外部，且其包含對現有資源的參考，則無法在實驗室中使用它。

   這其中唯一的例外是您**可以**參考現有的虛擬網路。 

- 您無法從建立自 Resource Manager 範本的實驗室 VM 建立公式。 

- 您無法從建立自 Resource Manager 範本的實驗室 VM 建立自訂映像。 

- 當您部署 Resource Manager 範本時，大部分的原則都不會進行評估。

   例如，您的實驗室原則可能指定使用者只能建立 5 個 VM。 不過，使用者可以部署會建立數十個 VM 的 Resource Manager 範本。 不會評估的原則包括：

   - 每位使用者的 VM 數目
   - 每位實驗室使用者的進階 VM 數目
   - 每位實驗室使用者的進階磁碟數目


### <a name="configure-environment-resource-group-access-rights-for-lab-users"></a>針對實驗室使用者設定環境資源群組存取權限

實驗室使用者可以部署 Resource Manager 範本。 但根據預設，他們具備讀者存取權限，這表示他們無法變更環境資源群組中的資源。 例如，他們無法停止或啟動其資源。

請遵循下列步驟，為您的實驗室使用者提供參與者存取權限。 接著，當他們部署 Resource Manager 範本時，他們將能編輯該環境中的資源。 


1. 在實驗室的 [概觀] 窗格中，選取 [設定與原則]。
1. 選取 [實驗室設定]。
1. 在 [實驗室設定] 窗格中，選取 [參與者]，為實驗室使用者授與寫入權限。

    ![設定實驗室使用者存取權限](./media/devtest-lab-create-environment-from-arm/configure-access-rights.png)

1. 選取 [ **儲存**]。

## <a name="next-steps"></a>後續步驟
* 一旦建立 VM 之後，您就能選取 VM 管理窗格上的 [連線] 來連線至 VM。
* 在實驗室的 [我的虛擬機器] 清單中選取環境，以檢視和管理環境中的資源。 
* [從 Azure 快速入門範本資源庫中探索 Azure Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates)。
