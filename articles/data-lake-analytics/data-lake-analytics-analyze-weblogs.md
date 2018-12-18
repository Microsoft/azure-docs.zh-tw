---
title: 使用 Azure Data Lake Analytics 來分析網站記錄
description: 了解如何使用資料湖分析來分析網站記錄。
services: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 3a196735-d0d9-4deb-ba68-c4b3f3be8403
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 12/05/2016
ms.openlocfilehash: 5d25aed196dd6b4ce92ae1cf18e556d1c32295d3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051962"
---
# <a name="analyze-website-logs-using-azure-data-lake-analytics"></a>使用 Azure Data Lake Analytics 來分析網站記錄
了解如何使用資料湖分析來分析網站記錄，特別是找出哪些訪客來源在嘗試瀏覽網站時遇到錯誤。

## <a name="prerequisites"></a>必要條件
* **Visual Studio 2015 或 Visual Studio 2013**。
* **[Visual Studio 適用的 Data Lake 工具](http://aka.ms/adltoolsvs)**。

    安裝適用於 Visual Studio 的 Data Lake 工具之後，您會在 Visual Studio 的 [工具] 功能表中看到 [Datat Lake] 項目：

    ![U-SQL Visual Studio 功能表](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-menu.png)
* **對於資料湖分析和適用於 Visual Studio 的資料湖工具有基本的認識**。 若要開始使用，請參閱：

  * [使用適用於 Visual Studio 的資料湖工具開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)。
* **資料湖分析帳戶。**  請參閱[建立 Azure Data Lake Analytics 帳戶可節省時間](data-lake-analytics-get-started-portal.md)。
* **安裝範例資料。** 在 Azure 入口網站中開啟 Data Lake Analytics 帳戶，按一下左側功能表上的 [指令碼範例]，然後再按一下 [複製範例資料]。 

## <a name="connect-to-azure"></a>連接到 Azure
您必須先連接至 Azure，然後才能建置及測試任何 U-SQL 指令碼。

**連接到資料湖分析**

1. 開啟 Visual Studio。
2. 按一下 [Data Lake] > [選項和設定]。
3. 按一下 [登入]，或者如果已有其他人登入則按一下 [變更使用者]，並遵循指示。
4. 按一下 [確定]  關閉 [選項和設定] 對話方塊。

**瀏覽您的資料湖分析帳戶**

1. 從 Visual Studio 中，按 **CTRL+ALT+S**，開啟 [伺服器總管]。
2. 在 [伺服器總管] 中展開 [Azure]，然後展開 [Data Lake Analytics]。 如果有資料湖分析帳戶，您就會看到其清單。 您無法從 Visual Studio 建立資料湖分析帳戶。 若要建立帳戶，請參閱[使用 Azure 入口網站開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md) 或[使用 Azure PowerShell 開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-powershell.md)。

## <a name="develop-u-sql-application"></a>開發 U-SQL 應用程式
U-SQL 應用程式基本上是 U-SQL 指令碼。 若要深入了解 U-SQL，請參閱 [開始使用 U-SQL 語言](data-lake-analytics-u-sql-get-started.md)。

您可以加入其他使用者定義的運算子至應用程式。  如需詳細資訊，請參閱 [針對資料湖分析工作開發 U-SQL 使用者定義運算子](data-lake-analytics-u-sql-develop-user-defined-operators.md)。

**建立並提交資料湖分析工作**

1. 按一下 [檔案] > [新增] > [專案]。
2. 選取 [U-SQL 專案] 類型。

    ![新的 U-SQL Visual Studio 專案](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-new-project.png)
3. 按一下 [確定]。 Visual Studio 會建立具有 Script.usql 檔案的方案。
4. 在 Script.usql 檔案中輸入下列指令碼：

        // Create a database for easy reuse, so you don't need to read from a file every time.
        CREATE DATABASE IF NOT EXISTS SampleDBTutorials;

        // Create a Table valued function. TVF ensures that your jobs fetch data from the weblog file with the correct schema.
        DROP FUNCTION IF EXISTS SampleDBTutorials.dbo.WeblogsView;
        CREATE FUNCTION SampleDBTutorials.dbo.WeblogsView()
        RETURNS @result TABLE
        (
            s_date DateTime,
            s_time string,
            s_sitename string,
            cs_method string,
            cs_uristem string,
            cs_uriquery string,
            s_port int,
            cs_username string,
            c_ip string,
            cs_useragent string,
            cs_cookie string,
            cs_referer string,
            cs_host string,
            sc_status int,
            sc_substatus int,
            sc_win32status int,
            sc_bytes int,
            cs_bytes int,
            s_timetaken int
        )
        AS
        BEGIN

            @result = EXTRACT
                s_date DateTime,
                s_time string,
                s_sitename string,
                cs_method string,
                cs_uristem string,
                cs_uriquery string,
                s_port int,
                cs_username string,
                c_ip string,
                cs_useragent string,
                cs_cookie string,
                cs_referer string,
                cs_host string,
                sc_status int,
                sc_substatus int,
                sc_win32status int,
                sc_bytes int,
                cs_bytes int,
                s_timetaken int
            FROM @"/Samples/Data/WebLog.log"
            USING Extractors.Text(delimiter:' ');
            RETURN;
        END;

        // Create a table for storing referrers and status
        DROP TABLE IF EXISTS SampleDBTutorials.dbo.ReferrersPerDay;
        @weblog = SampleDBTutorials.dbo.WeblogsView();
        CREATE TABLE SampleDBTutorials.dbo.ReferrersPerDay
        (
            INDEX idx1
            CLUSTERED(Year ASC)
            DISTRIBUTED BY HASH(Year)
        ) AS

        SELECT s_date.Year AS Year,
            s_date.Month AS Month,
            s_date.Day AS Day,
            cs_referer,
            sc_status,
            COUNT(DISTINCT c_ip) AS cnt
        FROM @weblog
        GROUP BY s_date,
                cs_referer,
                sc_status;

    若要了解 U-SQL，請參閱 [開始使用資料湖分析 U-SQL 語言](data-lake-analytics-u-sql-get-started.md)。    
5. 將新的 U-SQL 指令碼加入至專案並輸入下列資訊：

        // Query the referrers that ran into errors
        @content =
            SELECT *
            FROM SampleDBTutorials.dbo.ReferrersPerDay
            WHERE sc_status >=400 AND sc_status < 500;

        OUTPUT @content
        TO @"/Samples/Outputs/UnsuccessfulResponses.log"
        USING Outputters.Tsv();
6. 依序切換回至第一個 U-SQL 指令碼和 [提交]  按鈕，指定您的分析帳戶。
7. 從 [方案總管] 中，在 [Script.usql] 上按一下滑鼠右鍵，然後按一下 [建置指令碼]。 確認 [輸出] 窗格中的結果。
8. 從 [方案總管] 中，在 [Script.usql] 上按一下滑鼠右鍵，然後按一下 [提交指令碼]。
9. 確認 [分析帳戶] 是在您想要執行工作的帳戶，然後按一下 [提交]。 提交作業完成時，[適用於 Visual Studio 的資料湖工具結果] 視窗中便會出現提交結果和工作連結。
10. 請等待工作成功完成。  如果工作失敗，很可能是因為遺漏了原始檔。  請參閱本教學課程的＜必要條件＞一節。 如需其他疑難排解資訊，請參閱 [監視和疑難排解 Azure 資料湖分析工作](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)。

    工作完成之後，您會看到下列畫面：

    ![資料湖分析分析 weblog 網站記錄](./media/data-lake-analytics-analyze-weblogs/data-lake-analytics-analyze-weblogs-job-completed.png)
11. 現在針對 **Script1.usql**重複步驟 7 - 10。

**查看作業輸出**

1. 在 [伺服器總管] 中，依序展開 [Azure]、[Data Lake Analytics]、您的 Data Lake Analytics 帳戶，以及 [儲存體帳戶]，然後用滑鼠右鍵按一下預設的 Data Lake 儲存體帳戶，再按一下 [總管]。
2. 按兩下 [範例] 來開啟資料夾，然後再連按兩下 [輸出]。
3. 按兩下 **UnsuccessfulResponsees.log**。
4. 您也可以按兩下工作的圖形檢視內的輸出檔，直接瀏覽至輸出。

## <a name="see-also"></a>另請參閱
若要使用不同的工具開始使用資料湖分析，請參閱：

* [使用 Azure 入口網站開始使用 Data Lake Analytics](data-lake-analytics-get-started-portal.md)
* [透過 Azure PowerShell 開始使用 Data Lake Analytics](data-lake-analytics-get-started-powershell.md)
* [使用 .NET SDK 開始使用資料湖分析](data-lake-analytics-get-started-net-sdk.md)
