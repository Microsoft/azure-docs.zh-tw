---
title: 調整 Azure Kubernetes Service (AKS) 叢集
description: 調整 Azure Kubernetes Service (AKS) 叢集。
services: container-service
author: gabrtv
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 11/15/2017
ms.author: gamonroy
ms.custom: mvc
ms.openlocfilehash: 577fff2e659759647ffc7e96158ebcbe5a88ab25
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33934680"
---
# <a name="scale-an-azure-kubernetes-service-aks-cluster"></a>調整 Azure Kubernetes Service (AKS) 叢集

將 AKS 叢集調整為不同數目的節點很簡單。  選取想要的節點數目，然後執行 `az aks scale` 命令。  縮減規模時，會將節點仔細地[隔離並清空][kubernetes-drain] \(英文\)，以將對執行中應用程式造成的中斷情況降到最低。  相應增加時，`az` 命令會一直等待，直到 Kubernetes 叢集將節點標示 `Ready` 為止。

## <a name="scale-the-cluster-nodes"></a>調整叢集節點

使用 `az aks scale` 命令調整叢集節點。 下列範例會將名為 *myAKSCluster* 的叢集調整成單一節點。

```azurecli-interactive
az aks scale --name myAKSCluster --resource-group myResourceGroup --node-count 1
```

輸出：

```json
{
  "id": "/subscriptions/<Subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "location": "eastus",
  "name": "myAKSCluster",
  "properties": {
    "accessProfiles": {
      "clusterAdmin": {
        "kubeConfig": "..."
      },
      "clusterUser": {
        "kubeConfig": "..."
      }
    },
    "agentPoolProfiles": [
      {
        "count": 1,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "myAKSCluster",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D2_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "myK8sClust-myResourceGroup-4f48ee",
    "fqdn": "myk8sclust-myresourcegroup-4f48ee-406cc140.hcp.eastus.azmk8s.io",
    "kubernetesVersion": "1.7.7",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "..."
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "e70c1c1c-0ca4-4e0a-be5e-aea5225af017",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "myResourceGroup",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="next-steps"></a>後續步驟

使用 AKS 教學課程深入了解部署和管理 AKS。

> [!div class="nextstepaction"]
> [AKS 教學課程][aks-tutorial]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md