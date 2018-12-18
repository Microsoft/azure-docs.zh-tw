---
title: 關於 Azure 點對站路由 | Microsoft Docs
description: 本文將協助您了解點對站 VPN 路由如何運作。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/16/2018
ms.author: anzaman
ms.openlocfilehash: 620a2bf9221bdb7c46dc36a2b3ed23d853faff35
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031704"
---
# <a name="about-point-to-site-vpn-routing"></a>關於點對站 VPN 路由

本文將協助您了解 Azure 點對站 VPN 路由如何運作。 P2S VPN 路由行為取決於用戶端 OS、用於 VPN 連線的通訊協定，以及虛擬網路 (VNet) 彼此連線的方式。

Azure 目前支援兩種遠端存取通訊協定，即 IKEv2 和 SSTP。 許多用戶端作業系統都支援 IKEv2，包括 Windows、Linux、MacOS、Android 及 iOS。 SSTP 則只在 Windows 上才有支援。 如果您對網路拓撲進行變更並擁有 Windows VPN 用戶端，則必須重新下載並安裝 Windows 用戶端的 VPN 用戶端套件，才能將變更套用至用戶端。

> [!NOTE]
> 本文僅適用於 IKEv2。
>

## <a name="diagrams"></a>關於圖表

本文中有一些不同的圖表。 每節都示範一個不同的拓撲或設定。 基於本文的目的，站對站 (S2S) 和 VNet 對 VNet 連線會以相同方式運作，因為兩者皆為 IPsec 通道。 本文中的所有 VPN 閘道都是路由型。

## <a name="isolatedvnet"></a>一個隔離的 VNet

此範例中的點對站 VPN 閘道連線是用於未與任何其他虛擬網路 (VNet1) 建立連線或對等互連的 VNet。 在此範例中，使用 SSTP 或 IKEv2 的用戶端可以存取 VNet1。

![隔離的 VNet 路由](./media/vpn-gateway-about-point-to-site-routing/1.jpg "隔離的 VNet 路由")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端可以存取 VNet1

* 非 Windows 用戶端可以存取 VNet1

## <a name="multipeered"></a>多個對等互連的 VNet

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 已與 VNet2 對等互連。 VNet2 已與 VNet3 對等互連。 VNet1 已與 VNet4 對等互連。 VNet1 與 VNet3 之間沒有任何直接的對等互連。 VNet1 已啟用 [允許閘道器傳輸]，而 VNet2 已啟用 [使用遠端閘道]。

使用 Windows 的用戶端可以直接存取已對等互連的 VNet，但如果對 VNet 對等互連或網路拓撲進行了任何變更，就必須重新下載 VPN 用戶端。 非 Windows 用戶端可以存取已對等互連的 VNet。 存取權不可轉移且僅限於已直接對等互連的 VNet。

![多個對等互連的 VNet](./media/vpn-gateway-about-point-to-site-routing/2.jpg "多個對等互連的 VNet")

### <a name="address-space"></a>位址空間：

* VNet1：10.1.0.0/16

* VNet2：10.2.0.0/16

* VNet3：10.3.0.0/16

* VNet4：10.4.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、10.4.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、10.4.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端可以存取 VNet1、VNet2 及 VNet4，但必須重新下載 VPN 用戶端，才能讓所有拓撲變更生效。

* 非 Windows 用戶端可以存取 VNet1、VNet2 及 VNet4

## <a name="multis2s"></a>多個使用 S2S VPN 來連線的 VNet

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 使用站對站 VPN 連線來連線至 VNet2。 VNet2 使用站對站 VPN 連線來連線至 VNet3。 VNet1 與 VNet3 之間沒有任何直接的對等互連或站對站 VPN 連線。 所有站對站連線都不執行 BGP 來進行路由。

用戶端如果使用 Windows 或另一個支援的 OS，則只能存取 VNet1。 若要存取額外的 VNet，必須使用 BGP。

![多個 VNet 和 S2S](./media/vpn-gateway-about-point-to-site-routing/3.jpg "多個 VNet 和 S2S")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* VNet2：10.2.0.0/16

* VNet3：10.3.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端只能存取 VNet1

* 非 Windows 用戶端只能存取 VNet1

## <a name="multis2sbgp"></a>多個使用 S2S VPN (BGP) 來連線的 VNet

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 使用站對站 VPN 連線來連線至 VNet2。 VNet2 使用站對站 VPN 連線來連線至 VNet3。 VNet1 與 VNet3 之間沒有任何直接的對等互連或站對站 VPN 連線。 所有站對站連線都執行 BGP 來進行路由。

用戶端如果使用 Windows 或另一個支援的 OS，將可以存取所有使用站對站 VPN 連線來連線的 VNet，但必須手動將所連線 VNet 的路由新增至 Windows 用戶端。

![多個 VNet 和 S2S (BGP)](./media/vpn-gateway-about-point-to-site-routing/4.jpg "多個 VNet 和 S2S BGP")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* VNet2：10.2.0.0/16

* VNet3：10.3.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、10.3.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端可以存取 VNet1、VNet2 及 VNet3，但必須手動新增 VNet2 和 VNet3 的路由。

* 非 Windows 用戶端可以存取 VNet1、VNet2 及 VNet3

## <a name="vnetbranch"></a>一個 VNet 和一個分公司

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 未與任何其他虛擬網路建立連線/對等互連，但已透過未執行 BGP 的站對站 VPN 連線與內部部署站台建立連線。

Windows 和非 Windows 用戶端只能存取 VNet1。

![VNet 和分公司的相關路由](./media/vpn-gateway-about-point-to-site-routing/5.jpg "VNet 和分公司的相關路由")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* Site1：10.101.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端只能存取 VNet1

* 非 Windows 用戶端只能存取 VNet1

## <a name="vnetbranchbgp"></a>一個 VNet 和一個分公司 (BGP)

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 未與任何其他虛擬網路建立連線或對等互連，但已透過執行 BGP 的站對站 VPN 連線與內部部署站台 (Site1) 建立連線。

Windows 用戶端可以存取 VNet 和分公司 (Site1)，但必須手動將 Site1 的路由新增至用戶端。 非 Windows 用戶端可以存取 VNet，也可以存取內部部署的分公司。

![一個 VNet 和一個分公司 (BGP)](./media/vpn-gateway-about-point-to-site-routing/6.jpg "一個 VNet 和一個分公司")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* Site1：10.101.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至 Windows 用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.101.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端可以存取 VNet1 和 Site1，但必須手動新增 Site1 的路由。

* 非 Windows 用戶端可以存取 VNet1 和 Site1。


## <a name="multivnets2sbranch"></a>多個使用 S2S 和分公司來連線的 VNet

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 使用站對站 VPN 連線來連線至 VNet2。 VNet2 使用站對站 VPN 連線來連線至 VNet3。 VNet1 與 VNet3 網路之間沒有任何直接的對等互連或站對站 VPN 通道。 VNet3 使用站對站 VPN 連線來連線至分公司 (Site1)。 所有 VPN 連線都未執行 BGP。

所有用戶端都只能存取 VNet1。

![多 VNet S2S 和分公司](./media/vpn-gateway-about-point-to-site-routing/7.jpg "多 VNet S2S 和分公司")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* VNet2：10.2.0.0/16

* VNet3：10.3.0.0/16

* Site1：10.101.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、10.3.0.0/16、10.101.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端只能存取 VNet1

* 非 Windows 用戶端只能存取 VNet1

## <a name="multivnets2sbranchbgp"></a>多個使用 S2S 和分公司 (BGP) 來連線的 VNet

在此範例中，點對站 VPN 閘道連線是用於 VNet1。 VNet1 使用站對站 VPN 連線來連線至 VNet2。 VNet2 使用站對站 VPN 連線來連線至 VNet3。 VNet1 與 VNet3 網路之間沒有任何直接的對等互連或站對站 VPN 通道。 VNet3 使用站對站 VPN 連線來連線至分公司 (Site1)。 所有 VPN 連線都執行 BGP。

用戶端如果使用 Windows，將可以存取使用站對站 VPN 連線來連線的 VNet，但必須手動將 VNet2、VNet3 及 Site1 的路由新增至用戶端。 非 Windows 用戶端可以存取使用站對站 VPN 連線來連線的 VNet 和站台，無須任何手動介入操作。 此存取權可轉移，且用戶端可以存取所有已連線 VNet 和站台 (內部部署) 中的資源。

![多 VNet S2S 和分公司](./media/vpn-gateway-about-point-to-site-routing/8.jpg "多 VNet S2S 和分公司")

### <a name="address-space"></a>位址空間

* VNet1：10.1.0.0/16

* VNet2：10.2.0.0/16

* VNet3：10.3.0.0/16

* Site1：10.101.0.0/16

### <a name="routes-added"></a>新增的路由

* 新增至用戶端的路由：10.1.0.0/16、192.168.0.0/24

* 新增至非 Windows 用戶端的路由：10.1.0.0/16、10.2.0.0/16、10.3.0.0/16、10.101.0.0/16、192.168.0.0/24

### <a name="access"></a>Access

* Windows 用戶端可以存取 VNet1、VNet2、VNet3 及 Site1，但必須手動將 VNet2、VNet3 及 Site1 的路由新增至用戶端。

* 非 Windows 用戶端可以存取 VNet1、VNet2、VNet3 及 Site1。

## <a name="next-steps"></a>後續步驟

請參閱[使用 Azure 入口網站來建立 P2S VPN](vpn-gateway-howto-point-to-site-resource-manager-portal.md) 以開始建立 P2S VPN。
