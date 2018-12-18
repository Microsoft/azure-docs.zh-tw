---
title: Azure Multi-Factor Authentication 的安全性指引
description: 本文件提供搭配 Azure 帳戶使用 Azure MFA 的指引
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 8b6ed6a7e71da2f302df4b41656c8a63e93be2b0
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159068"
---
# <a name="security-guidance-for-using-azure-multi-factor-authentication-with-azure-ad-accounts"></a>搭配 Azure AD 帳戶使用 Azure Multi-Factor Authentication 的安全性指引

對於想要加強驗證程序的大多數組織而言，雙步驟驗證是首選。 Azure Multi-Factor Authentication (MFA) 協助公司符合其安全性和合規性需求，同時為使用者提供簡單的登入體驗。 本文涵蓋一些您在規劃採用 Azure MFA 時應該考慮的祕訣。

## <a name="deploy-azure-mfa-in-the-cloud"></a>在雲端部署 Azure MFA

有兩種方式可以[為所有使用者啟用 Azure MFA](howto-mfa-getstarted.md)。

* 為每個使用者購買授權 (Azure MFA、Azure AD Premium 或 Enterprise Mobility + Security)
* 建立 Multi-Factor Auth Provider，並依每個使用者或每次驗證付費

### <a name="licenses"></a>授權
![EMS](./media/multi-factor-authentication-security-best-practices/ems.png)

如果您有 Azure AD Premium 或 Enterprise Mobility + Security 授權，則您已經有 Azure MFA。 您的組織不需要任何其他項目，即可將雙步驟驗證功能延伸到所有使用者。 您只需要將授權指派給使用者，接著就可以開啟 MFA。

設定 Multi-Factor Authentication 時，請考慮下列訣竅：

* 不要建立依每次驗證的 Multi-Factor Auth Provider。 如果這麼做，您可能因為已有授權的使用者提出驗證要求而需要付費。
* 如果您沒有足夠的授權給所有使用者，您可以建立依每個使用者的 Multi-Factor Auth Provider，以涵蓋組織的其餘部分。 
* 唯有當您將內部部署 Active Directory 環境與 Azure AD 目錄同步處理時，才需要 Azure AD Connect。 如果您使用不與內部部署 Active Directory 執行個體同步處理的 Azure AD 目錄，就不需要 Azure AD Connect。

### <a name="multi-factor-auth-provider"></a>Multi-Factor Auth Provider
![Multi-Factor Auth Provider](./media/multi-factor-authentication-security-best-practices/authprovider.png)

如果沒有包含 Azure MFA 的授權，您可以[建立 MFA 驗證提供者](concept-mfa-authprovider.md)。

建立 Auth Provider 時，您必須選取目錄並考慮下列詳細資料：

* 您不需要 Azure AD 目錄即可建立多因素驗證提供者，但如果有則可以獲得更多功能。 將驗證提供者與 Azure AD 目錄相關聯時，將會啟用下列功能︰
  * 將雙步驟驗證延伸到所有使用者
  * 將其他功能提供給全域管理員，例如管理入口網站、自訂問候語和報告。
* 如果您將內部部署 Active Directory 環境與 Azure AD 目錄同步處理，則需要 DirSync 或 AAD Sync。如果您使用不與內部部署 Active Directory 執行個體同步處理的 Azure AD 目錄，就不需要 DirSync 或 AAD Sync。
* 選擇最適合您企業的使用情況模型。 使用量模型在選取之後，就無法再進行變更。 兩個模型如下︰
  * 每次驗證︰向您收取每次驗證的費用。 如果您想要所有存取特定應用程式的人員都使用雙步驟驗證，請使用這個模型。
  * 每個啟用的使用者︰會針對啟用 Azure MFA 的每個使用者向您數費。 如果您有某些使用者具有 Azure AD Premium 或 Enterprise Mobility Suite 授權，有些則沒有，請使用此模型。

### <a name="supportability"></a>支援能力
因為大多數使用者已經習慣僅使用密碼來驗證，所以公司務必讓所有使用者了解此程序。 這番了解可以避免使用者因為 MFA 的小問題就連絡技術人員。 不過，有一些案例是需要暫時停用 MFA。 使用下列指導方針了解如何處理這些案例：

* 如果使用者因為行動裝置應用程式未收到通知或電話來電而無法登入，請訓練您的技術支援工作人員處理這種情況。 技術支援人員可以[啟用單次許可](howto-mfa-mfasettings.md#one-time-bypass)，讓使用者「略過」雙步驟驗證而只需要驗證一次。 許可只是暫時性，經過指定的秒數之後就會到期。
* 請考慮使用 Azure MFA 中的[信任的 IP 功能](howto-mfa-mfasettings.md#trusted-ips)，以盡量避免雙步驟驗證。 受控或同盟租用戶的管理員可以利用此功能，讓從公司近端內部網路登入的使用者略過雙步驟驗證。 這些功能適用於擁有 Azure AD Premium、Enterprise Mobility Suite 或 Azure Multi-Factor Authentication 授權的 Azure AD 租用戶。

## <a name="best-practices-for-an-on-premises-deployment"></a>內部部署的最佳作法
如果您的公司決定採用自己的基礎結構來啟用 MFA，則您需要[在內部部署中部署 Azure Multi-Factor Authentication Server](howto-mfaserver-deploy.md)。 下圖顯示 MFA Server 元件：

![MFA Server 預設元件︰主控台、同步處理引擎、管理入口網站、雲端服務](./media/multi-factor-authentication-security-best-practices/server.png) \*依預設未安裝\**已安裝，但依預設未啟用

Azure Multi-Factor Authentication Server 可以使用同盟來保護雲端資源和內部部署資源的安全。 您必須擁有 AD FS 並讓它與您的 Azure AD 租用戶同盟。
設定 Multi-Factor Authentication Server 時，請考慮下列詳細資料：

* 如果您使用 Active Directory Federation Services (AD FS) 保護 Azure AD 資源，則第一個驗證步驟是使用 AD FS 在內部部署執行。 第二個步驟是使用宣告以在內部部署執行。
* 您不需要在 AD FS 同盟伺服器上安裝 Azure Multi-Factor Authentication Server。 不過，必須在執行 AD FS 的 Windows Server 2012 R2 上安裝適用於 AD FS 的 Multi-Factor Authentication 配接器。 您可以在不同的電腦上安裝伺服器 (只要是支援的版本即可) 以及在 AD FS 同盟伺服器上個別安裝 AD FS 配接器。 
* Multi-Factor Authentication AD FS 配接器安裝精靈會在 Active Directory 中建立名為 PhoneFactor Admins 的安全性群組，然後將 AD FS 服務帳戶新增至這個群組。 確認確實已在網域控制站上建立 PhoneFactor Admins 群組，而且 AD FS 服務帳戶是此群組的成員。 如有必要，請以手動方式將 AD FS 服務帳戶加入至網域控制站上的 PhoneFactor Admins 群組。

### <a name="user-portal"></a>使用者入口網站
使用者入口網站允許自助功能，且提供一組完整的使用者管理功能。 它在 Internet Information Server (IIS) 網站中執行。 遵循下列指導方針以設定此元件：

* 使用 IIS 6 或更新版本
* 安裝和註冊 ASP.NET v2.0.507207
* 請確定此伺服器可以部署在周邊網路中

### <a name="app-passwords"></a>應用程式密碼
如果您的組織使用 SSO 與 Azure AD 同盟，而且您想要使用 Azure MFA 時，請注意下列細節：

* 應用程式密碼由 Azure AD 驗證，因此會略過同盟。 唯有在設定應用程式密碼時才會使用同盟。
* 對於同盟 (SSO) 使用者，密碼會儲存在組織識別碼中。如果使用者離開公司，這些資訊必須使用 DirSync 流向組織識別碼。 停用/刪除帳戶可能需要長達三個小時才能完成同步處理，導致 Azure AD 中停用/刪除應用程式密碼時延遲。
* 應用程式密碼不會遵守內部部署用戶端存取控制設定。
* 應用程式密碼不適用內部部署驗證記錄/稽核功能。
* 某些進階架構設計在使用雙步驟驗證時，可能需要搭配使用組織使用者名稱和密碼及應用程式密碼，需視驗證的位置而定。 對於根據內部部署基礎結構進行驗證的用戶端，您可以使用組織使用者名稱和密碼。 對於根據 Azure AD 進行驗證的用戶端，您需要使用應用程式密碼。
* 根據預設，使用者無法建立應用程式密碼。 如果您需要允許使用者建立應用程式密碼，請選取 [允許使用者建立應用程式密碼以登入非瀏覽器應用程式] 選項。

## <a name="additional-considerations"></a>其他考量
針對部署在內部部署的每個元件，請使用此清單以了解其他考量和最佳做法：

- 搭配 [Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md) 來設定 Multi-Factor Authentication。
- 搭配 [RADIUS 驗證](howto-mfaserver-dir-radius.md)來安裝和設定 Azure MFA Server。
- 搭配 [IIS 驗證](howto-mfaserver-iis.md)來安裝和設定 Azure MFA Server。
- 搭配 [Windows 驗證](howto-mfaserver-windows.md)來安裝和設定 Azure MFA Server。
- 搭配 [LDAP 驗證](howto-mfaserver-dir-ldap.md)來安裝和設定 Azure MFA Server。
- 搭配[使用 RADIUS 的遠端桌面閘道和 Azure Multi-Factor Authentication Server](howto-mfaserver-nps-rdg.md)，以安裝和設定 Azure MFA Server。
- 安裝和設定 Azure MFA Server 和 [ndows Server Active Directory](howto-mfaserver-dir-ad.md)之間的同步處理。
- [部署 Azure Multi-Factor Authentication Server 行動裝置應用程式 Web 服務](howto-mfaserver-deploy-mobileapp.md)。
- [採用 Azure Multi-Factor Authentication 的進階 VPN 設定](howto-mfaserver-nps-vpn.md)，適用於使用 LDAP 或 RADIUS 的 Cisco ASA、Citrix Netscaler 和 Juniper/Pulse 安全 VPN 應用裝置。

## <a name="next-steps"></a>後續步驟
雖然這篇文章強調 Azure MFA 的一些最佳做法，還有其他資源您也可以在規劃 MFA 部署時使用。 以下清單有可以在此程序期間協助您的一些關鍵文章：

* [Azure Multi-Factor Authentication 中的報告](howto-mfa-reporting.md)
* [雙步驟驗證註冊體驗](../user-help/multi-factor-authentication-end-user-first-time.md)
* [Azure Multi-Factor Authentication 常見問題集](multi-factor-authentication-faq.md)
