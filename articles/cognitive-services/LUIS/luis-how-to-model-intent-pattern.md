---
title: 在 LUIS 應用程式中新增模式範本而非更多語句
titleSuffix: Azure Cognitive Services
description: 了解如何在 Language Understanding (LUIS) 應用程式中新增模式範本，以改善預測準確度。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.technology: luis
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: c97092385cdc28ba6e4d916ae8039f065f97d31d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47030807"
---
# <a name="how-to-add-patterns-to-improve-prediction-accuracy"></a>如何新增模式以改善預測準確度
在 LUIS 應用程式收到端點語句之後，請使用模式的[概念](luis-concept-patterns.md)，以針對在字組順序和字組選擇中顯露模式的語句，改善預測準確度。 模式使用[實體](luis-concept-entity-types.md)及其角色，擷取使用特定模式語法的資料。 

## <a name="add-template-utterance-to-create-pattern"></a>將範本語句新增至建立模式
1. 在 [我的應用程式]頁面上選取您的應用程式名稱加以開啟，然後在 [改善應用程式效能] 之下，選取左面板中的 [模式]。

    ![模式清單的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-1.png)

2. 選取模式的正確意圖。 

    ![選取意圖](./media/luis-how-to-model-intent-pattern/patterns-2.png)

3. 在範本文字方塊中，輸入範本語句並選取 Enter。 當您想要輸入實體名稱時，請使用正確的模式實體語法。 實體語法的開頭為 `{`。 實體清單隨即顯示。 選取正確的實體，然後選取 Enter。 

    ![模式的實體螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-3.png)

    如果您的實體在實體名稱之後包含一個角色 (以一個冒號 `:` 表示該角色)，例如 `{Location:Origin}`。 實體的角色清單會顯示在清單中。 選取角色，然後選取 Enter。 

    ![實體與角色的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-4.png)

    在您選取正確的實體之後，完成模式輸入，然後選取 Enter。 當您輸入好模式時，請[訓練](luis-how-to-train.md)您的應用程式。

    ![包含兩種實體類型的已輸入模式螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-5.png)

## <a name="search-patterns"></a>搜尋模式
搜尋可讓您找出包含某些指定文字的模式。  

1. 選取放大鏡圖示。

    ![模式頁面的螢幕擷取畫面 (已醒目提示搜尋工具圖示)](./media/luis-how-to-model-intent-pattern/search-icon.png)

    在模式清單右上角的搜尋方塊中輸入搜尋文字，然後選取 Enter。 模式清單會更新，只顯示包含搜尋文字的模式。

    ![意圖頁面的螢幕擷取畫面 (已醒目提示搜尋方塊中的搜尋文字)](./media/luis-how-to-model-intent-pattern/search-text.png)

    若要取消搜尋並還原完整的模式清單，請刪除您已輸入的搜尋文字。

<!-- TBD: should I be able to click on the magnifying glass again to close the search box? It doesn't reset the list. -->

## <a name="edit-a-pattern"></a>編輯模式
1. 若要編輯模式，請選取該模式行右端的省略符號 (***...***) 按鈕，然後選取 [編輯]。 

    ![模式列中編輯功能表項目的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-three-dots.png) 

2. 在文字方塊中輸入任何變更。 完成後，選取 Enter。 當您編輯好模式時，請[訓練](luis-how-to-train.md)您的應用程式。

    ![編輯模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/edit-pattern.png)

## <a name="reassign-individual-pattern-to-different-intent"></a>將個別模式重新指派至不同的意圖

若要將單一模式重新指派至不同的意圖，請選取模式文字右邊的意圖清單方塊，然後選取不同的意圖。

![將個別模式重新指派至不同意圖的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/reassign-individual-pattern.png)

## <a name="reassign-several-patterns-to-different-intent"></a>將數個模式重新指派至不同的意圖

若要將數個模式重新指派至不同的意圖，請選取每個模式左邊的核取方塊或選取頂端核取方塊。 [重新指派意圖] 選項會顯示在工具列上。 選取模式的正確意圖。 

![將數個模式重新指派至不同意圖的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/reassign-many-patterns.png)

## <a name="delete-a-single-pattern"></a>刪除單一模式

1. 若要刪除模式，請選取該模式行右端的省略符號 (***...***) 按鈕，然後選取 [刪除]。 

    ![刪除語句的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/patterns-three-dots-ddl.png)

2. 選取 [確定] 來確認刪除。

    ![刪除確認的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/confirm-delete.png)

## <a name="delete-several-patterns"></a>刪除數個模式

1. 若要刪除數個模式，請選取每個模式左邊的核取方塊，或選取頂端核取方塊。 [刪除模式] 選項會顯示在工具列上。 選取 [刪除模式]。  

    ![刪除數個模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/delete-many-patterns.png)

2. [刪除模式] 確認對話方塊隨即出現。 選取 [確定] 以完成刪除。

    ![刪除數個模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/delete-many-patterns-confirmation.png)

## <a name="filter-pattern-list-by-entity"></a>依實體篩選模式清單

若要依照特定實體篩選模式清單，請選取工具列中模式上方的 [實體篩選條件]。 

![依實體篩選模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/filter-entities-1.png)

套用篩選條件之後，實體名稱會出現在工具列下方。 

## <a name="filter-pattern-list-by-intent"></a>依意圖篩選模式清單

若要依照特定意圖篩選模式清單，請選取工具列中模式上方的 [意圖篩選條件]。 

![依意圖篩選模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/filter-intents-1.png)

套用篩選條件之後，意圖名稱會出現在工具列下方。 

## <a name="remove-entity-or-intent-filter"></a>移除實體或意圖篩選條件
篩選模式清單後，實體或意圖名稱會出現在工具列下方。 若要移除篩選條件，請選取名稱。

![依實體篩選模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/filter-entities-2.png)

系統會移除篩選條件，並顯示所有模式。 

## <a name="add-pattern-from-existing-utterance-on-intent-or-entity-page"></a>在意圖或實體頁面上從現有的語句新增模式
您可以在 [意圖] 或 [實體] 頁面上從現有的語句新增模式。 任何意圖或實體頁面上的所有語句都會顯示在一份清單中，其右邊資料行可供存取語句層級的選項，例如 [編輯]、[刪除] 和 [新增為模式]。

1. 在語句的所選資料列上，選取語句右邊的省略符號 (***...***) 按鈕，然後選取 [新增為模式]。

    [![](./media/luis-how-to-model-intent-pattern/add-pattern-from-utterance.png "語句資料表的螢幕擷取畫面 (已醒目提示 [選項] 功能表中的 [新增模式])")](./media/luis-how-to-model-intent-pattern/add-pattern-from-utterance.png)

2. 根據[語法規則](luis-concept-patterns.md#pattern-syntax)修改模式。 如果您選取的語句已標示實體，則這些實體已經為具有正確語法的模式。

    ![依實體篩選模式的螢幕擷取畫面](./media/luis-how-to-model-intent-pattern/confirm-patterns-modal.png)

## <a name="train-your-app-after-changing-model-with-patterns"></a>在利用模式變更模型後訓練您的應用程式
在您新增、編輯、移除或重新指派模式之後，[訓練](luis-how-to-train.md)並[發佈](luis-how-to-publish-app.md)您的應用程式，您的變更才會影響端點查詢。 

## <a name="next-steps"></a>後續步驟

* 了解如何使用 pattern.any 和角色[建立模式](luis-tutorial-pattern.md)。
* 了解如何[訓練](luis-how-to-train.md)您的應用程式。