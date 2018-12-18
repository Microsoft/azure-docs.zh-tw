---
title: 使用 RBAC 和 REST API 列出拒絕指派 - Azure | Microsoft Docs
description: 了解如何使用角色型存取控制 (RBAC) 和 REST API，來列出使用者、群組和應用程式的拒絕指派。
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 44c1d3b18bb9bdc63247379fe3f277cb6542f2da
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975577"
---
# <a name="list-deny-assignments-using-rbac-and-the-rest-api"></a>使用 RBAC 和 REST API 列出拒絕指派

目前，拒絕指派是**唯讀的**，而且只能由 Azure 設定。 即使您無法建立自己的拒絕指派，還是可以列出拒絕指派，因為它們可能會影響您的有效權限。 本文說明如何使用 RBAC 和 REST API 來列出拒絕指派。

## <a name="list-a-single-deny-assignment"></a>列出單一拒絕指派

1. 從下列要求著手：

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments/{deny-assignment-id}?api-version=2018-07-01-preview
    ```

1. 在 URI 中，將 *{scope}* 取代為您想要列出拒絕指派的範圍。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{deny-assignment-id}* 取代為您想要擷取的拒絕指派識別碼。

## <a name="list-multiple-deny-assignments"></a>列出多個拒絕指派

1. 從下列其中一個要求開始：

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview
    ```

    使用選擇性參數：

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. 在 URI 中，將 *{scope}* 取代為您想要列出拒絕指派的範圍。

    | 影響範圍 | 類型 |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | 訂用帳戶 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | 資源群組 |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | 資源 |

1. 將 *{filter}* 取代為您想要套用來篩選拒絕指派清單的條件。

    | Filter | 說明 |
    | --- | --- |
    | (無篩選條件) | 列出在指定範圍內、之上或之下的所有拒絕指派。 |
    | `$filter=atScope()` | 僅列出在指定範圍內和之上的拒絕指派。 不包含子範圍內的拒絕指派。 |
    | `$filter=denyAssignmentName%20eq%20'{deny-assignment-name}'` | 列出具有指定名稱的拒絕指派。 |

## <a name="list-deny-assignments-at-the-root-scope-"></a>列出根範圍 (/) 內的拒絕指派

1. 以[提高 Azure Active Directory 中全域管理員的存取權](elevate-access-global-admin.md)中所述的方式來提高您的存取權。

1. 使用下列要求：

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. 將 *{filter}* 取代為您想要套用來篩選拒絕指派清單的條件。 需要篩選條件。

    | Filter | 說明 |
    | --- | --- |
    | `$filter=atScope()` | 僅列出根範圍的拒絕指派。 不包含子範圍內的拒絕指派。 |
    | `$filter=denyAssignmentName%20eq%20'{deny-assignment-name}'` | 列出具有指定名稱的拒絕指派。 |

1. 移除已提高的存取權。

## <a name="next-steps"></a>後續步驟

- [了解拒絕指派](deny-assignments.md)
- [提高 Azure Active Directory 中全域管理員的存取權](elevate-access-global-admin.md)
- [Azure REST API 參考](/rest/api/azure/)
