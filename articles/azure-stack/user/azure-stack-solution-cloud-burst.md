---
title: 使用 Azure 建立跨雲端縮放解決方案 | Microsoft Docs
description: 了解如何使用 Azure 建立跨雲端縮放解決方案。
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
ms.openlocfilehash: 1f3dcc7da3e91d3805cae805422778978d0187c5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971326"
---
# <a name="tutorial-create-cross-cloud-scaling-solutions-with-azure"></a>教學課程：使用 Azure 建立跨雲端縮放解決方案

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

了解如何建立可提供手動觸發程序的跨雲端解決方案，以透過流量管理員使用自動縮放功能，從 Azure Stack 託管的 Web 應用程式切換到 Azure 託管的 Web 應用程式，藉此確保雲端公用程式在處理負載時具有彈性且可調整。

透過此模式，您的租用戶可能還無法在公用雲端中執行應用程式。 不過，若讓企業在內部部署環境中維持用來處理應用程式需求高峰的容量，似乎不切實際。 您的租用戶可以透過其內部部署解決方案來使用公用雲端的靈活性。

在本教學課程中，您將建置範例環境，用以：

> [!div class="checklist"]
> - 建立多節點 Web 應用程式。
> - 設定和管理持續部署 (CD) 程序。
> - 將 Web 應用程式發佈至 Azure Stack。
> - 建立發行。
> - 了解如何監視並追蹤您的部署。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力引進您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。

## <a name="prerequisites"></a>必要條件

-   Azure 訂用帳戶。 如有需要，請在開始之前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- Azure Stack 整合系統，或部署 Azure Stack 開發套件。
    - 您可以在[安裝 Azure Stack 開發套件](/articles/azure-stack/asdk/asdk-install)中找到如何安裝 Azure Stack 的指示。
    - [https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1](https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1) 此安裝可能需要幾個小時才能完成。

-   將 [App Service](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack。

-   在 Azure Stack 環境中建立[方案/供應項目](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview)。

-   在 Azure Stack 環境中建立[租用戶訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)。

-   建立租用戶訂用帳戶中建立 Web 應用程式。 記下新的 Web 應用程式 URL，以供後續使用。

-   在租用戶訂用帳戶中部署 VSTS 虛擬機器。

-   需要具有 .NET 3.5 的 Windows Server 2016 VM。 此 VM 將會建置在 Azure Stack 上的租用戶訂用帳戶中，作為私用組建代理程式。

-   您可以在 Azure Stack Marketplace 中取得[使用 SQL 2017 VM 的 Windows Server 2016 映像](https://docs.microsoft.com/azure/azure-stack/azure-stack-add-vm-image#add-a-vm-image-through-the-portal)。 如果無法取得此映像，可與 Azure Stack 操作員合作，以確實地將此映像新增至環境中。

## <a name="issues-and-considerations"></a>問題和考量

### <a name="scalability-considerations"></a>延展性考量

跨雲端縮放的關鍵要素是能視需求立即在公用和內部部署雲端基礎結構之間提供縮放功能，證明服務會如需求規定一樣保持一致且可靠。

### <a name="availability-considerations"></a>可用性考量

確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。

### <a name="manageability-considerations"></a>管理性考量

跨雲端解決方案保證能在環境間提供無縫式管理和熟悉介面。 建議您使用 PowerShell 進行跨平台管理。

## <a name="cross-cloud-scaling"></a>跨雲端縮放

### <a name="obtain-a-custom-domain-and-configure-dns"></a>取得自訂網域並設定 DNS

更新網域的 DNS 區域檔案。 Azure AD 會驗證自訂網域名稱的擁有權。 對於 Azure 中的 Azure/Office 365/外部 DNS 記錄使用 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal)，或在[不同的 DNS 註冊機構](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)新增 DNS 項目。

1.  向公用註冊機構註冊自訂網域。

2.  登入網域的網域名稱註冊機構。 已核准的系統管理員可能需要進行 DNS 更新。 

3.  透過新增 Azure AD 提供的 DNS 項目來更新網域的 DNS 區域檔案。 (DNS 項目不會影響郵件路由或 Web 託管行為。) 

### <a name="create-a-default-multi-node-web-app-in-azure-stack"></a>在 Azure Stack 中建立預設的多節點 Web 應用程式

設定混合式持續整合和持續部署 (CI/CD)，以將 Web 應用程式部署至 Azure 和 Azure Stack，並自動將變更推送至這兩個雲端。

> [!Note]  
> 需要適當映像聯合執行的 Azure Stack (Windows Server 和 SQL) 及 App Service 部署。 檢閱 App Service 文件中適用於 Azure Stack 操作員的「[開始使用 Azure Stack 上的 App Service 之前](/articles/azure-stack/azure-stack-app-service-before-you-get-started)」區段。

### <a name="add-code-to-visual-studio-team-services-project"></a>將程式碼新增至 Visual Studio Team Services 專案

1. 使用在 Visual Studio Team Services (VSTS) 上具有專案建立權限的帳戶登入 VSTS~~。~~

    混合式 CI/CD 可同時套用至應用程式程式碼和基礎結構程式碼。 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/)進行私用與託管的雲端開發。

    ![替代文字](media\azure-stack-solution-cloud-burst\image1.JPG)

2. 建立並開啟預設 Web 應用程式以**複製存放庫**。

    ![替代文字](media\azure-stack-solution-cloud-burst\image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署

1.  編輯 **WebApplication.csproj** 檔案。 選取 [Runtimeidentifier] 並新增 **win10-x64**。 (請參閱[獨立式部署](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)文件。) 

    ![替代文字](media\azure-stack-solution-cloud-burst\image3.png)

2.  使用 Team Explorer 將程式碼簽入 VSTS 中。

3.  確認應用程式程式碼已簽入 Visual Studio Team Services 中。

## <a name="create-the-build-definition"></a>建立組建定義

1. 登入 VSTS 以確認能夠建立組建定義。

2. 新增 **-r win10-x64** 程式碼。 這是觸發 .Net Core 的獨立部署時所需的程式碼。

    ![替代文字](media\azure-stack-solution-cloud-burst\image4.png)

3. 執行組建。 [獨立的部署組建](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)程序將會發佈可在 Azure 和 Azure Stack 上執行的成品。

## <a name="use-an-azure-hosted-agent"></a>使用 Azure 託管的代理程式

在 VSTS 中使用託管的代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動執行維護和升級，以支援持續而不間斷的開發、測試和部署。

### <a name="manage-and-configure-the-cd-process"></a>管理和設定 CD 程序

Visual Studio Team Services 和 Team Foundation Server (TFS) 提供具有高度設定和管理能力的管線，可用於對多個環境的發行 (例如開發、暫存、QA 和生產環境)；包括特定階段需要核准。

## <a name="create-release-definition"></a>建立發行定義

![替代文字](media\azure-stack-solution-cloud-burst\image5.png)

1.  在 VSO 的 [建置及發行] 頁面的 [發行] 索引標籤下選取**加號**，以新增發行。

    ![替代文字](media\azure-stack-solution-cloud-burst\image6.png)

2. 套用 Azure App Service 部署範本。

    ![替代文字](media\azure-stack-solution-cloud-burst\image7.png)

3. 在 [新增成品] 下，為 Azure 雲端建置應用程式新增成品。

    ![替代文字](media\azure-stack-solution-cloud-burst\image8.png)

4. 在 [管線] 索引標籤下選取環境的 [階段]、[工作] 連結，並設定 Azure 雲端環境值。

    ![替代文字](media\azure-stack-solution-cloud-burst\image9.png)

5. 設定 [環境名稱]，並選取 Azure 雲端端點的 Azure**訂用帳戶**。

    ![替代文字](media\azure-stack-solution-cloud-burst\image10.png)

6. 在 [環境名稱] 下設定所需的 **Azure App Service 名稱**。

    ![替代文字](media\azure-stack-solution-cloud-burst\image11.png)

7. 在 Azure 雲端託管環境的代理程式佇列下輸入 **Hosted VS2017**。

    ![替代文字](media\azure-stack-solution-cloud-burst\image12.png)

8. 在 [部署 Azure App Service] 功能表中，為環境選取有效的**套件或資料夾**。 對**資料夾位置**選取 [確定]。

    ![替代文字](media\azure-stack-solution-cloud-burst\image13.png)

    ![替代文字](media\azure-stack-solution-cloud-burst\image14.png)

9. 儲存所有變更，並返回**發行管線**。

    ![替代文字](media\azure-stack-solution-cloud-burst\image15.png)

10. 選取 Azure Stack 應用程式的組建，以新增成品。

    ![替代文字](media\azure-stack-solution-cloud-burst\image16.png)

11. 再新增一個套用 Azure App Service 部署的環境。

    ![替代文字](media\azure-stack-solution-cloud-burst\image17.png)

12. 將新環境命名為 Azure Stack。

    ![替代文字](media\azure-stack-solution-cloud-burst\image18.png)

13. 在 [工作] 索引標籤下找出 Azure Stack 環境。

    ![替代文字](media\azure-stack-solution-cloud-burst\image19.png)

14. 選取 Azure Stack 端點的訂用帳戶。

    ![替代文字](media\azure-stack-solution-cloud-burst\image20.png)

15. 將 Azure Stack Web 應用程式名稱設定為 App Service 名稱。

    ![替代文字](media\azure-stack-solution-cloud-burst\image21.png)

16. 選取 [Azure Stack 代理程式]。

    ![替代文字](media\azure-stack-solution-cloud-burst\image22.png)

17. 在 [部署 Azure App Service] 區段下，為環境選取有效的**套件或資料夾**。 對資料夾位置選取 [確定]。

    ![替代文字](media\azure-stack-solution-cloud-burst\image23.png)

    ![替代文字](media\azure-stack-solution-cloud-burst\image24.png)

18. 在 [變數] 索引標籤下新增名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數，並將其值設定為 **true**，範圍設定為 Azure Stack。

    ![替代文字](media\azure-stack-solution-cloud-burst\image25.png)

19. 選取兩個成品中的 [持續部署觸發程序] 圖示，並啟用**持續**部署觸發程序。

    ![替代文字](media\azure-stack-solution-cloud-burst\image26.png)

20. 選取 Azure Stack 環境中的 [預先部署] 條件圖示，並將觸發程序設定為 [發行之後]。

21. 儲存所有變更。

> [!Note]  
> 工作的某些設定可能已在從範本建立發行定義時自動定義為[環境變數](https://docs.microsoft.com/vsts/build-release/concepts/definitions/release/variables?view=vsts#custom-variables)。 這些設定無法在工作設定中修改；而是必須選取父環境項目才能編輯這些設定

## <a name="publish-to-azure-stack-via-visual-studio"></a>透過 Visual Studio 發佈至 Azure Stack

藉由建立端點，Visual Studio Online (VSTO) 組建可以將 Azure 服務應用程式部署到 Azure Stack。 VSTS 會連線至組建代理程式，後者再連線至 Azure Stack。

1.  登入 VSTO 並瀏覽至 [應用程式設定] 頁面。

2.  在 [設定] 上，選取 [安全性]。

3.  在 [VSTS 群組] 中，選取 [端點建立者]。

4.  在 [成員] 索引標籤上，選取 [新增]。

5.  在 [新增使用者和群組] 中，輸入使用者名稱，然後從使用者清單中選取該使用者。

6.  選取 [儲存變更]。

7.  在 [VSTS 群組] 清單中，選取 [端點管理員]。

8.  在 [成員] 索引標籤上，選取 [新增]。

9.  在 [新增使用者和群組] 中，輸入使用者名稱，然後從使用者清單中選取該使用者。

10. 選取 [儲存變更]。

端點資訊已存在，所以 VSTS 對 Azure Stack 的連線已可供使用。 Azure Stack 中的組建代理程式會取得來自 VSTS 的指示，然後代理程式會傳達與 Azure Stack 進行通訊所需的端點資訊。

## <a name="develop-the-application-build"></a>開發應用程式組建

> [!Note]  
> 需要適當映像聯合執行的 Azure Stack (Windows Server 和 SQL) 及 App Service 部署。 檢閱 App Service 文件中適用於 Azure Stack 操作員的「[開始使用 Azure Stack 上的 App Service 之前](/articles/azure-stack/azure-stack-app-service-before-you-get-started)」區段。

請使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/) (例如來自 VSTS 的 Web 應用程式程式碼) 以部署至這兩個雲端。

### <a name="add-code-to-a-vsts-project"></a>將程式碼新增至 VSTS 專案

1.  使用在 Azure Stack 上具有專案建立權限的帳戶登入 VSTS。 下一個螢幕擷取畫面顯示如何連線至 HybridCICD 專案。

2.  建立並開啟預設 Web 應用程式以**複製存放庫**。

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署

1.  編輯 **WebApplication.csproj** 檔案：選取 [Runtimeidentifier]，然後新增 win10-x64。 如需詳細資訊，請參閱[獨立式部署](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)文件。

2.  使用 Team Explorer 將程式碼簽入 VSTS。

3.  確認應用程式程式碼已簽入 Visual Studio Team Services 中。

### <a name="create-the-build-definition"></a>建立組建定義

1.  使用可以建立組建定義的帳戶來登入 VSTS。

2.  巡覽至專案的 [組建 Web 應用程式] 頁面。

3.  在 [引數] 中，新增 **-r win10-x64** 程式碼。 這是觸發 .Net Core 的獨立部署時所需的程式碼。

4.  執行組建。 [獨立的部署組建](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)程序將會發佈可在 Azure 和 Azure Stack 上執行的成品。

#### <a name="use-an-azure-hosted-build-agent"></a>使用 Azure 託管的組建代理程式

在 VSTS 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動執行代理程式的維護和升級，以支援持續而不間斷的開發週期。

### <a name="configure-the-continuous-deployment-cd-process"></a>設定持續部署 (CD) 程序

Visual Studio Team Services (VSTS) 和 Team Foundation Server (TFS) 提供具有高度設定和管理能力的管線，可用於對多個環境的發行 (例如開發、暫存、品質保證 (QA) 和生產環境)。 此程序中可以包括在應用程式生命週期的特定階段要求核准。

#### <a name="create-release-definition"></a>建立發行定義

建立發行定義是應用程式建置程序的最後一個步驟。 這個發行定義可用來建立發行並部署組建。

1.  登入 VSTS，並瀏覽至專案的 [建置與發行]。

2.  在 [發行] 索引標籤上，選取 [ + ]，然後挑選 [建立發行定義]。

3.  在 [選取範本] 中，選擇 [Azure App Service 部署]，然後選取 [套用]。

4.  在 [新增成品] 上，從**來源 (組建定義) 中選取 [Azure 雲端建置應用程式]。

5.  在 [管線] 索引標籤上，選取 [檢視環境工作] 的 [1 階段 1 個工作] 連結。

6.  在 [工作] 索引標籤上，輸入「Azure」作為 [環境名稱]，然後從 [Azure 訂用帳戶] 清單選取 [AzureCloud Traders-Web EP]。

7.  輸入 **Azure App Service 名稱**，也就是下一個螢幕擷取畫面中的 `northwindtraders`。

8.  在 [代理程式階段] 中，從 [代理程式佇列] 清單中選取 [Hosted VS2017]。

9.  在 [部署 Azure App Service] 中，為環境選取有效的**套件或資料夾**。

10. 在 [選取檔案或資料夾] 中，對 [位置] 選取 [確定]。

11. 儲存所有變更，並返回 [管線]。

12. 在 [管線] 索引標籤上，選取 [新增成品]，然後從**來源 (組建定義) **清單選擇 [NorthwindCloud Traders-Vessel]。

13. 在 [選取範本] 上，新增另一個環境。 挑選 [Azure App Service 部署]，然後選取 [套用]。

14. 輸入 `Azure Stack` 作為 [環境名稱]。

15. 在 [工作] 索引標籤上，尋找並選取 Azure Stack。

16. 從 [Azure 訂用帳戶] 清單中，選取 [AzureStack Traders-Vessel EP] 作為 Azure Stack 端點。

17. 輸入 Azure Stack Web 應用程式名稱作為 [App Service 名稱]。

18. 在 [代理程式選擇] 底下，從 [代理程式佇列] 清單中挑選 **AzureStack -b Douglas Fir**。

19. 在 [部署 Azure App Service] 中，為環境選取有效的**套件或資料夾**。 在 [選取檔案或資料夾] 中，對 [位置] 資料夾選取 [確定]。

20. 在 [變數] 索引標籤上，尋找名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數。 將變數值設定為 **true**，並將其範圍設定為 [Azure Stack]。

21. 在 [管線] 索引標籤上，選取 NorthwindCloud Traders-Web 成品的 [持續部署觸發程序] 圖示，並將 [持續部署觸發程序] 設定為 [啟用]。 針對 **NorthwindCloud Traders-Vessel** 成品執行相同的動作。

22. 針對 Azure Stack 環境，選取 [部署前的條件] 圖示，將觸發程序設定為 [發行之後]。

23. 儲存所有變更。

> [!Note]  
> 發行工作的某些設定已在從範本建立發行定義時自動定義為[環境變數](https://docs.microsoft.com/vsts/build-release/concepts/definitions/release/variables?view=vsts#custom-variables)。 這些設定無法在工作設定中修改，但是可以在父環境項目中修改。

## <a name="create-a-release"></a>建立發行

1.  在 [管線] 索引標籤上，開啟 [發行] 清單，然後選擇 [建立發行]。

2.  輸入發行的說明，確認已選取正確的成品，然後選擇 [建立]。 幾分鐘之後將會出現一個橫幅，指出新的發行已建立，且發行名稱會顯示為連結。 選擇連結以查看 [發行摘要] 頁面。

3.  [發行摘要] 頁面會顯示有關發行的詳細資料。 在下列 "Release-2" 螢幕擷取畫面中，[環境] 區段會顯示 Azure 的 [部署狀態] 為 [進行中]，Azure Stack 的狀態為 [成功]。 當 Azure 環境的部署狀態變更為 [成功] 時，便會出現橫幅指出發行已可供核准。 對部署若擱置或失敗，將會顯示藍色的 **(i)** 資訊圖示。 將滑鼠暫留在圖示上，即可查看快顯，其中會包含延遲或失敗的原因。

4.  其他檢視 (例如發行清單) 也會顯示指出核准擱置中的圖示。 這個圖示的快顯會顯示環境名稱以及更多與部署相關的詳細資料。 系統管理員可輕鬆地查看發行的整體進度，以及查看哪些版本正在等待核准。

## <a name="monitor-and-track-deployments"></a>監視和追蹤部署

1.  在 [Release-2] 摘要頁面上，選取 [記錄]。 在部署期間，此頁面會顯示來自代理程式的即時記錄。 左窗格會顯示每個環境部署中每個作業的狀態。

2.  選擇部署前或部署後核准 [動作] 資料行中的人形圖示，以查看哪些人核准 (或拒絕) 部署，以及該人員提供的訊息。

3.  部署完成後，整個記錄檔會顯示在右窗格中。 在左窗格中選取任何 [步驟]，以查看單一步驟的記錄檔，例如**初始化作業**。 能夠查看個別記錄，可讓您輕鬆地針對整體部署的各個部分進行追蹤和偵錯。 [儲存] 步驟的記錄檔，或是 [將所有記錄下載為 zip]。

4.  開啟 [摘要] 索引標籤可查看發行的一般資訊。 此檢視會顯示組建的詳細資料、組件所部署到的環境、部署狀態，以及其他關於發行的資訊。

5.  選取環境連結 (**Azure** 或 **Azure Stack**) 可查看特定環境現有和擱置中部署的資訊。 使用這些檢視來快速確認同一個組建已部署至兩個環境。

6.  在瀏覽器中開啟**已部署的生產應用程式**。 例如，針對 Azure App Service 網站，請開啟 URL [http://[您的應用程式名稱\].azurewebsites.net](http:// [your-app-name].azurewebsites.net)。

**Azure 與 Azure Stack 的整合提供可縮放的跨雲端解決方案**

具彈性且完善的多雲端服務可提供資料安全性、備份和備援、一致且快速的可用性、可縮放的儲存體和散發，以及異地相容的路由。 此手動觸發程序可確實地在託管的 Web 應用程式之間提供可靠且有效率的負載切換，確保重要資料的立即可用性。 

## <a name="next-steps"></a>後續步驟
- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。