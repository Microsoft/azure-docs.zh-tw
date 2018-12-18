---
title: 使用 Azure Data Lake Tools for Visual Studio Code
description: 了解如何使用 Azure Data Lake Tools for Visual Studio Code 來建立、測試和執行 U-SQL 指令碼。
services: data-lake-analytics
ms.service: data-lake-analytics
author: Jejiang
ms.author: jejiang
ms.reviewer: jasonwhowell
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.topic: conceptual
ms.date: 02/09/2018
ms.openlocfilehash: e470d27ccfea68f628b793f9afd695bb363e295a
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45630830"
---
# <a name="use-azure-data-lake-tools-for-visual-studio-code"></a>使用 Azure Data Lake Tools for Visual Studio Code

在此文章中，您將了解如何使用 Azure Data Lake Tools for Visual Studio Code (VS Code) 來建立、測試及執行 U-SQL 指令碼。 下列影片中也涵蓋此資訊︰

<a href="https://channel9.msdn.com/Series/AzureDataLake/Azure-Data-Lake-Tools-for-VSCode?term=ADL%20Tools%20for%20VSCode"><img src="./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-video.png"></a>

## <a name="prerequisites"></a>先決條件

Azure Data Lake Tools for VS Code 支援 Windows、Linux 與 macOS。 U-SQL 本機執行與本機偵錯僅適用於 Windows。

- [Visual Studio Code](https://www.visualstudio.com/products/code-vs.aspx)

針對 MacOS 和 Linux：
- [.NET Core SDK 2.0](https://www.microsoft.com/net/download/core)
- [Mono 5.2.x](http://www.mono-project.com/download/)

## <a name="install-azure-data-lake-tools"></a>安裝 Azure Data Lake 工具

安裝完先決條件之後，您就可以安裝 Azure Data Lake Tools for VS Code。

**安裝 Azure Data Lake Tools**

1. 開啟 Visual Studio Code。
2. 在左窗格中，選取 [延伸模組]。 在搜尋方塊中輸入 **Azure Data Lake Tools**。
3. 選取 [Azure Data Lake Tools] 旁的 [安裝]。 

   ![Data Lake Tools 的安裝選取項目](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extensions.png)

   幾秒鐘後，[安裝] 按鈕會變為 [重新載入]。
4. 選取 [重新載入] 以啟用 [Azure Data Lake Tools] 延伸模組。
5. 選取 [重新載入視窗] 進行確認。 您會在 [延伸模組] 窗格中看到 [Azure Data Lake Tools]。

 
## <a name="activate-azure-data-lake-tools"></a>啟動 Azure Data Lake Tools
建立一個 .usql 檔案或開啟現有的 .usql 檔案，以啟用延伸模組。 


## <a name="work-with-u-sql"></a>使用 U-SQL

若要使用 U-SQL，您需要開啟 U-SQL 檔案或資料夾。

**開啟範例指令碼**

開啟命令選擇區 (Ctrl+Shift+P)，然後輸入 **ADL: Open Sample Script**。 它會開啟此範例的另一個執行個體。 您也可以在此執行個體上編輯、設定及提交指令碼。

**開啟 U-SQL 專案的資料夾**

1. 從 Visual Studio Code 選取 [檔案] 功能表，然後選取 [開啟資料夾]。
2. 指定資料夾，然後選取 [選取資料夾]。
3. 選取 [檔案] 功能表，然後選取 [新增]。 專案中就會加入一個 Untitled-1 檔案。
4. 在 Untitled-1 檔案中輸入以下程式碼：

        @departments  = 
            SELECT * FROM 
                (VALUES
                    (31,    "Sales"),
                    (33,    "Engineering"), 
                    (34,    "Clerical"),
                    (35,    "Marketing")
                ) AS 
                      D( DepID, DepName );
         
        OUTPUT @departments
            TO "/Output/departments.csv"
        USING Outputters.Csv();

    這個指令碼會在 /output 資料夾中建立 departments.csv 檔案並納入一些資料。

5. 在開啟的資料夾中，將檔案儲存為 **myUSQL.usql**。

**編譯 U-SQL 指令碼**

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。 
2. 輸入 **ADL: Compile Script**。 編譯結果會出現在 [輸出] 視窗中。 您可以也在指令碼檔案上按一下滑鼠右鍵，然後選取 [ADL: Compile Script] 來編譯 U-SQL 作業。 編譯結果會出現在 [輸出] 窗格中。
 
**提交 U-SQL 指令碼**

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。 
2. 輸入 **ADL: Submit Job**。 您也可以在指令碼檔案上按一下滑鼠右鍵，然後選取 [ADL: Submit Job]。 

在提交 U-SQL 作業後，提交記錄會出現在 VS Code 的 [輸出] 視窗中。 作業檢視會出現在右窗格中。 如果提交成功，則作業 URL 也會出現。 您可以在網頁瀏覽器中開啟作業 URL 來追蹤即時的作業狀態。 

在作業檢視的 [摘要] 索引標籤上，您可以看到作業詳細資料。 主要功能包括重新提交指令碼、複製指令碼，以及在入口網站中開啟。 在作業檢視的 [資料] 索引標籤上，您可以參考輸入檔案、輸出檔案及資源檔案。 您可以將檔案下載到本機電腦。

![作業檢視中的 [摘要] 索引標籤](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-summary.png)

![作業檢視中的 [資料] 索引標籤](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-data.png)

**設定預設內容**

如果您尚未個別設定檔案的參數，便可以設定預設內容以將此設定套用至所有指令碼檔案。

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。 
2. 輸入 **ADL: Set Default Context**。 或在指令碼編輯器上按一下滑鼠右鍵，然後選取 [ADL: Set Default Context]。
3. 選擇您想要的帳戶、資料庫及結構描述。 此設定會儲存到 xxx_settings.json 設定檔。

   ![設定為預設內容的帳戶、資料庫及結構描述](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-sequence.png)

**設定指令碼參數**

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。 
2. 輸入 **ADL: Set Script Parameters**。
3. xxx_settings.json 檔案隨即開啟，其中含有下列屬性：

   - **account**：您 Azure 訂用帳戶底下編譯和執行 U-SQL 作業所需的 Azure Data Lake Analytics 帳戶。 您必須在編譯和執行 U-SQL 作業之前，先設定電腦帳戶。
   - **database**：您帳戶底下的資料庫。 預設值為 **master**。
   - **schema**：您資料庫底下的結構描述。 預設值為 **dbo**。
   - **optionalSettings**：
        - **priority**︰優先順序範圍是從 1 到 1000，其中 1 是最高優先順序。 預設值為 **1000**。
        - **degreeOfParallelism**︰平行處理原則的範圍是從 1 到 150。 預設值為您 Azure Data Lake Analytics 帳戶中允許的平行處理原則上限。 

   ![JSON 檔案的內容](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-setting.png)
      
> [!NOTE] 
> 在儲存設定後，如果您尚未設定預設內容，帳戶、資料庫和結構描述資訊就會出現在對應之 .usql 檔案左下角的狀態列上。

**設定 Git 忽略**

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。 
2. 輸入 **ADL: Set Git Ignore**。

   - 如果您的 VS Code 工作資料夾中沒有 **.gitIgnore** 檔案，系統就會在您的資料夾中建立名為 **.gitIgnore** 的檔案。 預設會在檔案中新增四個項目 (**usqlCodeBehindReference**、**usqlCodeBehindGenerated**、**.cache**、**obj**) 預設會新增至檔案。 您可以視需要進行更多更新。
   - 如果您的 VS Code 工作資料夾中已經有 **.gitIgnore** 檔案，且檔案中未包含四個項目 (**usqlCodeBehindReference**、**usqlCodeBehindGenerated**、**.cache**、**obj**)，工具就會在您的 **.gitIgnore** 檔案中新增這四個項目。

   ![.gitIgnore 檔案中的項目](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-gitignore.png)


## <a name="work-with-code-behind-files-c-sharp-python-and-r"></a>使用程式碼後置檔案：C Sharp、Python 和 R

Azure Data Lake Tools 支援多個自訂程式碼。 如需相關指示，請參閱[在 VS Code 中使用 Python、R 和 C Sharp 來開發適用於 Azure Data Lake Analytics 的 U-SQL](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)。

## <a name="work-with-assemblies"></a>使用組件

如需有關開發組件的資訊，請參閱[針對 Azure Data Lake Analytics 作業開發 U-SQL 組件](data-lake-analytics-u-sql-develop-assemblies.md)。

您可以使用 Data Lake Tools 在 Data Lake Analytics 目錄中註冊自訂程式碼組件。

**註冊組件**

您可以透過 **ADL: Register Assembly** 或 **ADL: Register Assembly (Advanced)** 命令來註冊組件。

**透過 ADL: Register Assembly 命令來進行註冊**
1.  選取 Ctrl+Shift+P 以開啟命令選擇區。
2.  輸入 **ADL: Register Assembly**。 
3.  指定本機組件路徑。 
4.  選取 Data Lake Analytics 帳戶。
5.  選取資料庫。

入口網站會在瀏覽器中開啟，並顯示組件註冊程序。  

有另一個可觸發 **ADL: Register Assembly** 命令的更便利方式，就是對「檔案總管」中的 .dll 檔案按一下滑鼠右鍵。 

**透過 ADL: Register Assembly (Advanced) 命令來進行註冊**
1.  選取 Ctrl+Shift+P 以開啟命令選擇區。
2.  輸入 **ADL: Register Assembly (Advanced)**。 
3.  指定本機組件路徑。 
4.  隨即會顯示 JSON 檔案。 請視需要檢閱並編輯組件相依性及資源參數。 指示會顯示在 [輸出] 視窗中。 若要繼續進行組件註冊，請儲存 (Ctrl + S) JSON 檔案。

    ![含有組件相依性和資源參數的 JSON 檔案](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-register-assembly-advance.png)
    
>[!NOTE]
>- Azure Data Lake Tools 會自動偵測 DLL 是否有任何相依項目。 在偵測到相依項目後，就會在 JSON 檔案中顯示這些相依項目。 
>- 您可以在註冊組件時上傳 DLL 資源 (例如 .txt、.png 和 .csv)。 

另一個可供觸發 **ADL: Register Assembly (Advanced)** 命令的方式，是對檔案總管中的 .dll 檔案按一下滑鼠右鍵。 

下列 U-SQL 程式碼示範如何呼叫組件。 在此範例中，組件名稱是 *test*。


        REFERENCE ASSEMBLY [test];

        @a = 
            EXTRACT 
                Iid int,
            Starts DateTime,
            Region string,
            Query string,
            DwellTime int,
            Results string,
            ClickedUrls string 
            FROM @"Sample/SearchLog.txt" 
            USING Extractors.Tsv();

        @d =
            SELECT DISTINCT Region 
            FROM @a;

        @d1 = 
            PROCESS @d
            PRODUCE 
                Region string,
            Mkt string
            USING new USQLApplication_codebehind.MyProcessor();

        OUTPUT @d1 
            TO @"Sample/SearchLogtest.txt" 
            USING Outputters.Tsv();


## <a name="use-u-sql-local-run-and-local-debug-for-windows-users"></a>針對 Windows 使用者使用 U-SQL 本機執行和本機偵錯
U-SQL 本機執行會先測試您的本機資料並在本機驗證您的指令碼，然後才將您的程式碼發行至 Data Lake Analytics。 您可以使用本機偵錯功能先完成下列工作，再將您的程式碼提交給 Data Lake Analytics： 
- 偵錯您的 C# 程式碼後置。 
- 逐步執行程式碼。 
- 在本機驗證您的指令碼。

本機執行與本機偵錯功能僅適用於 Windows 環境，在 macOS 與 Linux 型作業系統上不支援。

如需本機執行和本機偵錯的指示，請參閱[使用 Visual Studio Code 來進行 U-SQL 本機執行和本機偵錯](data-lake-tools-for-vscode-local-run-and-debug.md)。


## <a name="connect-to-azure"></a>連接到 Azure

您必須先連線到 Azure 帳戶，才能在 Data Lake Analytics 中編譯和執行 U-SQL 指令碼。

<b id="sign-in-by-command">使用命令來連線至 Azure</b>

1.  選取 Ctrl+Shift+P 以開啟命令選擇區。 
2.  輸入 **ADL: Login**。 登入資訊會出現在右下方。

    ![輸入登入命令](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login.png)

    ![登入和驗證的相關通知](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-login-info.png)

3.  選取 [複製及開啟] 以開啟[登入網頁](https://aka.ms/devicelogin)。 將程式碼貼到方塊中，然後選取 [繼續]。

    ![登入網頁](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login-paste-code.png)  
     
4.  依照網頁上的指示登入。 當您已連線時，您的 Azure 帳戶名稱會出現在 VS Code 視窗左下角的狀態列上。 

> [!NOTE] 
>- 如果您未登出，下次 Data Lake Tools 就會自動將您登入。
>- 如果您的帳戶已啟用雙因素驗證，建議您使用電話驗證而非使用 PIN 碼。


若要登出，請輸入命令 **ADL: Logout**。

**從總管連線至 Azure**

展開 [AZURE DATALAKE]，選取 [登入 Azure]，然後依照[使用命令來連線至 Azure](#sign-in-by-command) 的步驟 3 和步驟 4 進行操作。

![總管中的 [登入 Azure] 選取項目](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-sign-in-from-explorer.png )  

您無法從總管登出。 若要登出，請參閱[使用命令來連線至 Azure](#sign-in-by-command)。


## <a name="create-an-extraction-script"></a>建立擷取指令碼 
您可以使用 **ADL: Create EXTRACT Script** 命令或從 Azure Data Lake 總管，建立 .csv、.tsv 及 .txt 檔案的擷取指令碼。

**使用命令來建立擷取指令碼**

1. 選取 Ctrl+Shift+P 來開啟命令選擇區，然後輸入 **ADL: Create EXTRACT Script**。
2. 指定 Azure 儲存體檔案的完整路徑，然後選取 Enter 鍵。
3. 選取帳戶。
4. 針對 .txt 檔案，請選取用來擷取檔案的分隔符號。 

![建立擷取指令碼的程序](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-process.png)

系統會根據您的輸入來產生擷取指令碼。 對於無法偵測資料行的指令碼，請選擇兩個選項的其中一個。 如果未選擇，則只會產生一個指令碼。

![建立擷取指令碼的結果](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-result.png)

**從總管建立擷取指令碼**

有另一個可建立擷取指令碼的方式，就是在 Azure Data Lake Store 或 Azure Blob 儲存體中透過 .csv、.tsv 或 .txt 檔案的滑鼠右鍵 (捷徑) 功能表來進行。

![捷徑功能表中的 [建立 EXTRACT 指令碼] 命令](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu.png)

## <a name="integrate-with-azure-data-lake-analytics-through-a-command"></a>透過命令與 Azure Data Lake Analytics 整合

您可以存取 Azure Data Lake Analytics 資源來列出帳戶、存取中繼資料，以及檢視分析作業。 

**列出 Azure 訂用帳戶下的 Azure Data Lake Analytics 帳戶**

1. 選取 Ctrl+Shift+P 以開啟命令選擇區。
2. 輸入 **ADL: List Accounts**。 帳戶會出現在 [輸出] 窗格中。

**存取 Azure Data Lake Analytics 中繼資料**

1.  選取 Ctrl+Shift+P，然後輸入 **ADL: List Tables**。
2.  選取其中一個 Data Lake Analytics 帳戶。
3.  選取其中一個 Data Lake Analytics 資料庫。
4.  選取其中一個結構描述。 您就會看到資料表清單。

**檢視 Azure Data Lake Analytics 作業**
1.  開啟命令選擇區 (Ctrl+Shift+P)，然後選取 [ADL: Show Job]。 
2.  選取 Data Lake Analytics 帳戶或本機帳戶。 
3.  等候帳戶的作業清單出現。
4.  從作業清單中選取作業。 Data Lake Tools 會在右窗格開啟作業檢視，並在 VS Code 輸出中顯示一些資訊。

    ![作業清單](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-show-job.png)

## <a name="integrate-with-azure-data-lake-store-through-a-command"></a>透過命令與 Azure Data Lake Store 整合

您可以使用 Azure Data Lake Store 相關命令來進行下列作業：
 - [瀏覽 Azure Data Lake Store 資源](#list-the-storage-path) 
 - [預覽 Azure Data Lake Store 檔案](#preview-the-storage-file) 
 - [在 VS Code 中直接將檔案上傳至 Azure Data Lake Store](#upload-file-or-folder)
 - [在 VS Code 中直接從 Azure Data Lake Store 下載檔案](#download-file)

### <a name="list-the-storage-path"></a>列出儲存體路徑 

**透過命令選擇區來列出儲存體路徑**

1. 以滑鼠右鍵按一下指令碼編輯器，然後選取 [ADL: List Path]。
2. 選擇清單中的資料夾，或是選取 [輸入路徑] 或 [從根路徑瀏覽]。 (我們使用 [輸入路徑] 作為範例)。 
3. 選取您的 Data Lake Analytics 帳戶。
4. 瀏覽或輸入儲存體資料夾路徑 (例如 /output/)。  

命令選擇區會根據您的輸入列出路徑資訊。

![儲存體路徑結果](./media/data-lake-analytics-data-lake-tools-for-vscode/list-storage-path.png)

有一個可列出相對路徑的更便利方式，就是透過捷徑功能表。

**透過捷徑功能表來列出儲存體路徑**

在路徑字串上按一下滑鼠右鍵，然後選取 [列出路徑]。

![捷徑功能表上的 [列出路徑]](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-right-click-path.png)


### <a name="preview-the-storage-file"></a>預覽儲存體檔案

1. 以滑鼠右鍵按一下指令碼編輯器，然後選取 [ADL: Preview File]。
2. 選取您的 Data Lake Analytics 帳戶。 
3. 輸入 Azure 儲存體檔案路徑 (例如 /output/SearchLog.txt)。 

檔案會在 VS Code 中開啟。

![儲存體檔案的預覽步驟和結果](./media/data-lake-analytics-data-lake-tools-for-vscode/preview-storage-file.png)

有另一個可預覽檔案的方式，就是在指令碼編輯器中，透過檔案完整路徑或檔案相對路徑上的捷徑功能表來進行。 

### <a name="upload-a-file-or-folder"></a>上傳檔案或資料夾

1. 以滑鼠右鍵按一下指令碼編輯器，然後選取 [上傳檔案] 或 [上傳資料夾]。
2. 如果您已選取 [上傳檔案]，請選擇一或多個檔案。或者，如果您已選取 [上傳資料夾]，則請選擇整個資料夾。 然後選取 [上傳]。 
3. 選擇清單中的儲存體資料夾，或是選取 [輸入路徑] 或 [從根路徑瀏覽]。 (我們使用 [輸入路徑] 作為範例)。 
4. 選取您的 Data Lake Analytics 帳戶。 
5. 瀏覽或輸入儲存體資料夾路徑 (例如 /output/)。 
6. 選取 [選擇目前資料夾] 來指定您的上傳目的地。

![檔案或資料夾的上傳步驟和結果](./media/data-lake-analytics-data-lake-tools-for-vscode/upload-file.png)    

有另一個可將檔案上傳到儲存體的方式，就是在指令碼編輯器中，透過檔案完整路徑或檔案相對路徑上的捷徑功能表來進行。

您可以[監視上傳狀態](#check-storage-tasks-status)。


### <a name="download-a-file"></a>下載檔案 
您可以使用 **ADL: Download File** 或 **ADL: Download File (Advanced)** 命令來下載檔案。

**透過 ADL: Download File (Advanced) 命令來下載檔案**
1. 以滑鼠右鍵按一下指令碼編輯器，然後選取 [Download File (Advanced)]。
2. VS Code 會顯示一個 JSON 檔案。 您可以輸入檔案路徑，然後同時下載多個檔案。 指示會顯示在 [輸出] 視窗中。 若要繼續下載一或多個檔案，請儲存 (Ctrl+S) JSON 檔案。

    ![含有檔案下載路徑的 JSON 檔案](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-files.png)

[輸出] 視窗會顯示檔案下載狀態。

![含有下載狀態的 [輸出] 視窗](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-file-result.png)     

您可以[監視下載狀態](#check-storage-tasks-status)。

**透過 ADL: Download File 命令來下載檔案**

1. 在指令碼編輯器上按一下滑鼠右鍵，選取 [下載檔案]，然後從 [選取資料夾] 對話方塊中選取目的地資料夾。
2. 選擇清單中的資料夾，或是選取 [輸入路徑] 或 [從根路徑瀏覽]。 (我們使用 [輸入路徑] 作為範例)。 
3. 選取您的 Data Lake Analytics 帳戶。 
4. 瀏覽或輸入儲存體資料夾路徑 (例如：/output/)，然後選擇要下載的檔案。

![檔案的下載步驟和結果](./media/data-lake-analytics-data-lake-tools-for-vscode/download-file.png) 

有另一個可下載儲存體檔案的方式，就是在指令碼編輯器中，透過檔案完整路徑或檔案相對路徑上的捷徑功能表來進行。

您可以[監視下載狀態](#check-storage-tasks-status)。

### <a name="check-storage-tasks-status"></a>檢查儲存體工作的狀態
上傳和下載狀態會顯示在狀態列上。 請選取狀態列，然後狀態就會顯示在 [輸出] 索引標籤上。

![狀態列和輸出詳細資料](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-status.png)


## <a name="integrate-with-azure-data-lake-analytics-from-the-explorer"></a>從總管與 Azure Data Lake Analytics 整合

登入之後，您 Azure 帳戶的所有訂用帳戶都會列在左窗格中 [AZURE DATALAKE] 底下。 

![Data Lake 總管](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer.png)

### <a name="data-lake-analytics-metadata-navigation"></a>Data Lake Analytics 中繼資料瀏覽

展開您的 Azure 訂用帳戶。 在 [U-SQL 資料庫] 節點底下，您可以瀏覽 U-SQL 資料庫並檢視資料夾，例如 [結構描述]、[認證]、[組件]、[資料表] 及 [索引]。

### <a name="data-lake-analytics-metadata-entity-management"></a>Data Lake Analytics 中繼資料實體管理

展開 [U-SQL 資料庫]。 您可以建立資料庫、結構描述、資料表、資料表類型、索引或統計資料，方法是在對應的節點上按一下滑鼠右鍵，然後從捷徑功能表上選取 [要建立的指令碼]。 在已開啟的指令碼頁面上，根據您的需求編輯指令碼。 接著，在指令碼上按一下滑鼠右鍵並選取 [ADL: Submit Job] 來提交作業。 

建立完項目之後，在節點上按一下滑鼠右鍵，然後選取 [重新整理] 來顯示項目。 您也可以刪除項目，方法是在項目上按一下滑鼠右鍵，然後選取 [刪除]。

![Data Lake 總管中捷徑功能表上的 [要建立的指令碼] 命令](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create.png)

![新項目的指令碼頁面](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create-snippet.png)

### <a name="data-lake-analytics-assembly-registration"></a>Data Lake Analytics 組件註冊

您可以在對應的資料庫中註冊組件，方法是在 [組件] 節點上按一下滑鼠右鍵，然後選取 [註冊組件]。

![[組件] 節點捷徑功能表上的 [註冊組件] 命令](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer-register-assembly.png)

## <a name="integrate-with-azure-data-lake-store-from-the-explorer"></a>從總管與 Azure Data Lake Store 整合

瀏覽至 [Data Lake Store]：

- 您可以在資料夾節點上按一下滑鼠右鍵，然後使用捷徑功能表上的 [重新整理]、[刪除]、[上傳]、[上傳資料夾]、[複製相對路徑] 及 [複製完整路徑] 命令。

   ![Data Lake 總管中資料夾節點上的捷徑功能表命令](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-folder-menu.png)

- 您可以在檔案節點上按一下滑鼠右鍵，然後使用捷徑功能表上的 [預覽]、[下載]、[刪除]、[建立 EXTRACT 指令碼] (只適用於 CSV、TSV 和 TXT 檔案)，以及 [複製相對路徑] 和 [複製完整路徑] 命令。

   ![Data Lake 總管中檔案節點上的捷徑功能表命令](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-extract.png)

## <a name="integrate-with-azure-blob-storage-from-the-explorer"></a>從總管與 Azure Blob 儲存體整合

瀏覽至 Blob 儲存體：

- 您可以在 Blob 容器節點上按一下滑鼠右鍵，然後使用捷徑功能表上的 [重新整理]、[刪除 Blob 容器] 及 [上傳 Blob] 命令。

   ![Blob 儲存體底下 Blob 容器節點的捷徑功能表命令](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-blob-container-node.png)

- 您可以在資料夾節點上按一下滑鼠右鍵，然後使用捷徑功能表上的 [重新整理]和 [上傳 Blob] 命令。

   ![Blob 儲存體底下資料夾節點的捷徑功能表命令](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-folder-node.png)

- 您可以在檔案節點上按一下滑鼠右鍵，然後使用捷徑功能表上的 [預覽/編輯]、[下載]、[刪除]、[建立 EXTRACT 指令碼] (只適用於 CSV、TSV 和 TXT 檔案)，以及 [複製相對路徑] 和 [複製完整路徑] 命令。

    ![Blob 儲存體底下檔案節點的捷徑功能表命令](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu-2.png)

## <a name="open-the-data-lake-explorer-in-the-portal"></a>在入口網站中開啟 Data Lake 總管
1. 選取 Ctrl+Shift+P 以開啟命令選擇區。
2. 輸入 **Open Web Azure Storage Explorer**，或在指令碼編輯器中的相對路徑或完整路徑上按一下滑鼠右鍵，然後選取 [Open Web Azure Storage Explorer]。
3. 選取 Data Lake Analytics 帳戶。

Data Lake Tools 會在 Azure 入口網站中開啟 Azure 儲存體路徑。 您可以找到該路徑，並從網路預覽檔案。

## <a name="additional-features"></a>其他功能

Data Lake Tools for VSCode 支援下列功能︰

-   **IntelliSense 自動完成**：在關鍵字、方法和變數等項目周圍的快顯視窗中會顯示建議。 不同圖示代表不同類型的物件：

    - Scala 資料類型
    - 複雜資料類型
    - 內建 UDT
    - .NET 集合和類別
    - C# 運算式
    - 內建 C# UDF、UDO 和 UDAAG 
    - U-SQL 函式
    - U-SQL 時間範圍函式
 
    ![IntelliSense 物件類型](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-objects.png)
 
-   **Data Lake Analytics 中繼資料的 IntelliSense 自動完成**：Data Lake Tools 會將 Data Lake Analytics 中繼資料資訊下載到本機。 IntelliSense 功能會從 Data Lake Analytics 中繼資料自動填入物件。 這些物件包括資料庫、結構描述、資料表、檢視、資料表值函式、程序及 C# 組件。
 
    ![IntelliSense 中繼資料](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-metastore.png)

-   **IntelliSense 錯誤標記**：Data Lake Tools 會為 U-SQL 和 C# 的編輯錯誤加上底線。 
-   **語法醒目提示**：Data Lake Tools 會使用顏色來區分項目，例如變數、關鍵字、資料類型及函式。 

    ![帶有各種顏色的語法](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-syntax-highlights.png)

> [!NOTE]
> 建議您升級至 Azure Data Lake Tools for Visual Studio 2.3.3000.4 版或更新版本。 舊版目前已淘汰，不再提供下載。  
   
## <a name="next-steps"></a>後續步驟
- [在 VS Code 中使用 Python、R 和 C Sharp 來開發適用於 Azure Data Lake Analytics 的 U-SQL](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [使用 Visual Studio Code 來進行 U-SQL 本機執行和本機偵錯](data-lake-tools-for-vscode-local-run-and-debug.md)
- [教學課程：開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [教學課程：使用 Data Lake Tools for Visual Studio 開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
