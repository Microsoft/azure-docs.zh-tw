---
title: 針對已失敗 Azure Data Lake U-SQL 作業的使用者定義 C# 程式碼進行偵錯
description: 本文說明如何使用 Azure Data Lake Tools for Visual Studio 針對 U-SQL 失敗的頂點進行偵錯。
services: data-lake-analytics
ms.service: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: bcd0b01e-1755-4112-8e8a-a5cabdca4df2
ms.topic: conceptual
ms.date: 11/30/2017
ms.openlocfilehash: 1fd1ce54dea672b19baea84bbf3a40b4365093b8
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046213"
---
# <a name="debug-user-defined-c-code-for-failed-u-sql-jobs"></a>對 U-SQL 失敗作業的使用者定義 C# 程式碼進行偵錯

U-SQL 提供使用 C# 的擴充性模型。 在 U-SQL 指令碼中，可以輕鬆地呼叫 C# 函式，以及執行類 SQL 宣告式語言不支援的分析函式。 若要深入了解 U-SQL 擴充性，請參閱 [U-SQL 程式設計指南](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#use-user-defined-functions-udf)。 

在實務上，任何程式碼可能都需要偵錯，但很難透過受限的記錄檔，針對雲端上使用自訂程式碼的分散式工作進行偵錯。 [Azure Data Lake Tools for Visual Studio](http://aka.ms/adltoolsvs) 提供了一個稱為「頂點失敗偵錯」的功能，該功能可協助您更輕鬆地針對您的自訂程式碼中發生的錯誤進行偵錯。 當 U-SQL 作業失敗時，服務會保留失敗狀態，而工具會協助您將雲端失敗環境下載至本機電腦進行偵錯。 本機下載會擷取整個雲端環境，包括任何輸入資料和使用者程式碼。

下列影片示範 Azure Data Lake Tools for Visual Studio 中的頂點失敗偵錯。

> [!VIDEO https://www.youtube.com/embed/3enkNvprfm4]
>

> [!IMPORTANT]
> Visual Studio 需要下列兩個更新以使用此功能：[Microsoft Visual C++ 2015 可轉散發更新 3](https://www.microsoft.com/en-us/download/details.aspx?id=53840) 和[適用於 Windows 的通用 C 執行階段](https://www.microsoft.com/download/details.aspx?id=50410)。
>

## <a name="download-failed-vertex-to-local-machine"></a>將失敗的頂點下載至本機電腦

當您在 Azure Data Lake Tools for Visual Studio 中開啟失敗的作業時，您會看到一個黃色的警示列，[錯誤] 索引標籤中有詳細的錯誤訊息。

1. 按一下 [下載]  以下載所有必要的資源和輸入資料流。 如果下載未完成，請按一下 [重試]。

2. 在下載完成後按一下 [開啟]，以產生本機的偵錯環境。 將開啟新的偵錯解決方案，如果您在 Visual Studio 中開啟現有的解決方案，請確定在偵錯之前先儲存並予以關閉。

![Azure Data Lake Analytics U-SQL 偵錯 Visual Studio 下載 Vertex](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-download-vertex.png)

## <a name="configure-the-debugging-environment"></a>設定偵錯環境

> [!NOTE]
> 在進行偵錯之前，請確定核取 [例外狀況設定] 視窗中的 [Common Language Runtime 例外狀況] \(**Ctrl + Alt + E**)。

![Azure Data Lake Analytics U-SQL 偵錯 Visual Studio 設定](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-clr-exception-setting.png)

在新啟動的 Visual Studio 執行個體中，您可能會找到或可能不會找到使用者定義的 C# 原始程式碼：

1. [我可以在方案中找到我的原始程式碼](#source-code-is-included-in-debugging-solution)

2. [我無法在方案中找到我的原始程式碼](#source-code-is-not-included-in-debugging-solution)

### <a name="source-code-is-included-in-debugging-solution"></a>原始程式碼包含在偵錯方案中

有兩種情況會擷取 C# 原始程式碼：

1. 使用者程式碼定義在程式碼後置檔案 (在 U-SQL 專案中通常會命名為 `Script.usql.cs`) 中。

2. 針對 U-SQL 應用程式，使用者程式碼會定義在 C# 類別庫專案中，並使用**偵錯資訊**註冊為組件。

如果原始程式碼匯入到方案中，您可以使用 Visual Studio 偵錯工具 (監看式、變數等) 針對問題進行疑難排解：

1. 按 **F5** 開始偵錯。 程式碼會持續執行，直到例外狀況將它停止。

2. 開啟原始程式碼檔案並設定中斷點，然後按 **F5** 逐步針對程式碼進行偵錯。

    ![Azure Data Lake Analytics U-SQL 偵錯例外狀況](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-exception.png)

### <a name="source-code-is-not-included-in-debugging-solution"></a>原始程式碼未包含在偵錯方案中

如果使用者程式碼未包含在程式碼後置檔案中，或者您尚未使用**偵錯資訊**註冊組件，則原始程式碼不會自動包含在偵錯方案中。 在此情況下，您需要額外步驟以新增您的原始程式碼：

1. 以滑鼠右鍵按一下 [方案 'VertexDebug'] > [新增] > [現有專案]，以尋找組件原始程式碼，並將專案新增至偵錯方案。

    ![Azure Data Lake Analytics U-SQL 偵錯新增專案](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-add-project-to-debug-solution.png)

2. 取得 **FailedVertexDebugHost** 專案的專案資料夾路徑。 

3. 以滑鼠右鍵按一下新增的組件原始程式碼專案 > [屬性]，選取左側的 [建置] 索引標籤，並將結尾為 \bin\debug 的複製路徑貼上為 [輸出] > [輸出路徑]。 最後的輸出路徑會像是 "<DataLakeTemp path>\fd91dd21-776e-4729-a78b-81ad85a4fba6\loiu0t1y.mfo\FailedVertexDebug\FailedVertexDebugHost\bin\Debug\"。

    ![Azure Data Lake Analytics U-SQL 偵錯設定 pdb 路徑](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-set-pdb-path.png)

在這些設定之後，使用 **F5** 和中斷點開始偵錯。 您也可以使用 Visual Studio 偵錯工具 (監看式、變數等等) 對問題進行疑難排解。

> [!NOTE]
> 每次修改程式碼以產生更新的 .pdb 檔案之後，都會重建組件原始程式碼專案。

## <a name="resubmit-the-job"></a>重新提交作業

偵錯之後，如果專案成功完成，[輸出] 視窗就會顯示下列訊息：

    The Program 'LocalVertexHost.exe' has exited with code 0 (0x0).

![Azure Data Lake Analytics U-SQL 偵錯成功](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-succeed.png)

重新提交失敗的作業：

1. 針對具有程式碼後置方案的作業，請將 C# 程式碼複製到程式碼後置來源檔 (通常是 `Script.usql.cs`)。

2. 針對具有組件的作業，請以滑鼠右鍵按一下偵錯方案中的組件原始程式碼專案，並將更新的 .dll 組件註冊到您的 Azure Data Lake 目錄中。

3. 重新提交 U-SQL 作業。

## <a name="next-steps"></a>後續步驟

- [U-SQL 可程式性指南](data-lake-analytics-u-sql-programmability-guide.md)
- [針對 Azure Data Lake Analytics 作業開發 U-SQL 使用者定義運算子](data-lake-analytics-u-sql-develop-user-defined-operators.md)
- [使用本機執行和 Azure Data Lake U-SQL SDK 對 U-SQL 作業進行測試和偵錯](data-lake-analytics-data-lake-tools-local-run.md)
- [如何針對異常的週期性作業進行疑難排解](data-lake-analytics-data-lake-tools-debug-recurring-job.md)
