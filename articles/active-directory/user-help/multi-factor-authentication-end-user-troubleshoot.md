---
title: 針對雙步驟驗證進行疑難排解 - Azure Active Directory | Microsoft Docs
description: 為使用者提供處理 Azure Multi-Factor Authentication 和雙步驟驗證問題的指示。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: 8f3aef42-7f66-4656-a7cd-d25a971cb9eb
ms.workload: identity
ms.service: active-directory
ms.component: user-help
ms.topic: conceptual
ms.date: 07/16/2018
ms.author: lizross
ms.reviewer: kexia
ms.openlocfilehash: fd722da236f3e1fff18e2561756635cd95f098e0
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39346518"
---
# <a name="get-help-with-two-step-verification"></a>取得雙步驟驗證的說明

雙步驟驗證是貴組織用來保護您帳戶的安全性功能。 雙步驟驗證比只是使用密碼更安全，因為它依賴兩種形式的驗證：分別是您知道的資訊和您擁有的東西。 您知道的資訊就是密碼，而您擁有的東西就是手機或裝置。 使用雙步驟驗證有助於防止惡意入侵的駭客以您的身分登入，即使他們取得您的密碼也無法登入。

雖然 Microsoft 會提供雙步驟驗證，但是會由貴組織決定您是否要使用此功能。 如果貴組織要求您使用此功能，您無法選擇取消使用，就像是您無法選擇取消使用密碼來保護帳戶。

>[!Note]
>如果您需要有關在個人 Microsoft 帳戶上使用雙步驟驗證的詳細資訊，請參閱[關於雙步驟驗證](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification)一文。

## <a name="why-do-i-need-to-use-another-verification-method"></a>為何需要使用另一個驗證方法？

有許多原因會讓您可能需要使用替代驗證方法來登入帳戶。 例如︰

- **忘記攜帶手機或裝置。** 有些天您會將手機留在家裡，但仍需要登入工作。 此時，您應該會先嘗試透過不需要手機的其他方法來登入。

- **遺失手機，或是有新的電話號碼。** 如果您的手機遺失或有新號碼之後，可以使用不同方法登入，或要求系統管理員清除您的設定。 我們強烈建議您讓系統管理員知道您的手機已遺失或遭竊，系統管理員才能您的帳戶進行適當更新。 清除您的設定之後，下次登入時，系統會提示您重新[註冊雙步驟驗證](multi-factor-authentication-end-user-first-time.md)。

- **未收到驗證簡訊或來電。** 有幾個原因可能會造成您無法收到簡訊或來電。 如果您過去成功收到過簡訊或來電，則這可能是電話提供者問題，而非帳戶問題。 如果您經常因收訊不良而發生延遲，則建議在行動裝置上使用 [Microsoft Authenticator 應用程式](microsoft-authenticator-app-how-to.md)。 此應用程式可以產生用於登入的隨機安全驗證碼，無須任何手機訊號或網際網路連線。<br><br>如果您正嘗試接收文字訊息，請讓朋友傳送文字訊息給您，以測試您確實可以收到文字訊息；如果您收到多個訊息，請務必使用最新收到的那一組代碼。

- **應用程式密碼無效。** 針對不支援雙步驟驗證的較舊桌面應用程式，應用程式密碼會取代您的一般密碼。 請先確定您輸入的密碼正確。 如果未獲得修正，請嘗試登入[建立新的應用程式密碼](multi-factor-authentication-end-user-app-passwords.md)，或是連絡系統管理員，請他[刪除您現有的應用程式密碼](../authentication/howto-mfa-userdevicesettings.md)，讓您可以建立新密碼。

## <a name="sign-in-using-another-verification-method"></a>使用另一個驗證方法登入

1. 正常登入您的帳戶，然後在 [雙步驟驗證] 頁面上選擇 [以另一種方式登入] 的連結。

    ![變更登入驗證方法](./media/multi-factor-authentication-end-user-troubleshoot/two-factor-auth-signin-another-way.png)

    >[!Note]
    >如果您沒有看到 [以另一種方式登入] 的連結，表示您尚未設定任何其他的驗證方法。 您必須連絡系統管理員，請他協助您登入帳戶。 登入之後，請務必新增額外的驗證方法。 如需新增驗證方法的詳細資訊，請參閱[管理雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)一文。<br><br>如果您看到連結，但仍看不到任何其他驗證方法時，您必須連絡系統管理員，請他協助您登入帳戶。

2. 選擇您的替代驗證方法，並繼續雙步驟驗證的程序。

3. 回到您的帳戶之後，您可以更新驗證方法 (如有必要)。 如需新增或變更方法的詳細資訊，請參閱[管理雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)一文。

## <a name="i-didnt-find-an-answer-to-my-problem"></a>我找不到我的問題的解答

如果您已嘗試過這些步驟，但仍遭遇問題時，請連絡系統管理員，以取得更多協助。

## <a name="related-topics"></a>相關主題

* [管理您的雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)

* [Microsoft Authenticator 應用程式常見問題集](microsoft-authenticator-app-faq.md)
