---
title: 快速入門︰Azure AD 自助式密碼重設
description: 在本快速入門中，您將快速設定 Azure AD 自助式密碼重設，以允許使用者重設自己的密碼
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: quickstart
ms.date: 07/17/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: c40cb3192d514d990ea2a5d66e1484ff204e9b10
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39223552"
---
# <a name="quickstart-self-service-password-reset"></a>快速入門：自助式密碼重設

在本快速入門中，您會逐步設定自助式密碼重設 (SSPR) 以提供簡單的方法，讓 IT 系統管理員允許使用者重設密碼或解除鎖定帳戶。

## <a name="prerequisites"></a>必要條件

* 可運作且至少已啟用試用版授權的 Azure AD 租用戶。
* 具有全域系統管理員權限的帳戶。
* 您已知其密碼的非系統管理員測試使用者；如果您需要建立使用者，請參閱[快速入門：將新的使用者新增至 Azure Active Directory](../add-users-azure-active-directory.md) 一文。
* 該名非系統管理員測試使用者所屬的測試用試驗群組；如果您需要建立群組，請參閱[在 Azure Active Directory 中建立群組和新增使用者](../active-directory-groups-create-azure-portal.md)一文。

## <a name="enable-self-service-password-reset"></a>啟用自助式密碼重設

> [!VIDEO https://www.youtube.com/embed/Pa0eyqjEjvQ]

1. 從您現有的 Azure AD 租用戶，在 [Azure 入口網站] 的 [Azure Active Directory] 下選取 [密碼重設]。

2. 在 [屬性] 頁面的 [已啟用自助式密碼重設] 下方，選擇 [已選取]。
    * 在 [選取群組] 中，選擇您在本文的先決條件一節中建立的試驗群組。
    * 按一下 [檔案] 。

3. 在 [驗證方法] 頁面中，選擇下列項目：
   * 需要重設的方法數：**1**
   * 使用者可用方法：
      * **行動電話**
      * **辦公室電話**
   * 按一下 [檔案] 。

    ![驗證][Authentication]

4. 在 [註冊] 頁面中，選擇下列項目：
   * 登入時要求使用者註冊：**是**
   * 設定要求使用者重新確認其驗證資訊的等候天數：**365**

## <a name="test-self-service-password-reset"></a>測試自助式密碼重設

接著我們將以測試使用者測試您的 SSPR 組態。 由於 Microsoft 會強制執行 Azure 系統管理員帳戶的強式驗證需求，使用系統管理員帳戶的測試可能會有不同的結果。 如需有關系統管理員密碼原則的詳細資訊，請參閱我們的[密碼原則](concept-sspr-policy.md)一文。

1. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)。
2. 以非系統管理員測試使用者的身分登入，並註冊您的驗證電話。
3. 完成後，按一下標示為**狀況良好**的按鈕，然後關閉瀏覽器視窗。
4. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://aka.ms/sspr](https://aka.ms/sspr)。
5. 輸入非系統管理員測試使用者的使用者識別碼，以及 CAPTCHA 中的字元，然後按 [下一步]。
6. 依照驗證步驟重設您的密碼

## <a name="clean-up-resources"></a>清除資源

要停用自助式密碼重設很容易。 開啟您的 Azure AD 租用戶並移至 [密碼重設] > [屬性]，然後選取 [已啟用自助式密碼重設] 之下的 [無]。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何為僅限雲端的使用者快速設定自助式密碼重設。 若要了解如何完成更詳盡的推出，請繼續進行我們的推出指南。

> [!div class="nextstepaction"]
> [推出自助式密碼重設](howto-sspr-deployment.md)

[Authentication]: ./media/quickstart-sspr/sspr-authentication-methods.png "可供使用的 Azure AD 驗證方法和所需的數量"
