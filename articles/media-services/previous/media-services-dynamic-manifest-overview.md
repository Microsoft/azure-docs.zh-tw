---
title: 篩選器與動態資訊清單 | Microsoft Docs
description: 本主題說明如何建立篩選器，讓您的用戶端可以使用篩選器來串流特定的資料流區段。 媒體服務會建立動態資訊清單來封存此選擇性資料流。
services: media-services
documentationcenter: ''
author: cenkdin
manager: cfowler
editor: ''
ms.assetid: ff102765-8cee-4c08-a6da-b603db9e2054
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/22/2018
ms.author: cenkd;juliako
ms.openlocfilehash: 982af37a866f73292192b0c889e9eeb1e1291030
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783717"
---
# <a name="filters-and-dynamic-manifests"></a>篩選器與動態資訊清單
從 2.17 版開始，媒體服務可讓您為資產定義篩選。 這些篩選器是伺服器端規則，可讓您的客戶選擇執行如下的動作：只播放一段視訊 (而非播放完整視訊)，或只指定您客戶裝置可以處理的一部分音訊和視訊轉譯 (而非與該資產相關的所有轉譯)。 透過在您客戶要求下建立的 **動態資訊清單**可達成對資訊進行這樣的篩選，藉此根據指定的篩選器來串流視訊。

本主題討論對您的客戶很有幫助的常見篩選使用案例，並提供主題連結以示範如何以程式設計方式建立篩選。

## <a name="overview"></a>概觀
當您將內容傳遞給客戶時 (串流即時事件或點播視訊)，您的目標是在不同的網路條件下將高品質的視訊傳遞到各種裝置。 若要達成此目標，請執行下列動作：

* 將您的資料流編碼成多位元速率 ([彈性位元速率](http://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)) 視訊串流 (這會處理品質與網路條件)，並 
* 使用媒體服務 [動態封裝](media-services-dynamic-packaging-overview.md) 將資料流動態地重新封裝至不同的通訊協定 (這會處理不同裝置上的資料流)。 媒體服務支援傳遞下列自適性串流技術：HTTP 即時串流 (HLS)、Smooth Streaming 和 MPEG DASH。 

### <a name="manifest-files"></a>資訊清單檔案
當您將資產編碼以進行彈性位元速率資料流時，會建立 **資訊清單** (播放清單) 檔案 (此檔案是以文字或 XML 為基礎)。 **資訊清單** 檔案包含資料流中繼資料，例如：資料軌類型 (音訊、視訊或文字)、資料軌名稱、開始和結束時間、位元速率 (品質)、資料軌語言、簡報視窗 (持續時間固定的滑動視窗)，視訊轉碼器 (FourCC)。 此檔案也會透過提供下一個可播放視訊片段及其位置的相關資訊，來指示播放程式擷取下一個片段。 片段 (或區段) 實際上是視訊內容的「區塊」。

資訊清單檔案範例如下： 

    <?xml version="1.0" encoding="UTF-8"?>    
    <SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="330187755" TimeScale="10000000">

    <StreamIndex Chunks="17" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="8">
    <QualityLevel Index="0" Bitrate="5860941" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931300016E360000E4E1FF8C7076850A4580000000168E9093525" />
    <QualityLevel Index="1" Bitrate="4602724" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931100011EDC00002CD29FF8C7076850A45800000000168E9093525" />
    <QualityLevel Index="2" Bitrate="3319311" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FAC2CA5014016EC054808080A00000300020000030064C0800067C28000103667F8C7076850A4580000000168E9093525" />
    <QualityLevel Index="3" Bitrate="2195119" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C1000044AA0000ABA9FE31C1DA14291600000000168E9093525" />
    <QualityLevel Index="4" Bitrate="1469881" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C04000B71A0000E4E1FF8C7076850A4580000000168E9093525" />
    <QualityLevel Index="5" Bitrate="978815" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C08001E8480004C4B7F8C7076850A45800000000168E9093525" />
    <QualityLevel Index="6" Bitrate="638374" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C080013D60000C65DFE31C1DA1429160000000168E9093525" />
    <QualityLevel Index="7" Bitrate="388851" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DAC2CA505067E7C054830303200000300020000030064C040030D40003D093F8C7076850A45800000000168E9093525" />

    <c t="0" d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="9600000"/>
    </StreamIndex>


    <StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_128kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_128kbps">
    <QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="125658" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

    <c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>


    <StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_56kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_56kbps">
    <QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="53655" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

    <c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>

    </SmoothStreamingMedia>

### <a name="dynamic-manifests"></a>動態資訊清單
提供許多當您的用戶端需要比預設資產資訊清單檔更多彈性的 [案例](media-services-dynamic-manifest-overview.md#scenarios) 。 例如︰

* 裝置特有：只傳遞用來播放內容的裝置所支援的指定轉譯和/或指定的語言資料軌 (「轉譯篩選」)。 
* 縮小資訊清單以顯示即時事件的子剪輯 (「子剪輯篩選」)。
* 修剪視訊開頭 (「修剪視訊」)。
* 調整 簡報視窗 (DVR)，以便在播放程式中提供長度有限的 DVR 視窗 (「調整簡報視窗」)。

為達到這種彈性，媒體服務會根據預先定義的 **篩選器** 提供 [動態資訊清單](media-services-dynamic-manifest-overview.md#filters)。  一旦您定義了篩選器，您的用戶端便會使用篩選器來串流視訊的特定轉譯或子剪輯。 用戶端會在資料流 URL 中指定篩選器。 篩選器可以套用到下列 [動態封裝](media-services-dynamic-packaging-overview.md)支援的自適性串流通訊協定：HLS、MPEG-DASH 和 Smooth Streaming。 例如︰

含篩選器的 MPEG DASH URL

    http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf,filter=MyLocalFilter)

含篩選器的 Smooth Streaming URL

    http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyLocalFilter)


如需如何傳遞內容和建置資料流 URL 的詳細資訊，請參閱 [傳遞內容概觀](media-services-deliver-content-overview.md)。

> [!NOTE]
> 請注意，動態資訊清單不會變更資產和該資產的預設資訊清單。 您的用戶端可以選擇要求包含或不含篩選器的資料流。 
> 
> 

### <a id="filters"></a>篩選器
資產篩選器有兩種： 

* 全域篩選器 (可以套用到 Azure 媒體服務帳戶中所有的資產，擁有帳戶的存留期間) 和 
* 本機篩選器 (只能套用篩選器建立時與其相關聯的資產，擁有資產的存留期間)。 

全域和本機篩選器類型有完全相同的屬性。 這兩個的主要差異在於案例較適合哪一種篩選器。 全域篩選器通常適用於裝置設定檔 (轉譯篩選)，而本機篩選器可用於修剪特定的資產。

## <a id="scenarios"></a>常見案例
先前提過，將內容傳遞給客戶時 (串流即時事件或點播視訊)，您的目標是在不同的網路條件下將高品質的視訊傳遞到各種裝置。 此外，您可能會有其他對於篩選資產與使用 **動態資訊清單**相關的需求。 以下幾節提供不同篩選案例的簡短概觀。

* 僅指定某些裝置可以處理的音訊和視訊轉譯子集 (而非與該資產相關聯的所有轉譯)。 
* 僅播放視訊的某個區段 (而非播放整個視訊)。
* 調整 DVR 簡報視窗。

## <a name="rendition-filtering"></a>轉譯篩選
您可以選擇將資產編碼成多個編碼設定檔 (H.264 Baseline、H.264 High、AACL、AACH、Dolby Digital Plus)，及多個高品質位元速率。 不過，並非所有的用戶端裝置都支援您所有的資產和位元速率。 例如，較舊的 Android 裝置只支援 H.264 Baseline+AACL。 將較高的位元速率傳送到無法獲益的裝置，將會浪費頻寬及裝置計算。 這類裝置必須將所有指定的資訊解碼，才能縮小以顯示。

有了動態資訊清單，您可以建立裝置設定檔，例如行動、主控台、HD/SD 等，並包括您想要納入設定檔中的特質與品質。

![轉譯篩選範例][renditions2]

下列範例使用編碼器將夾層資產編碼成七個 ISO MP4 視訊轉譯 (從 180p 到 1080p)。 編碼的資產可以動態地封裝至下列任何一個資料流通訊協定：HLS、Smooth 和 MPEG DASH。  圖表頂端顯示不含篩選器的資產其 HLS 資訊清單 (包含全部七個轉譯)。  左下角顯示名為 "ott" 的篩選器已套用到 HLS 資訊清單。 "ott" 篩選器指定要移除所有不到 1Mbps 的位元速率，因此將最差的兩個品質等級從回應中去除。  右下角顯示名為 "mobile" 的篩選器已套用到 HLS 資訊清單中。 "mobile" 篩選器指定要移除的解析度大於 720p 的轉譯，因此去除兩個 1080p 的轉譯。

![轉譯篩選][renditions1]

## <a name="removing-language-tracks"></a>移除語言資料軌
您的資產可能包含多個音訊語言，例如英文、西班牙文、法文等。通常，播放程式 SDK 管理員會預設音訊資料軌選取範圍，以及每個使用者可選擇的可用音訊資料軌。 開發這類的播放程式 SDK 相當有挑戰性，因為在各個裝置特有的播放程式架構之間有不同的實作方式。 此外播放程式 API 在某些平台上受到限制，且不包含音訊選擇功能，因此使用者無法選取或變更預設的音訊資料軌。有了資產篩選器，您可以藉由建立只包含所需音訊語言的篩選器來控制行為。

![語言資料軌篩選][language_filter]

## <a name="trimming-start-of-an-asset"></a>修剪資產開頭
在大部分的即時資料流事件中，操作人員必須在實際的事件前先進行測試。 例如，他們可以在事件開始前包含如下 slate 訊息：「程式將立刻開始」。 如果程式正在進行封存，則測試和靜態圖像資料也會一併封存並包含在簡報中。 但是此資訊不會對用戶端顯示。 透過動態資訊清單，您可以建立開始時間篩選器，並從資訊清單中移除不必要的資料。

![開始修剪][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>從即時封存中建立子剪輯 (檢視)
許多即時事件的執行時間長，且即時封存可能包括多個事件。 即時事件結束後，廣播者可能想要將即時封存分解成邏輯程式的啟動和停止序列。 接下來，個別發佈這些虛擬程式，但不後續處理即時封存，亦不建立個別的資產 (這無法運用 CDN 中現有快取片段的優點)。 例如足球或籃球比賽的球季、棒球賽局，或是任何體育節目的個別事件，都是此類虛擬程式的範例。

使用動態資訊清單，您便可以使用開始/結束時間建立篩選器，並在即時封存頂端建立虛擬檢視。 

![子剪輯篩選][subclip_filter]

已篩選的資產：

![滑動][skiing]

## <a name="adjusting-presentation-window-dvr"></a>調整簡報視窗 (DVR)
目前，Azure 媒體服務提供持續時間可設為 5 分鐘到 25 小時的循環封存。 資訊清單篩選可以在封存頂端建立循環 DVR 視窗封存，而不會刪除媒體。 在許多情況下，廣播者想要提供受限制的 DVR 時段以隨著即時邊緣移動，並同時保留更大的封存視窗。 廣播者可能會想要使用超出 DVR 視窗的資料來反白顯示剪輯，或者想要為不同的裝置提供不同的 DVR 視窗。 例如，大多數的行動裝置不處理大型 DVR 時段 (您可以讓行動裝置有 2 分鐘的 DVR 時段，桌上型用戶端有一小時的 DVR 時段)。

![DVR 視窗][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>調整 LiveBackoff (即時位置)
資訊清單篩選可用來移除即時程式其即時邊緣幾秒鐘的時間。 篩選可讓廣播者在檢視者收到資料流前 (倒退 30 秒) 先觀賞預覽發佈點的簡報，並建立廣告插入點。 廣播者接著可將這些廣告及時推送到其用戶端架構，讓廣播者能夠收到與處理資訊，才有機會播放廣告。

除了廣告支援外，LiveBackoff 設定可用來調整檢視者位置，讓用戶端移動並點閱即時邊緣時，仍然可以從伺服器取得片段，而非得到 404 或 412 HTTP 錯誤。

![livebackoff_filter][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>將多個規則結合成單一篩選器
您可以將多個篩選規則結合成單一篩選器。 例如，您可以定義一個「範圍規則」，將靜態圖像從即時封存中移除，並同時篩選出可用的位元速率。 套用多個篩選規則時，最終結果會是所有規則的交集。

![多個規則][multiple-rules]

## <a name="create-filters-programmatically"></a>以程式設計方式建立篩選器
下列文章討論與篩選相關的媒體服務實體。 此文章也會示範如何以程式設計方式建立篩選。  

[使用 REST API 建立篩選器](media-services-rest-dynamic-manifest.md)。

## <a name="combining-multiple-filters-filter-composition"></a>結合多個篩選條件 (篩選器組合)
您也可以將多個篩選條件結合成單一 URL。 

下列案例示範您可能會想結合篩選條件的理由：

1. 您要篩選例如 Android 或 iPAD 等行動裝置的視訊品質 (以限制視訊品質)。 若要移除不必要的品質，您可以建立適用於裝置設定檔的全域篩選。 如本文章稍早所述，全域篩選可以用於相同媒體服務帳戶下所有的資產，而不需進一步的關聯。 
2. 您也想要修剪資產的開始和結束時間。 若要達到此目的，您可建立本機篩選條件，並設定開始/結束時間。 
3. 您必須結合這兩個篩選 (若不組合，您便需要將品質篩選加入修剪篩選，這會讓篩選更難以使用)。

若要結合篩選條件，您必須設定資訊清單/播放清單 URL 的篩選條件名稱，並以分號分隔。 假設您擁有名為 *MyMobileDevice* 的篩選條件，可篩選品質，且您有另一個名為 *MyStartTime* 的篩選條件，用來設定特定的開始時間。 您可以將它們結合如下：

    http://teststreaming.streaming.mediaservices.windows.net/3d56a4d-b71d-489b-854f-1d67c0596966/64ff1f89-b430-43f8-87dd-56c87b7bd9e2.ism/Manifest(filter=MyMobileDevice;MyStartTime)

您可以結合最多三個篩選。 

如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/) 。

## <a name="know-issues-and-limitations"></a>已知問題與限制
* 動態資訊清單會在 GOP 界限 (主要畫面格) 中運作，因此修剪後能夠有精確的 GOP。 
* 您可以讓本機和全域篩選器使用相同的篩選器名稱。 請注意，本機篩選有較高的優先權，會覆寫全域篩選器。
* 如果您更新篩選器，則資料流端點需要 2 分鐘的時間來重新整理規則。 如果內容是透過使用某些篩選器提供的 (並在 Proxy 和 CDN 快取中快取)，則更新這些篩選器會導致播放程式失敗。 建議在更新篩選器之後清除快取。 如果這個選項無法執行，請考慮使用不同的篩選器。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[將內容傳遞給客戶概觀](media-services-deliver-content-overview.md)

[renditions1]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/media-services-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/media-services-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/media-services-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/media-services-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/media-services-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/media-services-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/media-services-dynamic-manifest-overview/media-services-skiing.png
