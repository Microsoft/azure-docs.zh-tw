---
title: 在 HDInsight 上管理 ML 服務叢集 - Azure
description: 瞭解如何在 Azure HDInsight 中管理 ML 服務叢集上的各種工作。
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/19/2019
ms.openlocfilehash: e4c9124ebd0b61b8db1b1da964355a3c36b5bba5
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98930578"
---
# <a name="manage-ml-services-cluster-on-azure-hdinsight"></a>在 HDInsight 上管理 ML 服務叢集

在本文中，您將瞭解如何在 Azure HDInsight 上管理現有的 ML 服務叢集，以執行像是新增多個並行使用者、遠端連線至 ML 服務叢集、變更計算內容等工作。

## <a name="prerequisites"></a>先決條件

* HDInsight 上的 ML 服務叢集。 請參閱[使用 Azure 入口網站建立 Apache Hadoop 叢集](../hdinsight-hadoop-create-linux-clusters-portal.md)，然後選取 [ML 服務] 作為 [叢集類型]。

* 安全殼層 (SSH) 用戶端：SSH 用戶端可用來從遠端連線至 HDInsight 叢集，並直接在叢集上執行命令。 如需詳細資訊，請參閱搭配[HDInsight 使用 SSH。](../hdinsight-hadoop-linux-use-ssh-unix.md)

## <a name="enable-multiple-concurrent-users"></a>啟用多個並行使用者

為 RStudio 社群版本執行所在的邊緣節點新增更多使用者，即可藉此為 HDInsight 上的 ML 服務叢集啟用多個並行使用者。 當您建立 HDInsight 叢集時，您必須提供兩個使用者 (HTTP 使用者和 SSH 使用者)：

![HDI Azure 入口網站登入參數](./media/r-server-hdinsight-manage/hdi-concurrent-users1.png)

- **叢集登入使用者名稱**：透過 HDInsight 閘道 (用來保護您所建立的 HDInsight 叢集) 進行驗證的 HTTP 使用者。 此 HTTP 使用者用於存取 Apache Ambari UI、Apache Hadoop YARN UI，以及其他 UI 元件。
- **安全殼層 (SSH) 使用者名稱**：透過安全殼層存取叢集的 SSH 使用者。 此使用者是在 Linux 系統中適用於所有前端節點、背景工作節點和邊緣節點的使用者。 因此您可以使用安全殼層來存取遠端叢集中的任何節點。

HDInsight 上 ML 服務叢集中所使用的 R Studio Server 社群版本，只接受 Linux 使用者名稱和密碼作為登入機制。 但不支援傳遞權杖。 因此，當您首次嘗試存取 ML 服務叢集上的 R Studio 時，您需要登入兩次。

- 先透過 HDInsight 閘道使用 HTTP 使用者認證登入，

- 然後使用 SSH 使用者認證登入 RStudio。
  
目前，在佈建 HDInsight 叢集時，只可以建立一個 SSH 使用者帳戶。 因此，若要讓多個使用者存取 HDInsight 上的 ML 服務叢集，您必須在 Linux 系統中建立其他使用者。

因為 RStudio 會在叢集的邊緣節點上執行，所以有下列數個步驟：

1. 使用現有的 SSH 使用者登入邊緣節點
2. 在邊緣節點中新增更多 Linux 使用者
3. 使用 RStudio 社群版本搭配所建立的使用者

### <a name="step-1-use-the-created-ssh-user-to-sign-in-to-the-edge-node"></a>步驟 1：使用所建立的 SSH 使用者來登入邊緣節點

遵循[使用 SSH 連線到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md) 中的指示來存取邊緣節點。 HDInsight 上 ML 服務叢集的邊緣節點位址是 `CLUSTERNAME-ed-ssh.azurehdinsight.net`。

### <a name="step-2-add-more-linux-users-in-edge-node"></a>步驟 2：在邊緣節點中新增更多 Linux 使用者

若要將使用者新增至邊緣節點，請執行下列命令︰

```bash
# Add a user 
sudo useradd <yournewusername> -m

# Set password for the new user
sudo passwd <yournewusername>
```

以下螢幕擷取畫面會顯示輸出。

![螢幕擷取畫面輸出並行使用者](./media/r-server-hdinsight-manage/hdi-concurrent-users2.png)

當系統提示您輸入 [目前的 Kerberos 密碼] 時，只要按 **Enter** 加以忽略。 `useradd` 命令中的 `-m` 選項表示系統將為使用者建立主資料夾，這是 RStudio 社群版本所需的。

### <a name="step-3-use-rstudio-community-version-with-the-user-created"></a>步驟 3：使用 RStudio 社群版本搭配建立的使用者

從 `https://CLUSTERNAME.azurehdinsight.net/rstudio/` 存取 RStudio。 如果您在建立叢集之後首次進行登入，請輸入叢集系統管理員認證，後面接著您建立的 SSH 使用者認證。 如果這不是您第一次登入，則只需針對您所建立的 SSH 使用者輸入認證。

您也可以使用原始認證 (預設是 *sshuser*)，從另一個瀏覽器視窗同時登入。

也請注意，新增的使用者在 Linux 系統中沒有根權限，但是他們對遠端 HDFS 與 WASB 儲存體中的所有檔案具有相同的存取權。

## <a name="connect-remotely-to-microsoft-ml-services"></a>遠端連線到 Microsoft ML 服務

您可以設定從桌上型電腦上執行的 ML 用戶端遠端執行個體，來存取 HDInsight Spark 計算內容。 若要這樣做，在桌上型電腦上定義 RxSpark 計算內容時，您必須指定選項 (hdfsShareDir、shareDir、sshUsername、sshHostname、sshSwitches 和 sshProfileScript)：例如：

```r
myNameNode <- "default"
myPort <- 0

mySshHostname  <- '<clustername>-ed-ssh.azurehdinsight.net'  # HDI secure shell hostname
mySshUsername  <- '<sshuser>'# HDI SSH username
mySshSwitches  <- '-i /cygdrive/c/Data/R/davec'   # HDI SSH private key

myhdfsShareDir <- paste("/user/RevoShare", mySshUsername, sep="/")
myShareDir <- paste("/var/RevoShare" , mySshUsername, sep="/")

mySparkCluster <- RxSpark(
    hdfsShareDir = myhdfsShareDir,
    shareDir     = myShareDir,
    sshUsername  = mySshUsername,
    sshHostname  = mySshHostname,
    sshSwitches  = mySshSwitches,
    sshProfileScript = '/etc/profile',
    nameNode     = myNameNode,
    port         = myPort,
    consoleOutput= TRUE
)
```

如需詳細資訊，請參閱 [How to use RevoScaleR in an Apache Spark compute context](/machine-learning-server/r/how-to-revoscaler-spark#more-spark-scenarios) (如何在 Apache Spark 計算內容中使用 RevoScaleR) 中的 "Using Microsoft Machine Learning Server as an Apache Hadoop Client" (使用 Microsoft Machine Learning Server 作為 Apache Hadoop 用戶端) 一節

## <a name="use-a-compute-context"></a>使用計算內容

計算內容可讓您控制要在邊緣節點上本機執行計算，或散發到 HDInsight 叢集的節點中。  如需使用 RStudio 伺服器設定計算內容的範例，請參閱 [使用 RStudio 伺服器在 Azure HDInsight 的 ML 服務叢集上執行 R 腳本](machine-learning-services-quickstart-job-rstudio.md)。

## <a name="distribute-r-code-to-multiple-nodes"></a>將 R 程式碼分散到多個節點

有了 HDInsight 上的 ML 服務，您就可以採用現有的 R 程式碼，並使用 `rxExec` 在跨叢集中的多個節點執行。 執行參數掃掠或模擬時，這個函式非常有用。 以下是使用 `rxExec` 的範例程式碼：

```r
rxExec( function() {Sys.info()["nodename"]}, timesToRun = 4 )
```

如果您仍在使用 Spark 內容，此命令會針對執行程式碼 `(Sys.info()["nodename"])` 的背景工作節點傳回 nodename 值。 例如，在四節點叢集上時，您應會取得如下列程式碼片段的輸出：

```r
$rxElem1
    nodename
"wn3-mymlser"

$rxElem2
    nodename
"wn0-mymlser"

$rxElem3
    nodename
"wn3-mymlser"

$rxElem4
    nodename
"wn3-mymlser"
```

## <a name="access-data-in-apache-hive-and-parquet"></a>存取 Apache Hive 和 Parquet 中的資料

HDInsight ML 服務可讓您在 Hive 和 Parquet 中直接存取資料，以供 ScaleR 函式在 Spark 計算內容使用。 這些功能可透過新的 ScaleR 資料來源函式 (稱為 RxHiveData 和 RxParquetData) 來使用，而透過使用 Spark SQL 將資料直接載入到 Spark 資料框架供 ScaleR 進行分析，即可讓這些函式運作。

下面程式碼提供一些關於新函式使用方式的範例程式碼︰

```r
#Create a Spark compute context:
myHadoopCluster <- rxSparkConnect(reset = TRUE)

#Retrieve some sample data from Hive and run a model:
hiveData <- RxHiveData("select * from hivesampletable",
                       colInfo = list(devicemake = list(type = "factor")))
rxGetInfo(hiveData, getVarInfo = TRUE)

rxLinMod(querydwelltime ~ devicemake, data=hiveData)

#Retrieve some sample data from Parquet and run a model:
rxHadoopMakeDir('/share')
rxHadoopCopyFromLocal(file.path(rxGetOption('sampleDataDir'), 'claimsParquet/'), '/share/')
pqData <- RxParquetData('/share/claimsParquet',
                        colInfo = list(
                            age    = list(type = "factor"),
                            car.age = list(type = "factor"),
                            type = list(type = "factor")
                        ) )
rxGetInfo(pqData, getVarInfo = TRUE)

rxNaiveBayes(type ~ age + cost, data = pqData)

#Check on Spark data objects, cleanup, and close the Spark session:
lsObj <- rxSparkListData() # two data objs are cached
lsObj
rxSparkRemoveData(lsObj)
rxSparkListData() # it should show empty list
rxSparkDisconnect(myHadoopCluster)
```

如需如何使用這些新函式的詳細資訊，請透過 `?RxHivedata` 和 `?RxParquetData` 命令參閱 ML 服務中的線上說明。  

## <a name="install-additional-r-packages-on-the-cluster"></a>在叢集上安裝其他 R 封裝

### <a name="to-install-r-packages-on-the-edge-node"></a>在邊緣節點上安裝 R 封裝

如果您想要在邊緣節點上安裝其他 R 封裝，您可以在透過 SSH 連線至邊緣節點時，直接從 R 主控台內使用 `install.packages()`。 

### <a name="to-install-r-packages-on-the-worker-node"></a>在背景工作節點上安裝 R 封裝

若要在叢集的背景工作節點上安裝 R 封裝，就必須使用指令碼動作。 指令碼動作是一種 Bash 指令碼，可用來變更 HDInsight 叢集的設定或安裝其他軟體，例如其他 R 套件。 

> [!IMPORTANT]  
> 僅有在建立叢集之後，才可以使用指令碼動作來安裝其他 R 封裝。 在叢集建立期間，請勿使用此程序，因為指令碼依賴已完整設定的 ML 服務。

1. 遵循[使用指令碼動作來自訂叢集](../hdinsight-hadoop-customize-cluster-linux.md)中的步驟。

3. 針對 **提交指令碼動作**，提供下列資訊：

   * 針對 **指令碼類型**，選取 [自訂]。

   * 針對 **名稱**，提供指令碼動作的名稱。

     * 針對 **Bash 指令碼 URI**，輸入 `https://mrsactionscripts.blob.core.windows.net/rpackages-v01/InstallRPackages.sh`。 這是在背景工作節點上安裝其他 R 封裝的指令碼

   * 只選取 **背景工作** 的核取方塊。

   * **參數**︰要安裝的 R 套件。 例如， `bitops stringr arules`

   * 選取此核取方塊以 **持續此指令碼動作**。  

   > [!NOTE]
   > 1. 根據預設，系統在安裝所有的 R 封裝時，會透過已安裝 Machine Learning Server 相同版本的 Microsoft MRAN 存放庫快照。 如果您想要安裝較新版的套件，則會有不相容的風險。 不過，將 `useCRAN` 指定為套件清單的第一個元素 (例如 `useCRAN bitops, stringr, arules`)，這種安裝就可行。  
   > 2. 有些 R 套件需要額外的 Linux 系統程式庫。 為了方便起見，HDInsight ML 服務已預先安裝前 100 個最受歡迎的 R 封裝所需的相依性。 然而，如果您安裝的 R 封裝需要的程式庫不在這之中，則必須下載此處所使用的基底指令碼，並加入安裝系統程式庫的步驟。 接下來，您必須將修改過的指令碼上傳至 Azure 儲存體中的公用 Blob 容器，並使用修改過的指令碼來安裝封裝。
   >    如需開發腳本動作的詳細資訊，請參閱 [腳本動作開發](../hdinsight-hadoop-script-actions-linux.md)。

   ![Azure 入口網站提交腳本動作](./media/r-server-hdinsight-manage/submit-script-action.png)

4. 按一下 [建立] 執行指令碼。 指令碼完成之後，即可在所有的背景工作角色節點上使用 R 套件。

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 上運作 ML 服務叢集](r-server-operationalize.md)
* [在 HDInsight 上計算 ML 服務叢集的內容選項](r-server-compute-contexts.md)
* [HDInsight 上適用於 ML 服務叢集的 Azure 儲存體選項](r-server-storage.md)