---
title: 包含檔案
description: 包含檔案
services: search
author: HeidiSteen
ms.service: search
ms.topic: include
ms.date: 04/04/2018
ms.author: heidist
ms.custom: include file
ms.openlocfilehash: 8a4f794c8ef24a90498954629c131904621c5b43
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38755658"
---
您可以在訂用帳戶內建立多個服務，每個服務都會佈建於特定層上，僅受限於每一層所允許的服務個數。 例如，您最多可在基本層建立 12 個服務，並在同一個訂用帳戶內的 S1 層另外建立 12 個服務。 如需各層的詳細資訊，請參閱[選擇 Azure 搜尋服務的 SKU 或階層](../articles/search/search-sku-tier.md)。

最大服務限制可以視要求引發。 如果您需要相同訂用帳戶內的更多服務，請連絡 Azure 支援服務。

| 資源            | 免費&nbsp;<sup>1</sup> | 基本 | S1  | S2 | S3 | S3&nbsp;HD |
| ------------------- | ---- | ----- | --- | -- | -- | ----- |
| 服務數目上限    |1     | 12    | 12  | 6  | 6  | 6     |
| SU 中的最大調整規模&nbsp;<sup>2</sup> |N/A |3 SU |36 SU |36 SU |36 SU |36 SU |

<sup>1</sup> 免費服務是根據共用而非專用的資源。 共用資源上不支援相應增加。

<sup>2</sup> 搜尋單位 (SU) 是可計費單位，會以*複本*或*資料分割*形式配置。 您需要這兩種資源才能進行儲存、編製索引及查詢作業。 若要深入了解 SU 計算，請參閱[針對查詢和索引工作負載調整資源層級](../articles/search/search-capacity-planning.md)。 