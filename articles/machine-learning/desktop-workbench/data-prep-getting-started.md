---
title: 開始進行 Azure Machine Learning 的資料準備 | Microsoft Docs
description: 這是適用於 AML Workbench 資料準備區段的快速入門指南
services: machine-learning
author: cforbe
ms.author: cforbe
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/07/2017
ROBOTS: NOINDEX
ms.openlocfilehash: a3f50e8fbaa396d9ca5a0d14fe9f54c30bc24f41
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996276"
---
# <a name="getting-started-with-data-preparation"></a>開始進行資料準備

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



歡迎使用《資料準備快速入門指南》。 

資料準備提供一組工具，有效率地探索、了解及修正資料中的問題。 它可讓您取用許多不同形式的資料，並將該資料轉換為較適合下游使用的全新資料。

資料準備會安裝為 Azure Machine Learning Workbench 體驗的一部分。  在本機使用資料準備或部署至目標叢集和雲端，以作為執行階段或執行環境。

設計階段執行階段會基於擴充性而使用 Python，並取決於各種 Python 程式庫，例如 Pandas。 與 Azure ML Workbench 的其他元件一樣，您不需要安裝 Python，系統會為您安裝。 不過，如果您需要安裝額外的程式庫，這些程式庫需要安裝於 Azure ML Workbench Python 目錄中，而不是您平常的 Python 目錄。 您可以在[這裡](data-prep-python-extensibility-overview.md)找到更多關於如何安裝封裝的詳細資料。

您需要先有專案，才能使用資料準備，一旦建立該專案之後，您就能準備資料。 

選取螢幕左邊的 [資料] 圖示 ![資料來源圖示](media/data-prep-getting-started/data-source-icon.png) 以瀏覽至專案的 [資料] 區段。  再按一下 [+] 以**新增資料來源**。 [資料來源精靈] 應該會啟動，並在完成精靈之後將**資料來源** (.dsource) 檔案新增至專案。 根據預設，資料將透過方格來檢視。 在方格上方，也能選取 [計量] 檢視。 在 [計量] 檢視中，會顯示摘要統計資料。  檢閱摘要統計資料之後，按一下螢幕頂端的 [準備]。 [資料來源精靈的詳細資訊](data-source-wizard.md) 

## <a name="building-blocks-of-data-preparation"></a>適用於準備資料的建置組塊 ##
### <a name="the-package"></a>封裝 ###

封裝是您工作的主要容器。 封裝是要儲存到磁碟以及從磁碟載入的成品。 在用戶端內工作時，封裝經常會在背景中「自動儲存」。 

封裝的輸出/輸入可在 Python 中或透過 Jupyter Notebook 進行探索。

封裝可以跨多個執行階段執行，包括本機 Python、Spark (包含於 Docker 中) 及 HDInsight。

封裝包含一或多個資料流程，其為適用於資料的步驟和轉換。

封裝可能會使用另一個封裝作為資料來源 (稱為「參考資料流程」)。

### <a name="the-dataflow"></a>資料流程 ###
資料流程都具有來源及選擇性的轉換，這類轉換是透過一連串步驟和選擇性目的地來排列的。 按一下 [步驟]，即會重新執行所選取步驟 (含) 之前的所有來源和轉換。  方格中會顯示透過該步驟轉換的資料。 步驟可在資料流程內透過 [步驟清單] 新增、移動和刪除。

您可以開啟和關閉用戶端右邊的 [步驟清單]，以提供更多螢幕空間。

UI 中可同時存在多個資料流程，每個資料流程均會以 UI 中的索引標籤來呈現。

### <a name="the-source"></a>來源
來源是資料的來源處且為其原始格式。 資料準備封裝一律源自其來自另一個資料流程 (資料來源) 的資料。 它就是這個包含資訊的參考資料流程。 每個來源都擁有不同的使用者體驗，以使其能夠設定。 來源會產生資料的「矩形」/表格式檢視。 如果來源資料原本就「不齊右」，則會將結構正規化為「矩形」。 [附錄 2 提供目前支援的來源清單](data-prep-appendix2-supported-data-sources.md)。

### <a name="the-transform"></a>轉換 ###
轉換會取用指定格式的資料、在資料上執行某些作業 (例如變更資料類型)，然後產生新格式的資料。 每個轉換都有它自己的 UI 和行為。 在資料流程中，透過 [步驟] 將數個轉換鏈結在一起，是準備資料功能的核心。 [附錄 3 提供目前支援的轉換清單](data-prep-appendix3-supported-transforms.md)。

### <a name="the-inspector"></a>偵測器 ###

偵測器是資料的視覺效果，可用以提升對資料的了解。  了解資料和資料品質問題，可協助您決定應採取的動作 (轉換)。 某些偵測器支援產生轉換的動作。 例如，值計數偵測器可讓您選取值，然後套用篩選條件以包含該值或排除該值。 偵測器也可提供用於轉換的內容。 例如，選取一或多個資料行，會變更可套用的可能轉換。

資料行隨時可能擁有多個偵測器 (例如，資料行統計資料和長條圖)。 可能也會在多個資料行上具有偵測器的執行個體。 例如，所有數值資料行可能會同時具備長條圖。

偵測器會出現螢幕底部的 [Profiling Well] \(分析來源) 中。  將偵測器最大化，讓它們在主要內容區域內看起來更大。 將資料格視為預設的偵測器。 任何偵測器都能擴展到主要內容區域。 將主要內容區域中的偵測器最小化至 [Profiling Well] \(分析來源)。 按一下偵測器內的鉛筆圖示，即可自訂偵測器。 拖放即可重新排列 Well (來源) 內的偵測器。

某些偵測器支援 “Halo” 模式。 此模式會顯示在套用最後一個轉換之前的值或狀態。 舊值會以灰色顯示，且在前景中具有目前的值，並顯示轉換的影響。 [附錄 4 提供目前支援的偵測器清單](data-prep-appendix4-supported-inspectors.md)。

### <a name="the-destination"></a>目的地
 目的地是您已經在資料流程中備妥之後，才能在其中寫入資料或匯出至其中的地方。 指定的資料流程可以有多個目的地。 每個目的地都擁有不同的使用者體驗，以使其能夠設定。 目的地會取用「矩形」/表格格式的資料，並以指定的格式將它寫出至某個位置。 [附錄 5 提供目前支援的目的地清單](data-prep-appendix5-supported-destinations.md)。

### <a name="using-data-preparation"></a>使用資料準備 ###
資料準備會假設一個基本五步驟的方法/處理方法來進行資料準備。

#### <a name="step-1-ingestion"></a>步驟 1：擷取 ####
使用專案檢視內的 [新增資料來源] 選項，來匯入適用於資料準備的資料。  所有初始的資料擷取都會透過 [資料來源精靈] 來處理。

#### <a name="step-2-understandprofile-the-data"></a>步驟 2：了解/分析資料 ####

首先，查看每個資料行頂端的資料品質列。 綠色表示具有值的資料列。 灰色表示具有遺漏值、Null 等的資料列。紅色則表示錯誤值。 將滑鼠停留在該列上，以取得含有這三個貯體中每一個內含之確切資料列數的工具提示。 資料品質列會使用對數刻度，因此，請一律檢查實際數字，以取得遺漏資料量的粗略感覺。

![columns](media/data-prep-getting-started/columns.png)

接下來，使用其他偵測器的組合加上方格，更清楚了解資料特性。  開始制訂進一步分析所需之資料準備的相關假設。 大部分的偵測器都是在單一資料行或少量資料行上運作。  

很可能需要在數個資料行上有數個偵測器，才能了解資料。 您可以在 [Profiling Well] \(分析來源) 中捲動以瀏覽各種偵測器。 在 Well 內，您也可以將偵測器移到清單開頭，以便在可立即檢視的區域中看到它們。

![偵測器](media/data-prep-getting-started/inspectors.PNG)

已針對連續與類別目錄變數/資料行提供不同的偵測器。 [偵測器] 功能表會根據您擁有的變數/資料行類型來啟用和停用選項。

使用具有許多資料行的寬型資料集時，建議使用處理子集合的實用方法。 此方法包括將焦點放在少量資料行 (例如，5-10 行) 上、準備它們，然後處理其餘的資料行。 方格偵測器支援垂直分割資料行，因此，如果您擁有超過 300 個資料行，則需透過它們來「分頁」。
 

#### <a name="step-3-transform-the-data"></a>步驟 3：轉換資料 ####
轉換會變更資料並允許資料執行，以支援目前正在處理中的假設。 轉換會在右側的 [步驟清單] 中顯示為 [步驟]。 您也能夠按一下 [步驟清單] 中的任何任意點，透過 [步驟清單] 進行「時間移動」。

指定 [步驟] 左邊的綠色圖示表示它已經執行，而資料會反映轉換的執行。 [步驟] 左邊的垂直列表示偵測器中資料的目前狀態。

![steps](media/data-prep-getting-started/steps.PNG)

嘗試對資料進行小型經常性變更，並在每次因假設演變而變更之後進行驗證 (步驟 4)。

#### <a name="step-4-verify-the-impact-of-the-transformation"></a>步驟 4：驗證轉換的影響。 
決定假設是否正確。 如果正確，則開發下一個假設，並針對新的假設重複執行步驟 2-3。 如果不正確，則復原最後一個轉換，接著開發新的假設並重複執行步驟 2-3。

判斷轉換是否有正確影響的主要方法是使用偵測器。 使用現有的偵測器。 使用含 Halo 效果的偵測器會啟用或啟動多個偵測器，來檢視特定時間點的資料。

![Halo 偵測器](media/data-prep-getting-started/halo1.PNG) ![Halo 偵測器](media/data-prep-getting-started/halo2.PNG)

若要復原轉換，請繼續執行 UI 右側的 [步驟清單]。 ([步驟清單] 面板可能需要重新快顯。若要開啟它，按一下指向左邊的雙＞形箭號)。 在面板中，選取您想要復原的已執行轉換。 選取 UI 區塊右側的下拉式清單。 選取 [編輯] 進行變更，或 [刪除] 以從 [步驟清單] 和 [資料流程] 中移除轉換。

#### <a name="step-5-output"></a>步驟 5：輸出 
完成資料準備時，您可以將資料流程寫入輸出。 資料流程可以有多個輸出。 您可以從 [轉換] 功能表中，選取您想要以哪一種輸出形式寫入資料集。 您也可以選取輸出目的地。 

## <a name="list-of-appendices"></a>附錄清單 
[附錄 2 - 支援的資料來源](data-prep-appendix2-supported-data-sources.md)  
[附錄 3 - 支援的轉換](data-prep-appendix3-supported-transforms.md)  
[附錄 4 - 支援的偵測器](data-prep-appendix4-supported-inspectors.md)  
[附錄 5 - 支援的目的地](data-prep-appendix5-supported-destinations.md)  
[附錄 6 - Python 中的樣本篩選運算式](data-prep-appendix6-sample-filter-expressions-python.md)  
[附錄 7 - Python 中的樣本轉換工作流程運算式](data-prep-appendix7-sample-transform-data-flow-python.md)  
[附錄 8 - Python 中的樣本資料來源](data-prep-appendix8-sample-source-connections-python.md)  
[附錄 9 - Python 中的樣本目的地連線](data-prep-appendix9-sample-destination-connections-python.md)  
[附錄 10 - Python 中的樣本資料行轉換](data-prep-appendix10-sample-custom-column-transforms-python.md)  

## <a name="see-also"></a>另請參閱

[進階資料準備教學課程](tutorial-bikeshare-dataprep.md)