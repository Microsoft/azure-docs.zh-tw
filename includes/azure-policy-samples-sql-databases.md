---
title: 包含檔案
description: 包含檔案
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: include file
ms.openlocfilehash: 70128735aef64d273c63236a13b0ae28edb6077d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47003602"
---
### <a name="sql-databases"></a>SQL Database

|  |  |
|---------|---------|
| [允許的 SQL DB SKU](../articles/governance/policy/samples/allowed-sql-db-skus.md) | 要求 SQL 資料庫使用已核准的 SKU。 您需指定允許的 SKU 識別碼陣列或允許的 SKU 名稱陣列。 |
| [稽核 DB 層級威脅偵測設定](../articles/governance/policy/samples/audit-db-threat-det-setting.md) | 稽核 SQL 資料庫安全性警示原則是否未設定成指定的狀態。 您需指定一個指出已啟用或停用威脅偵測的值。  |
| [稽核 SQL Database 加密](../articles/governance/policy/samples/sql-database-encryption-audit.md) | 稽核 SQL 資料庫是否未啟用透明資料加密。 |
| [稽核 SQL DB 層級稽核設定](../articles/governance/policy/samples/audit-sql-db-audit-setting.md) | 稽核 SQL 資料庫稽核設定是否與指定的設定不符。 您需指定一個指出應啟用或停用稽核設定的值。  |
| [稽核透明資料加密狀態](../articles/governance/policy/samples/audit-trans-data-enc-status.md) | 稽核是否未啟用 SQL 資料庫透明資料加密。  |