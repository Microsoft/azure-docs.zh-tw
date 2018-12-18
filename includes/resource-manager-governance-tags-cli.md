---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/20/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: b9484336add0719749e9f0af56bdd70fa3906ef5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38752654"
---
若要在資源群組中新增兩個標記，請使用 [az group update](/cli/azure/group#az_group_update) 命令：

```azurecli-interactive
az group update -n myResourceGroup --set tags.Environment=Test tags.Dept=IT
```

假設您想要新增第三個標記。 對新標記再次執行命令。 新標記會附加至現有標記。

```azurecli-interactive
az group update -n myResourceGroup --set tags.Project=Documentation
```

資源不會從資源群組繼承標記。 目前，您的資源群組具有三個標記，但是資源並沒有任何標記。 若要將所有標記從資源群組套用至其資源，並在資源上保留現有的標記，請使用下列指令碼：

```azurecli-interactive
# Get the tags for the resource group
jsontag=$(az group show -n myResourceGroup --query tags)

# Reformat from JSON to space-delimited and equals sign
t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')

# Get the resource IDs for all resources in the resource group
r=$(az resource list -g myResourceGroup --query [].id --output tsv)

# Loop through each resource ID
for resid in $r
do
  # Get the tags for this resource
  jsonrtag=$(az resource show --id $resid --query tags)
  
  # Reformat from JSON to space-delimited and equals sign
  rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
  
  # Reapply the updated tags to this resource
  az resource tag --tags $t$rt --id $resid
done
```

或者，您可以從資源群組將標記套用至資源，而不保留現有標記：

```azurecli-interactive
# Get the tags for the resource group
jsontag=$(az group show -n myResourceGroup --query tags)

# Reformat from JSON to space-delimited and equals sign
t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')

# Get the resource IDs for all resources in the resource group
r=$(az resource list -g myResourceGroup --query [].id --output tsv)

# Loop through each resource ID
for resid in $r
do
  # Apply tags from resource group to this resource
  az resource tag --tags $t --id $resid
done
```

若要在單一標記中合併數個值，請使用 JSON 字串。

```azurecli-interactive
az group update -n myResourceGroup --set tags.CostCenter='{"Dept":"IT","Environment":"Test"}'
```

若要移除資源群組上的所有標記，請使用：

```azurecli-interactive
az group update -n myResourceGroup --remove tags
```
