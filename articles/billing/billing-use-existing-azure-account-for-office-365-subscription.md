---
title: 使用 Azure 帳戶註冊 Office 365 | Microsoft Docs
description: 了解如何使用 Azure 帳戶建立 Office 365 訂用帳戶
services: ''
documentationcenter: ''
author: JiangChen79
manager: adpick
editor: ''
tags: billing,top-support-issue
ms.assetid: ''
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: cwatson
ms.openlocfilehash: d48b55af705eddb42853fb236e5eb6a214d1a0ff
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394042"
---
# <a name="sign-up-for-an-office-365-subscription-with-your-azure-account"></a>使用 Azure 帳戶註冊 Office 365 訂用帳戶
如果您是 Azure 訂閱者，您可以使用 Azure 帳戶來註冊 Office 365 訂用帳戶。 如果您屬於具有 Azure 訂用帳戶的組織，您可以為現有 Azure Active Directory (Azure AD) 中的使用者建立 Office 365 訂用帳戶。 使用具有 Azure Active Directory 租用戶中全域管理員或帳務管理員權限的帳戶，來註冊 Office 365。 如需詳細資訊，請參閱[在 Azure AD 中檢查我的帳戶權限](#RoleInAzureAD)和[在 Azure Active Directory 中指派系統管理員角色](../active-directory/users-groups-roles/directory-assign-admin-roles.md)。

如果您已經有 Office 365 帳戶和 Azure 訂用帳戶，您可以[將 Office 365 租用戶關聯至 Azure 訂用帳戶](billing-add-office-365-tenant-to-azure-subscription.md)。

## <a name="get-an-office-365-subscription-by-using-your-azure-account"></a>使用 Azure 帳戶取得 Office 365 訂用帳戶

1. 移至 [Office 365 產品頁面](https://products.office.com/business)，然後選取計劃。
2. 按一下位於頁面右上角的 [登入]。

    ![Office 365 試用頁面的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/12-office-365-trial-page.png)
3. 使用 Azure 帳戶認證進行登入。 如果您正在建立組織的訂用帳戶，請使用屬於 Azure Active Directory 租用戶中全域管理員或帳務管理員目錄角色成員的 Azure 帳戶。

    ![Office 365 登入的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/13-office-365-sign-in.png)
4. 按一下 [立即試用] 。

    ![確認您的 Office 365 訂單的螢幕擷取畫面。](./media/billing-use-existing-azure-account-office-365-subscription/14-office-365-confirm-your-order.png)
5. 在訂單收據頁面上，按一下 [繼續] 。

    ![Office 365 訂單收據的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/15-office-365-order-receipt.png)

這樣就準備好了。 如果您已建立組織的 Office 365 訂用帳戶，請使用下列步驟來檢查您的 Azure AD 使用者目前在 Office 365 中。

1. 開啟 Office 365 系統管理中心。
2. 展開 [使用者]，然後按一下 [作用中使用者]。

    ![Office 365 系統管理中心使用者的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/16-office-365-admin-center-users.png)

在您註冊之後，系統會將 Office 365 訂用帳戶新增至 Azure 訂用帳戶隸屬的同一個 Azure Active Directory 執行個體。 如需詳細資訊，請參閱[深入了解 Azure 和 Office 365 訂用帳戶](billing-use-existing-office-365-account-azure-subscription.md#more-about-subs)和[Azure 訂用帳戶如何與 Azure Active Directory 產生關聯](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)。

## <a id="RoleInAzureAD"></a>在 Azure AD 中檢查我的帳戶權限
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [所有服務]，然後搜尋 [Active Directory]。

    ![Azure 入口網站中 Active Directory 的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/billing-more-services-active-directory.png)
3. 按一下 [使用者和群組] > [所有使用者]。
4. 選取使用者名稱。 

    ![顯示 Azure Active Directory 使用者的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/billing-users-groups.png)

5. 按一下 [目錄角色]。
  
    ![顯示 Azure 入口網站目錄角色的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/billing-user-directory-role.png)
6.  需具備**全域管理員**或**受限的管理員** > 和**帳務管理員**角色，才能為現有 Azure Active Directory 中的使用者建立 Office 365 訂用帳戶。

    ![顯示 Azure 入口網站目錄角色帳務管理員的螢幕擷取畫面](./media/billing-use-existing-azure-account-office-365-subscription/billing-directoryrole-limited.png)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。 
