---
title: 知識庫的開發生命週期 - QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker 在反覆的週期中學習模型變更、語句範例、發佈資料以及從端點查詢收集資料時，會有最佳學習成效。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 5af829b3355c6d68bace959b66f9511877d08b83
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040909"
---
# <a name="knowledge-base-lifecycle"></a>知識庫生命週期
QnA Maker 在反覆的週期中學習模型變更、語句範例、發佈資料以及從端點查詢收集資料時，會有最佳學習成效。 

![編寫週期](../media/qnamaker-concepts-lifecycle/kb-lifecycle.png)

## <a name="creating-a-qna-maker-knowledge-base"></a>建立 QnA Maker 知識庫
QnA Maker 知識庫 (KB) 端點會根據知識庫的內容對使用者查詢提供最相符的解答。 建立知識庫是指為問題、解答和相關聯的中繼資料設定內容存放庫的一次性動作。 知識庫可藉由對既有的內容編目來建立，這些內容包括常見問題集頁面、產品手冊或結構化問答組。 了解如何[建立知識庫](../How-To/create-knowledge-base.md)。

## <a name="testing-and-updating-the-knowledge-base"></a>測試和更新知識庫
知識庫在填入內容 (透過編輯或自動擷取) 後即可供測試。 您可以透過 [測試] 面板輸入常見的使用者查詢，並確認傳回的回應符合預期並有足夠的信賴分數，來完成測試。 您可以新增替代問題，以因應信賴分數偏低的狀況。 您也可以在查詢傳回「在知識庫預設回應中找不到相符項目」時新增解答。 「測試-更新」的這個密封迴圈會持續執行，直到您得到滿意的結果為止。 了解如何[測試知識庫](../How-To/test-knowledge-base.md)。

對於大型知識庫，測試可以透過 generateAnswer API 自動化。 

## <a name="publish-the-knowledge-base"></a>發佈知識庫
完成知識庫的測試後，您即可發佈知識庫。 在發佈時，會將經過測試的最新版知識庫推送至代表**已發佈**知識庫的專用 Azure 搜尋服務索引。 它也會建立可在您的應用程式或聊天機器人中呼叫的端點。

如此，對測試版知識庫所做的任何變更，都不會對可能正在生產應用程式中執行的已發行版本造成影響。

這些知識庫全都可以個別進行測試。 您可以使用 API，並在 generateAnswer 呼叫中加上 `isTest=true` 旗標，將測試版的知識庫設為目標。

了解如何[發佈知識庫](../How-To/publish-knowledge-base.md)。

## <a name="monitor-usage"></a>監視使用量
若要能夠記錄服務的聊天記錄，您必須在[建立 QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)時啟用 Application Insights。

您可以取得服務使用情形的各種分析。 深入了解如何使用 Application Insights 取得 [QnA Maker 服務的分析](../How-To/get-analytics-knowledge-base.md)。

您可以根據從分析得出的結論，適當[更新您的知識庫](../How-To/edit-knowledge-base.md)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [信賴分數](./confidence-score.md)

## <a name="see-also"></a>另請參閱 

[知識庫](./knowledge-base.md)
[QnA Maker 概觀](../Overview/overview.md)
