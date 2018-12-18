---
title: Azure Kubernetes Service (AKS) 的常見問題集
description: 提供一些關於 Azure Kubernetes Service (AKS) 常見問題的解答。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 08/17/2018
ms.author: iainfou
ms.openlocfilehash: 1e101e308ec350e9900c1347da730ca02b16c7bb
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377459"
---
# <a name="frequently-asked-questions-about-azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS) 的常見問題集

本文將說明關於 Azure Kubernetes Service (AKS) 的常見問題。

## <a name="which-azure-regions-provide-the-azure-kubernetes-service-aks-today"></a>目前，哪些 Azure 區域提供 Azure Kubernetes Service (AKS)？

如需可用區域的完整清單，請參閱 [AKS 區域和可用性][aks-regions]。

## <a name="does-aks-support-node-autoscaling"></a>AKS 是否支援節點自動調整？

是，從 Kubernetes 1.10 開始，可透過 [Kubernetes Autoscaler][auto-scaler] 使用自動調整。 如需如何設定及使用叢集自動調整程式的詳細資訊，請參閱 [AKS 上的叢集自動調整][aks-cluster-autoscale]。

## <a name="does-aks-support-kubernetes-role-based-access-control-rbac"></a>AKS 是否支援 Kubernetes 角色型存取控制 (RBAC)？

是，使用 Azure CLI 建立叢集時，預設會啟用 Kubernetes RBAC。 使用 Azure 入口網站或範本所建立的叢集，可以啟用 RBAC。

## <a name="can-i-deploy-aks-into-my-existing-virtual-network"></a>可以將 AKS 部署到我現有的虛擬網路嗎？

是，您可以使用[進階網路功能][aks-advanced-networking] \(英文\)，將 AKS 叢集部署到現有的虛擬網路。

## <a name="can-i-restrict-the-kubernetes-api-server-to-only-be-accessible-within-my-virtual-network"></a>我可以將 Kubernetes API 伺服器限制為只能從我的虛擬網路存取嗎？

目前沒有。 Kubernetes API 伺服器會以公用完整網域名稱 (FQDN) 公開。 您可以使用 [Kubernetes 角色型存取控制 (RBAC) 和 Azure Active Directory (AAD)][aks-rbac-aad] 來控制叢集的存取權

## <a name="are-security-updates-applied-to-aks-agent-nodes"></a>安全性更新是否會套用至 AKS 代理程式節點？

是，Azure 會透過夜間排程將安全性修補程式自動套用至叢集中的節點。 不過，您有責任確保節點已視需要重新啟動。 您有多個執行節點重新啟動的選項：

- 手動、透過 Azure 入口網站，或透過 Azure CLI。
- 藉由升級 AKS 叢集。 叢集會自動升級 [cordon 和 drain 節點][cordon-drain]，然後為每個節點的備份帶來最新的 Ubuntu 映像和新的修補程式版本或 Kubernetes 次要版本。 如需詳細資訊，請參閱[升級 AKS 叢集][aks-upgrade]。
- 使用 [Kured](https://github.com/weaveworks/kured)，這是一款針對 Kubernetes 所推出的的開放原始碼重新啟動精靈。 Kured 會以 [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 執行，並監視每個節點，查看是否有檔案指示需重新啟動。 系統會使用相同的 [cordon 和 drain 程序][cordon-drain] 作為叢集升級，跨叢集管理作業系統重新開機。

## <a name="why-are-two-resource-groups-created-with-aks"></a>為何會使用 AKS 建立兩個資源群組？

每個 AKS 部署皆跨越兩個資源群組：

- 第一個資源群組是您所建立的資源群組，其中僅包含 Kubernetes 服務資源。 AKS 資源提供者會在部署期間自動建立第二個資源群組，例如 MC_myResourceGRoup_myAKSCluster_eastus。
- 第二個資源群組 (例如，MC_myResourceGroup_myAKSCluster_eastus) 包含所有與該叢集相關聯的基礎結構資源。 這些資源包括 Kubernetes 節點 VM、虛擬網路和儲存體。 另外建立這個資源群組是為了簡化資源清除作業。

如果您建立要與 AKS 叢集搭配使用的資源 (例如，儲存體帳戶或保留的公用 IP 位址)，請將這些資源置於自動產生的資源群組中。

## <a name="can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-mc-resource-group"></a>是否可以修改 MC_* 資源群組中 AKS 資源的標記和其他屬性？

修改和刪除 MC_* 資源群組中資源的 Azure 所建立標記和其他屬性可能會導致非預期的結果，例如調整和升級錯誤。 系統支援建立和修改其他自訂標記，例如，以便指派業務單位或成本中心。 在 AKS 叢集中修改 MC_* 下的資源會中斷 SLO。

## <a name="what-kubernetes-admission-controllers-does-aks-support-can-admission-controllers-be-added-or-removed"></a>AKS 支援哪些 Kubernetes 許可控制器？ 是否可以新增或移除許可控制器？

AKS 支援下列[許可控制器][admission-controllers]：

- *NamespaceLifecycle*
- *LimitRanger*
- *ServiceAccount*
- *DefaultStorageClass*
- *DefaultTolerationSeconds*
- *MutatingAdmissionWebhook*
- *ValidatingAdmissionWebhook*
- *ResourceQuota*
- *DenyEscalatingExec*
- *AlwaysPullImages*

目前無法修改 AKS 中的許可控制器清單。

## <a name="is-azure-key-vault-integrated-with-aks"></a>Azure Key Vault 是否會與 AKS 整合？

AKS 目前並未原生整合到 Azure Key Vault。 不過，[Azure Key Vault FlexVolume for Kubernetes 專案][keyvault-flexvolume] \(英文\) 能讓 Kubernetes Pod 直接與 KeyVault 祕密進行整合。

## <a name="can-i-run-windows-server-containers-on-aks"></a>我是否可以在 AKS 上執行 Windows Server 容器？

若要執行 Windows Server 容器，您需要執行以 Windows Server 為基礎的節點。 目前在 AKS 中並未提供以 Windows Server 為基礎的節點。 不過，您可以使用 Virtual Kubelet 來排程 Azure 容器執行個體上的 Windows 容器，以及在 AKS 叢集當中進行管理。 如需詳細資訊，請參閱[使用 Virtual Kubelet 搭配 AKS][virtual-kubelet]。

## <a name="does-aks-offer-a-service-level-agreement"></a>AKS 是否提供服務等級協定？

在服務等級協定 (SLA) 中，提供者同意就未達到所發佈服務等級的情況，對客戶補償該項服務的費用。 由於 AKS 本身即免費，因此我們並不提供補償費用，亦無任何正式的 SLA。 不過，AKS 會嘗試將 Kubernetes API 伺服器的可用性維持在至少 99.5%。

<!-- LINKS - internal -->

[aks-regions]: ./container-service-quotas.md#region-availability
[aks-upgrade]: ./upgrade-cluster.md
[aks-cluster-autoscale]: ./autoscaler.md
[virtual-kubelet]: virtual-kubelet.md
[aks-advanced-networking]: ./configure-advanced-networking.md
[aks-rbac-aad]: ./aad-integration.md

<!-- LINKS - external -->

[auto-scaler]: https://github.com/kubernetes/autoscaler
[cordon-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[hexadite]: https://github.com/Hexadite/acs-keyvault-agent
[admission-controllers]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[keyvault-flexvolume]: https://github.com/Azure/kubernetes-keyvault-flexvol
