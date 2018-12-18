---
title: Azure AD SSPR 和 MFA 的聚合式註冊 (公開預覽)
description: Azure AD Multi-Factor Authenticaiton 和自助密碼重設註冊 (公開預覽)
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry, michmcla
ms.openlocfilehash: af57faddcc1413747b4bb847e27287ba86562175
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141304"
---
# <a name="converged-registration-for-self-service-password-reset-and-azure-multi-factor-authentication-public-preview"></a>自助密碼重設和 Azure Multi-Factor Authentication 的聚合式註冊 (公開預覽)

直到目前為止，使用者都必須在兩個不同的入口網站中註冊 Azure Multi-Factor Authentication (MFA) 和自助密碼重設 (SSPR) 的驗證方法。 由於 Azure MFA 和 SSPR 兩者使用的方法相似，許多使用者因此被混淆，而並未在兩個入口網站中註冊。 此差異會導致某些使用者在需要時卻無法使用 Azure MFA 或 SSPR，結果必須致電服務台，而可能讓使用者感到挫折。 現在，使用者只需註冊一次便可取得 Azure MFA 和 SSPR 的優點，不再需要為這些功能註冊兩次驗證方法。  

在您為貴組織啟用此新體驗之前，建議您先檢閱此文章以及我們的[使用者文件](https://aka.ms/securityinfoguide)，以了解此體驗將對使用者造成的影響。 您可以利用[使用者文件](https://aka.ms/securityinfoguide)來訓練使用者，讓他們準備好使用新體驗，並確保新體驗能順利推出。

|     |
| --- |
| 自助密碼重設和 Azure Multi-Factor Authentication 的聚合式註冊是 Azure Active Directory 的公開預覽功能。 如需有關預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

若要讓使用者在單一體驗中註冊 Azure Multi-Factor Authentication 和自助密碼重設的驗證方法，請完成下列步驟：

1. 以全域系統管理員或使用者系統管理員身分登入 Azure 入口網站。
2. 瀏覽至 [Azure Active Directory]、[使用者設定]、[管理存取面板預覽功能的設定]。
3. 在 [使用者可以使用預覽功能來登錄及管理安全性資訊] 底下，您可以選擇要針對 [已選取] 使用者群組或 [所有] 使用者來啟用。

使用者現在可以移至 [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo) 以針對 MFA 和 SSPR 註冊驗證方法。 若要深入了解您的使用者將在此新體驗中看到的內容，請參閱[使用者文件](https://aka.ms/securityinfoguide)。  

> [!NOTE]
> 一旦您啟用此體驗，透過新體驗註冊或確認電話號碼或行動裝置應用程式的使用者，將可以利用那些資料來使用 MFA 和 SSPR (如果那些方法已在 MFA 和 SSPR 原則中啟用)。 接著，如果您停用此體驗，瀏覽至先前 SSPR 註冊頁面 (aka.ms/ssprsetup) 的使用者將必須執行 MFA，然後才能存取頁面。  

## <a name="how-it-works"></a>運作方式

如果使用者之前已透過個別的 MFA 和 SSPR 註冊體驗註冊了驗證方法，他們就不需要再次註冊該資訊。 不過，如果您的要求需要使用者註冊 MFA 或 SSPR，則他們在登入時可能會看到檢閱安全性資訊的提示。

如果使用者已註冊可用於 MFA 的方法，系統會提示他們執行 MFA，然後才能存取新體驗。

如果您強制執行 MFA 或 SSPR 的註冊，而使用者尚未完成註冊，則系統會在他們登入時顯示註冊提示。

登入時收到註冊提示的使用者，將會看到下列體驗：

![聚合式註冊。 以新使用者的身分設定方法](./media/concept-registration-mfa-sspr-converged/concept-registration-add-methods.png)

> [!NOTE]
> 此體驗只有在使用者於登入時收到註冊提示時才會顯示。 直接前往新體驗 (aka.ms/setupsecurityinfo) 的使用者將會看到不同版本的體驗，本文稍後將會描述。

根據您的 MFA 或 SSPR 原則中啟用的方法，顯示的驗證方法將會有所不同。 系統會要求使用者必須註冊最小數目的驗證方法，以符合 MFA 原則、SSPR 原則 (或兩者皆是) 的規範。 如果使用者可註冊的驗證方法有彈性，則他們可以選取 [選擇安全性資訊] 來選擇其他驗證方法。  

> [!NOTE]
> 如果您允許使用行動裝置應用程式通知和行動裝置應用程式驗證碼，使用通知註冊 Microsoft Authenticator 應用程式的使用者，就能使用通知和驗證碼來確認其身分識別。

不同於先前的 MFA 註冊體驗，使用者進行新的註冊體驗時，不會收到要註冊應用程式密碼提示。 相反地，他們應依照應用程式密碼教學課程中所列出的步驟，在新的體驗中註冊應用程式密碼。  

當使用者完成註冊時，將會自動設定他們的預設 MFA 方法。 如果使用者已註冊驗證器應用程式，則預設方法將會設定為該應用程式。 如果使用者並未註冊驗證器應用程式，而且只有註冊其電話號碼，則預設方法將會設為撥打電話。 使用者可以移至 [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo)，並選取 [變更預設] 來變更預設方法。  

如果沒有強制執行註冊，使用者就可以在 [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo) 管理他們自己的驗證方法。 如果使用者先前已註冊可用於執行 MFA 的方法，系統會要求他們執行 MFA，然後才能存取頁面。  

![聚合式註冊。 以已註冊的使用者身分編輯方法](./media/concept-registration-mfa-sspr-converged/concept-registration-edit-methods.png)

在此頁面上，使用者會看到先前註冊的驗證方法和他們所註冊的驗證方法 (例如辦公室電話)。 使用者也可以新增、編輯或刪除其驗證方法 (不包含辦公室電話)。  

此新體驗的稽核記錄檔存在於稽核記錄檔的「驗證方法」類別底下。  

## <a name="known-issues"></a>已知問題

**使用者使用簡訊註冊手機時，預設的 MFA 方法會設為撥打電話**

   * 有些使用者可能會發現，當他們使用簡訊註冊電話號碼之後，他們的預設 MFA 方法會設為撥打電話。 使用者可以依照[管理安全性資訊 (預覽)](../user-help/security-info-manage-settings.md#change-your-info) 一文中的指示變更預設方法，以解決此問題。

**系統管理員停用預設方法後，使用者無法存取新的註冊體驗**

   * 如果有些使用者先前註冊的預設 MFA 方法已被系統管理員停用，則他們就無法存取新的註冊體驗。 以下是範例案例：
      1. 使用者先前已註冊電話號碼，並將預設方法設定為撥打電話。
      2. 系統管理員停用了以撥打電話作為租用戶的 MFA 方法。
      3. 使用者在登入時會收到提示，因為他們需要註冊其他方法才能滿足租用戶的 SSPR 原則。
      4. 使用者嘗試註冊，但無法存取頁面，因為沒有設定的預設方法，而卡在迴圈當中。

## <a name="next-steps"></a>後續步驟

[了解如何部署 Azure AD 自助密碼重設](howto-sspr-deployment.md)

[了解如何要求登入時進行多重要素驗證](howto-mfa-getstarted.md)

[使用者驗證方法設定文件](https://aka.ms/securityinfoguide)