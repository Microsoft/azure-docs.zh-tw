---
title: 使用 Azure 網路監看員管理封包擷取 - Azure CLI | Microsoft Docs
description: 此頁面說明如何使用 Azure CLI 管理網路監看員的封包擷取功能
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: cb0c1d10-f7f2-4c34-b08c-f73452430be8
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 70ec0f1acc54aae7de5f815d62a876ce3d033193
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966879"
---
# <a name="manage-packet-captures-with-azure-network-watcher-using-the-azure-cli"></a>使用 Azure CLI，利用 Azure 網路監看員管理封包擷取

> [!div class="op_single_selector"]
> - [Azure 入口網站](network-watcher-packet-capture-manage-portal.md)
> - [PowerShell](network-watcher-packet-capture-manage-powershell.md)
> - [Azure CLI](network-watcher-packet-capture-manage-cli.md)
> - [Azure REST API](network-watcher-packet-capture-manage-rest.md)

網路監看員封包擷取可讓您建立擷取工作階段來追蹤虛擬機器的流入和流出流量。 系統會為擷取工作階段提供篩選器，以確保您只會擷取到您想要的流量。 封包擷取有助於被動和主動地診斷網路異常。 其他用途包括收集網路統計資料、取得有關網路入侵的資訊，以及偵錯用戶端與伺服器間的通訊等等。 藉由能夠從遠端觸發封包擷取，這項功能可以減輕在所需機器上手動執行封包擷取的工作負擔，進而省下寶貴的時間。

若要執行本文的步驟，您需要[安裝適用於 Mac、Linux 和 Windows 的 Azure 命令列介面 (Azure CLI)](/cli/azure/install-azure-cli)。

本文會帶領您逐步完成封包擷取目前可用的不同管理工作。

- [**啟動封包擷取**](#start-a-packet-capture)
- [**停止封包擷取**](#stop-a-packet-capture)
- [**刪除封包擷取**](#delete-a-packet-capture)
- [**下載封包擷取**](#download-a-packet-capture)

## <a name="before-you-begin"></a>開始之前

本文假設您具有下列資源：

- 您想要用來建立封包擷取之區域中的網路監看員執行個體
- 已啟用封包擷取擴充功能的虛擬機器。

> [!IMPORTANT]
> 封包擷取需要在虛擬機器上執行代理程式。 代理程式會安裝為擴充功能。 如需 VM 擴充功能的指示，請瀏覽[虛擬機器擴充功能和功能](../virtual-machines/windows/extensions-features.md)。

## <a name="install-vm-extension"></a>安裝 VM 擴充功能

### <a name="step-1"></a>步驟 1

執行 `az vm extension set` Cmdlet 在客體虛擬機器上安裝封包擷取代理程式。

若為 Windows 虛擬機器：

```azurecli
az vm extension set --resource-group resourceGroupName --vm-name virtualMachineName --publisher Microsoft.Azure.NetworkWatcher --name NetworkWatcherAgentWindows --version 1.4
```

若為 Linux 虛擬機器：

```azurecli
az vm extension set --resource-group resourceGroupName --vm-name virtualMachineName --publisher Microsoft.Azure.NetworkWatcher --name NetworkWatcherAgentLinux--version 1.4
````

### <a name="step-2"></a>步驟 2

為了確定是否已安裝代理程式，請執行 `vm extension show` Cmdlet，並將資源群組和虛擬機器名稱傳遞給它。 檢查結果清單以確定代理程式已安裝。

```azurecli
az vm extension show --resource-group resourceGroupName --vm-name virtualMachineName --name NetworkWatcherAgentWindows
```

下列範例是執行 `az vm extension show` 所得回應的範例

```json
{
  "autoUpgradeMinorVersion": true,
  "forceUpdateTag": null,
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/extensions/NetworkWatcherAgentWindows",
  "instanceView": null,
  "location": "westcentralus",
  "name": "NetworkWatcherAgentWindows",
  "protectedSettings": null,
  "provisioningState": "Succeeded",
  "publisher": "Microsoft.Azure.NetworkWatcher",
  "resourceGroup": "{resourceGroupName}",
  "settings": null,
  "tags": null,
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "typeHandlerVersion": "1.4",
  "virtualMachineExtensionType": "NetworkWatcherAgentWindows"
}
```

## <a name="start-a-packet-capture"></a>啟動封包擷取

完成上述步驟之後，虛擬機器上便已安裝封包擷取代理程式。

### <a name="step-1"></a>步驟 1

下一步是擷取網路監看員執行個體。 網路監看員的名稱會傳遞給步驟 4 中的 `az network watcher show` Cmdlet。

```azurecli
az network watcher show --resource-group resourceGroup --name networkWatcherName
```

### <a name="step-2"></a>步驟 2

擷取儲存體帳戶。 此儲存體帳戶會用來儲存封包擷取檔案。

```azurecli
azure storage account list
```

### <a name="step-3"></a>步驟 3

可使用篩選器來限制封包擷取所儲存的資料。 下列範例會設定具有多個篩選器的封包擷取。  前三個篩選器只會收集從本機 IP 10.0.0.3 流往目的地連接埠 20、80 和 443 的連出 TCP 流量。  最後一個篩選器只會收集 UDP 流量。

```azurecli
az network watcher packet-capture create --resource-group {resourceGroupName} --vm {vmName} --name packetCaptureName --storage-account {storageAccountName} --filters "[{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"20\"},{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"80\"},{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"443\"},{\"protocol\":\"UDP\"}]"
```

下列範例是執行 `az network watcher packet-capture create` Cmdlet 後預期會得到的輸出。

```json
{
  "bytesToCapturePerPacket": 0,
  "etag": "W/\"b8cf3528-2e14-45cb-a7f3-5712ffb687ac\"",
  "filters": [
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "20"
    },
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "80"
    },
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "443"
    },
    {
      "localIpAddress": "",
      "localPort": "",
      "protocol": "UDP",
      "remoteIpAddress": "",
      "remotePort": ""
    }
  ],
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher_westcentralus/pa
cketCaptures/packetCaptureName",
  "name": "packetCaptureName",
  "provisioningState": "Succeeded",
  "resourceGroup": "NetworkWatcherRG",
  "storageLocation": {
    "filePath": null,
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/gwteststorage123abc",
    "storagePath": "https://gwteststorage123abc.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/{resourceGroupName}/p
roviders/microsoft.compute/virtualmachines/{vmName}/2017/05/25/packetcapture_16_22_34_630.cap"
  },
  "target": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}",
  "timeLimitInSeconds": 18000,
  "totalBytesPerSession": 1073741824
}
```

## <a name="get-a-packet-capture"></a>取得封包擷取

執行 `az network watcher packet-capture show-status` Cmdlet 以擷取目前正在執行或已完成之封包擷取的狀態。

```azurecli
az network watcher packet-capture show-status --name packetCaptureName --location {networkWatcherLocation}
```

下列範例是 `az network watcher packet-capture show-status` Cmdlet 的輸出。 下列範例是擷取停止 (Stopped) 的時候，其中 StopReason 為 TimeExceeded。 

```
{
  "additionalProperties": {
    "status": "Succeeded"
  },
  "captureStartTime": "2016-12-06T17:20:01.5671279Z",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher_westcentralus/pa
cketCaptures/packetCaptureName",
  "name": "packetCaptureName",
  "packetCaptureError": [],
  "packetCaptureStatus": "Stopped",
  "stopReason": "TimeExceeded"
}
```

## <a name="stop-a-packet-capture"></a>停止封包擷取

藉由執行 `az network watcher packet-capture stop` Cmdlet，如果擷取工作階段正在進行中，則會加以停止。

```azurecli
az network watcher packet-capture stop --name packetCaptureName --location westcentralus
```

> [!NOTE]
> 此 Cmdlet 若執行於目前正在執行的擷取工作階段或已停止的現有工作階段，則不會傳回任何回應。

## <a name="delete-a-packet-capture"></a>刪除封包擷取

```azurecli
az network watcher packet-capture delete --name packetCaptureName --location westcentralus
```

> [!NOTE]
> 刪除封包擷取不會刪除儲存體帳戶中的檔案。

## <a name="download-a-packet-capture"></a>下載封包擷取

封包擷取工作階段完成後，即可將擷取檔案上傳到 Blob 儲存體或 VM 上的本機檔案。 封包擷取的儲存位置會在建立工作階段時定義。 若要存取這些儲存至儲存體帳戶的擷取檔案，Microsoft Azure 儲存體總管是很便利的工具，您可以在這裡下載︰ http://storageexplorer.com/

如果指定了儲存體帳戶，封包擷取檔案便會儲存到儲存體帳戶的下列位置︰

```
https://{storageAccountName}.blob.core.windows.net/network-watcher-logs/subscriptions/{subscriptionId}/resourcegroups/{storageAccountResourceGroup}/providers/microsoft.compute/virtualmachines/{VMName}/{year}/{month}/{day}/packetCapture_{creationTime}.cap
```

## <a name="next-steps"></a>後續步驟

檢視[建立由警示觸發的封包擷取](network-watcher-alert-triggered-packet-capture.md)來了解如何透過虛擬機器警示自動化封包擷取

造訪[檢查 IP 流量驗證](diagnose-vm-network-traffic-filtering-problem.md)來得知 VM 是否允許特定流量流入或流出

<!-- Image references -->
