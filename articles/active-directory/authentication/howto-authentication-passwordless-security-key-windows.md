---
title: 無密碼安全性金鑰登入 Windows-Azure Active Directory
description: '瞭解如何使用 FIDO2 安全性金鑰來啟用無密碼安全性金鑰登入 Azure Active Directory (preview) '
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/24/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 04a46a691b2f629b64cfe09c22813b05c593af1c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743457"
---
# <a name="enable-passwordless-security-key-sign-in-to-windows-10-devices-with-azure-active-directory-preview"></a>使用 Azure Active Directory (preview 啟用無密碼安全性金鑰登入 Windows 10 裝置) 

本檔著重于如何使用 Windows 10 裝置來啟用 FIDO2 安全性金鑰型無密碼驗證。 在本文結尾，您將能夠使用 FIDO2 安全性金鑰，以 Azure AD 帳戶登入 Azure AD 和混合式 Azure AD 加入 Windows 10 裝置。

> [!NOTE]
> FIDO2 安全性金鑰是 Azure Active Directory 的公開預覽功能。 如需預覽的詳細資訊，請參閱  [Microsoft Azure 預覽的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="requirements"></a>規格需求

| 裝置類型 | 已聯結的 Azure AD | 已聯結的混合式 Azure AD |
| --- | --- | --- |
| [Azure AD Multi-Factor Authentication](howto-mfa-getstarted.md) | X | X |
| [合併的安全性資訊註冊預覽](concept-registration-mfa-sspr-combined.md) | X | X |
| 相容的 [FIDO2 安全性金鑰](concept-authentication-passwordless.md#fido2-security-keys) | X | X |
| WebAuthN 需要 Windows 10 1903 版或更高版本 | X | X |
| [Azure AD 已加入的裝置](../devices/concept-azure-ad-join.md) 需要 Windows 10 1909 版或更高版本 | X |   |
| [混合式 Azure AD 已加入的裝置](../devices/concept-azure-ad-join-hybrid.md) 需要 Windows 10 2004 版或更高版本 |   | X |
| 已完全修補的 Windows Server 2016/2019 網域控制站。 |   | X |
| [Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect) 1.4.32.0 版版或更新版本 |   | X |
| [Microsoft Intune](/intune/fundamentals/what-is-intune) (選擇性)  | X | X |
| 布建封裝 (選擇性)  | X | X |
| 群組原則 (選擇性)  |   | X |

### <a name="unsupported-scenarios"></a>不支援的案例

下列案例不受支援：

- Windows Server Active Directory Domain Services (AD DS) 加入網域 (僅限內部部署裝置) 部署。
- 使用安全性金鑰的 RDP、VDI 和 Citrix 案例。
- S/MIME 使用安全性金鑰。
- 使用安全性金鑰的「執行為」。
- 使用安全性金鑰登入伺服器。
- 如果您在線上時未使用安全性金鑰來登入您的裝置，您就無法使用它來登入或離線解除鎖定。
- 以包含多個 Azure AD 帳戶的安全性金鑰登入或解除鎖定 Windows 10 裝置。 此案例會利用最後一個新增至安全性金鑰的帳戶。 WebAuthN 可讓使用者選擇他們想要使用的帳戶。
- 將執行 Windows 10 版本1809的裝置解除鎖定。 若要獲得最佳體驗，請使用 Windows 10 1903 版或更高版本。

## <a name="prepare-devices-for-preview"></a>準備要預覽的裝置

您要在功能預覽期間試驗的 Azure AD 加入的裝置，必須執行 Windows 10 1909 版或更高版本。

混合式 Azure AD 加入的裝置必須執行 Windows 10 2004 版或更新版本。

## <a name="enable-security-keys-for-windows-sign-in"></a>啟用 Windows 登入的安全性金鑰

組織可能會選擇使用一或多個下列方法，根據組織的需求，讓 Windows 登入使用安全性金鑰：

- [使用 Intune 啟用](#enable-with-intune)
- [目標 Intune 部署](#targeted-intune-deployment)
- [使用布建套件啟用](#enable-with-a-provisioning-package)
- [僅啟用群組原則 (混合式 Azure AD 加入的裝置) ](#enable-with-group-policy)

> [!IMPORTANT]
> 已 **加入混合式 Azure AD 裝置** 的組織 **也** 必須完成文章中的步驟，在 Windows 10 FIDO2 安全性金鑰驗證運作之前，先 [啟用內部部署資源的 FIDO2 authentication](howto-authentication-passwordless-security-key-on-premises.md) 。
>
> 具有已 **加入 Azure AD 裝置** 的組織必須先完成此動作，裝置才能向 FIDO2 安全性金鑰驗證內部部署資源。

### <a name="enable-with-intune"></a>使用 Intune 啟用

若要使用 Intune 啟用安全性金鑰，請完成下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 流覽至 **Microsoft Intune**  >  **裝置註冊**  >  **Windows 註冊**  >  **Windows Hello 企業版**  >  **屬性**。
1. 在 [ **設定**] 底下，將 [ **使用安全性金鑰登入** ] 設定為 [ **啟用**]。

設定安全性金鑰以進行登入，並不會相依于設定 Windows Hello 企業版。

### <a name="targeted-intune-deployment"></a>目標 Intune 部署

若要以特定裝置群組為目標，以啟用認證提供者，請透過 Intune 使用下列自訂設定：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 流覽至 **Microsoft Intune**  >  **裝置配置**  >  **檔**  >  **建立設定檔**。
1. 使用下列設定來設定新的設定檔：
   - 名稱： Windows Sign-In 的安全性金鑰
   - 描述：啟用在 Windows 登入期間使用的 FIDO 安全性金鑰
   - 平台：Windows 10 及更新版本
   - 配置檔案類型：自訂
   - 自訂 OMA URI 設定：
      - 名稱：開啟 Windows Sign-In 的 FIDO 安全性金鑰
      - OMA-URI：./Device/Vendor/MSFT/PassportForWork/SecurityKey/UseSecurityKeyForSignin
      - 資料類型：整數
      - 值：1
1. 此原則可以指派給特定的使用者、裝置或群組。 如需詳細資訊，請參閱 [Microsoft Intune 中的指派使用者和裝置設定檔](/intune/device-profile-assign)。

![Intune 自訂裝置設定原則建立](./media/howto-authentication-passwordless-security-key/intune-custom-profile.png)

### <a name="enable-with-a-provisioning-package"></a>使用布建套件啟用

針對未受 Intune 管理的裝置，可以安裝布建套件以啟用此功能。 您可以從 [Microsoft Store](https://www.microsoft.com/p/windows-configuration-designer/9nblggh4tx22)安裝 Windows 設定設計工具應用程式。 請完成下列步驟以建立布建套件：

1. 啟動 Windows 設定設計工具。
1. 選取 **[** 檔案  >  **新增專案**]。
1. 為您的專案提供名稱，並記下建立專案的路徑，然後選取 **[下一步]**。
1. 將布建 *套件* 保留為選取的 **專案工作流程** ，然後選取 **[下一步]**。
1. 在 **[選擇要查看及設定的設定**] 底下選取 [*所有 Windows 桌上出版本*]，然後選取 **[下一步]**。
1. 選取 [完成]。
1. 在新建立的專案中，流覽至 **執行時間設定**  >  **WindowsHelloForBusiness**  >  **SecurityKeys**  >  **UseSecurityKeyForSignIn**。
1. 將 **UseSecurityKeyForSignIn** 設定為 [ *已啟用*]。
1. 選取 **匯出** 布建  >  **套件**
1. 在 [描述布建 **套件**] 下的 [**組建**] 視窗中保留預設值，然後選取 **[下一步]**。
1. 在 [建立布建 **套件的安全性詳細資料**] 下的 [**組建**] 視窗中保留預設值，然後選取 **[下一步**
1. 請在 [**選取要儲存** 布建套件的位置] 下的 [**組建**] 視窗中，或選取 [**下一步]** 來變更路徑。
1. 在 [組建布 **建套件**] 頁面上選取 [**建立**]。
1. 將 (*ppkg* 和 *貓*) 建立的兩個檔案儲存至您稍後可以將它們套用到電腦的位置。
1. 若要套用您建立的布建套件，請參閱套用布建 [套件](/windows/configuration/provisioning-packages/provisioning-apply-package)。

> [!NOTE]
> 執行 Windows 10 1903 版的裝置也必須啟用 (*EnableSharedPCMode*) 的共用電腦模式。 如需啟用此功能的詳細資訊，請參閱 [使用 Windows 10 設定共用或來賓電腦](/windows/configuration/set-up-shared-or-guest-pc)。

### <a name="enable-with-group-policy"></a>使用群組原則啟用

針對已 **加入混合式 Azure AD 的裝置**，組織可以設定下列群組原則設定，以啟用 FIDO 安全性金鑰登入。 您可以在 [**電腦** 設定] 底下找到此設定  >  **系統管理範本**  >  **系統**  >  **登**  >  **入安全性金鑰登入**：

- 將此原則設定為 [ **已啟用** ]，可讓使用者以安全性金鑰登入。
- 將此原則設定為 [ **已停** 用] 或 [ **未設定** ]，會阻止使用者以安全性金鑰登入。

此群組原則設定需要群組原則範本的更新版本 `CredentialProviders.admx` 。 這個新範本適用于下一版的 Windows Server 和 Windows 10 20H1。 這項設定可透過執行下列其中一種新版 Windows 的裝置來管理，或依照支援主題中的指導方針、 [如何建立和管理 Windows 中群組原則系統管理範本的集中存放區](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra)。

## <a name="sign-in-with-fido2-security-key"></a>使用 FIDO2 安全性金鑰登入

在下列範例中，名為 Bala Sandhu 的使用者已使用前一篇文章中的步驟來布建其 FIDO2 安全性金鑰，並 [啟用無密碼安全性金鑰登入](howto-authentication-passwordless-security-key.md#user-registration-and-management-of-fido2-security-keys)。 針對已加入混合式 Azure AD 的裝置，請確定您也已 [啟用無密碼安全性金鑰登入內部部署資源](howto-authentication-passwordless-security-key-on-premises.md)。 Bala 可以從 Windows 10 鎖定畫面選擇安全性金鑰認證提供者，然後插入安全性金鑰以登入 Windows。

![Windows 10 鎖定畫面上的安全性金鑰登入](./media/howto-authentication-passwordless-security-key/fido2-windows-10-1903-sign-in-lock-screen.png)

### <a name="manage-security-key-biometric-pin-or-reset-security-key"></a>管理安全性金鑰生物特徵辨識、PIN 或重設安全性金鑰

* Windows 10 版本1903或更高版本
   * 使用者可以在其裝置上開啟 **Windows 設定**>**帳戶**  >  **安全性金鑰**
   * 使用者可以變更 PIN 碼、更新生物識別，或重設其安全性金鑰

## <a name="troubleshooting-and-feedback"></a>疑難排解和意見反應

如果您想要在預覽這項功能時分享意見反應或遇到問題，請使用下列步驟透過 Windows 意見反應中樞應用程式共用：

1. 啟動 **意見反應中樞** ，並確認您已登入。
1. 在下列分類下提交意見反應：
   - 類別：安全性和隱私權
   - 子類別： FIDO
1. 若要捕獲記錄，請使用選項來 **重新建立我的問題**

## <a name="next-steps"></a>後續步驟

[啟用 Azure AD 和混合式 Azure AD 已加入裝置的內部部署資源存取權](howto-authentication-passwordless-security-key-on-premises.md)

[深入瞭解裝置註冊](../devices/overview.md)

[深入瞭解 Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)