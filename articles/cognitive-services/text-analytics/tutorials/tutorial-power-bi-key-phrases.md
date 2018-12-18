---
title: 教學課程：搭配 Power BI 使用文字分析
titleSuffix: Azure Cognitive Services
description: 了解如何使用文字分析，以從儲存在 Power BI 的文字中擷取關鍵片語。
services: cognitive-services
author: luiscabrer
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: tutorial
ms.date: 09/12/2018
ms.author: luisca
ms.openlocfilehash: fe6bc384e4190cd17df00ddf285701db8c4199a6
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45733324"
---
# <a name="tutorial-integrate-power-bi-with-the-text-analytics-cognitive-service"></a>教學課程：將 Power BI 與文字分析認知服務整合

Microsoft Power BI Desktop 是免費的應用程式，可讓您將您的資料連接、轉換並視覺化。 文字分析服務是 Microsoft Azure 認知服務的一部分，可提供自然語言處理。 假設有未經處理的非結構化文字，它能擷取最重要的片語、分析情感，然後找出已知的實體 (例如樂團)。 搭配使用這些工具，您便可快速了解客戶在說什麼，以及他們對此有何看法。

在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * 使用 Power Bi Desktop 匯入及轉換資料
> * 在 Power BI Desktop 中建立自訂函式
> * 將 Power BI Desktop 與文字分析關鍵片語 API 整合
> * 使用文字分析關鍵片語 API 從客戶意見反應擷取最重要的片語
> * 從客戶意見反應建立文字雲

## <a name="prerequisites"></a>必要條件
<a name="Prerequisites"></a>

- Microsoft Power BI Desktop。 [免費下載](https://powerbi.microsoft.com/get-started/)。
- Microsoft Azure 帳戶。 [開始免費試用](https://azure.microsoft.com/free/)或[登入](https://portal.azure.com/)。
- 具有文字分析 API 的認知服務 API 帳戶。 如果您沒有帳戶，可以[註冊](../../cognitive-services-apis-create-account.md)並使用免費層提供的 5,000 筆交易/月 (請參閱[定價詳細資料](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)以完成此教學課程)。
- 您在註冊過程中為您產生的[文字分析存取金鑰](../how-tos/text-analytics-how-to-access-key.md)。
- 客戶評論。 您可以使用[我們的範例資料](https://aka.ms/cogsvc/ta)，或是自己的資料。 此教學課程假設您使用我們的範例資料。

## <a name="load-customer-data"></a>載入客戶資料
<a name="LoadingData"></a>

若要開始，請開啟 Power BI Desktop，並載入您在[必要條件](#Prerequisites)中下載的逗點分隔值 (CSV) 檔案 `FabrikamComments.csv`。 這個檔案代表一家小型虛構公司的支援論壇中，一天之內的假設性活動。

> [!NOTE]
> Power BI 可以使用各種來源的資料，例如 Facebook 或 SQL 資料庫。 請深入了解 [Facebook 與 Power BI 的整合](https://powerbi.microsoft.com/integrations/facebook/)和 [SQL Server 與 Power BI 的整合](https://powerbi.microsoft.com/integrations/sql-server/)。

在 Power BI Desktop 主視窗中，選取 [常用] 功能區。 在 [外部資料] 功能區群組中，開啟 [取得資料] 下拉式功能表，然後選取 [文字/CSV]。

![[取得資料按鈕]](../media/tutorials/power-bi/get-data-button.png)

[開啟] 對話方塊隨即出現。 瀏覽到您的 [下載] 資料夾，或是您下載 `FabrikamComments.csv` 檔案的資料夾。 按一下 `FabrikamComments.csv`，然後按一下 [開啟] 按鈕。 [CSV 匯入] 對話方塊隨即出現。

![[CSV 匯入對話方塊]](../media/tutorials/power-bi/csv-import.png)

[CSV 匯入] 對話方塊可讓您確認 Power BI Desktop 是否已正確地偵測到字元集、分隔符號、標題資料列和資料行類型。 此資訊都正確，所以按一下 [載入]。

若要查看載入的資料，請按一下 Power BI 工作區左側邊緣的 [資料檢視] 按鈕。 隨即會開啟包含資料的資料表，就和在 Microsoft Excel 中一樣。

![[所匯入資料的初始檢視]](../media/tutorials/power-bi/initial-data-view.png)

## <a name="prepare-the-data"></a>準備資料
<a name="PreparingData"></a>

您可能需要先將 Power BI Desktop 中的資料進行轉換，才能讓資料準備好供文字分析服務的關鍵片語 API 進行處理。

範例資料包含 `subject` 資料行和 `comment` 資料行。 有了 Power BI Desktop 中的「合併資料行」功能，您就可以從這兩個資料行中的資料擷取關鍵片語，而不是只從 `comment` 資料行。

在 Power BI Desktop 中，選取 [常用] 功能區。 在 [外部資料] 群組中，按一下 [編輯查詢]。

![[常用功能區中的外部資料群組]](../media/tutorials/power-bi/edit-queries.png)

在視窗左側的 [查詢] 清單中選取 `FabrikamComments` (如果尚未選取)。

現在，選取資料表中的 `subject` 和 `comment` 資料行。 您可能需要水平捲動才能看到這些資料行。 請先按一下 `subject` 資料行標題，然後按住 Control 鍵並按一下 `comment` 資料行標題。

![[選取要合併的欄位]](../media/tutorials/power-bi/select-columns.png)

選取 [轉換] 功能區。 在功能區的 [文字資料行] 群組中，按一下 [合併資料行]。 [合併資料行] 對話方塊隨即出現。

![[使用合併資料行對話方塊合併欄位]](../media/tutorials/power-bi/merge-columns.png)

在 [合併資料行] 對話方塊中，選擇 `Tab` 作為分隔符號，然後按一下 [確定]。

您也可以考慮使用「移除空白」篩選條件來篩掉空白訊息，或是使用「清理轉換」移除不可列印的字元。 如果資料包含像是檔案範例中 `spamscore` 資料行的資料行，您可以使用「數字篩選條件」來略過「垃圾」評論。

## <a name="understand-the-api"></a>了解 API
<a name="UnderstandingAPI"></a>

文字分析服務的[關鍵片語 API](//westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c6) 可針對每個 HTTP 要求處理多達一千個文字文件。 Power BI 偏好一次處理一個記錄，所以在此教學課程中，您對 API 的呼叫每次只會包含單一文件。 關鍵片語 API 要求所處理的每個文件都必須具有下列欄位。

| | |
| - | - |
| `id`  | 這個文件在要求中的唯一識別碼。 回應中也會包含此欄位。 如此一來，如果您處理多份文件，就可以輕鬆地將擷取到的關鍵片語關聯至其來源文件。 在此教學課程中，因為您針對每個要求只會處理一個文件，您可以針對每個要求將 `id` 的值以硬式編碼設為相同的值。|
| `text`  | 要處理的文字。 此欄位的值來自您在[前一節](#PreparingData)中建立的 `Merged` 資料行，其中包含了結合的主旨行與評論文字。 關鍵片語 API 要求此資料的長度不得超過約 5,000 個字元。|
| `language` | 撰寫文件所使用之自然語言的代碼。 範例資料中的所有訊息都是英文，因此您可以針對此欄位以硬式編碼方式編寫 `en` 值。|

## <a name="create-a-custom-function"></a>建立自訂函式
<a name="CreateCustomFunction"></a>

現在，您已經準備好建立自訂函式，以整合 Power BI 與文字分析。 函式會收到要處理為參數的文字。 它會將資料轉換為所需的 JSON 格式 (以及反向轉換)，並對關鍵片語 API 提出 HTTP 要求。 接著，函式會剖析來自 API 的回應並傳回字串，其中包含所擷取關鍵片語的逗點分隔值清單。

> [!NOTE]
> Power BI Desktop 自訂函式會以 [Power Query M 公式語言](https://msdn.microsoft.com/library/mt211003.aspx) (簡稱 "M") 來撰寫。 M 是以 [F#](https://docs.microsoft.com/dotnet/fsharp/) 為基礎的功能性程式設計語言。 不過，不是程式設計師也能完成此教學課程；下面有所需的程式碼。

在 Power BI Desktop 中，確定您仍在 [查詢編輯器] 視窗中。 如果不是，請選取 [常用] 功能區，按一下 [外部資料] 群組中的 [編輯查詢]。

現在，在 [常用] 功能區的 [新增查詢] 群組中，開啟 [新增來源] 下拉式功能表，並選取 [空白查詢]。 

[查詢] 清單中會出現新的查詢，一開始名為 `Query1`。 按兩下此項目，並將它命名為 `KeyPhrases`。

現在，按一下 [常用] 功能區 [查詢] 群組中的 [進階編輯器]，以開啟 [進階編輯器] 視窗。 刪除該視窗中已有的程式碼，然後貼上下列程式碼。 

> [!NOTE]
> 下列範例假設文字分析 API 端點是以 `https://westus.api.cognitive.microsoft.com` 開頭。 文字分析允許您在 13 個不同區域中建立訂用帳戶。 如果您已在不同區域註冊服務，請務必使用所選區域的端點。 您可以透過登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)，選取您的文字分析訂用帳戶，然後選取 [概觀] 頁面來找到此端點。

```fsharp
// Returns key phrases from the text in a comma-separated list
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = Text.Lower(Text.Combine(jsonresp[documents]{0}[keyPhrases], ", "))
in  keyphrases
```

使用您的文字分析存取金鑰取代 `YOUR_API_KEY_HERE`。 您也可以透過登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)、選取您的文字分析訂用帳戶，然後選取 [概觀] 頁面來找到此金鑰。 請務必保留金鑰前後的引號。 然後按一下 [完成]。

## <a name="use-the-custom-function"></a>使用自訂函式
<a name="UseCustomFunction"></a>

現在您可以使用自訂函式，從每個客戶的意見擷取關鍵片語，並將它們儲存在資料表中的新資料行。 

在 Power BI Desktop 中，於 [查詢視窗] 中，切換回 `FabrikamComments` 查詢。 選取 [新增資料行] 功能區。 在 [一般] 群組中，按一下 [叫用自訂函數]。

![[叫用自訂函數按鈕]](../media/tutorials/power-bi/invoke-custom-function-button.png)<br><br>

[叫用自訂函數] 對話方塊隨即出現。 在 [新資料行名稱] 中，輸入 `keyphrases`。 在 [函數查詢] 中，選取您建立的自訂函式 `KeyPhrases`。

對話方塊中會出現新的欄位 **text (optional)**。 此欄位是我們要使用哪一個資料行，針對關鍵片語 API 的 `text` 參數提供值。 (請記住，您已經以硬式編碼方式編寫 `language` 與 `id` 參數的值)。從下拉式功能表選取`Merged` (我們[稍早](#PreparingData)透過合併主旨與訊息欄位所建立的資料行)。

![[叫用自訂函式]](../media/tutorials/power-bi/invoke-custom-function.png)

最後，按一下 [確定]。

如果一切都準備就緒，Power BI 會針對資料表中的每個資料列呼叫您的自訂函式一次。 它會傳送查詢至關鍵片語 API，並新增新的資料行到資料表來儲存結果。 但在開始進行之前，您可能需要指定驗證和隱私權設定。

## <a name="authentication-and-privacy"></a>驗證和隱私權
<a name="Authentication"></a>

關閉 [叫用自訂函數] 對話方塊之後，可能會出現橫幅要求您指定如何連線至關鍵片語 API。

![[認證橫幅]](../media/tutorials/power-bi/credentials-banner.png)

按一下 [編輯認證]，確定已在對話方塊中選取 `Anonymous`，然後按一下 [連線]。 

> [!NOTE]
> 選取 `Anonymous` 是因為文字分析服務會透過您使用的存取金鑰驗證您的身分，因此 Power BI 不需要為 HTTP 要求本身提供認證。

![[將驗證設定為匿名]](../media/tutorials/power-bi/access-web-content.png)

如果選擇匿名存取之後仍看到 [編輯認證] 橫幅，您可能忘了將文字分析存取金鑰貼入 `KeyPhrases` [自訂函式](#CreateCustomFunction)的程式碼中。

接下來，可能會出現橫幅要求您提供關於資料來源隱私權的資訊。 

![[隱私權橫幅]](../media/tutorials/power-bi/privacy-banner.png)

按一下 [繼續]，然後針對對話方塊中的每個資料來源選擇 `Public`。 然後按一下 [儲存]。

![[設定資料來源隱私權]](../media/tutorials/power-bi/privacy-dialog.png)

## <a name="create-the-word-cloud"></a>建立文字雲
<a name="WordCloud"></a>

處理好出現的各個橫幅後，按一下 [常用] 功能區中的 [關閉並套用] 以關閉查詢編輯器。

Power BI Desktop 需要一點時間來提出必要的 HTTP 要求。 在資料表的每個資料列中，新的 `keyphrases` 資料行會包含關鍵片語 API 在文字中所偵測到的關鍵片語。 

現在，您將使用此資料行來產生文字雲。 若要開始，請按一下工作區左邊 Power BI Desktop 主視窗中的 [報告] 按鈕。

> [!NOTE]
> 為何要使用所擷取的關鍵片語來產生文字雲，而不是使用每個評論的完整文字？ 關鍵片語可為我們提供客戶評論中的「重要」文字，而不只是「最常見的」文字。 此外，在產生的文字雲中，文字大小也不會因為相對少數的評論中頻繁使用某個文字而受到影響。

如果您尚未安裝文字雲自訂視覺效果，請安裝它。 在工作區右邊的 [視覺效果] 窗格中，按一下三個點 (**...**)，然後選擇 [從存放區匯入]。 然後搜尋「雲」，並按一下文字雲視覺效果旁的 [新增] 按鈕。 Power BI 會安裝文字雲視覺效果，並讓您知道它已成功安裝。

![[新增自訂視覺效果]](../media/tutorials/power-bi/add-custom-visuals.png)<br><br>

首先，按一下 [視覺效果] 面板中的 [文字雲] 圖示。

![[視覺效果面板中的文字雲圖示]](../media/tutorials/power-bi/visualizations-panel.png)

新的報告隨即出現在工作區中。 將 `keyphrases` 欄位從 [欄位] 面板拖曳至 [視覺效果] 面板的 [類別] 欄位。 文字雲會出現在報告中。

現在，切換到 [視覺效果] 面板的 [格式] 頁面。 在 [停止文字] 類別中，開啟 [預設停止文字] 以從雲端排除常見的簡短文字，例如 "of"。 

![[啟用預設停止文字]](../media/tutorials/power-bi/default-stop-words.png)

在這個面板中往下一點，關閉 [旋轉文字] 和 [標題]。

![[啟用焦點模式]](../media/tutorials/power-bi/word-cloud-focus-mode.png)

按一下報告中的 [焦點模式] 工具，以便更加了解我們的文字雲。 此工具會展開文字雲以填滿整個工作區，如下所示。

![[文字雲]](../media/tutorials/power-bi/word-cloud.png)

## <a name="more-text-analytics-services"></a>更多的文字分析服務
<a name="MoreServices"></a>

文字分析服務 (Microsoft Azure 所提供的其中一個認知服務) 也會提供情感分析和語言偵測。 如果客戶的意見反應並非全是英文，語言偵測便很實用。

這兩個另外的 API 類似關鍵片語 API。 這表示您可以使用幾乎相同於此教學課程中所建立的自訂函式，將它們與 Power BI Desktop 整合。 只要建立空白查詢並在 [進階編輯器] 中貼入下列適當的程式碼即可，方法同前。 (別忘了存取金鑰)！然後，同樣地，使用函式在資料表中新增資料行。

下面的情感分析函式會傳回分數，以指出文字中所表達的情感有多正面。

```fsharp
// Returns the sentiment score of the text, from 0.0 (least favorable) to 1.0 (most favorable)
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/sentiment",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    sentiment   = jsonresp[documents]{0}[score]
in  sentiment
```

以下是語言偵測函式的兩個版本。 第一個版本會傳回 ISO 語言代碼 (例如 `en` 代表英文)，第二個版本則會傳回「易記」名稱 (例如 `English`)。 您可能會注意到，兩個版本之間只有內文的最後一行不同。

```fsharp
// Returns the two-letter language code (for example, 'en' for English) of the text
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    = jsonresp[documents]{0}[detectedLanguages]{0}[iso6391Name]
in  language
```
```fsharp
// Returns the name (for example, 'English') of the language in which the text is written
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    = jsonresp[documents]{0}[detectedLanguages]{0}[name]
in  language
```

最後，以下是已提出的關鍵片語函式變化，其會以清單物件的形式傳回片語，而不是以逗號分隔片語單一字串的形式傳回。 

> [!NOTE]
> 傳回單一字串簡化了我們的文字雲範例。 另一方面，清單則是更有彈性的格式，可處理 Power BI 中所傳回的片語。 您可以使用查詢編輯器 [轉換] 功能區中的 [結構化資料行] 群組，以處理 Power BI Desktop 中的清單物件。

```fsharp
// Returns key phrases from the text as a list object
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = jsonresp[documents]{0}[keyPhrases]
in  keyphrases
```

## <a name="next-steps"></a>後續步驟
<a name="NextSteps"></a>

深入了解文字分析服務、Power Query M 公式語言或 Power BI。

> [!div class="nextstepaction"]
> [文字分析 API 參考](//westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c6)

> [!div class="nextstepaction"]
> [Power Query M 參考](//msdn.microsoft.com/library/mt211003.aspx)

> [!div class="nextstepaction"]
> [Power BI 文件](//powerbi.microsoft.com/documentation/powerbi-landing-page/)
