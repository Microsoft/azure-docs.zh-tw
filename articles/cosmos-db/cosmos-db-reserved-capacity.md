---
title: 預付 Azure Cosmos DB 資源費用以節省成本 | Microsoft Docs
description: 了解如何購買 Azure Cosmos DB 保留容量，以節省計算費用。
services: cosmos-db
author: rimman
manager: kfile
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: rimman
ms.reviewer: sngun
ms.openlocfilehash: 1be2d67d8a1ee51c4883ae1f50b80ad3a9691c2d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981962"
---
# <a name="prepay-for-azure-cosmos-db-resources-with-reserved-capacity"></a>以保留容量預付 Azure Cosmos DB 資源費用

Azure Cosmos DB 保留容量有助於您透過預付一年或三年期的 Azure Cosmos DB 資源來節省成本。 Azure Cosmos DB 保留容量可讓您在針對 Cosmos DB 資源佈建的輸送量上享有折扣，例如資料庫、容器 (資料表/集合/圖表)。 Azure Cosmos DB 保留容量可以大幅降低 Cosmos DB 成本，預付一或三年期承諾用量費用，即可節省高達一般價格的 65%。 保留容量提供帳單折扣，而且不會影響 Cosmos DB 資源的執行階段狀態。

Azure Cosmos DB 保留容量涵蓋針對資源佈建的輸送量，但不涵蓋儲存體和網路費用。 購買保留容量後，符合保留屬性的輸送量費用就不會再按照預付型方案的費率來收費。 如需有關保留的詳細資訊，請參閱 [Azure 保留](../billing/billing-save-compute-costs-reservations.md)一文。 

您可以從 [Azure 入口網站](https://portal.azure.com)購買 Azure Cosmos DB 保留容量。 若要購買保留容量：

* 您必須至少為一個企業或預付型方案訂用帳戶的擁有者角色。  
* 若為企業訂用帳戶，您必須在 [EA 入口網站](https://ea.azure.com/)中啟用 Azure 保留容量購買。  
* 若為雲端解決方案提供者 (CSP) 方案，則只有系統管理員代理人或銷售人員可以購買 Azure Cosmos DB 保留容量。

## <a name="determine-the-required-throughput-before-purchase"></a>購買之前，請決定所需的輸送量

保留大小應該以現有或即將部署的 Azure Cosmos DB 資源 (例如資料庫或容器 - 集合、資料表、圖表) 所使用的輸送量總量為基礎。 您可以透過下列方式來判斷所需的輸送量：

* 瀏覽至 [Azure 入口網站](https://portal.azure.com)、尋找您的 Azure Cosmos DB 帳戶、開啟 [計量] 刀鋒視窗，然後從 [輸送量] 索引標籤取得 3 至 6 個月期間的每秒平均輸送量詳細資料。 購買時，請提供此大小作為保留容量單位。

或者，如果您是 Enterprise 合約 (EA) 客戶，可以下載使用方式檔案，並參考使用方式檔案 [其他資訊] 區段中的 [服務類型] 值，以取得 Azure Cosmos DB 輸送量詳細資料。

您也可以針對預期在接下來一年或三年期間執行的 Azure Cosmos DB 帳戶，加總所有工作負載的平均輸送量，並使用該數量作為保留容量。

## <a name="buy-azure-cosmos-db-reserved-capacity"></a>購買 Azure Cosmos DB 保留容量

1. 登入 [Azure 入口網站](https://portal.azure.com)。  

2. 選取 [所有服務] > [保留] > [新增]。  

3. 從 [選取產品類型] 窗格中，選擇 [Azure Cosmos DB]，然後**選擇**購買新的保留容量。  

4. 請填寫必填欄位，如下表中所述：

   ![填寫保留容量表單](./media/cosmos-db-reserved-capacity/fill_reserved_capacity_form.png) 

   |欄位  |說明  |
   |---------|---------|
   |名稱   |    保留容量的名稱。 此欄位會自動填入 `CosmosDB_Reservation_<timeStamp>`。 您可以在建立保留容量時提供不同的名稱，或在建立保留容量之後重新命名。      |
   |訂用帳戶  |   用來支付 Azure Cosmos DB 保留容量費用的訂用帳戶。 收取預付費用時，會使用所選訂用帳戶的付款方式。 訂用帳戶類型必須是下列其中一項： <br/><br/>  [Enterprise 合約](https://azure.microsoft.com/pricing/enterprise-agreement/) (供應項目號碼：MS-AZR-0017P) - 針對企業訂用帳戶，費用會從註冊的承諾用量金額餘額扣除或當作超額部分收費。 <br/><br/> [預付型方案](https://azure.microsoft.com/offers/ms-azr-0003p/) (供應項目號碼：MS-AZR-0003P)- 針對預付型方案訂用帳戶，費用是透過訂用帳戶的信用卡或發票付款方式收取。    |
   |影響範圍   |   保留的範圍會控制多少訂用帳戶可享有與保留相關的計費權益，以及控制保留套用至特定訂用帳戶的方式。 保留的範圍為單一或共用的訂用帳戶。 如果您選取：   <br/><br/>  **單一訂用帳戶** - 保留容量折扣會套用至所選訂用帳戶中的 Azure Cosmos DB 執行個體。 <br/><br/>  **共用** – 保留容量折扣會套用至計費內容內任何訂用帳戶中執行的 Azure Cosmos DB 執行個體。 計費內容取決於您註冊 Azure 的方式。 針對企業客戶，共用範圍是註冊，並包含註冊中的所有訂用帳戶 (開發/測試訂用帳戶除外)。 針對預付型方案客戶，共用範圍是帳戶系統管理員所建立的所有預付型方案訂用帳戶。  <br/><br/> 您可以在購買保留容量之後變更保留範圍。  |
   |保留容量類型   |  保留容量類型是針對要求單位所佈建的輸送量。|
   |保留容量單位  |      您想要保留的輸送量數量。 您可以透過判斷每個區域的所有 Cosmos DB 資源 (例如資料庫或容器) 所需的輸送量計算這個值，然後乘以將與您 Cosmos DB 資料庫相關聯的區域數目。  <br/> 例如：如果您有五個區域，其中每個區域每秒為 1 百萬個要求單位，則購買保留容量時，請選擇每秒 5 百萬個要求單位。    |
   |詞彙  |   一年或三年。   |

5. 在 [成本] 區段中，檢閱保留容量的折扣和價格。 此保留容量價格適用於其輸送量針對所有區域而佈建的 Azure Cosmos DB 資源。  

6. 選取 [購買]。 購買成功時，您可以看到下列螢幕擷取畫面。 

   ![填寫保留容量表單](./media/cosmos-db-reserved-capacity/reserved_capacity_successful.png) 

購買保留容量之後，將會立即套用至符合保留容量條件的任何現有 Azure Cosmos DB 資源。 如果您沒有任何現有的 Azure Cosmos DB 資源，保留容量將會在您部署符合保留容量條件的新 Cosmos DB 執行個體時套用。 在這兩種情況下，保留容量的期間都會在成功購買後立即生效。 

當您的保留容量到期時，Azure Cosmos DB 執行個體將會繼續執行，而且將會以一般預付型方案費率計費。

## <a name="next-steps"></a>後續步驟

保留容量折扣會自動套用到符合保留容量範圍和屬性的 Azure Cosmos DB 資源中。 您可以透過 Azure 入口網站、PowerShell、CLI 或 API 來更新保留容量的範圍。

*  若要了解如何將保留容量折扣套用至 Azure Cosmos DB，請參閱[了解 Azure 保留折扣](../billing/billing-understand-cosmosdb-reservation-charges.md)

* 若要深入了解 Azure 保留項目，請參閱下列文章：

   * [什麼是 Azure 保留項目？](../billing/billing-save-compute-costs-reservations.md)  
   * [管理 Azure 保留](../billing/billing-manage-reserved-vm-instance.md)。  
   * [了解 Enterprise 註冊的保留項目使用量](../billing/billing-understand-reserved-instance-usage-ea.md)  
   * [了解隨用隨付方案訂用帳戶的保留項目使用量](../billing/billing-understand-reserved-instance-usage.md)
   * [合作夥伴中心雲端解決方案提供者 (CSP) 計畫中的 Azure 保留項目](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。

