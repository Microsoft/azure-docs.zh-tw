---
title: Azure Kubernetes Service (AKS) 上的 HTTP 應用程式路由附加元件
description: 使用 Azure Kubernetes Service (AKS) 上的 HTTP 應用程式路由附加元件。
services: container-service
author: lachie83
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/25/2018
ms.author: laevenso
ms.openlocfilehash: 93b9075831db356607438288a7c153dde2f8b210
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140694"
---
# <a name="http-application-routing"></a>HTTP 應用程式路由

HTTP 應用程式路由解決方案可讓您輕鬆存取已部署至 Azure Kubernetes Service (AKS) 叢集的應用程式。 啟用此解決方案時，它會在 AKS 叢集中設定輸入控制器。 部署應用程式時，此解決方案也會針對應用程式端點建立可公開存取的 DNS 名稱。

啟用附加元件時，它會在您的訂用帳戶中建立 DNS 區域。 如需 DNS 成本的詳細資訊，請參閱 [DNS 定價][dns-pricing]。

## <a name="http-routing-solution-overview"></a>HTTP 路由解決方案概觀

附加元件會部署兩個元件：[Kubernetes 輸入控制器][ingress]和 [External-DNS][external-dns] 控制器。

- **輸入控制器**：輸入控制器會使用 LoadBalancer 類型的 Kubernetes 服務來向網際網路公開。 輸入控制器會監看並實作 [Kubernetes 輸入資源][ingress-resource]，以建立應用程式端點的路由。
- **External-DNS 控制器**：監看 Kubernetes 輸入資源，並在叢集特有的 DNS 區域中建立 DNS A 記錄。

## <a name="deploy-http-routing-cli"></a>部署 HTTP 路由：CLI

部署 AKS 叢集時，可以使用 Azure CLI 來啟用 HTTP 應用程式路由附加元件。 若要這樣做，請使用 [az aks create][az-aks-create] 命令並搭配 `--enable-addons` 引數。

```azurecli
az aks create --resource-group myResourceGroup --name myAKSCluster --enable-addons http_application_routing
```

您也可以使用 [az aks enable-addons][az-aks-enable-addons] 命令，在現有 AKS 叢集上啟用 HTTP 路由。 若要在現有叢集上啟用 HTTP 路由，請新增 `--addons` 參數並指定 *http_application_routing*，如下列範例所示：

```azurecli
az aks enable-addons --resource-group myResourceGroup --name myAKSCluster --addons http_application_routing
```

部署或更新叢集之後，請使用 [az aks show][az-aks-show] 命令來擷取 DNS 區域名稱。 需要此名稱，才能將應用程式部署至 AKS 叢集。

```azurecli
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table

Result
-----------------------------------------------------
9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io
```

## <a name="deploy-http-routing-portal"></a>部署 HTTP 路由：入口網站

部署 AKS 叢集時，可透過 Azure 入口網站啟用 HTTP 應用程式路由附加元件。

![啟用 HTTP 路由功能](media/http-routing/create.png)

部署叢集之後，瀏覽至自動建立的 AKS 資源群組，並選取 DNS 區域。 記下 DNS 區域名稱。 需要此名稱，才能將應用程式部署至 AKS 叢集。

![取得 DNS 區域名稱](media/http-routing/dns.png)

## <a name="use-http-routing"></a>使用 HTTP 路由

HTTP 應用程式路由解決方案只會在所標註的輸入資源上觸發，如下所示：

```yaml
annotations:
  kubernetes.io/ingress.class: addon-http-application-routing
```

建立名為 **samples-http-application-routing.yaml** 的檔案，然後將下列 YAML 複製進來。 在第 43 行上，使用本文上一個步驟所收集的 DNS 區域名稱來更新 `<CLUSTER_SPECIFIC_DNS_ZONE>`。


```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: party-clippy
spec:
  template:
    metadata:
      labels:
        app: party-clippy
    spec:
      containers:
      - image: r.j3ss.co/party-clippy
        name: party-clippy
        tty: true
        command: ["party-clippy"]
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: party-clippy
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: party-clippy
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: party-clippy
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: party-clippy.<CLUSTER_SPECIFIC_DNS_ZONE>
    http:
      paths:
      - backend:
          serviceName: party-clippy
          servicePort: 80
        path: /
```

使用 [kubectl apply][kubectl-apply] 命令來建立資源。

```bash
$ kubectl apply -f samples-http-application-routing.yaml

deployment "party-clippy" created
service "party-clippy" created
ingress "party-clippy" created
```

使用 cURL 或瀏覽器來瀏覽至 samples-http-application-routing.yaml 檔案 host 區段中所指定的主機名稱。 應用程式最多需要一分鐘的時間，就能透過網際網路使用。

```bash
$ curl party-clippy.471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io

 _________________________________
/ It looks like you're building a \
\ microservice.                   /
 ---------------------------------
 \
  \
     __
    /  \
    |  |
    @  @
    |  |
    || |/
    || ||
    |\_/|
    \___/

```

## <a name="remove-http-routing"></a>移除 HTTP 路由

HTTP 路由解決方案可以使用 Azure CLI 來移除。 若要執行此動作，請執行下列命令，並替代您的 AKS 叢集和資源群組名稱。

```azurecli
az aks disable-addons --addons http_application_routing --name myAKSCluster --resource-group myResourceGroup --no-wait
```

## <a name="troubleshoot"></a>疑難排解

使用 [kubectl logs][kubectl-logs] 命令來檢視 External-DNS 應用程式的應用程式記錄。 記錄應該確認是否已成功建立 A 和 TXT DNS 記錄。

```
$ kubectl logs -f deploy/addon-http-application-routing-external-dns -n kube-system

time="2018-04-26T20:36:19Z" level=info msg="Updating A record named 'party-clippy' to '52.242.28.189' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
time="2018-04-26T20:36:21Z" level=info msg="Updating TXT record named 'party-clippy' to '"heritage=external-dns,external-dns/owner=default"' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
```

這些記錄也可在 Azure 入口網站中的 DNS 區域資源上看到。

![取得 DNS 記錄](media/http-routing/clippy.png)

使用 [kubectl logs][kubectl-logs] 命令來檢視 Nginx 輸入控制器的應用程式記錄。 記錄應該確認輸入資源的 `CREATE` 以及是否已重新載入控制器。 系統會記錄所有 HTTP 活動。

```bash
$ kubectl logs -f deploy/addon-http-application-routing-nginx-ingress-controller -n kube-system

-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.13.0
  Build:      git-4bc943a
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------

I0426 20:30:12.212936       9 flags.go:162] Watching for ingress class: addon-http-application-routing
W0426 20:30:12.213041       9 flags.go:165] only Ingress with class "addon-http-application-routing" will be processed by this ingress controller
W0426 20:30:12.213505       9 client_config.go:533] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0426 20:30:12.213752       9 main.go:181] Creating API client for https://10.0.0.1:443
I0426 20:30:12.287928       9 main.go:225] Running in Kubernetes Cluster version v1.8 (v1.8.11) - git (clean) commit 1df6a8381669a6c753f79cb31ca2e3d57ee7c8a3 - platform linux/amd64
I0426 20:30:12.290988       9 main.go:84] validated kube-system/addon-http-application-routing-default-http-backend as the default backend
I0426 20:30:12.294314       9 main.go:105] service kube-system/addon-http-application-routing-nginx-ingress validated as source of Ingress status
I0426 20:30:12.426443       9 stat_collector.go:77] starting new nginx stats collector for Ingress controller running in namespace  (class addon-http-application-routing)
I0426 20:30:12.426509       9 stat_collector.go:78] collector extracting information from port 18080
I0426 20:30:12.448779       9 nginx.go:281] starting Ingress controller
I0426 20:30:12.463585       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-nginx-configuration", UID:"2588536c-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"559", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-nginx-configuration
I0426 20:30:12.466945       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-tcp-services", UID:"258ca065-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"561", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-tcp-services
I0426 20:30:12.467053       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-udp-services", UID:"259023bc-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"562", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-udp-services
I0426 20:30:13.649195       9 nginx.go:302] starting NGINX process...
I0426 20:30:13.649347       9 leaderelection.go:175] attempting to acquire leader lease  kube-system/ingress-controller-leader-addon-http-application-routing...
I0426 20:30:13.649776       9 controller.go:170] backend reload required
I0426 20:30:13.649800       9 stat_collector.go:34] changing prometheus collector from  to default
I0426 20:30:13.662191       9 leaderelection.go:184] successfully acquired lease kube-system/ingress-controller-leader-addon-http-application-routing
I0426 20:30:13.662292       9 status.go:196] new leader elected: addon-http-application-routing-nginx-ingress-controller-5cxntd6
I0426 20:30:13.763362       9 controller.go:179] ingress backend successfully reloaded...
I0426 21:51:55.249327       9 event.go:218] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"party-clippy", UID:"092c9599-499c-11e8-a5e1-0a58ac1f0ef2", APIVersion:"extensions", ResourceVersion:"7346", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/party-clippy
W0426 21:51:57.908771       9 controller.go:775] service default/party-clippy does not have any active endpoints
I0426 21:51:57.908951       9 controller.go:170] backend reload required
I0426 21:51:58.042932       9 controller.go:179] ingress backend successfully reloaded...
167.220.24.46 - [167.220.24.46] - - [26/Apr/2018:21:53:20 +0000] "GET / HTTP/1.1" 200 234 "" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)" 197 0.001 [default-party-clippy-80] 10.244.0.13:8080 234 0.004 200
```

## <a name="clean-up"></a>清除

移除本文中所建立的相關聯 Kubernetes 物件。

```bash
$ kubectl delete -f samples-http-application-routing.yaml

deployment "party-clippy" deleted
service "party-clippy" deleted
ingress "party-clippy" deleted
```

## <a name="next-steps"></a>後續步驟

如需在 AKS 中安裝 HTTPS 所保護之輸入控制器的相關資訊，請參閱 [Azure Kubernetes Service (AKS) 上的 HTTPS 輸入][ingress-https]。

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-show]: /cli/azure/aks?view=azure-cli-latest#az-aks-show
[ingress-https]: ./ingress.md
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons


<!-- LINKS - external -->
[dns-pricing]: https://azure.microsoft.com/pricing/details/dns/
[external-dns]: https://github.com/kubernetes-incubator/external-dns
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[ingress-resource]: https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource
