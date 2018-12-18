---
title: 針對 SAML 型單一登入進行偵錯 - Azure Active Directory | Microsoft Docs
description: 針對 Azure Active Directory 中應用程式的 SAML 型單一登入進行偵錯。
services: active-directory
author: CelesteDG
documentationcenter: na
manager: mtillman
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/15/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: hirsin, dastrock, smalser
ms.openlocfilehash: 5895e918fbd3b67074069ccc81bd794a75725c9a
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39620931"
---
# <a name="debug-saml-based-single-sign-on-to-applications-in-azure-active-directory"></a>針對 Azure Active Directory 中應用程式的 SAML 型單一登入進行偵錯

了解如何在 Azure Active Directory (Azure AD) 中支援[安全性聲明標記語言 (SAML) 2.0](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) 的應用程式內尋找[單一登入](../manage-apps/what-is-single-sign-on.md)問題，並加以修正。 

## <a name="before-you-begin"></a>開始之前
建議您安裝 [My Apps 安全登入擴充功能](../user-help/active-directory-saas-access-panel-user-help.md#i-am-having-trouble-installing-the-my-apps-secure-sign-in-extension)。 此瀏覽器擴充功能可輕鬆地收集 SAML 要求和 SAML 回應資訊，以供您解決單一登入問題。 如果您無法安裝此擴充功能，本文會說明如何在已安裝/未安裝此擴充功能的情況下解決問題。

若要下載並安裝 My Apps 安全登入擴充功能，請使用下列其中一個連結。

- [Chrome](https://go.microsoft.com/fwlink/?linkid=866367)
- [Edge](https://go.microsoft.com/fwlink/?linkid=845176)
- [Firefox](https://go.microsoft.com/fwlink/?linkid=866366)


## <a name="test-saml-based-single-sign-on"></a>測試 SAML 型單一登入

若要在 AAD 與目標應用程式之間測試 SAML 型單一登入：

1.  以全域系統管理員或其他已獲得授權可管理應用程式的系統管理員身分，登入 [Azure 入口網站](https://portal.azure.com)。
2.  在左側的刀鋒視窗中，按一下 [Azure Active Directory]，然後按一下 [企業應用程式]。 
3.  從企業應用程式清單中，按一下要測試單一登入的應用程式，然後在左邊的選項中按一下 [單一登入]。
4.  若要開啟 SAML 型單一登入測試體驗，請在 [網域和 URL] 區段中，按一下 [測試 SAML 設定]。 如果 [測試 SAML 設定] 按鈕呈現灰色，則表示您需要先填妥並儲存所需的屬性。
5.  在 [測試單一登入] 刀鋒視窗中，使用公司的認證來登入目標應用程式。 您可以使用目前使用者或不同使用者的身分來登入。 如果您使用不同使用者的身分登入，會有提示要求您進行驗證。

    ![測試 SAML 頁面](./media/howto-v1-debug-saml-sso-issues/testing.png)


如果您成功登入，就表示您已通過測試。 在此情況下，Azure AD 會向應用程式發出 SAML 回應權杖。 應用程式使用 SAML 權杖成功地將您登入。

如果您在公司的登入頁面或應用程式的頁面上遇到錯誤，請使用下列其中一節來解決錯誤。


## <a name="resolve-a-sign-in-error-on-your-company-sign-in-page"></a>解決公司登入頁面上的登入錯誤

當您嘗試登入時，可能會在公司的登入頁面上看到錯誤。 

![登入錯誤](./media/howto-v1-debug-saml-sso-issues/error.png)

若要對這個錯誤進行偵錯，您需要錯誤訊息和 SAML 要求。 My Apps 安全登入擴充功能會自動收集此資訊，並在 Azure AD 上顯示解決指導方針。 

若要在已安裝 My Apps 安全登入擴充功能的情況下解決登入錯誤：

1.  在發生錯誤時，此擴充功能會將您重新導向回到 Azure AD 的 [測試單一登入] 刀鋒視窗。 
2.  在 [測試單一登入] 刀鋒視窗中，按一下 [下載 SAML 要求]。 
3.  根據錯誤和 SAML 要求中的值，您應該會看到特定的解決指導方針。 請檢閱該指導方針。

若要在未安裝 My Apps 安全登入擴充功能的情況下解決錯誤：

1. 複製頁面右下角的錯誤訊息。 錯誤訊息包括：
    - 相互關聯識別碼和時間戳記。 在向 Microsoft 建立支援案例時務必要有這些值，原因是這些值可協助工程師識別問題並提供正確的問題解決方式。
    - 可識別問題根本原因的陳述。
2.  返回 Azure AD，並尋找 [測試單一登入] 刀鋒視窗。
3.  在 [取得解決指導方針] 上方的文字方塊中，貼上錯誤訊息。
3.  按一下 [取得解決指導方針] 以顯示問題的解決步驟。 此指導方針可能需要來自 SAML 要求或 SAML 回應的資訊。 如果您未使用 My Apps 安全登入擴充功能，則可能需要 [Fiddler](http://www.telerik.com/fiddler) 之類的工具來擷取 SAML 要求和回應。
4.  確認 SAML 要求中的目的地，會對應到從 Azure Active Directory 中取得的 SAML 單一登入服務 URL
5.  確認 SAML 要求中的簽發者，就是您在 Azure Active Directory 中為應用程式設定的同一個識別碼。 Azure AD 會使用簽發者尋找您目錄中的應用程式。
6.  確認 AssertionConsumerServiceURL 是應用程式預期會從 Azure Active Directory 收到 SAML 權杖的位置。 您可以在 Azure Active Directory 中設定這個值，但如果它是 SAML 要求的一部分則不一定要設定。


## <a name="resolve-a-sign-in-error-on-the-application-page"></a>解決應用程式頁面上的登入錯誤

您可能會先成功登入，之後才在應用程式的頁面上看到錯誤。 當 Azure AD 核發權杖給應用程式，但應用程式不接受該回應時，就會發生這種情況。   

若要解決此錯誤：

1. 如果應用程式在 Azure AD 資源庫中，請確認您已遵循用於整合應用程式與 Azure AD 的所有步驟。 若要尋找應用程式的整合指示，請參閱 [SaaS 應用程式整合教學課程清單](../saas-apps/tutorial-list.md)。
2. 擷取 SAML 回應。
    - 如果您已安裝 My Apps 安全登入擴充功能，請從 [測試單一登入] 刀鋒視窗中，按一下 [下載 SAML 回應]。
    - 如果未安裝此擴充功能，則請使用 [Fiddler](http://www.telerik.com/fiddler) 之類的工具來擷取 SAML 回應。 
3. 請注意 SAML 回應權杖中的這些元素：
    - NameID 值和格式的使用者唯一識別碼
    - 在權杖中所發出的宣告
    - 用來簽署權杖的憑證。 如需如何檢閱 SAML 回應的相關資訊，請參閱[單一登入 SAML 通訊協定](single-sign-on-saml-protocol.md)。
4. 如需 SAML 回應的詳細資訊，請參閱[單一登入 SAML 通訊協定](single-sign-on-saml-protocol.md)。
5. 既然您已檢閱 SAML 回應，請參閱[登入後應用程式頁面的錯誤](../application-sign-in-problem-application-error.md)，以取得解決問題的指導方針。 
6. 如果您仍無法成功登入，則可以詢問應用程式廠商 SAML 回應中遺漏了什麼。


## <a name="next-steps"></a>後續步驟
既然應用程式已可使用單一登入，您可以[將 SaaS 應用程式的使用者佈建和解除佈建自動化](../active-directory-saas-app-provisioning.md)，也可以[開始使用條件式存取](../conditional-access/app-based-conditional-access.md)。


