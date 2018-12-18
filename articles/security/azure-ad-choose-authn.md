---
title: 針對 Azure AD 混合式身分識別解決方案選擇正確的驗證方法 | Microsoft Docs
description: 本指南可協助中大型組織內的 CEO、CIO、CISO、首席身分識別架構設計人員、企業架構設計人員，以及負責選擇 Azure AD 混合式身分識別解決方案的安全性和 IT 決策者。
services: active-directory
keywords: ''
author: martincoetzer
ms.author: martincoetzer
ms.date: 04/12/2018
ms.topic: article
ms.service: active-directory
ms.workload: identity
ms.openlocfilehash: a322edbc6825261dde0fd926a362ca037739e06e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388053"
---
# <a name="choose-the-right-authentication-method-for-your-azure-active-directory-hybrid-identity-solution"></a>針對 Azure Active Directory 混合式身分識別解決方案選擇正確的驗證方法 

本文是一系列文章中的開端，可協助組織實作完整的 Azure Active Directory (Azure AD) 混合式身分識別解決方案。 此解決方案可概述為[混合式身分識別數位轉換架構](https://aka.ms/aadframework) \(英文\)。 它涵蓋組織可以著重的業務成果和目標，以便實作強固且安全的混合式身分識別解決方案。 

架構的第一個商務結果會列出組織的需求，以便在使用者存取雲端應用程式時保護驗證程序。 驗證安全業務成果的第一個商務目標是，讓使用者能夠使用其內部部署使用者名稱和密碼登入雲端應用程式。 這個登入程序和使用者驗證方式，讓一切都可在雲端中達成。

選擇正確的驗證方法是組織想要將自己的應用程式移至雲端的第一個考量。 基於下列因素，請勿輕易做出此決策：

1. 它是組織想要移至雲端的第一項決策。 

2. 組織若要在雲端佔有一席之地，驗證方法非常重要。 控制對所有雲端資料和資源的存取。

3. 它是 Azure AD 中所有其他進階安全性和使用者體驗功能的基礎。

4. 驗證方法在實作之後難以變更。

身分識別是新的 IT 安全性控制平面。 所以驗證是組織在進入新雲端世界時的守護神。 組織需要能加強其安全性，並保護其雲端應用程式免於入侵者侵襲的身分識別控制平面。

### <a name="out-of-scope"></a>超出範圍
目前不含內部部署目錄使用量的組織不在本文的適用範圍內。 這類企業通常只需在雲端建立身分識別，不需要混合式身分識別解決方案。 僅限雲端的身分識別只存在雲端，不會與對應的內部部署身分識別相關聯。

## <a name="authentication-methods"></a>驗證方法
利用 Azure AD 混合式身分識別解決方案作為新的控制平面，驗證就能成為雲端存取的基礎。 選擇正確的驗證方法是設定 Azure AD 混合式身分識別解決方案的第一項重要決策。 請實作使用 Azure AD Connect 所設定的驗證方法，這也會在雲端佈建使用者。

若要選擇驗證方法，需要考慮實作您選取項目的時間、現有基礎結構、複雜度及成本。 這些因素對每個組織而言不同，並可能隨時間改變。 

>[!VIDEO https://www.youtube.com/embed/YtW2cmVqSEw]

Azure AD 針對混合式身分識別解決方案支援下列驗證方法。

### <a name="cloud-authentication"></a>雲端驗證
當您選擇此驗證方法時，Azure AD 會處理使用者的登入程序。 搭配無縫單一登入 (SSO)，使用者不必重新輸入認證，就能登入雲端應用程式。 若使用雲端驗證，有兩個選項可供您選擇： 

**Azure AD 密碼雜湊同步處理**。 這是在 Azure AD 中啟用內部部署目錄物件驗證的最簡單方式。 使用者可以使用他們在內部部署中所使用的相同使用者名稱和密碼，而不需部署任何其他基礎結構。 無論您選擇何種驗證方法，Azure AD 的某些進階功能 (例如 Identity Protection) 還是需要密碼雜湊同步處理。

> [!NOTE] 
> 密碼永遠都不會以純文字儲存，或在 Azure AD 中使用可回復的演算法進行加密。 如需密碼雜湊同步處理實際程序的詳細資訊，請參閱[使用 Azure AD Connect 同步來實作密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-password-hash-synchronization)。 

**Azure AD 傳遞驗證**。 藉由使用在一或多部內部部署伺服器上執行的軟體代理程式，為 Azure AD 驗證服務提供簡單密碼驗證。 伺服器會直接透過您的內部部署 Active Directory 來驗證使用者，這樣可以確保密碼驗證不會在雲端中發生。 

具有立即強制執行內部部署使用者帳戶狀態、密碼原則及登入時數等安全性需求的公司，會使用這個驗證方法。 如需實際傳遞驗證程序的詳細資訊，請參閱[使用 Azure AD 傳遞驗證來進行使用者登入](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta)。

### <a name="federated-authentication"></a>同盟驗證
當您選擇此驗證方法時，Azure AD 會將驗證程序交給另一個信任的驗證系統 (例如內部部署 Active Directory 同盟服務 (AD FS)) 來驗證使用者的密碼。

驗證系統可以提供其他進階驗證需求。 例如以智慧卡為基礎的驗證或第三方多重要素驗證。 如需詳細資訊，請參閱[部署 Active Directory 同盟服務](https://docs.microsoft.com/windows-server/identity/ad-fs/deployment/windows-server-2012-r2-ad-fs-deployment-guide) \(機器翻譯\)。

下一節將使用決策樹，協助您判斷何種驗證方法最適合您。 它會協助您針對 Azure AD 混合式身分識別解決方案部署雲端，判斷要部署雲端還是同盟驗證。

## <a name="decision-tree"></a>決策樹

![Azure AD 驗證決策樹](media/azure-ad/azure-ad-authn-image1.png)

## <a name="detailed-considerations"></a>詳細考量

### <a name="cloud-authentication-password-hash-synchronization"></a>雲端驗證：密碼雜湊同步處理

* **投入量**。 密碼雜湊同步處理需要最少的部署、維護和基礎結構投入量。  這個層級的投入量，通常適用於只需要使用者登入 Office 365、SaaS 應用程式和其他 Azure AD 型資源的組織。 開啟時，密碼雜湊同步處理屬於 Azure AD Connect 同步處理程序的一部分，每兩分鐘執行一次。

* **使用者體驗**。 若要改善使用者的登入體驗，請部署具有密碼雜湊同步處理的無縫 SSO。 當使用者登入時，無縫 SSO 會排除不必要的提示。

* **進階案例**。 如果組織選擇此選項，就能搭配 Azure AD Identity Protection 報表來使用身分識別的見解。 例如認證外洩的報表。 Windows Hello 企業版是另一個選項，其中具有[當您使用密碼雜湊同步處理時的特殊需求](https://docs.microsoft.com/windows/access-protection/hello-for-business/hello-identity-verification)。 

    需要多重要素驗證與密碼雜湊同步處理的組織，必須使用 Azure AD 多重要素驗證。 這些組織不能使用第三方或內部部署多重要素驗證方法。

* **商務持續性**。 使用碼雜湊同步處理搭配雲端驗證具有高可用性，因為雲端服務可擴展到所有 Microsoft 資料中心。 若要確保密碼雜湊同步處理不會在擴展期間關閉，請在待命組態中以預備模式部署第二個 Azure AD Connect 伺服器。

* **考量**。 目前密碼雜湊同步處理不會立即強制執行內部部署帳戶狀態中的變更。 在此情況下，使用者可以存取雲端應用程式，直到使用者帳戶狀態同步到 Azure AD 為止。 組織可能想要克服這項限制，可以在系統管理員對內部部署使用者帳戶狀態執行大量更新之後，執行新的同步處理循環。 例如停用帳戶。

> [!NOTE]
> 目前系統不會使用 Azure AD Connect，將密碼過期和帳戶鎖定狀態同步到 Azure AD。 

如需部署步驟，請參閱[實作密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-password-hash-synchronization)。

### <a name="cloud-authentication-pass-through-authentication"></a>雲端驗證：傳遞驗證  

* **投入量**。 針對傳遞驗證，您需要在現有伺服器上安裝一或多個 (我們建議 3 個) 輕量型代理程式。 這些代理程式必須能夠存取內部部署 Active Directory Domain Services，包括內部部署 AD 網域控制站。 它們需要對外存取網際網路，並存取您的網域控制站。 基於這個理由，不支援在周邊網路部署代理程式。 

    傳遞驗證需要對於網域控制站的未受限制網路存取權。 所有網路流量均會加密並受限於驗證要求。 如需此程序的詳細資訊，請參閱關於傳遞驗證的[安全性深入探討](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta-security-deep-dive)。

* **使用者體驗**。 若要改善使用者的登入體驗，請部署具有傳遞驗證的無縫 SSO。 在使用者登入之後，無縫 SSO 會排除不必要的提示。

* **進階案例**。 傳遞驗證會在登入時強制執行內部部署帳戶原則。 例如，當內部部署使用者帳戶狀態為停用、遭到鎖定、[密碼過期](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta-faq#what-happens-if-my-users-password-has-expired-and-they-try-to-sign-in-by-using-pass-through-authentication)或是在允許使用者登入的時間之外登入時，系統會拒絕您的存取。 

    需要多重要素驗證與傳遞驗證的組織，必須使用 Azure Multi-Factor Authentication (MFA)。 這些組織不能使用第三方或內部部署多重要素驗證方法。 無論您是否選擇了傳遞驗證，都需要部署密碼雜湊同步處理，才能使用進階功能。 例如 Identity Protection 的認證外洩報表。

* **商務持續性**。 我們建議您部署兩個額外的傳遞驗證代理程式。 它們是 Azure AD Connect 伺服器上第一個代理程式以外的額外項目。 這種額外部署可確保驗證要求的高可用性。 當您部署三個代理程式時，其中一個代理程式仍可在另一個代理程式關閉以進行維護時失敗。 

    除了傳遞驗證以外再部署密碼雜湊同步處理，還有另一個優點。 它可以在主要驗證方法無法使用時，當作備用驗證方法。

* **考量**。 當代理程式因為發生重大的內部部署失敗而無法驗證使用者的認證時，您可以使用密碼雜湊同步處理作為傳遞驗證的備用驗證方法。 容錯移轉至密碼雜湊同步處理的動作不會自動發生，您必須使用 Azure AD Connect 手動切換登入方法。 

    關於傳遞驗證 (包括替代識別碼支援) 的其他考量，請參閱[常見問題集](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta-faq)。

如需部署步驟，請參閱[實作傳遞驗證](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta)。

### <a name="federated-authentication"></a>同盟驗證

* **投入量**。 使用同盟驗證系統會依賴外部信任系統來驗證使用者。 某些公司想要搭配 Azure AD 混合式身分識別解決方案，重複使用他們現有的同盟系統投資。 同盟系統的維護和管理已超出 Azure AD 的控制範圍。 這有賴於組織使用同盟系統，確保同盟驗證系統會安全地部署並可處理驗證負載。 

* **使用者體驗**。 同盟驗證的使用者體驗取決於功能的實作、拓撲以及同盟伺服器陣列的組態。 某些組織需要這種彈性來調整和設定對於同盟伺服器陣列的存取，以符合他們的安全性需求。 例如，您可將內部連線的使用者和裝置設定為自動登入使用者，而不會提示他們提供認證。 這個組態可以運作是因為他們已登入其裝置。 若有需要，某些進階安全性功能可讓使用者的登入程序變得更加困難。

* **進階案例**。 當客戶有 Azure AD 原生不支援的驗證需求時，通常需要同盟驗證解決方案。 請參閱詳細資訊，協助您[選擇正確的登入選項](https://blogs.msdn.microsoft.com/samueld/2017/06/13/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365/)。 請細想下列常見需求：

    * 需要智慧卡或憑證的驗證。
    * 內部部署 MFA Server 或第三方多重要素驗證提供者。
    * 使用第三方驗證解決方案進行驗證。 請參閱 [Azure AD 同盟相容性清單](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-fed-compatibility)。
    * 需要 sAMAccountName (例如網域\使用者名稱)，而不是使用者主體名稱 (UPN) (例如 user@domain.com) 的登入。

* **商務持續性**。 同盟系統通常需要經過負載平衡的伺服器陣列，稱為伺服器陣列。 這個伺服器陣列是在內部網路以及周邊網路拓撲中設定，以確保驗證要求的高可用性。

    當無法再使用主要驗證方法時，請與同盟驗證一起部署密碼雜湊同步處理，作為備用驗證方法。 例如，當內部部署伺服器無法使用時。 某些大型企業組織需要同盟解決方案，以支援針對低度延遲的驗證要求使用地理 DNS 設定的多個網際網路輸入點。

* **考量**。 同盟系統通常需要對內部部署基礎結構進行更大量的投資。 大部分的組織如果已經有內部部署同盟投資，會選擇此選項。 若必須使用單一身分識別提供者，也會選擇此選項。 相較於雲端驗證解決方案，同盟在操作和疑難排解方面更為複雜。

針對無法在 Azure AD 中進行驗證且無法路由傳送的網域，您需要額外的組態才能實作使用者識別碼登入。 此需求稱為替代登入識別碼支援。 如需限制和需求，請參閱[設定替代登入識別碼](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configuring-alternate-login-id) \(機器翻譯\)。 如果您選擇使用同盟的第三方多重要素驗證提供者，請確定提供者支援 WS-Trust 以允許裝置加入 Azure AD。

如需部署步驟，請參閱[部署同盟服務](https://docs.microsoft.com/windows-server/identity/ad-fs/deployment/deploying-federation-servers)。

> [!NOTE] 
> 當您部署 Azure AD 混合式身分識別解決方案時，您必須實作其中一個支援的 Azure AD Connect 拓撲。 若要深入了解支援與不支援的設定，請參閱 [Azure AD Connect 的拓撲](https://docs.microsoft.com/azure/active-directory/hybrid/plan-connect-topologies)。

## <a name="architecture-diagrams"></a>架構圖

下圖概述每個驗證方法所需的高階架構元件，您可以將其與 Azure AD 混合式身分識別解決方案搭配使用。 它們提供概觀，協助您比較解決方案之間的差異。

* 密碼雜湊同步處理解決方案的簡潔度：

    ![Azure AD 混合式身分識別與密碼雜湊同步處理](media/azure-ad/azure-ad-authn-image2.png)

* 傳遞驗證的代理程式需求：

    ![Azure AD 混合式身分識別與傳遞驗證](media/azure-ad/azure-ad-authn-image3.png)

* 在組織的周邊與內部網路中同盟所需的元件：

    ![Azure AD 混合式身分識別與同盟驗證](media/azure-ad/azure-ad-authn-image4.png)

## <a name="comparing-methods"></a>比較方法

|考量|密碼雜湊同步處理 + 無縫 SSO|傳遞驗證 + 無縫 SSO|與 AD FS 同盟|
|:-----|:-----|:-----|:-----|
|驗證的發生位置？|在雲端|在雲端中，安全密碼驗證與內部部署驗證代理程式交換之後|內部部署|
|高於佈建系統的內部部署伺服器需求是什麼：Azure AD Connect？|None|每個額外的驗證代理程式需要 1 部伺服器|2 部以上的 AD FS 伺服器<br><br>周邊/DMZ 網路中需要 2 部以上的 WAP 伺服器|
|內部部署網際網路和網路功能除了佈建系統以外，還有哪些需求？|None|來自執行驗證代理程式之伺服器的[輸出網際網路存取](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta-quick-start)|對周邊 WAP 伺服器的[輸入網際網路存取](https://docs.microsoft.com/windows-server/identity/ad-fs/overview/ad-fs-requirements)<br><br>來自周邊 WAP 伺服器對 AD FS 伺服器的輸入網際網路存取<br><br>網路負載平衡|
|是否有 SSL 憑證需求？|否|否|是|
|是否有健康情況監視解決方案？|不需要|[Azure Active Directory 系統管理中心](https://docs.microsoft.com/azure/active-directory/hybrid/tshoot-connect-pass-through-authentication)提供的代理程式狀態|[Azure AD Connect Health](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-health-adfs)|
|使用者是否可以從公司網路中已加入網域的裝置中取得雲端資源的單一登入？|是，使用[無縫 SSO](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso)|是，使用[無縫 SSO](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso)|是|
|支援何種登入類型？|UserPrincipalName + 密碼<br><br>使用[無縫 SSO](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso) 的 Windows 整合式驗證<br><br>[替代登入識別碼](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-install-custom)|UserPrincipalName + 密碼<br><br>使用[無縫 SSO](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso) 的 Windows 整合式驗證<br><br>[替代登入識別碼](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-pta-faq)|UserPrincipalName + 密碼<br><br>sAMAccountName + 密碼<br><br>Windows 整合式驗證<br><br>[憑證和智慧卡驗證](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[替代登入識別碼](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|是否支援 Windows Hello 企業版？|[金鑰信任模型](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[使用 Intune 的憑證信任模型](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[金鑰信任模型](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[使用 Intune 的憑證信任模型](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[金鑰信任模型](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[憑證信任模型](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|多重要素驗證選項有哪些？|[Azure MFA](https://docs.microsoft.com/azure/multi-factor-authentication/)<br><br>[條件式存取的自訂控制項*](https://docs.microsoft.com/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](https://docs.microsoft.com/azure/multi-factor-authentication/)<br><br>[條件式存取的自訂控制項*](https://docs.microsoft.com/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](https://docs.microsoft.com/azure/multi-factor-authentication/)<br><br>[Azure MFA Server](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfaserver-deploy)<br><br>[第三方 MFA](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[條件式存取的自訂控制項*](https://docs.microsoft.com/azure/active-directory/conditional-access/controls#custom-controls-1)|
|支援哪些使用者帳戶狀態？|停用的帳戶<br>(最多 30 分鐘的延遲)|停用的帳戶<br><br>帳戶已鎖定<br><br>帳戶已過期<br><br>密碼已過期<br><br>登入時數|停用的帳戶<br><br>帳戶已鎖定<br><br>帳戶已過期<br><br>密碼已過期<br><br>登入時數|
|條件式存取選項有哪些？|[Azure AD 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)|[Azure AD 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)|[Azure AD 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)<br><br>[AD FS 宣告規則](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|是否支援封鎖舊版通訊協定？|[是](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[是](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[是](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|您是否可以自訂登入頁面上的標誌、影像和說明？|[是，使用 Azure AD Premium](https://docs.microsoft.com/azure/active-directory/customize-branding)|[是，使用 Azure AD Premium](https://docs.microsoft.com/azure/active-directory/customize-branding)|[是](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-federation-management#customlogo)|
|支援哪些進階案例？|[智慧型密碼鎖定](https://docs.microsoft.com/azure/active-directory/active-directory-secure-passwords)<br><br>[認證外洩報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-risk-events)|[智慧型密碼鎖定](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-smart-lockout)|多網站低延遲驗證系統<br><br>[AD FS 外部網路鎖定](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[與第三方身分識別系統整合](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-federation-compatibility)|

> [!NOTE] 
> Azure AD 條件式存取中的自訂控制項目前不支援裝置註冊。

## <a name="recommendations"></a>建議
您的身分識別系統可確保使用者能夠存取您移轉的雲端應用程式，以及企業營運應用程式，並使其可在雲端使用。 為了讓授權的使用者保有生產力，並讓有心人士遠離組織的敏感性資料，驗證會控制對應用程式的存取。

不論您選擇了何種驗證方法，請基於下列因素使用或啟用密碼雜湊同步處理：

1. **高可用性和災害復原**。 傳遞驗證和同盟均依賴內部部署基礎結構。 針對傳遞驗證，內部部署使用量包含傳遞驗證代理程式所需的伺服器硬體和網路。 針對同盟，內部部署使用量甚至更大。 它需要您周邊網路中的伺服器，來為驗證要求和內部同盟伺服器設定 Proxy。 

    若要避免發生單一失敗點，請部署備援伺服器。 這樣就算任何元件失敗，也還可以持續對驗證要求提供服務。 傳遞驗證和同盟都仰賴網域控制站以回應驗證要求，這也可能會失敗。 這些元件中有許多都可能需要維護，以維持良好狀況。 若未規劃維護並正確實作，就更有可能會中斷。 請使用密碼雜湊同步處理來避免中斷，因為 Microsoft Azure AD 雲端驗證服務會進行全域調整且永遠可供使用。

2. **內部部署中斷存續**。  因為發生網路攻擊或災害而導致的內部部署中斷後果可能很嚴重，包括品牌聲譽受損到因無法處理攻擊而使組織癱瘓。 最近有許多組織成為惡意程式碼攻擊的受害者，包括可使其內部部署伺服器當機的目標勒索軟體。 當 Microsoft 協助客戶處理這些類型的攻擊時，注意到兩類組織：

   * 先前開啟密碼雜湊同步處理的組織，會將其驗證方法變更為使用密碼雜湊同步處理。 這些組織只花了幾個小時就回到線上。 他們透過使用 Office 365 來存取電子郵件，不但解決了問題，還能存取其他雲端型工作負載。

   * 另一類是先前未啟用密碼雜湊同步處理的組織，此時他們必須求助於未受信任的外部消費者電子郵件系統，以進行通訊並解決問題。 在那些情況下，他們需要數週或更長的時間，才能再次上線並執行。

3. **身分識別保護**。 Azure AD Identity Protection 是保護雲端使用者的最佳方式之一。 Microsoft 會針對使用者和密碼清單持續掃描網際網路，以防範有心人士在暗網上進行銷售和提供。 Azure AD 可以使用此資訊來驗證組織內的任何使用者名稱和密碼是否遭到洩漏。 因此，無論您使用的是何種驗證方法 (同盟或傳遞驗證)，請務必啟用密碼雜湊同步處理。 系統會以報表方式呈現認證外洩。 請使用此資訊，在使用者嘗試使用外洩的密碼登入時，封鎖或強制使用者變更其密碼。

最後，根據 [Gartner](https://info.microsoft.com/landingIAMGartnerreportregistration.html)，Microsoft 具備一套最完整的身分識別和存取管理功能。 Microsoft 每個月會處理 [4500 億個驗證要求](https://www.microsoft.com/en-us/security/intelligence-report)，可提供從將近所有裝置對數千個 SaaS 應用程式 (例如 Office 365) 的存取權。 

## <a name="conclusion"></a>結論

本文概述了組織可用來設定並部署，以支援存取雲端應用程式的各種驗證選項。 為了符合各種商務、安全性和技術需求，組織可以在密碼雜湊同步處理、傳遞驗證和同盟等驗證方式之間進行選擇。 

請考量各種驗證方法。 部署解決方案的投入量，以及使用者的登入程序體驗，是否解決了您的商務需求？ 請評估貴組織是否需要每個驗證方法的進階案例，以及商務持續性功能。 最後，請評估各種驗證方法的考量事項。 其中是否有任何項目會防止您實作您的選擇？

## <a name="next-steps"></a>後續步驟

在現今的世界中，威脅時時刻刻、無所不在。 實作正確的驗證方法將可協助降低安全性風險，並保護您的身分識別。

[開始使用](https://docs.microsoft.com/azure/active-directory/get-started-azure-ad) Azure AD，並為組織部署正確的驗證解決方案。

如果您正考慮從同盟移轉到雲端驗證，請深入了解[變更登入方法](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-user-signin#changing-the-user-sign-in-method)。 為了協助您規劃並實作移轉，您可以使用[這些專案部署計劃](http://aka.ms/deploymentplans)。
