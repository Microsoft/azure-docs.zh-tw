---
title: Azure 上的自訂語音服務計量和配額 | Microsoft Docs
description: Azure 上自訂語音服務的計量和配額相關資訊。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 07/08/2017
ms.author: panosper
ms.openlocfilehash: bd39976691aab0c2333afe9fafc9c5a8cc518b67
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339554"
---
# <a name="custom-speech-service-meters-and-quotas"></a>自訂語音服務計量和配額

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

利用雲端式自訂語音服務，您可以自訂語音模型，以便進行語音轉換文字的轉譯。

若要開始使用自訂語音服務，請移至[自訂語音服務入口網站](https://cris.ai)。

如需目前的定價計量，請移至[自訂語音服務的認知服務定價](https://azure.microsoft.com/pricing/details/cognitive-services/custom-speech-service/)頁面。

## <a name="tiers-explained"></a>定價層說明
僅針對測試和原型，我們建議使用免費 F0 層。 針對生產系統，我們建議使用 S2 層。 使用 S2 層，您可以將部署調整為您的案例所需的縮放單位 (SU) 數。

> [!NOTE]
> 您「無法」在 F0 層與 S2 層之間移轉。
>

## <a name="meters-explained"></a>計量說明

### <a name="scale-out"></a>相應放大
相應放大就是我們隨著新的定價模型發行了新功能。 藉由使用相應放大，您可以控制您的模型可以處理的並行要求數目。

您可以在 [建立模型部署] 檢視中使用 SU 量值，以設定並行要求。 如需詳細資訊，請參閱[建立自訂語音轉換文字端點](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-create-endpoint.md)。 視您設想模型取用的傳輸數量而定，您可以選擇適當的 SU 數目。 

> [!NOTE]
> 每個縮放單位保證 5 個並行要求。 您可以視情況購買 1 或多個 SU。 因為 SU 數目會以增量為 1 的方式增加，所以並行要求數目保證會以增量為 5 的方式增加。
>

### <a name="log-management"></a>記錄管理
您可以選擇針對新部署的模型關閉音訊追蹤 (額外收費)。 自訂語音服務不會記錄該模型中的音訊要求或文字記錄。

## <a name="next-steps"></a>後續步驟
如需關於如何使用自訂語音服務的詳細資訊，請移至[自訂語音服務入口網站](https://cris.ai)。

* [開始使用](cognitive-services-custom-speech-get-started.md)
* [常見問題集](cognitive-services-custom-speech-faq.md)
* [詞彙](cognitive-services-custom-speech-glossary.md)
 