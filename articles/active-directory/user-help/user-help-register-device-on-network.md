---
title: 在組織的網路上註冊您的個人裝置 - Azure Active Directory | Microsoft Docs
description: 了解如何在組織的網路上註冊您的個人裝置，讓您可以存取組織的受保護資源。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: user-help
ms.workload: identity
ms.topic: conceptual
ms.date: 08/03/2018
ms.author: lizross
ms.reviewer: jairoc
ms.openlocfilehash: 7126a47bd90168c7d86fe9fcc05fab0a60955063
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40180477"
---
# <a name="register-your-personal-device-on-your-organizations-network"></a>在組織的網路上註冊您的個人裝置
在組織的網路上註冊您的個人裝置，通常是手機或平板電腦。 註冊您的裝置之後，它將能夠存取組織的受限資源。

>[!Note]
>本文基於示範目的使用 Windows 裝置，但您也可以註冊執行 iOS、Android 或 macOS 的裝置。

## <a name="what-happens-when-you-register-your-device"></a>當您註冊裝置時會發生什麼事
當您在組織的網路上註冊裝置時，將會發生下列動作：

- Windows 會在組織的網路上註冊您的裝置。

- (選擇性) 根據組織的選擇，可能會要求您透過 [Multi-Factor Authentication](multi-factor-authentication-end-user-first-time.md) 或[安全性資訊](user-help-security-info-overview.md)設定雙步驟驗證。

- (選擇性) 根據組織的選擇，您可能會在行動裝置管理 (例如 Microsoft Intune) 中自動註冊。 如需在 Microsoft Intune 中註冊的詳細資訊，請參閱[在 Intune 註冊您的裝置](https://docs.microsoft.com/intune-user-help/enroll-your-device-in-intune-all)。

- 您將使用個人 Microsoft 帳戶的使用者名稱與密碼來完成登入程序。

## <a name="to-register-your-windows-device"></a>註冊您的 Windows 裝置

依照下列步驟在網路上註冊您的個人裝置。

1. 開啟 [設定]，然後選取 [帳戶]。

    ![[設定] 畫面上的帳戶](./media/user-help-register-device-on-network/register-device-settings-accounts.png)

2. 選取 [電子郵件與帳戶]，然後選取 [加入 Microsoft 帳戶]。

    ![[電子郵件與帳戶] 與 [新增 Microsoft 帳戶] 連結](./media/user-help-register-device-on-network/register-device-email-and-accounts.png)

3. 在 [新增您的 Microsoft 帳戶] 畫面上，輸入適用於您個人 Microsoft 帳戶的電子郵件地址。

    ![[新增您的 Microsoft 帳戶] 畫面 (使用電子郵件)](./media/user-help-register-device-on-network/register-device-add-accounts.png)

4. 在 [輸入密碼] 畫面上，輸入您個人 Microsoft 帳戶的密碼，然後選取 [登入]。

    ![[輸入密碼] 畫面](./media/user-help-register-device-on-network/register-device-enter-password.png)

5. 完成剩餘的註冊程序，包括核准您的身分識別驗證要求 (如果您使用雙步驟驗證)，以及設定 Windows Hello (如有必要)。

## <a name="to-make-sure-youre-registered"></a>確定您已註冊
您可以透過查看設定來確定您已註冊。

1. 開啟 [設定]，然後選取 [帳戶]。

    ![[設定] 畫面上的帳戶](./media/user-help-register-device-on-network/register-device-settings-accounts.png)

2. 選取 [電子郵件與帳戶]，並確定您看到您的個人 Microsoft 帳戶。

    ![具有已連線之 contoso 帳戶的 [存取公司或學校資源] 畫面](./media/user-help-register-device-on-network/register-device-verify-account.png)

## <a name="next-steps"></a>後續步驟
將您的個人裝置註冊到組織的網路之後，您應該能夠存取您的大部分資源。

- 如果組織想要您加入您的工作裝置，請參閱[將您的工作裝置加入組織的網路](user-help-join-device-on-network.md)。



