---
title: 包含檔案
description: 包含檔案
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 06/18/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: e451171859efc49753131b145642aec4864db45d
ms.sourcegitcommit: 42a4d0e8fa84609bec0f6c241abe1c20036b9575
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/08/2021
ms.locfileid: "98015655"
---
| 資源 | 基本 | 標準 | Premium |
|---|---|---|---|
| 包含的儲存體<sup>1</sup> (GiB) | 10 | 100 | 500 |
| 儲存體限制 (TiB) | 20| 20 | 20 |
| 映像層大小上限 (GiB) | 200 | 200 | 200 |
| 每分鐘的 ReadOps<sup>2, 3</sup> | 1,000 | 3,000 | 10,000 |
| 每分鐘的 WriteOps<sup>2, 4</sup> | 100 | 500 | 2,000 |
| 下載頻寬 <sup>2</sup> (Mbps) | 30 | 60 | 100 |
| 上傳頻寬 <sup>2</sup> (Mbps) | 10 | 20 | 50 |
| Webhook | 2 | 10 | 500 |
| 異地複寫 | N/A | N/A | [支援][geo-replication] |
| 可用性區域 | N/A | N/A | [預覽][zones] |
| 內容信任 | N/A | N/A | [支援][content-trust] |
| 具有私人端點的私人連結 | N/A | N/A | [支援][plink] |
| &bull; 私人端點 | N/A | N/A | 10 |
| 服務端點 VNet 存取 | N/A | N/A | [預覽][vnet] |
| 客戶管理的金鑰 | N/A | N/A | [支援][cmk] |
| 存放庫範圍的權限 | N/A | N/A | [預覽][token]|
| &bull; 權杖 | N/A | N/A | 20,000 |
| &bull; 範圍對應 | N/A | N/A | 20,000 |
| &bull; 每個範圍對應的存放庫 | N/A | N/A | 500 |


<sup>1</sup> 每一層的每日費率包含的儲存體。 可以使用額外的儲存體，每個 GiB 需要額外的每日費率，以登錄儲存體限制為上限。 如需費率資訊，請參閱 [Azure Container Registry 定價][pricing]。

<sup>2</sup>*ReadOps*、*WriteOps* 和「頻寬」是最小預估值。 Azure Container Registry 致力於改善需要使用時的效能。

<sup>3</sup>[docker pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) 會根據映像中的圖層數目以及資訊清單擷取，來轉譯為多個讀取作業。

<sup>4</sup>[docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) 會根據必須推送的圖層數目，來轉譯為多個寫入作業。 `docker push` 包含 *ReadOps*，以擷取現有映像的資訊清單。

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md
[vnet]: ../articles/container-registry/container-registry-vnet.md
[plink]: ../articles/container-registry/container-registry-private-link.md
[cmk]: ../articles/container-registry/container-registry-customer-managed-keys.md
[token]: ../articles/container-registry/container-registry-repository-scoped-permissions.md
[zones]: ../articles/container-registry/zone-redundancy.md
