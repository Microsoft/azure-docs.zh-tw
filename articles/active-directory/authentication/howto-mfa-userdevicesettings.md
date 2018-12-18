---
title: 系統管理員管理使用者和裝置 - Azure MFA | Microsoft Docs
description: 說明如何變更使用者設定，如強迫使用者重複 proof-up 程序。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 2f73d9795ba807e5901568507ad2fae5b001c91a
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39161284"
---
# <a name="manage-user-settings-with-azure-multi-factor-authentication-in-the-cloud"></a>管理雲端中 Azure Multi-Factor Authentication 的使用者設定

身為管理員，您可以管理下列使用者和裝置設定：

* 要求使用者再次提供連絡方法
* 刪除應用程式密碼
* 要求在所有受信任的裝置上使用 MFA 

## <a name="require-users-to-provide-contact-methods-again"></a>要求使用者再次提供連絡方法
此設定會強制使用者再次完成註冊程序。 如果使用者擁有非瀏覽器應用程式的應用程式密碼，這些應用程式仍可繼續運作。  您也可以選取 [ **刪除選定使用者產生的所有現有應用程式密碼**] 來刪除使用者的應用程式密碼。

### <a name="how-to-require-users-to-provide-contact-methods-again"></a>如何要求使用者再次提供連絡方法
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側，選取 [Azure Active Directory] > [使用者和群組] > [所有使用者]。
3. 選取 [多重要素驗證]。 多重要素驗證頁面隨即開啟。 
4. 勾選您想要管理之使用者旁邊的方塊。 快速步驟選項清單隨即出現在右邊。 
5. 選取 [管理使用者設定]。
6. 勾選 [要求選取的使用者再次提供連絡方法]  方塊。
   ![提供連絡方法](./media/howto-mfa-userdevicesettings/reproofup.png)
7. 按一下 [儲存] 。
8. 按一下 [關閉]。

## <a name="delete-users-existing-app-passwords"></a>刪除使用者現有的應用程式密碼
此設定會刪除使用者建立的所有應用程式密碼。 與這些應用程式密碼相關的非瀏覽器應用程式會停止運作，直到建立新應用程式密碼為止。

### <a name="how-to-delete-users-existing-app-passwords"></a>如何刪除使用者的現有應用程式密碼
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側，選取 [Azure Active Directory] > [使用者和群組] > [所有使用者]。
3. 選取 [多重要素驗證]。 多重要素驗證頁面隨即開啟。 
6. 勾選您想要管理之使用者旁邊的方塊。 快速步驟選項清單隨即出現在右邊。 
7. 選取 [管理使用者設定]。
8. 勾選 [刪除選定使用者產生的所有現有應用程式密碼]  方塊。
   ![刪除應用程式密碼](./media/howto-mfa-userdevicesettings/deleteapppasswords.png)
9. 按一下 [儲存] 。
10. 按一下 [關閉]。

## <a name="restore-mfa-on-all-remembered-devices-for-a-user"></a>在使用者已記住的所有裝置上還原 MFA
Azure Multi-Factor Authentication 的可設定功能之一，是讓使用者可以選擇將裝置標示為受信任。 如需詳細資訊，請參閱[設定 Azure Multi-Factor Authentication 設定](howto-mfa-mfasettings.md#remember-multi-factor-authentication-for-devices-that-users-trust)。

使用者可以在其一般的裝置上設定天數內選擇退出雙步驟驗證。 如果帳戶遭到洩露或信任的裝置遺失，您必須能夠移除受信任的狀態，且再次要求進行雙步驟驗證。

**在所有已記住的裝置上還原 Multi-Factor Authentication** 設定，代表使用者在下次登入時受到查問需進行兩步驟驗證，無論他們是否選擇將其裝置標示為受信任。 

### <a name="how-to-restore-mfa-on-all-suspended-devices-for-a-user"></a>如何還原使用者所有暫停之裝置上的 MFA
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側，選取 [Azure Active Directory] > [使用者和群組] > [所有使用者]。
3. 選取 [多重要素驗證]。 多重要素驗證頁面隨即開啟。 
6. 勾選您想要管理之使用者旁邊的方塊。 快速步驟選項清單隨即出現在右邊。 
7. 選取 [管理使用者設定]。
8. 勾選 [在所有記住的裝置上還原多重要素驗證]
    方塊![刪除應用程式密碼](./media/howto-mfa-userdevicesettings/rememberdevices.png)
9. 按一下 [儲存] 。
10. 按一下 [關閉]。

## <a name="next-steps"></a>後續步驟

- 取得如何[設定 Azure Multi-Factor Authentication 設定](howto-mfa-mfasettings.md)的詳細資訊

- 如果您的使用者需要協助，請將他們指向[雙步驟驗證的使用者指南](../user-help/multi-factor-authentication-end-user.md)
