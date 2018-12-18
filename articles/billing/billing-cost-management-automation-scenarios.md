---
title: Azure 計費和成本管理自動化案例 | Microsoft Docs
description: 了解常見的計費和成本管理案例如何對應至不同的 API。
services: billing
documentationcenter: ''
author: Erikre
manager: dougeby
editor: ''
tags: billing
ms.assetid: 204b15b2-6667-4b6c-8ea4-f32c06f287fd
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 6/13/2018
ms.author: erikre
ms.openlocfilehash: 97f6339e5c9532250245a97a81dade12db139e98
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227362"
---
# <a name="billing-and-cost-management-automation-scenarios"></a>計費和成本管理自動化案例

以下指出計費和成本管理空間的常見案例，並對應至可用於這些案例的不同 API。 您可以在 API 對應案例底下，找到所有可用 API 及其所提供功能的摘要。 

## <a name="common-scenarios"></a>常見案例

您可以在各種案例中，使用計費和成本管理 API 來回應成本和使用量相關問題。  以下概要說明常見案例。

- **發票對帳** - Microsoft 向我收取的金額是否正確？  我的帳單內容為何，我是否可以自行計算？

- **交叉收費** - 現在我知道要支付的費用，我的組織中誰需要支付這些費用？

- **成本最佳化** - 我知道已支付的費用，不過如何才能充分利用花費在 Azure 上的每分每毫？

- **成本追蹤** - 我想要查看我在一段時間內使用 Azure 所花費的成本。 趨勢為何？ 如何改進？

- **Azure 當月支出** - 我在本月迄今的支出為何？ 我是否需要對 Azure 費用及/或使用量進行任何調整？ 我的 Azure 使用量在當月何時最高？

- **設定警示** - 我想要設定資源型使用量或貨幣型警示。

## <a name="scenario-to-api-mappings"></a>API 對應案例

|         API/案例        | 發票對帳    | 交叉收費    | 成本最佳化    | 成本追蹤    | 月中支出    | 警示    |
|:---------------------------:|:-------------------------:|:----------------:|:--------------------:|:----------------:|:------------------:|:---------:|
| 預算                     |                           |                  |           X          |                  |                    |     X     |
| Marketplace                |             X             |         X        |           X          |         X        |          X         |     X     |
| 價位表                 |             X             |         X        |           X          |         X        |          X         |           |
| 保留建議 |                           |                  |           X          |                  |                    |           |
| 保留詳細資料         |                           |                  |           X          |         X        |                    |           |
| 保留摘要       |                           |                  |           X          |         X        |                    |           |
| 使用量詳細資料               |             X             |         X        |           X          |         X        |          X         |     X     |
| 計費期間             |             X             |         X        |           X          |         X        |                    |           |
| 發票                    |             X             |         X        |           X          |         X        |                    |           |
| RateCard                    |             X             |                  |           X          |         X        |          X         |           |
| 未分級的使用量               |             X             |                  |           X          |                  |          X         |           |

> [!NOTE]
> 下列 API 對應案例不包含企業使用量 API。 請盡可能利用一般使用量 API 來解決接下來的網路新開發案例。

## <a name="api-summaries"></a>API 摘要

### <a name="consumption"></a>耗用量
(Web Direct + 企業客戶適用於除了以下所列之 API 以外的所有 API)

-   **預算** (「僅限企業客戶」)：使用[預算 API](https://docs.microsoft.com/rest/api/consumption/budgets) 來建立資源、資源群組或計費計量的成本或使用量預算。  建立預算之後，即可設定警示，在超過使用者定義的預算閾值時通知。 也可以設定到達預算金額時要發生的動作。
-   **Marketplace**：使用 [Marketplace 費用 API](https://docs.microsoft.com/rest/api/consumption/marketplaces) 來取得所有 Marketplace 資源 (Azure 協力廠商供應項目) 的費用和使用量資料。 此資料可用來加總所有 Marketplace 資源的成本，或調查特定資源的成本/使用量。
-   **價位表** (僅限企業客戶)：企業客戶可以使用[價位表 API](https://docs.microsoft.com/rest/api/consumption/pricesheet) 來擷取所有計量的自訂定價。 企業可以使用此資料搭配使用量詳細資料和 Marketplace 使用量資訊，透過使用量和 Marketplace 資料執行成本計算。 
-   **保留建議**：使用[保留建議 API](https://docs.microsoft.com/rest/api/consumption/reservationrecommendations) 來取得購買已保留 VM 執行個體的建議。 建議的設計目的是為了讓客戶分析預期節省的成本和購買數量。
-   **保留詳細資料**：使用[保留詳細資料 API](https://docs.microsoft.com/rest/api/consumption/reservationsdetails) 來查看先前所購買 VM 保留的相關資訊，例如已保留使用量與實際使用量的比較。 您可以查看每個 VM 層級的詳細資料。
-   **保留摘要**：使用[保留摘要 API](https://docs.microsoft.com/rest/api/consumption/reservationssummaries) 來查看先前所購買 VM 保留的相關彙總資訊，例如已保留使用量與實際使用量的彙總比較。 
-   **使用量詳細資料**：使用[使用量詳細資料 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 來取得所有 Azure 第一方資源的費用和使用量。 此資訊採用使用量詳細資料記錄格式，目前每天會針對每個計量發出一次。 此資訊可用來加總所有資源的成本，或調查特定資源的成本/使用量。
-   **RateCard**：Web Direct 客戶可以使用 [RateCard API](https://msdn.microsoft.com/library/azure/mt219005.aspx) 來取得其計量費率。 然後，他們可以使用傳回的資訊搭配其資源使用量資訊來手動計算預期的帳單。 
-   **未分級的使用量**：您可以使用[未分級的使用量 API](https://msdn.microsoft.com/library/azure/mt219003.aspx) 來取得 Azure 完成任何計量/收費之前的原始使用量資訊。

### <a name="billing"></a>計費
-   **計費期間**：使用[計費期間 API](https://docs.microsoft.com/rest/api/billing/billingperiods) 來決定要分析的計費期間，以及該期間的發票識別碼。 發票識別碼可搭配下列發票 API 使用。 
-   **發票**：使用[發票 API](https://docs.microsoft.com/rest/api/billing/invoices) 來取得指定計費期間之 PDF 格式發票的下載 URL。

### <a name="enterprise-consumption"></a>企業使用量
(僅限所有企業 API)

-   **餘額摘要**：使用[餘額摘要 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) 來取得餘額、新購買項目、Azure Marketplace 服務費用、調整，以及超額部分費用的每月摘要資訊。 您可以取得目前計費期間或任何過去期間的此資訊。 企業可以使用此資料來執行與手動計算之摘要費用的比較。 此 API 不提供資源特定資訊或成本的彙總檢視。
-   **使用量詳細資料**：使用[使用量詳細資料 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail) 來取得目前月份、特定計費期間或自訂日期期間的第一方 Azure 使用量詳細資訊。 企業可以使用此資料根據費率和使用量來手動計算帳單，也可以使用存在的部門/組織資訊來歸類組織內的成本。 此資料提供使用量/成本的資源特定檢視。
-   **Marketplace 網上商店費用**：使用 [Marketplace 網上商店費用 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) 來取得目前月份、特定計費期間或自訂日期期間的協力廠商 Azure 使用量詳細資訊。 企業可以使用此資料根據費率和使用量來手動計算帳單，也可以使用存在的部門/組織資訊來歸類組織內的成本。 Marketplace 商店費用 API 提供使用量/成本的資源特定檢視。
-   **價位表**：[價位表 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) 可針對指定註冊和計費期間的每個計量提供適用的費率。 您可以使用此費率資訊搭配使用量詳細資料和 Marketplace 使用量資訊來手動計算預期的帳單。
-   **計費期間**：使用[計費期間 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) 來取得計費期間清單以及一個屬性，而該屬性指向該計費期間內四組企業 API 資料的 API 路由 - BalanceSummary、UsageDetails、Marketplace Charges 和 PriceSheet。
-   **Azure 保留建議**：[保留的執行個體建議 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation) 會查看客戶的 7 天、30 天或 60 天虛擬機器使用量，並提供單一和共用購買建議。 保留的執行個體 API 可讓客戶分析預期節省的成本和建議的購買數量。

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="what-is-the-difference-between-the-enterprise-reporting-apis-and-the-consumption-apis-when-should-i-use-each"></a>企業報告 API 與使用量 API 之間有何差異？ 各自的使用時機為何？
這些 API 有一組類似的功能，並且可以回答同一組計費與成本管理空間相關問題。 不過，每個 API 的目標對象不同： 

- **企業報告 API**：這類 API 適用於已經與 Microsoft 簽署 Enterprise 合約的客戶，該合約會授與他們存取協商承諾用量和自訂定價的權限。 這類 API 要求使用可透過[企業版入口網站](https://ea.azure.com)取得的金鑰。 如需這些 API 的說明，請參閱[適用於企業客戶的報告 API 概觀](billing-enterprise-api.md)。

- **使用量 API**：這類 API 適用於所有的客戶，但有一些例外狀況。 如需詳細資訊，請參閱 [Azure 使用量 API 概觀](billing-consumption-api-overview.md)和[ Azure 使用量 API 參考](https://docs.microsoft.com/rest/api/consumption/)。 所提供的 API 是最新開發案例的建議解決方案。 

### <a name="what-is-the-difference-between-the-usage-details-api-and-the-usage-api"></a>使用量詳細資料 API 與使用量 API 之間有何差異？
這些 API 基本上提供不同的資料：

- **使用量詳細資料**：[使用量詳細資料 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 可提供每個計量器執行個體的 Azure 使用量和成本資訊。 提供的資料已通過 Azure 的成本計量系統，而且已隨著其他可能的變更套用成本：

    - 因為使用預付承諾用量而造成的帳戶變更
    - 因為 Azure 發現使用量差異而造成的帳戶變更

- **使用量**：[使用量 API](https://msdn.microsoft.com/library/Mt219003.aspx) 會在未經處理的 Azure 使用量資訊通過 Azure 的成本計量系統之前提供該資訊。 此資料可能與在 Azure 費用計量系統之後看到的使用量和/或費用金額沒有任何相互關聯。

### <a name="what-is-the-difference-between-the-invoice-api-and-the-usage-details-api"></a>發票 API 與使用量詳細資料 API 之間有何差異？
這些 API 可提供相同資料的不同檢視。 [發票 API](https://docs.microsoft.com/rest/api/billing/invoices) 僅適用於 Web Direct 客戶，並根據每個計量類型的彙總費用來提供每月帳單彙總。 [使用量詳細資料 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 可提供每天使用量/成本記錄的細微檢視，並可供企業和 Web Direct 客戶使用。

### <a name="what-is-the-difference-between-the-price-sheet-api-and-the-ratecard-api"></a>價位表 API 與 RateCard API 之間有何差異？
這些 API 提供類似的資料集，但有不同的對象。 相關資訊如下：

- 價位表 API：[價位表 API](https://docs.microsoft.com/rest/api/consumption/pricesheet) 可提供已針對企業客戶協商的自訂定價。
- RateCard API：[RateCard API](https://msdn.microsoft.com/library/mt219005.aspx) 可提供適用於 Web Direct 客戶的公開定價。

## <a name="next-steps"></a>後續步驟

- 如需使用 Azure API 以程式設計方式取得 Azure 使用量見解的資訊，請參閱 [Azure 使用量 API 概觀](billing-consumption-api-overview.md)和 [Azure 計費 API 概觀](billing-usage-rate-card-overview.md)。
- 若要比較包含每日詳細使用量檔案的發票與 Azure 入口網站中的成本管理報告，請參閱[了解 Microsoft Azure 帳單](billing-understand-your-bill.md)。
- 如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
