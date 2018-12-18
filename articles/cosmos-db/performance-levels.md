---
title: 已淘汰的 Azure Cosmos DB 效能層級 | Microsoft Docs
description: 了解 Azure Cosmos DB 中先前可用的 S1、S2 和 S3 效能層級。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 06/04/2018
ms.author: sngun
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d1bb7551e6dfb6c42853ab95096f17f5285c69c1
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34796643"
---
# <a name="retiring-the-s1-s2-and-s3-performance-levels"></a>淘汰 S1、S2 和 S3 效能層級

> [!IMPORTANT] 
> 本文所討論的 S1、S2 和 S3 效能層級即將淘汰，而且再也無法供新的 Azure Cosmos DB 帳戶使用。
>

本文概述 S1、S2 和 S3 效能層級，並討論如何將使用這些效能層級的集合移轉至單一分割區的集合。 閱讀本文後，您將能夠回答下列問題：

- [為何要淘汰 S1、S2 和 S3 效能層級？](#why-retired)
- [單一分割區集合和資料分割的集合與 S1、S2、S3 效能層級之比較為何？](#compare)
- [我該怎麼做才能確保不中斷地的存取我的資料？](#uninterrupted-access)
- [在移轉之後我的集合會如何變更？](#collection-change)
- [我移轉至單一資料分割集合之後，我的帳單將如何變更呢？](#billing-change)
- [如果我需要超過 10 GB 的儲存體會如何？](#more-storage-needed)
- [在計劃性移轉之前，我可以在 S1、S2 和 S3 效能層級之間變更嗎？](#change-before)
- [如何自己從 S1、S2、S3 效能層級移轉至單一資料分割集合？](#migrate-diy)
- [如果我是 EA 客戶會受到什麼影響？](#ea-customer)

<a name="why-retired"></a>

## <a name="why-are-the-s1-s2-and-s3-performance-levels-being-retired"></a>為何 S1、S2 和 S3 效能層級將要淘汰？

S1、S2 和 S3 效能層級不提供標準 Azure Cosmos DB 所提供的彈性。 使用 S1、S2、S3 效能層級，輸送量和儲存容量皆會預先設定且不提供彈性。 Azure Cosmos DB 現在可讓您自訂輸送量和儲存體，提供您更大的彈性能夠隨您的需求進行變更。

<a name="compare"></a>

## <a name="how-do-single-partition-collections-and-partitioned-collections-compare-to-the-s1-s2-s3-performance-levels"></a>單一分割區集合和資料分割的集合與 S1、S2、S3 效能層級之比較為何？

下表比較單一資料分割集合、資料分割集合和 S1、S2、S3 效能層級中可用的輸送量和儲存體選項。 美國東部 2 區域的範例如下︰

|   |資料分割的集合|單一資料分割集合|S1|S2|S3|
|---|---|---|---|---|---|
|最大輸送量|無限|10K RU/秒|250 RU/秒|1 K RU/秒|2.5 K RU/秒|
|輸送量下限|2.5 K RU/秒|400 RU/秒|250 RU/秒|1 K RU/秒|2.5 K RU/秒|
|儲存體上限|無限|10 GB|10 GB|10 GB|10 GB|
|價格 (每月)|輸送量：$6 / 100 RU/秒<br><br>儲存體：$0.25/GB|輸送量：$6 / 100 RU/秒<br><br>儲存體：$0.25/GB|$25 美元|$50 美元|$100 美元|

您是 EA 客戶嗎？ 如果是，請參閱[如果我是 EA 客戶會受到什麼影響？](#ea-customer)

<a name="uninterrupted-access"></a>

## <a name="what-do-i-need-to-do-to-ensure-uninterrupted-access-to-my-data"></a>我該怎麼做才能確保不中斷地的存取我的資料？

如果您有 S1、S2 或 S3 集合，就應該[使用 .NET SDK](#migrate-diy)，以程式設計方式將該集合移轉至單一分割區的集合。 

<a name="collection-change"></a>

## <a name="how-will-my-collection-change-after-the-migration"></a>在移轉之後我的集合會如何變更？

如果您有 S1 集合，就能以 400 RU/秒的輸送量將它們移轉至單一分割區的集合。 400 RU/秒是適用於單一資料分割集合的最低輸送量。 不過，單一資料分割集合中 400 RU/秒的成本，大致上與您已支付的 S1 集合與 250 RU/秒相同 – 因此您不需要支付供您使用的額外 150 RU/秒。

如果您有 S2 集合，就能以 1 K RU/秒的輸送量將它們移轉至單一分割區的集合。 您不會看到您的輸送量層級有任何變更。

如果您有 S3 集合，就能以 2.5 K RU/秒的輸送量將它們移轉至單一分割區的集合。 您不會看到您的輸送量層級有任何變更。

在這其中的每一個情況下，當您移轉集合之後，將能自訂輸送量層級，或視需要將它相應增加及減少，以便為使用者提供低延遲存取。 

<a name="billing-change"></a>

## <a name="how-will-my-billing-change-after-i-migrated-to-the-single-partition-collections"></a>當我移轉至單一分割區的集合之後，帳單會如何變更？

假設您在美國東部區域有 10 個 S1 集合，每個有 1 GB 的儲存空間，您將這 10 個 S1 集合以 400 RU/秒 (最小層級) 移轉至 10 個單一資料分割集合。 如果您一整個月保留 10 個單一資料分割集合，您的帳單會看起來如下︰

![10 個集合的 S1 價格與使用單一資料分割集合的 10 個集合價格比較會如何](./media/performance-levels/s1-vs-standard-pricing.png)

<a name="more-storage-needed"></a>

## <a name="what-if-i-need-more-than-10-gb-of-storage"></a>如果我需要超過 10 GB 的儲存空間怎麼辦？

無論您擁有具備 S1、S2 或 S3 效能層級的集合，或是擁有單一分割區的集合，全部都有 10 GB 的可用儲存空間。您可以利用幾乎不受限制的儲存空間，使用 Azure Cosmos DB 資料移轉工具來將資料移轉至分割區的集合。 如需資料分割集合優點的相關詳細資訊，請參閱 [Azure Cosmos DB 中的資料分割與規模調整](sql-api-partition-data.md)。 

<a name="change-before"></a>

## <a name="can-i-change-between-the-s1-s2-and-s3-performance-levels-before-the-planned-migration"></a>在計劃性移轉之前，我可以在 S1、S2 和 S3 效能層級之間變更嗎？

只有具備 S1、S2 和 S3 效能的現有帳戶可以變更，並[使用 .NET SDK](#migrate-diy)，以程式設計方式改變效能層級層。 如果您從 S1、S3 或 S3 變更為單一資料分割集合，將無法回到 S1、S2 或 S3 效能層級。

<a name="migrate-diy"></a>

## <a name="how-do-i-migrate-from-the-s1-s2-s3-performance-levels-to-single-partition-collections-on-my-own"></a>如何自己從 S1、S2、S3 效能層級移轉至單一資料分割集合？

您可以[使用 .NET SDK](#migrate-diy)，以程式設計方式從 S1、S2 和 S3 效能層級移轉至單一分割區的集合。 您可以在計劃移轉之前自行執行此動作，以利用單一分割區集合所提供的彈性輸送量選項。

### <a name="migrate-to-single-partition-collections-by-using-the-net-sdk"></a>使用 .NET SDK 移轉至單一分割區的集合

本節只涵蓋使用 [SQL .NET API](sql-api-sdk-dotnet.md) 來變更集合的效能層級，但程序類似於我們的其他 SDK。

以下是變更集合輸送量為每秒 5,000 要求單位的程式碼片段：
    
```csharp
    //Fetch the resource to be updated
    Offer offer = client.CreateOfferQuery()
                      .Where(r => r.ResourceLink == collection.SelfLink)    
                      .AsEnumerable()
                      .SingleOrDefault();

    // Set the throughput to 5000 request units per second
    offer = new OfferV2(offer, 5000);

    //Now persist these changes to the database by replacing the original resource
    await client.ReplaceOfferAsync(offer);
```

請造訪 [MSDN](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.aspx) 以檢視其他範例，並深入了解我們提供的方法：

* [**ReadOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readofferasync.aspx)
* [**ReadOffersFeedAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readoffersfeedasync.aspx)
* [**ReplaceOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.replaceofferasync.aspx)
* [**CreateOfferQuery**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.linq.documentqueryable.createofferquery.aspx)

<a name="ea-customer"></a>

## <a name="how-am-i-impacted-if-im-an-ea-customer"></a>如果我是 EA 客戶會受到什麼影響？

EA 客戶將會受保護的價格，直到其目前合約結束。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure Cosmos DB 的價格和管理資料，請探索這些資源：

1.  [在 Cosmos DB 中分割資料](sql-api-partition-data.md)。 了解單一資料分割容器和資料分割容器的差異，以及實作資料分割策略以順暢地調整的秘訣。
2.  [Cosmos DB 定價](https://azure.microsoft.com/pricing/details/cosmos-db/)。 深入了解佈建輸送量和使用儲存體的成本。
3.  [要求單位](request-units.md)。 了解不同作業類型，例如讀取、寫入、查詢的輸送量耗用量。
