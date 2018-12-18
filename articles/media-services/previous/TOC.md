# [媒體服務文件](index.md)

# [概觀](media-services-overview.md)
## [案例和可用性](scenarios-and-availability.md)
## [概念](media-services-concepts.md)

# 開始使用
## [建立和管理帳戶](media-services-portal-create-account.md)
## [設定開發環境](media-services-set-up-computer.md)
### [.NET](media-services-dotnet-how-to-use.md)
### [REST](media-services-rest-how-to-use.md)  
## [使用 Azure AD 驗證來存取 API](media-services-use-aad-auth-to-access-ams-api.md)
### [使用入口網站來管理 Azure AD 驗證](media-services-portal-get-started-with-aad.md)
### [使用 .NET 存取 API](media-services-dotnet-get-started-with-aad.md)
### [使用 REST 存取 API](media-services-rest-connect-with-aad.md)
### [使用 Azure CLI 來建立及設定 Azure AD 應用程式](media-services-cli-create-and-configure-aad-app.md)
### [使用 Azure PowerShell 來建立及設定 Azure AD 應用程式](media-services-powershell-create-and-configure-aad-app.md)

## 傳遞隨選影片
### [Azure 入口網站](media-services-portal-vod-get-started.md)
### [.NET SDK](media-services-dotnet-get-started.md)
### [Java](media-services-java-how-to-use.md)
### [REST](media-services-rest-get-started.md)
## 執行即時串流
### [Azure 入口網站](media-services-portal-live-passthrough-get-started.md)
### [.NET](media-services-dotnet-live-encode-with-onpremises-encoders.md)

# 作法
## 管理
### 實體
#### [.NET](media-services-dotnet-manage-entities.md)
#### [REST](media-services-rest-manage-entities.md)
### [串流端點](media-services-streaming-endpoints-overview.md)
#### [Azure 入口網站](media-services-portal-manage-streaming-endpoints.md)
#### [.NET](media-services-dotnet-manage-streaming-endpoints.md)
### 儲存體
#### [更換儲存體存取金鑰之後更新媒體服務](media-services-roll-storage-access-keys.md)
#### [管理跨多個儲存體帳戶的資產](meda-services-managing-multiple-storage-accounts.md)
### [配額和限制](media-services-quotas-and-limitations.md)
## [設定 Postman](media-rest-apis-with-postman.md)
### [隨選串流集合](postman-collection.md)
### [即時串流集合](postman-live-streaming-collection.md)
### [環境](postman-environment.md)
## 上傳內容
### 將檔案上傳至帳戶
#### [Azure 入口網站](media-services-portal-upload-files.md)
#### [.NET](media-services-dotnet-upload-files.md)
#### [REST](media-services-rest-upload-files.md)
### [透過 Aspera 上傳大型檔案](media-services-upload-files-with-aspera.md)
### [透過 StorSimple 上傳檔案](media-services-upload-files-from-storsimple.md)
### [複製現有的 Blob](media-services-copying-existing-blob.md)

## [編碼內容](media-services-encode-asset.md)
### [比較編碼器](media-services-compare-encoders.md)
### [管理編碼速度和並行功能](media-services-manage-encoding-speed.md)
### 媒體編碼器標準 (MES)
#### [媒體編碼器標準格式和轉碼器](media-services-media-encoder-standard-formats.md)
#### [使用 MES 自動產生位元速率階梯](media-services-autogen-bitrate-ladder-with-mes.md)
#### 以媒體編碼器標準編碼
##### [Azure 入口網站](media-services-portal-encode.md)
##### [.NET](media-services-dotnet-encode-with-media-encoder-standard.md)
##### [REST](media-services-rest-encode-asset.md)
#### [使用 MES 進階編碼](media-services-advanced-encoding-with-mes.md)
##### [自訂媒體編碼器標準預設值](media-services-custom-mes-presets-with-dotnet.md)
##### [如何搭配 .NET 使用媒體編碼器標準產生縮圖](media-services-dotnet-generate-thumbnail-with-mes.md)
##### [以 Media Encoder Standard 裁剪影片](media-services-crop-video.md)
##### [產生縮圖原件](generate-thumbnail-sprite.md)
#### MES 結構描述
##### [媒體編碼器標準結構描述](media-services-mes-schema.md)
##### [輸入中繼資料](media-services-input-metadata-schema.md)
##### [輸出中繼資料](media-services-output-metadata-schema.md)
#### [MES 預設值](media-services-mes-presets-overview.md) 
##### [H264 多重位元速率 1080p 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1.md)
##### [H264 多重位元速率 1080p](media-services-mes-preset-H264-Multiple-Bitrate-1080p.md)
##### [H264 多重位元速率 16x9 SD 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1.md)
##### [H264 多重位元速率 16x9 SD](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD.md)
##### [H264 多重位元速率 16x9 (iOS 適用)](media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS.md)
##### [H264 多重位元速率 4K 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1.md)
##### [H264 多重位元速率 4K](media-services-mes-preset-H264-Multiple-Bitrate-4K.md)
##### [H264 多重位元速率 4x3 SD 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1.md)
##### [H264 多重位元速率 4x3 SD](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD.md)
##### [H264 多重位元速率 4x3 (適用於 iOS)](media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS.md)
##### [H264 多重位元速率 720p 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1.md)
##### [H264 多重位元速率 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md)
##### [H264 單一位元速率 1080p 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1.md)
##### [H264 單一位元速率 1080p](media-services-mes-preset-H264-Single-Bitrate-1080p.md)
##### [H264 單一位元速率 16x9 SD 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1.md)
##### [H264 單一位元速率 16x9 SD](media-services-mes-preset-H264-Single-Bitrate-16x9-SD.md)
##### [H264 單一位元速率 4K 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1.md)
##### [H264 單一位元速率 4K](media-services-mes-preset-H264-Single-Bitrate-4K.md)
##### [H264 單一位元速率 4x3 SD 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1.md)
##### [H264 單一位元速率 4x3 SD](media-services-mes-preset-H264-Single-Bitrate-4x3-SD.md)
##### [H264 單一位元速率 720p 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1.md)
##### [H264 單一位元速率 720p](media-services-mes-preset-H264-Single-Bitrate-720p.md)
##### [H264 單一位元速率 720p (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-720p-for-Android.md)
##### [H264 單一位元速率高品質 SD (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android.md)
##### [H264 單一位元速率低品質 SD (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android.md)
### 媒體編碼器高階工作流程
#### [媒體編碼器高階工作流程格式和轉碼器](media-services-premium-workflow-encoder-formats.md)
#### 使用媒體編碼器高階工作流程進行編碼
##### [Media Encoder Premium Workflow](media-services-encode-with-premium-workflow.md)
##### [媒體編碼器高階工作流程教學課程](media-services-media-encoder-premium-workflow-tutorials.md)
##### [使用工作流程設計工具建立進階編碼工作流程](media-services-workflow-designer.md)
##### [進階工作流程與多個輸入](media-services-media-encoder-premium-workflow-multiplefilesinput.md)
### [建立會產生 fMP4 區塊的工作](media-services-generate-fmp4-chunks.md)
### 媒體處理器
#### [.NET](media-services-get-media-processor.md)
#### [REST](media-services-rest-get-media-processor.md)
### [錯誤碼](media-services-encoding-error-codes.md)
### 取代
#### [靜態封裝和加密](media-services-static-packaging.md)

## [即時串流](media-services-manage-channels-overview.md)
### [內部部署編碼器](media-services-live-streaming-with-onprem-encoders.md)
#### [建議的內部部署編碼器](media-services-recommended-encoders.md)
#### [Azure 入口網站](media-services-portal-live-passthrough-get-started.md)
#### [.NET](media-services-dotnet-live-encode-with-onpremises-encoders.md)
### [使用雲端編碼器即時串流](media-services-manage-live-encoder-enabled-channels.md)
#### [Azure 入口網站](media-services-portal-creating-live-encoder-enabled-channel.md)
#### [.NET](media-services-dotnet-creating-live-encoder-enabled-channel.md)
### [設定內部部署編碼器與雲端編碼器搭配運作](media-services-live-encoders-overview.md)
#### [FMLE 編碼器](media-services-configure-fmle-live-encoder.md)
#### [Haivision KB 編碼器](media-services-configure-kb-live-encoder.md)
#### [NewTek TriCaster 編碼器](media-services-configure-tricaster-live-encoder.md)
#### [Wirecast 編碼器](media-services-configure-wirecast-live-encoder.md)
### [處理長時間執行的作業](media-services-dotnet-long-operations.md)
### [分散式 MP4 即時內嵌規格](media-services-fmp4-live-ingest-overview.md)

## [裁剪內容](media-services-azure-media-clipper-overview.md)
### [開始使用](media-services-azure-media-clipper-getting-started.md)
### [載入影片](media-services-azure-media-clipper-load-assets.md)
### [設定鍵盤快速鍵](media-services-azure-media-clipper-keyboard-shortcuts.md)
### [設定當地語系化](media-services-azure-media-clipper-localization.md)
### [提交裁剪作業](media-services-azure-media-clipper-submit-job.md)
### [Azure 入口網站](media-services-azure-media-clipper-portal.md)

## [保護內容](media-services-content-protection-overview.md)
### [儲存體加密](media-services-rest-storage-encryption.md)
### [AES-128 加密](media-services-protect-with-aes128.md)
### [適用於 Stream 的 PlayReady/Widevine](media-services-protect-with-playready-widevine.md)
### [適用於 Stream 的 FairPlay](media-services-protect-hls-with-fairplay.md)
### [適用於 Windows 10 的離線 PlayReady](https://blogs.msdn.microsoft.com/playready4/2016/10/26/does-azure-media-services-support-offline-mode/)
### [適用於 iOS 的離線 FairPlay](media-services-protect-hls-with-offline-fairplay.md)
### [適用於 Android 的離線 Widevine](offline-widevine-for-android.md)
### [在 Azure 入口網站中進行設定](media-services-portal-protect-content.md)
### [傳遞 DRM 授權](media-services-deliver-keys-and-licenses.md)
### 建立 ContentKeys
#### [.NET](media-services-dotnet-create-contentkey.md)
#### [REST](media-services-rest-create-contentkey.md)
### 授權範本概觀
#### [PlayReady 授權範本](media-services-playready-license-template-overview.md)
#### [Widevine 授權範本](media-services-widevine-license-template-overview.md)
### 設定資產傳遞原則
#### [.NET](media-services-dotnet-configure-asset-delivery-policy.md)
#### [REST](media-services-rest-configure-asset-delivery-policy.md)
### 設定內容金鑰授權原則
#### [Azure 入口網站](media-services-portal-configure-content-key-auth-policy.md)
#### [.NET](media-services-dotnet-configure-content-key-auth-policy.md)
#### [REST](media-services-rest-configure-content-key-auth-policy.md)
### [將驗證權杖傳送至 AMS](media-services-pass-authentication-tokens.md)
### 參考設計
#### [混合式 DRM 系統設計](hybrid-design-drm-sybsystem.md)
#### [參考多重 DRM 設計](media-services-cenc-with-multidrm-access-control.md)

## [分析](media-services-analytics-overview.md)
### [使用 Azure 入口網站分析媒體](media-services-portal-analyze.md)
### [利用索引器 2 處理](media-services-process-content-with-indexer2.md)
### [利用索引器處理](media-services-index-content.md)
#### [工作預設](indexer-task-preset.md)
### [利用 Hyperlapse 處理](media-services-hyperlapse-content.md)
### [利用臉部偵測器處理](media-services-face-and-emotion-detection.md)
### [利用動態偵測器處理](media-services-motion-detection.md)
### [利用 Face Redactor 處理](media-services-face-redaction.md)
#### [Face Redactor 逐步解說](media-services-redactor-walkthrough.md)
### [利用視訊縮圖處理](media-services-video-summarization.md)
### [利用 OCR 處理](media-services-video-optical-character-recognition.md)
### [利用 Content Moderator](media-services-content-moderation.md)

## [設定遙測](media-services-telemetry-overview.md)
###[.NET](media-services-dotnet-telemetry.md)
###[REST](media-services-rest-telemetry.md)

## 調整
### [媒體處理](media-services-scale-media-processing-overview.md)
#### [Azure 入口網站](media-services-portal-scale-media-processing.md)
#### [.NET](media-services-dotnet-encoding-units.md)
### 串流端點
#### [Azure 入口網站](media-services-portal-scale-streaming-endpoints.md)

## [傳遞內容](media-services-deliver-content-overview.md)
### [動態封裝](media-services-dynamic-packaging-overview.md)
### [篩選器和動態資訊清單概觀](media-services-dynamic-manifest-overview.md)
#### [使用 .NET 建立篩選器](media-services-dotnet-dynamic-manifest.md)
#### [使用 REST 建立篩選器](media-services-rest-dynamic-manifest.md)
### [媒體服務延伸模組中的 CDN 快取原則](../../cdn/cdn-caching-policy.md?toc=%2fazure%2fmedia-services%2ftoc.json)
### 發佈內容
#### [Azure 入口網站](media-services-portal-publish.md)
#### [.NET](media-services-deliver-streaming-content.md)
#### [REST](media-services-rest-deliver-streaming-content.md)
### [依下載進行傳遞](media-services-deliver-asset-download.md)
### [容錯移轉串流案例](media-services-implement-failover.md)

## 取用
### [使用現有播放器來播放媒體](media-services-playback-content-with-existing-players.md)
### [使用媒體播放器來播放媒體](media-services-develop-video-players.md)
### 其他播放選項
#### [Smooth Streaming Windows 市集應用程式](media-services-build-smooth-streaming-apps.md)
#### [具有 DASH.js 的 HTML5 應用程式](media-services-embed-mpeg-dash-in-html5.md)
#### [Adobe 開放原始碼媒體架構播放器](media-services-use-osmf-smooth-streaming-client-plugin.md)
### [在用戶端插入廣告](media-services-inserting-ads-on-client-side.md)
### [授權 Microsoft Smooth Streaming 用戶端移轉套件](media-services-sspk.md)

## 整合
### [使用 Azure Functions 搭配媒體服務](media-services-dotnet-how-to-use-azure-functions.md)
### [Azure Functions 搭配媒體服務範例](https://github.com/Azure-Samples/media-services-dotnet-functions-integration)

## 監視
### 檢查作業進度
#### [REST](media-services-rest-check-job-progress.md)
#### [Azure 入口網站](media-services-portal-check-job-progress.md)
#### [.NET](media-services-check-job-progress.md)
### [監視搭配佇列儲存體的作業通知](media-services-dotnet-check-job-progress-with-queues.md)
### [監視搭配 webhook 的作業通知](media-services-dotnet-check-job-progress-with-webhooks.md)

## 疑難排解
### [常見問題集](media-services-frequently-asked-questions.md)
### [即時串流的疑難排解指南](media-services-troubleshooting-live-streaming.md)
### [錯誤碼](media-services-error-codes.md)
### [重試邏輯](media-services-retry-logic-in-dotnet-sdk.md)

# 參考
## [程式碼範例](https://azure.microsoft.com/resources/samples/?service=media-services)
## [Azure PowerShell (Resource Manager)](/powershell/module/azurerm.media)
## [Azure PowerShell (服務管理)](/powershell/module/servicemanagement/azure/?view=azuresmps-3.7.0)
## [.NET](/dotnet/api/microsoft.windowsazure.mediaservices.client)
## [REST](/rest/api/media/mediaservice)
## 規格
### [即時內嵌 - 分散式 MP4 即時內嵌規格](media-services-fmp4-live-ingest-overview.md)
### [即時內嵌 - 即時資料流中的已逾時中繼資料訊號](media-services-specifications-live-timed-metadata.md)
### [Smooth Streaming HEVC](media-services-specifications-ms-sstr-amendment-hevc.md)

# 資源
## [Azure 媒體服務社群](media-services-community.md)
## [Azure 藍圖](https://azure.microsoft.com/roadmap/?category=web-mobile)
## [定價](https://azure.microsoft.com/pricing/details/media-services/)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [版本資訊](media-services-release-notes.md)
## [影片](https://azure.microsoft.com/resources/videos/index/?services=media-services)
