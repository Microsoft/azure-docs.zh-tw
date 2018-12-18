---
title: 教學課程：建置單頁 Web 應用程式 - Bing 圖像式搜尋
titleSuffix: Azure Cognitive Services
description: 說明如何在單頁 Web 應用程式中使用 Bing 圖像式搜尋 API。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-visual-search
ms.topic: tutorial
ms.date: 10/04/2017
ms.author: v-brapel
ms.openlocfilehash: e3cd36d799256406b3ae12f35303bd2406468b3c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227175"
---
# <a name="tutorial-visual-search-single-page-web-app"></a>教學課程：圖像式搜尋單頁 Web 應用程式

Bing 圖像式搜尋 API 可提供與 Bing.com/images 上顯示的影像詳細資料類似的體驗。 使用圖像式搜尋可以指定影像並取得影像的相關深入解析，例如外觀上類似的影像、購物來源、包含影像的網頁等。 

本教學課程會從 Bing 影像搜尋教學課程來延伸單頁 Web 應用程式 (請參閱[單頁 Web 應用程式](../Bing-Image-Search/tutorial-bing-image-search-single-page-app.md))。 如需開始進行本教學課程所需的完整原始程式碼，請參閱[單頁 Web 應用程式 (原始程式碼)](../Bing-Image-Search/tutorial-bing-image-search-single-page-app-source.md)。 如需本教學課程最終的原始程式碼，請參閱[圖像式搜尋單頁 Web 應用程式](tutorial-bing-visual-search-single-page-app-source.md)。

涵蓋的工作包括：

> [!div class="checklist"]
> * 使用影像深入解析權杖呼叫 Bing 圖像式搜尋 API
> * 顯示類似的影像

## <a name="call-bing-visual-search"></a>呼叫 Bing 圖像式搜尋
編輯 Bing 影像搜尋教學課程，並在第 409 行將下列程式碼新增至指令碼元素的結尾處。 此程式碼會呼叫 Bing 圖像式搜尋 API，並顯示結果。

``` javascript
function handleVisualSearchResponse(){
    if(this.status !== 200){
        console.log(this.responseText);
        return;
    }
    let json = this.responseText;
    let response = JSON.parse(json);
    for (let i =0; i < response.tags.length; i++) {
        let tag = response.tags[i];
        if (tag.displayName === '') {
            for (let j = 0; j < tag.actions.length; j++) {
                let action = tag.actions[j];
                if (action.actionType === 'VisualSearch') {
                    let html = '';
                    for (let k = 0; k < action.data.value.length; k++) {
                        let item = action.data.value[k];
                        let height = 120;
                        let width = Math.max(Math.round(height * item.thumbnail.width / item.thumbnail.height), 120);
                        html += "<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width + "' height=" + height + " width=" + width + "'>";
                    }
                    showDiv("insights", html);
                    window.scrollTo({top: document.getElementById("insights").getBoundingClientRect().top, behavior: "smooth"});
                }
            }
        }
    }
}

function bingVisualSearch(insightsToken){
    let visualSearchBaseURL = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch',
        boundary = 'boundary_ABC123DEF456',
        startBoundary = '--' + boundary,
        endBoundary = '--' + boundary + '--',
        newLine = "\r\n",
        bodyHeader = 'Content-Disposition: form-data; name="knowledgeRequest"' + newLine + newLine;

    let postBody = {
        imageInfo: {
            imageInsightsToken: insightsToken
        }
    }
    let requestBody = startBoundary + newLine;
    requestBody += bodyHeader;
    requestBody += JSON.stringify(postBody) + newLine + newLine;
    requestBody += endBoundary + newLine;       
    
    let request = new XMLHttpRequest();

    try {
        request.open("POST", visualSearchBaseURL);
    } 
    catch (e) {
        renderErrorMessage("Error performing visual search: " + e.message);
    }
    request.setRequestHeader("Ocp-Apim-Subscription-Key", getSubscriptionKey());
    request.setRequestHeader("Content-Type", "multipart/form-data; boundary=" + boundary);
    request.addEventListener("load", handleVisualSearchResponse);
    request.send(requestBody);
}
```

## <a name="capture-insights-token"></a>擷取深入解析權杖
在第 151 行將下列程式碼新增至 `searchItemsRenderer` 物件中。 程式碼會新增經點按時會呼叫 `bingVisualSearch` 函式的**尋找類似項目**連結。 此函式會接收 imageInsightsToken 以作為引數。

``` javascript
html.push("<a href='javascript:bingVisualSearch(\"" + item.imageInsightsToken + "\");'>find similar</a><br>");
```

## <a name="display-similar-images"></a>顯示類似的影像
在第 601 行將新增下列 HTML 程式碼。 此標記程式碼會新增用來顯示 Bing 圖像式搜尋 API 呼叫結果的元素。

``` html
<div id="insights">
    <h3><a href="#" onclick="return toggleDisplay('_insights')">Similar</a></h3>
    <div id="_insights" style="display: none"></div>
</div>
```

當所有新的 JavaScript 程式碼與 HTML 元素皆設置妥當後，就會顯示搜尋結果以及**尋找類似項目**連結。 按一下此連結，可在 [類似] 區段中填入與您選擇的影像類似的影像。 您可能必須展開 [類似] 區段以顯示影像。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [圖像式搜尋單頁 Web應用程式來源](tutorial-bing-visual-search-single-page-app-source.md)
> [Bing 圖像式搜尋 API 參考](https://aka.ms/bingvisualsearchreferencedoc)