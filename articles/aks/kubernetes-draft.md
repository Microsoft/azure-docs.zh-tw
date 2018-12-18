---
title: 使用 Draft 搭配 AKS 和 Azure Container Registry
description: 使用 Draft 搭配 AKS 和 Azure Container Registry
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 08/15/2018
ms.author: iainfou
ms.openlocfilehash: faffd919e1c57f2dff21c42c2294a06eb82a2c0a
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43092605"
---
# <a name="use-draft-with-azure-kubernetes-service-aks"></a>使用 Draft 搭配 Azure Kubernetes Service (AKS)

Draft 是開放原始碼工具，可協助在 Kubernetes 叢集中封裝和部署應用程式容器，讓您自由地專注於開發週期--集中開發的「內部迴圈」。 Draft 會在程式碼開發期間，但在認可至版本控制之前運作。 當程式碼變更時，您可以使用 Draft 將應用程式快速地重新部署到 Kubernetes。 如需有關 Draft 的詳細資訊，請參閱 [Github 上的 Draft 文件][draft-documentation]。

本文說明如何在 AKS 上搭配使用 Draft 與 Kubernetes 叢集。

## <a name="prerequisites"></a>必要條件

本文中詳述的步驟假設您已建立 AKS 叢集，並建立與叢集的 `kubectl` 連線。 如果您需要這些項目，請參閱 [AKS 快速入門][aks-quickstart]。

您在 Azure Container Registry (ACR) 中需要私人 Docker 登錄。 如需如何建立 ACR 執行個體的步驟，請參閱 [Azure Container Registry 快速入門][acr-quickstart]。

Helm 也必須安裝在 AKS 叢集中。 如需如何安裝和設定 Helm 的詳細資訊，請參閱[使用 Helm 搭配 Azure Kubernetes Service (AKS)][aks-helm]。

最後，您必須安裝 [Docker](https://www.docker.com)。

## <a name="install-draft"></a>安裝 Draft

Draft CLI 是在開發系統上執行的用戶端，可讓您將程式碼部署到 Kubernetes 叢集。 若要在 Mac 上安裝 Draft CLI，請使用 `brew`。 如需其他安裝選項，請參閱 [Draft 安裝指南][draft-documentation] \(英文\)。

> [!NOTE]
> 如果您已安裝 0.12 版之前的 Draft，請先使用 `helm delete --purge draft` 從叢集中刪除 Draft，然後執行 `rm -rf ~/.draft` 以移除本機設定。 如果您在 MacOS 上，則請執行 `brew upgrade draft`。

```console
brew tap azure/draft
brew install draft
```

現在，使用 `draft init` 命令初始化 Draft：

```console
draft init
```

## <a name="configure-draft"></a>設定 Draft

Draft 會在本機建置容器映像，然後從本機登錄 (例如使用 Minikube) 部署映像，或使用您所指定的映像登錄。 本文使用 Azure Container Registry (ACR)，因此，您必須在 AKS 叢集與 ACR 登錄之間建立信任關係，然後設定 Draft 以將容器映像推送至 ACR。

### <a name="create-trust-between-aks-cluster-and-acr"></a>在 AKS 叢集與 ACR 之間建立信任

若要在 AKS 叢集與 ACR 登錄之間建立信任，請授與權限以供 AKS 叢集所使用的 Azure Active Directory 服務主體存取 ACR 登錄。 在下列命令中，提供您自己的 `<resourceGroupName>`，將 `<aksName>` 更換為 AKS 叢集的名稱，並將 `<acrName>` 更換為 ACR 登錄的名稱：

```azurecli
# Get the service principal ID of your AKS cluster
AKS_SP_ID=$(az aks show --resource-group <resourceGroupName> --name <aksName> --query "servicePrincipalProfile.clientId" -o tsv)

# Get the resource ID of your ACR instance
ACR_RESOURCE_ID=$(az acr show --resource-group <resourceGroupName> --name <acrName> --query "id" -o tsv)

# Create a role assignment for your AKS cluster to access the ACR instance
az role assignment create --assignee $AKS_SP_ID --scope $ACR_RESOURCE_ID --role contributor
```

若要深入了解這些用來存取 ACR 的步驟，請參閱[向 ACR 驗證](../container-registry/container-registry-auth-aks.md)。

### <a name="configure-draft-to-push-to-and-deploy-from-acr"></a>設定 Draft 以推送至 ACR 和從 ACR 部署

現在，AKS 和 ACR 之間已有信任關係，請允許從 AKS 叢集使用 ACR。

1. 設定 Draft 設定的「登錄」值。 在下列命令中，將 `<acrName>` 更換為 ACR 登錄的名稱：

    ```console
    draft config set registry <acrName>.azurecr.io
    ```

1. 使用 [az acr login][az-acr-login] 登入 ACR 登錄：

    ```azurecli
    az acr login --name <acrName>
    ```

AKS 與 ACR 之間已建立信任，因此不用密碼或祕密就能對 ACR 登錄進行推送或提取。 驗證會在 Azure Resource Manager 層級使用 Azure Active Directory 來進行。

## <a name="run-an-application"></a>執行應用程式

為了查看 Draft 的運作方式，讓我們從 [Draft 存放庫][draft-repo]部署應用程式範例。 首先，複製存放庫：

```console
git clone https://github.com/Azure/draft
```

變更為 Java 範例目錄：

```console
cd draft/examples/example-java/
```

使用 `draft create` 命令啟動程序。 此命令會建立在 Kubernetes 叢集中用來執行應用程式的成品。 這些項目包含 Dockerfile、Helm 圖表和 draft.toml 檔案 (Draft 組態檔)。

```
$ draft create

--> Draft detected Java (92.205567%)
--> Ready to sail
```

若要在 AKS 叢集中執行應用程式範例，請使用 `draft up` 命令。 此命令會建置 Dockerfile 以建立容器映像，將映像推送至 ACR，最後再安裝 Helm 圖表以在 AKS 中啟動應用程式。

此命令第一次執行時，推送和提取容器映像可能需要一些時間。 在快取基底圖層後，部署應用程式時所花費的時間就會大幅降低。

```
$ draft up

Draft Up Started: 'example-java': 01CMZAR1F4T1TJZ8SWJQ70HCNH
example-java: Building Docker Image: SUCCESS ⚓  (73.0720s)
example-java: Pushing Docker Image: SUCCESS ⚓  (19.5727s)
example-java: Releasing Application: SUCCESS ⚓  (4.6979s)
Inspect the logs with `draft logs 01CMZAR1F4T1TJZ8SWJQ70HCNH`
```

如果您在推送 Docker 映像時遇到問題，請確定您已使用 [az acr login][az-acr-login] 成功登入 ACR 登錄，然後再次嘗試 `draft up` 命令。

## <a name="test-the-application-locally"></a>在本機測試應用程式

若要測試應用程式，請使用 `draft connect` 命令。 此命令會使用 Proxy 對 Kubernetes Pod 建立安全的連線。 完成時，就可以在提供的 URL 存取應用程式。

> [!NOTE]
> 可能需要幾分鐘的時間來下載容器映像，並啟動應用程式。 如果存取應用程式時收到錯誤，請重試連線。

```
$ draft connect

Connect to java:4567 on localhost:49804
[java]: SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
[java]: SLF4J: Defaulting to no-operation (NOP) logger implementation
[java]: SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[java]: == Spark has ignited ...
[java]: >> Listening on 0.0.0.0:4567
```

若要存取應用程式，請使用 `draft connect` 輸出中指定的位址和連接埠 (例如，*http://localhost:49804*) 開啟網頁瀏覽器。 

![使用 Draft 執行的 Java 應用程式範例](media/kubernetes-draft/sample-app.png)

使用 `Control+C` 停止 Proxy 連線。

> [!NOTE]
> 您也可以使用 `draft up --auto-connect` 命令來建置和部署應用程式，然後立即連線到第一個執行的容器。

## <a name="access-the-application-on-the-internet"></a>在網際網路上存取應用程式

上一個步驟針對 AKS 叢集中的應用程式 Pod 建立了 Proxy 連線。 當您在開發並測試應用程式時，您可以讓應用程式在網際網路上供人使用。 若要在網際網路上公開應用程式，您必須建立 [LoadBalancer][kubernetes-service-loadbalancer] \(英文\) 類型的 Kubernetes 服務或建立[輸入控制器][kubernetes-ingress]。 我們要建立 LoadBalancer 服務。

首先，更新 values.yaml Draft 套件，以指定應該建立 LoadBalancer 類型的服務：

```console
vi charts/java/values.yaml
```

找出 service.type 屬性，並將其值從 ClusterIP 更新為 LoadBalancer，如下列扼要範例所示：

```yaml
[...]
service:
  name: java
  type: LoadBalancer
  externalPort: 80
  internalPort: 4567
[...]
```

儲存並關閉檔案，然後使用 `draft up` 重新執行應用程式：

```console
draft up
```

需要幾分鐘的時間，服務才能傳回公用 IP 位址。 若要監視進度，請使用 `kubectl get service` 命令搭配 watch 參數：

```console
kubectl get service --watch
```

一開始，服務的 EXTERNAL-IP 會顯示為 pending：

```
NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
example-java-java   LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
```

當 EXTERNAL-IP address 從 pending 變更為 IP 位址之後，請使用 `Control+C` 來停止 `kubectl` 監看程序：

```
NAME                TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
example-java-java   LoadBalancer   10.0.141.72   52.175.224.118  80:32150/TCP   7m
```

若要查看應用程式，請使用 `curl` 瀏覽至負載平衡器的外部 IP 位址：

```
$ curl 52.175.224.118

Hello World, I'm Java
```

## <a name="iterate-on-the-application"></a>針對應用程式反覆執行

現在，Draft 已設定好，且應用程式正在 Kubernetes 中執行，您已準備好反覆執行程式碼。 每當您想要測試更新的程式碼時，請執行 `draft up` 命令，以更新執行中的應用程式。

在此範例中，更新 Java 應用程式範例以變更顯示文字。 開啟 Hello.java 檔案：

```console
vi src/main/java/helloworld/Hello.java
```

更新輸出文字來顯示「Hello World, I'm Java in AKS!」：

```java
package helloworld;

import static spark.Spark.*;

public class Hello {
    public static void main(String[] args) {
        get("/", (req, res) -> "Hello World, I'm Java in AKS!");
    }
}
```

執行 `draft up` 命令以重新部署應用程式：

```console
$ draft up

Draft Up Started: 'example-java': 01CMZC9RF0TZT7XPWGFCJE15X4
example-java: Building Docker Image: SUCCESS ⚓  (25.0202s)
example-java: Pushing Docker Image: SUCCESS ⚓  (7.1457s)
example-java: Releasing Application: SUCCESS ⚓  (3.5773s)
Inspect the logs with `draft logs 01CMZC9RF0TZT7XPWGFCJE15X4`
```

若要查看更新後的應用程式，請再次使用 curl 瀏覽負載平衡器的 IP 位址：

```
$ curl 52.175.224.118

Hello World, I'm Java in AKS!
```

## <a name="next-steps"></a>後續步驟

如需有關使用 Draft 的詳細資訊，請參閱 GitHub 上的 Draft 文件。

> [!div class="nextstepaction"]
> [Draft 文件][draft-documentation]

<!-- LINKS - external -->
[draft-documentation]: https://github.com/Azure/draft/tree/master/docs
[kubernetes-service-loadbalancer]: https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer
[draft-repo]: https://github.com/Azure/draft

<!-- LINKS - internal -->
[acr-quickstart]: ../container-registry/container-registry-get-started-azure-cli.md
[aks-helm]: ./kubernetes-helm.md
[kubernetes-ingress]: ./ingress.md
[aks-quickstart]: ./kubernetes-walkthrough.md
[az-acr-login]: /cli/azure/acr#az-acr-login
