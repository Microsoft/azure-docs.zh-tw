---
title: 管理 Azure AD 應用程式 Proxy 的 SSO | Microsoft Docs
description: 深入了解使用應用程式 Proxy 進行單一登入的基本概念
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
ms.date: 05/21/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: dbdbe8b83af20f66ad2cc99d2a5665262479b4a9
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39364143"
---
# <a name="how-does-azure-ad-application-proxy-provide-single-sign-on"></a>Azure AD 應用程式 Proxy 如何提供單一登入？

單一登入是 Azure AD 應用程式 Proxy 的重要元素。  因為您的使用者只需要登入雲端中的 Azure Active Directory，所以它可提供最佳的使用者體驗。 使用者向 Azure Active Directory 驗證後，應用程式 Proxy 連接器就會處理向內部部署應用程式的驗證。 後端應用程式無法分辨透過應用程式 Proxy 的遠端使用者登入與已加入網域的裝置上一般用途之間的差異。 

若要使用 Azure Active Directory 單一登入您的應用程式，您需要選取 [Azure Active Directory] 作為預先驗證的方法。 如果您選取 [傳遞]，則使用者完全不會向 Azure Active Directory 驗證，但系統會將他們直接導向至應用程式。 您可以在第一次發佈應用程式時設定這項設定，或在 Azure 入口網站中巡覽至您的應用程式並編輯應用程式 Proxy 設定。 

若要查看您的單一登入選項，請遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 巡覽至 [Azure Active Directory]  >  [企業應用程式]  >  [所有應用程式]。
3. 選取您要管理其單一登入選項的應用程式。
4. 選取 [單一登入]。

   ![SSO 下拉式功能表](./media/application-proxy-single-sign-on/single-sign-on-mode.png)

下拉式功能表會顯示單一登入您的應用程式的五個選項：

* 已啟用 Azure AD 單一登入
* 密碼型登入
* 連結型登入
* 整合式 Windows 驗證
* 標頭型登入

## <a name="azure-ad-single-sign-on-disabled"></a>已啟用 Azure AD 單一登入

如果您不想使用 Azure Active Directory 整合來單一登入您的應用程式，請選擇 [已停用 Azure AD 單一登入]。 選取此選項後，使用者可能會進行兩次驗證。 首先，他們會向 Azure Active Directory 驗證，然後登入應用程式本身。 

如果內部部署應用程式不要求使用者進行驗證，但您想要新增 Azure Active Directory 作為遠端存取的安全性層，此選項是個不錯的選擇。 

## <a name="password-based-sign-on"></a>密碼型登入

如果您想要使用 Azure Active Directory 作為內部部署應用程式的密碼保存庫，則選擇 [密碼型登入]。 如果您的應用程式使用使用者名稱/密碼組合 (而不是存取權杖或標頭) 進行驗證，此選項是個不錯的選擇。 使用密碼型登入，使用者必須在第一次存取應用程式時進行登入。 此後，Azure Active Directory 會代表使用者提供使用者名稱和密碼。 

如需設定密碼型登入的詳細資訊，請參閱[使用應用程式 Proxy 單一登入的密碼保存庫](application-proxy-configure-single-sign-on-password-vaulting.md)。

## <a name="linked-sign-on"></a>連結型登入

如果您已經有為您的內部部署身分識別設定的單一登入解決方案，請選擇 [連結型登入]。 此選項可讓 Azure Active Directory 運用現有的 SSO 解決方案，但仍可讓使用者遠端存取應用程式。 

如需連結登入 (先前稱為現有單一登入) 的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](what-is-single-sign-on.md#how-does-single-sign-on-with-azure-active-directory-work)。

## <a name="integrated-windows-authentication"></a>整合式 Windows 驗證

如果內部部署應用程式使用整合式 Windows 驗證 (IWA)，或者如果您想要使用 Kerberos 限制委派 (KCD) 進行單一登入，請選擇 [整合式 Windows 驗證]。 使用此選項，使用者只需要向 Azure Active Directory 進行驗證，然後應用程式 Proxy 連接器會模擬使用者取得 Kerberos 權杖並登入應用程式。 

如需設定整合式 Windows 驗證的詳細資訊，請參閱 [使用應用程式 Proxy 單一登入的 Kerberos 限制委派](application-proxy-configure-single-sign-on-with-kcd.md)。

## <a name="header-based-sign-on"></a>標頭型登入 

如果應用程式使用標頭進行驗證，請選擇 [標頭型登入]。 使用此選項，使用者只需要向 Azure Active Directory 驗證。 Microsoft 與名為 PingAccess 的第三方驗證服務攜手合作，該服務已將 Azure Active Directory 存取權杖轉譯成應用程式的標頭格式。 

如需設定標頭型驗證的相關資訊，請參閱[使用應用程式 Proxy 單一登入的標頭型驗證](application-proxy-configure-single-sign-on-with-ping-access.md)。

## <a name="next-steps"></a>後續步驟

- [使用應用程式 Proxy 進行單一登入的密碼保存庫](application-proxy-configure-single-sign-on-password-vaulting.md)
- [可供使用應用程式 Proxy 單一登入的 Kerberos 限制委派](application-proxy-configure-single-sign-on-with-kcd.md)
- [使用應用程式 Proxy 單一登入的標頭型驗證](application-proxy-configure-single-sign-on-with-ping-access.md) 
