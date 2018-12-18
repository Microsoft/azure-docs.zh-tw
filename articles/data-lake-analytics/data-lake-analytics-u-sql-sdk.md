---
title: 使用 Azure Data Lake U-SQL SDK 在本機執行及測試 U-SQL 作業
description: 了解如何使用本機工作站上的命令列及程式設計介面，在本機執行及測試 U-SQL 作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: yanacai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.topic: conceptual
ms.date: 03/01/2017
ms.openlocfilehash: ae5334dcb93e34569131ab51dca99c310831082d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052082"
---
# <a name="run-and-test-u-sql-with-azure-data-lake-u-sql-sdk"></a>使用 Azure Data Lake U-SQL SDK 來執行及測試 U-SQL

在開發 U-SQL 指令碼時，通常會在本機執行並測試 U-SQL 指令碼，然後才送出至雲端。 Azure Data Lake 針對此案例提供稱為 Azure Data Lake U-SQL SDK 的 Nuget 套件，讓您可以輕鬆地調整 U-SQL 執行和測試。 此外，也可以將此 U-SQL 測試與 CI (持續整合) 系統整合以自動化編譯和測試。

如果您在意要如何使用 GUI 工具來手動本機執行並針對 U-SQL 指令碼進行偵錯，您可以使用 Azure Data Lake Tools for Visual Studio 來執行。 您可以在[這裡](data-lake-analytics-data-lake-tools-local-run.md)深入了解。

## <a name="install-azure-data-lake-u-sql-sdk"></a>安裝 Azure Data Lake U-SQL SDK

您可以在 Nuget.org 上的[這裡](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/)取得 Azure Data Lake U-SQL SDK。在使用它之前，您必須確定您具有下列相依性。

### <a name="dependencies"></a>相依性

Data Lake U-SQL SDK 需要下列相依性︰

- [Microsoft .NET Framework 4.6 或更新版本](https://www.microsoft.com/download/details.aspx?id=17851)。
- Microsoft Visual C++ 14 和 Windows SDK 10.0.10240.0 或更新版本 (在本文中稱為 CppSDK)。 有兩種方式可取得 CppSDK：

    - 安裝 [Visual Studio Community 版本](https://developer.microsoft.com/downloads/vs-thankyou)。 在 Program Files 資料夾下應該會有 \Windows Kits\10 資料夾，例如 C:\Program Files (x86)\Windows Kits\10。 您也會在 \Windows Kits\10\Lib 下找到 Windows 10 SDK 版本。 如果您看不見這些資料夾，請重新安裝 Visual Studio，並務必在安裝期間選取 Windows 10 SDK。 如果您已經隨 Visual Studio 一起安裝此項目，U-SQL 本機編譯器會自動尋找它。

    ![Data Lake Tools for Visual Studio 本機執行的 Windows 10 SDK](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-for-visual-studio-local-run-windows-10-sdk.png)

    - 安裝 [Data Lake Tools for Visual Studio](http://aka.ms/adltoolsvs)。 您可以在 C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\ADL Tools\X.X.XXXX.X\CppSDK 找到預先封裝的 Visual C++ 和 Windows SDK 檔案。 在此情況下，U-SQL 本機編譯器就無法自動找到相依性。 您必須為它指定 CppSDK 路徑。 您可以將檔案複製到另一個位置，或直接使用它。

## <a name="understand-basic-concepts"></a>了解基本概念

### <a name="data-root"></a>資料根

資料根資料夾是本機計算帳戶的「本機存放區」。 它就相當於 Data Lake Analytics 帳戶的 Azure Data Lake Store 帳戶。 切換到不同的資料根資料夾，就如同切換到不同的存放區帳戶。 如果您想要存取具有不同資料根資料夾的常用共用資料，便必須在指令碼中使用絕對路徑。 或者，您可以在資料根資料夾下建立檔案系統符號連結 (例如 NTFS 上的 **mklink**)，來指向共用資料。

資料根資料夾是用來︰

- 儲存本機中繼資料，包括資料庫、資料表、資料表值函式 (TVF)，以及組件。
- 查詢在 U-SQL 中定義為相對路徑的輸入和輸出路徑。 使用相對路徑能夠更容易將 U-SQL 專案部署至 Azure。

### <a name="file-path-in-u-sql"></a>U-SQL 中的檔案路徑

您可以在 U-SQL 指令碼中使用相對路徑和本機絕對路徑。 相對路徑是相對於指定的資料根資料夾路徑。 我們建議您使用 "/" 做為路徑分隔符號，讓您的指令碼與伺服器端相容。 以下是一些相對路徑及其對等絕對路徑的範例。 在這些範例中，C:\LocalRunDataRoot 是資料根資料夾。

|相對路徑|絕對路徑|
|-------------|-------------|
|/abc/def/input.csv |C:\LocalRunDataRoot\abc\def\input.csv|
|abc/def/input.csv  |C:\LocalRunDataRoot\abc\def\input.csv|
|D:/abc/def/input.csv |D:\abc\def\input.csv|

### <a name="working-directory"></a>工作目錄

當在本機執行 U-SQL 指令碼時，系統會在編譯期間於目前執行的目錄下建立一個工作目錄。 除了編譯輸出，本機執行所需的執行階段檔案會陰影複製到這個工作目錄。 工作目錄根資料夾稱為 "ScopeWorkDir"，而在工作目錄下的檔案如下：

|目錄/檔案|目錄/檔案|目錄/檔案|定義|說明|
|--------------|--------------|--------------|----------|-----------|
|C6A101DDCB470506| | |執行階段版本的雜湊字串|陰影複製本機執行所需的執行階段檔案|
| |Script_66AE4909AA0ED06C| |指令碼名稱 + 指令碼路徑的雜湊字串|編譯輸出和執行步驟記錄|
| | |\_script\_.abr|編譯器輸出|代數檔案|
| | |\_ScopeCodeGen\_.*|編譯器輸出|產生的受控碼|
| | |\_ScopeCodeGenEngine\_.*|編譯器輸出|產生的原生程式碼|
| | |參考的組件|組件參考|參考的組件檔案|
| | |deployed_resources|資源部署|資源部署檔案|
| | |xxxxxxxx.xxx[1..n]\_\*.\*|執行記錄檔|執行步驟的記錄檔|


## <a name="use-the-sdk-from-the-command-line"></a>從命令列使用 SDK

### <a name="command-line-interface-of-the-helper-application"></a>輔助應用程式的命令列介面

在 SDK directory\build\runtime 底下，LocalRunHelper.exe 是命令列輔助應用程式，能為大部分最常使用的本機執行函式提供介面。 請注意，命令和引數參數都區分大小寫。 若要叫用此應用程式︰

    LocalRunHelper.exe <command> <Required-Command-Arguments> [Optional-Command-Arguments]

不使用引數執行 LocalRunHelper.exe，或使用 **help** 參數顯示說明資訊︰

    > LocalRunHelper.exe help

        Command 'help' :  Show usage information
        Command 'compile' :  Compile the script
        Required Arguments :
            -Script param
                    Script File Path
        Optional Arguments :
            -Shallow [default value 'False']
                    Shallow compile

在說明資訊中︰

-  **Command** 提供命令的名稱。  
-  **Required Argument** 列出必須提供的引數。  
-  **Optional Argument** 列出選擇性的引數，並具有預設值。  選擇性的布林引數沒有參數，如果出現參數則表示其預設值的負值。

### <a name="return-value-and-logging"></a>傳回值和記錄

如果成功，輔助應用程式會傳回 **0**；如果失敗，則會傳回 **-1**。 根據預設，協助程式會將所有訊息傳送到目前的主控台。 不過，大部分的命令都支援 **-MessageOut path_to_log_file** 選擇性引數，該引數會將輸出重新導向至記錄檔。

### <a name="environment-variable-configuring"></a>環境變數設定

U-SQL 本機執行需要指定的資料根做為本機儲存體帳戶，以及針對相依性指定的 CppSDK 路徑。 您可以針對它們在命令列中設定引數，或是設定環境變數。

- 設定 **SCOPE_CPP_SDK** 環境變數。

    如果您是透過安裝 Data Lake Tools for Visual Studio 來取得 Microsoft Visual C++ 和 Windows SDK，請確認您有下列資料夾︰

        C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK

    定義一個名為 **SCOPE_CPP_SDK** 的新環境變數來指向此目錄。 或將資料夾複製到其他位置，並依同樣方式將 **SCOPE_CPP_SDK** 指定為該資料夾。

    除了設定環境變數，您可以在使用命令列時指定 **-CppSDK** 引數。 這個引數會覆寫預設的 CppSDK 環境變數。

- 設定 **LOCALRUN_DATAROOT** 環境變數。

    定義一個名為 **LOCALRUN_DATAROOT** 的新環境變數指向資料根目錄。

    除了設定環境變數，您可以在使用命令列時對資料根目錄路徑指定 **-DataRoot** 引數。 這個引數會覆寫預設的資料根環境變數。 您必須將這個引數加入您要執行的每個命令列，以覆寫所有作業的預設資料根環境變數。

### <a name="sdk-command-line-usage-samples"></a>SDK 命令列使用範例

#### <a name="compile-and-run"></a>編譯和執行

**run** 命令用來編譯指令碼，然後執行編譯的結果。 其命令列引數結合 **compile** 和 **excute** 的引數。

    LocalRunHelper run -Script path_to_usql_script.usql [optional_arguments]

下列為 **run** 的選擇性引數：


|引數|預設值|說明|
|--------|-------------|-----------|
|-CodeBehind|False|指令碼具有.cs 程式碼後置|
|-CppSDK| |CppSDK 目錄|
|-DataRoot| DataRoot 環境變數|本機執行的 DataRoot，預設為 'LOCALRUN_DATAROOT' 環境變數|
|-MessageOut| |將主控台上的訊息傾印成檔案|
|-Parallel|1|使用指定的平行處理原則執行計畫|
|-References| |列出程式碼後置額外的參考組件或資料檔案的路徑，以 ';' 分隔|
|-UdoRedirect|False|產生 Udo 組件重新導向設定|
|-UseDatabase|master|供程式碼後置暫時註冊組件使用的資料庫|
|-Verbose|False|顯示詳細的執行階段輸出|
|-WorkDir|目前的目錄|編譯器使用方式和輸出的目錄|
|-RunScopeCEP|0|要使用的 ScopeCEP 模式|
|-ScopeCEPTempPath|temp|用於串流資料的暫存路徑|
|-OptFlags| |最佳化工具旗標的逗號分隔清單|


以下是範例：

    LocalRunHelper run -Script d:\test\test1.usql -WorkDir d:\test\bin -CodeBehind -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB –Parallel 5 -Verbose

除了結合 **compile** 和 **excute**，您可以分別編譯和執行已編譯的可執行檔。

#### <a name="compile-a-u-sql-script"></a>編譯 U-SQL 指令碼

**compile** 命令用來將 U-SQL 指令碼編譯為可執行檔。

    LocalRunHelper compile -Script path_to_usql_script.usql [optional_arguments]

下列為 **compile** 的選擇性引數：


|引數|說明|
|--------|-----------|
| -CodeBehind [預設值 'False']|指令碼具有.cs 程式碼後置|
| -CppSDK [預設值 '']|CppSDK 目錄|
| -DataRoot [預設值 'DataRoot environment variable']|本機執行的 DataRoot，預設為 'LOCALRUN_DATAROOT' 環境變數|
| -MessageOut [預設值 '']|將主控台上的訊息傾印成檔案|
| -References [預設值 '']|列出程式碼後置額外的參考組件或資料檔案的路徑，以 ';' 分隔|
| -Shallow [預設值 'False']|淺層編譯|
| -UdoRedirect [預設值 'False']|產生 Udo 組件重新導向設定|
| -UseDatabase [預設值 'master']|供程式碼後置暫時註冊組件使用的資料庫|
| -WorkDir [預設值 'Current Directory']|編譯器使用方式和輸出的目錄|
| -RunScopeCEP [預設值 '0']|要使用的 ScopeCEP 模式|
| -ScopeCEPTempPath [預設值 'temp']|用於串流資料的暫存路徑|
| -OptFlags [預設值 '']|最佳化工具旗標的逗號分隔清單|


這裡有一些使用範例。

編譯 U-SQL 指令碼：

    LocalRunHelper compile -Script d:\test\test1.usql

編譯 U-SQL 指令碼並設定資料根資料夾。 請注意，這將會覆寫設定環境變數。

    LocalRunHelper compile -Script d:\test\test1.usql –DataRoot c:\DataRoot

編譯 U-SQL 指令碼並設定工作目錄、參考組件和資料庫：

    LocalRunHelper compile -Script d:\test\test1.usql -WorkDir d:\test\bin -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB

#### <a name="execute-compiled-results"></a>執行編譯的結果

**execute** 命令用來執行編譯的結果。   

    LocalRunHelper execute -Algebra path_to_compiled_algebra_file [optional_arguments]

下列為 **execute** 的選擇性引數：

|引數|預設值|說明|
|--------|-------------|-----------|
|-DataRoot | '' |中繼資料執行的資料根。 它預設為 **LOCALRUN_DATAROOT** 環境變數。|
|-MessageOut | '' |將主控台上的訊息傾印成檔案。|
|-Parallel | '1' |使用指定的平行處理原則層級執行產生本機執行步驟的指示器。|
|-Verbose | 'False' |顯示詳細執行階段輸出的指示器。|

以下是使用範例︰

    LocalRunHelper execute -Algebra d:\test\workdir\C6A101DDCB470506\Script_66AE4909AA0ED06C\__script__.abr –DataRoot c:\DataRoot –Parallel 5


## <a name="use-the-sdk-with-programming-interfaces"></a>透過程式設計介面使用 SDK

程式設計介面都位於 LocalRunHelper.exe 中。 您可以使用它們來整合 U-SQL SDK 的功能性及 C# 測試架構，以調整您的 U-SQL 指令碼本機測試。 在此文章中，我將會使用標準 C# 單元測試專案來示範如何使用這些介面來測試您的 U-SQL 指令碼。

### <a name="step-1-create-c-unit-test-project-and-configuration"></a>步驟 1︰建立 C# 單元測試專案和設定

- 透過 [檔案] > [新增] > [專案] > [Visual C#] > [測試] > [單元測試專案] 來建立 C# 單元測試專案。
- 加入 LocalRunHelper.exe 做為專案的參考。 LocalRunHelper.exe 位於 Nuget 套件中的 \build\runtime\LocalRunHelper.exe。

    ![Azure Data Lake U-SQL SDK 加入參考](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-add-reference.png)

- U-SQL SDK「僅」支援 x64 環境，請務必將建置平台目標設定為 [x64]。 您可以透過 [專案屬性] > [建置] > [平台目標] 來設定。

    ![Azure Data Lake U-SQL SDK 設定 x64 專案](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-x64.png)

- 請務必將測試環境設定為 [x64]。 在 Visual Studio 中，您可以透過 [測試] > [測試設定] > [預設處理器架構] > [x64] 來設定。

    ![Azure Data Lake U-SQL SDK 設定 x64 測試環境](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-test-x64.png)

- 請務必將 NugetPackage\build\runtime\ 下的所有相依性檔案複製到專案工作目錄 (通常位於 ProjectFolder\bin\x64\Debug 之下)。

### <a name="step-2-create-u-sql-script-test-case"></a>步驟 2：建立 U-SQL 指令碼測試案例

以下是 U-SQL 指令碼測試的範例程式碼。 若要進行測試，您需要準備指令碼、輸入檔和預期的輸出檔。

    using System;
    using Microsoft.VisualStudio.TestTools.UnitTesting;
    using System.IO;
    using System.Text;
    using System.Security.Cryptography;
    using Microsoft.Analytics.LocalRun;

    namespace UnitTestProject1
    {
        [TestClass]
        public class USQLUnitTest
        {
            [TestMethod]
            public void TestUSQLScript()
            {
                //Specify the local run message output path
                StreamWriter MessageOutput = new StreamWriter("../../../log.txt");

                LocalRunHelper localrun = new LocalRunHelper(MessageOutput);

                //Configure the DateRoot path, Script Path and CPPSDK path
                localrun.DataRoot = "../../../";
                localrun.ScriptPath = "../../../Script/Script.usql";
                localrun.CppSdkDir = "../../../CppSDK";

                //Run U-SQL script
                localrun.DoRun();

                //Script output 
                string Result = Path.Combine(localrun.DataRoot, "Output/result.csv");

                //Expected script output
                string ExpectedResult = "../../../ExpectedOutput/result.csv";

                Test.Helpers.FileAssert.AreEqual(Result, ExpectedResult);

                //Don't forget to close MessageOutput to get logs into file
                MessageOutput.Close();
            }
        }
    }

    namespace Test.Helpers
    {
        public static class FileAssert
        {
            static string GetFileHash(string filename)
            {
                Assert.IsTrue(File.Exists(filename));

                using (var hash = new SHA1Managed())
                {
                    var clearBytes = File.ReadAllBytes(filename);
                    var hashedBytes = hash.ComputeHash(clearBytes);
                    return ConvertBytesToHex(hashedBytes);
                }
            }

            static string ConvertBytesToHex(byte[] bytes)
            {
                var sb = new StringBuilder();

                for (var i = 0; i < bytes.Length; i++)
                {
                    sb.Append(bytes[i].ToString("x"));
                }
                return sb.ToString();
            }

            public static void AreEqual(string filename1, string filename2)
            {
                string hash1 = GetFileHash(filename1);
                string hash2 = GetFileHash(filename2);

                Assert.AreEqual(hash1, hash2);
            }
        }
    }


### <a name="programming-interfaces-in-localrunhelperexe"></a>LocalRunHelper.exe 中的程式設計介面

LocalRunHelper.exe 提供 U-SQL 本機編譯、執行等等的程式設計介面。介面如下所列。

**建構函式**

public LocalRunHelper([System.IO.TextWriter messageOutput = null])

|參數|類型|說明|
|---------|----|-----------|
|messageOutput|System.IO.TextWriter|針對輸出訊息，請設為 null 以使用主控台|

**屬性**

|屬性|類型|說明|
|--------|----|-----------|
|AlgebraPath|字串|代數檔案的路徑 (代數檔案是其中一個編譯結果)|
|CodeBehindReferences|字串|如果指令碼具有其他程式碼後置參考，請指定路徑並以 ';' 分隔|
|CppSdkDir|字串|CppSDK 目錄|
|CurrentDir|字串|目前的目錄|
|DataRoot|字串|資料根路徑|
|DebuggerMailPath|字串|偵錯工具郵件槽的路徑|
|GenerateUdoRedirect|布林|是否要產生載入重新導向覆寫設定的組件|
|HasCodeBehind|布林|指令碼是否具有程式碼後置|
|InputDir|字串|輸入資料的目錄|
|MessagePath|字串|訊息傾印檔案路徑|
|OutputDir|字串|輸出資料的目錄|
|平行處理原則|int|執行代數的平行處理原則|
|ParentPid|int|服務監視器結束的父項 PID，設定為 0 或負數以略過|
|ResultPath|字串|結果傾印檔案路徑|
|RuntimeDir|字串|執行階段目錄|
|ScriptPath|字串|尋找指令碼的位置|
|Shallow|布林|是否進行淺層編譯|
|TempDir|字串|Temp 目錄|
|UseDataBase|字串|指定程式碼後置暫存組件註冊要使用的資料庫，預設為 master|
|WorkDir|字串|慣用的工作目錄|


**方法**

|方法|說明|傳回|參數|
|------|-----------|------|---------|
|public bool DoCompile()|編譯 U-SQL 指令碼|成功時為 True| |
|public bool DoExec()|執行編譯的結果|成功時為 True| |
|public bool DoRun()|執行 U-SQL 指令碼 (編譯 + 執行)|成功時為 True| |
|public bool IsValidRuntimeDir(string path)|檢查指定的路徑是否為有效的執行階段路徑|有效則為 True|執行階段目錄的路徑|


## <a name="faq-about-common-issue"></a>有關常見問題的常見問題集

### <a name="error-1"></a>錯誤 1：
E_CSC_SYSTEM_INTERNAL: 內部錯誤! 無法載入檔案或組件 'ScopeEngineManaged.dll' 或其相依性的其中之一。 找不到指定的模組。

請檢查下列項目：

- 確定您使用 x64 環境。 建置目標平台和測試環境應該要是 x64，請參閱上方的＜步驟 1︰建立 C# 單元測試專案和設定＞。
- 確定您已經將 NugetPackage\build\runtime\ 下的所有相依性檔案複製到專案工作目錄。


## <a name="next-steps"></a>後續步驟

* 若要了解 U-SQL，請參閱 [開始使用 Azure Data Lake Analytics U-SQL 語言](data-lake-analytics-u-sql-get-started.md)。
* 若要記錄診斷資訊，請參閱 [為 Azure Data Lake Analytics 存取診斷記錄檔](data-lake-analytics-diagnostic-logs.md)。
* 若要了解更複雜的查詢，請參閱 [使用 Azure Data Lake Analytics 來分析網站記錄檔](data-lake-analytics-analyze-weblogs.md)。
* 若要檢視作業詳細資料，請參閱[針對 Azure Data Lake Analytics 作業使用作業瀏覽器和作業檢視](data-lake-analytics-data-lake-tools-view-jobs.md)。
* 若要使用頂點執行檢視，請參閱[在 Data Lake Tools for Visual Studio 中使用頂點執行檢視](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md)。
