---
title: 調整媒體處理概觀 | Microsoft Docs
description: 此主題為透過 Azure 媒體服務調整媒體處理的概觀。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 780ef5c2-3bd6-4261-8540-6dee77041387
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/30/2018
ms.author: juliako
ms.openlocfilehash: 698a85244d5341224dd9f513c5617b9086e36844
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033068"
---
# <a name="scaling-media-processing-overview"></a>調整媒體處理概觀
此頁面提供調整媒體處理的方式及原因的概觀。 

## <a name="overview"></a>概觀
媒體服務帳戶是與保留單元類型相關聯，後者決定媒體處理工作的速度。 您可以選擇下列的保留單元類型：**S1**、**S2** 或 **S3**。 例如，在執行相同編碼作業的前提下，使用 **S2** 保留單元類型的速度會比 **S1** 類型快。 如需詳細資訊，請參閱 [保留單元類型](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/)。

除了指定保留單元類型之外，您還可以指定使用保留單元來佈建帳戶。 佈建的保留單元數目可決定指定帳戶中可同時處理的媒體工作數目。 例如，如果帳戶有五個保留單元，則只要有工作需要處理，就會同時執行五個媒體工作。 剩餘的工作會在佇列中等待，當執行中的工作完成時，就循序地挑選來處理。 如果帳戶未佈建任何保留單元，則會循序地挑選工作。 在此情況下，一件工作完成與下一件工作開始之間的等待時間，視系統中的資源可用性而定。

## <a name="choosing-between-different-reserved-unit-types"></a>選擇不同的保留單元類型
下表可協助您在不同編碼速度之間進行選擇時做出決定。 它也針對[可供您下載的影片](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z)提供幾個基準測試案例來執行您自己的測試：

| 案例 | **S1** | **S2** | **S3** |
| --- | --- | --- | --- |
| 預定的使用案例 |單一位元速率編碼。 <br/>在 SD或如下解決方法的檔案、對時間不敏感、低成本。 |單一位元速率和多重位元速率編碼。<br/>SD 和 HD 編碼的一般使用方式。 |單一位元速率和多重位元速率編碼。<br/>Full HD 和 4K 解析度影片。 對時間敏感、周轉時間更快的編碼。 |
| 7 分鐘影片的基準測試 |編碼成相同解析度的單一位元速率 MP4 檔案，需要大約 5 分鐘。 |具有「H264 單一位元速率 720p」預設值的編碼需要大約 8 分鐘。<br/><br/>具有「H264 多重位元速率 720p」預設值的編碼需要大約 16.8 分鐘。 |具有「H264 單一位元速率 1080p」預設值的編碼需要大約 4 分鐘。<br/><br/>具有「H264 多重位元速率 1080p」預設值的編碼需要大約 8 分鐘。 |


## <a name="considerations"></a>考量
> [!IMPORTANT]
> 請檢閱本章節所述的考量。  
> 
> 

* 針對由媒體服務 v3 或影片索引子觸發的音訊分析和影片分析作業，強烈建議您使用 S3 單元類型。
* 如果使用共用的集區，也就是沒有任何保留單元，則編碼工作會和 S1 RU 有相同的效能。 不過，您的工作在已排入佇列的狀態下可以花費的時間沒有上限，而且在任何時候，最多只會執行一個工作。

## <a name="billing"></a>計費

您的費用取決於使用媒體保留單元的實際分鐘數。 如需詳細說明，請參閱[媒體服務定價](https://azure.microsoft.com/pricing/details/media-services/)頁面的＜常見問題集＞一節。   

## <a name="quotas-and-limitations"></a>配額和限制
如需配額和限制以及如何開啟支援票證的相關資訊，請參閱 [配額和限制](media-services-quotas-and-limitations.md)。

## <a name="next-step"></a>後續步驟
使用這些技術的其中之一達成調整媒體處理工作︰ 

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-encoding-units.md)
> * [入口網站](media-services-portal-scale-media-processing.md)
> * [REST](https://docs.microsoft.com/rest/api/media/operations/encodingreservedunittype)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
> 

> [!NOTE]
> 若要取得最新的 Java SDK 版本並開始使用 Java 進行開發，請參閱[開始使用適用於媒體服務的 Java 用戶端 SDK ](https://docs.microsoft.com/azure/media-services/media-services-java-how-to-use)。 <br/>
> 若要下載適用於媒體服務的最新 PHP SDK，請在 [Packagist 存放庫](https://packagist.org/packages/microsoft/windowsazure#v0.5.7)中尋找 0.5.7 版的 Microsoft/WindowAzure 套件。  

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

