---
title: 宣告感知應用程式 - Azure AD 應用程式 Proxy | Microsoft Docs
description: 如何發佈接受 ADFS 宣告的內部部署 ASP.NET 應用程式，讓您的使用者安全地進行遠端存取。
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
ms.date: 08/04/2017
ms.author: barbkess
ms.reviewer: harshja
ms.openlocfilehash: cf92b5b6ee3c6a529a43e7fa4cfeeb09954ad9ea
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39365386"
---
# <a name="working-with-claims-aware-apps-in-application-proxy"></a>在應用程式 Proxy 中使用宣告感知應用程式
[宣告感知應用程式](https://msdn.microsoft.com/library/windows/desktop/bb736227.aspx)會執行重新導向至 Security Token Service (STS)。 STS 會向使用者要求認證以交換權杖，然後將使用者重新導向至應用程式。 有幾種方法可以讓應用程式 Proxy 進行這些重新導向。 請按照本文的說明來設定對宣告感知應用程的部署。 

## <a name="prerequisites"></a>必要條件
確定宣告感知應用程式重新導向的 STS 可從內部部署網路外部使用。 您可以透過 Proxy 將 STS 公開或是允許外部連接，讓 STS 可供使用。 

## <a name="publish-your-application"></a>發佈您的應用程式

1. 根據 [使用應用程式 Proxy 發佈應用程式](application-proxy-publish-azure-portal.md)中的所述指示來發佈您的應用程式。
2. 瀏覽至入口網站中的應用程式頁面，然後選取 [單一登入]。
3. 如果您選擇 [Azure Active Directory] 作為您的 [預先驗證方法]，請選取 [Azure AD 單一登入已停用] 作為您的 [內部驗證方法]。 如果您選擇 [傳遞] 作為您的 [預先驗證方法]，則無需進行任何變更。

## <a name="configure-adfs"></a>設定 ADFS

您可以使用以下兩種方式之一為宣告感知應用程式設定 ADFS。 第一種方式是使用自訂網域。 第二種方式是使用 WS-同盟。 

### <a name="option-1-custom-domains"></a>選項 1：自訂網域

如果您應用程式的所有內部 URL 皆為完整網域名稱 (FQDN)，則您可為您的應用程式設定[自訂網域](application-proxy-configure-custom-domain.md)。 請使用自訂網域建立與內部 URL 相同的外部 URL。 當您的外部 URL 與內部 URL 相符時，無論使用者是在內部部署或遠端，STS 重新導向都會運作。 

### <a name="option-2-ws-federation"></a>選項 2：WS-同盟

1. 開啟 [ADFS 管理]。
2. 移至 [信賴憑證者信任]，並在您要使用「應用程式 Proxy 」來發佈的應用程式上按一下滑鼠右鍵，然後選擇 [屬性]。  

   ![信賴憑證者信任 - 以滑鼠右鍵按一下應用程式名稱 - 螢幕擷取畫面](./media/application-proxy-configure-for-claims-aware-applications/appproxyrelyingpartytrust.png)  

3. 在 [端點] 索引標籤的 [端點類型] 底下，選取 [WS-同盟]。
4. 在 [信任的 URL] 底下，輸入您在「應用程式 Proxy」的 [外部 URL] 底下輸入的 URL，然後按一下 [確定]。  

   ![新增端點 - 設定 [信任的 URL] 值 - 螢幕擷取畫面](./media/application-proxy-configure-for-claims-aware-applications/appproxyendpointtrustedurl.png)  

## <a name="next-steps"></a>後續步驟
* 對於非宣告感知的應用程式，請[啟用單一登入](application-proxy-single-sign-on.md)
* [啟用原生用戶端應用程式以與 Proxy 應用程式互動](application-proxy-configure-native-client-application.md)


