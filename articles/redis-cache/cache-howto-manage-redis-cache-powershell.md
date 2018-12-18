---
title: 使用 Azure PowerShell 管理 Azure Redis 快取 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 執行 Azure Redis 快取的管理工作。
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 1136efe5-1e33-4d91-bb49-c8e2a6dca475
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.author: wesmc
ms.openlocfilehash: 11b399bdbbbe92d1e1544f4cbae0824140ce52c1
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49364501"
---
# <a name="manage-azure-redis-cache-with-azure-powershell"></a>使用 Azure PowerShell 管理 Azure Redis 快取
> [!div class="op_single_selector"]
> * [PowerShell](cache-howto-manage-redis-cache-powershell.md)
> * [Azure CLI](cache-manage-cli.md)
> 
> 

本主題顯示如何執行一般工作，例如建立、更新及調整 Azure Redis 快取執行個體，如何重新產生存取金鑰，以及如何檢視您的快取的相關資訊。 如需 Azure Redis 快取的 PowerShell Cmdlet 完整清單，請參閱 [Azure Redis 快取的 Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.rediscache/?view=azurermps-6.6.0)。

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

如需傳統部署模型的詳細資訊，請參閱 [Azure Resource Manager 與傳統部署比較：了解資源的部署模型和狀態](../azure-resource-manager/resource-manager-deployment-model.md)。

## <a name="prerequisites"></a>必要條件
如果您已安裝 Azure PowerShell，其必須是 Azure PowerShell 1.0.0 或更新的版本。 您可以在 Azure PowerShell 命令提示字元下使用這個命令來檢查已安裝的 Azure PowerShell 版本。

    Get-Module azure | format-table version


首先，您必須使用此命令登入 Azure。

    Connect-AzureRmAccount

在 [Microsoft Azure 登入] 對話方塊中，指定 Azure 帳戶的電子郵件地址和密碼。

接下來，如果您有多個 Azure 訂用帳戶，請設定 Azure 訂用帳戶。 如果想查看目前的訂用帳戶清單，請執行這個命令。

    Get-AzureRmSubscription | sort SubscriptionName | Select SubscriptionName

若要指定訂用帳戶，請執行下列命令。 在下列範例中，訂用帳戶的名稱為 `ContosoSubscription`。

    Select-AzureRmSubscription -SubscriptionName ContosoSubscription

在將 Windows PowerShell 與 Azure 資源管理員搭配使用之前，您需要下列項目：

* Windows PowerShell 3.0 或 4.0 版本。 若要找出 Windows PowerShell 的版本，輸入：`$PSVersionTable`，並確認 `PSVersion` 的值是 3.0 或 4.0。 若要安裝相容版本，請參閱 [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)。

若要取得您在本教學課程中任何所見 Cmdlet 的詳細說明，請使用 Get-Help Cmdlet。

    Get-Help <cmdlet-name> -Detailed

例如，如需取得 `New-AzureRmRedisCache` Cmdlet 的說明，請輸入：

    Get-Help New-AzureRmRedisCache -Detailed

### <a name="how-to-connect-to-other-clouds"></a>如何連線到其他雲端
根據預設，Azure 環境是 `AzureCloud`，其代表全域 Azure 雲端執行個體。 若要連線至不同的執行個體，請使用 `Connect-AzureRmAccount` 命令搭配 `-Environment` 或使用 -`EnvironmentName` 命令列參數搭配所需的環境或環境名稱。

若要查看可用環境的清單，請執行 `Get-AzureRmEnvironment` Cmdlet。

### <a name="to-connect-to-the-azure-government-cloud"></a>連線到 Azure Government 雲端
如果要連線到 Azure Government 雲端，請使用下列其中一個命令。

    Connect-AzureRmAccount -EnvironmentName AzureUSGovernment

或

    Connect-AzureRmAccount -Environment (Get-AzureRmEnvironment -Name AzureUSGovernment)

如果要在 Azure Government 雲端建立快取，請使用下列其中一個位置。

* 美國政府維吉尼亞州
* 美國政府愛荷華州

如需 Azure Government 雲端的詳細資訊，請參閱 [Microsoft Azure Government](https://azure.microsoft.com/features/gov/) 和 [Microsoft Azure Government 開發人員指南](../azure-government-developer-guide.md)。

### <a name="to-connect-to-the-azure-china-cloud"></a>連線到 Azure 中國雲端
如果要連線到 Azure 中國雲端，請使用下列其中一個命令。

    Connect-AzureRmAccount -EnvironmentName AzureChinaCloud

或

    Connect-AzureRmAccount -Environment (Get-AzureRmEnvironment -Name AzureChinaCloud)

如果要在 Azure 中國雲端建立快取，請使用下列其中一個位置。

* 中國東部
* 中國北部

如需 Azure 中國雲端的詳細資訊，請參閱 [由中國的世紀互聯營運的 Azure 中國雲端](http://www.windowsazure.cn/)。

### <a name="to-connect-to-microsoft-azure-germany"></a>連線到 Microsoft Azure (德國)
若要連線到 Microsoft Azure (德國)，請使用下列其中一個命令。

    Connect-AzureRmAccount -EnvironmentName AzureGermanCloud


或

    Connect-AzureRmAccount -Environment (Get-AzureRmEnvironment -Name AzureGermanCloud)

若要在 Microsoft Azure (德國) 中建立快取，請使用下列其中一個位置。

* 德國中部
* 德國東北部

如需有關 Microsoft Azure (德國) 的詳細資訊，請參閱 [Microsoft Azure (德國)](https://azure.microsoft.com/overview/clouds/germany/)。

### <a name="properties-used-for-azure-redis-cache-powershell"></a>用於 Azure Redis 快取 PowerShell 的屬性
下表為使用 Azure PowerShell 建立和管理 Azure Redis 快取執行個體時，常用參數的屬性和說明。

| 參數 | 說明 | 預設值 |
| --- | --- | --- |
| 名稱 |快取的名稱 | |
| 位置 |快取的位置 | |
| resourceGroupName |資源群組名稱，將在其中建立快取 | |
| 大小 |快取的大小。 有效值為：P1、P2、P3、P4、C0、C1、C2、C3、C4、C5、C6、250MB、1GB、2.5GB、6GB、13GB、26GB、53GB |1GB |
| ShardCount |在啟用叢集的情況下建立進階快取時要建立的分區數目。 有效值為：1、2、3、4、5、6、7、8、9、10 | |
| SKU |指定快取的 SKU。 有效值為：Basic、Standard、Premium |標準 |
| RedisConfiguration |指定 Redis 組態設定。 如需每個設定的詳細資訊，請參閱以下的 [RedisConfiguration 屬性](#redisconfiguration-properties) 表格。 | |
| EnableNonSslPort |指出是否已啟用非 SSL 連接埠。 |False |
| MaxMemoryPolicy |這個參數已被取代，請改用 RedisConfiguration。 | |
| StaticIP |當快取是裝載在 VNET 中，為快取在子網路中指定唯一 IP 位址。 如果未提供，則會從子網路中為您選擇一個。 | |
| 子網路 |當快取是裝載在 VNET 中，指定要在其中部署快取的子網路。 | |
| VirtualNetwork |當快取是裝載在 VNET 中，指定要在其中部署快取的 VNET 之資源識別碼。 | |
| KeyType |指定更新存取金鑰時要重新產生哪一個存取金鑰。 有效值為：Primary、Secondary | |

### <a name="redisconfiguration-properties"></a>RedisConfiguration 屬性
| 屬性 | 說明 | 定價層 |
| --- | --- | --- |
| rdb-backup-enabled |是否已啟用 [Redis 資料持續性](cache-how-to-premium-persistence.md) |僅限進階版 |
| rdb-storage-connection-string |[Redis 資料持續性](cache-how-to-premium-persistence.md) |僅限進階版 |
| rdb-backup-frequency |[Redis 資料持續性](cache-how-to-premium-persistence.md) |僅限進階版 |
| maxmemory-reserved |設定非快取程序的 [保留記憶體](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) |標準和進階 |
| maxmemory-policy |設定快取的 [收回原則](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) |所有定價層 |
| notify-keyspace-events |設定 [Keyspace 通知](cache-configure.md#keyspace-notifications-advanced-settings) |標準和進階 |
| hash-max-ziplist-entries |設定小型彙總資料類型的 [記憶體最佳化](http://redis.io/topics/memory-optimization) |標準和進階 |
| hash-max-ziplist-value |設定小型彙總資料類型的 [記憶體最佳化](http://redis.io/topics/memory-optimization) |標準和進階 |
| set-max-intset-entries |設定小型彙總資料類型的 [記憶體最佳化](http://redis.io/topics/memory-optimization) |標準和進階 |
| zset-max-ziplist-entries |設定小型彙總資料類型的 [記憶體最佳化](http://redis.io/topics/memory-optimization) |標準和進階 |
| zset-max-ziplist-value |設定小型彙總資料類型的 [記憶體最佳化](http://redis.io/topics/memory-optimization) |標準和進階 |
| 資料庫 |設定資料庫數目。 這個屬性僅可以在建立快取時設定。 |標準和進階 |

## <a name="to-create-a-redis-cache"></a>建立 Redis 快取
使用 [New-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/new-azurermrediscache?view=azurermps-6.6.0) Cmdlet 建立新的 Azure Redis 快取執行個體。

> [!IMPORTANT]
> 您第一次使用 Azure 入口網站在訂用帳戶中建立 Redis 快取時，入口網站會為該訂用帳戶註冊 `Microsoft.Cache` 命名空間。 如果您嘗試使用 PowerShell 在訂用帳戶中建立第一個 Redis 快取，您必須先使用下列命令註冊該命名空間；否則 Cmdlet (例如 `New-AzureRmRedisCache` 和 `Get-AzureRmRedisCache`) 會失敗。
> 
> `Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.Cache"`
> 
> 

若要查看 `New-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help New-AzureRmRedisCache -detailed

    NAME
        New-AzureRmRedisCache

    SYNOPSIS
        Creates a new redis cache.


    SYNTAX
        New-AzureRmRedisCache -Name <String> -ResourceGroupName <String> -Location <String> [-RedisVersion <String>]
        [-Size <String>] [-Sku <String>] [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort
        <Boolean>] [-ShardCount <Integer>] [-VirtualNetwork <String>] [-Subnet <String>] [-StaticIP <String>]
        [<CommonParameters>]


    DESCRIPTION
        The New-AzureRmRedisCache cmdlet creates a new redis cache.


    PARAMETERS
        -Name <String>
            Name of the redis cache to create.

        -ResourceGroupName <String>
            Name of resource group in which to create the redis cache.

        -Location <String>
            Location in which to create the redis cache.

        -RedisVersion <String>
            RedisVersion is deprecated and will be removed in future release.

        -Size <String>
            Size of the redis cache. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of redis cache. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value, databases.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Redis Cache. If no value is provided, the default value is false and the
            non-SSL port will be disabled. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        -VirtualNetwork <String>
            The exact ARM resource ID of the virtual network to deploy the redis cache in. Example format: /subscriptions/{
            subid}/resourceGroups/{resourceGroupName}/providers/Microsoft.ClassicNetwork/VirtualNetworks/{vnetName}

        -Subnet <String>
            Required when deploying a redis cache inside an existing Azure Virtual Network.

        -StaticIP <String>
            Required when deploying a redis cache inside an existing Azure Virtual Network.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要使用預設參數建立快取，請執行下列命令。

    New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US"

`ResourceGroupName`、`Name` 和 `Location` 是必要參數，其餘則為選擇性的而且有預設值。 執行先前的命令會建立標準 SKU Azure Redis 快取執行個體，具有指定的名稱、位置和資源群組，大小為 1 GB，且停用非 SSL 連接埠。

若要建立進階快取，請指定大小為 P1 (6 GB - 60 GB)、P2 (13 GB - 130 GB)、P3 (26 GB - 260 GB) 或 P4 (53 GB - 530 GB)。 若要啟用叢集，使用 `ShardCount` 參數指定分區計數。 下列範例會建立具有 3 個分區的 P1 進階快取。 P1 進階快取的大小為 6 GB，因為我們指定三個分區，大小總計為 18 GB (3 x 6 GB)。

    New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P1 -ShardCount 3

若要指定 `RedisConfiguration` 參數的值，以索引鍵/值組的方式將值括在 `{}` 內，例如 `@{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}`。 下列範例會建立標準 1 GB 快取，具有 `allkeys-random` maxmemory 原則，且 keyspace 通知設為 `KEA`。 如需詳細資訊，請參閱 [Keyspace 通知 (進階設定)](cache-configure.md#keyspace-notifications-advanced-settings) 和[記憶體原則](cache-configure.md#memory-policies)。

    New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}

<a name="databases"></a>

## <a name="to-configure-the-databases-setting-during-cache-creation"></a>在快取建立期間設定資料庫設定
`databases` 設定僅可以在快取建立期間設定。 下列範例會使用 [New-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/New-AzureRmRedisCache?view=azurermps-6.6.0) Cmdlet 建立具有 48 個資料庫的進階 P3 (26 GB) 快取。

    New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P3 -RedisConfiguration @{"databases" = "48"}

如需 `databases` 屬性的詳細資訊，請參閱 [預設的 Azure Redis 快取伺服器組態](cache-configure.md#default-redis-server-configuration)。 如需使用 [New-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/new-azurermrediscache?view=azurermps-6.6.0) Cmdlet 建立快取的詳細資訊，請參閱先前的[建立 Redis 快取](#to-create-a-redis-cache)一節。

## <a name="to-update-a-redis-cache"></a>更新 Redis 快取
使用 [Set-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/Set-AzureRmRedisCache?view=azurermps-6.6.0) Cmdlet 更新 Azure Redis 快取執行個體。

若要查看 `Set-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Set-AzureRmRedisCache -detailed

    NAME
        Set-AzureRmRedisCache

    SYNOPSIS
        Set redis cache updatable parameters.

    SYNTAX
        Set-AzureRmRedisCache -Name <String> -ResourceGroupName <String> [-Size <String>] [-Sku <String>]
        [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort <Boolean>] [-ShardCount
        <Integer>] [<CommonParameters>]

    DESCRIPTION
        The Set-AzureRmRedisCache cmdlet sets redis cache parameters.

    PARAMETERS
        -Name <String>
            Name of the redis cache to update.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -Size <String>
            Size of the redis cache. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of redis cache. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Redis Cache. The default value is null and no change will be made to the
            currently configured value. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

`Set-AzureRmRedisCache` Cmdlet 可用來更新屬性，例如 `Size`、`Sku`、`EnableNonSslPort` 和 `RedisConfiguration` 的值。 

下列命令會更新名為 myCache 的 Redis 快取的 maxmemory-policy。

    Set-AzureRmRedisCache -ResourceGroupName "myGroup" -Name "myCache" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random"}

<a name="scale"></a>

## <a name="to-scale-a-redis-cache"></a>調整 Redis 快取
修改 `Size`、`Sku` 或 `ShardCount` 屬性時，可用 `Set-AzureRmRedisCache` 調整 Azure Redis 快取執行個體。 

> [!NOTE]
> 使用 PowerShell 調整快取，和從 Azure 入口網站調整快取有相同的限制和準則。 您可以調整具有下列限制的不同定價層。
> 
> * 您無法從較高的定價層調整至較低的定價層。
> * 您無法從**進階**快取向下調整至**標準**或**基本**快取。
> * 您無法從**標準**快取向下調整到**基本**快取。
> * 您可以從**基本**快取調整到**標準**快取，但您無法同時變更大小。 如果您需要不同的大小，您可以進行後續調整作業，調整到您需要的大小。
> * 您無法直接從**基本**快取調整至**進階**快取。 您必須在單一調整作業中從**基本**調整至**標準**，然後在後續的調整作業中從**標準**調整至**進階**。
> * 您無法從較大的大小向下調整至 **C0 (250 MB)** 的大小。
> 
> 如需詳細資訊，請參閱 [如何調整 Azure Redis 快取](cache-how-to-scale.md)。
> 
> 

下列範例示範如何將名為 `myCache` 的快取調整為 2.5 GB 快取。 請注意，此命令可用於基本或標準快取。

    Set-AzureRmRedisCache -ResourceGroupName myGroup -Name myCache -Size 2.5GB

發出此命令之後，會傳回快取的狀態 (類似於呼叫 `Get-AzureRmRedisCache`)。 請注意，`ProvisioningState` 為 `Scaling`。

    PS C:\> Set-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup -Size 2.5GB


    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/mygroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Scaling
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 150], [notify-keyspace-events, KEA],
                         [maxmemory-delta, 150]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : mygroup
    PrimaryKey         : ....
    SecondaryKey       : ....
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :

當調整作業完成時，`ProvisioningState` 會變更為 `Succeeded`。 如果您需要進行後續的調整作業，例如先從基本變更為標準，然後再變更大小，您必須等到先前作業完成，否則會收到類似下列的錯誤。

    Set-AzureRmRedisCache : Conflict: The resource '...' is not in a stable state, and is currently unable to accept the update request.

## <a name="to-get-information-about-a-redis-cache"></a>取得 Redis 快取的資訊
您可以使用 [Get-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/get-azurermrediscache?view=azurermps-6.6.0) Cmdlet 擷取快取的相關資訊。

若要查看 `Get-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Get-AzureRmRedisCache -detailed

    NAME
        Get-AzureRmRedisCache

    SYNOPSIS
        Gets details about a single cache or all caches in the specified resource group or all caches in the current
        subscription.

    SYNTAX
        Get-AzureRmRedisCache [-Name <String>] [-ResourceGroupName <String>] [<CommonParameters>]

    DESCRIPTION
        The Get-AzureRmRedisCache cmdlet gets the details about a cache or caches depending on input parameters. If both
        ResourceGroupName and Name parameters are provided then Get-AzureRmRedisCache will return details about the
        specific cache name provided.

        If only ResourceGroupName is provided than it will return details about all caches in the specified resource group.

        If no parameters are given than it will return details about all caches the current subscription.

    PARAMETERS
        -Name <String>
            The name of the cache. When this parameter is provided along with ResourceGroupName, Get-AzureRmRedisCache
            returns the details for the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache or caches. If ResourceGroupName is provided with Name
            then Get-AzureRmRedisCache returns the details of the cache specified by Name. If only the ResourceGroup
            parameter is provided, then details for all caches in the resource group are returned.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要傳回目前訂用帳戶中所有快取的相關資訊，請不帶任何參數執行 `Get-AzureRmRedisCache`。

    Get-AzureRmRedisCache

若要傳回特定資源群組中所有快取的相關資訊，請執行 `Get-AzureRmRedisCache` 並使用 `ResourceGroupName` 參數。

    Get-AzureRmRedisCache -ResourceGroupName myGroup

若要傳回特定快取的相關資訊，請執行 `Get-AzureRmRedisCache`，並使用 `Name` 參數包含快取名稱，和 `ResourceGroupName` 參數包含快取的資源群組。

    PS C:\> Get-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup

    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/myGroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Succeeded
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 62], [notify-keyspace-events, KEA],
                         [maxclients, 1000]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : myGroup
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :

## <a name="to-retrieve-the-access-keys-for-a-redis-cache"></a>擷取 Redis 快取的存取金鑰
若要擷取您快取的存取金鑰，您可以使用 [Get-AzureRmRedisCacheKey](https://docs.microsoft.com/powershell/module/azurerm.rediscache/Get-AzureRmRedisCacheKey?view=azurermps-6.6.0) Cmdlet。

若要查看 `Get-AzureRmRedisCacheKey`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Get-AzureRmRedisCacheKey -detailed

    NAME
        Get-AzureRmRedisCacheKey

    SYNOPSIS
        Gets the accesskeys for the specified redis cache.


    SYNTAX
        Get-AzureRmRedisCacheKey -Name <String> -ResourceGroupName <String> [<CommonParameters>]

    DESCRIPTION
        The Get-AzureRmRedisCacheKey cmdlet gets the access keys for the specified cache.

    PARAMETERS
        -Name <String>
            Name of the redis cache.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要擷取您快取的金鑰，請呼叫 `Get-AzureRmRedisCacheKey` Cmdlet，並傳入快取的名稱以及包含快取的資源群組名稱。

    PS C:\> Get-AzureRmRedisCacheKey -Name myCache -ResourceGroupName myGroup

    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : ABhfB757JgjIgt785JgKH9865eifmekfnn649303JKL=

## <a name="to-regenerate-access-keys-for-your-redis-cache"></a>重新產生 Redis 快取的存取金鑰
若要重新產生您快取的存取金鑰，可以使用 [New-AzureRmRedisCacheKey](https://docs.microsoft.com/powershell/module/azurerm.rediscache/New-AzureRmRedisCacheKey?view=azurermps-6.6.0) Cmdlet。

若要查看 `New-AzureRmRedisCacheKey`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help New-AzureRmRedisCacheKey -detailed

    NAME
        New-AzureRmRedisCacheKey

    SYNOPSIS
        Regenerates the access key of a redis cache.

    SYNTAX
        New-AzureRmRedisCacheKey -Name <String> -ResourceGroupName <String> -KeyType <String> [-Force] [<CommonParameters>]

    DESCRIPTION
        The New-AzureRmRedisCacheKey cmdlet regenerate the access key of a redis cache.

    PARAMETERS
        -Name <String>
            Name of the redis cache.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -KeyType <String>
            Specifies whether to regenerate the primary or secondary access key. Possible values are Primary or Secondary.

        -Force
            When the Force parameter is provided, the specified access key is regenerated without any confirmation prompts.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要重新產生快取的主要或次要金鑰，請呼叫 `New-AzureRmRedisCacheKey` Cmdlet，並傳入名稱、資源群組，且針對 `KeyType` 參數指定 `Primary` 或 `Secondary`。 在下列範例中，會重新產生快取的次要存取金鑰。

    PS C:\> New-AzureRmRedisCacheKey -Name myCache -ResourceGroupName myGroup -KeyType Secondary

    Confirm
    Are you sure you want to regenerate Secondary key for redis cache 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : c53hj3kh4jhHjPJk8l0jji785JgKH9865eifmekfnn6=

## <a name="to-delete-a-redis-cache"></a>刪除 Redis 快取
若要刪除 Redis 快取，請使用 [Remove-AzureRmRedisCache](https://docs.microsoft.com/powershell/module/azurerm.rediscache/remove-azurermrediscache?view=azurermps-6.6.0) Cmdlet。

若要查看 `Remove-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Remove-AzureRmRedisCache -detailed

    NAME
        Remove-AzureRmRedisCache

    SYNOPSIS
        Remove redis cache if exists.

    SYNTAX
        Remove-AzureRmRedisCache -Name <String> -ResourceGroupName <String> [-Force] [-PassThru] [<CommonParameters>

    DESCRIPTION
        The Remove-AzureRmRedisCache cmdlet removes a redis cache if it exists.

    PARAMETERS
        -Name <String>
            Name of the redis cache to remove.

        -ResourceGroupName <String>
            Name of the resource group of the cache to remove.

        -Force
            When the Force parameter is provided, the cache is removed without any confirmation prompts.

        -PassThru
            By default Remove-AzureRmRedisCache removes the cache and does not return any value. If the PassThru par
            is provided then Remove-AzureRmRedisCache returns a boolean value indicating the success of the operatio

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

在下列範例中，會移除名為 `myCache` 的快取。

    PS C:\> Remove-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup

    Confirm
    Are you sure you want to remove redis cache 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


## <a name="to-import-a-redis-cache"></a>匯入 Redis 快取
您可以使用 `Import-AzureRmRedisCache` Cmdlet 將資料匯入 Azure Redis 快取執行個體。

> [!IMPORTANT]
> 匯入/匯出僅供 [進階層](cache-premium-tier-intro.md) 快取使用。 如需匯入/匯出的詳細資訊，請參閱 [在 Azure Redis 快取中匯入與匯出資料](cache-how-to-import-export-data.md)。
> 
> 

若要查看 `Import-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Import-AzureRmRedisCache -detailed

    NAME
        Import-AzureRmRedisCache

    SYNOPSIS
        Import data from blobs to Azure Redis Cache.


    SYNTAX
        Import-AzureRmRedisCache -Name <String> -ResourceGroupName <String> -Files <String[]> [-Format <String>] [-Force]
        [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Import-AzureRmRedisCache cmdlet imports data from the specified blobs into Azure Redis Cache.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Files <String[]>
            SAS urls of blobs whose content should be imported into the cache.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -Force
            When the Force parameter is provided, import will be performed without any confirmation prompts.

        -PassThru
            By default Import-AzureRmRedisCache imports data in cache and does not return any value. If the PassThru
            parameter is provided then Import-AzureRmRedisCache returns a boolean value indicating the success of the
            operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).


以下命令能將資料從 SAS URI 所指定的 blob 匯入 Azure Redis 快取。

    PS C:\>Import-AzureRmRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Files @("https://mystorageaccount.blob.core.windows.net/mycontainername/blobname?sv=2015-04-05&sr=b&sig=caIwutG2uDa0NZ8mjdNJdgOY8%2F8mhwRuGNdICU%2B0pI4%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwd") -Force

## <a name="to-export-a-redis-cache"></a>匯出 Redis 快取
您可以使用 `Export-AzureRmRedisCache` Cmdlet 從 Azure Redis 快取執行個體匯出資料。

> [!IMPORTANT]
> 匯入/匯出僅供 [進階層](cache-premium-tier-intro.md) 快取使用。 如需匯入/匯出的詳細資訊，請參閱 [在 Azure Redis 快取中匯入與匯出資料](cache-how-to-import-export-data.md)。
> 
> 

若要查看 `Export-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Export-AzureRmRedisCache -detailed

    NAME
        Export-AzureRmRedisCache

    SYNOPSIS
        Exports data from Azure Redis Cache to a specified container.


    SYNTAX
        Export-AzureRmRedisCache -Name <String> -ResourceGroupName <String> -Prefix <String> -Container <String> [-Format
        <String>] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Export-AzureRmRedisCache cmdlet exports data from Azure Redis Cache to a specified container.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Prefix <String>
            Prefix to use for blob names.

        -Container <String>
            SAS url of container where data should be exported.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -PassThru
            By default Export-AzureRmRedisCache does not return any value. If the PassThru parameter is provided
            then Export-AzureRmRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).


以下命令能將資料從 Azure Redis 快取執行個體匯出到 SAS URI 所指定的容器。

        PS C:\>Export-AzureRmRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Prefix "blobprefix"
        -Container "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2015-04-05&sr=c&sig=HezZtBZ3DURmEGDduauE7
        pvETY4kqlPI8JCNa8ATmaw%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwdl"

## <a name="to-reboot-a-redis-cache"></a>重新啟動 Redis 快取
您可以使用 `Reset-AzureRmRedisCache` Cmdlet 重新啟動 Azure Redis 快取執行個體。

> [!IMPORTANT]
> 重新啟動僅適用於 [進階層](cache-premium-tier-intro.md) 快取。 如需重新啟動快取的詳細資訊，請參閱 [快取管理 - 重新啟動](cache-administration.md#reboot)。
> 
> 

若要查看 `Reset-AzureRmRedisCache`的可用參數清單及其說明，請執行下列命令。

    PS C:\> Get-Help Reset-AzureRmRedisCache -detailed

    NAME
        Reset-AzureRmRedisCache

    SYNOPSIS
        Reboot specified node(s) of an Azure Redis Cache instance.


    SYNTAX
        Reset-AzureRmRedisCache -Name <String> -ResourceGroupName <String> -RebootType <String> [-ShardId <Integer>]
        [-Force] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Reset-AzureRmRedisCache cmdlet reboots the specified node(s) of an Azure Redis Cache instance.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -RebootType <String>
            Which node to reboot. Possible values are "PrimaryNode", "SecondaryNode", "AllNodes".

        -ShardId <Integer>
            Which shard to reboot when rebooting a premium cache with clustering enabled.

        -Force
            When the Force parameter is provided, reset will be performed without any confirmation prompts.

        -PassThru
            By default Reset-AzureRmRedisCache does not return any value. If the PassThru parameter is provided
            then Reset-AzureRmRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).


以下命令會重新啟動指定快取的兩個節點。

        PS C:\>Reset-AzureRmRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -RebootType "AllNodes"
        -Force


## <a name="next-steps"></a>後續步驟
若要深入了解如何將 Windows PowerShell 與 Azure 搭配使用，請參閱下列資源：

* [MSDN 上的 Azure Redis 快取 Cmdlet 文件](https://docs.microsoft.com/powershell/module/azurerm.rediscache/?view=azurermps-6.6.0)
* [Azure Resource Manager Cmdlet](http://go.microsoft.com/fwlink/?LinkID=394765)：了解如何使用 Azure Resource Manager 模組中的 Cmdlet。
* [使用資源群組管理 Azure 資源](../azure-resource-manager/resource-group-template-deploy-portal.md)：了解如何在 Azure 入口網站中建立和管理資源群組。
* [Azure 部落格](https://azure.microsoft.com/blog/)：深入了解 Azure 的新功能。
* [Windows PowerShell 部落格](http://blogs.msdn.com/powershell)：深入了解 Windows PowerShell 的新功能。
* ["Hey, Scripting Guy!"部落格](http://blogs.technet.com/b/heyscriptingguy/)：從 Windows PowerShell 社群中取得實際的秘訣及訣竅。

