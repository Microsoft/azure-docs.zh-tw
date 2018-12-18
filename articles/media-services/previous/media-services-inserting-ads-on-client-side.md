---
title: 在用戶端插入廣告 | Microsoft Docs
description: 本主題說明如何在用戶端上插入廣告。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 65c9c747-128e-497e-afe0-3f92d2bf7972
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: juliako
ms.openlocfilehash: 0ca9c7ff1ac4cfbb50f251679286a933429bf03f
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37903042"
---
# <a name="inserting-ads-on-the-client-side"></a>在用戶端插入廣告
本文包含如何在用戶端上插入各種類型廣告的相關資訊。

如需了解即時串流視訊的隱藏式字幕和廣告支援，請參閱 [隱藏式字幕支援和廣告插入標準](media-services-live-streaming-with-onprem-encoders.md#cc_and_ads)。

> [!NOTE]
> Azure 媒體播放器目前不支援廣告。
> 
> 

## <a id="insert_ads_into_media"></a>將廣告插入您的媒體
Azure 媒體服務允許透過 Windows Media 平台插入廣告：Player Framework。 具備廣告支援的播放器架構都適用於 Windows 8、Silverlight、Windows Phone 8 和 iOS 裝置。 每一個播放器架構都有範例程式碼，教您如何實作播放器應用程式。 目前有三種不同的廣告可以插入 media:list 中。

* **線性** – 可暫停主要影片的完整框架廣告。
* **非線性** – 播放被當做主要影片而顯示的疊加廣告，通常是播放器中的標誌或其他靜態影像。
* **隨播** – 顯示在播放器之外的廣告。

廣告可以放在主要影片時間軸的任何時間點。 您必須告訴播放器何時播放廣告以及要播放哪些廣告。 使用一組標準的 XML 格式檔案即可搞定：Video Ad Service Template (VAST)、Digital Video Multiple Ad Playlist (VMAP)、Media Abstract Sequencing Template (MAST) 以及 Digital Video Player Ad Interface Definition (VPAID)。 VAST 檔案會指定要顯示的廣告。 VMAP 檔案會指定何時播放各種廣告而且包含 VAST XML。 MAST 檔案是另一種廣告排序的方法，而且也包含 VAST XML。 VPAID 檔案會定義影片播放器廣告和廣告或廣告伺服器之間的介面。

每個播放器架構的運作方式不同，而且分別涵蓋於各自的文章中。 本文說明用來插入廣告的基本機制。 影片播放程式應用程式會向廣告伺服器要求廣告。 廣告伺服器可以使用數種方法回應：

* 傳回 VAST 檔案
* 傳回 VMAP 檔案 (有內嵌 VAST)
* 傳回 MAST 檔案 (有內嵌 VAST)
* 傳回 VAST 檔案 (有 VPAID 廣告)

### <a name="using-a-video-ad-service-template-vast-file"></a>使用 Video Ad Service Template (VAST) 檔案
VAST 檔案會指定要顯示的廣告。 下列 XML 是線性廣告的 VAST 檔案範例：

```xml
    <VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="115571748">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://www.myserver.com/tracking-resource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <TrackingEvents>
                  <Tracking event="start"><![CDATA[http://www.myserver.com/start-tracking-resource]]></Tracking>
                  <Tracking event="midpoint"><![CDATA[http://www.myserver.com/midpoint-tracking-resource]]></Tracking>
                  <Tracking event="complete"><![CDATA http://www.myserver.com/complete-tracking-resource]]></Tracking>
                  <Tracking event="expand"><![CDATA[http://www.myserver.com/expand-tracking-resource]]></Tracking>
                </TrackingEvents>
                <VideoClicks>
                  <ClickThrough id="Atlas Redirect"><![CDATA[http://www.myserver.com/click-resource]]></ClickThrough>
                  <ClickTracking id="Spare"></ClickTracking>
                </VideoClicks>
                <MediaFiles>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_200_4x3.wmv]]>
                  </MediaFile>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_300_4x3.wmv]]>
                  </MediaFile>
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
          <Extensions>
            <Extension type="Atlas">
            </Extension>
          </Extensions>
        </InLine>
      </Ad>
    </VAST>
```

線性廣告是利用 <**Linear**> 元素來說明。 它會指定廣告的持續時間、追蹤事件、點選連結、點選追蹤，以及許多 **MediaFile** 元素。 追蹤事件是在 <**TrackingEvents**> 元素中指定，並允許廣告伺服器追蹤在檢視廣告時所發生的各種事件。 在此案例中會追蹤開始、中間點、完成及展開事件。 當顯示廣告時，就會發生開始事件。 當至少已經檢視 50% 的廣告時間軸時，就會發生中間點事件。 當廣告播放到結尾時，就會發生完成事件。 當使用者將視訊播放器展開至全螢幕時，就會發生展開事件。 點選連結是以 <**VideoClicks**> 元素內的 <**ClickThrough**> 元素指定，並指定當使用者按一下廣告時要顯示資源的 URI。 點選追蹤是在 <**ClickTracking**> 元素 (同樣位於 <**VideoClicks**> 元素內) 中指定，並指定在使用者按一下廣告時播放器要求的追蹤資源。 <**MediaFile**> 元素會指定廣告特定編碼的相關資訊。 若有一個以上的 <**MediaFile**> 元素，視訊播放器就可以選擇最適合平台的編碼。 

線性廣告可以依照指定的順序顯示。 若要這樣做，請將其他 <Ad> 元素加入至 VAST 檔案，並使用順序屬性指定順序。 下列範例會加以說明：

```xml
    <VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="1" sequence="0">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad1trackingResouce]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
      <Ad id="2" sequence="1">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad2trackingResouce]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:30</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
    </VAST>
```

非線性廣告也是在 <Creative> 元素中指定。 下列範例顯示 <Creative> 元素，該元素會說明非線性廣告。

```xml
    <Creative id="video" sequence="1" AdID="">
      <NonLinearAds>
        <NonLinear width="216" height="121" minSuggestedDuration="00:00:15">
          <StaticResource creativeType="image/png"><![CDATA[http://myserver/images/image.png]]></StaticResource>
          <StaticResource creativeType="image/jpg"><![CDATA[http://myserver/images/image.jpg]]></StaticResource>
        </NonLinear>
        <TrackingEvents>
             <Tracking event="acceptInvitation"><![CDATA[http://myserver/tracking/trackingID]></Tracking>
             <Tracking event="collapse"><![CDATA[http://myserver/tracking/trackingID2]]></Tracking>
         </TrackingEvents>
       </NonLinearAds>
    </Creative>
```

<**NonLinearAds**> 元素可以包含一或多個 <**NonLinear**> 元素，其中每一個元素都可說明一個非線性廣告。 <**NonLinear**> 元素會指定非線性廣告的資源。 資源可以是 <**StaticResouce**>、<**IFrameResource**> 或 <**HTMLResouce**>。 <**StaticResource**> 說明非 HTML 資源，並且定義 creativeType 屬性，該屬性會指定資源的顯示方式：

image/gif、image/jpeg、image/png – 資源在 HTML <**img**> 標記中顯示。

Application/x-javascript – 資源在 HTML <**script**> 標籤中顯示。

Application/x-shockwave-flash – 資源在 Flash Player 中顯示。

**IFrameResource** 說明可在 IFrame 中顯示的 HTML 資源。 **HTMLResource** 說明可插入網頁的 HTML 程式碼片段。 **TrackingEvents** 指定追蹤事件以及當發生事件時要求的 URI。 在此範例中，會追蹤 acceptInvitation 和 collapse 事件。 如需有關 **NonLinearAds** 元素和其子系的詳細資訊，請參閱 IAB.NET/VAST。 請注意，**TrackingEvents** 元素位於 **NonLinearAds** 元素內，而不是 **NonLinear** 元素中。

隨播廣告會在 <CompanionAds> 元素內定義。 <CompanionAds> 元素可以包含一或多個 <Companion> 元素。 每個 <Companion> 元素都會說明隨播廣告，而且可以包含 <StaticResource>、<IFrameResource> 或 <HTMLResource>，其指定方式與非線性廣告相同。 VAST 檔案可以包含多個隨播廣告，而且播放器應用程式可以選擇要顯示的最適當廣告。 如需 VAST 的詳細資訊，請參閱 [VAST 3.0](http://www.iab.net/media/file/VASTv3.0.pdf)。

### <a name="using-a-digital-video-multiple-ad-playlist-vmap-file"></a>使用 Digital Video Multiple Ad Playlist (VMAP) 檔案
VMAP 檔案可讓您指定何時插播廣告、每個插播多久、插播中可以顯示多少廣告，以及插播期間可以顯示哪些類型的廣告。 範例 VMAP 檔案中的下列項目會定義單一廣告插播：

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
              <Ad id="115571748">
                <InLine>
                  <AdSystem version="2.0 alpha">Atlas</AdSystem>
                  <AdTitle>Unknown</AdTitle>
                  <Description>Unknown</Description>
                  <Survey></Survey>
                  <Error></Error>
                  <Impression id="Atlas"><![CDATA[http://view.atdmt.com/000/sview/115571748/direct;ai.201582527;vt.2/01/634364885739970673]]></Impression>
                  <Creatives>
                    <Creative id="video" sequence="0" AdID="">
                      <Linear>
                        <Duration>00:00:32</Duration>
                        <MediaFiles>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_200_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_300_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_500" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="500" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_500_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_700" maintainAspectRatio="true" scaleable="true" delivery="progressive" bitrate="700" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv]]>
                          </MediaFile>
                        </MediaFiles>
                      </Linear>
                    </Creative>
                  </Creatives>
                </InLine>
              </Ad>
            </VAST>
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

VMAP 檔案開頭為 <VMAP> 元素，包含一或多個 <AdBreak> 元素，每個元素都會定義廣告插播。 每個廣告插播都會指定插播類型、插播 ID 和時間位移。 breakType 屬性會指定插播期間可以播放的廣告類型：線性、非線性或顯示。 顯示廣告會對應到 VAST 隨播廣告。 在逗號 (無空格) 分隔的清單中可以指定一個以上的廣告類型。 breakID 是廣告的選擇性識別碼。 timeOffset 會指定何時應該顯示廣告。 可以利用下列方式之一來指定：

1. 時間 – hh: mm: 或 hh:mm:ss.mmm 的格式，其中.mmm 為毫秒。 這個屬性的值會指定從視訊時間軸開始到廣告插播開始的時間。
2. 百分比 – n% 格式，其中 n 是視訊時間軸在播放廣告之前播放的百分比
3. 開始/結束 – 指定廣告應該在視訊顯示之前或之後顯示
4. 位置 – 指定當廣告插播的時機為未知時 (例如即時串流)，廣告插播的順序。 每個廣告插播的順序是以 #n 格式指定，其中 n 是大於或等於 1 的整數。 1 表示廣告應該在第一個機會時播放，2 表示廣告應該在第二個機會時播放，依此類推。

在 <AdBreak> 元素內可以有一個 <**AdSource**> 元素。 <**AdSource**> 元素包含下列屬性：

1. Id – 指定廣告來源的識別碼
2. allowMultipleAds – 布林值，指定在廣告插播期間是否可以顯示多個廣告
3. followRedirects – 選用布林值，指定視訊播放器是否應該接受廣告回應內的重新導向

<**AdSource**> 元素會提供播放器內嵌廣告回應或廣告回應的參考。 可以包含下列其中一個元素：

* <VASTAdData> 表示 VAST 廣告回應內嵌在 VMAP 檔案
* <AdTagURI> 從另一個系統參考廣告回應的 URI
* <CustomAdData> -表示非 VAST 回應的任意字串

在這個範例中，內嵌廣告回應是以 <VASTAdData> 元素指定，該元素包含 VAST 廣告回應。 如需其他元素的詳細資訊，請參閱 [VMAP](http://www.iab.net/guidelines/508676/digitalvideo/vsuite/vmap)。

<**AdBreak**> 元素也可以包含一個 <**TrackingEvents**> 元素。 <**TrackingEvents**> 元素可讓您追蹤廣告插播的開頭或結束，或廣告插播期間是否發生錯誤。 <**TrackingEvents**> 元素包含一或多個 <**Tracking**> 元素，每個元素會指定追蹤事件和追蹤 URI。 可能的追蹤事件如下：

1. breakStart – 追蹤廣告插播的開始
2. breakEnd – 追蹤廣告插播的完成
3. error – 追蹤廣告插播期間發生的錯誤

下列範例顯示會指定追蹤事件的 VMAP 檔案

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <!--Inline VAST -->
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
          <vmap:Tracking event="breakend">
            http://MyServer.com/breakend.gif
          </vmap:Tracking>
          <vmap:Tracking event="error">
            http://MyServer.com/error.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

如需 <**TrackingEvents**> 元素及其子系的詳細資訊，請參閱 http://iab.net/VMAP.pdf。

### <a name="using-a-media-abstract-sequencing-template-mast-file"></a>使用 Media Abstract Sequencing Template (MAST) 檔案
MAST 檔案可讓您指定觸發程序，定義何時顯示廣告。 以下是範例 MAST 檔案，其中包含片頭廣告、片中廣告和片尾廣告的觸發程序。

```xml
    <MAST xsi:schemaLocation="http://openvideoplayer.sf.net/mast http://openvideoplayer.sf.net/mast/mast.xsd" xmlns="http://openvideoplayer.sf.net/mast" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <triggers>
        <trigger id="preroll" description="preroll every item"  >
          <startConditions>
            <condition type="event" name="OnItemStart" />
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="midroll" description="midroll at 15 sec."  >
          <startConditions>
            <condition type="property" name="Position" value="00:00:15.0" operator="GEQ" />
          </startConditions>
          <endConditions>
            <condition type="event" name="OnItemEnd"/>
            <!--This 'resets' the trigger for the next clip-->
          </endConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="postroll" description="postroll"  >
          <startConditions>
            <condition type="event" name="OnItemEnd"/>
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>
      </triggers>
    </MAST>
```


MAST 檔案開頭為 **MAST** 元素，其中包含一個 **triggers** 元素。 <triggers> 元素包含一或多個 **trigger** 元素，定義何時應該播放廣告。 

**trigger** 元素包含 **startConditions** 元素，可指定何時應該開始播放廣告。 **startConditions** 元素包含一或多個 <condition> 元素。 當每個 <condition> 評估為 true 時，會起始或撤銷觸發程序，取決於 <condition> 是分別內含於 **startConditions** 或 **endConditions** 元素。 當多個 <condition> 元素都存在時，則會視為隱含 OR，任何評估為 true 的條件都會導致起始觸發程序。 <condition> 元素可以為巢狀。 當預設子系 <condition> 元素時，它們會被視為隱含 AND，所有條件必須評估為 true，才會起始觸發程序。 <condition> 元素包含會定義條件的下列屬性： 

1. **type** – 指定條件、事件或屬性的類型
2. **name** – 要在評估期間使用之屬性或事件的名稱
3. **value** – 屬性將會針對其進行評估的值
4. **operator** – 要在評估期間使用的運算：EQ (等於)、NEQ (不等於)、GTR (大於)、GEQ (大於或等於)、LT (小於)、LEQ (小於或等於)、MOD (模數)

**endConditions** 也會包含 <condition> 元素。 當條件評估為 true 時，會重設觸發程序。 <trigger> 元素也包含 <sources> 元素，該元素包含一或多個 <source> 元素。 <source> 元素定義廣告回應的 URI 與廣告回應的類型。 在此範例中，會對 VAST 回應指定 URI。 

```xml
    <trigger id="postroll" description="postroll"  >
      <startConditions>
        <condition/>
      </startConditions>
      <sources>
        <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
          <sources />
        </source>
      </sources>
    </trigger>
```

### <a name="using-video-player-ad-interface-definition-vpaid"></a>使用 Video Player-Ad Interface Definition (VPAID)
VPAID 是 API，用於啟用可執行廣告單元，以便與視訊播放器通訊。 如此可提供高度互動性與體驗。 使用者可以與廣告互動，而且廣告可以回應檢視者採取的動作。 例如，廣告可能會顯示按鈕，讓使用者檢視更多詳細資訊或廣告的加長版。 視訊播放器必須支援 VPAID API，可執行廣告必須實作 API。 當播放器向廣告伺服器要求廣告時，該伺服器可能會以包含 VPAID 廣告的 VAST 回應進行回應。

必須在如 Adobe Flash™ 或可以在網頁瀏覽器中執行的 JavaScript 執行階段環境中執行的程式碼中建立可執行廣告。 當廣告伺服器傳回包含 VPAID 廣告的 VAST 回應時，<MediaFile> 元素中屬性 apiFramework 的值必須是 "VPAID"。 這個屬性會指定包含的廣告是 VPAID 可執行廣告。 類型屬性必須設定為可執行的 MIME 類型，例如 “application/x-shockwave-flash” 或 “application/x-javascript”。 下列 XML 程式碼片段顯示來自包含 VPAID 可執行廣告之 VAST 回應的 <MediaFile> 元素。 

```xml
    <MediaFiles>
       <MediaFile id="1" delivery="progressive" type=”application/x-shockwaveflash”
                  width=”640” height=”480” apiFramework=”VPAID”>
           <!-- CDATA wrapped URI to executable ad -->
       </MediaFile>
    </MediaFiles>
```

可以使用 VAST 回應中 <Linear> 或 <NonLinear> 元素內的 <AdParameters> 元素，初始化可執行廣告。 如需 <AdParameters> 元素的詳細資訊，請參閱 [VAST 3.0](http://www.iab.net/media/file/VASTv3.0.pdf)。 如需 VPAID API 的詳細資訊，請參閱 [VPAID 2.0](http://www.iab.net/media/file/VPAID_2.0_Final_04-10-2012.pdf)。

## <a name="implementing-a-windows-or-windows-phone-8-player-with-ad-support"></a>實作包含廣告支援的 Windows 或 Windows Phone 8 播放器
Microsoft 媒體平台：Player Framework for Windows 8 和 Windows Phone 8 包含範例應用程式集合，為您示範如何使用架構實作視訊播放器應用程式。 您可以從 [適用於Windows 8 和 Windows Phone 8 的 Player Framework](https://playerframework.codeplex.com)下載 Player Framework 和範例。

當您開啟 Microsoft.PlayerFramework.Xaml.Samples 方案時，您會在專案內看到許多資料夾。 Advertising 資料夾包含與建立具有廣告支援的視訊播放器相關的範例程式碼。 在 Advertising 資料夾內有許多 XAML/cs 檔案，每個檔案都會顯示如何以不同方式插入廣告。 下列清單分別說明：

* AdPodPage.xaml 示範如何顯示廣告組合。
* AdSchedulingPage.xaml 示範如何排程廣告。
* FreeWheelPage.xaml 示範如何使用 FreeWheel 外掛程式排程廣告。
* MastPage.xaml 示範如何使用 MAST 檔案排程廣告。
* ProgrammaticAdPage.xaml 示範如何以程式設計方式在視訊中排程廣告。
* ScheduleClipPage.xaml 示範如何在沒有 VAST 檔案的情況下排程廣告。
* VastLinearCompanionPage.xaml 示範如何插入線性和隨播廣告。
* VastNonLinearPage.xaml 示範如何插入非線性廣告。
* VmapPage.xaml 示範如何使用 VMAP 檔案指定廣告。

每個範例會使用 Player Framework 定義的 MediaPlayer 類別。 大部分的範例會使用加入各種廣告回應格式支援的外掛程式。 ProgrammaticAdPage 範例以程式設計方式與 MediaPlayer 執行個體互動。

### <a name="adpodpage-sample"></a>AdPodPage 範例
此範例會使用 AdSchedulerPlugin 來定義何時顯示廣告。 在此範例中，片中廣告排定在 5 秒鐘之後播放。 廣告組合 (依序顯示的廣告群組) 是在從廣告伺服器傳回的 VAST 檔案中指定。 VAST 檔案的 URI 是在 <RemoteAdSource> 元素中指定。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">

        <mmppf:MediaPlayer.Plugins>
            <ads:AdSchedulerPlugin>
                <ads:AdSchedulerPlugin.Advertisements>

                    <ads:MidrollAdvertisement Time="00:00:05">
                        <ads:MidrollAdvertisement.Source>
                            <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_adpod.xml" Type="vast"/>
                        </ads:MidrollAdvertisement.Source>
                    </ads:MidrollAdvertisement>

                </ads:AdSchedulerPlugin.Advertisements>
            </ads:AdSchedulerPlugin>
            <ads:AdHandlerPlugin/>
        </mmppf:MediaPlayer.Plugins>
    </mmppf:MediaPlayer>
```

如需 AdSchedulerPlugin 的詳細資訊，請參閱 [Windows 8 和 Windows Phone 8 上 Player Framework 中的廣告](http://playerframework.codeplex.com/wikipage?title=Advertising&referringTitle=Windows%208%20Player%20Documentation)

### <a name="adschedulingpage"></a>AdSchedulingPage
此範例也會使用 AdSchedulerPlugin。 它會排程三個廣告，片頭廣告、片中廣告和片尾廣告。 每個廣告的 VAST 的 URI 是在 <RemoteAdSource> 元素中指定。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:PrerollAdvertisement>
                                <ads:PrerollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PrerollAdvertisement.Source>
                            </ads:PrerollAdvertisement>

                            <ads:MidrollAdvertisement Time="00:00:15">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                            <ads:PostrollAdvertisement>
                                <ads:PostrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PostrollAdvertisement.Source>
                            </ads:PostrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="freewheelpage"></a>FreeWheelPage
此範例使用 FreeWheelPlugin，它會指定來源屬性，指定指向 SmartXML 檔案的 URI，該檔案會指定廣告內容以及廣告排程資訊。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:FreeWheelPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/freewheel.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="mastpage"></a>MastPage
這個範例會使用可讓您使用 MAST 檔案的 MastSchedulerPlugin。 來源屬性會指定 MAST 檔案的位置。
```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:MastSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/mast.xml" />
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="programmaticadpage"></a>ProgrammaticAdPage
此範例以程式設計方式與 MediaPlayer 互動。 ProgrammaticAdPage.xaml 檔案會具現化 MediaPlayer：

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4"/>
```

ProgrammaticAdPage.xaml.cs 檔案會建立 AdHandlerPlugin，新增 TimelineMarker 以指定廣告何時應該顯示，然後新增 MarkerReached 事件的處理常式，它會載入指定 VAST 檔案之 URI 的 RemoteAdSource，然後播放廣告。

```csharp
    public sealed partial class ProgrammaticAdPage : Microsoft.PlayerFramework.Samples.Common.LayoutAwarePage
        {
            AdHandlerPlugin adHandler;

            public ProgrammaticAdPage()
            {
                this.InitializeComponent();
                adHandler = new AdHandlerPlugin();
                player.Plugins.Add(new AdHandlerPlugin());
                player.Markers.Add(new TimelineMarker() { Time = TimeSpan.FromSeconds(5), Type = "myAd" });
                player.MarkerReached += pf_MarkerReached;
            }

            async void pf_MarkerReached(object sender, TimelineMarkerRoutedEventArgs e)
            {
                if (e.Marker.Type == "myAd")
                {
                    var adSource = new RemoteAdSource() { Type = VastAdPayloadHandler.AdType, Uri = new Uri("http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml") };
                    //var adSource = new AdSource() { Type = DocumentAdPayloadHandler.AdType, Payload = SampleAdDocument };
                    var progress = new Progress<AdStatus>();
                    try
                    {
                        await player.PlayAd(adSource, progress, CancellationToken.None);
                    }
                    catch { /* ignore */ }
                }
            }
```

### <a name="scheduleclippage"></a>ScheduleClipPage
此範例會使用 AdSchedulerPlugin，藉由指定包含廣告的 .wmv 檔案來排程片中廣告。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.cloudapp.net/html5/media/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:AdSource Type="clip">
                                        <ads:AdSource.Payload>
                                            <ads:ClipAdPayload MediaSource="http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv" MimeType="video/x-ms-wmv" />
                                        </ads:AdSource.Payload>
                                    </ads:AdSource>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearcompanionpage"></a>VastLinearCompanionPage
這個範例說明如何使用 AdSchedulerPlugin 來排程具有隨播廣告的片中線性廣告。 <RemoteAdSource> 元素會指定 VAST 檔案的位置。

```xml
    <mmppf:MediaPlayer Grid.Row="1"  x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_companions.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearnonlinearpage"></a>VastLinearNonLinearPage
此範例會使用 AdSchedulerPlugin 來排程線性和非線性廣告。 VAST 檔案位置是以 <RemoteAdSource> 元素指定。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_nonlinear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vmappage"></a>VMAPPage
此範例會使用 VmapSchedulerPlugin 排程使用 VMAP 檔案的廣告。 VMAP 檔案的 URI 是在 <VmapSchedulerPlugin> 元素的 Source 屬性中指定。

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:VmapSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/vmap.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

## <a name="implementing-an-ios-video-player-with-ad-support"></a>實作具有廣告支援的 iOS 視訊播放器
Microsoft 媒體平台：Player Framework for iOS 包含範例應用程式集合，為您示範如何使用架構實作視訊播放器應用程式。 您可以從 [Azure Media Player Framework](https://github.com/Azure/azure-media-player-framework)下載 Player Framework 和範例。 此 github 頁面具有連接至 Wiki 的連結，該連結包含 Player Framework 的其他資訊和播放器範例的簡介： [Azure Media Player Wiki](https://github.com/Azure/azure-media-player-framework/wiki/How-to-use-Azure-media-player-framework)。

### <a name="scheduling-ads-with-vmap"></a>使用 VMAP 排程廣告
下列範例示範如何使用 VMAP 檔案排程廣告。

```csharp
    // How to schedule an Ad using VMAP.
    //First download the VMAP manifest

    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"http://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVMAP.xml"]])
            {
                [self logFrameworkError];
            }
            else
            {
                // Schedule a list of ads using the downloaded VMAP manifest
                if (![framework scheduleVMAPWithManifest:manifest])
                {
                    [self logFrameworkError];
                }          
            }
```

### <a name="scheduling-ads-with-vast"></a>使用 VAST 排程廣告
下列範例示範如何排程晚期繫結 VAST 廣告。


```csharp
    //Example:3 How to schedule a late binding VAST ad.
    // set the start time for the ad
    adLinearTime.startTime = 13;
    adLinearTime.duration = 0;
    // Specify the URI of the VAST file
    NSString *vastAd1=@"http://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml";
    // Create an AdInfo object
     AdInfo *vastAdInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URL to VAST file
    vastAdInfo1.clipURL = [NSURL URLWithString:vastAd1];
    // set running time of ad
    vastAdInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    vastAdInfo1.mediaTime.clipBeginMediaTime = 0;
    vastAdInfo1.mediaTime.clipEndMediaTime = 10;
    vastAdInfo1.policy = @"policy for late binding VAST";
    // specify ad type
    vastAdInfo1.type = AdType_Midroll;
    vastAdInfo1.appendTo=-1;
    adIndex = 0;
    // schedule ad
    if (![framework scheduleClip:vastAdInfo1 atTime:adLinearTime forType:PlaylistEntryType_VAST andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

   下列範例示範如何排程早期繫結 VAST 廣告。

```csharp
    //Example:4 Schedule an early binding VAST ad
    //Download the VAST file
    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"http://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml"]])
    {
        [self logFrameworkError];
    }
    else
    {
        adLinearTime.startTime = 7;
        adLinearTime.duration = 0;

        // Create AdInfo instance
        AdInfo *vastAdInfo2 = [[[AdInfo alloc] init] autorelease];
        vastAdInfo2.mediaTime = [[[MediaTime alloc] init] autorelease];
        vastAdInfo2.policy = @"policy for early binding VAST";
        // specify ad type
        vastAdInfo2.type = AdType_Midroll;
        vastAdInfo2.appendTo=-1;
        // schedule ad
        if (![framework scheduleVASTClip:vastAdInfo2 withManifest:manifest atTime:adLinearTime andGetClipId:&adIndex])
        {
            [self logFrameworkError];
        }
    }
```

下列範例示範如何使用粗略剪裁編輯 (RCE) 插入廣告

```csharp
    //Example:1 How to use RCE.
    // specify manifest for ad content
    NSString *secondContent=@"http://wamsblureg001orig-hs.cloudapp.net/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // specify ad length
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
    // append ad content
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程廣告組合。

```csharp
    //Example:5 Schedule an ad Pod.
    // Set start time for ad
    adLinearTime.startTime = 23;
    adLinearTime.duration = 0;

    // Specify URL to content
    NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    // Create an AdInfo instance
    AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URI to ad content
    adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
    // Set ad running time
    adpodInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    adpodInfo1.mediaTime.clipBeginMediaTime = 0;
    adpodInfo1.mediaTime.clipEndMediaTime = 17;
    adpodInfo1.policy = @"policy for ad pod 1";
    // Set ad type
    adpodInfo1.type = AdType_Midroll;
    adpodInfo1.appendTo=-1;
    adIndex = 0;
    // Schedule ad
    if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程非固定片中廣告。 不論檢視者執行任何搜尋，非固定廣告只會播放一次。

```csharp
    //Example:6 Schedule a single non sticky mid roll Ad
    // specify URL to content
    NSString *oneTimeAd=@"http://wamsblureg001orig-hs.cloudapp.net/5389c0c5-340f-48d7-90bc-0aab664e5f02/Windows%208_%20You%20and%20Me%20Together-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // create an AdInfo instance
    AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
    // set URL of ad
    oneTimeInfo.clipURL = [NSURL URLWithString:oneTimeAd];
    oneTimeInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    oneTimeInfo.mediaTime.clipBeginMediaTime = 0;
    oneTimeInfo.mediaTime.clipEndMediaTime = -1;
    oneTimeInfo.policy = @"policy for one-time ad";
    // set ad start time
    adLinearTime.startTime = 43;
    adLinearTime.duration = 0;
    // set ad type
    oneTimeInfo.type = AdType_Midroll;
    // non sticky ad
    oneTimeInfo.deleteAfterPlayed = YES;
    // schedule ad
    if (![framework scheduleClip:oneTimeInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程固定片中廣告。 固定廣告會在每次到達視訊時間軸上的指定點時顯示。

```csharp
    //Example:7 Schedule a single sticky mid roll Ad
    NSString *stickyAd=@"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *stickyAdInfo = [[[AdInfo alloc] init] autorelease];
    // set URI to ad
    stickyAdInfo.clipURL = [NSURL URLWithString:stickyAd];
    stickyAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    stickyAdInfo.mediaTime.clipBeginMediaTime = 0;
    stickyAdInfo.mediaTime.clipEndMediaTime = 15;
    stickyAdInfo.policy = @"policy for sticky mid-roll ad";
    // set ad start time
    adLinearTime.startTime = 64;
    adLinearTime.duration = 0;
    // set ad type
    stickyAdInfo.type = AdType_Midroll;
    stickyAdInfo.deleteAfterPlayed = NO;
    // schedule ad
    if (![framework scheduleClip:stickyAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程片尾廣告。

```csharp
    //Example:8 Schedule Post Roll Ad
    NSString *postAdURLString=@"http://wamsblureg001orig-hs.cloudapp.net/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
    postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
    postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    postAdInfo.mediaTime.clipBeginMediaTime = 0;
    // set ad length
    postAdInfo.mediaTime.clipEndMediaTime = 45;
    postAdInfo.policy = @"policy for post-roll ad";
    // set ad type
    postAdInfo.type = AdType_Postroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程片頭廣告。

```csharp
    //Example:9 Schedule Pre Roll Ad
    NSString *adURLString = @"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 40; //You could play a portion of an Ad. Yeh!
    adInfo.mediaTime.clipEndMediaTime = 59;
    adInfo.policy = @"policy for pre-roll ad";
    adInfo.appendTo = -1;
    adInfo.type = AdType_Preroll;
    adLinearTime.duration = 0;
    // schedule ad
    if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

下列範例示範如何排程片中重疊廣告。

```csharp
    // Example10: Schedule a Mid Roll overlay Ad
    NSString *adURLString = @"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    //Create AdInfo instance
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 0;
    // specify ad length
    adInfo.mediaTime.clipEndMediaTime = 20;
    adInfo.policy = @"policy for mid-roll overlay ad";
    adInfo.appendTo = -1;
    // specify ad type
    adInfo.type = AdType_Midroll;
    // specify ad start time & duration
    adLinearTime.startTime = 300;
    adLinearTime.duration = 20;
    // schedule ad            if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[開發視訊播放程式應用程式](media-services-develop-video-players.md)

