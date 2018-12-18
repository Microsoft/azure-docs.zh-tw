---
title: Azure Active Directory 傳遞驗證安全性深入探討 | Microsoft Docs
description: 本文說明 Azure Active Directory (Azure AD) 傳遞驗證如何保護您的內部部署帳戶
services: active-directory
keywords: Azure AD Connect 傳遞驗證, 安裝 Active Directory, Azure AD 的必要元件, SSO, 單一登入
documentationcenter: ''
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 7a787535d6e6e216df27c8cf7448fa584e85ef2b
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306607"
---
# <a name="azure-active-directory-pass-through-authentication-security-deep-dive"></a>Azure Active Directory 傳遞驗證安全性深入探討

本文提供 Azure Active Directory (Azure AD) 傳遞驗證運作方式的詳細說明。 此說明較著重在功能的安全性方面。 本文適用於安全性和 IT 管理員、法務長和資訊安全長，以及在中小型組織或大型企業負責 IT 安全性與合規性的其他 IT 專業人員。

處理的主題包括：
- 關於如何安裝及註冊驗證代理程式的詳細技術資訊。
- 關於使用者登入期間密碼加密的詳細技術資訊。
- 內部部署驗證代理程式與 Azure AD 之間的通道安全性。
- 關於驗證代理程式如何保持運作安全的詳細技術資訊。
- 其他與安全性相關的主題。

## <a name="key-security-capabilities"></a>主要的安全性功能

以下是此功能的重要安全性層面：
- 此功能建置在安全的多租用戶架構上，可隔離租用戶之間的登入要求。
- 內部部署密碼絕對不會以任何形式儲存在雲端。
- 接聽並回應密碼驗證要求的內部部署驗證代理程式，只會從網路內建立輸出連線。 不需要在周邊網路 (DMZ) 安裝這些驗證代理程式。 最佳做法是將執行驗證代理程式的所有伺服器視為階層 0 的系統 (請參閱[參考](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) \(機器翻譯\))。
- 只有標準連接埠 (80 和 443) 會用於驗證代理程式至 Azure AD 的輸出通訊。 您不需要在防火牆上開放輸入連接埠。 
  - 連接埠 443 會用於所有經過驗證的輸出通訊。
  - 連接埠 80 僅用於下載憑證撤銷清單 (CRL)，以確保此功能所使用的任何憑證都沒有被撤銷。
  - 如需網路需求的完整清單，請參閱 [Azure Active Directory 傳遞驗證：快速入門](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)。
- 使用者在登入期間所提供的密碼在雲端會經過加密，內部部署驗證代理程式才會接受密碼，對 Active Directory 進行驗證。
- Azure AD 和內部部署驗證代理程式之間的 HTTPS 通道可使用相互驗證受到保護。
- 與 [Azure AD 條件式存取原則](../active-directory-conditional-access-azure-portal.md) (包括 Multi-Factor Authentication (MFA)) 緊密配合、[封鎖舊版驗證](../conditional-access/conditions.md)，並[篩除暴力密碼破解攻擊](../authentication/howto-password-smart-lockout.md)，藉此保護您的使用者帳戶。

## <a name="components-involved"></a>涉及的元件

如需 Azure AD 作業、服務與資料安全性的一般詳細資訊，請參閱[信任中心](https://azure.microsoft.com/support/trust-center/)。 將傳遞驗證用於使用者登入時，需要下列元件：
- **Azure AD STS**：無狀態的 Security Token Service (STS)，用以處理登入要求，並在需要時，將安全性權杖核發給使用者的瀏覽器、用戶端或服務。
- **Azure 服務匯流排**：可為企業傳訊及轉送通訊提供雲端通訊的能力，讓您可以連接內部部署解決方案與雲端。
- **Azure AD Connect 驗證代理程式**：接聽並回應密碼驗證要求的內部部署元件。
- **Azure SQL Database**：保存租用戶之驗證代理程式的相關資訊，包括其中繼資料和加密金鑰。
- **Active Directory**：您的內部部署 Active Directory，其中儲存您的使用者帳戶及其密碼。

## <a name="installation-and-registration-of-the-authentication-agents"></a>驗證代理程式的安裝和註冊

當您執行下列其中一項時，即會安裝驗證代理程式並向 Azure AD 進行註冊：
   - [透過 Azure AD Connect 啟用傳遞驗證](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-quick-start#step-2-enable-the-feature)
   - [新增更多驗證代理程式，以確保登入要求的高可用性](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-quick-start#step-4-ensure-high-availability) 
   
使驗證代理程式運作涉及三個主要階段：

1. 驗證代理程式安裝
2. 驗證代理程式註冊
3. 驗證代理程式初始化

以下幾節將詳細討論這幾個階段。

### <a name="authentication-agent-installation"></a>驗證代理程式安裝

只有全域管理員可以在內部部署伺服器上安裝驗證代理程式 (使用 Azure AD Connect 或獨立安裝)。 安裝會將這兩個新項目新增至 **[控制台]** > **[程式]** > **[程式和功能]** 清單：
- 驗證代理程式應用程式本身。 此應用程式會以 [NetworkService](https://msdn.microsoft.com/library/windows/desktop/ms684272.aspx) 權限執行。
- 用來自動更新驗證代理程式的更新程式應用程式。 此應用程式會以 [LocalSystem](https://msdn.microsoft.com/library/windows/desktop/ms684190.aspx) 權限執行。

### <a name="authentication-agent-registration"></a>驗證代理程式註冊

安裝驗證代理程式之後，需要向 Azure AD 註冊本身。 Azure AD 會為每個驗證代理程式指派一個可用於與 Azure AD 進行安全通訊的唯一數位身分識別憑證。

註冊程序也會繫結驗證代理程式與您的租用戶。 這樣可確保 Azure AD 知道這個特定的驗證代理程式是唯一獲得授權，可為您的租用戶處理密碼驗證要求的驗證代理程式。 系統會針對您註冊的每個新驗證代理程式，重複這個程序。

驗證代理程式會使用下列步驟向 Azure AD 進行註冊：

![代理程式註冊](./media/how-to-connect-pta-security-deep-dive/pta1.png)

1. Azure AD 會先要求全域管理員使用其認證登入 Azure AD。 在登入期間，驗證代理程式會取得可代表全域管理員使用的存取權杖。
2. 驗證代理程式接著會產生一個金鑰組：公開金鑰和私密金鑰。
    - 此金鑰組是透過標準 RSA 2048 位元加密產生的。
    - 私密金鑰會存留在驗證代理程式所在的內部部署伺服器上。
3. 驗證代理程式會透過 HTTPS，向 Azure AD 提出一個「註冊」要求，且要求中包含下列元件：
    - 在步驟 1 中取得的存取權杖。
    - 在步驟 2 中產生的公開金鑰。
    - 憑證簽署要求 (CSR 或憑證要求)。 此要求會向作為其憑證授權單位 (CA) 的 Azure AD 申請數位身分識別憑證。
4. Azure AD 會在註冊要求中驗證存取權杖，並確認要求來自全域管理員。
5. 接著，Azure AD 會簽署數位身分識別憑證，然後將其傳回給驗證代理程式。
    - Azure AD 中的根 CA 會用來簽署憑證。 

     >[!NOTE]
     > 此 CA「不在」Windows 受信任的根憑證授權單位存放區中。
    - 此 CA 僅供傳遞驗證功能使用。 此 CA 只會在驗證代理程式註冊期間用來簽署 CSR。
    -  沒有任何其他 Azure AD 服務會使用此 CA。
    - 此憑證的主體 (辨別名稱或 DN) 設定為您的租用戶識別碼。 此 DN 是可唯一識別租用戶的 GUID。 此 DN 可將憑證的範圍限制為只能用於您的租用戶。
6. Azure AD 會將驗證代理程式的公開金鑰儲存在只有 Azure AD 可存取的 Azure SQL Database 中。
7. 在步驟 5 中核發的憑證會儲存在內部部署伺服器上的 Windows 憑證存放區 (具體而言是在 [CERT_SYSTEM_STORE_LOCAL_MACHINE](https://msdn.microsoft.com/library/windows/desktop/aa388136.aspx#CERT_SYSTEM_STORE_LOCAL_MACHINE) 位置) 中。 驗證代理程式和更新程式應用程式都會加以使用。

### <a name="authentication-agent-initialization"></a>驗證代理程式初始化

驗證代理程式在註冊後第一次啟動，或在伺服器重新啟動後啟動時，需要有適當的方式能夠與 Azure AD 服務安全地進行通訊，並開始接受密碼驗證要求。

![代理程式初始化](./media/how-to-connect-pta-security-deep-dive/pta2.png)

以下是驗證代理程式初始化的方式：

1. 驗證代理程式向 Azure AD 提出輸出啟動程序要求。 
    - 此要求會透過連接埠 443 和相互驗證的 HTTPS 通道提出。 此要求會使用先前在驗證代理程式註冊期間核發的相同憑證。
2. Azure AD 回應此要求，方式是將存取金鑰提供給您的租用戶專屬 (以您的租用戶識別碼識別) 的 Azure 服務匯流排佇列。
3. 驗證代理程式會對佇列建立持續的輸出 HTTPS 連線 (透過連接埠 443)。 
    - 驗證代理程式現在已可擷取並處理密碼驗證要求。

如果您在租用戶上註冊多個驗證代理程式，則初始化程序可確保每個驗證代理程式都連線至相同的服務匯流排佇列。 

## <a name="process-sign-in-requests"></a>處理登入要求 

下圖顯示傳遞驗證如何處理使用者登入要求。

![處理登入](./media/how-to-connect-pta-security-deep-dive/pta3.png)

傳遞驗證處理使用者登入要求的方式如下： 

1. 使用者嘗試存取應用程式，例如 [Outlook Web App](https://outlook.office365.com/owa)。
2. 如果使用者還未登入，應用程式將瀏覽器重新導向至 Azure AD 登入頁面。
3. Azure AD STS 服務會透過 [使用者登入] 頁面回應。
4. 使用者將其使用者名稱輸入 [使用者登入] 頁面中，然後選取 [下一步] 按鈕。
5. 使用者將其密碼輸入 [使用者登入] 頁面中，然後選取 [登入] 按鈕。
6. 使用者名稱與密碼會在 HTTPS POST 要求中提交至 Azure AD STS。
7. Azure AD STS 會針對您的租用戶上註冊的所有驗證代理程式，擷取 Azure SQL Database 中的公開金鑰，並使用這些金鑰為其密碼加密。 
    - Azure AD STS 會針對您的租用戶上註冊的 "N" 個驗證代理程式，產生 "N" 個加密密碼值。
8. Azure AD STS 會將密碼驗證要求 (其中包含使用者名稱和加密的密碼值) 放到您的租用戶專用的服務匯流排佇列上。
9. 初始化的驗證代理程式會持續連線至服務匯流排佇列，因此其中一個可用的驗證代理程式會擷取密碼驗證要求。
10. 驗證代理程式會找出其公開金鑰專屬的加密密碼值 (使用識別碼)，並使用其私密金鑰加以解密。
11. 驗證代理程式會嘗試使用 [Win32 LogonUser API](https://msdn.microsoft.com/library/windows/desktop/aa378184.aspx) (其 **dwLogonType** 參數設定為 **LOGON32_LOGON_NETWORK**)，對內部部署 Active Directory 驗證使用者名稱和密碼。 
    - 此 API 是 Active Directory 同盟服務 (AD FS) 在同盟登入案例中登入使用者所使用的相同 API。
    - 此 API 依賴 Windows Server 中的標準解析程序來尋找網域控制站。
12. 驗證代理程式會從 Active Directory 接收結果，例如成功、使用者名稱或密碼不正確、密碼過期等。
13. 驗證代理程式會透過連接埠 443 上，輸出相互驗證的 HTTPS 通道，將結果轉送回 Azure AD STS。 相互驗證會使用先前在註冊期間發給驗證代理程式的憑證。
14. Azure AD STS 會驗證此結果是否與您租用戶上的特定登入要求相互關聯。
15. Azure AD STS 會依照設定的方式，繼續進行登入程序。 例如，如果密碼驗證成功，使用者可能要經過 Multi-Factor Authentication，或重新導向回應用程式。

## <a name="operational-security-of-the-authentication-agents"></a>驗證代理程式的作業安全性

為確保傳遞驗證在作業上安全無虞，Azure AD 會定期更新驗證代理程式的憑證。 Azure AD 會觸發更新。 這些更新不受驗證代理程式本身的控管。

![作業安全性](./media/how-to-connect-pta-security-deep-dive/pta4.png)

若要透過 Azure AD 更新驗證代理程式的信任：

1. 驗證代理程式會定期 Ping Azure AD (每隔幾小時)，以檢查是否應更新其憑證。 憑證會在其到期日前 30 天更新。
    - 此檢查是透過相互驗證的 HTTPS 通道，使用註冊期間所核發的相同憑證執行的。
2. 如果服務指出應進行更新，驗證代理程式便會產生一個新的金鑰組：公開金鑰和私密金鑰。
    - 這些金鑰是透過標準 RSA 2048 位元加密產生的。
    - 私密金鑰永遠都不會離開內部部署伺服器。
3. 驗證代理程式接著會透過 HTTPS，向 Azure AD 提出一個「憑證更新」要求，且要求中包含下列元件：
    - 從 Windows 憑證存放區上的 CERT_SYSTEM_STORE_LOCAL_MACHINE 位置擷取的現有憑證。 此程序不涉及全域管理員，因此沒有代表全域管理員所需的存取權杖。
    - 在步驟 2 中產生的公開金鑰。
    - 憑證簽署要求 (CSR 或憑證要求)。 此要求會向作為其憑證授權單位的 Azure AD 申請新的數位身分識別憑證。
4. Azure AD 會在憑證更新要求中驗證現有的憑證。 接著，它會確認要求來自租用戶上註冊的驗證代理程式。
5. 如果現有的憑證仍然有效，Azure AD 則會簽署新的數位身分識別憑證，然後將新的憑證發回驗證代理程式。 
6. 如果現有的憑證已過期，Azure AD 會從租用戶的已註冊驗證代理程式清單中，刪除該驗證代理程式。 然後，全域管理員必須手動安裝並註冊新的驗證代理程式。
    - 使用 Azure AD 根 CA 來簽署憑證。
    - 將憑證的主體 (辨別名稱或 DN) 設定為您的租用戶識別碼，亦即可唯一識別租用戶的 GUID。 DN 會將憑證的範圍限定於您的租用戶。
6. Azure AD 會將驗證代理程式的新公開金鑰儲存在只有 Azure AD 可存取的 Azure SQL Database 中。 而與驗證代理程式相關聯的舊公開金鑰則會失效。
7. 然後，在步驟 5 中核發的新憑證會儲存在伺服器上的 Windows 憑證存放區 (具體而言是在 [CERT_SYSTEM_STORE_CURRENT_USER](https://msdn.microsoft.com/library/windows/desktop/aa388136.aspx#CERT_SYSTEM_STORE_CURRENT_USER) 位置) 中。
    - 由於信任更新程序會以非互動方式執行 (不需要有全域管理員的存在)，驗證代理程式不再具有存取權以更新 CERT_SYSTEM_STORE_LOCAL_MACHINE 位置中的現有憑證。 
    
   > [!NOTE]
   > 此程序並不會從 CERT_SYSTEM_STORE_LOCAL_MACHINE 位置中移除憑證本身。
8. 從現在起，會使用新的憑證來驗證。 憑證後續的每次更新都會取代 CERT_SYSTEM_STORE_LOCAL_MACHINE 位置中的憑證。

## <a name="auto-update-of-the-authentication-agents"></a>驗證代理程式的自動更新

發行新版本時，更新程式應用程式會自動更新驗證代理程式。 此應用程式不會處理租用戶的任何密碼驗證要求。 

Azure AD 會將新版的軟體當作已簽署的 **Windows Installer 套件 (MSI)** 進行裝載。 MSI 是使用以 SHA256 作為摘要演算法的 [Microsoft Authenticode](https://msdn.microsoft.com/library/ms537359.aspx) 簽署的。 

![自動更新](./media/how-to-connect-pta-security-deep-dive/pta5.png)

若要自動更新驗證代理程式：

1. 更新程式應用程式會每小時 Ping Azure AD 一次，以檢查是否有新版的驗證代理程式可供使用。 
    - 此檢查是透過相互驗證的 HTTPS 通道，使用註冊期間所核發的相同憑證執行的。 驗證代理程式和更新程式會共用伺服器上儲存的憑證。
2. 如果有新的版本可供使用，Azure AD 便會將已簽署的 MSI 傳回到更新程式。
3. 更新程式驗證 MSI 是否由 Microsoft 簽署。
4. 更新程式執行 MSI。 此動作涉及下列步驟：

 > [!NOTE]
 > 更新程式會以 [本機系統](https://msdn.microsoft.com/library/windows/desktop/ms684190.aspx) 權限執行。

    - 停止驗證代理程式服務
    - 在伺服器上安裝新版的驗證代理程式
    - 重新啟動驗證代理程式服務

>[!NOTE]
>如果您在租用戶上註冊了多個驗證代理程式，則 Azure AD 不會更新其憑證或者不會同時更新這些憑證。 但是，Azure AD 會逐漸更新，以確保登入要求的高可用性。
>


## <a name="next-steps"></a>後續步驟
- [目前的限制](how-to-connect-pta-current-limitations.md)：了解支援的情節和不支援的情節。
- [快速入門](how-to-connect-pta-quick-start.md)：開始使用 Azure AD 傳遞驗證。
- [從 AD FS 遷移到傳遞驗證](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx) \(英文\) - 從 AD FS (或其他同盟技術) 遷移到傳遞驗證的詳細指南。
- [智慧鎖定](../authentication/howto-password-smart-lockout.md)：在租用戶中設定智慧鎖定功能以保護使用者帳戶。
- [運作方式](how-to-connect-pta-how-it-works.md)：了解 Azure AD 傳遞驗證運作方式的基本概念。
- [常見問題集](how-to-connect-pta-faq.md)：常見問題集的答案。
- [疑難排解](tshoot-connect-pass-through-authentication.md)：了解如何解決傳遞驗證功能的常見問題。
- [Azure AD 無縫 SSO](how-to-connect-sso.md)：深入了解此互補功能。
