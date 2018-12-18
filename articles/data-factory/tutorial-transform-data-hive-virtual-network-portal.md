---
title: 在 Azure 虛擬網路中使用 Hive 轉換資料 | Microsoft Docs
description: 本教學課程提供逐步指示，說明如何使用 Azure Data Factory 中的 Hive 活動來轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/04/2018
ms.author: douglasl
ms.openlocfilehash: 60dc0e88998580732b50cb202fb5d00a7cfcae21
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106676"
---
# <a name="transform-data-in-azure-virtual-network-using-hive-activity-in-azure-data-factory"></a>在 Azure 虛擬網路中使用 Azure Data Factory 中的 Hive 活動轉換資料
在本教學課程中，您會使用 Azure 入口網站建立 Data Factory 管線，以在 Azure 虛擬網路 (VNet) 中的 HDInsight 叢集上，使用 Hive 活動來轉換資料。 您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。 
> * 建立自我裝載整合執行階段
> * 建立 Azure 儲存體和 Azure HDInsight 連結服務
> * 建立具有 Hive 活動的管線。
> * 觸發管線執行。
> * 監視管道執行 
> * 驗證輸出

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
- **Azure 儲存體帳戶**。 您會建立 hive 指令碼，並上傳至 Azure 儲存體。 Hive 指令碼的輸出會儲存在此儲存體帳戶中。 在此範例中，HDInsight 叢集會使用此 Azure 儲存體帳戶作為主要儲存體。 
- **Azure 虛擬網路。** 如果您沒有 Azure 虛擬網路，請依照[這些指示](../virtual-network/quick-create-portal.md)建立。 在此範例中，HDInsight 在 Azure 虛擬網路中。 以下是 Azure 虛擬網路的設定範例。 

    ![建立虛擬網路](media/tutorial-transform-data-using-hive-in-vnet-portal/create-virtual-network.png)
- **HDInsight 叢集。** 請遵循這篇文章來建立 HDInsight 叢集，並將它加入您在上一個步驟中建立的虛擬網路：[使用 Azure 虛擬網路延伸 Azure HDInsight](../hdinsight/hdinsight-extend-hadoop-virtual-network.md)。 以下是虛擬網路中的 HDInsight 設定範例。 

    ![虛擬網路中的 HDInsight](media/tutorial-transform-data-using-hive-in-vnet-portal/hdinsight-virtual-network-settings.png)
- **Azure PowerShell**(英文)。 遵循[如何安裝並設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示。
- **虛擬機器**。 建立 Azure 虛擬機器，並將它加入您 HDInsight 叢集所在的相同虛擬網路。 如需詳細資訊，請參閱[如何建立虛擬機器](../virtual-network/quick-create-portal.md#create-virtual-machines)。 

### <a name="upload-hive-script-to-your-blob-storage-account"></a>將 Hive 指令碼上傳至 Blob 儲存體帳戶

1. 使用下列內容建立名為 **hivescript.hql** 的 Hive SQL 檔案：

   ```sql
   DROP TABLE IF EXISTS HiveSampleOut; 
   CREATE EXTERNAL TABLE HiveSampleOut (clientid string, market string, devicemodel string, state string)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' 
   STORED AS TEXTFILE LOCATION '${hiveconf:Output}';

   INSERT OVERWRITE TABLE HiveSampleOut
   Select 
       clientid,
       market,
       devicemodel,
       state
   FROM hivesampletable
   ```
2. 在 Azure Blob 儲存體中，建立名為 **adftutorial** 的容器 (如果不存在)。
3. 建立名為 **hivescripts** 的資料夾。
4. 將 **hivescript.hql** 檔案上傳至 **hivescripts** 子資料夾。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 登入 [Azure 入口網站](https://portal.azure.com/)。    
2. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-data-factory-menu.png)
3. 在 [新增資料處理站] 頁面中，輸入 **ADFTutorialHiveFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到下列錯誤，請變更資料處理站的名稱 (例如 yournameMyAzureSsisDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name “MyAzureSsisDataFactory” is not available`
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
      若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
4. 針對 [版本] 選取 [V2]。
5. 選取 Data Factory 的 [位置]  。 清單中只會顯示資料處理站建立所支援的位置。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/tutorial-transform-data-using-hive-in-vnet-portal/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-transform-data-using-hive-in-vnet-portal/data-factory-home-page.png)
10. 按一下 [編寫與監視]，以在另一個索引標籤中啟動 Data Factory 使用者介面 (UI)。
11. 在 [開始使用] 頁面中，切換至左面板中的 [編輯] 索引標籤，如下圖所示： 

   ![編輯索引標籤](./media/tutorial-transform-data-using-hive-in-vnet-portal/get-started-page.png)

## <a name="create-a-self-hosted-integration-runtime"></a>建立自我裝載整合執行階段
由於 Hadoop 叢集在虛擬網路內，您必須在相同虛擬網路中安裝自我裝載的整合執行階段 (IR)。 在本節中，您會建立新的虛擬機器、將它加入相同的虛擬網路，並在上面安裝自我裝載的 IR。 自我裝載的 IR 讓 Data Factory 服務可以將處理要求分派給計算服務，像是虛擬網路內的 HDInsight。 也可以讓您在虛擬網路內的資料存放區和 Azure 之間移動資料。 如果資料存放區或計算是在內部部署環境中，也是使用自我裝載的 IR。 

1. 在 Azure Data Factory 使用者介面中，按一下視窗底部的 [連線]，切換至 [Integration Runtimes] 索引標籤，然後按一下工具列上的 [+ 新增] 按鈕。 

   ![新增整合執行階段功能表](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-integration-runtime-menu.png)
2. 在 [Integration Runtime 設定] 視窗中，選取 [執行資料移動，並分派活動到外部計算] 選項，然後按一下 [下一步]。 

   ![選取執行資料移動和分派活動選項](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-perform-data-movement-compute-option.png)
3. 選取 [私人網路]，然後按 [下一步]。
    
   ![選取私人網路](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-private-network.png)
4. 輸入 **MySelfHostedIR** 作為 [名稱]，然後按一下 [下一步]。 

   ![指定整合執行階段名稱](./media/tutorial-transform-data-using-hive-in-vnet-portal/integration-runtime-name.png) 
5. 按一下 [複製] 按鈕以複製整合執行階段的**驗證金鑰**，並加以儲存。 視窗保持開啟。 您需使用這個金鑰在虛擬機器註冊已安裝的 IR。 

   ![複製驗證金鑰](./media/tutorial-transform-data-using-hive-in-vnet-portal/copy-key.png)

### <a name="install-ir-on-a-virtual-machine"></a>在虛擬機器上安裝 IR

1. 在 Azure VM 上，下載[自我裝載整合執行階段](https://www.microsoft.com/download/details.aspx?id=39717)。 使用上一個步驟取得的**驗證金鑰**，手動註冊自我裝載整合執行階段。 

    ![監視整合執行階段](media/tutorial-transform-data-using-hive-in-vnet-portal/register-integration-runtime.png)

2. 自我裝載整合執行階段註冊成功時，您會看到下列訊息。 
   
    ![已成功註冊](media/tutorial-transform-data-using-hive-in-vnet-portal/registered-successfully.png)
3. 按一下 [啟動設定管理員]。 當節點已連線至雲端服務時，您會看到下列頁面： 
   
    ![節點已連線](media/tutorial-transform-data-using-hive-in-vnet-portal/node-is-connected.png)

### <a name="self-hosted-ir-in-the-azure-data-factory-ui"></a>Azure Data Factory 使用者介面中的自我裝載 IR

1. 在 **Azure Data Factory 使用者介面**中，您應該會看到自我裝載虛擬機器的名稱及其狀態。

   ![現有的自我裝載節點](./media/tutorial-transform-data-using-hive-in-vnet-portal/existing-self-hosted-nodes.png)
2. 按一下 [完成] 關閉 [整合執行階段設定] 視窗。 您會在整合執行階段的清單中看到自我裝載 IR。

   ![清單中的自我裝載 IR](./media/tutorial-transform-data-using-hive-in-vnet-portal/self-hosted-ir-in-list.png)


## <a name="create-linked-services"></a>建立連結的服務

在本節中，您會撰寫和部署兩個連結服務：
- 將 Azure 儲存體帳戶連結至資料處理站的 **Azure 儲存體連結服務**。 此儲存體是您的 HDInsight 叢集使用的主要儲存體。 在此案例中，您也會使用此 Azure 儲存體帳戶來存放 Hive 指令碼和指令碼的輸出。
- **HDInsight 連結服務**。 Azure Data Factory 會將 Hive 指令碼提交到此 HDInsight 叢集來執行。

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務

1. 切換至 [連結服務] 索引標籤，然後按一下 [ 新增]。

   ![新增連結服務按鈕](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-linked-service.png)    
2. 在 [新增連結服務] 視窗中，選取 [Azure Blob 儲存體]，然後按一下 [繼續]。 

   ![選取 Azure Blob 儲存體](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-azure-storage.png)
3. 在 [新增連結服務] 視窗中，執行下列步驟：

    1. 輸入 **AzureStorageLinkedService** 作為 [名稱]。
    2. 在 [透過整合執行階段連線] 選取 [MySelfHostedIR]。
    3. 在 [儲存體帳戶名稱] 選取您的 Azure 儲存體帳戶。 
    4. 若要測試與儲存體帳戶的連線，按一下 [測試連線]。
    5. 按一下 [檔案] 。
   
        ![指定 Azure Blob 儲存體帳戶](./media/tutorial-transform-data-using-hive-in-vnet-portal/specify-azure-storage-account.png)

### <a name="create-hdinsight-linked-service"></a>建立 HDInsight 連結服務

1. 再次按一下 [新增] 以建立另一個連結服務。 
    
   ![新增連結服務按鈕](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-linked-service.png)    
2. 切換至 [計算] 索引標籤，選取 [Azure HDInsight]，然後按一下 [繼續]。

    ![選取 Azure HDInsight](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-hdinsight.png)
3. 在 [新增連結服務] 視窗中，執行下列步驟：

    1. 輸入 **AzureHDInsightLinkedService** 作為 [名稱]。
    2. 選取 [使用您自己的 HDInsight]。 
    3. 選取您的 HDInsight 叢集作為 [Hdi 叢集]。 
    4. 輸入 HDInsight 叢集的 [使用者名稱]。
    5. 輸入使用者的 [密碼]。 
    
        ![Azure HDInsight 設定](./media/tutorial-transform-data-using-hive-in-vnet-portal/specify-azure-hdinsight.png)

本文假設您可以透過網際網路存取此叢集。 例如，您可以連線至位於 `https://clustername.azurehdinsight.net` 的叢集。 這個位址使用公用閘道，但如果您已使用網路安全性群組 (NSG) 或使用者定義的路由 (UDR) 來禁止從網際網路存取，則無法使用此位址。 為了讓 Data factory 能夠將作業提交至 Azure 虛擬網路中的 HDInsight 叢集，您需要設定 Azure 虛擬網路，使得 URL 可解析成 HDInsight 所使用之閘道的私人 IP 位址。

1. 從 Azure 入口網站，開啟 HDInsight 所在的虛擬網路。 開啟名稱開頭為 `nic-gateway-0` 的網路介面。 記下其私人 IP 位址。 例如，10.6.0.15。 
2. 如果您的 Azure 虛擬網路有 DNS 伺服器，請更新 DNS 記錄，以便 HDInsight 叢集 URL `https://<clustername>.azurehdinsight.net` 可解析成 `10.6.0.15`。 如果您的 Azure 虛擬網路中沒有 DNS 伺服器，您可以在所有已註冊為自我裝載整合執行階段節點的虛擬機器中編輯主機檔案 (C:\Windows\System32\drivers\etc)，在檔案中新增類似以下項目來暫時解決問題： 

    `10.6.0.15 myHDIClusterName.azurehdinsight.net`

## <a name="create-a-pipeline"></a>建立管線 
在此步驟中，您會建立具有 Hive 活動的新管道。 此活動會執行 Hive 指令碼，以傳回範例資料表的資料，並儲存到您定義的路徑。

請注意下列幾點：

- **scriptPath** 指向您用於 MyStorageLinkedService 的 Azure 儲存體帳戶上的 Hive 指令碼路徑。 路徑區分大小寫。
- **Output** 是 Hive 指令碼中使用的引數。 請使用 `wasb://<Container>@<StorageAccount>.blob.core.windows.net/outputfolder/` 格式，以指向您的 Azure 儲存體上現有的資料夾。 路徑區分大小寫。 

1. 在 Data Factory 使用者介面中，按一下左窗格中的 [+] \(加號)，然後按一下 [管線]。 

    ![新增管線功能表](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-pipeline-menu.png)
2. 在 [活動] 工具箱中展開 [HDInsight]，並將 [Hive] 活動拖放至管線設計工具介面。 

    ![拖放 Hive 活動](./media/tutorial-transform-data-using-hive-in-vnet-portal/drag-drop-hive-activity.png)
3. 在 [屬性] 視窗中，切換至[HDI 叢集] 索引標籤，選取 [AzureHDInsightLinkedService] 作為 [HDInsight 連結服務]。

    ![選取 HDInsight 連結服務](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-hdinsight-linked-service.png)
4. 切換至 [指令碼] 索引標籤，執行下列步驟： 

    1. 選取 [AzureStorageLinkedService] 作為 [指令碼連結服務]。 
    2. 在 [檔案路徑]，按一下 [瀏覽儲存體]。 
 
        ![瀏覽儲存體](./media/tutorial-transform-data-using-hive-in-vnet-portal/browse-storage-hive-script.png)
    3. 在 [選擇檔案或資料夾] 視窗中，瀏覽至 **adftutorial** 容器的 **hivescripts** 資料夾，選取 **hivescript.hql**，然後按一下 [完成]。  
        
        ![選擇檔案或資料夾](./media/tutorial-transform-data-using-hive-in-vnet-portal/choose-file-folder.png) 
    4. 確認您在 [檔案路徑] 看到 **adftutorial/hivescripts/hivescript.hql**。

        ![指令碼設定](./media/tutorial-transform-data-using-hive-in-vnet-portal/confirm-hive-script-settings.png)
    5. 在 [指令碼] 索引標籤中，展開 [進階] 區段。 
    6. 針對 [參數]，按一下 [從指令碼自動填滿]。 
    7. 以下列格式輸入 **Output** 參數的值：`wasb://<Blob Container>@<StorageAccount>.blob.core.windows.net/outputfolder/`。 例如： `wasb://adftutorial@mystorageaccount.blob.core.windows.net/outputfolder/` 。
 
        ![指令碼引數](./media/tutorial-transform-data-using-hive-in-vnet-portal/script-arguments.png)
1. 若要將成品發佈至 Data Factory，按一下 [發佈]。

    ![發佈](./media/tutorial-transform-data-using-hive-in-vnet-portal/publish.png)

## <a name="trigger-a-pipeline-run"></a>觸發管線執行

1. 首先，按一下工具列上的 [驗證] 按鈕驗證管線。 按一下[>>] (右箭頭) 關閉 [管線驗證輸出] 視窗。 

    ![驗證管線](./media/tutorial-transform-data-using-hive-in-vnet-portal/validate-pipeline.png) 
2. 若要觸發管線執行，按一下工具列上的 [觸發程序]，然後按一下 [立即觸發]。 

    ![立即觸發](./media/tutorial-transform-data-using-hive-in-vnet-portal/trigger-now-menu.png)

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1. 切換至左側的 [監視] 索引標籤。 您會在 [管線執行] 清單中看到管線執行。 

    ![監視管線回合](./media/tutorial-transform-data-using-hive-in-vnet-portal/monitor-pipeline-runs.png)
2. 若要重新整理清單，按一下 [重新整理]。
4. 若要檢視與此管線執行相關聯的活動執行，按一下 [動作] 資料行中的 [檢視活動執行]。 其他動作連結可停止/重新執行管線。 

    ![檢視活動執行](./media/tutorial-transform-data-using-hive-in-vnet-portal/view-activity-runs-link.png)
5. 您只會看到一個活動執行，因為 **HDInsightHive** 類型的管線中只有一個活動。 若要切換回前一個檢視，按一下頂端的 [管線] 連結。

    ![活動執行](./media/tutorial-transform-data-using-hive-in-vnet-portal/view-activity-runs.png)
6. 確認您在 **adftutorial** 容器的 **outputfolder** 中看到輸出檔案。 

    ![輸出檔案](./media/tutorial-transform-data-using-hive-in-vnet-portal/output-file.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 建立資料處理站。 
> * 建立自我裝載整合執行階段
> * 建立 Azure 儲存體和 Azure HDInsight 連結服務
> * 建立具有 Hive 活動的管線。
> * 觸發管線執行。
> * 監視管道執行 
> * 驗證輸出

進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[分支和鏈結 Data Factory 控制流程](tutorial-control-flow-portal.md)



