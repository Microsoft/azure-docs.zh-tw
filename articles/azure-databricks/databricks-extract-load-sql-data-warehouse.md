---
title: 教學課程：使用 Azure Databricks 執行 ETL 作業
description: 了解如何將資料從 Data Lake Store 擷取至 Azure Databricks 並轉換資料，然後將資料載入 Azure SQL 資料倉儲。
services: azure-databricks
author: nitinme
ms.author: nitinme
manager: cgronlun
editor: cgronlun
ms.service: azure-databricks
ms.custom: mvc
ms.topic: tutorial
ms.workload: Active
ms.date: 07/26/2018
ms.openlocfilehash: c416937f98f6bcb49f86fce18213ca4ed349c513
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902054"
---
# <a name="tutorial-extract-transform-and-load-data-using-azure-databricks"></a>教學課程：使用 Azure Databrick 擷取、轉換和載入資料

在此教學課程中，您將使用 Azure Databricks 執行 ETL (擷取、轉換及載入資料) 作業。 您會將資料從 Azure Data Lake Store 擷取至 Azure Databricks，並在 Azure Databricks 中執行資料轉換，然後將轉換後的資料載入 Azure SQL 資料倉儲。 

本教學課程中的步驟會使用適用於 Azure Databricks 的 SQL 資料倉儲連接器，將資料轉送至 Azure Databricks。 接著，當資料在 Azure Databricks 叢集和 Azure SQL 資料倉儲之間進行轉換時，此連接器會使用 Azure Blob 儲存體作為暫時儲存體。

下圖顯示此應用程式流程：

![搭配 Data Lake Store 與 SQL 資料倉儲的 Azure Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-extract-transform-load-sql-datawarehouse.png "搭配 Data Lake Store 與 SQL 資料倉儲的 Azure Databricks")

本教學課程涵蓋下列工作： 

> [!div class="checklist"]
> * 建立 Azure Databricks 工作區
> * 在 Azure Databricks 中建立 Spark 叢集
> * 建立 Azure Data Lake Store 帳戶
> * 將資料上傳至 Azure Data Lake Store
> * 在 Azure Databricks 中建立 Notebook
> * 從 Data Lake Store 擷取資料
> * 轉換 Azure Databricks 中的資料
> * 將資料載入 Azure SQL 資料倉儲

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

開始本教學課程之前，請確定您符合下列需求：
- 建立 Azure SQL 資料倉儲、建立伺服器層級的防火牆規則，並以伺服器管理員的身分連線至伺服器。請依照[快速入門：建立 Azure SQL 資料倉儲](../sql-data-warehouse/create-data-warehouse-portal.md)中的指示操作
- 建立 Azure SQL 資料倉儲的資料庫主要金鑰。 請依照[建立資料庫主要金鑰](https://docs.microsoft.com/sql/relational-databases/security/encryption/create-a-database-master-key)中的指示操作。
- 建立 Azure Blob 儲存體帳戶和其中所含的容器。 此外，擷取用來存取儲存體帳戶的存取金鑰。 請依照[快速入門：建立 Azure Blob 儲存體帳戶](../storage/blobs/storage-quickstart-blobs-portal.md)中的指示操作。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-an-azure-databricks-workspace"></a>建立 Azure Databricks 工作區

在本節中，您會使用 Azure 入口網站建立 Azure Databricks 工作區。 

1. 在 Azure 入口網站中，選取 [建立資源] > [資料 + 分析] > [Azure Databricks]。

    ![Azure 入口網站上的 Databricks](./media/databricks-extract-load-sql-data-warehouse/azure-databricks-on-portal.png "Azure 入口網站上的 Databricks")

3. 在 [Azure Databricks 服務] 底下，提供值以建立 Databricks 工作區。

    ![建立 Azure Databricks 工作區](./media/databricks-extract-load-sql-data-warehouse/create-databricks-workspace.png "建立 Azure Databricks 工作區")

    提供下列值： 
     
    |屬性  |說明  |
    |---------|---------|
    |**工作區名稱**     | 提供您 Databricks 工作區的名稱        |
    |**訂用帳戶**     | 從下拉式清單中選取您的 Azure 訂用帳戶。        |
    |**資源群組**     | 指定您是要建立新的資源群組，還是使用現有資源群組。 資源群組是存放 Azure 方案相關資源的容器。 如需詳細資訊，請參閱 [Azure 資源群組概觀](../azure-resource-manager/resource-group-overview.md)。 |
    |**位置**     | 選取 [美國東部 2]。 如需其他可用的區域，請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/services/)。        |
    |定價層     |  選擇 [標準] 或 [進階]。 如需這些定價層的詳細資訊，請參閱 [Databricks 定價頁面](https://azure.microsoft.com/pricing/details/databricks/)。       |

    選取 [釘選到儀表板]，然後選取 [建立]。

4. 建立帳戶需要幾分鐘的時間。 建立帳戶期間，入口網站右側會顯示 [提交 Azure Databricks 部署] 圖格。 您可能需要在儀表板上向右捲動以查看此圖格。 另外在畫面頂端附近還會顯示一個進度列。 您可以查看任何進度區域。

    ![Databricks 部署圖格](./media/databricks-extract-load-sql-data-warehouse/databricks-deployment-tile.png "Databricks 部署圖格")

## <a name="create-a-spark-cluster-in-databricks"></a>在 Databricks 中建立 Spark 叢集

1. 在 Azure 入口網站中，移至您所建立的 Databricks 工作區，然後選取 [啟動工作區]。

2. 系統會將您重新導向至 Azure Databricks 入口網站。 在入口網站中選取 [叢集]。

    ![Azure 上的 Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-on-azure.png "Azure 上的 Databricks")

3. 在 [新增叢集] 頁面上，提供值以建立叢集。

    ![在 Azure 上建立 Databricks Spark 叢集](./media/databricks-extract-load-sql-data-warehouse/create-databricks-spark-cluster.png "在 Azure 上建立 Databricks Spark 叢集")

    接受下列值以外的所有其他預設值：

    * 輸入叢集的名稱。
    * 針對本文，使用 **4.0** 執行階段建立叢集。 
    * 請確定您選取 **在活動 ___ 分鐘後終止** 核取方塊。 請提供用來終止叢集的叢集未使用持續時間 (以分鐘為單位)。
    
    選取 [建立叢集]。 叢集在執行後，您就可以將 Notebook 連結至叢集，並執行 Spark 作業。

## <a name="create-an-azure-data-lake-store-account"></a>建立 Azure Data Lake Store 帳戶

在本節中，您將會建立 Azure Data Lake Store 帳戶，並讓 Azure Active Directory 服務主體與其產生關聯。 稍後在本教學課程中，您會在 Azure Databricks 中使用此服務主體來存取 Azure Data Lake Store。 

1. 從 [Azure 入口網站](https://portal.azure.com)，選取 [建立資源] > [儲存體] > [Data Lake Store]。
3. 在 [新增 Data Lake Store] 刀鋒視窗中，提供如以下螢幕擷取畫面所示的值：
   
    ![建立新的 Azure Data Lake Store 帳戶](./media/databricks-extract-load-sql-data-warehouse/create-new-datalake-store.png "建立新的 Azure Data Lake 帳戶")

    提供下列值： 
     
    |屬性  |說明  |
    |---------|---------|
    |**名稱**     | 輸入 Data Lake Store 帳戶的唯一名稱。        |
    |**訂用帳戶**     | 從下拉式清單中選取您的 Azure 訂用帳戶。        |
    |**資源群組**     | 針對此教學課程，請選取建立 Azure Databricks 工作區時所使用的相同資源群組。  |
    |**位置**     | 選取 [美國東部 2]。  |
    |**定價套件**     |  請選取**隨用隨付**。 |
    | **加密設定** | 請保留預設設定值。 |

    選取 [釘選到儀表板]，然後選取 [建立]。

您現在將建立 Azure Active Directory 服務主體，並與您建立的 Data Lake Store 產生關聯。

### <a name="create-an-azure-active-directory-service-principal"></a>建立 Azure Active Directory 服務主體
   
1. 從 [Azure 入口網站](https://portal.azure.com)，選取 [所有服務]，然後搜尋 **Azure Active Directory**。

2. 選取 [應用程式註冊]。

   ![select app registrations](./media/databricks-extract-load-sql-data-warehouse/select-app-registrations.png)

3. 選取 [新增應用程式註冊]。

   ![新增應用程式](./media/databricks-extract-load-sql-data-warehouse/select-add-app.png)

4. 提供應用程式的名稱和 URL。 針對您想要建立的應用程式類型，選取 [Web 應用程式/API]。 提供登入 URL，然後選取 [建立]。

   ![名稱應用程式](./media/databricks-extract-load-sql-data-warehouse/create-app.png)

若要從 Azure Databricks 存取 Data Lake Store 帳戶，您必須具有所建立 Azure Active Directory 服務主體的下列值：
- 應用程式識別碼
- 驗證金鑰
- 租用戶識別碼

在下列章節中，您將針對稍早建立的 Azure Active Directory 服務主體擷取這些值。

### <a name="get-application-id-and-authentication-key-for-the-service-principal"></a>取得服務主體的應用程式識別碼和驗證金鑰

以程式設計方式登入時，您需要應用程式識別碼和驗證金鑰。 若要取得這些值，請使用下列步驟︰

1. 在 Azure Active Directory 中，從 [應用程式註冊]選取您的應用程式。

   ![選取應用程式](./media/databricks-extract-load-sql-data-warehouse/select-app.png)

2. 複製 [應用程式識別碼] 並儲存在您的應用程式碼中。 某些[應用程式範例](#log-in-as-the-application)會將這個值視為用戶端識別碼。

   ![用戶端識別碼](./media/databricks-extract-load-sql-data-warehouse/copy-app-id.png)

3. 若要產生驗證金鑰，請選取 [設定]。

   ![選取設定](./media/databricks-extract-load-sql-data-warehouse/select-settings.png)

4. 若要產生驗證金鑰，請選取 [金鑰]。

   ![選取金鑰](./media/databricks-extract-load-sql-data-warehouse/select-keys.png)

5. 提供金鑰的描述和金鑰的持續時間。 完成時，選取 [儲存]。

   ![儲存金鑰](./media/databricks-extract-load-sql-data-warehouse/save-key.png)

   儲存金鑰之後會顯示金鑰的值。 請複製此值，因為您之後就無法擷取金鑰。 您提供金鑰值和應用程式識別碼，能夠以應用程式方式登入。 將金鑰值儲存在應用程式可擷取的地方。

   ![儲存的金鑰](./media/databricks-extract-load-sql-data-warehouse/copy-key.png)

### <a name="get-tenant-id"></a>取得租用戶識別碼

以程式設計方式登入時，您需要將租用戶識別碼與您的驗證要求一起傳送。

1. 選取 **Azure Active Directory**。

   ![選取 Azure Active Directory](./media/databricks-extract-load-sql-data-warehouse/select-active-directory.png)

1. 若要取得租用戶識別碼，請選取 Azure AD 租用戶的 [屬性]。

   ![選取 Azure AD 屬性](./media/databricks-extract-load-sql-data-warehouse/select-ad-properties.png)

1. 複製 [目錄識別碼]。 這個值是您的租用戶識別碼。

   ![租用戶識別碼](./media/databricks-extract-load-sql-data-warehouse/copy-directory-id.png) 

## <a name="upload-data-to-data-lake-store"></a>將資料上傳至 Data Lake Store

在本節中，您會將資料檔案範例上傳至 Data Lake Store。 您稍後可以在 Azure Databricks 中使用此檔案來執行部分轉換。 您可以在此 [Github 存放庫](https://github.com/Azure/usql/blob/master/Examples/Samples/Data/json/radiowebsite/small_radio_json.json)中取得本教學課程中使用的資料範例 (**small_radio_json.json**)。

1. 從 [Azure 入口網站](https://portal.azure.com)，選取您建立的 Data Lake Store 帳戶。

2. 從 [概觀] 索引標籤上，按一下 [資料總管]。

    ![開啟資料總管](./media/databricks-extract-load-sql-data-warehouse/open-data-explorer.png "開啟資料總管")

3. 在資料總管中，按一下 [上傳]。

    ![上傳選項](./media/databricks-extract-load-sql-data-warehouse/upload-to-data-lake-store.png "上傳選項")

4. 在 [上傳檔案] 中，瀏覽至資料檔案範例所在的位置，然後選取 [新增選取的檔案]。

    ![上傳選項](./media/databricks-extract-load-sql-data-warehouse/upload-data.png "上傳選項")

5. 在本教學課程中，您會將資料檔案上傳至 Data Lake Store 的根目錄。 因此，檔案現在已位於 `adl://<YOUR_DATA_LAKE_STORE_ACCOUNT_NAME>.azuredatalakestore.net/small_radio_json.json`。

## <a name="associate-service-principal-with-azure-data-lake-store"></a>讓服務主體與 Azure Data Lake Store 產生關聯

在本節中，您會讓 Azure Data Lake Store 帳戶中的資料與您建立的 Azure Active Directory 服務主體產生關聯。 這可確保您可以從 Azure Databricks 存取 Data Lake Store 帳戶。 在本文的案例中，您會讀取 Data Lake Store 中的資料，以填入 SQL 資料倉儲中的資料表。 根據 [Data Lake Store 中的存取控制概觀](../data-lake-store/data-lake-store-access-control.md#common-scenarios-related-to-permissions)，對於 Data Lake Store 中的檔案若要有讀取存取權，您必須：

- 對資料夾結構中所有包含檔案的資料夾具備 [執行] 權限。
- 具備檔案本身的 [讀取] 權限。

請執行下列步驟以授與這些權限。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取您所建立的 Data Lake Store 帳戶，然後選取 [資料總管]。

    ![啟動資料總管](./media/databricks-extract-load-sql-data-warehouse/azure-databricks-data-explorer.png "啟動資料總管")

2. 在此案例中，由於範例資料檔案位於資料夾結構的根目錄，因此您只需要指派資料夾根目錄的 [執行] 權限。 若要這樣做，請從資料總管的根目錄選取 [存取]。

    ![新增資料夾的 ACL](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-folder-1.png "新增資料夾的 ACL")

3. 在 [存取] 下方，選取 [新增]。

    ![新增資料夾的 ACL](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-folder-2.png "新增資料夾的 ACL")

4. 在 [指派權限] 下方按一下 [選取使用者或群組]，並搜尋您先前建立的 Azure Active Directory 服務主體。

    ![新增 Data Lake Store 存取](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-folder-3.png "新增 Data Lake Store 存取")

    選取您要指派的 AAD 服務主體，然後按一下 [選取]。

5. 在 [指派權限] 下方，按一下 [選取權限] > [執行]。 保留其他預設值，並選取 [選取權限] 下方的 [確定]，然後選取 [指派權限] 下的 [確定]。

    ![新增 Data Lake Store 存取](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-folder-4.png "新增 Data Lake Store 存取")

6. 返回 [資料總管]，並按一下要指派讀取權限的檔案。 在 [檔案預覽] 下方，選取 [存取]。

    ![新增 Data Lake Store 存取](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-file-1.png "新增 Data Lake Store 存取")

7. 在 [存取] 下方，選取 [新增]。 在 [指派權限] 下方按一下 [選取使用者或群組]，並搜尋您先前建立的 Azure Active Directory 服務主體。

    ![新增 Data Lake Store 存取](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-folder-3.png "新增 Data Lake Store 存取")

    選取您要指派的 AAD 服務主體，然後按一下 [選取]。

8. 在 [指派權限] 下方，按一下 [選取權限] > [讀取]。 選取 [選取權限] 下方的 [確定]，然後選取 [指派權限] 下的 [確定]。

    ![新增 Data Lake Store 存取](./media/databricks-extract-load-sql-data-warehouse/add-adls-access-file-2.png "新增 Data Lake Store 存取")

    服務主體現在已有足夠的權限可讀取 Azure Data Lake Store 中的範例資料檔案。

## <a name="extract-data-from-data-lake-store"></a>從 Data Lake Store 擷取資料

在本節中，您會在 Azure Databricks 工作區中建立 Notebook，然後執行程式碼片段，將資料從 Data Lake Store 擷取至 Azure Databricks。

1. 在 [Azure 入口網站](https://portal.azure.com)中，移至您所建立的 Azure Databricks 工作區，然後選取 [啟動工作區]。

2. 在左側窗格中，選取 [工作區]。 從 [工作區] 下拉式清單選取 [建立] > [Notebook]。

    ![在 Databricks 中建立 Notebook](./media/databricks-extract-load-sql-data-warehouse/databricks-create-notebook.png "在 Databricks 中建立 Notebook")

2. 在 [建立 Notebook] 對話方塊中，輸入 Notebook 的名稱。 選取 [Scala] 作為語言，然後選取您先前建立的 Spark 叢集。

    ![在 Databricks 中建立 Notebook](./media/databricks-extract-load-sql-data-warehouse/databricks-notebook-details.png "在 Databricks 中建立 Notebook")

    選取 [建立] 。

3. 新增下列程式碼片段，並以您稍早儲存的 Azure Active Directory 服務主體值取代預留位置值。

        spark.conf.set("dfs.adls.oauth2.access.token.provider.type", "ClientCredential")
        spark.conf.set("dfs.adls.oauth2.client.id", "<APPLICATION-ID>")
        spark.conf.set("dfs.adls.oauth2.credential", "<AUTHENTICATION-KEY>")
        spark.conf.set("dfs.adls.oauth2.refresh.url", "https://login.microsoftonline.com/<TENANT-ID>/oauth2/token")

    按 **SHIFT + ENTER** 以執行程式碼單元。

4. 您現在可以將 Data Lake Store 中的 json 檔案範例以資料框架的形式載入 Azure Databricks。 在新的程式碼單元中貼上下列程式碼片段，取代預留位置的值，然後按 **SHIFT + ENTER**。

        val df = spark.read.json("adl://<DATA LAKE STORE NAME>.azuredatalakestore.net/small_radio_json.json")

5. 執行下列程式碼片段，以查看資料框架的內容。

        df.show()

    您會看到如下列程式碼片段的輸出：

        +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
        |               artist|     auth|firstName|gender|itemInSession|  lastName|   length|  level|            location|method|    page| registration|sessionId|                song|status|           ts|userId|
        +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
        | El Arrebato         |Logged In| Annalyse|     F|            2|Montgomery|234.57914| free  |  Killeen-Temple, TX|   PUT|NextSong|1384448062332|     1879|Quiero Quererte Q...|   200|1409318650332|   309|
        | Creedence Clearwa...|Logged In|   Dylann|     M|            9|    Thomas|340.87138| paid  |       Anchorage, AK|   PUT|NextSong|1400723739332|       10|        Born To Move|   200|1409318653332|    11|
        | Gorillaz            |Logged In|     Liam|     M|           11|     Watts|246.17751| paid  |New York-Newark-J...|   PUT|NextSong|1406279422332|     2047|                DARE|   200|1409318685332|   201|
        ...
        ...

您現在已將資料從 Azure Data Lake Store 擷取至 Azure Databricks。

## <a name="transform-data-in-azure-databricks"></a>轉換 Azure Databricks 中的資料

未經處理的範例資料 **small_radio_json.json** 擷取了廣播電台的聽眾，並且有不同資料行。 在本節中，您會將資料轉換為僅從資料集擷取特定資料行。 

1. 一開始，僅在已建立的資料框架中擷取 *firstName*、*lastName**gender**location* 和 *level* 資料行。

        val specificColumnsDf = df.select("firstname", "lastname", "gender", "location", "level")
        specificColumnsDf.show()

    您會取得如下列程式碼片段所示的輸出：

        +---------+----------+------+--------------------+-----+
        |firstname|  lastname|gender|            location|level|
        +---------+----------+------+--------------------+-----+
        | Annalyse|Montgomery|     F|  Killeen-Temple, TX| free|
        |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
        |     Liam|     Watts|     M|New York-Newark-J...| paid|
        |     Tess|  Townsend|     F|Nashville-Davidso...| free|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
        |     Alan|     Morse|     M|Chicago-Napervill...| paid|
        |Gabriella|   Shelton|     F|San Jose-Sunnyval...| free|
        |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
        |     Tess|  Townsend|     F|Nashville-Davidso...| free|
        |     Alan|     Morse|     M|Chicago-Napervill...| paid|
        |     Liam|     Watts|     M|New York-Newark-J...| paid|
        |     Liam|     Watts|     M|New York-Newark-J...| paid|
        |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
        |     Alan|     Morse|     M|Chicago-Napervill...| paid|
        |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
        |     Alan|     Morse|     M|Chicago-Napervill...| paid|
        |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
        +---------+----------+------+--------------------+-----+

2.  您可以進一步轉換此資料，將 **level** 資料行重新命名為 **subscription_type**。

        val renamedColumnsDf = specificColumnsDf.withColumnRenamed("level", "subscription_type")
        renamedColumnsDf.show()

    您會取得如下列程式碼片段所示的輸出。

        +---------+----------+------+--------------------+-----------------+
        |firstname|  lastname|gender|            location|subscription_type|
        +---------+----------+------+--------------------+-----------------+
        | Annalyse|Montgomery|     F|  Killeen-Temple, TX|             free|
        |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
        |     Liam|     Watts|     M|New York-Newark-J...|             paid|
        |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
        |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
        |Gabriella|   Shelton|     F|San Jose-Sunnyval...|             free|
        |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
        |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
        |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
        |     Liam|     Watts|     M|New York-Newark-J...|             paid|
        |     Liam|     Watts|     M|New York-Newark-J...|             paid|
        |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
        |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
        |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
        |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
        |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
        |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
        +---------+----------+------+--------------------+-----------------+

## <a name="load-data-into-azure-sql-data-warehouse"></a>將資料載入 Azure SQL 資料倉儲

在本節中，您會將已轉換的資料上傳至 Azure SQL 資料倉儲。 您可以使用適用於 Azure Databricks 的 Azure SQL 資料倉儲連接器，直接將資料框架以表格形式上傳至 SQL 資料倉儲。

如先前所述，在 Azure Databricks 和 Azure SQL 資料倉儲之間上傳資料時，SQL 資料倉儲連接器會使用 Azure Blob 儲存體作為暫時儲存位置。 因此，一開始您需提供要連線到儲存體帳戶的組態。 您必須已建立屬於本文必要條件的帳戶。

1. 提供可從 Azure Databricks 存取 Azure 儲存體帳戶的組態。

        val blobStorage = "<STORAGE ACCOUNT NAME>.blob.core.windows.net"
        val blobContainer = "<CONTAINER NAME>"
        val blobAccessKey =  "<ACCESS KEY>"

2. 指定在 Azure Databricks 與 Azure SQL 資料倉儲之間移動資料所用的暫存資料夾。

        val tempDir = "wasbs://" + blobContainer + "@" + blobStorage +"/tempDirs"

3. 執行下列程式碼片段，儲存組態中的 Azure Blob 儲存體存取金鑰。 這可確保您不必在 Notebook 中以純文字保留存取金鑰。

        val acntInfo = "fs.azure.account.key."+ blobStorage
        sc.hadoopConfiguration.set(acntInfo, blobAccessKey)

4. 提供用來連線至 Azure SQL 資料倉儲執行個體的值。 您必須已建立屬於必要條件的 SQL 資料倉儲。

        //SQL Data Warehouse related settings
        val dwDatabase = "<DATABASE NAME>"
        val dwServer = "<DATABASE SERVER NAME>" 
        val dwUser = "<USER NAME>"
        val dwPass = "<PASSWORD>"
        val dwJdbcPort =  "1433"
        val dwJdbcExtraOptions = "encrypt=true;trustServerCertificate=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
        val sqlDwUrl = "jdbc:sqlserver://" + dwServer + ".database.windows.net:" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass + ";$dwJdbcExtraOptions"
        val sqlDwUrlSmall = "jdbc:sqlserver://" + dwServer + ".database.windows.net:" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass

5. 執行下列程式碼片段，將已轉換的資料框架 **renamedColumnsDf** 以資料表形式載入 SQL 資料倉儲中。 此程式碼片段會在 SQL 資料庫中建立名為 **SampleTable** 的資料表。 請注意，Azure SQL DW 需要主要金鑰。  您可以在 SQL Server Management Studio 中執行 "CREATE MASTER KEY;" 命令，以建立主要金鑰。

        spark.conf.set(
          "spark.sql.parquet.writeLegacyFormat",
          "true")
        
        renamedColumnsDf.write
            .format("com.databricks.spark.sqldw")
            .option("url", sqlDwUrlSmall) 
            .option("dbtable", "SampleTable")
            .option( "forward_spark_azure_storage_credentials","True")
            .option("tempdir", tempDir)
            .mode("overwrite")
            .save()

6. 連線至 SQL 資料庫，並確認您已看到 **SampleTable**。

    ![確認範例資料表](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table.png "確認範例資料表")

7. 執行 SELECT 查詢，以確認資料表的內容。 其中應包含與 **renamedColumnsDf** 資料框架相同的資料。

    ![確認範例資料表內容](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table-content.png "確認範例資料表的內容")

## <a name="clean-up-resources"></a>清除資源

在本教學課程執行完後，您可以終止叢集。 若要這樣做，請從 Azure Databricks 工作區的左窗格中選取 [叢集]。 對於您想要終止的叢集，將游標移到 [動作] 資料行底下的省略符號上，然後選取 [終止] 圖示。

![停止 Databricks 叢集](./media/databricks-extract-load-sql-data-warehouse/terminate-databricks-cluster.png "停止 Databricks 叢集")

如果您不手動終止叢集，叢集將會自動停止，但前提是您已在建立叢集時選取 [在停止活動 __ 分鐘後終止] 核取方塊。 在這種情況下，叢集將會在停止運作達指定時間後自動停止。

## <a name="next-steps"></a>後續步驟 
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 Azure Databricks 工作區
> * 在 Azure Databricks 中建立 Spark 叢集
> * 建立 Azure Data Lake Store 帳戶
> * 將資料上傳至 Azure Data Lake Store
> * 在 Azure Databricks 中建立 Notebook
> * 從 Data Lake Store 擷取資料
> * 轉換 Azure Databricks 中的資料
> * 將資料載入 Azure SQL 資料倉儲

前往下一個教學課程，了解如何使用 Azure 事件中樞將即時資料串流到 Azure Databricks。

> [!div class="nextstepaction"]
>[使用事件中樞將資料串流至 Azure Databricks](databricks-stream-from-eventhubs.md)
