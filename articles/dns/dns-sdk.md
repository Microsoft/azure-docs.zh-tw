---
title: 使用 .NET SDK 在 Azure DNS 中建立 DNS 區域和記錄集 | Microsoft Docs
description: 如何使用 .NET SDK 在 Azure DNS 中建立 DNS 區域和記錄集。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.assetid: eed99b87-f4d4-4fbf-a926-263f7e30b884
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2016
ms.author: victorh
ms.openlocfilehash: 14860ae48e520f86ce9d5bea739605d1a4baf0c7
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173188"
---
# <a name="create-dns-zones-and-record-sets-using-the-net-sdk"></a>使用 .NET SDK 建立 DNS 區域和記錄集

您可以使用 DNS SDK 搭配 .NET DNS 管理程式庫，以自動化作業來建立、刪除或更新 DNS 區域、記錄集和記錄。 [這裡](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True)提供完整的 Visual Studio 專案。

## <a name="create-a-service-principal-account"></a>建立服務主體帳戶

一般而言，若想要獲得以程式設計方式存取 Azure 資源的權限，就必須透過專用帳戶而非您自己的使用者認證。 這些專用帳戶稱為「服務主體」帳戶。 若要使用 Azure DNS SDK 範例專案，您首先需要建立服務主體帳戶，並為其指派正確的權限。

1. 遵循 [這些指示](../azure-resource-manager/resource-group-authenticate-service-principal.md) 建立服務主體帳戶 (Azure DNS SDK 範例專案會採用密碼型驗證)。
2. 建立資源群組 ([方法在此](../azure-resource-manager/resource-group-template-deploy-portal.md))。
3. 使用 Azure RBAC 將資源群組的「DNS 區域參與者」權限授與服務主體帳戶 ([方法在此](../role-based-access-control/role-assignments-portal.md))。
4. 如果使用 Azure DNS SDK 範例專案，請如下編輯 'program.cs' 檔案︰

   * 針對 tenantId、clientId (也稱為帳戶識別碼)、secret (服務主體帳戶密碼) 和 subscriptionId 插入步驟 1 所使用的正確值。
   * 輸入在步驟 2 中選擇的資源群組名稱。
   * 輸入您選擇的 DNS 區域名稱。

## <a name="nuget-packages-and-namespace-declarations"></a>NuGet 封裝和命名空間宣告

若要使用 Azure DNS .NET SDK，您需要安裝 **Azure DNS 管理程式庫** NuGet 封裝以及其他必要的 Azure 封裝。

1. 在 **Visual Studio**中，開啟專案或新專案。
2. 移至 [工具]**>**[NuGet 封裝管理員]**>**[管理解決方案的 NuGet 封裝...]。
3. 按一下 [瀏覽]，啟用 [包括發行前版本] 核取方塊，然後在搜尋方塊中輸入 **Microsoft.Azure.Management.Dns**。
4. 選取封裝，然後按一下 [安裝]  將它加入至您的 Visual Studio 專案。
5. 重複上述程序以便一併安裝下列封裝︰**Microsoft.Rest.ClientRuntime.Azure.Authentication** 和 **Microsoft.Azure.Management.ResourceManager**。

## <a name="add-namespace-declarations"></a>新增命名空間宣告

新增下列命名空間宣告

```cs
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.Dns;
using Microsoft.Azure.Management.Dns.Models;
```

## <a name="initialize-the-dns-management-client"></a>初始化 DNS 管理用戶端

*DnsManagementClient* 包含管理 DNS 區域和記錄集所需的方法和屬性。  下列程式碼會登入服務主體帳戶，並建立 DnsManagementClient 物件。

```cs
// Build the service credentials and DNS management client
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var dnsClient = new DnsManagementClient(serviceCreds);
dnsClient.SubscriptionId = subscriptionId;
```

## <a name="create-or-update-a-dns-zone"></a>建立或更新 DNS 區域

若要建立 DNS 區域，首先要建立包含 DNS 區域參數的「區域」物件。 因為 DNS 區域未連結到特定區域，所以位置設定為 'global'。 在此範例中，還會對此區域新增 [Azure Resource Manager「標籤」](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/) 。

若要在 Azure DNS 中實際建立或更新區域，則會將包含區域參數的區域物件傳遞至「DnsManagementClient.Zones.CreateOrUpdateAsyc」  方法。

> [!NOTE]
> DnsManagementClient 支援三種作業模式︰同步 ('CreateOrUpdate')、非同步 ('CreateOrUpdateAsync')，或非同步並可存取 HTTP 回應 ('CreateOrUpdateWithHttpMessagesAsync')。  您可以根據應用程式的需要選擇上述任何模式。

Azure DNS 支援開放式同步存取，稱為 [Etag](dns-getstarted-create-dnszone.md)。 在此範例中，為 'If-None-Match' 標頭指定 "*" 會告訴 Azure DNS 建立 DNS 區域 (如果還沒有任何區域存在)。  如果指定的資源群組中已存在具有指定名稱的區域，呼叫就會失敗。

```cs
// Create zone parameters
var dnsZoneParams = new Zone("global"); // All DNS zones must have location = "global"

// Create a Azure Resource Manager 'tag'.  This is optional.  You can add multiple tags
dnsZoneParams.Tags = new Dictionary<string, string>();
dnsZoneParams.Tags.Add("dept", "finance");

// Create the actual zone.
// Note: Uses 'If-None-Match *' ETAG check, so will fail if the zone exists already.
// Note: For non-async usage, call dnsClient.Zones.CreateOrUpdate(resourceGroupName, zoneName, dnsZoneParams, null, "*")
// Note: For getting the http response, call dnsClient.Zones.CreateOrUpdateWithHttpMessagesAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*")
var dnsZone = await dnsClient.Zones.CreateOrUpdateAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*");
```

## <a name="create-dns-record-sets-and-records"></a>建立 DNS 記錄集和記錄

DNS 記錄是以記錄集形式管理。 記錄集是指一個區域內有相同名稱和記錄類型的一組記錄。  記錄集名稱會相對於區域名稱而非完整的 DNS 名稱。

若要建立或更新記錄集，則會建立「RecordSet」參數物件並傳遞給「DnsManagementClient.RecordSets.CreateOrUpdateAsync」 。 和 DNS 區域的情況一樣，作業模式也有三種︰同步 ('CreateOrUpdate')、非同步 ('CreateOrUpdateAsync')，或非同步並可存取 HTTP 回應 ('CreateOrUpdateWithHttpMessagesAsync')。

和 DNS 區域的情況一樣，記錄集上的作業包含開放式並行存取支援。  在此範例中，因為 'If-Match' 和 'If-None-Match' 都未指定，所以一律會建立記錄集。  這個呼叫會覆寫此 DNS 區域中任何具有相同名稱和記錄類型的現有記錄集。

```cs
// Create record set parameters
var recordSetParams = new RecordSet();
recordSetParams.TTL = 3600;

// Add records to the record set parameter object.  In this case, we'll add a record of type 'A'
recordSetParams.ARecords = new List<ARecord>();
recordSetParams.ARecords.Add(new ARecord("1.2.3.4"));

// Add metadata to the record set.  Similar to Azure Resource Manager tags, this is optional and you can add multiple metadata name/value pairs
recordSetParams.Metadata = new Dictionary<string, string>();
recordSetParams.Metadata.Add("user", "Mary");

// Create the actual record set in Azure DNS
// Note: no ETAG checks specified, will overwrite existing record set if one exists
var recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSetParams);
```

## <a name="get-zones-and-record-sets"></a>取得區域和記錄集

*DnsManagementClient.Zones.Get* 和 *DnsManagementClient.RecordSets.Get* 方法分別會擷取個別的區域和記錄集。 RecordSets 依其類型、名稱及所在的區域和資源群組來識別。 Zones 依其名稱及所在的資源群組來識別。

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);
```

## <a name="update-an-existing-record-set"></a>更新現有記錄集

若要更新現有 DNS 記錄集，請先擷取記錄集，接著更新記錄集內容，然後提交變更。  在此範例中，我們會從 'If-Match' 參數中所擷取的記錄集指定 'Etag'。 如果並行作業在此同時也修改了記錄集，則呼叫會失敗。

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);

// Add a new record to the local object.  Note that records in a record set must be unique/distinct
recordSet.ARecords.Add(new ARecord("5.6.7.8"));

// Update the record set in Azure DNS
// Note: ETAG check specified, update will be rejected if the record set has changed in the meantime
recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSet, recordSet.Etag);
```

## <a name="list-zones-and-record-sets"></a>列出區域和記錄集

若要列出區域，請使用*DnsManagementClient.Zones.List...* 方法，其支援列出指定資源群組中的所有區域，或是指定 Azure 訂用帳戶中的所有區域 (跨資源群組)。若要列出記錄集，請使用 *DnsManagementClient.RecordSets.List...* 方法，其支援列出指定區域中的所有記錄集，或只列出特定類型的記錄集。

請注意，在列出區域和記錄集時，其結果可能會分頁。  下列範例示範如何逐一查看結果頁面  (範例中使用了 '2' 的人為小型頁面大小來強制分頁；實際上應該忽略此參數並使用預設頁面大小)。

```cs
// Note: in this demo, we'll use a very small page size (2 record sets) to demonstrate paging
// In practice, to improve performance you would use a large page size or just use the system default
int recordSets = 0;
var page = await dnsClient.RecordSets.ListAllInResourceGroupAsync(resourceGroupName, zoneName, "2");
recordSets += page.Count();

while (page.NextPageLink != null)
{
    page = await dnsClient.RecordSets.ListAllInResourceGroupNextAsync(page.NextPageLink);
    recordSets += page.Count();
}
```

## <a name="next-steps"></a>後續步驟

下載 [Azure DNS .NET SDK 範例專案](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True)，其包括如何使用 Azure DNS .NET SDK 的其他範例，包括其他 DNS 記錄類型的範例。
