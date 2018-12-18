---
title: 嘗試登入 Azure 入口網站或 Azure 帳戶中心時，未在訂用帳戶中找到任何錯誤 | Microsoft Docs
description: 針對登入 Azure 入口網站或 Azure 帳戶中心時，未在訂用帳戶中找到任何錯誤這個問題，提供解決方案。
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing
ms.assetid: d1545298-99db-4941-8e97-f24a06bb7cb6
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: cwatson
ms.openlocfilehash: a1e90f946508f1ffc0a1ee812dde46ee733d715a
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392428"
---
# <a name="no-subscriptions-found-error-in-azure-portal-or-azure-account-center"></a>在 Azure 入口網站或 Azure 帳戶中心內，未於訂用帳戶中找到任何錯誤

嘗試登入 [Azure 入口網站](https://portal.azure.com/)或 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)時，您可能會收到「找不到訂用帳戶」錯誤訊息。 此文章會提供此問題的解決方案。

## <a name="symptom"></a>徵狀

當您嘗試登入 [Azure 入口網站](https://portal.azure.com/)或 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)時，會收到下列錯誤訊息：「找不到訂用帳戶」。

## <a name="cause"></a>原因

如果您選取錯誤的目錄或帳戶沒有足夠的權限，就會發生此問題。 

## <a name="solution"></a>解決方法

### <a name="scenario-1-error-message-is-received-in-the-azure-portalhttpsportalazurecom"></a>情節 1：在 [Azure 入口網站](https://portal.azure.com)收到錯誤訊息

若要修正此問題：

* 確定您已選取正確的 Azure 目錄，方法是按一下右上角的帳戶。

  ![選取 Azure 入口網站右上角的目錄](./media/billing-no-subscriptions-found/directory-switch.png)
* 若已選取正確的 Azure 目錄，但您仍收到錯誤訊息，請[將擁有者角色指派給您的帳戶](../role-based-access-control/role-assignments-portal.md)。

### <a name="scenario-2-error-message-is-received-in-the-azure-account-centerhttpsaccountwindowsazurecomsubscriptions"></a>案例 2：在 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)中收到錯誤訊息

請檢查您所使用的帳戶是否為帳戶管理員。 若要確認誰是帳戶管理員，請依照下列步驟操作︰

1. 登入 [Azure 入口網站中的訂用帳戶檢視](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。
1. 選取您想要檢查的訂用帳戶，然後查看 [設定]。
1. 選取 [屬性] 。 該訂用帳戶的帳戶管理員會顯示在 [帳戶管理員]  方塊中。  

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](http://go.microsoft.com/fwlink/?linkid=544831&clcid=0x409)以快速解決您的問題。 
