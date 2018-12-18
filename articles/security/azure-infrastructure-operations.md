---
title: Azure 生產環境運作與管理
description: 本文提供 Azure 生產網路管理和操作的一般描述。
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
ms.openlocfilehash: 21ae81f1d8423a9d05208ec6d8c4f31d909d2f9f
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173154"
---
# <a name="azure-production-operations-and-management"></a>Azure 生產環境運作與管理    
Azure 生產網路的管理和運作是 Azure 運作小組與 Azure SQL Database 之間的協調工作。 小組在環境中使用了數個系統和應用程式效能監視工具。 此外，他們會使用適當的工具監視網路裝置、伺服器、服務和應用程式程序。

為確保能在 Azure 環境中安全地執行服務，運作小組會實作多個層級的監視、記錄和報告，包括下列動作：

- Microsoft Monitoring Agent (MMA) 主要從許多地方 (包括網狀架構控制器 (FC) 和根作業系統 OS) 蒐集監視和診斷記錄資訊，並寫入至記錄檔。 代理程式最後會將經過處理的資訊子集，推送到預先設定的 Azure 儲存體帳戶。 此外，獨立式監視和診斷服務可讀取各種監視和診斷記錄資料並摘要說明資訊。 監視和診斷服務會將資訊寫入整合的記錄。 Azure 會使用自訂建置的 Azure 安全性監視，這是 Azure 監視系統的擴充功能。 它有一些元件可觀察、分析及報告平台中不同點的安全性相關事件。

- Azure SQL Database Windows Fabric 平台可為 Azure SQL Database 提供管理、部署、開發和作業監督服務。 此平台可提供分散式、多步驟的部署服務、健康情況監視、自動修復和服務版本合規性。 它可提供下列服務：

   - 高逼真度開發環境的服務建模功能 (資料中心叢集很昂貴且稀有)。
   - 適用於服務啟動和維護的單鍵部署和升級工作流程。
   - 透過自動化修復工作流程啟用自我修復功能的健康狀態報告。
   - 橫跨分散式系統各節點的即時監視、警示和偵錯設施。
   - 集中收集操作資料和計量，以便進行分散式根本原因分析和服務深入解析。
   - 用於部署、變更管理和監視的操作工具。
   - Azure SQL Database Windows Fabric 平台和看門狗指令碼會持續執行並且即時監視。

如果發生任何異常狀況，則會啟動 Azure 事件分級小組所遵循的事件回應程序。 相關的 Azure 支援人員會收到通知，以回應事件。 問題追蹤和解決已記載於集中式票證系統並加以管理。 依據保密合約 (NDA)，可應要求取得系統運作時間計量。

## <a name="corporate-network-and-multi-factor-access-to-production"></a>公司網路與生產環境的多重要素存取
公司網路使用者群包括 Azure 支援人員。 公司網路可支援內部公司函式，並可供存取用於 Azure 客戶支援的內部應用程式。 公司網路在邏輯和實體上與 Azure 生產網路分隔。 Azure 人員會使用 Azure 工作站和筆記型電腦存取公司網路。 所有使用者都必須具有 Azure Active Directory (Azure AD) 帳戶 (包括使用者名稱和密碼)，才能存取公司網路資源。 使用由 Microsoft Information Technology 核發給所有 Microsoft 人員、約聘員工及廠商並加以管理的 Azure AD 帳戶，即可存取公司網路。 唯一使用者識別碼可根據人員在 Microsoft 的雇用狀態來區分人員。

透過向 Active Directory 同盟服務 (AD FS) 驗證，控制內部 Azure 應用程式的存取權。 AD FS 是由 Microsoft Information Technology 主控的服務，可透過套用安全的權杖和使用者宣告來提供公司網路使用者的驗證。 AD FS 可讓內部 Azure 應用程式向 Microsoft 公司 Active Directory 網域驗證使用者。 若要從公司網路環境存取生產網路，使用者必須使用多重要素驗證進行驗證。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)
