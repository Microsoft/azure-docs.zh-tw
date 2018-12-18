---
title: Azure Stack 整合式系統的網路整合考量 | Microsoft Docs
description: 了解您可以執行哪些工作，來為資料中心網路與多節點 Azure Stack 的整合做規劃。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2018
ms.author: jeffgilb
ms.reviewer: wamota
ms.openlocfilehash: 9b1eb6878dcafba68c230255f3b3f43e005421ab
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340183"
---
# <a name="network-connectivity"></a>網路連線
本文提供 Azure Stack 網路基礎架構資訊，可協助您決定如何以最佳方式將 Azure Stack 整合至現有的網路環境。 

> [!NOTE]
> 若要從 Azure Stack 解析外部 DNS 名稱 (例如 www.bing.com)，您必須提供 DNS 伺服器來轉送 DNS 要求。 如需有關 Azure Stack DNS 需求的詳細資訊，請參閱 [Azure Stack 資料中心整合 - DNS](azure-stack-integrate-dns.md)。

## <a name="physical-network-design"></a>實體網路設計
Azure Stack 解決方案需要有彈性且高可用性的實體基礎結構，以支援其作業和服務。 從 ToR 上行連結至界限交換器僅限用 SFP + 或 SFP28 媒體和 1 GB、10 GB 或 25 GB 的速度。 請洽詢您原始設備製造商 (OEM) 的硬體廠商，以了解可用性。 下圖是我們建議的設計：

![建議的 Azure Stack 網路設計](media/azure-stack-network/recommended-design.png)


## <a name="logical-networks"></a>邏輯網路
邏輯網路代表基礎實體網路基礎結構的抽象概念。 其用來組織及簡化主機、虛擬機器和服務的網路指派。 建立邏輯網路時會建立網路站台，以定義虛擬區域網路 (VLAN)、IP 子網路，以及與每個實體位置中邏輯網路相關聯的 IP 子網路/VLAN 配對。

下表顯示邏輯網路，以及您必須為其進行規劃的相關 IPv4 子網路範圍：

| 邏輯網路 | 說明 | 大小 | 
| -------- | ------------- | ------------ | 
| 公用 VIP | Azure Stack 從這個網路使用總計 31 個位址。 有 8 個公用 IP 位址用於小型 Azure Stack 服務集，其餘位址由租用戶虛擬機器使用。 如果您打算使用 App Service 和 SQL 資源提供者，則會再使用 7 個位址。 其餘 15 個 IP 則保留給未來的 Azure 服務。 | /26 (62 部主機) - /22 (1022 部主機)<br><br>建議 = / 24 (254 部主機) | 
| 交換器基礎結構 | 路由用途的點對點 IP 位址、專屬的交換器管理介面，及指派給參數的回送位址。 | /26 | 
| 基礎結構 | 用於通訊的 Azure Stack 內部元件。 | /24 |
| 私人 | 用於存放網路和私用 VIP。 | /24 | 
| BMC | 用來與實體主機上的 BMC 通訊。 | /26 | 
| | | |

## <a name="network-infrastructure"></a>網路基礎結構
Azure Stack 的網路基礎結構包含交換器上設定的數個邏輯網路。 下圖顯示這些邏輯網路，及其如何整合機架頂端 (TOR)、基礎板管理控制器 (BMC) 與界限 (客戶網路) 交換器。

![邏輯網路圖表和交換器連線](media/azure-stack-network/NetworkDiagram.png)

### <a name="bmc-network"></a>BMC 網路
此網路專門用來將所有基礎板管理控制器 (也稱為服務處理器，例如 iDRAC、iLO、iBMC 等) 連線至管理網路。 如果硬體生命週期主機 (HLH) 存在，它會位在此網路上，並可提供用於維護或監視硬體的 OEM 專用軟體。 

HLH 也會裝載部署 VM (DVM)。 DVM 會在 Azure Stack 部署期間使用，並且在部署完成時移除。 在已連線的部署案例中，DVM 需要網際網路存取權才能測試、驗證及存取多個元件。 這些元件可以在公司網路的內部和外部，例如 NTP、DNS 和 Azure。 如需有關連線需求的詳細資訊，請參閱 [Azure Stack 防火牆整合中的 NAT 區段](azure-stack-firewall.md#network-address-translation)。 

### <a name="private-network"></a>私人網路
此 /24 (254 個主機 IP) 網路由 Azure Stack 區域所私有 (不擴充到超出 Azure Stack 區域的界限交換器裝置)，而且分成兩個子網路：

- **儲存體網路**。 /25 (126 個主機 IP) 網路用於支援使用空間直接存取和伺服器訊息區 (SMB) 儲存體流量，以及虛擬機器即時移轉。 
- **內部虛擬 IP 網路**。 /25 網路專屬於適用軟體負載平衡器的僅內部用 VIP。

### <a name="azure-stack-infrastructure-network"></a>Azure Stack 基礎結構網路
此 /24 網路專屬於內部 Azure Stack 元件，以便這些元件通訊和交換本身的資料。 此子網路需要可路由傳送的 IP 位址，但會使用存取控制清單 (ACL) 來保持由解決方案所私有。 此網路在進行路由傳送時，不應超出邊界交換器，但大小等同於 /27 網路的小範圍除外，這當中一些服務在需要存取外部資源和/或網際網路時會利用 /27 網路。 

### <a name="public-infrastructure-network"></a>公用基礎結構網路
此 /27 網路是來自先前所述 Azure Stack 基礎結構子網路中的小範圍，它不需要公用 IP 位址，但確實需要透過 NAT 或 Transparent Proxy 提供的網際網路存取權。 此網路將會配置給緊急修復主控台系統 (ERCS)，ERCS VM 在向 Azure 註冊期間以及在基礎結構備份期間，需要存取網際網路。 ERCS VM 應該可路由到您的管理網路，以進行疑難排解。

### <a name="public-vip-network"></a>公用 VIP 網路
公用 VIP 網路會指派給 Azure Stack 中的網路控制器。 它不是交換器上的邏輯網路。 SLB 會針對租用戶工作負載使用位址集區並指派 /32 網路。 在交換器路由表上，這些 /32 IP 會公告為可透過 BGP 使用的路由。 此網路包含外部可存取的 IP 位址或公用 IP 位址。 Azure Stack 基礎結構保留公用 VIP 網路的前 31 個位址，而剩下的由租用戶 VM 使用。 此子網路上的網路大小範圍從最小 /26 (64 部主機) 到最大 /22 (1022 部主機)，我們建議您規劃 /24 網路。

### <a name="switch-infrastructure-network"></a>交換器基礎結構網路
此 /26 網路是一個子網路，其中包含可路由傳送的點對點 IP /30 (2 個主機 IP) 子網路和回送 (這是用於頻內交換器管理與 BGP 路由器識別碼的專用 /32 子網路)。 此 IP 位址範圍必須可在 Azure Stack 解決方案外部路由傳送至您的資料中心，而這些 IP 位址可以是私人或公用 IP。

### <a name="switch-management-network"></a>交換器管理網路
此 /29 (6 個主機 IP) 網路專門用來與交換器的管理連接埠連線。 其允許部署、管理和疑難排解的頻外存取。 它是從上述的交換器基礎結構網路計算而來。

## <a name="publish-azure-stack-services"></a>發佈 Azure Stack 服務
您必須將 Azure Stack 服務開放給 Azure Stack 的外部使用者使用。 Azure Stack 會設定其基礎結構角色的各種端點。 這些端點是來自公用 IP 位址集區的指派 VIP。 會針對外部 DNS 區域中的每個端點建立一個 DNS 項目，這是在部署時間所指定。 例如，系統會將 portal.*&lt;region>.&lt;fqdn>* 的 DNS 主機項目指派給使用者入口網站。

### <a name="ports-and-urls"></a>連接埠和 URL
若要讓外部網路使用 Azure Stack 服務 (例如入口網站、Azure Resource Manager、DNS 等)，您必須允許針對特定 URL、連接埠和通訊協定允許至這些端點的輸入流量。
 
在上行連結至傳統 Proxy 伺服器的 Transparent Proxy 部署中，您必須允許特定連接埠和 URL 進行[輸入](https://docs.microsoft.com/azure/azure-stack/azure-stack-integrate-endpoints#ports-and-protocols-inbound)和[出](https://docs.microsoft.com/azure/azure-stack/azure-stack-integrate-endpoints#ports-and-urls-outbound)通訊。 其中包括用於身分識別、Marketplace、修補和更新、註冊和使用狀況資料的連接埠和 URL。

## <a name="next-steps"></a>後續步驟
[邊界連線能力](azure-stack-border-connectivity.md)