---
title: Azure AD Connect 和同盟 | Microsoft Docs
description: 此頁面是使用 Azure AD Connect 之 AD FS 作業的所有相關文件的中心位置。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: f9107cf5-0131-499a-9edf-616bf3afef4d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/09/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 214dd95bb277053794656e1ba3dd148c085688ce
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48900439"
---
# <a name="azure-ad-connect-and-federation"></a>Azure AD Connect 和同盟
Azure Active Directory (Azure AD) Connect 可讓您使用內部部署 Active Directory 同盟服務 (AD FS) 與 Azure AD 來設定同盟。 使用同盟登入，您可以讓使用者使用他們的內部部署密碼登入 Azure AD 服務，並且在使用公司網路時，無須再次輸入密碼就可登入服務。 您可以藉由使用具備 AD FS 的同盟選項來部署新安裝的 AD FS，或者您可以在 Windows Server 2012 R2 伺服器陣列中指定現有的安裝。

本主題是 Azure AD Connect 同盟相關功能的主要資訊來源。 它會列出所有相關的主題連結。 如需 Azure AD Connect 的連結，請參閱 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

## <a name="azure-ad-connect-federation-topics"></a>Azure AD Connect：同盟主題
| 話題 | 涵蓋內容和閱讀時機 |
|:--- |:--- |
| **Azure AD Connect 使用者登入選項** | |
| [了解使用者登入選項](plan-connect-user-signin.md) |了解各種使用者登入選項及其如何影響 Azure 登入使用者體驗。 |
| **使用 Azure AD Connect 安裝 AD FS** | |
| [先決條件](how-to-connect-install-custom.md#ad-fs-configuration-pre-requisites) |請查看透過 Azure AD Connect 成功安裝 AD FS 安裝的必要條件。 |
| [設定 AD FS 伺服器陣列](how-to-connect-install-custom.md#configuring-federation-with-ad-fs) |使用 Azure AD Connect 安裝新的 AD FS 伺服器陣列。 |
| [使用替代登入識別碼與 Azure AD 建立同盟關係](how-to-connect-fed-management.md#alternateid) | 使用替代登入識別碼設定同盟  |
| **修改 AD FS 組態** | |
| [修復信任](how-to-connect-fed-management.md#repairthetrust) |修復內部部署 AD FS 和 Office 365/Azure 之間目前的信任。 |
| [新增 AD FS 伺服器](how-to-connect-fed-management.md#addadfsserver) |在初始安裝之後，增加 AD FS 伺服器來擴充 AD FS 伺服器陣列。 |
| [新增 AD FS WAP 伺服器](how-to-connect-fed-management.md#addwapserver) |在初始安裝之後，增加 Web 應用程式 Proxy (WAP) 伺服器來擴充 AD FS 伺服器陣列。 |
| [新增新的同盟網域](how-to-connect-fed-management.md#addfeddomain) |新增要與 Azure AD 同盟的其他網域。 |
| [更新 SSL 憑證](how-to-connect-fed-ssl-update.md)| 更新 AD FS 伺服器陣列的 SSL 憑證。 |
| [更新 Office 365 和 Azure AD 的同盟憑證](how-to-connect-fed-o365-certs.md)|使用 Azure AD 來更新 O365 憑證。|
| **其他同盟組態** | |
| [將多個 Azure AD 執行個體與單一 AD FS 執行個體同盟](how-to-connect-fed-single-adfs-multitenant-federation.md) | 將多個 Azure AD 和單一 AD FS 伺服器陣列建立同盟關係| 
| [新增自訂公司標誌/圖例](how-to-connect-fed-management.md#customlogo) |指定要顯示於 AD FS 登入頁面的自訂標誌來修改登入體驗。 |
| [新增登入說明](how-to-connect-fed-management.md#addsignindescription) |變更 AD FS 登入頁面上的登入說明。 |
| [修改 AD FS 宣告規則](how-to-connect-fed-management.md#modclaims) |修改或新增對應至 Azure AD Connect 同步組態之 AD FS 中的宣告規則。 |


## <a name="additional-resources"></a>其他資源
* [將兩個 Azure AD 和單一 AD FS 建立同盟關係](how-to-connect-fed-single-adfs-multitenant-federation.md)
* [Azure 中的 AD FS 部署](how-to-connect-fed-azure-adfs.md)
* [使用 Azure 流量管理員在 Azure 中部署高可用性跨地區 AD FS](../active-directory-adfs-in-azure-with-azure-traffic-manager.md)
