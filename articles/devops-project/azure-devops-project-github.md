---
title: 使用 Azure DevOps Project 為現有的程式碼建立 CI/CD 管線 | Azure DevOps Services Tutorial
description: DevOps Project 可讓您輕鬆地開始使用 Azure。 它可協助您透過幾個簡單的步驟，使用您自己的程式碼和 GitHub 存放庫在您選擇的 Azure 服務上啟動應用程式。
services: vsts
documentationcenter: vs-devops-build
ms.author: mlearned
ms.manager: douge
editor: ''
ms.assetid: ''
ms.workload: web
ms.prod: devops
ms.technology: devops-cicd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 07/09/2018
author: mlearned
ms.custom: mvc
monikerRange: vsts
ms.openlocfilehash: 4e0e28ff9ea14e42e1df7ce35bb90e8720a0d0b6
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407292"
---
# <a name="create-a-cicd-pipeline-for-your-existing-code-with-the-azure-devops-project"></a>使用 Azure DevOps Project 為現有的程式碼建立 CI/CD 管線

Azure DevOps Project 提供簡化的體驗，讓您可以自備現有程式碼和 Git 存放庫，或者從其中一個範例應用程式中進行選擇，在 Azure 中建立持續整合 (CI) 和持續傳遞 (CD) 管線。

您將：

> [!div class="checklist"]
> * 建立 Azure DevOps Project
> * 設定對 GitHub 存放庫的存取並選擇架構
> * 設定 Azure DevOps Services 與 Azure 訂用帳戶 
> * 將變更認可至 GitHub 並自動部署至 Azure
> * 檢查 Azure DevOps Services CI/CD 管線
> * 設定 Azure Application Insights 監視

## <a name="prerequisites"></a>先決條件

* Azure 訂用帳戶。 您可以透過 [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/) 取得免費的訂用帳戶。
* 可存取其中包含 .NET、Java、PHP、Node、Python 或靜態 Web 程式碼的 GitHub 或外部 Git 存放庫。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

Azure DevOps Project 會在 Azure DevOps Services 中建立 CI/CD 管線。  您可以建立**新的 Azure DevOps Services** 組織或使用**現有組織**。  Azure DevOps Project 也會在您選擇的 **Azure 訂用帳戶**中建立 **Azure 資源**。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。

1. 在左側瀏覽列中選擇 [+ 新增] 圖示，然後搜尋 [DevOps 專案]。  選擇 [建立]。

    ![啟動持續傳遞](_img/azure-devops-project-github/fullbrowser.png)

1. 選取 [自備程式碼]。  完成時，選擇 [下一步]。

## <a name="configure-access-to-your-github-repository-and-choose-a-framework"></a>設定對 GitHub 存放庫的存取並選擇架構

1. 選取 [GitHub]。  您可以選擇性地選擇 [外部 Git 存放庫]。  選擇包含應用程式的 [存放庫] 和 [分支]。

1. 選取您的 [Web 架構]。  完成時，選擇 [下一步]。

    ![.NET Framework](_img/azure-devops-project-github/webframework.png)

1. 您在上一個步驟中選擇的應用程式架構，會指出這裡可用的 Azure 服務部署目標類型。  選取您所選擇的 [目標服務]。  完成時，選擇 [下一步]。

## <a name="configure-azure-devops-services-and-an-azure-subscription"></a>設定 Azure DevOps Services 與 Azure 訂用帳戶 

1. 建立**新的** Azure DevOps Services 組織或選擇**現有**組織。  為 Azure DevOps Project 選擇 [名稱]。  選取您應用程式的 [Azure 訂用帳戶]、[位置]，然後選擇 [名稱]。  完成時，選擇 [完成]。

1. 在幾分鐘後，Azure 入口網站中便會載入 **Azure DevOps Project 儀表板**。  系統會在您 Azure DevOps Services 組織中的存放庫中設定範例應用程式、執行建置，然後將您的應用程式部署到 Azure。  此儀表板可顯示您的 GitHub **程式碼存放庫**、**Azure DevOps Services CI/CD 管線**，與您**在 Azure 中的應用程式**。  若要檢視執行中的應用程式，請在儀表板右側選取 [瀏覽]。

    ![儀表板檢視](_img/azure-devops-project-github/dashboardnopreview.png) 
    
Azure DevOps Project 會自動設定 CI 建置和發行觸發程序。  您的程式碼會保留在 GitHub 或其他外部存放庫中。  

## <a name="commit-changes-to-github-and-automatically-deploy-to-azure"></a>將變更認可至 GitHub 並自動部署至 Azure 

您現在已準備好要與小組共同對應用程式執行 CI/CD 程序，以將您最新的工作自動部署至網站。  對 GitHub 存放庫進行的每個變更都會在 Azure DevOps 中起始一個建置，且 Azure DevOps CD 管線會對 Azure 執行部署。

1.  變更您的應用程式，並將變更**認可**至 GitHub 存放庫。
2.  不久後，Azure DevOps Services 中就會起始一個建置。  您可以使用 Azure DevOps Project 儀表板監視建置狀態，或使用 Azure DevOps Services 組織在瀏覽器中檢視。
3.  當建置完成後，請在瀏覽器中**重新整理您的應用程式**，以確認您已看到變更。

## <a name="examine-the-azure-devops-services-cicd-pipeline"></a>檢查 Azure DevOps Services CI/CD 管線

Azure DevOps Project 會在 Azure DevOps Services 組織中自動設定 Azure DevOps Services CI/CD 管線。  瀏覽管線，並視需要進行自訂。  請依照下面的步驟來熟悉 Azure DevOps Services 建置與發行管線。

1. 從 [Azure DevOps Project] 儀表板**頂端**選取 [建置管線]。  此連結會開啟瀏覽器索引標籤，並開啟新專案的 Azure DevOps Services 建置管線。

1. 將滑鼠游標移至 [狀態] 欄位旁的建置管線右側。 選取隨即出現的 [省略符號]。  這個動作會開啟功能表，您可以用它來啟動數個活動，例如將新的建置排入佇列、暫停建置，以及編輯建置管線。

1. 選取 [編輯]。

1. 從這個檢視**檢查建置管線的各種工作**。  建置會執行各種工作，例如從 Git 存放庫擷取來源、還原相依性，以及發佈用來進行部署的輸出。

1. 在建置管線的頂端，選取 [建置管線名稱]。

1. 將建置管線的**名稱**變更成較具描述性的名稱。  選取 [儲存並排入佇列]，然後選取 [儲存]。

1. 在建置管線名稱下，選取 [記錄]。  您會看到建置最近變更的稽核線索。  Azure DevOps Services 會追蹤對建置管線進行的任何變更，且可讓您比較版本。

1. 選取 [觸發程序]。  Azure DevOps Project 已自動建立 CI 觸發程序，且每次對存放庫的認可都會啟動新的建置。  您可以選擇性地選擇要在 CI 程序中包含還是排除分支。

1. 選取 [保留期]。  根據您的案例，您可以指定原則來保留或移除特定數目的建置。

1. 選取 [建置及發行]，然後選擇 [版本]。  Azure DevOps Project 已建立 Azure DevOps Services 發行管線，來管理 Azure 的部署。

1. 在瀏覽器的左側，選取您發行管線旁的**省略符號**，然後選擇 [編輯]。

1. 發行管線中包含 [管線]，會定義發行程序。  在 [成品] 下，選取 [置放]。  您在先前步驟中檢查的建置管線會產生用於成品的輸出。 

1. 在 [置放] 圖示右側，選取 [持續部署觸發程序]。  這個發行管線已啟用 CD 觸發程序，每次有新的建置成品可用時，它就會執行部署。  您可以選擇性地停用觸發程序，因此您的部署需要手動執行。 

1. 在瀏覽器左側，選取 [工作]。  工作是您部署程序所執行的活動。  在此範例中，會建立一個工作來部署到 **Azure App 服務**。

1. 在瀏覽器右側，選取 [檢視版本]。  此檢視會顯示版本的歷程記錄。

1. 選取您其中一個版本旁的 [省略符號]，然後選擇 [開啟]。  您可從這個檢視中瀏覽數個功能表，例如版本摘要、相關聯的工作項目及測試。

1. 選取 [認可]。  此檢視會顯示與特定部署相關聯的程式碼認可。 

1. 選取 [記錄]。  記錄包含關於部署程序的實用資訊。  您可以在部署期間和部署之後加以檢視。

## <a name="configure-azure-application-insights-monitoring"></a>設定 Azure Application Insights 監視

Azure Application Insights 可讓您輕鬆監視應用程式的效能和使用情形。  Azure DevOps Project 會自動為您的應用程式設定 Application Insights 資源。  您可以視需要進一步設定各種警示和監視功能。

1. 瀏覽至 Azure 入口網站中的 [Azure DevOps 專案] 儀表板。  在儀表板右下方，選擇應用程式的 [Application Insights] 連結。

1. [Application Insights] 刀鋒視窗會在 Azure 入口網站中開啟。  此檢視包含您應用程式的使用情形、效能和可用性監視資訊。

    ![Application Insights](_img/azure-devops-project-github/appinsights.png) 

1. 選取 [時間範圍]，然後選擇 [過去一小時]。  選取 [更新] 以篩選結果。  此時您會看到過去 60 分鐘內的所有活動。  選取 **x** 以結束時間範圍。

1. 選取 [警示]，然後選取 [+ 新增計量警示]。  

1. 輸入警示的 [名稱]。

1. 選取 [來源變更位置] 的下拉式清單。  選擇您的 [App Service 資源]
<!-- Could you please confirm if this should be "Source Alter on" instead of "Source Alert on"? -->。
1. 預設警示是**伺服器回應時間大於 1 秒**的警示。  選取 [計量] 下拉式清單，以檢查各種警示計量。  您可以輕鬆地設定各種警示，以改善應用程式的監視功能。

1. 選取 [透過電子郵件擁有者、參與者和讀取者進行通知] 的核取方塊。  您也可以選擇性地執行 Azure 邏輯應用程式，以在警示引發時執行其他動作。

1. 選擇 [確定] 以建立警示。  不久之後，警示就會在儀表板上呈現為作用中狀態。  **退出** [警示] 區域，並往回瀏覽至 [Application Insights] 刀鋒視窗。

1. 選取 [可用性]，然後選取 [+ 新增測試]。 

1. 輸入 [測試名稱]，然後選擇 [建立]。  此時會建立簡單的 Ping 測試，以確認應用程式的可用性。  幾分鐘後即會產生測試結果，且 Application Insights 儀表板會顯示可用性狀態。

## <a name="clean-up-resources"></a>清除資源

當您不再需要時，可以使用 Azure DevOps 專案儀表板上的 [刪除] 功能，將 Azure App 服務與在此快速入門中建立的相關資源刪除。

## <a name="next-steps"></a>後續步驟

當您在此教學課程中設定 CI/CD 程序時，會自動在您的 Azure DevOps Project 中建立建置和發行管線。 您可以修改這些建置和發行管線，以符合小組的需求。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Azure DevOps Project
> * 設定對 GitHub 存放庫的存取並選擇架構
> * 設定 Azure DevOps Services 與 Azure 訂用帳戶 
> * 將變更認可至 GitHub 並自動部署至 Azure
> * 檢查 Azure DevOps Services CI/CD 管線
> * 設定 Azure Application Insights 監視

若要深入了解 Azure DevOps Services CI/CD 管線，請參閱此教學課程：

> [!div class="nextstepaction"]
> [自訂 CD 程序](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)
