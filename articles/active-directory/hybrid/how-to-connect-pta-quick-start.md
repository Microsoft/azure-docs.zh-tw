---
title: Azure AD 傳遞驗證 - 快速入門 | Microsoft Docs
description: 本文說明如何開始使用 Azure Active Directory (Azure AD) 傳遞驗證。
services: active-directory
keywords: Azure AD Connect 傳遞驗證, 安裝 Active Directory, Azure AD 的必要元件, SSO, 單一登入
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 8b45acebf95d5bf24ff2045f5739c8584f374842
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320453"
---
# <a name="azure-active-directory-pass-through-authentication-quick-start"></a>Azure Active Directory 傳遞驗證：快速入門

## <a name="deploy-azure-ad-pass-through-authentication"></a>部署 Azure AD 傳遞驗證

Azure Active Directory (Azure AD) 傳遞驗證可讓您的使用者以相同密碼登入內部部署和雲端式應用程式。 傳遞驗證會直接向內部部署 Active Directory 驗證使用者的密碼，以決定是否讓使用者登入。

>[!IMPORTANT]
>如果您要從 AD FS (或其他同盟技術) 遷移至傳遞驗證，強烈建議您遵循我們在[此處](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx) \(英文\) 發佈的詳細部署指南。

請依照下列指示，在您的租用戶上部署傳遞驗證：

## <a name="step-1-check-the-prerequisites"></a>步驟 1：檢查必要條件

請確保已具備下列必要條件。

### <a name="in-the-azure-active-directory-admin-center"></a>於 Azure Active Directory 管理中心

1. 在 Azure AD 租用戶上建立僅限雲端的全域管理員帳戶。 如此一來，如果您的內部部署服務失敗或無法使用，您便可以管理租用戶組態。 了解如何[新增僅限雲端管理員帳戶](../active-directory-users-create-azure-portal.md)。 這是確保您不會遭租用戶封鎖的關鍵步驟。
2. 將一或多個[自訂網域名稱](../active-directory-domains-add-azure-portal.md)新增至 Azure AD 租用戶。 您的使用者可以使用其中一個網域名稱登入。

### <a name="in-your-on-premises-environment"></a>在內部部署環境中

1. 識別一部執行 Windows Server 2012 R2 或更新版本的伺服器來執行 Azure AD Connect。 根據需要驗證密碼之使用者所在的 Active Directory 樹系，將伺服器新增至同一個樹系。
2. 在上一個步驟中識別的伺服器上，安裝[最新版本的 Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594)。 如果您已執行 Azure AD Connect，請確定版本是 1.1.750.0 或更新版本。

    >[!NOTE]
    >Azure AD Connect 版本 1.1.557.0、1.1.558.0、1.1.561.0 和 1.1.614.0 具有與密碼雜湊同步處理相關的問題。 如果您「不」想要使用密碼雜湊同步處理搭配傳遞驗證，請閱讀 [Azure AD Connect 版本資訊](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-version-history#116470)。

3. 識別一或多部額外的伺服器 (執行 Windows Server 2012 R2 或更新版本) 來執行獨立驗證代理程式。 需要有這些額外的伺服器，才能確保登入要求的高可用性。 根據需要驗證密碼之使用者所在的 Active Directory 樹系，將伺服器新增至同一個樹系。

    >[!IMPORTANT]
    >在生產環境中，我們建議至少要有 3 個驗證代理程式在您的租用戶上執行。 系統限制每個租用戶只能有 12 個驗證代理程式。 因此，最佳做法是將執行驗證代理程式的所有伺服器視為階層 0 的系統 (請參閱[參考](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) \(機器翻譯\))。

4. 如果您的伺服器和 Azure AD 之間有防火牆，請設定下列項目：
   - 確定驗證代理程式會透過以下連接埠對 Azure AD 提出*輸出*要求：

    | 連接埠號碼 | 使用方式 |
    | --- | --- |
    | **80** | 驗證 SSL 憑證時下載憑證撤銷清單 (CRL) |
    | **443** | 處理所有與服務之間的輸出通訊 |
    | **8080** (選擇性) | 如果無法使用連接埠 443，則驗證代理程式會透過連接埠 8080 每隔十分鐘報告其狀態。 此狀態會顯示在 Azure 入口網站上。 連接埠 8080 「不」會用於使用者登入。 |

    如果您的防火牆會根據原始使用者強制執行規則，請開啟這些連接埠，讓來自以網路服務形式執行之 Windows 服務的流量得以通行。
   - 如果您的防火牆或 Proxy 允許建立 DNS 允許清單，便可將對 **\*.msappproxy.net** 與 **\*.servicebus.windows.net** 的連線加入允許清單。 如果不允許建立，請允許存取每週更新的 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。
   - 您的驗證代理程式必須存取 **login.windows.net** 與 **login.microsoftonline.com**才能進行初始註冊， 因此也請針對這些 URL 開啟您的防火牆。
   - 為了驗證憑證，請解除封鎖以下 URL：**mscrl.microsoft.com:80**、**crl.microsoft.com:80**、**ocsp.msocsp.com:80** 和 **www.microsoft.com:80**。 由於這些 URL 會用於其他 Microsoft 產品的憑證驗證，因此您可能已將這些 URL 解除封鎖。

## <a name="step-2-enable-the-feature"></a>步驟 2︰啟用功能

透過 [Azure AD Connect](whatis-hybrid-identity.md) 啟用傳遞驗證。

>[!IMPORTANT]
>您可以在 Azure AD Connect 主要或暫存伺服器上啟用傳遞驗證。 強烈建議您從主要伺服器啟用此功能。 如果您未來要設定 Azure AD Connect 預備伺服器，您**必須**繼續選擇傳遞驗證做為登入選項；選擇另一個選項將會**停用**租用戶上的傳遞驗證，並覆寫主要伺服器中的設定。

如果您是第一次安裝 Azure AD Connect，請選擇[自訂安裝路徑](how-to-connect-install-custom.md)。 在 [使用者登入] 頁面上，選擇 [傳遞驗證] 作為 [登入方法]。 成功完成時，傳遞驗證代理程式會安裝在 Azure AD Connect 所在的同一部伺服器上。 此外，您的租用戶上也會啟用傳遞驗證功能。

![Azure AD Connect：使用者登入](./media/how-to-connect-pta-quick-start/sso3.png)

如果您已安裝 Azure AD Connect (使用[快速安裝](how-to-connect-install-express.md)或[自訂安裝](how-to-connect-install-custom.md)路徑)，請在 Azure AD Connect 上選取 [變更使用者登入] 工作，並選取 [下一步]。 然後選取 [傳遞驗證] 作為登入方法。 成功完成時，傳遞驗證代理程式會安裝在 Azure AD Connect 所在的同一部伺服器上，且您的租用戶上會啟用此功能。

![Azure AD Connect：變更使用者登入](./media/how-to-connect-pta-quick-start/changeusersignin.png)

>[!IMPORTANT]
>傳遞驗證是租用戶層級的功能。 開啟此功能會影響租用戶中「所有」受控網域的使用者登入。 如果您從 Active Directory Federation Services (AD FS) 改為使用傳遞驗證，應等候至少 12 個小時再關閉 AD FS 基礎結構。 這裡的等候時間可確保使用者在轉換期間依然可以繼續登入 Exchange ActiveSync。 如需從 AD FS 遷移到傳遞驗證的詳細說明，請查看我們在[此處](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx) \(英文\) 發佈的詳細部署指南。

## <a name="step-3-test-the-feature"></a>步驟 3：測試功能

請遵循下列指示來確認您已正確啟用傳遞驗證：

1. 使用租用戶的全域管理員認證來登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 在左窗格中，選取 [Azure Active Directory]。
3. 選取 [Azure AD Connect]。
4. 確認 [傳遞驗證] 功能顯示為 [已啟用]。
5. 選取 [傳遞驗證]。 [傳遞驗證] 窗格會列出安裝驗證代理程式的伺服器。

![Azure Active Directory 管理中心：Azure AD Connect 窗格](./media/how-to-connect-pta-quick-start/pta7.png)

![Azure Active Directory 管理中心：傳遞驗證窗格](./media/how-to-connect-pta-quick-start/pta8.png)

在此階段，租用戶中所有受控網域的使用者都可以使用傳遞驗證來登入。 不過，同盟網域的使用者會繼續使用 ADFS 或您先前設定的其他同盟提供者來登入。 如果您將同盟網域轉換成受控網域，該網域中的所有使用者都會自動開始使用傳遞驗證來登入。 傳遞驗證功能不會影響僅限雲端的使用者。

## <a name="step-4-ensure-high-availability"></a>步驟 4：確保高可用性

如果您打算在生產環境中部署傳遞驗證，您應該安裝額外的獨立驗證代理程式。 請在執行 Azure AD Connect「以外」的伺服器上安裝這些「驗證代理程式」。 此設定可提供高可用性來滿足使用者登入要求。

>[!IMPORTANT]
>在生產環境中，我們建議至少要有 3 個驗證代理程式在您的租用戶上執行。 系統限制每個租用戶只能有 12 個驗證代理程式。 因此，最佳做法是將執行驗證代理程式的所有伺服器視為階層 0 的系統 (請參閱[參考](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) \(機器翻譯\))。

請依照下列指示來下載「驗證代理程式」軟體：

1. 若要下載最新版「驗證代理程式」(1.5.193.0 版或更新版本)，請使用您租用戶的全域管理員認證來登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 在左窗格中，選取 [Azure Active Directory]。
3. 依序選取 [Azure AD Connect]、[傳遞驗證] 及 [下載代理程式]。
4. 選取 [接受條款並下載] 按鈕。

![Azure Active Directory 管理中心：下載驗證代理程式按鈕](./media/how-to-connect-pta-quick-start/pta9.png)

![Azure Active Directory 管理中心：下載代理程式窗格](./media/how-to-connect-pta-quick-start/pta10.png)

>[!NOTE]
>您也可以直接[下載驗證代理程式軟體](https://aka.ms/getauthagent)。 請在安裝「驗證代理程式」_之前_，先檢閱並接受「驗證代理程式」的[服務條款](https://aka.ms/authagenteula) \(英文\)。

部署獨立「驗證代理程式」的方式有兩種：

第一種，您可以直接執行所下載的「驗證代理程式」可執行檔，並在出現提示時提供您租用戶的全域管理員認證，以互動方式進行部署。

第二種，您可以建立並執行自動部署指令碼。 當您想要一次部署多個「驗證代理程式」，或是在未啟用使用者介面或您無法使用「遠端桌面」來存取的 Windows 伺服器上安裝「驗證代理程式」時，這會相當有用。 以下是有關如何使用此方法的指示：

1. 執行下列命令來安裝「驗證代理程式」：`AADConnectAuthAgentSetup.exe REGISTERCONNECTOR="false" /q`。
2. 您可以使用 Windows PowerShell 來向我們的服務註冊「驗證代理程式」。 建立 PowerShell 認證物件 `$cred`，其中含有租用戶的全域管理員使用者名稱和密碼。 執行下列命令，取代 *\<使用者名稱\>* 和*\<密碼\>*：

        $User = "<username>"
        $PlainPassword = '<password>'
        $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
        $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $User, $SecurePassword
3. 移至 **C:\Program Files\Microsoft Azure AD Connect 驗證代理程式**，然後使用您建立的 `$cred` 物件來執行下列指令碼：

        RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft Azure AD Connect Authentication Agent\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature PassthroughAuthentication

## <a name="next-steps"></a>後續步驟
- [從 AD FS 遷移到傳遞驗證](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx) \(英文\) - 從 AD FS (或其他同盟技術) 遷移到傳遞驗證的詳細指南。
- [智慧鎖定](../authentication/howto-password-smart-lockout.md)：了解如何在租用戶中設定智慧鎖定功能以保護使用者帳戶。
- [目前的限制](how-to-connect-pta-current-limitations.md)：了解傳遞驗證支援的情節和不支援的情節。
- [技術深入探討](how-to-connect-pta-how-it-works.md)：了解傳遞驗證功能的運作方式。
- [常見問題集](how-to-connect-pta-faq.md)：常見問題集的答案。
- [疑難排解](tshoot-connect-pass-through-authentication.md)：了解如何解決傳遞驗證功能的常見問題。
- [安全性深入探討](how-to-connect-pta-security-deep-dive.md)：取得傳遞驗證功能的技術資訊。
- [Azure AD 無縫 SSO](how-to-connect-sso.md)：深入了解此互補功能。
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect)：使用 Azure Active Directory 論壇提出新功能要求。
