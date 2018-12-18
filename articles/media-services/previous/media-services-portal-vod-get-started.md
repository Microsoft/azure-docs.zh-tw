---
title: 使用 Azure 入口網站開始傳遞點播視訊 | Microsoft Docs
description: 本教學課程會逐步引導您在 Azure 入口網站中透過 Azure 媒體服務 (AMS) 應用程式實作基本點播視訊內容傳遞服務。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 6c98fcfa-39e6-43a5-83a5-d4954788f8a4
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/07/2017
ms.author: juliako
ms.openlocfilehash: 11babf8a66c38354499ce85fad424fed04c07c15
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33780597"
---
# <a name="get-started-with-delivering-content-on-demand-by-using-the-azure-portal"></a>使用 Azure 入口網站開始傳遞點播內容
[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

本教學課程會逐步引導您在 Azure 入口網站中透過 Azure 媒體服務 (AMS) 應用程式實作基本點播視訊內容傳遞服務。

## <a name="prerequisites"></a>先決條件
需要有下列項目，才能完成教學課程：

* 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。

本教學課程內容包括以下工作：

1. 啟動串流端點。
2. 上傳視訊檔案。
3. 將來源檔案編碼為一組自適性 MP4 檔案。
4. 發佈資產並取得串流和漸進式下載 URL。  
5. 播放您的內容。

## <a name="start-the-streaming-endpoint"></a>啟動串流端點

使用 Azure 媒體服務時，其中一個最常見的案例就是透過自適性串流傳遞影片。 媒體服務為您提供動態封裝。 使用動態封裝時，您可以用媒體服務所支援的 just-in-time 串流格式來提供自適性 MP4 編碼的內容。 範例包括 Apple HTTP Live Streaming (HLS)、Microsoft Smooth Streaming 和 Dynamic Adaptive Streaming over HTTP (DASH，也稱為 MPEG-DASH)。 藉由使用媒體服務自適性串流，您可以傳遞影片，但不需儲存每種串流格式的預先封裝版本。

> [!NOTE]
> 建立 Azure 媒體服務帳戶時，預設串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您要串流內容的串流端點必須處於 [執行中] 狀態。 

若要啟動串流端點：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 選取 [設定] > [串流端點]。 
3. 選取預設串流端點。 [預設串流端點詳細資料]  視窗隨即出現。
4. 選取 [啟動] 圖示。
5. 選取 [儲存] 按鈕。

## <a name="upload-files"></a>上傳檔案
若要使用媒體服務串流處理視訊，請上傳來源影片、將它們編碼成多個位元速率，以及發佈結果。 這一節涵蓋第一個步驟。 

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 選取 **設定** > **資產**。 然後，選取 [上傳] 按鈕。
   
    ![上傳檔案](./media/media-services-portal-vod-get-started/media-services-upload.png)
   
    [上傳視訊資產]  視窗隨即出現。
   
   > [!NOTE]
   > 媒體服務不會限制上傳影片的檔案大小。
   > 
   > 
3. 在您的電腦上，移至您要上傳的影片。 選取影片，然後選取 [確定]。  
   
    隨即開始上傳。 您可以在檔名底下看到進度。  

上傳完成後，新資產會列在 [資產] 窗格中。 

## <a name="encode-assets"></a>為資產編碼
若要利用動態封裝功能，您必須將來源檔案編碼成一組多位元速率的 MP4 檔案。 本節會示範編碼步驟。

### <a name="encode-assets-in-the-portal"></a>在入口網站中為資產編碼
在 Azure 入口網站中使用媒體編碼器標準為資產編碼：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 選取 **設定** > **資產**。 選取您要編碼的資產。
3. 選取 [編碼] 按鈕。
4. 在 [為資產編碼] 窗格中，選取 [媒體編碼器標準] 處理器和預設值。 如需預設值的相關資訊，請參閱[自動產生位元速率排行榜](media-services-autogen-bitrate-ladder-with-mes.md)和[媒體編碼器標準的工作預設值](media-services-mes-presets-overview.md)。 請務必選擇最適合輸入影片的預設值。 例如，如果您知道您的輸入視訊的解析度為 1920 &#215; 1080 像素，您可選擇 [H264 多位元速率 1080p] 預設值。 如果您有低解析度 (640 &#215; 360) 影片，您不應該使用 [H264 多重位元速率 1080p] 預設值。
   
   為了協助您管理資源，您可以編輯輸出資產的名稱和作業的名稱。
   
   ![為資產編碼](./media/media-services-portal-vod-get-started/media-services-encode1.png)
5. 選取 [建立] 。

### <a name="monitor-encoding-job-progress"></a>監視編碼作業進度
若要監視編碼作業的進度，請按一下 [設定]，然後選取 [作業]。

![工作](./media/media-services-portal-vod-get-started/media-services-jobs.png)

## <a name="publish-content"></a>發佈內容
若要提供 URL 給使用者，讓使用者可以利用這個 URL 來傳送或下載內容，您必須先建立定位器來發佈您的資產。 定位器可供存取資產中的檔案。 Azure 媒體服務支援兩種類型的定位器： 

* **串流 (OnDemandOrigin) 定位器**。 串流定位器可用於彈性資料流。 彈性資料流的範例包括 HLS、Smooth Streaming 和 MPEG-DASH。 若要建立串流定位器，您的資產必須包含 .ism 檔案。 
* **漸進式 (共用存取簽章) 定位器**。 漸進式定位器用於透過漸進式下載傳遞影片。

若要建立 HLS 資料流 URL，請在 URL 加上 (format=m3u8-aapl)：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{file name}.ism/Manifest(format=m3u8-aapl)

若要建置串流 URL 以播放 Smooth Streaming 資產，請使用下列 URL 格式：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{file name}.ism/Manifest

若要建置 MPEG-DASH 串流 URL，請將 (format=mpd-time-csf) 附加至 URL：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{file name}.ism/Manifest(format=mpd-time-csf)

共用存取簽章 URL 具有下列格式︰

    {blob container name}/{asset name}/{file name}/{shared access signature}

> [!NOTE]
> 在 2015 年 3 月之前，在 Azure 入口網站中建立的定位器有兩年的到期日。  
> 
> 

若要更新定位器的到期日，您可以使用 [REST API](https://docs.microsoft.com/rest/api/media/operations/locator#update_a_locator) 或 [.NET API](http://go.microsoft.com/fwlink/?LinkID=533259)。 

> [!NOTE]
> 當您更新共用存取簽章定位器的到期日時，URL 也會隨之變更。

### <a name="to-use-the-portal-to-publish-an-asset"></a>使用入口網站發佈資產
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 選取 [釘選到儀表板]  > 。 選取您要發佈的資產。
3. 選取 [發佈] 按鈕。
4. 選取定位器類型。
5. 選取 [新增] 。
   
    ![發佈影片](./media/media-services-portal-vod-get-started/media-services-publish1.png)

URL 便會加入 [已發佈的 URL] 清單中。

## <a name="play-content-from-the-portal"></a>從入口網站播放內容
您可以在 Azure 入口網站中的內容播放器上測試影片。

選取影片，然後選取 [播放] 按鈕。

![在 Azure 入口網站中播放影片](./media/media-services-portal-vod-get-started/media-services-play.png)

適用一些考量事項：

* 若要開始串流，請開始執行預設串流端點。
* 確定已發佈影片。
* Azure 入口網站媒體播放器會從預設串流端點播放。 如果您需要從非預設串流端點播放，請選取並複製 URL，然後將它貼到其他播放器中。 例如，您可以在 [Azure 媒體播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)上測試影片。

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
