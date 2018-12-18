---
title: 啟用 Azure AD SSPR 試驗
description: 在本教學課程中，您將為試驗使用者群組啟用 Azure AD 自助式密碼重設
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: a86547ad3eddb57328a2a0358ac453c979b84d37
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39163301"
---
# <a name="tutorial-complete-an-azure-ad-self-service-password-reset-pilot-roll-out"></a>教學課程：完成 Azure AD 自助式密碼重設試驗推出

在本教學課程中，您將在組織中啟用 Azure AD 自助式密碼重設 (SSPR) 的試驗推出，並使用非系統管理員帳戶進行測試。

自助式密碼重設的任何測試都務必以非系統管理員帳戶來執行。 Microsoft 會管理系統管理員帳戶的密碼重設原則，並要求使用更嚴密的驗證方法。 此原則不允許使用安全性問題和解答，而必須使用兩種方法進行重設。

> [!div class="checklist"]
> * 啟用自助式密碼重設
> * 以使用者身分測試 SSPR

## <a name="prerequisites"></a>必要條件

* 全域管理員帳戶

## <a name="enable-self-service-password-reset"></a>啟用自助式密碼重設

1. 使用全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 **Azure Active Directory**，然後選取 [密碼重設]。
1. 為組織中的使用者子集啟用自助式密碼，而先以試驗群組執行。
   * 在 [屬性] 頁面的 [已啟用自助式密碼重設] 下方，選擇 [已選取]，然後選擇試驗群組。
      * 只有您所選擇的特定 Azure AD 群組成員可以使用 SSPR 功能。 我們建議您定義使用者群組，並在部署此功能以用於概念證明時，使用這項設定。 這裡支援安全性群組的巢狀。
      * 確定您所選群組中的使用者已獲得適當的授權。
   * 按一下 [儲存] 
1. 在 [驗證方法] 頁面上
   * 將 [重設所需的方法數] 設定為 **2**
   * 選擇組織想要允許的 [使用者可用方法]。 在本教學課程中，請核取會啟用 [電子郵件]、[行動電話] 和 [辦公室電話] 的方塊。
   * 按一下 [儲存] 
1. 在 [註冊] 窗格上
   * 針對 [登入時要求使用者註冊]，選取 [是]。
   * 將 [要求使用者重新確認其驗證資訊的等候天數] 設定為 **180**。
   * 按一下 [儲存] 
1. 在 [通知] 頁面上
   * 將 [通知使用者密碼重設] 選項設定為 [是]。
   * 將 [當其他系統管理員重設其密碼時通知所有系統管理員] 設定為 [是]。
1. 在 [自訂] 頁面上
   * Microsoft 建議您將 [自訂技術服務人員連結] 設定為 [是]，並在 [自訂的技術服務人員電子郵件或 URL] 欄位中提供可讓使用者從您的組織中取得其他協助的電子郵件地址或網頁 URL。
   * 本教學課程中，我們會將 [自訂技術服務人員連結] 保留為 [否]。

現在，我們已為試驗群組中的雲端使用者設定自助式密碼重設。

## <a name="test-sspr-as-a-user"></a>以使用者身分測試 SSPR

使用屬於試驗群組成員的非系統管理員測試使用者，來測試自助式密碼重設。 **請注意，如果您使用已指派有任何系統管理員角色的帳戶，則驗證方法和數目可能會與您在 Microsoft 管理系統管理員原則時所選取的不同。**

1. 開啟新的 InPrivate 或 Incognito 模式瀏覽器視窗。
1. 使用位於 [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup) 的註冊入口網站，以測試使用者註冊自助式密碼重設。
1. 使用相同的測試使用者瀏覽至自助式密碼重設入口網站 [https://aka.ms/sspr](https://aka.ms/sspr)，並嘗試使用您在上一個步驟中提供的資訊重設密碼。
1. 您應該能夠成功重設密碼。

## <a name="clean-up-resources"></a>清除資源

如果您決定不再使用您在本教學課程中設定的功能，請進行下列變更。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 **Azure Active Directory**，然後選取 [密碼重設]。
1. 在 [屬性] 頁面的 [已啟用自助式密碼重設] 下方，選擇 [無]。
1. 按一下 [儲存] 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已啟用 Azure AD 自助式密碼重設。 請繼續進行下一個教學課程，以了解如何將內部部署 Active Directory Domain Services 基礎結構整合到自助式密碼重設體驗中。

> [!div class="nextstepaction"]
> [啟用 SSPR 內部部署回寫整合](tutorial-enable-writeback.md)
