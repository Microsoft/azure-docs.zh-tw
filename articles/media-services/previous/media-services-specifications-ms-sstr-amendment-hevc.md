---
title: Azure 媒體服務 - HEVC 的 Smooth Streaming 通訊協定 (MS-SSTR) 增修條款 | Microsoft Docs
description: 此規格說明在 Azure 媒體服務中採用 HEVC 的分散式 MP4 即時串流所適用的通訊協定和格式。 這是 Smooth Streaming 通訊協定文件 (MS-SSTR) 的增修條款，意在納入 HEVC 內嵌和串流的支援。 本文僅指出為了傳遞 HEVC 而需要的變更，但「(無變更)」是表示複製文字僅供說明之用，則屬例外。
services: media-services
documentationcenter: ''
author: cenkdin
manager: cfowler
editor: ''
ms.assetid: f27d85de-2cb8-4269-8eed-2efb566ca2c6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/07/2018
ms.author: johndeu;
ms.openlocfilehash: 78ec0e3ee4304e820bf64afa26440380887630a1
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786057"
---
# <a name="smooth-streaming-protocol-ms-sstr-amendment-for-hevc"></a>HEVC 的 Smooth Streaming 通訊協定 (MS-SSTR) 增修條款

## <a name="1-introduction"></a>1 簡介 

本文提供可啟用 HEVC 編碼視訊之 Smooth Streaming 的 Smooth Streaming 通訊協定規格 [MS-SSTR] 所適用的詳細修訂文件。 在此規格中，我們僅概述為了傳遞 HEVC 視訊轉碼器所需的變更。 本文依循與 [MS-SSTR] 規格相同的編號結構描述。 整份文件中顯示的空白標題，是為了讓讀者了解他們在 [MS SSTR] 規格中的位置。  「(無變更)」是表示複製文字僅供說明之用。

本文在 Smooth Streaming 資訊清單中提供 HEVC 視訊轉碼器訊號的技術實作需求，而標準參考已更新為參考包含 HEVC、HEVC 一般加密的現行 MPEG 標準，且 ISO Base Media 檔案格式已更新為與最新規格一致。 

參考的 Smooth Streaming 通訊協定規格 [MS-SSTR] 說明用來以下列方式傳遞即時和點播數位媒體 (例如音訊及視訊) 的電傳格式：從編碼器到 Web 伺服器、從伺服器到另一部伺服器，以及從伺服器到 HTTP 用戶端。
使用透過 HTTP 的 MPEG-4 ([[MPEG4-RA])](http://go.microsoft.com/fwlink/?LinkId=327787) 型資料結構傳遞，可幾近即時地在壓縮媒體內容的不同品質水準間進行切換。 其結果是，HTTP 用戶端使用者可享有穩定的播放體驗，即使用戶端電腦或裝置的網路和視訊轉譯條件有所變動，仍不受影響。

## <a name="11-glossary"></a>1.1 詞彙 

以下是 *[MS-GLOS]* 中定義的字詞：

>   **全域唯一識別碼 (GUID) 通用唯一識別碼 (UUID)**

以下是本文專用的字詞：

>  **編寫階段：** 樣本存留在用戶端上的時間，如 [[ISO/IEC-14496-12]](http://go.microsoft.com/fwlink/?LinkId=183695) 所定義。

>   **CENC**：一般加密，如 [ISO/IEC 23001-7] 第二版所定義。

>   **解碼階段：** 樣本必須在用戶端上解碼的時間，如 [[ISO/IEChttp://go.microsoft.com/fwlink/?LinkId=18369514496-12] 所定義。](http://go.microsoft.com/fwlink/?LinkId=183695)

**片段：** 可單獨下載、包含一或多個**樣本**的**媒體**單元。

>   **HEVC：** 高效率視訊編碼時，如 [ISO/IEC 23008-2] 所定義

>   **資訊清單：** 可讓用戶端發出**媒體**要求之**顯示**的相關中繼資料。 **媒體：** 用戶端用來播放**顯示**的壓縮音訊、視訊和文字資料。 **媒體格式：** 一個妥善定義的格式，可將音訊或視訊顯示為壓縮的**樣本**。

>   **顯示：** 播放單一影片所需的所有**資料流**和相關中繼資料的集合。 **要求：** 從用戶端傳送至伺服器的 HTTP 訊息，如 [[RFC2616]](http://go.microsoft.com/fwlink/?LinkId=90372) 所定義。 **回應：** 從伺服器傳送至用戶端的 HTTP 訊息，如 [[RFC2616]](http://go.microsoft.com/fwlink/?LinkId=90372) 所定義。

>   **樣本：** 儲存及處理**媒體**的最小基本單位 (例如框架)。

>   **「可能」、「應該」、「必須」、「不應」、「不可」：** 這些字詞 (全都會加上引號) 的用法如 [[RFC2119]](http://go.microsoft.com/fwlink/?LinkId=90317) 所說明。 選擇性行為的所有陳述均使用「可能」、「應該」或「不應」。

## <a name="12-references"></a>1.2 參考 
-----------

>   對 Microsoft Open Specifications 文件的參考不包含發行年份，因為連結會連至經常更新的最新版文件。 對其他文件的參考則包含發行年份 (如果有的話)。

 ### <a name="121-normative-references"></a>1.2.1 標準參考 

>  [MS-SSTR] Smooth Streaming 通訊協定 *v20140502* [http://download.microsoft.com/download/9/5/E/95EF66AF-9026-4BB0-A41D-A4F81802D92C/[MS-SSTR].pdf](http://download.microsoft.com/download/9/5/E/95EF66AF-9026-4BB0-A41D-A4F81802D92C/%5bMS-SSTR%5d.pdf)

>   [ISO/IEC 14496-12] 國際標準化組織，「資訊技術 -- 音訊-視覺化物件的編碼 -- 第 12 部分：ISO Base Media 檔案格式」，ISO/IEC 14496-12:2014，第4 版，Plus Corrigendum 1，修訂文件 1 和 2。
>   <http://standards.iso.org/ittf/PubliclyAvailableStandards/c061988_ISO_IEC_14496-12_2012.zip>

>   [ISO/IEC 14496-15] 國際標準化組織，「資訊技術 -- 音訊-視覺化物件的編碼 -- 第 15 部分：傳輸 ISO Base Media 檔案格式的 NAL 單元結構化視訊」，ISO 14496-15:2015，第 3 版。
>   <http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=65216>

>   [ISO/IEC 23008-2] 資訊技術 -- 異質環境中的高效率編碼和媒體傳遞 -- 第 2 部分：高效率視訊編碼：2013 或最新版本   <http://standards.iso.org/ittf/PubliclyAvailableStandards/c035424_ISO_IEC_23008-2_2013.zip>

>   [ISO/IEC 23001-7] 資訊技術 — MPEG 系統技術 — 第 7 部分：ISO Base Media 檔案格式檔案中的一般加密，CENC Edition 2:2015 <http://www.iso.org/iso/catalogue_detail.htm?csnumber=65271>

>   [RFC-6381] IETF RFC-6381，「貯體媒體類型的轉碼器和設定檔參數」<http://tools.ietf.org/html/rfc6381>

>   [MPEG4-RA] MP4 登錄授權單位，"MP4REG"，[http://www.mp4ra.org   ](http://go.microsoft.com/fwlink/?LinkId=327787)

>   [RFC2119] Bradner, S.，「在 RFC 中用來指出需求層級的關鍵字」，BCP 14，RFC 2119，1997 年 3 月，[http://www.rfc-editor.org/rfc/rfc2119.txt   ](http://go.microsoft.com/fwlink/?LinkId=90317)

### <a name="122-informative-references"></a>1.2.2 資訊參考 

>   [MS-GLOS] Microsoft Corporation，「*Windows 通訊協定主要詞彙*」。

>   [RFC3548] Josefsson, S., Ed.,，「Base16、Base32 和 Base64 資料編碼」，RFC 3548，2003 年 7 月，[http://www.ietf.org/rfc/rfc3548.txt   ](http://go.microsoft.com/fwlink/?LinkId=90432)

>   [RFC5234] Crocker, D., Ed., and Overell, P., ，「語法規格的增強型 BNF：ABNF」，STD 68，RFC 5234，2008 年 1 月，[http://www.rfc-editor.org/rfc/rfc5234.txt   ](http://go.microsoft.com/fwlink/?LinkId=123096)


## <a name="13-overview"></a>1.3 概觀 
---------

>   以下僅指出傳遞 HEVC 所需的 Smooth Streaming 規格變更。 未變更的小節會列出標題，以依照參考的 Smooth Streaming 規格 [MS-SSTR] 保有相對位置。

## <a name="14-relationship-to-other-protocols"></a>1.4 與通訊協定的關聯性 
--------------------------------

## <a name="15-prerequisitespreconditions"></a>1.5 先決條件/前置條件 
----------------------------

## <a name="16-applicability-statement"></a>1.6 適用性陳述 
------------------------

## <a name="17-versioning-and-capability-negotiation"></a>1.7 版本設定和功能交涉 
--------------------------------------

## <a name="18-vendor-extensible-fields"></a>1.8 廠商可延伸欄位 
-------------------------

>   「應該」使用下列方法來識別使用 HEVC 視訊格式的資料流：

>   * **媒體格式的自訂描述性代碼：** 這項功能由 **FourCC** 欄位提供，如 *2.2.2.5* 小節所指定。
>   實作者可將延伸模組註冊至 MPEG4-RA，以確保延伸模組不會衝突，如 [[ISO/IEC-14496-12]](http://go.microsoft.com/fwlink/?LinkId=183695) 所指定

## <a name="19-standards-assignments"></a>1.9 標準指派 
----------------------

# <a name="2-messages"></a>2 訊息 

## <a name="21-transport"></a>2.1 傳輸 
----------

## <a name="22-message-syntax"></a>2.2 訊息語法 
---------------

### <a name="221-manifest-request"></a>2.2.1 資訊清單要求 

### <a name="222-manifest-response"></a>2.2.2 資訊清單回應 

#### <a name="2221-smoothstreamingmedia"></a>2.2.2.1 SmoothStreamingMedia 

>   **MinorVersion (變數)：** 資訊清單回應訊息的次要版本。 「必須」設為 2。 (無變更)

>   **TimeScale (變數)：**「持續時間」屬性的時間間隔，指定為一秒內的增量數目。 預設值為
>   10000000. (無變更)

>   建議的值是 90000，用以表示包含分段 framerate 視訊的視訊畫面格和片段的確切持續時間 (例如，30/1.001 Hz)。

#### <a name="2222-protectionelement"></a>2.2.2.2 ProtectionElement 

在視訊或音訊資料流已套用一般加密 (CENC) 時，ProtectionElement「應該」存在。 HEVC 加密資料流「應該」符合一般加密第 2 版 [ISO/IEC 23001-7]。 只有 VCL NAL 單元中的配量資料才「應該」加密。

#### <a name="2223-streamelement"></a>2.2.2.3 StreamElement 

>   **StreamTimeScale (變數)：** 此資料流中的持續時間值和時間值的時間間隔，指定為一秒內的增量數目。 HEVC 資料流的建議值為 90000。 建議對音訊資料流使用符合波形取樣頻率的值 (例如，48000 或 44100)。

##### <a name="22231-streamprotectionelement"></a>2.2.2.3.1 StreamProtectionElement

#### <a name="2224-urlpattern"></a>2.2.2.4 UrlPattern 

#### <a name="225-trackelement"></a>2.2.5 TrackElement 

>   **FourCC (變數)：** 識別用於各個樣本之媒體格式的四字元代碼。 下列的值範圍保留為下列語意：

>  * "hev1"：此音軌的視訊樣本會使用 HEVC 視訊，採用 [ISO/IEC-14496-15] 所指定的 ‘hev1’ 樣本說明格式。

>   **CodecPrivateData (變數)：** 這項資料會指定媒體格式的特定參數和音軌中所有樣本通用的參數，以十六進位編碼位元組的字串表示。 位元組序列的格式和語意會隨著 **FourCC** 欄位的值而不同，如下所示：

>   * 當 TrackElement 說明 HEVC 視訊時，**FourCC** 欄位「應該」等於 **"hev1"**。

>   **CodecPrivateData** 欄位「應該」包含下列位元組序列十六進位編碼的字串表示法，指定於 ABNF [[RFC5234]](http://go.microsoft.com/fwlink/?LinkId=123096) 中：(與 MS-SSTR 相同)

>   * %x00 %x00 %x00 %x01 SPSField %x00 %x00 %x00 %x01 PPSField

>   * SPSField 包含序列參數集 (SPS)。

>   * PPSField 包含配量參數集 (PPS)。

>   注意：視訊參數集 (VPS) 未包含在 CodecPrivateData 中，但應包含在 'hvcC' 方塊中之預存檔案的檔案標題中。 使用 Smooth Streaming 通訊協定的系統必須使用自訂屬性「轉碼器」指示其他解碼參數 (例如，HEVC 層)。

##### <a name="22251-customattributeselement"></a>2.2.2.5.1 CustomAttributesElement 

#### <a name="226-streamfragmentelement"></a>2.2.6 StreamFragmentElement 

>   **SmoothStreamingMedia 的 MajorVersion** 欄位「必須」設為 2，且 **MinorVersion** 欄位「必須」設為 2。 (無變更)

##### <a name="22261-trackfragmentelement"></a>2.2.2.6.1 TrackFragmentElement 

### <a name="223-fragment-request"></a>2.2.3 片段要求 

>   **注意**：對 **MinorVersion** 2 和 ‘hev1’ 要求的預設媒體格式，為 [ISO/IEC 14496-12] ISO Base Media 檔案格式第四版和 [ISO/IEC 23001-7] 一般加密第二版中指定的 ‘iso8’ 品牌 ISO Base Media 檔案格式。

### <a name="224-fragment-response"></a>2.2.4 片段回應 

#### <a name="2241-moofbox"></a>2.2.4.1 MoofBox 

#### <a name="2242-mfhdbox"></a>2.2.4.2 MfhdBox 

#### <a name="2243-trafbox"></a>2.2.4.3 TrafBox 

#### <a name="2244-tfxdbox"></a>2.2.4.4 TfxdBox 

>   **TfxdBox** 已被取代，且其函式已取代為 [ISO/IEC 14496-12] 的 8.8.12 條款中指定的「磁軌片段解碼時間方塊」(‘tfdt’)。

>   **注意**：用戶端可能會加總「磁軌執行方塊」('trun') 中所列的樣本持續時間，或將樣本數目乘以預設樣本持續時間，以計算片段的持續時間。 'tfdt' 中的 baseMediaDecodeTime 加上片段持續時間，會等於下一個片段的 URL 時間參數。

>   「製作人參考時間方塊」(‘prft’)「應該」視需要插入到「影片片段方塊」(‘moof’) 前面，以指出與「影片片段方塊」所參考之第一個樣本的「磁軌片段解碼時間」相對應的 UTC 時間，如 [ISO/IEC 14496-12] 的 8.16.5 條款所指定。

#### <a name="2245-tfrfbox"></a>2.2.4.5 TfrfBox 

>   **TfrfBox** 已被取代，且其函式已取代為 [ISO/IEC 14496-12] 的 8.8.12 條款中指定的「磁軌片段解碼時間方塊」(‘tfdt’)。

>   **注意**：用戶端可能會加總「磁軌執行方塊」('trun') 中所列的樣本持續時間，或將樣本數目乘以預設樣本持續時間，以計算片段的持續時間。 'tfdt' 中的 baseMediaDecodeTime 加上片段持續時間，會等於下一個片段的 URL 時間參數。 前查詢位址已被取代，因為其即時串流會出現延遲。

#### <a name="2246-tfhdbox"></a>2.2.4.6 TfhdBox 

>   **TfhdBox** 和相關欄位中包含片段中每個樣本中繼資料的預設值。 **TfhdBox** 欄位的語法是 [[ISO/IEC-14496-12]](http://go.microsoft.com/fwlink/?LinkId=183695) 的 8.8.7 條款所定義之「磁軌片段標題方塊」的嚴格語法子集。

>   **BaseDataOffset (8 位元組)：**，從 **MdatBox** 欄位開頭到 **MdatBox** 欄位中的樣本欄位的位移 (以位元組為單位)。 若要指出此限制，必須設定 default-base-is-moof 旗標 (0x020000)。

#### <a name="2247-trunbox"></a>2.2.4.7 TrunBox 

>   **TrunBox** 和相關欄位中包含所要求之片段的每個樣本中繼資料。 **TrunBox** 的語法是 [[ISO/IEC-14496-](http://go.microsoft.com/fwlink/?LinkId=183695)*12]* 的 8.8.8 條款所定義之「第 1 版磁軌片段執行方塊」的嚴格子集。

>   **SampleCompositionTimeOffset (4 個位元組)：** 每個樣本為了讓片段中第一個顯示之樣本的顯示時間等於第一個解碼樣本的解碼時間，而進行調整的樣本組合時間位移。 「應該」使用反面視訊樣本組合位移，

>   如 [[ISO/IEC-14496-12]](http://go.microsoft.com/fwlink/?LinkId=183695) 所定義。

>   注意：這可以避免等同於最大解碼圖片緩衝移除延遲的視訊延遲音訊所造成的視訊同步處理錯誤，並可在可能有不同移除延遲的替代片段之間保持顯示時機。

>   本節中定義之欄位的語法 (指定於 ABNF [[RFC5234]](http://go.microsoft.com/fwlink/?LinkId=123096) 中) 仍保持相同，但有下列例外：

>   SampleCompositionTimeOffset = SIGNED_INT32

#### <a name="2248-mdatbox"></a>2.2.4.8 MdatBox 

#### <a name="2249-fragment-response-common-fields"></a>2.2.4.9 片段回應通用欄位 

### <a name="225-sparse-stream-pointer"></a>2.2.5 疏鬆資料流指標 

### <a name="226-fragment-not-yet-available"></a>2.2.6 片段尚無法使用 

### <a name="227-live-ingest"></a>2.2.7 即時內嵌 

#### <a name="2271-filetype"></a>2.2.7.1 FileType 

>   **FileType (變數)：** 指定 MPEG-4 ([[MPEG4-RA])](http://go.microsoft.com/fwlink/?LinkId=327787) 檔案的子類型和預定用法，以及高階屬性。

>   **MajorBrand (變數)：** 媒體檔案的主要品牌。 「必須」設為 "isml"。

>   **MinorVersion (變數)：** 媒體檔案的次要版本。 「必須」設為 1。

>   **CompatibleBrands (變數)：** 指定支援的 MPEG-4 品牌。
>   「必須」包含 "ccff" 和 "iso8"。

>   本節中定義之欄位的語法 (指定於 ABNF [[RFC5234]](http://go.microsoft.com/fwlink/?LinkId=123096) 中) 如下所示：

    FileType = MajorBrand MinorVersion CompatibleBrands
    MajorBrand = STRING_UINT32
    MinorVersion = STRING_UINT32
    CompatibleBrands = "ccff" "iso8" 0\*(STRING_UINT32)

**注意**：相容性品牌 ‘ccff’ 和 ‘iso8’ 表示片段符合「一般容器檔案格式」以及一般加密 [ISO/IEC 23001-7] 和 ISO Base Media 檔案格式第 4 版 [ISO/IEC 14496-12]。

#### <a name="2272-streammanifestbox"></a>2.2.7.2 StreamManifestBox 

##### <a name="22721-streamsmil"></a>2.2.7.2.1 StreamSMIL 

#### <a name="2273-liveservermanifestbox"></a>2.2.7.3 LiveServerManifestBox 

##### <a name="22731-livesmil"></a>2.2.7.3.1 LiveSMIL 

#### <a name="2274-moovbox"></a>2.2.7.4 MoovBox 

#### <a name="2275-fragment"></a>2.2.7.5 片段 

##### <a name="22751-track-fragment-extended-header"></a>2.2.7.5.1 磁軌片段擴充標題 

### <a name="228-server-to-server-ingest"></a>2.2.8 伺服器對等內嵌 

# <a name="3-protocol-details"></a>3 通訊協定詳細資料 


## <a name="31-client-details"></a>3.1 用戶端詳細資料 

### <a name="311-abstract-data-model"></a>3.1.1 抽象資料模型 

#### <a name="3111-presentation-description"></a>3.1.1.1 顯示說明 

>   「顯示說明」資料元素中包含顯示的所有中繼資料。

>   顯示中繼資料：顯示中的所有資料流通用的一組中繼資料。 「顯示中繼資料」包含下列指定於 *2.2.2.1* 小節中的欄位：

>   * **MajorVersion**
>   * **MinorVersion**
>   * **TimeScale**
>   * **Duration**
>   * **IsLive**
>   * **LookaheadCount**
>   * **DVRWindowLength**

>   包含 HEVC 資料流的顯示「應該」設定：

    MajorVersion = 2
    MinorVersion = 2

>   LookaheadCount = 0 (注意：方塊已被取代)

>   顯示也「應該」設定：

    TimeScale = 90000

>   資料流集合：「資料流說明」資料元素的集合，如 *3.1.1.1.2* 小節所指定。

>   保護說明：「保護系統中繼資料說明」資料元素的集合，如 *3.1.1.1.1* 小節所指定。

##### <a name="31111-protection-system-metadata-description"></a>3.1.1.1.1 保護系統中繼資料說明 

>   「保護系統中繼資料說明」資料元素中包含單一「內容保護系統」的特定中繼資料。 (無變更)

>   保護標題說明：關於單一「內容保護系統」的內容保護中繼資料。 「保護標題說明」包含下列指定於 *2.2.2.2* 小節中的欄位：

>   * **SystemID**
>   * **ProtectionHeaderContent**

##### <a name="31112-stream-description"></a>3.1.1.1.2 資料流說明 

###### <a name="311121-track-description"></a>3.1.1.1.2.1 磁軌說明 

###### <a name="3111211-custom-attribute-description"></a>3.1.1.1.2.1.1 自訂屬性說明 

##### <a name="3113-fragment-reference-description"></a>3.1.1.3 片段參考說明 

###### <a name="31131-track-specific-fragment-reference-description"></a>3.1.1.3.1 磁軌特定片段參考說明 

#### <a name="3112-fragment-description"></a>3.1.1.2 片段說明 

##### <a name="31121-sample-description"></a>3.1.1.2.1 樣本說明 

### <a name="312-timers"></a>3.1.2 計時器 

### <a name="313-initialization"></a>3.1.3 初始化 

### <a name="314-higher-layer-triggered-events"></a>3.1.4 較高層級觸發事件 

#### <a name="3141-open-presentation"></a>3.1.4.1 開啟顯示 

#### <a name="3142-get-fragment"></a>3.1.4.2 取得片段 

#### <a name="3143-close-presentation"></a>3.1.4.3 關閉顯示 

### <a name="315-processing-events-and-sequencing-rules"></a>3.1.5 處理事件和排序規則 

#### <a name="3151-manifest-request-and-manifest-response"></a>3.1.5.1 資訊清單要求和資訊清單回應 

#### <a name="3152-fragment-request-and-fragment-response"></a>3.1.5.2 片段要求和片段回應

## <a name="32-server-details"></a>3.2 伺服器詳細資料

## <a name="33-live-encoder-details"></a>3.3 即時編碼器詳細資料 

# <a name="4-protocol-examples"></a>4 通訊協定範例 

# <a name="5-security"></a>5 安全性 

## <a name="51-security-considerations-for-implementers"></a>5.1 實作者的安全性考量 
-----------------------------------------

>   如果使用此通訊協定傳輸的內容具有高度商業價值，則應使用「內容保護系統」來防止未經授權者使用該內容。 **ProtectionElement** 可用來傳輸與「內容保護系統」的使用有關的中繼資料。 受保護的音訊和視訊內容「應該」依照 MPEG 一般加密第二版：2015 [ISO/IEC 23001-7] 中的指定進行加密。

>   **注意**：對於 HEVC 視訊，只有 VCL NAL 中的配量資料會加密。 配量標題和其他 NAL 可在解密之前由顯示應用程式存取。 在安全的視訊路徑中，顯示應用程式無法使用加密資訊。

# <a name="52-index-of-security-parameters"></a>5.2 安全性參數的索引 
-----------------------------


| **安全性參數**  | **小節**         |
|-------------------------|---------------------|
| ProtectionElement       | *2.2.2.2*           |
| 一般加密方塊 | *[ISO/IEC 23001-7]* |

# <a name="53-common-encryption-boxes"></a>5.3 一般加密方塊
-----------------------

下列方塊可在一般加密套用時顯示於片段回應中，並且會指定於 [ISO/IEC 23001-7] 或 [ISO/IEC 14496-12] 中：

1.  保護系統特定標題方塊 ('pssh')

2.  樣本加密方塊 (‘senc’)

3.  樣本輔助資訊位移方塊 ('saio')

4.  樣本輔助資訊大小方塊 ('saiz')

5.  樣本群組說明方塊 (‘sgpd’)

6.  取樣至群組方塊 (‘sbgp’)

-----------------------

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[image1]: ./media/media-services-fmp4-live-ingest-overview/media-services-image1.png
[image2]: ./media/media-services-fmp4-live-ingest-overview/media-services-image2.png
[image3]: ./media/media-services-fmp4-live-ingest-overview/media-services-image3.png
[image4]: ./media/media-services-fmp4-live-ingest-overview/media-services-image4.png
[image5]: ./media/media-services-fmp4-live-ingest-overview/media-services-image5.png
[image6]: ./media/media-services-fmp4-live-ingest-overview/media-services-image6.png
[image7]: ./media/media-services-fmp4-live-ingest-overview/media-services-image7.png
