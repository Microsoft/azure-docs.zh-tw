---
title: 在 LUIS 應用程式中新增實體
titleSuffix: Azure Cognitive Services
description: 在 Language Understanding (LUIS) 應用程式中新增實體 (您應用程式定義域中的關鍵資料)。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: e82955da24e127e5536c2e40ad2cccf07c5fa173
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031997"
---
# <a name="manage-entities"></a>管理實體
識別出應用程式的[意圖](luis-concept-intent.md)之後，您需要使用[實體](luis-concept-entity-types.md)來[標示範例語句](luis-concept-utterance.md)。 實體是命令或問題的重要部分，對於您的用戶端應用程式執行其工作而言可能是不可或缺的。 

您可以在應用程式中，透過 [實體] 頁面上的 [實體清單] 來新增、編輯或刪除實體。 LUIS 提供兩種主要類型的實體：[預先建置的實體](luis-reference-prebuilt-entities.md)，以及您自己的自訂實體。

下列各節僅能從 LUIS 應用程式內部的 [建置] 區段中取得。 [建置] 連結位於上方瀏覽列中。 位於 [建置] 區段內部之後，從左側瀏覽功能表中選取 [實體]。 將實體新增到應用程式之後，如果實體是機器學習的，您就能在語句內[標示實體](luis-how-to-add-example-utterances.md)。 將應用程式定型且發佈之後，您就會收到[擷取](luis-concept-data-extraction.md)自預測的實體資料。 

## <a name="add-prebuilt-entity"></a>新增預先建置的實體
預先建置的實體定義於 [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 專案中。 新增到應用程式的常見預先建置實體為 *number* 和 *datetimeV2*。 

1. 在應用程式的 [建置] 區段中，接著按一下左面板中的 [實體]。
 
2. 在 [實體] 頁面上，選取 [管理預先建置的實體]。

3. 在 [新增或移除預先建置的實體] 對話方塊中，選取 [number] 和 [datetimeV2] 預先建置的實體。 然後選取 [完成]。

    ![[新增預先建置的實體] 對話方塊的螢幕擷取畫面](./media/add-entities/list-of-prebuilt-entities.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#prebuilt-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取預先建置的實體。

## <a name="add-simple-entities"></a>新增簡單實體
簡單實體是描述單一概念的一般實體。 

1. 在應用程式的 [建置] 區段中，接著按一下左面板中的 [實體]，然後選取 [建立新實體]。

2. 在快顯對話方塊中，於 [實體名稱] 方塊中輸入 `Airline`、從 [實體類型] 清單中選取 [簡單]，然後選取 [完成]。

    ![建立 Airline 簡單實體的對話方塊螢幕擷取畫面](./media/add-entities/create-simple-airline-entity.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#simple-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取簡單實體。 嘗試簡單實體[快速入門](luis-quickstart-primary-and-secondary-data.md)，以深入了解如何使用簡單實體。

## <a name="add-regular-expression-entities"></a>新增規則運算式實體
規則運算式實體可根據您提供的規則運算式，用來從語句中提取出資料。 

1. 在應用程式中，從左側瀏覽列選取 [實體]，然後選取 [建立新實體]。

2. 在快顯對話方塊中，於 [實體名稱] 方塊中輸入 `AirFrance Flight`、從 [實體類型] 清單中選取 [規則運算式]、輸入規則運算式 `AFR[0-9]{3,4}`，然後選取 [完成]。 

    這個 AirFrance Flight 規則運算式預期要有三個字元 (依照字面意思為 `AFR`)，後面接著 3 或 4 個數字。 數字可以是介於 0 到 9 之間的任何數字。 規則運算式會比對 AirFrance 班機號碼，例如："AFR101"、"ARF1302" 及 "AFR5006"。 請參閱[資料擷取](luis-concept-data-extraction.md)，以深入了解如何從端點 JSON 查詢回應擷取實體。

    ![建立規則運算式實體的對話方塊影像](./media/add-entities/regex-entity-create-dialog.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#regular-expression-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取規則運算式實體。 嘗試使用規則運算式實體[快速入門](luis-quickstart-intents-regex-entity.md)，以深入了解如何使用規則運算式實體。

## <a name="add-hierarchical-entities"></a>新增階層式實體
階層式實體是內容學習和內容相關實體的分類。 在下列範例中，實體包含來源和目的地位置。 

在語句 `Book 2 tickets from Seattle to Cairo` 中，Seattle 是來源位置，而 Cairo 是目的地位置。 每個位置的內容都不同，並且學習自語句中的字組順序和字組選擇。

若要新增階層式實體，請完成下列步驟： 

1. 在應用程式中，從左側瀏覽列選取 [實體]，然後選取 [建立新實體]。

2. 在快顯對話方塊中，於 [實體名稱] 方塊中輸入 `Location`，然後從 [實體類型] 清單中選取 [階層式]。

    ![新增階層式實體](./media/add-entities/hier-location-entity-creation.png)

3. 選取 [新增子系]，然後在 [第一個子系] 方塊中輸入 "Origin" (出發地)。 

4. 選取 [新增子系]，然後在 [第二個子系] 方塊中輸入 "Destination" (目的地)。 選取 [完成] 。
5. 
    >[!NOTE]
    >若要刪除子系，請選取它旁邊的垃圾桶圖示。

    >[!CAUTION]
    >在單一應用程式的所有實體中，子實體名稱必須是唯一的。 兩個不同的階層式實體不能包含同名的子實體。 

    請參閱[資料擷取](luis-concept-data-extraction.md#hierarchical-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取階層式實體。 嘗試階層式實體[快速入門](luis-quickstart-intent-and-hier-entity.md)，以深入了解如何使用階層式實體。

## <a name="add-composite-entities"></a>新增複合實體
您可以藉由建立複合實體，在數個現有實體之間定義關聯性。 在下列範例中，此實體包含機票數目、出發地和目的地位置。 

在語句 `Book 2 tickets from Seattle to Cairo` 中，數字 2 與預先建置的實體相符、Seattle 是出發地位置，而 Cairo 是目的地位置。 建立複合實體之後，每個實體都是較大父實體的一部分。

1. 在您的應用程式中，新增預先建置的實體 **number**。 如需相關指示，請參閱[新增預先建置的實體](#add-prebuilt-entity)。 

2. 新增階層式實體 `Location`，包括子類型：`origin`、`destination`。 如需詳細指示，請參閱[新增階層式實體](#add-hierarchical-entities)。 

3. 從左側瀏覽列選取 [實體]，然後選取 [建立新實體]。

4. 在快顯對話方塊中，於 [實體名稱] 方塊中輸入 `TicketsOrder`，然後從 [實體類型] 清單中選取 [複合]。

5. 選取 [新增子系] 來以新增子系。

6. 在 [第一個子系] 中，從清單選取實體 **number**。

7. 在 [第二個子系] 中，從清單選取實體 **Location::Origin**。 

8. 在 [第三個子系] 中，從清單選取實體 **Location::Destination**。 

9. 選取 [完成] 。

    ![建立複合實體的對話方塊影像](./media/add-entities/ticketsorder-composite-entity.png)

    >[!NOTE]
    >若要刪除子系，選取它旁邊的垃圾桶按鈕。

    請參閱[資料擷取](luis-concept-data-extraction.md#composite-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取複合實體。 嘗試複合實體[教學課程](luis-tutorial-composite-entity.md)，以深入了解如何使用複合實體。


## <a name="add-patternany-entities"></a>新增 pattern.any 實體
[Pattern.any](luis-concept-entity-types.md) 實體只有在[模式](luis-how-to-model-intent-pattern.md)中有效。 此實體可協助 LUIS 尋找變動長度與文字選項的實體結尾。 由於此實體會在模式中使用，因此，LUIS 就能得知實體結尾是在語句中的何處。

如果應用程式有 `FindBookInfo` 意圖，則書名可能會干擾意圖預測。 若要釐清書名中有哪些文字，請在模式內使用 Pattern.any。 LUIS 預測會從語句開始。 首先，針對實體檢查語句並加以比對，找到實體時，接著會檢查並比對模式。 

在語句 `Who wrote the book Ask and when was it published?` 中，書名 Ask 有點棘手，因為在上下文中標題結束的地方與語句其餘部分開始的地方並不明顯。 書名可以包括任何順序的文字，包括單一文字、具有標點符號的複雜片語，以及以無意義方式排序的字組。 模式讓您能夠建立可擷取完整又準確之實體的實體。 一旦找到書名之後，就會預測 `FindBookInfo` 意圖，因為那是模式的意圖。

1. 在應用程式的 [建置] 區段中，接著按一下左面板中的 [實體]，然後選取 [建立新實體]。

2. 在 [新增實體] 對話方塊中，於 [實體名稱] 方塊中輸入 `BookTitle`，然後選取 [Pattern.any] 作為**實體類型**。
 
    ![建立 pattern.any 實體的螢幕擷取畫面](./media/add-entities/create-pattern-any-entity.png)

    若要使用 pattern.any 實體，在 [模式] 頁面 (位於 [改善應用程式效能] 區段) 搭配正確的大括號語法來新增模式，例如「{BookTitle}」的作者是誰？」。

    請參閱[資料擷取](luis-concept-data-extraction.md#patternany-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取 Pattern.any 實體。 請嘗試[模式](luis-tutorial-pattern.md)教學課程，以深入了解如何使用 Pattern.any 實體。

如果您發現模式在包含 Pattern.any 時所擷取的實體不正確，請使用[明確清單](luis-concept-patterns.md#explicit-lists)來更正此問題。 

## <a name="add-role-to-pattern-based-entity"></a>將角色新增到以模式為基礎的實體
角色是以內容為基礎之實體的具名子類型。 它相當於[階層式](#add-hierarchical-entities)實體，但角色只適用於[模式](luis-how-to-model-intent-pattern.md)。 

例如，機票含有「出發城市」和「目的地城市」，但兩者都是城市。 LUIS 會判斷這兩者都是城市，而且可以根據文字順序和文字選項的內容來判斷出發和目的地城市。 

角色的語法為 **{Entity name:Role name}**，其中的實體名稱後面會接著冒號，然後是角色名稱。 例如，「預訂從 {Location:Origin} 到 {Location:Destination} 的機票」。

1. 在應用程式的 [建置] 區段中，接著選取左面板中的 [實體]。

2. 選取 [Create new entity] \(建立新實體\)。 輸入 `Location` 的名稱。 選取 [簡單] 類型，然後選取 [完成]。

3. 從左面板中選取 [實體]，然後選取在步驟 2 中建立的新實體 **Location**。

4. 在 [角色名稱] 文字方塊中，輸入角色 `Origin` 的名稱，然後按 Enter。 新增 `Destination` 的第二個角色名稱。 例如，航程具有出發和目的地城市。 這兩個角色為 "Origin" 和 "Destination"。

    ![將 Origin 角色新增到 Location 實體的螢幕擷取畫面](./media/add-entities/roles-enter-role-name-text.png)

    請參閱[資料擷取](luis-concept-data-extraction.md)，以深入了解如何從端點 JSON 查詢回應擷取角色。 請嘗試模式教學課程，以深入了解如何使用 Pattern.any 實體。

## <a name="add-list-entities"></a>新增清單實體
清單實體代表系統中一組固定且封閉的相關文字組。 

針對 drinks 清單實體，您可以有兩個標準化值：water 和 soda pop。 每個標準化名稱都會有同義字。 對於 water，同義字是 H20、gas、flat。 對於 soda pop，同義字是 fruit、cola、ginger。 當您建立實體時，不需要知道所有值。 檢閱包含同義字的實際使用者語句之後，即可新增其他語句。

|標準化名稱|同義字|
|--|--|
|Water|H20、gas、flat|
|Soda pop|Fruit、cola、ginger|

1. 在應用程式的 [建置] 區段中，接著按一下左面板中的 [實體]，然後選取 [建立新實體]。

2. 在 [新增實體] 對話方塊中，於 [實體名稱] 方塊中輸入 `Drinks`，然後選取 [清單] 作為**實體類型**。 選取 [完成] 。
 
    ![建立 Drinks 清單實體的對話方塊影像](./media/add-entities/menu-list-dialog.png)
  
3.  清單實體頁面可讓您新增標準化名稱。 在 [值] 文字方塊中，輸入適用於清單的項目 (例如，適用於 Drinks 清單的 `Water`)，然後按鍵盤上的 Enter。 

    ![在文字方塊中具有 Water 標準化值之 Drinks 清單實體的螢幕擷取畫面](./media/add-entities/entity-list-normalized-name.png)

4. 在標準化值 **Water** 的右邊，輸入同義字 `h20`、`flat` 及 `gas`，輸入每個項目之後按鍵盤上的 Enter。

    ![反白顯示 Water 同義字項目之 Drinks 清單實體的螢幕擷取畫面](./media/add-entities/menu-list-synonyms.png)

5. 如果您想要更多適用於該清單的標準化項目，請選取 [建議] 以查看[語意字典](luis-glossary.md#semantic-dictionary)的選項。

    ![已反白顯示建議項目之 Drinks 清單實體的螢幕擷取畫面](./media/add-entities/entity-list-recommended-list.png)

6. 選取建議清單中的項目以將它新增為標準化的值，或選取 [全部新增] 以新增所有項目。 

    ![反白顯示 beer 建議項目和 [全部新增] 按鈕之 Drinks 清單實體的螢幕擷取畫面](./media/add-entities/list-entity-add-suggestions.png)

    請參閱[資料擷取](luis-concept-data-extraction.md#list-entity-data)，以深入了解如何從端點 JSON 查詢回應擷取清單實體。 嘗試[快速入門](luis-quickstart-intent-and-list-entity.md)，以深入了解如何使用清單實體。

## <a name="import-list-entity-values"></a>匯入清單實體值
您可以將值匯入到現有的清單實體。

 1. 在清單實體頁面上，選取 [匯入清單]。

 2. 在 [匯入新項目] 對話方塊中，選取 [選擇檔案]，然後選取包含清單的 JSON 檔案。

    ![[匯入清單實體值] 快顯對話方塊的螢幕擷取畫面](./media/add-entities/menu-list-import-json-dialog-with-file.png)

    >[!NOTE]
    >LUIS 只會匯入副檔名為 ".json" 的檔案。

 3. 若要了解 JSON 中支援的清單語法，請選取 [了解支援的清單語法] 來展開對話方塊，然後顯示所允許語法的範例。 若要摺疊對話方塊並隱藏語法，請再次選取連結的標題。

 4. 選取 [完成] 。

    適用於 **Colors** 清單實體的有效 JSON 範例會以下列 JSON 格式化的程式碼顯示：

    ```
    [
        {
            "canonicalForm": "Blue",
            "list": [
                "navy",
                "royal",
                "baby"
            ]
        },
        {
            "canonicalForm": "Green",
            "list": [
                "kelly",
                "forest",
                "avacado"
            ]
        }
    ]  
    ```

## <a name="edit-entity-name"></a>編輯實體名稱
1. 在 [實體] 清單頁面上，選取清單中的實體。 此動作會帶您前往 [實體] 頁面。

2. 在 [實體] 頁面上，您可以藉由選取實體名稱旁邊的編輯圖示來編輯實體名稱。 實體類型是不可編輯的。 

## <a name="delete-entity"></a>刪除實體

在 [實體] 頁面上，選取 [刪除實體] 按鈕。 接著，在確認訊息中選取 [確定] 以確認刪除。
 
![反白顯示 [刪除實體] 按鈕之 Location 實體頁面的螢幕擷取畫面](./media/add-entities/entity-delete.png)

>[!NOTE]
>* 刪除階層式實體，即會刪除它的所有子系實體。
>* 刪除複合實體，只會刪除該複合並中斷複合關聯性，但不會刪除構成它的實體。

## <a name="changing-entity-type"></a>變更實體類型
LUIS 不允許您變更實體的類型，因為它不知道要新增或移除哪些項目來建構該實體。 若要變更類型，更好的方式是使用稍微不同的名稱來建立正確類型的新實體。 一旦建立實體之後，在每個語句中，移除已加上標籤的舊實體名稱，並新增新的實體名稱。 為所有語句加上標籤之後，刪除舊實體。 

## <a name="create-a-pattern-from-an-utterance"></a>從語句建立模式
請參閱[在意圖或實體頁面上從現有的語句新增模式](luis-how-to-model-intent-pattern.md#add-pattern-from-existing-utterance-on-intent-or-entity-page)。

## <a name="search-utterances"></a>搜尋語句
您可以使用工具列上的放大鏡圖示來搜尋和篩選語句。 

## <a name="train-your-app-after-changing-model-with-entities"></a>利用實體變更模型後將您的應用程式定型
在新增、編輯或移除實體之後，請將您的應用程式[定型](luis-how-to-train.md)並[發佈](luis-how-to-publish-app.md)，變更才會影響端點查詢。 

## <a name="next-steps"></a>後續步驟
既然您已新增意圖、語句和實體，您就會擁有基本的 LUIS 應用程式。 了解如何為您的應用程式[定型](luis-how-to-train.md)、[測試](luis-interactive-test.md)及[發佈](luis-how-to-publish-app.md)。
 
