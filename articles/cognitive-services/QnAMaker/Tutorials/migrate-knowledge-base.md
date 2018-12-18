---
title: 遷移預覽知識庫 - Qna Maker
titleSuffix: Azure Cognitive Services
description: 如何匯入知識庫
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 0cb8a185407c7b180a170f1f9b9d76aa28a24de5
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031623"
---
# <a name="migrate-a-knowledge-base-using-export-import"></a>使用匯出-匯入移轉知識庫
QnA Maker 於 2018 年 5 月 7 日在 \\\build\ 會議宣佈了公開上市。 QnA Maker GA 具有在 Azure 上建置的新架構。 使用 QnA Maker 免費預覽建立的知識庫必須移轉至 QnA Maker GA。 QnA Maker 預覽版即將在 2018 年 11 月 淘汰。 如需 QnA Maker GA 變更的詳細資訊，請參閱 QnA Maker GA 公告[部落格文章](https://aka.ms/qnamakerga-blog)。

QnA Maker 現在有[定價模型](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/qna-maker/)。

必要條件
> [!div class="checklist"]
> * 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
> * 設定新的 [QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)

## <a name="migrate-a-knowledge-base-from-qna-maker-preview-portal"></a>從 QnA Maker 預覽入口網佔移轉知識庫
1. 巡覽至 [QnA Maker 預覽入口網站](https://aka.ms/qnamaker-old-portal
)，然後按一下 **[我的服務]** 。
2. 按一下 [編輯] 圖示，選取您想要移轉的知識庫。

    ![編輯知識庫](../media/qnamaker-how-to-migrate-kb/preview-editkb.png)

3. 按一下 [下載知識庫] 以下載 .tsv 檔案，其中包含您的知識庫內容 - 問題、解答、中繼資料，以及從中進行擷取的資料來源名稱。

    ![下載知識庫](../media/qnamaker-how-to-migrate-kb/preview-download.png)

4. 使用您的 Azure 認證登入 [QnA Maker 入口網站](https://qnamaker.ai)，然後按一下 [建立新的服務]。

    ![建立知識庫 ](../media/qnamaker-how-to-create-kb/create-new-service.png)
    
5. 如果您尚未建立 QnA Maker 服務，請選取 [建立 QnA 服務]。 否則，請在步驟 2 中從下拉式清單選擇 QnA Maker 服務。 選取將裝載知識庫的 QnA Maker 服務。

    ![設定 QnA 服務](../media/qnamaker-how-to-create-kb/setup-qna-resource.png)

6. 建立空的知識庫 

    ![設定資料來源](../media/qnamaker-how-to-create-kb/set-data-sources.png)

    - 您的服務提供**名稱**。 重複的名稱和特殊字元均受支援。
    - 當您想要使用預覽知識庫中的資料時，請略過上傳檔案或 URL。 現在，您將建立空的知識庫。

7. 選取 [建立] 。

    ![建立知識庫](../media/qnamaker-how-to-create-kb/create-kb.png)

8. 在這個新知識庫中，開啟 [設定] 索引標籤，然後按一下 [匯入知識庫]。 這會匯入問題、解答及中繼資料，而且會保留從中進行擷取的資料來源名稱。

   ![匯入知識庫](../media/qnamaker-how-to-migrate-kb/Import.png)

9. 使用 [測試] 面板來 [測試] 新知識庫。 了解如何[測試知識庫](../How-To/test-knowledge-base.md)。
10. [發佈] 知識庫。 了解如何[發佈知識庫](../How-To/publish-knowledge-base.md)。
11. 在您的應用程式或 Bot 程式碼中使用下面的端點。 請參閱如何[建立 QnA Bot](../Tutorials/create-qna-bot.md)。

    ![QnA Maker 值](../media/qnamaker-tutorials-create-bot/qnamaker-settings-kbid-key.PNG)

此時，所有知識庫內容 (問題、解答和中繼資料，以及來源檔案名稱和 URL) 都會匯入新的知識庫。 

## <a name="chatlogs-and-alterations"></a>聊天記錄和變動
不會自動匯入變動 (同義字)。 使用 [V2 API](https://aka.ms/qnamaker-v2-apis) 從預覽堆疊匯出變動，以及使用 [V4 API](https://aka.ms/qnamaker-v4-apis) 在新堆疊中取代。

沒有任何方法可移轉聊天記錄，因為新堆疊會使用 Application Insights 來儲存聊天記錄。 不過，您可以從[預覽入口網站](https://aka.ms/qnamaker-old-portal)下載聊天記錄。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [編輯知識庫](../How-To/edit-knowledge-base.md)
