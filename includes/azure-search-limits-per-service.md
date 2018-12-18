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
ms.openlocfilehash: b4062aab5a453505ef4586f422a124d4bbf715cb
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38755682"
---
儲存體受限於磁碟空間，或者索引、文件或其他高層級資源的「數目上限」，取決於何者較早出現。 下表記載儲存體限制。 如需了解索引、文件和其他物件的限制上限，請參閱[依資源而定的限制](../articles/search/search-limits-quotas-capacity.md#index-limits)。

| 資源 | 免費 | 基本&nbsp;<sup>1</sup> | S1 | S2 | S3 | S3&nbsp;HD&nbsp;<sup>2</sup> |
| -------- | --- | --- | --- | --- | --- | --- |
| 服務等級協定 (SLA) <sup>3</sup>  |否 |yes |yes |yes |yes |是 |
| 每個資料分割的儲存體 |50 MB |2 GB |25 GB |100 GB |200 GB |200 GB |
| 每個服務的分割區 |N/A |1 |12 |12 |12 |3 |
| 分割區大小 |N/A |2 GB |25 GB |100 GB |200 GB |200 GB |
| 複本數 |N/A |3 |12 |12 |12 |12 |

<sup>1</sup> [基本] 具有一個固定的磁碟分割。 在這一層，會使用額外的 SU 為增加的查詢工作負載配置更多複本。

<sup>2</sup> S3 HD 上有 3 個分割區的硬式限制，低於 S3 的分割區限制。 較低的分割區限制是因為 S3 HD 的索引計數較高。 假設計算資源 (儲存體和處理) 和內容 (索引和文件) 的服務限制都存在，會先達到內容限制。


  <sup>3</sup> 專用資源上的可計費服務會提供服務等級協定 (SLA)。 免費服務和預覽功能不提供 SLA。 對於可計費服務，SLA 會在您為您的服務佈建足夠的備援性時生效。 查詢 (讀取) SLA 時需要兩個 (含) 以上的複本。 查詢和檢索 (讀寫) SLA 時需要三個 (含) 以上的複本。 分割區數目不是 SLA 考量。 