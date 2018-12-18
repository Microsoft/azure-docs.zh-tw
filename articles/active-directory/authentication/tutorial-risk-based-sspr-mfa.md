---
title: 風險型 MFA 和 SSPR 與 Azure Identity Protection
description: 在本教學課程中，您將啟用 Multi-Factor Authentication 和自助式密碼重設的 Azure Identity Protection 整合，以減少有風險的行為。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: fb9ec69476253eaa559fe763dcc2c92994505602
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39163260"
---
# <a name="tutorial-use-risk-events-to-trigger-multi-factor-authentication-and-password-changes"></a>教學課程：使用風險事件觸發 Multi-Factor Authentication 和密碼變更

在本教學課程中，您將啟用的 Azure Active Directory (Azure AD) Identity Protection 的功能，這是一項不只是監視和報告工具的 Azure AD Premium P2 功能。 若要保護組織的身分識別，您可以設定會自動因應風險行為的風險型原則。 這些原則可以自動進行封鎖或起始補救措施，包括要求密碼變更和強制執行 Multi-Factor Authentication。

Azure AD Identity Protection 原則可以與現有的條件式存取原則搭配使用，而提供多一層的保護。 您的使用者可能永遠不會觸發需要這些原則的風險行為，但身為系統管理員的您知道他們都受到保護。

可能觸發風險事件的項目包括：

* 認證外洩的使用者
* 從匿名 IP 位址登入
* 不可能到達非典型位置的移動
* 從受感染的裝置登入
* 從具有可疑活動的 IP 位址登入
* 從不熟悉的位置登入

如需 Azure AD Identity Protection 的詳細資訊，請參閱[什麼是 Azure AD Identity Protection](../active-directory-identityprotection.md) 一文

> [!div class="checklist"]
> * 啟用 Azure MFA 註冊
> * 啟用風險型密碼變更
> * 啟用風險型 Multi-Factor Authentication

## <a name="prerequisites"></a>必要條件

* 能夠存取可運作且至少已被指派試用版 Azure AD Premium P2 授權的 Azure AD 租用戶。
* 在 Azure AD 租用戶中具有全域系統管理員權限的帳戶。
* 已完成先前的自助式密碼重設 (SSPR) 和 Multi-Factor Authentication (MFA) 教學課程。

## <a name="enable-risk-based-policies-for-sspr-and-mfa"></a>啟用 SSPR 和 MFA 的風險型原則

啟用風險型原則是很簡單的程序。 下列步驟將引導您完成範例設定。

### <a name="enable-users-to-register-for-multi-factor-authentication"></a>讓使用者註冊 Multi-Factor Authentication

Azure AD Identity Protection 包含預設原則，可協助您讓使用者進行 Multi-Factor Authentication 的註冊，並輕鬆識別目前的註冊狀態。 啟用此原則後並不會隨即要求使用者執行 Multi-Factor Authentication，但會要求他們預先註冊。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 按一下 [所有服務]，然後瀏覽至 [Azure AD Identity Protection]。
1. 按一下 [MFA 註冊]。
1. 將 [強制執行原則] 設定為 [開啟]。
   1. 設定此原則後，會要求您的所有使用者註冊準備要以 Multi-Factor Authentication 使用的方法。
1. 按一下 [檔案] 。

   ![要求使用者在登入時使用 Azure AD Identity Protection 註冊 MFA](./media/tutorial-risk-based-sspr-mfa/risk-based-require-mfa-registration.png)

### <a name="enable-risk-based-password-changes"></a>啟用風險型密碼變更

Microsoft 與研究人員、執法機關，Microsoft 的各個安全性小組和其他受信任的來源合作，以找出使用者名稱和密碼的配對。 當其中一個配對符合您環境中的帳戶時，就可以使用下列原則觸發風險型密碼變更。

1. 按一下使用者風險原則。
1. 在 [條件] 下方選取 [使用者風險]，然後選擇 [中 (含) 以上]。
1. 按一下 [選取]，然後按一下 [完成]。
1. 在 [存取] 下方選擇 [允許存取]，然後選取 [需要變更密碼]。
1. 按一下 [選取]
1. 將 [強制執行原則] 設定為 [開啟]。
1. 按一下 [儲存] 

### <a name="enable-risk-based-multi-factor-authentication"></a>啟用風險型 Multi-Factor Authentication

大部分使用者都有可追蹤的正常行為，而當他們超出此標準範圍時，光是允許他們登入就可能會有風險。 您可以封鎖該使用者，或只是要求他們執行 Multi-Factor Authentication 以證明他們的真實身分。 若要在偵測到有風險的登入時啟用需要 MFA 的原則，請啟用下列原則。

1. 按一下登入風險原則
1. 在 [條件] 下方選取 [使用者風險]，然後選擇 [中 (含) 以上]。
1. 按一下 [選取]，然後按一下 [完成]。
1. 在 [存取] 下方選擇 [允許存取]，然後選取 [需要多重要素驗證]。
1. 按一下 [選取]
1. 將 [強制執行原則] 設定為 [開啟]。
1. 按一下 [儲存] 

## <a name="clean-up-resources"></a>清除資源

如果您已完成測試，且不再需要啟用風險型原則，請回到您要停用的每個原則，並將 [強制執行原則] 設定為 [關閉]。