---
title: 在 Azure Stack 上使用 MySQL 資料庫做為 PaaS | Microsoft Docs
description: 了解如何在 Azure Stack 上部署「MySQL 資源提供者」並提供 MySQL 資料庫即服務。
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
ms.openlocfilehash: 24ba595413cde07c420a94de234d7926e0eb0e7f
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36311058"
---
# <a name="use-mysql-databases-on-microsoft-azure-stack"></a>在 Microsoft Azure Stack 上使用 MySQL 資料庫

您可以部署 MySQL 資源提供者 API，以使用部署在 Azure Stack 中的 MySQL 資料庫。 如需有關資源提供者 API 的詳細資訊，請參閱 [Windows Azure 套件 MySQL 資源提供者 REST API 參考](https://msdn.microsoft.com/library/dn528442.aspx) \(英文\)。

MySQL 資料庫是網站上常見的資料庫，且支援許多網站平台。 例如，您可以使用 Web Apps 平台即服務 (PaaS) 附加元件來建立 WordPress 網站。

在部署資源提供者之後，您可以：

* 使用 Azure Resource Manager 部署範本來建立 MySQL 伺服器和資料庫。
* 提供 MySQL 資料庫即服務。  

## <a name="mysql-resource-provider-adapter-architecture"></a>MySQL 資源提供者配接器架構

此資源提供者包含下列元件：

* **MySQL 資源提供者配接器虛擬機器 (VM)**：這是執行提供者服務的 Windows Server VM。
* **資源提供者**：這會處理要求並存取資料庫資源。
* **裝載 MySQL 伺服器的伺服器**：為稱為主控伺服器的資料庫提供容量。 您可以自行建立 MySQL 執行個體，或是提供對外部 MySQL 執行個體的存取權。 [Azure Stack 快速入門資源庫](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows)提供範例範本，可用來：

  * 為您建立 MySQL 伺服器。
  * 從 Azure Marketplace 下載和部署 MySQL 伺服器。

> [!NOTE]
> 建立安裝在 Azure Stack 整合式系統上的主控伺服器時，必須從租用戶訂用帳戶建立。 無法從預設提供者訂用帳戶建立這些伺服器。 必須從租用戶入口網站或從具有適當登入的 PowerShell 工作階段來建立這些伺服器。 所有主控伺服器都是可計費的 VM，而且必須具有授權。 服務管理員可以是租用戶訂用帳戶的擁有者。

### <a name="required-privileges"></a>必要的權限

系統帳戶必須具有下列權限：

* **資料庫：** 建立、卸除
* **登入：** 建立、設定、卸除、授與、撤銷  

## <a name="next-steps"></a>後續步驟

[部署 MySQL 資源提供者](azure-stack-mysql-resource-provider-deploy.md)
