---
title: '透明資料加密 (的入口網站) 專用 SQL 集區 (先前為 SQL DW) '
description: 透明資料加密 (TDE) 的專用 SQL 集區 (先前的 SQL DW) Azure Synapse Analytics
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/30/2019
ms.author: jrasnick
ms.reviewer: rortloff
ms.custom: seo-lt-2019
ms.openlocfilehash: da4be6f4bc8335e0976a0a4a87c4d232b2a2285f
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98676301"
---
# <a name="get-started-with-transparent-data-encryption-tde-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>開始使用透明資料加密 (TDE) 的專用 SQL 集區 (先前的 SQL DW) Azure Synapse Analytics

> [!div class="op_single_selector"]
>
> * [安全性概觀](sql-data-warehouse-overview-manage-security.md)
> * [驗證](sql-data-warehouse-authentication.md)
> * [加密 (入口網站)](sql-data-warehouse-encryption-tde.md)
> * [加密 (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)

## <a name="required-permissions"></a>必要權限

您必須是系統管理員或 dbmanager 角色的成員，才能啟用透明資料加密 (TDE)。

## <a name="enabling-encryption"></a>啟用加密

若要啟用 TDE，請遵循下列步驟：

1. 在[Azure 入口網站](https://portal.azure.com)中開啟資料庫
2. 在資料庫刀鋒視窗中，按一下 [設定]  按鈕
3. 選取 **透明資料加密** 選項 ![ 入口網站設定](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png)
4. 選取 [ **開啟] 上** 的 ![ 入口網站設定](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-on.png)
5. 選取 [**儲存** 
    ![ 入口網站設定儲存]](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save.png)  

## <a name="disabling-encryption"></a>停用加密

若要停用 TDE，請遵循下列步驟：

1. 在[Azure 入口網站](https://portal.azure.com)中開啟資料庫
2. 在資料庫刀鋒視窗中，按一下 [設定]  按鈕
3. 選取 **透明資料加密** 選項 ![ 入口網站設定](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png)
4. 選取 **關閉** 設定 ![ 入口網站設定](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-off.png)
5. 選取 [**儲存** 
    ![ 入口網站] 設定 [儲存 2]](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save2.png)  

## <a name="encryption-dmvs"></a>加密 DMV

可以使用下列 DMW 來確認加密：

* [sys.databases](/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
* [sys.dm_pdw_nodes_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
