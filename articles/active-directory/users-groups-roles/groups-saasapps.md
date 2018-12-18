---
title: 使用群組管理 SaaS 應用程式的存取權 | Microsoft Docs
description: 如何使用 Azure Active Directory Premium 或 Basic 中的群組指派與 Azure Active Directory 整合的 SaaS 應用程式存取權。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 03/14/2017
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 5d8e6c0db2ce6c35935c2b00aa40de2a5d5a050f
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297617"
---
# <a name="using-a-group-to-manage-access-to-saas-applications"></a>使用群組管理 SaaS 應用程式的存取權
使用 Azure Active Directory (Azure AD) 搭配 Azure AD Premium 或 Azure AD Basic 授權，您可以使用群組指派與 Azure AD 整合之 SaaS 應用程式的存取權。 例如，如果您想要指派行銷部門使用五個不同 SaaS 應用程式的存取權，可以建立一個包含行銷部門中的使用者的群組，然後將該群組指派給行銷部門需要的這五個 SaaS 應用程式。 如此一來，您可以在同一個地方管理行銷部門的成員資格以節省時間。 接著當使用者新增為行銷群組的成員時，會將這些使用者指派給應用程式，並在使用者從行銷群組中移除時，從應用程式中移除其指派。 此功能可以搭配您能夠從 Azure AD 應用程式庫中新增的數百種應用程式使用。

> [!IMPORTANT]
> 唯有啟動 Azure AD Premium 試用或購買 Azure AD Premium 或 Azure AD Basic 授權後，您才可以使用這項功能。 以群組為基礎的指派僅支援安全性群組。 目前對應用程式的群組式指派並不支援巢狀群組成員資格。

**將使用者或群組存取權指派給 SaaS 應用程式**

1. 在 [Azure AD 系統管理中心](https://aad.portal.azure.com)內，選取 [企業應用程式]。
2. 選取從應用程式庫新增的應用程式以加以開啟。
3. 選取 [使用者和群組]，然後選取 [新增使用者]。
4. 在 [新增指派] 上，選取 [使用者和群組] 以開啟 [使用者和群組] 選擇清單。
6. 視需要選取多個群組或使用者，然後按一下或點選 [選取] 將其新增至 [新增指派] 清單。 在這個階段，您也可以將角色指派給使用者。
7. 選取 [指派] 以將使用者或群組指派給選取的企業應用程式。

### <a name="next-steps"></a>後續步驟
這些文章提供有關 Azure Active Directory 的其他資訊。

* [使用 Azure Active Directory 群組來管理資源的存取權](../fundamentals/active-directory-manage-groups.md)
* [Azure Active Directory 中的應用程式管理](../manage-apps/what-is-application-management.md)
* [設定群組設定的 Azure Active Directory Cmdlet](groups-settings-cmdlets.md)
* [什麼是 Azure Active Directory？](../fundamentals/active-directory-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
