---
title: 快速入門：建立第一個 Bing 自訂搜尋執行個體
titlesuffix: Azure Cognitive Services
description: 若要使用 Bing 自訂搜尋，必須建立可定義檢視或網路配量的自訂搜尋執行個體。 執行個體包含指定您要 Bing 搜尋的公用網域、子網站和網頁的設定，以及任何順位調整。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: quickstart
ms.date: 05/07/2017
ms.author: v-brapel
ms.openlocfilehash: 866d32aa4de45076fcbc4e413d8c2e67d5346878
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816215"
---
# <a name="quickstart-create-your-first-bing-custom-search-instance"></a>快速入門：建立您的第一個 Bing 自訂搜尋執行個體
若要使用 Bing 自訂搜尋，必須建立可定義檢視或網路配量的自訂搜尋執行個體。 執行個體包含指定您要 Bing 搜尋的公用網域、網站和網頁的設定，以及任何順位調整。 若要建立執行個體，請使用 Bing 自訂搜尋[入口網站](https://customsearch.ai)。 

## <a name="create-a-custom-search-instance"></a>建立自訂搜尋執行個體

建立 Bing 自訂搜尋執行個體：

1.  取得自訂搜尋 API 的金鑰。 請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)。
2.  按一下 [登入] 按鈕並使用 Microsoft 帳戶 (MSA) 登入入口網站。 
    - 如果您沒有 MSA，請按一下 [建立 Microsoft 帳戶]。 入口網站會要求您提供存取資料的權限。 按一下 [是] 。
    - 同意認知服務的條款。 選取 [我同意] 並按一下 [同意]。  
3.  登入之後，按一下 [新執行個體]，並為執行個體命名。 使用有意義的名稱，且名稱能夠描述搜尋傳回的內容類型。 您可以隨時變更名稱。 
4.  在 [搜尋體驗] 的 [使用中] 索引標籤下，輸入您要在搜尋中包括的一或多個網站的 URL。
5.  若要確認您的執行個體會傳回結果，請在右邊的預覽窗格中輸入查詢。 如果沒有任何結果，請指定新網站。 Bing 只會針對已編製索引的公用網站傳回結果。
6.  按一下 [發佈]，將設定變更發佈至生產環境。 出現提示時，請按一下 [發佈] 確認。
7.  按一下 [生產] > [端點]，複製 [自訂組態識別碼]。 您需要這個識別碼，才能呼叫自訂搜尋 API。

## <a name="next-steps"></a>後續步驟

依照下列使用說明指南中的指示，繼續使用您剛才建立的自訂搜尋執行個體：

- [設定您的自訂搜尋體驗](./define-your-custom-view.md)
- [呼叫您的自訂搜尋](./search-your-custom-view.md)
- [共用您的自訂搜尋](./share-your-custom-search.md)
- [設定託管的 UI 體驗](./hosted-ui.md)
- [使用裝飾標記醒目提示文字](./hit-highlighting.md)
- [頁面網頁](./page-webpages.md)
