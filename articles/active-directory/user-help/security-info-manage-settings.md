---
title: 管理安全性資訊 - Azure Active Directory | Microsoft Docs
description: 了解如何管理安全性資訊，包括如何進行雙步驟驗證設定。
services: active-directory
author: eross-msft
manager: mtillman
ms.reviewer: sahenry
ms.service: active-directory
ms.component: user-help
ms.workload: identity
ms.topic: conceptual
ms.date: 07/30/2018
ms.author: lizross
ms.openlocfilehash: abd2984574f80f03f276861782ff9ee51348d07e
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39391372"
---
# <a name="manage-your-security-info-preview"></a>管理您的安全性資訊 (預覽)

[!INCLUDE[preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

您可以使用您的安全性資訊，登入公司或學校帳戶或重設密碼。

當您登入時，根據貴組織的設定，您可能會看到表示「X 天內不要再問我」的核取方塊。 此核取方塊可讓您在系統管理員允許的天數內保持已登入裝置狀態，而不需要驗證。

## <a name="change-your-info"></a>變更您的資訊
根據系統管理員和貴組織所允許的內容，您可以更新或新增安全性資訊，或變更預設值。

### <a name="to-change-your-info"></a>若要變更您的資訊：

1. 登入您的公司或學校帳戶。

2. 移至 myapps.microsoft.com，從頁面的右上角選取您的名稱，然後選取 [設定檔]。

3. 在 [管理帳戶] 區域中，選取 [編輯安全性資訊]。

    ![已醒目提示 [編輯安全性資訊] 連結的 [設定檔] 畫面](media/security-info/security-info-profile.png)

4. 請使用預設方法來核准存取權並查看您目前的安全性資訊詳細資料，前提是您的系統管理員已為您的組織設定這項體驗。

5. 在 [保護您的帳戶安全] 頁面上，您可以：

    - 選取 [新增安全性資訊] 來新增額外的方法。

    - 選取 [變更預設值] 來變更預設方法。

    - 選取現有方法旁邊的**鉛筆**圖示來更新您的資訊。

    ![包含現有可編輯資訊的安全性資訊畫面](media/security-info/security-info-edit.png)

6. 在進行變更之後，即可離開此頁面，您的變更便會儲存。

如果您沒有看到這些選項，或無法存取 myapps.microsoft.com 頁面，有可能是您的組織使用自訂選項或自訂頁面。 您必須連絡您的系統管理員，以取得更多協助。

## <a name="manage-your-security-info-for-a-lost-or-potentially-compromised-device"></a>針對已遺失或可能遭到入侵的裝置管理您的安全性資訊

如果您遺失了裝置或裝置遭到入侵，您必須對所有先前受信任的裝置重新執行驗證程序。

### <a name="to-manage-your-security-info-for-lost-or-potentially-compromised-devices"></a>若要針對已遺失或可能遭到入侵的裝置管理您的安全性資訊：

1. 登入您的公司或學校帳戶。

2. 移至 myapps.microsoft.com，從頁面的右上角選取您的名稱，然後選取 [設定檔]。

3. 在 [管理帳戶] 區域中，選取 [忘記已記住裝置上的 MFA]。
    
    選擇這個選項表示在您登入之後，必須再次進行 Multi-Factor Authentication 程序。

    ![已醒目提示忘記連結的 [設定檔] 畫面](media/security-info/security-info-forget.png)

## <a name="common-problems-and-solutions-with-your-security-info"></a>安全性資訊的常見問題和解決方案

本文協助您針對安全性資訊進行疑難排解，包括雙步驟驗證相關問題。

|問題|解決方法|
|-------|--------|
|我沒帶手機|有可能是您沒有隨身攜帶手機，但仍然想要登入您的公司或學校帳戶。 若要修正此問題，您可以使用其他不需要手機的驗證方法登入，例如您的電子郵件地址或辦公室電話號碼。 若要將其他方法新增至您的安全性資訊，請依照[變更您的資訊](#change-your-info)一節中的步驟執行。|
|我的手機遺失或遭竊|不幸的是，您的電話可能遺失或遭竊。 在此情況下，強烈建議您讓貴組織了解此情況，讓 IT 人員可以重設應用程式密碼，並從您信任的裝置清單中清除所有已記住的裝置。 您也可以遵循[針對已遺失或可能遭到入侵的裝置管理您的安全性資訊](#manage-your-security-info-for-a-lost-or-potentially-compromised-device)一節中的步驟，忘記自有信任的裝置。|
|我收到新的電話號碼|修正此問題的方法有二種。 您可以使用不需要手機號碼的替代驗證方法 (例如電子郵件) 登入，如果沒有這個選項，您可以連絡貴組織的 IT 人員，請他們清除您的設定。 若要將其他方法新增至您的安全性資訊，請依照[變更您的資訊](#change-your-info)一節中的步驟執行。|
|我的預設方法錯誤|您可以在安全性選項中更新您的預設方法。 如需特定詳細資訊，您可以移至[變更您的資訊](#change-your-info)一節。|
|我的行動裝置沒收到簡訊或來電|如果您的行動裝置過去成功收到過簡訊或來電，則這個問題很可能是電話提供者的問題，而非您帳戶的問題。 請確定您的手機收訊良好，而且能夠接收簡訊和電話。 您可以要求朋友打電話或傳簡訊給您，作為測試。<br><br>如果您可以成功接收簡訊和電話訊息，但仍然沒有取得通知，可以嘗試使用不同的方法。 您可以依照[變更您的資訊](#change-your-info)一節中的步驟執行，將其他方法新增至您的安全性資訊。 如果您沒有其他方法可新增，可以連絡公司支援人員並要求他們清除您的設定，才可以在下次登入時設定您的方法。<br><br>如果您經常因收訊不良而發生問題，建議您在行動裝置上使用 Microsoft Authenticator 應用程式。 應用程式可以產生您用來登入的隨機安全驗證碼，而且這些代碼不需要任何手機訊號或網際網路連接。 如需 Microsoft Authenticator 應用程式的詳細資訊，請參閱[開始使用 Microsoft Authenticator 應用程式](https://docs.microsoft.com/azure/multi-factor-authentication/end-user/microsoft-authenticator-app-how-to)一文。|
|此表中沒有任何選項可解決我的問題|如果您嘗試過這些疑難排解步驟，但仍遭遇問題，請連絡您的公司支援人員，他們應該能夠協助您。|

## <a name="next-steps"></a>後續步驟

- 深入了解[安全性資訊 (預覽) 概觀](user-help-security-info-overview.md)中的安全性資訊。

- 深入了解[雙步驟驗證概觀](user-help-two-step-verification-overview.md)一文中的雙步驟驗證。 

- 請遵循下列其中一篇操作方式文章，深入了解如何在安全性資訊區域中設定您的裝置：

    - [將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)

    - [將安全性資訊設定為使用電話](security-info-setup-phone-number.md)

    - [將安全性資訊設定為使用簡訊](security-info-setup-text-msg.md)

    - [將安全性資訊設定為使用電子郵件](security-info-setup-email.md)

    - [將安全性資訊設定為使用安全性問題](security-info-setup-questions.md)

- 如果您已遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中取得登入問題的疑難排解秘訣和說明。