---
title: 適用於 DNS 別名 Azure SQL 的 PowerShell | Microsoft Docs
description: PowerShell Cmdlet (例如 New-AzureRMSqlServerDNSAlias) 可讓您將新的用戶端連線重新導向至不同的 Azure SQL Database 伺服器，而不需修改任何用戶端組態。
keywords: dns sql database
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.devlang: PowerShell
ms.topic: conceptual
ms.date: 02/05/2018
ms.reviewer: genemi;amagarwa;maboja
ms.author: dmalik
ms.openlocfilehash: 0353f503ea099b1355b6879efe0748a377115474
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34644309"
---
# <a name="powershell-for-dns-alias-to-azure-sql-database"></a>適用於 Azure SQL Database 之 DNS 別名的 PowerShell

本文提供的 PowerShell 指令碼會示範如何管理 Azure SQL Database 的 DNS 別名。 此指令碼會執行可採取下列動作的下列 Cmdlet：


程式碼範例中使用的 Cmdlet 如下所示：
- [New-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/New-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：在 Azure SQL Database 服務系統中建立新的 DNS 別名。 此別名會參照 Azure SQL Database 伺服器 1。
- [Get-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Get-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：取得並列出所有指派給 SQL DB 伺服器 1 的 DNS 別名。
- [Set-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Set-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：修改別名已設定要參照的伺服器名稱 (從伺服器 1 改為 SQL DB 伺服器 2)。
- [Remove-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Remove-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：使用別名的名稱，移除 SQL DB 伺服器 2 中的 DNS 別名。


上述 PowerShell Cmdlet 已新增至 **AzureRm.Sql** 模組 (從 5.1.1 版起)。




#### <a name="dns-alias-in-connection-string"></a>連接字串中的 DNS 別名

若要連線至特定 Azure SQL Database 伺服器，SQL Server Management Studio (SSMS) 等用戶端可以提供 DNS 別名名稱，而不必提供真實的伺服器名稱。 在下列伺服器字串範例中，any-unique-alias-name 別名會取代四個節點伺服器字串中第一個以點分隔的節點：
- 伺服器字串範例：`any-unique-alias-name.database.windows.net`。



## <a name="prerequisites"></a>先決條件

如果您想要執行本文中提供的 PowerShell 示範指令碼，則適用下列必要條件：

- Azure 訂用帳戶和帳戶。 若要取得免費試用，請按一下 [https://azure.microsoft.com/free/][https://azure.microsoft.com/free/]。

- Azure PowerShell 模組，搭配 **New-AzureRMSqlServerDNSAlias** Cmdlet。
    - 若要安裝或升級，請參閱[安裝 Azure PowerShell 模組][install-azurerm-ps-84p]。
    - 在 powershell\_ise.exe 中執行 `Get-Module -ListAvailable AzureRM;`，以尋找版本。

- 兩部 Azure SQL Database 伺服器。

## <a name="code-example"></a>程式碼範例

下列 PowerShell 程式碼範例一開始會將常值指派給數個變數。 若要執行程式碼，您必須先編輯所有預留位置值，使其符合您系統中的實際值。 或者，您也可以只是研究程式碼。 系統也會提供程式碼的主控台輸出。


```powershell
################################################################
###    Assign prerequisites.                                 ###
################################################################
cls;

$SubscriptionName             = '<EDIT-your-subscription-name>';
[string]$SubscriptionGuid_Get = '?'; # The script assigns this value, not you.

$SqlServerDnsAliasName = '<EDIT-any-unique-alias-name>';

$1ResourceGroupName = '<EDIT-rg-1>';  # Can be same or different than $2ResourceGroupName.
$1SqlServerName     = '<EDIT-sql-1>'; # Must differ from $2SqlServerName.

$2ResourceGroupName = '<EDIT-rg-2>';
$2SqlServerName     = '<EDIT-sql-2>';

# Login to your Azure subscription, first time per session.
Write-Host "You must log into Azure once per powershell_ise.exe session,";
Write-Host "  thus type 'yes' only the first time.";
Write-Host " ";
$yesno = Read-Host '[yes/no]  Do you need to log into Azure now?';
if ('yes' -eq $yesno)
{
    Connect-AzureRmAccount -SubscriptionName $SubscriptionName;
}

$SubscriptionGuid_Get = Get-AzureRmSubscription `
    -SubscriptionName $SubscriptionName;

################################################################
###    Working with DNS aliasing for Azure SQL DB server.    ###
################################################################

Write-Host '[1] Assign a DNS alias to SQL DB server 1.';
New-AzureRMSqlServerDNSAlias `
    –ResourceGroupName  $1ResourceGroupName `
    -ServerName         $1SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName;

Write-Host '[2] Get and list all the DNS aliases that are assigned to SQL DB server 1.';
Get-AzureRMSqlServerDNSAlias `
    –ResourceGroupName $1ResourceGroupName `
    -ServerName        $1SqlServerName;

Write-Host '[3] Move the DNS alias from 1 to SQL DB server 2.';
Set-AzureRMSqlServerDNSAlias `
    –ResourceGroupName  $2ResourceGroupName `
    -NewServerName      $2SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName `
    -OldServerResourceGroup  $1ResourceGroupName `
    -OldServerName           $1SqlServerName `
    -OldServerSubscriptionId $SubscriptionGuid_Get;

# Here your client, such as SSMS, can connect to your "$2SqlServerName"
# by using "$SqlServerDnsAliasName" in the server name.
# For example, server:  "any-unique-alias-name.database.windows.net".

# Remove-AzureRMSqlServerDNSAlias  - would fail here for SQL DB server 1.

Write-Host '[4] Remove the DNS alias from SQL DB server 2.';
Remove-AzureRMSqlServerDNSAlias `
    –ResourceGroupName  $2ResourceGroupName `
    -ServerName         $2SqlServerName `
    -ServerDNSAliasName $SqlServerDnsAliasName;
```


#### <a name="actual-console-output-from-the-powershell-example"></a>PowerShell 範例的實際主控台輸出

下列主控台輸出是從實際執行複製並貼上而來。


```
You must log into Azure once per powershell_ise.exe session,
  thus type 'yes' only the first time.
 
[yes/no]  Do you need to log into Azure now?: yes


Environment           : AzureCloud
Account               : gm@acorporation.com
TenantId              : 72f988bf-1111-1111-1111-111111111111
SubscriptionId        : 45651c69-2222-2222-2222-222222222222
SubscriptionName      : mysubscriptionname
CurrentStorageAccount : 

 
[1] Assign a DNS alias to SQL DB server 1.
[2] Get the DNS alias that is assigned to SQL DB server 1.
[3] Move the DNS alias from 1 to SQL DB server 2.
[4] Remove the DNS alias from SQL DB server 2.
ResourceGroupName ServerName         ServerDNSAliasName    
----------------- ----------         ------------------    
gm-rg-dns-1       gm-sqldb-dns-1     unique-alias-name-food
gm-rg-dns-1       gm-sqldb-dns-1     unique-alias-name-food
gm-rg-dns-2       gm-sqldb-dns-2     unique-alias-name-food


[C:\windows\system32\]
>> 
```

## <a name="next-steps"></a>後續步驟

如需 SQL Database 的 DNS 別名功能完整說明，請參閱 [Azure SQL Database 的 DNS 別名][dns-alias-overview-37v]。



<!-- Article links. -->

[https://azure.microsoft.com/free/]: https://azure.microsoft.com/free/

[install-azurerm-ps-84p]: https://docs.microsoft.com/powershell/azure/install-azurerm-ps

[dns-alias-overview-37v]: dns-alias-overview.md

