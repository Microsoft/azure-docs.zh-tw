---
title: 在 Azure Kubernetes Service (AKS) 中建立內部網路的輸入控制器
description: 了解如何在 Azure Kubernetes Service (AKS) 叢集中安裝及設定內部私人網路的 NGINX 輸入控制器。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 08/30/2018
ms.author: iainfou
ms.openlocfilehash: 76ad9d21f7b328e7f201d227cdd9ace51c62a3fd
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355137"
---
# <a name="create-an-ingress-controller-to-an-internal-virtual-network-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes Service (AKS) 中建立內部虛擬網路的輸入控制器

輸入控制器是一項可為 Kubernetes 服務提供反向 Proxy、可設定的流量路由和 TLS 終止的軟體。 Kubernetes 輸入資源可用來設定個別 Kubernetes 服務的輸入規則和路由。 透過輸入控制器和輸入規則，您可以使用單一 IP 位址將流量路由至 Kubernetes 叢集中的多個服務。

本文示範如何在 Azure Kubernetes Service (AKS) 叢集中部署 [NGINX 輸入控制器][nginx-ingress]。 輸入控制器設定於內部私人虛擬網路和 IP 位址上。 不允許外部存取。 然後，會有兩個應用程式在 AKS 叢集中執行，且均可透過單一 IP 位址來存取。

您也可以：

- [建立具有外部網路連線的基本輸入控制器][aks-ingress-basic]
- [啟用 HTTP 應用程式路由附加元件][aks-http-app-routing]
- [使用動態公用 IP 建立輸入控制器，並設定 Let's Encrypt 以自動產生 TLS 憑證][aks-ingress-tls]
- [使用靜態公用 IP 位址建立輸入控制器，並設定 Let's Encrypt 以自動產生 TLS 憑證][aks-ingress-static-tls]

## <a name="before-you-begin"></a>開始之前

本文使用 Helm 來安裝 NGINX 輸入控制器、cert-manager 及範例 Web 應用程式。 您需要在 AKS 叢集內將 Helm 初始化，並使用適用於 Tiller 的服務帳戶。 如需設定及使用 Helm 的詳細資訊，請參閱[在 Azure Kubernetes Service (AKS) 中使用 Helm 安裝應用程式][use-helm]。

本文也需要您執行 Azure CLI 2.0.41 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="create-an-ingress-controller"></a>建立輸入控制器

根據預設，NGINX 輸入控制器會使用靜態公用 IP 位址指派來建立。 常見的設定需求是使用內部私人網路和 IP 位址。 此方法可讓您將對服務的存取限定於內部使用者，而不提供外部存取。

使用下列範例資訊清單檔建立名為 *internal-ingress.yaml* 的檔案。 此範例會將 *10.240.0.42* 指派給 *loadBalancerIP* 資源。 請提供您自己的內部 IP 位址，用於輸入控制器。 請確定此 IP 位址在您的虛擬網路內尚未被使用。

```yaml
controller:
  service:
    loadBalancerIP: 10.240.0.42
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

現在，使用 Helm 部署 nginx-ingress 圖表。 若要使用在上一個步驟中建立的資訊清單檔，請新增 `-f internal-ingress.yaml` 參數：

> [!TIP]
> 下列範例會在 `kube-system` 命名空間中安裝輸入控制器。 您可以視需要為自己的環境指定不同的命名空間。 如果您的 AKS 叢集未啟用 RBAC，請將 `--set rbac.create=false` 新增至命令。

```console
helm install stable/nginx-ingress --namespace kube-system -f internal-ingress.yaml
```

為 NGINX 輸入控制器建立 Kubernetes 負載平衡器服務時，會指派您的內部 IP 位址，如下列範例輸出所示：

```
$ kubectl get service -l app=nginx-ingress --namespace kube-system

NAME                                              TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
alternating-coral-nginx-ingress-controller        LoadBalancer   10.0.97.109   10.240.0.42   80:31507/TCP,443:30707/TCP   1m
alternating-coral-nginx-ingress-default-backend   ClusterIP      10.0.134.66   <none>        80/TCP                       1m
```

尚未建立任何輸入規則，因此，如果您瀏覽至內部 IP 位址，將會顯示 NGINX 輸入控制器的預設 404 頁面。 輸入規則會於下列步驟中進行設定。

## <a name="run-demo-applications"></a>執行示範應用程式

為了檢視運作中的輸入控制器，我們將在 AKS 叢集中執行兩個示範應用程式。 在此範例中，會使用 Helm 來部署簡單 'Hello world' 應用程式的兩個執行個體。

在您可以安裝範例 Helm 圖表之前，先將 Azure 範例存放庫新增至您的 Helm 環境，如下所示：

```console
helm repo add azure-samples https://azure-samples.github.io/helm-charts/
```

使用下列命令，從 Helm 圖表建立第一個示範應用程式：

```console
helm install azure-samples/aks-helloworld
```

現在，請安裝示範應用程式的第二個執行個體。 對第二個執行個體指定新的標題，以便在視覺上區分這兩個應用程式。 您也要指定唯一的服務名稱：

```console
helm install azure-samples/aks-helloworld --set title="AKS Ingress Demo" --set serviceName="ingress-demo"
```

## <a name="create-an-ingress-route"></a>建立輸入路由

這兩個應用程式現在都已在您的 Kubernetes 叢集上執行。 若要將流量路由至每個應用程式，請建立 Kubernetes 輸入資源。 輸入資源會設定將流量路由至這兩個應用程式之一的規則。

在下列範例中，傳至位址 `http://10.240.0.42/` 的流量會路由傳送至名為 `aks-helloworld` 的服務。 傳至位址 `http://10.240.0.42/hello-world-two` 的流量會路由至 `ingress-demo` 服務。

建立名為 `hello-world-ingress.yaml` 的檔案，並複製到下列範例 YAML 中。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: aks-helloworld
          servicePort: 80
      - path: /hello-world-two
        backend:
          serviceName: ingress-demo
          servicePort: 80
```

使用 `kubectl apply -f hello-world-ingress.yaml` 命令建立輸入資源。

```
$ kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
```

## <a name="test-the-ingress-controller"></a>測試輸入控制器

若要測試輸入控制器的路由，請使用 Web 用戶端瀏覽至這兩個應用程式。 如有需要，您可以從 AKS 叢集上的 Pod 快速測試此僅供內部使用的功能。 建立測試 Pod，並將終端機工作階段與它連結：

```console
kubectl run -it --rm aks-ingress-test --image=debian
```

使用 `apt-get` 在 Pod 中安裝 `curl`：

```console
apt-get update && apt-get install -y curl
```

現在，請使用 `curl` 存取 Kubernetes 輸入控制器的位址，例如 *http://10.240.0.42*。 請提供您自己在本文的第一個步驟中部署輸入控制器時所指定的內部 IP 位址。

```console
curl -L http://10.240.0.42
```

先前並未提供位址的其他路徑，因此輸入控制器預設為 */* 路由。 此時會傳回第一個示範應用程式，如下列簡要範例輸出所示：

```
$ curl -L 10.240.0.42

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>Welcome to Azure Kubernetes Service (AKS)</title>
[...]
```

現在，將 */hello-world-two* 路徑新增至位址，例如 *http://10.240.0.42/hello-world-two*。 此時會傳回含有自訂標題的第二個示範應用程式，如下列簡要範例輸出所示：

```
$ curl -L -k http://10.240.0.42/hello-world-two

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>AKS Ingress Demo</title>
[...]
```

## <a name="next-steps"></a>後續步驟

本文包含 AKS 的一些外部元件。 若要深入了解這些元件，請參閱下列專案頁面：

- [Helm CLI][helm-cli]
- [NGINX 輸入控制器][nginx-ingress]

您也可以：

- [建立具有外部網路連線的基本輸入控制器][aks-ingress-basic]
- [啟用 HTTP 應用程式路由附加元件][aks-http-app-routing]
- [使用動態公用 IP 建立輸入控制器，並設定 Let's Encrypt 以自動產生 TLS 憑證][aks-ingress-tls]
- [使用靜態公用 IP 位址建立輸入控制器，並設定 Let's Encrypt 以自動產生 TLS 憑證][aks-ingress-static-tls]

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm#install-helm-cli
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
