---
title: 開始使用 Azure Active Directory 憑證式驗證
description: 了解如何在環境中設定憑證式驗證
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: article
ms.date: 01/15/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: annaba
ms.openlocfilehash: 2f6b9ee7a23341f27272a4614bed6be5ae422ce0
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297261"
---
# <a name="get-started-with-certificate-based-authentication-in-azure-active-directory"></a>開始在 Azure Active Directory 中使用憑證式驗證

在將您的 Exchange Online 帳戶連線到下列各項時，憑證式驗證可讓您由 Azure Active Directory 透過用戶端憑證在 Windows、Android 或 iOS 裝置上驗證您的身分︰

- Microsoft 行動應用程式，例如 Microsoft Outlook 與 Microsoft Word
- Exchange ActiveSync (EAS) 用戶端

設定這項功能之後，就不需要在行動裝置上的特定郵件和 Microsoft Office 應用程式中，輸入使用者名稱和密碼的組合。

本主題內容：

- 提供為 Office 365 企業版、商務版、教育版、美國政府方案的租用戶使用者，設定和使用憑證式驗證的步驟。 在 Office 365 China、US Government Defense 及 US Government Federal 方案中，這項功能處於預覽版。
- 假設您已經設定[公開金鑰基礎結構 (PKI)](https://go.microsoft.com/fwlink/?linkid=841737) 和 [AD FS](../hybrid/how-to-connect-fed-whatis.md)。

## <a name="requirements"></a>需求

若要設定憑證式驗證，必須符合下列陳述：

- 只有瀏覽器應用程式或使用新式驗證 (ADAL) 之原生用戶端的同盟環境才支援憑證式驗證 (CBA)。 唯一的例外狀況是適用於 Exchange Online (EXO) 的 Exchange Active Sync (EAS)，可以用於同盟和受控兩種帳戶。
- 務必要在 Azure Active Directory 中設定根憑證授權單位和任何中繼憑證授權單位。
- 每個憑證授權單位都必須有一份可透過網際網路對應 URL 來參考的憑證撤銷清單 (CRL)。
- 您至少必須在 Azure Active Directory 中設定一個憑證授權單位。 您可以在[設定憑證授權單位](#step-2-configure-the-certificate-authorities)一節中找到相關步驟。
- (僅 Exchange ActiveSync 用戶端適用) 用戶端憑證必須將 Exchange Online 中可路由傳送的使用者電子郵件地址，放在 [主體別名] 欄位的 [主體名稱] 或 [RFC822 名稱] 值中。 Azure Active Directory 要將 RFC822 值對應到目錄中的 [Proxy 位址] 屬性。
- 您的用戶端裝置必須至少可以存取一個發出用戶端憑證的憑證授權單位。
- 用於戶端驗證的用戶端憑證必須已經發給您的用戶端。

## <a name="step-1-select-your-device-platform"></a>步驟 1︰選取裝置平台

第一個步驟中，針對您要處理的裝置平台，您需要檢閱下列項目︰

- Office 行動應用程式支援
- 特定的實作需求

下列裝置平台有相關的資訊︰

- [Android](active-directory-certificate-based-authentication-android.md)
- [iOS](active-directory-certificate-based-authentication-ios.md)

## <a name="step-2-configure-the-certificate-authorities"></a>步驟 2︰設定憑證授權單位

若要在 Azure Active Directory 中設定您的憑證授權單位，為每個憑證授權單位下載下列項目：

* 憑證的公開部分 (「.cer」  格式)
* 憑證撤銷清單 (CRl) 所在的網際網路對應 URL

憑證授權單位的結構描述看起來像這樣︰

    class TrustedCAsForPasswordlessAuth
    {
       CertificateAuthorityInformation[] certificateAuthorities;
    }

    class CertificateAuthorityInformation

    {
        CertAuthorityType authorityType;
        X509Certificate trustedCertificate;
        string crlDistributionPoint;
        string deltaCrlDistributionPoint;
        string trustedIssuer;
        string trustedIssuerSKI;
    }

    enum CertAuthorityType
    {
        RootAuthority = 0,
        IntermediateAuthority = 1
    }

設定時，您可以使用 [Azure Active Directory PowerShell 第 2 版](/powershell/azure/install-adv2?view=azureadps-2.0)：

1. 以系統管理員權限啟動 Windows PowerShell。
2. 安裝 Azure AD 模組 [2.0.0.33](https://www.powershellgallery.com/packages/AzureAD/2.0.0.33) 版或更新版本。

        Install-Module -Name AzureAD –RequiredVersion 2.0.0.33

設定的第一個步驟，您需要與您的租用戶建立連線。 一旦您與租用戶的連線存在，您可以檢閱、新增、刪除、修改在您的目錄中定義的受信任的憑證授權單位。

### <a name="connect"></a>連線

若要與您的租用戶建立連線，使用 [Connect-AzureAD](/powershell/module/azuread/connect-azuread?view=azureadps-2.0) Cmdlet︰

    Connect-AzureAD

### <a name="retrieve"></a>擷取

若要擷取您的目錄中所定義的受信任的憑證授權單位，使用 [Get-AzureADTrustedCertificateAuthority](/powershell/module/azuread/get-azureadtrustedcertificateauthority?view=azureadps-2.0) Cmdlet。

    Get-AzureADTrustedCertificateAuthority

### <a name="add"></a>加

若要建立受信任的憑證授權單位，使用 [New-AzureADTrustedCertificateAuthority](/powershell/module/azuread/new-azureadtrustedcertificateauthority?view=azureadps-2.0) Cmdlet 並將 **crlDistributionPoint** 屬性設為正確值：

    $cert=Get-Content -Encoding byte "[LOCATION OF THE CER FILE]"
    $new_ca=New-Object -TypeName Microsoft.Open.AzureAD.Model.CertificateAuthorityInformation
    $new_ca.AuthorityType=0
    $new_ca.TrustedCertificate=$cert
    $new_ca.crlDistributionPoint=”<CRL Distribution URL>”
    New-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $new_ca

### <a name="remove"></a>移除

若要移除受信任的憑證授權單位，使用 [Remove-AzureADTrustedCertificateAuthority](/powershell/module/azuread/remove-azureadtrustedcertificateauthority?view=azureadps-2.0) Cmdlet。

    $c=Get-AzureADTrustedCertificateAuthority
    Remove-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[2]

### <a name="modify"></a>修改

若要修改受信任的憑證授權單位，使用 [Set-AzureADTrustedCertificateAuthority](/powershell/module/azuread/set-azureadtrustedcertificateauthority?view=azureadps-2.0) Cmdlet。

    $c=Get-AzureADTrustedCertificateAuthority
    $c[0].AuthorityType=1
    Set-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[0]

## <a name="step-3-configure-revocation"></a>步驟 3︰設定撤銷

若要撤銷用戶端憑證，Azure Active Directory 會從和憑證授權單位資訊一起上傳的 URL 中，擷取憑證撤銷清單 (CRL) 並加以快取。 在 CRL 中，上次發佈的時間戳記 ([生效日期] 屬性) 是用來確保 CRL 依然有效。 定期參考 CRL 以撤銷對清單所列憑證的存取權。

如果需要立即撤銷 (例如，使用者遺失裝置)，可以讓使用者的授權權杖失效。 使用 Windows PowerShell 設定這位特定使用者的 **StsRefreshTokenValidFrom** 欄位，即可讓授權權杖失效。 您必須為想要撤銷其存取權的每位使用者更新其 **StsRefreshTokenValidFrom** 欄位。

為了確保撤銷持續有效，您必須將 CRL 的 [生效日期] 設定為 **StsRefreshTokenValidFrom** 所設值之後的日期，並確保有問題的憑證位於 CRL 中。

下列步驟概述藉由設定 **StsRefreshTokenValidFrom** 欄位，來更新授權權杖並讓它失效的程序。

**設定撤銷：**

1. 使用管理員認證連線到 MSOL 服務：

        $msolcred = get-credential
        connect-msolservice -credential $msolcred

2. 擷取使用者目前的 StsRefreshTokensValidFrom 值︰

        $user = Get-MsolUser -UserPrincipalName test@yourdomain.com`
        $user.StsRefreshTokensValidFrom

3. 將目前的時間戳記設定為使用者新的 StsRefreshTokensValidFrom 值︰

        Set-MsolUser -UserPrincipalName test@yourdomain.com -StsRefreshTokensValidFrom ("03/05/2016")

您設定的日期必須是未來的日期。 如果不是未來的日期，則不會設定 **StsRefreshTokensValidFrom** 屬性。 如果是未來的日期，才會將 **StsRefreshTokensValidFrom** 設定為目前的時間 (而非 Set-MsolUser 命令指示的日期)。

## <a name="step-4-test-your-configuration"></a>步驟 4︰測試組態

### <a name="testing-your-certificate"></a>測試您的憑證

您應該試著使用您**裝置上的瀏覽器**登入 [Outlook Web Access](https://outlook.office365.com) 或 [SharePoint Online](https://microsoft.sharepoint.com)，這是第一個組態測試。

如果登入成功，您便知道︰

- 使用者憑證已佈建到您的測試裝置
- AD FS 已正確設定

### <a name="testing-office-mobile-applications"></a>測試 Office 行動應用程式

**在您的行動 Office 應用程式上測試憑證式驗證︰**

1. 在您的測試裝置上，安裝 Office 行動應用程式 (例如 OneDrive)。
3. 啟動應用程式。
4. 輸入您的使用者名稱，然後選取想要使用的使用者憑證。

您應該可以順利登入。

### <a name="testing-exchange-activesync-client-applications"></a>測試 Exchange ActiveSync 用戶端應用程式

若要透過憑證式驗證來存取 Exchange ActiveSync (EAS)，必須要有包含用戶端憑證的 EAS 設定檔供應用程式使用。

EAS 設定檔必須包含下列資訊：

- 要用於驗證的使用者憑證

- EAS 端點 (例如，outlook.office365.com)

您可以用行動裝置管理 (MDM)，例如 Intune，在裝置上設定和放置 EAS 設定檔，或以手動方式將憑證放在裝置的 EAS 設定檔中。

### <a name="testing-eas-client-applications-on-android"></a>在 Android 上測試 EAS 用戶端應用程式

**測試憑證驗證：**

1. 設定應用程式中符合上一節需求的 EAS 設定檔。
2. 開啟應用程式，然後確認正在同步處理郵件。

## <a name="next-steps"></a>後續步驟

[Android 裝置上憑證式驗證的其他相關資訊。](active-directory-certificate-based-authentication-android.md)

[iOS 裝置上憑證式驗證的其他相關資訊。](active-directory-certificate-based-authentication-ios.md)