---
title: 如何從 Visual Studio 將 Web 應用程式移轉並發佈至 Azure 雲端服務 | Microsoft Docs
description: 了解如何使用 Visual Studio 將 Web 應用程式移轉並發佈至 Azure 雲端服務
services: visual-studio-online
author: ghogen
manager: douge
ms.assetid: 9394adfd-a645-4664-9354-dd5df08e8c91
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/10/2017
ms.author: ghogen
ms.openlocfilehash: cf1e01d0165d91570c3985850b0a780b61c6b8e5
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42146179"
---
# <a name="how-to-migrate-and-publish-a-web-application-to-an-azure-cloud-service-from-visual-studio"></a>作法：從 Visual Studio 將 Web 應用程式移轉並發佈至 Azure 雲端服務

若要利用 Azure 的主機服務和延展性，您可以將 Web 應用程式移轉並部署至 Azure 雲端服務。 僅需要最低程度的變更。 本文僅涵蓋至雲端服務的部署，關於 App Service 的部分，請參閱[在 Azure App Service 中部署 Web 應用程式](app-service/app-service-deploy-local-git.md)。

> [!Important]
> 僅特定的 ASP.NET、Silverlight、WCF 及 WCF 工作流程專案支援此移轉。 ASP.NET Core 專案不支援。 請參閱[支援的專案範本](#supported-project-templates)。

## <a name="migrate-a-project-to-cloud-services"></a>將專案移轉至雲端服務

1. 以滑鼠右鍵按一下 Web 應用程式專案，並選取 [轉換] > **轉換成 Microsoft Azure 雲端服務專案**。 (請注意，如果您在解決方案中已經有 Web 角色專案，此命令不會顯示。)
1. Visual Studio 會在包含必要 Web 角色的解決方案中建立雲端服務專案。 此專案的名稱與您的應用程式專案相同，後面加上尾碼 `.Azure`。
1. 對於 MVC 2、MVC 3、MVC 4 和 Silverlight 商務應用程式所需的組件，Visual Studio 也會將 [複製到本機] 屬性設定為 true。 此屬性會將這些組件新增到用於部署的服務封裝中。

   > [!Important]
   > 如果此 Web 應用程式有其他必要組件或檔案，您必須手動設定這些檔案的屬性。 如需如何設定這些屬性的資訊，請參閱[將檔案包含在服務封裝內](#include-files-in-the-service-package)。

### <a name="errors-and-warnings"></a>錯誤和警告

出現的任何警告或發生均表示部署至 Azure 之前需要修正的問題，例如遺漏的組件。

在組建應用程式、使用計算模擬器在本機執行應用程式，或將應用程式發佈至 Azure 時，您可能會看到錯誤：「指定的路徑和 (或) 檔案名稱太長」。 此錯誤指出完整的 Azure 專案名稱長度超過 146 個字元。 若要修正該問題，請將您的解決方案移到路徑較短的不同資料夾。

如需如何將警告視為錯誤的詳細資訊，請參閱 [使用 Visual Studio 設定 Azure 雲端服務專案](vs-azure-tools-configuring-an-azure-project.md)。

### <a name="test-the-migration-locally"></a>在本機測試移轉

1. 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下新增的雲端服務專案，並按一下 [設定為啟始專案]。
1. 選取 [偵錯] > [開始偵錯] \(F5\) 啟動 Azure 偵錯環境。 此環境特別模擬各種 Azure 服務。

### <a name="use-an-azure-sql-database-for-your-application"></a>將 Azure SQL Database 用於應用程式

如果使用內部部署 SQL Server 資料庫的 Web 應用程式有連接字串，您必須改為將資料庫移轉到 Azure SQL Database，並更新連接字串。 如需此程序的指引，請參閱下列主題：

- [SQL Server 資料庫移轉至雲端 SQL Database](sql-database/sql-database-cloud-migrate.md)
- [使用 .NET (C#) 搭配 Visual Studio 連線及查詢 Azure SQL Database](sql-database/sql-database-connect-query-dotnet-visual-studio.md)。

## <a name="publish-the-application-to-azure-cloud-service"></a>將應用程式發佈至 Azure 雲端服務

1. 請依照[準備從 Visual Studio 發佈或部署 Azure 應用程式](vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio.md)所述，在您的 Azure 訂用帳戶中建立必要的雲端服務和儲存體帳戶。
1. 在 Visual Studio 中，以滑鼠右鍵按一下應用程式專案，並選取 [發佈至 Microsoft Azure...]，其與 [發佈...] 命令不同。
1. 在出現的 [發佈 Azure 應用程式] 中，使用您的 Azure 訂用帳戶使用的帳戶登入，並選取 [下一步 >]。
1. 在 [設定] > [一般設定] 索引標籤中，從 [雲端服務] 下拉式清單選取目標雲端服務，以及您選擇的環境和組態。 
1. 在 [設定] > [進階設定] 中，選取要使用的儲存體帳戶，然後選取 [下一步 >]。
1. 在 [診斷] 中，選擇是否要將資訊傳送至 Application Insights。
1. 選取 [下一步 >]檢視摘要，然後選取 [發佈] 開始部署。
1. Visual Studio 會開啟活動記錄視窗，您可以在此追蹤進度：

    ![VST_AzureActivityLog](./media/vs-azure-tools-migrate-publish-web-app-to-cloud-service/IC744149.png)

1. (選用) 若要取消部署程序，請以滑鼠右鍵按一下活動記錄檔中的該細目，然後選擇 [取消並移除]。 此命令會停止部署程序，並從 Azure 中刪除部署環境。 注意：若要在部署此部署環境後將其移除，您必須使用 [Azure 入口網站](https://portal.azure.com)。
1. (選用) 角色執行個體啟動後，Visual Studio 會自動在 [伺服器總管] > [雲端服務] 節點中顯示部署環境。 您可以從這裡檢視個別角色執行個體的狀態。
1. 若要在部署後存取應用程式，請在 [Azure 活動記錄檔] 中出現 [已完成] 狀態和 URL 時，選擇部署旁邊的箭號。 請參閱下表，以取得如何從 Azure 啟動特定類型的 Web 應用程式的詳細資料。

## <a name="using-the-compute-emulator-and-starting-application-in-azure"></a>使用計算模擬器，並且在 Azure 中啟動應用程式

選取 [偵錯] > [開始偵錯] \(F5)，即可在連接至 Visual Studio 偵錯工具的瀏覽器中啟動所有的應用程式類型。 對於 ASP.NET 空白 Web 應用程式專案，您必須先在您的應用程式中新增 `.aspx` 網頁，並將之設定為 Web 專案的開始頁面。

下表提供在 Azure 中啟動應用程式的詳細資料：

   | Web 應用程式類型 | 在 Azure 中執行 |
   | --- | --- | --- |
   | ASP.NET Web 應用程式<br/>(包括 MVC 2、MVC 3、MVC 4) | 在 **Azure 活動記錄檔**的 [部署] 索引標籤中選取該 URL。 |
   | ASP.NET 空白 Web 應用程式 | 如果您的應用程式中有預設 `.aspx` 網頁，請在 **Azure 活動記錄檔**的 [部署] 索引標籤中選取該 URL。 若要瀏覽至不同的網頁，請在瀏覽器中輸入下列格式的 URL：`<deployment_url>/<page_name>.aspx` |
   | Silverlight 應用程式<br/>Silverlight 商務應用程式<br/>Silverlight 瀏覽應用程式 | 使用下列格式的 URL 瀏覽至應用程式的特定網頁：`<deployment_url>/<page_name>.aspx` |
    WCF 服務應用程式<br/>WCF 工作流程服務應用程式 | 將 `.svc` 檔案設定做為 WCF 服務專案的起始頁。 然後瀏覽至 `<deployment_url>/<service_file>.svc` |
   | ASP.NET 動態實體<br/>ASP.NET 動態資料 Linq to SQL | 依照下一節所述，更新連接字串。 然後瀏覽至 `<deployment_url>/<page_name>.aspx`。 對於 Linq to SQL，您必須使用 Azure SQL Database。 |

## <a name="update-a-connection-string-for-aspnet-dynamic-entities"></a>更新 ASP.NET 動態實體的連接字串

1. 如先前的 (#use-an-azuresql-database-for-your-application) 所述，建立 ASP.NET 動態實體 Web 應用程式的 SQL Azure Database。
1. 從 Azure 入口網站新增此資料庫所需的資料表和欄位。
1. 使用下列格式，在 `web.config` 檔案中指定連接字串，並儲存檔案：

    ```xml
    <addname="tempdbEntities"connectionString="metadata=res://*/Model1.csdl|res://*/Model1.ssdl|res://*/Model1.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=<server name>\SQLEXPRESS;initial catalog=<database name>;integrated security=True;multipleactiveresultsets=True;App=EntityFramework&quot;"providerName="System.Data.EntityClient"/>
    ```

    使用 SQL Azure 資料庫的 ADO.NET 連接字串更新 *connectionString* 值，如下所示：

    ```xml
    XMLCopy<addname="tempdbEntities"connectionString="metadata=res://*/Model1.csdl|res://*/Model1.ssdl|res://*/Model1.msl;provider=System.Data.SqlClient;provider connection string=&quot;Server=tcp:<SQL Azure server name>.database.windows.net,1433;Database=<database name>;User ID=<user name>;Password=<password>;Trusted_Connection=False;Encrypt=True;multipleactiveresultsets=True;App=EntityFramework&quot;"providerName="System.Data.EntityClient"/>
    ```

## <a name="supported-project-templates"></a>支援的專案範本

可以移轉並發佈至雲端服務的應用程式必須使用下表的其中一個範本。 不支援 ASP.NET Core。

| 範本群組 | 專案範本 |
| --- | --- |
| Web | ASP.NET Web 應用程式 (.NET Framework) |
| Web | ASP.NET MVC 2 Web 應用程式 |
| Web | ASP.NET MVC 3 Web 應用程式 |
| Web | ASP.NET MVC4 Web 應用程式 |
| Web | ASP.NET 空白 Web 應用程式 (或網站) |
| Web | ASP.NET MVC 2 空白 Web 應用程式 |
| Web | ASP.NET 動態資料實體 Web 應用程式 |
| Web | ASP.NET 動態資料 Linq to SQL Web 應用程式 |
| Silverlight | Silverlight 應用程式 |
| Silverlight | Silverlight 商務應用程式 |
| Silverlight | Silverlight 瀏覽應用程式 |
| WCF | WCF 服務應用程式 |
| WCF | WCF 工作流程服務應用程式 |
| 工作流程 | WCF 工作流程服務應用程式 |

## <a name="next-steps"></a>後續步驟

- [準備從 Visual Studio 發佈或部署 Azure 應用程式](vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio.md)
- [設定具名的驗證認證](vs-azure-tools-setting-up-named-authentication-credentials.md)。
