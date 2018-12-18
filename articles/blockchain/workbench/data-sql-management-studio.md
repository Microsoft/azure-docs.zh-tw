---
title: 將 Azure Blockchain Workbench 資料搭配 SQL Server Management Studio 使用
description: 了解如何從 SQL Server Management Studio 連線至 Azure Blockchain Workbench 的 SQL Database。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 405104554b4ee8e773b6d2e39966a262f98beb6d
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241225"
---
# <a name="using-azure-blockchain-workbench-data-with-sql-server-management-studio"></a>將 Azure Blockchain Workbench 資料搭配 SQL Server Management Studio 使用

Microsoft SQL Server Management Studio 可供快速撰寫查詢並對 Azure Blockhain Workbench 的 SQL DB 測試查詢。 本節提供如何從 SQL Server Management Studio 連線至 Azure Blockchain Workbench 的 SQL Database 的逐步解說。

## <a name="prerequisites"></a>必要條件

* 下載 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017)。

## <a name="connecting-sql-server-management-studio-to-data-in-azure-blockchain-workbench"></a>在 Azure Blockchain Workbench 中將 SQL Server Management Studio 連線至資料

1. 開啟 SQL Server Management Studio，然後選取 [連線]。
2. 選取 [資料庫引擎]。

    ![資料庫引擎](./media/data-sql-management-studio/database-engine.png)

3. 在 [連線到伺服器] 對話方塊中，輸入伺服器名稱與您的資料庫認證。

    如果您使用的是由 Azure Blockchain Workbench 部署程序建立的認證，則使用者名稱將為 **dbadmin**，且密碼將為您在部署期間提供的密碼。

    ![輸入 SQL 認證](./media/data-sql-management-studio/sql-creds.png)

 4. SQL Server Management Studio 會顯示 Azure Blockchain Workbench 資料庫中的資料庫、資料庫檢視與已儲存程序的清單。

    ![資料庫清單](./media/data-sql-management-studio/db-list.png)

5. 若要檢視與任何資料庫檢視相關聯的資料，您可以使用以下步驟自動產生 select 陳述式。
6. 以滑鼠右鍵按一下物件總管中的任何資料庫檢視。
7. 選取 [指令碼檢視為]。
8. 選擇 [SELECT 至]。
9. 選取 [新增查詢編輯器視窗]。
10. 您可以選取 [新增查詢] 建立新的查詢。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的資料庫檢視](database-views.md)