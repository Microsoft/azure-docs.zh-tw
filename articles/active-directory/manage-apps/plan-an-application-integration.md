---
title: 開始整合 Azure AD 與應用程式
description: 本文章是整合 Azure Active Directory (AD) 與在內部部署應用程式和雲端應用程式的入門指南。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/16/2018
ms.author: kenwith
ms.reviewer: asteen
ms.openlocfilehash: db3d3623e175d582a2fe271d73aa452ca07b8e8d
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98735061"
---
# <a name="integrating-azure-active-directory-with-applications-getting-started-guide"></a>整合 Azure Active Directory 與應用程式入門指南

本主題摘錄了整合應用程式與 Azure Active Directory (AD) 的程序。 每個以下各節包含更詳細主題的簡短摘要，讓您可以識別本入門指南的哪些部分與您相關。

若要下載深入部署計劃，請參閱[後續步驟](#next-steps)。

## <a name="take-inventory"></a>取得清查
在整合應用程式與 Azure AD 之前，請務必知道您的所在位置與您要前往的位置。  下列問題的用意在於幫助您思考您的 Azure AD 應用程式整合專案。

### <a name="application-inventory"></a>應用程式清查
* 您所有應用程式的所在位置？ 擁有者為？
* 您的應用程式需要何種驗證？
* 誰需要存取哪些應用程式？
* 您是否要部署新應用程式？
  * 您將在公司內部建立並將它部署在 Azure 計算執行個體？
  * 您將使用 Azure 應用程式資源庫中所提供的其中一個應用程式？

### <a name="user-and-group-inventory"></a>使用者和群組清查
* 您的使用者帳戶所在的位置？
  * 內部部署 Active Directory
  * Azure AD
  * 您擁有在個別的應用程式資料庫中
  * 在未經約束的應用程式中
  * 以上皆是
* 個別使用者目前有哪些權限和角色指派？ 您需要檢閱其存取權或您確定使用者現在的存取和角色指派適當嗎？
* 是否已經在您的內部部署 Active Directory 中建立群組？
  * 您的群組的組織方式？
  * 有哪些群組成員？
  * 群組目前有哪些權限/角色指派？
* 您是否需要在整合之前清除使用者/群組資料庫？  (這是很重要的問題。 垃圾進，垃圾出 - 應當避免無用資料。)

### <a name="access-management-inventory"></a>存取管理清查
* 您如何目前管理使用者對應用程式存的取？ 需要變更嗎？  您是否已考慮其他管理存取權的方式（例如使用 [AZURE RBAC](../../role-based-access-control/role-assignments-portal.md) ）？
* 誰需要存取哪些內容？

也許您事先對這所有問題沒有答案，但是沒關係。  本指南可協助您回答其中一些問題，並做出一些明智的決策。

### <a name="find-unsanctioned-cloud-applications-with-cloud-discovery"></a>使用 Cloud Discovery 尋找待批准的雲端應用程式

如上所述，可能有應用程式到目前為止仍未受到組織的管理。  作為清查程序的一部分，您可以找到未經約束的雲端應用程式。 請參閱[設定 Cloud Discovery](/cloud-app-security/set-up-cloud-discovery)。

## <a name="integrating-applications-with-azure-ad"></a>整合應用程式與 Azure AD
以下文章將討論整合應用程式與 Azure AD 的各種不同方式，並提供一些指引。

* [決定要使用的 Active Directory](../fundamentals/active-directory-whatis.md)
* [使用 Azure 應用程式資源庫中的應用程式](what-is-single-sign-on.md)
* [整合 SaaS 應用程式教學課程清單](../saas-apps/tutorial-list.md)

### <a name="authentication-types"></a>驗證類型
每個應用程式可能有不同的驗證需求。 利用 Azure AD，可將簽署憑證用於使用 SAML 2.0、WS-同盟或 OpenID Connect 通訊協定，以及密碼單一登入的應用程式。 如需要與 Azure AD 搭配使用的應用程式驗證類型的詳細資訊，請參閱[在 Azure Active Directory 中管理同盟單一登入的憑證](manage-certificates-for-federated-single-sign-on.md)和[密碼式單一登入](what-is-single-sign-on.md)。

### <a name="enabling-sso-with-azure-ad-app-proxy"></a>使用 Azure AD 應用程式 Proxy 啟用 SSO
透過 Microsoft Azure AD 應用程式 Proxy，您可以從任何地方及任何裝置上安全地為位於您的私人網路上的應用程式提供存取。 在您的環境中安裝應用程式 Proxy 連接器之後，可以輕鬆地使用 Azure AD 來加以設定。

### <a name="integrating-custom-applications"></a>整合自訂應用程式
如果您想要將自訂應用程式新增至 Azure 應用程式資源庫，請參閱將 [您的應用程式發佈至 Azure AD 應用](../develop/v2-howto-app-gallery-listing.md)程式資源庫。

## <a name="managing-access-to-applications"></a>管理應用程式的存取
下列文章描述使用 Azure AD 連接器和 Azure AD 將應用程式與 Azure AD 整合之後，您可以管理對應用程式的存取的方式。

* [使用 Azure AD 管理應用程式的存取](what-is-access-management.md)
* [使用 Azure AD 連接器自動化](../app-provisioning/user-provisioning.md)
* [將使用者指派給應用程式](./assign-user-or-group-access-portal.md)
* [將群組指派給應用程式](./assign-user-or-group-access-portal.md)
* [共用帳戶](../enterprise-users/users-sharing-accounts.md)

## <a name="next-steps"></a>後續步驟
如需深入資訊，您可以從 [GitHub](../fundamentals/active-directory-deployment-plans.md) 下載 Azure Active Directory 部署計劃。 針對資源庫應用程式，您可以透過 [Azure 入口網站](https://portal.azure.com)下載單一登入、條件式存取和使用者布建的部署計畫。 

若要從 Azure 入口網站下載部署計劃：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [**企業應用程式**]  |  **挑選應用程式**  |  **部署計畫**。

請接受[部署計劃問卷調查](https://aka.ms/DeploymentPlanFeedback)，以提供關於部署計劃的意見反應。