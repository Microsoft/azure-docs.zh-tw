---
title: Azure Kubernetes Services (AKS) 的服務主體
description: 為 Azure Kubernetes Service (AKS) 中的叢集建立及管理 Azure Active Directory 服務主體
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: get-started-article
ms.date: 09/26/2018
ms.author: iainfou
ms.openlocfilehash: ef3139c4b3f06644b219e177fad0c094ed600fb6
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394585"
---
# <a name="service-principals-with-azure-kubernetes-service-aks"></a>服務主體與 Azure Kubernetes Service (AKS)

為了與 Azure API 互動，AKS 叢集需要 [Azure Active Directory (AD) 服務主體][aad-service-principal]。 需要服務主體才能動態建立及管理其他 Azure 資源，例如 Azure 負載平衡器或容器登錄 (ACR)。

此文章說明如何為您的 AKS 叢集建立及管理服務主體。

## <a name="before-you-begin"></a>開始之前

若要建立 Azure AD 服務主體，您必須有足夠權限向 Azure AD 租用戶註冊應用程式，並將應用程式指派給您訂用帳戶中的角色。 如果您沒有必要的權限，您可能需要要求您的 Azure AD 或訂用帳戶系統管理員指派必要權限，或或要求其預先建立服務主體以供您搭配 AKS 叢集使用。

您也必須安裝並設定 Azure CLI 版本 2.0.46 或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][install-azure-cli]。

## <a name="automatically-create-and-use-a-service-principal"></a>自動建立並使用服務主體

當您在 Azure 中或使用 [az aks create][az-aks-create] 命令來建立 AKS 叢集時，Azure 可以自動產生服務主體。

在下列 Azure CLI 範例中，未指定服務主體。 在此案例中，Azure CLI 會為 AKS 叢集建立服務主體。 若要成功完成此作業，您的 Azure 帳戶必須有建立服務主體的適當權限。

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --generate-ssh-keys
```

## <a name="manually-create-a-service-principal"></a>手動建立服務主體

若要手動使用 Azure CLI 建立服務主體，請使用 [az ad sp create-for-rbac][az-ad-sp-create] 命令。 在下列範例中，`--skip-assignment` 參數會防止指派任何額外的預設指派：

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

輸出類似於下列範例： 記下您自己的 `appId` 與 `password`。 當您在下一節中建立 AKS 叢集時，會使用這些值。

```json
{
  "appId": "559513bd-0c19-4c1a-87cd-851a26afd5fc",
  "displayName": "azure-cli-2018-09-25-21-10-19",
  "name": "http://azure-cli-2018-09-25-21-10-19",
  "password": "e763725a-5eee-40e8-a466-dc88d980f415",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

## <a name="specify-a-service-principal-for-an-aks-cluster"></a>為 AKS 叢集指定服務主體

若要在使用 [az aks create][az-aks-create] 命令建立 AKS 叢集時使用現有的服務主體，請使用 `--service-principal` 與 `--client-secret` 參數以從 [az ad sp create-for-rbac][az-ad-sp-create] 命令的輸出指定 `appId` 與 `password`：

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --service-principal <appId> \
    --client-secret <password>
```

若使用 Azure 入口網站來部署 AKS 叢集，請在 [建立 Kubernetes 叢集] 對話方塊的 [驗證] 頁面選擇 [設定服務主體]。 選取 [一般] 索引標籤，並指定下列值：

- **服務主體用戶端識別碼**是您的 *appId*
- **服務主體用戶端祕密** 是 *password* 值

![瀏覽至 Azure 投票的影像](media/kubernetes-service-principal/portal-configure-service-principal.png)

## <a name="additional-considerations"></a>其他考量

當使用 AKS 與 Azure AD 服務主體時，請記住下列考量。

- Kubernetes 的服務主體是叢集組態的一部分。 不過，請勿使用身分識別來部署叢集。
- 每個服務主體都會與 Azure AD 應用程式相關聯。 Kubernetes 叢集的服務主體可與任何有效的 Azure AD 應用程式名稱相關聯 (例如：*https://www.contoso.org/example*)。 應用程式的 URL 不一定是實際端點。
- 當您指定服務主體**用戶端識別碼**時，請使用 `appId` 的值。
- 在 Kubernetes 叢集中的主要和節點 VM 上，服務主體認證會儲存在 `/etc/kubernetes/azure.json` 檔案中。
- 當您使用 [az aks create][az-aks-create] 命令自動產生服務主體時，服務主體認證會寫入用來執行命令之電腦上的 `~/.azure/aksServicePrincipal.json` 檔案。
- 當您刪除使用 [az aks create][az-aks-create] 建立的叢集時，不會刪除自動建立的服務主體。
    - 若要刪除服務主體，請先使用 [az ad app list][az-ad-app-list] 取得服務主體的識別碼。 下列範例會查詢名為 myAKSCluster 的叢集，然後使用 [az ad app delete][az-ad-app-delete] 來刪除應用程式識別碼。 請以您自己的值取代這些名稱：

        ```azurecli
        az ad app list --query "[?displayName=='myAKSCluster'].{Name:displayName,Id:appId}" --output table
        az ad app delete --id <appId>
        ```

## <a name="next-steps"></a>後續步驟

如需有關 Azure Active Directory 服務主體的詳細資訊，請參閱[應用程式與服務主體物件][service-principal]

<!-- LINKS - internal -->
[aad-service-principal]:../active-directory/develop/app-objects-and-service-principals.md
[acr-intro]: ../container-registry/container-registry-intro.md
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[azure-load-balancer-overview]: ../load-balancer/load-balancer-overview.md
[install-azure-cli]: /cli/azure/install-azure-cli
[service-principal]:../active-directory/develop/app-objects-and-service-principals.md
[user-defined-routes]: ../load-balancer/load-balancer-overview.md
[az-ad-app-list]: /cli/azure/ad/app#az-ad-app-list
[az-ad-app-delete]: /cli/azure/ad/app#az-ad-app-delete
[az-aks-create]: /cli/azure/aks#az-aks-create
