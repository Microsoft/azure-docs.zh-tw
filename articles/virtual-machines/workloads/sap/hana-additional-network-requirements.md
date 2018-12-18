---
title: SAP HANA on Azure (大型執行個體) 的其他網路需求 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 的其他網路需求。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8ee89553846309bd96f3cad3648e337488cf07d3
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44167582"
---
# <a name="additional-network-requirements"></a>其他網路需求

有時候，您可能會將額外的網路需求當作 HANA 大型執行個體部署的一部分。 本文將說明下列網路需求：
- [新增更多 IP 位址或子網路](#adding-more-ip-addresses-or-subnets)
- [新增虛擬網路](#adding-vnets)
- [增加 Express Route 線路頻寬](#increasing-expressroute-circuit-bandwidth)
- [新增額外的 Express Route 線路](#adding-an-additional-expressroute-circuit)
- [刪除子網路](#deleting-a-subnet)
- [刪除虛擬網路](#deleting-a-vnet)
- [刪除 Express Route 線路](#deleting-an-expressroute-circuit)


## <a name="adding-more-ip-addresses-or-subnets"></a>新增更多 IP 位址或子網路

新增更多 IP 位址或子網路時，您可以使用 Azure 入口網站、PowerShell 或 CLI。

在此情況下，建議您將新的 IP 位址範圍新增到 VNet 位址空間中作為新的範圍，而不要產生新的彙總範圍。 不論是哪一種情況，您都必須向 Microsoft 提交這項變更，以允許從該新 IP 位址範圍連線到您用戶端中的「HANA 大型執行個體」單位。 您可以開啟 Azure 支援要求，以新增新的 VNet 位址空間。 收到確認之後，請執行後續步驟。

若要從 Azure 入口網站建立額外的子網路，請參閱[使用 Azure 入口網站建立虛擬網路](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network)一文，若要從 PowerShell 建立，則請參閱[使用 PowerShell 建立虛擬網路](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network)。

## <a name="adding-vnets"></a>新增 VNet

在一開始連接到一或多個 Azure VNet 之後，您可能會想要新增可存取 SAP HANA on Azure (大型執行個體) 的額外 VNet。 首先，請提交 Azure 支援要求，在該要求中同時包含識別特定 Azure 部署的具體資訊，以及「Azure VNet 位址空間」的 IP 位址空間範圍。 接著，「SAP HANA on Azure 服務管理」就會提供您連接額外 VNet 和 ExpressRoute.所需的必要資訊。 針對每個 VNet，您將需要一個唯一的授權金鑰，才能連接到 ExpressRoute 線路，再連接到 HANA 大型執行個體。

新增新 Azure VNet 的步驟：

1. 完成上架程序中的第一個步驟，請參閱_建立 Azure VNet_ 一節。
2. 完成上架程序中的第二個步驟，請參閱_建立閘道子網路_一節。
3. 使用新 VNet 的相關資訊來開啟 Azure 支援要求，並要求一個新的授權金鑰以將額外的 VNet 連接到 HANA 大型執行個體 ExpressRoute 線路。
4. 收到授權完成的通知之後，請使用 Microsoft 提供的授權資訊來完成上架程序中的第三個步驟，請參閱_連結 VNet_ 一節。

## <a name="increasing-expressroute-circuit-bandwidth"></a>增加 ExpressRoute 線路頻寬

請洽詢「SAP HANA on Azure 服務管理」。 如果系統建議您增加 SAP HANA on Azure (大型執行個體) ExpressRoute 線路的頻寬，請建立 Azure 支援要求。 (針對單一線路頻寬，您最多可以要求增加到 10 Gbps)。作業完成後，您便會收到通知；不需要進行任何額外動作，即可在 Azure 中啟用這個較高的速度。

## <a name="adding-an-additional-expressroute-circuit"></a>新增額外的 ExpressRoute 線路

如果系統告知您需要額外的 ExpressRoute 線路，請洽詢「SAP HANA on Azure 服務管理」，請提出 Azure 支援要求來建立新的 ExpressRoute 線路 (並取得可連線到此線路的授權資訊)。 在提出要求之前，必須先定義將在 VNet 上使用的位址空間，如此 SAP HANA on Azure 服務管理才能提供授權。

建立新線路且 SAP HANA on Azure 服務管理設定完成之後，您將會收到通知，當中含有您繼續作業所需的資訊。 請依照上面提供的步驟來建立新 VNet 並將其連接到這個額外的線路。 如果 Azure VNet 已經連接到同一 Azure 區域的另一個 SAP HANA on Azure (大型執行個體) ExpressRoute 線路，您就無法將它們連接到這個額外的線路。

## <a name="deleting-a-subnet"></a>刪除子網路

若要移除 VNet 子網路，您可以使用 Azure 入口網站、PowerShell 或 CLI。 如果您的 Azure VNet IP 位址範圍/Azure VNet 位址空間是一個彙總的範圍，後續就沒有需要追蹤 Microsoft 的事項。 唯一的例外是，VNet 仍然會傳播包含已刪除之子網路的 BGP 路由位址空間。 如果您將 Azure VNet IP 位址範圍/Azure VNet 位址空間定義成多個 IP 位址範圍，而其中一個已指派給已刪除的子網路，則您應該從「VNet 位址空間」中刪除該範圍，並接著通知「SAP HANA on Azure 服務管理」將它從允許 SAP HANA on Azure (大型執行個體) 進行通訊的範圍中移除。

若要刪除子網路，請參閱[刪除子網路](../../../virtual-network/virtual-network-manage-subnet.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#delete-a-subnet)以取得建立子網路的詳細資訊。

## <a name="deleting-a-vnet"></a>刪除 VNet

若要刪除虛擬網路，請參閱[刪除虛擬網路](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#delete-a-virtual-network)。 「SAP HANA on Azure 服務管理」會移除 SAP HANA on Azure (大型執行個體) ExpressRoute 線路上的現有授權，並移除用來與「HANA 大型執行個體」通訊的 Azure VNet IP 位址範圍/Azure VNet 位址空間。

移除 VNet 之後，請開啟 Azure 支援要求來提供要移除的 IP 位址空間範圍。

為了確保移除所有項目，請刪除下列項目：

- ExpressRoute 連線、VNet 閘道、VNet 閘道公用 IP 及 VNet

## <a name="deleting-an-expressroute-circuit"></a>刪除 ExpressRoute 線路

若要移除額外的 SAP HANA on Azure (大型執行個體) ExpressRoute 線路，請向 SAP HANA on Azure 服務管理開啟 Azure 支援要求，然後要求務必刪除該線路。 您可以視需要在 Azure 訂用帳戶內刪除或保留 VNet。 不過，您必須將「HANA 大型執行個體」ExpressRoute 線路與所連結 VNet 閘道之間的連線刪除。

如果您也想要移除 VNet，請依照上面＜刪除 VNet＞一節中的指導方針操作。

**後續步驟**

- 請參閱[如何在 Azure 上安裝和設定 SAP HANA (大型執行個體)](hana-installation.md)。
