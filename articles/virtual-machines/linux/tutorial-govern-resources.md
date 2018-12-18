---
title: 教學課程 - 使用 Azure CLI 控管 Azure 虛擬機器 | Microsoft Docs
description: 在此教學課程中，您將了解如何使用 Azure CLI，透過套用 RBAC、原則、鎖定與標籤來管理 Azure 虛擬機器
services: virtual-machines-linux
documentationcenter: virtual-machines
author: tfitzmac
manager: jeconnoc
editor: tysonn
ms.service: virtual-machines-linux
ms.workload: infrastructure
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: tutorial
ms.date: 10/12/2018
ms.author: tomfitz
ms.custom: mvc
ms.openlocfilehash: 715a8e5bab9e5d16b8c0e54298101df856d51a9a
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49309854"
---
# <a name="tutorial-learn-about-linux-virtual-machine-governance-with-azure-cli"></a>教學課程：了解如何使用 Azure CLI 控管 Linux 虛擬機器

[!INCLUDE [Resource Manager governance introduction](../../../includes/resource-manager-governance-intro.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 Azure CLI，則在本教學課程中，您必須執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="understand-scope"></a>了解範圍

[!INCLUDE [Resource Manager governance scope](../../../includes/resource-manager-governance-scope.md)]

在本教學課程中，您會將所有的管理設定套用到資源群，讓您可以在完成後輕易地移除這些設定。

先來建立該資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location "East US"
```

目前資源群組是空的。

## <a name="role-based-access-control"></a>角色型存取控制

您想要確定組織中的使用者具有這些資源的正確存取權等級。 您不想要授與不受限制的存取權給使用者，但是您又必須確定他們可以執行其工作。 [角色型存取控制](../../role-based-access-control/overview.md)可讓您管理哪些使用者具有權限可以在一個範圍內完成特定動作。

若要建立和移除角色指派，使用者必須具有 `Microsoft.Authorization/roleAssignments/*` 存取權。 此存取權是透過擁有者或使用者存取系統管理員角色來授與。

為了管理虛擬機器解決方案，有三個資源專屬角色可提供您經常需要的存取權：

* [虛擬機器參與者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [網路參與者](../../role-based-access-control/built-in-roles.md#network-contributor)
* [儲存體帳戶參與者](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

相較於將角色指派給個別使用者，使用 Azure Active Directory 群組來含括需要執行類似動作的使用者，通常會較為容易。 然後，將該群組指派給適當的角色。 在本文中，請使用現有的群組來管理虛擬機器，或使用入口網站來[建立 Azure Active Directory 群組](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。

建立新群組或找到現有群組後，請使用 [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) 命令，將新的 Azure Active Directory 群組指派給資源群組的虛擬機器參與者角色。

```azurecli-interactive
adgroupId=$(az ad group show --group <your-group-name> --query objectId --output tsv)

az role assignment create --assignee-object-id $adgroupId --role "Virtual Machine Contributor" --resource-group myResourceGroup
```

如果出現錯誤，指出**原則 <guid> 不存在於目錄中**，表示新群組未傳播至整個 Azure Active Directory。 請嘗試再次執行命令。

通常您需要針對網路參與者和儲存體帳戶參與者重複進行此程序，以確保已指派使用者來管理已部署的資源。 在本文中，您可以略過這些步驟。

## <a name="azure-policy"></a>Azure 原則

[Azure 原則](../../azure-policy/azure-policy-introduction.md)可協助您確認訂用帳戶中的所有資源均符合公司標準。 您的訂用帳戶已經有數個原則定義。 若要查看可用的原則定義，請使用 [az policy definition list](/cli/azure/policy/definition#az_policy_definition_list) 命令：

```azurecli-interactive
az policy definition list --query "[].[displayName, policyType, name]" --output table
```

您會看到現有的原則定義。 原則類型不是**內建**就是**自訂**。 查看定義，尋找符合您要指派之條件的定義。 在本文中，您指派的原則要：

* 限制所有資源的位置。
* 限制虛擬機器的 SKU。
* 稽核未使用受控磁碟的虛擬機器。

在下列範例中，您會根據顯示名稱擷取三個原則定義。 您使用 [az policy assignment create](/cli/azure/policy/assignment#az_policy_assignment_create) 命令將這些定義指派給資源群組。 針對某些原則，您要提供參數值來指定允許的值。

```azurecli-interactive
# Get policy definitions for allowed locations, allowed SKUs, and auditing VMs that don't use managed disks
locationDefinition=$(az policy definition list --query "[?displayName=='Allowed locations'].name | [0]" --output tsv)
skuDefinition=$(az policy definition list --query "[?displayName=='Allowed virtual machine SKUs'].name | [0]" --output tsv)
auditDefinition=$(az policy definition list --query "[?displayName=='Audit VMs that do not use managed disks'].name | [0]" --output tsv)

# Assign policy for allowed locations
az policy assignment create --name "Set permitted locations" \
  --resource-group myResourceGroup \
  --policy $locationDefinition \
  --params '{ 
      "listOfAllowedLocations": {
        "value": [
          "eastus", 
          "eastus2"
        ]
      }
    }'

# Assign policy for allowed SKUs
az policy assignment create --name "Set permitted VM SKUs" \
  --resource-group myResourceGroup \
  --policy $skuDefinition \
  --params '{ 
      "listOfAllowedSKUs": {
        "value": [
          "Standard_DS1_v2", 
          "Standard_E2s_v2"
        ]
      }
    }'

# Assign policy for auditing unmanaged disks
az policy assignment create --name "Audit unmanaged disks" \
  --resource-group myResourceGroup \
  --policy $auditDefinition
```

上述範例假設您已知道原則的參數。 如果您需要檢視參數，請使用：

```azurecli-interactive
az policy definition show --name $locationDefinition --query parameters
```

## <a name="deploy-the-virtual-machine"></a>部署虛擬機器

您已指派角色和原則，現在可以開始部署您的解決方案。 預設大小是 Standard_DS1_v2，也就是所允許之 SKU 的其中一個。 如果預設位置中沒有 SSH 金鑰，命令會加以建立。

```azurecli-interactive
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --generate-ssh-keys
```

部署完成之後，您可以套用更多的管理設定至解決方案。

## <a name="lock-resources"></a>鎖定資源

[資源鎖定](../../azure-resource-manager/resource-group-lock-resources.md)可避免組織中的使用者不小心刪除或修改重要資源。 不同於角色型存取控制，資源鎖定會對所有使用者和角色套用限制。 您可以將鎖定層級設定為 *CanNotDelete* 或 *ReadOnly*。

若要建立或刪除管理鎖定，您必須擁有 `Microsoft.Authorization/locks/*` 動作的存取權。 在內建角色中，只有 **擁有者** 和 **使用者存取管理員** 被授與這些動作的存取權。

若要鎖定虛擬機器和網路安全性群組，請使用 [az lock create](/cli/azure/lock#az_lock_create) 命令：

```azurecli-interactive
# Add CanNotDelete lock to the VM
az lock create --name LockVM \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVM \
  --resource-type Microsoft.Compute/virtualMachines

# Add CanNotDelete lock to the network security group
az lock create --name LockNSG \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVMNSG \
  --resource-type Microsoft.Network/networkSecurityGroups
```

若要測試鎖定，請嘗試執行下列命令：

```azurecli-interactive 
az group delete --name myResourceGroup
```

您會看到一則錯誤，指出刪除作業因鎖定而無法完成。 只有當您明確移除鎖定後，才能刪除資源群組。 [清除資源](#clean-up-resources)中會說明該步驟。

## <a name="tag-resources"></a>標記資源

您可將[標籤](../../azure-resource-manager/resource-group-using-tags.md)套用至 Azure 資源，以便以邏輯方式依照類別組織這些資源。 每個標記都是由一個名稱和一個值所組成。 例如，您可以將「環境」名稱和「生產」值套用至生產環境中的所有資源。

[!INCLUDE [Resource Manager governance tags CLI](../../../includes/resource-manager-governance-tags-cli.md)]

若要將標籤套用至虛擬機器，請使用 [az resource tag](/cli/azure/resource#az_resource_tag) 命令。 資源上的任何現有標記都不會保留。

```azurecli-interactive
az resource tag -n myVM \
  -g myResourceGroup \
  --tags Dept=IT Environment=Test Project=Documentation \
  --resource-type "Microsoft.Compute/virtualMachines"
```

### <a name="find-resources-by-tag"></a>依標籤尋找資源

若要尋找具有某標籤名稱和值的資源，請使用 [az resource list](/cli/azure/resource#az_resource_list) 命令：

```azurecli-interactive
az resource list --tag Environment=Test --query [].name
```

傳回的值可用於管理工作，例如停止具有某標籤值的所有虛擬機器。

```azurecli-interactive
az vm stop --ids $(az resource list --tag Environment=Test --query "[?type=='Microsoft.Compute/virtualMachines'].id" --output tsv)
```

### <a name="view-costs-by-tag-values"></a>依標籤值檢視成本

[!INCLUDE [Resource Manager governance tags billing](../../../includes/resource-manager-governance-tags-billing.md)]

## <a name="clean-up-resources"></a>清除資源

您無法刪除已鎖定的網路安全性群組，除非移除鎖定。 若要移除鎖定，請擷取鎖定的識別碼，並將這些識別碼提供給 [az lock delete](/cli/azure/lock#az_lock_delete) 命令：

```azurecli-interactive
vmlock=$(az lock show --name LockVM \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Compute/virtualMachines \
  --resource-name myVM --output tsv --query id)
nsglock=$(az lock show --name LockNSG \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Network/networkSecurityGroups \
  --resource-name myVMNSG --output tsv --query id)
az lock delete --ids $vmlock $nsglock
```

若不再需要，您可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令來移除資源群組、VM 和所有相關資源。 結束 SSH 工作階段並返回您的 VM，然後將資源刪除，如下所示：

```azurecli-interactive 
az group delete --name myResourceGroup
```


## <a name="next-steps"></a>後續步驟

您在本教學課程中建立了自訂 VM 映像。 您已了解如何︰

> [!div class="checklist"]
> * 將使用者指派給角色
> * 套用會強制執行標準的原則
> * 使用鎖定保護重要資源
> * 標記資源以進行計費和管理

請前進到下一個教學課程，以了解如何使虛擬機器具備高可用性。

> [!div class="nextstepaction"]
> [監視虛擬機器](tutorial-monitoring.md)

