---
title: 媒體服務版本資訊 | Microsoft Docs
description: 媒體服務版本資訊
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 3ca2d7af-1cf0-45fa-9585-3b73f3ee057d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: media
ms.devlang: dotnet
ms.topic: article
ms.date: 07/24/2018
ms.author: juliako
ms.openlocfilehash: cdfd19f2dfd599eacaa0759b63c94767e760a874
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42145341"
---
# <a name="azure-media-services-release-notes"></a>Azure 媒體服務版本資訊
Azure 媒體服務的這些版本資訊彙總了舊版發行後的變更和已知問題。

> [!NOTE]
> 我們想要收到客戶的意見，以針對影響到您的問題進行修正。 若要回報問題或提問，請在 [Azure 媒體服務 MSDN 論壇]中提交貼文。
> 
> 

## <a name="a-idissuescurrently-known-issues"></a><a id="issues"/>目前的已知問題
### <a name="a-idgeneralissuesmedia-services-general-issues"></a><a id="general_issues"/>媒體服務一般問題

| 問題 | 說明 |
| --- | --- |
| 有幾個常用的 HTTP 標題未提供於 REST API 中。 |如果您使用 REST API 來開發媒體服務應用程式，您會發現有些常用的 HTTP 標題欄位 (包括 CLIENT-REQUEST-ID、REQUEST-ID 和 RETURN-CLIENT-REQUEST-ID) 不受支援。 這些標頭將在未來的更新中加入。 |
| 不允許 percent-encoding。 |媒體服務會在建置串流內容的 URL 時，使用 IAssetFile.Name 屬性的值 (例如 `http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters`)。 基於此原因，不允許使用 percent-encoding。 Name 屬性的值不可有下列任何[百分比編碼保留字元](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#[]"。 此外，副檔名只能有一個 "."。 |
| 屬於 Azure Storage SDK 3.x 版的 ListBlobs 方法無法運作。 |媒體服務會根據 [2012-02-12](https://docs.microsoft.com/rest/api/storageservices/Version-2012-02-12) 版本產生 SAS URL。 如果您要使用 Storage SDK 列出 Blob 容器中的 Blob，請使用屬於 Storage SDK 2.x 版的 [CloudBlobContainer.ListBlobs](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.listblobs.aspx) 方法。 |
| 媒體服務節流機制會針對向服務發出過多要求的應用程式限制資源使用量。 服務可能會傳回「服務無法使用」503 HTTP 狀態碼。 |如需詳細資訊，請參閱[媒體服務錯誤碼](media-services-encoding-error-codes.md)中有關於 503 HTTP 狀態碼的說明。 |
| 當您查詢實體時，一次最多只能傳回 1000 個實體，因為公用 REST 第 2 版有 1,000 個查詢結果數目的限制。 |請依照[此 .NET 範例](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities)和[此 REST API 範例](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities)中的說明，使用 Skip 和 Take (.NET)/top (REST)。 |
| 某些用戶端在 Smooth Streaming 資訊清單中可能會遇到重複標記問題。 |如需詳細資訊，請參閱[本節](media-services-deliver-content-overview.md#known-issues)。 |
| 媒體服務 .NET SDK 物件無法序列化，因此無法與 Azure Redis 快取搭配運作。 |如果您嘗試序列化 SDK AssetCollection 物件以將其新增至 Azure Redis 快取，將會擲回例外狀況。 |


## <a name="a-idrestversionhistoryrest-api-version-history"></a><a id="rest_version_history"/>REST API 版本歷程記錄
如需媒體服務 REST API 版本歷程記錄的相關資訊，請參閱 [Azure 媒體服務 REST API 參考]。

## <a name="july-2018"></a>2018 年 7 月

在最新的服務版本中，作業失敗時服務所傳回的錯誤訊息中有一些次要的格式變更，內容是關於分成兩行或多行的原因。

## <a name="may-2018"></a>2018 年 5 月 

從 2018 年 5 月 12 日開始，即時通道將不再支援 RTP/MPEG-2 傳輸串流內嵌通訊協定。 請從 RTP/MPEG-2 移轉到 RTMP 或分散式 MP4 (Smooth Streaming) 內嵌通訊協定。

## <a name="october-2017-release"></a>2017 年 10 月版本
> [!IMPORTANT] 
> 媒體服務正逐漸停止對 Azure 存取控制服務驗證金鑰的支援。 自 2018 年 6 月 22 日起，您將無法再使用存取控制服務金鑰，透過程式碼對媒體服務後端進行驗證。 您必須依據[以Azure AD 為基礎的驗證](media-services-use-aad-auth-to-access-ams-api.md)更新您的程式碼，以使用 Azure Active Directory (Azure AD)。 請留意 Azure 入口網站中有關於這項變更的警告。

### <a name="updates-for-october-2017"></a>2017 年 10 月的更新
#### <a name="sdks"></a>SDK
* .NET SDK 已更新為支援 Azure AD 驗證。 Nuget.org 上的最新 .NET SDK 已移除對於存取控制服務驗證的支援，以鼓勵使用者盡快移轉至 Azure AD。 
* .JAVA SDK 已更新為支援 Azure AD 驗證。 對 Azure AD 驗證的支援已新增至 Java SDK。 如需如何搭配使用 Java SDK 與媒體服務的相關資訊，請參閱[開始使用適用於 Azure 媒體服務的 Java 用戶端 SDK](media-services-java-how-to-use.md)

#### <a name="file-based-encoding"></a>以檔案為基礎的編碼
* 您現在可以使用進階編碼器，將內容編碼至 H.265 高效視訊編碼 (HEVC) 視訊轉碼器。 如果您選擇 H.265，而非其他轉碼器 (例如 H.264)，定價並沒有差別。 如需 HEVC 專利授權的相關資訊，請參閱[線上服務條款](https://azure.microsoft.com/support/legal/)。
* 對於使用 H.265(HEVC) 視訊轉碼器編碼的來源視訊 (例如使用 iOS11 或 GoPro Hero 6 擷取的視訊)，您現在可以使用進階編碼器或標準編碼器來編碼這些視訊。 如需專利授權的相關資訊，請參閱[線上服務條款](https://azure.microsoft.com/support/legal/)。
* 對於包含多重語言音軌的內容，必須根據對應的檔案格式規格 (例如 ISO MP4) 正確地標示語言值。 如此，您就可以使用標準編碼器來編碼內容以進行串流。 結果串流定位器會列出可用的音訊語言。
* 標準編碼器現在支援兩個新的僅限音訊系統預設值，「AAC 音訊」和「AAC 好品質音訊」。 兩者都會產生立體聲進階音訊編碼 (AAC) 輸出，位元速率分別為 128 kbps 和 192 kbps。
* 進階編碼器現在支援以 QuickTime/MOV 檔案格式作為輸入。 視訊轉碼器必須屬於[這篇 GitHub 文章中所列的 Apple ProRes 類型](https://docs.microsoft.com/azure/media-services/media-services-media-encoder-standard-formats)之一。 音訊必須是 AAC 或脈衝碼調制 (PCM)。 進階編碼器不支援以 QuickTime/MOV 檔案中包裝的 DVC/DVCPro 視訊作為輸入 (舉例而言)。 但標準編碼器支援這些視訊轉碼器。
* 編碼器已進行下列錯誤修正：

    * 您現在可以使用輸入資產來提交作業。 這些作業完成後，您可以修改資產 (例如新增、刪除或重新命名資產中的檔案)，並提交其他作業。
    * 標準編碼器產生的 JPEG 縮圖在品質方面已有改善。
    * 標準編碼器可更完善地處理短片中的輸入中繼資料和縮圖產生。
    * 改善標準編碼器中使用的 H.264 解碼器，清除特定罕見構件。 

#### <a name="media-analytics"></a>媒體分析
Azure Media Redactor 的正式運作 - 此媒體處理器會透過將所選個人的臉部模糊化來執行視訊匿名，相當適合用於公共安全及新聞媒體案例。 

如需此新處理器的概觀，請參閱[此部落格文章](https://azure.microsoft.com/blog/azure-media-redactor/)。 如需文件和設定的相關資訊，請參閱[使用 Azure 媒體分析修訂臉部](media-services-face-redaction.md)。



## <a name="june-2017-release"></a>2017 年 6 月版本

媒體服務現在支援[以 Azure AD 為基礎的驗證](media-services-use-aad-auth-to-access-ams-api.md)。

> [!IMPORTANT]
> 目前，媒體服務支援存取控制服務驗證模型。 存取控制服務授權將在 2018 年 6 月 1 日停止。 建議您儘速移轉至 Azure AD 驗證模型。

## <a name="march-2017-release"></a>2017 年 3 月版本

您現在可以使用標準編碼器來[自動產生位元速率階梯](media-services-autogen-bitrate-ladder-with-mes.md)，方法是在您建立編碼工作時指定「彈性資料流」預設字串。 若要將視訊編碼以使用媒體服務進行串流處理，請使用「彈性資料流」預設值。 若要自訂特定案例的編碼預設，您可以開始使用[這些預設值](media-services-mes-presets-overview.md)。

您現在可以使用媒體編碼器標準或媒體編碼器進階工作流程來[建立會產生 fMP4 區塊的編碼工作](media-services-generate-fmp4-chunks.md)。 

## <a name="february-2017-release"></a>2017 年 2 月版本

自 2017 年 4 月 1 日起，您的帳戶中任何超過 90 天的作業記錄以及其相關工作記錄都會自動刪除。 即使記錄總數低於配額上限，然會執行刪除。 若要封存作業/工作資訊，您可以使用[使用媒體服務 .NET SDK 管理資產和相關的實體](media-services-dotnet-manage-entities.md)中說明的程式碼。

## <a name="january-2017-release"></a>2017 年 1 月版本

在媒體服務中，「串流端點」代表可以直接將內容傳遞給用戶端播放程式應用程式或內容傳遞網路 (CDN) 以進一步散發的串流服務。 媒體服務也提供順暢的 Azure 內容傳遞網路整合。 來自 StreamingEndpoint 服務的輸出資料流可以是即時資料流、隨選視訊，也可以是媒體服務帳戶中漸進式的資產下載。 每個媒體服務帳戶皆包含一個預設的串流端點。 您可以在帳戶下建立額外的串流端點。 

版本有 1.0 和 2.0 兩種版本。 從 2017 年 1 月 10 日開始，所有新建立的媒體服務帳戶都會包含 2.0 版預設串流端點。 您新增至此帳戶的額外串流端點也會是 2.0 版。 這項變更不會影響現有的帳戶。 現有的串流端點為 1.0 版，但可以升級至 2.0 版。 這項變更連帶產生了行為、帳單和功能上的變更。 如需詳細資訊，請參閱[串流端點概觀](media-services-streaming-endpoints-overview.md)。

從 2.15 版開始，媒體服務在串流端點實體中新增了下列屬性：

* CdnProvider 
* CdnProfile
* FreeTrialEndTime 
* StreamingEndpointVersion 

如需這些屬性的詳細資訊，請參閱 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。 

## <a name="december-2016-release"></a>2016 年 12 月版本

 現在，您可以使用媒體服務存取其服務的遙測/計量資料。 您可以使用目前的媒體服務版本收集直播頻道、串流端點和即時封存實體的遙測資料。 如需詳細資訊，請參閱[媒體服務遙測](media-services-telemetry-overview.md)。

## <a name="a-idjulychanges16july-2016-release"></a><a id="july_changes16"/>2016 年 7 月版本
### <a name="updates-to-the-manifest-file-ism-generated-by-encoding-tasks"></a>更新編碼工作所產生的資訊清單檔案 (*.ISM)
將編碼工作提交至「媒體編碼器標準」或「媒體編碼器進階」時，編碼工作會在輸出資產中產生[串流資訊清單檔案](media-services-deliver-content-overview.md) (*.ism)。 在最新的服務版本中，此串流資訊清單檔案的語法已更新。

> [!NOTE]
> 串流資訊清單 (.ism) 檔案的語法保留供內部使用。 這在未來的版本中有可能變更。 請勿修改或操作這個檔案的內容。
> 
> 

### <a name="a-new-client-manifest-ismc-file-is-generated-in-the-output-asset-when-an-encoding-task-outputs-one-or-more-mp4-files"></a>當編碼工作輸出一或多個 MP4 檔案時，會在輸出資產中產生新的用戶端資訊清單 (*.ISMC) 檔案
從最新的服務版本開始，在產生一或多個 MP4 檔案的編碼工作完成後，輸出資產也會包含一個串流用戶端資訊清單 (*.ismc) 檔案。 .Ismc 檔案有助於改善動態串流的效能。 

> [!NOTE]
> 用戶端資訊清單 (.ismc) 檔案的語法保留供內部使用。 這在未來的版本中有可能變更。 請勿修改或操作這個檔案的內容。
> 
> 

如需詳細資訊，請參閱 [此部落格](https://blogs.msdn.microsoft.com/randomnumber/2016/07/08/encoder-changes-within-azure-media-services-now-create-ismc-file/)。

### <a name="known-issues"></a>已知問題
某些用戶端在 Smooth Streaming 資訊清單中可能會遇到重複標記問題。 如需詳細資訊，請參閱[本節](media-services-deliver-content-overview.md#known-issues)。

## <a id="apr_changes16"></a>2016 年 4 月版本
### <a name="media-analytics"></a>媒體分析
 媒體服務引進了媒體分析，提供功能強大的視訊智慧。 如需詳細資訊，請參閱[媒體服務分析概觀](media-services-analytics-overview.md)。

### <a name="apple-fairplay-preview"></a>Apple FairPlay (預覽)
現在，您可以使用媒體服務透過 Apple FairPlay 將您的 HTTP 即時串流 (HLS) 內容動態加密。 您也可以使用媒體服務授權傳遞服務，將 FairPlay 授權傳遞給用戶端。 如需詳細資訊，請參閱「使用 Azure 媒體服務串流處理以 Apple FairPlay 保護的 HLS 內容」。

## <a id="feb_changes16"></a>2016 年 2 月版本
Media Services SDK for .NET (3.5.3) 的最新版本包含 Google Widevine 相關錯誤的修正。 AssetDeliveryPolicy 無法重複用於多個以 Widevine 加密的資產。 在此錯誤修正中，已將下列屬性新增至 SDK：WidevineBaseLicenseAcquisitionUrl。

    Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
        new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
    {
        {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl,"http://testurl"},

    };

## <a id="jan_changes_16"></a>2016 年 1 月版本
編碼保留單元已重新命名，以避免與編碼器名稱混淆。

基本、標準和進階編碼保留單元已分別重新命名為 S1、S2 和 S3 保留單元。 現在，使用「基本」編碼保留單元的客戶會在 Azure 入口網站中 (和帳單中) 看見 S1 標籤。 使用「標準」和「進階」的客戶則分別會看見 S2 和 S3 標籤。 

## <a id="dec_changes_15"></a>2015 年 12 月版本

### <a name="media-encoder-deprecation-announcement"></a>媒體編碼器淘汰通知

 媒體編碼器會從「媒體編碼器標準」發行起的約 12 個月內開始被取代。

### <a name="azure-sdk-for-php"></a>Azure SDK for PHP
Azure SDK 小組已發佈新版的 [Azure SDK for PHP](http://github.com/Azure/azure-sdk-for-php) 套件，其中包含媒體服務的更新與新功能。 特別是，Media Services SDK for PHP 現在支援最新的[內容保護](media-services-content-protection-overview.md)功能。 這些功能是採用 AES 和 DRM (PlayReady 與 Widevine)，並選擇性使用權杖限制的動態加密。 它也支援調整 [編碼單位](media-services-dotnet-encoding-units.md)大小。

如需詳細資訊，請參閱

* 以下[程式碼範例](http://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)可協助您快速上手：
  * **vodworkflow_aes.php**：此 PHP 檔案說明如何使用 AES-128 動態加密和金鑰傳遞服務。 其內容以[使用 AES-128 動態加密和金鑰傳遞服務](media-services-protect-with-aes128.md)中說明的 .NET 範例為基礎。
  * **vodworkflow_aes.php**：此 PHP 檔案說明如何使用 PlayReady 動態加密和授權傳遞服務。 其內容以[使用 PlayReady 和/或 Widevine 動態一般加密](media-services-protect-with-playready-widevine.md)中說明的 .NET 範例為基礎。
  * **scale_encoding_units.php**：此 PHP 檔案說明如何調整編碼保留單元大小。

## <a id="nov_changes_15"></a>2015 年 11 月版本
 現在，媒體服務可在雲端提供 Google Widevine 授權傳遞服務。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/)。 同時也參閱[本教學課程](media-services-protect-with-playready-widevine.md)和 [GitHub 存放庫](http://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm)。 

媒體服務所提供的 Widevine 授權傳遞服務為預覽功能。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/)。

## <a id="oct_changes_15"></a>2015 年 10 月版本
媒體服務現在也在下列資料中心推出：巴西南部、印度西部、印度南部和印度中部。 您現在可以使用 Azure 入口網站來[建立媒體服務帳戶](media-services-portal-create-account.md)，以及執行[媒體服務文件網頁](https://azure.microsoft.com/documentation/services/media-services/)中說明的各種工作。 這些資料中心並不會啟用即時編碼。 此外，並非所有類型的編碼保留單元都可用於這些資料中心。

* 巴西南部：只可以使用標準和基本編碼保留單元。
* 印度西部、印度南部和印度中部：只可以使用基本編碼保留單元。

## <a id="september_changes_15"></a>2015 年 9 月版本
媒體服務現在提供以 Widevine 模組 DRM 技術保護點播視訊和即時資料流的能力。 您可以使用下列傳遞服務合作夥伴來協助您傳遞 Widevine 授權：
* [Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/) 
* [EZDRM](http://ezdrm.com/) 
* [castLabs](http://castlabs.com/company/partners/azure/) 

如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/)。
  
您可以使用[媒體服務 .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/) (從 3.5.1 版開始) 或 REST API 設定 AssetDeliveryConfiguration，以使用 Widevine。 
* 媒體服務已新增對 Apple ProRes 視訊的支援。 您現在可以上傳使用 Apple ProRes 或其他轉碼器的 QuickTime 來源視訊檔案。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/announcing-support-for-apple-prores-videos-in-azure-media-services/)。
* 您現在可以使用媒體編碼器標準，進行字幕裁剪和即時封存擷取。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/)。
* 進行了下列篩選更新： 
  
  * 您現在可以搭配使用 Apple HLS 格式與僅限音訊的篩選條件。 您可以利用這項更新，在 URL 中指定 (audio-only=false) 以移除僅限音訊的曲目。
  * 現在，當您為資產定義篩選條件時，您可以在單一 URL 中結合多個 (最多三個) 篩選條件。
    
    如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。
* 媒體服務現在可在 HLS 第 4 版中支援 I-Frames。 I-Frames 支援最佳化向前快轉和倒轉的作業。 根據預設，所有 HLS 第 4 版輸出都會包含 I-Frames 播放清單 (EXT-X-I-FRAME-STREAM-INF)。
如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。

## <a id="august_changes_15"></a>2015 年 8 月版本
* 現在已有適用於 Java 0.8.0 版本的媒體服務 SDK 和新範例可供使用。 如需詳細資訊，請參閱
    
* Azure 媒體播放器已更新而具有多重音訊串流支援。 如需詳細資訊，請參閱 [此部落格文章](https://azure.microsoft.com/blog/2015/08/13/azure-media-player-update-with-multi-audio-stream-support/)。

## <a id="july_changes_15"></a>2015 年 7 月版本
* 宣布「媒體編碼器標準」正式運作。 如需詳細資訊，請參閱 [此部落格文章](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)。
  
    媒體編碼器標準使用[本節](http://go.microsoft.com/fwlink/?LinkId=618336)說明的預設值。 當您使用 4k 編碼的預設時，請取得進階保留單元類型。 如需詳細資訊，請參閱[調整編碼](media-services-scale-media-processing-overview.md)。
* 直播即時字幕用於媒體服務和媒體播放器。 如需詳細資訊，請參閱 [此部落格文章](https://azure.microsoft.com/blog/2015/07/08/live-real-time-captions-with-azure-media-services-and-player/)。

### <a name="media-services-net-sdk-updates"></a>媒體服務 .NET SDK 更新
媒體服務 .NET SDK 的現行版本為 3.4.0.0。 已完成的更新如下： 

* 實作了即時封存的支援。 您無法下載包含即時封存的資產。
* 實作了動態篩選的支援。
* 實作了可讓使用者在刪除資產時保留儲存體容器的功能。
* 完成了在通道中重試原則的相關錯誤修正。
* 已啟用媒體編碼器進階工作流程。

## <a id="june_changes_15"></a>2015 年 6 月版本
### <a name="media-services-net-sdk-updates"></a>媒體服務 .NET SDK 更新
媒體服務 .NET SDK 的現行版本為 3.3.0.0。 已完成的更新如下： 

* 新增了對 OpenId Connect 探索規格的支援。
* 新增了在識別提供者端處理金鑰變換的支援。

如果您使用的識別提供者會公開 OpenID Connect 探索文件 (如同 Azure AD、Google 和 Salesforce 所做的)，您可以指示媒體服務，從 OpenID Connect 探索規格取得 JSON Web 權杖 (JWT) 驗證的簽署金鑰。 

如需詳細資訊，請參閱[在媒體服務中使用 OpenID Connect 探索規格中的 JSON Web 金鑰來處理 JWT 驗證](http://gtrifonov.com/2015/06/07/using-json-web-keys-from-openid-connect-discovery-spec-to-work-with-jwt-token-authentication-in-azure-media-services/)。

## <a id="may_changes_15"></a>2015 年 5 月版本
發表了下列新功能：

* [使用媒體服務進行即時編碼的預覽功能](media-services-manage-live-encoder-enabled-channels.md)
* [動態資訊清單](media-services-dynamic-manifest-overview.md)
* [Azure 媒體超縮時攝影媒體處理器的預覽功能](https://azure.microsoft.com/blog/?p=286281&preview=1&_ppp=61e1a0b3db)

## <a id="april_changes_15"></a>2015 年 4 月版本
### <a name="general-media-services-updates"></a>一般媒體服務更新
* 發表了[媒體播放器](https://azure.microsoft.com/blog/2015/04/15/announcing-azure-media-player/)。
* 從媒體服務 REST 2.10 開始，設定為內嵌即時傳訊通訊協定 (RTMP) 的通道，會和主要與次要內嵌 URL 一起建立。 如需詳細資訊，請參閱[通道內嵌設定](media-services-live-streaming-with-onprem-encoders.md#channel_input)。
* Azure 媒體索引器已更新。
* 新增了西班牙文的支援。
* 新增了 XML 格式的新設定。

如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/)。

### <a name="media-services-net-sdk-updates"></a>媒體服務 .NET SDK 更新
媒體服務 .NET SDK 的現行版本為 3.2.0.0。 已完成的更新如下：

* 重大變更：TokenRestrictionTemplate.Issuer 和 TokenRestrictionTemplate.Audience 變更為字串類型。
* 完成了與建立自訂重試原則有關的更新。
* 完成了與上傳和下載檔案有關的錯誤修正。
* MediaServicesCredentials 類別現在接受以主要和次要存取控制端點作為驗證對象。

## <a id="march_changes_15"></a>2015 年 3 月版本
### <a name="general-media-services-updates"></a>一般媒體服務更新
* 媒體服務現在提供內容傳遞網路整合。 為了支援整合，已將 CdnEnabled 屬性新增至 StreamingEndpoint。 從 2.9 版開始，CdnEnabled 已可與 REST API 搭配使用。 如需詳細資訊，請參閱 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。 從 3.1.0.2 版開始，CdnEnabled 已可與 .NET SDK 搭配使用。 如需詳細資訊，請參閱 [StreamingEndpoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mediaservices.client.istreamingendpoint\(v=azure.10\).aspx)。
* 發表了媒體編碼器進階工作流程。 如需詳細資訊，請參閱[介紹 Azure 媒體服務中的進階編碼](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/)。

## <a id="february_changes_15"></a>2015 年 2 月版本
### <a name="general-media-services-updates"></a>一般媒體服務更新
媒體服務 REST API 的現行版本為 2.9。 從此版本開始，內容傳遞網路整合已可應用於串流端點。 如需詳細資訊，請參閱 [StreamingEndpoint](https://msdn.microsoft.com/library/dn783468.aspx)。

## <a id="january_changes_15"></a>2015 年 1 月版本
### <a name="general-media-services-updates"></a>一般媒體服務更新
宣布內容保護與動態加密公開上市。 如需詳細資訊，請參閱[媒體服務以正式推出的 DRM 技術增強串流安全性](https://azure.microsoft.com/blog/2015/01/29/azure-media-services-enhances-streaming-security-with-general-availability-of-drm-technology/)。

### <a name="media-services-net-sdk-updates"></a>媒體服務 .NET SDK 更新
媒體服務 .NET SDK 的現行版本為 3.1.0.1。

此版本中將預設 Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization.TokenRestrictionTemplate 建構函式標示為過時。 新的建構函式會接受 TokenType 做為引數。

    TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);


## <a id="december_changes_14"></a>2014 年 12 月版本
### <a name="general-media-services-updates"></a>一般媒體服務更新
* 媒體索引器新增了某些更新和新功能。 如需詳細資訊，請參閱 [Azure 媒體索引器 1.1.6.7 的版本資訊](https://azure.microsoft.com/blog/2014/12/03/azure-media-indexer-version-1-1-6-7-release-notes/)。
* 已新增可用來更新編碼保留單元的新 REST API。 如需詳細資訊，請參閱[使用 REST 執行 EncodingReservedUnitType](https://docs.microsoft.com/rest/api/media/operations/encodingreservedunittype)。
* 已新增金鑰傳遞服務的 CORS 支援。
* 已完成查詢授權原則選項的效能改進。
* 在中國資料中心，每位客戶現在都已有其各自的[金鑰傳遞 URL](https://docs.microsoft.com/rest/api/media/operations/contentkey#get_delivery_service_url) (就像其他資料中心)。
* 已新增 HLS 自動目標持續時間。 在執行即時資料流時，會一律動態封裝 HLS。 根據預設，媒體服務會根據主要畫面格間隔 (KeyFrameInterval)，自動計算 HLS 區段封裝比例 (FragmentsPerSegment)。 此方法也稱為接收自即時編碼器的圖片群組 (GOP)。 如需詳細資訊，請參閱[使用媒體服務即時資料流](http://msdn.microsoft.com/library/azure/dn783466.aspx)。

### <a name="media-services-net-sdk-updates"></a>媒體服務 .NET SDK 更新
[媒體服務 .NET SDK](http://www.nuget.org/packages/windowsazure.mediaservices/) 的現行版本為 3.1.0.0。 已完成的更新如下：

* .Net SDK 相依性已升級至 .NET 4.5 Framework。
* 已新增可用來更新編碼保留單元的新 API。 如需詳細資訊，請參閱[使用.NET 更新保留單元類型和增加編碼保留單元](media-services-dotnet-encoding-units.md)。
* 已新增權杖驗證的 JWT 支援。 如需詳細資訊，請參閱[媒體服務和動態加密中的 JWT 權杖驗證](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。
* 已在 PlayReady 授權範本中新增 BeginDate 和 ExpirationDate 的相對位移。

## <a id="november_changes_14"></a>2014 年 11 月版本
* 現在，您可以使用媒體服務透過 SSL 連線擷取即時的 Smooth Streaming (fMP4) 內容。 若要透過 SSL 擷取，請務必將擷取 URL 更新為 HTTPS。 目前媒體服務不支援將 SSL 用於自訂網域。 如需即時資料流的詳細資訊，請參閱[使用 Azure 媒體服務即時資料流](http://msdn.microsoft.com/library/azure/dn783466.aspx)。
* 目前您無法透過 SSL 連線擷取 RTMP 即時資料流。
* 您傳遞內容的來源串流端點必須是在 2014 年 9 月 10 日之後建立的，才能透過 SSL 進行串流處理。 如果您的串流 URL 是以 2014 年 9 月 10 日之後建立的串流端點為基礎，則 URL 會包含 "streaming.mediaservices.windows.net" (新格式)。 包含 "origin.mediaservices.windows.net" (舊格式) 的串流 URL 不支援 SSL。 如果您的 URL 是舊格式，而您想要透過 SSL 進行串流處理，請[建立新的串流端點](media-services-portal-manage-streaming-endpoints.md)。 若要透過 SSL 進行內容的串流處理，請使用以新的串流端點為基礎的 URL。

## <a id="october_changes_14"></a>2014 年 10 月版本
### <a id="new_encoder_release"></a>媒體服務編碼器版本
 發表了新版的媒體服務 Azure 媒體編碼器。 使用最新的媒體編碼器時，您只需要支付輸出 GB 的費用。 此外，新的編碼器是與舊版編碼器相容的功能。 如需詳細資訊，請參閱[媒體服務定價詳細資料]。

### <a id="oct_sdk"></a>媒體服務 .NET SDK
Media Services SDK for .NET 延伸模組的現行版本為 2.0.0.3。

Media Services SDK for .NET 的現行版本為 3.0.0.8。 已完成的更新如下：

* 實作了重試原則類別中的重整。
* 在 HTTP 要求標題中新增了使用者代理程式字串。
* 新增了 NuGet 還原建置步驟。
* 修正了案例測試，以使用來自存放庫的 x509 憑證。
* 新增了通道和串流端更新時適用的驗證設定。

### <a name="new-github-repository-to-host-media-services-samples"></a>新的 GitHub 儲存機制，以主控媒體服務範例
範例位於[媒體服務範例 GitHub 存放庫](https://github.com/Azure/Azure-Media-Services-Samples)中。

## <a id="september_changes_14"></a>2014 年 9 月版本
媒體服務 REST 中繼資料的現行版本為 2.7。 如需最新 REST 更新的詳細資訊，請參閱[媒體服務 REST API 參考](https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference)。

Media Services SDK for .NET 的現行版本為 3.0.0.7

### <a id="sept_14_breaking_changes"></a>重大變更
* 原始來源已重新命名為 [StreamingEndpoint]。
* 使用 Azure 入口網站來編碼並發行 MP4 檔案時的預設行為已變更。

### <a id="sept_14_GA_changes"></a>公開上市版本中的新功能/案例
* 引進了「媒體索引器」媒體處理器。 如需詳細資訊，請參閱[使用媒體索引器編製媒體檔案的索引](http://msdn.microsoft.com/library/azure/dn783455.aspx)。
* 您可以使用 [StreamingEndpoint] 實體新增自訂網域 (主機) 名稱。
  
    若要以自訂網域名稱作為媒體服務串流端點名稱，請將自訂主機名稱新增至您的串流端點。 請使用媒體服務 REST API 或 .NET SDK 新增自訂主機名稱。
  
    您必須考量下列事項：
  
  * 您必須具有自訂網域名稱的擁有權。
  * 網域名稱的擁有權必須通過媒體服務的驗證。 若要驗證網域，請建立對應 MediaServicesAccountId 父網域的 CName，以確認 DNS mediaservices-dns-zone。
  * 您必須建立另一個 CName，將自訂主機名稱 (例如，sports.contoso.com) 對應到您的媒體服務 StreamingEndpont 主機名稱 (例如，amstest.streaming.mediaservices.windows.net)。

    如需詳細資訊，請參閱 [StreamingEndpoint](http://msdn.microsoft.com/library/azure/dn783468.aspx) 一文中的 CustomHostNames 屬性。

### <a id="sept_14_preview_changes"></a>公用預覽版本中的新功能/案例
* 即時資料流預覽。 如需詳細資訊，請參閱[使用媒體服務即時資料流](http://msdn.microsoft.com/library/azure/dn783466.aspx)。
* 金鑰傳遞服務。 如需詳細資訊，請參閱[使用 AES-128 動態加密和金鑰傳遞服務](http://msdn.microsoft.com/library/azure/dn783457.aspx)。
* AES 動態加密。 如需詳細資訊，請參閱[使用 AES-128 動態加密和金鑰傳遞服務](http://msdn.microsoft.com/library/azure/dn783457.aspx)。
* PlayReady 授權傳遞服務。 
* PlayReady 動態加密。 
* 媒體服務 PlayReady 授權範本。 如需詳細資訊，請參閱[媒體服務 PlayReady 授權範本概觀]。
* 資料流儲存體加密資產。 如需詳細資訊，請參閱[資料流儲存體加密內容](http://msdn.microsoft.com/library/azure/dn783451.aspx)。

## <a id="august_changes_14"></a>2014 年 8 月版本
當您為資產編碼時，在編碼工作完成時會產生輸出資產。 在此版本之前，媒體服務編碼器會產生輸出資產的相關中繼資料。 自此版本開始，編碼器也會產生輸入資產的相關中繼資料。 如需詳細資訊，請參閱[輸入中繼資料]和[輸出中繼資料]。

## <a id="july_changes_14"></a>2014 年 7 月版本
Azure Media Services Packager 和 Encryptor 完成了下列錯誤修正：

* 當即時封存資產傳輸至 HLS 時，只會播放音訊：此問題已獲得修正，現在已會同時播放音訊和視訊。
* 當資產封裝至 HLS 和 AES 128 位元信封加密時，封裝的串流不會在 Android 裝置上播放：此問題已獲得修正，封裝的串流已可在支援 HLS 的 Android 裝置上播放。

## <a id="may_changes_14"></a>2014 年 5 月版本
### <a id="may_14_changes"></a>一般媒體服務更新
您現在可以使用[動態封裝]進行 HLS 第 3 版的串流處理。 若要串流處理 HLS 第 3 版，請將下列格式新增至原始定位器路徑：* .ism/manifest(format=m3u8-aapl-v3)。 如需詳細資訊，請參閱[此論壇](https://social.msdn.microsoft.com/Forums/en-US/13b8a776-9519-4145-b9ed-d2b632861fde/dynamic-packaging-to-hls-v3)。

動態封裝現在也支援根據使用 PlayReady 靜態加密的 Smooth Streaming，來傳遞使用 PlayReady 加密的 HLS (第 3 版和第 4 版)。 如需如何使用 PlayReady 為 Smooth Streaming 加密的相關資訊，請參閱[使用 PlayReady 保護 Smooth Streaming](http://msdn.microsoft.com/library/azure/dn189154.aspx)。

### <a name="may_14_donnet_changes"></a>媒體服務 .NET SDK 更新
媒體服務 .NET SDK 的現行版本為 3.0.0.5。 已完成的更新如下：

* 上傳和下載媒體資產時的速度和彈性更為理想。
* 提升了處理重試邏輯和暫時性例外狀況時的效能： 
  
  * 您在查詢、儲存變更、上傳或下載檔案時所造成的例外狀況，在暫時性錯誤偵測和重試邏輯方面已有改善。 
  * 現在，在出現 Web 例外狀況時 (例如，在存取控制服務權杖要求期間)，嚴重錯誤的失效速度已變快。

如需詳細資訊，請參閱 [Media Services SDK for .NET 中的重試邏輯]。

## <a id="april_changes_14"></a>2014 年 4 月編碼器版本
### <a name="april_14_enocer_changes"></a>媒體服務編碼器更新
* 新增了相關支援，而可擷取以 Grass Valley EDIUS 非線性編輯器撰寫的 AVI 檔案。 在此程序中，視訊會使用 Grass Valley HQ/HQX 轉碼器稍微進行壓縮。 如需詳細資訊，請參閱 [Grass Valley 發表透過雲端的 EDIUS 7 Streaming]。
*  新增了為媒體服務編碼器所產生的檔案指定命名慣例的支援。 如需詳細資訊，請參閱[控制媒體服務編碼器輸出檔案名稱](http://msdn.microsoft.com/library/azure/dn303341.aspx)。
*  新增了視訊和 (或) 音訊重疊的支援。 如需詳細資訊，請參閱[建立重疊](http://msdn.microsoft.com/library/azure/dn640496.aspx)。
*  新增了編結多個視訊片段的支援。 如需詳細資訊，請參閱[編結視訊片段](http://msdn.microsoft.com/library/azure/dn640504.aspx)。
* 修正了音訊是以 MPEG-1 Audio Layer 3 (也稱為 MP3) 編碼的 MP4 在轉碼方面的錯誤。

## <a id="jan_feb_changes_14"></a>2014 年 1/2 月版本
### <a name="jan_fab_14_donnet_changes"></a>媒體服務 .NET SDK 3.0.0.1、3.0.0.2 和 3.0.0.3
3.0.0.1 和 3.0.0.2 中的變更包括：

* 修正了使用 OrderBy 陳述式進行 LINQ 查詢的用法相關問題。
* [GitHub] 中的測試方案已分成單元測試和案例測試。

如需變更的詳細資訊，請參閱[媒體服務 .NET SDK 3.0.0.1 和 3.0.0.2 版本](http://gtrifonov.com/2014/02/07/windows-azure-media-services-net-sdk-3-0-0-2-release/index.html)。

3.0.0.3 版做了下列變更：

* Azure 儲存體相依性已升級，而使用 3.0.3.0 版。
* 已修正 3.0.*.* 版的回溯相容性問題。

## <a id="december_changes_13"></a>2013 年 12 月版本
### <a name="dec_13_donnet_changes"></a>媒體服務 .NET SDK 3.0.0.0
> [!NOTE]
> 3.0.x.x 版本沒有與 2.4.x.x 版本之間的回溯相容性。
> 
> 

媒體服務 SDK 目前的最新版本為 3.0.0.0。 您可以從 NuGet 下載最新封裝，或從 [GitHub] 取得。

從媒體服務 SDK 3.0.0.0 版開始，您可以重複使用 [Azure AD 存取控制服務](http://msdn.microsoft.com/library/hh147631.aspx)權杖。 如需詳細資訊，請參閱[使用 Media Services SDK for .NET 連線至媒體服務](http://msdn.microsoft.com/library/azure/jj129571.aspx) 中的「重複使用存取控制服務權杖」一節。

### <a name="dec_13_donnet_ext_changes"></a>媒體服務 .NET SDK 延伸模組 2.0.0.0
 媒體服務 .NET SDK 延伸模組是一組延伸方法和協助程式函式，能夠簡化您的程式碼，讓您輕鬆地以媒體服務進行開發。 您可以從[媒體服務 .NET SDK 延伸模組](https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev)取得最新版本。

## <a id="november_changes_13"></a>2013 年 11 月版本
### <a name="nov_13_donnet_changes"></a>媒體服務 .NET SDK 變更
自此版本起，Media Services SDK for .NET 已可處理在呼叫媒體服務 REST API 層時可能發生的暫時性錯誤。

## <a id="august_changes_13"></a>2013 年 8 月版本
### <a name="aug_13_powershell_changes"></a>Azure SDK 工具中包含的媒體服務 PowerShell Cmdlet
[Azure SDK 工具](https://github.com/Azure/azure-sdk-tools)中現在包含下列媒體服務 PowerShell Cmdlet：

* Get-AzureMediaServices 

    例如：`Get-AzureMediaServicesAccount`
* New-AzureMediaServicesAccount 
  
    例如：`New-AzureMediaServicesAccount -Name “MediaAccountName” -Location “Region” -StorageAccountName “StorageAccountName”`
* New-AzureMediaServicesKey 
  
    例如：`New-AzureMediaServicesKey -Name “MediaAccountName” -KeyType Secondary -Force`
* Remove-AzureMediaServicesAccount 
  
    例如：`Remove-AzureMediaServicesAccount -Name “MediaAccountName” -Force`

## <a id="june_changes_13"></a>2013 年 6 月版本
### <a name="june_13_general_changes"></a>媒體服務變更
本節說明的下列變更是 2013 年 6 月媒體服務版本所包含的更新：

* 能夠將多個儲存體帳戶連結至媒體服務帳戶。 
    * StorageAccount
    * Asset.StorageAccountName 和 Asset.StorageAccount
* 能夠更新 Job.Priority。 
* 通知相關的實體和屬性： 
    * JobNotificationSubscription
    * NotificationEndPoint
    * 工作 (Job)
* Asset.Uri 
* Locator.Name 

### <a name="june_13_dotnet_changes"></a>媒體服務 .NET SDK 變更
2013 年 6 月媒體服務 SDK 版本包含下列變更。 GitHub 提供的最新媒體服務 SDK。

* 自 2.3.0.0 版起，媒體服務 SDK 已可支援將多個儲存體帳戶連結至媒體服務帳戶的作業。 支援此功能的 API 如下：
  
    * IStorageAccount 類型
    * Microsoft.WindowsAzure.MediaServices.Client.CloudMediaContext.StorageAccounts 屬性
    * StorageAccount 屬性
    * StorageAccountName 屬性
  
    如需詳細資訊，請參閱[管理多個儲存體帳戶間的媒體服務資產](http://msdn.microsoft.com/library/azure/dn271889.aspx)。
* 通知相關的 API。 自 2.2.0.0 版起，您已能夠聽取 Azure 佇列儲存體通知。 如需詳細資訊，請參閱[處理媒體服務作業通知](http://msdn.microsoft.com/library/azure/dn261241.aspx)。
  
    * Microsoft.WindowsAzure.MediaServices.Client.IJob.JobNotificationSubscriptions 屬性
    * Microsoft.WindowsAzure.MediaServices.Client.INotificationEndPoint 類型
    * Microsoft.WindowsAzure.MediaServices.Client.IJobNotificationSubscription 類型
    * Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointCollection 類型
    * Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointType 類型
* 對 Storage Client SDK 2.0 (Microsoft.WindowsAzure.StorageClient.dll) 的相依性
* 對 OData 5.5 (Microsoft.Data.OData.dll) 的相依性

## <a id="december_changes_12"></a>2012 年 12 月版本
### <a name="dec_12_dotnet_changes"></a>媒體服務 .NET SDK 變更
* IntelliSense：針對許多類型新增了遺漏的 IntelliSense 文件。
* Microsoft.Practices.TransientFaultHandling.Core：已修正 SDK 對此組件的舊版仍有相依性的問題。 SDK 現已參考此組件的 5.1.1209.1 版。

修正在 2012 年 11 月 SDK 中發現的問題：

* IAsset.Locators.Count：在刪除所有定位器之後，現在已會在新的 IAsset 介面上正確報告此計數。
* IAssetFile.ContentFileSize：現在會在 IAssetFile.Upload(filepath) 上傳之後設定此值。
* IAssetFile.ContentFileSize：現在可在建立資產檔案後設定此屬性。 此檔案過去是唯讀的。
* IAssetFile.Upload(filepath)：已修正此同步上傳方法在將多個檔案上傳至資產時會擲回下列錯誤的問題。 此錯誤為「伺服器無法驗證要求。 請確定授權標頭的值 (包括簽章在內) 格式正確。」
* IAssetFile.UploadAsync：已修正可同時上傳的檔案限制為五個的問題。
* IAssetFile.UploadProgressChanged：此事件現在由 SDK 提供。
* IAssetFile.DownloadAsync(string, BlobTransferClient, ILocator, CancellationToken)：現在提供此方法多載。
* IAssetFile.DownloadAsync：已修正可同時下載的檔案限制為五個的問題。
* IAssetFile.Delete()：已修正若未上傳任何用於 IAssetFile 的檔案，呼叫刪除時可能會擲回例外狀況的問題。
* 作業：已修正在使用作業範本鏈結「MP4 to Smooth Streams 工作」與「PlayReady Protection 工作」時不會建立任何工作的問題。
* EncryptionUtils.GetCertificateFromStore()：此方法已不會因為無法根據憑證設定問題找到憑證，而擲回 Null 參考例外狀況。

## <a id="november_changes_12"></a>2012 年 11 月版本
本節說明的變更是 2012 年 11 月 (2.0.0.0 版) SDK 所包含的更新。 進行這些變更時，可能必須修改或重寫為 2012 年 6 月預覽 SDK 版本撰寫的程式碼。

* Assets
  
    * IAsset.Create(assetName) 是*唯一的*資產建立函式。 IAsset.Create 已不會在方法呼叫期間上傳檔案。 請使用 IAssetFile 進行上傳。
    * IAsset.Publish 方法和 AssetState.Publish 列舉值已從服務 SDK 中移除。 任何依存於此值的程式碼都必須重寫。
* FileInfo
  
    * 此類別已移除，並由 IAssetFile 取代。
  
* IAssetFiles
  
    * IAssetFile 已取代 FileInfo，且具有不同的行為。 若要加以使用，請具現化 IAssetFiles 物件，然後使用媒體服務 SDK 或儲存體 SDK 進行檔案上傳。 您可以使用下列 IAssetFile.Upload 多載：
  
        * IAssetFile.Upload(filePath)：此同步方法會封鎖執行緒，建議僅在上傳單一檔案時使用。
        * IAssetFile.UploadAsync(filePath, blobTransferClient, locator, cancellationToken)：此非同步方法是慣用的上傳機制。 
    
            已知錯誤：如果您使用取消權杖，則會取消上傳。 工作可能會發生許多取消狀態。 您必須正確捕捉並處理例外狀況。
* 定位器
  
    * 原始來源的特定版本已移除。 SAS 特定 context.Locators.CreateSasLocator(asset, accessPolicy) 將標示為已淘汰，或由公開上市版移除。 請參閱「新功能」底下的「定位器」一節，以瞭解更新後的行為。

## <a id="june_changes_12"></a>2012 年 6 月預覽版本
以下是 SDK 的 11 月版本中包含的新功能：

* 刪除實體
  
    * IAsset、IAssetFile、ILocator、IAccessPolicy 和 IContentKey 物件現已在物件層級上刪除 (即 IObject.Delete())，而無須在集合中刪除 (即 cloudMediaContext.ObjCollection.Delete(objInstance))。
* 定位器
  
    * 定位器現在必須使用 CreateLocator 方法來建立。 定位器必須使用 LocatorType.SAS 或 LocatorType.OnDemandOrigin 列舉值作為您要建立之特定定位器類型的引數。
    * 定位器已新增新的屬性，以便取得您的內容可使用的 URI。 定位器的這項重新設計旨在為日後的第三方擴充能力提供更大的彈性，以及提高媒體用戶端應用程式的易用性。
* 非同步方法支援
  
    * 所有方法皆已新增非同步支援。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

<!-- Anchors. -->

<!-- Images. -->

<!--- URLs. --->
[Azure 媒體服務 MSDN 論壇]: http://social.msdn.microsoft.com/forums/azure/home?forum=MediaServices
[Azure 媒體服務 REST API 參考]: https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference
[媒體服務定價詳細資料]: http://azure.microsoft.com/pricing/details/media-services/
[輸入中繼資料]: http://msdn.microsoft.com/library/azure/dn783120.aspx
[輸出中繼資料]: http://msdn.microsoft.com/library/azure/dn783217.aspx
[Deliver content]: http://msdn.microsoft.com/library/azure/hh973618.aspx
[Index media files with the Azure Media Indexer]: http://msdn.microsoft.com/library/azure/dn783455.aspx
[StreamingEndpoint]: http://msdn.microsoft.com/library/azure/dn783468.aspx
[Work with Media Services live streaming]: http://msdn.microsoft.com/library/azure/dn783466.aspx
[Use AES-128 dynamic encryption and the key delivery service]: http://msdn.microsoft.com/library/azure/dn783457.aspx
[Use PlayReady dynamic encryption and the license delivery service]: http://msdn.microsoft.com/library/azure/dn783467.aspx
[Preview features]: http://azure.microsoft.com/services/preview/
[媒體服務 PlayReady 授權範本概觀]: http://msdn.microsoft.com/library/azure/dn783459.aspx
[Stream storage-encrypted content]: http://msdn.microsoft.com/library/azure/dn783451.aspx
[Azure portal]: https://portal.azure.com
[動態封裝]: http://msdn.microsoft.com/library/azure/jj889436.aspx
[Nick Drouin's blog]: http://blog-ndrouin.azurewebsites.net/hls-v3-new-old-thing/
[Protect Smooth Streaming with PlayReady]: http://msdn.microsoft.com/library/azure/dn189154.aspx
[Media Services SDK for .NET 中的重試邏輯]: http://msdn.microsoft.com/library/azure/dn745650.aspx
[Grass Valley 發表透過雲端的 EDIUS 7 Streaming]: http://www.streamingmedia.com/Producer/Articles/ReadArticle.aspx?ArticleID=96351&utm_source=dlvr.it&utm_medium=twitter
[Control Media Services Encoder output file names]: http://msdn.microsoft.com/library/azure/dn303341.aspx
[Create overlays]: http://msdn.microsoft.com/library/azure/dn640496.aspx
[Stitch video segments]: http://msdn.microsoft.com/library/azure/dn640504.aspx
[Azure Media Services .NET SDK 3.0.0.1 and 3.0.0.2 releases]: http://www.gtrifonov.com/2014/02/07/windows-azure-media-services-.net-sdk-3.0.0.2-release/
[Azure AD Access Control Service]: http://msdn.microsoft.com/library/hh147631.aspx
[Connect to Media Services with the Media Services SDK for .NET]: http://msdn.microsoft.com/library/azure/jj129571.aspx
[Media Services .NET SDK extensions]: https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev
[Azure SDK tools]: https://github.com/Azure/azure-sdk-tools
[GitHub]: https://github.com/Azure/azure-sdk-for-media-services
[Manage Media Services assets across multiple Storage accounts]: http://msdn.microsoft.com/library/azure/dn271889.aspx
[Handle Media Services job notifications]: http://msdn.microsoft.com/library/azure/dn261241.aspx

