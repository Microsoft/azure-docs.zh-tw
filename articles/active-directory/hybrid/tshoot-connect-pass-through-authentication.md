---
title: Azure AD Connect：針對傳遞驗證進行疑難排解 | Microsoft Docs
description: 本文會說明如何針對 Azure Active Directory (Azure AD) 傳遞驗證進行疑難排解。
services: active-directory
keywords: 針對 Azure AD Connect 傳遞驗證進行疑難排解, 安裝 Active Directory, Azure AD, SSO, 單一登入的必要元件
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/24/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: d4f44deac1e8833af064e0abea4e65c5ca2b2f93
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309167"
---
# <a name="troubleshoot-azure-active-directory-pass-through-authentication"></a>針對 Azure Active Directory 傳遞驗證進行疑難排解

這篇文章可協助您尋找有關 Azure AD 傳遞驗證常見問題的疑難排解資訊。

>[!IMPORTANT]
>如果傳遞驗證發生使用者登入的問題，請不要在沒有可切換的僅限雲端全域管理員帳戶的情況下，停用此功能或解除安裝傳遞驗證代理程式。 了解如何[新增僅限雲端管理員帳戶 (英文)](../active-directory-users-create-azure-portal.md)。 這是確保您不會被租用戶封鎖的關鍵步驟。

## <a name="general-issues"></a>一般問題

### <a name="check-status-of-the-feature-and-authentication-agents"></a>檢查此功能和驗證代理程式的狀態

確定您租用戶上的傳遞驗證功能仍為 [已啟用]，而驗證代理程式的狀態會顯示 [作用中]，而不是 [非作用中]。 您可以前往 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)上的 [Azure AD Connect] 刀鋒視窗來檢查狀態。

![Azure Active Directory 管理中心 - Azure AD Connect 刀鋒視窗](./media/tshoot-connect-pass-through-authentication/pta7.png)

![Azure Active Directory 管理中心 - 傳遞驗證刀鋒視窗](./media/tshoot-connect-pass-through-authentication/pta11.png)

### <a name="user-facing-sign-in-error-messages"></a>使用者看到的登入錯誤訊息

如果使用者無法登入使用傳遞驗證，他們可能會在 Azure AD 登入畫面中看到下列其中之一的使用者錯誤： 

|Error|說明|解決方案
| --- | --- | ---
|AADSTS80001|無法連線至 Active Directory|確定代理程式伺服器和必須驗證其密碼的使用者都是相同 AD 樹系的成員，而且都能連線到 Active Directory。  
|AADSTS8002|連線至 Active Directory 時發生逾時|請檢查以確定 Active Directory 可用，並且會回應來自代理程式的要求。
|AADSTS80004|傳遞給代理程式的使用者名稱無效|請確定使用者嘗試用來登入的使用者名稱正確無誤。
|AADSTS80005|驗證發生無法預期的 WebException|暫時性錯誤。 重試要求。 如果持續發生失敗，請連絡 Microsoft 支援服務。
|AADSTS80007|和 Active Directory 通訊時發生錯誤|請檢查代理程式記錄檔以了解詳細資訊，並確認 Active Directory 如預期般運作。

### <a name="sign-in-failure-reasons-on-the-azure-active-directory-admin-center-needs-premium-license"></a>Azure Active Directory 管理中心上的登入失敗原因 (需要 Premium 授權)

如果您的租用戶已有與其相關聯的 Azure AD Premium 授權，您也可以查看 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)上的[登入活動報告](../reports-monitoring/concept-sign-ins.md)。

![Azure Active Directory 管理中心 - 登入報告](./media/tshoot-connect-pass-through-authentication/pta4.png)

巡覽至位在 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)的 **Azure Active Directory** -> [登入]，按一下特定使用者的登入活動。 尋找 [登入錯誤碼] 欄位。 使用下表，將該欄位的值對應至失敗的原因和解決方式：

|登入錯誤碼|登入失敗原因|解決方案
| --- | --- | ---
| 50144 | 使用者的 Active Directory 密碼已到期。 | 在您的內部部署 Active Directory 中重設使用者密碼。
| 80001 | 沒有可用的驗證代理程式。 | 安裝並註冊驗證代理程式。
| 80002 | 驗證代理程式的密碼驗證要求已逾時。 | 檢查是否可以從驗證代理程式連線到您的 Active Directory。
| 80003 | 驗證代理程式收到無效的回應。 | 如果有多位使用者發生一樣的問題，請檢查您的 Active Directory 設定。
| 80004 | 登入要求中使用的使用者主體名稱 (UPN) 不正確。 | 要求使用者以正確的使用者名稱登入。
| 80005 | 驗證代理程式：發生錯誤。 | 暫時性錯誤。 請稍後再試。
| 80007 | 驗證代理程式無法連線至 Active Directory。 | 檢查是否可以從驗證代理程式連線到您的 Active Directory。
| 80010 | 驗證代理程式無法連線將密碼解密。 | 如果問題一再出現，請安裝並註冊新的驗證代理程式。 然後解除安裝目前的代理程式。 
| 80011 | 驗證代理程式無法擷取解密金鑰。 | 如果問題一再出現，請安裝並註冊新的驗證代理程式。 然後解除安裝目前的代理程式。

## <a name="authentication-agent-installation-issues"></a>驗證代理程式安裝問題

### <a name="an-unexpected-error-occurred"></a>發生意外的錯誤

從伺服器[收集代理程式記錄](#collecting-pass-through-authentication-agent-logs)，並連絡 Microsoft 支援服務解決您的問題。

## <a name="authentication-agent-registration-issues"></a>驗證代理程式註冊問題

### <a name="registration-of-the-authentication-agent-failed-due-to-blocked-ports"></a>由於連接埠遭到封鎖，所以驗證代理程式註冊失敗。

確認已安裝驗證代理程式的伺服器，能與我們列在[這裡](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)的服務 URL 和連接埠通訊。

### <a name="registration-of-the-authentication-agent-failed-due-to-token-or-account-authorization-errors"></a>因為權杖或帳戶授權錯誤，所以驗證代理程式註冊失敗。

請確定您在所有 Azure AD Connect 或獨立驗證代理程式安裝和註冊作業中，使用僅限雲端的全域管理員帳戶。 啟用 MFA 的全域管理員帳戶有一個已知的問題，請暫時關閉 MFA (只是為了完成作業) 作為因應措施。

### <a name="an-unexpected-error-occurred"></a>發生意外的錯誤

從伺服器[收集代理程式記錄](#collecting-pass-through-authentication-agent-logs)，並連絡 Microsoft 支援服務解決您的問題。

## <a name="authentication-agent-uninstallation-issues"></a>驗證代理程式解除安裝問題

### <a name="warning-message-when-uninstalling-azure-ad-connect"></a>解除安裝 Azure AD Connect 時出現的警告訊息

如果您已在租用戶上啟用傳遞驗證，但您嘗試解除安裝 Azure AD Connect，它會顯示下列警告訊息：「除非您在其他伺服器上有安裝其他傳遞驗證代理程式，否則使用者將無法登入 Azure AD。」

在您解除安裝 Azure AD Connect 之前，請確認您的設定為[高可用性](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability)，以避免中斷使用者登入。

## <a name="issues-with-enabling-the-feature"></a>啟用此功能的問題

### <a name="enabling-the-feature-failed-because-there-were-no-authentication-agents-available"></a>因為沒有任何可用的驗證代理程式，所以啟用此功能失敗。

您至少必須有一個使用中的驗證代理程式，才能在租用戶上啟用傳遞驗證。 您可以安裝 Azure AD Connect 或獨立的驗證代理程式，來安裝驗證代理程式。

### <a name="enabling-the-feature-failed-due-to-blocked-ports"></a>因為連接埠遭到封鎖，所以啟用功能失敗。

確認已安裝 Azure AD Connect 的伺服器能與我們的服務 URL 和連接埠通訊，如[這裡](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)所列。

### <a name="enabling-the-feature-failed-due-to-token-or-account-authorization-errors"></a>因為權杖或帳戶授權錯誤，所以啟用功能失敗。

請確定您使用僅限雲端的全域管理員帳戶來啟用此功能。 啟用 Multi-Factor Authentication (MFA) 的全域管理員帳戶有一個已知的問題，請暫時關閉 MFA (只是為了完成作業) 作為因應措施。

## <a name="collecting-pass-through-authentication-agent-logs"></a>收集傳遞驗證代理程式記錄

根據發生的問題類型，您需要在不同的位置尋找傳遞驗證代理程式記錄。

### <a name="azure-ad-connect-logs"></a>Azure AD Connect 記錄

針對安裝相關的錯誤，請檢查 Azure AD Connect 記錄，位置在 **%ProgramData%\AADConnect\trace-\*.log**。

### <a name="authentication-agent-event-logs"></a>驗證代理程式事件記錄檔

如需與驗證代理程式相關的錯誤，請開啟伺服器上的事件檢視器應用程式，並於 **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin** 下查看。

如需詳細分析，請啟用「工作階段」記錄。 不要使用正常作業期間啟用的記錄檔執行驗證代理程式，此記錄檔只適用於進行疑難排解。 只有再次停用此記錄檔之後，才能看見記錄檔的內容。

### <a name="detailed-trace-logs"></a>詳細的追蹤記錄檔

若要針對使用者登入失敗進行疑難排解，請查看追蹤記錄，其位於 **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace\\**。 這些記錄包含使用傳遞驗證功能的特定使用者為什麼會登入失敗的原因。 這些錯誤也對應到先前的登入失敗原因資料表中所示的登入失敗原因。 以下是記錄項目範例：

```
    AzureADConnectAuthenticationAgentService.exe Error: 0 : Passthrough Authentication request failed. RequestId: 'df63f4a4-68b9-44ae-8d81-6ad2d844d84e'. Reason: '1328'.
        ThreadId=5
        DateTime=xxxx-xx-xxTxx:xx:xx.xxxxxxZ
```

您可以開啟命令提示字元並執行下列命令 (注意：請以您在記錄中看到的實際錯誤編號取代 '1328')，以取得錯誤 (前例中為 '1328') 的描述性詳細資料：

`Net helpmsg 1328`

![傳遞驗證](./media/tshoot-connect-pass-through-authentication/pta3.png)

### <a name="domain-controller-logs"></a>網域控制站記錄檔

如果已經啟用稽核記錄，您可以在網域控制站的安全性記錄檔中找到其他資訊。 查詢傳遞驗證代理程式所傳送之登入要求的簡單方式如下︰

```
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ProcessName'] and (Data='C:\Program Files\Microsoft Azure AD Connect Authentication Agent\AzureADConnectAuthenticationAgentService.exe')]]</Select>
    </Query>
    </QueryList>
```

## <a name="performance-monitor-counters"></a>效能監視器計數器

另一種監視驗證代理程式的方法就是，追蹤每個有安裝驗證代理程式之伺服器上的特定效能監視計數器。 使用下列全域計數器 (**# PTA authentications**、**#PTA failed authentications** 及 **#PTA successful authentications**) 和錯誤計數器 (**# PTA authentication errors**)：

![傳遞驗證效能監視器計數器](./media/tshoot-connect-pass-through-authentication/pta12.png)

>[!IMPORTANT]
>傳遞驗證使用多個驗證代理程式，且_不_進行負載平衡，而提供了高可用性。 視您的設定而定，_並非_所有的驗證代理程式都會收到數目大約_相同_的要求。 特定的驗證代理程式可能完全不會收到流量。
