---
title: 在 Azure Stack 上使用 SQL 資料庫 | Microsoft Docs
description: 了解如何在 Azure Stack 上部署 SQL 資料庫即服務，並了解部署 SQL Server 資源提供者配接器的快速步驟。
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/21/2018
ms.author: jeffgilb
ms.reviewer: jeffgo
ms.openlocfilehash: 55d0e51606e8768a01c0b5a7766dbafe24d97a0d
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36307820"
---
# <a name="use-sql-databases-on-microsoft-azure-stack"></a>在 Microsoft Azure Stack 上使用 SQL 資料庫

使用 SQL Server 資源提供者配接器 API，將 SQL 資料庫公開為 [Azure Stack](azure-stack-poc.md) 的服務。 安裝資源提供者並將它連接至一或多個 SQL Server 執行個體之後，您和您的使用者可以建立：

- 適用於雲端原生應用程式的資料庫。
- 使用 SQL 的網站。
- 使用 SQL 的工作負載。

資源提供者未提供 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 的所有資料庫管理功能。 例如，不支援可自動配置資源的彈性集區。 但是，資源提供者在 SQL Server 資料庫上支援類似的建立、讀取、更新及刪除 (CRUD) 作業。 如需資源提供者 API 的詳細資訊，請參閱 [Windows Azure 套件 SQL Server 資源提供者 REST API 參考](https://msdn.microsoft.com/library/dn528529.aspx)。

>[!NOTE]
SQL Server 資源提供者 API 與 Azure SQL Database 不相容。

## <a name="sql-resource-provider-adapter-architecture"></a>SQL 資源提供者配接器架構

資源提供者是由下列元件所組成：

- **SQL 資源提供者配接器虛擬機器 (VM)**，這是可執行提供者服務的 Windows Server VM。
- **資源提供者**，其可處理要求並存取資料庫資源。
- **裝載 SQL Server 的伺服器**：為稱為主控伺服器的資料庫提供容量。

您必須建立至少一個 SQL Server 執行處理，或提供外部 SQL Server 執行個體的存取權。

> [!NOTE]
> 建立安裝在 Azure Stack 整合式系統上的主控伺服器時，必須從租用戶訂用帳戶建立。 無法從預設提供者訂用帳戶建立這些伺服器。 必須從租用戶入口網站或使用 PowerShell 搭配適當登入來建立這些伺服器。 所有主控伺服器都是可計費的虛擬機器，而且必須具有授權。 服務管理員可以是租用戶訂用帳戶的擁有者。

## <a name="next-steps"></a>後續步驟

[部署 SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)
