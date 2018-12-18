---
title: 使用 Azure 網路監看員管理封包擷取 - PowerShell | Microsoft Docs
description: 此頁面說明如何使用 PowerShell 管理網路監看員的封包擷取功能
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 04d82085-c9ea-4ea1-b050-a3dd4960f3aa
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: d13c02696c9babae9fd04233ae7d2fdc75fca25f
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39090672"
---
# <a name="manage-packet-captures-with-azure-network-watcher-using-powershell"></a>使用 PowerShell 以 Azure 網路監看員管理封包擷取

> [!div class="op_single_selector"]
> - [Azure 入口網站](network-watcher-packet-capture-manage-portal.md)
> - [PowerShell](network-watcher-packet-capture-manage-powershell.md)
> - [Azure CLI](network-watcher-packet-capture-manage-cli.md)
> - [Azure REST API](network-watcher-packet-capture-manage-rest.md)

網路監看員封包擷取可讓您建立擷取工作階段來追蹤虛擬機器的流入和流出流量。 系統會為擷取工作階段提供篩選器，以確保您只會擷取到您想要的流量。 封包擷取有助於被動和主動地診斷網路異常。 其他用途包括收集網路統計資料、取得有關網路入侵的資訊，以及偵錯用戶端與伺服器間的通訊等等。 藉由能夠從遠端觸發封包擷取，這項功能可以減輕在所需機器上手動執行封包擷取的工作負擔，進而省下寶貴的時間。

本文會帶領您逐步完成封包擷取目前可用的不同管理工作。

- [**啟動封包擷取**](#start-a-packet-capture)
- [**停止封包擷取**](#stop-a-packet-capture)
- [**刪除封包擷取**](#delete-a-packet-capture)
- [**下載封包擷取**](#download-a-packet-capture)

## <a name="before-you-begin"></a>開始之前

本文假設您具有下列資源：

* 您想要用來建立封包擷取之區域中的網路監看員執行個體

* 已啟用封包擷取擴充功能的虛擬機器。

> [!IMPORTANT]
> 封包擷取需要虛擬機器擴充功能 `AzureNetworkWatcherExtension`。 若要在 Windows VM 上安裝擴充功能，請瀏覽[適用於 Windows 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/windows/extensions-nwa.md)，若要在 Linux VM 上安裝，則請瀏覽[適用於 Linux 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/linux/extensions-nwa.md)。

## <a name="install-vm-extension"></a>安裝 VM 擴充功能

### <a name="step-1"></a>步驟 1

```powershell
$VM = Get-AzureRmVM -ResourceGroupName testrg -Name VM1
```

### <a name="step-2"></a>步驟 2

下列範例會擷取執行 `Set-AzureRmVMExtension` cmdlet 所需的延伸資訊。 此 Cmdlet 會在客體虛擬機器上安裝封包擷取代理程式。

> [!NOTE]
> `Set-AzureRmVMExtension` cmdlet 可能需要幾分鐘的時間才能完成。

若為 Windows 虛擬機器：

```powershell
$AzureNetworkWatcherExtension = Get-AzureRmVMExtensionImage -Location WestCentralUS -PublisherName Microsoft.Azure.NetworkWatcher -Type NetworkWatcherAgentWindows -Version 1.4.13.0
$ExtensionName = "AzureNetworkWatcherExtension"
Set-AzureRmVMExtension -ResourceGroupName $VM.ResourceGroupName  -Location $VM.Location -VMName $VM.Name -Name $ExtensionName -Publisher $AzureNetworkWatcherExtension.PublisherName -ExtensionType $AzureNetworkWatcherExtension.Type -TypeHandlerVersion $AzureNetworkWatcherExtension.Version.Substring(0,3)
```

若為 Linux 虛擬機器：

```powershell
$AzureNetworkWatcherExtension = Get-AzureRmVMExtensionImage -Location WestCentralUS -PublisherName Microsoft.Azure.NetworkWatcher -Type NetworkWatcherAgentLinux -Version 1.4.13.0
$ExtensionName = "AzureNetworkWatcherExtension"
Set-AzureRmVMExtension -ResourceGroupName $VM.ResourceGroupName  -Location $VM.Location -VMName $VM.Name -Name $ExtensionName -Publisher $AzureNetworkWatcherExtension.PublisherName -ExtensionType $AzureNetworkWatcherExtension.Type -TypeHandlerVersion $AzureNetworkWatcherExtension.Version.Substring(0,3)
````

執行 `Set-AzureRmVMExtension` cmdlet 之後，下列範例會是成功回應。

```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   
```

### <a name="step-3"></a>步驟 3

為了確定是否已安裝代理程式，請執行 `Get-AzureRmVMExtension` Cmdlet，並將虛擬機器名稱和副檔名傳遞給它。

```powershell
Get-AzureRmVMExtension -ResourceGroupName $VM.ResourceGroupName  -VMName $VM.Name -Name $ExtensionName
```

下列範例是執行 `Get-AzureRmVMExtension` 所得回應的範例

```
ResourceGroupName       : testrg
VMName                  : testvm1
Name                    : AzureNetworkWatcherExtension
Location                : westcentralus
Etag                    : null
Publisher               : Microsoft.Azure.NetworkWatcher
ExtensionType           : NetworkWatcherAgentWindows
TypeHandlerVersion      : 1.4
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1/
                          extensions/AzureNetworkWatcherExtension
PublicSettings          : 
ProtectedSettings       : 
ProvisioningState       : Succeeded
Statuses                : 
SubStatuses             : 
AutoUpgradeMinorVersion : True
ForceUpdateTag          : 
```

## <a name="start-a-packet-capture"></a>啟動封包擷取

完成上述步驟之後，虛擬機器上便已安裝封包擷取代理程式。

### <a name="step-1"></a>步驟 1

下一步是擷取網路監看員執行個體。 此變數會在步驟 4 傳遞至 `New-AzureRmNetworkWatcherPacketCapture` Cmdlet。

```powershell
$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" }
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName  
```

### <a name="step-2"></a>步驟 2

擷取儲存體帳戶。 此儲存體帳戶會用來儲存封包擷取檔案。

```powershell
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName testrg -Name testrgsa123
```

### <a name="step-3"></a>步驟 3

可使用篩選器來限制封包擷取所儲存的資料。 下列範例會設定兩個篩選器。  一個篩選器只會收集從本機 IP 10.0.0.3 流往目的地連接埠 20、80 和 443 的連出 TCP 流量。  第二個篩選器只會收集 UDP 流量。

```powershell
$filter1 = New-AzureRmPacketCaptureFilterConfig -Protocol TCP -RemoteIPAddress "1.1.1.1-255.255.255" -LocalIPAddress "10.0.0.3" -LocalPort "1-65535" -RemotePort "20;80;443"
$filter2 = New-AzureRmPacketCaptureFilterConfig -Protocol UDP
```

> [!NOTE]
> 一個封包擷取可以定義多個篩選器。

### <a name="step-4"></a>步驟 4

執行 `New-AzureRmNetworkWatcherPacketCapture` cmdlet 可啟動封包擷取程序，並傳遞先前步驟中擷取的必要值。
```powershell

New-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $vm.Id -PacketCaptureName "PacketCaptureTest" -StorageAccountId $storageAccount.id -TimeLimitInSeconds 60 -Filter $filter1, $filter2
```

下列範例是執行 `New-AzureRmNetworkWatcherPacketCapture` Cmdlet 後預期會得到的輸出。

```
Name                    : PacketCaptureTest
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatcher
                          s/NetworkWatcher_westcentralus/packetCaptures/PacketCaptureTest
Etag                    : W/"3bf27278-8251-4651-9546-c7f369855e4e"
ProvisioningState       : Succeeded
Target                  : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1
BytesToCapturePerPacket : 0
TotalBytesPerSession    : 1073741824
TimeLimitInSeconds      : 60
StorageLocation         : {
                            "StorageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Storage/storageA
                          ccounts/examplestorage",
                            "StoragePath": "https://examplestorage.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-00000
                          0000000/resourcegroups/testrg/providers/microsoft.compute/virtualmachines/testvm1/2017/02/01/packetcapture_22_42_48_238.cap"
                          }
Filters                 : [
                            {
                              "Protocol": "TCP",
                              "RemoteIPAddress": "1.1.1.1-255.255.255",
                              "LocalIPAddress": "10.0.0.3",
                              "LocalPort": "1-65535",
                              "RemotePort": "20;80;443"
                            },
                            {
                              "Protocol": "UDP",
                              "RemoteIPAddress": "",
                              "LocalIPAddress": "",
                              "LocalPort": "",
                              "RemotePort": ""
                            }
                          ]


```

## <a name="get-a-packet-capture"></a>取得封包擷取

執行 `Get-AzureRmNetworkWatcherPacketCapture` Cmdlet 以擷取目前正在執行或已完成之封包擷取的狀態。

```powershell
Get-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

下列範例是 `Get-AzureRmNetworkWatcherPacketCapture` Cmdlet 的輸出。 下列範例是在擷取完成後。 PacketCaptureStatus 值為 Stopped，而 StopReason 為 TimeExceeded。 這個值說明封包擷取已順利完成，並執行了它的時間。
```
Name                    : PacketCaptureTest
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatcher
                          s/NetworkWatcher_westcentralus/packetCaptures/PacketCaptureTest
Etag                    : W/"4b9a81ed-dc63-472e-869e-96d7166ccb9b"
ProvisioningState       : Succeeded
Target                  : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1
BytesToCapturePerPacket : 0
TotalBytesPerSession    : 1073741824
TimeLimitInSeconds      : 60
StorageLocation         : {
                            "StorageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Storage/storageA
                          ccounts/examplestorage",
                            "StoragePath": "https://examplestorage.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-00000
                          0000000/resourcegroups/testrg/providers/microsoft.compute/virtualmachines/testvm1/2017/02/01/packetcapture_22_42_48_238.cap"
                          }
Filters                 : [
                            {
                              "Protocol": "TCP",
                              "RemoteIPAddress": "1.1.1.1-255.255.255",
                              "LocalIPAddress": "10.0.0.3",
                              "LocalPort": "1-65535",
                              "RemotePort": "20;80;443"
                            },
                            {
                              "Protocol": "UDP",
                              "RemoteIPAddress": "",
                              "LocalIPAddress": "",
                              "LocalPort": "",
                              "RemotePort": ""
                            }
                          ]
CaptureStartTime        : 2/1/2017 10:43:01 PM
PacketCaptureStatus     : Stopped
StopReason              : TimeExceeded
PacketCaptureError      : []
```

## <a name="stop-a-packet-capture"></a>停止封包擷取

藉由執行 `Stop-AzureRmNetworkWatcherPacketCapture` Cmdlet，如果擷取工作階段正在進行中，則會加以停止。

```powershell
Stop-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

> [!NOTE]
> 此 Cmdlet 若執行於目前正在執行的擷取工作階段或已停止的現有工作階段，則不會傳回任何回應。

## <a name="delete-a-packet-capture"></a>刪除封包擷取

```powershell
Remove-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

> [!NOTE]
> 刪除封包擷取不會刪除儲存體帳戶中的檔案。

## <a name="download-a-packet-capture"></a>下載封包擷取

封包擷取工作階段完成後，即可將擷取檔案上傳到 Blob 儲存體或 VM 上的本機檔案。 封包擷取的儲存位置會在建立工作階段時定義。 若要存取這些儲存至儲存體帳戶的擷取檔案，Microsoft Azure 儲存體總管是很便利的工具，您可以在這裡下載︰http://storageexplorer.com/

如果指定了儲存體帳戶，封包擷取檔案便會儲存到儲存體帳戶的下列位置︰

```
https://{storageAccountName}.blob.core.windows.net/network-watcher-logs/subscriptions/{subscriptionId}/resourcegroups/{storageAccountResourceGroup}/providers/microsoft.compute/virtualmachines/{VMName}/{year}/{month}/{day}/packetCapture_{creationTime}.cap
```

## <a name="next-steps"></a>後續步驟

檢視[建立由警示觸發的封包擷取](network-watcher-alert-triggered-packet-capture.md)來了解如何透過虛擬機器警示自動化封包擷取

造訪[檢查 IP 流量驗證](diagnose-vm-network-traffic-filtering-problem.md)來得知 VM 是否允許特定流量流入或流出

<!-- Image references -->














