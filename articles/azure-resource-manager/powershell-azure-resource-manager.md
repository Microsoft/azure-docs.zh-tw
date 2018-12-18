---
title: 使用 PowerShell 管理 Azure 解決方案 | Microsoft Docs
description: 使用 Azure PowerShell 和 Resource Manager 管理資源。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: b33b7303-3330-4af8-8329-c80ac7e9bc7f
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: powershell
ms.devlang: na
ms.topic: conceptual
ms.date: 07/20/2018
ms.author: tomfitz
ms.openlocfilehash: 7cda2a406c6c49e9252bfd5840e8f943e5b7043f
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39205794"
---
# <a name="manage-resources-with-azure-powershell"></a>使用 Azure PowerShell 管理資源

[!INCLUDE [Resource Manager governance introduction](../../includes/resource-manager-governance-intro.md)]

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="understand-scope"></a>了解範圍

[!INCLUDE [Resource Manager governance scope](../../includes/resource-manager-governance-scope.md)]

在本文中，您會將所有的管理設定套用到資源群，讓您可以在完成後輕易地移除這些設定。

先來建立資源群組。

```azurepowershell-interactive
Set-AzureRmContext -Subscription <subscription-name>
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

目前資源群組是空的。

## <a name="role-based-access-control"></a>角色型存取控制

[!INCLUDE [Resource Manager governance policy](../../includes/resource-manager-governance-rbac.md)]

### <a name="assign-a-role"></a>指派角色

在本文中，您會部署虛擬機器以及和其相關的虛擬網路。 為了管理虛擬機器解決方案，有三個資源專屬角色可提供您經常需要的存取權：

* [虛擬機器參與者](../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [網路參與者](../role-based-access-control/built-in-roles.md#network-contributor)
* [儲存體帳戶參與者](../role-based-access-control/built-in-roles.md#storage-account-contributor)

相較於將角色指派給個別使用者，為需要執行類似動作的使用者[建立 Azure Active Directory 群組](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)通常會更加容易。 然後，將該群組指派給適當的角色。 一言以蔽之，您要建立沒有成員的 Azure Active Directory 群組。 您仍然可以指派此群組給範圍內的角色。 

下列範例會建立群組，並將其指派給資源群組中的「虛擬機器參與者」角色。 若要執行 `New-AzureAdGroup` 命令，您必須使用 [Azure Cloud Shell](/azure/cloud-shell/overview) 或[下載 Azure AD PowerShell 模組](https://www.powershellgallery.com/packages/AzureAD/)。

```azurepowershell-interactive
$adgroup = New-AzureADGroup -DisplayName VMDemoContributors `
  -MailNickName vmDemoGroup `
  -MailEnabled $false `
  -SecurityEnabled $true
New-AzureRmRoleAssignment -ObjectId $adgroup.ObjectId `
  -ResourceGroupName myResourceGroup `
  -RoleDefinitionName "Virtual Machine Contributor"
```

通常您需要針對**網路參與者**和**儲存體帳戶參與者**重複程序，以確保已指派使用者來管理已部署的資源。 在本文中，您可以略過這些步驟。

## <a name="azure-policy"></a>Azure 原則

[Azure 原則](../azure-policy/azure-policy-introduction.md)可協助您確認訂用帳戶中的所有資源均符合公司標準。 您的訂用帳戶已經有數個原則定義。 若要查看可使用的原則定義，請使用：

```azurepowershell-interactive
(Get-AzureRmPolicyDefinition).Properties | Format-Table displayName, policyType
```

您會看到現有的原則定義。 原則類型不是**內建**就是**自訂**。 查看定義，尋找符合您要指派之條件的定義。 在本文中，您指派的原則要：

* 限制所有資源的位置
* 限制虛擬機器的 SKU
* 稽核未使用受控磁碟的虛擬機器

```azurepowershell-interactive
$locations ="eastus", "eastus2"
$skus = "Standard_DS1_v2", "Standard_E2s_v2"

$rg = Get-AzureRmResourceGroup -Name myResourceGroup

$locationDefinition = Get-AzureRmPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed locations"}
$skuDefinition = Get-AzureRmPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed virtual machine SKUs"}
$auditDefinition = Get-AzureRmPolicyDefinition | where-object {$_.properties.displayname -eq "Audit VMs that do not use managed disks"}

New-AzureRMPolicyAssignment -Name "Set permitted locations" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $locationDefinition `
  -listOfAllowedLocations $locations
New-AzureRMPolicyAssignment -Name "Set permitted VM SKUs" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $skuDefinition `
  -listOfAllowedSKUs $skus
New-AzureRMPolicyAssignment -Name "Audit unmanaged disks" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $auditDefinition
```

## <a name="deploy-the-virtual-machine"></a>部署虛擬機器

您已指派角色和原則，現在可以開始部署您的解決方案。 預設大小是 Standard_DS1_v2，也就是所允許之 SKU 的其中一個。 執行此步驟時，系統會提示您輸入認證。 您輸入的值會設定為虛擬機器的使用者名稱和密碼。

```azurepowershell-interactive
New-AzureRmVm -ResourceGroupName "myResourceGroup" `
     -Name "myVM" `
     -Location "East US" `
     -VirtualNetworkName "myVnet" `
     -SubnetName "mySubnet" `
     -SecurityGroupName "myNetworkSecurityGroup" `
     -PublicIpAddressName "myPublicIpAddress" `
     -OpenPorts 80,3389
```

部署完成之後，您可以套用更多的管理設定至解決方案。

## <a name="lock-resources"></a>鎖定資源

[!INCLUDE [Resource Manager governance locks](../../includes/resource-manager-governance-locks.md)]

### <a name="lock-a-resource"></a>鎖定資源

若要鎖定虛擬機器和網路安全性群組，請使用：

```azurepowershell-interactive
New-AzureRmResourceLock -LockLevel CanNotDelete `
  -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup
New-AzureRmResourceLock -LockLevel CanNotDelete `
  -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

只有當您明確移除鎖定後，才能刪除虛擬機器。 [清除資源](#clean-up-resources)中會說明該步驟。

## <a name="tag-resources"></a>標記資源

[!INCLUDE [Resource Manager governance tags](../../includes/resource-manager-governance-tags.md)]

### <a name="tag-resources"></a>標記資源

[!INCLUDE [Resource Manager governance tags Powershell](../../includes/resource-manager-governance-tags-powershell.md)]

若要將標記套用至虛擬機器，請使用：

```azurepowershell-interactive
$r = Get-AzureRmResource -ResourceName myVM `
  -ResourceGroupName myResourceGroup `
  -ResourceType Microsoft.Compute/virtualMachines
Set-AzureRmResource -Tag @{ Dept="IT"; Environment="Test"; Project="Documentation" } -ResourceId $r.ResourceId -Force
```

### <a name="find-resources-by-tag"></a>依標記尋找資源

若要以標記名稱和值尋找資源，請使用：

```azurepowershell-interactive
(Find-AzureRmResource -TagName Environment -TagValue Test).Name
```

傳回的值可用於管理工作，例如停止具有特定標記值的所有虛擬機器。

```azurepowershell-interactive
Find-AzureRmResource -TagName Environment -TagValue Test | Where-Object {$_.ResourceType -eq "Microsoft.Compute/virtualMachines"} | Stop-AzureRmVM
```

### <a name="view-costs-by-tag-values"></a>依標記值檢視成本

將標記套用至資源之後，您可以檢視具有這些標記之資源的成本。 成本分析顯示最新的使用方式需要一些時間，因此您可能還看不到成本。 當有成本可顯示時，您可以檢視訂用帳戶中所有資源群組的資源成本。 使用者必須擁有[訂用帳戶層級的計費資訊存取權](../billing/billing-manage-access.md)才能查看成本。

若要在入口網站中依標記檢視成本，請選取您的訂用帳戶，然後選取 [成本分析]。

![成本分析](./media/powershell-azure-resource-manager/select-cost-analysis.png)

接著，以標記值篩選，選取 [套用]。

![依標記檢視成本](./media/powershell-azure-resource-manager/view-costs-by-tag.png)

您也可以使用 [Azure 計費 API](../billing/billing-usage-rate-card-overview.md) 透過程式設計的方式檢視成本。

## <a name="clean-up-resources"></a>清除資源

您無法刪除已鎖定的網路安全性群組，除非移除鎖定。 若要移除鎖定，請使用︰

```azurepowershell-interactive
Remove-AzureRmResourceLock -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup
Remove-AzureRmResourceLock -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟
* 若要深入了解如何監視您的虛擬機器，請參閱[使用 Azure PowerShell 監視和更新 Windows 虛擬機器](../virtual-machines/windows/tutorial-monitoring.md)。
* 若要了解如何使用 Azure 資訊安全中心來實作建議的安全性作法，請參閱[使用 Azure 資訊安全中心監視虛擬機器安全性](../virtual-machines/windows/tutorial-azure-security.md)。
* 您可以將現有的資源移動到新的資源群組。 如需範例，請參閱 [將資源移至新的資源群組或訂用帳戶](resource-group-move-resources.md)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。
