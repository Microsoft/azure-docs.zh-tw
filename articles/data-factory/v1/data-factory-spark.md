---
title: 從 Azure Data Factory 叫用 Spark 程式 | Microsoft Docs
description: 了解如何從 Azure Data Factory 使用 MapReduce 活動來叫用 Spark 程式。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: ''
editor: ''
ms.assetid: fd98931c-cab5-4d66-97cb-4c947861255c
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 2c00e42284783439b0a01f6ba6bab31be053b1c9
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45736405"
---
# <a name="invoke-spark-programs-from-azure-data-factory-pipelines"></a>從 Azure Data Factory 叫用 Spark 程式管線

> [!div class="op_single_selector" title1="Transformation Activities"]
> * [Hive 活動](data-factory-hive-activity.md)
> * [Pig 活動](data-factory-pig-activity.md)
> * [MapReduce 活動](data-factory-map-reduce.md)
> * [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)
> * [Spark 活動](data-factory-spark.md)
> * [Machine Learning 批次執行活動](data-factory-azure-ml-batch-execution-activity.md)
> * [Machine Learning 更新資源活動](data-factory-azure-ml-update-resource-activity.md)
> * [預存程序活動](data-factory-stored-proc-activity.md)
> * [Data Lake Analytics U-SQL 活動](data-factory-usql-activity.md)
> * [.NET 自訂活動](data-factory-use-custom-activities.md)

> [!NOTE]
> 本文適用於第 1 版 Azure Data Factory (正式運作版)。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中使用 Apache Spark 活動來轉換資料](../transform-data-using-spark.md)。

## <a name="introduction"></a>簡介
Spark 活動是 Data Factory 所支援的其中一個[資料轉換活動](data-factory-data-transformation-activities.md)。 此活動會在 Azure HDInsight 中 Spark 叢集上執行指定的 Spark 程式。 

> [!IMPORTANT]
> - Spark 活動不支援使用 Azure Data Lake Store 作為主要儲存體的 HDInsight Spark 叢集。
> - Spark 活動僅支援現有 (您自己的) HDInsight Spark 叢集。 它不支援隨選 HDInsight 已連結服務。

## <a name="walkthrough-create-a-pipeline-with-a-spark-activity"></a>逐步解說：建立含有 Spark 活動的管線
以下是建立含有 Spark 活動之資料處理站管線的一般步驟： 

* 建立資料處理站。
* 建立 Azure 儲存體已連結服務，以將與 HDInsight Spark 叢集關聯的儲存體連結至資料處理站。
* 建立 HDInsight 已連結服務，以將 HDInsight 中的 Spark 叢集連結至資料處理站。
* 建立會參考儲存體已連結服務的資料集。 目前，您必須指定活動的輸出資料集，即使沒有產生任何輸出。 
* 建立含有 Spark 活動的管線，此活動會參考您所建立的 HDInsight 已連結服務。 此活動已使用您在上一個步驟中建立的資料集設定為輸出資料集。 輸出資料集是驅動排程 (每小時、每天) 的因子。 因此，即使活動並不會真的產生輸出，您仍然必須指定輸出資料集。

### <a name="prerequisites"></a>必要條件
1. 依照[建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)中的指示，建立一般用途的儲存體帳戶。

1. 依照教學課程[在 HDInsight 中建立 Spark 叢集](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md)中的指示，在 HDInsight 中建立 Spark 叢集。 將您在步驟 1 中建立的儲存體帳戶與此叢集產生關聯。

1. 下載並檢閱位於 [https://adftutorialfiles.blob.core.windows.net/sparktutorial/test.py](https://adftutorialfiles.blob.core.windows.net/sparktutorial/test.py) 的 Python 指令檔 **test.py**。

1. 將 **test.py** 上傳至您 Blob 儲存體 **adfspark** 容器中的 [pyFiles] 資料夾。 建立容器和資料夾 (如果不存在)。

### <a name="create-a-data-factory"></a>建立 Data Factory
若要建立資料處理站，請依照下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 選取 [新增] > [資料 + 分析] > [資料處理站]。

1. 在 [新增資料處理站] 刀鋒視窗上的 [名稱] 底下，輸入 **SparkDF**。

   > [!IMPORTANT]
   > Azure Data Factory 的名稱在全域必須是唯一的。 如果您看到「資料處理站名稱 SparkDF 無法使用」錯誤，請變更資料處理站名稱。 例如，使用 yournameSparkDFdate，然後重新建立資料處理站。 如需有關命名規則的詳細資訊，請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md)。

1. 在 [訂用帳戶] 底下，選取您想要建立資料處理站的 Azure 訂用帳戶。

1. 選取現有的資源群組，或建立 Azure 資源群組。

1. 選取 [釘選到儀表板] 核取方塊。

1. 選取 [建立] 。

   > [!IMPORTANT]
   > 若要建立 Data Factory 執行個體，您必須是訂用帳戶/資源群組層級的 [Data Factory 參與者](../../role-based-access-control/built-in-roles.md#data-factory-contributor)角色成員。

1. 您會在 Azure 入口網站的儀表板中看到所建立的資料處理站。

1. 建立資料處理站之後，您會看到 [資料處理站] 頁面，當中會顯示資料處理站的內容。 如果看不到 [資料處理站] 頁面，請在儀表板上選取您資料處理站的圖格。

    ![Data Factory 刀鋒視窗](./media/data-factory-spark/data-factory-blade.png)

### <a name="create-linked-services"></a>建立連結的服務
在此步驟中，您會建立兩個已連結的服務。 其中一個服務會將您的 Spark 叢集連結至資料處理站，另一個服務則會將您的儲存體連結至資料處理站。 

#### <a name="create-a-storage-linked-service"></a>建立儲存體連結服務
在此步驟中，您會將儲存體帳戶連結至資料處理站。 您在本逐步解說稍後的步驟中建立的資料集會參考此連結服務。 您在下一個步驟中定義的 HDInsight 連結服務也會參考此連結服務。 

1. 在 [資料處理站] 刀鋒視窗上，選取 [編寫及部署]。 隨即會出現「Data Factory 編輯器」。

1. 選取 [新增資料存放區]，然後選擇 [Azure 儲存體]。

   ![新增資料存放區](./media/data-factory-spark/new-data-store-azure-storage-menu.png)

1. 編輯器中會出現您用來建立儲存體已連結服務的 JSON 指令碼。

   ![AzureStorageLinkedService](./media/data-factory-build-your-first-pipeline-using-editor/azure-storage-linked-service.png)

1. 將**帳戶名稱**和**帳戶金鑰**取代成您儲存體帳戶的名稱和存取金鑰。 若要了解如何取得您的儲存體存取金鑰，請參閱[管理儲存體帳戶](../../storage/common/storage-account-manage.md#access-keys)，以了解如何檢視、複製及重新產生儲存體存取金鑰。

1. 若要部署已連結的服務，請選取命令列上的 [部署]。 成功部署已連結的服務之後，[Draft-1] 視窗就會消失。 您會在左側的樹狀檢視中看到 **AzureStorageLinkedService**。

#### <a name="create-an-hdinsight-linked-service"></a>建立 HDInsight 已連結服務
在此步驟中，您會建立一個 HDInsight 已連結服務，以將 HDInsight Spark 叢集連結至資料處理站。 HDInsight 叢集是用來執行此範例管線的 Spark 活動中指定的 Spark 程式。 

1. 在「Data Factory 編輯器」中，選取 [更多] > [新增計算] > [HDInsight 叢集]。

    ![建立 HDInsight 連結服務](media/data-factory-spark/new-hdinsight-linked-service.png)

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。 在 JSON 編輯器中，執行下列步驟：

    a. 指定 HDInsight Spark 叢集的 URI。 例如： `https://<sparkclustername>.azurehdinsight.net/` 。

    b. 指定能夠存取 Spark 叢集的使用者名稱。

    c. 指定該使用者的密碼。

    d. 指定與 HDInsight Spark 叢集關聯的儲存體已連結服務。 在此範例中是 AzureStorageLinkedService。

    ```json
    {
        "name": "HDInsightLinkedService",
        "properties": {
            "type": "HDInsight",
            "typeProperties": {
                "clusterUri": "https://<sparkclustername>.azurehdinsight.net/",
                "userName": "admin",
                "password": "**********",
                "linkedServiceName": "AzureStorageLinkedService"
            }
        }
    }
    ```

    > [!IMPORTANT]
    > - Spark 活動不支援使用 Azure Data Lake Store 作為主要儲存體的 HDInsight Spark 叢集。
    > - Spark 活動僅支援現有 (您自己的) HDInsight Spark 叢集。 它不支援隨選 HDInsight 已連結服務。

    如需有關 HDInsight 已連結服務的詳細資訊，請參閱 [HDInsight 已連結服務](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)。

1. 若要部署已連結的服務，請選取命令列上的 [部署]。 

### <a name="create-the-output-dataset"></a>建立輸出資料集
輸出資料集是驅動排程 (每小時、每天) 的因子。 因此，即使管線中的 Spark 活動並不會產生任何輸出，您仍然必須為該活動指定輸出資料集。 為活動指定輸入資料集是選擇性的。

1. 在 Data Factory 編輯器中，選取 [更多] > [新增資料集] > [Azure Blob 儲存體]。

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。 JSON 程式碼片段會定義名為 **OutputDataset** 的資料集。 此外，指定將結果儲存在名為 **adfspark** 的 Blob 容器及名為 **pyFiles/output** 的資料夾中。 如先前所述，此資料集是一個虛擬資料集。 此範例中的 Spark 程式不會產生任何輸出。 **availability** 區段指定每日產生一次輸出資料集。 

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "sparkoutput.txt",
                "folderPath": "adfspark/pyFiles/output",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": "\t"
                }
            },
            "availability": {
                "frequency": "Day",
                "interval": 1
            }
        }
    }
    ```
1. 若要部署資料集，請選取命令列上的 [部署]。


### <a name="create-a-pipeline"></a>建立管線
在此步驟中，您會建立含有 HDInsightSpark 活動的管線。 目前，輸出資料集是驅動排程的因子，因此即使活動並不會產生任何輸出，您仍然必須建立輸出資料集。 如果活動沒有任何輸入，您可以略過建立輸入資料集。 因此，在此範例中不會指定任何輸入資料集。

1. 在 Data Factory 編輯器中，選取 [更多] > [新增管線]。

1. 使用下列指令碼取代 Draft-1 視窗中的指令碼：

    ```json
    {
        "name": "SparkPipeline",
        "properties": {
            "activities": [
                {
                    "type": "HDInsightSpark",
                    "typeProperties": {
                        "rootPath": "adfspark\\pyFiles",
                        "entryFilePath": "test.py",
                        "getDebugInfo": "Always"
                    },
                    "outputs": [
                        {
                            "name": "OutputDataset"
                        }
                    ],
                    "name": "MySparkActivity",
                    "linkedServiceName": "HDInsightLinkedService"
                }
            ],
            "start": "2017-02-05T00:00:00Z",
            "end": "2017-02-06T00:00:00Z"
        }
    }
    ```
    請注意下列幾點：

    a. **type** 屬性會設為 **HDInsightSpark**。

    b. **rootPath** 屬性會設定為 **adfspark\\pyFiles**，其中 adfspark 是 Blob 容器，而 pyFiles 是該容器中的檔案資料夾。 在此範例中，Blob 儲存體是與 Spark 叢集關聯的儲存體。 您可以將檔案上傳至不同的儲存體帳戶。 如果您這麼做，請建立一個儲存體已連結服務，以將該儲存體帳戶連結至資料處理站。 然後，將連結服務的名稱指定為 **sparkJobLinkedService** 屬性的值。 如需有關此屬性和 Spark 活動所支援之其他屬性的詳細資訊，請參閱 [Spark 活動屬性](#spark-activity-properties)。

    c. **entryFilePath** 會設定為 **test.py**，也就是 Python 檔案。

    d. **getDebugInfo** 屬性會設為 **Always**，表示永遠產生記錄檔 (不論成功或失敗)。

    > [!IMPORTANT]
    > 建議您在生產環境中不要將這個屬性設定為 `Always`，除非您要針對問題進行疑難排解。

    e. **outputs** 區段有一個輸出資料集。 即使 Spark 程式不會產生任何輸出，您仍然必須指定輸出資料集。 輸出資料集會驅動管線的排程 (每小時、每天)。 

    如需有關 Spark 活動所支援之屬性的詳細資訊，請參閱 [Spark 活動屬性](#spark-activity-properties)。

1. 若要部署管線，請選取命令列上的 [部署]。

### <a name="monitor-a-pipeline"></a>監視管線
1. 在 [資料處理站] 刀鋒視窗上，選取 [監視及管理] 以在另一個索引標籤中啟動監視應用程式。

    ![監視及管理圖格](media/data-factory-spark/monitor-and-manage-tile.png)

1. 將頂端的 [開始時間] 篩選變更為 **2/1/2017**，然後選取 [套用]。

1. 由於管線的開始時間 (2017-02-01) 與結束時間 (2017-02-02) 之間只有一天，因此只會出現一個活動時段。 確認資料配量的狀態為 [就緒]。

    ![監視管線](media/data-factory-spark/monitor-and-manage-app.png)

1. 在 [活動時段] 清單中，選取一個活動回合以查看其相關詳細資料。 如果發生錯誤，您會在右窗格中看到它的詳細資訊。

### <a name="verify-the-results"></a>驗證結果

1. 移至[這個網站](https://CLUSTERNAME.azurehdinsight.net/jupyter)來啟動您 HDInsight Spark 叢集的 Jupyter Notebook。 您也可以開啟 HDInsight Spark 叢集的叢集儀表板，然後啟動 Jupyter Notebook。

1. 選取 [新增] > [PySpark] 來啟動新的 Notebook。

    ![Jupyter 新筆記本](media/data-factory-spark/jupyter-new-book.png)

1. 在第二個陳述式的結尾複製並貼上文字，並按 SHIFT+ENTER 來執行下列命令：

    ```sql
    %%sql

    SELECT buildingID, (targettemp - actualtemp) AS temp_diff, date FROM hvac WHERE date = \"6/1/13\"
    ```
1. 確認您看到來自 hvac 資料表的資料。 

    ![Jupyter 查詢結果](media/data-factory-spark/jupyter-notebook-results.png)

<!-- Removed bookmark #run-a-hive-query-using-spark-sql since it doesn't exist in the target article -->如需詳細指示，請參閱[執行 Spark SQL 查詢](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md)一節。 

### <a name="troubleshooting"></a>疑難排解
由於您將 getDebugInfo 設定為 **Always**，因此您會在 Blob 容器的 [pyFiles] 資料夾中看到一個 [log] 子資料夾。 [log] 資料夾中的記錄檔會提供額外的資訊。 發生錯誤時，此記錄檔特別有用。 在生產環境中，您可以將它設定為 **Failure**。

如需進行進一步的疑難排解，請執行下列步驟：


1. 移至 `https://<CLUSTERNAME>.azurehdinsight.net/yarnui/hn/cluster`。

    ![YARN UI 應用程式](media/data-factory-spark/yarnui-application.png)

1. 選取其中一個執行嘗試的 [Logs] \(記錄\)。

    ![應用程式頁面](media/data-factory-spark/yarn-applications.png)

1. 您會在記錄頁面中看到下列額外的錯誤資訊：

    ![記錄錯誤](media/data-factory-spark/yarnui-application-error.png)

下列各節提供資料處理站實體的相關資訊，以在您的資料處理站中使用 Spark 叢集和 Spark 活動。

## <a name="spark-activity-properties"></a>Spark 活動屬性
以下是含有 Spark 活動之管線的範例 JSON 定義： 

```json
{
    "name": "SparkPipeline",
    "properties": {
        "activities": [
            {
                "type": "HDInsightSpark",
                "typeProperties": {
                    "rootPath": "adfspark\\pyFiles",
                    "entryFilePath": "test.py",
                    "arguments": [ "arg1", "arg2" ],
                    "sparkConfig": {
                        "spark.python.worker.memory": "512m"
                    }
                    "getDebugInfo": "Always"
                },
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "MySparkActivity",
                "description": "This activity invokes the Spark program",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-02-01T00:00:00Z",
        "end": "2017-02-02T00:00:00Z"
    }
}
```

下表說明 JSON 定義中所使用的 JSON 屬性。

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| name | 管線中的活動名稱。 | 是 |
| 說明 | 說明活動用途的文字。 | 否 |
| type | 這個屬性必須設為 HDInsightSpark。 | 是 |
| 預設容器 | Spark 程式執行所在的 HDInsight 連結服務名稱。 | 是 |
| rootPath | 包含 Spark 檔案的 Blob 容器和資料夾。 檔案名稱有區分大小寫。 | 是 |
| entryFilePath | Spark 程式碼/套件之根資料夾的相對路徑。 | 是 |
| className | 應用程式的 Java/Spark 主要類別。 | 否 |
| arguments | Spark 程式的命令列引數清單。 | 否 |
| proxyUser | 要模擬來執行 Spark 程式的使用者帳戶。 | 否 |
| sparkConfig | 指定 [Spark 組態：應用程式屬性](https://spark.apache.org/docs/latest/configuration.html#available-properties) \(英文\) 中所列 Spark 組態屬性的值。 | 否 |
| getDebugInfo | 指定何時將 Spark 記錄檔複製到 HDInsight 叢集所使用 (或) sparkJobLinkedService 所指定的儲存體。 允許的值為 None、Always 或 Failure。 預設值為 None。 | 否 |
| sparkJobLinkedService | 存放 Spark 作業檔案、相依性和記錄的儲存體已連結服務。 如果您未指定此屬性的值，就會使用與 HDInsight 叢集關聯的儲存體。 | 否 |

## <a name="folder-structure"></a>資料夾結構
與 Pig 和 Hive 活動不同，Spark 活動並不支援內嵌指令碼。 Spark 作業也比 Pig/Hive 作業更具擴充性。 針對 Spark 作業，您可以提供多個相依性，例如 jar 套件 (置於 java CLASSPATH 中)、Python 檔案 (置於 PYTHONPATH 中) 及任何其他檔案。

請在 HDInsight 已連結服務所參考的 Blob 儲存體中，建立下列資料夾結構。 然後，將相依檔案上傳至根資料夾中 **entryFilePath** 所代表的適當子資料夾。 例如，將 Python 檔案上傳至根資料夾的 [pyFiles] 子資料夾，將 jar 檔案上傳至 [jars] 子資料夾。 在執行階段，Data Factory 服務會預期 Blob 儲存體中有下列資料夾結構︰ 

| Path | 說明 | 必要 | 類型 |
| ---- | ----------- | -------- | ---- |
| . | Spark 作業在儲存體已連結服務中的根路徑。 | 是 | 資料夾 |
| &lt;使用者定義&gt; | 指向 Spark 作業之輸入檔案的路徑。 | 是 | 檔案 |
| ./jars | 此資料夾下的所有檔案都會上傳並放在叢集的 Java 類別路徑中。 | 否 | 資料夾 |
| ./pyFiles | 此資料夾下的所有檔案都會上傳並放在叢集的 PYTHONPATH 中。 | 否 | 資料夾 |
| ./files | 此資料夾下的所有檔案都會上傳並放在執行程式工作目錄中。 | 否 | 資料夾 |
| ./archives | 此資料夾下的所有檔案都會解壓縮。 | 否 | 資料夾 |
| ./logs | 此資料夾儲存來自 Spark 叢集的記錄。| 否 | 資料夾 |

以下是一個儲存體範例，其中在 HDInsight 已連結服務所參考的 Blob 儲存體中包含兩個 Spark 作業檔案：

```
SparkJob1
    main.jar
    files
        input1.txt
        input2.txt
    jars
        package1.jar
        package2.jar
    logs

SparkJob2
    main.py
    pyFiles
        scrip1.py
        script2.py
    logs
```
