---
title: 設定在使用 Site Recovery 容錯移轉後用來連線至次要內部部署網站的 IP 定址 | Microsoft Docs
description: 說明如何設定在使用 Azure Site Recovery 容錯移轉後用來連線至次要內部部署網站 VM 的 IP 定址。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: rayne
ms.openlocfilehash: dc28611f4b27d263e19f76ce4313f8d531a8cba2
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49076996"
---
# <a name="set-up-ip-addressing-to-connect-to-a-secondary-on-premises-site-after-failover"></a>設定在容錯移轉後用來連線至次要內部部署網站的 IP 定址

在您將 System Center Virtual Machine Manager (VMM) 雲端中的 Hyper-V VM 容錯移轉至次要網站後，您必須能夠連線至複本 VM。 本文可協助您執行這項操作。 

## <a name="connection-options"></a>連線選項

容錯移轉之後，有幾種方式可處理複本 VM 的 IP 定址： 

- **在容錯移轉後保留相同的 IP 位址**：在此情況下，複寫的 VM 會具有與主要 VM 相同的 IP 位址。 這可以簡化容錯移轉後的網路相關問題，但需執行某些基礎結構工作。
- **在容錯移轉後使用不同的 IP 位址**：在此情況下，VM 在容錯移轉後會取得新的 IP 位址。 
 

## <a name="retain-the-ip-address"></a>保留 IP 位址

如果您想要保留主要網站的 IP 位址，在容錯移轉至次要網站後，您可以：

- 在主要和次要網站之間部署延伸的子網路。
- 執行從主要到次要網站的完整子網路容錯移轉。 您必須更新路由以指出 IP 位址的新位置。


### <a name="deploy-a-stretched-subnet"></a>部署延伸的子網路

在延伸的組態中，主要網站和次要網站可同時使用子網路。 在延伸的子網路中，如果您將機器及其 IP (第 3 層) 位址組態移至次要網站，網路即會自動將流量路由傳送至新位置。 

- 從第 2 層 (資料連結層) 的觀點而言，您需要可以管理延伸的 VLAN 的網路設備。
- 透過延伸 VLAN，潛在的錯誤網域會延伸到兩個網站。 這會成為單一失敗點。 雖然可能性不高，但在此情況下，您可能無法隔離廣播風暴之類的事件。 


### <a name="fail-over-a-subnet"></a>進行子網路容錯移轉

您可以進行整個子網路的容錯移轉以獲得延伸子網路的好處，但卻不用實際延伸。 在此解決方案中，子網路可用於來源或目標網站，但不能同時使用於兩者。

- 若要在發生容錯移轉時維持 IP 位址空間，您可以程式設計方式排列路由器基礎結構，將子網路從某個網站移至另一個網站。
- 當容錯移轉發生時，子網路會連同相關聯的 VM 一起移動。
- 此方法的主要缺點在於，如果發生失敗，您將必須移動整個子網路。

#### <a name="example"></a>範例

以下是完整的子網路容錯移轉範例。 

- 在容錯移轉前，主要網站有在子網路 192.168.1.0/24 中執行的應用程式。
- 在容錯移轉期間，這個子網路中的所有 VM 皆容錯移轉至次要網站，並保留其 IP 位址。 
- 所有網站之間的路由都必須修改，以反映子網路 192.168.1.0/24 中的所有 VM 現在都已移至次要網站的事實。

下圖顯示容錯移轉前後的子網路。


**容錯移轉之前**

![容錯移轉之前](./media/hyper-v-vmm-networking/network-design2.png)

**容錯移轉之後**

![容錯移轉之後](./media/hyper-v-vmm-networking/network-design3.png)

在容錯移轉後，Site Recovery 會為 VM 上的每個網路介面配置一個 IP 位址。 系統會針對每個 VM 執行個體，從相關網路中的靜態 IP 位址集區配置 IP 位址。

- 如果次要網站中的 IP 位址集區與來源網站中的相同，Site Recovery 會配置同一個 IP 位址 (來源 VM 的 IP 位址) 給複本 VM。 IP 位址會保留在 VMM 中，但不會設為 Hyper-V 主機上的容錯移轉 IP 位址。 Hyper-V 主機上的容錯移轉 IP 位址會在容錯移轉之前設定。
- 如果同一個 IP 位址無法使用，Site Recovery 會從集區配置另一個可用的 IP 位址。
- 如果 VM 使用 DHCP，Site Recovery 無法管理 IP 位址。 您必須確定次要網站上的 DHCP 伺服器可以從與來源網站相同的範圍配置位址。

### <a name="validate-the-ip-address"></a>驗證 IP 位址

啟用 VM 保護之後，您可以使用下列範例指令碼來驗證指派給 VM 的位址。 此 IP 位址會設定為容錯移轉 IP 位址，並在容錯移轉時指派給 VM：

    ```
    $vm = Get-SCVirtualMachine -Name <VM_NAME>
    $na = $vm[0].VirtualNetworkAdapters>
    $ip = Get-SCIPAddress -GrantToObjectID $na[0].id
    $ip.address 
    ```

## <a name="use-a-different-ip-address"></a>使用不同的 IP 位址

在此案例中，容錯移轉之 VM 的 IP 位址會變更。 此解決方案的缺點是需要維護。  DNS 和快取項目可能需要更新。 這可能會導致停機時間，您透過下列方式加以避免：

- 對內部網路應用程式使用低 TTL 值。
- 在 Site Recovery 復原計劃中使用下列指令碼，讓 DNS 伺服器能及時更新。 如果您使用動態 DNS 登錄，則不需要這個指令碼。

    ```
    param(
    string]$Zone,
    [string]$name,
    [string]$IP
    )
    $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
    $newrecord = $record.clone()
    $newrecord.RecordData[0].IPv4Address  =  $IP
    Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord
    ```
    
### <a name="example"></a>範例 

在此範例中，主要和次要網站有不同的 IP 位址，而且有第三個網站，可以從中存取裝載於主要或復原網站上的應用程式。

- 在容錯移轉前，應用程式在子網路 192.168.1.0/24 上裝載於主要網站。
- 在容錯移轉後，應用程式設定於子網路 172.16.1.0/24 中的次要網站上。
- 三個網站都可互相存取。
- 在容錯移轉之後，應用程式將於復原子網路中還原。
- 在此案例中，不需要容錯移轉整個子網路，而且不需任何變更，即可重新設定 VPN 或網路路由。 容錯移轉和某些 DNS 更新可確保應用程式可供存取。
- 如果 DNS 設定為允許動態更新，則 VM 在容錯移轉後啟動時，就會使用新的 IP 位址自行註冊。

**容錯移轉之前**

![不同的 IP 位址 - 容錯移轉之前](./media/hyper-v-vmm-networking/network-design10.png)

**容錯移轉之後**

![不同的 IP 位址 - 容錯移轉之後](./media/hyper-v-vmm-networking/network-design11.png)


## <a name="next-steps"></a>後續步驟

[執行容錯移轉](hyper-v-vmm-failover-failback.md)

