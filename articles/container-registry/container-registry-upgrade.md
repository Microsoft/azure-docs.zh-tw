---
title: 將傳統的 Azure 容器登錄升級
description: 將非受控的傳統容器登錄升級，以利用基本、標準和進階受控容器登錄的擴充功能集。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 08/28/2018
ms.author: marsma
ms.openlocfilehash: 7a377c607639f5c044e689b11380e9778f9c72c8
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189016"
---
# <a name="upgrade-a-classic-container-registry"></a>將傳統的容器登錄升級

有好幾個服務層 ([也就是所謂的 SKU](container-registry-skus.md)) 會提供 Azure Container Registry (ACR)。 初始的 ACR 版本提供了單一 SKU (傳統)，該 SKU 缺少基本、標準和進階 SKU (統稱為「受控」登錄) 固有的數種功能。

傳統 SKU 即將被取代，而且在 2019 年 3 月之後將無法使用。 本文將詳細說明如何將非受控的傳統登錄移轉至其中一個受控 SKU，以便利用其經過強化的功能集。

## <a name="why-upgrade"></a>為何要升級？

傳統登錄 SKU 即將被**取代**，而且從 **2019 年 3 月**起將無法使用。 所有現有的傳統登錄都應該在 2019 年 3 月之前進行升級。

因為傳統非受控登錄的取代計畫和功能有限，所有傳統登錄都應升級為基本、標準或進階受控登錄。 這些較高層級的 SKU 會將登錄更深入整合到 Azure 功能。

受控登錄可提供：

* Azure Active Directory 整合以便能夠[個別登入](container-registry-authentication.md#individual-login-with-azure-ad)
* 映像和標記刪除支援
* [異地複寫](container-registry-geo-replication.md)
* [Webhook](container-registry-webhook.md)

「傳統」登錄依賴您在建立登錄時，Azure 自動佈建在您 Azure 訂用帳戶中的儲存體帳戶。 相比之下，基本、標準和進階 SKU 會以透明的方式處理您的映像儲存體來利用 Azure 的[進階儲存體功能](container-registry-storage.md)。 您自己的訂用帳戶中未建立個別儲存體帳戶。

受控登錄儲存體可提供下列優點：

* 容器映像是進行[待用加密](container-registry-storage.md#encryption-at-rest)。
* 映像是使用[異地備援儲存體](container-registry-storage.md#geo-redundant-storage)所儲存，並確保使用多區域複寫來備份映像。
* 自由地[在 SKU 之間移動](container-registry-skus.md#changing-skus)的能力，讓您在選擇較高層級的 SKU 時獲得較高的輸送量。 運用每個 SKU，ACR 即可在您的需求增加時滿足輸送量需求。
* 適用於登錄和其儲存體的整合安全性模型能提供簡化的權限管理功能。 您只負責管理容器登錄的權限，而不必還要管理不同儲存體帳戶的權限。

如需 ACR 中映像儲存體的其他相關詳細資料，請參閱 [Azure Container Registry 中的容器映像儲存體](container-registry-storage.md)。

## <a name="migration-considerations"></a>移轉考量

當您將傳統登錄變更為受控登錄時，Azure 必須將所有現有容器映像從您訂用帳戶中的 ACR 所建儲存體帳戶，複製至 Azure 所管理的儲存體帳戶。 視登錄大小而定，此程序所需時間從數分鐘到數小時不等。

在轉換過程中，系統會封鎖所有 `docker push` 作業，但 `docker pull` 卻會持續運作。

在轉換過程中，請勿刪除或修改支援傳統登錄之儲存體帳戶的內容。 如果您這麼做，可能會導致容器映像損毀。

移轉完成之後，ACR 就不會再使用訂用帳戶中原本支援傳統登錄的儲存體帳戶。 在確認移轉成功之後，請考慮刪除該儲存體帳戶，以協助將成本降至最低。

>[!IMPORTANT]
> 從傳統升級為其中一個受控 SKU 是**單向程序**。 傳統登錄一旦轉換為基本、標準或進階，就無法還原為傳統。 不過，您可以在容量足供登錄使用的受控 SKU 之間自由地移動。

## <a name="how-to-upgrade"></a>如何升級

您可以透過數種方式將非受控的傳統登錄升級為其中一個受控 SKU。 在下列幾節中，我們會說明使用 [Azure CLI][azure-cli] 和 [Azure 入口網站][azure-portal]的程序。

## <a name="upgrade-in-azure-cli"></a>在 Azure CLI 中升級

若要在 Azure CLI 中將傳統登錄升級，請執行 [az acr update][az-acr-update] 命令，並為登錄指定新的 SKU。 在下列範例中，名為 myclassicregistry 的傳統登錄會升級為進階 SKU：

```azurecli-interactive
az acr update --name myclassicregistry --sku Premium
```

當移轉完成時，您應該會看到類似下面的輸出。 請注意 `sku` 是 "Premium" 且 `storageAccount` 是 "null"，這表示 Azure 現在管理此登錄的映像儲存體。

```JSON
{
  "adminUserEnabled": false,
  "creationDate": "2017-12-12T21:23:29.300547+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry",
  "location": "eastus",
  "loginServer": "myregistry.azurecr.io",
  "name": "myregistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "name": "Premium",
    "tier": "Premium"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

如果您指定之受控登錄 SKU 的最大容量小於傳統登錄的大小，您會收到類似下面的錯誤訊息。

`Cannot update the registry SKU due to reason: Registry size 12936251113 bytes exceeds the quota value 10737418240 bytes for SKU Basic. The suggested SKU is Standard.`

如果您收到類似錯誤，請再次執行 [az acr update][az-acr-update] 命令並指定所建議的 SKU (也就是可容納映像的下一個最高等級 SKU)。

## <a name="upgrade-in-azure-portal"></a>在 Azure 入口網站中升級

當您使用 Azure 入口網站來為傳統登錄升級時，Azure 會自動選取可容納映像的最低等級 SKU。 例如，如果您的登錄在映像中包含 12 GiB，Azure 便會自動選取將傳統登錄轉換為標準 (最大 100 GiB)。

若要使用 Azure 入口網站來為傳統登錄升級，請瀏覽至容器登錄 [概觀]，然後選取 [升級為受控登錄]。

![Azure 入口網站 UI 中的傳統登錄升級按鈕][update-classic-01-upgrade]

選取 [確定] 以確認您要升級為受控登錄。

![Azure 入口網站 UI 中的傳統登錄升級確認][update-classic-02-confirm]

在移轉期間，入口網站會指出登錄的 [佈建狀態] 是 [更新中]。 如先前所述，移轉期間會停用 `docker push` 作業，而且您不得在移轉進行時刪除或更新傳統登錄所使用的儲存體帳戶；如果您這麼做，可能會導致映像損毀。

![Azure 入口網站 UI 中的傳統登錄升級進度][update-classic-03-updating]

當移轉完成時，[佈建狀態] 會指出 [已成功]，然後您就可以再次對登錄進行 `docker push`。

![Azure 入口網站 UI 中的傳統登錄升級完成狀態][update-classic-04-updated]

## <a name="next-steps"></a>後續步驟

在將傳統登錄升級為基本、標準或進階後，Azure 就不會再使用原本支援傳統登錄的儲存體帳戶。 若要降低成本，請考慮刪除儲存體帳戶或帳戶內包含舊容器映像的 Blob 容器。

<!-- IMAGES -->
[update-classic-01-upgrade]: ./media/container-registry-upgrade\update-classic-01-upgrade.png
[update-classic-02-confirm]: ./media/container-registry-upgrade\update-classic-02-confirm.png
[update-classic-03-updating]: ./media/container-registry-upgrade\update-classic-03-updating.png
[update-classic-04-updated]: ./media/container-registry-upgrade\update-classic-04-updated.png

<!-- LINKS - internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[azure-cli]: /cli/azure/install-azure-cli
[azure-portal]: https://portal.azure.com