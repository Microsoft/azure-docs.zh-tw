---
title: 在 Azure 容器執行個體中更新容器
description: 了解如何在 Azure 容器執行個體的容器群組中更新執行中的容器。
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 08/01/2018
ms.author: marsma
ms.openlocfilehash: 5a42b0983b0f754b119fa304317e758a976fb4f6
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432611"
---
# <a name="update-containers-in-azure-container-instances"></a>在 Azure 容器執行個體中更新容器

在容器執行個體正常運作期間，您可能發現必須更新容器群組中的容器。 例如，您可能想要更新映像版本、變更 DNS 名稱、更新環境變數，或重新整理其應用程式已毀損的容器狀態。

## <a name="update-a-container-group"></a>更新容器群組

使用至少一個已修改的屬性來重新部署現有的群組，以更新容器群組中的容器。 當您更新容器群組時，群組中的所有執行中容器會就地重新啟動。

發出 create 命令 (或使用 Azure 入口網站) 來重新部署現有的容器群組，並指定現有群組的名稱。 當您在發出 create 命令來觸發重新部署時，請修改群組的至少一個有效屬性。 並非所有的容器群組屬性都適用於重新部署。 請參閱[需要刪除的屬性](#properties-that-require-delete)，以取得不支援的屬性清單。

下列 Azure CLI 範例會使用新的 DNS 名稱標籤建立一個容器群組。 因為修改了群組的 DNS 名稱標籤屬性，所以會重新部署容器群組，而且其容器會重新啟動。

使用 DNS 名稱標籤 *myapplication-staging* 進行初始部署：

```azurecli-interactive
# Create container group
az container create --resource-group myResourceGroup --name mycontainer \
    --image nginx:alpine --dns-name-label myapplication-staging
```

使用新的 DNS 名稱標籤 *myapplication*，更新容器群組：

```azurecli-interactive
# Update container group (restarts container)
az container create --resource-group myResourceGroup --name mycontainer \
    --image nginx:alpine --dns-name-label myapplication
```

## <a name="update-benefits"></a>更新優點

更新現有容器群組的主要優點是加快部署。 當您重新部署現有的容器群組時，會從先前部署所快取的映像層中提取其容器映像層。 只會提取已修改的映像層 (如果有的話)，而不會像新的部署一樣，從登錄中全新提取所有的映像層。

當您更新 (而非刪除或部署新的) 應用程式時，可以看到以較大容器映像為基礎的應用程式 (例如 Windows Server Core) 部署速度大幅提升。

## <a name="limitations"></a>限制

並非容器群組的所有屬性都支援更新。 若要變更容器群組的某些屬性，您必須先刪除該群組，然後予以重新部署。 如需詳細資訊，請參閱[需要刪除容器的屬性](#properties-that-require-container-delete)。

當您更新容器群組時，容器群組中的所有容器都會重新啟動。 您無法在多容器群組中執行特定容器的更新或就地重新啟動。

在更新之間，容器的 IP 位址通常不會變更，但不保證它維持不變。 只要容器群組部署至相同的基礎主機，容器群組就會保留其 IP 位址。 雖然罕見，但是當 Azure 容器執行個體會盡全力重新部署到相同的主機時，有一些 Azure 內部事件導致重新部署到不同的主機。 若要解決這個問題，一律對您的容器執行個體使用一個 DNS 名稱標籤。

無法更新已終止或已刪除的容器群組。 容器群組已停止 (處於「已終止」狀態) 或已遭到刪除後，群組就會部署為新的。

## <a name="properties-that-require-container-delete"></a>需要刪除容器的屬性

如先前所述，並非所有容器群組屬性都可以更新。 例如，若要變更連接埠或重新啟動容器的原則，您必須先刪除容器群組，然後予以重新建立。

在重新部署之前，這些屬性需要刪除容器群組：

* OS 類型
* CPU
* 記憶體
* 重新啟動原則
* 連接埠

當您刪除容器群組並加以重新建立時，它並非「重新部署」，而是新建立的。 所有映像層都會從登錄中全新提取，而不是從先前部署所快取的映像層提取。 容器的 IP 位址也可能會因為部署到不同的基礎主機而變更。

## <a name="next-steps"></a>後續步驟

本文中多次提及**容器群組**。 Azure 容器執行個體中的每個容器都會部署在容器群組中，而容器群組可以包含多個容器。

[Azure Container Instances 中的容器群組](container-instances-container-groups.md)

[部署多個容器群組](container-instances-multi-container-group.md)

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container?view=azure-cli-latest#az-container-create
[az-container-logs]: /cli/azure/container?view=azure-cli-latest#az-container-logs
[az-container-show]: /cli/azure/container?view=azure-cli-latest#az-container-show
[azure-cli-install]: /cli/azure/install-azure-cli
