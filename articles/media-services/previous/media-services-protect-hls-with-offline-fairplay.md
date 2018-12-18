---
title: 使用離線 Apple FairPlay 保護 HLS 內容 - Azure | Microsoft Docs
description: 本主題提供概觀，並示範如何使用 Azure 媒體服務，利用離線模式的 Apple FairPlay 動態加密您的 HTTP 即時串流 (HLS) 內容。
services: media-services
keywords: HLS, DRM, FairPlay Streaming (FPS), Offline, iOS 10, FairPlay 串流 (FPS), 離線
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/01/2017
ms.author: willzhan, dwgeo
ms.openlocfilehash: dc38772097dddb7c7135d55598373d7ab544f9ea
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785887"
---
# <a name="offline-fairplay-streaming-for-ios"></a>適用於 iOS 的離線 FairPlay 串流 
 Azure 媒體服務提供一組設計良好的[內容保護服務](https://azure.microsoft.com/services/media-services/content-protection/)，其中涵蓋：

- Microsoft PlayReady (英文)
- Google Widevine
- Apple FairPlay
- AES-128 加密

系統會根據各種不同串流通訊協定的要求，動態地對內容執行數位版權管理 (DRM)/進階加密標準 (AES) 加密。 媒體服務也會提供 DRM 授權/AES 解密金鑰傳遞服務。

除了透過各種不同的串流處理通訊協定來保護內容進行線上串流處理，受保護內容的離線模式也是一個經常被要求的功能。 下列案例需要離線模式支援：

* 在無法使用網際網路連線 (例如旅行期間) 時播放。
* 某些內容提供者可能不允許超出國家/地區的邊界來傳遞 DRM 授權。 如果使用者要在其國家/地區外旅行時觀賞內容，就需要離線下載。
* 在某些國家/地區，網際網路可用性和 (或) 頻寬仍然有限。 使用者可選擇先下載內容，才能觀賞解析度夠高的內容，從而獲得滿意的觀賞體驗。 因此，問題通常不在於網路可用性，而是網路頻寬有限。 Over-the-top (OTT)/線上視訊平台 (OVP) 提供者會要求離線模式支援。

本文會說明 FairPlay 串流 (FPS) 離線模式支援，此支援是以執行 iOS 10 或更新版本的裝置為目標。 其他 Apple 平台 (例如 watchOS、tvOS 或 macOS 上的 Safari) 則不支援此功能。

## <a name="preliminary-steps"></a>預備步驟
在 iOS 10+ 裝置上實作適用於 FairPlay 的離線 DRM 之前：

* 熟悉適用於 FairPlay 的線上內容保護。 如需詳細資訊，請參閱下列文章和範例：

    - [適用於 Azure 媒體服務的 Apple FairPlay 串流已正式推出](https://azure.microsoft.com/blog/apple-FairPlay-streaming-for-azure-media-services-generally-available/)
    - [使用 Apple FairPlay 或 Microsoft PlayReady 保護 HLS 內容](https://docs.microsoft.com/azure/media-services/media-services-protect-hls-with-FairPlay)
    - [線上 FPS 串流的範例](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-FairPlay/) \(英文\)

* 從 Apple 開發人員網路取得 FPS SDK。 FPS SDK 包含兩個元件：

    - FPS Server SDK，內含金鑰安全性模組 (KSM)、用戶端範例、規格，以及一組測試向量。
    - FPS Deployment Pack，內含 D 函式規格，以及有關如何產生 FPS 憑證、客戶專屬私密金鑰及應用程式祕密金鑰的指示。 Apple 只會將 FPS Deployment Pack 發給授權的內容提供者。

## <a name="configuration-in-media-services"></a>在媒體服務中設定
若要透過[媒體服務 .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices) 進行 FPS 離線模式設定，請使用媒體服務 .NET SDK 4.0.0.4 版或更新版本，其可提供設定 FPS 離線模式所需的 API。
您還需要工作程式碼，才能設定線上模式 FPS 內容保護。 在取得此程式碼來設定 FPS 線上模式內容保護之後，您只需進行下列兩項變更。

## <a name="code-change-in-the-fairplay-configuration"></a>FairPlay 組態中的程式碼變更
第一項變更是定義一個稱為 objDRMSettings.EnableOfflineMode 的「啟用離線模式」布林值，若能夠實現離線 DRM 案例，此值會是 true。 根據這個指標，對 FairPlay 組態進行下列變更：

```csharp
if (objDRMSettings.EnableOfflineMode)
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv, 
            RentalAndLeaseKeyType.PersistentUnlimited,
            0x9999);
    }
    else
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv);
    }
```

## <a name="code-change-in-the-asset-delivery-policy-configuration"></a>資產傳遞原則組態 中的程式碼變更
第二項變更是在 Dictionary<AssetDeliveryPolicyConfigurationKey, string> 中新增第三個索引鍵。
新增 AssetDeliveryPolicyConfigurationKey，如下所示：
 
```csharp
// FPS offline mode
    if (drmSettings.EnableOfflineMode)
    {
        objDictionary_AssetDeliveryPolicyConfigurationKey.Add(AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense, "true");
        Console.WriteLine("FPS OFFLINE MODE: AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense added into asset delivery policy configuration.");
    }

    // for IAssetDelivery for FPS
    IAssetDeliveryPolicy objIAssetDeliveryPolicy = objCloudMediaContext.AssetDeliveryPolicies.Create(
            drmSettings.AssetDeliveryPolicyName,
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            objDictionary_AssetDeliveryPolicyConfigurationKey);
```

此步驟完成後，FPS 資產傳遞原則中的 <Dictionary_AssetDeliveryPolicyConfigurationKey> 字串會包含下列三個項目：

* AssetDeliveryPolicyConfigurationKey.FairPlayBaseLicenseAcquisitionUrl 或 AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl (取決因素包括所使用的 FPS KSM/金鑰伺服器，以及您是否跨多個資產重複使用相同的資產傳遞原則等)
* AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs
* AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense

現在，您的媒體服務帳戶已設定為會提供離線 FairPlay 授權。

## <a name="sample-ios-player"></a>範例 iOS 播放程式
FPS 離線模式支援僅適用於 iOS 10 和更新版本。 FPS Server SDK (3.0 版或更新版本) 包含有關於 FPS 離線模式的文件和範例。 具體來說，FPS Server SDK (3.0 版或更新版本) 包含下列兩個與離線模式相關的項目：

* 文件：《Offline Playback with FairPlay Streaming and HTTP Live Streaming》(使用 FairPlay 串流和 HTTP 即時串流進行離線播放)。 Apple，2016 年 9 月 14 日發行。 在 FPS Server SDK 4.0 版中，這份文件會合併到主要 FPS 文件中。
* 程式碼範例：FPS 離線模式的 HLSCatalog 範例，位於 \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\。 HLSCatalog 應用程式範例會使用下列程式碼檔案來實作離線模式功能：

    - AssetPersistenceManager.swift 程式碼檔案：AssetPersistenceManager 是此範例中的主類別，會示範如何進行下列操作：

        - 管理下載 HLS 串流 (例如，用來啟動和取消下載，以及用來從裝置中刪除現有資產的 API)。
        - 監視下載進度。
    - AssetListTableViewController.swift 和 AssetListTableViewCell.swift 程式碼檔案：AssetListTableViewController 是此範例的主要介面。 它提供可供範例播放、下載、刪除或取消下載的資產清單。 

這些步驟會說明如何設定執行中的 iOS 播放程式。 假設您是從 FPS Server SDK 4.0.1 版中的 HLSCatalog 範例開始操作，請進行下列程式碼變更：

在 HLSCatalog\Shared\Managers\ContentKeyDelegate.swift 中，使用下列程式碼實作 `requestContentKeyFromKeySecurityModule(spcData: Data, assetID: String)` 方法。 將 "drmUr" 設為指派給 HLS URL 的變數。

```swift
    var ckcData: Data? = nil
    
    let semaphore = DispatchSemaphore(value: 0)
    let postString = "spc=\(spcData.base64EncodedString())&assetId=\(assetIDString)"
    
    if let postData = postString.data(using: .ascii, allowLossyConversion: true), let drmServerUrl = URL(string: self.drmUrl) {
        var request = URLRequest(url: drmServerUrl)
        request.httpMethod = "POST"
        request.setValue(String(postData.count), forHTTPHeaderField: "Content-Length")
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        request.httpBody = postData
        
        URLSession.shared.dataTask(with: request) { (data, _, error) in
            if let data = data, var responseString = String(data: data, encoding: .utf8) {
                responseString = responseString.replacingOccurrences(of: "<ckc>", with: "").replacingOccurrences(of: "</ckc>", with: "")
                ckcData = Data(base64Encoded: responseString)
            } else {
                print("Error encountered while fetching FairPlay license for URL: \(self.drmUrl), \(error?.localizedDescription ?? "Unknown error")")
            }
            
            semaphore.signal()
            }.resume()
    } else {
        fatalError("Invalid post data")
    }
    
    semaphore.wait()
    return ckcData
```

在 HLSCatalog\Shared\Managers\ContentKeyDelegate.swift 中，實作方法 `requestApplicationCertificate()`。 此實作取決於您會將憑證內嵌於裝置上 (僅限公開金鑰)，還是將憑證裝載於 Web 上。 下列實作會使用測試範例中所用的已裝載應用程式憑證。 將 "certUrl" 設為包含應用程式憑證 URL 的變數。

```swift
func requestApplicationCertificate() throws -> Data {

        var applicationCertificate: Data? = nil
        do {
            applicationCertificate = try Data(contentsOf: URL(string: certUrl)!)
        } catch {
            print("Error loading FairPlay application certificate: \(error)")
        }
        
        return applicationCertificate
    }
```

最終整合測試的影片 URL 和應用程式憑證 URL 都會在＜整合式測試＞一節中提供。

請在 HLSCatalog\Shared\Resources\Streams.plist 中新增您的測試影片 URL。 至於內容金鑰識別碼，請使用 FairPlay 授權取得 URL 與 skd 通訊協定作為唯一值。

![離線 FairPlay iOS 應用程式串流](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-ios-app-streams.png)

請使用您自己的測試影片 URL、FairPlay 授權取得 URL 和應用程式憑證 URL (如果您已設定的話)。 您也可以繼續前往下一節，其內含測試範例。

## <a name="integrated-test"></a>整合式測試
媒體服務中的三個測試範例可涵蓋下列三種案例：

* 受到 FPS 保護，具備影片、音訊及替代曲目
* 受到 FPS 保護，具備影片和音訊，但不具替代曲目
* 受到 FPS 保護，僅具備影片，而不具音訊

您可以在[這個示範網站](http://aka.ms/poc#22) (英文) 找到這些範例，以及裝載於 Azure Web 應用程式中的對應應用程式憑證。
對於 FPS Server SDK 第 3 版或第 4 版的範例，如果主要播放清單包含替代音訊，則在離線模式期間，它只會播放音訊。 因此，您必須移除替代音訊。 也就是說，前面所列的第二個和第三個範例會在線上及離線模式下運作。 所列出的第一個範例在離線模式期間只會播放音訊，然而線上串流會正常運作。

## <a name="faq"></a>常見問題集
下列常見問題集可協助您進行疑難排解：

- **為何在離線模式期間只播放音訊，但不會播放影片呢？** 這個行為似乎是範例應用程式的設計使然。 當離線模式期間存在替代曲目時 (HLS 便是如此)，iOS 10 和 iOS 11 都預設為播放替代曲目。為了彌補 FPS 離線模式的這項行為，請從串流中移除替代曲目。 若要對媒體服務執行此操作，請新增動態資訊清單篩選條件 "audio-only=false"。 換言之，HLS URL 的結尾是 .ism/manifest(format=m3u8-aapl,audio-only=false)。 
- **當我新增 audio-only=false 之後，為什麼它在離線模式期間仍舊只播放音訊，而不會播放影片呢？** 系統可能會根據內容傳遞網路 (CDN) 快取索引鍵設計來快取內容。 所以，請清除快取。
- **除了 iOS 10，iOS 11 也支援 FPS 離線模式嗎？** 是。 iOS 10 和 iOS 11 均可支援 FPS 離線模式。
- **為什麼我在 FPS Server SDK 中找不到《Offline Playback with FairPlay Streaming and HTTP Live Streaming》(使用 FairPlay 串流和 HTTP 即時串流進行離線播放) 文件呢？** 從 FPS Server SDK 第 4 版開始，此文件已合併至《FairPlay Streaming Programming Guide》(FairPlay 串流程式設計指南)。
- **在以下適用於 FPS 離線模式的 API 中，最後一個參數代表什麼？**
`Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(objX509Certificate2, pfxPassword, pfxPasswordId, askId, iv, RentalAndLeaseKeyType.PersistentUnlimited, 0x9999);`

    如需此 API 的文件，請參閱 [FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration 方法](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.mediaservices.client.FairPlay.FairPlayconfiguration.createserializedFairPlayoptionconfiguration?view=azure-dotnet)。 該參數代表以小時為單位的離線租用持續時間。
- **在 iOS 裝置上的下載/離線檔案結構為何？** iOS 裝置上所下載的檔案結構看起來如下列螢幕擷取畫面。 `_keys` 資料夾會儲存下載的 FPS 授權，每個授權服務主機一個存放區檔案。 `.movpkg` 資料夾會儲存音訊和影片內容。 第一個資料夾名稱的結尾是一個破折號後面接著一個數字，此資料夾包含影片內容。 數值為影片轉譯的 PeakBandwidth。 第二個資料夾名稱的結尾是一個破折號後面接著 0，此資料夾包含音訊內容。 第三個名為 "Data" 的資料夾包含 FPS 內容的主要播放清單。 最後，boot.xml 會提供 `.movpkg` 資料夾內容的完整描述。 

![離線 FairPlay iOS 範例應用程式檔案結構](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-file-structure.png)

範例 boot.xml 檔案：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

## <a name="summary"></a>總結
本文件包含下列步驟和資訊，以供您實作 FPS 離線模式：

* 透過媒體服務 .NET API 所進行的媒體服務內容保護設定，會在媒體服務中設定動態 FairPlay 加密和 FairPlay 授權傳遞。
* 以 FPS Server SDK 所提供的範例為基礎的 iOS 播放程式，會設定可在線上串流模式或離線模式下播放 FPS 內容的 iOS 播放程式。
* 範例 FPS 影片，可用來測試離線模式和線上串流。
* 常見問題集，可解答有關 FPS 離線模式的問題。
