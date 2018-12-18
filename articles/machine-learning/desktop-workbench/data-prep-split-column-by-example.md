---
title: 使用 Azure Machine Learning Workbench 進行分割資料行實例化轉換
description: 「分割資料行實例化」轉換的參考文件
services: machine-learning
author: ranvijaykumar
ms.author: ranku
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc, reference
ms.topic: article
ms.date: 09/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 3edf49484e5bc05a297b8d8969632fb902aa1714
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953730"
---
# <a name="split-column-by-example-transformation"></a>分割資料行實例化轉換

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


這項轉換可預測性地將資料列的內容分割為有意義的界限，而不需要使用者輸入。 分割演算法會在分析資料行的內容之後選取界限。 您可以透過下列定義這些界限：
* 固定的分隔符號、
* 特殊內容中出現的多個任意分隔符號；或是，
* 資料模式或特定實體類型

使用者也可以在進階模式中控制分割行為，他們可以在此模式中指定分隔符號，或提供所需分割的範例。

理論上，也可以使用一系列的*衍生資料行實例化*轉換，在 Workbench 中執行分割作業。 不過，如果有數個資料行，即使使用實例化方法來衍生每個個別的資料行，也可能非常耗時。 預測性分割可讓使用者輕鬆分割，而不需要為每個資料行提供範例。 

## <a name="how-to-perform-this-transformation"></a>如何執行此轉換

1. 請選取您要分割的資料行。 
2. 從 [轉換] 功能表選取 [分割資料行實例化]。 或者，以滑鼠右鍵按一下選取資料行的標頭，並從操作功能表選取 [分割資料行實例化]。 「轉換編輯器」隨即開啟，並在選取的資料行旁邊新增一個新資料行。 此時，Workbench 會分析輸入資料行，判斷分割界限，並合成程式以分割格線中顯示的資料行。 合成的程式會針對資料行中的所有資料列來執行。 分隔符號 (如果有的話) 會從最終結果排除。
3. 您可以按一下 [進階模式]，以更精細地控制分割轉換。 
4. 檢閱輸出，然後按一下 [確定] 以接受轉換。 

轉換旨在為所有資料列產生相同數量的結果資料行。 如果在確定的界限上無法分割任何資料列，依預設會針對所有資料行產生 *null*。 可以在**進階模式**中變更此行為。

### <a name="transform-editor-advanced-mode"></a>轉換編輯器：進階模式
**進階模式**針對分割資料行提供更豐富的體驗。 

選取 [保留分割符號資料行]，會在最終結果中包含分隔符號。 依預設會排除分隔符號。

指定**分隔符號**會覆寫分隔符號自動選取邏輯。 可將多個分隔符號 (每行一個) 指定為**分隔符號**。 所有字元皆可作為分隔符號以分割資料行。

有時候，在確定的界限上分割值會產生與其他大多數不同的資料行數。 在這些情況下，**填滿方向**用來決定填滿資料行的順序。

按一下 [顯示建議的範例]，顯示使用者應提供分割範例的代表資料列。 使用者可以按一下建議資料列右測的**向上**箭號，以將該資料列升級為範例。 

使用者可以藉由在**範例資料表**的標頭上按一下滑鼠右鍵，以**刪除資料行**或**插入新資料行**。

使用者可以從資料格複製值，並將其貼至另一個資料格，以提供分割的範例。

使用者可按一下「轉換編輯器」中的連結，以在**基本模式**和**進階模式**之間切換。

### <a name="transform-editor-send-feedback"></a>轉換編輯器：傳送意見反應

按一下 [傳送意見反應] 連結會開啟 [意見反應] 對話方塊，其中會顯示以使用者所提供參數選擇與範例預先填寫的意見方塊。 使用者應該檢閱意見方塊的內容並提供更多詳細資料以協助我們了解問題。 若使用者不想要與 Microsoft 分享資料，使用者在按下 [傳送意見反應] 按鈕之前應該先刪除預先填寫的範例資料。 


### <a name="editing-an-existing-transformation"></a>編輯現有的轉換

使用者可選取「轉換步驟」中的 [編輯] 選項，以編輯現有的**分割資料行實例化**轉換。 按一下 [編輯]，可在**進階模式**中開啟「轉換編輯器」，並顯示建立轉換期間提供的所有範例。

## <a name="examples-of-splitting-on-a-fixed-single-character-delimiter"></a>針對固定的單一字元分隔符號進行分割的範例

資料欄位通常由單一固定分隔符號 (例如 CSV 格式的逗號) 分隔。 分割轉換會嘗試自動推斷這些分隔符號。 例如，在下列案例中，會將 "." 自動推斷為分隔符號。

### <a name="splitting-ip-addresses"></a>分割 IP 位址

第一個資料行中的值可預測性地分成四個資料行。

|IP|IP_1|IP_2|IP_3|IP_4|
|:-----|:-----|:-----|:-----|:-----|
|192.168.0.102|192|168|0|102|
|192.138.0.101|192|138|0|101|
|192.168.0.102|192|168|0|102|
|192.158.1.202|192|158|1|202|
|192.168.0.102|192|168|0|102|
|192.169.1.102|192|169|1|102|

## <a name="examples-of-splitting-on-multiple-delimiters-within-particular-contexts"></a>在特定內容中分割多個分隔符號的範例

使用者的資料可能包含分隔不同欄位的許多不同的分隔符號。 此外，只有一些出現的分隔字串可能是分隔符號，並非全部。 例如，在下列情況所需的分隔符號集合是 "-"、"、" 以及 ":"，以產生所需的輸出。 不過，不是每個出現的 ":" 都應該是一個分割點，因為我們不想分割時間，而是將它保留在單一資料行中。 分割轉換會在輸入資料中出現的內容中推斷分隔符號，而不是任何可能出現的分隔符號。 轉換也知道常見的資料類型，例如日期和時間。   

### <a name="splitting-store-opening-timings"></a>分割開店時間

下列*時間*資料行中的值會預測性地分割成九個資料行，如下表所示。

|時間|
|:-----|
|星期一 - 星期五：上午 7:00 - 下午 6:00，星期六：上午 9:00 - 下午 5:00，星期日：休息|
|星期一 - 星期五：上午 9:00 - 下午 6:00，星期六：上午 4:00 - 下午 4:00，星期日：休息|
|星期一 - 星期五：上午 8:30 - 下午 7:00，星期六：上午 3:00 - 下午 2:30，星期日：休息|
|星期一 - 星期五：上午 8:00 - 下午 6:00，星期六：上午 2:00 - 下午 3:00，星期日：休息|
|星期一 - 星期五：上午 4:00 - 下午 7:00，星期六：上午 9:00 - 下午 4:00，星期日：休息|
|星期一 - 星期五：上午 8:30 - 下午 4:30，星期六：上午 9:00 - 下午 5:00，星期日：休息|
|星期一 - 星期五：上午 5:30 - 下午 6:30，星期六：上午 5:00 - 下午 4:00，星期日：休息|
|星期一 - 星期五：上午 8:30 - 下午 8:30，星期六：上午 6:00 - 下午 5:00，星期日：休息|
|星期一 - 星期五：上午 8:00 - 下午 9:00，星期六：上午 9:00 - 下午 8:00，星期日：休息|
|星期一 - 星期五：上午 10:00 - 下午 9:30，星期六：上午 9:30 - 下午 3:00，星期日：休息|

|Timings_1|Timings_2|Timings_3|Timings_4|Timings_5|Timings_6|Timings_7|Timings_8|Timings_9|
|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|
|星期一|星期五|上午 7:00|下午 6:00|星期六|上午 9:00|下午 5:00|星期日|關閉|
|星期一|星期五|上午 9:00|下午 6:00|星期六|上午 4:00|下午 4:00|星期日|關閉|
|星期一|星期五|上午 8:30|下午 7:00|星期六|上午 3:00|下午 2:30|星期日|關閉|
|星期一|星期五|上午 8:00|下午 6:00|星期六|上午 2:00|下午 3:00|星期日|關閉|
|星期一|星期五|上午 4:00|下午 7:00|星期六|上午 9:00|下午 4:00|星期日|關閉|
|星期一|星期五|上午 8:30|下午 4:30|星期六|上午 9:00|下午 5:00|星期日|關閉|
|星期一|星期五|上午 5:30|下午 6:30|星期六|上午 5:00|下午 4:00|星期日|關閉|
|星期一|星期五|上午 8:30|下午 8:30|星期六|上午 6:00|下午 5:00|星期日|關閉|
|星期一|星期五|上午 8:00|下午 9:00|星期六|上午 9:00|下午 8:00|星期日|關閉|
|星期一|星期五|上午 10:00|下午 9:30|星期六|上午 9:30|下午 3:00|星期日|關閉|

### <a name="splitting-iis-log"></a>分割 IIS 記錄

以下是多個任務分隔符號的另一個範例。 這個範例也包含內容分隔字元 "/"，該分隔字元不能在 URL 或檔案路徑中進行分割。 使用許多*衍生資料行實例化*轉換，並為每個欄位提供範例，以執行此分割是相當繁瑣的。 我們可以使用分割轉換以執行預測性分割，而不需要提供任何範例。

|logtext|
|:-----|
|192.128.138.20 - - [16/Oct/2016 16:22:33 -0200] "GET /images/picture.gif HTTP/1.1" 234 343 www.yahoo.com "http://www.example.com/" "Mozilla/4.0 (compatible; MSIE 4)" "-"|
|10.128.72.213 - - [17/Oct/2016 12:43:12 +0300] "GET /news/stuff.html HTTP/1.1" 200 6233 www.aol.com "http://www.sample.com/" "Mozilla/5.0 (MSIE)" "-"|
|192.165.71.165 - - [12/Nov/2016 14:22:44 -0500] "GET /sample.ico HTTP/1.1" 342 7342 www.facebook.com "-" "Mozilla/5.0 (Windows; U; Windows NT 5.1; rv:1.7.3)" "-"|
|10.166.64.165 - - [23/Nov/2016 01:52:45 -0800] "GET /style.css HTTP/1.1" 200 2552 www.google.com "http://www.test.com/index.html" "Mozilla/5.0 (Windows)" "-"|
|192.167.1.193 - - [16/Jan/2017 22:34:56 +0200] "GET /js/ads.js HTTP/1.1" 200 23462 www.microsoft.com "http://www.illustration.com/index.html" "Mozilla/5.0 (Windows)" "-"|
|192.147.76.193 - - [28/Jan/2017 26:36:16 +0800] "GET /search.php HTTP/1.1" 400 1777 www.bing.com "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)" "-"|
|192.166.64.165 - - [23/Mar/2017 01:55:25 -0800] "GET /style.css HTTP/1.1" 200 2552 www.google.com "http://www.test.com/index.html" "Mozilla/5.0 (Windows)" "-"|
|11.167.1.193 - - [16/Apr/2017 11:34:36 +0200] "GET /js/ads.js HTTP/1.1" 200 23462 www.microsoft.com "http://www.illustration.com/index.html" "Mozilla/5.0 (Windows)" "-"|

分割成：

|logtext_1|logtext_2|logtext_3|logtext_4|logtext_5|logtext_6|logtext_7|logtext_8|logtext_9|logtext_10|logtext_11|logtext_12|logtext_13|logtext_14|logtext_15|
|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|
|192.128.138.20|16/Oct/2016|16:22:33|-0200|GET|images/picture.gif|HTTP|1.1|234|343|www.yahoo.com|http://www.example.com/|Mozilla|4.0|相容；MSIE 4|
|10.128.72.213|17/Oct/2016|12:43:12|+0300|GET|news/stuff.html|HTTP|1.1|200|6233|www.aol.com|http://www.sample.com/|Mozilla|5.0|MSIE|
|192.165.71.165|12/Nov/2016|14:22:44|-0500|GET|sample.ico|HTTP|1.1|342|7342|www.facebook.com|-|Mozilla|5.0|Windows; U; Windows NT 5.1; rv:1.7.3|
|10.166.64.165|23/Nov/2016|01:52:45|-0800|GET|style.css|HTTP|1.1|200|2552|www.google.com|http://www.test.com/index.html|Mozilla|5.0|Windows|
|192.167.1.193|16/Jan/2017|22:34:56|+0200|GET|js/ads.js|HTTP|1.1|200|23462|www.microsoft.com|http://www.illustration.com/index.html|Mozilla|5.0|Windows|
|192.147.76.193|28/Jan/2017|26:36:16|+0800|GET|search.php|HTTP|1.1|400|1777|www.bing.com|-|Mozilla|4.0|相容; MSIE 6.0; Windows NT 5.1|
|192.166.64.165|23/Mar/2017|01:55:25|-0800|GET|style.css|HTTP|1.1|200|2552|www.google.com|http://www.test.com/index.html|Mozilla|5.0|Windows|
|11.167.1.193|16/Apr/2017|11:34:36|+0200|GET|js/ads.js|HTTP|1.1|200|23462|www.microsoft.com|http://www.illustration.com/index.html|Mozilla|5.0|Windows|

## <a name="examples-of-splitting-without-delimiters"></a>不需要分隔符號進行分割的範例
在某些情況下，沒有實際的分隔符號，而資料欄位可能會連續地接連發生。 在此情況下，分割轉換會自動偵測資料中的模式，以推測可能的分割點。 例如，在下列情況下，我們想要分隔金額與貨幣類型，而分割會自動推斷數值和非數值資料之間的界限作為分割點。

### <a name="splitting-amount-with-currency-symbol"></a>以貨幣符號分割金額

|Amount|Amount_1|Amount_2|
|:-----|:-----|:-----|
|\$14|$|14|
|£9|£|9|
|\$34|$|34|
|€ 18|€ |18|
|\$42|$|42|
|\$7|$|7|
|£42|£|42|
|\$16|$|16|
|€ 16|€ |16|
|\$15|$|15|
|\$16|$|16|
|€ 64|€ |64|

在下列範例中，我們想要分隔加權值與測量單位。 分割介面會再次自動偵測有意義的界限，並將其設為優先於其他可能的分隔符號，例如 "." 字元。 

### <a name="splitting-weights-with-units"></a>使用單位分割加權

|Weight|Weight_1|Weight_2|
|:-----|:-----|:-----|
|2.27KG|2.27|KG|
|1L|1|L|
|2.5KG|2.5|KG|
|2KG|2|KG|
|1.7KGA|1.7|KGA|
|3KG|3|KG|
|2KG|2|KG|
|125G|125|G|
|500G|500|G|
|1.5KGA|1.5|KGA|

## <a name="technical-notes"></a>技術說明

分割轉換功能是以**預測程式合成**技術為基礎。 在此技術中，會根據輸入資料自動學習資料轉換程式。 程式是利用網域特定語言合成。 DSL 是以特定規則運算式內容中出現的分隔符號和欄位為基礎。 有關此技術的詳細資訊可以在[本主題最新發佈](https://www.microsoft.com/research/publication/automated-data-extraction-using-predictive-program-synthesis/)中找到。 
