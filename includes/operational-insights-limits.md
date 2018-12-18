---
title: 包含檔案
description: 包含檔案
services: log-analytics
author: MGoedtel
ms.service: log-analytics
ms.topic: include
ms.date: 05/16/2018
ms.author: magoedte
ms.custom: include file
ms.openlocfilehash: 66cd09df128d454973d008adf4ffc5dd1017a18f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38755513"
---
下列限制適用於每個訂用帳戶的 Log Analytics 資源：

| 資源 | 預設限制 | 註解
| --- | --- | --- |
| 每一訂用帳戶的可用工作區數目 | 10 | 此限制無法增加。 |
| 每一訂用帳戶的付費工作區數目 | N/A | 您會受到資源群組內的資源數目和每一訂用帳戶的資源群組數目所限制 | 

>[!NOTE]
>從 2018 年 4 月 2 日起，新訂用帳戶中新的工作區將自動使用「每 GB」定價方案。  對於在 4 月 2 日之前建立的現有訂用帳戶或已繫結至現有 EA 註冊的訂用帳戶，您可以針對新的工作區繼續在這三個定價層之間進行選擇。 
>

下列限制適用於每個 Log Analytics 工作區：

|  | 免費 | 標準 | 高級 | 獨立 | OMS | 每 GB |
| --- | --- | --- | --- | --- | --- |--- |
| 每日所收集的資料量 |500 MB<sup>1</sup> |None |None | None | None | None
| 資料保留期間 |7 天 |1 個月 |12 個月 | 1 個月 <sup>2</sup> | 1 個月 <sup>2</sup>| 1 個月 <sup>2</sup>|

<sup>1</sup>當客戶達到每日 500MB 資料傳輸限制時，系統會停止資料分析，並在次日開始時繼續。 一天是以 UTC 為基礎。

<sup>2</sup> 獨立、OMS 及每 GB 定價方案的資料保留期限可增加到 730 天。

| 類別 | 限制 | 註解
| --- | --- | --- |
| 資料收集器 API | 單一篇文章的大小上限為 30 MB<br>欄位值的大小上限為 32 KB | 將較大的磁碟區分割成多篇文章<br>超過 32 KB 的欄位會被截斷。 |
| 搜尋 API | 傳回非彙總資料的 5000 筆記錄<br>彙總資料的 500000 筆記錄 | 彙總資料是包含 `summarize` 命令的搜尋
 
