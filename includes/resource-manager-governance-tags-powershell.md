---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 05/21/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 5dc4ce00685c74b2974cf1bfb5e8606eb3063e8d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38740528"
---
若要將兩個標記新增至資源群組，請使用 [Set-AzureRmResourceGroup](/powershell/module/azurerm.resources/set-azurermresourcegroup) 命令：

```azurepowershell-interactive
Set-AzureRmResourceGroup -Name myResourceGroup -Tag @{ Dept="IT"; Environment="Test" }
```

假設您想要新增第三個標記。 每當您將標記套用到資源或資源群組時，即會覆寫該資源或資源群組上現有的標記。 若要新增新的標記又不會遺失現有標記，您必須擷取現有標記、新增標記，然後重新套用標記的集合：

```azurepowershell-interactive
# Get existing tags and add a new tag
$tags = (Get-AzureRmResourceGroup -Name myResourceGroup).Tags
$tags.Add("Project", "Documentation")

# Reapply the updated set of tags 
Set-AzureRmResourceGroup -Tag $tags -Name myResourceGroup
```

資源不會從資源群組繼承標記。 目前，您的資源群組具有三個標記，但是資源並沒有任何標記。 若要將所有標記從資源群組套用至其資源，並在資源上保留所有非重複的現有標記，請使用下列指令碼︰

```azurepowershell-interactive
# Get the resource group
$group = Get-AzureRmResourceGroup myResourceGroup

if ($group.Tags -ne $null) {
    # Get the resources in the resource group
    $resources = Get-AzureRmResource -ResourceGroupName $group.ResourceGroupName

    # Loop through each resource
    foreach ($r in $resources)
    {
        # Get the tags for this resource
        $resourcetags = (Get-AzureRmResource -ResourceId $r.ResourceId).Tags
        
        # If the resource has existing tags, add new ones
        if ($resourcetags)
        {
            foreach ($key in $group.Tags.Keys)
            {
                if (-not($resourcetags.ContainsKey($key)))
                {
                    $resourcetags.Add($key, $group.Tags[$key])
                }
            }

            # Reapply the updated tags to the resource 
            Set-AzureRmResource -Tag $resourcetags -ResourceId $r.ResourceId -Force
        }
        else
        {
            Set-AzureRmResource -Tag $group.Tags -ResourceId $r.ResourceId -Force
        }
    }
}
```

或者，您可以從資源群組將標記套用至資源，而不保留現有標記：

```azurepowershell-interactive
# Get the resource group
$g = Get-AzureRmResourceGroup -Name myResourceGroup

# Find all the resources in the resource group, and for each resource apply the tags from the resource group
Get-AzureRmResource -ResourceGroupName $g.ResourceGroupName | ForEach-Object {Set-AzureRmResource -ResourceId $_.ResourceId -Tag $g.Tags -Force }
```

若要在單一標記中合併數個值，請使用 JSON 字串。

```azurepowershell-interactive
Set-AzureRmResourceGroup -Name myResourceGroup -Tag @{ CostCenter="{`"Dept`":`"IT`",`"Environment`":`"Test`"}" }
```

若要移除所有標記，您要傳遞空的雜湊表。

```azurepowershell-interactive
Set-AzureRmResourceGroup -Name myResourceGroup -Tag @{ }
```
