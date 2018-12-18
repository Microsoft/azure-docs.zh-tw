---
title: 了解 Azure 串流分析作業的相容性層級
description: 了解如何為 Azure 串流分析作業設定相容性層級，並了解最新相容性層級中的重大變更
services: stream-analytics
author: jasonwhowell
ms.author: jasonh
manager: kfile
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 01/03/2018
ms.openlocfilehash: 136b21f026d208c09b50dfa8601de692e518774e
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699105"
---
# <a name="compatibility-level-for-azure-stream-analytics-jobs"></a>Azure 串流分析作業的相容性層級
 
相容性層級是指 Azure 串流分析服務的版本專屬行為。 Azure 串流分析是受控服務，會定期更新功能及改善效能。 通常更新會自動提供給終端使用者。 不過，某些新功能可能會引進重大變更，例如現有作業的行為變更、從這些作業取用資料的流程變更等等。相容性層級用來代表串流分析中引入的重大變更。 重大變更一律會隨新的相容性層級引入。 

相容性層級可確保現有作業執行時不會發生任何失敗。 在建立新的串流分析作業時，最佳做法是使用提供給您的最新相容性層級來建立。 
 
## <a name="set-a-compatibility-level"></a>設定相容性層級 

相容性層級可控制串流分析作業的執行階段行為。 您可以使用入口網站或使用[建立作業 REST API 呼叫](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-job)來設定串流分析作業的相容性層級。 Azure 串流分析目前支援兩個相容性層級 - "1.0" 和 "1.1"。 根據預設，相容性層級會設定為 "1.0"，其是在 Azure 串流分析正式運作期間所引入。 若要更新預設值，瀏覽至您現有的串流分析作業 > 選取 [設定] 區段中的 [相容性層級] 選項，然後變更值。 

請確定在更新相容性層級之前停止作業。 如果您的作業處於執行中狀態，則無法更新相容性層級。 

![入口網站中的相容性層級](media\stream-analytics-compatibility-level/image1.png)

 
更新相容性層級時，T-SQL 編譯器會以對應至所選相容性層級的語法來驗證作業。 

## <a name="major-changes-in-the-latest-compatibility-level-11"></a>最新相容性層級 (1.1) 的重大變更

相容性層級 1.1 中引入了下列重大變更：

* **服務匯流排 XML 格式**  

  * **舊版：** Azure 串流分析使用 DataContractSerializer，因此訊息內容包含 XML 標籤。 例如︰
    
   @\u0006string\b3http://schemas.microsoft.com/2003/10/Serialization/\u0001{ "SensorId":"1", "Temperature":64\}\u0001 

  * **目前的版本：** 訊息內容直接包含串流，而不含其他標籤。 例如︰
  
   { "SensorId":"1", "Temperature":64} 
 
* **欄位名稱保持大小寫區分**  

  * **舊版：** 由 Azure 串流分析引擎處理時，欄位名稱會變更為小寫。 

  * **目前的版本：** 由 Azure 串流分析引擎處理欄位名稱時，欄位名稱可保持大小寫區分。 

  > [!NOTE] 
  > 使用 Edge 環境所裝載的串流分析作業尚無法使用保持大小寫區分的功能。 因此，如果您的作業裝載在 Edge 上，所有欄位名稱都會轉換為小寫。 

* **FloatNaNDeserializationDisabled**  

  * **舊版：** CREATE TABLE 命令並未在「浮點數 (FLOAT)」資料行型別中篩選具 NaN (不是數字 (Not-a-Number)。 例如，Infinity, -Infinity) 的事件，因其不符合這些數字的記載範圍。

  * **目前的版本：** CREATE TABLE 可讓您指定強式結構描述。 串流分析引擎會驗證資料是否符合此結構描述。 使用此模型，命令可以篩選具有 NaN 值的事件。 

* **停用 JSON 中日期時間字串的自動向上轉型 (upcast)。**  

  * **舊版：** JSON 剖析器會自動將具有日期/時間/區域資訊的字串值向上轉型 (upcast) 為日期時間型別，再將其轉換成 UTC。 這會導致時區資訊遺失。

  * **目前的版本：** 不再自動將具有日期/時間/區域資訊的字串值向上轉型 (upcast) 為日期時間型別。 如此一來，即可保留時區資訊。 

## <a name="next-steps"></a>後續步驟
* [Azure 串流分析的疑難排解指南](stream-analytics-troubleshooting-guide.md)
* [串流分析資源健康情況刀鋒視窗](stream-analytics-resource-health.md)