---
title: 使用 Azure 地圖服務進行搜尋 | Microsoft Docs
description: 使用 Azure 地圖服務來搜尋附近景點
author: dsk-2015
ms.author: dkshir
ms.date: 10/02/2018
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 761674c5839f0513532355116db07604f9e9d9dc
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816815"
---
# <a name="search-nearby-points-of-interest-using-azure-maps"></a>使用 Azure 地圖服務來搜尋附近景點

本教學課程說明如何使用 Azure 地圖服務來設定帳戶，然後使用地圖服務 API 來搜尋景點。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 Azure 地圖服務帳戶
> * 擷取地圖服務帳戶的主要金鑰
> * 使用地圖控制項 API 建立新的網頁
> * 使用地圖服務的搜尋服務來尋找附近景點

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com)。

<a id="createaccount"></a>

## <a name="create-an-account-with-azure-maps"></a>使用 Azure 地圖服務建立帳戶

使用下列步驟建立新的地圖服務帳戶：

1. 按一下 [Azure 入口網站](https://portal.azure.com)左上角的 [建立資源]。
2. 在 [搜尋 Marketplace] 方塊中，輸入 **Maps**。
3. 從 [結果] 中，選取 [地圖服務]。 按一下地圖下方顯示的 [建立] 按鈕。
4. 在 [建立地圖服務帳戶] 頁面上輸入下列值：
    * 新帳戶的 [名稱]。
    * 您想要使用於此帳戶的 [訂用帳戶]。
    * 此帳戶的 [資源群組] 名稱。 您可以選擇 [建立新的] 或 [使用現有的] 資源群組。
    * 選取 [資源群組位置]。
    * 閱讀 [授權] 和 [隱私權聲明]，然後選取核取方塊以接受條款。
    * 按一下 [ **建立** ] 按鈕。

    ![在入口網站中建立地圖服務帳戶](./media/tutorial-search-location/create-account.png)

<a id="getkey"></a>

## <a name="get-the-primary-key-for-your-account"></a>取得帳戶的主要金鑰

成功建立地圖服務帳戶後，擷取金鑰以便能查詢地圖服務 API。

1. 在入口網站中開啟地圖服務帳戶。
2. 在 [設定] 區段中，選取 [金鑰]。
3. 將 [主索引鍵] 複製到剪貼簿。 將其儲存在本機，以供本教學課程稍後使用。

    ![在入口網站中取得主索引鍵](./media/tutorial-search-location/get-key.png)

<a id="createmap"></a>

## <a name="create-a-new-map"></a>建立新的地圖

地圖控制項 API 是很方便的用戶端程式庫，可讓您輕鬆地將地圖服務整合到 Web 應用程式中。 它會隱藏裸機 REST 服務呼叫的複雜度，並使用可設定樣式和可自訂的元件提升生產力。 下列步驟顯示如何建立內嵌地圖控制項 API 的靜態 HTML 網頁。

1. 在您的本機機器上建立新檔案，並將其命名為 **MapSearch.html**。
2. 在檔案中新增下列 HTML 元件：

    ```HTML
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, user-scalable=no" />
        <title>Map Search</title>

        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/css/atlas.min.css?api-version=1" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/js/atlas.min.js?api-version=1"></script>
        <script src="https://atlas.microsoft.com/sdk/js/atlas-service.min.js?api-version=1"></script>

        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #map {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>

    <body>
        <div id="map"></div>
        <script>
            // Embed Map Control JavaScript code here
        </script>
    </body>

    </html>
    ```
    請注意，HTML 標頭包含 Azure 地圖控制項程式庫所裝載的 CSS 和 JavaScript 資源檔案。 請注意新增至 HTML 檔案 body 的 script 區段。 此區段會包含內嵌的 JavaScript 程式碼，以便存取 Azure 地圖服務 API。

3. 在 HTML 檔案的 script 區塊中新增下列 JavaScript 程式碼。 將字串 **\<您的帳戶金鑰\>** 取代為您從地圖服務帳戶所複製的主要金鑰。

    ```JavaScript
    // Instantiate map to the div with id "map"
    var MapsAccountKey = "<your account key>";
    var map = new atlas.Map("map", {
        "subscription-key": MapsAccountKey
    });
    ```
    此區段會為 Azure 地圖服務帳戶金鑰初始化地圖控制項 API。 **Atlas** 是包含 API 和相關視覺效果元件的命名空間。 **Atlas.Map** 可供控制視覺化互動式網路地圖。

4. 將變更儲存至檔案，並在瀏覽器中開啟 HTML 網頁。 這是您可以藉由呼叫 **atlas.map** 和使用帳戶金鑰來建立的最基本地圖。

   ![檢視地圖](./media/tutorial-search-location/basic-map.png)

<a id="usesearch"></a>

## <a name="add-search-capabilities"></a>新增搜尋功能

本節說明如何使用地圖服務的搜尋 API 在地圖上尋找景點。 這是針對開發人員所設計的 RESTful API，用於搜尋地址、景點及其他地理資訊。 搜尋服務會將經緯度資訊指派給指定的地址。 以下說明的**服務模組**可讓您使用地圖服務搜尋 API 來搜尋位置。

### <a name="service-module"></a>服務模組

1. 在地圖中新增圖層，以顯示搜尋結果。 在指令碼區塊中，將下列 JavaScript 程式碼新增至初始化地圖的程式碼後面。

    ```JavaScript
    // Initialize the pin layer for search results to the map
    var searchLayerName = "search-results";
    ```

2. 若要具現化用戶端服務，請在指令碼區塊中，將下列 JavaScript 程式碼新增至初始化地圖的程式碼後面。

    ```JavaScript
    var client = new atlas.service.Client(MapsAccountKey);
    ```

3. 地圖上的所有函式均應在地圖載入之後載入。 您可以藉由在地圖 eventListener 區塊內放入所有地圖函式來確定這一點。 新增下列幾行程式碼，以將 [eventListener](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addeventlistener) 新增至地圖，可確保地圖可在新增功能之前完整載入。
    
    ```JavaScript
         map.addEventListener("load", function() {
         });
    ```

4. 在**地圖負載 eventListener 內**新增下列指令碼區塊以建置查詢。 它會使用模糊搜尋服務，這是搜尋服務的基本搜尋 API。 模糊搜尋服務可處理大部分的模糊輸入，例如地址或景點 (POI) 權杖的任意組合。 它會在指定的半徑範圍內搜尋附近的加油站。 然後，回應剖析為 GeoJSON 格式，並轉換成點功能，新增為地圖上的圖釘。 指令碼的最後一個部分會使用地圖的 [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/models.cameraboundsoptions?view=azure-iot-typescript-latest) 屬性，為地圖新增觀景窗界限。

    ```JavaScript

            // Execute a POI search query then add pins to the map for each result once a response is received
            client.search.getSearchFuzzy("gasoline station", {
            lat: 47.6292,
            lon: -122.2337,
            radius: 100000
            }).then(response => {
       
            // Parse the response into GeoJSON 
            var geojsonResponse = new atlas.service.geojson.GeoJsonSearchResponse(response);

            // Create the point features that will be added to the map as pins
            var searchPins = geojsonResponse.getGeoJsonResults().features.map(poiResult => {
               var poiPosition = [poiResult.properties.position.lon, poiResult.properties.position.lat];
               return new atlas.data.Feature(new atlas.data.Point(poiPosition), {
                name: poiResult.properties.poi.name,
                address: poiResult.properties.address.freeformAddress,
                position: poiPosition[1] + ", " + poiPosition[0]
               });
            });

            // Add pins to the map for each POI
            map.addPins(searchPins, {
               name: searchLayerName,
               cluster: false, 
               icon: "pin-round-darkblue" 
            });

            // Set the camera bounds
            map.setCameraBounds({
               bounds: geojsonResponse.getGeoJsonResults().bbox,
               padding: 50
            );
        });
    ```
5. 儲存 **MapSearch.html** 檔案並重新整理瀏覽器。 您現在應該會看到地圖以西雅圖作為中心，並以藍色圖釘標示該區域內的汽油站位置。

   ![檢視地圖與搜尋結果](./media/tutorial-search-location/pins-map.png)

6. 您可以在瀏覽器中輸入下列 HTTPRequest，以查看地圖所呈現的原始資料。 將 \<您的帳戶金鑰\> 取代為主要金鑰。

   ```http
   https://atlas.microsoft.com/search/fuzzy/json?api-version=1.0&query=gasoline%20station&subscription-key=<your account key>&lat=47.6292&lon=-122.2337&radius=100000
   ```

至此，MapSearch 網頁可顯示模糊搜尋查詢所傳回的景點位置。 讓我們新增一些互動式功能和位置的詳細資訊。

## <a name="add-interactive-data"></a>新增互動式資料

我們到目前為止所建立的地圖只會查看搜尋結果的經緯度資料。 但如果您看一下地圖服務的搜尋服務所傳回的原始 JSON，就會看到它包含每個加油站的其他資訊，包括名稱和街道地址。 您可以使用互動式快顯方塊將該資料合併到地圖中。

1. 在 script 區塊中新增下列幾行，以建立搜尋服務所傳回之景點的快顯視窗：

    ```JavaScript
    // Add a popup to the map which will display some basic information about a search result on hover over a pin
    var popup = new atlas.Popup();
    map.addEventListener("mouseover", searchLayerName, (e) => {
        var popupContentElement = document.createElement("div");
        popupContentElement.style.padding = "5px";

        var popupNameElement = document.createElement("div");
        popupNameElement.innerText = e.features[0].properties.name;
        popupContentElement.appendChild(popupNameElement);

        var popupAddressElement = document.createElement("div");
        popupAddressElement.innerText = e.features[0].properties.address;
        popupContentElement.appendChild(popupAddressElement);

        var popupPositionElement = document.createElement("div");
        popupPositionElement.innerText = e.features[0].properties.position;
        popupContentElement.appendChild(popupPositionElement);

        popup.setPopupOptions({
            position: e.features[0].geometry.coordinates,
            content: popupContentElement
        });

        popup.open(map);
    });
    ```
    API **atlas.Popup** 可提供錨定在地圖上所需位置的資訊視窗。 此程式碼片段會設定快顯視窗的內容和位置。 它也會將事件接聽程式新增至會等候_滑鼠_變換快顯視窗 `map` 控制項。

2. 儲存檔案並重新整理瀏覽器。 現在，當您將滑鼠停留在任何搜尋圖釘上時，瀏覽器中的地圖會顯示資訊快顯視窗。

    ![Azure 地圖控制項和搜尋服務](./media/tutorial-search-location/popup-map.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 Azure 地圖服務建立帳戶
> * 取得帳戶的主要金鑰
> * 使用地圖控制項 API 建立新網頁
> * 使用搜尋服務來尋找附近景點

您可以在這裡存取本教學課程的程式碼範例：

> [使用 Azure 地圖服務搜尋位置](https://github.com/Azure-Samples/azure-maps-samples/blob/master/src/search.html)

下一個教學課程會示範如何顯示兩個位置之間的路線。

> [!div class="nextstepaction"]
> [前往目的地的路線](./tutorial-route-location.md)
