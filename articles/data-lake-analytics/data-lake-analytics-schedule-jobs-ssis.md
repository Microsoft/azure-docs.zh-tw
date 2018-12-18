---
title: 使用 SSIS 對 Azure Data Lake Analytics U-SQL 作業進行排程
description: 了解如何使用 SQL Server Integration Services 來對 U-SQL 作業進行排程。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/17/2018
ms.openlocfilehash: 6eb2452507c3adcb8532a0dd2e50ad701e35cd7f
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579213"
---
# <a name="schedule-u-sql-jobs-using-sql-server-integration-services-ssis"></a>使用 SQL Server Integration Services (SSIS) 對 U-SQL 作業進行排程

在本文件中，您會了解如何使用 SQL Server Integration Services (SSIS) 來協調及建立 U-SQL 作業。 

## <a name="prerequisites"></a>必要條件

[適用於 Integration Services 的 Azure Feature Pack](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017#scenario-managing-data-in-the-cloud) 能提供 [Azure Data Lake Analytics 工作](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017)和 [Azure Data Lake Analytics 連線管理員](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager?view=sql-server-2017)，以協助連線至 Azure Data Lake Analytics 服務。 若要使用此工作，請務必安裝：

- [下載並安裝適用於 Visual Studio 的 SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017)
- [安裝適用於 Integration Services (SSIS) 的 Azure Feature Pack](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017)

## <a name="azure-data-lake-analytics-task"></a>Azure Data Lake Analytics 工作

Azure Data Lake Analytics 工作能讓使用者將 U-SQL 作業提交至 Azure Data Lake Analytics 帳戶。 

[了解如何設定 Azure Data Lake Analytics 工作](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017)。

![SSIS 中的 Azure Data Lake Analytics 工作](./media/data-lake-analytics-schedule-jobs-ssis/data-lake-analytics-azure-data-lake-analytics-task-in-ssis.png)

您可以使用 SSIS 內建的功能和工作，從不同的位置取得 U-SQL 指令碼。下列案例示範如何針對不同的使用者案例設定 U-SQL 指令碼。

## <a name="scenario-1-use-inline-script-call-tvfs-and-stored-procs"></a>案例 1：使用內嵌指令碼呼叫 TVF 和預存程序

在 Azure Data Lake Analytics 工作編輯器中，將 [SourceType] 設定為 [DiretInput]，並將 U-SQL 陳述式置於 [USQLStatemnt] 中。

若要使維護和程式碼管理更為輕鬆，請只使用簡短的 U-SQL 指令碼作為指令碼指令碼，例如您可以呼叫 U-SQL 資料庫中的現有資料表值函式和預存程序。 

![編輯 SSIS 工作中的內嵌 U-SQL 指令碼](./media/data-lake-analytics-schedule-jobs-ssis/edit-inline-usql-script-in-ssis.png)

相關文章：[如何將參數傳遞至預存程序](#scenario-6-pass-parameters-to-u-sql-script)

## <a name="scenario-2-use-u-sql-files-in-azure-data-lake-store"></a>案例 2：在 Azure Data Lake Store 中使用 U-SQL 檔案

您也可以透過使用 Azure Feature Pack 中的 **Azure Data Lake Store 檔案系統工作**，來在 Azure Data Lake Store 中使用 U-SQL 檔案。 此方法可讓您使用儲存在雲端上的指令碼。

請遵循下列步驟，以設定 Azure Data Lake Store 檔案系統工作和 Azure Data Lake Analytics 工作之間的連線。

### <a name="set-task-control-flow"></a>設定工作控制流程

在 SSIS 套件設計檢視中，新增 **Azure Data Lake Store 檔案系統工作**和 **Foreach 迴圈容器**，並在該 Foreach 迴圈容器中新增 **Azure Data Lake Analytics 工作**。 Azure Data Lake Store 檔案系統工作可協助將您 ADLS 帳戶中的 U-SQL 檔案下載至暫時資料夾。 Foreach 迴圈容器和 Azure Data Lake Analytics 工作可協助將暫時資料夾中的每個 U-SQL 檔案，以 U-SQL 作業的形式提交至 Azure Data Lake Analytics 帳戶。

![在 Azure Data Lake Store 中使用 U-SQL 檔案](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-data-lake-store.png)

### <a name="configure-azure-data-lake-store-file-system-task"></a>設定 Azure Data Lake Store 檔案系統工作

1. 將 [Operation] 設定為 [CopyFromADLS]。
2. 設定 [AzureDataLakeConnection]，深入了解 [Azure Data Lake Store 連線管理員](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager?view=sql-server-2017)。
3. 設定 [AzureDataLakeDirectory]。 指向儲存 U-SQL 指令碼的資料夾。 使用對應至 Azure Data Lake Store 帳戶根資料夾的相對路徑。
4. 將 [Destination] 設定為會對已下載的 U-SQL 指令碼進行快取的資料夾。 此資料夾路徑將會在 Foreach 迴圈容器中用來進行 U-SQL 作業提交。 

![設定 Azure Data Lake Store 檔案系統工作](./media/data-lake-analytics-schedule-jobs-ssis/configure-azure-data-lake-store-file-system-task.png)

[深入了解 Azure Data Lake Store 檔案系統工作](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-store-file-system-task?view=sql-server-2017)。

### <a name="configure-foreach-loop-container"></a>設定 Foreach 迴圈容器

1. 在 [集合] 頁面中，將 [列舉程式] 設定為 [Foreach 檔案列舉程式]。

2. 將 [列舉程式設定] 群組底下的 [資料夾] 設定為包含已下載 U-SQL 指令碼的暫時資料夾。

3. 將 [列舉程式設定] 底下的 [資料夾] 設定為 `*.usql`，使迴圈容器能擷取結尾為 `.usql` 的檔案。

    ![設定 Foreach 迴圈容器](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-collection.png)

4. 在 [變數對應] 頁面中，新增使用者定義的變數以取得每個 U-SQL 檔案的檔案名稱。 將 [索引] 設定為 0 以取得檔案名稱。 在此範例中，會定義稱為 `User::FileName` 的變數。 此變數將會被用來在 Azure Data Lake Analytics 工作中以動態方式取得 U-SQL 指令碼檔案連線，並設定 U-SQL 作業名稱。

    ![設定 Foreach 迴圈容器以取得檔案名稱](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-variable-mapping.png)

### <a name="configure-azure-data-lake-analytics-task"></a>設定 Azure Data Lake Analytics 工作 

1. 將 [SourceType] 設定為 [FileConnection]。

2. 將 [FileConnection] 設定為指向從 Foreach 迴圈容器所傳回檔案物件的檔案連線。
    
    若要建立此檔案連線：

    1. 選擇 [FileConnection] 設定中的 **<New Connection...>**。
    2. 將 [使用類型] 設定為 [現有檔案]，並將 [檔案] 設定為任何現有檔案的檔案路徑。

        ![設定 Foreach 迴圈容器](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

    3. 在 [連線管理員] 檢視中，以滑鼠右鍵按一下剛剛建立的檔案連線，然後選擇 [屬性]。

    4. 在 [屬性] 視窗中，展開 [運算式]，然後將 [ConnectionString] 設定為在 Foreach 迴圈容器中定義的變數，例如 `@[User::FileName]`。

        ![設定 Foreach 迴圈容器](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-property-for-foreach-loop-container.png)

3. 將 [AzureDataLakeAnalyticsConnection] 設定為您想要提交作業的 Azure Data Lake Analytics 帳戶。 深入了解 [Azure Data Lake Analytics 連線管理員](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager?view=sql-server-2017)。

4. 設定其他作業組態。 [深入了解](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017)。

5. 使用**運算式**來以動態方式設定 U-SQL 作業：

    1. 在 [運算式] 頁面中，針對 [JobName] 新增新的運算式機碼值組。
    2. 將 [JobName] 的值設定為在 Foreach 迴圈容器中定義的變數，例如 `@[User::FileName]`。
    
        ![針對 U-SQL 作業名稱設定 SSIS 運算式](./media/data-lake-analytics-schedule-jobs-ssis/configure-expression-for-u-sql-job-name.png)

## <a name="scenario-3-use-u-sql-files-in-azure-blob-storage"></a>案例 3：在 Azure Blob 儲存體中使用 U-SQL 檔案

您可以透過使用 Azure Feature Pack 中的 **Azure Blob 下載工作**，來在 Azure Blob 儲存體中使用 U-SQL 檔案。 此方法可讓您使用雲端上的指令碼。

其步驟與[案例 2：在 Azure Data Lake Store 中使用 U-SQL 檔案](#scenario-2-use-u-sql-files-in-azure-data-lake-store)類似。 將 Azure Data Lake Store 檔案系統工作變更為 Azure Blob 下載工作。 [深入了解 Azure Blob 下載工作](https://docs.microsoft.com/sql/integration-services/control-flow/azure-blob-download-task?view=sql-server-2017)。

控制流程如下所示。

![在 Azure Data Lake Store 中使用 U-SQL 檔案](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-blob-storage.png)

## <a name="scenario-4-use-u-sql-files-on-the-local-machine"></a>案例 4：在本機電腦上使用 U-SQL 檔案

除了使用儲存在雲端上的 U-SQL 檔案之外，您也可以使用本機電腦上的檔案，或是和 SSIS 套件一起部署的檔案。

1. 以滑鼠右鍵按一下 SSIS 專案中的 [連線管理員]，然後選擇 [新增連線管理員]。

2. 選取 [檔案] 類型，然後按一下 [新增]。

3. 將 [使用類型] 設定為 [現有檔案]，並將 [檔案] 設定為本機電腦上的檔案。

    ![將檔案連線新增至本機檔案](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

4. 新增 [Azure Data Lake Analytics] 工作，然後：
    1. 將 [SourceType] 設定為 [FileConnection]。
    2. 將 [FileConnection] 設定為剛剛建立的檔案連線。

5. 完成其他 Azure Data Lake Analytics 工作的設定。

## <a name="scenario-5-use-u-sql-statement-in-ssis-variable"></a>案例 5：使用以 SSIS 變數產生的 U-SQL 陳述式

在某些情況下，您可能需要以動態方式產生 U-SQL 陳述式。 您可以搭配 **SSIS 運算式**及其他 SSIS 工作 (例如指令碼工作) 使用 **SSIS 變數**，來協助動態產生 U-SQL 陳述式。

1. 透過 [SSIS] > [變數] 最上層功能表，開啟 [變數] 工具視窗。

2. 新增 SSIS 變數並直接設定其值，或使用 [運算式] 來產生值。

3. 新增 [Azure Data Lake Analytics] 工作，然後：
    1. 將 [SourceType] 設定為 [Variable]。
    2. 將 [SourceVariable] 設定為剛剛建立的 SSIS 變數。

4. 完成其他 Azure Data Lake Analytics 工作的設定。

## <a name="scenario-6-pass-parameters-to-u-sql-script"></a>案例 6：將參數傳遞至 U-SQL 指令碼

在某些情況下，您可能需要以動態方式設定 U-SQL 指令碼中的 U-SQL 變數值。 Azure Data Lake Analytics 工作中的 [參數對應] 功能可在此案例上提供協助。 一共有兩個常見的使用者案例：

- 根據目前的日期和時間，以動態方式設定輸入和輸出檔案路徑變數。
- 設定預存程序的參數。

[深入了解如何設定 U-SQL 指令碼的參數](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017#parameter-mapping-page-configuration)。

## <a name="next-steps"></a>後續步驟

- [在 Azure 中執行 SSIS 套件](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)
- [適用於 Integration Services (SSIS) 的 Azure Feature Pack](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017#scenario-managing-data-in-the-cloud)
- [使用 Azure Data Factory 對 U-SQL 作業進行排程](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics)

