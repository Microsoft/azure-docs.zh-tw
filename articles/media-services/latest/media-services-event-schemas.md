---
title: Azure 媒體服務事件的 Azure 事件格線結構描述
description: 描述利用 Azure 事件格線提供給媒體服務事件的屬性
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: reference
ms.date: 08/17/2018
ms.author: juliako
ms.openlocfilehash: a6a6c459e170627d26aa1445f4f4dd193965fe70
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42140886"
---
# <a name="azure-event-grid-schemas-for-media-services-events"></a>Azure 媒體服務事件的 Azure 事件格線結構描述

本文提供媒體服務事件的結構描述和屬性。

如需範例指令碼和教學課程的清單，請參閱[媒體服務事件來源](../../event-grid/event-sources.md#azure-subscriptions)。

## <a name="available-event-types"></a>可用的事件類型

媒體服務會發出下列事件類型：

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.Media.JobStateChange| 作業的狀態有變更。 |
| Microsoft.Media.LiveEventConnectionRejected | 編碼器的連線嘗試遭到拒絕。 |
| Microsoft.Media.LiveEventEncoderConnected | 編碼器對即時事件建立連線。 |
| Microsoft.Media.LiveEventEncoderDisconnected | 編碼器中斷連線。 |
| Microsoft.Media.LiveEventIncomingDataChunkDropped | 媒體伺服器卸除資料區塊，原因是其來得太晚或有重疊的時間戳記 (新資料區塊的時間戳記小於先前資料區塊的結束時間)。 |
| Microsoft.Media.LiveEventIncomingStreamReceived | 媒體伺服器收到資料流或連線中每個資料軌的第一個資料區塊。 |
| Microsoft.Media.LiveEventIncomingStreamsOutOfSync | 媒體伺服器偵測到音訊和視訊資料流未同步。可作為警告，原因是使用者體驗可能不受影響。 |
| Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync | 媒體伺服器偵測到來自外部編碼器的兩個視訊資料流中，有任一個未同步。可作為警告，原因是使用者體驗可能不受影響。 |
| Microsoft.Media.LiveEventIngestHeartbeat | 當即時事件執行時，會針對每個資料軌每 20 秒發佈一次。 提供內嵌健康情況摘要。 |
| Microsoft.Media.LiveEventTrackDiscontinuityDetected | 媒體伺服器偵測到內送的資料軌發生中斷。 |

**即時**事件有兩種類別：資料流層級事件和資料軌層級事件。 

資料流層級事件會以每個資料流或連線為基礎來引發。 每個事件都有 `StreamId` 參數可識別連線或資料流。 每個資料流或連線都有一或多個不同類型的資料軌。 例如，來自編碼器的一個連線可能會有一個音訊資料軌和四個視訊資料軌。 資料流事件類型有：

* LiveEventConnectionRejected
* LiveEventEncoderConnected
* LiveEventEncoderDisconnected

資料軌層級事件會以每個資料軌為基礎來引發。資料軌事件類型有：

* LiveEventIncomingDataChunkDropped
* LiveEventIncomingStreamReceived
* LiveEventIncomingStreamsOutOfSync
* LiveEventIncomingVideoStreamsOutOfSync
* LiveEventIngestHeartbeat
* LiveEventTrackDiscontinuityDetected

## <a name="jobstatechange"></a>JobStateChange

下列範例顯示 **JobStateChange** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
    "eventType": "Microsoft.Media.JobStateChange",
    "eventTime": "2018-04-20T21:26:13.8978772",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "previousState": "Processing",
      "state": "Finished"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| previousState | 字串 | 工作在該事件之前的狀態。 |
| state | 字串 | 在此事件中，被通知的工作新狀態。 例如，「已排入佇列：作業正在等候資源」或「已排程：作業已準備好開始」。|

工作狀態可以是以下其中一個值：*已排入佇列*、*已排程*、*處理中*、*已完成*、*錯誤*、*已取消*、*取消中*

## <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected

下列範例顯示 **LiveEventConnectionRejected** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventConnectionRejected",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "b303db59-d5c1-47eb-927a-3650875fded1",
    "data": { 
      "StreamId":"Mystream1",
      "IngestUrl": "http://abc.ingest.isml",
      "EncoderIp": "118.238.251.xxx",
      "EncoderPort": 52859,
      "ResultCode": "MPE_INGEST_CODEC_NOT_SUPPORTED"
    },
    "dataVersion": "1.0"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| StreamId | 字串 | 資料流或連線的識別碼。 編碼器或客戶要負責在內嵌 URL 中新增此識別碼。 |  
| IngestUrl | 字串 | 即時事件所提供的內嵌 URL。 |  
| EncoderIp | 字串 | 編碼器的 IP。 |
| EncoderPort | 字串 | 此資料流來源編碼器的連接埠。 |
| ResultCode | 字串 | 連線遭到拒絕的原因。 結果碼列於下表。 |

結果碼如下：

| 結果碼 | 說明 |
| ----------- | ----------- |
| MPE_RTMP_APPID_AUTH_FAILURE | 不正確的內嵌 URL |
| MPE_INGEST_ENCODER_CONNECTION_DENIED | 編碼器 IP 不存在於所設定的 IP 允許清單中 |
| MPE_INGEST_RTMP_SETDATAFRAME_NOT_RECEIVED | 編碼器未傳送關於資料流的中繼資料。 |
| MPE_INGEST_CODEC_NOT_SUPPORTED | 不支援指定的轉碼器。 |
| MPE_INGEST_DESCRIPTION_INFO_NOT_RECEIVED | 在收到資料流的標頭之前就收到其片段。 |
| MPE_INGEST_MEDIA_QUALITIES_EXCEEDED | 指定的品質數目超過允許的上限。 |
| MPE_INGEST_BITRATE_AGGREGATED_EXCEEDED | 彙總的位元速率超過允許的上限。 |
| MPE_RTMP_FLV_TAG_TIMESTAMP_INVALID | 來自 RTMP 編碼器的視訊或音訊 FLVTag 時間戳記無效。 |

## <a name="liveeventencoderconnected"></a>LiveEventEncoderConnected

下列範例顯示 **LiveEventEncoderConnected** 事件的結構描述： 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderConnected",
    "eventTime": "2018-08-07T23:08:09.1710643",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| StreamId | 字串 | 資料流或連線的識別碼。 編碼器或客戶要負責在內嵌 URL 中提供此識別碼。 |
| IngestUrl | 字串 | 即時事件所提供的內嵌 URL。 |
| EncoderIp | 字串 | 編碼器的 IP。 |
| EncoderPort | 字串 | 此資料流來源編碼器的連接埠。 |

## <a name="liveeventencoderdisconnected"></a>LiveEventEncoderDisconnected

下列範例顯示 **LiveEventEncoderDisconnected** 事件的結構描述： 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderDisconnected",
    "eventTime": "2018-08-07T23:08:09.1710872",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "resultCode": "S_OK"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| StreamId | 字串 | 資料流或連線的識別碼。 編碼器或客戶要負責在內嵌 URL 中新增此識別碼。 |  
| IngestUrl | 字串 | 即時事件所提供的內嵌 URL。 |  
| EncoderIp | 字串 | 編碼器的 IP。 |
| EncoderPort | 字串 | 此資料流來源編碼器的連接埠。 |
| ResultCode | 字串 | 編碼器中斷連線的原因。 可能是正常中斷連線或錯誤所致。 結果碼列於下表。 |

錯誤結果碼如下：

| 結果碼 | 說明 |
| ----------- | ----------- |
| MPE_RTMP_SESSION_IDLE_TIMEOUT | RTMP 工作階段在閒置達允許時間限制後逾時。 |
| MPE_RTMP_FLV_TAG_TIMESTAMP_INVALID | 來自 RTMP 編碼器的視訊或音訊 FLVTag 時間戳記無效。 |
| MPE_CAPACITY_LIMIT_REACHED | 編碼器的資料傳送速度太快。 |
| 未知的錯誤碼 | 這些錯誤碼的範圍從記憶體錯誤到雜湊對應中有重複項目。 |

正常中斷連線的結果碼如下：

| 結果碼 | 說明 |
| ----------- | ----------- |
| S_OK | 編碼器已成功中斷連線。 |
| MPE_CLIENT_TERMINATED_SESSION | 編碼器已中斷連線 (RTMP)。 |
| MPE_CLIENT_DISCONNECTED | 編碼器已中斷連線 (FMP4)。 |
| MPI_REST_API_CHANNEL_RESET | 收到通道重設命令。 |
| MPI_REST_API_CHANNEL_STOP | 收到通道停止命令。 |
| MPI_REST_API_CHANNEL_STOP | 通道正在進行維護。 |
| MPI_STREAM_HIT_EOF | 編碼器傳送了 EOF 資料流。 |

## <a name="liveeventincomingdatachunkdropped"></a>LiveEventIncomingDataChunkDropped

下列範例顯示 **LiveEventIncomingDataChunkDropped** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingDataChunkDropped",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "03da9c10-fde7-48e1-80d8-49936f2c3e7d",
    "data": { 
      "TrackType": "Video",
      "TrackName": "Video",
      "Bitrate": 300000,
      "Timestamp": 36656620000,
      "Timescale": 10000000,
      "ResultCode": "FragmentDrop_OverlapTimestamp"
    },
    "dataVersion": "1.0"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| TrackType | 字串 | 資料軌的類型 (音訊/視訊)。 |
| TrackName | 字串 | 資料軌的名稱。 |
| Bitrate | integer | 資料軌的位元速率。 |
| Timestamp | 字串 | 資料區塊的卸除時間戳記。 |
| 時幅 | 字串 | 時間戳記的時幅。 |
| ResultCode | 字串 | 資料區塊卸除的原因。 **FragmentDrop_OverlapTimestamp** 或 **FragmentDrop_NonIncreasingTimestamp**。 |

## <a name="liveeventincomingstreamreceived"></a>LiveEventIncomingStreamReceived

下列範例顯示 **LiveEventIncomingStreamReceived** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamReceived",
    "eventTime": "2018-08-07T23:08:10.5069288Z",
    "id": "7f939a08-320c-47e7-8250-43dcfc04ab4d",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml/Streams(15864-stream0)15864-stream0",
      "trackType": "video",
      "trackName": "video",
      "bitrate": 2962000,
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "timestamp": "15336831655032322",
      "duration": "20000000",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| TrackType | 字串 | 資料軌的類型 (音訊/視訊)。 |
| TrackName | 字串 | 資料軌的名稱 (由編碼器提供，在 RTMP 的案例中，伺服器會以 TrackType_Bitrate 格式產生)。 |
| Bitrate | integer | 資料軌的位元速率。 |
| IngestUrl | 字串 | 即時事件所提供的內嵌 URL。 |
| EncoderIp | 字串  | 編碼器的 IP。 |
| EncoderPort | 字串 | 此資料流來源編碼器的連接埠。 |
| Timestamp | 字串 | 所收到資料區塊的第一個時間戳記。 |
| 時幅 | 字串 | 用來表示時間戳記的時幅。 |

## <a name="liveeventincomingstreamsoutofsync"></a>LiveEventIncomingStreamsOutOfSync

下列範例顯示 **LiveEventIncomingStreamsOutOfSync** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamsOutOfSync",
    "eventTime": "2018-08-10T02:26:20.6269183Z",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "minLastTimestamp": "319996",
      "typeOfStreamWithMinLastTimestamp": "Audio",
      "maxLastTimestamp": "366000",
      "typeOfStreamWithMaxLastTimestamp": "Video"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| MinLastTimestamp | 字串 | 所有資料軌 (音訊或視訊) 最後一個時間戳記之間的最小值。 |
| TypeOfTrackWithMinLastTimestamp | 字串 | 最後一個時間戳記為最小值的資料軌 (音訊或視訊) 類型。 |
| MaxLastTimestamp | 字串 | 所有資料軌 (音訊或視訊) 的所有時間戳記之間的最大值。 |
| TypeOfTrackWithMaxLastTimestamp | 字串 | 最後一個時間戳記為最大值的資料軌 (音訊或視訊) 類型。 |

## <a name="liveeventincomingvideostreamsoutofsync"></a>LiveEventIncomingVideoStreamsOutOfSync

下列範例顯示 **LiveEventIncomingVideoStreamsOutOfSync** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/LiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "6dd4d862-d442-40a0-b9f3-fc14bcf6d750",
    "data": {
      "FirstTimestamp": "2162058216",
      "FirstDuration": "2000",
      "SecondTimestamp": "2162057216",
      "SecondDuration": "2000"
    },
    "dataVersion": "1.0"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| FirstTimestamp | 字串 | 其中一個視訊類型資料軌/品質層級所收到的時間戳記。 |
| FirstDuration | 字串 | 具有第一個時間戳記的資料區塊持續時間。 |
| SecondTimestamp | 字串  | 其他某些視訊類型資料軌/品質層級所收到的時間戳記。 |
| SecondDuration | 字串 | 具有第二個時間戳記的資料區塊持續時間。 |

## <a name="liveeventingestheartbeat"></a>LiveEventIngestHeartbeat

下列範例顯示 **LiveEventIngestHeartbeat** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIngestHeartbeat",
    "eventTime": "2018-08-07T23:17:57.4610506",
    "id": "7f450938-491f-41e1-b06f-c6cd3965d786",
    "data": {
      "trackType": "audio",
      "trackName": "audio",
      "bitrate": 160000,
      "incomingBitrate": 155903,
      "lastTimestamp": "15336837535253637",
      "timescale": "10000000",
      "overlapCount": 0,
      "discontinuityCount": 0,
      "nonincreasingCount": 0,
      "unexpectedBitrate": false,
      "state": "Running",
      "healthy": true
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| TrackType | 字串 | 資料軌的類型 (音訊/視訊)。 |
| TrackName | 字串 | 資料軌的名稱 (由編碼器提供，在 RTMP 的案例中，伺服器會以 TrackType_Bitrate 格式產生)。 |
| Bitrate | integer | 資料軌的位元速率。 |
| IncomingBitrate | integer | 根據來自編碼器的資料區塊所計算出的位元速率。 |
| LastTimestamp | 字串 | 資料軌在過去 20 秒所收到的最新時間戳記。 |
| 時幅 | 字串 | 用來表示時間戳記的時幅。 |
| OverlapCount | integer | 在過去 20 秒有重疊時間戳記的資料區塊數目。 |
| DiscontinuityCount | integer | 在過去 20 秒所觀察到的中斷次數。 |
| NonIncreasingCount | integer | 在過去 20 秒所收到有過去時間戳記的資料區塊數目。 |
| UnexpectedBitrate | 布林 | 在過去 20 秒內，預期和實際的位元速率差異是否超過允許的限制。 只有當「IncomingBitrate >= 2* 位元速率」或「IncomingBitrate <= 位元速率/2」或「IncomingBitrate = 0」時，才為 true。 |
| State | 字串 | 即時事件的狀態。 |
| Healthy | 布林 | 根據計數和旗標來指出內嵌是否狀況良好。 如果 OverlapCount = 0 && DiscontinuityCount = 0 && NonIncreasingCount = 0 && UnexpectedBitrate = false，則健康情況為 true。 |

## <a name="liveeventtrackdiscontinuitydetected"></a>LiveEventTrackDiscontinuityDetected

下列範例顯示 **LiveEventTrackDiscontinuityDetected** 事件的結構描述： 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventTrackDiscontinuityDetected",
    "eventTime": "2018-08-07T23:18:06.1270405Z",
    "id": "5f4c510d-5be7-4bef-baf0-64b828be9c9b",
    "data": {
      "trackName": "video",
      "previousTimestamp": "15336837615032322",
      "trackType": "video",
      "bitrate": 2962000,
      "newTimestamp": "15336837619774273",
      "discontinuityGap": "575284",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| TrackType | 字串 | 資料軌的類型 (音訊/視訊)。 |
| TrackName | 字串 | 資料軌的名稱 (由編碼器提供，在 RTMP 的案例中，伺服器會以 TrackType_Bitrate 格式產生)。 |
| Bitrate | integer | 資料軌的位元速率。 |
| PreviousTimestamp | 字串 | 上一個片段的時間戳記。 |
| NewTimestamp | 字串 | 當前片段的時間戳記。 |
| DiscontinuityGap | 字串 | 上述兩個時間戳記之間的間距。 |
| 時幅 | 字串 | 用來表示時間戳記和中斷間距的時幅。 |

## <a name="common-event-properties"></a>常見的事件屬性

事件具有下列的最高層級資料：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 主題 | 字串 | EventGrid 主題。 此屬性有媒體服務帳戶的資源識別碼。 |
| 主旨 | 字串 | 媒體服務帳戶底下媒體服務通道的資源路徑。 串連主題和主旨即可獲得作業的資源識別碼。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 例如，"Microsoft.Media.JobStateChange"。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | 字串 | 事件的唯一識別碼。 |
| data | 物件 | 媒體服務事件資料。 |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

## <a name="next-steps"></a>後續步驟

[登記工作狀態變更事件](job-state-events-cli-how-to.md)