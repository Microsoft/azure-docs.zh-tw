---
title: 使用指令碼動作在 Hadoop 叢集上安裝 Solr - Azure
description: 深入了解如何使用指令碼動作來以 Solr 自訂 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/05/2016
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: e0c1f2ecbe86eda5161324bfe84c7cf8f7d31d60
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43110749"
---
# <a name="install-and-use-solr-on-windows-based-hdinsight-clusters"></a>在 Windows 型 HDInsight 叢集上安裝和使用 Solr

了解如何使用指令碼動作來以 Solr 自訂 Windows 型 HDInsight 叢集，以及如何使用 Solr 搜尋資料。

> [!IMPORTANT]
> 本文件的步驟只適用於 Windows HDInsight 叢集。 Windows 上的 HDInsight 只提供低於 HDInsight 3.4 的版本。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。 如需搭配以 Linux 為基礎的叢集使用 Solr 的詳細資訊，請參閱 [在 HDInsight Hadoop 叢集上安裝和使用 Solr (Linux)](hdinsight-hadoop-solr-install-linux.md)


您也可以使用「指令碼動作」 ，在 Azure HDInsight 的任一類型的叢集 (Hadoop、Storm、HBase、Spark) 上安裝 Solr。 您可以從 [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1) 的唯讀 Azure 儲存體 blob 中，取得用來在 HDInsight 叢集上安裝 Solr 的範例指令碼。

範例指令碼只能與 HDInsight 叢集版本 3.1 搭配使用。 如需 HDInsight 叢集版本的詳細資訊，請參閱 [HDInsight 叢集版本](hdinsight-component-versioning.md)。

本主題中使用的範例指令碼會以特定組態建立以 Windows 為基礎的 Solr 叢集。 如果您想要以不同的集合、分區、結構描述和複本等項目設定 Solr 叢集，則必須相應修改指令碼和 Solr 二進位檔。

**相關文章**

* [在 HDInsight Hadoop 叢集上安裝和使用 Solr (Linux)](hdinsight-hadoop-solr-install-linux.md)
* [在 HDInsight 叢集中建立 Hadoop](hdinsight-provision-clusters.md)：建立 HDInsight 叢集的一般資訊。
* [使用指令碼動作來自訂 HDInsight 叢集][hdinsight-cluster-customize]：關於使用指令碼動作來自訂 HDInsight 叢集的一般資訊。
* [開發 HDInsight 的指令碼動作指令碼](hdinsight-hadoop-script-actions.md)

## <a name="what-is-solr"></a>什麼是 Solr？
<a href="http://lucene.apache.org/solr/features.html" target="_blank">Apache Solr</a> 是可對資料執行強大全文搜尋作業的企業搜尋平台。 Hadoop 可儲存和管理大量資料，而 Apache Solr 則是提供搜尋功能以便快速擷取資料。

## <a name="install-solr-using-portal"></a>使用入口網站安裝 Solr
1. 使用 [自訂建立] 選項，依[在 HDInsight 中建立 Hadoop 叢集](hdinsight-provision-clusters.md)中的描述開始建立叢集。
2. 在精靈的 [指令碼動作] 頁面上，按一下 [加入指令碼動作] 以提供有關指令碼動作的詳細資料，如下所示：

    ![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-solr-install/hdi-script-action-solr.png "使用指令碼動作以自訂叢集")

    <table border='1'>
        <tr><th>屬性</th><th>值</th></tr>
        <tr><td>名稱</td>
            <td>指定指令碼動作的名稱。 例如，<b>安裝 Solr</b>。</td></tr>
        <tr><td>指令碼 URI</td>
            <td>指定為自訂叢集叫用的指令碼統一資源識別項 (URI)。 例如，<i>https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1</i></td></tr>
        <tr><td>節點類型</td>
            <td>指定執行自訂指令碼的節點。 您可以選擇 [所有節點]<b></b>、[僅限前端節點]<b></b> 或 [僅限背景工作節點]<b></b>。
        <tr><td>參數</td>
            <td>如果指令碼要求，請指定參數。 用來安裝 Solr 的指令碼不需要任何參數，因此可以讓此處空白。</td></tr>
    </table>

    您可以加入一個以上的指令碼動作，以在叢集上安裝多個元件。 加入指令碼之後，請按一下核取記號以開始建立叢集。

## <a name="use-solr"></a>使用 Solr
您必須從以某些資料檔案編製 Solr 的索引來開始。 然後，您可以使用 Solr 來對已編製索引的資料執行搜尋查詢。 執行下列步驟以在 HDInsight 叢集中使用 Solr：

1. **使用遠端桌面通訊協定 (RDP) 遠端登入到已安裝 Solr 的 HDInsight 叢集**。 從 Azure 入口網站，針對您所建立且已安裝 Solr 的叢集啟用遠端桌面，然後遠端登入到叢集。 如需指示，請參閱 [使用 RDP 連接至 HDInsight 叢集](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)。
2. **上傳資料檔案以對 Solr 編製索引**。 在對 Solr 編製索引時，會在其中放置可能需要搜尋的文件。 若要對 Solr 編製索引，請使用 RDP 遠端登入到叢集、瀏覽至桌面、開啟 Hadoop 命令列，然後瀏覽至 **C:\apps\dist\solr-4.7.2\example\exampledocs**。 執行以下命令：

        java -jar post.jar solr.xml monitor.xml

    您會在主控台上看到下列輸出：

        POSTing file solr.xml
        POSTing file monitor.xml
        2 files indexed.
        COMMITting Solr index changes to http://localhost:8983/solr/update..
        Time spent: 0:00:01.624

    post.jar 公用程式使用 **solr.xml** 和 **monitor.xml** 這兩個範例文件對 Solr 編製索引。 您可以在 Solr 安裝內取得 post.jar 公用程式和範例文件。
3. **使用 Solr 儀表板在已編製索引的文件內執行搜尋**。 在連往 HDInsight 叢集的 RDP 工作階段內，開啟 Internet Explorer，然後在 **http://headnodehost:8983/solr/#/** 啟動 Solr 儀表板。 在左窗格的 [核心選取器] 下拉式清單中，選取 [collection1]，然後在其中按一下 [查詢]。 舉例來說，若要選取並傳回 Solr 中的所有文件，請提供下列值：

   * 在 [q] 文字方塊中輸入 **:**\*\*。 如此便會傳回已在 Solr 中編製索引的所有文件。 如果您想要搜尋文件內的特定字串，您可以在此輸入該字串。
   * 在 [ **wt** ] 文字方塊中，選取輸出格式。 預設值是 [ **json**]。 按一下 [ **執行查詢**]。

     ![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-solr-install/hdi-solr-dashboard-query.png "在 Solr 儀表板上執行查詢")

     輸出中會傳回兩個我們之前用於對 Solr 編製索引的文件。 輸出結果類似下面：

           "response": {
               "numFound": 2,
               "start": 0,
               "maxScore": 1,
               "docs": [
                 {
                   "id": "SOLR1000",
                   "name": "Solr, the Enterprise Search Server",
                   "manu": "Apache Software Foundation",
                   "cat": [
                     "software",
                     "search"
                   ],
                   "features": [
                     "Advanced Full-Text Search Capabilities using Lucene",
                     "Optimized for High Volume Web Traffic",
                     "Standards Based Open Interfaces - XML and HTTP",
                     "Comprehensive HTML Administration Interfaces",
                     "Scalability - Efficient Replication to other Solr Search Servers",
                     "Flexible and Adaptable with XML configuration and Schema",
                     "Good unicode support: héllo (hello with an accent over the e)"
                   ],
                   "price": 0,
                   "price_c": "0,USD",
                   "popularity": 10,
                   "inStock": true,
                   "incubationdate_dt": "2006-01-17T00:00:00Z",
                   "_version_": 1486960636996878300
                 },
                 {
                   "id": "3007WFP",
                   "name": "Dell Widescreen UltraSharp 3007WFP",
                   "manu": "Dell, Inc.",
                   "manu_id_s": "dell",
                   "cat": [
                     "electronics and computer1"
                   ],
                   "features": [
                     "30\" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
                   ],
                   "includes": "USB cable",
                   "weight": 401.6,
                   "price": 2199,
                   "price_c": "2199,USD",
                   "popularity": 6,
                   "inStock": true,
                   "store": "43.17614,-90.57341",
                   "_version_": 1486960637584081000
                 }
               ]
             }
4. **建議步驟：從 Solr 將已編製索引的資料備份到與 HDInsight 叢集相關聯的 Azure Blob 儲存體**。 您最好從 Solr 叢集節點將已編製索引的資料備份到 Azure Blob 儲存體。 請執行下列步驟來進行此作業：

   1. 從 RDP 工作階段開啟 Internet Explorer，然後指向下列 URL：

           http://localhost:8983/solr/replication?command=backup

       您應該會看到如下所示的回應：
            
      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
          <response>
             <lst name="responseHeader">
               <int name="status">0</int>
               <int name="QTime">9</int>
             </lst>
            <str name="status">OK</str>
          </response>
      ```
      
   2. 在遠端工作階段中，瀏覽至 {SOLR_HOME}\{Collection}\data。 若為以範例指令碼建立的叢集，則應為 `C:\apps\dist\solr-4.7.2\example\solr\collection1\data`。 在此位置中，您應該會看到快照資料夾已使用類似 *snapshot.timestamp*** 的名稱建立。
   
   3. 壓縮快照資料夾，並上傳至 Azure Blob 儲存體。 從 Hadoop 命令列使用下列命令瀏覽至快照資料夾的位置：

      ```
      hadoop fs -CopyFromLocal snapshot._timestamp_.zip /example/data
      ```

   此命令會將快照複製到與叢集相關聯之預設儲存體帳戶內的容器下的 /example/data/。

## <a name="install-solr-using-aure-powershell"></a>使用 Azure PowerShell 安裝 Solr
請參閱[使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell)。  此範例示範如何使用 Azure PowerShell 安裝 Spark。 您需要自訂指令碼以使用 [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1)。

## <a name="install-solr-using-net-sdk"></a>使用 .NET SDK 安裝 Solr
請參閱[使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell)。 此範例示範如何使用 .NET SDK 安裝 Spark。 您需要自訂指令碼以使用 [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1)。

## <a name="see-also"></a>另請參閱
* [在 HDInsight Hadoop 叢集上安裝和使用 Solr (Linux)](hdinsight-hadoop-solr-install-linux.md)
* [在 HDInsight 叢集中建立 Hadoop](hdinsight-provision-clusters.md)：建立 HDInsight 叢集的一般資訊。
* [使用指令碼動作來自訂 HDInsight 叢集][hdinsight-cluster-customize]：關於使用指令碼動作來自訂 HDInsight 叢集的一般資訊。
* [開發 HDInsight 的指令碼動作指令碼](hdinsight-hadoop-script-actions.md)
* [在 HDInsight 叢集上安裝和使用 Spark][hdinsight-install-spark]：關於安裝 Spark 的指令碼動作範例。
* [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install.md)：關於安裝 Giraph 的指令碼動作範例。

[powershell-install-configure]: /powershell/azureps-cmdlets-docs
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-install-spark]: hdinsight-hadoop-spark-install.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster.md
