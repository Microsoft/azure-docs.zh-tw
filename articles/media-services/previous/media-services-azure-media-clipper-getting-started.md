---
title: 開始使用 Azure Media Clipper | Microsoft Docs
description: 開始使用 Azure Media Clipper，這是用來從 AMS 資產建置影片剪輯的工具
services: media-services
keywords: clip;subclip;encoding;media;剪輯;子剪輯;編碼;媒體
author: dbgeorge
manager: jasonsue
ms.author: dwgeo
ms.date: 11/10/2017
ms.topic: article
ms.service: media-services
ms.openlocfilehash: ac64d97aeeef6147aa62658c9ee440bf058f4db1
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783287"
---
# <a name="create-clips-with-azure-media-clipper"></a>使用 Azure Media Clipper 建立剪輯
本節說明開始使用 Azure Media Clipper 的基本步驟。 以下章節提供有關如何設定 Azure Media Clipper 的細節。

- 首先，將 Azure Media Player 和 Azure Media Clipper 的下列連結新增至您文件的標頭。 建議在 URL 中明確指定 Clipper 和 Azure Media Player 的版本。 不要在生產中使用這些資源的最新版本，因為它們依據需求還會變動。

```javascript
<!--Azure Media Player 2.1.4 or later is a prerequisite-->
<link href="//amp.azure.net/libs/amp/2.1.4/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
<script src="//amp.azure.net/libs/amp/2.1.4/azuremediaplayer.min.js"></script>
<!--Azure Media Clipper script and stylesheet-->
<link href="//amp.azure.net/libs/amc/0.1.0/azuremediaclipper.css" rel="stylesheet">
<script src="//amp.azure.net/libs/amc/0.1.0/azuremediaclipper.min.js"></script>
```

- 接下來，將下列類別新增至您想要具現化 Clipper 的 div 元素。

```javascript
<div id="root" class="azure-subclipper" />
```

(選擇性) 若要啟用深色佈景主題，新增深色面板類別：

```javascript
<div id="root" class="azure-subclipper dark-skin" />
```

- 接下來，使用下列 API 呼叫具現化 Clipper：

```javascript
var subclipper = new subclipper({
    selector: '#root',
    logLevel: 'info',
    restVersion: '2.0',
    minimumMarkerGap: 6,
    submitSubclipCallback: onSubmitSubclip,
    singleBitrateMp4Profile: {
            Codecs: [{
                    // Video Codec with single H264Layers
                }, {
                    // Audio Codec
                }]
    },
    multiBitrateMp4Profile: {
            Codecs: [{
                    // Video Codec with multiple H264Layers
                }, {
                    // Audio Codec
                }]
    },
    keymap: {
      // See below for more info
    },
   // Enable the Video Assets panel in the widget to automatically load assets (input contract)
    assetsPanelLoaderCallback: onLoadAssets,
    height: 600,
    subclippingMode: 'all',
    filterAssetsTypes: true,
    speedLevels: [
        { name: '4x', value: 4.0 },
        { name: '3x', value: 3.0 },
        { name: '2x', value: 2.0 },
        { name: 'normal', value: 1.0 },
        { name: '0.75x', value: 0.75 },
        { name: '0.5x', value: 0.5 },
    ],
    resetOnJobDone: false,
    autoplayVideo: true,
    language: 'en'    
});
```

初始化方法呼叫的參數如下：
- `selector` {必要，字串}：應該轉譯小工具之處相符 HTML 元素的 CSS 選取器。
- `restVersion` {必要，字串}：目標的 Azure Media Services REST API 版本。 REST 版本會定義小工具所產生的輸出格式。 目前僅支援 2.0。
- `submitSubclipCallback` {必要，承諾} 按一下小工具的 [提交] 按鈕時叫用的回呼函式。 回呼函式應該預期小工具所產生的輸出 (轉譯作業設定或篩選定義)。 如需詳細資訊，請參閱「提交子剪輯回呼」。
- `logLevel` {選擇性，{'info', 'warn', 'error'}}：要在瀏覽器主控台中顯示的記錄等級。 預設值：錯誤
- `minimumMarkerGap` {選擇性，int}：子剪輯的大小下限 (以秒為單位)。 注意：值應該大於或等於 6，6 也是預設值。
- `singleBitrateMp4Profile` {選擇性，JSON 物件} 要用於小工具所產生之轉譯作業設定的單一位元速率 mp4 設定檔。 如果未提供，它會使用[預設單一位元速率 MP4 設定檔](https://docs.microsoft.com/azure/media-services/media-services-mes-preset-h264-single-bitrate-1080p)。
- `multiBitrateMp4Profile` {選擇性，JSON 物件} 要用於小工具所產生之轉譯作業設定的多位元速率 mp4 設定檔。 如果未提供，它會使用[預設多位元速率 MP4 設定檔](https://docs.microsoft.com/azure/media-services/media-services-mes-preset-h264-multiple-bitrate-1080p)。
- `keymap` {選擇性，json 物件} 允許自訂小工具的鍵盤快速鍵。 如需詳細資訊，請參閱[可自訂的鍵盤快速鍵](media-services-azure-media-clipper-keyboard-shortcuts.md)。
- `assetsPanelLoaderCallback` {選擇性，承諾} 每次使用者向下捲動到窗格底部時，叫用回呼函式以將資產的新分頁載入 (非同步) 至資產窗格。 如需詳細資訊，請參閱「資產窗格載入器回呼」。
- `height` {選擇性，數字} 小工具高度總計 (不含資產窗格的高度下限為 600 px，含有資產窗格的高度下限為 850 px)。
- `subclippingMode` (選擇性，{'all', 'render', 'filter'})：允許子剪輯模式。 預設值為全部。
- `filterAssetsTypes` (選擇性，布林值)：filterAssetsTypes 可讓您從資產窗格顯示/隱藏篩選下拉式清單。 預設值為 true。
- `speedLevels` (選擇性，陣列)：speedLevels 允許為影片播放器設定不同的速度等級，請參閱 [Azure Media Player 文件](http://amp.azure.net/libs/amp/latest/docs/#amp.player.playbackspeedoptions)以取得詳細資訊。
- `resetOnJobDone` (選擇性，布林值)：resetOnJobDone 允許當成功提交作業時 Clipper 將子剪輯器重設為初始狀態。
- `autoplayVideo` (選擇性，布林值)：autoplayVideo 允許 Clipper 在載入時自動播放影片。 預設值為 true。
- `language` {選擇性，字串}：語言會設定小工具的語言。 如果未指定，小工具會嘗試根據瀏覽器語言將訊息當地語系化。 如果在瀏覽器中偵測不到任何語言，小工具預設為英文。 如需詳細資訊，請參閱[設定當地語系化](media-services-azure-media-clipper-localization.md)一節。
- `languages` {選擇性，JSON}：語言參數會將語言的預設字典取代為使用者定義的自訂字典。 如需詳細資訊，請參閱[設定當地語系化](media-services-azure-media-clipper-localization.md)一節。
- `extraLanguages` (選擇性，JSON)：extraLanaguages 參數會將新語言新增至預設字典。 如需詳細資訊，請參閱[設定當地語系化](media-services-azure-media-clipper-localization.md)一節。

## <a name="typescript-definition"></a>TypeScript 定義
Clipper 的 [TypeScript](https://www.typescriptlang.org/) 定義檔案可以在[這裡](http://amp.azure.net/libs/amc/latest/azuremediaclipper.d.ts)找到。

## <a name="azure-media-clipper-api"></a>Azure Media Clipper API
本節記載 Clipper 提供的 API 介面。

- `ready(handler)`：提供一種方式，在完全載入 Clipper 並準備好使用之後，儘速執行 JavaScript。
- `load(assets)`：將資產清單載入小工具時間軸 (不應與 assetsPanelLoaderCallback 搭配使用)。 請參閱這篇[文章](media-services-azure-media-clipper-load-assets.md)以取得如何將資產載入 Clipper 的詳細資料。
- `setLogLevel(level)`：設定要在瀏覽器主控台中顯示的記錄等級。 可能的值為：`info`、`warn`、`error`。
- `setHeight(height)`：設定小工具高度總計，以像素為單位 (不含資產窗格的高度下限為 600 px，含有資產窗格的高度下限為 850 px)。
- `version`：取得小工具版本。

## <a name="next-steps"></a>後續步驟
請參閱設定 Azure Media Clipper 的後續步驟：
- [將資產載入 Azure Media Clipper](media-services-azure-media-clipper-load-assets.md)
- [設定自訂鍵盤快速鍵](media-services-azure-media-clipper-keyboard-shortcuts.md)
- [從 Clipper 提交剪輯作業](media-services-azure-media-clipper-submit-job.md)
- [設定當地語系化](media-services-azure-media-clipper-localization.md)