---
title: 在 Azure HDInsight Linux 叢集上安裝 Presto
description: 了解如何使用指令碼動作，在以 Linux 為基礎的 HDInsight Hadoop 叢集上安裝 Presto 和 Airpal。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: jasonh
ms.openlocfilehash: b9ac9c49e633906e47244eedcb18a4cda4a6228d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978944"
---
# <a name="install-and-use-presto-on-hdinsight-hadoop-clusters"></a>在 HDInsight Hadoop 叢集上安裝和使用 Presto

在本文件中，您會了解如何使用指令碼動作，在 HDInsight Hadoop 叢集上安裝 Presto。 您也會了解如何在現有的 HDInsight 叢集上安裝 Airpal。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 **HDInsight 3.5 Hadoop 叢集**。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 版本](hdinsight-component-versioning.md)。

## <a name="what-is-presto"></a>什麼是 Presto？
[Presto](https://prestodb.io/overview.html) 是巨量資料的快速分散式 SQL 查詢引擎。 Presto 適合數 PB 資料的互動式查詢。 如需 Presto 的元件以及它們共同運作方式的詳細資訊，請參閱 [Presto 概念](https://github.com/prestodb/presto/blob/master/presto-docs/src/main/sphinx/overview/concepts.rst)。

> [!WARNING]
> 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務將協助釐清與解決這些元件的相關問題。
> 
> 自訂元件 (例如 Presto) 會獲得商務上合理的支援，協助您進一步針對問題進行疑難排解。 如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。
> 
> 


## <a name="install-presto-using-script-action"></a>使用指令碼動作來安裝 Presto

本節提供如何在使用 Azure 入口網站建立新叢集時使用範例指令碼的指示。 

1. 使用[佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中的步驟，開始佈建叢集。 請確定您是使用**自訂**叢集建立流程來建立叢集。 叢集必須符合下列需求。

    * 必須為隨附 HDInsight 3.6 版的 Hadoop 叢集。

    * 它必須使用 Azure 儲存體作為資料存放區。 尚未支援在將 Azure Data Lake Store 作為儲存體選項的叢集上使用 Presto。 

    ![使用自訂選項建立 HDInsight 叢集](./media/hdinsight-hadoop-install-presto/hdinsight-install-custom.png)

2. 在 [進階設定] 區域中，選取 [指令碼動作]，並提供下列資訊：
   
   * **名稱**：輸入指令碼動作的易記名稱。
   * **Bash 指令碼 URI**：`https://raw.githubusercontent.com/hdinsight/presto-hdinsight/master/installpresto.sh`
   * **HEAD**：勾選此選項
   * **WORKER**：勾選此選項
   * **ZOOKEEPER**︰清除此核取方塊
   * **參數**：將此欄位保留空白


3. 在 [指令碼動作] 區域的底部，按一下 [選取] 按鈕來儲存設定。 最後，按一下 [進階設定] 區域底部的 [選取] 按鈕來儲存設定資訊。

4. 繼續如 [佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中所述佈建叢集。

    > [!NOTE]
    > Azure PowerShell、Azure 傳統 CLI、HDInsight .NET SDK 或 Azure Resource Manager 範本也可用來套用指令碼動作。 您也可以將指令碼動作套用到執行中的叢集上。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。
    > 
    > 

## <a name="use-presto-with-hdinsight"></a>使用 Presto 搭配 HDInsight

若要在 HDInsight 叢集中搭配使用 Presto，請使用下列步驟：

1. 使用 SSH 連線到 HDInsight 叢集
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
   
    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。
     

2. 使用下列命令啟動 Presto 殼層。
   
        presto --schema default

3. 在所有 HDInsight 叢集中預設可用的範例資料表 (**hivesampletable**) 上執行查詢。
   
        select count (*) from hivesampletable;
   
    依預設，已設定 Presto 的 [Hive](https://prestodb.io/docs/current/connector/hive.html) 和 [TPCH](https://prestodb.io/docs/current/connector/tpch.html) 連接器。 Hive 連接器會設定為使用預設已安裝的 Hive 安裝，以便 Hive 的所有資料表都會自動顯示在 Presto 中。

    如需詳細資訊，請參閱 [Presto 文件](https://prestodb.io/docs/current/index.html)。

## <a name="use-airpal-with-presto"></a>使用 Airpal 搭配 Presto

[Airpal](https://github.com/airbnb/airpal#airpal) 是 Presto 以 web 作為基礎的開放原始碼查詢介面。 如需有關 Airpal 的詳細資訊，請參閱 [Airpal 文件](https://github.com/airbnb/airpal#airpal)。

使用下列步驟在邊緣節點上安裝 Airpal：

1. 使用 SSH 連線到已安裝 Presto 的 HDInsight 叢集前端節點︰
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
   
    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 一旦連線後，請執行下列命令。

        sudo slider registry  --name presto1 --getexp presto 
   
    您會看到類似以下 JSON 的輸出：

        {
            "coordinator_address" : [ {
                "value" : "10.0.0.12:9090",
                "level" : "application",
                "updatedTime" : "Mon Apr 03 20:13:41 UTC 2017"
        } ]

3. 記下輸出中的 **value** 屬性值。 您在叢集 edgenode 上安裝 Airpal 時將會需要這個值。 上述輸出中，您所需要的值是 **10.0.0.12:9090**。

4. 使用**[這裡](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2Fpresto-hdinsight%2Fmaster%2Fairpal-deploy.json)** 的範本來建立 HDInsight 叢集 edgenode 並提供值，如下列螢幕擷取畫面所示。

    ![HDInsight 在 Presto 叢集上安裝 Airpal](./media/hdinsight-hadoop-install-presto/hdinsight-install-airpal.png)

5. 按一下 [購買]。

6. 將變更套用到叢集設定後，您可以使用下列步驟來存取 Airpal web 介面。

    1. 在叢集對話方塊中，按一下 [應用程式]。

        ![HDInsight 在 Presto 叢集上啟動 Airpal](./media/hdinsight-hadoop-install-presto/hdinsight-presto-launch-airpal.png)

    2. 在 [安裝應用程式] 區域中，針對 Airpal 按一下 [入口網站]。

        ![HDInsight 在 Presto 叢集上啟動 Airpal](./media/hdinsight-hadoop-install-presto/hdinsight-presto-launch-airpal-1.png)

    3. 出現提示時，輸入您在建立 HDInsight Hadoop 叢集時指定的系統管理員認證。

## <a name="customize-a-presto-installation-on-hdinsight-cluster"></a>在 HDInsight 叢集上自訂 Presto 安裝

若要自訂安裝，請使用下列步驟：

1. 使用 SSH 連線到已安裝 Presto 的 HDInsight 叢集前端節點︰
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
   
    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 在 `/var/lib/presto/presto-hdinsight-master/appConfig-default.json` 檔案中進行設定變更。 如需 Presto 設定的詳細資訊，請參閱[以 YARN 作為基礎之叢集的 Presto 設定](https://prestodb.io/presto-yarn/installation-yarn-configuration-options.html)。

3. 將 Presto 目前執行中的執行個體終止並刪除。

        sudo slider stop presto1 --force
        sudo slider destroy presto1 --force

4. 使用自訂來啟動 Presto 的新執行個體。

       sudo slider create presto1 --template /var/lib/presto/presto-hdinsight-master/appConfig-default.json --resources /var/lib/presto/presto-hdinsight-master/resources-default.json

5. 等候新的執行個體就緒，並記下 Presto 協調者地址。


       sudo slider registry --name presto1 --getexp presto

## <a name="generate-benchmark-data-for-hdinsight-clusters-that-run-presto"></a>產生執行 Presto 之 HDInsight 叢集的基準測試資料

TPC-DS 是業界標準，可用於測量許多決策支援系統 (包括巨量資料系統) 的效能。 您可以使用 Presto 來產生資料，並將它與您自己的 HDInsight 基準測試資料比較，從而進行評估。 如需詳細資訊，請參閱[這裡](https://github.com/hdinsight/tpcds-datagen-as-hive-query/blob/master/README.md)。



## <a name="see-also"></a>另請參閱
* [在 HDInsight 叢集上安裝及使用 Hue](hdinsight-hadoop-hue-linux.md)。 Hue 是 Web UI，可讓您更輕鬆地建立、執行及儲存 Pig 和 Hive 作業。

* [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Giraph。 Giraph 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。

* [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Solr。 Solr 可讓您對儲存的資料執行功能強大的搜尋作業。

[hdinsight-install-r]: hdinsight-hadoop-r-scripts-linux.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md
