---
title: 概念、字詞和實體 - Azure 排程器 | Microsoft Docs
description: 了解 Azure 排程器中的概念、術語及實體階層，包括工作和工作集合。
services: scheduler
ms.service: scheduler
ms.suite: infrastructure-services
author: derek1ee
ms.author: deli
ms.reviewer: klam
ms.assetid: 3ef16fab-d18a-48ba-8e56-3f3e0a1bcb92
ms.topic: get-started-article
ms.date: 08/18/2016
ms.openlocfilehash: 07b7cce4b026464ba34296b54c4ae90d6d2b1afa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981156"
---
# <a name="concepts-terminology-and-entities-in-azure-scheduler"></a>Azure 排程器中的概念、術語及實體

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 會取代 Azure 排程器，Azure 排程器之後將無法使用。 若要排定作業，請[改為試用 Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md)。 

## <a name="entity-hierarchy"></a>實體階層

Azure 排程器 REST API 會公開並使用這些主要實體或資源：

| 實體 | 說明 |
|--------|-------------|
| **作業** | 定義單一週期性動作，以及簡單或複雜的執行策略。 動作可能包括 HTTP、儲存體佇列、服務匯流排佇列或服務匯流排主題要求。 | 
| **工作集合** | 包含作業群組，並維護集合中作業所共用的設定、配額及節流。 身為 Azure 訂用帳戶擁有者，您可以建立作業集合，並根據作業的使用方式或應用程式界限將作業群組在一起。 作業集合具有下列屬性： <p>- 受限於一個區域。 <br>- 可讓您強制執行配額，以限制集合中所有作業的使用方式。 <br>- 配額包含 MaxJobs 和 MaxRecurrence。 | 
| **工作歷程記錄** | 描述作業執行的詳細資料，例如，狀態和任何回應詳細資料。 |
||| 

## <a name="entity-management"></a>實體管理

概括來說，排程器 REST API 會公開用來管理實體的以下作業。

### <a name="job-management"></a>作業管理

支援建立和編輯作業的作業。 所有作業都必須屬於現有作業集合，因此沒有隱含的建立作業。 如需詳細資訊，請參閱[排程器 REST API - 作業](https://docs.microsoft.com/rest/api/scheduler/jobs)。 以下是這些作業的 URI 位址：

`https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}/jobs/{jobName}`

### <a name="job-collection-management"></a>作業集合管理

支援建立和編輯作業及作業集合的作業，這會對應到配額和共用設定。 例如，配額會指定最大作業數目和最小週期間隔。 如需詳細資訊，請參閱[排程器 REST API - 作業集合](https://docs.microsoft.com/rest/api/scheduler/jobcollections)。 以下是這些作業的 URI 位址：

`https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}`

### <a name="job-history-management"></a>作業記錄管理

支援使用 GET 作業擷取 60 天的作業執行記錄，例如作業經歷時間和作業執行結果。 包含根據狀況和狀態篩選的查詢字串參數支援。 如需詳細資訊，請參閱[排程器 REST API - 作業 - 列出作業記錄](https://docs.microsoft.com/rest/api/scheduler/jobs/listjobhistory)。 以下是此作業的 URI 位址：

`https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}/jobs/{jobName}/history`

## <a name="job-types"></a>工作類型

Azure 排程器支援多個作業類型： 

* HTTP 作業 (包括支援 SSL 的 HTTPS 作業)，如果您有現有服務或工作負載的端點，適用此類型
* 儲存體佇列作業，適用於使用儲存體佇列的工作負載，例如將訊息張貼到儲存體佇列
* 服務匯流排佇列作業，適用於使用服務匯流排佇列的工作負載
* 服務匯流排主題佇列作業，適用於使用服務匯流排主題的工作負載

## <a name="job-definition"></a>工作定義

概括來說，排程器作業包含下列基本部分：

* 要在工作計時器啟動時執行的動作
* 選用：執行作業的時間
* 選用：重複作業的時機和頻率
* 選用：主要動作失敗時要執行的錯誤動作

作業也包含系統提供的資料，例如已排定的下一次作業執行時間。 作業的程式碼定義是採用 JavaScript 物件標記法 (JSON) 格式的物件，其中包含以下元素：

| 元素 | 必要 | 說明 | 
|---------|----------|-------------| 
| [**startTime**](#start-time) | 否 | 包含時區位移的作業開始時間，格式為 [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) | 
| [**action**](#action) | 是 | 主要動作的詳細資料，其中可包括 **errorAction** 物件 | 
| [**errorAction**](#error-action) | 否 | 次要動作 (在主要動作失敗時執行) 的詳細資料 |
| [**recurrence**](#recurrence) | 否 | 週期性作業的詳細資料，例如頻率和間隔 | 
| [**retryPolicy**](#retry-policy) | 否 | 重試動作的頻率詳細資料 | 
| [**state**](#state) | 是 | 作業目前狀態的詳細資料 |
| [**status**](#status) | 是 | 作業目前狀態的詳細資料，而狀態會由服務控制 |
||||

以下範例說明 HTTP 動作的完整作業定義，其中包含更完整的元素詳細資料，將在後面幾節中加以描述： 

```json
"properties": {
   "startTime": "2012-08-04T00:00Z",
   "action": {
      "type": "Http",
      "request": {
         "uri": "http://contoso.com/some-method", 
         "method": "PUT",          
         "body": "Posting from a timer",
         "headers": {
            "Content-Type": "application/json"
         },
         "retryPolicy": { 
             "retryType": "None" 
         },
      },
      "errorAction": {
         "type": "Http",
         "request": {
            "uri": "http://contoso.com/notifyError",
            "method": "POST"
         }
      }
   },
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "weekDays": ["Monday", "Wednesday", "Friday"],
         "hours": [10, 22]
      },
      "count": 10,
      "endTime": "2012-11-04"
   },
   "state": "Disabled",
   "status": {
      "lastExecutionTime": "2007-03-01T13:00:00Z",
      "nextExecutionTime": "2007-03-01T14:00:00Z ",
      "executionCount": 3,
      "failureCount": 0,
      "faultedCount": 0
   }
}
```

<a name="start-time"></a>

## <a name="starttime"></a>startTime

在 **startTime** 物件中，您可以指定 [ISO 8601 格式](http://en.wikipedia.org/wiki/ISO_8601)的開始時間和時區位移。

<a name="action"></a>

## <a name="action"></a>動作

排程器作業會根據指定的排程來執行主要**動作**。 排程器支援 HTTP、儲存體佇列、服務匯流排佇列和服務匯流排主題動作。 如果主要**動作**失敗，排程器可以執行次要的 [**errorAction**](#errorAction) 來處理錯誤。 **action** 物件會描述以下元素：

* 動作的服務類型
* 動作的詳細資料
* 替代的 **errorAction**

上述範例會說明 HTTP 動作。 以下是儲存體佇列動作的範例：

```json
"action": {
   "type": "storageQueue",
   "queueMessage": {
      "storageAccount": "myStorageAccount",  
      "queueName": "myqueue",                
      "sasToken": "TOKEN",                   
      "message": "My message body"
    }
}
```

以下是服務匯流排佇列動作的範例：

```json
"action": {
   "type": "serviceBusQueue",
   "serviceBusQueueMessage": {
      "queueName": "q1",  
      "namespace": "mySBNamespace",
      "transportType": "netMessaging", // Either netMessaging or AMQP
      "authentication": {  
         "sasKeyName": "QPolicy",
         "type": "sharedAccessKey"
      },
      "message": "Some message",  
      "brokeredMessageProperties": {},
      "customMessageProperties": {
         "appname": "FromScheduler"
      }
   }
},
```

以下是服務匯流排主題動作的範例：

```json
"action": {
   "type": "serviceBusTopic",
   "serviceBusTopicMessage": {
      "topicPath": "t1",  
      "namespace": "mySBNamespace",
      "transportType": "netMessaging", // Either netMessaging or AMQP
      "authentication": {
         "sasKeyName": "QPolicy",
         "type": "sharedAccessKey"
      },
      "message": "Some message",
      "brokeredMessageProperties": {},
      "customMessageProperties": {
         "appname": "FromScheduler"
      }
   }
},
```

如需共用存取簽章 (SAS) 權杖的詳細資訊，請參閱[使用共用存取簽章進行授權](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

<a name="error-action"></a>

## <a name="erroraction"></a>errorAction

如果作業的主要**動作**失敗，排程器可以執行 **errorAction** 來處理錯誤。 在主要**動作**中，您可以指定 **errorAction** 物件，讓排程器可以呼叫錯誤處理端點或傳送使用者通知。 

例如，如果主要端點上發生災害，您可以使用 **errorAction** 來呼叫次要端點，或通知錯誤處理端點。 

如同主要**動作**，您可以讓錯誤動作以其他動作為基礎，使用簡單或複合邏輯。 

<a name="recurrence"></a>

## <a name="recurrence"></a>週期

如果作業的 JSON 定義中包含 **recurrence** 物件，則作業會重複執行，例如：

```json
"recurrence": {
   "frequency": "Week",
   "interval": 1,
   "schedule": {
      "hours": [10, 22],
      "minutes": [0, 30],
      "weekDays": ["Monday", "Wednesday", "Friday"]
   },
   "count": 10,
   "endTime": "2012-11-04"
},
```

| 屬性 | 必要 | 值 | 說明 | 
|----------|----------|-------|-------------| 
| **frequency** | 在使用 **recurrence** 時是必要的 | "Minute"、"Hour"、"Day"、"Week"、"Month"、"Year" | 發生次數之間的時間單位 | 
| **interval** | 否 | 1 到 1000 (含) | 正整數，根據 **frequency** 來決定每次發生作業的間隔時間單位數 | 
| **schedule** | 否 | 視情況而異 | 更複雜且進階的排程詳細資料。 請參考 **hours**、**minutes****weekDays**、**months** 和 **monthDays** | 
| **hours** | 否 | 1 到 24 | 包含小時標記的陣列，表示要執行作業的時間 | 
| **minutes** | 否 | 1 到 24 | 包含分鐘標記的陣列，表示要執行作業的時間 | 
| **months** | 否 | 1 到 12 | 包含月份的陣列，表示要執行作業的時間 | 
| **monthDays** | 否 | 視情況而異 | 包含月份中某幾天的陣列，表示要執行作業的時間 | 
| **weekDays** | 否 | "Monday"、"Tuesday"、"Wednesday"、"Thursday"、"Friday"、"Saturday"、"Sunday" | 包含一週中某幾天的陣列，表示要執行作業的時間 | 
| **count** | 否 | <無> | 循環次數。 預設為無限制地重複。 您無法同時使用 **count** 和 **endTime**，但會接受先完成的規則。 | 
| **endTime** | 否 | <無> | 停止循環的日期和時間。 預設為無限制地重複。 您無法同時使用 **count** 和 **endTime**，但會接受先完成的規則。 | 
||||

如需有關這些元素的詳細資訊，請參閱[建置複雜的排程和進階週期](../scheduler/scheduler-advanced-complexity.md)。

<a name="retry-policy"></a>

## <a name="retrypolicy"></a>RetryPolicy

針對排程器作業可能會失敗的情況，您可以設定重試原則來決定排程器是否要重試該作業，以及如何重試。 根據預設，排程器會以 30 秒的間隔重新嘗試執行四次。 您可以讓此原則更積極或更不積極，比方說，讓此原則每天重試動作兩次：

```json
"retryPolicy": { 
   "retryType": "Fixed",
   "retryInterval": "PT1D",
   "retryCount": 2
},
```

| 屬性 | 必要 | 值 | 說明 | 
|----------|----------|-------|-------------| 
| **retryType** | 是 | **固定**、**無** | 決定要指定重試原則 (**固定**) 或不要指定重試原則 (**無**)。 | 
| **retryInterval** | 否 | PT30S | 指定重試嘗試之間的間隔和頻率，格式為 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)。 最小值為 15 秒，而最大值為 18 個月。 | 
| **retryCount** | 否 | 4 | 指定嘗試重試的次數。 最大值為 20。 | 
||||

如需詳細資訊，請參閱[高可用性和可靠性](../scheduler/scheduler-high-availability-reliability.md)。

<a name="status"></a>

## <a name="state"></a>state

作業的狀態可能是 **Enabled (已啟用)**、**Disabled (已停用)**、**Completed (已完成)** 或 **Faulted (發生錯誤)**，例如： 

`"state": "Disabled"`

若要將作業變更為 **Enabled** 或 **Disabled** 狀態，您可以在這些作業上使用 PUT 或 PATCH 作業。
不過，如果作業狀態為 **Completed** 或 **Faulted**，則無法更新狀態，但您可以在作業上執行 DELETE 作業。 排程器會在 60 天後刪除已完成和發生錯誤的作業。 

<a name="status"></a>

## <a name="status"></a>status

作業啟動之後，排程器透過 **status** 物件傳回作業狀態的相關資訊，該物件只能由排程器控制。 不過，您可以在 **job** 物件中找到 **status** 物件。 以下是作業狀態包含的資訊：

* 上一次執行的時間 (如果有的話)
* 進行中工作的下次排定執行時間
* 作業執行的數目
* 失敗次數 (如果有的話)
* 錯誤次數 (如果有的話)

例如︰

```json
"status": {
   "lastExecutionTime": "2007-03-01T13:00:00Z",
   "nextExecutionTime": "2007-03-01T14:00:00Z ",
   "executionCount": 3,
   "failureCount": 0,
   "faultedCount": 0
}
```

## <a name="see-also"></a>另請參閱

* [什麼是 Azure 排程器？](scheduler-intro.md)
* [概念、術語及實體階層](scheduler-concepts-terms.md)
* [建置複雜的排程和進階週期](scheduler-advanced-complexity.md)
* [限制、配額、預設值及錯誤碼](scheduler-limits-defaults-errors.md)
* [Azure 排程器 REST API 參考](https://docs.microsoft.com/rest/api/schedule)
* [Azure 排程器 PowerShell Cmdlet 參考](scheduler-powershell-reference.md)
