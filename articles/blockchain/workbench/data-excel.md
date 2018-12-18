---
title: 在 Microsoft Excel 中使用 Azure Blockchain Workbench 資料
description: 了解如何在 Microsoft Excel 中載入與檢視 Azure Blockchain Workbench SQL DB 資料。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 3c257a47c796636d0b86aa6b246b17c0fd39bae3
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241231"
---
# <a name="view-azure-blockchain-workbench-data-with-microsoft-excel"></a>使用 Microsoft Excel 檢視 Azure Blockchain Workbench 資料

您可以使用 Microsoft Excel 檢視 Azure Blockchain Workbench SQL DB 中的資料。 本文提供執行以下動作所需的步驟：

* 從 Microsoft Excel 連線至 Blockchain Workbench 資料庫
* 查看 Blockchain Workbench 資料庫資料表與檢視
* 將 Blockchain Workbench 檢視資料載入 Excel

## <a name="connect-to-the-blockchain-workbench-database"></a>連線至 Blockchain Workbench 資料庫

若要連線至 Blockchain Workbench 資料庫：

1. 開啟 Microsoft Excel。
2. 在 [資料] 索引標籤上，選擇 [取得資料]。
3. 選取 [從 Azure]，然後選取 [從 Azure SQL 資料庫]。

   ![連線到 Azure SQL 資料庫](./media/data-excel/connect-sql-db.png)

4. 在 [SQL Server 資料庫] 對話方塊中：

    * 對於 [伺服器]，輸入 Blockchain Workbench 伺服器的名稱。
    * 對於 [資料庫 (選填)]，輸入資料庫名稱。

   ![提供資料庫伺服器與資料載](./media/data-excel/provide-server-db.png)

5. 在 [SQL Server 資料庫] 對話方塊導覽列中，選取 [資料庫]。 輸入 [使用者名稱] 與 [密碼]，然後選取 [連線]。

    > [!NOTE]
    > 如果您使用的是 Azure Blockchain Workbench 部署程序期間建立的認證，則 [使用者名稱] 為 `dbadmin`。 [密碼] 是您在部署 Blockchain Workbench 時建立的密碼。
    
   ![提供用於存取資料庫的認證](./media/data-excel/provide-credentials.png)

## <a name="look-at-database-tables-and-views"></a>查看資料庫資料表與檢視

[Excel 導覽器] 對話方塊會在您與資料庫連線後開啟。 您可以使用導覽器查看資料庫中的資料庫與檢視。 檢視是針對報告設計的，其名稱前會加上 **vw**。

   ![Excel 導覽器檢視預覽](./media/data-excel/excel-navigator.png)

## <a name="load-view-data-into-an-excel-workbook"></a>將檢視資料載入至 Excel 活頁簿

下一個範例顯示如何將資料從檢視載入至 Excel 活頁簿。

1. 在 [導覽器] 捲軸中，選取 [vwContractAction] 檢視。 [vwContractAction] 預覽會顯示與 Blockchain Workbench 資料庫中的合約相關的所有動作。
2. 選取 [載入] 以擷取檢視中的所有資料並放到 Excel 活頁簿中。

   ![從檢視載入的資料](./media/data-excel/view-data.png)

現在您已將資料載入，便可以使用 Excel 功能使用 Azure Blockchain Workbench 資料庫的中繼資料與交易資料建立您自己的報告。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的資料庫檢視](database-views.md)