---
title: 使用 Azure DevOps Project 將 ASP.NET 應用程式部署至 Azure SQL Database | Azure DevOps Services 教學課程
description: DevOps Project 可讓您輕鬆地開始使用 Azure。 Azure DevOps Project 可讓您以幾個快速步驟輕鬆地將 ASP.NET 部署至 Azure SQL Database。
ms.author: mlearned
ms.manager: douge
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 07/09/2018
author: mlearned
monikerRange: vsts
ms.openlocfilehash: d9c7c94e344daee5af87ce40ddf4dcb686696ded
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297328"
---
# <a name="tutorial--deploy-your-aspnet-app-and-azure-sql-database-with-the-azure-devops-project"></a>教學課程：使用 Azure DevOps Project 將 ASP.NET 應用程式部署至 Azure SQL Database

Azure DevOps 專案提供簡化的體驗，讓您可以自備現有程式碼和 Git 存放庫，或者從其中一個範例應用程式中進行選擇，以在 Azure 中建立持續整合 (CI) 和持續傳遞 (CD) 管線。  DevOps Project 會自動建立 Azure 資源 (例如 Azure SQL Database)、在 Azure DevOps 中建立及設定發行管線 (Azure DevOps 中包含 CI 的組建管線)，也會設定 CD 的發行管線，然後建立用於監視的 Azure Application Insights 資源。

您將：

> [!div class="checklist"]
> * 為 ASP.NET 應用程式和 Azure SQL Database 建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Azure DevOps Services 並自動部署至 Azure
> * 連線至 Azure SQL Server Database 
> * 清除資源

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 您可以透過 [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/) 取得免費的訂用帳戶。

## <a name="create-an-azure-devops-project-for-an-aspnet-app-and-azure-sql-database"></a>為 ASP.NET 應用程式和 Azure SQL Database 建立 Azure DevOps Project

Azure DevOps Project 會在 Azure 中建立 CI/CD 管線。  您可以建立**新的 Azure DevOps Services** 組織或使用**現有組織**。  Azure DevOps Project 也會在您選擇的 **Azure 訂用帳戶**中建立 **Azure 資源**，例如 Azure SQL Database。

1. 登入 [Microsoft Azure 入口網站](https://portal.azure.com)。

1. 在左側導覽列中選擇 [建立資源] 圖示，然後搜尋 [DevOps Project]。  選擇 [建立] 。

    ![啟動持續傳遞](_img/azure-devops-project-github/fullbrowser.png)

1. 選取 [.NET]，然後選擇 [下一步]。

1. 針對 [選擇應用程式架構]，選取 [ASP.NET]。

1. 選取 [新增資料庫]，然後選擇 [下一步]。  

1. 您在上一個步驟中選擇的應用程式架構，會指出這裡可用的 Azure 服務部署目標類型。  選取 [下一步] 。

## <a name="configure-azure-devops-services-and-an-azure-subscription"></a>設定 Azure DevOps Services 和 Azure 訂用帳戶

1. 建立**新的** Azure DevOps Services 組織或選擇**現有**組織。  為 Azure DevOps Project 選擇 [名稱]。  

1. 選取您的 **Azure 訂用帳戶**。

1. 選擇性地選取 [變更] 連結以檢視其他 Azure 組態設定，並在 [資料庫伺服器登入詳細資料] 區段中識別**使用者名稱**。  請**儲存****使用者名稱**以供本教學課程的後續步驟使用。
 
1. 如果您已執行上述步驟，請退出 Azure 組態區域，然後選擇 [完成]。  否則，請直接選取 [完成]。

1. 此程序需要幾分鐘的時間才能完成。  完成後，Azure 入口網站中會載入 Azure DevOps **Project 儀表板**。  您可以直接從 **Azure 入口網站**中的 [所有資源] 瀏覽至 [Azure DevOps 專案儀表板]。  若要檢視執行中的應用程式，請在儀表板右側選取 [瀏覽]。
    
## <a name="examine-the-azure-devops-services-ci-pipeline"></a>檢查 Azure DevOps Services CI 管線

Azure DevOps Project 會在 Azure DevOps Services 組織中自動設定完整的 Azure CI/CD 管線。  您可以瀏覽及自訂管線。  請遵循下列步驟以熟悉 Azure DevOps Services 組建管線。

1. 瀏覽至 [Azure DevOps 專案儀表板]。

1. 從 [Azure DevOps Project 儀表板] **頂端**選取 [建置管線]。  此連結會開啟瀏覽器索引標籤，並開啟新專案的 Azure DevOps Services 組建管線。

1. 將滑鼠游標移至 [狀態] 欄位旁的組建管線右側。 選取隨即出現的 [省略符號]。  這個動作會開啟功能表，您可以用它來執行數個活動，例如**將新的組建排入佇列**、**暫停組建**，以及**編輯組建管線**。

1. 選取 [編輯]。

1. 在這個檢視中，**檢查組建管線的各種工作**。  組建會執行各種工作，例如從 Azure DevOps Services Git 存放庫擷取來源、還原相依性，以及發佈用來進行部署的輸出。

1. 在組建管線的頂端，選取 [組建管線名稱]。

1. 將組建管線的**名稱**變得更具有描述性。  選取 [儲存並排入佇列]，然後選取 [儲存]。

1. 在組建管線名稱下，選取 [記錄]。  您會看到組建最近變更的稽核線索。  Azure DevOps Services 會追蹤對組建管線進行的任何變更，並可讓您比較版本。

1. 選取 [觸發程序]。  Azure DevOps Project 已自動建立 CI 觸發程序，且每次對存放庫的認可都會啟動新的組建。  您可以選擇性地選擇要在 CI 程序中包含還是排除分支。

1. 選取 [保留期]。  根據您的案例，您可以指定原則來保留或移除特定數目的組建。

## <a name="examine-the-azure-devops-services-cd-pipeline"></a>檢查 Azure DevOps Services CD 管線

Azure DevOps Project 會自動建立並設定必要的步驟，以從 Azure DevOps Services 組織部署至 Azure 訂用帳戶。  這些步驟包括設定 Azure 服務連線，以對 Azure 訂用帳戶驗證 Azure DevOps Services。  自動化也會建立 Azure DevOps Services 發行定義，且發行會將 CD 提供給 Azure。  請遵循下列步驟，對 Azure DevOps Services 發行定義進行更深入的檢查。

1. 選取 [建置及發行]，然後選擇 [版本]。  Azure DevOps Project 已建立 Azure DevOps Services 發行管線，來管理 Azure 的部署。

1. 在瀏覽器的左側，選取您發行管線旁的 [省略符號]，然後選擇 [編輯]。

1. 發行管線中包含 [管線]，會定義發行程序。  在 [成品] 下，選取 [置放]。  您在先前步驟中檢查的組建管線會產生用於成品的輸出。 

1. 在 [置放] 圖示右側，選取 [持續部署觸發程序] **圖示** (呈閃電狀)。這個發行管線具有已啟用的 CD 觸發程序。  每當有新的組建成品可用時，觸發程序就會啟動部署。  您可以選擇性地停用觸發程序，讓您的部署需要手動執行。 

1. Azure DevOps Project 設定了隨機的 SQL 密碼，並將此密碼用於發行管線。  在瀏覽器左側，選取 [變數]。 

1. **只有已變更 SQL Server 密碼時，才需要執行此步驟。**  目前有單一**密碼**變數。  在 [值] 文字方塊右側，選取 [掛鎖] 圖示。  **輸入**新密碼，然後選取 [儲存]。

1. 在瀏覽器左側選取 [工作]，然後選擇您的**環境**。  

1. 工作是您的部署程序所執行的活動，會在 [階段] 中分組。  此發行管線有單一階段。  此階段包含 [Azure App Service 部署] 和 [Azure SQL Database 部署] 工作。

1. 選取 [執行 Azure SQL] 工作，並檢查 SQL 部署所使用的各種屬性。  在 [部署套件] 下方，留意到此工作會使用 **SQL DACPAC 檔案**。

1. 在瀏覽器右側，選取 [檢視版本]。  此檢視會顯示版本的歷程記錄。

1. 選取您其中一個版本旁的 [省略符號]，然後選擇 [開啟]。  您可從這個檢視中瀏覽數個功能表，例如**版本摘要**、**相關聯的工作項目**及**測試**。

1. 選取 [認可]。  此檢視會顯示與特定部署相關聯的程式碼認可。 您可以比較版本，以查看部署的認可差異。

1. 選取 [記錄]。  記錄包含關於部署程序的實用資訊。  您可以在部署期間和部署之後加以檢視。

## <a name="commit-changes-to-azure-devops-services-and-automatically-deploy-to-azure"></a>將變更認可至 Azure DevOps Services 並自動部署至 Azure 

 > [!NOTE]
 > 下列步驟會以簡單的文字變更測試 CI/CD 管線。  您可以選擇性地對資料表進行 SQL Server 結構描述變更，以測試 SQL 部署程序。

您現在已準備好要與小組共同對應用程式執行 CI/CD 程序，以將您最新的工作自動部署至網站。  對 Azure DevOps Services Git 存放庫所做的每項變更會在 Azure DevOps Services 中啟動建置，而且會有 Azure DevOps Services CD 管線執行目標為 Azure 的部署。  請遵循下列步驟或使用其他技術，將變更認可至您的存放庫。  程式碼變更會啟動 CI/CD 程序，並自動將您的新變更部署至 Azure。

1. 從 Azure DevOps Services 功能表中選取 [程式碼]，並瀏覽至存放庫。

1. 瀏覽至 **SampleWebApplication\Views\Home** 目錄，選取 **Index.cshtml** 檔案旁的**省略符號**，然後選擇 [編輯]。

1. 對檔案進行變更，例如，變更其中一個 **div 標記**內的某些文字。  在右上方，選取 [認可]。  再次選取 [認可] 以推送您的變更。 

1. 不久之後，**Azure DevOps Services 中會啟動一個組建**，然後會執行發行以部署變更。  您可以使用 DevOps 專案儀表板監視**組建狀態**，或使用 Azure DevOps Services 組織在瀏覽器中檢視。

1. 當發行完成後，請在瀏覽器中**重新整理您的應用程式**，以確認您已看到變更。

## <a name="connect-to-the-azure-sql-server-database"></a>連線至 Azure SQL Server Database

您必須要有適當的權限才能連線至 Azure SQL Database。

1. 在 Azure DevOps 專案儀表板中，選取 [SQL Database] 以瀏覽至 SQL DB 的管理頁面。
   
1. 選取 [設定伺服器防火牆]，然後選取 [+ 新增用戶端 IP]。  

1. 選取 [ **儲存**]。  您的用戶端 IP 現在已可存取 **SQL Server Azure 資源**。

1. 重新瀏覽至 [SQL Database] 刀鋒視窗。 

1. 在畫面的右側，選取 [伺服器名稱] 以瀏覽至 **SQL Server** 的組態頁面。

1. 選取 [重設密碼]，輸入 [SQL Server 管理員登入] 的密碼，然後選取 [儲存]。  請**儲存**此密碼以供本教學課程的後續步驟使用。

1. 現在，您可以選擇性地使用 **SQL Server Management Studio** 或 **Visual Studio** 之類的用戶端工具連線至 Azure SQL Server 和資料庫。  請使用 [伺服器名稱] 屬性進行連線。

   如果您在一開始設定 DevOps Project 時未變更 DB 使用者名稱，則您的使用者名稱會是電子郵件地址的本機部分。  例如，如果您的電子郵件地址 johndoe@microsoft.com，您的使用者名稱就是 johndoe。

 > [!NOTE]
 > 如果您變更 SQL 登入的密碼，則必須依照**檢查 Azure DevOps Services CD 管線**一節中的說明，變更 Azure DevOps Services 發行管線變數中的密碼

## <a name="clean-up-resources"></a>清除資源

 > [!NOTE]
 > 下列步驟將會永久刪除資源。  請務必先仔細閱讀提示，再使用此功能。

如果您要測試，您可以清除資源，以避免產生費用。  當您不再需要時，可以使用 Azure DevOps 專案儀表板上的 [刪除] 功能，將 Azure SQL Database 和在本教學課程中建立的相關資源刪除。  **請注意**，刪除功能會終結 Azure DevOps Project 在 Azure 和 Azure DevOps Services 中建立的資料，且一旦終結後即無法再擷取。

1. 從 **Azure 入口網站**，瀏覽至 **Azure DevOps Project**。
2. 在儀表板**右上方**，選取 [刪除]。  讀取提示之後，請選取 [是] 以**永久刪除**資源。

## <a name="next-steps"></a>後續步驟

您可以選擇性地修改這些組建和發行管線，以符合小組的需求。 您也可以使用此 CI/CD 模式作為其他專案的範本。  您已了解如何︰

> [!div class="checklist"]
> * 為 ASP.NET 應用程式和 Azure SQL Database 建立 Azure DevOps Project
> * 設定 Azure DevOps Services 和 Azure 訂用帳戶 
> * 檢查 Azure DevOps Services CI 管線
> * 檢查 Azure DevOps Services CD 管線
> * 將變更認可至 Azure DevOps Services 並自動部署至 Azure
> * 連線至 Azure SQL Server Database 
> * 清除資源

若要深入了解 Azure 管線，請參閱此教學課程：

> [!div class="nextstepaction"]
> [自訂 CD 程序](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)

## <a name="videos"></a>影片

> [!VIDEO https://channel9.msdn.com/Events/Build/2018/BRK3308/player]