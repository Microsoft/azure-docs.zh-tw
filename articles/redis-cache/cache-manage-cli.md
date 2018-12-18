---
title: 使用 Azure 傳統 CLI 管理 Azure Redis Cache | Microsoft Docs
description: 了解如何在任何平台上安裝 Azure 傳統 CLI、如何使用它來連線到您的 Azure 帳戶，以及如何從傳統 CLI 建立及管理 Redis 快取。
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 964ff245-859d-4bc1-bccf-62e4b3c1169f
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: wesmc
ms.openlocfilehash: 0e8bbaad920f35028c51641779a3272f73f81f37
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978398"
---
# <a name="how-to-create-and-manage-azure-redis-cache-using-the-azure-classic-cli"></a>如何使用 Azure 傳統 CLI 來建立及管理 Azure Redis Cache
> [!div class="op_single_selector"]
> * [PowerShell](cache-howto-manage-redis-cache-powershell.md)
> * [Azure 傳統 CLI](cache-manage-cli.md)
>

Azure 傳統 CLI 是從任何平台管理 Azure 基礎結構的一個好方法。 此文章說明如何使用 Azure 傳統 CLI 來建立及管理您的 Azure Redis Cache 執行個體。

[!INCLUDE [outdated-cli-content](../../includes/contains-classic-cli-content.md)]
> [!NOTE]
> 如需最新的 Azure CLI 範例指令碼，請參閱 [Azure CLI Redis 快取範例](cli-samples.md)。

## <a name="prerequisites"></a>必要條件
若要使用 Azure 傳統 CLI 來建立及管理 Azure Redis Cache 執行個體，您必須完成下列步驟。

* 您必須具有 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。
* [安裝 Azure 傳統 CLI](../cli-install-nodejs.md)。
* 將您的 Azure CLI 安裝連結到個人 Azure 帳戶，或連結到公司或學校 Azure 帳戶，並使用 `azure login` 命令從傳統 CLI 登入。
* 在執行下列任何命令之前，執行 `azure config mode arm` 命令以將傳統 CLI 切換至資源管理員模式。 如需詳細資訊，請參閱[使用 Azure 傳統 CLI 來管理 Azure 資源與資源群組](../xplat-cli-azure-resource-manager.md)。

## <a name="redis-cache-properties"></a>Redis 快取屬性
當建立並更新 Redis 快取執行個體時會使用下列屬性。

| 屬性 | Switch | 說明 |
| --- | --- | --- |
| name |-n, --name |Redis 快取的名稱。 |
| 資源群組 |-g, --resource-group |資源群組的名稱。 |
| location |-l, --location |要建立快取的位置。 |
| size |-z, --size |Redis 快取的大小。 有效的值：[C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4] |
| sku |-x, --sku |Redis SKU。 應為其中一個：[Basic, Standard, Premium] |
| EnableNonSslPort |-e, --enable-non-ssl-port |Redis 快取的 EnableNonSslPort 屬性。 如果您想針對您的快取啟用非 SSL 連接埠，則加入此旗標 |
| Redis 設定 |-c, --redis-configuration |Redis 設定。 在這裡輸入設定金鑰和值的 JSON 格式字串。 格式："{"":"","":""}" |
| Redis 設定 |-f, --redis-configuration-file |Redis 設定。 在這裡輸入包含設定金鑰和值的檔案路徑。 檔案項目的格式：{"":"","":""} |
| 分區計數 |-r, --shard-count |建立在具有叢集之進階叢集快取的分區數目。 |
| 虛擬網路 |-v, --virtual-network |當快取是裝載在 VNET 中時，指定確切的虛擬網路 ARM 資源識別碼以將 Redis 快取部署到其中。 範例格式：/subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| 金鑰類型 |-t, --key-type |要更新的金鑰類型。 有效的值：[Primary, Secondary] |
| StaticIP |-p, --static-ip <static-ip> |當快取是裝載在 VNET 中，為快取在子網路中指定唯一 IP 位址。 如果未提供，則會從子網路中為您選擇一個。 |
| 子網路 |t, --subnet <subnet> |當快取是裝載在 VNET 中，指定要在其中部署快取的子網路。 |
| VirtualNetwork |-v, --virtual-network <virtual-network> |當快取是裝載在 VNET 中時，指定確切的虛擬網路 ARM 資源識別碼以將 Redis 快取部署到其中。 範例格式：/subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| 訂用帳戶 |-s, --subscription |訂用帳戶識別碼。 |

## <a name="see-all-redis-cache-commands"></a>查看所有的 Redis 快取命令
若要查看所有的 Redis 快取命令和參數，請使用 `azure rediscache -h` 命令。

    C:\>azure rediscache -h
    help:    Commands to manage your Azure Redis Cache(s)
    help:
    help:    Create a Redis Cache
    help:      rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
    help:
    help:    Delete an existing Redis Cache
    help:      rediscache delete [--name <name> --resource-group <resource-group> ]
    help:
    help:    List all Redis Caches within your Subscription or Resource Group
    help:      rediscache list [options]
    help:
    help:    Show properties of an existing Redis Cache
    help:      rediscache show [--name <name> --resource-group <resource-group>]
    help:
    help:    Change settings of an existing Redis Cache
    help:      rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
    help:
    help:    Renew the authentication key for an existing Redis Cache
    help:      rediscache renew-key [--name <name> --resource-group <resource-group> ]
    help:
    help:    Lists Primary and Secondary key of an existing Redis Cache
    help:      rediscache list-keys [--name <name> --resource-group <resource-group>]
    help:
    help:    Options:
    help:      -h, --help  output usage information
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="create-a-redis-cache"></a>建立 Redis Cache
若要建立 Redis 快取，請使用下列命令：

    azure rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]

如需有關此命令的詳細資訊，請執行 `azure rediscache create -h` 命令。

    C:\>azure rediscache create -h
    help:    Create a Redis Cache
    help:
    help:    Usage: rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
    help:
    help:    Options:
    help:      -h, --help                                               output usage information
    help:      -v, --verbose                                            use verbose output
    help:      -vv                                                      more verbose with debug output
    help:      --json                                                   use json output
    help:      -n, --name <name>                                        Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>                    Name of the Resource Group
    help:      -l, --location <location>                                Location to create cache.
    help:      -z, --size <size>                                        Size of the Redis Cache. Valid values: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4]
    help:      -x, --sku <sku>                                          Redis SKU. Should be one of : [Basic, Standard, Premium]
    help:      -e, --enable-non-ssl-port                                EnableNonSslPort property of the Redis Cache. Add this flag if you want to enable the Non SSL Port for your cache
    help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here. Format:"{"<key1>":"<value1>","<key2>":"<value2>"}"
    help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here. Format for the file entry: {"<key1>":"<value1>","<key2>":"<value2>"}
    help:      -r, --shard-count <shard-count>                          Number of Shards to create on a Premium Cluster Cache
    help:      -v, --virtual-network <virtual-network>                  The exact ARM resource ID of the virtual network to deploy the redis cache in. Example format: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1
    help:      -t, --subnet <subnet>                                    Required when deploying a redis cache inside an existing Azure Virtual Network
    help:      -p, --static-ip <static-ip>                              Required when deploying a redis cache inside an existing Azure Virtual Network
    help:      -s, --subscription <id>                                  the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="delete-an-existing-redis-cache"></a>刪除現有的 Redis 快取
若要刪除 Redis 快取，請使用下列命令：

    azure rediscache delete [--name <name> --resource-group <resource-group> ]

如需有關此命令的詳細資訊，請執行 `azure rediscache delete -h` 命令。

    C:\>azure rediscache delete -h
    help:    Delete an existing Redis Cache
    help:
    help:    Usage: rediscache delete [--name <name> --resource-group <resource-group> ]
    help:
    help:    Options:
    help:      -h, --help                             output usage information
    help:      -v, --verbose                          use verbose output
    help:      -vv                                    more verbose with debug output
    help:      --json                                 use json output
    help:      -n, --name <name>                      Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>  Name of the Resource Group under which the cache exists
    help:      -s, --subscription <subscription>      the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="list-all-redis-caches-within-your-subscription-or-resource-group"></a>列出您的訂用帳戶或資源群組中所有的 Redis 快取
若要列出您的訂用帳戶或資源群組中所有的 Redis 快取，請使用下列命令：

    azure rediscache list [options]

如需有關此命令的詳細資訊，請執行 `azure rediscache list -h` 命令。

    C:\>azure rediscache list -h
    help:    List all Redis Caches within your Subscription or Resource Group
    help:
    help:    Usage: rediscache list [options]
    help:
    help:    Options:
    help:      -h, --help                             output usage information
    help:      -v, --verbose                          use verbose output
    help:      -vv                                    more verbose with debug output
    help:      --json                                 use json output
    help:      -g, --resource-group <resource-group>  Name of the Resource Group
    help:      -s, --subscription <subscription>      the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="show-properties-of-an-existing-redis-cache"></a>顯示現有 Redis 快取的屬性
若要顯示現有的 Redis 快取屬性，請使用下列命令：

    azure rediscache show [--name <name> --resource-group <resource-group>]

如需有關此命令的詳細資訊，請執行 `azure rediscache show -h` 命令。

    C:\>azure rediscache show -h
    help:    Show properties of an existing Redis Cache
    help:
    help:    Usage: rediscache show [--name <name> --resource-group <resource-group>]
    help:
    help:    Options:
    help:      -h, --help                             output usage information
    help:      -v, --verbose                          use verbose output
    help:      -vv                                    more verbose with debug output
    help:      --json                                 use json output
    help:      -n, --name <name>                      Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>  Name of the Resource Group
    help:      -s, --subscription <subscription>      the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

<a name="scale"></a>

## <a name="change-settings-of-an-existing-redis-cache"></a>變更現有 Redis 快取的設定
若要變更現有的 Redis 快取設定，請使用下列命令：

    azure rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]

如需有關此命令的詳細資訊，請執行 `azure rediscache set -h` 命令。

    C:\>azure rediscache set -h
    help:    Change settings of an existing Redis Cache
    help:
    help:    Usage: rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
    help:
    help:    Options:
    help:      -h, --help                                               output usage information
    help:      -v, --verbose                                            use verbose output
    help:      -vv                                                      more verbose with debug output
    help:      --json                                                   use json output
    help:      -n, --name <name>                                        Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>                    Name of the Resource Group
    help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here.
    help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here.
    help:      -s, --subscription <subscription>                        the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="renew-the-authentication-key-for-an-existing-redis-cache"></a>更新現有的 Redis 快取驗證金鑰
若要更新現有 Redis 快取的驗證金鑰，請使用下列命令：

    azure rediscache renew-key [--name <name> --resource-group <resource-group> --key-type <key-type>]

針對 `key-type` 指定 `Primary` 或 `Secondary`。

如需有關此命令的詳細資訊，請執行 `azure rediscache renew-key -h` 命令。

    C:\>azure rediscache renew-key -h
    help:    Renew the authentication key for an existing Redis Cache
    help:
    help:    Usage: rediscache renew-key [--name <name> --resource-group <resource-group> ]
    help:
    help:    Options:
    help:      -h, --help                             output usage information
    help:      -v, --verbose                          use verbose output
    help:      -vv                                    more verbose with debug output
    help:      --json                                 use json output
    help:      -n, --name <name>                      Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>  Name of the Resource Group under which cache exists
    help:      -t, --key-type <key-type>              type of key to renew. Valid values are: 'Primary', 'Secondary'.
    help:      -s, --subscription <subscription>      the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)

## <a name="list-primary-and-secondary-keys-of-an-existing-redis-cache"></a>列出現有之 Redis 快取的主要和次要金鑰
若要列出現有之 Redis 快取的主要和次要金鑰，請使用下列命令：

    azure rediscache list-keys [--name <name> --resource-group <resource-group>]

如需有關此命令的詳細資訊，請執行 `azure rediscache list-keys -h` 命令。

    C:\>azure rediscache list-keys -h
    help:    Lists Primary and Secondary key of an existing Redis Cache
    help:
    help:    Usage: rediscache list-keys [--name <name> --resource-group <resource-group>]
    help:
    help:    Options:
    help:      -h, --help                             output usage information
    help:      -v, --verbose                          use verbose output
    help:      -vv                                    more verbose with debug output
    help:      --json                                 use json output
    help:      -n, --name <name>                      Name of the Redis Cache.
    help:      -g, --resource-group <resource-group>  Name of the Resource Group under which Cache exists
    help:      -s, --subscription <subscription>      the subscription identifier
    help:
    help:    Current Mode: arm (Azure Resource Management)
