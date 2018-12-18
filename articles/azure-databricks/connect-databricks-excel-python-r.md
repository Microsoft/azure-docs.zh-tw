---
title: 從 Excel、Python 或 R 連線到 Azure Databricks | Microsoft Docs
description: 了解如何使用 Simba 驅動程式將 Azure Databricks 連線至 Excel、Python 或 R。
services: azure-databricks
documentationcenter: ''
author: nitinme
manager: cgronlun
editor: cgronlun
ms.service: azure-databricks
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: nitinme
ms.openlocfilehash: 333ff3ac3de053eae604ffeab600df7d35874f69
ms.sourcegitcommit: d1eefa436e434a541e02d938d9cb9fcef4e62604
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/28/2018
ms.locfileid: "37085227"
---
# <a name="connect-to-azure-databricks-from-excel-python-or-r"></a>從 Excel、Python 或 R 連線到 Azure Databricks

在本文中，您會了解如何使用 Databricks ODBC 驅動程式來連線 Azure Databricks 與 Microsoft Excel、Python 或 R 語言。 一旦建立連線後，您可以從 Excel、Python 或 R 用戶端存取 Azure Databricks 中的資料。 您也可以使用用戶端來進一步分析資料。 

## <a name="prerequisites"></a>先決條件

* 您必須擁有 Azure Databricks 工作區、Spark 叢集，以及與叢集相關聯的範例資料。 如果您尚未具備這些先決條件，請完成[使用 Azure 入口網站在 Azure Databricks 上執行 Spark 作業](quickstart-create-databricks-workspace-portal.md)中的快速入門。

* 從 [Databricks 驅動程式下載頁面](https://databricks.com/spark/odbc-driver-download)下載 Databricks ODBC 驅動程式。 要安裝 32 位元或 64 位元版本，取決於要與 Azure Databricks 連線的應用程式。 例如，若要從 Excel 連線，則安裝 32 位元版本的驅動程式。 若要從 R 和 Python 連線，則安裝 64 位元版本的驅動程式。

* 在 Databricks 中設定個人存取權杖。 如需指示，請參閱[權杖管理](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management)。

## <a name="set-up-a-dsn"></a>設定 DNS

資料來源名稱 (DSN) 包含特定資料來源的相關資訊。 ODBC 驅動程式需要此 DSN 來連線到資料來源。 在本節中，您將設定可與 Databricks ODBC 驅動程式搭配使用的 DNS，藉此從 Microsoft Excel、Python 或 R 等用戶端連線到 Azure Databricks。

1. 從 Azure Databricks 工作區中，瀏覽至 Databricks 叢集。

    ![開啟 Databricks 叢集](./media/connect-databricks-excel-python-r/open-databricks-cluster.png "開啟 Databricks 叢集")

2. 在 [組態] 索引標籤下，按一下 [JDBC/ODBC] 索引標籤，並複製 [伺服器主機名稱] 和 [HTTP 路徑] 的值。 您需要這些值來完成本文中的步驟。

    ![取得 Databricks 組態](./media/connect-databricks-excel-python-r/get-databricks-jdbc-configuration.png "取得 Databricks 組態")

3. 在您的電腦上，根據應用程式啟動 **ODBC 資料來源**應用程式 (32 位元或 64 位元)。 若要從 Excel 連線，使用 32 位元版本。 若要從 R 和 Python 連線，使用 64 位元版本。

    ![啟動 ODBC](./media/connect-databricks-excel-python-r/launch-odbc-app.png "啟動 ODBC 應用程式")

4. 在 [使用者 DSN] 索引標籤下，按一下 [新增]。 在 [建立新的資料來源] 對話方塊中，選取 [Simba Spark ODBC 驅動程式]，然後按一下 [完成]。

    ![啟動 ODBC](./media/connect-databricks-excel-python-r/add-new-user-dsn.png "啟動 ODBC 應用程式")

5. 在 [Simba Spark ODBC 驅動程式] 對話方塊中，提供下列值：

    ![設定 DSN](./media/connect-databricks-excel-python-r/odbc-dsn-setup.png "設定 DSN")

    下表說明要在對話方塊中提供的值。
    
    |欄位  | 值  |
    |---------|---------|
    |**資料來源名稱**     | 提供資料來源的名稱。        |
    |**主機**     | 提供您在 Databricks 工作區複製的「伺服器主機名稱」值。        |
    |**連接埠**     | 輸入 443。        |
    |**驗證** > **機制**     | 選取「使用者名稱和密碼」。        |
    |**使用者名稱**     | 輸入 *token*。        |
    |**密碼**     | 輸入您在 Databricks 工作區複製的權杖值。 |
    
    在 DSN 設定對話方塊中，執行下列額外的步驟。
    
    * 按一下 [HTTP 選項]。 在開啟的對話方塊中，貼上從 Databricks 工作區複製的「HTTP 路徑」值。 按一下 [確定]。
    * 按一下 [SSL 選項]。 在開啟的對話方塊中，選取 [啟用 SSL] 核取方塊。 按一下 [確定]。
    * 按一下 [測試] 以測試與 Azure Databricks 的連線。 按一下 [確定] 以儲存組態。
    * 在 [ODBC 資料來源管理員] 對話方塊中，按一下 [確定]。

您現在可以設定 DNS。 在下一節中，您可以使用此 DSN 從 Excel、Python 或 R 連線到 Azure Databricks。

## <a name="connect-from-microsoft-excel"></a>從 Microsoft Excel 連線

在本節中，您將使用稍早建立的 DSN，將資料從 Azure Databricks 提取到 Microsoft Excel。 在開始之前，請確定您已將 Microsoft Excel 安裝在電腦上。 您可以從 [Microsoft Excel 試用版連結](https://products.office.com/excel)取得試用版 Excel。

1. 在 Microsoft Excel 中開啟空白的活頁簿。 從 [資料] 功能區，按一下 [取得資料]。 按一下 [從其他來源]，然後按一下 [從 ODBC]。

    ![從 Excel 啟動 ODBC](./media/connect-databricks-excel-python-r/launch-odbc-from-excel.png "從 Excel 啟動 ODBC")

2. 在 [從 ODBC] 對話方塊中，選取您稍早建立的 DSN，然後按一下 [確定]。

    ![選取 DSN](./media/connect-databricks-excel-python-r/excel-select-dsn.png "選取 DSN")

3. 如果出現要您輸入認證的提示，請在使用者名稱中輸入 **token**。 針對密碼，請提供從 Databricks 工作區中擷取的權杖值。

    ![提供 Databricks 的認證](./media/connect-databricks-excel-python-r/excel-databricks-token.png "選取 DSN")

4. 從 [導覽器] 視窗中，選取您要載入到 Excel 的 Databricks 資料表，然後按一下 [載入]。 

    ![將 dta 載入 Excel](./media/connect-databricks-excel-python-r/excel-load-data.png "將 dta 載入 Excel")

當您的 Excel 活頁簿中有資料後，您可以對其執行分析作業。

## <a name="connect-from-r"></a>從 R 連線

> [!NOTE]
> 本節針對如何使用 Azure Databricks 整合在您的桌面上執行的 R Studio 用戶端，提供相關資訊。 如需如何在 Azure Databricks 叢集本身使用 R Studio 的指示，請參閱 [Azure Databricks 上的 R Studio](https://docs.azuredatabricks.net/spark/latest/sparkr/rstudio.html)。

在本節中，您可以使用 R 語言的整合式開發環境 (IDE)，來參考 Azure Databricks 中的可用資料。 在開始之前，您必須已在電腦上安裝下列項目。

* R 語言的整合式開發環境 (IDE)。 本文使用桌面版 RStudio。 您可以從 [R Studio 下載](https://www.rstudio.com/products/rstudio/download/)進行安裝。
* 如果您使用桌面版 RStudio 作為 IDE，也請從 [http://aka.ms/rclient/](http://aka.ms/rclient/) 安裝 Microsoft R Client。 

開啟 RStudio 並執行下列步驟：

- 參考 `RODBC` 套件。 這可讓您使用稍早建立的 DSN 來連線到 Azure Databricks。
- 使用 DSN 建立連線。
- 在 Azure Databricks 中執行資料的 SQL 查詢。 在下列程式碼片段中，*radio_sample_data* 是已存在於 Azure Databricks 的資料表。
- 執行一些查詢作業來確認輸出。 

下列程式碼片段會執行以下工作：

    # reference the 'RODBC' package
    require(RODBC)
    
    # establish a connection using the DSN you created earlier
    conn <- odbcConnect("<ENTER DSN NAME HERE>")
    
    # run a SQL query using the connection you created
    res <- sqlQuery(conn, "SELECT * FROM radio_sample_data")
    
    # print out the column names in the query output
    names(res) 
        
    # print out the number of rows in the query output
    nrow (res)

## <a name="connect-from-python"></a>從 Python 連線

在本節中，您會使用 Python 的整合式開發環境 (例如 IDLE)，來參考 Azure Databricks 中的可用資料。 開始之前，請完成下列先決條件：

* 從[這裡](https://www.python.org/downloads/)安裝 Python。 從此連結安裝 Python 時，會同時安裝 IDLE。

* 從電腦上的命令提示字元中，安裝 `pyodbc` 套件。 執行以下命令：

      pip install pyodbc

開啟 IDLE 並執行下列步驟：

- 匯入 `pyodbc` 套件。 這可讓您使用稍早建立的 DSN 來連線到 Azure Databricks。
- 使用您稍早建立的 DSN 建立連線。
-  使用您建立的連線執行 SQL 查詢。 在下列程式碼片段中，*radio_sample_data* 是已存在於 Azure Databricks 的資料表。
- 執行查詢作業來確認輸出。

下列程式碼片段會執行以下工作：

```python
# import the `pyodbc` package:
import pyodbc

# establish a connection using the DSN you created earlier
conn = pyodbc.connect("DSN=<ENTER DSN NAME HERE>", autocommit = True)

# run a SQL query using the connection you created
cursor = conn.cursor()
cursor.execute("SELECT * FROM radio_sample_data")

# print the rows retrieved by the query.
for row in cursor.fetchall():
    print(row)

```

## <a name="next-steps"></a>後續步驟

* 若要了解可從中將資料匯入 Azure Databricks 的來源，請參閱 [Azure Databricks 的資料來源](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html#)


