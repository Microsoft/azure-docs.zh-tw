---
title: 刪除 Azure Container Registry 中的映像資源
description: 有關如何藉由刪除容器映像資料來有效地管理登錄大小的詳細資料。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: article
ms.date: 07/27/2018
ms.author: danlep
ms.openlocfilehash: a1644f68465cffa8cce27257bb91100c111af8a1
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857766"
---
# <a name="delete-container-images-in-azure-container-registry"></a>刪除 Azure Container Registry 中的容器映像

若要維護 Azure Container Registry 的大小，您應該定期刪除過時的映像資料。 雖然有些部署至生產環境的容器映像可能需要較長時間的儲存，但其他容器映像通常可以更快速地刪除。 例如，在自動化建置和測試案例中，您的登錄可以快速地填入可能永遠不會部署的映像，並可在完成建置和測試行程後立刻清除。

您可以用數種不同的方式刪除映像資料，因此務必了解每項刪除作業對於儲存體使用量有何影響。 本文會先介紹 Docker 登錄和容器映像的元件，然後說明數種可供刪除映像資料的方法。

## <a name="registry"></a>登錄

容器「登錄」是一項服務，可儲存及散發容器映像。 Docker 中樞是公用 Docker 容器登錄，而 Azure Container Registry 會在 Azure 中提供私人 Docker 容器登錄。

## <a name="repository"></a>存放庫

容器登錄可管理「存放庫」，也就是具有相同名稱但不同標記的容器映像集合。 例如，下列三個映像位於 "acr-helloworld" 存放庫中：

```
acr-helloworld:latest
acr-helloworld:v1
acr-helloworld:v2
```

存放庫名稱也可以包含[命名空間](container-registry-best-practices.md#repository-namespaces)。 命名空間可讓您使用以正斜線分隔的存放庫名稱，將映像分組，例如：

```
marketing/campaign10-18/web:v2
marketing/campaign10-18/api:v3
marketing/campaign10-18/email-sender:v2
product-returns/web-submission:20180604
product-returns/legacy-integrator:20180715
```

## <a name="components-of-an-image"></a>映像的元件

登錄中的容器映像會與一個或多個標記相關聯，具有一或多個層次，並由資訊清單識別。 了解這些元件彼此的關聯性，可協助您判斷用來釋放登錄空間的最佳方法。

### <a name="tag"></a>Tag

映像的「標記」可指定其版本。 存放庫中的單一映像可獲派一或多個標記，也可以「取消標記」。 也就是說，您可以刪除映像的所有標記，然而映像的資料 (其層次) 會保留在登錄中。

存放庫 (或存放庫和命名空間) 以及標記可定義映像的名稱。 您可以在推送或提取作業中指定映像名稱，以推送和提取映像。

在 Azure Container Registry 之類的私人登錄中，映像名稱也會包含登錄主機的完整名稱。 ACR 中映像的登錄主機格式為 acrname.azurecr.io。 例如，上一節 'marketing' 命名空間中第一個映像的完整名稱會是：

```
myregistry.azurecr.io/marketing/campaign10-18/web:v2
```

如需映像標記最佳做法的討論，請參閱 MSDN 上的 [Docker 標記：Docker 映像標記和版本設定的最佳做法][tagging-best-practices]部落格文章。

### <a name="layer"></a>層次

映像是由一或多個「層次」所組成，各自對應到可定義映像的 Dockerfile 中的資料行。 登錄中的映像會共用常見層次，以提高儲存效率。 例如，不同存放庫中的數個映像可能會共用相同的 Alpine Linux 基底層次，但是該層次只有一個複本會儲存在登錄中。

層次共用也可將層次分布最佳化，使其分布於有多個映像共用常見層次的節點。 例如，如果節點上現有的映像包含 Alpine Linux 層次作為其基底，則參考同一層的不同映像在後續提取中不會將層移轉到節點。 相反地，它會參考已存在於節點上的層次。

### <a name="manifest"></a>資訊清單

每個推送至容器登錄的容器映像都會於「資訊清單」相關聯。 在推送映像時，由登錄所產生的資訊清單，可唯一識別該映像並指定其層次。 您可以使用 Azure CLI 命令 [az acr repository show-manifests][az-acr-repository-show-manifests] 列出存放庫的資訊清單：

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName>
```

例如，列出 "acr-helloworld" 存放庫的資訊清單摘要：

```console
$ az acr repository show-manifests --name myregistry --repository acr-helloworld
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp": "2018-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp": "2018-07-12T15:50:53.5372468Z"
  },
  {
    "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
    "tags": [
      "v1"
    ],
    "timestamp": "2018-07-11T21:38:35.9170967Z"
  }
]
```

這裡所討論的資訊清單，與您可以在 Azure 入口網站中或利用 [docker manifest inspect][docker-manifest-inspect] 檢視的映像資訊清單不同。 在下一節中，「資訊清單摘要」是指由推送作業所產生的摘要，而非映像資訊清單中的 config.digest。 您可以依照**資訊清單摘要** (而非 config.digest) 提取和刪除映像。 下圖說明兩種摘要類型。

![Azure 入口網站中的資訊清單摘要和 config.digest][manifest-digest]

### <a name="manifest-digest"></a>資訊清單摘要

資訊清單是由唯一的 SHA-256 雜湊或「資訊清單摘要」識別。 每個映像--不論標記與否--都是由其摘要識別。 即使映像的層次資料與另一個映像的層次資料相同完全，摘要值也會是唯一的。 這個機制可讓您重複將標記相同的映像推送至登錄。 例如，您可以將 `myimage:latest` 重複推送至您的登錄，而不會發生錯誤，因為每個映像都是由其唯一摘要識別。

您可以在推送作業中指定其摘要，以從摘要中提取映像。 有些系統可能會設定為依照摘要提取，因為這可保證所提取的映像版本，即使標記相同的映像隨後推送至登錄亦然。

例如，由資訊清單摘要從 "acr-helloworld" 存放庫提取映像：

```console
$ docker pull myregistry.azurecr.io/acr-helloworld@sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108
```

> [!IMPORTANT]
> 如果您重複推送具有相同標記且已修改的映像，可以建立孤立映像 (也就是已取消標記的映像)，但仍會耗用您登錄中的空間。 當您依照標記列出或檢視映像時，已取消標記的映像不會顯示在 Azure CLI 或 Azure 入口網站中。 不過，其層次仍然存在，而且會耗用您登錄中的空間。 本文的[刪除已取消標記的映像](#delete-untagged-images)一節討論如何釋放已取消標記映像所使用的空間。

## <a name="delete-image-data"></a>刪除映像資料

您可以用數種方式從容器登錄中刪除映像資料：

* 刪除[存放庫](#delete-repository)：刪除存放庫內的所有映像和所有唯一層次。
* 依[標記](#delete-by-tag)刪除：刪除映像、標記、映像參考的所有唯一層次，以及與映像相關聯的所有其他標記。
* 刪除[資訊清單摘要](#delete-by-manifest-digest)：刪除映像、映像參考的所有唯一層次，以及與映像相關聯的所有標記。

## <a name="delete-repository"></a>刪除存放庫

刪除存放庫即可刪除存放庫中的所有映像，包括所有標籤、唯一層次及資訊清單。 當您刪除存放庫時，會復原該存放庫中的映像所使用的儲存空間。

下列 Azure CLI 命令可刪除 "acr-helloworld" 存放庫以及該存放庫內的所有標記和資訊清單。 如果已刪除資訊清單所參考的層次並未由登錄中任何其他映像所參考，則也會刪除其層次資料。

```azurecli
 az acr repository delete --name myregistry --repository acr-helloworld
```

## <a name="delete-by-tag"></a>依標記刪除

您可以在刪除作業中指定存放庫名稱和標記，以從存放庫中刪除個別映像。 當您依標記刪除時，會復原映像中任何唯一層次 (並未由登錄中任何其他映像共用的層次) 所使用的儲存空間。

若要依標記刪除，請使用 [az acr repository delete][az-acr-repository-delete] 並在 `--image` 參數中指定映像名稱。 映像獨有的所有層次，與該映像相關聯的任何其他標記都會遭到刪除。

例如，從 "myregistry" 登錄中刪除 "acr-helloworld:latest" 映像：

```azurecli
$ az acr repository delete --name myregistry --image acr-helloworld:latest
This operation will delete the manifest 'sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108' and all the following images: 'acr-helloworld:latest', 'acr-helloworld:v3'.
Are you sure you want to continue? (y/n): y
```

> [!TIP]
> 「依標記」刪除不應該與刪除標記 (取消標記) 混淆。 您可以使用 Azure CLI 命令 [az acr repository untag][az-acr-repository-untag] 來刪除標記。 當您取消標記映像時，不會釋放任何空間，因為其[資訊清單](#manifest)和層次資料仍保留在登錄中。 只有標記參考本身會被刪除。

## <a name="delete-by-manifest-digest"></a>依資訊清單摘要刪除

[資訊清單摘要](#manifest-digest)可以與一個、零個或多個標記產生關聯。 當您依摘要刪除時，由資訊清單參考的所有標記都會遭到刪除，因為任何層次的層次資料對映像而言都是唯一的。 共用的層次資料不會遭到刪除。

若要依摘要刪除，首先針對包含所要刪除映像的存放庫列出資訊清單摘要。 例如︰

```console
$ az acr repository show-manifests --name myregistry --repository acr-helloworld
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp": "2018-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp": "2018-07-12T15:50:53.5372468Z"
  }
]
```

接下來，在 [az acr repository delete][az-acr-repository-delete] 命令中指定您要刪除的摘要。 此命令的格式為：

```azurecli
az acr repository delete --name <acrName> --image <repositoryName>@<digest>
```

例如，若要刪除前面輸出中所列的最後一個資訊清單 (含標記 "v2")：

```console
$ az acr repository delete --name myregistry --image acr-helloworld@sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57
This operation will delete the manifest 'sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57' and all the following images: 'acr-helloworld:v2'.
Are you sure you want to continue? (y/n): y
```

"acr-helloworld:v2" 映像會從登錄中刪除，因為任何層次資料對該映像而言都是唯一的。 如果資訊清單與多個標記相關聯，也會一併刪除所有相關聯的標記。

## <a name="delete-untagged-images"></a>刪除已取消標記的映像

如[資訊清單摘要](#manifest-digest)一節所述，使用現有標記推送已修改的映像會**取消標記**先前推送的映像，因而產生孤立 (或「懸盪」) 的映像。 先前推送映像的資訊清單以及其層次資料都會保留在登錄中。 考量以下事件順序：

1. 推送具有 **latest** 標記的映像 *acr-helloworld*：`docker push myregistry.azurecr.io/acr-helloworld:latest`
1. 檢查 *acr-helloworld* 存放庫的資訊清單：

   ```console
   $ az acr repository show-manifests --name myregistry --repository acr-helloworld
   [
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

1. 修改 *acr-helloworld* Dockerfile
1. 推送具有 **latest** 標記的映像 *acr-helloworld*：`docker push myregistry.azurecr.io/acr-helloworld:latest`
1. 檢查 *acr-helloworld* 存放庫的資訊清單：

   ```console
   $ az acr repository show-manifests --name myregistry --repository acr-helloworld
   [
     {
       "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:38:35.9170967Z"
     },
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": null,
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

如您在序列中最後一個步驟的輸出中所見，現在有一個 `"tags"` 屬性為 `null` 的孤立資訊清單。 此資訊清單以及它所參考的任何唯一層次資料，仍存在於登錄中。 **若要刪除這類孤立映像及其層次資料，您必須依資訊清單摘要刪除**。

### <a name="list-untagged-images"></a>列出已取消標記的映像

您可以使用下列 Azure CLI 命令，列出存放庫中所有已取消標記的映像。 以適合您環境的值取代 `<acrName>` 和 `<repositoryName>`。

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName>  --query "[?tags==null].digest"
```

### <a name="delete-all-untagged-images"></a>刪除所有已取消標記的映像

請小心使用下列範例指令碼--無法復原已刪除的映像資料。

**Bash 中的 Azure CLI**

下列 Bash 指令碼會從存放庫中刪除所有已取消標記的映像。 它需要 Azure CLI 和 **xargs**。 根據預設，此指令碼不會執行任何刪除。 將 `ENABLE_DELETE` 值變更為 `true`，以啟用映像刪除。

> [!WARNING]
> 如果您有系統依照資訊清單摘要 (相對於映像名稱) 提取映像，則不應該執行此指令碼。 刪除已取消標記的映像，會導致這些系統無法從登錄中提取映像。 請考慮採用「唯一標記」配置(這是[建議的最佳做法][tagging-best-practices])，而不是依照資訊清單提取。

```bash
#!/bin/bash

# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to 'true' to enable image delete
ENABLE_DELETE=false

# Modify for your environment
REGISTRY=myregistry
REPOSITORY=myrepository

# Delete all untagged (orphaned) images
if [ "$ENABLE_DELETE" = true ]
then
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY  --query "[?tags==null].digest" -o tsv \
    | xargs -I% az acr repository delete --name $REGISTRY --image $REPOSITORY@% --yes
else
    echo "No data deleted. Set ENABLE_DELETE=true to enable image deletion."
fi
```

**PowerShell 中的 Azure CLI**

下列 PowerShell 指令碼會從存放庫中刪除所有已取消標記的映像。 它需要 PowerShell 和 Azure CLI。 根據預設，此指令碼不會執行任何刪除。 將 `$enableDelete` 值變更為 `$TRUE`，以啟用映像刪除。

> [!WARNING]
> 如果您有系統依照資訊清單摘要 (相對於映像名稱) 提取映像，則不應該執行此指令碼。 刪除已取消標記的映像，會導致這些系統無法從登錄中提取映像。 請考慮採用「唯一標記」配置(這是[建議的最佳做法][tagging-best-practices])，而不是依照資訊清單提取。

```powershell
# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to '$TRUE' to enable image delete
$enableDelete = $FALSE

# Modify for your environment
$registry = "myregistry"
$repository = "myrepository"

if ($enableDelete) {
    az acr repository show-manifests --name $registry --repository $repository --query "[?tags==null].digest" -o tsv `
    | %{ az acr repository delete --name $registry --image $repository@$_ --yes }
} else {
    Write-Host "No data deleted. Set `$enableDelete = `$TRUE to enable image deletion."
}
```

## <a name="next-steps"></a>後續步驟

如需 Azure Container Registry 中映像儲存體的詳細資訊，請參閱 [Azure Container Registry 中的容器映像儲存體](container-registry-storage.md)。

<!-- IMAGES -->
[manifest-digest]: ./media/container-registry-delete/01-manifest-digest.png

<!-- LINKS - External -->
[docker-manifest-inspect]: https://docs.docker.com/edge/engine/reference/commandline/manifest/#manifest-inspect
[portal]: https://portal.azure.com
[tagging-best-practices]: https://blogs.msdn.microsoft.com/stevelasker/2018/03/01/docker-tagging-best-practices-for-tagging-and-versioning-docker-images/

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[az-acr-repository-untag]: /cli/azure/acr/repository#az-acr-repository-untag
