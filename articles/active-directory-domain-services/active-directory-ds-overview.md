---
title: Azure Active Directory Domain Services 概觀 | Microsoft Docs
description: Azure Active Directory Domain Services 概觀
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 0d47178f-773e-45f9-9ff4-9e8cffa4ffa2
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/26/2017
ms.author: maheshu
ms.openlocfilehash: d30d164a3e2896b1a300bf665ca09f57ce97fdf3
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295864"
---
# <a name="azure-active-directory-ad-domain-services"></a>Azure Active Directory (AD) Domain Services
## <a name="overview"></a>概觀
Azure 基礎結構服務可讓您以敏捷的方式部署各種不同的運算解決方案。 利用 Azure 虛擬機器，您幾乎可以進行即時部署，而且只需要以分鐘計價。 利用對 Windows、Linux、SQL Server、Oracle、IBM、SAP 和 BizTalk 的支援性，您幾乎可以在所有作業系統上使用任何語言部署所有工作負載。 這些優點可讓您將內部部署的舊版應用程式移轉至 Azure，以便節約營運費用。

將內部部署應用程式移轉至 Azure 的一個重要層面是處理這些應用程式的身分識別需求。 目錄感知應用程式可能會依賴 LDAP 對公司目錄讀取或寫入存取，或仰賴 Windows 整合式驗證 (Kerberos 或 NTLM 驗證) 來驗證使用者。 在 Windows Server 執行的企業營運 (LOB) 應用程式通常都會部署在已加入網域的電腦，因此只要使用群組原則就能安全地加以管理。 為了將內部部署應用程式「隨即轉移」(lift-and-shift) 至雲端，就必須先為這些企業身分識別基礎結構解決相依性問題。

為了替他們部署在 Azure 的應用程式滿足身分識別需求，系統管理員經常採用下列幾種解決方案：

* 在執行於 Azure 基礎結構服務和企業目錄內部部署的諸多工作負載間，部署站對站 VPN 連線。
* 使用 Azure 虛擬機器設定複本網域控制站，來延伸公司的 AD 網域/樹系基礎結構。
* 使用已部署為 Azure 虛擬機器的網域控制站來部署 Azure 中的獨立網域。

這些方法都面臨高成本和管理負擔。 系統管理員需要在 Azure 中使用虛擬機器才能部署網域控制站。 此外，他們需為這些虛擬機器進行管理、保護、修補、監視、備份和疑難排解。 太依賴與內部部署目錄之間的 VPN 連線，使 Azure 中所部署的工作負載容易發生暫時性網路問題或網路中斷。 這些網路中斷會導致執行時間降低，並減損這些應用程式的可靠性。

我們將 Azure AD 網域服務設計為可提供更簡單的替代方法。

### <a name="watch-an-introductory-video"></a>觀看簡介影片

>[!VIDEO https://www.youtube.com/embed/T1Nd9APNceQ]

## <a name="introducing-azure-ad-domain-services"></a>Azure AD 網域服務簡介

Azure AD 網域服務提供受控網域服務，例如：加入網域、群組原則、LDAP、Kerberos/NTLM 驗證，與 Windows Server Active Directory 完全相容。 您可以取用這些網域服務，而不需要您在雲端中部署、管理和修補網域控制站。 Azure AD 網域服務與您現有的 Azure AD 租用戶整合，因此能讓使用者使用其公司認證登入。 此外，您可以使用現有的群組和使用者帳戶，安全地存取資源，以確保更順暢地將內部部署資源「隨即轉移」(lift-and-shift) 至 Azure 基礎結構服務。

Azure AD 網域服務的功能運作順暢，不論您的 Azure AD 租用戶是僅限雲端或與您的內部部署 Active Directory 同步。

### <a name="azure-ad-domain-services-for-cloud-only-organizations"></a>僅限雲端的組織的 Azure AD 網域服務

僅限雲端的 Azure AD 租用戶 (通常稱為「受控租用戶」) 沒有任何內部部署身分識別使用量。 換句話說，使用者帳戶、其密碼和群組成員資格為雲端原生 - 也就是在 Azure AD 中建立和管理。 考慮一下 Contoso 是僅限雲端的 Azure AD 租用戶。 如下圖中所示，Contoso 的系統管理員已在 Azure 基礎結構服務中設定虛擬網路。 應用程式和伺服器工作負載會部署在 Azure 虛擬機器的此虛擬網路中。 由於 Contoso 是僅限雲端的租用戶，所有的使用者身分識別、其認證和群組成員資格都是在 Azure AD 中建立和管理。

![Azure AD 網域服務概觀](./media/active-directory-domain-services-overview/aadds-overview.png)

Contoso 的 IT 系統管理員可以為其 Azure AD 租用戶啟用 Azure AD 網域服務，並選擇使網域服務可以在此虛擬網路中提供使用。 之後，Azure AD 網域服務會佈建受控網域，並使它可在虛擬網路中使用。 Contoso 的 Azure AD 租用戶中可使用的所有使用者帳戶、群組成員資格和使用者認證，也會在此新建立的網域中可供使用。 這項功能可讓組織中的使用者使用其公司認證登入網域，例如，透過遠端桌面從遠端連線到加入網域的電腦。 系統管理員可以使用現有群組成員資格在網域中為資源佈建存取權。 部署在虛擬網路的虛擬機器中的應用程式可以使用加入網域、LDAP 讀取、LDAP 繫結、NTLM、Kerberos 驗證、群組原則之類的功能。

Azure Active Directory Domain Services 所佈建的受控網域的幾個顯著層面如下所示：

* Contoso 的 IT 系統管理員不需要管理、修補或監視此網域或此受控網域的任何網域控制站。
* 不需管理對這個網域的 AD 複寫。 使用者帳戶、群組成員資格和來自 Contoso 的 Azure AD 租用戶認證會自動在此受控網域中提供。
* 因為網域是由 Azure Active Directory Domain Services 管理，Contoso 的 IT 系統管理員沒有這個網域的網域系統管理員或企業管理員權限。

### <a name="azure-ad-domain-services-for-hybrid-organizations"></a>混合式組織的 Azure AD 網域服務
具有混合式 IT 基礎結構的組織會混合取用雲端資源和內部部署資源。 這類組織會從其內部部署目錄同步身分識別資訊到其 Azure AD 租用戶。 隨著混合式組織尋求將他們的更多內部部署應用程式移轉至雲端，尤其是舊版目錄感知應用程式，Azure AD 網域服務對組織而言可說是非常有用。

Litware Corporation 已部署 [Azure AD Connect](../active-directory/hybrid/whatis-hybrid-identity.md)，以便從其內部部署目錄同步身分識別資訊到其 Azure AD 租用戶。 同步處理的身分識別資訊包括使用者帳戶、用於驗證的認證雜湊 (密碼同步) 與群組成員資格。

> [!NOTE]
> **混合式組織必須使用密碼同步處理，才能使用 Azure AD 網域服務**。 這是因為在 Azure AD 網域服務提供的受控網域中需要使用者的認證，才能透過 NTLM 或 Kerberos 驗證方法驗證這些使用者。
>
>

![Litware Corporation 的 Azure AD 網域服務](./media/active-directory-domain-services-overview/aadds-overview-synced-tenant.png)

上圖顯示具有混合式 IT 基礎結構的組織 (例如 Litware Corporation) 如何使用 Azure AD 網域服務。 Litware 需要網域服務的應用程式和伺服器工作負載會部署在 Azure 基礎結構服務的虛擬網路中。 Litware 的 IT 系統管理員可以為其 Azure AD 租用戶啟用 Azure AD 網域服務，並選擇使受控網域可以在此虛擬網路中提供使用。 由於 Litware 是使用混合式 IT 基礎結構的組織，使用者帳戶、群組和認證會從其內部部署目錄同步到其 Azure AD 租用戶。 這項功能可讓使用者使用其公司認證登入網域，例如，透過遠端桌面從遠端連線到加入網域的電腦。 系統管理員可以使用現有群組成員資格在網域中為資源佈建存取權。 部署在虛擬網路的虛擬機器中的應用程式可以使用加入網域、LDAP 讀取、LDAP 繫結、NTLM、Kerberos 驗證、群組原則之類的功能。

Azure Active Directory Domain Services 所佈建的受控網域的幾個顯著層面如下所示：

* 受控網域是獨立網域。 它不是 Litware 的內部部署網域的延伸。
* Litware 的 IT 系統管理員不需要管理、修補或監視此受控網域的網域控制站。
* 不需管理對這個網域的 AD 複寫。 使用者帳戶、群組成員資格和來自 Litware 內部部署目錄的認證會透過 Azure AD Connect 同步到 Azure AD。 在受控網域中會自動提供這些使用者帳戶、群組成員資格和認證。
* 因為網域是由 Azure AD 網域服務管理，Litware 的 IT 系統管理員沒有這個網域的網域系統管理員或企業管理員權限。

## <a name="benefits"></a>優點
使用 Azure AD 網域服務，您可以享用下列優點：

* **簡單** – 只要簡單按幾下，就可以滿足部署至 Azure 基礎結構服務的虛擬機器的身分識別需求。 您不需要部署和管理 Azure 中的身分識別基礎結構，或安裝至您的內部部署身分識別基礎結構的連線。
* **整合** - Azure AD 網域服務與 Azure AD 租用戶密切整合。 您現在可以使用 Azure AD 做為能夠同時迎合現代化應用程式及傳統目錄感知應用程式需求的整合雲端式企業目錄。
* **相容** - Azure AD 網域服務是以 Windows Server Active Directory 經實證的企業級基礎結構為基礎。 因此，您的應用程式可仰賴相容性更高的 Windows Server Active Directory 功能。 並非 Windows Server AD 中的所有功能目前都可在 Azure AD 網域服務中使用。 不過，可用的功能與您在內部部署基礎結構中依賴的相對應 Windows Server AD 功能相容。 LDAP、Kerberos、NTLM、群組原則和網域加入功能構成了成熟的供應項目，已對各種 Windows Server 版本經過測試並進一步修改。
* **符合成本效益** - 有了 Azure AD 網域服務，您可以免除為支援傳統目錄感知應用程式所需承受與管理身分識別基礎結構相關聯的基礎結構和管理負擔。 您可以將這些應用程式移至 Azure 基礎結構服務，並受益於營運費用的大幅節約。


## <a name="next-steps"></a>後續步驟
### <a name="learn-more-about-azure-ad-domain-services"></a>深入了解 Azure AD Domain Services
* [特性](active-directory-ds-features.md)
* [部署案例](active-directory-ds-scenarios.md)
* [了解 Azure AD Domain Services 是否適合您的使用案例](active-directory-ds-comparison.md)
* [了解 Azure AD Domain Services 如何與 Azure AD 目錄同步處理](active-directory-ds-synchronization.md)

### <a name="get-started-with-azure-ad-domain-services"></a>開始使用 Azure AD 網域服務
* [使用 Azure 入口網站來啟用 Azure AD Domain Services](active-directory-ds-getting-started.md)
