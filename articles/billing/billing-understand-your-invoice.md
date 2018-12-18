---
title: 了解您的 Azure 發票 | Microsoft Docs
description: 了解如何閱讀並了解 Azure 訂用帳戶的使用情形和帳單
services: ''
documentationcenter: ''
author: tonguyen10
manager: tonguyen
editor: ''
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/31/2017
ms.author: tonguyen
ms.openlocfilehash: 6601ade7db52043de8150a3fca671591b9b469fd
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37063454"
---
# <a name="understand-terms-on-your-microsoft-azure-invoice"></a>了解您 Microsoft Azure 發票上的字詞

發票會提供您費用的摘要，並提供付款的指示。 您可以從 [Azure 入口網站](https://portal.azure.com/)下載可攜式文件格式 (.pdf) 的發票，或可透過電子郵件傳送。 如需詳細資訊，請參閱[如何取得您的 Azure 帳單發票和每日使用量資料](billing-download-azure-invoice-daily-usage-date.md)。

幾個注意事項：

-   如果您使用的是免費試用的訂用帳戶，可以從 Azure 入口網站取得每日使用量資訊，但是您沒有發票。

-   在上一個計費期間結束時，最多將會有 24 小時的使用量顯示在您目前的發票中。

-   針對國際客戶而列在帳單明細上的費用僅為估算用途。 銀行換匯的費用將有所不同。

>[!VIDEO https://www.youtube.com/embed/jWG1lyJe3Mg]

## <a name="detailed-terms-and-descriptions-of-your-invoice"></a>您發票的詳細字詞和描述
下列各節列出您在發票上看到的大部分字詞以及每個字詞的說明。

### <a name="account-information"></a>帳戶資訊

發票的帳戶資訊區段位於第一頁的頂端，並會顯示您設定檔和訂用帳戶的相關資訊。

![發票的帳戶資訊區段](./media/billing-understand-your-invoice/1.png)

| 詞彙 | 說明 |
| --- | --- |
| 客戶訂購單號碼。 |您選擇性指派以進行追蹤的訂購單號碼 |
| 發票編號 |用於追蹤的 Microsoft 所產生之唯一發票編號 |
| 計費週期 |此發票涵蓋的日期範圍 |
| 發票日期 |發票產生的日期，通常是計費週期結束之後的一天 |
| 付款方法 |帳戶 (發票或信用卡) 上所使用的付款類型 |
| 帳單收件者 |為帳戶所列的帳單地址 |
| 訂用帳戶供應項目 (「隨收隨付」) |已購買之訂用帳戶供應項目的類型 (隨用隨付、BizSpark Plus、Azure Pass 等等)。 如需詳細資訊，請參閱 [Azure 供應項目類型](https://azure.microsoft.com/support/legal/offer-details/)。 |
| 帳戶擁有者的電子郵件 | 註冊 Microsoft Azure 時所使用的帳戶電子郵件地址。 <br /><br />若要變更電子郵件地址，請參閱[如何變更連絡人電子郵件、地址和電話號碼等 Azure 帳戶的設定檔資訊](billing-how-to-change-azure-account-profile.md)。 |

### <a name="understand-the-invoice-summary"></a>了解發票摘要
帳單的 [發票摘要] 區段會列出上次帳單週期之後的總交易量以及目前的使用量費用。

![發票摘要區段](./media/billing-understand-your-invoice/2.png)

訂用帳戶名稱 (「實際執行存放裝置」) 是此發票的訂用帳戶名稱。

#### <a name="understand-the-previous-charges"></a>了解先前的費用
發票的前期結餘、付款與未付餘額區段會摘要說明上次計費期間之後的交易。

| 詞彙 | 說明 |
| --- | --- |
| 前期結餘 |上次計費期間算起的應付總金額 |
| 付款 |套用至上次計費期間的總付款金額及信用額度 |
| 未付餘額 (從上一個計費週期) |上次計費期間起您帳戶中的任何信用額度或剩餘結餘 |

#### <a name="understand-the-current-charges"></a>了解目前費用
發票的 [目前費用] 區段會顯示有關目前帳單週期的每月費用詳細資料。

| 詞彙 | 說明 |
| --- | --- |
| 使用費用 |使用量費用是訂用帳戶目前帳單週期的每月總費用|
| 折扣 |目前計費期間適用的服務折扣|
| 調整 |目前帳單週期適用的其他信用額度 (免費使用量、信用額度等) 或未償費用。<br/><br/>例如，如果您享有 Visual Studio Enterprise 含 MSDN 供應項目，您會看到每月信用額度。 如果您取消訂用帳戶，您會看到超過您使用訂用帳戶供應項目獲得的每月信用額度的所有每月使用費用。 這些費用是目前計費期間開始日期到訂用帳戶取消日期所產生的。 |

#### <a name="sold-to-and-payment-instructions"></a>買方和付款指示

下表說明您發票第二頁上顯示的買方和付款指示。

| 詞彙 |說明 |
| --- | --- |
| 買方 |帳戶上的設定檔地址。 <br/><br/>如果您需要變更地址，請參閱[如何變更連絡人電子郵件、地址和電話號碼等 Azure 帳戶的設定檔資訊](billing-how-to-change-azure-account-profile.md)。|
| 付款指示 |依付款方式 (例如依信用卡或發票) 說明如何支付的指示。 |

#### <a name="usage-charges"></a>使用量費用

發票的使用量費用區段會顯示您費用計量層級的資訊。

![使用量費用區段](./media/billing-understand-your-invoice/3.png)

下表描述您發票上顯示的使用量費用資料行標頭。

| 詞彙 |說明 |
| --- | --- |
| Name |識別使用量的最上層服務 |
| 類型 |定義可能會影響費率的 Azure 服務類型 |
| 資源 |識別耗用計量的量值單位 |
| 區域 |針對以資料中心位置作為基礎所定價的某些服務，識別資料中心的位置 |
| 已耗用 |計費期間所使用之計量的數量 |
| 已包括 |目前計費期間免費提供之計量的數量 |
| 可計費 |顯示「已耗用的數量」和「已包括的數量」之間的差額。 我們會針對此數量向您收費。 對於不提供供應項目所含數量的隨收隨付供應項目，這個總數會與「已耗用的數量」相同 |
| 費率 |每個計費單位向您收取的費率 |
| 值 |顯示 [超額數量] 資料行和 [費率] 資料行兩者相乘的結果。 如果「已耗用的數量」沒有超過「已包括的數量」，則此資料行中沒有任何費用。 |
| 小計 |此計費期間所有費用稅前的總和 |
| 總計 |此計費期間所有費用稅後的總和 |

## <a name="how-do-i-make-sure-that-the-charges-in-my-invoice-are-correct"></a>如何確定我發票中的費用是否正確？
如果您需要發票費用的更多詳細資料，請參閱[了解您的 Microsoft Azure 帳單。](billing-understand-your-bill.md)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
