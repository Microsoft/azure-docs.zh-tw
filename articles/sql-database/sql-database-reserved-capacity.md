---
title: 預付 Azure SQL Database 虛擬核心的費用以節省成本 | Microsoft Docs
description: 了解如何購買 Azure SQL Database 保留容量，以節省計算費用。
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: e9edbc2a3bbc878f7a3cf99a3ca6efb4e69797ad
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394705"
---
# <a name="prepay-for-sql-database-compute-resources-with-azure-sql-database-reserved-capacity"></a>預付具有 Azure SQL Database 保留容量的 SQL Database 計算資源

在使用 Azure SQL Database 時，預付 Azure SQL Database 計算資源的費用，會比預付型方案的價格還要划算。 若使用 Azure SQL Database 保留容量，即表示您預先承諾會使用 SQL Database 一或三年的時間，來換取大幅的計算費用折扣。 若要購買 SQL Database 保留容量，您必須指定 Azure 區域、部署類型、效能層級與期間。 

您不需要對 SQL Database 執行個體指派保留容量。 只要符合已執行或新部署的 SQL Database 執行個體，就會自動獲得好處。 透過購買保留容量，您會預付為期一或三年的 SQL Database 執行個體計算費用。 購買保留容量後，符合保留屬性的 SQL Database 計算費用就不會再按照預付型方案的費率來收費。 保留容量並未涵蓋與 SQL Database 執行個體相關聯的軟體、網路或儲存體費用。 在保留期限結束時，計費好處會到期，隨後 SQL Database 便會按照預付型方案的費率來計費。 保留不會自動更新。 如需定價資訊，請參閱 [SQL Database 保留容量供應項目](https://azure.microsoft.com/pricing/details/sql-database/managed/)。

您可以在 [Azure 入口網站](https://portal.azure.com)購買 Azure SQL Database 保留容量。 若要購買 SQL Database 保留容量：
- 您必須至少為一個企業或預付型方案訂用帳戶的擁有者角色。
- 若為企業訂用帳戶，您必須在 [EA 入口網站](https://ea.azure.com)中啟用 Azure 保留容量購買。
-  針對雲端解決方案提供者 (CSP) 方案，則只有系統管理員代理人或銷售人員可以購買 SQL Database 保留容量。

有關如何向企業客戶和隨用隨付型方案客戶收取預訂費用的詳細資料，請參閱[了解 Enterprise 註冊的 Azure 保留使用量](../billing/billing-understand-reserved-instance-usage-ea.md)與[了解隨用隨付型方案訂用帳戶的 Azure 保留使用量](../billing/billing-understand-reserved-instance-usage.md)。

## <a name="determine-the-right-sql-size-before-purchase"></a>先決定正確的 SQL 大小再購買

需要多少保留容量大小，應根據特定區域內現有或即將部署的 SQL 單一資料庫和/或彈性集區所使用的總計算量，並使用相同效能層級和硬體世代來決定。 

例如，假設您要執行一個一般用途 Gen5 – 16 個虛擬核心的彈性集區，以及兩個商務關鍵性 Gen5 – 4 個虛擬核心的單一資料庫。 此外，假設您打算在下個月再部署一個一般用途 Gen5 – 16 個虛擬核心的彈性集區，以及一個商務關鍵性 Gen5 – 32 個虛擬核心的彈性集區。 還有，假設您知道您需要使用這些資源至少 1 年的時間。 在此情況下，您應該購買 32 (2x16) 個虛擬核心、為期 1 年的 SQL Database 單一/彈性集區一般用途 - Compute Gen5 保留容量，以及 40 (2x4 + 32) 個虛擬核心、為期 1 年的 SQL Database 單一/彈性集區商務關鍵性 - Compute Gen5 保留容量。

## <a name="buy-sql-database-reserved-capacity"></a>購買 SQL Database 保留容量

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務] > [保留]。
3. 選取 [新增]，然後在 [選取產品類型] 窗格中，選取 [SQL 資料庫] 以購買新的 SQL Database 保留容量。
4. 填寫必要欄位。 符合所選屬性的現有或全新單一資料庫或彈性集區，就有資格獲得保留容量折扣。 取得折扣的 SQL Database 執行個體實際數目取決於選取的範圍和數量。

   ![提交 SQL Database 保留容量購買前的螢幕擷取畫面](./media/sql-database-reserved-vcores/sql-reserved-vcores-purchase.png)

    | 欄位      | 說明|
    |:------------|:--------------|
    |名稱        |此保留項目的名稱。| 
    |訂用帳戶|用來支付 SQL Database 保留容量費用的訂用帳戶。 SQL Database 保留容量的預付費用會透過訂用帳戶的付款方式收取。 訂用帳戶類型必須是 Enterprise 合約 (供應項目號碼：MS-AZR-0017P) 或預付型方案 (供應項目號碼：MS-AZR-0003P)。 針對企業訂用帳戶，費用會從註冊的承諾用量金額餘額扣除或作為超額部分收費。 針對預付型方案訂用帳戶，費用是透過訂用帳戶的信用卡或發票付款方式收取。|    
    |影響範圍       |虛擬核心保留容量範圍可以涵蓋一個訂用帳戶或多個訂用帳戶 (共用範圍)。 如果您選取： <ul><li>單一訂用帳戶 - 虛擬核心保留容量折扣會套用至此訂用帳戶中的 SQL Database 執行個體。 </li><li>共用 - 虛擬核心保留容量折扣會套用至計費內容內任何訂用帳戶中執行的 SQL Database 執行個體。 針對企業客戶，共用範圍是註冊，並包含註冊中的所有訂用帳戶 (開發/測試訂用帳戶除外)。 針對預付型方案客戶，共用範圍是帳戶系統管理員所建立的所有預付型方案訂用帳戶。</li></ul>|
    |區域      |SQL Database 保留容量所涵蓋的 Azure 區域。|    
    |部署類型|您要為其購買保留容量的 SQL 資源類型。|
    |效能層級|SQL Database 執行個體的服務層。
    |期間        |一年或三年。|
    |數量    |SQL Database 保留容量內所購買的執行個體數目。 數量是可以取得帳單折扣的執行中 SQL Database 執行個體數目。 例如，如果您在美國東部執行 10 個 SQL Database 執行個體，那麼您會指定數量為 10，以最大化所有執行中機器的效益。 |

5. 檢閱 [成本] 區段中 SQL Database 保留容量的成本。
6. 選取 [購買]。
7. 選取 [檢視此保留] 以查看您的購買狀態。

## <a name="cancellations-and-exchanges"></a>取消和交換

如果您需要取消保留 SQL Database 保留容量，可能會有 12% 的提前解約金。 退款以您的購買價格或目前保留的價格為準，視孰者為低。 每年的退款金額上限為 50,000 美元。 您收到的退款是按比例計算的餘額扣除 12% 提前解約金的金額。 若要要求取消，請移至 Azure 入口網站中的保留區，並選取 [退款] 以建立支援要求。

如果您需要將 [SQL Database 保留容量] 保留區變更為另一個區域、部署類型、效能層級或期間，您可以將它與等值或更高價值的另一個保留區交換。 新保留區的期間開始日期不會延續自交換的保留區。 1 或 3 年的期間會從您建立新的保留區時起算。 若要要求交換，請移至 Azure 入口網站中的保留區，並選取 [交換] 以建立支援要求。

## <a name="vcore-size-flexibility"></a>vCore 大小彈性

vCore 大小彈性可協助您在效能層級和區域內相應增加或相應減少，而不會失去保留容量優勢。 SQL Database 保留容量為您提供彈性地在集區與單一資料庫之間暫時移動最忙碌的資料庫，作為一般作業 (在相同的區域和效能層級) 的一部分，而不會失去保留容量優勢。 透過在保留區中保留未套用的緩衝區，您可以在不超出您預算的情況下有效地管理效能高峰。

## <a name="next-steps"></a>後續步驟

虛擬核心保留容量折扣會自動套用至符合 SQL Database 保留容量範圍和屬性的 SQL Database 執行個體數目。 您可以透過 [Azure 入口網站](https://portal.azure.com)、PowerShell、CLI 或 API 來更新 SQL Database 保留容量的範圍。 

若要了解如何管理 SQL Database 保留容量，請參閱[管理 SQL Database 保留容量](../billing/billing-manage-reserved-vm-instance.md)。

若要深入了解 Azure 保留項目，請參閱下列文章：

- [什麼是 Azure 保留項目？](../billing/billing-save-compute-costs-reservations.md)
- [管理 Azure 保留項目](../billing/billing-manage-reserved-vm-instance.md)
- [了解 Azure 保留折扣](../billing/billing-understand-reservation-charges.md)
- [了解隨用隨付方案訂用帳戶的保留項目使用量](../billing/billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](../billing/billing-understand-reserved-instance-usage-ea.md)
- [合作夥伴中心雲端解決方案提供者 (CSP) 計畫中的 Azure 保留項目](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。

