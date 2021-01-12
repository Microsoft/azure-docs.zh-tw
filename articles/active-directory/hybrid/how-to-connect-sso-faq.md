---
title: Azure AD Connect：無縫單一登入 - 常見問題集 | Microsoft Docs
description: Azure Active Directory 無縫單一登入常見問題集的答案。
services: active-directory
keywords: 何謂 Azure AD Connect、安裝 Active Directory、Azure AD、SSO、單一登入的必要元件
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/07/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2dac4b461d4506015f0ef374eae37f67c445791d
ms.sourcegitcommit: 48e5379c373f8bd98bc6de439482248cd07ae883
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98107866"
---
# <a name="azure-active-directory-seamless-single-sign-on-frequently-asked-questions"></a>Azure Active Directory 無縫單一登入：常見問題集

本文將會解決 Azure Active Directory 無縫單一登入 (無縫 SSO) 的常見問題。 請隨時回來查看新內容。

**問：無縫 SSO 與哪些登入方法搭配運作**

無縫 SSO 可以與[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)或[傳遞驗證](how-to-connect-pta.md)登入方法合併使用。 然而，此功能無法搭配 Active Directory Federation Services (ADFS) 使用。

**問：無縫 SSO 是免費功能嗎？**

無縫 SSO 是免費功能，您不需要任何付費的 Azure AD 版本即可使用。

**問：[Microsoft Azure 德國雲端](https://www.microsoft.de/cloud-deutschland)和 [Microsoft Azure Government 雲端](https://azure.microsoft.com/features/gov/)是否有提供「無縫 SSO」功能？**

無縫 SSO 適用於 Azure Government 雲端。 如需詳細資訊，請參閱 [Azure Government 的混合式身分識別考量](./reference-connect-government-cloud.md)。

**問：哪些應用程式利用無縫 SSO 的 `domain_hint` 或 `login_hint` 參數功能？**

以下是一份不完整的應用程式清單，這些應用程式可以將這些參數傳送給 Azure AD，因此可使用「無縫 SSO」(亦即，使用者不需要輸入自己的使用者名稱或密碼) 為使用者提供無訊息的登入體驗：

| 應用程式名稱 | 要使用的應用程式 URL |
| -- | -- |
| 存取面板 | https:\//myapps.microsoft.com/contoso.com |
| 網路版 Outlook | https:\//outlook.office365.com/contoso.com |
| Office 365 入口網站 | https:\//portal.office.com?domain_hint=contoso.com, https:\//www.office.com?domain_hint=contoso.com |

此外，如果應用程式將登入要求傳送至設定為租用戶的 Azure AD 端點，使用者就會獲得無訊息登入體驗 - 也就是 https:\//login.microsoftonline.com/contoso.com/<..> or https:\//login.microsoftonline.com/<tenant_ID>/<..> - 而不是 Azure AD 的通用端點 - 也就是 https:\//login.microsoftonline.com/common/<...>。 以下是一份不完整的應用程式清單，列出會提出這類登入要求的應用程式。

| 應用程式名稱 | 要使用的應用程式 URL |
| -- | -- |
| SharePoint Online | https:\//contoso.sharepoint.com |
| Azure 入口網站 | https:\//portal.azure.com/contoso.com |

在上表中，請以您的網域名稱取代 "contoso.com"，以連至您租用戶的正確應用程式 URL。

如果想要將無訊息登入使用在其他應用程式上，請以意見反應區段告知。

**問：無縫 SSO 支援 `Alternate ID` 作為使用者名稱，而不是 `userPrincipalName`？**

是。 如[這裡](how-to-connect-install-custom.md)所述設定於 Azure AD Connect 時，無縫 SSO 支援 `Alternate ID` 作為使用者名稱。 並非所有 Microsoft 365 的應用程式都支援 `Alternate ID` 。 請參閱支援陳述式的特定應用程式文件。

**問：[Azure AD 聯結](../devices/overview.md)與「無縫 SSO」所提供的單一登入體驗有何差異？**

[Azure AD 聯結](../devices/overview.md)可為裝置已向 Azure AD 註冊的使用者提供 SSO。 這些裝置不一定要加入網域。 提供 SSO 時，使用的是「主要重新整理權杖」(或稱 *PRT*)，而不是 Kerberos。 在 Windows 10 裝置上可獲得最佳使用者體驗。 SSO 在 Microsoft Edge 瀏覽器上會自動執行。 在 Chrome 上則藉由使用瀏覽器擴充功能也能運作。

您可以在租用戶上同時使用「Azure AD 聯結」和「無縫 SSO」。 這兩個功能是互補的。 如果同時開啟這兩個功能，則來自「Azure AD 聯結」的 SSO 優先順序會高於「無縫 SSO」。

**問：我想要使用 Azure AD 註冊非 Windows 10 裝置，而不需要使用 AD FS。我可以改為使用無縫 SSO 嗎？**

是，此案例需要 2.1 版或更新版本的[加入工作場所用戶端](https://www.microsoft.com/download/details.aspx?id=53554)。

**問：如何才能變換 `AZUREADSSO` 電腦帳戶的 Kerberos 解密金鑰？**

請務必經常變換在內部部署 AD 樹系中建立之 `AZUREADSSO` 電腦帳戶 (這表示 Azure AD) 的 Kerberos 解密金鑰。

>[!IMPORTANT]
>強烈建議您至少每隔 30 天變換一次 Kerberos 解密金鑰。

請在執行 Azure AD Connect 的內部部署伺服器上依照下列步驟操作：

   > [!NOTE]
   >您將需要網域系統管理員和全域管理員認證，才能執行下列步驟。
   >如果您不是網域系統管理員，而且您已被網域系統管理員指派許可權，您應該呼叫 `Update-AzureADSSOForest -OnPremCredentials $creds -PreserveCustomPermissionsOnDesktopSsoAccount`

   **步驟 1.取得已啟用無縫 SSO 的 AD 樹系清單**

   1. 首先，下載並安裝 [Azure AD PowerShell](/powershell/azure/active-directory/overview)。
   2. 瀏覽到 `%programfiles%\Microsoft Azure Active Directory Connect` 資料夾。
   3. 使用此命令匯入順暢 SSO PowerShell 模組：`Import-Module .\AzureADSSO.psd1`。
   4. 以系統管理員身分執行 PowerShell。 在 PowerShell 中，呼叫 `New-AzureADSSOAuthenticationContext`。 此命令應提供一個快顯視窗，以便輸入租用戶的全域管理員認證。
   5. 呼叫 `Get-AzureADSSOStatus | ConvertFrom-Json`。 此命令會提供已啟用這項功能的 AD 樹系清單 (查看 [網域] 清單)。

   **步驟 2.在已設定 Kerberos 解密金鑰的每個 AD 樹系上更新該金鑰**

   1. 呼叫 `$creds = Get-Credential`。 出現提示時，輸入預定 Azure AD 樹系的網域系統管理員認證。

   > [!NOTE]
   >必須以 SAM 帳戶名稱格式 (contoso\johndoe 或 contoso.com\johndoe) 輸入網域系統管理員認證使用者名稱。 我們會利用使用者名稱的網域部分，使用 DNS 來尋找網域系統管理員的網域控制站。

   >[!NOTE]
   >使用的網域系統管理員帳戶不得為「受保護的使用者」群組的成員。 若是如此，作業會失敗。

   2. 呼叫 `Update-AzureADSSOForest -OnPremCredentials $creds`。 此命令會更新此特定 AD 樹系中 `AZUREADSSO` 電腦帳戶的 Kerberos 解密金鑰，並且在 Azure AD 中更新它。
   
   3. 針對您已設定此功能的每個 AD 樹系，重複上述步驟。
   
  >[!NOTE]
   >如果您要更新的樹系不是 Azure AD Connect 的樹系，請確定可以使用與通用類別目錄伺服器的連線 (TCP 3268 和 TCP 3269) 。

   >[!IMPORTANT]
   >確定您「並未」執行 `Update-AzureADSSOForest` 命令一次以上。 否則，此功能會停止運作，直到使用者的 Kerberos 票證過期並由您的內部部署 Active Directory 重新發出為止。

**問：如何停用無縫 SSO？**

   **步驟 1.在租用戶上停用功能**

   **選項 A：使用 Azure AD Connect 來停用**
    
   1. 執行 Azure AD Connect，選擇 [變更使用者登入頁面]，然後按 [下一步]。
   2. 取消選取 [啟用單一登入] 選項。 繼續執行精靈。

   完成精靈之後，租用戶就會停用無縫 SSO。 但是，您會在畫面上看到一個包含以下內容的訊息：

   「單一登入現已停用，但還要執行其他手動步驟才可完成清理。 深入了解」

   若要完成清理程序，請在執行 Azure AD Connect 的內部部署伺服器上依照步驟 2 和 3 來操作。

   **選項 B：使用 PowerShell 來停用**

   請在執行 Azure AD Connect 的內部部署伺服器上執行下列步驟：

   1. 首先，下載並安裝 [Azure AD PowerShell](/powershell/azure/active-directory/overview)。
   2. 瀏覽到 `%programfiles%\Microsoft Azure Active Directory Connect` 資料夾。
   3. 使用此命令匯入順暢 SSO PowerShell 模組：`Import-Module .\AzureADSSO.psd1`。
   4. 以系統管理員身分執行 PowerShell。 在 PowerShell 中，呼叫 `New-AzureADSSOAuthenticationContext`。 此命令應提供一個快顯視窗，以便輸入租用戶的全域管理員認證。
   5. 呼叫 `Enable-AzureADSSO -Enable $false`。
   
   此時，已停用無縫 SSO，但如果您想要啟用無縫 SSO，網域仍會保持設定。 如果您想要完全移除無縫 SSO 設定中的網域，請在完成上述步驟5之後，呼叫下列 Cmdlet： `Disable-AzureADSSOForest -DomainFqdn <fqdn>` 。

   >[!IMPORTANT]
   >使用 PowerShell 停用無縫 SSO 不會變更 Azure AD Connect 中的狀態。 在 [變更使用者登入] 頁面中，無縫 SSO 會顯示為已啟用。

   **步驟 2.取得已啟用無縫 SSO 的 AD 樹系清單**

   如果您已使用 Azure AD Connect 停用無縫 SSO，請依照下方的工作 1 到 4 操作。 如果您改為使用 PowerShell 停用無縫 SSO，請跳至下方的工作 5。

   1. 首先，下載並安裝 [Azure AD PowerShell](/powershell/azure/active-directory/overview)。
   2. 瀏覽到 `%programfiles%\Microsoft Azure Active Directory Connect` 資料夾。
   3. 使用此命令匯入順暢 SSO PowerShell 模組：`Import-Module .\AzureADSSO.psd1`。
   4. 以系統管理員身分執行 PowerShell。 在 PowerShell 中，呼叫 `New-AzureADSSOAuthenticationContext`。 此命令應提供一個快顯視窗，以便輸入租用戶的全域管理員認證。
   5. 呼叫 `Get-AzureADSSOStatus | ConvertFrom-Json`。 此命令會提供已啟用這項功能的 AD 樹系清單 (查看 [網域] 清單)。

   **步驟 3.從您看到的每個列出 AD 樹系，手動刪除 `AZUREADSSO` 電腦帳戶。**

## <a name="next-steps"></a>後續步驟

- [**快速入門**](how-to-connect-sso-quick-start.md) - 開始使用 Azure AD 無縫 SSO。
- [**技術性深入探討**](how-to-connect-sso-how-it-works.md) - 了解這項功能的運作方式。
- [**疑難排解**](tshoot-connect-sso.md) - 了解如何解決此功能的常見問題。
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) - 用於提出新的功能要求。
