---
title: 使用 Azure DevOps Project 將 ASP.NET Core 應用程式部署至 Azure Kubernetes Service (AKS) | Azure DevOps 教學課程
description: Azure DevOps Project 可讓您輕鬆地開始使用 Azure。 Azure DevOps Project 可讓您以幾個快速步驟輕鬆地將 ASP.NET Core 應用程式部署至 Azure Kubernetes Service (AKS)。
ms.author: mlearned
ms.manager: douge
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 07/09/2018
author: mlearned
monikerRange: vsts
ms.openlocfilehash: 55ea101b3a03fdb7fc375c4594cab36d4cd79978
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299112"
---
# <a name="tutorial--deploy-your-aspnet-core-app-to-azure-kubernetes-service-aks-with-the-azure-devops-project"></a>教學課程：使用 Azure DevOps Project 將 ASP.NET Core 應用程式部署至 Azure Kubernetes Service (AKS)

Azure DevOps 專案提供簡化的體驗，讓您可以自備現有程式碼和 Git 存放庫，或者從其中一個範例應用程式中進行選擇，以在 Azure 中建立持續整合 (CI) 和持續傳遞 (CD) 管線。  DevOps Project 會自動建立 Azure 資源 (例如 AKS)、在 Azure DevOps Services 中建立及設定發行管線 (Azure DevOps Services 中包含 CI/CD 的組建和發行管線)，然後建立用於監視的 Azure Application Insights 資源。

您將：

> [!div class="checklist"]
> * 為 ASP.NET Core 應用程式和 AKS 建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 AKS 叢集
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Git 並自動部署至 Azure
> * 清除資源

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 您可以透過 [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/) 取得免費的訂用帳戶。

## <a name="create-an-azure-devops-project-for-an-aspnet-core-app-and-aks"></a>為 ASP.NET Core 應用程式和 AKS 建立 Azure DevOps Project

Azure DevOps Project 會在 Azure 中建立 CI/CD 管線。  您可以建立**新的 Azure DevOps Services** 組織或使用**現有組織**。  Azure DevOps Project 也會在您選擇的 **Azure 訂用帳戶**中建立 **Azure 資源**，例如 AKS 叢集。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。

1. 選擇左側導覽列中的 [建立資源] 圖示，然後搜尋 [DevOps Project]。  選擇 [建立] 。

    ![啟動持續傳遞](_img/azure-devops-project-aks/fullbrowser.png)

1. 選取 [.NET]，然後選擇 [下一步]。

1. 針對 [選擇應用程式架構] 選取 [ASP.NET Core]，然後選取 [下一步]。

1. 選取 [Kubernetes 服務]，然後選擇 [下一步]。  

## <a name="configure-azure-devops-services-and-an-azure-subscription"></a>設定 Azure DevOps Services 和 Azure 訂用帳戶

1. 建立**新的** Azure DevOps 組織或選擇**現有**組織。  選擇專案的**名稱**。  

1. 選取您的 **Azure 訂用帳戶**。

1. 選取 [變更] 連結以檢視其他 Azure 組態設定，並識別 [Kubernetes 叢集] 的 [節點數目]。  此處有各種選項可用來設定 Azure 服務的類型和位置。
 
1. 退出 Azure 組態區域，然後選擇 [完成]。

1. 此程序需要幾分鐘的時間才能完成。  系統會在 Azure DevOps Services 組織的 Azure Repos Git 存放庫中設定 ASP.NET Core 應用程式範例、建立 AKS 叢集、執行 CI/CD 管線，然後將應用程式部署至 Azure。  

    完成後，Azure 入口網站中會載入 Azure DevOps **Project 儀表板**。  您可以直接從 **Azure 入口網站**中的 [所有資源] 瀏覽至 [Azure DevOps 專案儀表板]。  

    此儀表板會顯示 Azure Repos **程式碼存放庫**、**Azure DevOps Services CI/CD 管線**和 **AKS 叢集**。  您可以在 Azure DevOps Services 管線中進一步設定其他 CI/CD 選項。  若要檢視執行中的應用程式，請在儀表板右側選取 [瀏覽]。

## <a name="examine-the-aks-cluster"></a>檢查 AKS 叢集

Azure DevOps Project 會自動設定 AKS 叢集。  您可以瀏覽及自訂叢集。  請遵循下列步驟以熟悉 AKS。

1. 瀏覽至 [Azure DevOps 專案儀表板]。

1. 在 DevOps Project 儀表板的右側，選取 [Kubernetes 服務]。

1. AKS 叢集的刀鋒視窗隨即開啟。  在此檢視中，您可以執行多種動作，例如**監視容器健康情況**、**搜尋記錄**，以及開啟 **Kubernetes 儀表板**。

1. 在畫面右側，選取 [檢視 Kubernetes 儀表板]。  選擇性地依照下列步驟開啟 Kubernetes 儀表板。

## <a name="examine-the-azure-devops-services-ci-pipeline"></a>檢查 Azure DevOps Services CI 管線

Azure DevOps Project 會在 Azure DevOps 組織中自動設定 Azure CI/CD 管線。  您可以瀏覽及自訂管線。  請遵循下列步驟以熟悉 Azure DevOps Services CI/CD 管線。

1. 瀏覽至 [Azure DevOps 專案儀表板]。

1. 從 [Azure DevOps Project 儀表板] **頂端**選取 [建置管線]。  此連結會開啟瀏覽器索引標籤，並開啟新專案的 Azure DevOps Services 組建管線。

1. 將滑鼠游標移至 [狀態] 欄位旁的組建管線右側。 選取隨即出現的 [省略符號]。  這個動作會開啟功能表，您可以用它來執行數個活動，例如**將新的組建排入佇列**、**暫停組建**，以及**編輯組建管線**。

1. 選取 [編輯]。

1. 在這個檢視中，**檢查各種工作**。  組建會執行各種工作，例如從 Azure DevOps Services Repos Git 存放庫擷取來源、還原相依性，以及發佈用來進行部署的輸出。

1. 在組建管線的頂端，選取 [組建管線名稱]。

1. 將組建管線的**名稱**變得更具有描述性。  選取 [儲存並排入佇列]，然後選取 [儲存]。

1. 在組建管線名稱下，選取 [記錄]。  您會看到組建最近變更的稽核線索。  Azure DevOps Services 會追蹤對組建管線進行的任何變更，並可讓您比較版本。

1. 選取 [觸發程序]。  Azure DevOps Project 已自動建立 CI 觸發程序，且每次對存放庫的認可都會啟動新的組建。  您可以選擇性地選擇要在 CI 程序中包含還是排除分支。

1. 選取 [保留期]。  根據您的案例，您可以指定原則來保留或移除特定數目的組建。

## <a name="examine-the-azure-devops-services-cd-release-pipeline"></a>檢查 Azure DevOps Services CD 發行管線

Azure DevOps Project 會自動建立並設定必要的步驟，以從 Azure DevOps Services 組織部署至 Azure 訂用帳戶。  這些步驟包括設定 Azure 服務連線，以對 Azure 訂用帳戶驗證 Azure DevOps Services。  自動化也會建立 Azure DevOps Services 發行管線，且發行管線會將 CD 提供給 Azure。  請遵循下列步驟，對 Azure DevOps Services 發行管線進行更深入的檢查。

1. 選取 [建置及發行]，然後選擇 [版本]。  Azure DevOps Project 已建立 Azure DevOps Services 發行管線，來管理 Azure 的部署。

1. 在瀏覽器的左側，選取您發行管線旁的 [省略符號]，然後選擇 [編輯]。

1. 發行管線中包含 [管線]，會定義發行程序。  在 [成品] 下，選取 [置放]。  您在先前步驟中檢查的組建管線會產生用於成品的輸出。 

1. 在 [置放] 圖示右側，選取 [持續部署觸發程序] **圖示** (呈閃電狀)。這個發行管線具有已啟用的 CD 觸發程序。  每當有新的組建成品可用時，觸發程序就會建立部署。  您可以選擇性地停用觸發程序，讓您的部署需要手動執行。 

1. 在瀏覽器右側，選取 [檢視版本]。  此檢視會顯示版本的歷程記錄。

1. 選取您其中一個版本旁的 [省略符號]，然後選擇 [開啟]。  您可從這個檢視中瀏覽數個功能表，例如**版本摘要**、**相關聯的工作項目**及**測試**。

1. 選取 [認可]。  此檢視會顯示與特定部署相關聯的程式碼認可。 您可以比較版本，以查看部署的認可差異。

1. 選取 [記錄]。  記錄包含關於部署程序的實用資訊。  您可以在部署期間和部署之後加以檢視。

## <a name="commit-changes-to-azure-devops-services-and-automatically-deploy-to-azure"></a>將變更認可至 Azure DevOps Services 並自動部署至 Azure 

 > [!NOTE]
 > 下列步驟會對您的 Web 應用程式進行簡單的文字變更，以測試 CI/CD 管線。

您現在已準備好要與小組共同對應用程式執行 CI/CD 程序，以將您最新的工作自動部署至網站。  對 Azure DevOps Services Git 存放庫所做的每項變更會在 Azure DevOps Services 中啟動組件，而且會有 CD 管線將變更部署至 Azure。  請遵循下列步驟或使用其他技術，將變更認可至您的存放庫。  例如，您可以將 Git 存放庫**複製**到您最常用的工具或 IDE 中，然後推送此存放庫的變更。

1. 從 Azure DevOps Services 功能表中選取 [程式碼] 和 [檔案]，並瀏覽至存放庫。
1. 瀏覽至 **Views\Home** 目錄，選取 **Index.cshtml** 檔案旁的**省略符號**，然後選擇 [編輯]。

1. 對檔案進行變更，例如，變更其中一個 **div 標記**內的某些文字。  在右上方，選取 [認可]。  再次選取 [認可] 以推送您的變更。 

1. 不久之後，**Azure DevOps Services 中會啟動一個組建**，然後會執行發行以部署變更。  您可以使用 DevOps 專案儀表板監視**組建狀態**，或使用 Azure DevOps Services 組織在瀏覽器中檢視。

1. 當發行完成後，請在瀏覽器中**重新整理您的應用程式**，以確認您已看到變更。

## <a name="clean-up-resources"></a>清除資源

 > [!NOTE]
 > 下列步驟將會永久刪除資源。  請務必先仔細閱讀提示，再使用此功能。

如果您要測試，您可以清除資源，以避免產生費用。  當您不再需要時，可以使用 Azure DevOps 專案儀表板上的 [刪除] 功能，將 Azure Kubernetes 叢集和在本教學課程中建立的相關資源刪除。  **請注意**，刪除功能會終結 Azure DevOps Project 在 Azure 和 Azure DevOps Services 中建立的資料，且一旦終結後即無法再擷取。

1. 從 **Azure 入口網站**，瀏覽至 **Azure DevOps Project**。
2. 在儀表板**右上方**，選取 [刪除]。  讀取提示之後，請選取 [是] 以**永久刪除**資源。

## <a name="next-steps"></a>後續步驟

您可以選擇性地修改這些組建和發行管線，以符合小組的需求。 您也可以使用此 CI/CD 模式作為其他管線的範本。  您已了解如何︰

> [!div class="checklist"]
> * 為 ASP.NET Core 應用程式和 AKS 建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 AKS 叢集
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Git 並自動部署至 Azure
> * 清除資源

若要深入了解如何使用 Kubernetes 儀表板，請參閱下列資訊：

> [!div class="nextstepaction"]
> [使用 Kubernetes 儀表板](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)
