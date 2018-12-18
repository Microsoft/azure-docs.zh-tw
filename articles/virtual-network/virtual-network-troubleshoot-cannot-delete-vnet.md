---
title: 無法刪除 Azure 中的虛擬網路 | Microsoft Docs
description: 了解如何對您無法刪除 Azure 中之虛擬網路的問題進行疑難排解。
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
ms.openlocfilehash: d3ffcd9981ff0c57f6d625efcede3e930acd17b5
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34072104"
---
# <a name="troubleshooting-failed-to-delete-a-virtual-network-in-azure"></a>疑難排解：無法刪除 Azure 中的虛擬網路

當您嘗試刪除 Microsoft Azure 中的虛擬網路時，可能會收到錯誤。 本文提供可協助您解決此問題的疑難排解步驟。 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-guidance"></a>疑難排解指引 

1. [檢查虛擬網路中是否正在執行虛擬網路閘道](#check-whether-a-virtual-network-gateway-is-running-in-the-virtual-network)。
2. [檢查虛擬網路中是否正在執行應用程式閘道](#check-whether-an-application-gateway-is-running-in-the-virtual-network)。
3. [檢查虛擬網路中是否已啟用 Azure Active Directory 網域服務](#check-whether-azure-active-directory-domain-service-is-enabled-in-the-virtual-network)。
4. [檢查虛擬網路是否已連線到其他資源](#check-whether-the-virtual-network-is-connected-to-other-resource)。
5. [檢查虛擬機器是否仍在虛擬網路中執行](#check-whether-a-virtual-machine-is-still-running-in-the-virtual-network)。
6. [檢查虛擬網路是否卡在移轉狀態](#check-whether-the-virtual-network-is-stuck-in-migration)。

## <a name="troubleshooting-steps"></a>疑難排解步驟

### <a name="check-whether-a-virtual-network-gateway-is-running-in-the-virtual-network"></a>檢查虛擬網路中是否正在執行虛擬網路閘道

若要移除虛擬網路，您必須先移除虛擬網路閘道。

如果是傳統虛擬網路，請移至 Azure 入口網站中傳統虛擬網路的 [概觀]頁面。 在 [VPN 連線] 區段中，如果閘道正在虛擬網路中執行，您將會看到閘道的 IP 位址。 

![檢查閘道是否正在執行](media/virtual-network-troubleshoot-cannot-delete-vnet/classic-gateway.png)

如果是虛擬網路，請移至虛擬網路的 [概觀] 頁面。 檢查虛擬網路閘道的 [連線的裝置]。

![檢查連線的裝置](media/virtual-network-troubleshoot-cannot-delete-vnet/vnet-gateway.png)

先移除閘道中的任何**連線**物件，然後才能移除閘道。 

### <a name="check-whether-an-application-gateway-is-running-in-the-virtual-network"></a>檢查虛擬網路中是否正在執行應用程式閘道

移至虛擬網路的 [概觀] 頁面。 檢查應用程式閘道的 [連線的裝置]。

![檢查連線的裝置](media/virtual-network-troubleshoot-cannot-delete-vnet/app-gateway.png)

如果有應用程式閘道，您必須先移除它，才能刪除虛擬網路。

### <a name="check-whether-azure-active-directory-domain-service-is-enabled-in-the-virtual-network"></a>檢查虛擬網路中是否已啟用 Azure Active Directory 網域服務

如果 Active Directory 網域服務已啟用並連線到虛擬網路，您就無法刪除此虛擬網路。 

![檢查連線的裝置](media/virtual-network-troubleshoot-cannot-delete-vnet/enable-domain-services.png)

若要停用服務，請參閱[使用 Azure 入口網站停用 Azure Active Directory Domain Services](../active-directory-domain-services/active-directory-ds-disable-aadds.md)。

### <a name="check-whether-the-virtual-network-is-connected-to-other-resource"></a>檢查虛擬網路是否已連線到其他資源

檢查線路連結、連線和虛擬網路對等互連。 這其中任一項都會導致虛擬網路刪除失敗。 

建議的刪除順序如下：

1. 閘道連線
2. 閘道
3. IP
4. 虛擬網路對等互連
5. App Service Environment (ASE)

### <a name="check-whether-a-virtual-machine-is-still-running-in-the-virtual-network"></a>檢查虛擬機器是否仍在虛擬網路中執行

確定虛擬網路中沒有任何虛擬機器。

### <a name="check-whether-the-virtual-network-is-stuck-in-migration"></a>檢查虛擬網路是否卡在移轉狀態

如果虛擬網路卡在移轉狀態，就無法刪除。 執行下列命令以中止移轉，然後刪除虛擬網路。

    Move-AzureVirtualNetwork -VirtualNetworkName "Name" -Abort

## <a name="next-steps"></a>後續步驟

- [Azure 虛擬網路](virtual-networks-overview.md)
- [Azure 虛擬網路的常見問題 (FAQ)](virtual-networks-faq.md)
