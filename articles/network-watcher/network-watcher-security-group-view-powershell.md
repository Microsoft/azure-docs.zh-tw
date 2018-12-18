---
title: 使用 Azure 網路監看員安全性群組檢視分析網路安全性 - PowerShell | Microsoft Docs
description: 本文會說明如何使用 PowerShell，利用安全性群組檢視分析虛擬機器的安全性。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 04e76b49-6a1b-4d0f-9a9b-51cf2f4df5a2
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 9cc06e97730ac846e8aa42c2cee77dfe17be99bb
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39089641"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-powershell"></a>使用 PowerShell，利用安全性群組檢視分析虛擬機器的安全性

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [Azure CLI](network-watcher-security-group-view-cli.md)
> - [REST API](network-watcher-security-group-view-rest.md)

安全性群組檢視會傳回套用至虛擬機器之已設定且有效的網路安全性規則。 這項功能可用來稽核及診斷 VM 所設定的網路安全性群組和規則，以確保會正確允許或拒絕流量。 在本文中，我們會說明如何使用 PowerShell 來擷取虛擬機器所設定且有效的安全性規則

## <a name="before-you-begin"></a>開始之前

在此案例中，您會執行 `Get-AzureRmNetworkWatcherSecurityGroupView` Cmdlet 來擷取安全性規則資訊。

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。

## <a name="scenario"></a>案例

本文涵蓋的案例會擷取指定虛擬機器之已設定且有效的安全性規則。

## <a name="retrieve-network-watcher"></a>擷取網路監看員

第一步是擷取網路監看員執行個體。 此變數會傳遞至 `Get-AzureRmNetworkWatcherSecurityGroupView` Cmdlet。

```powershell
$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" }
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName
```

## <a name="get-a-vm"></a>取得 VM

必須有虛擬機器才能執行 `Get-AzureRmNetworkWatcherSecurityGroupView` Cmdlet。 下列範例會取得 VM 物件。

```powershell
$VM = Get-AzurermVM -ResourceGroupName testrg -Name testvm1
```

## <a name="retrieve-security-group-view"></a>擷取安全性群組檢視

下一步是擷取安全性群組檢視的結果。

```powershell
$secgroup = Get-AzureRmNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="viewing-the-results"></a>檢視結果

下列範例是所傳回結果的縮短回應。 結果顯示虛擬機器上所有有效且套用的安全性規則，並細分為 **NetworkInterfaceSecurityRules**、**DefaultSecurityRules**和 **EffectiveSecurityRules** 群組。

```
NetworkInterfaces : [
                      {
                        "NetworkInterfaceSecurityRules": [
                          {
                            "Name": "default-allow-rdp",
                            "Etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
                            "Id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/securityRules/default-allow-rdp",
                            "Protocol": "TCP",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "3389",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "Access": "Allow",
                            "Priority": 1000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "DefaultSecurityRules": [
                          {
                            "Name": "AllowVnetInBound",
                            "Id": "/subscriptions00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/defaultSecurityRules/",
                            "Description": "Allow inbound traffic from all VMs in VNET",
                            "Protocol": "*",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "*",
                            "SourceAddressPrefix": "VirtualNetwork",
                            "DestinationAddressPrefix": "VirtualNetwork",
                            "Access": "Allow",
                            "Priority": 65000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "EffectiveSecurityRules": [
                          {
                            "Name": "DefaultOutboundDenyAll",
                            "Protocol": "All",
                            "SourcePortRange": "0-65535",
                            "DestinationPortRange": "0-65535",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "ExpandedSourceAddressPrefix": [],
                            "ExpandedDestinationAddressPrefix": [],
                            "Access": "Deny",
                            "Priority": 65500,
                            "Direction": "Outbound"
                          },
                          ...
                        ]
                      }
                    ]
```

## <a name="next-steps"></a>後續步驟

請瀏覽[使用網路監看員稽核網路安全性群組 (NSG)](network-watcher-nsg-auditing-powershell.md) 以了解如何自動驗證網路安全性群組。


