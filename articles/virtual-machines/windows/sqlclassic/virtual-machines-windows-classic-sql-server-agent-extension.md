---
title: SQL VM 上的自動化管理工作 (傳統) | Microsoft 文件
description: 本主題說明如何管理 SQL Server 代理程式擴充功能，此擴充功能可將特定 SQL Server 管理工作自動化。 其中包括自動備份、自動修補和 Azure 金鑰保存庫整合。 本主題使用傳統部署模式。
services: virtual-machines-windows
documentationcenter: ''
author: rothja
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: a9bda2e7-cdba-427c-bc30-77cde4376f3a
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/12/2018
ms.author: jroth
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 55a31d7a0ab603dd7fe7de514d11d003e044240a
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008326"
---
# <a name="automate-management-tasks-on-azure-virtual-machines-with-the-sql-server-agent-extension-classic"></a>使用 SQL Server 代理程式延伸模組 (傳統) 自動化 Azure 虛擬機器上的管理工作
> [!div class="op_single_selector"]
> * [Resource Manager](../sql/virtual-machines-windows-sql-server-agent-extension.md)
> * [傳統](../classic/sql-server-agent-extension.md)
> 
>
 
Azure 虛擬機器會執行 SQL Server IaaS Agent 擴充功能 (SQLIaaSAgent) 以自動化系統管理工作。 本主題概述擴充功能所支援的服務，以及與安裝、狀態及移除相關的指示。

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 若要檢視這篇文章的 Resource Manager 版本，請參閱 [適用於 SQL Server VM Resource Manager 的 SQL Server Agent 擴充功能](../sql/virtual-machines-windows-sql-server-agent-extension.md)。

## <a name="supported-services"></a>支援的服務
SQL Server IaaS 代理程式擴充功能支援下列管理工作︰

| 系統管理功能 | 說明 |
| --- | --- |
| **SQL 自動備份** |針對 VM 中 SQL Server 的預設執行個體，將所有資料庫的備份排程自動化。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 自動備份 (傳統)](../classic/sql-automated-backup.md)。 |
| **SQL 自動修補** |設定維護期間 (在此期間會進行 VM 的重要 Windows 更新)，以避免在工作負載尖峰時段進行更新。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 自動修補 (傳統)](../classic/sql-automated-patching.md)。 |
| **Azure 金鑰保存庫整合** |讓您在 SQL Server VM 上自動安裝和設定 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [在 Azure VM 上設定 SQL Server 的 Azure 金鑰保存庫整合 (傳統)](../classic/ps-sql-keyvault.md)。 |

## <a name="prerequisites"></a>必要條件
在 VM 上使用 SQL Server IaaS 代理程式擴充功能的需求：

### <a name="operating-system"></a>作業系統：
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

### <a name="sql-server-versions"></a>SQL Server 版本：
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016

### <a name="azure-powershell"></a>Azure PowerShell：
[下載及設定最新的 Azure PowerShell 命令](/powershell/azure/overview)。

啟動 Windows PowerShell，然後使用 **Add-AzureAccount** 命令將它與您的 Azure 訂用帳戶連線。

    Add-AzureAccount

如果您有多個訂用帳戶，請使用 **Select-AzureSubscription** 以選取包含您目標傳統 VM 的訂用帳戶。

    Select-AzureSubscription -SubscriptionName <subscriptionname>

此時，您可以使用 **Get-AzureVM** 命令來取得傳統虛擬機器及其相關服務名稱的清單。

    Get-AzureVM

## <a name="installation"></a>安裝
針對傳統 VM，您必須使用 PowerShell 來安裝「SQL Server IaaS 代理程式擴充功能」並設定其相關服務。 請使用 **Set-AzureVMSqlServerExtension** PowerShell Cmdlet 來安裝擴充功能。 例如，下列命令會在 Windows Server VM (傳統) 上安裝擴充功能，並將它命名為 "SQLIaaSExtension"。

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -ReferenceName "SQLIaasExtension" -Version "1.2" | Update-AzureVM

如果更新到最新版的 SQL IaaS 代理程式擴充，您必須在更新擴充之後重新啟動虛擬機器。

> [!NOTE]
> 傳統虛擬機器沒有可透過入口網站安裝及設定「SQL IaaS 代理程式擴充功能」的選項。

> [!NOTE]
> 只有 [SQL Server VM 資源庫映像](../sql/virtual-machines-windows-sql-server-iaas-overview.md#get-started-with-sql-vms) (隨用隨付或自備授權) 可支援 SQL Server IaaS 代理程式擴充功能。 如果您在 OS 專用的 Windows Server 虛擬機器上手動安裝 SQL Server，或是部署自訂的 SQL Server VM VHD，則不支援此擴充功能。 在這些情況下，可以使用 PowerShell 以手動方式安裝和管理擴充功能，但強烈建議您改為安裝 SQL Server 資源庫映像，然後加以自訂。

## <a name="status"></a>狀態
驗證已安裝擴充功能的其中一項方法，是在 Azure 入口網站中檢視代理程式狀態。 請選取虛擬機器刀鋒視窗中所列的一部虛擬機器，然後按一下 [擴充功能]。 您應該會看到其中列出 **SQLIaaSAgent** 擴充功能。

![Azure 入口網站中的 SQL Server IaaS 代理程式擴充功能](./media/virtual-machines-windows-classic-sql-server-agent-extension/azure-sql-server-iaas-agent-portal.png)

您也可以使用 **Get-AzureVMSqlServerExtension** Azure Powershell Cmdlet。

    Get-AzureVM –ServiceName "service" –Name "vmname" | Get-AzureVMSqlServerExtension

## <a name="removal"></a>移除
在「Azure 入口網站」中，您可以按一下虛擬機器屬性 [擴充功能]  刀鋒視窗上的省略符號，來將擴充功能解除安裝。 然後按一下 [解除安裝]。

![將 Azure 入口網站中的 SQL Server IaaS 代理程式擴充功能解除安裝](./media/virtual-machines-windows-classic-sql-server-agent-extension/azure-sql-server-iaas-agent-uninstall.png)

您也可以使用 **Remove-AzureVMSqlServerExtension** Powershell Cmdlet。

    Get-AzureVM –ServiceName "service" –Name "vmname" | Remove-AzureVMSqlServerExtension | Update-AzureVM

## <a name="next-steps"></a>後續步驟
開始使用擴充功能所支援的其中一項服務。 如需詳細資訊，請參閱本文 [支援的服務](#supported-services) 一節中參考的主題。

如需在 Azure 虛擬機器上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](../sql/virtual-machines-windows-sql-server-iaas-overview.md)。

