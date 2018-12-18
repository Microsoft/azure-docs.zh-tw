---
title: 在 LUIS 應用程式中新增意圖
titleSuffix: Azure Cognitive Services
description: 將意圖新增至 LUIS 應用程式，以識別具有相同用意的問題或命令群組。
services: cognitive-services
author: diberry
manager: cgronlun
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.service: cognitive-services
ms.openlocfilehash: 9b7207a16a89e48ad64b2dbc48a5293d0cf8aa25
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036077"
---
# <a name="manage-intents"></a>管理意圖 
將[意圖](luis-concept-intent.md)新增至 LUIS 應用程式，以識別具有相同意圖的問題或命令群組。 

從頂端工具列中的 [建置] 區段管理意圖。 您可以從左面板中的 [意圖] 頁面新增及管理意圖。 

下列程序示範如何在 TravelAgent 應用程式中新增 "Bookflight" 意圖。

## <a name="add-intent"></a>新增意圖

1. 在 [我的應用程式] 頁面上按一下您的應用程式名稱加以開啟 (例如，TravelAgent)，然後按一下左方面板中的 [意圖]。 
2. 在 [意圖] 頁面上，按一下 [建立新意圖]。

3. 在 [建立新的意圖] 對話方塊中，輸入意圖名稱 "BookFlight"，然後按一下 [完成]。

    ![新增意圖](./media/luis-how-to-add-intents/Addintent-dialogbox.png)

    在新增意圖的意圖詳細資料頁面上，[新增語句](#add-an-utterance-on-intent-page)。

## <a name="rename-intent"></a>將意圖重新命名

1. 在 [意圖] 頁面上，按一下意圖名稱旁邊的 [重新命名] 圖示![將意圖重新命名](./media/luis-how-to-add-intents/Rename-Intent-btn.png)。 

2. 在 [意圖] 頁面上，目前的意圖名稱會顯示在對話方塊中。 編輯意圖名稱，然後按 Enter 鍵。 新名稱會儲存並顯示在意圖頁面上。

    ![編輯意圖](./media/luis-how-to-add-intents/EditIntent-dialogbox.png)

## <a name="delete-intent"></a>刪除意圖
刪除 [無] 意圖以外的意圖時，您可以選擇將所有語句新增至 [無] 意圖。 如果您需要移動語句而不是刪除語句，這極為實用。   

1. 在 [意圖] 頁面上，按一下意圖名稱右邊的 [刪除意圖] 按鈕。 

    ![刪除意圖按鈕](./media/luis-how-to-add-intents/DeleteIntent.png)

2. 在確認對話方塊上按一下 [是]。

<!--
    TBD: waiting for confirmation about which delete dialog is going to be in //BUILD

    ![Delete Intent Dialog](./media/luis-how-to-add-intents/DeleteIntent-Confirmation.png)
-->


## <a name="add-an-utterance-on-intent-page"></a>在意圖頁面上新增語句

在意圖頁面上，於意圖名稱下面的文字方塊中輸入您預期來自使用者的相關語句，例如 `book 2 adult business tickets to Paris tomorrow on Air France`，然後按 Enter 鍵。 
 
>[!NOTE]
>LUIS 會將所有語句轉換成小寫。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示語句)](./media/luis-how-to-add-intents/add-new-utterance-to-intent.png) 

語句會新增至目前意圖的語句清單。 新增語句後，[標示語句內的任何實體](luis-how-to-add-example-utterances.md)，然後[訓練](luis-how-to-train.md)您的應用程式。 

## <a name="create-a-pattern-from-an-utterance"></a>從語句建立模式
請參閱[在意圖或實體頁面上從現有的語句新增模式](luis-how-to-model-intent-pattern.md#add-pattern-from-existing-utterance-on-intent-or-entity-page)。

## <a name="edit-an-utterance-on-intent-page"></a>在意圖頁面上編輯語句

若要編輯語句，請選取該語句行右端的省略符號 (***...***) 按鈕，然後選取 [編輯]。 修改文字，然後按鍵盤上的 Enter 鍵。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示省略符號按鈕)](./media/luis-how-to-add-intents/edit-utterance.png) 

## <a name="reassign-utterances-on-intent-page"></a>在意圖頁面上重新指派語句
將一或多個語句重新指派給另一個意圖，即可變更這些語句的意圖。 

若要將單一語句重新指派給不同的意圖，請在語句的資料列右端，選取 [標示的意圖] 資料行底下正確的名稱意圖。 語句會從目前意圖的語句清單中移除。 

![BookFlight 意圖頁面的螢幕擷取畫面 (已選取 [標示的意圖] 資料行下面的語句意圖)](./media/luis-how-to-add-intents/reassign-1-utterance.png)

若要變更數個語句的意圖，請選取語句左邊的核取方塊，然後選取 [重新指派意圖]。 從清單中選取正確的意圖。

![BookFlight 意圖頁面的螢幕擷取畫面 (已核取意圖並醒目提示 [重新指派意圖] 按鈕)](./media/luis-how-to-add-intents/delete-several-utterances.png) 

## <a name="delete-utterances-on-intent-page"></a>在意圖頁面上刪除語句

若要刪除語句，請選取該語句行右端的省略符號 (***...***) 按鈕，然後選取 [刪除]。 此語句會從清單和 LUIS 應用程式中移除。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 [刪除] 選項)](./media/luis-how-to-add-intents/delete-utterance-ddl.png)

若要刪除數個語句：

1. 選取語句左邊的核取方塊，然後選取 [刪除語句]。 

    ![意圖詳細資料頁面的螢幕擷取畫面 (已核取語句並醒目提示 [刪除語句] 按鈕)](./media/luis-how-to-add-intents/delete-several-utterances.png)

2. 選取 [刪除語句?] 快顯對話方塊中的 [完成]。

## <a name="search-in-utterances-on-intent-page"></a>在意圖頁面上搜尋語句
您可以在意圖的語句清單中搜尋包含文字 (字組或片語) 的語句。 例如，您可能會注意到牽涉特定單字的錯誤，而想要尋找包含該特定單字的所有範例。 

1. 在工具列中選取放大鏡圖示。

    ![意圖頁面的螢幕擷取畫面 (已醒目提示放大鏡的搜尋圖示)](./media/luis-how-to-add-intents/magnifying-glass.png)

2. 搜尋文字方塊隨即出現。 在語句清單右上角的搜尋方塊中輸入單字或片語。 語句清單會更新，僅顯示包含您搜尋文字的語句。 

    ![意圖頁面的螢幕擷取畫面 (已醒目提示搜尋文字方塊)](./media/luis-how-to-add-intents/search-textbox.png)

    若要取消搜尋並還原完整的語句清單，請刪除您已輸入的搜尋文字。 若要關閉搜尋文字方塊，請在工具列中再次選取放大鏡圖示。

## <a name="prediction-discrepancy-errors-on-intent-page"></a>意圖頁面上的預測差異錯誤
意圖中的語句在所選意圖與預測分數之間，可能會有差異。 LUIS 會以紅色方塊圍住分數來指出此差異。 

![BookFlight 意圖頁面的螢幕擷取畫面 (已醒目提示預測差異分數)](./media/luis-how-to-add-intents/score-discrepancy.png) 

## <a name="filter-by-intent-prediction-discrepancy-errors-on-intent-page"></a>在意圖頁面上依意圖預測差異錯誤篩選
若要將語句清單篩選為僅限具有意圖預測差異的語句，在工具列中從 [全部顯示] 切換至 [僅限錯誤]。 

## <a name="filter-by-entity-type-on-intent-page"></a>在意圖頁面上依實體類型篩選
使用工具列上的 [實體篩選條件] 下拉式清單，依照實體篩選語句。 

![意圖頁面的螢幕擷取畫面 (已醒目提示實體類型篩選條件)](./media/luis-how-to-add-intents/filter-by-entities.png) 

若要移除篩選條件，請選取工具列底下含有該字詞或片語的藍色篩選方塊。  
<!-- TBD: waiting for ux fix - bug in ux of prebuit entity number -- when filtering by it, it doesn't show the list -->

## <a name="switch-to-token-view-on-intent-page"></a>在意圖頁面上切換至語彙基元檢視
切換 [語彙基元檢視] 以檢視語彙基元，而非實體類型名稱。 在鍵盤上，您也可以使用 **CTRL + E** 來切換檢視。 

![意圖頁面的螢幕擷取畫面 (已醒目提示 [語彙基元檢視])](./media/luis-how-to-add-intents/toggle-tokens-view.png)

## <a name="train-your-app-after-changing-model-with-intents"></a>在利用意圖變更模型後訓練您的應用程式
在您新增、編輯或移除意圖之後，[訓練](luis-how-to-train.md)並[發佈](luis-how-to-publish-app.md)您的應用程式，變更才會影響端點查詢。 

## <a name="next-steps"></a>後續步驟

將意圖新增至您的應用程式之後，下一項工作就是開始為您所新增的意圖新增[範例語句](luis-how-to-add-example-utterances.md)。 
