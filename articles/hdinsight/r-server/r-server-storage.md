---
title: 適用於 HDInsight 上 ML 服務的 Azure 儲存體解決方案 - Azure
description: 了解 HDInsight 上 ML 服務可用的不同儲存體選項
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 06/27/2018
ms.openlocfilehash: 5bd5919efa84f2dd22929075b806747b413ac346
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051788"
---
# <a name="azure-storage-solutions-for-ml-services-on-azure-hdinsight"></a>適用於 Azure HDInsight 上 ML 服務的 Azure 儲存體解決方案

HDInsight 上的 ML 服務可以使用數種儲存體解決方案，來保存資料、程式碼或包含分析結果的物件。 這些包括下列選項：

- [Azure Blob](https://azure.microsoft.com/services/storage/blobs/)
- [Azure Data Lake 儲存體](https://azure.microsoft.com/services/data-lake-store/)
- [Azure 檔案儲存體](https://azure.microsoft.com/services/storage/files/)

您也可以使用 HDInsight 叢集存取多個 Azure 儲存體帳戶或容器。 「Azure 檔案」儲存體是一個可在邊緣節點上使用的方便資料儲存體選項，可讓您掛接「Azure 儲存體」檔案共用 (例如掛接到 Linux 檔案系統)。 但是，只要是擁有受支援作業系統 (例如 Windows 或 Linux) 的系統，就可以掛接和使用「Azure 檔案」共用。 

當您在 HDInsight 中建立 Hadoop 叢集時，您可以指定一個 **Azure 儲存體**帳戶或一個 **Data Lake Store**。 來自該帳戶的特定儲存體容器，會保留您所建立叢集的檔案系統 (例如，Hadoop 分散式檔案系統)。 如需詳細資訊與指導方針，請參閱：

- [搭配 HDInsight 使用 Azure 儲存體](../hdinsight-hadoop-use-blob-storage.md)
- [搭配 Azure HDInsight 叢集使用 Data Lake Store](../hdinsight-hadoop-use-data-lake-store.md)

## <a name="use-azure-blob-storage-accounts-with-ml-services-cluster"></a>搭配 ML 服務叢集使用 Azure Blob 儲存體帳戶

如果您在建立 ML 服務叢集時指定一個以上的儲存體帳戶，下列指示說明如何針對在 ML 服務叢集上的資料存取和作業使用次要帳戶。 假設下列儲存體帳戶和容器：**storage1** 和名為 **container1** 的預設容器，以及具有 **container2** 的 **storage2**。

> [!WARNING]
> 基於效能目的，系統會在與您指定的主要儲存體帳戶相同資料中心內建立 HDInsight 叢集。 不支援在與 HDInsight 叢集不同的位置中使用儲存體帳戶。

### <a name="use-the-default-storage-with-ml-services-on-hdinsight"></a>搭配 HDInsight 上的 ML 服務使用預設儲存體

1. 使用 SSH 用戶端，連線到您叢集的邊緣節點。 如需搭配 HDInsight 叢集使用 SSH 的相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。
  
2. 在 /share 目錄建立範例檔案 mysamplefile.csv。 

        hadoop fs –mkdir /share
        hadoop fs –copyFromLocal mycsv.scv /share  

3. 切換至 R Studio 或其他 R 主控台，然後撰寫 R 程式碼，將名稱節點設定為**預設**和您想要存取的檔案位置。  

        myNameNode <- "default"
        myPort <- 0

        #Location of the data:  
        bigDataDirRoot <- "/share"  

        #Define Spark compute context:
        mySparkCluster <- RxSpark(nameNode=myNameNode, consoleOutput=TRUE)

        #Set compute context:
        rxSetComputeContext(mySparkCluster)

        #Define the Hadoop Distributed File System (HDFS) file system:
        hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

        #Specify the input file to analyze in HDFS:
        inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")

所有目錄和檔案的參考會指向儲存體帳戶 `wasb://container1@storage1.blob.core.windows.net`。 這是與 HDInsight 叢集關聯的「預設儲存體帳戶」。

### <a name="use-the-additional-storage-with-ml-services-on-hdinsight"></a>搭配 HDInsight 上的 ML 服務使用其他儲存體

現在，假設您想要處理名稱為 mysamplefile1.csv 的檔案，其所在位置為 **storage2** 中 **container2** 的 /private 目錄。

在 R 程式碼中，將名稱節點參考指向 **storage2** 儲存體帳戶。

    myNameNode <- "wasb://container2@storage2.blob.core.windows.net"
    myPort <- 0

    #Location of the data:
    bigDataDirRoot <- "/private"

    #Define Spark compute context:
    mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

    #Set compute context:
    rxSetComputeContext(mySparkCluster)

    #Define HDFS file system:
    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    #Specify the input file to analyze in HDFS:
    inputFile <-file.path(bigDataDirRoot,"mysamplefile1.csv")

現在，所有目錄和檔案的參考會指向儲存體帳戶 `wasb://container2@storage2.blob.core.windows.net`。 這是您已指定的「名稱節點」。

您必須在 **storage2** 上設定 /user/RevoShare/<SSH username> 目錄，如下所示：


    hadoop fs -mkdir wasb://container2@storage2.blob.core.windows.net/user
    hadoop fs -mkdir wasb://container2@storage2.blob.core.windows.net/user/RevoShare
    hadoop fs -mkdir wasb://container2@storage2.blob.core.windows.net/user/RevoShare/<RDP username>

## <a name="use-an-azure-data-lake-store-with-ml-services-cluster"></a>搭配 ML 服務叢集使用 Azure Data Lake Store 

若要搭配 HDInsight 叢集帳戶使用 Data Lake Store，您必須為叢集提供您想要使用之每個 Azure Data Lake Store 的存取權。 如需有關如何使用 Azure 入口網站並使用 Azure Data Lake Store 帳戶作為預設儲存體或作為其他存放區來建立 HDInsight 叢集的指示，請參閱 [使用 Azure 入口網站搭配 Data Lake Store 建立 HDInsight 叢集](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)。

接著，您會以和使用次要 Azure 儲存體帳戶的方式 (如先前程序所述) 很像的方式，在 R 指令碼中使用存放區。

### <a name="add-cluster-access-to-your-azure-data-lake-stores"></a>為 Azure Data Lake Store 新增叢集存取權
您可以使用與 HDInsight 叢集相關聯的 Azure Active Directory (Azure AD) 服務主體來存取 Data Lake Store。

1. 在建立 HDInsight 叢集時，從 [資料來源] 索引標籤中選取 [叢集 AAD 身分識別]。

2. 在 [叢集 AAD 身分識別] 對話方塊的 [選取 AD 服務主體] 底下，選取 [新建]。

在為服務主體命名並建立密碼後，請按一下 [管理 ADLS 存取]，以將該服務主體與您的 Data Lake Store 產生關聯。

建立叢集之後，您也可以新增叢集存取權到一或多個 Data Lake Store 帳戶。 為 Data Lake Store 開啟 Azure 入口網站入口，並移至 [資料總管] > [存取] > [新增]。 

### <a name="how-to-access-the-data-lake-store-from-ml-services-on-hdinsight"></a>如何從 HDInsight 上的 ML 服務存取 Data Lake Store

一旦您獲得 Data Lake Store 的存取權後，就可以在 HDInsight 上的 ML 服務叢集中使用該存放區，其方式就和使用次要 Azure 儲存體帳戶一樣。 唯一的差別在於前置詞 **wasb://** 會變更為 **adl://**，如下所示：


    # Point to the ADL store (e.g. ADLtest)
    myNameNode <- "adl://rkadl1.azuredatalakestore.net"
    myPort <- 0

    # Location of the data (assumes a /share directory on the ADL account)
    bigDataDirRoot <- "/share"  

    # Define Spark compute context
    mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

    # Set compute context
    rxSetComputeContext(mySparkCluster)

    # Define HDFS file system
    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    # Specify the input file in HDFS to analyze
    inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")

下列命令是用來搭配 RevoShare 目錄設定 Data Lake Store 帳戶，並新增來自先前範例的範例 .csv 檔案：


    hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user
    hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare
    hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare/<user>

    hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/share

    hadoop fs -copyFromLocal /usr/lib64/R Server-7.4.1/library/RevoScaleR/SampleData/mysamplefile.csv adl://rkadl1.azuredatalakestore.net/share

    hadoop fs –ls adl://rkadl1.azuredatalakestore.net/share


## <a name="use-azure-file-storage-with-ml-services-on-hdinsight"></a>搭配 HDInsight 上的 ML 服務使用 Azure 檔案儲存體

另外還有可在邊緣節點上使用的便利資料儲存選項，我們稱之為 [Azure 檔案]((https://azure.microsoft.com/services/storage/files/)。 它可以讓您將 Azure 儲存體的檔案共用掛接至 Linux 檔案系統。 此選項對於儲存資料檔案、R 指令碼與結果物件相當便利，該結果物件在稍後可以於邊緣節點 (而不是 HDFS) 上使用原生檔案系統時需要。 

Azure 檔案的主要優點是，只要是擁有受支援作業系統 (例如 Windows 或 Linux) 的系統，就可以掛接和使用檔案共用。 例如，您本人或您的小組成員所擁有的另一個 HDInsight 叢集，或是 Azure VM 甚或內部部署系統均可使用 Azure 檔案。 如需詳細資訊，請參閱

- [如何搭配 Linux 使用 Azure 檔案儲存體](../../storage/files/storage-how-to-use-files-linux.md)
- [如何在 Windows 上使用 Azure 檔案儲存體](../../storage/files/storage-dotnet-how-to-use-files.md)


## <a name="next-steps"></a>後續步驟

* [HDInsight 上的 ML 服務叢集概觀](r-server-overview.md)
* [開始使用 Hadoop 上的 ML 服務叢集](r-server-get-started.md)
* [在 HDInsight 上計算 ML 服務叢集的內容選項](r-server-compute-contexts.md)

