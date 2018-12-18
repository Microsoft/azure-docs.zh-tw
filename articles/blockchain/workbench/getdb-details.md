---
title: 取得 Azure Blockchain Workbench 資料庫詳細資料
description: 了解如何取得 Azure Blockchain Workbench 資料庫與資料庫伺服器資訊。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 4020500e20973e8818320424833815c1b5224f6c
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241189"
---
# <a name="get-information-about-your-azure-blockchain-workbench-database"></a>取得 Azure Blockchain Workbench 資料庫的相關資訊

本文示範如何取得 Azure Blockchain Workbench 資料庫的詳細資料。

## <a name="overview"></a>概觀

使用 Blockchain Workbench SQL DB 中的資料庫檢視可提供應用程式、工作流程與智慧合約執行的相關資訊。 開發人員可以在使用如 Microsoft Excel、PowerBI、Visual Studio 及 SQL Server Management Studio 等工具時運用此資訊。

開發人員在連線到資料庫之前，必須具備：

* 資料庫防火牆中允許的外部用戶端存取權。 本文與設定資料庫防火牆有關，說明如何允許存取權。
* 資料庫伺服器名稱與資料庫名稱。

## <a name="connect-to-the-blockchain-workbench-database"></a>連線至 Blockchain Workbench 資料庫

若要連線至資料庫：

1. 使用具有 Azure Blockchain Workbench 資源**擁有者**權限的帳戶登入 Azure 入口網站。
2. 在左側導覽窗格中，選取 [資源群組]。
3. 為 Blockchain Workbench 部署選擇資源群組的名稱。
4. 選取 [類型] 排序資源清單，然後選擇您的 **SQL 伺服器**。 下一個螢幕擷取畫面中的排序清單顯示兩個 SQL 資料庫，一個是 "master" 資料庫，另一個是使用 "lhgn" 作為**資源前置詞**的資料庫。

   ![排序後的 Blockchain Workbench 資源清單](./media/getdb-details/sorted-workbench-resource-list.png)

5. 若要檢視 Blockchain Workbench 資料庫的相關詳細資訊，請選取具有您部署 Blockchain Workbench 時所提供**資源前置詞**的資料庫連結。

   ![資料庫詳細資料](./media/getdb-details/workbench-db-details.png)

資料庫伺服器名稱與資料庫名稱可讓您使用開發或報告工具連線至 Blockchain Workbench 資料庫。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的資料庫檢視](database-views.md)