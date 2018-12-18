---
title: 擷取 Azure 匯入/匯出作業的狀態資訊 | Microsoft Docs
description: 了解如何取得 Microsoft Azure 匯入/匯出服務作業的狀態資訊。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 12/16/2016
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 76b2f73442261da4c791aaae8532d7a0dbbb6d95
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39520601"
---
# <a name="retrieving-state-information-for-an-importexport-job"></a>擷取匯入/匯出作業的狀態資訊
您可以呼叫 [Get Job](/rest/api/storageimportexport/jobs#Jobs_Get) 作業來擷取關於匯入和匯出作業的資訊。 傳回的資訊包括︰

-   作業的目前狀態。

-   每項作業已完成的近似百分比。

-   每個磁碟機的目前狀態。

-   Blob 的 URI，其中包含錯誤記錄檔和詳細資訊記錄的資訊 (如果啟用)。

下列各節將說明 `Get Job` 作業所傳回的資訊。

## <a name="job-states"></a>作業狀態
下表和狀態圖表說明作業在其生命週期中轉換的狀態。 可藉由呼叫 `Get Job` 作業來判斷作業的目前狀態。

![JobStates](./media/storage-import-export-retrieving-state-info-for-a-job/JobStates.png "JobStates")

下表說明作業可能經歷的各個狀態。

|作業狀態|說明|
|---------------|-----------------|
|`Creating`|呼叫 Put Job 作業之後，會建立一個作業，且其狀態會設為 `Creating`。 工作處於 `Creating` 狀態時，匯入/匯出服務會假設磁碟機尚未運送到資料中心。 工作最多可保持 `Creating` 狀態兩週，超過後會由服務自動刪除。<br /><br /> 如果您在作業處於時 `Creating` 狀態時呼叫 Update Job 屬性作業，作業會維持在 `Creating` 狀態，且逾時間隔會重設為 2 週。|
|`Shipping`|寄送包裹之後，您應該呼叫 Update Job 屬性作業將作業狀態更新為 `Shipping`。 只有當作業的 `DeliveryPackage` (貨運公司和追蹤號碼) 和 `ReturnAddress` 屬性都已設定時，才可以設定寄送狀態。<br /><br /> 作業最多將保持「運送中」狀態兩週。 如果已超過兩週，但未收到磁碟機，將會通知匯入/匯出服務操作員。|
|`Received`|資料中心收到所有磁碟機之後，作業狀態會設為「已收到」狀態。|
|`Transferring`|資料中心收到磁碟機，且至少已開始處理一個磁碟機之後，作業狀態將會設定為 `Transferring` 狀態。 如需詳細資訊，請參閱以下的 `Drive States` 一節。|
|`Packaging`|所有磁碟機處理完成之後，直到磁碟機寄回給客戶為止，工作將會設為 `Packaging` 狀態。|
|`Completed`|所有磁碟機都寄回給客戶之後，如果工作已完成且沒有錯誤，工作將會設為 `Completed` 狀態。 `Completed` 狀態持續 90 天後，將會自動刪除工作。|
|`Closed`|所有磁碟機都寄回給客戶之後，如果工作處理期間發生任何錯誤，工作將會設為 `Closed` 狀態。 `Closed` 狀態持續 90 天後，將會自動刪除工作。|

您只有在特定狀態才能取消作業。 已取消的作業會略過資料複製步驟，否則它會依照相同的狀態轉換為未取消的作業。

下表描述每個作業狀態可能發生的錯誤，以及在錯誤發生時對作業的影響。

|作業狀態|Event|解析 / 後續步驟|
|---------------|-----------|------------------------------|
|`Creating or Undefined`|作業的一或多個磁碟機已送達，但作業並非 `Shipping` 狀態或服務中的作業沒有記錄。|匯入/匯出服務作業小組會嘗試連絡客戶，以使用繼續進行作業的必要資訊建立或更新作業。<br /><br /> 如果作業小組無法在兩週內與客戶取得聯繫，作業小組會嘗試寄回磁碟機。<br /><br /> 無法寄回磁碟機且無法聯絡客戶的情況下，磁碟機將會在 90 天內安全地銷毀。<br /><br /> 請注意，作業在其狀態更新為 `Shipping` 之前無法處理。|
|`Shipping`|作業的包裹超過兩週未抵達。|作業小組將會通知遺失包裹的客戶。 根據客戶的回應，作業小組會延長間隔以等候包裹抵達或取消作業。<br /><br /> 在無法連絡客戶或客戶沒有在 30 天內回應的情況下，作業小組將採取動作，直接將作業從 `Shipping` 狀態移至 `Closed` 狀態。|
|`Completed/Closed`|磁碟機從未到達退貨地址，或者出貨中損毀 (僅適用於匯出工作)。|如果磁碟機未到達退貨地址，客戶應該先呼叫 Get Job 作業，或檢查入口網站中的作業狀態，以確保該磁碟機已寄達。 如果磁碟機已寄出，則客戶應該連絡貨運公司，嘗試找到磁碟機。<br /><br /> 如果磁碟機在寄送過程中損毀，客戶可能會要求其他匯出作業，或下載遺漏的 blob。|
|`Transferring/Packaging`|作業的退貨地址不正確或遺失。|作業小組將連絡作業的連絡人員才能取得正確的地址。<br /><br /> 無法聯絡客戶的情況下，磁碟機將會在 90 天內安全地銷毀。|
|`Creating / Shipping/ Transferring`|寄送包裹中包含未顯示於要匯入之磁碟機清單中的磁碟機。|將不會處理額外的磁碟機，且在作業完成時會寄回給客戶。|

## <a name="drive-states"></a>磁碟機狀態
下表和下圖說明個別磁碟機在匯入或匯出工作的轉換過程中的生命週期。 您可以呼叫 `Get Job` 作業並檢查 `DriveList` 屬性的 `State` 元素來擷取目前的磁碟機狀態。

![DriveStates](./media/storage-import-export-retrieving-state-info-for-a-job/DriveStates.png "DriveStates")

下表說明磁碟機可能經歷的各個狀態。

|磁碟機狀態|說明|
|-----------------|-----------------|
|`Specified`|針對匯入作業，使用 Put Job 作業建立作業時，磁碟機的初始狀態會是 `Specified` 狀態。 針對匯出工作，因為建立工作時未指定任何磁碟機，初始磁碟機狀態會是 `Received` 狀態。|
|`Received`|當匯入/匯出服務操作員已處理從貨運公司收到要進行匯入工作的磁碟機後，磁碟機會轉換成 `Received` 狀態。 針對匯出工作，初始磁碟機狀態會是 `Received`狀態。|
|`NeverReceived`|當工作的包裹抵達但包裹未包含磁碟機時，磁碟機會變成 `NeverReceived` 狀態。 如果自服務收到出貨資訊起已經過兩週，包裹卻仍未抵達資料中心，磁碟機也會變成此狀態。|
|`Transferring`|當服務開始將磁碟機中的資料傳輸到 Windows Azure 儲存體時，磁碟機將變成 `Transferring` 狀態。|
|`Completed`|當服務已順利傳輸所有資料而未產生錯誤時，磁碟機會變成 `Completed` 狀態。|
|`CompletedMoreInfo`|服務複製磁碟機的資料時若發生一些問題，磁碟機會變成 `CompletedMoreInfo` 狀態。 資訊可能包含錯誤、警告或有關覆寫 blob 的資訊訊息。|
|`ShippedBack`|當磁碟機從資料中心運回原地址時，磁碟機會變成 `ShippedBack` 狀態。|

下表說明磁碟機失敗狀態，以及針對每個狀態採取的動作。

|磁碟機狀態|Event|解析 / 後續步驟|
|-----------------|-----------|-----------------------------|
|`NeverReceived`|標示為 `NeverReceived` (因為在作業的出貨部分中未收到它) 的磁碟機會在其他出貨中寄出。|作業小組會將磁碟機移為 `Received` 狀態。|
|`N/A`|不屬於任何作業的磁碟機會以另一個作業的一部分送達資料中心。|磁碟機會標示為額外的磁碟機，並會在與原始包裹相關聯的工作完成時寄回給客戶。|

## <a name="faulted-states"></a>發生錯誤的狀態
當作業或磁碟機無法透過其預期的生命週期正常完成時，作業或磁碟機會移為 `Faulted` 狀態。 此時，作業小組會以電子郵件或電話連絡客戶。 當問題解決後，發生錯誤的作業或磁碟機將會移出 `Faulted` 狀態並移為適當的狀態。

## <a name="next-steps"></a>後續步驟

* [使用匯入/匯出服務 REST API](storage-import-export-using-the-rest-api.md)
