---
title: 對傳統部署模型中的 Azure VM 配置失敗進行疑難排解| Microsoft Docs
description: 在 Azure 中建立、重新啟動或調整傳統 VM 大小時，對配置失敗進行疑難排解
services: azure-service-management
documentationcenter: ''
author: genlin
manager: willchen
editor: ''
tags: top-support-issue,azure-resource-manager,azure-service-management
ms.assetid: bb939e23-77fc-4948-96f7-5037761c30e8
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 04/13/2018
ms.author: genli
ms.openlocfilehash: 786957f8b6aa8721a0613260f6c4a6ec6e06ef45
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411761"
---
# <a name="troubleshooting-steps-specific-to-allocation-failure-scenarios-in-the-classic-deployment-model"></a>傳統部署模型中配置失敗案例的具體疑難排解步驟

以下是造成釘選配置要求的常見配置案例。 我們將在本文稍後深入探討每一個案例。

- 調整 VM 大小，或將 VM 或角色執行個體加入至現有的雲端服務
- 重新啟動已部分停止 (已取消配置) 的 VM
- 重新啟動已完全停止 (已取消配置) 的 VM
- 預備和生產部署 (僅適用於平台即服務)
- 同質群組 (VM 或服務鄰近性)
- 以同質群組為基礎的虛擬網路

發生配置錯誤時，請檢查是否有任何列出的案例符合您所遇到的錯誤。 使用 Azure 平台傳回的配置錯誤來識別對應的案例。 如果您的要求已釘選，請移除一些釘選條件約束，向更多叢集開放您的要求，以提高配置成功的機會。
一般而言，如果錯誤並未陳述「不支援所要求的 VM 大小」，您一律都可以稍後再重試。 這是因為到時叢集可能已釋放足夠的資源來滿足您的要求。 如果問題在於不支援所要求的 VM 大小，請嘗試不同的 VM 大小。 否則，唯一的選項便是將釘選條件約束移除。

有兩個常見的失敗案例與同質群組有關。 在過去，同質群組是用來提供 VM 和服務執行個體的鄰近性，或用來啟用虛擬網路的建立。 在引進區域虛擬網路之後，建立虛擬網路已不再需要同質群組。 由於 Azure 基礎結構中的網路延遲縮短，原本建議使用同質群組來支援 VM 和服務鄰近性的情況已有所改變。

下圖顯示 (釘選的) 配置案例分類。 

![釘選配置分類](./media/virtual-machines-common-allocation-failure/Allocation3.png)

## <a name="resize-a-vm-or-add-vms-or-role-instances-to-an-existing-cloud-service"></a>調整 VM 大小，或將 VM 或角色執行個體加入至現有的雲端服務
**錯誤**

Upgrade_VMSizeNotSupported 或 GeneralError

**叢集釘選的原因**

必須在託管現有雲端服務的原始叢集上，嘗試要求調整 VM 大小，或將 VM 或角色執行個體加入現有的雲端服務。 建立新的雲端服務可讓 Azure 平台尋找另一個擁有可用資源，或支援您所要求之 VM 大小的叢集。

**因應措施**

如果錯誤是 Upgrade_VMSizeNotSupported*，請嘗試不同的 VM 大小。 如果使用不同的 VM 大小不可行，但可接受使用不同的虛擬 IP 位址 (VIP)，請建立新的雲端服務來裝載新的 VM，並將新的雲端服務加入至執行現有 VM 的區域虛擬網路。 如果現有的雲端服務不使用區域虛擬網路，您仍然可以為新的雲端服務建立新的虛擬網路，然後將 [現有的虛擬網路連接到新的虛擬網路](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/)。 深入了解 [區域虛擬網路](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/)。

如果錯誤是 GeneralError*，很可能是叢集支援資源的類型 (例如特定的 VM 大小)，但叢集目前沒有可用的資源。 類似上述案例，透過建立新的雲端服務 (請注意，新的雲端服務必須使用不同的 VIP) 新增所需的計算資源，並使用區域虛擬網路連接您的雲端服務。

## <a name="restart-partially-stopped-deallocated-vms"></a>重新啟動已部分停止 (已取消配置) 的 VM
**錯誤**

GeneralError*

**叢集釘選的原因**

部分解除配置表示您已停止 (已取消配置) 雲端服務中的一或多部 VM，而不是所有 VM。 停止 (取消配置) VM 時會釋放相關聯的資源。 因此，重新啟動已停止 (已取消配置) 的 VM 是一項新的配置要求。 在部分已取消配置的雲端服務中重新啟動 VM，相當於將 VM 加入現有的雲端服務。 必須在裝載現有雲端服務的原始叢集上嘗試提出配置要求。 另外建立一個雲端服務可讓 Azure 平台尋找另一個有可用資源，或支援您所要求之 VM 大小的叢集。

**因應措施**

如果可接受使用不同的 VIP，請刪除已停止 (已取消配置) 的 VM (但保留相關聯的磁碟)，並透過不同的雲端服務重新加回 VM。 使用區域虛擬網路連接您的雲端服務：

* 如果現有的雲端服務使用區域虛擬網路，只要將新的雲端服務加入至相同的虛擬網路即可。
* 如果現有的雲端服務不使用區域虛擬網路，請為新的雲端服務建立新的虛擬網路，然後將 [現有的虛擬網路連接到新的虛擬網路](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/)。 深入了解 [區域虛擬網路](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/)。

## <a name="restart-fully-stopped-deallocated-vms"></a>重新啟動已完全停止 (已取消配置) 的 VM
**錯誤**

GeneralError*

**叢集釘選的原因**

完整取消配置表示您已停止 (已取消配置) 雲端服務上的所有 VM。 必須在裝載雲端服務的原始叢集上嘗試提出配置要求以重新啟動這些 VM。 建立新的雲端服務可讓 Azure 平台尋找另一個擁有可用資源，或支援您所要求之 VM 大小的叢集。

**因應措施**

如果可接受使用不同的 VIP，請刪除原始已停止 (已取消配置) 的 VM (但保留相關聯的磁碟)，並刪除對應的雲端服務 (已停止 (已取消配置) VM 時，就已釋放相關聯的計算資源)。 建立新的雲端服務以加回 VM。

## <a name="stagingproduction-deployments-platform-as-a-service-only"></a>預備/生產部署 (僅適用於平台即服務)
**錯誤**

New_General* 或 New_VMSizeNotSupported*

**叢集釘選的原因**

雲端服務的預備環境部署和生產環境部署裝載於相同的叢集。 新增第二個部署時，將會在裝載第一個部署的相同叢集中嘗試提出對應的配置要求。

**因應措施**

請刪除第一個部署和原始的雲端服務，然後重新部署雲端服務。 這個動作可能將第一個部署安排到有足夠可用資源可滿足這兩個部署的叢集，或安排到支援所要求 VM 大小的叢集。

## <a name="affinity-group-vmservice-proximity"></a>同質群組 (VM/服務鄰近性)
**錯誤**

New_General* 或 New_VMSizeNotSupported*

**叢集釘選的原因**

任何指派給同質群組的計算資源都繫結至一個叢集。 該同質群組中新的計算資源要求，將於裝載現有資源的相同叢集中嘗試提出。 無論新的資源是透過新的雲端服務，或是透過現有的雲端服務來建立，都是如此。

**因應措施**

如果同質群組不是必要的，請勿使用同質群組或將計算資源分組為多個同質群組。

## <a name="affinity-group-based-virtual-network"></a>同質群組式虛擬網路
**錯誤**

New_General* 或 New_VMSizeNotSupported*

**叢集釘選的原因**

在導入區域虛擬網路之前，您必須先將虛擬網路與同質群組產生關聯。 如此一來，將會依上一節「配置案例：同質群組 (VM/服務鄰近性)」所述的相同條件約束，繫結放入同質群組中的計算資源。 計算資源會繫結至單一叢集。

**因應措施**

如果您不需要同質群組，請為您要加入的新資源建立新的區域虛擬網路，然後 [將現有的虛擬網路連接到新的虛擬網路](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/)。 深入了解 [區域虛擬網路](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/)。

此外，您也可以 [將同質群組式虛擬網路移轉至區域虛擬網路](https://azure.microsoft.com/blog/2014/11/26/migrating-existing-services-to-regional-scope/)，然後再重新加入需要的資源。


