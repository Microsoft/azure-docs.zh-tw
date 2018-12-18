---
title: Azure 媒體服務 v3 版本資訊 | Microsoft Docs
description: 為了讓您隨時掌握最新的開發訊息，此文章提供 Azure 媒體服務 v3 最新資訊。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 09/25/2018
ms.author: juliako
ms.openlocfilehash: ed2550c1df4645933fb968c54ee536995c810136
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219317"
---
# <a name="azure-media-services-v3-preview-release-notes"></a>Azure 媒體服務 v3 (預覽) 版本資訊 

為了讓您隨時掌握最新的開發訊息，此文章提供下列相關資訊：

* 最新版本
* 已知問題
* 錯誤修正
* 已被取代的功能
* 方案變更

## <a name="may-07-2018"></a>2018 年 5 月 7 日

### <a name="net-sdk"></a>.Net SDK

以下是 .Net SDK 現有的功能︰

1. **轉換**和**工作**，可編碼或分析媒體內容。 如需範例，請參閱[串流處理檔案](stream-files-tutorial-with-api.md)和[分析](analyze-videos-tutorial-with-api.md)。
2. **StreamingLocators**，用於將內容發佈及串流處理到使用者裝置
3. **StreamingPolicies** 和 **ContentKeyPolicies**，當傳遞內容時，可設定金鑰傳遞和內容保護 (DRM)。
4. **LiveEvents** 和 **LiveOutputs**，可設定內嵌和封存即時串流內容。
5. **資產**，可在 Azure 儲存體中儲存及發佈媒體內容。 
6. **StreamingEndpoint**，可設定和擴展動態套件、 加密和串流處理實況和點播媒體內容。

### <a name="known-issues"></a>已知問題

* 提交工作時，您可以使用 HTTPS URL、SAS URL 或位於 Azure Blob 儲存體中檔案的路徑來指定內嵌您的來源影片。 目前，AMS v3 不支援透過 HTTPS URL 的區塊傳送編碼。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [概觀](media-services-overview.md)
