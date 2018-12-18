---
title: 使用現有的撥放器來撥放內容 - Azure | Microsoft Docs
description: 本主題列出現有的播放程式，您可以使用來播放您的內容。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 7e9fcf89-0fb6-4fa4-96cb-666320684d69
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/02/2018
ms.author: juliako
ms.openlocfilehash: 3fe82b98163182c73a144b72da371e8aa195e8cf
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37435852"
---
# <a name="playing-your-content-with-existing-players"></a>使用現有播放器來播放您的內容
Azure 媒體服務支援許多熱門的串流格式，例如 Smooth Streaming、HTTP 即時資料流和 MPEG-Dash。 本主題會指引您可用來測試串流的現有播放程式。

### <a name="the-azure-portal-media-services-content-player"></a>Azure 入口網站媒體服務內容播放程式
**Azure** 入口網站提供內容播放程式，您可用來測試您的視訊。

按一下想用的視訊 (請確定它 [已發行](media-services-portal-publish.md))，按一下入口網站底部的 [ **播放** ] 按鈕。

適用一些考量事項：

* **媒體服務內容播放程式** 會從預設串流端點播放。 如果您想要從非預設串流端點播放，請使用其他播放程式。 例如， [Azure 媒體播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)。

![AMSPlayer][AMSPlayer]

### <a name="azure-media-player"></a>Azure 媒體播放器
使用 [Azure 媒體播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html) 播放以下任一格式的內容 (清除或受保護)：

* Smooth Streaming
* MPEG DASH
* HLS
* Progressive MP4

### <a name="flash-player"></a>Flash Player
#### <a name="aes-encrypted-with-token"></a>AES 加密與權杖
[http://aestoken.azurewebsites.net](http://aestoken.azurewebsites.net)

### <a name="silverlight-players"></a>Silverlight 播放程式

#### <a name="playready-with-token"></a>PlayReady 與權杖
[http://sltoken.azurewebsites.net](http://sltoken.azurewebsites.net)

### <a name="dash-players"></a>DASH 播放程式
[http://dashplayer.azurewebsites.net](http://dashplayer.azurewebsites.net)

[http://dashif.org](http://dashif.org)

### <a name="other"></a>其他
若要測試 HLS URL，您也可以使用：

* **Safari** 或
* **3ivx HLS 播放器** 。

## <a name="developing-video-players"></a>開發視訊播放器
如需瞭解如何自行開發播放程式，請參閱 [開發視訊播放器](media-services-develop-video-players.md)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[AMSPlayer]: ./media/media-services-playback-content-with-existing-players/media-services-portal-player.png
