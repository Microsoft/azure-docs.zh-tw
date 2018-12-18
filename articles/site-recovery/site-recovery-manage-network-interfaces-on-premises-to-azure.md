---
title: 在 Azure Site Recovery 中管理內部部署至 Azure 複寫的網路介面 | Microsoft Docs
description: 描述如何利用 Azure Site Recovery 管理內部部署至 Azure 複寫的網路介面
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: 0ca2d32910e57042d2cbc9d3b772d226d835b545
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352405"
---
# <a name="manage-virtual-machine-network-interfaces-for-on-premises-to-azure-replication"></a>管理內部部署至 Azure 複寫的虛擬機器網路介面

Azure 中的虛擬機器 (VM) 必須至少擁有一個連接的網路介面。 可連接的網路介面數量上限則不盡相同，須視 VM 大小支援的數量而定。

根據預設，系統會將 Azure 虛擬機器連接的第一個網路介面定義為主要網路介面。 虛擬機器中的其他所有網路介面均為次要網路介面。 除此之外，系統在傳送來自虛擬機器的所有輸出流量時，預設都會送出指派給主要網路介面之主要 IP 組態的 IP 位址。

在內部部署環境中，虛擬機器或伺服器可以使用多個網路介面來因應環境中不同網路的不同需求。 不同的網路通常有不同的用途，它們能用來執行升級、維護及網際網路存取等特定作業。 當您要從內部部署環境移轉或容錯移轉到 Azure 時，請務必讓同一部虛擬機器中的所有網路介面連接同一個虛擬網路。

根據預設，Azure Site Recovery 在 Azure 虛擬機器中建立的網路介面數量，等同於虛擬機器與內部部署伺服器的連線數量。 您可以編輯複寫虛擬機器的網路介面設定，避免在移轉或容錯移轉期間建立多餘的網路介面。

## <a name="select-the-target-network"></a>選取目標網路

針對 VMware 和實體機器，以及 Hyper-V (沒有 System Center Virtual Machine Manager) 虛擬機器，您可以為個別的虛擬機器指定目標虛擬網路。 針對以 Virtual Machine Manager 管理的 Hyper-V 虛擬機器，請使用[網路對應](site-recovery-network-mapping.md)來對應來源 Virtual Machine Manager 伺服器上的 VM 網路與目標 Azure 網路。

1. 在復原服務保存庫的 [複寫的項目]下方，選取任何複寫項目以存取該複寫項目的設定。

2. 選取 [計算和網路] 索引標籤以存取複寫項目的網路設定。

3. 在 [網路屬性] 下方，從可用網路介面清單中選擇虛擬網路。

    ![網路設定](./media/site-recovery-manage-network-interfaces-on-premises-to-azure/compute-and-network.png)

修改目標網路會影響該特定虛擬機器的所有網路介面。

對於 Virtual Machine Manager 雲端，修改網路對應會影響所有虛擬機器與其網路介面。

## <a name="select-the-target-interface-type"></a>選取目標介面類型

在 [計算和網路] 窗格的 [網路介面] 區段下方，您可以檢視和編輯網路介面設定， 也可以指定目標網路類型。

- **主要**網路介面是容錯移轉的必要項目。
- 其他所有選取的網路介面 (若有的話) 均為**次要**網路介面。
- 選取 [不使用] 可排除網路介面，避免在容錯移轉時建立。

根據預設，當您啟用複寫時，Site Recovery 會選取在內部部署伺服器上偵測到的所有網路介面。 它會將某個網路介面標示為**主要**，將其他所有網路介面標示為**次要**。 後續在內部部署伺服器上新增的介面，預設會標示為**不使用**。 當您新增更多網路介面時，請務必選取正確的 Azure 虛擬機器目標大小，以容納所有必要的網路介面。

## <a name="modify-network-interface-settings"></a>修改網路介面設定

您可以修改複寫項目網路介面的子網路和 IP 位址。 如果您未指定 IP 位址，Site Recovery 會在容錯移轉時將子網路中下一個可用的 IP 位址指派給網路介面。

1. 選取任一個可用的網路介面以開啟網路介面設定。

2. 從可用的子網路清單中選擇所需的子網路。

3. 輸入所需的 IP 位址 (視需要)。

    ![網路介面設定](./media/site-recovery-manage-network-interfaces-on-premises-to-azure/network-interface-settings.png)

4. 選取 [確定] 以完成編輯並返回 [計算和網路] 窗格。

5. 重複步驟 1-4 以修改其他網路介面的設定。

6. 選取 [儲存] 以儲存所有變更。

## <a name="next-steps"></a>後續步驟
  [深入了解](../virtual-network/virtual-network-network-interface-vm.md) Azure 虛擬機器的網路介面。
