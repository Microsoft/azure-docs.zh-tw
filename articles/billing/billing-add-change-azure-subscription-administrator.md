---
title: 新增或變更 Azure 系統管理員訂用帳戶角色| Microsoft Docs
description: 說明如何新增或變更 Azure 共同管理員、服務管理員和帳戶管理員
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing
ms.assetid: 13a72d76-e043-4212-bcac-a35f4a27ee26
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/14/2018
ms.author: genli
ms.openlocfilehash: b0e24e498acd823242b3613abb62df978466d56d
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918307"
---
# <a name="add-or-change-azure-subscription-administrators"></a>新增或變更 Azure 訂用帳戶系統管理員

若要管理對 Azure 資源的存取，您必須具有適當的系統管理員角色。 本文說明如何在訂用帳戶層級新增或變更使用者的系統管理員角色。

> [!div class="nextstepaction"]
> [協助改善 Azure 計費文件](https://go.microsoft.com/fwlink/p/?linkid=2010091)

## <a name="what-administrator-role-do-i-use"></a>我使用何種系統管理員角色？

Azure 有數個不同的角色。 若要管理對資源的存取，您可以使用傳統訂用帳戶系統管理員角色 (例如服務管理員和共同管理員)，或使用名為角色型存取控制 (RBAC) 的新式授權系統。 為了確保能更容易控制，以及簡化存取管理，建議您所有存取管理需求都使用 RBAC。 可能的話，建議您使用 RBAC 重新設定現有的存取原則。 如需詳細資訊，請參閱[什麼是角色型存取控制 (RBAC)](../role-based-access-control/overview.md) 和[了解 Azure 中的不同角色](../role-based-access-control/rbac-and-directory-admin-roles.md)。

<a name="add-an-admin-for-a-subscription"></a>

## <a name="add-an-rbac-owner-for-a-subscription-in-azure-portal"></a>在 Azure 入口網站中新增訂用帳戶的 RBAC 擁有者 

若要將某人新增為 Azure 訂用帳戶的管理員，請在訂用帳戶範圍為其指派[擁有者](../role-based-access-control/built-in-roles.md#owner)角色 (RBAC 角色)。 擁有者角色可以管理您指派之訂用帳戶中的資源，但沒有其他訂用帳戶的存取權限。

1. 請造訪 [Azure 入口網站中的**訂用帳戶**](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。
2. 選取您要授與存取權的訂用帳戶。
3. 選取 [新增] 。
   (如果沒有 [新增] 按鈕，表示您沒有權限新增權限。)
4. 在清單中選取 [存取控制 (IAM)]。
5. 在 [角色] 方塊中，選取 [擁有者]。 
6. 在 [存取權指派對象為] 方塊中，選取 [Azure AD 使用者、群組或應用程式]。 
7. 在 [選取] 方塊中，輸入要新增為擁有者的使用者電子郵件地址。 選取使用者，然後選取 [儲存]。

    ![顯示已選取 [擁有者] 角色的螢幕擷取畫面](./media/billing-add-change-azure-subscription-administrator/add-role.png)

這可授與使用者所有資源的完整存取權，包括將存取權委派給其他人的權限。 若要授與不同範圍 (例如資源群組) 的存取權，請瀏覽該範圍的 [存取控制 (IAM)] 刀鋒視窗。

## <a name="add-or-change-co-administrator"></a>新增或變更共同管理員

只有[擁有者](../role-based-access-control/built-in-roles.md#owner)才可新增為共同管理員。 具有[參與者](../role-based-access-control/built-in-roles.md#contributor)和[讀者](../role-based-access-control/built-in-roles.md#reader)等角色的其他使用者無法新增為共同管理員。

> [!TIP]
> 如果使用者需要管理 Azure 傳統部署，請將「擁有者」新增為共同管理員即可。 建議您所有其他用途都使用 RBAC。

1. 如果您尚未新增，請遵循上述指示將某人新增為擁有者。
2. **以滑鼠右鍵按一下**您剛才新增的「擁有者」使用者，然後選取 [新增為共同管理員]。 如果您沒有看到 [新增為共同管理員] 選項，請重新整理頁面或嘗試使用其他網際網路瀏覽器。 

    ![新增共同管理員的螢幕擷取畫面](./media/billing-add-change-azure-subscription-administrator/add-coadmin.png)

    若要移除共同管理員權限，請**以滑鼠右鍵按一下**「共同管理員」使用者，然後選取 [移除共同管理員]。

    ![移除共同管理員的螢幕擷取畫面](./media/billing-add-change-azure-subscription-administrator/remove-coadmin.png)

<a name="change-service-administrator-for-a-subscription"></a>

## <a name="change-the-service-administrator-for-an-azure-subscription"></a>變更 Azure 訂用帳戶的服務管理員

只有帳戶管理員可以變更訂用帳戶的服務管理員。 根據預設，註冊時系統會將服務管理員與帳戶管理員設為同一人。 如果服務管理員變更為不同的使用者，帳戶管理員就會失去 Azure 入口網站的存取權。 不過，帳戶管理員可隨時使用帳戶中心，將自己變更回服務管理員。

1. 請參閱[變更服務管理員的限制](#limits)，確定您的情況是否受支援。
1. 以帳戶管理員身分登入[帳戶中心](https://account.windowsazure.com/subscriptions)。
1. 選取一個訂用帳戶。
1. 選取右側的 [編輯訂用帳戶詳細資料]。

    ![顯示帳戶中心裡 [編輯訂用帳戶] 按鈕的螢幕擷取畫面](./media/billing-add-change-azure-subscription-administrator/editsub.png)
1. 在 [服務管理員] 方塊中，輸入新服務管理員的電子郵件地址。

    ![顯示應變更服務管理員電子郵件之方塊的螢幕擷取畫面](./media/billing-add-change-azure-subscription-administrator/changeSA.png)

<a name="limits"></a>

### <a name="limitations-for-changing-service-administrators"></a>變更服務管理員的限制

* 每個訂用帳戶都與 Azure AD 目錄相關聯。 若要尋找與訂用帳戶相關聯的目錄，請前往[**訂用帳戶**](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)，然後選取訂用帳戶以查看目錄。
* 如果您以公司或學校帳戶登入，可以將組織中的其他帳戶新增為服務管理員。 例如，abby@contoso.com 可以將 bob@contoso.com 新增為服務管理員，但若想新增 john@notcontoso.com 為服務管理員，則 contoso.com 目錄中必須要有 john@notcontoso.com 才能這麼做。 以公司或學校帳戶登入的使用者可以繼續將 Microsoft 帳戶使用者新增為服務管理員。

  | 登入方法 | 要將 Microsoft 帳戶使用者新增為服務管理員嗎？ | 要將同一組織中的公司或學校帳戶新增為服務管理員嗎？ | 要將不同組織中的公司或學校帳戶新增為服務管理員嗎？ |
  | --- | --- | --- | --- |
  |  Microsoft 帳戶 |是 |否 |否 |
  |  公司或學校帳戶 |是 |yes |否 |

## <a name="change-the-account-administrator-for-an-azure-subscription"></a>變更 Azure 訂用帳戶的帳戶管理員

帳戶管理員就是一開始註冊 Azure 訂用帳戶的使用者，也是負責訂用帳戶的計費擁有者。 若要變更訂用帳戶的帳戶管理員，請參閱[將 Azure 訂用帳戶的擁有權轉移給另一個帳戶](billing-subscription-transfer.md)。

<a name="check-the-account-administrator-of-the-subscription"></a>

**不確定帳戶管理員是誰嗎？** 請遵循下列步驟：

1. 請造訪 [Azure 入口網站中的**訂用帳戶**](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。
1. 選取您想要檢查的訂用帳戶，然後查看 [設定]。
1. 選取 [屬性] 。 該訂用帳戶的帳戶管理員會顯示在 [帳戶管理員] 方塊中。  

## <a name="learn-more-about-resource-access-control-and-active-directory"></a>深入了解資源存取控制和 Active Directory

* 若要深入了解 RBAC，請參閱[什麼是角色型存取控制 (RBAC)？](../role-based-access-control/overview.md)
* 若要深入了解 Azure 中的所有角色，請參閱[了解 Azure 中的不同角色](../role-based-access-control/rbac-and-directory-admin-roles.md)。
* 如需有關 Azure Active Directory 的詳細資訊，請參閱 [Azure 訂用帳戶如何與 Azure Active Directory 產生關聯](../active-directory/active-directory-how-subscriptions-associated-directory.md)和[指派 Azure Active Directory 中的管理員角色](../active-directory/users-groups-roles/directory-assign-admin-roles.md)。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
