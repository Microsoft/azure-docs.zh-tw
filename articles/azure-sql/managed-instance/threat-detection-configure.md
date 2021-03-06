---
title: 設定進階威脅防護
titleSuffix: Azure SQL Managed Instance
description: Advanced 威脅防護會偵測異常資料庫活動，指出 Azure SQL 受控執行個體中資料庫的潛在安全性威脅。
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 12/01/2020
ms.openlocfilehash: 69bebcf872f55055117acf5cef410d1f89eafe34
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446915"
---
# <a name="configure-advanced-threat-protection-in-azure-sql-managed-instance"></a>在 Azure SQL 受控執行個體中設定 Advanced 威脅防護
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

適用于[AZURE SQL 受控執行個體](sql-managed-instance-paas-overview.md)的[Advanced 威脅防護](../database/threat-detection-overview.md)會偵測異常活動，指出存取或惡意探索資料庫的不尋常且可能有害的嘗試。 Advanced 威脅防護可以識別 **潛在的 SQL 插入**、 **從不尋常的位置或資料中心進行** 存取、 **從不熟悉的主體或可能有害的應用程式存取**，以及 **暴力密碼破解 SQL 認證** -請參閱 [Advanced 威脅防護警示](../database/threat-detection-overview.md#alerts)中的詳細資料。

您可以透過[電子郵件通知](../database/threat-detection-overview.md#explore-detection-of-a-suspicious-event)或[Azure 入口網站](../database/threat-detection-overview.md#explore-alerts-in-the-azure-portal)，收到有關偵測到之威脅的通知

[Advanced 威脅防護](../database/threat-detection-overview.md) 是 [AZURE Defender for SQL](../database/azure-defender-for-sql.md)  供應專案的一部分，這是適用于 Advanced sql 安全性功能的整合套件。 您可以透過中央 Azure Defender for SQL 入口網站來存取和管理 Advanced 威脅防護。

##  <a name="azure-portal"></a>Azure 入口網站

1. 登入  [Azure 入口網站](https://portal.azure.com)。 
2. 流覽至您要保護之 SQL 受控執行個體實例的 [設定] 頁面。 在 [ **安全性**] 底下，選取 [ **安全中心**]。
3. 在 Azure Defender for SQL 設定頁面中
   - 開啟 **適用于 SQL 的 Azure Defender** 。
   - 設定在偵測到異常資料庫活動時，將 **警示傳送至** 電子郵件地址以接收安全性警示。
   - 選取儲存異常威脅稽核記錄的 [Azure 儲存體帳戶]。
   - 選取您要設定的 **Advanced 威脅防護類型** 。 深入瞭解 [Advanced 威脅防護警示](../database/threat-detection-overview.md)。
4. 按一下 [ **儲存** ]，以儲存新的或更新的 Azure DEFENDER for SQL 原則。

   :::image type="content" source="../database/media/azure-defender-for-sql/set-up-advanced-threat-protection-mi.png" alt-text="設定 advanced 威脅防護":::

## <a name="next-steps"></a>後續步驟

- 深入瞭解 [先進的威脅防護](../database/threat-detection-overview.md)。
- 若要瞭解受控實例，請參閱 [什麼是 AZURE SQL 受控執行個體](sql-managed-instance-paas-overview.md)。
- 深入瞭解 [Azure SQL Database 的 Advanced 威脅防護](../database/threat-detection-configure.md)。
- 深入瞭解 [SQL 受控執行個體的審核](./auditing-configure.md)。
- 深入瞭解 [Azure 資訊安全中心](../../security-center/security-center-introduction.md)。