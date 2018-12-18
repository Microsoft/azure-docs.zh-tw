---
title: Azure 事件中樞擷取概觀 | Microsoft Docs
description: 使用事件中樞擷取來擷取遙測資料
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: e53cdeea-8a6a-474e-9f96-59d43c0e8562
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: 603a5dfcf2137c15ae19ea248f3e0f4f136c22f1
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42145750"
---
# <a name="azure-event-hubs-capture"></a>Azure 事件中樞擷取

Azure 事件中樞擷取可讓您自動將事件中樞的串流資料傳遞到您選擇的 [Azure Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)或 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/) 帳戶，並另外增加了可指定時間或大小間隔的彈性。 設定擷取的作業很快，因此執行時不需要系統管理成本，而且它可以針對事件中樞的[輸送量單位](event-hubs-features.md#capacity)自動進行調整。 事件中樞擷取是將串流資料載入至 Azure 的最簡單方式，並可讓您專注於處理資料而非擷取資料。

事件中樞擷取可讓您在相同資料流上處理即時和批次型的管線。 這表示您可以建置會隨時間配合需求成長的解決方案。 不論您現在是要建置著眼於未來即時處理的批次型系統，或想要為現有即時解決方案新增有效率的冷路徑，事件中樞擷取都可以讓使用串流資料變得更簡單。

## <a name="how-event-hubs-capture-works"></a>事件中樞擷取的運作方式

事件中樞是用於輸入遙測的時間保留持久緩衝區，類似於分散式記錄。 在事件中樞調整大小的關鍵是 [資料分割取用者模型](event-hubs-features.md#partitions)。 每個資料分割都是獨立的資料區段，可獨立取用。 根據可設定的保留期限，此資料會隨時間而過時。 因此，給定的事件中樞永遠不會「太滿」。

事件中樞擷取可讓您指定自己的 Azure Blob 儲存體帳戶和容器，或是 Azure Data Lake Store 帳戶，可用來儲存擷取的資料。 這些帳戶可以和事件中樞位於相同區域，也可以位於另一個區域，從而新增至事件中樞擷取功能的彈性。

擷取的資料會以 [Apache Avro][Apache Avro] 格式寫入，此為精簡、快速、二進位的格式，可使用內嵌結構描述提供豐富的資料結構。 此格式廣泛運用在 Hadoop 生態系統、串流分析和 Azure Data Factory。 關於使用 Avro 的詳細資訊可在本文稍後看到。

### <a name="capture-windowing"></a>擷取範圍

事件中樞擷取可讓您設定要控制擷取的範圍。 此範圍是具有「先者勝出原則」的最小大小和時間組態，這表示所遇到的第一個觸發條件會導致擷取作業。 如果您有一個 15 分鐘 100 MB 的擷取範圍，且傳送速率為每秒 1 MB，則大小範圍會比時間範圍更早觸發。 每個分割區都會獨立擷取，並在擷取時寫入已完成的區塊 Blob，而且會以遇到擷取間隔的時間命名。 儲存體命名慣例如下︰

```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

請注意，日期值會以零填補；檔名範例可能是：

```
https://mystorageaccount.blob.core.windows.net/mycontainer/mynamespace/myeventhub/0/2017/12/08/03/03/17.avro
```

### <a name="scaling-to-throughput-units"></a>調整至輸送量單位

事件中樞的流量會受到 [輸送量單位](event-hubs-features.md#capacity)控制。 單一輸送量單位可允許每秒 1 MB 或每秒 1000 個事件的輸入，輸出則為此數量的兩倍。 標準事件中樞可以設定 1-20 個輸送量單位，您可以透過增加配額的[支援要求][support request]購買更多單位。 超過所購買輸送量單位的使用量將遭到節流。 事件中樞擷取會直接從內部的事件中樞儲存體複製資料，略過輸送量單位輸出配額，並將輸出節省下來以供串流分析或 Spark 等其他處理讀取器使用。

一旦設定，事件中樞擷取會在您傳送第一個事件時自動執行，並且持續執行。 為了讓下游處理更輕鬆地知道處理程序正在運作，事件中樞會在沒有資料時寫入空白檔案。 這個程序會提供可預測的頻率和標記，以供饋送給批次處理器。

## <a name="setting-up-event-hubs-capture"></a>設定事件中樞擷取

您可以使用 [Azure 入口網站](https://portal.azure.com)或使用 Azure Resource Manager 範本，在建立事件中樞時設定擷取功能。 如需詳細資訊，請參閱下列文章：

- [使用 Azure 入口網站啟用事件中樞擷取功能](event-hubs-capture-enable-through-portal.md)
- [使用 Azure Resource Manager 範本建立含有一個事件中樞的事件中樞命名空間並啟用擷取](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)

## <a name="exploring-the-captured-files-and-working-with-avro"></a>瀏覽擷取檔案並使用 Avro

事件中樞擷取會以 Avro 格式建立檔案，如設定之時間範圍內所指定。 您可以在任何工具 (例如 [Azure 儲存體總管][Azure Storage Explorer]) 檢視這些檔案。 您可以在本機下載檔案，以對其進行處理。

事件中樞擷取所產生的檔案會有下列 Avro 結構描述︰

![][3]

瀏覽 Avro 檔案的簡易方式是使用 Apache 所提供的 [Avro Tools][Avro Tools] jar。 下載這個 jar 之後，您可以執行下列命令來查看特定 Avro 檔案的結構描述︰

```shell
java -jar avro-tools-1.8.2.jar getschema <name of capture file>
```

此命令會傳回

```json
{

    "type":"record",
    "name":"EventData",
    "namespace":"Microsoft.ServiceBus.Messaging",
    "fields":[
                 {"name":"SequenceNumber","type":"long"},
                 {"name":"Offset","type":"string"},
                 {"name":"EnqueuedTimeUtc","type":"string"},
                 {"name":"SystemProperties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Properties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Body","type":["null","bytes"]}
             ]
}
```

您也可以使用 Avro Tools 將檔案轉換成 JSON 格式，並執行其他處理。

若要執行更進階的處理，請下載並安裝您所選平台適用的 Avro。 在本文撰寫當下，已有適用於 C、C++、C\#、Java、NodeJS、Perl、PHP、Python 和 Ruby 的實作。

Apache Avro 已完成適用於 [Java][Java] 和 [Python][Python] 的快速入門指南。 您也可以閱讀[開始使用事件中樞擷取](event-hubs-capture-python.md)一文。

## <a name="how-event-hubs-capture-is-charged"></a>事件中樞擷取的收費方式

事件中樞擷取的計量方式類似輸送量單位，屬於每小時的費用。 其費用與命名空間所購買的輸送量單位數目成正比。 當輸送量單位增加和減少時，事件中樞擷取也會增加和減少以提供相符的效能。 計量會串聯地發生。 如需定價詳細資訊，請參閱[事件中樞定價](https://azure.microsoft.com/pricing/details/event-hubs/)。 

## <a name="next-steps"></a>後續步驟

事件中樞擷取是將資料載入 Azure 中最簡單的方式。 使用 Azure Data Lake、Azure Data Factory 和 Azure HDInsight，即可使用您選擇的工具和平台，以您需要的規模執行批次處理和其他分析。

您可以造訪下列連結以深入了解事件中樞︰

* [開始傳送和接收事件](event-hubs-dotnet-framework-getstarted-send.md)
* [事件中樞概觀][Event Hubs overview]

[Apache Avro]: http://avro.apache.org/
[support request]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[Azure Storage Explorer]: http://azurestorageexplorer.codeplex.com/
[3]: ./media/event-hubs-capture-overview/event-hubs-capture3.png
[Avro Tools]: http://www-us.apache.org/dist/avro/avro-1.8.2/java/avro-tools-1.8.2.jar
[Java]: http://avro.apache.org/docs/current/gettingstartedjava.html
[Python]: http://avro.apache.org/docs/current/gettingstartedpython.html
[Event Hubs overview]: event-hubs-what-is-event-hubs.md
