---
title: 使用 Azure 原則來限制 VM 延伸模組安裝 | Microsoft Docs
description: 使用 Azure 原則來限制 VM 延伸模組部署。
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/23/2018
ms.author: roiyz;cynthn
ms.openlocfilehash: 529758a7b9fe4c8b669ade72273335389020fb65
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451198"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-linux-vms"></a>使用 Azure 原則來限制 Linux VM 上的延伸模組安裝

如果您想要防止在 Linux VM 上使用或安裝特定的延伸模組，便可以使用 CLI 來建立 Azure 原則，以限制某個資源群組內 VM 的延伸模組。 

本教學課程會使用 Azure Cloud Shell 內的 CLI，這會不斷更新至最新版本。 如果您想要在本機執行 Azure CLI，則必須安裝 2.0.26 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="create-a-rules-file"></a>建立規則檔

為了限制可安裝的延伸模組，您必須要有一個[規則](/azure/azure-policy/policy-definition#policy-rule)來提供識別延伸模組的邏輯。

以下範例說明如何在 Azure Cloud Shell 中建立規則檔來拒絕安裝 'Microsoft.OSTCExtensions' 所發佈的延伸模組，但如果您是在本機 CLI 中運作，則也可以建立本機檔案，然後將路徑 (~/clouddrive) 取代成您電腦上該本機檔案的路徑。

在 [bash Cloud Shell](https://shell.azure.com/bash) 中，輸入：

```azurecli-interactive 
vim ~/clouddrive/azurepolicy.rules.json
```

將下列 .json 複製並貼到檔案中。

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.OSTCExtensions/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/publisher",
                "equals": "Microsoft.OSTCExtensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

完成時，按 **Esc** 鍵，然後輸入 **:wq** 以儲存並關閉檔案。


## <a name="create-a-parameters-file"></a>建立參數檔案

您也需要一個[參數](/azure/azure-policy/policy-definition#parameters)檔，以建立供您用於傳遞要封鎖之延伸模組清單的結構。 

以下範例說明如何在 Cloud Shell 中為 Linux VM 建立參數檔，但如果您是在本機 CLI 中運作，則也可以建立本機檔案，然後將路徑 (~/clouddrive) 取代成您電腦上該本機檔案的路徑。

在 [bash Cloud Shell](https://shell.azure.com/bash) 中，輸入：

```azurecli-interactive
vim ~/clouddrive/azurepolicy.parameters.json
```

將下列 .json 複製並貼到檔案中。

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied. Example: CustomScriptForLinux, VMAccessForLinux etc.",
            "strongType": "type",
            "displayName": "Denied extension"
        }
    }
}
```

完成時，按 **Esc** 鍵，然後輸入 **:wq** 以儲存並關閉檔案。

## <a name="create-the-policy"></a>建立原則

原則定義是一個用來儲存您所要使用之設定的物件。 原則定義會使用規則檔和參數檔來定義原則。 請使用 [az policy definition create](/cli/azure/role/assignment?view=azure-cli-latest#az_role_assignment_create) 來建立原則定義。

在此範例中，規則和參數是您在 Cloud Shell 中以 .json 檔案形式建立並儲存的檔案。

```azurecli-interactive
az policy definition create \
   --name 'not-allowed-vmextension-linux' \
   --display-name 'Block VM Extensions' \
   --description 'This policy governs which VM extensions that are blocked.' \
   --rules '~/clouddrive/azurepolicy.rules.json' \
   --params '~/clouddrive/azurepolicy.parameters.json' \
   --mode All
```


## <a name="assign-the-policy"></a>指派原則

以下範例會使用 [az policy assignment create](/cli/azure/policy/assignment#az_policy_assignment_create) 將原則指派給資源群組。 任何建立在 **myResourceGroup** 資源群組中的 VM 都將無法安裝「Linux VM 存取」或適用於 Linux 的「自訂指令碼」延伸模組。 資源群組必須已存在，您才能指派原則。

請使用 [az account list](/cli/azure/account?view=azure-cli-latest#az_account_list) 來取得您的訂用帳戶 ID，以用來取代範例中的 ID。


```azurecli-interactive
az policy assignment create \
   --name 'not-allowed-vmextension-linux' \
   --scope /subscriptions/<subscription Id>/resourceGroups/myResourceGroup \
   --policy "not-allowed-vmextension-linux" \
   --params '{
        "notAllowedExtensions": {
            "value": [
                "VMAccessForLinux",
                "CustomScriptForLinux"
            ]
        }
    }'
```

## <a name="test-the-policy"></a>測試原則

建立新的 VM 並嘗試新增使用者來測試原則。


```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --generate-ssh-keys
```

使用「VM 存取」延伸模組來嘗試建立一個名為 **myNewUser** 的新使用者。

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --password 'mynewuserpwd123!'
```



## <a name="remove-the-assignment"></a>移除指派

```azurecli-interactive
az policy assignment delete --name 'not-allowed-vmextension-linux' --resource-group myResourceGroup
```
## <a name="remove-the-policy"></a>移除原則

```azurecli-interactive
az policy definition delete --name 'not-allowed-vmextension-linux'
```


## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Azure 原則](../../azure-policy/azure-policy-introduction.md)。
