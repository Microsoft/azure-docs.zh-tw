---
title: 如何為內部部署應用程式提供安全的遠端存取
description: 涵蓋如何使用 Azure AD 應用程式 Proxy 為您的內部部署應用程式提供安全的遠端存取。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/31/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 428f094dae2b9a69b58912190d2959a7dfc467ec
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39365257"
---
# <a name="how-to-provide-secure-remote-access-to-on-premises-applications"></a>如何為內部部署應用程式提供安全的遠端存取

現今的員工想要隨時隨地都能在任何裝置發揮生產力。 他們想要在自己的裝置上工作，不論這些裝置是平板電腦、手機或膝上型電腦。 而且他們期望能夠存取其所有的應用程式︰雲端中的 SaaS 應用程式以及內部部署的公司應用程式。 傳統上，提供內部部署應用程式的存取權會涉及虛擬私人網路 (VPN) 或非軍事區 (DMZ)。 這些解決方案不僅複雜且難以確保安全，而且設定及管理成本也很高。

還有更好的辦法！

在行動至上、雲端至上的世界裡，現代化的員工需要現代化的遠端存取解決方案。 Azure AD 應用程式 Proxy 是 Azure Active Directory 的一項功能，並提供遠端存取做為服務。 這表示它很容易部署、使用和管理。

[!INCLUDE [identity](../../../includes/azure-ad-licenses.md)]

## <a name="what-is-azure-active-directory-application-proxy"></a>什麼是 Azure Active Directory 應用程式 Proxy？
Azure AD 應用程式 Proxy 針對 Web 應用程式託管的內部部署，提供單一登入 (SSO) 及安全的遠端存取。 您想要發佈的部份應用程式包括 SharePoint 網站、Outlook Web Access 或您擁有的其他任何 LOB Web 應用程式。 這些內部部署 Web 應用程式會與 Azure AD (O365 所使用的相同身分識別和控制平台) 整合。 終端使用者可以使用和 O365 以及其他與 Azure AD 整合之 SaaS 應用程式相同的存取方式，來存取內部部署應用程式。 您不需要變更網路基礎結構，或需要 VPN 才能為使用者提供此解決方案。

## <a name="why-is-application-proxy-a-better-solution"></a>為什麼應用程式 Proxy 是較佳的解決方案？
Azure AD 應用程式 Proxy 可對所有內部部署應用程式提供簡單、安全且符合成本效益的遠端存取解決方案。

Azure AD 應用程式 Proxy：

* **簡單**
   * 您不需要變更或更新應用程式，即可使用應用程式 Proxy。 
   * 使用者享有一致的驗證體驗。 使用者可以使用 MyApps 入口網站，對於雲端中的 SaaS 應用程式和內部部署的應用程式取得單一登入。 
* **安全**
   * 當您使用 Azure AD 應用程式 Proxy 發佈應用程式時，您可以利用 Azure 中豐富的授權控制項和安全性分析。 您會取得雲端級別安全性和 Azure 安全性功能，例如條件式存取和雙步驟驗證。
   * 您不需要開啟透過防火牆的任何輸入連線，為使用者提供遠端存取。 
* **符合成本效益**
   * 應用程式 Proxy 在雲端中運作，因此可以節省時間和金錢。 內部部署解決方案則一般需要您設定及維護 DMZ、Edge Server 或其他複雜的基礎結構。  

## <a name="what-kind-of-applications-work-with-application-proxy"></a>哪種應用程式可與應用程式 Proxy 搭配運作？
透過 Azure AD 應用程式 Proxy，您可以存取不同類型的內部應用程式︰

* 使用[整合式 Windows 驗證](application-proxy-configure-single-sign-on-with-kcd.md)來進行驗證的 Web 應用程式  
* 使用表單架構或[標頭型](application-proxy-configure-single-sign-on-with-ping-access.md)存取的 Web 應用程式  
* 您想要公開給不同裝置上豐富應用程式的 Web API  
* 裝載在[遠端桌面閘道](application-proxy-integrate-with-remote-desktop-services.md)之後的應用程式  
* 與 Active Directory Authentication Library (ADAL) 整合的豐富型用戶端應用程式

## <a name="how-does-application-proxy-work"></a>Application Proxy 的運作方式為何？
需要設定才能使 Application Proxy 運作的兩個元件：連接器和外部端點。 

連接器是位於網路內部 Windows 伺服器上的輕量型代理程式。 連接器有助於從雲端中的 Application Proxy 服務到應用程式內部部署的流量流程。 連接器僅使用輸出連線，因此您不需要開啟任何輸入連接埠，或在 DMZ 中放置任何物件。 連接器是無狀態的，且在必要時會從雲端提取資訊。 如需連接器的詳細資訊，例如如何負載平衡和驗證，請參閱[了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)。 

外部端點是使用者在網路外部聯繫應用程式的方式。 外部端點可以直接存取您決定的外部 URL，也可以透過 MyApps 入口網站存取應用程式。 使用者存取這些端點的其中一個時，會在 Azure AD 中進行驗證，然後透過連接器路由至內部部署應用程式。

 ![Azure AD 應用程式 Proxy 圖表](./media/application-proxy/azureappproxxy.png)

1. 使用者會透過應用程式 Proxy 服務存取應用程式，然後被導向 Azure AD 登入頁面進行驗證。
2. 成功登入之後，系統會產生權杖並傳送給用戶端裝置。
3. 用戶端會將權杖傳送至應用程式 Proxy 服務，該服務會取出權杖的使用者主體名稱 (UPN) 和安全性主體名稱 (SPN)，然後將要求導向至應用程式 Proxy 連接器。
4. 如果您已設定單一登入，則連接器會代表使用者執行其他任何所需的驗證。
5. 連接器會將要求傳送至內部部署應用程式。  
6. 回應會透過應用程式 Proxy 服務與連接器傳送給使用者。

### <a name="single-sign-on"></a>單一登入
Azure AD 應用程式 Proxy 會針對使用整合式 Windows 驗證 (IWA) 的應用程式或宣告感知應用程式提供單一登入 (SSO)。 如果您的應用程式使用 IWA，應用程式 Proxy 會模擬使用 Kerberos 限制委派的使用者來提供 SSO。 如果您具有信任 Azure Active Directory 的宣告感知應用程式，則可以使用 SSO，因為使用者已由 Azure AD 驗證。

如需有關 Kerberos 的詳細資訊，請參閱[有關 Kerberos 限制委派 (KCD) 您想要知道的一切](https://blogs.technet.microsoft.com/applicationproxyblog/2015/09/21/all-you-want-to-know-about-kerberos-constrained-delegation-kcd)。

### <a name="managing-apps"></a>管理應用程式
一旦使用應用程式 Proxy 發佈應用程式後，即可管理該應用程式，如同管理 Azure 入口網站中的其他任何企業應用程式一般。 您可以使用 Azure Active Directory 安全性功能，例如條件式存取和雙步驟驗證、控制使用者權限，以及自訂您應用程式的商標。 

## <a name="get-started"></a>開始使用

設定 Application Proxy 之前，確定您有支援的 [Azure Active Directory 版本](https://azure.microsoft.com/pricing/details/active-directory/)，以及您是全域管理員的 Azure AD 目錄。

開始使用 Application Proxy 有兩個步驟：

1. [啟用應用程式 Proxy 並設定連接器](application-proxy-enable.md)。    
2. [發佈應用程式](application-proxy-publish-azure-portal.md) ：使用快速且簡單的精靈發佈內部部署應用程式並提供遠端存取。

## <a name="whats-next"></a>後續步驟
您發佈第一個應用程式後，應用程式 Proxy 還有其他更多用途：

* [啟用單一登入](application-proxy-configure-single-sign-on-with-kcd.md)
* [使用您自己的網域名稱發行應用程式](application-proxy-configure-custom-domain.md)
* [了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)
* [使用現有的內部部署 Proxy 伺服器](application-proxy-configure-connectors-with-proxy-servers.md) 
* [設定自訂首頁](application-proxy-configure-custom-home-page.md)

如需最新消息，請查閱 [應用程式 Proxy 部落格](http://blogs.technet.com/b/applicationproxyblog/)

