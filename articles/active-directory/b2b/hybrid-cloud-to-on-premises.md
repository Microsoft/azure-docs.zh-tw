---
title: 對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權 | Microsoft Docs
description: 說明如何透過 Azure AD B2B 共同作業對雲端 B2B 使用者提供內部部署應用程式的存取權。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 65f43eb28ef2ef51409469934d59e1add7e83666
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297209"
---
# <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-applications"></a>對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權

作為使用 Azure Active Directory (Azure AD) B2B 共同作業功能邀請夥伴組織的來賓使用者進入您 Azure AD 的組織，您現在可以對這些 B2B 使用者提供內部部署應用程式的存取權。 這些內部部署應用程式可以搭配 Kerberos 限制委派 (KCD) 來使用 SAML 型驗證或整合式 Windows 驗證 (IWA)。

## <a name="access-to-saml-apps"></a>SAML 應用程式的存取權

如果您的內部部署應用程式使用 SAML 型驗證，您可以透過 Azure 入口網站輕鬆地將這些應用程式提供給您的 Azure AD B2B 共同作業使用者。

您必須執行下列兩個動作：

- 使用非資源庫的應用程式範本整合 SAML 應用程式，如[設定對不在 Azure Active Directory 應用程式庫中的應用程式的單一登入](../manage-apps/configure-single-sign-on-non-gallery-applications.md)所述。 請務必記下您所使用的**登入 URL** 值。
-  使用 Azure AD 應用程式 Proxy 來發行內部部署應用程式，並將 **Azure Active Directory** 設定為驗證來源。 如需指示，請參閱[使用 Azure AD 應用程式 Proxy 發行應用程式](../manage-apps/application-proxy-publish-azure-portal.md)。 

   當您設定 [內部 Url] 設定時，請使用您在非資源庫的應用程式範本中指定的登入 URL。 如此一來，使用者就可以從組織界限外存取應用程式。 應用程式 Proxy 會為內部部署應用程式執行 SAML 單一登入。
 
   ![顯示內部部署應用程式設定內部 URL 和驗證](media/hybrid-cloud-to-on-premises/OnPremAppSettings.PNG)

## <a name="access-to-iwa-and-kcd-apps"></a>IWA 和 KCD 應用程式的存取權

針對使用整合式 Windows 驗證和 Kerberos 限制委派所保護的內部部署應用程式，若要讓 B2B 使用者獲得這些應用程式的存取權，您需要下列元件：

- **透過 Azure AD 應用程式 Proxy 的驗證**。 B2B 使用者必須能夠向內部部署應用程式進行驗證。 若要這麼做，您必須透過 Azure AD 應用程式 Proxy 發行內部部署應用程式。 如需詳細資訊，請參閱[開始使用應用程式 Proxy 並安裝連接器](../manage-apps/application-proxy-enable.md)和[使用 Azure AD 應用程式 Proxy 發佈應用程式](../manage-apps/application-proxy-publish-azure-portal.md)。
- **透過內部部署目錄中 B2B 使用者物件的授權**。 應用程式必須能夠執行使用者存取權檢查，並授與正確資源的存取權。 IWA 和 KCD 需要內部部署 Windows Server Active Directory 中的使用者物件才能完成此授權。 如[使用 KCD 單一登入的運作方式](../manage-apps/application-proxy-configure-single-sign-on-with-kcd.md#how-single-sign-on-with-kcd-works)所述，應用程式 Proxy 需要這個使用者物件，以模擬使用者並將 Kerberos 權杖交給應用程式。 

   B2B 使用者案例提供兩種方法，供您建立在內部部署目錄中進行授權所需的來賓使用者物件：

   - Microsoft Identity Manager (MIM) 和 [Microsoft Graph 的 MIM 管理代理程式](#create-b2b-guest-user-objects-through-mim-preview)。 
   - [PowerShell 指令碼](#create-b2b-guest-user-objects-through-a-script-preview)。 使用指令碼會是更輕量的解決方案，不需要 MIM 就可運作。 

下圖可針對 Azure AD 應用程式 Proxy 如何與內部部署目錄的 B2B 使用者物件產生一起運作，以對 B2B 使用者授與內部部署 IWA 和 KCD 應用程式的存取權，提供高階的概觀。 圖表下方有各編號步驟的詳細說明。

![MIM 和 B2B 指令碼解決方案的圖表](media/hybrid-cloud-to-on-premises/MIMScriptSolution.PNG)

1.  夥伴組織 (Fabrikam 租用戶) 的使用者獲邀進入 Contoso 租用戶。
2.  Contoso 租用戶中建立了來賓使用者物件 (例如，UPN 為 guest_fabrikam.com#EXT#@contoso.onmicrosoft.com 的使用者物件)。
3.  透過 MIM 或 B2B PowerShell 指令碼從 Contoso 匯入 Fabrikam 來賓。
4.  透過 MIM 或透過 B2B PowerShell 指令碼在內部部署目錄 (Contoso.com) 中建立了 Fabrikam 來賓使用者物件 (Guest#EXT#) 的代表或「足跡」。
5.  來賓使用者存取內部部署應用程式 app.contoso.com。
6.  使用 Kerberos 限制委派透過應用程式 Proxy 授權驗證要求。 
7.  因為來賓使用者物件存在於本機，所以驗證會成功。

### <a name="lifecycle-management-policies"></a>生命週期管理原則

您可以透過生命週期管理原則管理內部部署 B2B 使用者物件。 例如︰

- 您可以為來賓使用者設定多重要素驗證 (MFA) 原則，以在應用程式 Proxy 驗證期間使用 MFA。 如需詳細資訊，請參閱 [B2B 共同作業使用者的條件式存取](conditional-access.md)。
- 任何對雲端 B2B 使用者執行的贊助、存取權檢閱、帳戶驗證等，皆適用於內部部署使用者。 例如，如果透過生命週期管理原則刪除雲端使用者，則也會由 MIM 同步處理或透過 Azure AD Connect 同步處理刪除內部部署使用者。如需詳細資訊，請參閱[使用 Azure AD 存取權檢閱來管理來賓存取權](../governance/manage-guest-access-with-access-reviews.md)。

### <a name="create-b2b-guest-user-objects-through-mim-preview"></a>透過 MIM 建立 B2B 來賓使用者物件 (預覽)

若要了解如何使用 MIM 2016 Service Pack 1 和 Microsoft Graph 的 MIM 管理代理程式在內部部署目錄中建立來賓使用者物件，請參閱 [使用 Microsoft Identity Manager (MIM) 2016 SP1 與 Azure 應用程式 Proxy 進行 Azure AD 企業對企業 (B2B) 共同作業 (公開預覽)](https://docs.microsoft.com/microsoft-identity-manager/microsoft-identity-manager-2016-graph-b2b-scenario)。

### <a name="create-b2b-guest-user-objects-through-a-script-preview"></a>透過指令碼建立 B2B 來賓使用者物件 (預覽)

有 PowerShell 範例指令碼可供您作為起點，以在內部部署 Active Directory 中建立來賓使用者物件。

您可以從[下載中心](https://www.microsoft.com/download/details.aspx?id=51495)下載指令碼和讀我檔案。 請選擇 **Script and Readme to pull Azure AD B2B users on-prem.zip** 檔案。

使用指令碼之前，請確定您已檢閱相關讀我檔案中的先決條件和重要考量。 此外，也請了解指令碼僅供作為範例。 您的開發團隊或合作夥伴必須自訂和檢閱指令碼後再執行。

## <a name="license-considerations"></a>授權考量

請確定您已為存取內部部署應用程式的外部來賓使用者準備正確的用戶端存取授權 (CAL)。 如需詳細資訊，請參閱[用戶端存取授權和管理授權](https://www.microsoft.com/licensing/product-licensing/client-access-license.aspx)的＜外部連接器＞一節。 請就您的特定授權需求向您的 Microsoft 業務代表或地區轉售商洽詢。

## <a name="next-steps"></a>後續步驟

- [混合式組織的 Azure Active Directory B2B 共同作業](hybrid-organizations.md)

- 如需 Azure AD Connect 的概觀，請參閱[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)。

