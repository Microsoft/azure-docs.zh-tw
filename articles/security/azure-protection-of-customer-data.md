---
title: 在 Azure 中保護客戶資料
description: 本文說明 Azure 如何保護客戶資料。
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
ms.openlocfilehash: 0b702cec6113e6b31e34750872479dce162e4cb6
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173062"
---
# <a name="azure-customer-data-protection"></a>Azure 客戶資料保護   
預設會拒絕 Microsoft 運作與支援人員對於客戶資料的存取。 授與客戶資料的存取權時，需要經由主管核准，才能仔細管理和記錄存取權。 存取控制需求是由下列 Azure 安全性原則所建立：

- 依預設無法存取客戶資料。
- 客戶虛擬機器 (VM) 上沒有任何使用者或系統管理員帳戶。
- 授與完成工作所需的最低權限；稽核和記錄存取要求。

Microsoft 會將唯一的企業 Active Directory 帳戶指派給 Azure 支援人員。 Azure 依賴 Microsoft 企業 Active Directory (由 Microsoft Information Technology (MSIT) 管理) 來控制重要資訊系統存取權。 多重要素驗證是必要驗證，只能從安全的主控台授與存取權。

所有存取嘗試都會受到監視，而且可透過一組基本報告來顯示。

## <a name="data-protection"></a>資料保護
Azure 以預設和設為客戶選項的方式為客戶提供強大的資料安全性。

**資料隔離**：Azure 是一項多租用戶服務，意即將多位客戶的部署和 VM 儲存在同一個實體硬體上。 Azure 使用邏輯隔離來區隔每位客戶的資料。 隔離可提供多租用戶服務的規模和經濟優勢，同時嚴格地防止客戶存取彼此的資料。

**待用資料保護**：客戶有責任確保根據其標準將儲存在 Azure 中的資料加密。 Azure 提供各式各樣的加密功能，賦予客戶靈活彈性來選擇最符合本身需求的解決方案。 Azure Key Vault 會協助客戶以輕鬆的方式，控管雲端應用程式和服務用來加密資料的金鑰。 Azure 磁碟加密讓客戶能夠加密 VM。 Azure 儲存體服務加密可以加密所有放入客戶儲存體帳戶中的資料。

**傳輸中資料保護**：客戶可以對自己的 VM 與使用者之間的流量啟用加密。 Azure 會保護來自或送至外部元件的傳輸中資料以及內部傳輸中資料，例如兩部虛擬網路之間。 Azure 會依照 CESG/NCSC 的建議，使用業界標準的傳輸層安全性 (TLS) 1.2 或更新版通訊協定搭配 2,048 位元的 RSA/SHA256 加密金鑰，加密下列兩者之間的通訊：

- 客戶與雲端。
- Azure 系統與資料中心之間的內部通訊。

**加密**：客戶可以部署儲存體資料和傳輸中資料的加密，作為確保資料機密性與完整性的最佳做法。 客戶可以直接設定其 Azure 雲端服務，以使用 SSL 來保護來自網際網路的通訊，甚至保護其 Azure 託管 VM 之間的通訊。

**資料備援**：如果資料中心發生網路攻擊或實體損毀，Microsoft 可協助確保資料受到保護。 客戶可以選擇：

- 在國內儲存，以符合合規性或延遲考量。
- 在國外儲存，以符合安全性或災害復原目。

資料可以在所選地理區域內複寫以提供備援性，但不會傳輸到其外部。 客戶有多個選項可複寫資料，包括複本數目以及複寫資料中心的的數目和數目。

建立儲存體帳戶時，選取下列其中一個複寫選項：

- **本地備援儲存體 (LRS)**：本地備援儲存體可維護三個資料複本。 LRS 會在單一區域的單一設備內複寫三次。 LRS 可保護您的資料以避免一般的硬體故障，但無法避免單一設備的故障。
- **區域備援儲存體 (ZRS)**：區域備援儲存體可維護三個資料複本。 ZRS 會在二到三個設施中複寫三次，以提供比 LRS 更高的持久性。 複寫會發生於單一區域內或跨越兩個區域。 ZRS 有助於確保資料在單一地區內的持續性。
- **異地備援儲存體 (GRS)**：建立儲存體帳戶時，依預設會啟用異地備援儲存體。 GRS 可維護六個資料複本。 使用 GRS 時，系統會在主要區域內將您的資料複寫三次。 您的資料也會在與主要區域相距甚遠的次要區域內複寫三次，以提供最高等級的持久性。 在主要區域發生問題時，Azure 儲存體將會容錯移轉至次要區域。 GRS 有助於確保兩個不同區域中的資料持續性。

**資料解構**：當客戶刪除資料或離開 Azure 時，Microsoft 會遵循嚴格的標準，在重複使用之前覆寫儲存體資源，以及實際銷毀已解除委任的硬體。 Microsoft 會依照客戶要求以及在合約終止時，執行完整的資料刪除作業。

## <a name="customer-data-ownership"></a>客戶資料擁有權
Microsoft 不會檢查、核准或監視客戶部署至 Azure 的應用程式。 此外，Microsoft 不知道客戶選擇在 Azure 中儲存哪種資料。 對於客戶在 Azure 中輸入的資訊，Microsoft 並不會主張資料擁有權。

## <a name="records-management"></a>記錄管理
Azure 已建立後端資料的內部記錄保留需求。 客戶有責任找出自己的記錄保留需求。 對於儲存在 Azure 中的記錄，客戶需負責擷取其資料，並且在客戶指定的保留期間內於 Azure 外部保留內容。

Azure 允許客戶匯出資料以及稽核產品中的報告。 匯出的資料會儲存於本機，以便在客戶定義的保留期間內保留資訊。

## <a name="electronic-discovery-e-discovery"></a>電子探索 (e-discovery)
Azure 客戶在使用 Azure 服務時，有責任遵守電子探索需求。 如果 Azure 客戶必須保留其客戶資料，他們可以在本機匯出並儲存資料。 此外，客戶可以要求 Azure 客戶支援部門匯出其資料。 除了允許客戶匯出其資料，Azure 會在內部進行廣泛的記錄和監視。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
