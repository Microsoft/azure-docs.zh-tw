---
title: 在 Azure Machine Learning 中使用資料轉換進行資料準備 | Microsoft Docs
description: 本文提供 Azure Machine Learning 資料準備可使用的完整轉換清單。
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
ms.openlocfilehash: 26674648217e01b66fbe722cd013d51d867e9ba9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984368"
---
# <a name="use-data-transforms-for-data-preparation-in-azure-machine-learning"></a>在 Azure Machine Learning 中使用資料轉換進行資料準備

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

Azure Machine Learning 中的轉換會取用指定格式的資料、在資料上執行作業 (例如變更資料類型)，然後產生新格式的資料。 每個轉換都有它自己的介面和行為。 透過資料流程中的步驟將數個轉換鏈結在一起，您可以對資料執行複雜且可重複的轉換。 這是資料準備功能的核心。

以下是 Azure Machine Learning 中可用的轉換。 

## <a name="column-selection"></a>資料行選擇 
這份清單中的許多轉換可對單一資料行或多個資料行執行。 若要選取多個資料行，請使用 Ctrl 鍵。 若要選取某個範圍的資料行，請使用 Shift 鍵。

## <a name="transforms-from-the-main-menu-or-the-grid-header"></a>從主功能表或格線標題轉換 
請從主功能表上的 [轉換] 選項存取轉換。 您也可以用滑鼠右鍵按一下資料格線中的資料行名稱來選取轉換。 如果選取多個資料行，請用滑鼠右鍵按一下任一資料行可提供轉換功能表。

以右鍵按一下功能表只會顯示已選取資料類型的有效轉換。 主功能表會提供所有轉換，但會停用與所選資料行無關的轉換。

透過以滑鼠右鍵按一下資料格，可以使用內容相關的轉換子集。 這些轉換包括包括複製、取代和篩選。 這些轉換是資料類型感知，因此數字資料行的選項與字串資料行的選項不同。

## <a name="derive-column-by-example"></a>衍生資料行實例化
使用這項轉換可建立新的資料行，作為一或多個現有資料行的衍生。 轉換會查看輸入 (選取的) 資料行與指定範例，然後決定新資料行中所需的輸出。 

若要使用這項轉換，請選取一或多個資料行。 新增 (空白的) 衍生資料行實例化。 輸入您想要在衍生的資料行 (假設該資料行衍生自其他資料行) 中看見的範例，而且「實例化」技術會嘗試在資料行中填入所有其他資料格。 

對於複雜的範例，您可能需要提供一個以上的範例。 若要這樣做，請選取另一個資料格，並輸入另一個範例。

「實例化」技術使用選取的資料行嘗試判斷範例的意義。 如果叫用此轉換時，未選取任何資料行，則會使用目前資料列的所有資料格。 僅選取需要的資料行會產生更精確的結果。

您可以先選取資料行，然後再叫用轉換。 開啟轉換編輯器後，每個資料行頂端的核取方塊會指示選取哪些資料行作為輸入。 可以使用資料行標頭中的核取方塊來新增和移除選擇的資料行。

如需**衍生資料行實例化**轉換的詳細說明以及更多範例，請參閱[衍生資料行實例化參考](data-prep-derive-column-by-example.md)。  

## <a name="split-column-by-example"></a>分割資料行實例化
這項轉換會採用現有資料行並使用「實例化」引擎嘗試將資料行分割成 n 個其他資料行。 您可以在後續產生的資料行上執行自動分割。

如需**分割資料行實例化**轉換的詳細說明以及更多範例，請參閱[分割資料行實例化參考](data-prep-split-column-by-example.md)。

## <a name="expand-json"></a>展開 JSON

這項轉換可讓您展開包含有效 JSON 文字的資料行，從而新增多個資料行。

如需**展開 JSON** 轉換的詳細說明以及更多範例，請參閱[展開 JSON 參考](data-prep-expand-json.md)。


## <a name="combine-columns-by-example"></a>合併資料行實例化

這項轉換會合併多個資料行的值，從而新增資料行。 

如需**合併資料行實例化**轉換的詳細說明以及更多範例，請參閱[合併資料行實例化參考](data-prep-combine-columns-by-example.md)。


## <a name="duplicate-column"></a>複製資料行
這項轉換可完全複製一或多個選取的資料行，並提供每個衍生自原始資料行名稱的新名稱。

## <a name="text-clustering"></a>文字叢集 
此轉換旨在採用應該相同卻不一致的值，並將它們分組在一起。  

使用這項轉換，會針對單一資料行中的值分析相似性，並群組至叢集。 對於每個叢集而言，都有一個標準值，該值會取代叢集中所有執行個體，和所有執行個體值。 可以移除完整的叢集，且可以編輯標準值。 可從指定的叢集移除執行個體。 可以變更用於將執行個體分組至叢集的相似性分數閾值篩選條件。

根據預設，這項轉換會將所有叢集執行個體值取代為該叢集的標準值，建立新的資料行以包含新值。 您也可以將每個執行個體的相似性分數新增至新資料行 (可以命名該資料列)，以便稍後在資料流程中使用。

## <a name="replace-values"></a>取代值
使用這項轉換可將一個字串取代為另一個字串。 來源字串可以是部分字串或完整的儲存格，取代可以套用至單一或多個資料行。 搜尋字串支援搜尋特殊字元以及一般字元。 

## <a name="replace-na-values"></a>取代 NA 值
使用這項轉換可將各種形式的 NA (N/A、NA、null、NaN 等) 或空白字串取代為單一值使其一致。 這項轉換支援一或多個資料行，而且只有當選取資料行時才會列出這項轉換。 未選取任何資料行時，就不會出現在主要的轉換功能表上。

## <a name="replace-missing-values"></a>取代遺漏值
這項轉換會將遺漏值取代為單一值，而且支援一或多個資料行。 選取資料行時，才會列出這項轉換。 未選取任何資料行時，就不會出現在主要的轉換功能表上。

## <a name="replace-error-values"></a>取代錯誤值
這項轉換會將錯誤取代為單一值，而且支援一或多個資料行。 選取資料行時，才會列出這項轉換。 未選取任何資料行時，就不會出現在主要的轉換功能表上。

## <a name="trim-string"></a>修剪字串
這項轉換會將開頭和結尾的「空白字元」(包含空格、定位字元等) 從一或多個資料行中移除。

## <a name="adjust-precision"></a>調整有效位數
這項轉換會設定數值資料行的小數位數。

## <a name="rename-column"></a>重新命名資料行
這項轉換會變更所選資料行的名稱。 您也可以按一下資料行的名稱，在資料行標題中內嵌叫用此轉換。

## <a name="remove-column"></a>移除資料行
這項轉換會移除選取的資料行，而且可在單一或多個資料行上產生效用。 

## <a name="keep-column"></a>保留資料行
這項轉換只會保留選取的資料行，而且可在單一或多個資料行上產生效用。

## <a name="handle-path-column"></a>處理路徑資料行
在匯入檔案期間，**新增資料來源**功能會將路徑資料行自動新增至資料集。 它包含形成資料集路徑的完整檔案名稱。 這項轉換會新增資料行，或將該額外資料行從資料集移除。

## <a name="convert-field-type-to-numeric"></a>將欄位類型轉換成數值
這項轉換會將資料行類型變更為數值。 您可以指定非整數資料的分隔符號。 根據預設，這項轉換不會提示分隔符號，因此請使用 [編輯] 功能表項目以叫用編輯器。 此轉換適用於單一或多個資料行。

## <a name="convert-field-type-to-date"></a>將欄位類型轉換成日期
這項轉換會將資料行類型變更為日期。 使用預設日期/時間格式，但使用 `strftime` 指示詞可加以覆寫。 您也可以在固定日期添加時間值。

根據預設，這項轉換不會提示格式，因此請使用產生之步驟的 [編輯] 功能表項目以叫用編輯器。 此轉換適用於單一或多個資料行。

只有 9-22-1677 和 4-11-2262 之間的日期可以轉換為日期類型。

## <a name="convert-field-type-to-boolean"></a>將欄位類型轉換成布林值
這項轉換會將資料行類型變更為 true/false。 支援將多個值對應至 true 或 false，您可以編輯這些對應。 它也支援常數，可讓您對應至 true/false 對應表中不存在的值。 此轉換適用於單一或多個資料行。

## <a name="convert-field-type-to-string"></a>將欄位類型轉換成字串
這項轉換會將資料行類型變更為字串，而且可在單一或多個資料行上產生效用。

## <a name="convert-unix-timestamp-to-datetime"></a>將 Unix 時間戳記轉換為日期時間
即使在資料中表示為字串，這項轉換也了解 Unix 時間戳記格式。 在資料準備時，它會將時間戳記正確地轉換為日期類型。

## <a name="filter"></a>Filter
這項轉換會以一或多個資料行的值作為基礎，支援資料列篩選。 篩選條件會根據每個資料行的資料類型，因此您可以依「包含」或「不包含」來篩選字串資料行。 數值資料行可依「大於」或「小於」條件來篩選。

從主功能表或以滑鼠右鍵按一下資料行標題來叫用**篩選**轉換時，可使用選項將失敗的資料列派生至另一個資料流程。 主要資料流程會繼續篩選**加入**資料列，並建立新的資料流程，其中包含篩選**移除**的所有資料列。

## <a name="use-first-row-as-headers"></a>使用第一個資料列作為標題
這項轉換會將資料集的第一個資料列升級為資料行名稱。 如果某些資料行的第一個資料列中沒有資料，就會自動產生名稱。 使用此轉換表示最早從第 2 個資料列開始匯入資料。 根據**跳過資料列**設定，可能會在資料集更下層的資料列進行匯入。 您也可以使用它來移除提升功能，並且只使用自動產生的名稱。

## <a name="join"></a>Join
此轉換用來聯結兩個資料流程。 您可以根據預期結果來選取聯結的輸出：例如聯結的成功資料列、聯結的「左側」失敗資料列，或聯結的「右側」失敗資料列。

**聯結**轉換會從單一資料流程開始，然後選取該資料流程作為聯結的一端。 接著，它會提示您選取聯結另一端的另一個資料流程。 選取兩個流程之後，轉換需要選取待聯結之每一端聯結的單一資料行。 如果聯結需要多個資料行，請先建立衍生資料行，然後再啟動轉換，以建立僅用於聯結的新 (串連) 資料行。 轉換會嘗試建立聯結索引鍵，如果可能，將自動產生衍生的資料行。

完成聯結後，會顯示聯結的 Sankey 圖表檢視。 每條相關線的寬度代表該部分的聯結資料流程中移動的資料列數目。 使用右側面板，您可以選取成功的資料列、左側失敗資料列或右側失敗資料列。 您也可以選擇一個分支。

## <a name="append-rows"></a>附加資料列
這項轉換會將資料從另一個資料流程附加至目前的資料流程。 它會依位置對應資料行，以在結尾新增新的資料列。

## <a name="append-columns"></a>附加資料行
這項轉換會將新的資料行從另一個資料流程附加至目前的資料流程。 它會在右邊加入新的資料行。 它不會嘗試對齊資料列中的資料。

## <a name="summarize"></a>總結
這項轉換會計算一或多個選取的資料行中唯一值組合的彙總。 

支援的彙總為：COUNT、SUM、MIN、MAX、MEAN、VARIANCE 和 STANDARD DEVIATION。 特定資料行的彙總清單會篩選為僅適用於資料類型的彙總值。 會停用不適用的彙總。 例如，無法計算字串資料行的 MEAN，但可以計算 MIN 和 MAX。

編輯器可用時，會從資料行標題向上拖曳至左上方的面板，這裡會顯示要彙總的資料行。 此面板為階層式，因此您可以進行巢狀彙總。 右上方的編輯器面板可用於選取要套用至資料行的彙總。 單一資料行可以彙總一或多次。 至少選擇一個彙總之後，右下方的格線會預覽其彙總表單中的資料。 

這項轉換相當於 `ANSI-SQL GROUP BY`。

## <a name="remove-duplicates"></a>移除重複項目
這項轉換會在一或多個選取的資料行中具有重複值時，移除整個資料列。 如果未選擇任何資料行，則只會移除所有資料行值都相同的資料列。

## <a name="sort"></a>排序
這項轉換會排序資料。 排序可依單一資料行或多個資料行完成，且每個資料行可遞增排序 (預設) 或遞減排序 (可從編輯器中變更)。

這項轉換相當於 `ANSI-SQL ORDER BY`。

## <a name="output-transforms"></a>輸出轉換
下列轉換會輸出資料。 單一流程中可以有多個寫入區塊，以寫出不同點的資料。

### <a name="write-to-csv"></a>寫入至 CSV
這項轉換會從資料流程的目前點寫出 CSV 表單中的資料。 它會控制位置 (本機或遠端) 以及檔案周圍的各種設定。

### <a name="write-to-parquet"></a>寫入至 Parquet
這項轉換會從資料流程中的目前點寫出 Parquet 表單中的資料。 它會控制位置 (本機或遠端) 以及檔案周圍的各種設定。

## <a name="script-based-transforms"></a>指令碼型轉換
下列轉換會使用指令碼 (Python) 以執行核心產品中缺少的功能。 

>[!NOTE]
>使用任何轉換之前，請閱讀[使用 Python 擴充性](data-prep-python-extensibility-overview.md)。

### <a name="add-column-script"></a>新增資料行 (指令碼)
這項轉換會使用 Python 運算式在資料中新增資料行。
如需詳細資訊，請參閱[衍生新資料行的範例 Python 程式碼](data-prep-appendix10-sample-custom-column-transforms-python.md)。

### <a name="advanced-filter-script"></a>進階篩選 (指令碼)
使用此轉換可撰寫 Python 資料列層級篩選。
如需詳細資訊，請參閱[範例篩選條件運算式](data-prep-appendix6-sample-filter-expressions-python.md)。

### <a name="transform-dataflow-script"></a>轉換資料流程 (指令碼)
這項轉換可將 Python 套用至整個資料集。
如需詳細資訊，請參閱[轉換資料流程轉換範例](data-prep-appendix7-sample-transform-data-flow-python.md)。

這項轉換也可將 Python 套用至整個資料分割區。
如需詳細資訊，請參閱[轉換資料流程轉換範例](data-prep-appendix7-sample-transform-data-flow-python.md)。

### <a name="write-dataflow-script"></a>寫入資料流程 (指令碼)
這項轉換使用 Python 以寫出整個資料集。
如需詳細資訊，請參閱[衍生新資料行的範例 Python](data-prep-appendix9-sample-destination-connections-python.md)。



