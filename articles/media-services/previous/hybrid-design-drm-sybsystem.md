---
title: 使用 Azure 媒體服務的 DRM 子系統混合式設計 | Microsoft Docs
description: 本主題討論使用 Azure 媒體服務的 DRM 子系統混合式設計。
services: media-services
documentationcenter: ''
author: willzhan
manager: cfowler
editor: ''
ms.assetid: 18213fc1-74f5-4074-a32b-02846fe90601
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: willzhan;juliako
ms.openlocfilehash: 1584dab96d1cd772bf04620c68dbe1f133304a1c
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783557"
---
# <a name="hybrid-design-of-drm-subsystems"></a>DRM 子系統的混合式設計

本主題討論使用 Azure 媒體服務的 DRM 子系統混合式設計。

## <a name="overview"></a>概觀

Azure 媒體服務提供下列三個 DRM 系統支援：

* PlayReady
* Widevine (模組)
* FairPlay

DRM 支援包括 DRM 加密 (動態加密) 和授權傳遞，加上當作瀏覽器玩家 SDK 的所有 3 DRM Azure 媒體播放器支援。

如需 DRM/CENC 子系統設計和實作的詳細處理方式，請參閱[使用多重 DRM 和存取控制的 CENC](media-services-cenc-with-multidrm-access-control.md) 一文。

雖然我們為三種 DRM 系統提供完整的支援，但有時候客戶在 Azure 媒體服務之外，也需要使用自己的基礎結構/子系統的各種組件建置混合式的 DRM 子系統。

以下是客戶常問的一些問題：

* 「我可以使用自己的 DRM 授權伺服器嗎？」 (在此例中，客戶投資了內嵌商務邏輯的 DRM 授權伺服器叢集。)
* 「我可以只使用 Azure 媒體服務的 DRM 授權傳遞，但 AMS 中不裝載內容嗎？」

## <a name="modularity-of-the-ams-drm-platform"></a>AMS DRM 平台的模組化

Azure 媒體服務 DRM 是全面雲端視訊平台的一部分，設計富彈性和模組化。 您可以使用 Azure 媒體服務搭配下表所述的任何不同組合 (後文有資料表所用標記法說明)。 

|**內容裝載與來源**|**內容加密**|**DRM 授權傳遞**|
|---|---|---|
|AMS|AMS|AMS|
|AMS|AMS|協力廠商|
|AMS|協力廠商|AMS|
|AMS|協力廠商|協力廠商|
|協力廠商|協力廠商|AMS|

### <a name="content-hosting--origin"></a>內容裝載與來源

* AMS：影片資產裝載於 AMS 中，而資料流是透過 AMS 串流端點 (但不一定是動態封裝)。
* 協力廠商：影片是在 AMS 之外的協力廠商資料流平台上裝載和傳遞。

### <a name="content-encryption"></a>內容加密

* AMS：內容加密是由 AMS 動態加密動態/隨選執行。
* 協力廠商：內容加密是使用前置處理工作流程在 AMS 之外執行。

### <a name="drm-license-delivery"></a>DRM 授權傳遞

* AMS：DRM 授權是由 AMS 授權傳遞服務傳遞。
* 協力廠商：DRM 授權是由 AMS 之外的協力廠商 DRM 授權伺服器傳遞。

## <a name="configure-based-on-your-hybrid-scenario"></a>根據混合式案例設定

### <a name="content-key"></a>內容金鑰

透過內容金鑰的設定，您可以控制 AMS 動態加密和 AMS 授權傳遞服務的下列屬性：

* 用於動態 DRM 加密的內容金鑰。
* 由授權傳遞服務所傳遞的 DRM 授權內容：權限、內容金鑰和限制。
* **內容金鑰授權原則限制**的類型：開啟、IP 或權杖限制。
* 如果**使用內容金鑰授權原則限制**的**權杖**類型，則必須符合**內容金鑰授權原則限制**才能簽發授權。

### <a name="asset-delivery-policy"></a>資產傳遞原則

透過資產傳遞原則設定，您可以控制 AMS 動態封裝程式和 AMS 串流端點動態加密所使用的下列屬性：

* 串流處理通訊協定和 DRM 加密的組合，例如 CENC (PlayReady 和 Widevine) 下的 DASH、PlayReady 下的 Smooth Streaming、Widevine 或 PlayReady 下的 HLS。
* 每個有關 DRM 的預設/內嵌授權傳遞 URL。
* 無論 DASH MPD 或 HLS 播放清單的授權取得 URL (LA_URL) 是否分別包含 Widevine 和 FairPlay 的索引鍵識別碼 (KID) 查詢字串。

## <a name="scenarios-and-samples"></a>案例與範例

根據上一節的說明，下列五個混合式案例分別使用**內容金鑰**-**資產傳遞原則**設定組合 (資料表最後一個資料行中提及的範例)：

|**內容裝載與來源**|**DRM 加密**|**DRM 授權傳遞**|**設定內容金鑰**|**設定資產傳遞原則**|**範例**|
|---|---|---|---|---|---|
|AMS|AMS|AMS|yes|yes|範例 1|
|AMS|AMS|協力廠商|yes|yes|範例 2|
|AMS|協力廠商|AMS|yes|否|範例 3|
|AMS|協力廠商|外部|否|否|範例 4|
|協力廠商|協力廠商|AMS|yes|否|    

在範例中，PlayReady 保護對 DASH 和 Smooth Streaming 都有效。 下列的影片 URL 是 Smooth Streaming URL。 若要取得對應的 DASH URL，只要加上 "(format=mpd-time-csf)" 即可。 您可以使用 [Azure 媒體測試播放器](http://aka.ms/amtest)在瀏覽器中測試。 它可讓您設定要在哪些技術之下使用哪一個串流處理通訊協定。 Windows 10 的 IE11 和 MS Edge 都透過 EME 支援 PlayReady。 如需詳細資訊，請參閱[測試工具的詳細資料](https://blogs.msdn.microsoft.com/playready4/2016/02/28/azure-media-test-tool/)。

### <a name="sample-1"></a>範例 1

* 來源 (基底) URL：https://willzhanmswest.streaming.mediaservices.windows.net/1efbd6bb-1e66-4e53-88c3-f7e5657a9bbd/RussianWaltz.ism/manifest 
* PlayReady LA_URL (虛線與平滑)：https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/ 
* Widevine LA_URL (虛線)：https://willzhanmswest.keydelivery.mediaservices.windows.net/Widevine/?kid=78de73ae-6d0f-470a-8f13-5c91f7c4 
* FairPlay LA_URL (HLS)：https://willzhanmswest.keydelivery.mediaservices.windows.net/FairPlay/?kid=ba7e8fb0-ee22-4291-9654-6222ac611bd8 

### <a name="sample-2"></a>範例 2

* 來源 (基底) URL：http://willzhanmswest.streaming.mediaservices.windows.net/1a670626-4515-49ee-9e7f-cd50853e41d8/Microsoft_HoloLens_TransformYourWorld_816p23.ism/Manifest 
* PlayReady LA_URL (虛線與平滑)：http://willzhan12.cloudapp.net/PlayReady/RightsManager.asmx 

### <a name="sample-3"></a>範例 3

* 來源 URL：https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (虛線與平滑)：https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/ 

### <a name="sample-4"></a>範例 4

* 來源 URL：https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (虛線與平滑)：https://willzhan12.cloudapp.net/playready/rightsmanager.asmx 

## <a name="summary"></a>總結

總而言之，Azure 媒體服務 DRM 元件富有彈性，只要如本主題所述正確設定內容金鑰與資產傳遞原則，就可以在混合式案例中使用它們。

## <a name="next-steps"></a>後續步驟
檢視媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

