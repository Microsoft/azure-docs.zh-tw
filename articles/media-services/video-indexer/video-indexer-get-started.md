---
title: 註冊 Video Indexer 並上傳您的第一支影片 - Azure
titleSuffix: Azure Media Services
description: 了解如何使用 Video Indexer 入口網站，註冊及上傳您的第一支影片。
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: quickstart
ms.date: 01/25/2021
ms.author: juliako
ms.openlocfilehash: 5b38c731db141052e6700472020cd60b6a4d13a5
ms.sourcegitcommit: 95c2cbdd2582fa81d0bfe55edd32778ed31e0fe8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98797791"
---
# <a name="quickstart-how-to-sign-up-and-upload-your-first-video"></a>快速入門：如何註冊及上傳您的第一支影片

本快速入門示範如何登入 Video Indexer 網站，以及如何上傳您的第一支影片。

建立影片索引器帳戶時，您可以選擇免費試用帳戶 (您可取得特定的免費編製索引分鐘數) 或付費選項 (您不會受限於配額)。 使用免費試用時，影片索引器最多可為網站使用者提供 600 分鐘的免費編製索引，以及為 API 使用者提供 2400 分鐘的免費索引編製。 使用付費選項時，您建立的影片索引器帳戶會[連線到您的 Azure 訂用帳戶和 Azure 媒體服務帳戶](connect-to-azure.md)。 您需支付已編製索引的分鐘數，如需詳細資訊，請參閱[媒體服務定價](https://azure.microsoft.com/pricing/details/media-services/)。 

## <a name="sign-up-for-video-indexer"></a>註冊 Video Indexer

若要開始使用 Video Indexer 進行開發，請瀏覽至 [Video Indexer](https://www.videoindexer.ai/) 網站並註冊。

一旦您開始使用影片索引子，所有儲存的資料和上傳的內容都會使用 Microsoft 管理的金鑰進行待用加密。

> [!NOTE]
> 檢閱[規劃的影片索引子網站驗證變更](release-notes.md#planned-video-indexer-website-authenticatication-changes)。

## <a name="upload-a-video-using-the-video-indexer-website"></a>使用 Video Indexer 網站上傳影片

### <a name="supported-file-formats-for-video-indexer"></a>影片索引子支援的檔案格式

如需可與影片索引子搭配使用的檔案格式清單，請參閱[輸入容器/檔案格式](../latest/media-encoder-standard-formats.md#input-containerfile-formats)一文。

### <a name="upload-a-video"></a>上傳影片

1. 登入 [Video Indexer](https://www.videoindexer.ai/) 網站。
1. 若要上傳影片，請按 [上傳]  按鈕或連結。

    > [!NOTE]
    > 影片名稱不得超過 80 個字元。

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/video-indexer-upload.png" alt-text="上傳":::
1. 上傳您的影片後，Video Indexer 會開始編製索引及分析影片。 您會看到進度。 

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/progress.png" alt-text="上傳的進度":::
1. Video Indexer 完成分析後，您會收到電子郵件，內含您的影片連結以及在影片中找到的簡短描述。 例如：人員、讀出和書寫的單字、主題和已命名的實體。
1. 您稍後可以在媒體庫清單中找到您的影片，並執行不同的作業。 例如：搜尋、重新編制索引、編輯。

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/uploaded.png" alt-text="已上傳上傳":::

## <a name="supported-browsers"></a>支援的瀏覽器

如需詳細資訊，請參閱 [支援的瀏覽器](video-indexer-overview.md#supported-browsers)。

## <a name="see-also"></a>請參閱

如需詳細資訊，請參閱[上傳影片並編製索引](upload-index-videos.md)。

上傳影片並編製索引後，您可以開始使用 [Video Indexer](video-indexer-view-edit.md) 網站或 [Video Indexer 開發人員入口網站](video-indexer-use-apis.md)來查看影片的深入解析。 

[開始使用 API](video-indexer-use-apis.md)

## <a name="next-steps"></a>後續步驟

如需詳細的簡介，請造訪我們的[簡介實驗室](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md)。 

在研討會結束時，您將充分了解可從影片和音訊內容中擷取的資訊種類，且您將可更能找出與內容智慧相關的商機、在 Azure 上調整影片 AI，以及示範影片索引子的數個案例。

