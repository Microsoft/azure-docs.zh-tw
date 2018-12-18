---
title: Azure 虛擬網路 TAP 概觀 | Microsoft Docs
description: 了解虛擬網路 TAP。 虛擬網路 TAP 可為您提供虛擬機器網路流量的深層複本，以供串流至封包收集器。
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: kaanan
ms.openlocfilehash: 7270ab6203cfa3602fc36bc6fa7d30cd622ce3a3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946590"
---
# <a name="virtual-network-tap"></a>虛擬網路 TAP

Azure 虛擬網路 TAP (終端機存取點) 可讓您持續將虛擬機器網路流量串流到網路封包收集器或分析工具。 收集器或分析工具是由[網路虛擬設備](https://azure.microsoft.com/solutions/network-appliances/)合作夥伴所提供。 如需經驗證能與虛擬網路 TAP 相容的合作夥伴解決方案清單，請參閱[合作夥伴解決方案](#virtual-network-tap-partner-solutions)。

> [!IMPORTANT]
> 虛擬網路 TAP 目前在 WestCentralUS Azure 區域中屬於開發人員預覽版本。 若要使用虛擬網路 TAP，您必須將電子郵件傳送至 <azurevnettap@microsoft.com> 並註明訂用帳戶識別碼，以註冊預覽版。 當訂用帳戶註冊完成之後，您會收到電子郵件。 在收到確認電子郵件後才能使用此功能。 此開發人員預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="virtual-network-tap-partner-solutions"></a>虛擬網路 TAP 合作夥伴解決方案

### <a name="network-packet-brokers"></a>網路封包訊息代理程式

- [Big Switch Big Monitoring Fabric](https://www.bigswitch.com/products/big-monitoring-fabric/public-cloud/microsoft-azure)
- [Flowmon](https://www.flowmon.com/blog/azure-vtap)
- [Gigamon GigaSECURE](https://blog.gigamon.com/2018/09/13/why-microsofts-new-vtap-service-works-even-better-with-gigasecure-for-azure)
- [Ixia CloudLens](https://www.ixiacom.com/cloudlens/cloudlens-azure)

### <a name="security-analytics-networkapplication-performance-management"></a>安全性分析、網路/應用程式效能管理

- [ExtraHop Reveal(x)](https://www.extrahop.com/company/tech-partners/microsoft/)
- [Fidelis Cybersecurity](https://www.fidelissecurity.com/technology-partners/microsoft-azure )
- [Netscout vSTREAM]( https://www.netscout.com/technology-partners/microsoft/azure-vtap)
- [Nubeva Prisms](https://www.nubeva.com/azurevtap)
- [RSA NetWitness® Platform](https://www.rsa.com/azure)
- [Vectra Cognito](https://vectra.ai/microsoftazure)

下圖顯示虛擬網路 TAP 的運作方式。 您可以在部署於虛擬網路的虛擬機器上，對其所連結的[網路介面](virtual-network-network-interface.md)新增 TAP 設定。 目的地是和所監視網路介面相同的虛擬網路或[對等互連虛擬](virtual-network-peering-overview.md)網路中所含的虛擬網路 IP 位址。 虛擬網路 TAP 的收集器解決方案可以部署在 [Azure 內部負載平衡器](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#concepts)後方，以獲得高可用性。 若要評估個別解決方案的部署選項，請參閱[合作夥伴解決方案](#virtual-network-tap-partner-solutions)。

![虛擬網路 TAP 的運作方式](./media/virtual-network-tap/architecture.png)

## <a name="prerequisites"></a>必要條件

在建立虛擬網路 TAP 之前，您必須已收到您註冊預覽版的確認電子郵件，且有一或多部使用 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 部署模型所建立的虛擬機器，並有合作夥伴解決方案可用來在 WestCentralUS 區域中彙總 TAP 流量。 如果虛擬網路中沒有合作夥伴解決方案，請參閱[合作夥伴解決方案](#virtual-network-tap-partner-solutions)來加以部署。 您可以使用相同的虛擬網路 TAP 資源，以從相同或不同訂用帳戶中的多個網路介面彙總流量。 如果所監視的網路介面位在不同訂用帳戶中，則這兩個訂用帳戶必須與相同的 Azure Active Directory 租用戶相關聯。 此外，所監視的網路介面和用於彙總 TAP 流量的目的地端點可以位於相同區域中的對等互連虛擬網路。 如果您要使用這種部署模型，請確保[虛擬網路對等互連](virtual-network-peering-overview.md)已啟用，再設定虛擬網路 TAP。

## <a name="permissions"></a>權限

用來對網路介面套用 TAP 設定的帳戶，必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或屬於已指派下表中必要動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)：

| 動作 | 名稱 |
|---|---|
| Microsoft.Network/virtualNetworkTaps/* | 必須有此動作，才能建立、更新、讀取和刪除虛擬網路 TAP 資源 |
| Microsoft.Network/networkInterfaces/read | 必須有此動作，才能讀取要在其上設定 TAP 的網路介面資源 |
| Microsoft.Network/tapConfigurations/* | 必須有此動作，才能建立、更新、讀取和刪除網路介面上的 TAP 設定 |

## <a name="next-steps"></a>後續步驟

- 了解如何[建立虛擬網路 TAP](tutorial-tap-virtual-network-cli.md)。
