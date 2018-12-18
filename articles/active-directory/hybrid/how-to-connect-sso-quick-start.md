---
title: Azure AD Connect：無縫單一登入 - 快速入門 | Microsoft Docs
description: 本文描述如何開始使用 Azure Active Directory 無縫單一登入
services: active-directory
keywords: 何謂 Azure AD Connect、安裝 Active Directory、Azure AD、SSO、單一登入的必要元件
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: d7906323da8e9ee1571efe908084ae1433884e61
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405983"
---
# <a name="azure-active-directory-seamless-single-sign-on-quick-start"></a>Azure Active Directory 無縫單一登入：快速入門

## <a name="deploy-seamless-single-sign-on"></a>部署無縫單一登入

使用者位於連線到公司網路的公司桌上型電腦時，Azure Active Directory (Azure AD) 無縫單一登入 (無縫 SSO) 會自動登入使用者。 無縫 SSO 可讓使用者輕鬆存取雲端式應用程式，而不需要任何額外的內部部署元件。

若要部署無縫 SSO，請遵循下列步驟。

## <a name="step-1-check-the-prerequisites"></a>步驟 1：檢查必要條件

請確保已具備下列必要條件︰

* **設定 Azure AD Connect 伺服器**：如果您使用[傳遞驗證](how-to-connect-pta.md)作為登入方法，不需要進行額外的必要條件檢查。 如果您使用[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)作為登入方法，而且 Azure AD Connect 與 Azure AD 之間有防火牆，請確定︰
   - 您使用 Azure AD Connect 1.1.644.0 或更新版本。 
   - 如果您的防火牆或 Proxy 允許建立 DNS 白名單，便可將透過連接埠 443 進行的 **\*.msappproxy.net** URL 連線加入白名單。 如果不允許建立，請允許存取每週更新的 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。 只有啟用此功能時，此必要條件才適用。 不需要實際的使用者登入。

    >[!NOTE]
    >Azure AD Connect 版本 1.1.557.0、1.1.558.0、1.1.561.0 和 1.1.614.0 具有與密碼雜湊同步處理相關的問題。 如果您_不_想要使用密碼雜湊同步處理搭配傳遞驗證，請閱讀 [Azure AD Connect 版本資訊](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history#116470)，以深入了解。

* **使用支援的 Azure AD Connect 拓撲**：確定您會使用[這裡](plan-connect-topologies.md)所述的 Azure AD Connect 支援技術之一。

    >[!NOTE]
    >無縫 SSO 可支援多個 AD 樹系，無論其間是否有 AD 信任，都是如此。

* **設定網域管理員員認證**：下列情況的每個 Active Directory 樹系，均需擁有網域管理員認證：
    * 透過 Azure AD Connect 同步至 Azure AD。
    * 包含您要啟用無縫 SSO 的使用者。
    
* **啟用新式驗證**：您必須在租用戶上啟用[新式驗證](https://aka.ms/modernauthga)，此功能才能運作。

* **使用最新版的 Office 365 用戶端**：若要使用 Office 365 用戶端 (Outlook、Word、Excel 和其他產品) 來取得無訊息登入體驗，您的使用者需要 16.0.8730.xxxx 版或更新版本。

## <a name="step-2-enable-the-feature"></a>步驟 2︰啟用功能

透過 [Azure AD Connect](whatis-hybrid-identity.md) 啟用無縫 SSO。

如果您要執行 Azure AD Connect 的全新安裝，請選擇[自訂安裝路徑](how-to-connect-install-custom.md)。 在 [使用者登入] 頁面中，選取 [啟用單一登入] 選項。

>[!NOTE]
> 唯有登入方法為 [密碼雜湊同步] 或 [傳遞驗證] 時，此選項才可供選取。

![Azure AD Connect：使用者登入](./media/how-to-connect-sso-quick-start/sso8.png)

如果您已經安裝 Azure AD Connect，請在 Azure AD Connect 上選取 [變更使用者登入] 頁面，然後選取 [下一步]。 如果您使用 Azure AD Connect 1.1.880.0 版或更新版本，則預設會選取 [啟用單一登入] 選項。 如果您使用舊版的 Azure AD Connect，請選取 [啟用單一登入] 選項。

![Azure AD Connect：變更使用者登入](./media/how-to-connect-sso-quick-start/changeusersignin.png)

繼續執行精靈，直到抵達 [啟用單一登入] 頁面。 為下列情況的每個 Active Directory 樹系提供網域管理員認證：
    * 透過 Azure AD Connect 同步至 Azure AD。
    * 包含您要啟用無縫 SSO 的使用者。

完成精靈之後，無縫 SSO 就會在您的租用戶上啟用。

>[!NOTE]
> 網域管理員認證不會儲存在 Azure AD Connect 或 Azure AD 中。 這些認證僅用於啟用此功能。

請遵循下列指示來確認您已正確啟用無縫 SSO：

1. 使用租用戶的全域管理員認證來登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 在左窗格中，選取 [Azure Active Directory]。
3. 選取 [Azure AD Connect]。
4. 確認 [無縫單一登入] 功能顯示為 [已啟用]。

![Azure 入口網站：Azure AD Connect 窗格](./media/how-to-connect-sso-quick-start/sso10.png)

>[!IMPORTANT]
> 無縫 SSO 會在每個 AD 樹系中的內部部署 Active Directory (AD) 中，建立名為 `AZUREADSSOACC` 的電腦帳戶 (代表 Azure AD)。 需要此電腦帳戶才能讓該功能運作。 將 `AZUREADSSOACC` 電腦帳戶移至儲存其他電腦帳戶的組織單位 (OU)，確保它是透過相同方式進行管理，而且不會予以刪除。

## <a name="step-3-roll-out-the-feature"></a>步驟 3：推出功能

您可以使用下面提供的指示，為使用者逐步推出無縫 SSO。 您一開始可以使用 Active Directory 中的群組原則，將下列 Azure AD URL 新增至所有或已選取之使用者的內部網路區域設定：

- https://autologon.microsoftazuread-sso.com

此外，您也需要透過「群組原則」，啟用內部網路區域原則設定，稱為**允許透過指令碼更新狀態列**。 

>[!NOTE]
> 下列指示僅適用於 Windows 上的 Internet Explorer 和 Google Chrome (如果它與 Internet Explorer 共用一組受信任的網站 URL)。 如需如何在 macOS 上設定 Mozilla Firefox 和 Google Chrome 的指示，請閱讀下一節。

### <a name="why-do-you-need-to-modify-users-intranet-zone-settings"></a>為什麼需要修改使用者的內部網路區域設定？

瀏覽器預設會自動從指定的 URL 計算正確的區域 (網際網路或內部網路)。 例如，"http://contoso/" 會對應到內部網路區域，而 "http://intranet.contoso.com/" 則會對應到網際網路區域 (因為 URL 包含句點)。 除非將 URL 明確地新增至瀏覽器的內部網路區域，否則瀏覽器不會將 Kerberos 票證傳送給雲端端點 (例如 Azure AD URL)。

有兩種方式可修改使用者的內部網路區域設定：

| 選項 | 管理考量 | 使用者體驗 |
| --- | --- | --- |
| 群組原則 | 管理員鎖定內部網路區域設定的編輯 | 使用者無法修改自己的設定 |
| 群組原則喜好設定 |  管理員允許在內部網路區域設定上進行編輯 | 使用者可以修改自己的設定 |

### <a name="group-policy-option---detailed-steps"></a>「群組原則」選項 - 詳細步驟

1. 開啟群組原則管理編輯器工具。
2. 編輯套用至部分或所有使用者的群組原則。 此範例使用**預設網域原則**。
3. 瀏覽至 [使用者設定] > [系統管理範本] > [Windows 元件] > [Internet Explorer] > [網際網路控制台] > [安全性頁面]。 然後選取 [指派網站到區域清單]。
    ![單一登入](./media/how-to-connect-sso-quick-start/sso6.png)
4. 啟用原則，然後在對話方塊中輸入下列值：
   - **值名稱**：轉送 Kerberos 票證的 Azure AD URL。
   - **值** (資料)：**1** 表示內部網路區域。

    結果如下所示：

    值名稱： https://autologon.microsoftazuread-sso.com
  
    值 (資料)：1

   >[!NOTE]
   > 如果您想要禁止部分使用者使用無縫 SSO (例如，如果這些使用者在共用 Kiosk 上登入)，請將先前的值設定為 **4**。 此動作會將 Azure AD URL 新增至限制區域，而且隨時讓無縫 SSO 失敗。
   >

5. 選取 [確定]，然後再次選取 [確定]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso7.png)

6. 瀏覽至 [使用者設定] > [系統管理範本] > [Windows 元件] > [Internet Explorer] > [網際網路控制台] > [安全性頁面] > [內部網路區域]。 然後選取 [允許透過指令碼更新狀態列]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso11.png)

7. 啟用原則設定，然後選取 [確定]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso12.png)

### <a name="group-policy-preference-option---detailed-steps"></a>「群組原則喜好設定」選項 - 詳細步驟

1. 開啟群組原則管理編輯器工具。
2. 編輯套用至部分或所有使用者的群組原則。 此範例使用**預設網域原則**。
3. 瀏覽至 [使用者設定] > [喜好設定] > [Windows 設定] > [登錄] > [新增] > [登錄項目]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso15.png)

4. 在適當欄位中輸入下列值，然後按一下 [確定]。
   - **機碼路徑**：***Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\microsoftazuread-sso.com\autologon***
   - **值名稱**：***https***。
   - **值類型**：***REG_DWORD***。
   - **數值資料**︰***00000001***。
 
    ![單一登入](./media/how-to-connect-sso-quick-start/sso16.png)
 
    ![單一登入](./media/how-to-connect-sso-quick-start/sso17.png)

6. 瀏覽至 [使用者設定] > [系統管理範本] > [Windows 元件] > [Internet Explorer] > [網際網路控制台] > [安全性頁面] > [內部網路區域]。 然後選取 [允許透過指令碼更新狀態列]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso11.png)

7. 啟用原則設定，然後選取 [確定]。

    ![單一登入](./media/how-to-connect-sso-quick-start/sso12.png)

### <a name="browser-considerations"></a>瀏覽器考量

#### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (所有平台)

Mozilla Firefox 不會自動使用 Kerberos 驗證。 每個使用者都必須使用下列步驟，手動將 Azure AD URL 新增至其 Firefox 設定：
1. 執行 Firefox，並在網址列中輸入 `about:config`。 關閉任何您看到的通知。
2. 搜尋 **network.negotiate-auth.trusted-uris** 喜好設定。 此喜好設定列出 Firefox 進行 Kerberos 驗證的受信任網站。
3. 按一下滑鼠右鍵，然後選取 [修改]。
4. 在欄位中輸入 https://autologon.microsoftazuread-sso.com。
5. 選取 [確定]，然後重新開啟瀏覽器。

#### <a name="safari-macos"></a>Safari (macOS)

確定執行 macOS 的電腦已加入 AD。 將您的 macOS 裝置加入 AD 的指示並不在本文的討論之列。

#### <a name="google-chrome-all-platforms"></a>Google Chrome (所有平台)

如果您已覆寫環境中的 [AuthNegotiateDelegateWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthNegotiateDelegateWhitelist) \(英文\) 或 [AuthServerWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthServerWhitelist) \(英文\) 原則設定，請確定您也會將 Azure AD 的 URL (https://autologon.microsoftazuread-sso.com) 新增到這些設定。

#### <a name="google-chrome-macos-only"></a>Google Chrome (僅限 macOS)

針對 Mac OS 和其他非 Windows 平台上的 Google Chrome，請參閱 [Chromium 專案原則清單](https://dev.chromium.org/administrators/policy-list-3#AuthServerWhitelist)，以了解如何將 Azure AD URL 設為允許清單以進行整合式驗證的資訊。

使用協力廠商 Active Directory 群組原則延伸模組，向 Mac 使用者上的 Firefox 和 Google Chrome 推出 Azure AD URL，不在本文的範圍內。

#### <a name="known-browser-limitations"></a>已知的瀏覽器限制

無縫 SSO 無法在 Firefox 和 Edge 瀏覽器的私人瀏覽模式中運作。 如果瀏覽器是在「增強保護」模式中執行，它也無法在 Internet Explorer 上運作。

## <a name="step-4-test-the-feature"></a>步驟 4：測試功能

若要測試特定使用者的功能，請確認已具備下列所有條件：
  - 使用者是在公司裝置上登入。
  - 裝置已加入您的 Active Directory 網域。 裝置「不」需要[加入 Azure AD](../active-directory-azureadjoin-overview.md)。
  - 裝置能夠直接連線至網域控制站 (DC)，不論是在公司的有線或無線網路上進行，還是透過遠端存取連線 (例如 VPN 連線) 來進行。
  - 您已透過群組原則，向這位使用者[推出功能](##step-3-roll-out-the-feature)。

測試使用者只輸入使用者名稱而非密碼的案例：
   - 在新的私用瀏覽器工作階段中，登入 https://myapps.microsoft.com/。

若要測試使用者不需要輸入使用者名稱或密碼的案例，請使用下列其中一個步驟： 
   - 在新的私用瀏覽器工作階段中，登入 https://myapps.microsoft.com/contoso.onmicrosoft.com。 使用您的租用戶名稱取代 *contoso*。
   - 在新的私用瀏覽器工作階段中，登入 https://myapps.microsoft.com/contoso.com。 在您的租用戶上，以驗證過的網域 (非同盟網域) 取代 *contoso.com*。

## <a name="step-5-roll-over-keys"></a>步驟 5：變換金鑰

在步驟 2 中，Azure AD Connect 會在您已啟用無縫 SSO 的所有 Active Directory 樹系中建立電腦帳戶 (代表 Azure AD)。 若要深入了解，請參閱 [Azure Active Directory 無縫單一登入：技術深入探討](how-to-connect-sso-how-it-works.md)。

>[!IMPORTANT]
>如果電腦帳戶上的 Kerberos 解密金鑰外洩，則可用來針對其 AD 樹系中的任何使用者產生 Kerberos 票證。 惡意執行者接著可針對遭到入侵的使用者模擬行 Azure AD 登入。 強烈建議您定期變換這些 Kerberos 解密金鑰 (至少每隔 30 天一次)。

如需如何變換金鑰的指示，請參閱 [Azure Active Directory 無縫單一登入：常見問題集](how-to-connect-sso-faq.md#how-can-i-roll-over-the-kerberos-decryption-key-of-the-azureadssoacc-computer-account)。 我們正致力於引進自動變換金鑰的功能。

>[!IMPORTANT]
>您不需要在啟用此功能後「立即」執行此步驟。 至少每隔 30 天變換一次 Kerberos 解密金鑰。

## <a name="next-steps"></a>後續步驟

- [技術深入探討](how-to-connect-sso-how-it-works.md)：了解無縫單一登入功能的運作方式。
- [常見問題集](how-to-connect-sso-faq.md)：取得無縫單一登入常見問題集的答案。
- [疑難排解](tshoot-connect-sso.md)：了解如何解決無縫單一登入功能的常見問題。
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect)：使用 Azure Active Directory 論壇提出新功能要求。
