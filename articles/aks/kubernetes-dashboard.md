---
title: 使用 Web UI 管理 Azure Kubernetes 叢集
description: 了解如何使用 Azure Kubernetes Service (AKS) 中內建的 Kubernetes Web UI 儀表板
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 07/09/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: af48af596e86e0eb09fe45deabe13beedef57cd2
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39307920"
---
# <a name="access-the-kubernetes-dashboard-with-azure-kubernetes-service-aks"></a>存取 Azure Kubernetes Service (AKS) 中的 Kubernetes 儀表板

Kubernetes 包含的 Web 儀表板可用來執行基本的管理作業。 本文將說明如何使用 Azure CLI 來存取 Kubernetes 儀表板，然後引導您完成一些基本的儀表板作業。 如需 Kubernetes 儀表板的詳細資訊，請參閱 [Kubernetes Web UI 儀表板][kubernetes-dashboard]。

## <a name="before-you-begin"></a>開始之前

本文件中詳述的步驟假設您已建立 AKS 叢集，並建立與叢集的 `kubectl` 連線。 如果您需要建立 AKS 叢集，請參閱 [AKS 快速入門][aks-quickstart]。

您也必須安裝和設定 Azure CLI 版本 2.0.27 或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][install-azure-cli]。

## <a name="start-kubernetes-dashboard"></a>啟動 Kubernetes 儀表板

若要啟動 Kubernetes 儀表板，請使用 [az aks browse][az-aks-browse] 命令。 下列範例會在名為 myResourceGroup 的資源群組中，針對名為 myAKSCluster 的叢集開啟儀表：

```azurecli
az aks browse --resource-group myResourceGroup --name myAKSCluster
```

此命令會在您的開發系統與 Kubernetes API 之間建立 Proxy，並開啟 Kubernetes 儀表板的網頁瀏覽器。

### <a name="for-rbac-enabled-clusters"></a>對於已啟用 RBAC 的叢集

如果您的 AKS 叢集已使用 RBAC，必須先建立 ClusterRoleBinding，才能正確地存取儀表板。 根據預設，Kubernetes 儀表板會部署最低限度的讀取權限，並且會顯示 RBAC 存取錯誤。 Kubernetes 儀表板目前不支援透過使用者提供的認證，來決定存取層級，而是使用授與服務帳戶的角色。 叢集系統管理員可選擇將其他存取權授與 kubernetes-dashboard 服務帳戶，但這可能是權限提高的向量。 您也可以整合 Azure Active Directory 驗證，以提供更細微的存取層級。

若要建立繫結，請使用 [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] 命令，如下列範例所示。 

> [!WARNING]
> 此範例繫結不會套用任何額外的驗證元件，並且可能導致不安全的使用方式。 可存取此 URL 的任何人都可使用 Kubernetes 儀表板。 請不要讓 Kubernetes 儀表板可公開使用。
>
> 如需有關使用不同驗證方法的詳細資訊，請參閱 Kubernetes 儀表板 Wiki 上的[存取控制][dashboard-authentication]。

```console
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```

您現在可以在啟用 RBAC 的叢集中存取 Kubernetes 儀表板。 若要啟動 Kubernetes 儀表板，請使用 [az aks browse][az-aks-browse] 命令，如前面步驟所述。

## <a name="run-an-application"></a>執行應用程式

在 Kubernetes 儀表板中，按一下右上方視窗中的 [建立] 按鈕。 將部署命名為 `nginx`，並針對容器映像名稱輸入 `nginx:latest`。 在 [服務] 下選取 [外部]，針對連接埠和目標連接埠輸入 `80`。

準備就緒時，按一下 [部署] 建立部署。

![Kubernetes 服務建立對話方塊](./media/container-service-kubernetes-ui/create-deployment.png)

## <a name="view-the-application"></a>檢視應用程式

在 Kubernetes 儀表板上可以看到應用程式的狀態。 只要應用程式在執行中，每個元件旁就會有綠色的核取方塊。

![Kubernetes Pods](./media/container-service-kubernetes-ui/complete-deployment.png)

若要查看應用程式 Pod 的詳細資訊，請按一下左側功能表中的 **Pod**，然後選取 **NGINX** Pod。 這裡可以看到 Pod 的特定資訊，例如資源耗用量。

![Kubernetes 資源](./media/container-service-kubernetes-ui/running-pods.png)

若要尋找應用程式的 IP 位址，請按一下左側功能表中的 [服務]，然後選取 **NGINX** 服務。

![nginx 檢視](./media/container-service-kubernetes-ui/nginx-service.png)

## <a name="edit-the-application"></a>編輯應用程式

除建立和檢視應用程式之外，Kubernetes 儀表板還可用來編輯和更新應用程式部署。

若要編輯部署，請按一下左側功能表中的 [部署]，然後選取 **NGINX** 部署。 最後，選取右上方導覽列中的 [編輯]。

![Kubernetes 編輯](./media/container-service-kubernetes-ui/view-deployment.png)

找出 `spec.replica` 值，此值應為 1，請將它變更為 3。 如此做，NGINX 部署的複本計數會從 1 增加至 3。

就緒後請選取 [更新]。

![Kubernetes 編輯](./media/container-service-kubernetes-ui/edit-deployment.png)

## <a name="next-steps"></a>後續步驟

如需 Kubernetes 儀表板的詳細資訊，請參閱 Kubernetes 文件。

> [!div class="nextstepaction"]
> [Kubernetes Web UI 儀表板][kubernetes-dashboard]

<!-- LINKS - external -->
[kubernetes-dashboard]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
[dashboard-authentication]: https://github.com/kubernetes/dashboard/wiki/Access-control
[kubectl-create-clusterrolebinding]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-clusterrolebinding-em-
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-browse]: /cli/azure/aks#az-aks-browse
