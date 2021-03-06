---
title: 使用自訂 traefik 輸入控制器並設定 HTTPS
services: azure-dev-spaces
ms.date: 12/10/2019
ms.topic: conceptual
description: 瞭解如何設定 Azure Dev Spaces 以使用自訂 traefik 輸入控制器，並使用該輸入控制器來設定 HTTPS
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, 容器, Helm, 服務網格, 服務網格路由傳送, kubectl, k8s
ms.custom: devx-track-javascript
ms.openlocfilehash: 3938209e80eb211afc332997b5b241c12a0f6eb9
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212461"
---
# <a name="use-a-custom-traefik-ingress-controller-and-configure-https"></a>使用自訂 traefik 輸入控制器並設定 HTTPS

本文說明如何將 Azure Dev Spaces 設定為使用自訂 traefik 輸入控制器。 本文也會說明如何將該自訂輸入控制器設定為使用 HTTPS。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有帳戶，您可以建立[免費帳戶][azure-account-create]。
* [已安裝 Azure CLI][az-cli]。
* [Azure Kubernetes Service (AKS) 已啟用 Azure Dev Spaces 的叢集][qs-cli]。
* 已安裝[kubectl][kubectl] 。
* [已安裝 Helm 3][helm-installed]。
* 具有[DNS 區域][dns-zone][的自訂網域][custom-domain]。 本文假設自訂網域和 DNS 區域位於與您的 AKS 叢集相同的資源群組中，但您可以在不同的資源群組中使用自訂網域和 DNS 區域。

## <a name="configure-a-custom-traefik-ingress-controller"></a>設定自訂 traefik 輸入控制器

使用 [kubectl][kubectl]（Kubernetes 命令列用戶端）連接到您的叢集。 若要設定 `kubectl` 以連線到 Kubernetes 叢集，請使用 [az aks get-credentials][az-aks-get-credentials] 命令。 此命令會下載憑證並設定 Kubernetes CLI 以供使用。

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKS
```

若要驗證針對您叢集的連線，請使用 [kubectl get][kubectl-get] 命令來傳回叢集節點的清單。

```console
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
```

新增 [正式的穩定 Helm 存放庫][helm-stable-repo]，其中包含 traefik 輸入控制器 Helm 圖表。

```console
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

為 traefik 輸入控制器建立 Kubernetes 命名空間，並使用安裝它 `helm` 。

> [!NOTE]
> 如果您的 AKS 叢集未啟用 RBAC，請移除 *--set RBAC. enabled = true* 參數。

```console
kubectl create ns traefik
helm install traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set fullnameOverride=customtraefik --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0
```

> [!NOTE]
> 上述範例會建立輸入控制器的公用端點。 如果您需要改用輸入控制器的私用端點，請新增 *--set 服務。附注。\\ \\ kubernetes \\ . io/azure-load-平衡器-internal "= true* 參數至 *helm install* 命令。
> ```console
> helm install traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set fullnameOverride=customtraefik --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --set service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true --version 1.85.0
> ```
> 此私人端點會在您 AKS 叢集部署所在的虛擬網路中公開。

使用 [kubectl get][kubectl-get]取得 traefik 輸入控制器服務的 IP 位址。

```console
kubectl get svc -n traefik --watch
```

範例輸出會顯示 *traefik* 命名空間中所有服務的 IP 位址。

```console
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
traefik   LoadBalancer   10.0.205.78   <pending>     80:32484/TCP,443:30620/TCP   20s
...
traefik   LoadBalancer   10.0.205.78   MY_EXTERNAL_IP   80:32484/TCP,443:30620/TCP   60s
```

使用 az network DNS record，將 *a* 記錄新增至您的 DNS 區域，並在其中加入 traefik 服務的外部 IP 位址 [-設定新增記錄][az-network-dns-record-set-a-add-record]。

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address MY_EXTERNAL_IP
```

上述範例會將 *A* 記錄新增至 *MY_CUSTOM_DOMAIN* DNS 區域。

在本文中，您會使用 [Azure Dev Spaces 單車共享範例應用程式](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp) \(英文\) 來示範如何使用 Azure Dev Spaces。 從 GitHub 複製應用程式，並瀏覽至其目錄：

```cmd
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/BikeSharingApp/charts
```

開啟 [ [值]。 yaml][values-yaml] 並進行下列更新：
* 將 *<REPLACE_ME_WITH_HOST_SUFFIX>* 的所有實例取代為 *traefik。MY_CUSTOM_DOMAIN* 使用您的網域進行 *MY_CUSTOM_DOMAIN*。 
* 以*kubernetes.io/ingress.class： traefik # 自訂*輸入取代*kubernetes.io/ingress.class： traefik-Azds # Dev Spaces 特有*。 

以下是已更新檔案的範例 `values.yaml` ：

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
    hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
    hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
```

儲存變更並關閉該檔案。

使用，透過您的範例應用程式建立 *開發人員* 空間 `azds space select` 。

```console
azds space select -n dev -y
```

使用部署範例應用程式 `helm install` 。

```console
helm install bikesharingsampleapp . --dependency-update --namespace dev --atomic
```

上述範例會將範例應用程式部署至 *dev* 命名空間。

使用來顯示要存取範例應用程式的 Url `azds list-uris` 。

```console
azds list-uris
```

下列輸出顯示來自的範例 Url `azds list-uris` 。

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/  Available
http://dev.gateway.traefik.MY_CUSTOM_DOMAIN/         Available
```

開啟來自 `azds list-uris` 命令的公用 URL，來瀏覽至 *bikesharingweb* 服務。 在上述範例中，*bikesharingweb* 服務的公用 URL 是 `http://dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/`。

> [!NOTE]
> 如果您看到錯誤頁面，而不是*bikesharingweb*服務，請確認您已在*yaml*檔案**中更新** *kubernetes.io/ingress.class*注釋和主機。

使用 `azds space select` 命令在 *dev* 底下建立子空間，並列出 url 以存取子開發人員空間。

```console
azds space select -n dev/azureuser1 -y
azds list-uris
```

下列輸出顯示來自的範例 Url `azds list-uris` ，以存取 *azureuser1* 子開發人員空間中的範例應用程式。

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/  Available
http://azureuser1.s.dev.gateway.traefik.MY_CUSTOM_DOMAIN/         Available
```

從命令開啟公用 URL，以流覽至*azureuser1*子開發人員空間中的*bikesharingweb*服務 `azds list-uris` 。 在上述範例中， *azureuser1*子開發人員空間中*bikesharingweb*服務的公用 URL 是 `http://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/` 。

## <a name="configure-the-traefik-ingress-controller-to-use-https"></a>將 traefik 輸入控制器設定為使用 HTTPS

將 traefik 輸入控制器設定為使用 HTTPS 時，請使用 [cert 管理員][cert-manager] 來自動化 TLS 憑證的管理。 使用 `helm` 來安裝 *certmanager* 圖表。

```console
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace traefik
kubectl label namespace traefik certmanager.k8s.io/disable-validation=true
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace traefik --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
```

建立檔案 `letsencrypt-clusterissuer.yaml` ，並使用您的電子郵件地址更新 [電子郵件] 欄位。

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: traefik
```

> [!NOTE]
> 若要進行測試，您也可以使用*ClusterIssuer*的[預備伺服器][letsencrypt-staging-issuer]。

使用 `kubectl` 來套用 `letsencrypt-clusterissuer.yaml` 。

```console
kubectl apply -f letsencrypt-clusterissuer.yaml --namespace traefik
```

移除先前的 *traefik* *ClusterRole* 和 *ClusterRoleBinding*，然後將 traefik 升級為使用 HTTPS `helm` 。

> [!NOTE]
> 如果您的 AKS 叢集未啟用 RBAC，請移除 *--set RBAC. enabled = true* 參數。

```console
kubectl delete ClusterRole traefik
kubectl delete ClusterRoleBinding traefik
helm upgrade traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0 --set ssl.enabled=true --set ssl.enforced=true --set ssl.permanentRedirect=true
```

使用 [kubectl get][kubectl-get]取得 traefik 輸入控制器服務的更新 IP 位址。

```console
kubectl get svc -n traefik --watch
```

範例輸出會顯示 *traefik* 命名空間中所有服務的 IP 位址。

```console
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP          PORT(S)                      AGE
traefik   LoadBalancer   10.0.205.78   <pending>            80:32484/TCP,443:30620/TCP   20s
...
traefik   LoadBalancer   10.0.205.78   MY_NEW_EXTERNAL_IP   80:32484/TCP,443:30620/TCP   60s
```

使用[az network dns record][az-network-dns-record-set-a-add-record] ，將*a*記錄新增至您的 DNS 區域 traefik 服務的新外部 IP 位址-設定新增記錄，並使用 az network dns Record 來移除先前*的 A*記錄[-設定移除記錄][az-network-dns-record-set-a-remove-record]。

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address MY_NEW_EXTERNAL_IP

az network dns record-set a remove-record \
    --resource-group myResourceGroup \
    --zone-name  MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address PREVIOUS_EXTERNAL_IP
```

上述範例會更新*MY_CUSTOM_DOMAIN* DNS 區域中的*A*記錄，以使用*PREVIOUS_EXTERNAL_IP*。

更新 [值。 yaml][values-yaml] 以包含使用 *cert-管理員* 和 HTTPS 的詳細資料。 以下是已更新檔案的範例 `values.yaml` ：

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN
      secretName: dev-bikesharingweb-secret

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN
      secretName: dev-gateway-secret
```

使用下列內容升級範例應用程式 `helm` ：

```console
helm upgrade bikesharingsampleapp . --namespace dev --atomic
```

流覽至 *dev/azureuser1* 子空間中的範例應用程式，並注意您會重新導向至使用 HTTPS。

> [!IMPORTANT]
> 可能需要30分鐘或更久的時間，DNS 變更才能完成，且您的範例應用程式可供存取。

另請注意，頁面會載入，但瀏覽器會顯示一些錯誤。 開啟瀏覽器主控台會顯示與嘗試載入 HTTP 資源的 HTTPS 頁面相關的錯誤。 例如：

```console
Mixed Content: The page at 'https://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/devsignin' was loaded over HTTPS, but requested an insecure resource 'http://azureuser1.s.dev.gateway.traefik.MY_CUSTOM_DOMAIN/api/user/allUsers'. This request has been blocked; the content must be served over HTTPS.
```

若要修正此錯誤，請將[BikeSharingWeb/azds][azds-yaml]更新為使用*kubernetes.io/ingress.class*的*traefik* ，並將您的自訂網域用於 *$ (hostSuffix) *。 例如：

```yaml
...
    ingress:
      annotations:
        kubernetes.io/ingress.class: traefik
      hosts:
      # This expands to [space.s.][rootSpace.]bikesharingweb.<random suffix>.<region>.azds.io
      - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.traefik.MY_CUSTOM_DOMAIN
...
```

以*url*套件的相依性更新[BikeSharingWeb/package.json][package-json] 。

```json
{
...
    "react-responsive": "^6.0.1",
    "universal-cookie": "^3.0.7",
    "url": "0.11.0"
  },
...
```

更新[BikeSharingWeb/lib/helpers.js][helpers-js]中的*getApiHostAsync*方法，以使用 HTTPS：

```javascript
...
    getApiHostAsync: async function() {
        const apiRequest = await fetch('/api/host');
        const data = await apiRequest.json();
        
        var urlapi = require('url');
        var url = urlapi.parse(data.apiHost);

        console.log('apiHost: ' + "https://"+url.host);
        return "https://"+url.host;
    },
...
```

流覽至 `BikeSharingWeb` 目錄，並使用 `azds up` 來執行更新的 BikeSharingWeb 服務。

```console
cd ../BikeSharingWeb/
azds up
```

流覽至 *dev/azureuser1* 子空間中的範例應用程式，並注意您會被重新導向至使用 HTTPS，而不會發生任何錯誤。

## <a name="next-steps"></a>後續步驟

深入瞭解 Azure Dev Spaces 的運作方式。

> [!div class="nextstepaction"]
> [Azure Dev Spaces 如何運作](../how-dev-spaces-works.md)


[az-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-add-record
[az-network-dns-record-set-a-remove-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-remove-record
[custom-domain]: ../../app-service/manage-custom-dns-buy-domain.md#buy-the-domain
[dns-zone]: ../../dns/dns-getstarted-cli.md
[azds-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/azds.yaml
[azure-account-create]: https://azure.microsoft.com/free
[cert-manager]: https://cert-manager.io/
[helm-installed]: https://helm.sh/docs/intro/install/
[helm-stable-repo]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helpers-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/lib/helpers.js#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[letsencrypt-staging-issuer]: https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer
[package-json]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/package.json
[values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml