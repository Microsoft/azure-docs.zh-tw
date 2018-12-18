---
title: Azure 網路監看員連線疑難排解簡介 | Microsoft Docs
description: 本頁提供網路監看員連線疑難排解功能的概觀
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/11/2017
ms.author: jdial
ms.openlocfilehash: 0268c7e54aa82df12243f98fd72de836fbc82070
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30833994"
---
# <a name="introduction-to-connection-troubleshoot-in-azure-network-watcher"></a>Azure 網路監看員中連線疑難排解的簡介

網路監看員的連線疑難排解功能提供功能，來檢查從虛擬機器到虛擬機器 (VM) 的直接 TCP 連線、完整網域名稱 (FQDN)、URI 或 IPv4 位址。 網路案例很複雜，它們在實作時使用網路安全性群組、防火牆、使用者定義的路由和 Azure 所提供的資源。 複雜的設定使得針對連線問題進行疑難排解成為一項挑戰。 網路監看員有助於減少尋找並偵測連線問題的時間量。 傳回的結果可以提供對於連線問題是因為平台還是使用者設定問題的深入了解。 連線可以使用 [PowerShell](network-watcher-connectivity-powershell.md)、[Azure CLI](network-watcher-connectivity-cli.md) 和 [REST API](network-watcher-connectivity-rest.md) 來檢查。

> [!IMPORTANT]
> 若要進行連線移難排解，您從中進行移難排解的虛擬機器需要安裝 `AzureNetworkWatcherExtension` 虛擬機器擴充功能。 若要在 Windows VM 上安裝擴充功能，請瀏覽[適用於 Windows 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/windows/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)，若要在 Linux VM 上安裝，則請瀏覽[適用於 Linux 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/linux/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。 目的地端點上不需要擴充功能。

## <a name="response"></a>Response

下表顯示當連線疑難排解完成執行時所傳回的屬性。

|屬性  |說明  |
|---------|---------|
|ConnectionStatus     | 連線檢查的狀態。 可能的結果是 **Reachable** 和 **Unreachable**。        |
|AvgLatencyInMs     | 連線檢查期間的平均延遲，以毫秒為單位。 (只有在檢查狀態為可連線時顯示)        |
|MinLatencyInMs     | 連線檢查期間的最短延遲，以毫秒為單位。 (只有在檢查狀態為可連線時顯示)        |
|MaxLatencyInMs     | 連線檢查期間的最長延遲，以毫秒為單位。 (只有在檢查狀態為可連線時顯示)        |
|ProbesSent     | 檢查期間傳送的探查數目。 最大值是 100。        |
|ProbesFailed     | 檢查期間失敗的探查數目。 最大值是 100。        |
|Hops     | 從來源到目的地的逐躍點路徑。        |
|Hops[].Type     | 資源類型。 可能的值為 **Source**、**VirtualAppliance**、**VnetLocal** 和 **Internet**。        |
|Hops[].Id | 躍點的唯一識別碼。|
|Hops[].Address | 躍點的 IP 位址。|
|Hops[].ResourceId | 如果躍點是一項 Azure 資源，則為躍點的資源識別碼。 如果是網際網路資源，ResourceID 是 **Internet**。 |
|Hops[].NextHopIds | 下個採取躍點的唯一識別碼。|
|Hops[].Issues | 在該躍點檢查期間遇到之問題的集合。 如果沒有任何問題，則值會是空白。|
|Hops[].Issues[].Origin | 在目前躍點，發生問題之處。 可能的值包括：<br/> **Inbound** - 問題是在從上一個躍點到目前躍點的連結上<br/>**Outbound** - 問題是在從目前躍點到下一個躍點的連結上<br/>**Local** - 問題位於目前的躍點。|
|Hops[].Issues[].Severity | 偵測到之問題的嚴重性。 可能的值為 **Error** 和 **Warning**。 |
|Hops[].Issues[].Type |找到的問題類型。 可能的值包括： <br/>**CPU**<br/>**記憶體**<br/>**GuestFirewall**<br/>**DnsResolution**<br/>**NetworkSecurityRule**<br/>**UserDefinedRoute** |
|Hops[].Issues[].Context |找到之問題的相關詳細資料。|
|Hops[].Issues[].Context[].key |傳回之索引鍵值組的索引鍵。|
|Hops[].Issues[].Context[].value |傳回之索引鍵值組的值。|

以下是在躍點上發現問題的範例。

```json
"Issues": [
    {
        "Origin": "Outbound",
        "Severity": "Error",
        "Type": "NetworkSecurityRule",
        "Context": [
            {
                "key": "RuleName",
                "value": "UserRule_Port80"
            }
        ]
    }
]
```
## <a name="fault-types"></a>錯誤類型

連線疑難排解傳回連線的相關錯誤類型。 下表提供一份目前傳回錯誤類型的清單。

|類型  |說明  |
|---------|---------|
|CPU     | 高 CPU 使用率。       |
|記憶體     | 高記憶體使用率。       |
|GuestFirewall     | 由於虛擬機器防火牆設定，因此封鎖流量。        |
|DNSResolution     | 目的地位址的 DNS 解析失敗。        |
|NetworkSecurityRule    | NSG 規則封鎖流量 (傳回規則)        |
|UserDefinedRoute|因為使用者定義或系統路由而卸除流量。 |

### <a name="next-steps"></a>後續步驟

了解如何使用 [Azure 入口網站](network-watcher-connectivity-portal.md)、[PowerShell](network-watcher-connectivity-powershell.md)、[Azure CLI](network-watcher-connectivity-cli.md) 或 [REST API](network-watcher-connectivity-rest.md) 進行連線疑難排解。