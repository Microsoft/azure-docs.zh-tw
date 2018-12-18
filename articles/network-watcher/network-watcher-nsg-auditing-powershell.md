---
title: 使用 Azure 網路監看員安全性群組檢視自動化 NSG 稽核 | Microsoft Docs
description: 此頁面說明如何設定網路安全性群組的稽核
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 78a01bcf-74fe-402a-9812-285f3501f877
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 227ea446a75c167be27128b15de1d3c216e6856d
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34363371"
---
# <a name="automate-nsg-auditing-with-azure-network-watcher-security-group-view"></a>使用 Azure 網路監看員安全性群組檢視自動化 NSG 稽核

客戶通常面臨難以驗證其基礎結構的安全性狀態之挑戰。 這項挑戰在其 Azure 中的 VM 沒有不同。 請務必根據套用的網路安全性群組 (NSG) 規則，具有類似的安全性設定檔。 您現在可以使用安全性群組檢視，取得套用到 NSG 中 VM 的規則清單。 您可以每週頻率定義黃金 NSG 安全性設定檔並起始安全性群組檢視，並比較輸出與標準設定檔且建立報告。 如此一來，您可以輕鬆識別不符合指定安全性設定檔規定的所有 VM。

如果您不熟悉網路安全性群組，請參閱[網路安全性概觀](../virtual-network/security-overview.md)。

## <a name="before-you-begin"></a>開始之前

在此案例中，您可以比較已知的良好基準與針對虛擬機器傳回的安全性群組檢視結果。

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。 此案例也假設已有具有有效虛擬機器的資源群組可供使用。

## <a name="scenario"></a>案例

本文章涵蓋的案例會取得虛擬機器的安全性群組檢視。

在此案例中，您將會：

- 擷取已知的良好規則集
- 使用 REST API 擷取虛擬機器
- 取得虛擬機器的安全性群組檢視
- 評估回應

## <a name="retrieve-rule-set"></a>擷取規則集

此範例中的第一個步驟是使用現有的基準。 下列範例是使用此範例中用作為基準的 `Get-AzureRmNetworkSecurityGroup` cmdlet，擷取自現有網路安全性小組的某些 JSON。

```json
[
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "3389",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1000,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "default-allow-rdp",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/default-allow-rdp"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "111",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1010,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "MyRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/MyRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "112",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1020,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "My2ndRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/My2ndRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "5672",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Deny",
        "Priority":  1030,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "ThisRuleNeedsToStay",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/ThisRuleNeedsToStay"
    }
]
```

## <a name="convert-rule-set-to-powershell-objects"></a>將規則集轉換成 PowerShell 物件

在此步驟中，我們會讀取稍早使用預期要在此範例的網路安全性群組上之規則建立的 JSON 檔案。

```powershell
$nsgbaserules = Get-Content -Path C:\temp\testvm1-nsg.json | ConvertFrom-Json
```

## <a name="retrieve-network-watcher"></a>擷取網路監看員

下一步是擷取網路監看員執行個體。 `$networkWatcher` 變數會傳遞至 `AzureRmNetworkWatcherSecurityGroupView` Cmdlet。

```powershell
$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" } 
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName 
```

## <a name="get-a-vm"></a>取得 VM

必須有虛擬機器才能執行 `Get-AzureRmNetworkWatcherSecurityGroupView` Cmdlet。 下列範例會取得 VM 物件。

```powershell
$VM = Get-AzurermVM -ResourceGroupName "testrg" -Name "testvm1"
```

## <a name="retrieve-security-group-view"></a>擷取安全性群組檢視

下一步是擷取安全性群組檢視的結果。 此結果會與稍早顯示的「基準」JSON 相比較。

```powershell
$secgroup = Get-AzureRmNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="analyzing-the-results"></a>分析結果

回應會依網路介面群組。 所傳回的不同類型規則為有效及預設的安全性規則。 結果會依子網路或虛擬 NIC 上的套用方式進一步細分。

下列 PowerShell 指令碼會比較安全性群組檢視與 NSG 現有輸出的結果。 下列範例是結果與 `Compare-Object` cmdlet 比較方式的簡單範例。

```powershell
Compare-Object -ReferenceObject $nsgbaserules `
-DifferenceObject $secgroup.NetworkInterfaces[0].NetworkInterfaceSecurityRules `
-Property Name,Description,Protocol,SourcePortRange,DestinationPortRange,SourceAddressPrefix,DestinationAddressPrefix,Access,Priority,Direction
```

以下是結果範例。 您可以看到第一個規則集中的兩個規則未出現在比較中。

```
Name                     : My2ndRuleDoNotDelete
Description              : 
Protocol                 : *
SourcePortRange          : *
DestinationPortRange     : 112
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Allow
Priority                 : 1020
Direction                : Inbound
SideIndicator            : <=

Name                     : ThisRuleNeedsToStay
Description              : 
Protocol                 : TCP
SourcePortRange          : *
DestinationPortRange     : 5672
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Deny
Priority                 : 1030
Direction                : Inbound
SideIndicator            : <=
```

## <a name="next-steps"></a>後續步驟

如果設定已變更，請參閱[管理網路安全性群組](../virtual-network/manage-network-security-group.md)以追蹤有問題的網路安全性群組和安全性規則。













