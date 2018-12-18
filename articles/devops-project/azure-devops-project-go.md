---
title: 使用 Azure DevOps Project 建立適用於 Go Lang 的 CI/CD 管線 | 快速入門
description: DevOps Project 可讓您輕鬆地開始使用 Azure。 它可協助您透過幾個簡單的步驟，在 Azure 服務上啟動 Go Lang Web 應用程式。
ms.prod: devops
ms.technology: devops-cicd
services: vsts
documentationcenter: vs-devops-build
author: mlearned
manager: douge
editor: ''
ms.assetid: ''
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 07/09/2018
ms.author: mlearned
ms.custom: mvc
monikerRange: vsts
ms.openlocfilehash: 48eb91242e24d29dd8335c9565dfcaca4d920bd4
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408958"
---
# <a name="create-a-cicd-pipeline-for-go-with-the-azure-devops-project"></a>使用 Azure DevOps Project 建立適用於 Go 的 CI/CD 管線

使用 **Azure DevOps Project** 為您的 **Go** 應用程式設定持續整合 (CI) 與持續傳遞 (CD)。  Azure DevOps Project 可簡化 Azure DevOps Services 建置及發行管線的初始設定。

如果您沒有 Azure 訂用帳戶，可以透過 [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/) 取得一個免費的訂用帳戶。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

Azure DevOps Project 會在 Azure 中建立 CI/CD 管線。  您可以建立**新的 Azure DevOps Services** 組織或使用**現有組織**。  Azure DevOps Project 也會在您選擇的 **Azure 訂用帳戶**中建立 **Azure 資源**。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。

1. 在左側瀏覽列中選擇 [+ 建立資源] 圖示，然後搜尋 [DevOps Project]。  選擇 [建立]。

    ![啟動持續傳遞](_img/azure-devops-project-go/fullbrowser.png)

## <a name="select-a-sample-application-and-azure-service"></a>選取應用程式範例和 Azure 服務

1. 選取 **Go** 應用程式範例，然後選取 [下一步]。

1. **簡單的 Go 應用程式**是預設的架構。  選擇 [下一步]。 

1. 您在上一個步驟中選擇的應用程式架構，會指出可用來進行部署的 Azure 服務部署目標類型。  保留預設的 Azure 服務，然後選擇 [下一步]。

## <a name="configure-azure-devops-services-and-an-azure-subscription"></a>設定 Azure DevOps Services 與 Azure 訂用帳戶 

1. 建立**新的**免費 Azure DevOps Services 組織或選擇**現有**組織。  為您的 Azure DevOps 專案選擇 [名稱]。  選取您應用程式的 [Azure 訂用帳戶]、[位置]，然後選擇 [名稱]。  完成時，選擇 [完成]。

1. 在幾分鐘後，Azure 入口網站中便會載入 **DevOps Project 儀表板**。  系統會在您 Azure DevOps Services 組織中的存放庫內安裝一個簡易應用程式、執行建置，並將您的應用程式部署到 Azure。  此儀表板可顯示您的**程式碼存放庫**、**Azure CI/CD 管線**，與您**在 Azure 中的應用程式**。  若要檢視執行中的應用程式，請在儀表板右側選取 [瀏覽]。

    ![儀表板檢視](_img/azure-devops-project-go/dashboardnopreview.png) 

## <a name="commit-code-changes-and-execute-cicd"></a>認可程式碼變更並執行 CI/CD

Azure DevOps Project 會在您的 Azure DevOps Services 組織或 GitHub 帳戶中建立 Git 存放庫。  請遵循下列步驟來檢視存放庫，並變更您應用程式的程式碼。

1. 在 DevOps Project 儀表板的左側，選取您 [主要] 分支的連結。  此連結會開啟新建立 Git 存放庫的檢視。

1. 若要檢視存放庫複製 URL，請從瀏覽器右上方選取 [複製]。 您可以在最愛的 IDE 中複製 Git 存放庫。  在接下來的幾個步驟中，您可以使用網頁瀏覽器，直接進行和認可主要分支的程式碼變更。

1. 在瀏覽器的左側，瀏覽到 **views/index.html** 檔案。

1. 選取 [編輯]，來進行部分文字的變更。  例如，修改其中一個 div 標籤內的文字。

1. 選擇 [認可]，然後儲存您的變更。

1. 在瀏覽器中，瀏覽到 [Azure DevOps Project 儀表板]。  您現在應該會看到正在進行中的建置。  您所做的變更會透過 Azure CI/CD 管線自動建置及部署。

## <a name="examine-the-azure-cicd-pipeline"></a>檢查 Azure CI/CD 管線

Azure DevOps Project 會在您的 Azure DevOps Services 組織中自動設定完整的 Azure CI/CD 管線。  瀏覽管線，並視需要進行自訂。  依照下面的步驟來熟悉 Azure DevOps Services 建置及發行管線。

1. 從 [Azure DevOps Project] 儀表板**頂端**選取 [建置管線]。  此連結會開啟瀏覽器索引標籤，並開啟新專案的 Azure DevOps Services 建置管線。

1. 選取 [省略符號]。  這個動作會開啟功能表，您可以用它來啟動數個活動，例如將新的建置排入佇列、暫停建置，以及編輯建置管線。

1. 選取 [編輯]。

1. 在這個檢視中，**檢查建置管線的各種工作**。  建置會執行各種工作，例如從 Git 存放庫擷取來源、還原相依性，以及發佈用來進行部署的輸出。

1. 在建置管線的頂端，選取 [建置管線名稱]。

1. 將建置管線的**名稱**變更成較具描述性的名稱。  選取 [儲存並排入佇列]，然後選取 [儲存]。

1. 在建置管線名稱下，選取 [記錄]。  您會看到建置最近變更的稽核線索。  Azure DevOps Services 會追蹤對建置管線進行的任何變更，且可讓您比較版本。

1. 選取 [觸發程序]。  Azure DevOps Project 已自動建立 CI 觸發程序，且每次對存放庫的認可都會啟動新的建置。  您可以選擇性地選擇要在 CI 程序中包含還是排除分支。

1. 選取 [保留期]。  根據您的案例，您可以指定原則來保留或移除特定數目的建置。

1. 選取 [建置及發行]，然後選擇 [版本]。  Azure DevOps Project 已建立 Azure DevOps Services 發行管線，來管理 Azure 的部署。

1. 在瀏覽器的左側，選取您發行管線旁的**省略符號**，然後選擇 [編輯]。

1. 發行管線中包含定義發行程序的**管線**。  在 [成品] 下，選取 [置放]。  您在先前步驟中檢查的建置管線會產生用於成品的輸出。 

1. 在 [置放] 圖示右側，選取 [持續部署觸發程序]。  這個發行管線已啟用 CD 觸發程序，每次有新的建置成品可用時，它就會執行部署。  您可以選擇性地停用觸發程序，因此您的部署需要手動執行。 

1. 在瀏覽器左側，選取 [工作]。  工作是您部署程序所執行的活動。  在此範例中，會建立一個工作來部署到 **Azure App 服務**。

1. 在瀏覽器右側，選取 [檢視版本]。  此檢視會顯示版本的歷程記錄。

1. 選取您其中一個版本旁的 [省略符號]，然後選擇 [開啟]。  您可從這個檢視中瀏覽數個功能表，例如版本摘要、相關聯的工作項目及測試。

1. 選取 [認可]。  此檢視會顯示與特定部署相關聯的程式碼認可。 

1. 選取 [記錄]。  記錄包含關於部署程序的實用資訊。  您可以在部署期間和部署之後加以檢視。

## <a name="clean-up-resources"></a>清除資源

當您不再需要時，可以使用 Azure DevOps 專案儀表板中的 [刪除] 功能，將 Azure App 服務與在此快速入門中建立的相關資源刪除。

## <a name="next-steps"></a>後續步驟

若要深入了解如何修改建置和發行管線以符合小組的需求，請參閱此教學課程：

> [!div class="nextstepaction"]
> [自訂 CD 程序](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)
