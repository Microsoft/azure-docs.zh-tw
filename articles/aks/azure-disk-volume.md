---
title: 搭配 AKS 使用 Azure 磁碟
description: 搭配 AKS 使用 Azure 磁碟
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 05/21/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: aa9b92df84a48ef4cb706e9e89e0f6c0a25cd42a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39420487"
---
# <a name="volumes-with-azure-disks"></a>包含 Azure 磁碟的磁碟區

容器型應用程式常常需要存取和保存外部資料磁碟區中的資料。 Azure 磁碟可用來作為這個外部資料存放區。 本文將詳細說明如何在 Azure Kubernetes Service (AKS) 叢集中使用 Azure 磁碟作為 Kubernetes 磁碟區。

如需有關 Kubernetes 磁碟區的詳細資訊，請參閱 [Kubernetes 磁碟區][kubernetes-volumes]。

## <a name="create-an-azure-disk"></a>建立 Azure 磁碟

在掛接 Azure 受控磁碟作為 Kubernetes 磁碟區之前，磁碟必須存在於 AKS **節點**資源群組中。 使用 [az resource show][az-resource-show] 命令取得資源群組名稱。

```azurecli-interactive
$ az resource show --resource-group myResourceGroup --name myAKSCluster --resource-type Microsoft.ContainerService/managedClusters --query properties.nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

使用 [az disk create][az-disk-create] 命令來建立 Azure 磁碟。

將 `--resource-group` 更新成上一個步驟中收集的資源群組名稱，並將 `--name` 更新成您選擇的名稱。

```azurecli-interactive
az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name myAKSDisk  \
  --size-gb 20 \
  --query id --output tsv
```

一旦建立磁碟，您應該會看到類似下列的輸出。 這個值是磁碟識別碼，掛接磁碟時會使用此識別碼。

```console
/subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```
> [!NOTE]
> Azure 受控磁碟是按 SKU 的特定大小計費。 這些 SKU 範圍從 S4 或 P4 磁碟的 32GiB 到 S50 或 P50 磁碟的 4TiB。 此外，進階受控磁碟的輸送量和 IOPS 效能，同時取決於 SKU 和 AKS 叢集中節點的執行個體大小。 請參閱[受控磁碟的定價和效能][managed-disk-pricing-performance]。

> [!NOTE]
> 如果您需要在個別的資源群組中建立磁碟，也需要將叢集的 Azure Kubernetes Service (AKS) 服務主體新增到以 `Contributor` 角色持有磁碟的資源群組。 
>

## <a name="mount-disk-as-volume"></a>將磁碟掛接為磁碟區

藉由在容器規格中設定磁碟區，將 Azure 磁碟掛接至您的 Pod。

建立一個名為 `azure-disk-pod.yaml` 且含有下列內容的新檔案。 使用新建立的磁碟名稱更新 `diskName`，以及使用磁碟識別碼更新 `diskURI`。 此外，請記下 `mountPath`，這是 Pod 中 Azure 磁碟掛接所在的路徑。

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: azure-disk-pod
spec:
 containers:
  - image: microsoft/sample-aks-helloworld
    name: azure
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
 volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: myAKSDisk
          diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

使用 kubectl 來建立 Pod。

```azurecli-interactive
kubectl apply -f azure-disk-pod.yaml
```

您現在已有一個 Azure 磁碟掛接在 `/mnt/azure` 的執行中 Pod。

## <a name="next-steps"></a>後續步驟

深入了解使用 Azure 磁碟的 Kubernetes 磁碟區。

> [!div class="nextstepaction"]
> [Azure 磁碟的 Kubernetes 外掛程式][kubernetes-disks]

<!-- LINKS - external -->
[kubernetes-disks]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_disk/README.md
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/resource#az-resource-show
