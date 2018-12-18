---
title: 開發視訊播放器應用程式
description: 本主題會提供 Player Framework 和外掛程式的連結，可讓您開發自己的用戶端應用程式，以使用來自媒體服務的串流媒體。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 55e419fc-4c39-4902-9c62-f41cfcd86c6c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: juliako
ms.openlocfilehash: 71d8b020fe96094c15965fd82615e3182e333990
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783457"
---
# <a name="develop-video-player-applications"></a>開發視訊播放器應用程式
## <a name="overview"></a>概觀
Azure 媒體服務提供一些工具，供您用來建立適用於大部分平台的豐富、動態用戶端播放器應用程式，此處所述的平台包括：iOS 裝置、Android 裝置、Windows、Windows Phone、Xbox 和機上盒。 本主題也會提供 SDK 和 Player Framework 連結，可讓您開發自己的用戶端應用程式，使用來自 Azure 媒體服務的串流媒體。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 
 
## <a name="azure-media-player"></a>Azure 媒體播放器
[Azure 媒體播放器](http://aka.ms/ampinfo) 是一款網頁視訊播放器，可以在各種瀏覽器和裝置上播放 Microsoft Azure 媒體服務的媒體內容。 Azure Media Player 採用業界標準，例如 HTML5、媒體來源延伸模組 (MSE) 和加密媒體擴充功能 (EME)，提供豐富的調適性串流體驗。 無法在裝置或瀏覽器使用這些標準時，Azure Media Player 則會使用 Flash 和 Silverlight 做為後援技術。 不論使用何種播放技術，開發人員都會有統一的 JavaScript 介面來存取應用程式開發介面。 這樣我們就可以在各種裝置和瀏覽器上順利播放 Azure 媒體服務提供的內容。

我們可以利用 Microsoft Azure 媒體服務播放 DASH、Smooth Streaming 和 HLS 資料流等格式的內容。 Azure Media Player 會考量這些不同的格式，並根據平台/瀏覽器功能自動播放最合適的連結。 Microsoft Azure 媒體服務也允許利用 PlayReady 加密或 AES 128 位元信封加密，進行資產的動態加密。 只要設定正確，Azure Media Player 允許解密 PlayReady 和 AES 128 位元加密的內容。 

其他資訊：

* [Azure Media Player](http://aka.ms/ampinfo)
* [Azure Media Player 文件](http://aka.ms/ampdocs) 
* [Azure Media Player 開始使用部落格](https://azure.microsoft.com/blog/2015/04/15/announcing-azure-media-player/)
* [註冊以持續收到 Azure Media Player 的最新消息](http://aka.ms/ampsignup)
* [新增功能要求、概念和意見反應](http://aka.ms/ampuservoice) 

## <a name="other-tools-for-creating-player-applications"></a>其他可用來建立播放器應用程式的工具
您也可以使用任何下列 SDK：

* [Smooth Streaming Client SDK](http://www.iis.net/downloads/microsoft/smooth-streaming) 
* [Smooth Streaming Windows 市集應用程式](media-services-build-smooth-streaming-apps.md)
* [Microsoft 媒體平台：Player Framework](http://playerframework.codeplex.com/) 
* [HTML5 Player Framework 文件](http://playerframework.codeplex.com/wikipage?title=HTML5%20Player&referringTitle=Documentation) 
* [Microsoft Smooth Streaming Plugin for OSMF](https://www.microsoft.com/download/details.aspx?id=36057) 
* [Licensing Microsoft® Smooth Streaming Client Porting Kit](http://aka.ms/sspk) 
* [XBOX Video Application Development](http://xbox.create.msdn.com/) 

## <a name="advertising"></a>廣告
Azure 媒體服務允許透過 Windows Media 平台插入廣告：Player Framework。 具備廣告支援的播放器架構都適用於 Windows 8、Silverlight、Windows Phone 8 和 iOS 裝置。 每一個播放器架構都有範例程式碼，教您如何實作播放器應用程式。 目前有三種不同的廣告可以插入媒體中：

線性 – 可暫停主要影片的完整框架廣告

非線性 – 播放被當做主要影片而顯示的疊加廣告，通常是播放器中的標誌或其他靜態影像

隨播 – 顯示在播放器之外的廣告

廣告可以放在主要影片時間軸的任何時間點。 您必須告訴播放器何時播放廣告以及要播放哪些廣告。 使用一組標準的 XML 格式檔案即可搞定：Video Ad Service Template (VAST)、Digital Video Multiple Ad Playlist (VMAP)、Media Abstract Sequencing Template (MAST) 以及 Digital Video Player Ad Interface Definition (VPAID)。 VAST 檔案會指定要顯示的廣告。 VMAP 檔案會指定何時播放各種廣告而且包含 VAST XML。 MAST 檔案是另一種廣告排序的方法，而且也包含 VAST XML。 VPAID 檔案會定義影片播放器廣告和廣告或廣告伺服器之間的介面。 如需詳細資訊，請參閱 [插入廣告](https://msdn.microsoft.com/library/dn387398.aspx)。

如需了解即時資料流視訊的隱藏式字幕和廣告支援，請參閱 [支援的隱藏式輔助字幕和廣告插入標準](https://msdn.microsoft.com/library/c49e0b4d-357e-4cca-95e5-2288924d1ff3#caption_ad)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[透過 DASH.js 將 MPEG-DASH 彈性資料流視訊嵌入到 HTML5 應用程式](media-services-embed-mpeg-dash-in-html5.md)

[GitHub dash.js 存放庫](https://github.com/Dash-Industry-Forum/dash.js)

