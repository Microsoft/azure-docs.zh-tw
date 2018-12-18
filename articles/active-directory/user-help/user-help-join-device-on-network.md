---
title: 將您的工作裝置加入組織的網路 - Azure Active Directory | Microsoft Docs
description: 了解如何將您的工作裝置加入組織的網路。
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
ms.openlocfilehash: 34d3c12c83aeac7e92aa019abc38d9c4109883bd
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40180478"
---
# <a name="join-your-work-device-to-your-organizations-network"></a>將您的工作裝置加入組織的網路
將您工作用的 Windows 10 裝置加入組織的網路，如此就能存取可能受限的資源。

## <a name="what-happens-when-you-join-your-device"></a>當您加入裝置時會發生什麼事
當您將 Windows 10 裝置加入組織的網路時，將會發生下列動作：

- Windows 會向貴組織的網路註冊您的裝置，讓您可以使用個人帳戶存取您的資源。 註冊裝置之後，Windows 接著會將您的裝置加入網路，讓您能夠使用貴組織的使用者名稱和密碼來登入並存取受限的資源。

- (選擇性) 根據組織的選擇，可能會要求您透過 [Multi-Factor Authentication](multi-factor-authentication-end-user-first-time.md) 或[安全性資訊](user-help-security-info-overview.md)設定雙步驟驗證。

- (選擇性) 根據組織的選擇，您可能會在行動裝置管理 (例如 Microsoft Intune) 中自動註冊。 如需在 Microsoft Intune 中註冊的詳細資訊，請參閱[在 Intune 註冊您的裝置](https://docs.microsoft.com/intune-user-help/enroll-your-device-in-intune-all)。

- 您將使用透過組織帳戶的自動登入來完成登入程序。

## <a name="to-join-a-brand-new-windows-10-device"></a>加入全新的 Windows 10 裝置
如果您的裝置是全新且尚未設定的，您可以完成 Windows 全新體驗 (OOBE) 程序來將裝置加入網路。

1. 啟動新裝置並開始 OOBE 程序。

2. 在 [使用 Microsoft 帳戶登入] 畫面上，輸入您的公司或學校電子郵件地址。

    ![使用電子郵件地址的登入畫面](./media/user-help-join-device-on-network/join-device-oobe-signin.png)

3. 在 [輸入您的密碼] 畫面上，輸入您的密碼。

    ![[輸入您的密碼] 畫面](./media/user-help-join-device-on-network/join-device-oobe-password.png)

4. 在您的行動裝置上，核准您的裝置，讓它可以存取您的帳戶。 

    ![行動裝置通知畫面](./media/user-help-join-device-on-network/join-device-oobe-mobile.png)

5. 完成 OOBE 程序，包括設定隱私權設定，以及設定 Windows Hello (如有必要)。

    您的裝置現在已加入組織的網路。

## <a name="to-make-sure-youre-joined"></a>確定您已加入
您可以透過查看設定來確定您已加入。

1. 開啟 [設定]，然後選取 [帳戶]。

    ![[設定] 畫面上的帳戶](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. 選取 [存取公司或學校資源]，然後確定您看到類似**已連線到 *<your_organization>* Azure AD** 的文字。

    ![具有已連線之 contoso 帳戶的 [存取公司或學校資源] 畫面](./media/user-help-join-device-on-network/join-device-oobe-verify.png)


## <a name="to-join-an-already-configured-windows-10-device"></a>加入已經設定的 Windows 10 裝置
如果您已擁有裝置一段時間且已設定，則您可以依照下列步驟將裝置加入網路。

1. 開啟 [設定]，然後選取 [帳戶]。

2. 選取 [存取公司或學校資源]，然後選取 [連線]。

    ![[存取公司或學校資源] 與 [連線] 連結](./media/user-help-join-device-on-network/join-device-access-work-school-connect.png)

3. 在 [設定公司或學校帳戶] 畫面上，選取 [將此裝置加入 Azure Active Directory]。

    ![設定公司或學校帳戶畫面](./media/user-help-join-device-on-network/join-device-setup-join-aad.png)

4. 在 [讓我們協助您登入帳戶] 畫面上，輸入您的電子郵件地址 (例如 alain@contoso.com)，然後選取 [下一步]。

    ![[讓我們協助您登入帳戶] 畫面](./media/user-help-join-device-on-network/join-device-setup-get-signed-in.png)

5. 在 [輸入密碼] 畫面上，輸入您的密碼，然後選取 [登入]。

    ![輸入密碼](./media/user-help-join-device-on-network/join-device-setup-password.png)

6. 在您的行動裝置上，核准您的裝置，讓它可以存取您的帳戶。 

    ![行動裝置通知畫面](./media/user-help-join-device-on-network/join-device-setup-mobile.png)

7. 在 [確定這是您的組織] 畫面上，檢閱資訊以確定它是正確的，然後選取 [加入]。

    ![[確定這是您的組織] 驗證畫面](./media/user-help-join-device-on-network/join-device-setup-confirm.png)

8. 在 [您已就緒] 畫面上，按一下 [完成]。

    ![[您已就緒] 畫面](./media/user-help-join-device-on-network/join-device-setup-finish.png)

## <a name="to-make-sure-youre-joined"></a>確定您已加入
您可以透過查看設定來確定您已加入。

1. 開啟 [設定]，然後選取 [帳戶]。

    ![[設定] 畫面上的帳戶](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. 選取 [存取公司或學校資源]，然後確定您看到類似**已連線到 *<your_organization>* Azure AD** 的文字。

    ![具有已連線之 contoso 帳戶的 [存取公司或學校資源] 畫面](./media/user-help-join-device-on-network/join-device-setup-verify.png)

## <a name="next-steps"></a>後續步驟
將裝置加入組織的網路之後，您應該能夠使用公司或學校帳戶資訊來存取您的所有資源。

- 如果您的組織想要您註冊個人裝置 (例如您的手機)，請參閱[在組織的網路上註冊個人裝置](user-help-register-device-on-network.md)。

