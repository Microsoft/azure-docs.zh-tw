---
title: 教學課程 - 將 Azure CDN 新增至 Azure App Service Web 應用程式 | Microsoft Docs
description: 在本教學課程中，Azure 內容傳遞網路 (CDN) 會新增至 Azure App Service Web 應用程式，以從您在世界各地的客戶附近的伺服器快取和傳遞靜態檔案。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/14/2018
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: db4a9a43a971f66870c2079762b9c62802122baa
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093318"
---
# <a name="tutorial-add-azure-cdn-to-an-azure-app-service-web-app"></a>教學課程：將 Azure CDN 新增至 Azure App Service Web 應用程式

本教學課程說明如何將 [Azure 內容傳遞網路 (CDN)](cdn-overview.md) 新增至 [Azure App Service 中的 Web 應用程式](../app-service/app-service-web-overview.md)。 Web 應用程式是用來裝載 Web 應用程式、REST API 和行動後端的服務。 

以下是您將使用的範例靜態 HTML 網站首頁︰

![範例應用程式首頁](media/cdn-add-to-web-app/sample-app-home-page.png)

您將學到什麼：

> [!div class="checklist"]
> * 建立 CDN 端點。
> * 重新整理快取的資產。
> * 使用查詢字串來控制快取的版本。


## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

- [安裝 Git](https://git-scm.com/)
- [安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-the-web-app"></a>建立 Web 應用程式

若要建立您將使用的 Web 應用程式，請遵循[靜態 HTML 快速入門](../app-service/app-service-web-get-started-html.md)的**瀏覽至應用程式**步驟。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

開啟瀏覽器並瀏覽至 [Azure 入口網站](https://portal.azure.com)。

### <a name="dynamic-site-acceleration-optimization"></a>動態網站加速最佳化
如果您想要為 CDN 端點進行動態網站加速 (DSA) 的最佳化，您應使用 [CDN 入口網站](cdn-create-new-endpoint.md)建立設定檔和端點。 執行 [DSA 最佳化](cdn-dynamic-site-acceleration.md)後，具有動態內容的網頁將可顯著提升效能。 如需如何從 CDN 入口網站為 CDN 端點進行 DSA 最佳化的相關指示，請參閱[用來加速傳遞動態檔案的 CDN 端點組態](cdn-dynamic-site-acceleration.md#cdn-endpoint-configuration-to-accelerate-delivery-of-dynamic-files)。 否則，如果您不想要最佳化新端點，您可以依照下一節中的步驟，使用 Web 應用程式入口網站建立端點。 請注意，對於**來自 Verizon 的 Azure CDN** 設定檔，您無法在 CDN 端點建立之後變更其最佳化。

## <a name="create-a-cdn-profile-and-endpoint"></a>建立 CDN 設定檔和端點

在左側導覽中，選取 [應用程式服務]，然後選取您在[靜態 HTML 快速入門](../app-service/app-service-web-get-started-html.md)中建立的應用程式。

![在入口網站中選取 App Service 應用程式](media/cdn-add-to-web-app/portal-select-app-services.png)

在 [App Service] 頁面的 [設定] 區段中，選取 [網路] > [設定您應用程式的 Azure CDN]。

![在入口網站中選取 CDN](media/cdn-add-to-web-app/portal-select-cdn.png)

在 [Azure 內容傳遞網路] 頁面中，提供表格中所指定的 [新端點] 設定。

![在入口網站中建立設定檔和端點](media/cdn-add-to-web-app/portal-new-endpoint.png)

| 設定 | 建議的值 | 說明 |
| ------- | --------------- | ----------- |
| **CDN 設定檔** | myCDNProfile | CDN 設定檔是定價層相同的 CDN 端點集合。 |
| **定價層** | 標準 Akamai | [定價層](cdn-features.md)指定提供者和可用的功能。 本教學課程使用標準 Akamai。 |
| **CDN 端點名稱** | azureedge.net 網域中任何唯一的名稱 | 您可以在網域 *&lt;endpointname&gt;*.azureedge.net 上存取您的快取資源。

選取 [建立] 以建立 CDN 設定檔。

Azure 會建立設定檔和端點。 新端點會出現在 [端點] 清單中，而且其佈建後的狀態為 [執行中]。

![清單中的新端點](media/cdn-add-to-web-app/portal-new-endpoint-in-list.png)

### <a name="test-the-cdn-endpoint"></a>測試 CDN 端點

 因為需要一段時間才能傳播註冊，所以端點不會立即可供使用： 
   - 若為**來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
   - 若為**來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
   - 若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 90 分鐘內完成傳播。 

範例應用程式有 *index.html* 檔案以及包含其他靜態資產的 *css*、*img* 和 *js* 資料夾。 在 CDN 端點上，上述所有檔案的內容路徑都相同。 例如，下列 URL 可存取 css 資料夾中的 bootstrap.css 檔案︰

```
http://<appname>.azurewebsites.net/css/bootstrap.css
```

```
http://<endpointname>.azureedge.net/css/bootstrap.css
```

在瀏覽器中瀏覽至下列 URL：

```
http://<endpointname>.azureedge.net/index.html
```

![CDN 提供的範例應用程式首頁](media/cdn-add-to-web-app/sample-app-home-page-cdn.png)

 您會看到與您稍早在 Azure Web 應用程式中執行的相同分頁。 Azure CDN 已擷取原始 Web 應用程式的資產，並從 CDN 端點提供這些資產

若要確保此頁面已在 CDN 中快取，請重新整理此頁面。 CDN 有時需要相同資產的兩個要求，才能快取所要求的內容。

如需建立 Azure CDN 設定檔和端點的詳細資訊，請參閱[開始使用 Azure CDN](cdn-create-new-endpoint.md)。

## <a name="purge-the-cdn"></a>清除 CDN

CDN 會根據存留時間 (TTL) 組態，從原始 web 應用程式定期重新整理其資源。 預設 TTL 為 7 天。

您有時可能須在 TTL 到期日之前重新整理 CDN；例如將更新的內容部署至 Web 應用程式時。 若要觸發重新整理，請手動清除 CDN 資源。 

在本節的教學課程中，您會將變更部署到 Web 應用程式並清除 CDN，進而觸發 CDN 重新整理其快取。

### <a name="deploy-a-change-to-the-web-app"></a>將變更部署到 Web 應用程式

開啟 *index.html* 檔案並將 *V2* 新增至 H1 標題，如下列範例所示︰ 

```
<h1>Azure App Service - Sample Static HTML Site - V2</h1>
```

認可您的變更並將它部署至 Web 應用程式。

```bash
git commit -am "version 2"
git push azure master
```

部署完成後，瀏覽至 Web 應用程式 URL 即可看到變更。

```
http://<appname>.azurewebsites.net/index.html
```

![Web 應用程式標題中的 V2](media/cdn-add-to-web-app/v2-in-web-app-title.png)

若您瀏覽至首頁的 CDN 端點 URL，則會因為 CDN 中快取的版本尚未到期，所以不會看到變更。 

```
http://<endpointname>.azureedge.net/index.html
```

![CDN 標題中沒有 V2](media/cdn-add-to-web-app/no-v2-in-cdn-title.png)

### <a name="purge-the-cdn-in-the-portal"></a>在入口網站中清除 CDN

若要觸發 CDN 更新其快取的版本，請清除 CDN。

在入口網站的左側導覽中，選取 [資源群組]，然後選取您為 Web 應用程式 (myResourceGroup) 建立的資源群組。

![選取資源群組](media/cdn-add-to-web-app/portal-select-group.png)

在資源清單中，選取您的 CDN 端點。

![選取端點](media/cdn-add-to-web-app/portal-select-endpoint.png)

在 [端點] 頁面頂端選取 [清除]。

![選取清除](media/cdn-add-to-web-app/portal-select-purge.png)

輸入您想要清除的內容路徑。 您可以傳遞完整檔案路徑來清除個別檔案，也可以傳遞路徑區段來清除並重新整理資料夾中的所有內容。 因為您變更了 *index.html*，所以請確認該項目位在其中一個路徑內。

選取頁面底部的 [清除]。

![清除頁面](media/cdn-add-to-web-app/app-service-web-purge-cdn.png)

### <a name="verify-that-the-cdn-is-updated"></a>確認 CDN 已更新

請等到清除要求處理完成，這通常需要幾分鐘的時間。 若要查看目前的狀態，請選取頁面頂端的鈴鐺圖示。 

![清除通知](media/cdn-add-to-web-app/portal-purge-notification.png)

當您瀏覽至 *index.html*的 CDN 端點 URL 時，會看見您新增至標題的 *V2* 出現在首頁上，這表示 CDN 快取已重新整理。

```
http://<endpointname>.azureedge.net/index.html
```

![CDN 標題中的 V2](media/cdn-add-to-web-app/v2-in-cdn-title.png)

如需詳細資訊，請參閱[清除 Azure CDN 端點](../cdn/cdn-purge-endpoint.md)。 

## <a name="use-query-strings-to-version-content"></a>使用查詢字串來控制內容版本

Azure CDN 提供下列快取行為選項︰

* 忽略查詢字串
* 略過查詢字串的快取
* 快取每個唯一的 URL 

第一個選項是預設，這表示不管 URL 中的查詢字串為何，資產都只有一個快取的版本。 

在本節的教學課程中，您可將快取行為變更為快取每個唯一 URL。

### <a name="change-the-cache-behavior"></a>變更快取行為

在 Azure 入口網站的 [CDN 端點] 頁面中，選取 [快取]。

從 [查詢字串快取行為] 下拉式清單中，選取 [快取每個唯一 URL]。

選取 [ **儲存**]。

![選取查詢字串快取行為](media/cdn-add-to-web-app/portal-select-caching-behavior.png)

### <a name="verify-that-unique-urls-are-cached-separately"></a>確認唯一的 URL 會分開快取

在瀏覽器中，瀏覽至 CDN 端點首頁，並包含查詢字串︰ 

```
http://<endpointname>.azureedge.net/index.html?q=1
```

Azure CDN 會傳回目前的 Web 應用程式內容，其標題中包含 *V2*。 

若要確保此頁面已在 CDN 中快取，請重新整理此頁面。 

開啟 *index.html*並將 *V2* 變更為 *V3*，然後部署變更。 

```bash
git commit -am "version 3"
git push azure master
```

在瀏覽器中，移至含有新查詢字串 (例如 `q=2`) 的 CDN 端點 URL。 CDN 會取得目前的 *index.html* 檔案並顯示 *V3*。 但是，如果您瀏覽至含有 `q=1` 查詢字串的 CDN 端點，則會看到 *V2*。

```
http://<endpointname>.azureedge.net/index.html?q=2
```

![CDN 標題中的 V3，查詢字串 2](media/cdn-add-to-web-app/v3-in-cdn-title-qs2.png)

```
http://<endpointname>.azureedge.net/index.html?q=1
```

![CDN 標題中的 V2，查詢字串 1](media/cdn-add-to-web-app/v2-in-cdn-title-qs1.png)

此輸出會顯示每個查詢字串是以不同方式處理：

* 以前使用 q=1，因此會傳回快取內容 (V2)。
* q=2 是新的，因此會擷取及傳回最新的 Web 應用程式內容 (V3)。

如需詳細資訊，請參閱[使用查詢字串控制 Azure CDN 快取行為](../cdn/cdn-query-string.md)。

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>後續步驟

您已了解如何︰

> [!div class="checklist"]
> * 建立 CDN 端點。
> * 重新整理快取的資產。
> * 使用查詢字串來控制快取的版本。

在下列文章中了解如何將 CDN 效能最佳化：

> [!div class="nextstepaction"]
> [教學課程：將自訂網域新增至 Azure CDN 端點](cdn-map-content-to-custom-domain.md)


