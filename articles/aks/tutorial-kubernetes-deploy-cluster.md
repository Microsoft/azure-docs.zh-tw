---
title: Azure 上的 Kubernetes 教學課程 - 部署叢集
description: 在本 Azure Kubernetes Service (AKS) 教學課程中，您會建立 AKS 叢集，並使用 kubectl 連線至 Kubernetes 主要節點。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 08/14/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 80b011f9df389098095f58c02008da891b2aa8a7
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41918187"
---
# <a name="tutorial-deploy-an-azure-kubernetes-service-aks-cluster"></a>教學課程：部署 Azure Kubernetes Service (AKS) 叢集

Kubernetes 會提供容器化應用程式的分散式平台。 透過 AKS，您可以快速地佈建生產環境就緒的 Kubernetes 叢集。 在本教學課程 (3/7 部分) 中，將 Kubernetes 叢集部署在 AKS 中。 您會了解如何：

> [!div class="checklist"]
> * 建立資源互動的服務主體
> * 部署 Kubernetes AKS 叢集
> * 安裝 Kubernetes CLI (kubectl)
> * 設定 kubectl 以連線至您的 AKS 叢集

在後續的教學課程中，Azure Vote 應用程式會部署至叢集、相應放大並更新。

## <a name="before-you-begin"></a>開始之前

在先前的教學課程中，已建立容器映像並上傳到 Azure Container Registry 執行個體。 如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像][aks-tutorial-prepare-app]。

在進行本教學課程時，您必須執行 Azure CLI 2.0.44 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="create-a-service-principal"></a>建立服務主體

為了允許 AKS 叢集與其他 Azure 資源互動，則會使用 Azure Active Directory 服務主體。 此服務主體可由 Azure CLI 或入口網站自動建立，或者您可以預先建立一個並指派其他權限。 在本教學課程中，您會建立服務主體、授與在前一個教學課程中建立的 Azure Container Registry (ACR) 執行個體存取權，然後建立 AKS 叢集。

使用 [az ad sp create-for-rbac][] 命令建立服務主體。 `--skip-assignment` 參數會限制指派任何其他權限。

```azurecli
az ad sp create-for-rbac --skip-assignment
```

輸出類似於下列範例：

```
{
  "appId": "e7596ae3-6864-4cb8-94fc-20164b1588a9",
  "displayName": "azure-cli-2018-06-29-19-14-37",
  "name": "http://azure-cli-2018-06-29-19-14-37",
  "password": "52c95f25-bd1e-4314-bd31-d8112b293521",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

記下 appId 和密碼。 下列步驟中會使用這些值。

## <a name="configure-acr-authentication"></a>設定 ACR 驗證

若要存取儲存在 ACR 中的映像，您必須授與 AKS 服務主體從 ACR 提取映像的正確權限。

首先，使用 [az acr show][] 取得 ACR 資源識別碼。 將 `<acrName>` 登錄名稱更新為您 ACR 執行個體的登錄名稱，以及將資源群組更新為 ACR 執行個體所在的資源群組。

```azurecli
az acr show --resource-group myResourceGroup --name <acrName> --query "id" --output tsv
```

若要授與 AKS 叢集使用 ACR 中所儲存映像的正確存取權，請使用 [az role assignment create][] 建立角色指派。 以在前兩個步驟中蒐集的值取代 `<appId` 和 `<acrId>`。

```azurecli
az role assignment create --assignee <appId> --scope <acrId> --role Reader
```

## <a name="create-a-kubernetes-cluster"></a>建立 Kubernetes 叢集

AKS 叢集可使用 Kubernetes 角色型存取控制 (RBAC)。 這些控制可讓您根據指派給使用者的角色，來定義資源的存取權。 如果使用者被指派多個角色，則可以合併權限，且權限的範圍可設定為單一命名空間或整個叢集。 AKS 叢集的 Kubernetes RBAC 目前為預覽狀態。 根據預設，Azure CLI 會在您建立 AKS 叢集時自動啟用 RBAC。

使用 [az aks create][] 建立 AKS 叢集。 下列範例會在名為 myResourceGroup 的資源群組中建立名為 myAKSCluster 的叢集。 我們已在[先前的教學課程][aks-tutorial-prepare-acr]中建立此資源群組。 提供您自己的 `<appId>` 和 `<password>` (來自於先前建立服務主體的步驟中)。

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --service-principal <appId> \
    --client-secret <password> \
    --generate-ssh-keys
```

幾分鐘之後，部署就會完成，並以 JSON 格式傳回 AKS 部署的相關資訊。

## <a name="install-the-kubernetes-cli"></a>安裝 Kubernetes CLI

若要從本機電腦連線至 Kubernetes 叢集，您應使用 [kubectl][kubectl] (Kubernetes 命令列用戶端)。

如果您使用 Azure Cloud Shell，則 `kubectl` 已安裝。 您也可以使用 [az aks install-cli][] 命令將其安裝於本機：

```azurecli
az aks install-cli
```

## <a name="connect-to-cluster-using-kubectl"></a>使用 kubectl 連線至叢集

若要設定 `kubectl` 以連線至您的 Kubernetes 叢集，請使用 [az aks get-credentials][]。 下列範例會針對 myResourceGroup 中的 AKS 叢集名稱 myAKSCluster 取得認證：

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

若要確認與叢集的連線，請執行 [kubectl get nodes][kubectl-get] 命令：

```
$ kubectl get nodes

NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-66427764-0   Ready     agent     9m        v1.9.9
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，Kubernetes 叢集已部署在 AKS 中，且您已設定 `kubectl` 加以連線。 您已了解如何︰

> [!div class="checklist"]
> * 建立資源互動的服務主體
> * 部署 Kubernetes AKS 叢集
> * 安裝 Kubernetes CLI (kubectl)
> * 設定 kubectl 以連線至您的 AKS 叢集

繼續進行下一個教學課程，以了解如何將應用程式部署至叢集。

> [!div class="nextstepaction"]
> [在 Kubernetes 中部署應用程式][aks-tutorial-deploy-app]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-tutorial-deploy-app]: ./tutorial-kubernetes-deploy-application.md
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az ad sp create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az acr show]: /cli/azure/acr#az-acr-show
[az role assignment create]: /cli/azure/role/assignment#az-role-assignment-create
[az aks create]: /cli/azure/aks#az-aks-create
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-cli-install]: /cli/azure/install-azure-cli
