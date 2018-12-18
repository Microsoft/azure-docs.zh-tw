---
title: Azure Event Grid 的 Container Registry 事件結構描述
description: 描述利用 Azure Event Grid 提供給 Container Registry 事件的屬性
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.topic: reference
ms.date: 07/20/2018
ms.author: tomfitz
ms.openlocfilehash: 9ed918a7402abcbe79e302421f3b2ac725857464
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39188648"
---
# <a name="azure-event-grid-event-schema-for-container-registry"></a>Container Registry 的 Azure Event Grid 事件結構描述

本文提供 Container Registry 事件的屬性與結構描述。 如需事件結構描述的簡介，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。

## <a name="available-event-types"></a>可用的事件類型

blob 儲存體會發出下列事件類型：

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.ContainerRegistry.ImagePushed | 會在推送映像時引發。 |
| Microsoft.ContainerRegistry.ImageDeleted | 會在刪除映像時引發。 |

## <a name="example-event"></a>事件範例

下列範例顯示映像推送事件的結構描述： 

```json
[{
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "aci-helloworld:v1",
  "eventType": "ImagePushed",
  "eventTime": "2018-04-25T21:39:47.6549614Z",
  "data": {
    "id": "31c51664-e5bd-416a-a5df-e5206bc47ed0",
    "timestamp": "2018-04-25T21:39:47.276585742Z",
    "action": "push",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "size": 3023,
      "digest": "sha256:213bbc182920ab41e18edc2001e06abcca6735d87782d9cef68abd83941cf0e5",
      "length": 3023,
      "repository": "aci-helloworld",
      "tag": "v1"
    },
    "request": {
      "id": "7c66f28b-de19-40a4-821c-6f5f6c0003a4",
      "host": "demo.azurecr.io",
      "method": "PUT",
      "useragent": "docker/18.03.0-ce go/go1.9.4 git-commit/0520e24 os/windows arch/amd64 UpstreamClient(Docker-Client/18.03.0-ce \\\\(windows\\\\))"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

映像刪除事件的結構描述如下：

```json
[{
  "id": "f06e3921-301f-42ec-b368-212f7d5354bd",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "aci-helloworld",
  "eventType": "ImageDeleted",
  "eventTime": "2018-04-26T17:56:01.8211268Z",
  "data": {
    "id": "f06e3921-301f-42ec-b368-212f7d5354bd",
    "timestamp": "2018-04-26T17:56:00.996603117Z",
    "action": "delete",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "digest": "sha256:213bbc182920ab41e18edc2001e06abcca6735d87782d9cef68abd83941cf0e5",
      "repository": "aci-helloworld"
    },
    "request": {
      "id": "aeda5b99-4197-409f-b8a8-ff539edb7de2",
      "host": "demo.azurecr.io",
      "method": "DELETE",
      "useragent": "python-requests/2.18.4"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

## <a name="event-properties"></a>事件屬性

事件具有下列的最高層級資料：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 主題 | 字串 | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| 主旨 | 字串 | 發行者定義事件主體的路徑。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | 字串 | 事件的唯一識別碼。 |
| data | 物件 | blob 儲存體帳戶。 |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| id | 字串 | 事件識別碼。 |
| timestamp | 字串 | 事件發生的時間。 |
| 動作 | 字串 | 包含所提供事件的動作。 |
| 目標 | 物件 | 事件的目標。 |
| 要求 | 物件 | 產生事件的要求。 |
| actor | 物件 | 起始事件的代理程式。 在大部分的情況下，此值可能來自要求的授權內容。 |
| 來源 | 物件 | 產生事件的登錄節點。 也就是說，actor 啟始事件，而 source 產生事件。 |

target 物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| mediaType | 字串 | 參考物件的 MIME 類型。 |
| size | integer | 內容的位元組數目。 與長度欄位相同。 |
| digest | 字串 | 如 Registry V2 HTTP API 規格所定義的內容摘要。 |
| length | integer | 內容的位元組數目。 [與大小相同] 欄位。 |
| repository | 字串 | 存放庫名稱。 |
| url | 字串 | 內容的直接 URL。 |
| tag | 字串 | 標籤名稱。 |

request 物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| id | 字串 | 起始事件之要求的識別碼。 |
| addr | 字串 | 用戶端連線 (用於起始事件) 的 IP 或主機名稱，也可能是連接埠。 在標準 http 要求中，此值是 RemoteAddr。 |
| host | 字串 | 登錄執行個體的外部可存取主機名稱，如內送要求上的 http 主機標頭所指定。 |
| method | 字串 | 產生事件的要求方法。 |
| userAgent | 字串 | 要求的使用者代理程式標頭。 |

actor 物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| name | 字串 | 主旨或使用者名稱，與產生事件的要求內容相關聯。 |

source 物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| addr | 字串 | 產生事件的登錄節點 IP 或主機名稱和連接埠。 一般而言，此值與執行中的連接埠會由 os.Hostname() 進行解析。 |
| instanceID | 字串 | 執行中的應用程式執行個體。 會在每次重新啟動後變更。 |

## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
