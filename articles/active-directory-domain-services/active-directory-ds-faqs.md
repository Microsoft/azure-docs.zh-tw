---
title: 常見問題集 - Azure Active Directory Domain Services | Microsoft Docs
description: 關於「Azure Active Directory 網域服務」的常見問題集
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 48731820-9e8c-4ec2-95e8-83dba1e58775
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: maheshu
ms.openlocfilehash: 8747e597883d96756e676061bde04b920b13cadd
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44023200"
---
# <a name="azure-active-directory-domain-services-frequently-asked-questions-faqs"></a>Azure Active Directory Domain Services：常見問題集 (FAQ)
此頁面會回答有關 Azure Active Directory Domain Services 的常見問題。 請隨時回來查看最新消息。

## <a name="troubleshooting-guide"></a>疑難排解指南
關於 Azure AD Domain Services 在設定或管理上常見問題的解決方案，請參閱[疑難排解指南](active-directory-ds-troubleshooting.md)。

## <a name="configuration"></a>組態
### <a name="can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory"></a>我可以針對單一 Azure AD 目錄建立多個受控網域嗎？
否。 您只能針對單一 Azure AD 目錄，建立由 Azure AD Domain Services 所服務的單一受控網域服務。  

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network"></a>是否可以啟用 Azure Resource Manager 虛擬網路中的 Azure AD 網域服務？
是。 可以啟用 Azure Resource Manager 虛擬網路中的 Azure AD Domain Services。 不再支援傳統 Azure 虛擬網路建立新的受控網域。

### <a name="can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network"></a>是否可以將我現有的受控網域從傳統虛擬網路移轉到 Resource Manager 虛擬網路？
目前不支援。 Microsoft 在未來會提供一個機制，將現有的受控網域從傳統虛擬網路移轉到 Resource Manager 虛擬網路。

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription"></a>是否可以在 Azure CSP (雲端方案提供者) 訂用帳戶中啟用 Azure AD Domain Services？
是。 了解如何啟用 [Azure CSP 訂用帳戶中的 Azure AD 網域服務](active-directory-ds-csp.md)。

### <a name="can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory"></a>我可以在同盟 Azure AD 目錄中啟用 Azure AD Domain Services 嗎？ 我不要將密碼雜湊同步至 Azure AD。 我可以針對此目錄啟用 Azure AD Domain Services 嗎？
否。 Azure AD Domain Services 需要存取使用者帳戶的密碼雜湊，以透過 NTLM 或 Kerberos 驗證使用者。 在同盟目錄中，密碼雜湊不是儲存在 Azure AD 目錄中。 因此，Azure AD Domain Services 不適用於此類 Azure AD 目錄。

### <a name="can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription"></a>我可以在訂用帳戶內的多個虛擬網路中使用 Azure AD 網域服務嗎？
服務本身並不直接支援這種情況。 受控網域一次只能在一個虛擬網路上使用。 不過，您可以在多個虛擬網路之間設定連線，以將 Azure AD 網域服務公開至其他虛擬網路。 了解如何[在 Azure 中連線虛擬網路](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)。

### <a name="can-i-enable-azure-ad-domain-services-using-powershell"></a>我是否可以使用 PowerShell 來啟用「Azure AD 網域服務」？
是。 了解[如何使用 PowerShell 啟用 Azure AD 網域服務](active-directory-ds-enable-using-powershell.md)。

### <a name="can-i-enable-azure-ad-domain-services-using-a-resource-manager-template"></a>是否可以使用 Resource Manager 範本啟用 Azure AD Domain Services？
是。 了解[如何使用 PowerShell 啟用 Azure AD 網域服務](active-directory-ds-enable-using-powershell.md)。

### <a name="can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain"></a>可以將網域控制站新增至 Azure AD 網域服務的受控網域嗎？
否。 Azure Active Directory Domain Services 所提供的網域是受控網域。 您不需要佈建、設定或管理此網域的網域控制站 - Microsoft 會以服務形式提供這些管理活動。 因此，您無法為受控網域新增其他網域控制站 (讀寫或唯讀)。

### <a name="can-guest-users-invited-to-my-directory-use-azure-ad-domain-services"></a>受邀到我目錄的來賓使用者是否可以使用 Azure AD Domain Services？
否。 以 [Azure AD B2B](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md) 邀請流程受邀到您 Azure AD 目錄的來賓使用者會同步至您的 Azure AD Domain Services 受控網域。 不過，這些使用者的密碼不會儲存在您的 Azure AD 目錄中。 因此，Azure AD Domain Services 無法將這些使用者的 NTLM 和 Kerberos 雜湊同步至您的受控網域。 如此一來，這類使用者便無法登入受控網域，也無法將電腦加入到受控網域中。

## <a name="administration-and-operations"></a>管理和作業
### <a name="can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop"></a>我可以使用遠端桌面連線到我的受控網域的網域控制站嗎？
否。 您沒有權限透過遠端桌面連接至受控網域上的網域控制站。 「AAD DC 系統管理員」群組的成員可以使用 AD 系統管理工具，例如 Active Directory 管理中心 (ADAC) 或 AD PowerShell 來管理受控網域。 這些工具會在加入受控網域的 Windows 伺服器上，使用「遠端伺服器管理工具」功能安裝。

### <a name="ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain"></a>我已經啟用 Azure AD 網域服務。 我應該使用哪一個使用者帳戶來將電腦加入此網域？
系統管理群組「AAD DC 系統管理員」的成員都能將電腦加入網域。 此外，此群組的成員會被授與已加入網域之電腦的遠端桌面存取權限。

### <a name="do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services"></a>我有 Azure AD Domain Services 所提供的受控網域的網域系統管理員權限嗎？
否。 您不會取得受控網域的系統管理權限。 您無法在網域內使用「網域系統管理員」和「企業系統管理員」權限。 在內部部署 Active Directory 中，網域系統管理員或企業系統管理員群組的成員也不會獲得受控網域的網域/企業系統管理員權限。

### <a name="can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains"></a>我可以在受控網域上使用 LDAP 或其他 AD 系統管理工具來修改群組成員資格嗎？
否。 您無法在 Azure AD 網域服務所服務的網域上修改群組成員資格。 這同樣適用於使用者屬性。 但是，您可能會在 Azure AD 中或內部部署網域上變更群組成員資格或使用者屬性。 這類變更會自動同步處理到 Azure AD 網域服務。

### <a name="how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain"></a>我對 Azure AD 目錄所做的變更要多久才會反映在我的受控網域中？
使用 Azure AD UI 或 PowerShell 在您 Azure AD 目錄中進行的變更會同步至您的受控網域。 這個同步處理程序會在背景執行。 首次同步處理完成之後，通常需要 20 分鐘的時間，在 Azure AD 中所做的變更才會反映在您的受控網域中。

### <a name="can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services"></a>可以擴充 Azure AD Domain Services 所提供之受控網域的結構描述嗎？
否。 結構描述是由 Microsoft 針對受控網域進行管理。 Azure AD 網域服務不支援結構描述延伸模組。

### <a name="can-i-modify-or-add-dns-records-in-my-managed-domain"></a>是否可以在受控網域中修改或新增 DNS 記錄？
是。 「AAD DC 系統管理員」群組的成員會被授與「DNS 系統管理」權限，以在受控網域中修改 DNS 記錄。 他們可以在加入受控網域且執行 Windows Server 的電腦上，使用 DNS 管理員主控台管理 DNS。 若要使用 DNS 管理員主控台，請安裝 DNS 伺服器工具，這是伺服器上的「遠端伺服器管理工具」選擇性功能的一部分。 如需 [管理、監視及疑難排解 DNS 的公用程式](https://technet.microsoft.com/library/cc753579.aspx) 詳細資訊，可在 TechNet 上取得。

### <a name="what-is-the-password-lifetime-policy-on-a-managed-domain"></a>受控網域上的密碼存留期原則為何？
Azure AD Domain Services 受控網域上的預設密碼存留期為 90 天。 此密碼存留期不會與 Azure AD 中設定的密碼存留期同步。 因此，您可能會遇到使用者的密碼在您的受控網域中到期，但在 Azure AD 中卻仍然有效的情況。 在這種情況下，使用者必須變更他們在 Azure AD 中的密碼，而新密碼將會同步至您的受控網域。 此外，使用者帳戶的 'password-does-not-expire' 和 'user-must-change-password-at-next-logon' 屬性也不會同步至您的受控網域。

### <a name="does-azure-ad-domain-services-provide-ad-account-lockout-protection"></a>Azure Active Directory Domain Services 是否提供 AD 帳戶鎖定保護？
是。 2 分鐘內在受控網域中輸入不正確的密碼五次，即會導致使用者帳戶鎖定 30 分鐘。 30 分鐘後，使用者帳戶會自動解除鎖定。 在受控網域上輸入不正確的密碼，並不會鎖定 Azure AD 的使用者帳戶。 Azure AD Domain Services 受控網域內的使用者帳戶才會遭到鎖定。

## <a name="billing-and-availability"></a>計費與可用性
### <a name="is-azure-ad-domain-services-a-paid-service"></a>Azure AD 網域服務是付費服務嗎？
是。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/active-directory-ds/)。

### <a name="is-there-a-free-trial-for-the-service"></a>是否可以免費試用服務？
此服務隨附於 Azure 的免費試用版。 您可以註冊以 [免費試用 Azure 一個月](https://azure.microsoft.com/pricing/free-trial/)。

### <a name="can-i-pause-an-azure-ad-domain-services-managed-domain"></a>我能否暫停 Azure AD Domain Services 受控網域？ 
否。 啟用 Azure AD Domain Services 受控網域後，您選取的虛擬網路中即有該服務可供使用，直到您停用/刪除受控網域為止。 無法暫停服務。 除非您刪除受控網域，否則會以每小時計費。

### <a name="can-i-failover-azure-ad-domain-services-to-another-region-for-a-dr-event"></a>我可以針對 DR 事件將 Azure AD Domain Services 容錯移轉到另一個區域嗎？
否。  Azure AD Domain Services 目前不提供異地備援部署模型。 它受限於 Azure 區域中的單一虛擬網路。 如果您想要利用多個 Azure 區域，您需要在 Azure IaaS VM 上執行 Active Directory 網域控制站。  [這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/identity/adds-extend-domain)可以找到架構指引。

### <a name="can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services"></a>我可以從 Enterprise Mobility Suite (EMS) 中取得 Azure AD 網域服務嗎？ 我是否需要 Azure AD Premium 才能使用 Azure AD 網域服務？
否。 Azure AD 網域服務是隨用隨付的 Azure 服務，並不是 EMS 的一部分。 Azure AD Domain Services 可以與所有 Azure AD 版本 (免費、基本及進階) 搭配使用。 計費方式是每小時依據使用量計費。

### <a name="what-azure-regions-is-the-service-available-in"></a>哪些 Azure 區域提供此服務？
請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services/)頁面，以查看可使用 Azure AD 網域服務的 Azure 區域清單。
