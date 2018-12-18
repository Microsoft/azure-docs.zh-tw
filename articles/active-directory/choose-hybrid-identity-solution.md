---
title: 選擇 Azure 混合式身分識別解決方案 | Microsoft Docs
description: 取得可用身分識別解決方案和建議的基本了解，可讓您為貴組織做出最佳的身分識別控管決策。
keywords: ''
author: jeffgilb
manager: mtillman
ms.reviewer: jsnow
ms.author: billmath
ms.date: 03/02/2018
ms.topic: article
ms.prod: ''
ms.service: azure
ms.technology: ''
ms.assetid: ''
ms.custom: it-pro
ms.openlocfilehash: 905cedd6ee781ca163731a6f71b083acfc11b5eb
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "36332366"
---
# <a name="microsoft-hybrid-identity-solutions"></a>Microsoft 混合式身分識別解決方案
[Microsoft Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 混合式身分識別解決方案可讓您同步處理內部部署目錄物件與 Azure AD，仍同時在內部部署環境管理使用者。 在規劃同步處理內部部署 Windows Server Active Directory 與 Azure AD 時，首先要決定您要使用已同步處理的身分識別或同盟身分識別。 已同步處理的身分識別和密碼雜湊 (選擇性)，讓使用者能使用相同的密碼來存取內部部署和雲端式組織資源。 如需更進階的案例需求，例如單一登入 (SSO) 或內部部署 MFA，您必須將 Active Directory Federation Services (AD FS) 部署至同盟身分識別。 

有數個選項可用來設定混合式身分識別。 本文提供的資訊可協助您根據部署的容易度以及您特定的身分識別和存取管理需求，選擇最適合貴組織的選項。 當您考慮哪個身分識別模型最適合貴組織的需求時，您也需要考慮時間、現有的基礎結構、複雜度和成本。 這些因素對每個組織而言不同，並可能隨時間改變。 不過，如果您的需求有所變更，您也有彈性切換至不同的身分識別模型。

> [!TIP]
> 這些解決方案全都由 [Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 傳遞。

## <a name="synchronized-identity"></a>已同步處理的身分識別 
已同步處理的身分識別是同步處理內部部署目錄物件 (使用者和群組) 與 Azure AD 的最簡單方法。 

![已同步處理的混合式身分識別](./media/choose-hybrid-identity-solution/synchronized-identity.png)

雖然已同步處理的身分識別是最簡單且最快速的方法，但使用者仍需為雲端式資源維護個別的密碼。 若要避免這個問題，您也可以 (選擇性) [將使用者密碼的雜湊同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#what-is-password-synchronization)至 Azure AD 目錄。 同步處理密碼雜湊，可讓使用者使用其在使用內部部署環境使用的相同使用者名稱和密碼來登入雲端式組織資源。 Azure AD Connect 會定期檢查您的內部部署目錄變更，並且讓 Azure AD 目錄保持同步狀態。 當內部部署 Active Directory 的使用者屬性或密碼變更時，它會自動在 Azure AD 中更新。 

由於大部分組織只想要讓使用者登入 Office 365、SaaS 應用程式和其他 Azure AD 資源，因此建議使用預設的密碼同步處理選項。 如果您適用該選項，您必須在傳遞驗證與 AD FS 之間做決定。

> [!TIP]
> 使用者密碼會以代表實際使用者密碼的雜湊值形式儲存在內部部署 Windows Server Active Directory 中。 雜湊值是單向數學函式 (雜湊演算法) 的計算結果。 沒有任何方法可將單向函式的結果還原為純文字版本的密碼。 您無法使用密碼雜湊來登入您的內部部署網路。 當您選擇要同步處理密碼時，Azure AD Connect 會從內部部署 Active Directory 擷取密碼雜湊，並且在密碼雜湊同步處理至 Azure AD 之前套用額外的安全性處理。 密碼同步處理也可以搭配密碼回寫功能一起使用，以啟用 Azure AD 中的自助式密碼重設功能。 此外，您可以針對連至公司網路且加入網域的電腦使用者啟用單一登入 (SSO)。 使用單一登入，啟用的使用者只需要輸入使用者名稱即可安全地存取雲端資源。 

## <a name="pass-through-authentication"></a>傳遞驗證
[Azure AD 傳遞驗證](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication)會針對使用內部部署 Active Directory 並以 Azure AD 為基礎的服務，提供簡單的密碼驗證解決方案。 如果貴組織的安全性與合規性原則不允許傳送使用者的密碼 (即使以雜湊表單形式)，而您只需要對已加入網域的裝置支援桌面 SSO，建議您評估使用傳遞驗證。 相較於 AD FS，傳遞驗證不需要在 DMZ 中部署，可簡化部署基礎結構。 當使用者使用 Azure AD 登入時，此驗證方法會向您的內部部署 Active Directory 直接驗證使用者的密碼。

![傳遞驗證](./media/choose-hybrid-identity-solution/pass-through-authentication.png)

使用傳遞驗證時，不需要複雜的網路基礎結構，而且您不需要在雲端儲存內部部署密碼。 與單一登入結合的傳遞驗證，可在登入 Azure AD 或其他雲端服務時，提供真正整合的經驗。

傳遞驗證已透過 Azure AD Connect 設定，其使用可接聽密碼驗證要求的簡單內部部署代理程式。 代理程式可以輕鬆地部署到多部電腦，以提供高可用性和負載平衡。 因為所有通訊都是輸出通訊，所以不需要在 DMZ 中安裝連接器。 連接器的伺服器電腦需求如下：

- Windows Server 2012 R2 或更新版本
- 加入樹系中將透過它驗證使用者的網域

## <a name="federated-identity-ad-fs"></a>同盟身分識別 (AD FS)
如需更進一步控制使用者如何存取 Office 365 和其他雲端服務，您可以使用 [Active Directory Federation Services (AD FS)](https://docs.microsoft.com/windows-server/identity/ad-fs/overview/whats-new-active-directory-federation-services-windows-server) 設定與單一登入 (SSO) 的目錄同步作業。 讓使用者的登入與 AD FS 結成同盟，可將驗證委派給會驗證使用者認證的內部部署伺服器。 在此模型中，決不會將內部部署 Active Directory 認證傳送至 Azure AD。

![同盟身分識別](./media/choose-hybrid-identity-solution/federated-identity.png)

也稱為身分識別同盟，此登入方法可確保所有使用者驗證都是在內部部署環境中控制，並可讓系統管理員實作更嚴格的存取控制層級。 與 AD FS 的身分識別同盟是最複雜的選項，而且需要在內部部署環境中部署額外的伺服器。 身分識別同盟也認可您為 Active Directory 和 AD FS 基礎結構提供全天候的支援。 您必須提供此高層級的支援，因為如果內部部署網際網路存取、網域控制站或 AD FS 伺服器無法使用，使用者便無法登入雲端服務。

> [!TIP]
> 如果您選擇使用與 Active Directory Federation Services (AD FS) 同盟，則可以選擇性地設定密碼同步處理來作為 AD FS 基礎結構失敗時的備用方式。


## <a name="common-scenarios-and-recommendations"></a>常見案例和建議
以下是一些常見的混合式身分識別和存取管理案例，其中包含各自適合的混合式身分識別選項建議。

|我需要：|PWS 和 SSO<sup>1</sup>| PTA 和 SSO<sup>2</sup> | AD FS<sup>3</sup>|
|-----|-----|-----|-----|
|自動將我的內部部署 Active Directory 中建立的新使用者、連絡人及群組帳戶同步至雲端。|![建議](./media/choose-hybrid-identity-solution/ic195031.png)| ![建議](./media/choose-hybrid-identity-solution/ic195031.png) |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|設定 Office 365 混合式案例的租用戶|![建議](./media/choose-hybrid-identity-solution/ic195031.png)| ![建議](./media/choose-hybrid-identity-solution/ic195031.png) |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|讓我的使用者可以使用其內部部署密碼登入及存取雲端服務|![建議](./media/choose-hybrid-identity-solution/ic195031.png)| ![建議](./media/choose-hybrid-identity-solution/ic195031.png) |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|使用公司認證實作單一登入|![建議](./media/choose-hybrid-identity-solution/ic195031.png)| ![建議](./media/choose-hybrid-identity-solution/ic195031.png) |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|確定雲端中未儲存任何密碼雜湊| |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|啟用內部部署 Multi-Factor Authentication 解決方案| | |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|對使用者支援智慧卡驗證<sup>4</sup>| | |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|
|在 Office 入口網站中和 Windows 10 桌面上顯示密碼到期通知| | |![建議](./media/choose-hybrid-identity-solution/ic195031.png)|

> <sup>1</sup> 透過單一登入同步處理密碼。 

> <sup>2</sup> 傳遞驗證和單一登入。 

> <sup>3</sup> 與 AD FS 同盟的單一登入。

> <sup>4</sup> AD FS 可與您的企業 PKI 整合，以允許使用憑證登入。 這些憑證可以是透過信任的佈建管道 (例如 MDM、GPO、智慧卡憑證 (包括 PIV/CAC 卡) 或 Hello for Business (cert-trust)) 部署的軟性憑證。 如需智慧卡驗證支援的詳細資訊，請參閱[這個部落格](https://blogs.msdn.microsoft.com/samueld/2016/07/19/adfs-certauth-aad-o365/)。


## <a name="next-steps"></a>後續步驟
[深入了解 Azure 概念證明環境中的其他資訊](https://aka.ms/aad-poc)

[安裝 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771)

[監視混合式身分識別同步處理](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health)

