---
title: Private Link-適用於 MariaDB 的 Azure 資料庫
description: 瞭解適用於 MariaDB 的 Azure 資料庫私用連結的運作方式。
author: mksuni
ms.author: sumuth
ms.service: jroth
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 12f52cd497d606fda6bbea9b54f467522373eeb7
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98665135"
---
# <a name="private-link-for-azure-database-for-mariadb"></a>適用於 MariaDB 的 Azure 資料庫的 Private Link

Private Link 可讓您為適用於 MariaDB 的 Azure 資料庫建立私人端點，因此會將 Azure 服務放在您的私人虛擬網路內 (VNet) 。 私人端點會公開私人 IP，您可用來連線到適用於 MariaDB 的 Azure 資料庫資料庫伺服器，就像 VNet 中的任何其他資源一樣。

如需支援 Private Link 功能的 PaaS 服務清單，請參閱 Private Link [檔](../private-link/index.yml)。 私人端點是特定 [VNet](../virtual-network/virtual-networks-overview.md) 和子網內的私人 IP 位址。

> [!NOTE]
> 私人連結功能僅適用于一般用途或記憶體優化定價層中的適用於 MariaDB 的 Azure 資料庫伺服器。 確定資料庫伺服器是在其中一個定價層。

## <a name="data-exfiltration-prevention"></a>預防資料外洩

適用於 MariaDB 的 Azure 資料庫中的資料（例如，資料庫管理員）可以從一個系統中提取資料，並將它移到組織外部的其他位置或系統。 例如，使用者將資料移至第三方所擁有的儲存體帳戶。

請考慮在連接至適用於 MariaDB 的 Azure 資料庫實例的 Azure VM 內執行適用于 mariadb 工作臺的使用者案例。 這個適用于 mariadb 實例位於「美國西部」資料中心。 下列範例顯示如何使用網路存取控制來限制適用於 MariaDB 的 Azure 資料庫上的公用端點存取。

* 藉由將 [允許 Azure 服務] 設為 [關閉]，以停用所有 Azure 服務流量，以透過公用端點適用於 MariaDB 的 Azure 資料庫。 請確定不允許任何 IP 位址或範圍透過 [防火牆規則](concepts-firewall-rules.md) 或 [虛擬網路服務端點](concepts-data-access-security-vnet.md)來存取伺服器。

* 使用 VM 的私人 IP 位址，只允許流向適用於 MariaDB 的 Azure 資料庫的流量。 如需詳細資訊，請參閱[服務端點](concepts-data-access-security-vnet.md)和 [VNet 防火牆規則](howto-manage-vnet-portal.md)的相關文章。

* 在 Azure VM 上，使用網路安全性群組來縮小傳出連線的範圍 (Nsg) 和服務標記，如下所示：

    * 指定 NSG 規則，以允許服務標記 = SQL 的流量。WestUs-僅允許連接到美國西部的適用於 MariaDB 的 Azure 資料庫
    * 使用優先順序較高的 (指定 NSG 規則，) 拒絕服務標籤 = SQL-拒絕適用于 mariadb 資料庫在所有區域中的連線</br></br>

在此設定結束時，Azure VM 只能連接到美國西部區域中的適用於 MariaDB 的 Azure 資料庫。 不過，連線不受限於單一適用於 MariaDB 的 Azure 資料庫。 VM 仍可連接到美國西部區域中的任何適用於 MariaDB 的 Azure 資料庫，包括不屬於訂用帳戶的資料庫。 雖然在上述案例中，我們已將資料外泄範圍縮減到特定區域，但我們尚未完全消除此問題。</br>

使用 Private Link，您現在可以設定網路存取控制（例如 Nsg），以限制對私人端點的存取。 這麼一來，個別的 Azure PaaS 資源就會對應到特定的私人端點。 惡意 insider 只能存取對應的 PaaS 資源 (例如適用於 MariaDB 的 Azure 資料庫) 和其他資源。

## <a name="on-premises-connectivity-over-private-peering"></a>透過私人對等互連的內部部署連線

當您從內部部署電腦連線至公用端點時，您的 IP 位址必須使用伺服器層級防火牆規則新增至以 IP 為基礎的防火牆。 雖然此模型可針對開發或測試工作負載來允許個別機器的存取，但難以在生產環境中進行管理。

使用 Private Link 時，您可以使用 [Express Route](https://azure.microsoft.com/services/expressroute/) (ER) 、私用對等互連或 [VPN](../vpn-gateway/index.yml)通道來啟用私人端點的跨單位存取。 他們接著可以停用透過公用端點的所有存取，而不使用以 IP 為基礎的防火牆。

> [!NOTE]
> 在某些案例中，適用於 MariaDB 的 Azure 資料庫和 VNet 子網路是位於不同的訂用帳戶。 在這些情況下，您必須確保下列設定：
> - 請確定這兩個訂用帳戶都已註冊 **DBforMariaDB** 資源提供者。 如需詳細資訊，請參閱 [resource-manager-registration][resource-manager-portal]

## <a name="configure-private-link-for-azure-database-for-mariadb"></a>設定適用於 MariaDB 的 Azure 資料庫的 Private Link

### <a name="creation-process"></a>建立程序

需要私人端點才能啟用 Private Link。 您可以使用下列操作指南來完成這項操作。

* [Azure 入口網站](howto-configure-privatelink-portal.md)
* [CLI](howto-configure-privatelink-cli.md)

### <a name="approval-process"></a>核准流程

一旦網路系統管理員 (PE) 建立私人端點，系統管理員就可以管理 (PEC) 到適用於 MariaDB 的 Azure 資料庫的私人端點連接。 網路系統管理員與 DBA 之間的職責區分有助於管理適用於 MariaDB 的 Azure 資料庫連線。 

* 流覽至 Azure 入口網站中的適用於 MariaDB 的 Azure 資料庫 server 資源。 
    * 選取左窗格中的私人端點連線
    * 顯示 (Pec 的所有私人端點連接清單) 
    * 已建立對應的私人端點 (PE) 

![選取私人端點入口網站](media/concepts-data-access-and-security-private-link/select-private-link-portal.png)

* 從清單中選取個別的 PEC。

![選取要等待核准的私人端點](media/concepts-data-access-and-security-private-link/select-private-link.png)

* 適用于 mariadb 伺服器系統管理員可以選擇核准或拒絕 PEC，並選擇性地新增簡短的文字回應。

![選取私人端點訊息](media/concepts-data-access-and-security-private-link/select-private-link-message.png)

* 核准或拒絕之後，清單會反映適當的狀態以及回應文字

![選取私人端點的最終狀態](media/concepts-data-access-and-security-private-link/show-private-link-approved-connection.png)

## <a name="use-cases-of-private-link-for-azure-database-for-mariadb"></a>適用於 MariaDB 的 Azure 資料庫 Private Link 的使用案例

用戶端可以從相同的 VNet 連線到私人端點、在相同區域中對等互連 VNet，或透過跨區域的 VNet 對 VNet 連線來連接。 此外，用戶端可以使用 ExpressRoute、私人對等互連或 VPN 通道從內部部署環境進行連線。 以下是一個簡化的圖表，其中顯示常見的使用案例。

![選取私人端點總覽](media/concepts-data-access-and-security-private-link/show-private-link-overview.png)

### <a name="connecting-from-an-azure-vm-in-peered-virtual-network-vnet"></a>從對等互連虛擬網路 (VNet) 中的 Azure VM 進行連線
設定 [vnet 對等互連](../virtual-network/tutorial-connect-virtual-networks-powershell.md) ，以從對等互連 VNet 中的 Azure VM 建立對適用於 MariaDB 的 Azure 資料庫的連線能力。

### <a name="connecting-from-an-azure-vm-in-vnet-to-vnet-environment"></a>從 VNet 對 VNet 環境中的 Azure VM 進行連線
設定 [VNet 對 VNET VPN 閘道聯](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) 機，以從不同區域或訂用帳戶中的 Azure VM 建立對適用於 MariaDB 的 Azure 資料庫的連線能力。

### <a name="connecting-from-an-on-premises-environment-over-vpn"></a>透過 VPN 從內部部署環境連線
若要建立從內部部署環境到適用於 MariaDB 的 Azure 資料庫的連接，請選擇並執行其中一個選項：

* [點對站連線](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)
* [站對站 VPN 連線](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [ExpressRoute 線路](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

## <a name="private-link-combined-with-firewall-rules"></a>Private Link 搭配防火牆規則

當搭配防火牆規則使用 Private Link 時，可能會發生下列情況和結果：

* 如果您未設定任何防火牆規則，則根據預設，將無法存取適用於 MariaDB 的 Azure 資料庫的流量。

* 如果您設定公用流量或服務端點並建立私人端點，則不同類型之傳入流量會由對應類型的防火牆規則所授權。

* 如果您未設定任何公用流量或服務端點，而且您建立私人端點，則適用於 MariaDB 的 Azure 資料庫只能透過私人端點來存取。 如果您未設定公用流量或服務端點，則在拒絕或刪除所有已核准的私人端點之後，將無法存取適用於 MariaDB 的 Azure 資料庫的流量。

## <a name="deny-public-access-for-azure-database-for-mariadb"></a>拒絕適用於 MariaDB 的 Azure 資料庫的公用存取

如果您想要完全依賴私人端點來存取其適用於 MariaDB 的 Azure 資料庫，您可以在資料庫伺服器上設定 **拒絕公用網路存取** 設定，以停用 [防火牆規則](concepts-firewall-rules.md)和 [VNet 服務) 端點](concepts-data-access-security-vnet.md) (的所有公用端點設定。 

當此設定設為 *[是]* 時，只允許透過私人端點連線到您的適用於 MariaDB 的 Azure 資料庫。 當此設定設為 [ *否*] 時，用戶端可以根據您的防火牆或 VNet 服務端點設定來連線到您的適用於 MariaDB 的 Azure 資料庫。 此外，一旦設定私人網路的值之後，客戶就無法新增及/或更新現有的「防火牆規則」和「VNet 服務端點規則」。

> [!Note]
> 這項功能適用于所有的 Azure 區域，其中適用於 PostgreSQL 的 Azure 資料庫單一伺服器支援一般用途和記憶體優化定價層。
>
> 此設定不會影響您適用於 MariaDB 的 Azure 資料庫的 SSL 和 TLS 設定。

若要瞭解如何從 Azure 入口網站設定您適用於 MariaDB 的 Azure 資料庫的 **拒絕公用網路存取** ，請參閱 [如何設定拒絕公用網路存取](howto-deny-public-network-access.md)。

## <a name="next-steps"></a>下一步

若要深入瞭解適用於 MariaDB 的 Azure 資料庫安全性功能，請參閱下列文章：

* 若要設定適用於 MariaDB 的 Azure 資料庫的防火牆，請參閱 [防火牆支援](concepts-firewall-rules.md)。

* 若要瞭解如何為您的適用於 MariaDB 的 Azure 資料庫設定虛擬網路服務端點，請參閱 [設定從虛擬網路存取](concepts-data-access-security-vnet.md)。

* 如需適用於 MariaDB 的 Azure 資料庫連接的總覽，請參閱 [適用於 MariaDB 的 Azure 資料庫連線架構](concepts-connectivity-architecture.md)

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
