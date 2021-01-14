---
title: 將資源部署到租用戶
description: 描述如何在 Azure Resource Manager 範本中的租用戶範圍部署資源。
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: 0b3ddc63e49b272c93349ada91e9a1599ea4be4f
ms.sourcegitcommit: 0aec60c088f1dcb0f89eaad5faf5f2c815e53bf8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98186204"
---
# <a name="tenant-deployments-with-arm-templates"></a>使用 ARM 範本的租使用者部署

隨著您的組織逐漸成熟，您可能需要在 Azure AD 租使用者中，定義並指派 [原則](../../governance/policy/overview.md) 或 [azure 角色型存取控制 (azure RBAC) ](../../role-based-access-control/overview.md) 。 您可以使用租用戶層級範本，以宣告方式套用原則，並在全域層級指派角色。

## <a name="supported-resources"></a>支援的資源

並非所有的資源類型都可部署至租使用者層級。 本節將列出支援的資源類型。

針對 Azure 原則，請使用：

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)

若要 (azure RBAC) 的 Azure 角色型存取控制，請使用：

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)

針對部署至管理群組、訂用帳戶或資源群組的嵌套範本，請使用：

* [部署](/azure/templates/microsoft.resources/deployments)

若要建立管理群組，請使用：

* [managementGroups](/azure/templates/microsoft.management/managementgroups)

若要建立訂閱，請使用：

* [別名](/azure/templates/microsoft.subscription/aliases)

若要管理成本，請使用：

* [billingProfiles](/azure/templates/microsoft.billing/billingaccounts/billingprofiles)
* [指示](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/instructions)
* [invoiceSections](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/invoicesections)

若要設定入口網站，請使用：

* [tenantConfigurations](/azure/templates/microsoft.portal/tenantconfigurations)

## <a name="schema"></a>結構描述

您用於租用戶部署的結構描述與用於資源群組部署的結構描述不同。

針對範本，請使用：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    ...
}
```

所有部署範圍的參數檔案結構描述都相同。 針對參數檔案，請使用：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="required-access"></a>必要的存取權

部署範本的主體必須具備在租用戶範圍建立資源的權限。 主體必須擁有執行部署動作 (`Microsoft.Resources/deployments/*`) 並建立在範本中定義之資源的權限。 例如，若要建立管理群組，主體必須具備租用戶範圍的「參與者」權限。 若要建立角色指派，主體必須具備「擁有者」權限。

Azure Active Directory 的全域管理員不會自動具備指派角色的權限。 若要在租用戶範圍啟用範本部署，全域管理員必須執行下列步驟：

1. 提升帳戶存取權，讓全域管理員可以指派角色。 如需詳細資訊，請參閱[提高存取權以管理所有 Azure 訂用帳戶和管理群組](../../role-based-access-control/elevate-access-global-admin.md)。

1. 將「擁有者」或「參與者」指派給需要部署範本的主體。

   ```azurepowershell-interactive
   New-AzRoleAssignment -SignInName "[userId]" -Scope "/" -RoleDefinitionName "Owner"
   ```

   ```azurecli-interactive
   az role assignment create --assignee "[userId]" --scope "/" --role "Owner"
   ```

主體現在具備部署範本所需的權限。

## <a name="deployment-commands"></a>部署命令

用於租用戶部署的命令與用於資源群組部署的命令不同。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

針對 Azure CLI，使用 [az deployment tenant create](/cli/azure/deployment/tenant#az-deployment-tenant-create)：

```azurecli-interactive
az deployment tenant create \
  --name demoTenantDeployment \
  --location WestUS \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

針對 Azure PowerShell，使用 [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment)。

```azurepowershell-interactive
New-AzTenantDeployment `
  -Name demoTenantDeployment `
  -Location "West US" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

---

如需部署 ARM 範本的部署命令和選項的詳細資訊，請參閱：

* [使用 ARM 範本和 Azure 入口網站部署資源](deploy-portal.md)
* [使用 ARM 範本與 Azure CLI 來部署資源](deploy-cli.md)
* [使用 ARM 範本與 Azure PowerShell 來部署資源](deploy-powershell.md)
* [使用 ARM 範本部署資源，並 Azure Resource Manager REST API](deploy-rest.md)
* [使用部署按鈕從 GitHub 存放庫部署範本](deploy-to-azure-button.md)
* [從 Cloud Shell 部署 ARM 範本](deploy-cloud-shell.md)

## <a name="deployment-location-and-name"></a>部署位置和名稱

針對租用戶層級部署，您必須提供部署的位置。 部署的位置與您部署的資源位置不同。 部署位置會指定部署資料的儲存位置。 [訂](deploy-to-subscription.md) 用帳戶和 [管理群組](deploy-to-management-group.md) 部署也需要一個位置。 針對 [資源群組](deploy-to-resource-group.md) 部署，資源群組的位置會用來儲存部署資料。

您可以提供部署的名稱，或使用預設的部署名稱。 預設名稱是範本檔案的名稱。 例如，部署名為 **azuredeploy.json** 的範本會建立預設的部署名稱 **azuredeploy**。

對於每個部署名稱而言，此位置是不可變的。 當某個位置已經有名稱相同的現有部署時，您無法在其他位置建立部署。 例如，如果您在 **centralus** 中建立名為 **deployment1** 的租使用者部署，稍後就無法使用名稱 **deployment1** 建立另一個部署，而是 **westus** 的位置。 如果您收到錯誤代碼 `InvalidDeploymentLocation`，請使用不同的名稱或與先前該名稱部署相同的位置。

## <a name="deployment-scopes"></a>部署範圍

部署至租使用者時，您可以將資源部署至：

* 租使用者
* 租使用者中的管理群組
* subscriptions
* 資源群組

[延伸模組資源](scope-extension-resources.md)的範圍可以設定為與部署目標不同的目標。

部署範本的使用者必須擁有指定範圍的存取權。

本節說明如何指定不同的範圍。 您可以將這些不同的範圍結合在單一範本中。

### <a name="scope-to-tenant"></a>租使用者範圍

在範本的資源區段中定義的資源會套用至租使用者。

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-tenant.json" highlight="5":::

### <a name="scope-to-management-group"></a>管理群組的範圍

若要鎖定租使用者內的管理群組，請新增嵌套部署並指定 `scope` 屬性。

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>訂用帳戶的範圍

您也可以將租使用者中的訂用帳戶設為目標。 部署範本的使用者必須擁有指定範圍的存取權。

若要以租使用者內的訂用帳戶為目標，請使用嵌套部署和 `subscriptionId` 屬性。

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>將範圍設為資源群組

您也可以將租使用者中的資源群組設為目標。 部署範本的使用者必須擁有指定範圍的存取權。

若要以租使用者內的資源群組為目標，請使用嵌套部署。 請設定 `subscriptionId` 和 `resourceGroup` 屬性。 請勿設定嵌套部署的位置，因為它是部署在資源群組的位置。

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-rg.json" highlight="9,10,18":::

## <a name="create-management-group"></a>建立管理群組

下列範本會建立管理群組。

:::code language="json" source="~/quickstart-templates/tenant-deployments/new-mg/azuredeploy.json":::

如果您的帳戶沒有部署至租使用者的許可權，您仍然可以藉由部署至另一個範圍來建立管理群組。 如需詳細資訊，請參閱 [管理群組](deploy-to-management-group.md#management-group)。

## <a name="assign-role"></a>指派角色

下列範本會在租用戶範圍指派角色。

:::code language="json" source="~/quickstart-templates/tenant-deployments/tenant-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>後續步驟

* 若要瞭解如何指派角色，請參閱 [使用 Azure Resource Manager 範本新增 Azure 角色指派](../../role-based-access-control/role-assignments-template.md)。
* 您也可以在[訂用帳戶層級](deploy-to-subscription.md)或[管理群組層級](deploy-to-management-group.md)部署範本。
