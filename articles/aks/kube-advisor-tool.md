---
title: 檢查 Azure 上 Kubernetes 部署中的最佳做法實作
description: 了解如何使用 kube-advisor 檢查 Azure Kubernetes Service 上部署中的最佳做法實作
services: container-service
author: seanmck
ms.service: container-service
ms.topic: troubleshooting
ms.date: 09/13/2018
ms.author: seanmck
ms.openlocfilehash: e29308b7b1c17377cf1d627f2a32a2ba6ea4d077
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310087"
---
# <a name="checking-for-kubernetes-best-practices-in-your-cluster"></a>檢查叢集中的 Kubernetes 最佳做法

為確保最佳的應用程式效能和恢復能力，您應在 Kubernetes 部署上遵循的最佳做法有許多種。 您可以使用 kube-advisor 工具來尋找未遵循這些建議的部署。

## <a name="about-kube-advisor"></a>關於 kube-advisor

[kube-advisor 工具][kube-advisor-github]是專門設計為在叢集中執行的單一容器。 它會向 Kubernetes API 伺服器查詢部署的相關資訊，並傳回一套建議改善事項。

> [!NOTE]
> Microsoft 致力支援 kube advisor 工具。 問題和建議應在 GitHub 上建檔。

## <a name="running-kube-advisor"></a>執行 kube-advisor

若要在設定為[角色型存取控制 (RBAC)](aad-integration.md) 的叢集上執行此工具，請使用下列命令。 第一個命令會建立 Kubernetes 服務帳戶。 第二個命令會使用該服務帳戶在 Pod 中執行此工具，並設定為 Pod 結束之後便會刪除。 

```bash
kubectl apply -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml?token=ABLLDrNcuHMro9jQ0xduCaEbpzLupzQUks5bh3RhwA%3D%3D

kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never --overrides="{ \"apiVersion\": \"v1\", \"spec\": { \"serviceAccountName\": \"kube-advisor\" } }"
```

若您不是使用 RBAC，可以執行如下所示的命令：

```bash
kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never
```

幾秒內應該會顯示一個表格，說明您部署適用的可能改善事項。

![Kube-advisor 輸出](media/kube-advisor-tool/kube-advisor-output.png)

## <a name="checks-performed"></a>執行的檢查

此工具會驗證數個 Kubernetes 最佳做法，每個最佳做法都有專屬的建議補救措施。

### <a name="resource-requests-and-limits"></a>資源要求和限制

Kubernetes 支援[對 Pod 規格定義資源要求和限制][kube-cpumem]。 要求會定義執行容器所需的最小 CPU 和記憶體。 限制則會定義允許的最大 CPU 和記憶體。

根據預設，不會對 Pod 規格設定要求或限制。 這可能會導致過度排程節點和耗盡容器。 kube-advisor 工具會反白顯示不具有要求和限制設定的的 Pod。

## <a name="cleaning-up"></a>清除

如果您的叢集已啟用 RBAC，使用下列命令執行此工具之後，您可以清除 `ClusterRoleBinding`：

```bash
kubectl delete -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml?token=ABLLDrNcuHMro9jQ0xduCaEbpzLupzQUks5bh3RhwA%3D%3D
```

如果您對未啟用 RBAC 的叢集執行此工具，則不需要進行清除。

## <a name="next-steps"></a>後續步驟

- [針對 Azure Kubernetes Service 的問題進行疑難排解](troubleshooting.md)

<!-- RESOURCES -->

[kube-cpumem]: https://github.com/Azure/azure-quickstart-templates
[kube-advisor-github]: https://github.com/azure/kube-advisor