---
title: 將您的應用程式部署至 Azure 和 Azure Stack | Microsoft Docs
description: 了解如何使用混合式 CI/CD 管線將應用程式部署至 Azure 和 Azure Stack。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: mabrigg
ms.reviewer: Anjay.Ajodha
ms.openlocfilehash: febdb2e3ae4432c36ca839f81ba7a1d333df1a2f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951996"
---
# <a name="tutorial-deploy-apps-to-azure-and-azure-stack"></a>教學課程：將應用程式部署至 Azure 和 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

了解如何使用混合式持續整合/持續傳遞 (CI/CD) 管線，將應用程式部署至 Azure 和 Azure Stack。

在本教學課程中，您會建立環境範例，用以：

> [!div class="checklist"]
> * 根據對 Azure DevOps Services 存放庫的程式碼認可起始新的組建。
> * 自動將應用程式部署至全域 Azure 以進行使用者驗收測試。
> * 在程式碼通過測試時，自動將應用程式部署至 Azure Stack。

## <a name="benefits-of-the-hybrid-delivery-build-pipe"></a>混合式傳遞建置管道的優點

持續性、安全性和可靠性是應用程式部署的重要元素。 這些元素是組織不可或缺的，且對開發小組而言至關重要。 混合式 CI/CD 管線可讓您在內部部署環境與公用雲端間整合建置管道。 混合式傳遞模型也可讓您變更部署位置，而不需要變更應用程式。

使用混合式方法的其他優點包括：

* 您可以跨內部部署 Azure Stack 環境與 Azure 公用雲端來維護一組一致的開發工具。  通用工具組可讓您更輕鬆地實作 CI/CD 模式與做法。
* 部署在 Azure 或 Azure Stack 中的應用程式和服務可以互換，並可在任一位置執行相同的程式碼。 您可以利用內部部署和公用雲端的功能。

若要深入了解 CI 和 CD：

* [什麼是持續整合？](https://www.visualstudio.com/learn/what-is-continuous-integration/)
* [什麼是持續傳遞？](https://www.visualstudio.com/learn/what-is-continuous-delivery/)

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原能力、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。

## <a name="prerequisites"></a>必要條件

您必須備妥元件，才能建置混合式 CI/CD 管線。 下列元件需要時間來做好準備：

* Azure OEM/硬體合作夥伴可部署生產 Azure Stack。 所有使用者均可重新部署 Azure Stack 開發套件 (ASDK)。
* Azure Stack 操作員也必須部署 App Service、建立方案和供應項目、建立租用戶訂用帳戶，以及新增 Windows Server 2016 映像。

>[!NOTE]
>如果您已部署其中部分元件，請在開始本教學課程之前，先確定它們符合所有需求。

本教學課程假設您擁有 Azure 和 Azure Stack 的一些基本知識。 若要在開始本教學課程前深入了解，請閱讀下列文章：

* [Azure 簡介](https://azure.microsoft.com/overview/what-is-azure/)
* [Azure Stack 重要概念](https://docs.microsoft.com/azure/azure-stack/azure-stack-key-features)

### <a name="azure-requirements"></a>Azure 需求

* 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
* 在 Azure 中建立 [Web 應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-overview)。 請記下 Web 應用程式 URL，本教學課程需要用到它。

### <a name="azure-stack-requirements"></a>Azure Stack 需求

* 使用 Azure Stack 整合系統，或部署 Azure Stack 開發套件 (ASDK)。 若要部署 ASDK：
    * [教學課程：使用安裝程式部署 ASDK](https://docs.microsoft.com/azure/azure-stack/asdk/asdk-deploy) 會提供詳細的部署指示。
    * 使用 [ConfigASDK.ps1](https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1 ) PowerShell 指令碼，將 ASDK 部署後步驟自動化。

    > [!Note]
    > ASDK 安裝程序需要約七個小時才能完成，因此請據以規劃時段。

 * 將 [App Service](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack。
 * 在 Azure Stack 中建立[方案/供應項目](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview)。
 * 在 Azure Stack 中建立[租用戶訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)。
 * 在租用戶訂用帳戶中建立 Web 應用程式。 記下新的 Web 應用程式 URL，以供後續使用。
 * 在租用戶訂用帳戶中部署 Windows Server 2012 虛擬機器。 您將以此伺服器作為組建伺服器，並用它來執行 Azure DevOps Services。
* 對虛擬機器 (VM) 提供附有 .NET 3.5 的 Windows Server 2016 映像。 此 VM 會建置在 Azure Stack 上，作為私用組建代理程式。

### <a name="developer-tool-requirements"></a>開發人員工具需求

* 建立 [Azure DevOps Services 工作區](https://docs.microsoft.com/azure/devops/repos/tfvc/create-work-workspaces)。 註冊程序會建立名為 **MyFirstProject** 的專案。
* [安裝 Visual Studio 2017](https://docs.microsoft.com/visualstudio/install/install-visual-studio) 並[登入 Azure DevOps Services](https://www.visualstudio.com/docs/setup-admin/team-services/connect-to-visual-studio-team-services)。
* 連線至專案並[在本機複製](https://www.visualstudio.com/docs/git/gitquickstart)。

 > [!Note]
 > Azure Stack 環境需要同步發佈正確的映像以執行 Windows Server 和 SQL Server。 此環境也必須部署 App Service。

## <a name="prepare-the-private-azure-pipelines-agent-for-azure-devops-services-integration"></a>準備私人 Azure Pipelines 代理程式以進行 Azure DevOps Services 整合

### <a name="prerequisites"></a>必要條件

Azure DevOps Services 會使用服務主體對 Azure Resource Manager 進行驗證。 Azure DevOps Services 必須要有**參與者**角色，才能在 Azure Stack 訂用帳戶中佈建資源。

下列步驟說明要設定驗證所必要進行的工作：

1. 建立服務主體，或使用現有服務主體。
2. 建立服務主體的驗證金鑰。
3. 透過角色型存取控制來驗證 Azure Stack 訂用帳戶，讓服務主體名稱 (SPN) 成為參與者角色的一部分。
4. 使用 Azure Stack 端點和 SPN 資訊在 Azure DevOps Services 中建立新的服務定義。

### <a name="create-a-service-principal"></a>建立服務主體

請參閱[建立服務主體](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)指示來建立服務主體。 選擇 [Web 應用程式/API] 作為 [應用程式類型]，或[使用 PowerShell 指令碼](https://github.com/Microsoft/vsts-rm-extensions/blob/master/TaskModules/powershell/Azure/SPNCreation.ps1#L5) (英文)，如[以現有的服務主體建立 Azure Resource Manager 服務連線](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal) (英文) 一文中所述。

 > [!Note]  
 > 如果您使用指令碼來建立 Azure Stack Azure Resource Manager 端點，則需要傳遞 **-azureStackManagementURL** 參數和 **-environmentName** 參數。 例如︰  
> `-azureStackManagementURL https://management.local.azurestack.external -environmentName AzureStack`

### <a name="create-an-access-key"></a>建立存取金鑰

服務主體需要金鑰來進行驗證。 請使用下列步驟來產生金鑰。

1. 在 Azure Active Directory 中，從 [應用程式註冊]選取您的應用程式。

    ![選取應用程式](media\azure-stack-solution-hybrid-pipeline\000_01.png)

2. 記下 [應用程式識別碼] 的值。 在 Azure DevOps Services 中設定服務端點時，將會使用該值。

    ![應用程式識別碼](media\azure-stack-solution-hybrid-pipeline\000_02.png)

3. 若要產生驗證金鑰，請選取 [設定]。

    ![編輯應用程式設定](media\azure-stack-solution-hybrid-pipeline\000_03.png)

4. 若要產生驗證金鑰，請選取 [金鑰]。

    ![設定金鑰設定](media\azure-stack-solution-hybrid-pipeline\000_04.png)

5. 提供金鑰的描述，並設定金鑰的持續時間。 完成時，選取 [儲存]。

    ![金鑰的描述和持續時間](media\azure-stack-solution-hybrid-pipeline\000_05.png)

    儲存金鑰之後，就會顯示金鑰**值**。 請複製此值，否則之後就無法再取得此值。 您需要提供**金鑰值**和應用程式識別碼，以應用程式身分登入。 將金鑰值儲存在應用程式可擷取的地方。

    ![金鑰值](media\azure-stack-solution-hybrid-pipeline\000_06.png)

### <a name="get-the-tenant-id"></a>取得租用戶識別碼

在設定服務端點的過程中，Azure DevOps Services 需要將**租用戶識別碼**對應至 Azure Stack 戳記已部署到的 AAD 目錄。 使用下列步驟來取得租用戶識別碼。

1. 選取 **Azure Active Directory**。

    ![租用戶的 Azure Active Directory](media\azure-stack-solution-hybrid-pipeline\000_07.png)

2. 若要取得租用戶識別碼，請選取 Azure AD 租用戶的 [屬性]。

    ![檢視租用戶屬性](media\azure-stack-solution-hybrid-pipeline\000_08.png)

3. 複製 [目錄識別碼]。 這個值是您的租用戶識別碼。

    ![目錄識別碼](media\azure-stack-solution-hybrid-pipeline\000_09.png)

### <a name="grant-the-service-principal-rights-to-deploy-resources-in-the-azure-stack-subscription"></a>授與在 Azure Stack 訂用帳戶中部署資源的服務主體權限

若要存取您的訂用帳戶中的資源，您必須將應用程式指派給角色。 請決定哪個角色最能代表應用程式的權限。 若要深入了解可用的角色，請參閱 [RBAC：內建角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)。

您可以針對訂用帳戶、資源群組或資源的層級設定範圍。 較低的範圍層級會繼承較高層級的權限。 例如，為資源群組的讀取者角色新增應用程式，代表該角色可以讀取資源群組及其任何資源。

1. 瀏覽至您想要讓應用程式指派至的範圍層級。 例如，若要在訂用帳戶範圍指派角色，請選取 [訂用帳戶]。

    ![選取訂用帳戶](media\azure-stack-solution-hybrid-pipeline\000_10.png)

2. 在 [訂用帳戶] 中，選取 [Visual Studio Enterprise]。

    ![Visual Studio Enterprise](media\azure-stack-solution-hybrid-pipeline\000_11.png)

3. 在 [Visual Studio Enterprise] 中，選取 [存取控制 (IAM)]。

    ![存取控制 (IAM)](media\azure-stack-solution-hybrid-pipeline\000_12.png)

4. 選取 [新增] 。

    ![加](media\azure-stack-solution-hybrid-pipeline\000_13.png)

5. 在 [新增權限] 中，選取您要指派給應用程式的角色。 在此範例中為**擁有者**角色。

    ![擁有者角色](media\azure-stack-solution-hybrid-pipeline\000_14.png)

6. 根據預設，Azure Active Directory 應用程式不會顯示在可用選項中。 若要尋找應用程式，您必須在 [選取] 欄位中提供其名稱，以進行搜尋。 選取應用程式。

    ![應用程式搜尋結果](media\azure-stack-solution-hybrid-pipeline\000_16.png)

7. 選取 [儲存] 以完成角色指派。 您在使用者清單中看到應用程式已指派給該範圍的角色。

### <a name="role-based-access-control"></a>角色型存取控制

‎Azure 角色型存取控制 (RBAC) 提供 Azure 的更細緻存取權管理。 藉由使用 RBAC，您可以控制使用者執行其作業所需的存取層級。 如需角色型存取控制的詳細資訊，請參閱[管理對 Azure 訂用帳戶資源的存取](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal?toc=%252fazure%252factive-directory%252ftoc.json)。

### <a name="azure-devops-services-agent-pools"></a>Azure DevOps Services 代理程式集區

您可以將代理程式組織到代理程式集區中，而不要分開管理每個代理程式。 代理程式集區會定義該集區中所有代理程式的共用界限。 在 Azure DevOps Services 中，代理程式集區的範圍限於 Azure DevOps Services 組織，這表示您可以在專案間共用代理程式集區。 若要深入了解代理程式集區，請參閱[建立代理程式集區和佇列](https://docs.microsoft.com/azure/devops/pipelines/agents/pools-queues?view=vsts)。

### <a name="add-a-personal-access-token-pat-for-azure-stack"></a>新增 Azure Stack 的個人存取權杖 (PAT)

建立個人存取權杖來存取 Azure DevOps Services。

1. 登入您的 Azure DevOps Services 組織，並選取您的組織設定檔名稱。

2. 選取 [管理安全性] 以存取權杖建立頁面。

    ![使用者登入](media\azure-stack-solution-hybrid-pipeline\000_17.png)

    ![選取專案](media\azure-stack-solution-hybrid-pipeline\000_18.png)

    ![新增個人存取權杖](media\azure-stack-solution-hybrid-pipeline\000_18a.png)

    ![建立權杖](media\azure-stack-solution-hybrid-pipeline\000_18b.png)

3. 複製權杖。

    > [!Note]
    > 儲存權杖資訊。 這項資訊不會儲存，當您離開網頁後就不會再次顯示。

    ![個人存取權杖](media\azure-stack-solution-hybrid-pipeline\000_19.png)

### <a name="install-the-azure-devops-services-build-agent-on-the-azure-stack-hosted-build-server"></a>在裝載 Azure Stack 的組建伺服器上安裝 Azure DevOps Services 組建代理程式

1. 連線至您在 Azure Stack 主機上部署的組建伺服器。
2. 下載組建代理程式並使用您的個人存取權杖 (PAT) 以服務的形式加以部署，然後以 VM 管理員帳戶執行。

    ![下載組建代理程式](media\azure-stack-solution-hybrid-pipeline\010_downloadagent.png)

3. 瀏覽至已解壓縮的組建代理程式所在的資料夾。 從提升權限的命令提示字元執行 **config.cmd** 檔案。

    ![已解壓縮的組建代理程式](media\azure-stack-solution-hybrid-pipeline\000_20.png)

    ![註冊組建代理程式](media\azure-stack-solution-hybrid-pipeline\000_21.png)

4. 當 config.cmd 完成時，組建代理程式資料夾會以其他檔案加以更新。 含有已解壓縮內容的資料夾應如下所示：

    ![組建代理程式資料夾更新](media\azure-stack-solution-hybrid-pipeline\009_token_file.png)

    您可以在 Azure DevOps Services 資料夾中看到代理程式。

## <a name="endpoint-creation-permissions"></a>端點建立權限

藉由建立端點，Visual Studio Online (VSTO) 組建可以將 Azure 服務應用程式部署到 Azure Stack。 Azure DevOps Services 會連線至組建代理程式，後者再連線至 Azure Stack。

![VSTO 中的 NorthwindCloud 應用程式範例](media\azure-stack-solution-hybrid-pipeline\012_securityendpoints.png)

1. 登入 VSTO 並瀏覽至 [應用程式設定] 頁面。
2. 在 [設定] 上，選取 [安全性]。
3. 在 [Azure DevOps Services 群組] 中，選取 [端點建立者]。

    ![NorthwindCloud 端點建立者](media\azure-stack-solution-hybrid-pipeline\013_endpoint_creators.png)

4. 在 [成員] 索引標籤上，選取 [新增]。

    ![新增成員](media\azure-stack-solution-hybrid-pipeline\014_members_tab.png)

5. 在 [新增使用者和群組] 中，輸入使用者名稱，然後從使用者清單中選取該使用者。
6. 選取 [儲存變更]。
7. 在 [Azure DevOps Services 群組] 清單中，選取 [端點管理員]。

    ![NorthwindCloud 端點管理員](media\azure-stack-solution-hybrid-pipeline\015_save_endpoint.png)

8. 在 [成員] 索引標籤上，選取 [新增]。
9. 在 [新增使用者和群組] 中，輸入使用者名稱，然後從使用者清單中選取該使用者。
10. 選取 [儲存變更]。

端點資訊已存在，所以 Azure DevOps Services 對 Azure Stack 的連線已可供使用。 Azure Stack 中的組建代理程式會取得來自 Azure DevOps Services 的指示，然後代理程式會傳達與 Azure Stack 進行通訊所需的端點資訊。
## <a name="create-an-azure-stack-endpoint"></a>建立 Azure Stack 端點

您可以依照[以現有的服務主體建立 Azure Resource Manager 服務連線](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal) (英文) 一文中的指示，使用現有的服務主體建立服務連線，並使用下列對應：

- 環境：Azure Stack
- 環境 URL：類似於 `https://management.local.azurestack.external`
- 訂用帳戶識別碼：Azure Stack 中的使用者訂用帳戶識別碼
- 訂用帳戶名稱：Azure Stack 中的使用者訂用帳戶名稱
- 服務主體用戶端識別碼：本文[這一節](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-pipeline#create-a-service-principal)中的主體識別碼。
- 服務主體金鑰：同一篇文章中的金鑰 (如果您使用指令碼，則是密碼)。
- 租用戶識別碼：您依照[取得租用戶識別碼](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-pipeline#get-the-tenant-id)中的指示，所擷取的租用戶識別碼。

現在已建立端點，所以 VSTS 到 Azure Stack 的連線已可供使用。 Azure Stack 中的組建代理程式會取得來自 VSTS 的指示，然後代理程式會傳達與 Azure Stack 進行通訊所需的端點資訊。

![組建代理程式](media\azure-stack-solution-hybrid-pipeline\016_save_changes.png)

## <a name="develop-your-application-build"></a>開發應用程式組建

在教學課程的這個部分中，您會：

* 將程式碼新增至 Azure DevOps Services 專案。
* 建立獨立的 Web 應用程式部署。
* 設定持續部署程序

> [!Note]
 > Azure Stack 環境需要同步發佈正確的映像以執行 Windows Server 和 SQL Server。 此環境也必須部署 App Service。 檢閱 App Service 文件的「必要條件」一節，以取得 Azure Stack 運算子需求。

混合式 CI/CD 可同時套用至應用程式程式碼和基礎結構程式碼。 請使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/) (例如來自 Azure DevOps Services 的 Web 應用程式程式碼) 以部署至這兩個雲端。

### <a name="add-code-to-an-azure-devops-services-project"></a>將程式碼新增至 Azure DevOps Services 專案

1. 使用在 Azure Stack 上具有專案建立權限的組織登入 Azure DevOps Services。 下一個螢幕擷取畫面顯示如何連線至 HybridCICD 專案。

    ![連線到專案](media\azure-stack-solution-hybrid-pipeline\017_connect_to_project.png)

2. 建立並開啟預設 Web 應用程式以**複製存放庫**。

    ![複製存放庫](media\azure-stack-solution-hybrid-pipeline\018_link_arm.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署

1. 編輯 **WebApplication.csproj** 檔案：選取 **Runtimeidentifier**，然後新增 `win10-x64.`。如需詳細資訊，請參閱[獨立的部署](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)文件。

    ![設定 Runtimeidentifier](media\azure-stack-solution-hybrid-pipeline\019_runtimeidentifer.png)

2. 使用 Team Explorer 將程式碼簽入 Azure DevOps Services 中。

3. 確認應用程式程式碼已簽入 Azure DevOps Services 中。

### <a name="create-the-build-pipeline"></a>建立組建管線

1. 使用可建立組建管線的組織登入 Azure DevOps Services。

2. 瀏覽至專案的 [建置 Web 應用程式] 頁面。

3. 在 [引數] 中，新增 **-r win10-x64** 程式碼。 這是觸發 .Net Core 的獨立部署時所需的程式碼。

    ![新增引數組建管線](media\azure-stack-solution-hybrid-pipeline\020_publish_additions.png)

4. 執行組建。 [獨立的部署組建](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)程序將會發佈可在 Azure 和 Azure Stack 上執行的成品。

### <a name="use-an-azure-hosted-build-agent"></a>使用 Azure 託管的組建代理程式

在 Azure DevOps Services 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動執行代理程式的維護和升級，以支援持續而不間斷的開發週期。

### <a name="configure-the-continuous-deployment-cd-process"></a>設定持續部署 (CD) 程序

Azure DevOps Services 和 Team Foundation Server (TFS) 提供具有高度設定和管理能力的管線，可用於對多個環境的發行 (例如開發、暫存、品質保證 (QA) 和生產環境)。 此程序中可以包括在應用程式生命週期的特定階段要求核准。

### <a name="create-release-pipeline"></a>建立發行管線

建立發行管線是應用程式建置程序的最後一個步驟。 這個發行管線可用來建立發行並部署組建。

1. 登入 Azure DevOps Services，並瀏覽至 **Azure Pipelines** 以找出您的專案。
2. 在 [發行] 索引標籤上，選取 **\[ + ]**，然後挑選 [建立發行定義]。

   ![建立發行管線](media\azure-stack-solution-hybrid-pipeline\021a_releasedef.png)

3. 在 [選取範本] 中，選擇 [Azure App Service 部署]，然後選取 [套用]。

    ![套用範本](media\azure-stack-solution-hybrid-pipeline\102.png)

4. 在 [新增成品] 上，從 [來源 (組建定義)] 下拉式功能表選取 [Azure 雲端建置應用程式]。

    ![新增成品](media\azure-stack-solution-hybrid-pipeline\103.png)

5. 在 [管線] 索引標籤上，選取 [檢視環境工作] 的 [1 階段 1 個工作] 連結。

    ![管線檢視工作](media\azure-stack-solution-hybrid-pipeline\104.png)

6. 在 [工作] 索引標籤上，輸入「Azure」作為 [環境名稱]，然後從 [Azure 訂用帳戶] 下拉式清單選取 [AzureCloud Traders-Web EP]。

    ![設定環境變數](media\azure-stack-solution-hybrid-pipeline\105.png)

7. 輸入 [Azure App Service 名稱]，也就是下一個螢幕擷取畫面中的「northwindtraders」。

    ![App Service 名稱](media\azure-stack-solution-hybrid-pipeline\106.png)

8. 在 [代理程式階段] 中，從 [代理程式佇列] 下拉式清單中選取 [Hosted VS2017]。

    ![裝載的代理程式](media\azure-stack-solution-hybrid-pipeline\107.png)

9. 在 [部署 Azure App Service] 中，為環境選取有效的**套件或資料夾**。

    ![選取套件或資料夾](media\azure-stack-solution-hybrid-pipeline\108.png)

10. 在 [選取檔案或資料夾] 中，對 [位置] 選取 [確定]。

    ![Alt 鍵文字](media\azure-stack-solution-hybrid-pipeline\109.png)

11. 儲存所有變更，並返回 [管線]。

    ![Alt 鍵文字](media\azure-stack-solution-hybrid-pipeline\110.png)

12. 在 [管線] 索引標籤上，選取 [新增成品]，然後從 [來源 (組建定義)] 下拉式清單選擇 [NorthwindCloud Traders-Vessel]。

    ![新增成品](media\azure-stack-solution-hybrid-pipeline\111.png)

13. 在 [選取範本] 上，新增另一個環境。 挑選 [Azure App Service 部署]，然後選取 [套用]。

    ![選取範本](media\azure-stack-solution-hybrid-pipeline\112.png)

14. 輸入「Azure Stack」作為 [環境名稱]。

    ![環境名稱](media\azure-stack-solution-hybrid-pipeline\113.png)

15. 在 [工作] 索引標籤上，尋找並選取 Azure Stack。

    ![Azure Stack 環境](media\azure-stack-solution-hybrid-pipeline\114.png)

16. 從 [Azure 訂用帳戶] 下拉式清單中，選取「AzureStack Traders-Vessel EP」作為 Azure Stack 端點。

    ![Alt 鍵文字](media\azure-stack-solution-hybrid-pipeline\115.png)

17. 輸入 Azure Stack Web 應用程式名稱作為 [App Service 名稱]。

    ![App Service 名稱](media\azure-stack-solution-hybrid-pipeline\116.png)

18. 在 [代理程式選擇] 底下，從 [代理程式佇列] 下拉式清單中挑選「AzureStack -bDouglas Fir」。

    ![挑選代理程式](media\azure-stack-solution-hybrid-pipeline\117.png)

19. 在 [部署 Azure App Service] 中，為環境選取有效的**套件或資料夾**。 在 [選取檔案或資料夾] 中，對 [位置] 資料夾選取 [確定]。

    ![挑選套件或資料夾](media\azure-stack-solution-hybrid-pipeline\118.png)

    ![核准位置](media\azure-stack-solution-hybrid-pipeline\119.png)

20. 在 [變數] 索引標籤上，尋找名為 **VSTS_ARM_REST_IGNORE_SSL_ERRORS** 的變數。 將變數值設定為 **true**，並將其範圍設定為 [Azure Stack]。

    ![設定變數](media\azure-stack-solution-hybrid-pipeline\120.png)

21. 在 [管線] 索引標籤上，選取 NorthwindCloud Traders-Web 成品的 [持續部署觸發程序] 圖示，並將 [持續部署觸發程序] 設定為 [啟用]。  針對 "NorthwindCloud Traders-Vessel" 成品執行相同的動作。

    ![設定持續部署觸發程序](media\azure-stack-solution-hybrid-pipeline\121.png)

22. 針對 Azure Stack 環境，選取 [部署前的條件] 圖示，將觸發程序設定為 [發行之後]。

    ![設定部署前的條件觸發程序](media\azure-stack-solution-hybrid-pipeline\122.png)

23. 儲存您的所有變更。

> [!Note]
> 發行工作的某些設定可能已在您從範本建立發行管線時自動定義為[環境變數](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts#custom-variables)。 您無法在工作設定中修改這些設定。 不過，您可以在父環境項目中編輯這些設定。

## <a name="create-a-release"></a>建立發行

現在，您已完成對發行管線的修改，接著即可開始進行部署。 若要這樣做，您可以從發行管線建立發行。 發行可能會建立自動；例如，持續部署觸發程序會設定於發行管線中。 這表示修改原始程式碼將會啟動新的建置，繼而產生新發行。 但在本節中，您將手動建立新發行。

1. 在 [管線] 索引標籤上，開啟 [發行] 下拉式清單，然後選擇 [建立發行]。

    ![建立發行](media\azure-stack-solution-hybrid-pipeline\200.png)

2. 輸入發行的說明，確認已選取正確的成品，然後選擇 [建立]。 幾分鐘之後將會出現一個橫幅，指出新的發行已建立，且發行名稱會顯示為連結。 選擇連結以查看 [發行摘要] 頁面。

    ![發行建立橫幅](media\azure-stack-solution-hybrid-pipeline\201.png)

3. [發行摘要] 頁面會顯示有關發行的詳細資料。 在下列 "Release-2" 螢幕擷取畫面中，[環境] 區段會顯示 Azure 的 [部署狀態] 為 [進行中]，Azure Stack 的狀態為 [成功]。 當 Azure 環境的部署狀態變更為 [成功] 時，便會出現橫幅指出發行已可供核准。 對部署若擱置或失敗，將會顯示藍色的 **(i)** 資訊圖示。 將滑鼠暫留在圖示上，即可查看快顯，其中會包含延遲或失敗的原因。

    ![發行摘要頁面](media\azure-stack-solution-hybrid-pipeline\202.png)

其他檢視 (例如發行清單) 也會顯示指出核准擱置中的圖示。 這個圖示的快顯會顯示環境名稱以及更多與部署相關的詳細資料。 系統管理員可輕鬆地查看發行的整體進度，以及查看哪些版本正在等待核准。

### <a name="monitor-and-track-deployments"></a>監視和追蹤部署

本節說明如何監控並追蹤所有部署。 用於部署兩個 Azure App Service 網站的發行可提供良好範例。

1. 在 [Release-2] 摘要頁面上，選取 [記錄]。 在部署期間，此頁面會顯示來自代理程式的即時記錄。 左窗格會顯示每個環境部署中每個作業的狀態。

    您可以選擇部署前或部署後核准 [動作] 資料行中的人形圖示，以查看哪些人核准 (或拒絕) 部署，以及該人員提供的訊息。

2. 部署完成後，整個記錄檔會顯示在右窗格中。 您可以在左窗格中選取任何 [步驟]，以查看單一步驟的記錄檔，例如「初始化作業」。 能夠查看個別記錄，可讓您輕鬆地針對整體部署的各個部分進行追蹤和偵錯。 您也可以 [儲存] 步驟的記錄檔，或是 [將所有記錄下載為 zip]。

    ![發行記錄](media\azure-stack-solution-hybrid-pipeline\203.png)

3. 開啟 [摘要] 索引標籤可查看發行的一般資訊。 此檢視會顯示組建的詳細資料、組件所部署到的環境、部署狀態，以及其他關於發行的資訊。

4. 選取環境連結 (**Azure** 或 **Azure Stack**) 可查看特定環境現有和擱置中部署的資訊。 您可以使用這些檢視來快速確認同一個組建已部署至兩個環境。

5. 在瀏覽器中開啟**已部署的生產應用程式**。 例如，針對 Azure App Service 網站，請開啟 URL `http://[your-app-name].azurewebsites.net`。

## <a name="next-steps"></a>後續步驟

* 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。
