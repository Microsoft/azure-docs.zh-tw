---
title: Cloud Cruiser 和 Microsoft Azure 計費 API 整合 | Microsoft Docs
description: 提供 Microsoft Azure 計費合作夥伴 Cloud Cruiser 將 Azure 計費 API 整合至其產品的經驗所得來的獨特觀點。  這特別適用於有興趣使用/嘗試將 Cloud Cruiser 用於 Microsoft Azure Pack 的客戶。
services: ''
documentationcenter: ''
author: tonguyen
manager: tonguyen
editor: ''
tags: billing
ms.assetid: b65128cf-5d4d-4cbd-b81e-d3dceab44271
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 10/09/2017
ms.author: mobandyo
ms.openlocfilehash: 487636ffb0efc35c282e14d835c6669ed9d47315
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32771492"
---
# <a name="cloud-cruiser-and-microsoft-azure-billing-api-integration"></a>Cloud Cruiser 和 Microsoft Azure 計費 API 整合
本文描述從新的 Microsoft Azure 計費 API 所收集來的資訊如何用來在 Cloud Cruiser 中進行工作流程成本模擬與分析。

## <a name="azure-ratecard-api"></a>Azure RateCard API
RateCard API 提供來自 Azure 的費率資訊。 以適當的認證進行驗證之後，您可以查詢 API 以收集 Azure 上可用服務的中繼資料，以及與優惠識別碼相關聯的費率。

以下是來自 API 的範例回應，顯示 A0 (Windows) 執行個體的價格：

    {
        "MeterId": "0e59ad56-03e5-4c3d-90d4-6670874d7e29",
        "MeterName": "Compute Hours",
        "MeterCategory": "Virtual Machines",
        "MeterSubCategory": "A0 VM (Windows)",
        "Unit": "Hours",
        "MeterRates":
        {
            "0": 0.029
        },
        "EffectiveDate": "2014-08-01T00:00:00Z",
        "IncludedQuantity": 0.0,
        "MeterStatus": "Active"
    },

### <a name="cloud-cruisers-interface-to-azure-ratecard-api"></a>Azure RateCard API 的 Cloud Cruiser 介面
Cloud Cruiser 可以用不同的方式使用 RateCard API 資訊。 在這篇文章中，我們會說明如何使用它進行 IaaS 工作負載成本模擬及分析。

為了示範這個使用案例，請想像執行於 Microsoft Azure Pack (WAP) 之數個執行個體的工作負載。 目標是要在 Azure 上模擬相同的工作負載，並評估這類移轉的成本。 若要建立這個模擬，有兩個主要的工作要執行：

1. **匯入和處理從 RateCard API 收集的服務資訊。** 這項工作也會在活頁簿上執行，其中從 RateCard API 擷取的內容會轉換並發佈為新的費率方案。 新的費率方案會在模擬中用來評估 Azure 價格。
2. **標準化 WAP 服務和 IaaS 的 Azure 服務。** 根據預設，WAP 服務以個別資源 (CPU、記憶體大小、磁碟大小等) 為基礎，而 Azure 服務以執行個體大小 (A0、A1、A2 等等) 為基礎。 第一個工作可以由 Cloud Cruiser 的 ETL 引擎執行，稱為活頁簿，其中資源整合為執行個體大小，類似 Azure 執行個體服務。

### <a name="import-data-from-the-ratecard-api"></a>從 RateCard API 匯入資料
Cloud Cruiser 活頁簿提供自動化的方式收集和處理來自 RateCard API 的資訊。  ETL (擷取-轉換-載入) 活頁簿可讓您設定資料的集合、轉換和發佈至 Cloud Cruiser 資料庫。

每個活頁簿可以有一個或多個集合，讓您將來自不同資源的資訊相互關聯，以補充或強化使用狀況資料。 以下兩個螢幕擷取畫面顯示如何在現有活頁簿中建立新的「集合」，並將資訊從 RateCard API 匯入到「集合」：

![圖 1 - 建立新的集合][1]

![圖 2 - 從新集合匯入資料][2]

將資料匯入活頁簿之後，就可以建立多個步驟及轉換程序，以修改資料與建立資料模型。 以這個範例而言，因為我們只對基礎結構即為服務 (IaaS) 感興趣，所以我們可以使用轉換步驟移除不必要的資料列或記錄，它們和服務相關，而不是 IaaS。

以下螢幕擷取畫面顯示轉換步驟，用來處理從 RateCard API 所收集的資料：

![圖 3 - 可處理收集自 RateCard API 之資料的轉換步驟][3]

### <a name="defining-new-services-and-rate-plans"></a>定義新的服務和費率方案
有不同的方式可定義 Cloud Cruiser 上的服務。 其中一個選項是從使用情況資料匯入服務。 使用公用雲端，其中的服務已由提供者定義時，通常會使用這個方法。

費率方案是一組費率或價格，可根據生效日期、客戶群組或其他選項套用至不同服務。 費率方案也可以在 Cloud Cruiser 上用來建立模擬或「假設」案例，以了解服務中的變更如何影響工作負載的總成本。

在此範例中，我們會使用來自 RateCard API 的服務資訊定義 Cloud Cruiser 中的新服務。 同樣地，我們可以使用和服務相關聯的費率，在 Cloud Cruiser 上建立新的費率方案。

在轉換程序結束時，就可以建立新的步驟，並將來自 RateCard API 的資料發佈做為新的服務和費率。

![圖 4- 將來自 RateCard API 的資料發佈為新的服務和費率][4]

### <a name="verify-azure-services-and-rates"></a>確認 Azure 服務和費率
發佈服務及費率之後，您可以在 Cloud Cruiser 的 [ *服務* ] 索引標籤中確認匯入服務的清單：

![圖 5- 驗證新的服務][5]

在 [ *費率方案* ] 索引標籤上，您可以利用匯入自 RateCard API 的費率檢查名為 "AzureSimulation" 的費率方案。

![圖 6- 驗證新的費率方案及相關費率][6]

### <a name="normalize-wap-and-azure-services"></a>標準化 WAP 和 Azure 服務
根據預設，WAP 會根據計算、記憶體和網路資源等使用情況提供使用情況資訊。 在 Cloud Cruiser 中，您可以直接根據這些資源的配置或計量使用情況來定義服務。 例如，您可以設定每小時 CPU 使用量的基本費率，或為配置給執行個體的 GB 記憶體收費。

以這個範例而言，為了比較 WAP 和 Azure 之間的成本，我們必須將 WAP 上的資源使用情況彙總套組，進而將其對應至 Azure 服務。 此轉換可以在活頁簿中輕鬆地實作：

![圖 7 - 轉換 WAP 資料以將服務標準化][7]

活頁簿的最後一個步驟是將資料發佈至 Cloud Cruiser 資料庫。 在此步驟期間，使用情況資料現在整合為服務 (進而對應至 Azure 服務)，並繫結至預設的費率來建立費用。

完成活頁簿之後，您可以在排程器上加入工作，並指定活頁簿執行的頻率和時間，即可自動化資料的處理。

![圖 8 - 活頁簿排程][8]

### <a name="create-reports-for-workload-cost-simulation-analysis"></a>建立工作負載成本模擬分析的報告
收集使用情況並將費用載入 Cloud Cruiser 資料庫之後，我們就可以使用 Cloud Cruiser Insights 模組來建立我們想要的工作負載成本模擬。

為了示範這種案例，我們建立了下列報告：

![成本比較][9]

上圖顯示以服務為基準的成本比較，其比較在 WAP (深藍色) 與 Azure (淺藍色) 之間為每個特定服務執行工作負載的價格。

下圖顯示依照部門細分的相同資料。 該圖顯示每個部門在 WAP 和 Azure 上執行其工作負載的成本，以及以節約列 (綠色) 顯示兩者之間的成本差異。

## <a name="azure-usage-api"></a>Azure 使用情況 API
### <a name="introduction"></a>簡介
Microsoft 最近推出 Azure Usage API，允許訂閱者以程式設計方式提取使用情況資料來洞悉其耗用量。 Cloud Cruiser 客戶可以透過此 API 利用更豐富的資料集。

Cloud Cruiser 可以數種方式使用和 Usage API 的整合。 可透過 API 使用的資料粒度 (每小時使用量資訊) 和資源中繼資料資訊可提供必要的資料集來支援彈性的回報或退款模型。 

在本教學課程中，我們會提供 Cloud Cruiser 如何使用 Usage API 資訊獲得好處的一個範例。 更具體來說，我們將在 Azure 上建立資源群組、將帳戶結構的標記相關聯，然後描述提取和處理標記資訊到 Cloud Cruiser 的程序。

最終的目標是要能夠建立和下方報告類似的報告，而且能夠根據標記填入的帳戶結構分析成本和耗用量。

![圖 10 - 含有使用標記之細項的報告][10]

### <a name="microsoft-azure-tags"></a>Microsoft Azure 標記
可透過 Azure Usage API 使用的資料不僅包括耗用量資訊，還包括內含於其相關聯之任何標記的資源中繼資料。 標記可提供簡單的方式組織您的資源，但是為了有效率，您必須確認：

* 標記在佈建階段正確套用至資源
* 標記正確用於回報/退款程序，以將使用情況繫結至組織的帳戶結構。

兩個需求都頗具挑戰性，特別是當佈建或收費端上有手動程序時。 輸入錯誤、不正確或甚至是遺漏標記，都是客戶使用標籤時的常見抱怨，這些錯誤可能會使收費端的工作非常困難。

使用新的 Azure Usage API，Cloud Cruiser 可以提取資源標記資訊，並透過複雜的 ETL 工具 (稱為活頁簿) 修正這些常見的標記錯誤。 透過運用規則運算式和資料相互關聯的轉換作業，Cloud Cruiser 能夠識別不正確標記的資源並套用正確的標記，確保資源和取用者之間有正確的關聯。

在收費端，Cloud Cruiser 可自動化回報/退款程序，並且可以使用標記資訊將使用情況繫結至適當的取用者 (部門、事業處、專案等)。 這項自動化作業提供非常重大的改進，並且可以確保一致且可稽核的收費程序。

### <a name="creating-a-resource-group-with-tags-on-microsoft-azure"></a>使用 Microsoft Azure 上的標記建立資源群組
本教學課程的第一個步驟是在 Azure 入口網站上建立資源群組，然後建立新的標記以產生和資源之間的關聯。 為了這個範例，我們會建立下列標記：部門、環境、擁有者、專案。

以下螢幕擷取畫面顯示具有相關聯標記的範例資源群組。

![圖 11 - 在 Azure 入口網站上具有相關標記的資源群組][11]

下一步是將資訊從 Usage API 提取到 Cloud Cruiser。 Usage API 目前提供 JSON 格式的回應。 擷取的資料範例如下：

    {
      "id": "/subscriptions/bb678b04-0e48-4b44-XXXX-XXXXXXX/providers/Microsoft.Commerce/UsageAggregates/Daily_BRSDT_20150623_0000",
      "name": "Daily_BRSDT_20150623_0000",
      "type": "Microsoft.Commerce/UsageAggregate",
      "properties":
      {
        "subscriptionId": "bb678b04-0e48-4b44-XXXX-XXXXXXXXX",
        "usageStartTime": "2015-06-22T00:00:00+00:00",
        "usageEndTime": "2015-06-23T00:00:00+00:00",
        "meterName": "Compute Hours",
        "meterRegion": "",
        "meterCategory": "Virtual Machines",
        "meterSubCategory": "Standard_D1 VM (Non-Windows)",
        "unit": "Hours",
        "instanceData": "{\"Microsoft.Resources\":{\"resourceUri\":\"/subscriptions/bb678b04-0e48-4b44-XXXX-XXXXXXXX/resourceGroups/DEMOUSAGEAPI/providers/Microsoft.Compute/virtualMachines/MyDockerVM\",\"location\":\"eastus\",\"tags\":{\"Department\":\"Sales\",\"Project\":\"Demo Usage API\",\"Environment\":\"Test\",\"Owner\":\"RSE\"},\"additionalInfo\":{\"ImageType\":\"Canonical\",\"ServiceType\":\"Standard_D1\"}}}",
        "meterId": "e60caf26-9ba0-413d-a422-6141f58081d6",
        "infoFields": {},
        "quantity": 8

      },
    },


### <a name="import-data-from-the-usage-api-into-cloud-cruiser"></a>將資料從 Usage API 匯入到 Cloud Cruiser
Cloud Cruiser 活頁簿提供自動化的方式收集和處理來自 Usage API 的資訊。 ETL (擷取-轉換-載入) 活頁簿可讓您設定資料的集合、轉換和發佈至 Cloud Cruiser 資料庫。

每個活頁簿可以有一個或多個集合。 這可讓您將來自不同資源的資訊相互關聯，以補充或強化使用狀況資料。 在此範例中，我們會在 Azure 範本活頁簿中建立新的工作表 (UsageAPI) 並設定新的「集合」以從 Usage API 匯入資訊。

![圖 3 - 匯入至 UsageAPI 工作表的 Usage API 資料][12]

請注意，此活頁簿已有從 Azure 匯入服務的其他工作表 (*ImportServices*)，以及從 Billing API 處理耗用量資訊的工作表 (*PublishData*)。

接下來，我們要使用 Usage API來填入 UsageAPI 工作表，然後在 PublishData 工作表上建立此資訊與來自 Billing API 之耗用量資料的相互關聯。

### <a name="processing-the-tag-information-from-the-usage-api"></a>處理來自 Usage API 的標記資訊
將資料匯入活頁簿之後，我們會在 UsageAPI 工作表中建立轉換步驟以處理來自 API 的資訊。 第一個步驟是使用「JSON 分割」處理器，從單一欄位擷取標記，再為每個標記建立欄位 (部門、專案、擁有者和環境)。

![圖 4 - 建立新的標記資訊欄位][13]

請注意，「網路」服務遺漏了標記資訊 (黃色方塊)，但我們可以藉由查看 *ResourceGroupName* 欄位，確認這個服務屬於相同的資源群組。 由於我們已經擁有該資源群組中其他資源的標記，所以稍後我們可以在程序中使用這項資訊，將遺漏的標記套用至此資源。

下一步是建立查閱資料表，將來自標記的資訊關聯至 *ResourceGroupName*。 下一個步驟會使用此查閱資料表，以利用標記資訊充實消耗量資料。

### <a name="adding-the-tag-information-to-the-consumption-data"></a>將標記資訊加入至消耗量資料
現在我們可以跳至處理來自 Billing API 之耗用量資訊的 *PublishData* 工作表，並新增從標記擷取的欄位。 此程序的執行方式為查看上一個步驟中建立的查閱資料表，使用 *ResourceGroupName* 做為查閱的金鑰。

![圖 5 - 將來自查閱的資訊填入帳戶結構中][14]

請注意，已套用「網路」服務的適當帳戶結構欄位，利用遺漏標記修正問題。 我們也在目標資源群組已外的帳戶結構欄位中填入「其他」以在報告中區別它們。

現在我們只需要加入發佈使用情況資料的步驟即可。 在這個步驟中，在費率方案上定義之每個服務的適當費率會套用至使用情況資訊，產生費用也會載入至資料庫。

最棒的部分是您只需要完成此程序一次。 完成活頁簿之後，您只需要將它加入至排程器，它就會依照排程時間每小時或每日執行一次。 然後就只需建立新報告或自訂現有報告，進而藉由分析資料從雲端使用情況取得有意義的見解。

### <a name="next-steps"></a>後續步驟
* 如需建立 Cloud Cruiser 活頁簿和報告的詳細指示，請參閱 Cloud Cruiser 的線上 [文件](http://docs.cloudcruiser.com/) (需要有效的登入)。  如需有關 Cloud Cruiser 的詳細資訊，請連絡 [info@cloudcruiser.com](mailto:info@cloudcruiser.com)。
* 請參閱 [深入了解 Microsoft Azure 資源耗用量](billing-usage-rate-card-overview.md) 以取得 Azure 資源使用情況和 RateCard API 的概觀。
* 查看 [Azure 計費 REST API 參考](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c) 以取得屬於 Azure 資源管理員所提供之 API 集合的兩個 API 之詳細資訊。
* 如果您想要探究範例程式碼，請查看 [Azure 程式碼範例](https://azure.microsoft.com/documentation/samples/?term=billing)上的＜Microsoft Azure 計費 API 程式碼範例＞。

### <a name="learn-more"></a>詳細資訊
* 若要深入了解 Azure Resource Manager，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)一文。

<!--Image references-->

[1]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Create-New-Workbook-Collection.png "圖 1 - 建立新的集合"
[2]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Import-Data-From-RateCard.png "圖 2 - 從新集合匯入資料"
[3]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Transformation-Steps-Process-RateCard-Data.png "圖 3 - 可處理收集自 RateCard API 之資料的轉換步驟"
[4]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Publish-RateCard-Data-New-Services-Rates.png "圖 4- 將來自 RateCard API 的資料發佈為新的服務和費率"
[5]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Verify-Azure-Services-And-Pricing1.png "圖 5- 驗證新的服務"
[6]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Verify-Azure-Services-And-Pricing2.png "圖 6- 驗證新的費率方案及相關費率"
[7]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Transforming-WAP-Normalize-Services.png "圖 7 - 轉換 WAP 資料以將服務標準化"
[8]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Workbook-Scheduling.png "圖 8 - 活頁簿排程"
[9]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Workload-Cost-Simulation-Report.png "圖 9 - 工作負載成本比較案例的範例報告"
[10]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/1_ReportWithTags.png "圖 10 - 含有使用標記之細項的報告"
[11]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/2_ResourceGroupsWithTags.png "圖 11 - 在 Azure 入口網站上具有相關標記的資源群組"
[12]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/3_ImportIntoUsageAPISheet.png "圖 12 - 匯入至 UsageAPI 工作表的 Usage API 資料"
[13]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/4_NewTagField.png "圖 13 - 建立新的標記資訊欄位"
[14]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/5_PopulateAccountStructure.png "圖 14 - 將來自查閱的資訊填入帳戶結構中"
