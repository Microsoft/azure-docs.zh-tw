---
title: 開始使用 Azure 自訂語音服務 | Microsoft Docs
description: 訂閱自訂語音服務，並將服務活動連結至 Azure 訂用帳戶，以將模型定型並執行部署。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 02/08/2017
ms.author: panosper
ms.openlocfilehash: 4bbd4c57556fd4bfd176c915e26be4a4d198418a
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339928"
---
# <a name="get-started-with-custom-speech-service"></a>開始使用自訂語音服務

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

瀏覽自訂語音服務的主要功能，並了解如何針對應用程式的需求來建置、部署和使用原音與語言模型。 在「自訂語音服務」入口網站上完成註冊後，便可以找到更廣泛的文件和逐步指示。

## <a name="samples"></a>範例  
我們會提供絕佳範例供您進行本課程，其位置在[這裡](https://github.com/Microsoft/Cognitive-Custom-Speech-Service)。

## <a name="prerequisites"></a>必要條件  

### <a name="subscribe-to-custom-speech-service-and-get-a-subscription-key"></a>訂閱自訂語音服務並取得訂用帳戶金鑰
在進行上述範例之前，您必須先訂閱自訂語音服務，並取得訂用帳戶金鑰，請參閱[訂用帳戶](https://portal.azure.com/#create/Microsoft.CognitiveServices/apitype/CustomSpeech)或遵循[這裡](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-subscribe.md)的說明。 本教學課程可以同時使用主要金鑰和次要金鑰。 請務必遵循最佳做法以保護您的 API 金鑰祕密。

### <a name="get-the-client-library-and-example"></a>取得用戶端程式庫和範例
您可以透過 [SDK](https://www.microsoft.com/cognitive-services/en-us/SDK-Sample?api=bing%20speech&category=sdk) 下載用戶端程式庫和範例。 下載的 zip 檔案必須解壓縮至您選擇的資料夾，大部分使用者選擇 Visual Studio 2015 資料夾。

## <a name="creating-a-custom-acoustic-model"></a>建立自訂原音模型
若要針對特殊網域自訂原音模型，您必須有語音資料集合。 這個集合是由一組含有語音資料的音訊檔，以及含有各音訊檔抄本的文字檔所組成的。 音訊資料要能代表您想要使用辨識器的案例

例如：如果您想要更清楚地辨識吵雜工廠環境中的語音，音訊檔內就應該包含人們在吵雜工廠中說話的聲音。
如果您想要讓單一演講者的表現發揮最佳效果，例如，您想要抄寫美國總統富蘭克林‧羅斯福的所有「爐邊談話」，則音訊檔內就應該包含許多只有該名演講者的範例。

您可以在[這裡](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-create-acoustic-model.md)找到如何建立自訂原音模型的詳細說明。

## <a name="creating-a-custom-language-model"></a>建立自訂語言模型
自訂語言模型的建立程序與原音模型的建立程序相似，差別只在其中沒有音訊資料，而只有文字。 文字中應該包含您預期使用者會說出，或是您會讓已登入使用者在應用程式中說出 (或輸入) 的許多查詢或語句範例。

您可以在[這裡](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-create-language-model.md)找到如何建立自訂語言模型的詳細說明。

## <a name="creating-a-custom-speech-to-text-endpoint"></a>建立自訂的語音轉文字端點
在建立好自訂原音模型和 (或) 語言模型時，您可以將這些模型部署至自訂的語音轉文字端點。 若要建立新的自訂端點，請從頁面頂端的 [CRIS] 功能表按一下 [部署]。 這會帶您前往名為「部署」的現行自訂端點資料表。 如果您尚未建立任何端點，此資料表會是空白的。 目前的地區設定會反映在資料表標題。 如果您想要建立不同語言的部署，請按一下 [變更地區設定]。 您可以在＜變更地區設定＞一節中找到其他支援語言的資訊。

您可以在[這裡](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-create-endpoint.md)找到如何建立自訂語音轉文字端點的詳細說明。

## <a name="using-a-custom-speech-endpoint"></a>使用自訂的語音端點
您可以使用和預設 Microsoft 認知服務語音端點極為類似的方式，將要求傳送至 CRIS 語音轉文字端點。 請注意，這些端點的功能與「語音 API」的預設端點相同。 因此，可透過用戶端程式庫或 REST API 供「語音 API」使用的功能，同樣也可供自訂端點使用。

您可以在[這裡](CustomSpeech-How-to-Topics/cognitive-services-custom-speech-use-endpoint.md)找到如何使用自訂語音轉文字端點的詳細說明。


請注意，透過 CRIS 所建立的端點所能處理的不同並行要求數目，取決於訂用帳戶是與哪一個服務層相關聯。 如果所要求的辨識數超過限制，這些端點便會傳回錯誤碼 429 (太多要求)。 如需詳細資訊，請瀏覽[定價資訊](https://www.microsoft.com/cognitive-services/en-us/pricing)。 此外，免費層也有每月要求配額限制。 如果您在免費層存取的端點數超過每月配額限制，服務便會傳回錯誤碼「403 禁止」。

服務會假設音訊是即時傳輸的。 如果其傳送速度較快，系統便會將要求視為執行中，直到其即時持續時間過去。

* [概觀](cognitive-services-custom-speech-home.md)
* [常見問題集](cognitive-services-custom-speech-faq.md)
* [詞彙](cognitive-services-custom-speech-glossary.md)
