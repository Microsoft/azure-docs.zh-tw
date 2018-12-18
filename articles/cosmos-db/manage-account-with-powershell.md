---
title: Azure Cosmos DB 自動化 - 使用 Powershell 管理 | Microsoft Docs
description: 使用 Azure Powershell 管理 Azure Cosmos DB 資料庫帳戶。
services: cosmos-db
author: dmakwana
manager: kfile
editor: ''
tags: azure-resource-manager
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 04/21/2017
ms.author: dimakwan
ms.openlocfilehash: 60b13c8284708ce46d62b6659b2631a4f551c2ab
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983103"
---
# <a name="create-an-azure-cosmos-db-account-using-powershell"></a>使用 PowerShell 建立 Azure Cosmos DB 帳戶

下列指南說明使用 Azure Powershell 自動管理 Azure Cosmos DB 資料庫帳戶的命令。 它也包含在[多重區域資料庫帳戶][distribute-data-globally.md]中管理帳戶金鑰和容錯移轉優先順序的命令。 更新資料庫帳戶可讓您修改一致性原則和新增/移除區域。 如需跨平台管理 Azure Cosmos DB 資料庫帳戶，您可以使用 [Azure CLI](cli-samples.md)、[資源提供者 REST API][rp-rest-api] 或 [Azure 入口網站](create-sql-api-dotnet.md#create-account)。

## <a name="getting-started"></a>開始使用

按照[如何安裝和設定 Azure PowerShell][powershell-install-configure] 的指示進行安裝，並在 Powershell 中登入 Azure Resource Manager 帳戶。

### <a name="notes"></a>注意

* 如果您想要執行下列命令但不需要使用者確認，請在命令附加 `-Force` 旗標。
* 下列所有命令都是同步的。

## <a id="create-documentdb-account-powershell"></a> 建立 Azure Cosmos DB 帳戶

此命令可讓您建立 Azure Cosmos DB 資料庫帳戶。 將新的資料庫帳戶設定為單一區域具或有特定[一致性原則](consistency-levels.md)的[多重區域][distribute-data-globally.md]。

    $locations = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0}, @{"locationName"="<read-region-location>"; "failoverPriority"=1})
    $iprangefilter = "<ip-range-filter>"
    $consistencyPolicy = @{"defaultConsistencyLevel"="<default-consistency-level>"; "maxIntervalInSeconds"="<max-interval>"; "maxStalenessPrefix"="<max-staleness-prefix>"}
    $CosmosDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy; "ipRangeFilter"=$iprangefilter}
    New-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName <resource-group-name>  -Location "<resource-group-location>" -Name <database-account-name> -Properties $CosmosDBProperties
    
* `<write-region-location>` 資料庫帳戶之寫入區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能為 0。 每個資料庫帳戶必須有一個寫入區域。
* `<read-region-location>` 資料庫帳戶之讀取區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能大於 0。 每個資料庫帳戶可以有多個讀取區域。
* `<ip-range-filter>` 會以 CIDR 形式指定一組 IP 位址或 IP 位址範圍，以作為所指定資料庫帳戶的允許用戶端 IP 清單。 IP 位址/範圍必須以逗號分隔，而且不得包含任何空格。 如需詳細資訊，請參閱 [Azure Cosmos DB 防火牆支援](firewall-support.md)
* `<default-consistency-level>` Azure Cosmos DB 帳戶的預設一致性層級。 如需詳細資訊，請參閱 [Azure Cosmos DB 的一致性層級](consistency-levels.md)。
* `<max-interval>` 搭配「限定過期」一致性使用時，這個值代表所容許的過期時間量 (以秒為單位)。 此值可接受的範圍是 1 - 100。
* `<max-staleness-prefix>` 搭配「限定過期」一致性使用時，這個值代表所容許的過期要求數。 此值可接受的範圍是 1 - 2,147,483,647。
* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<resource-group-location>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 Azure 資源群組的位置。
* `<database-account-name>` 要建立之 Azure Cosmos DB 資料庫帳戶的名稱。 只能使用小寫字母、數字及 '-' 字元，且長度必須為 3 到 50 個字元。

範例： 

    $locations = @(@{"locationName"="West US"; "failoverPriority"=0}, @{"locationName"="East US"; "failoverPriority"=1})
    $iprangefilter = ""
    $consistencyPolicy = @{"defaultConsistencyLevel"="BoundedStaleness"; "maxIntervalInSeconds"=5; "maxStalenessPrefix"=100}
    $CosmosDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy; "ipRangeFilter"=$iprangefilter}
    New-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Location "West US" -Name "docdb-test" -Properties $CosmosDBProperties

### <a name="notes"></a>注意
* 上述範例會建立一個有兩個區域的資料庫帳戶。 也可以建立一個有一個區域 (指定為寫入區域，容錯移轉優先順序值為 0)，或有兩個以上區域的資料庫帳戶。 如需詳細資訊，請參閱[多重區域資料庫帳戶][distribute-data-globally.md]。
* locations 必須是已正式推出 Azure Cosmos DB 的區域。 [Azure 區域頁面](https://azure.microsoft.com/regions/#services)會提供目前的區域清單。

## <a id="update-documentdb-account-powershell"></a> 更新 Azure Cosmos DB 資料庫帳戶

此命令可讓您更新 Azure Cosmos DB 資料庫帳戶屬性。 這包括一致性原則，以及資料庫帳戶存在的位置。

> [!NOTE]
> 此命令可讓您新增及移除區域，但不允許您修改容錯移轉優先順序。 若要修改容錯移轉優先順序，請參閱[下方](#modify-failover-priority-powershell)。

    $locations = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0}, @{"locationName"="<read-region-location>"; "failoverPriority"=1})
    $iprangefilter = "<ip-range-filter>"
    $consistencyPolicy = @{"defaultConsistencyLevel"="<default-consistency-level>"; "maxIntervalInSeconds"="<max-interval>"; "maxStalenessPrefix"="<max-staleness-prefix>"}
    $CosmosDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy; "ipRangeFilter"=$iprangefilter}
    Set-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName <resource-group-name> -Name <database-account-name> -Properties $CosmosDBProperties
    
* `<write-region-location>` 資料庫帳戶之寫入區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能為 0。 每個資料庫帳戶必須有一個寫入區域。
* `<read-region-location>` 資料庫帳戶之讀取區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能大於 0。 每個資料庫帳戶可以有多個讀取區域。
* `<default-consistency-level>` Azure Cosmos DB 帳戶的預設一致性層級。 如需詳細資訊，請參閱 [Azure Cosmos DB 的一致性層級](consistency-levels.md)。
* `<ip-range-filter>` 會以 CIDR 形式指定一組 IP 位址或 IP 位址範圍，以作為所指定資料庫帳戶的允許用戶端 IP 清單。 IP 位址/範圍必須以逗號分隔，而且不得包含任何空格。 如需詳細資訊，請參閱 [Azure Cosmos DB 防火牆支援](firewall-support.md)
* `<max-interval>` 搭配「限定過期」一致性使用時，這個值代表所容許的過期時間量 (以秒為單位)。 此值可接受的範圍是 1 - 100。
* `<max-staleness-prefix>` 搭配「限定過期」一致性使用時，這個值代表所容許的過期要求數。 此值可接受的範圍是 1 - 2,147,483,647。
* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<resource-group-location>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 Azure 資源群組的位置。
* `<database-account-name>` 要更新之 Azure Cosmos DB 資料庫帳戶的名稱。

範例： 

    $locations = @(@{"locationName"="West US"; "failoverPriority"=0}, @{"locationName"="East US"; "failoverPriority"=1})
    $iprangefilter = ""
    $consistencyPolicy = @{"defaultConsistencyLevel"="BoundedStaleness"; "maxIntervalInSeconds"=5; "maxStalenessPrefix"=100}
    $CosmosDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy; "ipRangeFilter"=$iprangefilter}
    Set-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -Properties $CosmosDBProperties

## <a id="delete-documentdb-account-powershell"></a> 刪除 Azure Cosmos DB 資料庫帳戶

此命令可讓您刪除現有的 Azure Cosmos DB 資料庫帳戶。

    Remove-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"
    
* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` 要刪除之 Azure Cosmos DB 資料庫帳戶的名稱。

範例：

    Remove-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="get-documentdb-properties-powershell"></a> 取得 Azure Cosmos DB 資料庫帳戶的屬性

此命令可讓您取得現有 Azure Cosmos DB 資料庫帳戶的屬性。

    Get-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"

* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` Azure Cosmos DB 資料庫帳戶的名稱。

範例：

    Get-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="update-tags-powershell"></a> 更新 Azure Cosmos DB 資料庫帳戶的標記

下列範例說明如何設定 Azure Cosmos DB 資料庫帳戶的 [Azure 資源標記][azure-resource-tags]。

> [!NOTE]
> 此命令可以透過附加 `-Tags` 旗標與對應的參數，來結合建立或更新命令。

範例：

    $tags = @{"dept" = "Finance"; environment = "Production"}
    Set-AzureRmResource -ResourceType "Microsoft.DocumentDB/databaseAccounts"  -ResourceGroupName "rg-test" -Name "docdb-test" -Tags $tags

## <a id="list-account-keys-powershell"></a> 列出帳戶金鑰

當您建立 Azure Cosmos DB 帳戶時，服務會產生兩個主要存取金鑰，用於存取 Azure Cosmos DB 帳戶時的驗證。 透過提供這兩個存取金鑰，Azure Cosmos DB 讓您可以重新產生金鑰，同時又不需中斷 Azure Cosmos DB 帳戶。 也提供驗證唯讀作業的唯讀金鑰。 有兩個讀寫金鑰 (主要和次要) 和兩個唯讀金鑰 (主要和次要)。

    $keys = Invoke-AzureRmResourceAction -Action listKeys -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"

* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` Azure Cosmos DB 資料庫帳戶的名稱。

範例：

    $keys = Invoke-AzureRmResourceAction -Action listKeys -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="list-connection-strings-powershell"></a> 列出連接字串

對於 MongoDB 帳戶，您可以使用下列命令來擷取將您的 MongoDB 應用程式連線到資料庫帳戶的連接字串。

    $keys = Invoke-AzureRmResourceAction -Action listConnectionStrings -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"

* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` Azure Cosmos DB 資料庫帳戶的名稱。

範例：

    $keys = Invoke-AzureRmResourceAction -Action listConnectionStrings -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="regenerate-account-key-powershell"></a> 重新產生帳戶金鑰

您應定期變更 Azure Cosmos DB 帳戶的存取金鑰，讓連線更加安全。 指派的兩個存取金鑰可讓您在重新產生一個存取金鑰的同時，使用另一個存取金鑰維持 Azure Cosmos DB 帳戶連線。

    Invoke-AzureRmResourceAction -Action regenerateKey -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>" -Parameters @{"keyKind"="<key-kind>"}

* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` Azure Cosmos DB 資料庫帳戶的名稱。
* `<key-kind>` 您想要重新產生的其中一種金鑰類型：["Primary"|"Secondary"|"PrimaryReadonly"|"SecondaryReadonly"]。

範例：

    Invoke-AzureRmResourceAction -Action regenerateKey -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -Parameters @{"keyKind"="Primary"}

## <a id="modify-failover-priority-powershell"></a> 修改 Azure Cosmos DB 資料庫帳戶的容錯移轉優先順序

如果是多重區域資料庫帳戶，您可以變更 Azure Cosmos DB 資料庫帳戶所在之不同區域的容錯移轉優先順序。 如需有關 Azure Cosmos DB 資料庫帳戶中容錯移轉的詳細資訊，請參閱[使用 Azure Cosmos DB 全域散發資料][distribute-data-globally]。

    $failoverPolicies = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0},@{"locationName"="<read-region-location>"; "failoverPriority"=1})
    Invoke-AzureRmResourceAction -Action failoverPriorityChange -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>" -Parameters @{"failoverPolicies"=$failoverPolicies}

* `<write-region-location>` 資料庫帳戶之寫入區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能為 0。 每個資料庫帳戶必須有一個寫入區域。
* `<read-region-location>` 資料庫帳戶之讀取區域的位置名稱。 必須有這個位置，容錯移轉優先順序值才能大於 0。 每個資料庫帳戶可以有多個讀取區域。
* `<resource-group-name>` 新的 Azure Cosmos DB 資料庫帳戶所屬的 [Azure 資源群組][azure-resource-groups]的名稱。
* `<database-account-name>` Azure Cosmos DB 資料庫帳戶的名稱。

範例：

    $failoverPolicies = @(@{"locationName"="East US"; "failoverPriority"=0},@{"locationName"="West US"; "failoverPriority"=1})
    Invoke-AzureRmResourceAction -Action failoverPriorityChange -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -Parameters @{"failoverPolicies"=$failoverPolicies}

## <a name="next-steps"></a>後續步驟

* 若要使用 .NET 進行連線，請參閱[使用 .NET 進行連線和查詢](create-sql-api-dotnet.md)。
* 若要使用 Node.js 進行連線，請參閱[使用 Node.js 和 MongoDB 應用程式進行連線和查詢](create-mongodb-nodejs.md)。

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[powershell-install-configure]: https://docs.microsoft.com/azure/powershell-install-configure
[scaling-globally]: distribute-data-globally.md#EnableGlobalDistribution
[distribute-data-globally]: distribute-data-globally.md
[azure-resource-groups]: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-resource-tags]: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags
[rp-rest-api]: https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/
