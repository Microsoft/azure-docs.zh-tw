---
title: 資料科學 Team 專案的平台和工具 - Azure  | Microsoft Docs
description: 逐項列出及討論可以讓企業對 Team 資料科學程序進行標準化的資料和分析資源。
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/04/2017
ms.author: deguhath
ms.openlocfilehash: 0c5cfaad096b5702f321eead1f3da0d9225f4b53
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302855"
---
# <a name="platforms-and-tools-for-data-science-projects"></a>資料科學專案的平台和工具

Microsoft 提供雲端或內部部署平台的完整範圍資料和分析服務與資源。 它們可以部署以有效率且可調整地執行資料科學專案。 [Team 資料科學程序](overview.md) (TDSP) 中提供小組以可追蹤、版本控制及共同作業的方式實作資料科學專案的指引。  如需人員角色的大綱，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。

可供使用 TDSP 之資料科學小組利用的資料與分析服務包括：

- 資料科學虛擬機器 (Windows 和 Linux CentOS)
- HDInsight Spark 叢集
- SQL 資料倉儲
- Azure Data Lake
- HDInsight Hive 叢集
- Azure 檔案儲存體
- SQL Server 2016 R Services

在本文件中，我們簡短描述資源，並且提供 TDSP 小組已發佈之教學課程和逐步解說的連結。 它們可以協助您了解如何逐步使用它們，以及開始使用它們來建置智慧型應用程式。 這些資源的其他資訊可以在其產品分頁上取得。 

## <a name="data-science-virtual-machine-dsvm"></a>資料科學虛擬機器 (DSVM)

Microsoft 對於 Windows 和 Linux 所提供的資料科學虛擬機器，包含資料科學模型化和開發活動的熱門工具。 包含的工具例如：

- Microsoft R Server Developer Edition 
- Anaconda Python 散佈
- Python 和 R 適用的 Jupyter Notebook 
- Visual Studio Community Edition，具有 Windows / Eclipse on Linux 上的 Python 和 R 工具
- 適用於 Windows 的 Power BI desktop
- Windows / Postgres on Linux 上的 SQL Server 2016 Developer Edition

它也包含 **ML 和 AI 工具**，例如 CNTK (Microsoft 的 Open Source Deep Learning 工具組)、xgboost、mxnet 和 Vowpal Wabbit。

目前 DSVM 可用於 **Windows** 和 **Linux CentOS** 作業系統。 根據您規劃在上面執行之資料科學專案的需要，選擇 DSVM 的大小(CPU 核心數目和記憶體數量)。 

如需 Windows 版本 DSVM 的詳細資訊，請參閱 Azure Marketplace 上的 [Microsoft 資料科學虛擬機器](https://azure.microsoft.com/marketplace/partners/microsoft-ads/standard-data-science-vm/)。 對於 Linux 版本 DSVM，請參閱 [Linux 資料科學虛擬機器](https://azure.microsoft.com/marketplace/partners/microsoft-ads/linux-data-science-vm/)。

若要深入了解如何在 DSVM 上有效率地執行某些一般資料科學工作，請參閱[您可以在資料科學虛擬機器上做的十件事](../data-science-virtual-machine/vm-do-ten-things.md)


## <a name="azure-hdinsight-spark-clusters"></a>Azure HDInsight Spark 叢集

Apache Spark 是一個開放原始碼平行處理架構，可支援記憶體內部處理，大幅提升巨量資料分析應用程式的效能。 Spark 處理引擎是專為速度、易用性及精密分析打造的產品。 Spark 的記憶體內計算功能，使其成為機器學習和圖表計算中反覆演算法的絕佳選擇。 Spark 也能與 Azure Blob 儲存體 (WASB) 相容，因此您可以輕鬆地使用 Spark 處理儲存在 Azure 中的現有資料。

當您在 HDInsight 中建立 Spark 叢集時，就是建立了已安裝及設定 Spark 的 Azure 計算資源。 在 HDInsight 中建立 Spark 叢集需要約 10 分鐘。 系統會將要處理的資料儲存在 Azure Blob 儲存體。 如需關於搭配叢集使用 Azure Blob 儲存體的詳細資訊，請參閱[在 HDInsight 上搭配 Hadoop 使用 HDFS 相容的 Azure Blob 儲存體](../../hdinsight/hdinsight-hadoop-use-blob-storage.md)。

Microsoft 的 TDSP 小組已發佈兩個端對端逐步解說，示範如何使用 Azure HDInsight Spark 叢集來建立資料科學解決方案，其中一個使用 Python，而另一個使用 Scala。 如需 Azure HDInsight **Spark 叢集**的詳細資訊，請參閱[概觀：HDInsight Linux 上的 Apache Spark](../../hdinsight/spark/apache-spark-overview.md)。 若要了解如何在 Azure HDInsight Spark 叢集上使用 **Python** 來建立資料科學解決方案，請參閱[在 Azure HDInsight 上使用 Spark 的資料科學概觀](spark-overview.md)。 若要了解如何在 Azure HDInsight Spark 叢集上使用 **Scala** 來建立資料科學解決方案，請參閱[在 Azure 上使用 Scala 與 Spark 的資料科學](scala-walkthrough.md)。 


##  <a name="azure-sql-data-warehouse"></a>Azure SQL 資料倉儲

Azure SQL 資料倉儲可讓您輕鬆地在數秒鐘的時間內調整計算資源，而不會過度佈建或過度支付。 它也會提供可暫停使用計算資源的唯一選項，讓您有彈性能更妥當地管理雲端成本。 部署可調整計算資源的能力可讓您將所有資料帶入 Azure SQL 資料倉儲。 儲存體成本很低，您可以只對您想要分析的資料集部分執行計算。 

如需 Azure SQL 資料倉儲的詳細資訊，請參閱 [SQL 資料倉儲](https://azure.microsoft.com/services/sql-data-warehouse)網站。 若要深入了解如何使用 SQL 資料倉儲建置端對端進階分析解決方案，請參閱[Team 資料科學程序實務：使用 SQL 資料倉儲](sqldw-walkthrough.md)。


## <a name="azure-data-lake"></a>Azure Data Lake

Azure Data Lake 是企業級的存放庫，可於單一位置收集每種資料類型，再正式加上任何需求或結構描述。 這種彈性可讓每種類型的資料都保存在 Data Lake，不論其大小或結構或者它內嵌的速度。 組織可以使用 Hadoop 或進階分析，在這些 Data Lake 中尋找模式。 Data Lake 也可以在策劃資料並且將其移至資料倉儲之前，作為較低成本資料準備的存放庫。

如需有關 Azure Data Lake 的詳細資訊，請參閱[簡介 Azure Data Lake](https://azure.microsoft.com/blog/introducing-azure-data-lake/)。 若要了解如何使用 Azure Data Lake 建置可調整端對端資料科學解決方案，請參閱 [Azure Data Lake 中的可調整資料科學︰端對端逐步解說](data-lake-walkthrough.md)


## <a name="azure-hdinsight-hive-hadoop-clusters"></a>Azure HDInsight Hive (Hadoop) 叢集

Apache Hive 是適用於 Hadoop 的資料倉儲系統，可透過 HiveQL (與 SQL 類似的查詢語言) 執行資料摘要、查詢以及資料分析。 Hive 可以互動方式用於探索資料，或者建立可重複使用的批次處理工作。

Hive 可讓您將結構投影在大量非結構化資料上。 定義結構後，您不需使用 Jave 或 MapReduce，甚至不需具備其相關知識，即可使用 Hive 查詢該資料。 HiveQL (Hive 查詢語言) 可讓您使用類似於 T-SQL 的陳述式撰寫查詢。

對於資料科學家，Hive 可以在 Hive 查詢中執行 Python 使用者定義函式 (UDF) 以處理記錄。 這項能力會大幅擴充資料分析中 Hive 查詢的功能。 具體來說，它可讓資料科學家以幾乎完全熟悉的語言執行擴充功能工程：類似 SQL 的 HiveQL 和 Python。 

如需有關 Azure HDInsight Hive 叢集的詳細資訊，請參閱[在 HDInsight 中將 Hive 和 HiveQL 與 Hadoop 搭配使用](../../hdinsight/hadoop/hdinsight-use-hive.md)。 若要了解如何使用 Azure HDInsight Hive 叢集來建置可調整端對端資料科學解決方案，請參閱 [Team 資料科學程序實務：使用 HDInsight Hadoop 叢集](hive-walkthrough.md)。


## <a name="azure-file-storage"></a>Azure 檔案儲存體 

Azure 檔案儲存體是使用標準伺服器訊息區塊 (SMB) 通訊協定，在雲端中提供檔案共用功能的服務。 SMB 2.1 和 SMB 3.0 皆受到支援。 使用 Azure 檔案儲存體時，您可以快速地將依賴檔案共用功能的舊式應用程式移轉至 Azure，而不必浪費成本來重新撰寫程式。 在 Azure 虛擬機器、雲端服務或內部部署中執行的應用程式，可掛接雲端中的檔案共用，就像桌面應用程式掛接一般 SMB 共用一樣。 可同時掛接和存取檔案儲存體共用的應用程式元件數量沒有限制。

對於資料科學專案特別有用的功能是建立 Azure 檔案存放區，作為與專案小組成員共用專案資料的位置。 然後每個成員會存取 Azure 檔案儲存體中資料的相同複本。 他們也可以使用此檔案儲存體，共用在執行專案期間產生的功能集。 如果專案是用戶端參與，您的用戶端可以在他們自己的 Azure 訂用帳戶底下建立 Azure 檔案存放體，與您共用專案資料和功能。 如此一來，用戶端會具有專案資料資產的完整控制權。 如需 Azure 檔案儲存體的詳細資訊，請參閱[在 Windows 上開始使用 Azure 檔案儲存體](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-files)和[如何搭配 Linux 使用 Azure 檔案儲存體](../../storage/files/storage-how-to-use-files-linux.md)。


## <a name="sql-server-2016-r-services"></a>SQL Server 2016 R Services

R Services (資料庫內) 提供用於開發和部署智慧型應用程式的平台，可以發掘新的見解。 您可以使用豐富且功能強大的 R 語言 (包括 R 社群提供的許多套件) 來建立模型，並為 SQL Server 資料產生預測。 因為 R Services (資料庫內) 會整合 R 語言與 SQL Server，所以分析會貼近資料，這樣可以排除與移動資料相關聯的成本和安全性風險。

R Services (資料庫內) 支援開放原始碼 R 語言與一組完整的 SQL Server 工具和技術。 它們提供更優異的效能、安全性、可靠性及管理能力。 您可以使用方便且熟悉的工具來部署 R 解決方案。 生產應用程式可以使用 Transact-SQL 呼叫 R 執行階段，並且擷取預測和視覺效果。 您也可以使用 ScaleR 程式庫來改善 R 解決方案的規模和效能。 如需詳細資訊，請參閱 [SQL Server R Services](https://docs.microsoft.com/sql/advanced-analytics/r/sql-server-r-services)。

Microsoft 的 TDSP 小組已發佈兩個端對端逐步解說，示範如何在 SQL Server 2016 R Services 中建置資料科學解決方案：一個適用於 R 程式設計人員，另一個適用於 SQL 開發人員。 若是 **R 程式設計人員**，請參閱[資料科學端對端逐步解說](https://docs.microsoft.com/sql/advanced-analytics/tutorials/walkthrough-data-science-end-to-end-walkthrough)。 若是 **SQL 開發人員**，請參閱[適用於 SQL 開發人員的資料庫內進階分析 (教學課程)](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-in-database-r-for-sql-developers)。


## <a name="appendix"></a>附錄：用來設定資料科學專案的工具

### <a name="install-git-credential-manager-on-windows"></a>在 Windows 上安裝 Git 認證管理員

如果您遵照 **Windows** 上的 TDSP，您需要安裝 **Git 認證管理員 (GCM)**，以與 Git 存放庫通訊。 若要安裝 GCM，您必須先安裝 **Chocolaty**。 若要安裝 Chocolaty 和 GCM，以**系統管理員**身分在 Windows PowerShell 中執行下列命令：  

    iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
    choco install git-credential-manager-for-windows -y
    

### <a name="install-git-on-linux-centos-machines"></a>在 Linux (CentOS) 機器上安裝 Git

執行下列 bash 命令以在 Linux (CentOS) 機器上安裝 Git：

    sudo yum install git


### <a name="generate-public-ssh-key-on-linux-centos-machines"></a>產生 Linux (CentOS) 機器的公開 SSH 金鑰

如果您使用 Linux (CentOS) 機器來執行 git 命令，您必須將機器的公用 SSH 金鑰新增至 Azure DevOps Services，讓 Azure DevOps Services 能夠辨識此機器。 首先，您必須產生公開 SSH 金鑰，並將金鑰新增至 Azure DevOps Services 安全性設定頁面中的 SSH 公開金鑰。 

- 若要產生 SSH 金鑰，請執行下列兩個命令： 

        ssh-keygen
        cat .ssh/id_rsa.pub

![](./media/platforms-and-tools/resources-1-generate_ssh.png)

- 複製整個 ssh 金鑰，包括 ssh-rsa。 
- 登入您的 Azure DevOps Services。 
- 在分頁右上角，按一下 [<您的名稱\>]，然後按一下 [安全性]。 
    
    ![](./media/platforms-and-tools/resources-2-user-setting.png)

- 按一下 [SSH 公開金鑰]，然後按一下 [+新增]。 

    ![](./media/platforms-and-tools/resources-3-add-ssh.png)

- 將剛剛複製的 ssh 金鑰貼至文字方塊，並且儲存。


## <a name="next-steps"></a>後續步驟

此外也會提供完整的端對端逐步解說，說明 **特定案例** 之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)主題中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。 

針對使用 Azure Machine Learning Studio 的 Team Data Science Process，如需其中執行步驟的範例，請參閱[使用 Azure ML](http://aka.ms/datascienceprocess) 學習路徑。