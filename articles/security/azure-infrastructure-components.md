---
title: Azure 資訊系統元件與界限
description: 本文提供 Microsoft Azure 架構和管理的一般描述。
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
ms.openlocfilehash: b2e8ef232e1b25c7d000f4683830ff2e188047fb
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39186471"
---
# <a name="azure-information-system-components-and-boundaries"></a>Azure 資訊系統元件與界限
本文提供 Azure 架構和管理的一般描述。 Azure 系統環境是由下列網路所組成：

- Microsoft Azure 生產網路 (Azure 網路)
- Microsoft 公司網路 (公司網路)

不同的 IT 團隊會負責這些網路的運作與維護。

## <a name="azure-architecture"></a>Azure 架構
Azure 是一個雲端運算平台和基礎結構，可透過資料中心網路建置、部署及管理應用程式和服務。 Microsoft 會管理這些資料中心。 Azure 會根據您指定的資源數目，建立以資源需求為主的虛擬機器 (VM)。 這些 VM 會在 Azure Hypervisor 上執行，其設計訴求是在雲端使用而且不可公開存取。

每個 Azure 實體伺服器節點上都有直接在硬體上執行的 Hypervisor。 Hypervisor 可將一個節點分成數量不拘的客體 VM。 每個節點也會有一部「根」VM，用來執行主機作業系統。 每部 VM 都已啟用 Windows 防火牆。 您可以藉由設定服務定義檔來定義哪些連接埠是可定址的。 這些連接埠是內部或外部環境中唯一可開放和定址的。 磁碟和網路的所有流量和存取都是由 Hypervisor 和根作業系統居中協調。

在主機層，Azure VM 會執行經過自訂且強化的最新 Windows Server 版本。 Azure 會使用 Windows Server 版本，其中只包含主控 VM 所需的元件。 這可改善效能並減少受攻擊面。 機器界限是由 Hypervisor 增強，該 Hypervisor 並不信賴作業系統安全性。

### <a name="azure-management-by-fabric-controllers"></a>Azure 網狀架構控制器管理

在 Azure 中，在實體伺服器 (刀鋒/節點) 上執行的 VM 會分組為大約 1000 個「叢集」。 VM 是由相應放大的備援平台軟體元件 (稱為網狀架構控制器 (FC)) 獨立管理。

每個 FC 會管理在其叢集中執行的應用程式生命週期，，以及佈建和監視受其控制的硬體健康狀態。 它會執行自主作業，例如在判斷伺服器失敗時，讓 VM 執行個體在狀況良好的伺服器上再生。 FC 也會執行應用程式管理作業，例如部署、更新及相應放大應用程式。

資料中心會分成數個叢集。 叢集會隔離 FC 層級的錯誤，以免特定類別的錯誤影響到出錯叢集以外的伺服器。 處理特定 Azure 叢集的 FC 會分組到 FC 叢集中。

### <a name="hardware-inventory"></a>硬體詳細目錄

FC 會針對啟動程序的設定程序，準備這期間的 Azure 硬體和網路裝置詳細目錄。 任何進入 Azure 生產環境的新硬體和網路元件都必須遵循啟動程序設定程序。 FC 會負責管理 datacenter.xml 組態檔中所列的整個庫存。

### <a name="fc-managed-operating-system-images"></a>FC 管理的作業系統映像

作業系統小組會提供虛擬硬碟格式的映像，這些映像會部署於 Azure 生產環境中的所有主機和客體 VM 上。 小組會透過自動化離線建置程序來建構這些基底映像。 基底映像是一個作業系統版本，其中的核心和其他核心元件已經過修改和最佳化，以支援 Azure 環境。

網狀架構管理的作業系統映像有三種類型：

- 主機 – 在主機 VM 上執行的自訂作業系統。
- 原生：在租用戶上執行的原生作業系統 (例如 Azure 儲存體)。 此作業系統沒有任何 Hypervisor。
- 客體︰在客體 VM 上執行的客體作業系統。

主機和原生 FC 管理作業系統的設計訴求是在雲端使用而且不可公開存取。

#### <a name="host-and-native-operating-systems"></a>主機和原生作業系統

主機和原生都是強化的作業系統映像，這些映像會裝載網狀架構代理程式，並且在計算節點 (作為節點上的第一個 VM 執行) 和儲存體節點上執行。 使用主機和原生最佳化基底映像的好處是，它會減少 API 或未用元件所暴露的介面區。 API 或未用元件會帶來高度安全性風險並提高作業系統的使用量。 縮減使用量的作業系統只包含 Azure 所需的元件。

#### <a name="guest-operating-system"></a>客體作業系統

在客體作業系統 VM 上執行的 Azure 內部元件沒有機會執行遠端桌面通訊協定。 基準組態設定的所有變更都必須經歷變更和發行管理程序。

## <a name="azure-datacenters"></a>Azure 資料中心
Microsoft 雲端基礎結構和運作 (MCIO) 小組會為所有 Microsoft 線上服務，管理實體基礎結構和資料中心設施。 MCIO 主要負責管理資料中心內的實體和環境控制項，以及管理和支援外部周邊網路裝置 (例如邊緣路由器和資料中心路由器)。 MCIO 也負責設定資料中心機櫃上最基本的伺服器硬體。 客戶與 Azure 沒有任何直接互動。

## <a name="service-management-and-service-teams"></a>服務管理和服務小組
Azure 服務的支援會由許多稱為「服務小組」的工程團隊所管理。 每個服務小組均負責一個區域的 Azure 支援。 每個服務小組都必須有全天候待命的工程師可調查及解決服務失敗。 根據預設，對於在 Azure 中運作的硬體，服務小組沒有實體存取權。

服務小組包括︰

- 應用程式平台
- Azure Active Directory
- Azure 計算
- Azure Net
- 雲端工程服務
- ISSD：安全性
- 多重要素驗證
- SQL Database
- 儲存體

## <a name="types-of-users"></a>使用者類型
Microsoft 的員工 (或約聘人員) 都會被視為內部使用者。 其他所有使用者則會被視為外部使用者。 所有 Azure 內部使用者都有其員工狀態，該狀態是依照可定義其客戶資料存取權 (有權存取或無權存取) 的敏感度層級進行分類。 下表說明 Azure 的使用者權限 (驗證發生後的授權權限)：

| 角色 | 內部或外部 | 敏感度層級 | 已獲授權的權限和執行的功能 | 存取類型
| --- | --- | --- | --- | --- |
| Azure 資料中心工程師 | 內部 | 無權存取客戶資料 | 管理廠房的實體安全性。 進行資料中心的內外部巡邏並監視所有進入點。 針對某些在資料中心內提供一般服務 (備餐、清潔) 或 IT 工作的不明人員，執行資料中心的內外部陪同服務。 進行網路硬體的例行監視和維護。 使用各種工具執行事件管理與修復工作。 進行資料中心內實體硬體的例行監視和維護。 視需要，以財產擁有者身份存取環境。 能夠執行鑑識調查、記錄事件的報告，以及要求必要的安全性訓練和政策需求。 控管重要安全性工具 (例如掃描器和記錄收集) 的操作擁有權和維護。 | 持續存取環境。 |
| Azure 事件分級 (快速回應工程師) | 內部 | 有權存取客戶資料 | 管理 MCIO、支援和工程小組之間的通訊。 將平台事件、部署問題和服務要求分級。 | 對環境的 Just-in-time 存取，但對非客戶系統的持續存取會受限。 |
| Azure 部署工程師 | 內部 | 有權存取客戶資料 | 在 Azure 的支援下，部署和升級平台元件、軟體和排定的組態變更。 | 對環境的 Just-in-time 存取，但對非客戶系統的持續存取會受限。 |
| Azure 客戶中斷支援 (租用戶) | 內部 | 有權存取客戶資料 | 偵錯和診斷個別計算租用戶和 Azure 帳戶的平台中斷和錯誤。 分析錯誤。 推動平台或客戶方面的重大修正，以及透過支援推動技術改善。 | 對環境的 Just-in-time 存取，但對非客戶系統的持續存取會受限。 |
| Azure 即時網站工程師 (監視工程師) 和事件 | 內部 | 有權存取客戶資料 | 使用診斷工具診斷及緩和平台的健康問題。 推動磁碟區驅動程式的修正，修復中斷造成的事項，並協助中斷後的還原動作。 | 對環境的 Just-in-time 存取，但對非客戶系統的持續存取會受限。 |
|Azure 客戶 | 外部 | N/A | N/A | N/A |

Azure 會使用唯一識別碼來驗證組織使用者和客戶 (或代表組織使用者採取行動的程序)。 這適用於所有屬於 Azure 環境一部分的資產和裝置。

### <a name="azure-internal-authentication"></a>Azure 內部驗證

Azure 內部元件之間的通訊受 TLS 加密保護。 在大部分的情況下，X.509 憑證都會自我簽署。 可從 Azure 網路外部存取連線的憑證是例外狀況，如同 FC 的憑證。 FC 具有 Microsoft 授權單位 (CA) 所簽發的憑證，而 Microsoft 授權單位則由受信任的根 CA 支援。 這讓 FC 公開金鑰得以輕易變換。 此外，Microsoft 開發人員工具會使用 FC 公用金鑰。 當開發人員提交新的應用程式映像時，這些映像會以 FC 公開金鑰加密以便保護任何內嵌的祕密。

### <a name="azure-hardware-device-authentication"></a>Azure 硬體裝置驗證

FC 會維護一組認證 (金鑰和/或密碼)，這一組認證用於向其控制之下的各種硬體裝置驗證 FC 本身。 Microsoft 會使用系統來防止有人存取這些認證。 具體而言，這些認證的傳輸、保存和使用都是為防止 Azure 開發人員、系統管理員和備份服務/人員存取敏感性、機密或私人資訊而設計。

Microsoft 會根據 FC 的主要身分識別公開金鑰使用加密。 這會發生在 FC 安裝和 FC 重新設定階段，以移轉用來存取網路硬體裝置的認證。 當 FC 需要認證時，FC 就會擷取認證並將其解密。

### <a name="network-devices"></a>網路裝置

Azure 網路小組會設定網路服務帳戶，讓 Azure 用戶端能夠向網路裝置 (路由器、交換器及負載平衡器) 進行驗證。

## <a name="secure-service-administration"></a>安全的服務管理
Azure 作業人員必須使用安全的系統管理工作站 (SAW)。 客戶可以使用具特殊權限的存取工作站來實作類似的控制項。 系統管理人員可透過 SAW 使用個別指派的系統管理帳戶，該帳戶不同於使用者的標準使用者帳戶。 SAW 會為敏感性帳戶提供可靠的工作站，並以該帳戶分隔作法為基礎。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為協助保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)
