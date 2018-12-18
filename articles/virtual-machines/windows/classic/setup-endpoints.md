---
title: 在傳統 Windows VM 上設定端點 | Microsoft Docs
description: 了解如何在 Azure 入口網站中設定傳統 Windows VM 的端點，以允許與 Azure 中的 Windows 虛擬機器進行通訊。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: 8afc21c2-d3fb-43a3-acce-aa06be448bb6
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 06/09/2017
ms.author: cynthn
ms.openlocfilehash: cca9adb40557cf7bf9e1d4129fc6bd61cbf0df4f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38618234"
---
# <a name="how-to-set-up-endpoints-on-a-classic-windows-virtual-machine-in-azure"></a>如何在 Azure 中的傳統 Windows 虛擬機器上設定端點
在 Azure 中使用傳統部署模型建立的所有 Windows 虛擬機器，都可以自動透過私人網路通道與同一雲端服務或虛擬網路中的其他虛擬機器通訊。 不過，網際網路或其他虛擬網路上的電腦需要端點，才能將傳入網路流量導向至虛擬機器。 本文也適用於 [Linux 虛擬機器](../../linux/classic/setup-endpoints.md)。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

在 **Resource Manager** 部署模型中，是使用**網路安全性群組 (NSG)** 來設定端點。 如需詳細資訊，請參閱 [允許使用 Azure 入口網站從外部存取您的 VM](../nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

當您在 Azure 入口網站建立 Windows 虛擬機器時，通常也會自動為您建立像是遠端桌面和 Windows PowerShell 遠端等的通用端點。 建立虛擬機器或日後有需要時，您可以設定其他端點。

[!INCLUDE [virtual-machines-common-classic-setup-endpoints](../../../../includes/virtual-machines-common-classic-setup-endpoints.md)]

## <a name="next-steps"></a>後續步驟
* 若要使用 Azure PowerShell Cmdlet 來設定 VM 端點，請參閱 [Add-AzureEndpoint](https://msdn.microsoft.com/library/azure/dn495300.aspx)。
* 若要使用 Azure PowerShell Cmdlet 來管理端點上的 ACL，請參閱 [使用 PowerShell 管理端點的存取控制清單 (ACL)](../../../virtual-network/virtual-networks-acl-powershell.md)。
* 如果您已在 Resource Manager 部署模型中建立虛擬機器，則可以使用 Azure PowerShell 來 [建立網路安全性群組](../../../virtual-network/tutorial-filter-network-traffic.md) 以控制對 VM 的流量。
