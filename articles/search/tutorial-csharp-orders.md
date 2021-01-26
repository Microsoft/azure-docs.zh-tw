---
title: 排序結果的 C# 教學課程
titleSuffix: Azure Cognitive Search
description: 此 C# 教學課程會示範如何排序搜尋結果。 其是以先前的飯店專案為基礎所建置，依主要屬性、次要屬性進行排序，並包含評分設定檔來新增提升準則。
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/26/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 1f8100dd6340383eadec5d10b7f23db59ba0ebdf
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98786380"
---
# <a name="tutorial-order-search-results-using-the-net-sdk"></a>教學課程：使用 .NET SDK 排序搜尋結果

在本教學課程系列中，已傳回結果並且以[預設順序](index-add-scoring-profiles.md#what-is-default-scoring)顯示。 在本教學課程中，您將會新增主要和次要排序準則。 作為依數值排序的替代方案，最後一個範例示範如何依自訂評分設定檔對結果進行排名。 我們也會進一步探討如何顯示「複雜類型」  。

在本教學課程中，您會了解如何：
> [!div class="checklist"]
> * 根據一個屬性排序結果
> * 根據多個屬性排序結果
> * 根據評分設定檔排序結果

## <a name="overview"></a>總覽

本教學課程會擴充在[將分頁新增至搜尋結果](tutorial-csharp-paging.md)教學課程中所建立的無止盡捲動專案。

您可以在下列專案中找到本教學課程中的最終版本程式碼：

* [5-order-results (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/5-order-results)

## <a name="prerequisites"></a>必要條件

* [2b-add-infinite-scroll (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2b-add-infinite-scroll) 解決方案。 此專案可以是您自己在上一個教學課程中建置的版本，或從 GitHub 複製的版本。

本教學課程已更新為使用 [Azure.Search.Documents (第 11 版)](https://www.nuget.org/packages/Azure.Search.Documents/) 套件。 如需舊版的 .NET SDK，請參閱 [Microsoft.Azure.Search (第 10 版) 程式碼範例](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10)。

## <a name="order-results-based-on-one-property"></a>根據一個屬性排序結果

根據一個屬性 (假設是旅館評等) 排序結果時，我們不只想排序結果，我們也想要確認順序正確無誤。 將 [評等] 欄位新增至結果可讓我們確認結果已正確排序。

在此練習中，我們也會在結果的顯示新增更多項目：每家旅館最便宜和最貴的房間價格。

若要啟用排序，不需要修改任何模型。 只有檢視和控制器需要更新。 首先，開啟首頁控制器。

### <a name="add-the-orderby-property-to-the-search-parameters"></a>將 OrderBy 屬性新增至搜尋參數

1. 在 HomeController.cs 中，加入 **OrderBy** 選項並包含評等屬性，並將遞減排序次序。 在 **Index(SearchData model)** 方法中，將下列行新增至搜尋參數。

    ```cs
    options.OrderBy.Add("Rating desc");
    ```

    >[!Note]
    > 預設順序是遞增的，不過您可以加入 `asc` 至屬性以進行清除。 藉由新增來指定遞減順序 `desc` 。

1. 現在，執行應用程式，並輸入任何一般搜尋字詞。 結果的順序可能不正確，因為您 (開發人員) 和使用者都沒有容易的方法可以驗證結果！

1. 讓我們清楚顯示結果是依評等排序的。 首先，將 hotels.css 檔案中的 **box1** 和 **box2** 類別取代為下列類別 (這些是此教學課程中所需的所有新類別)。

    ```html
    textarea.box1A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box1B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box2A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box2B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box3 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }
    ```

    > [!Tip]
    > 瀏覽器通常會快取 CSS 檔案，這可能導致使用舊的 CSS 檔案，而忽略您的編輯。 解決此問題的好方法是在連結新增包含版本參數的查詢字串。 例如：
    >
    >```html
    >   <link rel="stylesheet" href="~/css/hotels.css?v1.1" />
    >```
    >
    > 如果您認為瀏覽器在使用舊版 CSS 檔案，請更新版本號碼。

1. 將 [ **評** 等] 屬性加入至 **Select** 參數，在 **索引 (SearchData 模型)** 方法中，讓結果包含下列三個欄位：

    ```cs
    options.Select.Add("HotelName");
    options.Select.Add("Description");
    options.Select.Add("Rating");
    ```

1. 開啟檢視 (index.cshtml)，並將轉譯迴圈 ( **&lt;!-- Show the hotel data. --&gt;** ) 取代為下列程式碼。

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var ratingText = $"Rating: {result[i].Document.Rating}";

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. 在第一次顯示的頁面中，以及透過無限捲動呼叫的後續頁面中都必須能取得評等。 在這兩個情況的後者中，我們必須更新控制器中的 **Next** 動作和檢視中的 **scrolled** 函式。 從控制器開始，將 **Next** 方法變更為下列程式碼。 此程式碼會建立評等文字，並進行通訊。

    ```cs
    public async Task<ActionResult> Next(SearchData model)
    {
        // Set the next page setting, and call the Index(model) action.
        model.paging = "next";
        await Index(model);

        // Create an empty list.
        var nextHotels = new List<string>();

        // Add a hotel details to the list.
        await foreach (var result in model.resultList.GetResultsAsync())
        {
            var ratingText = $"Rating: {result.Document.Rating}";
            var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";
    
            string fullDescription = result.Document.Description;
    
            // Add strings to the list.
            nextHotels.Add(result.Document.HotelName);
            nextHotels.Add(ratingText);
            nextHotels.Add(fullDescription);
        }

        // Rather than return a view, return the list of data.
        return new JsonResult(nextHotels);
    }
    ```

1. 現在，更新檢視中的 **scrolled** 函式，以顯示評等文字。

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 3) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 2] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. 現在，再次執行應用程式。 搜尋任何一般字詞 (例如，"wifi")，並確認結果會依旅館評等遞減順序排序。

    ![根據評等排序](./media/tutorial-csharp-create-first-app/azure-search-orders-rating.png)

    您會發現幾個旅館有相同評等，因此它們在顯示中的出現順序是找到資料的順序，而這是任意的。

    在探討新增排序的第二層級之前，讓我們新增一些程式碼來顯示房間價格範圍。 我們新增此程式碼來顯示從「複雜類型」  擷取資料，也讓我們可以討論根據價格排序 (可能先顯示最便宜的)。

### <a name="add-the-range-of-room-rates-to-the-view"></a>將房間價格範圍新增到檢視

1. 將包含最便宜和最貴房間價格的屬性新增至 Hotel.cs 模型。

    ```cs
    // Room rate range
    public double cheapest { get; set; }
    public double expensive { get; set; }
    ```

1. 在首頁控制器中 **Index(SearchData model)** 動作的結尾計算房間價格。 在儲存暫存資料之後加入計算。

    ```cs
    // Ensure TempData is stored for the next call.
    TempData["page"] = page;
    TempData["searchfor"] = model.searchText;

    // Calculate the room rate ranges.
    await foreach (var result in model.resultList.GetResultsAsync())
    {
        var cheapest = 0d;
        var expensive = 0d;

        foreach (var room in result.Document.Rooms)
        {
            var rate = room.BaseRate;
            if (rate < cheapest || cheapest == 0)
            {
                cheapest = (double)rate;
            }
            if (rate > expensive)
            {
                expensive = (double)rate;
            }
        }
        model.resultList.Results[n].Document.cheapest = cheapest;
        model.resultList.Results[n].Document.expensive = expensive;
    }
    ```

1. 在控制器的 **Index(SearchData model)** 動作方法中，將 **Rooms** 屬性新增至 **Select** 參數。

    ```cs
    options.Select.Add("Rooms");
    ```

1. 變更檢視中的轉譯迴圈，以在結果的第一頁顯示價格範圍。

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var rateText = $"Rates from ${result[i].Document.cheapest} to ${result[i].Document.expensive}";
        var ratingText = $"Rating: {result[i].Document.Rating}";

        string fullDescription = result[i].Document.Description;

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. 變更首頁控制器中的 **Next** 方法，為後續結果頁面通訊價格範圍。

    ```cs
    public async Task<ActionResult> Next(SearchData model)
    {
        // Set the next page setting, and call the Index(model) action.
        model.paging = "next";
        await Index(model);

        // Create an empty list.
        var nextHotels = new List<string>();

        // Add a hotel details to the list.
        await foreach (var result in model.resultList.GetResultsAsync())
        {
            var ratingText = $"Rating: {result.Document.Rating}";
            var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";

            string fullDescription = result.Document.Description;

            // Add strings to the list.
            nextHotels.Add(result.Document.HotelName);
            nextHotels.Add(ratingText);
            nextHotels.Add(rateText);
            nextHotels.Add(fullDescription);
        }

        // Rather than return a view, return the list of data.
        return new JsonResult(nextHotels);
    }
    ```

1. 更新檢視中的 **scrolled** 函式，以處理房間價格文字。

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 4) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. 執行應用程式，並確認房間價格範圍會顯示。

    ![顯示房間價格範圍](./media/tutorial-csharp-create-first-app/azure-search-orders-rooms.png)

搜尋參數的 **OrderBy** 屬性不會接受 **Rooms.BaseRate** 之類的項目提供最便宜房間價格，即使房間已經根據價格排序。 在此情況下，會議室不會依費率排序。 為了顯示範例資料集中的旅館並根據房間價格排序，您需要在您的首頁控制器中排序結果，並按照想要的順序將結果傳送至檢視。

## <a name="order-results-based-on-multiple-values"></a>根據多個值排序結果

現在考慮的問題是，如何區分相同價格的旅館。 其中一種方法可能是在上一次整修旅館時的次要順序，以便讓較新的整修旅館在結果中出現較高的位置。

1. 若要加入第二個層級的順序，請將 **LastRenovationDate** 新增至搜尋結果，並在索引中加入 **OrderBy** **(SearchData 模型)** 方法。

    ```cs
    options.Select.Add("LastRenovationDate");

    options.OrderBy.Add("LastRenovationDate desc");
    ```

    >[!Tip]
    > 可在 **OrderBy** 清單中輸入任意數目的屬性。 如果旅館有相同的評等和整修日期，可以輸入第三個屬性來區分它們。

1. 同樣地，我們需要在檢視中查看整修日期，以確認排序是正確的。 針對這類整修，可能只有年份已足夠。 將檢視中的轉譯迴圈變更為下列程式碼。

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var rateText = $"Rates from ${result[i].Document.cheapest} to ${result[i].Document.expensive}";
        var lastRenovatedText = $"Last renovated: { result[i].Document.LastRenovationDate.Value.Year}";
        var ratingText = $"Rating: {result[i].Document.Rating}";

        string fullDescription = result[i].Document.Description;

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
        @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. 變更首頁控制器中的 **Next** 方法，以轉送最近整修日期的年份元件。

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel details to the list.
            await foreach (var result in model.resultList.GetResultsAsync())
            {
                var ratingText = $"Rating: {result.Document.Rating}";
                var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";
                var lastRenovatedText = $"Last renovated: {result.Document.LastRenovationDate.Value.Year}";

                string fullDescription = result.Document.Description;

                // Add strings to the list.
                nextHotels.Add(result.Document.HotelName);
                nextHotels.Add(ratingText);
                nextHotels.Add(rateText);
                nextHotels.Add(lastRenovatedText);
                nextHotels.Add(fullDescription);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

1. 變更檢視中的 **scrolled** 函式，以顯示整修文字。

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 5) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2B">' + data[i + 3] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. 執行應用程式。 搜尋一般字詞，例如，"pool" (游泳池) 或 "view" (景觀)，並確認相同評等的旅館現在會依整修日期遞減順序顯示。

    ![依整修日期排序](./media/tutorial-csharp-create-first-app/azure-search-orders-renovation.png)

## <a name="order-results-based-on-a-scoring-profile"></a>根據評分設定檔排序結果

到目前為止，本教學課程中提供的範例會示範如何排序數值 (評等和整修日期) ，提供 _確切_ 順序的順序。 不過，某些搜尋和某些資料本身不支援兩個資料元素之間進行這類簡單比較。 針對全文檢索搜尋查詢，認知搜尋包含 _排名_ 的概念。 您可以指定 _計分設定檔_ 來影響結果的排名，以提供更複雜且更具定性的比較。

[評分設定檔](index-add-scoring-profiles.md) 是在索引架構中定義。 旅館資料上已經設定數個評分設定檔。 讓我們看看評分設定檔的定義方式，然後嘗試撰寫程式碼搜尋它們。

### <a name="how-scoring-profiles-are-defined"></a>評分設定檔是如何定義的

評分設定檔是在設計階段于搜尋索引中定義。 由 Microsoft 主控的唯讀飯店索引有三個評分設定檔。 本節將探索評分設定檔，並向您說明如何在程式碼中使用。

1. 以下是當您未指定任何 **OrderBy** 或 **>scoringprofile** 參數時，所使用之旅館資料集的預設評分設定檔。 如果搜尋文字出現在旅館名稱、描述或標籤清單 (amenities (便利設施)) 中，則此設定檔會提升旅館的「分數」  。 請注意評分的權數會如何偏好某些欄位。 如果搜尋文字出現在其他欄位中 (下面未列出)，則其權數為 1。 很明顯地，分數愈高，結果會顯示在視圖中愈高。

     ```cs
    {
        "name": "boostByField",
        "text": {
            "weights": {
                "Tags": 3,
                "HotelName": 2,
                "Description": 1.5,
                "Description_fr": 1.5,
            }
        }
    }
    ```

1. 如果提供的參數包含一或多個標記清單 (我們呼叫「便利」 ) ，則下列替代計分設定檔會大幅提升分數。 此設定檔的重點是，「必須」  提供包含文字的參數。 如果參數是空的或未提供，系統會擲回錯誤。

    ```cs
    {
        "name":"boostAmenities",
        "functions":[
            {
            "fieldName":"Tags",
            "freshness":null,
            "interpolation":"linear",
            "magnitude":null,
            "distance":null,
            "tag":{
                "tagsParameter":"amenities"
            },
            "type":"tag",
            "boost":5
            }
        ],
        "functionAggregation":0
    },
    ```

1. 在第三個設定檔中，旅館評等可大幅提升分數。 上次整修日期也會提升分數，但只有當該資料是在目前日期的 730 天 (2 年) 內時才有效。

    ```cs
    {
        "name":"renovatedAndHighlyRated",
        "functions":[
            {
            "fieldName":"Rating",
            "freshness":null,
            "interpolation":"linear",
            "magnitude":{
                "boostingRangeStart":0,
                "boostingRangeEnd":5,
                "constantBoostBeyondRange":false
            },
            "distance":null,
            "tag":null,
            "type":"magnitude",
            "boost":20
            },
            {
            "fieldName":"LastRenovationDate",
            "freshness":{
                "boostingDuration":"P730D"
            },
            "interpolation":"quadratic",
            "magnitude":null,
            "distance":null,
            "tag":null,
            "type":"freshness",
            "boost":10
            }
        ],
        "functionAggregation":0
    }
    ```

    現在，讓我們看看這些設定檔是否如預期運作。

### <a name="add-code-to-the-view-to-compare-profiles"></a>將程式碼新增至檢視以比較設定檔

1. 開啟 index.cshtml 檔案，並將 &lt;body&gt; 區段取代為下列程式碼。

    ```html
    <body>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        <table>
            <tr>
                <td></td>
                <td>
                    <h1 class="sampleTitle">
                        <img src="~/images/azure-logo.png" width="80" />
                        Hotels Search - Order Results
                    </h1>
                </td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <!-- Display the search text box, with the search icon to the right of it. -->
                    <div class="searchBoxForm">
                        @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
                    </div>

                    <div class="searchBoxForm">
                        <b>&nbsp;Order:&nbsp;</b>
                        @Html.RadioButtonFor(m => m.scoring, "Default") Default&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "RatingRenovation") By numerical Rating&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "boostAmenities") By Amenities&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "renovatedAndHighlyRated") By Renovated date/Rating profile&nbsp;&nbsp;
                    </div>
                </td>
            </tr>

            <tr>
                <td valign="top">
                    <div id="facetplace" class="facetchecks">

                        @if (Model != null && Model.facetText != null)
                        {
                            <h5 class="facetheader">Amenities:</h5>
                            <ul class="facetlist">
                                @for (var c = 0; c < Model.facetText.Length; c++)
                                {
                                    <li> @Html.CheckBoxFor(m => m.facetOn[c], new { @id = "check" + c.ToString() }) @Model.facetText[c] </li>
                                }

                            </ul>
                        }
                    </div>
                </td>
                <td>
                    @if (Model != null && Model.resultList != null)
                    {
                        // Show the total result count.
                        <p class="sampleText">
                            @Html.DisplayFor(m => m.resultList.Count) Results <br />
                        </p>

                        <div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()">

                            <!-- Show the hotel data. -->
                            @for (var i = 0; i < Model.resultList.Results.Count; i++)
                            {
                                var rateText = $"Rates from ${Model.resultList.Results[i].Document.cheapest} to ${Model.resultList.Results[i].Document.expensive}";
                                var lastRenovatedText = $"Last renovated: { Model.resultList.Results[i].Document.LastRenovationDate.Value.Year}";
                                var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                                string amenities = string.Join(", ", Model.resultList.Results[i].Document.Tags);
                                string fullDescription = Model.resultList.Results[i].Document.Description;
                                fullDescription += $"\nAmenities: {amenities}";

                                // Display the hotel details.
                                @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                                @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                                @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
                                @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
                                @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
                            }
                        </div>

                        <script>
                            function scrolled() {
                                if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                                    $.getJSON("/Home/Next", function (data) {
                                        var div = document.getElementById('myDiv');

                                        // Append the returned data to the current list of hotels.
                                        for (var i = 0; i < data.length; i += 5) {
                                            div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                            div.innerHTML += '<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                                            div.innerHTML += '<textarea class="box2B">' + data[i + 3] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                                        }
                                    });
                                }
                            }
                        </script>
                    }
                </td>
            </tr>
        </table>
    }
    </body>
    ```

1. 開啟 SearchData.cs 檔案，並將 **SearchData** 類別取代為下列程式碼。

    ```cs
    public class SearchData
    {
        public SearchData()
        {
        }

        // Constructor to initialize the list of facets sent from the controller.
        public SearchData(List<string> facets)
        {
            facetText = new string[facets.Count];

            for (int i = 0; i < facets.Count; i++)
            {
                facetText[i] = facets[i];
            }
        }

        // Array to hold the text for each amenity.
        public string[] facetText { get; set; }

        // Array to hold the setting for each amenitity.
        public bool[] facetOn { get; set; }

        // The text to search for.
        public string searchText { get; set; }

        // Record if the next page is requested.
        public string paging { get; set; }

        // The list of results.
        public DocumentSearchResult<Hotel> resultList;

        public string scoring { get; set; }       
    }
    ```

1. 開啟 hotels.css 檔案，並新增下列 HTML 類別。

    ```html
    .facetlist {
        list-style: none;
    }
    
    .facetchecks {
        width: 250px;
        display: normal;
        color: #666;
        margin: 10px;
        padding: 5px;
    }

    .facetheader {
        font-size: 10pt;
        font-weight: bold;
        color: darkgreen;
    }
    ```

### <a name="add-code-to-the-controller-to-specify-a-scoring-profile"></a>將程式碼新增至控制器以指定評分設定檔

1. 開啟首頁控制器檔案。 新增下列 **using** 陳述式 (協助建立清單)。

    ```cs
    using System.Linq;
    ```

1. 針對此範例，我們需要 **Index** 的初始呼叫多做一些事，而不只是傳回初始檢視。 該方法現在會搜尋最多 20 個便利設施以在檢視中顯示。

    ```cs
        public async Task<ActionResult> Index()
        {
            InitSearch();

            // Set up the facets call in the search parameters.
            SearchOptions options = new SearchOptions();
            // Search for up to 20 amenities.
            options.Facets.Add("Tags,count:20");

            SearchResults<Hotel> searchResult = await _searchClient.SearchAsync<Hotel>("*", options);

            // Convert the results to a list that can be displayed in the client.
            List<string> facets = searchResult.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

            // Initiate a model with a list of facets for the first view.
            SearchData model = new SearchData(facets);

            // Save the facet text for the next view.
            SaveFacets(model, false);

            // Render the view including the facets.
            return View(model);
        }
    ```

1. 我們需要兩個私用方法來儲存暫存位置的 Facet，以及從暫存位置復原它們並填入模型。

    ```cs
        // Save the facet text to temporary storage, optionally saving the state of the check boxes.
        private void SaveFacets(SearchData model, bool saveChecks = false)
        {
            for (int i = 0; i < model.facetText.Length; i++)
            {
                TempData["facet" + i.ToString()] = model.facetText[i];
                if (saveChecks)
                {
                    TempData["faceton" + i.ToString()] = model.facetOn[i];
                }
            }
            TempData["facetcount"] = model.facetText.Length;
        }

        // Recover the facet text to a model, optionally recoving the state of the check boxes.
        private void RecoverFacets(SearchData model, bool recoverChecks = false)
        {
            // Create arrays of the appropriate length.
            model.facetText = new string[(int)TempData["facetcount"]];
            if (recoverChecks)
            {
                model.facetOn = new bool[(int)TempData["facetcount"]];
            }

            for (int i = 0; i < (int)TempData["facetcount"]; i++)
            {
                model.facetText[i] = TempData["facet" + i.ToString()].ToString();
                if (recoverChecks)
                {
                    model.facetOn[i] = (bool)TempData["faceton" + i.ToString()];
                }
            }
        }
    ```

1. 我們必須視需要設定  **OrderBy** 和 **ScoringProfile** 參數。 以下列程式碼取代現有 **Index(SearchData model)** 方法。

    ```cs
    public async Task<ActionResult> Index(SearchData model)
    {
        try
        {
            InitSearch();

            int page;

            if (model.paging != null && model.paging == "next")
            {
                // Recover the facet text, and the facet check box settings.
                RecoverFacets(model, true);

                // Increment the page.
                page = (int)TempData["page"] + 1;

                // Recover the search text.
                model.searchText = TempData["searchfor"].ToString();
            }
            else
            {
                // First search with text. 
                // Recover the facet text, but ignore the check box settings, and use the current model settings.
                RecoverFacets(model, false);

                // First call. Check for valid text input, and valid scoring profile.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }
                if (model.scoring == null)
                {
                    model.scoring = "Default";
                }
                page = 0;
            }

            // Setup the search parameters.
            var options = new SearchOptions
            {
                SearchMode = SearchMode.All,

                // Skip past results that have already been returned.
                Skip = page * GlobalVariables.ResultsPerPage,

                // Take only the next page worth of results.
                Size = GlobalVariables.ResultsPerPage,

                // Include the total number of results.
                IncludeTotalCount = true,
            };
            // Select the data properties to be returned.
            options.Select.Add("HotelName");
            options.Select.Add("Description");
            options.Select.Add("Tags");
            options.Select.Add("Rooms");
            options.Select.Add("Rating");
            options.Select.Add("LastRenovationDate");

            List<string> parameters = new List<string>();
            // Set the ordering based on the user's radio button selection.
            switch (model.scoring)
            {
                case "RatingRenovation":
                    // Set the ordering/scoring parameters.
                    options.OrderBy.Add("Rating desc");
                    options.OrderBy.Add("LastRenovationDate desc");
                    break;

                case "boostAmenities":
                    {
                        options.ScoringProfile = model.scoring;

                        // Create a string list of amenities that have been clicked.
                        for (int a = 0; a < model.facetOn.Length; a++)
                        {
                            if (model.facetOn[a])
                            {
                                parameters.Add(model.facetText[a]);
                            }
                        }

                        if (parameters.Count > 0)
                        {
                            options.ScoringParameters.Add($"amenities-{ string.Join(',', parameters)}");
                        }
                        else
                        {
                            // No amenities selected, so set profile back to default.
                            options.ScoringProfile = "";
                        }
                    }
                    break;

                case "renovatedAndHighlyRated":
                    options.ScoringProfile = model.scoring;
                    break;

                default:
                    break;
            }

            // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
            model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options);

            // Ensure TempData is stored for the next call.
            TempData["page"] = page;
            TempData["searchfor"] = model.searchText;
            TempData["scoring"] = model.scoring;
            SaveFacets(model, true);

            // Calculate the room rate ranges.
            await foreach (var result in model.resultList.GetResultsAsync())
            {
                var cheapest = 0d;
                var expensive = 0d;

                foreach (var room in result.Document.Rooms)
                {
                    var rate = room.BaseRate;
                    if (rate < cheapest || cheapest == 0)
                    {
                        cheapest = (double)rate;
                    }
                    if (rate > expensive)
                    {
                        expensive = (double)rate;
                    }
                }

                result.Document.cheapest = cheapest;
                result.Document.expensive = expensive;
            }
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }

        return View("Index", model);
    }
    ```

    閱讀每個 **switch** 選項的註解。

1. 如果您已完成上一節根據多個屬性排序的其他程式碼，則我們不需要對 **Next** 動作進行任何變更。

### <a name="run-and-test-the-app"></a>執行和測試應用程式

1. 執行應用程式。 您應該會在檢視中看到便利設施的完整集合。

1. 針對排序，選取 [By numerical Rating] \(依數值評分\) 會提供您已在此教學課程中實作的數值排序，當評等相同時則依整修日期排序。

   ![根據評分排序 "beach" (海灘)](./media/tutorial-csharp-create-first-app/azure-search-orders-beach.png)

1. 現在，請嘗試 [By Amenities] \(依便利設施\) 設定檔。 選取不同便利設施，然後驗證有這些便利設施的旅館在結果清單中獲得提升。

   ![根據設定檔排序 "beach"](./media/tutorial-csharp-create-first-app/azure-search-orders-beach-profile.png)

1. 請嘗試 [By Renovated date/Rating profile] \(依整修日期/評分設定檔\)，以查看是否取得如預期的結果。 只有最近已整修的旅館會獲得 _freshness_ 提升。

### <a name="resources"></a>資源

如需詳細資訊，請參閱下列[將評分設定檔新增至 Azure 認知搜尋索引](./index-add-scoring-profiles.md)。

## <a name="takeaways"></a>重要心得

請考慮此專案的下列重點：

* 使用者會希望搜尋結果已排序，最相關的先出現。
* 資料需要結構化才能更容易地排序。 我們無法輕鬆地依「最便宜」優先來排序，因為資料未結構化為不需要其他程式碼就能排序。
* 您可以依許多層級來排序，以區分在排序較高層級有相同值的結果。
* 某些結果適合依遞增順序排序 (例如，與定點的距離)，而某些適合遞減順序 (例如，客戶的評等)。
* 當資料集不適用 (或不夠聰明) 數值比較時，可以定義評分設定檔。 為每個結果評分有助於智慧地排序和顯示結果。

## <a name="next-steps"></a>後續步驟

您已完成這一系列的 C# 教學課程，您應該獲得了寶貴的 Azure 認知搜尋 API 知識。

如需進一步的參考和教學課程，您可以瀏覽 [Microsoft Learn](/learn/browse/?products=azure)，或 [Azure 認知搜尋文件](./index.yml)中的其他教學課程。