---
title: 使用適用於 .NET 的用戶端程式庫管理 Batch 帳戶資源 - Azure | Microsoft Docs
description: 使用 Batch Management .NET 程式庫建立、刪除和修改 Azure Batch 帳戶資源。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 16279b23-60ff-4b16-b308-5de000e4c028
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/24/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 65f8eb0752a181eda312515e557bb733c091e2e5
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505375"
---
# <a name="manage-batch-accounts-and-quotas-with-the-batch-management-client-library-for-net"></a>使用適用於 .NET 的 Batch 管理用戶端程式庫來管理 Batch 帳戶和配額

> [!div class="op_single_selector"]
> * [Azure 入口網站](batch-account-create-portal.md)
> * [Batch Management .NET](batch-management-dotnet.md)
> 
> 

您可以透過使用 [Batch Management .NET][api_mgmt_net] 程式庫來自動化 Batch 帳戶的建立、刪除、金鑰管理和配額探索，可降低 Azure Batch 應用程式中的維護負擔。

* **建立和刪除 Batch 帳戶** 。 比方說，身為獨立軟體廠商 (ISV)，您為每個獲派不同 Batch 帳戶的客戶針對計費目的提供服務，您可以將帳戶建立和刪除功能加入至客戶入口網站。
* **擷取和重新產生帳戶金鑰** 。 這可協助您符合強制定期變換帳戶金鑰或讓帳戶金鑰過期的安全性原則。 當您在各種不同的 Azure 區域中有數個 Batch 帳戶時，將此變換程序自動化可增加解決方案的效率。
* **檢查帳戶配額** 並採取反復試驗的猜測，判斷哪一個 Batch 帳戶有哪些限制。 藉由在開始作業、建立集區或新增計算節點之前檢查您的帳戶配額，您可以主動地調整建立計算資源的位置或時機。 您可以決定在帳戶中配置其他資源之前，哪些帳戶需要增加配額。
* 藉由在同一個應用程式中使用 Batch Management .NET、[Azure Active Directory][aad_about] 和 [Azure Resource Manager][resman_overview]，**結合其他 Azure 服務的功能**可獲得完整功能的管理體驗。 透過使用這些功能和其 API，您可以提供順暢的驗證體驗、建立和刪除資源群組的能力，以及上面所述的功能，以獲得端對端管理解決方案。

> [!NOTE]
> 雖然這篇文章著重在以程式設計方式管理 Batch 帳戶、金鑰和配額，您可以使用 [Azure 入口網站][azure_portal]執行許多活動。 如需詳細資訊，請參閱[使用 Azure 入口網站建立 Azure Batch 帳戶](batch-account-create-portal.md)和 [Azure Batch 服務的配額和限制](batch-quota-limit.md)。
> 
> 

## <a name="create-and-delete-batch-accounts"></a>建立和刪除 Batch 帳戶
如上所述，Batch Management API 的主要功能之一就是在 Azure 區域內建立和刪除 Batch 帳戶。 若要這樣做，您將使用 [BatchManagementClient.Account.CreateAsync][net_create] 和 [DeleteAsync][net_delete]，或其同步對應項目。

下列程式碼片段會建立一個帳戶、從 Batch 服務取得新建立的帳戶，然後將它刪除。 在本文的此程式碼片段與其他程式碼片段中，`batchManagementClient` 是 [BatchManagementClient][net_mgmt_client] 完整初始化的執行個體。

```csharp
// Create a new Batch account
await batchManagementClient.Account.CreateAsync("MyResourceGroup",
    "mynewaccount",
    new BatchAccountCreateParameters() { Location = "West US" });

// Get the new account from the Batch service
AccountResource account = await batchManagementClient.Account.GetAsync(
    "MyResourceGroup",
    "mynewaccount");

// Delete the account
await batchManagementClient.Account.DeleteAsync("MyResourceGroup", account.Name);
```

> [!NOTE]
> 使用 Batch Management .NET 程式庫和其 BatchManagementClient 類別的應用程式需要**服務管理員**或**共同管理員**存取權，以使用擁有要管理的 Batch 帳戶的訂用帳戶。 如需詳細資訊，請參閱以 [Azure Active Directory](#azure-active-directory) 一節和 [AccountManagement][acct_mgmt_sample] 程式碼範例。
> 
> 

## <a name="retrieve-and-regenerate-account-keys"></a>擷取和重新產生帳戶金鑰
使用 [ListKeysAsync][net_list_keys] 從您的訂用帳戶內的任何 Batch 帳戶取得主要和次要帳戶金鑰。 您可以使用 [RegenerateKeyAsync][net_regenerate_keys] 重新產生這些金鑰。

```csharp
// Get and print the primary and secondary keys
BatchAccountListKeyResult accountKeys =
    await batchManagementClient.Account.ListKeysAsync(
        "MyResourceGroup",
        "mybatchaccount");
Console.WriteLine("Primary key:   {0}", accountKeys.Primary);
Console.WriteLine("Secondary key: {0}", accountKeys.Secondary);

// Regenerate the primary key
BatchAccountRegenerateKeyResponse newKeys =
    await batchManagementClient.Account.RegenerateKeyAsync(
        "MyResourceGroup",
        "mybatchaccount",
        new BatchAccountRegenerateKeyParameters() {
            KeyName = AccountKeyType.Primary
            });
```

> [!TIP]
> 您可以為您的管理應用程式建立簡化的連線工作流程。 首先，取得您想要使用 [ListKeysAsync][net_list_keys] 管理的 Batch 帳戶的帳戶金鑰。 接著，在初始化 Batch .NET 程式庫的 [BatchSharedKeyCredentials][net_sharedkeycred] 類別 (初始化時 [BatchClient][net_batch_client] 時使用) 時使用此金鑰。
> 
> 

## <a name="check-azure-subscription-and-batch-account-quotas"></a>檢查 Azure 訂用帳戶和 Batch 帳戶配額
Azure 訂用帳戶和 Batch 之類的個別 Azure 服務均具有預設配額，限制其中特定實體的數目。 如需 Azure 訂用帳戶的預設配額，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md)。 如需 Batch 服務的預設配額，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md)。 藉由使用 Batch Management .NET 程式庫，您可以檢查您的應用程式中的這些配額。 這可讓您在加入帳戶或計算資源 (如集區和計算節點) 之前進行配置決策。

### <a name="check-an-azure-subscription-for-batch-account-quotas"></a>檢查 Azure 訂用帳戶和 Batch 帳戶配額
在區域中建立 Batch 帳戶之前，您可以檢查您的 Azure 訂用帳戶，以查看您是否能夠將帳戶加入該區域中。

在以下的程式碼片段中，我們先使用 [BatchManagementClient.Account.ListAsync][net_mgmt_listaccounts] 來取得訂用帳戶內所有 Batch 帳戶的集合。 在我們取得此集合後，我們會判斷目標區域中有多少個帳戶。 接著，我們使用 [BatchManagementClient.Subscriptions][net_mgmt_subscriptions] 來取得 Batch 帳戶配額，並判斷該區域中可以建立多少個帳戶 (如果有的話)。

```csharp
// Get a collection of all Batch accounts within the subscription
BatchAccountListResponse listResponse =
        await batchManagementClient.Account.ListAsync(new AccountListParameters());
IList<AccountResource> accounts = listResponse.Accounts;
Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}",
    creds.SubscriptionId,
    accounts.Count);

// Get a count of all accounts within the target region
string region = "westus";
int accountsInRegion = accounts.Count(o => o.Location == region);

// Get the account quota for the specified region
SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Subscriptions.GetSubscriptionQuotasAsync(region);
Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

// Determine how many accounts can be created in the target region
Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);
```

在上述程式碼片段中，`creds` 是 [TokenCloudCredentials][azure_tokencreds] 的執行個體。 若要查看建立此物件的範例，請參閱 GitHub 上的 [AccountManagement][acct_mgmt_sample] 程式碼範例。

### <a name="check-a-batch-account-for-compute-resource-quotas"></a>檢查 Batch 帳戶的計算資源配額
在 Batch 方案中增加計算資源之前，您可以檢查以確定您要配置的資源不會超過帳戶的配額。 在下列程式碼片段中，我們會列印名為 `mybatchaccount` 的 Batch 帳戶配額資訊。 在您自己的應用程式中，您可以使用這類資訊來判斷帳戶是否可以處理要建立的其他資源。

```csharp
// First obtain the Batch account
BatchAccountGetResponse getResponse =
    await batchManagementClient.Account.GetAsync("MyResourceGroup", "mybatchaccount");
AccountResource account = getResponse.Resource;

// Now print the compute resource quotas for the account
Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);
```

> [!IMPORTANT]
> 雖然 Azure 訂用帳戶和服務有預設配額，這許多限制都可以透過在 [Azure 入口網站][azure_portal]中提出要求來提高。 例如，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md) 以取得增加您的 Batch 帳戶配額的指示。
> 
> 

## <a name="use-azure-ad-with-batch-management-net"></a>搭配 Batch Management .NET 使用 Azure AD

Batch Management .NET 程式庫是 Azure 資源提供者用戶端，並與 [Azure Resource Manager][resman_overview] 搭配使用，以程式設計方式管理帳戶資源。 驗證透過任何 Azure 資源提供者用戶端 (包括 Batch Management .NET 程式庫)，以及透過 [Azure Resource Manager][resman_overview] 所提出的要求時，都需要 Azure AD。 如需搭配 Batch Management .NET 程式庫使用 Azure AD 的詳細資訊，請參閱[使用 Azure Active Directory 驗證 Batch 方案](batch-aad-auth.md)。 

## <a name="sample-project-on-github"></a>GitHub 上的範例專案

若要查看 Batch Management .NET 的實際運作，請查看 GitHub 上的 [AccountManagment][acct_mgmt_sample] 範例專案。 AccountManagment 範例應用程式會示範下列作業：

1. 使用 [ADAL][aad_adal] 向 Azure AD 取得安全性權杖。 如果使用者尚未登入，系統會提示使用者輸入其 Azure 認證。
2. 使用從 Azure AD 取得的安全性權杖，建立 [SubscriptionClient][resman_subclient] 以查詢 Azure 來取得與帳戶相關聯的訂用帳戶清單。 如果清單包含多個訂用帳戶，使用者可以從清單中選取一個訂用帳戶。
3. 取得與所選訂用帳戶相關聯的認證。
4. 使用認證來建立 [ResourceManagementClient][resman_client] 物件。
5. 使用 [ResourceManagementClient][resman_client] 物件來建立資源群組。
6. 使用 [BatchManagementClient][net_mgmt_client] 物件來執行數個 Batch 帳戶作業：
   * 在新的資源群組中建立 Batch 帳戶。
   * 從 Batch 服務取得新建立的帳戶。
   * 列印新帳戶的帳戶金鑰。
   * 重新產生帳戶的新主要金鑰。
   * 列印帳戶的配額資訊。
   * 列印訂用帳戶的配額資訊。
   * 列印訂用帳戶內的所有帳戶。
   * 刪除新建立的帳戶。
7. 刪除資源群組。

刪除新建立的 Batch 帳戶和資源群組之前，您可以在 [Azure 入口網站][azure_portal]中檢視它們：

若要成功執行範例應用程式，必須先向 Azure 入口網站中的 Azure AD 租用戶註冊範例應用程式，並將權限授與 Azure Resource Manager API。 請依照[使用 Active Directory 驗證 Batch Management 解決方案](batch-aad-auth-management.md)中所提供的步驟執行。


[aad_about]:../active-directory/fundamentals/active-directory-whatis.md "什麼是 Azure Active Directory？"
[aad_adal]: ../active-directory/active-directory-authentication-libraries.md
[aad_auth_scenarios]:../active-directory/develop/authentication-scenarios.md "Azure AD 的驗證案例"
[aad_integrate]:../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md "整合應用程式與 Azure Active Directory"
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_mgmt_net]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[azure_portal]: http://portal.azure.com
[azure_storage]: https://azure.microsoft.com/services/storage/
[azure_tokencreds]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.tokencloudcredentials.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_list_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.listkeysasync.aspx
[net_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.createasync.aspx
[net_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.deleteasync.aspx
[net_regenerate_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.regeneratekeyasync.aspx
[net_sharedkeycred]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.auth.batchsharedkeycredentials.aspx
[net_mgmt_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.aspx
[net_mgmt_subscriptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.subscriptions.aspx
[net_mgmt_listaccounts]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.iaccountoperations.listasync.aspx
[resman_api]: https://msdn.microsoft.com/library/azure/mt418626.aspx
[resman_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx
[resman_subclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.subscriptions.subscriptionclient.aspx
[resman_overview]: ../azure-resource-manager/resource-group-overview.md

[1]: ./media/batch-management-dotnet/portal-01.png
[2]: ./media/batch-management-dotnet/portal-02.png
[3]: ./media/batch-management-dotnet/portal-03.png
