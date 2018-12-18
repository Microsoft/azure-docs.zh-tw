---
title: Azure 中的自訂角色 | Microsoft Docs
description: 了解如何使用 Azure 角色型存取控制 (RBAC) 定義自訂角色，以對 Azure 中的資源進行微調存取管理。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: e4206ea9-52c3-47ee-af29-f6eef7566fa5
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 281c426170985d43401a13988218126ea3951634
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405609"
---
# <a name="custom-roles-in-azure"></a>Azure 中的自訂角色

如果[內建角色](built-in-roles.md)不符合組織的特定需求，您可以建立自己的自訂角色。 就像內建角色一樣，您可以將自訂角色指派給訂用帳戶、資源群組和資源範圍的使用者、群組和服務主體。 自訂角色是存放在 Azure Active Directory (Azure AD) 目錄中，而且可以跨訂用帳戶共用。 每個目錄最多可以有 2000 個自訂角色。 可以使用 Azure PowerShell、Azure CLI 和 REST API 建立自訂角色。

## <a name="custom-role-example"></a>自訂角色範例

以下顯示自訂角色以 JSON 格式顯示時的外觀。 此自訂角色可用於監視和重新啟動虛擬機器。

```json
{
  "Name":  "Virtual Machine Operator",
  "Id":  "88888888-8888-8888-8888-888888888888",
  "IsCustom":  true,
  "Description":  "Can monitor and restart virtual machines.",
  "Actions":  [
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
  "NotActions":  [

                 ],
  "DataActions":  [

                  ],
  "NotDataActions":  [

                     ],
  "AssignableScopes":  [
                           "/subscriptions/{subscriptionId1}",
                           "/subscriptions/{subscriptionId2}",
                           "/subscriptions/{subscriptionId3}"
                       ]
}
```

建立自訂角色時，它會以橙色資源圖示顯示在 Azure 入口網站中。

![自訂角色圖示](./media/custom-roles/roles-custom-role-icon.png)

## <a name="steps-to-create-a-custom-role"></a>建立自訂角色的步驟

1. 判斷您所需的權限

    在建立自訂角色時，您必須知道可用來定義權限的資源提供者作業。 若要檢視作業的清單，您可以使用 [Get-AzureRMProviderOperation](/powershell/module/azurerm.resources/get-azurermprovideroperation) 或 [az provider operation list](/cli/azure/provider/operation#az-provider-operation-list) 命令。
    若要指定自訂角色的權限，您可以將作業增至[角色定義](role-definitions.md)的 `Actions` 或 `NotActions` 屬性。 如果您有資料作業，可以將它們新增至 `DataActions` 或 `NotDataActions` 屬性。

2. 建立自訂角色

    您可以使用 Azure PowerShell 或 Azure CLI 來建立自訂角色。 一般而言，您可以從使用現有的內建角色開始，然後針對您的需求進行修改。 接著，您可以使用 [New-AzureRmRoleDefinition](/powershell/module/azurerm.resources/new-azurermroledefinition) 或 [az role definition create](/cli/azure/role/definition#az-role-definition-create) 命令來建立自訂角色。 若要建立自訂角色，您必須擁有所有 `AssignableScopes` 的 `Microsoft.Authorization/roleDefinitions/write` 權限，例如[擁有者](built-in-roles.md#owner)或[使用者存取系統管理員](built-in-roles.md#user-access-administrator)。

3. 測試自訂角色

    一旦具有自訂角色，您必須測試它來驗證是否如預期般運作。 稍後如需進行調整，您可以更新自訂角色。

如需如何建立自訂角色的逐步教學課程，請參閱[教學課程：使用 Azure PowerShell 建立自訂角色](tutorial-custom-role-powershell.md)或[教學課程：使用 Azure CLI 建立自訂角色](tutorial-custom-role-cli.md)。

## <a name="custom-role-properties"></a>自訂角色屬性

自訂角色具有下列屬性。

| 屬性 | 必要 | 類型 | 說明 |
| --- | --- | --- | --- |
| `Name` | 是 | 字串 | 自訂角色的顯示名稱。 當角色定義是訂用帳戶層級資源時，角色定義可在多個共用相同 Azure AD 目錄的訂用帳戶中使用。 此顯示名稱在 Azure AD 目錄範圍中必須是唯一的。 可以包含字母、數字、空格和特殊字元。 字元數目上限是 128。 |
| `Id` | 是 | 字串 | 自訂角色的唯一識別碼。 針對 Azure PowerShell 和 Azure CLI，當您建立新角色時，會自動產生這個識別碼。 |
| `IsCustom` | 是 | 字串 | 表示這是否為自訂角色。 若為自訂角色，請設定為 `true`。 |
| `Description` | 是 | 字串 | 自訂角色的描述。 可以包含字母、數字、空格和特殊字元。 字元數目上限是 1024。 |
| `Actions` | 是 | String[] | 字串陣列，指定角色允許執行的管理作業。 如需詳細資訊，請參閱 [Actions](role-definitions.md#actions)。 |
| `NotActions` | 否 | String[] | 字串陣列，指定從所允許 `Actions` 中排除的管理作業。 如需詳細資訊，請參閱 [NotActions](role-definitions.md#notactions)。 |
| `DataActions` | 否 | String[] | 字串陣列，指定角色允許對物件內資料執行的管理作業。 如需詳細資訊，請參閱 [DataActions (預覽)](role-definitions.md#dataactions-preview)。 |
| `NotDataActions` | 否 | String[] | 字串陣列，指定從所允許 `DataActions` 中排除的資料作業。 如需詳細資訊，請參閱 [NotDataActions (預覽)](role-definitions.md#notdataactions-preview)。 |
| `AssignableScopes` | 是 | String[] | 字串陣列，指定自訂角色可用於指派的範圍。 目前無法設定為根範圍 (`"/"`) 或管理群組範圍。 如需詳細資訊，請參閱 [AssignableScopes](role-definitions.md#assignablescopes) 和[使用 Azure 管理群組來組織資源](../governance/management-groups/index.md#custom-rbac-role-definition-and-assignment)。 |

## <a name="who-can-create-delete-update-or-view-a-custom-role"></a>誰可以建立、刪除、更新或檢視自訂角色

就像內建角色一樣，`AssignableScopes` 屬性會指定角色可用於指派的範圍。 自訂角色的 `AssignableScopes` 屬性也會控制誰可以建立、刪除、更新或檢視自訂角色。

| Task | 作業 | 說明 |
| --- | --- | --- |
| 建立/刪除自訂角色 | `Microsoft.Authorization/ roleDefinition/write` | 獲得授權可對自訂角色的所有 `AssignableScopes` 執行此作業的使用者，可以建立 (或刪除) 用於這些範圍的自訂角色。 例如，訂用帳戶、資源群組和資源的[擁有者](built-in-roles.md#owner)和[使用者存取系統管理員](built-in-roles.md#user-access-administrator)。 |
| 更新自訂角色 | `Microsoft.Authorization/ roleDefinition/write` | 獲得授權可對自訂角色的所有 `AssignableScopes` 執行此作業的使用者，可以在這些範圍中更新自訂角色。 例如，訂用帳戶、資源群組和資源的[擁有者](built-in-roles.md#owner)和[使用者存取系統管理員](built-in-roles.md#user-access-administrator)。 |
| 檢視自訂角色 | `Microsoft.Authorization/ roleDefinition/read` | 獲得授權可在範圍中執行此作業的使用者，可以檢視可指派給該範圍的自訂角色。 所有內建角色都允許指派自訂角色。 |

## <a name="next-steps"></a>後續步驟
- [使用 Azure PowerShell 建立自訂角色](custom-roles-powershell.md)
- [使用 Azure CLI 建立自訂角色](custom-roles-cli.md)
- [了解角色定義](role-definitions.md)
