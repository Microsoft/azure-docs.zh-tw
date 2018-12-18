---
title: 在 Azure Site Recovery 中對應兩個 Azure 區域間的虛擬網路 | Microsoft Docs
description: Azure Site Recovery 可協調虛擬機器和實體伺服器的複寫、容錯移轉及復原作業。 了解如何容錯移轉到 Azure 或次要資料中心。
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
editor: ''
ms.assetid: 44813a48-c680-4581-a92e-cecc57cc3b1e
ms.service: site-recovery
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: 95e6a388d0638d2fd477d33aaf7c39cf120e29aa
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353421"
---
# <a name="map-virtual-networks-in-different-azure-regions"></a>對應不同 Azure 區域中的虛擬網路


本文說明如何彼此對應位於不同 Azure 區域之 Azure 虛擬網路的兩個執行個體。 網路對應可確保在目標 Azure 區域中建立複寫的虛擬機器時，也會在與來源虛擬機器之虛擬網路對應的虛擬網路上建立虛擬機器。  

## <a name="prerequisites"></a>必要條件
對應網路之前，請確定您已在來源區域和目標 Azure 區域中建立 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)。

## <a name="map-virtual-networks"></a>對應虛擬網路

若要將位於一個 Azure 區域的 Azure 虛擬網路 (來源網路) 對應至位於另一個區域的虛擬網路 (目標網路)，針對 Azure 虛擬機器，移至 [Site Recovery 基礎結構] > [網路對應]。 建立網路對應。

![[網路對應] 視窗 - 建立網路對應](./media/site-recovery-network-mapping-azure-to-azure/network-mapping1.png)


在下列範例中，虛擬機器正在東亞地區中執行。 正在將虛擬機器複寫至東南亞地區。

若要建立從東亞地區到東南亞地區的網路對應，請選取來源網路的位置及目標網路的位置。 然後選取 [確定]。

![[新增網路對應] 視窗 - 選取來源網路的來源和目標位置](./media/site-recovery-network-mapping-azure-to-azure/network-mapping2.png)


重複上述程序，以建立從東南亞地區到東亞地區的網路對應。

![[新增網路對應] 窗格 - 選取目標網路的來源和目標位置](./media/site-recovery-network-mapping-azure-to-azure/network-mapping3.png)


## <a name="map-a-network-when-you-enable-replication"></a>啟用複寫時對應網路

當您第一次將虛擬機器從一個 Azure 區域複製到另一個區域時，如果沒有網路對應，您可以在設定複寫時設定目標網路。 Azure Site Recovery 會根據此設定，建立從來源區域到目標區域以及從目標區域到來源區域的網路對應。   

![[組態設定] 窗格 - 選擇目標位置](./media/site-recovery-network-mapping-azure-to-azure/network-mapping4.png)

根據預設，Site Recovery 會在目標區域中建立與來源網路完全相同的網路。 Site Recovery 會藉由新增 **-asr** 作為來源網路的名稱尾碼來建立網路。 若要選擇已建立的網路，請選取 [自訂]。

![[自訂目標設定] 窗格 - 設定目標資源群組名稱和目標虛擬網路名稱](./media/site-recovery-network-mapping-azure-to-azure/network-mapping5.png)

如果已發生網路對應，您就無法在啟用複寫時變更目標虛擬網路。 在此情況下，若要變更目標虛擬網路，請修改現有的網路對應。  

![[自訂目標設定] 窗格 - 設定目標資源群組名稱](./media/site-recovery-network-mapping-azure-to-azure/network-mapping6.png)

![[修改網路對應] 窗格 - 修改現有的目標虛擬網路名稱](./media/site-recovery-network-mapping-azure-to-azure/modify-network-mapping.png)

> [!IMPORTANT]
> 如果您修改從區域 A 到區域 B 的網路對應，請確定也會修改從區域 B 到區域 A 的網路對應。
>
>


## <a name="subnet-selection"></a>子網路選取項目
目標虛擬機器的子網路會根據來源虛擬機器的子網路名稱來選取。 如果目標網路中可用的子網路具備與來源虛擬機器相同的名稱，則會針對目標虛擬機器設定該子網路。 如果目標網路中沒有相同名稱的子網路，則會依字母順序設定第一個子網路作為目標子網路。

若要修改子網路，請移至虛擬機器的 [計算與網路] 設定。

![[計算與網路] 計算屬性視窗](./media/site-recovery-network-mapping-azure-to-azure/modify-subnet.png)


## <a name="ip-address"></a>IP 位址

以下列各節所述的方式，針對目標虛擬機器的每個網路介面設定 IP 位址。

### <a name="dhcp"></a>DHCP
如果來源虛擬機器的網路介面使用 DHCP，也會將目標虛擬機器的網路介面設定為使用 DHCP。

### <a name="static-ip-address"></a>靜態 IP 位址
如果來源虛擬機器的網路介面使用靜態 IP 位址，也會將目標虛擬機器的網路介面設定為使用靜態 IP 位址。 下列各節說明如何設定靜態 IP 位址。

### <a name="ip-assignment-behavior-during-failover"></a>容錯移轉期間的 IP 指派行為
#### <a name="1-same-address-space"></a>1.相同的位址空間

如果來源子網路和目標子網路具有相同的位址空間，會將來源虛擬機器之網路介面的 IP 位址設定為目標 IP 位址。 如果無法使用相同的 IP 位址，則會設定下一個可用的 IP 位址作為目標 IP 位址。

#### <a name="2-different-address-spaces"></a>2.不同的位址空間

如果來源子網路和目標子網路具有不同的位址空間，會將目標子網路的下一個可用 IP 位址設定為目標 IP 位址。


### <a name="ip-assignment-behavior-during-test-failover"></a>測試容錯移轉期間的 IP 指派行為
#### <a name="1-if-the-target-network-chosen-is-the-production-vnet"></a>1.若選擇的目標網路是生產 vNet
- 復原 IP (目標 IP) 將會靜態 IPM，但它**將不會與針對容錯移轉保留的 IP 位址相同**。
- 指派的 IP 位址將會是從子網路位址範圍結尾開始的下一個可用 IP。
- 例如，若來源 VM 靜態 IP 是設定為 10.0.0.19 且測試容錯移轉已使用設定的生產網路 ***dr-PROD-nw*** (子網路範圍為 10.0.0.0/24) 嘗試。 </br>
已容錯移轉的 VM 將會被指派從子網路位址範圍結尾開始的下一個可用 IP，亦即 10.0.0.254。 </br>

**注意：** 術語**生產 vNet** 指的是災害復原設定期間對應的「目標網路」。
#### <a name="2-if-the-target-network-chosen-is-not-the-production-vnet-but-has-the-same-subnet-range-as-production-network"></a>2.若選擇的目標網路不是生產 vNet 但有與生產網路相同的子網路範圍

- 復原 IP (目標 IP) 將具有與針對容錯移轉保留之 **IP 位址** 相同的 IP 位址 (例如，已設定的靜態 IP 位址)。 假設相同的 IP 位址可用。
- 若設定的靜態 IP 已指派給其他 VM/裝置，則復原 IP 將會是從子網路位址範圍結尾開始的下一個可用 IP。
- 例如，若來源 VM 靜態 IP 是設定為 10.0.0.19 且測試容錯移轉已使用測試網路 ***dr-NON-PROD-nw*** (具有與生產網路相同的子網路範圍，亦即 10.0.0.0/24) 嘗試。 </br>
  已容錯移轉的 VM 將會被指派下列靜態 IP </br>
    - 已設定的 IP：10.0.0.19，若此 IP 可用。
    - 下一個可用的 IP：10.0.0.254，若 IP 位址 10.0.0.19 已在使用中。


若要修改各個網路介面上的目標 IP，請移至虛擬機器的 [計算與網路] 設定。</br>
最佳做法是一律選擇測試網路以執行測試容錯移轉。
## <a name="next-steps"></a>後續步驟

* 檢閱[複寫 Azure 虛擬機器的網路指引](site-recovery-azure-to-azure-networking-guidance.md)。
