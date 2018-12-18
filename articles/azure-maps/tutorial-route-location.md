---
title: 使用 Azure 地圖服務尋找路線 | Microsoft Docs
description: 使用 Azure 地圖服務的景點路線
author: dsk-2015
ms.author: dkshir
ms.date: 10/02/2018
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 3bf1aa6d1b9bd65c28ef99ddbac71fb75daf99e7
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816713"
---
# <a name="route-to-a-point-of-interest-using-azure-maps"></a>使用 Azure 地圖服務的景點路線

本教學課程說明如何使用 Azure 地圖服務帳戶和路線規劃服務 SDK，來尋找景點路線。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用地圖控制項 API 建立新的網頁
> * 設定地址座標
> * 查詢路線規劃服務，以了解如何前往景點

## <a name="prerequisites"></a>必要條件

在繼續之前，請遵循上一個教學課程中的步驟來[建立 Azure 地圖服務帳戶](./tutorial-search-location.md#createaccount)，並[取得帳戶的訂用帳戶金鑰](./tutorial-search-location.md#getkey)。

<a id="getcoordinates"></a>

## <a name="create-a-new-map"></a>建立新的地圖

下列步驟顯示如何建立內嵌地圖控制項 API 的靜態 HTML 網頁。

1. 在您的本機機器上建立新檔案，並將其命名為 **MapRoute.html**。
2. 在檔案中新增下列 HTML 元件：

    ```HTML
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, user-scalable=no" />
        <title>Map Route</title>
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/css/atlas.min.css?api-version=1" type="text/css"/>
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
    HTML 標頭會內嵌 CSS 的資源位置和 Azure 地圖服務程式庫的 JavaScript 檔案。 HTML 檔案主體中的 script 區段將會包含用來存取 Azure 地圖服務 API 的內嵌 JavaScript 程式碼。

3. 在 HTML 檔案的 script 區塊中新增下列 JavaScript 程式碼。 將字串 **\<您的帳戶金鑰\>** 取代為您從地圖服務帳戶所複製的主要金鑰。

    ```JavaScript
    // Instantiate map to the div with id "map"
    var MapsAccountKey = "<your account key>";
    var map = new atlas.Map("map", {
        "subscription-key": MapsAccountKey
    });
    ```
    **atlas.Map** 是 Azure 地圖控制項 API 的元件，可供控制視覺化互動式網路地圖。

4. 儲存檔案並在瀏覽器中將其開啟。 至此，您已擁有可以進一步發展的基本地圖了。

   ![檢視基本地圖](./media/tutorial-route-location/basic-map.png)

## <a name="set-start-and-end-points"></a>設定起點和終點

在此教學課程中，將起點設在 Microsoft，並將終點設在西雅圖的加油站。

1. 在 **MapRoute.html** 檔案的同一個 script 區塊中，新增下列 JavaScript 程式碼以建立路線的起點和終點：

    ```JavaScript
    // Create the GeoJSON objects which represent the start and end point of the route
    var startPoint = new atlas.data.Point([-122.130137, 47.644702]);
    var startPin = new atlas.data.Feature(startPoint, {
        title: "Microsoft",
        icon: "pin-round-blue"
    });

    var destinationPoint = new atlas.data.Point([-122.3352, 47.61397]);
    var destinationPin = new atlas.data.Feature(destinationPoint, {
        title: "Contoso Oil & Gas",
        icon: "pin-blue"
    });
    ```
    此程式碼會建立兩個 [GeoJSON 物件](https://en.wikipedia.org/wiki/GeoJSON)，代表路線的起點和終點。

2. 新增下列 JavaScript 程式碼，以在地圖中新增起點和終點的標示：

    ```JavaScript
    // Fit the map window to the bounding box defined by the start and destination points
    var swLon = Math.min(startPoint.coordinates[0], destinationPoint.coordinates[0]);
    var swLat = Math.min(startPoint.coordinates[1], destinationPoint.coordinates[1]);
    var neLon = Math.max(startPoint.coordinates[0], destinationPoint.coordinates[0]);
    var neLat = Math.max(startPoint.coordinates[1], destinationPoint.coordinates[1]);
    map.setCameraBounds({
        bounds: [swLon, swLat, neLon, neLat],
        padding: 50
    });

    map.addEventListener("load", function () { 
        // Add pins to the map for the start and end point of the route
        map.addPins([startPin, destinationPin], {
            name: "route-pins",
            textFont: "SegoeUi-Regular",
            textOffset: [0, -20]
        });
    });
    ```
    **map.setCameraBounds** 會根據起點和終點座標來調整地圖視窗。 **map.addEventListener** 可確保新增至地圖的所有地圖函式，都會在地圖完整載入之後載入。 事件接聽程式內的 **map.addPins** API 會在地圖控制項中以視覺化元件的形式新增景點。

3. 儲存 **MapRoute.html** 檔案並重新整理瀏覽器。 現在地圖會以西雅圖作為中心，而且您會看到以圓形藍色圖釘標示的起點，和以藍色圖釘標示的終點。

   ![檢視標有起點和終點的地圖](./media/tutorial-route-location/map-pins.png)

<a id="getroute"></a>

## <a name="get-directions"></a>取得指示

本節說明如何使用地圖服務的路線規劃服務 API，尋找從給定起點到終點的路線。 路線規劃服務會提供 API 來規劃兩個位置之間「最快速」、「最短」、「最環保」或「驚心動魄」的路線。 它也可讓使用者使用 Azure 廣泛的歷史路況資料庫，並預測任何日期和時間的路線時間，以規劃日後的路線。 如需詳細資訊，請參閱[取得路線指示](https://docs.microsoft.com/rest/api/maps/route/getroutedirections)。 以下所有功能都應該新增至**地圖載入 eventListener 內**，以確保會在地圖完整載入之後載入。

1. 首先，在地圖上新增一個圖層來顯示路線路徑 (亦即 linestring)。 在 script 區塊中新增下列 JavaScript 程式碼。

    ```JavaScript
    // Initialize the linestring layer for routes on the map
    var routeLinesLayerName = "routes";
    map.addLinestrings([], {
        name: routeLinesLayerName,
        color: "#2272B9",
        width: 5,
        cap: "round",
        join: "round",
        before: "labels"
    });
    ```

2. 藉由將下列 Javascript 程式碼新增至指令碼區塊，來具現化用戶端服務。
    ```JavaScript
    var client = new atlas.service.Client(MapsAccountKey);
    ```

3. 新增下列程式碼區塊，以建構路由查詢字串。
    ```JavaScript
    // Construct the route query string
    var routeQuery = startPoint.coordinates[1] +
        "," +
        startPoint.coordinates[0] +
        ":" +
        destinationPoint.coordinates[1] +
        "," +
        destinationPoint.coordinates[0];
    ```

4. 若要取得路線，請將下列程式碼區塊新增至指令碼。 它會透過 [getRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/services.route?view=azure-iot-typescript-latest#getroutedirections) 方法來查詢 Azure 地圖服務路線服務，然後使用 [getGeoJsonRoutes](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.geojson.geojsonroutedirectionsresponse?view=azure-iot-typescript-latest#getgeojsonroutes) 將回應剖析為 GeoJSON 格式。 然後它會將所有回應線條全都新增到地圖上以呈現路線。 如需詳細資訊，您可以查看[在地圖上新增線條](./map-add-shape.md#addALine)。

    ```JavaScript
    // Execute the query then add the route to the map once a response is received  
    client.route.getRouteDirections(routeQuery).then(response => {
         // Parse the response into GeoJSON
         var geoJsonResponse = new atlas.service.geojson.GeoJsonRouteDirectionsResponse(response);

         // Get the first in the array of routes and add it to the map
         map.addLinestrings([geoJsonResponse.getGeoJsonRoutes().features[0]], {
             name: routeLinesLayerName
         });
    });
    ```

5. 儲存 **MapRoute.html** 檔案並重新整理網頁瀏覽器。 如果您成功連線到地圖服務的 API，您應該會看到類似下面的地圖。

    ![Azure 地圖控制項和路線規劃服務](./media/tutorial-route-location/map-route.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用地圖控制項 API 建立新的網頁
> * 設定地址座標
> * 查詢路線規劃服務以了解如何前往景點

您可以在這裡存取本教學課程的程式碼範例：

> [使用 Azure 地圖服務尋找路線](https://github.com/Azure-Samples/azure-maps-samples/blob/master/src/route.html)

下一個教學課程會示範如何建立具有限制 (例如，行進模式或貨物類型) 的路線查詢 ，然後在同一個地圖上顯示多條路線。

> [!div class="nextstepaction"]
> [尋找不同行進模式的路線](./tutorial-prioritized-routes.md)
