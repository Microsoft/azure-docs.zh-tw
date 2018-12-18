---
title: 如何使用 Azure CLI 在 Azure VMSS 上設定系統和使用者指派的受控識別
description: 使用 Azure CLI 在 Azure VMSS 上設定系統和使用者指派受控識別的逐步指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/15/2018
ms.author: daveba
ms.openlocfilehash: eb8ec68bc7e19af77e94bdf38f8e2bc3322d7fc6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993505"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-virtual-machine-scale-set-using-azure-cli"></a>使用 Azure CLI 在虛擬機器擴展集上設定 Azure 資源受控識別

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Azure 資源受控識別會在 Azure Active Directory 中為 Azure 服務提供自動受控識別。 您可以使用此身分識別來向任何支援 Azure AD 驗證的服務進行驗證，不需要任何您程式碼中的認證。 

在本文中，您將了解如何使用 Azure CLI，在 Azure 虛擬機器擴展集 (VMSS) 上執行下列 Azure 資源受控識別作業：
- 在 Azure VMSS 上啟用和停用系統指派受控識別
- 在 Azure VMSS 上新增和移除使用者指派受控識別


## <a name="prerequisites"></a>必要條件

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 若要執行本文中的管理作業，您的帳戶需要下列 Azure 角色型存取控制指派：

    > [!NOTE]
    > 不需要其他 Azure AD 目錄角色指派。

    - [虛擬機器參與者](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor)，可建立虛擬機器擴展集，並從虛擬機器擴展集啟用和移除系統和/或使用者指派的受控識別。
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可為虛擬機器擴展集指派和移除使用者指派的受控識別。
- 若要執行 CLI 指令碼範例，您有三個選項：
    - 從 Azure 入口網站使用 [Azure Cloud Shell](../../cloud-shell/overview.md) (請參閱下一節)。
    - 請透過每個程式碼區塊右上角的 [立即試用] 按鈕，使用內嵌的 Azure Cloud Shell。
    - 如果您偏好使用本機 CLI 主控台，請[安裝最新版的 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0.13 或更新版本)。 
      
      > [!NOTE]
      > 命令已更新，以反映最新版的 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="system-assigned-managed-identity"></a>系統指派的受控識別

在本節中，您將了解如何使用 Azure CLI 啟用和停用 Azure VMSS 的系統指派受控識別。

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-virtual-machine-scale-set"></a>在 Azure 虛擬機器擴展集建立期間啟用系統指派的受控識別

若要建立已啟用系統指派受控識別的虛擬機器擴展集：

1. 如果您要在本機主控台中使用 Azure CLI，請先使用 [az login](/cli/azure/reference-index#az-login) 登入 Azure。 使用您想部署虛擬機器擴展集且已與 Azure 訂用帳戶相關聯的帳戶：

   ```azurecli-interactive
   az login
   ```

2. 使用 [az group create](/cli/azure/group/#az-group-create)，為您的虛擬機器擴展集和其相關資源建立[資源群組](../../azure-resource-manager/resource-group-overview.md#terminology)。 如果您已經有想要使用的資源群組，您可以略過此步驟：

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

3. 使用 [az vmss create](/cli/azure/vmss/#az-vmss-create) 建立虛擬機器擴展集。 下列範例會依 `--assign-identity` 參數的要求，建立具有系統指派受控識別且名為 *myVMSS* 的虛擬機器擴展集。 `--admin-username` 和 `--admin-password` 參數會指定登入虛擬機器的系統管理使用者名稱和密碼帳戶。 請針對您的環境適當地更新這些值： 

   ```azurecli-interactive 
   az vmss create --resource-group myResourceGroup --name myVMSS --image win2016datacenter --upgrade-policy-mode automatic --custom-data cloud-init.txt --admin-username azureuser --admin-password myPassword12 --assign-identity --generate-ssh-keys
   ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-virtual-machine-scale-set"></a>在現有 Azure 虛擬機器擴展集上啟用系統指派的受控識別

如果您需要在現有 Azure 虛擬機器擴展集上啟用系統指派的受控識別：

1. 如果您要在本機主控台中使用 Azure CLI，請先使用 [az login](/cli/azure/reference-index#az-login) 登入 Azure。 使用與包含虛擬機器擴展集的 Azure 訂用帳戶相關聯的帳戶。

   ```azurecli-interactive
   az login
   ```

2. 使用 [az vmss identity assign](/cli/azure/vmss/identity/#az-vmss-identity-assign) 命令在現有的 VM 上啟用系統指派的受控識別：

   ```azurecli-interactive
   az vmss identity assign -g myResourceGroup -n myVMSS
   ```

### <a name="disable-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>從 Azure 虛擬機器擴展集停用系統指派的受控識別

如果您的虛擬機器擴展集已不再需要系統指派的受控識別，但仍需要使用者指派的受控識別，請使用下列命令：

```azurecli-interactive
az vmss update -n myVM -g myResourceGroup --set identity.type='UserAssigned' 
```

如果您的虛擬機器不再需要系統指派的受控識別，而且沒有使用者指派的受控識別，請使用下列命令：

> [!NOTE]
> 值 `none` 會區分大小寫。 它必須是小寫字母。 

```azurecli-interactive
az vmss update -n myVM -g myResourceGroup --set identity.type="none"
```

若要移除 Azure 資源受控識別 VM 擴充功能 (計劃在 2019 年 1 月淘汰)，請使用 [az vmss identity remove](/cli/azure/vmss/identity/#az-vmss-remove-identity) 命令從 VMSS 移除系統指派的受控識別：

```azurecli-interactive
az vmss extension delete -n ManagedIdentityExtensionForWindows -g myResourceGroup -vmss-name myVMSS
```

## <a name="user-assigned-managed-identity"></a>使用者指派的受控識別

在本節中，您將了解如何使用 Azure CLI 啟用和移除使用者指派的受控識別。

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-a-virtual-machine-scale-set"></a>在虛擬機器擴展集建立期間指派使用者指派的受控識別

本節會逐步引導您建立虛擬機器，並將使用者指派的受控識別指派給 VMSS。 如果您已經有想要使用的 VMSS，請略過本節並繼續進行下一節。

1. 如果您已經有想要使用的資源群組，可以略過此步驟。 使用 [az group create](/cli/azure/group/#az-group-create) 建立[資源群組](~/articles/azure-resource-manager/resource-group-overview.md#terminology)，以便控制及部署使用者指派的受控識別。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<LOCATION>` 參數的值。 ：

   ```azurecli-interactive 
   az group create --name <RESOURCE GROUP> --location <LOCATION>
   ```

2. 使用 [az identity create](/cli/azure/identity#az-identity-create)，建立使用者指派的受控識別。  `-g` 參數會指定資源群組，以在其中建立使用者指派的受控識別，而 `-n` 參數會指定此資源群組的名稱。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 參數的值：

   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurecli-interactive
   az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
   ```
   回應會包含所建立使用者指派受控識別的詳細資料，與下列內容類似。 指派給使用者指派受控識別的資源 `id` 值會使用於下列步驟。

   ```json
   {
        "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
        "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
        "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>",
        "location": "westcentralus",
        "name": "<USER ASSIGNED IDENTITY NAME>",
        "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
        "resourceGroup": "<RESOURCE GROUP>",
        "tags": {},
        "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
        "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

3. 使用 [az vmss create](/cli/azure/vmss/#az-vmss-create) 建立 VMSS。 下列範例會依 `--assign-identity` 參數的指定內容，建立與新的使用者指派受控識別建立關聯的 VMSS。 別忘了以您自己的值取代 `<RESOURCE GROUP>`、`<VMSS NAME>`、`<USER NAME>`、`<PASSWORD>`、`<USER ASSIGNED IDENTITY>` 參數的值。 

   ```azurecli-interactive 
   az vmss create --resource-group <RESOURCE GROUP> --name <VMSS NAME> --image UbuntuLTS --admin-username <USER NAME> --admin-password <PASSWORD> --assign-identity <USER ASSIGNED IDENTITY>
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-virtual-machine-scale-set"></a>將使用者指派的受控識別指派給現有的虛擬機器擴展集

1. 使用 [az identity create](/cli/azure/identity#az-identity-create)，建立使用者指派的受控識別。  `-g` 參數會指定資源群組，以在其中建立使用者指派的受控識別，而 `-n` 參數會指定此資源群組的名稱。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 參數的值：

    > [!IMPORTANT]
    > 目前不支援在建立使用者指派的受控識別名稱中使用特殊字元 (例如底線)。 請使用英數字元。 請隨時回來查看是否有更新內容。  如需詳細資訊，請參閱[常見問題集和已知問題](known-issues.md)

    ```azurecli-interactive
    az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
    ```
回應會包含所建立使用者指派受控識別的詳細資料，與下列內容類似。

   ```json
   {
        "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
        "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY >/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
        "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY>",
        "location": "westcentralus",
        "name": "<USER ASSIGNED IDENTITY>",
        "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
        "resourceGroup": "<RESOURCE GROUP>",
        "tags": {},
        "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
        "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

2. 使用 [az vmss identity assign](/cli/azure/vmss/identity#az-vm-assign-identity)，將使用者指派的受控識別指派給您的 VMSS。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<VMSS NAME>` 參數的值。 `<USER ASSIGNED IDENTITY>` 是使用者所指派身分識別的資源 `name` 屬性 (在上一個步驟中建立)：

    ```azurecli-interactive
    az vmss identity assign -g <RESOURCE GROUP> -n <VMSS NAME> --identities <USER ASSIGNED IDENTITY>
    ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>從 Azure 虛擬機器擴展集移除使用者指派的受控識別

若要從虛擬機器擴展集移除使用者指派的受控識別，請使用 [az vmss identity remove](/cli/azure/vmss/identity#az-vmss-identity-remove)。 如果這是指派給虛擬機器擴展集的唯一使用者指派的受控識別，將會從識別類型值中移除 `UserAssigned`。  請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<VMSS NAME>` 參數的值。 `<USER ASSIGNED IDENTITY>` 將會是使用者指派受控識別的 `name` 屬性，您可以使用 `az vmss identity show`，在虛擬機器擴展集的識別區段中找到它：

```azurecli-interactive
az vmss identity remove -g <RESOURCE GROUP> -n <VMSS NAME> --identities <USER ASSIGNED IDENTITY>
```

如果您的虛擬機器擴展集沒有系統指派的受控識別，而您想要從其中移除所有使用者指派的受控識別，請使用下列命令：

> [!NOTE]
> 值 `none` 會區分大小寫。 它必須是小寫字母。

```azurecli-interactive
az vmss update -n myVMSS -g myResourceGroup --set identity.type="none" identity.userAssignedIdentities=null
```

如果您的虛擬機器擴展集同時具有系統指派和使用者指派的受控識別，您可以藉由切換為僅使用系統指派的受控識別，來移除所有使用者指派的身分識別。 使用下列命令：

```azurecli-interactive
az vmss update -n myVMSS -g myResourceGroup --set identity.type='SystemAssigned' identity.userAssignedIdentities=null 
```

## <a name="next-steps"></a>後續步驟

- [Azure 資源受控識別概觀](overview.md)
- 如需建立 Azure 虛擬機器擴展集的完整快速入門，請參閱： 

  - [使用 CLI 建立虛擬機器擴展集](../../virtual-machines/linux/tutorial-create-vmss.md#create-a-scale-set)

















