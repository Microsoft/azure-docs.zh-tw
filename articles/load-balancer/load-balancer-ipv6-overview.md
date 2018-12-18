---
title: Azure Load Balancer 的 IPv6 概觀 | Microsoft Docs
description: 了解 Azure Load Balancer 和負載平衡 VM 的 IPv6 支援。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
keywords: ipv6, azure load balancer, 雙重堆疊, 公用 ip, 原生 ipv6, 行動, iot
ms.assetid: 6a1d583f-a305-40fd-a94b-fa42e1943bbb
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/24/2018
ms.author: kumud
ms.openlocfilehash: fc7cda30beaea8a9e15794c9330832e9ca651eb7
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918683"
---
# <a name="overview-of-ipv6-for-azure-load-balancer"></a>Azure Load Balancer 的 IPv6 概觀


>[!NOTE] 
>Azure Load Balancer 支援兩種不同類型：基本和標準。 本文討論基本 Load Balancer。 如需標準 Load Balancer 的詳細資訊，請參閱[標準 Load Balancer 概觀](load-balancer-standard-overview.md)。

網際網路面向的負載平衡器可以部署 IPv6 位址。 除了 IPv4 連線功能，這還具有下列功能︰

* 原生端對端 IPv6 連線能力 - 在公用網際網路用戶端和 Azure 虛擬機器 (VM) 之間透過負載平衡器連接。
* 原生端對端 IPv6 輸出連線能力 - 在 VM 和公用網際網路上已啟用 IPv6 的用戶端之間。

下圖說明 Azure Load Balancer 的 IPv6 功能。

![配置有 IPv6 的 Azure Load Balancer](./media/load-balancer-ipv6-overview/load-balancer-ipv6.png)

部署之後，已啟用 IPv4 或 IPv6 的網際網路用戶端可以與網際網路面向的 Azure Load Balancer 公用 IPv4 或 IPv6 位址 (或主機名稱) 進行通訊。 負載平衡器會使用網路位址轉譯 (NAT) 將 IPv6 封包傳送至 VM 的私人 IPv6 位址。 IPv6 網際網路用戶端無法與 VM 的 IPv6 位址直接通訊。

## <a name="features"></a>特性

透過 Azure Resource Manager部署的原生 IPv6 支援提供︰

1. 為網際網路上 IPv6 用戶端提供負載平衡的 IPv6 服務
2. 在 VM上提供原生 IPv6 和 IPv4 端點 (「雙重堆疊」)
3. 由輸入和輸出起始的原生 IPv6 連線
4. 支援的通訊協定 - 如 TCP、UDP、HTTP(S) - 提供完整的服務架構

## <a name="benefits"></a>優點

這項功能具有下列主要優點：

* 符合政府法規，要求新的應用程式可以被僅有 IPv6 的用戶端存取
* 可讓行動和物聯網 (IOT) 開發人員使用雙重堆疊 (IPv4 + IPv6) 的 Azure 虛擬機器處理成長的行動和 IOT 市場

## <a name="details-and-limitations"></a>詳細資料和限制

詳細資料

* Azure DNS 服務包含 IPV4 A 和 IPv6 AAAA 名稱記錄，以及兩種負載平衡器記錄的回應。 用戶端選擇要與哪一個位址 (IPv4 或 IPv6) 通訊。
* 當 VM 起始連接至公用網際網路 IPv6 連線裝置的連線時，VM 的來源 IPv6 位址是經網路位址轉譯 (NAT) 後的負載平衡器公用 IPv6 位址。
* 執行 Linux 作業系統的 VM 必須設定為透過 DHCP 接收 IPv6 IP 位址。 Azure 資源庫中許多 Linux 映像已設定為支援 IPv6，不需要修改。 如需詳細資訊，請參閱 [設定 Linux VM 的 DHCPv6](load-balancer-ipv6-for-linux.md)
* 如果您選擇在負載平衡器使用健全狀況探查，請建立 IPv4 探查，並同時用在 IPv4 和 IPv6 端點。 如果 VM 上的服務中斷，IPv4 和 IPv6 端點會從輪替中拿掉。

限制

* 您無法在 Azure 入口網站中新增 IPv6 負載平衡規則。 只能透過範本、CLI、PowerShell 建立規則。
* 您可能不會將現有 VM 升級為使用 IPv6 位址。 您必須部署新的 VM。
* 可將單一 IPv6 位址指派給每個 VM 中的單一網路介面。
* 公用 IPv6 位址不能指派給 VM， 只能指派給負載平衡器。
* 您無法針對您的公用 IPv6 位址設定反向 DNS 查閱。
* 使用 IPv6 位址的 VM 不可是 Azure 雲端服務的成員。 它們不可以連接到 Azure 虛擬網路 (VNet)，也不能透過其 IPv4 位址與彼此通訊。
* 私人 IPv6 位址可部署到資源群組中的個別 VM，但無法透過擴充集部署到資源群組。
* Azure VM 無法透過 IPv6 與其他 VM、其他 Azure 服務、或內部部署裝置連線， 只能透過 IPv6 與 Azure Load Balancer 通訊。 不過，它們可以使用 IPv4 與這些其他資源通訊。
* 雙重堆疊 (IPv4 + IPv6) 部署支援 IPv4 的網路安全性群組 (NSG) 保護。 NSG 不適用於 IPv6 端點。
* VM 上的 IPv6 端點不會直接向網際網路公開， 而是擺在負載平衡器後。 透過 IPv6 存取只能存取負載平衡器規則中指定的連接埠。
* **目前不支援**變更 IPv6 的 IdleTimeout 參數。 預設為四分鐘。
* **目前不支援**變更 IPv6 的 loadDistributionMethod 參數。
* **目前不支援**保留的 IPv6 IP (其中 IPAllocationMethod = 靜態)。
* 不支援 NAT64 (IPv6 至 IPv4 的轉譯)。

## <a name="next-steps"></a>後續步驟

了解如何部署配置有 IPv6 的負載平衡器。

* [依區域的 IPv6 可用性](https://go.microsoft.com/fwlink/?linkid=828357)
* [使用範本部署配置有 IPv6 的負載平衡器](load-balancer-ipv6-internet-template.md)
* [使用 Azure PowerShell 部署配置有 IPv6 的負載平衡器](load-balancer-ipv6-internet-ps.md)
* [使用 Azure CLI 部署配置有 IPv6 的負載平衡器](load-balancer-ipv6-internet-cli.md)
