---
title: 使用 Azure 和 Azure Stack 建立異地分散應用程式解決方案 | Microsoft Docs
description: 了解如何使用 Azure 和 Azure Stack 建立異地分散應用程式解決方案。
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
ms.openlocfilehash: 19e7506dac82e4d12d5aecbdb5ae1c14fb944c29
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961530"
---
# <a name="tutorial-create-a-geo-distributed-app-solution-with-azure-and-azure-stack"></a>教學課程：使用 Azure 和 Azure Stack 建立異地分散應用程式解決方案

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

了解如何根據各種計量，使用異地分散應用程式模式將流量導向至特定端點。 使用以地理位置為基礎的路由和端點組態來建立流量管理員設定檔，可確保資訊會根據區域需求、公司與國際法規和您的資料需求路由至端點。

在本教學課程中，您將建置範例環境，用以：

> [!div class="checklist"]
> - 建立異地分散應用程式。
> - 使用流量管理員設定應用程式的目標。

## <a name="use-the-geo-distributed-apps-pattern"></a>使用異地分散應用程式模式

透過異地分散模式，您的應用程式將可跨越區域。 您可以預設為公用雲端，但某些使用者可能會需要將資料保留在其區域中。 您可以根據使用者的需求將他們導向至最適合的雲端。

### <a name="issues-and-considerations"></a>問題和考量

#### <a name="scalability-considerations"></a>延展性考量

您將在本教學課程中建置的解決方案無法處理延展性的問題。 不過，如果與其他 Azure 和內部部署技術和解決方案搭配使用，即可滿足延展性方面的需求。 如需透過流量管理員使用自動調整建立混合式解決方案的相關資訊，請參閱[使用 Azure 建立跨雲端縮放解決方案](azure-stack-solution-cloud-burst.md)。

#### <a name="availability-considerations"></a>可用性考量

和延展性考量一樣，此解決方案無法直接處理可用性的問題。 但同樣和延展性考量相仿，此解決方案中可以實作 Azure 和內部部署技術與解決方案，以確保所有相關元件皆具有高可用性。

### <a name="when-to-use-this-pattern"></a>使用此模式的時機

- 您的組織有海外分公司，且需要自訂的區域安全性和發佈原則。

- 您的每個組織辦公室皆提取員工、商務和設備資料，而需要各自的當地法規和時區的報告活動。

- 只要對單一區域內和跨區域的多個應用程式部署進行應用程式的水平相應放大，即可達到高延展性需求，以處理極高的負載需求。

### <a name="planning-the-topology"></a>規劃拓樸

在建置分散式應用程式的使用量之前，最好先具備下列知識：

-   **應用程式的自訂網域：** 客戶將用來存取應用程式的自訂網域名稱為何？ 範例應用程式的自訂網域名稱是 *www.scalableasedemo.com*。

-   **流量管理員網域：** 建立 [Azure 流量管理員設定檔](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles)時必須選擇網域名稱。 此名稱會與 *trafficmanager.net* 尾碼結合，以註冊流量管理員所管理的網域項目。 就範例應用程式而言，選擇的名稱是 *scalable-ase-demo*。 因此，流量管理員所管理的完整網域名稱是 *scalable-ase-demo.trafficmanager.net*。

-   **調整應用程式使用量的策略：** 應用程式使用量是否會分散到單一區域中的多個 App Service 環境？ 多個區域嗎？ 兩種方法混用嗎？ 決策依據應來自於客戶流量的來源位置，以及其餘應用程式的支援後端基礎結構的可調整性。 例如，對於 100% 無狀態的應用程式，可以使用每一 Azure 區域多個 App Service 環境的組合，乘以跨多個 Azure 區域部署的 App Service 環境數，來大幅調整應用程式。 由於有 15 個以上的全球 Azure 區域可供選擇，客戶將可真正建置全球性超高延展性的應用程式使用量。 在本文所使用的範例應用程式中，有三個 App Service 環境建立在單一 Azure 區域 (美國中南部) 中。

-   **App Service 環境的命名慣例：** 每個 App Service 環境都需要唯一名稱。 有兩個或更多 App Service 環境時，命名慣例將有助於識別每個 App Service 環境。 範例應用程式中使用簡單的命名慣例。 三個 App Service 環境的名稱分別是 fe1ase、fe2ase 和 fe3ase。

-   **應用程式的命名慣例：** 由於將會部署多個應用程式執行個體，每個部署的應用程式執行個體都要有名稱。 使用 App Service 環境時，多個 App Service 環境可以使用相同的應用程式名稱。 由於每個 App Service 環境都有唯一的網域尾碼，開發人員可以選擇在每個環境中重複使用相同的應用程式名稱。 例如，開發人員可以將應用程式命名如下：myapp.foo1.p.azurewebsites.net、myapp.foo2.p.azurewebsites.net、myapp.foo3.p.azurewebsites.net，依此類推。就此案例中的應用程式而言，每個應用程式執行個體都有唯一名稱。 所使用的應用程式執行個體名稱是 webfrontend1、webfrontend2 和 webfrontend3。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。

## <a name="part-1-create-a-geo-distributed-app"></a>第 1 部分：建立異地分散應用程式

在此部分中，您將建立 Web 應用程式。

> [!div class="checklist"]
> - 建立 Web 應用程式並發佈
> - 將程式碼新增至 VSTS 專案
> - 將應用程式組建指向多個雲端目標。
> - 管理和設定 CD 程序

### <a name="prerequisites"></a>必要條件

需要 Azure 訂用帳戶並安裝 Azure Stack。

### <a name="geo-distributed-app-steps"></a>異地分散應用程式的步驟

### <a name="obtain-a-custom-domain-and-configure-dns"></a>取得自訂網域並設定 DNS

更新 DNS 區域檔案，以建立 Web 應用程式並發佈網域。 然後，Azure AD 會驗證自訂網域名稱的擁有權。 對於 Azure 中的 Azure/Office 365/外部 DNS 記錄使用 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal)，或在[不同的 DNS 註冊機構](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)新增 DNS 項目。

1. 向公用註冊機構註冊自訂網域。

2. 登入網域的網域名稱註冊機構。 已核准的系統管理員可能需要進行 DNS 更新。

3. 透過新增 Azure AD 提供的 DNS 項目來更新網域的 DNS 區域檔案。 DNS 項目不會變更郵件路由或 Web 裝載等行為。

### <a name="create-web-apps-and-publish"></a>建立 Web 應用程式並發佈

設定混合式 CI/CD 以將 Web 應用程式部署至 Azure 和 Azure Stack，並自動將變更推送至這兩個雲端。

> [!Note]  
> 需要適當映像聯合執行的 Azure Stack (Windows Server 和 SQL) 及 App Service 部署。 檢閱 App Service 文件中適用於 Azure Stack 操作員的「[開始使用 Azure Stack 上的 App Service 之前](/articles/azure-stack/azure-stack-app-service-before-you-get-started)」區段。

#### <a name="add-code-to-vsts-project"></a>將程式碼新增至 VSTS 專案

1. 使用在 VSTS 上**具有專案建立權限的帳戶**登入 Visual Studio。

    混合式持續整合/持續傳遞 (CI/CD) 可同時套用至應用程式程式碼和基礎結構程式碼。 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/)進行私用與託管的雲端開發。

    ![替代文字](media\azure-stack-solution-geo-distributed\image1.JPG)

2. 建立並開啟預設 Web 應用程式以**複製存放庫**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>在這兩個雲端中建立 Web 應用程式部署

1.  編輯 **WebApplication.csproj** 檔案：選取 [Runtimeidentifier]，然後新增 **win10-x64**。 (請參閱[獨立式部署](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)文件。)

    ![替代文字](media\azure-stack-solution-geo-distributed\image3.png)

1.  使用 Team Explorer **將程式碼簽入 VSTS 中**。

2.  確認**應用程式程式碼**已簽入 Visual Studio Team Services 中。

### <a name="create-the-build-definition"></a>建立組建定義

1. **登入 VSTS** 以確認能夠建立組建定義。

2. 新增 **-r win10-x64** 程式碼。 這是觸發 .Net Core 的獨立部署時所需的程式碼。

    ![替代文字](media\azure-stack-solution-geo-distributed\image4.png)

3. **執行組建**。 [獨立的部署組建](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)程序將會發佈可在 Azure 和 Azure Stack 上執行的成品。

**使用 Azure 託管的代理程式**

在 VSTS 中使用託管的代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動執行維護和升級，以支援持續而不間斷的開發、測試和部署。

### <a name="manage-and-configure-the-cd-process"></a>管理和設定 CD 程序

Visual Studio Team Services (VSTS) 和 Team Foundation Server (TFS) 提供具有高度設定和管理能力的管線，可用於對多個環境的發行 (例如開發、暫存、QA 和生產環境)；包括特定階段需要核准。

#### <a name="create-release-definition"></a>建立發行定義


![替代文字](media\azure-stack-solution-geo-distributed\image5.png)

1.  在 Visual Studio Online (VSO) 的 [建置及發行] 頁面的 [發行] 索引標籤下選取**加號**按鈕，以新增發行。

    ![替代文字](media\azure-stack-solution-geo-distributed\image6.png)

2. 套用 **Azure App Service 部署**範本。

    ![替代文字](media\azure-stack-solution-geo-distributed\image7.png)

3. 在 [新增成品] 下拉式功能表中，為 Azure 雲端建置應用程式**新增成品**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image8.png)

4. 在 [管線] 索引標籤下選取環境的 [階段]、[工作] 連結，並設定 Azure 雲端環境值。

    ![替代文字](media\azure-stack-solution-geo-distributed\image9.png)

5. 設定 [環境名稱]，並選取 Azure 雲端端點的 Azure**訂用帳戶**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image10.png)

6. 在 [環境名稱] 下設定所需的 **Azure App Service 名稱**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image11.png)

7. 在 Azure 雲端託管環境的代理程式佇列下輸入 **Hosted VS2017**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image12.png)

8. 在 [部署 Azure App Service] 功能表中，為環境選取有效的**套件或資料夾**。 對**資料夾位置**選取 [確定]。

    ![替代文字](media\azure-stack-solution-geo-distributed\image13.png)

    ![替代文字](media\azure-stack-solution-geo-distributed\image14.png)

9. 儲存所有變更，並返回**發行管線**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image15.png)

10. 選取 Azure Stack 應用程式的組建，以新增**新成品**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image16.png)

11. 再新增一個套用 **Azure App Service 部署**的環境。

    ![替代文字](media\azure-stack-solution-geo-distributed\image17.png)

12. 將新環境命名為 **Azure Stack**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image18.png)

13. 在 [工作] 索引標籤下找出 Azure Stack 環境。

    ![替代文字](media\azure-stack-solution-geo-distributed\image19.png)

14. 選取 Azure Stack 端點的**訂用帳戶**。

  ![替代文字](media\azure-stack-solution-geo-distributed\image20.png)

15. 將 Azure Stack Web 應用程式名稱設定為 **App Service 名稱**。

    ![替代文字](media\azure-stack-solution-geo-distributed\image21.png)

16. 選取 [Azure Stack 代理程式]。

    ![替代文字](media\azure-stack-solution-geo-distributed\image22.png)

17. 在 [部署 Azure App Service] 區段下，為環境選取有效的**套件或資料夾**。 對**資料夾位置**選取 [確定]。

    ![替代文字](media\azure-stack-solution-geo-distributed\image23.png)

    ![替代文字](media\azure-stack-solution-geo-distributed\image24.png)

18. 在 [變數] 索引標籤下新增名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數，並將其值設定為 `true`，範圍設定為 `Azure Stack`。

    ![替代文字](media\azure-stack-solution-geo-distributed\image25.png)

19. 選取兩個成品中的 [持續部署觸發程序] 圖示，並啟用**持續**部署觸發程序。

    ![替代文字](media\azure-stack-solution-geo-distributed\image26.png)

20. 選取 Azure Stack 環境中的 [預先部署] 條件圖示，並將觸發程序設定為 [發行之後]。

21. 儲存所有變更。

> [!Note]  
>  工作的某些設定可能已在您從範本建立發行定義時自動定義為[環境變數](https://docs.microsoft.com/vsts/build-release/concepts/definitions/release/variables?view=vsts#custom-variables)。 這些設定無法在工作設定中修改；您必須選取父環境項目才能編輯這些設定。

## <a name="part-2-update-web-app-options"></a>第 2 部分：更新 Web 應用程式選項

[Azure Web Apps](https://docs.microsoft.com/azure/app-service/app-service-web-overview) 提供可高度擴充、自我修復的 Web 主機服務。 

![替代文字](media\azure-stack-solution-geo-distributed\image27.png)

> [!div class="checklist"]
> - 將現有的自訂 DNS 名稱對應至 Azure Web Apps
> - 使用 **CNAME 記錄或 **A 記錄**將自訂 DNS 名稱對應至 App Service。

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>將現有的自訂 DNS 名稱對應至 Azure Web Apps

> [!Note]  
>  請對所有自訂 DNS 名稱使用 CNAME，但根網域除外 (例如 northwind.com)。

若要將即時網站及其 DNS 網域名稱移轉至 App Service，請參閱[將作用中的 DNS 名稱移轉至 Azure App Service](https://docs.microsoft.com/azure/app-service/app-service-custom-domain-name-migrate)。

### <a name="prerequisites"></a>必要條件

若要完成本教學課程：

-   [建立 App Service 應用程式](https://docs.microsoft.com/azure/app-service/)，或使用您在另一個教學課程中建立的應用程式。

-   購買網域名稱，並確定您可存取網域提供者的 DNS 登錄。

更新網域的 DNS 區域檔案。 Azure AD 會驗證自訂網域名稱的擁有權。 對於 Azure 中的 Azure/Office 365/外部 DNS 記錄使用 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal)，或在[不同的 DNS 註冊機構](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)新增 DNS 項目。

-   向公用註冊機構註冊自訂網域。

-   登入網域的網域名稱註冊機構。 (已核准的系統管理員可能需要進行 DNS 更新。)

-   透過新增 Azure AD 提供的 DNS 項目來更新網域的 DNS 區域檔案。

例如，若要新增 DNS 項目 fornorthwindcloud.comandwww.northwindcloud.com，請設定 thenorthwindcloud.com 根網域的 DNS 設定。

> [!Note]  
>  網域名稱可使用 [Azure 入口網站](https://docs.microsoft.com/azure/app-service/custom-dns-web-site-buydomains-web-app)來購買。  
> 若要將自訂 DNS 名稱對應至 Web 應用程式，Web 應用程式的 [App Service 方案](https://azure.microsoft.com/pricing/details/app-service/)必須是付費層 (**共用**、**基本**、**標準**或**進階**)。



### <a name="create-and-map-cname-and-a-records"></a>建立和對應 CNAME 與 A 記錄

#### <a name="access-dns-records-with-domain-provider"></a>存取網域提供者中的 DNS 記錄

> [!Note]  
>  請使用 Azure DNS 來設定 Azure Web Apps 的自訂 DNS 名稱。 如需詳細資訊，請參閱[使用 Azure DNS 為 Azure 服務提供自訂網域設定](https://docs.microsoft.com/azure/dns/dns-custom-domain)。

1.  登入主要網域提供者的網站。

2.  尋找管理 DNS 記錄的頁面。 每個網域提供者有自己的 DNS 記錄介面。 在網站中尋找標示為 [網域名稱]、[DNS] 或 [名稱伺服器管理] 的區域。

您可以在 [我的網域] 中檢視 DNS 記錄頁面。 尋找名為 [區域檔案]、[DNS 記錄] 或 [進階設定] 的連結。

下列螢幕擷取畫面是 DNS 記錄頁面的範例：

![DNS 記錄頁面範例](media\azure-stack-solution-geo-distributed\image28.png)

1.  在 [網域名稱註冊機構] 中，選取 [新增或建立] 以建立記錄。 某些提供者有不同的連結來新增其他記錄類型。 請參閱提供者的文件。

2.  新增 CNAME 記錄以將子網域對應至應用程式的預設主機名稱。

  在 www.northwindcloud.com 網域範例中，請新增將名稱 www 對應至 <app\_name>.azurewebsites.net 的 CNAME 記錄。

新增 CNAME 之後，DNS 記錄分頁看起來如下列範例所示：

![入口網站瀏覽至 Azure 應用程式](media\azure-stack-solution-geo-distributed\image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>在 Azure 中啟用 CNAME 記錄對應

1.  在新的索引標籤中，登入 Azure 入口網站。

2.  瀏覽至 App Service。

3.  選取 Web 應用程式。

4.  在 Azure 入口網站之應用程式分頁的左側導覽中，選取 [自訂網域]。

5.  選取 [新增主機名稱] 旁的 **+** 圖示。

1.  輸入完整網域名稱，例如 `www.northwindcloud.com`。

2.  選取 [驗證]。

3.  如有指示，請再將其他類型的記錄 (`A` 或 `TXT`) 新增至網域名稱註冊機構 DNS 記錄。 Azure 會提供這些記錄的類型與值：

    a.  **A** 記錄對應至應用程式的 IP 位址。

    b.  **TXT** 記錄對應至應用程式的預設主機名稱 <app_name>.azurewebsites.net。 App Service 只會在設定時使用此記錄，以確認自訂網域擁有權。 確認之後，請刪除 TXT 記錄。

4.  在網域註冊機構索引標籤中完成這項工作並重新驗證，直到 [新增主機名稱] 按鈕啟動為止。

5.  確定 **[主機名稱記錄類型] 設為 [CNAME (www.example.com 或任何子網域)]。

6.  選取 [新增主機名稱]。

7.  輸入完整網域名稱，例如 `northwindcloud.com`。

8.  選取 [驗證]。

9.  [新增] 已啟動。

10. 確定 **[主機名稱記錄類型] 設為 [A 記錄 (example.com)]。

11. **新增主機名稱**。

  可能需要一些時間，新的主機名稱才會反映在應用程式的 [自訂網域] 頁面中。 嘗試重新整理瀏覽器以更新資料。
  
  ![替代文字](media\azure-stack-solution-geo-distributed\image31.png) 
  
  如果發生錯誤，頁面底部會出現驗證錯誤通知。 ![驗證錯誤](media\azure-stack-solution-geo-distributed\image32.png)

> [!Note]  
>  您可以重複執行上述步驟，以對應萬用字元網域 (\*.northwindcloud.com)。 如此即可直接將其他子網域新增至此應用程式服務，而無須為每個子網域建立個別的 CNAME 記錄。 請依照註冊機構的指示進行此設定。

#### <a name="test-in-a-browser"></a>在瀏覽器中測試

瀏覽至您稍早設定的 DNS 名稱 (例如 `northwindcloud.com`、www.northwindcloud.com。

## <a name="part-3-bind-a-custom-ssl-cert"></a>第 3 部分：繫結自訂 SSL 憑證

在此部分中：

> [!div class="checklist"]
> - 將自訂 SSL 憑證繫結至 App Service
> - 為應用程式強制使用 HTTPS
> - 使用指令碼來自動繫結 SSL 憑證

> [!Note]  
> 如有需要，請在 Azure 入口網站中取得客戶的 SSL 憑證，並將其繫結至 Web 應用程式。 遵循 [App Service 憑證教學課程](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)。

### <a name="prerequisites"></a>必要條件

若要完成本教學課程：

-   [建立 App Service 應用程式](https://docs.microsoft.com/azure/app-service/)
-   [將自訂 DNS 名稱對應至 Web 應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
-   取得受信任憑證授權單位所核發的 SSL 憑證，並使用金鑰簽署要求

### <a name="requirements-for-your-ssl-certificate"></a>SSL 憑證的需求

若要在 App Service 中使用憑證，憑證必須符合以下所有需求︰

-   由受信任的憑證授權單位簽署

-   以受密碼保護的 PFX 檔案形式匯出

-   包含長度至少為 2048 位元的私密金鑰

-   包含憑證鏈結中的所有中繼憑證

> [!Note]  
>  **橢圓曲線密碼編譯 (ECC) 憑證**可與 App Service 搭配使用，但不在本指南的討論範圍內。 如需建立 ECC 憑證方面的協助，請洽詢憑證授權單位。 

#### <a name="prepare-the-web-app"></a>準備 Web 應用程式

若要將自訂 SSL 憑證繫結至 Web 應用程式，[App Service 方案](https://azure.microsoft.com/pricing/details/app-service/)必須為**基本**、**標準**或**進階**層。

#### <a name="sign-in-to-azure"></a>登入 Azure

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，並瀏覽至 Web 應用程式。

2.  從左側功能表中選取 [App Service]，然後選取 Web 應用程式名稱。

![選取 Web 應用程式](media\azure-stack-solution-geo-distributed\image33.png)

#### <a name="check-the-pricing-tier"></a>檢查定價層

1.  在 Web 應用程式頁面的左側導覽中，捲動到 [設定] 區段，然後選取 [相應增加 (App Service 方案)]。

    ![相應增加功能表](media\azure-stack-solution-geo-distributed\image34.png)

1.  確定 Web 應用程式不在**免費**或**共用** 層中。 系統會以深藍色方塊醒目顯示 Web 應用程式目前的層。

    ![檢查定價層](media\azure-stack-solution-geo-distributed\image35.png)

**免費**和**共用**層中不支援自訂 SSL。 若要升級，請依照下一節中的步驟操作，或**選擇您的定價層**頁面，然後跳至[上傳並繫結 SSL 憑證](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。

#### <a name="scale-up-your-app-service-plan"></a>相應增加您的 App Service 方案

1.  選取**基本****標準**或**高階**層的其中一個。

2.  選取 [選取] 。

![選擇定價層](media\azure-stack-solution-geo-distributed\image36.png)

顯示通知時，表示調整作業已完成。

![相應增加通知](media\azure-stack-solution-geo-distributed\image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>繫結您的 SSL 憑證，並合併中繼憑證

將多個憑證合併到鏈結中。

1. 在文字編輯器中**開啟您收到的每個憑證**。

2. 為合併的憑證建立一個檔案，並取名為 *mergedcertificate.crt*。 在文字編輯器中，將每個憑證的內容複製到這個檔案中。 憑證的順序應該遵循在憑證鏈結中的順序，開頭為您的憑證，以及結尾為根憑證。 看起來會像下列範例：

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>將憑證匯出為 PFX

使用憑證所產生的私密金鑰，將合併的 SSL 憑證匯出。

系統會透過 OpenSSL 建立私密金鑰檔案。 若要將憑證匯出為 PFX，請執行下列命令，並將預留位置 *<private-key-file>* 和 *<merged-certificate-file>* 取代為私密金鑰路徑與合併的憑證檔案。

```PowerShell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

出現提示時，請定義您之後將 SSL 憑證上傳至 App Service 時所使用的密碼。

如果您使用 IIS 或 **Certreq.exe** 產生憑證要求，請將憑證安裝至本機電腦，然後[將憑證匯出為 PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx)。

#### <a name="upload-the-ssl-certificate"></a>上傳 SSL 憑證

1.  在 Web 應用程式的左側導覽中選取 [SSL 設定]。

2.  選取 [上傳憑證]。

3.  在 [PFX 憑證檔案] 中，選取 PFX 檔案。

4.  4. 在 [憑證密碼] 中，輸入匯出 PFX 檔案時所建立的密碼。

5.  選取 [上傳] 。

![Upload certificate](media\azure-stack-solution-geo-distributed\image38.png)

當 App Service 完成憑證上傳作業時，憑證會出現在 [SSL 設定] 頁面中。

![替代文字](media\azure-stack-solution-geo-distributed\image39.png)

#### <a name="bind-your-ssl-certificate"></a>繫結 SSL 憑證

1.  在 [SSL 繫結] 區段中，選取 [新增繫結]。

    > [!Note]  
    >  如果憑證已上傳，但未出現在 [主機名稱] 下拉式清單的網域名稱中，請嘗試重新整理瀏覽器頁面。

1.  在 [新增 SSL 繫結] 頁面中，使用下拉式清單選取要保護的網域名稱，以及要使用的憑證。

2.  在 [SSL 類型]中，選擇要使用[**伺服器名稱指示 (SNI)**](http://en.wikipedia.org/wiki/Server_Name_Indication) 還是以 IP 為基礎的 SSL。

-   **以 SNI 為基礎的 SSL** - 可能會新增多個以 SNI 為基礎的 SSL 繫結。 此選項可允許多個 SSL 憑證保護同一個 IP 位址上的多個網域。 大多數現代化的瀏覽器 (包括 Internet Explorer、Chrome、Firefox 和 Opera) 都支援 SNI (可在[伺服器名稱指示](http://wikipedia.org/wiki/Server_Name_Indication)找到更完整的瀏覽器支援資訊)。

-   **以 IP 為基礎的 SSL** - 可能只會新增一個以 IP 為基礎的 SSL 繫結。 此選項只允許一個 SSL 憑證保護專用的公用 IP 位址。 若要保護多個網域，請全部使用相同的 SSL 憑證來保護它們。 這是 SSL 繫結的傳統選項。

    1.  選取 [新增繫結]。

    ![替代文字](media\azure-stack-solution-geo-distributed\image40.png)

當 App Service 完成憑證的上傳時，憑證會出現在 [SSL 繫結] 區段中。

![替代文字](media\azure-stack-solution-geo-distributed\image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>將 IP SSL 的 A 記錄重新對應

如果不在 Web 應用程式中使用以 IP 為基礎的 SSL，請跳至[測試自訂網域的 HTTPS](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。

根據預設，Web 應用程式會使用共用的公用 IP 位址。 當憑證與以 IP 為基礎的 SSL 繫結時，App Service 會為 Web 應用程式建立新的專用 IP 位址。

當 A 記錄對應至 Web 應用程式時，必須使用專用的 IP 位址更新網域登錄。

[自訂網域] 頁面會以新的專用 IP 位址進行更新。 請複製此 [IP 位址](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)，然後將 [A 記錄](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)重新對應至這個新的 IP 位址。

#### <a name="test-https"></a>測試 HTTPS

在不同的瀏覽器中瀏覽至 https://<your.custom.domain>，以確定可存取 Web 應用程式。

![替代文字](media\azure-stack-solution-geo-distributed\image42.png)

> [!Note]  
> 如果發生憑證驗證錯誤，這可能是自我簽署的憑證所致，或是在匯出至 PFX 檔案時遺漏了中繼憑證。

#### <a name="enforce-https"></a>強制使用 HTTPS

根據預設，任何人皆可使用 HTTP 存取 Web 應用程式。 所有 HTTP 要求都可重新都導向至 HTTPS 連接埠。

在 [Web 應用程式] 頁面中，選取 [SSL 設定]。 然後，在 [僅限 HTTPS] 中選取 [開啟]。

![強制使用 HTTPS](media\azure-stack-solution-geo-distributed\image43.png)

當作業完成時，請瀏覽至指向應用程式的任何 HTTP URL。 例如︰

-   http://<app_name>.azurewebsites.net
-   http://northwindcloud.com
-   <http://www.northwindcloud.com>

#### <a name="enforce-tls-1112"></a>強制使用 TLS 1.1/1.2

應用程式依預設會允許 [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0，但業界標準 (例如 [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)) 已不再將它視為安全的。 若要強制使用更高的 TLS 版本，請遵循下列步驟：

1.  在 Web 應用程式頁面的左側導覽中，選取 [SSL 設定]。

2.  在 [TLS 版本] 中，選取最低的 TLS 版本。

![強制使用 TLS 1.1 或 1.2](media\azure-stack-solution-geo-distributed\image44.png)

### <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔

1.  選取 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。

2.  在 [建立流量管理員設定檔] 中，如下所示操作：

    1.  在 [名稱]中，提供設定檔的名稱。 此名稱在 trafficmanager.net 區域內必須是唯一的，而且會產生 DNS 名稱 trafficmanager.net，用以存取流量管理員設定檔。

    2.  在 [路由方法] 中，選取 [地理路由方法]。

    3.  在 [訂用帳戶] 中，選取要用來建立此設定檔的訂用帳戶。

    4.  在 [資源群組]中，建立新的資源群組來放置此設定檔。

    5.  在 [資源群組位置] 中，選取資源群組的位置。 這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。

    6.  選取 [建立] 。

    7.  當流量管理員設定檔的全球部署完成時，它會列為個別資源群組的其中一個資源。

    ![替代文字](media\azure-stack-solution-geo-distributed\image45.png)

### <a name="add-traffic-manager-endpoints"></a>新增流量管理員端點

1.  在入口網站的搜尋列中，搜尋您在上一節建立的 **流量管理員設定檔** 名稱，然後在顯示的結果中選取流量管理員設定檔。

2.  在 [流量管理員設定檔] 的 [設定] 區段中，選取 [端點]。

3.  選取 [新增] 。

4.  新增 Azure Stack 端點。

5.  針對 [類型]，選取 [外部端點]。

6.  提供此端點的 [名稱]，最好是 Azure Stack 的名稱。

7.  針對完整網域名稱 (**FQDN**)，請使用 Azure Stack Web 應用程式的外部 URL。

8.  在 [地區對應] 下方，選取資源所在的區域/洲別，例如**歐洲**。

9.  在顯示的 [國家/區域] 下拉式清單下方，選取將套用到此端點的國家/地區，例如**德國**。

10. 維持不勾選 [新增為已停用]。

11. 選取 [確定] 。

12. 新增 Azure 端點：

    1.  針對 [類型]，選取 [Azure 端點]。

    2.  提供此端點的 [名稱]。

    3.  針對 [目標資源類型]，選取 [App Service]。

    4.  針對 [目標資源]，選取 [選擇應用程式服務]，以顯示相同訂用帳戶下的 Web Apps 清單。 在 [資源] 中，挑選要作為第一個端點的應用程式服務。

13. 在 [地區對應] 下方，選取資源所在的區域/洲別，例如**北美洲/中美洲/加勒比海**。

14. 在顯示的 [國家/區域] 下拉式清單下方，將此清單保留為空白，以選取上述所有的區域群組。

15. 維持不勾選 [新增為已停用]。

16. 選取 [確定]

  > [!Note]  
  >  建立至少一個地理範圍為 [全部 (全球)] 的端點，作為資源的預設端點。

1.  這兩個端點新增完畢後，它們會顯示在 [流量管理員設定檔] 中，而且監視狀態是 [線上]。

  ![替代文字](media\azure-stack-solution-geo-distributed\image46.png)

**國際企業需依賴 Azure 異地分散功能**

透過 Azure 流量管理員和地理位置專屬端點導向資料流量，國際企業得以遵循地區法規，並且維持資料的合規性和安全性，這是地區企業的發展和遠端位置而言都至關重要。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。