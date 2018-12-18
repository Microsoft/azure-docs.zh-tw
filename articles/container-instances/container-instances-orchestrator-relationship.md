---
title: Azure 容器執行個體和容器協調流程
description: 了解 Azure 容器執行個體與容器 Orchestrator 的互動方式。
services: container-instances
author: seanmck
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 03/23/2018
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: e1455cba004facfa03dca21544eec754f5dc60be
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32165556"
---
# <a name="azure-container-instances-and-container-orchestrators"></a>Azure 容器執行個體和容器 Orchestrator

由於規模較小和應用程式方向的緣故，容器很適合用於敏捷式傳遞環境和微服務式架構。 自動執行和管理大量容器以及其互動方式的工作稱為「協調流程」。 熱門容器協調器包括 Kubernetes、DC/OS 和 Docker Swarm。

Azure 容器執行個體提供了一些基本的協調流程平台排程功能。 但未涵蓋這些平台所提供的較高價值服務，因此事實上可與這些平台互補。 本文說明 Azure 容器執行個體所能處理的項目範圍，以及完整的容器 Orchestrator 如何與其互動。

## <a name="traditional-orchestration"></a>傳統協調流程

協調流程的標準定義包含下列工作：

- **排程**：在給定容器映像和資源要求的情況下，尋找適合用來執行容器的電腦。
- **親和性/反親和性**：指定一組容器應該在彼此附近執行 (以獲取效能) 或是彼此離得夠遠來執行 (以獲得可用性)。
- **健康情況監視**：監視容器的失敗，並自動為容器重新排程。
- **容錯移轉**：追蹤每部電腦上執行的項目，並將容器從失敗的電腦重新排程到狀況良好的節點。
- **調整**：以手動或自動方式新增或移除容器執行個體以符合需求。
- **網路**：提供重疊的網路以協調容器，讓它們跨越多部主機電腦彼此通訊。
- **服務探索**：讓容器可以自動找到彼此，即使它們移到不同的主機電腦和變更 IP 位址。
- **協調的應用程式升級**：管理容器的升級以避免應用程式停機，並可在發生錯誤時復原。

## <a name="orchestration-with-azure-container-instances-a-layered-approach"></a>使用 Azure 容器執行個體的協調流程：分層式方法

Azure 容器執行個體可提供分層式協調流程方法，提供執行單一容器所需的所有排程和管理功能，同時讓 Orchestrator 平台管理在其上執行的多個容器工作。

容器執行個體的基礎結構皆由 Azure 負責管理，因此 Orchestrator 平台本身不需要擔心如何尋找用來執行單一容器的適當主機電腦。 雲端的彈性可確保永遠有適當的主機電腦可供使用。 相反地，Orchestrator 可以專注於簡化多容器架構開發的工作，包括調整作業與協調的升級。

## <a name="potential-scenarios"></a>可能的案例

雖然 Orchestrator 與 Azure 容器執行個體的整合技術仍未成熟，但我們預期可能會出現幾個不同的環境：

### <a name="orchestration-of-container-instances-exclusively"></a>獨佔的容器執行個體協調流程

因為這些執行個體啟動速度快且依秒計費，以獨佔方式立基於 Azure 容器執行個體的環境可提供最快的方法來開始使用，並處理高度變化的工作負載。

### <a name="combination-of-container-instances-and-containers-in-virtual-machines"></a>結合容器執行個體與虛擬機器中的容器

針對長時間執行的穩定工作負載，協調專用虛擬機器叢集中的容器所需的成本，一般會比使用 Azure 容器執行個體來執行相同容器還低。 不過，容器執行個體能提供絕佳的解決方案來快速擴展和收縮整體容量，以處理未預期的使用量或短暫出現的高峰使用量。

Orchestrator 可以直接排程使用 Azure 容器執行個體的其他容器，並在不再需要時加以刪除，而不是相應放大叢集中的虛擬機器數目，然後將其他容器部署至這些電腦。

## <a name="sample-implementation-azure-container-instances-connector-for-kubernetes"></a>實作範例：Kubernetes 的 Azure 容器執行個體連接器

為了示範容器協調流程平台如何與 Azure 容器執行個體整合，我們已開始建置 [Kubernetes 的連接器範例][aci-connector-k8s]。

Kubernetes 的連接器會模擬 [kubelet][kubelet-doc]，方法是註冊為具有無限容量的節點，並將 [Pod][pod-doc] 的建立分派為 Azure 容器執行個體中的容器群組。

其他 Orchestrator 的連接器可以同樣方式建置來與平台基本項目整合，以結合 Orchestrator API 的能力與在 Azure 容器執行個體中管理容器的速度和簡便性。

> [!WARNING]
> Kubernetes 的 ACI 連接器是「實驗性的」，不應用在生產環境中。

## <a name="next-steps"></a>後續步驟

使用[快速入門指南](container-instances-quickstart.md)對 Azure Container Instances 建立您的第一個容器。

<!-- IMAGES -->

<!-- LINKS -->
[aci-connector-k8s]: https://github.com/virtual-kubelet/virtual-kubelet/tree/master/providers/azure
[kubelet-doc]: https://kubernetes.io/docs/admin/kubelet/
[pod-doc]: https://kubernetes.io/docs/concepts/workloads/pods/pod/