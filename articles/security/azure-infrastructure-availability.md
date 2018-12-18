---
title: Azure 基礎結構可用性
description: 本文描述備援層級，以提供客戶資料的最大可用性。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2018
ms.author: terrylan
ms.openlocfilehash: 3679361522105d1f3c55eae6b1f33c1c6baa1d55
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171488"
---
# <a name="azure-infrastructure-availability"></a>Azure 基礎結構可用性
Azure 會根據透過虛擬化技術達到的龐大備援能力，提供穩定的可用性。 Azure 提供許多備援層級，以提供客戶資料的最大可用性。

## <a name="temporary-outages-and-natural-disaster"></a>暫時性中斷和天然災害
Microsoft Cloud 基礎結構和營運小組負責設計、建置、運作及加強保護雲端基礎結構。 這個小組可確保 Azure 基礎結構提供高可用性和可靠性、高效率及智慧型延展性。 小組會提供更安全、更私密且受信任的雲端。

不斷電供應系統和大量的行動電源在發生短期電源中斷時，可確保持續維持電力。 緊急發電機則可為長時間中斷以及計劃性維護提供備用電源。 如果發生天然災害，資料中心會使用現場燃料儲量運作。

高速且耐用的光纖網路會將資料中心與其他主要中樞和網際網路使用者連線在一起。 計算節點在更接近使用者的位置主控工作負載，以降低延遲、提供異地備援，並提升整體服務恢復功能。 工程師小組全天候工作，以確保服務能持續供應。

Microsoft 可透過進階監視和事件回應、服務支援與備份容錯移轉功能來確保高可用性。 分散各地的 Microsoft 作業中心全天候運作。 Azure 網路是世界上最大的網路之一。 光纖和內容發佈網路可連接資料中心和邊緣節點，以確保高效能和可靠性。

## <a name="disaster-recovery"></a>災害復原
Azure 會將您的資料長久保留在兩個位置中。 您可以選擇備份站台的位置。 在這兩個位置中，Azure 會持續維護 3 份健康情況良好的資料複本。

## <a name="database-availability"></a>資料庫可用性
Azure 可確保可透過具有持續資料庫可用性的網際網路閘道，從網際網路存取資料庫。 監視會以 5 分鐘的時間間隔評估作用中資料庫的健康情況和狀態。

## <a name="storage-availability"></a>儲存體可用性
Azure 會透過可高度擴充且耐久的儲存體服務提供儲存體，如此可提供連線端點。 這表示應用程式可以直接存取儲存體服務。 儲存體服務會有效率地處理傳入的儲存體要求，並維持交易完整性。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為協助保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)
