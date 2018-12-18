---
title: 使用析診斷記錄針對 Azure 串流分進行疑難排解
description: 本文說明如何分析 Azure 串流分析中的診斷記錄。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/20/2017
ms.openlocfilehash: 9001a2962806ee3e691fa448dde162d12c6ecdd2
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30905856"
---
# <a name="troubleshoot-azure-stream-analytics-by-using-diagnostics-logs"></a>使用析診斷記錄對 Azure 串流分進行疑難排解

有時候，Azure 串流分析作業會非預期地停止處理。 請務必要設法解決這類事件的問題。 事件發生的原因可能是非預期的查詢結果、裝置的連線狀況，或未預期的服務中斷。 串流分析中的診斷記錄可協助您在事件發生當下找出問題原因，並減少復原時間。

## <a name="log-types"></a>記錄類型

串流分析提供開兩種記錄類型： 
* [活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) (一律開啟)。 活動記錄可讓您對所執行作業有深入的了解。
* [診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) (可設定)。 診斷記錄可讓您更深入地了所有與作業相關的發生事件。 診斷記錄會在作業建立時開始執行，並在作業刪除時結束。 這些記錄涵蓋作業更新時與作業執行時的情形。

> [!NOTE]
> 您可以使用 Azure 儲存體、Azure 事件中樞和 Azure Log Analytics 等服務來分析不一致的資料。 這些服務將會依據各自的計價模式向您收取費用。
>

## <a name="turn-on-diagnostics-logs"></a>開啟診斷記錄

診斷記錄預設為 [關閉]。 若要開啟診斷記錄，請完成下列步驟︰

1.  登入 Azure 入口網站並前往串流作業刀鋒視窗。 在 [監視] 下，選取 [診斷記錄]。

    ![瀏覽到診斷記錄的刀鋒視窗](./media/stream-analytics-job-diagnostic-logs/image1.png)  

2.  選取 [開啟診斷]。

    ![開啟診斷記錄](./media/stream-analytics-job-diagnostic-logs/image2.png)

3.  在 [診斷設定] 頁面的 [狀態]  上，選取 [開啟]。

    ![變更診斷記錄的狀態](./media/stream-analytics-job-diagnostic-logs/image3.png)

4.  設定封存目標 (儲存體帳戶、事件中樞、Log Analytics)。 然後選取您要收集的記錄類別 (執行、編寫)。 

5.  儲存新的診斷組態。

設定診斷大約需要 10 分鐘才會生效。 之後，記錄會開始出現在設定的封存目標中 (您可以在 [診斷記錄]頁面中看到這些記錄)：

![瀏覽到診斷記錄的刀鋒視窗 - 封存目標](./media/stream-analytics-job-diagnostic-logs/image4.png)

如需有關診斷設定的詳細資訊，請參閱[收集並取用來自 Azure 資源的診斷資料](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)。

## <a name="diagnostics-log-categories"></a>診斷記錄類別

我們目前會擷取兩種診斷記錄︰

* **編寫**。 擷取作業編寫相關記錄：作業建立、新增及刪除輸入與輸出、新增及更新查詢、開始及停止作業。
* **執行**。 擷取作業執行期間發生的事件︰
    * 連線錯誤
    * 資料處理錯誤，包括：
        * 不符合查詢定義的事件 (不相符的欄位類型與值或遺漏欄位等)
        * 運算式評估錯誤
    * 其他事件和錯誤

## <a name="diagnostics-logs-schema"></a>診斷記錄結構描述

所有記錄會儲存為 JSON 格式。 每個項目皆包含下列常見的字串欄位︰

Name | 說明
------- | -------
分析 | 記錄的時間戳記 (UTC 時間)。
ResourceId | 作業執行資源的識別碼 (大寫)。 其中包含訂用帳戶識別碼、資源群組，以及作業名稱。 例如，**/SUBSCRIPTIONS/6503D296-DAC1-4449-9B03-609A1F4A1C87/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.STREAMANALYTICS/STREAMINGJOBS/MYSTREAMINGJOB**。
category | 記錄類別 (**執行**或**編寫**)。
operationName | 記錄的作業名稱。 例如，**傳送事件︰SQL 輸出將失敗寫入 mysqloutput**。
status | 作業的狀態。 例如，**失敗**或**成功**。
層級 | 記錄層級。 例如，**錯誤**、**警告**或**資訊**。
properties | 記錄項目特定詳細資料 (序列化為 JSON 字串)。 如需詳細資訊，請參閱下列幾節。

### <a name="execution-log-properties-schema"></a>執行記錄屬性結構描述

執行記錄包含在執行串流分析作業期間所發生事件的資訊。 屬性結構描述會根據事件類型而有所不同。 我們目前有下列類型的執行記錄︰

### <a name="data-errors"></a>資料錯誤

作業處理資料時發生的任何錯誤皆包含於此類記錄中。 這些記錄最常於資料讀取、序列化和寫入作業時建立。 這些記錄不包含連線錯誤。 連線錯誤視為一般事件。

Name | 說明
------- | -------
來源 | 發生錯誤的作業輸入或輸出名稱。
訊息 | 與錯誤相關的訊息。
類型 | 錯誤類型。 例如，**DataConversionError**、**CsvParserError** 或 **ServiceBusPropertyColumnMissingError**。
資料 | 包含有助於正確找到錯誤來源的資料。 視其大小，資料可能會遭到截斷。

資料錯誤會隨 **operationName** 值的不同而有下列結構描述：
* **序列化事件**。 序列化事件會在事件讀取作業期間發生。 當輸入上的資料不符合查詢結構描述時就會發生這類事件，發生原因可能是下列其中一個：
    * *事件 (還原) 序列化期間類型不相符*：找出造成錯誤的欄位。
    * *無法讀取事件，序列化無效*：針對發生錯誤的輸入資料位置列出資訊。 包含 Blob 輸入的 Blob 名稱、位移和資料範例。
* **傳送事件**。 傳送事件會發生在寫入作業期間。 這些事件會識別造成錯誤的串流事件。

### <a name="generic-events"></a>一般事件

一般事件涵蓋所有其他事件。

Name | 說明
-------- | --------
Error | (選用) 錯誤資訊。 這通常是例外狀況資訊 (如果有的話)。
訊息| 記錄訊息。
類型 | 訊息類型。 對應至錯誤的內部分類。 例如，**JobValidationError**或 **BlobOutputAdapterInitializationFailure**。
相互關連識別碼 | 唯一識別作業執行的 [GUID (英文)](https://en.wikipedia.org/wiki/Universally_unique_identifier)。 從作業開始直到作業停止的所有執行記錄項目皆具有同一個**相互關聯識別碼**值。

## <a name="next-steps"></a>後續步驟

* [串流分析介紹](stream-analytics-introduction.md)
* [開始使用串流分析](stream-analytics-real-time-fraud-detection.md)
* [調整串流分析作業](stream-analytics-scale-jobs.md)
* [串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
