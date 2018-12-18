---
title: 使用應用程式 Proxy 進行單一登入 | Microsoft Docs
description: 涵蓋如何使用 Azure AD 應用程式 Proxy 提供單一登入。
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
ms.date: 05/24/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: H1Hack27Feb2017, it-pro
ms.openlocfilehash: 4ce76f1156d4d8d85f5e10bb750b012f93ba7afb
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39366675"
---
# <a name="kerberos-constrained-delegation-for-single-sign-on-to-your-apps-with-application-proxy"></a>可供使用應用程式 Proxy 單一登入應用程式的 Kerberos 限制委派

您可以針對透過應用程式 Proxy 發佈並使用整合式 Windows 驗證保護的內部部署應用程式，提供單一登入。 這些應用程式需要 Kerberos 票證以進行存取。 應用程式 Proxy 會使用 Kerberos 限制委派 (KCD) 來支援這些應用程式。 

您可以在 Active Directory 中提供應用程式 Proxy 連接器權限來模擬使用者，以使用「整合式 Windows 驗證」(IWA) 啟用應用程式的單一登入。 連接器會使用此權限來代表其傳送和接收權杖。

## <a name="how-single-sign-on-with-kcd-works"></a>使用 KCD 單一登入的運作方式
此圖表說明使用者嘗試存取採用 IWA 之內部部署應用程式時的流程。

![Microsoft AAD 驗證流程圖](./media/application-proxy-configure-single-sign-on-with-kcd/AuthDiagram.png)

1. 使用者輸入 URL，以透過「應用程式 Proxy」存取內部部署應用程式。
2. 「應用程式 Proxy」將要求重新導向至 Azure AD 驗證服務，以進行預先驗證。 此時，Azure AD 會套用任何適用的驗證和授權原則，例如多重要素驗證。 若使用者通過驗證，Azure AD 會建立權杖並將它傳送給使用者。
3. 使用者將權杖傳遞給「應用程式 Proxy」。
4. 「應用程式 Proxy」會驗證權杖並從中擷取「使用者主體名稱」(UPN)，然後再透過雙重驗證安全通道，將要求、UPN 和「服務主體名稱」(SPN) 傳送至「連接器」。
5. 「連接器」會與內部部署 AD 執行「Kerberos 限制委派」(KCD) 交涉，以模擬使用者將 Kerberos 權杖傳送至應用程式。
6. Active Directory 會將應用程式的 Kerberos 權杖傳送至「連接器」。
7. 「連接器」會使用從 AD 接收的 Kerberos 權杖，將原始要求傳送至應用程式伺服器。
8. 應用程式會傳送回應至「連接器」，然後再傳回至「應用程式 Proxy」服務，最後再傳回給使用者。

## <a name="prerequisites"></a>必要條件
開始使用 IWA 應用程式的單一登入之前，請確定您的環境已完成下列設定和組態︰

* 您的應用程式 (例如 SharePoint Web 應用程式) 已設為使用「整合式 Windows 驗證」。 如需詳細資訊，請參閱[啟用支援 Kerberos 驗證](https://technet.microsoft.com/library/dd759186.aspx)，或者若是使用 SharePoint，請參閱[為 SharePoint 2013 中的 Kerberos 驗證做規劃](https://technet.microsoft.com/library/ee806870.aspx)。
* 您的所有應用程式都有[服務主體名稱](https://social.technet.microsoft.com/wiki/contents/articles/717.service-principal-names-spns-setspn-syntax-setspn-exe.aspx)。
* 執行「連接器」的伺服器與執行應用程式的伺服器，皆已加入網域且屬於相同網域或信任網域。 如需有關加入網域的詳細資訊，請參閱 [將電腦加入網域](https://technet.microsoft.com/library/dd807102.aspx)。
* 執行「連接器」的伺服器有權限讀取使用者的 TokenGroupsGlobalAndUniversal 屬性。 這個預設設定可能已受到環境強化安全性所影響。

### <a name="configure-active-directory"></a>設定 Active Directory
根據您的「應用程式 Proxy 連接器」和應用程式伺服器是否位於相同的網域，Active Directory 組態會有所不同。

#### <a name="connector-and-application-server-in-the-same-domain"></a>連接器和應用程式伺服器位於相同網域
1. 在 Active Directory 中，移至 [工具] > [使用者和電腦]。
2. 選取正在執行連接器的伺服器。
3. 按一下滑鼠右鍵，然後選取 [屬性] > [委派]。
4. 選取 [信任這台電腦，但只委派指定的服務]。 
5. 在 [這個帳戶可以呈送委派認證的服務] 下方，新增應用程式伺服器的 SPN 身分識別值。 這可讓「應用程式 Proxy 連接器」針對清單中所定義的應用程式，在 AD 中模擬使用者。

   ![[連接器 SVR 屬性] 視窗螢幕擷取畫面](./media/application-proxy-configure-single-sign-on-with-kcd/Properties.jpg)

#### <a name="connector-and-application-server-in-different-domains"></a>連接器和應用程式伺服器位於不同網域
1. 如需跨網域使用 KCD 的先決條件清單，請參閱 [跨網域的 Kerberos 限制委派](https://technet.microsoft.com/library/hh831477.aspx)。
2. 使用「連接器」伺服器的 `principalsallowedtodelegateto` 屬性，讓應用程式 Proxy 能夠委派連接器伺服器。 應用程式伺服器為 `sharepointserviceaccount`，而委派的伺服器為 `connectormachineaccount`。 在 Windows 2012 R2 中，以此程式碼作為範例：

        $connector= Get-ADComputer -Identity connectormachineaccount -server dc.connectordomain.com

        Set-ADComputer -Identity sharepointserviceaccount -PrincipalsAllowedToDelegateToAccount $connector

        Get-ADComputer sharepointserviceaccount -Properties PrincipalsAllowedToDelegateToAccount

Sharepointserviceaccount 可以是 SPS 電腦帳戶，或是用來執行 SPS 應用程式集區的服務帳戶。

## <a name="configure-single-sign-on"></a>設定單一登入 
1. 根據 [使用應用程式 Proxy 發佈應用程式](application-proxy-publish-azure-portal.md)中的所述指示來發佈您的應用程式。 請務必選取 [Azure Active Directory] 作為 [預先驗證方法]。
2. 應用程式出現於企業應用程式清單後，將其選取並按一下 [單一登入]。
3. 將單一登入模式設定為 [整合式 Windows 驗證]。  
4. 輸入應用程式伺服器的 [內部應用程式 SPN]  。 在此範例中，已發佈應用程式的 SPN 為 http/www.contoso.com。 此 SPN 必須在連接器可以呈送委派認證的服務清單中。 
5. 針對要代表使用者使用的連接器選擇 [委派的登入身分識別]。 如需詳細資訊，請參閱[使用不同的內部部署和雲端身分識別](#Working-with-different-on-premises-and-cloud-identities)

   ![進階應用程式組態](./media/application-proxy-configure-single-sign-on-with-kcd/cwap_auth2.png)  


## <a name="sso-for-non-windows-apps"></a>非 Windows 應用程式的 SSO

Azure AD 應用程式 Proxy 的 Kerberos 委派流程會在 Azure AD 在雲端驗證使用者開始。 一旦要求到達內部部署，Azure AD 應用程式 Proxy 連接器會利用與本機 Active Directory 互動，代表使用者發出 Kerberos 票證。 此程序稱為「Kerberos 限制委派 (KCD)」。 在下一個階段中，要求會傳送至具有此 Kerberos 票證的後端應用程式。 

有許多定義如何傳送這類要求的通訊協定。 預期大部分的非 Windows 伺服器都會與 SPNEGO 進行交涉。 Azure AD 應用程式 Proxy 支援此通訊協定，但預設為停用。 您可將伺服器設定為 SPNEGO 或標準 KCD ，但無法同時設定為兩者。

如果您為 SPNEGO 設定連接器電腦，請確定該連接器群組中的所有其他連接器也都已採用 SPNEGO 進行設定。 預定採用標準 KCD 的應用程式則應透過其他並非針對 SPNEGO 進行設定的連接器路由傳送。
 

若要啟用 SPNEGO：

1. 開啟以系統管理員身分執行的命令提示字元。
2. 透過命令提示字元，在需要 SPNEGO 的連接器伺服器中執行下列命令。

    ```
    REG ADD "HKLM\SOFTWARE\Microsoft\Microsoft AAD App Proxy Connector" /v UseSpnegoAuthentication /t REG_DWORD /d 1
    net stop WAPCSvc & net start WAPCSvc
    ```

如需有關 Kerberos 的詳細資訊，請參閱[有關 Kerberos 限制委派 (KCD) 您想要知道的一切](https://blogs.technet.microsoft.com/applicationproxyblog/2015/09/21/all-you-want-to-know-about-kerberos-constrained-delegation-kcd)。

非 Windows 應用程式通常會使用使用者名稱或 SAM 帳戶名稱，而不是網域的電子郵件地址。 如果這種情況適用於您的應用程式，就必須設定指定的登入身分識別欄位，將您的雲端身分識別連線到您的應用程式身分識別。 

## <a name="working-with-different-on-premises-and-cloud-identities"></a>使用不同的內部部署和雲端身分識別
應用程式 Proxy 會假設使用者在雲端與內部部署中具有完全相同的身分識別。 如果事實並非如此，您仍然可以使用 KCD 進行單一登入。 為每個應用程式設定 [委派的身分識別登入]，以指定在執行單一登入時所應使用的身分識別。  

此功能可讓具有不同內部部署與雲端身分識別的許多組織，可從雲端單一登入到內部部署應用程式，而不需要使用者輸入不同的使用者名稱與密碼。 這包括下列組織：

* 在內部有多個網域 (joe@us.contoso.com、joe@eu.contoso.com)，並且在雲端有單一網域 (joe@contoso.com)。
* 在內部有無法路由傳送的網域名稱 (joe@contoso.usa)，並且在雲端有合法網域名稱。
* 請勿在內部使用網域名稱 (joe)
* 在內部部署和雲端中使用不同別名。 例如：joe-johns@contoso.com 對上 joej@contoso.com  

使用應用程式 Proxy，您就可以選取要用來取得 Kerberos 票證的身分識別。 這項設定會因應用程式而異。 其中的一些選項適合不接受電子郵件地址格式的系統，另外的選項則設計用於替代登入。

![[委派的登入身分識別] 參數螢幕擷取畫面](./media/application-proxy-configure-single-sign-on-with-kcd/app_proxy_sso_diff_id_upn.png)

如果使用委派的登入身分識別，則組織的所有網域或樹系中的這個值可能不是唯一。 您可以藉由使用兩個不同的連接器群組發佈這些應用程式兩次來避免此問題。 因為每個應用程式有不同的使用者對象，您可以將其「連接器」加入不同的網域。

### <a name="configure-sso-for-different-identities"></a>設定不同身分識別的 SSO
1. 設定 Azure AD Connect 設定，讓主要的身分識別會是電子郵件地址 (郵件)。 這是在自訂程序中完成 (透過變更同步設定中的 [使用者主體名稱] 欄位)。 這些設定也決定使用者如何登入 Office 365、Windows 10 裝置與其他使用 Azure AD 作為其身分識別存放區的應用程式。  
   ![識別使用者螢幕擷取畫面 - [使用者主體名稱] 下拉式清單](./media/application-proxy-configure-single-sign-on-with-kcd/app_proxy_sso_diff_id_connect_settings.png)  
2. 在您想要修改之應用程式的應用程式組態設定中，選取要使用的 [委派的登入識別]  ：

   * 使用者主體名稱 (例如 joe@contoso.com)
   * 替代使用者主體名稱 (例如 joed@contoso.local)
   * 使用者主體名稱的使用者名稱部分 (例如 joe)
   * 替代使用者主體名稱的使用者名稱部分 (例如 joed)
   * 內部部署 SAM 帳戶名稱 (視網域控制站組態而定)

### <a name="troubleshooting-sso-for-different-identities"></a>疑難排解不同身分識別的 SSO
如果 SSO 程序發生錯誤，它會顯示在連接器電腦事件記錄中，如[疑難排解](../application-proxy-back-end-kerberos-constrained-delegation-how-to.md)所述。
但在某些情況下，要求會成功傳送至後端應用程式，同時此應用程式會以各種其他 HTTP 回應來回覆。 疑難排解這些情況應該要從檢查連接器電腦上應用程式 Proxy 工作階段事件記錄中的事件編號 24029 開始。 用於委派的使用者身分識別會出現在事件詳細資料的 [使用者] 欄位內。 若要開啟工作階段記錄，請選取事件檢視器檢視功能表中的 [顯示分析與偵錯記錄檔]  。

## <a name="next-steps"></a>後續步驟

* [如何設定應用程式 Proxy 應用程式以使用 Kerberos 限制委派](../application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
* [使用應用程式 Proxy 疑難排解您遇到的問題](application-proxy-troubleshoot.md)


如需最新消息，請查閱 [應用程式 Proxy 部落格](http://blogs.technet.com/b/applicationproxyblog/)

