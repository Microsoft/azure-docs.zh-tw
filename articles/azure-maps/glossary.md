---
title: Azure 地圖服務詞彙 | Microsoft Docs
description: 與 Azure 地圖服務、位置服務，以及 GIS 相關的常用字詞詞彙。
author: rbrundritt
ms.author: richbrun
ms.date: 09/18/2018
ms.topic: resource
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: d220fea64f860ebe5b660f7ebe5ad7db7aec4534
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46369855"
---
# <a name="glossary"></a>詞彙

以下是經常搭配 Azure 地圖服務使用的字詞清單。

## <a name="a"></a>A

<a name="address-validation"></a> **Address validation** \(地址驗證\)：驗證地址是否存在的程序。

<a name="advanced-routing"></a> **Advanced routing** \(進階路線規劃\)：能使用道路路線規劃資料執行進階作業 (例如計算可抵達範圍 (等時線)、距離矩陣，以及批次路線要求) 的服務集合。

<a name="aerial-imagery"></a> **Aerial imagery** \(空照圖影像\)：請參閱[衛星影像](#satellite-imagery)。 

<a name="along-a-route-search"></a> **Along a route search** \(沿路線搜尋\)：能相對於路線路徑尋找位於指定繞道時間或距離範圍內之資料的空間查詢。

<a name="altitude"></a> **Altitude** \(高度\)：位於參考平面上之某點的高度或垂直相對高度。 高度的測量是以指定的參考數據為基礎，例如平均海平面。 請參閱＜相對高度＞。

<a name="ambiguous"></a> **Ambiguous** \(不明確\)：資料分類中的不確定狀態，適用於針對指定屬性可以適當地為物件指派兩個或以上之值的情況。 例如，對 "CA" 進行地理編碼時，會傳回兩個不明確的結果："Canada" 和 "California"，因為 "CA"針對這兩者分別為國碼 (地區碼) 和州/省代碼。 

<a name="annotation"></a> **Annotation** \(註釋\)：顯示在地圖上以為使用者提供資訊的文字或圖形。 註釋可以識別或描述特定的地圖實體、提供地圖上某個區域的一般資訊，或是提供有關地圖本身的資訊。

<a name="antimeridian"></a> **Antimeridian** \(對向子午線\)：也稱為 180<sup></sup> 度經線，這是經度 -180 度和 180 度的交會點。 其為地球本初子午線的相反。

<a name="application-programming-interface-api"></a> **Application Programming Interface (API)** \(應用程式開發介面\)：可讓開發人員建立應用程式的規格。

<a name="api-key"></a> **API key** \(API 金鑰\)：請參閱＜Azure 地圖服務金鑰＞。

<a name="area-of-interest-aoi"></a> **Area of Interest (AOI)** \(感興趣區域\)：用來針對地圖或資料庫生產定義焦點區域的範圍。

<a name="asset-tracking"></a> **Asset tracking** \(資產追蹤\)：追蹤如人員、車輛或其他物件等資產之位置的程序。

<a name="asynchronous-request"></a> **Asynchronous request** \(非同步要求\)：此 HTTP 要求會開啟連線並向伺服器提出要求，並會在伺服器傳回非同步要求的識別碼之後關閉連線。 伺服器會持續處理要求，而使用者可以使用該識別碼來查詢其狀態。 當要求處理完成之後，使用者便可以下載回應。 此類型的要求通常是用於長時間執行的處理程序。

<a name="autocomplete"></a> **Autocomplete** \(自動完成\)：應用程式中可透過使用者所輸入的字母，預測出其想要輸入之字詞的功能。 

<a name="autosuggest"></a> **Autosuggest** \(自動建議\)：應用程式中可預測出使用者想要輸入之內容邏輯可能性的功能。

<a name="azure-location-based-services-lbs"></a> **Azure Location Based Services (LBS)** \(Azure 位置服務\)：Azure 地圖服務處於預覽狀態時的舊版名稱。

<a name="azure-maps-key"></a> **Azure Maps key** \(Azure 地圖服務金鑰\)：Azure 地圖服務金鑰是用來驗證使用者的 Azure 地圖服務應用程式或服務要求的唯一字串。 

## <a name="b"></a>B

<a name="base-map"></a> **Base map** \(基本地圖\)：地圖應用程式顯示背景參考資訊 (例如道路、地標及政治界線) 的部分。

<a name="batch-request"></a> **Batch request** \(批次要求\)：將多個要求結合成單一要求的程序。

<a name="bearing"></a> **Bearing** \(方位\)：某點與另一個點之間的相對水平方向。 這是以相對於北方的角度來表示，依順時針方向從 0 度到 360 度。 

<a name="boundary"></a> **Boundary** \(界線\)：分隔相鄰政治實體 (例如國家/地區、行政區及房地產) 的線條或多邊形。 界線為可能或可能不會依循實體特徵 (例如河流、山或牆) 的線條。

<a name="bounds"></a> **Bounds** \(邊界\)：請參閱[週框方塊](#bounding-box)。

<a name="bounding-box"></a> **Bounding box** \(週框方塊\)：用來代表地圖上矩形區域的座標集合。 

## <a name="c"></a>C

<a name="cadastre"></a> **Cadastre** \(土地清冊\)：已註冊土地及房產的記錄。 另請參閱[地塊](#parcel)。

<a name="camera"></a> **Camera** \(觀景窗\)：以互動式地圖控制的內容來說，觀景窗會定義地圖的視野。 觀景窗的檢視區會取決於數個地圖參數：中心、縮放層級、傾斜度、方位。 

<a name="centroid"></a> **Centroid** \(距心\)：特徵的幾何中心。 線條的距心將會是該線條的中間點，而多邊形的距心則會是其區域的中心點。

<a name="choropleth-map"></a> **Choropleth map** \(分區著色圖\)：依顯示於地圖上之統計變數的測量，以不同色彩強調各區域的主題地圖。 例如，依據每個美國州份相對於其他州的人口，以各種色彩標示其界限。

<a name="concave-hull"></a> **Concave hull** \(凹殼\)：代表會包含指定資料集中所有形狀之可能凹面幾何的形狀。 所產生的形狀類似於將資料包覆在塑膠膜中並對它加熱，這會導致間距較長之兩點間的線條朝內向其他資料點縮緊。

<a name="consumption-model"></a> **Consumption model** \(耗用模型\)：能定義車輛耗用燃料或電力之速率的資訊。 另請參閱[耗用模型文件](consumption-model.md)。

<a name="control"></a> **Control** \(控制項\)：自封式或可重複使用的元件，其包含能定義介面之行為集合的圖形化使用者介面。 以地圖控制項為例，其通常是使用者介面載入互動式地圖的部分。

<a name="convex-hull"></a> **Convex hull** \(凸殼\)：代表會包含指定資料集中所有形狀之最小凸面幾何的形狀。 所產生的形狀類似於以橡皮筋纏住資料集。

<a name="coordinate"></a> **Coordinate** \(座標\)：包含用來代表地圖上位置的經度和緯度值。

<a name="coordinate-system"></a> **Coordinate system** \(座標系統\)：用來定義 2D 或 3D 空間中之點位置的參考架構。

<a name="country-code"></a> **Country code** \(國碼 (地區碼)\)：國家/地區的 ISO 標準唯一識別碼。 ISO2 為國家/地區的二字元代碼 (例如 US)，而 ISO3 則代表三字元代碼 (例如 USA)。

<a name="country-subdivision"></a> **Country subdivision** \(國家/地區從屬行政區\)：國家/地區的第一級從屬行政區，通常稱為州或省 (市)。

<a name="country-secondary-subdivision"></a> **Country secondary subdivision** \(國家/地區第二從屬行政區\)：國家/地區的第二級從屬行政區，通常稱為郡 (縣市)。

<a name="country-tertiary-subdivision"></a> **Country tertiary subdivision** \(國家/地區第三從屬行政區\)：國家/地區的第三級從屬行政區，通常是區之類的具名區域。

<a name="cross-street"></a> **Cross street** \(交叉路口\)：兩條或多條街道的交會點。

<a name="cylindrical-projection"></a> **Cylindrical projection** \(圓柱形投影\)：能將球狀體或球體的點轉換成正切或正割圓柱的投影。 圓柱接著會由上而下切開，並壓扁為平面。

## <a name="d"></a>D

<a name="datum"></a> **Datum** \(數據\)：測量系統的參考規格，此系統可為平面上的座標位置 (水平數據) 或平面上方或下方的高度 (垂直數據)。

<a name="dbf-file"></a> **DBF file** \(DBF 檔案\)：搭配形狀檔 (SHP) 使用的資料庫檔案格式。

<a name="degree-minutes-seconds-dms"></a> **Degree Minutes Seconds (DMS)** \(度分秒\)：用於描述緯度和經度的測量單位。 一度為圓形 1/360<sup></sup> 的大小。 一度會進一步分割成 60 分鐘，而一分鐘則會分割成 60 秒。

<a name="delaunay-triangulation"></a> **Delaunay triangulation** \(德洛涅三角剖分\)：從點的資料集建立連續且不重疊之三角形網狀結構的技術。 每個三角形結構內都不會包含資料集中的點。

<a name="demographics"></a> **Demographics** \(人口統計\)：人類人口的統計特性 (例如年齡、生育率和收入)。

<a name="destination"></a> **Destination** \(目的地\)：某人要移動至的終點或位置。

<a name="digital-elevation-model-dem"></a> **Digital Elevation Model (DEM)** \(數值高程模型\)：相對於平面之高度值的資料集，使用某個共同數據依固定間隔擷取自某個區域。 DEM 通常是用來代表地勢。

<a name="dijkstra's-algorithm"></a> **Dijkstra's algorithm** \(戴克斯特拉演算法\)：能查看網路的連接性以找出兩點之間最短路徑的演算法。

<a name="distance-matrix"></a> **Distance matrix** \(距離矩陣\)：包含一組來源與目的地之旅行時間和距離資訊的矩陣。 

## <a name="e"></a>E

<a name="elevation"></a> **Elevation** \(相對高度\)：位於參考平面或數據 (通常為平均海平面) 上方或下方之點或物件的垂直距離。 相對高度通常指的是土地的垂直高度。

<a name="envelope"></a> **Envelope** \(外殼\)：請參閱[週框方塊](#bounding-box)。

<a name="extended-postal-code"></a> **Extended postal code** \(擴充郵遞區號\)：可能包含額外資訊的郵遞區號。 以美國為例，其郵遞區號為五位數，但擴充郵遞區號 (也稱為 zip+4) 則會額外包含四個數字。 這些額外的數字是用來識別五位數投遞區域內的地理區段 (例如城市街區、集合住宅或郵政信箱)，以協助提升郵件分類和投遞的效率。

<a name="extent"></a> **Extent** \(範圍\)：請參閱[週框方塊](#bounding-box)。

## <a name="f"></a>F

<a name="federated-authentication"></a> **Federated authentication** \(同盟驗證\)：允許在多個網頁和行動應用程式上使用單一登入/驗證機制的驗證方式。 

<a name="feature"></a> **Feature** \(特徵\)：結合幾何與額外中繼資料資訊的物件。 

<a name="feature-collection"></a> **Feature collection** \(特徵集合\)：特徵物件的集合。

<a name="find-along-route"></a> **Find along route** \(沿路線尋找\)：能相對於路線路徑尋找位於指定繞道時間或距離範圍內之資料的空間查詢。

<a name="find-nearby"></a> **Find nearby** \(尋找鄰近地點\)：能從某個點沿固定直線距離 (如烏鴉飛行般) 搜尋的空間查詢。

<a name="fleet-management"></a> **Fleet management** \(車隊管理\)：商用車輛 (例如汽車、卡車、船隻及飛機) 的管理。 車隊管理可以包含數種功能，例如動產融資、維護、車載資通訊 (追蹤與診斷)，以及駕駛員、速度、燃料及健康與安全管理。 車隊管理為仰賴運輸之公司所使用的程序，以將風險降到最低並減少整體運輸與員工成本，同時確保符合政府法規的規範。

<a name="free-flow-speed"></a> **Free flow speed** \(自由車流速率\)：理想情況下的預期自由車流速率。 通常為規定的速度限制。

<a name="free-form-address"></a> **Free form address** \(自由格式地址\)：以單一文字行代表的完整地址。

<a name="fuzzy-search"></a> **Fuzzy search** \(模糊搜尋\)：接受可能為地址或景點之自由格式文字字串的搜尋。 

## <a name="g"></a>G

<a name="geocode"></a> **Geocode** \(地理代碼\)：已轉換成可用來在地圖上顯示特定位置之座標的地址或位置。 

<a name="geocoding"></a> **Geocoding** \(地理編碼\)：也稱為前向地理編碼，這是將位置的地址轉換成座標的程序。 

<a name="geodesic-path"></a> **Geodesic path** \(測地路徑\)：曲面上兩點之間的最短路徑。 在 Azure 地圖服務上呈現時，基於麥卡托投影的原因，此路徑會以曲線的形式呈現。

<a name="geofence"></a> **Geofence** \(地理柵欄\)：已定義的地理區域，可用來在裝置進入或離開該區域時觸發事件。

<a name="geojson"></a> **GeoJSON**：常見的 JSON 型檔案，可用來儲存地理向量資料，例如點、線條及多邊形。 **注意**：Azure 地圖服務使用的是 GeoJSON 的擴充版本，如[這裡所述](extend-geojson.md)。

<a name="geometry"></a> **Geometry** \(幾何\)：代表點、線條或多邊形等空間物件。

<a name="geometrycollection"></a> **GeometryCollection**：幾何物件的集合。

<a name="geopol"></a> **GeoPol**：指的是地緣政治性的敏感性資料，例如有爭議的邊界和地點名稱。

<a name="georeference"></a> **Georeference** \(地理座標參考\)：將地理資料或影像與已知座標系統配對的程序。 此程序可能會對資料進行移位、旋轉、縮放或扭曲。

<a name="georss"></a> **GeoRSS**：能將空間資料新增至 RSS 摘要的 XML 擴充功能。

<a name="gis"></a> **GIS**："Geographic Information System" \(地理資訊系統\) 的縮寫。 這是經常用來描述地圖產業的字眼。

<a name="gml"></a> **GML**：也稱為 "Geography Markup Language" \(地理標記語言\)。 可用來儲存空間資料的 XML 檔案擴充功能。

<a name="gps"></a> **GPS**：也稱為 "Global Positioning System" \(全球定位系統\)，這是用來判斷裝置在地球上之位置的衛星系統。 環繞地球移動的衛星會傳輸訊號，讓位於地球上任何位置的 GPS 接收器都能透過三邊測量計算出自己的位置。

<a name="gpx"></a> **GPX**：也稱為 GPS eXchange 格式，這是通常會從 GPS 裝置建立的 XML 檔案格式。  

<a name="great-circle-distance"></a> **Great-circle distance** \(大圓距離\)：球體上兩點之間的最短距離。

<a name="greenwich-mean-time-gmt"></a> **Greenwich Mean Time (GMT)** \(格林威治標準時間\)：本初子午線的時間。本初子午線會通過位於英國格林威治的皇家天文台。

<a name="guid"></a> **GUID**：全域唯一識別碼。 用來唯一識別介面、類別、類型程式庫、元件類別或記錄的字串。

## <a name="h"></a>H

<a name="haversine-formula"></a> **Haversine formula** \(半正矢公式\)：經常用來計算球體上兩點間大圓距離的公式。

<a name="hd-maps"></a> **HD maps** \(HD 地圖\)：也稱為高解析度地圖，包含自動駕駛所需的高畫質道路網路資訊，例如車道線、交通標誌，以及交通號誌。

<a name="heading"></a> **Heading** \(方向\)：某個物體所指向或面向的方向。 另請參閱[方位](#heading)。

<a name="heatmap"></a> **Heatmap** \(熱度圖\)：以一系列色彩來代表特定區域中點密度的資料視覺效果。 另請參閱[主題地圖](#thermatic-map)。

<a name="hybrid-imagery"></a> **Hybrid imagery** \(混疊影像\)：上方疊有道路資料和標籤的衛星或空照圖影像。

## <a name="i"></a>I

<a name="iana"></a> **IANA**："Internet Assigned Numbers Authority" \(美國網際網路號碼分配局\) 的縮寫。 監督全域 IP 位址配置的非營利組織。

<a name="isochrone"></a> **Isochrone** \(等時線\)：等時線能定義出從特定位置朝任何方向，以某種運輸模式在指定時間內所能移動到的區域。 另請參閱[可抵達範圍](#reachable-range)。

<a name="isodistance"></a> **Isodistance** \(等距線\)：針對指定位置，等距線能定義出朝任何方向以某個運輸模式在指定距離內所能移動到的區域。 另請參閱[可抵達範圍](#reachable-range)。

## <a name="k"></a>K

<a name="kml"></a> **KML**：也稱為 "Keyhole Markup Language"，這是用來儲存地理向量資料 (例如點、線條及多邊形) 的常見 XML 檔案格式。 

## <a name="l"></a>L

<a name="landsat"></a> **Landsat** \(大地衛星\)：由 NASA 所開發的多光譜、環繞地球移動的衛星，能收集土地影像以用於各種產業 (例如農業、林業及地圖編制)。

<a name="latitude"></a> **Latitude** \(緯度\)：從赤道朝北方或南方測量、以度數為單位的角距離。

<a name="level-of-detail"></a> **Level of detail** \(詳細資料等級\)：請參閱[縮放層級](#zoom-level)。

<a name="lidar"></a> **Lidar** \(光學雷達\)："light detection and ranging" \(光線偵測與測距\) 的縮寫。 這個遠端感應技術會使用雷射來測量反射面的距離。

<a name="linear-interpolation"></a> **Linear interpolation** \(線性插補\)：使用已知值之間的線性距離來預估未知的值。

<a name="linestring"></a> **LineString**：用來代表線條的幾何。 也稱為聚合線條。 

<a name="localization"></a> **Localization** \(當地語系化\)：對不同語言及文化特性 (Culture) 的支援。

<a name="logistics"></a> **Logistics** \(物流\)：以協調的方式移動人員、車輛、補給物或資產的程序。

<a name="longitude"></a> **Longitude** \(經度\)：從本初子午線朝東方或西方測量、以度數為單位的角距離。

## <a name="m"></a>M

<a name="map-tile"></a> **Map Tile** \(地圖圖格\)：代表地圖畫布分割區的矩形映像。 如需詳細資訊，請參閱[縮放層級和圖格格線](zoom-levels-and-tile-grid.md)文件。

<a name="marker"></a> **Marker** \(標記\)：也稱為釘選或圖釘，這是代表地圖上位置的圖示。

<a name="mercator-projection"></a> **Mercator projection** \(麥卡托投影\)：這個圓柱形地圖投影現已成為航海用途的標準地圖投影，因為它能將固定路線的線條 (稱為恆向線) 表示為能保留經線角度的直線區段。 與地球表面的真實佈局相比，所有平面的地圖投影都會扭曲地圖形狀或大小。 麥卡托投影會放大遠離赤道的區域，因此當您接近兩極時，地圖上較小的區域看起來將會比較大。 

<a name="multilinestring"></a> **MultiLineString**：代表 LineString 物件集合的幾何。 

<a name="multipoint"></a> **MultiPoint**：代表 Point 物件集合的幾何。

<a name="multipolygon"></a> **MultiPolygon**：代表 Polygon 物件集合的幾何。 例如，若要顯示夏威夷的界線，每個島嶼都會具有多邊形的外框，因此夏威夷的界線就會是 MultiPolygon。

<a name="municipality"></a> **Municipality** \(自治區\)：縣市或鄉鎮。 

<a name="municipality-subdivision"></a> **Municipality subdivision** \(自治區從屬行政區\)：自治區的從屬行政區，例如社區或是當地區域名稱 (例如「市中心」)。

## <a name="n"></a>N

<a name="navigation-bar"></a> **Navigation bar** \(瀏覽列\)：地圖上用來調整縮放層級、傾斜度、旋轉，以及切換基本地圖圖層的控制項集合。

<a name="nearby-search"></a> **Nearby search** \(鄰近搜尋\)：能從某個點沿固定直線距離 (如烏鴉飛行般) 搜尋的空間查詢。

<a name="neutral-ground-truth"></a> **Neutral Ground Truth** \(自然地面真相\)：能以其所代表之區域的正式語言及本機指令碼 (若可用) 來轉譯標籤的地圖。

## <a name="o"></a>O

<a name="origin"></a> **Origin** \(來源\)：使用者所在的起點或位置。

## <a name="p"></a>P

<a name="panning"></a> **Panning** \(移動瀏覽\)：在維持固定縮放層級的同時，將地圖朝任何方向移動的程序。

<a name="parcel"></a> **地塊**：土地或房產界線的平面圖。

<a name="pitch"></a> **傾斜度**：地圖相對於垂直的傾斜程度，其中 0 為垂直向下俯視地圖。

<a name="point"></a> **Point** \(點\)：代表地圖上單一位置的幾何。 

<a name="points-of-interest-poi"></a> **Points of interest (POI)** \(景點\)：商業場所、地標或熱門的感興趣位置。

<a name="polygon"></a> **Polygon** \(多邊形\)：代表地圖上某個區域的立體幾何。 

<a name="polyline"></a> **Polyline** \(聚合線條\)：用來代表線條的幾何。 也稱為 LineString。 

<a name="position"></a> **Position** \(位置\)：某個點的經度、緯度及高度 (x,y,z 座標)。

<a name="post-code"></a> **Post code** \(郵區編號\)：請參閱[郵遞區號](#postal-code)。

<a name="postal-code"></a> **Postal code** \(郵政編碼\)：具有特定格式的一系列字母或數字 (或兩者)，由國家/地區的郵政服務用來將地理區域分割成較小的區域，以簡化郵件的投遞。

<a name="prime-meridian"></a> **Prime meridian** \(本初子午線\)：代表經度 0 度的經度線。 一般而言，經度的值會隨著朝西方行進而逐漸遞減 (直到 180 度為止)，並會隨著朝東方行進而逐漸遞增 (直到 -180 度為止)。 

<a name="prj"></a> **PRJ**：經常伴隨形狀檔檔案出現的文字檔，其包含資料集所在之投影座標系統的相關資訊。

<a name="projection"></a> **Projection** \(投影\)：以地圖投影 (例如橫麥卡托投影、阿爾伯斯投影及羅賓森投影) 為基礎的投影座標系統。 它們能將地球球面的地圖投影到 2D 笛卡兒座標平面。 投影的座標系統有時會被稱為「地圖投影」。

## <a name="q"></a>Q

<a name="quadkey"></a> **Quadkey**：適用於四元樹並排系統內之圖格的基底 4 位址索引。 如需詳細資訊，請參閱[縮放層級和圖格格線](zoom-levels-and-tile-grid.md)文件。

<a name="quadtree"></a> **Quadtree** \(四元樹\)：其中的每個節點都具有四個子系的資料結構。 Azure 地圖服務的並排系統使用的是四元樹結構，這表示當使用者將地圖放大一個層級時，每個地圖圖格都會分割成四個子圖格。  如需詳細資訊，請參閱[縮放層級和圖格格線](zoom-levels-and-tile-grid.md)文件。

<a name="queries-per-second-qps"></a> **Queries Per Second (QPS)** \(每秒查詢數目\)：在一秒內可以對服務或平台做出查詢或要求的數目。 

## <a name="r"></a>R

<a name="radial-search"></a> **Radial search** \(輻射式搜尋\)：能從某個點沿固定直線距離 (如烏鴉飛行般) 搜尋的空間查詢。 

<a name="raster-data"></a> **Raster data** \(點陣資料\)：組織成資料列和資料行 (或圖格) 的資料格 (或像素) 矩陣，其中每個資料格都包含代表資訊 (例如溫度) 的值。 點陣包含數位空照圖相片、來自衛星的影像、數位圖片，以及掃瞄的地圖。

<a name="raster-layer"></a> **Raster layer** \(點陣圖層\)：由點陣影像所組成的圖格圖層。

<a name="reachable-range"></a> **Reachable range** \(可抵達範圍\)：可抵達範圍能定義出從某個位置朝任何方向，以某種旅行運輸模式在指定時間或距離內所能移動到的區域。 另請參閱[等時線](#isochrone)和[等距線](#isodistance)。

<a name="remote-sensing"></a> **Remote sensing** \(遠端感應\)：從遠處收集並解譯感應器資料的程序。

<a name="rest-service"></a> **REST service** \(REST 服務\)：REST 是 "Representational State Transfer" \(具象狀態傳輸\) 的縮寫。 REST 服務是一種 URL 型 Web 服務，其仰賴基本 Web 技術來進行通訊，最常見的方式為 HTTP GET 和 POST 要求。 與傳統的 SOAP 型服務相比，這些類型的服務通常較為快速，體積也較小。

<a name="reverse-geocode"></a> **Reverse geocode** \(反向地理代碼\)：透過座標判斷出其在地圖上所代表之地址的程序。

<a name="reproject"></a> **Reproject** \(重新投影\)：請參閱[轉換](#transformation)。

<a name="rest-service"></a> **REST service** \(REST 服務\)："Representational State Transfer" \(具象狀態傳輸\) 的縮寫。 用來在非集中式分散環境中的對等之間交換資訊的架構。 REST 允許位於不同電腦上的程式，以獨立於作業系統或平台的方式互相通訊，方法是傳送超文字傳輸通訊協定 (HTTP) 要求至統一資源定位器 (URL) 並取回資料。

<a name="route"></a> **Route** \(路線\)：兩個或更多位置之間的路徑，其也可能包含其他資訊，例如針對路線上導航點的指示。

<a name="requests-per-second-rps"></a> **Requests Per Second (RPS)** \(每秒要求數目\)：請參閱[每秒查詢數目 (QPS)](#queries-per-second-qps)。 

<a name="rss"></a> **RSS**：根據來源的不同，可為 "Really Simple Syndication" \(真正簡易新聞訂閱方式\)、"Resource Description Framework (RDF) Site Summary" \(資源描述架構 (RDF) 網站摘要\)，或 "Rich Site Summary" \(具意網站摘要\) 的縮寫。 它是用來共用不同網站上內容的簡易、結構化 XML 格式。 RSS 文件會包含重要的中繼資料元素，例如作者、日期、標題、簡短描述，以及超文字連結。 此資訊能協助使用者 (或 RSS 發行者服務) 決定有哪些內容值得他們做出進一步的調查。

## <a name="s"></a>S

<a name="satellite-imagery"></a> **Satellite imagery** \(衛星影像\)：由飛機和衛星朝正下方捕捉到的影像。

<a name="software-development-kit-sdk"></a> **Software development kit (SDK)** \(軟體開發套件\)：由文件、範例程式碼，以及範例應用程式組成的集合，可協助開發人員使用 API來建置應用程式。

<a name="shapefile-shp"></a> **Shapefile (SHP)** \(形狀檔\)：也稱為 ESRI 形狀檔，這是用來儲存地理特徵的位置、形狀及特質的向量資料儲存格式。 形狀檔會以相關檔案集合的形式儲存。

<a name="spherical-mercator-projection"></a> **Spherical Mercator projection** \(球形麥卡托投影\)：請參閱 [Web 麥卡托](#web-mercator)。 

<a name="spatial-query"></a> **Spatial query** \(空間查詢\)：對執行空間作業的服務所提出的要求。 例如輻射式搜尋或沿路線搜尋。

<a name="spatial-reference"></a> **Spatial reference** \(空間參考\)：以座標為基礎的當地、區域或全球系統，用來精確地找出地理實體。 它能定義用來將地圖座標與真實世界位置相關聯的座標系統。 空間參考能確保來自不同圖層或來源的空間資料能順利整合，以達成準確地檢視或分析。 Azure 地圖服務會使用 [EPSG:3857](https://epsg.io/3857) \(英文\) 座標參考系統，並針對輸入幾何資料使用 WGS 84。 

<a name="sql-spatial"></a> **SQL spatial** \(SQL 空間\)：指的是 SQL Azure 和 SQL Server 2008 及更新版本所內建的空間功能。 此空間功能也會以 .NET 程式庫的形式提供，並可獨立於 SQL Server 使用。 如需詳細資訊，請參閱[空間資料 (SQL Server) 文件](https://docs.microsoft.com/sql/relational-databases/spatial/spatial-data-sql-server)。

<a name="synchronous-request"></a> **Synchronous request** \(同步要求\)：會開啟連線並等候回應的 HTTP 要求。 瀏覽器會限制可從某個頁面提出之並行 HTTP 要求的數目。 如果同時提出多個長時間執行的同步要求，便有可能會達到此限制。在此情況下，在其中一個要求完成之前，後續的要求將會被延遲。

## <a name="t"></a>T

<a name="telematics"></a> **Telematics** \(車載資通訊\)：透過電信裝置傳送、接收及儲存資訊，同時影響遠端物件上的控制項。 

<a name="temporal-data"></a> **Temporal data** \(時態性資料\)：特別在指時間或日期的資料。 時態性資料指的可能是離散事件 (例如閃電)、移動物件 (例如火車)，或是重複觀察 (例如流量感應器的計數)。

<a name="terrain"></a> **Terrain** \(地勢\)：土地上具有特定特性 (例如多沙地勢或山陵地勢) 的區域。

<a name="thematic-maps"></a> **Thematic maps** \(主題地圖\)：主題地圖是用來反映關於某地理區域之某個主題的簡易地圖。 此地圖類型的常見案例，是依據某些資料計量為行政區 (例如國家/地區) 上色。

<a name="tile-layer"></a> **Tile layer** \(圖格圖層\)：透過將地圖圖格 (矩形區段) 組合成連續圖層來顯示的圖層。 圖格可能是點陣影像圖格或向量圖格。 點陣圖格圖層通常會預先轉譯，並以影像的形式儲存在伺服器上。 這可能會佔用大量儲存體空間。 向量圖格圖層會在用戶端應用程式內即時轉譯，使伺服器端的儲存體需求變得較小。

<a name="time-zone"></a> **Time zone** \(時區\)：地球上的某個區域，基於法律、商業和社會目的而遵循統一的標準時間。 時區通常會遵循國家/地區及其從屬行政區的界線。

<a name="transaction"></a> **Transaction** \(交易\)：Azure 地圖服務會使用交易式授權模型，其中：

- 每要求 15 個地圖或流量圖格，便會建立一個交易。
- 每針對 Azure 地圖服務的其中一個服務 (例如搜尋或路線規劃) 提出一個 API 呼叫，便會建立一個交易。

<a name="transformation"></a> **Transformation** \(轉換\)：在不同地理座標系統之間轉換資料的程序。 例如，您可能會有一些在英國擷取的資料，這些資料會以 OSGB 1936 地理座標系統為基礎。 Azure 地圖服務會使用 WGS84 的 [EPSG:3857](https://epsg.io/3857) \(英文\) 座標參考系統變體。 因此，若要正確顯示資料，Azure 地圖服務便需要將座標從其中一個系統轉換成另一個系統。

<a name="traveling-salesmen-problem-tsp"></a> **Traveling Salesmen Problem (TSP)** \(旅行推銷員問題\)：在這個漢米頓迴路問題中，推銷員必須以最有效率的方式造訪一系列商店，並返回起點。  

<a name="trilateration"></a> **Trilateration** \(三邊測量\)：透過測量三個點之間的距離，根據地球表面上的兩個點找出第三點之位置的程序。

<a name="turn-by-turn-navigation"></a> **Turn-by-turn navigation** \(轉向建議導航\)：能在使用者接近下一個需要執行駕駛動作的位置時，為路線的每個步驟提供路線指示的應用程式。

## <a name="v"></a>V

<a name="vector-data"></a> **Vector data** \(向量資料\)：以點、線條或多邊形表式的座標型資料。

<a name="vector-tile"></a> **Vector tile** \(向量圖格\)：適用於使用與地圖控制項相同的圖格系統來儲存地理空間向量資料的開放式資料規格。 另請參閱[圖格圖層](#tile-layer)。

<a name="vehicle-routing-problem-vrp"></a> **Vehicle Routing Problem (VRP)** \(車輛路線問題\)：在考量一組限制的情況下為車隊計算一組具順序路線的問題類別。 這些限制可能包含如遞送時間窗口、多路線能力，以及旅行時間限制等項目。

<a name="voronoi-diagram"></a> **Voronoi diagram** \(沃羅諾伊圖表\)：將空間分割為區域或資料格，並環繞一系列幾何物件 (通常為點特徵) 的圖表。 這些資料格 (或多邊形) 必須滿足德勞內三角的準則。 與集合中任何其他物件相比，區域內所有位置與其所環繞之物件的距離都較為接近。 沃羅諾伊圖表通常是用來勾勒出受地理特徵影響之周遭區域的輪廓。 

## <a name="w"></a>W

<a name="waypoint"></a> **Waypoint** \(導航點\)：導航點是由經度和緯度所定義的指定地理位置，用於導航目的。 它通常用來代表某人用來於路線上行進的點。

<a name="waypoint-optimization"></a> **Waypoint optimization** \(導航點最佳化\)：對一系列導航點進行重新排序的程序，以最小化通過所有所提供之導航點所需的旅行時間或距離。 根據最佳化的複雜度，通常會被稱為[旅行推銷員問題](#traveling-salesmen-problem-tsp)或[車輛路線問題](#vehicle-routing-problem-vrp)。

<a name="web-map-service-wms"></a> **Web Map Service (WMS)** \(網路地圖服務\)：WMS 為 "Open Geographic Consortium (OGC)" \(開放地理空間協會\) 標準，能定義以影像為基礎的地圖服務。 WMS 服務能針對地圖內的特定區域以隨選方式提供地圖影像。 影像會包含預先轉譯的符號學，並可能轉譯為數種具名樣式的其中一種 (若服務已定義)。

<a name="web-mercator"></a> **Web Mercator** \(Web 麥卡托\)：也稱為球形麥卡托投影，這是麥卡托投影的微幅變體，並主要用於 Web 型地圖程式。 它所使用的公式，與標準麥卡托投影用於較小規模地圖的公式相同。 不過，Web 麥卡托在所有縮放層級都會使用球形公式，而較大規模的麥卡托地圖通常會使用橢圓體形式的投影。 此差異在全球規模並無法察覺，但會導致當地區域的地圖與相同縮放層級的真實橢圓體麥卡托地圖具有些微的差異。

<a name="wgs84"></a> **WGS84**：用來將空間座標關聯至地圖表面上位置的常數集合。 WGS84 數據是大部分線上地圖提供者和 GPS 裝置所使用的標準數據。 Azure 地圖服務會使用 WGS84 的 [EPSG:3857](https://epsg.io/3857) \(英文\) 座標參考系統變體。

## <a name="z"></a>Z

<a name="z-coordinate"></a> **Z-coordinate** \(Z 座標\)：請參閱[高度](#altitude)。 

<a name="zip-code"></a> **Zip code** \(郵遞區號\)：請參閱[郵政編碼](#postal-code)。

<a name="Zoom level"></a> **Zoom level** \(縮放層級\)：指定詳細資料的層級，以及地圖的可見程度。 一路縮小到層級 0 時，通常會顯示完整的世界地圖，但只能顯示有限的詳細資料，例如國家/地區名稱和邊界，以及海洋名稱。 放大到層級 17 時，地圖將會顯示涵蓋數個城市街區的區域，並提供詳細的道路資訊。 如需詳細資訊，請參閱[縮放層級和圖格格線](zoom-levels-and-tile-grid.md)文件。

