---
title: 如何使用 REST 取得媒體處理器執行個體 | Microsoft Docs
description: 了解如何建立媒體處理器元件，為 Azure 媒體服務的媒體內容進行編碼、格式轉換、加密或解密。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: f9ff1997-0da6-4528-aaed-792837e5be41
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2017
ms.author: juliako
ms.openlocfilehash: 3e0bd654deca9db8ac13f4af9c4732ba42c01e97
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785687"
---
# <a name="how-to-get-a-media-processor-instance"></a>如何取得媒體處理器執行個體
> [!div class="op_single_selector"]
> * [.NET](media-services-get-media-processor.md)
> * [REST](media-services-rest-get-media-processor.md)
> 
> 

## <a name="overview"></a>概觀
媒體處理器是一項元件，可處理特定處理視訊或音訊工作，例如編碼、格式轉換、加密或解密媒體內容。 提交至媒體服務的所有工作都需要媒體處理器，才能編碼、加密或轉換視訊或音訊內容。 

## <a name="azure-media-processors"></a>Azure 媒體處理器 

下列主題提供媒體處理器的清單：

* [編碼媒體處理器](scenarios-and-availability.md#encoding-media-processors)
* [分析媒體處理器](scenarios-and-availability.md#analytics-media-processors)

>[!NOTE]
>在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

## <a name="connect-to-media-services"></a>連線到媒體服務

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 


## <a name="get-a-media-processor"></a>取得媒體處理器

下列 REST 呼叫示範如何依名稱取得媒體處理器執行個體 (在此案例中，是**媒體編碼器標準**)。 

要求：

    GET https://media.windows.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer <token>
    x-ms-version: 2.17
    Host: media.windows.net

回應：

    . . .

    {  
       "odata.metadata":"https://media.windows.net/api/$metadata#MediaProcessors",
       "value":[  
          {  
             "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
             "Description":"Media Encoder Standard",
             "Name":"Media Encoder Standard",
             "Sku":"",
             "Vendor":"Microsoft",
             "Version":"1.1"
          }
       ]
    }


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
既然您已了解如何取得媒體處理器執行個體，請移至[如何為資產編碼](media-services-rest-get-started.md)文章，其示範如何使用媒體編碼器標準將資產編碼。

