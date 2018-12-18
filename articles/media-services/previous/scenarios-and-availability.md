---
title: 跨資料中心的 Microsoft Azure 媒體服務功能情節和可用性 | Microsoft Docs
description: 本主題概述跨資料中心的 Microsoft Azure 媒體服務功能和服務情節和可用性。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: 0e9537f09049d5fe056819e6f8b9718cfc52f489
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352800"
---
# <a name="scenarios-and-availability-of-media-services-features-across-datacenters"></a>跨資料中心的媒體服務功能情節和可用性

Microsoft Azure 媒體服務 (AMS) 可讓您安全地上傳、儲存、編碼和封裝視訊或音訊內容，以用於隨選和即時串流傳遞給各種用戶端 (例如電視、電腦和行動裝置)。

AMS 在世界各地多個資料中心運作。 這些資料中心會依據地理區域分組，提供您彈性來選擇要建置應用程式的位置。 您可以檢閱 [地區及其位置的清單](https://azure.microsoft.com/regions/)。 

本主題說明[即時](#live_scenarios)傳遞內容或[隨選](#vod_scenarios)傳遞內容的常見情節。 本主題也會詳述跨資料中心的媒體功能和服務可用性。

## <a name="overview"></a>概觀

### <a name="prerequisites"></a>必要條件

若要開始使用 Azure 媒體服務，您應該具備下列項目：

* 一個 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com)。
* Azure 媒體服務帳戶。 如需詳細資訊，請參閱[建立帳戶](media-services-portal-create-account.md)。
* 您想要串流內容的串流端點必須處於 [執行中] 狀態。

    建立 AMS 帳戶時，會將**預設**串流端點新增至處於 [已停止] 狀態的帳戶。 若要開始串流處理您的內容並利用動態封裝和動態加密功能，串流端點必須處於 [執行中] 狀態。

### <a name="commonly-used-objects-when-developing-against-the-ams-odata-model"></a>針對 AMS OData 模型開發時常用的物件

下列影像顯示針對媒體服務 OData 模型進行開發時一些最常用的物件。

按一下影像可以完整大小檢視。  

<a href="./media/media-services-overview/media-services-overview-object-model.png" target="_blank"><img src="./media/media-services-overview/media-services-overview-object-model-small.png"></a> 

您可以[在此](https://media.windows.net/API/$metadata?api-version=2.15)檢視整個模型。  

## <a name="protect-content-in-storage-and-deliver-streaming-media-in-the-clear-non-encrypted"></a>保護儲存體中的內容並提供不加密的串流媒體

![VoD 工作流程](./media/scenarios-and-availability/scenarios-and-availability01.png)

1. 將高品質媒體檔上傳到資產。

    建議您將儲存體加密選項套用到資產，以便內容在上傳時以及在儲存體中存放時可以得到保護。
2. 編碼成一組調適性位元速率 MP4 檔案。

    建議您將儲存體加密選項套用到輸出資產，以便保護存放的內容。
3. 設定資產傳遞原則 (用於動態封裝)。

    如果您的資產是已加密的儲存體， **必須** 設定資產傳遞原則。
4. 藉由建立 OnDemand 定位器的方法來發行資產。
5. 串流發佈的內容。

如需資料中心內可用性的資訊，請參閱[可用性](#availability)一節。

## <a name="protect-content-in-storage-deliver-dynamically-encrypted-streaming-media"></a>保護儲存體中的內容、提供動態加密串流處理媒體

![利用 PlayReady 保護](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

1. 將高品質媒體檔上傳到資產。 將儲存體加密選項套用到資產。
2. 編碼成一組調適性位元速率 MP4 檔案。 將儲存體加密選項套用到輸出資產。
3. 針對您想要在播放期間動態加密的資產，建立加密內容金鑰。
4. 設定內容金鑰授權原則。
5. 設定資產傳遞原則 (供動態封裝和動態加密使用)。
6. 藉由建立 OnDemand 定位器的方法來發行資產。
7. 串流發佈的內容。

如需資料中心內可用性的資訊，請參閱[可用性](#availability)一節。

## <a name="use-media-analytics-to-derive-actionable-insights-from-your-videos"></a>使用媒體分析從您的視訊衍生可採取行動的見解

媒體分析是一組口說與視覺元件，組織或企業可利用它，從其影片檔輕鬆製作能採取行動的見解。 如需詳細資訊，請參閱 [Azure 媒體服務分析概觀](media-services-analytics-overview.md)。

1. 將高品質媒體檔上傳到資產。
2. 使用[媒體分析概觀](media-services-analytics-overview.md)一節中所述的其中一個媒體分析服務，來處理您的視訊。
3. 媒體分析的媒體處理器會產生 MP4 檔案或 JSON 檔案。 如果媒體處理器產生了 MP4 檔案，您可以漸進式下載檔案。 如果媒體處理器產生了 JSON 檔案，您可以從 Azure Blob 儲存體下載檔案。

如需資料中心內可用性的資訊，請參閱[可用性](#availability)一節。

## <a name="deliver-progressive-download"></a>提供漸進式下載

1. 將高品質媒體檔上傳到資產。
2. 編碼成單一 MP4 檔案。
3. 藉由建立 OnDemand 或 SAS 定位器的方法來發行資產。

    如果使用 SAS 定位器，內容是從 Azure blob 儲存體下載的。 在此情況下，您不需要有已啟動狀態的串流端點。
4. 漸進式下載內容。

## <a id="live_scenarios"></a>傳遞即時串流事件 

1. 使用各種即時串流通訊協定 (例如 RTMP 或 Smooth Streaming) 擷取即時內容。
2. (選擇性) 將您的串流編碼成調適性位元速率串流。
3. 預覽您的即時串流。
4. 透過一般串流通訊協定 (例如，MPEG DASH、Smooth、HLS) 直接將內容傳遞給客戶，或傳遞至內容傳遞網路 (CDN) 供進一步的發佈。

    -或-

    記錄並儲存擷取的內容以於稍後進行串流 (隨選視訊)。

執行即時串流時，您可以選擇下列其中一個路由：

### <a name="working-with-channels-that-receive-multi-bitrate-live-stream-from-on-premises-encoders-pass-through"></a>使用可從內部部署編碼器接收多位元速率即時串流的通道 (即時通行)

下圖顯示 **即時通行** 工作流程中涉及的 AMS 平台主要部分。

![即時工作流程](./media/scenarios-and-availability/media-services-live-streaming-current.png)

如需詳細資訊，請參閱 [使用通道，從內部部署編碼器接收多位元速率即時串流](media-services-live-streaming-with-onprem-encoders.md)。

### <a name="working-with-channels-that-are-enabled-to-perform-live-encoding-with-azure-media-services"></a>使用啟用的通道來以 Azure 媒體服務執行即時編碼

下圖顯示 AMS 平台的主要部分，與通道可以使用媒體服務執行即時編碼的即時串流工作流程有關。

![即時工作流程](./media/scenarios-and-availability/media-services-live-streaming-new.png)

如需詳細資訊，請參閱 [使用啟用的通道以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

如需資料中心內可用性的資訊，請參閱[可用性](#availability)一節。

## <a name="consuming-content"></a>使用內容

Azure 媒體服務提供一些工具，供您用來建立適用於大部分平台的豐富、動態用戶端播放器應用程式，此處所述的平台包括：iOS 裝置、Android 裝置、Windows、Windows Phone、Xbox 和機上盒。 下列主題也會提供 SDK 和 Player Framework 連結，可讓您開發自己的用戶端應用程式，使用來自媒體服務的串流媒體。 如需詳細資訊，請參閱[開發視訊付款人應用程式](media-services-develop-video-players.md)。

## <a name="enabling-azure-cdn"></a>啟用 Azure CDN

媒體服務支援與 Azure CDN 整合。 如需如何啟用 Azure CDN 的資訊，請參閱 [如何管理媒體服務帳戶中的串流端點](media-services-portal-manage-streaming-endpoints.md)。

## <a id="scaling"></a>調整媒體服務帳戶

AMS 客戶可以使用其 AMS 帳戶來調整串流端點、媒體處理和儲存體。

* 媒體服務客戶可以選擇一個**標準**串流端點或一個**進階**串流端點。 大多數的串流工作負載都適合使用**標準**串流端點。 其所含的功能與**進階**串流端點相同，並會自動調整輸出頻寬。 

    **進階**串流端點適合進階工作負載，提供專用並能靈活調整的頻寬容量。 擁有**進階**串流端點的客戶預設會取得一個串流單位 (SU)。 藉由新增 SU 可以調整串流端點。 每個 SU 都可為應用程式提供額外的頻寬容量。 如需調整**進階**串流端點的詳細資訊，請參閱[調整串流端點](media-services-portal-scale-streaming-endpoints.md)主題。

* 媒體服務帳戶是與保留單元類型相關聯，後者決定媒體處理工作的速度。 您可以選擇下列的保留單元類型：**S1**、**S2** 或 **S3**。 例如，在執行相同編碼作業的前提下，使用 **S2** 保留單元類型的速度會比 **S1** 類型快。

    除了指定保留單元類型之外，您還可以指定使用**保留單元** (RU) 來佈建帳戶。 佈建的 RU 數目可決定指定帳戶中可同時處理的媒體工作數目。

    >[!NOTE]
    >RU 用於平行化所有媒體處理，包括使用 Azure 媒體索引器的索引作業。 不過，與編碼不同，索引工作的處理速度不會因為使用較快的保留單元而變快。

    如需詳細資訊，請參閱[調整媒體處理](media-services-portal-scale-media-processing.md)。
* 您也可以透過新增儲存體帳戶，來調整媒體服務帳戶。 每個儲存體帳戶的限制為 500 TB。 若想將儲存空間擴大為超過預設限制，您可以選擇將多個儲存體帳戶附加至單一媒體服務帳戶。 如需詳細資訊，請參閱[管理儲存體帳戶](meda-services-managing-multiple-storage-accounts.md)。

## <a id="availability"></a> 跨資料中心的媒體服務功能可用性

本節詳述跨資料中心的媒體服務功能可用性。

### <a name="ams-accounts"></a>AMS 帳戶

#### <a name="availability"></a>可用性

若要判斷資料中心的媒體服務是否可用，請瀏覽至 https://azure.microsoft.com/status/ 並捲動到「媒體」資料表。

### <a name="streaming-endpoints"></a>串流端點 

媒體服務客戶可以選擇一個**標準**串流端點或一個**進階**串流端點。 如需詳細資訊，請參閱[調整](#scaling)一節。

#### <a name="availability"></a>可用性

|名稱|狀態|資料中心
|---|---|---|
|標準|GA|全部|
|進階|GA|全部|

### <a name="live-encoding"></a>即時編碼

#### <a name="availability"></a>可用性

所有資料中心內都提供，但不含：德國、巴西南部、印度西部、印度南部和印度中部。 

### <a name="encoding-media-processors"></a>編碼媒體處理器

AMS 提供兩個隨選編碼器：**媒體編碼器標準**和**媒體編碼器進階工作流程**。 如需詳細資訊，請參閱 [Azure 隨選媒體編碼器的概觀和比較](media-services-encode-asset.md)。 

#### <a name="availability"></a>可用性

|媒體處理器名稱|狀態|資料中心
|---|---|---|
|Media Encoder Standard|GA|全部|
|媒體編碼器高階工作流程|GA|所有區域 (中國除外)|

### <a name="analytics-media-processors"></a>分析媒體處理器

媒體分析是一組語音與視覺元件，可讓組織或企業從其影片檔輕鬆地產生能採取行動的見解。 如需詳細資訊，請參閱 [Azure 媒體服務分析概觀](media-services-analytics-overview.md)。

#### <a name="availability"></a>可用性

|媒體處理器名稱|狀態|資料中心
|---|---|---|
|Azure 媒體臉部偵測器|預覽|全部|
|Azure Media Hyperlapse|預覽|全部|
|Azure Media Indexer|GA|全部|
|Azure 媒體動作偵測器|預覽|全部|
|Azure 媒體 OCR|預覽|全部|
|Azure 媒體Media Redactor|預覽|全部|
|Azure Media Stabilizer|預覽|全部|
|Azure 媒體視訊縮圖|預覽|全部|
|Azure 媒體索引器 2|預覽|所有區域 (中國和美國聯邦政府區域除外)|

### <a name="protection"></a>保護

Microsoft Azure 媒體服務可讓您保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。 如需詳細資訊，請參閱[保護 AMS 內容](media-services-content-protection-overview.md)。

#### <a name="availability"></a>可用性

|加密|狀態|資料中心|
|---|---|---| 
|儲存體|GA|全部|
|AES-128 金鑰|GA|全部|
|Fairplay|GA|全部|
|PlayReady|GA|全部|
|Widevine|GA|全部，但不含德國、美國聯邦政府和中國。

### <a name="reserved-units-rus"></a>保留單元 (RU)

佈建的保留單元數目可決定指定帳戶中可同時處理的媒體工作數目。 

如需詳細資訊，請參閱[調整](#scaling)一節。

#### <a name="availability"></a>可用性

所有資料中心內都提供。

### <a name="reserved-unit-ru-type"></a>保留單元 (RU) 類型

媒體服務帳戶是與保留單元類型相關聯，後者決定媒體處理工作的處理速度。 您可以選擇下列的保留單元類型：S1、S2 或 S3。

如需詳細資訊，請參閱[調整](#scaling)一節。

#### <a name="availability"></a>可用性

|RU 類型名稱|狀態|資料中心
|---|---|---|
|S1|GA|全部|
|S2|GA|全部，但不含巴西南部和印度西部|
|S3|GA|全部，但不含印度西部|

## <a name="next-steps"></a>後續步驟

檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

