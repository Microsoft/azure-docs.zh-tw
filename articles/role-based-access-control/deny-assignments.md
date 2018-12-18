---
title: 了解 Azure RBAC 中的拒絕指派 | Microsoft Docs
description: 了解 Azure 資源之角色型存取控制 (RBAC) 中的拒絕指派。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: ''
ms.openlocfilehash: 8ef3a2ec44c5eff80d3a50a6c56805667e164ba8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980163"
---
# <a name="understand-deny-assignments"></a>了解拒絕指派

與角色指派相同，「拒絕指派」也會針對拒絕存取權繫結一組在特定範圍拒絕使用者、群組或服務主體的動作。 拒絕指派也可以排除主體並防止繼承到子範圍，這與角色指派不同。 目前，拒絕指派是**唯讀的**，而且只能由 Azure 設定。 此文章說明如何定義拒絕指派。

## <a name="deny-assignment-properties"></a>拒絕指派屬性

 拒絕指派有下列屬性：

> [!div class="mx-tableFixed"]
> | 屬性 | 必要 | 類型 | 說明 |
> | --- | --- | --- | --- |
> | `DenyAssignmentName` | 是 | 字串 | 拒絕指派的顯示名稱。 名稱在指定範圍內必須是唯一的。 |
> | `Description` | 否 | 字串 | 拒絕指派的描述。 |
> | `Permissions.Actions` | 至少一個 Actions 或一個 DataActions | String[] | 一個字串陣列，指定拒絕指派要封鎖存取權的管理作業。 |
> | `Permissions.NotActions` | 否 | String[] | 一個字串陣列，指定要從拒絕指派排除的管理作業。 |
> | `Permissions.DataActions` | 至少一個 Actions 或一個 DataActions | String[] | 一個字串陣列，指定拒絕指派要封鎖存取權的資料作業。 |
> | `Permissions.NotDataActions` | 否 | String[] | 一個字串陣列，指定要從拒絕指派排除的資料作業。 |
> | `Scope` | 否 | 字串 | 一個字串， 指定拒絕指派要套用的範圍。 |
> | `DoNotApplyToChildScopes` | 否 | 布林值 | 指定拒絕指派是否要套用到子範圍。 預設值為 false。 |
> | `Principals[i].Id` | 是 | String[] | 要套用拒絕指派的 Azure AD 主體物件識別碼 (使用者、群組或服務主體) 陣列。 設定為空 GUID `00000000-0000-0000-0000-000000000000` 以代表每個人。 |
> | `Principals[i].Type` | 否 | String[] | 由 Principals[i].Id 代表的物件類型陣列。設定為 `Everyone` 以代表每個人。 |
> | `ExcludePrincipals[i].Id` | 否 | String[] | 不要套用拒絕指派的 Azure AD 主體物件識別碼 (使用者、群組或服務主體) 陣列。 |
> | `ExcludePrincipals[i].Type` | 否 | String[] | 由 ExcludePrincipals[i].Id 代表的物件類型陣列。 |
> | `IsSystemProtected` | 否 | 布林值 | 指定此拒絕指派是否由 Azure 建立且無法編輯或刪除。 目前，所有拒絕指派都受系統保護。 |

## <a name="everyone-principal"></a>每個人主體

為支援拒絕指派，我們引進每個人主體。 每個人主體代表 Azure AD 目錄中的所有使用者、群組與服務主體。 若主體識別碼是零值 GUID `00000000-0000-0000-0000-000000000000` 且主體類型是 `Everyone`，則主體代表每個人。 每個人主體可與 `ExcludePrincipals` 合併以拒絕除了某些使用者之外的每個人。 每個人主體具有下列條件約束：

- 只能在 `Principals` 中使用，而無法在 `ExcludePrincipals` 中使用。
- `Principals[i].Type` 必須設為 `Everyone`。

## <a name="next-steps"></a>後續步驟

* [使用 RBAC 與 REST API 列出拒絕指派](deny-assignments-rest.md)
* [了解角色定義](role-definitions.md)
