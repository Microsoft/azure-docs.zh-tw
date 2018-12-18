---
title: Azure 虛擬機器擴展集概觀 | Microsoft Docs
description: 了解 Azure 虛擬機器擴展集以及如何自動調整應用程式
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc
ms.date: 03/27/2018
ms.author: negat
ms.openlocfilehash: 8ded9b20bd70d18b8a68df0c9775f3a56f8b185b
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30247370"
---
# <a name="what-are-virtual-machine-scale-sets"></a>什麼是虛擬機器擴展集？
Azure 虛擬機器擴展集可讓您建立和管理一組負載平衡的相同 VM。 VM 執行個體的數目可以自動增加或減少，以因應需求或已定義的排程。 擴展集可為您的應用程式提供高可用性，並且可讓您集中管理、設定和更新大量的 VM。 透過虛擬機器擴展集，您將可針對計算、巨量資料和容器工作負載等領域建立大規模的服務。


## <a name="why-use-virtual-machine-scale-sets"></a>為何要使用虛擬機器擴展集？
為了提供備援能力及改善效能，我們常會將應用程式分散於多個執行個體間。 客戶可透過將要求分配給其中一個應用程式執行個體的負載平衡器，來存取您的應用程式。 如果您需要執行維護或更新應用程式執行個體，您的客戶就必須被分配到另一個可用的應用程式執行個體。 為了因應更多客戶需求，您可能需要增加執行應用程式的應用程式執行個體數目。

Azure 虛擬機器擴展集可提供對跨 VM 執行的應用程式進行管理的能力，以及[自動調整資源](virtual-machine-scale-sets-autoscale-overview.md)和流量負載平衡等功能。 擴展集具有以下主要優勢：

- **易於建立和管理多個 VM**
    - 如果您有許多 VM 會執行您的應用程式，則務必要在您的環境間維護一致的組態。 您的應用程式若要有穩定的效能，所有 VM 之間的 VM 大小、磁碟組態和應用程式安裝都必須要一致。
    - 透過擴展集，所有 VM 執行個體都會從相同的基礎 OS 映像和組態建立。 此方法可讓您輕鬆地管理數百個 VM，而不需要執行額外的組態工作或網路管理。
    - 擴展集支援使用 [Azure 負載平衡器](../load-balancer/load-balancer-overview.md)以進行基本的第 4 層流量分配，並可使用 [Azure 應用程式閘道](../application-gateway/application-gateway-introduction.md)以進行進階的第 7 層流量分配和 SSL 終止。

- **提供高可用性和應用程式恢復能力**
    - 擴展集可用來執行多個應用程式執行個體。 如果其中一個 VM 執行個體發生問題時，客戶將可在盡可能不中斷的情況下，透過其他 VM 執行個體繼續存取您的應用程式。
    - 如需更高的可用性，您可以使用[可用性區域](../availability-zones/az-overview.md)，將擴展集中的 VM 執行個體自動散發到單一資料中心內或多個資料中心之間。

- **可讓您的應用程式隨著資源需求的變更自動進行調整**
    - 在一天或一週之中，客戶對您應用程式的需求可能會變動。 為了符合客戶的需求，擴展集可在應用程式需求提高時自動增加 VM 執行個體的數目，並在需求降低時減少 VM 執行個體的數目。
    - 自動調整也可在需求偏低時盡可能減少沒有必要執行應用程式的 VM 執行個體，並且在需求增加時自動新增額外的 VM 執行個體，讓客戶仍能繼續享有一定水準的效能。 這項功能有助於降低成本，並且在必要時有效建立 Azure 資源。

- **大規模運作**
    - 擴展集最多支援可 1,000 個 VM 執行個體。 如果您建立並上傳自己的自訂 VM 映像，則限制為 300 個 VM 執行個體。
    - 若要讓生產工作負載達到最佳效能，請使用 [Azure 受控磁碟](../virtual-machines/windows/managed-disks-overview.md)和[進階儲存體](../virtual-machines/windows/premium-storage.md)。


## <a name="differences-between-virtual-machines-and-scale-sets"></a>虛擬機器與擴展集的差異
擴展集是從虛擬機器建立的。 透過擴展集將可提供管理和自動化層，以執行及調整您的應用程式。 您可以改以手動方式建立和管理個別的 VM，或整合現有的工具以建立類似的自動化層級。 下表列出擴展集相較於手動管理多個 VM 執行個體所具備的優點。

| 案例                           | 手動 VM 群組                                                                    | 虛擬機器擴展集 |
|------------------------------------|----------------------------------------------------------------------------------------|---------------------------|
| 新增其他 VM 執行個體        | 以手動程序建立、設定及確保合規性                             | 從集中的組態自動建立 |
| 流量平衡和分配 | 以手動程序建立及設定 Azure 負載平衡器或應用程式閘道      | 自動建立及整合 Azure 負載平衡器或應用程式閘道 |
| 高可用性與備援能力   | 手動建立可用性設定組，或在各個可用性區域間分配及追蹤 VM | 自動將 VM 執行個體分配到各個可用性區域或可用性設定組 |
| VM 的調整                     | 手動監視和 Azure 自動化                                                 | 根據主機計量、客體計量、Application Insights 或排程進行自動調整 |

擴展集不會產生額外費用。 您只需支付基礎計算資源的費用，例如 VM 執行個體、負載平衡器或受控磁碟儲存體。 管理和自動化功能 (例如自動調整和備援能力) 不會產生使用 VM 的額外費用。


## <a name="next-steps"></a>後續步驟
若要開始使用，請先在 Azure 入口網站中建立虛擬機器擴展集。

> [!div class="nextstepaction"]
> [在 Azure 入口網站中建立擴展集](quick-create-portal.md)
