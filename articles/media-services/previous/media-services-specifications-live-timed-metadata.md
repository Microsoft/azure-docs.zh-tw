---
title: Azure 媒體服務 - 針對即時串流中的計時中繼資料傳送訊號 | Microsoft Docs
description: 此規格描繪了「媒體服務」所支援的兩個模式，這兩個模式用於針對即時串流內的計時中繼資料傳送訊號。 這包括針對一般計時中繼資料訊號 (以及針對廣告拼接插入的 SCTE-35 訊號傳送) 的支援。
services: media-services
documentationcenter: ''
author: cenkdin
manager: cfowler
editor: johndeu
ms.assetid: 265b94b1-0fb8-493a-90ec-a4244f51ce85
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/17/2018
ms.author: johndeu;
ms.openlocfilehash: 2e736872dc3e471af7c5b3f758516910a02067fe
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786077"
---
# <a name="signaling-timed-metadata-in-live-streaming"></a>針對即時串流中的計時中繼資料傳送訊號


## <a name="1-introduction"></a>1 簡介 
為促進插入廣告或用戶端播放程式上的自訂事件，廣播者通常使用影片中內嵌的計時中繼資料。 為啟用這些案例，「媒體服務」提供針對計時中繼資料與媒體傳輸的支援，從即時串流頻道的內嵌點到用戶端應用程式。
此規格描繪了「媒體服務」所支援的兩個模式，這兩個模式用於針對即時串流訊號內的計時中繼資料。

1. 留意 [SCTE-67] 所述之建議實務的 [SCTE-35] 訊號傳送

2. 針對非 [SCTE-35] 之訊息的一般計時中繼資料訊號傳送模式

### <a name="12-conformance-notation"></a>1.2 一致性標記法
此文件中「必須」、「不得」、「必要」、「應」、「不應」、「應該」、「不應該」、「建議」、「可能」與「選擇性」需依 RFC 2119 所述來解釋

### <a name="13-terms-used"></a>1.3 使用條款

| 詞彙              | 定義                                                                                                                                                                                                                       |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 呈現時間 | 事件呈現給檢視者的時間。 此時間代表使用者會看到事件的媒體時間軸位置。 例如，SCTE-35 splice_info() 命令訊息的呈現時間是 splice_time()。 |
| 抵達時間      | 事件訊息抵達的時間。 此時間通常與事件的呈現時間截然不同，因為事件訊息是在事件的呈現時間之前傳送。                                     |
| 疏鬆資料軌      | 不連續且時間已與父代或控制播放軌同步的媒體播放軌。                                                                                                                                    |
| 來源            | Azure 媒體串流服務                                                                                                                                                                                                |
| 通道接收      | Azure 媒體即時串流服務                                                                                                                                                                                           |
| HLS               | Apple HTTP 即時串流通訊協定                                                                                                                                                                                               |
| DASH              | HTTP 上的動態自適性串流                                                                                                                                                                                             |
| Smooth            | Smooth Streaming 通訊協定                                                                                                                                                                                                        |
| MPEG2-TS          | MPEG 2 傳輸串流                                                                                                                                                                                                         |
| RTMP              | 即時多媒體通訊協定                                                                                                                                                                                                    |
| uimsbf            | 不帶正負號的整數，最顯著位元在前。                                                                                                                                                                                    |

-----------------------

## <a name="2-timed-metadata-ingest"></a>2 計時中繼資料內嵌
## <a name="21-rtmp-ingest"></a>2.1 RTMP 內嵌
RTMP 支援以內嵌在 RTMP 串流內之 AMF 提示訊息方式傳送的計時中繼資料訊號。 提示訊息可能在實際事件之前某個時間傳送，或者必須發生廣告拼接插入。 為支援此案例，拼接或區段的實際時間會在提示訊息內傳送。 如需詳細資訊，請參閱 [AMF0]。

下表說明「媒體服務」將內嵌之 AMF 訊息承載的格式。

AMF 訊息的名稱可用來區分相同類型的多個事件串流。  針對 [SCTE-35] 訊息，AMF 訊息的名稱必須是 “onAdCue”，如 [SCTE-67] 所建議。  必須忽略下面未列出的所有欄位，以便未來此設計的創新不會被抑制。

### <a name="signal-syntax"></a>訊號語法
針對 RTMP 簡單模式，「媒體服務」支援名為 "onAdCue" 且具有下列格式的單一 AMF 提示訊息：

### <a name="simple-mode"></a>簡單模式

| 欄位名稱 | 欄位類型 | 必要？ | 說明                                                                                                             |
|------------|------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| 提示        | 字串     | 必要 | 事件訊息。  應該是 "SpliceOut" 以指定簡單模式拼接。                                              |
| id         | 字串     | 必要 | 描述拼接或區段的唯一識別碼。 識別訊息的此執行個體                            |
| duration   | 數字     | 必要 | 拼接的持續時間。 單位是小數秒。                                                                |
| elapsed    | 數字     | 選用 | 當訊號重複以支援收聽時，此欄位應該是開始拼接後經過的呈現時間量。 單位是小數秒。 當使用簡單模式時，此值不應該超過拼接的原始時間長度。                                                  |
| 分析       | 數字     | 必要 | 應該是拼接的時間 (以呈現時間表示)。 單位是小數秒。                                     |

---------------------------

### <a name="scte-35-mode"></a>SCTE-35 模式

| 欄位名稱 | 欄位類型 | 必要？ | 說明                                                                                                             |
|------------|------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| 提示        | 字串     | 必要 | 事件訊息。  針對 [SCTE-35] 訊息，這必須經過 base64 (IETF RFC 4648) 二進位編碼 splice_info_section()，訊息才能傳送到 HLS、Smooth 與 Dash 用戶端，以符合 [SCTE-67] 規範。                                              |
| type       | 字串     | 必要 | 可識別訊息配置的 URN 或 URL；例如 "urn:example:signaling:1.0"。  針對 [SCTE-35] 訊息，這必須是 "urn:scte:scte35:2013a:bin"，訊息才能傳送到 HLS、Smooth 與 Dash 用戶端，以符合 [SCTE-67] 規範。  |
| id         | 字串     | 必要 | 描述拼接或區段的唯一識別碼。 識別訊息的此執行個體。  具有同等語意的訊息應該有相同的值。|
| duration   | 數字     | 必要 | 事件或廣告拼接區段的期間 (若已知)。 若未知，此值應該是 0。                                                                 |
| elapsed    | 數字     | 選用 | 當 [SCTE-35] 廣告訊號重複以收聽時，此欄位應該是開始拼接後經過的呈現時間量。 單位是小數秒。 在 [SCTE-35] 模式中，此值可能會超過原本指定的拼接或區段持續時間。                                                  |
| 分析       | 數字     | 必要 | 事件或拼接的呈現時間。  呈現時間與期間應該與類型 1 或 2 的串流存取點 (SAP) 一致，如 [ISO-14496-12] 附錄 I 中所定義。針對 HLS 輸出，時間與期間應該與區段界限一致。 呈現時間與相同事件串流中不同事件訊息的期間不能重疊。 單位是小數秒。

---------------------------

#### <a name="211-cancelation-and-updates"></a>2.1.1 取消與更新

您可以透過傳送具有相同呈現時間與 ID 的多個訊息來取消或更新訊息。 呈現時間與 ID 可唯一識別事件，而且針對符合預滾動限制式之特定呈現時間所接收的最後一個訊息是針對其採取動作的訊息。 更新的事件會取代先前接收的任何訊息。 預滾動限制式是四秒。 系統將會針對在呈現時間前最後四秒接收的訊息採取動作。

## <a name="22-fragmented-mp4-ingest-smooth-streaming"></a>2.2 分散的 MP4 內嵌 (Smooth Streaming)
如需即時串流內嵌的需求，請參閱 [LIVE-FMP4]。 以下各節提供有關計時呈現中繼資料內嵌的詳細資料。  計時呈現中繼資料會內嵌為疏鬆播放軌，疏鬆播放軌同時在即時伺服器中繼資料方塊 (請參閱 MS-SSTR) 與影片方塊 (‘moov’) 中定義。  每個疏鬆片段都是由影片片段方塊 (‘moof’) 與媒體資料方塊 (‘mdat’) 所組成，其中 ‘mdat’ 方塊是二進位訊息。

#### <a name="221-live-server-manifest-box"></a>2.2.1 即時伺服器資訊清單方塊
疏鬆播放軌必須使用 \<textstream\> 項目在即時伺服器資訊清單方塊中宣告，而且它必須已設定下列屬性：

| **屬性名稱** | **欄位類型** | **必要？** | **說明**                                                                                                                                                                                                                                                 |
|--------------------|----------------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| systemBitrate      | 數字         | 必要      | 必須是 “0”，指出具有未知可變位元速率的播放軌。                                                                                                                                                                                                 |
| parentTrackName    | 字串         | 必要      | 必須是父代播放軌的名稱，疏鬆播放軌時間碼的時幅與父系播放軌對應。 父系播放軌不能是疏鬆播放軌。                                                                                                                    |
| manifestOutput     | BOOLEAN        | 必要      | 必須是 “true”，指出疏鬆播放軌將內嵌在 Smooth 用戶端資訊清單中。                                                                                                                                                               |
| Subtype            | 字串         | 必要      | 必須是四個字元的代碼 “DATA”。                                                                                                                                                                                                                         |
| 配置             | 字串         | 必要      | 必須是可識別訊息配置的 URN 或 URL；例如 "urn:example:signaling:1.0"。 針對 [SCTE-35] 訊息，這必須是 "urn:scte:scte35:2013a:bin"，訊息才能傳送到 HLS、Smooth 與 Dash 用戶端，以符合 [SCTE-67] 規範。 |
| trackName          | 字串         | 必要      | 必須是疏鬆播放軌的名稱。trackName 可用來區分具有相同配置的不同事件串流。 每個唯一的事件串流都必須有唯一的播放軌名稱。                                                                           |
| timescale          | 數字         | 選用      | 必須是父代播放軌的時幅。                                                                                                                                                                                                                      |

-------------------------------------

### <a name="222-movie-box"></a>2.2.2 影片方塊

影片方塊 (‘moov’) 依循即時伺服器資訊清單方塊作為疏鬆播放軌之串流標題的一部分。

‘moov’ 方塊應該要包含 **TrackHeaderBox (‘tkhd’)** 方塊，如 [ISO-14496-12] 中所定義並具有下列限制式：

| **欄位名稱** | **欄位類型**          | **必要？** | **說明**                                                                                                |
|----------------|-------------------------|---------------|----------------------------------------------------------------------------------------------------------------|
| duration       | 64 位元不帶正負號的整數 | 必要      | 應該要是 0，因為播放軌方塊具有零個範例，而且播放軌中的範例總期間為 0。 |
-------------------------------------

‘moov’ 方塊應該要包含 **HandlerBox (‘hdlr’)**，如 [ISO-14496-12] 中所定義並具有下列限制：

| **欄位名稱** | **欄位類型**          | **必要？** | **說明**   |
|----------------|-------------------------|---------------|-------------------|
| handler_type   | 32 位元不帶正負號的整數 | 必要      | 應該要是 ‘meta’。 |
-------------------------------------

‘stsd’ 方塊應該包含 MetaDataSampleEntry 方塊，此方塊應該具有 [ISO-14496-12] 中所定義的程式碼名稱。  例如，針對 SCTE-35 訊息，程式碼名稱應該是 'scte'。

### <a name="223-movie-fragment-box-and-media-data-box"></a>2.2.3 影片片段方塊與媒體資料方塊

疏鬆播放軌片段由影片片段方塊 (‘moof’) 與媒體資料方塊 (‘mdat’) 所組成。

MovieFragmentBox (‘moof’) 方塊必須包含 **TrackFragmentExtendedHeaderBox (‘uuid’)** 方塊，如 [MS-SSTR] 中所定義並具有下列欄位：

| **欄位名稱**         | **欄位類型**          | **必要？** | **說明**                                                                               |
|------------------------|-------------------------|---------------|-----------------------------------------------------------------------------------------------|
| fragment_absolute_time | 64 位元不帶正負號的整數 | 必要      | 必須是事件的抵達時間。 此值與父代播放軌的訊息一致。   |
| fragment_duration      | 64 位元不帶正負號的整數 | 必要      | 必須是事件的持續時間。 持續時間可以是零，指出持續時間未知。 |

------------------------------------


MediaDataBox (‘mdat’) 方塊必須是下列格式：

| **欄位名稱**          | **欄位類型**                   | **必要？** | **說明**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-------------------------|----------------------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| version                 | 32 位元不帶正負號的整數 (uimsbf) | 必要      | 決定 ‘mdat’ 方塊內容的格式。 將會忽略無法辨識的版本。 目前唯一支援的版本是 1。                                                                                                                                                                                                                                                                                                                                                      |
| id                      | 32 位元不帶正負號的整數 (uimsbf) | 必要      | 識別訊息的此執行個體。 具有對等語意的訊息應該有相同的值；亦即，處理具有相同 id 的任一事件訊息就已足夠。                                                                                                                                                                                                                                                                                                            |
| presentation_time_delta | 32 位元不帶正負號的整數 (uimsbf) | 必要      | fragment_absolute_time (在 TrackFragmentExtendedHeaderBox 中所指定) 與 presentation_time_delta 的總和必須是事件的呈現時間。 呈現時間與期間應該與類型 1 或 2 的串流存取點 (SAP) 一致，如 [ISO-14496-12] 附錄 I 中所定義。針對 HLS 輸出，時間與期間應該與區段界限一致。 呈現時間與相同事件串流中不同事件訊息的期間不能重疊。 |
| Message                 | 位元組陣列                       | 必要      | 事件訊息。 針對 [SCTE-35] 訊息，訊息是二進位 splice_info_section()，雖然 [SCTE-67] 建議其他項目。 針對 [SCTE-35] 訊息，這必須是 "splice_info_section()"，訊息才能傳送到 HLS、Smooth 與 Dash 用戶端，以符合 [SCTE-67] 規範。 針對 [SCTE-35] 訊息，二進位 splice_info_section() 是 ‘mdat’ 方塊的承載，而且未進行 base64 編碼。                                                            |

------------------------------


### <a name="224-cancelation-and-updates"></a>2.2.4 取消與更新
您可以透過傳送具有相同呈現時間與 ID 的多個訊息來取消或更新訊息。  呈現時間與 ID uniquely 可唯一識別事件。 針對符合預滾動限制式之特定呈現時間所接收的最後一個訊息是針對其採取動作的訊息。 更新的訊息會取代先前接收的任何訊息。  預滾動限制式是四秒。 系統將會針對在呈現時間前最後四秒接收的訊息採取動作。 


## <a name="3-timed-metadata-delivery"></a>3 計時中繼資料傳遞

事件串流資料對「媒體服務」而言是不透明的。 「媒體服務」只會在內嵌端點與用戶端端點之間傳送三小部分的資訊。 下列屬性會傳遞到用戶端，而且符合 [SCTE-67] 和/或用戶端的串流通訊協定規範。

1.  配置 – 可識別訊息配置的 URN 或 URL。

2.  呈現時間 – 媒體時間軸上之事件的呈現時間。

3.  持續時間 – 事件的持續時間。

4.  ID – 選擇性的事件唯一識別碼。

5.  訊息 – 事件資料。


## <a name="31-smooth-streaming-delivery"></a>3.1 Smooth Streaming 傳遞

請參閱 [MS-SSTR] 中的疏鬆資料軌處理詳細資料。

#### <a name="smooth-client-manifest-example"></a>Smooth 用戶端資訊清單範例
~~~ xml
<?xml version=”1.0” encoding=”utf-8”?>
<SmoothStreamingMedia MajorVersion=”2” MinorVersion=”0” TimeScale=”10000000” IsLive=”true” Duration=”0”
  LookAheadFragmentCount=”2” DVRWindowLength=”6000000000”>

  <StreamIndex Type=”video” Name=”video” Subtype=”” Chunks=”0” TimeScale=”10000000”
    Url=”QualityLevels({bitrate})/Fragments(video={start time})”>
    <QualityLevel Index=”0” Bitrate=”230000”
      CodecPrivateData=”250000010FC3460B50878A0B5821FF878780490800704704DC0000010E5A67F840” FourCC=”WVC1”
      MaxWidth=”364” MaxHeight=”272”/>
    <QualityLevel Index=”1” Bitrate=”305000”
      CodecPrivateData=”250000010FC3480B50878A0B5821FF87878049080894E4A7640000010E5A67F840” FourCC=”WVC1”
      MaxWidth=”364” MaxHeight=”272”/>
    <c t=”0” d=”20000000” r=”300” />
  </StreamIndex>
  <StreamIndex Type=”audio” Name=”audio” Subtype=”” Chunks=”0” TimeScale=”10000000”
    Url=”QualityLevels({bitrate})/Fragments(audio={start time})”>
    <QualityLevel Index=”0” Bitrate=”96000” CodecPrivateData=”1000030000000000000000000000E00042C0”
      FourCC=”WMAP” AudioTag=”354” Channels=”2” SamplingRate=”44100” BitsPerSample=”16” PacketSize=”4459”/>
    <c t=”0” d=”20000000” r=”300” />
  </StreamIndex>
  <StreamIndex Type=”text” Name=”scte35-event-stream” Subtype=”DATA” Chunks=”0” TimeScale=”10000000”
    ParentStreamIndex=”video” ManifestOutput=”true” 
    Url=”QualityLevels({bitrate})/Fragments(captions={start time})”>
    <QualityLevel Index=”0” Bitrate=”0” CodecPrivateData=”” FourCC=””>
      <CustomAttributes>
        <Attribute Name=”Scheme” Value=”urn:scte:scte35:2013a:bin”/>
      </CustomAttributes>
    </QualityLevel>
    <!-- <f> contains base-64 encoded splice_info_section() message 
    <c t=”600000000” d=”300000000”>
<f>PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTUwMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</f>
    </c>
    <c t=”1200000000” d=”400000000”>      <f>PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTYyMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</f>
    </c>
  </StreamIndex>
</SmoothStreamingMedia> 
~~~ 

## <a name="32--apple-hls-delivery"></a>3.2  Apple HLS 傳遞
Apple HTTP 即時串流 (HLS) 的計時中繼資料可能是內嵌在自訂 M3U 標籤內的區段播放清單中。  應用程式層可以剖析 M3U 播放清單並處理 M3U 標籤。 「Azure 媒體服務」會將計時中繼資料內嵌在 [HLS] 中所定義的 EXT-X-CUE 標籤中。  EXT-X-CUE 標籤目前由 DynaMux 用於類型為 ADI3 的訊息。  若要支援 SCTE-35 與非 SCTE-35 訊息，請將 EXT-X-CUE 標籤的屬性設定為如下所定義的樣子：

| **屬性名稱** | **類型**                      | **必要？**                             | **說明**                                                                                                                                                                                                                                                                      |
|--------------------|-------------------------------|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 提示                | 加上括號的字串                 | 必要                                  | 已編碼為 base64 字串的訊息，如 [IETF RFC 4648](http://tools.ietf.org/html/rfc4648) 中所述。 針對 [SCTE-35] 訊息，這是經過 base64 編碼的 splice_info_section()。                                                                                                |
| 類型               | 加上括號的字串                 | 必要                                  | 可識別訊息配置的 URN 或 URL；例如 “urn:example:signaling:1.0”。 針對 [SCTE-35] 訊息，類型接受特殊值 “scte35”。                                                                                                                                |
| ID                 | 加上括號的字串                 | 必要                                  | 事件的唯一識別碼。 如果內嵌訊息時未指定 ID，「Azure 媒體服務」將會產生一個唯一 ID。                                                                                                                                          |
| 持續時間           | 十進位浮點數數字 | 必要                                  | 事件的持續時間。 若未知，此值應該是 0。 單位是小數秒。                                                                                                                                                                                           |
| ELAPSED            | 十進位浮點數數字 | 選擇性，但對滑動視窗而言是必要的 | 當訊號重複以支援滑動呈現視窗時，此欄位必須是事件開始後經過的呈現時間量。 單位是小數秒。 此值可能會超過原本指定的拼接或區段持續時間。 |
| TIME               | 十進位浮點數數字 | 必要                                  | 事件的呈現時間。 單位是小數秒。                                                                                                                                                                                                                    |


HLS 播放程式應用程式層將會使用 TYPE 來識別訊息格式、將訊息解碼、套用必要時間轉換，以及處理事件。  根據事件時間戳記，事件時間在父代播放軌的區段播放清單中是同步的。  它們是插入在最接近的區段 (#EXTINF tag) 前面。

#### <a name="hls-segment-playlist-example"></a>HLS 區段播放清單範例
~~~
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-ALLOW-CACHE:NO
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:6
#EXT-X-PROGRAM-DATE-TIME:1970-01-01T00:00:00.000+00:00
#EXTINF:6.000000,no-desc
Fragments(video=0,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=60000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
#EXT-X-CUE: ID=”metadata-12.000000”,TYPE=”urn:example:signaling:1.0”,TIME=”12.000000”, DURATION=”18.000000”,CUE=”HrwOi8vYmWVkaWEvhhaWFRlRDa=”
Fragments(video=120000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=180000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=240000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=300000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=360000000,format=m3u8-aapl)
#EXT-X-CUE: ID=”metadata-42.000000”,TYPE=”urn:example:signaling:1.0”,TIME=”42.000000”, DURATION=”60.000000”,CUE=”PD94bWwgdm0iMS4wIiBlbmNvpD4=”
#EXTINF:6.000000,no-desc
Fragments(video=420000000,format=m3u8-aapl)
#EXTINF:6.000000,no-desc
Fragments(video=480000000,format=m3u8-aapl)
…
~~~

#### <a name="hls-message-handling"></a>HLS 訊息處理

事件是在每個視訊語音訊播放軌的區段播放清單中收到訊號通知。EXT-X-CUE 標籤的位置必須一律位於第一個 HLS 區段之前 (針對拼接外或區段開始) 或位於最後一個 HLS 區段之後 (針對拼接內或區段結束)，這是其 TIME 與 DURATION 屬性所指的目標，此為 [HLS] 所要求。

當滑動呈現視窗啟用時，EXT-X-CUE 標籤必須重複足夠的次數，讓拼接或區段在區段播放清單中完整描述，而且必須使用 ELAPSED 屬性來指出拼接或屬性啟用的時間量，如 [HLS] 所要求。

當滑動呈現視窗啟用時，若它們所指的媒體時間已從滑動呈現視窗滾出，標籤會從區段播放清單移除。

## <a name="33--dash-delivery"></a>3.3  DASH 傳遞
[DASH] 提供三種傳送訊號給事件的方式：

1.  在 MPD 中傳送訊號給事件
2.  事件在 Representation (使用事件訊息方塊 (‘emsg’) 中接獲以頻內方式傳送的訊號通知
3.  1 與 2 兩者的組合

事件在 MPD 中接獲訊號通知對 VOD 串流而言非常實用，因為用戶端在 MPD 下載之後可立即存取所有事件。 頻內解決方案對即時串流而言非常實用，因為用戶端不需要重新下載 MPD。 針對以時間為基礎的區段，用戶端會透過新增目前區段的持續時間與時間戳記來判斷下一個區段的 URL。 若要求失敗，用戶端會假設不連續並下載 MPD，但除非在未下載 MPD 的情況下繼續串流。

「Azure 媒體服務」會使用事件訊息方塊在 MPD 與頻內訊號處理中進行兩個訊號處理。

#### <a name="mpd-signaling"></a>MPD 訊號處理

系統會使用 EventStream 元素在 MPD 中傳送訊號通知事件，該元素出現在 Period 元素內。

EventStream 元素具有下列屬性：

| **屬性名稱** | **類型**                | **必要？** | **說明**                                                                                                                                                                                                                                                                                   |
|--------------------|-------------------------|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| scheme_id_uri      | 字串                  | 必要      | 識別訊息的配置。 配置是設定為即時伺服器資訊清單方塊中 Scheme 屬性的值。 值應該是可識別訊息配置的 URN 或 URL；例如 “urn:example:signaling:1.0”。                                                                |
| value              | 字串                  | 選用      | 由配置擁有者用來自訂訊息語意的額外字串值。 若要區分具有相同配置的多個事件串流，值必須設定為事件串流的名稱 (適用於 Smooth 內嵌的 trackName，或適用於 RTMP 內嵌的 AMF 訊息名稱)。 |
| 時幅          | 32 位元不帶正負號的整數 | 必要      | ‘emsg’ 方塊內時間與持續時間的時幅 (以每秒滴答數表示)。                                                                                                                                                                                                       |


EventStream 元素包含零或多個 Event 元素，而且它們具有下列屬性：

| **屬性名稱**  | **類型**                | **必要？** | **說明**                                                                                                                                                                                                             |
|---------------------|-------------------------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| presentation_time   | 64 位元不帶正負號的整數 | 選用      | 必須是相對於 Period 開始時之事件的媒體呈現時間。 呈現時間與持續時間應該與類型 1 或 2 之串流存取點 (SAP) 一致，如 [ISO-14496-12] 附錄 I 中所定義。 |
| duration            | 32 位元不帶正負號的整數 | 選用      | 事件的持續時間。 若持續時間未知，則必須略過此項目。                                                                                                                                                 |
| id                  | 32 位元不帶正負號的整數 | 選用      | 識別訊息的此執行個體。 具有同等語意的訊息應該有相同的值。 如果內嵌訊息時未指定 ID，「Azure 媒體服務」將會產生一個唯一 ID。             |
| Event 元素值 | 字串                  | 必要      | 以 base64 字串表示的事件訊息，如 [IETF RFC 4648](http://tools.ietf.org/html/rfc4648) 中所述。                                                                                                                   |

#### <a name="xml-syntax-and-example-for-dash-manifest-mpd-signaling"></a>DASH 資訊清單 (MPD) 訊號的 XML 語法與範例

~~~ xml
<!-- XML Syntax -->
<xs:complexType name=”EventStreamType”>
  <xs:sequence>
    <xs:element name=”Event” type=”EventType” minOccurs=”0” maxOccurs=”unbounded”/>
    <xs:any namespace=”##other” processContents=”lax” minOccurs=”0” maxOccurs=”unbounded”/>
  </xs:sequence>
  <xs:attribute ref=”xlink:href”/>
  <xs:attribute ref=”xlink:actuate” default=”onRequest”/>
  <xs:attribute name=”schemeIdUri” type=”xs:anyURI” use=”required”/>
  <xs:attribute name=”value” type=”xs:string”/>
  <xs:attribute name=”timescale” type=”xs:unsignedInt”/>
</xs:complexType>
<!-- Event -->
<xs:complexType name=”EventType”>
  <xs:sequence>
    <xs:any namespace=”##other” processContents=”lax” minOccurs=”0” maxOccurs=”unbounded”/>
  </xs:sequence>
  <xs:attribute name=”presentationTime” type=”xs:unsignedLong” default=”0”/>
  <xs:attribute name=”duration” type=”xs:unsignedLong”/>
  <xs:attribute name=”id” type=”xs:unsignedInt”/>
  <xs:anyAttribute namespace=”##other” processContents=”lax”/>
</xs:complexType><?xml version=”1.0” encoding=”utf-8”?>


<!-- Example Section in MPD -->

<EventStream schemeIdUri=”urn:example:signaling:1.0” timescale=”1000” value=”player-statistics”>
  <Event presentationTime=”0” duration=”10000” id=”0”> PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTUwMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</Event>
  <Event presentationTime=”20000” duration=”10000” id=”1”> PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTYyMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</Event>
</EventStream>
~~~

>[!NOTE]
>請注意，presentationTime 是事件的呈現時間，而不是訊息的抵達時間。
>

### <a name="431-in-band-event-message-box-signaling"></a>4.3.1 頻內事件訊息方塊訊號
頻內事件串流要求 MPD 在「自適性集合」層必須有 InbandEventStream 元素。  此元素有必要 schemeIdUri 屬性與選擇性 timescale 屬性，它們也出現在事件訊息方塊中 (‘emsg’)。  具有 MPD 中未定義之配置識別碼的事件訊息方塊不應該存在。 若 DASH 用戶端偵測到具有 MPD 中未定義之配置的事件訊息方塊，用戶端預期會忽略它。
事件訊息方塊 (‘emsg’) 針對與媒體呈現時間相關的一般事件提供訊號處理。 若存在，任何 ‘emsg’ 方塊都應該放在任何 ‘moof’ 方塊之前。

### <a name="dash-event-message-box-emsg"></a>DASH 事件訊息方塊 ‘emsg’
~~~
Box Type: `emsg’
Container: Segment
Mandatory: No
Quantity: Zero or more
~~~

~~~ c
aligned(8) class DASHEventMessageBox extends FullBox(‘emsg’, version = 0, flags =0) 
{
    string scheme_id_uri;
    string value;
    unsigned int(32) timescale;
    unsigned int(32) presentation_time_delta;
    unsigned int(32) event_duration;
    unsigned int(32) id;
    unsigned int(8) message_data[];
}
~~~

DASHEventMessageBox 的欄位定義如下：

| **欄位名稱**          | **欄位類型**          | **必要？** | **說明**                                                                                                                                                                                                                                                                                                                                                    |
|-------------------------|-------------------------|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| scheme_id_uri           | 字串                  | 必要      | 識別訊息的配置。 配置是設定為即時伺服器資訊清單方塊中 Scheme 屬性的值。 值應該是可識別訊息配置的 URN 或 URL；例如 “urn:example:signaling:1.0”。 針對 [SCTE-35] 訊息，這會接受特殊值 “urn:scte:scte35:2013a:bin”，雖然 [SCTE-67] 建議其他項目。 |
| 值                   | 字串                  | 必要      | 由配置擁有者用來自訂訊息語意的額外字串值。 若要區分具有相同配置的多個事件串流，值必須設定為事件串流的名稱 (適用於 Smooth 內嵌的 trackName，或適用於 RTMP 內嵌的 AMF 訊息名稱)。                                                                  |
| 時幅               | 32 位元不帶正負號的整數 | 必要      | ‘emsg’ 方塊內時間與持續時間的時幅 (以每秒滴答數表示)。                                                                                                                                                                                                                                                                        |
| presentation_time_delta | 32 位元不帶正負號的整數 | 必要      | 事件呈現時間與此區段中最早的呈現時間之間的媒體呈現時間差異。 呈現時間與持續時間應該與類型 1 或 2 之串流存取點 (SAP) 一致，如 [ISO-14496-12] 附錄 I 中所定義。                                                                                            |
| event_duration          | 32 位元不帶正負號的整數 | 必要      | 事件的持續時間，或指定 0xFFFFFFFF 以指出未知的持續時間。                                                                                                                                                                                                                                                                                          |
| id                      | 32 位元不帶正負號的整數 | 必要      | 識別訊息的此執行個體。 具有同等語意的訊息應該有相同的值。 如果內嵌訊息時未指定 ID，「Azure 媒體服務」將會產生一個唯一 ID。                                                                                                                                                    |
| Message_data            | 位元組陣列              | 必要      | 事件訊息。 針對 [SCTE-35] 訊息，訊息資料是二進位 splice_info_section()，雖然 [SCTE-67] 建議其他項目。                                                                                                                                                                                                                                 |

### <a name="332-dash-message-handling"></a>3.3.2 DASH 訊息處理

事件會以頻內方式接收傳送的訊號通知，針對視訊語音訊播放軌，都是在 ‘emsg’ 方塊內。  針對 presentation_time_delta 為 15 秒或低於 15 秒的所有區段要求，都會發生訊號處理。 當滑動呈現視窗啟用時，若事件訊息的時間與持續時間總和小於資訊清單中媒體資料的時間，事件訊息會從 MPD 移除。  換句話說，當其所指之媒體時間已滾出滑動呈現視窗之外，就會從資訊清單移除事件訊息。

## <a name="4-scte-35-ingest"></a>4 SCTE-35 內嵌

[SCTE-35] 訊息是使用配置 **“urn:scte:scte35:2013a:bin”** (針對 Smooth 內嵌) 與類型 **“scte35”** (針對 RTMP 內嵌) 以二進位格式內嵌。 為促進 [SCTE-35] 計時的轉換 (以 MPEG-2 傳輸串流呈現時間戳記 (PTS) 為基礎)，PTS (pts_time + pts_adjustment of the splice_time()) 與媒體時間軸之間的對應是由事件呈現時間 (適用於 Smooth 內嵌的 fragment_absolute_time 欄位，或適用於 RTMP 內嵌的時間欄位) 所提供。 對應是必要的，因為 33 位元 PTS 值大約每隔 26.5 小時就會滾動一次。

Smooth Streaming 內嵌要求媒體資料方塊 (‘mdat’) 必須包含 [SCTE-35] 表 8-1 中所定義的 **splice_info_section()**。 針對 RTMP 內嵌，AMF 訊息的 cue 屬性是設定為 base64encoded **splice_info_section()**。 當訊息具有上述格式時，它們會被傳送到 HLS、Smooth 與 Dash 用戶端，以符合 [SCTE-67] 的規範。


## <a name="5-references"></a>5 參考

**[SCTE-35]** ANSI/SCTE 35 2013a – 適用於纜線的數位程式插入提示，2013a

**[SCTE-67]** ANSI/SCTE 67 2014 –SCTE 35 建議實務：適用於纜線的數位程式插入提示訊息

**[DASH]** ISO/IEC 23009-1 2014 – 資訊科技 – HTTP (DASH) 上的動態自適性串流 – 第 1 部分：媒體呈現描述與區段格式，第 2 版

**[HLS]** [「HTTP 即時串流」，draft-pantos-http-live-streaming-14，2014 年 10 月 14 日，](http://tools.ietf.org/html/draft-pantos-http-live-streaming-14)

**[MS-SSTR]** [「Microsoft Smooth Streaming 通訊協定」，2014 年 5 月 15 日](http://download.microsoft.com/download/9/5/E/95EF66AF-9026-4BB0-A41D-A4F81802D92C/%5bMS-SSTR%5d.pdf)

**[AMF0]** [「動作訊息格式 AMF0」](http://download.macromedia.com/pub/labs/amf/amf0_spec_121207.pdf)

**[LIVE-FMP4]** [Azure 媒體服務分散式 MP4 即時內嵌規格](https://docs.microsoft.com/azure/media-services/media-services-fmp4-live-ingest-overview)

**[ISO-14496-12]** ISO/IEC 14496-12：第 12 部分 ISO 基底媒體檔案格式，2012-07-15 第 4 版。

**[RTMP]** [「Adobe 的即時訊息處理通訊協定」，2012 年 12 月 21 日](https://www.adobe.com/devnet/rtmp.html) 

------------------------------------------

## <a name="next-steps"></a>後續步驟
檢視媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
