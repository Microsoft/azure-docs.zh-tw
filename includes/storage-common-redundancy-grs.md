---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/26/2018
ms.author: jeking
ms.custom: include file
ms.openlocfilehash: 922f4d5dd8c71bc9365523b1d539d1b0754fff15
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580934"
---
異地備援儲存體 (GRS) 設計為將您的資料複寫到與主要區域相隔數百英哩遠的次要區域，以在指定年份為物件提供至少 99.99999999999999% (16 9's) 的持久性。 如果您的儲存體帳戶已啟用 GRS，即使發生主要區域因全區中斷或嚴重損壞而無法復原的情況，您的資料仍會是永久性。

如果您選擇 GRS，您有兩個相關的選項可供選擇：

* GRS 會將您的資料複寫到次要區域中的另一個資料中心，但如果 Microsoft 起始從主要到次要區域的容錯移轉，該資料則為唯讀。 
* 讀取權限異地備援儲存體 (RA-GRS) 是以 GRS 為基礎。 RA-GRS 會將您的資料複寫到次要區域中的另一個資料中心，也會為您提供從次要區域讀取的選項。 使用 RA-GRS，不論 Microsoft 是否起始從主要到次要的容錯移轉，您都可以從次要區域讀取。 

對於已啟用 GRS 或 RA-GRS 的儲存體帳戶，會使用本地備援儲存體 (LRS) 先複寫所有資料。 更新會先認可到主要位置，並使用 LRS 進行複寫。 接著會使用 GRS，以非同步的方式將更新複寫到次要區域。 當資料寫入次要位置時，也會使用 LRS 在該位置中複寫。 

主要和次要區域會管理分散在儲存體縮放單位內不同容錯網域和升級網域之間的複本。 儲存體縮放單位是資料中心內的基本複寫單位。 這個層級的複寫是由 LRS 提供；如需詳細資訊，請參閱[本地備援儲存體 (LRS)：適用於 Azure 儲存體的低成本資料備援](../articles/storage/common/storage-redundancy-lrs.md)。

當您決定要使用的複寫選項時，請記住下列幾點：

* 區域備援儲存體 (ZRS) 會利用同步複寫提供高可用性，而且在某些情況下可能是比 GRS 或 RA-GRS 更好的選擇。 如需有關 ZRS 的詳細資訊，請參閱 [ZRS](../articles/storage/common/storage-redundancy-zrs.md)。
* 由於非同步方式複寫會涉及到延遲，在發生地區性災難的情況下，如果資料無法從主要區域復原，則很有可能會遺失尚未複寫到次要地區的變更。
* 根據 GRS，除非 Microsoft 起始容錯移轉至次要區域，否則複本無法使用。 如果 Microsoft 能夠起始容錯移轉到次要區域，您將會在完成容錯移轉之後具有該資料的讀取和寫入權限。 如需詳細資訊，請參閱[災害復原指南](../articles/storage/common/storage-disaster-recovery-guidance.md)。
* 如果應用程式需要從次要區域讀取，請啟用 RA-GRS。