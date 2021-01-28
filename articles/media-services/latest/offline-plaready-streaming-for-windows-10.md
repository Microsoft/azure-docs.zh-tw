---
title: 設定離線 PlayReady 串流
description: 本文說明如何設定 Azure 媒體服務 v3 帳戶，以供 Windows 10 離線的串流 PlayReady。
services: media-services
keywords: DASH, DRM, Widevine Offline Mode, ExoPlayer, Android, Widevine 離線模式
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: ea3d895970c6a18286d719471a693f774efaa387
ms.sourcegitcommit: 4e70fd4028ff44a676f698229cb6a3d555439014
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98955219"
---
# <a name="offline-playready-streaming-for-windows-10-with-media-services-v3"></a>使用媒體服務 v3 進行 Windows 10 的離線 PlayReady 串流

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure 媒體服務支援在具備 DRM 保護的情況下離線下載/播放。 本文涵蓋適用於 Windows 10/PlayRead 用戶端的 Azure 媒體服務離線支援。 您可以在以下文章中閱讀和適用於 iOS/FairPlay 與 Android/Widevine 裝置的離線模式支援相關資訊：

- [適用於 iOS 的離線 FairPlay 串流](offline-fairplay-for-ios.md)
- [適用于 Android 的離線 Widevine 串流](offline-widevine-for-android.md)

> [!NOTE]
> 離線 DRM 只會在您下載內容時，針對授權提出單一要求付費。 任何錯誤都不計費。

## <a name="overview"></a>概觀

本節提供一些離線模式播放的背景資訊，尤其是開發該技術的原因：

* 在某些國家/地區，網際網路可用性和/或頻寬仍有限制。 使用者可以選擇先下載，以便能以夠高的解析度觀賞內容，來獲得令人滿意的檢視體驗。 在此情況下，更常見的問題不是網路可用性，而是有限的網路頻寬。 OTT/OVP 提供者正在要求提供離線模式支援。
* Netflix 2016 年第 3 季股東會議中，Netflix CEO Reed Hastings 揭露了一項資訊，那就是下載內容是「頻繁被要求的功能」，並且「我們對此持開放態度」。
* 某些內容提供者可能不允許在國家/地區的邊界之外傳遞 DRM 授權。 如果使用者想要在需要出國旅行時仍能觀賞內容，就需要離線下載。
 
以下是我們在實作離線模式時面臨的挑戰：

* 許多播放器和編碼器工具支援 MP4，但 MP4 容器與 DRM 並未綁定，因此彼此之間沒有約束力；
* 長期來看，可以使用 CFF 搭配 CENC。 但是目前工具/播放器支援生態系統尚未成形。 而我們現在就需要一個解決方案。
 
這個概念是：將採用 H264/AAC 的 Smooth Streaming ([PIFF](/iis/media/smooth-streaming/protected-interoperable-file-format)) 檔案格式與 PlayReady (AES-128 CTR) 綁定，並產生約束力。 個別 Smooth Streaming .ismv 檔案 (假設影片中的音訊是多工的) 本身是 fMP4，而且可用於播放。 如果 Smooth Streaming 內容透過 PlayReady 加密，每個 .ismv 檔案都會變成受 PlayReady 保護的分散式 MP4。 我們可以選擇具備偏好位元速率的 .ismv 檔案，然後將它重新命名為 .mp4 以用於下載。

裝載受 PlayReady 保護的 MP4 以用於漸進式下載時有兩個選項可選擇：

* 一個是可以將此 MP4 放在相同的容器/媒體服務資產中，並利用 Azure 媒體服務串流端點進行漸進式下載；
* 另一個是使用 SAS 定位器直接從 Azure 儲存體進行漸進式下載，繞過 Azure 媒體服務。
 
您可以使用兩種類型的 PlayReady 授權傳遞服務：

* Azure 媒體服務中的 PlayReady 授權傳遞服務；
* 裝載在任何位置的 PlayReady 授權伺服器。

以下是兩組測試資產，第一個使用 AMS 中的 PlayReady 授權傳遞服務，另一個則使用我在 Azure VM 上裝載的 PlayReady 授權伺服器：

資產 1：

* 漸進式下載 URL： [https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (AMS)：`https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/`

資產 2：

* 漸進式下載 URL： [https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (內部部署) ： `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx`

針對播放測試，我們使用 Windows 10 上的通用 Windows 應用程式。 在 [Windows 10 通用範例](https://github.com/Microsoft/Windows-universal-samples)中，有一個名為[彈性資料流樣本](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/AdaptiveStreaming)的基本播放器樣本。 我們只需要自行新增程式碼，以選擇下載的視訊並使用它做為來源，而不是作為彈性資料流來源。 這些變更是在按鈕按下事件處理常式中：

```csharp
private async void LoadUri_Click(object sender, RoutedEventArgs e)
{
    //Uri uri;
    //if (!Uri.TryCreate(UriBox.Text, UriKind.Absolute, out uri))
    //{
    // Log("Malformed Uri in Load text box.");
    // return;
    //}
    //LoadSourceFromUriTask = LoadSourceFromUriAsync(uri);
    //await LoadSourceFromUriTask;

    //willzhan change start
    // Create and open the file picker
    FileOpenPicker openPicker = new FileOpenPicker();
    openPicker.ViewMode = PickerViewMode.Thumbnail;
    openPicker.SuggestedStartLocation = PickerLocationId.ComputerFolder;
    openPicker.FileTypeFilter.Add(".mp4");
    openPicker.FileTypeFilter.Add(".ismv");
    //openPicker.FileTypeFilter.Add(".mkv");
    //openPicker.FileTypeFilter.Add(".avi");

    StorageFile file = await openPicker.PickSingleFileAsync();

    if (file != null)
    {
       //rootPage.NotifyUser("Picked video: " + file.Name, NotifyType.StatusMessage);
       this.mediaPlayerElement.MediaPlayer.Source = MediaSource.CreateFromStorageFile(file);
       this.mediaPlayerElement.MediaPlayer.Play();
       UriBox.Text = file.Path;
    }
    else
    {
       // rootPage.NotifyUser("Operation cancelled.", NotifyType.ErrorMessage);
    }

    // On small screens, hide the description text to make room for the video.
    DescriptionText.Visibility = (ActualHeight < 500) ? Visibility.Collapsed : Visibility.Visible;
}
```

![受 PlayReady 保護 fMP4 的離線模式播放](./media/offline-playready-for-windows/offline-playready1.jpg)

由於影片受 PlayReady 保護，因此螢幕擷取畫面將無法包含影片。

總之，我們已經在 Azure 媒體服務上實現離線模式：

* 內容轉碼和 PlayReady 加密可以在 Azure 媒體服務或其他工具中完成；
* 內容可以裝載於 Azure 媒體服務或 Azure 儲存體中以進行漸進式下載；
* PlayReady 授權傳遞服務可以從 Azure 媒體服務或其他位置提供；
* 所準備的 Smooth Streaming 內容仍可用來透過 DASH 進行線上串流，或使用 PlayReady 以 DRM 內容的方式進行串流。

## <a name="next-steps"></a>下一步

[設計具有存取控制的多重 DRM 內容保護系統](design-multi-drm-system-with-access-control.md)
