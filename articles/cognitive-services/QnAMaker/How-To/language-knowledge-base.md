---
title: 非英文知識庫 - QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker 支援多種語言的知識庫內容。 不過，個別的 QnA Maker 服務均應保留給單一語言使用。 針對特定 QnA Maker 服務而建立的第一個知識庫，會設定該服務的語言。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: b983fb21e8e67a422b6757619d1d0dfe8b6b9dcc
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033901"
---
# <a name="language-support-of-knowledge-base-content-for-qna-maker"></a>QnA Maker 知識庫內容的語言支援
QnA Maker 支援多種語言的知識庫內容。 不過，個別的 QnA Maker 服務均應保留給單一語言使用。 針對特定 QnA Maker 服務而建立的第一個知識庫，會設定該服務的語言。 請參閱[這裡](../Overview/languages-supported.md)以取得支援的語言清單。

系統會從要擷取的資料來源內容自動辨識語言。 在您建立新的 QnA Maker 服務和該服務中新的知識庫之後，您可以確認語言已正確設定。

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [資源群組]，並瀏覽至部署 QnA Maker 服務的資源群組，然後選取 [Azure 搜尋服務] 資源。

    ![選取 [Azure 搜尋服務] 資源](../media/qnamaker-how-to-language-kb/select-azsearch.png)

3. 選取 **testkb** 索引。 此 Azure 搜尋索引一律是第一個建立的索引，且其中包含該服務中所有知識庫的已儲存內容。 

    ![選取測試 KB](../media/qnamaker-how-to-language-kb/select-testkb.png)

4. 選取顯示 testkb 詳細資料的 [欄位] 區段。

    ![選取欄位](../media/qnamaker-how-to-language-kb/selectfields.png)

5. 勾選 [分析器] 的方塊以檢視語言詳細資料。

    ![選取分析器](../media/qnamaker-how-to-language-kb/select-analyzer.png)

6. 您應該會發現分析器已設定為特定語言。 這是在知識庫建立步驟執行期間自動偵測到的語言。 此語言在資源建立後即無法變更。

    ![已選取的分析器](../media/qnamaker-how-to-language-kb/selected-analyzer.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure Bot 服務建立 QnA Bot](../Tutorials/create-qna-bot.md)
