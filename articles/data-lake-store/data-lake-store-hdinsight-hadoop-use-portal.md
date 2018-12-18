---
title: 使用 Azure 入口網站建立搭配 Azure Data Lake Storage Gen1 運作的 Azure HDInsight 叢集 | Microsoft Docs
description: 使用 Azure 入口網站建立和使用搭配 Azure Data Lake Storage Gen1 運作的 HDInsight 叢集
services: data-lake-store,hdinsight
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: a8c45a83-a8e3-4227-8b02-1bc1e1de6767
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 28bed3578c653c8081868b3d950ab6332879784a
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46124316"
---
# <a name="create-hdinsight-clusters-with-azure-data-lake-storage-gen1-by-using-the-azure-portal"></a>使用 Azure 入口網站建立搭配 Azure Data Lake Storage Gen1 的 HDInsight 叢集
> [!div class="op_single_selector"]
> * [使用 Azure 入口網站](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [使用 PowerShell (針對預設儲存體)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [使用 PowerShell (針對額外儲存體)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [使用 Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

了解如何利用 Azure 入口網站，建立使用 Azure Data Lake Storage Gen1 帳戶作為預設儲存體或其他儲存體的 HDInsight 叢集。 即使其他儲存體是 HDInsight 叢集的選用項目，還是建議將您的商務資料儲存在其他儲存體帳戶中。

## <a name="prerequisites"></a>先決條件
開始此教學課程之前，確保您已符合下列需求：

* **Azure 訂用帳戶**。 請移至[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Data Lake Storage Gen1 帳戶**。 遵循[使用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示操作。 您也必須在此帳戶上建立根資料夾。  在此教學課程中，會使用名為 __/clusters__ 的根資料夾。
* **Azure Active Directory 服務主體**。 此教學課程提供有關如何在 Azure Active Directory (Azure AD) 中建立服務主體的指示。 不過，您必須是 Azure AD 系統管理員，才能建立服務主體。 如果您是系統管理員，就可以略過此先決條件並繼續進行此教學課程。

    >[!NOTE]
    >唯有您是 Azure AD 系統管理員，才可以建立服務主體。 您的 Azure AD 系統管理員必須先建立服務主體，您才能建立搭配 Data Lake Storage Gen1 的 HDInsight 叢集。 此外，必須使用憑證來建立服務主體，如[使用憑證來建立服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md#create-service-principal-with-self-signed-certificate)所述。
    >

## <a name="create-an-hdinsight-cluster"></a>建立 HDInsight 叢集

在此節中，您會建立一個使用 Data Lake Storage Gen1 帳戶作為預設或其他儲存體的 HDInsight 叢集。 此文章僅著重於設定 Data Lake Storage Gen1 帳戶的部分。  如需一般叢集建立資訊和程序，請參閱 [在 HDInsight 中建立 Hadoop 叢集](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。

### <a name="create-a-cluster-with-data-lake-storage-gen1-as-default-storage"></a>建立以 Data Lake Storage Gen1 做為預設儲存體的叢集

**建立使用 Data Lake Storage Gen1 作為預設儲存體帳戶的 HDInsight 叢集**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 請遵循[建立叢集](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md#create-clusters)，以取得建立 HDInsight 叢集的一般資訊。
3. 在 [儲存體] 刀鋒視窗的 [主要儲存體類型] 下，選取 [Azure Data Lake Storage Gen1]，然後輸入下列資訊：

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.1.adls.storage.png "將服務主體新增至 HDInsight 叢集")

    - **選取 Data Lake Store 帳戶**： 選取現有的 Data Lake Storage Gen1 帳戶。 需要現有的 Data Lake Storage Gen1 帳戶。  請參閱[先決條件](#prerequisites)。
    - **根路徑**：輸入要儲存叢集特定檔案的路徑。 在螢幕擷取畫面上，它是 __/clusters/myhdiadlcluster/__，其中 __/clusters__ 資料夾必須已存在，而入口網站會建立 *myhdicluster* 資料夾。  *myhdicluster* 是叢集名稱。
    - **Data Lake Store 存取**：設定 Data Lake Storage Gen1 帳戶與 HDInsight 叢集之間的存取。 如需指示，請參閱[設定 Data Lake Storage Gen1 存取](#configure-data-lake-store-access)。
    - **其他儲存體帳戶**：新增 Azure 儲存體帳戶作為叢集的其他儲存體帳戶。 若要新增其他 Data Lake Storage Gen1 帳戶，做法是設定某個 Data Lake Storage Gen1 帳戶作為主要儲存體類型，同時授與叢集存取其他 Data Lake Storage Gen1 帳戶資料的權限。 [設定 Data Lake Storage Gen1 存取](#configure-data-lake-store-access)。

4. 在 [Data Lake Store 存取權] 上按一下 [選取]，然後繼續進行叢集建立作業，如[在 HDInsight 中建立 Hadoop 叢集](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md)所述。


### <a name="create-a-cluster-with-data-lake-storage-gen1-as-additional-storage"></a>建立以 Data Lake Storage Gen1 做為額外儲存體的叢集

下列指示說明如何建立使用 Azure 儲存體帳戶作為預設儲存體，並使用 Data Lake Storage Gen1 帳戶作為其他儲存體的 HDInsight 叢集。

**建立使用 Data Lake Storage Gen1 作為額外儲存體帳戶的 HDInsight 叢集**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 請遵循[建立叢集](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md#create-clusters)，以取得建立 HDInsight 叢集的一般資訊。
3. 在 [儲存體] 刀鋒視窗的 [主要儲存體類型] 下，選取 [Azure 儲存體]，然後輸入下列資訊：

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.1.png "將服務主體新增至 HDInsight 叢集")

    - **選取方法**：使用下列其中一個選項：

        * 若要指定屬於 Azure 訂用帳戶的儲存體帳戶，請選取 [我的訂用帳戶]，然後選取儲存體帳戶。
        * 若要指定 Azure 訂用帳戶外部的儲存體帳戶，請選取 [存取金鑰]，然後提供外部儲存體帳戶資訊。

    - **預設容器**：使用預設值或指定您自己的名稱。

    - 其他儲存體帳戶：新增其他 Azure 儲存體帳戶作為額外儲存體。
    - Data Lake Store 存取：設定 Data Lake Storage Gen1 帳戶與 HDInsight 叢集之間的存取。 如需指示，請參閱[設定 Data Lake Storage Gen1 存取](#configure-data-lake-storage-gen1-access)。

## <a name="configure-data-lake-storage-gen1-access"></a>設定 Data Lake Storage Gen1 存取 

在此節中，您將會使用 Azure Active Directory 服務主體設定 Data Lake Storage Gen1 與 HDInsight 叢集之間的存取。 

### <a name="specify-a-service-principal"></a>指定服務主體

從 Azure 入口網站中，您可以使用現有的服務主體或建立一個新的服務主體。

**從 Azure 入口網站中建立服務主體**

1. 從 [儲存體] 刀鋒視窗中，按一下 [Data Lake Store 存取]。
2. 在 [Data Lake Storage Gen1 存取] 刀鋒視窗中，按一下 [新建]。
3. 按一下 [服務主體]，然後遵循指示建立服務主體。
4. 如果您決定要在未來再次使用它，請下載憑證。 如果您想要在建立其他 HDInsight 叢集時使用相同的服務主體，下載憑證非常實用。

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.2.png "將服務主體新增至 HDInsight 叢集")

4. 按一下 [存取] 來設定資料夾存取。  請參閱[設定檔案權限](#configure-file-permissions)。


**使用 Azure 入口網站中現有的服務主體**

1. 按一下 [Data Lake Store 存取]。
1. 在 [Data Lake Storage Gen1 存取] 刀鋒視窗中，按一下 [使用現有項目]。
2. 按一下 [服務主體]，然後選取一個服務主體。 
3. 上傳與您所選取服務主體建立關聯的憑證 (.pfx 檔案)，然後輸入憑證密碼。

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.5.png "將服務主體新增至 HDInsight 叢集")

4. 按一下 [存取] 來設定資料夾存取。  請參閱[設定檔案權限](#configure-file-permissions)。


### <a name="configure-file-permissions"></a>設定檔案權限

這些設定會視帳戶是作為預設儲存體帳戶還是其他儲存體帳戶而不同：

- 作為預設儲存體

    - Data Lake Storage Gen1 帳戶根層級的權限
    - HDInsight 叢集儲存體根層級的權限。 例如，稍早在教學課程中使用的 __/clusters__ 資料夾。
- 作為其他儲存體

    - 您需要其檔案存取權之資料夾的權限。

**指派 Data Lake Storage Gen1 帳戶根層級的權限**

1. 在 [Data Lake Storage Gen1 存取權] 刀鋒視窗中，按一下 [存取]。 隨即會開啟 [選取檔案權限] 刀鋒視窗。 其中列出您訂用帳戶中的所有 Data Lake Storage Gen1 帳戶。
2. 請將滑鼠游標暫留 (請勿按一下) 在 Data Lake Storage Gen1 帳戶的名稱上，以顯示核取方塊，然後選取此核取方塊。

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.3.png "將服務主體新增至 HDInsight 叢集")

  根據預設，__READ__、__WRITE__ 和 __EXECUTE__ 會全部選取。

3. 按一下頁面底部的 [選取]。
4. 按一下 [執行] 來指派權限。
5. 按一下 [完成] 。

**指派 HDInsight 叢集根層級的權限**

1. 在 [Data Lake Storage Gen1 存取權] 刀鋒視窗中，按一下 [存取]。 隨即會開啟 [選取檔案權限] 刀鋒視窗。 其中列出您訂用帳戶中的所有 Data Lake Storage Gen1 帳戶。
1. 從 [選取檔案權限] 刀鋒視窗中，按一下要顯示其內容的 Data Lake Storage Gen1 名稱。
2. 選取 HDInsight 叢集儲存體根目錄左邊的核取方塊，即可選取該資料夾。 根據先前的螢幕擷取畫面，叢集儲存體根目錄為您在選取 Data Lake Storage Gen1 作為預設儲存體時所指定的 __/clusters__ 資料夾。
3. 設定資料夾的權限。  根據預設，讀取、寫入和執行會全部選取。
4. 按一下頁面底部的 [選取]。
5. 按一下 **[執行]**。
6. 按一下 [完成] 。

如果您使用 Data Lake Storage Gen1 作為額外儲存體，則必須僅針對您想要從 HDInsight 叢集存取的資料夾指派權限。 例如，在下方的螢幕擷取畫面中，您僅會將存取提供給 Data Lake Storage Gen1 帳戶中的 **mynewfolder** 資料夾。

![將服務主體權限指派給 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.3-1.png "將服務主體權限指派給 HDInsight 叢集")


## <a name="verify-cluster-set-up"></a>確認叢集設定

叢集設定完成之後，在叢集刀鋒視窗上執行下列任一或這兩個步驟來驗證結果：

* 若要驗證叢集的相關儲存體是否為您指定的 Data Lake Storage Gen1 帳戶，請按一下左窗格中的 [儲存體帳戶]。

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.6-1.png "將服務主體新增至 HDInsight 叢集")

* 若要驗證服務主體是否正確地與 HDInsight 叢集相關聯，請按一下左窗格中的 [Data Lake Storage Gen1 存取權]。

    ![將服務主體新增至 HDInsight 叢集](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.6.png "將服務主體新增至 HDInsight 叢集")


## <a name="examples"></a>範例

在您設定以 Data Lake Storage Gen1 做為儲存體的叢集之後，請參考一些範例以了解如何使用 HDInsight 叢集來分析 Data Lake Storage Gen1 中儲存的資料。

### <a name="run-a-hive-query-against-data-in-a-data-lake-storage-gen1-account-as-primary-storage"></a>針對 Data Lake Storage Gen1 帳戶 (做為主要儲存體) 中的資料執行 Hive 查詢

若要執行 Hive 查詢，請使用 Ambari 入口網站中的 Hive 檢視介面。 如需有關如何使用 Ambari Hive 檢視的指示，請參閱[在 HDInsight 中搭配 Hadoop 使用 Hive 檢視](../hdinsight/hadoop/apache-hadoop-use-hive-ambari-view.md)。

當您使用 Data Lake Storage Gen1 帳戶中的資料時，有幾個字串需要變更。

如果您使用以 Data Lake Storage Gen1 做為主要儲存體而建立的叢集，則資料路徑為︰adl://<data_lake_storage_gen1_account_name>/azuredatalakestore.net/path/to/file。 從 Data Lake Storage Gen1 帳戶中儲存的範例資料建立資料表的 Hive 查詢，類似於下列陳述式：

    CREATE EXTERNAL TABLE websitelog (str string) LOCATION 'adl://hdiadlsg1storage.azuredatalakestore.net/clusters/myhdiadlcluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/'

說明：
* `adl://hdiadlsg1storage.azuredatalakestore.net/` 是 Data Lake Storage Gen1 帳戶的根。
* `/clusters/myhdiadlcluster` 是您在建立叢集時指定之叢集資料的根目錄。
* `/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/` 是您在查詢中使用之範例檔案的位置。

### <a name="run-a-hive-query-against-data-in-a-data-lake-storage-gen1-account-as-additional-storage"></a>針對 Data Lake Storage Gen1 帳戶 (做為額外儲存體) 中的資料執行 Hive 查詢

如果您建立的叢集使用 Blob 儲存體做為預設儲存體，則範例資料不會包含在做為額外儲存體的 Data Lake Storage Gen1 帳戶中。 在這種情況下，請先將資料從 Blob 儲存體傳送至 Data Lake Storage Gen1 帳戶，然後如上述範例所示執行查詢。

如需如何將資料從 Blob 儲存體複製到 Data Lake Storage Gen1 帳戶的詳細資訊，請參閱下列文章：

* [使用 Distcp 在 Azure 儲存體 Blob 與 Data Lake Storage Gen1 之間複製資料](data-lake-store-copy-data-wasb-distcp.md)
* [使用 AdlCopy 將資料從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)

### <a name="use-data-lake-storage-gen1-with-a-spark-cluster"></a>使用 Data Lake Storage Gen1 搭配 Spark 叢集
您可以使用 Spark 叢集對 Data Lake Storage Gen1 中儲存的資料執行 Spark 作業。 如需詳細資訊，請參閱[使用 HDInsight Spark 叢集來分析 Data Lake Storage Gen1 中的資料](../hdinsight/spark/apache-spark-use-with-data-lake-store.md)。


### <a name="use-data-lake-storage-gen1-in-a-storm-topology"></a>在 Storm 拓撲中使用 Data Lake Storage Gen1
您可以使用 Data Lake Storage Gen1 帳戶從 Storm 拓撲寫入資料。 如需有關如何達到這種情況的指示，請參閱[搭配使用 Azure Data Lake Storage Gen1 與 HDInsight 上的 Apache Storm](../hdinsight/storm/apache-storm-write-data-lake-store.md)。

## <a name="see-also"></a>另請參閱
* [搭配使用 Data Lake Storage Gen1 與 Azure HDInsight 叢集](../hdinsight/hdinsight-hadoop-use-data-lake-store.md)
* [PowerShell：建立 HDInsight 叢集以使用 Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-powershell.md)

[makecert]: https://msdn.microsoft.com/library/windows/desktop/ff548309(v=vs.85).aspx
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx
