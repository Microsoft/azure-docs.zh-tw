---
title: 使用範本部署 Web 應用程式 - Azure Cosmos DB | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 範本，來部署 Azure Cosmos DB 帳戶、Azure App Service Web Apps，以及範例 Web 應用程式。
services: cosmos-db, app-service\web
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 02/23/2018
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: dca9d7900ce229b1cddbef8d0dee44bc0061dc42
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34611301"
---
# <a name="deploy-azure-cosmos-db-and-azure-app-service-web-apps-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本部署 Azure Cosmos DB 和 Azure App Service Web Apps
本教學課程示範如何使用 Azure Resource Manager 範本，來部署和整合 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)、[Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714) Web 應用程式及範例 Web 應用程式。

使用 Azure Resource Manager 範本，您可以輕鬆自動化 Azure 資源的部署和設定。  本教學課程示範如何部署 Web 應用程式，以及自動設定 Azure Cosmos DB 帳戶連接資訊。

完成本教學課程後，您就能夠回答下列問題：  

* 如何使用 Azure Resource Manager 範本，來部署和整合 Azure Cosmos DB 帳戶與 Azure App Service 中的 Web 應用程式？
* 如何使用 Azure Resource Manager 範本，來部署和整合 Azure Cosmos DB 帳戶、App Service Web Apps 中的 Web 應用程式，以及 Webdeploy 應用程式？

<a id="Prerequisites"></a>

## <a name="prerequisites"></a>先決條件
> [!TIP]
> 雖然本教學課程沒有假設您先前已有使用 Azure Resource Manager 範本或 JSON 的經驗，但是，如果您需要修改參考的範本或部署選項，還是需要每個領域的知識。
> 
> 

在依照本教學課程中的指示進行之前，請先確定您已備妥 Azure 訂用帳戶。 Azure 是訂閱型平台。  如需取得訂用帳戶的詳細資訊，請參閱[購買選項](https://azure.microsoft.com/pricing/purchase-options/)、[成員供應項目](https://azure.microsoft.com/pricing/member-offers/)或[免費試用版](https://azure.microsoft.com/pricing/free-trial/)。

## <a id="CreateDB"></a>步驟 1︰下載範本檔案
讓我們從下載本教學課程所需的範本檔案開始。

1. 將[建立 Azure Cosmos DB 帳戶、Web Apps 和部署示範應用程式範例](https://portalcontent.blob.core.windows.net/samples/DocDBWebsiteTodo.json)範本下載至本機資料夾 (例如 C:\Azure Cosmos DBTemplates)。 這個範本將會部署 Azure Cosmos DB 帳戶、App Service Web 應用程式和 Web 應用程式。  它還會自動設定 Web 應用程式以連線到 Azure Cosmos DB 帳戶。
2. 將[建立 Azure Cosmos DB 帳戶和 Web Apps 範例](https://portalcontent.blob.core.windows.net/samples/DocDBWebSite.json)範本下載至本機資料夾 (例如，C:\Azure Cosmos DBTemplates)。 這個範本將會部署 Azure Cosmos DB 帳戶、App Service Web 應用程式，並修改網站的應用程式設定來輕鬆呈現 Azure Cosmos DB 連線資訊，但不包含 Web 應用程式。  

<a id="Build"></a>

## <a name="step-2-deploy-the-azure-cosmos-db-account-app-service-web-app-and-demo-application-sample"></a>步驟 2：部署 Azure Cosmos DB 帳戶、App Service Web 應用程式與示範應用程式範例
現在讓我們來部署第一個範本。

> [!TIP]
> 這個範本不會驗證下列範本中輸入的 Web 應用程式名稱和 Azure Cosmos DB 帳戶名稱 a) 是否有效且 b) 可用。  強烈建議您先確認打算提供之名稱的可用性，再提交部署。
> 
> 

1. 登入 [Azure 入口網站](https://portal.azure.com)，按一下 [新增]，並搜尋「範本部署」。
    ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment1.png)
2. 選取範本部署項目，然後按一下 [建立] ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment2.png)
3. 按一下 [編輯範本]，貼上 DocDBWebsiteTodo.json 範本檔案的內容，然後按一下 [儲存]。
   ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment3.png)
4. 按一下 [編輯參數]，提供每個必要參數的值，然後按一下 [確定]。  參數如下所示：
   
   1. SITENAME：指定 App Service Web 應用程式名稱，並用來建構您將用來存取 Web 應用程式的 URL (例如，如果您指定 "mydemodocdbwebsite"，則將存取 Web 應用程式的 URL 會是 mydemodocdbwebsite.azurewebsites.net)。
   2. HOSTINGPLANNAME︰指定要建立的主控方案 App Service 名稱。
   3. LOCATION：指定要在其中建立 Azure Cosmos DB 和 Web 應用程式資源的 Azure 位置。
   4. DATABASEACCOUNTNAME：指定要建立的 Azure Cosmos DB 帳戶名稱。   
      
      ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment4.png)
5. 選擇現有的資源群組，或提供名稱建立新的資源群組，然後選擇資源群組的位置。

    ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment5.png)
6. 依序按一下 [檢閱法律條款]、[購買] 和 [建立] 以開始部署。  選取 [釘選到儀表板]  讓產生的部署輕鬆簡單顯示在 Azure 入口網站的首頁上。
   ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment6.png)
7. 部署完成時，隨即開啟 [資源群組] 窗格。
   ![資源群組窗格的螢幕擷取畫面](./media/create-website/TemplateDeployment7.png)  
8. 若要使用應用程式，請巡覽至 Web 應用程式 URL (上述範例中的 URL 會是 http://mydemodocdbwebapp.azurewebsites.net) )。  您會看到下列的 Web 應用程式：
   
   ![範例待辦事項應用程式](./media/create-website/image2.png)
9. 請繼續在 Web 應用程式中建立幾個工作，然後回到 Azure 入口網站的 [資源群組] 窗格。 按一下 [資源] 清單中的 Azure Cosmos DB 帳戶資源，然後按一下 [資料總管]。
10. 執行預設查詢 "SELECT * FROM c"，然後檢查結果。  請注意查詢已擷取您在步驟 7 所建立待辦事項的 JSON 表示法。  任意嘗試查詢；例如，嘗試執行 SELECT * FROM c WHERE c.isComplete = true，以傳回所有已標示為完成的待辦事項。
11. 任意瀏覽 Azure Cosmos DB 入口網站體驗，或修改範例 Todo 應用程式。  當您準備好時，讓我們來部署另一個範本。

<a id="Build"></a> 

## <a name="step-3-deploy-the-document-account-and-web-app-sample"></a>步驟 3：部署文件帳戶和 Web 應用程式範例
現在讓我們來部署第二個範本。  這個範本非常有用，它示範如何將帳戶端點和主要金鑰等 Azure Cosmos DB 連接資訊插入 Web 應用程式，以當做應用程式設定或自訂的連接字串。 例如，您或許擁有想要使用 Azure Cosmos DB 帳戶部署的 Web 應用程式，以及在部署期間自動填入的連接資訊。

> [!TIP]
> 這個範本不會驗證下面輸入的 Web 應用程式名稱和 Azure Cosmos DB 帳戶名稱 a) 是否有效且 b) 可用。  強烈建議您先確認打算提供之名稱的可用性，再提交部署。
> 
> 

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [新增]，並搜尋「範本部署」。
    ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment1.png)
2. 選取範本部署項目，然後按一下 [建立] ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment2.png)
3. 按一下 [編輯範本]，貼上 DocDBWebSite.json 範本檔案的內容，然後按一下 [儲存]。
   ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment3.png)
4. 按一下 [編輯參數]，提供每個必要參數的值，然後按一下 [確定]。  參數如下所示：
   
   1. SITENAME：指定 App Service Web 應用程式名稱，並用來建構您將用來存取 Web 應用程式的 URL (例如：如果您指定 "mydemodocdbwebsite"，則將存取 Web 應用程式的 URL 會是 mydemodocdbwebsite.azurewebsites.net)。
   2. HOSTINGPLANNAME︰指定要建立的主控方案 App Service 名稱。
   3. LOCATION：指定要在其中建立 Azure Cosmos DB 和 Web 應用程式資源的 Azure 位置。
   4. DATABASEACCOUNTNAME：指定要建立的 Azure Cosmos DB 帳戶名稱。   
      
      ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment4.png)
5. 選擇現有的資源群組，或提供名稱建立新的資源群組，然後選擇資源群組的位置。

    ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment5.png)
6. 依序按一下 [檢閱法律條款]、[購買] 和 [建立] 以開始部署。  選取 [釘選到儀表板]  讓產生的部署輕鬆簡單顯示在 Azure 入口網站的首頁上。
   ![範本部署 UI 的螢幕擷取畫面](./media/create-website/TemplateDeployment6.png)
7. 部署完成時，隨即開啟 [資源群組] 窗格。
   ![資源群組窗格的螢幕擷取畫面](./media/create-website/TemplateDeployment7.png)  
8. 按一下 [資源] 清單中的 Web 應用程式資源，然後按一下 [應用程式設定] ![資源群組的螢幕擷取畫面](./media/create-website/TemplateDeployment9.png)  
9. 請注意出現的 Azure Cosmos DB 端點以及每個 Azure Cosmos DB 主要金鑰的應用程式設定。

    ![應用程式設定的螢幕擷取畫面](./media/create-website/TemplateDeployment10.png)  
10. 任意繼續瀏覽探索 Azure 入口網站，或遵循其中一個 Azure Cosmos DB [範例](http://go.microsoft.com/fwlink/?LinkID=402386)，來建立您自己的 Azure Cosmos DB 應用程式。

<a name="NextSteps"></a>

## <a name="next-steps"></a>後續步驟
恭喜！ 您已使用 Azure Resource Manager 範本部署了 Azure Cosmos DB、App Service Web 應用程式及範例 Web 應用程式。

* 若要深入了解 Azure Cosmos DB，請按一下[這裡](http://azure.com/docdb)。
* 若要深入了解 Azure App Service Web Apps，請按一下 [這裡](http://go.microsoft.com/fwlink/?LinkId=325362)。
* 若要深入了解 Azure 資源管理員範本，請按一下 [這裡](https://msdn.microsoft.com/library/azure/dn790549.aspx)。

## <a name="whats-changed"></a>變更的項目
* 如需從網站變更為 App Service 的指南，請參閱： [Azure App Service 及其對現有 Azure 服務的影響](http://go.microsoft.com/fwlink/?LinkId=529714)

> [!NOTE]
> 如果您想在註冊 Azure 帳戶前開始使用 Azure App Service，請移至 [試用 App Service](http://go.microsoft.com/fwlink/?LinkId=523751)，即可在 App Service 中立即建立短期入門 Web 應用程式。 不需要信用卡；無需承諾。
> 
> 

