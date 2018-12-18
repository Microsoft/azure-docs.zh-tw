---
title: 教學課程 - 使用 Azure CLI 建立自訂角色 | Microsoft Docs
description: 開始使用 Azure CLI 建立自訂角色。
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 06/12/2018
ms.author: rolyon
ms.openlocfilehash: 6302ae3bbb97f8f40733074b9d9dc708d10641ca
ms.sourcegitcommit: 65b399eb756acde21e4da85862d92d98bf9eba86
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/22/2018
ms.locfileid: "36322588"
---
# <a name="tutorial-create-a-custom-role-using-azure-cli"></a>教學課程：使用 Azure CLI 建立自訂角色

如果內建的角色無法滿足您組織的特定需求，您可以建立自己的[自訂角色](built-in-roles.md)。 在此教學課程中，您會使用 Azure CLI 建立名為讀者支援票證的自訂角色。 自訂角色可讓使用者檢視訂用帳戶中的一切，也可開啟支援票證。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立自訂角色
> * 列出自訂角色
> * 更新自訂角色
> * 刪除自訂角色

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>先決條件

若要完成本教學課程，您需要：

- 建立自訂角色的權限，例如[擁有者](built-in-roles.md#owner)或[使用者存取管理員](built-in-roles.md#user-access-administrator)
- 已在本機安裝 [Azure CLI](/cli/azure/install-azure-cli)

## <a name="sign-in-to-azure-cli"></a>登入 Azure CLI

登入 [Azure CLI](/cli/azure/authenticate-azure-cli)。

## <a name="create-a-custom-role"></a>建立自訂角色

建立自訂角色的最簡單方法就是從 JSON 範本著手，新增您的變更，然後建立新的角色。

1. 檢閱 [Microsoft.Support 資源提供者](resource-provider-operations.md#microsoftsupport)的作業清單。 最好先了解可用來建立權限的作業。

    | 作業 | 說明 |
    | --- | --- |
    | Microsoft.Support/register/action | 支援資源提供者的暫存器 |
    | Microsoft.Support/supportTickets/read | 取得支援票證的詳細資料 (包括狀態、嚴重性、連絡人詳細資料和通訊)，或取得跨訂用帳戶之支援票證的清單。 |
    | Microsoft.Support/supportTickets/write | 建立或更新支援票證。 您可以建立技術、帳單、配額或訂用帳戶管理相關問題的支援票證。 您可以更新現有支援票證的嚴重性、連絡人詳細資料和通訊。 |
    
1. 建立名為 **ReaderSupportRole.json** 的新檔案。

1. 在編輯器中開啟 ReaderSupportRole.json 並新增下列 JSON。

    如需不同屬性的相關資訊，請參閱[自訂角色](custom-roles.md)。

    ```json
    {
        "Name":  "",
        "IsCustom":  true,
        "Description":  "",
        "Actions":  [
    
                    ],
        "NotActions":  [
    
                       ],
        "DataActions":  [
    
                        ],
        "NotDataActions":  [
    
                           ],
        "AssignableScopes":  [
                                 "/subscriptions/{subscriptionId1}"
                             ]
    }
    ```
    
1. 將下列作業新增至 `Actions` 屬性。 這些動作可讓使用者檢視訂用帳戶中的一切，以及建立支援票證。

    ```
    "*/read",
    "Microsoft.Support/*"
    ```

1. 使用 [az account list](/cli/azure/account#az-account-list) 命令取得訂用帳戶的識別碼。

    ```azurecli
    az account list --output table
    ```

1. 在 `AssignableScopes` 中，使用您的訂用帳戶識別碼來取代 `{subscriptionId1}`。

    您必須新增明確的訂用帳戶識別碼，否則無法將角色匯入您的訂用帳戶。

1. 將 `Name` 和 `Description` 屬性變更為 [讀者支援票證] 和 [檢視訂用帳戶中的所有資訊，包括開啟的支援票證]。

    JSON 檔案看起來應該如下所示：

    ```json
    {
        "Name":  "Reader Support Tickets",
        "IsCustom":  true,
        "Description":  "View everything in the subscription and also open support tickets.",
        "Actions":  [
                        "*/read",
                        "Microsoft.Support/*"
                    ],
        "NotActions":  [
    
                       ],
        "DataActions":  [
    
                        ],
        "NotDataActions":  [
    
                           ],
        "AssignableScopes":  [
                                 "/subscriptions/00000000-0000-0000-0000-000000000000"
                             ]
    }
    ```
    
1. 若要建立新的自訂角色，請使用 [az role definition create](/cli/azure/role/definition#az-role-definition-create) 命令並指定 JSON 角色定義檔案。

    ```azurecli
    az role definition create --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

    新的自訂角色現在可以使用，而且可以指派給使用者、群組或服務主體 (就像內建角色一樣)。

## <a name="list-custom-roles"></a>列出自訂角色

- 若要列出所有的自訂角色，請使用 [az role definition list](/cli/azure/role/definition#az-role-definition-list) 命令搭配 `--custom-role-only` 參數。

    ```azurecli
    az role definition list --custom-role-only true
    ```
    
    ```Output
    [
      {
        "additionalProperties": {},
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ],
        "description": "View everything in the subscription and also open support tickets.",
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
        "name": "22222222-2222-2222-2222-222222222222",
        "permissions": [
          {
            "actions": [
              "*/read",
              "Microsoft.Support/*",
              "Microsoft.Resources/deployments/*",
              "Microsoft.Insights/diagnosticSettings/*/read"
            ],
            "additionalProperties": {},
            "dataActions": [],
            "notActions": [],
            "notDataActions": []
          }
        ],
        "roleName": "Reader Support Tickets",
        "roleType": "CustomRole",
        "type": "Microsoft.Authorization/roleDefinitions"
      }
    ]
    ```
    
    您也可以在 Azure 入口網站中看見自訂角色。

    ![匯入 Azure 入口網站之自訂角色的螢幕擷取畫面](./media/tutorial-custom-role-cli/custom-role-reader-support-tickets.png)

## <a name="update-a-custom-role"></a>更新自訂角色

若要更新自訂角色，請更新 JSON 檔案，然後更新自訂角色。

1. 開啟 ReaderSupportRole.json 檔案。

1. 在 `Actions` 中，新增用以建立和管理資源群組部署 `"Microsoft.Resources/deployments/*"` 的作業。 請務必在前一個作業之後包含逗號。

    更新號的 JSON 檔案看起來應該如下所示：

    ```json
    {
        "Name":  "Reader Support Tickets",
        "IsCustom":  true,
        "Description":  "View everything in the subscription and also open support tickets.",
        "Actions":  [
                        "*/read",
                        "Microsoft.Support/*",
                        "Microsoft.Resources/deployments/*"
                    ],
        "NotActions":  [
    
                       ],
        "DataActions":  [
    
                        ],
        "NotDataActions":  [
    
                           ],
        "AssignableScopes":  [
                                 "/subscriptions/00000000-0000-0000-0000-000000000000"
                             ]
    }
    ```
        
1. 若要更新自訂檔案，請使用 [az role definition update](/cli/azure/role/definition#az-role-definition-update) 命令並指定已更新的 JSON 檔案。

    ```azurecli
    az role definition update --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*",
            "Microsoft.Resources/deployments/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```
    
## <a name="delete-a-custom-role"></a>刪除自訂角色

- 使用 [az role definition delete](/cli/azure/role/definition#az-role-definition-delete) 命令，並指定要刪除自訂角色的角色名稱或角色識別碼。

    ```azurecli
    az role definition delete --name "Reader Support Tickets"
    ```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure CLI 建立自訂角色](custom-roles-cli.md)