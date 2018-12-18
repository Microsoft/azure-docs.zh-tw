---
title: 在 Azure 入口網站中使用媒體編碼器標準為資產編碼 | Microsoft Docs
description: 本教學課程將逐步引導您完成在 Azure 入口網站中使用媒體編碼器標準為資產編碼的步驟。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 107d9e9a-71e9-43e5-b17c-6e00983aceab
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2017
ms.author: juliako
ms.openlocfilehash: 470eb8613416f441c1becee628acf3c898591c84
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785647"
---
# <a name="encode-an-asset-by-using-media-encoder-standard-in-the-azure-portal"></a>在 Azure 入口網站中使用媒體編碼器標準為資產編碼

> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 
> 

使用 Azure 媒體服務時，其中一個最常見案例是提供自適性串流給您的用戶端。 媒體服務支援下列自適性串流技術：Apple HTTP Live Streaming (HLS)、Microsoft Smooth Streaming 和 Dynamic Adaptive Streaming over HTTP (DASH，也稱為 MPEG-DASH)。 若要針對自適性串流準備您的視訊，請先將來源視訊編碼為多位元速率檔案。 您可以使用 Azure 媒體編碼器標準來編碼視訊。  

媒體服務為您提供動態封裝。 透過動態封裝，您可以利用 HLS、Smooth Streaming 和 MPEG-DASH 傳遞多位元速率 MP4，而不必以這些串流格式重新封裝。 當您使用動態封裝時，您可以儲存及播放單一儲存格式的檔案。 媒體服務會根據用戶單的要求建置及傳遞適當的回應。

若要利用動態封裝功能，您必須將來源檔案編碼成一組多位元速率的 MP4 檔案。 本文稍後會示範編碼步驟。

若要了解如何調整媒體處理，請參閱[使用 Azure 入口網站調整媒體處理](media-services-portal-scale-media-processing.md)。

## <a name="encode-in-the-azure-portal"></a>在 Azure 入口網站中編碼

若要使用媒體編碼器標準為內容編碼：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 選取 **設定** > **資產**。 選取您要編碼的資產。
3. 選取 [編碼] 按鈕。
4. 在 [為資產編碼] 窗格中，選取 [媒體編碼器標準] 處理器和預設值。 如需預設值的相關資訊，請參閱[自動產生位元速率排行榜](media-services-autogen-bitrate-ladder-with-mes.md)和[媒體編碼器標準的工作預設值](media-services-mes-presets-overview.md)。 請務必選擇最適合輸入影片的預設值。 例如，如果您知道您的輸入視訊的解析度為 1920 &#215; 1080 像素，您可選擇 [H264 多位元速率 1080p] 預設值。 如果您有低解析度 (640 &#215; 360) 影片，您不應該使用 [H264 多重位元速率 1080p] 預設值。
   
   為了協助您管理資源，您可以編輯輸出資產的名稱和作業的名稱。
   
   ![為資產編碼](./media/media-services-portal-vod-get-started/media-services-encode1.png)
5. 選取 [建立] 。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
* 在 Azure 入口網站中[監視編碼作業的進度](media-services-portal-check-job-progress.md)。  

