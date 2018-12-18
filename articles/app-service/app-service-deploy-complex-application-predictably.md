---
title: 透過可預測方式在 Azure 中佈建和部署微服務
description: 學習如何在 Azure App Service 中將包含微服務的應用程式佈建和部署為單一單位，並且使用 JSON 資源群組範本和 PowerShell 指令碼為可預測的方式。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: jimbe
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/06/2016
ms.author: cephalin
ms.openlocfilehash: 049f5211e800dace4b8968cd9e3db9ad968f8813
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050741"
---
# <a name="provision-and-deploy-microservices-predictably-in-azure"></a>透過可預測方式在 Azure 中佈建和部署微服務
本教學課程示範如何在 [Azure App Service](https://azure.microsoft.com/services/app-service/) 中將包含[微服務](https://en.wikipedia.org/wiki/Microservices)的應用程式佈建和部署為單一單位，並且使用 JSON 資源群組範本和 PowerShell 指令碼的可預測方式。 

佈建和部署包含高低耦合微服務的高級別應用程式時，重複性和可預測性是成功的重要關鍵。 [Azure App Service](https://azure.microsoft.com/services/app-service/) 可讓您建立微服務，其中包括 Web 應用程式、行動應用程式、API 應用程式和邏輯應用程式。 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 可讓您將所有微服務當成一個單位來進行管理，以及管理資源相依性 (例如資料庫和原始檔控制設定)。 現在，您也可以使用 JSON 範本和簡單 PowerShell 指令碼來部署這類應用程式。 

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="what-you-will-do"></a>將執行的作業
在教學課程中，您將部署的應用程式包括：

* 兩個 Web 應用程式 (即兩個微服務)
* 後端 SQL Database
* 應用程式設定、連接字串和原始檔控制
* Application insights、警示、自動調整設定

## <a name="tools-you-will-use"></a>將使用的工具
在本教學課程中，您將使用下列工具。 這不是工具的完整討論，因此將著重在端對端案例，並且只對每個工具進行簡短介紹，以及您可以在哪裡找到更多資訊。 

### <a name="azure-resource-manager-templates-json"></a>Azure 資源管理員範本 (JSON)
例如，每次在 Azure App Service 中建立 Web 應用程式時，Azure 資源管理員都會使用 JSON 範本來建立具有元件資源的整個資源群組。 來自 [Azure Marketplace](/azure/marketplace) 的複雜範本，可包括資料庫、儲存體帳戶、App Service 方案、Web 應用程式本身、警示規則、應用程式設定、自動調整設定等等，而且您可以透過 PowerShell 使用所有這類範本。 如需如何下載和使用這些範本的詳細資訊，請參閱 [搭配使用 Azure PowerShell 與 Azure 資源管理員](../powershell-azure-resource-manager.md)。

如需 Azure 資源管理員範本的詳細資訊，請參閱 [編寫 Azure 資源管理員範本](../azure-resource-manager/resource-group-authoring-templates.md)

### <a name="azure-sdk-26-for-visual-studio"></a>Azure SDK 2.6 for Visual Studio
最新 SDK 已增強 JSON 編輯器中的資源管理員範本支援。 您可以使用此項快速從頭開始建立資源群組範本，或開啟現有 JSON 範本 (例如已下載的資源庫範本) 進行修改、填入參數檔案，甚至直接從 Azure 資源群組方案部署資源群組。

如需詳細資訊，請參閱 [Azure SDK 2.6 for Visual Studio](https://azure.microsoft.com/blog/2015/04/29/announcing-the-azure-sdk-2-6-for-net/)。

### <a name="azure-powershell-080-or-later"></a>Azure PowerShell 0.8.0 或更新版本
自 0.8.0 版開始，Azure PowerShell 安裝除了 Azure 模組之外還包括 Azure 資源管理員模組。 這個新模組可讓您編寫資源群組部署的指令碼。

如需詳細資訊，請參閱 [搭配使用 Azure PowerShell 與 Azure 資源管理員](../powershell-azure-resource-manager.md)

### <a name="azure-resource-explorer"></a>Azure 資源總管
此[預覽工具](https://resources.azure.com)可讓您瀏覽您的訂用帳戶與個別資源中的所有資源群組的 JSON 定義。 在此工具中，您可以編輯資源的 JSON 定義、刪除整個階層的資源，以及建立新的資源。  此工具中立即可用的資訊對於範本撰寫十分有用，因為它會顯示您需要針對特定類型的資源所設定的屬性、正確值等。您甚至可以在 [Azure 入口網站](https://portal.azure.com/)中建立資源群組，然後在總管工具中檢查其 JSON 定義，協助您建立資源群組的樣本。

### <a name="deploy-to-azure-button"></a>部署至 Azure 按鈕
如果您使用 GitHub 進行原始檔控制，則可以將 [部署至 Azure 按鈕](https://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/) 放入 README.MD，而此檔案會啟用 Azure 的轉換金鑰部署 UI。 雖然您可以針對任何簡單 Web 應用程式執行這項作業，但是您可以將 azuredeploy.json 檔案放在儲存機制根目錄中，以擴充此項來啟用部署整個資源群組。 此 JSON 檔案包含資源群組範本，將供 [部署至 Azure] 按鈕用來建立資源群組。 如需範例，請參閱將在本教學課程中使用的 [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) 範例。

## <a name="get-the-sample-resource-group-template"></a>取得範例資源群組範本
因此，現在讓我們著手進行。

1. 巡覽至 [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) App Service 範例。
2. 在 readme.md 中，按一下 [ **部署至 Azure**]。
3. 您會進入 [deploy-to-azure](https://deploy.azure.com) 網站，並要求您輸入部署參數。 請注意，大部分的欄位都會填入儲存機制名稱以及一些隨機字串。 您可以視需要變更所有欄位，但唯一必須輸入的項目是 SQL Server 管理登入和密碼，然後按 [ **下一步**]。
   
   ![](./media/app-service-deploy-complex-application-predictably/gettemplate-1-deploybuttonui.png)
4. 接著，按一下 [ **部署** ] 啟動部署程序。 程序執行並完成之後，請按一下 http://todoappXXXX.azurewebsites.net 連結來瀏覽已部署的應用程式。 
   
   ![](./media/app-service-deploy-complex-application-predictably/gettemplate-2-deployprogress.png)
   
   在您第一次瀏覽至 UI 時，UI 會比較慢，因為應用程式剛剛啟動，但確信它是功能完整的應用程式。
5. 回到 [部署] 頁面，並按一下 [管理]  連結，以在 Azure 入口網站中查看新的應用程式。
6. 在 [ **Essentials** ] 下拉式清單中，按一下 [資源群組] 連結。 也請注意 Web 應用程式已連線至 [ **外部專案**] 下的 GitHub 儲存機制。 
   
   ![](./media/app-service-deploy-complex-application-predictably/gettemplate-3-portalresourcegroup.png)
7. 在資源群組刀鋒視窗中，請注意，資源群組中已經有兩個 Web 應用程式和一個 SQL Database。
   
   ![](./media/app-service-deploy-complex-application-predictably/gettemplate-4-portalresourcegroupclicked.png)

剛剛您在短短幾分鐘內看到的所有項目都是完整部署的雙微服務應用程式，以及所有元件、相依性、設定、資料庫和連續發行 (由 Azure 資源管理員中的自動化協調流程所設定)。 這項作業是透過兩件事完成：

* [部署至 Azure] 按鈕
* 儲存機制根目錄中的 azuredeploy.json

您可以部署這個相同的應用程式數十次、數百次或數千次，而且每次的組態都完全相同。 這種方法的重複性和可預測性可讓您輕鬆並自信地部署高延展性應用程式。

## <a name="examine-or-edit-azuredeployjson"></a>檢查 (或編輯) AZUREDEPLOY.JSON
現在，讓我們看看已如何設定 GitHub 儲存機制。 您將使用 Azure .NET SDK 中的 JSON 編輯器，因此，如果尚未安裝 [Azure .NET SDK 2.6](https://azure.microsoft.com/downloads/)，請立即安裝。

1. 使用慣用的 git 工具複製 [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) 儲存機制。 在下面的螢幕擷取畫面中，我將在 Visual Studio 2013 的 Team Explorer 中中進行這項作業。
   
   ![](./media/app-service-deploy-complex-application-predictably/examinejson-1-vsclone.png)
2. 在 Visual Studio 中，從儲存機制根目錄中開啟 azuredeploy.json。 如果您看不到 [JSON 大綱] 窗格，則需要安裝 Azure .NET SDK。
   
   ![](./media/app-service-deploy-complex-application-predictably/examinejson-2-vsjsoneditor.png)

我不打算詳述 JSON 格式，但是 [其他資源](#resources) 小節具有了解資源群組範本語言的連結。 在這裡，我只會說明可協助您開始建立專屬自訂範本以進行應用程式部署的有趣功能。

### <a name="parameters"></a>參數
查看 parameters 區段，了解大部分這些參數都是 [ **部署至 Azure** ] 按鈕提示您輸入的參數。 [ **部署至 Azure** ] 按鈕後面的網站會使用 azuredeploy.json 中所定義的參數來填入輸入 UI。 這些參數用於整個資源定義 (例如資源名稱、屬性值等)。

### <a name="resources"></a>資源
在 resources 節點中，您可以看到已定義 4 個最上層資源，包括 SQL Server 執行個體、App Service 方案和兩個 Web 應用程式。 

#### <a name="app-service-plan"></a>App Service 方案
讓我們從 JSON 中的簡單根層級資源開始。 在 [JSON 大綱] 中，按一下名為 **[hostingPlanName]** 的 App Service 方案反白顯示對應的 JSON 程式碼。 

![](./media/app-service-deploy-complex-application-predictably/examinejson-3-appserviceplan.png)

請注意， `type` 項目指定 App Service 方案 (很久以前稱為伺服器陣列) 的字串，並會使用 JSON 檔案中所定義的參數來填寫其他項目和屬性，而且此資源沒有任何巢狀資源。

> [!NOTE]
> 也請注意，`apiVersion` 的值告訴 Azure 搭配使用哪一個版本的 REST API 與 JSON 資源定義，而且可能會影響資源在 `{}` 內的格式。 
> 
> 

#### <a name="sql-server"></a>SQL Server
接下來，按一下 [JSON 大綱] 中名為 **SQLServer** 的 SQL Server 資源。

![](./media/app-service-deploy-complex-application-predictably/examinejson-4-sqlserver.png)

請注意反白顯示 JSON 程式碼的下列資訊：

* 參數的使用確保透過讓所建立的資源彼此一致的方式來命名和設定所建立的資源。
* SQLServer 資源有兩個巢狀資源，其各有不同的 `type` 值。
* `“resources”: […]` 內巢狀資源 (其中已定義資料庫和防火牆規則) 的 `dependsOn` 項目指定根層級 SQLServer 資源的資源識別碼。 這會告訴 Azure 資源管理員：「建立此資源之前，那個其他資源必須已經存在；如果那個其他資源定義在範本中，則請先建立它」。
  
  > [!NOTE]
  > 如需如何使用 `resourceId()` 函式的詳細資訊，請參閱 [Azure Resource Manager 範本函式](../azure-resource-manager/resource-group-template-functions-resource.md#resourceid)。
  > 
  > 
* `dependsOn` 項目的作用是 Azure 資源管理員可以知道可平行建立哪些資源，以及必須依序建立哪些資源。 

#### <a name="web-app"></a>Web 應用程式
現在，讓我們繼續討論更為複雜的實際 Web 應用程式本身。 按一下 [ JSON 大綱] 中的 [variables(‘apiSiteName’)] Web 應用程式，以反白顯示其 JSON 程式碼。 您會發現事情變得越來越有趣。 因此，我將逐一討論所有功能：

##### <a name="root-resource"></a>根資源
Web 應用程式與兩個不同的資源相依。 這表示只有在建立 App Service 方案和 SQL Server 執行個體之後，Azure 資源管理員才會建立 Web 應用程式。

![](./media/app-service-deploy-complex-application-predictably/examinejson-5-webapproot.png)

##### <a name="app-settings"></a>應用程式設定
應用程式設定也會定義為巢狀資源。

![](./media/app-service-deploy-complex-application-predictably/examinejson-6-webappsettings.png)

在 `config/appsettings` 的 `properties` 項目中，您會有兩個格式為 `“<name>” : “<value>”` 的應用程式設定。

* `PROJECT` 是 [KUDU 設定](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) ，告訴 Azure 部署在多專案 Visual Studio 方案中使用哪個專案。 稍後會示範如何設定原始檔控制，但是，因為 ToDoApp 程式碼是在多專案 Visual Studio 方案中，所以我們需要這項設定。
* `clientUrl` 只是應用程式程式碼所使用的應用程式設定。

##### <a name="connection-strings"></a>連接字串
連接字串也會定義為巢狀資源。

![](./media/app-service-deploy-complex-application-predictably/examinejson-7-webappconnstr.png)

在 `config/connectionstrings` 的 `properties` 項目中，每個連接字串也會定義為「名稱:值」配對，並且具有特定格式：`“<name>” : {“value”: “…”, “type”: “…”}`。 針對 `type` 項目，可能的值為 `MySql`、`SQLServer`、`SQLAzure` 和 `Custom`。

> [!TIP]
> 如需連接字串型別的限定清單，請在 Azure PowerShell 中執行下列命令：\[Enum]::GetNames("Microsoft.WindowsAzure.Commands.Utilities.Websites.Services.WebEntities.DatabaseType")
> 
> 

##### <a name="source-control"></a>原始檔控制
原始檔控制也會定義為巢狀資源。 Azure 資源管理員使用這項資源來設定持續發佈 (請參閱後面 `IsManualIntegration` 上的警告)，並且在處理 JSON 檔案期間開始自動部署應用程式程式碼。

![](./media/app-service-deploy-complex-application-predictably/examinejson-8-webappsourcecontrol.png)

`RepoUrl` 和 `branch` 應該相當具直覺式，而且應該指向 Git 儲存機制以及從發佈之分支的名稱。 同樣地，這些是透過輸入參數所定義。 

請注意，在 `dependsOn` 項目，除了 Web 應用程式資源本身之外，`sourcecontrols/web` 也與 `config/appsettings` 和 `config/connectionstrings` 相依。 原因是設定 `sourcecontrols/web` 之後，Azure 部署程序會自動嘗試部署、建置和啟動應用程式碼。 因此，插入此相依性有助於確定在執行應用程式碼之前，應用程式可以存取所需的應用程式設定和連接字串。 

> [!NOTE]
> 也請注意，`IsManualIntegration` 設為 `true`。 這個屬性是本教學課程中的必要屬性，因為您實際上未擁有 GitHub 存放庫，因此無法實際將從 [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) (也就是將自動存放庫更新推送至 Azure) 設定持續發佈的權限授與 Azure。 只有在之前已在 [Azure 入口網站](https://portal.azure.com/)中設定過擁有者的 GitHub 認證時，才能使用所指定儲存機制的預設值 `false`。 換句話說，如果您之前已使用使用者認證來設定 [Azure 入口網站](https://portal.azure.com/)中任何應用程式的 GitHub 或 BitBucket 原始檔控制，則 Azure 會記住認證，並在未來從 GitHub 或 BitBucket 部署任何應用程式時使用它們。 不過，如果您還沒有這麼做，則 Azure 資源管理員嘗試設定 Web 應用程式的原始檔控制設定時，JSON 範本的部署會失敗。因為它無法使用儲存機制擁有者的認證登入 GitHub 或 BitBucket。
> 
> 

## <a name="compare-the-json-template-with-deployed-resource-group"></a>比較具有已部署資源群組的 JSON 範本
您可以在這裡瀏覽 [Azure 入口網站](https://portal.azure.com/)中的所有 Web 應用程式刀鋒視窗，但還有另一個工具十分有用。 移至 [Azure 資源總管](https://resources.azure.com) 預覽工具，其可提供訂用帳戶中所有資源群組的 JSON 呈現，因為它們實際存在於 Azure 後端。 您也可以看到 Azure 中資源群組的 JSON 階層如何與用來建立它的範本檔案中的階層對應。

例如，移至 [Azure 資源總管](https://resources.azure.com) 工具並展開總管中的節點時，可以看到資源群組以及收集在其個別資源型別下方的根層級資源。

![](./media/app-service-deploy-complex-application-predictably/ARM-1-treeview.png)

如果您向下鑽研至 Web 應用程式，則應該可以看到與下面螢幕擷取畫面類似的 Web 應用程式組態詳細資料：

![](./media/app-service-deploy-complex-application-predictably/ARM-2-jsonview.png)

同樣地，巢狀資源的階層應該與 JSON 範本檔案中的階層極為類似，而且您應該會看到 JSON 窗格中正確反映的應用程式設定、連接字串等。 沒有這裡的設定可能表示 JSON 檔案發生問題，並可協助您疑難排解 JSON 範本檔案。

## <a name="deploy-the-resource-group-template-yourself"></a>自行部署資源群組範本
[ **部署至 Azure** ] 按鈕不錯，但是，只有在已將 azuredeploy.json 推送至 GitHub 時，它才可讓您在 azuredeploy.json 中部署資源群組範本。 Azure .NET SDK 也提供工具，讓您直接從本機電腦部署任何 JSON 範本檔案。 若要這樣做，請遵循下面的步驟：

1. 在 Visual Studio 中，按一下 [檔案] > [新增] > [專案]。
2. 按一下 [Visual C#] > [雲端] > [Azure 資源群組]，然後按一下 [確定]。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-1-vsproject.png)
3. 在 [選取 Azure 範本] 中，選取 [空白範本]，然後按一下 [確定]。
4. 將 azuredeploy.json 拖曳至新專案的 [ **樣本** ] 資料夾。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-2-copyjson.png)
5. 從 [方案總管]，開啟複製的 azuredeploy.json。
6. 按一下 [ **加入資源**]，將一些標準 Application Insight 資源加入 JSON 檔案，但此作業僅供示範之用。 如果您只想部署 JSON 檔案，請跳至部署步驟。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-3-newresource.png)
7. 選取 [Web Apps 的 Application Insights]，並確定已選取現有 App Service 方案和 Web 應用程式，然後按一下 [新增]。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-4-newappinsight.png)
   
   您現在可以看到數個新資源 (根據資源和其作用而定) 與 App Service 方案或 Web 應用程式相依。 這些資源不是透過其現有定義所啟用，而您將變更這項行為。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-5-appinsightresources.png)
8. 在 [JSON 大綱] 中，按一下 **appInsights AutoScale** 反白顯示其 JSON 程式碼。 這是您 App Service 方案的調整設定。
9. 在反白顯示的 JSON 程式碼中，找到 `location` 和 `enabled` 屬性，並如下所示設定它們。
   
   ![](./media/app-service-deploy-complex-application-predictably/deploy-6-autoscalesettings.png)
10. 在 [JSON 大綱] 中，按一下 **CPUHigh appInsights** 反白顯示其 JSON 程式碼。 這是警示。
11. 找到 `location` 和 `isEnabled` 屬性，並如下所示設定它們。 請對其他三個警示 (紫色燈泡) 執行相同的動作。
    
    ![](./media/app-service-deploy-complex-application-predictably/deploy-7-alerts.png)
12. 您現在可以開始進行部署。 以滑鼠右鍵按一下專案，然後選取  **部署** > **New 部署ment**。
    
    ![](./media/app-service-deploy-complex-application-predictably/deploy-8-newdeployment.png)
13. 登入 Azure 帳戶 (如果您尚未這樣做)。
14. 在您的訂用帳戶中選取現有的資源群組，或選取 **azuredeploy.json**，然後按一下 [編輯參數] 建立新的資源群組。
    
    ![](./media/app-service-deploy-complex-application-predictably/deploy-9-deployconfig.png)
    
    您現在可以透過好用的資料表編輯範本檔案中定義的所有參數。 定義預設值的參數已經有其預設值，而定義允許值清單的參數則會顯示為下拉式清單。
    
    ![](./media/app-service-deploy-complex-application-predictably/deploy-10-parametereditor.png)
15. 填入所有空的參數，並在 [repoUrl](https://github.com/azure-appservice-samples/ToDoApp.git) 中使用 **GitHub repo address for ToDoApp**。 然後按一下 [ **儲存**]。
    
    ![](./media/app-service-deploy-complex-application-predictably/deploy-11-parametereditorfilled.png)
    
    > [!NOTE]
    > 自動調整是**標準**層級或更高層級所提供的功能，而計劃層級警示是**基本**層級或更高層級所提供的功能，您需要將 **sku** 參數設為**標準**或**進階**，才能看到所有新的 App Insights 資源都已啟動。
    > 
    > 
16. 按一下 [ **部署**]。 如果您選取 [儲存密碼]，則會將密碼**以純文字形式**儲存至參數檔。 否則，系統會在部署過程要求您輸入資料庫密碼。

這樣就大功告成了！ 現在，您只需要移至 [Azure 入口網站](https://portal.azure.com/)和 [Azure 資源總管工具](https://resources.azure.com)，即可查看已新增 JSON 已部署應用程式中的新警示和自動調整設定。

您在本節中的步驟主要是完成下列作業：

1. 準備範本檔案
2. 建立要與範本檔案一起使用的參數檔案
3. 部署具有參數檔案的範本檔案

最後一個步驟是透過 PowerShell Cmdlet 輕鬆完成。 若要查看 Visual Studio 在部署您的應用程式時所執行的作業，請開啟 Scripts\Deploy-AzureResourceGroup.ps1。 在該處有很多程式碼，但我只想要討論部署具有參數檔案之範本檔案所需的所有相關程式碼。

![](./media/app-service-deploy-complex-application-predictably/deploy-12-powershellsnippet.png)

最後一個 Cmdlet ( `New-AzureResourceGroup`) 是實際執行動作的 Cmdlet。 所有這些動作都是在告訴您，在工具的協助下，透過可預測方式部署您的雲端應用程式相當簡單。 每次對具有相同參數檔案的相同範本執行 Cmdlet 時，結果都會相同。

## <a name="summary"></a>總結
在 DevOps 中，重複性和可預測性是任何成功部署包含微服務之高級別應用程式的關鍵。 在本教學課程中，您已使用 Azure 資源管理員範本將雙微服務應用程式部署至 Azure 以做為單一資源群組。 希望這可讓您了解如何開始將 Azure 中的應用程式轉換成範本，而且可以透過可預測方式進行佈建和部署。 

<a name="resources"></a>

## <a name="more-resources"></a>其他資源
* [Azure 資源管理員範本語言](../azure-resource-manager/resource-group-authoring-templates.md)
* [編寫 Azure 資源管理員範本](../azure-resource-manager/resource-group-authoring-templates.md)
* [Azure 資源管理員範本函式](../azure-resource-manager/resource-group-template-functions.md)
* [使用 Azure 資源管理員範本部署應用程式](../azure-resource-manager/resource-group-template-deploy.md)
* [搭配使用 Azure PowerShell 與 Azure 資源管理員](../azure-resource-manager/powershell-azure-resource-manager.md)
* [Azure 中的資源群組部署疑難排解](../azure-resource-manager/resource-manager-common-deployment-errors.md)

