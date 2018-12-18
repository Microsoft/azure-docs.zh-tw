---
title: 在本機電腦上執行 Azure Data Lake U-SQL 指令碼
description: 了解如何使用 Azure Data Lake Tools for Visual Studio 在本機電腦上執行 U-SQL 作業。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: 8f110a88558b4479d8fdadb2967c7dedeca60ce9
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43043552"
---
# <a name="run-u-sql-scripts-on-your-local-machine"></a>在本機電腦上執行 U-SQL 指令碼

在開發 U-SQL 指令碼時，您可以藉由在本機執行指令碼來節省時間和金錢。 Azure Data Lake Tools for Visual Studio 可支援在本機電腦上執行 U-SQL 指令碼。 

## <a name="basic-concepts-for-local-runs"></a>本機執行的基本概念

以下圖表顯示的是本機執行的元件，以及元件對應至雲端執行的方式。

|元件|本機執行|雲端執行|
|---------|---------|---------|
|儲存體|本機資料根資料夾|預設 Azure Data Lake Store 帳戶|
|計算|U-SQL 本機執行引擎|Azure Data Lake Analytics 服務|
|執行環境|本機電腦上的工作目錄|Azure Data Lake Analytics 叢集|

下列各節會提供關於本機執行元件的詳細資訊。

### <a name="local-data-root-folders"></a>本機資料根資料夾

本機根資料夾是本機計算帳戶的**本機存放區**。 在本機電腦上，本機檔案系統中的任何資料夾都可以作為本機資料根資料夾。 它就和 Data Lake Analytics 帳戶的預設 Azure Data Lake Store 帳戶一樣。 切換到不同的資料根資料夾，就如同切換到不同的預設存放區帳戶。 

資料根資料夾的用途如下︰
- 儲存中繼資料。 範例有資料庫、資料表，資料表值函式及組件。
- 查詢在 U-SQL 指令碼中定義為相對路徑的輸入和輸出路徑。 藉由使用相對路徑，您可以更輕鬆地將 U-SQL 指令碼部署至 Azure。

### <a name="u-sql-local-run-engines"></a>U-SQL 本機執行引擎

U-SQL 本機執行引擎是 U-SQL 作業的**本機計算帳戶**。 使用者能透過 Azure Data Lake Tools for Visual Studio 在本機執行 U-SQL 作業。 本機執行也支援透過 Azure Data Lake U-SQL SDK 命令列和程式設計介面來進行。 深入了解 [Azure Data Lake U-SQL SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/)。

### <a name="working-directories"></a>工作目錄

在執行 U-SQL 指令碼時，需要有工作目錄資料夾才能快取編譯結果、執行記錄，以及執行其他功能。 在 Azure Data Lake Tools for Visual Studio 中，工作目錄即是 U-SQL 專案的工作目錄。 它位於 `<U-SQL project root path>/bin/debug>` 底下。 每當觸發新執行時，工作目錄就會清空。

## <a name="local-runs-in-microsoft-visual-studio"></a>Microsoft Visual Studio 中的本機執行

Azure Data Lake Tools for Visual Studio 擁有內建的本機執行引擎。 工具會以本機計算帳戶的形式呈現引擎。 若要在本機執行 U-SQL 指令碼，請在指令碼的編輯器邊界下拉式功能表中選取 [Local-machine] 或 [Local-project] 帳戶。 然後，選取 [提交]。

![將 U-SQL 指令碼提交至本機帳戶](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-submit-script-to-local-account.png) 
 
## <a name="local-runs-with-a-local-machine-account"></a>使用 Local-machine 帳戶進行本機執行

**Local-machine** 帳戶是共用本機電腦帳戶，擁有一個作為本機存放區帳戶的本機資料根資料夾。 資料根資料夾的位置預設為 **C:\Users\<使用者名稱>\AppData\Local\USQLDataRoot**。 您也可以透過 [工具] > [Data Lake] > [選項和設定] 來設定此位置。

![設定本機資料的根資料夾](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-local-data-root.png)
  
本機執行須有 U-SQL 專案。 U-SQL 專案的工作目錄可以當做 U-SQL 本機執行工作目錄。 本機執行時產生的編譯結果、執行記錄和其他作業執行相關檔案會存放在工作目錄資料夾下。 每當您重新執行指令碼時，工作目錄中的所有檔案都會清空並重新產生。

## <a name="local-runs-with-a-local-project-account"></a>使用 Local-project 帳戶進行本機執行

**Local-project** 帳戶是依照專案區分的本機計算帳戶，每個專案都有各自獨立的本機資料根資料夾。 在 Visual Studio 的 [方案總管] 中開啟的每個作用中 U-SQL 專案，都有對應的 `(Local-project: <project name>)` 帳戶。 這些帳戶會在 Visual Studio 的 [伺服器總管] 和 U-SQL 指令碼編輯器邊界中列出。  

**Local-project** 帳戶提供了一個精簡又獨立的開發環境。 **Local-machine** 帳戶具有共用的本機資料根資料夾，可儲存中繼資料以及所有本機作業的輸入和輸出資料。 但是 **Local-project** 帳戶會在每次 U-SQL 指令碼執行時，於 U-SQL 專案的工作目錄底下建立暫存的本機資料根資料夾。 這個暫存的資料根資料夾會在重建或重新執行發生時清空。 

U-SQL 專案可透過專案參考和屬性來管理這個隔離的本機執行環境。 您可以在專案和所參考的資料庫環境中設定 U-SQL 指令碼的輸入資料來源。

### <a name="manage-the-input-data-source-for-a-local-project-account"></a>管理 Local-project 帳戶的輸入資料來源 

U-SQL 專案會建立本機資料根資料夾，並設定 **Local-project** 帳戶的資料。 每當重建和本機執行發生時，U-SQL 專案工作目錄下的暫存資料根資料夾都會清空及重新建立。 在本機作業執行前，U-SQL 專案設定的所有資料來源都會複製到這個暫存本機資料根資料夾。 

您可以設定資料來源的根資料夾。 以滑鼠右鍵按一下 [U-SQL 專案] > [屬性] > [測試資料來源]。 以 **Local-project** 帳戶執行 U-SQL 指令碼時，[測試資料來源] 資料夾內的所有檔案和子資料夾都會複製到暫存本機資料根資料夾。 子資料夾底下的檔案也包含在內。 本機作業執行後，您可以在專案工作目錄的暫存本機資料根資料夾下方找到輸出結果。 在重建及清理專案時，這個輸出內容會全部遭到刪除和清空。 

![設定專案的測試資料來源](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-project-test-data-source.png)

### <a name="manage-a-referenced-database-environment-for-a-local-project-account"></a>管理 **Local-project** 帳戶的參考資料庫環境 

如果 U-SQL 查詢使用的是 U-SQL 資料庫物件，或是以該物件進行查詢，您必須先在本機將資料庫環境準備妥當，才能在本機執行該 U-SQL 指令碼。 對於 **Local-project** 帳戶，您可以透過 U-SQL 專案參考來管理 U-SQL 資料庫相依性。 您可以將 U-SQL 資料庫專案參考新增到 U-SQL 專案。 在以 **Local-project** 帳戶執行 U-SQL 指令碼之前，所有參考資料庫都會部署到暫存本機資料根資料夾。 而在每次執行時，暫存資料根資料夾都會清空，成為全新的隔離環境。

請參閱此相關文章：
* 了解如何在 [U-SQL 資料庫專案](data-lake-analytics-data-lake-tools-develop-usql-database.md)中管理 U-SQL 資料庫定義和參考。

## <a name="the-difference-between-local-machine-and-local-project-accounts"></a>**Local-machine** 和 **Local-project** 帳戶之間的差異

**Local-machine** 帳戶會在使用者的本機電腦上模擬 Azure Data Lake Analytics 帳戶。 它的體驗與 Azure Data Lake Analytics 帳戶相同。 **Local-project** 帳戶則會提供方便使用者使用的本機開發環境。 這個環境可協助使用者先在本機部署資料庫參考和輸入資料再執行指令碼。 **Local-machine** 帳戶提供共用的永久環境，供使用者透過所有專案存取。 **Local-project** 帳戶提供每個專案各地獨立的開發環境。 每次執行時，系統都會重新整理此環境。 **Local-project** 帳戶能讓使用者迅速套用新變更，因而創造快速的開發體驗。

下表會顯示更多 **Local-machine** 和 **Local-project** 帳戶之間的差異：

|差異角度|Local-machine|Local-project|
|----------------|---------------|---------------|
|本機存取|所有專案均可存取。|只有對應的專案能存取這個帳戶。|
|本機資料根資料夾|永久本機資料夾。 透過 [工具] > [Data Lake] > [選項和設定] 設定。|每次本機執行時，會在 U-SQL 專案工作資料夾下建立暫存資料夾。 資料夾會在重建或重新執行發生時清空。|
|U-SQL 指令碼的輸入資料|永久本機資料根資料夾下方的相對路徑。|透過 [U-SQL 專案屬性] > [測試資料來源] 設定。 所有檔案和子資料夾都會在本機執行前複製到暫存資料根資料夾。|
|U-SQL 指令碼的輸出資料|永久本機資料根資料夾下方的相對路徑。|輸出到暫存資料根資料夾。 結果會在重建或重新執行發生時清空。|
|參考資料庫部署|以 **Local-machine** 帳戶執行時，不會自動部署參考資料庫。 提交到 Azure Data Lake Analytics 帳戶時也是如此。|參考資料庫會在本機執行前自動部署到 **Local-project** 帳戶。 所有資料庫環境都會在重建或重新執行發生時清空及重新部署。|

## <a name="a-local-run-with-the-u-sql-sdk"></a>使用 U-SQL SDK 進行本機執行

您可以在 Visual Studio 中本機執行 U-SQL 指令碼，也可以使用 Azure Data Lake U-SQL SDK，利用命令列和程式設計介面在本機執行 U-SQL 指令碼。 透過這些介面，您可以將 U-SQL 本機執行和測試自動化。

深入了解 [Azure Data Lake U-SQL SDK](data-lake-analytics-u-sql-sdk.md)。

## <a name="next-steps"></a>後續步驟

- [如何設定 Azure Data Lake Analytics 的 CI/CD 管線](data-lake-analytics-cicd-overview.md)。
- [如何測試 Azure Data Lake Analytics 程式碼](data-lake-analytics-cicd-test.md)。
