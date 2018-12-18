---
title: 什麼是自訂語音服務？
titlesuffix: Azure Cognitive Services
description: 自訂語音服務是一種雲端式服務，可讓使用者自訂語音模型以進行語音轉文字的轉譯。
services: cognitive-services
author: PanosPeriorellis
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: overview
ms.date: 02/07/2017
ms.author: panosper
ROBOTS: NOINDEX
ms.openlocfilehash: f90fc40a42806cfb002da2d9943eaa41736df4d7
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222738"
---
# <a name="what-is-custom-speech-service"></a>什麼是自訂語音服務？

自訂語音服務是一種雲端式服務，可讓使用者自訂語音模型以進行語音轉文字的轉譯。
若要使用自訂語音服務，請參閱[自訂語音服務入口網站](https://cris.ai)。

自訂語音服務可讓您根據應用程式和使用者的需求，建立自訂的語言模型和原音模型。 您可以將特定的語音及/或文字資料上傳至自訂語音服務，藉以建立可與 Microsoft 現有的最新語音模型搭配使用的自訂模型。

例如，如果您將語音互動新增至行動電話、平板電腦或 PC 應用程式，則可以建立可與 Microsoft 的原音模型搭配使用的自訂語言模型，以建立特別為您的應用程式設計的語音轉文字端點。 如果您的應用程式專用於特定環境，或由特定使用者族群使用，則您也可以使用此服務建立並部署自訂原音模型。


## <a name="how-do-speech-recognition-systems-work"></a>語音辨識系統如何運作？
語音辨識系統由數個相互搭配運作的元件所組成。 其中，原音模型和語言模型是兩項最重要的元件。

原音模型是會將簡短的音訊片斷標入指定語言中數種音素或聲音單位之一的分類器。 例如，“speech” 這個字即由四個音素 “s p iy ch” 所組成。 這些分類以每秒 100 次的速率執行。

語言模型是字組序列的機率分布。 語言模型可根據字組序列本身的可能性，協助系統判定發音類似的字組序列。 例如，"recognize speech" 與 "wreck a nice beach" 發音類似，但前者的假設較為可能發生，因此會依據語言模型為其指派較高的分數。

原音和語言模型都是從訓練資料中學習而來的統計模型。 因此，在執行這些模型時，若使用於應用程式中的語音類似於在訓練期間觀察到的資料，這些模型將會有最佳效能。 Microsoft 語音轉文字引擎中的原音和語言模型經過龐大語音和文字集合的訓練，可針對最常見的使用案例提供最高階的效能，例如，在您的智慧型手機、平板電腦或 PC 上與 Cortana 互動、以語音搜尋網路，或以口述方式傳送簡訊給好友。

## <a name="why-use-the-custom-speech-service"></a>為何要使用自訂語音服務？
Microsoft 語音轉文字引擎具有世界級水準，並且適用於上述案例。 然而，若預期您應用程式的語音查詢會包含特定的詞彙項目，像是一般語音中很少出現的產品名稱或行話，則應可藉由自訂語言模型強化效能。

例如，如果將應用程式建置為會利用語音搜尋 MSDN，很有可能會較一般語音應用程式更常出現像是 “object-oriented” 或 “namespace” 或 “dot net” 等字詞。 自訂語言模型能讓系統學習此情況。

## <a name="next-steps"></a>後續步驟

如需關於如何使用自訂語音服務的詳細資訊，請參閱 [自訂語音服務入口網站] (https://cris.ai)。

* [快速入門](cognitive-services-custom-speech-get-started.md)
* [常見問題集](cognitive-services-custom-speech-faq.md)
* [詞彙](cognitive-services-custom-speech-glossary.md)
