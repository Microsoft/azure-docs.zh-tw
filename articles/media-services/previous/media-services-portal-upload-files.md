---
title: 在 Azure 入口網站中將檔案上傳至媒體服務帳戶 | Microsoft Docs
description: 本教學課程逐步引導您完成在 Azure 入口網站中將檔案上傳至媒體服務帳戶的步驟。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 3ad3dcea-95be-4711-9aae-a455a32434f6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/07/2017
ms.author: juliako
ms.openlocfilehash: 36e1f797263e367a73fde140d979243f96e83948
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33780546"
---
# <a name="upload-files-to-a-media-services-account-in-the-azure-portal"></a>在 Azure 入口網站中將檔案上傳至媒體服務帳戶
> [!div class="op_single_selector"]
> * [入口網站](media-services-portal-upload-files.md)
> * [.NET](media-services-dotnet-upload-files.md)
> * [REST](media-services-rest-upload-files.md)
> 
> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 

在 Azure 媒體服務中，您會將數位檔案上傳到到資產。 資產可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式輔助字幕檔案 (以及這些檔案的中繼資料)。 上傳檔案之後，您的內容會安全地儲存在雲端，以進一步進行處理和串流處理。

> [!NOTE]
> 媒體服務有處理檔案的檔案大小上限。 如需有關檔案大小限制的詳細資訊，請參閱[媒體服務配額和限制](media-services-quotas-and-limitations.md)。
>

## <a name="upload-files"></a>上傳檔案
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 選取 **設定** > **資產**。 然後，選取 [上傳] 按鈕。
   
    ![上傳檔案](./media/media-services-portal-vod-get-started/media-services-upload.png)
   
    [上傳視訊資產]  視窗隨即出現。
   
   > [!NOTE]
   > 媒體服務不會限制上傳影片的檔案大小。
 
3. 在您的電腦上，移至您要上傳的影片。 選取影片，然後選取 [確定]。  
   
    隨即開始上傳。 您可以在檔名底下看到進度。  

上傳完成後，新資產會列在 [資產] 窗格中。 

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
* 了解如何[將上傳的資產編碼](media-services-portal-encode.md)。

* 您也可以使用 Azure Functions，以在檔案到達所設定的容器時觸發編碼作業。 如需詳細資訊，請參閱[媒體服務：整合 Microsoft Azure 媒體服務與 Azure Functions 和 Logic Apps](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/) 中的範例。


