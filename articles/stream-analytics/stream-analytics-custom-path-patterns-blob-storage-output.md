---
title: Azure 串流分析 Blob 儲存體輸出的自訂日期時間路徑模式 (預覽)
description: ''
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: da29c6bd8ddc1e2f62a78fb683df5e1784141722
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452559"
---
# <a name="custom-datetime-path-patterns-for-azure-stream-analytics-blob-storage-output-preview"></a>Azure 串流分析 Blob 儲存體輸出的自訂日期時間路徑模式 (預覽)

Azure 串流分析支援在 Blob 儲存體輸出檔案路徑中使用自訂日期和時間格式規範。 自訂日期時間路徑模式可讓您指定與 Hive 串流慣例一致的輸出格式，讓 Azure 串流分析得以將資料傳送至 Azure HDInsight 和 Azure Databricks 以進行下游處理。 自訂日期時間路徑模式可以使用您 Blob 輸出 [路徑前置詞] 欄位中的 `datetime` 關鍵字，輕易地實作。 例如： `{datetime:yyyy}`。

針對 [Azure 入口網站](https://portal.azure.com/?Microsoft_Azure_StreamAnalytics_bloboutputcustomdatetimeformats=true)使用此連結切換功能旗標，該旗標會啟用 Blob 儲存體輸出預覽的自訂日期時間路徑模式。 此功能即將在主要入口網站中啟用。

## <a name="supported-tokens"></a>支援的權杖

下列格式規範權杖可以單獨使用，或者合併使用以達成自訂日期時間格式：

|格式規範   |說明   |範例時間 2018-01-02T10:06:08 的結果|
|----------|-----------|------------|
|{datetime:yyyy}|四位數的年份|2018|
|{datetime:MM}|從 01 到 12 的月份|01|
|{datetime:M}|從 1 到 12 的月份|1|
|{datetime:dd}|從 01 到 31 的日期|02|
|{datetime:d}|從 1 到 12 的日期|2|
|{datetime:HH}|使用 24 小時格式的小時，從 00 到 23|10|
|{datetime:mm}|從 00 到 24 的分鐘數|06|
|{datetime:m}|從 0 到 24 的分鐘數|6|
|{datetime:ss}|從 00 到 60 的秒數|08|

如果您不希望使用自訂日期時間模式，可以將 {date} 和/或 {time} 權杖新增至 [路徑前置詞]，以產生具有內建日期時間格式的下拉式清單。

![串流分析舊日期時間格式](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-old-date-time-formats.png)

## <a name="extensibility-and-restrictions"></a>擴充性及限制

您可以在路徑模式中使用任意數量的權杖、`{datetime:<specifier>}`，直到達到 [路徑前置詞] 的字元限制為止。 若是單一權杖內的格式規範無法合併，應是其超過已由日期和時間下拉式清單列出的合併。 

針對 `logs/MM/dd` 的路徑分割：

|有效運算式   |無效運算式   |
|----------|-----------|
|`logs/{datetime:MM}/{datetime:dd}`|`logs/{datetime:MM/dd}`|

您可能會在 [路徑前置詞] 中使用相同的格式規範多次。 每次都必須重複權杖。

## <a name="hive-streaming-conventions"></a>Hive 串流慣例

Blob 儲存體的自訂路徑模式可與 Hive 串流慣例一起使用，該慣例預期資料夾以資料夾名稱中的 `column=` 標示。

例如： `year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}/hour={datetime:HH}`。

自訂輸出可以免除改變資料表及手動將分割新增至 Azure 串流分析與 Hive 之間連接埠資料的麻煩。 相反地，許多資料夾可以使用以下方式自動新增：

```
MSCK REPAIR TABLE while hive.exec.dynamic.partition true
```

### <a name="example"></a>範例

根據 [Azure 串流分析 Azure 入口網站](stream-analytics-quick-create-portal.md)快速入門指南，建立儲存體帳戶、資源群組、串流分析作業和輸入來源。 使用在快速入門指南中使用的相同範例資料，也可以在 [GitHub](https://raw.githubusercontent.com/Azure/azure-stream-analytics/master/Samples/GettingStarted/HelloWorldASA-InputStream.json) 上取得。

以下列組態建立 Blob 輸出接收：

![串流分析會建立 Blob 輸出接收](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-create-output-sink.png)

完整路徑模式如下所示：

```
year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}
```

當您啟動作業時，以路徑模式為基礎的資料夾結構會在 Blob 容器中建立。 您可以向下切入至日層級。

![具有自訂路徑模式的串流分析 Blob 輸出](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-blob-output-folder-structure.png)

## <a name="next-steps"></a>後續步驟

* [了解來自 Azure 串流分析的輸出](stream-analytics-define-outputs.md)
