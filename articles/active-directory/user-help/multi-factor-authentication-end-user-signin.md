---
title: 使用雙步驟驗證的 Azure MFA 登入 | Microsoft Docs
description: 本頁面將指引您移至何處才能看到 Azure MFA 可用的各種登入方法。
keywords: 使用者驗證, 登入經驗, 使用行動電話登入, 使用辦公室電話登入
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: b310b762-471b-4b26-887a-a321c9e81d46
ms.workload: identity
ms.service: active-directory
ms.component: user-help
ms.topic: conceptual
ms.date: 04/02/2017
ms.author: lizross
ms.reviewer: librown
ms.openlocfilehash: e7886d3c8aa1ef9ee1e392030aabf0d187568e24
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343061"
---
# <a name="the-sign-in-experience-with-azure-multi-factor-authentication"></a>Azure Multi-Factor Authentication 的登入體驗
> [!NOTE]
> 本文的目的逐步解說典型的登入體驗。 如需登入的說明或疑難排解問題，請參閱 [使用 Azure Multi-Factor Authentication 時碰到困難](multi-factor-authentication-end-user-troubleshoot.md)。

## <a name="what-will-your-sign-in-experience-be"></a>您的登入體驗將會如何？
您的登入體驗會根據您選擇來做為第二個要素的項目而不同︰撥打電話、驗證應用程式或簡訊。 選擇最符合您的情況的一個選項：

| 您要如何登入？ |
| --- |
| [透過電話撥打我的行動或辦公室電話](#signing-in-with-a-phone-call) |
| [傳送簡訊到我的行動電話號碼](#signing-in-with-a-text-message)
| [從 Microsoft 驗證器應用程式使用通知](#signing-in-with-the-microsoft-authenticator-app-using-notification) |
| [從 Microsoft 驗證器應用程式使用驗證碼](#signing-in-with-the-microsoft-authenticator-app-using-verification-code) |
| [使用替代方法，因為現在無法使用我慣用的方法](#signing-in-with-an-alternate-method) |

## <a name="signing-in-with-a-phone-call"></a>透過撥打電話來登入
下列資訊將說明透過撥打您的行動或辦公室電話來進行雙步驟驗證體驗。

1. 使用您的使用者名稱和密碼登入應用程式或服務，例如 Office 365。  
2. Microsoft 打電話給您。  
3. 請接聽電話並按 # 鍵。  

## <a name="signing-in-with-a-text-message"></a>透過簡訊登入
下列資訊將說明透過傳送簡訊到您的行動電話來進行雙步驟驗證體驗：

1. 使用您的使用者名稱和密碼登入應用程式或服務，例如 Office 365。
2. Microsoft 會傳送包含數字代碼的簡訊給您。
3. 請在登入頁面上提供的方塊中輸入此代碼。

## <a name="signing-in-with-the-microsoft-authenticator-app"></a>使用 Microsoft 驗證器應用程式登入
下列資訊將說明使用 Microsoft Authenticator 應用程式進行雙步驟驗證的體驗。 應用程式有兩種不同的使用方式。 您可以在裝置上接收推送通知，或是開啟應用程式以取得驗證碼。

### <a name="to-sign-in-with-a-notification-from-the-microsoft-authenticator-app"></a>從 Microsoft 驗證器應用程式使用通知登入
1. 使用您的使用者名稱和密碼登入應用程式或服務，例如 Office 365。
2. Microsoft 會將通知傳送到您裝置上的 Microsoft 驗證器應用程式。

  ![Microsoft 傳送通知](./media/multi-factor-authentication-end-user-signin/notify.png)

3. 在電話上開啟通知，然後選取 [驗證] 鍵。 如果貴公司要求 PIN，在此處輸入。
4. 您現在應已登入。

### <a name="to-sign-in-using-a-verification-code-with-the-microsoft-authenticator-app"></a>使用驗證碼登入 Microsoft 驗證器應用程式

如果您使用 Microsoft Authenticator 應用程式取得驗證碼，則當您開啟應用程式時，會在您的帳戶名稱下面看到一個數字。 這個數字每 30 秒會變更一次，所以您不會使用相同的數字兩次。 當系統要求您輸入驗證碼時，開啟應用程式並使用當下所顯示的任何數字。

1. 使用您的使用者名稱和密碼登入應用程式或服務，例如 Office 365。
2. Microsoft 提示您輸入驗證碼。

  ![輸入驗證碼](./media/multi-factor-authentication-end-user-signin/verify3.png)

3. 在電話上開啟 Microsoft 驗證器應用程式，並在登入的方塊中輸入驗證碼。

## <a name="signing-in-with-an-alternate-method"></a>使用替代方法登入
有時候您設定做為慣用驗證方法的電話或裝置不在手邊。 這種情況就是為什麼我們建議您為帳戶設定備份方法。 下一節示範當主要方法無法使用時，如何使用替代方法進行登入。

1. 使用您的使用者名稱和密碼登入應用程式或服務，例如 Office 365。
2. 選取 [使用不同的驗證選項]。 根據您已設定的驗證選項數量而定，您會看到不同的驗證選項。
3. 選擇替代方法並且登入。

  ![使用替代方法](./media/multi-factor-authentication-end-user-signin/alt.png)

## <a name="next-steps"></a>後續步驟

如果您碰到雙步驟驗證登入的問題，可從[使用 Azure Multi-Factor Authentication 時碰到困難](multi-factor-authentication-end-user-troubleshoot.md)獲得詳細資訊。

了解如何[管理雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)。

了解如何[開始使用 Microsoft 驗證器應用程式](microsoft-authenticator-app-how-to.md)，如此一來，因此您可以使用通知登入，而不是經由文字訊息和來電。
