---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/25/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: a0a777151216cb70b696da088b8a0d34779ebc39
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47396116"
---
下表說明 Azure 儲存體的預設限制。 *輸入*限制是指傳送至某個儲存體帳戶的所有資料 (要求)。 *輸出*限制是指從某個儲存體帳戶接收的所有資料 (回應)。

| 資源 | 預設限制 |
| --- | --- |
| 每個區域中每個訂用帳戶的儲存體帳戶數目，包括標準和進階帳戶 | 200 |
| 儲存體帳戶容量上限<sup>1</sup> | 500 TiB |
| 每一儲存體帳戶的 Blob 容器、Blob、檔案共用、資料表、佇列、實體或訊息的數目上限 | 沒有限制 |
| 每一儲存體帳戶的要求率上限<sup>1</sup> | 每秒 20,000 個要求 |
| 每一儲存體帳戶的輸入上限<sup>1</sup> (美國地區) | 如果啟用 RA-GRS/GRS，則為 10 Gbps，LRS/ZRS 為 20 Gbps<sup>2</sup> |
| 每一儲存體帳戶的輸出上限<sup>1</sup> (美國地區) | 如果啟用 RA-GRS/GRS，則為 20 Gbps，LRS/ZRS 為 30 Gbps |
| 每一儲存體帳戶的輸入上限<sup>1</sup> (非美國區域) | 如果啟用 RA-GRS/GRS，則為 5 Gbps，LRS/ZRS 為 10 Gbps<sup>2</sup> |
| 每一儲存體帳戶的輸出上限<sup>1</sup> (非美國區域) | 如果啟用 RA-GRS/GRS，則為 10 Gbps，LRS/ZRS 為 15 Gbps |

<sup>1</sup> Azure 儲存體帳戶支援較高的容量、要求率、輸入和要求輸出限制。 如需與提高的限制有關的詳細資訊，請參閱[宣佈較大型、較大規模的儲存體帳戶](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/)。 若要要求提高帳戶限制，請連絡 [Azure 支援](https://azure.microsoft.com/support/faq/)。

<sup>2</sup>[Azure 儲存體複寫](https://docs.microsoft.com/azure/storage/common/storage-redundancy)選項包括：
* **RA-GRS**：讀取權限異地備援儲存體。 如果已啟用 RA-GRS，次要位置的輸出目標會與主要位置的輸出目標完全相同。
* **GRS**：異地備援儲存體。 
* **ZRS**：區域備援儲存體。
* **LRS**：本地備援儲存體。 

