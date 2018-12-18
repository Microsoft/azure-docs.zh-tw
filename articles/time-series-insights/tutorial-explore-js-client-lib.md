---
title: 探索 Azure 時間序列深入解析 JavaScript 用戶端程式庫
description: 了解 Azure 時間序列深入解析 JavaScript 用戶端程式庫和相關的程式設計模型。
author: ashannon7
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: tutorial
ms.date: 06/05/2018
ms.author: anshan
ms.openlocfilehash: 85d6ac0084343dc821a4eaf91c8053c9885ccfeb
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46366041"
---
# <a name="tutorial-explore-the-azure-time-series-insights-javascript-client-library"></a>教學課程：探索 Azure 時間序列深入解析 JavaScript 用戶端程式庫

為了協助 Web 開發人員查詢時間序列深入解析 (TSI) 中儲存的資料並將其視覺化，我們開發出以 JavaScript D3 為基礎的 TSI 用戶端程式庫。  此教學課程會使用 Web 應用程式範例，來引導您探索 TSI 用戶端程式庫，以及相關的程式設計模型。

此教學課程中討論的各個主題會讓您有機會以程式庫進行試驗，以了解如何存取 TSI 資料，以及使用圖表控制項來轉譯並視覺化資料。 課程目標是要讓您獲得足夠的詳細資料，以供您在 Web 應用程式中使用程式庫。

在此教學課程中，您會了解：

> [!div class="checklist"]
> * TSI 範例應用程式。
> * TSI JavaScript 用戶端程式庫。
> * 應用程式範例如何使用程式庫來將 TSI 資料視覺化。

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-introduce-the-open-source-time-series-insights-javascript-sdkbr"></a>在此影片中，我們會介紹開放原始碼「時間序列深入解析 JavaScript SDK」。</br>

> [!VIDEO https://www.youtube.com/embed/X8sSm7Pl9aA]

## <a name="prerequisites"></a>先決條件

此教學課程使用在大多數新式網頁瀏覽器 (包括 [Edge](/microsoft-edge/devtools-guide)、[Chrome](https://developers.google.com/web/tools/chrome-devtools/)、[FireFox](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools)、[Safari](https://developer.apple.com/safari/tools/) 等等) 中皆能找到的「開發人員工具」功能 (也稱為 DevTools 或 F12)。 如果您不熟悉此功能，建議您先在瀏覽器中瀏覽此功能，然後再繼續。

## <a name="time-series-insights-sample-application"></a>時間序列深入解析範例應用程式

此教學課程通篇使用時間序列深入解析範例應用程式來瀏覽應用程式背後的原始程式碼，包括 TSI JavaScript 用戶端程式庫的使用方式。 此範例為單頁 Web 應用程式 (SPA)，用來說明如何使用程式庫。 此範例將說明如何從範例 TSI 環境查詢資料，並將其視覺化。

1. 瀏覽至[時間序列深入解析應用程式範例](https://insights.timeseries.azure.com/clientsample)。 您會看到類似下圖且包含登入提示的頁面：

   ![TSI 用戶端範例登入提示](media/tutorial-explore-js-client-lib/tcs-sign-in.png)

2. 選取 [登入]，然後輸入或選取您的認證。 請使用企業或組織帳戶 (Azure Active Directory) 或個人帳戶 (Microsoft 帳戶或 MSA)。

   ![TSI 用戶端範例認證提示](media/tutorial-explore-js-client-lib/tcs-sign-in-enter-account.png)

3. 成功登入後，您會看到類似下圖的頁面。 此頁面會顯示數個已填入 TSI 資料的樣式或範例圖表。 您的使用者帳戶和**登出**選項會顯示在右上角：

   ![登入之後的 TSI 用戶端範例主頁面](media/tutorial-explore-js-client-lib/tcs-main-after-signin.png)

### <a name="page-source-and-structure"></a>網頁原始碼和結構

首先，我們將檢視瀏覽器所轉譯網頁背後的 HTML 和 JavaScript 原始程式碼。 我們不會帶您看過所有元素，但您會了解各主要區段，以便對網頁的運作方式有所概念：

1. 在您的瀏覽器中開啟 [開發人員工具]。 檢查組成目前網頁的 HTML 元素 (又稱為 HTML 或 DOM 樹狀結構)。

2. 展開 `<head>` 和 `<body>` 元素，並注意下列各節：

   - 在 `<head>` 元素底下，您會找到一些元素，這些元素會提取其他檔案以協助網頁運作：
     - 用來參照 Azure Active Directory Authentication Library **adal.min.js** (也稱為 ADAL) 的 `<script>` 元素。 ADAL 是一個 JavaScript 程式庫，會提供 OAuth 2.0 驗證 (登入)，並且可讓您取得用來存取 API 的權杖。
     - 用於樣式表 (也稱為 CSS，例如 **sampleStyles.css** 和 **tsiclient.css**) 的多個 `<link>` 元素。 樣式表可用來控制視覺效果網頁樣式詳細資料，例如色彩、字型、間距等等。
     - 用來參照 TSI 用戶端 JavaScript 程式庫 **tsiclient.js** 的 `<script>` 元素。 頁面會使用此程式庫來呼叫 TSI 服務 API 並轉譯頁面上的圖表控制項。

     >[!NOTE]
     > ADAL JavaScript 程式庫的原始程式碼可從 [azure-activedirectory-library-for-js 存放庫](https://github.com/AzureAD/azure-activedirectory-library-for-js)取得。
     > TSI 用戶端 JavaScript 程式庫的原始程式碼可從 [tsiclient 存放庫](https://github.com/Microsoft/tsiclient)取得。

   - 在 `<body>` 元素底下，您會找到 `<div>` 元素 (作為容器來定義網頁上項目的版面配置) 以及另一個 `<script>` 元素：
     - 第一個 `<div>` 元素會指定 [登入] 對話方塊 (`id="loginModal"`)。
     - 第二個 `<div>` 元素則作為下列元素的父項：
       - 標頭 `<div>` 元素，用於頁面頂端附近的狀態訊息和登入資訊 (`class="header"`)。
       - 用於頁面主體元素其餘部分的 `<div>` 元素，包括所有圖表 (`class="chartsWrapper"`)。
       - `<script>` 區段，其中包含所有用來控制頁面的 JavaScript。

   [![使用開發人員工具的 TSI 用戶端範例](media/tutorial-explore-js-client-lib/tcs-devtools-callouts-head-body.png)](media/tutorial-explore-js-client-lib/tcs-devtools-callouts-head-body.png#lightbox)

3. 展開 `<div class="chartsWrapper">` 元素，您會發現更多子 `<div>` 元素。 這些元素會用來定位每個圖表控制項範例。 請注意其中有好幾對 `<div>` 元素，每個圖表範例各會使用一對：

   - 第一個 (`class="rowOfCardsTitle"`) 元素包含用來彙總圖表所述項目的敘述性標題。 例如：「靜態折線圖與完整大小的圖例」。
   - 第二個 (`class="rowOfCards"`) 元素是父系，包含其他用來將資料列中實際圖表控制項定位的子 `<div>` 元素。

   ![主體 div 元素](media/tutorial-explore-js-client-lib/tcs-devtools-callouts-body-divs.png)

4. 現在，展開位於 `<div class="chartsWrapper">` 元素正下方的 `<script type="text/javascript">` 元素。 請留意葉面層級 JavaScript 區段的開頭處，這部分可用來處理所有頁面邏輯：驗證、呼叫 TSI 服務 API、轉譯圖表控制項等等：

   ![主體指令碼](media/tutorial-explore-js-client-lib/tcs-devtools-callouts-body-script.png)

## <a name="tsi-javascript-client-library-concepts"></a>TSI JavaScript 用戶端程式庫概念

雖然我們不會詳細檢閱 TSI 用戶端程式庫 **tsclient.js**，但基本上此程式庫提供兩個重要類別的抽象概念：

- **用於呼叫 TSI 查詢 API 的包裝函式方法**：可讓您使用彙總運算式來查詢 TSI 資料的 REST API。 這些方法會組織在程式庫的 `TsiClient.Server` 命名空間下。
- **用於建立和填入數種圖表控制項類型的方法**：用來在網頁中轉譯 TSI 彙總資料的方法。 這些方法會組織在程式庫的 `TsiClient.UX` 命名空間下。

下列概念是通用的，且一般會適用於 TSI 用戶端程式庫 API。

### <a name="authentication"></a>驗證

如前所述，此範例是 SPA，會在 ADAL 中使用 OAuth 2.0 支援進行使用者驗證。 以下是這個指令碼區段中的一些重點：

1. 使用 ADAL 進行驗證時，用戶端應用程式必須在 Azure Active Directory (Azure AD) 應用程式登錄中註冊其本身。 作為 SPA，此應用程式會註冊為使用「隱含」的 OAuth 2.0 授權授與流程。 作為對應，應用程式會在執行階段指定一些註冊屬性 (例如，用戶端識別碼 GUID (`clientId`) 和重新導向 URI (`postLogoutRedirectUri`))，以參與該流程。

2. 之後，應用程式會從 Azure AD 要求「存取權杖」。 系統會對特定服務/API 識別碼發出一組有限權限的存取權杖 (https://api.timeseries.azure.com)。 此服務/API 識別碼也稱為權杖「對象」。 這些權杖權限是由系統代表所登入的使用者來發出。 服務/API 的識別碼仍是應用程式的 Azure AD 註冊中所包含的另一個屬性。 ADAL 將存取權杖傳回給應用程式後，系統在存取 TSI 服務 API 時就會以「持有人權杖」的形式來傳遞存取權杖。

   [!code-javascript[head-sample](~/samples-javascript/pages/tutorial/index.html?range=145-204&highlight=4-9,36-39)]

### <a name="control-identification"></a>控制項識別

如前所述，`<body>` 元素內的 `<div>` 元素會提供網頁上示範的所有圖表控制項的版面配置。 每個 `<div>` 元素各自都會針對圖表控制項的位置和視覺效果屬性指定相關屬性，包括 `id` 屬性。 `id` 屬性會提供唯一識別碼，可用於 JavaScript 程式碼以找出並繫結至每個用於轉譯和更新的控制項。

### <a name="aggregate-expressions"></a>彙總運算式

TSI 用戶端程式庫 API 會大量使用彙總運算式。 彙總運算式可讓您建構一或多個「搜尋字詞」。 這些 API 在設計上類似於[時間序列深入解析總管](https://insights.timeseries.azure.com/demo)使用搜尋範圍、WHERE 述詞、量值和 split-by 值的方式。 大多數的程式庫 API 採用彙總運算式陣列，以供服務用來建置 TSI 資料查詢。

### <a name="call-pattern"></a>呼叫模式

圖表控制項的填入和轉譯會遵循一般模式。 您會發現整個網頁 JavaScript 一直在使用這個模式，以將 TSI 範例應用程式控制項具現化和載入：

1. 宣告 `array` 以保存一或多個 TSI 彙總運算式：

   ```javascript
   var aes =  [];
   ```

2. 建置 1 到 n 個彙總運算式物件，並將其新增到彙總運算式陣列：

   ```javascript
   var ae = new tsiClient.ux.aggregateExpression(predicateObject, measureObject, measureTypes, searchSpan, splitByObject, color, alias, contextMenuActions);
   aes.push(ae);
   ```

   **aggregateExpression 參數**

   | 參數 | 說明 | 範例 |
   | --------- | ----------- | ------- |
   | `predicateObject` | 資料篩選運算式。 |`{predicateString: "Factory = 'Factory3'"}` |
   | `measureObject`   | 使用中量值的屬性名稱。 | `{property: 'Temperature', type: "Double"}` |
   | `measureTypes`    | 量值屬性所需的彙總。 | `['avg', 'min']` |
   | `searchSpan`      | 彙總運算式的持續時間和間隔大小。 | `{from: startDate, to: endDate, bucketSize: '2m'}` |
   | `splitByObject`   | 您想要作為分割依據的字串屬性 (選擇性 - 可為 null)。 | `{property: 'Station', type: 'String'}` |
   | `color`         | 所要轉譯物件的色彩。 | `'pink'` |
   | `alias`           | 彙總運算式的易記名稱。 | `'Factory3Temperature'` |
   | `contextMenuActions` | 要繫結至視覺效果中時間序列物件的動作陣列 (選擇性)。 | 如需詳細資訊，請參閱[進階功能一節中的快顯功能表。](#popup-context-menus) |

3. 使用 `TsiClient.Server` API 呼叫 TSI 查詢以要求彙總資料：

   ```javascript
   tsiClient.server.getAggregates(token, envFQDN, aeTsxArray);
   ```

   **getAggregates 參數**

   | 參數 | 說明 | 範例 |
   | --------- | ----------- | ------- |
   | `token`     | TSI API 的存取權杖。 |  `authContext.getTsiToken()`如需詳細資訊，請參閱[驗證](#authentication)一節。 |
   | `envFQDN`   | TSI 環境的完整網域名稱 (FQDN)。 | 從 Azure 入口網站，例如：`10000000-0000-0000-0000-100000000108.env.timeseries.azure.com`。 |
   | `aeTsxArray` | TSI 查詢運算式的陣列。 | 依照先前的說明使用 `aes` 變數：`aes.map(function(ae){return ae.toTsx()}`。 |

4. 將 TSI 查詢所傳回的壓縮結果轉換為視覺效果的 JSON。

   ```javascript
   var transformedResult = tsiClient.ux.transformAggregatesForVisualization(result, aes);
   ```

5. 使用 `TsiClient.UX` API 建立圖表控制項，並將它繫結至頁面上的其中一個 `<div>` 元素。

   ```javascript
   var lineChart = new tsiClient.ux.BarChart(document.getElementById('chart3'));
   ```

6. 使用轉換後的 JSON 資料物件填入圖表控制項，並在頁面上轉譯該控制項。

   ```javascript
   lineChart.render(transformedResult, {grid: true, legend: 'compact', theme: 'light'}, aes);
   ```

## <a name="rendering-controls"></a>轉譯控制項

TSI 用戶端程式庫目前公開了八個唯一分析控制項：折線圖、圓形圖、橫條圖、熱度圖、階層控制項、可存取的方格、離散事件時間軸和狀態轉換時間軸。

### <a name="line-bar-pie-chart-examples"></a>折線圖、橫條圖、圓形圖範例

請查看應用程式中所示範的一些標準圖表控制項背後的程式碼，以及用來建立這些控制項的程式設計模型/模式。 具體來說，請檢查 `// Example 3/4/5` 註解底下的 HTML 區段，其會轉譯識別碼值為 `chart3`、`chart4` 和 `chart5` 的控制項。

回想一下[網頁原始碼和結構一節](#page-source-and-structure)中的步驟 3，圖表控制項會在網頁上成列排列，每個控制項各有一個敘述性標題資料列。 在此範例中，所填入的三個圖表皆位於「來自相同資料的多個圖表類型」標題 `<div>` 元素底下，並且繫結至該標題下的三個 `<div>` 元素：

[!code-javascript[code-sample1-line-bar-pie](~/samples-javascript/pages/tutorial/index.html?range=59-73&highlight=1,5,9,13)]

接下來的 JavaScript 程式碼區段會使用稍早說明的模式：建置 TSI 彙總運算式，使用這些運算式來查詢 TSI 資料，然後轉譯三個圖表。 請留意從 `tsiClient.ux` 命名空間 `LineChart`、`BarChart`、`PieChart` 用來建立及轉譯個別圖表的三種類型。 也請注意這三個圖表全都能使用相同的彙總運算式資料 `transformedResult`：

[!code-javascript[code-sample2-line-bar-pie](~/samples-javascript/pages/tutorial/index.html?range=241-262&highlight=13-14,16-17,19-20)]

三個圖表在轉譯時會呈現如下：

[![來自相同資料的多個圖表類型](media/tutorial-explore-js-client-lib/tcs-multiple-chart-types-from-the-same-data.png)](media/tutorial-explore-js-client-lib/tcs-multiple-chart-types-from-the-same-data.png#lightbox)

## <a name="advanced-features"></a>進階功能

TSI 用戶端程式庫也會公開一些選擇性的進階功能以供您利用。

### <a name="states-and-events"></a>狀態和事件

所提供的其中一個進階功能範例是能夠將狀態轉換和離散事件新增至圖表。 此功能可用於醒目提示事件、警示和狀態切換，例如開啟/關閉。

請查看 `// Example 10` 註解底下的 HTML 區段背後的程式碼。 此程式碼會轉譯「具有多個序列類型的折線圖」標題底下的線條控制項，並將其繫結至識別碼值為 `chart10` 的 `<div>` 元素。

1. 先定義名為 `events4` 的結構，以保存要追蹤的狀態變更元素。此結構包含：

   - 名為 `Component States` 的字串索引鍵。
   - 代表狀態的值物件陣列。 每個物件各包含：
     - 包含 JavaScript ISO 時間戳記的字串索引鍵。
     - 包含狀態特性的陣列：色彩和描述。

2. 然後，為「事件」定義 `events5` 結構，其中包含要追蹤之事件元素的陣列。陣列結構的圖形與針對 `events4` 所概述的結構相同。

3. 最後轉譯折線圖，傳入兩個具有圖表選項參數的結構：`events:` 和 `states:`。 請留意其他用來指定 `tooltip:`、`theme:` 或 `grid:` 的選項參數。

[!code-javascript[code-sample-states-events](~/samples-javascript/pages/tutorial/index.html?range=337-389&highlight=5,26,51)]

在視覺呈現上，用來表示事件的菱形標記/快顯視窗，與時間刻度上的彩色橫條/快顯視窗，皆表示狀態的變更：

[![具有多個序列類型的折線圖](media/tutorial-explore-js-client-lib/tcs-line-charts-with-multiple-series-types.png)](media/tutorial-explore-js-client-lib/tcs-line-charts-with-multiple-series-types.png#lightbox)

### <a name="pop-up-context-menus"></a>快顯功能表

進階功能的另一個範例是自訂快顯功能表 (以滑鼠右鍵按一下快顯功能表)。 自訂快顯功能表可用來啟用應用程式範圍內的動作和邏輯上的後續步驟。

請查看 `// Example 13/14/15` 註解底下的 HTML 區段背後的程式碼。 此程式碼一開始會轉譯「有快顯功能表可供建立圓形圖/橫條圖的折線圖」標題底下的折線圖，而此圖會繫結至識別碼值為 `chart13` 的 `<div>` 元素。 使用快顯功能表，折線圖就可讓您動態建立圓形圖和橫條圖，且繫結至識別碼為 `chart14` 和 `chart15` 的 `<div>` 元素。 此外，圓形圖和橫條圖也都會使用快顯功能表來啟用自己的功能：分別能夠將圓形圖的資料複製到橫條圖，以及將橫條圖的資料列印至瀏覽器的主控台視窗。

1. 先定義一系列的自訂動作。 每個動作都有一個包含一或多個元素的陣列。 每個元素分別定義單一快顯功能表項目：

   - `barChartActions`：此動作會定義圓形圖的快顯功能表，其包含一個元素來定義單一項目：
     - `name`：用於功能表項目的文字：「將參數列印至主控台」。
     - `action`：與功能表項目相關聯的動作。 此動作一律是匿名函式，會根據用來建立圖表的彙總運算式採用三個引數。 在此情況下，這些引數會寫入至瀏覽器的主控台視窗：
       - `ae`：彙總運算式陣列。
       - `splitBy`：splitBy 值。
       - `timestamp`：時間戳記。

   - `pieChartActions`：此動作會定義橫條圖的快顯功能表，其包含一個元素來定義單一項目。 圖形和結構描述和先前的 `barChartActions` 元素相同，但請注意 `action` 函式極為不同：它會將橫條圖具現化並轉譯。 也請注意，`ae` 引數會用來指定在執行階段於功能表項目開啟時所傳遞的彙總運算式陣列。 此函式也會對 `barChartActions` 快顯功能表設定 `ae.contextMenu` 屬性。
   - `contextMenuActions`：此動作會定義折線圖的快顯功能表，其包含三個元素來定義三個功能表項目。 每個元素的形狀和結構描述皆與先前說明的元素相同。 和 `barChartActions` 元素一樣，第一個項目會將三個函式引數寫入至瀏覽器主控台視窗。 與 `pieChartActions` 元素類似，第二的兩個項目會分別將圓形圖和橫條圖具現化與轉譯。 第二的兩個項目也會分別對 `pieChartActions` 和 `barChartActions` 快顯功能表設定其 `ae.contextMenu` 屬性。

2. 然後，系統會將兩個彙總運算式推送到 `aes` 彙總運算式陣列，並且為每個項目指定 `contextMenuActions` 陣列。 這些運算式會與折線圖控制項搭配使用。

3. 最後，一開始只會轉譯折線圖，然後可在執行階段從中轉譯出圓形圖和橫條圖。

[!code-javascript[code-sample-context-menus](~/samples-javascript/pages/tutorial/index.html?range=461-540&highlight=7,16,29,61-64,78)]

螢幕擷取畫面顯示這些圖表，以及其各自的快顯功能表。 系統會使用折線圖快顯功能表選項，以動態方式建立圓形圖和橫條圖。

[![有快顯功能表可供建立圓形圖/橫條圖的折線圖](media/tutorial-explore-js-client-lib/tcs-line-chart-with-context-menu-to-create-pie-bar-chart.png)](media/tutorial-explore-js-client-lib/tcs-line-chart-with-context-menu-to-create-pie-bar-chart.png#lightbox)

### <a name="brushes"></a>筆刷

筆刷可用來界定時間範圍，以定義縮放和瀏覽等動作。

用來說明筆刷的程式碼會顯示在先前的「有快顯功能表可供建立圓形圖/橫條圖的折線圖」範例中，以說明[快顯功能表](#popup-context-menus-section)。

1. 筆刷動作與快顯功能表類似，兩者都會定義筆刷的一系列自訂動作。 每個動作都有一個包含一或多個元素的陣列。 每個元素分別定義單一快顯功能表項目：
   - `name`：用於功能表項目的文字：「將參數列印至主控台」。
   - `action`：與功能表項目相關聯的動作，一律為匿名函式，且會採用兩個引數。 在此情況下，這些引數會寫入至瀏覽器的主控台視窗：
      - `fromTime`：筆刷選取項目的「從」時間戳記。
      - `toTime`：筆刷選取項目的「到」時間戳記。

2. 筆刷動作會新增為另一個圖表選項屬性。 請留意會傳遞至 `linechart.Render` 呼叫的 `brushContextMenuActions: brushActions` 屬性。

[!code-javascript[code-sample-brushes](~/samples-javascript/pages/tutorial/index.html?range=526-540&highlight=1,13)]

![有快顯功能表可供使用筆刷建立圓形圖/橫條圖的折線圖](media/tutorial-explore-js-client-lib/tcs-line-chart-with-context-menu-to-create-pie-bar-chart-brushes.png)

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 登入並瀏覽 TSI 應用程式範例和其來源。
> * 使用 TSI JavaScript 用戶端程式庫中的 API。
> * 使用 JavaScript 來以 TSI 資料建立和填入圖表控制項。

如前所述，TSI 應用程式範例會使用示範資料集。 若要了解如何建立自己的 TSI 環境和資料集，請繼續參閱下列文章：

> [!div class="nextstepaction"]
> [教學課程：建立 Azure 時間序列深入解析環境](tutorial-create-populate-tsi-environment.md)


