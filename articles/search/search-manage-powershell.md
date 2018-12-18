---
title: 使用 Powershell 指令碼管理 Azure 搜尋服務 | Microsoft Docs
description: 使用 PowerShell 指令碼管理 Azure 搜尋服務。 建立或更新 Azure 搜尋服務，並管理 Azure 搜尋服務的系統管理金鑰
author: HeidiSteen
manager: cgronlun
tags: azure-resource-manager
services: search
ms.service: search
ms.devlang: powershell
ms.topic: conceptual
ms.date: 08/15/2016
ms.author: heidist
ms.openlocfilehash: 1d0024af3bbf9edfe8c43032878a8b61e35cea9c
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39000985"
---
# <a name="manage-your-azure-search-service-with-powershell"></a>使用 PowerShell 管理 Azure 搜尋服務
> [!div class="op_single_selector"]
> * [入口網站](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> 
> 

本主題說明用來執行多種 Azure 搜尋服務管理工作的 PowerShell 命令。 我們將逐步建立搜尋服務、調整及管理其 API 金鑰。
這些命令和 [Azure 搜尋管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement)中提供的管理選項相同。

## <a name="prerequisites"></a>必要條件
* 您必須有 Azure PowerShell 1.0 或更新版本。 如需指示，請參閱 [安裝並設定 Azure PowerShell](/powershell/azure/overview)。
* 您必須在 PowerShell 中登入 Azure 訂用帳戶，如下所述。

首先，您必須使用此命令登入 Azure。

    Connect-AzureRmAccount

在 [Microsoft Azure 登入] 對話方塊中，指定 Azure 帳戶的電子郵件地址和密碼。

或者，您可以 [非互動的方式使用服務主體登入](../azure-resource-manager/resource-group-authenticate-service-principal.md)。

如果您有多個 Azure 訂用帳戶，您必須設定 Azure 訂用帳戶。 如果想查看目前的訂閱帳戶清單，請執行這個命令。

    Get-AzureRmSubscription | sort SubscriptionName | Select SubscriptionName

若要指定訂用帳戶，請執行下列命令。 在下列範例中，訂用帳戶的名稱為 `ContosoSubscription`。

    Select-AzureRmSubscription -SubscriptionName ContosoSubscription

## <a name="commands-to-help-you-get-started"></a>可協助您開始使用的命令
    $serviceName = "your-service-name-lowercase-with-dashes"
    $sku = "free" # or "basic" or "standard" for paid services
    $location = "West US"
    # You can get a list of potential locations with
    # (Get-AzureRmResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Search'}).Locations
    $resourceGroupName = "YourResourceGroup" 
    # If you don't already have this resource group, you can create it with 
    # New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

    # Register the ARM provider idempotently. This must be done once per subscription
    Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.Search"

    # Create a new search service
    # This command will return once the service is fully created
    New-AzureRmResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateUri "https://gallery.azure.com/artifact/20151001/Microsoft.Search.1.0.9/DeploymentTemplates/searchServiceDefaultTemplate.json" `
        -NameFromTemplate $serviceName `
        -Sku $sku `
        -Location $location `
        -PartitionCount 1 `
        -ReplicaCount 1

    # Get information about your new service and store it in $resource
    $resource = Get-AzureRmResource `
        -ResourceType "Microsoft.Search/searchServices" `
        -ResourceGroupName $resourceGroupName `
        -ResourceName $serviceName `
        -ApiVersion 2015-08-19

    # View your resource
    $resource

    # Get the primary admin API key
    $primaryKey = (Invoke-AzureRmResourceAction `
        -Action listAdminKeys `
        -ResourceId $resource.ResourceId `
        -ApiVersion 2015-08-19).PrimaryKey

    # Regenerate the secondary admin API Key
    $secondaryKey = (Invoke-AzureRmResourceAction `
        -ResourceType "Microsoft.Search/searchServices/regenerateAdminKey" `
        -ResourceGroupName $resourceGroupName `
        -ResourceName $serviceName `
        -ApiVersion 2015-08-19 `
        -Action secondary).SecondaryKey

    # Create a query key for read only access to your indexes
    $queryKeyDescription = "query-key-created-from-powershell"
    $queryKey = (Invoke-AzureRmResourceAction `
        -ResourceType "Microsoft.Search/searchServices/createQueryKey" `
        -ResourceGroupName $resourceGroupName `
        -ResourceName $serviceName `
        -ApiVersion 2015-08-19 `
        -Action $queryKeyDescription).Key

    # View your query key
    $queryKey

    # Delete query key
    Remove-AzureRmResource `
        -ResourceType "Microsoft.Search/searchServices/deleteQueryKey/$($queryKey)" `
        -ResourceGroupName $resourceGroupName `
        -ResourceName $serviceName `
        -ApiVersion 2015-08-19

    # Scale your service up
    # Note that this will only work if you made a non "free" service
    # This command will not return until the operation is finished
    # It can take 15 minutes or more to provision the additional resources
    $resource.Properties.ReplicaCount = 2
    $resource | Set-AzureRmResource

    # Delete your service
    # Deleting your service will delete all indexes and data in the service
    $resource | Remove-AzureRmResource

## <a name="next-steps"></a>後續步驟
您現已建立服務，接著可以採取後續步驟：建立[索引](search-what-is-an-index.md)、[查詢索引](search-query-overview.md)，最後建立並管理使用 Azure 搜尋服務的搜尋應用程式。

* [在 Azure 入口網站中建立 Azure 搜尋服務索引](search-create-index-portal.md)
* [在 Azure 入口網站中使用搜尋總管查詢 Azure 搜尋服務索引](search-explorer.md)
* [設定索引子以從其他服務載入資料](search-indexer-overview.md)
* [如何以 .NET 使用 Azure 搜尋服務](search-howto-dotnet-sdk.md)
* [分析 Azure 搜尋服務的流量](search-traffic-analytics.md)

