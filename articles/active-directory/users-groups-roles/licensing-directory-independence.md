---
title: Azure Active Directory 租用戶互動的特性 | Microsoft Docs
description: 了解您的目錄為完全獨立的資源，以管理 Azure Active Directory 租用戶
services: active-tenant
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: article
ms.workload: identity
ms.component: users-groups-roles
ms.date: 10/10/2017
ms.author: curtand
ms.custom: H1Hack27Feb2017;it-pro
ms.reviewer: piotrci
ms.openlocfilehash: 85f7cddb7231bf9c5e45de87af3c922148f214be
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37861196"
---
# <a name="understand-how-multiple-azure-active-directory-tenants-interact"></a>了解多個 Azure Active Directory 租用戶如何互動

在 Azure Active Directory (Azure AD) 中，每個租用戶都是完全獨立的資源：邏輯上獨立於您所管理之其他租用戶的對等。 租用戶之間沒有任何父子關聯性。 這個在租用戶之間的獨立性包括資源獨立性、系統管理獨立性，以及同步處理獨立性。

## <a name="resource-independence"></a>資源獨立性
* 當您在某個租用戶中建立或刪除資源時，並不會影響另一個租用戶中的任何資源 (但有外部使用者有部分例外)。 
* 如果將您的其中一個網域名稱與某個租用戶搭配使用，該網域名稱就無法與任何其他租用戶搭配使用。

## <a name="administrative-independence"></a>系統管理獨立性
如果 'Contoso' 租用戶的非系統管理使用者建立 'Test' 測試租用戶，則：

* 系統預設會將建立租用戶的使用者新增為該新租用戶的外部使用者，並將該租用戶中的全域管理員角色指派給他。
* 'Contoso' 租用戶的系統管理員並沒有 'Test' 租用戶的直接系統管理權限，除非 'Test' 的系統管理員明確地授與他們這些權限。 不過，'Contoso' 的系統管理員如果可控制建立 'Test' 的使用者帳戶，便可控制對 'Test' 租用戶的存取權。
* 如果您為某個使用者新增/移除在某個租用戶中的系統管理員角色，此變更並不會影響到該使用者在另一個租用戶中擁有的系統管理員角色。

## <a name="synchronization-independence"></a>同步處理獨立性
您可以單獨設定每個 Azure AD 租用戶，以從下列任一工具的單一執行個體同步處理資料：

* Azure AD Connect 工具：可與單一 AD 樹系同步處理資料。
* Azure Active Directory Connector for Forefront Identity Manager：可與一或多個內部部署樹系和/或非 Azure AD 資料來源同步處理資料。

## <a name="add-an-azure-ad-tenant"></a>新增 Azure AD 租用戶
若要在 Azure 入口網站中新增 Azure AD 租用戶，請使用 Azure AD 全域管理員的帳戶登入[Azure 入口網站](https://portal.azure.com)，並且在左邊選取 [新增]。

> [!NOTE]
> 與其他 Azure 資源不同，您的租用戶並非 Azure 訂用帳戶的子資源。 如果您的 Azure 訂用帳戶被取消或到期，您仍然可以使用 Azure PowerShell、Azure Graph API 或「Office 365 系統管理中心」來存取租用戶資料。 您也可以[將另一個訂用帳戶與租用戶建立關聯](../fundamentals/active-directory-how-subscriptions-associated-directory.md)。
>

## <a name="next-steps"></a>後續步驟
如需 Azure AD 授權問題和最佳做法的廣泛概觀，請參閱[什麼是 Azure Active Directory 租用戶授權？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)。
