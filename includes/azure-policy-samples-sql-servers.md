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
ms.openlocfilehash: 76e161be1adca4aa2ec7b682a13b0a42e4b79412
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47003598"
---
### <a name="sql-servers"></a>SQL Server

|  |  |
|---------|---------|
| [在沒有 Azure Active Directory 系統管理員時稽核](../articles/governance/policy/samples/audit-no-aad-admin.md) | 在 SQL Server 沒有指派的 Azure Active Directory 系統管理員時稽核。 |
| [稽核伺服器層級威脅偵測設定](../articles/governance/policy/samples/audit-sql-ser-threat-det-setting.md) | 稽核 SQL 資料庫安全性警示原則是否未設定成指定的狀態。 您需指定一個指出已啟用或停用威脅偵測的值。  |
| [稽核 SQL Server 稽核設定](../articles/governance/policy/samples/sql-server-audit.md) | 根據是否已啟用稽核設定，來稽核 SQL 伺服器。 |
| [稽核 SQL Server 層級稽核設定](../articles/governance/policy/samples/audit-sql-ser-leve-audit-setting.md) | 稽核 SQL Server 稽核設定是否與指定的設定不符。 您需指定一個指出應啟用或停用稽核設定的值。 |
| [要求 SQL Server 12.0 版](../articles/governance/policy/samples/req-sql-12.md) | 要求 SQL Server 使用 12.0 版。  |