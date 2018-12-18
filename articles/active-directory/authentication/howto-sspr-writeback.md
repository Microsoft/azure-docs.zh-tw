---
title: 如何為 Azure AD SSPR 設定密碼回寫
description: 使用 Azure AD 和 Azure AD Connect 將密碼回寫到內部部署目錄
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 1ae74f7c43e763962224683954b28e5941136c08
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295813"
---
# <a name="how-to-configure-password-writeback"></a>如何：設定密碼回寫

使用密碼回寫時，我們建議您使用 [Azure AD Connect](../hybrid/how-to-connect-install-express.md) 的自動更新功能。

下列步驟假設您已使用[快速](../hybrid/how-to-connect-install-express.md)或[自訂](../hybrid/how-to-connect-install-custom.md)設定在您的環境中設定 Azure AD Connect。

1. 若要設定和啟用密碼回寫，請登入 Azure AD Connect 伺服器，然後啟動 **Azure AD Connect** 設定精靈。
2. 在 [歡迎] 頁面上，選取 [設定]。
3. 在 [其他工作] 頁面上，選取 [自訂同步處理選項]，然後選取 [下一步]。
4. 在 [連線到 Azure AD] 頁面上，輸入全域管理員認證，然後選取 [下一步]。
5. 在 [連線目錄] 和 [網域/OU] 篩選頁面上，選取 [下一步]。
6. 在 [選用功能] 頁面上，選取 [密碼回寫] 旁邊的方塊，然後選取 [下一步]。
   ![在 Azure AD Connect 中啟用密碼回寫][Writeback]
7. 在 [準備好設定] 頁面上，選取 [設定]，然後等待程序完成。
8. 看到設定完成時，選取 [結束]。

如需了解有關密碼回寫的常見疑難排解工作，請參閱我們疑難排解文章中的[針對密碼回寫問題進行疑難排解](active-directory-passwords-troubleshoot.md#troubleshoot-password-writeback)一節。

## <a name="active-directory-permissions"></a>Active Directory 權限

如果您想要在 SSPR 的範圍中，Azure AD Connect 公用程式中指定的帳戶必須設定好下列項目：

* **重設密碼** 
* **變更密碼** 
* `lockoutTime` 的**寫入權限**
* `pwdLastSet` 的**寫入權限**
* 下列其中一項的**延伸權限**：
   * 該樹系中「每個網域」的根物件
   * 要在 SSPR 範圍中的使用者組織單位 (OU)

如果您不確定上述所指的是哪個帳戶，請開啟 Azure Active Directory Connect 組態 UI，然後選取 [檢視目前的設定] 選項。 您需要新增權限的帳戶會列在 [同步處理的目錄] 底下。

如果您設定這些權限，每個樹系的 MA 服務帳戶便可代表該樹系內的使用者帳戶管理密碼。 

> [!IMPORTANT]
> 如果您忘了指派這些權限，則即使回寫看似設定正確，但當使用者嘗試從雲端管理其內部部署密碼時，還是會遇到錯誤。
>

> [!NOTE]
> 最多可能需要一小時或更久，這些權限才會複寫至您目錄中的所有物件。
>

若要設定適當權限以進行密碼回寫，請完成下列步驟：

1. 以具有適當網域管理權限的帳戶開啟 [Active Directory 使用者及電腦]。
2. 從 [檢視] 功能表中，確定已開啟 [進階功能]。
3. 在左面板中，於代表網域根目錄的物件上按一下滑鼠右鍵，然後選取 [屬性] > [安全性] > [進階]。
4. 從 [權限] 索引標籤中，選取 [新增]。
5. 挑選要套用權限的帳戶 (從 Azure AD Connect 安裝程式)。
6. 在 [套用至] 下拉式清單中，選取 [下階使用者] 物件。
7. 在 [權限] 底下，核取下列方塊：
    * **重設密碼**
    * **變更密碼**
    * **寫入 lockoutTime**
    * **寫入 pwdLastSet**
8. 選取 [套用]/[確定] 以套用變更並結束任何開啟的對話方塊。

## <a name="next-steps"></a>後續步驟

[什麼是密碼回寫？](concept-sspr-writeback.md)

[Writeback]: ./media/howto-sspr-writeback/enablepasswordwriteback.png "在 Azure AD Connect 中啟用密碼回寫"
