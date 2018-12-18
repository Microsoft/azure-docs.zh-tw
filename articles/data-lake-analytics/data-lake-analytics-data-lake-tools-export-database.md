---
title: 使用 Azure Data Lake Tools for Visual Studio 匯出 U-SQL 資料庫
description: 了解如何使用 Azure Data Lake Tools for Visual Studio 匯出 U-SQL 資料庫，並自動將它匯入至本機帳戶。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 11/27/2017
ms.openlocfilehash: e4eea3cb4b16460c7e17bb6575c4e6cf8dda5a0a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047249"
---
# <a name="export-a-u-sql-database"></a>匯出 U-SQL 資料庫

在本文中，了解如何使用 [Azure Data Lake Tools for Visual Studio](http://aka.ms/adltoolsvs)，以單一 U-SQL 指令碼與已下載資源的形式匯出 U SQL 資料庫。 您可以在相同的程序中，將匯出的資料庫匯入至本機帳戶。

客戶通常會維護多個開發、測試和實際執行環境。 這些環境同時裝載在本機帳戶上、開發人員的本機電腦上，以及 Azure 的 Azure Data Lake Analytics 帳戶中。 

當您在開發與測試環境中開發及微調 U-SQL 查詢時，開發人員通常需要在生產資料庫中重新建立其工作。 「資料庫匯出精靈」可協助加速此流程。 開發人員可使用該精靈，將現有的資料庫環境和範例資料複製到其他 Data Lake Analytics 帳戶。

## <a name="export-steps"></a>匯出步驟

### <a name="step-1-export-the-database-in-server-explorer"></a>步驟 1：在 [伺服器總管] 中匯出資料庫

您有權限的所有 Data Lake Analytics 帳戶會在 [伺服器總管] 中列出。 匯出資料庫：

1. 在 [伺服器總管] 中，展開包含您想要匯出之資料庫的帳戶。
2. 以滑鼠右鍵按一下資料庫，然後選取 [匯出]。 
   
    ![伺服器總管 - 匯出資料庫](./media/data-lake-analytics-data-lake-tools-export-database/export-database.png)

     如果 [匯出] 功能表選項無法使用，您需要[將工具更新至最新版本](http://aka.ms/adltoolsvs)。

### <a name="step-2-configure-the-objects-that-you-want-to-export"></a>步驟 2：設定想要匯出的物件

如果您只需要大型資料庫的一小部分，則可以在匯出精靈中設定您想要匯出的物件子集。 

匯出動作是透過執行 U-SQL 作業來完成。 因此，從 Azure 帳戶匯出會產生一些成本。

![資料庫匯出精靈 - 選取匯出物件](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard.png)

### <a name="step-3-check-the-objects-list-and-other-configurations"></a>步驟 3：檢查物件清單和其他設定

在此步驟中，您可以在 [匯出物件清單] 方塊中確認選取的物件。 如果有任何錯誤，請選取 [上一步] 返回，並正確地設定您想要匯出的物件。

您也可以設定匯出目標的其他設定。 設定描述於下表中列出：

|組態|說明|
|-------------|-----------|
|目的地名稱|此名稱指出您要儲存匯出之資料庫資源的位置。 範例包括組件、其他檔案和範例資料。 系統會在您本機資料根資料夾底下建立具有此名稱的資料夾。|
|專案目錄|此路徑定義您要儲存匯出之 U-SQL 指令碼的位置。 所有資料庫物件定義都會儲存在此位置。|
|僅限結構描述|如果您選取此選項，則只會匯出資料庫定義和資源 (例如組件和其他檔案)。|
|結構描述和資料|如果您選取此選項，則會匯出資料庫定義、資源和資料。 已匯出資料表最頂端的 N 列。|
|自動匯入本機資料庫|如果您選取此選項，匯出的資料庫會在匯出完成時，自動匯入至您的本機資料庫。|

![資料庫匯出精靈 - 匯出物件清單和其他設定](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-configuration.png)

### <a name="step-4-check-the-export-results"></a>步驟 4：檢查匯出結果

匯出完成時，您可以在精靈中的 [記錄] 視窗檢視匯出的結果。 下列範例顯示如何尋找匯出的 U-SQL 指令碼和資料庫資源，包括組件、其他檔案和範例資料：

![資料庫匯出精靈 - 匯出結果](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-completed.png)

## <a name="import-the-exported-database-to-a-local-account"></a>將匯出的資料庫匯入至本機帳戶

將匯出的資料庫匯入的最方便的方式，是在步驟 3 的匯出程序期間選取 [自動匯入本機資料庫] 核取方塊。 如果您沒有選取此方塊，首先，在匯出記錄中尋找匯出的 U-SQL 指令碼。 然後，在本機執行 U-SQL 指令碼，將資料庫匯入至您的本機帳戶。

## <a name="import-the-exported-database-to-a-data-lake-analytics-account"></a>將匯出的資料庫匯入至 Data Lake Analytics 帳戶

將資料庫匯入至不同的 Data Lake Analytics 帳戶：

1. 將匯出的資源 (包括組件、其他檔案和範例資料) 上傳到要匯入的目標 Data Lake Analytics 帳戶預設的 Azure Data Lake Store 帳戶。 您可以在本機資料根資料夾下尋找匯出的資源資料夾。 將整個資料夾上傳至預設 Data Lake Store 帳戶的根資料夾。
2. 上傳完成時，將匯出的 U-SQL 指令碼提交至您想要匯入資料庫的目標 Data Lake Analytics 帳戶。

## <a name="known-limitations"></a>已知限制

目前，如果您在步驟 3 中選取 [結構描述和資料] 選項，工具會執行 U-SQL 作業來匯出儲存在資料表中的資料。 因為這個緣故，資料匯出程序可能會變慢，而可能會對您產生費用。 

## <a name="next-steps"></a>後續步驟

* [了解 U-SQL 資料庫](https://msdn.microsoft.com/library/azure/mt621299.aspx) 
* [使用本機執行和 Azure Data Lake U-SQL SDK 對 U-SQL 作業進行測試和偵錯](data-lake-analytics-data-lake-tools-local-run.md)


