---
title: 使用 REST API 來建立自訂角色 - Azure | Microsoft Docs
description: 了解如何使用 REST API 來建立適用於角色型存取控制 (RBAC) 的自訂角色。 這包括如何列出、建立、更新及刪除自訂角色。
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: 1f90228a-7aac-4ea7-ad82-b57d222ab128
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: conceptual
ms.date: 06/20/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 8a1bbe8217e2d4a9846f56124e248e19cbe70b19
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37436057"
---
# <a name="create-custom-roles-using-the-rest-api"></a>使用 REST API 來建立自訂角色

如果[內建角色](built-in-roles.md)不符合組織的特定需求，您可以建立自己的自訂角色。 本文說明如何使用 REST API 來建立和管理自訂角色。

## <a name="list-roles"></a>列出角色

若要列出所有角色或使用角色的顯示名稱來取得單一角色的相關資訊，請使用[角色定義 - 列出](/rest/api/authorization/roledefinitions/list) REST API。 若要呼叫此 API，您必須能夠存取該範圍的 `Microsoft.Authorization/roleDefinitions/read` 作業。 有數個[內建角色](built-in-roles.md)已獲得此作業的存取權。

1. 從下列要求著手：

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. 在 URI 內，將 *{scope}* 取代為要列出角色的範圍。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{filter}* 取代為您要針對角色清單篩選套用的條件。

    | Filter | 說明 |
    | --- | --- |
    | `$filter=atScopeAndBelow()` | 列出在指定的範圍及其任何子範圍內可供指派的角色。 |
    | `$filter=roleName%20eq%20'{roleDisplayName}'` | 使用角色確切顯示名稱的 URL 編碼型式。 例如 `$filter=roleName%20eq%20'Virtual%20Machine%20Contributor'` |

### <a name="get-information-about-a-role"></a>取得角色的相關資訊

若要使用角色定義識別碼來取得角色的相關資訊，請使用[角色定義 - 取得](/rest/api/authorization/roledefinitions/get) REST API。 若要呼叫此 API，您必須能夠存取該範圍的 `Microsoft.Authorization/roleDefinitions/read` 作業。 有數個[內建角色](built-in-roles.md)已獲得此作業的存取權。

若要使用角色的顯示名稱來取得單一角色的相關資訊，請參閱先前的[列出角色](custom-roles-rest.md#list-roles)一節。

1. 使用[角色定義 - 列出](/rest/api/authorization/roledefinitions/list) REST API 來取得角色的 GUID 識別碼。 針對內建角色，您也可以從[內建角色](built-in-roles.md)取得識別碼。

1. 從下列要求著手：

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. 在 URI 內，將 *{scope}* 取代為要列出角色的範圍。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{roleDefinitionId}* 取代為角色定義的 GUID 識別碼。

## <a name="create-a-custom-role"></a>建立自訂角色

若要建立自訂角色，請使用[角色定義 - 建立或更新](/rest/api/authorization/roledefinitions/createorupdate) REST API。 若要呼叫此 API，您必須能夠存取所有 `assignableScopes` 的 `Microsoft.Authorization/roleDefinitions/write` 作業。 在內建角色中，只有[擁有者](built-in-roles.md#owner)和[使用者存取系統管理員](built-in-roles.md#user-access-administrator)會獲得這項作業的存取權。 

1. 檢閱可用來為自訂角色建立權限的[資源提供者作業](resource-provider-operations.md)清單。

1. 使用 GUID 工具來產生將用來作為自訂角色識別碼的唯一識別碼。 此識別碼的格式：`00000000-0000-0000-0000-000000000000`

1. 從下列要求和本文著手：

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId}"
        ]
      }
    }
    ```

1. 在 URI 內，將 *{scope}* 取代為自訂角色的第一個 `assignableScopes`。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{roleDefinitionId}* 取代為自訂角色的 GUID 識別碼。

1. 在要求本文內的 `assignableScopes` 屬性中，將 *{roleDefinitionId}* 取代為角色定義的 GUID 識別碼。

1. 將 *{subscriptionId}* 取代為您的訂用帳戶識別碼。

1. 在 `actions` 屬性中，新增角色允許執行的作業。

1. 在 `notActions` 屬性中，新增從允許的 `actions` 中排除的作業。

1. 在 `roleName` 和 `description` 屬性中，指定唯一角色名稱和描述。 如需有關屬性的資訊，請參閱[自訂角色](custom-roles.md)。

    以下顯示要求本文範例：

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
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
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ]
      }
    }
    ```

## <a name="update-a-custom-role"></a>更新自訂角色

若要更新自訂角色，請使用[角色定義 - 建立或更新](/rest/api/authorization/roledefinitions/createorupdate) REST API。 若要呼叫此 API，您必須能夠存取所有 `assignableScopes` 的 `Microsoft.Authorization/roleDefinitions/write` 作業。 在內建角色中，只有[擁有者](built-in-roles.md#owner)和[使用者存取系統管理員](built-in-roles.md#user-access-administrator)會獲得這項作業的存取權。 

1. 使用[角色定義 - 列出](/rest/api/authorization/roledefinitions/list)或[角色定義 - 取得](/rest/api/authorization/roledefinitions/get) REST API 來取得自訂角色的相關資訊。 如需詳細資訊，請參閱稍早的[列出角色](custom-roles-rest.md#list-roles)一節。

1. 從下列要求著手：

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. 在 URI 內，將 *{scope}* 取代為自訂角色的第一個 `assignableScopes`。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{roleDefinitionId}* 取代為自訂角色的 GUID 識別碼。

1. 根據自訂角色的相關資訊，以下列格式建立要求本文：

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId}"
        ]
      }
    }
    ```

1. 以您想要對自訂角色進行的變更更新要求本文。

    以下顯示已新增診斷設定動作的要求本文範例：

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
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
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ]
      }
    }
    ```

## <a name="delete-a-custom-role"></a>刪除自訂角色

若要刪除自訂角色，請使用[角色定義 - 刪除](/rest/api/authorization/roledefinitions/delete) REST API。 若要呼叫此 API，您必須能夠存取所有 `assignableScopes` 的 `Microsoft.Authorization/roleDefinitions/delete` 作業。 在內建角色中，只有[擁有者](built-in-roles.md#owner)和[使用者存取系統管理員](built-in-roles.md#user-access-administrator)會獲得這項作業的存取權。 

1. 使用[角色定義 - 列出](/rest/api/authorization/roledefinitions/list)或[角色定義 - 取得](/rest/api/authorization/roledefinitions/get) REST API 來取得自訂角色的 GUID 識別碼。 如需詳細資訊，請參閱稍早的[列出角色](custom-roles-rest.md#list-roles)一節。

1. 從下列要求著手：

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. 在 URI 內，將 *{scope}* 取代為要刪除自訂角色的範圍。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{roleDefinitionId}* 取代為自訂角色的 GUID 識別碼。

## <a name="next-steps"></a>後續步驟

- [Azure 中的自訂角色](custom-roles.md)
- [使用 RBAC 和 REST API 來管理存取權](role-assignments-rest.md)
- [Azure REST API 參考](/rest/api/azure/)