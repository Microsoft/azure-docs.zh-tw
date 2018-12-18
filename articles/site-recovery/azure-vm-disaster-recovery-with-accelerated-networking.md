---
title: 加速網路與 Azure 虛擬機器災害復原 | Microsoft Docs
description: 說明如何啟用加速網路與 Azure Site Recovery，以進行 Azure 虛擬機器災害復原
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: bb4a3f433f213984c4c351439760ddd3327cda73
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353131"
---
# <a name="accelerated-networking-with-azure-virtual-machine-disaster-recovery"></a>加速網路與 Azure 虛擬機器災害復原

加速網路可以對 VM 啟用單一根目錄 I/O 虛擬化 (SR-IOV)，大幅提升其網路效能。 這個高效能路徑會略過資料路徑的主機，進而減少延遲、抖動和 CPU 使用率，供支援的 VM 類型中最嚴苛的網路工作負載使用。 下圖顯示兩部 VM 之間的通訊，一部具備加速網路而另一步沒有︰

![比較](./media/azure-vm-disaster-recovery-with-accelerated-networking/accelerated-networking-benefit.png)

Azure Site Recovery 可讓您針對容錯移轉至不同 Azure 區域的 Azure 虛擬機器，利用加速網路的優點。 本文說明如何針對使用 Azure Site Recovery 所複寫的 Azure 虛擬機器，啟用加速網路。

## <a name="prerequisites"></a>必要條件

開始之前，請確定您了解：
-   Azure 虛擬機器[複寫架構](azure-to-azure-architecture.md)
-   針對 Azure 虛擬機器[設定複寫](azure-to-azure-tutorial-enable-replication.md)
-   [容錯移轉](azure-to-azure-tutorial-failover-failback.md) Azure 虛擬機器

## <a name="accelerated-networking-with-windows-vms"></a>使用 Windows VM 的加速網路

Azure Site Recovery 只有在來源虛擬機器已啟用加速網路時，才支援針對已複寫虛擬機器啟用加速網路。 如果您的來源虛擬機器未啟用加速網路，您可以在[這裡](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms)深入了解如何針對 Windows 虛擬機器啟用加速網路。

### <a name="supported-operating-systems"></a>受支援的作業系統
您可以從 Azure 資源庫直接使用下列發行版本：
* **Windows Server 2016 Datacenter**
* **Windows Server 2012 R2 Datacenter**

### <a name="supported-vm-instances"></a>支援的 VM 執行個體
大多數一般用途和具有 2 個以上 vCPU 的計算最佳化執行個體大小，皆支援加速網路。  這些支援的系列為：D/DSv2 和 F/Fs

在支援超執行緒的執行個體中，加速網路可在具有 4 個以上 vCPU 的 VM 執行個體上進行支援作業。 支援的系列為：D/DSv3、E/ESv3、Fsv2 和 Ms/Mms

如需 VM 執行個體的詳細資訊，請參閱 [Windows VM 大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="accelerated-networking-with-linux-vms"></a>使用 Linux VM 的加速網路

Azure Site Recovery 只有在來源虛擬機器已啟用加速網路時，才支援針對已複寫虛擬機器啟用加速網路。 如果您的來源虛擬機器未啟用加速網路，您可以在[這裡](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms)深入了解如何針對 Linux 虛擬機器啟用加速網路。

### <a name="supported-operating-systems"></a>受支援的作業系統
您可以從 Azure 資源庫直接使用下列發行版本：
* **Ubuntu 16.04**
* **SLES 12 SP3**
* **RHEL 7.4**
* **CentOS 7.4**
* **CoreOS Linux**
* **具有反向移植核心的 Debian "Stretch"**
* **Oracle Linux 7.4**

### <a name="supported-vm-instances"></a>支援的 VM 執行個體
大多數一般用途和具有 2 個以上 vCPU 的計算最佳化執行個體大小，皆支援加速網路。  這些支援的系列為：D/DSv2 和 F/Fs

在支援超執行緒的執行個體中，加速網路可在具有 4 個以上 vCPU 的 VM 執行個體上進行支援作業。 支援的系列為：D/DSv3、E/ESv3、Fsv2 和 Ms/Mms。

如需 VM 執行個體的詳細資訊，請參閱 [Linux VM 大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="enabling-accelerated-networking-for-replicated-vms"></a>針對已複寫 VM 啟用加速網路

當您針對 Azure 虛擬機器[啟用複寫](azure-to-azure-tutorial-enable-replication.md)時，Site Recovery 會自動偵測虛擬機器網路介面是否已啟用加速網路。 如果已啟用加速網路，Site Recovery 會自動在已複寫虛擬機器網路介面上設定加速網路。

加速網路的狀態，可以在已複寫虛擬機器 [計算與網路] 設定的 [網路介面] 區段底下確認。

![加速網路設定](./media/azure-vm-disaster-recovery-with-accelerated-networking/compute-network-accelerated-networking.png)

如果您在啟用複寫之後，在來源虛擬機器上啟用加速網路，可以藉由下列程序，針對已複寫虛擬機器網路介面啟用加速網路：
1. 開啟已複寫虛擬機器的 [計算與網路] 設定
2. 在 [網路介面] 區段底下，按一下網路介面的名稱
3. 從 [目標] 資料行底下的加速網路下拉式清單，選取 [已啟用]

![啟用加速網路](./media/azure-vm-disaster-recovery-with-accelerated-networking/network-interface-accelerated-networking-enabled.png)

現有已複寫虛擬機器 (Site Recovery 先前未自動啟用加速網路) 也應該遵循上述程序。

## <a name="next-steps"></a>後續步驟
- 深入了解[加速網路的優點](../virtual-network/create-vm-accelerated-networking-powershell.md#benefits)。
- 深入了解適用於 [Windows 虛擬機器](../virtual-network/create-vm-accelerated-networking-powershell.md#limitations-and-constraints)和 [Linux 虛擬機器](../virtual-network/create-vm-accelerated-networking-cli.md#limitations-and-constraints)的加速網路限制。
- 深入了解自動執行應用程式容錯移轉的[復原計劃](site-recovery-create-recovery-plans.md)。
