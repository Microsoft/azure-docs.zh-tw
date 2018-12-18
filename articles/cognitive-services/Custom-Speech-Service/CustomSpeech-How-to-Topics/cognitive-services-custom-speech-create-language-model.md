---
title: 使用自訂語音服務建立語言模型的教學課程 - Microsoft 認知服務 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Microsoft 認知服務中的自訂語音服務來建立語言模型。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: tutorial
ms.date: 05/03/2017
ms.author: panosper
ms.openlocfilehash: 6af2da9ffc7678a58fcf1c647ba89c586066d2ad
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339091"
---
# <a name="tutorial-create-a-custom-language-model"></a>教學課程：建立自訂語言模型

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

在本教學課程中，您會針對文字查詢或您希望使用者在應用程式中說出或輸入的語句，建立自訂語言模型。 然後，您可以使用此自訂語言模型搭配 Microsoft 中現有的頂級語音模型，來為您的應用程式新增語音互動。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 準備資料
> * 匯入語言資料集
> * 建立自訂語言模型

如果您沒有認知服務帳戶，請在開始前建立[免費帳戶](https://cris.ai)。

## <a name="prerequisites"></a>必要條件

開啟[認知服務訂用帳戶](https://cris.ai/Subscriptions)頁面，確任您的認知服務帳戶已連線到訂用帳戶。

如果沒有列出任何訂用帳戶，您可以按一下 [取得免費訂用帳戶] 按鈕，讓認知服務為您建立帳戶。 或是，您可以按一下 [與現有訂用帳戶連線] 按鈕，連線到 Azure 入口網站中建立的自訂搜尋服務訂用帳戶。

如需在 Azure 入口網站中建立自訂搜尋服務訂用帳戶的資訊，請參閱[在 Azure 入口網站中建立認知服務 API 帳戶](../../cognitive-services-apis-create-account.md)。

## <a name="prepare-the-data"></a>準備資料

若要為您的應用程式建立自訂語言模型，您需要提供語句範例的清單給系統，例如：

*   「他上週長了蕁麻疹。」
*   「病患的疝氣傷口復原得很好。」

句子不需要是完整句子或文法正確，但應精確地反映在部署中系統預期會遇到的語音輸入。 這些範例應反映使用者使用您應用程式時的方式和工作內容。

語言模型資料應使用 US-ASCII 或 UTF-8 寫入純文字檔案中 (視地區設定而定)。 對於 en-US，這兩個編碼方式皆可支援。 對於 zh-CN，只有 UTF-8 可支援 (BOM 是選擇性的)。 文字檔案中的每一行皆應包含一個範例 (句子、語句或或查詢)。

如果您想讓某些句子擁有較高權重 (重要性)，您可以將它多次新增至您的資料。 理想的重複次數是介於 10 到 100 之間。 如果將其正規化為 100，您可以輕鬆地加權與此相應的句子。

下表摘要說明語言資料的主要需求。

| 屬性 | 值 |
|----------|-------|
| 文字編碼方式 | en-US: US-ACSII 或 UTF-8 或 zh-CN: UTF-8|
| 每一行的語句數目 | 1 |
| 檔案大小上限 | 200 MB |
| 備註 | 避免重複字元超過 4 次，例如 'aaaaa'|
| 備註 | 沒有 '\t' 這類特殊字元，或 [Unicode 字元資料表](http://www.utf8-chartable.de/)中 U+00A1 以外的任何其他 UTF-8 字元|
| 備註 | URI 也會遭到拒絕，因為沒有可讀出 URI 的專屬方式|

由於文字匯入時，經過了文字正規化，因此可由系統處理。 不過，有一些重要的正規化，必須由使用者在上傳資料_之前_完成。 請參閱[轉譯指導方針](cognitive-services-custom-speech-transcription-guidelines.md)，以在準備語言資料時判斷適當的語言。

## <a name="import-the-language-data-set"></a>匯入語言資料集

按一下 [原音資料集] 資料列中的 [匯入] 按鈕，網站會顯示上傳新資料集的頁面。

當您準備好匯入您的語言資料集時，登入[自訂語音服務入口網站](https://cris.ai)。  然後按一下上方功能區的 [自訂語音] 下拉式選單，然後選取 [調整的資料]。 如果這是您第一次將資料上傳到自訂語音服務，您會看到名為「資料集」的空白資料表。

若要匯入新資料集，按一下 [語言資料集] 資料列中的 [匯入] 按鈕，網站會顯示上傳新資料集的頁面。 輸入的名稱和描述，以便在未來識別資料集。 接下來，使用 [選擇檔案] 按鈕來尋找語言資料的文字檔。 之後，按一下 [匯入]，即可上傳資料集。 視資料集的大小而定，這可能需要數分鐘的時間。

![嘗試](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets-import.png)

匯入完成時，您會返回語言資料的資料表，並且會看到您語言資料集的對應項目。 請注意，其已獲派唯一識別碼 (GUID)。 資料也會有狀態來反映其目前的狀態。 當資料排入佇列以進行處理時，其狀態會是「等待中」，資料正在通過驗證時會顯示「處理中」，資料可供使用時，則會顯示「完成」。 資料驗證會對檔案中的文字執行一系列檢查，以及執行一些資料的文字正規化。

當狀態為「完成」時，您可以按一下 [檢視報表] 來查看語言資料的驗證報告。 您會看到通過驗證或驗證失敗的語句數目，以及失敗語句的詳細資料。 在下列範例中，有兩個範例因為不正確的字元而驗證失敗 (在此資料集中，第一個有兩個表情符號，而第二個有數個 ASCII 可列印字元集以外的字元)。

![嘗試](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets-report.png)

當語言的資料集狀態為「完成」時，即可用來建立自訂語言模型。

![嘗試](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets.png)

## <a name="create-a-custom-language-model"></a>建立自訂語言模型

當您的語言資料準備就緒之後，從 [功能表] 下拉式功能表按一下 [語言模型]，以啟動自訂語言模型的建立程序。 此頁面包含名為「語言模型」的資料表，其中包含您目前的自訂語言模型。 如果您尚未建立任何自訂語言模型，此資料表會是空白的。 目前的地區設定會反映在資料表標題。 如果您想要建立不同語言的語言模型，請按一下 [變更地區設定]。 您可以在[變更地區設定](cognitive-services-custom-speech-change-locale.md)一節中找到其他支援語言的資訊。 若要建立新的模型，按一下資料表標題下方的 [新建] 連結。

在 [建立語言模型] 頁面上輸入「名稱」和「描述」，可協助您持續追蹤此模型的相關資訊，例如使用的資料集。 接下來，從下拉式功能表中選取 [底語言模型]。 此模型會是自訂的起點。 有兩種基底語言模型可選擇。 _Microsoft 搜尋與聽寫 LM_ 適用於應用程式上的語音導向作業，例如命令、搜尋查詢或聽寫。 _Microsoft 交談 LM_ 適用於辨識以交談形式說出的語音。 這類型的語音通常會指向另一個人，並且發生在話務中心或會議中。

指定基底語言模型之後，使用 [語言資料] 下拉式功能表選取您想用於自訂的語言資料集

![嘗試](../../../media/cognitive-services/custom-speech-service/custom-speech-language-models-create2.png)

與建立原音模型時一樣，您可以選擇是否在處理完成時，對新模型執行離線測試。 由於這是語音轉文字的效能評估，因此離線測試需要原音資料集。

若要為您的語言模型執行離線測試，請選取 [離線測試] 旁的核取方塊。 從下拉式功能表中選取原音模型。 如果您尚未建立任何自訂原音模型，Microsoft 基底原音式模型將會是功能表中唯一的模型。 如果您已挑選了交談 LM 基底模型，您需要在此使用交談 AM。 如果您使用搜尋與聽寫 LM 模型，您必須選取搜尋與聽寫 AM 模型。

最後，選取您想要用來執行評估的原音資料集。

當您準備好要啟動處理程序時，請按下 [建立]。 這會使您返回語言模型資料表。 資料表中將會有對應此模型的新項目。 狀態會反映模型的狀態，並且會經過幾個狀態，包括「等待中」、「處理中」和「完成」。

當模型達到「完成」狀態時，即可部署至端點。 按一下「檢視結果」會顯示離線測試的結果 (若有執行的話)。

如果您想要變更模型上某些部分的「名稱」或「描述」，您可以在語言模型資料表中，使用適當資料列中的 [編輯] 連結。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已開發搭配文字使用的自訂語言模型。 若要建立與音訊檔案和轉譯搭配使用的自訂原音模型，請繼續前往建立原音模型的教學課程。

> [!div class="nextstepaction"]
> [建立自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)
