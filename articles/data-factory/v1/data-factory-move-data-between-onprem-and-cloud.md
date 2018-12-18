---
title: 移動資料 - 資料管理閘道 | Microsoft Docs
description: 設定資料閘道器以在內部部署與雲端之間移動資料。 使用 Azure Data Factory 中的資料管理閘道移動資料。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.assetid: 7bf6d8fd-04b5-499d-bd19-eff217aa4a9c
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: d695627efddc0ca02c3d9299f4b8a13bdc85e8fb
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42141114"
---
# <a name="move-data-between-on-premises-sources-and-the-cloud-with-data-management-gateway"></a>利用資料管理閘道在內部部署來源和雲端之間移動資料
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[使用 Data Factory 在內部部署和雲端之間複製資料](../tutorial-hybrid-copy-powershell.md)。

本文提供使用 Data Factory 整合內部部署資料存放區與雲端資料存放區資料的概觀。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文和其他 Data Factory 核心概念文章：[資料集](data-factory-create-datasets.md)和[管線](data-factory-create-pipelines.md)。

## <a name="data-management-gateway"></a>資料管理閘道
若要將資料移入/移出內部部署資料存放區，您必須在內部部署機器上安裝資料管理閘道。 您可以將閘道安裝在與資料存放區相同或相異的電腦上，只要閘道可以連接資料存放區即可。

> [!IMPORTANT]
> 如需資料管理閘道的詳細資料，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 一文。 

以下逐步解說將示範如何使用將資料從內部部署 **SQL Server** 資料庫移至 Azure Blob 儲存體的管線來建立 Data Factory。 在逐步解說中，您會在電腦上安裝及設定資料管理閘道。

## <a name="walkthrough-copy-on-premises-data-to-cloud"></a>逐步解說︰將內部部署資料複製到雲端
在本逐步解說中，您會執行下列步驟： 

1. 建立資料處理站。
2. 建立資料管理閘道。 
3. 為來源和接收的資料存放區建立已連結的服務。
4. 建立資料集來代表輸入和輸出資料。
5. 建立具有複製活動的管線來移動資料。

## <a name="prerequisites-for-the-tutorial"></a>教學課程的必要條件
開始進行本逐步解說之前，您必須具備下列必要條件：

* **Azure 訂用帳戶**。  如果您沒有訂用帳戶，則只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [免費試用](http://azure.microsoft.com/pricing/free-trial/) 一文。
* **Azure 儲存體帳戶**。 在本教學課程中，您會使用 Blob 儲存體作為**目的地/接收**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文以取得建立步驟。
* **SQL Server**。 在本教學課程中，您會使用內部部署 SQL 資料庫作為**來源**資料存放區。 

## <a name="create-data-factory"></a>建立資料處理站
在此步驟中，您將使用 Azure 入口網站來建立名為 **ADFTutorialOnPremDF**的 Azure Data Factory 執行個體。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 依序按一下 [建立資源]、[智慧 + 分析] 及 [Data Factory]。

   ![新增->DataFactory](./media/data-factory-move-data-between-onprem-and-cloud/NewDataFactoryMenu.png)  
3. 在 [新增 Data Factory] 頁面中，輸入 **ADFTutorialOnPremDF** 作為 [名稱]。

    ![新增至儀表板](./media/data-factory-move-data-between-onprem-and-cloud/OnPremNewDataFactoryAddToStartboard.png)

   > [!IMPORTANT]
   > Azure Data Factory 的名稱在全域必須是唯一的。 如果您收到錯誤： **Data Factory 名稱 “ADFTutorialOnPremDF” 無法使用**，請變更 Data Factory 名稱 (例如 yournameADFTutorialOnPremDF)，然後嘗試重新建立。 執行此教學課程中的其餘步驟時，請使用此名稱來取代 ADFTutorialOnPremDF。
   >
   > Data Factory 的名稱未來可能會註冊為 **DNS** 名稱，因此會變成公開可見的名稱。
   >
   >
4. 選取您想要建立 Data Factory 的 [Azure 訂用帳戶]  。
5. 請選取現有的 **資源群組** ，或建立資源群組。 在教學課程中，建立名稱如下的資源群組：**ADFTutorialResourceGroup**。
6. 按一下 [新增 Data Factory] 頁面上的 [建立]。

   > [!IMPORTANT]
   > 若要建立 Data Factory 執行個體，您必須是訂用帳戶/資源群組層級的 [Data Factory 參與者](../../role-based-access-control/built-in-roles.md#data-factory-contributor) 角色成員。
   >
   >
7. 建立完成之後，您會看到 [Data Factory] 頁面，如下圖所示：

   ![Data Factory 首頁](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDataFactoryHomePage.png)

## <a name="create-gateway"></a>建立閘道
1. 在 [Data Factory] 頁面中，按一下 [製作和部署] 圖格來啟動 Data Factory 的 [編輯器]。

    ![[製作和部署] 磚](./media/data-factory-move-data-between-onprem-and-cloud/author-deploy-tile.png)
2. 在 [Data Factory 編輯器] 中，按一下工具列上的 [...**更多]**，然後按一下 [新增資料閘道]。 或者，您也可以在樹狀檢視中，以滑鼠右鍵按一下 [資料閘道]，再按一下 [新增資料閘道]。

   ![工具列上的 [新增資料閘道]](./media/data-factory-move-data-between-onprem-and-cloud/NewDataGateway.png)
3. 在 [建立] 頁面上，輸入 **adftutorialgateway** 作為 [名稱]，然後按一下 [確定]。     

    ![[建立閘道] 頁面](./media/data-factory-move-data-between-onprem-and-cloud/OnPremCreateGatewayBlade.png)

    > [!NOTE]
    > 在本逐步解說中，您會建立只具有一個節點的邏輯閘道 (內部部署 Windows 機器)。 您可以將多個內部部署機器關聯到閘道以相應放大資料管理閘道。 您可以增加可在節點上同時執行的資料移動作業數目來進行相應增加。 這項功能也適用於具有單一節點的邏輯閘道。 如需得詳細資料，請參閱[在 Azure Data Factory 中調整資料管理閘道](data-factory-data-management-gateway-high-availability-scalability.md)一文。  
4. 在 [設定] 頁面中，按一下 [直接安裝在此電腦上]。 此動作會下載閘道的安裝套件、在電腦上安裝、設定和註冊閘道。  

   > [!NOTE]
   > 使用 Internet Explorer 或 Microsoft ClickOnce 相容的 Web 瀏覽器。
   >
   > 如果您使用 Chrome，請移至 [Chrome 線上應用程式商店](https://chrome.google.com/webstore/)，使用關鍵字 "ClickOnce" 進行搜尋，選擇其中一個 ClickOnce 擴充功能並安裝。
   >
   > 針對 Firefox 進行相同的操作 (安裝附加元件)。 按一下工具列上的 [開啟功能表] 按鈕 (右上角的**三條水平線**)，按一下 [附加元件]，以「ClickOnce」關鍵字進行搜尋，選擇其中一個 ClickOnce 延伸模組並安裝。    
   >
   >

    ![閘道 - [設定] 頁面](./media/data-factory-move-data-between-onprem-and-cloud/OnPremGatewayConfigureBlade.png)

    這是最簡單的方式 (一鍵)，透過單一步驟即可下載、安裝、設定和註冊閘道。 您可以看到 **Microsoft 資料管理閘道組態管理員**應用程式已安裝在電腦上。 您也可以在 **C:\Program Files\Microsoft Data Management Gateway\2.0\Shared** 資料夾中找到執行檔 **ConfigManager.exe**。

    您也可以使用此頁面中的連結手動下載與安裝閘道器，並使用 [新金鑰]  文字方塊中顯示的金鑰來加以註冊。

    如需閘道的所有詳細資料，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 一文。

   > [!NOTE]
   > 您必須是本機電腦上的系統管理員，才能成功安裝和設定「資料管理閘道」。 您可以將其他使用者加入至 [資料管理閘道使用者]  本機 Windows 群組。 此群組的成員可以使用「資料管理閘道組態管理員」工具來設定閘道器。
   >
   >
5. 等候幾分鐘，或等候直到您看見下列通知訊息︰

    ![閘道安裝成功](./media/data-factory-move-data-between-onprem-and-cloud/gateway-install-success.png)
6. 在電腦上啟動**資料管理閘道組態管理員**應用程式。 在 [搜尋] 視窗中，輸入**資料管理閘道**以存取這個公用程式。 您也可以在 **C:\Program Files\Microsoft Data Management Gateway\2.0\Shared** 資料夾中找到執行檔 **ConfigManager.exe**

    ![閘道器組態管理員](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDMGConfigurationManager.png)
7. 確認您有看到 `adftutorialgateway is connected to the cloud service` 訊息。 底部的狀態列會顯示 [已連接到雲端服務] 和一個**綠色的核取記號**。

    在 [首頁] 索引標籤上，您也可以執行下列作業︰

   * 從 Azure 入口網站中使用 [註冊] 按鈕，以金鑰**註冊**閘道。
   * **停止**在閘道電腦上執行的資料管理閘道主機服務。
   * 將更新安裝作業**排程**在一天當中的指定時間。
   * 檢視閘道的 **上次更新**時間。
   * 指定可以安裝閘道更新的時間。
8. 切換到 [設定]  索引標籤。在 [憑證]  區段中指定的憑證，可用來加密/解密在入口網站指定的內部部署資料存放區認證 (選擇性)。 按一下 [變更]  以改用您自己的憑證。 根據預設，閘道器會使用由 Data Factory 服務自動產生的憑證。

    ![閘道器憑證組態](./media/data-factory-move-data-between-onprem-and-cloud/gateway-certificate.png)

    您也可以在 [設定] 索引標籤上執行下列動作︰

   * 檢視或匯出閘道所使用的憑證。
   * 變更閘道使用的 HTTPS 端點。    
   * 設定閘道要使用的 HTTP Proxy。     
9. (選擇性) 切換到 [診斷] 索引標籤，如果您想啟用詳細資訊記錄功能，以便對閘道的任何問題進行疑難排解，請勾選 [啟用詳細資訊記錄] 選項。 在 [應用程式及服務記錄檔]  ->  [資料管理閘道] 節點之下的 [事件檢視器] 中可找到記錄資訊。

    ![[診斷] 索引標籤](./media/data-factory-move-data-between-onprem-and-cloud/diagnostics-tab.png)

    您也可以在 [診斷]  索引標籤上執行以下動作：

   * 使用 **測試連線** 一節來對使用閘道器的內部部署資料來源。
   * 按一下 [檢視記錄檔]  以查看 [事件檢視器] 視窗中的資料管理閘道記錄檔。
   * 按一下 [傳送記錄檔]  將含有過去七天記錄檔的 zip 檔案上傳到 Microsoft，以幫助針對問題進行疑難排解。
10. 在 [診斷] 索引標籤的 [測試連接] 區段中，選取 [SqlServer] 做為資料存放區的類型，輸入資料庫伺服器名稱和資料庫名稱，指定驗證類型，輸入使用者名稱和密碼，然後按一下 [測試] 來測試閘道是否可連線到資料庫。
11. 切換到網頁瀏覽器，然後在 **Azure 入口網站**中，依序在 [設定] 頁面和 [新增資料閘道] 頁面中，按一下 [確定]。
12. 左側的樹狀檢視中，[資料閘道] 下方應該會顯示 **adftutorialgateway**。  如果按一下，應該會看到相關聯的 JSON。

## <a name="create-linked-services"></a>建立連結的服務
在此步驟中，您會建立兩個連結服務：**AzureStorageLinkedService** 和 **SqlServerLinkedService**。 **SqlServerLinkedService** 會連結內部部署 SQL Server 資料庫，而 **AzureStorageLinkedService** 連結服務則會將 Azure Blob 存放區連結至 Data Factory。 稍後在本逐步解說中，您會建立可將內部部署 SQL Server 資料庫的資料複製到 Azure Blob 存放區的管線。

#### <a name="add-a-linked-service-to-an-on-premises-sql-server-database"></a>在內部部署 SQL Server 資料庫中新增連結服務
1. 在 [Data Factory 編輯器] 中，按一下工具列上的 [新增資料存放區]，然後選取 [SQL Server]。

   ![新增 SQL Server 連結服務](./media/data-factory-move-data-between-onprem-and-cloud/NewSQLServer.png)
2. 在右邊的 [JSON 編輯器] 中，執行下列步驟：

   1. 將 **gatewayName** 指定為 **adftutorialgateway**。    
   2. 在 [connectionString] 中，執行下列步驟：    

      1. 針對 **servername**，輸入裝載 SQL Server 資料庫的伺服器名稱。
      2. 針對 **databasename**，輸入資料庫的名稱。
      3. 按一下工具列上的 [加密] 按鈕。 您會看到認證管理員應用程式。

         ![認證管理員應用程式](./media/data-factory-move-data-between-onprem-and-cloud/credentials-manager-application.png)
      4. 在 [設定認證] 對話方塊中，指定驗證類型、使用者名稱和密碼，然後按一下 [確定]。 如果連線成功，加密的認證會儲存在 JSON 中，並關閉對話方塊。
      5. 請關閉啟動對話方塊的空白瀏覽器索引標籤 (如果未自動關閉)，然後返回具有 Azure 入口網站的索引標籤。

         在閘道電腦上，這些認證會以 Data Factory 服務所擁有的憑證**加密** 。 如果您想要改用與「資料管理閘道」關聯的憑證，請參閱 [安全地設定認證](#set-credentials-and-security)。    
   3. 按一下命令列上的 [部署]  ，部署 SQL Server 連結服務。 您應該會在樹狀檢視中看到連結服務。

      ![樹狀檢視中的 SQL Server 連結服務](./media/data-factory-move-data-between-onprem-and-cloud/sql-linked-service-in-tree-view.png)    

#### <a name="add-a-linked-service-for-an-azure-storage-account"></a>新增 Azure 儲存體帳戶的連結服務
1. 在 [Data Factory 編輯器] 中，按一下命令列上的 [新增資料存放區]，然後按一下 [Azure 儲存體]。
2. 在 [帳戶名稱] 中輸入您的 Azure 儲存體帳戶名稱。
3. 在 [帳戶金鑰] 中輸入您的 Azure 儲存體帳戶金鑰。
4. 按一下 [部署] 以部署 **AzureStorageLinkedService**。

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立代表複製作業的輸入和輸出資料的輸入和輸出資料集 (內部部署 SQL Server 資料庫 => Azure Blob 儲存體)。 建立資料集之前，請執行下列步驟 (詳細步驟隨附於清單後)：

* 在您新增為 Data Factory 連結服務的 SQL Server 資料庫中，建立名為 **emp** 的資料表，並在資料表中插入幾個範例項目。
* 在您加入 Data Factory 作為連結服務的 Azure Blob 儲存體帳戶中，建立名為 **adftutorial** 的 Blob 容器。

### <a name="prepare-on-premises-sql-server-for-the-tutorial"></a>準備用於教學課程的內部部署 SQL Server
1. 在您指定用於內部部署 SQL Server 連結服務 (**SqlServerLinkedService**) 的資料庫中，使用下列 SQL 指令碼在資料庫中建立 **emp** 資料表。

    ```SQL   
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50),
        CONSTRAINT PK_emp PRIMARY KEY (ID)
    )
    GO
    ```
2. 在資料表中插入一些範例：

    ```SQL
    INSERT INTO emp VALUES ('John', 'Doe')
    INSERT INTO emp VALUES ('Jane', 'Doe')
    ```

### <a name="create-input-dataset"></a>建立輸入資料集

1. 在 [Data Factory 編輯器] 中，按一下命令列上的 [...**更多]**、按一下命令列上的 [新增資料集]，然後按一下 [SQL Server 資料表]。
2. 使用下列文字取代右窗格中的 JSON：

    ```JSON   
    {        
        "name": "EmpOnPremSQLTable",
        "properties": {
            "type": "SqlServerTable",
            "linkedServiceName": "SqlServerLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "external": true,
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "externalData": {
                    "retryInterval": "00:01:00",
                    "retryTimeout": "00:10:00",
                    "maximumRetry": 3
                }
            }
        }
    }     
    ```     
   請注意下列幾點：

   * **type** 設定為 **SqlServerTable**。
   * **tableName** 設定為 **emp**。
   * **linkedServiceName** 設定為 **OnPremSqlLinkedService** (您稍早已在此逐步解說建立此連結服務)。
   * 針對並非由 Azure Data Factory 中另一個管線所產生的輸入資料集，您必須將 **external** 設定為 **true**。 它代表輸入資料產生於 Azure Data Factory 服務外部。 您可以使用 **Policy** 區段中的 **externalData** 元素，選擇性地指定任何外部資料原則。    

   如需 JSON 屬性的詳細資訊，請參閱[在 SQL Server 來回移動資料](data-factory-sqlserver-connector.md)。
3. 按一下命令列上的 [部署]  來部署資料集。  

### <a name="create-output-dataset"></a>建立輸出資料集

1. 在 [Data Factory 編輯器] 中，按一下命令列的 [新增資料集]，然後按一下 [Azure Blob 儲存體]。
2. 使用下列文字取代右窗格中的 JSON：

    ```JSON   
    {
        "name": "OutputBlobTable",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "adftutorial/outfromonpremdf",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
     }
    ```   
   請注意下列幾點：

   * **type** 設定為 **AzureBlob**。
   * **linkedServiceName** 設定為 **AzureStorageLinkedService** (您已在步驟 2 中建立此連結服務)。
   * **folderPath** 設定為 **adftutorial/outfromonpremdf**，其中 outfromonpremdf 是 adftutorial 容器中的資料夾。 建立 **adftutorial** 容器 (如果尚未存在)。
   * **availability** 設定為**每小時** (**frequency** 設為 **hour**，**interval** 設為 **1**)。  Data Factory 服務會每隔一小時在 Azure SQL Database 的 **emp** 資料表中產生輸出資料配量。

   如果您沒有指定**輸出資料表**的 **fileName**，**folderPath** 中產生的檔案會依照下列格式命名：Data<Guid>.txt (例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt)。

   若要根據 **SliceStart** 時間動態設定 **folderPath** 和 **fileName**，請使用 partitionedBy 屬性。 在下列範例中，folderPath 使用 SliceStart (所處理配量的開始時間) 中的年、月和日，fileName 使用 SliceStart 中的小時。 例如，如果配量產生於 2014-10-20T08:00:00，folderName 設定為 wikidatagateway/wikisampledataout/2014/10/20，而 fileName 設定為 08.csv。

    ```JSON
    "folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
    "fileName": "{Hour}.csv",
    "partitionedBy":
    [

        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
    ],
    ```

    如需 JSON 屬性的詳細資訊，請參閱[在 Azure Blob 儲存體來回移動資料](data-factory-azure-blob-connector.md)。
3. 按一下命令列上的 [部署]  來部署資料集。 確認您已在樹狀檢視中看到這兩個資料集。  

## <a name="create-pipeline"></a>建立管線
在此步驟中，您會建立一個**管線**，其中包含一個使用 **EmpOnPremSQLTable** 做為輸入和 **OutputBlobTable** 做為輸出的**複製活動**。

1. 在 [Data Factory 編輯器] 中，按一下 **[...更多]** 和 [新增管線]。
2. 使用下列文字取代右窗格中的 JSON：    

    ```JSON   
     {
         "name": "ADFTutorialPipelineOnPrem",
         "properties": {
         "description": "This pipeline has one Copy activity that copies data from an on-prem SQL to Azure blob",
         "activities": [
           {
             "name": "CopyFromSQLtoBlob",
             "description": "Copy data from on-prem SQL server to blob",
             "type": "Copy",
             "inputs": [
               {
                 "name": "EmpOnPremSQLTable"
               }
             ],
             "outputs": [
               {
                 "name": "OutputBlobTable"
               }
             ],
             "typeProperties": {
               "source": {
                 "type": "SqlSource",
                 "sqlReaderQuery": "select * from emp"
               },
               "sink": {
                 "type": "BlobSink"
               }
             },
             "Policy": {
               "concurrency": 1,
               "executionPriorityOrder": "NewestFirst",
               "style": "StartOfInterval",
               "retry": 0,
               "timeout": "01:00:00"
             }
           }
         ],
         "start": "2016-07-05T00:00:00Z",
         "end": "2016-07-06T00:00:00Z",
         "isPaused": false
       }
     }
    ```   
   > [!IMPORTANT]
   > 將 **start** 屬性的值替換為目前日期，並將 **end**值替換為隔天的日期。
   >
   >

   請注意下列幾點：

   * 在 activities 區段中，只會有 **type** 設定為 **Copy** 的活動。
   * 活動的**輸入**設定為 **EmpOnPremSQLTable**，活動的**輸出**則設定為 **OutputBlobTable**。
   * 在 **typeProperties** 區段中，**來源類型**指定為 **SqlSource**，**接收類型**指定為 **BlobSink**。
   * **SqlSource** 的 **sqlReaderQuery** 屬性指定為 SQL 查詢 `select * from emp`。

   開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2014-10-14T16:32:41Z。 **end** 時間為選擇性項目，但在本教學課程中會用到。

   如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9/9/9999** 做為 **end** 屬性的值。

   您定義會根據為每個 Azure Data Factory 資料集定義的 **Availability** 屬性來處理資料配量的持續時間。

   在此範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。        
3. 按一下命令列的 [部署]  ，以部署資料集 (資料表是矩形的資料集)。 確認管線顯示在樹狀檢視的**管線**節點底下。  
4. 現在，按兩下 [X] 關閉頁面，以回到 **ADFTutorialOnPremDF** 的 [Data Factory] 頁面。

**恭喜！** 您已成功建立 Azure Data Factory、連結服務、資料集和管線，以及排定的管線。

#### <a name="view-the-data-factory-in-a-diagram-view"></a>在圖表檢視中檢視 Data Factory
1. 在 **Azure 入口網站**中，按一下 [ADFTutorialOnPremDF] Data Factory 首頁上的 [圖表] 圖格。 ：

    ![圖表連結](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDiagramLink.png)
2. 您應該會看到如下圖所示的圖表：

    ![圖表檢視](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDiagramView.png)

    您可以將管線和資料集放大、縮小、放大到 100%、縮放至適當比例和自動定位，以及顯示歷程資訊 (反白顯示所選取項目的上游和下游項目)。  您可以按兩下物件 (輸入/輸出資料集或管線) 查看其屬性。

## <a name="monitor-pipeline"></a>監視管線
在此步驟中，您會使用 Azure 入口網站來監視 Azure Data Factory 的運作情形。 您也可以使用 PowerShell Cmdlet 來監視資料集和管線。 如需監視的詳細資料，請參閱 [監視和管理管線](data-factory-monitor-manage-pipelines.md)。

1. 在圖中按兩下 **EmpOnPremSQLTable**。  

    ![EmpOnPremSQLTable 配量](./media/data-factory-move-data-between-onprem-and-cloud/OnPremSQLTableSlicesBlade.png)
2. 請注意，上面的所有資料配量都是**就緒**狀態，因為管線持續期間 (開始時間到結束時間) 是過去的時間。 這也是因為您已將資料插入 SQL Server 資料庫中，而資料一直留存於其中。 確認下方的 [問題配量]  區段中沒有顯示任何配量。 若要檢視所有配量，請按一下配量清單底部的 [更多資訊]。
3. 現在，在 [資料集] 頁面中，按一下 [OutputBlobTable]。

    ![OputputBlobTable 配量](./media/data-factory-move-data-between-onprem-and-cloud/OutputBlobTableSlicesBlade.png)
4. 按一下清單中的任何資料配量，您應該會看到 [資料配量] 頁面。 您會看到該配量的活動執行。 您通常只會看到一個活動執行。  

    ![資料配量刀鋒視窗](./media/data-factory-move-data-between-onprem-and-cloud/DataSlice.png)

    若配量不是處於 [就緒] 狀態，您可以在 [未就緒的上游配量] 清單中看到未就緒且阻礙目前配量執行的上游配量。
5. 按一下底部清單中的 [活動執行]，可查看 [活動執行詳細資料]。

   ![[活動執行詳細資料] 頁面](./media/data-factory-move-data-between-onprem-and-cloud/ActivityRunDetailsBlade.png)

   您會看到輸送量、持續時間和用來傳輸資料的閘道等資訊。
6. 按一下 [X] 關閉所有頁面，直到您
7. 回到 **ADFTutorialOnPremDF** 的首頁為止。
8. (選擇性) 依序按一下 [管線] 及 [ADFTutorialOnPremDF]，然後深入檢視輸入資料表 (**已使用**) 或輸出資料集 (**已產生**)。
9. 使用 [Microsoft 儲存體總管](http://storageexplorer.com/)等工具來確認每個小時會建立一個 Blob/檔案。

   ![Azure 儲存體總管](./media/data-factory-move-data-between-onprem-and-cloud/OnPremAzureStorageExplorer.png)

## <a name="next-steps"></a>後續步驟
* 如需資料管理閘道的所有詳細資料，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 一文。
* 若要了解如何使用複製活動將資料從來源資料存放區移動到接收資料存放區，請參閱 [從 Azure Blob 複製資料到 Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) 。
