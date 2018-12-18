---
title: 使用 Azure 網路監看員來針對虛擬網路閘道和連線進行疑難排解 - REST | Microsoft Docs
description: 此頁面說明如何搭配使用 Azure 網路監看員和 REST，以針對虛擬網路閘道和連線進行疑難排解
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: e4d5f195-b839-4394-94ef-a04192766e55
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2017
ms.author: jdial
ms.openlocfilehash: e3c172a0e88988acfb0244fff4603dc644aee03f
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39091018"
---
# <a name="troubleshoot-virtual-network-gateway-and-connections-using-azure-network-watcher"></a>使用 Azure 網路監看員來針對虛擬網路閘道和連線進行疑難排解

> [!div class="op_single_selector"]
> - [入口網站](diagnose-communication-problem-between-networks.md)
> - [PowerShell](network-watcher-troubleshoot-manage-powershell.md)
> - [Azure CLI](network-watcher-troubleshoot-manage-cli.md)
> - [REST API](network-watcher-troubleshoot-manage-rest.md)

網路監看員提供了許多功能，因為它的作用就是為了讓您了解您在 Azure 中的網路資源。 這些功能的其中之一便是資源疑難排解。 您可以透過入口網站、PowerShell、CLI 或 REST API 呼叫資源疑難排解。 一經呼叫，網路監看員就會檢查虛擬網路閘道或連線的健全狀況，並傳回其調查結果。

本文會帶領您逐步完成資源疑難排解目前可用的不同管理工作。

- [針對虛擬網路閘道進行疑難排解](#troubleshoot-a-virtual-network-gateway)
- [針對連線進行疑難排解](#troubleshoot-connections)

## <a name="before-you-begin"></a>開始之前

使用 ARMclient 透過 PowerShell 呼叫 REST API。 您可以在 chocolatey 的 [ARMClient on Chocolatey](https://chocolatey.org/packages/ARMClient) 上找到 ARMClient

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。

如需支援的閘道類型清單，請瀏覽[支援的閘道類型](network-watcher-troubleshoot-overview.md#supported-gateway-types)。

## <a name="overview"></a>概觀

網路監看員疑難排解可讓您針對虛擬網路閘道和連線所發生的問題進行疑難排解。 要求進行資源疑難排解時會查詢並檢查記錄。 檢查完成時，就會傳回結果。 疑難排解 API 要求是執行時間很長的要求，可能需要幾分鐘的時間才會傳回結果。 記錄會儲存在儲存體帳戶的容器中。

## <a name="log-in-with-armclient"></a>使用 ARMClient 登入

```PowerShell
armclient login
```

## <a name="troubleshoot-a-virtual-network-gateway"></a>針對虛擬網路閘道進行疑難排解


### <a name="post-the-troubleshoot-request"></a>POST 疑難排解要求

下列範例會查詢虛擬網路閘道的狀態。

```powershell

$subscriptionId = "00000000-0000-0000-0000-000000000000"
$resourceGroupName = "ContosoRG"
$NWresourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$vnetGatewayName = "ContosoVNETGateway"
$storageAccountName = "contososa"
$containerName = "gwlogs"
$requestBody = @"
{
'TargetResourceId': '/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Network/virtualNetworkGateways/${vnetGatewayName}',
'Properties': {
'StorageId': '/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Storage/storageAccounts/${storageAccountName}',
'StoragePath': 'https://${storageAccountName}.blob.core.windows.net/${containerName}'
}
}
"@


armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${NWresourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/troubleshoot?api-version=2016-03-30" $requestBody -verbose
```

由於這項作業的執行時間很長，回應標頭中會傳回用於查詢作業的 URI 和結果的 URI，如下列回應所示：

**重要的值**

* **Azure AsyncOperation** - 此屬性包含用來查詢非同步疑難排解作業的 URI
* **Location** - 此屬性包含作業完成時結果所在的 URI

```
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 10
x-ms-request-id: 8a1167b7-6768-4ac1-85dc-703c9c9b9247
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operations/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30
Strict-Transport-Security: max-age=31536000; includeSubDomains
Cache-Control: no-cache
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operationResults/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30
Server: Microsoft-HTTPAPI/2.0; Microsoft-HTTPAPI/2.0
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 4364d88a-bd08-422c-a716-dbb0cdc99f7b
x-ms-routing-request-id: NORTHCENTRALUS:20170112T183202Z:4364d88a-bd08-422c-a716-dbb0cdc99f7b
Date: Thu, 12 Jan 2017 18:32:01 GMT

null
```

### <a name="query-the-async-operation-for-completion"></a>查詢非同步作業是否完成

使用作業 URI 來查詢作業的進度，如下列範例所示︰

```powershell
armclient get "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operations/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30" -verbose
```

當作業正在進行時，回應會顯示 **InProgress**，如下列範例所示︰

```json
{
  "status": "InProgress"
}
```

作業完成時，狀態會變更為 **Succeeded**。

```json
{
  "status": "Succeeded"
}
```

### <a name="retrieve-the-results"></a>擷取結果

一旦傳回的狀態是 **Succeeded**，請在 operationResult URI 上呼叫 GET 方法來擷取結果。

```powershell
armclient get "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operationResults/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30" -verbose
```

下列回應是查詢閘道疑難排解的結果時，通常會傳回的降級回應的範例。 請參閱[了解結果](#understanding-the-results)，以釐清回應中的屬性代表什麼意思。

```json
{
  "startTime": "2017-01-12T10:31:41.562646-08:00",
  "endTime": "2017-01-12T18:31:48.677Z",
  "code": "Degraded",
  "results": [
    {
      "id": "PlatformInActive",
      "summary": "We are sorry, your VPN gateway is in standby mode",
      "detail": "During this time the gateway will not initiate or accept VPN connections with on premises VPN devices or other Azure VPN Gateways. This is a transient state while the Azure platform is being updated.",
      "recommendedActions": [
        {
          "actionText": "If the condition persists, please try resetting your Azure VPN gateway",
          "actionUri": "https://azure.microsoft.com/documentation/articles/vpn-gateway-resetgw-classic/",
          "actionUriText": "resetting the VPN Gateway"
        },
        {
          "actionText": "If your VPN gateway isn't up and running by the expected resolution time, contact support",
          "actionUri": "http://azure.microsoft.com/support",
          "actionUriText": "contact support"
        }
      ]
    },
    {
      "id": "NoFault",
      "summary": "This VPN gateway is running normally",
      "detail": "There aren't any known Azure platform problems affecting this VPN Connection",
      "recommendedActions": [
        {
          "actionText": "If you are still experience problems with the VPN gateway, please try resetting the VPN gateway.",
          "actionUri": "https://azure.microsoft.com/documentation/articles/vpn-gateway-resetgw-classic/",
          "actionUriText": "resetting VPN gateway"
        },
        {
          "actionText": "If you are experiencing problems you believe are caused by Azure, contact support",
          "actionUri": "http://azure.microsoft.com/support",
          "actionUriText": "contact support"
        }
      ]
    }
  ]
}
```


## <a name="troubleshoot-connections"></a>針對連線進行疑難排解

下列範例會查詢連線的狀態。

```powershell

$subscriptionId = "00000000-0000-0000-0000-000000000000"
$resourceGroupName = "ContosoRG"
$NWresourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$connectionName = "VNET2toVNET1Connection"
$storageAccountName = "contososa"
$containerName = "gwlogs"
$requestBody = @{
"TargetResourceId": "/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Network/connections/${connectionName}",
"Properties": {
"StorageId": "/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Storage/storageAccounts/${storageAccountName}",
"StoragePath": "https://${storageAccountName}.blob.core.windows.net/${containerName}"
}

}
armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${NWresourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/troubleshoot?api-version=2016-03-30 $requestBody"
```

> [!NOTE]
> 無法同時針對連線和其相對應的閘道執行疑難排解作業。 作業必須完成，才能在先前的資源上執行。

由於這是長時間執行的交易，回應標頭中會傳回用於查詢作業的 URI 和結果的 URI，如下列回應所示：

**重要的值**

* **Azure AsyncOperation** - 此屬性包含用來查詢非同步疑難排解作業的 URI
* **Location** - 此屬性包含作業完成時結果所在的 URI

```
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 10
x-ms-request-id: 8a1167b7-6768-4ac1-85dc-703c9c9b9247
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operations/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30
Strict-Transport-Security: max-age=31536000; includeSubDomains
Cache-Control: no-cache
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operationResults/8a1167b7-6768-4ac1-85dc-703c9c9b9247?api-version=2016-03-30
Server: Microsoft-HTTPAPI/2.0; Microsoft-HTTPAPI/2.0
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 4364d88a-bd08-422c-a716-dbb0cdc99f7b
x-ms-routing-request-id: NORTHCENTRALUS:20170112T183202Z:4364d88a-bd08-422c-a716-dbb0cdc99f7b
Date: Thu, 12 Jan 2017 18:32:01 GMT

null
```

### <a name="query-the-async-operation-for-completion"></a>查詢非同步作業是否完成

使用作業 URI 來查詢作業的進度，如下列範例所示︰

```powershell
armclient get "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operations/843b1c31-4717-4fdd-b7a6-4c786ca9c501?api-version=2016-03-30"
```

當作業正在進行時，回應會顯示 **InProgress**，如下列範例所示︰

```json
{
  "status": "InProgress"
}
```

作業完成時，狀態會變更為 **Succeeded**。

```json
{
  "status": "Succeeded"
}
```

下列回應是查詢連線疑難排解的結果時，通常會傳回的回應範例。

### <a name="retrieve-the-results"></a>擷取結果

一旦傳回的狀態是 **Succeeded**，請在 operationResult URI 上呼叫 GET 方法來擷取結果。

```powershell
armclient get "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Network/locations/westcentralus/operationResults/843b1c31-4717-4fdd-b7a6-4c786ca9c501?api-version=2016-03-30"
```

下列回應是查詢連線疑難排解的結果時，通常會傳回的回應範例。

```json
{
  "startTime": "2017-01-12T14:09:19.1215346-08:00",
  "endTime": "2017-01-12T22:09:23.747Z",
  "code": "UnHealthy",
  "results": [
    {
      "id": "PlatformInActive",
      "summary": "We are sorry, your VPN gateway is in standby mode",
      "detail": "During this time the gateway will not initiate or accept VPN connections with on premises VPN devices or other Azure VPN Gateways. This 
is a transient state while the Azure platform is being updated.",
      "recommendedActions": [
        {
          "actionText": "If the condition persists, please try resetting your Azure VPN gateway",
          "actionUri": "https://azure.microsoft.com/documentation/articles/vpn-gateway-resetgw-classic/",
          "actionUriText": "resetting the VPN gateway"
        },
        {
          "actionText": "If your VPN Connection isn't up and running by the expected resolution time, contact support",
          "actionUri": "http://azure.microsoft.com/support",
          "actionUriText": "contact support"
        }
      ]
    },
    {
      "id": "NoFault",
      "summary": "This VPN Connection is running normally",
      "detail": "There aren't any known Azure platform problems affecting this VPN Connection",
      "recommendedActions": [
        {
          "actionText": "If you are still experience problems with the VPN gateway, please try resetting the VPN gateway.",
          "actionUri": "https://azure.microsoft.com/documentation/articles/vpn-gateway-resetgw-classic/",
          "actionUriText": "resetting VPN gateway"
        },
        {
          "actionText": "If you are experiencing problems you believe are caused by Azure, contact support",
          "actionUri": "http://azure.microsoft.com/support",
          "actionUriText": "contact support"
        }
      ]
    }
  ]
}
```

## <a name="understanding-the-results"></a>了解結果

動作文字會提供有關如何解決問題的一般指引。 如果問題有可行動作，則會提供附有其他指引的連結。 如果沒有其他指引，回應中會提供 URL 以供您開啟支援案例。  如需回應屬性和所含內容的詳細資訊，請瀏覽[網路監看員疑難排解概觀](network-watcher-troubleshoot-overview.md)

如需從 Azure 儲存體帳戶下載檔案的指示，請參閱[以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 另一項可用工具為儲存體總管。 有關儲存體總管的詳細資訊可以在下列連結找到︰[儲存體總管](http://storageexplorer.com/)

## <a name="next-steps"></a>後續步驟

如果設定已變更而停止了 VPN 連線，請參閱[管理網路安全性群組](../virtual-network/manage-network-security-group.md)以追蹤可能有問題的網路安全性群組和安全性規則。
