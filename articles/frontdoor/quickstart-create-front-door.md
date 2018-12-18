---
title: Azure 快速入門 - 使用 Azure 入口網站建立 Front Door 設定檔以讓應用程式具有高可用性
description: 本快速入門文章將說明如何建立 Front Door，以供高可用性且高效能的全域 Web 應用程式使用。
services: front-door
documentationcenter: ''
author: sharad4u
editor: ''
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/31/2018
ms.author: sharadag
ms.openlocfilehash: 39e7626e6b4c545649e39ff2120d1f1fd105d764
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46994661"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application"></a>快速入門：為高可用性的全域 Web 應用程式建立 Front Door

本快速入門將說明如何建立 Front Door 設定檔，以便為您的全域 Web 應用程式提供高可用性和高效能。 

本快速入門所述的案例包含兩個在不同 Azure 區域中執行的 Web 應用程式執行個體。 根據相等的[加權和相同優先順序的後端](front-door-routing-methods.md)來建立 Front Door 設定，有助於將使用者流量導向至正在執行應用程式的網站後端中最接近的一組。 Front Door 會持續監視 Web 應用程式，並在最接近的網站無法使用時，自動容錯移轉至下一個可用的後端。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="sign-in-to-azure"></a>登入 Azure 
在 https://portal.azure.com 登入 Azure 入口網站。

## <a name="prerequisites"></a>必要條件
要進行本快速入門，您必須部署兩個在不同 Azure 區域 (美國東部和西歐) 中執行的 Web 應用程式執行個體。 這兩個 Web 應用程式執行個體都會以主動/主動模式執行，也就是，不同於主動/待命設定會將其中一個用來作為容錯移轉，它們其中任一個隨時都能接受流量。

1. 在畫面左上方，選取 [建立資源] > [Web] > [Web 應用程式] > [建立]。
2. 在 [Web 應用程式] 中，輸入或選取下列資訊，然後在未指定任何值的地方輸入預設設定：

     | 設定         | 值     |
     | ---              | ---  |
     | 名稱           | 輸入 Web 應用程式的唯一名稱  |
     | 資源群組          | 選取 [新增]，然後輸入 *myResourceGroupFD1* |
     | App Service 方案/位置         | 選取 [ **新增**]。  在 App Service 方案中，輸入「myAppServicePlanEastUS」，然後選取 [確定]。 
     |      位置  |   美國東部        |
    |||

3. 選取 [建立] 。
4. 成功部署 Web 應用程式時，便已建立好預設網站。
5. 使用下列設定重複步驟 1 至 3，在不同 Azure 區域中建立第二個網站：

     | 設定         | 值     |
     | ---              | ---  |
     | 名稱           | 輸入 Web 應用程式的唯一名稱  |
     | 資源群組          | 選取 [新增]，然後輸入 *myResourceGroupFD2* |
     | App Service 方案/位置         | 選取 [ **新增**]。  在 App Service 方案中，輸入「myAppServicePlanWestEurope」，然後選取 [確定]。 
     |      位置  |   西歐      |
    |||


## <a name="create-a-front-door-for-your-application"></a>為您的應用程式建立 Front Door
### <a name="a-add-a-frontend-host-for-front-door"></a>A. 新增適用於 Front Door 的前端主機
建立 Front Door 設定，根據兩個後端之間的最低延遲來引導使用者流量。

1. 在畫面的左上方，選取 [建立資源] > [網路] > [Front Door] > [建立]。
2. 在 [建立 Front Door] 中，您要開始新增基本資訊，並提供您想要在其中設定 Front Door 的訂用帳戶。 同樣地，就像任何其他 Azure 資源，如果您要建立新的，也必須提供資源群組和資源群組區域。 最後，您需要為 Front Door 提供名稱。
3. 填入基本資訊之後，您需要定義的第一個步驟是適用於設定的**前端主機**。 結果應該是有效的網域名稱，例如 `myappfrontend.azurefd.net`。 此主機名稱必須是全域唯一的，但 Front Door 將負責進行該驗證。 

### <a name="b-add-application-backend-and-backend-pools"></a>B. 新增應用程式後端和後端集區

接下來，您需要在後端集區中設定應用程式後端，讓 Front Door 知道您應用程式的所在位置。 

1. 按一下 [+] 圖示以新增後端集區，然後指定您後端集區的**名稱**，假設是 `myBackendPool`。
2. 接下來，按一下 [新增後端] 以新增您稍早建立的網站。
3. 針對 [目標主機類型] 選取「應用程式服務」、選取您用來建立網站的訂用帳戶，然後選擇 [目標主機名稱] 中的第一個網站，也就是 *myAppServicePlanEastUS.azurewebsites.net*。
4. 目前依現狀保留其餘欄位，然後按一下 [新增]。
5. 重複執行步驟 2 到 4 以新增另一個網站，也就是 *myAppServicePlanWestEurope.azurewebsites.net*
6. 您可以選擇性地選擇更新後端集區的健康情況探查和負載平衡設定，但預設值應該也能運作。 按一下 [新增] 。


### <a name="c-add-a-routing-rule"></a>C. 新增路由規則
最後，按一下路由規則上的 [+] 圖示以設定路由規則。 這需要將您的前端主機對應到後端集區，如果要求會傳入 `myappfrontend.azurefd.net`，基本上就需要設定此對應，然後再將它轉送到後端集區 `myBackendPool`。 按一下 [新增]，為您的 Front Door 新增路由規則。 您現在最好應該要建立 Front Door，因此請按一下 [檢閱及建立]。

>[!WARNING]
> 您**必須**確保 Front Door 中的每部前端主機都具有一個路由規則，其中含有與其相關聯的預設路徑 ('/\*')。 也就是，在您的所有路由規則中，至少必須有一個適用於您每部前端主機的路由規則，這類主機會以預設路徑 ('/\*') 來定義。 無法執行這項操作，可能導致您的使用者流量無法被正確路由傳送。

## <a name="view-front-door-in-action"></a>檢視動作中的 Front Door
一旦建立 Front Door 之後，將需要幾分鐘的時間，才能在世界各地部署該設定。 完成後，存取您所建立的前端主機，也就是前往網頁瀏覽器並點擊 URL `myappfrontend.azurefd.net`。 您的要求將會從後端集區中指定的後端，自動路由傳送到最接近您的後端。 

### <a name="view-front-door-handle-application-failover"></a>檢視 Front Door 處理應用程式容錯移轉
如果您想要測試 Front Door 動作中的即時全域容錯移轉，可以前往您所建立的其中一個網站並停止它。 根據針對後端集區所定義的健康情況探查設定，我們會立即將流量容錯移轉至另一個網站部署。 您也可以藉由在 Front Door 的後端集區設定中停用後端來測試行為。 

## <a name="clean-up-resources"></a>清除資源
若不再需要，可刪除資源群組、Web 應用程式和所有相關資源。

## <a name="next-steps"></a>後續步驟
在本快速入門中，您建立了 Front Door，讓您能夠針對需要高可用性和最大效能的 Web 應用程式引導使用者流量。 若要深入了解如何路由傳送流量，請閱讀 Front Door 所使用的[路由方法](front-door-routing-methods.md)。