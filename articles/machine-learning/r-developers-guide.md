---
title: Azure 的 R 開發人員指南 - R 程式設計 | Microsoft Docs
description: 本文概述各種方式，讓資料科學家能夠在 Azure 中對 R 程式設計語言運用其現有技術。 Azure 提供了許多服務，供 R 開發人員利用以將其資料科學工作負載延伸到雲端。
services: machine-learning
documentationcenter: ''
author: AnalyticJeremy
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: R
ms.topic: article
ms.date: 09/12/2018
ms.author: jepeach
ms.openlocfilehash: bc00bd3b61398355c663d133c0c9a66c2a52aa8d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045874"
---
# <a name="r-developers-guide-to-azure"></a>Azure 的 R 開發人員指南
<img src="media/r-developers-guide/logo_r.svg" alt="R logo" align="right" width="200" />

由於要處理的資料數量不斷增加，許多資料科學家想盡辦法來利用雲端運算的能力以便完成其分析工作。  本文概述各種方式，讓資料科學家能夠在 Azure 中對 [R 程式設計語言](https://www.r-project.org)運用其現有技術。

Microsoft 已完全採用 R 程式設計語言作為資料科學家的頂級工具。  藉由提供許多不同的選項供 R 開發人員在 Azure 中執行其程式碼，該公司讓資料科學家能夠在解決大規模專案時，將其資料科學工作負載延伸到雲端。

讓我們檢查一下各種選項以及其各自最受矚目的案例。

## <a name="azure-services-with-r-language-support"></a>具有 R 語言支援的 Azure 服務
本文涵蓋下列可支援 R 語言的 Azure 服務：

|服務                                                          |說明                                                                       |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------|
|[資料科學虛擬機器](#data-science-virtual-machine)    |自訂 VM，可作為資料科學工作站或作為自訂計算目標|
|[HDInsight 上的 ML 服務](#ml-services-on-hdinsight)            |叢集式系統，可對跨許多節點的大型資料集執行 R 分析   |
|[Azure Databricks](#azure-databricks)                            |支援 R 和其他語言的共同作業 Spark 環境               |
|[Azure Machine Learning Studio](#azure-machine-learning-studio)  |在 Azure 的機器學習實驗中執行自訂的 R 指令碼                      |
|[Azure Batch](#azure-batch)                                      |提供各種不同選項，以便以經濟實惠的方式跨叢集中的許多節點執行 R 程式碼|
|[Azure Notebooks](#azure-notebooks)                              |免費 (但功能受限) 的雲端式 Jupyter Notebook 版本                  |
|[Azure SQL Database](#azure-sql-database)                        |在 SQL Server 資料庫引擎內執行 R 指令碼                            |

## <a name="data-science-virtual-machine"></a>資料科學虛擬機器
[資料科學虛擬機器 (DSVM)](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) 是 Microsoft Azure 雲端平台上的自訂 VM 映像，專為進行資料科學建置。 其具有許多熱門的資料科學工具，包括：
* [Microsoft R Open](https://mran.microsoft.com/open/)
* [Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server)
* [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop)
* [RStudio Server](https://www.rstudio.com/products/rstudio/#Server)

DSVM 可以搭配 Windows 或 Linux 作業系統來佈建。  您可以透過兩種不同的方式使用 DSVM：作為自訂叢集的互動式工作站或計算平台。

### <a name="as-a-workstation"></a>作為工作站
如果您想要在雲端中快速且輕鬆地開始使用 R，這是您的最佳選擇。  已在本機工作站上使用過 R 的人會相當熟悉此環境。  不過，R 環境不會使用本機資源，而是會在雲端 VM 上執行。  如果資料已儲存在 Azure 中，其另外的好處是可讓 R 指令碼以「更接近資料」的方式執行。 您不必透過網際網路傳輸資料，而是可以透過 Azure 的內部網路存取資料，以提供更快的存取速度。

DSVM 特別適用於小型的 R 開發人員團隊。  您不必為每位開發人員投資功能強大的工作站，團隊成員也不必將他們所會使用的各種軟體套件版本同步，每位開發人員都可以在需要時立即啟動 DSVM 執行個體。

### <a name="as-a-compute-platform"></a>作為計算平台
除了作為工作站，DSVM 也可作為適用於 R 專案的可彈性調整計算平台。  使用 <code>[AzureDSVM](https://github.com/Azure/AzureDSVM)</code> R 套件，您就可以透過程式設計方式控制 DSVM 執行個體的建立和刪除。  您可以將執行個體組成叢集，並部署要在雲端執行的分散式分析。  這整個程序可以由本機工作站上所執行的 R 程式碼來控制。

若要深入了解 DSVM，請參閱[適用於 Linux 和 Windows 的 Azure 資料科學虛擬機器簡介。](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="ml-services-on-hdinsight"></a>HDInsight 上的 ML 服務
[Microsoft ML 服務](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-overview)可讓資料科學家、統計學家以及 R 程式設計人員隨其所需存取 HDInsight 上可調整大小的分散式分析方法。  此解決方案所提供的最新功能，適用於幾乎任何大小的資料集上進行之以 R 為基礎的分析，且不論資料集是載入到 Azure Blob 或 Data Lake 儲存體。

這是企業級解決方案，可讓您跨叢集調整 R 程式碼。  藉由利用 Microsoft <code>[RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)</code> 套件中的函式，您在 HDInsight 上的 R 指令碼可以跨叢集中的許多節點平行執行資料處理函式。
<ph id="ph1">&lt;code&gt;[RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)&lt;/code&gt;</ph> package, your R scripts on HDInsight can run data processing functions in parallel across many nodes in a cluster.  這可讓 R 處理更大規模的資料，在工作站上執行的單一執行緒 R 就做不到這一點。

這項調整能力讓 HDInsight 上的 ML 服務成為絕佳選項，可讓 R 開發人員處理大規模資料集。  其提供彈性且可擴充的平台供您在雲端中執行 R 指令碼。

如需如何建立 ML 服務叢集的逐步解說，請參閱[開始使用 Azure HDInsight 上的 ML 服務](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-get-started)一文。

## <a name="azure-databricks"></a>Azure Databricks
[Azure Databricks](https://azure.microsoft.com/services/databricks/) 是一個針對 Microsoft Azure 雲端服務平台進行最佳化的 Apache Spark 分析平台。  Databricks 由 Apache Spark 的創立者所設計，可與 Azure 整合，提供一鍵式設定、順暢的工作流程以及互動式的工作區，可讓資料科學家、資料工程師及企業分析師共同作業。

平台的 Notebook 系統會啟用 Databricks 中的共同作業功能。  使用者可以和系統內的其他使用者建立、共用和編輯 Notebook。  這些 Notebook 可讓使用者撰寫程式碼，以針對 Databricks 環境中所管理的 Spark 叢集來執行。  這些 Notebook 完全支援 R，並可讓使用者透過 `SparkR` 和 `sparklyr` 套件存取 Spark。

因為 Databricks 建置在 Spark 之上，而且特別著重於共同作業，所以資料科學家團隊通常會使用此平台來共同處理複雜的大型資料集分析。  由於 Databricks 中的 Notebook 不僅支援 R 也支援其他語言，所以特別適用於分析師會使用不同語言來進行其主要工作的團隊。

[何謂 Azure Databricks？](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks)一文
可以提供更多關於此平台的詳細資料，並協助您開始使用。

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio
[Azure Machine Learning Studio](https://azure.microsoft.com/services/machine-learning-studio/) 是共同作業式的拖放工具，可供您在雲端中建置、測試及部署預測性分析解決方案。  它可讓新興的資料科學家建立和部署機器學習模型，而不需要撰寫龐大的程式碼。

ML Studio 同時支援 R 和 Python。  您可以透過兩種方式搭配使用 R 與 ML Studio。

### <a name="custom-r-scripts-in-your-experiments"></a>在實驗中自訂 R 指令碼
首先，您可以藉由撰寫自訂 R 指令碼，擴充 ML Studio 的資料操作和機器學習功能。
雖然 ML Studio 包含各種不同的資料準備和分析模組，但它所擁有的功能比不上已經成熟的語言 (例如 R)。因此，這項服務的設計，是要讓您能夠在所提供的模組不符合需求的情況下，引進您自己的自訂 R 指令碼。

若要利用這項功能，請將「執行 R 指令碼」模組拖放到實驗中。  然後在 [屬性] 窗格中使用程式碼編輯器，來撰寫新的 R 指令碼或貼上現有指令碼。  在指令碼中，您可以參考外部 R 套件。  您可以使用指令碼來操作資料，也可以將不屬於標準 ML Studio 模型庫的複雜 ML 模型定型。

如需在 ML Studio 實驗中使用 R 的完整介紹，請參閱 [Azure Machine Learning 的 R 程式設計語言快速入門教學課程。](https://docs.microsoft.com/azure/machine-learning/studio/r-quickstart)

### <a name="create-manage-and-deploy-experiments-from-your-local-r-environment"></a>從本機 R 環境建立、管理及部署實驗
另一種搭配使用 R 與 ML Studio 的方式，是使用 <code>[AzureML](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html)</code> 套件來監視和控制在 R 程式設計環境上所進行的測試程序。
<ph id="ph1">&lt;code&gt;[AzureML](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html)&lt;/code&gt;</ph> package to monitor and control the experimentation process with the R programming environment.  此套件會由 Microsoft 維護，其可讓您在 Azure ML 中上傳和下載資料集，以便詢問實驗、將 R 函式發佈為 Azure ML Web 服務，以及透過現有 Web 服務執行 R 資料並擷取輸出。

此套件可讓您更容易地將 Azure ML 作為 R 程式碼的可調整部署平台。  您不必在 UI 中按一下並拖曳，而是可以使用您已熟悉的工具自動進行整個部署程序。

## <a name="azure-batch"></a>Azure Batch
對於大規模的 R 作業，您可以使用 [Azure Batch](https://azure.microsoft.com/services/batch/)。  此服務可提供雲端級別的作業排程和計算管理，讓您可以跨數十個、數百個或數千個虛擬機器調整 R 工作負載。  因為它是一般化的運算平台，所以有幾個選項可供您在 Azure Batch 上執行 R 作業。

其中一個選項是使用 Microsoft 的 <code>[doAzureParallel](https://github.com/Azure/doAzureParallel)</code> 套件。  此 R 套件是 `foreach` 套件的平行後端。  它可讓 `foreach` 迴圈的每次反覆運算在 Azure Batch 叢集內的節點上平行執行。  如需套件的簡介，請閱讀 [doAzureParallel：直接從 R 工作階段利用 Azure 的彈性計算](https://azure.microsoft.com/blog/doazureparallel/)部落格文章。

另一個在 Azure Batch 中執行 R 指令碼的選項，是在 Azure 入口網站中將程式碼與 "RScript.exe" 組合成 Batch 應用程式。  如需詳細的逐步解說，請參閱 [Azure Batch 上的 R 工作負載。](https://azure.microsoft.com/blog/r-workloads-on-azure-batch/)

第三個選項是使用 [Azure 分散式資料工程工具組](https://github.com/Azure/aztk) (AZTK)，其可讓您使用 Azure Batch 中的 Docker 容器佈建隨選 Spark 叢集。  這可讓您以經濟實惠的方式在 Azure 中執行 Spark 作業。  藉由搭配使用 [SparklyR 與 AZTK](https://github.com/Azure/aztk/wiki/SparklyR-on-Azure-with-AZTK)，便可在雲端中以輕鬆且經濟實惠的方式相應放大 R 指令碼。

## <a name="azure-notebooks"></a>Azure Notebooks
對於偏好使用 Notebook 來將程式碼帶入 Azure 的 R 開發人員來說，[Azure Notebook](https://notebooks.azure.com) 是低成本、低摩擦的方法。  它是一項免費服務，可供任何人在瀏覽器中使用 [Jupyter](https://jupyter.org/) 來開發和執行程式碼，Jupyter 是開放原始碼專案，可將 Markdown Prose、可執行程式碼和圖形合併至單一畫布。

雖然 Azure Notebook 適用於小規模專案，但由於有某些限制，因此不適用於大規模的資料科學專案。  目前，此服務限制每個 Notebook 的程序只能使用 4 GB 記憶體，資料集只能有 1 GB。  不過，若要發佈較小型的分析，這會是簡單、免費的選項。

## <a name="azure-sql-database"></a>連接字串
[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 是 Microsoft 的智慧型、完全受控關聯式雲端資料庫服務。  它可讓您使用 SQL Server 的完整功能，而不需要麻煩地設定基礎結構。  這包括[機器學習服務](https://docs.microsoft.com/sql/advanced-analytics/what-is-sql-server-machine-learning?view=sql-server-2017)，這是 SQL 服務最近新增的項目。

這項功能可提供內嵌的預測性分析和資料科學引擎，其可在 SQL Server 資料庫內以預存程序的形式執行 R 程式碼、以 包含 R 陳述式的 T-SQL 指令碼形式執行 R 程式碼，或以包含 T-SQL 的 R 程式碼形式執行 R 程式碼。  您不必從資料庫擷取資料並載入至 R 環境，而是可以將 R 程式碼直接載入到資料庫，並讓它直接與資料一起執行。

雖然機器學習服務自 2016 年起就已成為內部部署 SQL Server 的一部分，但對於 Azure SQL Database 來說，這還是相當新的功能。  它目前是[有限預覽版](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#azure-sql-database-roadmap)，但將會持續發展。


### <a name="next-steps"></a>後續步驟
* [在 Azure 上使用 mrsdeploy 執行 R 程式碼](http://blog.revolutionanalytics.com/2017/03/running-your-r-code-azure.html)
* [雲端中的 Machine Learning Server](https://docs.microsoft.com/machine-learning-server/install/machine-learning-server-in-the-cloud)
* [Machine Learning Server 和 Microsoft R 的其他資源](https://docs.microsoft.com/machine-learning-server/resources-more)
* [Azure 上的 R](https://github.com/yueguoguo/r-on-azure) -概述套件、工具和搭配使用 R 與 Azure 的案例研究

---

<sub>R 標誌是 &copy; 2016 The R Foundation，係依據 [Creative Commons Attribution-ShareAlike 4.0 國際授權](https://creativecommons.org/licenses/by-sa/4.0/)的條款來使用。</sub>
