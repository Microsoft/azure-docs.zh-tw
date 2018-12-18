---
title: Azure AD Connect：針對連線問題進行疑難排解 | Microsoft Docs
description: 說明如何使用 Azure AD Connect 疑難排解連線問題。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 3aa41bb5-6fcb-49da-9747-e7a3bd780e64
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 7f34b0015a4a6d7b8c2c23f539cf21232adaec58
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304667"
---
# <a name="troubleshoot-connectivity-issues-with-azure-ad-connect"></a>對 Azure AD Connect 的連線問題進行疑難排解
這篇文章說明 Azure AD Connect 與 Azure AD 之間的連線的運作方式，以及如何疑難排解連線問題。 這些問題最有可能出現在具有 Proxy 伺服器的環境中。

## <a name="troubleshoot-connectivity-issues-in-the-installation-wizard"></a>在安裝精靈中疑難排解連線問題
Azure AD Connect 使用「新式驗證」(使用 ADAL 程式庫) 來進行驗證。 安裝精靈和同步處理引擎會要求必須正確設定 machine.config，因為這兩個是 .NET 應用程式。

在本文中，我們將說明 Fabrikam 如何透過其 Proxy 連接至 Azure AD。 Proxy 伺服器名為 fabrikamproxy，並且正在使用連接埠 8080。

首先，我們必須確定已正確設定 [**machine.config**](how-to-connect-install-prerequisites.md#connectivity) 。  
![machineconfig](./media/tshoot-connect-connectivity/machineconfig.png)

> [!NOTE]
> 在某些非 Microsoft 部落格中，指出了應該改為變更 miiserver.exe.config。 不過，每次升級時都會覆寫這個檔案，因此，即使在初始安裝期間能運作，在第一次升級時系統也會停止運作。 基於該理由，建議您改為更新 machine.config。
>
>

Proxy 伺服器也必須開啟必要的 URL。 如需官方清單，請參閱 [Office 365 URL 與 IP 位址範圍](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)。

在這些 URL 當中，下表是能夠連接到 Azure AD 的最基本項目。 這份清單並未包含任何選用的功能，例如密碼回寫或 Azure AD Connect Health。 在此記錄以便協助疑難排解初始設定。

| URL | Port | 說明 |
| --- | --- | --- |
| mscrl.microsoft.com |HTTP/80 |用來下載 CRL 清單。 |
| \*.verisign.com |HTTP/80 |用來下載 CRL 清單。 |
| \*.entrust.com |HTTP/80 |用來下載 MFA 的 CRL 清單。 |
| \*.windows.net |HTTPS/443 |用來登入 Azure AD。 |
| secure.aadcdn.microsoftonline-p.com |HTTPS/443 |用於 MFA。 |
| \*.microsoftonline.com |HTTPS/443 |用來設定您的 Azure AD 目錄及匯入/匯出資料。 |

## <a name="errors-in-the-wizard"></a>精靈中的錯誤
安裝精靈會使用兩種不同的安全性內容。 在 [連線到 Azure AD]  頁面上，使用的是目前登入的使用者。 在 [設定] 頁面上，它會變更為[執行同步處理引擎服務的帳戶](reference-connect-accounts-permissions.md#adsync-service-account)。 如果發生問題，問題最有可能已經出現在精靈中的 [連線到 Azure AD] 頁面，因為 Proxy 組態是全域組態。

下列問題是您會在安裝精靈中遇到的最常見錯誤。

### <a name="the-installation-wizard-has-not-been-correctly-configured"></a>安裝精靈未正確設定
當精靈本身無法連線到 Proxy 時，就會出現此錯誤。  
![nomachineconfig](./media/tshoot-connect-connectivity/nomachineconfig.png)

* 如果您看到此錯誤，請確認是否已經正確設定 [machine.config](how-to-connect-install-prerequisites.md#connectivity) 。
* 如果看起來正確，請依照 [確認 Proxy 連線](#verify-proxy-connectivity) 中的步驟，查看問題是否也出現在精靈以外的地方。

### <a name="a-microsoft-account-is-used"></a>使用了 Microsoft 帳戶
如果您使用 **Microsoft 帳戶**而不是**學校或組織帳戶**，就會看到一個一般錯誤。  
![使用了 Microsoft 帳戶](./media/tshoot-connect-connectivity/unknownerror.png)

### <a name="the-mfa-endpoint-cannot-be-reached"></a>無法連線 MFA 端點
如果無法連線到 **https://secure.aadcdn.microsoftonline-p.com** 端點，而您的全域系統管理員已啟用 MFA，就會出現此錯誤。  
![nomachineconfig](./media/tshoot-connect-connectivity/nomicrosoftonlinep.png)

* 如果您看到此錯誤，請確認是否已將 **secure.aadcdn.microsoftonline-p.com** 端點新增到 Proxy。

### <a name="the-password-cannot-be-verified"></a>無法驗證密碼
如果安裝精靈成功連線到 Azure AD，但密碼本身無法獲得驗證，您就會看到此錯誤：  
![badpassword](./media/tshoot-connect-connectivity/badpassword.png)

* 密碼是暫時密碼，而且必須變更嗎？ 實際上是正確的密碼嗎？ 嘗試登入 https://login.microsoftonline.com (在 Azure AD Connect 伺服器以外的另一部電腦上)，並確認該帳戶是否可使用。

### <a name="verify-proxy-connectivity"></a>確認 Proxy 連線
若要確認 Azure AD Connect 伺服器是否確實能夠與 Proxy 和網際網路連線，請使用一些 PowerShell 來查看 Proxy 是否允許 Web 要求。 在 PowerShell 命令提示字元中，執行 `Invoke-WebRequest -Uri https://adminwebservice.microsoftonline.com/ProvisioningService.svc`。 (就技術而言，第一個呼叫是對 https://login.microsoftonline.com 發出，而這個 URI 同樣能夠運作，但另一個 URI 的回應速度較快。)

PowerShell 會使用 machine.config 中的組態來連絡 Proxy。 winhttp/netsh 中設定應該不會影響這些 Cmdlet。

如果 Proxy 設定正確，您應該會收到成功的狀態：![proxy200](./media/tshoot-connect-connectivity/invokewebrequest200.png)

如果您收到 [無法連接至遠端伺服器]，則表示 PowerShell 正嘗試進行直接呼叫而未使用 Proxy，或是 DNS 設定不正確。 請確定 **machine.config** 檔案設定正確。
![unabletoconnect](./media/tshoot-connect-connectivity/invokewebrequestunable.png)

如果 Proxy 設定不正確，您將會收到錯誤：![proxy200](./media/tshoot-connect-connectivity/invokewebrequest403.png)
![proxy407](./media/tshoot-connect-connectivity/invokewebrequest407.png)

| Error | 錯誤文字 | 註解 |
| --- | --- | --- |
| 403 |禁止 |Proxy 尚未對要求的 URL 開放。 重新瀏覽 Proxy 組態，並確定 [URL](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) 已經開啟。 |
| 407 |需要 Proxy 驗證 |Proxy 伺服器要求提供登入資訊，但並未提供任何登入資訊。 如果您的 Proxy 伺服器需要驗證，請務必在 machine.config 中進行這項設定。此外，也請確定您將網域帳戶用於執行精靈的使用者，以及用於服務帳戶。 |

### <a name="proxy-idle-timeout-setting"></a>Proxy 閒置逾時設定
當 Azure AD Connect 將匯出要求傳送至 Azure AD 時，Azure AD 在產生回應之前，可能需要 5 分鐘的時間來處理要求。 特別是當相同的匯出要求中包含多個具有大型群組成員資格的群組時，可能就會發生這個情況。 請確認 Proxy 閒置逾時設定為 5 分鐘以上。 否則，可能會在 Azure AD Connect 伺服器上觀察到 Azure AD 的間歇性連線問題。

## <a name="the-communication-pattern-between-azure-ad-connect-and-azure-ad"></a>Azure AD Connect 與 Azure AD 之間的通訊模式
如果您已依照上述這些步驟操作卻仍然無法連接，這時可以開始查看網路記錄檔。 本節說明正常和成功的連線模式。 它也會列出常見的假象，當您閱讀網路記錄檔時可以略過。

* 會有一些對 https://dc.services.visualstudio.com 的呼叫。 並不需要在 Proxy 中開啟此 URL，安裝即可成功，因此可以忽略這些呼叫。
* 您會看到 DNS 解析列出要在 DNS 命名空間 nsatc.net 中的實際主機，以及其他不在 microsoftonline.com 底下的命名空間。 不過，實際伺服器名稱上沒有任何 Web 服務要求，因此您不需要將這些 URL 新增到 Proxy。
* 端點 adminwebservice 和 provisioningapi 是探索端點，可用來尋找要使用的實際端點。 這些端點會依據您的區域而有所不同。

### <a name="reference-proxy-logs"></a>參考 Proxy 記錄檔
以下是實際 Proxy 記錄檔的傾印及取得它的安裝精靈頁面 (已移除至相同端點的重複項目)。 本節可以作為您自己 Proxy 和網路記錄檔的參考。 您環境中實際的端點可能會有所不同 (特別是以「斜體字」表示的 URL)。

**連接至 Azure AD**

| 時間 | URL |
| --- | --- |
| 1/11/2016 8:31 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:31 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:32 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:32 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:33 |connect://provisioningapi.microsoftonline.com:443 |
| 1/11/2016 8:33 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**設定**

| 時間 | URL |
| --- | --- |
| 1/11/2016 8:43 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:43 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:43 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:46 |connect://provisioningapi.microsoftonline.com:443 |
| 1/11/2016 8:46 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**初始同步處理**

| 時間 | URL |
| --- | --- |
| 1/11/2016 8:48 |connect://login.windows.net:443 |
| 1/11/2016 8:49 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:49 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:49 |connect://*bba800-anchor*.microsoftonline.com:443 |

## <a name="authentication-errors"></a>驗證錯誤
本節涵蓋可能從 ADAL (Azure AD Connect 所使用的驗證程式庫) 和 PowerShell 傳回的錯誤。 其中所說明的錯誤應可幫助您了解後續步驟。

### <a name="invalid-grant"></a>無效的授與
無效的使用者名稱或密碼。 如需詳細資訊，請參閱[無法驗證密碼](#the-password-cannot-be-verified)。

### <a name="unknown-user-type"></a>不明的使用者類型
Azure AD 目錄找不到或無法解析。 可能是您嘗試以未驗證網域中的使用者名稱登入？

### <a name="user-realm-discovery-failed"></a>使用者領域探索失敗
網路或 Proxy 組態問題。 無法連線到網路。 請參閱[針對安裝精靈中的連線問題進行疑難排解](#troubleshoot-connectivity-issues-in-the-installation-wizard)。

### <a name="user-password-expired"></a>使用者密碼過期
您的認證已過期。 請變更密碼。

### <a name="authorization-failure"></a>授權失敗
無法授權使用者在 Azure AD 中執行動作。

### <a name="authentication-cancelled"></a>驗證取消
已取消 Multi-Factor Authentication (MFA) 查問。

### <a name="connect-to-ms-online-failed"></a>連線到 MS Online 失敗
驗證成功，但 Azure AD PowerShell 發生驗證問題。

### <a name="azure-ad-global-admin-role-needed"></a>需要 Azure AD 全域系統管理員角色
已成功驗證使用者。 不過，未對使用者指派全域系統管理員角色。 這是對使用者[指派全域系統管理員角色的方式](../users-groups-roles/directory-assign-admin-roles.md)。 

### <a name="privileged-identity-management-enabled"></a>已啟用 Privileged Identity Management
驗證成功。 已啟用 Privileged Identity Management，而且您目前不是全域管理員。 如需詳細資訊，請參閱 [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md)。

### <a name="company-information-unavailable"></a>無法取得公司資訊
驗證成功。 無法從 Azure AD 擷取公司資訊。

### <a name="domain-information-unavailable"></a>無法取得網域資訊
驗證成功。 無法從 Azure AD 擷取網域資訊。

### <a name="unspecified-authentication-failure"></a>未指定的驗證失敗
在安裝精靈中顯示為未預期的錯誤。 如果您嘗試使用 **Microsoft 帳戶**而不是**學校或組織帳戶**，就可能發生此錯誤。

## <a name="troubleshooting-steps-for-previous-releases"></a>舊版的疑難排解步驟
從組建編號 1.1.105.0 (於 2016 年 2 月發行) 版本開始即已淘汰登入小幫手。 應該已不再需要本節及組態設定，但仍保留供參考之用。

若要讓登入小幫手能夠運作，必須設定 winhttp。 透過 [**netsh**](how-to-connect-install-prerequisites.md#connectivity)，即可完成這項設定。  
![netsh](./media/tshoot-connect-connectivity/netsh.png)

### <a name="the-sign-in-assistant-has-not-been-correctly-configured"></a>登入小幫手未正確設定
當登入小幫手無法連線到 Proxy 或 Proxy 不允許該要求時，就會出現此錯誤。
![nonetsh](./media/tshoot-connect-connectivity/nonetsh.png)

* 如果您看到此錯誤，請查看 [netsh](how-to-connect-install-prerequisites.md#connectivity) 中的 Proxy 組態，並確認它是否正確。
  ![netshshow](./media/tshoot-connect-connectivity/netshshow.png)
* 如果看起來正確，請依照 [確認 Proxy 連線](#verify-proxy-connectivity) 中的步驟，查看問題是否也出現在精靈以外的地方。

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
