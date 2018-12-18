---
title: 授與建立 Azure 企業版訂用帳戶的權限 | Microsoft Docs
description: 了解如何授與使用者或服務主體以程式設計方式建立 Azure 企業版訂用帳戶的權限。
services: azure-resource-manager
author: adpick
manager: adpick
editor: ''
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/05/2018
ms.author: adpick
ms.openlocfilehash: 86e457cf553c84386937c35bab1ab0fd20518bed
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368506"
---
# <a name="grant-access-to-create-azure-enterprise-subscriptions-preview"></a>授與建立 Azure 企業版訂用帳戶的權限 (預覽)

因為您是 [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) 上的 Azure 客戶，所以可以授與另一位使用者或服務主體建立由您帳戶付費的訂用帳戶。 在本文中，您將了解如何使用[角色型存取控制 (RBAC)](../active-directory/role-based-access-control-configure.md)來共用建立訂用帳戶的能力，以及稽核訂用帳戶建立的方法。 您對於要共用的帳戶必須具有「擁有者」角色。

若要建立訂用帳戶，請參閱[以程式設計方式建立 Azure 企業版訂用帳戶 (預覽)](programmatically-create-subscription.md)。

## <a name="delegate-access-to-an-enrollment-account-using-rbac"></a>使用 RBAC 來委派註冊帳戶的存取權

若要讓另一個使用者或服務主體能夠針對特定帳戶建立訂用帳戶，請[將註冊帳戶範圍的 RBAC 擁有者角色授與他們](../active-directory/role-based-access-control-manage-access-rest.md)。 下列範例會將註冊帳戶上的「擁有者」角色授與租用帳戶中 `principalId` 為 `<userObjectId>` (用於 SignUpEngineering@contoso.com) 的使用者。 若要尋找註冊帳戶識別碼與服務主體識別碼，請參閱[以程式設計方式建立 Azure 企業版訂用帳戶 (預覽)](programmatically-create-subscription.md)。

# <a name="resttabrest"></a>[REST](#tab/rest)

```json
PUT  https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentGuid>?api-version=2015-07-01

{
  "properties": {
    "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
    "principalId": "<userObjectId>"
  }
}
```
順利在註冊帳戶範圍指派「擁有者」角色之後，Azure 會以角色指派資訊來回應：

```json
{
  "properties": {
    "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
    "principalId": "<userObjectId>",
    "scope": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "createdOn": "2018-03-05T08:36:26.4014813Z",
    "updatedOn": "2018-03-05T08:36:26.4014813Z",
    "createdBy": "<assignerObjectId>",
    "updatedBy": "<assignerObjectId>"
  },
  "id": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
  "type": "Microsoft.Authorization/roleAssignments",
  "name": "<roleAssignmentGuid>"
}
```

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

使用 [New-AzureRmRoleAssignment](../active-directory/role-based-access-control-manage-access-powershell.md) 將您註冊帳戶的「擁有者」存取權授與另一個使用者。

```azurepowershell-interactive
New-AzureRmRoleAssignment -RoleDefinitionName Owner -ObjectId <userObjectId> -Scope /providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

使用 [az role assignment create](../active-directory/role-based-access-control-manage-access-azure-cli.md) 將您註冊帳戶的「擁有者」存取權授與另一個使用者。

```azurecli-interactive 
az role assignment create --role Owner --assignee-object-id <userObjectId> --scope /providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

----

在使用者成為您註冊帳戶的「RBAC 擁有者」之後，他們便能以程式設計方式在該帳戶下建立訂用帳戶。 由委派的使用者所建立的訂用帳戶仍然會以原始「帳戶擁有者」作為「服務管理員」，但預設也會以委派的使用者作為「擁有者」。 

## <a name="audit-who-created-subscriptions-using-activity-logs"></a>使用活動記錄來稽核建立訂用帳戶的人員

若要追蹤透過此 API 建立的訂用帳戶，請使用[租用戶活動記錄 API](/rest/api/monitor/tenantactivitylogs)。 目前無法使用 PowerShell、CLI 或 Azure 入口網站來追蹤訂用帳戶的建立。

1. 以 Azure AD 租用戶的租用戶管理員身分[提高存取權](../active-directory/role-based-access-control-tenant-admin-access.md)，然後將 `/providers/microsoft.insights/eventtypes/management`範圍的「讀者」角色指派給稽核使用者。
1. 以稽核使用者身分呼叫[租用戶活動記錄 API](/rest/api/monitor/tenantactivitylogs)，以查看訂用帳戶建立活動。 範例：

```
GET "/providers/Microsoft.Insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '{greaterThanTimeStamp}' and eventTimestamp le '{lessThanTimestamp}' and eventChannels eq 'Operation' and resourceProvider eq 'Microsoft.Subscription'" 
```

> [!NOTE]
> 若要以便利的方式從命令列呼叫此 API，請嘗試 [ARMClient](https://github.com/projectkudu/ARMClient)。

## <a name="next-steps"></a>後續步驟

* 使用者或服務主體在獲得建立訂用帳戶的權限後，您便可使用該身分識別[以程式設計方式建立 Azure 企業版訂用帳戶](programmatically-create-subscription.md)。
* 如需有關使用 .NET 來建立訂用帳戶的範例，請參閱 [GitHub 上的範例程式碼](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core) \(英文\)。
* 若要深入了解 Azure Resource Manager 及其 API，請參閱 [Azure Resource Manager 概觀](resource-group-overview.md)。
* 若要深入了解如何使用管理群組來管理大量訂用帳戶，請參閱[使用 Azure 管理群組來組織資源](management-groups-overview.md)
* 若要查看適用於大型組織在訂用帳戶治理上的完整最佳做法指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)
