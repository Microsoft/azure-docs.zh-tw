---
title: 使用 Azure 網路監看員管理網路安全性群組流量記錄 - REST API | Microsoft Docs
description: 此頁面說明如何使用 REST API 在 Azure 網路監看員中管理網路安全性群組流量記錄
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 2ab25379-0fd3-4bfe-9d82-425dfc7ad6bb
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 003335aad0452e7a2dbfff49ed29a6b99b5d54d2
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39089624"
---
# <a name="configuring-network-security-group-flow-logs-using-rest-api"></a>使用 REST API 設定網路安全性群組流量記錄

> [!div class="op_single_selector"]
> - [Azure 入口網站](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

網路安全性群組流量記錄是網路監看員的一項功能，可讓您檢視透過網路安全性群組傳輸之輸入和輸出 IP 流量的相關資訊。 這些流量記錄是以 json 格式撰寫，會顯示每一規則的輸出和輸入流量、流量套用至的 NIC、有關流量的 5 個 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及流量是被允許或拒絕。

## <a name="before-you-begin"></a>開始之前

使用 ARMclient 透過 PowerShell 呼叫 REST API。 您可以在 chocolatey 的 [ARMClient on Chocolatey](https://chocolatey.org/packages/ARMClient) 上找到 ARMClient

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。

> [!Important]
> 針對網路監看員 REST API 呼叫，要求 URI 中的資源群組名稱是包含網路監看員的資源群組，而非您要執行診斷動作的資源。

## <a name="scenario"></a>案例

本文章涵蓋的案例會示範如何使用 REST API 啟用、停用以及查詢流程記錄。 若要深入了解網路安全性群組流程記錄，請造訪[網路安全性群組流程記錄 - 概觀](network-watcher-nsg-flow-logging-overview.md)。

在此案例中，您將會：

* 啟用流量記錄
* 停用流程記錄
* 查詢流量記錄狀態

## <a name="log-in-with-armclient"></a>使用 ARMClient 登入

使用 Azure 認證登入 Armclient。

```PowerShell
armclient login
```

## <a name="register-insights-provider"></a>註冊 Insights 提供者

若要讓流量記錄成功運作，必須註冊 **Microsoft.Insights** 提供者。 如果您不確定是否已註冊 **Microsoft.Insights** 提供者，請執行下列程式碼。

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
armclient post "https://management.azure.com//subscriptions/${subscriptionId}/providers/Microsoft.Insights/register?api-version=2016-09-01"
```

## <a name="enable-network-security-group-flow-logs"></a>啟用網路安全性群組流量記錄

用來啟用流量記錄的命令如下列範例所示︰

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'true',
    'retentionPolicy' : {
            days: 5,
            enabled: true
        }
    }
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

前述範例中所傳回的回應如下所示︰

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    }
  }
}
```

## <a name="disable-network-security-group-flow-logs"></a>停用網路安全性群組流量記錄

使用下列範例來停用流量記錄。 呼叫等同於啟用流程記錄，除了針對已啟用屬性設定的 **false**。

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'false',
    'retentionPolicy' : {
            days: 5,
            enabled: true
        }
    }
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

前述範例中所傳回的回應如下所示︰

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": false,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    }
  }
}
```

## <a name="query-flow-logs"></a>查詢流量記錄

下列 REST 呼叫會查詢網路安全性群組上的流程記錄狀態。

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/queryFlowLogStatus?api-version=2016-12-01" $requestBody
```

以下是所回傳之回應的範例：

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
   "retentionPolicy": {
      "days": 5,
      "enabled": true
    }
  }
}
```

## <a name="download-a-flow-log"></a>下載流量記錄

流量記錄的儲存位置會在建立時定義。 若要存取這些儲存至儲存體帳戶的流量記錄，Microsoft Azure 儲存體總管是很便利的工具，您可以在這裡下載︰http://storageexplorer.com/

如果指定了儲存體帳戶，封包擷取檔案便會儲存到儲存體帳戶的下列位置︰

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

## <a name="next-steps"></a>後續步驟

了解如何[使用 PowerBI 視覺化 NSG 流量記錄](network-watcher-visualize-nsg-flow-logs-power-bi.md)

了解如何[使用開放原始碼工具視覺化 NSG 流量記錄](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
