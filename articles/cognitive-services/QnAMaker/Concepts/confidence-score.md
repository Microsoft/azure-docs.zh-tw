---
title: 信賴分數 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 信賴分數代表使用者的問題和傳回的回應之間的符合程度。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 33da5cf5724b8314ce813f12eb077d9a15ec1b2a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041518"
---
# <a name="confidence-score"></a>信賴分數

信賴分數代表使用者的問題和傳回的回應之間的符合程度。

針對知識庫的內容配對使用者的查詢時，可以傳回多個回應。 回應是依照遞減信賴分數的順位次序傳回。

信賴分數介於 0 到 100 之間。

|分數值|信賴度|
|--|--|
|100|使用者查詢和 KB 問題完全相符|
|90|高度信賴 - 大部分的字組相符|
|40-60|適度信賴 - 重要的字組相符|
|10|低度信賴 - 重要的字組不相符|
|0|沒有任何字組相符|


## <a name="similar-confidence-scores"></a>類似的信賴分數
多個回應有類似的信賴分數時，表示查詢過於不具體，因此有多個同等可能性的相符答案。 請嘗試進一步調整 QnA，以便每個 QnA 實體的意圖彼此相異。


## <a name="improving-confidence-scores"></a>改善信賴分數
若要改善使用者查詢特定回應的信賴分數，您可以將使用者查詢加入至知識庫做為該回應的替代問題。
   
## <a name="confidence-score-differences"></a>信賴分數差異
在測試和發佈的知識庫版本之間，即使內容相同，不過答案的信賴分數可能會略微變化。 這是因為測試和發佈的知識庫位於不同的 Azure 搜尋服務索引中。

在這裡查看[發佈](../How-To/publish-knowledge-base.md)作業如何進行。


## <a name="no-match-found"></a>未找到相符項目
順位排定程式找不到適當的相符項目時，會傳回 0.0 的信賴分數或「無」，而且預設回應是「在資料庫中找不到適當的相符項目」。 在呼叫端點的聊天機器人和應用程式的程式碼中，您可以覆寫這個預設回應。 或者，您也可以在 Azure 中設定覆寫回應，這會變更特定 QnA Maker 服務中部署的所有知識庫預設值。

1. 移至 [Azure 入口網站](http://portal.azure.com)，並瀏覽到代表您建立的 QnA Maker 服務之用的資源群組。

2. 按一下以開啟 **App Service**。

    ![存取 App Service](../media/qnamaker-concepts-confidencescore/set-default-response.png)

3. 按一下 [應用程式設定]，並編輯 [DefaultAnswer] 欄位成為所需的預設回應。 按一下 [檔案] 。

    ![變更預設回應](../media/qnamaker-concepts-confidencescore/change-response.png)

4. 重新啟動您的應用程式服務

    ![QnA Maker 應用程式服務重新啟動](../media/qnamaker-faq/qnamaker-appservice-restart.png)


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [支援的資料來源](./data-sources-supported.md)

## <a name="see-also"></a>另請參閱 

[QnA Maker 概觀](../Overview/overview.md)
