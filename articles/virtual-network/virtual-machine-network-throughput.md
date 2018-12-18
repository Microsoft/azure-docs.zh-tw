---
title: Azure 虛擬機器網路輸送量 | Microsoft Docs
description: 了解 Azure 虛擬機器網路輸送量。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/13/2017
ms.author: jdial
ms.openlocfilehash: f22b6f361f0c5bea547721309bb0f75b62f18d92
ms.sourcegitcommit: 562a537ed9b96c9116c504738414e5d8c0fd53b1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2018
ms.locfileid: "27778950"
---
# <a name="virtual-machine-network-bandwidth"></a>虛擬機器網路頻寬

Azure 提供各種 VM 大小和類型，各有不同的效能功能組合。 其中一個功能是網路輸送量 (或頻寬)，測量單位是 MB/秒 (Mbps)。 因為虛擬機器裝載於共用硬體，網路容量必須平均分給共用相同硬體的虛擬機器。 相對於較小的虛擬機器，大型的虛擬機器會配置較多的頻寬。
 
配置給每個虛擬機器的網路頻寬是依據從虛擬機器的流出 (輸出) 流量來計量。 所有離開虛擬機器的網路流量都會計算到配置的限制，不會因為目的地不同而有差異。 例如，如果虛擬機器有 1,000 Mbps 的限制，則無論輸出流量的目的地是相同虛擬網路中的另一部虛擬機器，或是 Azure 之外的虛擬機器，皆會套用該限制。
 
輸入則不會直接計量或限制。 不過，還有其他因素，例如 CPU 和儲存體限制可能會影響虛擬機器處理內送資料的能力。

加速網路是設計來改善網路效能 (包括延遲、輸送量和 CPU 使用率) 的功能。 雖然加速網路可提升虛擬機器的輸送量，但不能超過虛擬機器的配置頻寬。 若要深入了解加速網路，請參閱適用於 [Windows](create-vm-accelerated-networking-powershell.md) 或 [Linux](create-vm-accelerated-networking-cli.md) 虛擬機器的＜加速網路＞。
 
Azure 虛擬機器必須連結一個 (但可以有數個) 網路介面。 配置給虛擬機器的頻寬，是虛擬機器上所有網路介面的所有輸出流量總和。 換句話說，配置的頻寬是以每個虛擬機器為依據，跟連結多少個網路介面到虛擬機器無關。 若要了解不同 Azure 虛擬機器大小支援的網路介面有多少，請參閱 Azure [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 和 [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虛擬機器大小。 

## <a name="expected-network-throughput"></a>預期的網路輸送量

預期的輸出輸送量和每個虛擬機器大小支援的網路介面數目，皆詳細列在 Azure [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 和 [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虛擬機器大小中。 選取類型，例如一般用途，然後在產生的頁面上選取大小的系列，例如 Dv2 系列。 每個數列都有資料表，且最後一個資料行包含網路規格，標題為**最大 NIC 數 / 預期的網路效能 (Mbps)**。 

輸送量限制會套用至虛擬機器。 輸送量不會受到下列因素影響：
- **網路介面數目**：頻寬限制是虛擬機器所有輸出流量的累計。
- **加速網路**：雖然此功能有助於達到已發佈的限制，但不會變更限制。
- **流量目的地**：所有目的地都會計入輸出限制。
- **通訊協定**：通過所有通訊協定的所有輸出流量都會計入限制。

## <a name="next-steps"></a>後續步驟

- [最佳化虛擬機器作業系統的網路輸送量](virtual-network-optimize-network-bandwidth.md)
- 針對虛擬機器，[測試網路輸送量](virtual-network-bandwidth-testing.md)。
