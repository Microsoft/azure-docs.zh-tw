---
title: 如何使用 Azure Machine Learning 資料準備的深度指南 | Microsoft Docs
description: 本文件提供如何使用 Azure Machine Learning 資料準備來解決資料問題的概觀和詳細資料
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: ''
ms.devlang: ''
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 7536e67d0ae4973008c8acc91a99a7d0d286f9b8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988728"
---
# <a name="data-preparations-user-guide"></a>資料準備使用者指南 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


Azure Machine Learning 資料準備體驗提供許多豐富的功能。 本文說明體驗最深的部分。

### <a name="step-execution-history-and-caching"></a>執行步驟、歷程記錄和快取 
資料準備步驟歷程記錄會基於效能考量來保留一系列的快取。 如果您選取某個步驟，它會點擊快取，而不會重新執行。 如果您在步驟歷程記錄結尾處具有一個寫入區塊，而您在步驟中來回移動但並未進行任何變更，則不會在第一次之後觸發寫入。 如果您執行下列動作，就會發生新的寫入，並覆寫前一個：

- 對寫入區塊進行變更。
- 新增新的轉換區塊，並將其移至會產生快取失效之寫入區塊上方。
- 在會產生快取失效的寫入區塊上方變更區塊的屬性。
- 在樣本上選取 [重新整理] (因而使所有快取失效)。

### <a name="error-values"></a>錯誤值

對於輸入值的資料轉換可能失敗，因為無法適當地處理該值。 例如，若起因為類型強制型轉作業，如果無法將輸入字串值轉換為指定的目標類型，強制型轉即會失敗。 類型強制型轉作業可能會將字串類型的資料行轉換為數值或布林值的類型，或嘗試複製不存在的資料行。 (此失敗的發生是因為將*刪除資料行 X* 作業移到*複製資料行 X* 作業之前。)

在這些情況下，資料準備會產生錯誤值作為輸出。 錯誤值會指出上一個作業因指定值而失敗。 在內部，會將它們視為第一級值類型，但它們的存在並不會改變資料行的基礎類型，即使資料行完全是由錯誤值組成也一樣。

錯誤值很容易識別。 它們會以紅色反白顯示為「錯誤」。 通常若要判斷原因，請將滑鼠暫留在錯誤值上方以取得失敗的文字描述。

錯誤值傳播。 發生錯誤值之後，它會在大部分情況下透過大多數作業傳播為錯誤。 有三種方式可以取代或移除它們：

* Replace
    -  以滑鼠右鍵按一下資料行，然後選取 [取代錯誤值]。 接著，您可以針對資料行中找到的每個錯誤值選擇取代值。

* 移除
    - 資料準備包含互動式篩選條件來保留或移除錯誤值。
    - 以滑鼠右鍵按一下資料行，然後選取 [篩選資料行]。 若要保留或移除錯誤值，請建立條件式並具備「是錯誤」或「不是錯誤」的條件。

* 使用 Python 運算式，有條件地在錯誤值上運作。 如需詳細資訊，請參閱[關於 Python 擴充功能的小節](data-prep-python-extensibility-overview.md)。

### <a name="sampling"></a>取樣
資料來源檔案會接收來自一或多個來源 (從本機檔案系統或遠端位置) 的未經處理資料。 「樣本」區塊可讓您指定是否要產生樣本來處理資料子集。 操作資料的樣本，而不是大型資料集，往往可在後續步驟中執行作業時得到更好的效能。

針對每個資料來源檔案，可以產生及儲存多個樣本。 但是，只能將一個樣本設為使用中的樣本。 您可以在 [資料來源] 精靈中，或藉由編輯「樣本」區塊，來建立、編輯或刪除樣本。 參考資料來源的任何資料準備檔案原本就會使用資料來源檔案中指定的樣本。

有許多取樣策略可供使用，各有不同的可設定參數。

#### <a name="top"></a>前幾個
這項策略可套用至本機或遠端檔案。 它會將前 N 個資料列 (由計數指定) 列入資料來源。

#### <a name="random-n"></a>隨機 N 個 
這項策略只能套用至本機檔案。 它會將隨機的 N 個資料列 (由計數指定) 列入資料來源。 您可以提供特定的種子以確保產生相同的樣本 (假設計數也一樣)。

#### <a name="random-"></a>隨機 % 
這項策略可套用至本機或遠端檔案。 在這兩種情況下，必須提供機率和種子，類似於「隨機 N 個」策略。

針對遠端檔案的範例，則需提供其他參數：

- 產生器的範例 
  - 選取 Spark 叢集或遠端 Docker 計算目標，以用於產生樣本。 您必須事先為專案建立計算目標，才能讓它出現在此清單中。 請依照[如何在 Azure Machine Learning 中使用 GPU](how-to-use-gpu.md) 的＜建立新的計算目標＞一節中的步驟來建立計算目標。
- 儲存體的範例 
  - 提供中繼儲存位置來儲存遠端樣本。 此路徑必須是與輸入檔案位置不同的目錄。

#### <a name="full-file"></a>完整檔案 
這項策略只能套用至本機檔案，將完整檔案納入資料來源。 如果檔案太大，此選項可能會使得應用程式中未來的作業變慢。 您可能會發現它更適合使用不同的取樣策略。


### <a name="fork-merge-and-append"></a>分支、合併及附加

在資料集上套用篩選條件時，作業會將資料劃分為兩個結果集，其中一組代表篩選條件中成功的記錄，另一組則代表失敗的記錄。 在任一情況下，使用者可選擇要顯示哪一個結果集。 使用者可以捨棄另一個資料集，或將它放入新的資料流程。 第二種選項稱為分支。

若要進行分支： 
1. 若要分支，請選取資料行、按一下滑鼠右鍵，然後選取 [篩選資料行]。

2. 在 [我想要] 下方，選取 [保留資料列] 以顯示通過篩選條件的結果集。

3. 選取 [移除資料列] 來顯示失敗的集合。

4. 在 [條件] 之後，選取 [建立包含已篩選出資料列的資料流程]，將未顯示的結果集分支到新的資料流程。


這種作法通常用來區隔出一組需要額外準備作業的資料。 在您備妥分支的資料集之後，通常會將資料與原始資料流程中的結果集合併。 若要執行合併 (分支作業的相反)，請使用下列其中一個動作：

- 附加資料列。 以垂直方式合併兩或多個資料流程 (資料列取向)。 
- 附加資料行。 以水平方式合併兩或多個資料流程 (資料行取向)。


>[!NOTE]
>如果發生資料行衝突，附加資料行即會失敗。


合併作業之後，來源資料流程將能參考一或多個資料流程。 資料準備會在步驟清單下方，於應用程式右下角提供一則通知來通知您。


所參考資料流程中的任何作業都需要父資料流程，以重新整理從所參考資料流程中使用的樣本。 在這種情況下，確認對話方塊會取代右下角的資料流程參考通知。 該對話方塊會確認您需要重新整理資料流程，以便將變更同步處理至任何相依性資料流程。

### <a name="list-of-appendices"></a>附錄清單 
* [支援的資料來源](data-prep-appendix2-supported-data-sources.md)  
* [支援的轉換](data-prep-appendix3-supported-transforms.md)  
* [支援的偵測器](data-prep-appendix4-supported-inspectors.md)  
* [支援的目的地](data-prep-appendix5-supported-destinations.md)  
* [Python 中篩選運算式的範例](data-prep-appendix6-sample-filter-expressions-python.md)  
* [Python 中轉換工作流程運算式的範例](data-prep-appendix7-sample-transform-data-flow-python.md)  
* [Python 中資料來源的範例](data-prep-appendix8-sample-source-connections-python.md)  
* [Python 中目的地連線的範例](data-prep-appendix9-sample-destination-connections-python.md)  
* [Python 中資料行轉換的範例](data-prep-appendix10-sample-custom-column-transforms-python.md)  
