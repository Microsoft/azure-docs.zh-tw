---
title: 使用 Azure Container Registry 中的 Helm 存放庫
description: 了解如何搭配 Azure Container Registry 使用 Helm 來儲存您應用程式的圖表
services: container-registry
author: iainfoulds
ms.service: container-registry
ms.topic: article
ms.date: 09/24/2018
ms.author: iainfou
ms.openlocfilehash: f04b4015e201d272fa624b1de265792564ac14cd
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045918"
---
# <a name="use-azure-container-registry-as-a-helm-repository-for-your-application-charts"></a>使用 Azure Container Registry 作為應用程式圖表的 Helm 存放庫

若要快速管理及部署 Kubernetes 的應用程式，您可以使用[開放原始碼 Helm 套件管理員][helm]。 使用 Helm 時，應用程式會定義成儲存在 Helm 圖表存放庫中的「圖表」。 這些圖表會定義設定和相依性，且在整個應用程式生命週期可進行版本設定。 Azure Container Registry 可用來作為 Helm 圖表存放庫的主機。

使用 Azure Container Registry 時，您會擁有一個私人、安全的 Helm 圖表存放庫，此存放庫可與組建管線或其他 Azure 服務整合。 Azure Container Registry 中的 Helm 圖表存放庫包含異地複寫功能，可讓您的圖表靠近部署並提供備援。 您只需支付圖表所使用儲存體的費用，且所有 Azure Container Registry 價格層都有提供。

此文章說明如何使用儲存在 Azure Container Registry 中的 Helm 圖表存放庫。

> [!IMPORTANT]
> 此功能目前為預覽狀態。 若您同意[補充的使用規定][terms-of-use]，即可取得預覽。 在公開上市 (GA) 之前，此功能的某些領域可能會變更。

## <a name="before-you-begin"></a>開始之前

若要完成此文章中的步驟，必須符合下列先決條件︰

- **Azure Container Registry** - 在您的 Azure 訂用帳戶中建立容器登錄。 例如，使用 [Azure 入口網站](container-registry-get-started-portal.md)或 [Azure CLI](container-registry-get-started-azure-cli.md)。
- **Helm 用戶端** - 用來搜尋及安裝圖表。 此外，您還需要一部在 Kubernetes 叢集內初始化的 Helm 伺服器 (Tiller)。 如有需要，您可以[建立 Azure Kubernetes Service 叢集][aks-quickstart]。 如需有關如何安裝及使用 Helm 的詳細資訊，請參閱[安裝 Helm][helm-install]。
- **Azure CLI 2.0.46 版或更新版本** - 請執行 `az --version` 來找出版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="add-a-repository-to-helm-client"></a>將存放庫新增至 Helm 用戶端

Helm 存放庫是一個可以儲存 Helm 圖表的 HTTP 伺服器。 Azure Container Registry 可為 Helm 圖表提供此儲存體，並在您於存放庫中新增和移除圖表時管理索引定義。

若要新增 Azure Container Registry 作為 Helm 圖表存放庫，您可以使用 Azure CLI。 使用此方法時，會以 Azure Container Registry 所支持存放庫的 URI 和認證更新您的 Helm 用戶端。 您無須手動指定此存放庫資訊，因此舉例來說，認證並不會暴露在命令歷程記錄中。

如有需要，請登入 Azure CLI 並依照提示進行操作：

```azurecli
az login
```

使用 [az configure][az-configure] 命令，以您 Azure Container Registry 的名稱設定 Azure CLI 預設值。 在下列範例中，以您的登錄名稱取代 `<acrName>`：

```azurecli
az configure --defaults acr=<acrName>
```

現在，使用 [az acr helm repo add][az-acr-helm-repo-add] 命令，將您的 Azure Container Registry Helm 圖表存放庫新增至 Helm 用戶端。 此命令會取得 Helm 用戶端所使用 Azure Container Registry 的驗證權杖。 驗證權杖的有效期為 1 小時。 與 `docker login` 類似，您可以在未來的 CLI 工作階段中執行此命令，以向 Azure Container Registry Helm 圖表存放庫驗證 Helm 用戶端：

```azurecli
az acr helm repo add
```

## <a name="add-a-chart-to-the-repository"></a>將圖表新增至存放庫

針對此文章，我們將從公用 Helm *stable* 存放庫取得現有的 Helm 圖表。 *stable* 存放庫是一個經過策劃的公用存放庫，當中包含常見的應用程式圖表。 套件維護人員可以將其圖表提交給 *stable* 存放庫，方式就像 Docker Hub 為常見的容器映像提供公用登錄一樣。 接著，便可將從公用 *stable* 存放庫下載的圖表推送到您的私人 Azure Container Registry 存放庫。 在大多數情況下，您會為您開發的應用程式建置並上傳自己的圖表。 如需有關如何建置您自己 Helm 圖表的詳細資訊，請參閱[開發 Helm 圖表][develop-helm-charts]。

首先，在 *~/acr-helm* 建立一個目錄，然後下載現有的 *stable/wordpress* 圖表：

```console
mkdir ~/acr-helm && cd ~/acr-helm
helm fetch stable/wordpress
```

列出已下載的圖表，並記下檔案名稱中所包含的 Wordpress 版本。 `helm fetch stable/wordpress` 命令並未指定特定的版本，因此擷取的是「最新」版本。 所有 Helm 圖表都會在檔案名稱中包含依循 [SemVer 2][semver2] 標準的版本號碼。 在下列範例輸出中，Wordpress 圖表的版本為 *2.1.10*：

```
$ ls

wordpress-2.1.10.tgz
```

現在，使用 Azure CLI [az acr helm push][az-acr-helm-push] 命令，將圖表推送至 Azure Container Registry 中的 Helm 圖表存放庫。 指定在上一個步驟中所下載 Helm 圖表的名稱，例如 *wordpress-2.1.10.tgz*：

```azurecli
az acr helm push wordpress-2.1.10.tgz
```

片刻之後，Azure CLI 就會回報已儲存您的圖表，如以下範例輸出所示：

```
$ az acr helm push wordpress-2.1.10.tgz

{
  "saved": true
}
```

## <a name="list-charts-in-the-repository"></a>列出存放庫中的圖表

Helm 用戶端會維護一份遠端存放庫內容的本機快取複本。 對遠端存放庫進行變更時，並不會自動更新 Helm 用戶端在本機已知的可用圖表清單。 當您跨存放庫搜尋圖表時，Helm 會使用其本機快取索引。 若要使用在上一個步驟中上傳的圖表，必須更新本機 Helm 存放庫索引。 您可以為 Helm 用戶端中的存放庫重新編製索引，或使用 Azure CLI 來更新存放庫索引。 每次您將圖表新增至存放庫時，都必須完成此步驟：

```azurecli
az acr helm repo add
```

透過將圖表儲存在您的存放庫中，並在本機提供已更新的索引，您便可以使用一般 Helm 用戶端命令來進行搜尋或安裝。 若要查看您存放庫中的所有圖表，請使用 `helm search <acrName>`。 提供您自己的 Azure Container Registry 名稱：

```console
helm search <acrName>
```

這會列出在上一個步驟中推送的 Wordpress 圖表，如以下範例輸出所示：

```
$ helm search myacrhelm

NAME                CHART VERSION   APP VERSION DESCRIPTION
helmdocs/wordpress  2.1.10          4.9.8       Web publishing platform for building blogs and websites.
```

您也可以搭配 Azure CLI 使用 [az acr helm list][az-acr-helm-list]來列出圖表：

```azurecli
az acr helm list
```

## <a name="show-information-for-a-helm-chart"></a>顯示 Helm 圖表的資訊

若要檢視存放庫中特定圖表的資訊，您可以再次使用一般 Helm 用戶端。 若要查看名為 *wordpress*之圖表的資訊，請使用 `helm inspect`。

```console
helm inspect <acrName>/wordpress
```

未提供任何版本號碼時，會使用「最新」版本。 Helm 會傳回您圖表的相關詳細資訊，如以下扼要的範例輸出所示：

```
$ helm inspect myacrhelm/wordpress

appVersion: 4.9.8
description: Web publishing platform for building blogs and websites.
engine: gotpl
home: http://www.wordpress.com/
icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png
keywords:
- wordpress
- cms
- blog
- http
- web
- application
- php
maintainers:
- email: containers@bitnami.com
  name: bitnami-bot
name: wordpress
sources:
- https://github.com/bitnami/bitnami-docker-wordpress
version: 2.1.10
[...]
```

您也可以使用 Azure CLI [az acr helm show][az-acr-helm-show] 命令來顯示圖表的資訊。 同樣地，預設會傳回圖表的「最新」版本。 您可以附加 `--version` 來列出圖表的特定版本，例如 *2.1.10*：

```azurecli
az acr helm show wordpress
```

## <a name="install-a-helm-chart-from-the-repository"></a>從存放庫安裝 Helm 圖表

安裝您存放庫中的 Helm 圖表時，會以先指定存放庫名稱再指定圖表名稱的方式安裝。 使用 Helm 用戶端來安裝 Wordpress 圖表：

```console
helm install <acrName>/wordpress
```

> [!TIP]
> 如果您推送至 Azure Container Registry Helm 圖表存放庫並稍後回到新的 CLI 工作階段，您的本機 Helm 用戶端就會需要已更新的驗證權杖。 若要取得新的驗證權杖，請使用 [az acr helm repo add][az-acr-helm-repo-add] 命令。

下列步驟會在安裝程序期間完成：

- Helm 用戶端會搜尋本機存放庫索引。
- 對應的圖表會下載自 Azure Container Registry 存放庫。
- 此圖表是使用您 Kubernetes 叢集中的 Tiller 來部署的。

下列扼要的範例輸出顯示透過 Helm 圖表部署的 Kubernetes 資源：

```
$ helm install myacrhelm/wordpress

NAME:   irreverent-jaguar
LAST DEPLOYED: Thu Sep 13 21:44:20 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                                          READY  STATUS   RESTARTS  AGE
irreverent-jaguar-wordpress-7ff46d9b8c-b7v6m  0/1    Pending  0         1s
irreverent-jaguar-mariadb-0                   0/1    Pending  0         1s
[...]
```

## <a name="delete-a-helm-chart-from-the-repository"></a>從存放庫中刪除 Helm 圖表

若要從存放庫中刪除圖表，請使用 [az acr helm delete][az-acr-helm-delete] 命令。 指定圖表的名稱 (例如 *wordpress*)，以及要刪除的版本 (例如 *2.1.10*)。

```azurecli
az acr helm delete wordpress --version 2.1.10
```

如果您想要刪除所指定圖表的所有版本，請省略 `--version` 參數。

將會繼續在 `helm search <acrName>` 中傳回圖表。 同樣地，Helm 用戶端並不會自動更新存放庫中的可用圖表清單。 若要更新 Helm 用戶端存放庫索引，請再次使用 [az acr helm repo add][az-acr-helm-repo-add] 命令：

```azurecli
az acr helm repo add
```

## <a name="next-steps"></a>後續步驟

此文章使用了來自公用 *stable* 存放庫的現有 Helm 圖表。 如需有關如何建立及部署 Helm 圖表的詳細資訊，請參閱[開發 Helm 圖表][develop-helm-charts]。

Helm 圖表可以用來作為容器建置程序的一部分。 如需詳細資訊，請參閱[使用 Azure Container Registry 工作][acr-tasks]。

如需有關如何使用及管理 Azure Container Registry 的詳細資訊，請參閱[最佳做法][acr-bestpractices]。

<!-- LINKS - external -->
[helm]: https://helm.sh/
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm
[develop-helm-charts]: https://docs.helm.sh/developing_charts/
[semver2]: https://semver.org/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-quickstart]: ../aks/kubernetes-walkthrough.md
[acr-bestpractices]: container-registry-best-practices.md
[az-configure]: /cli/azure/reference-index#az-configure
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-helm-repo-add]: /cli/azure/acr/helm/repo#az-acr-helm-repo-add
[az-acr-helm-push]: /cli/azure/acr/helm#az-acr-helm-push
[az-acr-helm-list]: /cli/azure/acr/helm#az-acr-helm-list
[az-acr-helm-show]: /cli/azure/acr/helm#az-acr-helm-show
[az-acr-helm-delete]: /cli/azure/acr/helm#az-acr-helm-delete
[acr-tasks]: container-registry-tasks-overview.md
