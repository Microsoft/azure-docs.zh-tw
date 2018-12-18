---
title: 大量資料的伺服器工作負載預測 - Azure | Microsoft Docs
description: 如何使用 Azure Machine Learning Workbench 定型巨量資料的機器學習模型。
services: machine-learning
documentationcenter: ''
author: daden
manager: mithal
editor: daden
ms.assetid: ''
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/15/2017
ms.author: daden
ROBOTS: NOINDEX
ms.openlocfilehash: 8f3bd4e62aa85c69a0bfafeacf13bc3e472136d5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964696"
---
# <a name="server-workload-forecasting-on-terabytes-of-data"></a>資料的伺服器工作負載預測 (TB)

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

本文說明資料科學家如何使用 Azure Machine Learning Workbench 來開發需要使用巨量資料的解決方案。 您可以從大型資料集的範例開始，經歷資料準備、特徵設計和機器學習，然後將此程序延伸至整個大型資料集。 

您將學會 Machine Learning Workbench 的下列主要功能：
* 在計算目標之間輕鬆切換。 您可以設定不同的計算目標並在實驗中使用它們。 在此範例中，您會使用 Ubuntu DSVM 與 Azure HDInsight 叢集作為計算目標。 您也可以根據資源的可用性來設定計算目標。 特別是在將 Spark 叢集相應放大至更多背景工作節點之後，您可以透過 Machine Learning Workbench 使用資源來加速實驗執行。
* 執行歷程記錄追蹤。 您可以使用 Machine Learning Workbench 追蹤機器學習模型的效能和其他您感興趣的計量資訊。
* 機器學習模型的運算化。 您可以使用 Machine Learning Workbench 內建的工具，將定型的機器學習模型部署為 Azure Container Service 上的 Web 服務。 您也可以使用 Web 服務，透過 REST API 呼叫來取得迷你批次預測。 
* 大量資料的支援。

> [!NOTE]
> 如需程式碼範例及其他與此範例相關的資料，請參閱 [GitHub](https://github.com/Azure/MachineLearningSamples-BigData)。
> 

## <a name="use-case-overview"></a>使用案例概觀

對於管理自有基礎結構的科技公司而言，預測伺服器的工作負載是常見的商務需求。 為了減少基礎結構的成本，在未充分利用的伺服器上執行的服務應群組在一起，以便在較少量的機器上執行。 而在過度使用的伺服器上執行的服務，則應該用更多機器執行。 

在此案例中，您將焦點放在每部機器 (或伺服器) 的工作負載預測。 特別的是，您會使用每部伺服器上的工作階段資料，預測未來伺服器的工作負載類別。 您使用 [Apache Spark ML](https://spark.apache.org/docs/2.1.1/ml-guide.html) 中的隨機樹系分類器，將每個伺服器的負載歸類為低、中、高三類。 在此範例中的機器學習技術和工作流程可以輕易地延伸到其他類似的問題。 


## <a name="prerequisites"></a>必要條件

要執行此範例所需符合的必要條件如下：

* [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
* 已安裝 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md)。 若要安裝程式並建立工作區，請參閱[快速入門安裝指南](quickstart-installation.md)。 如果您有多個訂用帳戶，您可以[將所需訂用帳戶設定為目前作用中的訂用帳戶](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az_account_set)。
* Windows 10 (本範例中的指示大多與 macOS 系統通用)。
* 適用於 Linux (Ubuntu) 的資料科學虛擬機器 (DSVM)，最好位於資料所在的美國東部區域。 您可以依照[這些指示](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro)來佈建 Ubuntu DSVM。 您也可以參閱[這篇快速入門](https://ms.portal.azure.com/#create/microsoft-ads.linux-data-science-vm-ubuntulinuxdsvmubuntu)。 我們建議使用至少 8 個核心和 32 GB 記憶體的虛擬機器。 

請依照[指示](../desktop-workbench/known-issues-and-troubleshooting-guide.md#remove-vm-execution-error-no-tty-present)為 AML Workbench 啟用虛擬機器上的無密碼 sudoer 存取。  您可以選擇使用[以 SSH 金鑰為基礎的驗證，以在 AML Workbench 中建立及使用虛擬機器](experimentation-service-configuration.md#using-ssh-key-based-authentication-for-creating-and-using-compute-targets)。 在此範例中，我們會使用密碼來存取虛擬機器。  儲存含有 DSVM 資訊的下列表格，以供後續步驟使用：

 欄位名稱| 值 |  
 |------------|------|
DSVM IP 位址 | xxx|
 使用者名稱  | xxx|
 密碼   | xxx|


 您可選擇使用任何已安裝 [Docker 引擎](https://docs.docker.com/engine/)的虛擬機器。

* 配備 Hortonworks Data Platform 3.6 版和 Spark 2.1.x 版的 HDInsight Spark 叢集，最好位於資料所在的美國東部區域。 如需如何建立 HDInsight 叢集的詳細資訊，請前往[在 Azure HDInsight 中建立 Apache Spark 叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters)。 我們建議使用具有三個背景工作的叢集，而背景工作各有 16 個核心和 112 GB 的記憶體。 或者，您可只選擇 VM 類型 `D12 V2` 作為前端節點，選擇 `D14 V2` 作為背景工作節點。 叢集的部署大約需要 20 分鐘。 您需要叢集名稱、SSH 使用者名稱和密碼，才能試用此範例。 儲存含有 Azure HDInsight 叢集資訊的下列表格，以供後續步驟使用：

 欄位名稱| 值 |  
 |------------|------|
 叢集名稱| xxx|
 使用者名稱  | xxx (預設是 sshuser)|
 密碼   | xxx|


* Azure 儲存體帳戶。 您可遵循[這些指示](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account)建立帳戶。 此外，在此儲存體帳戶中建立兩個名為 `fullmodel` 和 `onemonthmodel` 的私人 Blob 容器。 此儲存體帳戶用來儲存中繼計算結果和機器學習模型。 您需要儲存體帳戶名稱和存取金鑰，才能試用此範例。 儲存含有 Azure 儲存體帳戶資訊的下列表格，以供後續步驟使用：

 欄位名稱| 值 |  
 |------------|------|
 儲存體帳戶名稱| xxx|
 存取金鑰  | xxx|


在先決條件清單中建立的 Ubuntu DSVM 和 Azure HDInsight 叢集是計算目標。 計算目標是 Machine Learning Workbench 中的計算資源，可能與 Workbench 執行所在的電腦不同。   

## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Machine Learning Workbench。
2.  在 [專案] 頁面上，選取 **+** 符號，然後選取 [新增專案]。
3.  在 [建立新專案] 窗格中，填入新專案的資訊。
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「大量資料的工作負載預測」，然後選取範本。
5.  選取 [建立] 。

您可以依照[這裡的指示](./tutorial-classifying-iris-part-1.md)，使用預先建立的 git 存放庫建立 Workbench 專案。  
執行 `git status` 來檢查檔案的狀態，以進行版本追蹤。

## <a name="data-description"></a>資料說明

此範例中使用的資料是綜合伺服器的工作負載資料， 並裝載於可在美國東部區域公開存取的 Azure Blob 儲存體帳戶中。 儲存體帳戶的具體資訊位於 [`Config/storageconfig.json`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Config/fulldata_storageconfig.json) 的 `dataFile` 欄位，格式為 "wasb://<BlobStorageContainerName>@<StorageAccountName>.blob.core.windows.net/<path>"。 您可以直接使用 Blob 儲存體中的資料。 若有多位使用者同時使用此儲存體，您可以使用 [azcopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-linux) 將資料下載到自己的儲存體中，以獲得更好的實驗體驗。 

資料大小總計約為 1 TB。 每個檔案大約 1-3 GB，採用 CSV 檔案格式，且沒有標頭。 每個資料列都代表特定伺服器上的交易負載。 資料結構描述的詳細資訊如下所示：

資料行編號 | 欄位名稱| 類型 | 說明 |  
|------------|------|-------------|---------------|
1  | `SessionStart` | DateTime |    工作階段開始時間
2  |`SessionEnd`    | DateTime | 工作階段結束時間
3 |`ConcurrentConnectionCounts` | 整數  | 並行連線數目
4 | `MbytesTransferred` | 兩倍 | 已傳輸的標準化資料 (以 MB 為單位)
5 | `ServiceGrade` | 整數  |  工作階段的服務等級
6 | `HTTP1` | 整數 |  工作階段使用 HTTP1 或 HTTP2
7 |`ServerType` | 整數    |伺服器類型
8 |`SubService_1_Load` | 兩倍 |   子服務 1 負載
9 | `SubService_2_Load` | 兩倍 |  子服務 2 負載
10 | `SubService_3_Load` | 兩倍 |     子服務 3 負載
11 |`SubService_4_Load` | 兩倍 |  子服務 4 負載
12 | `SubService_5_Load`| 兩倍 |      子服務 5 負載
13 |`SecureBytes_Load`  | 兩倍 | 安全位元組負載
14 |`TotalLoad` | 兩倍 | 伺服器上的總負載
15 |`ClientIP` | 字串|    用戶端 IP 位址
16 |`ServerIP` | 字串|    伺服器 IP 位址



請注意，預期的資料類型會列在上表中。 由於遺漏值和不良資料的問題，所以不保證資料類型如同預期。 資料處理應將這點納入考量。 


## <a name="scenario-structure"></a>案例結構

此範例中的檔案整理如下。

| 檔案名稱 | 類型 | 說明 |
|-----------|------|-------------|
| `Code` | 資料夾 | 此資料夾包含範例中的所有程式碼。 |
| `Config` | 資料夾 | 此資料夾包含組態檔。 |
| `Image` | 資料夾 | 此資料夾用於儲存讀我檔案的影像。 |
| `Model` | 資料夾 | 此資料夾用於儲存從 Blob 儲存體下載的模型檔案。 |
| `Code/etl.py` | Python 檔案 | 此 Python 檔案用於資料準備和特徵設計。 |
| `Code/train.py` | Python 檔案 | 此 Python 檔案用於訓練三個類別的多類別模型。  |
| `Code/webservice.py` | Python 檔案 | 此 Python 檔案用於運算化。  |
| `Code/scoring_webservice.py` | Python 檔案 |  此 Python 檔案用於轉換資料和呼叫 Web 服務。 |
| `Code/O16Npreprocessing.py` | Python 檔案 | 此 Python 檔案用於 scoring_webservice.py 的資料前置處理。  |
| `Code/util.py` | Python 檔案 | 此 Python 檔案包含可供讀取和寫入 Azure Blob 的程式碼。  
| `Config/storageconfig.json` | JSON 檔案 | Azure Blob 容器的組態檔，可儲存中繼結果和模型，以便處理及訓練一個月的資料。 |
| `Config/fulldata_storageconfig.json` | Json 檔案 | Azure blob 容器的組態檔，可儲存中繼結果和模型，以便處理及訓練整個資料集。|
| `Config/webservice.json` | JSON 檔案 | scoring_webservice.py 的組態檔。|
| `Config/conda_dependencies.yml` | YAML 檔案 | Conda 相依性檔案。 |
| `Config/conda_dependencies_webservice.yml` | YAML 檔案 | Web 服務的 Conda 相依性檔案。|
| `Config/dsvm_spark_dependencies.yml` | YAML 檔案 | Ubuntu DSVM 的 Spark 相依性檔案。 |
| `Config/hdi_spark_dependencies.yml` | YAML 檔案 | HDInsight Spark 叢集的 Spark 相依性檔案。 |
| `README.md` | Markdown 檔案 | markdown 讀我檔案。 |
| `Code/download_model.py` | Python 檔案 | 此 Python 檔案用於將模型檔案從 Azure Blob 下載到本機磁碟。 |
| `Docs/DownloadModelsFromBlob.md` | Markdown 檔案 | markdown 檔案，其中包含如何執行 `Code/download_model.py` 的指示。 |



### <a name="data-flow"></a>資料流

[`Code/etl.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/etl.py) 中的程式碼會從可公開存取容器 ([`Config/storageconfig.json`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Config/fulldata_storageconfig.json) 的 `dataFile` 欄位) 載入資料。 它包含資料準備和特徵設計，並將中繼計算結果和模型儲存到您自己的私人容器。 [`Code/train.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/train.py) 中的程式碼會從私人容器載入中繼計算結果、訓練多類別分類模型，且將定型的機器學習模型寫入私人容器。 

您應使用一個容器進行一個月資料集的實驗，然後使用另一個容器進行完整資料集的實驗。 因為資料和模型都會儲存為 Parquet 檔案，每個檔案實際上都是容器中的資料夾，其中包含多個 blob。 所產生的容器如下所示：

| blob 前置詞名稱 | 類型 | 說明 |
|-----------|------|-------------|
| featureScaleModel | Parquet | 數值特徵的標準 scaler 模型。 |
| stringIndexModel | Parquet | 非數值特徵的字串索引子模型。|
| oneHotEncoderModel|Parquet | 分類特徵的獨熱 (One-hot) 編碼器模型。 |
| mlModel | Parquet | 定型的機器學習服務模型。 |
| info| Python pickle 檔案 | 已轉換資料的相關資訊，包括可用於訓練測試分割的訓練開始、訓練結束、持續時間和時間戳記，以及可用於編製索引和獨熱 (One-hot) 編碼的資料行。

上表中的所有檔案和 blob 均會用於運算化。


### <a name="model-development"></a>模型開發

#### <a name="architecture-diagram"></a>架構圖表


下圖顯示使用 Machine Learning Workbench 來開發模型的端對端工作流程：![架構](media/scenario-big-data/architecture.PNG)

接下來，我們會在 Machine Learning Workbench 中使用遠端計算目標功能來示範模型開發。 我們會先載入少量樣本資料並在 Ubuntu DSVM 上執行指令碼 [`Code/etl.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/etl.py)，以便快速反覆運算。 我們可藉由傳入額外引數來進一步限制我們在 [`Code/etl.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/etl.py) 中執行的工作，以便更快反覆運算。 最後，我們會使用 HDInsight 叢集以完整資料進行訓練。     

[`Code/etl.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/etl.py) 檔案載入並準備資料，然後執行特徵設計。 它接受兩個引數：
* 一個是 Blob 儲存體容器的組態檔，用來儲存中繼計算結果和模型。
* 一個是偵錯組態，以便更快反覆運算。

第一個引數 `configFilename` 是一個本機組態檔，您在其中儲存 blob 儲存體資訊並指定載入資料的位置。 它會預設為 [`Config/storageconfig.json`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Config/storageconfig.json)，且即將用於一個月資料執行。 我們也加入您需要在完整資料集上使用的 [`Config/fulldata_storageconfig.json`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Config/fulldatastorageconfig.json)。 組態中的內容如下所示： 

| 欄位 | 類型 | 說明 |
|-----------|------|-------------|
| storageAccount | 字串 | Azure 儲存體帳戶名稱 |
| storageContainer | 字串 | Azure 儲存體帳戶中用來儲存中繼結果的容器 |
| storageKey | 字串 |Azure 儲存體帳戶存取金鑰 |
| dataFile|字串 | 資料來源檔案  |
| duration| 字串 | 資料來源檔案中資料的持續時間|

修改 `Config/storageconfig.json` 和 `Config/fulldata_storageconfig.json`來設定儲存體帳戶、儲存體金鑰和 blob 容器，以儲存中繼結果。 根據預設，一個月資料執行的 blob 容器為 `onemonthmodel`，而完整資料集執行的 blob 容器為 `fullmodel`。 請確定您在儲存體帳戶中建立這兩個容器。 [`Config/fulldata_storageconfig.json`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Config/fulldatastorageconfig.json) 中的 `dataFile` 欄位設定哪些資料載入 [`Code/etl.py`](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Code/etl.py) 中。 `duration` 欄位設定資料包含的範圍。 如果持續時間設為 ONE_MONTH，則載入的資料應該只是 2016 年 6 月資料的七個檔案中的一個 csv 檔案。 如果持續時間為 FULL，則會載入完整資料集 (1 TB)。 您不需要變更這兩個組態檔中的 `dataFile` 和 `duration`。

第二個引數是 DEBUG。 將它設定為 FILTER_IP 可啟用更快速的反覆運算。 當您要對指令碼進行偵錯時，使用此參數很有用。

> [!NOTE]
> 在下列所有命令中，將所有引數變數取代為實際值。
> 


#### <a name="model-development-on-the-docker-of-ubuntu-dsvm"></a>Docker of Ubuntu DSVM 上的模型開發

#####  <a name="1-set-up-the-compute-target"></a>1.設定計算目標

選取 [檔案]  >  [開啟命令提示字元]，從 Machine Learning Workbench 啟動命令列。 然後，執行： 

```az ml computetarget attach remotedocker --name dockerdsvm --address $DSVMIPaddress  --username $user --password $password ```

在專案的 aml_config 資料夾中會建立下列兩個檔案：

-  dockerdsvm.compute：這個檔案包含遠端執行目標的連線和設定資訊。
-  dockerdsvm.runconfig：這個檔案是 Workbench 應用程式中使用的一組執行選項。

瀏覽至 dockerdsvm.runconfig，變更下列欄位的設定如下：

    PrepareEnvironment: true 
    CondaDependenciesFile: Config/conda_dependencies.yml 
    SparkDependenciesFile: Config/dsvm_spark_dependencies.yml

執行下列命令來準備專案環境：

```az ml experiment prepare -c dockerdsvm```


將 `PrepareEnvironment` 設為 true，Machine Learning Workbench 會在您每次提交作業時建立執行階段環境。 `Config/conda_dependencies.yml` 和 `Config/dsvm_spark_dependencies.yml` 包含執行階段環境的自訂。 藉由編輯這兩個 YMAL 檔案，您一律可以修改 Conda 相依性、Spark 組態和 Spark 相依性。 在此範例中，我們新增了 `azure-storage` 和 `azure-ml-api-sdk` 作為 `Config/conda_dependencies.yml` 中額外的 Python 套件。 我們也在 `Config/dsvm_spark_dependencies.yml` 中新增 `spark.default.parallelism`、`spark.executor.instances`、`spark.executor.cores`。 

#####  <a name="2-data-preparation-and-feature-engineering-on-dsvm-docker"></a>2.DSVM Docker 上的資料準備和特徵設計

在 DSVM Docker 上執行指令碼 `etl.py`。 使用偵錯參數篩選具有特定伺服器 IP 位址的已載入資料：

```az ml experiment submit -t dockerdsvm -c dockerdsvm ./Code/etl.py ./Config/storageconfig.json FILTER_IP```

瀏覽至側邊欄，選取 [執行] 以查看 `etl.py` 的執行歷程記錄。 請注意，執行時間大約兩分鐘。 如果您打算變更程式碼來包含新特徵，請提供 FILTER_IP 作為第二個引數，進而提供更快的反覆運算。 在處理自己的機器學習問題時，您可能需要執行這個步驟數次，才能瀏覽資料集或建立新特徵。 

自訂限制要載入哪些資料，且進一步篩選要處理哪些資料，您可加快模型開發中的反覆運算程序。 進行實驗時，您應該將程式碼中的變更定期儲存到 Git 存放庫。 請注意，我們在 `etl.py` 中使用下列程式碼來啟用私人容器的存取：

```python
def attach_storage_container(spark, account, key):
    config = spark._sc._jsc.hadoopConfiguration()
    setting = "fs.azure.account.key." + account + ".blob.core.windows.net"
    if not config.get(setting):
        config.set(setting, key)

# attach the blob storage to the spark cluster or VM so that the storage can be accessed by the cluster or VM        
attach_storage_container(spark, storageAccount, storageKey)
```


接下來，在 DSVM Docker 上不用偵錯參數 FILTER_IP 執行 `etl.py` 指令碼：

```az ml experiment submit -t dockerdsvm -c dockerdsvm ./Code/etl.py ./Config/storageconfig.json FALSE```

瀏覽至側邊欄，選取 [執行] 以查看 `etl.py` 的執行歷程記錄。 請注意，執行時間大約四分鐘。 此步驟處理過的結果會儲存至容器，並且載入以供在 train.py 中進行定型。 此外，字串索引子、編碼器管線和標準 scaler 也會儲存到私人容器， 這些將用於運算化。 


##### <a name="3-model-training-on-dsvm-docker"></a>3.DSVM Docker 上的模型訓練

在 DSVM Docker 上執行指令碼 `train.py`：

```az ml experiment submit -t dockerdsvm -c dockerdsvm ./Code/train.py ./Config/storageconfig.json```

此步驟會從 `etl.py` 的執行時載入中繼計算結果，並定型機器學習模型。 這個步驟大約需要兩分鐘。

當您順利完成小型資料實驗，即可繼續對完整資料集執行實驗。 您可以從使用相同的程式碼開始，然後變更引數和計算目標進行實驗。  

####  <a name="model-development-on-the-hdinsight-cluster"></a>HDInsight 叢集上的模型開發

##### <a name="1-create-the-compute-target-in-machine-learning-workbench-for-the-hdinsight-cluster"></a>1.在 Machine Learning Workbench 中建立 HDInsight 叢集的計算目標

```az ml computetarget attach cluster --name myhdi --address $clustername-ssh.azurehdinsight.net --username $username --password $password```

在 aml_config 資料夾中會建立下列兩個檔案：
    
-  myhdi.compute：這個檔案包含遠端執行目標的連線和設定資訊。
-  myhdi.runconfig：這個檔案是 Workbench 應用程式中使用的一組執行選項。


瀏覽至 myhdi.runconfig，變更下列欄位的設定如下：

    PrepareEnvironment: true 
    CondaDependenciesFile: Config/conda_dependencies.yml 
    SparkDependenciesFile: Config/hdi_spark_dependencies.yml

執行下列命令來準備專案環境：

```az ml experiment prepare -c myhdi```

此步驟可能需要多達 7 分鐘的時間。

##### <a name="2-data-preparation-and-feature-engineering-on-hdinsight-cluster"></a>2.HDInsight 叢集上的資料準備和特徵設計

以 HDInsight 叢集上的完整資料執行 `etl.py` 指令碼：

```az ml experiment submit -a -t myhdi -c myhdi ./Code/etl.py Config/fulldata_storageconfig.json FALSE```

因為此作業會持續相當長的時間 (約兩小時)，您可以使用 `-a` 來停用輸出資料流。 作業完成後，您可以在 [執行歷程記錄] 中檢視驅動程式和控制器記錄。 如果您的叢集較大，一律可以在 `Config/hdi_spark_dependencies.yml` 中重新設定組態，以使用多個執行個體或更多核心。 例如，如果您的叢集有四個背景工作節點，您可以將 `spark.executor.instances` 的值從 5 增加至 7。 您可以在您儲存體帳戶的 **fullmodel** 容器中查看此步驟的輸出。 


##### <a name="3-model-training-on-hdinsight-cluster"></a>3.HDInsight 叢集上的模型訓練

在 HDInsight 叢集上執行指令碼 `train.py`：

```az ml experiment submit -a -t myhdi -c myhdi ./Code/train.py Config/fulldata_storageconfig.json```

因為此作業會持續相當長的時間 (約 30 分鐘)，您可以使用 `-a` 來停用輸出資料流。

#### <a name="run-history-exploration"></a>執行歷程記錄探勘

執行歷程記錄是一項功能，可讓您追蹤 Machine Learning Workbench 中的實驗。 根據預設，它會追蹤實驗的持續時間。 在我們的這個範例中，當我們移至 `Code/etl.py` 的完整資料集進行實驗時，發現持續時間大幅增加。 您也可以記錄特定計量以便追蹤。 若要啟用計量追蹤功能，將下列幾行程式碼新增至 Python 檔案的標頭：
```python
# import logger
from azureml.logging import get_azureml_logger

# initialize logger
run_logger = get_azureml_logger()
```
以下是追蹤特定計量的範例：

```python
run_logger.log("Test Accuracy", testAccuracy)
```

在 Workbench 的右側邊欄中，瀏覽至 [執行] 以查看每個 Python 檔案的執行歷程記錄。 您也可以移至您的 GitHub 存放庫。 其中已建立名稱以 "AMLHistory" 開頭的新分支，以利您追蹤在每次執行中對指令碼所做的變更。 


### <a name="operationalize-the-model"></a>模型運算化

在本節中，您會讓在先前步驟中建立為 Web 服務的模型付諸實行 (稱之為「運算化」)。 您也會了解如何使用 Web 服務來預測工作負載。 使用機器語言運算化的命令列介面 (CLI) 將程式碼和相依性封裝為 Docker 映像，並將模型發佈為容器化的 Web 服務。

您可以使用 Machine Learning Workbench 中的命令列提示字元執行 CLI。  也可以遵循[安裝指南](./deployment-setup-configuration.md#using-the-cli)在 Ubuntu Linux 上執行 CLI。 

> [!NOTE]
> 在下列所有命令中，將所有引數變數取代為實際值。 完成這個部份大約需要 40 分鐘。
> 

選擇唯一字串作為運算化的環境。 在這裡，我們使用 "[unique]" 字串來表示您選擇的字串。

1. 建立運算化環境，並建立資源群組。

        az ml env setup -c -n [unique] --location eastus2 --cluster -z 5 --yes

   請注意，您可以使用 Container Service 作為環境，方法是在 `az ml env setup` 命令中使用 `--cluster`。 您可以將機器學習服務模型在 [Azure Container Service](https://docs.microsoft.com/azure/container-service/kubernetes/container-service-intro-kubernetes) 上運算化。 它使用 [Kubernetes](https://kubernetes.io/) 來進行自動化部署、調整和管理容器化的應用程式。 執行這個命令大約需要 20 分鐘。 使用下列命令檢查部署是否順利完成： 

        az ml env show -g [unique]rg -n [unique]

   執行下列命令將部署環境設定為您剛建立的環境：

        az ml env set -g [unique]rg -n [unique]

2. 建立並使用模型管理帳戶。 若要建立模型管理帳戶，執行下列命令：

        az ml account modelmanagement create --location  eastus2 -n [unique]acc -g [unique]rg --sku-instances 4 --sku-name S3 

   使用模型管理進行運算化，方法是執行：

        az ml account modelmanagement set  -n [unique]acc -g [unique]rg  

   模型管理帳戶是用於管理模型和 Web 服務。 在 Azure 入口網站中，您可以看到剛才建立的新模型管理帳戶。 您可以使用它來查看模型、資訊清單、Docker 映像，以及使用此模型管理帳戶所建立的服務。

3. 下載並註冊模型。

   將 **fullmodel** 容器中的模型下載到程式碼目錄中的本機電腦。 不要使用 "vmlSource.parquet" 名稱下載 parquet 資料檔案。 它不是模型檔案，它是中繼運算結果。 您也可以重複使用 Git 存放庫中的模型檔案。 如需詳細資訊，請參閱 [GitHub](https://github.com/Azure/MachineLearningSamples-BigData/blob/master/Docs/DownloadModelsFromBlob.md)。 

   在 CLI 中移至 `Model` 資料夾並註冊模型，如下：

        az ml model register -m  mlModel -n vmlModel -t fullmodel
        az ml model register -m  featureScaleModel -n featureScaleModel -t fullmodel
        az ml model register -m  oneHotEncoderModel -n  oneHotEncoderModel -t fullmodel
        az ml model register -m  stringIndexModel -n stringIndexModel -t fullmodel
        az ml model register -m  info -n info -t fullmodel

   每個命令的輸出都會提供下一個步驟中所需的模型識別碼。 請將它們儲存在文字檔中，以供日後使用。

4. 建立 Web 服務的資訊清單。

   資訊清單包含 Web 服務的核心、所有機器學習模型和執行階段環境相依性的程式碼。 在 CLI 中移至 `Code` 資料夾，然後執行下列命令：

        az ml manifest create -n $webserviceName -f webservice.py -r spark-py -c ../Config/conda_dependencies_webservice.yml -i $modelID1 -i $modelID2 -i $modelID3 -i $modelID4 -i $modelID5

   輸出會提供下一個步驟的資訊清單識別碼。 

   停留在 `Code` 目錄中，您可以執行下列命令來測試 webservice.py： 

        az ml experiment submit -t dockerdsvm -c dockerdsvm webservice.py

5. 建立 Docker 映像。 

        az ml image create -n [unique]image --manifest-id $manifestID

   輸出會提供下一個步驟的映像識別碼。此 Docker 映像會用於 Container Service。 

6. 將 Web 服務部署至 Container Service 叢集。

        az ml service create realtime -n [unique] --image-id $imageID --cpu 0.5 --memory 2G

   輸出會提供服務識別碼。 您需要使用它來取得授權金鑰和服務 URL。

7. 在 Python 程式碼中呼叫 Web 服務，以迷你批次進行評分。

   使用下列命令來取得授權金鑰：

         az ml service keys realtime -i $ServiceID 

   使用下列命令來取得服務評分 URL：

        az ml service usage realtime -i $ServiceID

   以正確的服務評分 URL 和授權金鑰修改 `./Config/webservice.json` 中的內容。 保留原始檔案中的 "Bearer" 並取代 "xxx" 的部份。 
   
   移至專案的根目錄，並測試 Web 服務以進行迷你批次評分，方法是使用下列命令：

        az ml experiment submit -t dockerdsvm -c dockerdsvm ./Code/scoring_webservice.py ./Config/webservice.json

8. 調整 Web 服務。 

   如需詳細資訊，請參閱[如何調整 Azure Container Service 叢集上的運算化](how-to-scale-clusters.md)。
 

## <a name="next-steps"></a>後續步驟

此範例強調如何使用 Machine Learning Workbench 來訓練巨量資料的機器學習模型，並將定型的模型運算化。 特別是，您已學會如何設定和使用不同的計算目標，以及執行追蹤計量的歷程記錄，並使用不同的執行。

您可以擴充程式碼以瀏覽交叉驗證和超參數調整。 若要深入了解交叉驗證和超參數調整，請參閱[此 GitHub 資源](https://github.com/Azure/MachineLearningSamples-DistributedHyperParameterTuning)。  

若要深入了解時間序列預測，請參閱[此 GitHub 資源](https://github.com/Azure/MachineLearningSamples-EnergyDemandTimeSeriesForecasting)。