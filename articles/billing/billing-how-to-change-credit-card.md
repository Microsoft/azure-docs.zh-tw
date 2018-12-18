---
title: 變更用於支付 Azure 訂閱費的信用卡 | Microsoft Docs
description: 描述如何變更用來支付 Azure 訂用帳戶的信用卡
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing
ms.assetid: 15252ced-1841-4a66-ae79-2e58af1d3370
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 6b0db12da7c187ce80f1d55ce8b9900601033068
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34068308"
---
# <a name="add-update-or-remove-a-credit-or-debit-card-for-azure"></a>新增、更新或移除用於支付 Azure 訂閱費的信用卡或轉帳卡

您可以在帳戶中心新增信用卡、更新現有的信用卡或刪除不再使用的信用卡。 您必須是[帳戶管理員](billing-subscription-transfer.md#whoisaa)才能進行這些變更。

**想要改成按發票付款嗎？** 請參閱[按發票支付 Azure 訂閱費](billing-how-to-pay-by-invoice.md)。
 
<a id="addcard"></a>

## <a name="add-a-new-credit-or-debit-card"></a>新增信用卡或轉帳卡

1. 以帳戶管理員身分登入[帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取一個訂用帳戶。
1. 在頁面右側，選取 [管理付款方式] 。

    ![顯示已選取 [管理付款方式] 選項的螢幕擷取畫面。](./media/billing-how-to-change-credit-card/changesub_new.png)
1. 選取 [+] 以新增信用卡/轉帳卡。

    ![顯示付款方式旁編輯選項的螢幕擷取畫面。](./media/billing-how-to-change-credit-card/editcard_new.png)
1. 輸入信用卡或轉帳卡詳細資料。
1. 選取 [ **儲存**]。 

如果在新增信用卡之後收到錯誤，請參閱[在 Azure 註冊時信用卡遭拒](billing-credit-card-fails-during-azure-sign-up.md)。

## <a name="update-existing-credit-or-debit-card"></a>更新現有的信用卡或轉帳卡

如果您的信用卡換卡，但號碼維持不變，請更新現有信用卡的詳細資料即可，例如有效期限。 如果您的信用卡號碼因為卡片遺失、遭竊或過期而變更，請遵循[將信用卡新增為付款方式](#addcard)一節中的步驟。 您不需要更新 CVV。

1. 以帳戶管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取連結至信用卡/轉帳卡的訂閱。
1. 選取 [管理付款方式]。
1. 選取要更新的信用卡/轉帳卡旁的 [編輯]。
1. 更新信用卡或轉帳卡詳細資料。
1. 選取 [ **儲存**]。

## <a name="use-a-different-credit-card-for-the-azure-subscription"></a>針對 Azure 訂閱使用另一張信用卡

1. 以帳戶管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取連結至信用卡/轉帳卡的訂閱。
1. 在頁面右側，選取 [管理付款方式] 。
1. 在您要使用的信用卡/轉帳卡旁按一下 [改用]。 這也會更新其他目前與該信用卡/轉帳卡相關聯的訂閱。 

## <a name="remove-a-credit-or-debit-card-from-the-account"></a>從帳戶中移除信用卡或轉帳卡

1. 以帳戶管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取連結至信用卡/轉帳卡的訂閱。
3. 在頁面右側，選取 [管理付款方式] 。
4. 對您要刪除的信用卡，按一下 [刪除]。

如果您的信用卡與其他使用中 Microsoft 訂用帳戶相關聯，您就無法將它從 Azure 帳戶中移除。 從所有有效的 Microsoft 訂閱中移除該信用卡，並再試一次。

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="my-subscription-is-disabled-why-cant-i-remove-my-credit-card-now"></a>我的訂閱已停用。 為什麼無法立即移除信用卡？

停用或取消訂閱後，需要 90 天系統才會將其永久刪除。 我們在保留期間會保留您的付款方式，以防您想重新啟用訂閱。 保留期過後，系統便會完全刪除訂閱。

如果您在 90 天的保留期結束前需要移除信用卡或轉帳卡，請[重新啟用訂用帳戶](billing-subscription-become-disable.md)。 如需無法重新啟用，請[連絡 Azure 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

### <a name="why-do-i-keep-getting-your-login-session-has-expired-please-click-here-to-log-back-in"></a>為什麼我一直收到「您的登入工作階段已過期。 請按一下這裡重新登入」錯誤訊息？

如果您已登出並重新登入，但還是一直收到此錯誤訊息，請用私密瀏覽工作階段再試一次。

### <a name="how-do-i-use-a-different-card-for-each-subscription-i-have"></a>如何針對我的每個訂閱使用不同的信用卡/轉帳卡？

很抱歉，如果您已經為所有的訂閱設定使用相同的信用卡/轉帳卡，就不能改用其他信用卡。 不過，註冊新訂閱時，可以為該訂閱選擇新的付款方式。

### <a name="how-do-i-make-payments"></a>如何付款？

如果您將信用卡或轉帳卡設定為付款方式，我們便會在每次計費期間結束後自動向您的信用卡/轉帳卡扣款。 您不需要進行任何動作。

如果您使用[按發票付款](billing-how-to-pay-by-invoice.md)，請將您的款項寄到發票底部所列的地址。

### <a name="how-do-i-make-a-one-time-payment"></a>如何進行單次付款？

很抱歉，Azure 目前未針對信用卡或轉帳卡提供單次付款支援。 

### <a name="how-do-i-change-the-tax-id"></a>如何變更統一編號？

若要新增或更新統一編號，請造訪 [Azure 帳戶中心內的 [設定檔]](https://account.azure.com/Profile)，然後選取 [稅務記錄]。 此統一編號會用於計算免稅金額，並出現在您的發票上。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
