---
title: 建置您的第一個 Data Factory (Azure 入口網站) | Microsoft Docs
description: 在本教學課程中，您會使用 Azure 入口網站中的 Data Factory 編輯器，建立 Azure Data Factory 管線範例。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: ''
editor: ''
ms.assetid: d5b14e9e-e358-45be-943c-5297435d402d
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: f3dc27b35f35f3aaa649b03777ff13d385561673
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45732221"
---
# <a name="tutorial-build-your-first-data-factory-by-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站建置您的第一個資料處理站
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-build-your-first-pipeline.md)
> * [Azure 入口網站](data-factory-build-your-first-pipeline-using-editor.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Azure Resource Manager 範本](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> 本文適用於第 1 版 Azure Data Factory (正式運作版)。 如果您使用 Data Factory 服務的目前版本，請參閱[快速入門：使用 Data Factory 來建立資料處理站](../quickstart-create-data-factory-dot-net.md)。

在本文中，您會了解如何使用 [Azure 入口網站](https://portal.azure.com/) 來建立您的第一個資料處理站。 若要使用其他工具/SDK 進行本教學課程，請選取下拉式清單的其中一個選項。 

本教學課程中的管線有一個活動︰Azure HDInsight Hive 活動。 此活動會在 HDInsight 叢集上執行 Hive 指令碼，以轉換輸入資料來產生輸出資料。 管線已排定每個月在指定的開始與結束時間之間執行一次。 

> [!NOTE]
> 本教學課程中的資料管線會轉換輸入資料來產生輸出資料， 如需說明如何使用 Data Factory 複製資料的教學課程，請參閱[教學課程：將資料從 Azure Blob 儲存體複製到 Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
> 
> 一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱 [Data Factory 排程和執行](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

## <a name="prerequisites"></a>必要條件
請參閱[教學課程概觀](data-factory-build-your-first-pipeline.md)，並遵循＜先決條件＞一節的步驟。

本文不提供 Data Factory 服務的概念性概觀。 如需有關服務的詳細資訊，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)。  

## <a name="create-a-data-factory"></a>建立 Data Factory
資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源複製資料到目的地資料存放區。 另一個範例是，HDInsight Hive 活動會執行 Hive 指令碼來轉換輸入資料，以產生輸出資料。 

若要建立資料處理站，請依照下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 選取 [新增] > [資料 + 分析] > [資料處理站]。

   ![建立刀鋒視窗](./media/data-factory-build-your-first-pipeline-using-editor/create-blade.png)

1. 在 [新增資料處理站] 刀鋒視窗上的 [名稱] 底下，輸入 **GetStartedDF**。

   ![新增 Data Factory 刀鋒視窗](./media/data-factory-build-your-first-pipeline-using-editor/new-data-factory-blade.png)

   > [!IMPORTANT]
   > 資料處理站的名稱必須是全域唯一的名稱。 如果您收到「資料處理站名稱 GetStartedDF 無法使用」錯誤，請變更資料處理站名稱。 例如，使用 yournameGetStartedDF，然後重新建立資料處理站。 如需有關命名規則的詳細資訊，請參閱 [Data Factory：命名規則](data-factory-naming-rules.md)。
   >
   > 資料處理站的名稱未來可能會註冊為 DNS 名稱，而且可能會變成公開可見的名稱。
   >
   >
1. 在 [訂用帳戶] 底下，選取您想要建立資料處理站的 Azure 訂用帳戶。

1. 請選取現有的資源群組，或建立資源群組。 在教學課程中，建立名稱為 **ADFGetStartedRG**的資源群組。

1. 在 [位置] 下，選取資料處理站的位置。 下拉式清單中只會顯示 Data Factory 服務支援的區域。

1. 選取 [釘選到儀表板] 核取方塊。

1. 選取 [建立] 。

   > [!IMPORTANT]
   > 若要建立 Data Factory 執行個體，您必須是訂用帳戶/資源群組層級的 [Data Factory 參與者](../../role-based-access-control/built-in-roles.md#data-factory-contributor)角色成員。
   >
   >
1. 在儀表板上，您會看到 [正在部署 Data Factory] 狀態的下列圖格︰    

   ![部署 Data Factory 狀態](./media/data-factory-build-your-first-pipeline-using-editor/creating-data-factory-image.png)

1. 建立資料處理站之後，您會看到 [資料處理站] 頁面，當中會顯示資料處理站的內容。     

    ![Data Factory 刀鋒視窗](./media/data-factory-build-your-first-pipeline-using-editor/data-factory-blade.png)

在資料處理站中建立管線之前，您必須先建立一些資料處理站項目。 您會先建立連結服務，將資料存放區/計算連結至您的資料存放區。 然後，定義輸入和輸出資料集，以表示連結資料存放區中的輸入/輸出資料。 最後，您建立的管線會包含使用這些資料集的活動。

## <a name="create-linked-services"></a>建立連結的服務
在此步驟中，您會將您的 Azure 儲存體帳戶和隨選 HDInsight 叢集連結到您的資料處理站。 儲存體帳戶會保留此範例中管線的輸入和輸出資料。 HDInsight 連結服務是用來執行此範例中管線活動所指定的 Hive 指令碼。 找出您案例中使用哪些[資料存放區](data-factory-data-movement-activities.md)/[計算服務](data-factory-compute-linked-services.md)。 然後藉由建立連結服務，將這些服務連結至資料處理站。  

### <a name="create-a-storage-linked-service"></a>建立儲存體連結服務
在此步驟中，您會將儲存體帳戶連結至資料處理站。 在本教學課程中，您會使用相同的儲存體帳戶來存放輸入/輸出資料及 HQL 指令碼檔案。

1. 在 **GetStartedDF** 的 [資料處理站] 刀鋒視窗中，選取 [製作和部署]。 您會看到 [Data Factory 編輯器]。

   ![[製作和部署] 圖格](./media/data-factory-build-your-first-pipeline-using-editor/data-factory-author-deploy.png)

1. 選取 [新增資料存放區]，然後選擇 [Azure 儲存體]。

   ![[新增資料存放區] 刀鋒視窗](./media/data-factory-build-your-first-pipeline-using-editor/new-data-store-azure-storage-menu.png)

1. 在編輯器中，您會看到用來建立儲存體連結服務的 JSON 指令碼。

   ![儲存體連結服務](./media/data-factory-build-your-first-pipeline-using-editor/azure-storage-linked-service.png)

1. 將**帳戶名稱**取代成您儲存體帳戶的名稱。 將**帳戶金鑰**取代成儲存體帳戶的存取金鑰。 若要了解如何取得您的儲存體存取金鑰，請參閱[管理儲存體帳戶](../../storage/common/storage-account-manage.md#access-keys)，以了解如何檢視、複製及重新產生儲存體存取金鑰。

1. 選取命令列上的 [部署] 以部署連結服務。

    ![[部署] 按鈕](./media/data-factory-build-your-first-pipeline-using-editor/deploy-button.png)

   成功部署連結服務之後，[Draft-1] 視窗就會消失。 您會在左側的樹狀檢視中看到 **AzureStorageLinkedService**。

    ![AzureStorageLinkedService](./media/data-factory-build-your-first-pipeline-using-editor/StorageLinkedServiceInTree.png)    

### <a name="create-an-hdinsight-linked-service"></a>建立 HDInsight 連結服務
在此步驟中，您可將隨選 HDInsight 叢集連結至 Data Factory。 HDInsight 叢集會自動在執行階段上建立。 該叢集會在處理完成和閒置達到指定的時間後加以刪除。

1. 在 Data Factory 編輯器中，選取 [更多] > [新增計算] > [隨選 HDInsight 叢集]。

    ![新增計算](./media/data-factory-build-your-first-pipeline-using-editor/new-compute-menu.png)

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。 此 JSON 程式碼片段描述用來建立隨選 HDInsight 叢集的屬性。

    ```JSON
    {
        "name": "HDInsightOnDemandLinkedService",
        "properties": {
            "type": "HDInsightOnDemand",
            "typeProperties": {
                "version": "3.5",
                "clusterSize": 1,
                "timeToLive": "00:05:00",
                "osType": "Linux",
                "linkedServiceName": "AzureStorageLinkedService"
            }
        }
    }
    ```

    下表提供程式碼片段中所使用之 JSON 屬性的描述。

   | 屬性 | 說明 |
   |:--- |:--- |
   | clusterSize |指定 HDInsight 叢集的大小。 |
   | timeToLive | 指定刪除 HDInsight 叢集前的閒置時間。 |
   | 預設容器 | 指定用來儲存 HDInsight 產生之記錄檔的儲存體帳戶。 |

    請注意下列幾點：

     a. 資料處理站會使用 JSON 屬性，為您建立以 Linux 為基礎的 HDInsight 叢集。 如需詳細資訊，請參閱[隨選 HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)。

     b. 您可以使用自己的 HDInsight 叢集，不必使用隨選的 HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)。

     c. HDInsight 叢集會在您於 JSON 屬性中指定的 Blob 儲存體 (**linkedServiceName**) 建立預設容器。 HDInsight 不會在刪除叢集時刪除此容器。 這是設計的行為。 在使用 HDInsight 隨選連結服務時，除非有現有的即時叢集 (**timeToLive**)，否則每次處理配量時，就會建立 HDInsight 叢集。 此叢集會在處理完成時自動刪除。

     隨著處理的配量越來越多，您會在 Blob 儲存體中看到許多容器。 如果在對作業進行疑難排解時不需要這些容器，建議您加以刪除以降低儲存成本。 這些容器的名稱遵循下列模式："adf**yourdatafactoryname**-**linkedservicename**-datetimestamp"。 請使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具刪除 Blob 儲存體中的容器。

     如需詳細資訊，請參閱[隨選 HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)。

1. 選取命令列上的 [部署] 以部署連結服務。

    ![部署選項](./media/data-factory-build-your-first-pipeline-using-editor/ondemand-hdinsight-deploy.png)

1. 確認您有在左側的樹狀檢視中看到 **AzureStorageLinkedService** 和 **HDInsightOnDemandLinkedService**。

    ![含連結服務的樹狀檢視](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-linked-services.png)

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集來代表 Hive 處理的輸入和輸出資料。 這些資料集是您先前在本教學課程中建立的 AzureStorageLinkedService。 連結服務會指向儲存體帳戶。 針對保留輸入和輸出資料的儲存體，資料集會指定其中的容器、資料夾和檔案名稱。   

### <a name="create-the-input-dataset"></a>建立輸入資料集
1. 在 Data Factory 編輯器中，選取 [更多] > [新增資料集] > [Azure Blob 儲存體]。

    ![新增資料集](./media/data-factory-build-your-first-pipeline-using-editor/new-data-set.png)

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。 在 JSON 程式碼片段中，您會建立名為 **AzureBlobInput** 的資料集，代表管線中活動的輸入資料。 此外，您指定將輸入資料在名為 **adfgetstarted** 的 Blob 容器及名為 **inputdata** 的資料夾中。

    ```JSON
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "input.log",
                "folderPath": "adfgetstarted/inputdata",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Month",
                "interval": 1
            },
            "external": true,
            "policy": {}
        }
    }
    ```
    下表提供程式碼片段中所使用之 JSON 屬性的描述。

   | 屬性 | 說明 |
   |:--- |:--- |
   | type |類型屬性會設為 **AzureBlob**，因為資料位於 Blob 儲存體。 |
   | 預設容器 |表示您先前建立的 AzureStorageLinkedService。 |
   | folderPath | 指定包含輸入 Blob 的 Blob 容器和資料夾。 | 
   | fileName |這是選用屬性。 如果您省略此屬性，會挑選位於 folderPath 的所有檔案。 在本教學課程中，只會處理 input.log 檔案。 |
   | type |記錄檔為文字格式，因此我們會使用 **TextFormat**。 |
   | columnDelimiter |記錄檔案中的資料行會以逗號字元 (`,`) 分隔。 |
   | frequency/interval |頻率設為「**每月**」且間隔為 **1**，表示每個月都會可取得輸入配量。 |
   | external | 如果輸入資料不是由此管線產生，此屬性會設為 **true**。 在本教學課程中，input.log 檔案不是由此管線產生，因此我們會將屬性設為 **true**。 |

    如需這些 JSON 屬性的詳細資訊，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md#dataset-properties)。

1. 選取命令列的 [部署] 以部署新建立的資料集。 您會看到左側樹狀檢視中的資料集。

### <a name="create-the-output-dataset"></a>建立輸出資料集
現在，您會建立輸出資料集來代表 Blob 儲存體中儲存的輸出資料。

1. 在 Data Factory 編輯器中，選取 [更多] > [新增資料集] > [Azure Blob 儲存體]。

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。 在 JSON 程式碼片段中，建立名為 **AzureBlobOutput**的資料集，並指定由 Hive 指令碼產生的資料結構。 您也可以指定將結果儲存在名為 **adfgetstarted** 的 Blob 容器及名為 **partitioneddata** 的資料夾中。 **availability** 區段會指定每月產生一次輸出資料集。

    ```JSON
    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
          "folderPath": "adfgetstarted/partitioneddata",
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ","
          }
        },
        "availability": {
          "frequency": "Month",
          "interval": 1
        }
      }
    }
    ```
    請參閱＜建立輸入資料集＞一節，了解這些屬性的說明。 您並未在輸出資料集上設定外部屬性，因為資料集是由 Data Factory 服務產生。

1. 選取命令列的 [部署] 以部署新建立的資料集。

1. 驗證資料集已成功建立。

    ![含連結服務的樹狀檢視](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-data-set.png)

## <a name="create-a-pipeline"></a>建立管線
在此步驟中，您會建立第一個具有 HDInsightHive 活動的管線。 輸入配量為每月可用 (頻率是「月」，間隔為 1)。 每月產生輸出配量。 活動的排程器屬性也設定為每月。 輸出資料集設定及活動排程器必須相符。 目前，輸出資料集是驅動排程的因子，因此即使活動並不會產生任何輸出，您仍然必須建立輸出資料集。 如果活動沒有任何輸入，您可以略過建立輸入資料集。 本節結尾會說明下列 JSON 程式碼片段中使用的屬性。

1. 在 Data Factory 編輯器中，選取 [更多] > [新增管線]。

    ![新增管線選項](./media/data-factory-build-your-first-pipeline-using-editor/new-pipeline-button.png)

1. 複製下列程式碼片段並貼到 [Draft-1] 視窗。

   > [!IMPORTANT]
   > 在 JSON 程式碼片段中使用您的儲存體帳戶名稱取代 **storageaccountname**。
   >
   >

    ```JSON
    {
        "name": "MyFirstPipeline",
        "properties": {
            "description": "My first Azure Data Factory pipeline",
            "activities": [
                {
                    "type": "HDInsightHive",
                    "typeProperties": {
                        "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                        "scriptLinkedService": "AzureStorageLinkedService",
                        "defines": {
                            "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                            "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                        }
                    },
                    "inputs": [
                        {
                            "name": "AzureBlobInput"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobOutput"
                        }
                    ],
                    "policy": {
                        "concurrency": 1,
                        "retry": 3
                    },
                    "scheduler": {
                        "frequency": "Month",
                        "interval": 1
                    },
                    "name": "RunSampleHiveActivity",
                    "linkedServiceName": "HDInsightOnDemandLinkedService"
                }
            ],
            "start": "2017-07-01T00:00:00Z",
            "end": "2017-07-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```

    您會在 JSON 程式碼片段中建立一個管線，其中包括在 HDInsight 叢集上使用 Hive 處理「資料」的單一活動。

    Hive 指令碼檔案 (**partitionweblogs.hql**) 儲存於儲存體帳戶 (透過名為 **AzureStorageLinkedService** 的 scriptLinkedService 來指定)。 其可以在 **adfgetstarted** 容器的 **script** 資料夾中找到。

    **defines** 區段用來指定執行階段設定，該設定將傳遞到 Hive 指令碼作為 Hive 設定值。 範例包括 ${hiveconf:inputtable} 和 ${hiveconf:partitionedtable}。

    管線的 **start** 和 **end** 屬性會指定管線的作用中期間。

    在活動 JSON 中，您會指定 Hive 指令碼要在透過 **linkedServiceName**：**HDInsightOnDemandLinkedService** 指定的計算上執行。

   > [!NOTE]
   > 如需範例中使用的 JSON 屬性詳細資料，請參閱 [Data Factory 中的管線及活動](data-factory-create-pipelines.md)中的＜管線 JSON＞一節。
   >
   >
1. 確認下列項目：

   a. **input.log** 檔案存在於 Blob 儲存體中 **adfgetstarted** 容器的 **inputdata** 資料夾中。

   b. **partitionweblogs.hql** 檔案存在於 Blob 儲存體中 **adfgetstarted** 容器的 **script** 資料夾中。 如果您沒有看見這些檔案，請遵循[教學課程概觀](data-factory-build-your-first-pipeline.md) 中＜先決條件＞一節中的步驟。

   c. 在管線 JSON 中使用您的儲存體帳戶名稱取代 **storageaccountname**。

1. 選取命令列上的 [部署] 以部署管線。 由於 **start** 和 **end** 時間設定在過去，且 **isPaused** 設為 **false**，管線 (管線中的活動) 會在部署之後立即執行。

1. 確認您在樹狀檢視中看到管線。

    ![管線的樹狀檢視](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-pipeline.png)



## <a name="monitor-a-pipeline"></a>監視管線
### <a name="monitor-a-pipeline-by-using-the-diagram-view"></a>使用 [圖表] 檢視來監視管線
1. 在 [Data Factory] 刀鋒視窗中，選取 [圖表]。

    ![[圖表] 圖格](./media/data-factory-build-your-first-pipeline-using-editor/diagram-tile.png)

1. 在 [圖表] 檢視中，您會看到管線的概觀，以及在本教學課程中使用的資料集。

    ![圖表檢視](./media/data-factory-build-your-first-pipeline-using-editor/diagram-view-2.png)

1. 若要檢視管線中的所有活動，以滑鼠右鍵按一下圖表中的管線，再選取 [開啟管線]。

    ![開啟管線功能表](./media/data-factory-build-your-first-pipeline-using-editor/open-pipeline-menu.png)

1. 確認您在管線中看到了 **Hive 活動**。

    ![開啟管線檢視](./media/data-factory-build-your-first-pipeline-using-editor/open-pipeline-view.png)

    如要回到上一個檢視，請選取頂端功能表中的 [資料處理站]。

1. 在 [圖表] 檢視中，按兩下 [AzureBlobInput] 資料集。 確認配量為 [就緒] 狀態。 可能需要數分鐘的時間，配量才會顯示為 [就緒] 狀態。 如果一段時間之後還未顯示，請查看輸入檔案 (**input.log**) 是否放置在正確的容器 (**adfgetstarted**) 和資料夾 (**inputdata**) 中。

   ![輸入配量處於就緒狀態](./media/data-factory-build-your-first-pipeline-using-editor/input-slice-ready.png)

1. 關閉 [AzureBlobInput] 刀鋒視窗。

1. 在 [圖表] 檢視中，按兩下 **AzureBlobOutput** 資料集。 您會看到目前正在處理的配量。

   ![正在處理資料集](./media/data-factory-build-your-first-pipeline-using-editor/dataset-blade.png)

1. 處理完成之後，您會在 [就緒] 狀態中看到配量。

   ![處於就緒狀態的資料集](./media/data-factory-build-your-first-pipeline-using-editor/dataset-slice-ready.png)  

   > [!IMPORTANT]
   > 建立隨選 HDInsight 叢集通常需要大約 20 分鐘的時間。 管線預計需要大約 30 分鐘的時間來處理配量。
   >
   >

1. 當配量處於 [就緒] 狀態時，檢查您 Blob 儲存體中 **adfgetstarted** 容器內 **partitioneddata** 資料夾的輸出資料。  

   ![輸出資料](./media/data-factory-build-your-first-pipeline-using-editor/three-ouptut-files.png)

1. 選取配量，以在 [資料配量] 刀鋒視窗中查看其詳細資訊。

    ![資料配量資訊](./media/data-factory-build-your-first-pipeline-using-editor/data-slice-details.png)

1. 在 [活動回合] 清單中，選取一個活動回合以查看其相關詳細資訊。 (在此案例中，指的是 Hive 活動。)資訊會出現在 [活動回合詳細資料] 刀鋒視窗。   

    ![[活動回合詳細資料] 視窗](./media/data-factory-build-your-first-pipeline-using-editor/activity-window-blade.png)    

   從記錄檔中，您可以看到所執行的 Hive 查詢和狀態資訊。 這些記錄檔適合用來排解任何疑難問題。
   如需詳細資訊，請參閱[使用 Azure 入口網站刀鋒視窗監視及管理管線](data-factory-monitor-manage-pipelines.md)。

> [!IMPORTANT]
> 配量處理成功時就會刪除輸入檔案。 因此，如果您想要重新執行配量或再次進行本教學課程，請將輸入檔案 (**input.log**) 上傳至 **adfgetstarted** 容器的 **inputdata** 資料夾。
>
>

### <a name="monitor-a-pipeline-by-using-the-monitor--manage-app"></a>使用「監視及管理應用程式」來監視管線
您也可以使用「監視及管理應用程式」來監視您的管線。 如需使用此應用程式的詳細資訊，請參閱[使用監視及管理應用程式來監視及管理 Data Factory 管線](data-factory-monitor-manage-app.md)。

1. 在資料處理站首頁上選取 [監視及管理] 圖格。

    ![監視及管理圖格](./media/data-factory-build-your-first-pipeline-using-editor/monitor-and-manage-tile.png)

1. 在「監視及管理」應用程式中，變更 [開始時間] 和 [結束時間] 以符合您管線的開始和結束時間。 選取 [套用] 。

    ![監視及管理應用程式](./media/data-factory-build-your-first-pipeline-using-editor/monitor-and-manage-app.png)

1. 選取 [活動時段] 清單中的活動時段以查看其詳細資訊。

    ![活動時段清單](./media/data-factory-build-your-first-pipeline-using-editor/activity-window-details.png)

## <a name="summary"></a>總結
在本教學課程中，您已在 HDInsight Hadoop 叢集上執行 Hive 指令碼，建立了資料處理站來處理資料。 您已使用 Azure 入口網站中使用 Data Factory 編輯器來執行下列步驟︰  

* 建立資料處理站。
* 建立兩個連結服務：
   * Azure 儲存體連結服務，可將保留輸入/輸出檔案的 Blob 儲存體連結至資料處理站。
   * HDInsight 隨選連結服務，可將隨選 HDInsight Hadoop 叢集連結至資料處理站。 Data Factory 會即時建立 HDInsight Hadoop 叢集以處理輸入資料及產生輸出資料。
* 建立兩個資料集，以說明管線中 HDInsight Hive 活動的輸入和輸出資料。
* 建立具有 HDInsight Hive 活動的管線。

## <a name="next-steps"></a>後續步驟
在本文中，您已經建立可在隨選 HDInsight 叢集上執行 Hive 指令碼，且含有轉換活動 (HDInsight 活動) 的管線。 若要查看如何使用複製活動將資料從 Blob 儲存體複製到 SQL 資料庫，請參閱[教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

## <a name="see-also"></a>另請參閱
| 話題 | 說明 |
|:--- |:--- |
| [管線](data-factory-create-pipelines.md) |本文協助您了解 Data Factory 中的管線和活動，以及如何使用這些來為您的案例或業務建構端對端的資料導向工作流程。 |
| [資料集](data-factory-create-datasets.md) |本文協助您了解 Data Factory 中的資料集。 |
| [排程和執行](data-factory-scheduling-and-execution.md) |本文說明 Data Factory 應用程式模型的排程和執行層面。 |
| [使用監視及管理應用程式來監視和管理管線](data-factory-monitor-manage-app.md) |本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 |
