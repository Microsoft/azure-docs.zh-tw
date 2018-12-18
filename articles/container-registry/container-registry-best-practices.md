---
title: Azure Container Registry 中的最佳做法
description: 了解如何依照這些最佳做法來有效地使用 Azure Container Registry。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: quickstart
ms.date: 04/10/2018
ms.author: marsma
ms.openlocfilehash: a3932ff621782b8ab97f27ef052aeee8e1d2a3ac
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423499"
---
# <a name="best-practices-for-azure-container-registry"></a>Azure Container Registry 的最佳做法

藉由依照這些最佳做法進行操作，您將可協助發揮最大效能，以及以符合成本效益的方式在 Azure 中使用您的私人 Docker 登錄。

## <a name="network-close-deployment"></a>網路鄰近部署

在您部署容器的相同 Azure 區域中建立容器登錄。 將登錄放在於網路上鄰近您容器主機的區域有助於降低延遲和成本。

網路鄰近部署是使用私人容器登錄的主要原因之一。 Docker 映像擁有一個有效的[分層建構](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)，可允許您進行增量部署。 不過，新節點必須提取指定之映象所需的所有層級。 這個初始的 `docker pull` 可快速加總達數 GB。 讓私人登錄鄰近您的部署可將網路延遲降到最低。
此外，所有公用雲端 (包括 Azure) 都實作網路輸出費用。 將映像從一個資料中心提取到另一個資料中心除了增添延遲之外，也會增添網路輸出費用。

## <a name="geo-replicate-multi-region-deployments"></a>異地複寫多區域部署

如果您要將容器部署到多個區域，請使用 Azure Container Registry 的[異地複寫](container-registry-geo-replication.md)功能。 不論您是要從區域資料中心為全球客戶提供服務，還是您的開發小組位於不同位置，您都可以藉由異地複寫登錄來簡化登錄管理並將延遲降到最低。 只有[進階](container-registry-skus.md)註冊才能使用異地複寫功能。

若要了解如何使用異地複寫，請參閱由三部分組成的教學課程 [Azure Container Registry 中的異地複寫](container-registry-tutorial-prepare-registry.md)。

## <a name="repository-namespaces"></a>儲存機制命名空間

您可以利用儲存機制命名空間，來允許在組織內的多個群組之間共用單一登錄。 登錄可以在各個部署與小組之間共用。 Azure Container Registry 支援巢狀命名空間，可讓您進行群組隔離。

例如，請思考一下下列容器映像標記。 全公司使用的映象 (例如 `aspnetcore`) 會放在根命名空間中，而「生產」及「行銷」群組所擁有的容器映像則會各自使用自己的命名空間。

```
contoso.azurecr.io/aspnetcore:2.0
contoso.azurecr.io/products/widget/web:1
contoso.azurecr.io/products/bettermousetrap/refundapi:12.3
contoso.azurecr.io/marketing/2017-fall/concertpromotions/campaign:218.42
```

## <a name="dedicated-resource-group"></a>專用資源群組

由於容器登錄是跨多個容器主機使用的資源，因此登錄應該位於自己的資源群組中。

雖然您可以對特定主機類型 (例如「Azure 容器執行個體」) 進行實驗，但完成後，您可能會想要刪除該容器執行個體。 不過，您也可能會想要保留已推送到 Azure Container Registry 的映像集合。 藉由將登錄放在它自己的資源群組中，即可將刪除容器執行個體資源群組時意外刪除該登錄中映像集合的風險降到最低。

## <a name="authentication"></a>驗證

向 Azure Container Registry 進行驗證時，有兩種主要的案例：個人驗證和服務 (或「遠端控制」) 驗證。 下表提供這些案例的簡要概觀，以及各個案例的建議驗證方法。

| 類型 | 範例案例 | 建議的方法 |
|---|---|---|
| 個人身分識別 | 將映像提取至其開發電腦或從該電腦提取映像的開發人員。 | [az acr login](/cli/azure/acr?view=azure-cli-latest#az-acr-login) |
| 遠端控制/服務身分識別 | 未直接涉及使用者的組建和部署管線。 | [服務主體](container-registry-authentication.md#service-principal) |

如需有關 Azure Container Registry 驗證的深入資訊，請參閱[向 Azure Container Registry 進行驗證](container-registry-authentication.md)。

## <a name="manage-registry-size"></a>管理登錄大小

每個[容器登錄 SKU][container-registry-skus] 的儲存體條件約束都要符合典型案例：「基本」可用於開始使用、「標準」可用於大部分的實際執行應用程式，而「進階」可進行大規模效能和[異地複寫][container-registry-geo-replication]。 在整個登錄生命週期，您應該定期刪除未使用的內容來管理其大小。

您可以在 Azure 入口網站的容器登錄 [概觀] 中找到登錄的目前使用量：

![Azure 入口網站中的登錄使用量資訊][registry-overview-quotas]

您可以使用 [Azure CLI][azure-cli] 或 [Azure 入口網站][azure-portal]來管理登錄大小。 只有受控 SKU (基本、標準、進階) 才支援存放庫和映像刪除 -- 您無法刪除傳統登錄中的存放庫、映像或標記。

### <a name="delete-in-azure-cli"></a>在 Azure CLI 中刪除

使用 [az acr repository delete][az-acr-repository-delete] 命令可刪除存放庫，或存放庫中的內容。

若要刪除存放庫，包括存放庫內的所有標記和映像圖層資料，當您執行 [az acr repository delete][az-acr-repository-delete] 時，僅指定存放庫名稱。 在下列範例中，要刪除 *myapplication* 存放庫，以及存放庫內的所有標記和映像圖層資料：

```azurecli
az acr repository delete --name myregistry --repository myapplication
```

您也可以使用 `--tag` 和 `--manifest` 引數，從存放庫刪除映像資料。 如需這些引數的詳細資訊，請參閱 [az acr repository delete 命令參考][az-acr-repository-delete]。

### <a name="delete-in-azure-portal"></a>在 Azure 入口網站中刪除

若要從 Azure 入口網站中的登錄刪除存放庫，請先瀏覽至容器登錄。 接著，在 [服務] 下方，選取 [存放庫]，並以滑鼠右鍵按一下您要刪除的存放庫。 選取 [刪除] 可將存放庫和它所包含的 Docker 映像刪除。

![在 Azure 入口網站中刪除存放庫][delete-repository-portal]

您也可以類似的方式，從存放庫刪除標記。 瀏覽至存放庫，在 [標記] 下以滑鼠右鍵按一下您要刪除的標記，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在數個個別提供不同功能的層 (稱為 SKU) 中都有提供 Azure Container Registry。 如需有關可用 SKU 的詳細資料，請參閱 [Azure Container Registry SKU](container-registry-skus.md)。

<!-- IMAGES -->
[delete-repository-portal]: ./media/container-registry-best-practices/delete-repository-portal.png
[registry-overview-quotas]: ./media/container-registry-best-practices/registry-overview-quotas.png

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[azure-cli]: /cli/azure
[azure-portal]: https://portal.azure.com
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-skus]: container-registry-skus.md
