---
title: 使用適用於 Azure Data Lake Analytics 作業的作業瀏覽器和作業檢視
description: 本文說明如何使用適用於 Azure Data Lake Analytics 作業的作業瀏覽器和作業檢視。
services: data-lake-analytics
ms.service: data-lake-analytics
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonwhowell
ms.assetid: bdf27b4d-6f58-4093-ab83-4fa3a99b5650
ms.topic: conceptual
ms.date: 08/02/2017
ms.openlocfilehash: 474478c8049dd97558b49b1df4b00655268fc0b3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43044093"
---
# <a name="use-job-browser-and-job-view-for-azure-data-lake-analytics"></a>使用適用於 Azure Data Lake Analytics 的作業瀏覽器和作業檢視
Azure Data Lake Analytics 服務會將提交的作業封存在[查詢存放區](#query-store)中。 在本文中，您會了解如何在適用於 Visual Studio 的 Azure Data Lake 工具中使用作業瀏覽器和作業檢視，來尋找作業歷程記錄資訊。 

依預設，Azure Data Lake Analytics 服務會封存作業 30 天。 您可以藉由設定自訂的到期原則，從 Azure 入口網站設定有效期限。 期限到期後，您將無法存取作業資訊。 

## <a name="prerequisites"></a>必要條件
請參閱[適用於 Visual Studio 的 Azure Data Lake 工具](data-lake-analytics-data-lake-tools-get-started.md#prerequisites)。

## <a name="open-the-job-browser"></a>開啟作業瀏覽器
透過 Visual Studio 中的**伺服器總管 > Azure > Data Lake Analytics > 作業**來存取作業瀏覽器。  您可以使用作業瀏覽器來存取 Data Lake Analytics 帳戶的查詢存放區。 作業瀏覽器的左側會顯示查詢資料存放區 (顯示基本的作業資訊)，右側會顯示作業檢視 (顯示詳細的作業資訊)。

## <a name="job-view"></a>作業檢視
作業檢視會顯示作業的詳細資訊。 若要開啟作業，您可以在作業瀏覽器中按兩下作業，或按一下作業檢視從 Data Lake 功能表將其開啟。 您應該會看到一個填入作業 URL 的對話方塊。

![Data Lake 工具 Visual Studio 作業瀏覽器](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view.png)

作業檢視包含︰

* 工作摘要
  
    重新整理作業檢視，以查看執行中作業的最新資訊。
  
  * 作業狀態 (圖形)：
    
      作業狀態說明作業階段︰
    
      ![Azure Data Lake Analytics 作業階段狀態](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-phases.png)
    
    * 準備中︰將您的指令碼上傳到雲端，使用編譯服務以編譯和最佳化指令碼。
    * 已排入佇列︰當作業等待到足夠的資源，或是作業超過每個帳戶限制的並行作業數上限時，便會排入佇列。 優先順序設定會決定排入佇列的作業順序，數字越低表示優先順序愈高。
    * 執行中︰作業正在 Data Lake Analytics 帳戶中實際執行。
    * 完成中︰作業正在完成 (例如，正在完成檔案)。
      
      作業在每個階段都可能會失敗。 例如，在準備階段的編譯錯誤，在排入佇列階段的逾時錯誤，以及在執行階段的執行錯誤等等。
  * 基本資訊
    
      基本作業資訊會在作業摘要面板的下半部顯示。
    
      ![Azure Data Lake Analytics 作業階段狀態](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-info.png)
    
    * 作業的結果︰成功或失敗。 作業在每個階段都可能會失敗。
    * 總持續時間︰提交時間和結束時間之間的時鐘時間 (持續時間)。
    * 總計算時間︰每個頂點執行時間的總和，您可以將其視為只有在一個頂點中執行作業的時間。 如需頂點的詳細資訊，請參閱頂點總計。
    * 提交/開始/結束時間︰Data Lake Analytics 服務接收作業提交/開始執行作業/成功與否結束作業的時間。
    * 編譯/已排入佇列/執行中︰花費在準備中/已排入佇列/執行中階段的時鐘時間。
    * 帳戶︰用於執行作業的 Data Lake Analytics 帳戶。
    * 作者︰提交作業的使用者，可以是實際人員的帳戶或系統帳戶。
    * 優先順序：作業的優先順序。 編號愈低，優先順序愈高。 它只會影響作業在佇列中的順序。 設定較高的優先順序，不會優先於執行中的作業。
    * 平行處理原則︰並行 Azure Data Lake Analytics 單位 (ADLAU) (也稱為頂點) 的要求數目上限。 目前，一個頂點等於一個雙虛擬核心和 6 GB RAM 的 VM，但這在未來的 Data Lake Analytics 更新中可能會升級。
    * 剩餘的位元組︰作業完成之前需要處理的位元組。
    * 已讀取/寫入的位元組︰作業開始執行後已讀取/寫入的位元組。
    * 總頂點︰作業作會分成許多的作業作片段，每份作業片段就稱為頂點。 這個值描述作業是由多少工作片段所組成。 您可以將頂點視為基本處理單位，也就是 Azure Data Lake Analytics 單位 (ADLAU)，且頂點可以在平行處理原則中執行。 
    * 完成/執行中/失敗︰完成/執行中/失敗的頂點計數。 頂點會因為使用者程式碼和系統失敗而失敗，但系統會自動重試幾次失敗的頂點。 如果頂點在重試後仍然失敗，則整個作業將會失敗。
* 作業圖形
  
    U-SQL 指令碼代表將輸入資料轉換為輸出資料的邏輯。 指令碼會在準備中階段進行編譯並最佳化至實體的執行計畫。 作業圖形可顯示實體的執行計畫。  下圖說明此程序：
  
    ![Azure Data Lake Analytics 作業階段狀態](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-logical-to-physical-plan.png)
  
    作業分成許多的工作片段。 每份工作片段稱為頂點。 頂點分組為超級頂點 (也稱為階段)，並以作業圖形視覺化呈現。 作業圖形中的綠色階段牌子顯示階段。
  
    階段中每的一個頂點都在進行同樣的工作，只是對象為相同資料的不同片段。 例如，如果您有一個 TB 資料的檔案，而且有數百個頂點在讀取它，則每個頂點都會讀取其中一個區塊。 這些頂點會在相同階段進行分組，並針對輸入檔的不同片段執行相同的工作。
  
  * <a name="state-information"></a>階段資訊
    
      在特定階段中，牌子會顯示一些數字。
    
      ![Azure Data Lake Analytics 作業圖形階段](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-stage.png)
    
    * SV1 擷取︰階段的名稱，由數字和作業方法所命名。
    * 84 個頂點︰這個階段的頂點總計數。 此圖指示在這個階段有多少工作區分為片段。
    * 12.90 秒/頂點︰這個階段頂點的平均執行時間。 此圖中的計算方式是加總 (每個頂點執行時間) / (頂點總計數)。 這表示如果您可以指派平行處理原則中執行的所有頂點，整個階段會在 12.90 秒內完成。 這也表示如果在這個階段中的所有工作都是循序完成，成本會是頂點數量乘以平均回應時間。
    * 已寫入 850,895 個資料列︰在這個階段中寫入的資料列總計數。
    * R/W︰在這個階段中讀取/寫入的資料量 (位元組)。
    * 色彩︰階段會使用色彩以表示不同的頂點狀態。
      
      * 綠色表示頂點已成功。
      * 橙色表示頂點已重試。 重試的頂點失敗後，系統會自動重試一次並且如果成功，則整體階段便成功完成。 如果頂點重試但仍失敗，色彩會變成紅色，整個作業也跟著失敗。
      * 紅色代表失敗，這表示系統已重試幾次特定頂點，但仍失敗。 這個情況會造成整個作業失敗。
      * 藍色表示特定頂點正在執行。
      * 白色表示頂點正在等待。 頂點可能會等待當 ADLAU 變成可用時進行排程，或者可能會等待輸入，因為其輸入資料可能還未就緒。
      
      您可以將滑鼠游標停留在狀態上，找到階段的更多詳細資料︰
      
      ![Azure Data Lake Analytics 作業圖形階段詳細資料](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-stage-details.png)
  * 頂點︰描述頂點的詳細資料，例如頂點總數是多少，已完成多少頂點，頂點失敗或仍在執行/等待等。
  * 跨/內部組合讀取的資料︰檔案和資料會儲存在分散式檔案系統的多個組合中。 這裡的值描述在相同組合或跨組合中，已讀取的資料量。
  * 總計算時間︰在階段中每個頂點執行時間的總和，您可以將其視為階段中所有工作只有在一個頂點中執行的時間。
  * 寫入/讀取的資料和資料列︰指出多少資料或資料列已讀取/寫入，或需要讀取。
  * 頂點讀取失敗︰描述多少頂點在讀取資料時失敗。
  * 頂點重複捨棄︰如果頂點的執行速度太慢，系統可能會排程多個頂點來執行相同的工作片段。 其中一個頂點順利完成之後，將會捨棄其餘的頂點。 頂點重複捨棄會記錄階段中視為重複項目而捨棄的頂點數目。
  * 頂點撤銷︰頂點已順利完成，但因為某些原因而在稍後重新執行。 例如，如果下游頂點失去中繼輸入資料，它會要求上游頂點重新執行。
  * 頂點排程執行︰已排程頂點的總時間。
  * 讀取的最小/平均/最大頂點資料︰最小/平均/最大的每一個頂點讀取資料。
  * 持續時間︰階段花費的時鐘時間，您需要載入設定檔以查看此值。
  * 工作播放
    
      Data Lake Analytics 執行作業，並封存執行作業資訊的頂點，例如頂點啟動、停止和失敗的時間，以及重試的方式等。所有的資訊會自動記錄於查詢存放區中，並儲存在其作業設定檔。 您可以透過作業檢視中的「載入設定檔」來下載作業設定檔，並在下載作業設定檔之後檢視作業播放。
    
      作業播放是叢集中所進行作業的概括視覺效果。 作業播放可讓您觀看作業執行進度，並在很短的時間內 (通常少於 30 秒) 以視覺化方式偵測出效能異常和瓶頸。
  * 作業熱度圖顯示 
    
      您可以透過作業圖形中的 [顯示] 下拉式清單選取作業熱度圖。 
    
      ![Azure Data Lake Analytics 作業圖形熱度圖顯示](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-heat-map-display.png)
    
      它會顯示作業的 I/O、時間和輸送量，讓您可以發現作業花費最多時間在哪些地方，或您的作業是否為 I/O 界限作業等等。
    
      ![Azure Data Lake Analytics 作業圖形熱度圖範例](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-graph-heat-map-example.png)
    
    * 進度︰作業執行進度，請參閱[階段資訊](#stage-information)中的資訊。
    * 讀取/寫入的資料︰每個階段中，讀取/寫入總資料的熱度圖。
    * 計算時間︰總和 (每個頂點執行時間) 的熱度圖，您可以將其視為階段中所有工作只有在一個頂點中執行的時間。
    * 每個節點的平均執行時間︰總和 (每個頂點執行時間) 的熱度圖 / (頂點數量)。 這表示如果您可以指派平行處理原則中執行的所有頂點，整個階段會在這個時間範圍內完成。
    * 輸入/輸出輸送量︰每個階段輸入/輸出輸送量的熱度圖，您可以藉此確認您的作業是否為 I/O 界限作業。
* 中繼資料作業
  
    您可以在 U-SQL 指令碼中執行某些中繼資料作業，例如建立資料庫、捨棄資料表等。這些作業會在編譯之後顯示於中繼資料作業。 您可以在這裡找到判斷提示、建立實體並捨棄實體。
  
    ![Azure Data Lake Analytics 作業檢視中繼資料作業](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-metadata-operations.png)
* 狀態記錄
  
    狀態記錄為也會以視覺化方式呈現在作業摘要中，但您可以在此取得更多詳細資料。 您可以找到詳細的資訊，例如作業何時已進行準備、排入佇列、開始執行以及結束。 此外您也可以找到作業已編譯的次數 (CcsAttempts: 1)，作業實際分派到叢集中的時間 (詳細資料︰將作業分派到叢集) 等等。
  
    ![Azure Data Lake Analytics 作業檢視狀態記錄](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-state-history.png)
* 診斷
  
    此工具會自動診斷作業執行。 當作業中有一些錯誤或效能問題時您會收到警示。 請注意，您必須下載設定檔以取得完整的資訊。 
  
    ![Azure Data Lake Analytics 作業檢視診斷](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-view-diagnostics.png)
  
  * 警告︰警示會在此處顯示編譯器警告。 警示出現後，您可以按一下「x 問題」連結以查看更多詳細資料。
  * 頂點執行時間過長︰如果任一個頂點用盡時間 (例如 5 小時)，問題就會顯示在這裡。
  * 資源使用狀況︰如果您配置過多或不足所需的平行處理原則，問題就會顯示在這裡。 您也可以按一下 [資源使用狀況] 以查看更多詳細資料並執行假設案例，以尋找更好的資源配置 (如需詳細資料，請參閱本指南)。
  * 記憶體檢查︰如果任一個頂點使用超過 5 GB 的記憶體，問題就會顯示在這裡。 如果作業執行使用的記憶體超過系統限制，則系統可能會終止作業執行。

## <a name="job-detail"></a>作業詳細資料
作業詳細資料顯示作業的詳細資訊，包括指令碼、資源和頂點執行檢視。

![Azure Data Lake Analytics 作業詳細資料](./media/data-lake-analytics-data-lake-tools-view-jobs/data-lake-tools-job-details.png)

* 指令碼
  
    作業的 U-SQL 指令碼會儲存於查詢存放區中。 您可以檢視原始 U-SQL 指令碼，並視需要重新提交。
* 資源
  
    您可以透過資源找到儲存於查詢存放區中的作業編譯輸出。 比方說，您可以在這裡找到用來顯示作業圖形的 “algebra.xml”、您已註冊的組件等。
* 頂點執行檢視
  
    它會顯示頂點執行詳細資料。 作業設定檔會封存每個頂點執行記錄檔，例如讀取/寫入的總資料、執行階段、狀態等。透過這個檢視，您可以取得作業如何執行的詳細資料。 如需詳細資訊，請參閱[在適用於 Visual Studio 的 Data Lake 工具中使用頂點執行檢視](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md)。

## <a name="next-steps"></a>後續步驟
* 若要記錄診斷資訊，請參閱 [為 Azure Data Lake Analytics 存取診斷記錄檔](data-lake-analytics-diagnostic-logs.md)
* 若要了解更複雜的查詢，請參閱 [使用 Azure Data Lake Analytics 來分析網站記錄檔](data-lake-analytics-analyze-weblogs.md)。
* 若要使用頂點執行檢視，請參閱[在 Data Lake Tools for Visual Studio 中使用頂點執行檢視](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md)

