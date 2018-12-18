---
title: 使用 PowerShell 管理 Azure Analysis Services | Microsoft Docs
description: 使用 PowerShell 的 Azure Analysis Services 管理。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: reference
ms.date: 06/25/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 6c648a9cb6b8d8dbfb60f1a5a6ebc386c57460b0
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42887241"
---
# <a name="manage-azure-analysis-services-with-powershell"></a>使用 PowerShell 管理 Azure Analysis Services

本文說明用來執行 Azure Analysis Services 伺服器和資料庫管理工作的 PowerShell Cmdlet。 

伺服器管理工作，例如建立或刪除伺服器、暫停或繼續伺服器作業，或是變更服務等級 (層級)，會使用 Azure Resource Manager (資源) Cmdlet 和 Analysis Services (伺服器) Cmdlet。 其他資料庫管理工作 (例如新增或移除角色成員、處理或資料分割) 會使用與 SQL Server Analysis Services 相同之 SqlServer 模組內含的 Cmdlet。

## <a name="permissions"></a>權限
大部分的 PowerShell 工作需要您在您管理的 Analysis Services 伺服器上具備系統管理員權限。 排定的 PowerShell 工作都是自動的作業。 執行排程器的帳戶和服務主體必須具有 Analysis Services 伺服器上的管理員權限。 

針對使用 AzureRm Cmdlet 的伺服器作業，您的帳戶或執行排程器的帳戶也必須屬於 [Azure 角色型存取控制 (RBAC)](../role-based-access-control/overview.md) 中之資源的 Owner 角色。 

## <a name="resource-management-operations"></a>資源管理作業 
模組 - [AzureRM.AnalysisServices](https://www.powershellgallery.com/packages/AzureRM.AnalysisServices)

|Cmdlet|說明| 
|------------|-----------------| 
|[Get-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/get-azurermanalysisservicesserver)|取得伺服器執行個體的詳細資料。|  
|[New-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/new-azurermanalysisservicesserver)|建立伺服器執行個體。|   
|[New-AzureRmAnalysisServicesFirewallConfig](/powershell/module/azurerm.analysisservices/new-azurermanalysisservicesfirewallconfig)|建立新的 Analysis Services 防火牆設定。|   
|[New-AzureRmAnalysisServicesFirewallRule](/powershell/module/azurerm.analysisservices/new-azurermanalysisservicesfirewallrule)|建立新的 Analysis Services 防火牆規則。|   
|[Remove-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/remove-azurermanalysisservicesserver)|移除伺服器執行個體。|  
|[Resume-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/resume-azurermanalysisservicesserver)|繼續伺服器執行個體。|  
|[Suspend-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/suspend-azurermanalysisservicesserver)|暫停伺服器執行個體。| 
|[Set-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/set-azurermanalysisservicesserver)|修改伺服器執行個體。|   
|[Test-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/test-azurermanalysisservicesserver)|測試伺服器執行個體的存在。| 

## <a name="server-management-operations"></a>伺服器管理作業

模組 - [Azure.AnalysisServices](https://www.powershellgallery.com/packages/Azure.AnalysisServices)

|Cmdlet|說明| 
|------------|-----------------| 
|[Add-AzureAnalysisServicesAccount](/powershell/module/azure.analysisservices/add-azureanalysisservicesaccount)|新增已驗證的帳戶以用於 Azure Analysis Services 伺服器 Cmdlet 要求。| 
|[Export-AzureAnalysisServicesInstance](/powershell/module/azurerm.analysisservices/export-azureanalysisservicesinstancelog)|從目前登入環境中之 Analysis Services 伺服器的執行個體匯出記錄，如 Add-AzureAnalysisServicesAccount 命令所指定|  
|[Restart-AzureAnalysisServicesInstance](/powershell/module/azurerm.analysisservices/restart-azureanalysisservicesinstance)|在目前登入的環境中重新啟動 Analysis Services 伺服器的執行個體；在 Add-AzureAnalysisServicesAccount 命令中指定。|  
|[Sync-AzureAnalysisServicesInstance](/powershell/module/azurerm.analysisservices/restart-azureanalysisservicesinstance)|將 Analysis Services 伺服器之指定執行個體上的指定資料庫，同步處理到目前登入環境中的所有查詢向外延展執行個體，如 Add-AzureAnalysisServicesAccount 命令所指定|  

## <a name="database-operations"></a>資料庫作業

Azure Analysis Services 資料庫作業會使用與 SQL Server Analysis Services 相同的 [SqlServer 模組](https://www.powershellgallery.com/packages/SqlServer)。 不過，Azure Analysis Services 並不支援所有的 Cmdlet。 若要深入了解，請參閱 [SQL Server PowerShell](https://docs.microsoft.com/sql/powershell/sql-server-powershell)。

SqlServer 模組提供特定工作的資料庫管理 Cmdlet，以及接受「表格式模型指令碼語言」(TMSL) 查詢或指令碼的一般用途 Invoke-ASCmd Cmdlet。 Azure Analysis Services 支援 SqlServer 模組中的下列 Cmdlet。

  
|Cmdlet|說明|
|------------|-----------------| 
|[Add-RoleMember](https://docs.microsoft.com/powershell/module/sqlserver/Add-RoleMember)|將成員新增到資料庫角色。| 
|[Backup-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/backup-asdatabase)|備份 Analysis Services 資料庫。|  
|[Remove-RoleMember](https://docs.microsoft.com/powershell/module/sqlserver/remove-rolemember)|從資料庫角色移除成員。|   
|[Invoke-ASCmd](https://docs.microsoft.com/powershell/module/sqlserver/invoke-ascmd)|執行 TMSL 指令碼。|
|[Invoke-ProcessASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processasdatabase)|處理資料庫。|  
|[Invoke-ProcessPartition](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processpartition)|處理資料分割。| 
|[Invoke-ProcessTable](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processtable)|處理資料表。|  
|[Merge-Partition](https://docs.microsoft.com/powershell/module/sqlserver/merge-partition)|合併資料分割。|  
|[Restore-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/restore-asdatabase)|還原 Analysis Services 資料庫。| 
  

## <a name="related-information"></a>相關資訊

* [下載 SQL Server PowerShell 模組](https://docs.microsoft.com/sql/ssms/download-sql-server-ps-module)   
* [下載 SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)   
* [PowerShell 資源庫中的 SqlServer 模組](https://www.powershellgallery.com/packages/SqlServer)    
* [相容性層級 1200 和更高層級的表格式模型程式設計](https://msdn.microsoft.com/library/mt712541.aspx)
