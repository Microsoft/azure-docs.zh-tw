---
title: 以 Azure Data Lake Tools for Visual Studio Code 本機執行 U-SQL 及偵錯
description: 了解如何使用 Azure Data Lake Tools for Visual Studio Code 在本機執行 U-SQL 作業並對其進行偵錯。
services: data-lake-analytics
ms.service: data-lake-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jasonwhowell
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.topic: conceptual
ms.date: 07/14/2017
ms.openlocfilehash: bf98562224c2da770541f731ba93ec2e5dc1718d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041315"
---
# <a name="run-u-sql-and-debug-locally-in-visual-studio-code"></a>以 Visual Studio Code 在本機執行 U-SQL 及偵錯
本文說明如何在本機開發電腦上執行 U-SQL 作業，以便加速早期的編碼階段，或以 Visual Studio Code 在本機對程式碼進行偵錯。 如需如何使用 Azure Data Lake Tools for Visual Studio Code 的指示，請參閱[使用 Azure Data Lake Tools for Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)。 

## <a name="set-up-the-u-sql-local-run-environment"></a>設定 U-SQL 本機執行環境

1. 按 Ctrl+Shift+P 開啟命令選擇區，然後輸入 **ADL: Download Local Run Package** 下載套件。  

   ![下載 ADL LocalRun 相依性套件](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/downloadtheadllocalrunpackage.png)

2. 從 [輸出] 窗格中顯示的路徑找出相依性套件，然後安裝 BuildTools 和 Win10SDK 10240。 路徑範例如下：  
`C:\Users\xxx\AppData\Roaming\LocalRunDependency` 

   ![找出相依性套件](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/LocateDependencyPath.png)

   2.1 如果要安裝 **BuildTools**，按一下 LocalRunDependency 資料夾中的 visualcppbuildtools_full.exe，然後遵循精靈的指示。   

    ![安裝 BuildTools](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/InstallBuildTools.png)

   2.2 如果要安裝 **Win10SDK 10240**，按一下 LocalRunDependency/Win10SDK_10.0.10240_2 資料夾中的 sdksetup.exe，然後遵循精靈的指示。  

    ![安裝 Win10SDK 10240](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/InstallWin10SDK.png)

3. 設定環境變數。 將 **SCOPE_CPP_SDK** 環境變數設定為：  
`C:\Users\XXX\AppData\Roaming\LocalRunDependency\CppSDK_3rdparty`  


## <a name="start-the-local-run-service-and-submit-the-u-sql-job-to-a-local-account"></a>啟動本機執行服務並將 U-SQL 作業提交給本機帳戶 
如果您是初次使用，還沒[設定 U-SQL 本機執行環境](#set-up-the-u-sql-local-run-environment)，請使用 **ADL: Download Local Run Package** 下載本機執行套件。

1. 選取 Ctrl+Shift+P 來開啟命令選擇區，然後輸入 **ADL: Start Local Run Service**。   
2. 選取 [接受] 來首次接受 Microsoft 軟體授權條款。 

   ![接受 Microsoft 軟體授權條款](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/AcceptEULA.png)   
3. 隨即會開啟 CMD 主控台。 如果您是初次使用者，就必須輸入 **3**，然後找出用於資料輸入和輸出的本機資料夾路徑。 其他選項則可以使用預設值。 

   ![Data Lake Tools for Visual Studio Code 本機執行 CMD](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-run-cmd.png)
4. 選取 Ctrl+Shift+P 來開啟命令選擇區，輸入 **ADL: Submit Job**，然後選取 [Local] \(本機\) 以將作業提交給您的本機帳戶。

   ![Data Lake Tools for Visual Studio Code 選取本機](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-select-local.png)
5. 在提交作業後，您就可以檢視提交詳細資料。 若要檢視提交詳細資料，選取 [輸出] 視窗中的 [jobUrl]。 您也可以從 CMD 主控台檢視作業提交狀態。 如果您想要知道更多的作業詳細資料，請在 CMD 主控台中輸入 **7**。

   ![Data Lake Tools for Visual Studio Code 本機執行輸出](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-run-result.png)
   ![Data Lake Tools for Visual Studio Code 本機執行 CMD 狀態](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-localrun-cmd-status.png) 


## <a name="start-a-local-debug-for-the-u-sql-job"></a>開始 U-SQL 作業的本機偵錯  
初次使用者：

1. 如果您還沒[設定 U-SQL 本機執行環境](#set-up-the-u-sql-local-run-environment)，使用 **ADL: Download Local Run Package** 下載本機執行套件。
2. 如訊息方塊中的建議安裝 .NET Core SDK 2.0 (如果尚未安裝)。
 
  ![提醒安裝 Dotnet](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/remind-install-dotnet.png)
3. 如訊息方塊中的建議安裝 C# for Visual Studio Code (如果尚未安裝)。 按一下 [安裝] 繼續作業，然後重新啟動 VSCode。

    ![提醒安裝 C#](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/install-csharp.png)

遵循下列步驟執行本機偵錯：
  
1. 選取 Ctrl+Shift+P 來開啟命令選擇區，然後輸入 **ADL: Start Local Run Service**。 隨即會開啟 CMD 主控台。 請確定您已設定 **DataRoot**。
2. 在您的 C# 程式碼後置中設定中斷點。
3. 回到指令碼編輯器，以滑鼠右鍵按一下並選取 **ADL: Local Debug**。
    
   ![Data Lake Tools for Visual Studio Code 本機偵錯結果](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-debug-result.png)


## <a name="next-steps"></a>後續步驟
* [使用 Azure Data Lake Tools for Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
* [針對 VSCode 中的 Azure Data Lake Analytics 使用 Python、R、CSharp 開發 U-SQL](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
* [使用 PowerShell 開始使用 Data Lake Analytics](data-lake-analytics-get-started-powershell.md)
* [使用 Azure 入口網站開始使用 Data Lake Analytics](data-lake-analytics-get-started-portal.md)
* [使用適用於 Visual Studio 的 Data Lake 工具來開發 U-SQL 應用程式](data-lake-analytics-data-lake-tools-get-started.md)
* [使用 Data Lake Analytics (U-SQL) 目錄](data-lake-analytics-use-u-sql-catalog.md)
