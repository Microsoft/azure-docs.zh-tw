---
title: Azure AD Domain Services：比較 Azure AD Domain Services 與 DIY 網域控制器 | Microsoft Docs
description: 比較 Azure Active Directory Domain Services 與 DIY 網域控制器
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 165249d5-e0e7-4ed1-aa26-91a05a87bdc9
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/07/2017
ms.author: maheshu
ms.openlocfilehash: acb125ef807193e79e45bca7e217efe3ffb31698
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505945"
---
# <a name="how-to-decide-if-azure-ad-domain-services-is-right-for-your-use-case"></a>如何判斷 Azure AD 網域服務是否適合您的使用案例
Azure AD Domain Services 可讓您在 Azure 基礎結構服務中部署工作負載，而不必擔心要在 Azure 中維護您的身分識別基礎結構。 此受控服務不同於您自行部署及管理的典型 Windows Server Active Directory 部署。 服務容易部署，並且提供自動化的健康情況監視和修復。 我們持續不斷地提升該服務，以新增常見部署案例的支援。

若要決定是否使用 Azure AD Domain Services，我們建議參閱下列閱讀資料：

* 請參閱 [Azure AD 網域服務所提供的功能](active-directory-ds-features.md)清單。
* 檢閱常見 [Azure AD 網域服務的部署案例](active-directory-ds-scenarios.md)。
* 最後， [比較 Azure AD 網域服務與自己動手做 AD 選項](active-directory-ds-comparison.md#compare-azure-ad-domain-services-to-diy-ad-domain-in-azure)。

## <a name="compare-azure-ad-domain-services-to-diy-ad-domain-in-azure"></a>比較 Azure AD 網域服務與 Azure 中的 DIY AD 網域
下表可協助您決定使用 Azure AD 網域服務或在 Azure 中管理自己的 AD 基礎結構。

| **功能** | **Azure AD 網域服務** | **Azure VM 中的「自己動手做」AD** |
| --- |:---:|:---:|
| [**受管理的服務**](active-directory-ds-comparison.md#managed-service) |**&#x2713;** |**&#x2715;** |
| [**安全的部署**](active-directory-ds-comparison.md#secure-deployments) |**&#x2713;** |系統管理員必須確保部署安全。 |
| [**DNS 伺服器**](active-directory-ds-comparison.md#dns-server) |           **&#x2713;** (受控服務) |**&#x2713;** |
| [**Domain or Enterprise administrator privileges**](active-directory-ds-comparison.md#domain-or-enterprise-administrator-privileges) |**&#x2715;** |**&#x2713;** |
| [**加入網域**](active-directory-ds-comparison.md#domain-join) |**&#x2713;** |**&#x2713;** |
| [**使用 NTLM 和 Kerberos 的網域驗證**](active-directory-ds-comparison.md#domain-authentication-using-ntlm-and-kerberos) |**&#x2713;** |**&#x2713;** |
| [**Kerberos 限制委派**](active-directory-ds-comparison.md#kerberos-constrained-delegation)|資源型|資源型與帳戶型|
| [**自訂 OU 結構**](active-directory-ds-comparison.md#custom-ou-structure) |**&#x2713;** |**&#x2713;** |
| [**結構描述延伸模組**](active-directory-ds-comparison.md#schema-extensions) |**&#x2715;** |**&#x2713;** |
| [**AD 網域/樹系信任**](active-directory-ds-comparison.md#ad-domain-or-forest-trusts) |**&#x2715;** |**&#x2713;** |
| [**LDAP read**](active-directory-ds-comparison.md#ldap-read) |**&#x2713;** |**&#x2713;** |
| [**安全的 LDAP (LDAPS)**](active-directory-ds-comparison.md#secure-ldap) |**&#x2713;** |**&#x2713;** |
| [**LDAP write**](active-directory-ds-comparison.md#ldap-write) |**&#x2715;** |**&#x2713;** |
| [**Group Policy**](active-directory-ds-comparison.md#group-policy) |**&#x2713;** |**&#x2713;** |
| [**地理位置分散部署**](active-directory-ds-comparison.md#geo-dispersed-deployments) |**&#x2715;** |**&#x2713;** |

#### <a name="managed-service"></a>受控服務
Azure AD 網域服務網域是由 Microsoft 管理。 您不必擔憂修補、更新、監視、備份，以及確保網域的可用性。 這些管理工作是 Microsoft Azure 針對受控網域提供的服務。

#### <a name="secure-deployments"></a>安全的部署
根據 Microsoft 的 AD 部署安全性建議，會安全地鎖定受控網域。 這些建議源自於 AD 產品團隊數十年設計和支援 AD 部署的經驗。 在自己動手做部署中，您需要採取特定的部署步驟才能鎖定/保護您的部署。

#### <a name="dns-server"></a>DNS 伺服器
Azure AD 網域服務管理的網域包含受控 DNS 服務。 「AAD DC 系統管理員」群組的成員可以在受控網域上管理 DNS。 此群組的成員可獲得受控網域的完整 DNS 管理權限。 使用遠端伺服器管理工具 (RSAT) 套件中包含 [DNS 管理主控台] 可以執行 DNS 管理。
[詳細資訊](active-directory-ds-admin-guide-administer-dns.md)

#### <a name="domain-or-enterprise-administrator-privileges"></a>網域或企業系統管理員權限
AAD DS 受控網域上不提供這些提高的權限。 無法對 AAD-DS 受控網域部署需要這些提高權限的應用程式。 委派系統管理群組 (名為「AAD DC 系統管理員」) 的成員可以使用較小的系統管理權限子集。 這些權限包括設定 DNS、設定群組原則、取得已加入網域的電腦系統管理員權限。

#### <a name="domain-join"></a>加入網域
您可以將虛擬機器加入受控網域，其做法類似於您將電腦加入至 AD 網域。

#### <a name="domain-authentication-using-ntlm-and-kerberos"></a>使用 NTLM 和 Kerberos 的網域驗證
透過 Azure AD 網域服務，您可以使用公司認證來向受控網域進行驗證。 憑證會與您的 Azure AD 租用戶保持同步。 對於已同步處理的租用戶，Azure AD Connect 會確保在內部部署進行的認證變更會同步處理至 Azure AD。 在 DIY 網域設定中，您可能需要為使用者設定與內部部署 AD 的 AD 網域信任，以便使用其公司認證進行驗證。 或者，您可能需要設定 AD 複寫，以確保使用者密碼會同步處理至 Azure 網域控制站虛擬機器。

#### <a name="kerberos-constrained-delegation"></a>Kerberos 限制委派
在 Active Directory Domain Services 的受控網域上，您沒有「網域系統管理員」權限。 因此，您無法設定帳戶型 (傳統) Kerberos 限制委派。 不過，您可以設定更安全的資源型限制委派。
[詳細資訊](active-directory-ds-enable-kcd.md)

#### <a name="custom-ou-structure"></a>自訂 OU 結構
「AAD DC 系統管理員」群組的成員可以在受控網域內建立自定 OU。 建立自訂 OU 的使用者會獲得整個 OU 的完整管理權限。
[詳細資訊](active-directory-ds-admin-guide-create-ou.md)

#### <a name="schema-extensions"></a>結構描述延伸模組
您無法擴充 Azure Active Directory Domain Services 受控網域的基底結構描述。 因此，依賴 AD 結構描述擴充功能 (例如，使用者物件之下的新屬性) 的應用程式無法轉移至 AAD DS 網域。

#### <a name="ad-domain-or-forest-trusts"></a>AD 網域或樹系信任
無法設定受控網域以設定與其他網域的信任關係 (輸入/輸出)。 因此，資源樹系部署案例無法使用 Azure AD Domain Services。 同樣地，在您不想將密碼同步處理至 Azure AD 的部署中，無法使用 Azure AD Domain Services。

#### <a name="ldap-read"></a>LDAP 讀取
受控網域支援 LDAP 讀取工作負載。 因此，您可以部署對受控網域執行 LDAP 讀取作業的應用程式。

#### <a name="secure-ldap"></a>安全的 LDAP
您可以設定 Azure AD 網域服務，以提供受控網域的安全 LDAP 存取，包括透過網際網路。
[詳細資訊](active-directory-ds-admin-guide-configure-secure-ldap.md)

#### <a name="ldap-write"></a>LDAP 寫入
受控網域對使用者物件而言是唯讀的。 因此，對使用者物件的屬性執行 LDAP 寫入作業的應用程式不適用於受控網域。 此外，不能在受控網域中變更使用者密碼。 另一個範例是不允許在受控網域中修改群組成員資格或群組屬性。 不過，在 Azure AD (透過 PowerShell/Azure 入口網站) 或內部部署 AD 中所做的使用者屬性或密碼變更會同步處理至 AAD-DS 受控網域。

#### <a name="group-policy"></a>群組原則
「AADDC 電腦」和「AADDC 使用者」容器各有內建 GPO。 您可以自訂這些內建 GPO，來設定群組原則。 「AAD DC 系統管理員」群組成員也可以建立自訂 GPO (群組原則物件)，並將它們連結至現有的 OU (包括自訂 OU)。
[詳細資訊](active-directory-ds-admin-guide-administer-group-policy.md)

#### <a name="geo-dispersed-deployments"></a>分散各地的部署
Azure AD 網域服務受控網域可以在Azure 的單一虛擬網路中使用。 在世界各地多個 Azure 區域中都需要可用網域控制站的案例中，在 Azure IaaS VM 中設定網域控制站可能是更好的替代方法。


## <a name="do-it-yourself-diy-ad-deployment-options"></a>「自己動手做」(DIY) AD 部署選項
在有些部署使用案例中，您可能需要 Windows Server AD 安裝所提供的一些功能。 在這些情況下，請考慮下列其中一個自己動手做 (DIY) 選項︰

* **獨立雲端網域︰** 您可以使用已設定為網域控制站的 Azure 虛擬機器來設定獨立「雲端網域」。 此基礎結構不會與內部部署 AD 環境整合。 此選項需要一組不同的「雲端認證」才能登入/管理雲端中的 VM。
* **資源樹系部署︰** 您可以使用設定為網域控制站的 Azure 虛擬機器，在資源樹系拓撲中設定網域。 接下來，您可以設定與內部部署 AD 環境的 AD 信任關係。 您可以將電腦 (Azure VM) 的網域加入至雲端中的這個資源樹系。 使用者驗證會透過內部部署目錄的 VPN/ExpressRoute 連線發生。
* **將內部部署網域擴充至 Azure︰** 您可以使用 VPN/ExpressRoute 連線將 Azure 虛擬網路連線到內部部署網路。 此設定可讓 Azure VM 加入您的內部部署 AD。 另一個替代方法是在 Azure 中將內部部署網域的複本網域控制站升級為 VM。 您可以接著將它設定為透過 VPN/ExpressRoute 連線複寫到內部部署目錄。 此部署模式有效地將內部部署網域擴充至 Azure。

> [!NOTE]
> 您可能認為 DIY 選項比較適合您的部署使用案例。 請考慮 [分享意見反應](active-directory-ds-contact-us.md) ，幫助我們了解哪些功能可協助您在未來選擇 Azure AD 網域服務。 此意見反應協助我們提升服務，使其更符合您的部署需求和使用案例。
>
>

我們已發行 [在 Azure 虛擬機器上部署 Windows Server Active Directory 的指導方針](https://msdn.microsoft.com/library/azure/jj156090.aspx) ，讓 DIY 安裝變得更容易。

## <a name="related-content"></a>相關內容
* [功能 - Azure AD 網域服務](active-directory-ds-features.md)
* [部署案例 - Azure AD 網域服務](active-directory-ds-scenarios.md)
* [在 Azure 虛擬機器上部署 Windows Server Active Directory 的指導方針](https://msdn.microsoft.com/library/azure/jj156090.aspx)
