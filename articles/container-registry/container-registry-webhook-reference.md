---
title: Azure Container Registry Webhook 結構描述參考
description: Webhook 會向 JSON 承載參考要求 Azure Container Registry。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 12/02/2017
ms.author: marsma
ms.openlocfilehash: f62477a4c68abf1617d9689047913fd820ee5461
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32166002"
---
# <a name="azure-container-registry-webhook-reference"></a>Azure Container Registry Webhook 參考

當某些動作對產生事件的容器登錄執行時，您可以針對產生事件的容器登錄[設定 Webhook](container-registry-webhook.md)。 例如，您可以啟用容器映像 `push` 和 `delete` 作業上觸發的 Webhook。 當 Webhook 觸發時，Azure Container Registry 就會發出 HTTP 或 HTTPS 要求，當中包含對您所指定端點之事件的相關資訊。 然後，您的端點就可以據此處理 webhook 並採取行動。

下列各節詳細說明受支援事件所產生的 webhook 要求之結構描述。 事件區段包含事件類型的承載結構描述、範例要求承載，以及一或多個會觸發 webhook 的範例命令。

如需設定 Azure Container Registry 的 Webhook 相關資訊，請參閱[使用 Azure Container Registry Webhook](container-registry-webhook.md)。

## <a name="webhook-requests"></a>Webhook 要求

### <a name="http-request"></a>HTTP 要求

當您設定 webhook 時，觸發的 Webhook 會使 HTTP `POST` 要求您指定的 URL 端點。

### <a name="http-headers"></a>HTTP 標頭

如果您尚未指定 webhook 的 `Content-Type` 自訂標頭，Webhook 要求就會包含 `application/json` 的 `Content-Type`。

不會有其他標頭新增至超出您可能已針對 webhook 指定的這些自訂標頭。

## <a name="push-event"></a>推送事件

當容器映像推送到存放庫時，會觸發 Webhook。

### <a name="push-event-payload"></a>推送事件裝載

|元素|類型|說明|
|-------------|----------|-----------|
|`id`|字串|Webhook 事件的識別碼。|
|`timestamp`|Datetime|Webhook 事件觸發的時間。|
|`action`|字串|觸發 webhook 事件的動作。|
|[目標](#target)|複雜類型|觸發 webhook 事件的事件目標。|
|[要求](#request)|複雜類型|產生 webhook 事件的要求。|

### <a name="target"></a>目標

|元素|類型|說明|
|------------------|----------|-----------|
|`mediaType`|字串|參考物件的 MIME 類型。|
|`size`|Int32|內容的位元組數目。 與長度欄位相同。|
|`digest`|字串|如 Registry V2 HTTP API 規格所定義的內容摘要。|
|`length`|Int32|內容的位元組數目。 [與大小相同] 欄位。|
|`repository`|字串|存放庫名稱。|
|`tag`|字串|映像標記名稱。|

### <a name="request"></a>要求

|元素|類型|說明|
|------------------|----------|-----------|
|`id`|字串|起始事件之要求的識別碼。|
|`host`|字串|登錄執行個體的外部可存取主機名稱，如內送要求上的 HTTP 主機標頭所指定。|
|`method`|字串|產生事件的要求方法。|
|`useragent`|字串|要求的使用者代理程式標頭。|

### <a name="payload-example-push-event"></a>承載範例：推送事件

```JSON
{
  "id": "cb8c3971-9adc-488b-bdd8-43cbb4974ff5",
  "timestamp": "2017-11-17T16:52:01.343145347Z",
  "action": "push",
  "target": {
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "size": 524,
    "digest": "sha256:80f0d5c8786bb9e621a45ece0db56d11cdc624ad20da9fe62e9d25490f331d7d",
    "length": 524,
    "repository": "hello-world",
    "tag": "v1"
  },
  "request": {
    "id": "3cbb6949-7549-4fa1-86cd-a6d5451dffc7",
    "host": "myregistry.azurecr.io",
    "method": "PUT",
    "useragent": "docker/17.09.0-ce go/go1.8.3 git-commit/afdb6d4 kernel/4.10.0-27-generic os/linux arch/amd64 UpstreamClient(Docker-Client/17.09.0-ce \\(linux\\))"
  }
}
```

會觸發**推送**事件 webhook 的範例 [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/) 命令：

```bash
docker push myregistry.azurecr.io/hello-world:v1
```

## <a name="delete-event"></a>刪除事件

刪除存放庫或資訊清單時所觸發的 Webhook。 刪除標記時不會觸發。

### <a name="delete-event-payload"></a>刪除事件裝載

|元素|類型|說明|
|-------------|----------|-----------|
|`id`|字串|Webhook 事件的識別碼。|
|`timestamp`|Datetime|Webhook 事件觸發的時間。|
|`action`|字串|觸發 webhook 事件的動作。|
|[目標](#delete_target)|複雜類型|觸發 webhook 事件的事件目標。|
|[要求](#delete_request)|複雜類型|產生 webhook 事件的要求。|

### <a name="delete_target"></a> 目標

|元素|類型|說明|
|------------------|----------|-----------|
|`mediaType`|字串|參考物件的 MIME 類型。|
|`digest`|字串|如 Registry V2 HTTP API 規格所定義的內容摘要。|
|`repository`|字串|存放庫名稱。|

### <a name="delete_request"></a> 要求

|元素|類型|說明|
|------------------|----------|-----------|
|`id`|字串|起始事件之要求的識別碼。|
|`host`|字串|登錄執行個體的外部可存取主機名稱，如內送要求上的 HTTP 主機標頭所指定。|
|`method`|字串|產生事件的要求方法。|
|`useragent`|字串|要求的使用者代理程式標頭。|

### <a name="payload-example-delete-event"></a>承載範例：刪除事件

```JSON
{
    "id": "afc359ce-df7f-4e32-bdde-1ff8aa80927b",
    "timestamp": "2017-11-17T16:54:53.657764628Z",
    "action": "delete",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "digest": "sha256:80f0d5c8786bb9e621a45ece0db56d11cdc624ad20da9fe62e9d25490f331d7d",
      "repository": "hello-world"
    },
    "request": {
      "id": "3d78b540-ab61-4f75-807f-7ca9ecf559b3",
      "host": "myregistry.azurecr.io",
      "method": "DELETE",
      "useragent": "python-requests/2.18.4"
    }
  }
```

會觸發**刪除** 事件 webhook 的範例 [Azure CLI 2.0](/cli/azure/acr) 命令：

```azurecli
# Delete repository
az acr repository delete -n MyRegistry --repository MyRepository

# Delete manifest
az acr repository delete -n MyRegistry --repository MyRepository --tag MyTag --manifest
```

## <a name="next-steps"></a>後續步驟

[使用 Azure Container Registry Webhook](container-registry-webhook.md)