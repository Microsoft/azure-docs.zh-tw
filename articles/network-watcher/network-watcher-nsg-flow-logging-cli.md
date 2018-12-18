---
title: 使用 Azure 網路監看員管理網路安全性群組流量記錄 - Azure CLI | Microsoft Docs
description: 此頁面說明如何使用 Azure CLI 在 Azure 網路監看員中管理網路安全性群組流量記錄
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 2dfc3112-8294-4357-b2f8-f81840da67d3
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: e603ef749dbe66eda1c235b62c5155c4af6dc9db
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46955141"
---
# <a name="configuring-network-security-group-flow-logs-with-azure-cli"></a>使用 Azure CLI 設定網路安全性群組流量記錄

> [!div class="op_single_selector"]
> - [Azure 入口網站](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

網路安全性群組流量記錄是網路監看員的一項功能，可讓您檢視透過網路安全性群組傳輸之輸入和輸出 IP 流量的相關資訊。 這些流量記錄是以 json 格式撰寫，會顯示每一規則的輸出和輸入流量、流量套用至的 NIC、有關流量的 5 個 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及流量是被允許或拒絕。

若要執行此文章中的步驟，您必須[安裝適用於 Mac、Linux 和 Windows 的 Azure 命令列介面 (CLI)](/cli/azure/install-azure-cli)。

## <a name="register-insights-provider"></a>註冊 Insights 提供者

若要讓流量記錄成功運作，必須註冊 **Microsoft.Insights** 提供者。 如果您不確定是否已註冊 **Microsoft.Insights** 提供者，請執行下列程式碼。

```azurecli
az provider register --namespace Microsoft.Insights
```

## <a name="enable-network-security-group-flow-logs"></a>啟用網路安全性群組流量記錄

用來啟用流量記錄的命令如下列範例所示︰

```azurecli
az network watcher flow-log configure --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName
```

您指定的儲存體帳戶不能為其設定網路規則，以限制只能對 Microsoft 服務或特定虛擬網路進行網路存取。 儲存體帳戶和您要啟用流程記錄的 NSG 可以位於相同或不同的 Azure 訂用帳戶。 如果您使用不同訂用帳戶，兩者都必須與相同的 Azure Active Directory 租用戶相關聯。 為每個訂用帳戶所使用的帳戶必須具有[必要的權限](required-rbac-permissions.md)。 

如果儲存體帳戶位於不同的資源群組或訂用帳戶，而不是在網路安全性群組中，則指定儲存體帳戶的完整識別碼，而不是指定其名稱。 例如，如果儲存體帳戶位於名為 RG-Storage 的資源群組，則並非指定先前命令中的 storageAccountName，而是要指定 /subscriptions/{SubscriptionID}/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/storageAccountName。

## <a name="disable-network-security-group-flow-logs"></a>停用網路安全性群組流量記錄

使用下列範例來停用流量記錄︰

```azurecli
az network watcher flow-log configure --resource-group resourceGroupName --enabled false --nsg nsgName
```

## <a name="download-a-flow-log"></a>下載流量記錄

流量記錄的儲存位置會在建立時定義。 若要存取這些儲存至儲存體帳戶的流量記錄，Microsoft Azure 儲存體總管是很便利的工具，您可以在這裡下載︰ http://storageexplorer.com/

如果指定了儲存體帳戶，封包擷取檔案便會儲存到儲存體帳戶的下列位置︰

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

如需記錄結構的相關資訊，請造訪[網路安全性群組流量記錄概觀](network-watcher-nsg-flow-logging-overview.md)

## <a name="next-steps"></a>後續步驟

了解如何[使用 PowerBI 視覺化 NSG 流量記錄](network-watcher-visualize-nsg-flow-logs-power-bi.md)

了解如何[使用開放原始碼工具視覺化 NSG 流量記錄](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
