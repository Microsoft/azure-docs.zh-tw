---
title: 將驗證權杖傳遞到 Azure 媒體服務 | Microsoft Docs
description: 了解如何將驗證權杖從用戶端傳送到 Azure 媒體服務金鑰傳遞服務
services: media-services
keywords: content protection, DRM, token authentication, 內容保護, 權杖驗證
documentationcenter: ''
author: dbgeorge
manager: jasonsue
editor: ''
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/01/2017
ms.author: dwgeo
ms.openlocfilehash: b6aca2928465b73e35ac15f01bb776b1f69add0b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783117"
---
# <a name="learn-how-clients-pass-tokens-to-the-azure-media-services-key-delivery-service"></a>了解用戶端如何將權杖傳遞到 Azure 媒體服務金鑰傳遞服務
客戶常常詢問播放程式如何將權杖傳遞至 Azure 媒體服務金鑰傳遞服務以進行認證，讓播放器能夠取得金鑰。 媒體服務支援簡單 Web 權杖 (SWT) 和 JSON Web 權杖 (JWT) 格式。 不論您在系統中使用的是一般加密或進階加密標準 (AES) 信封加密，權杖驗證都會套用至任何類型的金鑰。

 您可以根據設定為目標的播放程式和平台，透過以下方式利用播放程式傳遞權杖：

- 透過 HTTP 授權標頭。
    > [!NOTE]
    > 根據 OAuth 2.0 規格，系統會預期 "Bearer" 前置詞。 某個範例播放程式的權杖設定已裝載於 Azure 媒體播放器[示範頁面](http://ampdemo.azureedge.net/) \(英文\) 上。 若要設定影片來源，請選擇 [AES (JWT 權杖)] 或 [AES (SWT 權杖)]。 權杖會透過授權標頭來傳遞。

- 透過新增含有 “token=tokenvalue” 的 URL 查詢參數。  
    > [!NOTE]
    > 不應該使用 "Bearer" 前置碼。 因為權杖已透過 URL 傳送，所以您需要為權杖字串提供防禦。 以下是示範如何執行此動作的 C# 範例程式碼：

    ```csharp
    string armoredAuthToken = System.Web.HttpUtility.UrlEncode(authToken);
    string uriWithTokenParameter = string.Format("{0}&token={1}", keyDeliveryServiceUri.AbsoluteUri, armoredAuthToken);
    Uri keyDeliveryUrlWithTokenParameter = new Uri(uriWithTokenParameter);
    ```

- 透過 CustomData 欄位。
此選項僅適用於取得 PlayReady 授權，透過 PlayReady 授權取得查問的 CustomData 欄位。 在此案例中，權杖必須位於 XML 文件中，如下所述：

    ```xml
    <?xml version="1.0"?>
    <CustomData xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyCustomData/v1"> 
        <Token></Token> 
    </CustomData>
    ```
    將您的驗證權杖放置於 Token 元素中。

- 透過替代的 HTTP 即時串流 (HLS) 播放清單。 如果您需要針對 iOS/Safari 上的 AES + HLS 播放設定權杖驗證，則沒有任何方式可讓您直接傳送權杖。 如需如何替代播放清單來實現此案例的詳細資訊，請參閱這篇[部落格文章](http://azure.microsoft.com/blog/2015/03/06/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/) \(英文\)。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]