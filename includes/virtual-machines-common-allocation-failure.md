---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows, azure-resource-manager
author: genlin
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 04/14/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: 2eb7fb82b358d4ec8628bfa546b572ee3cbe47fa
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40210093"
---
建立虛擬機器 (VM)、重新啟動已停止 (已解除配置) 的 VM 或調整 VM 大小時，Microsoft Azure 會配置計算資源給您的訂用帳戶。 我們正持續投資額外的基礎結構和功能，以確保始終擁有所有 VM 類型可供支援客戶需求。 不過，您可能偶爾會遇到因特定區域中對 Azure 服務的空前需求成長，而導致資源配置失敗的情況。 當您嘗試在 VM 顯示下列錯誤碼和訊息的區域中建立或啟動 VM，便可能發生此問題：

**錯誤碼**：AllocationFailed 或 ZonalAllocationFailed

**錯誤訊息**：「配置失敗。 我們在此區域沒有足夠的容量可供要求的 VM 大小使用。 如需了解如何提高配置成功率，請參閱 http://aka.ms/allocation-guidance」

本文說明一些常見的配置失敗原因，並建議可能的補救方法。

若本文中未提及您的 Azure 問題，請前往 [MSDN 及 Stack Overflow 上的 Azure 論壇](https://azure.microsoft.com/support/forums/)。 您可以在這些論壇上張貼您的問題，或將問題貼到 Twitter 上的 @AzureSupport。 此外，您也可以在 [Azure 支援](https://azure.microsoft.com/support/options/)網站上選取 [取得支援]，來提出 Azure 支援要求。

在您的慣用區域提供您的慣用 VM 類型之前，我們會建議遇到部署問題的客戶考慮採用下表中的指引作為暫時的因應措施。 

請識別與您案例最相符的情況，然後使用對應的建議因應措施來重試配置要求，以提高配置成功率。 或者，您也一律可以稍後再重試。 這是因為叢集、區域或地區屆時可能已釋出足夠的資源來滿足您的要求。 


## <a name="resize-a-vm-or-add-vms-to-an-existing-availability-set"></a>調整 VM 大小或將 VM 新增至現有的可用性設定組

### <a name="cause"></a>原因

必須在裝載現有可用性設定組的原始叢集上，嘗試要求調整 VM 大小，或將 VM 新增至現有的可用性設定組。 叢集支援所要求的 VM 大小，但叢集可能目前沒有足夠的容量。 

### <a name="workaround"></a>因應措施

如果 VM 可以屬於不同的可用性設定組，請在不同的可用性設定組 (位於相同區域) 中建立 VM。 然後，這個新的 VM 就可以加入至相同的虛擬網路。

停止 (解除配置) 相同可用性設定組中的所有 VM，然後重新啟動每一部 VM。
若要停止，請按一下 [資源群組] > [您的資源群組] > [資源] > [您的可用性設定組] > [虛擬機器] > [您的虛擬機器] > [停止]。
所有 VM 都停止之後，請選取第一個 VM，然後按一下 [啟動]。
此步驟可確保執行新的配置嘗試，而且可以選取有足夠容量的新叢集。

## <a name="restart-partially-stopped-deallocated-vms"></a>重新啟動已部分停止 (已取消配置) 的 VM

### <a name="cause"></a>原因

部分解除配置表示您已停止 (已取消配置) 可用性設定組中的一或多部 VM，而不是所有 VM。 當您將 VM 解除配置時，會釋出相關聯的資源。 在已部分解除配置的可用性設定組中重新啟動 VM，與將 VM 新增至現有的可用性設定組一樣。 因此，必須在裝載現有可用性設定組但可能容量不足的原始叢集上，嘗試提出配置要求。

### <a name="workaround"></a>因應措施

停止 (解除配置) 相同可用性設定組中的所有 VM，然後重新啟動每一部 VM。
若要停止，請按一下 [資源群組] > [您的資源群組] > [資源] > [您的可用性設定組] > [虛擬機器] > [您的虛擬機器] > [停止]。
所有 VM 都停止之後，請選取第一個 VM，然後按一下 [啟動]。
這可確保執行新的配置嘗試，而且可以選取有足夠容量的新叢集。

## <a name="restart-fully-stopped-deallocated-vms"></a>重新啟動已完全停止 (已取消配置) 的 VM

### <a name="cause"></a>原因

完整取消配置表示您已停止 (已取消配置) 可用性設定組中的所有 VM。 提出配置要求來重新啟動這些 VM 時，將會以區域或地區內支援所需大小的所有叢集為目標。 請依據本文中的建議變更您的配置要求，然後重試要求以提高配置成功率。 

### <a name="workaround"></a>因應措施

如果您使用較舊版的 VM 系列或大小 (例如 Dv1、DSv1、Av1、D15v2 或 DS15v2)，請考慮移轉至較新的版本。 請參閱這些適用於特定 VM 大小的建議。
如果您無法選擇使用不同的 VM 大小，請嘗試部署至相同地區內的不同區域。 如需有關每個區域中可用 VM 大小的詳細資訊，請參閱 https://aka.ms/azure-regions

如果您使用可用性區域，請嘗試該區域內另一個可能有可用容量可供所要求 VM 大小使用的區域。

如果您的配置要求相當大 (超過 500 個核心)，請參閱下列小節中的指引，將要求分解成較小的部署。

## <a name="allocation-failures-for-older-vm-sizes-av1-dv1-dsv1-d15v2-ds15v2-etc"></a>較舊版 VM 大小 (Av1、Dv1、DSv1、D15v2、DS15v2 等) 的配置失敗

隨著我們擴充 Azure 基礎結構，我們會部署更新一代、專為支援最新虛擬機器類型而設計的硬體。 有些較舊系列的 VM 無法在我們最新一代的基礎結構上執行。 因此，客戶可能偶爾會遇到這些舊版 SKU 的配置失敗問題。 為了避免這個問題，我們鼓勵使用舊版系列虛擬機器的客戶考慮依據下列建議移轉至對等的較新版 VM：這些 VM 已針對最新硬體進行最佳化，將可讓您享有更好的定價和效能。 

|傳統 VM 系列/大小|建議的較新版 VM 系列/大小|詳細資訊|
|----------------------|----------------------------|--------------------|
|Av1 系列|[Av2 系列](../articles/virtual-machines/windows/sizes-general.md#av2-series)|https://azure.microsoft.com/blog/new-av2-series-vm-sizes/
|Dv1 或 DSv1 系列 (D1 到 D5)|[Dv3 或 DSv3 系列](../articles/virtual-machines/windows/sizes-general.md#dsv3-series-sup1sup)|https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/
|Dv1 或 DSv1 系列 (D11 到 D14)|[Ev3 或 ESv3 系列](../articles/virtual-machines/windows/sizes-memory.md#esv3-series)|
|D15v2 或 DS15v2|如果您使用 Resource Manager 部署模型來利用較大的 VM 大小，請考慮移轉至 E16v3/E16sv3 或 E32v3/E32sv3。 這些 VM 是專為在最新一代硬體上執行而設計。 如果您使用 Resource Manager 部署模型來確保 VM 執行個體與單一客戶專用的硬體隔離，請考慮移轉至新的隔離式 VM 大小 (E64i_v3 或 E64is_v3)，這些 VM 是專為在最新一代硬體上執行而設計。 |https://azure.microsoft.com/blog/new-isolated-vm-sizes-now-available/


## <a name="allocation-failures-for-large-deployments-more-than-500-cores"></a>大型部署 (超過 500 個核心) 的配置失敗

請縮減所要求 VM 大小的執行個體數目，然後重試部署作業。 此外，針對較大型的部署，您可以評估 [Azure 虛擬機器擴展集](https://docs.microsoft.com/azure/virtual-machine-scale-sets/)。 VM 執行個體的數目可以因應需求或已定義的排程來自動增加或減少，您的配置成功率將可提高，因為可以將部署分散到多個叢集。 

## <a name="background-information"></a>背景資訊
### <a name="how-allocation-works"></a>配置的運作方式
Azure 資料中心的伺服器分割成叢集。 通常會嘗試向多個叢集提出配置要求，但配置要求可能帶有某些條件約束，而強制 Azure 平台只嘗試向一個叢集提出要求。 在本文中，這種情況稱為「釘選到叢集」。 下圖 1 說明於嘗試向多個叢集提出一般配置的情況。 圖 2 說明釘選到叢集 2 的配置案例，因為叢集 2 是現有雲端服務 CS_1 或可用性設定組的裝載位置。
![配置圖表](./media/virtual-machines-common-allocation-failure/Allocation1.png)

### <a name="why-allocation-failures-happen"></a>配置失敗的原因
當配置要求已釘選到叢集時，由於可用的資源集區較小，很可能找不到可用的資源。 此外，如果配置要求已釘選到叢集，但該叢集不支援您所要求的資源類型，即使叢集有可用的資源，您的要求仍會失敗。 下面圖 3 說明因為唯一候選叢集沒有可用的資源，而導致已釘選的配置發生失敗的情況。 圖 4 說明因唯一候選叢集不支援所要求的 VM 大小 (雖然叢集有可用的資源)，而導致已釘選的配置失敗的情況。

![釘選配置失敗](./media/virtual-machines-common-allocation-failure/Allocation2.png)


