---
title: 在 Azure 入口網站中發佈內容 | Microsoft Docs
description: 本教學課程將逐步引導您完成在 Azure 入口網站中發佈內容的步驟。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 92c364eb-5a5f-4f4e-8816-b162c031bb40
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2017
ms.author: juliako
ms.openlocfilehash: d93bfc548134f730f4fad49a37593c861d6b6cbb
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785767"
---
# <a name="publish-content-in-the-azure-portal"></a>在 Azure 入口網站中發佈內容
> [!div class="op_single_selector"]
> * [入口網站](media-services-portal-publish.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [REST](media-services-rest-deliver-streaming-content.md)
> 
> 

## <a name="overview"></a>概觀
> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 
> 

若要提供 URL 給使用者，讓使用者可以利用這個 URL 來傳送或下載內容，您必須先建立定位器來發佈您的資產。 定位器可供存取資產檔案。 Azure 媒體服務支援兩種類型的定位器： 

* **串流 (OnDemandOrigin) 定位器**。 串流定位器可用於彈性資料流。 彈性資料流的範例包括 Apple HTTP Live Streaming (HLS)、Microsoft Smooth Streaming 和 Dynamic Adaptive Streaming over HTTP (DASH，也稱為 MPEG-DASH)。 若要建立串流定位器，您的資產必須包含 .ism 檔案。 例如： http://amstest.streaming.mediaservices.windows.net/61b3da1d-96c7-489e-bd21-c5f8a7494b03/scott.ism/manifest。
* **漸進式 (共用存取簽章) 定位器**。 漸進式定位器用於透過漸進式下載傳遞影片。

若要建立 HLS 資料流 URL，請在 URL 加上 (format=m3u8-aapl)：

    {streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest(format=m3u8-aapl)

若要建置串流 URL 以播放 Smooth Streaming 資產，請使用下列 URL 格式：

    {streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest

若要建置 MPEG-DASH 串流 URL，請將 (format=mpd-time-csf) 附加至 URL：

    {streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest(format=mpd-time-csf)

共用存取簽章 URL 具有下列格式︰

    {blob container name}/{asset name}/{file name}/{shared access signature}

如需詳細資訊，請參閱[傳遞內容概觀](media-services-deliver-content-overview.md)。

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

## <a name="play-content-in-the-portal"></a>在入口網站中播放內容
您可以在 Azure 入口網站中的內容播放器上測試影片。

選取影片，然後選取 [播放] 按鈕。

![在 Azure 入口網站中播放影片](./media/media-services-portal-vod-get-started/media-services-play.png)

適用一些考量事項：

* 確定已發佈影片。
* Azure 入口網站媒體播放器會從預設串流端點播放。 如果您需要從非預設串流端點播放，請選取並複製 URL，然後將它貼到其他播放器中。 例如，您可以在 [Azure 媒體播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)上測試影片。
* 您正在進行串流的串流端點必須正在執行。  

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

