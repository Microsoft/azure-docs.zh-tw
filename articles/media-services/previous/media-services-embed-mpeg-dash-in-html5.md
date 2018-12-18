---
title: 透過 DASH.js 將 MPEG-DASH 彈性資料流視訊嵌入到 HTML5 應用程式 | Microsoft Docs
description: 本主題示範如何使用 DASH.js 在 HTML5 應用程式中嵌入 MPEG-DASH 彈性資料流視訊。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 5aa0e7b6-f5c3-4cc1-aa33-ed16ea4780c2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: juliako
ms.openlocfilehash: 2b0e6bf643f55e1809b29def7766c58b59f4bb50
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783377"
---
# <a name="embedding-an-mpeg-dash-adaptive-streaming-video-in-an-html5-application-with-dashjs"></a>透過 DASH.js 將 MPEG-DASH 彈性資料流視訊嵌入到 HTML5 應用程式
## <a name="overview"></a>概觀
MPEG-DASH 符合 ISO 的視訊內容彈性資料流標準，能為想要傳遞高品質彈性視訊資料流輸出的開發人員帶來相當大的幫助。 透過 MPEG-DASH，視訊資料流在網路擁塞時會自動調整為較低畫質的內容。 這會減少檢視者在播放程式下載接下來數秒的播放內容 (亦即緩衝) 時，看到視訊「暫停」的可能性。 當網路不再擁塞，視訊播放程式會改為高品質的資料流。 這種調整所需頻寬的能力也會讓視訊的開始時間變快。 這表示會在快速下載但低品質區段中播放頭幾秒的內容，一旦已緩衝足夠的內容，就會升級為高品質內容。

Dash.js 是以 JavaScript 撰寫的開放原始碼 MPEG-DASH 視訊播放程式。 其目標是要在需要播放視訊的應用程式中，提供一個健全、跨平台、並可自由重複使用的播放程式。 它可在任何支援 W3C Media Source Extensions (MSE) 的瀏覽器 (亦即今日的 Chrome、Microsoft Edge 與 IE11 ) 中播放 MPEG-DASH (其他瀏覽器已表示其支援 MSE 的用途)。 如需 DASH.js 的詳細資訊，請參閱 GitHub dash.js 存放庫。

## <a name="creating-a-browser-based-streaming-video-player"></a>建立以瀏覽器為基礎的資料流視訊播放程式
若要建立簡單的網頁來顯示附有播放、暫停、倒轉等應有控制項的視訊播放器，您必須：

1. 建立 HTML 網頁
2. 新增視訊標記
3. 新增 dash.js 播放程式
4. 初始化播放程式
5. 新增部分 CSS 樣式
6. 在實作 MSE 的瀏覽器中檢視結果

只需數行的 JavaScript 程式碼，就能完成播放程式的初始化。 使用 dash.js 其實很簡單，只要在以瀏覽器為基礎的應用程式中嵌入 MPEG-DASH 視訊即可。

## <a name="creating-the-html-page"></a>建立 HTML 網頁
第一個步驟是建立一個包含**視訊**元素的標準 HTML 頁面，然後將此檔案儲存為 basicPlayer.html，如下列範例說明：

```html
    <!DOCTYPE html>
    <html>
      <head><title>Adaptive Streaming in HTML5</title></head>
      <body>
        <h1>Adaptive Streaming with HTML5</h1>
        <video id="videoplayer" controls></video>
      </body>
    </html>
```

## <a name="adding-the-dashjs-player"></a>新增 DASH.js 播放程式
若要將 dash.js 參考實作新增至該應用程式，您必須從 1.0 版的 dash.js 專案捕捉 dash.all.js 檔案。 這應該儲存在您應用程式的 JavaScript 資料夾中。 此檔案可讓您很方便地將所有必要的 dash.js 程式碼提取到一個檔案中。 如果您瀏覽過 dash.js 存放庫，就會發現各個檔案、測試程式碼等等，但如果您只是要使用 dash.js，那麼 dash.all.js 就是您所需的檔案。

若要在應用程式中新增 dash.js 播放程式，請將指令碼標記新增到 basicPlayer.html 的標頭區段：

```html
    <!-- DASH-AVC/265 reference implementation -->
    < script src="js/dash.all.js"></script>
```

接下來，建立一個會在頁面載入時初始化播放程式的函式。 在要載入 dash.all.js 的那一行之後新增下列指令碼：

```html
    <script>
    // setup the video element and attach it to the Dash player
    function setupVideo() {
      var url = "http://wams.edgesuite.net/media/MPTExpressionData02/BigBuckBunny_1080p24_IYUV_2ch.ism/manifest(format=mpd-time-csf)";
      var context = new Dash.di.DashContext();
      var player = new MediaPlayer(context);
                      player.startup();
                      player.attachView(document.querySelector("#videoplayer"));
                      player.attachSource(url);
    }
    </script>
```

此函式會先建立 DashContext。 這用來為特定的執行階段環境設定應用程式。 從技術觀點來看，其會定義當建構應用程式時，相依性插入架構應使用的類別。 在大部分情況下，您會使用 Dash.di.DashContext。

接著，執行個體化 dash.js 架構的主要類別，亦即 MediaPlayer。 這個類別包含所需的核心方法，例如播放和暫停，並會管理與視訊元素間的關聯性，及管理可描述待播放視訊的 Media Presentation Description (MPD) 檔案解譯。

這會呼叫 MediaPlayer 類別的 startup () 函式，以確保播放程式準備好播放視訊。 此外，這個函式可確保已載入所有必要的類別 (如內容所定義)。 一旦播放程式準備就緒，您可以使用 attachview () 函式將視訊元素附加到播放程式。 startup 函式可讓 MediaPlayer 將視訊資料流插入元素中，並視需要控制播放。

將 MPD 檔案的 URL 傳遞到 MediaPlayer，讓它知道預期要播放的視訊。 一旦頁面整個載入後，就必須執行剛剛建立的 setupVideo() 函式。 做法是使用內文元素的載入事件。 將 <body> 元素變更為：

```html
    <body onload="setupVideo()">
```

最後，使用 CSS 設定視訊元素的大小。 這在可調資料流環境中特別重要，因為當隨著多變的網路狀況調適播放時，正在播放的視訊大小可能會改變。 在此簡單的示範中，只會強制視訊元素變成可用瀏覽器視窗的 80%，做法是在頁面的標頭區段中新增下列 CSS：

```html
    <style>
    video {
      width: 80%;
      height: 80%;
    }
    </style>
```

## <a name="playing-a-video"></a>播放視訊
若要播放視訊，請將瀏覽器指向 basicPlayback.html 檔案，然後按一下所顯示視訊播放程式上的 [播放]。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[開發視訊播放程式應用程式](media-services-develop-video-players.md)

[GitHub dash.js 存放庫](https://github.com/Dash-Industry-Forum/dash.js) 

