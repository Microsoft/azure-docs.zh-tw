---
title: 從 Azure Container Service (ACS) 遷移至 Azure Kubernetes Service (AKS)
description: 從 Azure Container Service (ACS) 遷移至 Azure Kubernetes Service (AKS)
services: container-service
author: noelbundick
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 06/13/2018
ms.author: nobun
ms.custom: mvc
ms.openlocfilehash: e42b0e7bd1bce40b7c58d75cb07f5a3f8afa5836
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385036"
---
# <a name="migrating-from-azure-container-service-acs-to-azure-kubernetes-service-aks"></a>從 Azure Container Service (ACS) 遷移至 Azure Kubernetes Service (AKS)

本文旨在協助您進行移轉的規劃與執行，以成功地將搭配 Kubernetes 的 Azure Container Service (ACS) 遷移至 Azure Kubernetes Service (AKS)。 本指南會詳述 ACS 與 AKS 之間的差異、提供移轉程序的概觀，以及協助您做出關鍵決定。

## <a name="differences-between-acs-and-aks"></a>ACS 與 AKS 之間的差異

ACS 和 AKS 在某些重要區域上會有差異，而這會對移轉造成影響。 進行移轉之前，您應該檢閱並規劃解決下列差異的方法。

* AKS 節點使用[受控磁碟](../virtual-machines/windows/managed-disks-overview.md)
    * 非受控磁碟需要先進行轉換，才能將其連結至 AKS 節點
    * 必須將 Azure 磁碟的自訂 `StorageClass` 物件從 `unmanaged` 變更為 `managed`
    * 任何 `PersistentVolumes` 都必須使用 `kind: Managed`
* AKS 目前僅支援一個代理程式集區
* 以 Windows Server 為基礎的節點目前為[個人預覽版](https://azure.microsoft.com/blog/kubernetes-on-azure/)
* 查閱 AKS 的[支援區域](https://docs.microsoft.com/azure/aks/container-service-quotas)清單
* AKS 是受控服務，其中包含託管的 Kubernetes 控制平面。 如果您之前曾修改 ACS 主機的組態，您可能需要修改應用程式

### <a name="differences-between-kubernetes-versions"></a>Kubernetes 版本之間的差異

如果您要遷移至較新版的 Kubernetes (例如：從 1.7.x 到 1.9.x)，您需要注意幾個 k8s API 的變更。

* [將 ThirdPartyResource 遷移至 CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/migrate-third-party-resource/)
* [1.8 版和 1.9 版中的工作負載 API 變更](https://kubernetes.io/docs/reference/workloads-18-19/)。

## <a name="migration-considerations"></a>移轉考量

### <a name="agent-pools"></a>代理程式集區

雖然 AKS 負責管理 Kubernetes 控制平面，但您仍然要定義新叢集中要包含的節點大小和數目。 假設您想要以 1:1 的方式將 AKS 對應至 ACS，您需要擷取現有的 ACS 節點資訊。 您會在建立新 AKS 叢集時用到此資料。

範例：

| 名稱 | Count | VM 大小 | 作業系統 |
| --- | --- | --- | --- |
| agentpool0 | 3 | Standard_D8_v2 | Linux |
| agentpool1 | 1 | Standard_D2_v2 | Windows |

由於在移轉期間會有額外的虛擬機器部署到訂用帳戶，因此您應確認您的配額與限制足以讓這些資源使用。 您可以檢閱 [Azure 訂用帳戶和服務限制](https://docs.microsoft.com/azure/azure-subscription-service-limits)來深入了解。 若要檢查您目前的配額，請前往 Azure 入口網站中的 [[訂用帳戶] 刀鋒視窗](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)並選取您的訂用帳戶，然後選取 [`Usage + quotas`]。

### <a name="networking"></a>網路功能

對於複雜的應用程式，您的遷移作業通常會進行一段時間，而非全部一次完成。 這表示，舊的和新的環境可能需要透過網路進行通訊。 先前能夠使用 `ClusterIP` 服務進行通訊的應用程式可能需要公開為 `LoadBalancer` 類型並適當地加以保護。

若要完成移轉，您需要將用戶端指向 AKS 上執行的新服務。 若要將流量重新導向，建議的方式是將 DNS 更新為指向位於您 AKS 叢集前方的 Load Balancer。

### <a name="stateless-applications"></a>無狀態應用程式

無狀態應用程式移轉是最直接的案例。 您會將 YAML 定義套用到新叢集，並驗證一切如預期般運作，然後將流量重新導向，讓新叢集開始運作。

### <a name="stateful-applications"></a>具狀態應用程式

遷移具狀態應用程式需要仔細規劃，才能避免資料遺失或發生未預期的停機。

#### <a name="highly-available-applications"></a>高度可用的應用程式

有些具狀態應用程式可以在高可用性組態中部署，並且可以在複本之間複製資料。 如果此描述是您目前的部署，則您可能可以在新的 AKS 叢集上建立新成員，並在影響程度最小的情況下遷移至下游呼叫端。 此案例的移轉步驟通常是：

1. 在 AKS 上建立新的次要複本
2. 等候資料完成複寫
3. 進行容錯移轉，讓要次複本成為新的主要複本
4. 將流量指向 AKS 叢集

#### <a name="migrating-persistent-volumes"></a>遷移永續性磁碟區

果您要將現有永續性磁碟區遷移至 AKS，有幾個因素需要考慮。 一般而言，相關步驟如下：

1. (選擇性) 停止寫入應用程式 (需要停機時間)
2. 快照磁碟
3. 從快照集建立新的受控磁碟
4. 在 AKS 中建立永續性磁碟區
5. 將 Pod 規格更新為[使用現有的磁碟區](https://docs.microsoft.com/azure/aks/azure-disk-volume)，而不是 PersistentVolumeClaims (靜態佈建)
6. 將應用程式部署至 AKS
7. 驗證
8. 將流量指向 AKS 叢集

> **重要**：如果您選擇不要停止寫入，您必須將資料複寫到新的部署中，因為寫入的資料會因為磁碟快照而遺失

存在的開放原始碼工具可協助您建立受控磁碟，以及在 Kubernetes 叢集之間遷移磁碟區。

* [noelbundick/azure-cli-disk-extension](https://github.com/noelbundick/azure-cli-disk-copy-extension) - 跨資源群組和 Azure 區域複製及轉換磁碟
* [yaron2/azure-kube-cli](https://github.com/yaron2/azure-kube-cli) - 列舉 ACS Kubernetes 磁碟區，並將其遷移至 AKS 叢集

#### <a name="azure-files"></a>Azure 檔案

不同於磁碟，Azure 檔案服務可同時掛接到多部主機。 Azure 和 Kubernetes 都不會阻止您在 AKS 叢集中建立仍在 ACS 叢集中使用的 Pod。 若要避免資料遺失和非預期的行為，您應該確定這兩個叢集不會同時寫入相同的檔案。

如果您的應用程式可以託管多個指向相同檔案共用的複本，您可以遵循無狀態移轉步驟，並將您的 YAML 定義部署到新叢集。

如果不是，則可能的移轉方法包含下列步驟：

1. 在複本計數為 0 的情況下，將應用程式部署至 AKS
2. 將 ACS 上的應用程式調整為 0 個 (需要停機時間)
3. 將 AKS 上的應用程式調整為最多 1 個
4. 驗證
5. 將流量指向 AKS 叢集

在您想要以空白的共用開始，然後再複製來源資料，您可以使用 [`az storage file copy`](https://docs.microsoft.com/cli/azure/storage/file/copy?view=azure-cli-latest) 命令來遷移您的資料。

### <a name="deployment-strategy"></a>部署策略

建議您使用現有 CI/CD 管線將已知的良好組態部署至 AKS。 您會複製現有的部署工作，並確定 `kubeconfig` 指向新的 AKS 叢集。

如果在無法這麼做的情況下，您必須從 ACS 匯出資源定義，然後將其套用至 AKS。 您可以使用 `kubectl` 匯出物件。

```console
kubectl get deployment -o=yaml --export > deployments.yaml
```

視您的需求而定，另外還有數個開放原始碼工具可提供協助：

* [heptio/ark](https://github.com/heptio/ark) - 需要 k8s 1.7
* [yaron2/azure-kube-cli](https://github.com/yaron2/azure-kube-cli)
* [mhausenblas/reshifter](https://github.com/mhausenblas/reshifter)

## <a name="migration-steps"></a>移轉步驟

### <a name="1-create-an-aks-cluster"></a>1.建立 AKS 叢集

您可以遵循文件內容，透過 Azure 入口網站、Azure CLI 或 Resource Manager 範本來[建立 AKS 叢集](https://docs.microsoft.com/azure/aks/create-cluster)。

> 您可以在 GitHub 上的 [Azure/AKS](https://github.com/Azure/AKS/tree/master/examples/vnet) 存放庫中，找到 AKS 的 Azure Resource Manager 範本範例

### <a name="2-modify-applications"></a>2.修改應用程式

對 YAML 定義進行任何必要的修改。 例如：將 `apps/v1beta1` 取代為 `Deployments` 的 `apps/v1`

### <a name="3-optional-migrate-volumes"></a>3.(選擇性) 遷移磁碟區

將磁碟區從 ACS 叢集遷移到 AKS 叢集。 您可以在[遷移永續性磁碟區](#Migrating-Persistent-Volumes)一節中找到更多詳細資料。

### <a name="4-deploy-applications"></a>4.部署應用程式

使用 CI/CD 系統將應用程式部署到 AKS，或使用 kubectl 來套用 YAML 定義。

### <a name="5-validate"></a>5.驗證

驗證您的應用程式是否正如預期般運作，而且所有遷移的資料都已完成複製。

### <a name="6-redirect-traffic"></a>6.重新導向流量

將 DNS 更新為將用戶端指向您的 AKS 部署。

### <a name="7-post-migration-tasks"></a>7.移轉後工作

如果您已遷移磁碟區，並選擇不停止寫入，則您必須將資料複製到新叢集。
