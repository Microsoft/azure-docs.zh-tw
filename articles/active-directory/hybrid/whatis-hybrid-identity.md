---
title: 使用 Azure Active Directory 連接 Active Directory。 | Microsoft Docs
description: Azure AD Connect 會整合您的內部部署目錄與 Azure Active Directory。 這可讓您為與 Azure AD 整合的 Office 365、Azure 和 SaaS 應用程式提供通用身分識別。
keywords: Azure AD Connect 簡介, Azure AD Connect 概觀, 何謂 Azure AD Connect, 安裝 active directory
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 59bd209e-30d7-4a89-ae7a-e415969825ea
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/18/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 40ac3ca92c65607df056b883608dde60d816143e
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181767"
---
# <a name="hybrid-identity-and-microsofts-identity-solutions"></a>混合式身分識別和 Microsoft 的身分識別解決方案
現今的企業和公司混合使用內部部署和雲端應用程式的情形越來越多。  在內部部署環境和雲端中擁有應用程式以及需要存取這些應用程式的使用者，已成為一項困難的案例。

Microsoft 的身分識別解決方案可跨越內部部署和雲端架構功能，建立單一使用者身分識別以用於所有資源的驗證和授權，不論位於何處。 我們稱之為混合式身分識別。

## <a name="what-is-azure-ad-connect"></a>何謂 Azure AD Connect？

Azure AD Connect 是一種 Microsoft 工具，其設計目的是要符合並完成混合式身分識別的目標。  這可讓您為與 Azure AD 整合之 Office 365、Azure 和 SaaS 應用程式的使用者提供通用身分識別。  它可提供下列功能：
    
- [同步處理](how-to-connect-sync-whatis.md) - 此元件負責建立使用者、群組和其他物件。 它也負責確保您的內部部署使用者和群組的身分識別資訊符合雲端。  它會負責同步處理密碼雜湊與 Azure AD。
-   [AD FS 與同盟整合](how-to-connect-fed-whatis.md) - 同盟是 Azure AD Connect 的選用組件，可用來使用內部部署 AD FS 基礎結構來設定混合環境。 它也提供 AD FS 管理功能，例如憑證更新與其他的 AD FS 伺服器部署。
-   [傳遞驗證](how-to-connect-pta.md) - 另一個選用元件，可讓使用者在內部部署環境和雲端中使用相同的密碼，但不需要額外的同盟環境基礎結構
-   [狀況監控](whatis-hybrid-identity-health.md) - Azure AD Connect Health 可以提供健全監控，並在 Azure 入口網站中提供檢視此活動的中央位置。 


![何謂 Azure AD Connect](./media/whatis-hybrid-identity/arch.png)



## <a name="what-is-azure-ad-connect-health"></a>何謂 Azure AD Connect Health？

Azure Active Directory (Azure AD) Connect Health 可協助您監視和了解內部部署身分識別基礎結構和同步處理服務。 它提供重要身分識別元件 (例如 Active Directory Federation Services (AD FS) 伺服器、Azure AD Connect 伺服器 (也稱為同步處理引擎)、Active Directory 網域控制站等) 的監視功能，讓您維護可靠的 Office 365 或 Microsoft Online Services 連線。它也使這些元件的相關重要資料點變得容易存取，讓您取得使用情況和其他重要情資，以做出明智的決策。

這些資訊會呈現在 [Azure AD Connect Health 入口網站](https://aka.ms/aadconnecthealth)中。 在 Azure AD Connect Health 入口網站中，您可以檢視警示、效能監視、使用情況分析等資訊。 Azure AD Connect Health 就像功能濾鏡，可讓您集中監控重要身分識別元件的健康狀態。

![Azure AD Connect Health 是什麼](./media/whatis-hybrid-identity-health/aadconnecthealth2.png)


隨著 Azure AD Connect Health 的功能越來越多，入口網站透過身分識別的功能濾鏡，提供單一儀表板。 您會有更為穩固健全的整合式環境，讓使用者的辦事能力更強。


## <a name="why-use-azure-ad-connect"></a>為何要使用 Azure AD Connect？
將內部部署目錄與 Azure AD 整合可提供一個通用身分識別來存取雲端和內部部署資源，讓使用者變得更有生產力。 使用者和組織可以享受到下列好處：

* 使用者可以使用單一身分識別來存取內部部署應用程式和雲端服務，例如 Office 365。
* 單一工具即可提供輕鬆進行同步處理和登入的部署經驗。
* 提供您案例的最新功能。 Azure AD Connect 會取代舊版的身分識別整合工具，如 DirSync 和 Azure AD Sync。如需詳細資訊，請參閱 [混合式身分識別目錄整合工具比較](plan-hybrid-identity-design-considerations-tools-comparison.md)。

## <a name="why-use-azure-ad-connect-health"></a>為何使用 Azure AD Connect Health？
將內部部署目錄與 Azure AD 整合時，使用者會更具生產力，因為只要一個通用身分識別，即可存取雲端和內部部署資源。 然而，此整合也會帶來挑戰，必須確保此環境良好健全，才能讓使用者從任何裝置可靠地存取內部部署和雲端中的資源。 Azure AD Connect Health 協助您監視及深入了解用來存取 Office 365 或其他 Azure AD 應用程式的內部部署身分識別基本結構。 使用方式相當簡單，您只需將代理程式安裝在各個內部部署身分識別伺服器中即可。

### <a name="azure-ad-connect-health-for-ad-fshow-to-connect-health-adfsmd"></a>[適用於 AD FS 的 Azure AD Connect Health](how-to-connect-health-adfs.md)
在 Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 和 Windows Server 2016 上，適用於 AD FS 的 Azure AD Connect Health 支援 AD FS 2.0。 它也支援監視可驗證外部網路存取的 AD FS Proxy 或 Web 應用程式 Proxy 伺服器。 藉由輕鬆且快速地安裝健康情況代理程式，適用於 AD FS 的 Azure AD Connect Health 可提供您一組重要功能。

#### <a name="key-benefits-and-best-practices"></a>重要優勢和最佳做法

- *強化的安全性*
  - [外部網路鎖定趨勢](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs)
  - [失敗的登入報告](how-to-connect-health-adfs.md#risky-ip-report-public-preview) 
  - 在[符合隱私權規範](reference-connect-health-user-privacy.md)    
- 取得所有[重大 ADFS 系統問題](how-to-connect-health-alert-catalog.md#alerts-for-active-directory-federation-services)的警示
    - 伺服器設定和可用性 
    - [效能和連線能力](how-to-connect-health-adfs.md#performance-monitoring-for-ad-fs) 
  - 定期維護    
- *容易部署及管理*
  - 快速[代理程式安裝](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs) 
  - 代理程式自動升級至最新版 
  - 幾分鐘內就可在入口網站中取得資料    
- *豐富的[使用量計量](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs)* 
  - 應用程式使用量排名
  - 網路位置與 TCP 連線
  - 每一部伺服器的權杖要求數    
- 良好的使用者體驗 
  - 來自 Azure 入口網站的儀表板方式
  - [透過電子郵件警示](how-to-connect-health-adfs.md#alerts-for-ad-fs)    

#### <a name="feature-highlight"></a>功能特點

*   利用警示進行監視，以了解 AD FS 與 AD FS Proxy 伺服器何時健康狀態不良
*   重大警示的電子郵件通知
*   效能資料的趨勢，適合用於 AD FS 的容量規劃
*   透過樞紐 (應用程式、使用者、網路位置等) 提供 AD FS 登入的使用情況分析
*   AD FS 報告，例如嘗試使用錯誤使用者名稱/密碼的前 50 名使用者及其最後一個 IP 位址
*   失敗 AD FS 登入的高風險 IP 報告

如需深入了解，請參閱[搭配 AD FS 使用 Azure AD Connect Health](how-to-connect-health-adfs.md)

### <a name="azure-ad-connect-health-for-synchow-to-connect-health-syncmd"></a>[適用於同步處理的 Azure AD Connect Health](how-to-connect-health-sync.md)
適用於同步處理的 Azure AD Connect Health 可監視及提供內部部署 Active Directory 與 Azure AD 之間發生的同步處理相關資訊。 適用於同步處理的 Azure AD Connect Health 提供下列一組主要功能：

* 利用警示進行監視，以了解 Azure AD Connect 伺服器 (也稱為同步處理引擎) 何時健康狀態不良
* 重大警示的電子郵件通知
* 同步處理操作情資，包括同步處理作業的延遲圖表以及不同作業 (如新增、更新、刪除) 的趨勢
* 有關同步處理屬性和上次成功匯出至 Azure AD 的快速概覽資訊
* 物件層級同步處理錯誤的相關報告\(不需要 Azure AD Premium\)

如需深入了解，請參閱[使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)

### <a name="azure-ad-connect-health-for-ad-dshow-to-connect-health-addsmd"></a>[適用於 AD DS 的 Azure AD Connect Health](how-to-connect-health-adds.md)
適用於 Active Directory Domain Services (AD DS) 的 Azure AD Connect Health 可以監視安裝在 Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 及 Windows Server 2016 上的網域控制站。 健康狀態代理程式可讓您從雲端監視內部部署 AD DS 環境。 適用於 AD DS 的 Azure AD Connect Health 提供下列一組主要功能：

* 監視警示可偵測網域控制站健全狀況不良，並發送重大警示的電子郵件通知
* 網域控制站儀表板可讓您快速檢視網域控制站的健全狀況和操作狀態
* 複寫狀態儀表板擁有最新的複寫資訊，以及偵測到錯誤時可用的疑難排解指南連結
* 可在任一處快速存取常用效能計數器的效能資料圖形，其為疑難排解和監視用途所必需

如需深入了解，請參閱[搭配 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)

## <a name="next-steps"></a>後續步驟


- [硬體和必要條件](how-to-connect-install-prerequisites.md) 
- [快速設定](how-to-connect-install-express.md)
- [自訂設定](how-to-connect-install-custom.md)
- [密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)|
- [傳遞驗證](how-to-connect-pta.md)
- [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)
- [安裝 Azure AD Connect Health 代理程式](how-to-connect-health-agent-install.md) 
- [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)
- [版本歷程記錄](reference-connect-version-history.md)
- [目錄整合工具比較](plan-hybrid-identity-design-considerations-tools-comparison.md)
- [Azure AD Connect 常見問題集](reference-connect-faq.md)









