---
title: 使用 Azure DevOps Project 將 ASP.NET 應用程式部署至 Azure 虛擬機器 | Azure DevOps Services 教學課程
description: DevOps Project 可讓您輕鬆地開始使用 Azure。 Azure DevOps Project 可讓您以幾個快速步驟輕鬆地將 ASP.NET 應用程式部署至 Azure 虛擬機器。
ms.author: mlearned
ms.manager: douge
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 07/09/2018
author: mlearned
monikerRange: vsts
ms.openlocfilehash: b05e2c2c46aa9bfa8c92d3d3c5c83d018c547b9f
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299129"
---
# <a name="tutorial--deploy-your-aspnet-app-to-azure-virtual-machines-with-the-azure-devops-project"></a>教學課程：使用 Azure DevOps Project 將 ASP.NET 應用程式部署至 Azure 虛擬機器

Azure DevOps 專案提供簡化的體驗，讓您可以自備現有程式碼和 Git 存放庫，或者從其中一個範例應用程式中進行選擇，以在 Azure 中建立持續整合 (CI) 和持續傳遞 (CD) 管線。  DevOps Project 會自動建立 Azure 資源 (例如新的 Azure 虛擬機器)、在 Azure DevOps 中建立及設定發行管線 (Azure DevOps 中包含 CI 的組建管線)，也會設定 CD 的發行管線，然後建立用於監視的 Azure Application Insights 資源。

您將：

> [!div class="checklist"]
> * 為 ASP.NET 應用程式建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Azure DevOps Services 並自動部署至 Azure
> * 設定 Azure Application Insights 監視
> * 清除資源

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 您可以透過 [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/) 取得免費的訂用帳戶。

## <a name="create-an-azure-devops-project-for-an-aspnet-app"></a>為 ASP.NET 應用程式建立 Azure DevOps Project

Azure DevOps Project 會在 Azure 中建立 CI/CD 管線。  您可以建立**新的 Azure DevOps Services** 組織或使用**現有組織**。  Azure DevOps Project 也會在您選擇的 **Azure 訂用帳戶**中建立 **Azure 資源**，例如虛擬機器。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。

1. 在左側導覽列中選擇 [+ 新增] 圖示，然後搜尋 [DevOps Project]。  選擇 [建立] 。

    ![啟動持續傳遞](_img/azure-devops-project-github/fullbrowser.png)

1. 選取 [.NET]，然後選擇 [下一步]。

1. 針對 [選擇應用程式架構] 選取 [ASP.NET]，然後選擇 [下一步]。 

1. 您在上一個步驟中選擇的應用程式架構，會指出這裡可用的 Azure 服務部署目標類型。  選取 [虛擬機器]，然後選擇 [下一步]。

## <a name="configure-azure-devops-services-and-an-azure-subscription"></a>設定 Azure DevOps Services 和 Azure 訂用帳戶

1. 建立**新的** Azure DevOps Services 組織或選擇**現有**組織。  為 Azure DevOps Project 選擇 [名稱]。  

1. 選取 [Azure 訂用帳戶服務]。  您可以選擇性地選取 [變更] 連結，然後輸入更多組態詳細資料，例如變更 Azure 資源的位置。
 
1. 為新的 Azure 虛擬機器資源輸入 [虛擬機器名稱]、[使用者名稱] 和 [密碼]，然後選擇 [完成]。

1. Azure 虛擬機器需要幾分鐘的時間才會準備就緒。  系統會在 Azure DevOps Services 組織的存放庫中設定 ASP.NET 應用程式範例、執行組建和發行，然後將應用程式部署至新建立的 Azure VM。  

    完成後，Azure 入口網站中會載入 Azure DevOps **專案儀表板**。  您可以直接從 **Azure 入口網站**中的 [所有資源] 瀏覽至 [Azure DevOps 專案儀表板]。  

    此儀表板會顯示 Azure DevOps Services **程式碼存放庫**、**Azure CI/CD 管線**，和您**在 Azure 中執行的應用程式**。    

    ![儀表板檢視](_img/azure-devops-project-vms/dashboardnopreview.png)

1. Azure DevOps Project 會自動設定 CI 組建和發行觸發程序，而將任何程式碼變更自動部署至您的存放庫。  您可以在 Azure DevOps 中進一步設定其他選項。  若要檢視執行中的應用程式，請在儀表板右側選取 [瀏覽]。
    
## <a name="examine-the-azure-devops-services-ci-pipeline"></a>檢查 Azure DevOps Services CI 管線
 
Azure DevOps Project 會在 Azure DevOps Services 組織中自動設定完整的 Azure CI/CD 管線。  您可以瀏覽及自訂管線。  請遵循下列步驟以熟悉 Azure DevOps Services 組建管線。

1. 從 [Azure DevOps Project 儀表板] **頂端**選取 [建置管線]。  此連結會開啟瀏覽器索引標籤，並開啟新專案的 Azure DevOps Services 組建管線。

1. 將滑鼠游標移至 [狀態] 欄位旁的組建管線右側。 選取隨即出現的 [省略符號]。  這個動作會開啟功能表，您可以用它來執行數個活動，例如**將新的組建排入佇列**、**暫停組建**，以及**編輯組建管線**。

1. 選取 [編輯]。

1. 在這個檢視中，**檢查組建管線的各種工作**。  組建會執行各種工作，例如從 Azure DevOps Services Git 存放庫擷取來源、還原相依性，以及發佈用來進行部署的輸出。

1. 在組建管線的頂端，選取 [組建管線名稱]。

1. 將組建管線的**名稱**變得更具有描述性。  選取 [儲存並排入佇列]，然後選取 [儲存]。

1. 在組建管線名稱下，選取 [記錄]。  您會看到組建最近變更的稽核線索。  Azure DevOps Services 會追蹤對組建管線進行的任何變更，並可讓您比較版本。

1. 選取 [觸發程序]。  Azure DevOps Project 已自動建立 CI 觸發程序，且每次對存放庫的認可都會啟動新的組建。  選擇性地選擇要在 CI 程序中包含還是排除分支。

1. 選取 [保留期]。  根據您的案例，您可以指定原則來保留或移除特定數目的組建。

## <a name="examine-the-azure-devops-services-cd-pipeline"></a>檢查 Azure DevOps Services CD 管線

Azure DevOps Project 會自動建立並設定必要的步驟，以從 Azure DevOps Services 組織部署至 Azure 訂用帳戶。  這些步驟包括設定 Azure 服務連線，以對 Azure 訂用帳戶驗證 Azure DevOps Services。  自動化也會建立 Azure DevOps Services CD 管線，且此管線會將 CD 提供給 Azure 虛擬機器。  請遵循下列步驟，對 Azure DevOps Services CD 管線進行更深入的檢查。

1. 選取 [建置及發行]，然後選擇 [版本]。  Azure DevOps Project 已建立 Azure DevOps Services 發行管線，來管理 Azure 的部署。

1. 在瀏覽器的左側，選取您發行管線旁的 [省略符號]，然後選擇 [編輯]。

1. 發行管線中包含 [管線]，會定義發行程序。  在 [成品] 下，選取 [置放]。  您在先前步驟中檢查的組建管線會產生用於成品的輸出。 

1. 在 [置放] 圖示右側，選取 [持續部署觸發程序] **圖示** (呈閃電狀)。這個發行管線具有已啟用的 CD 觸發程序。  每當有新的組建成品可用時，觸發程序就會啟動部署。  您可以選擇性地停用觸發程序，讓您的部署需要手動執行。 

1. 在瀏覽器左側選取 [工作]，然後選擇您的**環境**。  

1. 工作是您的部署程序所執行的活動，會在 [階段] 中分組。  此發行管線有兩個**階段**。  第一個階段包含 [Azure 資源群組部署] 工作，會設定要部署的 VM，並將新的 VM 新增至 [Azure DevOps 部署群組]。  Azure DevOps 中的 VM 部署群組會管理**部署目標**機器的邏輯群組。

1. 在第二個階段中會建立 [IIS Web 應用程式管理] 工作，以在 VM 上建立 IIS 網站。  此外也會建立一個 [IIS Web 應用程式部署] 工作，用以部署網站。

1. 在瀏覽器右側，選取 [檢視版本]。  此檢視會顯示版本的歷程記錄。

1. 選取您其中一個版本旁的 [省略符號]，然後選擇 [開啟]。  您可從這個檢視中瀏覽數個功能表，例如**版本摘要**、**相關聯的工作項目**及**測試**。

1. 選取 [認可]。  此檢視會顯示與特定部署相關聯的程式碼認可。 請比較版本，以查看部署的認可差異。

1. 選取 [記錄]。  記錄包含關於部署程序的實用資訊。  您可以在部署期間和部署之後加以檢視。

## <a name="commit-changes-to-azure-devops-services-and-automatically-deploy-to-azure"></a>將變更認可至 Azure DevOps Services 並自動部署至 Azure 

您現在已準備好要與小組共同對應用程式執行 CI/CD 程序，以將您最新的工作自動部署至網站。  對 Azure DevOps Services Git 存放庫所做的每項變更會在 Azure DevOps Services 中啟動建置，而且會有 Azure DevOps Services CD 管線執行目標為 Azure VM 的部署。  請遵循下列步驟或使用其他技術，將變更認可至您的存放庫。  程式碼變更會起始 CI/CD 程序，並自動將您的新變更部署至 Azure VM 上的 IIS 網站。

1. 從 Azure DevOps Services 功能表中選取 [程式碼]，並瀏覽至存放庫。

1. 瀏覽至 **Views\Home** 目錄，選取 **Index.cshtml** 檔案旁的**省略符號**，然後選擇 [編輯]。

1. 對檔案進行變更，例如，變更其中一個 **div 標記**內的某些文字。  在右上方，選取 [認可]。  再次選取 [認可] 以推送您的變更。 

1. 不久之後，**Azure DevOps Services 中會啟動一個組建**，然後會執行發行以部署變更。  使用 DevOps Project 儀表板監視**組建狀態**，或使用 Azure DevOps Services 組織在瀏覽器中檢視。

1. 當發行完成後，請在瀏覽器中**重新整理您的應用程式**，以確認您已看到變更。

## <a name="configure-azure-application-insights-monitoring"></a>設定 Azure Application Insights 監視

Azure Application Insights 可讓您輕鬆監視應用程式的效能和使用情形。  Azure DevOps Project 會自動為應用程式設定 Application Insights 資源。  您可以視需要進一步設定各種警示和監視功能。

1. 瀏覽至 **Azure 入口網站**中的 [Azure DevOps 專案] 儀表板。  在儀表板右下方，選擇應用程式的 [Application Insights] 連結。

1. [Application Insights] 刀鋒視窗會在 Azure 入口網站中開啟。  此檢視包含您應用程式的使用情形、效能和可用性監視資訊。

    ![Application Insights](_img/azure-devops-project-github/appinsights.png) 

1. 選取 [時間範圍]，然後選擇 [過去一小時]。  選取 [更新] 以篩選結果。  此時您會看到過去 60 分鐘內的所有活動。  選取 **x** 以結束時間範圍。

1. 您可以在儀表板頂端附近找到 [警示] 和其他幾個實用的連結。  選取 [警示]，然後選取 [+ 新增計量警示]。

1. 輸入警示的 [名稱]。

1. 預設警示是**伺服器回應時間大於 1 秒**的警示。  選取 [計量] 下拉式清單，以檢查各種警示計量。  例如，您可以為 ASP.NET 應用程式設定 [ASP.NET 要求執行時間]。  您可以輕鬆地設定各種警示，以改善應用程式的監視功能。

1. 選取 [透過電子郵件擁有者、參與者和讀取者進行通知] 的核取方塊。  您也可以選擇性地執行 Azure 邏輯應用程式，以在警示引發時執行其他動作。

1. 選擇 [確定] 以建立警示。  不久之後，警示就會在儀表板上呈現為作用中狀態。  **退出** [警示] 區域，並往回瀏覽至 [Application Insights] 刀鋒視窗。

1. 選取 [可用性]，然後選取 [+ 新增測試]。 

1. 輸入 [測試名稱]，然後選擇 [建立]。  這會建立簡單的 Ping 測試，以確認應用程式的可用性。  幾分鐘後即會產生測試結果，且 Application Insights 儀表板會顯示可用性狀態。

## <a name="clean-up-resources"></a>清除資源

 > [!NOTE]
 > 下列步驟將會永久刪除資源。  請務必先仔細閱讀提示，再使用此功能。

如果您要測試，您可以清除資源，以避免產生費用。  當您不再需要時，可以使用 Azure DevOps 專案儀表板上的 [刪除] 功能，將 Azure 虛擬機器和在本教學課程中建立的相關資源刪除。  **請注意**，刪除功能會終結 Azure DevOps Project 在 Azure 和 Azure DevOps 中建立的資料，且一旦終結後即無法再擷取。

1. 從 **Azure 入口網站**，瀏覽至 **Azure DevOps Project**。
2. 在儀表板**右上方**，選取 [刪除]。  讀取提示之後，請選取 [是] 以**永久刪除**資源。

## <a name="next-steps"></a>後續步驟

您可以選擇性地修改這些組建和發行管線，以符合小組的需求。 您也可以使用此 CI/CD 模式作為其他專案的範本。  您已了解如何︰

> [!div class="checklist"]
> * 為 ASP.NET 應用程式建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Azure DevOps Services 並自動部署至 Azure
> * 設定 Azure Application Insights 監視
> * 清除資源

若要深入了解 Azure CI/CD 管線，請參閱此教學課程：

> [!div class="nextstepaction"]
> [自訂 CD 程序](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)
