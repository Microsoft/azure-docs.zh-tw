---
title: 開始將 Azure AD 與應用程式整合 | Microsoft Docs
description: 本文章是整合 Azure Active Directory (AD) 與在內部部署應用程式和雲端應用程式的入門指南。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/16/2018
ms.author: barbkess
ms.reviewer: asteen
ms.openlocfilehash: d52ec316f9f5540d4d0d0fe0bc4e4bf778e1daf7
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44345435"
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
* 您如何目前管理使用者對應用程式存的取？ 需要變更嗎？  曾經考量過管理存取的其他方式，例如使用 [RBAC](../../role-based-access-control/role-assignments-portal.md) 之類？
* 誰需要存取哪些內容？

也許您事先對這所有問題沒有答案，但是沒關係。  本指南可協助您回答其中一些問題，並做出一些明智的決策。

### <a name="find-unsanctioned-cloud-applications-with-cloud-discovery"></a>使用 Cloud Discovery 尋找待批准的雲端應用程式

如上所述，可能有應用程式到目前為止仍未受到組織的管理。  作為清查程序的一部分，您可以找到未經約束的雲端應用程式。 請參閱[設定 Cloud Discovery](/cloud-app-security/set-up-cloud-discovery)。

## <a name="integrating-applications-with-azure-ad"></a>整合應用程式與 Azure AD
以下文章將討論整合應用程式與 Azure AD 的各種不同方式，並提供一些指引。

* [決定要使用的 Active Directory](../fundamentals/active-directory-administer.md)
* [使用 Azure 應用程式資源庫中的應用程式](what-is-single-sign-on.md)
* [整合 SaaS 應用程式教學課程清單](../active-directory-saas-tutorial-list.md)

### <a name="authentication-types"></a>驗證類型
每個應用程式可能有不同的驗證需求。 利用 Azure AD，可將簽署憑證用於使用 SAML 2.0、WS-同盟或 OpenID Connect 通訊協定，以及密碼單一登入的應用程式。 如需要與 Azure AD 搭配使用的應用程式驗證類型的詳細資訊，請參閱[在 Azure Active Directory 中管理同盟單一登入的憑證](manage-certificates-for-federated-single-sign-on.md)和[密碼式單一登入](what-is-single-sign-on.md)。

### <a name="enabling-sso-with-azure-ad-app-proxy"></a>使用 Azure AD 應用程式 Proxy 啟用 SSO
透過 Microsoft Azure AD 應用程式 Proxy，您可以從任何地方及任何裝置上安全地為位於您的私人網路上的應用程式提供存取。 在您的環境中安裝應用程式 Proxy 連接器之後，可以輕鬆地使用 Azure AD 來加以設定。

### <a name="integrating-custom-applications"></a>整合自訂應用程式
如果您正在撰寫新的應用程式，並想要協助開發人員運用 Azure AD 的強大功能，請參閱 [引導開發人員](../active-directory-applications-guiding-developers-for-lob-applications.md)。

如果您想要加入您的自訂應用程式至 Azure 應用程式資源庫，請參閱 [使用 Azure AD 自助 SAML 組態「自備應用程式」](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-now-in-preview/)。

## <a name="managing-access-to-applications"></a>管理應用程式的存取
下列文章描述使用 Azure AD 連接器和 Azure AD 將應用程式與 Azure AD 整合之後，您可以管理對應用程式的存取的方式。

* [使用 Azure AD 管理應用程式的存取](what-is-access-management.md)
* [使用 Azure AD 連接器自動化](user-provisioning.md)
* [將使用者指派給應用程式](../active-directory-applications-guiding-developers-assigning-users.md)
* [將群組指派給應用程式](../active-directory-applications-guiding-developers-assigning-groups.md)
* [共用帳戶](../active-directory-sharing-accounts.md)

## <a name="next-steps"></a>後續步驟
如需深入資訊，您可以從 [GitHub](https://aka.ms/deploymentplans) 下載 Azure Active Directory 部署計劃。 對於資源庫應用程式，您可以透過 [Azure 入口網站](https://portal.azure.com)下載單一登入、條件式存取和使用者佈建的部署計劃。 

若要從 Azure 入口網站下載部署計劃：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [企業應用程式] | [挑選應用程式] | [部署計劃]。

請接受[部署計劃問卷調查](https://aka.ms/DeploymentPlanFeedback)，以提供關於部署計劃的意見反應。
