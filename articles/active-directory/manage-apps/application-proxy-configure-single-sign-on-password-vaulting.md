---
title: 使用 Azure AD 應用程式 Proxy 設定應用程式的單一登入 | Microsoft Docs
description: 在 Azure 入口網站中使用 Azure AD 應用程式 Proxy，為您已發佈的內部部署應用程式啟動單一登入。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/20/2017
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: fa12dd5e9dbe25bad947abed5ab1c732d231b25c
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39362817"
---
# <a name="password-vaulting-for-single-sign-on-with-application-proxy"></a>使用應用程式 Proxy 進行單一登入的密碼保存庫

Azure Active Directory 應用程式 Proxy 可發佈內部部署應用程式，讓遠端員工也可以安全地存取它們，進而幫助您改進生產力。 在 Azure 入口網站中，您也可以設定這些應用程式的單一登入 (SSO)。 您的使用者只需要向 Azure AD 驗證，便可以存取您的企業應用程式，而不必再次登入。

應用程式 Proxy 支援數個[單一登入模式](application-proxy-single-sign-on.md)。 密碼型登入適用於使用使用者名稱/密碼組合進行驗證的應用程式。 當您設定應用程式的密碼型登入您時，您的使用者必須登入一次內部部署應用程式。 之後，Azure Active Directory 會儲存登入資訊，並且會在您的使用者從遠端存取時，自動將登入資訊提供給應用程式。 

您應該已經使用應用程式 Proxy 發行並測試您的應用程式。 如果還沒，請依照[使用 Azure AD 應用程式 Proxy 發佈應用程式](application-proxy-publish-azure-portal.md)的步驟操作，然後回到這裡。 

## <a name="set-up-password-vaulting-for-your-application"></a>為應用程式設定密碼儲存庫存

1. 以系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [Azure Active Directory]  >  [企業應用程式]  >  [所有應用程式]。
3. 從清單中選取您要設定 SSO 的應用程式。  
4. 選取 [單一登入]。

   ![選取單一登入](./media/application-proxy-configure-single-sign-on-password-vaulting/select-sso.png)

5. 在 SSO 模式中，選擇 [密碼型登入]。
6. 在登入 URL 輸入頁面 URL，使用者將在該頁面輸入其使用者名稱和密碼，以登入公司網路外部的應用程式。 這可能是您透過應用程式 Proxy 發佈應用程式時建立的外部 URL。 

   ![選擇密碼型登入並輸入您的 URL](./media/application-proxy-configure-single-sign-on-password-vaulting/password-sso.png)

7. 選取 [ **儲存**]。

<!-- Need to repro?
7. The page should tell you that a sign-in form was successfully detected at the provided URL. If it doesn't, select **Configure [your app name] Password Single Sign-on Settings** and choose **Manually detect sign-in fields**. Follow the instructions to point out where the sign-in credentials go. 
-->

## <a name="test-your-app"></a>測試應用程式

移至您設定用於遠端存取應用程式的外部 URL。 使用該應用程式的認證登入 (或您設定有存取權之測試帳戶的認證)。 一旦您成功登入，應該能夠離開應用程式後再回到應用程式，不需要再次輸入您的認證。 

## <a name="next-steps"></a>後續步驟

- 閱讀其他實作[使用應用程式 Proxy 進行單一登入](application-proxy-single-sign-on.md)的做法
- 深入了解[使用 Azure AD 應用程式 Proxy 遠端存取應用程式的安全性考量](application-proxy-security.md)