---
title: 註冊應用程式以使用 Azure Active Directory | Microsoft Docs
description: 針對 IT 專業人員所撰寫，本文提供整合 Azure 應用程式與 Active Directory 的指導方針。
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: 006aaf7ca5066c552f9c0b797549d7e90ac9beb7
ms.sourcegitcommit: f5b8410738bee1381407786fcb9d3d3ab838d813
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98208687"
---
# <a name="develop-line-of-business-apps-for-azure-active-directory"></a>開發適用於 Azure Active Directory 的企業營運應用程式
本指南概述如何針對 Azure Active Directory (AD) ，開發企業營運 (LoB) 應用程式。目標物件是 Active Directory/Microsoft 365 全域系統管理員。

## <a name="overview"></a>概觀
建立與 Azure AD 整合的應用程式，可讓您組織中的使用者使用 Microsoft 365 進行單一登入。 將應用程式置於 Azure AD 中可讓您掌控應用程式設定的驗證原則。 若要深入瞭解條件式存取，以及如何使用多重要素驗證保護應用程式 (MFA) 請參閱設定 [存取規則](../authentication/tutorial-enable-azure-mfa.md)。

註冊應用程式以使用 Azure Active Directory。 註冊應用程式意謂著開發人員可以使用 Azure AD 來驗證使用者，以及要求對使用者資源 (例如電子郵件、行事曆及文件) 的存取權。

您目錄的任何成員 (不是來賓) 都可以註冊應用程式，亦稱為 *建立應用程式物件*。 如果您無法註冊應用程式，則表示目錄的全域管理員已限制這項功能，因此您可能需要聯繫它們，才能 [取得適當的許可權](../roles/delegate-app-roles.md#assign-built-in-application-admin-roles) 來註冊應用程式。 若要深入瞭解如何限制使用者的詳細資訊，請參閱 [Azure Active Directory 中的委派應用程式註冊許可權](../roles/delegate-app-roles.md#restrict-who-can-create-applications)。

註冊應用程式可讓任一使用者執行下列動作：

* 取得 Azure AD 識別的應用程式的身分識別
* 取得應用程式可用來向 AD 驗證其身分的一個或多個密碼/金鑰
* 在 Azure 入口網站中以自訂名稱、標誌等指定應用程式的品牌形象
* 將 Azure AD 授權功能套用至其應用程式，包括：

  * 角色型存取控制 (RBAC)
  * 以 Azure Active Directory 做為 oAuth 授權伺服器 (保護應用程式公開的 API)
* 宣告讓應用程式如預期般運作所需的必要權限，包括：

     - 應用程式權限 (僅限全域系統管理員)。 例如：另一個 Azure AD 應用程式中的角色成員資格，或相對於「Azure 資源」、「資源群組」或「訂用帳戶」的角色成員資格
     - 委派的權限 (任何使用者)。 例如：Azure AD、登入及讀取設定檔

> [!NOTE]
> 根據預設，任何成員都可以註冊應用程式。 若要了解如何限制向特定成員註冊應用程式的權限，請參閱 [如何將應用程式新增到 Azure AD](../develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance)。
>
>

身為全域系統管理員，若要協助開發人員完成讓應用程式進入生產階段的準備工作，您必須執行下列動作：

* 設定存取規則 (存取原則/MFA)
* 設定應用程式要求指派使用者並指派使用者
* 隱藏預設的使用者同意體驗

## <a name="configure-access-rules"></a>設定存取規則
針對您的 SaaS 應用程式設定每個應用程式的存取規則 例如，您可以要求執行 MFA 或只允許受信任網路上的使用者進行存取。 如需這方面的詳細資料，請參閱[設定存取規則](../authentication/tutorial-enable-azure-mfa.md)文件。

## <a name="configure-the-app-to-require-user-assignment-and-assign-users"></a>設定應用程式要求指派使用者並指派使用者
根據預設，使用者不須獲得指派，即可存取應用程式。 但是，如果應用程式公開角色，或您想要讓應用程式出現在使用者的我的應用程式上，您應該需要使用者指派。

如果您是 Azure AD Premium 或 Enterprise Mobility Suite (EMS) 的訂閱者，則強烈建議使用群組。 將群組指派給應用程式，可讓您將持續進行的存取管理委派給群組擁有者。 您可以建立群組，或使用群組管理功能要求您組織中負責的對象建立群組。

[將使用者和群組指派給應用程式](./assign-user-or-group-access-portal.md)  


## <a name="suppress-user-consent"></a>隱藏使用者同意
根據預設，每個使用者都必須經歷同意體驗才能登入。 同意體驗 (要求使用者將權限授與應用程式) 會使得不熟悉做這類決定的使用者不知所措。

對於您信任的應用程式，您可以代表您的組織來同意應用程式，以簡化使用者體驗。

如需 Azure 中的使用者同意和同意體驗的詳細資訊，請參閱 [瞭解 Azure AD 應用程式同意體驗](../develop/application-consent-experience.md)。

## <a name="related-articles"></a>相關文章
* [使用 Azure AD 應用程式 Proxy 啟用對內部部署應用程式的安全遠端存取](application-proxy.md)
* [使用 Azure AD 管理應用程式的存取](what-is-access-management.md)
