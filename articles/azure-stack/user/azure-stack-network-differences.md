---
title: Azure Stack 網路服務差異與注意事項 | Microsoft Docs
description: 使用 Azure Stack 中的網路服務時，瞭解有關差異和注意事項。
services: azure-stack
keywords: ''
author: mattbriggs
manager: femila
ms.author: brenduns
ms.date: 08/02/2018
ms.topic: article
ms.service: azure-stack
ms.reviewer: scottnap
ms.openlocfilehash: 50fe3c0c7fda745047c71afb8eedf7fa8806c4ec
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41946581"
---
# <a name="considerations-for-azure-stack-networking"></a>Azure Stack 網路服務的注意事項

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 網路服務有許多 Azure 網路服務所提供的功能。 不過，在部署 Azure Stack 網路之前，您應該了解一些主要差異。

本文提供 Azure Stack 網路服務及其功能的獨特考量概觀。 若要深入了解 Azure Stack 與 Azure 之間的大致差異，請參閱[主要考量](azure-stack-considerations.md)文章。

## <a name="cheat-sheet-networking-differences"></a>速查表：網路服務差異

| 服務 | 功能 | Azure (全域) | Azure Stack |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| DNS | 多租用戶 DNS | 支援 | 尚不支援 |
|  | DNS AAAA 記錄 | 支援 | 不支援 |
|  | 每一訂用帳戶的 DNS 區域 | 100 (預設值)<br>可在要求時增加。 | 100 |
|  | 每一區域的 DNS 記錄集 | 5000 (預設值)<br>可在要求時增加。 | 5000 |
|  | 區域委派的名稱伺服器 | Azure 為建立的每個使用者 (租用戶) 區域提供四部名稱伺服器。 | Azure Stack 為建立的每個使用者 (租用戶) 區域提供兩部名稱伺服器。 |
| 虛擬網路 | 虛擬網路對等互連 | 透過 Azure 骨幹網路來連線同一區域中兩個虛擬網路。 | 尚不支援 |
|  | IPv6 位址 | 您可以指派 IPv6 位址作為[網路介面設定](https://docs.microsoft.com/azure/virtual-network/virtual-network-network-interface-addresses#ip-address-versions)的一部分。 | 僅支援 IPv4。 |
|  | DDoS 保護計劃 | 支援 | 尚不支援。 |
|  | 擴展集 IP 設定 | 支援 | 尚不支援。 |
|  | 私用存取服務 (子網路) | 支援 | 尚不支援。 |
|  | 服務端點 | 針對與 Azure 服務的內部 (非網際網路) 連線提供支援。 | 尚不支援。 |
| 僅支援 IPv4。 | 服務端點原則 | 支援 | 尚不支援。 |
|  | 服務通道 | 支援 | 尚不支援。  |
| 網路安全性群組 | 增強型安全性規則 | 支援 | 尚不支援。 |
|  | 有效的安全性規則 | 支援 | 尚不支援。 |
|  | 應用程式安全性群組 | 支援 | 尚不支援。 |
| 虛擬網路閘道 | 點對站 VPN 閘道 | 支援 | 尚不支援。 |
|  | Vnet 對 Vnet 閘道 | 支援 | 尚不支援。 |
|  | 虛擬網路閘道類型 | Azure 支援 VPN<br> ExpressRoute <br> Hyper Net | Azure Stack 目前僅支援 VPN 類型。 |
|  | VPN 閘道 SKU | 支援基本、GW1、GW2、GW3、標準的高效能、超高效能。 | 支援基本、標準和高效能 SKU。 |
|  | VPN 類型 | Azure 支援「原則式」和「路由式」。 | Azure Stack 僅支援「路由式」。 |
|  | BGP 設定 | Azure 支援設定「BGP 對等互連位址」和「對等互連權重」。 | 在 Azure Stack 中會自動設定「BGP 對等互連位址」和「對等互連權重」。 使用者無法使用自己的值來進行這些設定。 |
|  | 預設閘道站台 | Azure 支援為強制通道設定預設站台。 | 尚不支援。 |
|  | 調整閘道大小 | Azure 支援在部署後調整閘道大小。 | 不支援調整大小。 |
|  | 主動/主動設定 | 支援 | 尚不支援。 |
|  | IKE/IPSec 原則 | Azure 支援自訂 IPSec 原則設定。 | 尚不支援。 |
|  | UsePolicyBasedTrafficSelectors | Azure 支援將原則式流量選取器與路由式閘道連線搭配使用。 | 尚不支援。 |
| 負載平衡器 | SKU | 支援「基本負載平衡器」和「標準負載平衡器」 | 僅支援「基本負載平衡器」。  不支援 SKU 屬性。 |
|  | 區域 | 支援「可用性區域」。 | 尚不支援 |
|  | 服務端點的輸入 NAT 規則支援 | Azure 支援指定「輸入 NAT」規則的「服務端點」。 | Azure Stack 尚不支援「服務端點」，因此無法指定這些項目。 |
|  | 通訊協定 | Azure 支援指定 GRE 或 ESP。 | Azure Stack 不支援「通訊協定類別」。 |
| 公用 IP 位址 | 公用 IP 位址版本 | Azure 支援 IPv6 和 IPv4 | 僅支援 IPv4。 |
| 網路介面 | 取得有效的路由表 | 支援 | 尚不支援。 |
|  | 取得有效的 ACL | 支援 | 尚不支援。 |
|  | 啟用加速網路 | 支援 | 尚不支援。 |
|  | IP 轉送 | 預設為停用。  可以啟用。 | 不支援切換此設定。  預設為 [開啟]。 |
|  | 每一介面有多個 IP 設定 | 支援 | 尚不支援。 |
|  | 應用程式安全性群組 | 支援 | 尚不支援。 |
|  | 內部 DNS 名稱標籤 | 支援 | 尚不支援。 |
|  | 私人 IP 位址版本 | 支援 IPv4 和 IPv6。 | 僅支援 IPv4。 |
|  | 主要 IP 設定 | 支援。 識別介面上的主要 IP 設定。 | 尚不支援。 |
| 網路監看員 | 網路監看員租用戶網路監視功能 | 支援 | 尚不支援。 |
| CDN | 內容傳遞網路設定檔 | 支援 | 尚不支援。 |
| 應用程式閘道 | 第 7 層負載平衡 | 支援 | 尚不支援。 |
| 流量管理員 | 針對最佳應用程式效能和可靠性路由連入流量。 | 支援 | 尚不支援。 |
| ExpressRoute | 設定從內部部署基礎結構或共置設備到 Microsoft 雲端服務的快速私人連線。 | 支援 | 將 Azure Stack 連接到 Express Route 電路的支援。 |

## <a name="next-steps"></a>後續步驟

[Azure Stack 中的 DNS](azure-stack-dns.md)