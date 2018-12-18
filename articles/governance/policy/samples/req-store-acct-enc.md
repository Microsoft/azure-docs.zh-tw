---
title: Azure 原則範例 - 要求儲存體帳戶加密
description: 此範例原則會要求儲存體帳戶使用 Blob 加密。
services: azure-policy
author: DCtheGeek
manager: carmonm
ms.service: azure-policy
ms.topic: sample
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: mvc
ms.openlocfilehash: ad43acc319a79bf3f864441d6c404145df7074c7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972958"
---
# <a name="require-storage-account-encryption"></a>需要儲存體帳戶加密

此原則會要求儲存體帳戶使用 Blob 加密。

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-template"></a>範例範本

[!code-json[main](../../../../policy-templates/samples/built-in-policy/require-storageaccount-encryption/azurepolicy.json "Require storage account encryption")]

您可以使用 [Azure 入口網站](#deploy-with-the-portal) [PowerShell](#deploy-with-powershell) 或 [Azure CLI](#deploy-with-azure-cli) 來部署此範本。

## <a name="deploy-with-the-portal"></a>使用入口網站部署

[![部署至 Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/?feature.customportal=false&microsoft_azure_policy=true&microsoft_azure_policy_policyinsights=true&feature.microsoft_azure_security_policy=true&microsoft_azure_marketplace_policy=true#blade/Microsoft_Azure_Policy/CreatePolicyDefinitionBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-policy%2Fmaster%2Fsamples%2Fbuilt-in-policy%2Frequire-storageaccount-encryption%2Fazurepolicy.json)

## <a name="deploy-with-powershell"></a>使用 PowerShell 部署

[!INCLUDE [sample-powershell-install](../../../../includes/sample-powershell-install-no-ssh.md)]

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition -Name "require-storageaccount-encryption" -DisplayName "Require storage account encryption" -description "This policy ensures encryption for storage accounts is turned on." -Policy 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-storageaccount-encryption/azurepolicy.rules.json' -Parameter 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-storageaccount-encryption/azurepolicy.parameters.json' -Mode All
$definition
$assignment = New-AzureRMPolicyAssignment -Name <assignmentname> -Scope <scope>  -PolicyDefinition $definition
$assignment
```

### <a name="clean-up-powershell-deployment"></a>清除 PowerShell 部署

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="deploy-with-azure-cli"></a>使用 Azure CLI 進行部署

[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

```azurecli-interactive
az policy definition create --name 'require-storageaccount-encryption' --display-name 'Require storage account encryption' --description 'This policy ensures encryption for storage accounts is turned on.' --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-storageaccount-encryption/azurepolicy.rules.json' --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-storageaccount-encryption/azurepolicy.parameters.json' --mode All

az policy assignment create --name <assignmentname> --scope <scope> --policy "require-storageaccount-encryption"
```

### <a name="clean-up-azure-cli-deployment"></a>清除 Azure CLI 部署

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

- 在 [Azure 原則範例](index.md)檢閱更多範例