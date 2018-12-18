---
title: 開始使用 Azure 雲端服務和 ASP.NET | Microsoft Docs
description: 了解如何使用 ASP.NET MVC 和 Azure 建立多層式應用程式。 應用程式在雲端服務中執行，具備 Web 角色和背景工作角色。 它使用 Entity Framework、SQL Database 及 Azure 儲存體佇列和 Blob。
services: cloud-services, storage
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: d7aa440d-af4a-4f80-b804-cc46178df4f9
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 05/15/2017
ms.author: jeconnoc
ms.openlocfilehash: 819a2f81ca5403a3656bf713cf0ee3ae58050a4b
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003108"
---
# <a name="get-started-with-azure-cloud-services-and-aspnet"></a>開始使用 Azure 雲端服務和 ASP.NET

## <a name="overview"></a>概觀
本教學課程示範如何建立具有 ASP.NET MVC 前端的多層式 .NET 應用程式，並將它部署到 [Azure 雲端服務](cloud-services-choose-me.md)。 應用程式會使用 [Azure SQL Database](http://msdn.microsoft.com/library/azure/ee336279)、[Azure Blob 服務](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/unstructured-blob-storage)和 [Azure 佇列服務](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/queue-centric-work-pattern)。 您可以從 MSDN Code Gallery [下載 Visual Studio 專案](http://code.msdn.microsoft.com/Simple-Azure-Cloud-Service-e01df2e4) 。

本教學課程將示範如何在本機建置並執行應用程式、如何將應用程式部署至 Azure 並且在雲端執行，以及如何從頭建置應用程式。 之後，如果需要，也可以自從頭建置應用程式開始，然後執行測試和部署步驟。

## <a name="contoso-ads-application"></a>Contoso Ads 應用程式
此應用程式是廣告看板。 使用者透過輸入文字和上傳影像來建立廣告。 他們可以看到含有縮圖影像的廣告清單，也可以在選取廣告來查看詳細資料時查看完整大小的影像。

![Ad list](./media/cloud-services-dotnet-get-started/list.png)

此應用程式會使用 [以佇列為中心的工作模式](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/queue-centric-work-pattern) ，將建立縮圖的 CPU 密集工作轉變為後端程序。

## <a name="alternative-architecture-web-apps-and-webjobs"></a>替代架構：Web Apps 和 WebJob
本教學課程示範如何在 Azure 雲端服務中執行前端與後端。 替代方式是在 [Azure Web Apps](/azure/app-service/) 中執行前端，並且將 [WebJob](http://go.microsoft.com/fwlink/?LinkId=390226) 功能用於後端。 如需使用 WebJob 的教學課程，請參閱 [開始使用 Azure WebJob SDK](https://github.com/Azure/azure-webjobs-sdk/wiki)。 如需如何選擇最符合您的案例之服務的詳細資訊，請參閱 [Azure 網站、雲端服務和虛擬機器比較](../app-service/choose-web-site-cloud-service-vm.md)。

## <a name="what-youll-learn"></a>您將學到什麼
* 如何安裝 Azure SDK 好讓電腦適合用於進行 Azure 開發。
* 如何建立含一個 ASP.NET MVC Web 角色與一個背景工作角色的 Visual Studio 雲端服務專案。
* 如何使用 Azure 儲存體模擬器在本機測試雲端服務專案。
* 如何將雲端專案發行至 Azure 雲端服務，並使用 Azure 儲存體帳戶加以測試。
* 如何上傳檔案並將檔案儲存在 Azure Blob 服務。
* 如何將 Azure 佇列服務用於層級之間的通訊。

## <a name="prerequisites"></a>必要條件
本教學課程假設您了解 [Azure 雲端服務的基本概念](cloud-services-choose-me.md)，例如「Web 角色」和「背景工作角色」術語。  同時也假設您知道如何在 Visual Studio 中使用 [ASP.NET MVC](http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started) 或 [Web Form](http://www.asp.net/web-forms/tutorials/aspnet-45/getting-started-with-aspnet-45-web-forms/introduction-and-overview) 專案。 範例應用程式使用 MVC，但大多數的教學課程內容亦適用於 Web Form。

您不需 Azure 訂用帳戶即可在本機執行應用程式，但需要訂用帳戶才能將應用程式部署至雲端。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A55E3C668)或是[申請免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A55E3C668)。

本教學課程指示同時適用以下兩個產品：

* Visual Studio 2013
* Visual Studio 2015
* Visual Studio 2017

如果您尚未安裝任一個產品，安裝 Azure SDK 時，將為您自動安裝 Visual Studio。

## <a name="application-architecture"></a>應用程式架構
本應用程式會將廣告儲存在 SQL 資料庫中，使用 Entity Framework Code First 來建立表格和存取資料。 針對每個廣告，資料庫都會儲存兩個 URL：一個用於完整大小的影像，而另一個用於縮圖。

![Ad table](./media/cloud-services-dotnet-get-started/adtable.png)

使用者上傳影像時，執行 Web 角色的前端會將影像儲存在 [Azure Blob](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/unstructured-blob-storage)，並將廣告資訊儲存在資料庫中，加上指向 Blob 的 URL。 同時會將訊息寫入 Azure 佇列。 執行背景工作角色的後端程序會定期輪詢佇列，看看是否有新訊息。 出現新訊息時，背景工作角色會建立該影像的縮圖，並更新該廣告的縮圖 URL 資料庫欄位。 下圖顯示應用程式的這些部分的互動情況。

![Contoso Ads architecture](./media/cloud-services-dotnet-get-started/apparchitecture.png)

[!INCLUDE [install-sdk](../../includes/install-sdk-2017-2015-2013.md)]

## <a name="download-and-run-the-completed-solution"></a>下載並執行已完成的方案
1. 下載並解壓縮 [已完成的方案](http://code.msdn.microsoft.com/Simple-Azure-Cloud-Service-e01df2e4)(英文)。
2. 啟動 Visual Studio。
3. 從 [檔案] 功能表中，選擇 [開啟專案]，導覽至方案的下載位置，然後開啟方案檔。
4. 按 CTRL+SHIFT+B 建置解決方案。

    根據預設，Visual Studio 會自動還原未包含在 *.zip* 檔案中的 NuGet 封裝內容。 如果套件未還原，請移至 [管理方案的 NuGet 套件] 對話方塊，然後按一下右上方的 [還原] 按鈕來手動安裝。
5. 在 [方案總管] 中，確定已選取 **ContosoAdsCloudService** 作為啟動專案。
6. 如果您使用 Visual Studio 2015 或更新版本，變更 ContosoAdsWeb 專案中的 Web.config 檔案和 ContosoAdsCloudService 專案中的 ServiceConfiguration.Local.cscfg 檔案的應用程式中的 SQL Server 連接字串。 在每個案例中，將 "(localdb)\v11.0" 變更為 "(localdb)\MSSQLLocalDB"。
7. 按 CTRL+F5 執行應用程式。

    在本機執行雲端服務專案時，Visual Studio 會自動叫用 Azure「計算模擬器」和 Azure「儲存體模擬器」。 計算模擬器會使用您電腦的資源來模擬 Web 角色和背景工作角色環境。 儲存體模擬器則使用 [SQL Server Express LocalDB](http://msdn.microsoft.com/library/hh510202.aspx) 資料庫來模擬 Azure 雲端儲存體。

    第一次執行雲端服務專案時，需要約一分鐘的時間，才能啟動模擬器。 模擬器啟動完成時，預設的瀏覽器會開啟到應用程式首頁。

    ![Contoso Ads architecture](./media/cloud-services-dotnet-get-started/home.png)
8. 按一下 [建立廣告]。
9. 輸入部分測試資料，選取要上傳的 *.jpg* 影像，然後按一下 [ **建立**]。

    ![Create page](./media/cloud-services-dotnet-get-started/create.png)

    應用程式會進入索引頁面，但不會顯示新廣告的縮圖，因為處理尚未發生。
10. 稍候片刻，然後重新整理索引頁面以查看縮圖。

     ![索引頁面](./media/cloud-services-dotnet-get-started/list.png)
11. 按一下廣告的 [詳細資料]  以查看完整大小的影像。

     ![Details page](./media/cloud-services-dotnet-get-started/details.png)

您剛剛完全是在本機電腦上執行應用程式，並未連線至雲端。 儲存體模擬器會將佇列和 Blob 資料儲存在 SQL Server Express LocalDB 資料庫，而應用程式會將廣告資料儲存在另一個 LocalDB 資料庫。 Entity Framework Code First 會在 Web 應用程式第一次嘗試存取時自動建立廣告資料庫。

在下一節中，您會將方案設定為在雲端中執行時，將 Azure 雲端資源用於佇列、Blob 和應用程式資料庫。 您也可以改用雲端儲存體和資料庫資源繼續在本機執行。 這只是設定連接字串的問題而已，您將看到作法。

## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure
您將執行下列步驟，在雲端中執行應用程式：

* 建立 Azure 雲端服務。
* 建立 Azure SQL Database。
* 建立 Azure 儲存體帳戶。
* 將方案設定為在 Azure 中執行時使用 Azure SQL 資料庫。
* 將方案設定為在 Azure 中執行時使用 Azure 儲存體帳戶。
* 將專案部署至 Azure 雲端服務。

### <a name="create-an-azure-cloud-service"></a>建立 Azure 雲端服務
Azure 雲端服務是應用程式將執行所在的環境。

1. 在瀏覽器中，開啟 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [建立資源] > [計算] > [雲端服務]。

3. 在 [DNS 名稱] 輸入方塊中，輸入雲端服務的 URL 前置詞。

    URL 必須是唯一的。  如果您選擇的前置詞已在使用中，您會收到錯誤訊息。
4. 指定服務的新資源群組。 按一下 [新建]，然後在 [資源群組] 輸入方塊中輸入名稱，例如 CS_contososadsRG。

5. 選擇您要部署應用程式的區域。

    此欄位會指定將託管您的雲端服務的資料中心。 針對生產應用程式，您應該選擇距離客戶最近的區域。 針對此教學課程，請選擇離您最近的區域。
5. 按一下頁面底部的 [新增] 。

    在下面的影像中，已建立 URL 為 CSvccontosoads.cloudapp.net 的雲端服務。

    ![New Cloud Service](./media/cloud-services-dotnet-get-started/newcs.png)

### <a name="create-an-azure-sql-database"></a>建立 Azure SQL 資料庫
應用程式在雲端中執行時，將會使用雲端式資料庫。

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [建立資源] > [資料庫] > [SQL Database]。
2. 在 [ **資料庫名稱** ] 方塊中，輸入 *contosoads*。
3. 在 [資源群組] 中，按一下 [使用現有的] 並選取用於雲端服務的資源群組。
4. 在下面的影像中，按一下 [伺服器 - 進行必要設定] 和 [建立新伺服器]。

    ![透過通道連至資料庫伺服器](./media/cloud-services-dotnet-get-started/newdb.png)

    或者，如果您的訂用帳戶已有伺服器，您可以從下拉式清單選取該伺服器。
5. 在 [伺服器名稱] 方塊中，輸入 csvccontosodbserver。

6. 輸入系統管理員的 [登入名稱] 和 [密碼]。

    如果您選取了 [建立新伺服器]，則不要在此輸入現有的名稱和密碼。 而是輸入新的名稱和密碼；您現在定義的名稱和密碼將供未來存取資料庫時使用。 如果選取先前建立的伺服器，系統會提示您提供先前建立之管理使用者帳戶的密碼。
7. 選取您為雲端服務選取的相同 [位置]。

    當雲端服務與資料庫位於不同的資料中心 (不同的區域) 時，延遲時間會增加，而且您將由於使用資料中心外的頻寬而付費。 資料中心內的頻寬則是免費的。
8. 勾選 [允許 Azure 服務存取 SQL Server]。
9. 對新的伺服器按一下 [選取]。

    ![新的 SQL Database 伺服器](./media/cloud-services-dotnet-get-started/newdbserver.png)
10. 按一下頁面底部的 [新增] 。

### <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶
Azure 儲存體帳戶可提供在雲端中儲存佇列和 Blob 資料的資源。

在真實世界應用程式中，您一般會為應用程式資料與記錄資料建立不同的帳戶，以及為測試資料與生產資料建立不同的帳戶。 在本教學課程中，您只會使用一個帳戶。

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [建立資源] > [儲存體] > [儲存體帳戶 - Blob、檔案、資料表、佇列]。
2. 在 [名稱] 方塊中，輸入 URL 前置詞。

    此前置詞加上您在方塊下面看到的文字，就是儲存體帳戶的唯一 URL。 如果您輸入的前置詞已有人使用，將必須選擇不同的前置詞。
3. 將 [部署模型] 設定為 [傳統]。

4. 將 [複寫] 下拉式清單設定為 [本機備援儲存體]。

    對儲存體帳戶啟用異地複寫時，儲存的內容會複寫至次要資料中心，以便能在主要位置發生重大災害時容錯移轉。 地理區域複寫會引發額外成本。 對於測試和開發帳戶，您通常不會想要付費使用地理區域複寫功能。 如需詳細資訊，請參閱 [建立、管理或刪除儲存體帳戶](../storage/common/storage-create-storage-account.md)。

5. 在 [資源群組] 中，按一下 [使用現有的] 並選取用於雲端服務的資源群組。
6. 將 [位置] 下拉式清單設為與您為雲端服務所選的相同區域。

    當雲端服務與儲存體帳戶位於不同的資料中心 (不同的區域) 時，延遲時間會增加，而且您將由於使用資料中心外的頻寬而付費。 資料中心內的頻寬則是免費的。

    Azure 同質群組提供一種機制，可將資料中心內的資源之間的距離縮至最短，因而可以減少延遲時間。 本教學課程不會使用同質群組。 如需詳細資訊，請參閱 [如何在 Azure 中建立同質群組](https://msdn.microsoft.com/library/azure/gg715317.aspx)(英文)。
7. 按一下頁面底部的 [新增] 。

    ![New storage account](./media/cloud-services-dotnet-get-started/newstorage.png)

    在影像中，已建立 URL 為 `csvccontosoads.core.windows.net`的儲存體帳戶。

### <a name="configure-the-solution-to-use-your-azure-sql-database-when-it-runs-in-azure"></a>將方案設定為在 Azure 中執行時使用 Azure SQL 資料庫
每個 Web 專案和背景工作角色專案都有自己的資料庫連接字串，在 Azure 中執行應用程式時，每個都必須指向 Azure SQL 資料庫。

您將對 Web 角色使用 [Web.config 轉換](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations) ，以及對背景工作角色使用雲端服務環境設定。

> [!NOTE]
> 在本節與下一節中，您將把認證儲存在專案檔中。 [請勿將敏感性資料儲存在公用原始程式碼存放庫](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#secrets)。
>
>

1. 在 ContosoAdsWeb 專案中，開啟應用程式 *Web.config* 檔案的 *Web.Release.config* 轉換檔、刪除包含 `<connectionStrings>` 元素的註解區塊，並將下列程式碼貼上至其位置。

    ```xml
    <connectionStrings>
        <add name="ContosoAdsContext" connectionString="{connectionstring}"
        providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
    </connectionStrings>
    ```

    將檔案保持開啟以供編輯。
2. 在 [Azure 入口網站](https://portal.azure.com)中，依序按一下左側窗格中的 [SQL Database]、您在本教學課程中建立的資料庫、[顯示連接字串]。

    ![Show connection strings](./media/cloud-services-dotnet-get-started/showcs.png)

    入口網站會顯示連接字串，以及密碼的預留位置。

    ![連接字串](./media/cloud-services-dotnet-get-started/connstrings.png)
3. 在 *Web.Release.config* 轉換檔中，刪除 `{connectionstring}`，並將 Azure 入口網站中的 ADO.NET 連接字串貼至其位置。
4. 將您貼入 *Web.Release.config* 轉換檔的連接字串中的 `{your_password_here}` 替換成您為新的 SQL 資料庫建立的密碼。
5. 儲存檔案。  
6. 選取並複製連接字串 (不含含括的引號)，在以下設定背景工作角色專案的步驟中將會使用該資訊。
7. 在 [方案總管] 中，於雲端服務中的 [角色] 下，以滑鼠右鍵按一下 **ContosoAdsWorker**，然後按一下 [屬性]。

    ![Role properties](./media/cloud-services-dotnet-get-started/rolepropertiesworker.png)
8. 按一下 [設定]  索引標籤。
9. 將 [服務組態] 變更為 [雲端]。
10. 針對  設定選取 [值] 欄位，`ContosoAdsDbConnectionString`然後將您從教學課程前一節複製的連接字串貼上。

     ![Database connection string for worker role](./media/cloud-services-dotnet-get-started/workerdbcs.png)
11. 儲存您的變更。  

### <a name="configure-the-solution-to-use-your-azure-storage-account-when-it-runs-in-azure"></a>將方案設定為在 Azure 中執行時使用 Azure 儲存體帳戶
Web 角色專案和背景工作角色專案的 Azure 儲存體帳戶連接字串，會儲存在雲端服務專案的環境設定中。 當應用程式在本機和在雲端執行時，每個專案都有要使用的一組個別設定。 您將為 Web 和背景工作角色更新雲端環境設定。

1. 在 [方案總管] 中，於 **ContosoAdsCloudService** 專案的 [角色] 下的 **ContosoAdsWeb** 上按一下滑鼠右鍵，然後按一下 [屬性]。

    ![Role properties](./media/cloud-services-dotnet-get-started/roleproperties.png)
2. 按一下 [設定]  索引標籤。在 [服務組態] 下拉式方塊中，選取 [雲端]。

    ![Cloud configuration](./media/cloud-services-dotnet-get-started/sccloud.png)
3. 選取 **StorageConnectionString** 項目，該行右端就會出現省略符號 (**...**) 按鈕。 按一下省略符號按鈕，開啟 [ **建立儲存體帳戶連接字串** ] 對話方塊。

    ![Open Connection String Create box](./media/cloud-services-dotnet-get-started/opencscreate.png)
4. 在 [建立儲存體帳戶連接字串] 對話方塊中，按一下 [您的訂用帳戶]、選擇您稍早建立的儲存體帳戶，然後按 [確定]。 如果您尚未登入，將提示您輸入 Azure 帳戶憑證。

    ![Create Storage Connection String](./media/cloud-services-dotnet-get-started/createstoragecs.png)
5. 儲存您的變更。
6. 依照您針對 `StorageConnectionString` 連接字串執行的程序，設定 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 連接字串。

    此連接字串用於記錄。
7. 遵循您用於 **ContosoAdsWeb** 角色的相同程序，來設定 **ContosoAdsWorker** 角色的兩個連接字串。 請記得將 [服務組態] 設為 [雲端]。

您使用 Visual Studio UI 設定的角色環境設定會儲存在 ContosoAdsCloudService 專案的下列檔案中：

* *ServiceDefinition.csdef* - 定義設定名稱。
* *ServiceConfiguration.Cloud.cscfg* - 提供應用程式在雲端執行時使用的值。
* *ServiceConfiguration.Local.cscfg* - 提供應用程式在本機執行時使用的值。

例如，ServiceDefinition.csdef 包括下列定義：

```xml
<ConfigurationSettings>
    <Setting name="StorageConnectionString" />
    <Setting name="ContosoAdsDbConnectionString" />
</ConfigurationSettings>
```

此外， *ServiceConfiguration.Cloud.cscfg* 檔案也包括您在 Visual Studio 中為那些設定輸入的值。

```xml
<Role name="ContosoAdsWorker">
    <Instances count="1" />
    <ConfigurationSettings>
        <Setting name="StorageConnectionString" value="{yourconnectionstring}" />
        <Setting name="ContosoAdsDbConnectionString" value="{yourconnectionstring}" />
        <!-- other settings not shown -->

    </ConfigurationSettings>
    <!-- other settings not shown -->

</Role>
```

`<Instances>` 設定指定 Azure 將在其上執行背景工作角色程式碼的虛擬機器數目。 [後續步驟](#next-steps) 一節包含向外擴大雲端服務之詳細資訊的連結。

### <a name="deploy-the-project-to-azure"></a>將專案部署至 Azure
1. 在 [方案總管] 的 **ContosoAdsCloudService** 雲端專案上按一下滑鼠右鍵，然後選取 [發佈]。

   ![Publish menu](./media/cloud-services-dotnet-get-started/pubmenu.png)
2. 在 [發佈 Azure 應用程式] 精靈的 [登入] 步驟中，按 [下一步]。

    ![Sign in step](./media/cloud-services-dotnet-get-started/pubsignin.png)
3. 在精靈的 [設定] 步驟中，按 [下一步]。

    ![Settings step](./media/cloud-services-dotnet-get-started/pubsettings.png)

    [ **進階** ] 索引標籤中的預設設定適用於本教學課程。 如需進階索引標籤的詳細資訊，請參閱 [發行 Azure 應用程式精靈](https://docs.microsoft.com/azure/vs-azure-tools-publish-azure-application-wizard)。
4. 在 [摘要] 步驟中，按一下 [發佈]。

    ![Summary step](./media/cloud-services-dotnet-get-started/pubsummary.png)

   [ **Azure 活動記錄檔** ] 視窗即會在 Visual Studio 中開啟。
5. 按一下向右箭頭圖示以展開部署詳細資料。

    完成部署最多可能需要大約 5 分鐘或更久的時間。

    ![Azure Activity Log window](./media/cloud-services-dotnet-get-started/waal.png)
6. 當部署狀態為完成時，按一下 [Web 應用程式 URL]  來啟動應用程式。
7. 您可以透過建立、檢視和編輯一些廣告來測試應用程式，正如同您在本機執行應用程式一般。

> [!NOTE]
> 完成測試後，請刪除或停止雲端服務。 即使您未使用雲端服務，因為已為其保留虛擬機器資源，因此仍會累積費用。 如果您讓它保持執行，找到您 URL 的任何人都可以建立和檢視廣告。 在 [Azure 入口網站](https://portal.azure.com)中，移至雲端服務的 [概觀] 索引標籤，然後按一下頁面頂端的 [刪除] 按鈕。 如果您只想暫時避免他們存取網站，請改為按一下 [停止]。 在那個情況下，將持續累積費用。 當您不再需要 SQL 資料庫和儲存體帳戶時，可以遵循類似程序來加以刪除。
>
>

## <a name="create-the-application-from-scratch"></a>從頭開始建立應用程式
如果您尚未下載 [完整的應用程式](http://code.msdn.microsoft.com/Simple-Azure-Cloud-Service-e01df2e4)，請立即下載。 您會從所下載的專案複製檔案到新專案。

建立 Contoso Ads 應用程式包含下列步驟：

* 建立雲端服務 Visual Studio 方案。
* 更新和加入 NuGet 封裝。
* 設定專案參考。
* 設定連接字串。
* 加入程式碼檔案。

建立方案後，您會檢閱雲端服務專案和 Azure Blob 佇列獨特的程式碼。

### <a name="create-a-cloud-service-visual-studio-solution"></a>建立雲端服務 Visual Studio 方案
1. 在 Visual Studio 中，從 [ **New Project** from the **新增專案** ]。
2. 在 [新增專案] 對話方塊的左窗格中，展開 [Visual C#]，選擇 [雲端] 範本，然後選擇 [Azure 雲端服務] 範本。
3. 將專案和方案命名為 ContosoAdsCloudService，然後按一下 [ **確定**]。

    ![New Project](./media/cloud-services-dotnet-get-started/newproject.png)
4. 在 [新增 Azure 雲端服務] 對話方塊中，新增 Web 角色和背景工作角色。 將 Web 角色命名為 ContosoAdsWeb，並將背景工作角色命名為 ContosoAdsWorker。 (使用右窗格中的鉛筆圖示來變更角色的預設名稱。)

    ![New Cloud Service Project](./media/cloud-services-dotnet-get-started/newcsproj.png)
5. 看見 Web 角色的 [新增 ASP.NET 專案] 對話方塊時，選取 MVC 範本，然後按一下 [變更驗證]。

    ![變更驗證](./media/cloud-services-dotnet-get-started/chgauth.png)
6. 在 [變更驗證] 對話方塊中，選擇 [不需要驗證]，然後按一下 [確定]。

    ![不需要驗證](./media/cloud-services-dotnet-get-started/noauth.png)
7. 在 [新增 ASP.NET 專案] 對話方塊中，按一下 [確定]。
8. 在 [方案總管] 中，於方案 (不是其中一個專案) 上按一下滑鼠右鍵，並選擇 [新增 - 新專案]。
9. 在 [新增專案] 對話方塊中，於左窗格的 [Visual C#] 下選擇 [Windows]，然後按一下 [類別庫] 範本。  
10. 將專案命名為 *ContosoAdsCommon*，然後按一下 [確定]。

    您需要自 Web 和背景工作角色專案參考 Entity Framework 內容和資料模型。 作為替代方式，您可以在 Web 角色專案中定義 EF 相關的類別，並從背景工作角色專案參考該專案。 但在替代方法中，您的背景工作角色專案會有 Web 組件的參考，而這是專案所不需要的內容。

### <a name="update-and-add-nuget-packages"></a>更新和加入 NuGet 封裝
1. 開啟方案的 [ **管理 NuGet 封裝** ] 對話方塊。
2. 在視窗頂端，選取 [更新] 。
3. 尋找 WindowsAzure.Storage 套件，如果它不在清單中，選取它並且選取要在其中更新它的 Web 和背景工作角色專案，然後按一下 [更新]。

    儲存體用戶端程式庫的更新頻率高於 Visual Studio 專案範本的更新頻率，因此，系統會經常要求您更新新建立專案的版本。
4. 在視窗頂端，選取 [瀏覽] 。
5. 尋找 *EntityFramework* NuGet 封裝，並將它安裝在這三個專案中。
6. 尋找 *Microsoft.WindowsAzure.ConfigurationManager* NuGet 封裝，並在背景工作角色專案中安裝。

### <a name="set-project-references"></a>設定專案參考
1. 在 ContosoAdsWeb 專案中，設定 ContosoAdsCommon 專案的參考。 以滑鼠右鍵按一下 ContosoAdsWeb 專案，然後按一下 [參考]  -  [新增參考]。 在 [參考管理員] 對話方塊中，於左側窗格選取 [方案 - 專案]，選取 [ContosoAdsCommon]，然後按一下 [確定]。
2. 在 ContosoAdsWorker 專案中，設定 ContosAdsCommon 專案的參考。

    ContosoAdsCommon 將包含將由前端與後端使用的 Entity Framework 資料模型和內容類別。
3. 在 ContosoAdsWorker 專案中，設定 `System.Drawing`的參照。

    此組件是由後端用來將影像轉換為縮圖。

### <a name="configure-connection-strings"></a>設定連接字串
在本節中，您將針對本機測試用途來設定 Azure 儲存體和 SQL 連接字串。 本教學課程中稍早的部署指示，說明如何針對應用程式在雲端中執行而設定連接字串。

1. 在 ContosoAdsWeb 專案中，開啟應用程式的 Web.config 檔案，在 `connectionStrings` 元素後方插入下列`configSections`  元素。

    ```xml
    <connectionStrings>
        <add name="ContosoAdsContext" connectionString="Data Source=(localdb)\v11.0; Initial Catalog=ContosoAds; Integrated Security=True; MultipleActiveResultSets=True;" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    如果您使用 Visual Studio 2015 或更新版本，將 "v11.0" 取代為 "MSSQLLocalDB"。
2. 儲存您的變更。
3. 在 ContosoAdsCloudService 專案中，於 [角色] 下的 ContosoAdsWeb 上按一下滑鼠右鍵，然後按一下 [屬性]。

    ![Role properties](./media/cloud-services-dotnet-get-started/roleproperties.png)
4. 在 **ContosAdsWeb [Role]** 屬性視窗中，按一下 [設定] 索引標，然後按 [新增設定]。

    將 [服務組態] 保持設為 [所有組態]。
5. 新增名為 *StorageConnectionString*的設定。 將 [類型] 設為 *ConnectionString*，並將 [值] 設為 *UseDevelopmentStorage=true*。

    ![New connection string](./media/cloud-services-dotnet-get-started/scall.png)
6. 儲存您的變更。
7. 遵循相同程序，在 ContosoAdsWorker 角色屬性中加入儲存體連接字串。
8. 繼續在 [ **ContosoAdsWorker [Role]** ] 屬性視窗中，加入另一個連接字串：

   * 名稱：ContosoAdsDbConnectionString
   * 類型：字串
   * 值：貼上用於 Web 角色專案的相同連接字串。 (下列範例適用於 Visual Studio 2013。 如果您複製此範例且您是使用 Visual Studio 2015 或更新版本，請記得變更資料來源。)

       ```
       Data Source=(localdb)\v11.0; Initial Catalog=ContosoAds; Integrated Security=True; MultipleActiveResultSets=True;
       ```

### <a name="add-code-files"></a>加入程式碼檔案
在本節中，您會將檔案從所下載的方案複製到新方案。 下一節將示範和說明此程式碼的重要部分。

若要加入檔案到專案或資料夾，請以滑鼠右鍵按一下專案或資料夾，然後按一下   - **現有項目**。 選取您需要的檔案，然後按一下 [加入] 。 如果詢問您是否要取代現有的檔案，請按一下 [是] 。

1. 在 ContosoAdsCommon 專案中，刪除 *Class1.cs* 檔案，並在其位置新增來自所下載專案的 *Ad.cs* 和 *ContosoAdscontext.cs* 檔案。
2. 在 ContosoAdsWeb 專案中，從所下載的專案加入下列檔案。

   * *Global.asax.cs*。  
   * 在 Views\Shared 資料夾中：\_Layout.cshtml 檔案。
   * 在 Views\Home 資料夾中：Index.cshtml。
   * 在 Controllers 資料夾中，新增檔案︰AdController.cs。
   * 在 Views\Ad 資料夾中 (請先建立此資料夾)：五個 .cshtml 檔案。
3. 在 ContosoAdsWorker 專案中，從所下載的專案加入 *WorkerRole.cs* 。

您現在可以如本教學課程中稍早所指示般建置並執行應用程式，而應用程式將使用本機資料庫和儲存體模擬器資源。

以下小節說明與使用 Azure 環境、Blob 和佇列相關的程式碼。 本教學檔案未說明如何使用 Scaffolding 建立 MVC 控制器和檢視、如何編寫能與 SQL Server 資料庫搭配使用的 Entity Framework 程式碼，或 ASP.NET 4.5 中非同步程式設計的基本概念。 如需這些主題的相關資訊，請參閱下列資源：

* [開始使用 MVC 5](http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started)
* [開始使用 EF 6 和 MVC 5](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc)
* [.NET 4.5 中非同步程式設計的簡介](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices#async)(英文)。

### <a name="contosoadscommon---adcs"></a>ContosoAdsCommon - Ad.cs
Ad.cs 檔案可定義廣告類別列舉，以及廣告資訊的 POCO 實體類別。

```csharp
public enum Category
{
    Cars,
    [Display(Name="Real Estate")]
    RealEstate,
    [Display(Name = "Free Stuff")]
    FreeStuff
}

public class Ad
{
    public int AdId { get; set; }

    [StringLength(100)]
    public string Title { get; set; }

    public int Price { get; set; }

    [StringLength(1000)]
    [DataType(DataType.MultilineText)]
    public string Description { get; set; }

    [StringLength(1000)]
    [DisplayName("Full-size Image")]
    public string ImageURL { get; set; }

    [StringLength(1000)]
    [DisplayName("Thumbnail")]
    public string ThumbnailURL { get; set; }

    [DataType(DataType.Date)]
    [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
    public DateTime PostedDate { get; set; }

    public Category? Category { get; set; }
    [StringLength(12)]
    public string Phone { get; set; }
}
```

### <a name="contosoadscommon---contosoadscontextcs"></a>ContosoAdsCommon - ContosoAdsContext.cs
ContosoAdsContext 類別可指定廣告類別用於 DbSet 集合，Entity Framework 會儲存在 SQL 資料庫中。

```csharp
public class ContosoAdsContext : DbContext
{
    public ContosoAdsContext() : base("name=ContosoAdsContext")
    {
    }
    public ContosoAdsContext(string connString)
        : base(connString)
    {
    }
    public System.Data.Entity.DbSet<Ad> Ads { get; set; }
}
```

類別含兩個建構函式。 第一個是由 Web 專案所使用，指定儲存在 Web.config 檔案的連接字串名稱。 第二個建構函式可讓您傳入背景工作角色專案使用的實際連接字串，因為它沒有 Web.config 檔案。 您稍早已看見儲存此連接字串的位置，之後您會看到程式碼如何在具現化 DbContext 類別時擷取連接字串。

### <a name="contosoadsweb---globalasaxcs"></a>ContosoAdsWeb - Global.asax.cs
自 `Application_Start` 方法呼叫的程式碼會建立 images Blob 容器和 images 佇列 (如果尚不存在)。 這可確保每當您開始使用新儲存體帳戶或開始在新電腦上使用儲存體模擬器時，將自動建立所需的 Blob 容器和佇列。

此程式碼使用來自 *.cscfg* 檔案的儲存體連接字串來取得儲存體帳戶的存取。

```csharp
var storageAccount = CloudStorageAccount.Parse
    (RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));
```

之後會取得 *images* Blob 容器的參考、建立容器 (如果尚不存在)，並設定新容器的存取權限。 依預設，新的容器只能允許具有儲存體帳戶認證的用戶端存取 Blob。 網站需要 Blob 處於公用狀態，網站才能使用指向影像 Blob 的 URL 來顯示影像。

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var imagesBlobContainer = blobClient.GetContainerReference("images");
if (imagesBlobContainer.CreateIfNotExists())
{
    imagesBlobContainer.SetPermissions(
        new BlobContainerPermissions
        {
            PublicAccess =BlobContainerPublicAccessType.Blob
        });
}
```

類似的程式碼可取得 *images* 佇列的參考，並建立新佇列。 在此情況下，即不需要變更權限。

```csharp
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
var imagesQueue = queueClient.GetQueueReference("images");
imagesQueue.CreateIfNotExists();
```

### <a name="contosoadsweb---layoutcshtml"></a>ContosoAdsWeb - \_Layout.cshtml
*_Layout.cshtml* 檔案可設定頁首與頁尾中的應用程式名稱，並建立 "Ads" 功能表項目。

### <a name="contosoadsweb---viewshomeindexcshtml"></a>ContosoAdsWeb - Views\Home\Index.cshtml
Views\Home\Index.cshtml 檔案在首頁上顯示類別連結。 連結會將查詢字串變數中 `Category` 列舉的整數值傳遞至 [廣告索引] 頁面。

```razor
<li>@Html.ActionLink("Cars", "Index", "Ad", new { category = (int)Category.Cars }, null)</li>
<li>@Html.ActionLink("Real estate", "Index", "Ad", new { category = (int)Category.RealEstate }, null)</li>
<li>@Html.ActionLink("Free stuff", "Index", "Ad", new { category = (int)Category.FreeStuff }, null)</li>
<li>@Html.ActionLink("All", "Index", "Ad", null, null)</li>
```

### <a name="contosoadsweb---adcontrollercs"></a>ContosoAdsWeb - AdController.cs
在 AdController.cs 檔案中，建構函式會呼叫 `InitializeStorage` 方法來建立 Azure 儲存體用戶端程式庫物件，該物件可提供用於處理 Blob 和佇列的 API。

之後，程式碼可取得 images Blob 容器的參考，如您稍早在 Global.asax.cs 中所見。 在執行該動作時，它會設定適用 Web 應用程式的預設 [重試原則](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling) 。 預設指數輪詢重試原則，可能會因為對暫時性的錯誤進行反覆重試，使得 Web 應用程式停止回應超過一分鐘。 此處指定的重試原則會在每次嘗試後等候 3 秒，最多嘗試 3 次。

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
blobClient.DefaultRequestOptions.RetryPolicy = new LinearRetry(TimeSpan.FromSeconds(3), 3);
imagesBlobContainer = blobClient.GetContainerReference("images");
```

類似的程式碼可取得 *images* 佇列的參考。

```csharp
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
queueClient.DefaultRequestOptions.RetryPolicy = new LinearRetry(TimeSpan.FromSeconds(3), 3);
imagesQueue = queueClient.GetQueueReference("images");
```

多數的控制器程式碼通常用於使用 DbContext 類別來處理 Entity Framework 資料模型。 例外狀況為 HttpPost `Create` 方法，它會上傳檔案，並將檔案儲存在 Blob 儲存體。 模型繫結器可為方法提供 [HttpPostedFileBase](http://msdn.microsoft.com/library/system.web.httppostedfilebase.aspx) 物件。

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<ActionResult> Create(
    [Bind(Include = "Title,Price,Description,Category,Phone")] Ad ad,
    HttpPostedFileBase imageFile)
```

如果使用者選取要上傳的檔案，程式碼會上傳檔案、將檔案儲存在 Blob，並以指向 Blob 的 URL 更新廣告資料庫記錄。

```csharp
if (imageFile != null && imageFile.ContentLength != 0)
{
    blob = await UploadAndSaveBlobAsync(imageFile);
    ad.ImageURL = blob.Uri.ToString();
}
```

執行上傳的程式碼位於 `UploadAndSaveBlobAsync` 方法。 它會為 Blob 建立 GUID 名稱、上傳並儲存檔案，然後傳回參考至儲存的 Blob。

```csharp
private async Task<CloudBlockBlob> UploadAndSaveBlobAsync(HttpPostedFileBase imageFile)
{
    string blobName = Guid.NewGuid().ToString() + Path.GetExtension(imageFile.FileName);
    CloudBlockBlob imageBlob = imagesBlobContainer.GetBlockBlobReference(blobName);
    using (var fileStream = imageFile.InputStream)
    {
        await imageBlob.UploadFromStreamAsync(fileStream);
    }
    return imageBlob;
}
```

HttpPost `Create` 方法上傳 Blob 並更新資料庫後，它會建立佇列訊息通知後端程序，有一個影像已準備好轉換成縮圖。

```csharp
string queueMessageString = ad.AdId.ToString();
var queueMessage = new CloudQueueMessage(queueMessageString);
await queue.AddMessageAsync(queueMessage);
```

HttpPost `Edit` 方法的程式碼也是類似的，不同的是如果使用者選取新的影像檔案，就必須刪除現有的所有 Blob。

```csharp
if (imageFile != null && imageFile.ContentLength != 0)
{
    await DeleteAdBlobsAsync(ad);
    imageBlob = await UploadAndSaveBlobAsync(imageFile);
    ad.ImageURL = imageBlob.Uri.ToString();
}
```

下一個範例顯示當您刪除廣告時會刪除 Blob 的程式碼：

```csharp
private async Task DeleteAdBlobsAsync(Ad ad)
{
    if (!string.IsNullOrWhiteSpace(ad.ImageURL))
    {
        Uri blobUri = new Uri(ad.ImageURL);
        await DeleteAdBlobAsync(blobUri);
    }
    if (!string.IsNullOrWhiteSpace(ad.ThumbnailURL))
    {
        Uri blobUri = new Uri(ad.ThumbnailURL);
        await DeleteAdBlobAsync(blobUri);
    }
}
private static async Task DeleteAdBlobAsync(Uri blobUri)
{
    string blobName = blobUri.Segments[blobUri.Segments.Length - 1];
    CloudBlockBlob blobToDelete = imagesBlobContainer.GetBlockBlobReference(blobName);
    await blobToDelete.DeleteAsync();
}
```

### <a name="contosoadsweb---viewsadindexcshtml-and-detailscshtml"></a>ContosoAdsWeb - Views\Ad\Index.cshtml 和 Details.cshtml
*Index.cshtml* 檔案會顯示縮圖與其他廣告資料。

```razor
<img src="@Html.Raw(item.ThumbnailURL)" />
```

*Details.cshtml* 檔案會顯示完整大小的影像。

```razor
<img src="@Html.Raw(Model.ImageURL)" />
```

### <a name="contosoadsweb---viewsadcreatecshtml-and-editcshtml"></a>ContosoAdsWeb - Views\Ad\Create.cshtml 和 Edit.cshtml
Create.cshtml 和 Edit.cshtml 檔案可指定表單編碼，供控制器取得 `HttpPostedFileBase` 物件。

```razor
@using (Html.BeginForm("Create", "Ad", FormMethod.Post, new { enctype = "multipart/form-data" }))
```

`<input>` 元素告知瀏覽器提供檔案選取對話方塊。

```razor
<input type="file" name="imageFile" accept="image/*" class="form-control fileupload" />
```

### <a name="contosoadsworker---workerrolecs---onstart-method"></a>ContosoAdsWorker - WorkerRole.cs - OnStart 方法
Azure 背景工作角色環境會在背景工作角色開始時呼叫 `WorkerRole` 類別中的 `OnStart` 方法，並且會在 `OnStart` 方法執行結束時呼叫 `Run` 方法。

`OnStart` 方法會從 *.cscfg* 檔案取得資料庫連接字串，傳遞至 Entity Framework DbContext 類別。 預設會使用 SQLClient 提供者，使得您不需指定提供者。

```csharp
var dbConnString = CloudConfigurationManager.GetSetting("ContosoAdsDbConnectionString");
db = new ContosoAdsContext(dbConnString);
```

之後，方法會取得儲存體帳戶的參考，並建立 Blob 容器和佇列 (如果尚不存在)。 該動作的程式碼與 Web 角色 `Application_Start` 方法中的程式碼類似。

### <a name="contosoadsworker---workerrolecs---run-method"></a>ContosoAdsWorker - WorkerRole.cs - Run 方法
`Run` 方法完成其初始化工作時，系統會呼叫`OnStart` 方法。 此方法會執行無限迴圈，監看是否有新的佇列訊息，並在訊息抵達時加以處理。

```csharp
public override void Run()
{
    CloudQueueMessage msg = null;

    while (true)
    {
        try
        {
            msg = this.imagesQueue.GetMessage();
            if (msg != null)
            {
                ProcessQueueMessage(msg);
            }
            else
            {
                System.Threading.Thread.Sleep(1000);
            }
        }
        catch (StorageException e)
        {
            if (msg != null && msg.DequeueCount > 5)
            {
                this.imagesQueue.DeleteMessage(msg);
            }
            System.Threading.Thread.Sleep(5000);
        }
    }
}
```

在每次迴圈的反覆運算之後，如果沒有找到佇列訊息，程式會休息一秒。 如此可避免背景工作角色產生過度的 CPU 時間和儲存體交易成本。 Microsoft 客戶諮詢小組曾經遇過一個案例是，開發人員忘記納入這一點便部署到生產環境，之後便離開去度假。 當他回到工作崗位時，因為他疏忽所造成的代價比度假費用還高。

有時佇列訊息的內容會造成處理錯誤。 這稱為 *有害訊息*，而如果您只是記錄錯誤並重新啟動迴圈，可能會不斷的嘗試處理該訊息。  因此，Catch 區塊包含 If 陳述式，檢查以查看應用程式已嘗試處理目前訊息的次數，而如果已超過 5 次，即會從佇列中刪除訊息。

`ProcessQueueMessage` 。

```csharp
private void ProcessQueueMessage(CloudQueueMessage msg)
{
    var adId = int.Parse(msg.AsString);
    Ad ad = db.Ads.Find(adId);
    if (ad == null)
    {
        throw new Exception(String.Format("AdId {0} not found, can't create thumbnail", adId.ToString()));
    }

    CloudBlockBlob inputBlob = this.imagesBlobContainer.GetBlockBlobReference(ad.ImageURL);

    string thumbnailName = Path.GetFileNameWithoutExtension(inputBlob.Name) + "thumb.jpg";
    CloudBlockBlob outputBlob = this.imagesBlobContainer.GetBlockBlobReference(thumbnailName);

    using (Stream input = inputBlob.OpenRead())
    using (Stream output = outputBlob.OpenWrite())
    {
        ConvertImageToThumbnailJPG(input, output);
        outputBlob.Properties.ContentType = "image/jpeg";
    }

    ad.ThumbnailURL = outputBlob.Uri.ToString();
    db.SaveChanges();

    this.imagesQueue.DeleteMessage(msg);
}
```

此程式碼會讀取資料庫以取得影像 URL、將影像轉換為縮圖、將縮圖儲存在 Blob 中，以縮圖 Blob URL 更新資料庫，並刪除佇列訊息。

> [!NOTE]
> 為求簡化，`ConvertImageToThumbnailJPG` 方法中的程式碼會使用 SSystem.Drawing namespace 中的類別。 不過，此命名空間中類別的設計原意是要與 Windows Form 搭配使用。 不支援將它們用於 Windows 或 ASP.NET 服務。 如需映像處理選項的詳細資訊，請參閱[動態映像產生](http://www.hanselman.com/blog/BackToBasicsDynamicImageGenerationASPNETControllersRoutingIHttpHandlersAndRunAllManagedModulesForAllRequests.aspx)和[深入調整映像大小](http://www.hanselminutes.com/313/deep-inside-image-resizing-and-scaling-with-aspnet-and-iis-with-imageresizingnet-author-na)。
>
>

## <a name="troubleshooting"></a>疑難排解
當您遵循本教學課程中的指示進行時，若有任何項目無法運作，以下是一些常見錯誤，以及如何解決的相關資訊。

### <a name="serviceruntimeroleenvironmentexception"></a>ServiceRuntime.RoleEnvironmentException
當您在 Azure 中執行應用程式，或使用 Azure 計算模擬器在本機執行應用程式時，Azure 會提供 `RoleEnvironment` 物件。  如果您在本機執行時遇到此錯誤，請確定您已將 ContosoAdsCloudService 專案設為啟動專案。 如此可將專案設定為使用 Azure 計算模擬器執行。

應用程式使用 Azure RoleEnvironment 的一個用途是，取得儲存在 *.cscfg* 檔案中的連接字串值，因此，此例外狀況的另一個原因為遺漏連接字串。 請確定您在 ContosoAdsWeb 專案中同時為雲端和本機組態建立 StorageConnectionString 設定，也在 ContosoAdsWorker 專案中為這兩個組態建立連接字串。 如果您在整個方案中執行 [全部尋找]  ，搜尋 StorageConnectionString，您應該會看見它在 6 個檔案中出現 9 次。

### <a name="cannot-override-to-port-xxx-new-port-below-minimum-allowed-value-8080-for-protocol-http"></a>無法覆寫連接埠 xxx。 低於最小值的新連接埠，對通訊協定 http 允許的值為 8080
請嘗試變更 Web 專案使用的連接埠號碼。 以滑鼠右鍵按一下 ContosoAdsWeb 專案，然後按一下 [ **屬性**]。 按一下 [Web] 索引標籤，然後變更 [專案 URI] 設定中的連接埠號碼。

如需可能解決問題的另一個替代方法，請參閱下一節。

### <a name="other-errors-when-running-locally"></a>在本機執行時的其他錯誤
依預設，新的雲端服務專案會使用 Azure 計算模擬器精簡版來模擬 Azure 環境。 這是完整計算模擬器的輕量版本，在某些情況下，完整版模擬器能夠正常運作而精簡版則無法。  

若要將專案變更為使用完整版模擬器，請以滑鼠右鍵按一下 ContosoAdsCloudService 專案，然後按一下 [ **屬性**]。 在 [屬性] 視窗中，按一下 [Web] 索引標籤，然後按 [使用完整版模擬器] 選項按鈕。

為使用完整版模擬器執行應用程式，您必須使用管理員權限開啟 Visual Studio。

## <a name="next-steps"></a>後續步驟
Contoso Ads 應用程式特意保持簡潔，以做為入門的教學課程。 例如，它不會實作[相依性插入](http://www.asp.net/mvc/tutorials/hands-on-labs/aspnet-mvc-4-dependency-injection)或[存放庫和工作單位模式](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/advanced-entity-framework-scenarios-for-an-mvc-web-application#repo)、不會[使用介面來記錄](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry#log)、不會使用 [EF Code First 移轉](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/migrations-and-deployment-with-the-entity-framework-in-an-asp-net-mvc-application)來管理資料模型變更或 [EF 連線復原](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application)來管理暫時性網路錯誤等等。

以下是可示範更多真實世界程式碼編寫作法的一些雲端服務範例應用程式，依較不複雜到較複雜的順序列出：

* [PhluffyFotos](http://code.msdn.microsoft.com/PhluffyFotos-Sample-7ecffd31)。 概念類似 Contoso Ads，但實作更多功能和更多真實世界程式碼編寫實務。
* [具有表格、佇列和 Blob 的 Azure 雲端服務多層式應用程式](http://code.msdn.microsoft.com/windowsazure/Windows-Azure-Multi-Tier-eadceb36)。 介紹 Azure 儲存體資料表以及 Blob 和佇列。 根據舊版的 Azure SDK for .NET，將需要一些修改，以使用目前的版本。

如需針對雲端進行開發的一般資訊，請參閱「 [使用 Azure 建置實際的雲端應用程式](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/introduction)」。

如需 Azure 儲存體最佳作法和模式的簡介影片，請參閱 [Microsoft Azure 儲存體 - 新功能、最佳作法和模式](http://channel9.msdn.com/Events/Build/2014/3-628)。

如需詳細資訊，請參閱下列資源：

* [Azure 雲端服務第 1 部分：簡介](http://justazure.com/microsoft-azure-cloud-services-part-1-introduction/)
* [如何管理雲端服務](cloud-services-how-to-manage-portal.md)
* [Azure 儲存體](https://docs.microsoft.com/azure/storage/)
* [如何選擇雲端服務提供者](https://azure.microsoft.com/overview/choosing-a-cloud-service-provider/)
