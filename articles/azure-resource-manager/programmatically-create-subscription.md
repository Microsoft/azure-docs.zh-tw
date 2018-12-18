---
title: 以程式設計方式建立 Azure 企業版訂用帳戶 | Microsoft Docs
description: 了解如何以程式設計方式建立額外的 Azure 企業版或 Enterprise 開發/測試訂用帳戶。
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
ms.openlocfilehash: 2bfa9944d85fde65ad8dbd73ddda11fa405df2f8
ms.sourcegitcommit: 99a6a439886568c7ff65b9f73245d96a80a26d68
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39358344"
---
# <a name="programmatically-create-azure-enterprise-subscriptions-preview"></a>以程式設計方式建立 Azure 企業版訂用帳戶 (預覽)

身為 [Enterprise Agreement (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) 上的 Azure 客戶，您可以透過程式設計方式建立 EA (MS-AZR-0017P) 和 EA 開發/測試 (MS-AZR-0148P) 訂用帳戶。 在本文中，可了解如何使用 Azure Resource Manager 以程式設計方式建立訂用帳戶。

當您以此 API 建立 Azure 訂用帳戶時，該訂用帳戶會受合約規範，此合約是您向 Microsoft 或授權經銷商取得 Microsoft Azure 服務時所簽署的合約。 若要深入了解，請參閱 [Microsoft Azure 法律資訊](https://azure.microsoft.com/support/legal/)。

## <a name="prerequisites"></a>必要條件

針對您想要建立訂用帳戶的註冊帳戶，您必須要有擁有者或參與者角色。 有兩種方式可取得這些角色：

* 您的註冊系統管理員可以[讓您成為帳戶擁有者](https://ea.azure.com/helpdocs/addNewAccount) \(英文\) (需登入)，這可讓您成為註冊帳戶的擁有者。 請依照您收到的邀請電子郵件中的指示來手動建立初始訂用帳戶。 請先確認帳戶擁有權，並手動建立初始 EA 訂用帳戶，才能繼續進行下一步。 不能光只是將該帳戶新增至註冊。

* 註冊帳戶的現有擁有者可以[授與您存取權](grant-access-to-create-subscription.md)。 同樣地，如果您想要使用服務主體來建立 EA 訂用帳戶，您必須[授與該服務主體建立訂用帳戶的能力](grant-access-to-create-subscription.md)。

## <a name="find-accounts-you-have-access-to"></a>尋找您可以存取的帳戶

在您被新增至 Azure EA 註冊作為「帳戶擁有者」之後，Azure 會使用帳戶與註冊的關聯性來判斷由誰支付訂用帳戶的費用。 所有在該帳戶下所建立訂用帳戶的費用，都會由該帳戶所在的 EA 註冊支付。 若要建立訂用帳戶，您必須傳入註冊帳戶與使用者主體的相關值，才能擁有訂用帳戶。 

若要執行下列命令，您必須登入「帳戶擁有者」的*主目錄* (建立訂用帳戶時預設的所在目錄)。

# <a name="resttabrest"></a>[REST](#tab/rest)

要求列出所有註冊帳戶：

```json
GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts?api-version=2018-03-01-preview
```

Azure 會以您可以存取的所有註冊帳戶清單來回應：

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "SignUpEngineering@contoso.com"
      }
    },
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "BillingPlatformTeam@contoso.com"
      }
    }
  ]
}
```

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

使用 [Get-AzureRmEnrollmentAccount 命令](/powershell/module/azurerm.billing/get-azurermenrollmentaccount)列出您可以存取的所有註冊帳戶。

```azurepowershell-interactive
Get-AzureRmEnrollmentAccount
```

Azure 會以帳戶的「物件識別碼」和電子郵件清單來回應。

```azurepowershell
ObjectId                               | PrincipalName
747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

使用 [az billing enrollment-account list](https://aka.ms/EASubCreationPublicPreviewCLI) 命令來列出您可以存取的所有註冊帳戶。

```azurecli-interactive 
az billing enrollment-account list
```

Azure 會以帳戶的「物件識別碼」和電子郵件清單來回應。

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "SignUpEngineering@contoso.com"
      }
    },
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "BillingPlatformTeam@contoso.com"
      }
    }
  ]
}
```

---

使用 `principalName` 屬性來識別要為訂用帳戶付費的帳戶。 使用 `id` 作為您在下一個步驟中用來建立訂用帳戶的 `enrollmentAccount` 值。

## <a name="create-subscriptions-under-a-specific-enrollment-account"></a>在特定的註冊帳戶下建立訂用帳戶 

下列範例會建立一個要求來建立名為 *Dev Team Subscription* 的訂用帳戶，而訂用帳戶供應項目方案則為 *MS-AZR-0017P* (一般 EA)。 註冊帳戶為 `747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (預留位置值，此值為 GUID)，這是 SignUpEngineering@contoso.com 的註冊帳戶。 這也會依選擇新增兩個使用者作為該訂用帳戶的「RBAC 擁有者」。

# <a name="resttabrest"></a>[REST](#tab/rest)

在要求的路徑中使用 `enrollmentAccount` 的 `id` 來建立訂用帳戶。

```json
POST https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Subscription/createSubscription?api-version=2018-03-01-preview

{
  "displayName": "Dev Team Subscription",
  "offerType": "MS-AZR-0017P",
  "owners": [
    {
      "objectId": "<userObjectId>"
    },
    {
      "objectId": "<servicePrincipalObjectId>"
    }
  ]
}
```

| 元素名稱  | 必要 | 類型   | 說明                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | 否      | 字串 | 訂用帳戶的顯示名稱。 如果未指定，會設定為供應項目的名稱，例如「Microsoft Azure 企業版」。                                 |
| `offerType`   | 是      | 字串 | 訂用帳戶的供應項目方案。 EA 的兩個選項為 [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (生產環境使用) 和 [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (開發/測試環境使用，必須[使用 EA 入口網站來開啟](https://ea.azure.com/helpdocs/DevOrTestOffer))。                |
| `owners`      | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之使用者的「物件識別碼」。  |

在回應中，會傳回一個 `subscriptionOperation` 物件供您監視。 建立完訂用帳戶之後，`subscriptionOperation` 物件會傳回 `subscriptionLink` 物件，其中包含訂用帳戶 ID。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

若要使用此預覽板模組，請先執行 `Install-Module AzureRM.Subscription -AllowPrerelease` 來安裝它。 若要確認 `-AllowPrerelease` 是否可運作，請從[取得 PowerShellGet 模組](/powershell/gallery/installing-psget)安裝最新版的 PowerShellGet。

使用 [New-AzureRmSubscription](/powershell/module/azurerm.subscription) 搭配以 `enrollmentAccount` 物件識別碼作為 `EnrollmentAccountObjectId` 參數來建立新的訂用帳戶。 

```azurepowershell-interactive
New-AzureRmSubscription -OfferType MS-AZR-0017P -Name "Dev Team Subscription" -EnrollmentAccountObjectId 747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx -OwnerObjectId <userObjectId>,<servicePrincipalObjectId>
```

| 元素名稱  | 必要 | 類型   | 說明                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `Name` | 否      | 字串 | 訂用帳戶的顯示名稱。 如果未指定，會設定為供應項目的名稱，例如「Microsoft Azure 企業版」。                                 |
| `OfferType`   | 是      | 字串 | 訂用帳戶的供應項目方案。 EA 的兩個選項為 [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (生產環境使用) 和 [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (開發/測試環境使用，必須[使用 EA 入口網站來開啟](https://ea.azure.com/helpdocs/DevOrTestOffer))。                |
| `EnrollmentAccountObjectId`      | 是       | 字串 | 用來建立訂用帳戶並加以收費的註冊帳戶物件識別碼。 此值是您從 `Get-AzureRmEnrollmentAccount` 取得的 GUID。 |
| `OwnerObjectId`      | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之使用者的「物件識別碼」。  |
| `OwnerSignInName`    | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之使用者的電子郵件地址。 您可以使用此參數來取代 `OwnerObjectId`。|
| `OwnerApplicationId` | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之服務主體的「應用程式識別碼」。 您可以使用此參數來取代 `OwnerObjectId`。| 

若要查看所有參數的完整清單，請參閱 [New-AzureRmSubscription](/powershell/module/azurerm.subscription.preview)。

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

若要使用此預覽板延伸模組，請先執行 `az extension add --name subscription` 來安裝它。

使用 [az account create](/cli/azure/ext/subscription/account?view=azure-cli-latest#-ext-subscription-az-account-create) 搭配以 `enrollmentAccount` 物件識別碼作為 `enrollment-account-object-id` 參數來建立新的訂用帳戶。

```azurecli-interactive 
az account create --offer-type "MS-AZR-0017P" --display-name "Dev Team Subscription" --enrollment-account-object-id "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx" --owner-object-id "<userObjectId>","<servicePrincipalObjectId>"
```

| 元素名稱  | 必要 | 類型   | 說明                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `display-name` | 否      | 字串 | 訂用帳戶的顯示名稱。 如果未指定，會設定為供應項目的名稱，例如「Microsoft Azure 企業版」。                                 |
| `offer-type`   | 是      | 字串 | 訂用帳戶的供應項目方案。 EA 的兩個選項為 [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (生產環境使用) 和 [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (開發/測試環境使用，必須[使用 EA 入口網站來開啟](https://ea.azure.com/helpdocs/DevOrTestOffer))。                |
| `enrollment-account-object-id`      | 是       | 字串 | 用來建立訂用帳戶並加以收費的註冊帳戶物件識別碼。 此值是您從 `az billing enrollment-account list` 取得的 GUID。 |
| `owner-object-id`      | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之使用者的「物件識別碼」。  |
| `owner-upn`    | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之使用者的電子郵件地址。 您可以使用此參數來取代 `owner-object-id`。|
| `owner-spn` | 否       | 字串 | 在建立訂用帳戶之後，任何您想要在該訂用帳戶上新增為「RBAC 擁有者」之服務主體的「應用程式識別碼」。 您可以使用此參數來取代 `owner-object-id`。| 

若要查看所有參數的完整清單，請參閱 [az account create](/cli/azure/ext/subscription/account?view=azure-cli-latest#-ext-subscription-az-account-create)。

----

## <a name="limitations-of-azure-enterprise-subscription-creation-api"></a>Azure 企業版訂用帳戶建立 API 的限制

- 使用此 API 只能建立「Azure 企業版」訂用帳戶。
- 每一帳戶僅限 50 個訂用帳戶。 之後便只能使用「帳戶中心」來建立訂用帳戶。
- 帳戶下必須至少有一個 EA 或 EA 開發/測試訂用帳戶，也就是說，「帳戶擁有者」已至少手動註冊過一次。
- 不是「帳戶擁有者」但已透過 RBAC 新增至註冊帳戶的使用者並無法使用「帳戶中心」來建立訂用帳戶。
- 您無法選取要作為訂用帳戶建立位置的租用戶。 訂用帳戶一律會建立在「帳戶擁有者」的主租用戶中。 若要將訂用帳戶移至不同的租用戶，請參閱[變更訂用帳戶租用戶](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)。

## <a name="next-steps"></a>後續步驟

* 如需有關使用 .NET 來建立訂用帳戶的範例，請參閱 [GitHub 上的範例程式碼](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core) \(英文\)。
* 由於您已建立訂用帳戶，您可以將這項功能授與其他使用者和服務主體。 如需詳細資訊，請參閱 [ Azure Enterprise 訂用帳戶 (預覽)](grant-access-to-create-subscription.md)。
* 若要深入了解如何使用管理群組來管理大量訂用帳戶，請參閱[使用 Azure 管理群組來組織資源](management-groups-overview.md)
