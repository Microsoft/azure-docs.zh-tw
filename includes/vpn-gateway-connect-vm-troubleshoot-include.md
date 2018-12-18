---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: a22cc1f1bffb0126c4fe23bd9f28b292bfdc48a8
ms.sourcegitcommit: baed5a8884cb998138787a6ecfff46de07b8473d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "30198038"
---
如果您無法透過 VPN 連線與虛擬機器連線，請檢查下列各項：

- 確認您的 VPN 連線成功。
- 確認您是連線至 VM 的私人 IP 位址。
- 如果您可以使用私人 IP 位址 (而非電腦名稱) 來連線至 VM，請確認您已正確設定 DNS。 如需 VM 的名稱解析運作方式的詳細資訊，請參閱 [VM 的名稱解析](../articles/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

當您透過點對站連線時，請檢查下列其他項目︰

- 請使用 'ipconfig' 來檢查指派給所連線電腦上的乙太網路介面卡之 IPv4 位址。 如果 IP 位址位在您要連線的 VNet 位址範圍內，或在您 VPNClientAddressPool 的位址範圍內，這稱為重疊位址空間。 當您的位址空間以這種方式重疊時，網路流量不會連線到 Azure，它會保留在本機網路上。
- 請確認 VPN 用戶端設定套件是在針對 VNet 指定的 DNS 伺服器 IP 位址之後產生。 如果您已更新 DNS 伺服器 IP 位址，請產生並安裝新的 VPN 用戶端設定套件。

如需針對 RDP 連線進行疑難排解的詳細資訊，請參閱[針對 VM 的遠端桌面連線進行疑難排解](../articles/virtual-machines/windows/troubleshoot-rdp-connection.md)。