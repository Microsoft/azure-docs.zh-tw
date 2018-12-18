---
title: 如何測試 Azure Data Lake Analytics 程式碼
description: 了解如何為 Azure Data Lake Analytics 的 U-SQL 指令碼和擴充 C# 程式碼新增測試案例。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: 82ffcc6f891a64650375121b9418daad33dc2628
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301687"
---
# <a name="test-your-azure-data-lake-analytics-code"></a>測試 Azure Data Lake Analytics 程式碼

Azure Data Lake 提供 U-SQL 語言，其中結合了宣告式 SQL 與命令式 C#，可讓您處理任何規模的資料。 在本文件中，您將了解如何為 U-SQL 和擴充 C# UDO (使用者定義運算子) 程式碼建立測試案例。

## <a name="test-u-sql-scripts"></a>測試 U-SQL 指令碼

U-SQL 指令碼已經過編譯和最佳化，成為可在雲端上的電腦或本機電腦上執行的程式碼。 編譯和最佳化程序會將整個 U-SQL 指令碼視為一體。 您不能針對個別陳述式進行傳統的「單元測試」。 不過，只要使用 U-SQL 測試 SDK 和本機執行 SDK，您就能進行指令碼層級的測試。

### <a name="create-test-cases-for-u-sql-script"></a>為 U-SQL 指令碼建立測試案例

Azure Data Lake Tools for Visual Studio 可讓您建立 U-SQL 指令碼測試案例。

1.  在 [方案總管] 中以滑鼠右鍵按一下 U-SQL 指令碼，然後選取 [建立單元測試]。
2.  建立新的測試專案，或將測試案例插入現有的測試專案中。

    ![Data Lake Tools for Visual Studio -- 建立 U-SQL 測試專案](./media/data-lake-analytics-cicd-test/data-lake-tools-create-usql-test-project.png) 

    ![Data Lake Tools for Visual Studio -- 建立 U-SQL 測試專案組態](./media/data-lake-analytics-cicd-test/data-lake-tools-create-usql-test-project-configure.png) 

### <a name="manage-the-test-data-source"></a>管理測試資料來源

測試 U-SQL 指令碼時，您需要測試輸入檔案。 您可以在 U-SQL 專案屬性中設定 [測試資料來源]，藉以管理測試資料。 

當您呼叫 U-SQL 測試 SDK 中的 `Initialize()` 介面時，系統會在測試專案的工作目錄下建立暫存本機資料根資料夾，並且會在執行 U-SQL 指令碼測試案例之前，將測試資料來源資料夾內的所有檔案和子資料夾 (連同子資料夾下的檔案) 複製到暫存本機資料的根資料夾。 您可以利用分號分隔測試資料資料夾路徑，以新增其他測試資料來源資料夾。

![Data Lake Tools for Visual Studio -- 設定專案測試資料來源](./media/data-lake-analytics-cicd-test/data-lake-tools-configure-project-test-data-source.png)

### <a name="manage-the-database-environment-for-testing"></a>管理測試的資料庫環境

如果您的 U-SQL 指令碼使用的是 U-SQL 資料庫物件，或是會利用該物件進行查詢 (例如在呼叫預存程序時)，則需要在執行 U-SQL 測試案例之前，先將資料庫環境初始化。 U-SQL 測試 SDK 中的 `Initialize()` 介面可協助您將 U-SQL 專案參考的所有資料庫，部署至測試專案工作目錄中的暫存本機資料根資料夾。 

深入了解[如何管理 U-SQL 專案的 U-SQL 資料庫專案參考](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project)。

### <a name="verify-test-results"></a>驗證測試結果

`Run()` 介面會傳回作業執行結果。 0 表示成功，1 表示失敗。 您也可以使用 C# 判斷提示函式來驗證輸出。 

### <a name="run-test-cases-in-visual-studio"></a>在 Visual Studio 中執行測試案例

U-SQL 指令碼測試專案建置於 C# 單元測試架構之上。 建置專案後，您可以透過 [測試總管] > [播放清單] 執行所有測試案例。 或者，以滑鼠右鍵按一下 .cs 檔案，然後選取 [執行測試]。

## <a name="test-c-udos"></a>測試 C# UDO

### <a name="create-test-cases-for-c-udos"></a>為 C# UDO 建立測試案例

您可以使用 C# 單元測試架構來測試 C# UDO (使用者定義運算子)。 在測試 UDO 時，需要準備對應的 **IRowset** 物件作為輸入。

有兩種方式可以建立 IRowset 物件：

- 從檔案載入資料來建立 IRowset：

    ```csharp
    //Schema: "a:int, b:int"
    USqlColumn<int> col1 = new USqlColumn<int>("a");
    USqlColumn<int> col2 = new USqlColumn<int>("b");
    List<IColumn> columns = new List<IColumn> { col1, col2 };
    USqlSchema schema = new USqlSchema(columns);

    //Generate one row with default values
    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Get data from file
    IRowset rowset = UnitTestHelper.GetRowsetFromFile(@"processor.txt", schema, output.AsReadOnly(), discardAdditionalColumns: true, rowDelimiter: null, columnSeparator: '\t');
    ```

- 使用資料收集中的資料來建立 IRowset：

    ```csharp
    //Schema: "a:int, b:int"
    USqlSchema schema = new USqlSchema(
        new USqlColumn<int>("a"),
        new USqlColumn<int>("b")
    );

    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Generate Rowset with specified values
    List<object[]> values = new List<object[]>{
        new object[2] { 2, 3 },
        new object[2] { 10, 20 }
    };

    IEnumerable<IRow> rows = UnitTestHelper.CreateRowsFromValues(schema, values);
    IRowset rowset = UnitTestHelper.GetRowsetFromCollection(rows, output.AsReadOnly());
    ```

### <a name="verify-test-results"></a>驗證測試結果

呼叫 UDO 函式後，您可以使用 C# 判斷提示函式，透過結構描述和資料列集值驗證來檢驗結果。 您可以在 Visual Studio 中透過 [檔案] > [新增] > [專案]，使用 U-SQL C# UDO 單元測試範例專案中的範例程式碼。

### <a name="run-test-cases-in-visual-studio"></a>在 Visual Studio 中執行測試案例

建置測試專案後，您可以在 [測試總管] > [播放清單] 執行所有測試案例，或以滑鼠右鍵按一下 .cs 檔案，然後選擇 [執行測試]。

## <a name="run-test-cases-in-azure-devops"></a>在 Azure DevOps 中執行測試案例

**U-SQL 指令碼測試專案**和 **C# UDO 測試專案**皆會繼承 C# 單元測試專案。 Azure DevOps 中的 [Visual Studio 測試工作](https://docs.microsoft.com/azure/devops/pipelines/test/getting-started-with-continuous-testing?view=vsts)可執行這些測試案例。 

### <a name="run-u-sql-test-cases-in-azure-devops"></a>在 Azure DevOps 中執行 U-SQL 測試案例

若要進行 U-SQL 測試，請確定您已在組建電腦上載入 `CPPSDK`，然後將 `CPPSDK` 路徑傳至 USqlScriptTestRunner(cppSdkFolderFullPath: \@"")。

**什麼是 CPPSDK？**

CPPSDK 是包含 Microsoft Visual C++ 14 和 Windows SDK 10.0.10240.0 的套件。 這是 U-SQL 執行階段所需的環境。 您可以在 Azure Data Lake Tools for Visual Studio 安裝資料夾下取得此套件：

- 在 Visual Studio 2015 中，其位於 `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`
- 在 Visual Studio 2017 中，其位於 `C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\SDK\ScopeCppSDK`

**在 Azure DevOps 組建代理程式中準備 CPPSDK**

最常用來在 Azure DevOps 中準備 CPPSDK 相依性的方法如下：

1.  將包含 CPPSDK 程式庫的資料夾壓縮成 ZIP 檔案。
2.  將 .zip 檔案簽入您的來源控制系統中。 (.zip 檔案可確保您會簽入 CPPSDK 資料夾下的所有程式庫，而使 ".gitignore" 不會忽略某些檔案。)   
3.  在組建管線中解壓縮 .zip 檔案。
4.  將 `USqlScriptTestRunner` 指向組建電腦上解壓縮的資料夾。

### <a name="run-c-udo-test-cases-in-azure-devops"></a>在 Azure DevOps 中執行 C# UDO 測試案例

若要進行 C# UDO 測試，請確實參考 UDO 所需的下列組件。 如果您透過 [Nuget 套件 Microsoft.Azure.DataLake.USQL.Interfaces](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.Interfaces/) 參考組件，請務必在組建管線中新增 NuGet 還原工作。

* Microsoft.Analytics.Interfaces
* Microsoft.Analytics.Types
* Microsoft.Analytics.UnitTest

## <a name="next-steps"></a>後續步驟

- [如何設定 Azure Data Lake Analytics 的 CI/CD 管線](data-lake-analytics-cicd-overview.md)
- [在本機電腦上執行 U-SQL 指令碼](data-lake-analytics-data-lake-tools-local-run.md)
- [使用 U-SQL 資料庫專案開發 U-SQL 資料庫](data-lake-analytics-data-lake-tools-develop-usql-database.md)

