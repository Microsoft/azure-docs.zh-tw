---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 04/09/2018
ms.author: jdial
ms.custom: include file
ms.openlocfilehash: f1dc43b6fea4ebfb47439ec64b573cc471614d98
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31531730"
---
> [!div class="op_single_selector"]
> * [Azure 入口網站](../articles/virtual-network/virtual-network-multiple-ip-addresses-portal.md)
> * [PowerShell](../articles/virtual-network/virtual-network-multiple-ip-addresses-powershell.md)
> * [Azure CLI](../articles/virtual-network/virtual-network-multiple-ip-addresses-cli.md)
>

Azure 虛擬機器 (VM) 可連接一或多個網路介面 (NIC)。 任何 NIC 都可以獲派一或多個靜態或動態公用及私人 IP 位址。 將多個 IP 位址指派給 VM 可啟用下列功能：

* 在單一伺服器上，以不同 IP 位址和 SSL 憑證裝載多個網站或服務。
* 做為網路虛擬設備，例如防火牆或負載平衡器。
* 能夠將任何 NIC 的任何私人 IP 位址新增到 Azure Load Balancer 後端集區。 在過去，只能將主要 NIC 的主要 IP 位址新增到後端集區。 若要深入了解如何負載平衡多個 IP 設定，請參閱[負載平衡多個 IP 組態](../articles/load-balancer/load-balancer-multiple-ip.md?toc=%2fazure%2fvirtual-network%2ftoc.json)文章。

連接到 VM 的每個 NIC 皆有一或多個 IP 組態與其相關聯。 每個組態會獲派一個靜態或動態私人 IP 位址。 每個組態可能也會有一個關聯的公用 IP 位址資源。 公用 IP 位址資源具有任一動態或靜態 IP 位址指派給它。 若要深入了解 Azure 中的 IP 位址，請閱讀 [Azure 中的 IP 位址](../articles/virtual-network/virtual-network-ip-addresses-overview-arm.md)文章。 

可以指派給一個 NIC 的私人 IP 位址數目有所限制。 Azure 訂用帳戶中可以使用的公用 IP 位址數目也有限制。 請參閱 [Azure 限制](../articles/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)一文以取得詳細資料。
