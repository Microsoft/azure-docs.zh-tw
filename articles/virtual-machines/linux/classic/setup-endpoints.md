---
title: 在傳統 Linux VM 上設定端點 | Microsoft Docs
description: 了解如何在 Azure 入口網站中設定 Linux VM 的端點，以允許與 Azure 中的 Linux 虛擬機器進行通訊
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: f3749738-1109-4a1d-8635-40e4bd220e91
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 06/09/2017
ms.author: cynthn
ms.openlocfilehash: 03cb90dcdcc7a7407898ddd8cbc30f1af0833676
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38295682"
---
# <a name="how-to-set-up-endpoints-on-a-linux-classic-virtual-machine-in-azure"></a>如何在 Azure 中的 Linux 傳統虛擬機器上設定端點
在 Azure 中使用傳統部署模型建立的所有 Linux 虛擬機器，都可以自動透過私人網路通道與同一雲端服務或虛擬網路中的其他虛擬機器通訊。 不過，網際網路或其他虛擬網路上的電腦需要端點，才能將傳入網路流量導向至虛擬機器。 本文也適用於 [Windows 虛擬機器](../../windows/classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

在 **Resource Manager** 部署模型中，是使用**網路安全性群組 (NSG)** 來設定端點。 如需詳細資訊，請參閱[開啟連接埠和端點](../nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

當您在 Azure 入口網站中建立 Linux 虛擬機器時，通常會自動為您建立安全殼層 (SSH) 的端點。 建立虛擬機器或日後有需要時，您可以設定其他端點。

[!INCLUDE [virtual-machines-common-classic-setup-endpoints](../../../../includes/virtual-machines-common-classic-setup-endpoints.md)]

## <a name="next-steps"></a>後續步驟
* 您也可以使用 [Azure 命令列介面](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)建立 VM 端點。 執行 **azure vm endpoint create** 命令。
* 如果您已在 Resource Manager 部署模型中建立虛擬機器，您可以使用 Resource Manager 模式中的 Azure CLI 來 [建立網路安全性群組](../../../virtual-network/tutorial-filter-network-traffic-cli.md) 以控制 VM 的流量。
