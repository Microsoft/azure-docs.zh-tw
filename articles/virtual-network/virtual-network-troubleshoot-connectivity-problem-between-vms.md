---
title: 為 Azure VM 之間的連線問題疑難排解 | Microsoft Docs
description: 了解如何為 Azure VM 之間的連線問題進行疑難排解。
services: virtual-network
documentationcenter: na
author: chadmath
manager: cshepard
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 02f6e2648c007cfe87984e8de886b8b76e1e8d16
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409910"
---
# <a name="troubleshooting-connectivity-problems-between-azure-vms"></a>為 Azure VM 之間的連線問題疑難排解

您可能會遇到 Azure 虛擬機器 (VM) 之間的連線問題。 本文提供可協助您解決此問題的疑難排解步驟。 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="symptom"></a>徵狀

某個 Azure VM 無法連線至另一個 Azure VM。

## <a name="troubleshooting-guidance"></a>疑難排解指引 

1. [檢查 NIC 是否設定不正確](#step-1-check-whether-nic-is-misconfigured)
2. [檢查 NSG 或 UDR 是否封鎖網路流量](#step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr)
3. [檢查 VM 防火牆是否封鎖網路流量](#step-3-check-whether-network-traffic-is-blocked-by-vm-firewall)
4. [檢查 VM 應用程式或服務是否正在接聽連接埠](#step-4-check-whether-vm-app-or-service-is-listening-on-the-port)
5. [檢查問題是否因 SNAT 而造成](#step-5-check-whether-the-problem-is-caused-by-snat)
6. [檢查傳統 VM 的 ACL 是否封鎖流量](#step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm)
7. [檢查是否已為傳統 VM 建立端點](#step-7-check-whether-the-endpoint-is-created-for-the-classic-vm)
8. [嘗試連線至 VM 網路共用](#step-8-try-to-connect-to-a-vm-network-share)
9. [檢查內部 Vnet 連線](#step-9-check-inter-vnet-connectivity)

## <a name="troubleshooting-steps"></a>疑難排解步驟

請遵循下列步驟來疑難排解問題。 完成每個步驟之後，請檢查是否已解決問題。 

### <a name="step-1-check-whether-nic-is-misconfigured"></a>步驟 1：檢查 NIC 是否設定不正確

請遵循[如何重設 Azure Windows VM 的網路介面](../virtual-machines/windows/reset-network-interface.md)中的步驟。 

如果修改網路介面 (NIC) 後發生問題，請遵循下列步驟：

**多個 NIC VM**

1. 新增 NIC。
2. 修正不正確的 NIC 的問題，或移除不正確的 NIC。  然後再次新增 NIC。

如需詳細資訊，請參閱[在虛擬機器中新增或移除網路介面](virtual-network-network-interface-vm.md)。

**單一 NIC VM** 

- [重新部署 Windows VM](../virtual-machines/windows/redeploy-to-new-node.md)
- [重新部署 Linux VM](../virtual-machines/linux/redeploy-to-new-node.md)

### <a name="step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr"></a>步驟 2：檢查 NSG 或 UDR 是否封鎖網路流量

使用[網路監看員 IP 流量驗證](../network-watcher/network-watcher-ip-flow-verify-overview.md)和 [NSG 流量記錄](../network-watcher/network-watcher-nsg-flow-logging-overview.md)來判斷是否有「網路安全性群組」(NSG) 或「使用者定義路由」(UDR) 受到流量干擾。

### <a name="step-3-check-whether-network-traffic-is-blocked-by-vm-firewall"></a>步驟 3：檢查 VM 防火牆是否封鎖網路流量

停用防火牆，然後測試結果。 如果未解決此問題，請確認防火牆設定，然後重新啟用防火牆。

### <a name="step-4-check-whether-vm-app-or-service-is-listening-on-the-port"></a>步驟 4：檢查 VM 應用程式或服務是否正在接聽連接埠

您可以使用下列其中一個方法來檢查 VM 應用程式或服務是否正在接聽連接埠。

- 執行下列命令來檢查伺服器是否正在接聽該通訊埠。

**Windows VM**

    netstat –ano

**Linux VM**

    netstat -l

- 在虛擬機器上執行 **telnet** 命令以測試連接埠上。 如果測試失敗，不會設定應用程式或服務去接聽該連接埠。

### <a name="step-5-check-whether-the-problem-is-caused-by-snat"></a>步驟 5：檢查問題是否因 SNAT 而造成

在某些情況下，VM 會放置在對 Azure 外部資源有相依性的負載平衡解決方案後方。 在這些情況下，如果您遇到間歇性連線問題，則此問題可能是由 [SNAT 連接埠耗盡](../load-balancer/load-balancer-outbound-connections.md)所造成。 若要解決此問題，請為在負載平衡器後方的每個 VM 建立一個 VIP (若為傳統 VM，則建立 ILPIP)，並以 NSG 或 ACL 保護。 

### <a name="step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm"></a>步驟 6：檢查傳統 VM 的 ACL 是否封鎖流量

存取控制清單 (ACL) 提供針對虛擬機器端點選擇性允許或拒絕流量的功能。 如需詳細資訊，請參閱[在端點上管理 ACL](../virtual-machines/windows/classic/setup-endpoints.md#manage-the-acl-on-an-endpoint)。

### <a name="step-7-check-whether-the-endpoint-is-created-for-the-classic-vm"></a>步驟 7：檢查是否已為傳統 VM 建立端點

您在 Azure 中使用傳統部署模型建立的所有虛擬機器，都可以自動透過私人網路通道與同一雲端服務或虛擬網路中的其他虛擬機器通訊。 不過，其他虛擬網路上的電腦需要端點，才能將傳入網路流量導向至虛擬機器。 如需詳細資訊，請參閱[如何設定端點](../virtual-machines/windows/classic/setup-endpoints.md)。

### <a name="step-8-try-to-connect-to-a-vm-network-share"></a>步驟 8：嘗試連線至 VM 網路共用

如果您無法連線到 VM 網路共用，此問題可能是由 VM 中無法使用的 NIC 所造成的。 若要刪除無法使用的 NIC，請參閱[如何刪除無法使用的 NIC](../virtual-machines/troubleshooting/reset-network-interface.md#delete-the-unavailable-nics)

### <a name="step-9-check-inter-vnet-connectivity"></a>步驟 9：檢查內部 Vnet 連線

使用[網路監看員 IP 流量驗證](../network-watcher/network-watcher-ip-flow-verify-overview.md)和 [NSG 流量記錄](../network-watcher/network-watcher-nsg-flow-logging-overview.md)來判斷是否有 NSG 或 UDR 受到流量干擾。 您也可以在[這裡](https://support.microsoft.com/en-us/help/4032151/configuring-and-validating-vnet-or-vpn-connections)驗證您的內部 Vnet 組態。

### <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。