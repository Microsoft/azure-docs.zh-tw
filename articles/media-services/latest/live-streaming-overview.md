---
title: 使用 Azure 媒體服務之即時串流的概觀 | Microsoft Docs
description: 本主題提供使用 Azure 媒體服務 v3 進行即時串流的概觀。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 06/06/2018
ms.author: juliako
ms.openlocfilehash: e9ecf1ba3022ca057fa09bad2413aa19d902ae23
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972174"
---
# <a name="live-streaming-with-azure-media-services-v3"></a>使用 Azure 媒體服務 v3 進行即時串流

使用 Azure 媒體服務傳遞即時串流時，通常涉及下列元件：

* 相機，用來廣播事件。
* 即時視訊編碼器，它會將相機 (或膝上型電腦等其他裝置) 中的訊號轉換成資料流，再傳送至媒體服務即時串流服務。 訊號也可能包括廣告 SCTE-35 和 AD 提示。 
* 媒體服務即時串流服務，可讓您內嵌、預覽、封裝、記錄、加密內容，並向您的客戶廣播內容，或向 CDN 廣播以進一步散發。

本文提供詳細的概觀，並包含使用媒體服務進行即時串流所需之主要元件的圖表。

## <a name="overview-of-main-components"></a>主要元件概觀

在媒體服務中，[LiveEvent](https://docs.microsoft.com/rest/api/media/liveevents) 會負責處理即時串流內容。 LiveEvent 會提供輸入端點 (內嵌 URL)，接著您再提供給內部部署即時編碼器。 LiveEvent 會從即時編碼器接收 RTMP 或 Smooth Streaming 格式的即時輸入資料流，並透過一或多個 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 進行串流處理。 [LiveOutput](https://docs.microsoft.com/rest/api/media/liveoutputs) 可讓您控制即時資料流的發佈、記錄和 DVR 時間設定。 LiveEvent 也會提供預覽端點 (預覽 URL)，您可以先用來預覽和驗證資料流，再進一步處理和傳遞。 

媒體服務提供**動態封裝**，可讓您預覽及廣播 MPEG DASH、HLS、Smooth Streaming 串流格式的內容，而不必手動重新封裝成這些串流格式。 您可以使用任何 HLS、DASH 或 Smooth 相容的播放程式進行播放。 您也可以使用 [Azure 媒體播放器](http://amp.azure.net/libs/amp/latest/docs/index.html)來測試您的資料流。

媒體服務可讓您傳遞利用進階加密標準 (AES-128) 或下列三個主要數位版權管理 (DRM) 系統中任一個所動態加密 (**動態加密**) 的內容：Microsoft PlayReady、Google Widevine 和 Apple FairPlay。 媒體服務也提供服務，可傳遞 AES 金鑰和 DRM (PlayReady、Widevine 和 FairPlay) 授權給授權用戶端。

如有需要，您也可以套用 [動態篩選]，這可用來控制傳送至播放程式的曲目編號、格式和位元速率。 媒體服務也支援插入廣告。


## <a name="liveevent-types"></a>LiveEvent 類型

[LiveEvent](https://docs.microsoft.com/rest/api/media/liveevents) 可以是下列兩種類型之一：即時編碼和即時通行。 

### <a name="live-encoding-with-media-services"></a>使用媒體服務進行即時編碼

![即時編碼](./media/live-streaming/live-encoding.png)

內部部署即時編碼器會將單一位元速率資料流傳送至 LiveEvent，可以使用下列通訊協定之一，以媒體服務執行即時編碼：RTMP 或 Smooth Streaming (分散式 MP4)。 LiveEvent 接著會執行即時編碼，將內送單一位元速率資料流編碼成多位元速率 (自適性) 視訊資料流。 接到要求時，媒體服務會傳遞串流給客戶。

建立這種 LiveEvent 類型時，請指定 [基本] (LiveEventEncodingType.Basic)。

### <a name="pass-through"></a>傳遞

![即時通行](./media/live-streaming/pass-through.png)

即時通行已透過內部部署即時編碼器，針對長時間執行的即時資料流或 24x7 線性即時編碼最佳化。 內部部署編碼器會傳送多位元速率 **RTMP** 或 **Smooth Streaming** (分散式 MP4) 到針對**即時通行**傳遞所設定的 LiveEvent。 **即時通行**傳遞就是內嵌的資料流會通過 **LiveEvent**，無需進一步的處理。 

即時通行 LiveEvent 搭配 Smooth Streaming 內嵌通訊協定使用時，最多可支援 4K 解析度和 HEVC 即時通行。 

建立這種 LiveEvent 類型時，請指定 [無] (LiveEventEncodingType.None)。

> [!NOTE]
> 如果您在很長一段時間內進行多個事件，而且已投資內部部署編碼器時，使用傳遞方法是進行即時串流的最經濟實惠方式。 請參閱 [價格](https://azure.microsoft.com/pricing/details/media-services/) 詳細資料。
> 

## <a name="liveevent-types-comparison"></a>LiveEvent 類型比較 

下表比較兩種 LiveEvent 類型的功能。

| 功能 | 即時通行 LiveEvent | 基本 LiveEvent |
| --- | --- | --- |
| 單一位元速率輸入會在雲端編碼為多重位元速率 |否 |yes |
| 最大解析度、分層數目 |4Kp30  |720p、6 層、30 fps |
| 輸入通訊協定 |RTMP、Smooth Streaming |RTMP、Smooth Streaming |
| 價格 |請參閱 [定價頁面](https://azure.microsoft.com/pricing/details/media-services/) 並按一下 [即時影片] 索引標籤 |請參閱 [定價頁面](https://azure.microsoft.com/pricing/details/media-services/) |
| 最長執行時間 |全天候 |全天候 |
| 插入靜態圖像支援 |否 |yes |
| 支援透過 API 發出廣告訊號|否 |yes |
| 支援透過頻內 SCTE35 發出廣告訊號|yes |yes |
| 傳遞 CEA 608/708 字幕 |yes |yes |
| 在比重摘要內能從短暫延遲中復原的能力 |yes |否 (經過 6 秒以上且未有任何輸入資料時，LiveEvent 便會開始靜態圖像)|
| 支援未統一輸入的 GOP |yes |否 - 輸入必須固定為 2 秒 GOP |
| 支援變動畫面播放速率輸入 |yes |否 – 輸入必須為固定畫面播放速率。<br/>輕微的差異可以接受，例如：處於高速動態場景的情況。 但編碼器無法降至 10 個畫面/秒的標準。 |
| 在輸入摘要遺失時自動關閉 LiveEvent |否 |經過 12 個小時，如果沒有 LiveOutput 仍在執行 |

## <a name="liveevent-states"></a>LiveEvent 狀態 

LiveEvent 的目前狀態。 可能的值包括：

|State|說明|
|---|---|
|**已停止**| 這是 LiveEvent 建立後的初始狀態 (除非指定選取自動啟動)。此狀態中不會計費。 在此狀態下，LiveEvent 屬性可以更新，但是不允許串流。|
|**啟動中**| 正在啟動 LiveEvent。 此狀態中不會計費。 在此狀態期間允許任何更新或串流。 如果發生錯誤，LiveEvent 會回到「已停止」狀態。|
|**執行中**| LiveEvent 能夠處理即時資料流。 現在針對使用量計費。 您必須停止 LiveEvent 來防止進一步計費。|
|**停止中**| 正在停止 LiveEvent。 此暫時性狀態中不會計費。 在此狀態期間允許任何更新或串流。|
|**刪除中**| 正在刪除 LiveEvent。 此暫時性狀態中不會計費。 在此狀態期間允許任何更新或串流。|

## <a name="liveoutput"></a>LiveOutput

[LiveOutput](https://docs.microsoft.com/rest/api/media/liveoutputs) 可讓您控制即時資料流的發佈、記錄和 DVR 時間設定。 LiveEvent 和 LiveOutput 的關聯性類似於傳統媒體，此處的頻道 (LiveEvent) 有常數內容資料流，而節目 (LiveOutput) 的範圍是該 LiveEvent 上的某些計時事件。
您可以透過設定 **ArchiveWindowLength** 屬性，指定您想要保留已記錄 LiveOutput 內容的時數。 **ArchiveWindowLength** 是 ISO 8601 時間範圍內的持續封存時間長度 (數位影像錄影機或 DVR)。 此值最小可以設定為 5 分鐘，最大可以設定為 25 個小時。 

**ArchiveWindowLength** 也指定用戶端可從目前即時位置往回搜尋的最大次數。 LiveOutput 在超過指定的時間量後還是可以執行，但是會持續捨棄落後時間範圍長度的內容。 此屬性值也會決定用戶端資訊清單可以擴充的時間。

每個 LiveOutput 都與一個[資產](https://docs.microsoft.com/rest/api/media/assets)相關聯，並將資料記錄到附加至媒體服務帳戶之 Azure 儲存體中的容器。 若要發佈 LiveOutput，您必須建立相關聯資產的 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators)。 擁有此定位器，可讓您建置可提供給用戶端的串流 URL。

LiveEvent 支援最多三個同時執行的 LiveOutput，因此您可以建立相同內送資料流的多個封存。 這可讓您視需要發行和封存事件的不同部分。 例如，您的商務需求是廣播 24x7 即時線性摘要，但您想要建立當天節目的「錄製」，以提供給客戶作為可點播觀看的隨選內容。  在此案例中，您會先建立封存時間不到 1 小時 (含) 的主要 LiveOutput，讓客戶調整作為主要即時資料流。 您會建立此 LiveOutput 的 StreamingLocator，並將它發佈至您的應用程式或網站作為「即時」摘要。  執行摘要時，您可以利用程式設計方式在節目開頭建立第二個並行 LiveOutput (或提早 5 分鐘以提供稍後要修剪的一些控點)。此第二個 LiveOutput 可在節目或活動結束 5 分鐘後停止，您可以接著建立新的 StreamingLocator，將此節目發佈為應用程式類別目錄中的隨選資產。  您可以針對想要視需要立即共用的其他節目範圍或精選重複此程序多次，在此期間，第一個 LiveOutput 中的「即時」摘要會繼續廣播線性摘要。  此外，您可以利用動態篩選支援來修剪 LiveOutput 中的封存頭尾，以在節目之間引進「重疊安全」，並達成更精確的內容開頭和結尾。 您也可以將封存內容提交[轉換](https://docs.microsoft.com/rest/api/media/transforms)，以編碼為多種輸出格式或建立畫面精準的多種輸出格式子剪輯，作為其他服務的摘要整合。

## <a name="streamingendpoint"></a>StreamingEndpoint

讓資料流流入 LiveEvent 之後，您即可建立「資產」、LiveOutput 和 StreamingLocator 來開始串流事件。 這會封存資料流，並透過 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 將它提供給檢視器。

建立媒體服務帳戶時，預設串流端點會新增至「已停止」狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於「執行中」狀態。

## <a name="billing"></a>計費

LiveEvent 只要其狀態轉換為「執行中」，即會開始計費。 若要停止 LiveEvent 計費，您必須停止 LiveEvent。

> [!NOTE]
> 當 [LiveEvent](https://docs.microsoft.com/rest/api/media/liveevents) 上的 **LiveEventEncodingType** 設定為 [基本] 時，媒體服務會在輸入摘要遺失且沒有 LiveOutput 仍在執行 12 小時後，自動關閉器任何仍處於「執行中」狀態的 LiveEvent。 不過，您仍將需要支付 LiveEvent 處於「執行中」狀態時間的費用。
>

下表顯示 LiveEvent 狀態如何對應至計費模式。

| LiveEvent 狀態 | 會計費嗎？ |
| --- | --- |
| 啟動中 |無 (暫時性狀態) |
| 執行中 |是 |
| 停止中 |無 (暫時性狀態) |
| 已停止 |否 |

## <a name="next-steps"></a>後續步驟

[即時串流教學課程](stream-live-tutorial-with-api.md)
