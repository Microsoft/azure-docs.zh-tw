---
title: SQL VM 上的自動化管理工作 (Resource Manager) | Microsoft 文件
description: 本文說明如何管理 SQL Server 代理程式擴充功能，此擴充功能可將特定 SQL Server 管理工作自動化。 其中包括自動備份、自動修補和 Azure 金鑰保存庫整合。
services: virtual-machines-windows
documentationcenter: ''
author: rothja
manager: craigg
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/12/2018
ms.author: jroth
ms.openlocfilehash: c663aec02d4d1808426a9f05a6674d5504563a63
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39012256"
---
# <a name="automate-management-tasks-on-azure-virtual-machines-with-the-sql-server-agent-extension-resource-manager"></a>使用 SQL Server 代理程式延伸模組 (Resource Manager) 自動化 Azure 虛擬機器上的管理工作
> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-sql-server-agent-extension.md)
> * [傳統](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md)

SQL Server IaaS 代理程式擴充功能 (SQLIaaSExtension) 會在 Azure 虛擬機器上執行，以將管理工作自動化。 本文概述擴充功能所支援的服務，以及與安裝、狀態及移除相關的指示。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

如需檢視這篇文章的精簡版本，請參閱 [SQL Server IaaS 代理程式擴充功能 (傳統)](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md)。

## <a name="supported-services"></a>支援的服務
SQL Server IaaS 代理程式擴充功能支援下列管理工作︰

| 系統管理功能 | 說明 |
| --- | --- |
| **SQL 自動備份** |針對 VM 中 SQL Server 的預設執行個體，將所有資料庫的備份排程自動化。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 自動備份 (Resource Manager)](virtual-machines-windows-sql-automated-backup.md)。 |
| **SQL 自動修補** |設定維護期間 (在此期間會進行 VM 的重要 Windows 更新)，以避免在工作負載尖峰時段進行更新。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 自動修補 (Resource Manager)](virtual-machines-windows-sql-automated-patching.md)。 |
| **Azure 金鑰保存庫整合** |讓您在 SQL Server VM 上自動安裝和設定 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [在 Azure VM (Resource Manager) 上設定 SQL Server 的 Azure 金鑰保存庫整合](virtual-machines-windows-ps-sql-keyvault.md)。 |

一旦安裝並執行 SQL Server IaaS 代理程式擴充功能之後，Azure 入口網站中虛擬機器的 SQL Server 面板即會提供這些管理功能，亦可透過 Azure PowerShell (若為 SQL Server Marketplace 映像) 以及 Azure PowerShell (若為手動安裝擴充功能) 使用這些功能。 

## <a name="prerequisites"></a>必要條件
在 VM 上使用 SQL Server IaaS 代理程式擴充功能的需求：

**作業系統**：

* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

**SQL Server 版本**：

* SQL Server 2012
* SQL Server 2014
* SQL Server 2016

**Azure PowerShell**：

* [下載及設定最新的 Azure PowerShell 命令](/powershell/azure/overview)

> [!IMPORTANT]
> 目前不支援將 [SQL Server IaaS 代理程式擴充功能](virtual-machines-windows-sql-server-agent-extension.md)用於 Azure 上的 SQL Server FCI。 建議您在參與 FCI 的 VM 上解除安裝此擴充功能。 在解除安裝代理程式後，此擴充功能所支援的功能即不適用於 SQL VM。

## <a name="installation"></a>安裝
當您佈建其中一個 SQL Server 虛擬機器資源庫映像時，系統會自動安裝 SQL Server IaaS 代理程式擴充功能。 如果您需要在其中一個 SQL Server 虛擬機器上手動重新安裝擴充功能，請使用下列 PowerShell 命令：

```powershell
Set-AzureRmVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SQLIaasExtension" -Version "1.2" -Location "East US 2"
```

> [!IMPORTANT]
> 如果擴充功能尚未安裝，安裝此擴充功能時 SQL Server 服務會重新啟動。

> [!NOTE]
> 只有 [SQL Server VM 資源庫映像](virtual-machines-windows-sql-server-iaas-overview.md#get-started-with-sql-vms) (隨用隨付或自備授權) 可支援 SQL Server IaaS 代理程式擴充功能。 如果您在 OS 專用的 Windows Server 虛擬機器上手動安裝 SQL Server，或是部署自訂的 SQL Server VM VHD，則不支援此擴充功能。 在這些情況下，可以使用 PowerShell 以手動方式安裝和管理擴充功能，但您無法在 Azure 入口網站中取得 SQL Server 組態設定。 不過，強烈建議您安裝 SQL Server VM 資源庫映像，然後加以自訂。

## <a name="status"></a>狀態
其中一項驗證已安裝擴充功能的方法，是在 Azure 入口網站中檢視代理程式狀態。 請選取虛擬機器視窗中的 [所有設定]，然後按一下 [擴充功能]。 您應該會看到列出 **SQLIaaSExtension** 擴充功能。

![Azure 入口網站中的 SQL Server IaaS 代理程式擴充功能](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-portal.png)

您也可以使用 **Get-AzureRmVMSqlServerExtension** Azure PowerShell Cmdlet。

    Get-AzureRmVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"

前一個命令會確認已安裝代理程式，並提供一般狀態資訊。 您也可以使用下列命令取得有關自動備份和修補的特定狀態資訊。

    $sqlext = Get-AzureRmVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings

## <a name="removal"></a>移除
在 Azure 入口網站中，您可以按一下虛擬機器屬性之 [擴充功能] 視窗上的省略符號，以將擴充功能解除安裝。 然後按一下 [刪除] 。

![將 Azure 入口網站中的 SQL Server IaaS 代理程式擴充功能解除安裝](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-uninstall.png)

您也可以使用 **Remove-AzureRmVMSqlServerExtension** PowerShell Cmdlet。

    Remove-AzureRmVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SQLIaasExtension"

## <a name="next-steps"></a>後續步驟
開始使用擴充功能所支援的其中一項服務。 如需詳細資訊，請參閱本文[支援的服務](#supported-services)一節中所參考的文章。

如需在 Azure 虛擬機器上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)。

