---
title: 使用 Azure 搜尋服務在搜尋方塊中新增自動完成的教學課程 | Microsoft Docs
description: 舉例說明如何使用 Azure 搜尋服務的自動完成和建議 API，來對以資料為中心的應用程式改善終端使用者體驗。
manager: pablocas
author: mrcarter8
services: search
ms.service: search
ms.devlang: NA
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: mcarter
ms.openlocfilehash: 63f4d9f72b9bf81ea772123d65db0659fd3ffa5c
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45578169"
---
# <a name="tutorial-add-auto-complete-to-your-search-box-using-azure-search"></a>教學課程：使用 Azure 搜尋服務在搜尋方塊中新增自動完成

在本教學課程中，您會了解如何使用 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/) 和 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions?view=azure-dotnet) 中的[建議](https://docs.microsoft.com/rest/api/searchservice/suggestions)、[自動完成](https://docs.microsoft.com/rest/api/searchservice/autocomplete)和 [Facet](search-faceted-navigation.md)，來建置功能強大的搜尋方塊。 *建議*可根據使用者到目前為止已輸入的內容，提供實際結果的建議。 自動完成是 Azure 搜尋服務的一項[新預覽功能](search-api-preview.md)，可提供來自索引的詞彙以完成使用者目前正在輸入的內容。 我們會比較多種技術，藉由直接在使用者輸入搜尋內容時為他們提供豐富的搜尋結果，以提升使用者的產能，並輕鬆快速地找到他們所要尋找的內容。

本教學課程會逐步引導您了解 ASP.NET MVC 型應用程式，以使用 C# 呼叫 [Azure 搜尋服務 .NET 用戶端程式庫](https://aka.ms/search-sdk)，並讓您了解 JavaScript，以便直接呼叫 Azure 搜尋服務 REST API。 本教學課程的應用程式會以填入了 [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) 資料範例的索引作為目標。 您可以使用已在 NYC Jobs 示範中完成設定的索引，也可以透過 NYCJobs 解決方案範例中的資料載入器填入您自己的索引。 此範例會使用 [jQuery UI](https://jqueryui.com/autocomplete/) 和 [XDSoft](https://xdsoft.net/jqplugins/autocomplete/) JavaScript 程式庫來建置支援自動完成的搜尋方塊。 使用這些元件以及 Azure 搜尋服務，您就會看到多個範例，了解如何在搜尋方塊中支援自動完成與預先輸入。

您會執行下列工作：

> [!div class="checklist"]
> * 下載並設定解決方案
> * 將搜尋服務資訊新增至應用程式設定
> * 實作搜尋輸入方塊
> * 針對從遠端來源提取的自動完成清單新增支援 
> * 使用 .Net SDK 和 REST API 擷取建議與自動完成
> * 支援用戶端快取以改善效能 

## <a name="prerequisites"></a>先決條件

* Visual Studio 2017。 您可以使用免費的 [Visual Studio 2017 Community 版本](https://www.visualstudio.com/downloads/)。 

* 下載本教學課程的[原始程式碼](https://github.com/azure-samples/search-dotnet-getting-started)範例。

* (選擇性) 有效的 Azure 帳戶和 Azure 搜尋服務。 如果您沒有 Azure 帳戶，您可以註冊[免費試用](https://azure.microsoft.com/free/)。 如需有關服務佈建的說明，請參閱[建立搜尋服務](search-create-service-portal.md)。 您可以選擇是否使用帳戶和服務，原因是您也可以使用手上要用於其他示範的已裝載 NYCJobs 索引來完成本教學課程。

* (選擇性) 下載 [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) 程式碼範例，以將 NYCJobs 資料匯入所擁有 Azure 搜尋服務的索引內。

> [!Note]
> 如果您使用免費的 Azure 搜尋服務，則最多只能使用三個索引。 NYCJobs 資料載入器會建立兩個索引。 請確定服務內有空間可接受新的索引。

### <a name="set-up-azure-search-optional"></a>設定 Azure 搜尋服務 (選擇性)

如果您想要將 NYCJobs 應用程式範例的資料匯入所擁有的索引內，請遵循本節中的步驟。 此為選用步驟。  如果您想要使用所提供的索引範例，請直接跳到下一節來執行範例。

1. 在 NYCJobs 程式碼範例的 DataLoader 資料夾中，於 Visual Studio 開啟 DataLoader.sln 解決方案檔案。

1. 更新 Azure 搜尋服務的連線資訊。  開啟 DataLoader 專案中的 App.config，然後變更 TargetSearchServiceName 和 TargetSearchServiceApiKey appSettings 以反映 Azure 搜尋服務和 Azure 搜尋服務 API 金鑰。  這些資料可以在 Azure 入口網站中找到。

1. 按 F5 啟動應用程式。  這會建立 2 個索引，並匯入 NYCJob 資料範例。

1. 在教學課程的程式碼範例中，於 Visual Studio 開啟 AutocompleteTutorial.sln 解決方案檔案。  開啟 AutocompleteTutorial 專案內的 Web.config，並將 SearchServiceName 和 SearchServiceApiKey 值變更為與上述值相同。

### <a name="running-the-sample"></a>執行範例

現在您已經準備好執行教學課程的應用程式範例。  在 Visual Studio 中開啟 AutocompleteTutorial.sln 解決方案檔案以執行教學課程。  此解決方案包含 ASP.NET MVC 專案。  按 F5 執行專案，並在所選擇的瀏覽器中載入頁面。  在頂端，您會看到可供選取 C# 或 JavaScript 的選項。  C# 選項會從瀏覽器呼叫 HomeController，並使用 Azure 搜尋服務 .Net SDK 來擷取結果。  JavaScript 選項會直接從瀏覽器呼叫 Azure 搜尋服務 REST API。  此選項會將控制器拿到流程外，因此其效能通常會顯著提升。  您可以選擇適合您需求和語言喜好設定的選項。  頁面上有幾個自動完成範例，且各有一些指導方針。  每個範例都有一些建議的文字範例可供您嘗試。  請嘗試在每個搜尋方塊中輸入幾個字母來看看會發生什麼事情。

## <a name="how-this-works-in-code"></a>此功能在程式碼中的運作方式

您已經看過瀏覽器中的範例，接下來請逐步了解第一個範例的細節，以檢閱所涉及的元件和元件的運作方式。

### <a name="search-box"></a>搜尋方塊

不論您選擇哪一種語言，其搜尋方塊都完全相同。  開啟 \Views\Home 資料夾底下的 Index.cshtml 檔案。 搜尋方塊本身很簡單：

```html
<input class="searchBox" type="text" id="example1a" placeholder="search">
```

上面是簡單的輸入文字方塊，並附有用於設定樣式的類別、JavaScript 所要參考的識別碼，和預留位置文字。  其神奇之處在 JavaScript 中。

### <a name="javascript-code-c"></a>JavaScript 程式碼 (C#)

C# 語言範例會在 Index.cshtml 中使用 JavaScript，以運用 jQuery UI 的自動完成程式庫。  此程式庫會對 MVC 控制器發出非同步呼叫以擷取建議，藉此在搜尋方塊中新增自動完成體驗。  請看第一個範例的 JavaScript 程式碼：

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

此程式碼會在頁面載入期間於瀏覽器中執行，以對「example1a」輸入方塊設定自動完成。  `minLength: 3` 可確保在搜尋方塊中至少有三個字元時，才會顯示建議。  來源值很重要：

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

這一行會讓自動完成 API 知道，要到哪裡取得項目清單以便在搜尋方塊下顯示。  由於這是 MVC 專案，它會在 HomeController.cs 中呼叫建議函式。  下一節會有更多這方面的探討。  它也會傳遞幾個參數來控制醒目提示、模糊比對和字詞。  自動完成 JavaScript API 會新增字詞參數。

#### <a name="extending-the-sample-to-support-fuzzy-matching"></a>延伸範例以支援模糊比對

當使用者在搜尋方塊中拼錯字組時，模糊搜尋依然可讓其根據相近的字組取得結果。  請將來源程式行變更為啟用模糊比對，試試這項功能。

將下面這行從：

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

變更為以下程式碼：

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

按 F5 啟動應用程式。

試試看輸入「execative」之類的字詞，並注意其變回「executive」結果的情形 (雖然這些結果並未與您所輸入的字母完全相符)。

### <a name="homecontrollercs-c"></a>HomeController.cs (C#)

我們已經檢閱過此範例的 JavaScript 程式碼，接著請看看會使用 Azure 搜尋服務 .NET SDK 實際擷取建議的 C# 控制器程式碼。

1. 開啟 Controllers 目錄下的 HomeController.cs 檔案。 

1. 您可能會先注意到類別頂端稱為 InitSearch 的方法。  此方法會建立已對 Azure 搜尋服務進行驗證的 HTTP 索引用戶端。  如果您想要深入了解其運作方式，請瀏覽下列教學課程：[如何從 .NET 應用程式使用 Azure 搜尋服務](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)

1. 移至建議函式。

```csharp
public ActionResult Suggest(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    // Call suggest API and return results
    SuggestParameters sp = new SuggestParameters()
    {
        UseFuzzyMatching = fuzzy,
        Top = 5
    };

    if (highlights)
    {
        sp.HighlightPreTag = "<b>";
        sp.HighlightPostTag = "</b>";
    }

    DocumentSuggestResult resp = _indexClient.Documents.Suggest(term, "sg", sp);

    // Convert the suggest query results to a list that can be displayed in the client.
    List<string> suggestions = resp.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = suggestions
    };
}
```

建議函式會採用兩個參數，以決定是要傳回命中結果醒目提示，還是要使用搜尋字詞輸入外加模糊比對。  此方法會建立 SuggestParameters 物件，然後傳遞給建議 API。 結果接著會轉換為 JSON，以供在用戶端中顯示。
(選擇性) 在建議函式開頭新增中斷點，然後逐步執行程式碼。  請注意 SDK 所傳回的回應，以及它會如何轉換為方法所傳回的結果。

頁面上的其他範例會遵循相同模式，新增命中結果醒目提示、自動完成建議的預先輸入和 Facet，以支援自動完成結果的用戶端快取。  檢閱上述每一個項目以了解其運作方式，以及如何在搜尋體驗中運用這些項目。

### <a name="javascript-language-example"></a>JavaScript 語言範例

在 JavaScript 語言範例中，IndexJavaScript.cshtml 頁面中的 JavaScript 程式碼會運用 jQuery UI 自動完成。  這是一個程式庫，會擔當重任來呈現美觀的搜尋方塊，並讓您輕鬆地對 Azure 搜尋服務發出非同步呼叫以擷取建議。  請看第一個範例的 JavaScript 程式碼：

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: function (request, response) {
        $.ajax({
            type: "POST",
            url: suggestUri,
            dataType: "json",
            headers: {
                "api-key": searchServiceApiKey,
                "Content-Type": "application/json"
            },
            data: JSON.stringify({
                top: 5,
                fuzzy: false,
                suggesterName: "sg",
                search: request.term
            }),
                success: function (data) {
                    if (data.value && data.value.length > 0) {
                        response(data.value.map(x => x["@@search.text"]));
                    }
                }
            });
        },
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

如果您比較這個範例與上述會呼叫首頁控制器的範例，您會發現幾個相似之處。  `minLength` 和 `position` 的自動完成設定完全相同。  這邊的重大變更是來源。  系統不會在首頁控制器中呼叫建議方法，而是會在 JavaScript 函式中建立 REST 要求並使用 ajax 來執行。  回應接著會「成功」處理，並作為來源。

## <a name="takeaways"></a>重要心得

本教學課程會示範一些基本步驟，讓您了解如何建置搜尋方塊來支援自動完成和建議。  您已看到如何建置 ASP.NET MVC 應用程式，並使用 Azure 搜尋服務 .Net SDK 或 REST API 來擷取建議。

## <a name="next-steps"></a>後續步驟

請將建議和自動完成整合到搜尋體驗。  考慮直接使用 .Net SDK 或 REST API 可如何協助您，讓您為輸入搜尋內容的使用者提供 Azure 搜尋服務的強大功能，從而使其提高生產力。

> [!div class="nextstepaction"]
> [自動完成 REST API](https://docs.microsoft.com/rest/api/searchservice/autocomplete)
> [建議 REST API](https://docs.microsoft.com/rest/api/searchservice/suggestions)
> [建立索引 REST API 上的 Facet 索引屬性](https://docs.microsoft.com/rest/api/searchservice/create-index)

