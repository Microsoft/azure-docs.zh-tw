---
title: 利用 Azure 網路監看員進行連線疑難排解 - Azure 入口網站 | Microsoft Docs
description: 了解如何運用 Azure 入口網站，來使用 Azure 網路監看員的連線疑難排解功能。
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
ms.date: 08/03/2017
ms.author: jdial
ms.openlocfilehash: 21e004e12a5111eb0e5fc7764c1e07fcb68c447d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986195"
---
# <a name="troubleshoot-connections-with-azure-network-watcher-using-the-azure-portal"></a>使用 Azure 入口網站利用 Azure 網路監看員進行連線疑難排解

> [!div class="op_single_selector"]
> - [入口網站](network-watcher-connectivity-portal.md)
> - [PowerShell](network-watcher-connectivity-powershell.md)
> - [Azure CLI](network-watcher-connectivity-cli.md)
> - [Azure REST API](network-watcher-connectivity-rest.md)

了解如何使用連線疑難排解，來確認是否可以建立從虛擬機器到指定端點的直接 TCP 連線。

## <a name="before-you-begin"></a>開始之前

本文假設您具有下列資源：

* 您想要進行連線疑難排解之區域中的網路監看員執行個體。
* 要用來進行連線疑難排解的虛擬機器。

> [!IMPORTANT]
> 若要進行連線移難排解，您從中進行移難排解的虛擬機器需要安裝 `AzureNetworkWatcherExtension` 虛擬機器擴充功能。 若要在 Windows VM 上安裝擴充功能，請瀏覽[適用於 Windows 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/windows/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)，若要在 Linux VM 上安裝，則請瀏覽[適用於 Linux 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/linux/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。 目的地端點上不需要擴充功能。

## <a name="check-connectivity-to-a-virtual-machine"></a>檢查與虛擬機器的連線

這個範例會檢查透過連接埠 80 的目的地虛擬機器連線。

瀏覽至您的網路監看員，然後按一下 [連線疑難排解]。 選取要作為連線能力檢查起點的虛擬機器。 在 [目的地] 區段中選擇 [選取虛擬機器]，然後選擇要測試的正確虛擬機器和連接埠。

按一下 [檢查] 之後，系統就會檢查指定連接埠上虛擬機器之間的連線能力。 在範例中，目的地 VM 無法連線，因此會顯示躍點清單。

![查看虛擬機器的連線能力結果][1]

## <a name="check-remote-endpoint-connectivity"></a>檢查遠端端點的連線能力

若要檢查遠端端點的連線能力和延遲，請在 [目的地] 區段中選擇 [手動指定] 選項按鈕，輸入 URL 和連接埠，然後按一下 [檢查]。  這會用於網站與儲存體端點之類的遠端端點。

![查看網站的連線能力結果][2]

## <a name="next-steps"></a>後續步驟

檢視[建立由警示觸發的封包擷取](network-watcher-alert-triggered-packet-capture.md)來了解如何透過虛擬機器警示自動化封包擷取

造訪[檢查 IP 流量驗證](diagnose-vm-network-traffic-filtering-problem.md)來得知 VM 是否允許特定流量流入或流出

[1]: ./media/network-watcher-connectivity-portal/figure1.png
[2]: ./media/network-watcher-connectivity-portal/figure2.png