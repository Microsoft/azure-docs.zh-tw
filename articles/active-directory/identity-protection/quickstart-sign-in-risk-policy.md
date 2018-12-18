---
title: 快速入門 - 針對 Azure Active Directory Identity Protection 偵測到工作階段風險時封鎖存取 | Microsoft Docs
description: 在此快速入門中，您會了解如何設定 Azure Active Directory (Azure AD) Identity Protection 登入風險條件式存取原則，根據工作階段風險封鎖登入。
services: active-directory
keywords: identity protection, conditional access to apps, conditional access with Azure AD, secure access to company resources, conditional access policies, Identity Protection, 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: identity-protection
ms.topic: article
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/13/2018
ms.author: markvi
ms.reviewer: raluthra
ms.openlocfilehash: 6879822e8451e6170bf17fff4ab66b672f711a93
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45632382"
---
# <a name="quickstart-block-access-when-a-session-risk-is-detected-with-azure-active-directory-identity-protection"></a>快速入門：針對 Azure Active Directory Identity Protection 偵測到工作階段風險時封鎖存取  

為了讓您的環境受到保護，您可以封鎖可疑使用者，不讓他們登入。 Azure Active Directory (Azure AD) Identity Protection 會分析每一次登入，並且計算非由使用者帳戶合法擁有者嘗試執行登入的可能性。 可能性 (低、中、高) 是以稱為登入風險等級的計算值形式表示。 透過設定登入風險條件，您可以設定登入風險條件式存取原則來回應特定的登入風險等級。 

此快速入門示範如何設定登入風險條件式存取原則，該原則會在偵測到中 (含) 以上的登入風險等級時封鎖登入。 

![建立原則](./media/quickstart-sign-in-risk-policy/1004.png)


如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。



## <a name="prerequisites"></a>先決條件 

若要完成此教學課程中的案例，您需要：

- **存取 Azure AD Premium P2 版** - Azure AD Identity Protection 是 Azure AD Premium P2 的功能。 

- **Identity Protection** - 此快速入門中的案例需要啟用 Identity Protection。 如果您不知道如何啟用 Identity Protection，請參閱[啟用 Azure Active Directory Identity Protection](../identity-protection/enable.md)。

- **Tor 瀏覽器** - [Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en)的設計目的是協助您保留您的線上隱私權。 Identity Protection 偵測到來自 Tor 瀏覽器的登入是**從匿名 IP 位址登入**，這種登入有中等風險等級。 如需詳細資訊，請參閱 [Azure Active Directory 風險事件](../reports-monitoring/concept-risk-events.md)。  

- **稱為 Alain Charon 的測試帳戶** - 如果您不知道如何建立測試帳戶，請參閱[新增新使用者](../fundamentals/add-users-azure-active-directory.md#add-a-new-user)。


## <a name="test-your-sign-in"></a>測試您的登入 

此步驟的目標是確保您的測試帳戶可以使用 Tor 瀏覽器存取租用戶。

**若要測試您的登入：**

1. 請以 **Alain Charon** 身分登入 [Azure 入口網站](https://portal.azure.com)。

2. 登出。 


## <a name="create-your-conditional-access-policy"></a>建立條件式存取原則 

此快速入門中的案例會使用來自 Tor 瀏覽器的登入，以產生偵測到**從匿名 IP 位址登入**的風險事件。 此風險事件的風險等級是「中」。 若要回應此風險事件，您可以將登入風險條件設定為「中」。 

此節說明如何建立必要的登入風險條件式存取原則。 為您的原則進行下列設定：

|設定 |值|
|---     | --- |
| 使用者  | Alain Charon  |
| 條件 | 登入風險，中 (含) 以上 |
| 控制 | 封鎖存取 |
 

![建立原則](./media/quickstart-sign-in-risk-policy/201.png)

 


**設定條件式存取原則：**

1. 以全域管理員身分登入 [Azure 入口網站](https://portal.azure.com)。

2. 移至 [Azure AD Identity Protection 頁面](https://portal.azure.com/#blade/Microsoft_AAD_ProtectionCenter/IdentitySecurityDashboardMenuBlade/Overview)。
 
3. 在 [Azure AD Identity Protection] 頁面的 [設定] 區段中，按一下 [登入風險原則]。
 
4. 在 [原則] 頁面上，按一下 [指派] 區段中的 [使用者]。

5. 在 [使用者] 頁面上，按一下 [選取使用者]。

6. 在 [選取使用者] 頁面上，選取 **Alain Charon**，然後按一下 [選取]。

7. 在 [使用者] 頁面上，按一下 [完成]。 

8. 在 [原則] 頁面上，按一下 [指派] 區段中的 [條件]。

9. 在 [條件] 頁面上，按一下 [登入風險]。

10. 在 [登入風險] 頁面上，選取 [中 (含) 以上]，然後按一下 [選取]。 

11. 在 [條件] 頁面上，按一下 [完成]。

12. 在 [原則] 頁面上，按一下 [控制] 區段中的 [存取]。

13. 在 [存取] 頁面上，按一下 [允許存取]，選取 [需要多重要素驗證]，然後按一下 [選取]。

14. 在 [原則] 頁面上，按一下 [儲存]。  


## <a name="test-your-conditional-access-policy"></a>測試條件式存取原則

若要測試您的原則，請嘗試使用 Tor 瀏覽器以 **Alan Charon** 身分登入 [Azure 入口網站](https://portal.azure.com)。 您的登入嘗試應該會遭到條件式存取原則封鎖。

![Multi-Factor Authentication](./media/quickstart-sign-in-risk-policy/203.png)


## <a name="clean-up-resources"></a>清除資源

當您不再需要時，即可刪除測試使用者、Tor 瀏覽器，並停用登入風險條件式存取原則：

- 如果您不知道如何刪除 Azure AD 使用者，請參閱[如何新增或刪除使用者](../fundamentals/add-users-azure-active-directory.md#delete-a-user)。

- 如需移除 Tor 瀏覽器的指示，請參閱[解除安裝](https://tb-manual.torproject.org/en-US/uninstalling.html)。


