---
title: 了解 Azure 帳單 | Microsoft Docs
description: 學習如何閱讀並了解 Azure 訂用帳戶的使用情形和帳單
services: ''
documentationcenter: ''
author: tonguyen10
manager: tonguyen
editor: ''
tags: billing
ms.assetid: 32eea268-161c-4b93-8774-bc435d78a8c9
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/14/2018
ms.author: cwatson
ms.openlocfilehash: be2e774bc4220d5d93f555263fa9f342f449f8b2
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423437"
---
# <a name="understand-your-bill-for-microsoft-azure"></a>了解 Microsoft Azure 帳單
若要了解您的 Azure 帳單，請比較您包含詳細每日使用量檔案的發票，與 Azure 入口網站中的成本管理報告。

>[!NOTE]
>本文不適用於 Enterprise 合約 (EA) 客戶。 如果您是 EA 客戶，[可以到企業版入口網站上存取發票文件](https://ea.azure.com/helpdocs/understandingYourInvoice)  

若要取得發票的 PDF 和每日詳細使用量 CSV 檔案下載的複本，請參閱[取得 Azure 計費發票及每日使用量資料](billing-download-azure-invoice-daily-usage-date.md)。 

如需您發票的詳細字詞和描述，以及詳細的每日使用量檔案，請參閱[了解您 Microsoft Azure 發票上的字詞](billing-understand-your-invoice.md)和[了解您 Microsoft Azure 詳細使用量上的字詞](billing-understand-your-usage.md)。 

如需成本管理報告的詳細資料，請參閱 [Azure 入口網站成本管理](https://docs.microsoft.com/azure/billing/billing-getting-started)。

> [!div class="nextstepaction"]
> [協助改善 Azure 計費文件](https://go.microsoft.com/fwlink/p/?linkid=2010091)

## <a name="charges"></a>如何確定我發票中的費用是否正確？

>[!VIDEO https://www.youtube.com/embed/3YegFD769Pk]

如果您需要發票費用的更多詳細資料，有幾個選項供您選擇。

### <a name="option-1-review-your-invoice-and-compare-the-usage-and-costs-with-the-detailed-usage-csv-file"></a>選項 1：檢閱您的發票，並將使用量和成本與詳細使用量 CSV 檔案進行比較

詳細使用量 CSV 檔案會顯示依計費期間和每日使用量的費用。 若要取得詳細使用量 CSV 檔案，請參閱[取得您的 Azure 帳單發票和每日使用量資料](https://docs.microsoft.com/azure/billing/billing-download-azure-invoice-daily-usage-date)。

使用量費用會顯示在計量等級中。 下列字詞在發票和詳細使用量檔案代表相同意義。 例如，發票上的計費週期相當於詳細使用量檔案中所顯示的計費期間。

 | 發票 (PDF) | 詳細使用量 (CSV)|
 | --- | --- |
|計費週期 | 計費期間 |
 |名稱 |計量類別 |
 |類型 |計量子類別 |
 |資源 |計量名稱 |
 |區域 |計量區域 |
 |已耗用 |已耗用的數量 |
 |已包括 |已包括的數量 |
 |可計費 |超額數量 |

您發票的**使用量費用**區段會包含您計費期間所耗用的每個計量器總計值。 例如，下列螢幕擷取畫面顯示 Azure 排程器服務的使用量費用。

![發票使用量費用](./media/billing-understand-your-bill/1.png)

詳細使用量 CSV 的**明細**區段會顯示相同的費用。 「已使用」數量和「值」會與發票相符。

![CSV 使用量費用](./media/billing-understand-your-bill/2.png)

若要查看這筆費用的每日明細，請移至 CSV 的**每日使用量**區段。 在*計量表類別*下篩選「排程器」，即可查看哪幾天使用了計量以及使用了多少。 *資源*和*資源群組*資訊也會列出以供比較。 「已使用」值加起來應該等於發票上所顯示的值。

![CSV 中的 [每日使用量] 區段](./media/billing-understand-your-bill/3.png)

若要取得每日成本，請將「已使用」數量乘上**明細**區段中的「費率」值。

若要深入了解發票，請參閱[了解您的 Azure 發票](billing-understand-your-invoice.md)。

若要了解 CSV 檔案中的每一欄，請參閱[了解您的 Azure 詳細使用量](billing-understand-your-invoice.md)。

### <a name="option-2-review-your-invoice-and-compare-with-the-usage-and-costs-in-the-azure-portal"></a>選項 2：檢閱您的發票，並與 Azure 入口網站中的使用量和成本進行比較

Azure 入口網站也可協助您確認費用。Azure 入口網站提供成本管理圖表，可讓您取得使用量和發票費用的快速概觀。

若要從上面繼續進行範例，請造訪[訂用帳戶頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)，選取您的訂用帳戶，然後選擇 [成本分析]。 您可以從該處指定時間範圍，並查看 Azure 排程器服務的使用量費用。

![Azure 入口網站中的成本分析檢視](./media/billing-understand-your-bill/4.png)

若要在 [成本記錄] 中查看每日成本的明細，請按一下該資料列。

![Azure 入口網站中的成本記錄檢視](./media/billing-understand-your-bill/5.png)

若要進一步了解，請參閱[使用 Azure 計費與成本管理避免非預期的成本](billing-getting-started.md#costs)。

## <a name="external"></a>外部服務費用呢？
外部服務 (也稱為 Azure Marketplace 訂單) 是由獨立服務廠商提供，並會分開計費。 費用不會顯示在 Azure 發票上。 若要深入了解，請參閱[了解您的 Azure 外部服務費用](billing-understand-your-azure-marketplace-charges.md)。

## <a name="payment"></a>我要如何進行付款？

如果您設定信用卡或轉帳卡作為付款方式，我們會在計費期間結束後的 10 天內自動扣款。 在您的信用卡對帳單上，明細項目的名稱會是 **MSFT Azure**。

如果您透過[開立發票來付款](billing-how-to-pay-by-invoice.md)，請將您的款項匯到發票底部所列的位置。 如需詳細說明，請[連絡客戶支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

## <a name="how-do-i-check-the-status-of-a-payment-made-by-credit-card"></a>如何檢查信用卡付款的狀態？

[建立支援票證](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以要求查看付款的狀態。 

## <a name="are-there-different-azure-customer-types-how-do-i-know-what-customer-type-i-am"></a>是否有不同的 Azure 客戶類型？ 我如何知道自己是屬於哪一種客戶類型？
目前有不同類型的 Azure 客戶。 若要進一步了解您的定價和帳單，請參閱下列客戶類型描述。

- **企業**：企業客戶已經與 Azure 簽署一份 Enterprise 合約，以便進行承諾用量金額協商和存取 Azure 資源的自訂定價。
- **Web Direct**：Web Direct 客戶未與 Azure 簽署任何自訂合約。 這些客戶已透過 azure.com 註冊 Azure 服務並收到所有 Azure 資源的公開價格。
- **雲端服務提供者**：雲端服務提供者通常是一些受僱於終端客戶，在 Azure 之上建置解決方案的公司。

## <a name="why-dont-i-see-the-cost-the-resource-i-have-created-in-my-bill"></a>為什麼在我的帳單中，看不到我所建立資源的成本？
Azure 並不是直接根據資源成本而收費。 收費是以一個或多個計量為依據，而這些計量是用來追蹤資源整個存留期內的使用量。 接著，這些計量會用來計算帳單金額。 請參閱下列的 Azure 計量詳細資訊。

## <a name="how-does-azure-charge-metering-work"></a>Azure 如何根據計量來收費？
當您備妥單一 Azure 資源 (例如虛擬機器) 時，也會建立一或多個計量執行個體。 這些計量是用來追蹤一段時間內的資源使用量。 每個計量都會發出使用量記錄，而在我們的成本計量系統中，Azure 會利用這些記錄來計算費用。 

例如，在 Azure 中建立單一虛擬機器時，可能也會建立下列計量來追蹤虛擬機器的使用量：

- 計算時數
- IP 位址時數
- 資料傳入
- 資料傳出
- 標準受控磁碟
- 標準受控磁碟作業
- 標準 IO 磁碟
- 標準 IO - 區塊 Blob 讀取
- 標準 IO - 區塊 Blob 寫入
- 標準 IO - 區塊 Blob 刪除

建立 VM 之後，上述每一個計量會開始發出使用量記錄。 接著，Azure 的計量系統和計量的價格會使用這個使用量，來判斷應向客戶收取多少費用。

> [!Note]
> 上述範例計量可能只是 VM 建立時一併建立之計量器的一小部分。

## <a name="what-is-the-difference-between-azure-1st-party-charges-and-azure-marketplace-charges"></a>Azure 第一方費用和 Azure Marketplace 費用之間有何差別？
Azure 第一方費用適用於 Azure 直接開發和提供的資源。 Azure Marketplace 費用適用於第三方軟體廠商建立而且可以透過 Azure Marketplace 取得的資源。 例如，Barracuda Firewall 是協力廠商提供的 Azure Marketplace 資源。 適用於該防火牆和其相對應之計量的所有費用都會顯示為 Marketplace 費用。 

## <a name="tips-for-cost-management"></a>成本管理的秘訣
- 使用[定價計算機](https://azure.microsoft.com/pricing/calculator/)和來[估計成本](https://aka.ms/azure-tco-calculator)，並取得[每項服務的詳細定價資訊](https://azure.microsoft.com/pricing/)。
- [定期在 Azure 入口網站檢閱您的使用量和成本](billing-getting-started.md#costs)。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
