---
title: 開始在 Node.js 中使用 Azure 搜尋服務 | Microsoft Docs
description: 逐步了解如何使用 Node.js 作為程式設計語言，在 Azure 的雲端託管搜尋服務上建置搜尋應用程式。
author: jj09
manager: jlembicz
services: search
ms.service: search
ms.topic: conceptual
ms.date: 04/26/2017
ms.author: jjed
ms.openlocfilehash: 713e5304ca54039413f68ebc189dd049e77cdaa0
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39527570"
---
# <a name="get-started-with-azure-search-in-nodejs"></a>開始在 Node.js 中使用 Azure 搜尋服務
> [!div class="op_single_selector"]
> * [入口網站](search-get-started-portal.md)
> * [.NET](search-howto-dotnet-sdk.md)
> 
> 

瞭解如何建置使用 Azure 搜尋服務提供搜尋體驗的自訂 Node.js 搜尋應用程式。 本教學課程利用 [Azure 搜尋服務 REST API](https://msdn.microsoft.com/library/dn798935.aspx) 來建構在此練習中所使用的物件和作業。

我們使用 [Node.js](https://Nodejs.org)NPM、[Sublime Text 3](http://www.sublimetext.com/3) 及 Windows 8.1 上的 Windows PowerShell 來開發和測試此代碼。

若要執行此範例，必須要有 Azure 搜尋服務，您可以在 [Azure 入口網站](https://portal.azure.com)註冊此服務。 如需逐步指示，請參閱 [在入口網站中建立 Azure 搜尋服務](search-create-service-portal.md) 。

## <a name="about-the-data"></a>關於資料
此範例應用程式使用的 [美國地理服務中心 (USGS)](http://geonames.usgs.gov/domestic/download_data.htm)資料已依據羅德島州進行篩選，藉此減少資料集的大小。 我們將使用此資料建置可傳回地標建築物 (例如醫院和學校) 及地理特徵 (例如河流、湖泊和山峰) 的搜尋應用程式。

在此應用程式中， **DataIndexer** 程式會使用 [索引子](https://msdn.microsoft.com/library/azure/dn798918.aspx) 建構來建置及載入索引，以從公用 Azure SQL Database 擷取篩選過的 USGS 資料集。 程式碼中提供線上資料來源的認證和連接資訊。 不需要進一步設定。

> [!NOTE]
> 我們在此資料集套用了一個篩選，以維持不超過免費版定價層的 10,000 個文件的數量上限。 如果使用標準版定價層，就不會套用此限制。 如需各個定價層的容量詳細資料，請參閱 [搜尋服務限制](search-limits-quotas-capacity.md)。
> 
> 

<a id="sub-2"></a>

## <a name="find-the-service-name-and-api-key-of-your-azure-search-service"></a>尋找 Azure 搜尋服務的服務名稱和 API 金鑰
建立服務之後，請返回入口網站取得 URL 或 `api-key`。 如果想要連接至搜尋服務，您必須同時擁有 URL 和 `api-key` 才能驗證呼叫。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導向列中，按一下 [搜尋服務]  列出為您的訂用帳戶佈建的所有 Azure 搜尋服務。
3. 選取您要使用的服務。
4. 您應在服務儀表板上看到基本資訊磚，例如用於存取系統管理金鑰的鑰匙圖示。
5. 複製服務 URL、系統管理金鑰和查詢金鑰。 稍後您需要將這三個項目加到 config.js 檔案中。

## <a name="download-the-sample-files"></a>下載範例檔案
使用以下其中一種方法下載範例。

1. 移至 [search-node-indexer-demo](https://github.com/Azure-Samples/search-node-indexer-demo)。
2. 按一下 [下載 ZIP] ，儲存 .zip 檔案，然後解壓縮其中所含的所有檔案。

所有後續的檔案修改及執行陳述式都會用到此資料夾中的檔案。

## <a name="update-the-configjs-with-your-search-service-url-and-api-key"></a>更新 config.js， 方法為使用您的搜尋服務 URL 及 API 金鑰
使用先前複製的 URL 和 API 金鑰，在組態檔案中指定 URL、系統管理金鑰和查詢金鑰。

系統管理金鑰可將服務作業的完整控制權限授與給您，包括建立或刪除索引，以及載入文件。 相較之下，查詢金鑰僅用於唯讀作業，通常由連接到 Azure 搜尋服務的用戶端應用程式所用。

在此範例中，我們包含查詢金鑰，協助您在客戶端應用程式中確實熟悉使用查詢金鑰的最佳做法。

下面的螢幕擷取畫面顯示在文字編輯器中開啟 **config.js** 的畫面，並與相關項目區分，讓您得知哪些位置可以使用對搜尋服務有效的值更新檔案。

![][5]

## <a name="host-a-runtime-environment-for-the-sample"></a>為範例託管執行階段環境
此範例需要 HTTP 伺服器，讓您安裝全球使用的 NPM。

使用 PowerShell 視窗以執行以下命令。

1. 瀏覽至內含 **package.json** 檔案的資料夾。
2. 輸入 `npm install`。
3. 輸入 `npm install -g http-server`。

## <a name="build-the-index-and-run-the-application"></a>建置索引並執行應用程式
1. 輸入 `npm run indexDocuments`。
2. 輸入 `npm run build`。
3. 輸入 `npm run start_server`。
4. 將您的瀏覽器導向至 `http://localhost:8080/index.html`

## <a name="search-on-usgs-data"></a>搜尋 USGS 資料
USGS 資料集包含與羅德島州相關的記錄。 如果您在空白的搜尋方塊中按一下 [搜尋] ，預設會出現前 50 個項目。

輸入搜尋詞彙，讓搜尋引擎運作一下。 試著輸入區域名稱。 "Roger Williams" 是羅德島州的第一任州長。 許多公園、建築物和學校都是以他的姓名命名。

![][9]

此外，您也可以試著使用這些字詞：

* Pawtucket
* Pembroke
* goose +cape

## <a name="next-steps"></a>後續步驟
這是第一個以 Node.js 和 USGS 資料集為基礎的 Azure 搜尋服務教學課程。 我們會漸漸擴充本教學課程，以示範其他您可能會想用在自訂方案中的搜尋功能。

如果您已有一些 Azure 搜尋服務的背景知識，可以利用此範例做為試用建議工具 (預先輸入或自動完成查詢)、篩選及多面向導覽的跳板。 您也可以新增計數和批次處理文件，讓使用者可以逐頁查看結果，藉此改進搜尋結果頁面。

不熟悉 Azure 搜尋服務嗎？ 建議您嘗試學習其他教學課程，深入了解您還可以建立哪些東西。 請瀏覽我們的 [文件頁面](https://azure.microsoft.com/documentation/services/search/) 以尋找更多資源。 

<!--Image references-->
[1]: ./media/search-get-started-Nodejs/create-search-portal-1.PNG
[2]: ./media/search-get-started-Nodejs/create-search-portal-2.PNG
[3]: ./media/search-get-started-Nodejs/create-search-portal-3.PNG
[5]: ./media/search-get-started-Nodejs/AzSearch-Nodejs-configjs.png
[9]: ./media/search-get-started-Nodejs/rogerwilliamsschool.png
