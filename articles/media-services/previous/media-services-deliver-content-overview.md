---
title: 將內容傳遞給客戶 | Microsoft Docs
description: 本主題簡介當您利用 Azure 媒體服務傳遞內容時會涉及各個方面。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 89ede54a-6a9c-4814-9858-dcfbb5f4fed5
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2017
ms.author: juliako
ms.openlocfilehash: 1d1506e26beec3cc48a904ddeb9bbb4e7656a08e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783747"
---
# <a name="deliver-content-to-customers"></a>將內容傳遞給客戶
當您將串流或點播視訊內容傳遞給客戶時，您的目標是在不同的網路條件下將高品質的視訊傳遞到各種裝置。

若要達成此目標，您可以：

* 將您的串流編碼成多位元速率 (調適性位元速率) 視訊串流。 這會負責處理品質和網路狀況。
* 使用 Microsoft Azure 媒體服務 [動態封裝](media-services-dynamic-packaging-overview.md) 將串流動態地重新封裝至不同的通訊協定。 這會負責處理不同裝置上的串流處理。 媒體服務支援傳遞下列自適性串流技術： <br/>
    * **HTTP 即時串流** (HLS) - 將 "(format=m3u8-aapl)" 路徑新增至 URL 的 "/Manifest" 部分，來告訴串流原始伺服器傳回 HLS 內容以供在 **Apple iOS** 原生裝置上取用 (如需詳細資料，請參閱[定位器](#locators)和 [URL](#URLs))，
    * **MPEG DASH** - 將 "(format=mpd-time-csf)" 路徑新增至 URL 的 "/Manifest" 部分，來告訴串流原始伺服器傳回 MPEG-DASH (如需詳細資料，請參閱[定位器](#locators)和 [URLs](#URLs))，
    * **Smooth Streaming**。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 

本文提供內容傳遞概念的重要概觀。

若要查閱已知問題，請參閱 [已知問題](media-services-deliver-content-overview.md#known-issues)。

## <a name="dynamic-packaging"></a>動態封裝
媒體服務提供的動態封裝，可讓您以媒體服務支援的串流格式 (MPEG-DASH、HLS、Smooth Streaming) 提供調適性位元速率 MP4 或 Smooth Streaming 編碼內容，而不必重新封裝成這些串流格式。 我們建議您利用動態封裝傳遞內容。

若要利用動態封裝功能，您必須將您的夾層 (來源) 檔案編碼成一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案。

使用動態封裝，您可以儲存及播放單一儲存格式的檔案。 媒體服務會根據您的要求建置及傳遞適當的回應。

動態封裝適用於標準和高階串流端點。 

如需詳細資訊，請參閱 [動態封裝](media-services-dynamic-packaging-overview.md)。

## <a name="filters-and-dynamic-manifests"></a>篩選器與動態資訊清單
您可以使用媒體服務為資產定義篩選器。 這些篩選器是伺服器端規則，可協助您的客戶執行如下的動作：播放一段特定視訊，或指定您客戶裝置可以處理的一部分音訊和視訊轉譯 (而非與該資產相關的所有轉譯)。 透過當您的客戶要求根據一或多個指定的篩選器來串流視訊時所建立的 *動態資訊清單* ，可達成此篩選。

如需詳細資訊，請參閱 [篩選器與動態資訊清單](media-services-dynamic-manifest-overview.md)。

## <a name="a-idlocatorslocators"></a><a id="locators"/>定位器
若要提供 URL 給使用者，讓使用者可以利用這個 URL 來串流或下載內容，您必須先建立定位器來發佈您的資產。 定位器提供一個登入點，讓使用者可以存取資產包含的檔案。 媒體服務支援兩種類型的定位器：

* OnDemandOrigin 定位器。 這些定位器可用來串流媒體 (例如，MPEG-DASH、HLS 或 Smooth Streaming) 或漸進式下載檔案。
* 共用存取簽章 (SAS) URL 定位器。 這些定位器可用來將媒體檔案下載到本機電腦。

「存取原則」  可用來定義權限 (例如讀取、寫入和列出)，以及用戶端可存取特定資產多久的時間。 請注意，建立 OrDemandOrigin 定位器時，不應使用列出權限 (AccessPermissions.List)。

定位器有到期日。 Azure 入口網站會為定位器設定 100 年後的到期日。

> [!NOTE]
> 如果您在 2015 年 3 月之前使用 Azure 入口網站建立定位器，這些定位器已設定為兩年之後到期。
> 
> 

若要更新定位器的到期日，請使用 [REST](https://docs.microsoft.com/rest/api/media/operations/locator#update_a_locator) 或 [.NET](http://go.microsoft.com/fwlink/?LinkID=533259) API。 請注意，當您更新 SAS 定位器的到期日，URL 也會隨之變更。

定位器並不是用來控制每個使用者的權限。 您可以使用數位版權管理 (DRM) 方案，給予個別使用者不同的存取權限。 如需詳細資訊，請參閱 [保護媒體](http://msdn.microsoft.com/library/azure/dn282272.aspx)。

建立定位器時，由於 Azure 儲存體需要執行一些儲存體和傳播程序，因此有時候會延遲 30 秒的時間。

## <a name="adaptive-streaming"></a>調適性串流處理
調適性位元速率技術可讓視訊播放器應用程式判斷網路狀況，並由數種位元速率中進行選擇。 當網路通訊降級時，用戶端可以選取較低的位元速率，讓播放能夠以較低的視訊品質繼續。 當網路狀況改善時，用戶端可以切換至較高的位元速率，以提高視訊品質。 Azure 媒體服務支援下列調適性位元速率技術：HTTP 即時串流 (HLS)、Smooth Streaming 和 MPEG-DASH。

若要提供漸進式下載 URL 給使用者，您必須先建立 OnDemandOrigin 定位器。 建立定位器可針對包含您要串流之內容的資產，提供其基底路徑。 不過，若要能夠串流此內容，您還需要進一步修改此路徑。 若要建構串流資訊清單檔案的完整 URL，您必須串連定位器的路徑值和資訊清單 (filename.ism) 的檔案名稱。 接著，在定位器路徑後面附加 **/Manifest** 和適當的格式 (如果需要)。

> [!NOTE]
> 您也可以透過 SSL 連線串流您的內容。 若要這樣做，請確定您的串流 URL 以 HTTPS 開頭。 請注意，目前 AMS 不支援使用 SSL 搭配自訂網域。  
> 

只有當您傳遞內容的來源串流端點是在 2014 年 9 月 10 日之後建立時，才能透過 SSL 串流。 如果您的串流 URL 是根據 2014 年 9 月 10 日之後建立的串流端點，則 URL 會包含 "streaming.mediaservices.windows.net"。 包含 "origin.mediaservices.windows.net" (舊格式) 的串流 URL 不支援 SSL。 如果您的 URL 是舊格式，而且您希望能夠透過 SSL 串流，請建立新的串流端點。 使用根據新的串流端點的 URL，透過 SSL 串流內容。

## <a name="a-idurlsstreaming-url-formats"></a><a id="URLs"/>Streaming URL 格式

### <a name="mpeg-dash-format"></a>MPEG-DASH 格式
{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=mpd-time-csf)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)

### <a name="apple-http-live-streaming-hls-v4-format"></a>Apple HTTP Live Streaming (HLS) V4 格式
{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

### <a name="apple-http-live-streaming-hls-v3-format"></a>Apple HTTP Live Streaming (HLS) V3 格式
{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl-v3)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

### <a name="apple-http-live-streaming-hls-format-with-audio-only-filter"></a>Apple HTTP Live Streaming (HLS) 格式搭配僅限音訊的篩選條件
根據預設，只有音訊的播放軌是包含在 HLS 資訊清單中。 這對於行動電話通訊網路的 Apple Store 認證是必要條件。 在此情況下，如果用戶端沒有足夠的頻寬，或透過 2G 網路連線，播放就會切換成只有音訊。 這可協助保護內容串流，而不需要緩衝處理，但是沒有視訊。 在某些情況中，播放程式緩衝處理可能比只有音訊更好。 如果您想要移除只有音訊的播放軌，可以將 **audio-only=false** 加入 URL。

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3,audio-only=false)

如需詳細資訊，請參閱 [Dynamic Manifest Composition support and HLS output additional features (動態資訊清單組合支援和 HLS 輸出額外功能)](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。

### <a name="smooth-streaming-format"></a>Smooth Streaming 格式
{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest

範例：

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

### <a id="fmp4_v20"></a>Smooth Streaming 2.0 資訊清單 (舊版資訊清單)
根據預設，Smooth Streaming 資訊清單格式包含重複的標記 (r-tag)。 不過，有些播放程式不支援 r-tag。 使用這些播放程式的用戶端可以使用停用 r-tag 的格式︰

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=fmp4-v20)

    http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=fmp4-v20)

## <a name="progressive-download"></a>漸進式下載
使用漸進式下載可以在下載完整檔案前就開始播放媒體。 您無法漸進式下載 .ism* (ismv、isma、ismt 或 ismc) 檔案。

若要漸進式下載內容，請使用 OnDemandOrigin 類型的定位器。 下列範例顯示的 URL 採用的是 OnDemandOrigin 類型的定位器：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

您必須解密任何您想要從原始服務串流的儲存體加密資產，這樣才能漸進式下載。

## <a name="download"></a>下載
若要將您的內容下載到用戶端裝置，您必須建立一個 SAS 定位器。 您可以利用 SAS 定位器存取檔案所在的 Azure 儲存體容器。 若要建立下載 URL，您必須將檔案名稱內嵌在主機與 SAS 簽章之間。

下列範例顯示的 URL 是採用 SAS 定位器：

    https://test001.blob.core.windows.net/asset-ca7a4c3f-9eb5-4fd8-a898-459cb17761bd/BigBuckBunny.mp4?sv=2012-02-12&se=2014-05-03T01%3A23%3A50Z&sr=c&si=7c093e7c-7dab-45b4-beb4-2bfdff764bb5&sig=msEHP90c6JHXEOtTyIWqD7xio91GtVg0UIzjdpFscHk%3D

您必須考量下列事項：

* 您必須解密任何您想要從原始服務串流的儲存體加密資產，這樣才能漸進式下載。
* 未在 12 小時內完成的下載，最後一定會失敗。

## <a name="streaming-endpoints"></a>串流端點

串流端點代表可以直接將內容傳遞給用戶端播放程式應用程式，或傳遞給內容傳遞網路 (CDN) 進行進一步發佈的串流服務。 來自串流端點服務的輸出串流可以是即時資料流，或媒體服務帳戶中的隨選視訊資產。 串流端點有兩種類型：「標準」和「進階」。 如需詳細資訊，請參閱[串流端點概觀](media-services-streaming-endpoints-overview.md)。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 

## <a name="known-issues"></a>已知問題
### <a name="changes-to-smooth-streaming-manifest-version"></a>Smooth Streaming 資訊清單版本變更
在 2016 年 7 月之前的服務版本 -- 使用動態封裝串流媒體編碼器標準、媒體編碼器高階工作流程或舊版 Azure 媒體編碼器所產生的資產時 -- 傳回的 Smooth Streaming 資訊清單會符合 2.0 版。 在 2.0 版中，片段持續期間不使用所謂的重複 ('r') 標籤。 例如︰

<?xml version="1.0" encoding="UTF-8"?>
    <SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="8000" TimeScale="1000">
        <StreamIndex Chunks="4" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="3" Subtype="" Name="video" TimeScale="1000">
            <QualityLevel Index="0" Bitrate="1000000" FourCC="AVC1" MaxWidth="640" MaxHeight="360" CodecPrivateData="00000001674D4029965201405FF2E02A100000030010000003032E0A000F42400040167F18E3050007A12000200B3F8C70ED0B16890000000168EB7352" />
            <c t="0" d="2000" n="0" />
            <c d="2000" />
            <c d="2000" />
            <c d="2000" />
        </StreamIndex>
    </SmoothStreamingMedia>

在 2016 年 7 月之後的服務版本中，產生的 Smooth Streaming 資訊清單會符合 2.2 版，即片段持續時間使用重複標籤。 例如︰

    <?xml version="1.0" encoding="UTF-8"?>
    <SmoothStreamingMedia MajorVersion="2" MinorVersion="2" Duration="8000" TimeScale="1000">
        <StreamIndex Chunks="4" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="3" Subtype="" Name="video" TimeScale="1000">
            <QualityLevel Index="0" Bitrate="1000000" FourCC="AVC1" MaxWidth="640" MaxHeight="360" CodecPrivateData="00000001674D4029965201405FF2E02A100000030010000003032E0A000F42400040167F18E3050007A12000200B3F8C70ED0B16890000000168EB7352" />
            <c t="0" d="2000" r="4" />
        </StreamIndex>
    </SmoothStreamingMedia>

某些舊版的 Smooth Streaming 用戶端可能不支援重複標籤，因此將無法載入資訊清單。 若要解決這個問題，您可以使用舊版資訊清單格式參數 **(format=fmp4-v20)** ，或將用戶端更新為支援重複標籤的最新版本。 如需詳細資訊，請參閱 [Smooth Streaming 2.0](media-services-deliver-content-overview.md#fmp4_v20)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-topics"></a>相關主題
[啟動儲存體金鑰之後更新媒體服務定位器](media-services-roll-storage-access-keys.md)

