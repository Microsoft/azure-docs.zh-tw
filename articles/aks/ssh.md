---
title: 透過 SSH 連線至 Azure Kubernetes Service (AKS) 叢集
description: 了解如何使用 Azure Kubernetes Service (AKS) 叢集節點建立 SSH 連線，以進行疑難排解和維護工作。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 08/21/2018
ms.author: iainfou
ms.openlocfilehash: f0097c2ba42378c66bedd614032b63c23a2483dd
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42447051"
---
# <a name="connect-with-ssh-to-azure-kubernetes-service-aks-cluster-nodes-for-maintenance-or-troubleshooting"></a>使用 SSH 連線到 Azure Kubernetes Service (AKS) 叢集節點以進行維護或疑難排解

在 Azure Kubernetes Service (AKS) 叢集的生命週期中，您可能需要存取 AKS 節點。 此存取可能用於維護、記錄收集，或其他疑難排解作業。 AKS 節點是 Linux VM，因此可以使用 SSH 進行存取。 基於安全考量，AKS 節點不會公開至網際網路。

本文會示範如何使用私人 IP 位址以 AKS 節點建立 SSH 連線。

## <a name="add-your-public-ssh-key"></a>新增公開 SSH 金鑰

根據預設，建立 AKS 叢集時，會產生 SSH 金鑰。 如果在建立 AKS 叢集時您未指定自己的 SSH 金鑰，請新增公用 SSH 金鑰至 AKS 節點。 

若要新增 SSH 金鑰至 AKS 節點，請完成下列步驟：

1. 使用 [az aks show][az-aks-show] 取得 AKS 叢集資源的資源群組名稱。 提供您自己的核心資源群組和 AKS 叢集名稱：

    ```azurecli
    az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
    ```

1. 使用 [az vm list][az-vm-list] 命令列出 AKS 叢集資源群組中的虛擬機器。 這些虛擬機器是您的 AKS 節點：

    ```azurecli
    az vm list --resource-group MC_myResourceGroup_myAKSCluster_eastus -o table
    ```

    下列範例輸出顯示 AKS 節點：

    ```
    Name                      ResourceGroup                                  Location
    ------------------------  ---------------------------------------------  ----------
    aks-nodepool1-79590246-0  MC_myResourceGroupAKS_myAKSClusterRBAC_eastus  eastus
    ```

1. 若要將 SSH 金鑰新增至節點，請使用 [az vm user update][az-vm-user-update] 命令。 提供資源群組名稱，以及上一個步驟中取得的其中一個 AKS 節點。 根據預設，AKS 節點的使用者名稱是 *azureuser*。 提供您自己的 SSH 公開金鑰位置，例如 *~/.ssh/id_rsa.pub*，或貼上 SSH 公開金鑰的內容：

    ```azurecli
    az vm user update \
      --resource-group MC_myResourceGroup_myAKSCluster_eastus \
      --name aks-nodepool1-79590246-0 \
      --username azureuser \
      --ssh-key-value ~/.ssh/id_rsa.pub
    ```

## <a name="get-the-aks-node-address"></a>取得 AKS 節點位址

AKS 節點不會公開至網際網路。 對於 SSH 至 AKS 節點，您可以使用私人 IP 位址。

使用 [az vm list-ip-addresses][az-vm-list-ip-addresses] 命令檢視 AKS 叢集節點的私人 IP 位址。 提供您自己的 AKS 叢集資源群組名稱，此名稱會在前一個 [az-aks-show][az-aks-show] 步驟中取得：

```azurecli
az vm list-ip-addresses --resource-group MC_myAKSCluster_myAKSCluster_eastus -o table
```

下列範例輸出顯示 AKS 節點的私人 IP 位址：

```
VirtualMachine            PrivateIPAddresses
------------------------  --------------------
aks-nodepool1-79590246-0  10.240.0.4
```

## <a name="create-the-ssh-connection"></a>建立 SSH 連線

若要建立到 AKS 節點的 SSH 連線，您可以在 AKS 叢集中執行協助程式 Pod。 此協助程式 Pod 為您提供到叢集的 SSH 存取權，以及額外的 SSH 節點存取權。 若要建立和使用此協助程式 Pod，請完成下列步驟：

1. 執行 `debian` 容器映像，並將終端機工作階段與它連結。 您可以使用此容器搭配 AKS 叢集中的任何節點來建立 SSH 工作階段：

    ```console
    kubectl run -it --rm aks-ssh --image=debian
    ```

1. 基底 Debian 映像不包含 SSH 元件。 終端機工作階段連線到容器後，請使用 `apt-get` 安裝 SSH 用戶端，如下所示：

    ```console
    apt-get update && apt-get install openssh-client -y
    ```

1. 在未連線到容器的新終端機視窗中，，請使用 [kubectl get pods][kubectl-get] 命令列出 AKS 叢集上的 Pod。 在前一個步驟中建立的 Pod 名稱開頭為 *aks-ssh*，如下列範例所示：

    ```
    $ kubectl get pods
    
    NAME                       READY     STATUS    RESTARTS   AGE
    aks-ssh-554b746bcf-kbwvf   1/1       Running   0          1m
    ```

1. 在本文的第一個步驟中，您將公用 SSH 金鑰新增至 AKS 節點。 現在，將私人 SSH 金鑰複製到 Pod 中。 這個私密金鑰會用來將 SSH 建立至 AKS 節點中。

    提供上一個步驟中所取得您自己的 *aks-ssh* Pod 名稱。 如有需要，將 *~/.ssh/id_rsa* 變更到私人 SSH 金鑰的位置：

    ```console
    kubectl cp ~/.ssh/id_rsa aks-ssh-554b746bcf-kbwvf:/id_rsa
    ```

1. 回到容器的終端機工作階段，更新複製的 `id_rsa` 私人 SSH 金鑰使用權限，讓它是使用者唯讀：

    ```console
    chmod 0600 id_rsa
    ```

1. 現在，建立到 AKS 節點的 SSH 連線。 同樣地，AKS 節點的預設使用者名稱是 *azureuser*。 在 SSH 金鑰首次受到信任時，接受提示以繼續進行連線。 接著會提供您 AKS 節點的 bash 提示：

    ```console
    $ ssh -i id_rsa azureuser@10.240.0.4
    
    ECDSA key fingerprint is SHA256:A6rnRkfpG21TaZ8XmQCCgdi9G/MYIMc+gFAuY9RUY70.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.240.0.4' (ECDSA) to the list of known hosts.
    
    Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1018-azure x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      Get cloud support with Ubuntu Advantage Cloud Guest:
        http://www.ubuntu.com/business/services/cloud
    
    [...]
    
    azureuser@aks-nodepool1-79590246-0:~$
    ```

## <a name="remove-ssh-access"></a>移除 SSH 存取

完成時，`exit` SSH 工作階段，然後 `exit` 互動式容器工作階段。 這個容器工作階段關閉時，會刪除用來從 AKS 叢集存取 SSH 的 Pod。

## <a name="next-steps"></a>後續步驟

如需其他疑難排解資料，您可以[檢視 kubelet 記錄][view-kubelet-logs]或[檢視 Kubernetes 主要節點記錄][view-master-logs]。

<!-- EXTERNAL LINKS -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- INTERNAL LINKS -->
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-vm-list]: /cli/azure/vm#az-vm-list
[az-vm-user-update]: /cli/azure/vm/user#az-vm-user-update
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[view-kubelet-logs]: kubelet-logs.md
[view-master-logs]: view-master-logs.md
