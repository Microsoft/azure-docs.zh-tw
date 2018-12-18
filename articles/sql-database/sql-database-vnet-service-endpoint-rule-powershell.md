---
title: 使用 PowerShell 以建立虛擬網路服務端點和 Azure SQL 中的規則 | Microsoft Docs
description: 提供 PowerShell 指令碼，為 Azure SQL Database 和 SQL 資料倉儲建立和管理虛擬服務端點。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: PowerShell
ms.topic: conceptual
author: DhruvMsft
ms.author: dmalik
ms.reviewer: genemi, vanto
manager: craigg
ms.date: 06/14/2018
ms.openlocfilehash: 50e88dd11b8a883a4d2999ad2d0419cbf7176078
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161143"
---
# <a name="use-powershell-to-create-a-virtual-service-endpoint-and-rule-for-azure-sql-database-and-sql-data-warehouse"></a>使用 PowerShell 以建立虛擬服務端點和 Azure SQL Database 與 SQL 資料倉儲的規則

Azure [SQL Database](sql-database-technical-overview.md) 和 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)都支援虛擬服務端點。 

> [!NOTE]
> 本主題適用於 Azure SQL 伺服器，以及在 Azure SQL Server 上建立的 SQL Database 和 SQL 資料倉儲資料庫。 為了簡單起見，參考 SQL Database 和 SQL 資料倉儲時都會使用 SQL Database。

本文提供並說明可執行下列動作的 PowerShell 指令碼：

1. 在您的子網路上建立 Microsoft Azure「虛擬服務端點」。
2. 將端點加入 Azure SQL Database 伺服器的防火牆，以建立「虛擬網路規則」。

有關為何要建立規則的說明，請參閱：[Azure SQL Database 的虛擬服務端點][sql-db-vnet-service-endpoint-rule-overview-735r]。

> [!TIP]
> 如果您需要的是評估 SQL Database 的虛擬服務端點「類型名稱」或將它加入子網路，則可以直接跳到更為[直接的 PowerShell 指令碼](#a-verify-subnet-is-endpoint-ps-100)。

#### <a name="major-cmdlets"></a>主要 Cmdlet

本文重點在於名為 **New-AzureRmSqlServerVirtualNetworkRule** 的 Cmdlet。此 Cmdlet 會將子網路端點加入 Azure SQL Database 伺服器的存取控制清單 (ACL)，以藉此建立規則。

下列清單顯示在您準備呼叫 **New-AzureRmSqlServerVirtualNetworkRule** 之前，必須先執行的一系列其他「主要」 Cmdlet。 在本文中，這些呼叫發生於[指令碼 3「虛擬網路規則」](#a-script-30)中：

1. [New-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig)：建立子網路物件。

2. [New-AzureRmVirtualNetwork](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetwork)：建立您的虛擬網路，並指定它的子網路。

3. [Set-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/Set-AzureRmVirtualNetworkSubnetConfig)：為您的子網路指派虛擬服務端點。

4. [Set-AzureRmVirtualNetwork](https://docs.microsoft.com/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork)：保存對虛擬網路的更新。

5. [New-AzureRmSqlServerVirtualNetworkRule](https://docs.microsoft.com/powershell/module/azurerm.sql/new-azurermsqlservervirtualnetworkrule)：子網路成為端點之後，將子網路加入 Azure SQL Database 伺服器的 ACL 作為虛擬網路規則。
    - 從 Azure RM PowerShell 模組 5.1.1 版開始，提供參數 **-IgnoreMissingVnetServiceEndpoint**。

#### <a name="prerequisites-for-running-powershell"></a>執行 PowerShell 的必要條件

- 您已經可以登入 Azure，例如透過 [Azure 入口網站][http-azure-portal-link-ref-477t]。
- 您已經可以執行 PowerShell 指令碼。

> [!NOTE]
> 針對要新增至伺服器的 Vnet/子網路，請確認已開啟服務端點，否則建立 Vnet 防火牆規則將會失敗。

#### <a name="one-script-divided-into-four-chunks"></a>分成四個區塊的單一指令碼

示範的 PowerShell 指令碼會分割成一系列較小的指令碼。 分割指令碼有助於學習，並提供彈性。 指令碼需按照指定的順序執行。 如果您目前沒有時間執行指令碼，我們的實際測試輸出顯示在指令碼 4 的後面。






<a name="a-script-10" />

## <a name="script-1-variables"></a>指令碼 1：變數

第一個 PowerShell 指令碼會將值指派給變數。 後續的指令碼將會仰賴這些變數。

> [!IMPORTANT]
> 執行此指令碼之前，您可以依本身需求來編輯這些值。 比方說，如果您已經有一個資源群組，則可能會將資源群組名稱編輯為指派的值。
>
>  您的訂用帳戶名稱應編輯到指令碼中。

#### <a name="powershell-script-1-source-code"></a>PowerShell 指令碼 1 原始程式碼

```powershell
######### Script 1 ########################################
##   LOG into to your Azure account.                     ##
##   (Needed only one time per powershell.exe session.)  ##
###########################################################

$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]';
if ('yes' -eq $yesno) { Connect-AzureRmAccount; }

###########################################################
##  Assignments to variables used by the later scripts.  ##
###########################################################

# You can edit these values, if necessary.

$SubscriptionName = 'yourSubscriptionName';
Select-AzureRmSubscription -SubscriptionName $SubscriptionName;

$ResourceGroupName = 'RG-YourNameHere';
$Region            = 'westcentralus';

$VNetName            = 'myVNet';
$SubnetName          = 'mySubnet';
$VNetAddressPrefix   = '10.1.0.0/16';
$SubnetAddressPrefix = '10.1.1.0/24';
$VNetRuleName        = 'myFirstVNetRule-ForAcl';

$SqlDbServerName         = 'mysqldbserver-forvnet';
$SqlDbAdminLoginName     = 'ServerAdmin';
$SqlDbAdminLoginPassword = 'ChangeYourAdminPassword1';

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql';  # Official type name.

Write-Host 'Completed script 1, the "Variables".';
```





<a name="a-script-20" />

## <a name="script-2-prerequisites"></a>指令碼 2：必要條件

此指令碼會針對端點動作所在的下一個指令碼作準備。 此指令碼會為您建立以下列出的項目，但僅限於它們尚未存在時。 如果您確定這些項目已存在，則可跳過指令碼 2：

- Azure 資源群組
- Azure SQL Database 伺服器

#### <a name="powershell-script-2-source-code"></a>PowerShell 指令碼 2 原始程式碼

```powershell
######### Script 2 ########################################
##   Ensure your Resource Group already exists.          ##
###########################################################

Write-Host "Check whether your Resource Group already exists.";

$gottenResourceGroup = $null;

$gottenResourceGroup = Get-AzureRmResourceGroup `
  -Name        $ResourceGroupName `
  -ErrorAction SilentlyContinue;

if ($null -eq $gottenResourceGroup)
{
    Write-Host "Creating your missing Resource Group - $ResourceGroupName.";

    $gottenResourceGroup = New-AzureRmResourceGroup `
      -Name $ResourceGroupName `
      -Location $Region;

    $gottenResourceGroup;
}
else { Write-Host "Good, your Resource Group already exists - $ResourceGroupName."; }

$gottenResourceGroup = $null;

###########################################################
## Ensure your Azure SQL Database server already exists. ##
###########################################################

Write-Host "Check whether your Azure SQL Database server already exists.";

$sqlDbServer = $null;

$sqlDbServer = Get-AzureRmSqlServer `
  -ResourceGroupName $ResourceGroupName `
  -ServerName        $SqlDbServerName `
  -ErrorAction       SilentlyContinue;

if ($null -eq $sqlDbServer)
{
    Write-Host "Creating the missing Azure SQL Database server - $SqlDbServerName.";

    Write-Host "Gather the credentials necessary to next create an Azure SQL Database server.";

    $sqlAdministratorCredentials = New-Object `
      -TypeName     System.Management.Automation.PSCredential `
      -ArgumentList `
        $SqlDbAdminLoginName, `
        $(ConvertTo-SecureString `
            -String      $SqlDbAdminLoginPassword `
            -AsPlainText `
            -Force `
         );

    if ($null -eq $sqlAdministratorCredentials)
    {
        Write-Host "ERROR, unable to create SQL administrator credentials.  Now ending.";
        return;
    }

    Write-Host "Create your Azure SQL Database server.";

    $sqlDbServer = New-AzureRmSqlServer `
      -ResourceGroupName $ResourceGroupName `
      -ServerName        $SqlDbServerName `
      -Location          $Region `
      -SqlAdministratorCredentials $sqlAdministratorCredentials;

    $sqlDbServer;
}
else { Write-Host "Good, your Azure SQL Database server already exists - $SqlDbServerName."; }

$sqlAdministratorCredentials = $null;
$sqlDbServer                 = $null;

Write-Host 'Completed script 2, the "Prerequisites".';
```






<a name="a-script-30" />

## <a name="script-3-create-an-endpoint-and-a-rule"></a>指令碼 3：建立端點及規則

此指令碼會建立具有子網路的虛擬網路。 然後，指令碼會將 **Microsoft.Sql** 端點類型指派給您的子網路。 最後，指令碼會將您的子網路加入 SQL Database 伺服器的存取控制清單 (ACL)，以藉此建立規則。

#### <a name="powershell-script-3-source-code"></a>PowerShell 指令碼 3 原始程式碼

```powershell
######### Script 3 ########################################
##   Create your virtual network, and give it a subnet.  ##
###########################################################

Write-Host "Define a subnet '$SubnetName', to be given soon to a virtual network.";

$subnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

Write-Host "Create a virtual network '$VNetName'." `
  "  Give the subnet to the virtual network that we created.";

$vnet = New-AzureRmVirtualNetwork `
  -Name              $VNetName `
  -AddressPrefix     $VNetAddressPrefix `
  -Subnet            $subnet `
  -ResourceGroupName $ResourceGroupName `
  -Location          $Region;

###########################################################
##   Create a Virtual Service endpoint on the subnet.    ##
###########################################################

Write-Host "Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.";

$vnet = Set-AzureRmVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

Write-Host "Persist the updates made to the virtual network > subnet.";

$vnet = Set-AzureRmVirtualNetwork `
  -VirtualNetwork $vnet;

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

###########################################################
##   Add the Virtual Service endpoint Id as a rule,      ##
##   into SQL Database ACLs.                             ##
###########################################################

Write-Host "Get the subnet object.";

$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

$subnet = Get-AzureRmVirtualNetworkSubnetConfig `
  -Name           $SubnetName `
  -VirtualNetwork $vnet;

Write-Host "Add the subnet .Id as a rule, into the ACLs for your Azure SQL Database server.";

$vnetRuleObject1 = New-AzureRmSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnet.Id;

$vnetRuleObject1;

Write-Host "Verify that the rule is in the SQL DB ACL.";

$vnetRuleObject2 = Get-AzureRmSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName;

$vnetRuleObject2;

Write-Host 'Completed script 3, the "Virtual-Netowrk-Rule".';
```






<a name="a-script-40" />

## <a name="script-4-clean-up"></a>指令碼 4：清除

此最終指令碼會刪除先前指令碼所建立的示範用資源。 不過，指令碼在刪除下列項目之前會要求確認：

- Azure SQL Database 伺服器
- Azure 資源群組

您可以在指令碼 1 完成後的任何時間執行指令碼 4。

#### <a name="powershell-script-4-source-code"></a>PowerShell 指令碼 4 原始程式碼

```powershell
######### Script 4 ########################################
##   Clean-up phase A:  Unconditional deletes.           ##
##                                                       ##
##   1. The test rule is deleted from SQL DB ACL.        ##
##   2. The test endpoint is deleted from the subnet.    ##
##   3. The test virtual network is deleted.             ##
###########################################################

Write-Host "Delete the rule from the SQL DB ACL.";

Remove-AzureRmSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName `
  -ErrorAction            SilentlyContinue;

Write-Host "Delete the endpoint from the subnet.";

$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

Remove-AzureRmVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -VirtualNetwork $vnet;

Write-Host "Delete the virtual network (thus also deletes the subnet).";

Remove-AzureRmVirtualNetwork `
  -Name              $VNetName `
  -ResourceGroupName $ResourceGroupName `
  -ErrorAction       SilentlyContinue;

###########################################################
##   Clean-up phase B:  Conditional deletes.             ##
##                                                       ##
##   These might have already existed, so user might     ##
##   want to keep.                                       ##
##                                                       ##
##   1. Azure SQL Database server                        ##
##   2. Azure resource group                             ##
###########################################################

$yesno = Read-Host 'CAUTION !: Do you want to DELETE your Azure SQL Database server AND your Resource Group?  [yes/no]';
if ('yes' -eq $yesno)
{
    Write-Host "Remove the Azure SQL DB server.";
    
    Remove-AzureRmSqlServer `
      -ServerName        $SqlDbServerName `
      -ResourceGroupName $ResourceGroupName `
      -ErrorAction       SilentlyContinue;
    
    Write-Host "Remove the Azure Resource Group.";
    
    Remove-AzureRmResourceGroup `
      -Name        $ResourceGroupName `
      -ErrorAction SilentlyContinue;
}
else
{
    Write-Host "Skipped over the DELETE of SQL Database and resource group.";
}

Write-Host 'Completed script 4, the "Clean-Up".';
```






<a name="a-actual-output" />

## <a name="actual-output-from-scripts-1-through-4"></a>指令碼 1 到 4 的實際輸出

測試執行的輸出以簡短格式顯示如下。 如果您不想立即實際執行 PowerShell 指令碼，此輸出也許會有幫助。

```
[C:\WINDOWS\system32\]
0 >> C:\Demo\PowerShell\sql-database-vnet-service-endpoint-powershell-s1-variables.ps1
Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]: yes


Environment           : AzureCloud
Account               : xx@microsoft.com
TenantId              : 11111111-1111-1111-1111-111111111111
SubscriptionId        : 22222222-2222-2222-2222-222222222222
SubscriptionName      : MySubscriptionName
CurrentStorageAccount : 



[C:\WINDOWS\system32\]
0 >> C:\Demo\PowerShell\sql-database-vnet-service-endpoint-powershell-s2-prerequisites.ps1
Check whether your Resource Group already exists.
Creating your missing Resource Group - RG-YourNameHere.


ResourceGroupName : RG-YourNameHere
Location          : westcentralus
ProvisioningState : Succeeded
Tags              : 
ResourceId        : /subscriptions/22222222-2222-2222-2222-222222222222/resourceGroups/RG-YourNameHere

Check whether your Azure SQL Database server already exists.
Creating the missing Azure SQL Database server - mysqldbserver-forvnet.
Gather the credentials necessary to next create an Azure SQL Database server.
Create your Azure SQL Database server.

ResourceGroupName        : RG-YourNameHere
ServerName               : mysqldbserver-forvnet
Location                 : westcentralus
SqlAdministratorLogin    : ServerAdmin
SqlAdministratorPassword : 
ServerVersion            : 12.0
Tags                     : 
Identity                 : 

Completed script 2, the "Prerequisites".



[C:\WINDOWS\system32\]
0 >> C:\Demo\PowerShell\sql-database-vnet-service-endpoint-powershell-s3-vnet-rule.ps1
Define a subnet 'mySubnet', to be given soon to a virtual network.
Create a virtual network 'myVNet'.   Give the subnet to the virtual network that we created.
WARNING: The output object type of this cmdlet will be modified in a future release.
Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
Persist the updates made to the virtual network > subnet.

Get the subnet object.
Add the subnet .Id as a rule, into the ACLs for your Azure SQL Database server.
ProvisioningState Service       Locations      
----------------- -------       ---------      
Succeeded         Microsoft.Sql {westcentralus}
                                               
Verify that the rule is in the SQL DB ACL.
                                               
Completed script 3, the "Virtual-Network-Rule".



[C:\WINDOWS\system32\]
0 >> C:\Demo\PowerShell\sql-database-vnet-service-endpoint-powershell-s4-clean-up.ps1
Delete the rule from the SQL DB ACL.

Delete the endpoint from the subnet.


Delete the virtual network (thus also deletes the subnet).
CAUTION !: Do you want to DELETE your Azure SQL Database server AND your Resource Group?  [yes/no]: yes
Remove the Azure SQL DB server.

ResourceGroupName        : RG-YourNameHere
ServerName               : mysqldbserver-forvnet
Location                 : westcentralus
SqlAdministratorLogin    : ServerAdmin
SqlAdministratorPassword : 
ServerVersion            : 12.0
Tags                     : 
Identity                 : 

Remove the Azure Resource Group.
True
Completed script 4, the "Clean-Up".
```

我們的主要 PowerShell 指令碼到此結束。





<a name="a-verify-subnet-is-endpoint-ps-100" />

## <a name="verify-your-subnet-is-an-endpoint"></a>確認您的子網路是端點

子網路可能已被指派 **Microsoft.Sql** 類型名稱，這表示它已是虛擬服務端點。 您可以使用 [Azure 入口網站][http-azure-portal-link-ref-477t]從端點建立虛擬網路規則。

或者，您可能不確定您的子網路是否具有 **Microsoft.Sql** 類型名稱。 您可以執行下列 PowerShell 指令碼以進行下列動作：

1. 確定您的子網路是否具有 **Microsoft.Sql** 類型名稱。
2. 選擇性地指派類型名稱 (若沒有的話)。
    - 指令碼會先要求您「確認」，然後再套用缺少的類型名稱。

#### <a name="phases-of-the-script"></a>指令碼的階段

以下是 PowerShell 指令碼的各個階段：

1. 登入您的 Azure 帳戶，每個 PS 工作階段只需登入一次。  指派變數。
2. 搜尋您的虛擬網路，然後搜尋您的子網路。
3. 您的子網路是否已標記為 **Microsoft.Sql** 端點伺服器類型？
4. 在您的子網路上加入類型名稱是 **Microsoft.Sql** 的虛擬服務端點。

> [!IMPORTANT]
> 執行此指令碼之前，您必須先編輯指派給靠近指令碼最上方 $ 變數的值。

#### <a name="direct-powershell-source-code"></a>直接的 PowerShell 原始程式碼

除非要求確認時您回應「是」，否則此 PowerShell 指令碼不會更新任何項目。 此指令碼可以將類型名稱 **Microsoft.Sql** 加入您的子網路。 不過，只有當您的子網路缺少類型名稱時，此指令碼才會嘗試加入。

```powershell
### 1. LOG into to your Azure account, needed only once per PS session.  Assign variables.

$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]';
if ('yes' -eq $yesno) { Connect-AzureRmAccount; }

# Assignments to variables used by the later scripts.
# You can EDIT these values, if necessary.

$SubscriptionName  = 'yourSubscriptionName';
Select-AzureRmSubscription -SubscriptionName "$SubscriptionName";

$ResourceGroupName   = 'yourRGName';
$VNetName            = 'yourVNetName';
$SubnetName          = 'yourSubnetName';
$SubnetAddressPrefix = 'Obtain this value from the Azure portal.'; # Looks roughly like: '10.0.0.0/24'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql';  # Do NOT edit. Is official value.

### 2. Search for your virtual network, and then for your subnet.

# Search for the virtual network.
$vnet = $null;
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

if ($vnet -eq $null)
{
    Write-Host "Caution: No virtual network found by the name '$VNetName'.";
    Return;
}

$subnet = $null;
for ($nn=0; $nn -lt $vnet.Subnets.Count; $nn++)
{
    $subnet = $vnet.Subnets[$nn];
    if ($subnet.Name -eq $SubnetName)
    { break; }
    $subnet = $null;
}

if ($subnet -eq $null)
{
    Write-Host "Caution: No subnet found by the name '$SubnetName'";
    Return;
}

### 3. Is your subnet tagged as 'Microsoft.Sql' endpoint server type?

$endpointMsSql = $null;
for ($nn=0; $nn -lt $subnet.ServiceEndpoints.Count; $nn++)
{
    $endpointMsSql = $subnet.ServiceEndpoints[$nn];
    if ($endpointMsSql.Service -eq $ServiceEndpointTypeName_SqlDb)
    {
        $endpointMsSql;
        break;
    }
    $endpointMsSql = $null;
}

if ($endpointMsSql -ne $null)
{
    Write-Host "Good: Subnet found, and is already tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'.";
    Return;
}
else
{
    Write-Host "Caution: Subnet found, but not yet tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'.";

    # Ask the user for confirmation.
    $yesno = Read-Host 'Do you want the PS script to apply the endpoint type name to your subnet?  [yes/no]';
    if ('no' -eq $yesno) { Return; }
}

### 4. Add a Virtual Service endpoint of type name 'Microsoft.Sql', on your subnet.

$vnet = Set-AzureRmVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

# Persist the subnet update.
$vnet = Set-AzureRmVirtualNetwork `
  -VirtualNetwork $vnet;

for ($nn=0; $nn -lt $vnet.Subnets.Count; $nn++)
{ $vnet.Subnets[0].ServiceEndpoints; }  # Display.
```

#### <a name="actual-output"></a>實際輸出

下列區塊顯示實際的回饋 (已經過編輯以利閱讀)。

```powershell
<# Our output example (with cosmetic edits), when the subnet was already tagged:

Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]: no


Environment           : AzureCloud
Account               : xx@microsoft.com
TenantId              : 11111111-1111-1111-1111-111111111111
SubscriptionId        : 22222222-2222-2222-2222-222222222222
SubscriptionName      : MySubscriptionName
CurrentStorageAccount : 


ProvisioningState : Succeeded
Service           : Microsoft.Sql
Locations         : {westcentralus}

Good: Subnet found, and is already tagged as an endpoint of type 'Microsoft.Sql'.
#>
```




<!-- Link references: -->

[sql-db-vnet-service-endpoint-rule-overview-735r]: sql-database-vnet-service-endpoint-rule-overview.md

[http-azure-portal-link-ref-477t]: https://portal.azure.com/

