---
title: 適用於 Apache Kafka 的 Azure 事件中樞 | Microsoft Docs
description: 已啟用 Kafka 的 Azure 事件中樞的概觀和簡介
services: event-hubs
documentationcenter: .net
author: basilhariri
manager: timlt
ms.service: event-hubs
ms.topic: article
ms.date: 08/16/2018
ms.author: bahariri
ms.openlocfilehash: 16c101068be48ba1435ef230b29c679fcef17d08
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42140495"
---
# <a name="azure-event-hubs-for-apache-kafka-preview"></a>適用於 Apache Kafka 的 Azure 事件中樞 (預覽)

事件中樞提供的 Kafka 端點可讓您現有的 Kafka 型應用程式使用，作為執行您本身的 Kafka 叢集以外的另一項選擇。 事件中樞支援 [Apache Kafka 1.0](https://kafka.apache.org/10/documentation.html) 和更新的用戶端版本，並且可與您現有的 Kafka 應用程式搭配使用，包括 MirrorMaker。 

## <a name="what-does-event-hubs-for-kafka-provide"></a>適用於 Kafka 的事件中樞提供哪些功能？

適用於 Kafka 的事件中樞功能在 Azure 事件中樞之上提供了通訊協定標頭，與 Kafka 1.0 版和更新版本之間具有讀取和寫入 Kafka 主題的二進位相容性。 您可以從應用程式開始使用 Kafka 端點，不必變更程式碼，但需要稍微變更設定。 您可以更新設定中的連接字串，以指向事件中樞所公開的 Kafka 端點，而非指向 Kafka 叢集。 然後，您可以開始將事件從使用 Kafka 通訊協定的應用程式串流到事件中樞。 

Kafka 和事件中樞在概念上幾乎是相同的：兩者都具有為了串流資料而建置的分割記錄。 下表提供 Kafka 與事件中樞之間的概念對應。

### <a name="kafka-and-event-hub-conceptual-mapping"></a>Kafka 和事件中樞的概念對應

| Kafka 概念 | 事件中樞概念|
| --- | --- |
| 叢集 | 命名空間 |
| 話題 | 事件中樞 |
| 資料分割 | 資料分割|
| 取用者群組 | 取用者群組 |
| Offset | Offset|

### <a name="key-differences-between-kafka-and-event-hubs"></a>Kafka 與事件中樞的主要差異

[Apache Kafka](https://kafka.apache.org/) 是一項軟體，可供您隨處執行，而事件中樞則是類似於 Azure Blob 儲存體的雲端服務。 並沒有任何伺服器或網路需要管理，也沒有需要設定的代理程式。 您會建立命名空間，作為您的主題存留所在的 FQDN，然後在該命名空間中建立事件中樞或主題。 如需關於事件中樞和命名空間的詳細資訊，請參閱[事件中樞功能](event-hubs-features.md#namespace)。 作為雲端服務，事件中樞會使用穩定的單一虛擬 IP 位址作為端點，因此用戶端不需要了解叢集內的代理程式或機器。 

事件中樞內的調整由您所購買的輸送量單位數所控制，每個輸送量單位可讓您獲得每秒輸送 1 MB 或每秒輸入 1000 個事件的配額。 根據預設，事件中樞會在您達到限制時使用[自動擴充](event-hubs-auto-inflate.md)功能相應增加輸送量單位；此功能也可供適用於 Kafka 的事件中樞功能使用。 

### <a name="security-and-authentication"></a>安全性和驗證

Azure 事件中樞的所有通訊都需要 SSL 或 TLS，且會使用共用存取簽章 (SAS) 進行驗證。 事件中樞內的 Kafka 端點也有此需求。 為了與 Kafka 相容，事件中樞使用 SASL PLAIN 進行驗證，並使用 SASL SSL 維護傳輸安全性。 如需與事件中樞的安全性有關的詳細資訊，請參閱[事件中樞驗證和安全性](event-hubs-authentication-and-security-model-overview.md)。

## <a name="other-event-hubs-features-available-for-kafka"></a>適用於 Kafka 的其他事件中樞功能

適用於 Kafka 的事件中樞功能可讓您以不同的通訊協定執行寫入和讀取，讓您目前的 Kafka 產生者可以繼續透過 Kafka 發行，並讓您能透過事件中樞新增讀取器，例如 Azure 串流分析或 Azure Functions。 此外，[擷取](event-hubs-capture-overview.md)和[異地災害復原](event-hubs-geo-dr.md)等事件中樞功能也可供適用於 Kafka 的事件中樞功能使用。

## <a name="features-that-are-not-supported-in-the-preview"></a>預覽版不支援的功能

適用於 Kafka 整合的事件中樞在公開預覽階段不支援下列 Kafka 功能：

*   等冪產生者
*   交易
*   壓縮
*   以大小為基礎的保留期
*   記錄壓縮
*   將分割區新增至現有主題
*   HTTP Kafka API 支援
*   Kafka Connect
*   Kafka Streams

## <a name="next-steps"></a>後續步驟

本文概略介紹了適用於 Kafka 的事件中樞。 若要深入了解，請參閱下列連結：

* [如何建立已啟用 Kafka 的事件中樞](event-hubs-create-kafka-enabled.md)
* [從您的 Kafka 應用程式串流到事件中樞](event-hubs-quickstart-kafka-enabled-event-hubs.md)
* 開始使用[事件中樞教學課程](event-hubs-dotnet-standard-getstarted-send.md)
* [事件中樞常見問題集](event-hubs-faq.md)

 
 

