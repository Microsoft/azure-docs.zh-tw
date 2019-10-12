---
title: 從 Azure Container Service （ACS）遷移至 Azure Kubernetes Service （AKS）
description: 從 Azure Container Service （ACS）遷移至 Azure Kubernetes Service （AKS）。
services: container-service
author: noelbundick
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 06/13/2018
ms.author: nobun
ms.custom: mvc
ms.openlocfilehash: 66f76a8a706f60df786786cbd1ce00b7eafd8d7e
ms.sourcegitcommit: cd70273f0845cd39b435bd5978ca0df4ac4d7b2c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2019
ms.locfileid: "71097891"
---
# <a name="migrate-from-azure-container-service-acs-to-azure-kubernetes-service-aks"></a>從 Azure Container Service （ACS）遷移至 Azure Kubernetes Service （AKS）

本文可協助您規劃和執行 Azure Container Service （ACS）與 Kubernetes 和 Azure Kubernetes Service （AKS）之間的成功遷移。 為了協助您做出重要的決策，本指南將詳細說明 ACS 與 AKS 之間的差異，並提供遷移程式的總覽。

## <a name="differences-between-acs-and-aks"></a>ACS 與 AKS 之間的差異

在某些影響遷移的主要領域中，ACS 和 AKS 會有所不同。 在進行任何遷移之前，您應該先複習並規劃解決下列差異：

* AKS 節點會使用[受控磁碟](../virtual-machines/windows/managed-disks-overview.md)。
    * 您必須先轉換非受控磁碟，才能將它們附加至 AKS 節點。
    * Azure `StorageClass`磁碟的自訂物件必須從`unmanaged`變更為`managed`。
    * 任何`PersistentVolumes`應使用`kind: Managed`。
* AKS 支援[多個節點](https://docs.microsoft.com/azure/aks/use-multiple-node-pools)集區（目前處於預覽狀態）。
* 以 Windows Server 為基礎的節點目前在 AKS 中處於[預覽](https://azure.microsoft.com/blog/kubernetes-on-azure/)狀態。
* AKS 支援一組有限的[區域](https://docs.microsoft.com/azure/aks/quotas-skus-regions)。
* AKS 是受控服務，其中包含託管的 Kubernetes 控制平面。 如果您先前已修改 ACS 主機的設定，您可能需要修改應用程式。

## <a name="differences-between-kubernetes-versions"></a>Kubernetes 版本之間的差異

如果您要遷移至較新版本的 Kubernetes，請參閱下列資源以瞭解 Kubernetes 版本控制策略：

* [Kubernetes 版本和版本誤差支援原則](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-versions)

## <a name="migration-considerations"></a>移轉考量

### <a name="agent-pools"></a>代理程式集區

雖然 AKS 會管理 Kubernetes 控制平面，但您仍會定義要包含在新叢集中的節點大小和數目。 假設您想要以 1:1 的方式將 AKS 對應至 ACS，您需要擷取現有的 ACS 節點資訊。 當您建立新的 AKS 叢集時，請使用此資料。

範例：

| Name | Count | VM 大小 | 作業系統 |
| --- | --- | --- | --- |
| agentpool0 | 3 | Standard_D8_v2 | Linux |
| agentpool1 | 1 | Standard_D2_v2 | Windows |

由於在移轉期間會有額外的虛擬機器部署到訂用帳戶，因此您應確認您的配額與限制足以讓這些資源使用。 

如需詳細資訊，請參閱[Azure 訂用帳戶和服務限制](https://docs.microsoft.com/azure/azure-subscription-service-limits)。 若要檢查您目前的配額，請在 Azure 入口網站[中，移](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)至 [訂用帳戶] 分頁，選取您的訂閱，然後選取 [**使用量 + 配額**]。

### <a name="networking"></a>網路功能

對於複雜的應用程式，您的遷移作業通常會進行一段時間，而非全部一次完成。 這表示新舊環境可能需要透過網路進行通訊。 先前使用`ClusterIP`服務進行通訊的應用程式可能需要公開為類型`LoadBalancer` ，並適當地加以保護。

若要完成遷移，您需要將用戶端指向在 AKS 上執行的新服務。 我們建議您將 DNS 更新為指向位於 AKS 叢集前方的 Load Balancer，以重新導向流量。

### <a name="stateless-applications"></a>無狀態應用程式

無狀態應用程式移轉是最直接的案例。 您會將 YAML 定義套用至新的叢集，並確定所有專案都如預期般運作，然後重新導向流量以啟用新的叢集。

### <a name="stateful-applications"></a>具狀態應用程式

請仔細規劃您的具狀態應用程式遷移，以避免資料遺失或非預期的停機時間。

#### <a name="highly-available-applications"></a>高可用性應用程式

您可以在高可用性設定中部署一些具狀態的應用程式。 這些應用程式可以跨複本複製資料。 如果您目前使用這種部署類型，您可以在新的 AKS 叢集上建立新的成員，然後在下游呼叫端上以最小的效果進行遷移。 一般而言，此案例的遷移步驟如下：

1. 在 AKS 上建立新的次要複本。
2. 等待資料複寫。
3. 故障切換，讓次要複本成為新的主要複本。
4. 將流量指向 AKS 叢集。

#### <a name="migrating-persistent-volumes"></a>遷移持續性磁碟區

如果您要將現有的持續性磁碟區遷移至 AKS，通常會遵循下列步驟：

1. 停止寫入應用程式。 （這是選擇性步驟，需要停機）。
2. 製作磁碟的快照集。
3. 從快照集建立新的受控磁碟。
4. 在 AKS 中建立持續性磁碟區。
5. 將 pod 規格更新為[使用現有的磁碟](https://docs.microsoft.com/azure/aks/azure-disk-volume)區，而不是 PersistentVolumeClaims （靜態布建）。
6. 將應用程式部署至 AKS。
7. 檢查.
8. 將流量指向 AKS 叢集。

> [!IMPORTANT]
> 如果您選擇不停止寫入，則必須將資料複寫到新的部署。 否則，您將會錯過在磁碟快照集之後所寫入的資料。

某些開放原始碼工具可協助您建立受控磁碟，並在 Kubernetes 叢集之間遷移磁碟區：

* [Azure CLI 磁碟複製延伸](https://github.com/noelbundick/azure-cli-disk-copy-extension)模組會在資源群組和 Azure 區域之間複製和轉換磁碟。
* [Azure KUBE CLI 擴充](https://github.com/yaron2/azure-kube-cli)功能會列舉 ACS Kubernetes 磁碟區，並將其遷移至 AKS 叢集。

#### <a name="azure-files"></a>Azure 檔案儲存體

不同於磁碟，Azure 檔案服務可同時掛接到多部主機。 在您的 AKS 叢集中，Azure 和 Kubernetes 不會防止您建立 ACS 叢集仍然使用的 pod。 若要防止資料遺失和非預期的行為，請確定叢集不會同時寫入相同的檔案。

如果您的應用程式可以裝載多個指向相同檔案共用的複本，請遵循無狀態遷移步驟，並將您的 YAML 定義部署至新叢集。 如果不是，則可能的移轉方法包含下列步驟：

1. 將您的應用程式部署至複本計數為0的 AKS。
2. 將 ACS 上的應用程式調整為0。 （此步驟需要停機時間。）
3. 將 AKS 上的應用程式調整為1。
4. 檢查.
5. 將流量指向 AKS 叢集。

如果您想要從空的共用開始，並複製來源資料，您可以使用[`az storage file copy`](https://docs.microsoft.com/cli/azure/storage/file/copy?view=azure-cli-latest)命令來遷移資料。

### <a name="deployment-strategy"></a>部署策略

建議您使用現有的 CI/CD 管線，將已知良好的設定部署至 AKS。 複製現有的部署工作，並確定`kubeconfig`指向新的 AKS 叢集。

如果無法這麼做，請從 ACS 匯出資源定義，然後將它們套用至 AKS。 您可以使用 `kubectl` 匯出物件。

```console
kubectl get deployment -o=yaml --export > deployments.yaml
```

根據您的部署需求，有數個開放原始碼工具可以提供協助：

* [Velero](https://github.com/heptio/ark)（此工具需要 Kubernetes 1.7）。
* [Azure Kube CLI 擴充功能](https://github.com/yaron2/azure-kube-cli)
* [ReShifter](https://github.com/mhausenblas/reshifter)

## <a name="migration-steps"></a>移轉步驟

1. 透過 [Azure 入口網站]、[Azure CLI] 或 [Azure Resource Manager] 範本來[建立 AKS](https://docs.microsoft.com/azure/aks/create-cluster)叢集。

   > [!NOTE]
   > 在 GitHub 上的[Azure/AKS](https://github.com/Azure/AKS/tree/master/examples/vnet)存放庫中尋找適用于 AKS 的範例 Azure Resource Manager 範本。

2. 對您的 YAML 定義進行任何必要的變更。 例如，將取代`apps/v1beta1` `apps/v1`為`Deployments`。

3. [遷移磁碟](#migrating-persistent-volumes)區（選擇性）從您的 ACS 叢集到 AKS 叢集。

4. 使用您的 CI/CD 系統將應用程式部署至 AKS。 或使用 kubectl 來套用 YAML 定義。

5. 檢查. 請確定您的應用程式如預期般運作，且已複製任何已遷移的資料。

6. 重新導向流量。 將 DNS 更新為將用戶端指向您的 AKS 部署。

7. 完成遷移後工作。 如果您已遷移磁碟區，並選擇不停止寫入，請將該資料複製到新的叢集。
