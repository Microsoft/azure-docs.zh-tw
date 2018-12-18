---
title: 使用 Azure AD 應用程式 Proxy 發佈應用程式 | Microsoft Docs
description: 在 Azure 入口網站中使用 Azure AD 應用程式 Proxy 將內部部署應用程式發佈至雲端。
services: active-directory
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: barbkess
ms.reviewer: japere
ms.custom: it-pro
ms.openlocfilehash: 973a6201a227e6c2e295e6e5ea2f40c302572504
ms.sourcegitcommit: 76797c962fa04d8af9a7b9153eaa042cf74b2699
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42141725"
---
# <a name="publish-applications-using-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 發佈應用程式

Azure Active Directory (AD) 應用程式 Proxy 可藉由發佈要透過網際網路存取的內部部署應用程式，協助您支援遠端背景工作角色。 您可以透過 Azure 入口網站來發佈這些應用程式，提供從您網路外部的安全遠端存取。

本文會逐步引導您完成使用應用程式 Proxy 發佈內部部署應用程式。 完成本文之後，您的使用者將能夠從遠端存取您的應用程式。 且您可以設定應用程式的單一登入、個人化資訊或安全性需求等額外功能。

如果您是應用程式 Proxy 的新手，可至[如何為內部部署應用程式提供安全的遠端存取](application-proxy.md)深入了解這個功能。

## <a name="before-you-begin"></a>開始之前

本文假設您已經安裝並註冊連接器。 如果您仍然需要執行這些步驟，請參閱[開始使用應用程式 Proxy 並安裝連接器](application-proxy-enable.md)。

## <a name="publish-an-on-premises-app-for-remote-access"></a>發佈可遠端存取的內部部署應用程式

遵循這些步驟可發佈您的應用程式與應用程式 Proxy。 如果您尚未下載並為您的組織設定連接器，請先移至[開始使用應用程式 Proxy 並安裝連接器](application-proxy-enable.md)，然後發佈您的應用程式。

> [!TIP]
> 如果您是第一次測試應用程式 Proxy，請選擇設定為密碼型驗證的應用程式。 應用程式 Proxy 支援其他類型的驗證，但密碼型應用程式是開始使用並快速執行最簡單的方式。 

1. 在 [Azure 入口網站](https://portal.azure.com/)中，以系統管理員身分登入。
2. 選取 [Azure Active Directory]  >  [企業應用程式]  >  [新增應用程式]。

  ![新增企業應用程式](./media/application-proxy-publish-azure-portal/add-app.png)

3. 依序選取 [所有]、[內部部署應用程式]。  

  ![新增您自己的應用程式](./media/application-proxy-publish-azure-portal/add-your-own.png)

4. 提供您的應用程式的下列資訊：

   - **名稱**︰會出現在存取面板上和 Azure 入口網站的應用程式名稱。 

   - **內部 URL**：要從您私人網路中存取應用程式的 URL。 您可以提供後端伺服器上要發佈的特定路徑，而伺服器的其餘部分則不發佈。 如此一來，您可以在相同的伺服器上將不同網站發佈為不同應用程式，並給予各自的名稱和存取規則。

     > [!TIP]
     > 如果您發佈路徑，請確定其中包含您的應用程式的所有必要映像、指令碼和樣式表。 例如，如果您的應用程式位於 https://yourapp/app 並使用位於 https://yourapp/media 的映像，您應該發佈 https://yourapp/ 作為路徑。 此內部 URL 不一定是您使用者所看見的登陸頁面。 如需詳細資訊，請參閱[針對發佈應用程式設定自訂的首頁](application-proxy-configure-custom-home-page.md)。

   - **外部 URL**：您的使用者要前往此位址才能在您的網路之外存取您的應用程式。 如果您不想使用預設的應用程式 Proxy 網域，請閱讀 [Azure AD Application Proxy 中的自訂網域](application-proxy-configure-custom-domain.md)。
   - **預先驗證**︰應用程式 Proxy 在給予您的應用程式存取權前，用來驗證使用者的方式。 

     - Azure Active Directory︰應用程式 Proxy 會重新導向使用者以使用 Azure AD 登入，進而驗證目錄和應用程式的權限。 建議您將這個選項保持為預設值，讓您可以利用諸如條件式存取以及 Multi-Factor Authentication 等 Azure AD 安全性功能。
     - 即時通行︰使用者不必向 Azure Active Directory 進行驗證即可存取應用程式。 您還是可以在後端設定驗證需求。
   - **連接器群組**︰連接器會處理針對應用程式的遠端存取，連接器群組可協助您依區域、網路或用途組織連接器和應用程式。 如果您尚未建立任何連接器群組，您的應用程式就會指派給 [預設]。

>[!NOTE]
>如果您的應用程式使用 WebSocket 進行連線，請確認您的連接器版本至少為 1.5.612.0 並支援 WebSocket，而且指派的連接器群組只會使用這些連接器。

   ![設定您的應用程式](./media/application-proxy-publish-azure-portal/configure-app.png)
5. 如有必要，請設定其他設定。 對於大部分的應用程式，您應該在其預設狀態中保留這些設定。 
   - **後端應用程式逾時**：只有當您的應用程式太慢而無法驗證和連線時，才將此值設定為 [長]。 
   - **使用僅限 HTTP Cookie**：將此值設定為 [是]，讓應用程式 Proxy Cookie 在 HTTP 回應標頭中包含 HTTPOnly 旗標。
   - **轉譯標頭中的 URL**：除非您的應用程式需要驗證要求中的原始主機標頭，否則請將此值保留為 [是]。
   - **轉譯應用程式主體中的 URL**：除非您有其他內部部署應用程式的硬式編碼 HTML 連結，且未使用自訂網域，否則請將此值保留為 [否]。 如需詳細資訊，請參閱[使用應用程式 Proxy 連結轉譯](application-proxy-configure-hard-coded-link-translation.md)。
   
   ![設定您的應用程式](./media/application-proxy-publish-azure-portal/additional-settings.png)

6. 選取 [新增] 。


## <a name="add-a-test-user"></a>新增測試使用者 

若要測試您的應用程式是否已正確發佈，可新增一個測試使用者帳戶。 請確認此帳戶已有權限，可存取來自公司網路內部的應用程式。

1. 在 [快速啟動] 刀鋒視窗中，選取 [指派測試使用者]。

  ![指派測試使用者](./media/application-proxy-publish-azure-portal/assign-user.png)

2. 在 [使用者和群組] 刀鋒視窗上，選取 [新增]。

  ![新增使用者或群組](./media/application-proxy-publish-azure-portal/add-user.png)

3. 在 [新增] 指派刀鋒視窗上，選取 [使用者和群組]，然後選擇您要新增的帳戶。 
4. 選取 [指派]。

## <a name="test-your-published-app"></a>測試已發佈的應用程式

在瀏覽器中瀏覽至您在發行步驟所設定的外部 URL。 您應該會看到開始畫面，且能夠使用您設定的測試帳戶登入。

![測試已發佈的應用程式](./media/application-proxy-publish-azure-portal/test-app.png)


## <a name="next-steps"></a>後續步驟
- [下載連接器](application-proxy-enable.md)和[建立連接器群組](application-proxy-connector-groups.md)，在不同的網路和位置上發佈應用程式。

- 為您新發佈的應用程式[設定單一登入](application-proxy-configure-single-sign-on-password-vaulting.md)
