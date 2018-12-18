---
title: 在 Azure Kubernetes 中運用 Helm 部署容器
description: 使用 Helm 封裝工具， 在 Azure Container Service 中將容器部署在 Kubernetes 叢集
services: container-service
author: sauryadas
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/10/2017
ms.author: saudas
ms.custom: mvc
ms.openlocfilehash: 882e785968f94473e80c7a14e5a68498add37735
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38634148"
---
# <a name="use-helm-to-deploy-containers-on-a-kubernetes-cluster"></a>使用 Helm 在 Kubernetes 叢集部署容器

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

[Helm](https://github.com/kubernetes/helm/) 是開放原始碼的封裝工具，可協助您安裝和管理 Kubernetes 應用程式的生命週期。 Helm 類似於 Apt-get 和 Yum 等 Linux 封裝管理工具，可用於管理 Kubernetes 圖表 (即預先設定的 Kubernetes 資源封裝)。 本文將說明如何在部署於 Azure Container Service 中的 Kubernetes 叢集使用 Helm。

Helm 包含兩個元件︰ 
* Helm CLI是在本機或雲端執行的用戶端  

* Tiller是在 Kubernetes 叢集執行的伺服器，管理 Kubernetes 應用程式的生命週期 
 
## <a name="prerequisites"></a>先決條件

* 在 Azure Container Service 中[建立 Kubernetes 叢集](container-service-kubernetes-walkthrough.md)

* 在本機[安裝和設定 `kubectl`](../container-service-connect.md)

* 在本機[安裝 Helm](https://github.com/kubernetes/helm/blob/master/docs/install.md)

## <a name="helm-basics"></a>Helm 基本概念 

若要檢視您安裝 Tiller 及部署您應用程式的 Kubernetes 叢集的相關資訊，請輸入下列命令︰

```bash
kubectl cluster-info 
```
![kubectl 叢集資訊](./media/container-service-kubernetes-helm/clusterinfo.png)
 
安裝 Helm 之後，輸入下列命令，便可在 Kubernetes 叢集安裝 Tiller︰

```bash
helm init --upgrade
```
順利完成後，您會看到如下所示的輸出︰

![Tiller 安裝](./media/container-service-kubernetes-helm/tiller-install.png)
 
 
 
 
若要檢視存放庫中所有的 Helm 圖表，輸入下列命令︰

```bash 
helm search 
```

您應該會看到如以下的輸出：

![Helm 搜尋](./media/container-service-kubernetes-helm/helm-search.png)
 
若要更新圖表以取得最新版本，請輸入︰

```bash 
helm repo update 
```
## <a name="deploy-an-nginx-ingress-controller-chart"></a>部署 Nginx 輸入控制器圖表 
 
若要部署 Nginx 輸入控制器圖表，請輸入單一命令︰

```bash
helm install stable/nginx-ingress 
```
![部署輸入控制器](./media/container-service-kubernetes-helm/nginx-ingress.png)

如果您輸入 `kubectl get svc` 來檢視所有在叢集執行的服務，您會看到 IP 位址指派至輸入控制器。 (如果指派進行中，您會看到 `<pending>`。 它需要幾分鐘來完成。) 

指派 IP 位址後，瀏覽至外部 IP 位址的值，以查看 Nginx 後端執行。 
 
![輸入 IP 位址](./media/container-service-kubernetes-helm/ingress-ip-address.png)


若要查看安裝於叢集上的圖表清單，請輸入︰

```bash
helm list 
```

您可以將命令縮寫成 `helm ls`。
 
 
 
 
## <a name="deploy-a-mariadb-chart-and-client"></a>部署 MariaDB 圖表和用戶端

現在部署 MariaDB 圖表和 MariaDB 用戶端來連接至資料庫。

若要部署 MariaDB 圖表，輸入下列命令︰

```bash
helm install --name v1 stable/mariadb
```

其中 `--name` 是用於標示版本的標記。

> [!TIP]
> 如果部署失敗，執行 `helm repo update`，然後再試一次。
>
 
 
若要檢視部署在叢集中的所有圖表，請輸入︰

```bash 
helm list
```
 
若要檢視執行在叢集中的所有部署，請輸入︰

```bash
kubectl get deployments 
``` 
 
 
最後，若要執行 Pod 來存取用戶端，請輸入︰

```bash
kubectl run v1-mariadb-client --rm --tty -i --image bitnami/mariadb --command -- bash  
``` 
 
 
若要連接至用戶端，請輸入下列命令，將 `v1-mariadb` 更換為您的部署名稱︰

```bash
sudo mysql –h v1-mariadb
```
 
 
您現在可以使用標準 SQL 命令來建立資料庫、資料表等。例如，`Create DATABASE testdb1;` 會建立空白資料庫。 
 
 
 
## <a name="next-steps"></a>後續步驟

* 如需管理 Kubernetes 圖表的詳細資訊，請參閱[Helm 文件](https://github.com/kubernetes/helm/blob/master/docs/index.md)。 

