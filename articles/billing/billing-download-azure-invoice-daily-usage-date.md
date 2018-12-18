---
title: 下載 Azure 帳單發票和每日使用量資料 | Microsoft Docs
description: 說明如何下載或檢視您的 Azure 帳單發票和每日使用量資料。
keywords: 帳單發票,發票下載,azure 發票,azure 使用量
services: billing
documentationcenter: ''
author: genlin
manager: tonguyen
editor: ''
tags: billing
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: cwatson
ms.openlocfilehash: f0cdfef50c07674a08766933f2f7edfc946462a4
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395010"
---
# <a name="download-or-view-your-azure-billing-invoice-and-daily-usage-data"></a>下載或檢視您的 Azure 帳單發票和每日使用量資料
您可以從 [Azure 入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)下載您的發票，或透過電子郵件傳送它。 若要下載您的每日使用量，請移至 [Azure 帳戶中心](https://account.azure.com/Subscriptions)。 只有某些角色有權取得帳單發票和使用量資訊，例如帳戶管理員。 若要深入了解取得計費資訊的存取權，請參閱[使用角色管理 Azure 計費的存取權](billing-manage-access.md)。

本文不適用於 Enterprise 合約 (EA) 客戶。 如果您是 EA 客戶，系統會將您的發票直接傳送給申請管理員。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

> [!div class="nextstepaction"]
> [協助改善 Azure 計費文件](https://go.microsoft.com/fwlink/p/?linkid=2010091)

## <a name="get-your-invoice-in-email-pdf"></a>透過電子郵件取得發票 (.pdf)
您可以選擇加入並設定其他收件者來透過電子郵件接收您的 Azure 發票。 這項功能可能無法用於支援供應項目、Enterprise 合約或 Azure in Open 等特定訂用帳戶。

1. 在[訂用帳戶頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)中，選取您的訂用帳戶。 針對您擁有的每個訂用帳戶選擇加入。 依序按一下 [發票] 和 [以電子郵件寄送我的發票]。 

    ![顯示選擇加入流程的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/InvoicesDeepLink.PNG)
    
2. 按一下 [選擇加入] 並接受條款。

    ![顯示加入流程步驟 2 的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/InvoiceArticleStep2.PNG)
 
3. 接受合約後，您就可以設定其他收件者。 移除收件者時，不會再儲存電子郵件地址。 如果您改變心意，則必須予以重新新增。

    ![顯示加入流程步驟 3 的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/InvoiceArticleStep3.PNG)
    
如果在依照這些步驟執行之後並未收到電子郵件，請在[設定檔的通訊喜好設定](https://account.windowsazure.com/profile)中確定您的電子郵件地址是正確的。

### <a name="opt-out-from-getting-your-invoice-in-email"></a>選擇不要透過電子郵件取得發票
如果您不想透過電子郵件取得發票，請按一下 [退出以電子郵件傳送發票]。 這會移除任何已設定為透過電子郵件接收發票的電子郵件地址。 如果您選擇重新加入，則必須重新設定收件者。

 ![顯示退出流程的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/InvoiceArticleStep4.PNG)

## <a name="download-invoice-from-azure-portal-pdf"></a>從 Azure 入口網站下載發票 (.pdf)

1. 以[具有發票存取權的使用者](billing-manage-access.md)身分，在 Azure 入口網站中的 [訂用帳戶] [頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)上選取您的訂用帳戶。

2. 選取 [發票]。 

    ![顯示 [帳單和使用量] 選項的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/billingandusage.png) 

3. 按 [下載發票] 以檢視 PDF 發票的副本。 如果它顯示**無法使用**，請參閱[為什麼我沒有看到上期的發票？](#noinvoice)

    ![顯示計費期間、下載發票和每個計費期間總費用的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/billing4.png)

4. 按一下計費期間，還可檢視每日使用量。 

如需發票的詳細資訊，請參閱[了解 Microsoft Azure 帳單](billing-understand-your-bill.md)。 如需管理成本上的協助，請參閱[使用 Azure 計費與成本管理避免非預期的成本](billing-getting-started.md)。

## <a name="download-usage-from-the-account-center-csv"></a>從帳戶中心下載使用量 (.csv)

1. 以帳戶管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/subscriptions)。

2. 選取您需要發票和使用資訊的訂用帳戶。

3. 選取 [帳單記錄]。 

    ![顯示帳單記錄選項的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/Billinghisotry.png)

4. 您可以查看過去六個計費期間和目前未收帳期間的對帳單。 

    ![顯示計費期間、下載發票和每日使用量的選項，以及每個計費期間總費用的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/billingSum.png)

5. 選取 [檢視目前的對帳單]，以查看產生估計時的估計費用。 此資訊每天只會更新一次，可能不會包含您的所有使用量。 您的每月發票可能會與這項估計有所不同。

    ![顯示 [檢視目前對帳單] 選項的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/billingSum2.png)

    ![顯示目前費用預估的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/billingSum3.png)

6. 選取 [下載使用量]，以 CSV 檔案形式下載每日使用量資料。 如果您看到兩個可用版本，請下載第 2 個版本。

    ![顯示 [下載使用量] 選項的螢幕擷取畫面](./media/billing-download-azure-invoice-daily-usage-date/DLusage.png)

只有帳戶管理員可以存取 Azure 帳戶中心。 其他計費系統管理員 (例如擁有者) 可以使用 [計費 API](billing-usage-rate-card-overview.md)取得使用量資訊。

如需每日使用量的詳細資訊，請參閱[了解 Microsoft Azure 帳單](billing-understand-your-bill.md)。 如需管理成本上的協助，請參閱[使用 Azure 計費與成本管理避免非預期的成本](billing-getting-started.md)。

## <a name="noinvoice"></a>為什麼我沒有看到上期的發票？

您沒有看到發票的可能原因如下︰

- 您的訂用帳戶有尚未超過的每月信用額度，或您有免費試用版。 只有在您積欠費用時，才會產生發票。

- 您的 Azure 訂閱不滿 30 天。

- 尚未產生發票。 等到計費期間結束。

- 如果您不是帳戶管理員，可能無法存取較舊的發票。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。

