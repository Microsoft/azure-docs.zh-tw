---
title: 使用 RBAC 和 Azure Resource Manager 範本來管理存取權 | Microsoft Docs
description: 了解如何使用角色型存取控制 (RBAC) 和 Azure Resource Manager 範本來管理使用者、群組和應用程式的存取權。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/17/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 5e080614d4f0001a0bf1b44dd402f37db2463e03
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39206615"
---
# <a name="manage-access-using-rbac-and-azure-resource-manager-templates"></a>使用 RBAC 和 Azure Resource Manager 範本來管理存取權

[角色型存取控制 (RBAC)](overview.md) 是您對 Azure 中的資源存取進行管理的機制。 除了使用 Azure PowerShell 或 Azure CLI 外，您可以使用 RBAC 和 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)來管理 Azure 資源的存取權。 如果您需要以一致性和重複的方式來部署資源，範本可以派上用場。 本文描述如何使用 RBAC 和範本來管理存取權。

## <a name="example-template-to-create-a-role-assignment"></a>建立角色指派時可以套用的範例範本

在 RBAC 中，若要授與存取權，您可以建立角色指派。 下列範本會示範：
- 如何將角色指派給資源群組範圍中的使用者、群組或應用程式
- 如何將擁有者、參與者和讀者角色指定為參數

若要使用範本，您必須指定下列輸入：
- 資源群組的名稱
- 要為其指派角色之使用者、群組或應用程式讀唯一識別碼
- 要指派的角色
- 將用於角色指派的唯一識別碼

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The principal to assign the role to"
      }
    },
    "builtInRoleType": {
      "type": "string",
      "allowedValues": [
        "Owner",
        "Contributor",
        "Reader"
      ],
      "metadata": {
        "description": "Built-in role to assign"
      }
    },
    "roleNameGuid": {
      "type": "string",
      "metadata": {
        "description": "A new GUID used to identify the role assignment"
      }
    }
  },
  "variables": {
    "Owner": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '8e3af657-a8ff-443c-a75c-2fe8c4bcb635')]",
    "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
    "Reader": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]",
    "scope": "[resourceGroup().id]"
  },
  "resources": [
    {
      "type": "Microsoft.Authorization/roleAssignments",
      "apiVersion": "2017-05-01",
      "name": "[parameters('roleNameGuid')]",
      "properties": {
        "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('scope')]"
      }
    }
  ]
}
```

以下顯示在部署範本之後，向使用者指派讀者角色的範例。

![使用範本的角色指派](./media/role-assignments-template/role-assignment-template.png)

## <a name="deploy-template-using-azure-powershell"></a>使用 Azure PowerShell 來部署範本

若要使用 Azure PowerShell 來部署先前的範本，請按照下列步驟執行。

1. 建立名為 rbac-rg.json 的新檔案並複製先前的範本。

1. 登入 [Azure PowerShell](/powershell/azure/authenticate-azureps)。

1. 取得使用者、群組或應用程式的唯一識別碼。 例如，您可以使用 [Get-AzureRmADUser](/powershell/module/azurerm.resources/get-azurermaduser) 命令來列出 Azure AD 使用者。

    ```azurepowershell
    Get-AzureRmADUser
    ```

1. 使用 GUID 工具來產生將用於角色指派的唯一識別碼。 此識別碼的格式：`11111111-1111-1111-1111-111111111111`

1. 建立範例 Azure 資源群組。

    ```azurepowershell
    New-AzureRmResourceGroup -Name ExampleGroup -Location "Central US"
    ```

1. 使用 [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) 命令來開始部署。

    ```azurepowershell
    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile rbac-rg.json
    ```

    系統會要求您指定必要參數。 以下顯示輸出的範例。

    ```Output
    PS /home/user> New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile rbac-rg.json
    
    cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
    Supply values for the following parameters:
    (Type !? for Help.)
    principalId: 22222222-2222-2222-2222-222222222222
    builtInRoleType: Reader
    roleNameGuid: 11111111-1111-1111-1111-111111111111
    
    DeploymentName          : rbac-rg
    ResourceGroupName       : ExampleGroup
    ProvisioningState       : Succeeded
    Timestamp               : 7/17/2018 7:46:32 PM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name             Type                       Value
                              ===============  =========================  ==========
                              principalId      String                     22222222-2222-2222-2222-222222222222
                              builtInRoleType  String                     Reader
                              roleNameGuid     String                     11111111-1111-1111-1111-111111111111
    
    Outputs                 :
    DeploymentDebugLogLevel :
    ```

## <a name="deploy-template-using-the-azure-cli"></a>使用 Azure CLI 部署範本

若要使用 Azure CLI 來部署先前的範本，請按照下列步驟執行。

1. 建立名為 rbac-rg.json 的新檔案並複製先前的範本。

1. 登入 [Azure CLI](/cli/azure/authenticate-azure-cli)。

1. 取得使用者、群組或應用程式的唯一識別碼。 例如，您可以使用 [az ad user list](/cli/azure/ad/user#az-ad-user-list) 命令來列出 Azure AD 使用者。

    ```azurecli
    az ad user list
    ```

1. 使用 GUID 工具來產生將用於角色指派的唯一識別碼。 此識別碼的格式：`11111111-1111-1111-1111-111111111111`

1. 建立範例 Azure 資源群組。

    ```azurecli
    az group create --name ExampleGroup --location "Central US"
    ```

1. 使用 [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create) 命令來開始部署。

    ```azurecli
    az group deployment create --resource-group ExampleGroup --template-file rbac-rg.json
    ```

    系統會要求您指定必要參數。 以下顯示輸出的範例。

    ```Output
    C:\Azure\Templates>az group deployment create --resource-group ExampleGroup --template-file rbac-rg.json
    Please provide string value for 'principalId' (? for help): 22222222-2222-2222-2222-222222222222
    Please provide string value for 'builtInRoleType' (? for help):
     [1] Owner
     [2] Contributor
     [3] Reader
    Please enter a choice [1]: 3
    Please provide string value for 'roleNameGuid' (? for help): 11111111-1111-1111-1111-111111111111
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/rbac-rg",
      "name": "rbac-rg",
      "properties": {
        "additionalProperties": {
          "duration": "PT9.5323924S",
          "outputResources": [
            {
              "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/ExampleGroup/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
              "resourceGroup": "ExampleGroup"
            }
          ],
          "templateHash": "0000000000000000000"
        },
        "correlationId": "33333333-3333-3333-3333-333333333333",
        "debugSetting": null,
        "dependencies": [],
        "mode": "Incremental",
        "outputs": null,
        "parameters": {
          "builtInRoleType": {
            "type": "String",
            "value": "Reader"
          },
          "principalId": {
            "type": "String",
            "value": "22222222-2222-2222-2222-222222222222"
          },
          "roleNameGuid": {
            "type": "String",
            "value": "11111111-1111-1111-1111-111111111111"
          }
        },
        "parametersLink": null,
        "providers": [
          {
            "id": null,
            "namespace": "Microsoft.Authorization",
            "registrationState": null,
            "resourceTypes": [
              {
                "aliases": null,
                "apiVersions": null,
                "locations": [
                  null
                ],
                "properties": null,
                "resourceType": "roleAssignments"
              }
            ]
          }
        ],
        "provisioningState": "Succeeded",
        "template": null,
        "templateLink": null,
        "timestamp": "2018-07-17T19:00:31.830904+00:00"
      },
      "resourceGroup": "ExampleGroup"
    }
    ```
    
## <a name="next-steps"></a>後續步驟

- [建立及部署第一個 Azure Resource Manager 範本](../azure-resource-manager/resource-manager-create-first-template.md)
- [了解 Azure Resource Manager 範本的的結構和語法](../azure-resource-manager/resource-group-authoring-templates.md)
- [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/?term=rbac)
