---
title: Azure Kubernetes 網路原則 | Microsoft Docs
description: 了解 Kubernetes 網路原則來保護 Kubernetes 叢集。
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/25/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: b4f8577724781e5df10846a5fc4e30c8320403f2
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219765"
---
# <a name="azure-kubernetes-network-policies-overview"></a>Azure Kubernetes 網路原則概觀

網路原則提供 Pod 的微型分割，就像網路安全性群組 (NSG) 提供 VM 的微型分割一樣。 Azure 網路原則實作支援標準 Kubernetes 網路原則規格。 您可以使用標籤選取一組 Pod，並定義輸入和輸出規則清單，而這些規則指定允許進出這些 Pod 的流量類型。 在 [Kubernetes 文件](https://kubernetes.io/docs/concepts/services-networking/network-policies/)中深入了解 Kubernetes 網路原則。

![Kubernetes 網路原則概觀](./media/kubernetes-network-policies/kubernetes-network-policies-overview.png)

Azure 網路原則與 Azure CNI 搭配運作，而 Azure CNI 提供容器的 VNet 整合。 目前只有 Linux 節點上才支援。 實作會根據定義的原則來設定「Linux IP 資料表」規則，以強制執行流量篩選。

## <a name="planning-security-for-your-kubernetes-cluster"></a>規劃 Kubernetes 叢集的安全性
實作叢集的安全性時，請使用網路安全性群組 (NSG) 來篩選北南流量 (即進出您叢集子網路的流量)，並針對東西流量使用 Kubernetes 網路原則 (即 叢集中 Pod 之間的流量)。

## <a name="using-azure-kubernetes-network-policies"></a>使用 Azure Kubernetes 網路原則
您可以透過下列方式使用 Azure 網路原則，以提供 Pod 的微型分割。

### <a name="acs-engine"></a>ACS-engine
ACS-Engine 是一種工具，產生在 Azure 中部署 Kubernetes 叢集的 Azure Resource Manager 範本。 叢集設定是在 JSON 檔案中指定，產生範本時會將該檔案傳遞到工具。 若要深入了解支援叢集設定及其描述的完整清單，請參閱＜Microsoft Azure Container Service 引擎 - 叢集定義＞。

若要在使用 acs-engine 所部署的叢集上啟用原則，請將叢集定義檔案中的 networkPolicy 設定值指定為 "azure"。

#### <a name="example-configuration"></a>設定範例

以下 JSON 範例設定會建立新的虛擬網路和子網路，並使用 Azure CNI 在其中部署 Kubernetes 叢集。 建議您使用 [記事本] 編輯 JSON 檔案。 
```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      “kubernetesConfig”: {
         "networkPolicy": "azure"
       }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "<specify a cluster name>",
      "vmSize": "Standard_D2s_v3"
    },
    "agentPoolProfiles": [
      {
        "name": "agentpool",
        "count": 2,
        "vmSize": "Standard_D2s_v3",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
   "linuxProfile": {
      "adminUsername": "<specify admin username>",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "<cut and paste your ssh key here>"
          }
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "<enter the client ID of your service principal here >",
      "secret": "<enter the password of your service principal here>"
    }
  }
}

```
### <a name="creating-your-own-kubernetes-cluster-in-azure"></a>在 Azure 中建立專屬 Kubernetes 叢集
實作可以用來提供自行部署的 Kubernetes 叢集 Pod 網路原則，而不需要依賴 ACS-Engine 這類工具。 在此情況下，您會先安裝 CNI 外掛程式，並在叢集的每部虛擬機器上啟用它。 如需詳細指示，請參閱[部署自行部署的 Kubernetes 叢集外掛程式](deploy-container-networking.md#deploy-plug-in-for-a-kubernetes-cluster)。

部署叢集之後，請執行下列 `kubectl` 命令下載 Azure 網路原則 *daemonset*，並將其套用至叢集。

  ```
  kubectl apply -f https://raw.githubusercontent.com/Azure/acs-engine/master/parts/k8s/addons/kubernetesmasteraddons-azure-npm-daemonset.yaml

  ```
解決方案也是開放原始碼，而程式碼位於 [Azure Container Networking repository](https://github.com/Azure/azure-container-networking/tree/master/npm) (Azure 容器網路存放庫)。



## <a name="next-steps"></a>後續步驟
- 了解 [Azure Kubernetes Service](../aks/intro-kubernetes.md)。
-  了解[容器網路服務](container-networking-overview.md)。
- [部署外掛程式](deploy-container-networking.md) (適用於 Kubernetes 叢集或 Docker 容器)。