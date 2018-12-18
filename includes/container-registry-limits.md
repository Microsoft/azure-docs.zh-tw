---
title: 包含檔案
description: 包含檔案
services: container-registry
author: mmacy
ms.service: container-registry
ms.topic: include
ms.date: 08/30/2018
ms.author: marsma
ms.custom: include file
ms.openlocfilehash: bddb17a5163333a5aeb86b296a21867f0611d12f
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43303860"
---
| 資源 | 基本 | 標準 | 進階 |
|---|---|---|---|---|
| 儲存體<sup>1</sup> | 10 GiB | 100 GiB| 500 GiB |
| 映像層大小上限 | 20 GiB | 20 GiB | 50 GiB |
| 每分鐘的 ReadOps<sup>2, 3</sup> | 1,000 | 3,000 | 10,000 |
| 每分鐘的 WriteOps<sup>2, 4</sup> | 100 | 500 | 2,000 |
| 下載頻寬 Mbps<sup>2</sup> | 30 | 60 | 100 |
| 上傳頻寬 Mbps<sup>2</sup> | 10 | 20 | 50 |
| Webhook | 2 | 10 | 100 |
| 異地複寫 | N/A | N/A | [支援][geo-replication] |
| 內容信任 (預覽) | N/A | N/A | [支援][content-trust] |

<sup>1</sup> 指定的儲存體限制為每個層級內含的儲存體數量。 對於超過這些限制的影像儲存，您需要依每 GiB 的每日費率另外支付費用。 如需費率資訊，請參閱 [Container Registry 定價][pricing]。

<sup>2</sup> *ReadOps*、*WriteOps* 和「頻寬」是最小預估值。 ACR 致力於改善需要使用時的效能。

<sup>3</sup> [docker pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) 會根據映像中的圖層數目以及資訊清單擷取，來轉譯為多個讀取作業。

<sup>4</sup> [docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) 會根據必須推送的圖層數目，來轉譯為多個寫入作業。 `docker push` 包含 *ReadOps*，以擷取現有映像的資訊清單。

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md