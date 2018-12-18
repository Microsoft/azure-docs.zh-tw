---
title: 刪除虛擬網路閘道：Azure 入口網站：Resource Manager | Microsoft Docs
description: 在 Resource Manager 部署模型中使用 Azure 入口網站刪除虛擬網路閘道。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: ''
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/20/2017
ms.author: cherylmc
ms.openlocfilehash: b014d43ab25124d3e08e19ca190b320fc8456593
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27593451"
---
# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>使用入口網站刪除虛擬網路閘道

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (傳統)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)

本文會提供您方法來刪除使用 Resource Manager 部署模型部署的 Azure VPN 閘道。 如果您想要刪除 VPN 閘道組態的虛擬網路閘道，您可以採取幾種不同的方法。

- 如果您想要刪除所有內容並重頭開始 (例如使用測試環境的情況)，您可以刪除資源群組。 刪除資源群組時，會一併刪除群組內的所有資源。 只有在您不想保留資源群組中的任何資源時，才建議使用此方法。 您無法使用這種方法，選擇性地只刪除一些資源。

- 如果您想要保留資源群組中的部分資源，刪除虛擬網路閘道會變得稍微複雜一點。 您必須先刪除任何相依於閘道的資源，才可以刪除虛擬網路閘道。 您遵循的步驟取決於您所建立的連線類型以及每個連線的相依資源。

> [!IMPORTANT]
> 以下指示說明如何刪除使用 Resource Manager 部署模型部署的 Azure VPN 閘道。 若要刪除使用傳統部署模型部署的 VPN 閘道，請依循[此處](vpn-gateway-delete-vnet-gateway-classic-powershell.md)的描述使用 Azure PowerShell。


## <a name="delete-a-vpn-gateway"></a>刪除 VPN 閘道

若要刪除虛擬網路閘道，您必須先刪除虛擬網路閘道的每個相關資源。 由於相依性，您必須依照特定順序刪除資源。

[!INCLUDE [delete gateway](../../includes/vpn-gateway-delete-vnet-gateway-portal-include.md)]

此時，虛擬網路閘道已經刪除。 後續步驟會協助您刪除不再使用的任何資源。

### <a name="to-delete-the-local-network-gateway"></a>刪除區域網路閘道

1. 在 [所有資源] 中，找出與每個連線相關聯的區域網路閘道。
2. 在區域網路閘道的 [概觀] 刀鋒視窗中，按一下 [刪除]。

### <a name="to-delete-the-public-ip-address-resource-for-the-gateway"></a>刪除閘道的公用 IP 位址資源

1. 在 [所有資源] 中，找出與閘道相關聯的公用 IP 位址資源。 如果虛擬網路閘道為主動-主動，您會看到兩個公用 IP 位址。 
2. 在公用 IP 位址的 [概觀] 頁面上，按一下 [刪除]，然後按一下 [是] 以確認。

### <a name="to-delete-the-gateway-subnet"></a>刪除閘道子網路

1. 在 [所有資源] 中，找出虛擬網路。 
2. 在 [子網路] 刀鋒視窗中，按一下 [GatewaySubnet]，然後按一下 [刪除]。 
3. 按一下 [是] 確認要刪除閘道子網路。

## <a name="deleterg"></a>藉由刪除資源群組來刪除 VPN 閘道

如果您不在乎保留資源群組中任何資源，而只想要從頭開始，您可以刪除整個資源群組。 這是移除所有項目的快速方法。 下列步驟僅適用於 Resource Manager 部署模型。

1. 在 [所有資源] 中，找出資源群組，然後按一下以開啟刀鋒視窗。
2. 按一下 [刪除] 。 在 [刪除] 刀鋒視窗中，檢視受影響的資源。 請確定您想要刪除所有資源。 如果不是，請使用這篇文章上方之[刪除 VPN 閘道](#deletegw)中的步驟。
3. 若要繼續，請輸入您想要刪除之資源群組的名稱，然後按一下 [刪除]。
