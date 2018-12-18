---
title: 使用 Azure 和 Azure Stack 建立邊緣機器學習解決方案 | Microsoft Docs
description: 了解如何使用 Azure 和 Azure Stack 以 Python 建立邊緣機器學習解決方案。
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
ms.date: 09/26/2018
ms.author: mabrigg
ms.reviewer: Anjay.Ajodha
ms.openlocfilehash: 28ff8dbf073596e5f9565c56ae903af6af68f3e2
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353701"
---
# <a name="tutorial-create-an-edge-machine-learning-solution-with-azure-and-azure-stack"></a>教學課程：使用 Azure 和 Azure Stack 建立邊緣機器學習解決方案

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

了解如何使用 Azure 和 Azure Stack 建立邊緣機器學習解決方案。

在本教學課程中，您將建置範例環境，用以：

> [!div class="checklist"]
> - 建立儲存體帳戶和容器以供存放乾淨資料。
> - 在 Azure 入口網站中建立 Ubuntu 資料科學虛擬機器 (DSVM)。
> - 在 Azure 中部署 Azure Machine Learning 服務，以便建置模型並加以定型。
> - 建立 Azure Machine Learning 服務帳戶。
> - 部署和使用 Azure Container Registry。
> - 將 Kubernetes 叢集部署至 Azure Stack。
> - 建立資料的 Azure Stack 儲存體帳戶和儲存體佇列。
> - 建立新的 Azure Stack 函式，以將乾淨資料從 Azure Stack 移至 Azure。

**此解決方案的使用時機**

 -  貴組織使用 DevOps 方法，或具有一個打算在不久的將來使用的方法。
 -  您想要跨 Azure Stack 實作和公用雲端來實作 CI/CD 做法。
 -  您想要跨雲端和內部部署環境來合併 CI/CD 管線。
 -  您想要能夠使用雲端或內部部署服務來開發應用程式。
 -  您想要跨雲端和內部部署應用程式來運用一致的開發人員技能。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。

## <a name="prerequisites"></a>必要條件

建置此使用案例需要幾個元件，您可能需要花一點時間來做好準備：

 -  Azure OEM/硬體合作夥伴可部署生產 Azure Stack，而所有使用者均可部署 ASDK

 -  Azure Stack 操作員也必須部署 App Service、建立方案和供應項目、建立租用戶訂用帳戶，以及新增 Windows Server 2016 映像

 -  需要混合式網路和 App Service 安裝程式 (請深入了解[應用程式與 VNet 的整合。](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet))

 -  必須在部署前先設定私用[組建和發行代理程式](https://docs.microsoft.com/vsts/pipelines/agents/agents?view=vsts)以供進行 VSTS 整合 (請確定所使用的現有元件都符合需求後再開始進行。)

必須先對 Azure 和 Azure Stack 有所了解。 若要先深入了解再繼續，請從下列主題著手：

 -  [Azure 簡介](https://azure.microsoft.com/overview/what-is-azure/)

 -  [Azure Stack 重要概念](https://docs.microsoft.com/azure/azure-stack/azure-stack-key-features)

 -  [Azure Stack 混合式 CI/CD 解決方案指南](/azure/azure-stack/user/azure-stack-solution-pipeline)

**Azure**

 -  Azure 訂用帳戶 (建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F))

 -  [Web 應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-overview)在 Azure 中所建立的新 Web 應用程式 URL

 -  [在 Azure 上部署 Azure Container Services (ACS) Kubernetes](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-template-kubernetes-deploy)

 -  Azure Machine Learning 服務 (預覽) 部署 [4 部分教學課程](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/tutorial-classifying-iris-part-1)

 -  Azure Machine Learning 測試[帳戶](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/experimentation-service-template)

**Azure Stack**

 -  Azure Stack 整合系統，或部署 Azure Stack 開發套件。

    - 您可以在[安裝 Azure Stack 開發套件](/articles/azure-stack/asdk/asdk-install)中找到如何安裝 Azure Stack 的指示。
     - [https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1](https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1) 此安裝可能需要幾個小時才能完成。

 -  將 [App Service](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack

 -  Azure Stack 環境中的[方案/供應項目](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview)

 -  Azure Stack 環境中的[租用戶訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)

 -  Ubuntu Server VM 映像 (可於 [Azure Stack Marketplace](https://buildazure.com/2016/05/04/azure-marketplace-ubuntu-server-16-04-lts/) 取得)。此 VM 會建置在 Azure Stack 上的租用戶訂用帳戶中，以作為私用組建代理程式以及 Kubernetes VM。 如果無法取得此映像，可向 Azure Stack 操作員尋求協助，以確實地將此映像新增至環境中。 (請勿使用 18.04 組建的 Ubuntu，因為其目前不受支援。)

 -  租用戶訂用帳戶中的 Web 應用程式 (請記下新的 Web 應用程式 URL 以供稍後使用。)

 -  在租用戶訂用帳戶中部署 VSTS Linux 架構私用組建代理程式虛擬機器

 -  [在 Azure Stack 上部署 Azure Container Services (ACS) Kubernetes](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-solution-template-kubernetes-deploy)

**開發人員工具**

 -  [VSTS 工作區](https://www.visualstudio.com/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services) (註冊程序會建立名為 "MyFirstProject" 的專案)

 -  [Visual Studio 2017](https://docs.microsoft.com/visualstudio/install/install-visual-studio) 安裝和 [VSTS](https://www.visualstudio.com/docs/setup-admin/team-services/connect-to-visual-studio-team-services) 登入

 -  專案的[本機複製品](https://www.visualstudio.com/docs/git/gitquickstart)

 -  安裝[適用於 Windows 10 的 Linux 子系統](https://docs.microsoft.com/windows/wsl/install-win10) (適用於 BASH 和 SSH)

 -  安裝[適用於 Windows 的 Docker](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

 -  安裝 [Azure Machine Learning Workbench (預覽)](https://aka.ms/azureml-wb-msi)

 -  安裝 [Python 環境](https://www.python.org/ftp/python/3.7.0/python-3.7.0rc1-amd64.exe)

**VSTS**

 -  **VSTS 帳戶。** 快速設定用於建置、測試和部署的持續整合。 如需 VSTS 帳戶的詳細資訊，請參閱[建立 VSTS 帳戶](https://docs.microsoft.com/vsts/accounts/create-account-msa-or-work-student?view=vsts)。

 -  **程式碼存放庫。** 將現有程式碼存放庫用於 GitHub、BitBucket、DropBox、One Drive 和 TFS 中，可讓 VSTS 平台利用多個程式碼存放庫來簡化開發管線。 如需程式碼存放庫的詳細資訊，請參閱[開始使用 Git 和 VSTS](https://docs.microsoft.com/vsts/git/gitquickstart?view=vsts&tabs=visual-studio) 教學課程。

 -  **服務連線。** 連線至外部和遠端服務，以執行測試、建置或部署工作。 如需 VSTS 服務連線的詳細資訊，請參閱[組建和發行的服務端點](https://docs.microsoft.com/vsts/build-release/concepts/library/service-endpoints?view=vsts)。

 -  **組建定義。** 定義自動化建置程序，並透過工作目錄撰寫一組工作。 如需組建定義的詳細資訊，請參閱[建立組建定義](https://docs.microsoft.com/vsts/build-release/actions/ci-cd-part-1?view=vsts)文件。

 -  **發行定義。** 針對應用程式成品部署所在的環境集合定義部署程序。 如需發行定義的詳細資訊，請參閱[建立發行定義](https://docs.microsoft.com/vsts/build-release/actions/ci-cd-part-1?view=vsts)文件。

 -  **裝載的 VSTS Linux 組建代理程式集區。** 使用 Microsoft 所管理和維護的裝載代理程式，快速建置、測試和部署應用程式。 如需所裝載 VSTS 組建代理程式的詳細資訊，請參閱[裝載的代理程式](https://docs.microsoft.com/vsts/build-release/concepts/agents/hosted?view=vsts)文件。

## <a name="step-1-create-a-storage-account"></a>步驟 1：建立儲存體帳戶

建立儲存體帳戶和容器以供存放乾淨資料。

1.  登入 [*Azure 入口網站*](https://portal.azure.com/)。

2.  在 Azure 入口網站中，展開左側功能表以開啟服務的功能表，然後選擇 [所有服務]。 向下捲動至 [儲存體]，然後選擇 [儲存體帳戶]。 在 [儲存體帳戶] 視窗中，選擇 [新增]。

3.  輸入儲存體帳戶的名稱。

    > [!Note]  
    > 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。 儲存體帳戶名稱在 Azure 中必須是獨一無二的。 Azure 入口網站會指出所選取的儲存體帳戶名稱是否已在使用中。

4.  指定所要使用的部署模型：[Resource Manager]。

5.  選取儲存體帳戶類型：[一般用途 v1]，然後指定效能層︰[標準]。

6.  選取儲存體帳戶的複寫選項︰[GRS]。

7.  選取新的儲存體帳戶訂用帳戶。

8.  指定新的資源群組，或選取現有的資源群組。

9.  選取儲存體帳戶的地理位置。

10. 選取 [建立]  以建立儲存體帳戶。

    ![替代文字](\media\azure-stack-solution-machine-learning\image1.png)

11.  選擇最近建立的儲存體帳戶。

12.  選取 [Blob]。

    ![替代文字](media\azure-stack-solution-machine-learning\image2.png)

13.  選取 [+ 容器]，然後選取 [容器]。

    ![替代文字](media\azure-stack-solution-machine-learning\image3.png)

14.  將容器命名為 **uploadeddata**，然後選擇 [容器] 存取類型。

15.  選取 [建立]。

    ![替代文字](media\azure-stack-solution-machine-learning\image4.png)

## <a name="step-2-create-a-data-science-virtual-machine"></a>步驟 2：建立資料科學虛擬機器

在 Azure 入口網站中建立 Ubuntu 資料科學虛擬機器 (DSVM)。

1.  從 [https://portal.azure.com](https://portal.azure.com/) 登入 Azure 入口網站

2.  選取 [+新增] 連結，並搜尋「適用於 Linux Ubuntu CSP 的資料科學虛擬機器」

    ![替代文字](media\azure-stack-solution-machine-learning\image5.png)

1.  選擇清單中的 [適用於 Linux (Ubuntu) 的資料科學虛擬機器]，並遵循建立 DSVM 的螢幕指示。

    ![替代文字](media\azure-stack-solution-machine-learning\image6.png)

> ![重要]  
> 請**選擇** [密碼] 作為 [驗證類型]。

將新的 DSVM 放在與新建立的儲存體帳戶相同的資源群組中。 所有 Edge ML 物件全都會部署到 Azure 中的這個資源群組內。

1.  在 [設定] 中 設定選用功能

    a.  選取稍早建立的 [儲存體帳戶]。

    b.  建立新的 [虛擬網路]、[子網路] 和 [公用 IP]。它預設應該會根據資源群組名稱來建立名稱。

    c.  建立新的 [NSG]。它應該會自動建立已套用正確規則的這個項目。

    d.  在 [診斷儲存體帳戶] 中，選取稍早建立的儲存體帳戶。

    e.  注意：為 Azure 訂用帳戶啟用並設定 AAD 後，也可以一併啟用 Azure 資源的受控識別。

2.  選取 [確定] 。

### <a name="connect-to-the-dsvm"></a>連線至 DSVM

DSVM 建立好之後，請從適用於 Linux 的 Windows 子系統連線至 VM。

```Bash  
    ssh <user>@<DSVM Public IP>
```

1.  當系統提示您確認 VM 連線時，請輸入 YES。

2.  輸入為 DSVM 所建立的密碼。

### <a name="update-the-dsvm"></a>更新 DSVM 

```Bash  
sudo su 
apt-get update 
apt-get -y upgrade 
apt-get -y dist-upgrade 
apt-get -y autoremove
```

## <a name="step-3-deploy-azure-machine-learning-services"></a>步驟 3：部署 Azure Machine Learning 服務

在 Azure 中部署 Azure Machine Learning 服務。

Azure Machine Learning 服務 (預覽) 是整合式端對端的資料科學和進階分析解決方案。 它可以協助專業資料科學家以雲端規模準備資料、開發測試及部署模型。

本節說明：

> [!div class="checklist"]
> - 為 Azure Machine Learning 服務建立服務帳戶
> - 安裝並登入 Azure Machine Learning Workbench。
> - 在 Workbench 中建立專案
> - 在該專案中執行指令碼
> - 存取命令列介面 (CLI)

Azure Machine Learning 服務是 Microsoft Azure 產品組合的一部分，因此需要 Azure 訂用帳戶。 若要取得 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

您必須具備適當的權限，才能建立「資源群組」、「虛擬機器」等資產。

您可以在下列作業系統上安裝 Azure Machine Learning Workbench 應用程式：

 -  Windows 10 或 Windows Server 2016
 -  macOS Sierra 或 High Sierra

## <a name="step-4-create-azure-machine-learning-services"></a>步驟 4：建立 Azure Machine Learning 服務

建立 Azure Machine Learning 服務帳戶。

使用 Azure 入口網站來佈建 Azure Machine Learning 帳戶：

1.  使用要供 Azure 訂用帳戶使用的認證來登入 [Azure 入口網站](https://portal.azure.com/)。 若要取得 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

    ![替代文字](media\azure-stack-solution-machine-learning\image7.png)

1.  選取入口網站左上角的 [建立資源] 按鈕 (+)。

    ![在 Azure 入口網站中建立資源](media\azure-stack-solution-machine-learning\image8.png)

1.  在搜尋列中輸入「Machine Learning」。 選取名為 **Machine Learning 測試 (預覽)** 的搜尋結果。

    ![Azure Machine Learning 搜尋](media\azure-stack-solution-machine-learning\image9.png)

1.  在 [Machine Learning 測試] 窗格中，捲動到底部並選取 [建立] 即可開始定義測試帳戶。

    ![Azure Machine Learning - 建立測試帳戶](media\azure-stack-solution-machine-learning\image10.png)

1.  在 [ML 測試] 窗格中，設定 Machine Learning 測試帳戶。

    | 設定 | 教學課程的建議值 | 說明 |
    |---------------------------------------|----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | 測試帳戶名稱 | 唯一的名稱 | 輸入可識別帳戶的唯一名稱。 使用最能識別此測試的部門或專案名稱。 這個名稱長度應介於 2 到 32 個字元之間。 應該只包含英數字元及虛線 (-) 字元。 |
    | 訂用帳戶 | 訂用帳戶 | 選擇要用於測試的 Azure 訂用帳戶。 如果有多個訂用帳戶存在，請選擇資源計費的適當訂用帳戶。 |
    | 資源群組 | 資源群組 | 使用訂用帳戶中的現有資源群組，或輸入一個名稱來為此測試帳戶建立新的資源群組。 |
    | 位置 | 最接近使用者的區域 | 選擇最接近使用者與資料資源的位置。 |
    | 基座數目 | 2 | 輸入基座數目。 了解[基座如何影響定價](https://azure.microsoft.com/pricing/details/machine-learning/)。<br><br>本快速入門只需要兩個基座。 您可以視需要在 Azure 入口網站中可以新增或移除基座。 |
    | 儲存體帳戶 | 唯一的名稱 | 選取 [建立新的]，並且提供名稱以建立 [Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=portal)。 名稱必須為 3 到 24 個字元，且只能包含英數字元。 或者，選取 [使用現有的]，然後從清單中選取現有的儲存體帳戶。 需要儲存體帳戶，並且是用來儲存專案構件並執行歷程記錄資料。 |
    | 測試帳戶的工作區 | IrisGarden<br>(教學課程中使用的名稱) | 提供此帳戶的工作區名稱。 這個名稱長度應介於 2 到 32 個字元之間。 應該只包含英數字元及虛線 (-) 字元。 此工作區包含建立、管理及發行實驗所需的工具。 |
    | 指派工作區的擁有者 | 帳戶 | 選取身分為工作區擁有者的自有帳戶。 |
    | 建立模型管理帳戶 | **檢查** | 建立模型管理帳戶。 此帳戶可用來將模型部署為即時 Web 服務並進行管理。 <br><br>雖為選擇性，但建議您在與測試帳戶的同時建立模型管理帳戶。 |
    | 帳戶名稱 | 唯一的名稱 | 選擇可識別模型管理員帳戶的唯一名稱。 使用最能識別此測試的部門或專案名稱。 這個名稱長度應介於 2 到 32 個字元之間。 應該只包含英數字元及虛線 (-) 字元。 |
    | 模型管理定價層 | **DEVTEST** | 選取 [未選取任何定價層] 來指定新模型管理帳戶的定價層。 為了節省成本，請選取 DEVTEST 定價層 (如果在訂用帳戶上可用，且限量提供)。 否則選取 S1 定價層。 選擇 [選取] 以儲存定價層選取項目。 |
    | 釘選到儀表板 | 核取 | 選取 [釘選到儀表板] 選項，在 Azure 入口網站的前儀表板頁面上輕鬆追蹤 Machine Learning 測試帳戶。 |

    ![Machine Learning 測試帳戶組態](media\azure-stack-solution-machine-learning\image11.png)

1.  選取 [建立] 開始建立測試帳戶及模型管理帳戶的程序。

    ![Machine Learning 測試帳戶組態](media\azure-stack-solution-machine-learning\image12.png)

    建立帳戶可能需要一些時間。 選取 Azure 入口網站工具列上的 [通知] 圖示 (鈴鐺)，即可檢查部署程序的狀態。

    ![Azure 入口網站通知](media\azure-stack-solution-machine-learning\image13.png)

### <a name="install-and-log-in-to-workbench"></a>安裝及登入 Workbench 

Azure Machine Learning Workbench 適用於 Windows 或 macOS。 請查看[支援的平台](https://docs.microsoft.com/azure/machine-learning/service/quickstart-installation)清單。

**警告：** 安裝作業可能需要大約 1 小時才能完成。

1.  下載並啟動最新的 Workbench 安裝程式。

    > [!Important]  
    > 在磁碟上完整下載安裝程式，然後從該處執行它。 不要直接從瀏覽器的下載小工具執行它。<br>**在 Windows 上：<br>** a. 下載 [AmlWorkbenchSetup.msi](https://aka.ms/azureml-wb-msi)。<br>b. 在檔案總管中按兩下所下載的安裝程式。

1.  請依照安裝程式中畫面上的指示來完成。

    **安裝作業最慢可能需要 30 分鐘的時間才能完成。 **
    
    `Windows: C:\\Users\\<user>\\AppData\\Local\\AmlWorkbench`
    
    安裝程式將會下載並設定所有必要的相依項目，例如 Python、Miniconda 及其他相關程式庫。 此安裝也包括 Azure 跨平台命令列工具或 Azure CLI。

1.  選取安裝程式最後一個畫面上的 [啟動 Workbench] 按鈕以啟用 Workbench。

    如果安裝程式已關閉，請使用 **Machine Learning Workbench** 桌面捷徑加以啟動。

1.  選取 [使用 Microsoft 帳戶登入] 以向 Azure Machine Learning Workbench 進行驗證。 使用在 Azure 入口網站中用來建立測試和模型管理帳戶的相同認證。

    登入之後，Workbench 會使用它在 Azure 訂用帳戶中找到的第一個測試帳戶，並顯示與該帳戶相關聯的所有工作區和專案。

    > [!Tip]  
    > 使用 Workbench 應用程式視窗左下角的圖示，切換為不同的測試帳戶。

### <a name="create-a-new-project-in-workbench"></a>在 Workbench 中建立新的專案

1.  開啟 Azure Machine Learning Workbench 應用程式，並登入 (如果需要)。

    - 在 Windows 上，使用 **Machine Learning Workbench** 桌面捷徑來加以啟動。

    - 在 macOS 上，選取啟動列中的 **Azure ML Workbench**。

1.  在 [專案] 窗格中選取加號 (+)，然後選擇 [新增專案]。

    ![新增工作區](media\azure-stack-solution-machine-learning\image14.png)

1.  填妥表單欄位並選取 [建立] 按鈕，以在 Workbench 中建立新專案。

    | 欄位 | 教學課程的建議值 | 說明 |
    |-------------------------------------|------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | 專案名稱 | myIris | 輸入可識別帳戶的唯一名稱。 使用最能識別此測試的部門或專案名稱。 這個名稱長度應介於 2 到 32 個字元之間。 應該只包含英數字元及虛線 (-) 字元。 |
    | 專案目錄 | c:\Temp\ | 指定要在其中建立專案的目錄。 |
    | 專案描述 | 保留空白 | 適合用於描述專案的選擇性欄位。 |
    | Visualstudio.com GIT 存放庫 URL | 保留空白 | 選擇性欄位。 在 Visual Studio Team Services 上讓專案與 Git 存放庫相關聯，以便進行原始檔控制和共同作業。 [了解如何設定存放庫](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/using-git-ml-project)。 |
    | 選取的工作區 | IrisGarden (如果存在的話) | 選擇在 Azure 入口網站中針對測試帳戶建立的工作區。 <br>使用快速入門，便會列出名為 IrisGarden 的工作區。 否則，使用名稱為測試帳戶或具有慣用帳戶名稱的工作區。 |
    | 專案範本 | 分類鳶尾花 | 範本包含可用來瀏覽產品的指令碼和資料。 此範本包含在本快速入門及此文件網站中的其他教學課程中需要使用的指令碼和資料。 |

    ![新增專案](media\azure-stack-solution-machine-learning\image15.png)

1.  建立新的專案，專案儀表板隨即開啟並顯示該專案。 瀏覽專案首頁、資料來源、筆記本及原始程式檔。

    ![開啟專案](media\azure-stack-solution-machine-learning\image16.png)

### <a name="attach-a-dsvm-compute-target"></a>連結 DSVM 計算目標

建立 DSVM 之後，請將它連結至 Azure ML 專案。

1.  從 Azure ML Workbench 應用程式內，選取 [檔案]->[開啟 PowerShell] 來啟動 Azure ML Workbench CLI

    ![替代文字](media\azure-stack-solution-machine-learning\image17.png)

1.  PowerShell 提示字元開啟後，使用下列命令：

    ```PowerShell  
        az login
    ```

1.  收到下列提示：

     ![替代文字](media\azure-stack-solution-machine-learning\image18.png)

1.  瀏覽至提示中詳述的網站，並輸入所提供的程式碼。

    ![替代文字](media\azure-stack-solution-machine-learning\image19.png)

1.  在出現提示時選取 [繼續]，然後選取 Azure ML 實驗帳戶所關聯的 Azure 帳戶。

    ![替代文字](media\azure-stack-solution-machine-learning\image20.png)

1.  Azure ML Workbench CLI 會接著傳送下列提示：

    ![替代文字](media\azure-stack-solution-machine-learning\image21.png)

1.  當 ML 帳戶和工作區的登入顯示為成功時，連結 DSVM。

    ```PowerShell  
        az ml computetarget attach remotedocker --name <compute target name> --address <ip address or FQDN> --username <admin username> --password <admin password>
    ```

    隨即會出現下列通知：

    ![替代文字](media\azure-stack-solution-machine-learning\image22.png)

    ```PowerShell  
        # prepare the Docker image on the DSVM 
        az ml experiment prepare -c <compute target name>
    ```

此程序需要進行一段時間，並且會在取下各種 Docker 映像、加以註冊，然後對這些映像套用所需的程式碼與應用程式時產生大量文字。

現在可以在此 DSVM 上執行實驗。

### <a name="create-a-data-preparation-package"></a>建立資料準備封裝

接下來，在 Azure Machine Learning Workbench 中開始準備資料。 在 Workbench 中執行的每個轉換都會以 JSON 格式儲存在本機資料準備套件 (\*.dprep 檔案) 中。 此資料準備套件是資料準備工作在 Workbench 中的主要容器。

此資料準備套件稍後可遞交給執行階段，例如 local-C\#/CoreCLR、Scala/Spark 或 Scala/HDI。

1.  選取資料夾圖示，即可開啟 [檔案] 檢視，然後選取 **iris.csv** 來開啟該檔案。

    此檔案包含具有 5 個資料行和 50 個資料列的資料表。 四個資料行是數值特徵資料行。 第五個資料行是字串目標資料行。 所有資料行都沒有標頭名稱。

    ![iris.csv](media\azure-stack-solution-machine-learning\image23.png)

1.  在 [資料檢視] 中，選取加號 (**+**) 來新增新的資料來源。 [新增資料來源] 頁面隨即開啟。

    ![Azure Machine Learning Workbench 中的資料檢視](media\azure-stack-solution-machine-learning\image24.png)

1.  選取 [文字檔] (\*.csv、\*.json、\*.txt 等)。

    ![Azure Machine Learning Workbench 中的資料來源](media\azure-stack-solution-machine-learning\image25.png)

1.  選取 [下一步] 。

2.  瀏覽至 **iris.csv** 檔案，然後選取 [完成]。 這會使用參數的預設值，例如分隔符號和資料類型。

    > [!Important]  
    > 針對此練習，請從目前的專案目錄內選取 **iris.csv** 檔案。 否則，接下來的步驟可能會失敗。

    ![選取鳶尾花](media\azure-stack-solution-machine-learning\image26.png)

1.  名為 `*iris-1.dsource` 的新檔案隨即建立。 由於範例專案中隨附未編號的 **iris.dsource** 檔案，因此會以 `-1` 為檔案指定唯一的名稱。

    檔案隨即開啟，並且顯示資料。 從 **Column1** 至 **Column5** 的一系列資料行標題，會自動新增到此資料集。 捲動到底部，並注意資料集的最後一個資料列是空的。 資料列是空的，因為 CSV 檔案中有額外的分行符號。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image27.png)

1.  選取 [計量] 按鈕。 隨即產生並顯示長條圖。

    選取 [資料] 按鈕，即可切換回到資料檢視。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image28.png)

1.  觀察長條圖。 每個資料行會計算出一組完整的統計資料。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image29.png)

1.  選取 [準備] 按鈕，開始建立資料準備套件。 [準備] 對話方塊隨即開啟。

    此範例專案包含預設選取的 **iris.dprep** 資料準備檔案。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image30.png)

1.  從功能表中選取 [+ 新增資料準備套件]，以建立新的資料準備套件。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image31.png)

1.  輸入新的值作為套件名稱 (使用 **iris-1**)，然後選取 [確定]。

    名為 **iris-1.dprep** 的新資料準備套件隨即建立，並在資料準備編輯器中開啟。

    ![鳶尾花資料檢視](media\azure-stack-solution-machine-learning\image32.png)

    接下來需要準備資料。

1.  選取各個資料行標頭，讓標頭文字可進行編輯。 然後，將每個資料行重新命名，如下所示：

    分別針對五個資料行，依序輸入 [萼片長度]、[萼片寬度]、[花瓣長度]、[花瓣寬度] 和 [物種]。

    ![將資料行重新命名](media\azure-stack-solution-machine-learning\image33.png)

1.  計算相異值：

    1.  選取 [物種] 資料行。

    2.  按一下滑鼠右鍵加以選取。

    3.  從功能表中選取 [值計數]。

        [偵測器] 窗格會在資料下方開啟。 此時會出現有四個橫條的長條圖。 目標資料行有四個相異值：**Iris-virginica**、**Iris-versicolor**、**Iris-setosa** 和 **(null)** 值。

    ![選取值計數](media\azure-stack-solution-machine-learning\image34.png)

    ![值計數長條圖](media\azure-stack-solution-machine-learning\image35.png)

1.  若要篩選出 null 值，請選取 [(null)] 長條，然後選取減號 (**-**)。

    接著，(null) 資料列會變成灰色，表示它已被篩選掉。

    ![篩選掉 null](media\azure-stack-solution-machine-learning\image36.png)

1.  請注意 [步驟] 窗格中詳述的個別資料準備步驟。 將資料行重新命名並篩選 null 值資料列時，每個動作都會記錄為資料準備步驟。 編輯個別的步驟，以調整其設定、重新排列步驟，並移除步驟。

    ![替代文字](media\azure-stack-solution-machine-learning\image37.png)

1.  關閉資料準備編輯器。 在包含圖形圖示的 **iris-1** 索引標籤上，選取 **x** 圖示以關閉索引標籤。工作會自動儲存到顯示在 [資料準備] 標題之下的 **iris-1.dprep** 檔案中。

    ![關閉](media\azure-stack-solution-machine-learning\image38.png)

### <a name="generate-python-code-to-invoke-a-data-preparation-package"></a>產生 Python 程式碼來叫用資料準備套件

您可直接在 Python 或 Jupyter Notebook 中探索資料準備套件的輸出。 套件可以跨多個執行階段執行，包括本機 Python、Spark (包含於 Docker 中) 及 HDInsight。

1.  在 [資料準備] 索引標籤下尋找 **iris-1.dprep** 檔案。

2.  以滑鼠右鍵按一下 **iris-1.dprep** 檔案，然後從快顯功能表中選取 [產生資料存取碼檔案]。

    ![產生程式碼](media\azure-stack-solution-machine-learning\image39.png)

    名為 **iris-1.py** 的新檔案隨即開啟，其中包含下列幾行程式碼，以便叫用建立為資料準備套件的邏輯：

    ```Python
    # Use the Azure Machine Learning data preparation package
    from azureml.dataprep import package

    # Use the Azure Machine Learning data collector to log various metrics
    from azureml.logging import get_azureml_logger
    logger = get_azureml_logger()

    # This call will load the referenced package and return a DataFrame.
    # If run in a PySpark environment, this call returns a
    # Spark DataFrame. If not, it will return a Pandas DataFrame.
    df = package.run('iris-1.dprep', dataflow_idx=0)
    # Remove this line and add code that uses the DataFrame
    df.head(10)
    ```

    視執行此程式碼所在的內容而定，drep 代表不同種類的 DataFrame：

    -  在 Python 執行階段上執行時，會使用 [pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)。

    -  在 Spark 內容中執行時，則會使用 [Spark 資料框架](https://spark.apache.org/docs/latest/sql-programming-guide.html)。

### <a name="review-irissklearnpy-and-the-configuration-files"></a>檢閱 iris_sklearn.py 和組態檔

1.  在開啟的專案中，選取最左邊窗格上的 [檔案] 按鈕 (資料夾圖示)，以在專案資料夾中開啟檔案清單。

    ![開啟 Azure Machine Learning Workbench 專案](media\azure-stack-solution-machine-learning\image40.png)

1.  選取 **iris_sklearn.py** Python 指令檔。

    ![選擇指令碼](media\azure-stack-solution-machine-learning\image41.png)

    程式碼隨即在 Workbench 內的新文字編輯器索引標籤中開啟。

    > [!Note]  
    > 因為此範例專案會經常更新，所顯示的程式碼可能不會與上述程式碼完全相同。

    ![開啟檔案](media\azure-stack-solution-machine-learning\image42.png)

1.  檢查 Python 指令碼，以熟悉程式碼撰寫樣式。

    **iris_sklearn.py** 指令碼會執行下列工作：  

    -  載入名為 **iris.dprep** 的預設資料準備套件，以建立 [pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)。

    -   新增隨機功能，讓問題更難解決。 隨機性是必要的，因為鳶尾花是能以接近 100% 的精確度輕鬆地分類的小型資料集。

    -  使用 scikit-learn 機器學習程式庫來建置羅吉斯迴歸模型。 根據預設，此程式庫隨附 Azure Machine Learning Workbench。

    -  將模型序列化，方法為在 `outputs` 資料夾的檔案中使用 [pickle](https://docs.python.org/3/library/pickle.html) 程式庫。

    -  載入序列化模型，然後將它還原序列化回到記憶體。

    -  使用已還原序列化的模型來對新記錄進行預測。

    -  繪製兩個圖表、混淆矩陣和多級接收者作業特性 (ROC) 曲線，方法為使用 [matplotlib](https://matplotlib.org/) 程式庫，然後將它們儲存在輸出資料夾中。 如果不明顯，請在環境中安裝此程式庫。

    -  自動在執行歷程記錄中繪製正規化比率和模型精確度。 系統會使用整個 `run_logger` 物件來記錄正規化速率並將精確度制定到記錄內。

### <a name="run-irissklearnpy-in-the-local-environment"></a>在本機環境中執行 iris_sklearn.py

1.  啟動 Azure Machine Learning 命令列介面 (CLI)：

    1.  啟動 Azure Machine Learning Workbench。

    2.  從 [Workbench] 功能表，選取 [檔案]> [開啟命令提示字元]。

    Azure Machine Learning 命令列介面 (CLI) 視窗會從 Windows 上的專案資料夾 `C:\Temp\\myIris\` 開始。 這個專案與在本教學課程的第 1 部分中建立的專案相同。

    > [!Important]  
    > 使用此 CLI 視窗來完成下一個步驟。

1.  在 CLI 視窗中，安裝 Python 繪圖程式庫 **matplotlib** (如果還未安裝)。

    **Iris_sklearn.py** 指令碼具有兩個 Python 套件的相依性：**scikit-learn** 和 **matplotlib**。 **scikit-learn** 套件是由 Azure Machine Learning Workbench 安裝，方便您使用。 儘管如此，可能還是需要安裝 **matplotlib**。

    如果未安裝 **matplotlib** 就繼續進行，本教學課程中的程式碼仍可順利執行。 不過，程式碼就不會產生混淆矩陣輸出和歷程記錄視覺效果中所示的多類別 ROC 曲線圖。

    ```CLI
    pip install matplotlib
    python -m pip install --upgrade pip
    ```

    此安裝大約需要一分鐘的時間。

1.  返回 Workbench 應用程式。

2.  尋找名為 **iris_sklearn.py** 的索引標籤。

    ![尋找含指令碼的索引標籤](media\azure-stack-solution-machine-learning\image43.png)

1.  在該索引標籤的工具列中，選取 **local** 作為執行環境，以及選取 iris_sklearn.pyas 作為要執行的指令碼。 可能已選取這些項目。

    ![替代文字](media\azure-stack-solution-machine-learning\image44.png)

1.  移至工具列的右邊，然後在 [引數] 欄位中輸入 0.01。

    這個值對應於羅吉斯迴歸模型的正則化速率。

    ![本機和指令碼選擇](media\azure-stack-solution-machine-learning\image45.png)

1.  選取 [執行] 按鈕。 立即會排定作業。 作業會列在 Workbench 視窗右邊的 [作業] 窗格。

    ![本機和指令碼選擇](media\azure-stack-solution-machine-learning\image46.png)

    在幾分鐘之後，作業的狀態會從 [提交中] 轉換為 [執行中]，最後變成 [已完成]。

1.  選取 [作業] 窗格的作業狀態文字中的 [已完成]。

    ![執行 sklearn](media\azure-stack-solution-machine-learning\image47.png)

    快顯視窗隨即會開啟，並顯示執行的標準輸出 (stdout) 文字。 若要關閉 stdout 文字，請選取快顯視窗上右上方的 [關閉] (**x**) 按鈕。

    ![標準輸出](media\azure-stack-solution-machine-learning\image48.png)

1.  在 [作業] 窗格的相同作業狀態中，選取 [已完成] 狀態和開始時間正上方的藍色文字 **iris_sklearn.py \[n\] * *(n* 是執行編號*)。 [執行屬性] 視窗隨即開啟，並顯示下列該特定執行的資訊：

    -  [執行屬性] 資訊

    -  **輸出**

    -  **計量**

    -  **視覺效果**，若有的話

    -  **記錄檔**

    執行完成時，快顯視窗會顯示下列結果：

    > [!Note]  
    > 因為本教學課程稍早已對訓練集引入了一些隨機化項目，確切的結果可能與如下所示的有所不同。

    ```Python  
        Python version: 3.5.2 |Continuum Analytics, Inc.| (default, Jul  5 2016, 11:41:13) [MSC v.1900 64 bit (AMD64)]

        Iris dataset shape: (150, 5)
        Regularization rate is 0.01
        LogisticRegression(C=100.0, class_weight=None, dual=False, fit_intercept=True,
            intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
            penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
            verbose=0, warm_start=False)
        Accuracy is 0.6792452830188679

        ==========================================
        Serialize and deserialize using the outputs folder.

        Export the model to model.pkl
        Import the model from model.pkl
        New sample: [[3.0, 3.6, 1.3, 0.25]]
        Predicted class is ['Iris-setosa']
        Plotting confusion matrix...
        Confusion matrix in text:
        [[50  0  0]
        [ 1 37 12]
        [ 0  4 46]]
        Confusion matrix plotted.
        Plotting ROC curve....

        ROC curve plotted.
        Confusion matrix and ROC curve plotted. See them in Run History details pane.

    ```

1.  關閉 [執行屬性] 索引標籤，然後返回 **iris_sklearn.py** 索引標籤。

1.  針對其他執行重複進行。

在範圍從 `0.001` 至 `10` 的 [引數] 欄位中輸入一系列的值。 選取 [執行] 來執行程式碼數次。 每次變更的引數值會在程式碼中傳送至羅吉斯迴歸模型，每次產生不同的結果。

### <a name="review-the-run-history-in-detail"></a>檢閱詳細的執行歷程記錄

在 Azure Machine Learning Workbench 中，每個指令碼執行都會以執行歷程記錄的形式擷取。 開啟 [執行] 檢視來檢視特定指令碼的執行歷程記錄。

1.  若要開啟 [執行] 的清單，請選取左邊工具列上的 [執行] 按鈕 (時鐘圖示)。 然後選取 **iris_sklearn.py** 以顯示 iris_sklearn.py 的 [執行儀表板]。

    ![執行檢視](media\azure-stack-solution-machine-learning\image49.png)

1.  [執行儀表板] 索引標籤隨即開啟。

    檢閱跨多個執行擷取的統計資料。 索引標籤頂端中的圖形轉譯。每次執行都有連續的數字，且執行詳細資料會列在畫面底部的資料表中。

    ![執行儀表板](media\azure-stack-solution-machine-learning\image50.png)

1.  篩選資料表，然後選取任何圖表來檢視每個執行的狀態、持續時間、精確度，以及正規化速率。

2.  選取 [執行] 資料表中兩個或更多執行旁邊的核取方塊。 選取 [比較] 按鈕來開啟詳細的比較窗格。 檢閱並排比較。

3.  若要返回 [執行儀表板]，選取 [比較] 窗格左上方的 [執行清單] 返回按鈕。

    ![返回執行清單](media\azure-stack-solution-machine-learning\image51.png)

1.  選取個別執行以查看執行詳細資料檢視。 請注意，所選取執行的統計資料會列在 [執行屬性] 區段中。 寫入到輸出資料夾的檔案會列於 [輸出] 區段，並從該處下載檔案。

    ![執行詳細資料](media\azure-stack-solution-machine-learning\image52.png)

兩個繪圖，混淆矩陣和多類別 ROC 曲線，會在**視覺效果**區段中轉譯。 所有記錄檔也都可在 **Logs** 區段找到。

### <a name="run-scripts-in-the-azure-machine-learning-ml-workbench-cli-window"></a>在 Azure Machine Learning (ML) Workbench CLI 視窗中執行指令碼

1.  啟動 Azure Machine Learning 命令列介面 (CLI)：

    1.  啟動 Azure Machine Learning Workbench。

    2.  從 [Workbench] 功能表，選取 [檔案] > [開啟命令提示字元]。

    CLI 命令提示字元會從 Windows 上的專案資料夾 `C:\\Temp\\myIris` 開始。 這是在本教學課程的第 1 部分中建立的專案。

    > [!Important]  
    > 使用此 CLI 視窗來完成下一個步驟。

1.  在 CLI 視窗中，登入 Azure。 [進一步了解 az 登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)。

    如果您已登入，請略過此步驟。

1.  在命令提示字元中，輸入：

    ```CLI
        az login
    ```

    此命令會傳回要在瀏覽器中的 [https://aka.ms/devicelogin](https://aka.ms/devicelogin) 使用的程式碼。

1.  在瀏覽器中移至 [https://aka.ms/devicelogin](https://aka.ms/devicelogin)。 輸入您從 CLI 所收到的程式碼。

    對 Azure 資源進行驗證時，Workbench 應用程式和 CLI 會使用獨立的認證快取。 在快取的權杖過期前，您都不需要再次驗證。

1.  如果組織有多個 Azure 訂用帳戶 (企業環境)，請設定要使用的訂用帳戶。 尋找訂用帳戶、使用訂用帳戶識別碼設定它，然後加以測試。

1.  使用以下命令列出每個已存取的 Azure 訂用帳戶：

    ```CLI
        az account list -o table 
    ```

    **az account list** 命令會傳回可供登入使用的訂用帳戶。 如果有多個訂用帳戶，請找出所使用訂用帳戶的訂用帳戶識別碼值。

1.  設定要當作預設帳戶的 Azure 訂用帳戶：

    ```CLI
        az account set -s <the-subscription-id
    ```

    其中 <the-subscription-id> 是所使用訂用帳戶的識別碼值。 請勿包含中括弧。

1.  要求目前訂用帳戶的詳細資料，以確認新的訂用帳戶設定。

    ```CLI
        az account show
    ```

1.  在 CLI 視窗中，提交 **iris_sklearn.py** 指令碼當作實驗。

    iris_sklearn.py 執行會針對本機計算內容執行。

1.  在 Windows 上：

    ```CLI
        az ml experiment submit -c local .\\iris_sklearn.py
    ```

1.  在 macOS 上：

    ```CLI
        az ml experiment submit -c local iris_sklearn.py
    ```

    輸出應該會類似： 

    ```
        RunId: myIris_1521077190506

    Executing user inputs .....
    ===========================

    Python version: 3.5.2 |Continuum Analytics, Inc.| (default, Jul  2 2016, 17:52:12) 
    [GCC 4.2.1 Compatible Apple LLVM 4.2 (clang-425.0.28)]

    Iris dataset shape: (150, 5)
    Regularization rate is 0.01
    LogisticRegression(C=100.0, class_weight=None, dual=False, fit_intercept=True,
            intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
            penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
            verbose=0, warm_start=False)
    Accuracy is 0.6792452830188679

    ==========================================
    Serialize and deserialize using the outputs folder.

    Export the model to model.pkl
    Import the model from model.pkl
    New sample: [[3.0, 3.6, 1.3, 0.25]]
    Predicted class is ['Iris-setosa']
    Plotting confusion matrix...
    Confusion matrix in text:
    [[50  0  0]
    [ 1 37 12]
    [ 0  4 46]]
    Confusion matrix plotted.
    Plotting ROC curve....
    ROC curve plotted.
    Confusion matrix and ROC curve plotted. See them in Run History details page.

    Execution Details
    =================
    RunId: myIris_1521077190506

    ```

6.  檢閱輸出。 全都應該與 Workbench 執行指令碼時的輸出和結果相同。

7.  返回 Workbench，然後：

    選取左邊窗格上的資料夾圖示以列出專案檔案。  開啟名為 **run.py** 的 Python 指令碼。 此指令碼適合用於透過各種正則化速率的迴圈。 

    ![返回執行清單](media\azure-stack-solution-machine-learning\image53.png)

1.  以此比率執行實驗多次。

    此指令碼會以 `10.0` 的正規化速率 (非常大的數字) 來啟動 ` aniris_sklearn.pyjob`。 接著，指令碼會在下列回合中將比率縮減為一半，並依此類推，直到比率不小於 `0.005` 為止。 此指令碼包含下列程式碼：

    ![返回執行清單](media\azure-stack-solution-machine-learning\image54.png)

1.  從命令列執行 **run.py** 指令碼，如下所示：

    ```CLI
        python run.py
    ```
此命令會使用不同的正規化比率多次提交 `iris_sklearn.py`

當 `run.py` 完成時，Workbench 的執行歷程記錄清單檢視中會顯示不同計量的圖表。

### <a name="run-scripts-in-an-ubuntu-based-data-science-virtual-machine-dsvm-on-azure"></a>在 Azure 上的 Ubuntu 架構資料科學虛擬機器 (DSVM) 中執行指令碼

若要在遠端 Linux 電腦上的 Docker 容器中執行指令碼，需要有 SSH 存取權 (使用者名稱和密碼) 才能執行該遠端電腦。 此外，該電腦必須安裝並執行 Docker 引擎。

1.  編輯在 aml_config 下產生的 <your DSVM Name>.runconfig 檔案，並將架構從預設值 PySpark 變更為 Python：

    ```yaml  
    Framework: Python
    ```
1.  發出如同先前在 CLI 視窗中執行的相同命令，但這次使用 *<DSVM>* 目標在遠端 Docker 容器中執行 iris_sklearn.py：(將 <DSVM> 替換成資料科學虛擬機器名稱，但不含括弧)。

    ```CLI
        az ml experiment submit -c <DSVM> iris_sklearn.py
    ```

命令會如同在 docker-python 環境中執行，不同之處在於是在遠端 Linux VM 上執行。 CLI 視窗會顯示相同的輸出資訊。

### <a name="download-the-model-pickle-file"></a>下載模型序列化檔案

在教學課程的前一個部分中，**iris_sklearn.py** 指令碼是在 Machine Learning Workbench 本機執行。 此動作會使用受歡迎的 Python 物件序列化套件 [pickle](https://docs.python.org/3/library/pickle.html) 來序列化羅吉斯迴歸模型。

1.  開啟 Machine Learning Workbench 應用程式。 然後開啟在本教學課程系列的先前部分中所建立的 **myIris** 專案。

2.  開啟專案後，請選取左窗格上的 [檔案] 按鈕 (資料夾圖示)，在專案資料夾中開啟檔案清單。

3.  選取 **iris_sklearn.py** 檔案。 Python 程式碼隨即會在 Workbench 內新的文字編輯器索引標籤中開啟。

4.  檢閱 **iris_sklearn.py** 檔案來查看序列化檔案產生的位置。 選擇 Ctrl+F 來開啟 [尋找] 對話方塊，然後在 Python 程式碼中尋找字組 **pickle**。

這個程式碼片段會示範 pickle 輸出檔案產生的方式。 磁碟上的輸出 pickle 檔案名為 **model.pkl**。

    ```Python
        print("Export the model to model.pkl")
        f = open('./outputs/model.pkl', 'wb')
        pickle.dump(clf1, f)
        f.close()

    ```

1.  在先前執行的輸出檔案中，找出模型序列化檔案。

    **iris_sklearn.py** 指令碼執行時，模型檔案會使用 **model.pkl** 名稱寫入至 **outputs** 資料夾。 此資料夾位於選擇來執行指令碼的執行環境中，而不是在本機專案資料夾中。 

    1. 若要找出檔案，請選取左窗格上的 [執行] 按鈕 (時鐘圖示)，以開啟 [所有執行] 清單。  

    2. [所有執行] 索引標籤隨即開啟。 在執行的資料表中，選取最近執行的其中一個，其目標為 **local** 且指令碼名稱為 **iris_sklearn.py**。  

    3. [執行屬性] 窗格隨即開啟。 在窗格的右上角區段中，注意 [輸出] 區段。 d. 若要下載 pickle 檔案，請選取 **model.pkl** 檔案旁的核取方塊，然後選取 [下載]。 請將檔案儲存到專案資料夾的根目錄。 後續步驟中需要檔案。  

    ![下載 pickle 檔案](media\azure-stack-solution-machine-learning\image55.png)

### <a name="get-scoring-script-and-schema-files"></a>取得評分指令碼和結構描述檔案

若要部署 Web 服務以及模型檔案，則需要評分指令碼。 (選擇性) 需要 Web 服務輸入資料的結構描述。 評分指令碼會從目前資料夾載入 **model.pkl** 檔案，並使用它來產生新的預測。

1.  開啟 Machine Learning Workbench 應用程式。 然後開啟在本教學課程系列的前一個部分中所建立的 **myIris** 專案。

2.  開啟專案後，請選取左窗格上的 [檔案] 按鈕 (資料夾圖示)，在專案資料夾中開啟檔案清單。

3.  選取 **score_iris.py** 檔案。 Python 指令碼隨即開啟。 這個檔案會用作評分檔案。

    ![評分檔案](media\azure-stack-solution-machine-learning\image56.png)

1.  若要取得結構描述檔案，請執行指令碼。 選取命令列中的 **local** 環境和 **score_iris.py** 指令碼，然後選取 [執行]。

    此指令碼會在 **Outputs** 區段中建立 JSON 檔案，它會擷取模型所需的輸入資料結構描述。

1.  請注意 [專案儀表板] 窗格右邊的 [作業] 窗格。 等候最新的 score_iris.py 作業顯示綠色 [已完成] 狀態。 然後選取最新作業執行的超連結 **score_iris.py**，以查看執行詳細資料。

2.  在 [執行屬性] 頁面上，於 [輸出] 區段中，選取新建立的 **service_schema.json** 檔案。 選取檔案名稱旁邊的核取方塊，然後選取 [下載]。 將檔案儲存到專案根資料夾。

3.  返回上一個索引標籤和 **score_iris.py** 指令碼。 使用資料收集可從 Web 服務擷取模型輸入和預測。 後面是資料收集的相關步驟。

4.  請檢閱檔案頂端的程式碼，該程式碼會匯入 **ModelDataCollector** 類別。 其包含模型資料收集功能：

    ```Python  
        from azureml.datacollector import ModelDataCollector
    ```

1.  在具現化 **ModelDataCollector** 的 **init()** 函式中檢閱下列幾行程式碼：

    ```Python  
        global inputs_dc, prediction_dc
        inputs_dc = ModelDataCollector('model.pkl',identifier="inputs")
        prediction_dc = ModelDataCollector('model.pkl', identifier="prediction")
    ```

1.  在收集輸入和預測資料的 **run(input_df)** 函式中檢閱下列幾行程式碼：

    ```Python  
        inputs_dc.collect(input_df)
        prediction_dc.collect(pred)
    ```

現在，讓環境準備好將模型運算化。

## <a name="step-5-deploy-and-use-azure-container-registry"></a>步驟 5：部署和使用 Azure Container Registry

部署和使用 Azure Container Registry。

使用 **az acr create** 命令來建立 Azure Container Registry。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 資源群組是相同的。

    ```CLI
        az acr create --resource-group <ResourceGroup> --name  <acrName> --sku Basic
    ```

### <a name="container-registry-login"></a>Container Registry 登入

使用 **az acr login** 命令登入 ACR 執行個體。 在建立容器登錄時，為它提供唯一名稱。

    ```CLI
        az acr login --name <acrName>
    ```

此命令在完成之後會傳回「登入成功」訊息。

### <a name="prepare-to-operationalize-locally-for-development-and-testing-the-service"></a>準備在本機運作以進行服務的開發和測試

使用「本機模式」部署，於本機電腦上的 Docker 容器中執行，以及進行開發與測試。

Docker 引擎必須在本機執行，才能完成將模型作業化的下列步驟。 在每個命令的結尾處使用 `-h` 旗標，以顯示對應的說明訊息。

    > [!Note]  
    > If Docker engine is not locally available, proceed by creating a cluster in Azure for deployment and keep the cluster for re-use, or delete it after the tutorial to avoid ongoing charges.

    > [!Note]  
    > Web services deployed locally do not appear in Azure Portal's list of services. They will be running in Docker on the local machine.

1.  開啟命令列介面 (CLI)。 在 Machine Learning Workbench 應用程式的 [檔案] 功能表上，選取 [開啟命令提示字元]。

    命令提示字元會在目前的專案資料夾位置 **c:\\temp\\myIris** 中開啟。

1.  請確定已在訂用帳戶中註冊 Azure 資源提供者 **Microsoft.ContainerRegistry**。 在步驟 3 中建立環境之前，先登錄這個資源提供者。 使用下列命令來查看是否已登錄：

    ```CLI
        az provider list --query "\[\].{Provider:namespace, Status:registrationState}" --out table
    ```

    檢視此輸出：

    ```CLI
        Provider                                    Status 
        --------                                    --------
        Microsoft.Authorization                     Registered 
        Microsoft.ContainerRegistry                 Registered 
        microsoft.insights                          Registered 
        Microsoft.MachineLearningExperimentation    Registered 
    ```

    如果未登錄 **Microsoft.ContainerRegistry**，請使用下列命令：

    ```CLI
    az provider register --namespace Microsoft.ContainerRegistry
    ```

    註冊可能需要幾分鐘的時間。 使用上一個 **az provider list** 命令或下列命令來檢查註冊狀態：

    ```CLI
    az provider show -n Microsoft.ContainerRegistry
    ```

    第三行輸出顯示 **"registrationState": "Registering"**。 請稍候片刻並重複 **show** 命令，直到輸出顯示 **"registrationState": "Registered** 為止。

1.  建立環境。 每個環境執行此步驟一次。

    下列設定命令需要訂用帳戶的參與者存取權。 需要資源群組部署目的地的參與者存取權。 使用 -g 旗標，將資源群組名稱指定為設定命令的一部分。

    ```CLI
    az ml env setup -n <new deployment environment name> --location <e.g. eastus2> -g <existing resource group name>
    ```

    請遵循螢幕上的指示來佈建用於儲存 Docker 映像的儲存體帳戶、列出 Docker 映像的 Azure Container Registry，以及收集遙測的 Azure Application Insights 帳戶。 **如果使用 -c 參數，此命令會額外建立容器服務叢集**。

    叢集名稱可用來識別環境。 位置應該與建立自 Azure 入口網站的模型管理帳戶位置相同。

    為了確定該環境已成功設定，請使用下列命令來檢查狀態：

    ```CLI
    az ml env show -n <deployment environment name> -g <existing resource group name>
    ```

    請先確定「佈建狀態」具有「成功」值 (如下所示)，再於步驟 5 中設定環境：

    ![佈建狀態](media\azure-stack-solution-machine-learning\image57.png)

1.  設定環境。

    設定完成之後，使用下列命令來設定作業化環境所需的環境變數。 使用先前在步驟 3 使用的相同環境名稱。 使用當設定程序完成時，在命令視窗中輸出的相同資源群組名稱。

    ```CLI
        az ml env set -n <deployment environment name> -g <existing resource group name>
    ```

1.  若要確認您已針對本機 Web 服務部署正確設定運作化環境，請輸入下列命令：

    ```CLI
    az ml env show
    ```

    現在，建立即時 Web 服務。

    > [!Note]  
    > 針對後續 Web 服務部署重複使用模型管理帳戶和環境。 不需要為每個 Web 服務建立這些項目。 帳戶或環境可以有多個與其相關聯的 Web 服務。

### <a name="create-a-real-time-web-service-by-using-separate-commands"></a>使用個別命令建立即時 Web 服務

您也可以個別執行這些步驟，作為以上所示 **az ml service create realtime** 命令的替代方式。

首先，註冊模型。 然後產生資訊清單、建立 Docker 映像及建立 web 服務。 此逐步解說方法可在每個步驟提供更大的彈性。 此外，或許能重複使用先前步驟所產生的實體。

1. 藉由提供序列化檔案名稱來登錄模型。

    ```CLI
    az ml model register --model model.pkl --name model.pkl
    ```

    此命令會產生模型識別碼。

2.  建立資訊清單。

    若要建立資訊清單，請使用下列命令，並提供來自前一個步驟的模型識別碼輸出： 
    
    ```CLI
    az ml manifest create --manifest-name <new manifest name> -f score_iris.py -r python -i <model ID> -s service_schema.json
    ```

    此命令會產生資訊清單識別碼。

3.  建立 Docker 映像。

    若要建立 Docker 映像，請使用下列命令，並提供來自前一個步驟的資訊清單識別碼值輸出。 也可以透過 `-c` 參數來使用 Conda 相依性。 
    
    ```CLI
    az ml image create -n irisimage --manifest-id <manifest ID> -c aml_config\conda_dependencies.yml
    ```
    
    此命令會產生 Docker 映像識別碼。

2.  建立服務。

    若要建立服務，請使用下列命令，並提供來自前一個步驟的模型識別碼輸出： 
    
    ```CLI
    az ml service create realtime --image-id <image ID> -n irisapp --collect-model-data true
    ```
    
    此命令會產生 Web 服務識別碼。
    
    接下來，執行 Web 服務。

## <a name="step-6-deploy-a-kubernetes-cluster-to-azure-stack"></a>步驟 6：將 Kubernetes 叢集部署至 Azure Stack

將 Kubernetes 叢集部署至 Azure Stack。

可在 Azure Stack 上使用 Azure Container Services (ACS) 引擎所產生的 Azure Resource Manager 範本來安裝 Kubernetes。 [*Kubernetes*](https://kubernetes.io/) 是一種開放原始碼系統，可進行自動化部署、調整和管理容器中的應用程式。 [*容器*](https://www.docker.com/what-container)包含類似於 VM 的映像中。 不同於 VM，容器映像只會包含其執行應用程式所需的資源，例如程式碼、執行程式碼的執行階段、特定程式庫和設定等。

使用 Kubernetes 可執行下列作業：

 -  開發可大幅調整、可升級，並且可在短短數秒內完成部署的應用程式。

 -  簡化應用程式設計，並透過不同的 Helm 應用程式改善其可靠性。 [Helm](https://github.com/kubernetes/helm) 是開放原始碼的封裝工具，有助於安裝和管理 Kubernetes 應用程式的生命週期。

 -  透過調整與升級功能輕鬆監視和診斷應用程式的健康情況。

> [!Note]  
> 安裝叢集需要約一小時的時間，請做出適當規劃。

### <a name="prerequisites-for-kubernetes-cluster-deployment"></a>Kubernetes 叢集部署的必要條件

若要開始使用，請確認權限和 Azure Stack 整備程度：

1.  驗證**建立 Kubernetes 叢集 (預覽)** 項目可於 Azure Stack Marketplace 取得。 如果沒有此項目，請與 Azure Stack 操作員合作，將此項目新增至 Azure Stack 環境。

2.  確認您可以在 Azure Active Directory (Azure AD) 租用戶中建立應用程式。 需要權限才能部署 Kubernetes。

    如需檢查權限的指示，請參閱[*檢查 Azure Active Directory 權限*](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)。

3.  產生在 Azure Stack 上用來登入 Linux VM 的 SSH 公開和私密金鑰組。 建立叢集時需要公開金鑰。 如需指示，請參閱[產生 SSH 的驗證金鑰](#generate-an-authenticatio-key-for-ssh)。

4.  確認 Azure Stack 租用戶入口網站中的訂用帳戶有效，且公用 IP 位址數量足供新增應用程式。

    叢集無法部署至 Azure Stack 的**系統管理員**訂用帳戶。 使用**使用者**訂用帳戶。

###  <a name="generate-an-authentication-key-for-ssh"></a>產生 SSH 的驗證金鑰

從 Linux 工作階段的 Windows 子系統內，使用下列命令來產生驗證金鑰： 

1. 輸入：

    ```Bash  
    ssh-keygen -t rsa
    ```
    
2. 出現提示時選取 `id_rsa`。 
3. 出現提示時建立複雜密碼。 請務必記下此密碼以供稍後使用。 
    輸出類似如下範例： 
    
    ```Bash  
    Your identification has been saved in `id_rsa`.
    Your public key has been saved in `id_rsa.pub`. 
    The key fingerprint is: SHA256:lUtUUjzaqWqGeolEPKeBmsnrhcNGM9Dn2OxYatt05SE  <user>@<machine-name>
    The key's randomart image is:  
    ```
    ![替代文字](media\azure-stack-solution-machine-learning\image58.png)

4. 產生金鑰後，使用下列命令貼上金鑰資訊： 
    ```Bash
    cat id_rsa.pub
    ```
    輸出類似如下範例： 
    ```Bash
    ssh-rsa  AAAAB3NzaC1yc2EAAAADAQABAAABAQDHaWrAktR3BlQ356T8Qvv8O2Q/U/hsXQwS9xJbuduuc9lkJwddzgmNCyM9PooZWYtGVXyHU6SC3YH1XkwqGtKhtPb03d24hmhX1euAaqIqHHSp4WgUS5s1gNsp37L8QCSGNCnF31FWavI8bnjOjccUkMowKl8iyGN++5UyQgNuynEVUbFJjrntoDL66HUu88xDxTcVB7rqDr2QKFwYJkg4HSoHYpezJd7W8kcmv33eh0xs8nlDA7Dzu7zXpyVc54bH9XtPR6EUXaQa+UqKaNlQNiJqEs+1X/zNuK9V6NLVNiO0h3jCHI3K8o4VnZyRKHCQProasiiPLUUJ6SF/RTLN  <user>@<machine-name>
    ```
    
5. 將產生的金鑰複製到 [SSH 公用金鑰] 欄位。

### <a name="create-a-service-principal-in-azure-ad"></a>在 Azure AD 中建立服務主體

1.  登入全域 [*Azure 入口網站*](http://www.poartal.azure.com/)。

2.  使用與 Azure Stack 執行個體相關聯的 Azure AD 租用戶來登入。

3.  建立 Azure AD 應用程式。

    1. 選取 [Azure Active Directory] > [+ 應用程式註冊]> [新應用程式註冊]。
    2. 輸入應用程式的 [名稱]。 
    3. 選取 [Web 應用程式/API]。 
    4. 針對 [登入 URL] 輸入 `http://localhost`。 
    5. 選取 [建立] 。

1.  記下 [應用程式識別碼]。 建立叢集時需要此識別碼，而且系統會將此識別碼參考為 [服務主體用戶端識別碼]。

2.  選取 [設定] > [金鑰]。

    1. 輸入 [說明]。 
    2. 針對 [到期日]，選取 [永久有效]。 
    3. 選取 [ **儲存**]。 請記下金鑰字串。 建立叢集時需要此金鑰字串，而且系統會將此金鑰字串參考為 [服務主體用戶端密碼]。

### <a name="give-the-service-principal-access"></a>指定服務主體存取權

對服務主體提供訂用帳戶的存取權，讓服務主體能夠建立資源。

1.  登入[系統管理入口網站](https://adminportal.local.azurestack.external/)。

2.  選取 [更多服務] > [使用者訂用帳戶] > [+ 新增]。

3.  選取所建立的訂用帳戶。

4.  選取 [存取控制 (IAM)] > 選取 [+ 新增]。

5.  選取 [擁有者] 角色。

6.  選取為服務主體建立的應用程式名稱。 如有需要，請在搜尋方塊中輸入名稱。

7.  選取 [ **儲存**]。

8.  開啟 [Azure Stack 入口網站](https://portal.local.azurestack.external)。

9.  選取 [+ 新增] > [計算] > [Kubernetes 叢集]。 選取 [建立] 。

    ![部署解決方案範本](media\azure-stack-solution-machine-learning\image59.png)

10. 在 [建立 Kubernetes 叢集] 中選取 [基本資料]。

    ![Deploy Solution Template](media\azure-stack-solution-machine-learning\image60.png)

11. 輸入 **Linux VM 系統管理員使用者名稱**。 這是屬於 Kubernetes 叢集和 DVM 的 Linux 虛擬機器的使用者名稱。

12. 輸入用來對建立為 Kubernetes 叢集和 DVM 一部分的所有 Linux 機器進行授權的 **SSH 公開金鑰**。

13. 輸入在區域中具有唯一性的**主要設定檔 DNS 前置詞**。 這必須是區域中唯一的名稱，例如 `ask8s-12345`。 根據最佳做法，請試著選擇與資源群組名稱相同的名稱。

    > [!Note]  
    > 對於每個叢集，都應使用全新且唯一的主要設定檔 DNS 前置詞。

14. 輸入**代理程式集區設定檔計數**。 此計數包含叢集中的代理程式數目。 可能的值介於 1 到 4 之間。

15. 輸入**服務主體 ClientId**。Kubernetes Azure 雲端提供者會使用此識別碼。

16. 輸入建立服務主體應用程式時所建立的**服務主體用戶端密碼**。

17. 輸入 **Kubernetes Azure 雲端提供者版本**。 這是 Kubernetes Azure 提供者的版本。 Azure Stack 會為每個 Azure Stack 版本發行一個自訂 Kubernetes 組建。

18. 選取**訂用帳戶**識別碼。

19. 輸入新資源群組的名稱，或選取現有的資源群組。 資源名稱必須是小寫的英數字元。

20. 選取資源群組的 [位置]。 這是選擇用來安裝 Azure Stack 的區域。

### <a name="specify-the-azure-stack-settings"></a>指定 Azure Stack 設定

1.  選取 [Azure Stack 戳記設定]。

    ![部署解決方案範本](media\azure-stack-solution-machine-learning\image61.png)

2.  輸入 [租用戶 Azure Resource Manager 端點]。 這是要連線以建立 Kubernetes 叢集之資源群組的 Azure Resource Manager 端點。 需要向 Azure Stack 操作員取得整合系統的端點。 如需 Azure Stack 開發套件 (ASDK)，請使用 `https://management.local.azurestack.external`。

3.  輸入租用戶的**租用戶識別碼**。 請參閱[*取得租用戶識別碼*](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)來尋找租用戶識別碼值。

### <a name="install-kubectl-on-windows-powershell-environment"></a>在 Windows PowerShell 環境中安裝 kubectl

從 WSL 環境中執行下列命令，以在 WSL 環境中安裝 kubectl。

```PowerShell  
Install-script -name install-kubectl -scope CurrentUser -force
Install-kubectl.ps1 -downloadlocation “C:\Users\<Current User>\Documents\Kube
```

### <a name="install-kubectl-on-the-windows-subsystem-for-linux-environment"></a>在 Linux 環境的 Windows 子系統上安裝 kubectl

從 WSL 環境中執行下列命令，以在 WSL 環境中安裝 kubectl。

```Bash  
    apt-get update && apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y kubectl
```

### <a name="configure-kubectl"></a>設定 kubectl

若要讓 kubectl 尋找及存取 Kubernetes 叢集，則需要 kubeconfig 檔案。 使用 kube-up.sh 或部署 Minikube 叢集來建立叢集時，會自動建立此檔案。 如需建立叢集的詳細資訊，請參閱[*快速入門指南*](https://kubernetes.io/docs/setup/)。 若要存取其他使用者所建立的叢集，請參閱[*共用叢集存取文件*](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)。 根據預設，kubectl 組態位於 **~.kube/config**。

### <a name="check-the-kubectl-configuration"></a>檢查 kubectl 組態

藉由取得叢集狀態來確認 kubectl 是否已正確設定：

```Bash  
kubectl cluster-info
```

若有顯示 url 回應，則 kubectl 已正確設定而可存取叢集。

如果顯示下列訊息，則 kubectl 未正確設定或無法連線至 Kubernetes 叢集：

```Bash  
The connection to the server <server-name:port> was refused -  did you specify the right host or port?
```

例如，在本機膝上型電腦上執行 Kubernetes 叢集時，可能需要工具 (minikube 等) 來重新執行上述命令。

如果 kubectl 叢集資訊傳回 url 回應，但叢集仍無法存取，請使用下列命令來確認組態是否適當：

```Bash  
> kubectl cluster-info dump
```

### <a name="enable-shell-autocompletion"></a>啟用殼層自動完成

kubectl 包含自動完成支援，可快速且輕鬆地啟用殼層。

完成指令碼本身會由 kubectl 產生，並可從設定檔來存取。

### <a name="connect-to-the-kubernetes-cluster"></a>連線至 Kubernetes 叢集

若要連線至叢集，則需要 Kubernetes 命令列用戶端 (**kubectl**)。 您可以在 [Azure Container Service 文件](https://docs.microsoft.com/azure/container-service/dcos-swarm/)中找到連線和管理叢集的相關指示。

任何 SSH 用戶端皆可用於連線 Kubernetes 叢集。 但建議使用適用於 Linux 的 Windows 子系統 (WSL)。 連線到 Kubernetes 叢集的機器會在為叢集所建立的資源群組內，而且會以 vmd-edge-ml-stack 前置詞作為開頭。

```Bash  
ssh <user>@vmd-edge-ml-stack.<FQDN of Kubernetes Machine in  Azure Stack>
```

連線到 Kubernetes 機器後，請執行下列機器以取得 Kubernetes 組態檔。

```Bash  
sudo find / -name \*kubeconfig\* -type f
```

輸出會看起來類似這樣：

```Bash  
/var/lib/waagent/custom-script/download/0/acs-engine/_output/edgemlstack/kubeconfig/kubeconfig.<regionname>.json
```

複製此檔案路徑資訊，然後執行下列命令，貼入從上面複製而來的 kubeconfig 檔案路徑：

```Bash  
sudo cat  /var/lib/waagent/custom-script/download/0/acs-engine/_output/edgemlstack/kubeconfig/kubeconfig.<regionname>.json
```

輸出會是大型文字區塊，也就是未經處理的 JSON 內容。 複製此輸出文字，然後將此程式碼貼到 Visual Studio 檔案，並儲存為 JSON 檔案。 這會產生儲存在本機的 kubeconfig.json 檔案。 (儲存至 /mnt/c/users/<current user>/documents/Kube 目錄成為 kubeconfig.json)

### <a name="configure-the-kubernetes-cluster"></a>設定 Kubernetes 叢集

取得本機 JSON 檔案後，在新的 WSL 工作階段中使用下列命令來設定叢集。

```Bash  
    cd /mnt/c/users/<current user>/documents/Kube
    kubectl proxy
    kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    kubectl proxy
    set KUBECONFIG=”/mnt/c/users/<current user>/documents/Kube/kubeconfig.json”
    kubectl.exe config view
```

系統會定義 Kubernetes 組態集 (請參閱以下輸出)。

![替代文字](media\azure-stack-solution-machine-learning\image62.png)

啟動本機 Proxy 服務：

```Bash  
kubectl proxy
```

瀏覽至 kubernetes 叢集 UI，位址如下：`https://localhost:8001`。

![替代文字](media\azure-stack-solution-machine-learning\image63.png)

現在，您會有位置可供部署容器，並有位於雲端但可在內部部署環境看到的容器。

![替代文字](media\azure-stack-solution-machine-learning\image64.png)

使用所選擇的程式碼編輯器，自訂 **iris_deployment.yaml** 檔案 (位於 /mnt/c/users/<current user>/documents/Kube 目錄)，讓 **webservicename** 和容器的**映像**與**名稱**符合部署。

![替代文字](media\azure-stack-solution-machine-learning\image65.png)

將容器連接埠設定為 **5001。**

![替代文字](media\azure-stack-solution-machine-learning\image66.png)

然後建立 **imagePullSecret**：

### <a name="create-a-secret-in-the-cluster-that-holds-the-authorization-token"></a>在保有授權權杖的叢集中建立祕密

Kubernetes 叢集會使用 **docker-registry** 類型的祕密向容器登錄進行驗證，以提取私有映像。

建立此祕密，並將它命名為 **azuremlcr**:

```Bash  
kubectl create secret docker-registry azuremlcr --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password= "<your-pword>" --docker-email=<your-email>
```

尋找：

- **<your-registry-server>** 是 Azure Container Registry 的**登入伺服器**。
- **<your-name>** 是 Azure Container Registry 的**使用者名稱**。
- **<your-pword>** 是 Azure Container Registry 的**密碼**。 請務必以引號括住密碼。
- **<your-email>** 是具有容器 R/W 存取權的使用者。
- 在 **Azure Container** **Registry** 的 [存取金鑰] 下可找到這項資訊。
- Docker 認證現已在叢集中設定為祕密，名稱為 **azuremlcr**。

儲存 **iris_deployment.yaml** 檔案 (位於 /mnt/c/users/<current user>/documents/Kube 目錄)。

### <a name="create-the-kubernetes-container"></a>建立 Kubernetes 容器

```Bash  
kubectl.exe create -f /mnt/c/users/<current  user>/documents/Kube/iris_deployment.yaml
```

    ![Alt text](media\azure-stack-solution-machine-learning\image67.png)

檢查部署的狀態：

```Bash  
Kubectl get deployments
```

    ![Alt text](media\azure-stack-solution-machine-learning\image68.png)

此部署可能需要一些時間。

### <a name="configure-visual-studio-team-services-to-deploy-automatically"></a>設定 Visual Studio Team Services 以進行自動部署

#### <a name="create-a-team-project"></a>建立 Team 專案

1.  確定[專案集合系統管理員群組成員資格。](https://docs.microsoft.com/vsts/organizations/security/set-project-collection-level-permissions?view=vsts) 若要建立 Team 專案，[建立新專案] 權限必須設為 [允許]。

2.  從 [專案] 頁面選取 [新增專案]。

    ![替代文字](media\azure-stack-solution-machine-learning\image69.png)

1.  將專案命名為 **HybridMLIris**。

2.  將其初始原始檔控制類型選為 [Git]

3.  選取程序，並選取 [建立]。

    ![替代文字](media\azure-stack-solution-machine-learning\image70.png)

### <a name="import-some-code--create-repository"></a>匯入某些程式碼 建立存放庫

需要 YAML 程式碼的 Git 存放庫。

#### <a name="add-user-to-the-git-repo"></a>將使用者新增至 GIT 存放庫

1.  從預設專案儀表板中，選取 [產生 Git 認證]。

    ![替代文字](media\azure-stack-solution-machine-learning\image71.png)

1.  必要時輸入密碼，並儲存 Git 認證。

    ![替代文字](media\azure-stack-solution-machine-learning\image72.png)

1.  選取 [初始化] 按鈕並建立**讀我檔案**，將存放庫初始化。

    ![替代文字](media\azure-stack-solution-machine-learning\image73.png)

#### <a name="clone-the-git-repository-locally-and-upload-the-code"></a>在本機複製 Git 存放庫，並上傳程式碼。 

1.  在電腦上建立目錄 `c:\\users\\<User>\\source\\repos\\hybridMLIris`，並複製存放庫

    ```Bash  
    sudo mkdir /mnt/c/users/<User>/source sudo mkdir /mnt/c/users/<User>/source/repos sudo mkdir /mnt/c/users/<User>/source/repos/hybridMLIris cd /mnt/c/users/<User>/source/repos/hybridMLIris sudo git clone  https://<yourvstssite>.visualstudio.com/HybridMLIris/_git/HybridMLIris
    ```

    ![替代文字](media\azure-stack-solution-machine-learning\image74.png)

1.  瀏覽至新複製的存放庫：

    ```Bash  
    ls
    cd ./HybridMLIris
    ```
    
    ![替代文字](media\azure-stack-solution-machine-learning\image75.png)

1.  將 **iris_deployment.yaml** 檔案複製到存放庫。

    ```Bash  
    cp /mnt/c/users/<User>/documents/Kube/iris_deployment.yaml  /mnt/c/users/<User>/source/repos/HybridMLIris/HybridMLIris/iris_deployment.yaml
    ``` 

1.  認可 GIT 中的變更

    ```Bash  
    git add . git commit -m Added Deployment YAML git push
    ```

    ![替代文字](media\azure-stack-solution-machine-learning\image76.png)

### <a name="prepare-the-private-build-and-release-agent-for-vsts-integration"></a>準備 VSTS 整合所需的私用組建和發行代理程式

#### <a name="prerequisites"></a>必要條件

VSTS 會使用服務主體對 Azure Resource Manager 進行驗證。 VSTS 必須處於「參與者」狀態，才能在 Azure Stack 訂用帳戶中佈建資源。**以下是啟用這類驗證時所須設定的概要步驟：**

1.  應建立服務主體，或可使用現有的服務主體。

2.  必須建立服務主體的驗證金鑰。

3.  必須透過角色型存取控制來驗證 Azure Stack 訂用帳戶，讓 SPN 成為參與者角色的一部分。

4.  必須使用 Azure Stack 端點和 SPN 資訊在 VSTS 中建立新的服務定義。

#### <a name="service-principal-creation"></a>服務主體建立

請參閱[服務主體建立](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)指示以建立服務主體，並選擇 [Web 應用程式/API] 作為 [應用程式類型]。

**存取金鑰建立**

服務主體需要金鑰才能進行驗證，請依照本節中的步驟產生金鑰。

1.  在 Azure Active Directory 中，從 [應用程式註冊] 選取應用程式。

    ![選取應用程式](media\azure-stack-solution-machine-learning\image77.png)

1.  記下 [應用程式識別碼] 的值。**在 VSTS 中設定服務端點時，將會使用該值。**

    ![替代文字](media\azure-stack-solution-machine-learning\image78.png)

1.  若要產生驗證金鑰，請選取 [設定]。

    ![選取設定](media\azure-stack-solution-machine-learning\image79.png)

1.  選取 [金鑰]。

    ![選取金鑰](media\azure-stack-solution-machine-learning\image80.png)

1.  提供金鑰的描述和金鑰的持續時間。 完成時，選取 [儲存]。

    ![儲存金鑰](media\azure-stack-solution-machine-learning\image81.png)

儲存金鑰之後會顯示金鑰的值。 複製此值以供稍後使用。 必須有**金鑰值**和應用程式識別碼，才能以應用程式的形式登入。 將金鑰值儲存在應用程式可擷取的地方。

![替代文字](media\azure-stack-solution-machine-learning\image82.png)

#### <a name="get-tenant-id"></a>取得租用戶識別碼

在設定服務端點的過程中，VSTS 需要將**租用戶識別碼**對應至 Azure Stack 戳記已部署到的 AAD 目錄。 請依照本節中的步驟收集租用戶識別碼。

1.  選取 **Azure Active Directory**。

    ![選取 Azure Active Directory](media\azure-stack-solution-machine-learning\image83.png)

1.  若要取得租用戶識別碼，請選取 Azure AD 租用戶的 [屬性]。

    ![選取 Azure AD 屬性](media\azure-stack-solution-machine-learning\image84.png)

1.  複製 [目錄識別碼]。 這個值是租用戶識別碼。

    ![租用戶識別碼](media\azure-stack-solution-machine-learning\image85.png)

授與在 Azure Stack 訂用帳戶中部署資源的服務主體權限

若要存取訂用帳戶中的資源，請將應用程式指派給角色。 決定哪個角色代表應用程式的正確權限。 若要深入了解可用的角色，請參閱 [RBAC：內建角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)。

針對訂用帳戶、資源群組或資源的層級設定範圍。 較低的範圍層級會繼承較高層級的權限。 例如，為資源群組的讀取者角色新增應用程式，可讓該角色讀取資源群組及其所包含的任何資源。

1.  瀏覽至所要指派應用程式的範圍層級。 例如，若要在訂用帳戶範圍指派角色，請選取 [訂用帳戶]。

    ![替代文字](media\azure-stack-solution-machine-learning\image86.jpeg)

1.  選取**訂用帳戶** (資源群組或資源) 來指派應用程式。

    ![選取要指派的訂用帳戶](media\azure-stack-solution-machine-learning\image87.png)

1.  選取 [存取控制 (IAM)]。

    ![選取存取](media\azure-stack-solution-machine-learning\image88.png)

1.  選取 [新增] 。

    ![選取 [新增]](media\azure-stack-solution-machine-learning\image89.png)

1.  選取要指派應用程式的角色。 下圖顯示**擁有者**角色。

    ![替代文字](media\azure-stack-solution-machine-learning\image90.png)

1.  根據預設，Azure Active Directory 應用程式不會顯示在可用選項中。 若要尋找應用程式，請在搜尋欄位中**提供名稱**，並加以選取。

    ![替代文字](media\azure-stack-solution-machine-learning\image91.png)

1.  選取 [儲存] 以完成角色指派。 應用程式會顯示在指派給該範圍角色的使用者清單中。

### <a name="role-based-access-control"></a>角色型存取控制

Azure 角色型存取控制 (RBAC) 可以對 Azure 和 Azure Stack 進行更細緻的存取權管理。 使用 RBAC，只會授與使用者執行其作業所需的存取權。 如需角色型存取控制的詳細資訊，請參閱[管理對 Azure 訂用帳戶資源的存取](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal?toc=%252fazure%252factive-directory%252ftoc.json)。

**VSTS 代理程式集區**

代理程式會組織到**代理程式集區**中，而不必個別管理每個代理程式。 代理程式集區會定義該集區中所有代理程式的共用界限。 在 VSTS 中，代理程式集區的範圍限於 VSTS 帳戶中，因此可在 Team 專案間共用。 如需關於如何建立 VSTS 代理程式集區的詳細資訊和教學課程，請參閱[建立代理程式集區和佇列](https://docs.microsoft.com/vsts/build-release/concepts/agents/pools-queues?view=vsts)。

**新增 Azure Stack 的個人存取權杖 (PAT)**

 -  登入 VSTS 帳戶，並選取**帳戶設定檔**名稱。

 -  選取 [管理安全性] 以存取權杖建立頁面。

![替代文字](media\azure-stack-solution-machine-learning\image92.png)

![替代文字](media\azure-stack-solution-machine-learning\image93.jpeg)

![替代文字](media\azure-stack-solution-machine-learning\image94.jpeg)

> [!Note]  
> 取得權杖資訊。 離開此畫面後將不會再次顯示該資訊。

1.  複製**權杖**。

    ![替代文字](media\azure-stack-solution-machine-learning\image95.png)

#### <a name="install-the-vsts-build-agent-on-the-azure-stack-hosted-build-server"></a>在裝載 Azure Stack 的組建伺服器上安裝 VSTS 組建代理程式

1.  連線至 Azure Stack 主機上所部署的**組建伺服器**。

    > [!Note]  
    > 請確定可從公用網際網路存取 Azure Stack 裝載的組建伺服器。 如果不行，請與 Azure Stack 操作員合作來取得存取權。

    ```Bash  
    ssh <user>@<buildserver.publicip>
    ```

2.  將 Ubuntu 組建伺服器升級至最新版本 (18.04)：

    ```Bash  
    sudo su
    apt-get update
    apt-get upgrade
    apt-get dist-upgrade
    apt-get autoremove
    do-release-upgrade -d
    ```

    > [!Note]  
    > 這項作業需要一些時間。

2.  安裝組建伺服器的必要應用程式。 從 Ubuntu 架構組件伺服器的 Bash 殼層，執行下列命令：

    ```Bash  
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
    sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
    wget -q https://packages.microsoft.com/config/ubuntu/18.04/prod.list 
    sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
    sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
    sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list
    sudo apt-get install apt-transport-https
    sudo apt-get update
    sudo apt-get install liblttng-ust0 libcurl3 libssl1.0.0 curl lsb-release libkrb5-3 zlib1g libicu60 aspnetcore-runtime-2.1 dotnet-sdk-2.1
    wget https://github.com/PowerShell/PowerShell/releases/download/v6.1.0-preview.3/powershell-preview_6.1.0-preview.3-1.ubuntu.18.04_amd64.deb
    sudo dpkg -i powershell-preview_6.1.0-preview.3-1.ubuntu.18.04_amd64.deb
    sudo apt-get install -f
    AZ_REPO=$(lsb_release -cs)
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
        sudo tee /etc/apt/sources.list.d/azure-cli.list
    curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add –
    sudo apt-get update && sudo apt-get install azure-cli
    ```

2.  下載組建代理程式並使用**個人存取權杖 (PAT)** 以服務的形式加以部署，然後以 VM 管理員帳戶執行。

    ![替代文字](media\azure-stack-solution-machine-learning\image96.png)

    ```Bash  
        cd \home\<user>
        sudo mkdir myagent && cd myagent
        wget https://vstsagentpackage.azureedge.net/agent/2.134.2/vsts-agent-linux-x64-2.134.2.tar.gz
        sudo tar zxvf vsts-agent-linux-x64-2.134.2.tar.gz
    ```

2.  移至解壓縮的組建代理程式資料夾。 執行下列程式碼。

    ```Bash  
        cd ..
        sudo chmod -R 777 myagent
        cd myagent
        ./config.sh
    ```

    ![替代文字](media\azure-stack-solution-machine-learning\image97.png)

2.  在 **./config.sh** 完成後，請執行下列程式碼，以在伺服器開機時啟用服務，並啟動服務：

    ```Bash  
    sudo ./svc.sh install
    sudo ./svc.sh start
    ```

代理程式現在會顯示在 VSTS 資料夾中。

#### <a name="endpoint-creation-permissions"></a>端點建立權限

使用者可以建立端點，讓 VSTO 組建能夠將 Azure 服務應用程式部署至堆疊。 VSTS 會連線至組建代理程式，然後再連接 Azure Stack。

![替代文字](media\azure-stack-solution-machine-learning\image98.png)

1.  在 [設定] 功能表上，選取 [安全性]。

2.  在左側的 [VSTS 群組] 清單中，選取 [端點建立者]。

    ![替代文字](media\azure-stack-solution-machine-learning\image99.png)

3.  在 [成員] 索引標籤上，選取 [+新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image100.png)

1.  輸入**使用者名稱**，並從清單中選取使用者名稱。

2.  選取 [儲存變更]。

    ![替代文字](media\azure-stack-solution-machine-learning\image101.png)

3.  在左側的 [VSTS 群組] 清單中，選取 [端點管理員]。

4.  在 [成員] 索引標籤上，選取 [+新增]。

5.  輸入**使用者名稱**，然後從清單中選取該使用者。

6.  選取 [儲存變更]。

    ![buchatech](media\azure-stack-solution-machine-learning\image102.jpeg)

    Azure Stack 中的組建代理程式會取得來自 VSTS 的指示，然後傳達與 Azure Stack 進行通訊所需的端點資訊。

    VSTS 與 Azure Stack 的連線現已備妥。

    ![替代文字](media\azure-stack-solution-machine-learning\image103.png)

### <a name="configure-build-and-release-definitions"></a>設定組建和發行定義

連線已建立，接著要以手動方式將所建立的 Azure 端點、AKS 和 Azure Container Registry 對應至組建和發行定義。

#### <a name="create-the-build-definition-for-the-yaml-code"></a>建立 YAML 程式碼的組建定義

1.  選取 [組建和發行] 中樞下的 [組建] 區段，並建立新的定義。

    ![替代文字](media\azure-stack-solution-machine-learning\image104.png)

1.  選取 VSTS Git，然後選取稍早建立的存放庫。

    ![替代文字](media\azure-stack-solution-machine-learning\image105.png)

1.  選取空的管線作為範本

    ![替代文字](media\azure-stack-solution-machine-learning\image106.png)

1.  將組建命名為**複製成品**，並選取代理程式佇列的 Azure Stack 組建伺服器。

    ![替代文字](media\azure-stack-solution-machine-learning\image107.png)

1.  在程序中選取第 1 階段，並將其重新命名為**複製成品**，然後在該階段中**新增工作**：

    ![替代文字](media\azure-stack-solution-machine-learning\image108.png)

1.  從 [公用程式] 清單選取 [發行組建成品]，然後選取 [新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image109.png)

1.  選取 [要發行的路徑]，然後選取 **iris_deployment.yaml** 檔案。

    ![替代文字](media\azure-stack-solution-machine-learning\image110.png)

1.  將成品命名為 **iris_deployment**，然後選取要作為 **Visual Studio Team Services/TFS** 的發行位置。

    ![替代文字](media\azure-stack-solution-machine-learning\image111.png)

1.  選取 [儲存並排入佇列]。

    ![替代文字](media\azure-stack-solution-machine-learning\image112.png)

1.  檢查組建的狀態，方法是選取組建識別碼。

    ![替代文字](media\azure-stack-solution-machine-learning\image113.png)

成功情況類似如下範例：

![替代文字](media\azure-stack-solution-machine-learning\image114.png)

#### <a name="create-the-release-definition-for-the-yaml-code"></a>建立 YAML 程式碼的發行定義

1.  選取 [組建和發行] 中樞下的 [發行] 區段，並建立新的定義

    ![替代文字](media\azure-stack-solution-machine-learning\image115.png)

1.  選取空的管線作為範本。

    ![替代文字](media\azure-stack-solution-machine-learning\image106.png)

1.  將環境命名為 Azure Stack。

    ![替代文字](media\azure-stack-solution-machine-learning\image116.png)

1.  選取 [成品] 和 [+新增] 來新增成品

2.  選取 [組建] 作為來源類型，選取 [HybridMLIris] 作為專案。

3.  然後，選取稍早為來源建立的組建定義。

4.  然後選取 [新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image117.png)

1.  從環境中選取 [Azure Stack]，然後將新的工作新增至 Azure Stack

    ![替代文字](media\azure-stack-solution-machine-learning\image118.png)


1.  在代理程式階段中，將 [代理程式佇列] 設定為 [Azure Stack 裝載的組建伺服器]。

    ![替代文字](media\azure-stack-solution-machine-learning\image119.png)

1.  將新工作新增至這個階段，選取 [部署] 下的 [部署至 Kubernetes] 工作，然後選取 [新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image120.png)


1.  將其命名為 **Kubectl Apply** (預設名稱)，然後選取套用命令。

    ![替代文字](media\azure-stack-solution-machine-learning\image121.png)

    現在，建立新的 Kubernetes 服務連線。

#### <a name="create-kubernetes-service-endpoint"></a>建立 Kubernetes 服務端點

1.  在 [Kubernetes 服務連線] 下，選取 [+ 新增] 按鈕，然後從清單中選取 [Kubernetes]。 您可以使用這個端點連接 **VSTS** 和 **Azure Container Service (AKS)**。

2.  **連線名稱**：提供連線名稱。

3.  **伺服器 URL**：提供容器服務位址，格式為 http://{API 伺服器位址}

4.  **Kubeconfig**：若要取得 Kubeconfig 值，請在以系統管理員權限啟動的命令提示字元中，執行下列 Azure 命令。

    > [!Important]  
    > 使用此 CLI 視窗來完成下一個步驟。

6.  在 CLI 視窗中，登入 Azure。 [進一步了解 az 登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)。

7.  在命令提示字元中，輸入：

    ```CLI
        az login
    ```

10. 此命令會傳回要在瀏覽器中的 <https://aka.ms/devicelogin> 使用的程式碼。

11. 在瀏覽器中移至 <https://aka.ms/devicelogin>。 出現提示時，在瀏覽器中輸入您在 CLI 中收到的程式碼。

    ![Kubernetes 服務端點](media\azure-stack-solution-machine-learning\image122.png)

1.  在命令提示字元中輸入下列命令，以取得 Kubernetes 叢集的存取認證。

### <a name="azure-ml-workbench-cli"></a>Azure ML Workbench CLI

az aks get-credentials resource-group <yourResourceGroup> name <yourazurecontainerservice>

![Kubernetes 服務端點](media\azure-stack-solution-machine-learning\image123.png)

1.  瀏覽至主目錄下的 **.kube** 資料夾 (例如：C:\\Users\\<user>\\Documents\\Kube)

2.  複製 **config** 檔案的內容，並將它貼入 [Kubernetes 連線] 視窗中。 選取 [確定] 按鈕。

    ![Kubernetes 服務端點](media\azure-stack-solution-machine-learning\image124.png)
    

3.  建立並選取 Kubernetes 端點後，請選取 [使用組態檔] 核取方塊以新增組態檔。 然後瀏覽至「連結的成品」中的 iris_deployment.yaml 檔案。

    ![替代文字](media\azure-stack-solution-machine-learning\image125.png)

    ![替代文字](media\azure-stack-solution-machine-learning\image126.png)

4.  儲存發行定義。

#### <a name="check-the-status-of-the-release-definition"></a>檢查發行定義的狀態。 

儲存之後，VSTS 發行定義應該會自動開始發行。

在 WSL 命令提示字元中執行快速命令，然後檢查 Kubernetes UI，來檢查部署的狀態。

```Bash  
kubectl get deployments
```

部署程序進行時，輸出應看起來應類似下列範例。

![替代文字](media\azure-stack-solution-machine-learning\image127.png)

```Bash  
kubectl proxy
```

kubernetes UI 開始執行之後，瀏覽至部署 ([**https://localhost:8001/**](https://localhost:8001/))，然後瀏覽至 [工作負載] -> [複本集]。

![替代文字](media\azure-stack-solution-machine-learning\image128.png)

### <a name="deploy-the-yaml-service"></a>部署 YAML 服務

#### <a name="upload-the-irisserviceyaml-to-the-repository-and-sync-changes"></a>將 iris_service.yaml 上傳至存放庫並同步處理變更

1.  瀏覽至新複製的存放庫：

    ```Bash  
    cd /mnt/c/users/<User>/source/repos/HybridMLIris/HybridMLIris/
    ```

    ![替代文字](media\azure-stack-solution-machine-learning\image75.png)

1.  將 **iris_service.yaml** 檔案複製到存放庫。

    ```Bash  
    cp /mnt/c/users/<User>/documents/Kube/iris_service.yaml  /mnt/c/users/<User>/source/repos/HybridMLIris/HybridMLIris/iris_service.yaml
    ```

1.  認可 GIT 中的變更

    ```Bash  
    git add .
    git commit -m “Added Service YAML” 
    git push
    ```

![替代文字](media\azure-stack-solution-machine-learning\image129.png)

#### <a name="update-the-build-definition-for-the-yaml-code"></a>更新 YAML 程式碼的組建定義

1.  選取 [組建和發行] 中樞下的 [組建] 區段，然後選取稍早建立的定義。

    ![替代文字](media\azure-stack-solution-machine-learning\image130.png)

2.  選取 [編輯] 按鈕來編輯定義。

    ![替代文字](media\azure-stack-solution-machine-learning\image131.png)

3.  在階段中 [新增工作]。 從 [公用程式] 清單選取 [發行組建成品]，然後選取 [新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image108.png)

4.  將其命名為 **Kubectl Apply** (預設名稱)，然後選取套用命令。



#### <a name="update-the-release-definition-for-the-yaml-code"></a>更新 YAML 程式碼的發行定義

1.  選取 [組建和發行] 中樞下的 [發行] 區段，然後選取稍早建立的發行定義。 然後選取 [編輯] 連結。

    ![替代文字](media\azure-stack-solution-machine-learning\image132.png)

1.  選取 [Azure Stack] 環境，然後將新的工作新增至 Azure Stack。

    ![替代文字](media\azure-stack-solution-machine-learning\image133.png)

1.  在這個階段中新增 [新工作]，選取 [部署] 下的 [部署至 Kubernetes] 工作，然後選取 [新增]。

    ![替代文字](media\azure-stack-solution-machine-learning\image134.png)

1.  將其命名為 **Kubectl Apply** (預設名稱)，然後選取套用命令。

    ![替代文字](media\azure-stack-solution-machine-learning\image109.png)

1.  將 Kubernates 服務連線設定為稍早建立的 Azure Stack 連線，然後選取 [使用組態檔] 核取方塊以新增組態檔。 瀏覽至「連結的成品」中的 iris_service.yaml 檔案。

    ![替代文字](media\azure-stack-solution-machine-learning\image135.png)


    ![替代文字](media\azure-stack-solution-machine-learning\image136.png)

1.  儲存發行定義。

#### <a name="check-the-status-of-the-release-definition"></a>檢查發行定義的狀態

儲存之後，VSTS 發行定義應該會自動開始發行。

在 WSL 命令提示字元中執行快速命令，然後檢查 Kubernetes UI，來檢查部署的狀態。

```Bash  
kubectl get deployments
```

部署程序進行時，輸出應看起來應類似下列範例。

![替代文字](media\azure-stack-solution-machine-learning\image127.png)


```Bash  
kubectl proxy
```

kubernetes UI 開始執行之後，瀏覽至部署 ([**https://localhost:8001/**](https://localhost:8001/))，然後瀏覽至 [工作負載] -> [複本集]。

![替代文字](media\azure-stack-solution-machine-learning\image137.png)


### <a name="kubernetes-scoring-and-validation"></a>Kubernetes 評分和驗證

啟動 Kubernetes UI

```Bash  
kubectl proxy
```

瀏覽至 Kubernetes UI，然後移至 [部署] -> [Iris-Deployment] -> [新增複本集] -> [Iris-Deployment-xxxxxxxxx] (其中 xxxxxxxxx 是部署識別碼)。

![替代文字](media\azure-stack-solution-machine-learning\image138.png)

然後瀏覽至 [服務]，然後選取服務的 [外部端點]，以驗證其是否能運作。

![替代文字](media\azure-stack-solution-machine-learning\image139.png)

驗證訊息應該會類似下面所示內容：

![替代文字](media\azure-stack-solution-machine-learning\image140.png)

#### <a name="create-azure-stack-scoring-function-app-in-the-azure-stack-portal"></a>在 Azure Stack 入口網站中建立 Azure Stack 評分函式應用程式

必須有函式應用程式才能裝載每個函式的執行。 函式應用程式可讓函式群組為邏輯單位，以方便您管理、部署和共用資源。

1.  從 Azure Stack 使用者入口網站中，選取左上角找到的 [+ 新增] 按鈕，然後選取 [Web + 行動] >[函式應用程式]。

    ![替代文字](media\azure-stack-solution-machine-learning\image141.png)

1.  將函式命名為 **data-functions**，並將它放在與剩餘機器學習服務內容相同的資源群組中。 讓工具針對耗用量自動建立新的 App Service 方案，並使用稍早為應用程式儲存體建立的儲存體帳戶。

    ![定義新的函式應用程式設定](media\azure-stack-solution-machine-learning\image142.png)

1.  選取 [建立] 以佈建並部署函式應用程式。

2.  選取入口網站右上角的 [通知] 圖示，查看是否有**部署成功**訊息。

    ![定義新的函式應用程式設定](media\azure-stack-solution-machine-learning\image143.png)

1.  選取 [前往資源]，以檢視新的函式應用程式。

    ![替代文字](media\azure-stack-solution-machine-learning\image144.png)

1.  選取 [函式]，然後選取 [+ 新增函式] 按鈕，即可建立新的函式。

    ![替代文字](media\azure-stack-solution-machine-learning\image145.png)

1.  選取 HTTP 觸發程序

    ![替代文字](media\azure-stack-solution-machine-learning\image146.png)

1.  選取 [C\#] 作為語言並將函式命名為 **clean-score-data**，然後將授權層級設定為 [匿名]。

    ![替代文字](media\azure-stack-solution-machine-learning\image147.png)

1.  在函式中複製貼上 clean-score-data 的程式碼範例內容。

    ![替代文字](media\azure-stack-solution-machine-learning\image148.png)

#### <a name="use-postman-to-validate-functions"></a>使用 Postman 來驗證函式

若要確定您已正確設定 Kbernetes 和 Azure Functions，您可以使用免費的應用程式 Postman 來測試及驗證結構描述和函式。 若要開始此程序，您需要先從 Kubernetes 執行個體中收集部分資訊。

1.  瀏覽至 Kubernetes UI，然後移至 [部署] -> [Iris-Deployment] -> [新增複本集] -> [Iris-Deployment-xxxxxxxxx] (其中 xxxxxxxxx 是部署識別碼)

    ![替代文字](media\azure-stack-solution-machine-learning\image138.png)

1.  然後瀏覽至 [服務] 並複製 [外部端點]。

    ![替代文字](media\azure-stack-solution-machine-learning\image149.png)

1.  如有需要，下載並安裝[此處](https://www.getpostman.com/apps)的 Postman 應用程式。

2.  登入 Postman 應用程式，並關閉 [新增檔案] 對話方塊。

    ![替代文字](media\azure-stack-solution-machine-learning\image150.png)

1.  從 Postman 應用程式內，選取 [POST]。

    ![替代文字](media\azure-stack-solution-machine-learning\image151.png)

1.  將 [外部端點] URL 貼到 Postman 應用程式的 [要求 URL] 底下，並在 URL 結尾新增 \\score，如下所示。

    ![替代文字](media\azure-stack-solution-machine-learning\image152.png)

1.  選取 [本文] 索引標籤，然後將資料類型選為 [未經處理]，然後選取 [JSON]。

    ![替代文字](media\azure-stack-solution-machine-learning\image153.png)

1.  從網頁瀏覽器瀏覽至 [外部端點]。 將下列內容新增至 URL **/swagger.json**。這會導致服務 Swagger 檔案用來測試設定。

    ![替代文字](media\azure-stack-solution-machine-learning\image154.png)

1.  複製 **Swagger.JSON** 檔案中所列的範例。

2.  在 Postman 應用程式中，將範例貼到「張貼」的本文，然後選取 [傳送]。 它應該會傳回類似下列所示的值。

    ![替代文字](media\azure-stack-solution-machine-learning\image155.png)

## <a name="step-7-create-an-azure-stack-storage-account-and-storage-queue"></a>步驟 7：建立 Azure Stack 儲存體帳戶和儲存體佇列

建立資料的 Azure Stack 儲存體帳戶和儲存體佇列。

1.  登入 Azure Stack 使用者入口網站。 (每個 Azure Stack 各有唯一的入口網站 URL)

2.  在 Azure Stack 使用者入口網站中，展開左側功能表以開啟服務的功能表，然後選擇 [所有服務]。 向下捲動至 [儲存體]，然後選擇 [儲存體帳戶]。 在 [儲存體帳戶] 視窗中，選擇 [新增]。

3.  輸入儲存體帳戶的名稱。

4.  選取儲存體帳戶的複寫選項︰[LRS]。

5.  指定新的資源群組，或選取現有的資源群組。

6.  選取 [本機] 作為儲存體帳戶位置。

7.  選取 [建立] 以建立儲存體帳戶。

    ![替代文字](media\azure-stack-solution-machine-learning\image156.png)

1.  選擇最近建立的儲存體帳戶。

2.  選取 [佇列]。

    ![替代文字](media\azure-stack-solution-machine-learning\image157.png)

1.  選取 [+ 佇列] 並命名佇列，然後選取 [確定]。

    ![替代文字](media\azure-stack-solution-machine-learning\image158.png)

1.  取得儲存體佇列的 [連接字串]，並加以複製。

    ![替代文字](media\azure-stack-solution-machine-learning\image159.png)

1.  瀏覽至 Azure 函式應用程式，然後選取 [應用程式設定]。

    ![替代文字](media\azure-stack-solution-machine-learning\image160.png)

1.  從函式應用程式的 [應用程式設定] 內，向下捲動至 [應用程式設定]，然後選取 [+ 新增設定]。

    ![替代文字](media\azure-stack-solution-machine-learning\image161.png)

1.  在 [名稱] 欄位中輸入儲存體帳戶的名稱，並於結尾新增 _STORAGE

這可讓應用程式了解這是儲存體帳戶端點。

1.  然後在 [值] 欄位中貼上連接字串。

    ![替代文字](media\azure-stack-solution-machine-learning\image162.png)

1.  向上捲動至 [應用程式設定] 頂端，然後選取 [儲存]。

    ![替代文字](media\azure-stack-solution-machine-learning\image163.png)

### <a name="update-the-scoring-function-to-use-storage-queue"></a>更新評分函式以使用儲存體佇列

1.  在函式上選取 [整合]，並取消選取 [GET] 選項。

2.  選取 [ **儲存**]。

3.  然後從 [輸出] 選取 [+ 新增輸出]。

    ![替代文字](media\azure-stack-solution-machine-learning\image164.png)

1.  然後選取 [Azure 佇列儲存體]，並選擇 [選取]。

    ![替代文字](media\azure-stack-solution-machine-learning\image165.png)

1.  將 [佇列名稱] 更新為稍早建立的儲存體佇列，然後將 [儲存體帳戶連線] 設定為稍早建立的儲存體帳戶連線，並選取 [儲存]。

    ![替代文字](media\azure-stack-solution-machine-learning\image166.png)

## <a name="step-8-create-a-function-to-handle-clean-data"></a>步驟 8：建立函式以處理乾淨資料

建立新的 Azure Stack 函式，以將乾淨資料從 Azure Stack 移至 Azure。

1.  選取 [函式]，然後選取 [+ 新增函式] 按鈕，即可建立新的函式。

    ![替代文字](media\azure-stack-solution-machine-learning\image167.png)

1.  選取 [計時器觸發程序]。

    ![替代文字](media\azure-stack-solution-machine-learning\image168.png)

1.  選取 [C\#] 作為語言，並將函式命名為 **upload-to-azure**，然後將排程設定為 **0 0 \*/1 \* \* \***，這在 CRON 標記法中是指一小時一次。

    ![替代文字](media\azure-stack-solution-machine-learning\image169.png)

### <a name="get-the-connection-string-to-the-azure-hosted-storage-account"></a>取得 Azure 裝載的儲存體帳戶的連接字串

1.  瀏覽至 <https://portal.azure.com>，然後選取稍早建立的 [Azure 儲存體帳戶]。

2.  選取 [存取金鑰]，然後複製儲存體帳戶的 [連接字串]。

    ![替代文字](media\azure-stack-solution-machine-learning\image170.png)

### <a name="update-the-upload-to-azure-function-to-use-the-azure-hosted-storage"></a>更新 upload-to-azure 函式來使用 Azure 裝載的儲存體

1.  瀏覽至 Azure 函式應用程式，然後選取 [應用程式設定]。

    ![替代文字](media\azure-stack-solution-machine-learning\image171.png)

1.  從函式應用程式的 [應用程式設定] 內，向下捲動至 [應用程式設定]，然後選取 [+ 新增設定]。

    ![替代文字](media\azure-stack-solution-machine-learning\image172.png)

1.  在 [名稱] 欄位中輸入儲存體帳戶的名稱，並於結尾新增 _STORAGE

這可讓應用程式了解這是儲存體帳戶端點。

1.  然後在 [值] 欄位中貼上 Azure 裝載的儲存體帳戶連接字串。

    ![替代文字](media\azure-stack-solution-machine-learning\image173.png)

1.  向上捲動至 [應用程式設定] 頂端，然後選取 [儲存]。

    ![替代文字](media\azure-stack-solution-machine-learning\image174.png)

1.  瀏覽回 **upload-to-azure** 函式。

2.  在函式上選取 [整合]。

3.  然後從 [輸出] 選取 [+ 新增輸出]。

    ![替代文字](media\azure-stack-solution-machine-learning\image175.png)

1.  然後選取 [Azure Blob 儲存體]，並選擇 [選取]。

    ![替代文字](media\azure-stack-solution-machine-learning\image176.png)

1.  將 [路徑] 更新為稍早建立的儲存體容器 (格式為 **uploadeddata/{rand-guid}.txt**)，然後將 [儲存體帳戶連線] 設定為稍早建立的 Azure 儲存體帳戶連線，並選取 [儲存]。

    ![替代文字](media\azure-stack-solution-machine-learning\image177.png)

1.  在函式中複製貼上 **upload-to-azure** 的程式碼範例內容。

2.  視需要修改為指向儲存體帳戶連接字串。

3.  儲存並執行程式碼。

    ![替代文字](media\azure-stack-solution-machine-learning\image178.png)

1.  檢查 Azure 裝載的儲存體帳戶，以查看已從 Azure 剖析到雲端的資料：成功情況類似如下範例。

    ![替代文字](media\azure-stack-solution-machine-learning\image179.png)

資料已由 Azure Stack 裝載的 Kubernetes 機器學習服務來處理為敏感性資料，並已透過 Azure Stack 裝載的函式應用程式，從內部部署 Azure Stack 上傳至 Azure 公用雲端，可供在邊緣/中斷連線的案例中暫存要上傳的資料。

## <a name="next-steps"></a>後續步驟

 - 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。