---
title: Azure Container Registry 中的映像儲存體
description: 有關您的 Docker 容器映像如何儲存在 Azure Container Registry 中的詳細資料，包括安全性、備援和容量。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 03/21/2018
ms.author: marsma
ms.openlocfilehash: 65ff60be992440c69e50a084b467a8efbb19574e
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307144"
---
# <a name="container-image-storage-in-azure-container-registry"></a>Azure Container Registry 中的容器映像儲存體

每個[基本、標準和進階](container-registry-skus.md) Azure 容器登錄都可以從進階 Azure 儲存體功能受益，例如適用於映像資料安全性的待用資料加密，以及適用於映像資料保護的異地備援。 下節描述 Azure Container Registry (ACR) 中映像儲存體的功能和限制。

## <a name="encryption-at-rest"></a>待用資料加密

您登錄中的所有容器映像在待用時都已加密。 Azure 會在儲存映像之前自動將它加密，並在您的應用程式或服務提取映像時將它解密。

## <a name="geo-redundant-storage"></a>異地備援儲存體

Azure 使用異地備援儲存體配置來避免您的容器映像遺失。 Azure Container Registry 會自動將您的容器映像複寫到多個地理位置相距甚遠的資料中心，以避免它們在區域性的儲存體失敗時遺失。

## <a name="geo-replication"></a>異地複寫

針對需要更高可用性保證的案例，請考慮使用「進階」登錄的[異地複寫](container-registry-geo-replication.md)功能。 異地複寫有助於避免在「嚴重」的區域性失敗情況 (不僅是儲存體失敗) 中無法存取您的登錄。 異地複寫也提供其他好處，例如可在分散式開發或部署案例中加快推送和提取速度的網路鄰近映像儲存體。

## <a name="image-limits"></a>映像限制

下表說明 Azure 容器登錄的容器映像和儲存體限制。

| 資源 | 限制 |
| -------- | :---- |
| 儲存機制 | 沒有限制 |
| 映像 | 沒有限制 |
| 層次 | 沒有限制 |
| 標記 | 沒有限制|
| 儲存體 | 5 TB |

大量的存放庫和標籤可能會影響您的登錄效能。 定期刪除未使用的存放庫、標籤和映像，將此作為登錄維護例行工作的一部分。 存放庫、映像和標籤等登錄資源一旦刪除就「無法」復原。 如需刪除登錄資源的詳細資訊，請參閱[刪除 Azure Container Registry 中的容器映像](container-registry-delete.md)。

## <a name="storage-cost"></a>儲存成本

如需定價的完整詳細資訊，請參閱 [Azure Container Registry 定價][pricing]。

## <a name="next-steps"></a>後續步驟

如需不同 Azure Container Registry SKU (基本、標準、進階) 的詳細資訊，請參閱 [Azure Container Registry SKU](container-registry-skus.md)。

<!-- IMAGES -->

<!-- LINKS - External -->
[portal]: https://portal.azure.com
[pricing]: http://aka.ms/acr/pricing

<!-- LINKS - Internal -->
