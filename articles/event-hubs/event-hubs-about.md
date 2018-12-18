---
title: Azure 事件中樞是什麼？ | Microsoft Docs
description: 深入了解 Azure 事件中樞，這是一個每秒可內嵌數百萬個事件的巨量資料串流服務。
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.topic: overview
ms.custom: mvc
ms.date: 08/01/2018
ms.author: shvija
ms.openlocfilehash: 336756e72c6c07f52101ddd0cd99a6ef9744d3c3
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49351756"
---
# <a name="what-is-azure-event-hubs"></a>Azure 事件中樞是什麼？

Azure 事件中樞是巨量資料串流平台和事件擷取服務，每秒可接收和處理數百萬個事件。 事件中樞可以處理及儲存分散式軟體和裝置所產生的事件、資料或遙測。 傳送至事件中樞的資料可以透過任何即時分析提供者或批次/儲存體配接器來轉換和儲存。 

事件中樞會用於下列幾個常見案例：

- 異常偵測 (詐騙/極端值)
- 應用程式記錄檔
- 分析管線 (例如點選流)
- 即時儀表板
- 封存資料
- 交易處理
- 使用者遙測處理
- 裝置遙測串流 

## <a name="why-use-event-hubs"></a>為何使用事件中樞？

只有在能輕鬆地處理資料並從資料來源中獲得即時見解時，資料才顯得有價值。 事件中樞提供低延遲的分散式串流處理平台，並與 Azure 內部或外部的資料及分析服務完美整合，藉以建置完整的巨量資料管線。

事件中樞代表事件管線的「大門」，在方案架構中通常稱為「事件擷取器」。 事件擷取器是介於事件發佈者和事件取用者之間的元件或服務，它能將事件串流的生產與這些事件的取用彼此脫鉤。 事件中樞提供具有時間保留緩衝的統一串流平台，可分開處理事件產生者和事件取用者。 

下列各節會說明 Azure 事件中樞服務的主要功能： 

## <a name="fully-managed-paas"></a>完全受控 PaaS 

事件中樞是僅需一點設定或管理成本的受控服務，因此您可將重心放在商務解決方案上。 [Apache Kafka 生態系統的事件中樞](event-hubs-for-kafka-ecosystem-overview.md)提供您無須管理、設定或執行叢集的 PaaS Kafka 體驗。

## <a name="support-for-real-time-and-batch-processing"></a>支援即時和批次處理

即時內嵌、緩衝、儲存及處理您的資料流，以取得可採取動作的見解。 事件中樞會使用[分割的取用者模型](event-hubs-features.md#partitions)，可讓多個應用程式同時處理資料流，並讓您控制處理的速度。

近乎即時地[擷取 ](event-hubs-capture-overview.md)[Azure Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)或 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)  中的資料，以用於長期保留或微批次處理。 您可以在用來取得即時分析的相同資料流上達到此目的。 設定擷取的作業很快，因此執行時不需要系統管理成本，而且它可以針對事件中樞的 [輸送量單位](event-hubs-features.md#throughput-units)自動進行調整。 事件中樞擷取可讓您專注於資料處理，而不是資料擷取。

Azure 事件中樞也整合了 [Azure Functions](/azure/azure-functions/) 來達到無伺服器架構。

## <a name="scalable"></a>可調整 

透過事件中樞，您可以先使用以 MB 為單位的資料流，然後成長到 GB 或 TB。 [自動擴充](event-hubs-auto-inflate.md)功能是用來調整輸送量單位數目以符合使用量需求的多個選項之一。 

## <a name="rich-ecosystem"></a>豐富的生態系統

[Apache Kafka 生態系統的事件中樞](event-hubs-for-kafka-ecosystem-overview.md)可讓 [Apache Kafka (1.0 版和更新版本)](https://kafka.apache.org/) 用戶端和應用程式與事件中樞通訊，而且不需要管理任何叢集。
 
透過適用於各種[語言 (.NET、Java、Python、Go、Node.js)](https://github.com/Azure/azure-event-hubs) 的廣大生態系統，您可以輕鬆地開始處理來自事件中樞的串流。 所有支援的用戶端語言皆提供低階整合。

## <a name="key-architecture-components"></a>重要架構元件

事件中樞提供訊息串流處理能力，但特性迴異於傳統企業傳訊。 事件中樞功能是以高輸送量和事件處理案例為重點。 事件中樞包含下列[重要元件](event-hubs-features.md)：

- **事件產生者**：任何將資料傳送至事件中樞的實體。 事件發佈者可以使用 HTTPS、AMQP 1.0 或 Apache Kafka (1.0 版或更新版本) 發佈事件
- **分割區**：每一個取用者只會讀取訊息資料流的特定子集或分割區。
- **取用者群組**：檢視整個事件中樞 (狀態、位置或位移) 的窗口。 取用者群組能讓多個取用應用程式擁有自己的事件串流檢視，以及按照自己的步調及運用自己的位移自行讀取串流。
- **輸送量單位**：預先購買的容量單位，可控制事件中樞的輸送量容量。
- **事件接收者**：任何從事件中樞讀取事件資料的實體。 所有事件中樞取用者都透過 AMQP 1.0 工作階段連線，而可供取用的事件都透過工作階段傳遞。 所有 Kafka 取用者都是透過 Kafka 通訊協定 1.0 和更新版本連線。

下圖顯示事件中樞串流處理架構︰

![事件中樞](./media/event-hubs-about/event_hubs_architecture.png)


## <a name="next-steps"></a>後續步驟

若要開始使用事件中樞，請參閱下列文章：

* [內嵌至事件中樞](event-hubs-quickstart-portal.md)
* [事件中樞功能概觀](event-hubs-features.md)
* [常見問題集](event-hubs-faq.md)


