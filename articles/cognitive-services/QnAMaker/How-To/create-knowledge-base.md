---
title: 建立知識庫 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 將閒聊新增至您的 Bot，讓它變得更健談且吸引人。 QnA Maker 可讓您輕鬆地將預先填入的一組最常見閒聊，新增至您的 KB。 這可以是您 Bot 的閒聊起始點，並可節省您從頭撰寫它們的時間和成本。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 274c2289c75f44c5a1c8dd3799612a23f46a6d67
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037689"
---
# <a name="create-a-knowledge-base"></a>建立知識庫

透過 QnA Maker，您可在建立知識庫時輕鬆地新增現有的資料來源。 您可以從下列文件類型建立新的 QnA Maker 知識庫：

<!-- added for scanability -->
* 常見問題集頁面
* 產品手冊
* 結構化文件

## <a name="steps"></a>步驟

1. 使用您的 Azure 認證登入 [QnA Maker 入口網站](https://qnamaker.ai)，然後選取 [建立新的服務]。

    ![建立知識庫 ](../media/qnamaker-how-to-create-kb/create-new-service.png)

2. 如果您尚未建立 QnA Maker 服務，請選取 [建立 QnA 服務]。 

3. 在 QnA Maker 入口網站中從**步驟 2** 的清單中，選取您的 Azure 租用戶、Azure 訂用帳戶名稱，以及與 QnA Maker 服務相關聯的 Azure 資源名稱。 選取將裝載知識庫的 Azure QnA Maker 服務。

    ![設定 QnA 服務](../media/qnamaker-how-to-create-kb/setup-qna-resource.png)

4. 輸入知識庫的名稱和新知識庫的資料來源。

    ![設定資料來源](../media/qnamaker-how-to-create-kb/set-data-sources.png)

    - 您的服務提供**名稱**。 重複的名稱和特殊字元均受支援。
    - 針對您想要擷取的資料新增 URL。 您可以在[這裡](../Concepts/data-sources-supported.md)查看相關資訊，以了解支援的來源類型。
    - 針對您想要擷取的資料上傳檔案。 請參閱[定價資訊](https://aka.ms/qnamaker-pricing)，以了解您可以新增多少文件。
    - 如果您想要手動新增 QnA，可以略過上圖中顯示的**步驟 4**。

5. 將**閒聊**新增至您的 KB。 從 3 個預先定義的特質中選擇一個，以選擇為您的 Bot 新增閒聊支援。 

    <!-- TBD: add back in when chit chat how-to is merged
    ![Add chit-chat to KB ](../media/qnamaker-how-to-chitchat/create-kb-chit-chat.png)
    -->

6. 選取 [建立您的 KB]。

    ![建立知識庫](../media/qnamaker-how-to-create-kb/create-kb.png)

7. 擷取資料需要幾分鐘的時間。

    ![擷取](../media/qnamaker-how-to-create-kb/hang-tight-extraction.png)

8. 成功建立知識庫後，系統會將您重新導向至 [知識庫] 頁面。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [新增閒聊特質](./chit-chat-knowledge-base.md)
