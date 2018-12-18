---
title: Azure Cosmos DB 連接器的 Power BI 教學課程 | Microsoft Docs
description: 使用本 Power BI 教學課程以匯入 JSON、建立具深入資訊的報告以及使用 Azure Cosmos DB 和 Power BI 連接器將資料視覺化。
keywords: power bi 教學課程，視覺化資料，power bi 連接器
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 08/17/2018
ms.author: sngun
ms.openlocfilehash: b5b3a96991a2150e553af01e3fda73e04116d7fb
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452406"
---
# <a name="power-bi-tutorial-for-azure-cosmos-db-visualize-data-using-the-power-bi-connector"></a>Azure Cosmos DB 的 Power BI 教學課程：使用 Power BI 連接器將資料視覺化
[PowerBI.com](https://powerbi.microsoft.com/) 是一項線上服務，您可以在其中建立及共用具有您和組織之重要資料的儀表板和報告。  Power BI Desktop 是一項專用的報告撰寫工具，可讓您從各種資料來源擷取資料、合併和轉換資料、建立功能強大的報告和視覺效果，以及將報告發佈至 Power BI。  有了最新版的 Power BI Desktop，您現在可以透過 Power BI 的 Azure Cosmos DB 連接器連線到 Azure Cosmos DB 帳戶。   

在本 Power BI 教學課程中，我們將逐步解說在 Power BI Desktop 中連線到 Azure Cosmos DB 帳戶、瀏覽到我們要使用瀏覽器擷取資料的集合、使用 Power BI Desktop 查詢編輯器將 JSON 資料轉換成表格格式，以及建置報表並將其發行至 PowerBI.com 等作業。

完成本教學課程後，您將能夠回答下列問題：  

* 如何使用 Power BI Desktop，利用 Azure Cosmos DB 中的資料來建置報表？
* 如何在 Power BI Desktop 中連線到 Azure Cosmos DB 帳戶？
* 如何在 Power BI Desktop 中從集合擷取資料？
* 如何在 Power BI Desktop 中轉換巢狀 JSON 資料？
* 如何在 PowerBI.com 中發佈及共用我的報告？

> [!NOTE]
> 適用於 Azure Cosmos DB 的 Power BI 連接器會連線到 Power BI Desktop 以擷取和轉換資料。 在 Power BI Desktop 中建立的報告接著可以發行至 PowerBI.com。 您無法在 PowerBI.com 中直接擷取和轉換 Azure Cosmos DB 資料。 

> [!NOTE]
> 目前僅支援 Azure Cosmos DB SQL API 和 Gremlin API 帳戶的 Azure Cosmos DB 與 Power BI 連接器連線。

## <a name="prerequisites"></a>必要條件
在依照本 Power BI 教學課程中的指示進行之前，請先確定您可以存取下列資源：

* [最新版的 Power BI Desktop](https://powerbi.microsoft.com/desktop)
* 存取我們的示範帳戶或您 Azure Cosmos DB 帳戶中的資料。
  * 示範帳戶會填入此教學課程中所顯示的火山資料。 此示範帳戶未受限於任何 SLA，而是僅供示範之用。  我們保留對此示範帳戶隨時進行修改的權利，包括 (但不限於) 終止帳戶、變更金鑰、限制存取權、變更和刪除資料，不事先通知或告知原因。
    * URL： `https://analytics.documents.azure.com`
    * 唯讀金鑰：`MSr6kt7Gn0YRQbjd6RbTnTt7VHc5ohaAFu7osF0HdyQmfR+YhwCH2D2jcczVIR1LNK3nMPNBD31losN7lQ/fkw==`
  * 或者，若要建立您自己的帳戶，請參閱[使用 Azure 入口網站建立 Azure Cosmos DB 資料庫帳戶](https://azure.microsoft.com/documentation/articles/create-account/)。 然後，若要取得類似於本教學課程所使用的範例火山資料 (但不包含 GeoJSON 區塊)，請參閱 [NOAA 網站](https://www.ngdc.noaa.gov/nndc/struts/form?t=102557&s=5&d=5)，然後使用 [Azure Cosmos DB 資料移轉工具](import-data.md)匯入資料。

若要在 PowerBI.com 上共用您的報告，您必須有 PowerBI.com 中的帳戶。  若要深入了解免費的 Power BI 和 Power BI Pro，請瀏覽 [https://powerbi.microsoft.com/pricing](https://powerbi.microsoft.com/pricing)。

## <a name="lets-get-started"></a>現在就開始吧
在本教學課程中，我們假設您是研究世界各地火山的地質學家。  火山資料儲存在 Azure Cosmos DB 帳戶中，而 JSON 文件看起來如同下列範例文件。

    {
        "Volcano Name": "Rainier",
           "Country": "United States",
          "Region": "US-Washington",
          "Location": {
            "type": "Point",
            "coordinates": [
              -121.758,
              46.87
            ]
          },
          "Elevation": 4392,
          "Type": "Stratovolcano",
          "Status": "Dendrochronology",
          "Last Known Eruption": "Last known eruption from 1800-1899, inclusive"
    }

您想要從 Azure Cosmos DB 帳戶擷取火山資料，並在如同下列報表的互動式 Power BI 報表中將資料視覺化。

![藉由使用 Power BI 連接器完成這個 Power BI 教學課程，您將可以使用 Power BI Desktop 火山報告將資料視覺化](./media/powerbi-visualize/power_bi_connector_pbireportfinal.png)

準備好要試試看了嗎？ 現在就開始吧。

1. 在您的工作站上執行 Power BI Desktop。
2. Power BI Desktop 啟動時，會顯示 [歡迎使用]  畫面。
   
    ![Power BI Desktop 歡迎使用畫面 - Power BI 連接器](./media/powerbi-visualize/power_bi_connector_welcome.png)
3. 您可以 [取得資料]、檢視 [最近的來源]，或直接從 [歡迎使用] 畫面 [開啟其他報告]。  按一下右上角的 X，以關閉畫面。 Power BI Desktop 的 [報告]  檢視隨即顯示。
   
    ![Power BI Desktop 報告檢視 - Power BI 連接器](./media/powerbi-visualize/power_bi_connector_pbireportview.png)
4. 選取 [首頁] 功能區，然後按一下 [取得資料]。  此時應會出現 [取得資料]  視窗。
5. 按一下 [Azure]、選取 [Azure Cosmos DB (搶鮮版 (Beta))]，然後按一下 [連線]。 

    ![Power BI Desktop 取得資料 - Power BI 連接器](./media/powerbi-visualize/power_bi_connector_pbigetdata.png)   
6. 在 [預覽版連接器] 頁面上，按一下 [繼續]。 [Azure Cosmos DB] 視窗隨即出現。
7. 依照下列所示方式指定您要從中擷取資料的 Azure Cosmos DB 帳戶端點 URL，然後按一下 [確定]。 若要使用您的帳戶，可以從 Azure 入口網站 [[金鑰](manage-account.md#keys)] 刀鋒視窗的 [URI] 方塊中擷取 URL。 若要使用示範帳戶，請輸入 `https://analytics.documents.azure.com` 作為 URL。 
   
    資料庫名稱、集合名稱、SQL 陳述式皆保留空白，這些欄位是選用的。  我們將使用「瀏覽器」來選取資料庫和集合，以識別資料來自何處。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 桌面連接視窗](./media/powerbi-visualize/power_bi_connector_pbiconnectwindow.png)
8. 如果是第一次連接到此端點，系統會提示您提供帳戶金鑰。 如果是您自己的帳戶，請從 Azure 入口網站 [[唯讀金鑰](manage-account.md#keys)] 刀鋒視窗的 [主要金鑰] 方塊中擷取金鑰。 如果是示範帳戶，金鑰是 `MSr6kt7Gn0YRQbjd6RbTnTt7VHc5ohaAFu7osF0HdyQmfR+YhwCH2D2jcczVIR1LNK3nMPNBD31losN7lQ/fkw==`。 輸入適當的金鑰，然後按一下 [連接]。
   
    建議您在建置報告時使用唯讀金鑰。  這樣可避免非必要地將主要金鑰暴露於潛在的安全性風險下。 唯讀金鑰可從 Azure 入口網站的 [[金鑰](manage-account.md#keys)] 刀鋒視窗取得，或使用上面提供的示範帳戶資訊。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 帳戶金鑰](./media/powerbi-visualize/power_bi_connector_pbidocumentdbkey.png)
    
    > [!NOTE] 
    > 如果您收到指出「找不到指定的資料庫」的錯誤， 請參閱此 [Power BI 問題](https://community.powerbi.com/t5/Issues/Document-DB-Power-BI/idi-p/208200) \(英文\) 中的因應措施步驟。
    
9. 順利連接帳戶後，會出現 [瀏覽器]  窗格。  [瀏覽器]  會顯示帳戶下的資料庫清單。
10. 按一下並展開作為報告資料來源的資料庫，如果您使用示範帳戶，請選取 [volcanodb]。   
11. 現在，選取要從中擷取資料的集合。 如果您使用示範帳戶，請選取 **volcano1**。
    
    [預覽] 窗格會顯示 [記錄]  項目的清單。  一份文件會顯示為 Power BI 中的 [記錄]  類型。 同樣地，文件內的巢狀 JSON 區塊也是 [記錄] 。
    
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 瀏覽器視窗](./media/powerbi-visualize/power_bi_connector_pbinavigator.png)
12. 按一下 [編輯] 以在新視窗中啟動 [查詢編輯器] 來轉換資料。

## <a name="flattening-and-transforming-json-documents"></a>簡維化和轉換 JSON 文件
1. 切換至 [Power BI 查詢編輯器] 視窗，中央窗格內會顯示 [文件] 資料行。
   ![Power BI Desktop 查詢編輯器](./media/powerbi-visualize/power_bi_connector_pbiqueryeditor.png)
2. 按一下位於 [文件]  資料行標頭右側的展開器。  此時會出現含有欄位清單的內容功能表。  選取您報告所需的欄位，例如火山名稱、國家、區域、位置、高度、類型、狀態和已知的上次爆發時間。 取消核取 [使用原始資料行名稱作為前置詞] 方塊，然後再按一下 [確定]。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 展開文件](./media/powerbi-visualize/power_bi_connector_pbiqueryeditorexpander.png)
3. 中央窗格會顯示所選欄位的結果預覽。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 壓平合併結果](./media/powerbi-visualize/power_bi_connector_pbiresultflatten.png)
4. 在我們的範例中，[位置] 屬性是文件中的 GeoJSON 區塊。  如您所見，[位置] 顯示為 Power BI Desktop 中的 [記錄]  類型。  
5. 按一下位於 [Document.Location] 資料行標頭右側的展開器。  此時會出現含有類型和座標欄位的內容功能表。  請選取座標欄位，並確定未選取 [使用原始資料行名稱作為前置詞]，然後按一下 [確定]。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 位置記錄](./media/powerbi-visualize/power_bi_connector_pbilocationrecord.png)
6. 中央窗格現在會顯示 [清單]  類型的座標資料行。  如本教學課程一開始所說明，本教學課程中的 GeoJSON 資料屬於 Point 類型，具有座標陣列中所記錄的緯度和經度值。
   
    coordinates[0] 項目代表經度，coordinates[1] 則代表緯度。
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 座標清單](./media/powerbi-visualize/power_bi_connector_pbiresultflattenlist.png)
7. 為了將座標陣列壓平合併，請建立名為 LatLong 的 [自訂資料行]。  選取 [新增資料行] 功能區，然後按一下 [自訂資料行]。  [自訂資料行] 視窗會隨即出現。
8. 提供新資料行的名稱，例如 LatLong。
9. 接下來，指定新資料行的自訂公式。  在我們的範例中，我們將依照下列方式使用以下公式，串連以逗號分隔的緯度和經度值： `Text.From([coordinates]{1})&","&Text.From([coordinates]{0})`。 按一下 [確定]。
   
    如需資料分析運算式 (DAX) (包括 DAX 函數) 的詳細資訊，請瀏覽 [Power BI Desktop 中的 DAX 基礎](https://support.powerbi.com/knowledgebase/articles/554619-dax-basics-in-power-bi-desktop)。
   
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 新增自訂資料行](./media/powerbi-visualize/power_bi_connector_pbicustomlatlong.png)

10. 現在，中央窗格會顯示已填入值的新 LatLong 資料行。
    
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 自訂 LatLong 資料行](./media/powerbi-visualize/power_bi_connector_pbicolumnlatlong.png)
    
    如果您在新的資料行中收到錯誤，請確定在 [查詢設定] 下套用的步驟與下圖相同︰
    
    ![套用的步驟應該是來源、瀏覽、展開的文件、展開的 Document.Location、新增的自訂](./media/powerbi-visualize/power-bi-applied-steps.png)
    
    如果步驟不同，請刪除額外的步驟，然後再試一次新增自訂資料行。 

11. 按一下 [關閉並套用]  以儲存資料模型。
    
    ![Azure Cosmos DB Power BI 連接器的 Power BI 教學課程 - 關閉並套用](./media/powerbi-visualize/power_bi_connector_pbicloseapply.png)

<a id="build-the-reports"></a>
## <a name="build-the-reports"></a>建置報告
[Power BI Desktop 報告] 檢視可做為您開始建立報告以視覺化資料的起始點。  您可以將欄位拖放到 [報告]  畫布上，以建立報告。

![Power BI Desktop 報告檢視 - Power BI 連接器](./media/powerbi-visualize/power_bi_connector_pbireportview2.png)

在 [報告] 檢視中，您應該會看到：

1. [欄位]  窗格，您會在這裡看到資料模型清單，以及您可以用於報告的欄位。
2. [視覺效果]  窗格。 一份報告可以包含一或多個視覺效果。  請從 [視覺效果]  窗格中選擇適合本身需求的視覺化類型。
3. [報告]  畫布，您可以在此處建置報告的視覺效果。
4. [報告]  頁面。 您可以在 Power BI Desktop 中心增多個報告頁面。

以下說明建立簡單的互動式地圖檢視報告的基本步驟。

1. 在此處的範例中，我們將建立會顯示每個火山所在位置的地圖檢視。  在 [視覺效果]  窗格中，按一下地圖視覺類型，如上方的螢幕擷取畫面所強調顯示。  您應該會看到地圖視覺化類型繪製於 [報告]  畫布上。  [視覺效果]  窗格也應該會顯示一組與地圖視覺化類型相關的屬性。
2. 現在，將 LatLong 欄位從 [欄位] 窗格拖放到 [視覺效果] 窗格中的 [位置] 屬性上。
3. 接下來，將 [火山名稱] 欄位拖放到 [圖例]  屬性上。  
4. 然後，將 [高度] 欄位拖放到 [大小]  屬性上。  
5. 您現在應該會看到地圖視覺化顯示一組表示每個火山所在位置的泡泡，且泡泡的大小會與火山的高度相關聯。
6. 現在您已建立基本報告。  您可以新增更多視覺效果，以進一步自訂報告。  在本例中，我們新增 [火山類型] 交叉分析篩選器，讓報告更具互動性。  
   
    ![完成 Azure Cosmos DB 的 Power BI 教學課程後之最終 Power BI Desktop 報告的螢幕擷取畫面](./media/powerbi-visualize/power_bi_connector_pbireportfinal.png)
7. 在 [檔案] 功能表上按一下 [儲存]，並將檔案儲存為 PowerBITutorial.pbix。

## <a name="publish-and-share-your-report"></a>發佈和共用您的報告
若要共用您的報告，您必須有 PowerBI.com 中的帳戶。

1. 在 Power BI Desktop 中，按一下 [首頁]  功能區。
2. 按一下 [發佈] 。  系統會提示您輸入 PowerBI.com 帳戶的使用者名稱和密碼。
3. 認證經過驗證後，報告即會發佈至您選取的目的地。
4. 按一下 [開啟 Power BI 中的 'PowerBITutorial.pbix']  在 PowerBI.com 上查看與共用您的報表。
   
    ![成功發佈到 Power BI！ 在 Power BI 中開啟教學課程](./media/powerbi-visualize/power_bi_connector_open_in_powerbi.png)

## <a name="create-a-dashboard-in-powerbicom"></a>在 PowerBI.com 中建立儀表板
現在，您有一份報表可在 PowerBI.com 上共用。

當您從 Power BI Desktop發佈報告至 PowerBI.com 時，在您的 PowerBI.com 租用戶中會產生 [報表] 和 [資料集]。 例如，您將名為 **PowerBITutorial** 的報表發佈到 PowerBI.com，會在 PowerBI.com 的 [報表] 和 [資料集] 區段看到 PowerBITutorial。

   ![PowerBI.com 中新報表和資料集的螢幕擷取畫面](./media/powerbi-visualize/powerbi-reports-datasets.png)

若要建立可共用的儀表板，按一下 PowerBI.com 報表上的 [釘選即時頁面]  按鈕。

   ![PowerBI.com 中新報表和資料集的螢幕擷取畫面](./media/powerbi-visualize/power-bi-pin-live-tile.png)

依照 [從報表釘選圖格](https://powerbi.microsoft.com/documentation/powerbi-service-pin-a-tile-to-a-dashboard-from-a-report/#pin-a-tile-from-a-report) 的指示建立新的儀表板。 

您也可以對之前建立的儀表板進行臨機操作修改。 不過，建議您使用 Power BI Desktop 進行修改，再將報表重新發佈至 PowerBI.com。

<!-- ## Refresh data in PowerBI.com
There are two ways to refresh data, ad hoc and scheduled.

For an ad hoc refresh, simply click on the eclipses (…) by the **Dataset**, e.g. PowerBITutorial. You should see a list of actions including **Refresh Now**. Click **Refresh Now** to refresh the data.

![Screenshot of Refresh Now in PowerBI.com](./media/powerbi-visualize/power-bi-refresh-now.png)

For a scheduled refresh, do the following.

1. Click **Schedule Refresh** in the action list. 

    ![Screenshot of the Schedule Refresh in PowerBI.com](./media/powerbi-visualize/power-bi-schedule-refresh.png)
2. In the **Settings** page, expand **Data source credentials**. 
3. Click on **Edit credentials**. 
   
    The Configure popup appears. 
4. Enter the key to connect to the Azure Cosmos DB account for that data set, then click **Sign in**. 
5. Expand **Schedule Refresh** and set up the schedule you want to refresh the dataset. 
6. Click **Apply** and you are done setting up the scheduled refresh.
-->
## <a name="next-steps"></a>後續步驟
* 若要深入了解 Power BI，請參閱 [開始使用 Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/)。
* 若要深入了解 Azure Cosmos DB，請參閱 [Azure Cosmos DB 文件登陸頁面](https://azure.microsoft.com/documentation/services/cosmos-db/)。

