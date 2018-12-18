---
title: Azure 雲端方案提供者的 Azure Active Directory Domain Services | Microsoft Docs
description: Azure 雲端方案提供者的 Azure Active Directory Domain Services。
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mahesh-unnikrishnan
editor: curtand
ms.assetid: 56ccb219-11b2-4e43-9f07-5a76e3cd8da8
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/08/2017
ms.author: maheshu
ms.openlocfilehash: 8d87312591f44bac5fd9a4ff63eccc19333a870c
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42145732"
---
# <a name="azure-active-directory-ad-domain-services-for-azure-cloud-solution-providers-csp"></a>Azure 雲端方案提供者 (CSP) 的 Azure Active Directory (AD) Domain Services
本文章說明如何使用 Azure CSP 訂用帳戶中的 Azure Active Directory Domain Services。

## <a name="overview-of-azure-csp"></a>Azure CSP 的概觀
Azure CSP 是針對 Microsoft 合作夥伴的方案，為各種 Microsoft 雲端服務提供授權通路。 Azure CSP 使得合作夥伴能夠管理銷售、擁有計費關係、提供技術及計費支援，並成為客戶的單一連絡窗口。 此外，Azure CSP 會提供一組完整的工具，包括自助服務入口網站和隨附的 API。 這些工具使得 CSP 合作夥伴能夠輕鬆地佈建及管理 Azure 資源，以及提供客戶和其訂用帳戶的帳單。

[合作夥伴中心入口網站](https://docs.microsoft.com/azure/cloud-solution-provider/overview/partner-center-overview)可做為所有 Azure CSP 合作夥伴的進入點。 它提供豐富的客戶管理功能、自動化處理，還有更多功能。 Azure CSP 合作夥伴可以使用網頁型 UI 或使用 PowerShell 與各種 API 呼叫來使用合作夥伴中心的功能。

下圖說明 CSP 模型高層的運作方式。 Contoso 有 Azure AD Active Directory。 它們與某個 CSP 有合作關係，該 CSP 會在其 Azure CSP 訂用帳戶中部署和管理資源。 Contoso 可能也有定期 (直接) 的 Azure 訂用帳戶，這部分是直接向 Contoso 計費。

![CSP 模型概觀](./media/csp/csp_model_overview.png)

CSP 合作夥伴的租用戶有三個特殊的專員群組 - 管理專員、技術服務人員專員和銷售專員。 管理專員群組已指派給 Contoso 的 Azure AD 目錄中的租用戶系統管理員角色。 如此一來，屬於 CSP 合作夥伴管理專員群組的使用者，會在 Contoso 的 Azure AD 目錄中有租用戶管理員權限。 當 CSP 合作夥伴為 Contoso 佈建 Azure CSP 訂用帳戶時，其管理專員群組即獲指派該訂用帳戶的擁有者角色。 如此一來，CSP 合作夥伴的管理專員具有代表 Contoso 佈建 Azure 資源 (例如：虛擬機器、虛擬網路以及 Azure AD Domain Services) 的必要權限。

如需詳細資訊，請參閱 [Azure CSP 概觀](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-overview)

## <a name="benefits-of-using-azure-ad-domain-services-in-an-azure-csp-subscription"></a>在 Azure CSP 訂用帳戶中使用 Azure AD Domain Services 的優點
Azure AD Domain Services 在 Azure 中提供 Windows Server AD 相容服務，例如 LDAP、Kerberos/NTLM 驗證、網域加入、群組原則和 DNS。 十多年來，已經建置許多應用程式以對使用這些功能的 AD 運作。 許多獨立軟體廠商 (ISV) 已在其客戶的內部部署建置和部署應用程式。 支援這些應用程式的工作很繁重，因為通常需要部署這些應用程式所在不同的環境的存取權。 有了 Azure CSP 訂用帳戶，您便擁有 Azure 調整與彈性較簡單的替代方案。

Azure AD Domain Services 現在支援 Azure CSP 訂用帳戶。 您現在可以在繫結至客戶的 Azure AD 目錄的 Azure CSP 訂用帳戶中部署應用程式。 如此一來，您的員工 (支援人員) 可以使用組織的公司認證來管理、管控和服務部署應用程式所在的虛擬機器。 此外，您可以為客戶的 Azure AD 目錄佈建 Azure AD Domain Services 受控網域。 您的應用程式會連接到您客戶的受控網域。 因此，應用程式內依賴於 Kerberos/NTLM、LDAP 或 [System.DirectoryServices API](https://msdn.microsoft.com/library/system.directoryservices) 的功能可對客戶的目錄運作順暢。 一般客戶可因為使用您的應用程式作為服務而大幅獲益，而不需擔心維護應用程式部署所在的基礎結構。

您在該訂用帳戶中所使用的所有 Azure 資源的計費 (包括 Azure AD Domain Services) 會退回給您。 至於銷售、計費、技術支援等，您可以保有與客戶的關係的完整控制權。有了 Azure CSP 平台的彈性，一小組的支援專員即可以服務已部署您的應用程式執行個體的許多這類客戶。


## <a name="csp-deployment-models-for-azure-ad-domain-services"></a>Azure AD 網域服務的 CSP 部署模型
有兩種方式，可讓您使用 Azure AD Domain Services 搭配 Azure CSP 訂用帳戶。 根據客戶的安全性和簡單性考量挑選適合的一個。

### <a name="direct-deployment-model"></a>直接部署模型
在此部署模型中，會在屬於 Azure CSP 訂用帳戶的虛擬網路內啟用 Azure AD Domain Services。 CSP 合作夥伴的管理專員擁有下列權限：
* 客戶的 Azure AD 目錄中的全域系統管理員權限。
* Azure CSP 訂用帳戶上的訂用帳戶擁有者權限。

![直接部署模型](./media/csp/csp_direct_deployment_model.png)

在此部署模型中，CSP 提供者的管理專員可以管理客戶的身分識別。 這些管理專員可以在客戶的 Azure AD 目錄等之內佈建新使用者、群組、新增應用程式。此部署模型可能適合不具專用的身分識別管理員或偏好由 CSP 合作夥伴來代表他們管理身分識別的較小型組織。


### <a name="peered-deployment-model"></a>對等部署模型
在此部署模型中，會在屬於客戶，也就是客戶所支付直接 Azure 訂用帳戶的虛擬網路內啟用 Azure AD Domain Services。 然後 CSP 合作夥伴才能在屬於客戶 CSP 訂用帳戶的虛擬網路內部署應用程式。 之後可以使用 Azure 虛擬網路對等互連來連接虛擬網路。 如此一來，Azure CSP 訂用帳戶中由 CSP 合作夥伴部署的工作負載/應用程式可以連接到在客戶的直接 Azure 訂用帳戶中佈建的客戶受控網域。

![對等部署模型](./media/csp/csp_peered_deployment_model.png)

此部署模型提供權限的區隔，讓 CSP 合作夥伴的技術服務人員專員管理 Azure 訂用帳戶，以及在其內部署和管理資源。 不過，CSP 合作夥伴的技術服務人員專員不需要對客戶的 Azure AD 目錄具備全域系統管理員權限。 客戶的身分識別系統管理員可以繼續為其組織管理身分識別。

在 ISV (獨立軟體廠商) 提供其內部部署應用程式裝載版本的案例中 (也需要連線到客戶的 AD)，可能適合此部署模型。


## <a name="administering-azure-ad-domain-services-managed-domains-in-csp-subscriptions"></a>管理 CSP 訂用帳戶中的 Azure AD Domain Services 受控網域
在 Azure CSP 訂用帳戶中管理受控網域時，適用下列重要事項：

* **CSP 管理專員可以使用其認證來佈建受控網域**：Azure AD Domain Services 支援 Azure CSP 訂用帳戶。 因此，屬於 CSP 合作夥伴管理專員群組的使用者可以提供新的 Azure AD Domain Services 受控網域。

* **CSP 可以使用 PowerShell 編寫為客戶 建立新受控網域的指令碼**：如需詳細資訊，請參閱[如何使用 PowerShell 啟用 Azure AD Domain Services](active-directory-ds-enable-using-powershell.md)。

* **CSP 管理專員無法使用其認證在受控網域執行進行中的管理工作**：CSP 管理使用者無法使用其認證在受控網域內執行例行管理工作。 這些使用者對客戶的 Azure AD 目錄而言屬於外部，而其認證在客戶的 Azure AD 目錄內無法使用。 因此，Azure AD Domain Services 沒有這些使用者 Kerberos 和 NTLM 密碼雜湊的存取權。 如此一來，這類使用者在 Azure AD Domain Services 的受控網域上無法通過驗證。

  > [!WARNING]
  > **您必須在客戶的目錄內建立使用者帳戶，才能在受控網域上執行進行中的管理工作。**
  > 您無法使用 CSP 管理使用者的認證來登入受控網域。 請使用屬於客戶 Azure AD 目錄的使用者帳戶認證來執行此動作。 對於將虛擬機器加入受控網域、管理 DNS、管理群組原則之類工作，您需要這些認證。
  >

* **針對進行中管理建立的使用者帳戶必須新增至 'AAD DC Administrators' 群組**：'AAD DC Administrators' 群組具有在受控網域上執行特定委派的系統管理工作的權限。 這些工作包括設定 DNS、建立組織單位、管理群組原則等。若要讓 CSP 合作夥伴在受控網域上執行這類工作，需要客戶的 Azure AD 目錄中建立使用者帳戶。 您必須將此帳戶的認證與 CSP 合作夥伴的管理專員分享。 此外，必須將此使用者帳戶新增至 'AAD DC Administrators' 群組，以讓受控網域上的組態工作可使用此使用者帳戶來執行。


## <a name="next-steps"></a>後續步驟
* [註冊參加 Azure CSP 計畫](https://docs.microsoft.com/partner-center/enrolling-in-the-csp-program)，然後開始透過 Azure CSP 建立企業。
* 檢閱 [Azure CSP 中可用的 Azure 服務](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-available-services)清單。
* [使用 PowerShell 啟用 Azure AD Domain Services](active-directory-ds-enable-using-powershell.md)
* [開始使用 Azure AD Domain Services](active-directory-ds-getting-started.md)
