---
title: 在 QnA Maker 知識庫中新增閒聊
titleSuffix: Azure Cognitive Services
description: 當您建立 KB 時，將個人閒聊新增至您的 Bot，讓它變得更健談且吸引人。 QnA Maker 可讓您輕鬆地將預先填入的一組最常見閒聊，新增至您的 KB。 這可以是您 Bot 的閒聊起始點，並可節省您從頭撰寫它們的時間和成本。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/24/2018
ms.author: tulasim
ms.openlocfilehash: 6f542d095211242fcf6dcb13d4426534e32e57ab
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041722"
---
# <a name="add-chit-chat-to-a-knowledge-base"></a>在知識庫中新增閒聊

將閒聊新增至您的 Bot，讓它變得更健談且吸引人。 QnA Maker 中的閒聊功能可讓您輕鬆地將預先填入的一組最常見閒聊，新增至您的資料庫 (KB)。 這可以是您 Bot 的特質起始點，並將節省您從頭撰寫它們的時間和成本。  

此資料集有大約 100 個以三個角色發音的閒聊案例 - 專業人員、朋友、卡通人物。 選擇最類似您 Bot 聲音的角色。 假設有使用者查詢，QnA Maker 會嘗試將它與最接近的已知閒聊 QnA 比對。 

不同特質的一些範例：<!-- added quotes so acrolinx doesn't score these sentences -->
|使用者查詢|專業人員|朋友|卡通人物|
|--|--|--|--|
|`You are awesome`|`I aim to serve.`|`That's so nice of you!`|`Flattery. I like it.`|
|`Are you hungry?`|`I don't need to eat.`|`I only do food for thought.`|`Eating would require a lot of things I don't have. Like a digestive system. And silverware.`|
|`Sing a song`|`I'm afraid I'm not musically inclined.`|`La la la, tra la la. I'm awesome at this.`|`You can't handle my dulcet tones.`|
|`Who made you?`|`There wouldn't be time to list everyone.`|`So many people!`|`Nerds.`|


> [!NOTE]
> 閒聊支援目前僅以英文提供。 

## <a name="add-chit-chat-during-kb-creation"></a>在知識庫建立期間新增閒聊
在知識庫建立期間，新增來源 URL 和檔案之後，有可供新增閒聊的選項。 選擇您想要作為聊天基底的特質。 如果您不想新增閒聊，或如果您的資料來源中已經有閒聊支援，請選擇 [無]。 
   
![在建立期間新增閒聊](../media/qnamaker-how-to-chit-chat/create-kb-chit-chat.png)

## <a name="add-chit-chat-to-an-existing-kb"></a>在現有知識庫中新增閒聊
選取您的知識庫，並瀏覽至 [設定] 頁面。 有一個連結可連到適當 **.tsv** 格式的所有閒聊資料集。 下載您想要的特質，然後將它上傳為檔案來源。 當您下載並上傳檔案時，務必不要編輯格式或中繼資料。 
  
![在現有知識庫中新增閒聊](../media/qnamaker-how-to-chit-chat/add-chit-chat-dataset.png)

## <a name="edit-your-chit-chat-questions-and-answers"></a>編輯您的閒聊問題和回答
當您編輯知識庫時，您會看到閒聊的新來源，其是以您選取的特質為基礎。 您現在可以新增變更的問題或編輯回應，就像處理任何其他來源一樣。 

![編輯閒聊 QnA](../media/qnamaker-how-to-chit-chat/edit-chit-chat.png)

## <a name="add-additional-chit-chat-questions-and-answers"></a>新增其他閒聊問題和回答
您可以新增不在預先定義集合中的新閒聊 QnA。 確保您不會複製閒聊集已經涵蓋的 QnA 配對。 當您新增任何新的閒聊 QnA 時，它會新增到您的**編輯**來源。 若要確保順位排定程式了解這是閒聊，請新增中繼資料索引鍵/值配對「編輯：閒聊」，如下圖所示：
   
![新增閒聊 QnA](../media/qnamaker-how-to-chit-chat/add-new-chit-chat.png)

## <a name="delete-chit-chat-from-an-existing-kb"></a>從現有知識庫中刪除閒聊
選取您的知識庫，並瀏覽至 [設定] 頁面。 您的特定閒聊來源會以檔案列示，內含選取的特質名稱。 您可以將此當作原始程式檔刪除。

![從知識庫中刪除閒聊](../media/qnamaker-how-to-chit-chat/delete-chit-chat.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [匯入知識庫](../Tutorials/migrate-knowledge-base.md)

## <a name="see-also"></a>另請參閱 

[QnA Maker 概觀](../Overview/overview.md)
