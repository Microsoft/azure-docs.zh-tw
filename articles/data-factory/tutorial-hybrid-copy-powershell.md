---
title: 使用 Azure Data Factory 將資料從 SQL Server 複製到 Blog 儲存體 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 中的自我裝載整合執行階段，將資料從內部部署資料存放區複製到 Azure 雲端。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
ms.openlocfilehash: ea5e393ebe204041d96d18481a5c64d2877755f2
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304606"
---
# <a name="tutorial-copy-data-from-an-on-premises-sql-server-database-to-azure-blob-storage"></a>教學課程：將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體
在本教學課程中，您會使用 Azure PowerShell 建立資料處理站管線，以將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體。 您要建立及使用自我裝載的整合執行階段，其會在內部部署與雲端資料存放區之間移動資料。 

> [!NOTE]
> 本文不提供 Data Factory 服務的詳細簡介。 如需詳細資訊，請參閱 [Azure Data Factory 簡介](introduction.md)。 

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立自我裝載整合執行階段。
> * 建立 SQL Server 和 Azure 儲存體連結的服務。 
> * 建立 SQL Server 和 Azure Blob 資料集。
> * 建立具有複製活動的管線來移動資料。
> * 啟動管線執行。
> * 監視管道執行。

## <a name="prerequisites"></a>先決條件
### <a name="azure-subscription"></a>Azure 訂用帳戶
開始之前，如果您還沒有 Azure 訂用帳戶，[請建立免費帳戶](https://azure.microsoft.com/free/)。

### <a name="azure-roles"></a>Azure 角色
若要建立資料處理站執行個體，您用來登入 Azure 的使用者帳戶必須具備「參與者」或「擁有者」角色，或必須是 Azure 訂用帳戶的「管理員」。 

若要檢視您在訂用帳戶中所擁有的權限，請移至 Azure 入口網站，選取右上角的使用者名稱，然後選取 [權限]。 如果您有多個訂用帳戶的存取權，請選取適當的訂用帳戶。 如需將使用者新增至角色的範例指示，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../role-based-access-control/role-assignments-portal.md)文章。

### <a name="sql-server-2014-2016-and-2017"></a>SQL Server 2014、2016 和 2017
在本教學課程中，您會使用內部部署 SQL 資料庫作為「來源」資料存放區。 您在本教學課程中建立於 Data Factory 的管線會將資料從此內部部署 SQL Server 資料庫 (來源) 複製到 Azure Blob 儲存體 (接收)。 然後在 SQL Server 資料庫中建立名為 **emp** 的資料表，並在資料表中插入幾個範例項目。 

1. 啟動 SQL Server Management Studio。 如果它尚未安裝在電腦上，請移至[下載 SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。 

1. 使用您的認證連線到 SQL Server 執行個體。 

1. 建立範例資料庫。 在樹狀檢視中，以滑鼠右鍵按一下 [資料庫]，然後選取 [新增資料庫]。 
 
1. 在 [新增資料庫] 視窗中，輸入資料庫的名稱，然後選取 [確定]。 

1. 若要建立 **emp** 資料表並在其中插入一些範例資料，請針對資料庫執行下列查詢指令碼：

   ```
       INSERT INTO emp VALUES ('John', 'Doe')
       INSERT INTO emp VALUES ('Jane', 'Doe')
       GO
   ```

1. 在樹狀檢視中，以滑鼠右鍵按一下您建立的資料庫，然後選取 [新增查詢]。

### <a name="azure-storage-account"></a>Azure 儲存體帳戶
在本教學課程中，您可以使用一般用途的 Azure 儲存體帳戶 (特別是 Blob 儲存體) 作為目的地/接收資料存放區。 如果您沒有一般用途的 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)戶。 您在本教學課程中建立於 Data Factory 的管線會將資料從內部部署 SQL Server 資料庫 (來源) 複製到此 Azure Blob 儲存體 (接收)。 

#### <a name="get-storage-account-name-and-account-key"></a>取得儲存體帳戶名稱和帳戶金鑰
您會在此教學課程中使用 Azure 儲存體帳戶的名稱和金鑰。 執行下列作業，以取得儲存體帳戶的名稱和金鑰： 

1. 使用您的使用者名稱和密碼登入 [Azure 入口網站](https://portal.azure.com)。 

1. 在左側功能表中，選取 [更多服務]，使用 **Storage** 關鍵字進行篩選，然後選取 [儲存體帳戶]。

    ![搜尋儲存體帳戶](media/tutorial-hybrid-copy-powershell/search-storage-account.png)

1. 在儲存體帳戶清單中，篩選您的儲存體帳戶 (如有需要)，然後選取您的儲存體帳戶。 

1. 在 [儲存體帳戶] 視窗中，選取 [存取金鑰]。

    ![取得儲存體帳戶名稱和金鑰](media/tutorial-hybrid-copy-powershell/storage-account-name-key.png)

1. 在 [儲存體帳戶名稱] 和 [金鑰1] 方塊中複製值，然後將它們貼到 [記事本] 或另一個編輯器中，以供稍後在教學課程中使用。 

#### <a name="create-the-adftutorial-container"></a>建立 adftutorial 容器 
在這一節中，您會在 Azure Blob 儲存體中建立一個名為 **adftutorial** 的 Blob 容器。 

1. 在 [儲存體帳戶] 視窗中，切換至 [概觀]，然後選取 [Blob]。 

    ![選取 Blob 選項](media/tutorial-hybrid-copy-powershell/select-blobs.png)

1. 在 [Blob 服務] 視窗中，選取 [容器]。 

    ![新增容器按鈕](media/tutorial-hybrid-copy-powershell/add-container-button.png)

1. 在 [新容器] 視窗的 [名稱] 方塊中，輸入 **adftutorial**，然後選取 [確定]。 

    ![輸入容器名稱](media/tutorial-hybrid-copy-powershell/new-container-dialog.png)

1. 在容器清單中選取 [adftutorial]。  

    ![選取容器](media/tutorial-hybrid-copy-powershell/seelct-adftutorial-container.png)

1. 保留 [容器] 視窗以供 **adftutorial** 開啟。 您可以在本快速入門結尾處使用它來確認輸出。 Data Factory 會在此容器中自動建立輸出資料夾，因此您不需要建立輸出資料夾。

    ![容器視窗](media/tutorial-hybrid-copy-powershell/container-page.png)

### <a name="windows-powershell"></a>Windows PowerShell

#### <a name="install-azure-powershell"></a>安裝 Azure PowerShell
如果您的電腦上沒有最新版的 Azure PowerShell，請加以安裝。 

1. 移至 [Azure SDK 下載](https://azure.microsoft.com/downloads/)。 

1. 在 [命令列工具] 的 [PowerShell] 區段中，選取 [Windows 安裝]。 

1. 若要安裝 Azure PowerShell，請執行 MSI 檔案。 

如需詳細指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。 

#### <a name="log-in-to-powershell"></a>登入 PowerShell

1. 在您的電腦上啟動 PowerShell，並透過完成本快速入門教學課程，讓它保持開啟狀態。 如果您關閉並重新開啟它，則需要再次執行這些命令。

    ![啟動 PowerShell](media/tutorial-hybrid-copy-powershell/search-powershell.png)

1. 執行下列命令，然後輸入您用來登入 Azure 入口網站的 Azure 使用者名稱和密碼：
       
    ```powershell
    Connect-AzureRmAccount
    ```        

1. 如果您有多個 Azure 訂用帳戶，請執行下列命令來選取您需要使用的訂用帳戶。 以您的 Azure 訂用帳戶識別碼取代 **SubscriptionId**：

    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<SubscriptionId>"       
    ```

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令複製到 PowerShell，指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱 (以雙引號括住；例如 `"adfrg"`)，然後執行命令。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup"
    ```

1. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    New-AzureRmResourceGroup $resourceGroupName $location
    ``` 

    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$resourceGroupName` 變數，然後執行一次命令。

1. 定義資料處理站名稱的變數，以便稍後在 PowerShell 命令中使用。 名稱必須以字母或數字開頭，並且只能包含字母、數字和虛線 (-) 字元。

    > [!IMPORTANT]
    >  將資料處理站名稱更新為全域唯一的名稱。 例如，ADFTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFTutorialFactory"
    ```

1. 定義 Data Factory 位置的變數： 

    ```powershell
    $location = "East US"
    ```  

1. 若要建立資料處理站，請執行以下 `Set-AzureRmDataFactoryV2` Cmdlet： 
    
    ```powershell       
    Set-AzureRmDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

> [!NOTE]
> 
> * 資料處理站的名稱必須是全域唯一的名稱。 如果發生下列錯誤，請變更名稱，並再試一次。
>    ```
>    The specified data factory name 'ADFv2TutorialDataFactory' is already in use. Data factory names must be globally unique.
>    ```
> * 若要建立資料處理站執行個體，您用來登入 Azure 的使用者帳戶必須具備「參與者」或「擁有者」角色，或必須是 Azure 訂用帳戶的「管理員」。
> * 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (Azure HDInsight 等) 可位於其他區域。
> 
> 

## <a name="create-a-self-hosted-integration-runtime"></a>建立自我裝載整合執行階段

在本節中，您可以建立自我裝載整合執行階段，並使用 SQL Server 資料庫將它與內部部署電腦產生關聯。 自我裝載的整合執行階段是一項元件，可將資料從您電腦上的 SQL Server 複製到 Azure blob 儲存體。 

1. 建立整合執行階段名稱的變數。 使用唯一的名稱，並記下此名稱。 您將在本教學課程稍後使用它。 

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```

1. 建立自我裝載整合執行階段。 

    ```powershell
    Set-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $integrationRuntimeName -Type SelfHosted -Description "selfhosted IR description"
    ``` 
    以下是範例輸出：

    ```json
    Id                : /subscriptions/<subscription ID>/resourceGroups/ADFTutorialResourceGroup/providers/Microsoft.DataFactory/factories/onpremdf0914/integrationruntimes/myonpremirsp0914
    Type              : SelfHosted
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Name              : myonpremirsp0914
    Description       : selfhosted IR description
    ```

1. 若要擷取所建立整合執行階段的狀態，請執行下列命令：

    ```powershell
   Get-AzureRmDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
    ```

    以下是範例輸出：
    
    ```json
    Nodes                     : {}
    CreateTime                : 9/14/2017 10:01:21 AM
    InternalChannelEncryption :
    Version                   :
    Capabilities              : {}
    ScheduledUpdateDate       :
    UpdateDelayOffset         :
    LocalTimeZoneOffset       :
    AutoUpdate                :
    ServiceUrls               : {eu.frontend.clouddatahub.net, *.servicebus.windows.net}
    ResourceGroupName         : <ResourceGroup name>
    DataFactoryName           : <DataFactory name>
    Name                      : <Integration Runtime name>
    State                     : NeedRegistration
    ```

1. 若要擷取*驗證金鑰*，用以向雲端中的 Data Factory 服務註冊自我裝載整合執行階段，請執行下列命令。 複製其中一個金鑰 (不含引號)，以註冊您在下一個步驟中安裝於電腦上的自我裝載整合執行階段。 

    ```powershell
    Get-AzureRmDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
    ```
    
    以下是範例輸出：
    
    ```json
    {
        "AuthKey1":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
        "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
    }
    ```

## <a name="install-the-integration-runtime"></a>安裝整合執行階段
1. 在本機 Windows 電腦下載 [Azure Data Factory Integration Runtime](https://www.microsoft.com/download/details.aspx?id=39717)，然後執行安裝。 

1. 在 [歡迎使用 Microsoft Integration Runtime 設定] 精靈中，選取 [下一步]。  

1. 在 [使用者授權合約] 視窗中，接受條款和授權合約，然後選取 [下一步]。 

1. 在 [目的地資料夾] 視窗中，選取 [下一步]。 

1. 在 [準備好要安裝 Microsoft Integration Runtime] 視窗中，選取 [安裝]。 

1. 如果您看到一則有關正在設定不使用時要進入睡眠或休眠模式的電腦警告訊息，請選取 [確定]。 

1. 如果顯示 [電源選項] 視窗，請將它關閉，然後切換到安裝視窗。 

1. 在 [完成 Microsoft Integration Runtime 設定] 精靈中，選取 [完成]。

1. 在 [註冊 Integration Runtime (自我裝載)] 視窗中，貼上您在上一節中儲存的金鑰，然後選取 [註冊]。 

    ![監視整合執行階段](media/tutorial-hybrid-copy-powershell/register-integration-runtime.png)

    當自我裝載的整合執行階段註冊成功時，隨即顯示下列訊息： 

    ![已成功註冊](media/tutorial-hybrid-copy-powershell/registered-successfully.png)

1. 在 [新增 Integration Runtime (自我裝載) 節點] 視窗中，選取 [下一步]。 

    ![新增 Integration Runtime 節點視窗](media/tutorial-hybrid-copy-powershell/new-integration-runtime-node-page.png)

1. 在 [內部網路通訊通道] 視窗中，選取 [略過]。  
    您可以選取 TLS/SSL 憑證，以便保護在多節點整合執行階段環境中的內部節點通訊。

    ![內部網路通訊通道視窗](media/tutorial-hybrid-copy-powershell/intranet-communication-channel-page.png)

1. 在 [註冊 Integration Runtime (自我裝載)] 視窗中，選取 [啟動組態管理員]。 

1. 當節點連線至雲端服務時，隨即顯示下列訊息：

    ![節點已連線](media/tutorial-hybrid-copy-powershell/node-is-connected.png)

1. 執行下列作業，以測試 SQL Server 資料庫的連線能力：

    ![[診斷] 索引標籤](media/tutorial-hybrid-copy-powershell/config-manager-diagnostics-tab.png)   

    a. 在 [組態管理員] 視窗中，切換到 [診斷] 索引標籤。

    b. 在 [資料來源類型] 方塊中，選取 [SqlServer]。

    c. 輸入伺服器名稱。

    d. 輸入資料庫名稱。 

    e. 選取驗證模式。 

    f. 輸入使用者名稱。 

    g. 輸入與使用者名稱相關聯的密碼。

    h. 若要確認整合執行階段可以連線到 SQL Server，請選取 [測試]。  
    如果連線成功，隨即顯示綠色的核取記號。 否則，您會收到與失敗相關聯的錯誤訊息。 修正所有問題，並確定整合執行階段可連線到您的 SQL Server 執行個體。

    請記下所有先前的值，以便稍後在本教學課程中使用。
    
## <a name="create-linked-services"></a>建立連結的服務
若要將資料存放區和計算服務連結到資料處理站，請在資料處理站中建立連結的服務。 在本教學課程中，您會將 Azure 儲存體帳戶和內部部署 SQL Server 執行個體連結到資料存放區。 連結的服務具有連線資訊，可供 Data Factory 服務在執行階段中用來連線。 

### <a name="create-an-azure-storage-linked-service-destinationsink"></a>建立 Azure 儲存體連結服務 (目的地/接收)
在此步驟中，您會將您的 Azure 儲存體帳戶連結到資料處理站。

1. 使用下列程式碼，在 *C:\ADFv2Tutorial* 資料夾中建立名為 AzureStorageLinkedService.json 的 JSON 檔案。 如果 ADFv2Tutorial 資料夾不存在，請加以建立。  

    > [!IMPORTANT]
    > 儲存檔案前，以您的 Azure 儲存體帳戶的名稱和金鑰取代 \<accountName> 和 \<accountKey>。 您已在[必要條件](#get-storage-account-name-and-account-key)一節中記下這些資料。

   ```json
    {
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>;EndpointSuffix=core.windows.net"
                }
            }
        },
        "name": "AzureStorageLinkedService"
    }
   ```

1. 在 PowerShell 中，切換至 C:\ADFv2Tutorial 資料夾。

1. 若要建立 AzureStorageLinkedService 連結服務，請執行下列 `Set-AzureRmDataFactoryV2LinkedService` Cmdlet： 

   ```powershell
   Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
   ```

   以下是範例輸出：

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

    如果您收到「找不到檔案」錯誤，請執行 `dir` 命令來確認檔案存在。 如果檔案名稱具有 .txt 副檔名 (例如，AzureStorageLinkedService.json.txt)，請將它移除，然後再次執行 PowerShell 命令。 

### <a name="create-and-encrypt-a-sql-server-linked-service-source"></a>建立及加密 SQL Server 連結服務 (來源)
在此步驟中，您要將內部部署 SQL Server 執行個體連結至資料處理站。

1. 使用下列程式碼，在 C:\ADFv2Tutorial 資料夾中建立名為 SqlServerLinkedService.json 的 JSON 檔案：

    > [!IMPORTANT]
    > 選取以您用於連線到 SQL Server 的驗證為基礎的區段。

    **使用 SQL 驗證 (sa)：**

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<servername>;Database=<databasename>;User ID=<username>;Password=<password>;Timeout=60"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }
   ```    

    **使用 Windows 驗證：**

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<server>;Database=<database>;Integrated Security=True"
                },
                "userName": "<user> or <domain>\\<user>",
                "password": {
                    "type": "SecureString",
                    "value": "<password>"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }    
    ```

    > [!IMPORTANT]
    > - 選取以您用於連線到 SQL Server 執行個體的驗證為基礎的區段。
    > - 以您的整合執行階段名稱取代 **\<整合執行階段名稱>**。
    > - 儲存檔案之前，以您的 SQL Server 執行個體值取代 **\<servername>**、**\<databasename>**、**\<username>** 和 **\<password>**。
    > - 如果您需要在使用者帳戶或伺服器名稱中使用反斜線 (\\)，請在它的前面加上逸出字元 (\\)。 例如，使用 mydomain\\\\myuser。 

1. 若要將敏感性資料加密 (使用者名稱、密碼等)，請執行 `New-AzureRmDataFactoryV2LinkedServiceEncryptedCredential` Cmdlet。  
    此加密可確保使用資料保護應用程式開發介面 (DPAPI) 來加密認證。 已加密的認證會本機儲存在自我裝載的整合執行階段節點 (本機電腦) 上。 輸出承載可以重新導向至另一個包含加密認證的 JSON 檔案 (在此例中是 encryptedLinkedService.json)。
    
   ```powershell
   New-AzureRmDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -IntegrationRuntimeName $integrationRuntimeName -File ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
   ```

1. 執行下列命令，以建立 EncryptedSqlServerLinkedService：

   ```powershell
   Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -File ".\encryptedSqlServerLinkedService.json"
   ```


## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立輸入和輸出資料集。 其代表複製作業的輸入和輸出資料，而複製作業會將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體。

### <a name="create-a-dataset-for-the-source-sql-server-database"></a>建立來源 SQL Server 資料庫的資料集
在此步驟中，您要定義代表 SQL Server 資料庫執行個體中資料的資料集。 資料集的類型為 SqlServerTable。 它會參考您在前一個步驟中建立的 SQL Server 連結服務。 此連結服務具有連線資訊，可供 Data Factory 服務在執行階段用來連線到您的 SQL Server 執行個體。 此資料集會指定包含資料的資料庫中的 SQL 資料表。 在本教學課程中，**emp** 資料表包含來源資料。 

1. 使用下列程式碼，在 C:\ADFv2Tutorial 資料夾中建立名為 SqlServerDataset.json 的 JSON 檔案：  

    ```json
    {
       "properties": {
            "type": "SqlServerTable",
            "typeProperties": {
                "tableName": "dbo.emp"
            },
            "structure": [
                 {
                    "name": "ID",
                    "type": "String"
                },
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "linkedServiceName": {
                "referenceName": "EncryptedSqlServerLinkedService",
                "type": "LinkedServiceReference"
            }
        },
        "name": "SqlServerDataset"
    }
    ```

1. 若要建立 SqlServerDataset 資料集，請執行 `Set-AzureRmDataFactoryV2Dataset` Cmdlet。

    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerDataset" -File ".\SqlServerDataset.json"
    ```

    以下是範例輸出：

    ```json
    DatasetName       : SqlServerDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Structure         : {"name": "ID" "type": "String", "name": "FirstName" "type": "String", "name": "LastName" "type": "String"}
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-dataset-for-azure-blob-storage-sink"></a>建立 Azure Blob 儲存體 (接收) 的資料集
在此步驟中，您會定義的資料集代表要複製到 Azure Blob 儲存體的資料。 此資料集的類型為 AzureBlob。 它是指您稍早在本教學課程中建立的 Azure 儲存體連結服務。 

此連結服務具有連線資訊，可供資料處理站在執行階段用來連線到您的 Azure 儲存體帳戶。 這個資料集會指定 Azure 儲存體中的資料夾，而資料會從 SQL Server 資料庫複製到這個資料夾。 在本教學課程中，資料夾是 adftutorial/fromonprem，其中 `adftutorial` 是 blob 容器，而 `fromonprem` 是資料夾。 

1. 使用下列程式碼，在 C:\ADFv2Tutorial 資料夾中建立名為 AzureBlobDataset.json 的 JSON 檔案：

    ```json
    {
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/fromonprem",
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        },
        "name": "AzureBlobDataset"
    }
    ```

1. 若要建立 AzureBlobDataset 資料集，請執行 `Set-AzureRmDataFactoryV2Dataset` Cmdlet。

    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureBlobDataset" -File ".\AzureBlobDataset.json"
    ```

    以下是範例輸出：

    ```json
    DatasetName       : AzureBlobDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

## <a name="create-a-pipeline"></a>建立管線
在本教學課程中，您會建立具有複製活動的管線。 複製活動會使用 SqlServerDataset 作為輸入資料集，而使用 AzureBlobDataset 作為輸出資料集。 來源類型設為 SqlSource，而接收類型設為 BlobSink。

1. 使用下列程式碼，在 C:\ADFv2Tutorial 資料夾中建立名為 SqlServerToBlobPipeline.json 的 JSON 檔案：

    ```json
    {
       "name": "SQLServerToBlobPipeline",
        "properties": {
            "activities": [       
                {
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource"
                        },
                        "sink": {
                            "type":"BlobSink"
                        }
                    },
                    "name": "CopySqlServerToAzureBlobActivity",
                    "inputs": [
                        {
                            "referenceName": "SqlServerDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "AzureBlobDataset",
                            "type": "DatasetReference"
                        }
                    ]
                }
            ]
        }
    }
    ```

1. 若要建立 SqlServerToBlobPipeline 管道，請執行 `Set-AzureRmDataFactoryV2Pipeline` Cmdlet。

    ```powershell
    Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SQLServerToBlobPipeline" -File ".\SQLServerToBlobPipeline.json"
    ```

    以下是範例輸出：

    ```json
    PipelineName      : SQLServerToBlobPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Activities        : {CopySqlServerToAzureBlobActivity}
    Parameters        :  
    ```

## <a name="create-a-pipeline-run"></a>建立管線執行
啟動 SQLServerToBlobPipeline 管道的管道執行，並擷取管道執行識別碼，方便後續監視。

```powershell
$runId = Invoke-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'SQLServerToBlobPipeline'
```

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1. 若要持續檢查 SQLServerToBlobPipeline 管道的執行狀態，請在 PowerShell 中執行下列指令碼，並印出最終結果：

    ```powershell
    while ($True) {
        $result = Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

        if (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
            Start-Sleep -Seconds 30
        }
        else {
            Write-Host "Pipeline 'SQLServerToBlobPipeline' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
    }
    ```

    執行範例的輸出如下：

    ```jdon
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : copy
    PipelineRunId     : 4ec8980c-62f6-466f-92fa-e69b10f33640
    PipelineName      : SQLServerToBlobPipeline
    Input             :  
    Output            :  
    LinkedServiceName :
    ActivityRunStart  : 9/13/2017 1:35:22 PM
    ActivityRunEnd    : 9/13/2017 1:35:42 PM
    DurationInMs      : 20824
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    ```

1. 您可以取得 SQLServerToBlobPipeline管線 的執行識別碼，並執行下列命令來檢查詳細的活動執行結果： 

    ```powershell
    Write-Host "Pipeline 'SQLServerToBlobPipeline' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "CopySqlServerToAzureBlobActivity"}).Output.ToString()
    ```

    執行範例的輸出如下：

    ```json
    {
      "dataRead": 36,
      "dataWritten": 24,
      "rowsCopied": 2,
      "copyDuration": 3,
      "throughput": 0.01171875,
      "errors": [],
      "effectiveIntegrationRuntime": "MyIntegrationRuntime",
      "billedDuration": 3
    }
    ```

## <a name="verify-the-output"></a>驗證輸出
管道會自動在 `adftutorial` Blob 容器中建立名為 fromonprem 的輸出資料夾。 確認您在輸出資料夾中看到 *dbo.emp.txt* 檔案。 

1. 在 Azure 入口網站的 [adftutorial] 容器視窗中，選取 [重新整理] 可查看輸出資料夾。

    ![已建立輸出資料夾](media/tutorial-hybrid-copy-powershell/fromonprem-folder.png)
1. 在資料夾清單中選取 `fromonprem`。 
1. 確認您看到名為 `dbo.emp.txt` 的檔案。

    ![輸出檔案](media/tutorial-hybrid-copy-powershell/fromonprem-file.png)


## <a name="next-steps"></a>後續步驟
此範例中的管線會在 Azure Blob 儲存體中的不同位置之間複製資料。 您已了解如何︰

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立自我裝載整合執行階段。
> * 建立 SQL Server 和 Azure 儲存體連結的服務。 
> * 建立 SQL Server 和 Azure Blob 資料集。
> * 建立具有複製活動的管線來移動資料。
> * 啟動管線執行。
> * 監視管道執行。

如需 Data Factory 所支援的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。

若要了解如何將大量資料從來源複製到目的地，請繼續進行以下教學課程：

> [!div class="nextstepaction"]
>[複製大量資料](tutorial-bulk-copy.md)
