---
title: Azure HDInsight 上的 ML 服務簡介
description: 了解如何使用 HDInsight 上的 ML 服務來建立適用於巨量資料分析的應用程式。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 06/27/2018
ms.openlocfilehash: af143ef1ec79868294af7d7511548dc54f8ceb8d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049265"
---
# <a name="introduction-to-ml-services-and-open-source-r-capabilities-on-hdinsight"></a>HDInsight 上的 ML 服務與開放原始碼 R 功能簡介

> [!NOTE]
> 在 2017 年 9 月，Microsoft R Server 以全新的 **Microsoft Machine Learning Server** 或 ML Server 名稱發行。 因此，HDInsight 上的 R 伺服器叢集現在稱為 HDInsight 上的**機器學習服務**或 **ML 服務**叢集。 如需 R 伺服器名稱變更的相關資訊，請參閱 [Microsoft R 伺服器現為 Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/rebranding-microsoft-r-server#get-support-for-r-server)。

在 Azure 中建立 HDInsight 叢集時，可選擇 Microsoft Machine Learning Server 作為部署選項。 提供此選項的叢集類型稱為 **ML 服務**。 這項功能可讓資料科學家、統計學家以及 R 程式設計人員隨其所需存取 HDInsight 上可調整大小的分散式分析方法。

[!INCLUDE [hdinsight-price-change](../../../includes/hdinsight-enhancements.md)]

HDInsight 上的 ML 服務所提供的最新功能，適用於幾乎任何大小的資料集上所進行的 R 型分析，且不論資料集是載入 Azure Blob 或 Data Lake 儲存體。 ML 服務叢集是根據開放原始碼 R 所建置，因此您建置的 R 型應用程式可以運用 8000 多個開放原始碼 R 套件中的任何一個。 ScaleR 中的常式與 Microsoft 的巨量資料分析套件亦可供使用。

叢集的邊緣節點提供便利的地方，以便連線到叢集以及執行 R 指令碼。 有了邊緣節點之後，即可選擇跨邊緣節點伺服器的核心，執行 ScaleR 的平行分散式函數。 您也可以使用 ScaleR 的 Hadoop Map Reduce 或 Spark 計算內容，跨越叢集的節點來執行這些函數。

可以下載分析所產生的模型或預測，以便在內部部署使用。 它們也可以在 Azure 中的其他地方實際運作，特別是透過 [Azure Machine Learning Studio](http://studio.azureml.net) [Web 服務](../../machine-learning/studio/publish-a-machine-learning-web-service.md)。

## <a name="get-started-with-ml-services-on-hdinsight"></a>開始使用 HDInsight 上的 ML 服務

若要在 Azure HDInsight 中建立 ML 服務叢集，請在使用 Azure 入口網站建立 HDInsight 叢集時，選取 **ML 服務**叢集類型。 ML 服務叢集類型包括在叢集資料節點上的 ML Server，以及在邊緣節點上的 ML Server，可當作 ML 型分析的登陸區域。 請參閱[開始使用 HDInsight 上的 ML 服務](r-server-get-started.md)，以了解如何建立叢集的相關逐步解說。

## <a name="why-choose-ml-services-in-hdinsight"></a>為何選擇 HDInsight 中的 ML 服務？

HDInsight 中的 ML 服務提供下列優點︰

### <a name="ai-innovation-from-microsoft-and-open-source"></a>根據 Microsoft 和開放原始碼的 AI 創新

  ML 服務包括一組高擴充性、分散式的演算法，例如 [RevoscaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)、[revoscalepy](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) 及 [microsoftML](https://docs.microsoft.com/machine-learning-server/python-reference/microsoftml/microsoftml-package)，可用於大小比實體記憶體還要大的資料，並且可透過分散式方式在各種平台上執行。 深入了解產品隨附之 Microsoft 自訂 [R 套件](https://docs.microsoft.com/machine-learning-server/r-reference/introducing-r-server-r-package-reference)和 [Python 套件](https://docs.microsoft.com/machine-learning-server/python-reference/introducing-python-package-reference)的集合。
  
  ML 服務會在單一企業等級平台上，將這些來自開放原始碼社群 (R、Python 及 AI 工具組) 的 Microsoft 創新和貢獻都串連在一起。 任何 R 或 Python 開放原始碼機器學習套件，都可以與任何來自 Microsoft 的專屬創新搭配運作。

### <a name="simple-secure-and-high-scale-operationalization-and-administration"></a>簡單、安全且高擴充性的運作和管理

  依賴傳統範例和環境的企業，會朝運作投入許多時間和精力。 這會導致成本和延遲擴大，包括：模型的平移時間、讓它們保持有效及最新狀態的反覆項目、法規核准，以及管理整個運作的權限。

  ML 服務提供企業級的[運作](https://docs.microsoft.com/machine-learning-server/what-is-operationalization)，也就是在機器學習模型完成後，只要點擊幾下就可以產生 Web 服務 API。 這些 [Web 服務](https://docs.microsoft.com/machine-learning-server/operationalize/concept-what-are-web-services)裝載於伺服器方格或雲端中，而且可以與企業營運應用程式整合。 部署到彈性方格的能力可讓您根據您的商務需求 (針對批次和即時評分) 順暢地進行調整。 如需指示，請參閱[讓 HDInsight 上的 ML 服務能運作](r-server-operationalize.md)。

<!---
* **Deep ecosystem engagements to deliver customer success with optimal total cost of ownership**

  Individuals embarking on the journey of making their applications intelligent or simply wanting to learn the new world of AI and machine learning, need the right resources to help them get started. In addition to this documentation, Microsoft provides several learning resources and has engaged several training partners to help you ramp up and become productive quickly.
--->

## <a name="key-features-of-ml-services-on-hdinsight"></a>HDInsight 上的 ML 服務主要功能

HDInsight 上的 ML 服務包含下列功能。

| 功能分類 | 說明 |
|------------------|-------------|
| 已啟用 R | [R 套件](https://docs.microsoft.com/machine-learning-server/r-reference/introducing-r-server-r-package-reference)適用於以 R 撰寫的解決方案、具有 R 的開放原始碼散佈，以及用於執行指令碼的執行階段基礎結構。 |
| 已啟用 Python | [Python 套件](https://docs.microsoft.com/machine-learning-server/python-reference/introducing-python-package-reference)適用於以 Python 撰寫的解決方案、具有 Python 的開放原始碼散佈，以及用於執行指令碼的執行階段基礎結構。
| [預先定型的模型](https://docs.microsoft.com/machine-learning-server/install/microsoftml-install-pretrained-models) | 針對視覺化分析和文字情感分析，準備好要對您提供的資料進行評分。 |
| [部署和取用](r-server-operationalize.md) | 讓您的伺服器能運作，並將解決方案部署為 Web 服務。 |
| [遠端執行](r-server-hdinsight-manage.md#connect-remotely-to-microsoft-ml-services) | 從用戶端工作站中，在您網路上的 ML 服務啟動遠端工作階段。 |


## <a name="data-storage-options-for-ml-services-on-hdinsight"></a>HDInsight 上適用於 ML 服務的資料儲存體選項

HDInsight 叢集 HDFS 檔案系統的預設儲存體可以與 Azure 儲存體帳戶或 Azure Data Lake Store 關聯。 此關聯可確保在分析期間，上傳至叢集儲存體的任何資料皆保有永續性，而且即使刪除叢集之後還是可以取用資料。 有各種工具可用於處理將資料傳輸到您所選儲存體選項的作業，包括儲存體帳戶的入口網站型上傳工具，以及 [AzCopy](../../storage/common/storage-use-azcopy.md) 公用程式。

不論使用中的儲存體選項為何，您都可以選擇在叢集佈建程序期間，啟用其他 Blob 和 Data Lake 儲存體的存取權。 如需有關新增其他帳戶存取權的詳細資訊，請參閱[開始使用 HDInsight 上的 ML 服務](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-get-started)。 若要深入了解如何使用多個儲存體帳戶，請參閱[適用於 HDInsight 上 ML 服務的 Azure 儲存體選項](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-storage)文章。

您也可以將 [Azure 檔案](../../storage/files/storage-how-to-use-files-linux.md)作為在邊緣節點上使用的儲存體選項。 Azure 檔案可讓您將建立於 Azure 儲存體的檔案共用掛接至 Linux 檔案系統。 如需適用於 HDInsight 叢集上 ML 服務的這些資料儲存體選項詳細資訊，請參閱[適用於 HDInsight 上 ML 服務的 Azure 儲存體選項](r-server-storage.md)。

## <a name="access-ml-services-edge-node"></a>存取 ML 服務邊緣節點

您可以使用瀏覽器連線到邊緣節點上的 Microsoft ML Server。 它預設會在叢集建立期間安裝。 如需詳細資訊，請參閱[開始使用 HDInsight 上的 ML 服務](r-server-get-started.md)。 您也可以使用 SSH/PuTTY 來存取 R 主控台，從命令列連線到叢集邊緣節點。

## <a name="develop-and-run-r-scripts"></a>開發和執行 R 指令碼

您所建立與執行的 R 指令碼，可以任意運用 8000 多種開放原始碼 R 套件，以及 ScaleR 程式庫中的平行與分散式常式。 一般而言，以 ML 服務在邊緣節點上執行的指令碼，會在該節點上的 R 解譯器內執行。 但是必須以設定為 Hadoop Map Reduce (RxHadoopMR) 或 Spark (RxSpark) 之計算內容呼叫 ScaleR 函數的那些步驟除外。 在此情況中，函數會以分散方式，跨越與參考資料相關聯之叢集的那些資料 (工作) 節點執行。 如需不同計算內容選項的詳細資訊，請參閱[適用於 HDInsight 上 ML 服務的計算內容選項](r-server-compute-contexts.md)。

## <a name="operationalize-a-model"></a>模型運作

完成資料模型化時，可以在 Azure 中或內部部署中運作模型，為新的資料進行預測。 這個程序稱為評分。 評分可在 HDInsight、Azure Machine Learning 或內部部署中完成。

### <a name="score-in-hdinsight"></a>在 HDInsight 中評分

若要在 HDInsight 中評分，可以針對已載入至儲存體帳戶的新資料檔案，撰寫可呼叫模型的 R 函數以進行預測。 然後將預測存回儲存體帳戶。 您可以視需要在叢集的邊緣節點上執行這個常式，或使用排程作業來進行。

### <a name="score-in-azure-machine-learning-aml"></a>Azure Machine Learning 中的評分 (AML)

若要使用 Azure Machine Learning 來評分，請使用 Azure Machine Learning R 套件開放原始碼 (稱為 [AzureML](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html)) 來將您的模型發佈為 Azure Web 服務。 為了方便起見，此套件已預先安裝在邊緣節點。 接下來，使用 Azure Machine Learning 中的功能建立 Web 服務的使用者介面，然後呼叫評分所需的 Web 服務。

如果您選擇這個選項，就必須將所有 ScaleR 模型物件轉換成對等的開放原始碼模型物件，才可搭配 Web 服務使用。 針對此轉換，您可以使用 ScaleR 強制型轉函數 (例如，適用於集成模型的 `as.randomForest()`) 來完成。

### <a name="score-on-premises"></a>內部部署評分

若要在建立模型之後進行內部部署評分，可以在 R 中將模型序列化，下載它、將它還原序列化，然後使用它進行新資料的評分。 您可以使用稍早在[在 HDInsight 中評分](#scoring-in-hdinsight)中所述的方法，或使用 [Web 服務](https://docs.microsoft.com/machine-learning-server/operationalize/concept-what-are-web-services) \(英文\) 進行新資料的評分。

## <a name="maintain-the-cluster"></a>維護叢集

### <a name="install-and-maintain-r-packages"></a>安裝及維護 R 套件

由於 R 指令碼大多數的步驟會在邊緣節點上執行，因此在邊緣節點上需要有大多數您所使用的 R 套件。 若要在邊緣節點上安裝其他 R 套件，可以在 R 中使用 `install.packages()` 方法。

如果您只是在整個叢集中使用來自 ScaleR 程式庫的常式，則通常不需要在資料節點上安裝其他 R 套件。 但是，您可能需要其他套件，以支援在資料節點上使用 **rxExec** 或 **RxDataStep** 執行。

在這種情況下，您可以在建立叢集之後，使用指令碼動作來安裝其他套件。 如需詳細資訊，請參閱[管理 HDInsight 叢集中的 ML 服務](r-server-hdinsight-manage.md)。

### <a name="change-hadoop-mapreduce-memory-settings"></a>變更 Hadoop MapReduce 記憶體設定

您可以在執行 MapReduce 作業時，修改叢集以變更 ML 服務的可用記憶體數量。 若要修改叢集，請使用可透過叢集的 Azure 入口網站刀鋒視窗存取的 Apache Ambari UI。 如需如何存取叢集的 Ambari UI 的指示，請參閱[使用 Ambari Web UI 管理 HDInsight 叢集](../hdinsight-hadoop-manage-ambari.md)。

您也可以在對 **RxHadoopMR** 的呼叫中使用 Hadoop 參數，變更 ML 服務可用的記憶體數量，如下所示：

    hadoopSwitches = "-libjars /etc/hadoop/conf -Dmapred.job.map.memory.mb=6656"  

### <a name="scale-your-cluster"></a>調整叢集的大小

您可以透過入口網站，相應增加或相應減少 HDInsight 上的現有 ML 服務叢集。 藉由相應增加，您可以取得處理較大型工作所需的額外處理能力，或於叢集閒置時調回處理能力。 如需如何調整叢集規模的指示，請參閱[管理 HDInsight 叢集](../hdinsight-administer-use-portal-linux.md)。

### <a name="maintain-the-system"></a>維護系統

離峰期間，系統會在 HDInsight 叢集的基底 Linux VM 上執行維護，以套用 OS 修補程式和其他更新。 一般而言，維護作業會在每星期一和星期四上午 3:30 (以 VM 的本地時間為準) 進行。 執行更新時，系統會以一次僅影響四分之一以內叢集的方式進行。

由於前端節點是備援節點，且並非所有資料節點都會受到影響，因此在這段期間執行的任何工作可能會變慢。 不過，應該都可執行完成。 除非發生需要重建叢集的嚴重失敗，否則您擁有的任何自訂軟體或本機資料，在這些維護事件中皆會保留。

## <a name="ide-options-for-ml-services-on-hdinsight"></a>HDInsight 上適用於 ML 服務的 IDE 選項

HDInsight 叢集的 Linux 邊緣節點，是 R 型分析的登陸區域。 HDInsight 最近的幾個版本，提供在邊緣節點上將 RStudio Server 安裝為瀏覽器型 IDE 的預設選項。 比起僅使用 R 主控台，使用 RStudio Server 當作開發及執行 R 指令碼的 IDE 可大幅提升生產力。

此外，您可以安裝電腦 IDE 並透過使用遠端 MapReduce 或 Spark 計算內容，使用它來存取叢集。 選項包括 Microsoft 的 [Visual Studio R 工具](https://www.visualstudio.com/features/rtvs-vs.aspx) (RTVS)、RStudio 與 Walware 的 Eclipse 型 [StatET](http://www.walware.de/goto/statet)。

此外，透過 SSH 或 PuTTY 連線，您就可以在 Linux 命令提示字元中輸入 **R** 來存取邊緣節點上的 R 主控台。 使用主控台介面時，在另一個視窗中針對 R 指令碼開發執行文字編輯器，並視需要剪下指令碼區段並貼上到 R 主控台中是非常方便的。

## <a name="pricing"></a>價格

與 ML 服務 HDInsight 叢集相關聯的價格結構，類似於 HDInsight 叢集類型的價格。 以各種名稱、資料和邊緣節點的基礎 VM 大小為基準，再加上彈性工時加費。 如需詳細資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

## <a name="next-steps"></a>後續步驟

若要深入了解如何使用 HDInsight 叢集上的 ML 服務，請參閱下列主題：

* [開始使用 HDInsight 上的 ML 服務叢集](r-server-get-started.md)
* [在 HDInsight 上計算 ML 服務叢集的內容選項](r-server-compute-contexts.md)
* [HDInsight 上適用於 ML 服務叢集的儲存體選項](r-server-storage.md)
