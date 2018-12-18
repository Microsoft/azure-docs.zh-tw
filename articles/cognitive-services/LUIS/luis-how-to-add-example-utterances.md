---
title: 在 LUIS 應用程式中新增範例語句
titleSuffix: Azure Cognitive Services
description: 了解如何在 Language Understanding (LUIS) 應用程式中新增語句。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: 387668263a6bab6e12a21adf04aebfbbf108a006
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036485"
---
# <a name="add-example-utterances-and-label-with-entities"></a>使用實體新增範例語句和標籤

範例語句是使用者問題或命令的文字範例。 若要教導 Language Understanding (LUIS)，您需要將[範例語句](luis-concept-utterance.md)新增至[意圖](luis-concept-intent.md)。

一般而言，會先將範例語句新增至意圖，然後在意圖頁面上建立實體和標籤語句。 若您要先建立實體，請參閱[新增實體](luis-how-to-add-entities.md)。

## <a name="add-an-utterance"></a>新增語句
在意圖頁面上，於意圖名稱下面的文字方塊中輸入您預期來自使用者的相關範例語句 (例如 `book 2 adult business tickets to Paris tomorrow on Air France`)，然後按 Enter 鍵。 
 
>[!NOTE]
>LUIS 會將所有語句轉換成小寫。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示語句)](./media/luis-how-to-add-example-utterances/add-new-utterance-to-intent.png) 

語句會新增至目前意圖的語句清單。 

## <a name="ignoring-words-and-punctuation"></a>忽略字組和標點符號
若您想要忽略範例語句中的特定字組或標點符號，請搭配使用[模式](luis-concept-patterns.md#pattern-syntax)與_忽略_語法。 

## <a name="add-simple-entity-label"></a>新增簡單實體標籤
在下列程序中，您會在意圖頁面上建立和標示下列語句內的自訂項目：

```
book me 2 adult business tickets to Paris tomorrow on Air France
```

1. 在語句中選取 [Air France] \(法國航空\)，將它標示為簡單實體。

    > [!NOTE]
    > 選取字組將其標示為實體時：
    > * 對於單一字組，只需要選取它。 
    > * 對於兩個或多個字組的集合，請選取集合開頭與集合結尾。

2. 在出現的實體下拉式清單方塊中，您可以選取現有實體，或新增實體。 若要新增實體，請在文字方塊中輸入其名稱，然後選取 [建立新的實體]。 
 
    ![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示簡單實體標籤選項)](./media/luis-how-to-add-example-utterances/create-airline-simple-entity.png)

3. 在 [What type of entity do you want to create?] \(您要建立哪種類型的實體?\) 快顯對話方塊中，驗證實體名稱，並選取簡單實體類型，然後選取 [完成]。

    ![確認對話方塊的影像](./media/luis-how-to-add-example-utterances/create-simple-airline-entity.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#simple-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取簡單實體。 嘗試簡單實體[快速入門](luis-quickstart-primary-and-secondary-data.md)，以深入了解如何使用簡單實體。


## <a name="add-list-entity-and-label"></a>新增清單實體和標籤
清單實體代表系統中的一組固定封閉 (確切文字相符項目) 相關字組。 

針對 drinks 清單實體，您可以有兩個標準化值：water 和 soda pop。 每個標準化名稱都會有同義字。 對於 water，同義字是 H20、gas、flat。 對於 soda pop，同義字是 fruit、cola、ginger。 當您建立實體時，不需要知道所有值。 檢閱包含同義字的實際使用者語句之後，即可新增其他語句。

|標準化名稱|同義字|
|--|--|
|Water|H20、gas、flat|
|Soda pop|Fruit、cola、ginger|

從意圖頁面建立新的清單實體時，您將會做兩件可能不明顯的事項。 首先，您新增第一個清單項目來建立新清單。 其次，使用您從語句中選取的字組或片語命名第一個清單項目。 雖然您稍後可以從實體頁面變更這些值，但選取包含您想要用於清單項目名稱的字組語句可能更為快速。

例如，若您想要建立一份飲料類型清單，並且已從語句中選取 `h2o` 這個字來建立實體，則這份清單會有一個項目，其名稱為 h20。 若您想要更一般的名稱，則應該選擇使用更一般名稱的語句。 

1. 在語句中，選取清單中第一個項目的字組，並在文字方塊中輸入清單名稱，然後選取 [建立新的實體]。   

    ![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 [建立新的實體])](./media/luis-how-to-add-example-utterances/create-drink-list-entity.png)

2. 在 [What type of entity do you want to create?] \(您要建立哪種類型的實體?\) 對話方塊中，新增此清單項目的同義字。 針對 drink 清單中的 water 項目，新增 `h20`、`perrier` 和 `waters`，然後選取 [完成]。 請注意，已新增 "waters"，因為符合權杖層級的清單同義字。 在英文文化特性中，該層級是字組層級，因此除非 "waters" 在清單中，否則與 "water" 不相符。 

    ![[What type of entity do you want to create?] \(您要建立哪種類型的實體?\) 對話方塊的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/drink-list-ddl.png)

    這份飲料清單只有一種類型的飲料 (water)。 標示其他語句，或從左導覽的 [實體] 中編輯實體，即可新增更多飲料類型。 [編輯](luis-how-to-add-entities.md#add-list-entities)實體可讓您選擇輸入具有對應同義字的其他項目，或[匯入](luis-how-to-add-entities.md#import-list-entity-values)清單。 

    請參閱[資料擷取](luis-concept-data-extraction.md#list-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取清單實體。 嘗試[快速入門](luis-quickstart-intent-and-list-entity.md)，以深入了解如何使用清單實體。

## <a name="add-synonyms-to-the-list-entity"></a>將同義字新增至清單實體 
在語句中選取字組或片語，以將同義字新增至清單實體。 若您有 [Drink] \(飲料\) 清單實體，而且想要將 `agua` 新增為 water 的同義字，請遵循下列步驟：

在語句中，選取同義字 (例如 `aqua` 表示 water)，並在下拉式清單中選取清單實體名稱 (例如 [Drink] \(飲料\))，然後選取 [Set as synonym]\(設定為同義字\)，再選取與其同義的清單項目 (例如 **water**)。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 [Create a new synonym] \(建立新的同義字\))](./media/luis-how-to-add-example-utterances/set-agua-as-synonym.png)

## <a name="create-new-item-for-list-entity"></a>建立清單實體的新項目
在語句中選取字組或片語，以建立現有清單實體的新項目。 若您有 [Drink] \(飲料\)，而且想要將 `tea` 新增為新項目，請遵循下列步驟：

在語句中，選取新清單項目的字組 (例如 `tea`)，並在下拉式清單中選取清單實體名稱 (例如 [Drink] \(飲料\))，然後選取 [Create a new synonym] \(建立新的同義字\)。 

![新增清單項目的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/list-entity-create-new-item.png)

字組現在會以藍色醒目提示。 若您將滑鼠指標停留在該字組上方，則會顯示其中顯示清單項目名稱 (例如 tea) 的標籤。

![新清單項目標籤的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/list-entity-item-name-tag.png)

## <a name="wrap-entities-in-composite-label"></a>將實體包裝在複合標籤中
從**實體**建立複合實體。 您無法從 [意圖] 頁面建立複合實體。 建立複合實體之後，您可以在 [意圖] 頁面上將實體包裝在語句中。 

假設語句 `book 2 tickets from Seattle to Cairo`，複合語句可以傳回單一父實體中票證計數 (2)、來源 (Seattle) 和目的地 (Cairo) 位置的實體資訊。 

遵循這些[步驟](luis-how-to-add-entities.md#add-prebuilt-entity)，新增 **number** 個預先建置的實體。 建立實體之後，語句中的 `2` 會是藍色，指出它是已標示實體。 預先建置的實體由 LUIS 所標示。 您無法從單一語句新增或移除預先建置的實體標籤。 您只能新增或移除應用程式中預先建置的實體，來新增或移除所有預先建置的標籤。

遵循這些[步驟](#add-hierarchical-entity-and-label)，建立**位置**階層式實體。 在範例語句中標示來源和目的地位置。 

將實體包裝在複合實體中之前，請確定以藍色醒目提示所有子實體，表示已在語句中標示它們。

1. 若要將個別實體包裝到複合中，請選取複合實體語句中的第一個已標示實體。 在範例語句 `book 2 tickets from Seattle to Cairo` 中，第一個實體是數字 2。 下拉式清單即會出現，顯示此選項的選擇。

    ![已選取數字和已醒目提示下拉式選項的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/wrap-1.png)

2. 從下拉式清單中選取 [Wrap composite entity] \(包裝複合實體\)。 

    ![用於包裝複合實體之下拉式選項的螢幕擷取畫面 (已醒目提示 [Wrap in composite entity] \(包裝在複合實體中\))](./media/luis-how-to-add-example-utterances/wrap-2.png)

3. 選取複合實體的最後一個字組。 在此範例的語句中，選取 "Location::Destination" (代表 Cairo)。 複合語句的所有字組 (包括非實體字組) 下方現在會有綠色線條。

    ![醒目提示數字之 [BookFlight Intent] \(BookFlight 意圖\) 頁面的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/wrap-composite.png)

4. 從下拉式清單中選取複合實體名稱。 在此範例中會是 **TicketOrder**。

    ![包裝字組與複合實體的螢幕擷取畫面 (在下拉式清單中反白顯示複合實體名稱)](./media/luis-how-to-add-example-utterances/wrap-4.png)

    當您正確地包裝實體時，整個片語下方會有綠色線條。

    ![已醒目提示複合實體之語句的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/wrap-5.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#composite-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取複合實體。 嘗試複合實體[教學課程](luis-tutorial-composite-entity.md)，以深入了解如何使用複合實體。

## <a name="add-hierarchical-entity-and-label"></a>新增階層式實體和標籤
階層式實體是內容學習和內容相關實體的分類。 在下列範例中，實體包含來源和目的地位置。 

在語句 `Book 2 tickets from Seattle to Cairo` 中，Seattle 是來源位置，而 Cairo 是目的地位置。 每個位置的內容都不同，並且學習自語句中的字組順序和字組選擇。

1. 在 [意圖] 頁面上，於語句中選取 [Seattle]，並輸入實體名稱 'Location'，然後選取 [建立新的實體]。

    ![[Create Hierarchical Entity Labeling] \(建立階層式實體標籤\) 對話方塊的螢幕擷取畫面](./media/luis-how-to-add-example-utterances/create-hier-ent-1.png)

2. 在快顯對話方塊中，針對 [實體類型] 選取 [hierarchical] \(階層式\)，並新增 `Origin` 和 `Destination` 作為子系，然後選取 [完成]。

    ![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 ToLocation 實體)](./media/luis-how-to-add-example-utterances/create-location-hierarchical-entity.png)

3. 語句中的字組已標上父階層式實體。 您需要將字組指派給子實體。 返回至意圖頁面上的語句。 選取字組，並從下拉式清單中選擇您所建立的實體名稱，然後遵循右側的功能表來選擇正確的子實體。

    ![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 ToLocation 實體)](./media/luis-how-to-add-example-utterances/label-tolocation.png)

    >[!CAUTION]
    >在單一應用程式的所有實體中，子實體名稱必須是唯一的。 兩個不同的階層式實體不能包含同名的子實體。 

    請參閱[資料擷取](luis-concept-data-extraction.md#hierarchical-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取階層式實體。 嘗試階層式實體[快速入門](luis-quickstart-intent-and-hier-entity.md)，以深入了解如何使用階層式實體。


## <a name="remove-entity-labels-from-utterances"></a>從語句中移除實體標籤
您可以在 [意圖] 頁面上從語句移除機器學習實體標籤。 若實體不是機器學習實體，則無法從語句中予以移除。 若您需要從語句中移除非機器學習實體，則需要刪除整個應用程式中的實體。 

若要從語句中移除機器學習實體標籤，請選取語句中的實體。 然後選取所出現實體下拉式方塊中的 [移除標籤]。

![意圖詳細資料頁面的螢幕擷取畫面 (已醒目提示 [移除標籤])](./media/luis-how-to-add-example-utterances/remove-label.png) 

## <a name="add-prebuilt-entity-label"></a>新增預先建置的實體標籤
若您將預先建置的實體新增至 LUIS 應用程式，則不需要標示具有這些實體的語句。 若要深入了解預先建置的實體和其新增方式，請參閱[新增實體](luis-how-to-add-entities.md#add-prebuilt-entity)。

## <a name="add-regular-expression-entity-label"></a>新增規則運算式實體標籤
若您將規則運算式實體新增至 LUIS 應用程式，則不需要標示具有這些實體的語句。 若要深入了解規則運算式實體和其新增方式，請參閱[新增實體](luis-how-to-add-entities.md#add-regular-expression-entities)。

## <a name="create-a-pattern-from-an-utterance"></a>從語句建立模式
請參閱[在意圖或實體頁面上從現有的語句新增模式](luis-how-to-model-intent-pattern.md#add-pattern-from-existing-utterance-on-intent-or-entity-page)。

## <a name="add-patternany-entity-label"></a>新增 pattern.any 實體標籤
若您將 pattern.any 實體新增至 LUIS 應用程式，則無法標示具有這些實體的語句。 它們只有在模式中有效。 若要深入了解 pattern.any 實體和其新增方式，請參閱[新增實體](luis-how-to-add-entities.md#add-patternany-entities)。

<!--
Fix this - moved to luis-how-to-add-intents.md - how ?

## Search in utterances
## Prediction discrepancy errors
## Filter by intent prediction discrepancy errors
## Filter by entity type
## Switch to token view
## Delete utterances
## Edit an utterance
## Reassign utterances

-->
## <a name="train-your-app-after-changing-model-with-utterances"></a>在利用語句變更模型後訓練您的應用程式
在您新增、編輯或移除語句之後，[訓練](luis-how-to-train.md)並[發佈](luis-how-to-publish-app.md)您的應用程式，您的變更才會影響端點查詢。 

## <a name="next-steps"></a>後續步驟

在意圖中標示語句之後，您現在可以建立[複合實體](luis-how-to-add-entities.md)。
