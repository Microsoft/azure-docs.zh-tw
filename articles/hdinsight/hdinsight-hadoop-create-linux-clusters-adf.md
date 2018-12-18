---
title: '教學課程：使用 Data Factory 在 Azure HDInsight 中建立隨選 Hadoop 叢集 '
description: 了解如何使用 Azure Data Factory 在 HDInsight 中建立隨選 Handooop 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: jasonh
ms.openlocfilehash: 567bac8a12a841eed2df1467b94a2a91c86ff7b4
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666146"
---
# <a name="tutorial-create-on-demand-hadoop-clusters-in-hdinsight-using-azure-data-factory"></a>教學課程：使用 Azure Data Factory 在 HDInsight 中建立隨選 Hadoop 叢集
[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

在本文中，您會了解如何使用 Azure Data Factory，在 Azure HDInsight 中隨選建立 Hadoop 叢集。 接著，您會在 Azure Data Factory 中使用資料管線，以執行 Hive 作業並刪除該叢集。 在本教學課程結束時，您會了解如何讓巨量資料作業執行能夠運作，其中會依排程執行叢集建立、作業執行及叢集刪除。

本教學課程涵蓋下列工作： 

> [!div class="checklist"]
> * 建立 Azure 儲存體帳戶
> * 了解 Azure Data Factory 活動
> * 使用 Azure 入口網站建立資料處理站
> * 建立連結的服務
> * 建立管線
> * 觸發管線
> * 監視管線
> * 驗證輸出

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

- Azure PowerShell。 如需指示，請參閱 [安裝並設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.7.0)。

- Azure Active Directory 服務主體。 當您建立服務主體之後，請務必使用連結文章中的指示來擷取**應用程式識別碼**和**驗證金鑰**。 在本教學課程後續的內容中，您會需要這些值。 此外，請確定服務主體是訂用帳戶的「參與者」角色成員，或建立叢集所在的資源群組成員。 如需擷取所需的值並為角色指派權限的指示，請參閱[建立 Azure Active Directory 服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)。

## <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶

在本節中，您會建立儲存體帳戶，以用來作為隨選建立 HDInsight 叢集的預設儲存體。 這個儲存體帳戶也會包含範例 HiveQL 指令碼 (**hivescript.hql**)，可讓您用來模擬在叢集上執行的範例 Hive 作業。

本節會使用 Azure PowerShell 指令碼來建立儲存體帳戶，並複製儲存體帳戶內的必要檔案。 本節中的 Azure PowerShell 範例指令碼會執行下列工作：

1. 登入 Azure。
1. 建立 Azure 資源群組。
1. 建立 Azure 儲存體帳戶。
1. 在儲存體帳戶中建立 Blob 容器
1. 將下列範例 HiveQL 指令碼 (**hivescript.hql**) 複製到 Blob 容器。 您可以在 [https://hditutorialdata.blob.core.windows.net/adfv2hiveactivity/hivescripts/hivescript.hql](https://hditutorialdata.blob.core.windows.net/adfhiveactivity/script/partitionweblogs.hql) 中找到此指令碼。 此範例指令碼已可在另一個公用 Blob 容器中使用。 下列 PowerShell 指令碼會將這些檔案複製到所建立的 Azure 儲存體帳戶。


**使用 Azure PowerShell 建立儲存體帳戶並複製檔案：**
> [!IMPORTANT]
> 指定指令碼會建立之 Azure 資源群組和 Azure 儲存體帳戶的名稱。
> 記下指令碼所輸出的**資源群組名稱**、**儲存體帳戶名稱**和**儲存體帳戶金鑰**。 您在下一節中需要這些資料。

```powershell
$resourceGroupName = "<Azure Resource Group Name>"
$storageAccountName = "<Azure Storage Account Name>"
$location = "East US 2"

$sourceStorageAccountName = "hditutorialdata"  
$sourceContainerName = "adfv2hiveactivity"

$destStorageAccountName = $storageAccountName
$destContainerName = "adfgetstarted" # don't change this value.

####################################
# Connect to Azure
####################################
#region - Connect to Azure subscription
Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
Login-AzureRmAccount
#endregion

####################################
# Create a resource group, storage, and container
####################################

#region - create Azure resources
Write-Host "`nCreating resource group, storage account and blob container ..." -ForegroundColor Green

New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
New-AzureRmStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $destStorageAccountName `
    -type Standard_LRS `
    -Location $location

$destStorageAccountKey = (Get-AzureRmStorageAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $destStorageAccountName)[0].Value

$sourceContext = New-AzureStorageContext `
    -StorageAccountName $sourceStorageAccountName `
    -Anonymous
$destContext = New-AzureStorageContext `
    -StorageAccountName $destStorageAccountName `
    -StorageAccountKey $destStorageAccountKey

New-AzureStorageContainer -Name $destContainerName -Context $destContext
#endregion

####################################
# Copy files
####################################
#region - copy files
Write-Host "`nCopying files ..." -ForegroundColor Green

$blobs = Get-AzureStorageBlob `
    -Context $sourceContext `
    -Container $sourceContainerName

$blobs|Start-AzureStorageBlobCopy `
    -DestContext $destContext `
    -DestContainer $destContainerName

Write-Host "`nCopied files ..." -ForegroundColor Green
Get-AzureStorageBlob -Context $destContext -Container $destContainerName
#endregion

Write-host "`nYou will use the following values:" -ForegroundColor Green
write-host "`nResource group name: $resourceGroupName"
Write-host "Storage Account Name: $destStorageAccountName"
write-host "Storage Account Key: $destStorageAccountKey"

Write-host "`nScript completed" -ForegroundColor Green
```

**確認已建立儲存體帳戶**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取左側窗格上的 [資源群組]。
1. 按兩下您在 PowerShell 指令碼中建立的資源群組名稱。 如果列出太多的資源群組，請使用篩選器。
1. 除非您與其他專案共用資源群組，否則在 [資源] 圖格上會列出一個資源。 該資源是您先前指定名稱的儲存體帳戶。 選取儲存體帳戶名稱。
1. 選取 [Blob] 圖格。
1. 選取 [adfgetstarted] 容器。 您會看到名為 **hivescripts** 的資料夾。
1. 開啟該資料夾，並確定它包含範例指令碼檔案 **hivescript.hql**。

## <a name="understand-the-azure-data-factory-activity"></a>了解 Azure Data Factory 活動

[Azure Data Factory](../data-factory/introduction.md) 會協調並自動移動和轉換資料。 Azure Data Factory 可以適時建立 HDInsight Hadoop 叢集來處理輸入資料配量，並在處理完成時刪除叢集。 

在 Azure Data Factory 中，資料處理站可以有一或多個資料管線。 資料管線具有一或多個活動。 活動可分為兩種：

- [資料移動活動](../data-factory/copy-activity-overview.md)：您可以使用資料移動活動，將資料從來源資料存放區移到目的地資料存放區。
- [資料轉換活動](../data-factory/transform-data.md)。 使用資料轉換活動以處理/轉換資料。 HDInsight Hive 活動是 Data Factory 所支援的其中一個轉換活動。 您在本教學課程中使用 Hive 轉換活動。

在本文中，您會設定 Hive 活動，以建立隨選 HDInsight Hadoop 叢集。 當活動執行以處理資料時，即會發生下列情況：

1. 系統會適時自動建立 HDInsight Hadoop 叢集來處理配量。 

1. 會在叢集上執行 HiveQL 指令碼以處理輸入資料。 在本教學課程中，與 Hive 活動相關聯的 HiveQL 指令碼會執行下列動作︰
    
    - 使用現有的資料表 (*hivesampletable*) 來建立另一個資料表 **HiveSampleOut**。
    - 在 **HiveSampleOut** 中只填入原始資料表 *hivesampletable* 的特定資料行。
    
1. 處理完成後，會刪除 HDInsight Hadoop 叢集，且叢集在設定的一段時間會處於閒置狀態 (timeToLive 設定)。 如果下一個資料配量可用於在此 timeToLive 閒置時間中進行處理，相同的叢集會用來處理配量。  

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 在 Azure 入口網站中，選取 [建立資源] > [資料 + 分析] > [Data Factory]。

    ![入口網站上的 Azure Data Factory](./media/hdinsight-hadoop-create-linux-clusters-adf/data-factory-azure-portal.png "入口網站上的 Azure Data Factory")

1. 輸入或選取值，如下列螢幕擷取畫面所示：

    ![使用 Azure 入口網站建立 Azure Data Factory](./media/hdinsight-hadoop-create-linux-clusters-adf/create-data-factory-portal.png "使用 Azure 入口網站建立 Azure Data Factory")

    輸入或選取下列值：
    
    |屬性  |說明  |
    |---------|---------|
    |**名稱** |  輸入資料處理站的名稱。 此名稱必須是全域唯一的。|
    |**訂用帳戶**     |  選取 Azure 訂用帳戶。 |
    |**資源群組**     | 選取 [使用現有的]，然後選取您使用 PowerShell 指令碼建立的資源群組。 |
    |**版本**     | 選取 [V2 (預覽)] |
    |**位置**     | 系統會自動將位置設定為您先前在建立資源群組時所指定的位置。 針對本教學課程，位置會設定為 [美國東部 2]。 |
    

1. 選取 [釘選到儀表板]，然後選取 [建立]。 您應該會在入口網站儀表板上看到標題為**正在提交部署**的新圖格。 建立資料處理站可能需要 2 到 4 分鐘的時間。

    ![範本部署進度](./media/hdinsight-hadoop-create-linux-clusters-adf/deployment-progress-tile.png "範本部署進度") 
 
1. 建立資料處理站之後，入口網站就會顯示該資料處理站的概觀。

    ![Azure Data Factory 概觀](./media/hdinsight-hadoop-create-linux-clusters-adf/data-factory-portal-overview.png "Azure Data Factory 概觀")

1. 選取 [撰寫與監視] 以啟動 Azure Data Factory 撰寫與監視入口網站。

## <a name="create-linked-services"></a>建立連結的服務

在本節中，您會在資料處理站中撰寫兩個連結的服務。

- 將 Azure 儲存體帳戶連結至資料處理站的 **Azure 儲存體連結服務**。 隨選 HDInsight 叢集會使用此儲存體。 它也會包含在叢集上執行的 Hive 指令碼。
- **隨選 HDInsight 連結服務**。 Azure Data Factory 會自動建立 HDInsight 叢集並執行 Hive 指令碼。 然後在 HDInsight 叢集的閒置時間達到預先設定的時間後，系統就會刪除該叢集。

###  <a name="create-an-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務

1. 從 [現在就開始吧] 頁面的左側窗格中，選取 [編輯] 圖示。

    ![建立 Azure Data Factory 連結的服務](./media/hdinsight-hadoop-create-linux-clusters-adf/data-factory-edit-tab.png "建立 Azure Data Factory 連結的服務")

1. 選取視窗左下角的 [連線]，然後選取 [+新增]。

    ![在 Azure Data Factory 中建立連線](./media/hdinsight-hadoop-create-linux-clusters-adf/data-factory-create-new-connection.png "在 Azure Data Factory 中建立連線")

1. 在 [新增連結服務] 對話方塊中，選取 [Azure Blob 儲存體]，然後選取 [繼續]。

    ![建立適用於 Data Factory 的 Azure 儲存體連結服務](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-storage-linked-service.png "建立適用於 Data Factory 的 Azure 儲存體連結服務")

1. 提供儲存體連結服務的名稱、選取您在 PowerShell 指令碼中建立的 Azure 儲存體帳戶，然後選取 [完成]。

    ![提供 Azure 儲存體連結服務的名稱](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-storage-linked-service-details.png "提供 Azure 儲存體連結服務的名稱")

### <a name="create-an-on-demand-hdinsight-linked-service"></a>建立隨選 HDInsight 連結服務

1. 再次選取 [+新增] 按鈕以建立另一個連結服務。

1. 在 [新增連結服務] 視窗中，選取 [計算] > [Azure HDInsight]，然後選取 [繼續]。

    ![建立適用於 Azure Data Factory 的 HDInsight 連結服務](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-linked-service.png "建立適用於 Azure Data Factory 的 HDInsight 連結服務")

1. 在 [新增連結服務] 視窗中提供必要的值。

    ![提供 HDInsight 連結服務的值](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-linked-service-details.png "提供 HDInsight 連結服務的值")

    輸入下列值，並使其餘各項保留預設值。

    | 屬性 | 說明 |
    | --- | --- |
    | 名稱 | 輸入 HDInsight 連結服務的名稱 |
    | 類型 | 選取 [隨選 HDInsight] |
    | Azure 儲存體連結服務 | 選取您稍早建立的儲存體連結服務。 |
    | 叢集類型 | 選取 [Hadoop] |
    | 存留時間 | 提供您想要讓 HDInsight 叢集在自動刪除之前可供使用的持續時間。|
    | 服務主體識別碼 | 提供您在必要條件中建立的 Azure Active Directory 服務主體應用程式識別碼 |
    | 服務主體金鑰 | 提供 Azure Active Directory 服務主體的驗證金鑰 |
    | 叢集名稱前置詞 | 提供值以作為資料處理站所建立全部叢集類型的前置詞 |
    | 資源群組 | 選取您稍早使用 PowerShell 指令碼所建立的資源群組| 
    | 叢集 SSH 使用者名稱 | 輸入 SSH 使用者名稱 |
    | 叢集 SSH 密碼 | 提供 SSH 使用者的密碼 |

    選取 [完成]。

## <a name="create-a-pipeline"></a>建立管線

1. 選取 **+** (加號) 按鈕，然後選取 [管線]。

    ![在 Azure Data Factory 中建立管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-create-pipeline.png "在 Azure Data Factory 中建立管線")

1. 在 [活動] 工具箱中展開 [HDInsight]，並將 [Hive] 活動拖曳至管線設計工具介面。 在 [一般] 索引標籤上，提供活動的名稱。

    ![將活動新增至 Data Factory 管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-add-hive-pipeline.png "將活動新增至 Data Factory 管線")

1. 確定您已選取 Hive 活動，然後選取 [HDI 叢集] 索引標籤，並從 [HDInsight 連結服務] 下拉式清單中，選取您稍早為 HDInsight 建立的連結服務。

    ![針對管線提供 HDInsight 叢集詳細資料](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-hive-activity-select-hdinsight-linked-service.png "針對管線提供 HDInsight 叢集詳細資料")

1. 選取 [指令碼] 索引標籤並完成下列步驟：
    
    1. 針對 [指令碼連結服務]，選取 [HDIStorageLinkedService]。 這個值是您稍早建立的儲存體連結服務。
    
    1. 針對 [檔案路徑]，選取 [瀏覽儲存體]，然後瀏覽至範例 Hive 指令碼所在的位置。 如果您稍早執行過 PowerShell 指令碼，則此位置應該位於 `adfgetstarted/hivescripts/hivescript.hql`。
    
        ![針對管線提供 Hive 指令碼詳細資料](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-provide-script-path.png "針對管線提供 Hive 指令碼詳細資料")
    
    1. 在 [進階] > [參數] 下方，選取 [從指令碼自動填滿]。 此選項會在 Hive 指令碼中尋找在執行階段需要值的任何參數。 您使用的指令碼 (**hivescript.hql**) 具有**輸出**參數。 使用 `wasb://<Container>@<StorageAccount>.blob.core.windows.net/outputfolder/` 格式來提供值，以指向您 Azure 儲存體上現有的資料夾。 路徑區分大小寫。 這是將儲存指令碼輸出的路徑。
    
        ![針對 Hive 指令碼提供參數](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-provide-script-parameters.png "針對 Hive 指令碼提供參數")

1. 選取 [驗證] 來驗證管線。 選取 **>>** (右箭頭) 按鈕以關閉驗證視窗。

    ![驗證 Azure Data Factory 管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-validate-all.png "驗證 Azure Data Factory 管線")

1. 最後，選取 [全部發行]，將成品發行至 Azure Data Factory。

    ![發行 Azure Data Factory 管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-publish-pipeline.png "發行 Azure Data Factory 管線")

## <a name="trigger-a-pipeline"></a>觸發管線

1. 從設計工具介面的工具列，選取 [觸發] > [立即觸發]。

    ![觸發 Azure Data Factory 管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-trigger-pipeline.png "觸發 Azure Data Factory 管線")

1. 在快顯提要欄位中選取 [完成]。

## <a name="monitor-a-pipeline"></a>監視管線

1. 切換至左側的 [監視] 索引標籤。 您會在 [管線執行] 清單中看到管線執行。 請注意 [狀態] 資料行下的執行狀態。

    ![監視 Azure Data Factory 管線](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-monitor-pipeline.png "監視 Azure Data Factory 管線")

1. 選取 [重新整理] 可重新整理狀態。

1. 您也可以選取 [檢視活動執行] 圖示，以查看與管線相關聯的活動執行。 在下列螢幕擷取畫面中，您只會看到一個活動執行，因為您建立的管線中只有一個活動。 若要切換回上一個檢視，請選取接近頁面頂端的 [管線]。

    ![監視 Azure Data Factory 管線活動](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-monitor-pipeline-activity.png "監視 Azure Data Factory 管線活動")


## <a name="verify-the-output"></a>驗證輸出

1. 若要驗證輸出，請在 Azure 入口網站中，瀏覽至您針對本教學課程使用的儲存體帳戶。 您應該會看到下列資料夾或容器：

    - 您會看到 **adfgerstarted/outputfolder**，其中包含作為管線一部分來執行的 Hive 指令碼輸出。

    - 您會看到 **adfhdidatafactory-\<linked-service-name>-\<timestamp>** 容器。 此容器是在管線執行過程中所建立 HDInsight 叢集的預設儲存位置。

    - 您會看到 **adfjobs** 容器，其中具有 Azure Data Factory 作業記錄。  

        ![驗證 Azure Data Factory 管線輸出](./media/hdinsight-hadoop-create-linux-clusters-adf/hdinsight-data-factory-verify-output.png "驗證 Azure Data Factory 管線輸出")


## <a name="clean-up-the-tutorial"></a>清除教學課程

建立了隨選 HDInsight 叢集，就不需要明確地刪除 HDInsight 叢集。 叢集會根據您建立管線時所提供的設定而刪除。 不過，即使刪除叢集之後，與叢集相關聯的儲存體帳戶還是會繼續存在。 這是刻意設計的行為，以維持完整的資料。 不過，如果您不想保存資料，則可以刪除您所建立的儲存體帳戶。

或者，可以刪除您針對此教學課程建立的整個資源群組。 這會刪除您建立的儲存體帳戶和 Azure Data Factory。

### <a name="delete-the-resource-group"></a>刪除資源群組

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取左側窗格上的 [資源群組]。
1. 選取您在 PowerShell 指令碼中建立的資源群組名稱。 如果列出太多的資源群組，請使用篩選器。 它會開啟資源群組。
1. 除非您與其他專案共用資源群組，否則 [資源]  圖格應列出預設儲存體帳戶和 Data Factory。
1. 選取 [刪除資源群組]。 這麼做會刪除儲存體帳戶和此儲存體帳戶中儲存的資料。

    ![刪除資源群組](./media/hdinsight-hadoop-create-linux-clusters-adf/delete-resource-group.png "刪除資源群組")

1. 輸入資源群組名稱以確認刪除，然後選取 [刪除]。


## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Azure Data Factory 來建立隨選 HDInsight 叢集並執行 Hive 作業。 前往下一篇文章，以了解如何使用自訂設定來建立 HDInsight 叢集。

> [!div class="nextstepaction"]
>[使用自訂設定來建立 Azure HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)


