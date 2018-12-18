---
title: Azure 媒體服務遙測 | Microsoft Docs
description: 本文提供 Azure 媒體服務遙測的概觀。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 95c20ec4-c782-4063-8042-b79f95741d28
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/29/2017
ms.author: juliako
ms.openlocfilehash: 97df0876afd8b7258f985ab375b14f4aabde6e22
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786137"
---
# <a name="azure-media-services-telemetry"></a>Azure 媒體服務遙測

Azure 媒體服務 (AMS) 可讓您存取其服務的遙測/計量資料。 目前的 AMS 版本可讓您收集直播**頻道**、**串流端點**和即時**封存**實體的遙測資料。 

遙測資料會寫入您指定的 Azure 儲存體帳戶中的儲存體資料表 (一般而言，您可使用與您的 AMS 帳戶相關聯的儲存體帳戶)。 

遙測系統不會管理資料保留。 您可以刪除儲存體資料表來移除舊的遙測資料。

本主題討論如何設定及使用 AMS 遙測。

## <a name="configuring-telemetry"></a>設定遙測

您可以設定元件層級細微度的遙測。 詳細資料層級可分為「正常」和「詳細資訊」兩種。 目前，這兩個層級會傳回相同的資訊。 建議使用「正常」。 

下列主題說明如何啟用遙測︰

[啟用搭配 .NET 的遙測](media-services-dotnet-telemetry.md) 

[啟用搭配 REST 的遙測](media-services-rest-telemetry.md)

## <a name="consuming-telemetry-information"></a>取用遙測資訊

遙測資料會寫入設定媒體服務帳戶的遙測時所指定之儲存體帳戶中的 Azure 儲存體資料表。 本節描述計量的儲存體資料表。

您可以使用下列其中一個方法取用遙測資料︰

- 直接從 Azure 表格儲存體 (例如使用儲存體 SDK) 中讀取資料。 如需遙測儲存體資料表的說明，請參閱[這個](https://msdn.microsoft.com/library/mt742089.aspx)主題中的**取用遙測資訊**。

或

- 使用媒體服務 .NET SDK 中的支援讀取儲存體資料，如[這個](media-services-dotnet-telemetry.md)主題所述。 


下述遙測結構描述可在 Azure 資料表儲存體的限制內提供良好的效能︰

- 資料會依帳戶識別碼和服務識別碼分割，以允許獨立查詢每個服務的遙測。
- 磁碟分割包含指定磁碟分割大小合理上限的日期。
- 資料列索引鍵是逆時針方向順序排列，可允許查詢指定服務最新的遙測項目。

這樣可提高許多常見查詢的效率︰

- 以平行方式獨立下載個別服務的資料。
- 擷取某日期範圍內指定服務的所有資料。
- 擷取服務的最新資料。

### <a name="telemetry-table-storage-output-schema"></a>遙測資料表儲存體輸出結構描述

遙測資料會以彙總方式儲存在一個名為 "TelemetryMetrics20160321" 的資料表中，其中的 "20160321" 是建立資料表的日期。 遙測系統會以 00:00 UTC 為基準，為每一天建立個別的資料表。 此資料表用來儲存週期性值，例如指定時間範圍內的內嵌位元速率、傳送的位元組等。 

屬性|值|範例/附註
---|---|---
PartitionKey|{帳戶識別碼} _ {實體識別碼}|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66<br/<br/>帳戶識別碼包含在分割區索引鍵中，以簡化多個媒體服務帳戶寫入同一儲存體帳戶的工作流程。
RowKey|{到午夜的秒數}_{隨機值}|01688_00199<br/><br/>資料列索引鍵以到午夜的秒數開始，以允許磁碟分割內的前 n 個樣式查詢。 如需詳細資訊，請參閱[本篇文章](../../cosmos-db/table-storage-design-guide.md#log-tail-pattern)。 
Timestamp|日期/時間|Azure 資料表中的自動時間戳記 2016-09-09T22:43:42.241Z
類型|提供遙測資料之實體的類型|頻道/串流端點/封存<br/><br/>事件類型只是字串值。
Name|遙測事件的名稱|ChannelHeartbeat/StreamingEndpointRequestLog
ObservedTime|遙測事件發生時間 (UTC)|2016-09-09T22:42:36.924Z<br/><br/>觀察的時間由傳送遙測資料的實體提供 (例如頻道)。 元件之間可能有時間同步問題，因此這個值是近似值
ServiceID|{服務識別碼}|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
實體特定屬性|如事件所定義|StreamName：stream1、Bitrate 10123…<br/><br/>其餘的屬性是針對指定的事件類型定義。 Azure 資料表內容是機碼值組。  (亦即資料表中的不同資料列有不同的屬性集)。

### <a name="entity-specific-schema"></a>實體特定結構描述

有三種類型的實體特定遙測資料項目，分別以下列頻率推送︰

- 串流端點：每 30 秒
- 直播頻道︰每分鐘
- 即時封存︰每分鐘

**串流端點**

屬性|值|範例
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Azure 資料表中的自動時間戳記 2016-09-09T22:43:42.241Z
類型|類型|StreamingEndpoint
Name|Name|StreamingEndpointRequestLog
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|服務識別碼|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
HostName|端點的主機名稱|builddemoserver.origin.mediaservices.windows.net
StatusCode|記錄 HTTP 狀態|200
ResultCode|結果碼詳細資料|S_OK
RequestCount|彙總的總要求|3
BytesSent|傳送的彙總位元組|2987358
ServerLatency|平均伺服器延遲 (包括儲存體)|129
E2ELatency|平均端對端延遲|250

**直播頻道**

屬性|值|範例/附註
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Azure 資料表中的自動時間戳記 2016-09-09T22:43:42.241Z
類型|類型|通道
Name|Name|ChannelHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|服務識別碼|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
TrackType|曲目視訊/音訊/文字的類型|視訊/音訊
TrackName|曲目名稱|video/audio_1
Bitrate|曲目位元速率|785000
CustomAttributes||   
IncomingBitrate|實際連入位元速率|784548
OverlapCount|內嵌中的重疊|0
DiscontinuityCount|曲目不連續|0
LastTimestamp|上次內嵌資料時間戳記|1800488800
NonincreasingCount|由於未增加的時間戳記而捨棄的片段數|2
UnalignedKeyFrames|我們是否收到主要畫面未對齊的片段 (跨品質等級) |True
UnalignedPresentationTime|我們是否收到呈現方式時間未對齊的片段 (跨品質等級/曲目)|True
UnexpectedBitrate|如果下列條件成立則為 True：音訊/視訊曲目的計算的/實際的位元速率 > 40,000 bps，且 IncomingBitrate == 0 或 IncomingBitrate 和 actualBitrate 相差 50% |True
Healthy|如果下列條件成立則為 True <br/>overlapCount、 <br/>DiscontinuityCount、 <br/>NonIncreasingCount、 <br/>UnalignedKeyFrames、 <br/>UnalignedPresentationTime 及 <br/>UnexpectedBitrate<br/> 均為 0|True<br/><br/>Healthy 是複合函數，當下列任一條件成立時會傳回 false︰<br/><br/>- OverlapCount > 0<br/>- DiscontinuityCount > 0<br/>- NonincreasingCount > 0<br/>- UnalignedKeyFrames == True<br/>- UnalignedPresentationTime == True<br/>- UnexpectedBitrate == True

**即時封存**

屬性|值|範例/附註
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Azure 資料表中的自動時間戳記 2016-09-09T22:43:42.241Z
類型|類型|封存
Name|Name|ArchiveHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|服務識別碼|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
ManifestName|程式 URL|asset-eb149703-ed0a-483c-91c4-e4066e72cce3/a0a5cfbf-71ec-4bd2-8c01-a92a2b38c9ba.ism
TrackName|曲目名稱|audio_1
TrackType|曲目類型|音訊/視訊
CustomAttribute|十六進位字串，用以區別名稱和位元速率相同的不同曲目 (多重攝影機角度)|
Bitrate|曲目位元速率|785000
Healthy|如果下列條件成立則為 True：FragmentDiscardedCount == 0 && ArchiveAcquisitionError == False|True (這兩個值不會出現在計量，但它們存在於來源事件中)<br/><br/>Healthy 是複合函數，當下列任一條件成立時會傳回 false︰<br/><br/>- FragmentDiscardedCount > 0<br/>- ArchiveAcquisitionError == True

## <a name="general-qa"></a>一般問答集

### <a name="how-to-consume-metrics-data"></a>如何使用計量資料？

計量資料在客戶的儲存體帳戶中會儲存為一系列的 Azure 資料表。 使用下列工具可取用這項資料︰

- AMS SDK
- Microsoft Azure 儲存體總管 (支援匯出至逗點分隔值格式和在 Excel 中處理)
- REST API

### <a name="how-to-find-average-bandwidth-consumption"></a>如何計算平均頻寬使用量？

平均頻寬使用量是某段時間的 BytesSent 平均值。

### <a name="how-to-define-streaming-unit-count"></a>如何定義串流單位計數？

串流單位計數可定義為服務的串流端點中的尖峰輸送量除以一個串流端點的尖峰輸送量。 一個串流端點的尖峰可用輸送量為 160 Mbps。
例如，假設客戶之服務的尖峰輸送量為 40 MBps (某段時間的 BytesSent 最大值)。 則串流單位計數等於 (40 MBps)*(8 位元/位元組)/(160 Mbps) = 2 個串流單位。

### <a name="how-to-find-average-requestssecond"></a>如何計算每秒平均要求數？

若要計算每秒平均要求數，請計算某段時間的平均要求數 (RequestCount)。

### <a name="how-to-define-channel-health"></a>如何定義頻道健康情況？

頻道健康情況可定義為複合布林函數，當下列任一條件成立時就為 false：

- OverlapCount > 0
- DiscontinuityCount > 0
- NonincreasingCount > 0
- UnalignedKeyFrames == True 
- UnalignedPresentationTime == True 
- UnexpectedBitrate == True


### <a name="how-to-detect-discontinuities"></a>如何偵測不連續？

若要偵測不連續，請找出 DiscontinuityCount > 0 的所有頻道資料項目。 對應的 ObservedTime 時間戳記表示發生不連續的時間。

### <a name="how-to-detect-timestamp-overlaps"></a>如何偵測時間戳記重疊？

若要偵測時間戳記重疊，請找出 OverlapCount > 0 的所有頻道資料項目。 對應的 ObservedTime 時間戳記表示發生時間戳記重疊的時間。

### <a name="how-to-find-streaming-request-failures-and-reasons"></a>如何找出串流要求失敗及原因？

若要找出串流要求失敗及原因，請找出 ResultCode 不等於 S_OK 的所有串流端點資料項目。 對應的 StatusCode 欄位表示要求失敗的原因。

### <a name="how-to-consume-data-with-external-tools"></a>如何透過外部工具使用資料？

使用下列工具可處理和視覺化遙測資料︰

- PowerBI
- Application Insights
- Azure 監視器 (先前稱為 Shoebox)
- AMS 即時儀表板
- Azure 入口網站 (擱置中版本)

### <a name="how-to-manage-data-retention"></a>如何管理資料保留？

遙測系統不提供資料保留管理或舊記錄自動刪除的功能。 因此，您需要從儲存體資料表手動管理和刪除舊記錄。 關於具體作法，您可以參考儲存體 SDK。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
