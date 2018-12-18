---
title: 兩步驟驗證和 AD FS - Azure MFA | Microsoft Docs
description: 這是說明如何開始使用 Azure MFA 和 AD FS 的 Azure Multi-Factor Authentication 頁面。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 6ce31bba9fb2f237586c6e59cbb5be7643c35aa1
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39161428"
---
# <a name="getting-started-with-azure-multi-factor-authentication-and-active-directory-federation-services"></a>開始使用 Azure Multi-Factor Authentication Server 和 Active Directory Federation Services

<center>![雲端](./media/multi-factor-authentication-get-started-adfs/adfs.png)</center>

如果組織已使用 AD FS 讓內部部署 Active Directory 與 Azure Active Directory 同盟，就會有兩個使用 Azure Multi-Factor Authentication 的選項。

* 使用 Azure Multi-Factor Authentication 或 Active Directory Federation Services 保護雲端資源
* 使用 Azure Multi-Factor Authentication Server 保護雲端和內部部署資源

下表摘要說明保護使用 Azure Multi-Factor Authentication 和 AD FS 兩者保護資源的驗證體驗

| 驗證體驗 - 瀏覽器型應用程式 | 驗證體驗 - 非瀏覽器型應用程式 |
|:--- |:--- |:--- |
| 使用 Azure Multi-Factor Authentication 保護 Azure AD 資源 |<li>第一個驗證步驟是使用 AD FS 在內部部署執行。</li> <li>第二個步驟是使用雲端驗證執行的電話式方法。</li> |
| 使用 Active Directory Federation Services 保護 Azure AD 資源 |<li>第一個驗證步驟是使用 AD FS 在內部部署執行。</li><li>第二個步驟是使用宣告以在內部部署執行。</li> |

有關同盟使用者之應用程式密碼的警告：

* 應用程式密碼是使用雲端驗證進行驗證，因此會略過同盟。 唯有在設定應用程式密碼時才能主動使用同盟。
* 應用程式密碼不會遵守內部部署用戶端存取控制設定。
* 您會失去應用程式密碼的內部部署驗證記錄功能。
* 帳戶停用/刪除最長可能需要三個小時才能完成目錄同步處理，導致雲端身分識別中的應用程式密碼停用/刪除延遲。

如需使用 AD FS 設定 Azure Multi-Factor Authentication 或 Azure Multi-Factor Authentication Server 的相關資訊，請參閱下列各文章：

* [使用 Azure Multi-Factor Authentication 和 AD FS 保護雲端資源](howto-mfa-adfs.md)
* [搭配 Windows Server 2012 R2 AD FS 使用 Azure Multi-Factor Authentication Server 保護雲端和內部部署資源](howto-mfaserver-adfs-2012.md)
* [搭配 AD FS 2.0 使用 Azure Multi-Factor Authentication Server 保護雲端和內部部署資源](howto-mfaserver-adfs-2.md)
