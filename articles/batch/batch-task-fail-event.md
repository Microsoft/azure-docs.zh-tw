---
title: Azure Batch 工作失敗事件 | Microsoft Docs
description: Batch 工作失敗事件的參考。
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/20/2017
ms.author: danlep
ms.openlocfilehash: c4636ebbfb6737be07d01a3861ec6b9da66ff966
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30313091"
---
# <a name="task-fail-event"></a>工作失敗事件

 當工作未成功完成時，就會發出此事件。 目前所有非零的結束代碼皆視為失敗。 除了工作完成事件外，還會發出此事件，此事件可用於偵測工作何時失敗。


 下列範例顯示工作失敗事件內文。

```
{
    "jobId": "job-0000000001",
    "id": "task-5",
    "taskType": "User",
    "systemTaskVersion": 0,
    "nodeInfo": {
        "poolId": "pool-001",
        "nodeId": "tvm-257509324_1-20160908t162728z"
    },
    "multiInstanceSettings": {
        "numberOfInstances": 1
    },
    "constraints": {
        "maxTaskRetryCount": 2
    },
    "executionInfo": {
        "startTime": "2016-09-08T16:32:23.799Z",
        "endTime": "2016-09-08T16:34:00.666Z",
        "exitCode": 1,
        "retryCount": 2,
        "requeueCount": 0
    }
}
```

|元素名稱|類型|注意|
|------------------|----------|-----------|
|jobId|字串|包含工作的作業識別碼。|
|id|字串|工作識別碼。|
|taskType|字串|工作類型。 可以是 'JobManager'，表示這是作業管理員工作，或是 'User'，表示不是作業管理員工作。 對於作業準備工作、作業發行工作或開始工作。不會發出此事件。|
|systemTaskVersion|Int32|這是作業上的內部重試計數器。 Batch 服務可在內部重試工作，以處理暫時性問題。 這些問題包含內部排程錯誤，或嘗試從不正常狀態的計算節點復原。|
|[nodeInfo](#nodeInfo)|複雜類型|包含工作執行所在計算節點的相關資訊。|
|[multiInstanceSettings](#multiInstanceSettings)|複雜類型|指定工作為需要多個計算節點的多重執行個體工作。  請參閱 [multiInstanceSettings](https://docs.microsoft.com/rest/api/batchservice/get-information-about-a-task) (英文) 以取得詳細資訊。|
|[constraints](#constraints)|複雜類型|套用至此工作的執行限制。|
|[executionInfo](#executionInfo)|複雜類型|包含執行工作的相關資訊。|

###  <a name="nodeInfo"></a> nodeInfo

|元素名稱|類型|注意|
|------------------|----------|-----------|
|poolId|字串|工作執行所在集區的識別碼。|
|nodeId|字串|工作執行所在節點的識別碼。|

###  <a name="multiInstanceSettings"></a> multiInstanceSettings

|元素名稱|類型|注意|
|------------------|----------|-----------|
|numberOfInstances|Int32|工作需要的計算節點數目。|

###  <a name="constraints"></a> constraints

|元素名稱|類型|注意|
|------------------|----------|-----------|
|maxTaskRetryCount|Int32|工作重試次數上限。 如果工作的結束代碼不是零，Batch 服務會重試工作。<br /><br /> 請注意，這個值會特別控制重試次數。 Batch 服務會嘗試工作一次，然後可一直重試直到達此限制。 例如，如果重試計數上限為 3，則 Batch 可嘗試工作最多 4 次 (一次首次嘗試，3 次重試)。<br /><br /> 如果重試計數上限為 0，則 Batch 服務不會重試工作。<br /><br /> 如果重試計數上限為 -1，則 Batch 服務會無限制地重試工作。<br /><br /> 預設值為 0 (不重試)。|


###  <a name="executionInfo"></a> executionInfo

|元素名稱|類型|注意|
|------------------|----------|-----------|
|startTime|Datetime|工作開始執行的時間。 「執行」與 **running** 狀態對應，因此如果工作會指定資源檔或應用程式套件，則開始時間會反映工作開始下載或部署下載項目的時間。  如果已重新啟動或重試工作，則這是最近一次工作開始執行的時間。|
|EndTime|Datetime|工作完成的時間。|
|exitCode|Int32|工作的結束代碼。|
|retryCount|Int32|Batch 服務已重試工作的次數。 如果工作結束時的結束代碼不是零，便會重試工作，直到次數達指定的 MaxTaskRetryCount。|
|requeueCount|Int32|Batch 服務因為使用者要求而將工作重新排入佇列的次數。<br /><br /> 當使用者將節點從集區中移除 (透過調整集區大小或將集區縮小)，或當作業正停用時，使用者可指定將節點上的執行中工作重新排入佇列以執行。 此計數會追蹤因為這些理由而將工作重新排入佇列的次數。|
