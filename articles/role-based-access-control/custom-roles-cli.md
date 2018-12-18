---
title: 使用 Azure CLI 建立自訂角色 | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立適用於角色型存取控制 (RBAC) 的自訂角色。 這包括如何列出、建立、更新及刪除自訂角色。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/20/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 3b5d18a3e0bf846137dfdf68b8e5dd9e2db58792
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37437251"
---
# <a name="create-custom-roles-using-azure-cli"></a>使用 Azure CLI 建立自訂角色

如果內建的角色無法滿足組織的特定需求，您可以建立自己的[自訂角色](built-in-roles.md)。 本文說明如何使用 Azure CLI 來建立和管理自訂角色。

## <a name="prerequisites"></a>先決條件

若要建立自訂角色，您需要：

- 建立自訂角色的權限，例如[擁有者](built-in-roles.md#owner)或[使用者存取管理員](built-in-roles.md#user-access-administrator)
- 已在本機安裝 [Azure CLI](/cli/azure/install-azure-cli)

## <a name="list-custom-roles"></a>列出自訂角色

若要列出可供指派的自訂角色，請使用 [az role definition list](/cli/azure/role/definition#az-role-definition-list)。 下列範例會列出目前訂用帳戶中的所有自訂角色。

```azurecli
az role definition list --custom-role-only true --output json | jq '.[] | {"roleName":.roleName, "roleType":.roleType}'
```

```azurecli
az role definition list --output json | jq '.[] | if .roleType == "CustomRole" then {"roleName":.roleName, "roleType":.roleType} else empty end'
```

```Output
{
  "roleName": "My Management Contributor",
  "type": "CustomRole"
}
{
  "roleName": "My Service Reader Role",
  "type": "CustomRole"
}
{
  "roleName": "Virtual Machine Operator",
  "type": "CustomRole"
}

...
```

## <a name="create-a-custom-role"></a>建立自訂角色

若要建立自訂角色，請使用 [az role definition create](/cli/azure/role/definition#az-role-definition-create)。 角色定義可以是 JSON 描述或包含 JSON 描述的檔案路徑。

```azurecli
az role definition create --role-definition <role_definition>
```

下列範例會建立一個名為 Virtual Machine Operator 的自訂角色。 這個自訂角色會指派 Microsoft.Compute、Microsoft.Storage 和 Microsoft.Network 資源提供者之所有讀取作業的存取權，以及指派啟動、重新啟動和監視虛擬機器的存取權。 這個自訂角色可用於兩個訂用帳戶中。 這個範例使用 JSON 檔案做為輸入。

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111",
    "/subscriptions/33333333-3333-3333-3333-333333333333"
  ]
}
```

```azurecli
az role definition create --role-definition ~/roles/vmoperator.json
```

## <a name="update-a-custom-role"></a>更新自訂角色

若要更新自訂角色，請先使用 [az role definition list](/cli/azure/role/definition#az-role-definition-list) 來擷取角色定義。 接著，對角色定義進行想要的變更。 最後，使用 [az role definition update](/cli/azure/role/definition#az-role-definition-update)儲存更新的角色定義。

```azurecli
az role definition update --role-definition <role_definition>
```

下列範例會將 Microsoft.Insights/diagnosticSettings/ 作業新增到 Virtual Machine Operator 自訂角色的 Actions 中。

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111",
    "/subscriptions/33333333-3333-3333-3333-333333333333"
  ]
}
```

```azurecli
az role definition update --role-definition ~/roles/vmoperator.json
```

## <a name="delete-a-custom-role"></a>刪除自訂角色

若要刪除自訂角色，請使用 [az role definition delete](/cli/azure/role/definition#az-role-definition-delete)。 若要指定要刪除的角色，請使用角色名稱或角色識別碼。 若要決定角色識別碼，請使用 [az role definition list](/cli/azure/role/definition#az-role-definition-list)。

```azurecli
az role definition delete --name <role_name or role_id>
```

下列範例會刪除 Virtual Machine Operator 自訂角色。

```azurecli
az role definition delete --name "Virtual Machine Operator"
```

## <a name="next-steps"></a>後續步驟

- [教學課程：使用 Azure CLI 建立自訂角色](tutorial-custom-role-cli.md)
- [Azure 中的自訂角色](custom-roles.md)
- [Azure Resource Manager 資源提供者作業](resource-provider-operations.md)