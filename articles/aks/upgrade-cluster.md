---
title: 升級 Azure Kubernetes Service (AKS) 叢集
description: 瞭解如何升級 Azure Kubernetes Service (AKS) 叢集，以取得最新的功能和安全性更新。
services: container-service
ms.topic: article
ms.date: 12/17/2020
ms.openlocfilehash: 1d3c275758a1e241a531b65d1897903153efab94
ms.sourcegitcommit: ca215fa220b924f19f56513fc810c8c728dff420
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2021
ms.locfileid: "98567452"
---
# <a name="upgrade-an-azure-kubernetes-service-aks-cluster"></a>升級 Azure Kubernetes Service (AKS) 叢集

AKS 叢集生命週期的一部分牽涉到執行定期升級至最新的 Kubernetes 版本。 請務必套用最新的安全性版本，或升級以取得最新的功能。 本文說明如何升級 AKS 叢集中的主要元件或單一的預設節點集區。

針對使用多個節點集區或 Windows Server 節點的 AKS 叢集，請參閱 [在 AKS 中升級節點集][nodepool-upgrade]區。

## <a name="before-you-begin"></a>開始之前

本文會要求您執行 Azure CLI 2.0.65 版版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

> [!WARNING]
> AKS 叢集升級會觸發隔離並清空您的節點。 如果您有可用的低可用計算配額，升級可能會失敗。 如需詳細資訊，請參閱 [增加配額](../azure-portal/supportability/resource-manager-core-quotas-request.md)

## <a name="check-for-available-aks-cluster-upgrades"></a>檢查可用的 AKS 叢集升級

若要檢查哪些 Kubernetes 版本可用於您的叢集，請使用 [az aks get-upgrades][az-aks-get-upgrades] 命令。 下列範例會檢查 *myResourceGroup* 中 *myAKSCluster* 的可用升級：

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

> [!NOTE]
> 當您升級支援的 AKS 叢集時，無法略過 Kubernetes 次要版本。 所有升級都必須依主要版本號碼循序執行。 例如，允許在 *1.14. x*  ->  *1.15.* x 或 *1.15.* x  ->  *1.16* 之間進行升級，但不允許 *1.14. x*  ->  *1.16. x* 。 
> > 略過多個版本只能在從 _不支援的版本_ 升級回支援的 _版本_ 時進行。 例如，從不支援的 *1.10. x* 升級 > 支援的 *1.15。 x* 可以完成。

下列範例輸出顯示叢集可以升級為 *1.19.1* 和 *1.19.3* 版：

```console
Name     ResourceGroup    MasterVersion    Upgrades
-------  ---------------  ---------------  --------------
default  myResourceGroup  1.18.10          1.19.1, 1.19.3
```

如果沒有可用的升級，您將會收到下列訊息：

```console
ERROR: Table output unavailable. Use the --query option to specify an appropriate query. Use --debug for more info.
```

## <a name="customize-node-surge-upgrade"></a>自訂節點激增升級

> [!Important]
> 節點浪湧需要每個升級作業要求的最大浪湧計數的訂用帳戶配額。 例如，具有5個節點集區（每個節點的計數為4個節點）的叢集總共有20個節點。 如果每個節點集區的最大激增值為50%，則需要有10個節點的額外計算和 IP 配額 (2 個節點 * 5 集區) 是完成升級的必要條件。
>
> 如果使用 Azure CNI，請驗證子網中是否有可用的 ip，以 [滿足 AZURE CNI 的 IP 需求](configure-azure-cni.md)。

根據預設，AKS 會使用一個額外的節點，將升級設定為激增。 最大浪湧設定的預設值，可讓 AKS 在隔離/清空現有的應用程式以取代舊版本的節點之前，建立額外的節點，以將工作負載中斷降至最低。 您可以針對每個節點集區自訂最大激增值，以便在升級速度與升級中斷之間進行取捨。 藉由增加最大激增值，升級程式會更快完成，但為最大激增設定較大的值可能會在升級過程中導致中斷。 

例如，最大的最大激增值100% 可提供最快速的升級程式 (將節點計數加倍) 但也會導致節點集區中的所有節點同時清空。 您可能會想要使用較高的值，例如用於測試環境的值。 針對生產節點集區，建議使用33% 的 max_surge 設定。

AKS 可接受整數值和最大激增的百分比值。 整數（例如 "5"）指出五個要激增的額外節點。 "50%" 的值表示集區中目前節點計數一半的激增值。 最大激增百分比值最少可以是1%，最大值為100%。 百分比值會無條件進位到最接近的節點計數。 如果在升級時，最大的最大激增值低於目前的節點計數，則會使用目前的節點計數來取得最大的最大激增值。

在升級期間，最大激增值最少可以是1，而最大值等於節點集區中的節點數目。 您可以設定較大的值，但是最大激增的節點數目上限不會高於升級時集區中的節點數目。

> [!Important]
> 節點集區上的最大浪湧設定是永久性的。  後續 Kubernetes 升級或節點版本升級將會使用此設定。 您可以隨時變更節點集區的最大激增值。 針對生產節點集區，我們建議使用最大激增設定33%。

使用下列命令來設定新的或現有節點集區的最大激增值。

```azurecli-interactive
# Set max surge for a new node pool
az aks nodepool add -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 33%
```

```azurecli-interactive
# Update max surge for an existing node pool 
az aks nodepool update -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 5
```

## <a name="upgrade-an-aks-cluster"></a>升級 AKS 叢集

透過適用於您的 AKS 叢集的可用版本清單，使用 [az aks upgrade][az-aks-upgrade] 命令進行升級。 在升級過程中，AKS 將會： 
- 將新的緩衝區節點 (或任意數量 [的) 節點](#customize-node-surge-upgrade) ，新增至執行指定 Kubernetes 版本的叢集。 
- [隔離並清空][kubernetes-drain] 其中一個舊節點，將執行應用程式的中斷情況降到最低 (如果您使用的是最大值，則會 [隔離和清空][kubernetes-drain] 多個節點，同時與指定的緩衝區節點數目) 。 
- 當舊節點完全清空時，系統會將它重新安裝映射以接收新的版本，而且它會變成要升級之下列節點的緩衝區節點。 
- 此程式會重複，直到叢集中的所有節點都已升級為止。 
- 在程式結束時，將會刪除最後一個緩衝區節點，以維護現有的代理程式節點計數和區域平衡。

```azurecli-interactive
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION
```

叢集升級需要幾分鐘的時間，具體取決於您擁有多少節點。

> [!IMPORTANT]
> 請確認任何 `PodDisruptionBudgets` (pdb) 至少允許一次移動1個 pod 複本，否則清空/收回作業將會失敗。
> 如果清空作業失敗，則依設計，升級作業會失敗，以確保應用程式不會中斷。 請更正導致作業停止 (錯誤的 Pdb、缺少配額等) ，然後重新嘗試操作的原因。

若要確認升級是否成功，請使用 [az aks show][az-aks-show] 命令：

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

下列範例輸出顯示叢集現在會執行 *1.18.10*：

```json
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.18.10              Succeeded            myakscluster-dns-379cbbb9.hcp.eastus.azmk8s.io
```

## <a name="set-auto-upgrade-channel"></a>設定自動升級通道

除了手動升級叢集，您也可以在叢集上設定自動升級通道。 可用的升級通道如下：

|管道| 動作 | 範例
|---|---|---|
| `none`| 停用自動升級，並將叢集保留在其目前版本的 Kubernetes| 如果保持不變，則為預設設定|
| `patch`| 自動將叢集升級為最新支援的修補程式版本（當它可供使用，同時維持次要版本相同）。| 例如，如果叢集正在執行版本 *1.17.7* ，且有 *1.17.9*、 *1.18.4*、 *1.18.6* 和 *1.19.1* 版本可供使用，則您的叢集會升級為 *1.17.9*|
| `stable`| 自動將叢集升級至次要版本 *n-1* 上最新支援的修補程式版本，其中 *n* 是最新支援的次要版本。| 例如，如果叢集正在執行版本 *1.17.7* ，且有 *1.17.9*、 *1.18.4*、 *1.18.6* 和 *1.19.1* 版可供使用，則您的叢集會升級為 *1.18.6*。
| `rapid`| 自動將叢集升級至最新支援的次要版本上最新支援的修補程式版本。| 如果叢集位於 *n 2* 次要版本的 Kubernetes 版本，其中 *n* 是最新支援的次要版本，則叢集會先升級至 *n-1* 次要版本上最新支援的修補程式版本。 例如，如果叢集正在執行版本 *1.17.7* ，且有 *1.17.9*、 *1.18.4*、 *1.18.6* 和 *1.19.1* 版可供使用，則您的叢集首先會升級至 *1.18.6*，然後升級為 *1.19.1*。

> [!NOTE]
> 叢集自動升級只會更新 GA 版本的 Kubernetes，而且不會更新為預覽版本。

自動升級叢集的程式，會遵循與手動升級叢集相同的程式。 如需詳細資訊，請參閱 [升級 AKS][upgrade-cluster]叢集。

AKS 叢集的叢集自動升級是預覽功能。

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

`AutoUpgradePreview`使用[az feature register][az-feature-register]命令註冊功能旗標，如下列範例所示：

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n AutoUpgradePreview
```

狀態需要幾分鐘的時間才會顯示「已註冊」。 使用 [az feature list][az-feature-list] 命令來確認註冊狀態：

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AutoUpgradePreview')].{Name:name,State:properties.state}"
```

當您準備好時，請使用 [az provider register][az-provider-register]命令重新整理 *>microsoft.containerservice* 資源提供者的註冊：

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

若要在建立叢集時設定自動升級通道，請使用 *自動升級通道* 參數，類似下列範例。

```azurecli-interactive
az aks create --resource-group myResourceGroup --name myAKSCluster --auto-upgrade-channel stable --generate-ssh-keys
```

若要在現有的叢集上設定自動升級通道，請更新 *自動升級通道* 參數，如下例所示。

```azurecli-interactive
az aks update --resource-group myResourceGroup --name myAKSCluster --auto-upgrade-channel stable
```

## <a name="next-steps"></a>後續步驟

本文說明如何升級現有的 AKS 叢集。 若要深入了解部署和管理 AKS 叢集，請參閱教學課程集合。

> [!div class="nextstepaction"]
> [AKS 教學課程][aks-tutorial-prepare-app]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az-aks-upgrade]: /cli/azure/aks#az-aks-upgrade
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list&preserve-view=true
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register&preserve-view=true
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[upgrade-cluster]:  #upgrade-an-aks-cluster
