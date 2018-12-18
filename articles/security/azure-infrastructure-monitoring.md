---
title: Azure 基礎結構監視
description: 本文討論 Azure 生產網路的監視。
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
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 59f54487d89aee199e35e741ac4683d4784818a0
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39172478"
---
# <a name="azure-infrastructure-monitoring"></a>Azure 基礎結構監視   

## <a name="configuration-and-change-management"></a>組態和變更管理
Azure 會每年審查和更新組態設定以及硬體、軟體和網路裝置的基準組態。 從開發和/或測試環境進入生產環境之前，變動會經過開發、測試及核准。

Azure 安全性與合規性小組及服務小組會檢閱 Azure 服務所需的基準組態。 服務小組的檢閱是在部署其生產服務之前的測試過程中進行。

## <a name="vulnerability-management"></a>弱點管理
安全性更新管理可協助系統抵禦已知的弱點。 Azure 使用整合式部署系統，管理 Microsoft 軟體安全性更新的散發與安裝。 Azure 也能夠利用 Microsoft Security Response Center (MSRC) 的資源。 MSRC 會每天夜以繼日地尋找、監視、回應及解決安全性事件和雲端弱點。

## <a name="vulnerability-scanning"></a>弱點掃描
弱點掃描會在伺服器作業系統、資料庫與網路裝置上執行。 弱點掃描至少每季進行。 Azure 與獨立的技術顧問簽約，以執行 Azure 界限的滲透測試。 紅隊 (Red-team) 演練也會定期執行，而結果會用於進行安全性改善。

## <a name="protective-monitoring"></a>防護監視
Azure 安全性已定義主動監視的需求。 服務小組會根據這些需求來設定主動監視工具。 主動監視工具包括 Microsoft Monitoring Agent (MMA) 和 System Centre Operations Manager。 這些工具已設定成在需要立即採取行動的情況下，對 Azure 安全性人員提供即時警示。

## <a name="incident-management"></a>事件管理
Microsoft 會實作安全性事件管理程序，以便在發生事件時協調回應。

如果 Microsoft 察覺到未經授權存取其設備或設施中儲存的客戶資料，或未經授權存取這類設備或設施導致遺失、洩漏或改變客戶資料，Microsoft 就會採取下列動作：

- 立即將安全性事件通知客戶。
- 立即調查安全性事件，並將詳細的安全性事件資訊提供給客戶。
- 採取合理和立即的步驟來降低影響，並將安全性事件所造成的任何損害降到最低。

已建立事件管理架構，其中會定義角色和配置職責。 Azure 安全性事件管理小組負責管理安全性事件，包括必要時提升及確保專家小組的參與程度。 Azure 作業管理員負責監督安全性和隱私權事件的調查與解決。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)
