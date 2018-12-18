---
title: 能源需求預測技術指南 | Microsoft Docs
description: 具有能源需求預測之 Microsoft Cortana Intelligence 的解決方案範本技術指南。
services: machine-learning
documentationcenter: ''
author: yijichen
manager: cgronlun
editor: yijichen
ms.assetid: 7f1a866b-79b7-4b97-ae3e-bc6bebe8c756
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2016
ms.author: yijichen
ms.openlocfilehash: ebf037a139b3e443238bc2fc5d0ed73e697dddf5
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842544"
---
# <a name="technical-guide-to-the-cortana-intelligence-solution-template-for-demand-forecast-in-energy"></a>能源需求預測之 Cortana Intelligence 解決方案範本的技術指南
## <a name="overview"></a>**概觀**
解決方案範本的設計是要加速在 Cortana Intelligence 套件之上建置 E2E 示範的程序。 已部署的範本會以所需的 Cortana Intelligence 元件佈建您的訂用帳戶，並建立兩者間的關聯性。 它也會在資料管線上植入從資料模擬應用程式所產生的範例資料。 從提供的連結下載資料模擬器並將它安裝在本機電腦上；如需使用模擬器的指示，請參閱 readme.txt 檔案。 由模擬器產生的資料將會產生資料管線，並開始產生機器學習服務預測，然後您可以在 Power BI 儀表板上將其視覺化。

您可以在 [這裡](https://gallery.cortanaintelligence.com/SolutionTemplate/Demand-Forecasting-for-Energy-1)

部署程序將引導您完成數個步驟來設定解決方案的認證。 務必記錄這些認證，例如您在部署期間提供的解決方案名稱、使用者名稱和密碼。

這份文件的目標在於說明參考架構與隨著此方案範本佈建在您的訂用帳戶的不同元件。 文件也會示範如何使用您自己的實際資料來取代範例資料，以便看到您自己的資料的見解/預測。 此外，文件將說明如果想要以您自己的資料自訂解決方案，您需要修改的解決方案範本部份。 最後會提供如何建置此方案範本的 Power BI 儀表板的指示。

## <a name="details"></a>**詳細資料**
![](media/cortana-analytics-technical-guide-demand-forecast/ca-topologies-energy-forecasting.png)

### <a name="architecture-explained"></a>所說明的架構
部署解決方案時，會啟動 Cortana Analytics Suite 內的各種 Azure 服務 (也就是事件中樞、串流分析、HDInsight、Data Factory、Machine Learning 等)。 此架構圖說明概括而言如何從端對端建構能源需求預測解決方案範本。 您將可以調查這些服務，方法是在隨著解決方案部署而建立的解決方案範本圖表上按一下。 下列章節說明每個片段。

## <a name="data-source-and-ingestion"></a>**資料來源及擷取**
### <a name="synthetic-data-source"></a>綜合資料來源
針對此範本，使用的資料來源是從桌面應用程式產生，您將會下載應用程式並於部署成功後在本機執行。 當您在解決方案範本圖表上選取稱為「能源預測資料模擬器」的第一個節點時，可以在屬性列中找到下載及安裝此應用程式的指示。 此應用程式會將在解決方案流程的其餘部分使用的資料點或事件送入 [Azure 事件中樞](#azure-event-hub)服務。

只有當它在您的電腦上執行時，事件產生應用程式才會填入 Azure 事件中樞。

### <a name="azure-event-hub"></a>Azure 事件中樞
[Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/) 服務是綜合資料來源所提供的輸入收件者。

## <a name="data-preparation-and-analysis"></a>**資料準備和分析**
### <a name="azure-stream-analytics"></a>Azure 串流分析
[Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)服務用來為來自 [Azure 事件中樞](#azure-event-hub)服務的輸入串流提供近乎即時的分析，並將結果發佈到 [Power BI](https://powerbi.microsoft.com) 儀表板，以及將所有未經處理的內送事件保存至 [Azure 儲存體](https://azure.microsoft.com/services/storage/)服務，供 [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) 服務後續處理。

### <a name="hdinsight-custom-aggregation"></a>HDInsight 自訂彙總
Azure HDInsight 服務用來執行 [Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼 (由 Azure Data Factory 協調)，以提供使用 Azure 串流分析服務封存的原始事件的彙總。

### <a name="azure-machine-learning"></a>Azure Machine Learning
使用 [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 服務 (由 Azure Data Factory 協調) 預測特定區域 (提供所收到輸入資料) 的未來用電量。

## <a name="data-publishing"></a>**資料發佈**
### <a name="azure-sql-database-service"></a>Azure SQL Database 服務
[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 服務用來儲存 (由 Azure Data Factory 管理) Azure Machine Learning 服務收到的預測，供 [Power BI](https://powerbi.microsoft.com) 儀表板中取用。

## <a name="data-consumption"></a>**資料耗用量**
### <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com) 服務用來顯示儀表板，其中包含 [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)服務提供的彙總，以及利用 [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 服務產生並儲存在 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 中的需求預測結果。 如需如何建置此方案範本的 Power BI 儀表板的指示，請參閱下一節。

## <a name="how-to-bring-in-your-own-data"></a>**如何帶入您自己的資料**
本節說明如何將您自己的資料帶入 Azure，以及對於您放入這個架構的資料，需要變更哪些區域。

您放入的任何資料集不太可能會符合用於這個解決方案範本的資料集。 了解您的資料與需求對於如何修改此範本以搭配您自己的資料而言非常重要。 如果您是第一次使用 Azure Machine Learning 服務，可以使用 [如何建立您的第一個實驗](machine-learning/studio/create-experiment.md)中的範例來取得簡介。

下列各節將討論推出新資料集將需要修改的範本區段。

### <a name="azure-event-hub"></a>Azure 事件中樞
[Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)服務非常廣泛，因而資料可以 CSV 或 JSON 格式張貼至中樞。 Azure 事件中樞中未發生任何特殊處理，但務必了解提供給它的資料。

這份文件不會描述如何內嵌您的資料，但您可以使用[事件中樞 API](event-hubs/event-hubs-programming-guide.md)，輕鬆地將事件或資料傳送到 Azure 事件中樞。

### <a name="azure-stream-analytics"></a>Azure 串流分析
[Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/) 服務用來提供近乎即時分析，方法是從資料串流讀取並輸出資料至任意數目的來源。

針對能源需求預測的解決方案範本，Azure 串流分析查詢包含兩個子查詢，每個均從 Azure 事件中樞服務取用事件作為輸入，並且輸出至兩個不同的位置。 這些輸出包含一個 Power BI 資料集和一個 Azure 儲存體位置。

可以透過以下方式找到 [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/) 查詢：

* 登入 [Azure 入口網站](https://portal.azure.com/)
* 找出部署解決方案時所產生的串流分析作業 ![](media/cortana-analytics-technical-guide-demand-forecast/icon-stream-analytics.png)。 一個用於將資料推送至 Blob 儲存體 (例如 mytest1streaming432822asablob)，另一種用於將資料推送至 Power BI (例如 mytest1streaming432822asapbi)。
* 選取

  * [輸入] 以檢視查詢輸入
  * [查詢] 以檢視查詢本身
  * [輸出] 以檢視不同的輸出

Azure 串流分析查詢建構的相關資訊可在 MSDN 上的 [串流分析查詢參考](https://msdn.microsoft.com/library/azure/dn834998.aspx) 中找到。

在此解決方案中，會將資料集 (具有近乎即時的內送資料串流相關分析資訊) 輸出到 Power BI 儀表板的 Azure 串流分析作業會在這個解決方案範本中提供。 因為對於內送資料格式具有隱含知識，必須根據您的資料格式變更這些查詢。

其他 Azure 串流分析作業會將所有[事件中樞](https://azure.microsoft.com/services/event-hubs/)事件輸出到 [Azure 儲存體](https://azure.microsoft.com/services/storage/)，因為會將完整的事件資訊串流至儲存體，因此不論資料格式為何都不需進行修改。

### <a name="azure-data-factory"></a>Azure Data Factory
[Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) 服務會協調資料的移動和處理。 在能源需求預測解決方案範本中，Data Factory 包含 12 個[管線](data-factory/concepts-pipelines-activities.md)，這些管線會使用各種技術來移動和處理資料。

  您可以開啟隨著解決方案部署建立的解決方案範本圖表底端的 Data Factory 節點，以存取您的 Data Factory。 您在 Azure 入口網站中會看到 Data Factory。 如果您在您的資料集下看到錯誤，您可以忽略這些，錯誤是因為在啟動資料產生器之前即已部署 Data Factory。 這些錯誤不會防止您的 Data Factory 運作。

本節將討論 [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) 中包含的必要[管線](data-factory/concepts-pipelines-activities.md)和[活動](data-factory/concepts-pipelines-activities.md)。 下圖是解決方案的圖表檢視：

![](media/cortana-analytics-technical-guide-demand-forecast/ADF2.png)

此 Factory 的五個管線包含 [Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼，用來分割及彙總資料。 如上述，指令碼會位在安裝期間建立的 [Azure 儲存體](https://azure.microsoft.com/services/storage/)帳戶中。 其位置會是：demandforecasting\\\\script\\\\hive\\\\ (或 https://[解決方案名稱].blob.core.windows.net/demandforecasting)。

類似於 [Azure 串流分析](#azure-stream-analytics-1)查詢，[Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼對於內送資料格式具有隱含知識，必須根據您的資料格式和[特徵工程設計](machine-learning/team-data-science-process/create-features.md)需求來變更這些查詢。

#### <a name="aggregatedemanddatato1hrpipeline"></a>*AggregateDemandDataTo1HrPipeline*
這個[管線](data-factory/concepts-pipelines-activities.md)管線包含單一活動 - 使用 [HDInsightLinkedService](https://msdn.microsoft.com/library/azure/dn893526.aspx) 的 [HDInsightHive](data-factory/transform-data-using-hadoop-hive.md) 活動，它會執行 [Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼，透過 Azure 串流分析作業將分站層級中每 10 秒流入的需求資料，彙總到每小時區域層級，並放入 [Azure 儲存體](https://azure.microsoft.com/services/storage/)中。

此資料分割工作的 [Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼為 AggregateDemandRegion1Hr.hql

#### <a name="loadhistorydemanddatapipeline"></a>*LoadHistoryDemandDataPipeline*
這個[管線](data-factory/concepts-pipelines-activities.md)包含兩個活動︰

* 使用 [HDInsightLinkedService](https://msdn.microsoft.com/library/azure/dn893526.aspx) 的 [HDInsightHive](data-factory/transform-data-using-hadoop-hive.md) 活動，它會執行 Hive 指令碼，在 Azure 串流分析作業期間將分站層級中的每小時歷程需求資料，彙總到每小時區域層級，並放入 Azure 儲存體中
* [複製](https://msdn.microsoft.com/library/azure/dn835035.aspx) 活動，會將彙總資料從 Azure 儲存體 Blob 移至解決方案範本安裝期間所佈建的 Azure SQL Database。

此工作的 [Hive](http://blogs.msdn.com/b/bigdatasupport/archive/2013/11/11/get-started-with-hive-on-hdinsight.aspx) 指令碼為 AggregateDemandHistoryRegion.hql。

#### <a name="mlscoringregionxpipeline"></a>*MLScoringRegionXPipeline*
這些[管線](data-factory/concepts-pipelines-activities.md)包含數個活動，而且其最終結果為來自與這個方案範本相關聯的 Azure Machine Learning 實驗評分的預測。 這些管線幾乎完全相同，差別在於它們各自只會處理不同區域 (由傳入 ADF 管線的不同 RegionID 所完成) 以及每個區域的 hive 指令碼。  
包含在此管線的活動為：

* 使用 [HDInsightLinkedService](https://msdn.microsoft.com/library/azure/dn893526.aspx) 的 [HDInsightHive](data-factory/transform-data-using-hadoop-hive.md) 活動，它會執行 Hive 指令碼來執行 Azure Machine Learning 實驗所需的彙總及特徵設計。 此工作的 Hive 指令碼是個別的 PrepareMLInputRegionX.hql。
* [複製](https://msdn.microsoft.com/library/azure/dn835035.aspx)活動，它會將來自 [HDInsightHive](data-factory/transform-data-using-hadoop-hive.md) 活動的結果移動至可供 [AzureMLBatchScoring](https://msdn.microsoft.com/library/azure/dn894009.aspx) 活動存取的單一 Azure 儲存體 Blob。
* 呼叫 Azure Machine Learning 實驗的 [AzureMLBatchScoring](https://msdn.microsoft.com/library/azure/dn894009.aspx) 活動，會導致將結果放入單一 Azure 儲存體 Blob。

#### <a name="copyscoredresultregionxpipeline"></a>*CopyScoredResultRegionXPipeline*
這個[管線](data-factory/concepts-pipelines-activities.md)包含單一活動 - [複製](https://msdn.microsoft.com/library/azure/dn835035.aspx)活動，它會將 Azure Machine Learning 實驗的結果從個別 ***MLScoringRegionXPipeline*** 移動至隨著解決方案範本安裝佈建的 Azure SQL Database。

#### <a name="copyaggdemandpipeline"></a>*CopyAggDemandPipeline*
這個[管線](data-factory/concepts-pipelines-activities.md)包含單一活動 - [複製](https://msdn.microsoft.com/library/azure/dn835035.aspx)活動，它會將彙總的持續需求資料從 ***LoadHistoryDemandDataPipeline*** 移動至隨著解決方案範本安裝佈建的 Azure SQL Database。

#### <a name="copyregiondatapipeline-copysubstationdatapipeline-copytopologydatapipeline"></a>*CopyRegionDataPipeline、CopySubstationDataPipeline、CopyTopologyDataPipeline*
這個[管線](data-factory/concepts-pipelines-activities.md)包含單一活動 - [複製](https://msdn.microsoft.com/library/azure/dn835035.aspx)活動，它會將安裝解決方案範本時上傳至 Azure 儲存體 Blob 的區域/分站/Topologygeo 參考資料，移動到安裝解決方案範本時所佈建的 Azure SQL Database。

### <a name="azure-machine-learning"></a>Azure Machine Learning
用於此解決方案範本的 [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 實驗會提供區域需求預測。 實驗因取用的資料集而不同，因而需要特別針對帶入的資料進行修改或取代。

## <a name="monitor-progress"></a>**監視進度**
一旦資料產生器啟動，管線會開始合成，解決方案的不同元件會遵循 Data Factory 發出的命令開始動作。 您有兩種方式可以監視管線。

1. 檢查 Azure Blob 儲存體中的資料。

    其中一個串流分析作業會將原始內送資料寫入 blob 儲存體。 如果您從成功部署解決方案的畫面按一下解決方案的 [Azure Blob 儲存體]元件，然後在右窗格中按一下 [開啟]，系統會帶您前往 [Azure 入口網站](https://portal.azure.com)。 到達那裡之後，按一下 [Blob]。 在接下來的面板中，您會看到容器的清單。 按一下 [energysadata]。 在接下來的面板中，您會看到 [demandongoing] 資料夾。 在 rawdata 資料夾內，您會看到具有如 date=2016-01-28 等名稱的資料夾。如果您看到這些資料夾，則表示原始資料已經成功在您的電腦上產生並儲存在 blob 儲存體。 您應該會在這些資料夾中看到具有有限大小 (MB) 的檔案。
2. 檢查 Azure SQL Database 中的資料。

    管線的最後一個步驟是將資料 (例如從機器學習的預測) 寫入至 SQL Database。 您可能必須等候最多兩個小時，資料才會出現在 SQL Database。 監視您的 SQL Database 中有多少資料可用的其中一個方法是透過 [Azure 入口網站](https://portal.azure.com/)。 在左側面板找到並按一下 SQL DATABASES![](media/cortana-analytics-technical-guide-demand-forecast/SQLicon2.png)。 然後找到您的資料庫 (亦即 demo123456db) 並且按一下它。 在下一個頁面的 [連接至您的資料庫] 區段下方，按一下 [對您的 SQL Database 執行 Transact-SQL 查詢]。

    在這裡，您可以按一下 [新增查詢] 並查詢資料列數目 (例如，從 DemandRealHourly 選取 count(*)。隨著資料庫成長，資料表中的資料列數目應該會增加)。
3. 檢查 Power BI 儀表板中的資料。

    您可以設定 Power BI 最忙碌路徑儀表板以監視未經處理的內送資料。 請遵循＜Power BI 儀表板＞一節中的指示。

## <a name="power-bi-dashboard"></a>**Power BI 儀表板**
### <a name="overview"></a>概觀
本節說明如何設定 Power BI 儀表板，以視覺化方式檢視來自 Azure 串流分析 (熱路徑) 的即時資料，以及來自 Azure 機器學習 (冷路徑) 的預測結果。

### <a name="setup-hot-path-dashboard"></a>設定最忙碌路徑儀表板
下列步驟將引導您如何以視覺化方式，檢視解決方案部署時所產生串流分析作業的即時資料輸出。 需要 [Power BI 線上版](http://www.powerbi.com/) 帳戶，才能執行下列步驟。 如果您沒有帳戶，您可以 [建立一個](https://powerbi.microsoft.com/pricing)。

1. 在 Azure 串流分析 (ASA) 中加入 Power BI 輸出。

   * 您必須依照 [Azure 串流分析及 Power BI：適用於串流資料即時可見度的即時分析儀表板](stream-analytics/stream-analytics-power-bi-dashboard.md)中的指示，將 Azure 串流分析作業的輸出設定為 Power BI 儀表板。
   * 找出 [Azure 入口網站](https://portal.azure.com)中的串流分析作業。 作業名稱應該是︰您的解決方案名稱+"streamingjob"+隨機數字+"asapbi" (也就是 demostreamingjob123456asapbi)。
   * 針對 ASA 工作加入 PowerBI 輸出。 將 [輸出別名] 設定為 **‘PBIoutput’**。 將 [資料集名稱] 和 [資料表名稱] 命名為 **‘EnergyStreamData’**。 新增了輸出之後，按一下頁面底部的 [啟動]  以啟動串流分析作業。 您應該會收到確認訊息 (例如，「串流分析作業 myteststreamingjob12345asablob 啟動成功」)。
2. 登入 [Power BI 線上版](http://www.powerbi.com)

   * 在 [我的工作區] 中的左側面板 [資料集] 區段中，您應該可以看到新的資料集出現在 Power BI 的左面板上。 這是您在上一個步驟中從 Azure 串流分析推入的串流資料。
   * 請確定 [視覺效果]  窗格開啟，並顯示在螢幕的右邊。
3. 建立 [依時間戳記的需求] 磚︰

   * 在左側面板 [資料集] 區段中按一下資料集 ‘EnergyStreamData’  。
   * 按一下 [折線圖] 圖示 ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic8.png)。
   * 按一下 [欄位]  面板中的 ’EnergyStreamData’。
   * 按一下 [時間戳記]  ，並確定它顯示在 [座標軸] 底下。 按一下 [載入]  ，並確定它顯示在 [值] 底下。
   * 按一下上方的 [儲存]  並將報告命名為 “EnergyStreamDataReport”。 名為 “EnergyStreamDataReport” 的報告會顯示在左側 [導覽] 窗格的 [報告] 區段中。
   * 按一下此折線圖右上角的 [釘選視覺]![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic6.png) 圖示，[釘選至儀表板] 視窗就會出現以供您選擇儀表板。 選取 [EnergyStreamDataReport]，然後按一下 [釘選]。
   * 在儀表板上將滑鼠停留在此磚中，按一下右上角的 [編輯] 圖示可將其標題變更為「依時間戳記的需求」
4. 根據適當的資料集建立其他儀表板圖格。 最終的儀表板檢視：![](media/cortana-analytics-technical-guide-demand-forecast/PBIFullScreen.png)

### <a name="setup-cold-path-dashboard"></a>設定冷路徑儀表板
在冷路徑資料管線中，基本目標是取得每個區域的需求預測。 Power BI 會連接到 Azure SQL Database 做為其資料來源，即預測結果的儲存位置。

> [!NOTE]
> 1) 需要幾個小時的時間才能收集足夠用於儀表板的預測結果。 建議您在啟動資料產生器後的 2 到 3 個小時再開始此程序。 2) 在此步驟中，必要條件就是下載並安裝免費版軟體 [Power BI 桌面版](https://powerbi.microsoft.com/desktop)。
>
>

1. 取得資料庫認證。

   在移至後續步驟之前，您將需要**資料庫伺服器名稱、資料庫名稱、使用者名稱和密碼** 。 以下是引導您如何尋找的步驟。

   * 一旦您的解決方案範本圖表上的 [Azure SQL Database] 變成綠色，請按一下它，然後按一下 [開啟]。 您將會前往 Azure 入口網站，而且您的資料庫資訊頁面也會開啟。
   * 在該頁面上，您可以找到 [資料庫] 區段。 它會列出您已建立的資料庫。 您的資料庫名稱應該是 **「您的解決方案名稱 + 隨機數字 + 'db'」** (例如 "mytest12345db")。
   * 按一下您的資料庫，在新的快顯面板上方，就可以找到您的資料庫伺服器名稱。 您的資料庫伺服器名稱應該是 `"Your Solution Name + Random Number + 'database.windows.net,1433'"` (例如，"mytest12345.database.windows.net,1433")。
   * 您的資料庫**使用者名稱**和**密碼**使用解決方案部署期間記錄的相同使用者名稱和密碼。
2. 更新冷路徑 Power BI 檔案的資料來源

   * 確定您已安裝最新版本的 [Power BI Desktop](https://powerbi.microsoft.com/desktop)。
   * 在您下載的 **"DemandForecastingDataGeneratorv1.0"** 資料夾中，按兩下 **‘Power BI Template\DemandForecastPowerBI.pbix’** 檔案。 初始的視覺效果是根據虛擬資料所形成。 **注意：** 如果您看到錯誤訊息，請確定您已安裝最新版本的 Power BI Desktop。

     一旦開啟，請在檔案的頂端按一下 [編輯查詢] 。 在快顯視窗中，按兩下右面板上的 [來源]。
     ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic1.png)
   * 在快顯視窗中，將 [伺服器]和 [資料庫] 取代為您自己的伺服器和資料庫名稱，然後按一下 [確定]。 針對伺服器名稱，請確定您指定連接埠 1433 (**YourSoutionName.database.windows.net, 1433**)。 忽略畫面上出現的警告訊息。
   * 在下一個快顯視窗中，您會在左側窗格上看到兩個選項 ([Windows] 和 [資料庫])。 按一下 [資料庫]，填入您的 [使用者名稱] 和 [密碼] \(這是當您首次部署解決方案並建立 Azure SQL Database 時輸入的使用者名稱和密碼)。 在 [選取要套用這些設定的層級] 中，請勾選資料庫層級選項。 然後按一下 [連接] 。
   * 一旦引導您回到上一頁，請關閉視窗。 隨即快顯訊息 - 按一下 [套用]。 最後，按一下 [儲存]  按鈕以儲存變更。 您的 Power BI 檔案現在已建立與伺服器的連線。 如果視覺效果是空的，請確定將視覺效果上的選取範圍都清除，以將所有資料視覺化，成法是按一下圖例右上角的橡皮擦圖示。 使用重新整理按鈕在視覺效果上反映新的資料。 最初，您只會在視覺效果上看到種子資料，因為 Data Factory 排定為每 3 個小時重新整理。 3 小時後，當您重新整理資料時，會看到新的預測反映在視覺效果中。
3. (選擇性) 將冷路徑儀表板發佈至 [Power BI 線上版](http://www.powerbi.com/)。 請注意，這個步驟需要 Power BI 帳戶 (或 Office 365 帳戶)。

   * 按一下 [發佈]，幾秒鐘後會出現一個視窗顯示「發佈至 Power BI 成功!」， 並帶有綠色核取記號。 按一下 [在 Power BI 中開啟 demoprediction.pbix] 下方的連結。 若要尋找詳細的指示，請參閱[從 Power BI Desktop 發佈](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop)。
   * 若要建立新儀表板：在左側窗格中按一下 [儀表板] 區段旁的 **+** 號。 為這個新的儀表板輸入名稱「需求預測示範」。
   * 一旦您開啟報告，請按一下 ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic6.png)，將所有視覺效果釘選到儀表板。 若要尋找詳細的指示，請參閱 [從報告將磚釘選至 Power BI 儀表板](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report)。
     前往儀表板頁面並調整視覺效果的大小和位置，以及編輯其標題。 若要尋找如何編輯您的磚的詳細說明，請參閱 [編輯磚 -- 調整大小、移動、重新命名、釘選、刪除、加入超連結](https://powerbi.microsoft.com/documentation/powerbi-service-edit-a-tile-in-a-dashboard/#rename)。 以下是具有釘選了一些冷路徑視覺效果的範例儀表板。

     ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic7.png)
4. (選擇性) 排程資料來源的重新整理。

   * 若要排程資料的重新整理，請將滑鼠移到 **EnergyBPI-Final** 資料集，按一下 ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic3.png)，然後選擇 [排程重新整理]。
     **附註：** 如果您看到警告訊息，請按一下 [編輯認證]，並確定您的資料庫認證與步驟 1 中所述相同。

     ![](media/cortana-analytics-technical-guide-demand-forecast/PowerBIpic4.png)
   * 展開 **排程重新整理** 一節。 開啟「將您的資料保持最新」。
   * 根據您的需求排程重新整理。 若要尋找詳細資訊，請參閱 [Power BI 中的資料重新整理](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/)。

## <a name="how-to-delete-your-solution"></a>**如何刪除解決方案**
請確定您在未積極使用解決方案時已將資料產生器停止，因為執行資料產生器將會產生較高的成本。 如果不使用解決方案，請將其刪除。 刪除解決方案時，會將您在部署解決方案時，於訂用帳戶中佈建的所有元件刪除。 如果要刪除解決方案，請在解決方案範本左側面板中按一下該解決方案的名稱，然後按一下 [刪除]。

## <a name="cost-estimation-tools"></a>**成本估計工具**
下列兩項工具可協助您進一步了解在您的訂用帳戶中執行能源需求預測解決方案範本所牽涉的總成本：

* [Microsoft Azure Cost Estimator Tool (線上版)](https://azure.microsoft.com/pricing/calculator/)
* [Microsoft Azure Cost Estimator Tool (桌面版)](http://www.microsoft.com/download/details.aspx?id=43376)

## <a name="acknowledgements"></a>**通知**
本文是 Microsoft 的資料科學家 Yijing Chen 與軟體工程師 Qiu Min 所撰寫。
