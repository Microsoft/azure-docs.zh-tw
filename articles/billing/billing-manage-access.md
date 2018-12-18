---
title: 使用角色來管理對 Azure 帳單的存取 | Microsoft Docs
description: ''
services: ''
documentationcenter: ''
author: vikramdesai01
manager: vikdesai
editor: ''
tags: billing
ms.assetid: e4c4d136-2826-4938-868f-a7e67ff6b025
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/22/2017
ms.author: cwatson
ms.openlocfilehash: 623856f05eed44eca3752d56f047f9bb282bdc8e
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392072"
---
# <a name="manage-access-to-billing-information-for-azure-using-role-based-access-control"></a>使用角色型存取控制來管理對 Azure 帳單資訊的存取

您可以透過將下列其中一個使用者角色指派給您的訂用帳戶，將 Azure 帳單資訊的存取權授與您的小組成員：帳戶管理員、服務管理員、共同管理員、擁有者、參與者，以及帳單讀者。 他們將可在 [Azure 入口網站](https://portal.azure.com/)中存取帳單資訊，並且可以使用[計費 API](billing-usage-rate-card-overview.md) 透過程式設計方式取得發票 (選擇加入後) 和使用量詳細資料。 如需有關角色授權者及角色功能的詳細資訊，請參閱 [Azure RBAC 中的角色](../role-based-access-control/built-in-roles.md)。

## <a name="opt-in"></a> 允許額外的使用者存取發票

「帳戶管理員」必須使用 [Azure 入口網站](https://portal.azure.com/)來加入其他使用者，才能允許這些使用者及透過 API 存取發票。

1. 如果您是「帳戶管理員」，請從 Azure 入口網站中的 [[訂用帳戶] 刀鋒視窗](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)選取您的訂用帳戶。

1. 選取 [發票]，然後選取 [存取發票]。

    ![螢幕擷取畫面顯示如何委派存取發票](./media/billing-manage-access/AA-optin.png)

1. **開啟**存取權，然後儲存變更，以允許訂用帳戶限域角色中的使用者下載發票。

    ![螢幕擷取畫面顯示委派存取發票的開關](./media/billing-manage-access/AA-optinAllow.png)

加入即可讓訂用帳戶上的「服務管理員」、「共同管理員」、「擁有者」、「參與者」、「讀者」及「帳單讀者」在 Azure 入口網站中下載 PDF 發票。 不過，2016 年 12 月以前的發票目前僅供「帳戶管理員」使用。

「帳戶管理員」也可以設定透過電子郵件傳送發票。 若要深入了解，請參閱[透過電子郵件取得發票](billing-download-azure-invoice-daily-usage-date.md)。

## <a name="adding-users-to-the-billing-reader-role"></a>將使用者新增到帳單讀者角色

「帳單讀者」角色可在 Azure 入口網站中以唯讀方式存取訂用帳戶帳單資訊，但無法存取 VM 和儲存體帳戶之類的服務。 請將「帳單讀者」角色指派給需要存取訂用帳戶帳單資訊但不需要能夠管理 Azure 服務的使用者。 此角色適用於組織內只負責執行 Azure 訂用帳戶之財務和成本管理的使用者。

1. 從 Azure 入口網站中的 [[訂用帳戶] 刀鋒視窗](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)選取您的訂用帳戶。

1. 選取 [存取控制 (IAM)]，然後按一下 [新增]。

    ![顯示 [訂用帳戶] 刀鋒視窗中 IAM 的螢幕擷取畫面](./media/billing-manage-access/select-iam.PNG)

1. 在 [選取角色] 頁面中，選取 [帳單讀者]。

    ![顯示快顯檢視中 [帳單讀者] 的螢幕擷取畫面](./media/billing-manage-access/select-roles.PNG)

1. 輸入您想要邀請之使用者的電子郵件，然後按一下 [確定] 以傳送邀請。

    ![顯示輸入電子郵件以邀請某位使用者的螢幕擷取畫面](./media/billing-manage-access/add-user.PNG)

1. 依照邀請電子郵件中的指示，以「帳單讀者」身分登入。

    ![顯示「帳單讀者」在 Azure 入口網站中可見內容的螢幕擷取畫面](./media/billing-manage-access/billing-reader-view.png)

> [!NOTE]
> 「帳單讀者」功能目前為預覽版，尚未支援企業 (EA) 訂用帳戶或非全球雲端。

## <a name="adding-users-to-other-roles"></a>將使用者新增到其他角色

具備其他角色 (例如「擁有者」和「參與者」) 的使用者不僅可存取帳單資訊，還可存取 Azure 服務。 若要管理這些角色，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../role-based-access-control/role-assignments-portal.md)。

## <a name="who-can-access-the-account-centerhttpsaccountwindowsazurecom"></a>誰可以存取[帳戶中心](https://account.windowsazure.com)？

只有「帳戶管理員」可以登入「帳戶中心」。 「帳戶管理員」是訂用帳戶的法定擁有者。 註冊或購買 Azure 訂用帳戶的人員預設即是「帳戶管理員」，除非[已將訂用帳戶擁有權轉移](billing-subscription-transfer.md)給其他人。 「帳戶管理員」可以建立訂用帳戶、取消訂用帳戶、變更訂用帳戶的帳單地址，以及管理訂用帳戶的存取原則。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
