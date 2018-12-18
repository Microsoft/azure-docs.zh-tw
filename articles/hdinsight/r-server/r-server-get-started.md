---
title: 開始使用 HDInsight 上的 ML 服務 - Azure
description: 了解如何在包含 ML 服務的 HDInsight 叢集上建立 Apache Spark，並在叢集上提交 R 指令碼。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 06/27/2018
ms.openlocfilehash: 7b3d2d47db733d1290bccca0e44958098451324e
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046407"
---
# <a name="get-started-with-ml-services-on-azure-hdinsight"></a>開始使用 Azure HDInsight 上的 ML 服務

Azure HDInsight 可讓您建立 ML 服務叢集。 此選項可讓 R 指令碼使用 Spark 和 MapReduce 來執行分散式計算。 在本文中，您將了解如何在 HDInsight 上建立 ML 服務叢集，以及如何執行 R 指令碼以示範使用 Spark 進行分散式 R 計算的方式。

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**：開始進行本教學課程之前，您必須擁有 Azure 訂用帳戶。 如需詳細資訊，請參閱[取得 Microsoft Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **安全殼層 (SSH) 用戶端**：SSH 用戶端可用來從遠端連線至 HDInsight 叢集，並直接在叢集上執行命令。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。


<a name="create-hdi-custer-with-aure-portal"></a>
## <a name="create-the-cluster-using-the-azure-portal"></a>使用 Azure 入口網站建立叢集

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 按一下 [建立資源] > [資料 + 分析] > [HDInsight]。

3. 從 [基本概念]，輸入以下資訊：

    * **叢集名稱**︰HDInsight 叢集的名稱。
    * **訂用帳戶**：選取要使用的訂用帳戶。
    * **叢集登入使用者名稱**和**叢集登入密碼**：透過 HTTPS 存取叢集時使用的登入資訊。 您會使用這些認證來存取例如 Ambari Web UI 或 REST API 等服務。
    * **安全殼層 (SSH) 使用者名稱**：透過 SSH 存取叢集時使用的登入資訊。 依預設，密碼要與叢集登入密碼相同。
    * **資源群組**：在其中建立叢集的資源群組。
    * **位置**：在其中建立叢集的 Azure 區域。

        ![叢集基本詳細資料](./media/r-server-get-started/clustername.png)

4. 選取 [叢集類型]，並且在 [叢集組態] 區段中設定下列值︰

    * **叢集類型**：ML 服務

    * **作業系統**：Linux

    * **版本**：ML Server 9.3 (HDI 3.6)。 ML Server 9.3 的版本資訊可在 [docs.microsoft.com](https://docs.microsoft.com/machine-learning-server/whats-new-in-machine-learning-server) 上取得。

    * **ML 伺服器的 R Studio Community 版本**︰邊緣節點上依預設會安裝以瀏覽器為基礎的 IDE。 如果您不想加以安裝，請清除此核取方塊。 如果您選擇安裝它，則在建立後，可在叢集的入口網站應用程式刀鋒視窗上，取得用來存取 RStudio Server 登入的 URL。

        ![叢集基本詳細資料](./media/r-server-get-started/clustertypeconfig.png)

4. 選取叢集類型之後，請使用 [選取] 按鈕來設定叢集類型。 接下來，使用 [下一步] 按鈕來完成基本組態。

5. 從 [儲存體] 區段中，選取或建立儲存體帳戶。 本文件的步驟是，將此區段中的其他欄位保留為預設值。 使用 [下一步] 按鈕以儲存儲存體組態。

    ![設定 HDInsight 的儲存體帳戶](./media/r-server-get-started/cluster-storage.png)

6. 從 [摘要] 區段中，檢閱叢集組態。 使用 [編輯] 連結來變更所有不正確的設定。 最後，使用 [建立] 按鈕建立叢集。

    ![設定 HDInsight 的儲存體帳戶](./media/r-server-get-started/clustersummary.png)

    > [!NOTE]
    > 建立叢集可能需要花費 20 分鐘的時間。

<a name="connect-to-rstudio-server"></a>
## <a name="connect-to-rstudio-server"></a>連線到 RStudio 伺服器

如果您選擇安裝 RStudio Server Community Edition 作為您 HDInsight 叢集的一部分，您就可以使用下列兩種方法的其中一個來存取 RStudio 登入：

* **選項 1** - 移至下列 URL (其中，**CLUSTERNAME** 是您建立的 ML 服務叢集的名稱)：

        https://CLUSTERNAME.azurehdinsight.net/rstudio/

* **選項 2** - 在 Azure 入口網站中開啟 ML 服務叢集，並在 [快速連結] 下按一下 [ML 服務儀表板]。

     ![設定 HDInsight 的儲存體帳戶](./media/r-server-get-started/dashboard-quick-links.png)

    從 [叢集儀表板] 中，按一下 [R Studio Server]。

    ![設定 HDInsight 的儲存體帳戶](./media/r-server-get-started/r-studio-server-dashboard.png)

   > [!IMPORTANT]
   > 不論使用哪一種方法，第一次登入時都必須驗證兩次。  第一次出現驗證提示時，請提供「叢集管理員的使用者識別碼」和「密碼」。 第二次出現驗證提示時，請提供「SSH 使用者識別碼」和「密碼」。 之後再登入時，只需要提供 SSH 認證。

一旦連線後，您的畫面看起來應該像下列螢幕擷取畫面：

![連線到 RStudio](./media/r-server-get-started/connect-to-r-studio.png)

## <a name="run-a-sample-job"></a>執行範例作業

您可以使用 ScaleR 函式來提交作業。 以下是用來執行作業的命令範例：

    # Set the HDFS (WASB) location of example data.
    bigDataDirRoot <- "/example/data"

    # Create a local folder for storaging data temporarily.
    source <- "/tmp/AirOnTimeCSV2012"
    dir.create(source)

    # Download data to the tmp folder.
    remoteDir <- "https://packages.revolutionanalytics.com/datasets/AirOnTimeCSV2012"
    download.file(file.path(remoteDir, "airOT201201.csv"), file.path(source, "airOT201201.csv"))
    download.file(file.path(remoteDir, "airOT201202.csv"), file.path(source, "airOT201202.csv"))
    download.file(file.path(remoteDir, "airOT201203.csv"), file.path(source, "airOT201203.csv"))
    download.file(file.path(remoteDir, "airOT201204.csv"), file.path(source, "airOT201204.csv"))
    download.file(file.path(remoteDir, "airOT201205.csv"), file.path(source, "airOT201205.csv"))
    download.file(file.path(remoteDir, "airOT201206.csv"), file.path(source, "airOT201206.csv"))
    download.file(file.path(remoteDir, "airOT201207.csv"), file.path(source, "airOT201207.csv"))
    download.file(file.path(remoteDir, "airOT201208.csv"), file.path(source, "airOT201208.csv"))
    download.file(file.path(remoteDir, "airOT201209.csv"), file.path(source, "airOT201209.csv"))
    download.file(file.path(remoteDir, "airOT201210.csv"), file.path(source, "airOT201210.csv"))
    download.file(file.path(remoteDir, "airOT201211.csv"), file.path(source, "airOT201211.csv"))
    download.file(file.path(remoteDir, "airOT201212.csv"), file.path(source, "airOT201212.csv"))

    # Set directory in bigDataDirRoot to load the data.
    inputDir <- file.path(bigDataDirRoot,"AirOnTimeCSV2012")

    # Create the directory.
    rxHadoopMakeDir(inputDir)

    # Copy the data from source to input.
    rxHadoopCopyFromLocal(source, bigDataDirRoot)

    # Define the HDFS (WASB) file system.
    hdfsFS <- RxHdfsFileSystem()

    # Create info list for the airline data.
    airlineColInfo <- list(
    DAY_OF_WEEK = list(type = "factor"),
    ORIGIN = list(type = "factor"),
    DEST = list(type = "factor"),
    DEP_TIME = list(type = "integer"),
    ARR_DEL15 = list(type = "logical"))

    # Get all the column names.
    varNames <- names(airlineColInfo)

    # Define the text data source in HDFS.
    airOnTimeData <- RxTextData(inputDir, colInfo = airlineColInfo, varsToKeep = varNames, fileSystem = hdfsFS)

    # Define the text data source in local system.
    airOnTimeDataLocal <- RxTextData(source, colInfo = airlineColInfo, varsToKeep = varNames)

    # Specify the formula to use.
    formula = "ARR_DEL15 ~ ORIGIN + DAY_OF_WEEK + DEP_TIME + DEST"

    # Define the Spark compute context.
    mySparkCluster <- RxSpark()

    # Set the compute context.
    rxSetComputeContext(mySparkCluster)

    # Run a logistic regression.
    system.time(
        modelSpark <- rxLogit(formula, data = airOnTimeData)
    )

    # Display a summary.
    summary(modelSpark)

<a name="connect-to-edge-node"></a>
## <a name="connect-to-the-cluster-edge-node"></a>連線到叢集邊緣節點

在本節中，您將了解如何使用 SSH 連線至 ML 服務 HDInsight 叢集的邊緣節點。 如需了解如何使用 SSH，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

連線至 ML 服務叢集邊緣節點的 SSH 命令為：

   `ssh USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net`

若要尋找您叢集的 SSH 命令，請從 Azure 入口網站中按一下叢集名稱，並按一下 [SSH + 叢集登入]，然後針對 [主機名稱]，選取邊緣節點。 如此即可顯示邊緣節點的 SSH 端點資訊。

![邊緣節點 SSH 端點的影像](./media/r-server-get-started/sshendpoint.png)

如果您已經使用密碼保護您 SSH 使用者帳戶的安全，系統會提示您輸入密碼。 如果您使用的是公開金鑰，您可能必須使用 `-i` 參數來指定對應的私密金鑰。 例如︰

    ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net

連線之後，您將看見類似以下的提示：

    sshuser@ed00-myrclu:~$

<a name="use-r-console"></a>
## <a name="use-the-r-console"></a>使用 R 主控台

1. 在 SSH 工作階段中，使用以下命令啟動 R 主控台：  

        R

2. 您應該會看到 ML Server 的版本輸出，以及其他資訊。
    
3. 您可以透過 `>` 提示，輸入 R 程式碼。 HDInsight 上的 ML 服務包含可讓您輕鬆與 Hadoop 互動並執行分散式計算的套件。 例如，若要檢視 HDInsight 叢集的預設檔案系統根目錄，可使用下列命令：

        rxHadoopListFiles("/")

4. 您也可以使用 WASB 樣式定址。

        rxHadoopListFiles("wasb:///")

5. 若要結束 R 主控台，請使用下列命令：

        quit()

## <a name="automated-cluster-creation"></a>自動化的叢集建立

您可以使用 SDK 和 PowerShell 自動建立 HDInsight 的 ML 服務叢集。

<!---* To create an ML Server cluster using an Azure Resource Management template, see [Deploy an R Server for HDInsight cluster](https://azure.microsoft.com/resources/templates/101-hdinsight-rserver/).--->
* 若要使用 .NET SDK 建立 ML 服務叢集，請參閱[在 HDInsight 中使用 .NET SDK 建立以 Linux 為基礎的叢集](../hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)。
* 若要使用 Powershell 建立 ML 服務叢集，請參閱關於[使用 Azure PowerShell 建立 HDInsight 叢集](../hdinsight-hadoop-create-linux-clusters-azure-powershell.md)的文章。

## <a name="delete-the-cluster"></a>刪除叢集

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](../hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何在 Azure HDInsight 中建立新的 ML 服務叢集，以及從 SSH 工作階段使用 R 主控台的基本概念。 下列文章說明，在 HDInsight 上管理和使用 ML 服務的其他方式：

* [從 Visual Studio R 工具提交作業](r-server-submit-jobs-r-tools-vs.md)
* [在 HDInsight 上管理 ML 服務叢集](r-server-hdinsight-manage.md)
* [在 HDInsight 上運作 ML 服務叢集](r-server-operationalize.md)
* [在 HDInsight 上計算 ML 服務叢集的內容選項](r-server-compute-contexts.md)
* [HDInsight 上適用於 ML 服務叢集的 Azure 儲存體選項](r-server-storage.md)
